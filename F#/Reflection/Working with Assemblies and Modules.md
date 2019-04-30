## Working  with Assemblies and Modules





The types in the System.Reflection namespace form a logical  hierarchy, at the top of which you find the Assembly class, as you can see in [Figure 18-1](#ch18fig01). All the classes in the  hierarchy shown in the figure belong to the System.Reflection namespace, except  System.Type. FieldInfo, PropertyInfo, and EventInfo inherit from the MemberInfo  abstract class, whereas MethodInfo and ConstructorInfo inherit from the  MethodBase abstract class (which in turn derives from MemberInfo).

![Image from book](./images/fig748_01.jpg)
Figure 18-1: The Reflection logical  hierarchy

In this section, I describe the Assembly, AssemblyName, and Module  classes.

### The Assembly  Type

As its name implies, the Assembly type represents a .NET  assembly. This type offers no constructor method because you never actually  create an assembly, but simply get a reference to an existing assembly. There  are many ways to perform this operation, as described in the following  section.

#### Loading an  Assembly

The Assembly type exposes several static methods that return  a reference to an assembly, either running or not (that is, stored on disk but  currently not running):

```
' Get a reference to the assembly this code is running in.
Dim asm As Assembly = Assembly.GetExecutingAssembly()

' Get a reference to the assembly a type belongs to.
asm = Assembly.GetAssembly(GetType(System.Data.DataSet))
' Another way to reach the same result.
asm = GetType(System.Data.DataSet).Assembly

' Get a reference to an assembly given its display name.
' (The argument can be the assembly's full name, which includes
' version, culture, and public key.)
asm = Assembly.Load("mscorlib")

' Get a reference to an assembly given its filename or its full name.
asm = Assembly.LoadFrom("c:\myapp\mylib.dll")

' Another way to get a reference to an assembly given its path. (See text for notes.)
asm = Assembly.LoadFile("c:\myapp\mylib.dll")
```

A few subtle differences exist among the Load, LoadFrom, and  LoadFile methods, and also a few minor changes from .NET Framework 1.1 might  impact the way existing applications behave, as explained in the following list.  These differences have to do with how the assembly is located and the binding  context in which the assembly is loaded. The *binding* *context* works like a cache for loaded assemblies so that the  .NET runtime doesn't have to locate the same assembly again and again each time  the application asks for it. (See the section titled "[Previously Loaded Assemblies and GAC  Searches](BBL0089.html#1410)" in [Chapter 17](BBL0085.html#1344), "Assemblies and Resources.")

- The Load method takes the assembly name, either the short  name or the fully qualified name (that includes version, culture, and public key  token). If a fully qualified name is provided, this method searches the GAC  first and, in general, it follows the same probing sequence that the .NET  runtime applies when loading an assembly because your code references one of its  types. (See [Chapter  17](BBL0085.html#1344) for details about the probing process.) Assemblies loaded with the Load  method become part of the execution context; the main advantage of assemblies  loaded in this context is that the .NET runtime is able to resolve their  dependencies. You can enumerate assemblies in the execution context with this  code:

  ```
  For Each refAsm As Assembly In AppDomain.CurrentDomain.GetAssemblies()
     Console.WriteLine(refAsm.FullName)
  Next
  ```

- The LoadFrom method takes either a relative or absolute file  path; if relative, the assembly is searched in the application's base directory.  An assembly loaded with this method becomes part of the LoadFrom context. If an  NGen image for the assembly exists, it won't be used, but if an assembly with  the same identity can be found by probing or is already loaded in the LoadFrom  context, the method returns that assembly instead, a behavior that can be quite  confusing. When an assembly in the LoadFrom context is executed, the .NET  runtime is able to locate its dependencies correctly only if they are under the  application's base directory or are already loaded in the LoadFrom  context.

  

  

- The LoadFile method also takes a file path. It works  similarly to LoadFrom, but the assembly is loaded in a different context, and  the .NET runtime is unable to find its dependencies, unless they are already  loaded in the Load context or you handle the AssemblyResolve event of the  AppDomain object. (See the [next  section](#ch18lev3sec2) for more details about this event.)

To make things more complicated, the behavior of these methods has  changed slightly in .NET Framework 2.0. First, both LoadFrom and LoadFile apply  the probing policy (which they ignored in .NET Framework 1.1). Second, these  methods check the identity of the assembly and load the assembly from the GAC if  possible. There is a small probability that these minor changes may break your  existing code, so pay attention when you are migrating your reflection-intensive  applications to Microsoft Visual Basic 2005.

Another minor difference from version 1.1 is that if loading of an  assembly fails once, it will continue to fail even if you remedy the error,  until that AppDomain exists. In other words, you can't just trap the exception  and ask the user to install the requested assembly in the GAC or in the  application's base directory. Instead, you'll have to restart the application or  at least load the assembly in a different AppDomain.

**Version 2005  of VB or Version 2.0 of .NET** .NET Framework 2.0 has the ability to  load an assembly for inspection purposes only, using either the  ReflectionOnlyLoad or the ReflectionOnlyLoadFrom method:

```
' Load the System.Data.dll for reflection purposes.
asm = Assembly.ReflectionOnlyLoad(_
   "System.Data, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089")

' Load a file for reflection purposes, given its path.
asm = Assembly.ReflectionOnlyLoadFrom("c:\myapp\mylib.dll")
```

You can enumerate members and perform most other  reflection-related operations when you load an assembly in this way, but you  can't instantiate a type in these assemblies and therefore you can't execute any  code inside them. Unlike the LoadFrom and LoadFile methods, you are allowed to  load the assembly even though Code Access Security (CAS) settings would  otherwise prevent you from doing so.

Another difference from other load methods is that the  ReflectionOnlyLoad and ReflectionOnlyLoadFrom methods ignore the binding policy.  Thus, you can load exactly the assemblies you are pointing to—you can even  inspect assemblies compiled for a different process architecture—except if you  load an assembly with the same identity as one that is already loaded in the  inspection context, the latter assembly is returned.





Assemblies loaded with these two methods become part of yet  another context, known as the inspection context. There is exactly one  inspection context in each AppDomain, and you can enumerate the assemblies it  contains with this code:

```
For Each refAsm As Assembly In AppDomain.CurrentDomain.ReflectionOnlyGetAssemblies()
   Console.WriteLine(refAsm.FullName)
Next
```

The Assembly object has a new read-only property,  ReflectionOnly, which returns True if the assembly is loaded in the inspection  context.

#### AppDomain  Events

When the .NET runtime successfully loads an assembly, either  as the result of a JIT-compilation action or while executing an Assembly.Load*Xxxx* method, the AppDomain instance that represents the  current application domain fires an AssemblyLoad event. You can use this event  to determine exactly when an assembly is loaded, for example, for diagnostics  purposes:

```
Sub TestAppDomainLoadAssemblyEvent()
   ' Subscribe to the AppDomain.AssemblyLoad event.
   Dim appDom As AppDomain = AppDomain.CurrentDomain
   AddHandler appDom.AssemblyLoad, AddressOf AppDomain_AssemblyLoad
   ' This statement causes the JIT compilation of DoSomethingWithXml method,
   ' which in turn loads the System.Xml.dll assembly.
   DoSomethingWithXml()
   ' Unsubscribe from the event.
   RemoveHandler appDom.AssemblyLoad, AddressOf AppDomain_AssemblyLoad
End Sub

Private Sub DoSomethingWithXml()
   ' This statement causes the loading of the System.Xml.dll assembly
   ' (assuming that no other XML-related type has been used already and that
   ' the program has been compiled in Release mode).
   Dim doc As New System.Xml.XmlDocument()
End Sub

Sub AppDomain_AssemblyLoad(ByVal sender As Object, ByVal e As AssemblyLoadEventArgs)
   Console.WriteLine("Assembly {0} is being loaded", e.LoadedAssembly.Location)
End Sub
```

Notice that methods in applications compiled in Debug mode are  JIT-compiled earlier; therefore, the previous code snippet delivers the expected  results only if compiled in Release mode.

When the .NET runtime isn't able to load an assembly, the current  AppDomain instance fires an AssemblyResolve event. By handling this event, you  can tell the CLR where the assembly is located. For example, let's suppose that  you want to force an application to search for dependent assemblies—either  private or shared—in a given folder. As you might recall from [Chapter 17](BBL0085.html#1344), by  default weakly typed assemblies must be located in the application's folder or  one of its subfolders, but the AssemblyResolve event enables you effectively to  override the .NET standard binding policy. The handler for this event is  peculiar: it is implemented as a Function  that returns an Assembly instance (the assembly that our code loaded manually)  or Nothing if the load operation must fail:

```
Sub AppDomainAssemblyResolveEvent()
   ' Subscribe to the AppDomain.AssemblyResolve event.
   Dim appDom As AppDomain = AppDomain.CurrentDomain
   AddHandler appDom.AssemblyResolve, AddressOf AppDomain_AssemblyResolve
   ' Attempt to load an assembly that isn't in the private path.
   Dim asm As Assembly = Assembly.Load("EvaluatorLibrary")
   Console.WriteLine("Found {0} assembly at {1}", asm.FullName, asm.Location)
   ' Unsubscribe from the event.
   RemoveHandler appDom.AssemblyResolve, AddressOf AppDomain_AssemblyResolve
End Sub

Private Function AppDomain_AssemblyResolve(ByVal sender As Object, _
      ByVal e As ResolveEventArgs) As Assembly
   ' Search the assembly in a different directory.
   Dim searchDir As String = "c:\myassemblies"
   For Each dllFile As String In Directory.GetFiles(searchDir, "*.dll")
      Try
         Dim asm As Assembly = Assembly.LoadFile(dllFile)
         ' If the DLL is an assembly and its name matches, we've found it.
         If asm.GetName().Name = e.Name Then Return asm
      Catch ex As Exception
         ' Ignore DLLs that aren't valid assemblies.
      End Try
   Next
   ' If we get here, return Nothing to signal that the search failed.
   Return Nothing
End Function
```

The AssemblyResolve event lets you do wonders, if used  appropriately. For example, you might load an assembly from a network share; or  you might store all your assemblies in a database binary field and load them  when needed, leveraging the Assembly.Load overload that takes a Byte array as an  argument.

**Version  2005 of VB or Version 2.0 of .NET** The AppDomain type also exposes the  ReflectionOnlyAssemblyResolve event. As its name suggests, this event is similar  to AssemblyResolve, except it fires when the resolution of an assembly fails in  the reflection-only context, that is, when the ReflectionOnlyLoad or the  ReflectionOnlyLoadFrom method fails. The ReflectionOnlyAssemblyResolve event  also fires when the .NET runtime successfully locates the assembly you're  loading for reflection-only purposes but fails to load one of the assemblies  that the target assembly depends on.

#### Properties and  Methods

Once you have a valid reference to an Assembly object, you  can query its properties to learn additional information about it. For example,  the FullName property returns a string that holds information about the version  and the public key token (this data is the same as the string returned by the  ToString method).





```
' This is the ADO.NET assembly.
asm = GetType(System.Data.DataSet).Assembly
Console.WriteLine(asm.FullName)
   ' => System.Data, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089
```

The Location and CodeBase read-only properties both return the  actual location of the assembly's file, so you can learn where assemblies in the  GAC are actually stored, for example:

```
Console.WriteLine(asm.Location)
   ' => C:\WINDOWS\assembly\GAC_32\System.Data\2.0.0.0__ b77a5c561934e089\System.Data.dll
```

When you're not working with assemblies downloaded from the  Internet, the information these properties return differs only in format:

```
' …(Continuing previous example)…
Console.WriteLine(asm.CodeBase)
   ' => file:///C:/WINDOWS/assembly/GAC_32/System.Data/2.0.0.0__
        b77a5c561934e089/System.Data.dll
```

The GlobalAssemblyCache property returns a Boolean value that  tells you whether the assembly was loaded from the GAC. The ImageRuntimeVersion  returns a string that describes the version of the CLR stored in the assembly's  manifest (for example, v.2.0.50727). The EntryPoint property returns a  MethodInfo object that describes the entry point method for the assembly, or it  returns Nothing if the assembly has no entry point (for example, if it's a DLL  class library). MethodInfo objects are described in the section titled "[Enumerating  Members](BBL0095.html#1471)" later in this chapter.

The Assembly class exposes many instance methods, the majority of  which enable you to enumerate all the modules, files, and types in the assembly.  For example, the GetTypes method returns an array with all the types (classes,  interfaces, and so on) defined in an assembly:

```
' Enumerate all the types defined in an assembly.
For Each ty As Type In asm.GetTypes()
   Console.WriteLine(ty.FullName)
Next
```

You can also list only the public types that an assembly exports  by using the GetExportedTypes method.

The Assembly class overloads the GetType method inherited from  System.Object so that it can take a type name and return the specified Type  object.

```
' Next statement assumes that the asm variable is pointing to System.Data.dll.
Dim ty2 As Type = asm.GetType("System.Data.DataTable")
```

If the assembly doesn't contain the specified type, the GetType  method returns Nothing. By passing True as its second argument, you can have  this method throw a TypeLoadException if the specified type isn't found, and you  can have the type name compared in a case-insensitive way by passing True as a  third argument:

```
' This statement doesn't raise any exception because type name
' is compared in a case-insensitive way.
Dim ty3 As Type = asm.GetType("system.data.datatable", True, True)
```

Finally, two methods of the Assembly class return an  AssemblyName object, which is described in the [next section](#ch18lev2sec2).

### The AssemblyName  Type

The AssemblyName class represents the object that .NET uses  to hold the identity and to retrieve information about an assembly. A fully  specified AssemblyName object has a name, a culture, and a version number, but  the runtime can also use partially filled AssemblyName objects when searching  for an assembly to be bound to caller code. Most often, you get a reference to  an existing AssemblyName object by using the GetName property of the Assembly  object:

```
' Get a reference to an assembly and its AssemblyName.
Dim asm As Assembly = Assembly.Load("mscorlib")
Dim an As AssemblyName = asm.GetName()
```

You can also get an array of AssemblyName objects using the  GetReferencedAssemblies method:

```
' Get information on all the assemblies referenced by the current assembly.
Dim anArr() As AssemblyName
anArr = Assembly.GetExecutingAssembly().GetReferencedAssemblies()
```

Most of the properties of the AssemblyName type are  self-explanatory, and some of them are also properties of the Assembly type (as  is the case of the FullName and CodeBase properties):

```
Console.WriteLine(an.FullName)
   ' => mscorlib, Version=2.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089

' The ProcessorArchitecture property is new in .NET Framework 2.0.
' It can be MSIL, X86, IA64, Amd64, or None.
Console.WriteLine(an.ProcessorArchitecture.ToString())   ' => X86

' These properties come from the version object.
Console.WriteLine(an.Version.Major)                      ' => 2
Console.WriteLine(an.Version.Minor)                      ' => 0
Console.WriteLine(an.Version.Build)                      ' => 0
Console.WriteLine(an.Version.Revision)                   ' => 0
' You can also get the version as a single number.
Console.WriteLine(an.Version)                            ' => 2.0.0.0
```

A few methods of the AssemblyName object return a Byte array. For  example, you can get the public key and the public key token by using the  GetPublicKey and GetPublicKeyToken methods:

```
' Display the public key token of the assembly.
For Each b As Byte In an.GetPublicKeyToken()
   Console.Write("{0} ", b)
Next
```

The CultureInfo property gets or sets the culture supported by the  assembly, or returns Nothing if the assembly is culture-neutral.





Unlike most other reflection types, the AssemblyName type has a  constructor, which lets you create an AssemblyName instance from the display  name of an assembly:

```
Dim an2 As New AssemblyName("mscorlib, Version=2.0.0.0, Culture=neutral, " _
   & "PublicKeyToken=b77a5c561934e089, ProcessorArchitecture=x86")
```

### The Module  Type

The Module class represents one of the modules in an  assembly; don't confuse it with a Visual Basic Module block, which corresponds  to a Type object. You can enumerate all the elements in an assembly by using the  Assembly.GetModules method:

```
' Enumerate all the modules in the mscorlib assembly.
Dim asm As Assembly = Assembly.Load("mscorlib")
' (Note that Module is a reserved word in Visual Basic.)
For Each mo As [Module] In asm.GetModules()
   Console.WriteLine("{0} – {1}", mo.Name, mo.ScopeName)
Next
```

The preceding code produces only one output line:

```
mscorlib.dll - CommonLanguageRuntimeLibrary
```

The Name property returns the name of the actual DLL or EXE,  whereas the ScopeName property is a readable string that represents the module.  The vast majority of .NET assemblies (and all the assemblies you can build with  Microsoft Visual Studio 2005 without using the Assembly Linker tool) contain  only one module. This module is the one that contains the assembly manifest, and  you can get a reference to it by means of the Assembly.ManifestModule  property:

```
Dim manModule As [Module] = asm.ManifestModule
```

In general, you rarely need to work with the Module type, and  I won't cover it in more detail in this book.