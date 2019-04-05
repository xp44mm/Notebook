## The DirectoryInfo and FileInfo Types

The DirectoryInfo and FileInfo types represent individual  directories and files. Both types inherit from the FileSystemInfo abstract class  and therefore have several properties in common, namely, Name, FullName,  Extension, Exists, Attributes, CreationTime, CreationTimeUtc, LastWriteTime,  LastWriteTimeUtc, LastAccessTime, and LastAccessTimeUtc. They also have two  methods in common: Delete and Refresh, where the latter ensures that all  properties are up-to-date.

You can get a reference to a DirectoryInfo or FileInfo object by  using its constructor method, which takes the path of a specific directory or  file:

```
' Create a DirectoryInfo object that points to C:\.
Dim diRoot As New DirectoryInfo("c:\")
' Create a FileInfo object that points to c:\autoexec.bat.
Dim fiAutoexec As New FileInfo("c:\autoexec.bat")
```

Once you have a reference to a DirectoryInfo object, you can use  its methods to enumerate the folder's contents and get other DirectoryInfo or  FileInfo objects. (You can also apply filter criteria.)

```
' List the directories in c:\.
For Each di As DirectoryInfo In diRoot.GetDirectories()
   Console.WriteLine(di.Name)
Next

' List all the *.txt files in c:\.
For Each fi As FileInfo In diRoot.GetFiles("*.txt")
   Console.WriteLine(fi.Name)
Next
```

The DirectoryInfo.GetFileSystemInfos method returns an array of  FileSystemInfo objects. Both the DirectoryInfo and FileInfo types inherit from  the FileSystemInfo type, so you can write process both files and subdirectories  in a folder with a single loop:

```
For Each fsi As FileSystemInfo In diRoot.GetFileSystemInfos()
   ' Use the [dir] or [file] prefix.
   Dim prefix As String = Nothing
   If CBool(fsi.Attributes And FileAttributes.Directory) Then
      prefix = "dir"
   Else
      prefix = "file"
   End If
   ' Print type, name, and creation date.
   Console.WriteLine("[{0}] {1} – {2}", prefix, fsi.Name, fsi.CreationTime)
Next
```

Most of the members of the DirectoryInfo and FileInfo types  perform the same action as do the static methods with the same or similar names  exposed by the Directory and File types. For example, the FileInfo.CreationTime  property enables you to read and modify the creation date of a file, just like  the File object's GetCreationTime and SetCreationTime methods do. Among the few  exceptions is the FileInfo.Length property, which returns the length of a  file:

```
' List all empty files in c:\.
For Each fi As FileInfo In diRoot.GetFiles()
   If fi.Length = 0 Then Console.WriteLine(fi.Name)
Next
```

You can get the parent directory of a file in two ways: the  DirectoryName property returns the name of the directory, whereas the Directory  property returns the DirectoryInfo object that represents that directory:

```
' List all the files in the same directory as the FileInfo object named fiDoc.
For Each fi As FileInfo In fiDoc.Directory.GetFiles()
   Console.WriteLine(fi.Name)
Next
```

You can create a new folder by means of the CreateSubdirectory  method of the DirectoryInfo object:

```
' Create a folder named Reports in the c:\tempdocs directory.
Dim diDocs As New DirectoryInfo("c:\tempdocs")
diDocs.CreateSubdirectory("Reports")
```

Both the DirectoryInfo and the FileInfo types expose a MoveTo and  a Delete method, but the DirectoryInfo.Delete method can take a Boolean  argument, which, if True, causes the deletion of the entire subdirectory  tree:

```
' (Continuing previous code snippet…)
' Delete the c:\tempdocs directory and its subfolders.
diDocs.Delete(True)
```

**Version 2005  of VB or Version 2.0 of .NET** In version 2.0 of the .NET Framework,  the FileInfo object has been expanded with the IsReadOnly property (True if the  file is read-only) and three methods: Encrypt, Decrypt, and Replace. For more  details about these methods, read the description of methods with the same names  in the section titled "[The Directory and File Types](BBL0075.html#1184)" earlier in the chapter:

```
' Encrypt all the writable files in the c:\private directory.
Dim diPrivate As New DirectoryInfo("c:\private")
For Each fi As FileInfo In diPrivate.GetFiles()
   If Not fi.IsReadOnly Then fi.Encrypt()
Next
```

Finally, the FileInfo object exposes six methods that open a file,  namely, Open, OpenRead, OpenWrite, OpenText, CreateText, AppendText. They have  the same purpose as the static methods with the same names exposed by the File  type.

|      | Note | At the end of this overview of the DirectoryInfo and  FileInfo objects you might wonder whether you should use the instance methods of  these types rather than the static methods with the same names exposed by the  Directory and File types, respectively. In most cases, there is no "correct"  decision and it's mostly a matter of programming style and the specific needs  that arise in a given program. For example, I find myself more comfortable with  the Directory and File types, but I use the DirectoryInfo and FileInfo objects  if I need to buffer the data about a directory or a file, or if I need to  process files and directories in a uniform manner (as made possible by the  FileSystemInfo base class). Interestingly, the FileInfo object doesn't expose  some of the methods that have been added to the File type in .NET Framework 2.0,  such as ReadAllText or WriteAllLines; thus, in general, using the File type  gives you a little extra flexibility that is missing from the FileInfo  class. |
| ---- | ---- | ------------------------------------------------------------ |
|      |      |                                                              |