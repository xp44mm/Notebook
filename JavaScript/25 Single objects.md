## 25 Single objects


In this book, JavaScript’s style of object-oriented programming (OOP) is introduced in four steps. This chapter covers step 1; the next chapter covers steps 2–4. The steps are (fig. 8):

1. **Single objects (this chapter):** How do *objects*, JavaScript’s basic OOP building blocks, work in isolation?
2. **Prototype chains (next chapter):** Each object has a chain of zero or more *prototype objects*. Prototypes are JavaScript’s core inheritance mechanism.
3. **Classes (next chapter):** JavaScript’s *classes* are factories for objects. The relationship between a class and its instances is based on prototypal inheritance.
4. **Subclassing (next chapter):** The relationship between a *subclass* and its *superclass* is also based on prototypal inheritance.

![img](https://exploringjs.com/impatient-js/img-book/a5f3d243b908caeb74fa598e75fa5a8c0feee148.svg)

Figure 8: This book introduces object-oriented programming in JavaScript in four steps.

### 25.1 What is an object?



In JavaScript:

- An object is a set of *properties* (key-value entries).
- A property key can only be a string or a symbol.

#### 25.1.1 Roles of objects: record vs. dictionary



Objects play two roles in JavaScript:

- Records: Objects-as-records have a fixed number of properties, whose keys are known at development time. Their values can have different types.
- Dictionaries: Objects-as-dictionaries have a variable number of properties, whose keys are not known at development time. All of their values have the same type.

These roles influence how objects are explained in this chapter:

- First, we’ll explore objects-as-records. Even though property keys are strings or symbols under the hood, they will appear as fixed identifiers to us, in this part of the chapter.
- Later, we’ll explore objects-as-dictionaries. Note that Maps are usually better dictionaries than objects. However, some of the operations that we’ll encounter, can also be useful for objects-as-records.

### 25.2 Objects as records

Let’s first explore the role *record* of objects.

#### 25.2.1 Object literals: properties



*Object literals* are one way of creating objects-as-records. They are a stand-out feature of JavaScript: you can directly create objects – no need for classes! This is an example:

```js
const jane = {
  first: 'Jane',
  last: 'Doe', // optional trailing comma
};
```

In the example, we created an object via an object literal, which starts and ends with curly braces `{}`. Inside it, we defined two *properties* (key-value entries):

- The first property has the key `first` and the value `'Jane'`.
- The second property has the key `last` and the value `'Doe'`.

We will later see other ways of specifying property keys, but with this way of specifying them, they must follow the rules of JavaScript variable names. For example, you can use `first_name` as a property key, but not `first-name`). However, reserved words are allowed:

```js
const obj = {
  if: true,
  const: true,
};
```

In order to check the effects of various operations on objects, we’ll occasionally use `Object.keys()` in this part of the chapter. It lists property keys:

```js
> Object.keys({a:1, b:2})
[ 'a', 'b' ]
```

#### 25.2.2 Object literals: property value shorthands



Whenever the value of a property is defined via a variable name and that name is the same as the key, you can omit the key.

```js
function createPoint(x, y) {
  return {x, y};
}
assert.deepEqual(
  createPoint(9, 2),
  { x: 9, y: 2 }
);
```

#### 25.2.3 Getting properties

This is how you *get* (read) a property (line A):

```js
const jane = {
  first: 'Jane',
  last: 'Doe',
};

// Get property .first
assert.equal(jane.first, 'Jane'); // (A)
```

Getting an unknown property produces `undefined`:

```js
assert.equal(jane.unknownProperty, undefined);
```

#### 25.2.4 Setting properties

This is how you *set* (write to) a property:

```js
const obj = {
  prop: 1,
};
assert.equal(obj.prop, 1);
obj.prop = 2; // (A)
assert.equal(obj.prop, 2);
```

We just changed an existing property via setting. If we set an unknown property, we create a new entry:

```js
const obj = {}; // empty object
assert.deepEqual(
  Object.keys(obj), []);

obj.unknownProperty = 'abc';
assert.deepEqual(
  Object.keys(obj), ['unknownProperty']);
```

#### 25.2.5 Object literals: methods



The following code shows how to create the method `.says()` via an object literal:

```js
const jane = {
  first: 'Jane', // data property
  says(text) {   // method
    return `${this.first} says “${text}”`; // (A)
  }, // comma as separator (optional at end)
};
assert.equal(jane.says('hello'), 'Jane says “hello”');
```

During the method call `jane.says('hello')`, `jane` is called the *receiver* of the method call and assigned to the special variable `this`. That enables method `.says()` to access the sibling property `.first` in line A.

#### 25.2.6 Object literals: accessors



There are two kinds of accessors in JavaScript:

- A *getter* is a method-like entity that is invoked by getting a property.
- A *setter* is a method-like entity that is invoked by setting a property.

##### 25.2.6.1 Getters



A getter is created by prefixing a method definition with the modifier `get`:

```js
const jane = {
  first: 'Jane',
  last: 'Doe',
  get full() {
    return `${this.first} ${this.last}`;
  },
};

assert.equal(jane.full, 'Jane Doe');
jane.first = 'John';
assert.equal(jane.full, 'John Doe');
```

##### 25.2.6.2 Setters



A setter is created by prefixing a method definition with the modifier `set`:

```js
const jane = {
  first: 'Jane',
  last: 'Doe',
  set full(fullName) {
    const parts = fullName.split(' ');
    this.first = parts[0];
    this.last = parts[1];
  },
};

jane.full = 'Richard Roe';
assert.equal(jane.first, 'Richard');
assert.equal(jane.last, 'Roe');
```


### 25.3 Spreading into object literals (`...`)



Inside a function call, spreading (`...`) turns the iterated values of an *iterable* object into arguments.

Inside an object literal, a *spread property* adds the properties of another object to the current one:

```js
> const obj = {foo: 1, bar: 2};
> {...obj, baz: 3}
{ foo: 1, bar: 2, baz: 3 }
```

If property keys clash, the property that is mentioned last “wins”:

```js
> const obj = {foo: 1, bar: 2, baz: 3};
> {...obj, foo: true}
{ foo: true, bar: 2, baz: 3 }
> {foo: true, ...obj}
{ foo: 1, bar: 2, baz: 3 }
```

All values are spreadable, even `undefined` and `null`:

```js
> {...undefined}
{}
> {...null}
{}
> {...123}
{}
> {...'abc'}
{ '0': 'a', '1': 'b', '2': 'c' }
> {...['a', 'b']}
{ '0': 'a', '1': 'b' }
```

Property `.length` of strings and of Arrays is hidden from this kind of operation (it is not *enumerable*; see §25.7.3 “Property attributes and property descriptors” for more information).

#### 25.3.1 Use case for spreading: copying objects



You can use spreading to create a copy of an object `original`:

```js
const copy = {...original};
```

Caveat – copying is *shallow*: `copy` is a fresh object with duplicates of all properties (key-value entries) of `original`. But if property values are objects, then those are not copied themselves; they are shared between `original` and `copy`. Let’s look at an example:

```js
const original = { a: 1, b: {foo: true} };
const copy = {...original};
```

The first level of `copy` is really a copy: If you change any properties at that level, it does not affect the original:

```js
copy.a = 2;
assert.deepEqual(
  original, { a: 1, b: {foo: true} }); // no change
```

However, deeper levels are not copied. For example, the value of `.b` is shared between original and copy. Changing `.b` in the copy also changes it in the original.

```js
copy.b.foo = false;
assert.deepEqual(
  original, { a: 1, b: {foo: false} });
```

**JavaScript doesn’t have built-in support for deep copying**

*Deep copies* of objects (where all levels are copied) are notoriously difficult to do generically. Therefore, JavaScript does not have a built-in operation for them (for now). If you need such an operation, you have to implement it yourself.

#### 25.3.2 Use case for spreading: default values for missing properties

If one of the inputs of your code is an object with data, you can make properties optional by specifying default values that are used if those properties are missing. One technique for doing so is via an object whose properties contain the default values. In the following example, that object is `DEFAULTS`:

```js
const DEFAULTS = {foo: 'a', bar: 'b'};
const providedData = {foo: 1};

const allData = {...DEFAULTS, ...providedData};
assert.deepEqual(allData, {foo: 1, bar: 'b'});
```

The result, the object `allData`, is created by copying `DEFAULTS` and overriding its properties with those of `providedData`.

But you don’t need an object to specify the default values; you can also specify them inside the object literal, individually:

```js
const providedData = {foo: 1};

const allData = {foo: 'a', bar: 'b', ...providedData};
assert.deepEqual(allData, {foo: 1, bar: 'b'});
```

#### 25.3.3 Use case for spreading: non-destructively changing properties

So far, we have encountered one way of changing a property `.foo` of an object: We *set* it (line A) and mutate the object. That is, this way of changing a property is *destructive*.

```js
const obj = {foo: 'a', bar: 'b'};
obj.foo = 1; // (A)
assert.deepEqual(obj, {foo: 1, bar: 'b'});
```

With spreading, we can change `.foo` *non-destructively* – we make a copy of `obj` where `.foo` has a different value:

```js
const obj = {foo: 'a', bar: 'b'};
const updatedObj = {...obj, foo: 1};
assert.deepEqual(updatedObj, {foo: 1, bar: 'b'});
```



### 25.4 Methods



#### 25.4.1 Methods are properties whose values are functions

Let’s revisit the example that was used to introduce methods:

```js
const jane = {
  first: 'Jane',
  says(text) {
    return `${this.first} says “${text}”`;
  },
};
```

Somewhat surprisingly, methods are functions:

```js
assert.equal(typeof jane.says, 'function');
```

Why is that? We learned in the chapter on callable values, that ordinary functions play several roles. *Method* is one of those roles. Therefore, under the hood, `jane` roughly looks as follows.

```js
const jane = {
  first: 'Jane',
  says: function (text) {
    return `${this.first} says “${text}”`;
  },
};
```

#### 25.4.2 `.call()`: specifying `this` via a parameter

Remember that each function `someFunc` is also an object and therefore has methods. One such method is `.call()` – it lets you call a function while specifying `this` via a parameter:

```js
someFunc.call(thisValue, arg1, arg2, arg3);
```

##### 25.4.2.1 Methods and `.call()`

If you make a method call, `this` is an implicit parameter that is filled in via the receiver of the call:

```js
const obj = {
  method(x) {
    assert.equal(this, obj); // implicit parameter
    assert.equal(x, 'a');
  },
};

obj.method('a'); // receiver is `obj`
```

The method call in the last line sets up `this` as follows:

```js
obj.method.call(obj, 'a');
```

As an aside, that means that there are actually two different dot operators:

1. One for accessing properties: `obj.prop`
2. One for making method calls: `obj.prop()`

They are different in that (2) is not just (1) followed by the function call operator `()`. Instead, (2) additionally specifies a value for `this`.

##### 25.4.2.2 Functions and `.call()`

If you function-call an ordinary function, its implicit parameter `this` is also provided – it is implicitly set to `undefined`:

```js
function func(x) {
  assert.equal(this, undefined); // implicit parameter
  assert.equal(x, 'a');
}

func('a');
```

The method call in the last line sets up `this` as follows:

```js
func.call(undefined, 'a');
```

`this` being set to `undefined` during a function call, indicates that it is a feature that is only needed during a method call.

Next, we’ll examine the pitfalls of using `this`. Before we can do that, we need one more tool: method `.bind()` of functions.

#### 25.4.3 `.bind()`: pre-filling `this` and parameters of functions

`.bind()` is another method of function objects. This method is invoked as follows:

```js
const boundFunc = someFunc.bind(thisValue, arg1, arg2);
```

`.bind()` returns a new function `boundFunc()`. Calling that function invokes `someFunc()` with `this` set to `thisValue` and these parameters: `arg1`, `arg2`, followed by the parameters of `boundFunc()`.

That is, the following two function calls are equivalent:

```js
boundFunc('a', 'b')
someFunc.call(thisValue, arg1, arg2, 'a', 'b')
```

##### 25.4.3.1 An alternative to `.bind()`

Another way of pre-filling `this` and parameters is via an arrow function:

```js
const boundFunc2 = (...args) =>
  someFunc.call(thisValue, arg1, arg2, ...args);
```

##### 25.4.3.2 An implementation of `.bind()`

Considering the previous section, `.bind()` can be implemented as a real function as follows:

```js
function bind(func, thisValue, ...boundArgs) {
  return (...args) =>
    func.call(thisValue, ...boundArgs, ...args);
}
```

##### 25.4.3.3 Example: binding a real function

Using `.bind()` for real functions is somewhat unintuitive because you have to provide a value for `this`. Given that it is `undefined` during function calls, it is usually set to `undefined` or `null`.

In the following example, we create `add8()`, a function that has one parameter, by binding the first parameter of `add()` to `8`.

```js
function add(x, y) {
  return x + y;
}

const add8 = add.bind(undefined, 8);
assert.equal(add8(1), 9);
```

##### 25.4.3.4 Example: binding a method

In the following code, we turn method `.says()` into the stand-alone function `func()`:

```js
const jane = {
  first: 'Jane',
  says(text) {
    return `${this.first} says “${text}”`; // (A)
  },
};

const func = jane.says.bind(jane, 'hello');
assert.equal(func(), 'Jane says “hello”');
```

Setting `this` to `jane` via `.bind()` is crucial here. Otherwise, `func()` wouldn’t work properly because `this` is used in line A.

#### 25.4.4 `this` pitfall: extracting methods



We now know quite a bit about functions and methods and are ready to take a look at the biggest pitfall involving methods and `this`: function-calling a method extracted from an object can fail if you are not careful.

In the following example, we fail when we extract method `jane.says()`, store it in the variable `func`, and function-call `func()`.

```js
const jane = {
  first: 'Jane',
  says(text) {
    return `${this.first} says “${text}”`;
  },
};
const func = jane.says; // extract the method
assert.throws(
  () => func('hello'), // (A)
  {
    name: 'TypeError',
    message: "Cannot read property 'first' of undefined",
  });
```

The function call in line A is equivalent to:

```js
assert.throws(
  () => jane.says.call(undefined, 'hello'), // `this` is undefined!
  {
    name: 'TypeError',
    message: "Cannot read property 'first' of undefined",
  });
```

So how do we fix this? We need to use `.bind()` to extract method `.says()`:

```js
const func2 = jane.says.bind(jane);
assert.equal(func2('hello'), 'Jane says “hello”');
```

The `.bind()` ensures that `this` is always `jane` when we call `func()`.

讀書筆記：提取函數時，要綁定自己。

You can also use arrow functions to extract methods:

```js
const func3 = text => jane.says(text);
assert.equal(func3('hello'), 'Jane says “hello”');
```

##### 25.4.4.1 Example: extracting a method

The following is a simplified version of code that you may see in actual web development:

```js
class ClickHandler {
  constructor(id, elem) {
    this.id = id;
    elem.addEventListener('click', this.handleClick); // (A)
  }
  handleClick(event) {
    alert('Clicked ' + this.id);
  }
}
```

In line A, we don’t extract the method `.handleClick()` properly. Instead, we should do:

```js
elem.addEventListener('click', this.handleClick.bind(this));
```



#### 25.4.5 `this` pitfall: accidentally shadowing `this`

**Accidentally shadowing `this` is only an issue with ordinary functions**

Arrow functions don’t shadow `this`.

Consider the following problem: when you are inside an ordinary function, you can’t access the `this` of the surrounding scope because the ordinary function has its own `this`. In other words, a variable in an inner scope hides a variable in an outer scope. That is called [*shadowing*](https://exploringjs.com/impatient-js/ch_variables-assignment.html#shadowing-variables). The following code is an example:

```js
const prefixer = {
  prefix: '==> ',
  prefixStringArray(stringArray) {
    return stringArray.map(
      function (x) {
        return this.prefix + x; // (A)
      });
  },
};
assert.throws(
  () => prefixer.prefixStringArray(['a', 'b']),
  /^TypeError: Cannot read property 'prefix' of undefined$/);
```

In line A, we want to access the `this` of `.prefixStringArray()`. But we can’t since the surrounding ordinary function has its own `this` that *shadows* (blocks access to) the `this` of the method. The value of the former `this` is `undefined` due to the callback being function-called. That explains the error message.

The simplest way to fix this problem is via an arrow function, which doesn’t have its own `this` and therefore doesn’t shadow anything:

```js
const prefixer = {
  prefix: '==> ',
  prefixStringArray(stringArray) {
    return stringArray.map(
      (x) => {
        return this.prefix + x;
      });
  },
};
assert.deepEqual(
  prefixer.prefixStringArray(['a', 'b']),
  ['==> a', '==> b']);
```

We can also store `this` in a different variable (line A), so that it doesn’t get shadowed:

```js
prefixStringArray(stringArray) {
  const that = this; // (A)
  return stringArray.map(
    function (x) {
      return that.prefix + x;
    });
},
```

Another option is to specify a fixed `this` for the callback via `.bind()` (line A):

```js
prefixStringArray(stringArray) {
  return stringArray.map(
    function (x) {
      return this.prefix + x;
    }.bind(this)); // (A)
},
```

Lastly, `.map()` lets us specify a value for `this` (line A) that it uses when invoking the callback:

```js
prefixStringArray(stringArray) {
  return stringArray.map(
    function (x) {
      return this.prefix + x;
    },
    this); // (A)
},
```

#### 25.4.6 Avoiding the pitfalls of `this`



We have seen two big `this`-related pitfalls:

1. Extracting methods
2. Accidentally shadowing `this`

One simple rule helps avoid the second pitfall:

> “Avoid the keyword `function`”: Never use ordinary functions, only arrow functions (for real functions) and method definitions.

Following this rule has two benefits:

- It prevents the second pitfall because ordinary functions are never used as real functions.
- `this` becomes easier to understand because it will only appear inside methods (never inside ordinary functions). That makes it clear that `this` is an OOP feature.

However, even though I don’t use (ordinary) function *expressions* anymore, I do like function *declarations* syntactically. You can use them safely if you don’t refer to `this` inside them. The static checking tool ESLint can warn you during development when you do this wrong via a built-in rule.

Alas, there is no simple way around the first pitfall: whenever you extract a method, you have to be careful and do it properly – for example, by binding `this`.

#### 25.4.7 The value of `this` in various contexts



What is the value of `this` in various contexts?

Inside a callable entity, the value of `this` depends on how the callable entity is invoked and what kind of callable entity it is:

- Function call:
  - Ordinary functions: `this === undefined` (in strict mode)
  - Arrow functions: `this` is same as in surrounding scope (lexical `this`)
- Method call: `this` is receiver of call
- `new`: `this` refers to newly created instance

You can also access `this` in all common top-level scopes:

- `<script>` element: `this === globalThis`
- ECMAScript modules: `this === undefined`
- CommonJS modules: `this === module.exports`

However, I like to pretend that you can’t access `this` in top-level scopes because top-level `this` is confusing and rarely useful.

### 25.5 Objects as dictionaries (advanced)

Objects work best as records. But before ES6, JavaScript did not have a data structure for dictionaries (ES6 brought Maps). Therefore, objects had to be used as dictionaries, which imposed a significant constraint: keys had to be strings (symbols were also introduced with ES6).

We first look at features of objects that are related to dictionaries but also useful for objects-as-records. This section concludes with tips for actually using objects as dictionaries (spoiler: use Maps if you can).

#### 25.5.1 Arbitrary fixed strings as property keys

So far, we have always used objects as records. Property keys were fixed tokens that had to be valid identifiers and internally became strings:

```js
const obj = {
  mustBeAnIdentifier: 123,
};

// Get property
assert.equal(obj.mustBeAnIdentifier, 123);

// Set property
obj.mustBeAnIdentifier = 'abc';
assert.equal(obj.mustBeAnIdentifier, 'abc');
```

As a next step, we’ll go beyond this limitation for property keys: In this section, we’ll use arbitrary fixed strings as keys. In the next subsection, we’ll dynamically compute keys.

Two techniques allow us to use arbitrary strings as property keys.

First, when creating property keys via object literals, we can quote property keys (with single or double quotes):

```js
const obj = {
  'Can be any string!': 123,
};
```

Second, when getting or setting properties, we can use square brackets with strings inside them:

```js
// Get property
assert.equal(obj['Can be any string!'], 123);

// Set property
obj['Can be any string!'] = 'abc';
assert.equal(obj['Can be any string!'], 'abc');
```

You can also use these techniques for methods:

```js
const obj = {
  'A nice method'() {
    return 'Yes!';
  },
};

assert.equal(obj['A nice method'](), 'Yes!');
```

#### 25.5.2 Computed property keys



So far, property keys were always fixed strings inside object literals. In this section we learn how to dynamically compute property keys. That enables us to use either arbitrary strings or symbols.

The syntax of dynamically computed property keys in object literals is inspired by dynamically accessing properties. That is, we can use square brackets to wrap expressions:

```js
const obj = {
  ['Hello world!']: true,
  ['f'+'o'+'o']: 123,
  [Symbol.toStringTag]: 'Goodbye', // (A)
};

assert.equal(obj['Hello world!'], true);
assert.equal(obj.foo, 123);
assert.equal(obj[Symbol.toStringTag], 'Goodbye');
```

The main use case for computed keys is having symbols as property keys (line A).

Note that the square brackets operator for getting and setting properties works with arbitrary expressions:

```js
assert.equal(obj['f'+'o'+'o'], 123);
assert.equal(obj['==> foo'.slice(-3)], 123);
```

Methods can have computed property keys, too:

```js
const methodKey = Symbol();
const obj = {
  [methodKey]() {
    return 'Yes!';
  },
};

assert.equal(obj[methodKey](), 'Yes!');
```

For the remainder of this chapter, we’ll mostly use fixed property keys again (because they are syntactically more convenient). But all features are also available for arbitrary strings and symbols.



#### 25.5.3 The `in` operator: is there a property with a given key?



The `in` operator checks if an object has a property with a given key:

```js
const obj = {
  foo: 'abc',
  bar: false,
};

assert.equal('foo' in obj, true);
assert.equal('unknownKey' in obj, false);
```

##### 25.5.3.1 Checking if a property exists via truthiness

You can also use a truthiness check to determine if a property exists:

```js
assert.equal(
  obj.foo ? 'exists' : 'does not exist',
  'exists');
assert.equal(
  obj.unknownKey ? 'exists' : 'does not exist',
  'does not exist');
```

The previous checks work because `obj.foo` is truthy and because reading a missing property returns `undefined` (which is falsy).

There is, however, one important caveat: truthiness checks fail if the property exists, but has a falsy value (`undefined`, `null`, `false`, `0`, `""`, etc.):

```js
assert.equal(
  obj.bar ? 'exists' : 'does not exist',
  'does not exist'); // should be: 'exists'
```

#### 25.5.4 Deleting properties



You can delete properties via the `delete` operator:

```js
const obj = {
  foo: 123,
};
assert.deepEqual(Object.keys(obj), ['foo']);

delete obj.foo;

assert.deepEqual(Object.keys(obj), []);
```

#### 25.5.5 Listing property keys



|                                  | enumerable | non-enumerable | string | symbol |
| :------------------------------- | :--------- | :------------- | :----- | :----- |
| `Object.keys()`                  | `✔`        |                | `✔`    |        |
| `Object.getOwnPropertyNames()`   | `✔`        | `✔`            | `✔`    |        |
| `Object.getOwnPropertySymbols()` | `✔`        | `✔`            |        | `✔`    |
| `Reflect.ownKeys()`              | `✔`        | `✔`            | `✔`    | `✔`    |

Each of the methods in tbl. 18 returns an Array with the own property keys of the parameter. In the names of the methods, you can see that the following distinction is made:

- A *property key* can be either a string or a symbol.
- A *property name* is a property key whose value is a string.
- A *property symbol* is a property key whose value is a symbol.

The next section describes the term *enumerable* and demonstrates each of the methods.

##### 25.5.5.1 Enumerability



*Enumerability* is an *attribute* of a property. Non-enumerable properties are ignored by some operations – for example, by `Object.keys()` (see tbl. 18) and by spread properties. By default, most properties are enumerable. The next example shows how to change that. It also demonstrates the various ways of listing property keys.

```js
const enumerableSymbolKey = Symbol('enumerableSymbolKey');
const nonEnumSymbolKey = Symbol('nonEnumSymbolKey');

// We create enumerable properties via an object literal
const obj = {
  enumerableStringKey: 1,
  [enumerableSymbolKey]: 2,
}

// For non-enumerable properties, we need a more powerful tool
Object.defineProperties(obj, {
  nonEnumStringKey: {
    value: 3,
    enumerable: false,
  },
  [nonEnumSymbolKey]: {
    value: 4,
    enumerable: false,
  },
});

assert.deepEqual(
  Object.keys(obj),
  [ 'enumerableStringKey' ]);
assert.deepEqual(
  Object.getOwnPropertyNames(obj),
  [ 'enumerableStringKey', 'nonEnumStringKey' ]);
assert.deepEqual(
  Object.getOwnPropertySymbols(obj),
  [ enumerableSymbolKey, nonEnumSymbolKey ]);
assert.deepEqual(
  Reflect.ownKeys(obj),
  [
    'enumerableStringKey', 'nonEnumStringKey',
    enumerableSymbolKey, nonEnumSymbolKey,
  ]);
```

`Object.defineProperties()` is explained later in this chapter.

#### 25.5.6 Listing property values via `Object.values()`

`Object.values()` lists the values of all enumerable properties of an object:

```js
const obj = {foo: 1, bar: 2};
assert.deepEqual(
  Object.values(obj),
  [1, 2]);
```

#### 25.5.7 Listing property entries via `Object.entries()`

`Object.entries()` lists key-value pairs of enumerable properties. Each pair is encoded as a two-element Array:

```
const obj = {foo: 1, bar: 2};
assert.deepEqual(
  Object.entries(obj),
  [
    ['foo', 1],
    ['bar', 2],
  ]);
```



#### 25.5.8 Properties are listed deterministically

Own (non-inherited) properties of objects are always listed in the following order:

1. Properties with string keys that contain integer indices (that includes Array indices):
   In ascending numeric order
2. Remaining properties with string keys:
   In the order in which they were added
3. Properties with symbol keys:
   In the order in which they were added

The following example demonstrates how property keys are sorted according to these rules:

```js
> Object.keys({b:0,a:0, 10:0,2:0})
[ '2', '10', 'b', 'a' ]
```

**The order of properties**

The ECMAScript specification describes in more detail how properties are ordered.

#### 25.5.9 Assembling objects via `Object.fromEntries()`

Given an iterable over [key, value] pairs, `Object.fromEntries()` creates an object:

```js
assert.deepEqual(
  Object.fromEntries([['foo',1], ['bar',2]]),
  {
    foo: 1,
    bar: 2,
  }
);
```

`Object.fromEntries()` does the opposite of `Object.entries()`.

To demonstrate both, we’ll use them to implement two tool functions from the library Underscore in the next subsubsections.

##### 25.5.9.1 Example: `pick(object, ...keys)`

[`pick`](https://underscorejs.org/#pick) returns a copy of `object` that only has those properties whose keys are mentioned as arguments:

```js
const address = {
  street: 'Evergreen Terrace',
  number: '742',
  city: 'Springfield',
  state: 'NT',
  zip: '49007',
};
assert.deepEqual(
  pick(address, 'street', 'number'),
  {
    street: 'Evergreen Terrace',
    number: '742',
  }
);
```

We can implement `pick()` as follows:

```js
function pick(object, ...keys) {
  const filteredEntries = Object.entries(object)
    .filter(([key, _value]) => keys.includes(key));
  return Object.fromEntries(filteredEntries);
}
```

##### 25.5.9.2 Example: `invert(object)`

`invert` returns a copy of `object` where the keys and values of all properties are swapped:

```js
assert.deepEqual(
  invert({a: 1, b: 2, c: 3}),
  {1: 'a', 2: 'b', 3: 'c'}
);
```

We can implement `invert()` like this:

```js
function invert(object) {
  const mappedEntries = Object.entries(object)
    .map(([key, value]) => [value, key]);
  return Object.fromEntries(mappedEntries);
}
```

##### 25.5.9.3 A simple implementation of `Object.fromEntries()`

The following function is a simplified version of `Object.fromEntries()`:

```js
function fromEntries(iterable) {
  const result = {};
  for (const [key, value] of iterable) {
    let coercedKey;
    if (typeof key === 'string' || typeof key === 'symbol') {
      coercedKey = key;
    } else {
      coercedKey = String(key);
    }
    result[coercedKey] = value;
  }
  return result;
}
```



#### 25.5.10 The pitfalls of using an object as a dictionary

If you use plain objects (created via object literals) as dictionaries, you have to look out for two pitfalls.

The first pitfall is that the `in` operator also finds inherited properties:

```js
const dict = {};
assert.equal('toString' in dict, true);
```

We want `dict` to be treated as empty, but the `in` operator detects the properties it inherits from its prototype, `Object.prototype`.

The second pitfall is that you can’t use the property key `__proto__` because it has special powers (it sets the prototype of the object):

```js
const dict = {};

dict['__proto__'] = 123;
// No property was added to dict:
assert.deepEqual(Object.keys(dict), []);
```

So how do we avoid these pitfalls?

- Whenever you can, use Maps. They are the best solution for dictionaries.
- If you can’t, use a library for objects-as-dictionaries that does everything safely.
- If you can’t, use an object without a prototype.

The following code demonstrates using objects without prototypes as dictionaries:

```js
const dict = Object.create(null); // no prototype

assert.equal('toString' in dict, false); // (A)

dict['__proto__'] = 123;
assert.deepEqual(Object.keys(dict), ['__proto__']);
```

We avoided both pitfalls: First, a property without a prototype does not inherit any properties (line A). Second, in modern JavaScript, `__proto__` is implemented via `Object.prototype`. That means that it is switched off if `Object.prototype` is not in the prototype chain.



### 25.6 Standard methods (advanced)

`Object.prototype` defines several standard methods that can be overridden to configure how an object is treated by the language. Two important ones are:

- `.toString()`
- `.valueOf()`

#### 25.6.1 `.toString()`

`.toString()` determines how objects are converted to strings:

```
> String({toString() { return 'Hello!' }})
'Hello!'
> String({})
'[object Object]'
```

#### 25.6.2 `.valueOf()`

`.valueOf()` determines how objects are converted to numbers:

```
> Number({valueOf() { return 123 }})
123
> Number({})
NaN
```

### 25.7 Advanced topics

The following subsections give brief overviews of a few advanced topics.

#### 25.7.1 `Object.assign()`

`Object.assign()` is a tool method:

```js
Object.assign(target, source_1, source_2, ···)
```

This expression assigns all properties of `source_1` to `target`, then all properties of `source_2`, etc. At the end, it returns `target` – for example:

```js
const target = { foo: 1 };

const result = Object.assign(
  target,
  {bar: 2},
  {baz: 3, bar: 4});

assert.deepEqual(
  result, { foo: 1, bar: 4, baz: 3 });
// target was modified and returned:
assert.equal(result, target);
```

The use cases for `Object.assign()` are similar to those for spread properties. In a way, it spreads destructively.

#### 25.7.2 Freezing objects



`Object.freeze(obj)` makes `obj` completely immutable: You can’t change properties, add properties, or change its prototype – for example:

```js
const frozen = Object.freeze({ x: 2, y: 5 });
assert.throws(
  () => { frozen.x = 7 },
  {
    name: 'TypeError',
    message: /^Cannot assign to read only property 'x'/,
  });
```

There is one caveat: `Object.freeze(obj)` freezes shallowly. That is, only the properties of `obj` are frozen but not objects stored in properties.

#### 25.7.3 Property attributes and property descriptors



Just as objects are composed of properties, properties are composed of *attributes*. The value of a property is only one of several attributes. Others include:

- `writable`: Is it possible to change the value of the property?
- `enumerable`: Is the property considered by `Object.keys()`, spreading, etc.?

When you are using one of the operations for handling property attributes, attributes are specified via *property descriptors*: objects where each property represents one attribute. For example, this is how you read the attributes of a property `obj.foo`:

```js
const obj = { foo: 123 };
assert.deepEqual(
  Object.getOwnPropertyDescriptor(obj, 'foo'),
  {
    value: 123,
    writable: true,
    enumerable: true,
    configurable: true,
  });
```

And this is how you set the attributes of a property `obj.bar`:

```js
const obj = {
  foo: 1,
  bar: 2,
};

assert.deepEqual(Object.keys(obj), ['foo', 'bar']);

// Hide property `bar` from Object.keys()
Object.defineProperty(obj, 'bar', {
  enumerable: false,
});

assert.deepEqual(Object.keys(obj), ['foo']);
```

Enumerability is covered in greater detail earlier in this chapter. For more information on property attributes and property descriptors, consult *Speaking JavaScript*.