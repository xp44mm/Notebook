## Regular Expressions at Work

Most of the examples so far have showed many possible applications of regular expressions, yet they just scratch the surface of the power of regular expressions. In this last section, I illustrate more examples and provide hints of how to make the most of this powerful feature of the .NET Framework.

### Common Regex Patterns

For your convenience, I have prepared a list of recurring patterns, which you can often use as is in your code (see Table 14-3). To help you find the pattern that suits your needs, the list includes some patterns that I covered earlier in this chapter as well as patterns that I discuss in following sections. You should enclose the pattern between a pair of `\b` sequences if you want to find individual words, or between the `^` and `$` characters if you want to test whether the entire input string matches the pattern.

##### Table 14-3: Common Regular Expression Patterns

* `\d+`
  Positive integer. 

* `[+-]?\d+`
  A positive or negative integer whose sign is optional.

* `[+-]?\d+(\.\d+)?`
  A floating-point number whose sign and decimal portion are optional.

* `[+-]?\d+(\.\d+)?(E[+-]?\d+)?`
  A floating-point number that can be optionally expressed in exponential format (e.g., 1.23E+12); the mantissa sign and the exponent sign are optional.

* `[0-9A-Fa-f]+`
  A hexadecimal number.

* `\w+`
  A sequence of alphanumeric and underscore characters; same as the `[A–Za–z0–9_]+` sequence.

* `[A–Z]+`
  An all-uppercase word.

* `[A–Z][a–z]+`
  A proper name (the initial character is uppercase, and then all lowercase characters).

* `[A–Z][A–Za–z']+`
  A last name (the initial character is uppercase, and string can contain other uppercase characters and apostrophes, as in O'Brian).

* `[A–Za–z]{1,10}`
  A word of 10 characters or fewer.

* `[A–Za–z]{11,}`
  A word of 11 characters or more.

* `[A-Za–z_]\w*`
  A valid Visual Basic and C# identifier that begins with a letter or underscores and optionally continues with letters, digits, or underscores.

* `(?<q>["'])*?\k<q>`
  A quoted string enclosed in either single or double quotation marks.

* `(10|11|12|0?[1–9])(?<sep>[-/])(30|31|2\d|1\d|0?[1–9])\k<sep>(\d{4}|\d{2})`
  A U.S. date in the `mm-dd-yyyy` or `mm/dd/yyyy` format. Month and day numbers can have a leading zero; month number must be in the range 1–12; day number must be in the range 1–31 (but invalid dates such as 2/30/2004 are matched); year number can have two or four digits and isn't validated.

* `(30|31|2\d|1\d|0?[1–9])(?<sep>[-/])(10|11|12|0?[1–9])\<sep>(\d{4}|\d{2})`
  A European date in the `dd-mm-yyyy` or `dd/mm/yyyy` format. (See previous entry for more details.)

* `(2[0–3]|[01]\d|\d):[0–5]\d`
  A time value in the hh:mm 24-hour format; leading zero for hour value is optional.

* `\(\d{3}\)–\d{3}–\d{4}`
  A phone number such as (123)-456-7890.

* `\d{5}(-\d{4})?`
  A U.S. ZIP Code.

* `\d{3}-\d{2}-\d{4}`
  A U.S. social security number (SSN).

* `((\d{16}|\d{4}(-\d{4}){3})|(\d{4}(\d{4}){3}))`
  A 16-digit credit card number that can embed optional dashes or spaces to define four groups of four digits, for example, 1234567812345678, 1234-5678-1234-5678, or 1234 5678 1234 5678. (Needless to say, it doesn't validate whether it is a valid credit card number.)

* `([0–9A–Fa–f]{32}|[0–9A–Fa–f]{8}-([0–9A–Fa–f]{4}-){3}[0–9A–Fa–f]{12})`
  A 32-digit GUID, with or without embedded dashes, as in .

* `([A–Za–z]:)?\\?([^/:*?<>"|\\]+\\)*[^/:*?<>"|\\]+`
  A Windows filename, with or without a drive and a directory name.

* `(http|https)://([\w-]+\.)+[\w-]+(/[\w- ./?%&=]*)?`
  An Internet URL; you should use the regular expression in case-in-sensitive mode to also match prefixes such as HTTP or Https.

* `\w+([-+.]\w+)*@\w+([-.]\w+)*\.\w+([-.]\w+)*`
  An Internet e-mail address.

* `((25[0-5]|2[0-4]\d|1\d\d|[1-9]\d|\d)\.){3}(25[0-5]|2[0-4]\d|1\d\d|[1-9]\d|\d)`
  A four-part IP address, such as 192.168.0.1; the pattern verifies that each number is in the range 0–255.

* `([1-5]\d{4}|6[0-4]\d{3}|65[0-4]\d{2}|655[0-2]\d|6553[0-4]|\d{1,4})`
  A 16-bit integer that can be assigned to a UShort variable, in the range of 0 to 65,535.

* `(-?[12]\d{4}|-?3[0-1]\d{3}|-?32[0-6]\d{2}|-?327[0-5]\d|-?3276[0-7]|-32768|-?\d{1,4})`
  A 16-bit integer that can be assigned to a Short variable, in the range of –32,768 to 32,767.

* `^(?=.*\d)(?=.*[a-z])(?=.*[A-Z])\w{8,}$`
  A password of at least eight alphanumeric characters that contains at least one digit, one lowercase character, and one uppercase character. `Replace` the `\w` term with `[0-9A-Za-z@.]` to allow some symbols so that users can use their e-mail address as a password.

You can find more regular expressions in the Visual Studio Regular Expression Editor dialog box (see Figure 14-2) or by browsing the huge regular expression library you can find at http://www.regexlib.com. If you are serious about regular expressions, don't miss The Regulator free utility, which you can download from http://regex.osherove.com/.

~~!Image from book~~ 
~~Figure 14-2: Setting the ValidationExpression property of a RegularExpressionValidator ASP.NET control by selecting one of the common regular expressions you find in the Regular Expression Editor dialog box~~ 

### Searching for Words and Quoted Strings

A quite common operation with regular expressions is splitting a long string into words. Apparently, this is also the simplest task you can perform with regular expressions:

```F#
let text: String = "A word with àccéntèd vowels, and the 123 number."
let pattern: String = @"\w+"
for m: Match in Regex.Matches(text, pattern) do
    Console.WriteLine(m.Value)
```

The problem with this oversimplified approach is that it also includes sequences of digits and underscores in the collection of results, and you might not want that. A better attempt is as follows:

```F#
let pattern = "[A–Za–z]+"
```

This works better, but fails to include entire words if they contain accented characters or characters from other alphabets, such as Greek or Cyrillic. Under previous versions of the .NET Framework, you could solve this issue by using the little-used `\p` sequence, which allows you to specify a Unicode character class. For example, the `\p{Ll}` sequence matches any lowercase character, whereas the `\p{Lu}` sequence matches any uppercase character. The solution to the problem is therefore as follows:

```F#
let pattern = "(\p{Lu}|\p{Ll})+"
```

The character class subtraction feature, introduced in .NET Framework 2.0, offers a new solution to the problem, based on the consideration that you can "subtract" the digits and the underscore from the range of characters expressed by the `\w` sequence:

```F#
let pattern = "[\w-[0-9_]]+"
```

When extracting words, you often want to discard *noise words*, such as articles (the, a, an), conjunctions (and, or), and so forth. You might discard these words inside the For Each loop, but it's more elegant to have the regular expression get rid of them:

```F#
let pattern = @"\b(?!(the|a|an|and|or|on|of|with)\b)\w+"
let text = "A fox and another animal on the lawn"
for m: Match in Regex.Matches(text, pattern, RegexOptions.IgnoreCase) do
    Console.Write ("{0} ", m.Value) // => fox another animal lawn
```

The `\w+` in the previous pattern specifies that we are looking for a word, but the `(?!...\b)` expression specifies that the match must not begin with one of the noise words; the neat result is that the pattern matches all the words except those in the noise list.

Another common problem related to parsing is when you need to consider a quoted string as an individual word, such as when you parse the command passed to a command-line utility. (In this specific case, you might define a Sub Main that takes an array of strings as an argument and let the .NET Framework do the job for you, but it wouldn't work in the most general case.) The following regular expression matches an individual word or a string embedded in either single or double quotation marks:

```F#
// For simplicity's sake, use \w+ to match an individual word.
let pattern = @"(?<q>[""']).*?\k<q>|\w+"
```

Notice that the `.*?` does a lazy matching so that it matches any character between the quotation marks but won't match the closing quotation mark.

Sometimes you might want to extract just *unique* words, such as when you want to make a dictionary of all the words in a text file or a set of text files. A possible solution is to extract all the words and use a `Hashtable` object to remember the words found so far:

```F#
let text: String = "one two three two zone four three"
let re = new Regex("\w+")
let words = new Hashtable()
for m : Match in re.Matches(text) do
    if not <| words.Contains(m.Value) then
        Console.Write("{0} ", m.Value)
        words.Add(m.Value, null)
```

Quite surprisingly, you can achieve the same result with a single, albeit complex, regular expression:

```F#
let pattern = @"(?<word>\b\w+\b)(?!.+\b\k<word>\b)"
for m : Match in Regex.Matches(text, pattern) do
    Console.Write(m.Value + " ")
```

The expression `(?<word>\b\w+\b)` matches a sequence of alphanumeric characters `(\w)` on a word boundary `(\b)` and assigns this sequence the name "word". The `(?!)` construct means that the word just matched must not be followed by the word already matched (the back reference `\k<word>)` even if there are other characters in the middle (represented by the `.+` sequence). Translated into plain English, the regular expression means "match any word in the text that isn't followed by another instance of the same word" or, more simply, "match all the words that appear only once in the document or the last occurrence of a repeated word." As you can see, all the unique words are found correctly, even though their order is different from the previous example:

```
one two zone four three
```

Notice that the `\b` characters in the regular expression prevent partial matches ("one" doesn't match the trailing portion of "zone"). A slightly different regular expression can find the duplicated words in a document:

```F#
let pattern = @"(?<word>\b\w+\b)(?=.+\b\k<word>\b)"
```

where the `(?=)` construct means that the word match must be followed by another instance of itself. (Notice that this pattern finds all the duplicates; therefore, it finds two duplicates if there are three occurrences of a given word.) Although the regex-only techniques are very elegant, the look-ahead `(?=)` clause makes them relatively inefficient: for example, on a source text of about 1 million characters, the regex-only technique is approximately 8 times slower than the technique that uses an auxiliary `Hashtable` to keep track of all the words already parsed.

One last type of word search I want to explain is the *proximity search*, which is when you search two strings that must be found close to each other in the source string, with no more than *N* words between them. For example, given the "one two three two zone four three" source string, a proximity search for the words "one" and "four" with *N* equal to 4 would be successful, whereas it would fail with *N* equal to 3. The pattern for such a proximity search is quite simple:

```F#
let pattern = @"\bone(\W+\w+){0,4}\W+\bfour\b"
if Regex.IsMatch(text, pattern, RegexOptions.IgnoreCase) then
    // At least one occurrence of the words "one" and "four"
    // with four or fewer words between them.
    ()
```

You can also define a function that takes the input string, the two words, and the maximum distance between them and returns a `MatchCollection` object:

```F#
let ProximityMatches(text : String, word1 : String, word2 : String, maxDistance : int) : MatchCollection =
    let pattern = sprintf @"\b%s(\W+\w+){0,%d}\W+\b%s\b" word1 maxDistance word2
    let re = new Regex(pattern, RegexOptions.IgnoreCase)
    re.Matches(text)
```

Thus, the previous code snippet would become

```F#
let mc: MatchCollection = ProximityMatches(text, "one", "four", 4)
if mc.Count > 0 then
    ()
```

### Validating Strings, Numbers, and Dates

As I explained earlier in this chapter, typically you can use a search pattern as a validation pattern by simply enclosing it in the `^` and `$` symbols and using the `IsMatch` method instead of the `Matches` method. For example, the following code checks that a string—presumably the Text property of a TextBox control—contains a five-digit U.S. ZIP Code:

```F#
let pattern = @"^\d{5}$"
if Regex.IsMatch(text, pattern) then
   // It's a string containing five digits.
   ()
```

Things become more interesting when you want to *exclude* a few combinations from the set of valid strings, which you can do with the `(?!)` clause. For example, the 00000 sequence isn't a valid ZIP code, and you can exclude it by using the following pattern:

```F#
let pattern = @"^(?!00000)\d{5}$"
```

You can use the `(?=)` look-ahead assertion to check that the input string contains all characters in a given class, regardless of their position. For example, you can use the following pattern to enforce a robust password policy and ensure that the end user types a password of at least eight characters and that it contains a combination of digits and uppercase and lowercase letters:

```F#
let pattern = @"^(?=.*\d)(?=.*[a-z])(?=.*[A-Z])\w{8,}$"
```

Let's see how this pattern works. The first `(?=.*\d)` clause makes the search fail at the very beginning if the portion of the input string to its right (and therefore, the entire input string) doesn't contain any digits. The `(?=.*[a–z])` clause checks that the input string contains a lowercase character, and the `(?=.*[A–Z])` clause does the same for uppercase characters. These three look-ahead clauses don't consume any characters, and therefore the remaining `\w{8,}` clause can check that the input string contains at least eight characters.

Validating a number in a given range poses a few interesting problems. In general, you might not want to use regular expressions to validate numbers or dates because the `Parse` and `TryParse` methods exposed by the `DateTime` type and all numeric types offer more flexibility and cause fewer headaches. However, in some cases, regular expressions can be a viable solution even for this task, for example, when you want to extract valid numbers and dates from a longer document.

Checking that an integer has up to the specified number of digits is a trivial problem, of course:

```F#
// Validate an integer in the range of 0 to 9,999; accept leading zeros.
let pattern = @"^\d{1,4}$"
```

The negative `(?!)` look-ahead clause lets you rule out a few cases, for example:

```F#
// Validate an integer in the range 1 to 9,999; reject leading zeros.
let pattern = "^(?!0)\d{1,4}$"

// Validate an integer in the range 0 to 9,999; reject leading zeros.
// (Same as previous one, but accept a single zero as a special case.)
let pattern = "^(0|(?!0)\d{1,4})$"

```

If the upper limit of the accepted range isn't a number in the form 99…999, you can still use regular expressions to do the validation, but the pattern becomes more complex. For example, the following pattern checks that a number is in the range 0 to 255 with no leading zeros:

```F#
let pattern = @"^(25[0–5]|2[0–4]\d|1\d\d|[1–9]\d|\d)$"
```

The 25[0–5] clause validates numbers in the range 250 to 255; the `2[0–4]\d` clause validates numbers in the range 200 to 249; the `1\d\d` clause validates numbers in the range 100 to 199; the `[1–9]\d` clause takes care of the numbers 10 to 99; finally, the `\d` clause covers the range 0 to 9. A slight modification of this pattern allows you to validate a four-part IP address, such as 192.168.0.11:

```F#
let pattern = @"^((25[0-5]|2[0-4]\d|1\d\d|[1-9]\d|\d)\.){3}(25[0-5]|2[0-4]\d|1\d\d|[1-9]\d|\d)$"
```

Things quickly become complicated with larger ranges:

```F#
// Validate an integer number in the range 0 to 65,535; leading zeros are OK.
let pattern = "^([1-5]\d{4}|6[0-4]\d{3}|65[0-4]\d{2}|655[0-2]\d|6553[0-4]|\d{1,4})$"
```

Numbers that can have a leading sign require special treatment:

```F#
// Validate an integer in the range -32,768 to 32,767; leading zeros are OK.
let pattern = @"^(-?[12]\d{4}|-?3[0-1]\d{3}|-?32[0-6]\d{2}|-?327[0-5]\d|-?3276[0-7]|-32768|-?\d{1,4})$"
```

Notice in the previous pattern that the special case –32768 must be dealt with separately; all the remaining clauses have an optional minus sign in front of them. You can use a similar technique to validate a time value:

```F#
// Validate a time value in the format hh:mm; the hour number can have a leading zero.
let pattern = @"^(2[0–3]|[01]\d|\d):[0–5]\d$"
```

Validating a date value is much more difficult because each month has a different number of days and, above all, because the valid day range for February depends on whether the year is a leap year. Before I illustrate the complete pattern for solving this problem, let's see how we can use a regular expression to check whether a two-digit number is a multiple of 4:

```F#
// If the first digit is even, the second digit must be 0, 4, or 8.
// If the first digit is odd, the second digit must be 2 or 6.
let pattern = "^([02468][048]|[13579][26])$"
```

Under the simplified assumption that the year number has only two digits, and therefore the date refers to a year in the current century, we can simplify the regular expression significantly because the year 2000 was a leap year, unlike 1900 and 2100. To better explain the final regular expression I have split the pattern onto four lines:

```F#
// This portion deals with months with 31 days.
let p1: String = @"(0?[13578]|10|12)/(3[01]|[012]\d|\d)/\d{2}"
// This portion deals with months with 30 days.
let p2: String = @"(0?[469]|11)/(30|[012]\d|\d)/\d{2}"
// This portion deals with February 29 in leap years.
let p3: String = @"(0?2)/29/([02468][048]|[13579][26])"
// This portion deals with other days in February.
let p4: String = @"(0?2)/(2[0-8]|[01]\d|\d)/\d{2}"

// Put all the patterns together.
let pattern = String.Format("^({0}|{1}|{2}|{3})$", p1, p2, p3, p4)
// Check the date.
if Regex.IsMatch(text, pattern) then
    // Date is valid.
    ()
```

If the year number can have either two or four digits, we must take into account the fact that all years divisible by 100 are not leap years, except if they are divisible by 400. (For example, 1900 isn't a leap year, but 2000 is.) This constraint makes the regular expression more complicated, but by now you should be experienced enough to understand how the following code works:

```F#
// This portion deals with months with 31 days.
let s1 : String = @"(0?[13578]|10|12)/(3[01]|[12]\d|0?[1–9])/(\d\d)?\d\d"
// This portion deals with months with 30 days.
let s2 : String = @"(0?[469]|11)/(30|[12]\d|0?[1-9])/(\d\d)?\d\d "
// This portion deals with days 1–28 in February in all years.
let s3 : String = @"(0?2)/(2[0–8]|[01]\d|0?[1–9])/(\d\d)?\d\d"
// This portion deals with February 29 in years divisible by 400.
let s4 : String = @"(0?2)/29/(1600|2000|2400|2800|00)"
// This portion deals with February 29 in noncentury leap years.
let s5 : String = @"(0?2)/29/(\d\d)?(0[48]|[2468][048]|[13579][26])"
// Put all the patterns together.
let pattern = String.Format("^({0}|{1}|{2}|{3}|{4})$", s1, s2, s3, s4, s5)
```

(Notice that I might have merged the portions s4 and s5 in a single subexpression that validates all leap years, but I kept the two expressions separate for clarity's sake.) It's easy to derive a similar regular expression for dates in dd/mm/yy format and to account for separators other than the dash character.

### Searching for Nested Tags

When you apply regular expressions to HTML or XML files, you must take the hierarchical natures of these files into account. For example, let's say that you want to extract the contents of `<table>…</table>` sections in an HTML file. You can't simply use a pattern such as this:

```regex
<table[\s>].*?</table>
```

because it would return bogus results when you apply it to a text that contains nested tables, such as this:

```html
<table border=1><tr><td><table>…</table></td><td>…</td></tr></table>
```

In cases like these, the balancing group definition construct shown in Table 14-1 can help because it lets you take nested tags into account. (For a great example of how you can use this construct, read http://blogs.msdn.com/bclteam/archive/2005/03/15/396452.aspx.) However, this construct is quite difficult to use and has some limitations, the most notable of which is that it doesn't work well if you're looking for a series of nested tags, as when you want to display all `<table>`, `<tr>`, and `<td>` blocks. In cases like these, you need two nested loops, as in the following code:

```F#
let tags = [ "table"; "tr"; "td"; "div"; "span" ]
let openTag = new Regex(String.concat "|" tags |> sprintf @"<(?<tag>(%s))[\s>]", RegexOptions.IgnoreCase)

//每个标签对应一个正则表达式，备查
let relevantTags =
    tags |> List.map(fun tag ->
        let pat = String.Format(@"((?<open><{0})[\s>]|(?<close>)</{0}>)", tag)
        let re = new Regex(pat, RegexOptions.IgnoreCase)
        tag,re
    ) |> dict

// Find all nested HTML tags in a file. (e.g., <table>…</table>)
let findNestedTags (text:string) =
    // We've found an open tag. Let's look for open and close versions of this tag.
    let rec loop (m0: Match) (mr: Match) (openTags: int) =
        if mr.Groups.["open"].Success then
            let m2 = mr.NextMatch()
            loop m0 m2 (openTags + 1)
        elif mr.Groups.["close"].Success then
            if openTags = 1 then
                text.[m0.Index .. mr.Index + mr.Length - 1]
            else
                let m2 = mr.NextMatch()
                loop m0 m2 (openTags - 1)
        else
            sprintf "%A" [m0.Result("$`");m0.Result("$0");m0.Result("$'")]

    openTag.Matches(text)
    |> Seq.map(fun m0 ->
        let tag: String = m0.Groups.["tag"].Value
        let rr = relevantTags.[tag]
        let mr = rr.Match(text, m0.Index)
        loop m0 mr 0
    )

type Test(output: ITestOutputHelper) =
    [<Fact>]
    member this.``Searching for Nested Tags``() =
        let text: String = "<table border=1><tr><td><table></table></td><td></td></tr><td></table>"
        for found in findNestedTags text do
            output.WriteLine(sprintf "%A" found)

```

Once you understand how this code works, you can easily modify it to match other hierarchical entities, for example, parentheses in a math expression or nested type definitions in a .vb source file.

### Parsing Data Files

Even though XML has emerged as the standard technology in exchange information, many legacy applications still output data in older and simpler formats. Two such formats are fixed-width text files and delimited text files. Microsoft Visual Basic 2005 supports a new `TextFieldParser` object that can simplify this task remarkably (see Chapter 15, "Files, Directories, and Streams"), but you can also solve this problem with regular expressions in a very elegant manner. 

Let's consider a fixed-width data file such as this one:

```
John  Evans   New York
Ann   Beebe   Los Angeles
```

Each text line contains information about first name (6 characters), last name (8 characters), and city. The largest city has 9 characters, but usually we can assume that the last field takes all the characters up to the end of the current line. Reading this file requires very few lines of code:

```F#
let pattern: String = "^(?<first>.{6})(?<last>.{8})(?<city>.+)$"
let re = new Regex(pattern)
use sr = new StreamReader("c:\data.txt")
while not <| sr.EndOfStream do
    let m: Match = re.Match(sr.ReadLine())
    Console.WriteLine("First={0}, Last={1}, City={2}",  
        m.Groups.["first"].Value.TrimEnd(), 
        m.Groups.["last"].Value.TrimEnd(),  
        m.Groups.["city"].Value.TrimEnd())
```

The expression `(?<first>.{6})` creates a group named "first" that corresponds to the initial 6 characters. Likewise, `(?<last>.{8})` creates a group named "last" that corresponds to the next 8 characters. Finally, `(?<city>.+)` creates a group for all the remaining characters on the line and names it as "city". The `^` and `$` characters stand for the beginning and the end of the line, respectively.

The beauty of this approach is that it is quite easy to adapt the code to different field widths and to work with delimited fields. For example, if the fixed-width fields are separated by semicolons, you simply modify the regular expression as follows, without touching the remaining code:

```F#
let pattern = @"^(?<first>.{6});(?<last>.{8});(?<city>.+)$"
```

Let's now adapt the parsing program to another quite common exchange format: delimited text files. In this case, each field is separated from the next one by a comma, a semicolon, a tab, or another special character. To further complicate things, such files usually allow values embedded in single or double quotation marks; in this case, you can't just use the `Split` method of the `String` type to do the parsing because your result would be bogus if a quoted value happens to include the delimiter (as in "Evans, John").

In such cases, regular expressions are a real lifesaver. In fact, you just need to use a different regular expression pattern with the same parsing code used in previous examples. Let's start with the simplified assumption that there are no quoted strings in the file, as in the following:

```
John , Evans, New York
Ann, Beebe, Los Angeles
```

I threw in some extra white spaces to add interest to the discussion. These spaces should be ignored when you're parsing the text. Here is the regular expression that can be used to parse such a comma-delimited series of values:

```F#
let pattern = @"^\s*(?<first>.*?)\s*,\s*(?<last>.*?)\s*,\s*(?<city>.*?)\s*$"
```

You don't need to modify other portions of the parsing code I showed previously. It is essential that `\s*` sequences and the delimiter character (the comma, in this specific case) are placed outside the `(?)` construct so that they aren't included in named groups. Also notice that we use the `.*?` sequence to avoid matching the delimiter character or the spaces that might surround it.

Next, let's see how to parse quoted fields, such as those found in the following text file:

```
'John, P.' , "Evans" , "New York"
'Robert "Zare"', "" , "Los Angeles, CA"
```

Text fields can be surrounded by both single and double quotation marks and they can contain the comma symbol as well as the quotation mark not used as a delimiter. The regular expression that can parse this text file is more complex:

```F#
let pattern = 
    [
        @"(?<q1>[""'])(?<first>.*?)\k<q1>"
        @"(?<q2>[""'])(?<last>.*?)\k<q2>"
        @"(?<q3>[""'])(?<city>.*?)\k<q3>"
    ]
    |> String.concat @"\s*,\s*"
    |> sprintf @"^\s*%s\s*$"
```

The `(?<q1>["'])` subexpression matches either the single or the double leading quotation mark delimiter and assigns this group the name "q1". (The double quotation mark character is doubled because it appears in a Visual Basic string.) The `\k<q1>` subexpression is a back reference to whatever the q1 group found and therefore matches whichever quotation mark character was used at the beginning of the field. The q2 and q3 groups have the same role for the next two fields. Once again, you don't need to change any other statement in the parsing routine.

The previous pattern has a small defect, though. Many programs that output data in delimited format enclose a text field in quotation marks only if the field contains the delimiter character. For example, in the following data file the first and last fields in the first record are enclosed in quotation marks because they embed a comma, but the fields in the second record aren't.

```
"John, P." , Evans , "Los Angeles, CA"
Robert, Zare, New York
```

To solve this minor problem I need to introduce one of the most powerful features of regular expressions: conditional matching. Look closely at the following pattern:

```F#
let pattern = 
    [
        @"(?<q1>[""']?)(?<first>.*?)(?(q1)\k<q1>)"
        @"(?<q2>[""']?)(?<last>.*?)(?(q2)\k<q2>)"
        @"(?<q3>[""']?)(?<city>.*?)(?(q3)\k<q3>)"
    ]
    |> String.concat @"\s*,\s*"
    |> sprintf @"^\s*%s\s*$"
```

The `(?<q1>["']?)` is similar to the pattern used in the previous example, except it has a trailing `?` character; therefore, it matches an optional single or double quotation mark character. Later in the same line you find the `(?(q1)\k<q1>)` clause, which tests whether the q1 group is defined and, if so, matches its value. In other words, if the q1 group actually matched the single or double quotation mark character, the expression `(?(q1)\k<q1>)` matches it again; otherwise, the expression is ignored. The same reasoning applies to the other two fields in the record.

The `(?(expr)…)` clause has an optional "no" portion (see Table 14-1), so you might even match a portion of a string if a previous group has *not* been matched.

### Parsing and Evaluating Expressions

A nice and somewhat surprising application of regular expressions is in expression evaluation. In the section titled "The `Replace` Method" earlier in this chapter, you saw how you can evaluate the result of an addition operation embedded in a string such as "12+34", thanks to the overload of the `Replace` method that takes a callback function. Of course, you don't have to stop at additions, and in fact you can create a complete and quite versatile expression evaluator built on a single regular expression and some support code. Creating such a regular expression isn't a trivial task, though. Let's analyze the Evaluate method a piece at a time:

```F#
// A floating-point number, with optional leading and trailing spaces
let num: String = @"\s*[+-]?\d+\.?\d*\b\s*"
// A number inside a pair of parentheses
let nump: String = @"\s*\((?<nump>" + num + ")\)\s*"

let numg name = sprintf "(?<%s>%s)" name num

// Math operations
let add: String = sprintf @"(?<![*/^]\s*)%s\+%s(?!\s*[*/^])" (numg "add1") (numg "add2")
let subt: String = sprintf @"(?<![*/^]\s*)%s\-%s(?!\s*[*/^])" (numg "sub1") (numg "sub2")
let mul: String = sprintf @"(?<!\^\s*)%s\*%s(?!\s*\^)" (numg "mul1") (numg "mul2")
let div: String = sprintf @"(?<!\^\s*)%s/%s(?!\s*\^)" (numg "div1") (numg "div2")
let modu: String = sprintf @"(?<!\^\s*)%s\s+mod\s+%s(?!\s*\^)" (numg "mod1") (numg "mod2")
let pow: String = sprintf @"%s\^%s" (numg "pow1") (numg "pow2")

// 1-operand and 2-operand functions
let fone: String = sprintf @"%s\s*\(%s\)" "(?<fone>(exp|log|log10|abs|sqr|sqrt|sin|cos|tan|asin|acos|atan))" (numg "fone1")
let ftwo: String = sprintf @"%s\s*\(%s,%s\)" "(?<ftwo>(min|max))" (numg "ftwo1")  (numg "ftwo2")

// Put everything in a single regex.
let pattern: String =
    [fone; ftwo; modu; pow; div; mul; subt; add; nump]
    |> String.concat "|"
    |> sprintf "(%s)"

/// 匹配表达式
let reEval = new Regex(pattern, RegexOptions.IgnoreCase)

/// 匹配数字
let reNumber = new Regex("^" + num + "$")
```

The pattern corresponding to the num constant represents a floating-point number, optionally preceded by a plus or minus sign. Let's now consider the regular expression that defines the addition operation: it consists of two numbers, each one forming a named group (add1 and add2); the two numbers are separated by the `+` symbol. Additionally, the pattern is preceded by a `(?<![*/^]\s*)` negative look-behind assertion, which ensures that the first operand doesn't follow an operator with a higher priority than addition (that is, the multiplication, division, or raising to power operator). Similarly, the second operand is followed by the `(?!\s*[*/^])` negative look-ahead assertion, which ensures that the addition isn't followed by an operation with higher priority. The patterns for other math operations are similar, so I won't describe them in detail. The body of the `Evaluate` function follows:

```F#
let Evaluate(expr : String) : Double =
    // Add a space after a +/– used for additions and subtractions to ensure
    // they are not mistakenly taken as the leading sign of a number.
    let mutable expr = Regex.Replace(expr, @"(?<=[0-9)]\s*)[+-](?=[0-9(])", "$0 ")

    // Loop until the expression is reduced to a number.
    while not <| reNumber.IsMatch(expr) do
        // Replace only the first subexpression that can be processed.
        let newExpr: String = reEval.Replace(expr, PerformOperation, 1)
        // If the expression hasn't been simplified, there must be a problem.
        if expr = newExpr then raise <| new ArgumentException("Invalid expression")
        // Reenter the loop with the new expression.
        expr <- newExpr

    // Convert to a floating-point number and return.
    Double.Parse(expr)
```

At the top of the Do loop, the `reNumber` regular expression checks whether the expression contains a number: in this case, the loop is exited and the value of the number is returned to the caller. If this isn't the case, the loop is repeated in the attempt to simplify the expression using the `reEval` regular expression; if the expression doesn't change, it means that the expression can't be simplified further because it is malformed, and the method throws an exception. If the expression has been simplified, the loop is reentered.

The `PerformOperation` callback method is where the actual math operations are carried out. Detecting which operator has been matched is simple because all the groups defined by the various operators have different names:

```F#
let fones =
    dict [
            "exp", Math.Exp
            "log", Math.Log
            "log10", Math.Log10
            "abs", Math.Abs
            "sqrt", Math.Sqrt
            "sin", Math.Sin
            "cos", Math.Cos
            "tan", Math.Tan
            "asin", Math.Asin
            "acos", Math.Acos
            "atan", Math.Atan
    ]

let PerformOperation(m : Match) : String =
    let biop name op =
        [1;2]
        |> Seq.map(sprintf "%s%d" name)
        |> Seq.map(fun tag -> Double.Parse(m.Groups.[tag].Value))
        |> Seq.reduce op
        |> sprintf "%f"

    if m.Groups.["nump"].Length > 0 then
        m.Groups.["nump"].Value.Trim()
    elif m.Groups.["add1"].Length > 0 then biop "add" (+)
    elif m.Groups.["sub1"].Length > 0 then biop "sub" (-)
    elif m.Groups.["mul1"].Length > 0 then biop "mul" ( * )
    elif m.Groups.["div1"].Length > 0 then biop "div" (/)
    elif m.Groups.["mod1"].Length > 0 then biop "mod" (fun a b -> Math.IEEERemainder(a,b))
    elif m.Groups.["pow1"].Length > 0 then biop "pow" ( ** )

    elif m.Groups.["fone"].Length > 0 then
        let operand: Double = Double.Parse(m.Groups.["fone1"].Value)
        match m.Groups.["fone"].Value.ToLower() with
        | fname when fones.ContainsKey fname  -> fones.[fname] operand |> sprintf "%f"
        | _ -> failwith "unrealized"

    elif m.Groups.["ftwo"].Length > 0 then
        let operand1: Double = Double.Parse(m.Groups.["ftwo1"].Value)
        let operand2: Double = Double.Parse(m.Groups.["ftwo2"].Value)
        match m.Groups.["ftwo"].Value.ToLower() with
        | "min"  ->
            let result = Math.Min(operand1, operand2)
            result.ToString()
        | "max"  ->
            let result = Math.Max(operand1, operand2)
            result.ToString()
        | _ -> failwith "unrealized"
    else failwith "unrealized"
```

It's easy to create a console application or a Windows Forms program that asks the user for an expression and displays the expression value. Figure 14-3 shows such a program in action. The only interesting piece of code is the method that runs when the user clicks the Eval button:

```F#
let txtExpression_Text = ""
let mutable lblResult_Text = ""

let btnEval_Click(sender : Object, e : EventArgs) =
    try
        // Read and evaluate the expression typed in the txtExpression field.
        let res: Double = Evaluate(txtExpression_Text)
        // Display the result in a Label control.
        lblResult_Text <- res.ToString()
    with ex ->
        lblResult_Text <- "ERROR " + ex.Message
```

!img 
Figure 14-3: The demo application that tests the Evaluate method 

The Evaluate function and the `PerformOperation` helper method have a total of only 65 executable statements, yet they implement a full-fledged expression evaluator that you can easily expand to support additional operators and functions. Such conciseness is possible thanks to the power of regular expressions and, in particular, to the ability to specify look-behind and look-ahead negative assertions, which ensures that the priority of the various operators is honored.

### Parsing Code

Developers spend a lot of time with code; thus, you might consider source files as one of the primary types of data you deal with. As such, source files are great candidates for being parsed and processed by means of regular expressions. In the following example, I illustrate a very simple console application that reads a Visual Basic source file and outputs information about the total number of lines, blank lines, comment lines, and executable lines it contains, and then it splits the file into types and their members and displays detailed information about each of them.

The Main procedure of this application is just a driver for the `CodeStats` class:

```F#
let Main(args : String[]) =
    let fileName: String = args.[0]
    let code: String = File.ReadAllText(fileName)
    let stats = new CodeStats("FILE", fileName, code)
    Console.Write(stats.Description(0))
```

The `CodeStats` type is where the actual parse occurs. Each instance of this class has public fields to store information about line count, plus a `Members` collection that can contain other `CodeStats` instances. For example, the `CodeStats` object for the file contains a collection of `CodeStats` objects related to the types defined in the file, and `CodeStats` objects related to types have a collection of `CodeStats` objects related to type members (methods and property procedures). For the sake of simplicity, I account for neither nested types nor less common blocks such as custom events.


```F#
let reTotalLines = new Regex("^.*$", RegexOptions.Multiline)
let reBlankLines = new Regex(@"^\s*$", RegexOptions.Multiline)
let reCommentLines = new Regex(@"^\s*('|Rem).*$",  RegexOptions.Multiline ||| RegexOptions.IgnoreCase)

let maybe = sprintf @"(%s)?\s*"

let reTypes =
    let pat =
        [
            maybe "Public|Friend|Private|Protected|Protected Friend"
            maybe "Shadows|NotInheritable|MustInherit"
            @"(?<type>(Class|Module|Interface|Enum|Structure))\s+(?<name>\w+)"
            @"[\w\W]+?"
            @"\bEnd \k<type>"
        ] |> String.concat ""

    new Regex(pat, RegexOptions.IgnoreCase ||| RegexOptions.Multiline)

let reMembers =
    let pat =
        [
            maybe "Public|Friend|Private|Protected|Protected Friend"
            maybe "Default|Shared"
            maybe "ReadOnly|WriteOnly"
            maybe "Overloads"
            maybe "Shadows|Overridable|Overrides|MustOverride|NotOverridable"
            @"(?<type>(Function|Sub|Property))\s+(?<name>\w+)"
            @"[\w\W]+?"
            @"\bEnd \k<type>"
        ] |> String.concat ""

    new Regex(pat, RegexOptions.IgnoreCase ||| RegexOptions.Multiline)

type baseCodeStats(tp:string, name : String, code : String) =
    let totalLines = reTotalLines.Matches(code).Count
    let blankLines = reBlankLines.Matches(code).Count
    let commentLines = reCommentLines.Matches(code).Count
    let executableLines = totalLines - blankLines - commentLines

    let percent lines = float lines / float totalLines

    member this.selfDescription(indentLevel : int) =
        let indentSpace i = new String(' ', i * 2)
        let indent = indentSpace indentLevel
        let nextIndent = indentSpace (indentLevel+1)
        let sb = new StringBuilder()
        sb.AppendFormat(indent + "{0} {1}\n", tp, name)
            .AppendFormat(nextIndent + "Total lines: {0,6}\n", totalLines)
            .AppendFormat(nextIndent + "Blank lines: {0,6}{1,9:P1}\n", blankLines, percent blankLines)
            .AppendFormat(nextIndent + "Comment lines: {0,6}{1,9:P1}\n", commentLines, percent commentLines)
            .AppendFormat(nextIndent + "Executable lines: {0,6}{1,9:P1}\n", executableLines, percent executableLines)
            .AppendLine("")

type MemberCodeStats(tp:string, name : String, code : String) =
    inherit baseCodeStats(tp, name, code)

    member this.Description(indentLevel : int) =
        let sb = this.selfDescription(indentLevel)
        sb.ToString()

type TypeCodeStats(tp:string, name : String, code : String) =
    inherit baseCodeStats(tp, name, code)

    member this.Description(indentLevel : int) =
        let sb = this.selfDescription(indentLevel)

        // Ask child objects to display their description.
        let members =
            [
                for m: Match in reMembers.Matches(code) do
                    yield MemberCodeStats(m.Groups.["type"].Value,m.Groups.["name"].Value, m.Value)
            ]

        for stats: MemberCodeStats in members do
            sb.Append(stats.Description(indentLevel + 1)) |> ignore

        sb.ToString()

type CodeStats(tp:string, name : String, code : String) =
    inherit baseCodeStats(tp, name, code)

    member this.Description(indentLevel : int) =
        let sb = this.selfDescription(indentLevel)

        // Ask child objects to display their description.
        let types =
            [
                for m: Match in reTypes.Matches(code) do
                    yield TypeCodeStats(m.Groups.["type"].Value,m.Groups.["name"].Value, m.Value)
            ]

        for stats: TypeCodeStats in types do
            sb.Append(stats.Description(indentLevel + 1)) |> ignore

        sb.ToString()
```

Interestingly, I ran this code to count lines in its own source file, so I learned that I could create an effective and useful programming utility with just 63 executable statements, which is quite a good result. (See Figure 14-4.)

```F#
FILE D:\repos\xp44mm\Programming Microsoft Visual Basic 2005 The Language\14 RegularExpressions\ProjectStats\Module1.vb
  Total lines:          86
  Blank lines:          14    16.3%
  Comment lines:         3     3.5%
  Executable lines:     69    80.2%

  Module MainModule
    Total lines:          12
    Blank lines:           3    25.0%
    Comment lines:         0     0.0%
    Executable lines:      9    75.0%

    Sub Main
      Total lines:           8
      Blank lines:           1    12.5%
      Comment lines:         0     0.0%
      Executable lines:      7    87.5%

  Class CodeStats
    Total lines:          70
    Blank lines:           9    12.9%
    Comment lines:         3     4.3%
    Executable lines:     58    82.9%

    Sub New
      Total lines:          19
      Blank lines:           2    10.5%
      Comment lines:         0     0.0%
      Executable lines:     17    89.5%

    Function Description
      Total lines:          21
      Blank lines:           1     4.8%
      Comment lines:         1     4.8%
      Executable lines:     19    90.5%
```

Figure 14-4: Using the `CodeStats` class to count how many statements its own source code contains



### Playing with Regular Expressions (Literally)

By now, you should be convinced that regular expressions are too powerful to be used only for plain text searches and substitutions. In this last example, I want to prove that regular expressions can be useful when you'd never suspect that searches are involved and that you can use them simply to perform pattern matching.

Let's consider the game of poker. I won't build an entire application that plays poker (nor encourage gambling in any way...), but I will focus on a very small programming problem that is related to this game. How would you write a method that evaluates the score corresponding to a hand of five cards? You can solve this problem in a variety of ways, with numerous If and Select Case statements, but the solution offered by regular expressions can hardly be beaten as far as elegance, performance, and conciseness are concerned.



The following method accepts five strings, each one corresponding to a card in the hand and each one consisting of a character pair: the first character stands for the card value and can be a digit 1–9, or T, J, Q, or K (where T stands for Ten); the second character of the pair is the card's suit and can be C (clubs), D (diamonds), H (hearts), or S (spades). The code in the method sorts the five cards by their value, and then builds two separate strings—one containing the five values and the other containing the five suits—and tests them against suitable regular expressions, starting from the most complex and moving to the simpler ones. (Testing the regular expressions in this order is crucial; otherwise, a plain straight would be mistakenly reported as a straight flush and a pair would appear to be a full house or a four-of-a-kind.)

```F#
// Notice that we must account for the fact that the values T,J,Q,K
// don't appear in this order after being sorted alphabetically.
let straight = "12345|23456|34567|45678|56789|6789T|789JT|89JQT|9JKQT|1JKQT"

let flush = @"(.)\1\1\1\1"

let fourOfAKind = @"(.)\1\1\1"

// A full house can be either 3+2 or 2+3 cards with the same values.
let fullHouse = @"(.)\1\1(.)\2|(.)\3(.)\4\4"

let threeOfAKind = @"(.)\1\1"

let twoPairs = @"(.)\1.?(.)\2"

let OnePair = @"(.)\1"

// Sort the array and create the sequence of values and of suits.
let unzipCards (cards : String[]) =
    cards
    |> Array.sort
    |> Array.map (fun s->s.[0],s.[1])
    |> Array.unzip
    |> fun (a,b) -> String a,String b

let EvalPokerScore(cards : String[]) : String =
    let values, suits = unzipCards cards

    // Check each sequence in order.
    if Regex.IsMatch(values, straight) && Regex.IsMatch(suits, flush) then
        "StraightFlush"
    elif Regex.IsMatch(values, fourOfAKind) then
        "FourOfAKind"
    elif Regex.IsMatch(values, fullHouse) then
        "FullHouse"
    elif Regex.IsMatch(suits, flush) then
        "Flush"
    elif Regex.IsMatch(values,  straight) then
        "Straight"
    elif Regex.IsMatch(values, threeOfAKind) then
        "ThreeOfAKind"
    elif Regex.IsMatch(values, twoPairs) then
        "TwoPairs"
    elif Regex.IsMatch(values, OnePair) then
        "OnePair"
    else
        "HighCard"
```

Here are a few examples of how you can call the method and the results it returns:

```F#
Assert.Equal(EvalPokerScore[|"1H"; "4H"; "3H"; "5H"; "2H"|], "StraightFlush")
Assert.Equal(EvalPokerScore[|"9C"; "9S"; "8H"; "TD"; "9D"|], "ThreeOfAKind")
Assert.Equal(EvalPokerScore[|"8C"; "KC"; "TC"; "QC"; "9C"|], "Flush")
Assert.Equal(EvalPokerScore[|"TC"; "KC"; "QD"; "8D"; "9H"|], "HighCard")
```

The `EvalPokerScore` method is so concise that you might be surprised to learn that you can simplify it further. The trick is simple and leverages the fact that patterns are just strings and can be stored in a data structure. In this case, you can use a two-dimensional array so that you can test each pattern in a For loop. (Use a pattern that always matches, such as a plain dot, if you aren't interested in matching either the values or the suits.)

```F#
    // (Replace the sequences of If statements in previous listing with this code.)
    let scores : String[][] =
        [|
            [|straight     ; flush; "StraightFlush"|]
            [|fourOfAKind  ; "."  ; "FourOfAKind"  |]
            [|fullHouse    ; "."  ; "FullHouse"    |]
            [|"."          ; flush; "Flush"        |]
            [|straight     ; "."  ; "Straight"     |]
            [|threeOfAKind ; "."  ; "ThreeOfAKind" |]
            [|twoPairs     ; "."  ; "TwoPairs"     |]
            [|OnePair      ; "."  ; "OnePair"      |]
        |]

    scores
    |> Array.tryFind(fun score ->
        Regex.IsMatch(values, score.[0]) && Regex.IsMatch(suits, score.[1]))
    |> function Some score -> score.[2] | _ -> "HighCard"
```

The preceding code highlights the fact that regular expressions have the ability to replace code with just data, in this case a series of If statements with strings stored in an array. In this particular example, this feature isn't especially useful (other than to make the code more concise). In many applications, however, this ability can make a big difference. For example, you can store all the validation patterns in a database or an XML file so that you can actually change the behavior of your application without even recompiling its code.