# byref parameters in F#

Some Times we need to change Parameters of Functions like interoperating with other .NET Languages. If you want to achieve this in F# You need to use a combination of values, the `byref` keyword and Address-of operator(&). Like Below Example.

**Example**

```F#
let incrementParam(a: int byref) = a <- a + 1
let mutable b = 30
incrementParam(&b)
```

Above example showing that `incrementParam`'s argument a is of type `int byref`. The `byref` keyword tells the compiler to expect the address of a mutable value. In the next line we use the `mutable` keyword so that when we pass `b` to `incrementParam`. The function can successfully change `b`'s value.

Lastly when we call `incrementParam`, we provide the address of the mutable values and we do this by using the address-of operator(&). `byref` parameter works with value type like primitive.

## Passing by Reference

Passing an F# value by reference involves [byrefs](https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/byrefs), which are managed pointer types. Guidance for which type to use is as follows:

- Use `inref<'T>` if you only need to read the pointer.
- Use `outref<'T>` if you only need to write to the pointer.
- Use `byref<'T>` if you need to both read from and write to the pointer.

```fsharp
let example1 (x: inref<int>) = printfn "It's %d" x

let example2 (x: outref<int>) = x <- x + 1

let example3 (x: byref<int>) =
    printfn "It'd %d" x
    x <- x + 1

// No need to make it mutable, since it's read-only
let x = 1
example1 &x

// Needs to be mutable, since we write to it
let mutable y = 2
example2 &y
example3 &y // Now 'y' is 3
```

Because the parameter is a pointer and the value is mutable, any changes to the value are retained after the execution of the function.

You can use a tuple as a return value to store any `out` parameters in .NET library methods. Alternatively, you can treat the `out`parameter as a `byref` parameter. The following code example illustrates both ways.

```fsharp
// TryParse has a second parameter that is an out parameter
// of type System.DateTime.
let (b, dt) = System.DateTime.TryParse("12-20-04 12:21:00")

printfn "%b %A" b dt

// The same call, using an address of operator.
let mutable dt2 = System.DateTime.Now
let b2 = System.DateTime.TryParse("12-20-04 12:21:00", &dt2)

printfn "%b %A" b2 dt2
```

https://docs.microsoft.com/en-us/dotnet/fsharp/language-reference/byrefs

# Byrefs

The `byref`/`inref`/`outref` types, which are a managed pointers. They have restrictions on usage so that you cannot compile a program that is invalid at runtime.

## Syntax

```fsharp
// Byref types as parameters
let f (x: byref<'T>) = ()
let g (x: inref<'T>) = ()
let h (x: outref<'T>) = ()

// Calling a function with a byref parameter
let mutable x = 3
f &x
```

## Byref, inref, and outref

There are three forms of `byref`:

- `inref<'T>`, a managed pointer for reading the underlying value.
- `outref<'T>`, a managed pointer for writing to the underlying value.
- `byref<'T>`, a managed pointer for reading and writing the underlying value.

A `byref<'T>` can be passed where an `inref<'T>` is expected. Similarly, a `byref<'T>` can be passed where an `outref<'T>` is expected.

## Using byrefs

To use a `inref<'T>`, you need to get a pointer value with `&`:

```fsharp
open System

let f (dt: inref<DateTime>) =
    printfn "Now: %s" (dt.ToString())

let dt = DateTime.Now
f &dt // Pass a pointer to 'dt'
```

To write to the pointer by using an `outref<'T>` or `byref<'T>`, you must also make the value you grab a pointer to `mutable`.

```fsharp
open System

let f (dt: byref<DateTime>) =
    printfn "Now: %s" (dt.ToString())
    dt <- DateTime.Now

// Make 'dt' mutable
let mutable dt = DateTime.Now

// Now you can pass the pointer to 'dt'
f &dt
```

If you are only writing the pointer instead of reading it, consider using `outref<'T>` instead of `byref<'T>`.

### Inref semantics

Consider the following code:

```fsharp
let f (x: inref<SomeStruct>) = x.SomeField
```

Semantically, this means the following:

- The holder of the `x` pointer may only use it to read the value.
- Any pointer acquired to `struct` fields nested within `SomeStruct` are given type `inref<_>`.

The following is also true:

- There is no implication that other threads or aliases do not have write access to `x`.
- There is no implication that `SomeStruct` is immutable by virtue of `x` being an `inref`.

However, for F# value types that **are** immutable, the `this` pointer is inferred to be an `inref`.

All of these rules together mean that the holder of an `inref` pointer may not modify the immediate contents of the memory being pointed to.

### Outref semantics

The purpose of `outref<'T>` is to indicate that the pointer should only be read from. Unexpectedly, `outref<'T>` permits reading the underlying value despite its name. This is for compatibility purposes. Semantically, `outref<'T>` is no different than `byref<'T>`.

### Interop with C#

C# supports the `in ref` and `out ref` keywords, in addition to `ref` returns. The following table shows how F# interprets what C# emits:

| C# construct                | F# infers    |
| :-------------------------- | :----------- |
| `ref` return value          | `outref<'T>` |
| `ref readonly` return value | `inref<'T>`  |
| `in ref` parameter          | `inref<'T>`  |
| `out ref` parameter         | `outref<'T>` |

The following table shows what F# emits:

| F# construct                                   | Emitted construct              |
| :--------------------------------------------- | :----------------------------- |
| `inref<'T>` argument                           | `[In]` attribute on argument   |
| `inref<'T>` return                             | `modreq` attribute on value    |
| `inref<'T>` in abstract slot or implementation | `modreq` on argument or return |
| `outref<'T>` argument                          | `[Out]` attribute on argument  |

## Scoping for byrefs

A `let`-bound value cannot have its reference exceed the scope in which it was defined. For example, the following is disallowed:

```fsharp
let test2 () =
    let x = 12
    &x // Error: 'x' exceeds its defined scope!

let test () =
    let x =
        let y = 1
        &y // Error: `y` exceeds its defined scope!
    ()
```

This prevents you from getting different results depending on if you compile with optimizations on or off.

# Understanding byref, ref and &

Well, I came to understand that F# is able to manage references (some sort of C++ like references). This enables the possibilities to change value of parameters passed in functions and also enables the programmer to return more than a single value. However here's what I need to know:

1. Ref keyword: The keyword `ref` is used to create, from a value, a reference to that value of the inferred type. So

   ```
   let myref = ref 10
   ```

   This means that F# will create an object of type `Ref<int>` putting there (in the mutable field) my `int 10`.

   OK. So I assume that `ref` is used to create instances of the `Ref<'a>` type. Is it correct?

2. Access value: In order to access a value stored in reference I can do this:

   ```
   let myref = ref 10
   let myval = myref.Value
   let myval2 = !myref
   ```

   While the `:=` operator just lets me edit the value like this:

   ```
   let myref = ref 10
   myref.Value <- 30
   myref := 40
   ```

   So `!` (Bang) dereferences my reference. And `:=` edit it. I suppose this is correct too.

3. The `&` operator: What does this operator do? Is it to be applied to a reference type? No, I guess it must be applied to a mutable value and what does this return? The reference? The address? If using interactive:

   ```
   let mutable mutvar = 10;;
   &a;;
   ```

   The last line throws an error so I do not understand what the `&` operator is for.

4. ByRef: What about `byref`? That's very important to me, but I realize I do not understand it. I understand it is used in function regarding parameter passing. One uses `byref` when he wants that the passed value can be edited (this is a bit against the functional language's philosophy but f# is something more than that). Consider the following:

   ```
   let myfunc (x: int byref) =
       x <- x + 10
   ```

   This is strange. I know that if you have a reference `let myref = ref 10` and then do this to edit the value: `myref <- 10` it arises an error because it should be like this: `myref := 10`. However, the fact that in that function I can edit `x` using the `<-` operator means that `x` is not a reference, right?

   If I assume that `x` is not a reference, then I assume also that, in functions, when using `byref` on a parameter, that parameter can have the mutable syntax applied to. So it is just a matter of syntax, if I assume this I am OK, and, in fact, everything works (no compiler errors). However, what is `x`?

5. Calling functions: How can I use a function utilizing byref parameters?

   The `&` operator is involved but could you explain this better please? In this article: [MSDN Parameters and Arguments](http://msdn.microsoft.com/en-us/library/dd233213.aspx) the following example is provided:

   ```F#
   type Incrementor(z) =
       member this.Increment(i : int byref) =
          i <- i + z
   
   let incrementor = new Incrementor(1)
   let mutable x = 10
   // A: Not recommended: Does not actually increment the variable. (Me: why?)
   incrementor.Increment(ref x)
   // Prints 10.
   printfn "%d" x  
   
   let mutable y = 10
   incrementor.Increment(&y) (* Me: & what does it return? *)
   // Prints 11.
   printfn "%d" y 
   
   let refInt = ref 10
   incrementor.Increment(refInt) (* Why does it not work in A, but here it does? *)
   // Prints 11.
   printfn "%d" !refInt
   ```

## Answer

**Ref keyword** Yes, when you write `let a = ref 10` you're essentially writing `let a = new Ref<int>(10)` where the `Ref<T>` type has a mutable field `Value`.

**Access value** The `:=` and `!` operators are just shortcuts for writing:

```
a.Value <- 10  // same as writing: a := 10
a.Value        // same as writing: !a
```

**ByRef** is a special type that can be (reasonably) used only in method parameters. It means that the argument should be essentially a pointer to some memory location (allocated on heap or stack). It corresponds to `out` and `ref` modifiers in C#. Note that you cannot create local variable of this type.

**The & operator** is a way to create a value (a pointer) that can be passed as an argument to a function/method expecting a `byref` type.

**Calling functions** the example with `byref` works because you're passing the method a reference to a local mutable variable. Via the reference, the method can change the value stored in that variable.

The following doesn't work:

```
let a = 10            // Note: You don't even need 'mutable' here
bar.Increment(ref a)  
```

The reason is that you're creating a new instance of `Ref<int>` and you're copying the value of `a` into this instance. The `Increment` method then modifies the value stored on heap in the instance of `Ref<int>`, but you don't have a reference to this object anymore.

```
let a = ref 10
bar.Increment(a)  
```

This works, because `a` is a value of type `Ref<int>` and you're passing a pointer to the heap-allocated instance to `Increment` and then get the value from heap-allocated reference cell using `!a`.

(You can use values created using `ref` as arguments for `byref` because the compiler handles this case specially - it will automatically take reference of the `Value` field because this is a useful scenario...).