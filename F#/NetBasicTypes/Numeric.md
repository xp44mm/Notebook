## Numeric Types

I illustrated most of the operations you can perform on numbers in Chapter 2, "Basic Language Concepts." In this section, I complete the discussion showing you the members exposed by the basic numeric classes, particularly the methods related to parsing and formatting.

As you know, `Short`, `Integer`, and `Long` types are just aliases for the `Int16`, `Int32`, and `Int64` .NET classes. By recognizing their true nature and by using their methods and properties, you can better exploit these types. This section applies to all the numeric types in the .NET Framework, such as `Boolean`, `Byte`, `SByte`, Short (`Int16`), Integer (`Int32`), Long (`Int64`), UShort (`UInt16`), UInteger (`UInt32`), ULong (`UInt64`), `Single`, `Double`, and `Decimal`.

### Properties and Methods

All numeric types—and all .NET classes, for that matter—expose the `ToString` method, which converts their numeric value to a string. This method is especially useful when you're appending the number value to another string:

```FSharp
let myValue: Double = 123.45
let res: String = "The final value is " + myValue.ToString()
```

The `ToString` method is culturally aware and by default uses the culture associated with the current thread. For example, it uses a comma as a decimal separator if the current thread's culture is Italian or German. Numeric types overload the `ToString` method to take either a format string or a custom formatter object. (For more detail, refer to the section titled "Formatting Numeric Values" earlier in this chapter.)

```FSharp
// Convert an integer to hexadecimal.
Console.WriteLine(1234.ToString("X"))        // => 4D2
// Display PI with 6 digits (in all).
let d: Double = Math.PI
Console.WriteLine(d.ToString("G6"))          // => 3.14159六位有效数字
```

You can use the `CompareTo` method to compare a number with another numeric value of the same type. This method returns 1, 0, or -1, depending on whether the current instance is greater than, equal to, or less than the value passed as an argument:

```FSharp
let sngValue: Single = 1.23f
// Compare the Single variable sngValue with 1.
// Note that you must force the argument to Single.
match sngValue.CompareTo(single(1)) with
| 1 -> Console.WriteLine("sngValue is > 1")
| 0 -> Console.WriteLine("sngValue is = 1")
| -1 -> Console.WriteLine("sngValue is < 1")
| _ -> ()
```

The argument must be the same type as the value to which you're applying the `CompareTo` method, so you must convert it if necessary. ~~You can use a conversion function, such as the single function in the preceding code, or append a conversion character, such as ! for Single, I for Integer, and so on:~~

```FSharp
// …(Another way to write the previous code snippet)…
match sngValue.CompareTo(1.0f) with
…
```

All the numeric classes expose the `MinValue` and `MaxValue` static fields, which return the smallest and greatest value that you can express with the corresponding type:

```FSharp
// Display the greatest value you can store in a Double variable.
Console.WriteLine(Double.MaxValue)      // => 1.79769313486232E+308
```

The numeric classes that support floating-point values—namely, `Single` and `Double` classes—expose a few additional read-only static properties. The `Epsilon` property returns the smallest positive (nonzero) number that can be stored in a variable of that type:

```FSharp
Console.WriteLine(Single.Epsilon)        // => 1.401298E-45
Console.WriteLine(Double.Epsilon)        // => 4.94065645841247E-324
```

The `NegativeInfinity` and `PositiveInfinity` fields return a constant that represents an infinite value, whereas the `NaN` field returns a constant that represents the Not-a-Number value (`NaN` is the value you obtain, for example, when evaluating the square root of a negative number). In some cases, you can use infinite values in expressions:

```FSharp
// Any number divided by infinity gives 0.
Console.WriteLine(1 / Double.PositiveInfinity)      // => 0
```

The `Single` and `Double` classes also expose static methods that enable you to test whether they contain special values, such as `IsInfinity`, `IsNegativeInfinity`, `IsPositiveInfinity`, and `IsNaN`.

### Formatting Numbers

All the numeric classes support an overloaded form of the `ToString` method that enables you to apply a format string:

```FSharp
let intValue: Int32 = 12345
Console.WriteLine(intValue.ToString("##,##0.00"))   // => 12,345.00
```

The method uses the current locale to interpret the formatting string. For example, in the preceding code it uses the comma as the thousands separator and the period as the decimal separator if running on a U.S. system, but reverses the two separators on an Italian system. You can also pass a `CultureInfo` object to format a number for a given culture:

```FSharp
let ci = new CultureInfo("it-IT")
Console.WriteLine(intValue.ToString("##,##0.00", ci))     // => 12.345,00
```

The previous statement works because the `ToString` takes an `IFormatProvider` object to format the current value, and the `CultureInfo` object exposes this interface. In this section, I show you how you can take advantage of another .NET object that implements this interface, the `NumberFormatInfo` object.

The `NumberFormatInfo` class exposes many properties that determine how a numeric value is formatted, such as `NumberDecimalSeparator` (the decimal separator character), `NumberGroupSeparator` (the thousands separator character), `NumberDecimalDigits` (number of decimal digits), `CurrencySymbol` (the character used for currency), and many others. The simplest way to create a valid `NumberFormatInfo` object is by means of the `CurrentInfo` shared method of the `NumberFormatInfo` class; the returned value is a read-only `NumberFormatInfo` object based on the current locale:

```FSharp
let nfi: NumberFormatInfo = NumberFormatInfo.CurrentInfo
```

(You can also use the `InvariantInfo` property, which returns a `NumberFormatInfo` object that is culturally independent.)

The problem with the preceding code is that the returned `NumberFormatInfo` object is readonly, so you can't modify any of its properties. This object is therefore virtually useless because the `ToString` method implicitly uses the current locale anyway when formatting a value. The solution is to create a clone of the default `NumberFormatInfo` object and then modify its properties, as in the following snippet:

```FSharp
// Format a number with current locale formatting options, but use a comma
// for the decimal separator and a space for the thousands separator.
// (You need DirectCast because the Clone method returns an Object.)
let nfi: NumberFormatInfo = NumberFormatInfo.CurrentInfo.Clone() :?> NumberFormatInfo
// The nfi object is writable, so you can change its properties.
nfi.NumberDecimalSeparator <- ","
nfi.NumberGroupSeparator <- " "
// You can now format a value with the custom NumberFormatInfo object.
let sngValue: Single = 12345.5f
Console.WriteLine(sngValue.ToString("##,##0.00", nfi)) // => 12 345,50
```

For the complete list of `NumberFormatInfo` properties and methods, see the MSDN documentation.

### Parsing Strings into Numbers

All numeric types support the `Parse` static method, which parses the string passed as an argument and returns the corresponding numeric value. The simplest form of the `Parse` method takes one string argument:

```FSharp
// Next line assigns 1234 to the variable.
let shoValue: Short = Short.Parse("1234")
```

An overloaded form of the `Parse` method takes a `NumberStyle` enumerated value as its second argument. `NumberStyle` is a bit-coded value that specifies which portions of the number are allowed in the string being parsed. Valid `NumberStyle` values are `AllowLeadingWhite` (1), `AllowTrailingWhite` (2), `AllowLeadingSign` (4), `AllowTrailingSign` (8), `AllowParentheses` (16), `AllowDecimalPoint` (32), `AllowThousand` (64), `AllowExponent` (128), `AllowCurrencySymbol` (256), and `AllowHexSpecifier` (512). You can specify which portions of the strings are valid by using the `|||` bitwise operator on these values, or you can use some predefined compound values, such as `Any` (511, allows everything), `Integer` (7, allows trailing sign and leading/trailing white), `Number` (111, like `Integer` but allows thousands separator and decimal point), `Float` (167, like `Integer` but allows decimal separator and exponent), and `Currency` (383, allows everything except exponent).

The following example extracts a `Double` from a string and recognizes white spaces and all the supported formats:

```FSharp
let dblValue: Double = Double.Parse(" 1,234.56E6  ", NumberStyles.Any)
   // dblValue is assigned the value 1234560000.
```

You can be more specific about what is valid and what isn't:

```FSharp
let style: NumberStyles = NumberStyles.AllowDecimalPoint ||| NumberStyles.AllowLeadingSign
// This works and assigns –123.45 to sngValue.
let sngValue: Single = Single.Parse("-123.45", style)
Assert.Equal(sngValue,-123.45f)

let ex = Assert.Throws<FormatException>(fun () ->
    // This throws a FormatException because of the thousands separator.
    let _ = Single.Parse("12,345.67", style)
    ()
)
Assert.Equal(ex.Message, "Input string was not in a correct format.")
```

A third overloaded form of the `Parse` method takes any `IFormatProvider` object; thus, you can pass it a `CultureInfo` object:

```FSharp
// Parse a string according to Italian rules.
sngValue = Single.Parse("12.345,67", new CultureInfo("it-IT"))
```

**Version 2005 of VB or Version 2.0 of .NET** All the numeric types in .NET Framework 2.0 expose a new method named `TryParse`, which allows you to avoid time-consuming exceptions if a string doesn't contain a number in a valid format. ~~(This method is available in .NET Framework 1.1 only for the Double type.)~~ The `TryParse` method takes a variable by reference in its second argument and returns true if the parsing operation is successful:

```FSharp
let mutable intValue: Int32 = Int32.MinValue
if Int32.TryParse("12345", &intValue) then
    // intValue contains the result of the parse operation.
    ()
else
    // The string doesn't contain an integer value in a valid format.
    ()
```

A second overload of the `TryParse` method takes a `NumberStyles` enumerated value and an `IFormatProvider` object in its second and third arguments:

```FSharp
let style: NumberStyles = NumberStyles.AllowDecimalPoint ||| NumberStyles.AllowLeadingSign
let mutable aValue: Single = Single.MinValue
if Single.TryParse("-12.345,67", style, new CultureInfo("it-IT"), &aValue) then
   ()
```

### The Convert Type

The `System.Convert` class exposes several static methods that help in converting to and from the many data types available in .NET. In their simplest form, these methods can convert any base type to another type and are therefore equivalent to the conversion functions that Visual Basic offers:

```FSharp
// Convert the string "123.45" to a Double (same as float function).
let dblValue: Double = Convert.ToDouble("123.45")
```

The `Convert` class exposes many ToXxxx methods, one for each base type: `ToBoolean`, `ToByte`, `ToChar`, `ToDateTime`, `ToDecimal`, `ToDouble`, `ToInt16`, `ToInt32`, `ToInt64`, `ToSByte`, `ToSingle`, `ToString`, `ToUInt16`, `ToUInt32`, and `ToUInt64`:

```FSharp
// Convert a Double value to an integer (same as CInt function).
let intValue: Int32 = Convert.ToInt32(dblValue)
```

The ToXxxx methods that return an integer type—namely, `ToByte`, `ToSByte`, `ToInt16`, `ToInt32`, `ToInt64`, `ToUInt16`, `ToUInt32`, and `ToUInt64`—expose an overload that takes a string and a base and convert a string holding a number in that base. The base can only be `2`, `8`, `10`, or `16`:

```FSharp
// Convert from a string holding a binary representation of a number.
let result: Int32 = Convert.ToInt32("11011", 2)         // => 27
// Convert from an octal number.
let result = Convert.ToInt32("777", 8)                  // => 511
// Convert from an hexadecimal number.
let result = Convert.ToInt32("AC", 16)                  // => 172
```

You can perform the conversion in the opposite direction—that is, from an integer into the string representation of a number in a different base—by means of overloads of the `ToString` method:

```FSharp
// Determine the binary representation of a number.
let text: String = Convert.ToString(27, 2)               // => 11011
// Determine the hexadecimal representation of a number. (Note: result is lowercase.)
let text = Convert.ToString(172, 16)                     // => ac
```

The `Convert` class exposes two methods that make conversions to and from Base64-encoded strings a breeze. (This is the format used for Multipurpose Internet Mail Extensions (MIME) e-mail attachments.) The `ToBase64String` method takes an array of bytes and encodes it as a Base64 string. The `FromBase64String` method does the conversion in the opposite direction:

```FSharp
// An array of 16 bytes (two identical sequences of 8 bytes)
let b1: Byte[] = 
    [|12; 45; 213; 88; 11; 220; 34; 0; 12; 45; 213; 88; 11; 220; 34; 0|]
    |> Array.map(uint8)

// Convert it to a Base64 string.
let s64: String = Convert.ToBase64String(b1)
Assert.Equal(s64,"DC3VWAvcIgAMLdVYC9wiAA==")
// Convert it back to an array of bytes, and display it.
let b2: Byte[] = Convert.FromBase64String(s64)
Assert.Equal<Byte[]>(b1,b2)
```

A new option in .NET Framework 2.0 enables you to insert a line separator automatically every 76 characters in the value returned by a `ToBase64String` method:

```FSharp
let s64 = Convert.ToBase64String(b1, Base64FormattingOptions.InsertLineBreaks)
```

In addition, the `Convert` class exposes the `ToBase64CharArray` and `FromBase64CharArray` methods, which convert a `Byte` array to and from a `Char` array instead of a `String`. Finally, the class also exposes a generic `ChangeType` method that can convert (or at least, attempt to convert) a value to any other type. You must use the `typeof<>` operator to create the `System.Type` object to pass in the method's second argument:

```FSharp
// Convert a value to Double.
Console.WriteLine(Convert.ChangeType(value, typeof<Double>))
```

### Random Number Generator

~~Visual Basic 2005 still supports the time-honored Randomize statement and Rnd function for backward compatibility with Visual Basic 6,~~ but serious .NET developers should use the `System.Random` class instead. You can set the seed for random number generation in this class's constructor method:

```FSharp
// The argument must be a 32-bit integer.
let rand = new Random(12345)
```

When you pass a given seed number, you always get the same random sequence. To get different sequences each time you run the application, you can have the seed depend on the current time:

```FSharp
// You need these conversions because the Ticks property
// returns a 64-bit value that must be truncated to a 32-bit integer.
let rand = new Random(int(DateTime.Now.Ticks &&& int64 Int32.MaxValue))
```

Once you have an initialized `Random` object, you can extract random positive 32-bit integer values each time you query its `Next` method:

```FSharp
for i = 1 to 10 do
    let v = rand.Next()
    Console.WriteLine(sprintf "%d" v)
```

You can also pass one or two arguments to keep the return value in the desired range:

```FSharp
// Get a value in the range 0 to 1000.
let intValue: Int32 = rand.Next(1000)
// Get a value in the range 100 to 1,000.
let intValue = rand.Next(100, 1000)
```

The `NextDouble` method is ~~similar to the Rnd function in the Microsoft.VisualBasic library in~~ that it returns a random floating-point number between 0 and 1:

```FSharp
let dblValue: Double = rand.NextDouble()
```

Finally, you can fill a `Byte` array with random values with the `NextBytes` method:

```FSharp
// Get an array of 100 random byte values.
let mutable  buffer = Array.zeroCreate<Byte>(100)
rand.NextBytes(buffer)
```

---

##### Note

Although the `Random` type is OK in most kinds of applications, for example, when developing card games, the values it generates are easily reproducible and aren't random enough to be used in cryptography. For a more robust random value generator, you should use the `RNGCryptoServiceProvider` class, in the `System.Security.Cryptography` namespace.

---

