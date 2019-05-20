## Equality, Hashing, and Comparison

Many efficient algorithms over structured data are built on primitives that efficiently compare and hash representations of information. In Chapter 5, you saw a number of predefined generic operations, including generic comparison, equality, and hashing, accessed via functions such as:

```F#
val compare : 'T -> 'T -> int when 'T : comparison
val (=) : 'T -> 'T -> bool when 'T : equality
val (<) : 'T -> 'T -> bool when 'T : comparison
val (<=) : 'T -> 'T -> bool when 'T : comparison
val (>) : 'T -> 'T -> bool when 'T : comparison
val (>=) : 'T -> 'T -> bool when 'T : comparison
val min : 'T -> 'T -> 'T when 'T : comparison
val max : 'T -> 'T -> 'T when 'T : comparison
val hash : 'T -> int when 'T : equality
```

First, note that these are *generic* operations—they can be used on objects of many different types. This can be seen by the use of `'T` in the signatures of these operations. The operations take one or two parameters of the same type. For example, you can apply the `=` operator to two `Form` objects, or two `System.DateTime` objects, or two `System.Type` objects, and something reasonable happens. Some other important derived generic types, such as the immutable (persistent) `Set<_>` and `Map<_,_>` types in the F# library, also use generic comparison on their key type:

```F#
type Set<'T when 'T : comparison> = ...
type Map<'Key, 'Value when 'Key : comparison> = ...
```

These operations and types are all *constrained*, in this case by the `equality` and/or `comparison` constraints. The purpose of constraints on type parameters is to make sure the operations are used only on a particular set of types. For example, consider equality and ordered comparison on a `System.Net.WebClient` object. Equality is permitted, because the default for nearly all .NET object types is reference equality:

```F#
let client1 = new System.Net.WebClient()
let client2 = new System.Net.WebClient()
client1 = client1 // true
client1 = client2 // false
```

Ordered comparison isn't permitted, however:

```F#
> client1 <= client2;;
error FS0001: The type 'System.Net.WebClient' does not support the 'comparison' constraint. For example, it does not support the 'System.IComparable' interface
```

That's good! There is no natural ordering for `WebClient` objects, or at least no ordering is provided by the .NET libraries.

Equality and comparison can work over the structure of types. For example, you can use the equality operators on a tuple only if the constituent parts of the tuple also support equality. This means that using equality on a tuple of `WebClient` objects is permitted:

```F#
> (client1, client2) = (client1, client2);;
val it : bool =  true
> (client1, client2) = (client2, client1);;
val it : bool = false
```

But using ordered comparison of a tuple isn't:

```F#
> (client1, "Data for client 1") <= (client2, " Data for client 2");;
error FS0001: The type 'System.Net.WebClient' does not support the 'comparison' constraint. For example, it does not support the 'System.IComparable' interface
```

Again, that's good—this ordering would be a bug in your code. Now, let's take a closer look at when `equality` and `comparison` constraints are satisfied in F#.

* The `equality` constraint is satisfied if the type definition doesn't have the `NoEquality` attribute, and any dependencies also satisfy the `equality` constraint.

* The `comparison` constraint is satisfied if the type definition doesn't have the `NoComparison` attribute, the type definition implements `System.IComparable`, and any dependencies also satisfy the `comparison` constraint.

An `equality` constraint is relatively weak, because nearly all CLI types satisfy it. A `comparison` constraint is a stronger constraint, because it usually implies that a type must implement `System.IComparable`.

### Asserting Equality, Hashing, and Comparison Using Attributes

These attributes control the comparison and equality semantics of type definitions:

* `StructuralEquality` and `StructuralComparison`: Indicate that a structural type must support equality and comparison.

* `NoComparison` and `NoEquality`: Indicate that a type doesn't support equality or comparison.

* `CustomEquality` and `CustomComparison`: Indicate that a structural type supports custom equality and comparison.

Let's look at examples of these. Sometimes you may want to assert that a structural type must support structural equality, and you want an error at the definition of the type if it doesn't. Do this by adding the `StructuralEquality` or `StructuralComparison` attributes to the type:

```F#
[<StructuralEquality; StructuralComparison>]
type MiniIntegerContainer = MiniIntegerContainer of int
```

This adds extra checking. In the following example, the code gives an error at compile time—the type can't logically support automatic structural comparison, because one of the element types doesn't support ordered comparison:

```F#
[<StructuralEquality; StructuralComparison>]
type MyData = MyData of int * string * string * System.Net.WebClient

   error FS1177: The struct, record or union type 'MyData' has the 'StructuralComparison' attribute but the component type 'System.Net.WebClient' does not satisfy the 'comparison' constraint
```

### Fully Customizing Equality, Hashing, and Comparison on a Type

Many types in the .NET libraries come with custom equality, hashing, and comparison implementations. For example, `System.DateTime` has custom implementations of these.

F# also allows you to define custom equality, hashing, and comparison for new type definitions. For example, values of a type may carry a unique integer tag that can be used for this purpose. In such cases, we recommend that you take full control of your destiny and define custom comparison and equality operations on your type. For example, Listing 9-5 shows how to customize equality, hashing (using the predefined `hash` function), and comparison based on a unique `stamp` integer value. The type definition includes an implementation of `System.IComparable` and overrides of `Object.Equals` and `Object.GetHashCode`.

##### Listing 9-5. Customizing equality, hashing, and comparison for a record type definition

```F#
/// A type abbreviation indicating we're using integers for unique stamps
/// on objects
type stamp = int

/// A structural type containing a function that can't be compared for equality
[<CustomEquality; CustomComparison>]
type MyThing =
    {Stamp : stamp; Behavior : int -> int}
     
    override x.Equals(yobj) =
        match yobj with
        | :? MyThing as y -> x.Stamp = y.Stamp
        | _ -> false
        
    override x.GetHashCode() = hash x.Stamp
    
    interface System.IComparable with
        member x.CompareTo yobj =
            match yobj with
            | :? MyThing as y -> compare x.Stamp y.Stamp
            | _ -> invalidArg "yobj" "cannot compare values of different types"
```

The `System.IComparable` interface is defined in the .NET libraries:

```F#
namespace System

type IComparable =
    abstract CompareTo : obj -> int
```

Recursive calls to compare subexpressions are processed using the functions:

```F#
val hash : 'T -> int when 'T : equality
val (=) : 'T -> 'T -> bool when 'T : equality
val compare : 'T -> 'T -> int when 'T : comparison
```

Listing 9-6 shows the same for a union type, this time using some helper functions.

##### Listing 9-6.  Customizing generic hashing and comparison on a union type

```F#
let inline equalsOn keyof x (yobj : obj) =
    match yobj with
    | :? 'T as y -> (keyof x = keyof y)
    | _ -> false

let inline hashOn keyof x =  hash (keyof x)

let inline compareOn keyof x (yobj : obj) =
    match yobj with
    | :? 'T as y -> compare (keyof x) (keyof y)
    | _ -> invalidArg "yobj" "cannot compare values of different types"

//type stamp = int

[<CustomEquality; CustomComparison>]
type MyUnionType =
    | MyUnionType of stamp * (int -> int)

    static member Stamp (MyUnionType (s, _)) = s

    override x.Equals y = equalsOn MyUnionType.Stamp x y
    override x.GetHashCode() = hashOn MyUnionType.Stamp x
    interface System.IComparable with
        member x.CompareTo y = compareOn MyUnionType.Stamp x y
```

Listing 9-6 also shows how to implement the `System.Object` method `GetHashCode`. This follows the same pattern as generic equality. Finally, you can declare that a structural type should use reference equality:

```F#
[<ReferenceEquality>]
type MyFormWrapper = MyFormWrapper of System.Windows.Forms.Form * (int -> int)
```

There is no such thing as reference comparison (the object pointers used by .NET move around, so the ordering would change). You can implement that by using a unique tag and custom comparison.

### Suppressing Equality, Hashing, and Comparison on a Type

You can suppress equality on an F# defined type by using the `NoEquality` attribute on the definition of the type. This means the type isn't considered to satisfy the equality constraint. Likewise, you can suppress comparison on an F# defined type by using the `NoComparison` attribute on the definition of the type:

```F#
[<NoEquality; NoComparison>]
type MyProjections =
    | MyProjections of (int * string) * (string -> int)
```

Adding these attributes to your library types makes client code safer, because it's less likely to inadvertently rely on equality and comparison over types for which these operations make no sense.

### Customizing Generic Collection Types

Programmers love defining new generic collection types. This is done less often in .NET and F# programming than in other languages, because the F# and .NET built-in collections are so good, but it's still important.

Equality and comparison play a role here. For example, it's common to have collections in which some of the values can be indexed using hashing, compared for equality when searching, or compared using an ordering. For example, seeing a constraint on this signature on a library type would come as no surprise:

```F#
type Graph<'Node when 'Node : equality>() = ...
```

The presence of the constraint is somewhat reassuring, because the requirement on node types is made clearer. Sometimes it's also desirable to be able to compare entire containers; for example, to compare one set with another, one map with another, or one graph with another. Consider the simplest generic collection type of all, which holds only one element. You can define it easily in F#:

```F#
type MiniContainer<'T> = MiniContainer of 'T
```

In this case, this is a structural type definition, and F# infers that there is an equality and comparison dependency on `'T`. All done! You can use `MiniContainer<_>` with values of any type, and you can do equality and comparison on `MiniContainer` values only if the element type also supports equality and comparison. Perfect.

If `MiniContainer` is a class type or has customized comparison and equality logic, however, then you need to be more explicit about dependencies. You can declare dependencies using the `EqualityConditionalOn` and `ComparisonConditionalOn` attributes on the type parameter. You should also use the operators `Unchecked.equals`, `Unchecked.hash`, and `Unchecked.compare` to process elements recursively. With these attributes, `MiniContainer<A>` satisfies the equality and comparison constraints if A itself satisfies these constraints. Here's a full example:

```F#
type MiniContainer<[<EqualityConditionalOn; ComparisonConditionalOn>]'T>(el : 'T) =
    member x.Value = el
    override x.Equals(yobj) =
        match yobj with
        | :? MiniContainer<'T> as y -> Unchecked.equals x.Value y.Value
        | _ -> false

    override x.GetHashCode() = Unchecked.hash x.Value

    interface System.IComparable with
        member x.CompareTo yobj =
            match yobj with
            | :? MiniContainer<'T> as y -> Unchecked.compare x.Value y.Value
            | _ -> invalidArg "yobj" "cannot compare values of different types"
```

---

##### Note  

Be careful about using generic equality, hashing, and comparison on mutable data. Changing the value of a field may change the value of the hash or the results of the operation. it's normally better to use the operations on immutable data or on data with custom implementations.

---

