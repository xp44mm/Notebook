## Introducing the Imperative .NET Collections

The .NET framework comes equipped with an excellent set of imperative collections under the namespace `System.Collections.Generic`. You've seen some of these already. The following sections will look at some simple uses of these collections.

### Using Resizable Arrays

As mentioned in Chapter 3, the .NET framework comes with a type `System.Collections.Generic. List<'T>`, which, although named `List`, is better described as a resizable array, The F# library includes the following type abbreviation for this purpose:

```fs
type ResizeArray<'T> = System.Collections.Generic.List<'T>
```

Here is a simple example of using this data structure:

```fs
> let names = new ResizeArray<string>();;
val names : ResizeArray<string>
> for name in ["Claire"; "Sophie"; "Jane"] do
      names.Add(name);;
> names.Count;;
val it : int = 3
> names.[0];;
val it : string = "Claire"
> names.[1];;
val it : string = "Sophie"
> names.[2];;
val it : string = "Jane"
```

Resizable arrays use an underlying array for storage, and they support constant-time random-access lookup. In many situations, this makes a resizable array more efficient than an F# list, which supports efficient access only from the head (left) of the list. You can find the full set of members supported by this type in the .NET documentation. Commonly used properties and members include `Add`, `Count`, `ConvertAll`, `Insert`, `BinarySearch`, and `ToArray`. A module `ResizeArray` is included in the F# library; it provides operations over this type in the style of the other F# collections.

Like other .NET collections, values of type `ResizeArray<'T>` support the `seq<'T>` interface. There is also an overload of the new constructor for this collection type that lets you specify initial values via a `seq<'T>`. This means you can create and consume instances of this collection type using sequence expressions, as follows:

```fs
> let squares = new ResizeArray<int>(seq {for i in 0 .. 100 -> i * i});;
val squares : ResizeArray<int>
> for x in squares do
      printfn "square: %d" x;;
square: 0
square: 1
square: 4
square: 9
...
square: 9801
square: 10000
```

### Using Dictionaries

The type `System.Collections.Generic.Dictionary<'Key,'Value>` is an efficient hash-table structure that is excellent for storing associations between keys and values. Using this collection from F# code requires a little care, because it must be able to correctly hash the key type. For simple key types such as integers, strings, and tuples, the default hashing behavior is adequate. Here is a simple example:

```fs
> open System.Collections.Generic;;
> let capitals = new Dictionary<string, string>(HashIdentity.Structural);;
val capitals : Dictionary<string,string> = dict []
> capitals.["USA"] <- "Washington";;
> capitals.["Bangladesh"] <- "Dhaka";;
> capitals.ContainsKey("USA");;
val it : bool = true
> capitals.ContainsKey("Australia");;
val it : bool = false
> capitals.Keys;;
val it : Dictionary'2.KeyCollection<string,string> = seq ["USA"; "Bangladesh"]
> capitals.["USA"];;
val it : string = "Washington"
```

读书笔记：`Dictionary<'Key,'Value>`类型Key为字符串时可以不区分大小写。

```fs
let dic = new Dictionary<string, any>(StringComparer.OrdinalIgnoreCase);

dic.Keys.Contains("Key", StringComparer.OrdinalIgnoreCase);
```

Dictionaries are compatible with the type `seq<KeyValuePair<'key,'value>>`, where `KeyValuePair` is a type from the `System.Collections.Generic` namespace and simply supports the properties `Key` and `Value`. Armed with this knowledge, you can use iteration to perform an operation for each element of the collection, as follows:

```fs
> for kvp in capitals do
      printfn "%s has capital %s" kvp.Key kvp.Value;;
USA has capital Washington
Bangladesh has capital Dhaka
```

### Using Dictionary's TryGetValue

The `Dictionary` method `TryGetValue` is of special interest, because its use in F# is a little nonstandard. This method takes an input value of type `'Key` and looks it up in the table. It returns a bool indicating whether the lookup succeeded: true if the given key is in the dictionary and false otherwise. The value itself is returned via a .NET idiom called an out parameter. In F# code, three ways of using .NET methods rely on out parameters:

• You may use a local `mutable` in combination with the address-of operator `&`.

• You may use a reference cell.

• You may simply not give a parameter, and the result is returned as part of a tuple.

Here's how you do it using a mutable local:

```fs
open System.Collections.Generic
let lookupName nm (dict : Dictionary<string, string>) =
    let mutable res = ""
    let foundIt = dict.TryGetValue(nm, &res)
    if foundIt then res
    else failwithf "Didn't find %s" nm
```

The use of a reference cell can be cleaner. For example:

```fs
let res = ref ""
> capitals.TryGetValue("Australia", res);;
val it : bool = false
> capitals.TryGetValue("USA", res);;
val it : bool = true
> !res;;
val it : string = "Washington"
```

Finally, with this last technique you don't pass the final parameter, and instead the result is returned as part of a tuple:

```fs
> capitals.TryGetValue("Australia");;
val it : bool * string = (false, null)
> capitals.TryGetValue("USA");;
val it : bool * string = (true, "Washington")
```

Note that the value returned in the second element of the tuple may be null if the lookup fails when this technique is used; null values will be discussed in the section “Working with null Values” in Chapter 6.

### Using Dictionaries with Compound Keys

You can use dictionaries with compound keys, such as tuple keys of type (`int * int`). If necessary, you can specify the hash function to be used for these values when creating the instance of the dictionary. The default is to use generic hashing, also called structural hashing, a topic that will be covered in more detail in Chapter 9. To indicate this explicitly, specify `FSharp.Collections.HashIdentity.Structural` when creating the collection instance. In some cases, this can also lead to performance improvements, because the F# compiler often generates a hashing function appropriate for the compound type.

This example uses a dictionary with a compound key type to represent sparse maps:

```fs
> open System.Collections.Generic;;
> open FSharp.Collections;;
> let sparseMap = new Dictionary<(int * int), float>();;
val sparseMap : Dictionary<(int * int),float> = dict []
> sparseMap.[(0,2)] <- 4.0;;
> sparseMap.[(1021,1847)] <- 9.0;;
> sparseMap.Keys;;
val it : Dictionary'2.KeyCollection<(int * int),float> = seq [(0, 2); (1021, 1847)]
```

### Some Other Mutable Data Structures

Some other important mutable data structures in the F# and .NET libraries are:

• `System.Collections.Generic.SortedList<'Key,'Value>`: A collection of sorted values. Searches are done by a binary search. The underlying data structure is a single array.

• `System.Collections.Generic.SortedDictionary<'Key,'Value>`: A collection of key/value pairs sorted by the key, rather than hashed. Searches are done by a binary search. The underlying data structure is a single array.

• `System.Collections.Generic.Stack<'T>`: A variable-sized last-in/first-out (LIFO) collection.

• `System.Collections.Generic.Queue<'T>`: A variable-sized first-in/first-out (FIFO) collection.

• `System.Text.StringBuilder`: A mutable structure for building string values.

• `FSharp.Collections.HashSet<'Key>`: A hash table structure holding only keys and no values. Since .NET 3.5, a `HashSet<'T>` type is available in the `System.Collections.Generic` namespace.