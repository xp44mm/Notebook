### 40.1 Creating regular expressions

#### 40.1.1 Literal vs. constructor



The two main ways of creating regular expressions are:

- Literal: compiled statically (at load time).

  ```
  /abc/ui
  ```

- Constructor: compiled dynamically (at runtime).

  ```
  new RegExp('abc', 'ui')
  ```

Both regular expressions have the same two parts:

- The *body* `abc` – the actual regular expression.
- The *flags* `u` and `i`. Flags configure how the pattern is interpreted. For example, `i` enables case-insensitive matching. A list of available flags is given [later in this chapter](https://exploringjs.com/impatient-js/ch_regexps.html#reg-exp-flags).

#### 40.1.2 Cloning and non-destructively modifying regular expressions

There are two variants of the constructor `RegExp()`:

- `new RegExp(pattern : string, flags = '')` [ES3]

  A new regular expression is created as specified via `pattern`. If `flags` is missing, the empty string `''` is used.

- `new RegExp(regExp : RegExp, flags = regExp.flags)` [ES6]

  `regExp` is cloned. If `flags` is provided, then it determines the flags of the clone.

The second variant is useful for cloning regular expressions, optionally while modifying them. Flags are immutable and this is the only way of changing them – for example:

```js
function copyAndAddFlags(regExp, flagsToAdd='') {
  // The constructor doesn’t allow duplicate flags;
  // make sure there aren’t any:
  const newFlags = [...new Set(regExp.flags + flagsToAdd)].join('');
  return new RegExp(regExp, newFlags);
}
assert.equal(/abc/i.flags, 'i');
assert.equal(copyAndAddFlags(/abc/i, 'g').flags, 'gi');
```

### 40.2 Syntax

#### 40.2.1 Syntax characters

At the top level of a regular expression, the following *syntax characters* are special. They are escaped by prefixing a backslash (`\`).

```js
\ ^ $ . * + ? ( ) [ ] { } |
```

In regular expression literals, you must escape slashes:

```js
> /\//.test('/')
true
```

In the argument of `new RegExp()`, you don’t have to escape slashes:

```
> new RegExp('/').test('/')
true
```

#### 40.2.2 Basic atoms

*Atoms* are the basic building blocks of regular expressions.

- *Pattern characters* are all characters *except* syntax characters (`^`, `$`, etc.). Pattern characters match themselves. Examples: `A b %`

- `.` matches any character. You can use the flag `/s` (`dotall`) to control if the dot matches line terminators or not ([more below](https://exploringjs.com/impatient-js/ch_regexps.html#reg-exp-flags)).

- Character escapes

   

  (each escape matches a single fixed character):

  - Control escapes (for a few control characters):
    - `\f`: form feed (FF)
    - `\n`: line feed (LF)
    - `\r`: carriage return (CR)
    - `\t`: character tabulation
    - `\v`: line tabulation
  - Arbitrary control characters: `\cA` (Ctrl-A), …, `\cZ` (Ctrl-Z)
  - Unicode code units: `\u00E4`
  - Unicode code points (require flag `/u`): `\u{1F44D}`

- Character class escapes

   

  (each escape matches one out of a set of characters):

  - `    \d`: digits (same as `[0-9]`)
    - `\D`: non-digits

  - `\w`: “word” characters (same as `[A-Za-z0-9_]`, related to identifiers in programming languages)
  - `\W`: non-word characters
  
  - `    \s`
  : whitespace (space, tab, line terminators, etc.)
  
  - `\S`: non-whitespace
  
  - Unicode property escapes[ES2018]: `\p{White_Space}`,`\P{White_Space}`, etc.

    - Require flag `/u`.
- Described in the next subsection.

#### 40.2.3 Unicode property escapes [ES2018]

##### 40.2.3.1 Unicode character properties

In the Unicode standard, each character has *properties* – metadata describing it. Properties play an important role in defining the nature of a character. Quoting [the Unicode Standard, Sect. 3.3, D3](http://www.unicode.org/versions/Unicode9.0.0/ch03.pdf):

> The semantics of a character are determined by its identity, normative properties, and behavior.

These are a few examples of properties:

- `Name`: a unique name, composed of uppercase letters, digits, hyphens, and spaces – for example:
  - A: `Name = LATIN CAPITAL LETTER A`
- `🙂`: `Name = SLIGHTLY SMILING FACE`
  
- `General_Category`: categorizes characters – for example:
  - x: `General_Category = Lowercase_Letter`
- $: `General_Category = Currency_Symbol`
  
- `White_Space`: used for marking invisible spacing characters, such as spaces, tabs and newlines – for example:
  - \t: `White_Space = True`
- π: `White_Space = False`
  
- `Age`: version of the Unicode Standard in which a character was introduced – for example: The Euro sign € was added in version 2.1 of the Unicode standard.
  - €: `Age = 2.1`

- `Block`: a contiguous range of code points. Blocks don’t overlap and their names are unique. For example:
  - S: `Block = Basic_Latin` (range U+0000..U+007F)
- `🙂`: `Block = Emoticons` (range U+1F600..U+1F64F)
  
- `Script`: is a collection of characters used by one or more writing systems.
  - Some scripts support several writing systems. For example, the Latin script supports the writing systems English, French, German, Latin, etc.
- Some languages can be written in multiple alternate writing systems that are supported by multiple scripts. For example, Turkish used the Arabic script before it transitioned to the Latin script in the early 20th century.
  - Examples:
    - α: `Script = Greek`
    - Д: `Script = Cyrillic`

##### 40.2.3.2 Unicode property escapes

Unicode property escapes look like this:

1. `\p{prop=value}`: matches all characters whose property `prop` has the value `value`.
2. `\P{prop=value}`: matches all characters that do not have a property `prop` whose value is `value`.
3. `\p{bin_prop}`: matches all characters whose binary property `bin_prop` is True.
4. `\P{bin_prop}`: matches all characters whose binary property `bin_prop` is False.

Comments:

- You can only use Unicode property escapes if the flag `/u` is set. Without `/u`, `\p` is the same as `p`.

- Forms (3) and (4) can be used as abbreviations if the property is `General_Category`. For example, the following two escapes are equivalent:

  ```js
  \p{Lowercase_Letter}
  \p{General_Category=Lowercase_Letter}
  ```

Examples:

- Checking for whitespace:

  ```js
  > /^\p{White_Space}+$/u.test('\t \n\r')
  true
  ```

- Checking for Greek letters:

  ```js
  > /^\p{Script=Greek}+$/u.test('μετά')
  true
  ```

- Deleting any letters:

  ```js
  > '1π2ü3é4'.replace(/\p{Letter}/ug, '')
  '1234'
  ```

- Deleting lowercase letters:

  ```js
  > 'AbCdEf'.replace(/\p{Lowercase_Letter}/ug, '')
  'ACE'
  ```

Further reading:

- Lists of Unicode properties and their values: [“Unicode Standard Annex #44: Unicode Character Database”](https://unicode.org/reports/tr44/#Properties) (Editors: Mark Davis, Laurențiu Iancu, Ken Whistler)

#### 40.2.4 Character classes

A *character class* wraps *class ranges* in square brackets. The class ranges specify a set of characters:

- `[«class ranges»]` matches any character in the set.
- `[^«class ranges»]` matches any character not in the set.

Rules for class ranges:

- Non-syntax characters stand for themselves: `[abc]`
- Only the following four characters are special and must be escaped via slashes:`^ \ - ]`
  - `^` only has to be escaped if it comes first.
  - `-` need not be escaped if it comes first or last.
- Character escapes (`\n`, `\u{1F44D}`, etc.) have the usual meaning.
  - Watch out: `\b` stands for backspace. Elsewhere in a regular expression, it matches word boundaries.
- Character class escapes (`\d`, `\p{White_Space}`, etc.) have the usual meaning.
- Ranges of characters are specified via dashes: `[a-z]`

#### 40.2.5 Groups

- Positional capture group:`  (#+)`

  - Backreference: `\1`, `\2`, etc.

- Named capture group[ES2018]:`  (?<hashes>#+)`

  - Backreference: `\k`

- Noncapturing group: `(?:#+)`

#### 40.2.6 Quantifiers

By default, all of the following quantifiers are *greedy* (they match as many characters as possible):

- `?`: match never or once
- `*`: match zero or more times
- `+`: match one or more times
- `{n}`: match `n` times
- `{n,}`: match `n` or more times
- `{n,m}`: match at least `n` times, at most `m` times.

To make them *reluctant* (so that they match as few characters as possible), put question marks (`?`) after them:

```
> /".*"/.exec('"abc"def"')[0]  // greedy
'"abc"def"'
> /".*?"/.exec('"abc"def"')[0] // reluctant
'"abc"'
```

#### 40.2.7 Assertions

- `^` matches only at the beginning of the input

- `$` matches only at the end of the input

- `  \b`
  

  matches only at a word boundary

  - `\B` matches only when not at a word boundary

##### 40.2.7.1 Lookahead

**Positive lookahead:** `(?=«pattern»)` matches if `pattern` matches what comes next.

Example: sequences of lowercase letters that are followed by an `X`.

```js
> 'abcX def'.match(/[a-z]+(?=X)/g)
[ 'abc' ]
```

Note that the `X` itself is not part of the matched substring.

**Negative lookahead:** `(?!«pattern»)` matches if `pattern` does not match what comes next.

Example: sequences of lowercase letters that are not followed by an `X`.

```js
> 'abcX def'.match(/[a-z]+(?!X)/g)
[ 'ab', 'def' ]
```

##### 40.2.7.2 Lookbehind [ES2018]

**Positive lookbehind:** `(?<=«pattern»)` matches if `pattern` matches what came before.

Example: sequences of lowercase letters that are preceded by an `X`.

```js
> 'Xabc def'.match(/(?<=X)[a-z]+/g)
[ 'abc' ]
```

**Negative lookbehind:** `(? matches if `pattern` does not match what came before.

Example: sequences of lowercase letters that are not preceded by an `X`.

```
> 'Xabc def'.match(/(?<!X)[a-z]+/g)
[ 'bc', 'def' ]
```

Example: replace “.js” with “.html”, but not in “Node.js”.

```
> 'Node.js: index.js and main.js'.replace(/(?<!Node)\.js/g, '.html')
'Node.js: index.html and main.html'
```

#### 40.2.8 Disjunction (`|`)

Caveat: this operator has low precedence. Use groups if necessary:

- `^aa|zz$` matches all strings that start with `aa` and/or end with `zz`. Note that `|` has a lower precedence than `^` and `$`.
- `^(aa|zz)$` matches the two strings `'aa'` and `'zz'`.
- `^a(a|z)z$` matches the two strings `'aaz'` and `'azz'`.

### 40.3 Flags

| Literal flag | Property name | ES     | Description                   |
| :----------- | :------------ | :----- | :---------------------------- |
| `g`          | `global`      | ES3    | Match multiple times          |
| `i`          | `ignoreCase`  | ES3    | Match case-insensitively      |
| `m`          | `multiline`   | ES3    | `^` and `$` match per line    |
| `s`          | `dotall`      | ES2018 | Dot matches line terminators  |
| `u`          | `unicode`     | ES6    | Unicode mode (recommended)    |
| `y`          | `sticky`      | ES6    | No characters between matches |

The following regular expression flags are available in JavaScript (tbl. [20](https://exploringjs.com/impatient-js/ch_regexps.html#tbl:reg-exp-flags-table) provides a compact overview):

- `/g` (`.global`): fundamentally changes how the following methods work.

  - `RegExp.prototype.test()`
  - `RegExp.prototype.exec()`
  - `String.prototype.match()`

  How, is explained in [§40.6 “Flag `/g` and its pitfalls”](https://exploringjs.com/impatient-js/ch_regexps.html#regexp-flag-g). In a nutshell, without `/g`, the methods only consider the first match for a regular expression in an input string. With `/g`, they consider all matches.

- `/i` (`.ignoreCase`): switches on case-insensitive matching:

  ```js
  > /a/.test('A')
  false
  > /a/i.test('A')
  true
  ```

- `/m` (`.multiline`): If this flag is on, `^` matches the beginning of each line and `$` matches the end of each line. If it is off, `^` matches the beginning of the whole input string and `$` matches the end of the whole input string.

  ```js
  > 'a1\na2\na3'.match(/^a./gm)
  [ 'a1', 'a2', 'a3' ]
  > 'a1\na2\na3'.match(/^a./g)
  [ 'a1' ]
  ```

- `/u` (`.unicode`): This flag switches on the Unicode mode for a regular expression. That mode is explained in the next subsection.

- `/y` (`.sticky`): This flag mainly makes sense in conjunction with `/g`. When both are switched on, any match must directly follow the previous one (that is, it must start at index `.lastIndex` of the regular expression object). Therefore, the first match must be at index 0.

  ```js
  > 'a1a2 a3'.match(/a./gy)
  [ 'a1', 'a2' ]
  > '_a1a2 a3'.match(/a./gy) // first match must be at index 0
  null
  
  > 'a1a2 a3'.match(/a./g)
  [ 'a1', 'a2', 'a3' ]
  > '_a1a2 a3'.match(/a./g)
  [ 'a1', 'a2', 'a3' ]
  ```

  The main use case for `/y` is tokenization (during parsing).

- `/s` (`.dotall`): By default, the dot does not match line terminators. With this flag, it does:

  ```js
  > /./.test('\n')
  false
  > /./s.test('\n')
  true
  ```

  Workaround if `/s` isn’t supported: Use `[^]` instead of a dot.

  ```js
  > /[^]/.test('\n')
  true
  ```

#### 40.3.1 Flag: Unicode mode via `/u`

The flag `/u` switches on a special Unicode mode for regular expressions. That mode enables several features:

- In patterns, you can use Unicode code point escapes such as `\u{1F42A}` to specify characters. Code unit escapes such as `\u03B1` only have a range of four hexadecimal digits (which corresponds to the basic multilingual plane).

- In patterns, you can use Unicode property escapes such as `\p{White_Space}`.

- Many escapes are now forbidden. For example: `\a \- \:`

  Pattern characters always match themselves:

  ```js
  > /pa-:/.test('pa-:')
  true
  ```

  Without `/u`, there are some pattern characters that still match themselves if you escape them with backslashes:

  ```js
  > /\p\a\-\:/.test('pa-:')
  true
  ```

  With `/u`:

  - `\p` starts a Unicode property escape.
  - The remaining “self-matching” escapes are forbidden. As a consequence, they can now be used for new features in the future.

- The atomic units for matching are Unicode characters (code points), not JavaScript characters (code units).

The following subsections explain the last item in more detail. They use the following Unicode character to explain when the atomic units are Unicode characters and when they are JavaScript characters:

```js
const codePoint = '🙂';
const codeUnits = '\uD83D\uDE42'; // UTF-16

assert.equal(codePoint, codeUnits); // same string!
```

I’m only switching between `🙂` and `\uD83D\uDE42`, to illustrate how JavaScript sees things. Both are equivalent and can be used interchangeably in strings and regular expressions.

##### 40.3.1.1 Consequence: you can put Unicode characters in character classes

With `/u`, the two code units of `🙂` are treated as a single character:

```js
> /^[🙂]$/u.test('🙂')
true
```

Without `/u`, `🙂` is treated as two characters:

```js
> /^[\uD83D\uDE42]$/.test('\uD83D\uDE42')
false
> /^[\uD83D\uDE42]$/.test('\uDE42')
true
```

Note that `^` and `$` demand that the input string have a single character. That’s why the first result is `false`.

##### 40.3.1.2 Consequence: the dot operator (`.`) matches Unicode characters, not JavaScript characters

With `/u`, the dot operator matches Unicode characters:

```js
> '🙂'.match(/./gu).length
1
```

`.match()` plus `/g` returns an Array with all the matches of a regular expression.

Without `/u`, the dot operator matches JavaScript characters:

```js
> '\uD83D\uDE80'.match(/./g).length
2
```

##### 40.3.1.3 Consequence: quantifiers apply to Unicode characters, not JavaScript characters

With `/u`, a quantifier applies to the whole preceding Unicode character:

```js
> /^🙂{3}$/u.test('🙂🙂🙂')
true
```

Without `/u`, a quantifier only applies to the preceding JavaScript character:

```js
> /^\uD83D\uDE80{3}$/.test('\uD83D\uDE80\uDE80\uDE80')
true
```

### 40.4 Properties of regular expression objects

Noteworthy:

- Strictly speaking, only `.lastIndex` is a real instance property. All other properties are implemented via getters.
- Accordingly, `.lastIndex` is the only mutable property. All other properties are read-only. If you want to change them, you need to copy the regular expression (consult [§40.1.2 “Cloning and non-destructively modifying regular expressions”](https://exploringjs.com/impatient-js/ch_regexps.html#cloning-regexps) for details).

#### 40.4.1 Flags as properties

Each regular expression flag exists as a property with a longer, more descriptive name:

```js
> /a/i.ignoreCase
true
> /a/.ignoreCase
false
```

This is the complete list of flag properties:

- `.dotall` (`/s`)
- `.global` (`/g`)
- `.ignoreCase` (`/i`)
- `.multiline` (`/m`)
- `.sticky` (`/y`)
- `.unicode` (`/u`)

#### 40.4.2 Other properties

Each regular expression also has the following properties:

- `.source` [ES3]: The regular expression pattern

  ```js
  > /abc/ig.source
  'abc'
  ```

- `.flags` [ES6]: The flags of the regular expression

  ```js
  > /abc/ig.flags
  'gi'
  ```

- `.lastIndex` [ES3]: Used when flag `/g` is switched on. Consult [§40.6 “Flag `/g` and its pitfalls”](https://exploringjs.com/impatient-js/ch_regexps.html#regexp-flag-g) for details.

### 40.5 Methods for working with regular expressions

#### 40.5.1 In general, regular expressions match anywhere in a string

Note that, in general, regular expressions match anywhere in a string:

```js
> /a/.test('__a__')
true
```

You can change that by using assertions such as `^` or by using the flag `/y`:

```js
> /^a/.test('__a__')
false
> /^a/.test('a__')
true
```

#### 40.5.2 `regExp.test(str)`: is there a match? [ES3]

The regular expression method `.test()` returns `true` if `regExp` matches `str`:

```js
> /bc/.test('ABCD')
false
> /bc/i.test('ABCD')
true
> /\.mjs$/.test('main.mjs')
true
```

With `.test()` you should normally avoid the `/g` flag. If you use it, you generally don’t get the same result every time you call the method:

```js
> const r = /a/g;
> r.test('aab')
true
> r.test('aab')
true
> r.test('aab')
false
```

The results are due to `/a/` having two matches in the string. After all of those were found, `.test()` returns `false`.

#### 40.5.3 `str.search(regExp)`: at what index is the match? [ES3]

The string method `.search()` returns the first index of `str` at which there is a match for `regExp`:

```js
> '_abc_'.search(/abc/)
1
> 'main.mjs'.search(/\.mjs$/)
4
```

#### 40.5.4 `regExp.exec(str)`: capturing groups [ES3]

##### 40.5.4.1 Getting a match object for the first match

Without the flag `/g`, `.exec()` returns the captures of the first match for `regExp` in `str`:

```js
assert.deepEqual(
  /(a+)b/.exec('ab aab'),
  {
    0: 'ab',
    1: 'a',
    index: 0,
    input: 'ab aab',
    groups: undefined,
  }
);
```

The result is a *match object* with the following properties:

- `[0]`: the complete substring matched by the regular expression
- `[1]`: capture of positional group 1 (etc.)
- `.index`: where did the match occur?
- `.input`: the string that was matched against
- `.groups`: captures of named groups

##### 40.5.4.2 Named capture groups [ES2018]

The previous example contained a single positional group. The following example demonstrates named groups:

```js
assert.deepEqual(
  /(?<as>a+)b/.exec('ab aab'),
  {
    0: 'ab',
    1: 'a',
    index: 0,
    input: 'ab aab',
    groups: { as: 'a' },
  }
);
```

In the result of `.exec()`, you can see that a named group is also a positional group – its capture exists twice:

- Once as a positional capture (property `'1'`).
- Once as a named capture (property `groups.as`).

##### 40.5.4.3 Looping over multiple matches

If you want to retrieve all matches of a regular expression (not just the first one), you need to switch on the flag `/g`. Then you can call `.exec()` multiple times and get one match each time. After the last match, `.exec()` returns `null`.

```js
> const regExp = /(a+)b/g;
> regExp.exec('ab aab')
{ 0: 'ab', 1: 'a', index: 0, input: 'ab aab', groups: undefined }
> regExp.exec('ab aab')
{ 0: 'aab', 1: 'aa', index: 3, input: 'ab aab', groups: undefined }
> regExp.exec('ab aab')
null
```

Therefore, you can loop over all matches as follows:

```js
const regExp = /(a+)b/g;
const str = 'ab aab';

let match;
// Check for null via truthiness
// Alternative: while ((match = regExp.exec(str)) !== null)
while (match = regExp.exec(str)) {
  console.log(match[1]);
}
// Output:
// 'a'
// 'aa'
```

Sharing regular expressions with `/g` has a few pitfalls, which are explained [later](https://exploringjs.com/impatient-js/ch_regexps.html#regexp-flag-g).

 **Exercise: Extract quoted text via `.exec()`**

```
exercises/regexps/extract_quoted_test.mjs
```

#### 40.5.5 `str.match(regExp)`: return all matching substrings [ES3]

Without `/g`, `.match()` works like `.exec()` – it returns a single match object.

With `/g`, `.match()` returns all substrings of `str` that match `regExp`:

```js
> 'ab aab'.match(/(a+)b/g)
[ 'ab', 'aab' ]
```

If there is no match, `.match()` returns `null`:

```js
> 'xyz'.match(/(a+)b/g)
null
```

You can use the Or operator to protect yourself against `null`:

```js
const numberOfMatches = (str.match(regExp) || []).length;
```

#### 40.5.6 `str.replace(searchValue, replacementValue)` [ES3]

`.replace()` is overloaded – it works differently, depending on the types of its parameters:

- If `searchValue` is:

  - Regular expression without `/g`: Replace first match of this regular expression.
- Regular expression with `/g`: Replace all matches of this regular expression.
  - String: Replace first occurrence of this string (the string is interpreted verbatim, not as a regular expression). Alas, there is no way to replace every occurrence of a string. [Later in this chapter](https://exploringjs.com/impatient-js/ch_regexps.html#escapeForRegExp), we’ll see a tool function that converts a string into a regular expression that matches this string (e.g., `'*'` becomes `/\*/`).
  
- If `replacementValue` is:

  - String: Replace matches with this string. The character `$` has special meaning and lets you insert captures of groups and more (read on for details).
- Function: Compute strings that replace matches via this function.

The next two subsubsections assume that a regular expression with `/g` is being used.

##### 40.5.6.1 `replacementValue` is a string

If the replacement value is a string, the dollar sign has special meaning – it inserts text matched by the regular expression:

| Text | Result                                    |
| ---- | ----------------------------------------- |
| `$$` | single `$`                                |
| `$&` | complete match                            |
| $`   | text before match                         |
| `$'` | text after match                          |
| `$n` | capture of positional group `n` (`n` > 0) |
| `$`  | capture of named group `name` [ES2018]    |

Example: Inserting the text before, inside, and after the matched substring.

```js
> 'a1 a2'.replace(/a/g, "($`|$&|$')")
'(|a|1 a2)1 (a1 |a|2)2'
```

Example: Inserting the captures of positional groups.

```js
> const regExp = /^([A-Za-z]+): (.*)$/ug;
> 'first: Jane'.replace(regExp, 'KEY: $1, VALUE: $2')
'KEY: first, VALUE: Jane'
```

Example: Inserting the captures of named groups.

```js
> const regExp = /^(?<key>[A-Za-z]+): (?<value>.*)$/ug;
> 'first: Jane'.replace(regExp, 'KEY: $<key>, VALUE: $<value>')
'KEY: first, VALUE: Jane'
```

##### 40.5.6.2 `replacementValue` is a function

If the replacement value is a function, you can compute each replacement. In the following example, we multiply each non-negative integer that we find by two.

```js
assert.equal(
  '3 cats and 4 dogs'.replace(/[0-9]+/g, (all) => 2 * Number(all)),
  '6 cats and 8 dogs'
);
```

The replacement function gets the following parameters. Note how similar they are to match objects. These parameters are all positional, but I’ve included how one might name them:

- `all`: complete match
- `g1`: capture of positional group 1
- Etc.
- `index`: where did the match occur?
- `input`: the string in which we are replacing
- `groups` [ES2018]: captures of named groups (an object)

**Exercise: Change quotes via `.replace()` and a named group**

```
exercises/regexps/change_quotes_test.mjs
```

#### 40.5.7 Other methods for working with regular expressions

`String.prototype.split()` is described [in the chapter on strings](https://exploringjs.com/impatient-js/ch_strings.html#string-api-extracting). Its first parameter of `String.prototype.split()` is either a string or a regular expression. If it is the latter, then captures of groups appear in the result:

```js
> 'a:b : c'.split(':')
[ 'a', 'b ', ' c' ]
> 'a:b : c'.split(/ *: */)
[ 'a', 'b', 'c' ]
> 'a:b : c'.split(/( *):( *)/)
[ 'a', '', '', 'b', ' ', ' ', 'c' ]
```

### 40.6 Flag `/g` and its pitfalls

The following two regular expression methods work differently if `/g` is switched on:

- `RegExp.prototype.exec()`
- `RegExp.prototype.test()`

Then they can be called repeatedly and deliver all matches inside a string. Property `.lastIndex` of the regular expression is used to track the current position inside the string – for example:

```js
const r = /a/g;
assert.equal(r.lastIndex, 0);

assert.equal(r.test('aa'), true); // 1st match?
assert.equal(r.lastIndex, 1); // after 1st match

assert.equal(r.test('aa'), true); // 2nd match?
assert.equal(r.lastIndex, 2); // after 2nd match

assert.equal(r.test('aa'), false); // 3rd match?
assert.equal(r.lastIndex, 0); // start over
```

The next subsections explain the pitfalls of using `/g`. They are followed by a subsection that explains how to work around those pitfalls.

#### 40.6.1 Pitfall: You can’t inline a regular expression with flag `/g`

A regular expression with `/g` can’t be inlined. For example, in the following `while` loop, the regular expression is created fresh, every time the condition is checked. Therefore, its `.lastIndex` is always zero and the loop never terminates.

```js
let count = 0;
// Infinite loop
while (/a/g.test('babaa')) {
  count++;
}
```

#### 40.6.2 Pitfall: Removing `/g` can break code

If code expects a regular expression with `/g` and has a loop over the results of `.exec()` or `.test()`, then a regular expression without `/g` can cause an infinite loop:

```js
function countMatches(regExp) {
  let count = 0;
  // Infinite loop
  while (regExp.exec('babaa')) {
    count++;
  }
  return count;
}
countMatches(/a/); // Missing: flag /g
```

Why? Because `.exec()` always returns the first result, a match object, and never `null`.

#### 40.6.3 Pitfall: Adding `/g` can break code

With `.test()`, there is another caveat: if you want to check exactly once if a regular expression matches a string, then the regular expression must not have `/g`. Otherwise, you generally get a different result every time you call `.test()`:

```js
function isMatching(regExp) {
  return regExp.test('Xa');
}
const myRegExp = /^X/g;
assert.equal(isMatching(myRegExp), true);
assert.equal(isMatching(myRegExp), false);
```

Normally, you won’t add `/g` if you intend to use `.test()` in this manner. But it can happen if, for example, you use the same regular expression for testing and for replacing.

#### 40.6.4 Pitfall: Code can break if `.lastIndex` isn’t zero

If you match a regular expression multiple times via `.exec()` or `.test()`, the current position inside the input string is stored in the regular expression property `.lastIndex`. Therefore, code that matches multiple times may break if `.lastIndex` is not zero:

```js
function countMatches(regExp) {
  let count = 0;
  while (regExp.exec('babaa')) {
    count++;
  }
  return count;
}

const myRegExp = /a/g;
myRegExp.lastIndex = 4;
assert.equal(countMatches(myRegExp), 1); // should be 3
```

Note that `.lastIndex` is always zero in newly created regular expressions, but it may not be if the same regular expression is used multiple times.

#### 40.6.5 Dealing with `/g` and `.lastIndex`

As an example of dealing with `/g` and `.lastIndex`, we will implement the following function:

```js
countMatches(regExp, str)
```

It counts how often `regExp` has a match inside `str`. How do we prevent a wrong `regExp` from breaking our code? Let’s look at three approaches.

First, we can throw an exception if `/g` isn’t set or `.lastIndex` isn’t zero:

```js
function countMatches(regExp, str) {
  if (!regExp.global) {
    throw new Error('Flag /g of regExp must be set');
  }
  if (regExp.lastIndex !== 0) {
    throw new Error('regExp.lastIndex must be zero');
  }
  
  let count = 0;
  while (regExp.test(str)) {
    count++;
  }
  return count;
}
```

Second, we can clone the parameter. That has the added benefit that `regExp` won’t be changed.

```js
function countMatches(regExp, str) {
  const cloneFlags = regExp.flags + (regExp.global ? '' : 'g');
  const clone = new RegExp(regExp, cloneFlags);

  let count = 0;
  while (clone.test(str)) {
    count++;
  }
  return count;
}
```

Third, we can use `.match()` to count occurrences, which doesn’t change or depend on `.lastIndex`.

```js
function countMatches(regExp, str) {
  if (!regExp.global) {
    throw new Error('Flag /g of regExp must be set');
  }
  return (str.match(regExp) || []).length;
}
```

### 40.7 Techniques for working with regular expressions

#### 40.7.1 Escaping arbitrary text for regular expressions

The following function escapes an arbitrary text so that it is matched verbatim if you put it inside a regular expression:

```js
function escapeForRegExp(str) {
  return str.replace(/[\\^$.*+?()[\]{}|]/g, '\\$&'); // (A)
}
assert.equal(escapeForRegExp('[yes?]'), String.raw`\[yes\?\]`);
assert.equal(escapeForRegExp('_g_'), String.raw`_g_`);
```

In line A, we escape all syntax characters. We have to be selective because the regular expression flag `/u` forbids many escapes – for example: `\a \: \-`

The regular expression method `.replace()` only lets you replace plain text once. With `escapeForRegExp()`, we can work around that limitation and replace plain text multiple times:

```js
const plainText = ':-)';
const regExp = new RegExp(escapeForRegExp(plainText), 'ug');
assert.equal(
  ':-) :-) :-)'.replace(regExp, '🙂'), '🙂 🙂 🙂');
```

#### 40.7.2 Matching everything or nothing

Sometimes, you may need a regular expression that matches everything or nothing – for example, as a default value.

- Match everything: `/(?:)/`

  The empty group `()` matches everything. We make it non-capturing (via `?:`), to avoid unnecessary work.

  ```js
  > /(?:)/.test('')
  true
  > /(?:)/.test('abc')
  true
  ```

- Match nothing: `/.^/`

  `^` only matches at the beginning of a string. The dot moves matching beyond the first character and now `^` doesn’t match anymore.

  ```js
  > /.^/.test('')
  false
  > /.^/.test('abc')
  false
  ```