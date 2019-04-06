## The Path  Type

The Path class is the simplest type in the System.IO  namespace. It exposes static fields and methods that can help you process file  and directory paths. Four static fields return information about valid drive and  filename separators; you might want to query them to prepare your programs to  run on other operating systems if and when the .NET Framework is ported to  platforms other than the Windows operating system:

```
Console.WriteLine(Path.AltDirectorySeparatorChar)  ' => /
Console.WriteLine(Path.DirectorySeparatorChar)     ' => \
Console.WriteLine(Path.PathSeparator)              ' => ;
Console.WriteLine(Path.VolumeSeparatorChar)        ' => :
```

The GetInvalidPathChars and GetInvalidFileNameChars methods return  an array containing the characters that can't be used in paths and filenames,  respectively:

```
' Note: the actual output from following methods includes unprintable characters.
Console.WriteLine(Path.GetInvalidPathChars())      ' => <>|
Console.WriteLine(Path.GetInvalidFileNameChars())  ' => <>|:*?\/
```

The GetTempPath and GetTempFileName methods take no arguments and  return the location of the Windows temporary directory and the name of a  temporary file, respectively:

```
Console.WriteLine(Path.GetTempPath)
    ' => C:\Documents and Settings\Francesco\Local Settings\Temp
Console.WriteLine(Path.GetTempFileName)
    ' => C:\Documents and Settings\Francesco\Local Settings\Temp\tmp1FC7.tmp
```

Other methods enable you to extract information from a file path  without having to worry about whether the file or the directory exists:

```
Dim file As String = "C:\MyApp\Bin\MyApp.exe"
Console.WriteLine(Path.GetDirectoryName(file))            ' => C:\MyApp\Bin
Console.WriteLine(Path.GetFileName(file))                 ' => MyApp.exe
Console.WriteLine(Path.GetExtension(file))                ' => .exe

Console.WriteLine(Path.GetFileNameWithoutExtension(file)) ' => MyApp
Console.WriteLine(Path.GetPathRoot(file))                 ' => C:\
Console.WriteLine(Path.HasExtension(file))                ' => True
Console.WriteLine(Path.IsPathRooted(file))                ' => True
```

You can use the GetDirectoryName on files and directory names; in  the latter case, it returns the name of the parent directory. For example, you  can use this technique to retrieve the name of the main Windows directory (which  is the parent folder of the Windows System32 directory):

```
Dim winDir As String = Path.GetDirectoryName(Environment.SystemDirectory)
```

The GetFullPath method expands a relative path to an absolute  path, taking the current directory into account:

```
' Next line assumes that current directory is C:\MyApp.
Console.WriteLine(Path.GetFullPath("MyApp.Exe"))         ' => C:\MyApp\MyApp.Exe
```

The GetFullPath has a nice feature: it normalizes paths that  contain double dots and enables you to prevent attacks based on malformed paths.  For example, let's say that you must allow access to the c:\public directory and  prevent access to the c:\private folder. If you check the folder without  normalizing the path, a malicious hacker might access a file in the private  folder by providing a string such as c:\public\‥\private\*filename*.

The ChangeExtension method returns a filename with a different  extension:

```
Console.WriteLine(Path.ChangeExtension("MyApp.Exe", "dat"))  ' => MyApp.dat
```

Finally, the Combine method takes a path and a filename and  combines them into a valid filename, adding or discarding backslash characters  as required:

```
Console.WriteLine(Path.Combine("C:\MyApp", "MyApp.Dat"))    ' => C:\MyApp\MyApp.Dat
```