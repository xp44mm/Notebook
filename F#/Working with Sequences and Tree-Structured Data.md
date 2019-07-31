# Working with Sequences and Tree-Structured Data

In the previous chapter, you learned to manipulate textual data, including how to convert text to structured data through parsing and back again through formatting. In this chapter, you will focus on F# programming techniques related to structured data.

Structured data and its processing are broad topics that lie at the heart of much applied F# programming. In this chapter, you will learn about a number of applied topics in structured programming, including:

- producing, transforming, and querying in-memory data—in particular, using sequence-based programming

- defining and working with domain models, including some efficiency-related representation techniques

- working with tree-structured data, including recursive processing and rewriting of trees

- building new data structures, including implementing equality and comparison operations for these types

- defining structured views of existing data structures using active patterns

Closely related to programming with structured data is the use of recursion in F# to describe some algorithms. In order to use recursion effectively over structured data, you will also learn about tail calls and how they affect the use of the “stack” as a computational resource.

## Getting Started with Sequences

Many programming tasks require the iteration, aggregation, and transformation of data streamed from various sources. One important and general way to code these tasks is in terms of values of the type `System.Collections.Generic.IEnumerable<type>`, which is typically abbreviated to `seq<type>` in F# code. A `seq<type>` is a value that can be iterated, producing results of type `type` on demand. Sequences are used to wrap collections, computations, and data streams and are frequently used to represent the results of database queries. The following sections will present some simple examples of working with `seq<type>` values.

### Using Range Expressions

You can generate simple sequences using *range expressions*. For integer ranges, these take the form of `seq {n .. m}` for the integer expressions n and m:

```F#
> seq {0 .. 2};;
val it : seq<int> = seq [0; 1; 2;]
```

You can also specify range expressions using other numeric types, such as double and single:

```F#
> seq {-100.0 .. 100.0};;
val it : seq<float> = seq [-100.0; -99.0; -98.0; -97.0; ...]

```

By default, F# Interactive shows the value of a sequence only to a limited depth; `seq<'T>` values are lazy in the sense that they compute and return the successive elements on demand. This means you can create sequences representing very large ranges, and the elements of the sequence are computed only if they’re required by a subsequent computation. In the next example, you don’t actually create a concrete data structure containing one trillion elements, but rather you create a sequence value that has the potential to yield this number of elements on demand. The default printing performed by F# Interactive forces this computation up to depth 4:

```F#
> seq {1 .. 10000000 };;
val it : seq<int> =  seq [1 ; 2; 3; 4; ...]
```

The default increment for range expressions is always 1. A different increment can be used via range expressions of the form `seq {n .. skip .. m}`:

```F#
> seq {1 .. 2 .. 5};;
val it : seq<int> = seq [1; 3; 5]
> seq {1 .. -2 .. -5};;
val it : seq<int> = seq [1; -1; -3; -5]
```

If the skip causes the final element to be overshot, then the final element isn’t included in the result:

```F#
> seq {0 .. 2 .. 5};;
val it : seq<int> = seq [0; 2; 4]

```

The `(..)` and `(.. ..)` operators are overloaded operators in the same sense as `(+)` and `(-)`, which means their behavior can be altered for user-defined types. Chapter 6 discussed this in more detail.

### Iterating a Sequence

You can iterate sequences using the `for ... in ... do` construct, as well as the `Seq.iter` function discussed in the next section. Here is a simple example of the first:

```F#
> let range = seq {0 .. 2 .. 6};;
val range : seq<int>
> for i in range do printfn "i = %d" i;;
i = 0
i = 2
i = 4
i = 6
```

This construct forces the iteration of the entire seq. Use it with care when you’re working with sequences that may yield a large number of elements.

### Transforming Sequences with Functions

Any value of type `seq<type>` can be iterated and transformed using functions in the `FSharp.Collections.Seq` module. For example:

```F#
> let range = seq {0 .. 10};;
val range : seq<int>
> range |> Seq.map (fun i -> (i, i*i));;
val it : seq<int * int> = seq [(0, 0); (1, 1); (2, 4); (3, 9); ...]

```

Table 9-1 shows some important functions in this library module. The following operators necessarily evaluate all the elements of the input seq immediately:

- `Seq.iter`: This iterates all elements, applying a function to each.

- `Seq.toList`: This iterates all elements, building a new list.

- `Seq.toArray`: This iterates all elements, building a new array.

Table 9-1.  Some Important Functions from the Seq Module

```F#
Seq.append   : seq<'T> -> seq<'T> -> seq<'T>
Seq.concat   : seq<#seq<'T>> -> seq<'T>
Seq.choose   : ('T -> 'U option) -> seq<'T> -> seq<'U>
Seq.delay    : (unit -> seq<'T>) -> seq<'T>
Seq.empty    : seq<'T>
Seq.iter     : ('T -> unit) -> seq<'T> -> unit
Seq.filter   : ('T -> bool) -> seq<'T> -> seq<'T>
Seq.map      : ('T -> 'U) -> seq<'T> -> seq<'U>
Seq.singleton: 'T -> seq<'T>
Seq.truncate : int -> seq<'T> -> seq<'T>
Seq.toList   : seq<'T> -> 'T list
Seq.ofList   : 'T list -> seq<'T>
Seq.toArray  : seq<'T> -> 'T[]
Seq.ofArray  : 'T[] -> seq<'T>
```

Most other operators in the Seq module return one or more `seq<type>` values and force the computation of elements in any input `seq<type>` values only on demand.

### Which Types Can Be Used as Sequences?

Table 9-1 includes many uses of types such as `seq<'T>`. When a type appears as the type of an argument, the function accepts any value that’s compatible with this type. Chapter 5 explained the notions of subtyping and compatibility in more detail; the concept should be familiar to OO programmers, because it’s the same as that used by languages such as C#. In practice, you can easily discover which types are compatible with which others by using F# Interactive and tools such as Visual Studio: when you hover over a type name, the compatible types are shown. You can also refer to the online documentation for the F# libraries and the .NET Framework, which you can easily obtain using the major search engines.

Here are some of the types compatible with `seq<'T>`:

- Array types: For example, `int[]` is compatible with `seq<int>`.

- F# list types: For example, `int list` is compatible with `seq<int>`.

- All other F# and .NET collection types: For example, `SortedList<string>` in the `System.Collections.Generic` namespace is compatible with `seq<string>`.

### Using Lazy Sequences from External Sources

Sequences are frequently used to represent the process of streaming data from an external source, such as from a database query or from a computer’s file system. For example, the following recursive function constructs a `seq<string>` that represents the process of recursively reading the names of all the files under a given path. The return types of `Directory.GetFiles` and `Directory.GetDirectories` are `string[]`; and, as noted earlier, this type is always compatible with `seq<string>`:

```F#
open System 
open System.IO 
let rec allFiles dir = 
    Seq.append 
        (dir |> Directory.GetFiles) 
        (dir |> Directory.GetDirectories |> Seq.map allFiles |> Seq.concat)

```

This gives the following type:

```F#
val allFiles : dir:string -> seq<string>
```

Here is an example of the function being used on one of our machines:

```F#
> allFiles Environment.SystemDirectory;;
val it : seq<string> =
  seq
    ["c:\WINDOWS\system32\12520437.cpx"; "c:\WINDOWS\system32\12520850.cpx";
     "c:\WINDOWS\system32\aaclient.dll";
     "c:\WINDOWS\system32\accessibilitycpl.dll"; ...]
```

The `allFiles` function is interesting, partly because it shows many aspects of F# working seamlessly together:

- Functions as values: The function `allFiles` is recursive and is used as a first-class function value within its own definition.

- Pipelining: The pipelining operator `|>` provides a natural way to present the transformations applied to each subdirectory name.

- Type inference: Type inference computes all types in the obvious way, without any type annotations.

- NET interoperability: The `System.IO.Directory` operations provide intuitive primitives, which can then be incorporated in powerful ways using succinct F# programs.

- Laziness where needed: The function `Seq.map` applies the argument function lazily (on demand), which means subdirectories aren’t read until required.

One subtlety with programming with on-demand or lazy values such as sequences is that side effects like reading and writing from an external store shouldn’t generally happen until the lazy sequence value is consumed. For example, the previous `allFiles` function reads the system directory as soon as `allFiles` is applied to its argument. This may not be appropriate if the contents of the system directory are changing. You can delay the computation of the sequence by using the library function `Seq.delay` or by using a sequence expression, covered in the next section, in which delays are inserted automatically by the F# compiler.

### Using Sequence Expressions

Functions are a powerful way of working with `seq<type>` values. However, F# also provides a convenient and compact syntax called *sequence expressions* for specifying sequence values that can be built using operations such as `choose`, `map`, `filter`, and `concat`. You can also use sequence expressions to specify the shapes of lists and arrays. It’s valuable to learn how to use sequence expressions for the following reasons:

- They’re a compact way of specifying interesting data and generative processes.

- They’re used to specify database queries when using data-access layers such as Microsoft’s Language Integrated Queries (LINQ). See Chapter 15 for examples of using sequence expressions this way.

- They’re one particular use of *computation expressions*, a more general concept that has several uses in F# programming. This chapter discusses computation expressions, and we will show how to use them for asynchronous and parallel programming in Chapter 13.

The simplest form of a sequence expression is `seq { for value in expr .. expr -> expr }`. Here, `->` should be read as “yield.” This is a shorthand way of writing `Seq.map` over a range expression. For example, you can generate an enumeration of numbers and their squares as follows:

```F#
> let squares = seq { for i in 0 .. 10 -> (i, i * i) };;
val squares : seq<int * int>
```

The more complete form of this construct is `seq { for pattern in sequence -> expression }`. The pattern allows you to decompose the values yielded by the input enumerable. For example, you can consume the elements of squares using the pattern `(i, iSquared)`:

```F#
> seq { for (i, iSquared) in squares -> (i, iSquared, i * iSquared) };;
val it : seq<int * int * int> =
  seq [(0, 0, 0); (1, 1, 1); (2, 4, 8); (3, 9, 27); ...]
```

The input sequence can be a `seq<type>` or any type supporting a `GetEnumerator` method.

### Enriching Sequence Expressions with Additional Logic

A sequence expression often begins with `for ... in ...`, but you can use additional constructs. For example:

- A secondary iteration: `for pattern in seq do seq-expr`

- A filter: `if expression then seq-expr`

- A conditional: `if expression then seq-expr else seq-expr`

- A let binding: `let pattern = expression in seq-expr`

- Yielding a value: `yield expression`

Secondary iterations generate additional sequences, all of which are collected and concatenated together. Filters let you skip elements that don’t satisfy a given predicate. To show both of these in action, the following computes a checkerboard set of coordinates for a rectangular grid:

```F#
let checkerboardCoordinates n =
   seq { for row in 1 .. n do
            for col in 1 .. n do
                let sum = row + col
                if sum % 2 = 0 then
                    yield (row, col)}

```

```F#
> checkerboardCoordinates 3;;
val it : seq<int * int> = seq [(1, 1); (1, 3); (2, 2); (3, 1); ...]
```

Using let clauses in sequence expressions allows you to compute intermediary results. For example, the following code gets the creation time and last-access time for each file in a directory:

```F#
let fileInfo dir =
    seq { for file in Directory.GetFiles dir do
            let creationTime = File.GetCreationTime file
            let lastAccessTime = File.GetLastAccessTime file
            yield (file, creationTime, lastAccessTime)}
```

In the previous examples, each step of the iteration produces zero or one result. The final yield of a sequence expression can also be another sequence, signified through the use of the `yield!` keyword. The following sample shows how to redefine the `allFiles` function from the previous section using a sequence expression. Note that multiple generators can be included in one sequence expression; the results are implicitly collated together using `Seq.append`:

```F#
let rec allFiles dir =
    seq { 
            for file in Directory.GetFiles dir do
                yield file
            for subdir in Directory.GetDirectories dir do
                yield! allFiles subdir}
```

### Generating Lists and Arrays Using Sequence Expressions

You can also use range and sequence expressions to build list and array values. The syntax is identical, except the surrounding braces are replaced by the usual `[ ]` for lists and `[| |]` for arrays (Chapter 4 discussed arrays in more detail):

```F#
> [1 .. 4];;
val it: int list = [1; 2; 3; 4]
> [for i in 0 .. 3 -> (i, i * i)];;
val it : (int * int) list = [(0, 0); (1, 1); (2, 4); (3, 9)]
> [|for i in 0 .. 3 -> (i, i * i)|];;
val it : (int * int) [] = [|(0, 0); (1, 1); (2, 4); (3, 9)|]
```

---

##### Caution

F# lists and arrays are finite data structures built immediately rather than on demand, so you must take care that the length of the sequence is suitable. For example, `[1I .. 1000000000I ]` attempts to build a list that is 1 billion elements long.

---

## More on Working with Sequences

In this section, we will look at more techniques for working with sequences of data. These extend the initial techniques you learned in the previous section. For example, consider the `map` and `filter` operations. You can use these operators in a straightforward manner to query and transform in-memory data. For instance, given a table representing some people in your contacts list, you can select those names that start with the letter A:

```F#
/// A table of people in our startup
let people =
    [("Amber", 27, "Design")
     ("Wendy", 35, "Events")
     ("Antonio", 40, "Sales")
     ("Petra", 31, "Design")
     ("Carlos", 34, "Marketing")]

/// Extract information from the table of people
let namesOfPeopleStartingWithA =
    people
      |> Seq.map (fun (name, age, dept) -> name)
      |> Seq.filter (fun name -> name.StartsWith "A")
      |> Seq.toList
```

At the end of the set of sequence operations, we use `Seq.toList` to evaluate the sequence once and convert the results into a concrete list. Similarly, the following finds all those in the design department:

```F#
/// Extract the names of designers from the table of people
let namesOfDesigners =
    people
      |> Seq.filter(fun (_, _, dept) -> dept = "Design")
      |> Seq.map(fun (name, _, _) -> name)
      |> Seq.toList
```

The output from evaluating these declarations is:

```F#
val people : (string * int * string) list =
  [("Amber", 27, "Design"); ("Wendy", 35, "Events"); ("Antonio", 40, "Sales");
   ("Petra", 31, "Design"); ("Carlos", 34, "Marketing")]
val namesOfPeopleStartingWithA : string list = ["Amber"; "Antonio"]
val namesOfDesigners : string list = ["Amber"; "Petra"]
```

The `map` and `filter` operations on sequences, arrays, lists, and other structured data types are examples of queries. In these cases, the queries are expressed by using pipelined combinators that accept the data structure as input. In database terminology, the map and filter operations are, respectively, called “select” and “where.” Sequence-based functional programming gives you the tools to apply in-memory query logic on all types that are compatible with the F# sequence type, such as F# lists, arrays, sequences, and anything else that implements the `IEnumerable<'a>`/`seq<'a>` interface.

In the rest of this section, you will learn about additional operations over sequences. Some further statistical and numeric operations, such as summing and averaging over sequences, will be described in Chapter 10.

### Using Other Sequence Operators: Truncate and Sort

The `Seq` module contains many other useful functions in addition to the map and filter operators, some of which were described in Chapter 3. For instance, a useful query-like function is `Seq.truncate`, which takes the first n elements of a sequence and discards the rest. Likewise, you can sort a sequence by using `Seq.sort`. In the following example, you extract the first 3000 even numbers from an unbounded stream of random numbers, and for each result you return a pair of the number and its square. The sort operation uses the default comparison semantics for the type of elements in the sequence.

```F#
/// A random-number generator
let rand = System.Random()

/// An infinite sequence of numbers
let randomNumbers = seq { while true do yield rand.Next(100000) }

/// The first 10 random numbers, sorted
let firstTenRandomNumbers =
    randomNumbers
      |> Seq.truncate 10
      |> Seq.sort // sort ascending
      |> Seq.toList

/// The first 3000 even random numbers and sort them
let firstThreeThousandEvenNumbersWithSquares =
    randomNumbers
      |> Seq.filter (fun i -> i % 2 = 0)  // "where"
      |> Seq.truncate 3000
      |> Seq.sort // sort ascending
      |> Seq.map (fun i -> i, i*i) // "select"
      |> Seq.toList
```

The output for evaluating this expression is:

```F#
// random – results will vary!
val rand : Random
val randomNumbers : seq<int>
val firstTenRandomNumbers : int list =
  [9444; 14443; 15015; 20448; 31038; 46145; 69447; 85050; 85509; 92181]
val firstThreeThousandEvenNumbersWithSquares : (int * int) list =
  [(56, 3136); (62, 3844); (66, 4356); (68, 4624); (70, 4900); (86, 7396);
   (144, 20736); (238, 56644); (248, 61504); (250, 62500); ... ]
```

The operations Seq.sortBy and Seq.sortWith take custom key-extraction and key-comparison functions, respectively. For example, the next code sample takes the first ten numbers from our random-number sequence and sorts them by the last digit only (so 17510 appears before 16351, because the last digit 0 is lower than 1):

```F#
/// The first 10 random numbers, sorted by last digit
let firstTenRandomNumbersSortedByLastDigit =
    randomNumbers
      |> Seq.truncate 10
      |> Seq.sortBy (fun x -> x % 10)
      |> Seq.toList
```

The output is:

```F#
val firstTenRandomNumbersSortedByLastDigit : int list =
  [51220; 56640; 88543; 97424; 90744; 11784; 23316; 1368; 71878; 89719]
```

### Selecting Multiple Elements from Sequences

Often, you will find yourself writing sequence transformations that select multiple elements or zero-or-one elements for each input element in the sequence. This is in contrast to the operation `Seq.map`, which always selects one new element. The types of `Seq.collect` and `Seq.choose` are:

```F#
module Seq =
    val choose : chooser : ('T -> 'U option) -> source : seq<'T> -> seq<'U>
    val collect : mapping : ('T -> #seq<'U>) -> source : seq<'T> -> seq<'U>
    val map : mapping : ('T -> 'U) -> source : seq<'T> -> seq<'U>
```

For example, given a list of numbers 1 to 10, the following sample shows how to select the “triangle” of numbers 1, 1, 2, 1, 2, 3, 1, 2, 3, 4, and so on. At the end of the set of sequence operations, we use `Seq.toList` to evaluate the sequence once and convert the results into a concrete list:

```F#
// Take the first 10 numbers and build a triangle 1, 1, 2, 1, 2, 3, 1, 2, 3, 4, ...
let triangleNumbers =
    [ 1 .. 10 ]
      |> Seq.collect (fun i -> [ 1 .. i ] )
      |> Seq.toList
```

The output is:

```F#
val triangleNumbers : int list =
  [1; 1; 2; 1; 2; 3; 1; 2; 3; 4; 1; 2; 3; 4; 5; 1; 2; 3; 4; 5; 6; 1; 2; 3; 4; 5; 6; 7; 1; 2; 3; 4; 5; 6; 7; 8; 1; 2; 3; 4; 5; 6; 7; 8; 9; 1; 2; 3; 4; 5; 6; 7; 8; 9; 10]
```

Likewise, you can use `Seq.choose` for the special case in which returning only zero-or-one elements, returning an option value `None` to indicate no new elements are produced and `Some value` to indicate the production of one element. The `Seq.choose` operation is logically the same as combining `filter` and `map` operations into a single step. In the example below, you will create an 8 x 8 “game board” full of random numbers, in which each element of the data structure includes the index position and a “game value” at that position. (Note that you will reuse this data structure later to explore several other sequence operations.) 

```F#
let gameBoard =
    [ for i in 0 .. 7 do
        for j in 0 .. 7 do
            yield (i, j, rand.Next(10)) ]
```

We then select the positions containing an even number:

```F#
let evenPositions =
   gameBoard
     |> Seq.choose (fun (i, j, v) -> if v % 2 = 0 then Some (i, j) else None)
     |> Seq.toList
```

The output is:

```F#
// random – results will vary!
val gameBoard : (int * int * int) list =  [(0, 0, 9); (0, 1, 2); (0, 2, 8); (0, 3, 2); ...]
val evenPositions : (int * int) list =  [(0, 1); (0, 2); (0, 3); ... ]
```

### Finding Elements and Indexes in Sequences

One of the most common operations in sequence programming is searching a sequence for an element that satisfies a specific criteria. The operations you use to do this are `Seq.find`, `Seq.findIndex`, and `Seq.pick`. These will raise an exception if an element or index is not found, and they also have corresponding nonfailing partners `Seq.tryFind`, `Seq.tryFindIndex`, and `Seq.tryPick`. The types of these operations are:

```F#
module Seq =
    val find : predicate : ('T -> bool) -> source : seq<'T> -> 'T
    val findIndex : predicate : ('T -> bool) -> source : seq<'T> -> int
    val pick : chooser : ('T -> 'U option) -> source : seq<'T> -> 'U
    val tryFind : predicate : ('T -> bool) -> source : seq<'T> -> 'T option
    val tryFindIndex : predicate : ('T -> bool) -> source : seq<'T> -> int option
    val tryPick : chooser : ('T -> 'U option) -> source : seq<'T> -> 'U option
```

For example, given the board you previously defined, you can search the game board for the first location with a zero game value by using `Seq.tryFind`. This will return `None` if the search fails and `Some value` if it succeeds. Likewise, you can combine a selection and projection into a single operation using `Seq.tryPick`.

```F#
let firstElementScoringZero =
   gameBoard |> Seq.tryFind (fun (i, j, v) -> v = 0)
let firstPositionScoringZero =
   gameBoard |> Seq.tryPick (fun (i, j, v) -> if v = 0 then Some(i, j) else None)

```

The output is:

```F#
// random – results will vary!
val firstElementScoringZero : (int * int * int) option = Some (1, 5, 0)
val firstPositionScoringZero : (int * int) option = Some (1, 5)
```

### Grouping and Indexing Sequences

Search operations such as `Seq.find` search the input sequence linearly, from left to right. Frequently, it will be critical to the performance of your code to index your operations over your data structures. There are numerous ways to do this, usually by building an indexed data structure, such as a `Dictionary` (see Chapter 4) or an indexed functional `map` (see Chapter 3).

In the process of building an indexed collection, you will often start with sequences of data and then group the data into buckets. Grouping is useful for many other reasons when categorizing data for structural and statistical purposes. In the following example, you will group the elements of the game board defined previously into buckets according to the game value at each position:

```F#
open System.Collections.Generic
let positionsGroupedByGameValue =
   gameBoard
    |> Seq.groupBy (fun (i, j, v) -> v)
    |> Seq.sortBy (fun (k, v) -> k)
    |> Seq.toList
```

The output is:

```F#
val positionsGroupedByGameValue : (int * seq<int * int * int>) list =
  [(0, <seq>); (1, <seq>); (2, <seq>); (3, <seq>); (4, <seq>); (5, <seq>);
   (6, <seq>); (7, <seq>); (8, <seq>); (9, <seq>)]
```

Note that each element in the grouping is a key and a sequence of values for that key. It is very common to immediately post-process the results of grouping into a dictionary or map. The built-in functions `dict` and `Map.ofSeq` are useful for this purpose, as they build dictionaries and immutable tree-maps, respectively. For example:

```F#
let positionsIndexedByGameValue =
   gameBoard
    |> Seq.groupBy (fun (i, j, v) -> v)
    |> Seq.sortBy (fun (k, v) -> k)
    |> Seq.map (fun (k, v) -> (k, Seq.toList v))
    |> dict
let worstPositions = positionsIndexedByGameValue.[0]
let bestPositions = positionsIndexedByGameValue.[9]
```


The output is: 

```F#
// random – results will vary!
val positionsIndexedByGameValue :
  IDictionary<int,(int * int * int) list>
val worstPositions : (int * int * int) list =
  [|(0, 6, 0); (2, 1, 0); (2, 3, 0); (6, 0, 0); (7, 0, 0)|]
val bestPositions : (int * int * int) list =
  [|(0, 3, 9); (0, 7, 9); (5, 3, 9); (6, 4, 9); (6, 7, 9)|]

```

Note that for highly-optimized grouping and indexing, it can be useful to rewrite your core indexed tables into direct uses of highly optimized data structures, such as `System.Collections.Generic.Dictionary`, as described in Chapter 4.

### Folding Sequences

Some of the most general functions supported by most F# data structures are `fold` and `foldBack`. These apply a function to each element of a collection and accumulate a result. For `fold` and `foldBack`, the function is applied in left-to-right or right-to-left order, respectively. If you use the name `fold`, the ordering typically is left to right. Both functions also take an initial value for the accumulator. For example:

```F#
> List.fold (fun acc x -> acc + x) 0 [4; 5; 6];;
val it : int = 15
```

These can be used with the `||>` pipelining operator, which takes two arguments in a tuple and pipes them into a function expecting two (curried) arguments:

```F#
> (0, [4;5;6]) ||> List.fold (fun acc x -> acc + x);;
val it : int = 15
> (0.0, [4.0; 5.0; 6.0]) ||> Seq.fold (fun acc x -> acc + x);;
val it : float = 15.0
> ([4; 5; 6; 3; 5], System.Int32.MaxValue) ||> List.foldBack (fun x acc -> min x acc);;
val it : int = 3
```

The following are equivalent, but no explicit anonymous function values are used:

```F#
> (0, [4; 5; 6]) ||> List.fold (+);;
val it : int = 15
> (0.0, [4.0; 5.0; 6.0]) ||> Seq.fold (+);;
val it : float = 15.0
> ([4; 5; 6; 3; 5], System.Int32.MaxValue ) ||> List.foldBack min;;
val it : int = 3
```

If used carefully, the various `foldBack` operators are pleasantly compositional, because they let you apply a selection function as part of the accumulating function:

```F#
> ([(3, "three"); (5, "five")], System.Int32.MaxValue) ||> List.foldBack (fst >> min);;
val it : int = 3

```

The F# library also includes more direct accumulation functions, such as `Seq.sum` and `Seq.sumBy`. These use a fixed accumulation function (addition) with a fixed initial value (zero), and are described in Chapter 10.

---

##### Caution 

Folding operators are very powerful and can help you avoid many explicit uses of recursion or loops in your code. they’re sometimes overused in functional programming, however, and they can be hard for novice users to read and understand. take the time to document uses of these operators, or consider using them to build simpler operators that apply a particular accumulation function.

---

### Cleaning Up in Sequence Expressions

It’s common to implement sequence computations that access external resources such as databases but that return their results on demand. This raises a difficulty: How do you manage the lifetime of the resources for the underlying operating-system connections? One elegant solution is via use bindings in sequence expressions:

- When a `use` binding occurs in a sequence expression, the resource is initialized each time a client enumerates the sequence.

- The connection is closed when the client disposes of the enumerator.

For example, consider the following function that creates a sequence expression that reads the first two lines of a file on demand:

```F#
open System.IO
let firstTwoLines file =
    seq { 
            use s = File.OpenText(file)
            yield s.ReadLine()
            yield s.ReadLine() }
```

Let’s now create a file and a sequence that reads the first two lines of the file on demand:

```F#
> File.WriteAllLines("test1.txt", [|"Es kommt ein Schiff";
"A ship is coming"|]);;
> let twolines () = firstTwoLines "test1.txt";;
val twolines : unit -> seq<string>

```

At this point, the file hasn’t yet been opened, and no lines have been read from the file. If you now iterate the sequence expression, the file is opened, the first two lines are read, and the results are consumed from the sequence and printed. Most important, the file has now also been closed, because the `Seq.iter` function is careful to dispose of the underlying enumerator it uses for the sequence, which in turn disposes of the file handle generated by `File.OpenText`:

```F#
> twolines() |> Seq.iter (printfn "line = '%s'")
line = 'Es kommt ein Schiff'
line = A ship is coming'
```

### Expressing Operations Using Sequence Expressions

Sequences written using `map`, `filter`, `choose`, and `collect` can often be rewritten to use a sequence expression. For example, the triangleNumbers and evenPositions examples can also be written as:

```F#
let triangleNumbers =
    [ for i in 1 .. 10 do
        for j in 1 .. i do
            yield (i, j) ]

let evenPositions =
    [ for (i, j, v) in gameBoard do
        if v % 2 = 0 then
            yield (i, j) ]
```

The output in each case is the same. In many cases, rewriting to use generative sequence expressions results in considerably clearer code. There are pros and cons to using sequence-expression syntax for some parts of queries:

- Sequence expressions are very good for the subset of queries expressed using iteration (for), filtering (if/then), and mapping (yield). They’re particularly good for queries containing multiple nested for statements.

- Other query constructs, such as ordering, truncating, grouping, and aggregating, must be expressed directly using aggregate operators, such as `Seq.sortBy` and `Seq.groupBy`, or by using the more general notion of “query expressions” (see Chapter 14).

- Some queries depend on the index position of an item within a stream. These are best expressed directly using aggregate operators such as `Seq.map`.

- Many queries are part of a longer series of transformations chained by |> operators. Often, the type of the data being transformed at each step varies substantially through the chain of operators. These queries are best expressed using operator chains.

---

##### Note

F# also has a more general “query-expression” syntax that includes support for specifying grouping, aggregation, and joining operations. this is mostly used for querying external data sources and will be discussed in Chapter 13.

---

## Structure beyond Sequences: Domain Modeling

In Chapters 5 and 6, you learned about some simple techniques to represent record and tree-structured data in F# using record types, union types, and object types. Together this constitutes domain modeling using types. Further, in Chapter 8, you learned how to move from an unstructured, textual representation such as XML to a domain model in the form of a structured representation. In the following sections, you will learn techniques for working with domain models.

Let’s look at the design of the type Scene from the previous chapter, in Listing 8-1, repeated below. The type Scene uses fewer kinds of nodes than the concrete XML representation from Chapter 8; the concrete XML has node kinds Circle, Square, Composite, and Ellipse, whereas Scene has just three (Rect, Ellipse, and Composite), with two derived constructors, Circle and Square, defined as static members of the Scene:

```F#
open System.Drawing

type Scene =
  | Ellipse of RectangleF
  | Rect of RectangleF
  | Composite of Scene list
  static member Circle(center:PointF,radius) =
    Ellipse(
        RectangleF(
            center.X-radius,center.Y-radius, radius*2.0f,radius*2.0f))
    /// A derived constructor
    static member Square(left,top,side) =
        Rect(RectangleF(left,top,side,side))
```

This is a common step when building a domain model; details are dropped and unified to make the domain model simpler and more general. Extra functions are then added that compute specific instances of the domain model. This approach has pros and cons:

- Manipulations are easier to program if you have fewer constructs in your domain model.

- You must be careful not to eliminate truly valuable information from a domain model. For some applications, it may really matter if the user specified a Square or a Rectangle in the original input; for example, an editor for this data may provide different options for editing these objects.

The domain model uses the types PointF and RectangleF from the System.Drawing namespace. This simplification is a design decision that should be assessed: PointF and RectangleF use 32-bit, low-precision, floating-point numbers, which may not be appropriate if you’re eventually rendering on high-precision display devices. You should be wary of deciding on domain models on the basis of convenience alone, although of course this is useful during prototyping.

The lesson here is that you should look carefully at your domain models, trimming out unnecessary nodes and unifying constructs where possible, but only so long as doing so helps you achieve your ultimate goals.

Common operations on domain models include traversals that collect information and transformations that generate new models from old ones. For example, the domain model from Listing 8-1 has the property that, for nearly all purposes, the Composite nodes are irrelevant (this wouldn’t be the case if you added an extra construct, such as an Intersect node). This means you can flatten to a sequence of Ellipse and Rectangle nodes:

```F#
let rec flatten scene =
    seq { match scene with
            | Composite scenes -> for x in scenes do yield! flatten x
            | Ellipse _ | Rect _ -> yield scene }
```

Here, flatten is defined using sequence expressions that were introduced in Chapter 3. Its type is:

```F#
val flatten : scene:Scene -> seq<Scene>
```

Let’s look at this more closely. Recall that sequences are on-demand (lazy) computations. Using functions that recursively generate `seq<'T>` objects can lead to inefficiencies in your code if your domain model is deep. It’s often better to traverse the entire tree in an eager way (eager traversals run to completion immediately). For example, it’s typically faster to use an accumulating parameter to collect a list of results. Here’s an example:

```F#
let rec flattenAux scene acc =
    match scene with
    | Composite(scenes) -> List.foldBack flattenAux scenes acc
    | Ellipse _
    | Rect _ -> scene :: acc
let flatten2 scene = flattenAux scene [] |> Seq.ofList
```

The following does an eager traversal using a local mutable instance of a ResizeArray as the accumulator and then returns the result as a sequence. This example uses a local function and ensures that the mutable state is locally encapsulated:

```F#
let flatten3 scene =
    let acc = new ResizeArray<_>()
    let rec flattenAux s =
        match s with
        | Composite(scenes) -> scenes |> List.iter flattenAux
        | Ellipse _ | Rect _ -> acc.Add s
    flattenAux scene
    Seq.readonly acc
```

The types of these are:

```F#
val flatten : scene: Scene -> seq<Scene>
val flattenAux : scene: Scene -> acc: Scene list -> Scene list
val flatten2 : scene: Scene -> seq<Scene>
val flatten3 : scene: Scene -> seq<Scene>
```

There is no hard and fast rule about which of these is best. For prototyping, the second option—doing an efficient, eager traversal with an accumulating parameter—is often the most effective. Even if you implement an accumulation using an eager traversal, however, returning the result as an on-demand sequence still can give you added flexibility later in the design process.

---

##### Note 

domain modeling with F# types is a rich topic. the F# expert Scott Wlaschin and others in the domain-driven-design community have written extensively on domain modeling techniques for real-world business problems. You can read about this at http://fsharpforfunandprofit.com. domain modeling with types is not restricted to business problems, but rather occurs under different names in almost every field of programming.

---

### Transforming Domain Models

In the previous section, you saw examples of accumulating traversals over a domain model. It’s common to traverse domain models in other ways:

- Leaf rewriting (mapping): Translating some leaf nodes of the representation but leaving the overall shape of the model unchanged

- Bottom-up rewriting: Traversing a model, but making local transformations on the way up

- Top-down rewriting: Traversing a model, but before traversing each subtree, attempting to locally rewrite the tree according to some particular set of rules

- Accumulating and rewriting transformations: For example, transforming the model left to right but accumulating a parameter along the way

For example, this mapping transformation rewrites all leaf ellipses to rectangles:

```F#
let rec rectanglesOnly scene =
    match scene with
    | Composite scenes -> Composite (scenes |> List.map rectanglesOnly)
    | Ellipse rect | Rect rect -> Rect rect
```

Often, whole classes of transformations are abstracted into aggregate transformation operations, taking functions as parameters. For example, here is a function that applies one function to each leaf rectangle:

```F#
let rec mapRects f scene =
    match scene with
    | Composite scenes -> Composite (scenes |> List.map (mapRects f))
    | Ellipse rect -> Ellipse (f rect)
    | Rect rect -> Rect (f rect)
```

The types of these functions are:

```F#
val rectanglesOnly : scene:Scene -> Scene
val mapRects : f:(RectangleF -> RectangleF) -> scene:Scene -> Scene
```

Here is a use of the `mapRects` function that adjusts the aspect ratio of all the RectangleF values in the scene (RectangleF values support an Inflate method):

```F#
let adjustAspectRatio scene =
    scene |> mapRects (fun r -> RectangleF.Inflate(r, 1.1f, 1.0f / 1.1f))
```

## Using On-Demand Computation with Domain Models

Sometimes it’s feasible to delay loading or processing some portions of a domain model. For example, imagine that the XML for the small geometric language from the previous section included a construct such as the following, in which the File nodes represent entire subtrees defined in external files:

```xml
<Composite>
     <File file='spots.xml'/>
     <File file='dots.xml'/>
</Composite>
```

It may be useful to delay loading these files. One general way to do this is to add a Delay node to the Scene type:

```F#
type Scene =
    | Ellipse of RectangleF
    | Rect of RectangleF
    | Composite of Scene list
    | Delay of Lazy<Scene>
```

You can then extend the extractScene function of Listing 8-1 with the following code to handle this node:

```F#
let rec extractScene (node : XmlNode) =
  let attribs = node.Attributes
  let childNodes = node.ChildNodes
  match node.Name with
  | "Circle" ->
    ...
  | "File" ->
    let file = attribs.GetNamedItem("file").Value
    let scene = lazy (let d = XmlDocument()
                        d.Load(file)
                        extractScene(d :> XmlNode))
    Scene.Delay scene
```

Code that analyzes domain models (for example, via pattern matching) must typically be adjusted to force the computation of delayed values. One way to handle this is to first call a function to eliminate immediately delayed values:

```F#
let rec getScene scene =
  match scene with
  | Delay d -> getScene (d.Force())
  | _ -> scene

```

Here is the function flatten2 from the “Processing Domain Models” section, but redefined to first eliminate delayed nodes:

```F#
let rec flattenAux scene acc =
  match getScene(scene) with
  | Composite scenes -> List.foldBack flattenAux scenes acc
  | Ellipse _ | Rect _ -> scene :: acc
  | Delay _ -> failwith "this lazy value should have been eliminated by getScene"
let flatten2 scene = flattenAux scene []

```

It’s generally advisable to have a single representation of laziness within a single domain model design. For example, the following abstract syntax design uses laziness in too many ways:

```F#
type SceneVeryLazy =
  | Ellipse of Lazy<RectangleF>
  | Rect of Lazy<RectangleF>
  | Composite of seq<SceneVeryLazy>
  | LoadFile of string

```

The shapes of ellipses and rectangles are lazy computations; each Composite node carries a `seq<SceneVeryLazy>` value to compute subnodes on demand, and a LoadFile node is used for delayed file loading. This is a bit of a mess, because a single Delay node would, in practice, cover all these cases.

---

##### Note 

the Lazy<'T>type is defined in System and represents delayed computations. You access a lazy value via the Value property. F# includes the special keyword lazy for constructing values of this type. Chapter 8 also covered lazy computations.

---

### Caching Properties in Domain Models

For high-performance applications of domain models, it can occasionally be useful to cache computations of some derived attributes within the model itself. For example, let’s say you want to compute bounding boxes for the geometric language described in Listing 9-1. It’s potentially valuable to cache this computation at Composite nodes. You can use a type such as the following to hold a cache:

```F#
type SceneWithCachedBoundingBox =
  | EllipseInfo of RectangleF
  | RectInfo of RectangleF
  | CompositeInfo of CompositeScene
and CompositeScene =
  { Scenes: SceneWithCachedBoundingBox list
  mutable BoundingBoxCache : RectangleF option  }

```

This is useful for prototyping, although you should be careful to encapsulate the code that is responsible for maintaining this information. Listing 9-1 shows the full code for doing this.

Listing 9-1. Adding the cached computation of a local attribute to a domain model

```F#
type CompositeInfo =
  { Scenes: SceneWithCachedBoundingBox list
  mutable BoundingBoxCache : RectangleF option  }
and SceneWithCachedBoundingBox =
  | EllipseInfo of RectangleF
  | RectInfo of RectangleF
  | CompositeInfo of CompositeInfo
  member x.BoundingBox =
match x with
| EllipseInfo rect | RectInfo rect -> rect
| CompositeInfo info ->
match info.BoundingBoxCache with
| Some v -> v
| None ->
let bbox =
info.Scenes
|> List.map (fun s -> s.BoundingBox)
|> List.reduce (fun r1 r2 -> RectangleF.Union(r1, r2))
info.BoundingBoxCache <- Some bbox
bbox
  /// Create a Composite node with an initially empty cache
  static member Composite scenes = CompositeInfo { Scenes=scenes; BoundingBoxCache=None }
  static member Ellipse rect = EllipseInfo rect
  static member Rect rect = CompositeInfo rect
```

Other attributes that are sometimes cached include the hash values of tree-structured terms and the computation of all the identifiers in a subexpression. The use of caches makes it more awkward to pattern-match on terms. This issue can be largely solved by using active patterns, covered later in this chapter.

### Memoizing Construction of Domain Model Nodes

In some cases, domain model nodes can end up consuming significant portions of the application’s memory budget. In this situation, it can be worth memoizing some or all of the nodes constructed in the model. You can even go as far as memoizing all equivalent nodes, ensuring that equivalence between nodes can be implemented by pointer equality, a technique often called hash-consing. Listing 9-2 shows a domain model for propositional logic terms that ensures that any two nodes that are syntactically identical are shared via a memoizing table. Propositional logic terms are terms constructed using P AND Q, P OR Q, NOT P, and variables a, b, and so on. A noncached version of the expressions is:

```F#
type Prop =
    | And of Prop * Prop
    | Or of Prop * Prop
    | Not of Prop
    | Var of string
    | True
```

Listing 9-2.  Memoizing the construction of domain model nodes

```F#

type Prop =
    | Prop of int
and PropRepr =
    | AndRepr of Prop * Prop
    | OrRepr of Prop * Prop
    | NotRepr of Prop
    | VarRepr of string
    | TrueRepr

open System.Collections.Generic

module PropOps =
    let uniqStamp = ref 0
    type internal PropTable() =
let fwdTable = new Dictionary<PropRepr, Prop>(HashIdentity.Structural)
let bwdTable = new Dictionary<int, PropRepr>(HashIdentity.Structural)
member t.ToUnique repr =
if fwdTable.ContainsKey repr then fwdTable.[repr]
else let stamp = incr uniqStamp; !uniqStamp
let prop = Prop stamp
fwdTable.Add (repr, prop)
bwdTable.Add (stamp, repr)
prop
member t.FromUnique (Prop stamp) =
bwdTable.[stamp]
 let internal table = PropTable ()
    // Public construction functions
    let And (p1, p2) = table.ToUnique (AndRepr (p1, p2))
    let Not p = table.ToUnique (NotRepr p)
    let Or (p1, p2)  = table.ToUnique (OrRepr (p1, p2))
    let Var p = table.ToUnique (VarRepr p)
    let True = table.ToUnique TrueRepr
    let False = Not True
    // Deconstruction function
    let getRepr p = table.FromUnique p
```

You construct terms using the operations in PropOps much as you would construct terms using the 
nonmemoized representation:

```F#

> open PropOps;;
> True;;
val it : Prop = Prop 1
> let prop = And (Var "x",Var "y");;
val prop  : Prop = Prop 5
> let repr = getRepr prop;;
val repr : PropRepr = AndRepr (Prop 3, Prop 4)
> let prop2 = And (Var "x",Var "y");;
val prop2 : Prop = Prop 5
```


In this example, when you create two trees using the same specification, And (Var "x",Var "y"), you get back the same Prop object with the same stamp 5. You can also use memoization techniques to implement interesting algorithms; in Chapter 12, you will see an important representation of propositional logic called a binary decision diagram (BDD) that is based on a memoization table similar to that in the previous example.

The use of unique integer stamps and a lookaside table in the previous representation also has some drawbacks; it’s harder to pattern match on abstract syntax representations, and you may need to reclaim and recycle stamps and remove entries from the lookaside table if a large number of terms is created or if the overall set of stamps must remain compact. You can solve the first problem by using active patterns, which will be covered next. If necessary, you can solve the second problem by scoping stamps in an object that encloses the uniqStamp state, the lookaside table, and the construction functions. Alternatively, you can explicitly reclaim the stamps by using the IDisposable idiom described in Chapter 6, although this approach can be intrusive to your application.