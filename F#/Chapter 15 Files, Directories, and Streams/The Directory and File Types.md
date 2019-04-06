## The Directory and File Types

The Directory and File types contain only static methods  that set or return information about entries in the file system. I cover both  types in one section because they share most of their methods.

### Enumerating  Directories and Files

Thanks to the GetDirectories and GetFiles methods of the  Directory type, you need very little code to iterate over all the directories  and files of a directory tree. For example, the following code displays the  structure of a directory tree and (optionally) the names of files in each  folder:

```
Sub DisplayDirTree(ByVal dir As String, ByVal showFiles As Boolean, _
   Optional ByVal level As Integer = 0)
   ' Display the name of this folder with correct indentation.
   Console.WriteLine(New String("-"c, level * 2) & dir)

Try
   ' Display all files in this folder with correct indentation.
    If showFiles Then
       For Each fname As String In Directory.GetFiles(dir)
          Console.WriteLine(New String(" "c, level * 2 + 2) & fname)
       Next
    End If
    ' A recursive call to display all the subdirectories in this folder
    For Each subdir As String In Directory.GetDirectories(dir)
       DisplayDirTree(subdir, showFiles, level + 1)
      Next
   Catch
      ' Do nothing if any error (presumably "Drive not ready").
   End Try
End Sub
```

You can pass a directory name to the DisplayDirTree procedure or  display the directory tree of all the drives in your system by using the  GetLogicalDrives method of the Directory type:

```
' Warning: this loop is going to take a *lot* of time.
For Each rootDir As String In Directory.GetLogicalDrives()
   DisplayDirTree(rootDir, True)
Next
```

The GetFiles and GetDirectories methods can take a second argument  containing wildcards to filter the result:

```
' Display all the *.txt files in C:\DOCS.
For Each fname As String In Directory.GetFiles("c:\docs", "*.txt")
   Console.WriteLine(fname)
Next
```

**Version 2005  of VB or Version 2.0 of .NET** A new, welcome addition to the GetFiles  and GetDirectories methods is the ability to automatically search in  subdirectories. For example, the following code displays all the DLLs in the  c:\windows directory tree:

```
For Each file As String In Directory.GetFiles("c:\windows", "*.dll", _
      System.IO.SearchOption.AllDirectories)
   Console.WriteLine(file)
Next
```

Notice that you must fully qualify the SearchOption argument  because both the System.IO and the Microsoft.VisualBasic.FileIO namespaces  contain a type with this name.

### Manipulating  Directories and Files

As their names suggest, the SetCurrentDirectory and  GetCurrentDirectory methods of the Directory type set and return the current  directory:

```
' Save the current directory.
Dim currDir As String = Directory.GetCurrentDirectory
' Change the current directory to something else.

Directory.SetCurrentDirectory("C:\Temp")
…
' Restore the current directory.
Directory.SetCurrentDirectory(currDir)
```

The Directory.CreateDirectory method creates a directory and all  the intermediate directories in the path if necessary:

```
' Next line works even if the C:\MyApp directory doesn't exist yet.
Directory.CreateDirectory("C:\MyApp\Data")
```

The Directory and File types have several methods in common. The  Exists method checks whether a file or a directory exists, the Delete method  removes it, and the Move method moves a file or an entire directory to a  different folder and possibly renames it in the process:

```
If File.Exists("c:\data.txt") Then
   File.Move("c:\data.txt", "d:\data.txt")
End If
```

By default, the Directory.Delete method can remove only an empty  directory, but it has an overload that enables you to remove an entire directory  tree:

```
' Delete the c:\tempdir folder and all its subfolders.
Directory.Delete("c:\tempdir", True)
```

You can use the GetCreationTime, GetLastAccessTime,  GetLastWriteTime, and GetAttributes static methods to display information about  a file or a directory or to filter files according to their attributes:

```
' Display only read-only .txt files in the c:\docs folder.
For Each fname As String In Directory.GetFiles("c:\docs", "*.txt")
   If CBool(File.GetAttributes(fname) And FileAttributes.ReadOnly) Then
      Console.WriteLine(fname)
   End If
Next
```

The SetCreationTime, SetLastWriteTime, and SetLastAccessTime  methods let you modify the date attributes of a file or directory:

```
' Change the access date and time of all files in C:\Docs.
For Each fname As String In Directory.GetFiles("C:\Docs")
   File.SetLastAccessTime(fname, Date.Now)
Next
```

You can use the SetCreationTime method to create a "touch" utility  that modifies the last write time of all the files specified on its command  line:

```
' Change the access date/time of all files whose names are passed on the command line.
Sub Main(ByVal args() As String)
   For Each fname As String In args
      File.SetCreationTime(fname, Date.Now)
   Next
End Sub
```

Each Get*Xxxx*Time and Set*Xxxx*Time method that reads or modifies a date attribute has a  matching Get*Xxxx*TimeUtc and Set*Xxxx*TimeUtc method that works with coordinated universal time  (UTC), that is, an absolute DateTime value that isn't affected by the current  time zone. These methods were added in Microsoft .NET Framework version 1.1 to  enable you to compare files that are scattered across the Internet. For example,  you can use the File.GetLastWriteUtc method to implement a replication program  that compares the files at two Internet sites in different time zones and  overwrites the older one with the newer version.

The Directory type doesn't expose a GetAttributes method, but the  File.GetAttributes method works also for directories, so this limitation isn't  an issue. The SetAttributes and GetAttributes methods set or return a bit-coded  FileAttributes value, which is a combination of Normal (no attributes), Archive,  ReadOnly, Hidden, System, Directory, Compressed, Encrypted, Temporary,  NotContentIndexed, and a few other values:

```
' Display system and hidden files in C:\.
For Each fname As String In Directory.GetFiles("C:\")
   Dim attr As FileAttributes = File.GetAttributes(fname)
   ' Display the file if marked as hidden or system (or both).
   If CBool(attr And FileAttributes.Hidden) Or CBool(attr And FileAttributes.System) Then
      Console.WriteLine(fname)
   End If
Next
```

With a little bit of tweaking, you can make the If expression more  concise, as follows:

```
If CBool(attr And (FileAttributes.Hidden Or FileAttributes.System)) Then
   …
```

The File type exposes a few methods that are missing in the  Directory type. The Copy method can copy a file and overwrite the destination if  necessary:

```
' True in the last argument means "overwrite the target file" if it exists already.
File.Copy("c:\data.bin", "c:\backup\data.bin", True)
```

**Version 2005  of VB or Version 2.0 of .NET** Three methods of the File type are new  in .NET Framework 2.0. The Replace method performs a move+copy operation as a  single command: it creates a backup copy of the destination file, and then  copies the source file to the destination file. An optional fourth argument, if  True, tells .NET to ignore any error that might occur when merging the  attributes or the ACL of the two files involved in the command:

```
' Back up the current contents of c:\data.bin into c:\data.bak, and
' then copy the contents of c:\newdata.bin into c:\data.bin.
File.Replace("c:\data.bin", "c:\newdata.bin", "c:\data.bak", True)
```

The Encrypt method encrypts a file on an NTFS file system  partition so that it can be read only by the current user; the process can be  reversed by running the Decrypt method:

```
' Ensure that no other user account can read a file during a lengthy operation.
Try
   File.Encrypt("c:\secretdata.txt")
   …

Finally
   ' Or just delete the file…
   File.Decrypt("c:\secretdata.txt")
End Try
```

### Reading and  Writing Files

**Version 2005 of VB or Version 2.0 of  .NET** In addition to the operations illustrated in the [previous section](#ch15lev2sec2), the File object  can perform atomic read and write operations on text and binary files in a very  simple manner. All the methods that enable you to perform these tasks have been  added in .NET Framework 2.0.

You read an entire text file by means of the ReadAllText method,  and write it using the WriteAllText method:

```
' Read a text file, convert its contents to uppercase, and save it to another file.
Dim text As String = File.ReadAllText("c:\testfile.txt")
File.WriteAllText("c:\upper.txt", text.ToUpper())
```

Alternatively, you can read and write an array of strings by means  of the ReadAllLines and WriteAllLines methods:

```
' Read the source file into an array of strings.
Dim lines() As String = File.ReadAllLines("c:\source.txt")
Dim count As Integer = 0
' Delete empty lines, by moving non-empty lines toward lower indices.
For i As Integer = 0 To lines.Length - 1
   If lines(i).Trim.Length > 0 Then
      lines(count) = lines(i)
      count += 1
   End If
Next
' Trim excess lines and write to destination file.
ReDim Preserve lines(count - 1)
File.WriteAllLines("c:\dest.txt", lines)
```

The AppendAllText method appends a string to an existing text file  or creates a text file if the file doesn't exist yet:

```
' Append a message to a log file, creating the file if necessary.
Dim msg As String = String.Format("Application started at {0}{1}", Now, ControlChars.CrLf)
File.AppendAllText("c:\log.txt", msg)
```

The five methods shown so far have an overloaded version that  accepts a System.Text.Encoding object. (See [Chapter 12](BBL0059.html#931), ".NET Basic Types," for  more details.) The ReadAllBytes and WriteAllBytes methods are similar, except  they work with a Byte array and therefore are more useful with binary files:

```
' Very simple encryption of a binary file
Dim bytes() As Byte = File.ReadAllBytes("c:\source.dat")
' Flip every other bit in each byte.
For i As Integer = 0 To bytes.Length - 1
   bytes(i) = bytes(i) Xor CByte(&H55)

Next
' Write it to a different file.
File.WriteAllBytes("c:\dest.dat", bytes)
```

In addition to the read and write methods that process the entire  file, the File type exposes methods that open the file for reading, writing, or  appending data and return a FileStream object. The most flexible of these  methods is the Open method, which takes a filename and up to three additional  arguments:

```
Dim fs As FileStream = File.Open(FileName, FileMode, FileAccess, FileShare)
```

Let's see these arguments in more detail:

- The FileMode argument can be Append, Create, CreateNew,  Open, OpenOrCreate, or Truncate. Open and Append modes fail if the file doesn't  exist; Create and CreateNew fail if the file exists already. Use OpenOrCreate to  open a file or to create one if it doesn't exist yet.
- The FileAccess argument specifies what the application wants  to do with the file and can be Read, Write, or ReadWrite.
- The FileShare argument tells which operations other  FileStreams can perform on the open file. It can be None (all operations are  prohibited), ReadWrite (all operations are allowed), Read, Write, Delete (new in  .NET Framework 2.0), or Inheritable (not supported directly by  Win32).

The File class exposes three variants of the Open method:  Create, OpenRead, and OpenWrite. Like the generic Open method, these variants  return a FileStream object. There are also three specific methods for working  with text files (CreateText, OpenText, and AppendText), which return a  StreamReader or StreamWriter object. I explain how to use the FileStream, the  StreamReader, and the StreamWriter objects later in this  chapter.