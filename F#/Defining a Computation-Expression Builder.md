### Defining a Computation-Expression Builder

Table 16-2. Some Typical Computation-Expression Builder Members, as Required by the F# Compiler

```F#
member Bind : M<'T> * ('T -> M<'U>) -> M<'U>
```

Used to de-sugar `let!` and `do!` within computation expressions

```F#
member Return : 'T -> M<'T>
```

Used to de-sugar `return` within computation expressions

```F#
member ReturnFrom : M<'T> -> M<'T>
```

Used to de-sugar `return!` within computation expressions

```F#
member Delay : (unit -> M<'T>) -> M<'T>
```

Used to ensure that side effects within a computation expression are performed when expected

```F#
member For : seq<'T> * ('T -> M<'U>) -> M<'U>
```

Used to de-sugar `for ... do ...` within computation expressions. `M<'U>` can optionally be `M<unit>`.

```F#
member While : (unit -> bool) * M<'T> -> M<'T>
```

Used to de-sugar `while ... do ...` within computation expressions. `M<'T>` may optionally be `M<unit>`.

```F#
member Using : 'T * ('T -> M<'T>) -> M<'T> when 'T :> IDisposable
```

Used to de-sugar `use` bindings within computation expressions

```F#
member Combine : M<'T> * M<'T> -> M<'T>
```

Used to de-sugar sequencing within computation expressions. The first `M<'T>` may optionally be `M<unit>`.

```F#
member Zero : unit -> M<'T>
```

Used to de-sugar empty `else` branches of `if/then` constructs within computation expressions



Most of the elements of a builder are usually implemented in terms of simpler primitives. For example, assume you're defining a builder for some type `M<'T>` and you already have implementations of functions `bindM` and `returnM` with the types: