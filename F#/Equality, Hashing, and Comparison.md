# Equality, Hashing, and Comparison

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

First, note that these are *generic* operations¡ªthey can be used on objects of many different types. This can be seen by the use of `'T` in the signatures of these operations. The operations take one or two parameters of the same type. For example, you can apply the `=` operator to two `Form` objects, or two `System.DateTime` objects, or two `System.Type` objects, and something reasonable happens. Some other important derived generic types, such as the immutable (persistent) `Set<_>` and `Map<_,_>` types in the F# library, also use generic comparison on their key type:

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

Ordered comparison isn¡¯t permitted, however:

```F#
> client1 <= client2;;
error FS0001: The type 'System.Net.WebClient' does not support the 'comparison' constraint. 
For example, it does not support the 'System.IComparable' interface
```

That¡¯s good! There is no natural ordering for `WebClient` objects, or at least no ordering is provided by the .NET libraries.

Equality and comparison can work over the structure of types. For example, you can use the equality operators on a tuple only if the constituent parts of the tuple also support equality. This means that using equality on a tuple of `WebClient` objects is permitted:

```F#
> (client1, client2) = (client1, client2);;
val it : bool =  true
> (client1, client2) = (client2, client1);;
val it : bool = false
```

But using ordered comparison of a tuple isn¡¯t:

```F#
> (client1, "Data for client 1") <= (client2, " Data for client 2");;
error FS0001: The type 'System.Net.WebClient' does not support the 'comparison' constraint. 
For example, it does not support the 'System.IComparable' interface
```

