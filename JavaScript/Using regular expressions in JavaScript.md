https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions

## Using regular expressions in JavaScript

Regular expressions are used with the `RegExp` methods `test()` and `exec()` and with the `String` methods `match()`, `replace()`, `search()`, and `split()`. These methods are explained in detail in the [JavaScript reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference).

| Method                                                       | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`exec()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/exec) | Executes a search for a match in a string. It returns an array of information or `null` on a mismatch. |
| [`test()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test) | Tests for a match in a string. It returns `true` or `false`. |
| [`match()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/match) | Returns an array containing all of the matches, including capturing groups, or `null` if no match is found. |
| [`matchAll()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/matchAll) | Returns an iterator containing all of the matches, including capturing groups. |
| [`search()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/search) | Tests for a match in a string. It returns the index of the match, or `-1` if the search fails. |
| [`replace()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace) | Executes a search for a match in a string, and replaces the matched substring with a replacement substring. |
| [`replaceAll()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replaceAll) | Executes a search for all matches in a string, and replaces the matched substrings with a replacement substring. |
| [`split()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split) | Uses a regular expression or a fixed string to break a string into an array of substrings. |

When you want to know whether a pattern is found in a string, use the `test()` or `search()` methods; for more information (but slower execution) use the `exec()` or `match()` methods. If you use `exec()` or `match()` and if the match succeeds, these methods return an array and update properties of the associated regular expression object and also of the predefined regular expression object, `RegExp`. If the match fails, the `exec()` method returns `null` (which coerces to `false`).

In the following example, the script uses the `exec()` method to find a match in a string.

```js
var myRe = /d(b+)d/g;
var myArray = myRe.exec('cdbbdbsbz');
```

If you do not need to access the properties of the regular expression, an alternative way of creating `myArray` is with this script:

```js
var myArray = /d(b+)d/g.exec('cdbbdbsbz'); 
//outputs Array [ 'dbbd', 'bb', index: 1, input: 'cdbbdbsbz' ]
```
similar to 
```js
"cdbbdbsbz".match(/d(b+)d/g); 
//outputs Array [ "dbbd" ]
```

(See [different behaviors](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions#g-different-behaviors) for further info about the different behaviors.)

If you want to construct the regular expression from a string, yet another alternative is this script:

```js
var myRe = new RegExp('d(b+)d', 'g');
var myArray = myRe.exec('cdbbdbsbz');
```

With these scripts, the match succeeds and returns the array and updates the properties shown in the following table.

| Object    | Property or index                                            | Description                                                  | In this example                                |
| :-------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :--------------------------------------------- |
| `myArray` |                                                              | The matched string and all remembered substrings.            | `['dbbd', 'bb', index: 1, input: 'cdbbdbsbz']` |
| `index`   | The 0-based index of the match in the input string.          | `1`                                                          |                                                |
| `input`   | The original string.                                         | `'cdbbdbsbz'`                                                |                                                |
| `[0]`     | The last matched characters.                                 | `'dbbd'`                                                     |                                                |
| `myRe`    | `lastIndex`                                                  | The index at which to start the next match. (This property is set only if the regular expression uses the g option, described in [Advanced Searching With Flags](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions#Advanced_searching_with_flags).) | `5`                                            |
| `source`  | The text of the pattern. Updated at the time that the regular expression is created, not executed. | `'d(b+)d'`                                                   |                                                |

As shown in the second form of this example, you can use a regular expression created with an object initializer without assigning it to a variable. If you do, however, every occurrence is a new regular expression. For this reason, if you use this form without assigning it to a variable, you cannot subsequently access the properties of that regular expression. For example, assume you have this script:

```js
var myRe = /d(b+)d/g;
var myArray = myRe.exec('cdbbdbsbz');
console.log('The value of lastIndex is ' + myRe.lastIndex);

// "The value of lastIndex is 5"
```

However, if you have this script:

```js
var myArray = /d(b+)d/g.exec('cdbbdbsbz');
console.log('The value of lastIndex is ' + /d(b+)d/g.lastIndex);

// "The value of lastIndex is 0"
```

The occurrences of `/d(b+)d/g` in the two statements are different regular expression objects and hence have different values for their `lastIndex` property. If you need to access the properties of a regular expression created with an object initializer, you should first assign it to a variable.

### Advanced searching with flags

Regular expressions have six optional flags that allow for functionality like global and case insensitive searching. These flags can be used separately or together in any order, and are included as part of the regular expression.

| Flag | Description                                                  | Corresponding property        |
| :--- | :----------------------------------------------------------- | :---------------------------- |
| `g`  | Global search.                                               | `RegExp.prototype.global`     |
| `i`  | Case-insensitive search.                                     | `RegExp.prototype.ignoreCase` |
| `m`  | Multi-line search.                                           | `RegExp.prototype.multiline`  |
| `s`  | Allows `.` to match newline characters. (Added in ES2018, not yet supported in Firefox). | `RegExp.prototype.dotAll`     |
| `u`  | "unicode"; treat a pattern as a sequence of unicode code points. | `RegExp.prototype.unicode`    |
| `y`  | Perform a "sticky" search that matches starting at the current position in the target string. See [`sticky`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/sticky). | `RegExp.prototype.sticky`     |

To include a flag with the regular expression, use this syntax:

```js
var re = /pattern/flags;
```

or

```js
var re = new RegExp('pattern', 'flags');
```

Note that the flags are an integral part of a regular expression. They cannot be added or removed later.

For example, `re = /\w+\s/g` creates a regular expression that looks for one or more characters followed by a space, and it looks for this combination throughout the string.

```js
var re = /\w+\s/g;
var str = 'fee fi fo fum';
var myArray = str.match(re);
console.log(myArray);

// ["fee ", "fi ", "fo "]
```

You could replace the line:

```js
var re = /\w+\s/g;
```

with:

```js
var re = new RegExp('\\w+\\s', 'g');
```

and get the same result.

The behavior associated with the `g` flag is different when the `.exec()` method is used. The roles of "class" and "argument" get reversed: In the case of `.match()`, the string class (or data type) owns the method and the regular expression is just an argument, while in the case of `.exec()`, it is the regular expression that owns the method, with the string being the argument.  Contrast this *`str.match(re)`* versus *`re.exec(str)`*. The `g` flag is used with the **`.exec()`** method to get iterative progression.

```js
var xArray; while(xArray = re.exec(str)) console.log(xArray);
// produces: 
// ["fee ", index: 0, input: "fee fi fo fum"]
// ["fi ", index: 4, input: "fee fi fo fum"]
// ["fo ", index: 7, input: "fee fi fo fum"]
```

The `m` flag is used to specify that a multiline input string should be treated as multiple lines. If the `m` flag is used, `^` and `$` match at the start or end of any line within the input string instead of the start or end of the entire string.

## Examples

**Note:** Several examples are also available in:

- The reference pages for [`exec()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/exec), [`test()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test), [`match()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/match), [`matchAll()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/matchAll), [`search()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/search), [`replace()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace), [`split()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split)
- This guide articles': [character classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions/Character_Classes), [assertions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions/Assertions), [groups and ranges](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions/Groups_and_Ranges), [quantifiers](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions/Quantifiers), [Unicode property escapes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions/Unicode_Property_Escapes)

### Using special characters to verify input



In the following example, the user is expected to enter a phone number. When the user presses the "Check" button, the script checks the validity of the number. If the number is valid (matches the character sequence specified by the regular expression), the script shows a message thanking the user and confirming the number. If the number is invalid, the script informs the user that the phone number is not valid.

Within non-capturing parentheses `(?:` , the regular expression looks for three numeric characters `\d{3}` OR `|` a left parenthesis `\(` followed by three digits` \d{3}`, followed by a close parenthesis `\)`, (end non-capturing parenthesis `)`), followed by one dash, forward slash, or decimal point and when found, remember the character `([-\/\.])`, followed by three digits `\d{3}`, followed by the remembered match of a dash, forward slash, or decimal point `\1`, followed by four digits `\d{4}`.

The `Change` event activated when the user presses Enter sets the value of `RegExp.input`.

#### HTML

```html
<p>
  Enter your phone number (with area code) and then click "Check".
  <br>
  The expected format is like ###-###-####.
</p>
<form action="#">
  <input id="phone">
    <button onclick="testInfo(document.getElementById('phone'));">Check</button>
</form>
```

#### JavaScript

```js
var re = /(?:\d{3}|\(\d{3}\))([-\/\.])\d{3}\1\d{4}/;
function testInfo(phoneInput) {
  var OK = re.exec(phoneInput.value);
  if (!OK) {
    console.error(phoneInput.value + ' isn\'t a phone number with area code!'); 
  } else {
    console.log('Thanks, your phone number is ' + OK[0]);}
} 
```

#### Result