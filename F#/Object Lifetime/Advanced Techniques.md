## Advanced Techniques

As I promised at the beginning of this chapter, you can boost your application's performance if you understand the garbage collection process more thoroughly. In the remaining sections, you'll learn about generations, weak references, resurrections, and how these techniques can help you write better .NET software.

### Generations

If the garbage collector had to visit all the objects referenced by an application, the garbage collection process might impose a severe overhead. Fortunately, some recurring patterns in object creation make it possible for the garbage collector to use heuristics that can significantly reduce the total execution time.

It has been observed that, from a statistical point of view, objects created early in the program's lifetime tend to stay alive longer than objects created later in a program do. Here's how you can intuitively justify this theory: objects created early are usually assigned to global variables and will be set to `null` only when the application ends, whereas objects created inside a class constructor method are usually released when the object is set to `null`. Finally, objects created inside a procedure are often destroyed when the procedure exits (unless they have been assigned to a variable defined outside the procedure, for example, an array or a collection).

The garbage collector has a simple way of determining how "old" an object is. Each object maintains a counter telling how many garbage collections that object has survived. The value of this counter is the object's *generation*. The higher this number is, the smaller the chances are that the object is collected during the next garbage collection.

The current version of the CLR supports only three distinct generation values. The generation value of an object that has never undergone a garbage collection is 0; if the object survives a garbage collection, its generation becomes 1; if it survives a second garbage collection, its generation becomes 2. Any subsequent garbage collection leaves the generation counter at 2 (or destroys the object). For example, an object that has a `Finalize` method always survives to the first garbage collection and is promoted to generation 1 because, as I explained earlier, the CLR can't sweep it out of the heap when the garbage collection occurs.

The CLR uses the generation counter to optimize the garbage collection process—for example, by moving the generation-2 objects toward the beginning of the heap, where they are likely to stay until the program terminates; they are followed by generation-1 objects and finally by generation-0 objects. This algorithm has proven to speed up the garbage collection process because it reduces the fragmentation of the managed heap.

You can learn the current generation of any object by passing it to the `GC.GetGeneration` method. The following code should give you a taste of how this method works:

```FSharp
let s: String = "dummy string"
// This is a generation-0 object.
Console.WriteLine(GC.GetGeneration(s))          // => 0
// Make it survive a first garbage collection.
GC.Collect(): GC.WaitForPendingFinalizers()
Console.WriteLine(GC.GetGeneration(s))          // => 1
// Make it survive a second garbage collection.
GC.Collect(): GC.WaitForPendingFinalizers()
Console.WriteLine(GC.GetGeneration(s))          // => 2
// Subsequent garbage collections don't increment the generation counter.
GC.Collect(): GC.WaitForPendingFinalizers()
Console.WriteLine(GC.GetGeneration(s))          // => 2
```

The `GC.Collect` method is overloaded to take a generation value as an argument, which results in the garbage collection of all the objects whose generation is lower than or equal to that value:

```FSharp
// Reclaim memory for unused generation-0 objects.
GC.Collect(0)
```

In general, the preceding statement is faster than running a complete garbage collection. To understand why, let's examine the three steps the garbage collection process consists of:

1. The garbage collector marks root objects and in general, all the objects directly or indirectly reachable from the application.
2. The heap is compacted, and all the marked (reachable) objects are moved toward the beginning of the managed heap to create a block of free memory near the end of the heap. Objects are sorted in the heap depending on their generation, with generation-2 objects near the beginning of the heap and generation-0 objects near the end of the heap, just before the free memory block. (To avoid time-consuming memory move operations, objects larger than approximately 85 KB are allocated in a separate heap that's never compacted.)
3. Root object variables in the main application are updated to point to the new positions of objects in the managed heap.

You speed up the second step (fewer objects must be moved in the heap) as well as the third step (because only a subset of all variables are updated) when you collect only generation-0 objects. Under certain conditions, even the first step is completed in less time, but this optimization technique might seem counterintuitive and requires an additional explanation.

Let's say that the garbage collector reaches a generation-1 object while traversing the object graph. Let's call this object A. In general, the collector can't simply ignore the portion of the object graph that has object A as its root because this object might point to one or more generation-0 objects that need to be marked. (For example, this might happen if object A is an array that contains objects created later in the program's lifetime.) However, the CLR can detect whether fields of object A have been modified since the previous garbage collection. If it turns out that object A hasn't been written to in the meantime, it means that object A can point only to generation-1 and generation-2 objects, so there is no reason for the collector to analyze that portion of the object graph because it was analyzed during the previous garbage collection and can't point to any generation-0 objects. (Of course, a similar reasoning applies when you use the `GC.Collect(1)` statement to collect only generation-0 and generation-1 objects.)

The CLR often attempts to collect only generation-0 objects to improve overall performance; if the garbage collection is successful in freeing enough memory, no further steps are taken. Otherwise, the CLR attempts to collect generation-0 and generation-1 objects; if this second attempt fails to free enough memory, it collects all three generations. This means that older-generation objects might live undisturbed in the heap a long time after the application logically killed them. The exact details of the type of garbage collection the CLR performs each time are vastly undocumented and might change over time.

**Version 2005 of VB or Version 2.0 of .NET** You can determine how many garbage collections of a given generation have occurred by querying the new `CollectionCount` method of the GC type:

```FSharp
// Determine how many 2-gen collections have occurred so far.
let count: Int32 = GC.CollectionCount(2)
```

### Garbage Collection and Performance

Before moving to a different topic, I want to show you how efficient .NET is at managing memory. Let's again run a code snippet similar to the one I showed in the section titled "The Finalize Method" earlier in this chapter, but comment out the statement that explicitly sets the `Person` object to `null`:

```FSharp
// Compile this code with optimizations enabled.
let Main() =
    Console.WriteLine("About to create a Person object.")
    let mutable aPerson = new Person()
    // aPerson <- null
    // After this point, aPerson is a candidate for garbage collection.
    Console.WriteLine("About to fire a garbage collection.")
    GC.Collect()
    GC.WaitForPendingFinalizers()
    Console.WriteLine("About to terminate the application")
```

You might expect that the `Person` object is kept alive until the method exits. However, if optimizations are enabled and you run the application in Release mode, the JIT compiler is smart enough to detect that the object isn't used after the call to its constructor, so it marks it as a candidate for garbage collection. As a result, the code behaves exactly as if you explicitly set the variable to `null` after the last statement that references it. In other words, setting a variable to `null` as soon as you are done with an object doesn't necessarily make your code more efficient because the JIT compiler can apply this optimization technique automatically.

In one special case, however, explicitly setting a variable to `null` can affect performance positively. This happens when you destroy an object in the middle of a loop. In this case, the Visual Basic compiler can't automatically detect whether the variable is going to be used during subsequent iterations of the loop, and therefore the garbage collector can't automatically reclaim the memory used by the object. By clearing the object variable explicitly, you can help the garbage collector understand that the object can be reclaimed.

```FSharp
// Use the aPerson object inside the loop.
for i = 1 to 100 do
    if i<=50 then
        // Use the object only in the first 50 iterations.
        Console.WriteLine(sprintf "%A" aPerson)
    // Explicitly set the variable to Nothing after its last use.
    if i = 50 then 
        aPerson <- null
    else
        // Do something else here, but don't use the aPerson variable.
        ()
```

The fact that an object can be destroyed any time after the last time you reference it in code, and well before its reference goes out of scope, can have a surprisingly dangerous effect if the object wraps an unmanaged resource that is freed in the object's `Finalize` method. Say that you have authored a type named `WinFile`, which opens a file using the OpenFile API method and closes the file in the `Finalize` method:

```FSharp
let TestWinFile() =
    let wfile = new WinFile(@"c:\data.txt")
    let handle: Int32 = wfile.Handle
    // Process the file by passing the handle to native Windows methods.
    ()
    // (The file is automatically closed in WinFile's Finalize.)
```

The problem here is that the garbage collector might collect the `WinFile` object and indirectly fire its `Finalize` method, which in turn would close the file before the procedure has completed its tasks. You might believe that adding a reference to the `WinFile` object at the end of the procedure would do the trick, but you'd be wrong. Consider this code:

```FSharp
let DoNothingProc(obj : Object) =
    // No code here
    ()
let TestWinFile() =
    // A failed attempt to keep the object alive until the end of the method
    DoNothingProc(wfile)
```

Surprisingly, the Visual Basic compiler is smart enough to realize that the `DoNothingProc` doesn't really use the object reference, and therefore passing the object to this procedure won't keep the `WinFile` object alive. The only method that is guaranteed to work in this case is the `GC.KeepAlive` method, whose name says it all:

```FSharp
let TestWinFile() =
    …
    // Keep the object alive until the end of the method.
    GC.KeepAlive(wfile)
```

In practice, however, you should never need the `GC.KeepAlive` method. In fact, if you authored the `WinFile` type correctly, it should expose the `IDisposable` interface, and therefore the actual code should look like this:

```FSharp
let TestWinFile() =
    use wfile = new WinFile(@"c:\data.txt")
    let handle: Int32 = wfile.Handle
    // Process the file by passing the handle to native Windows methods.
    ()
```

**Version 2005 of VB or Version 2.0 of .NET** The GC object in .NET Framework version 2.0 has two new methods that enable you to let the garbage collector know that an unmanaged object consuming a lot of memory has been allocated or released so that the collector can fine-tune its performance and optimize its scheduling. You should invoke the `AddMemoryPressure` method to inform the garbage collector that the specified number of bytes has been allocated in the unmanaged memory; after releasing the object, you should invoke the `RemoveMemoryPressure` method to notify the CLR that an unmanaged object has been released and that the specified amount of memory is available again.

```FSharp
// Allocate an unmanaged object that takes approximately 1 MB of memory.
let mutable obj = new UnmanagedResource()
GC.AddMemoryPressure(1048576)        // = 2^20
…
// Release the object here.
obj <- null
GC.RemoveMemoryPressure(1048576)
```

Even though .NET garbage collection is quite efficient, it can be a major source of overhead. In fact, not counting file and database operations, garbage collection is among the slowest activities that can take place during your application's lifetime; thus, it's your responsibility to keep the number of collections to a minimum. If you suspect that an application is running slowly because of too frequent garbage collections, use the Performance utility to monitor the following performance counters of the .NET CLR Memory object, as shown in Figure 9-1:

- \# Gen 0 Collections, # Gen 1 Collections, # Gen 2 Collections (the number of garbage collections fired by the CLR)
- % Time In GC (the percentage of CPU time spent performing garbage collections)
- Gen 0 Heap Size, Gen 1 Heap Size, Gen 2 Heap Size, Large Object Heap Size (the size of the four heaps used by the garbage collector)
- \# Bytes In All Heaps (the sum of the four heaps used by the application, that is, Generation-0, Generation-1, Generation-2, and Large Object heaps)

![Image from book](images/fig389%5F01%5F0%`2Ejpg`) 
Figure 9-1: Using the Performance utility to monitor .NET memory performance counters 

If the value of the first two sets of counters is suspiciously high, you can conclude that garbage collections are killing your application's performance and should browse your source code looking for the causes of the higher-than-usual activity of the managed heap. I have compiled a list of issues you should pay attention to and which counters can help you solve your performance problem:

1. Consider whether you can reduce the usage of the managed heap by defining structures instead of sealed classes. Structures defined as local variables are allocated on the stack and don't take space in the managed heap. On the other hand, the assignment of a structure to an `Object` variable causes the structure to be boxed, which takes memory from the heap and adds overhead, so you should take this detail into account when opting for a structure. Also, don't use a structure if your type implements one or more interfaces because invoking a method through an interface variable forces the boxing of the structure.

2. Use `Char` variables instead of `String` variables if possible because `Char` is a value type and `Char` objects don't take space in the managed heap. More important, attempt to reduce the number of temporary strings used in expressions. When many concatenation operations are involved, use a `StringBuilder` object. (I cover the StringBuilder type in Chapter 12, ".NET Basic Types.")

3. Allocate disposable objects inside Using blocks; if a Using block can't be used, allocate the disposable object inside a Try…Catch block and invoke the `Dispose` method from inside the Finally clause.

4. Allocate long-lasting objects earlier and assign them to global variables. This technique ensures that these objects will be moved near the beginning of the managed heap when the application starts and won't move from there during the application's lifetime. You can detect whether you have many objects that are candidates for this treatment by monitoring the Protected Memory From Gen 0 and Protected Memory From Gen 1 performance counters.

5. Avoid creating objects larger than about 85 KB if possible. These objects are stored in a separate area known as a large objects heap; moving these large objects in memory would add too much overhead, and therefore .NET never compacts this heap. As a result, the large object heap might become fragmented. An example of such a large object is an array of `Double` numbers (8 bytes each) with more than about 10,880 elements. You can detect whether you have many large objects by looking at the Large Object Heap Size performance counter.

6. As a rule, never fire a garbage collection by means of the `GC.Collect` method. In client applications, such as a Console or a Windows Forms application, you might start a collection only after an intensive user-interface action, for example, after loading or saving a file, so that the user won't perceive the extra overhead. Never use the `GC.Collect` method from inside a server-side application, such as an ASP.NET application or a COM+ library. You can detect whether you have too many `GC.Collect` methods in your code by monitoring the # Induced GC performance counter.

7. Never implement the `Finalize` method without a good reason to do so. If you do need a `Finalize` method, adopt the recommended pattern described earlier in this chapter. You can detect whether your finalizable objects can be the cause of a performance problem by looking at the Finalization Survivors performance counter.

8. If you must implement the `Finalize` method, ensure that the object doesn't take a significant amount of managed memory in addition to unmanaged resources. You can achieve this by wrapping the unmanaged resource in a nested class, as described in the section titled "A Simplified Approach to Finalization" earlier in this chapter.

### Weak Object References

The .NET Framework provides a special type of object reference that doesn't keep an object alive: the *weak reference*. A weakly referenced object can be reclaimed during a garbage collection and must be re-created afterward if you want to use it again. Typical candidates for this technique are objects that take a lot of memory but whose state can be re-created with relatively little effort. For example, consider a class whose main purpose is to provide an optimized cache for the contents of text files. Traditionally, whenever you cache a large amount of data you must decide how much memory you set aside for the cache, but reserving too much memory for the cache might make the overall performance worse.

You don't have this dilemma if you use weak references. You can store as much data in the cache as you wish because you know that the system will automatically reclaim that memory when it needs it. Here's the complete source code for the `CachedFile` class:

```FSharp
// The constructor takes the name of the file to read.
// The name of the file cached
type CachedFile(filename : String) =
    // A weak reference to the string that contains the text
    let mutable wrText : WeakReference = null

    // Read the contents of the file.
    member this.ReadFile() : String =
        let text: String = File.ReadAllText(filename)
        // Create a weak reference to this string.
        wrText <- new WeakReference(text)
        text

    // Return the textual content of the file.
    member this.GetText() : String =
        let mutable text: Object = null
        // Retrieve the target of the weak reference.
        if wrText <> null then text <- wrText.Target
        if text <> null then
            // If nonnull, the data is still in the cache.
            text.ToString()
        else
            // Otherwise, read it and put it in the cache again.
            this.ReadFile()

```

There are two points of interest in this class. First, the `ReadFile` passes the value to be returned to the caller (the text variable) to the constructor of the `WeakReference` class and therefore creates a weak reference to the string. The second point of interest is in the `GetText` method, where the code queries the `WeakReference.Target` property. If this property returns a non-null value, it means the weak reference hasn't been broken and still points to the original, cached string. Otherwise, the `ReadFile` method is invoked so that the file contents are read from disk and cached once again before being returned to the caller. Using the `CachedFile` type is easy:

```FSharp
// Read and cache the contents of the @"c:\alargefile.txt" file.
let cf = new CachedFile(@"c:\alargefile.txt")
Console.WriteLine(cf.GetText())
…
// Uncomment next 2 lines to force a garbage collection.
// GC.Collect()
// GC.WaitForPendingFinalizers()
…
// Read the contents again sometime later.
// (No disk access is performed, unless a garbage collection has occurred in the meantime.)
Console.WriteLine(cf.GetText())
```

By tracing into the `CachedFile` class, you can easily prove that in most cases the file contents can be retrieved through the weak reference and that the disk isn't accessed again. By uncommenting the statement in the middle of the previous code snippet, however, you force a garbage collection, in which case the internal `WeakReference` object won't keep the `String` object alive and the code in the class will read the file again. The key point in this technique is that the client code doesn't know whether the cached data is used or a disk access is required. It just uses the `CachedFile` object as an optimized building block for dealing with large text files.

Remember that you create a weak reference by passing your object to the constructor of a `System.WeakReference` object. However, if the object is also pointed to by a regular, nonweak reference, it will survive any intervening garbage collection. In our example, this means that the code using the `CachedFile` class should not store the return value of the `GetText` method in a string variable because that would prevent the string from being garbage collected until that variable is explicitly set to `null` or goes out of scope:

```FSharp
// The wrong way of using the CachedFile class
let cf = new CachedFile(@"c:\alargefile.txt")
let text: String = cf.GetText()
// The text string will survive any garbage collection.
```

### Object Resurrection

Weak references are fine to create a cache of objects that are frequently used and that can be re-created in a relatively short time. The technique I illustrate in this section offers a slightly different solution to the same problem.

Earlier in this chapter, I briefly described the technique known as *object resurrection* through which an object being finalized can store a reference to itself in a variable defined outside the current instance so that this new reference keeps the object alive. Object resurrection is likely to be useful only in unusual scenarios, such as when you're implementing a pool of objects whose creation and destruction are time-consuming. For example, let's consider the following sample class, which implements an array containing a sequence of random `Double` values:

```FSharp
// The constructor creates the random array.
type RandomArray(length : Int32) =
    let rand = new Random()
    // This is a time-consuming operation.
    let values = [|
        for i = 1 to length do
            yield rand.NextDouble()
    |]

    // This array stores the elements.
    member this.Values = values
```

(Notice that you'll typically apply resurrection to objects that are far more complicated than this one and that use unmanaged resources as well. I am using `RandomArray` only for illustration purposes.)

If the main application creates and destroys thousands of `RandomArray` objects, it makes sense to implement a mechanism by which a `RandomArray` object is returned to an internal pool when the application doesn't need it any longer; if the application later requests another `RandomArray` object with the same number of elements, the object is taken from the pool instead of going through the relatively slow initialization process.

The internal pool is implemented as a static `ArrayList` object. To ensure that the application uses a pooled object, if available, the `RandomArray` class has a private constructor and a static public factory method named `Create`. Also, the class implements the `Finalize` method to ensure that the instance is correctly returned to the pool when the application has finished with it. Here's the new version of the class:

```FSharp
// The pool of objects
let Pool = new ArrayList()

// The constructor creates the random array.
type RandomArray(length : Int32) =
    let rand = new Random()
    // This is a time-consuming operation.
    let values = [|
        for i = 1 to length do
            yield rand.NextDouble()
    |]

    // This array stores the elements.
    member this.Values = values

    override this.Finalize() =
        // Resurrect the object by putting it into the pool.
        Pool.Add(this) |> ignore

// The factory method
let Create(length : Int32) : RandomArray =
    // Check whether there is an element in the pool with the
    // requested number of elements.
    seq { 
        for i = 0 to Pool.Count - 1 do
            yield Pool.[i] :?> RandomArray
    }
    |> Seq.tryFindIndex(fun ra -> ra.Values.Length = length)
    |> function 
        | Some i ->
            // Remove the element from the pool.
            let ra = Pool.[i] :?> RandomArray
            Pool.RemoveAt(i)
            // Reregister for finalization, in case no Dispose method is invoked.
            GC.ReRegisterForFinalize(ra)
            ra
        | _ -> 
            // If no suitable element in the pool, create a new element.
            new RandomArray(length)
```

The key point in the previous code is the `GC.ReRegisterForFinalize` method call in the `Create` method. Without this call, the object handed to the application wouldn't execute the `Finalize` method and therefore it would miss the chance of being returned to the pool when the application logically destroys the object. (The companion code includes a better version of the `RandomArray` type that implements `IDisposable` to let the client application explicitly return an instance to the pool.)

Here's a piece of code that uses the `RandomArray` object:

```FSharp
// Create and use a few RandomArray objects.
let ra1: RandomArray = RandomArray.Create(10000)
let ra2: RandomArray = RandomArray.Create(20000)
let ra3: RandomArray = RandomArray.Create(30000)
…
// Clear some of them.

ra2 = null
ra3 = null
// Simulate a garbage collection, which moves the two cleared objects to the pool.
GC.Collect()
GC.WaitForPendingFinalizers()

// Create a few more objects, which will be taken from the pool.
let ra4: RandomArray = RandomArray.Create(20000)
let ra5: RandomArray = RandomArray.Create(30000)
```

A pooled object might improve performance in other ways, for example, by creating a few instances in advance so that no initialization time is spent when the application asks for them. Object pooling can be useful in cases when performance isn't the main issue—for example, when you have a threshold for the number of objects that can be created.

### Garbage Collection on Multi-CPU Computers

When the .NET runtime is executing on a workstation, it's important that the user interface work as smoothly as possible, even at the cost of some loss of overall performance. On the other hand, performance becomes the top priority when an enterprise-level .NET application runs on a server machine. To account for these differences, the .NET Framework comes with two types of garbage collectors: the workstation version and the multi-CPU server version.

When running on a single-CPU machine, the collector always works in workstation mode. In this mode, the collector runs on a concurrent thread to minimize pauses, but the additional thread-switching activity can slow the application as a whole. When the workstation version is used on a multi-CPU system, you have the option of running the garbage collector in concurrent mode: in this case, the GC thread runs on a separate CPU to minimize pauses. You activate concurrent mode by adding an entry to the configuration file:

```xml
<configuration>
   <runtime>
      <gcConcurrent enabled="true" />
   </runtime>
</configuration>
```

When the server version is used, objects are allocated from multiple heaps; the program freezes during a garbage collection, and each CPU works concurrently to compact one of the heaps. This technique improves overall performance and scales well when you install additional CPUs. The server version can be used only on multi-CPU systems or on systems equipped with a hyperthreaded CPU.

**Version 2005 of VB or Version 2.0 of .NET** In previous versions of the .NET Framework, the server and the workstation versions of the garbage collector resided in different DLLs (mscorsvr.dll and mscorwks.dll) and you explicitly had to select a different DLL to enable the server version. In .NET Framework 2.0, the two versions have been merged in the mscorwks.dll, and you enable the server version by adding an entry to the configuration file:

```xml
<configuration>
   <runtime>
      <gcServer enabled="true" />
   </runtime>
</configuration>
```

Also new in .NET Framework 2.0 is the `System.Runtime.GCSettings` type, which exposes a static property that enables you to detect whether the server version of the garbage collector is used:

```FSharp
if System.Runtime.GCSettings.IsServerGC then
    Console.WriteLine("The server version of the garbage collector is used.")
```