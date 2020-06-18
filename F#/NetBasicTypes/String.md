## String Types

As discussed in Chapter 2, Visual Basic supports the `String` data type, which maps to the `System.String` class. Because `System.String` is a full-featured type, you can manipulate strings by means of the many methods it exposes, a technique that in general provides more flexibility and better performance than does using the functions provided by the `Microsoft.VisualBasic` assembly, such as `Trim` or `Left`.

To begin with, the `String` class exposes many overloaded constructor methods, so you can create a string in a variety of ways—for example, as a sequence of *N* same characters:

```FSharp
// A sequence of <N> characters—similar to the VB6 String function
// (Note the c suffix to make "A" a Char rather than a String.)
let s = new String('A', 10)                // => AAAAAAAAAA
```

(This technique duplicates the functionality of the `String` function in Microsoft Visual Basic 6, which was dropped because `String` is now a reserved keyword.)

### Properties and Methods

The only properties of the `String` class are `Length` and `Chars`. The former returns the number of characters in the string; the latter returns the character at a given zero-based index:

```FSharp
let s: String = "ABCDEFGHIJ"
Console.WriteLine(s.Length)                 // => 10
// Note that index is always zero-based.
Console.WriteLine(s.Chars(3))               // => D
```

Sometimes the reference nature of the `String` type causes behaviors you might not anticipate. For example, consider this simple code:

```FSharp
let s: String = null
Console.WriteLine(s.Length)
```

You probably expect that the second statement displays the value zero, but this statement actually throws a `NullReferenceException` because the `String` object hasn't been initialized. A simple way to avoid this problem is to make a habit of initializing all `String` variables explicitly, as in this code:

```FSharp
let s: String = ""
```

You can tap the power of the `String` class by invoking one of its many methods. Visual Basic 2005 strings are richer in functionality than are Visual Basic 6 strings, and they enable you to adopt a more object-oriented, concise syntax in your applications. For example, see how much simpler and more readable the operation of inserting a substring is under Visual Basic 2005:

```FSharp
let s: String = "ABCDEFGHIJ"
// The VB6 way of inserting a substring after the third character
let ss = s.[0..2] + "1234" + s.[3..]
// The VB2005 object-oriented way to perform the same operation
let sss = s.Insert(3, "1234")      // => ABC1234DEFGHIJ
sss = ss
```

Here's another example of the compactness that you can get by using the `String` methods. Let's say you want to trim all the space and tab characters from the beginning of a string. With Visual Basic 2005, you simply have to load all the characters to be trimmed in an array of `Chars` and pass the array to the `TrimStart` function:

```FSharp
let cArr: Char[] = [|' '; '\t'|]
let s = s.TrimStart(cArr)
```

(You can use the same pattern with the `TrimEnd` and `Trim` functions.) In many cases, the Visual Basic 2005 methods can deliver better performance because you can state more precisely what you're after. For example, you can determine whether a string begins or ends with a given sequence of characters by a variety of means under Visual Basic 6, but none of the available techniques is especially efficient. Visual Basic 2005 strings solve this problem elegantly with the `StartsWith` or `EndsWith` method:

```FSharp
// Check whether the string starts with "abc" and ends with "xyz."
let ok = s.StartsWith("abc") && s.EndsWith("xyz")
```

You can iterate over all the characters of a string by using a For Each loop:

```FSharp
let s: String = "ABCDE"
for c: Char in s do
   Console.Write(sprintf "%c." c)        // => A.B.C.D.E.
```

**Version 2005 of VB or Version 2.0 of .NET** In earlier versions of the language, iterating over all the characters of a string by means of a For Each loop was a relatively slow process because it involved the creation of an `IEnumerator` object (see the section titled "The IEnumerable Interface" in Chapter 10). This kind of loop is much faster in Visual Basic 2005 because the compiler automatically translates the For Each loop into the following more efficient For…Next loop:

```FSharp
// This is how Visual Basic actually compiles a For Each loop over a string.
for index = 0 to s.Length - 1 do
   let c: Char = s.Chars(index)
   …
```

#### Comparing and Searching Strings

Many methods in the `String` type enable you to compare strings or search a substring inside the current string. In version 2.0 of the .NET Framework, the `String` type overloads both the instance and the static version of the `Equals` method to take an additional `StringComparison` enum type that specifies the locale to be used for the comparison and whether the comparison is case sensitive. In general, it is recommended that you use the static version of this method because it works well even if one or both the strings to be compared are `null`:

```FSharp
// Compare two strings in case-sensitive mode, using Unicode values.
let m: Boolean = String.Equals(s1, s2)

// Compare two strings in case-insensitive mode, using the current locale.
let m = String.Equals(s1, s2, StringComparison.CurrentCultureIgnoreCase)

// Compare two strings in case-sensitive mode, using the invariant locale.
let m = String.Equals(s1, s2, StringComparison.InvariantCulture)

// Compare the numeric Unicode values of all the characters in the two strings.
let m = String.Equals(s1, s2, StringComparison.Ordinal)
```

(Read on for more information about locales and the `CultureInfo` type.) In general, you should use the `Ordinal` and `OrdinalIgnoreCase` enumerated values if possible because they are more efficient. For example, these values can be fine when comparing file paths, registry keys, and XML and HTML tags.

The `InvariantCulture` and `InvariantCultureIgnoreCase` values are arguably the least useful ones and should be used in the rare cases when you compare strings that are linguistically meaningful but don't have a cultural meaning; using these values ensures that comparisons yield the same results on all machines, regardless of the culture of the current thread.

If you need to detect whether a string is less than or greater than another string, you should use the static `Compare` method, which can also compare substrings, in either case-sensitive or case-insensitive mode. The return value is -1, 0, or 1 depending on whether the first string is less than, equal to, or greater than the second string:

```FSharp
// Compare two strings in case-sensitive mode, using the current culture.
let res: Int32 = String.Compare(s1, s2)
match res with
| -1 -> Console.WriteLine("s1 < s2")
| 0 -> Console.WriteLine("s1 = s2")
| 1 -> Console.WriteLine("s1 > s2")
| _ -> failwith ""

// Compare the first 10 characters of two strings in case-insensitive mode.
// (Second and fourth arguments are the index of first char to compare.)
let res = String.Compare(s1, 0, s2, 0, 10, true)
```

In .NET Framework 2.0, you can also pass a `StringComparison` enum value to specify whether you want to use the invariant locale, the current locale, or the numeric Unicode value of individual characters, and whether you want the comparison to be in case-insensitive mode:

```FSharp
// Compare two strings using the local culture in case-insensitive mode.
let res = String.Compare(s1, s2, StringComparison.CurrentCultureIgnoreCase)
// Compare two substrings using the invariant culture in case-sensitive mode.
let res = String.Compare(s1, 0, s2, 0, 10, StringComparison.InvariantCulture)
// Compare two strings by the numeric code of their characters.
let res = String.Compare(s1, s2, StringComparison.Ordinal)
```

The `StringComparer` type (also new in .NET Framework 2.0) offers an alternative technique for doing string comparisons. The static properties of this type return an `IComparer` object able to perform a specific type of comparison on strings. For example, here's how you can perform a case-insensitive comparison according to the current culture:

```FSharp
// Compare two strings using the local culture in case-insensitive mode.
let res = StringComparer.CurrentCultureIgnoreCase.Compare(s1, s2)
// Compare two strings using the invariant culture in case-sensitive mode.
let res = StringComparer.InvariantCulture.Compare(s1, s2)
```

The `StringComparer` object has the added advantage that you can pass it to methods that take an `IComparer` argument, such as the `Array.Sort` method, but it has some shortcomings, too; for one, you can't use the `StringComparer.Compare` method to compare substrings.

The `String` type exposes also the `CompareTo` instance method, which compares the current string to the passed argument and returns -1, 0, or 1, exactly as the static `Compare` method does. However, the `CompareTo` method doesn't offer any of the options of the `Compare` method and therefore should be avoided unless you really mean to compare using the current culture's rules. In addition, because it is an instance method, you should always check that the instance isn't `null` before invoking the method:

```FSharp
// This statement throws an exception if s1 is Nothing.
if s1.CompareTo(s2) > 0 then Console.WriteLine("s1 > s2")
```

When comparing just the numeric Unicode value of individual characters, you can save a few CPU cycles by using the `CompareOrdinal` static method; in general, however, you should use this method only to test equality because it seldom makes sense to decide whether a string is greater than or less than another string based on Unicode numeric values:

```FSharp
if String.CompareOrdinal(s1, s2) = 0 then Console.WriteLine("s1 = s2")
```

A .NET string variable is considered to be *null* if it is `null` and is considered to be *empty* if it points to a zero-character string. In Visual Basic .NET 2003, you must test these two conditions separately, but .NET Framework 2.0 introduces the handy `IsNullOrEmpty` static method:

```FSharp
// These two statements are equivalent, but only the latter works in Visual Basic 2005.
let s: String = "ABCDE"
if s = null || s.Length = 0 then Console.WriteLine("Empty string")
if String.IsNullOrEmpty(s) then Console.WriteLine("Empty string")
```

The simplest way to check whether a string appears inside another string is by means of the `Contains` method, also new in .NET Framework 2.0:

```FSharp
// The Contains method works only in case-sensitive mode.
let s = "ABCDEFGHI ABCDEF"
let found: Boolean = s.Contains("BCD")           // => True
let found = s.Contains("bcd")                    // => False
```

You can detect the actual position of a substring inside a string by means of the `IndexOf` and `LastIndexOf` methods, which return the index of the first and last occurrence of a substring, respectively, or -1 if the search fails:

```FSharp
let pos: Int32 = s.IndexOf("CDE")            // => 2
let pos = s.LastIndexOf("CDE")                       // => 12
// Both IndexOf and LastIndexOf are case sensitive by default.
let pos = s.IndexOf("cde")                           // => -1
// but they offer an overload that can specify case-insensitivity.
let pos = s.LastIndexOf("cde", StringComparison.CurrentCultureIgnoreCase)  // =>12
```

The `StartsWith` and `EndsWith` methods enable you to check quickly whether a string starts with or ends with a given substring. By default, these methods perform a case-sensitive comparison using the current culture:

```FSharp
let m = s.StartsWith("ABC")                      // => True
let m = s.EndsWith("def")                        // => False
```

In .NET Framework 2.0, these two methods have been expanded to support a `StringComparison` argument and can take a `CultureInfo` object as a third argument so that you can specify the locale to be used when comparing characters and whether the comparison is case insensitive:

```FSharp
// Both these statements assign True to the variable.
let m = s.StartsWith("abc", StringComparison.CurrentCultureIgnoreCase)
let m = s.EndsWith("CDE", true, CultureInfo.InvariantCulture)
```

The `IndexOfAny` and `LastIndexOfAny` methods return the first and last occurrence, respectively, of a character among those in the specified array. Both these methods can take an optional starting index in the second argument and a character count in the third argument:

```FSharp
let chars: Char[] = [|'D'; 'F'; 'I'|]
let pos = s.IndexOfAny(chars)                     // => 3
let pos = s.LastIndexOfAny(chars)                 // => 15
let pos = s.IndexOfAny(chars, 6)                  // => 8
let pos = s.IndexOfAny(chars, 6, 2)               // => -1
```

#### Modifying and Extracting Strings

The simplest way to create a new string from an existing string is by means of the `Substring` method, which extracts a substring starting at a given index and with the specified number of arguments. This method corresponds therefore to the Mid function in the Microsoft.VisualBasic library, and you can also use it to simulate the Right function:

```FSharp
let s: String = "ABCDEFGHI ABCDEF"
// Extract the substring after the 11th character. Same as Mid(s, 11)
let result: String = s.Substring(10)        // => ABCDEF
// Extract 4 characters after the 11th character. Same as Mid(s, 11, 4)
let result = s.Substring(10, 4)                   // => ABCD
// Extract the last 4 characters. Same as Right(s, 4)
let result = s.Substring(s.Length - 4)            // => CDEF
```

The `Insert` method returns the new string created by inserting a substring at the specified index, whereas the `Remove` method removes a given number of characters, starting at the specified index:

```FSharp
let result = s.Insert(4, "-123-")                  // => ABCD-123-EFGHI ABCDEF
let result = s.Remove(4, 3)                        // => ABCDHI ABCDEF
```

In .NET Framework 2.0, the `Remove` method has been overloaded with a version that takes only the start index; you can use this new version as a surrogate for the "classic" Left function:

```FSharp
// Extract the first 4 characters. Same as Left(s, 4)
let result = s.Remove(4)                       // => ABCD
```

I already hinted at the `TrimStart` method previously. This method, together with the `TrimEnd` and `Trim` methods, enables you to discard spaces and other characters that are found at the left side, the right side, or both sides of a string. By default these methods trim white-space characters (that is, spaces, tabs, and newlines), but you can pass one or more arguments to specify which characters have to be trimmed:

```FSharp
let t: String = " 001234.560 "
let result = t.Trim()                        // => "001234.560"
let result = t.TrimStart(' ', '0')           // => "1234.560 "
let result = t.TrimEnd(' ', '0')             // => "  001234.56"
let result = t.Trim(' ', '0')                // => "1234.56"
```

The opposite operation of trimming is padding. You can pad a string with a given character, either to the left or the right, to bring the string to the specified length, by means of the `PadLeft` and `PadRight` methods. The most obvious reason for using these methods is to align strings and numbers:

```FSharp
// Right-align a number in a field that is 8-char wide.
let number: Int32 = 1234
let result = number.ToString().PadLeft(8)        // => "   1234"
```

As their name suggests, the `ToLower` and `ToUpper` methods return a new string obtained by converting all the characters of a string to lowercase or uppercase. Version 2.0 of the .NET Framework also provides the new `ToLowerInvariant` and `ToUpperInvariant` methods, which convert a string by using the casing rules of the invariant culture:

```FSharp
// Convert the s string to lowercase, using the current culture's rules.
let result = s.ToLower()
// Convert the s string to uppercase, using the invariant culture's rules.
let result = s.ToUpperInvariant()
```

Microsoft guidelines suggest that you use the `ToUpperInvariant` method when preparing a normalized string for comparison in case-insensitive mode because this is the way the `Compare` method works internally when you use the `StringComparison.InvariantCultureIgnoreCase`. As odd as it may sound, under certain cultures converting two strings to uppercase and then comparing them might deliver a different result from the one you receive if you convert them to lowercase before the comparison.

The `Replace` method replaces all the occurrences of a single character or a substring with another character or a substring. All the searches are performed in case-sensitive mode:

```FSharp
let k: String = "ABCDEFGHI ABCDEF"
let result = k.Replace("BCDE", "--")          // => A--FGHI A--F
let result = k.Replace(' ', ',')
```

#### Working with String and Char Arrays

A few methods take or return an array of `String` or `Char` elements. For example, the `ToCharArray` method returns an array containing all the characters in the string. You typically use this method when processing the characters separately is more efficient than extracting them from the string is. For example, consider the problem of checking whether two strings contain the same characters, regardless of their order (in other words, whether one string is the anagram of the other).

```FSharp
let s1: String = "file"
let s2: String = "life"
// Transform both strings to an array of characters.
let chars1: Char[] = s1.ToCharArray()
let chars2: Char[] = s2.ToCharArray()
// Sort both arrays.
Array.Sort(chars1)
Array.Sort(chars2)
// Build two new strings from the sorted arrays, and compare them.
let sorted1 = new String(chars1)
let sorted2 = new String(chars2)
// Compare them. (You can use case-insensitive comparison, if necessary.)
let m: Boolean = String.Compare(sorted1, sorted2) = 0     // => True
```

You can split a string into an array of substrings by using the `Split` method, which takes a list of separators and an optional maximum number of elements in the result array. This method has been improved in .NET Framework 2.0 and it now has the ability to process separators of any length and to optionally drop empty elements in the result array:

```FSharp
let x: String = "Hey, Visual Basic Rocks!"
let arr: String[] = x.Split(' ', ',', '.')
// The result contains the words "Hey", "", "Visual", "Basic", "Rocks!"
let numOfWords: Int32 = arr.Length            // => 5

// Same as before, but no more than 100 elements, and drop empty items.
let separators: Char[] = [|' '; ','; '.'|]
let arr2: String[] = x.Split(separators, 100, StringSplitOptions.RemoveEmptyEntries)
// The result contains the words "Hey", "Visual", "Basic", "Rocks!"
let numOfWords = arr2.Length                 // => 4
```

The new ability of using multi-character strings as a separator is quite useful when you need to retrieve the individual lines of a string containing pairs of carriage return-line feeds (CR-LF):

```FSharp
// Count the number of nonempty lines in a text file.
let crlfs: String[] = [|Environment.NewLine|]
let lines: String[] = File.ReadAllText(@"c:\data.txt").Split(crlfs, StringSplitOptions.None)
let numOfLines: Int32 = lines.Length
```

You can perform the opposite operation, that is, concatenating all the elements of a string array, by using the `Join` static method. This method optionally takes the initial index and the maximum number of elements to be considered.

```FSharp
// (Continuing the previous example…)
// Reassemble the string by adding CR-LF pairs, but skip the first array element.
let newText: String = String.Join(Environment.NewLine, lines, 1, lines.Length - 1)
```

### The Missing Methods

In spite of the large number of methods exposed by the `String` type, at times you must write your own function to accomplish recurring tasks. For example, there is no built-in method similar to `PadLeft` or `PadRight` but capable of centering a string in a field of a given size. Here's the code that performs this task:

```FSharp
let PadCenter(s : String, width : Int32,  padChar : Char,  truncate:bool) : String =
    let diff: Int32 = width - s.Length
    if diff = 0 || (diff < 0 && not truncate) then
        // Return the string as is.
        s
    elif diff < 0 then
        // Truncate the string.
        s.Substring(0, width)
    else
        // Half of the extra chars go to the left, the remaining ones go to the right.
        s.PadLeft(width - diff / 2, padChar).PadRight(width, padChar)
```

The `Microsoft.VisualBasic` library contains a few functions that don't have a corresponding method in the `String` type. For example, there is no native .NET method similar to `StrReverse`, which reverses the characters in a string. Here's the remedy, in the form of a custom method that also enables you to reverse only a portion of the string:

```FSharp
let StringReverse(s : String, startIndex : Int32, count : Int32) : String =
    let chars: Char[] = s.ToCharArray()
    let count = if count < 0 then s.Length - startIndex else count
    Array.Reverse(chars, startIndex, count)
    new String(chars)
```

Occasionally, you might need to duplicate a string a given number of times. The Microsoft.VisualBasic library offers the `StrDup` function, but this function works only with one-character strings. Here's a better method based on the `CopyTo` method that enables you to duplicate a string of any length:

```FSharp
let StringDuplicate(s : String, count : Int32) : String =
   // Prepare a character array of given length.
   let chars = Array.zeroCreate<Char>(s.Length * count - 1)
   // Copy the string into the array multiple times.
   for i = 0 to count - 1 do
      s.CopyTo(0, chars, i * s.Length, s.Length)
   new String(chars)
```

Or you can use the following one-liner, which builds a string of spaces whose length is equal to the number of repetitions and then replaces each space with the string to be repeated:

```FSharp
// Repeat the Text string a number of times equal to Count.
let dupstring: String = new String(' ', Count).Replace(" ", Text)
```

You can use the `IndexOf` method in a loop to count the number of occurrences of a substring:

```FSharp
let CountSubstrings(source : String, search : String) : Int32 =
    let mutable count: Int32 = -1
    let mutable index: Int32 = -1
    while index < 0 do
        count <- count + 1
        index <- source.IndexOf(search, index + 1)
    count
```

You can also calculate the number of occurrences of a substring with the following technique, which is more concise but slightly less efficient than is the previous method because it creates two temporary strings:

```FSharp
let count = source.Replace(search, search + "*").Length - source.Length
```

### String Optimizations

An important detail to keep in mind is that a `String` object is *immutable*: once you create a string, its contents can never change. In fact, all the methods shown so far don't modify the original `String`; rather, they return another `String` object that you might or might not assign to the same `String` variable. Understanding this detail enables you to avoid a common programming mistake:

```FSharp
let s: String = "abcde"
// You *might* believe that next statement changes the string…
s.ToUpper()
// but it isn't the case because the result wasn't assigned back to s.
Console.WriteLine(s)                 // => abcde
// This is the correct way to invoke string methods.
let s = s.ToUpper()
```

If you do assign the result to the same variable, the original string becomes unreachable from the application (unless there are other variables pointing to it) and will eventually be disposed of by the garbage collector. Because `String` values are immutable, the compiler can optimize the resulting code in ways that wouldn't be possible otherwise. For example, consider this code fragment:

```FSharp
let s1: String = "1234" + "5678"
let s2: String = "12345678"
Console.WriteLine(s1 = s2)                     // => True
```

The compiler computes the concatenation (+) operator at compile time and realizes that both variables contain the same sequence of characters, so it can allocate only one block of memory for the string and have the two variables pointing to it. Because the string is immutable, a new object is created behind the scenes as soon as you attempt to modify the characters in that string:

```FSharp
// …(Continuing the previous code fragment)…
// Attempt to modify the S1 string using the Mid statement.
Mid(s1, 2, 1) = "x"
// Prove that a new string was created behind the scenes.
Console.WriteLine(s1 = s2)                     // => False
```

Because of this behavior, you never really need to invoke the `Clone` method to explicitly create a copy of the string. Simply use the string as you would normally, and the compiler creates a copy for you if and when necessary.

The CLR can optimize string management by maintaining an internal pool of string values known as an *intern pool* for each .NET application. If the value being assigned to a string variable coincides with one of the strings already in the intern pool, no additional memory is created and the variable receives the address of the string value in the pool. As shown earlier, the compiler is capable of using the intern pool to optimize string initialization and have two string variables pointing to the same `String` object in memory. This optimization step isn't performed at run time, though, because the search in the pool takes time and in most cases it would fail, adding overhead to the application without bringing any benefit.

```FSharp
// Prove that no optimization is performed at run time.
s1 = "1234"
s1 <- s1 + "5678"
s2 = "12345678"
// These two variables point to different String objects.
Console.WriteLine(s1 = s2)                     // => False
```

You can optimize string management by using the `Intern` static method. This method searches a string value in the intern pool and returns a reference to the pool element that contains the value if the value is already in the pool. If the search fails, the string is added to the pool and a reference to it is returned. Notice how you can "manually" optimize the preceding code snippet by using the `String.Intern` method:

```FSharp
s1 = "ABCD"
s1 <- s1 + "EFGH"
// Move S1 to the intern pool.
s1 = String.Intern(s1)
// Assign S2 a string constant (that we know is in the pool).
s2 = "ABCDEFGH"
// These two variables point to the same String object.
Console.WriteLine(s1 = s2)                     // => True
```

This optimization technique makes sense only if you're working with long strings that appear in multiple portions of the applications. Another good time to use this technique is when you have many instances of a server-side component that contain similar string variables, such as a database connection string. Even if these strings don't change during the program's lifetime, they're usually read from a file, and, therefore, the compiler can't optimize their memory allocation automatically. Using the `Intern` method, you can help your application produce a smaller memory footprint. You can also use the `IsInterned` static method to check whether a string is in the intern pool (in which case the string itself is returned) or not (in which case the method returns `null`):

```FSharp
// Continuing previous example…
if String.IsInterned(s1) <> null then
   // This block is executed because s1 is in the intern pool.
```

Here's another simple performance tip: try to gather multiple string concatenations in the same statement instead of spreading them across separate lines. The Visual Basic compiler can optimize multiple concatenation operations only if they're in the same statement.

### The CultureInfo Type

The `System.Globalization.CultureInfo` class defines an object that you can inspect to determine some key properties of any installed languages. You can also use the object as an argument in many methods of the `String` and other types. The class exposes the `CurrentCulture` static property, which returns the `CultureInfo` object for the current language:

```FSharp
// Get information about the current locale.
let ci: CultureInfo = CultureInfo.CurrentCulture
// Assuming that the current language is Italian, we get:
Console.WriteLine(ci.Name)                            // => it
Console.WriteLine(ci.EnglishName)                     // => Italian
Console.WriteLine(ci.NativeName)                      // => italiano
Console.WriteLine(ci.LCID)                            // => 16
Console.WriteLine(ci.TwoLetterISOLanguageName)        // => it
Console.WriteLine(ci.ThreeLetterISOLanguageName)      // => ita
Console.WriteLine(ci.ThreeLetterWindowsLanguageName)  // => ITA
```

You can get additional information about the locale through the `TextInfo` object, exposed by the property with the same name:

```FSharp
let ti: TextInfo = ci.TextInfo
Console.WriteLine(ti.ANSICodePage)                     // => 1252
Console.WriteLine(ti.EBCDICCodePage)                   // => 20280
Console.WriteLine(ti.OEMCodePage)                      // => 850
Console.WriteLine(ti.ListSeparator)                    // => ;
```

The `CultureInfo` object exposes two properties, `NumberFormat` and `DateTimeFormat`, which return information about how numbers and dates are formatted according to a given locale. For example, consider this code:

```FSharp
// How do you spell "Sunday" in German?
// First create a CultureInfo object for German/Germany.

// (Note that you must pass a string in the form "locale-COUNTRY" if
// a given language is spoken in multiple countries.)
let ciDe = new CultureInfo("de-DE")
// Next get the corresponding DateTimeFormatInfo object.
let dtfi: DateTimeFormatInfo = ciDe.DateTimeFormat
// Here's the answer.
Console.WriteLine(dtfi.GetDayName(DayOfWeek.Sunday))     // => Sonntag
```

You'll find the "locale-COUNTRY" strings in many places of the .NET Framework. The `GetCultures` static method returns an array of all the installed cultures, so you can inspect all the languages that your operating system supports:

```FSharp
// Get info on all the installed cultures.
let ciArr: CultureInfo[] = CultureInfo.GetCultures(CultureTypes.AllCultures)
// Print abbreviation and English name of each culture.
for c: CultureInfo in ciArr do
   Console.WriteLine("{0} ({1})", c.Name, c.EnglishName)
```

**Version 2005 of VB or Version 2.0 of .NET** The `GetCultureInfo` static method, new in version 2.0, enables you to retrieve a cached, readonly version of a `CultureInfo` object. When you repeatedly use this method to ask for the same culture, the same cached `CultureInfo` object is returned, thus saving instantiation time:

```FSharp
let ci1: CultureInfo = CultureInfo.GetCultureInfo("it-IT")
let ci2: CultureInfo = CultureInfo.GetCultureInfo("it-IT")
// Prove that the second call returned a cached object.
Console.WriteLine(ci1 = ci2)                         // => True
```

The auxiliary `TextInfo` object permits you to convert a string to uppercase, lowercase, or title case (for example, "These Are Four Words") for a given language:

```FSharp
// Create a CultureInfo object for Canadian French. (Use a cached object if possible.)
let ciFr: CultureInfo = CultureInfo.GetCultureInfo("fr-CA")
// Convert a string to title case using Canadian French rules.
let s = ciFr.TextInfo.ToTitleCase(s)
```

Most of the string methods whose result depends on the locale accept a `CultureInfo` object as an argument, namely, `Compare`, `StartsWith`, `EndsWith`, `ToLower`, and `ToUpper`. ~~(This feature is new to .NET Framework 2.0 for the last four methods.)~~ Let's see how you can pass this object to the `String.Compare` method so that you can compare strings according to the collation rules defined by a given language. One overloaded version of the `Compare` method takes four arguments: the two strings to be compared, a `Boolean` value that indicates whether the comparison is case insensitive, and a `CultureInfo` object that specifies the language to be used:

```FSharp
// Compare two strings in case-insensitive mode according to rules of Italian language.
let s1: String = "cioè"
let s2: String = "CIOÈ"
// You can create a CultureInfo object on the fly.
if String.Compare(s1, s2, true, new CultureInfo("it")) = 0 then
   Console.WriteLine("s1 = s2")
```

Here also is an overloaded version that compares two substrings:

```FSharp
if String.Compare(s1, 1, s2, 1, 4, true, new CultureInfo("it")) = 1 then
   Console.WriteLine("s1's first four chars are greater than s2's")
```

If you don't pass any `CultureInfo` object to the `Compare` method, the comparison is performed using the locale associated with the current thread. You can change this locale value by assigning a `CultureInfo` object to the `CurrentCulture` property of the current thread, as follows:

```FSharp
// Use Italian culture for all string operations and comparisons.
Thread.CurrentThread.CurrentCulture <- new CultureInfo("it-IT")
```

You can also compare values according to an invariant culture so that the order in which your results are evaluated is the same regardless of the locale of the current thread. In this case, you can pass the return value of the `CultureInfo.InvariantCulture` static property:

```FSharp
if String.Compare(s1, s2, true, CultureInfo.InvariantCulture) = 0 then …
```

.NET Framework 2.0 offers the new `StringComparison` enumerated type that enables you to perform comparisons and equality tests in using the current culture, the invariant culture, and the numeric values of individual characters, both in case-sensitive and case-insensitive ways. For more details and examples, read the section titled "Comparing and Searching Strings" earlier in this chapter.

### The Encoding Class

All .NET strings store their characters in Unicode format, so sometimes you might need to convert them to and from other formats—for example, ASCII or the UCS Transformation Format 7 (UTF-7) or UTF-8 variants of the Unicode format. You can do this with the `Encoding` class in the `System.Text` namespace.

The first thing to do when converting a .NET Unicode string to or from another format is create the proper encoding object. The `Encoding` class opportunely exposes the most common encoding objects through the following static properties: `ASCII`, `Unicode` (little-endian byte order), `BigEndianUnicode`, `UTF7`, `UTF8`, `UTF32`, and `Default` (the system's current ANSI code page). Here's an example of how you can convert a Unicode string to a sequence of bytes that represent the same string in ASCII format:

```FSharp
let text: String = "A Unicode string with accented vowels: àèéìòù"
let uni: Encoding = Encoding.Unicode
let uniBytes: Byte[] = uni.GetBytes(text)
let ascii: Encoding = Encoding.ASCII
let asciiBytes: Byte[] = Encoding.Convert(uni, ascii, uniBytes)

// Convert the ASCII bytes back to a string.
let asciiText: String = String(ascii.GetChars(asciiBytes))
Console.WriteLine(asciiText)  // => A Unicode string with accented vowels: ?????
```

You can also create other `Encoding` objects with the `GetEncoding` static method, which takes either a code page number or code page name and throws a `NotSupportedException` if the code page isn't supported:

```FSharp
// Get the encoding object for code page 1252.
let enc: Encoding = Encoding.GetEncoding(1252)
```

The `GetEncodings` method (new in .NET Framework 2.0) returns an array of `EncodingInfo` objects, which provide information on all the `Encoding` objects and code pages installed on the computer:

```FSharp
for ei: EncodingInfo in Encoding.GetEncodings() do
    Console.WriteLine("Name={0}, DisplayName={1}, CodePage={2}", ei.Name, ei.DisplayName, ei.CodePage)
```

The `GetChars` method expects that the byte array you feed it contains an integer number of characters. (For example, it must end with the second byte of a two-byte character.) This requirement can be a problem when you read the byte array from a file or from another type of stream, and you're working with a string format that allows one, two, or three bytes per character. In such cases, you should use a `Decoder` object, which remembers the state between consecutive calls. For more information, read the MSDN documentation.

### Formatting Numeric Values

The `Format` static method of the `String` class enables you to format a string and include one or more numeric or date values in it, in a way similar to the `Console.Write` method. The string being formatted can contain placeholders for arguments, in the format {N} where *N* is an index that starts at 0:

```FSharp
// Print the value of a string variable.
let xyz: String = "foobar"
let msg = String.Format("The value of {0} variable is {1}.", "XYZ", xyz)
   // => The value of XYZ variable is foobar.
```

If the argument is numeric, you can add a colon after the argument index and then a character that indicates what kind of formatting you're requesting. The available characters are `G` (general), `N` (number), `C` (currency), `D` (decimal), `E` (scientific), `F` (fixed-point), `P` (percent), `R` (round-trip), and `X` (hexadecimal):

```FSharp
// Format a Currency according to current locale.
let msg = String.Format("Total is {0:C}, balance is {1:C}", 123.45, -67)
   // => Total is $123.45, balance is ($67.00)
```

The number format uses commas—or to put it more precisely, the thousands separator defined by the current locale—to group digits:

```FSharp
let msg = String.Format("Total is {0:N}", 123456.78)    // => Total is 123,456.78
```

You can append an integer after the N character to round or extend the number of digits after the decimal point:

```FSharp
let msg = String.Format("Total is {0:N4}", 123456.785555)   // => Total is 123,456.7856
```

The decimal format works with integer values only and throws a `FormatException` if you pass a noninteger argument; you can specify a length that, if longer than the result, causes one or more leading zeros to be added:

```FSharp
let msg = String.Format("Total is {0:D8}", 123456)           // => Total is 00123456
```

The fixed-point format is useful with decimal values, and you can indicate how many decimal digits should be displayed (two if you omit the length):

```FSharp
let msg = String.Format("Total is {0:F3}", 123.45678)        // => Total is 123.457
```

The scientific (or exponential) format displays numbers as `n.mmmE+eeee`, and you can control how many decimal digits are used in the mantissa portion:

```FSharp
let msg = String.Format("Total is {0:E}", 123456.789)    // => Total is 1.234568E+005
let msg = String.Format("Total is {0:E3}", 123456.789)   // => Total is 1.235E+005
```

The general format converts to either fixed-point or exponential format, depending on which format delivers the most compact result:

```FSharp
let msg = String.Format("Total is {0:G}", 123456)    // => Total is 123456
let msg = String.Format("Total is {0:G4}", 123456)   // => Total is 1.235E+05
```

The percent format converts a number to a percentage with two decimal digits by default, using the format specified for the current culture:

```FSharp
let msg = String.Format("Percentage is {0:P}", 0.123)   // => Total is 12.30 %
```

The round-trip format converts a number to a string containing all significant digits so that the string can be converted back to a number later without any loss of precision:

```FSharp
// The number of digits you pass after the "R" character is ignored.
let msg = String.Format("Value of PI is {0:R}", Math.PI)
   // => Value of PI is 3.1415926535897931
```

Finally, the hexadecimal format converts numbers to hexadecimal strings. If you specify a length, the number is padded with leading zeros if necessary:

```FSharp
msg = String.Format("Total is {0:X8}", 65535)    // => Total is 0000FFFF
```

You can build custom format strings by using a few special characters, whose meaning is summarized in Table 12-1. Here are a few examples:

```FSharp
let msg = String.Format("Total is {0:##,###.00}", 1234.567)    // => Total is 1,234.57
let msg = String.Format("Percentage is {0:##.000%}", .3456)    // => Percentage is 34.560%

// An example of prescaler
let msg = String.Format("Length in {0:###,.00}", 12344)      // => Total is 12.34

// Two examples of exponential format
let msg = String.Format("Total is {0:#.#####E+00}", 1234567)  // => Total is 1.23457E+06
let msg = String.Format("Total is {0:#.#####E0}", 1234567)    // => Total is 1.23457E6

// Two examples with separate sections
let msg = String.Format("Total is {0:##;<##>}", -123)         // => Total is <123>
let msg = String.Format("Total is {0:#;(#);zero}", 1234567)   // => Total is 1234567
```

In some cases, you can use two or three sections to avoid If or Select Case logic. For example, you can replace the following code:

```FSharp
if n1 > n2 then
   msg <- "n1 is greater than n2"
elif n1 < n2 then
   msg <- "n1 is less than n2"
else
   msg <- "n1 is equal to n2"

```

with the more concise but somewhat more cryptic code:

```FSharp
let msg = String.Format("n1 is {0:greater than;less than;equal to} n2", n1 – n2)
```

A little-known feature of the `String.Format` method—as well as all the methods that use it internally, such as the `Console.Write` method—is that it enables you to specify the width of a field and decide to align the value to the right or the left:

```FSharp
// Build a table of numbers, their square, and their square root.
// This prints the header of the table.
Console.WriteLine("{0,-5} | {1,7} | {2,10:N2}", "N", "N^2", "Sqrt(N)")
for n in [1. .. 100.] do
   // N is left-aligned in a field 5-char wide,
   // N^2 is right-aligned in a field 7-char wide, and Sqrt(N) is displayed with
   // 2 decimal digits and is right-aligned in a field 10-char wide.
   Console.WriteLine("{0,-5} | {1,7} | {2,10:N2}", n, n ** 2., Math.Sqrt(n))
```

You specify the field width after the comma symbol; use a positive width for right-aligned values and a negative width for left-aligned values. If you want to provide a predefined format, use a colon as a separator after the width value. As shown in the previous example, field widths are also supported with numeric, string, and date values.

##### Table 12-1: Special Formatting Characters in Custom Formatting Strings

| Format | Description |
| ---------- | ------------------------------------------------------------ |
| # | Placeholder for a digit or a space. |
| 0 | Placeholder for a digit or a zero. |
| . | Decimal separator. |
| , | Thousands separator; if used immediately before the decimal separator, it works as a prescaler. (For each comma in this position, the value is divided by 1,000 before formatting.) |
| % | Displays the number as a percentage value. |
| E+000 | Displays the number in exponential format, that is, with an E followed by the sign of the exponent, and then a number of exponent digits equal to the number of zeros after the plus sign. |
| E-000 | Like the previous exponent symbol, but the exponent sign is displayed only if negative. |
| ; | Section separator. The format string can contain one, two, or three sections. If there are two sections, the first applies to positive and zero values, and the second applies to negative values. If there are three sections, they are used for positive, negative, and zero values, respectively. |
| \char | Escape character, to insert characters that otherwise would be taken as special characters (for example, \; to insert a semicolon and \\ to insert a backslash). |
| '…' "…" | A group of literal characters. You can add a sequence of literal characters by enclosing them in single or double quotation marks. |
| Other | Any other character is taken literally and inserted in the result string as is. |

Finally, you can insert literal braces by doubling them in the format string:

```FSharp
Console.WriteLine(" {{{0}}}", 123)           // => {123}
```

### Formatting Date Values

The `String.Format` method also supports date and time values with both standard and custom formats. Table 12-2 summarizes all the standard date and time formats and makes it easy to find the format you're looking for at a glance.

##### Table 12-2: Standard Formats for Date and Time Values [^1] 

| Format | Description                                                  | Pattern                            | Example                               |
| ------ | ------------------------------------------------------------ | ---------------------------------- | ------------------------------------- |
| d      | ShortDatePattern                                             | MM/dd/yyyy                         | 1/6/2005                              |
| D      | LongDatePattern                                              | dddd, MMMM dd, yyyy                | Thursday, January 06, 2005            |
| f      | Full date and time (long date and short time)                | dddd, MMMM dd, yyyy HH:mm          | Thursday, January 06, 2005 3:54 PM    |
| F      | FullDateTimePattern (long date and long time)                | dddd, MMMM dd, yyyy HH:mm:ss       | Thursday, January 06, 2005 3:54:20 PM |
| g      | general (short date and short time)                          | MM/dd/yyyy HH:mm                   | 1/6/2005 3:54 PM                      |
| G      | General (short date and long time)                           | MM/dd/yyyy HH:mm:ss                | 1/6/2005 3:54:20 PM                   |
| M,m    | MonthDayPattern                                              | MMMM dd                            | January 06                            |
| Y,y    | YearMonthPattern                                             | MMMM, yyyy                         | January, 2005                         |
| t      | ShortTimePattern                                             | HH:mm                              | 3:54 PM                               |
| T      | LongTimePattern                                              | HH:mm:ss                           | 3:54:20 PM                            |
| s      | SortableDateTimePattern (conforms to ISO 8601) using current culture | yyyy-MM-dd HH:mm:ss                | 2005-01-06T15:54:20                   |
| u      | UniversalSortableDateTimePattern (conforms to ISO 8601), unaffected by current culture | yyyy-MM-dd HH:mm:ss                | 2002-01-06 20:54:20Z                  |
| U      | UniversalSortableDateTimePattern                             | dddd, MMMM dd, yyyy HH:mm:ss       | Thursday, January 06, 2005 5:54:20 PM |
| R,r    | RFC1123Pattern                                               | ddd, dd MMM yyyy HH':'mm':'ss'GMT' | Thu, 06 Jan 2005 15:54:20 GMT         |
| O,o    | RoundtripKind (useful to restore all properties when parsing) | yyyy-MM-dd HH:mm:ss.fffffffK       | 2005-01-06T15:54: 20.0000000-08:00    |

[^1]: Notice that formats U, u, R, and r use Universal (Greenwich) Time, regardless of the local time zone, so example values for these formats are 5 hours ahead of example values for other formats (which assume local time to be U.S. Eastern time). The Pattern column specifies the corresponding custom format string made up of the characters listed in Table 12-3. 

```FSharp
let aDate: DateTime = DateTime.Parse("5/17/2005 3:54 PM")
let msg = String.Format("Event DateTime Time is {0:f}", aDate)
    // => Event Date Time is Tuesday, May 17, 2005 3:54 PM
```

If you can't find a standard date and time format that suits your needs, you can create a custom format by putting together the special characters listed in Table 12-3:

```FSharp
let msg = String.Format("Current year is {0:yyyy}", DateTime.Now) // => Current year is 2005
```

The default date separator (/) and default time separator (:) formatting characters are particularly elusive because they're replaced by the default date and time separator defined for the current locale. In some cases—most notably when formatting dates for a structured query language (SQL) SELECT or INSERT command—you want to be sure that a given separator is used on all occasions. In this case, you must use the backslash escape character to force a specific separator:

```FSharp
// Format a date in the format mm/dd/yyyy, regardless of current locale.
msg = String.Format(@"{0:MM\/dd\/yyyy}", aDate)   // => 05/17/2005
```

##### Table 12-3: Character Sequences That Can Be Used in Custom Date and Time Formats

| Format  | Description                                                  |
| ------- | ------------------------------------------------------------ |
| d       | Day of month (one or two digits as required).                |
| dd      | Day of month (always two digits, with a leading zero if required). |
| ddd     | Day of week (three-character abbreviation).                  |
| dddd    | Day of week (full name).                                     |
| M       | Month number (one or two digits as required).                |
| MM      | Month number (always two digits, with a leading zero if required). |
| MMM     | Month name (three-character abbreviation).                   |
| MMMM    | Month name (full name).                                      |
| y       | Year (last one or two digits, no leading zero).              |
| yy      | Year (last two digits).                                      |
| yyyy    | Year (four digits).                                          |
| H       | Hour in 24-hour format (one or two digits as required).      |
| HH      | Hour in 24-hour format (always two digits, with a leading zero if required). |
| h       | Hour in 12-hour format (one or two digits as required).      |
| hh      | Hour in 12-hour format.                                      |
| m       | Minutes (one or two digits as required).                     |
| mm      | Minutes (always two digits, with a leading zero if required). |
| s       | Seconds (one or two digits as required).                     |
| ss      | Seconds.                                                     |
| t       | The first character in the AM/PM designator.                 |
| f       | Second fractions, represented in one digit. (ff means second fractions in two digits, fff in three digits, and so on up to 7 fs in a row.) |
| F       | Second fractions, represented in an optional digit. Similar to f, except it can be used with `DateTime.ParseExact` without throwing an exception if there are fewer digits than expected. ~~(New in .NET Framework 2.0.).~~ |
| tt      | The AM/PM designator.                                        |
| z       | Time zone offset, hour only (one or two digits as required). |
| zz      | Time zone offset, hour only (always two digits, with a leading zero if required). |
| zzz     | Time zone offset, hour and minute (hour and minute values always have two digits, with a leading zero if required). |
| K       | The Z character if the `Kind` property of the `DateTime` value is `Utc`; the time zone offset (e.g., "-8:00") if the `Kind` property is `Local`; an empty character if the `Kind` property is `Unspecified`. ~~(New in .NET Framework 2.0.).~~ |
| /       | Default date separator.                                      |
| :       | Default time separator.                                      |
| \char   | Escape character, to include literal characters that would be otherwise considered special characters. |
| %format | Includes a predefined date/time format in the result string. |
| '…' "…" | A group of literal characters. You can add a sequence of literal characters by enclosing them in single or double quotation marks. |
| other   | Any other character is taken literally and inserted in the result string as is. |

### The Char Type

The `Char` class represents a single character. There isn't much to say about this data type, other than it exposes a number of useful static methods that enable you to test whether a single character meets a given criterion. All these methods are overloaded and take either a single `Char`, or a `String` plus an index in the string. For example, you check whether a character is a digit as follows:

```FSharp
// Check an individual Char value.
let ok: Boolean = Char.IsDigit('1')      // => True
// Check the Nth character in a string.
let ok = Char.IsDigit("A123", 0)         // => False
```

This is the list of the most useful static methods that test single characters: `IsControl`, `IsDigit`, `IsLetter`, `IsLetterOrDigit`, `IsLower`, `IsNumber`, `IsPunctuation`, `IsSeparator`, `IsSymbol`, `IsUpper`, and `IsWhiteSpace`.

You can convert a character to uppercase and lowercase with the `ToUpper` and `ToLower` static methods. By default these methods work according to the current thread's locale, but you can pass them an optional `CultureInfo` object, or you can use the culture-invariant versions `ToUpperInvariant` and `ToLowerInvariant`:

```FSharp
let newChar: Char = Char.ToUpper('a')                   // => A
let newChar = Char.ToLower('H', new CultureInfo("it-IT"))     // => h
let loChar: Char = Char.ToLowerInvariant('G')           // => g
```

You can convert a string into a `Char` by means of the `CChar` operator or the `Char.Parse` method. Or you can use the new `TryParse` static method ~~(added in .NET Framework 2.0)~~ to check whether the conversion is possible and perform it in one operation:

```FSharp
if Char.TryParse("a", newChar) then
   // newChar contains the a character.
```

The `Char` class doesn't directly expose the methods to convert a character into its Unicode value and back. If you don't want to use the `Asc`, `AscW`, `Chr`, and `ChrW` methods in the `Microsoft.VisualBasic.Strings` module, you can resort to a pair of static methods in the `System.Convert` type:

```FSharp
// Get the Unicode value of a character (same as Asc, AscW).
let uni: Short = Convert.ToInt16('A')
// Convert back to a character.
let ch: Char = Convert.ToChar(uni)
```

I cover the `Convert` class in more detail later in this chapter.

### The StringBuilder Type

As you know, a `String` object is immutable, and its value never changes after the string has been created. This means that any time you apply a method that changes its value, you're actually creating a new `String` object. For example, the following statement:

```FSharp
S <- S.Insert(3, "1234")
```

doesn't modify the original string in memory. Instead, the `Insert` method creates a new `String` object, which is then assigned to the `S` object variable. The original string object in memory is eventually reclaimed during the next garbage collection unless another variable points to it. The superior memory allocation scheme of .NET ensures that this mechanism adds a relatively low overhead; nevertheless, too many allocate and release operations can degrade your application's performance. The `System.Text.StringBuilder` object offers a solution to this problem.

You can think of a `StringBuilder` object as a buffer that can contain a string with the ability to grow from zero characters to the buffer's current capacity. Until you exceed that capacity, the string is assembled in the buffer and no memory is allocated or released. If the string becomes longer than the current capacity, the `StringBuilder` object transparently creates a larger buffer. The default buffer initially contains 16 characters, but you can change this by assigning a different capacity in the `StringBuilder` constructor or by assigning a value to the `Capacity` property:

```FSharp
// Create a StringBuilder object with initial capacity of 1,000 characters.
let sb = new StringBuilder(1000)
```

You can process the string held in the `StringBuilder` object with several methods, most of which have the same name as and work similarly to methods exposed by the `String` class—for example, the `Insert`, `Remove`, and `Replace` methods. The most common way to build a string inside a `StringBuilder` object is by means of its `Append` method, which takes an argument of any type and appends it to the current internal string:

```FSharp
// Create a comma-delimited list of the first 100 integers.
for n = 1 to 100 do
   // Note that two Append methods are faster than a single Append,
   // whose argument is the concatenation of N and ",".
   sb.Append(n)
   sb.Append(",")

// Insert a string at the beginning of the buffer.
sb.Insert(0, "List of numbers: ")
Console.WriteLine(sb)   // => List of numbers: 1,2,3,4,5,6,…
```

The `Length` property returns the current length of the internal string:

```FSharp
// Continuing previous example…
Console.WriteLine("Length is {0}.", sb.Length)    // => Length is 309.
```

There's also an `AppendFormat` method, which enables you to specify a format string, much like the `String.Format` method, and an `AppendLine` method ~~(new in .NET Framework 2.0)~~, which appends a string and the default line terminator:

```FSharp
for n = 1 to 100 do
   sb.AppendLine(CStr(n))

```

The following procedure compares how quickly the `String` and `StringBuilder` classes perform a large number of string concatenations:

```FSharp
let TIMES: Int32 = 10000

let sw = new Stopwatch()
sw.Start()
let mutable s: String = ""
for i = 1 to TIMES do
    s <- s + i.ToString() + ","
sw.Stop()
Console.WriteLine("Regular string: {0} milliseconds",  sw.ElapsedMilliseconds)

let sw = new Stopwatch()
sw.Start()
let sb = new StringBuilder(TIMES * 4)
for i = 1 to TIMES do
    // Notice how you can merge two Append methods.
    sb.Append(i).Append(",") |> ignore
sw.Stop()
Console.WriteLine("StringBuilder: {0} milliseconds.",  sw.ElapsedMilliseconds)
```

The results of this benchmark can really be astonishing because they show that the `StringBuilder` object can be more than 100 times faster than the regular `String` class is. The actual ratio depends on how many iterations you have and how long the involved strings are. For example, when TIMES is set to 20,000 on my computer, the standard string takes 5 seconds to complete the loop, whereas the `StringBuilder` type takes only 8 milliseconds!

### The SecureString Type

The way .NET strings are implemented has some serious implications related to security. In fact, if you store confidential information in a string—for example, a password or a credit card number—another process that can read your application's address space can also read your data. Although getting access to a process's address space isn't really a trivial task, consider that some portions of your address space are often saved in the operating system's swap file, where reading them is much easier.

The fact that strings are immutable means that you can't really clear a string after using it. Worse, because a string is subject to garbage collection, there might be several copies of it in memory, which in turn increases the probability that one of them goes in the swap file. A .NET Framework version 1.1 application that wants to ensure the highest degree of confidentiality should stay clear of standard strings and use some other technique, for example, an encrypted `Char` or `Byte` array, which is decrypted only one instant before using the string and cleared immediately afterward.

Version 2.0 of the .NET Framework makes this process easier with the introduction of the `SecureString` type, in the `System.Security` namespace. Basically, a `SecureString` instance is an array of characters that is encrypted using the Data Protection API (DPAPI). Unlike the standard string, and similarly to the `StringBuilder` type, the `SecureString` type is mutable: you build it one character at a time by means of the `AppendChar` method, similar to what you do with the `StringBuilder` type, and you can also insert, remove, and modify individual characters by means of the `InsertAt`, `SetAt`, and `RemoveAt` methods. Optionally, you can make the string immutable by invoking the `MakeReadOnly` method. Finally, to reduce the number of copies that float in memory, `SecureString` instances are *pinned*, which means that they can't be moved around by the garbage collector.

The `SecureString` type is so secure that it exposes neither a method to initialize it from a string nor a method that returns its contents as clear text: the former task requires a series of calls to `AppendChar`, the latter can be performed with the help of the `Marshal` type, as I'll explain shortly. A `SecureString` isn't serializable and therefore you can't even save to file for later retrieval. And of course you can't initialize it from a string burnt into your code, which would defy the intended purpose of this type. What are your options for correctly initializing a `SecureString` object, then?

A first option is to store the password as clear text in an access control list (ACL)—protected file and read it one char at a time. This option isn't bulletproof, though, and in some cases it can't be applied anyway because the confidential data is entered by the user at run time.

Another option—actually the only option that you can adopt when the user enters the confidential data at run time—is to have the user enter the text one character at a time, and then encrypt it on the fly. The following code snippet shows how you can fake a password-protected `TextBox` control in a Windows `Forms` application that never stores its contents in a regular string:

```FSharp
let password = new SecureString()

let txtPassword_KeyPress(sender : Object, e : KeyPressEventArgs) = // Handles txtPassword.KeyPress
    match Convert.ToInt16(e.KeyChar) with // asc
    | 8s ->
        // Backspace: remove the char from the secure string.
        if txtPassword.SelectionStart > 0 then
            password.RemoveAt(txtPassword.SelectionStart - 1)
         
    | ascii when ascii >= 32s ->
        // Delete current selection.
        if txtPassword.SelectionLength > 0 then

            for i in [txtPassword.SelectionStart +  txtPassword.SelectionLength - 1 .. -1 .. txtPassword.SelectionStart ] do
                password.RemoveAt(i)

            // Regular character: insert it in the secure string.
            if txtPassword.SelectionStart = txtPassword.SelectionLength then
                password.AppendChar(e.KeyChar)
            else
                password.InsertAt(txtPassword.SelectionStart, e.KeyChar)
         
            // Display (and store) an asterisk in the text box.
            e.KeyChar <- '*'
    | _ -> ()
```

As I mentioned before, the `SecureString` object doesn't expose any method that returns its contents as clear text. Instead, you must use a couple of methods of the `Marshal` type, in the `System.Runtime.InteropServices` namespace:

```FSharp
// Convert the password into an unmanaged BSTR.
let ptr: IntPtr = Marshal.SecureStringToBSTR(password)
// For demo purposes, convert the BSTR into a regular string and use it.
let pw: String = Marshal.PtrToStringBSTR(ptr)

…
// Clear the unmanaged BSTR used for the password.
Marshal.ZeroFreeBSTR(ptr)
```

Of course, the previous code isn't really secure because at one point you have assigned the password to a regular string. In some cases, this is unavoidable, but at least this approach ensures that the clear text string exists for a shorter amount of time. An alternative, better approach is to have the unmanaged Basic string (BSTR) processed by a piece of unmanaged code.

You really see the benefits of this technique when you use a member that accepts a `SecureString` instance, for example, the `Password` property of the `ProcessStartInfo` type:

```FSharp
// Run Notepad under a different user account.
let psi = new ProcessStartInfo("notepad.exe")
psi.UseShellExecute <- false
psi.UserName <- "Francesco"
psi.Password <- password
Process.Start(psi)
```