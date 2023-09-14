## Enums

I briefly covered enumerated values in Chapter 2. Now I complete the description of `Enum` blocks by mentioning all the methods you can apply to them.

Any `Enum` you define in your application derives from `System.Enum`, which in turn inherits from `System.ValueType`. Ultimately, therefore, user-defined `Enum`s are value types, but they are special in that you can't define additional properties, methods, or events. All the methods they expose are inherited from `System.Enum`. (It's illegal to explicitly inherit a class from `System.Enum` in Visual Basic.)

All the examples in this section refer to the following `Enum` block:

```FSharp
// This Enum defines the data type accepted for a value entered by the end user.
type DataEntry =
| IntegerNumber  = 0
| FloatingNumber = 1
| CharString     = 2
| DateTime       = 3
```

By default, the first enumerated type is assigned the value 0. You can change this initial value if you want, but you aren't encouraged to do so. In fact, it is advisable that 0 be a valid value for any `Enum` blocks you define; otherwise, a uninitialized `Enum` variable would contain an invalid value.

The .NET documentation defines a few guidelines for `Enum` values:

- Use names without the `Enum` suffix; use singular names for regular `Enum` types and plural for bit-coded `Enum` types.
- Use PascalCase for the name of both the `Enum` and its members. (An exception is constants from the Windows application API, which are usually all uppercase.)
- Use 32-bit integers unless you need a larger range, which normally happens only if you have a bit-coded `Enum` with more than 32 possible values.
- Don't use `Enum`s for open sets, that is, sets that you might need to expand in the future (for example, operating system versions).

### Displaying and Parsing `Enum` Values

The `Enum` class overrides the `ToString` method to return the value as a readable string format. This method is useful when you want to expose a (nonlocalized) string to the end user:

```FSharp
let de: DataEntry = DataEntry.DateTime
// Display the numeric value.
Console.WriteLine(de)                 // => 3
// Display the symbolic value.
Console.WriteLine(de.ToString())      // => DateTime
```

Or you can use the capability to pass a format character to an overloaded version of the `ToString` method. The only supported format characters are `G`, `g` (general), `X`, `x` (hexadecimal), `F`, `f` (fixed-point), and `D`, `d` (decimal):

```FSharp
// General and fixed formats display the Enum name.
Console.WriteLine(de.ToString("F"))   // => DateTime
// Decimal format displays Enum value.
Console.WriteLine(de.ToString("D"))   // => 3
// Hexadecimal format displays eight hex digits.
Console.WriteLine(de.ToString("X"))   // => 00000003
```

The opposite of `ToString` is the `Parse` shared method, which takes a string and converts it to the corresponding enumerated value:

```FSharp
// This statement works only if Option Strict is Off.
let de: obj = Enum.Parse(typeof<DataEntry>, "CharString")
```

Two things are worth noticing in the preceding code. First, the `Parse` method takes a `Type` argument, so you typically use the `typeof<_>` operator. Second, `Parse` is a static method and you must use `Enum` as a prefix; ~~`Enum` is a reserved Visual Basic word, so you must either enclose it in brackets or use its complete `System.Enum` name.~~

Being inherited from the generic `Enum` class, the `Parse` method returns a generic object, so you have to ~~set Option Strict to Off (as in the previous snippet) or~~ use an explicit cast to assign it to a specific enumerated variable:

```FSharp
// You can use the GetType method (inherited from System.Object)
// to get the Type object required by the Parse method.
let de = Enum.Parse(typeof<DataEntry>, "CharString") :?> DataEntry
```

The `Parse` method throws an `ArgumentException` if the name doesn't correspond to a defined enumerated value. Names are compared in a case-sensitive way, but you can pass a true optional argument if you don't want to take the string case into account:

```FSharp
// *** This statement throws an exception.
Console.WriteLine(Enum.Parse(de.GetType(), "charstring"))
// This works well because case-insensitive comparison is used.
Console.WriteLine(Enum.Parse(de.GetType(), "charstring", true))
```

### Other `Enum` Methods

The `GetUnderlyingType` static method returns the base type for an enumerated class:

```FSharp
Console.WriteLine(Enum.GetUnderlyingType(de.GetType()))    // => System.Int32
```

The `IsDefined` method enables you to check whether a numeric value is acceptable as an enumerated value of a given class:

```FSharp
if Enum.IsDefined(typeof<DataEntry>, 3) then
   // 3 is a valid value for the DataEntry class.
   let de = enum<DataEntry>(3)
```

The `IsDefined` method is useful because the `enum<_>` operator doesn't check whether the value being converted is in the valid range for the target enumerated type. In other words, the following statement doesn't throw an exception:

```FSharp
// This code produces an invalid result, yet it doesn't throw an exception.
let de = enum<DataEntry>(123)
```

Another way to check whether a numeric value is acceptable for an `Enum` object is by using the `GetName` method, which returns the name of the enumerated value or returns null if the value is invalid:

```FSharp
if Enum.GetName(typeof<DataEntry>, 3) <> null then
   let de = enum<DataEntry>(3)
```

You can quickly list all the values of an enumerated type with the `GetNames` and `GetValues` methods. The former returns a String array holding the individual names (sorted by the corresponding values); the latter returns an Object array that holds the numeric values:

```FSharp
// List all the values in DataEntry.
let names: String[] = Enum.GetNames(typeof<DataEntry>)
let values: Array = Enum.GetValues(typeof<DataEntry>)
for i = 0 to names.Length - 1 do
    Console.WriteLine("{0} = {1}", names.[i], unbox<int>(values.GetValue(i)))
```

Here's the output of the preceding code snippet:

```FSharp
IntegerNumber = 0
FloatingNumber = 1
CharString = 2
DateTime = 3
```

### Bit-Coded Values

The .NET Framework supports a special `Flags` attribute that you can use to specify that an `Enum` object represents a bit-coded value. For example, let's create a new class named `ValidDataEntry` class, which enables the developer to specify two or more valid data types for values entered by an end user:

```FSharp
[<Flags>] 
type ValidDataEntry =
| None = 0              // Always define an Enum value equal to 0.
| IntegerNumber = 0b0001
| FloatingNumber = 0b0010
| CharString = 0b0100
| DateTime = 0b1000
```

The `FlagsAttribute` class doesn't expose any property, and its constructor takes no arguments: the presence of this attribute is sufficient to label this `Enum` type as bit-coded.

Bit-coded enumerated types behave exactly like regular `Enum` values do except their `ToString` method recognizes the `Flags` attribute. When an enumerated type is composed of two or more flag values, this method returns the list of all the corresponding values, separated by commas:

```FSharp
let vde: ValidDataEntry = ValidDataEntry.IntegerNumber ||| ValidDataEntry.DateTime
Console.WriteLine(vde.ToString())       // => IntegerNumber, DateTime
```

If no bit is set, the `ToString` method returns the name of the enumerated value corresponding to the zero value:

```FSharp
let vde2: ValidDataEntry = ...
Console.WriteLine(vde2.ToString())     // => None
```

If the value doesn't correspond to a valid combination of bits, the `Format` method returns the number unchanged:

```FSharp
vde = enum<ValidDataEntry>(123)
Console.WriteLine(vde.ToString())     // => 123
```

The `Parse` method is also affected by the `Flags` attribute:

```FSharp
let vde = Enum.Parse(vde.GetType(), "IntegerNumber, FloatingNumber") :?> ValidDataEntry
Console.WriteLine(int vde)        // => 3
```

.NET enum types are simple integer-like value types associated with a particular name. They're typically used for specifying flags to `APIs`; for example, ``FileMode`` in the ``System.IO`` namespace is an enum type with values such as ``FileMode.Open`` and ``FileMode.Create``. .NET enum types are easy to use from F# and can be combined using bitwise AND, OR, and XOR operations using the `&&&`, `|||`, and `^^^` operators. Most commonly, the `|||` operator is used to combine multiple flags. On occasion, you may have to mask an attribute value using `&&&` and compare the result to enum 0.

> You will see how to define .NET-compatible enum types in F# at the end of Chapter 6.

#### Bitwise Operators

Visual Basic supports four bitwise operators: `~~~`, `&&&`, `|||`, and `^^^`. These operators can be used only on signed and unsigned integers and combine the individual bits in their operands. These operators can be used to test or change one or more bits in a number:

```FSharp
// Test the rightmost (least significant) bit.
let number: Int32 = 12345
if (number &&& 1) = 1 then Console.WriteLine("Number is odd.")

// Clear the last eight bits (two equivalent ways).
let number = number &&& 0xFFFFFF00
let number = number &&& ~~~ 255

// Set the last four bits.
let number = number ||| 15

// Flip the most significant bit.
let number = number ^^^ 0x10000000
```

Not surprisingly, these operators are most useful with bit-coded values, that is, integer values that pack multiple pieces of information in their individual bits (or groups of bits), as is often the case with numbers passed to Windows API methods or values used to communicate with hardware devices. However, if you are familiar with binary math you can often use these operators to simplify or optimize operations and tests on regular integers. Here are a few examples:

```FSharp
// Round down to the nearest even number.
let number = if (number % 2) = 1 then number - 1 else number
// Alternative way (just clears the least significant bit)
let number = number &&& ~~~ 1

// Round up to the nearest odd number.
let number = if (number % 2) = 1 then number + 1 else number
// Alternative way (just sets the least significant bit)
let number = number ||| 1

// Test whether all the numbers in a group are zero.
let ok = a = 0 && b = 0 && c = 0
// Alternative way (relies on the fact that the result of Or is zero
// only if both its operands are zero)
let ok = (a ||| b ||| c) = 0

// Test whether all numbers in a group are positive.
let ok = a >= 0 && b >= 0 && c >= 0
// Alternative way (relies on the fact the sign bit of the result of Or
// is zero only if the sign bit of all its operands is also zero)
let ok = (a ||| b ||| c) >= 0

// Test whether all numbers in a group are negative.
let ok = a < 0 && b < 0 && c < 0
// Alternative way (relies on the fact the sign bit of the result of And
// is set only if the sign bit of all its operands is also set)
let ok = (a &&& b &&& c) < 0

// Test whether two numbers have the same sign.
let ok = (a >= 0 && b >= 0) || (a < 0 && b < 0)
// Alternative way (relies on the fact that the sign bit of the result from
// a Xor operator is zero if operands have same sign)
let ok = a ^^^ b >= 0
```

~~If you are a Visual Basic 6 developer, you have surely used `And` and `Or` operators to combine `Boolean` expressions in If statements. Visual Basic 2005 still supports these operators when used in this fashion, but it is recommended that you switch to the newer `AndAlso` and `OrElse` logical operators, which are illustrated next.~~



## Enumerated Types

The `enum` operator is a generic operator that takes one type parameter that represents the type of the `enum` to convert to. When it converts to an enumerated type, type inference attempts to determine the type of the `enum` that you want to convert to. In the following example, the variable `col1` is not explicitly annotated, but its type is inferred from the later equality test. Therefore, the compiler can deduce that you are converting to a `Color` enumeration. Alternatively, you can supply a type annotation, as with `col2` in the following example.

```fsharp
type Color =
    | Red = 1
    | Green = 2
    | Blue = 3

// The target type of the conversion cannot be determined by type inference, so the type parameter must be explicit.
let col1 = enum<Color> 1

// The target type is supplied by a type annotation.
let col2 : Color = enum 2
```

You can also specify the target enumeration type explicitly as a type parameter, as in the following code:

F#

```fsharp
let col3 = enum<Color> 3
```

Note that the enumeration casts work only if the underlying type of the enumeration is compatible with the type being converted. In the following code, the conversion fails to compile because of the mismatch between `int32` and `uint32`.

F#

```fsharp
// Error: types are incompatible
let col4 : Color = enum 2u
```

For more information, see [Enumerations](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/enumerations).