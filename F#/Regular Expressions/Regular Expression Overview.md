## Regular Expression Overview

The Microsoft .NET Framework comes with a very powerful regular expression engine that's accessible from any .NET language, so you can leverage the parsing power of languages such as Perl without having to switch away from your favorite language.

### The Fundamentals

`Regex` is the most important class in this group, and any regular expression code instantiates at least an object of this class (or uses one of the `Regex` static methods). This object represents an immutable regular expression. You instantiate this object by passing to it the search pattern, written using the special regular expression language, which I describe later:

```F#
// This regular expression defines any group of two characters
// consisting of a vowel followed by a digit (\d).
let re = new Regex(@"[aeiou]\d")
```

The `Matches` method of the `Regex` object applies the regular expression to the string passed as an argument; it returns a `MatchCollection` object, a read-only collection that represents all the nonoverlapping matches:

```F#
let re = new Regex(@"[aeiou]\d")
// This source string contains three groups that match the Regex.
let text: String = "a1 = a1 & e2"
// Get the collection of matches.
let mc: MatchCollection = re.Matches(text)
// How many occurrences did we find?
Console.WriteLine(mc.Count) // => 3
```

You can also pass to the `Matches` method a second argument, which is interpreted as the index where the search begins.

The `MatchCollection` object contains individual `Match` objects, which expose properties such as `Value` (the matching string that was found), `Index` (the position of the matching string in the source string), and `Length` (the length of the matching string, which is useful when the regular expression can match strings of different lengths):

```F#
// …(Continuing the previous example)…
for m: Match in mc do
    // Display text and position of this match.
    Console.WriteLine("'{0}' at position {1}" , m.Value, m.Index)
```

The preceding code displays these lines in the console window:

```
'a1' at position 0
'a1' at position 5
'e2' at position 10
```

The `Regex` object is also capable of modifying a source string by searching for a given regular expression and replacing it with something else:

```F#
let text: String = "a1 = a1 & e2"
// Search for the "a" character followed by a digit.
let re2 = new Regex(@"a\d")
// Drop the digit that follows the "a" character.
let res: String = re2.Replace(text, "a") // => a = a & e2
```

The `Regex` class also exposes static versions of the `Match`, `Matches`, and `Replace` methods. You can use these static methods when you don't want to instantiate a `Regex` object explicitly:

```F#
// This code snippet is equivalent to the previous one, but it doesn't
// instantiate a Regex object.
let res = Regex.Replace(text, @"a\d", "a")
```

~~The best way to learn regular expressions is, not surprisingly, through practice. To help you in this process, I have created a RegexTester application that enables you to test any regular expression against any source string or text file. (See Figure 14-1.) This application has been a valuable tool for me in exploring regular expression intricacies, and I routinely use it whenever I have a doubt about how a construct works.~~

~~!Image from book~~
~~Figure 14-1: The RegexTester application, enabling you to experiment with all the most important methods and options of the Regex object~~ 

### The Regular Expression Language

Table 14-1 lists all the constructs that are legal as regular expression patterns, grouped in the following categories:

- **Character escapes** 

  Used to match single characters. You need them to deal with non-printable characters (such as the newline and the tab characters) and to provide escaped versions for the characters that have a special meaning inside regular expression patterns. Together with substitutions, these are the only sequences that can appear in a replacement pattern.

- **Character classes** 

  Offer a means to match one character from a group that you specify between brackets, as in `[aeiou]`. You don't need to escape special characters when they appear in brackets except in the cases of the dash and the closing bracket, which are the only characters that have special meaning inside brackets. For example, `[()[\]{}]` matches opening and closing parentheses, brackets, and braces. (Notice that the `]` character is escaped, but the `[` character isn't.)

- **Atomic zero-width assertions** 

  Specify where the matching string should be but don't consume characters. For example, the `abc$` regular expression matches any abc word immediately before the end of a line without also matching the end of the line.

- **Quantifiers** 

  Specify that a subexpression must be repeated a given number of times. A particular quantifier applies to the character, character class, or group that immediately precedes it. For example, `\w+` matches all the words with one or more characters, whereas `\w{3,}` matches all the words with at least three characters. Quantifiers can be divided in two categories: greedy and lazy. A *greedy* quantifier, such as `*` and `+`, always matches as many characters as possible, whereas a *lazy* quantifier, such as `*?` and `+?`, attempts to match as few characters as possible.

- **Grouping constructors** 

  Can capture and name groups of subexpressions as well as increase the efficiency of regular expressions with noncapturing look-ahead and look-behind modifiers. For example, `(abc)+` matches repeated sequences of the "abc" string; `(?<total>\d+)` matches a group of one or more consecutive digits and assigns it the name total​, which can be used later inside the same regular expression pattern or for substitution purposes.

- **Substitutions** 

  Can be used only inside a replacement pattern and, together with character escapes, are the only constructs that can be used inside replacement patterns. For example, when the sequence `${total}` appears in a replacement pattern, it inserts the value of the group named total. Parentheses have no special meanings in replacement patterns, so you don't need to escape them.

- **Backreference constructs** 

  Enable you to reference a previous group of characters in the regular expression pattern by using its group number or name. You can use these constructs as a way to say "match the same thing again." For example, `(?<value>\d+)=\k<value>` matches identical numbers separated by an = symbol, as in the "123=123" sequence.

* **Alternating constructs** 

  Provide a way to specify alternatives; for example, the sequence `I (am|have)` can match both the "I am" and "I have" strings.

* **Miscellaneous constructs** 

  Include constructs that allow you to modify one or more regular expression options in the middle of the pattern. For example, `A(?i)BC` matches all the variants of the ABC word that begin with uppercase A (such as Abc, ABc, AbC, and ABC). See Table 14-2 for a description of all the regular expression options.

##### Table 14-1: The Regular Expression Language 

###### Character escapes

* any character
  Characters other than `.$^{[(\|)*+?\` match themselves.

* `\a`

    The bell alarm character (same as `\x07`).

* `\b`
  The backspace (same as `\x08`), but only when used between brackets or in a replacement pattern. Otherwise, it matches a word boundary.

* `\t`
  The tab character (same as `\x09`).

* `\r`
  The carriage return (same as `\x0D`).

* `\v`
  The vertical tab character (same as `\x0B`).

* `\f`
  The form-feed character (same as `\x0C`).

* `\n`
  The newline character (same as `\x0A`).

* `\e`
  The escape character (same as `\x1B`).

* `\040`
  An ASCII character expressed in octal notation (must have up to three octal digits). For example, `\040` is a space.

* `\x20`
  An ASCII character expressed in hexadecimal notation (must have exactly two digits). For example, `\x20` is a space.

* `\cC`
  An ASCII control character. For example, `\cC` is control+C.

* `\u0020`
  A Unicode character in hexadecimal notation (must have exactly four digits). For example, `\u0020` is a space.

* `\*`
  When the backslash is followed by a character in a way that doesn't form an escape sequence, it matches the character. For example, `\*` matches the `*` character.

###### Character classes

* `.`
  The dot character matches any character except the newline character. It matches any character, including newline, if you're using the `Singleline` option.

  补充：点表示除`\n`以外的字符，即`[^\n]`，注意点号匹配`\r`。当换行符是`\r\n`时，一个更明确的做法是使用正则表达式`[^\r\n]`匹配行内任意字符。

* `[aeiou]`
  Any character in the list between the opening and closing brackets; `[aeiou]` matches any vowel.

* `[^aeiou]`
  Any character except those in the list between the opening and closing brackets; `[^aeiou]` matches any non-vowel.

* `[a–zA–Z]`
  The - (dash) character enables you to specify ranges of characters: `[a–zA–Z]` matches any lowercase or uppercase character; `[^0–9]` matches any non-digit character. Notice, however, that accented letters aren't matched.

* `[a-z-[aeiou]]`
  Character class subtraction: when a pair of brackets is nested in another pair of brackets and is preceded by a minus sign, the regular expression matches all the characters in the outer pair except those in the inner pair. For example, `[a-z-[aeiou]]` matches any lowercase character that isn't a vowel. (Support for character class subtractions has been added in .NET Framework version 2.0.)

* `\w`
  A word character, which is an alphanumeric character or the underscore character; same as `[a-zA-Z_0-9]` but also matches accented letters and other alphabetical symbols.

* `\W`
  A nonword character; same as `[^a-zA-Z_0-9]` but also excludes accented letters and other alphabetical symbols.

* `\s`
  A white-space character, which is a space, a tab, a form-feed, a newline, a carriage return, or a vertical-feed character; same as `[ \f\n\r\t\v]`.

* `\S`
  A character other than a white-space character; same as `[^ \f\n\r\t\v]`.

* `\d`
  A decimal digit; same as `[0-9]`.

* `\D`
  A non-digit character; same as `[^0-9]`.

* `\p{name}`
  A character included in the named character class specified by {name}; supported names are Unicode groups and block ranges, for example, Ll, Nd, or Z.

* `\P{name}`
  A character not included in groups and block ranges specified in {name}.

###### Atomic zero-width assertions

* `^`
  The beginning of the string (or the beginning of the line if you're using the `Multiline` option).

* `$`
  The end of the string (or the end of the line if you're using the `Multiline` option).

* `\A`
  The beginning of a string (like `^` but ignores the `Multiline` option).

* `\Z`
  The end of the string or the position before the newline character at the end of the string (like `$` but ignores the `Multiline` option).

* `\z`
  Exactly the end of the string, whether or not there's a newline character (ignores the `Multiline` option).

* `\G`
  The position at which the current search started—usually one character after the point at which the previous search ended.

* `\b`
  The word boundary between `\w` (alphanumeric) and `\W` (non-alphanumeric) characters. It indicates the first and last characters of a word delimited by spaces or other punctuation symbols.

* `\B`
  Not on a word boundary.

###### Quantifiers

* `*`
  Zero or more matches; for example, `\bA\w*` matches a word that begins with A and is followed by zero or more alpha-numeric characters; same as `{0,}`.

* `+`
  One or more matches; for example, `\b[aeiou]+\b` matches a word composed only of vowels; same as `{1,}`.

* `?`
  Zero or one match; for example, `\b[aeiou]\d?\b` matches a word that starts with a vowel and is followed by zero or one digits; same as `{0,1}`.

* `{N}`
  Exactly N matches; for example, `[aeiou]{4}` matches four consecutive vowels.

* `{N,}`
  At least N matches; for example, `\d{3,}` matches groups of three or more digits.

* `{N,M}`
  Between N and M matches; for example, `\d{3,5}` matches groups of three, four, or five digits.

* `*?`
  Lazy `*`; the first match that consumes as few repeats as possible.

* `+?`
  Lazy `+`; the first match that consumes as few repeats as possible, but at least one.

* `??`
  Lazy `?`; zero repeats if possible, or one.

* `{N}?`
  Lazy `{N}`; equivalent to `{N}`.

* `{N,}?`
  Lazy `{N,}`; as few repeats as possible, but at least N.

* `{N,M}?`
  Lazy `{N,M}`; as few repeats as possible, but between N and M.

###### Grouping constructs

* `(substr)`
  Captures the matched substring. These captures are numbered automatically, based on the order of the left parenthesis, starting at 1. The zeroth capturing group is the text matched by the whole regular expression pattern.

* `(?<name>expr)` or `(?'name'expr)`
  Captures the subexpression and assigns it a name. The name must not contain any punctuation symbols.

* `(?:expr)`
  Noncapturing group, that is, a group that doesn't appear in the `Groups` collection of the `Match` object.

* `(?imnsx-imnsx: expr)`
  Enables or disables the options specified in the subexpression. For example, `(?i-s)` uses case-insensitive searches and disables single-line mode (see Table 14-2 for information about regular expression options).

* `(?=expr)`
  Zero-width positive look-ahead assertion; continues match only if the subexpression matches at this position on the right. For example, `\w+(?=,)` matches a word followed by a comma, without matching the comma.

* `(?!expr)`
  Zero-width negative look-ahead assertion; continues match only if the subexpression doesn't match at this position on the right. For example, `\w+\b(?![,:;])` matches a word that isn't followed by a comma, a colon, or a semicolon.

* `(?<=expr)`
  Zero-width positive look-behind assertion; continues match only if the subexpression matches at this position on the left. For example, `(?<=[,:])\w+` matches a word that follows a comma or semicolon, without matching the comma or semicolon. This construct doesn't backtrack.

* `(?<!expr)`
  Zero-width negative look-behind assertion; continues match only if the subexpression doesn't match at this position on the left. For example, `(?<!,)\b\w+` matches a word that doesn't follow a comma.

* `(?>expr)`
  Nonbacktracking subexpression; the subexpression is fully matched once, and it doesn't participate in backtracking. The subexpression matches only strings that would be matched by the subexpression alone.

* `(?<name1-name2>expr)` or `(?'name1-name2'expr)`
  Balancing group definition. Deletes the definition of the previously defined group name2 and stores in group name1 the interval between the previously defined name2 group and the current group. If no group name2 is defined, the match backtracks. Because deleting the last definition of name2 reveals the previous definition of name2, this construct allows the stack of captures for group name2 to be used as a counter for keeping track of nested constructs such as parentheses.

###### Substitution

* `$N`
  Substitutes the last substring matched by group number N ($0 replaces the entire match).

* `${name}`
  Substitutes the last substring matched by a `(?<name>)` group.

* `$&`
  Substitutes the entire match (same as $0).

* `$_`
  Substitutes the entire source string.

* `` $` ``
  Substitutes the portion of the source string up to the match.

* `$'`
  Substitutes the portion of the source string that follows the match.

* `$+`
  Substitutes the last captured group.

* `$$`
  A single dollar symbol (only when it appears in a substitution pattern).

###### Back reference constructs

* `\N` or `\NN`
  Back reference to a previous group. For example, `(\w)\1` finds doubled word characters, such as ss in expression. A backslash followed by a single digit is always considered a back reference (and throws a parsing exception if such a numbered reference is missing); a backslash followed by two digits is considered a numbered back reference if there's a corresponding numbered reference; otherwise, it's considered an octal code. In case of ambiguity, use the `\k<name>` construct.

* `\k<name>` or `\k'name'`
  Named back reference. `(?<char>\w)\d\k<char>` matches a word character followed by a digit and then by the same word character, as in the "B2B" string.

###### Alternating constructs

* `|`
  Either/or. For example, `vb|c#|java`. Leftmost successful match wins.

* `(?(expr)yes|no)`
  Matches the yes part if the expression matches at this point; otherwise, matches the no part. The expression is turned into a zero-width assertion. If the expression is the name of a named group or a capturing group number, the alternation is interpreted as a capture test (see next case).

* `(?(name)yes|no)`
  Matches the yes part if the named capture string has a match; otherwise, matches the no part. The no part can be omitted. If the given name doesn't correspond to the name or number of a capturing group used in this expression, the alternation is interpreted as an expression test (see previous case).

###### Miscellaneous constructs

* `(?imnsx-imnsx)`
  Enables or disables one or more regular expression options. For example, it allows case sensitivity to be turned on or off in the middle of a pattern. Option changes are effective until the closing parenthesis (see also the corresponding grouping construct, which is a cleaner form).

* `(?# comment)`
  Inline comment inserted within a regular expression. The text that follows the `#` sign and continues until the first closing) character is ignored.

* `#`
  X-mode comment; the text that follows an unescaped `#` until the end of line is ignored. This construct requires that the `x` option or the `RegexOptions.IgnorePatternWhiteSpace` enumerated option be activated.

### Regular Expression Options

The `Match`, `Matches`, and `Replace` static methods of the `Regex` object support an optional argument, which lets you specify one or more options to be applied to the regular expression search (see Table 14-2). For example, the following code searches for all occurrences of the "abc" word, regardless of its case:

```F#
let source: String = "ABC Abc abc"
let mc: MatchCollection = Regex.Matches(source, "abc")
Console.WriteLine(mc.Count) // => 1
let mc = Regex.Matches(source, "abc", RegexOptions.IgnoreCase)
Console.WriteLine(mc.Count) // => 3
```

By default, the `Regex` class transforms the regular expression into a sequence of opcodes, which are then interpreted when the pattern is applied to a specific source string. If you specify the `RegexOptions.Compiled` option, however, the regular expression is compiled into IL rather than regular expression opcodes. This feature enables the Just-In-Time (JIT) compiler to convert the expression to native CPU instructions, which clearly deliver better performance:

```F#
// Create a compiled regular expression that searches
// words that start with uppercase or lowercase A.
let reComp = new Regex(@"\Aw+", RegexOptions.IgnoreCase ||| RegexOptions.Compiled)
```

The extra performance that the `Compiled` option can buy you varies depending on the specific regular expression, but you can reasonably expect a twofold increase in speed in most cases. However, the extra compilation step adds some overhead, so you should use this option only if you plan to use the regular expression multiple times. Another factor that you should take into account when using the `RegexOptions.Compiled` option is that the compiled IL code isn't unloaded when the `Regex` object is reclaimed by the garbage collector—it continues to take memory until the application terminates. So you should preferably limit the number of compiled regular expressions. Also, consider that the `Regex` class caches all regular expression opcodes in memory, so a regular expression isn't generally reparsed each time it's used. The caching mechanism also works when you use static methods and don't explicitly create `Regex` instances.

The `RegexOptions.IgnorePatternWhitespace` option tells the `Regex` object to ignore spaces, tabs, and newline characters in the pattern and to enable #-prefixed remarks. You see the usefulness of this option when you want to format the pattern with a more meaningful layout and to explain what each of its portions does:

```F#
// Match a string optionally enclosed in single or double quotation marks.
let pattern: String = """
    \s*         # ignore leading spaces
    (           # two cases: quoted or unquoted string
    (?<quote>   # case 1: define a group named 'quote' '
    ['"])       # the group is a single or a double quote
    .*?         # a sequence of characters (lazy matching)
    \k<quote>   # followed by the same quote char
    |           # end of case 1
    [^'""]+     # case 2: a string without quotes
    )           # end of case 2
    \s*         # ignore trailing spaces
"""
let re = new Regex(pattern, RegexOptions.IgnorePatternWhitespace)
```

Because spaces are ignored, to match the space character you must use either the `[]` character class or the `\x20` character escape (or another equivalent escape sequence) when the `IgnorePatternWhitespace` option is used.

The `RegexOptions.Multiline` option enables multi-line mode, which is especially useful when you're parsing text files instead of plain strings. This option modifies the meaning and the behavior of the `^` and `$` assertions so that they match the start and end of each line of text, respectively, rather than the start or end of the whole string. Thanks to this option, you need only a handful of statements to create a grep-like utility that displays how many occurrences of the regular expression passed in the first argument are found in the files indicated by the second argument:

```F#
let FileGrep (pattern: String) (filespec: String) =
    // Create the regular expression (throws if pattern is invalid).
    let filePattern = new Regex(pattern, RegexOptions.IgnoreCase ||| RegexOptions.Multiline)

    // Apply the regular expression to each file in specified or current directory.
    let dirname: String =
        match Path.GetDirectoryName(filespec) with
        | "" -> Directory.GetCurrentDirectory()
        | nm -> nm

    let search: String = Path.GetFileName(filespec)

    for fname: String in Directory.GetFiles(dirname, search) do
        // Read file contents and apply the regular expression to it.
        let text: String = File.ReadAllText(fname)
        let mc: MatchCollection = filePattern.Matches(text)
        // Display filename if one or more matches.
        if mc.Count > 0 then
            Console.WriteLine("{0} [{1} matches]", fname, mc.Count)
```

For example, you can use the `FileGrep` utility to find all .vb source files in the current directory that contain the definition of a public `ArrayList` variable:

```F#
FileGrep @"^\s*Public\s+\w+\s+As\s+(New\s+)?ArrayList" "*.vb"
```

(For simplicity's sake, the regular expression doesn't account for variants of the basic syntax, such as the presence of the `ReadOnly` keyword or of the complete `System.Collections.ArrayList` class name.) It's easy to modify this code to display details about all occurrences or to extend the search to an entire directory tree.

~~**Note**~~
~~The Windows operating system includes a little-known command-line utility named `FindStr`, which supports searches with regular expressions and recursion over subdirectories, case-insensitive matches, display of lines that do *not* include the pattern, and so forth. Learn more by typing `FindStr /?` at the command prompt.~~

Another way to specify a regular expression option is by means of the `(?imnsx-imnsx)` construct, which lets you enable or disable one or more options from the current position to the end of the pattern string. The following code snippet finds all Dim, Private, and Public variable declarations at the beginning of individual text lines. Note that the regular expression options are specified inside the pattern string instead of as an argument of the `Regex.Matches` method:

```F#
// The pattern matches from the keyword to the end of the line.
let pattern: String = @"(?im)^\s+(dim|public|private) \w+: .+(?=\r\n)"
let source: String = File.ReadAllText("Module1.vb")
let mc: MatchCollection = Regex.Matches(source, pattern)
```

### Table 14-2: Regular Expression Options
These regular expression options can be specified when you create the `Regex` object. If a character is provided in the middle column, they can be specified also from inside a `(?)` construct. All these options are turned off by default.

* `None`
  No option.

* `IgnoreCase` (?i)
  Case insensitivity match.

* `Singleline` (?s)
  Singleline mode; changes the behavior of the `.`(dot) character so that it matches any character (instead of any character but the newline character).

* `Multiline` (?m)
  Multiline mode; changes the behavior of `^` and `$` so that they match the beginning and end of individual lines, respectively, instead of the whole string.

* `ExplicitCapture` (?n)
  Captures only explicitly named or numbered groups of the form `(?<name>)` so that naked parentheses act as noncapturing groups without your having to use the `(?:)` construct.

* `IgnorePatternWhitespace` (?x)
  Ignores unescaped white space from the pattern and enables comments marked with `#`. Significant spaces in the pattern must be specified as `[ ]` or `\x20`.

* `CultureInvariant`
  Uses the culture implied by `CultureInfo.InvariantCulture`, instead of the locale assigned to the current thread.

* `Compiled`
  Compiles the regular expression and generates IL code; this option generates faster code at the expense of longer startup time.

* `ECMAScript`
  Enables ECMAScript-compliant behavior. This flag can be used only in conjunction with the `IgnoreCase`, `Multiline`, and `Compiled` flags.

* `RightToLeft`
  Specifies that the search is from right to left instead of from left to right. If a starting index is specified, it should point to the end of the string.

