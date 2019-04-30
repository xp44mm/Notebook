## Other Stream Types

Stream readers and writers aren't just for files, even though files are undoubtedly the most common kind of stream. In this last section, I cover other common types that derive from the `Stream` class.

### Memory Streams

The `MemoryStream` object allows you to deal with memory as if it were a temporary file, a technique that usually delivers better performance than does using an actual file. The following code snippet shows how to write to and then read back 10 random numbers from a stream; this example is similar to a code snippet illustrated earlier in this chapter, except this code uses a memory stream instead of a file stream:

```FSharp
// Create a memory stream with initial capacity of 1 KB.
let st = new MemoryStream(1024)
let bw = new BinaryWriter(st)
let rand = new Random()
// Write 10 random Double values to the stream.
let lst =
    [| for i = 1 to 10 do yield rand.NextDouble() |]
lst |> Array.iter(fun f -> bw.Write(f))

// Rewind the stream to the beginning and read back the data.
st.Seek(0L, SeekOrigin.Begin) |> ignore
let br = new BinaryReader(st)
let bs = br.BaseStream
Assert.Same(bs,st)
let res = 
    [|
        while bs.Position < bs.Length do // br.PeekChar() > -1 error
            yield br.ReadDouble()
    |]

bw.Close()
br.Close()
st.Close()
Assert.Equal<float[]>(lst,res)
```

Of course, in this particular example you might have used an array to store random values and read them back. However, the approach based on streams lets you move from a memory stream to a file-based stream by changing only one statement (the stream constructor).

---

##### PeekChar

In general, this arises when the incorrect encoding is set on the `BinaryReader` (the default one is UTF-8 which will be used in your code) from what is encoded in the stream. `PeekChar()` and `ReadChar()` methods in `BinaryReader` will attempt to decode the bytes in the stream to fit in with the specified encoding and can fault if they are not compatible. These methods also fail when trying to read a surrogate character.

---

This example writes three strings to a `MemoryStream` and then reads them back; it shows how to work with length-prefixed strings and two techniques for reading fixed-length strings:

```FSharp
let st = new MemoryStream(1000)
let bw = new BinaryWriter(st)
// The BinaryWriter.Write method outputs a length-prefixed string.
let s1 = "a length-prefixed string"
bw.Write(s1)

// We'll use this 1-KB buffer for both reading and writing.
let buffer = Array.zeroCreate<Char>(1023)

let s2: String = "13 Characters"           // A fixed-length string
s2.CopyTo(0, buffer, 0, s2.Length)         // Copy into the buffer.
bw.Write(buffer, 0, s2.Length)             // Output first 13 chars in buffer.
bw.Write(buffer, 0, s2.Length)             // Do it a second time.

// Rewind the stream, and prepare to read from it.
st.Seek(0L, SeekOrigin.Begin) |> ignore
let br = new BinaryReader(st)

// Reading the length-prefixed string is simple.
Assert.Equal(br.ReadString(),s1)

// Read the fixed-length string (13 characters) into the buffer.
br.Read(buffer, 0, 13) |> ignore
let ss = new String(buffer, 0, 13)          // Convert to a string.
Assert.Equal(ss,s2)

// Another way to read a fixed-length string (13 characters)
// (ReadChars returns a Char array that we can pass to the string constructor.)
let ss = new String(br.ReadChars(13))
Assert.Equal(ss,s2)
```

### String-Based Streams

If the data you want to read is already contained in a string variable, you can use a `StringReader` object to retrieve portions of it. For example, you can load the entire contents of a text file or a multiline text box control into a string and then extract the individual lines by using the `StringReader.ReadLine` method:

```FSharp
// The veryLongString variable contains the text to parse.
let strReader = new StringReader(veryLongString)
// Display individual lines of text.
while strReader.Peek() > -1 do
   Console.WriteLine(strReader.ReadLine())
```

Of course, you can implement the same technique in other, equivalent ways—for example, by using the `Split` function to get an array with all the individual lines of code—but the solution based on the `StringReader` object is more resource-friendly because it doesn't duplicate the data in memory. As a matter of fact, the `StringReader` and `StringWriter` types don't even create an internal `Stream` object to store the characters; rather they use the string itself as the backing store for the stream. (This fact explains why these two types don't expose the `BaseStream` property.)

You use a `StringWriter` object to output values to a string. However, you can't associate it with a `String` object because .NET strings are immutable. Instead, you have to create a `StringBuilder` and then associate it with a `StringWriter` object:

```FSharp
// Create a string with the space-separated abbreviated names of weekdays.
let sb = new StringBuilder()
// The StringWriter associated with the StringBuilder
let strWriter = new StringWriter(sb)

// Output day names to the string.
for d: String in DateTimeFormatInfo.CurrentInfo.AbbreviatedDayNames do
   strWriter.Write(d)
   strWriter.Write(" ")        // Append a space.

Console.WriteLine(sb.ToString())           // => Sun Mon Tue Wed Thu Fri Sat
```

### Network Streams

The .NET Framework supports exchanging data over the network using Transmission Control Protocol (TCP) by means of `TcpClient` and `TcpListener` types, both in the `System.Net.Sockets` namespace. TCP ensures that data is either correctly received or an exception is thrown, and for this reason it is the preferred protocol when you can't afford to lose data. The applications involved in data exchange can reside on the same computer or on different computers over the LAN or connected through the Internet. The actual data sent over the wire is read from and written to a `NetworkStream` object, also in the `System.Net.Sockets` namespace.

TCP assumes you have a server application that listens to requests coming from one or more clients. The client and the server applications must agree on the port number as well as the format of the data being sent over the wire. In the following example, the client application sends the name of a text file as a string terminated with a CR-LF character. The server application receives the filename sent from the client, searches the file in a specific directory, and sends the file's contents back to the client. Because the file contents can be of any length, the server application prefixes the actual data with a line containing the length of the data so that the client can read the exact number of bytes from `NetworkStream`. (In a real-world application, using a length prefix is important because a single server-side send operation can translate into multiple receive operations on the client side.) Both applications terminate when the client sends an empty filename to the server.

The server application must create a `TcpListener` object that listens to a given port and accepts incoming requests from clients. The `AcceptTcpClient` method waits until a connection is made and returns the `TcpClient` object that represents the client making the request. The `GetStream` method of this `TcpClient` object returns a `NetworkStream` that the server application can use to read data from clients and send them a result.

```FSharp
// Send a length-prefixed string.
let SendData(ns : NetworkStream, data : String) =
    // Send it back to the client.
    let sw = new StreamWriter(ns)
    sw.WriteLine(data.Length)
    sw.Write(data)
    sw.Flush()                            // This is VERY important.
    sw.Close()

// The server application
let server() =
    // Listen to port 2048.

    let localhostAddress: IPAddress = IPAddress.Loopback
    let tcpList = new TcpListener(localhostAddress, 2048)
    tcpList.Start()

    let mutable breakLoop = false

    while not breakLoop do
        // Wait for the next client to make a request.
        Console.WriteLine("Waiting for data from clients...")
        let tcpCli: TcpClient = tcpList.AcceptTcpClient()
        // Read data sent by the client (a CR-LF-separated string in this case).
        let ns: NetworkStream = tcpCli.GetStream()
        let sr = new StreamReader(ns)
        let receivedData: String = sr.ReadLine()

        if receivedData <> "" then
            // Read a file with this name from the C:\docs directory.
            let fileName: String = Path.Combine(@"c:\docs", receivedData)
            Console.WriteLine("Reading file {0}...", fileName)

            let resultData: String = 
                try
                    File.ReadAllText(fileName)
                with ex -> 
                    "*** ERROR: " + ex.Message
         
            SendData(ns, resultData)
      
        // Release resources and close the NetworkStream.
        sr.Close()
        ns.Close()
        tcpCli.Close()
        // Exit if the client sent an empty string.
        breakLoop <- receivedData = ""
   
    // Reject client requests from now on.
    tcpList.Stop()
```

The client code must instantiate a `TcpClient` object that references the server application by means of the server application's URL and port number. (The port number should be in the range of 1024 to 65535.) Next, the client invokes the `TcpClient.GetStream` method to retrieve the `NetworkStream` object that can be used to send and receive data from the server.

```FSharp
// Read a length-prefixed string.
let ReadData(ns : NetworkStream) : String =
   let sr = new StreamReader(ns)
   let dataLength: Int32 = int(sr.ReadLine())
   let buffer = Array.zeroCreate<Char>(dataLength - 1)
   sr.Read(buffer, 0, dataLength) |> ignore
   sr.Close()
   new String(buffer)

// The client application
let client() =
    let mutable breakLoop = false

    while not breakLoop do
        // Ask the end user for a filename.
        Console.Write("Enter a file name [an empty string to quit] >> ")
        let fileName: String = Console.ReadLine()

        // This code assumes a server on the local machine is listening to port 2048.
        let tcpCli = new TcpClient("localhost", 2048)
        // Retrieve the stream that can send and receive data.
        let ns: NetworkStream = tcpCli.GetStream()
        // Send a CR-LF-terminated string to the server.
        let sw = new StreamWriter(ns)
        sw.WriteLine(fileName)
        sw.Flush()                          // This is VERY important!

        if fileName <> "" then
            // Receive data from the server application and display it.
            let resultData: String = ReadData(ns)
            Console.WriteLine(resultData)
      
        // Release resources and close the NetworkStream.
        sw.Close()
        ns.Close()
        breakLoop <- fileName = ""
```

To test this code, create a directory named c:\docs and store a few text files in it. Compile the server and the client applications as separate `Console` projects and run both of them. Enter the name of a file in the client application and wait for the server to send back the textual content of the file.

Alternatively, you can create a solution that contains both these projects; then, right-click the solution item in the Solution Explorer window and select the Properties command. Select the Startup Project page in the dialog box that appears, click the Multiple Startup Projects radio button, and set the Action value to Start for both the server-side and client-side projects. (See Figure 15-3.) If you now press F5 or select the Start command from the Debug menu, both projects will be launched.

![Image from book](images/fig649%5F01%5F0%`2Ejpg`) 
Figure 15-3: The solution's Property Pages dialog box, which lets you decide which project to run when you press the F5 key  

The `NetworkStream` type exposes a few other members of interest. For example the `DataAvailable` property returns True if there is data waiting to be read. Network streams don't support the seek operation; thus, the `CanSeek` property always returns False and the `Seek` method throws an exception, as do the `Length` and `Position` properties.





**Version 2005 of VB or Version 2.0 of .NET** The most important feature added in .NET Framework 2.0 is the support for timeouts through the `ReadTimeout` and `WriteTimeout` properties. By default, all read and write operations wait until data can be received or sent, but if you assign a value (in milliseconds) to these properties, any operation that doesn't complete within the specified timeout throws an `IOException` object.

---

##### Note

As with all the applications that exchange data using TCP, these sample applications (more precisely, the server application) might open a security hole on your computer because in theory a remote client might use the application to read the contents of a file on your computer. It is therefore essential that you protect your system by using other methods (for example, a firewall) and that you terminate the server application as soon as you're done with your experiments.

---

### Buffered Streams

Most of the stream types in the .NET Framework provide a transparent mechanism for buffering, for example, the `FileStream` type, whereas other types never require buffering because they already map to a block of memory, as is the case with the `MemoryStream`, the `StringReader`, and `StringWriter` types. The remaining stream types, for example, the `NetworkStream` type, don't use any internal cache mechanism; thus, writing to and reading small pieces of data from them can be extremely inefficient. In such cases, you can improve performance by using an auxiliary `BufferedStream` object.

Using a `BufferedStream` object is quite simple and amounts to using this object to "wrap" another (unbuffered) stream. For example, let's say that you have the following piece of code:

```FSharp
// Initialize the array here (omitted).
let arr =[|""|]

// Send the array of strings to a TCP server application.
let tcpCli = new TcpClient("localhost", 2048)
let ns: NetworkStream = tcpCli.GetStream()
let sw = new StreamWriter(ns) 

for s in arr do
   sw.WriteLine(s)
```

You can make this code faster using a `BufferedStream` object by replacing the statement in bold type with these two lines:

```FSharp
let bufStream = new BufferedStream(ns, 8192)
let sw = new StreamWriter(bufStream)
```

The second argument passed to the `BufferedStream`'s constructor is the size of the buffer; if omitted a default size of 4,096 bytes is used.

The great thing about the `BufferedStream` type is that it manages its internal buffer in a very smart way. If you read or write a piece of data larger than the buffer's size, the buffer isn't even used for that specific read or write operation; if you only read and write large pieces of data, the buffer isn't even allocated. You get the best performance with the `BufferedStream` if you perform a series of read or write operations, but don't alternate often between reads and writes.

### Compressed Streams

**Version 2005 of VB or Version 2.0 of .NET** Two new types for compressing data have been added in version 2.0 of the .NET Framework: `DeflateStream` and `GZipStream`, both in the `System.IO.Compression` namespace. Both types enable you to compress and uncompress the bytes that flow through the stream; they expose a very similar interface and are virtually interchangeable; the only substantial difference is the format of their compressed output.

The `DeflateStream` type uses the `Deflate` compression algorithm, a patent-free algorithm that combines the LZ77 algorithm and Huffman coding; the main advantage of this algorithm is that data of any length can be compressed and uncompressed using an intermediate buffer of limited size. The `GZipStream` type uses the same `Deflate` algorithm, but it includes a cyclic redundancy check (CRC) value to detect data corruption.

The peculiarity of these two types is that they can work only together with another stream-based object. In fact, a `DeflatedStream` or `GZipStream` object "wraps" another stream that actually writes to the actual medium (if you are compressing data) or reads from the medium (if you are uncompressing data). Given the similarities between the two types, I show how to use just the `DeflateStream` type.

Compressing data is the easiest operation if you already have the data in a `Byte` array. For example, the following code uses the `File.ReadAllBytes` method to read the entire source file (that is, the uncompressed file) and compresses it into a new file:

```FSharp
let uncompressedFile: String = "test.txt"
let compressedFile: String = "test.zip"
// Read the source (uncompressed) file in the buffer.

let buffer: Byte[] = File.ReadAllBytes(uncompressedFile)
// Open the destination (compressed) file with a FileStream object.
let outStream = new FileStream(compressedFile, FileMode.Create)
// Wrap a DeflateStream object around the output stream.
let zipStream = new DeflateStream(outStream, CompressionMode.Compress)
// Write the contents of the buffer.
zipStream.Write(buffer, 0, buffer.Length)
// Flush compressed data and close all output streams.
zipStream.Flush()
zipStream.Close()
outStream.Close()
```

Things are slightly more complicated if the source file is too long to be read in memory and you must process it in chunks. The following reusable procedure adopts a more resource-savvy approach:

```FSharp
let CompressFile(uncompressedFile : String, compressedFile : String) =
    // Open the source (uncompressed) file, using a 4-KB input buffer.
    use inStream = new FileStream(uncompressedFile, FileMode.Open, FileAccess.Read, FileShare.None, 4096)
    // Open the destination (compressed) file with a FileStream object.
    use outStream = new FileStream(compressedFile, FileMode.Create)
    // Wrap a DeflateStream object around the output stream.
    use zipStream = new DeflateStream(outStream, CompressionMode.Compress)
    // Prepare a 4-KB read buffer .
    let buffer =Array.zeroCreate<Byte>(4095)

    let mutable terminal = false
    while not terminal do
        // Read up to 4 KB from the input file; exit if no more bytes.
        let readBytes: Int32 = inStream.Read(buffer, 0, buffer.Length)
        terminal <- readBytes = 0
        // Write the contents of the buffer to the compressed stream.
        zipStream.Write(buffer, 0, readBytes)
            
    // Flush and close all streams.
    zipStream.Flush()
    // Close the DeflateStream object.
    // Close the output FileStream object.
    // Close the input FileStream object.
```

When uncompressing a compressed file, you have no choice: you must process the incoming data in chunks. Here's the reason: when you read *N* compressed bytes, you don't know how many data bytes the decompress process will create. Here's a reusable method that implements all the necessary steps:

```FSharp
let UncompressFile(compressedFile : String, uncompressedFile : String) =
    // Open the output (uncompressed) file, use a 4-KB output buffer.
    use outStream = new FileStream(uncompressedFile, FileMode.Create,  FileAccess.Write, FileShare.None, 4096)
    // Open the source (compressed) file.
    use inStream = new FileStream(compressedFile, FileMode.Open)
    // Wrap the DeflateStream object around the input stream.
    use zipStream = new DeflateStream(inStream, CompressionMode.Decompress)
    // Prepare a 4-KB buffer.
    let buffer =Array.zeroCreate<Byte>(4095)

    let mutable terminal = false
    while not terminal do
        // Read enough compressed bytes to fill the 4-KB buffer.
        let bytesRead: Int32 = zipStream.Read(buffer, 0, 4096)
        // Exit if no more bytes were read.
        terminal <- bytesRead = 0
        // Else, write these bytes to the uncompressed file and loop.
        outStream.Write(buffer, 0, bytesRead)
            
    // Ensure that cached bytes are written correctly and close all streams.
    outStream.Flush()
    // Close the DeflateStream object.
    // Close the input FileStream object.
    // Close the output FileStream object.
```

Let's recap the rules for using the `DeflateStream` type correctly. When compressing data, you pass the output stream to the `DeflateStream`'s constructor and specify `CompressionMode.Compress` in the second argument, and then you write the (uncompressed) bytes with the `DeflateStream.Write` method. Conversely, when uncompressing data, you pass the input stream to the `DeflateStream`'s constructor and specify `CompressionMode.Decompress` in the second argument, and then you read (compressed) bytes with the `DeflateStream.Read` method.

Finally, remember that you can use the `DeflateStream` and `GZipStream` types in a chain of streams; for example, you can output data to a `BufferedStream` object, which cascades to a `DeflateStream` object, which uses a `NetworkStream` object to send the compressed data across the wire. You can see a use for compressed streams in the section titled "A Practical Example: Compressed Serialization" in Chapter 21, "Serialization."