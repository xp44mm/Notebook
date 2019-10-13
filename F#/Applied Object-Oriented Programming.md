# CHAPTER 8 Applied Object-Oriented Programming

This chapter could also be called “being functional in an object-oriented world.” By now you've mastered the F# syntax and style, but there is still plenty to be said about using F# in the object-oriented landscape of the .NET platform.

In this chapter, you learn how to create F# types that integrate better with other object- oriented languages like C#. For example, with operator overloading, you can allow your types to be used with symbolic code. Also, events enable your classes to notify consumers to receive notifications when certain actions occur. This not only enables you to be more expressive in your F# code, but also reduces the friction with non-F# developers when sharing your code.

## Operators

A symbolic notation for manipulating objects can simplify your code by eliminating methods like Add and Remove, and instead using more intuitive counterparts like `+` and `-`. Also, if an object represents a collection of data, being able to index directly into it with the `.[ ]` or `.[expr .. expr]` notation can make it much easier to access the data an object holds.

### Operator Overloading

Operator overloading is a term used for adding new meaning to an existing operator, like `+` or `-`. Instead of only working on primitive types, you can overload those operators to accept custom types you create. Operator overloading in F# can be done by simply adding a static method to a type. After that, you can use the operator as if it natively supported the type. The F# compiler will simply translate the code into a call to your static method.

---

##### Warning

Be sure to use an operator that makes sense. For example, overloading `+` for a custom type shouldn't subtract values or remove items from a collection.

---

When you overload an operator, the operator's name must be in parentheses. For ex ample, `(+)` overloads the plus operator. To overload a unary operator, simply prefix the operator with a tilde (~).

Example 8-1 overloads the `+` and `−` operators to represent operators on a bottle. Note the `~−` operator allows for unary negation.

##### Example 8-1. Operator overloading

``` F#
[<Measure>]
type ml
type Bottle(capacity : float<ml>) =
    new() = new Bottle(0.0<ml>)
    member this.Volume = capacity
    static member (+) (lhs : Bottle, rhs) =
        new Bottle(lhs.Volume + rhs)
    static member (-) (lhs : Bottle, rhs) =
        new Bottle(lhs.Volume - rhs)
    static member (~-) (rhs : Bottle) =
        new Bottle(rhs.Volume * −1.0<1>)
    override this.ToString() =
        sprintf "Bottle(%.1fml)" (float capacity)
```

Once the proper operators have been added, then you can intuitively use the class using the standard symbolic operators you would expect (well, a bottle really can't have a negative volume, but you get the idea):

``` F#
> let half = new Bottle(500.0<ml>);;
val half : Bottle = Bottle(500.0ml)
> half + 500.0<ml>;;
val it : Bottle = Bottle(1000.0ml) {Volume = 1000.0;}
> half - 500.0<ml>;;
val it : Bottle = Bottle(0.0ml) {Volume = 0.0;}
> -half;;
val it : Bottle = Bottle(-500.0ml) {Volume = −500.0;}
```

You can also add overloaded operators to discriminated unions and record types the same way they are added to class types:

``` F#
type Person =
    | Boy of string
    | Girl of string
    | Couple of Person * Person
    static member (+) (lhs, rhs) =
        match lhs, rhs with
        | Couple(_), _
        | _, Couple(_)
            -> failwith "Three's a crowd!"
        | _ -> Couple(lhs, rhs)
```

Using the `+` operator, we can now take two Person objects and convert them into a Couple union case:

``` F#
> Boy("Dick") + Girl("Jane");;
val it : Person = Couple (Boy "Dick",Girl "Jane")
```

---

##### Warning

F# allows you to define arbitrary symbolic operators for types; however, only a subset of these can be used natively with C# and VB.NET. Only the following operators are recognized by those languages:

Binary operators:

``` F#
+, −, *, /, %, &, |, <<, >>, +=, −=, *=, /=, %=
```

Unary operators:

``` F#
~+, ~−, ~!, ~++, ~−−
```

Otherwise the method will be exposed as a friendly name describing the symbols used. For example, operator `+−+` will be exposed as a static method named `op_PlusMinusPlus`.

Note that this list does not contain `>` and `<`. To define binary comparison operators you must implement `IComparable`.

---

### Indexers

For types that are a collection of values, an indexer is a natural way to extend the met aphor by enabling developers to index directly into the object. For example, you can get an arbitrary element of an array by using the `.[ ]` operator.

In F#, you can add an indexer by simply adding a property named Item. This property will be evaluated whenever you call the `.[ ]` method on the class.

Example 8-2 creates a class Year and adds an indexer to allow you to look up the nth day of the year.

##### Example 8-2. Adding an indexer to a class

``` F#
open System
type Year(year : int) =
    member this.Item (idx : int) =
        if idx < 1 || idx > 365 then
            failwith "Invalid day range"
        let dateStr = sprintf "1-1-%d" year
        DateTime.Parse(dateStr).AddDays(float (idx - 1))
```

The following FSI session shows using the indexer for type Year:

``` F#
> // Using a custom indexer
let eightyTwo   = new Year(1982)
let specialDay = eightyTwo.[171];;
val eightyTwo : Year
val specialDay : DateTime = 6/20/1982 12:00:00 AM
> specialDay.Month, specialDay.Day, specialDay.DayOfWeek;;
val it : int * int * DayOfWeek = (6, 20, Sunday)
```


But what if you want to not only read values from an indexer, but write them back as well? Also, what if you wanted the indexer to accept a non-integer parameter? The F# language can accommodate both of those requests.

Example 8-3 defines an indexer for a type that accepts a non-integer parameter. In a different take on the Year class example, the indexer takes a month and date tuple.

##### Example 8-3. Non-integer indexer

``` F#
type Year2(year : int) =
    member this.Item (month : string, day : int) =
        let monthIdx =
            match month.ToUpper() with
            | "JANUARY" -> 1  | "FEBRUARY" -> 2  | "MARCH"     -> 3
            | "APRIL"   -> 4  | "MAY"      -> 5  | "JUNE"      -> 6
            | "JULY"    -> 7  | "AUGUST"   -> 8  | "SEPTEMBER" -> 9
            | "OCTOBER" -> 10 | "NOVEMBER" -> 11 | "DECEMBER"  -> 12
            | _ -> failwithf "Invalid month [%s]" month
        let dateStr = sprintf "1-1-%d" year
        DateTime.Parse(dateStr).AddMonths(monthIdx - 1).AddDays(float (day - 1))
```

Year2 uses a more intuitive indexer that makes the type easier to use:

``` F#
> // Using a non-integer index
let O'seven = new Year2(2007)
let randomDay = O'seven.["April", 7];;
val O'seven : Year2
val randomDay : DateTime = 4/7/2007 12:00:00 AM
> randomDay.Month, randomDay.Day, randomDay.DayOfWeek;;
val it : int * int * DayOfWeek = (4, 7, Saturday)
```

The read-only indexers you've seen so far aren't ideal for every situation. What if you wanted to provide a read/write indexer to not only access elements but to also update their values? To make a read/write indexer, simply add an explicit getter and setter to the Item property.

Example 8-4 defines a type `WordBuilder` that allows you to access and update letters of a word at a given index.

##### Example 8-4. Read/write indexer

``` F#
open System.Collections.Generic
type WordBuilder(startingLetters : string) =
    let m_letters = new List<char>(startingLetters)
    member this.Item
        with get idx   = m_letters.[idx]
        and  set idx c = m_letters.[idx] <- c
    member this.Word = new string (m_letters.ToArray())
```

The syntax for writing a new value using the indexer is the same as when updating a value in an array:

``` F#
> let wb = new WordBuilder("Jurassic Park");;
val wb : WordBuilder
> wb.Word;;
val it : string = "Jurassic Park"
> wb.[10];;
val it : char = 'a'
> wb.[10] <- 'o';;
val it : unit = ()
> wb.Word;;
val it : string = "Jurassic Pork"
```

### Adding Slices

Similar to indexers, slices allow you to index into a type that represents a collection of data. The main difference is that where an indexer returns a single element, a slice produces a new collection of data. Slices work by providing syntax support for providing an optional lower bound and an optional upper bound for an indexer.

To add a one-dimensional slice, add a method named `GetSlice` that takes two option types as parameters. Example 8-5 defines a slice for a type called `TextBlock` where the slice returns a range of words in the body of the text.

##### Example 8-5. Providing a slice to a class

``` F#
type TextBlock(text : string) =
    let words = text.Split([| ' ' |])
    member this.AverageWordLength =
        words |> Array.map float |> Array.average
    member this.GetSlice(lowerBound : int option, upperBound : int option) =
        let words =
            match lowerBound, upperBound with
            // Specify both upper and lower bounds
            | Some(lb), Some(ub) -> words.[lb..ub]
            // Just one bound specified
            | Some(lb), None     -> words.[lb..]
            | None,     Some(ub) -> words.[..ub]
            // No lower or upper bounds
            | None,      None -> words.[*]
        words
```

Using our `TextBlock` class we can see:

``` F#
> // Using a custom slice operator
let text = "The quick brown fox jumped over the lazy dog"
let tb = new TextBlock(text);;
val text : string = "The quick brown fox jumped over the lazy dog"
val tb : TextBlock
> tb.[..5];;
val it : string [] = [|"The"; "quick"; "brown"; "fox"; "jumped"; "over"|]
> tb.[4..7];;
val it : string [] = [|"jumped"; "over"; "the"; "lazy"|]
```

If you need a more refined way to slice your data, you can provide a two-dimension slicer. This allows you to specify two ranges of data. To define a 2-D slice, simply add a method called `GetSlice` again, except that the method should now take four option-type parameters. In order, they are: first-dimension lower bound, first-dimension upper bound, second-dimension lower bound, and second-dimension upper bound.

Example 8-6 defines a class that encapsulates a sequence of data points. The type pro vides a 2-D slice that returns only those points within the given range.

##### Example 8-6. Defining a two-dimensional slice

``` F#
open System
type DataPoints(points : seq<float * float>) =

    member this.GetSlice(xlb, xub, ylb, yub) =
        let getValue optType defaultValue =
            match optType with
            | Some(x) -> x
            | None    -> defaultValue
        let minX = getValue xlb Double.MinValue
        let maxX = getValue xub Double.MaxValue
        let minY = getValue ylb Double.MinValue
        let maxY = getValue yub Double.MaxValue
        // Return if a given tuple representing a point is within range
        let inRange (x, y) =
            (minX < x && x < maxX &&
             minY < y && y < maxY)
        Seq.filter inRange points
```

The following FSI session shows the 2-D slice in action. The slice allows you to only get data points that fall within a specific range, allowing you to isolate certain sections for further analysis:

``` F#
> // Define 1,000 random points with value between 0.0 to 1.0
let points =
  seq {
      let rng = new Random()
      for i = 0 to 1000 do
          let x = rng.NextDouble()
          let y = rng.NextDouble()
          yield (x, y)
  };;
val points : seq<float * float>
> points;;
val it : seq<float * float> =
  seq
    [(0.6313018304, 0.8639167859); (0.3367836328, 0.8642307673);
     (0.7682324386, 0.1454097885); (0.6862907925, 0.8801649925); ...]
> let d = new DataPoints(points);;
val d : DataPoints
> // Get all values where the x and y values are greater than 0.5.
d.[0.5.., 0.5..];;
val it : seq<float * float> =
  seq
    [(0.7774719744, 0.8764193784); (0.927527673, 0.6275711235);
     (0.5618227448, 0.5956258004); (0.9415076491, 0.8703156262); ...]
> // Get all values where the x-value is between 0.90 and 0.99,
// with no restriction on the y-value.
d.[0.90 .. 0.99, *];;
val it : seq<float * float> =
  seq
    [(0.9286256581, 0.08974070246); (0.9780231351, 0.5617941956);
     (0.9649922103, 0.1342076446); (0.9498346559, 0.2927135142); ...]
```

Slices provide an expressive syntax for extracting data from types. However, you cannot add a slice notation for a type higher than two dimensions.

## Generic Type Constraints

Adding generic type parameters to a class declaration enables the type to be used in conjunction with any other type. However, what if you want the class to actually do something with that generic type? Not just hold a reference to an instance `'a`, but do something meaningful with `'a` as well. Because `'a` could be anything, you are out of luck.

Fortunately, you can add a generic type constraint, which will enable you to still have the class be generic but put a restriction on the types it accepts. For example, you can force the generic class to only accept types that implement a specific interface. This allows you to restrict which types can be used as generic type parameters so that you can do more with those generic types.

A generic type constraint is simply a form of type annotation. Whenever you would have written a generic type `'a`, you can add type constraints using the `when` keyword. Example 8-7 defines a generic class `GreaterThanList` where the type parameter must also implement the `IComparable` interface. This allows instances of the generic type to be cast to the `IComparable` interface, and ultimately enables items to be checked to be sure they are greater than the first element in the list.

##### Example 8-7. Generic type constraints

``` F#
open System
open System.Collections.Generic

exception NotGreaterThanHead

/// Keep a list of items where each item added to it is
/// greater than the first element of the list.
type GreaterThanList< 'a when 'a :> IComparable<'a> >(minValue : 'a) =
    let m_head = minValue
    let m_list = new List<'a>()
    do m_list.Add(minValue)
    member this.Add(newItem : 'a) =
        // Casting to IComparable wouldn't be possible
        // if 'a weren't constrainted
        let ic = newItem :> IComparable<'a>
        if ic.CompareTo(m_head) >= 0 then
            m_list.Add(newItem)
        else
            raise NotGreaterThanHead
    member this.Items = m_list :> seq<'a>
```

There are many ways to constrain types in F#. Using multiple type constraints allows you to refine a type parameter; to combine multiple type constraints, simply use the `and` keyword:

``` F#
/// Given a type x which contains a constructor and implements IFoo,
/// returns an instance of that type cast as IFoo.
let fooFactory<'a when 'a : (new : unit -> 'a) and 'a :> IFoo >() =
    // Creating new instance of 'a because the type constraint enforces
    // that there be a default constructor.
    let x = new 'a()
    // Comparing x with a new instance, because we know 'a implements IFoo
    let ifoo = x :> IFoo
    ifoo
```

Most type constraints deal with kind of “type”

*Value type constraint*

The value type constraint enforces that the type argument must be a value type. The syntax for the value type constraint is as follows:

``` F#
'a : struct
```

*Reference type constraint*

The reference constraint enforces that the type argument must be a reference type. The syntax for the reference constraint is as follows:

``` F#
'a : not struct
```


*Enumeration constraint*

With the enumeration constraint, the type argument must be an enumeration type whose underlying type must be of the given base type (such as int). The syntax for the enumeration constraint is as follows:

``` F#
'a : enum<basetype>
```

*Null constraint*

The null constraint requires that the type argument be assignable to `null`. This is true for all objects, except those defined in F# without the `[<AllowNullLiteral>]` attribute:

``` F#
'a : null
```

*Unmanaged constraint*

The provided type must be unmanaged (either a primitive value type like int or float, or a non-generic struct with only unmanaged types as fields):

``` F#
'a : unmanaged
```

*Delegate constraint*

The delegate constraint requires that the type argument must be a delegate type, with the given signature and return type. (Delegates are covered in the next section.) The syntax for defining a delegate constraint is as follows:

``` F#
'a : delegate<tupled-args, return-type>
```



The second set of constraints has to do with the capabilities of the type; these are the most common types of generic constraints.

*Type constraint*

The subtype constraint enforces that the generic type parameter must match or be a subtype of a given type. (This also applies to implementing interfaces.) This allows you to cast `'a` as a known type. The syntax for a subtype constraint is as follows:

``` F#
'a :> type
```

*Constructor constraint*

The default constructor constraint enforces that the type parameter must have a default (parameter-less) constructor. This allows you to instantiate new instances of the generic type. The syntax for a default constructor constraint is as follows:

``` F#
'a : (new : unit -> 'a)
```

*Equality constraint*

The type must support equality. Nearly all .NET types satisfy this constraint by the use of the default referential equality. F# types annotated with the `[<NoEquality>]` attribute do not satisfy this constraint.

``` F#
'a : equality
```

*Comparison constraint*

The type must support comparison by implementing the `System.IComparable` interface. F# records and discriminated unions support the comparison constraint by default unless annotated with the `[<NoComparison>]` attribute.

``` F#
'a : comparison
```

## Delegates

So far, the classes and types we have created have been useful, but they needed to be acted upon in order for something to happen. However, it would be beneficial if types could be proactive instead of reactive. That is, you want types that act upon others, and not the other way around.

Consider Example 8-8, which defines a type `CoffeeCup` that lets interested parties know when the cup is empty. This is helpful in case you wanted to automatically get a refill and avoid missing out on a tasty cup of joe. This is done by keeping track of a list of functions to call when the cup is eventually empty.

##### Example 8-8. Creating proactive types

``` F#
open System.Collections.Generic

[<Measure>]
type ml

type CoffeeCup(amount : float<ml>) =
    let mutable m_amountLeft = amount
    let mutable m_interestedParties = List<(CoffeeCup -> unit)>()
    
    member this.Drink(amount) =
        printfn "Drinking %.1f..." (float amount)
        m_amountLeft <- max (m_amountLeft - amount) 0.0<ml>
        if m_amountLeft <= 0.0<ml> then
            this.LetPeopleKnowI'mEmpty()
            
    member this.Refil(amountAdded) =
        printfn "Coffee Cup refilled with %.1f" (float amountAdded)
        m_amountLeft <- m_amountLeft + amountAdded
        
    member this.WhenYou'reEmptyCall(func) =
        m_interestedParties.Add(func)
        
    member private this.LetPeopleKnowI'mEmpty() =
        printfn "Uh ho, I'm empty! Letting people know..."
        for interestedParty in m_interestedParties do
            interestedParty(this)
```

When used in an FSI session, the following transpires after a refreshing cup of coffee has been emptied:

``` F#
> let cup = new CoffeeCup(100.0<ml>);;
val cup : CoffeeCup
> // Be notified when the cup is empty
cup.WhenYou'reEmptyCall(
    (fun cup ->
        printfn "Thanks for letting me know..."
        cup.Refil(50.0<ml>) ));;
val it : unit = ()
> cup.Drink(75.0<ml>);;
Drinking 75.0...
val it : unit = ()
> cup.Drink(75.0<ml>);;
Drinking 75.0...
Uh ho, I'm empty! Letting people know...
Thanks for letting me know...
Coffee Cup refilled with 50.0
val it : unit = ()
```

Abstractly, we can describe the `CoffeeCup` class as a pattern enabling consumers—people who have an instance of the class—to subscribe to an event—when the cup is empty. This turns out to be very useful, and makes up the foundation of most graphical programming environments (so-called event-driven programming). For example, windowing systems allow you to provide functions to be called when a button is clicked or when a check box is toggled. However, our custom implementation of keeping a list of function values is clunky at best.


Fortunately, the .NET framework has rich support for this pattern through the use of delegates and events. A delegate is very similar to F#’s function values. An event is simply the mechanism for calling the functions provided by all interested parties.

When used together, delegates and events form a contract. The first class publishes an event, which enables other classes to subscribe to that event. When the class's event is fired, the delegates provided by all subscribers are executed.

Delegates can be thought of as an alternate form of function values. They represent a function pointer to a method and can be passed around as parameters or called directly, just like function values. Delegates, however, offer a couple of advantages over the F# functions you are used to, as we see shortly.

### Defining Delegates

To define a delegate, simply use the following syntax. The first type represents the parameters passed to the delegate and the second type is the return type of the delegate when called:

``` F#
type ident = delegate of type1 -> type2
```

Example 8-9 shows how to create a simple delegate and how similar it is to a regular function value. To instantiate a delegate type, you use the new keyword and pass the body of the delegate in as a parameter. To execute a delegate, you must call its `Invoke` method.

##### Example 8-9. Defining and using delegates

``` F#
let functionValue x y =
    printfn "x = %d, y = %d" x y
    x + y
    
// Defining a delegate
type DelegateType = delegate of int * int -> int

// Construct a delegate value
let delegateValue1 =
    new DelegateType(
        fun x y ->
            printfn "x = %d, y = %d" x y
            x + y
    )
    
// Calling function values and delegates
let functionResult = functionValue 1 2
let delegateResult = delegateValue1.Invoke(1, 2)
```

F# has a bit of magic that eliminates the need for instantiating some delegate types. If a class member takes a delegate as a parameter, rather than passing an instance of that delegate type, you can pass in a lambda expression, provided it has the same signature as the delegate.

The following code snippet defines a delegate `IntDelegate` and a class method `ApplyDelegate`. `ApplyDelegate` is called twice, the first using an explicit instance of `IntDelegate`, and again using an implicit instance. The lambda expression provided would create a valid instance of `IntDelegate` and so the F# compiler creates one behind the scenes.

This dramatically cleans up F# code that interacts with .NET components that expect delegates, such as the Parallel Extensions to the .NET Framework, discussed in Chapter 11:

``` F#
type IntDelegate = delegate of int -> unit

type ListHelper =
    /// Invokes a delegate for every element of a list
    static member ApplyDelegate (l : int list, d : IntDelegate) =
        l |> List.iter (fun x -> d.Invoke(x))

// Explicitly constructing the delegate
ListHelper.ApplyDelegate([1 .. 10], new IntDelegate(fun x -> printfn "%d" x))

// Implicitly constructing the delegate
ListHelper.ApplyDelegate([1 .. 10], (fun x -> printfn "%d" x))
```

### Combining Delegates

Although delegates are very similar to function values, there are two key differences. First, delegates can be combined. Combining multiple delegates together allows you to execute multiple functions with a single call to `Invoke`. Delegates are coalesced by calling the `Combine` and `Remove` static methods on the `System.Delegate` type, which is the base class for all delegates.

Example 8-10 shows combining delegates so that a single call to `Invoke` will execute multiple delegates at once.

##### Example 8-10. Combining delegates

``` F#
open System.IO

type LogMessage = delegate of string -> unit

let printToConsole  =
    LogMessage(fun msg -> printfn "Logging to console: %s..." msg)

let appendToLogFile =
    LogMessage(fun msg -> printfn "Logging to file: %s..." msg
                          use file = new StreamWriter("Log.txt", true)
                          file.WriteLine(msg))

let doBoth = LogMessage.Combine(printToConsole, appendToLogFile)

let typedDoBoth = doBoth :?> LogMessage
```

The following is the output from the FSI session. Invoking the `typedDoBoth` delegate executes both the `printToConsole` delegate and the `appendToLogFile` delegate. F# function values cannot be combined in this way:

``` F#
> typedDoBoth.Invoke("[some important message]");;
Logging to console: [some important message]...
Logging to file: [some important message]...
val it : unit = ()
```

The second main difference between delegate types and function values is that you can invoke a delegate asynchronously, so the delegate's execution will happen on a separate thread and your program will continue as normal. To invoke a delegate asynchronously, call the `BeginInvoke` and `EndInvoke` methods.

Unfortunately, delegates created in F# do not have the `BeginInvoke` and `EndInvoke` methods, so to create delegates that can be invoked asynchronously you must define them in VB.NET or C#. In Chapter 11, we look at parallel and asynchronously programming in F# and simple ways to work around this.

## Events

Now that you understand delegates, let's look at how to take advantage of them to create events. Events are just syntactic sugar for properties on classes that are delegates. So when an event is raised, it is really just invoking the combined delegates associated with the event.

### Creating Events

Let's start with a simple example and work backward from that. Unlike C# or VB.NET, there is no event keyword in F#.

Example 8-11 creates a `NoisySet` type that fires events whenever items are added or removed from the set. Rather than keeping track of event subscribers manually, it uses the `Event<'Del, 'Arg>` type, which is discussed shortly.

##### Example 8-11. Events and the Event<_,_> type

``` F#
type SetAction = Added | Removed

type SetOperationEventArgs<'a>(value : 'a, action : SetAction) =
    inherit System.EventArgs()
    member this.Action = action
    member this.Value = value
    
type SetOperationDelegate<'a> = delegate of obj * SetOperationEventArgs<'a> -> unit

// Contains a set of items that fires events whenever
// items are added.
type NoisySet<'a when 'a : comparison>() =
    let mutable m_set = Set.empty : Set<'a>
    let m_itemAdded =
        new Event<SetOperationDelegate<'a>, SetOperationEventArgs<'a>>()
    let m_itemRemoved =
        new Event<SetOperationDelegate<'a>, SetOperationEventArgs<'a>>()
    member this.Add(x) =
        m_set <- m_set.Add(x)
        // Fire the 'Add' event
        m_itemAdded.Trigger(this, new SetOperationEventArgs<_>(x, Added))
    member this.Remove(x) =
        m_set <- m_set.Remove(x)
        // Fire the 'Remove' event
        m_itemRemoved.Trigger(this, new SetOperationEventArgs<_>(x, Removed))
    // Publish the events so others can subscribe to them
    member this.ItemAddedEvent   = m_itemAdded.Publish
    member this.ItemRemovedEvent = m_itemRemoved.Publish
```

#### Delegates for events

The first thing to point out in the example is the delegate used for the event. In .NET, the idiomatic way to declare events is to have the delegate return unit and take two parameters. The first parameter is the source, or object raising the event, the second parameter is the delegate's arguments passed in an object derived from `System.EventArgs`.

In the example, the `SetOperationEventArgs` type stores all relevant information for the event, such as the value of the item being added or removed. Many events, however, are raised without needing to send any additional information, in which case the `EventArgs` class is not inherited from.

#### Creating and raising the event

Once the delegate has been defined for the event, the next step is to actually create the event. In F#, this is done by creating a new instance of the `Event<_,_>` type:

``` F#
let m_itemAdded =
    new Event<SetOperationDelegate<'a>, SetOperationEventArgs<'a>>()
```

We cover the `Event<_,_>` type shortly, but note that whenever you want to raise the event or add subscribers to the event, you must use that type. Raising the event is done by calling the `Trigger` method, which takes the parameters to the delegates to be called:

``` F#
// Fire the 'Add' event
m_itemAdded.Trigger(this, new SetOperationEventArgs<_>(x, Added))
```

#### Subscribing to events

Finally, to subscribe to a class's events, use the `AddHandler` method on the event property. From then on, whenever the event is raised, the delegate you passed to `AddHandler` will be called.

If you no longer want your event handler to be called, you can call the `RemoveHandler` method.

---

##### Warning

Be sure to remove an event handler when an object no longer needs to subscribe to an event. Otherwise, the event raiser will still have a reference to the object and therefore will prevent the subscriber from being garbage collected. This is a very common way to introduce memory leaks in .NET applications.

---

Example 8-12 shows the `NoisySet<_>` type in action.

##### Example 8-12. Adding and removing event handlers

``` F#
> // Using events
let s = new NoisySet<int>()
let setOperationHandler =
    new SetOperationDelegate<int>(
        fun sender args ->
            printfn "%d was %A" args.Value args.Action
    )
s.ItemAddedEvent.AddHandler(setOperationHandler)
s.ItemRemovedEvent.AddHandler(setOperationHandler);;
val s : NoisySet<int>
val setOperationHandler : SetOperationDelegate<int>
> s.Add(9);;
9 was Added
val it : unit = ()
> s.Remove(9);;
9 was Removed
val it : unit = ()
```

As you can see from Example 8-12, all the work of combining delegates and eventually raising the event was handled by the `Event<'Del, 'Arg>` class. But what is it exactly? And why does it need two generic parameters? 

### The Event<_,_> Class

The `Event<'Del, 'Arg>` type keeps track of the delegates associated with the event, and makes it easier to fire and publish an event. The type takes two generic parameters. The first is the type of delegate associated with the event, which in the previous example was `SetOperationDelegate`. The second generic parameter is the type of the delegate's argument, which was `SetOperationEventArgs`. (Note that this ignores the delegate's actual first parameter, the `sender` object.) 

If your events don't follow this pattern where the first parameter is the `sender` object, then you should use the `DelegateEvent<'Del>` type instead. It is used the same way, except that its arguments are passed in as an obj array.

Example 8-13 defines a clock type with a single event that gets fired every second, notifying subscribers of the current hour, minute, and second. Note that in order to trigger an event with type `DelegateEvent<_>` you must pass in an obj array for all of its parameters. As a reminder, the `box` function converts its parameter to type `obj`.

##### Example 8-13. The DelegateEvent type

``` F#
open System
type ClockUpdateDelegate = delegate of int * int * int -> unit
type Clock() =
    let m_event = new DelegateEvent<ClockUpdateDelegate>()
    member this.Start() =
        printfn "Started..."
        while true do
            // Sleep one second...
            Threading.Thread.Sleep(1000)
            let hour   = DateTime.Now.Hour
            let minute = DateTime.Now.Minute
            let second = DateTime.Now.Second
            m_event.Trigger( [| box hour; box minute; box second |] )
    member this.ClockUpdate = m_event.Publish
```

When finally run, Example 8-13 produces the following output:

``` F#
> // Non-standard event types
let c = new Clock();;
val c : Clock
> // Adding an event handler
c.ClockUpdate.AddHandler(
    new ClockUpdateDelegate(
        fun h m s -> printfn "[%d:%d:%d]" h m s
    )
);;
val it : unit = ()
> c.Start();;
Started...
[14:48:27]
[14:48:28]
[14:48:29]
```

### The Observable Module

Using events as a way to make class types proactive and notify others when particular events occur is helpful. However, events in F# don't have to be associated with a particular class, much like functions in F# don't have to be members of a particular class.

The `Observable` module defines a series of functions for creating instances of the `IObservable<_>` interface, which allows you to treat events like first-class citizens much like objects and functions. This way you can use functional programming techniques like composition and data transformation on events.

Example 8-14 defines a `JukeBox` type that raises an event whenever a new song is played. (The specifics of the mysterious `CLIEvent` attribute are covered later.)

##### Example 8-14. Compositional events

``` F#
[<Measure>]
type minute

[<Measure>]
type bpm = 1/minute

type MusicGenre = Classical | Pop | HipHop | Rock | Latin | Country

type Song = { Title : string; Genre : MusicGenre; BPM : int<bpm> }

type SongChangeArgs(title : string, genre : MusicGenre, bpm : int<bpm>) =
    inherit System.EventArgs()
    member this.Title = title
    member this.Genre = genre
    member this.BeatsPerMinute = bpm
type SongChangeDelegate = delegate of obj * SongChangeArgs -> unit
type JukeBox() =
    let m_songStartedEvent = new Event<SongChangeDelegate, SongChangeArgs>()
    member this.PlaySong(song) =
        m_songStartedEvent.Trigger(
            this,
            new SongChangeArgs(song.Title, song.Genre, song.BPM)
        )
    [<CLIEvent>]
    member this.SongStartedEvent = m_songStartedEvent.Publish
```

Rather than just adding event handlers directly to the `JukeBox` type, we can use the `Observable` module to create new, more specific event handlers. In Example 8-15, the `Observable.filter` and `Observable.partition` methods are used to create two new events, `slowSongEvent` and `fastSongEvent`, which will be raised whenever the `JukeBox` plays songs that meet a certain criteria.

The advantage is that you can take an existing event and transform it into new events, which can be passed around as values.

##### Example 8-15. Using the Event module

``` F#
> // Use the Observable module to only subscribe to specific events
let jb = new JukeBox()
let fastSongEvent, slowSongEvent =
    jb.SongStartedEvent
    // Filter event to just dance music
    |> Observable.filter(fun songArgs ->
            match songArgs.Genre with
            | Pop | HipHop | Latin | Country -> true
            | _ -> false)
    // Split the event into 'fast song' and 'slow song'
    |> Observable.partition(fun songChangeArgs ->
            songChangeArgs.BeatsPerMinute >= 120<bpm>);;
val jb : JukeBox
val slowSongEvent : IObservable<SongChangeArgs>
val fastSongEvent : IObservable<SongChangeArgs>
> // Add event handlers to the IObservable event
slowSongEvent.Add(fun args -> printfn
                                  "You hear '%s' and start to dance slowly..."
                                  args.Title)
fastSongEvent.Add(fun args -> printfn
                                  "You hear '%s' and start to dance fast!"
                                  args.Title);;
> jb.PlaySong( { Title = "Burnin Love"; Genre = Pop; BPM = 120<bpm> } );;
You hear 'Burnin Love' and start to dance fast!
val it : unit = ()
```

However, `Observable.filter` and `Observable.partition` are not the only useful methods in the module.

#### Observable.add

`Observable.add` simply subscribes to the event. Typically this method is used at the end of a series of pipe-forward operations and is a cleaner way to subscribe to events than calling `AddHandler` on the event object.

`Observable.add` has the following signature:

``` F#
val add : ('a -> unit) -> IObservable<'a> -> unit
```

Example 8-16 pops up a message box whenever the mouse moves into the bottom half of a form.

##### Example 8-16. Subscribing to events with Observable.add

``` F#
open System.Windows.Forms
let form = new Form(Text="Keep out of the bottom!", TopMost=true)
form.MouseMove
|> Observable.filter (fun moveArgs -> moveArgs.Y > form.Height / 2)
|> Observable.add    (fun moveArgs -> MessageBox.Show("Moved into bottom half!")
                                      |> ignore)
form.ShowDialog()
```

#### Observable.merge

`Observable.merge` takes two input events and produces a single output event, which will be fired whenever either of its input events is raised.

`Observable.merge` has the following signature:

``` F#
val merge: IObservable<'a> -> IObservable<'a> -> IObservable<'a>
```

This is useful for when trying to combine and simplify events. For example, in the previous example, if a consumer didn't care about specifically dancing to slow or fast songs, both the song events could be combined into a single `justDance` event:

``` F#
> // Combine two song events
let justDanceEvent = Observable.merge slowSongEvent fastSongEvent
justDanceEvent.Add(fun args -> printfn "You start dancing, regardless of tempo!");;
val justDanceEvent : System.IObservable<SongChangeArgs>
> // Queue up another song
jb.PlaySong(
     { Title = "Escape (The Pina Colada Song)"; Genre = Pop; BPM = 70<bpm> } );;
You hear 'Escape (The Pina Colada Song)' and start to dance slowly...
You start dancing, regardless of tempo!
val it : unit = ()
```

#### Observable.map

Sometimes when working with an event value, you want to transform it into some sort of function that is easier to work with. `Observable.map` allows you to convert an event with a given argument type into another. It has the following signature:

``` F#
val map: ('a -> 'b) -> IObservable<'a> -> IObservable<'b>
```

When using Windows Forms, all mouse click events are given in pixels relative to the top left of the form. So for a given form `f`, the top left corner is at position `(0, 0)` and the bottom right corner is at position `(f.Width, f.Height)`. Example 8-17 uses `Event.map` to create a new `Click` event that remaps the positions to points relative to the center of the form.

##### Example 8-17. Transforming event arguments with Event.map

``` F#
> // Create the form
open System.Windows.Forms
let form = new Form(Text="Relative Clicking", TopMost=true)
form.MouseClick.AddHandler(
    new MouseEventHandler(
        fun sender clickArgs ->
            printfn "MouseClickEvent    @ [%d, %d]" clickArgs.X clickArgs.Y
    )
)
// Create a new click event relative to the center of the form
let centeredClickEvent =
    form.MouseClick
    |> Observable.map (fun clickArgs -> clickArgs.X - (form.Width  / 2),
                                        clickArgs.Y - (form.Height / 2))
// Subscribe
centeredClickEvent
|> Observable.add (fun (x, y) -> printfn "CenteredClickEvent @ [%d, %d]" x y);;

val form : Form =  System.Windows.Forms.Form, Text: Relative Clicking
val centeredClickEvent : System.IObservable<int * int>
> // The output is from clicking the dialog twice, first in the
// top left corner and then in the center.
form.ShowDialog();;
MouseClickEvent    @ [4, 8]
CenteredClickEvent @ [−146, −142]
MouseClickEvent    @ [150, 123]
CenteredClickEvent @ [0, −27]
val it : DialogResult = Cancel
```

Using the `Observable` module enables you to think about events in much the same way as function values. This adds a great deal of expressiveness and enables you to do more with events in F# than you can in other .NET events.

### Creating .NET Events

There is just one extra step for creating events in F#. To create an event in F# code, you need to create a property that returns an instance of the `IEvent<_, _>` interface. However, to create an event that can be used by other .NET languages, you must also add the `[<CLIEvent>]` attribute. This is just a hint to the F# compiler specifying how to generate the code for the class. To F# consumers of your code, the class will behave identically; however, to other .NET languages, rather than being a property of type `IEvent<_,_>` there will be a proper .NET event in its place.

Example 8-18 revisits our coffee cup example, but rather than keeping an explicit list of functions to notify when the coffee cup is empty, a proper .NET event is used instead.

##### Example 8-18. Creating .NET compatible events

``` F#
open System

[<Measure>]
type ml

type EmptyCoffeeCupDelegate = delegate of obj * EventArgs ->  unit

type EventfulCoffeeCup(amount : float<ml>) =
    let mutable m_amountLeft = amount
    let m_emptyCupEvent = new Event<EmptyCoffeeCupDelegate, EventArgs>()
    member this.Drink(amount) =
        printfn "Drinking %.1f..." (float amount)
        m_amountLeft <- max (m_amountLeft - amount) 0.0<ml>
        if m_amountLeft = 0.0<ml> then
            m_emptyCupEvent.Trigger(this, new EventArgs())
    member this.Refil(amountAdded) =
        printfn "Coffee Cup refilled with %.1f" (float amountAdded)
        m_amountLeft <- m_amountLeft + amountAdded
    [<CLIEvent>]
    member this.EmptyCup = m_emptyCupEvent.Publish
```
