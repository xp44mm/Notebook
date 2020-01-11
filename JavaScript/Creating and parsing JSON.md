## Creating and parsing JSON

JSON (“JavaScript Object Notation”) is a storage format that uses text to encode data. Its syntax is a subset of JavaScript expressions. As an example, consider the following text, stored in a file `jane.json`:

```
{
  "first": "Jane",
  "last": "Porter",
  "married": true,
  "born": 1890,
  "friends": [ "Tarzan", "Cheeta" ]
}
```

JavaScript has the global namespace object `JSON` that provides methods for creating and parsing JSON.

### 42.1 The discovery and standardization of JSON

A specification for JSON was published by Douglas Crockford in 2001, at [`json.org`](http://json.org/). He explains:

> I discovered JSON. I do not claim to have invented JSON because it already existed in nature. What I did was I found it, I named it, I described how it was useful. I don’t claim to be the first person to have discovered it; I know that there are other people who discovered it at least a year before I did. The earliest occurrence I’ve found was, there was someone at Netscape who was using JavaScript array literals for doing data communication as early as 1996, which was at least five years before I stumbled onto the idea.

Later, JSON was standardized as [ECMA-404](https://www.ecma-international.org/publications/standards/Ecma-404.htm):

- 1st edition: October 2013
- 2nd edition: December 2017

#### 42.1.1 JSON’s grammar is frozen

Quoting the ECMA-404 standard:

> Because it is so simple, it is not expected that the JSON grammar will ever change. This gives JSON, as a foundational notation, tremendous stability.

Therefore, JSON will never get improvements such as optional trailing commas, comments, or unquoted keys – independently of whether or not they are considered desirable. However, that still leaves room for creating supersets of JSON that compile to plain JSON.

### 42.2 JSON syntax

JSON consists of the following parts of JavaScript:

- Compound:
  - Object literals:
    - Property keys are double-quoted strings.
    - Property values are JSON values.
    - No trailing commas are allowed.
  - Array literals:
    - Elements are JSON values.
    - No holes or trailing commas are allowed.
- Atomic:
  - `null` (but not `undefined`)
  - Booleans
  - Numbers (excluding `NaN`, `+Infinity`, `-Infinity`)
  - Strings (must be double-quoted)

As a consequence, you can’t (directly) represent cyclic structures in JSON.

### 42.3 Using the `JSON` API

The global namespace object `JSON` contains methods for working with JSON data.

#### 42.3.1 `JSON.stringify(data, replacer?, space?)`

`.stringify()` converts JavaScript `data` to a JSON string. In this section, we are ignoring the parameter `replacer`; it is explained in [§42.4 “Customizing stringification and parsing”](https://exploringjs.com/impatient-js/ch_json.html#json-replacers-revivers).

##### 42.3.1.1 Result: a single line of text

If you only provide the first argument, `.stringify()` returns a single line of text:

```
assert.equal(
  JSON.stringify({foo: ['a', 'b']}),
  '{"foo":["a","b"]}' );
```

##### 42.3.1.2 Result: a tree of indented lines

If you provide a non-negative integer for `space`, then `.stringify()` returns one or more lines and indents by `space` spaces per level of nesting:

```
assert.equal(
JSON.stringify({foo: ['a', 'b']}, null, 2),
`{
  "foo": [
    "a",
    "b"
  ]
}`);
```

##### 42.3.1.3 Details on how JavaScript data is stringified

**Primitive values:**

- Supported primitive values are stringified as expected:

  ```
  > JSON.stringify('abc')
  '"abc"'
  > JSON.stringify(123)
  '123'
  > JSON.stringify(null)
  'null'
  ```

- Unsupported numbers: `'null'`

  ```
  > JSON.stringify(NaN)
  'null'
  > JSON.stringify(Infinity)
  'null'
  ```

- Other unsupported primitive values are not stringified; they produce the result `undefined`:

  ```
  > JSON.stringify(undefined)
  undefined
  > JSON.stringify(Symbol())
  undefined
  ```

**Objects:**

- If an object has a method `.toJSON()`, then the result of that method is stringified:

  ```
  > JSON.stringify({toJSON() {return true}})
  'true'
  ```

  Dates have a method `.toJSON()` that returns a string:

  ```
  > JSON.stringify(new Date(2999, 11, 31))
  '"2999-12-30T23:00:00.000Z"'
  ```

- Wrapped primitive values are unwrapped and stringified:

  ```
  > JSON.stringify(new Boolean(true))
  'true'
  > JSON.stringify(new Number(123))
  '123'
  ```

- Arrays are stringified as Array literals. Unsupported Array elements are stringified as if they were `null`:

  ```
  > JSON.stringify([undefined, 123, Symbol()])
  '[null,123,null]'
  ```

- All other objects – except for functions – are stringified as object literals. Properties with unsupported values are omitted:

  ```
  > JSON.stringify({a: Symbol(), b: true})
  '{"b":true}'
  ```

- Functions are not stringified:

  ```
  > JSON.stringify(() => {})
  undefined
  ```

#### 42.3.2 `JSON.parse(text, reviver?)`

`.parse()` converts a JSON `text` to a JavaScript value. In this section, we are ignoring the parameter `reviver`; it is explained [§42.4 “Customizing stringification and parsing”](https://exploringjs.com/impatient-js/ch_json.html#json-replacers-revivers).

This is an example of using `.parse()`:

```
> JSON.parse('{"foo":["a","b"]}')
{ foo: [ 'a', 'b' ] }
```

#### 42.3.3 Example: converting to and from JSON

The following class implements conversions from (line A) and to (line B) JSON.

```
class Point {
  static fromJson(jsonObj) { // (A)
    return new Point(jsonObj.x, jsonObj.y);
  }

  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
  
  toJSON() { // (B)
    return {x: this.x, y: this.y};
  }
}
```

- Converting JSON to a point: We use the static method `Point.fromJson()` to parse JSON and create an instance of `Point`.

  ```
  assert.deepEqual(
    Point.fromJson(JSON.parse('{"x":3,"y":5}')),
    new Point(3, 5) );
  ```

- Converting a point to JSON: `JSON.stringify()` internally calls [the previously mentioned method `.toJSON()`](https://exploringjs.com/impatient-js/ch_json.html#stringification-details).

  ```
  assert.equal(
    JSON.stringify(new Point(3, 5)),
    '{"x":3,"y":5}' );
  ```

**Exercise: Converting an object to and from JSON**

```
exercises/json/to_from_json_test.mjs
```

### 42.4 Customizing stringification and parsing (advanced)

Stringification and parsing can be customized as follows:

- `JSON.stringify(data, replacer?, space?)`

  The optional parameter `replacer` contains either:

  - An Array with names of properties. If a value in `data` is stringified as an object literal, then only the mentioned properties are considered. All other properties are ignored.
  - A *value visitor*, a function that can transform JavaScript data before it is stringified.

- `JSON.parse(text, reviver?)`

  The optional parameter `reviver` contains a value visitor that can transform the parsed JSON data before it is returned.

#### 42.4.1 `.stringfy()`: specifying which properties of objects to stringify

If the second parameter of `.stringify()` is an Array, then only object properties, whose names are mentioned there, are included in the result:

```
const obj = {
  a: 1,
  b: {
    c: 2,
    d: 3,
  }
};
assert.equal(
  JSON.stringify(obj, ['b', 'c']),
  '{"b":{"c":2}}');
```

#### 42.4.2 `.stringify()` and `.parse()`: value visitors

What I call a *value visitor* is a function that transforms JavaScript data:

- `JSON.stringify()` lets the value visitor in its parameter `replacer` transform JavaScript data before it is stringified.
- `JSON.parse()` lets the value visitor in its parameter `reviver` transform parsed JavaScript data before it is returned.

In this section, JavaScript data is considered to be a tree of values. If the data is atomic, it is a tree that only has a root. All values in the tree are fed to the value visitor, one at a time. Depending on what the visitor returns, the current value is omitted, changed, or preserved.

A value visitor has the following type signature:

```
type ValueVisitor = (key: string, value: any) => any;
```

The parameters are:

- `value`: The current value.

- ```
  this
  ```

  : Parent of current value. The parent of the root value

   

  ```
  r
  ```

   

  is

   

  ```
  {'': r}
  ```

  .

  - Note: `this` is an implicit parameter and only available if the value visitor is an ordinary function.

- `key`: Key or index of the current value inside its parent. The key of the root value is `''`.

The value visitor can return:

- `value`: means there won’t be any change.
- A different value `x`: leads to `value` being replaced with `x` in the output tree.
- `undefined`: leads to `value` being omitted in the output tree.

#### 42.4.3 Example: visiting values

The following code shows in which order a value visitor sees values:

```
const log = [];
function valueVisitor(key, value) {
  log.push({this: this, key, value});
  return value; // no change
}

const root = {
  a: 1,
  b: {
    c: 2,
    d: 3,
  }
};
JSON.stringify(root, valueVisitor);
assert.deepEqual(log, [
  { this: { '': root }, key: '',  value: root   },
  { this: root        , key: 'a', value: 1      },
  { this: root        , key: 'b', value: root.b },
  { this: root.b      , key: 'c', value: 2      },
  { this: root.b      , key: 'd', value: 3      },
]);
```

As we can see, the replacer of `JSON.stringify()` visits values top-down (root first, leaves last). The rationale for going in that direction is that we are converting JavaScript values to JSON values. And a single JavaScript object may be expanded into a tree of JSON-compatible values.

In contrast, the reviver of `JSON.parse()` visits values bottom-up (leaves first, root last). The rationale for going in that direction is that we are assembling JSON values into JavaScript values. Therefore, we need to convert the parts before we can convert the whole.

#### 42.4.4 Example: stringifying unsupported values

`JSON.stringify()` has no special support for regular expression objects – it stringifies them as if they were plain objects:

```
const obj = {
  name: 'abc',
  regex: /abc/ui,
};
assert.equal(
  JSON.stringify(obj),
  '{"name":"abc","regex":{}}');
```

We can fix that via a replacer:

```
function replacer(key, value) {
  if (value instanceof RegExp) {
    return {
      __type__: 'RegExp',
      source: value.source,
      flags: value.flags,
    };
  } else {
    return value; // no change
  }
}
assert.equal(
JSON.stringify(obj, replacer, 2),
`{
  "name": "abc",
  "regex": {
    "__type__": "RegExp",
    "source": "abc",
    "flags": "iu"
  }
}`);
```

#### 42.4.5 Example: parsing unsupported values

To `JSON.parse()` the result from the previous section, we need a reviver:

```
function reviver(key, value) {
  // Very simple check
  if (value && value.__type__ === 'RegExp') {
    return new RegExp(value.source, value.flags);
  } else {
    return value;
  }
}
const str = `{
  "name": "abc",
  "regex": {
    "__type__": "RegExp",
    "source": "abc",
    "flags": "iu"
  }
}`;
assert.deepEqual(
  JSON.parse(str, reviver),
  {
    name: 'abc',
    regex: /abc/ui,
  });
```

### 42.5 FAQ

#### 42.5.1 Why doesn’t JSON support comments?

Douglas Crockford explains why in [a Google+ post from 1 May 2012](https://web.archive.org/web/20190308024153/https://plus.google.com/+DouglasCrockfordEsq/posts/RK8qyGVaGSr):

> I removed comments from JSON because I saw people were using them to hold parsing directives, a practice which would have destroyed interoperability. I know that the lack of comments makes some people sad, but it shouldn’t.
>
> Suppose you are using JSON to keep configuration files, which you would like to annotate. Go ahead and insert all the comments you like. Then pipe it through JSMin [a minifier for JavaScript] before handing it to your JSON parser.