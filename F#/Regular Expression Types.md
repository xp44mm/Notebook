## Regular Expression Types


Now that I have illustrated the fundamentals of regular expressions, it's time to examine all the types in the `System.Text.RegularExpressions` namespace.

### The Regex Type

As you've seen in the preceding section, the `Regex` type provides two overloaded constructors—one that takes only the pattern and another that also takes a bit-coded value that specifies the required regular expression options:

```F#
// This Regex object can search the word "dim" in a case-insensitive way.
let re = new Regex(@"\bdim\b", RegexOptions.IgnoreCase)
```

The `Regex` class exposes only two properties, both of which are read-only. The `Options` property returns the second argument passed to the object constructor, while the `RightToLeft` property returns `true` if you specified the `RightToLeft` option. (The regular expression matches from right to left.) No property returns the regular expression pattern, but you can use the `ToString` method for this purpose.

#### Searching for Substrings

The `Matches` method searches the regular expression inside the string provided as an argument and returns a `MatchCollection` object that contains zero or more `Match` objects, one for each nonintersecting match. The `Matches` method is overloaded to take an optional starting index:

```F#
// Get the collection that contains all the matches.
let mc: MatchCollection = re.Matches(source)

// Print all the matches after the 100th character in the source string.
for m: Match in re.Matches(source, 100) do
    Console.WriteLine(m.ToString())
```

You can change the behavior of the `Matches` method (as well as the `Match` method, described later) by using a `\G` assertion to disable scanning. In this case, the match must be found exactly where the scan begins. This point is either at the index specified as an argument (or the first character if this argument is omitted) or immediately after the point where the previous match terminates. In other words, the `\G` assertion finds only *consecutive* matches:

```F#
// Finds consecutive groups of space-delimited numbers.
let re = new Regex("\G\s*\d+")
// Note that search stops at the first non-numeric group.
Console.WriteLine(re.Matches("12 34 56 ab 78").Count) // => 3
```

Sometimes, you don't really want to list all the occurrences of the pattern when determining whether the pattern is contained in the source string would suffice. For example, this is usually the case when you are checking that a value typed by the end user complies with the expected format (for example, it's a phone number or a social security number in a valid format). If that's your interest, the `IsMatch` method is more efficient than the `Matches` method is because it stops the scan as soon as the first match is found. You pass to this method the input string and an optional start index:

```F#
// Check whether the input string is a date in the format mm-dd-yy or
// mm-dd-yyyy. (The source string can use slashes as date separators and
// can contain leading or trailing white spaces.)
let re2 = new Regex("^\s*\d{1,2}(/|-)\d{1,2}\1(\d{4}|\d{2})\s*$")
if re2.IsMatch(" 12/10/2001 ") then
    Console.WriteLine("The date is formatted correctly.")
    // (We don't check whether month and day values are in valid range.)
```

The regular expression pattern in the preceding code requires an explanation:

1. The `^` and `$` characters mean that the source string must contain one date value and nothing else. These characters must be used to check whether the source string *matches* the pattern, rather than *contains* it.
2. The `\s*` subexpression at the beginning and end of the string means that we accept leading and trailing white spaces.
3. The `\d{1,2}` subexpression means that the month and day numbers can have one or two digits, whereas the `(\d{4}|\d{2})` subexpression means that the year number can have four or two digits. The four-digit case must be tested first; otherwise, only the first two digits are matched.
4. The `(/|-)` subexpression means that we take either the slash or the dash as the date separator between the month and day numbers.
5. The `\1` subexpression means that the separator between day and year numbers must be the same separator used between month and day numbers.

The `Matches` method has an undocumented feature that becomes very handy when parsing very long strings. When you use the return value of this method in a For Each loop—as I did in the majority of examples shown so far—this method performs a sort of lazy evaluation: instead of processing the entire string, it stops the parsing process as soon as the first `Match` object can be returned to the calling program. When the Next statement is reached and the next iteration of the loop begins, it restarts the parsing process where it had left previously, and so forth. If you exit the loop with an Exit For statement, the remainder of the string is never parsed, which can be very convenient if you are looking for a specific match and don't need to list all of them. You can easily prove this feature with this code:

```F#
let sw = new Stopwatch()
sw.Start()
for m: Match in re.Matches(text) do
    // Show how long it took to find this match.
    Console.WriteLine("Elapsed {0} milliseconds", sw.ElapsedMilliseconds)
sw.Stop()
```

The output in the console window proves that the first `Match` object was returned almost instantaneously, whereas it took some millions of CPU cycles to locate the character at the end of the string:

```
Elapsed 0 milliseconds
Elapsed 80 milliseconds
```

Keep in mind that this lazy evaluation feature of the `Matches` method is disabled if you query other members of the returned `MatchCollection` object, for example, the Count property. It's quite obvious that the only way to count the occurrences of the pattern is to process the entire input string.

In some special cases you might want to have even more control on the parsing process, for example, to skip portions of the string that aren't of interest for your purposes. In these cases, you can use the `Match` method, which returns only the first `Match` object and lets you iterate over the remaining matches using the `Match.NextMatch` method, as this example demonstrates:

```F#
// Search all the dates in a source string.
let source: String = " 12-2-1999 10/23/2001 4/5/2001 "
let re = new Regex("\s*\d{1,2}(/|-)\d{1,2}\1(\d{4}|\d{2})")
// Find the first match.
let mutable m: Match = re.Match(source)
// Enter the following loop only if the search was successful.
while m.Success do
    // Display the match, but discard leading and trailing spaces.
    Console.WriteLine(m.ToString().Trim())
    // Find the next match; exit if not successful.
    m <- m.NextMatch()
```

The `Split` method is similar to the `String.Split` method except it defines the delimiter by using a regular expression rather than a single character. For example, the following code prints all the elements in a comma-delimited list of numbers, ignoring leading and trailing white-space characters:

```F#
let source: String = "123, 456,,789"
let re = new Regex("\s*,\s*")
for s: String in re.Split(source) do
    // Note that the third element is a null string.
    Console.Write(s + "-") // => 123-456--789-
```

(You can modify the pattern to `\s*[ ,]+\s*` to discard empty elements.) The `Split` method supports several overloaded variations, which let you define the maximum count of elements to be extracted and a starting index (if there are more elements than the given limit, the last element contains the remainder of the string):

```F#
// Split max 5 items.
let arr: String[] = re.Split(source, 5)
// Split max 5 items, starting at the 100th character.
let arr2: String[] = re.Split(source, 5, 100)
```

#### The Replace Method

The `Regex.Replace` method lets you selectively replace portions of the source string. This method requires that you create numbered or named groups of characters in the pattern and then use those groups in the replacement pattern. The following code example takes a string that contains one or more dates in the mm-dd-yy format (including variations with a / separator or a four-digit year number) and converts them to the dd-mm-yy format while preserving the original date separator:

```F#
let source: String = "12-2-1999 10/23/2001 4/5/2001 "
let pattern: String = 
    @"\b(?<mm>\d{1,2})(?<sep>(/|-))(?<dd>\d{1,2})\k<sep>(?<yy>(\d{4}|\d{2}))\b"
let re = new Regex(pattern)
Console.WriteLine(re.Replace(source, "${dd}${sep}${mm}${sep}${yy}"))
    // => 2-12-1999 23/10/2001 5/4/2001
```

The pattern string is similar to the one shown previously, with an important difference: it defines four groups—named mm, dd, yy, and sep—that are later rearranged in the replacement string. The `\b` assertion at the beginning and end of the pattern ensures that the date is a word of its own.

The `Replace` method supports other overloaded variants. For example, you can pass two additional numeric arguments, which are interpreted as the maximum number of substitutions and the starting index:

```F#
// Expand all "ms" abbreviations to "Microsoft" (regardless of their case).
let text: String = "Welcome to ms Ms ms MS"
let re2 = new Regex("\bMS\b", RegexOptions.IgnoreCase)
// Replace up to two occurrences, starting at the 12th character.
Console.WriteLine(re2.Replace(text, "Microsoft", 2, 12))
    // => Welcome to ms Microsoft Microsoft MS
```

If the replacement operation does something more sophisticated than simply delete or change the order of named groups, you can use an overloaded version of the `Replace` function that takes a delegate argument pointing to a filter function you've defined elsewhere in the application. This feature gives you tremendous flexibility, as the following code demonstrates:

```F#
// The callback method
let DoSum(m : Match) : String =
   // Parse the two operands.
   let args: String[] = m.Value.Split('+')
   let n1 = int(args.[0])
   let n2 = int(args.[1])
   // Return their sum, as a string.
   (n1 + n2).ToString()

let TestReplaceWithCallback() =
   // This pattern defines two integers separated by a plus sign.
   let re = new Regex("\d+\s*\+\s*\d+")
   let source: String = "a = 100 + 234: b = 200+345"
   // Replace all sum operations with their results.
   Console.WriteLine(re.Replace(source, DoSum))
      // => a = 334: b = 545
```

The delegate must point to a function that takes a `Match` object and returns a `String` object. The code inside this function can query the `Match` object properties to learn more about the match. For example, you can use the `Index` property to peek at what immediately precedes or follows in the source string so that you can make a more informed decision.

Callback functions are especially useful to convert individual matches to uppercase, lowercase, or proper case:

```F#
let ConvertToUpperCase(m : Match) : String =
   m.Value.ToUpper()

let ConvertToLowerCase(m : Match) : String =
   m.Value.ToLower()

let ConvertToProperCase(m : Match) : String =
   // We must convert to lowercase first, to ensure that ToTitleCase works as intended.
   CultureInfo.CurrentCulture.TextInfo.ToTitleCase(m.Value.ToLower())
```

Here's an example:

```F#
// Convert country names in the text string to uppercase.
let text: String = "I visited italy, france, and then GERMANY."
let re2 = new Regex(@"\b(Usa|France|Germany|Italy|Great Britain)\b", RegexOptions.IgnoreCase)
let text = re2.Replace(text, ConvertToProperCase)
Console.WriteLine(text) // => I visited Italy, France, and then Germany.
```

#### Static Methods

All the methods shown so far are also available as static methods; therefore, in many cases you don't need to create a `Regex` object explicitly. You generally pass the regular expression pattern to the static method as a second argument after the source string. For example, you can split a string into individual words as follows:

```F#
// \W means "any nonalphanumeric character."
let words: String[] = Regex.Split("Split these words", "\W+")
```

The `Regex` class also exposes a few static methods that have no instance method counterpart. The `Escape` method takes a string and converts the special characters `.$^{[(|)*+?\` to their equivalent escaped sequence. This method is especially useful when you let the end user enter the search pattern:

```F#
Console.WriteLine(Regex.Escape("(x)")) // => \(x\)

// Check whether the character sequence the end user entered in
// the txtChars TextBox control is contained in the source string.
if Regex.IsMatch(source, Regex.Escape(txtChars.Text)) then ...
```

The `Unescape` static method converts a string that contains escaped sequences back into its unescaped equivalent. This method can be useful even if you don't use regular expressions; for example, to build strings that contain carriage returns, line feeds, and other nonprintable chapters by using a C#-like syntax and without having to concatenate string constants and subexpressions based on the Chr function:

```F#
let s = Regex.Unescape(@"First line\r\nSecond line ends with null char\x00")
```

#### The CompileToAssembly Method

When you use the `RegexOptions.Compiled` value in the `Regex` constructor, you can expect a slight delay for the regular expression to be compiled to IL. In most cases, this delay is negligible, but you can avoid it if you want by using the `CompileToAssembly` static method to precompile one or more regular expressions. The result of this precompilation is a separate assembly that contains one Regex-derived type for each regular expression you've precompiled. The following code shows how you can use the `CompileToAssembly` method to create an assembly that contains two precompiled regular expressions:

```F#
// The namespace for both compiled regex types in this sample
let nsName: String = "CustomRegex"
// The first regular expression compiles to a type named RegexWords.
// (The last argument means that the type is public.)
let rci1 = new RegexCompilationInfo("\w+", RegexOptions.Compiled, "RegexWords", nsName, true)
// The second regular expression compiles to a type named RegexIntegers.
let rci2 = new RegexCompilationInfo("\d+", RegexOptions.Compiled, "RegexIntegers", nsName, true)
// Create the array that defines all compiled regular expressions.
let regexInfo: RegexCompilationInfo[] = [|rci1; rci2|]

// Compile these types to an assembly named "CustomRegularExpressions".
let an = new System.Reflection.AssemblyName()
an.Name <- "CustomRegularExpressions"
Regex.CompileToAssembly(regexInfo, an)
```

The preceding code creates an assembly named CustomRegularExpressions.dll in the same directory as the current application's executable. You can add a reference to this assembly from any Visual Studio 2005 project and use the two `RegexWords` and `RegexIntegers` types, or you can load these types using reflection (see Chapter 18). In the former case, you can use a strongly typed variable:

```F#
let reWords = new CustomRegex.RegexWords()
for m: Match in reWords.Matches("A string containing five words") do
   Console.WriteLine(m.Value)
```

### The MatchCollection and Match Types

The `MatchCollection` class represents a set of matches. It has no constructor because you can create a `MatchCollection` object only by using the `Regex.Matches` method.

The `Match` class represents a single match. You can obtain an instance of this class either by iterating on a `MatchCollection` object or directly by means of the `Match` method of the `Regex` class. The `Match` object is immutable and has no public constructor.

The main properties of the `Match` class are `Value`, `Length`, and `Index`, which return the matched string, its length, and the index at which it appears in the source string. The `ToString` method returns the same string as the `Value` property does. I already showed you how to use the `IsSuccess` property of the `Match` class and its `NextMatch` method to iterate over all the matches in a string.

You must pay special attention when the search pattern matches an empty string, for example, `\d*` (which matches zero or more digits). When you apply such a pattern to a string, you typically get one or more empty matches, as you can see here:

```F#
let re = new Regex("\d*")
for m: Match in re.Matches("1a23bc456de789") do
   // The output from this loop shows that some matches are empty.
   Console.Write(m.Value + ",") // => 1,,23,,,456,,,789,,
```

As I explained earlier, a search generally starts where the previous search ends. However, the rule is different when the engine finds an empty match because it advances by one character before repeating the search. You would get trapped in an endless loop if the engine didn't behave this way.

If the pattern contains one or more groups, you can access the corresponding `Group` object by means of the `Match` object's `Groups` collection, which you can index by the group number or group name. I discuss the `Group` object shortly, but you can already see how you can use the `Groups` collection to extract the variable names and values in a series of assignments:

```F#
let source: String = "a = 123: b=456"
let re2 = new Regex("(\s*)(?<name>\w+)\s*=\s*(?<value>\d+)")
for m: Match in re2.Matches(source) do
   Console.WriteLine("Variable: {0}, Value: {1}", 
      m.Groups.["name"].Value, m.Groups.["value"].Value)
```

This is the result displayed in the console window:

```
Variable: a, Value: 123
Variable: b, Value: 456
```

The `Result` method takes a replace pattern and returns the string that would result if the match were replaced by that pattern:

```F#
// This code produces exactly the same result as the preceding snippet.
for m: Match in re2.Matches(source) do
   Console.WriteLine(m.Result("Variable: ${name}, Value: ${value}"))
```

### The Group Type

The `Group` class represents a single group in a `Match` object and exposes a few properties whose meanings should be evident. The properties are `Value` (the text associated with the group), `Index` (its position in the source string), `Length` (the group's length), and `Success` (`true` if the group has been matched). This code sample is similar to the preceding example, but it also displays the index in the source string where each matched variable was found:

```F#
let text: String = "a = 123: b=456"
let re = new Regex("(\s*)(?<name>\w+)\s*=\s*(?<value>\d+)")
for m: Match in re.Matches(text) do
   let g: Group = m.Groups.["name"]
   // Get information on variable name and value.
   Console.Write("Variable '{0}' found at index {1}", g.Value, g.Index)
   Console.WriteLine(", value is {0}", m.Groups.["value"].Value)
```

This is the result displayed in the console window:

```
Variable 'a' found at index 0, value is 123
Variable 'b' found at index 9, value is 456
```

The following example is more complex but also more useful. It shows how you can parse `<A>` tags in an HTML file and display the anchor text (the text that appears underlined on an HTML page) and the URL it points to. As you can see, it's just a matter of a few lines of code:

```F#
let re = new Regex(@"<A\s+HREF\s*=\s*("".+?""|.+?)>(.+?)</A>", RegexOptions.IgnoreCase)
// Load the contents of an HTML file.
let text: String = File.ReadAllText("test.htm")
// Display all occurrences.
let mutable m: Match = re.Match(text)
while m.Success do
   Console.WriteLine("{0} => {1}", m.Groups.[2].Value, m.Groups.[1].Value)
   m <- m.NextMatch()
```

To understand how the preceding code works, you must keep in mind that the `<A>` tag is followed by one or more spaces and then by an HREF attribute, which is followed by an equal sign and then the URL, which can be enclosed in quotation marks. All the text that follows the closing angle bracket up to the ending tag `</A>` is the anchor text. The regular expression uses the `.+?` lazy quantifier so as not to match too many characters and miss the delimiting quotation mark or closing angle bracket.

The regular expression defined in the preceding code defines two unnamed groups—the URL and the anchor text—so displaying details for all the `<A>` tags in the HTML file is just a matter of looping over all the matches. The regular expression syntax is complicated by the fact that quotation mark characters must be doubled when they appear in a Visual Basic string constant.

A few methods in the `Regex` class can be useful to get information about the groups that the parser finds in the regular expression. The `GetGroupNames` method returns an array with the names of all groups; the `GroupNameFromNumber` returns the name of the group with a given index; and the `GroupNumberFromName` returns the index of a group with a given name. See the MSDN documentation for more information.

### The CaptureCollection and Capture Types

The search pattern can include one or more capturing groups, which are named or unnamed subexpressions enclosed in parentheses. Capturing groups can be nested and can capture multiple substrings of the source strings because of quantifiers. For example, when you apply the `(\w)+` pattern to the "abc" string, you get one match for the entire string and three captured substrings, one for each character.

Fortunately, in most cases you don't need to work with captures because analyzing the source string at the match and group levels is often sufficient. You can access the collection of capture substrings through the `Captures` method of either the `Match` or the `Group` object. This method returns a `CaptureCollection` object that in turn contains one or more `Capture` objects. Individual `Capture` objects enable you to determine where individual captured substrings appear in the source string. The following code displays all the captured strings in the "abc def" string:

```F#
let text: String = "abc def"
let re = new Regex("(\w)+")
// Get the name or numbers of all the groups.
let groups: String[] = re.GetGroupNames()

// Iterate over all matches.
for m: Match in re.Matches(text) do
    // Display information on this match.
    Console.WriteLine("Match '{0}' at index {1}", m.Value, m.Index)
    // Iterate over the groups in each match.
    for s: String in groups do
        // Get a reference to the corresponding group.
        let g: Group = m.Groups.[s]
        // Get the capture collection for this group.
        let cc: CaptureCollection = g.Captures
        // Display the number of captures.
        Console.WriteLine(" Found {0} capture(s) for group {1}", cc.Count, s)
        // Display information on each capture.
        for c: Capture in cc do
            Console.WriteLine(" '{0}' at index {1}", c.Value, c.Index)
```

The text that follows is the result produced in the console window. (Notice that group 0 always refers to the match expression itself.)

```
Match 'abc' at index 0
  Found 1 capture(s) for group 0
    'abc' at index 0
  Found 3 capture(s) for group 1
    'a' at index 0
    'b' at index 1
    'c' at index 2
Match 'def' at index 4

Found 1 capture(s) for group 0
  'def' at index 4
Found 3 capture(s) for group 1
  'd' at index 4
  'e' at index 5
  'f' at index 6
```

