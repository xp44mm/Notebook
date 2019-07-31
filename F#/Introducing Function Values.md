## Introducing Function Values

This section will cover the foundational building block of F# functional programming: function values. We will begin with a simple and well-known example: using function values to transform one list into another.

One of the primary uses of F# lists is as a general-purpose, concrete data structure for storing ordered input lists and ordered results. Input lists are often transformed into output lists using collection functions that transform, select, filter, and categorize elements of the list according to a range of criteria. These collection functions provide an excellent introduction to how to use function values. Let’s take a closer look at this in the following code sample, which continues from the definition of http from Listing 2-2 in Chapter 2:

```F#
> let sites = ["http://www.bing.com"; "http://www.google.com"];;
val sites : string list = ["http://www.bing.com"; "http://www.google.com"]
> let fetch url = (url, http url);;
val fetch : url:string -> string * string
> List.map fetch sites;;
val it : (string * string) list =
  [("http://www.bing.com",
    "<!DOCTYPE html PUBLIC ...
</body></html>");
   ("http://www.google.com",
    "<!doctype html>...
</script>")]
```

The first interaction defines sites as a literal list of URLs, and the second defines the function `fetch`. The third calls the collection function `List.map`. This accepts the function value `fetch` as the first argument and the list sites as the second argument. The function applies `fetch` to each element of the list and collects the results in a new list.

Types are one useful way to help learn what a function does. Here’s the type of `List.map`:

```F#
val map : mapping: ('T -> 'U) -> list: 'T list -> 'U list
```

This says `List.map` accepts a function value as the first argument and a list as the second argument, and it returns a list as the result. The function argument can have any type `'T -> 'U`, and the elements of the input list must have a corresponding type `'T`. The symbols `'T` and `'U` are called type parameters, and functions that accept type parameters are called generic. Chapter 5 will discuss type parameters in detail.

---

##### Tip

You can often deduce the behavior of a function from its type, especially if its type involves type parameters. For example, look at the type of `List.map`. using type parameters, you can observe that the type `'T list` of the input list is related to the type `'T` accepted by the function passed as the first parameter. Similarly, the type `'U` returned by this function is related to the type `'U` list of the value returned by `List.map`. From this, it’s reasonable to conclude that `List.map` calls the function parameter for items in the list and constructs its result using the values returned.

---

### Using Function Values

Function values are so common in F# programming that it’s convenient to define them without giving them names. Here is a simple example:

```F#
> let primes = [2; 3; 5; 7];;
val primes : int list = [2; 3; 5; 7]
> let primeCubes = List.map (fun n -> n * n * n) primes;;
val primeCubes: int list = [8; 27; 125; 343]
```

The definition of `primeCubes` uses the *lambda function* or *anonymous function value* (`fun n -> n * n * n`). Such values are similar to function definitions but are unnamed and appear as an expression rather than as a let declaration. `fun` is a keyword meaning function, n represents the argument to the function, and `n * n * n` is the result of the function. The overall type of the anonymous function expression is `int -> int`. You could use an anonymous function instead of the intermediary function `fetch` in the earlier sample:

```F#
let resultsOfFetch = List.map (fun url -> (url, http url)) sites
```

You will see anonymous functions throughout this book. Here is another example:

```F#
> List.map (fun (_,p) -> String.length p) resultsOfFetch;;
val it : int list = [56601; 52321]
```

Here you see two things:

- The argument of the anonymous function is a tuple pattern. Using a tuple pattern automatically extracts the second element from each tuple and gives it the name p within the body of the anonymous function.

- Part of the tuple pattern is a wildcard pattern, indicated by an underscore. This indicates that you don’t care what the first part of the tuple is; you’re interested only in extracting the length from the second part of the pair.

### Computing with Collection Functions

Functions such as `List.map` are called combinators or collection functions, and they’re powerful constructs, especially when combined with the other features of F#. Here is a longer example that uses the collection functions `Array.filter` and `List.map` to count the number of URL links in an HTML page and then collects stats on a group of pages (this sample uses the function http defined in Chapter 2):

```F#
let delimiters = [| ' '; '\n'; '\t'; '<'; '>'; '=' |]
let getWords (s: string) = s.Split delimiters
let getStats site =
    let url = "http://" + site
    let html = http url
    let hwords = html |> getWords
    let hrefs = html |> getWords |> Array.filter (fun s -> s = "href")
    (site, html.Length, hwords.Length, hrefs.Length)
```

Here, you use the function `getStats` with three web pages:

```F#
> let sites = ["www.bing.com"; "www.google.com"; "search.yahoo.com"];;
val sites : string list
> sites |> List.map getStats;;
val it : (string * int * int * int) list =
    [("www.bing.com", 56601, 3230, 30);
     ("www.google.com", 52314, 2975, 31);
     ("search.yahoo.com", 17691, 1568, 40)]
```

The function `getStats` computes the length of the HTML for the given website, the number of words in the text of that HTML, and the approximate number of links on that page.

The previous code sample extensively uses the `|>` operator to pipeline operations, discussed in “Pipelining with |>” (see sidebar). The F# library design ensures that a common, consistent set of collection functions is defined for each structured type. Table 3-7 shows how the same convention is used for the map abstraction.

Table 3-7. A Recurring Collection Function Design Pattern from the F# Library

```F#
  List.map : ('T -> 'U) -> 'T list -> 'U list
 Array.map : ('T -> 'U) -> 'T [] -> 'U []
Option.map : ('T -> 'U) -> 'T option -> 'U option
   Seq.map : ('T -> 'U) -> seq<'T> -> seq<'U>
```

### Using Fluent Notation on Collections

Some programmers prefer to use “fluent” notation for collection functions, using notation such as

```F#
sites.map(fun x -> ...).sort()
```

rather than

```F#
xs |> List.map(fun x -> ...)  |> List.sort
```

For example, you may use

```F#
sites.map(getStats)
```

to replace

```F#
sites |> List.map getStats
```

This option is available through the nuget package `FSharp.Core.Fluent` which you can add to your script and/or project through the techniques described in Chapter 2. This provides a slightly more succinct way of transforming data through a set of (lowercase named) instance extension members on `List`, `Array`, and `seq` types. Chapter 6 will discuss extension members in detail. This style may require the use of a type annotation. In general, both the “pipelining” and the “fluent” styles are used in F# programming, depending on the methods and properties available for a particular data type. For largely historical reasons the fluent style is not used particularly often with the collection types `List`, `Array`, and `seq`. However, the package `FSharp.Core.Fluent` does enable it for those who wish to adopt it for consistency reasons.

---

##### pipelining with |>

the |> forward pipe operator is perhaps the most important operator in F# programming. Its definition is deceptively simple:

```F#
let (|>) x f = f x

```

here is how to use the operator to compute the cubes of three numbers:

```F#
[1;2;3] |> List.map (fun x -> x * x * x)
```


this produces `[1; 8; 27]`, just as if you had written:

```F#
List.map (fun x -> x * x * x) [1; 2; 3]
```


In a sense, |> is function application in reverse. however, using |> has distinct advantages:

- Clarity: When used in conjunction with functions such as `List.map`, the |> operator allows you to perform the data transformations and iterations in a forward-chaining, pipelined style.

- Type inference: using the |> operator lets type information flow from input objects to the functions manipulating those objects. F# uses information collected from type inference to resolve some language constructs, such as property accesses and method overloading. this relies on information being propagated left to right through the text of a program. In particular, typing information to the right of a position isn’t taken into account when resolving property access and overloads.

For completeness, here is the type of the operator:

```F#
val (|>) : 'T -> ('T -> 'U) -> 'U
```

---

### Composing Functions with >>

You saw earlier how to use the |> forward pipe operator to pipe values through a number of functions. This was a small example of the process of computing with functions, an essential and powerful programming technique in F#. This section will cover ways to compute new function values from existing ones using compositional techniques. First, let’s look at function composition. For example, consider the following code:

```F#
let google = http "http://www.google.com"
google |> getWords |> Array.filter (fun s -> s = "href") |> Array.length
let countLinks = getWords >> Array.filter (fun s -> s = "href") >> Array.length
google |> countLinks
```

You define countLinks as the composition of three function values using the >> forward composition operator. This operator is defined in the F# library, as follows:

```F#
let (>>) f g x = g(f(x))
```

You can see from the definition that `f >> g` gives a function value that first applies f to the x and then applies g. Here is the type of >>:

```F#
val (>>) : ('T -> 'U) -> ('U -> 'V) -> ('T -> 'V)
```

Note that >> has only two arguments, here named f and g. The operator takes two functions and returns a function.

F# is good at optimizing basic constructions of pipelines and composition sequences from functions—for example, the function `countLinks` shown earlier becomes a single function that directly calls the three functions in the pipeline in sequence. This means sequences of compositions can be used with relatively low overhead.

### Building Functions with Partial Application

Composing functions is just one way to compute interesting new functions. Another useful way is to use partial application. Here’s an example, with x and y in Cartesian coordinates:

```F#
let shift (dx, dy) (px, py) = (px + dx, py + dy)
let shiftRight = shift (1, 0)
let shiftUp = shift (0, 1)
let shiftLeft = shift (-1, 0)
let shiftDown = shift (0, -1)
```

The last four functions are defined by calling shift with only one argument, in each case leaving a residue function that expects an additional argument. F# Interactive reports the types as follows:

```F#
val shiftRight : (int * int -> int * int)
val shiftUp : (int * int -> int * int)
val shiftLeft : (int * int -> int * int)
val shiftDown : (int * int -> int * int)
```

Here is an example of how to use shiftRight and how to partially apply shift to new arguments (2, 2):

```F#
> shiftRight (10, 10);;
val it : int * int = (11, 10)
> List.map (shift (2,2)) [(0,0); (1,0); (1,1); (0,1)];;
val it : (int * int) list = [(2, 2); (3, 2); (3, 3); (2, 3)]
```

In the second example, the function `shift` takes two pairs as arguments. You bind the first parameter to (2, 2). The result of this partial application is a function that takes one remaining tuple parameter and returns the value shifted by two units in each direction. This resulting function can now be used in conjunction with `List.map`.

### Using Local Functions

Partial application is one way in which functions can be computed rather than simply defined. This technique becomes very powerful when combined with additional local definitions. Here’s a simple and practical example, representing an idea common in graphics programming:

```F#
open System.Drawing

let remap (r1:RectangleF) (r2:RectangleF) =
    let scalex = r2.Width / r1.Width
    let scaley = r2.Height / r1.Height
    let mapx x = r2.Left + (x - r1.Left) * scalex
    let mapy y = r2.Top + (y - r1.Top) * scaley
    let mapp (p:PointF) = PointF(mapx p.X, mapy p.Y)
    mapp
let rect1 = RectangleF(100.0f, 100.0f, 100.0f, 100.0f)
let rect2 = RectangleF(50.0f, 50.0f, 200.0f, 200.0f)
let mapp = remap rect1 rect2
```

The function `remap` computes a new function value `mapp` that maps points in one rectangle to points in another. F# Interactive reports the type as follows:

```F#
val remap : r1:RectangleF -> r2:RectangleF -> (PointF -> PointF)
val mapp : (PointF -> PointF)
```

The type `Rectangle` is defined in the library `System.Drawing.dll` and represents rectangles specified by integer coordinates. The computations on the interior of the transformation are performed in floating point to improve precision. You can use this as follows:

```F#
> mapp (PointF(100.0f, 100.0f));;
val it : PointF = {X=50, Y=50}
> mapp (Point(150.0f, 150.0f));;
val it : PointF = {X=150, Y=150}
> mapp (Point(200.0f, 200.0f));;
val it : PointF = {X=200, Y=200}

```

The intermediate values `scalex` and `scaley` are computed only once, despite the fact that you’ve called the resulting function `mapp` three times.

In the previous example, `mapx`, `mapy`, and `mapp` are local functions—functions defined locally as part of the implementation of `remap`. Local functions can be context dependent; in other words, they can be defined in terms of any values and parameters that happen to be in scope. Local functions are said to capture the values they depend on. Here, `mapx` is defined in terms of `scalex`, `scaley`, `r1`, and `r2` and captures all of these values.

---

##### Note

local and partially applied functions are, if necessary, implemented by taking the closure of the variables they depend on and storing them away until needed. In optimized F# code, the F# compiler often avoids this and instead passes extra arguments to the function implementations. Closure is a powerful technique that is used frequently in this book. It’s often used in conjunction with functions, as in this chapter, but it is also used with object expressions, sequence expressions, and class definitions.

---

### Iterating with Functions

It’s common to use data to drive control. In functional programming, the distinction between data and control is often blurred: function values can be used as data, and data can influence control flow. One example is using a function such as `List.iter` to iterate over a list:

```F#
let sites = ["http://www.bing.com"; "http://www.google.com"; "http://search.yahoo.com"]
sites |> List.iter (fun site -> printfn "%s, length = %d" site (http site).Length)
```

The function `List.iter` simply calls the given function (here an anonymous function) for each element in the input list. Some additional iteration techniques are available in F#, particularly by using values of type `seq<type>`, which will be discussed in “Getting Started with Sequences” later in this chapter.

### Abstracting Control with Functions

As a second example of how you can abstract control using functions, let’s consider the common pattern of timing the execution of an operation (measured in wall-clock time). First, let’s explore how to use `System.DateTime.Now` to get the wall-clock time:

```F#
> open System;;
> let start = DateTime.Now;;
val start : DateTime = 13/06/2015 9:54:36 p.m.
> http "http://www.newscientist.com";;
val it : string = "<!DOCTYPE html...</html>"
> let finish = DateTime.Now;;
val finish : DateTime = 13/06/2015 9:54:39 p.m.
> let elapsed = finish - start;;
val elapsed : TimeSpan = 00:00:01.1550660

```

Note that the type `TimeSpan` has been inferred from the use of the overloaded operator in the expression `finish - start`. Chapter 6 will discuss overloaded operators in depth. You can now wrap up this technique as a function time that acts as a new control operator:

```F#
open System
let time f =
    let start = DateTime.Now
    let res = f()
    let finish = DateTime.Now
    (res, finish - start)
```

This function runs the input function f but takes the time on either side of the call. It then returns both the result of the function and the elapsed time. The inferred type is as follows:

```F#
val time : f:(unit -> 'a) -> 'a * TimeSpan
```

Here `'a` is a *type variable* that stands for any type, and thus the function can be used to time functions that return any kind of result. Note that F# automatically infers a generic type for the function, a technique called *automatic generalization* that lies at the heart of F# type inference. Chapter 5 will discuss automatic generalization in detail. Here is an example of using the time function, which again reuses the http function defined in Chapter 2:

```F#
> time (fun () -> http "http://www.newscientist.com");;
val it : string * TimeSpan = ...   (The HTML text and time will be shown here)
```

### Using Object Methods as First-Class Functions

You can use existing .NET methods as first-class functions. You will learn more about methods in Chapter 6. You can use both static and instance methods as first-class values. The following uses the .NET method `System.IO.Path.GetExtension` as a first-class method:

```F#
open System.IO
[ "file1.txt"; "file2.txt"; "file3.sh" ]
    |> List.map Path.GetExtension
```

This gives the result:

```F#
val it : string list = [".txt"; ".txt"; ".sh"]
```

Sometimes you need to add extra type information to indicate which overload of the method is required. Chapter 6 will discuss method overloading in more detail. For example, the following causes an error:

```F#
> open System;;
> let f = Console.WriteLine;;
error FS0041: A unique overload for method 'WriteLine' could not be determined based on 
type information prior to this program point. A type annotation may be needed.
```

However, the following succeeds:

```F#
> let f = (Console.WriteLine : string -> unit);;
val f : (string -> unit)
```

### Some Common Uses of Function Values

The function `remap` from the previous section generates values of type `PointF -> PointF`, representing “transformations for points.” You haven’t needed to define a new type for “transformations”—you just use functions as a way of modeling transformations. Many useful concepts can be modeled using function types and values. For example:

- Actions. The type `unit -> unit` can be used to model actions—operations that run and perform some unspecified side effect. For example, consider the expression (`fun () -> printfn "Hello World"`).

- Predicates. Types of the form `type -> bool` can be used to model predicates, which return true for a subset of values

- Counting Functions. Types of the form `type -> int` can be used to model counting functions, which give a number for each input; the numbers may then be accumulated.

- Statistical Functions. Types of the form `type -> float` can be used to model statistical functions, which give a floating point number for each input. These may be used as an input into a statistical routine such `Seq.averageBy`.

- Key Functions. Types of the form `type -> keytype` can be used to model key functions, which give a key value for each input. For example, a key function may be an integer associated with each input. Operations such as `Seq.groupBy` and `Seq.sortBy` accept key functions.

- Orderings. Types of the form `type -> type -> int` can be used to model comparison functions over the type `type`. You also see `type * type -> int` used for this purpose, where a tuple is accepted as the first argument. In both cases, a negative result indicates less than, zero indicates equals, and a positive result indicates greater than. Operations such as `Seq.sortWith` accept ordering functions.

- Callbacks. Types of the form `type -> unit` can be used to model callbacks. Callbacks are often run in response to a system event, such as when a user clicks a user interface element. In this case, the parameter sent to the handler has some specific type, such as `System.EventArgs`.

- Delayed Computations. Types of the form `unit -> type` can be used to model delayed computations, which are values that, when required, produce a value of type `type`. For example, a threading library can accept such a function value and execute it on a different thread, eventually producing a value of type `type`. Delayed computations are related to lazy computations and sequence expressions; these are discussed in “Using Sequence Expressions” in Chapter 9.

- Sinks. Types of the form `type -> unit` can be used to model sinks, which are function values that, when required, consume a value of type `type`. For example, a logging API may use a sink to consume values and write them to a log file.

- Generators. Types of the form `int -> type` can be used to model generators, which are used to initialize a collection. The parameter may be an integer index into the collection. For example, `Array.init` accepts a generator function. More complex generators may propagate state and optionally indicate the end of the generation process. For an example, see `Seq.unfold`.

- Binary Operators. Types of the form `type -> type -> type` can be used to model binary operations, which take two input values and combine them into one. Operations such as `List.reduce` accept binary operators.

- Transformations. Types of the form `type1 -> type2` can be used to model transformers, which are functions that transform each element of a collection. You see this pattern in the map operator for many common collection types.

- Accumulators. Types of the form `type1 -> type2 -> type2` can be used to model visitor accumulating functions, which are functions that visit each element of a collection (type `type1`) and accumulate a result (type `type2`). For example, a visitor function that accumulates integers into a set of integers has type `int -> Set<int> -> Set<int>`.

---

##### Note

the power of function types to model many different concepts is part of the enduring appeal of functional programming. this is one of the refreshing features F# brings to programming: many simple abstractions are modeled in very simple ways and often have simple implementations through orthogonal, unified constructs, such as anonymous function values and function compositions.

---
