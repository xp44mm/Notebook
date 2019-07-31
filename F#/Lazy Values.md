## Lazy Values

Memoization is a form of caching. Another important variation on caching is a simple lazy value, a delayed computation of type `FSharp.Control.Lazy<'T>` for some type `'T`. Lazy values are usually formed by using the special keyword `lazy` (you can also make them explicitly by using the functions in the `FSharp.Core.Lazy` module). For example:

```F#
> let sixty = lazy (30 + 30);;
val sixty : Lazy<int> = Value is not created.
> sixty.Force();;
val it : int = 60
```

Lazy values of this kind are implemented as thunks holding either a function value that computes the 
result or the actual computed result. The lazy value is computed only once, and thus its effects are executed only once. For example, in the following code fragment, “Hello world” is printed only once:

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



