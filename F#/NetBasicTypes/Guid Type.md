### The Guid Type

The `System.Guid` type exposes several static and instance methods that can help you work with globally unique identifiers (GUIDs), that is, those 128-bit numbers that serve to uniquely identify elements and that are ubiquitous in Windows programming. The `NewGuid` static method is useful for generating a new unique identifier:

```FSharp
// Create a new GUID.
let guid1: Guid = Guid.NewGuid()
```

If you already have a GUID—for example, a GUID you have read from a database field—you can initialize a `Guid` variable by passing the GUID representation as a string or as an array of bytes to the type's constructor:

```FSharp
// Initialize from a string.
let guid2 = new Guid("f16f990e-dd16-43af-a5ea-6340c47b32b0")
```

There are only two more things you can do with a `Guid` object: you can convert it to a Byte array with the `ToByteArray` method, and you can compare two `Guid` values for equality using the `Equals` method (inherited from `System.Object`):

```FSharp
// Convert to an array of bytes.
let bytes: byte[] = guid1.ToByteArray()
for b: byte in bytes do
   Console.Write("{0} ", b)

// Compare two GUIDs.
if guid1.Equals(guid2) then
   Console.WriteLine("GUIDs are same.")
else
   Console.WriteLine("GUIDs are different.")
```

