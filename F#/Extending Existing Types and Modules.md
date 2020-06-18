# Extending Existing Types and Modules

The final topic covered in this chapter is how you can define ad hoc dot-notation extensions to existing library types and modules. This technique is used less commonly than the others in this chapter but can be invaluable in certain circumstances. For example, the following definition adds the member `IsPrime` to `Int32`.

```F#
module NumberTheoryExtensions =
    let factorize i =
        let lim = int (sqrt (float i))
        let rec check j =
        if j > lim  then None
        elif (i %  j) = 0 then Some (i / j, j)
        else check (j + 1)
        check 2
    type System.Int32 with
        member i.IsPrime = (factorize i).IsNone
        member i.TryFactorize() = factorize i
```

The `IsPrime` extension property and the `TryFactorize` extension method are then available for use in conjunction with int32 values whenever the `NumberTheoryExtensions` module has been opened. For example:

```F#
> open NumberTheoryExtensions;;
> (2 + 1).IsPrime;;
  val it : bool = true
> (6093704 + 11).TryFactorize();;
  val it : (int * int) option = Some (1218743, 5)
```

These type extensions are called F#-style extension members. Since F# 3.1, an additional kind of extension member is supported called a C#-style extension member. These can be declared in other .NET languages and then accessed by opening a namespace. They can also be declared in F# code. C#-style extension members can only be instance methods; i.e., they can’t be static and can’t be properties.

```F#
module CSharpStyleExtensions =
  open System.Runtime.CompilerServices
  let factorize i =
    let lim = int (sqrt (float i))
    let rec check j =
    if j > lim  then None
    elif (i %  j) = 0 then Some (i / j, j)
    else check (j + 1)
    check 2

  [<Extension>]
  type Int32Extensions() =

    [<Extension>]
    static member IsPrime2(i:int) = (factorize i).IsNone

    [<Extension>]
    static member TryFactorize2(i:int) = factorize i

  [<Extension>]
  type ResizeArrayExtensions() =
    [<Extension>]
    static member Product(values:ResizeArray<int>) =
        let mutable total = 1
        for v in values do
        total <- total * v
        total
    [<Extension>]
    static member inline GenericProduct(values:ResizeArray<'T>) =
        let mutable total = LanguagePrimitives.GenericOne<'T>
        for v in values do
        total <- total * v
        total
```

C#-style extension members are declared as an (attributed) static method in an (attributed) class. The method takes an extra “this” argument. C#-style extension members are used as an instance method taking one fewer parameters. At the usage site they must minimally take at least “zero” arguments through a () parameter. For example:

```F#
> open CSharpStyleExtensions ;;
> (2 + 1).IsPrime2();;
  val it : bool = true
> (6093704 + 11).TryFactorize2();;
  val it : (int * int) option = Some (1218743, 5)
```

Despite the limitations of C#-style extension members, they have an important advantage that is useful for some F# API designs: for generic types, C#-style extension methods can constrain the generic type parameters to either a particular instantiation or some other generic constraint. For example, in the code just reviewed, the `Product` method constrains the type of the input `ResizeArray` to be `int`. Likewise, `GenericProduct` constrains the `ResizeArray` to be a type `'T`, which support zero and multiplication (see Chapter 5 for more discussion on generic constraints). Normal F# extensions can’t operate on constrained types like this, which sometimes makes a mix of F# and C# extensions useful when designing “Fluent” APIs. For example, see the F# community library FSharp.Core.Fluent on GitHub, which uses exactly such a mix.

```F#
> open System.Collections.Generic;;
> let arr = ResizeArray([ 1 .. 10 ]);;
val arr = ResizeArray<int>
> let arr2 = ResizeArray([ 1L .. 10L ]);;
val arr2 = ResizeArray<int64>
> arr.Product();;
val it : int = 3628800
> arr.GenericProduct();;
val it : int = 3628800
> arr2.GenericProduct();;
val it : int64 = 3628800L
```

Type extensions can be given in any assembly, but priority is always given to the intrinsic members of a type when resolving dot-notation.

Modules can also be extended, in a fashion. For example, say you think the List module is missing an obvious function such as `List.pairwise` to return a new list of adjacent pairs. You can extend the set of values accessed by the path `List` by defining a new module `List`:

```F#
module List =
    let rec pairwise l =
        match l with
        | [] | [_] -> []
        | h1 :: ((h2 :: _) as t) -> (h1, h2) :: pairwise t
```

```F#
> List.pairwise [1; 2; 3; 4];;
val it : (int * int) list = [(1,2); (2,3);  (3,4)]
```

■ Note 

type extensions are a good technique for equipping simple type definitions with extra functionality. however, don’t fall into the trap of adding too much functionality to an existing type via this route. instead, it’s often simpler to use additional modules and types. For example, the module FSharp.Collections.List contains extra functionality associated with the F# list type.

USING MODULES AND TYPES TO ORGANIZE CODE

You often have to choose whether to use modules or object types to organize your code. here are some of the rules for using these to organize your code effectively and to lay the groundwork for applying good .net library and framework design principles to your code:

• Use modules when prototyping and to organize scripts, ad hoc algorithms, initialization code, and active patterns.

• Use concrete types (records, discriminated unions, and class types) to implement concrete data structures. in the long term, plan on completely hiding the implementation of these types. You will see how to do this in Chapter 7. You can provide dot-notation operations to help users to access parts of the data structure. avoid revealing other representation details.

• Use object interface types for types that have several possible implementations.

• implement object interface types by private concrete types or by object expressions.

• in polished libraries, most concrete types exposed in an implementation should also implement one or more object interface types. For example, collections should implement `IEnumerable<'T>`, and many types should implement `IDisposable`.

• avoid relying on or revealing complex type hierarchies. in particular, avoid relying on implementation inheritance, except as an internal implementation technique or when doing GUI programming or authoring very large objects.

• avoid nesting modules or types inside other modules or types, especially in public APIs. nested modules and types are useful implementation details, but they’re rarely made public in APIs. Deep hierarchical organization can be confusing; when you’re designing a library, you should place nearly all public modules and types immediately inside a well-named namespace.