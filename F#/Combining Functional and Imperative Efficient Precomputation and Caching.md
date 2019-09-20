## Combining Functional and Imperative Efficient Precomputation and Caching

All experienced programmers are familiar with the concept of precomputation, in which computations are performed as soon as some of the inputs to a function are known. The following sections cover a number of manifestations of precomputation in F# programming, as well as the related topics of memoization and caching. These represent one common pattern in which imperative programming is used safely and non-intrusively within a broader setting of functional programming.

### Precomputation and Partial Application

Let’s say you’re given a large input list of words, and you want to compute a function that checks whether a word is in this list. You would do this:

```F#
let isWord (words : string list) =
    let wordTable = Set.ofList words
    fun w -> wordTable.Contains(w)
```

Here, isWord has the following type:

```F#
val isWord : words:string list -> (string -> bool)
```

The efficient use of this function depends crucially on the fact that useful intermediary results are computed after only one argument is applied and a function value is returned. For example:

```F#
> let isCapital = isWord ["London"; "Paris"; "Warsaw"; "Tokyo"];;
val isCapital : (string -> bool)
> isCapital "Paris";;
val it : bool = true
> isCapital "Manchester";;
val it : bool = false
```

Here, the internal table wordTable is computed as soon as isCapital is applied to one argument. It would be wasteful to write isCapital as:

```F#
let isCapitalSlow word = isWord ["London"; "Paris"; "Warsaw"; "Tokyo"] word
```

This function computes the same results as isCapital. It does so inefficiently, however, because isWord is applied to both its first argument and its second argument every time you use the function isCapitalSlow. This means the internal table is rebuilt every time the function isCapitalSlow is applied, somewhat defeating the point of having an internal table in the first place. In a similar vein, the definition of isCapital shown previously is more efficient than either isCapitalSlow2 or isCapitalSlow3 in the following:

```F#
let isWordSlow2 (words : string list) (word : string) =
    List.exists (fun word2 -> word = word2) words
let isCapitalSlow2 word = isWordSlow2 ["London"; "Paris"; "Warsaw"; "Tokyo"] word

let isWordSlow3 (words : string list) (word : string) =
    let wordTable = Set<string>(words)
    wordTable.Contains(word)
let isCapitalSlow3 word = isWordSlow3 ["London"; "Paris"; "Warsaw"; "Tokyo"] word
```

The first uses an inappropriate data structure for the lookup (an F# list, which has O(n) lookup time), and the second attempts to build a better intermediate data structure (an F# set, which has O(log n) lookup time), but does so on every invocation.

There are often trade-offs among different intermediate data structures, or when deciding whether to use them at all. For example, in the previous example you could just as well use a `HashSet` as the internal data structure. This approach, in general, gives better lookup times (constant time), but it requires slightly more care to use, because a `HashSet` is a mutable data structure. In this case, you don’t mutate the data structure after you create it, and you don’t reveal it to the outside world, so it’s entirely safe:

```F#
let isWord (words : string list) =
    let wordTable = HashSet<string>(words)
    fun word -> wordTable.Contains word
```

### Precomputation and Objects

The examples of precomputation given thus far are variations on the theme of computing functions, introduced in Chapter 3. The functions, when computed, capture the precomputed intermediate data structures. It’s clear, however, that precomputing via partial applications and functions can be subtle, because it matters when you apply the first argument of a function (triggering the construction of intermediate data structures) and when you apply the subsequent arguments (triggering the real computation that uses the intermediate data structures).

Luckily, functions don’t just have to compute functions; they can also return more sophisticated values, such as objects. This can help make it clear when precomputation is being performed. It also allows you to build richer services based on precomputed results. For example, Listing 4-1 shows how to use precomputation as part of building a name-lookup service. The returned object includes both a Contains method and a ClosestPrefixMatch method.

##### Listing 4-1. Precomputing a eord table before creating an object

```F#
open System

type NameLookupService =
    abstract Contains : string -> bool

let buildSimpleNameLookup (words : string list) =
    let wordTable = HashSet<_>(words)
    {new NameLookupService with
        member t.Contains w = wordTable.Contains w}
```

The internal data structure used in Listing 4-1 is the same as before: an F# set of type `FSharp.Collections.Set<string>`. The service can now be instantiated and used as follows:

```F#
> let capitalLookup = buildSimpleNameLookup ["London"; "Paris"; "Warsaw"; "Tokyo"];;
val capitalLookup : NameLookupService
> capitalLookup.Contains "Paris";;
val it : bool = true
```

In passing, note the following about this implementation:

* You can extend the returned service to support a richer set of queries of the underlying information by adding further methods to the object returned.

Precomputation of the kind used previously is an essential technique for implementing many services and abstractions, from simple functions to sophisticated computation engines. You will see further examples of these techniques in Chapter 9.

### Memoizing Computations

Precomputation is one important way to amortize the costs of computation in F#. Another is called memoization. A memoizing function avoids recomputing its results by keeping an internal table, often called a lookaside table. For example, consider the well-known Fibonacci function, whose naive, unmemoized version is:

```F#
let rec fib n = if n <= 2 then 1 else fib (n - 1) + fib (n - 2)
```

Not surprisingly, a version keeping a lookaside table is much faster:

```F#
open System.Collections.Generic

let fibFast =
    let t = new Dictionary<int, int>()
    let rec fibCached n =
        if t.ContainsKey n then t.[n]
        elif n <= 2 then 1
        else 
            let res = fibCached (n - 1) + fibCached (n - 2)
            t.Add (n, res)
            res
    fun n -> fibCached n

// From Chapter 2, but modified to use stop watch.
let time f =
    let sw = System.Diagnostics.Stopwatch.StartNew()
    let res = f()
    let finish = sw.Stop()
    (res, sw.Elapsed.TotalMilliseconds |> sprintf "%f ms")
time(fun () -> fibFast 30)
time(fun () -> fibFast 30)
time(fun () -> fibFast 30)

```

the result is:

```F#
val fibFast : (int -> int)
val time : f:(unit -> 'a) -> 'a * string
val it : int * string = (832040, "0.727200 ms")
val it : int * string = (832040, "0.066100 ms")
val it : int * string = (832040, "0.077400 ms")
```

On one of our laptops, with n = 30, there’s an order of magnitude speed up from the first to second run. Listing 4-2 shows how to write a generic function that encapsulates the memoization technique.

##### Listing 4-2. A generic memoization function

```F#
open System.Collections.Generic

let memoize (f : 'T -> 'U) =
    let t = new Dictionary<'T, 'U>(HashIdentity.Structural)
    fun n ->
        if t.ContainsKey n then t.[n]
        else
            let res = f n
            t.Add (n, res)
            res

let rec fibFast =
    memoize (fun n -> if n <= 2 then 1 else fibFast (n - 1) + fibFast (n - 2))
```

Here, the functions have the types:

```F#
val memoize : f:('T -> 'U) -> ('T -> 'U) when 'T : equality
val fibFast : (int -> int)
```

In the definition of fibFast, you use let rec because fibFast is self-referential—that is, used as part of its own definition. You can think of fibFast as a computed, recursive function. Such a function generates an informational warning when used in F# code, because it’s important to understand when this feature of F# is being used; you then suppress the warning with `#nowarn "40"`. As with the examples of computed functions from the previous section, omit the extra argument from the application of memoize, because including it would lead to a fresh memoization table being allocated each time the function fibNotFast was called:

```F#
let rec fibNotFast n =
    memoize (fun n -> if n <= 2 then 1 else fibNotFast (n - 1) + fibNotFast (n - 2)) n
```

Due to this subtlety, it’s often a good idea to define your memoization strategies to generate objects other than functions (think of functions as very simple kinds of objects). For example, Listing 4-3 shows how to define a new variation on memoize that returns a Table object that supports both a lookup and a Discard method.

##### Listing 4-3. A generic memoization service

```F#
open System.Collections.Generic

type Table<'T, 'U> =
    abstract Item : 'T -> 'U with get
    abstract Discard : unit -> unit

let memoizeAndPermitDiscard f =
    let lookasideTable = new Dictionary<_, _>(HashIdentity.Structural)
    {new Table<'T, 'U> with
        member t.Item
            with get(n) =
                if lookasideTable.ContainsKey(n) then
                    lookasideTable.[n]
                else
                    let res = f n
                    lookasideTable.Add(n, res)
                    res
        member t.Discard() =
            lookasideTable.Clear()}

#nowarn "40" // do not warn on recursive computed objects and functions

let rec fibFast =
    memoizeAndPermitDiscard (fun n ->
        printfn "computing fibFast %d" n
        if n <= 2 then 1 else fibFast.[n - 1] + fibFast.[n - 2])

```

In Listing 4-3, lookup uses the a.[b] associative Item lookup property syntax, and the Discard method 
discards any internal partial results. The functions have these types:

```F#
val memoizeAndPermitDiscard : ('T -> 'U) -> Table<'T, 'U> when 'T : equality
val fibFast : Table<int,int>
```

This example shows how fibFast caches results but recomputes them after a Discard:

```F#
> fibFast.[3];;
computing fibFast 3
computing fibFast 2
computing fibFast 1
val it : int = 2
> fibFast.[5];;
computing fibFast 5
computing fibFast 4
val it : int = 5

> fibFast.Discard();;
> fibFast.[5];;
computing fibFast 5
computing fibFast 4
computing fibFast 3
computing fibFast 2
computing fibFast 1
val it : int = 5
```

---

##### Note  

memoization relies on the memoized function being stable and idempotent. In other words, it always returns the same results, and no additional interesting side effects are caused by further invocations of the function. In addition, memoization strategies rely on mutable internal tables. the implementation of memoize shown in this chapter isn’t thread safe, because it doesn’t lock this table during reading or writing. this is fine if the computed function is used only from at most one thread at a time, but in a multithreaded application, use memoization strategies that utilize internal tables protected by locks, such as a .net ReaderWriterLock. Chapter 11 will further discuss thread synchronization and mutable state.

---

### Lazy Values

Memoization is a form of caching. Another important variation on caching is a simple lazy value, a delayed computation of type `FSharp.Control.Lazy<'T>` for some type `'T`. Lazy values are usually formed by using the special keyword `lazy` (you can also make them explicitly by using the functions in the `FSharp.Core.Lazy` module). For example:

```F#
> let sixty = lazy (30 + 30);;
val sixty : Lazy<int> = Value is not created.
> sixty.Force();;
val it : int = 60
```

Lazy values of this kind are implemented as thunks holding either a function value that computes the 
result or the actual computed result. The lazy value is computed only once, and thus its effects are executed only once. For example, in the following code fragment, ¡°Hello world¡± is printed only once:

```F#
> let sixtyWithSideEffect = lazy (printfn "Hello world"; 30 + 30);;
val sixtyWithSideEffect: Lazy<int> = Value is not created.
> sixtyWithSideEffect.Force();;
Hello world
val it : int = 60
> sixtyWithSideEffect.Force();;
val it : int = 60
```

Lazy values are implemented by a simple data structure containing a mutable reference cell. The definition of this data structure is in the F# library source code.

### Other Variations on Caching and Memoization

You can apply many different caching and memoization techniques in advanced programming, and this chapter can’t cover them all. Some common variations are:

* Using an internal data structure that records only the last invocation of a function and basing the lookup on a very cheap test on the input.

* Using an internal data structure that contains both a fixed-size queue of input keys and a dictionary of results. Entries are added to both the table and the queue as they’re computed. When the queue is full, the input keys for the oldest computed results are dequeued, and the computed results are discarded from the dictionary.

* Some of these techniques are encapsulated in the .NET type `System.Runtime.Caching.MemoryCache` that is found in the system library System.Runtime.Caching.dll.

### Mutable Reference Cells

One mutable record type that you will see occasionally (especially in F# code before F# 4.0) is the general-purpose type of mutable reference cells, or ¡°ref cells¡± for short. These play much the same role as pointers in other imperative programming languages. You can see how to use mutable reference cells in this example:

```F#
> let cell1 = ref 1;;
val cell1 : int ref = {contents = 1;}
> cell1.Value;;
val it : int = 1
> cell1 := 3;;
val it : unit = ()
> cell1;;
val it : int ref = {contents = 3;}
> cell1.Value;;
val it : int = 3
The type is 'T ref, and three associated functions are ref, !, and :=. The types of these are:
val ref : 'T -> 'T ref
val (:=) : 'T ref -> 'T -> unit
val (!) : 'T ref -> 'T

```

These allocate a reference cell, mutate the cell, and read the cell, respectively. The operation `cell1 := 3` is the key one; after this operation, the value returned by evaluating the expression `!cell1` is changed. You can also use either the contents field or the Value property to access the value of a reference cell. Both the 'T ref type and its operations are defined in the F# library as simple record-data structures with a single mutable field. The type 'T ref is a synonym for a type `FSharp.Core.Ref<'T>` that is defined in this way:

```F#
type 'T ref =
    {mutable contents : 'T}
    member cell.Value = cell.contents
let (!) r = r.contents
let (:=) r v = r.contents <- v
let ref v = {contents = v}
```

When using F# 4.0 you don’t need to use ref cells as much as with previous versions of F#, as a single ¡°let mutable¡± can usually be used as an alternative.

