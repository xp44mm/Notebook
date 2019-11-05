## The Stream Type

The `Stream` abstract type represents a sequence of bytes going to or coming from a storage medium (such as a file) or a physical or virtual device (such as a parallel port, an interprocess communication pipe, or a TCP/IP socket). Streams allow you to read from and write to a backing store, which can correspond to one of several storage mediums. For example, you can have file streams, memory streams, and network streams.

Because it's an abstract class, you don't create a `Stream` object directly, and you rarely use a `Stream` variable in your code. Rather, you typically work with types that inherit from it, such as the `FileStream` and the `NetworkStream` types.

### Stream Operations

The fundamental operations you can perform on streams are read, write, and seek. Not all types of streams support all these operations—for example, the `NetworkStream` object doesn't support seeking. You can check which operations are allowed by using the stream's `CanRead`, `CanWrite`, and `CanSeek` properties.

Most `Stream` objects perform data buffering in a transparent way. For example, data isn't immediately written to disk when you write to a file stream; instead, bytes are buffered and are eventually flushed when the stream is closed or when you issue an explicit `Flush` method. Buffering can improve performance remarkably. File streams are buffered, whereas memory streams aren't because there's no point in buffering a stream that maps to memory. You can use a `BufferedStream` object to add buffering capability to a stream object that doesn't offer it natively—for example, the `NetworkStream` object. Using `BufferedStream` in this fashion can improve performance remarkably if the application sends many small data packets rather than a few large ones.

Most of the properties of the `Stream` type—and of types that inherit from `Stream`—work as you would intuitively expect them to work:

- The `Length` property returns the total size of the stream, whereas the `Position` property determines the current position in the stream (that is, the offset of the next byte that will be read or written). You can change the stream's length using the `SetLength` method and change the position using the `Seek` method, but not all `Stream` types support these two methods.
- The `Read` method reads a number of bytes from the specified position into a Byte array, advances the stream pointer, and finally returns the number of bytes read. The `ReadByte` method reads and returns a single byte.
- The `Write` method writes a number of bytes from an array into the stream, and then advances the stream pointer. The `WriteByte` method writes a single byte to the stream.
- The `Close` method closes the stream and releases all the associated resources. The `Flush` method empties a buffered stream and ensures that all its contents are written to the underlying store. (It has no effect on nonbuffered streams.)
- In .NET Framework 2.0, you can use the `CanTimeout` read-only property to determine whether the stream supports timeouts in read and write operations; if this is the case, you can read or set these timeouts by means of the `ReadTimeout` and `WriteTimeout` properties. (These values are in milliseconds.)

Specific streams can implement additional methods and properties, such as the following:

- The `FileStream` class exposes the `Handle` property (which returns the operating system file handle) and the `Lock` and `Unlock` methods (which lock or unlock a portion of the file). When you're working with `FileStream` objects, the `SetLength` method actually trims or extends the underlying file.
- The `MemoryStream` class exposes the `Capacity` property (which returns the number of bytes allocated to the stream), the `WriteTo` method (which copies the entire contents to another stream), and the `GetBuffer` method (which returns the array of unsigned bytes from which the stream was created).
- The `NetworkStream` class exposes the `DataAvailable` property (which returns true when data is available on the stream for reading).

### Stream Readers and Writers

Because the generic `Stream` object can read and write only individual bytes or groups of bytes, most of the time you use auxiliary stream reader and stream writer objects that let you work with more structured data, such as a line of text or a Double value. The .NET Framework offers several stream reader and writer pairs:

- The `BinaryReader` and `BinaryWriter` types can work with primitive data in binary format, such as a Single value or an encoded string.
- The `StreamReader` and `StreamWriter` types can work with strings of text, such as the text you read from or write to a text file. These types can work in conjunction with an `Encoder` object, which determines how characters are encoded in the stream.
- The `StringReader` type can read from a string; the `StringWriter` class can write to a `StringBuilder`. (It can't write to a string because .NET strings are immutable.)
- `TextReader` and `TextWriter` are abstract types that define how to work with strings of text in Unicode format. `TextReader` is the base type for the `StreamReader` and `StringReader` types; `TextWriter` is the base type for the `StreamWriter` and `StringWriter` types.
- The `XmlTextReader` and `XmlTextWriter` types work with XML text.
- The `ResourceReader` and `ResourceWriter` types work with resource files.

### Reading and Writing Text Files

You typically use a `StreamReader` object to read from a text file. You can obtain a reference to such an object in many ways:

```FSharp
// With the File.OpenText static method
let fileName: String = @"c:\test.txt"
let sr: StreamReader = File.OpenText(fileName)

// With the OpenText instance method of a FileInfo object
let fi2 = new FileInfo(fileName)
let sr2: StreamReader = fi2.OpenText()

// By passing a FileStream from the Open method of the File class to
// the StreamReader's constructor method
// (This technique lets you specify mode, access, and share mode.)
let st3: FileStream = File.Open(fileName, FileMode.Open, FileAccess.ReadWrite, FileShare.ReadWrite)
let sr3 = new StreamReader(st3)

// By opening a FileStream on the file and then passing it
// to the StreamReader's constructor method
let fs4 = new FileStream(fileName, FileMode.Open)
let sr4 = new StreamReader(fs4)

// By getting a FileStream from the OpenRead method of the File class
// and passing it to the StreamReader's constructor
let sr5 = new StreamReader(File.OpenRead(fileName))

// By passing the filename to the StreamReader's constructor
let sr6 = new StreamReader(fileName)

// By passing the filename and encoding
let sr7 = new StreamReader(@"c:\autoexec.bat", System.Text.Encoding.Unicode)
let sr8 = new StreamReader(fileName, System.Text.Encoding.ASCII)

// As before, but we let the system decide the best encoding.
let sr9 = new StreamReader(fileName, true)
```

**Version 2005 of VB or Version 2.0 of .NET** The `FileStream` type in .NET Framework 2.0 has been expanded with new constructors to support two important new features. First, you can pass a `FileSecurity` object when you create a file, to specify the ACL associated with the file itself. Second, you can pass a `FileOptions` bit-coded value to specify additional options when opening a file. Supported values are `None`, `SequentialScan` (optimize caching for sequential access), `RandomAccess` (optimize caching for random access), `WriteThrough` (write data directly to disk, without buffering it), `Encrypted` (encrypt the file so that it can be read only by the same user account), `Asynchronous` (the file can be used for asynchronous reading and writing), and `DeleteOnClose` (the file is temporary and must be deleted when it's closed).

```FSharp
// Create a file for sequential reading and writing, with a 2-KB buffer;
// the file will be deleted when closed.
let fs10 = 
    new FileStream(@"c:\test.tmp", 
        FileMode.CreateNew, 
        FileAccess.ReadWrite, 
        FileShare.Read, 
        2048, 
        FileOptions.SequentialScan ||| FileOptions.DeleteOnClose)
```

After you get a reference to a `StreamReader` object, you can use one of its many methods to read one or more characters or whole text lines. The `Peek` method returns the code of the next character in the stream without actually extracting it, or it returns the special [-1]() value if there are no more characters. In practice, this method is used to test an end-of-file condition:

```FSharp
// Display all the text lines in the c:\test.txt file.
let sr = new StreamReader(@"c:\test.txt")
while sr.Peek() <> -1 do
   Console.WriteLine(sr.ReadLine())

sr.Close()
```

**Version 2005 of VB or Version 2.0 of .NET** In .NET Framework 2.0 you can rewrite the previous code in a more robust and readable style by means of the new `EndOfStream` read-only property and the `Using` statement:

```FSharp
use sr = new StreamReader(@"c:\test.txt")
while not sr.EndOfStream do
    Console.WriteLine(sr.ReadLine())
```

You can also read one character at a time using the `Read` method, or you can read all the remaining characters using the `ReadToEnd` method:

```FSharp
// Read the entire contents of C:\test.doc in one shot.
let sr = new StreamReader(@"c:\test.txt")
let fileContents: String = sr.ReadToEnd()
```

If you opened the `StreamReader` through a `Stream` object, you can use the `Stream` object's `Seek` method to move the pointer or read its current position. If you did not open the `StreamReader` through a `Stream` object, you can still access the inner `Stream` object that the .NET runtime creates anyway, through the `StreamReader`'s `BaseStream` property:

```FSharp
// …(Continuing previous code example)…
// If the file is longer than 100 chars, process it again, one character at a
// time (admittedly a silly thing to do, but it's just a demo).
if fileContents.Length >= 100 then
   // Reset the stream's pointer to the beginning.
   sr.BaseStream.Seek(0L, SeekOrigin.Begin) |> ignore
   // Read individual characters until EOF is reached.
   while not sr.EndOfStream do
      // Read method returns an integer, so convert it to Char.
      Console.Write(sr.Read().ToString())

sr.Close()
```

You use a `StreamWriter` object to write to a text file. As with the `StreamReader` object, you can create a `StreamWriter` object in many ways:

```FSharp
// By means of the CreateText static method of the File type
let fileName: String = @"c:\text.dat"
let sw1: StreamWriter = File.CreateText(fileName)

// By passing a FileStream from the Open method of the File class to
// the StreamWriter's constructor method
 let st2: FileStream = File.Open(fileName,  FileMode.Create, FileAccess.ReadWrite, FileShare.None)
let sw2 = new StreamWriter(st2)

// By opening a FileStream on the file and then passing it
// to the StreamWriter's constructor
let fs3 = new FileStream(fileName, FileMode.Open)
let sw3 = new StreamWriter(fs3)

// By getting a FileStream from the OpenWrite method of the File type
// and passing it to the StreamWriter's constructor
let sw4 = new StreamWriter(File.OpenWrite(fileName))

// By passing the filename to the StreamWriter's constructor
let sw5 = new StreamWriter(fileName)
```

Other overloads of the `StreamWriter`'s constructor allow you to specify whether the file is to be opened in append mode, the size of the buffer, and an `Encoding` object:

```FSharp
// Open the c:\test.dat file in append mode, be prepared to output
// ASCII characters, and use a 2-KB buffer.
let sw6 = new StreamWriter(@"c:\test.new", true, Encoding.ASCII, 2024)
```

The `NewLine` property (new in .NET Framework 2.0) lets you specify a nonstandard value for the line termination character:

```FSharp
// Terminate each line with a null character followed by a newline character.
sw6.NewLine <- "\r\n"
```

The `StreamWriter` class exposes the `Write` and `WriteLine` methods: the `Write` method can write the textual representation of any basic data type (Integer, Double, and so on); the `WriteLine` method works only with strings and automatically appends a newline character. Leave the `AutoFlush` property set to False (the default value) if you want the `StreamWriter` to adopt a limited form of caching; you'll probably need to issue a `Flush` method periodically in this case. Set this property to True for those streams or devices, such as the console window, from which the user expects immediate feedback.

The following code uses a `StreamReader` object to read from a file and a `StreamWriter` object to copy the text to another file after converting the text to uppercase:

```FSharp
use sr = new StreamReader(@"c:\test.txt")
use sw = new StreamWriter(@"c:\test.new")
while not sr.EndOfStream do
    sw.WriteLine(sr.ReadLine().ToUpper())
// This actually writes data to the file and closes it.
```

If you're working with smaller text files, you can also trade some memory for speed and do without a loop. The following code uses a single `Using` block, but the Visual Basic compiler correctly expands it into two nested blocks, as in the previous code snippet:

```FSharp
use sr = new StreamReader(@"c:\test.txt")
use sw = new StreamWriter(@"c:\test.new")
sw.WriteLine(sr.ReadToEnd().ToUpper())
```

You should always close the `Stream` object after using it, either by means of a `Using` block or explicitly with a `Close` method. If you fail to do so, the stream keeps the file open until the next garbage collection calls the `Stream`'s `Finalize` method. There are at least two reasons why you'd rather close the stream manually. First, if the file is kept open longer than strictly necessary, you can't delete or move the underlying file, nor can another application open it for reading and/or writing (depending on the access mode you specified when opening the file). The second reason is performance: the code in the `Stream`'s `Close` method calls the `GC.SuppressFinalize` method, so the `Stream` object isn't finalized and therefore the resources it uses are released earlier.

### Reading and Writing Binary Files

The `BinaryReader` and `BinaryWriter` types are suitable for working with binary streams; one such stream might be associated with a file containing data in native format. In this context, *native format* means the actual bits used to store the value in memory. You can't create a `BinaryReader` or `BinaryWriter` object directly from a filename as you can with the `StreamReader` and `StreamWriter` objects. Instead, you must create a `Stream` object explicitly and pass it to the constructor method of either the `BinaryReader` or the `BinaryWriter` class:

```FSharp
// Associate a stream with a new file opened with write access.
let st: FileStream = File.Open(@"c:\values.dat", FileMode.Create, FileAccess.Write)
// Create a BinaryWriter associated with the output stream.
let bw = new BinaryWriter(st)
```

Working with the `BinaryWriter` object is especially simple because its `Write` method is overloaded to accept all the primitive .NET types, including signed and unsigned integers, Single, Double, and String values. The following code snippet writes 10 random Double values to a binary file:

```FSharp
// …(Continuing previous example)…
// Save 10 Double values to the file.
let rand = new Random()
for i = 1 to 10 do
   bw.Write(rand.NextDouble())

// Flush the output data to the file.
bw.Close()
```

The `BinaryReader` class exposes many `ReadXxxx` methods, one for each possible native data type. Unlike the `StreamReader` type, which exposes an `EndOfStream` property, the `BinaryReader` type requires that you use the `PeekChar` method to check whether other bytes are available:

```FSharp
// Associate a stream with an existing file, opened with read access.
let st2: FileStream = File.Open(@"c:\values.dat", FileMode.Open, FileAccess.Read)
// Create a BinaryReader associated with the input stream.
use br2 = new BinaryReader(st2)
// Loop until data is available.
while not(br2.PeekChar() = -1) do
    // Read the next element. (We know it's a Double.)
    Console.WriteLine(br2.ReadDouble())

// Next statement closes both the BinaryReader and the underlying stream.
```

Outputting strings with a `BinaryWriter` requires some additional care, however. Passing a string to the `Write` method outputs a length-prefixed string to the stream. If you want to write only the actual characters (as happens when you're working with fixed-length strings), you must pass the `Write` method a `Char` array. The `Write` method is overloaded to take additional arguments that specify which portion of the array should be written.

Reading back strings requires different techniques as well, depending on how the string was written. You use the `ReadString` method for length-prefixed strings and the `ReadChars` method for fixed-length strings. You can see an example of these methods in action in the section "Memory Streams" later in this chapter.

---

##### Note

File streams can be opened for asynchronous read and write operations, which can speed up your code's performance significantly. You'll learn about asynchronous file operations in Chapter 20.

---

### Working with Fixed-Length and Delimited Data Files

**Version 2005 of VB or Version 2.0 of .NET** In Chapter 14 you saw how you can use regular expressions to read text files that use either delimited or fixed-length fields. The new `TextFieldParser` type (in the `Microsoft.VisualBasic.FileIO` namespace) offers a simpler way to accomplish the same task. As the name of its namespace suggests, this type is part of the `Microsoft.VisualBasic` library and doesn't "officially" belong to the .NET Framework. (C# developers can of course use this class by adding a reference to the `Microsoft.VisualBasic.dll` assembly.)

Using the `TextFieldParser` to read text files with delimited fields is a trivial procedure. You open the data file by passing the filename to the `TextFieldParser` constructor, together with an `Encoding` object, and then you assign an array of delimiters to the `Delimiters` property and optionally set the `TrimWhiteSpace` property to False if you don't want to discard leading and trailing spaces (the default value for this property is True):

```FSharp
let parser = new TextFieldParser(@"c:\data.txt", Encoding.Default)
// Field separator can be either a comma or a semicolon.
parser.Delimiters <- [|","; ";"|]
parser.TrimWhiteSpace <- true
```

(Other overloads of the constructor take a `Stream` or a `TextReader` object.) Next, you need a loop that processes the file one line at a time until the `EndOfData` property returns True; the `ReadFields` method reads the next record (that is, the next line of text) and splits it into fields:

```FSharp
while not parser.EndOfData do
   let fields: String[] = parser.ReadFields()
   // Process each field in current record here.
   …
parser.Close()
```

Conveniently, the `TextFieldParser` type correctly interprets quoted strings (even if they embed one of the delimiter characters). For example, consider the following data file:

```csv
"John P.", Evans, New York
Robert, Zare, "Los Angeles, CA"
```

Here's the Visual Basic code that can process it:

```FSharp
use parser = new TextFieldParser("data.txt", Encoding.Default)
parser.Delimiters <- [|","|]
parser.TrimWhiteSpace <- true

while not parser.EndOfData do
    let fields: String[] = parser.ReadFields()
    Console.WriteLine("First={0}, Last={1}, Location={2}", fields.[0], fields.[1], fields.[2])
```

The result in the console window proves that both quotation marks and surrounding spaces have been correctly removed, and that the comma inside the quoted city name hasn't been mistakenly taken as a field separator:

```FSharp
First=John P., Last=Evans, City=new York
First=Robert, Last=Zare, City=Los Angeles, CA
```

Reading text files containing fixed-length fields with the `TextFieldParser` is equally simple, the only differences being that you must set the `TextFieldType` property to the `FieldType.FixedWidth` enumerated value and assign an array of Integers to the `FieldWidths` property; each element of this array is interpreted as the width of the corresponding field. For example, let's say that we have the following data file:

```FSharp
John P. Evans     Dallas
Robert  Zare      Boston
```

Here's the code fragment that can read it:

```FSharp
use parser = new TextFieldParser("data2.txt", Encoding.Default)
parser.TextFieldType <- FieldType.FixedWidth
parser.FieldWidths <- [| 8; 10; 6|]
while not parser.EndOfData do
    let fields: String[] = parser.ReadFields()
    Console.WriteLine("First={0}, Last={1}, Location={2}", fields.[0], fields.[1], fields.[2])
```

Keep in mind that quotation marks *aren't* automatically stripped off when reading fixed-width text files.

The `PeekChars` method returns a number of characters without actually reading them from the stream. This method is especially useful with data files in which the first field contains a special code that affects the format of the current line. For example, consider the following data file that contains a mix of invoice headers and invoice details:

```FSharp
IH 1   12/04/2005 John Evans
ID 12    Monitor XY        129.99
ID 4     Printer YZ        212.00
ID 3     Hard disks Z4     159.00
IH 2   12/06/2005 Robert Zare
ID 8     Monitor XY        129.99
ID 2     Notebook ABC      850.00
```

The first three characters in each line specify whether the line is an invoice header (IH) or invoice detail (ID); in the former case, the line contains the invoice number, date, and customer; in the latter case, the line contains the quantity, description, and unit price. The format of the two records is different; in this specific case they have the same number of fields, but this is just a coincidence because the number of fields might differ as well. Here's the Visual Basic code that can interpret this data file:

```FSharp
use parser = new TextFieldParser("data3.txt", Encoding.Default)
parser.TextFieldType <- FieldType.FixedWidth
let headerWidths: Int32[] = [|3; 4; 11; 12|]
let detailWidths: Int32[] = [|3; 6; 18; 6|]
parser.FieldWidths <- headerWidths

while not parser.EndOfData do
    let code: String = parser.PeekChars(2)
    if code = "IH" then
        parser.FieldWidths <- headerWidths
        let fields: String[] = parser.ReadFields()
        Console.WriteLine("Invoice #{0}, DateTime={1}, Customer={2}", fields.[0], fields.[1], fields.[2])
    elif code = "ID" then
        parser.FieldWidths <- detailWidths
        let fields: String[] = parser.ReadFields()
        Console.WriteLine(" #{0} {1} at ${2} each", fields.[0], fields.[1], fields.[2])
    else
        raise <| new MalformedLineException("Invalid record code")
```

The `MalformedLineException` type is defined in the `Microsoft.VisualBasic.FileIO` namespace; the `TextFieldParser` object throws this exception when the format of the current record doesn't match what it expects to find. When your code catches this exception, you can attempt to solve the problem by querying its `ErrorLine` property (the text line that caused the problem) or the `ErrorLineNumber` (the number of the malformed line). The line number is returned also by the `LineNumber` property of the `MalformedLineException` type. If you aren't sure about the integrity of the file being parsed, you should write code like this:

```FSharp
while not parser.EndOfData do
   try
      // Process fields here.
      …
   with :? MalformedLineException as ex -> 
      Console.WriteLine("Line #{0} is malformed. Ignored.", ex.LineNumber)
```

Before we move to a different topic, notice that .NET Framework 2.0 doesn't offer a type that writes data files in either delimited or fixed-width format. The reason is evident: writing such files is too easy, thanks to the `String.Format` method and its many options, and few developers would resort to a separate class for such a simple task. For example, the following statement can output a line of text consisting of three left-aligned, fixed-length fields:

```FSharp
// sw is a StreamWriter object, p is a Person object that exposes
// the FirstName, LastName, and City properties.
sw.WriteLine(String.Format("{0,-8}{1,-10}{2,-16}", p.FirstName, p.LastName, p.City))
```

Creating a delimited file is even simpler because you just need to put double quotation marks around fields that might contain the delimiter character:

```FSharp
// Output data to a semicolon-delimited file.
sw.WriteLine(String.Format(@"""{0}"";""{1}"";""{2}""", p.FirstName, p.LastName, p.City))
```