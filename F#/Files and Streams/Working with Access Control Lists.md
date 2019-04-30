## Working with Access Control Lists

**Version 2005 of VB or Version 2.0 of .NET** One of the most important new features in .NET Framework 2.0 is the support for reading and modifying Windows access control lists (ACLs) from managed code without having to call functions in the Windows API as was necessary in previous .NET versions. To support this new feature, Microsoft introduced the `System.Security.AccessControl` namespace, added a few types to the `System.Security.Principal` namespace, and, above all, added several methods to all the .NET types that represent system resources to which an ACL can be associated. Examples of such resources are files, directories, the registry, Active Directory objects, and many types in the `System.Threading` namespace. In this chapter, I focus on file resource exclusively, but the concepts I introduce are valid for other resource types.

### Account Names and Security Identifiers

Before you can see how to manipulate file ACLs, you must become familiar with three classes in `System.Security.Principal` namespace. The `IdentityReference` is an abstract type that represents a Windows identity and is the base class for the other two classes, `NTAccount` and `SecurityIdentifier`.

The `IdentityReference` type has one important property, Value, which returns the textual representation of the identity. The `NTAccount` type overrides this property to return an account or group name, such as [BUILTIN\Users]() or [NT AUTHORITY\SYSTEM](), whereas the `SecurityIdentifier` class overrides the property to return the textual representation of a security identifier (SID), for example, S-1-5-21-583907252-1563985344-1957994488-1003. (This representation is also known as Security Descriptor Definition Language format, or SDDL.) The following code creates an `NTAccount` object and translates it to the security identifier (SID) format by means of the `Translate` method that the `NTAccount` type inherits from `IdentityReference`:

```FSharp
#r "System.Security.Principal.Windows.dll"

let nta = new NTAccount(@"BUILTIN\Administrators")
let sia: SecurityIdentifier = nta.Translate(typeof<SecurityIdentifier>) :?> SecurityIdentifier

Console.WriteLine("Name={0}, SID={1}", nta.Value, sia.Value)
   // => Name=BUILTIN\Administrators, SID=S-1-5-32-544
```

The `NTAccount` type also exposes a constructor that takes two arguments, the domain name and the account name:

```FSharp
let nta = new NTAccount("CADomain", "Francesco")
```

Interestingly, Microsoft Visual Basic 2005 supports the equal to (=) and not equal to (<>) operators to enable you to test two `NTAccount` or two `SecurityIdentifier` objects. The `=` operator always returns false when comparing two objects of different types; therefore, you must always convert an `NTAccount` to a `SecurityIdentifier` object (or vice versa) before comparing the objects:

```FSharp
// (Continuing previous code snippet…)
let isSameAccount: Boolean = (nta = sia)              // => syntax error
let isSameAccount = nta.Translate(typeof<SecurityIdentifier>):?> SecurityIdentifier = sia
Assert.True(isSameAccount)
```

The constructor of the `SecurityIdentifier` type is overloaded to take either a SID in textual format or a `WellKnownSidType` enumerated value, such as `AccountGuestSid` (guest users), `AnonymousSid` (the anonymous account), and `LocalSystemSid` (the Local System account):

```FSharp
// Create the SecurityIdentifier corresponding to the Administrators group.
// (Second argument must be non-Nothing for some kinds of well-known SIDs.)
let sia = new SecurityIdentifier(WellKnownSidType.BuiltinAdministratorsSid, null)
Assert.Equal(sia.Value, "S-1-5-32-544")
let nta: NTAccount = sia.Translate(typeof<NTAccount>) :?> NTAccount
Assert.Equal(nta.Value, @"BUILTIN\Administrators")

// Here's another way to get a reference to the same account.
let sia = new SecurityIdentifier("S-1-5-32-544")
```

You can retrieve the `SecurityIdentifier` object corresponding to the current Windows user as follows:

```FSharp
let siUser: SecurityIdentifier = WindowsIdentity.GetCurrent().User
```

Another common use of the `SecurityIdentifier` type is for checking whether a user is in a given group or role:

```FSharp
// Create the WindowsPrincipal corresponding to current user.
let wp = new WindowsPrincipal(WindowsIdentity.GetCurrent())
// Create the SecurityIdentifier for the BUILTIN\Administrator group.
let siAdmin = new SecurityIdentifier(WellKnownSidType.BuiltinAdministratorsSid, null)
// Check whether the current user is an administrator.
Assert.False(wp.IsInRole(siAdmin))
```

### The DirectorySecurity and FileSecurity Types

The `System.Security.AccessControl` namespace includes nearly all the types that let you control the ACLs associated with a Windows resource, such as the `FileSecurity`, `DirectorySecurity`, and `RegistrySecurity` types. (Only two ACL-related types aren't in this namespace, namely, `System.DirectoryServices.ActiveDirectorySecurity` and `Microsoft.Iis.Metabase.MetaKeySecurity`.) In this section, I focus on the `FileSecurity` and `DirectorySecurity` objects, which, not surprisingly, are very similar. In fact, both of them inherit from the `FileSystemSecurity` class, which, in turn, inherits from `NativeObjectSecurity`.

You can get a reference to a `FileSecurity` or `DirectorySecurity` object in one of the following two ways. First, you can pass a path to its constructor, together with an `AccessControlSection` enumerated value that specifies which security information you're interested in:

```FSharp
#r "System.IO.FileSystem.AccessControl.dll"

// Retrieve only access information related to the c:\docs folder.
let dirSec = new DirectorySecurity(@"c:\docs", AccessControlSections.Access)
// Retrieve all security information related to the c:\test.doc file.
let fileSec = new FileSecurity(@"c:\test.txt", AccessControlSections.All)
```

(Valid values for `AccessControlSections` are `Access`, `Owner`, `Audit`, `Group`, `All`, and `None`.) Second, you can use the `GetAccessControl` method exposed by the ~~Directory, File,~~ `DirectoryInfo`, and `FileInfo` types:

```FSharp
// (This code is equivalent to previous snippet.)
let dirSec = DirectoryInfo(@"c:\docs").GetAccessControl(AccessControlSections.Access)
let fileSec = FileInfo(@"c:\test.txt").GetAccessControl(AccessControlSections.All)
```

The simplest operation you can perform with a `FileSecurity` or a `DirectorySecurity` object is retrieving the discretionary access control list (DACL) or system access control list (SACL) associated with the resource in SDDL format:

```FSharp
//run test as admin
// Get access-related security information for the c:\test.txt file.
let sd = fileSec.GetSecurityDescriptorSddlForm(AccessControlSections.Access)
Assert.Equal(sd,"D:(A;ID;FA;;;BA)(A;ID;FA;;;SY)(A;ID;0x1301bf;;;AU)(A;ID;0x1200a9;;;BU)")
```



---

##### Note

A discretionary access control list (DACL) defines who is granted or denied access to an object. Each Windows object is associated with a DACL, which consists of a list of access control entries (ACEs); each ACE defines a trustee and specifies the access rights that are granted, denied, or audited for that trustee. If the object has no DACL, everyone can use the object; otherwise, each ACE is tested until the user (or the process that is impersonating the user) is granted the access to the object. If no ACE grants this permission, the user is prevented from using the object. A system access control list (SACL) enables administrators to log attempts to use a given object. A SACL contains one or more ACEs; each ACE specifies a trustee and the type of access (from that trustee) that causes the system to create an entry in the security event log. The entry in the log can be generated when the access succeeds, fails, or both.

---



An SDDL string is rarely useful, though, or at least it is hard for humans to decode. To get the ACL in readable format you can use one of the following methods: `GetOwner` (to retrieve the owner of the resource), `GetGroup` (to retrieve the primary group associated with the owner), `GetAccessRules` (to retrieve the collection of access rules), and `GetAuditRules` (to retrieve the collection of audit rules). These three methods have similar syntax.



The `GetOwner` and `GetGroup` methods return a single object that derives from `IdentityReference`, therefore either an `NTAccount` or a `SecurityIdentifier` object. You specify which object you want to be returned by passing a proper `System.Type` object as an argument:

```FSharp
// Get the owner of the C:\Test.doc as an NTAccount object.
let nta: NTAccount = fileSec.GetOwner(typeof<NTAccount>) :?> NTAccount

// Get the primary group of the owner of C:\Test.doc as a SecurityIdentifier object.
let sia: SecurityIdentifier = fileSec.GetGroup( typeof<SecurityIdentifier>) :?> SecurityIdentifier
```

Once you have an `NTAccount` or a `SecurityIdentifier` object you can query all of its properties, as shown in the previous section.

The `GetAccessRules` method returns a collection of `AccessRule` objects, where each individual member in the collection tells whether a given action is granted or denied to a given user or group of users. Similar to the `GetOwner` and `GetGroup` methods, you must pass a `System.Type` object that specifies whether the user name or group is expressed by means of an `NTAccount` object or a `SecurityIdentifier` object:

```FSharp
// First argument tells whether to include access rules explicitly set for the object.
// Second argument tells whether to include inherited rules.
let rules = fileSec.GetAccessRules(true, true, typeof<NTAccount>)
for fsar: AuthorizationRule in rules do
    let fsar = fsar :?> FileSystemAccessRule
    output.WriteLine("{0,-25}{1,-30}{2,-8}{3,-6}", fsar.IdentityReference.Value,  fsar.FileSystemRights, fsar.AccessControlType, fsar.IsInherited)
```

The `FileSystemAccessRule.FileSystemRights` property returns a bit-coded `FileSystemRights` enumerated type that exposes values such as `Read`, `Write`, `Modify`, `Delete`, `ReadAndExecute`, `FullControl`, `ReadAttributes`, `WriteAttributes`, and many others. (See Table 15-1.) The `FileSystemAccessRule.AccessControlType` returns an enumerated value that can only be `Allow` or `Deny`. Here's the kind of output the previous code produces in the console window:

```FSharp
User                     Rights                         Access Inherited
------------------------------------------------------------------------
BUILTIN\Administrators   FullControl                    Allow  true
NT AUTHORITY\SYSTEM      FullControl                    Allow  true
DESKTOP01\FrancescoB     Write                          Deny   false
BUILTIN\Users            ReadAndExecute, Synchronize    Allow  true
```



##### Table 15-1: Values of the FileSystemRights Enumerated Type 

* `AppendData`
  Specifies the right to append data to the end of a file. 


* `ChangePermissions`
  Specifies the right to change the security and audit rules associated with a file or folder. 


* `CreateDirectories`
  Specifies the right to create a folder. This right requires the `Synchronize` right. If you don't explicitly set the `Synchronize` right when creating a file or folder, the `Synchronize` right will be set automatically for you. 


* `CreateFiles`
  Specifies the right to create a file. This right requires the `Synchronize` right. If you don't explicitly set the `Synchronize` right when creating a file or folder, the `Synchronize` right will be set automatically for you. 


* `Delete`
  Specifies the right to delete a folder or file. 


* `DeleteSubdirectoriesAndFiles`
  Specifies the right to delete a folder and any files contained within that folder. 


* `ExecuteFile`
  Specifies the right to run an application file. 


* `FullControl`
  Specifies the right to exert full control over a folder or file and to modify access control and audit rules. 


* `ListDirectory`
  Specifies the right to list the contents of a folder. 


* `Modify`
  Specifies the right to read, write, list folder contents, delete folders and files, and run application files. 


* `Read`
  Specifies the right to open and copy folders or files as read-only. It includes the right to read file system attributes, extended file system attributes, and access and audit rules. 


* `ReadAndExecute`
  Specifies the right to open and copy folders or files as read-only and to run application files. It includes the right to read file system attributes,extended file system attributes, and access and audit rules. 


* `ReadAttributes`
  Specifies the right to open and copy file system attributes from a folder or file. It doesn't include the right to read data, extended file system attributes, or access and audit rules. 


* `ReadData`
  Specifies the right to open and copy a file or folder. It doesn't include the right to read file system attributes, extended file system attributes, or access and audit rules. 


* `ReadExtendedAttributes`
  Specifies the right to open and copy extended file system attributes from a folder or file. It doesn't include the right to read data, file system attributes, or access and audit rules. 


* `ReadPermissions`
  Specifies the right to open and copy access and audit rules from a folder or file. It doesn't include the right to read data, file system attributes, and extended file system attributes. 


* `Synchronize`
  Specifies the right to synchronize a file or folder. The right to create a file or folder requires this right. If you don't explicitly set this right when creating a file, the right will be set automatically for you. 


* `TakeOwnership`
  Specifies the right to change the owner of a folder or file. 


* `Traverse`
  Specifies the right to list the contents of a folder and to run applications contained within that folder. 


* `Write`
  Specifies the right to create folders and files and to add or remove data from files. It includes the ability to write file system attributes, extended file system attributes, and access and audit rules. 


* `WriteAttributes`
  Specifies the right to open and write file system attributes to a folder or file. It doesn't include the ability to write data, extended attributes, or access and audit rules. 


* `WriteData`
  Specifies the right to open and write to a file or folder. It doesn't include the right to open and write file system attributes, extended file system attributes, or access and audit rules. 


* `WriteExtendedAttributes`
  Specifies the right to open and write extended file system attributes to a folder or file. It doesn't include the ability to write data, attributes, or access and audit rules. 

You can compare these results with the actual permissions set for the specific file. To do so, right-click the file in Windows Explorer, select the Properties command from the context menu, and switch to the Security tab, as shown in Figure 15-2. (If you don't see this tab, select the Folder Options command from the Tools menu in Windows Explorer, switch to the View tab, and ensure that the Use Simple File Sharing option is cleared.) Some attributes are visible in the Advanced Security Settings dialog box, which you display by clicking the Advanced button.



![Image from book](images/fig631%5F01%5F0%`2Ejpg`) 
Figure 15-2: The Security tab of the Properties dialog box (left) and the Advanced Security Settings dialog box (right) of a file 



You can change security-related information as well. For example, the `FileSecurity` object exposes the `SetOwner` method for changing the owner of a file:

```FSharp
// Transfer the ownership of the c:\test.doc file to the System account.
let nta = new NTAccount(@"NT AUTHORITY\SYSTEM")
fileSec.SetOwner(nta)
```



### Modifying ACLs

You can do more than just change the owner of a file or directory object. In fact, you can specify exactly who can (or can't) do what, by creating or manipulating a `FileSecurity` or `DirectorySecurity` object and associating it with a file or directory. This is made possible by the `SetAccessControl` method exposed by the ~~Directory, File,~~ `DirectoryInfo`, and `FileInfo` types.

The simplest technique for changing the ACL of a file or a directory is by cloning the ACL obtained from another object, as in this code:

```FSharp
// Create a copy of the all permissions associated with c:\test.doc.
// (You can also copy just the access permissions, for example.)
let sddl: String = fileSec.GetSecurityDescriptorSddlForm(AccessControlSections.All)
let fileSec2 = new FileSecurity()
fileSec2.SetSecurityDescriptorSddlForm(sddl)
// Enforce these permissions on the c:\data.txt file.
FileInfo(@"c:\data.txt").SetAccessControl(fileSec2)
```

For tasks that are more complex than just copying an existing ACL you create individual `FileSystemAccessRule` objects and pass them to the `FileSecurity.AddAccessRule` method:

```FSharp
// Create an access rule that grants full control to administrators.
let ntAcc1 = new NTAccount(@"BUILTIN\Administrators")
let fsar1 = new FileSystemAccessRule(ntAcc1, FileSystemRights.FullControl,  AccessControlType.Allow)
// Create another access rule that denies write permissions to ASPNET user.
let ntAcc2 = new NTAccount(@"DESKTOP01\ASPNET")
let fsar2 = new FileSystemAccessRule(ntAcc2, FileSystemRights.Write,  AccessControlType.Deny)
// Create a FileSecurity object that contains these two access rules.
let fsec = new FileSecurity()
fsec.AddAccessRule(fsar1)
fsec.AddAccessRule(fsar2)
// Assign these permissions to the c:\data.txt file.
FileInfo(@"c:\data.txt").SetAccessControl(fsec)
```

An overload of the constructor lets you specify how permissions inherited from the parent object (the containing directory, in the case of the file system) should be dealt with.

```FSharp
let fsar = 
    new FileSystemAccessRule(
        ntAcc2,  
        FileSystemRights.Write ||| FileSystemRights.Read,  
        InheritanceFlags.ContainerInherit ||| InheritanceFlags.ObjectInherit,  
        PropagationFlags.None, 
        AccessControlType.Allow
    )
```

The `ContainerInherit` flag means that the rule is propagated to all containers that are children of the current object, whereas the `ObjectInherit` flag means that the rule is propagated to all objects that are children of the current object. In the case of the file system, if the current object is a folder, the `ContainerInherit` flag affects its subdirectories and the `ObjectInherit` flag affects the files contained in the folder.

The `PropagationFlags.None` flag means that the rule applies to both the object and its children. The other two values for this flag are `InheritOnly` and `NoPropagateInherit`. The former value means that the rule applies to child objects but not to the object itself, thus you can enforce a rule for all the files and directories in a folder without affecting the folder itself; the latter value means that rule inheritance applies for only one level, therefore the rule affects the children of an object but not its grandchildren. In practice, these two flags aren't used often.

The `FileSecurity` type exposes other methods that enable you to modify the set of access rules contained in the object: `ModifyAccessRule` changes an existing access rule; `PurgeAccessRules` removes all the rules associated with a given `IdentityReference` object; `RemoveAccessRuleSpecific` removes a specific access rule; `ResetAccessRule` adds the specified access rule and removes all the matching rules in one operation. Read MSDN documentation for details about these methods.

The `FileSecurity` object is also capable of reading and modifying the audit rules associated with a file or directory. (Audit rules specify which file operations on a file, either successful or not, are logged by the system.) For example, the following code displays all the audit rules associated with a file:

```FSharp
Console.WriteLine("{0,-25}{1,-30}{2,-8}{3,-6}", "User", "Rights", "Outcome", "Inherited")
Console.WriteLine(new String('-', 72))
for fsar in fsec.GetAuditRules(true, true, typeof<NTAccount>) do
    let fsar = fsar :?> FileSystemAuditRule
    Console.WriteLine("{0,-25}{1,-30}{2,-8}{3,-6}", 
        fsar.IdentityReference.Value,  
        fsar.FileSystemRights, 
        fsar.AuditFlags, 
        fsar.IsInherited)
```

The `AuditFlags` property is an enumerated value that can be `Success` or `Failure`. Similarly to what happens with access rules, you can use the `AddAuditRule` method to add an audit rule to the `FileSecurity` object, the `ModifyAuditRule` method to change an existing audit rule, and so forth.