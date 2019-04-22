## The Directory and File Types

The `Directory` and `File` types contain only static methods that set or return information about entries in the file system. I cover both types in one section because they share most of their methods.

### Enumerating Directories and Files

Thanks to the `GetDirectories` and `GetFiles` methods of the `Directory` type, you need very little code to iterate over all the directories and files of a directory tree. For example, the following code displays the structure of a directory tree and (optionally) the names of files in each folder:

```FSharp
let rec DisplayDirTree(dir : String, showFiles : Boolean,  level : Int32) =
    // Display the name of this folder with correct indentation.
    output.WriteLine(String('-', level * 2) + dir)

    try
        // Display all files in this folder with correct indentation.
        if showFiles then
            for fname: String in Directory.GetFiles(dir) do
                output.WriteLine(String(' ', level * 2 + 2) + fname)

        // A recursive call to display all the subdirectories in this folder
        for subdir: String in Directory.GetDirectories(dir) do
            DisplayDirTree(subdir, showFiles, level + 1)
    with _ ->
        // Do nothing if any error (presumably "Drive not ready").
        ()
```

You can pass a directory name to the `DisplayDirTree` procedure or display the directory tree of all the drives in your system by using the `GetLogicalDrives` method of the `Directory` type:

```FSharp
// Warning: this loop is going to take a *lot* of time.
for rootDir: String in Directory.GetLogicalDrives() do
   DisplayDirTree(rootDir, true, 0)
```

The `GetFiles` and `GetDirectories` methods can take a second argument containing wildcards to filter the result:

```FSharp
// Display all the *.txt files in C:\DOCS.
for fname: String in Directory.GetFiles(@"c:\docs", "*.txt") do
   Console.WriteLine(fname)
```

**Version 2005 of VB or Version 2.0 of .NET** A new, welcome addition to the `GetFiles` and `GetDirectories` methods is the ability to automatically search in subdirectories. For example, the following code displays all the `DLLs` in the c:\windows directory tree:

```FSharp
for file: String in Directory.GetFiles(@"c:\windows", "*.dll",  System.IO.SearchOption.AllDirectories) do
   Console.WriteLine(file)
```

Notice that you must fully qualify the `SearchOption` argument because both the `System.IO` and the `Microsoft.VisualBasic.FileIO` namespaces contain a type with this name.

### Manipulating Directories and Files

As their names suggest, the `SetCurrentDirectory` and `GetCurrentDirectory` methods of the `Directory` type set and return the current directory:

```FSharp
// Save the current directory.
let currDir: String = Directory.GetCurrentDirectory()
// Change the current directory to something else.

Directory.SetCurrentDirectory(@"C:\Temp")
…
// Restore the current directory.
Directory.SetCurrentDirectory(currDir)
```

The `Directory.CreateDirectory` method creates a directory and all the intermediate directories in the path if necessary:

```FSharp
// Next line works even if the C:\MyApp directory doesn't exist yet.
Directory.CreateDirectory(@"C:\MyApp\Data")
```

The `Directory` and `File` types have several methods in common. The `Exists` method checks whether a file or a directory exists, the `Delete` method removes it, and the `Move` method moves a file or an entire directory to a different folder and possibly renames it in the process:

```FSharp
if File.Exists(@"c:\data.txt") then
   File.Move(@"c:\data.txt", @"d:\data.txt")
```

By default, the `Directory.Delete` method can remove only an empty directory, but it has an overload that enables you to remove an entire directory tree:

```FSharp
// Delete the c:\tempdir folder and all its subfolders.
Directory.Delete(@"c:\tempdir", true)
```

You can use the `GetCreationTime`, `GetLastAccessTime`, `GetLastWriteTime`, and `GetAttributes` static methods to display information about a file or a directory or to filter files according to their attributes:

```FSharp
// Display only read-only .txt files in the c:\docs folder.
for fname: String in Directory.GetFiles(@"c:\docs", "*.txt") do
   if File.GetAttributes(fname) &&& FileAttributes.ReadOnly = FileAttributes.ReadOnly then
      Console.WriteLine(fname)
```

The `SetCreationTime`, `SetLastWriteTime`, and `SetLastAccessTime` methods let you modify the date attributes of a file or directory:

```FSharp
// Change the access date and time of all files in C:\Docs.
for fname: String in Directory.GetFiles(@"C:\Docs") do
   File.SetLastAccessTime(fname, DateTime.Now)
```

You can use the `SetCreationTime` method to create a "touch" utility that modifies the last write time of all the files specified on its command line:

```FSharp
// Change the access date/time of all files whose names are passed on the command line.
let Main(filenames : String[]) =
   for fname: String in filenames do
      File.SetCreationTime(fname, DateTime.Now)
```

Each `GetXxxxTime` and `SetXxxxTime` method that reads or modifies a date attribute has a matching `GetXxxxTimeUtc` and `SetXxxxTimeUtc` method that works with coordinated universal time (UTC), that is, an absolute `DateTime` value that isn't affected by the current time zone. These methods were added in Microsoft .NET Framework version 1.1 to enable you to compare files that are scattered across the Internet. For example, you can use the `File.GetLastWriteUtc` method to implement a replication program that compares the files at two Internet sites in different time zones and overwrites the older one with the newer version.

The `Directory` type doesn't expose a `GetAttributes` method, but the `File.GetAttributes` method works also for directories, so this limitation isn't an issue. The `SetAttributes` and `GetAttributes` methods set or return a bit-coded `FileAttributes` value, which is a combination of `Normal` (= 128 no attributes), `Archive`, `ReadOnly`, `Hidden`, `System`, `Directory`, `Compressed`, `Encrypted`, `Temporary`, `NotContentIndexed`, and a few other values:

```FSharp
// Display system and hidden files in C:\.
for fname: String in Directory.GetFiles("C:\\") do
    let attr: FileAttributes = File.GetAttributes(fname)
    // Display the file if marked as hidden or system (or both).
    if attr &&& FileAttributes.Hidden = FileAttributes.Hidden || 
        attr &&& FileAttributes.System = FileAttributes.System then
        output.WriteLine(fname)
```

With a little bit of tweaking, you can make the If expression more concise, as follows:

```FSharp
if int (attr &&& (FileAttributes.Hidden ||| FileAttributes.System)) > 0 then
```

The `File` type exposes a few methods that are missing in the `Directory` type. The `Copy` method can copy a file and overwrite the destination if necessary:

```FSharp
// true in the last argument means "overwrite the target file" if it exists already.
File.Copy(@"c:\data.bin", @"c:\backup\data.bin", true)
```

**Version 2005 of VB or Version 2.0 of .NET** Three methods of the `File` type are new in .NET Framework 2.0. The `Replace` method performs a move+copy operation as a single command: it creates a backup copy of the destination file, and then copies the source file to the destination file. An optional fourth argument, if true, tells .NET to ignore any error that might occur when merging the attributes or the ACL of the two files involved in the command:

```FSharp
// Back up the current contents of c:\data.bin into c:\data.bak, and
// then copy the contents of c:\newdata.bin into c:\data.bin.
File.Replace(@"c:\data.bin", @"c:\newdata.bin", @"c:\data.bak", true)
```

The `Encrypt` method encrypts a file on an NTFS file system partition so that it can be read only by the current user; the process can be reversed by running the `Decrypt` method:

```FSharp
// Ensure that no other user account can read a file during a lengthy operation.
try
   File.Encrypt(@"c:\secretdata.txt")
   …

finally
   // Or just delete the file…
   File.Decrypt(@"c:\secretdata.txt")

```

### Reading and Writing Files

**Version 2005 of VB or Version 2.0 of .NET** In addition to the operations illustrated in the previous section, the `File` object can perform atomic read and write operations on text and binary files in a very simple manner. All the methods that enable you to perform these tasks have been added in .NET Framework 2.0.

You read an entire text file by means of the `ReadAllText` method, and write it using the `WriteAllText` method:

```FSharp
// Read a text file, convert its contents to uppercase, and save it to another file.
let text: String = File.ReadAllText(@"c:\testfile.txt")
File.WriteAllText(@"c:\upper.txt", text.ToUpper())
```

Alternatively, you can read and write an array of strings by means of the `ReadAllLines` and `WriteAllLines` methods:

```FSharp
// Read the source file into an array of strings.
let lines: String[] = File.ReadAllLines(@"c:\source.txt")
let mutable count: Int32 = 0
// Delete empty lines, by moving non-empty lines toward lower indices.
for i: Int32 in [0..lines.Length - 1] do
   if lines.[i].Trim().Length > 0 then
      lines.[count] <- lines.[i]
      count <- count + 1

// Trim excess lines and write to destination file.
let lines = lines.[0..count-1]
File.WriteAllLines(@"c:\dest.txt", lines)
```

The `AppendAllText` method appends a string to an existing text file or creates a text file if the file doesn't exist yet:

```FSharp
// Append a message to a log file, creating the file if necessary.
let msg: String = String.Format("Application started at {0}\n", DateTime.Now)
File.AppendAllText(@"c:\log.txt", msg)
```

The five methods shown so far have an overloaded version that accepts a `System.Text.Encoding` object. (See Chapter 12, ".NET Basic Types," for more details.) The `ReadAllBytes` and `WriteAllBytes` methods are similar, except they work with a Byte array and therefore are more useful with binary files:

```FSharp
// Very simple encryption of a binary file
let bytes: Byte[] = File.ReadAllBytes(@"c:\source.dat")
// Flip every other bit in each byte.
for i: Int32 in [0..bytes.Length - 1] do
    bytes.[i] <- bytes.[i] ^^^ byte(0x55)

// Write it to a different file.
File.WriteAllBytes(@"c:\dest.dat", bytes)
```

In addition to the read and write methods that process the entire file, the `File` type exposes methods that open the file for reading, writing, or appending data and return a `FileStream` object. The most flexible of these methods is the `Open` method, which takes a filename and up to three additional arguments:

```FSharp
let fs: FileStream = File.Open(FileName, FileMode, FileAccess, FileShare)
```

Let's see these arguments in more detail:

- The `FileMode` argument can be `Append`, `Create`, `CreateNew`, `Open`, `OpenOrCreate`, or `Truncate`. `Open` and `Append` modes fail if the file doesn't exist; `Create` and `CreateNew` fail if the file exists already. Use `OpenOrCreate` to open a file or to create one if it doesn't exist yet.
- The `FileAccess` argument specifies what the application wants to do with the file and can be `Read`, `Write`, or `ReadWrite`.
- The `FileShare` argument tells which operations other `FileStream`s can perform on the open file. It can be `None` (all operations are prohibited), `ReadWrite` (all operations are allowed), `Read`, `Write`, `Delete` (new in .NET Framework 2.0), or `Inheritable` (not supported directly by Win32).

The `File` class exposes three variants of the `Open` method: `Create`, `OpenRead`, and `OpenWrite`. Like the generic `Open` method, these variants return a `FileStream` object. There are also three specific methods for working with text files (`CreateText`, `OpenText`, and `AppendText`), which return a `StreamReader` or `StreamWriter` object. I explain how to use the `FileStream`, the `StreamReader`, and the `StreamWriter` objects later in this chapter.