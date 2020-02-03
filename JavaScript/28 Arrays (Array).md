## 28 Arrays (`Array`)

### 28.1 The two roles of Arrays in JavaScript

Arrays play two roles in JavaScript:

- Tuples: Arrays-as-tuples have a fixed number of indexed elements. Each of those elements can have a different type.
- Sequences: Arrays-as-sequences have a variable number of indexed elements. Each of those elements has the same type.

In practice, these two roles are often mixed.

Notably, Arrays-as-sequences are so flexible that you can use them as (traditional) arrays, stacks, and queues (see exercise later in this chapter).

### 28.2 Basic Array operations

#### 28.2.1 Creating, reading, writing Arrays

The best way to create an Array is via an *Array literal*:

```js
const arr = ['a', 'b', 'c'];
```

The Array literal starts and ends with square brackets `[]`. It creates an Array with three *elements*: `'a'`, `'b'`, and `'c'`.

To read an Array element, you put an index in square brackets (indices start at zero):

```js
assert.equal(arr[0], 'a');
```

To change an Array element, you assign to an Array with an index:

```js
arr[0] = 'x';
assert.deepEqual(arr, ['x', 'b', 'c']);
```

The range of Array indices is 32 bits (excluding the maximum length): `[0, 2**32−1)`

#### 28.2.2 The `.length` of an Array

Every Array has a property `.length` that can be used to both read and change(!) the number of elements in an Array.

The length of an Array is always the highest index plus one:

```js
> const arr = ['a', 'b'];
> arr.length
2
```

If you write to the Array at the index of the length, you append an element:

```js
> arr[arr.length] = 'c';
> arr
[ 'a', 'b', 'c' ]
> arr.length
3
```

Another way of (destructively) appending an element is via the Array method `.push()`:

```js
> arr.push('d');
> arr
[ 'a', 'b', 'c', 'd' ]
```

If you set `.length`, you are pruning the Array by removing elements:

```js
> arr.length = 1;
> arr
[ 'a' ]
```

#### 28.2.3 Clearing an Array

To clear (empty) an Array, you can either set its `.length` to zero:

```js
const arr = ['a', 'b', 'c'];
arr.length = 0;
assert.deepEqual(arr, []);
```

or you can assign a new empty Array to the variable storing the Array:

```js
let arr = ['a', 'b', 'c'];
arr = [];
assert.deepEqual(arr, []);
```

The latter approach has the advantage of not affecting other locations that point to the same Array. If, however, you do want to reset a shared Array for everyone, then you need the former approach.

#### 28.2.4 Spreading into Array literals

Inside an Array literal, a *spread element* consists of three dots (`...`) followed by an expression. It results in the expression being evaluated and then iterated over. Each iterated value becomes an additional Array element – for example:

```js
> const iterable = ['b', 'c'];
> ['a', ...iterable, 'd']
[ 'a', 'b', 'c', 'd' ]
```

That means that we can use spreading to create a copy of an Array:

```js
const original = ['a', 'b', 'c'];
const copy = [...original];
```

Spreading is also convenient for concatenating Arrays (and other iterables) into Arrays:

```js
const arr1 = ['a', 'b'];
const arr2 = ['c', 'd'];

const concatenated = [...arr1, ...arr2, 'e'];
assert.deepEqual(
  concatenated,
  ['a', 'b', 'c', 'd', 'e']);
```

Due to spreading using iteration, it only works if the value is iterable:

```js
> [...'abc'] // strings are iterable
[ 'a', 'b', 'c' ]
> [...123]
TypeError: number 123 is not iterable
> [...undefined]
TypeError: undefined is not iterable
```

**Spreading into Array literals is shallow**

Similar to [spreading into object literals](https://exploringjs.com/impatient-js/ch_single-objects.html#spreading-into-object-literals), spreading into Array literals creates *shallow* copies. That is, nested Arrays are not copied.

#### 28.2.5 Arrays: listing indices and entries

Method `.keys()` lists the indices of an Array:

```js
const arr = ['a', 'b'];
assert.deepEqual(
  [...arr.keys()], // (A)
  [0, 1]);
```

`.keys()` returns an iterable. In line A, we spread to obtain an Array.

Listing Array indices is different from listing properties. The former produces numbers; the latter produces stringified numbers (in addition to non-index property keys):

```js
const arr = ['a', 'b'];
arr.prop = true;

assert.deepEqual(
  Object.keys(arr),
  ['0', '1', 'prop']);
```

Method `.entries()` lists the contents of an Array as [index, element] pairs:

```js
const arr = ['a', 'b'];
assert.deepEqual(
  [...arr.entries()],
  [[0, 'a'], [1, 'b']]);
```

#### 28.2.6 Is a value an Array?

Following are two ways of checking if a value is an Array:

```js
> [] instanceof Array
true
> Array.isArray([])
true
```

`instanceof` is usually fine. You need `Array.isArray()` if a value may come from another *realm*. Roughly, a realm is an instance of JavaScript’s global scope. Some realms are isolated from each other (e.g., [Web Workers](https://exploringjs.com/impatient-js/ch_async-js.html#web-workers) in browsers), but there are also realms between which you can move data – for example, same-origin iframes in browsers. `x instanceof Array` checks the prototype chain of `x` and therefore returns `false` if `x` is an Array from another realm.

`typeof` categorizes Arrays as objects:

```js
> typeof []
'object'
```

### 28.3 `for-of` and Arrays

We have already encountered the `for-of` loop. This section briefly recaps how to use it for Arrays.

#### 28.3.1 `for-of`: iterating over elements

The following `for-of` loop iterates over the elements of an Array.

```js
for (const element of ['a', 'b']) {
  console.log(element);
}
// Output:
// 'a'
// 'b'
```

#### 28.3.2 `for-of`: iterating over [index, element] pairs

The following `for-of` loop iterates over [index, element] pairs. Destructuring (described later), gives us convenient syntax for setting up `index` and `element` in the head of `for-of`.

```js
for (const [index, element] of ['a', 'b'].entries()) {
  console.log(index, element);
}
// Output:
// 0, 'a'
// 1, 'b'
```

### 28.4 Array-like objects

Some operations that work with Arrays require only the bare minimum: values must only be *Array-like*. An Array-like value is an object with the following properties:

- `.length`: holds the length of the Array-like object.
- `[0]`: holds the element at index 0 (etc.). Note that if you use numbers as property names, they are always coerced to strings. Therefore, `[0]` retrieves the value of the property whose key is `'0'`.

For example, `Array.from()` accepts Array-like objects and converts them to Arrays:

```js
// If you omit .length, it is interpreted as 0
assert.deepEqual(
  Array.from({}),
  []);

assert.deepEqual(
  Array.from({length:2, 0:'a', 1:'b'}),
  [ 'a', 'b' ]);
```

The TypeScript interface for Array-like objects is:

```
interface ArrayLike<T> {
  length: number;
  [n: number]: T;
}
```

 **Array-like objects are relatively rare in modern JavaScript**

Array-like objects used to be common before ES6; now you don’t see them very often.

### 28.5 Converting iterable and Array-like values to Arrays

There are two common ways of converting iterable and Array-like values to Arrays: spreading and `Array.from()`.

#### 28.5.1 Converting iterables to Arrays via spreading (`...`)

Inside an Array literal, spreading via `...` converts any iterable object into a series of Array elements. For example:

```js
// Get an Array-like collection from a web browser’s DOM
const domCollection = document.querySelectorAll('a');

// Alas, the collection is missing many Array methods
assert.equal('map' in domCollection, false);

// Solution: convert it to an Array
const arr = [...domCollection];
assert.deepEqual(
  arr.map(x => x.href),
  ['https://2ality.com', 'https://exploringjs.com']);
```

The conversion works because the DOM collection is iterable.

#### 28.5.2 Converting iterables and Array-like objects to Arrays via `Array.from()` (advanced)

`Array.from()` can be used in two modes.

##### 28.5.2.1 Mode 1 of `Array.from()`: converting

The first mode has the following type signature:

```ts
.from<T>(iterable: Iterable<T> | ArrayLike<T>): T[]
```

Interface `Iterable` is shown in the chapter on synchronous iteration. Interface `ArrayLike` appeared earlier in this chapter.

With a single parameter, `Array.from()` converts anything iterable or Array-like to an Array:

```js
> Array.from(new Set(['a', 'b']))
[ 'a', 'b' ]
> Array.from({length: 2, 0:'a', 1:'b'})
[ 'a', 'b' ]
```

##### 28.5.2.2 Mode 2 of `Array.from()`: converting and mapping

The second mode of `Array.from()` involves two parameters:

```ts
.from<T, U>(
  iterable: Iterable<T> | ArrayLike<T>,
  mapFunc: (v: T, i: number) => U,
  thisArg?: any)
  : U[]
```

In this mode, `Array.from()` does several things:

- It iterates over `iterable`.
- It calls `mapFunc` with each iterated value. The optional parameter `thisArg` specifies a `this` for `mapFunc`.
- It applies `mapFunc` to each iterated value.
- It collects the results in a new Array and returns it.

In other words: we are going from an iterable with elements of type `T` to an Array with elements of type `U`.

This is an example:

```js
> Array.from(new Set(['a', 'b']), x => x + x)
[ 'aa', 'bb' ]
```

### 28.6 Creating and filling Arrays with arbitrary lengths

The best way of creating an Array is via an Array literal. However, you can’t always use one: The Array may be too large, you may not know its length during development, or you may want to keep its length flexible. Then I recommend the following techniques for creating, and possibly filling, Arrays.

#### 28.6.1 Do you need to create an empty Array that you’ll fill completely later on?

```js
> new Array(3)
[ , , ,]
```

Note that the result has three *holes* (empty slots) – the last comma in an Array literal is always ignored.

#### 28.6.2 Do you need to create an Array filled with a primitive value?

```js
> new Array(3).fill(0)
[0, 0, 0]
```

Caveat: If you use `.fill()` with an object, then each Array element will refer to this object (sharing it).

```js
const arr = new Array(3).fill({});
arr[0].prop = true;
assert.deepEqual(
  arr, [
    {prop: true},
    {prop: true},
    {prop: true},
  ]);
```

The next subsection explains how to fix this.

#### 28.6.3 Do you need to create an Array filled with objects?

```js
> Array.from({length: 3}, () => ({}))
[{}, {}, {}]
```

#### 28.6.4 Do you need to create a range of integers?

```js
function createRange(start, end) {
  return Array.from({length: end - start}, (_, i) => i+start);
}
assert.deepEqual(
  createRange(2, 5),
  [2, 3, 4]);
```

Here is an alternative, slightly hacky technique for creating integer ranges that start at zero:

```js
/** Returns an iterable */
function createRange(end) {
  return new Array(end).keys();
}
assert.deepEqual(
  [...createRange(4)],
  [0, 1, 2, 3]);
```

This works because `.keys()` treats *holes* like `undefined` elements and lists their indices.

#### 28.6.5 Use a Typed Array if the elements are all integers or all floats

If you are dealing with Arrays of integers or floats, consider *Typed Arrays*, which were created for this purpose.



### 28.7 Multidimensional Arrays

JavaScript does not have real multidimensional Arrays; you need to resort to Arrays whose elements are Arrays:

```js
function initMultiArray(...dimensions) {
  function initMultiArrayRec(dimIndex) {
    if (dimIndex >= dimensions.length) {
      return 0;
    } else {
      const dim = dimensions[dimIndex];
      const arr = [];
      for (let i=0; i<dim; i++) {
        arr.push(initMultiArrayRec(dimIndex+1));
      }
      return arr;
    }
  }
  return initMultiArrayRec(0);
}

const arr = initMultiArray(4, 3, 2);
arr[3][2][1] = 'X'; // last in each dimension
assert.deepEqual(arr, [
  [ [ 0, 0 ], [ 0, 0 ], [ 0, 0 ] ],
  [ [ 0, 0 ], [ 0, 0 ], [ 0, 0 ] ],
  [ [ 0, 0 ], [ 0, 0 ], [ 0, 0 ] ],
  [ [ 0, 0 ], [ 0, 0 ], [ 0, 'X' ] ],
]);
```

### 28.8 More Array features (advanced)

In this section, we look at phenomena you don’t encounter often when working with Arrays.

#### 28.8.1 Array indices are (slightly special) property keys

You’d think that Array elements are special because you are accessing them via numbers. But the square brackets operator `[]` for doing so is the same operator that is used for accessing properties. It coerces any value (that is not a symbol) to a string. Therefore, Array elements are (almost) normal properties (line A) and it doesn’t matter if you use numbers or strings as indices (lines B and C):

```js
const arr = ['a', 'b'];
arr.prop = 123;
assert.deepEqual(
  Object.keys(arr),
  ['0', '1', 'prop']); // (A)

assert.equal(arr[0], 'a');  // (B)
assert.equal(arr['0'], 'a'); // (C)
```

To make matters even more confusing, this is only how the language specification defines things (the theory of JavaScript, if you will). Most JavaScript engines optimize under the hood and do use actual integers to access Array elements (the practice of JavaScript, if you will).

Property keys (strings!) that are used for Array elements are called *indices*. A string `str` is an index if converting it to a 32-bit unsigned integer and back results in the original value. Written as a formula:

```js
ToString(ToUint32(str)) === str
```

##### 28.8.1.1 Listing indices

When listing property keys, indices are treated specially – they always come first and are sorted like numbers (`'2'` comes before `'10'`):

```js
const arr = [];
arr.prop = true;
arr[1] = 'b';
arr[0] = 'a';

assert.deepEqual(
  Object.keys(arr),
  ['0', '1', 'prop']);
```

Note that `.length`, `.entries()` and `.keys()` treat Array indices as numbers and ignore non-index properties:

```js
assert.equal(arr.length, 2);
assert.deepEqual(
  [...arr.keys()], [0, 1]);
assert.deepEqual(
  [...arr.entries()], [[0, 'a'], [1, 'b']]);
```

We used a spread element (`...`) to convert the iterables returned by `.keys()` and `.entries()` to Arrays.

#### 28.8.2 Arrays are dictionaries and can have holes

We distinguish two kinds of Arrays in JavaScript:

- An Array `arr` is *dense* if all indices `i`, with 0 ≤ `i` < `arr.length`, exist. That is, the indices form a contiguous range.
- An Array is *sparse* if the range of indices has *holes* in it. That is, some indices are missing.

Arrays can be sparse in JavaScript because Arrays are actually dictionaries from indices to values.

**Recommendation: avoid holes**

So far, we have only seen dense Arrays and it’s indeed recommended to avoid holes: They make your code more complicated and are not handled consistently by Array methods. Additionally, JavaScript engines optimize dense Arrays, making them faster.

##### 28.8.2.1 Creating holes

You can create holes by skipping indices when assigning elements:

```js
const arr = [];
arr[0] = 'a';
arr[2] = 'c';

assert.deepEqual(Object.keys(arr), ['0', '2']); // (A)

assert.equal(0 in arr, true); // element
assert.equal(1 in arr, false); // hole
```

In line A, we are using `Object.keys()` because `arr.keys()` treats holes as if they were `undefined` elements and does not reveal them.

Another way of creating holes is to skip elements in Array literals:

```js
const arr = ['a', , 'c'];
assert.deepEqual(Object.keys(arr), ['0', '2']);
```

You can also delete Array elements:

```js
const arr = ['a', 'b', 'c'];
assert.deepEqual(Object.keys(arr), ['0', '1', '2']);
delete arr[1];
assert.deepEqual(Object.keys(arr), ['0', '2']);
```

##### 28.8.2.2 How do Array operations treat holes?

Alas, there are many different ways in which Array operations treat holes.

Some Array operations remove holes:

```js
> ['a',,'b'].filter(x => true)
[ 'a', 'b' ]
```

Some Array operations ignore holes:

```js
> ['a', ,'a'].every(x => x === 'a')
true
```

Some Array operations ignore but preserve holes:

```js
> ['a',,'b'].map(x => 'c')
[ 'c', , 'c' ]
```

Some Array operations treat holes as `undefined` elements:

```js
> Array.from(['a',,'b'], x => x)
[ 'a', undefined, 'b' ]
> [...['a',,'b'].entries()]
[[0, 'a'], [1, undefined], [2, 'b']]
```

`Object.keys()` works differently than `.keys()` (strings vs. numbers, holes don’t have keys):

```js
> [...['a',,'b'].keys()]
[ 0, 1, 2 ]
> Object.keys(['a',,'b'])
[ '0', '2' ]
```

There is no rule to remember here. If it ever matters how an Array operation treats holes, the best approach is to do a quick test in a console.

### 28.9 Adding and removing elements (destructively and non-destructively)

JavaScript’s `Array` is quite flexible and more like a combination of array, stack, and queue. This section explores ways of adding and removing Array elements. Most operations can be performed both *destructively* (modifying the Array) and *non-destructively* (producing a modified copy).

#### 28.9.1 Prepending elements and Arrays

In the following code, we destructively prepend single elements to `arr1` and an Array to `arr2`:

```js
const arr1 = ['a', 'b'];
arr1.unshift('x', 'y'); // prepend single elements
assert.deepEqual(arr1, ['x', 'y', 'a', 'b']);

const arr2 = ['a', 'b'];
arr2.unshift(...['x', 'y']); // prepend Array
assert.deepEqual(arr2, ['x', 'y', 'a', 'b']);
```

Spreading lets us unshift an Array into `arr2`.

Non-destructive prepending is done via spread elements:

```js
const arr1 = ['a', 'b'];
assert.deepEqual(
  ['x', 'y', ...arr1], // prepend single elements
  ['x', 'y', 'a', 'b']);
assert.deepEqual(arr1, ['a', 'b']); // unchanged!

const arr2 = ['a', 'b'];
assert.deepEqual(
  [...['x', 'y'], ...arr2], // prepend Array
  ['x', 'y', 'a', 'b']);
assert.deepEqual(arr2, ['a', 'b']); // unchanged!
```

#### 28.9.2 Appending elements and Arrays

In the following code, we destructively append single elements to `arr1` and an Array to `arr2`:

```js
const arr1 = ['a', 'b'];
arr1.push('x', 'y'); // append single elements
assert.deepEqual(arr1, ['a', 'b', 'x', 'y']);

const arr2 = ['a', 'b'];
arr2.push(...['x', 'y']); // append Array
assert.deepEqual(arr2, ['a', 'b', 'x', 'y']);
```

Spreading lets us push an Array into `arr2`.

Non-destructive appending is done via spread elements:

```js
const arr1 = ['a', 'b'];
assert.deepEqual(
  [...arr1, 'x', 'y'], // append single elements
  ['a', 'b', 'x', 'y']);
assert.deepEqual(arr1, ['a', 'b']); // unchanged!

const arr2 = ['a', 'b'];
assert.deepEqual(
  [...arr2, ...['x', 'y']], // append Array
  ['a', 'b', 'x', 'y']);
assert.deepEqual(arr2, ['a', 'b']); // unchanged!
```

#### 28.9.3 Removing elements

These are three destructive ways of removing Array elements:

```js
// Destructively remove first element:
const arr1 = ['a', 'b', 'c'];
assert.equal(arr1.shift(), 'a');
assert.deepEqual(arr1, ['b', 'c']);

// Destructively remove last element:
const arr2 = ['a', 'b', 'c'];
assert.equal(arr2.pop(), 'c');
assert.deepEqual(arr2, ['a', 'b']);

// Remove one or more elements anywhere:
const arr3 = ['a', 'b', 'c', 'd'];
assert.deepEqual(arr3.splice(1, 2), ['b', 'c']); //從索引1開始，包括索引1在内的，2個元素。
assert.deepEqual(arr3, ['a', 'd']);
```

`.splice()` is covered in more detail in the quick reference at the end of this chapter.

Destructuring via a rest element lets you non-destructively remove elements from the beginning of an Array.

```js
const arr1 = ['a', 'b', 'c'];
// Ignore first element, extract remaining elements
const [, ...arr2] = arr1;js

assert.deepEqual(arr2, ['b', 'c']);
assert.deepEqual(arr1, ['a', 'b', 'c']); // unchanged!
```

Alas, a rest element must come last in an Array. Therefore, you can only use it to extract suffixes.


### 28.10 Methods: iteration and transformation (`.find()`, `.map()`, `.filter()`, etc.)

In this section, we take a look at Array methods for iterating over Arrays and for transforming Arrays.

#### 28.10.1 Callbacks for iteration and transformation methods

All iteration and transformation methods use callbacks. The former feed all iterated values to their callbacks; the latter ask their callbacks how to transform Arrays.

These callbacks have type signatures that look as follows:

```js
callback: (value: T, index: number, array: Array<T>) => boolean
```

That is, the callback gets three parameters (it is free to ignore any of them):

- `value` is the most important one. This parameter holds the iterated value that is currently being processed.
- `index` can additionally tell the callback what the index of the iterated value is.
- `array` points to the current Array (the receiver of the method call). Some algorithms need to refer to the whole Array – e.g., to search it for answers. This parameter lets you write reusable callbacks for such algorithms.

What the callback is expected to return depends on the method it is passed to. Possibilities include:

- `.map()` fills its result with the values returned by its callback:

  ```js
  > ['a', 'b', 'c'].map(x => x + x)
  [ 'aa', 'bb', 'cc' ]
  ```

- `.find()` returns the first Array element for which its callback returns `true`:

  ```js
  > ['a', 'bb', 'ccc'].find(str => str.length >= 2)
  'bb'
  ```

Both of these methods are described in more detail later.

#### 28.10.2 Searching elements: `.find()`, `.findIndex()`

`.find()` returns the first element for which its callback returns a truthy value (and `undefined` if it can’t find anything):

```js
> [6, -5, 8].find(x => x < 0)
-5
> [6, 5, 8].find(x => x < 0)
undefined
```

`.findIndex()` returns the index of the first element for which its callback returns a truthy value (and `-1` if it can’t find anything):

```js
> [6, -5, 8].findIndex(x => x < 0)
1
> [6, 5, 8].findIndex(x => x < 0)
-1
```

`.findIndex()` can be implemented as follows:

```js
function findIndex(arr, callback) {
  for (const [i, x] of arr.entries()) {
    if (callback(x, i, arr)) {
      return i;
    }
  }
  return -1;
}
```

#### 28.10.3 `.map()`: copy while giving elements new values

`.map()` returns a modified copy of the receiver. The elements of the copy are the results of applying `map`’s callback to the elements of the receiver.

All of this is easier to understand via examples:

```js
> [1, 2, 3].map(x => x * 3)
[ 3, 6, 9 ]
> ['how', 'are', 'you'].map(str => str.toUpperCase())
[ 'HOW', 'ARE', 'YOU' ]
> [true, true, true].map((_x, index) => index)
[ 0, 1, 2 ]
```

`.map()` can be implemented as follows:

```js
function map(arr, mapFunc) {
  const result = [];
  for (const [i, x] of arr.entries()) {
    result.push(mapFunc(x, i, arr));
  }
  return result;
}
```


#### 28.10.4 `.flatMap()`: mapping to zero or more values

The type signature of `Array.prototype.flatMap()` is:

```
.flatMap<U>(
  callback: (value: T, index: number, array: T[]) => U|Array<U>,
  thisValue?: any
): U[]
```

Both `.map()` and `.flatMap()` take a function `callback` as a parameter that controls how an input Array is translated to an output Array:

- With `.map()`, each input Array element is translated to exactly one output element. That is, `callback` returns a single value.
- With `.flatMap()`, each input Array element is translated to zero or more output elements. That is, `callback` returns an Array of values (it can also return non-Array values, but that is rare).

This is `.flatMap()` in action:

```js
> ['a', 'b', 'c'].flatMap(x => [x,x])
[ 'a', 'a', 'b', 'b', 'c', 'c' ]
> ['a', 'b', 'c'].flatMap(x => [x])
[ 'a', 'b', 'c' ]
> ['a', 'b', 'c'].flatMap(x => [])
[]
```

##### 28.10.4.1 A simple implementation

You could implement `.flatMap()` as follows. Note: This implementation is simpler than the built-in version, which, for example, performs more checks.

```js
function flatMap(arr, mapFunc) {
  const result = [];
  for (const [index, elem] of arr.entries()) {
    const x = mapFunc(elem, index, arr);
    // We allow mapFunc() to return non-Arrays
    if (Array.isArray(x)) {
      result.push(...x);
    } else {
      result.push(x);
    }
  }
  return result;
}
```

What is `.flatMap()` good for? Let’s look at use cases!

##### 28.10.4.2 Use case: filtering and mapping at the same time

The result of the Array method `.map()` always has the same length as the Array it is invoked on. That is, its callback can’t skip Array elements it isn’t interested in. The ability of `.flatMap()` to do so is useful in the next example.

We will use the following function `processArray()` to create an Array that we’ll then filter and map via `.flatMap()`:

```js
function processArray(arr, callback) {
  return arr.map(x => {
    try {
      return { value: callback(x) };
    } catch (e) {
      return { error: e };
    }
  });
}
```

Next, we create an Array `results` via `processArray()`:

```js
const results = processArray([1, -5, 6], throwIfNegative);
assert.deepEqual(results, [
  { value: 1 },
  { error: new Error('Illegal value: -5') },
  { value: 6 },
]);

function throwIfNegative(value) {
  if (value < 0) {
    throw new Error('Illegal value: '+value);
  }
  return value;
}
```

We can now use `.flatMap()` to extract just the values or just the errors from `results`:

```js
const values = results.flatMap(
  result => result.value ? [result.value] : []);
assert.deepEqual(values, [1, 6]);
  
const errors = results.flatMap(
  result => result.error ? [result.error] : []);
assert.deepEqual(errors, [new Error('Illegal value: -5')]);
```

##### 28.10.4.3 Use case: mapping to multiple values

The Array method `.map()` maps each input Array element to one output element. But what if we want to map it to multiple output elements?

That becomes necessary in the following example:

```js
> stringsToCodePoints(['many', 'a', 'moon'])
['m', 'a', 'n', 'y', 'a', 'm', 'o', 'o', 'n']
```

We want to convert an Array of strings to an Array of Unicode characters (code points). The following function achieves that via `.flatMap()`:

```js
function stringsToCodePoints(strs) {
  return strs.flatMap(str => [...str]);
}
```


#### 28.10.5 `.filter()`: only keep some of the elements

The Array method `.filter()` returns an Array collecting all elements for which the callback returns a truthy value.

For example:

```js
> [-1, 2, 5, -7, 6].filter(x => x >= 0)
[ 2, 5, 6 ]
> ['a', 'b', 'c', 'd'].filter((_x,i) => (i%2)===0)
[ 'a', 'c' ]
```

`.filter()` can be implemented as follows:

```js
function filter(arr, filterFunc) {
  const result = [];
  for (const [i, x] of arr.entries()) {
    if (filterFunc(x, i, arr)) {
      result.push(x);
    }
  }
  return result;
}
```

#### 28.10.6 `.reduce()`: deriving a value from an Array (advanced)

Method `.reduce()` is a powerful tool for computing a “summary” of an Array `arr`. A summary can be any kind of value:

- A number. For example, the sum of all elements of `arr`.
- An Array. For example, a copy of `arr`, where each element is twice the original element.
- Etc.

`reduce` is also known as `foldl` (“fold left”) in functional programming and popular there. One caveat is that it can make code difficult to understand.

`.reduce()` has the following type signature (inside an `Array`):

```ts
.reduce<U>(
  callback: (accumulator: U, element: T, index: number, array: T[]) => U,
  init?: U)
  : U
```

`T` is the type of the Array elements, `U` is the type of the summary. The two may or may not be different. `accumulator` is just another name for “summary”.

To compute the summary of an Array `arr`, `.reduce()` feeds all Array elements to its callback one at a time:

```js
const accumulator_0 = callback(init, arr[0]);
const accumulator_1 = callback(accumulator_0, arr[1]);
const accumulator_2 = callback(accumulator_1, arr[2]);
// Etc.
```

`callback` combines the previously computed summary (stored in its parameter `accumulator`) with the current Array element and returns the next `accumulator`. The result of `.reduce()` is the final accumulator – the last result of `callback` after it has visited all elements.

In other words: `callback` does most of the work; `.reduce()` just invokes it in a useful manner.

You could say that the callback folds Array elements into the accumulator. That’s why this operation is called “fold” in functional programming.

##### 28.10.6.1 A first example

Let’s look at an example of `.reduce()` in action: function `addAll()` computes the sum of all numbers in an Array `arr`.

```js
function addAll(arr) {
  const startSum = 0;
  const callback = (sum, element) => sum + element;
  return arr.reduce(callback, startSum);
}
assert.equal(addAll([1,  2, 3]), 6); // (A)
assert.equal(addAll([7, -4, 2]), 5);
```

In this case, the accumulator holds the sum of all Array elements that `callback` has already visited.

How was the result `6` derived from the Array in line A? Via the following invocations of `callback`:

```js
callback(0, 1) --> 1
callback(1, 2) --> 3
callback(3, 3) --> 6
```

Notes:

- The first parameters are the current accumulators (starting with parameter `init` of `.reduce()`).
- The second parameters are the current Array elements.
- The results are the next accumulators.
- The last result of `callback` is also the result of `.reduce()`.

Alternatively, we could have implemented `addAll()` via a `for-of` loop:

```js
function addAll(arr) {
  let sum = 0;
  for (const element of arr) {
    sum = sum + element;
  }
  return sum;
}
```

It’s hard to say which of the two implementations is “better”: the one based on `.reduce()` is a little more concise, while the one based on `for-of` may be a little easier to understand – especially if you are not familiar with functional programming.

##### 28.10.6.2 Example: finding indices via `.reduce()`

The following function is an implementation of the Array method `.indexOf()`. It returns the first index at which the given `searchValue` appears inside the Array `arr`:

```js
const NOT_FOUND = -1;
function indexOf(arr, searchValue) {
  return arr.reduce(
    (result, elem, index) => {
      if (result !== NOT_FOUND) {
        // We have already found something: don’t change anything
        return result;
      } else if (elem === searchValue) {
        return index;
      } else {
        return NOT_FOUND;
      }
    },
    NOT_FOUND);
}
assert.equal(indexOf(['a', 'b', 'c'], 'b'), 1);
assert.equal(indexOf(['a', 'b', 'c'], 'x'), -1);
```

One limitation of `.reduce()` is that you can’t finish early (in a `for-of` loop, you can `break`). Here, we always immediately return the result once we have found it.

##### 28.10.6.3 Example: doubling Array elements

Function `double(arr)` returns a copy of `inArr` whose elements are all multiplied by 2:

```js
function double(inArr) {
  return inArr.reduce(
    (outArr, element) => {
      outArr.push(element * 2);
      return outArr;
    },
    []);
}
assert.deepEqual(
  double([1, 2, 3]),
  [2, 4, 6]);
```

We modify the initial value `[]` by pushing into it. A non-destructive, more functional version of `double()` looks as follows:

```js
function double(inArr) {
  return inArr.reduce(
    // Don’t change `outArr`, return a fresh Array
    (outArr, element) => [...outArr, element * 2],
    []);
}
assert.deepEqual(
  double([1, 2, 3]),
  [2, 4, 6]);
```

This version is more elegant but also slower and uses more memory.



### 28.11 `.sort()`: sorting Arrays

`.sort()` has the following type definition:

```ts
sort(compareFunc?: (a: T, b: T) => number): this
```

By default, `.sort()` sorts string representations of the elements. These representations are compared via `<`. This operator compares *lexicographically* (the first characters are most significant). You can see that when sorting numbers:

```js
> [200, 3, 10].sort()
[ 10, 200, 3 ]
```

When sorting human-language strings, you need to be aware that they are compared according to their code unit values (char codes):

```js
> ['pie', 'cookie', 'éclair', 'Pie', 'Cookie', 'Éclair'].sort()
[ 'Cookie', 'Pie', 'cookie', 'pie', 'Éclair', 'éclair' ]
```

As you can see, all unaccented uppercase letters come before all unaccented lowercase letters, which come before all accented letters. Use `Intl`, the JavaScript internationalization API, if you want proper sorting for human languages.

Note that `.sort()` sorts *in place*; it changes and returns its receiver:

```js
> const arr = ['a', 'c', 'b'];
> arr.sort() === arr
true
> arr
[ 'a', 'b', 'c' ]
```

#### 28.11.1 Customizing the sort order

You can customize the sort order via the parameter `compareFunc`, which must return a number that is:

- negative if `a < b`
- zero if `a === b`
- positive if `a > b`

**Tip for remembering these rules**

A negative number is *less than* zero (etc.).

#### 28.11.2 Sorting numbers

You can use this helper function to sort numbers:

```js
function compareNumbers(a, b) {
  if (a < b) {
    return -1;
  } else if (a === b) {
    return 0;
  } else {
    return 1;
  }
}
assert.deepEqual(
  [200, 3, 10].sort(compareNumbers),
  [3, 10, 200]);
```

The following is a quick and dirty alternative.

```js
> [200, 3, 10].sort((a,b) => a - b)
[ 3, 10, 200 ]
```

The downsides of this approach are:

- It is cryptic.
- There is a risk of numeric overflow or underflow, if `a-b` becomes a large positive or negative number.

#### 28.11.3 Sorting objects

You also need to use a compare function if you want to sort objects. As an example, the following code shows how to sort objects by age.

```js
const arr = [ {age: 200}, {age: 3}, {age: 10} ];
assert.deepEqual(
  arr.sort((obj1, obj2) => obj1.age - obj2.age),
  [{ age: 3 }, { age: 10 }, { age: 200 }] );
```



### 28.12 Quick reference: `Array`

Legend:

- `R`: method does not change the Array (non-destructive).
- `W`: method changes the Array (destructive).

#### 28.12.1 `new Array()`

`new Array(n)` creates an Array of length `n` that contains `n` holes:

```
// Trailing commas are always ignored.
// Therefore: number of commas = number of holes
assert.deepEqual(new Array(3), [,,,]);
```

`new Array()` creates an empty Array. However, I recommend to always use `[]` instead.

#### 28.12.2 Static methods of `Array`

- `Array.from(iterable: Iterable | ArrayLike): T[]` [ES6]

- `Array.from(iterable: Iterable | ArrayLike, mapFunc: (v: T, k: number) => U, thisArg?: any): U[]` [ES6]

  Converts an iterable or [an Array-like object](https://exploringjs.com/impatient-js/ch_arrays.html#array-like-objects) to an Array. Optionally, the input values can be translated via `mapFunc` before they are added to the output Array.

  Examples:

  ```js
  > Array.from(new Set(['a', 'b'])) // iterable
  [ 'a', 'b' ]
  > Array.from({length: 2, 0:'a', 1:'b'}) // Array-like object
  [ 'a', 'b' ]
  ```

- `Array.of(...items: T[]): T[]` [ES6]

  This static method is mainly useful for subclasses of `Array`, where it serves as a custom Array literal:

  ```js
  class MyArray extends Array {}
  
  assert.equal(
    MyArray.of('a', 'b') instanceof MyArray, true);
  ```

#### 28.12.3 Methods of `Array.prototype`

- `.concat(...items: Array): T[]` [R, ES3]

  Returns a new Array that is the concatenation of the receiver and all `items`. Non-Array parameters (such as `'b'` in the following example) are treated as if they were Arrays with single elements.

  ```js
  > ['a'].concat('b', ['c', 'd'])
  [ 'a', 'b', 'c', 'd' ]
  ```

- `.copyWithin(target: number, start: number, end=this.length): this` [W, ES6]

  Copies the elements whose indices range from (including) `start` to (excluding) `end` to indices starting with `target`. Overlapping is handled correctly.

  ```js
  > ['a', 'b', 'c', 'd'].copyWithin(0, 2, 4)
  [ 'c', 'd', 'c', 'd' ]
  ```

  If `start` or `end` is negative, then `.length` is added to it.

- `.entries(): Iterable<[number, T]>` [R, ES6]

  Returns an iterable over [index, element] pairs.

  ```js
  > Array.from(['a', 'b'].entries())
  [ [ 0, 'a' ], [ 1, 'b' ] ]
  ```

- `.every(callback: (value: T, index: number, array: Array) => boolean, thisArg?: any): boolean` [R, ES5]

  Returns `true` if `callback` returns a truthy value for every element. Otherwise, it returns `false`. It stops as soon as it receives a falsy value. This method corresponds to universal quantification (“for all”, `∀`) in mathematics.

  ```js
  > [1, 2, 3].every(x => x > 0)
  true
  > [1, -2, 3].every(x => x > 0)
  false
  ```

  Related method: `.some()` (“exists”).

- `.fill(value: T, start=0, end=this.length): this` [W, ES6]

  Assigns `value` to every index between (including) `start` and (excluding) `end`.

  ```js
  > [0, 1, 2].fill('a')
  [ 'a', 'a', 'a' ]
  ```

  Caveat: Don’t use this method to fill an Array with an object `obj`; then each element will refer to `obj` (sharing it). In this case, it’s better to [use `Array.from()`](https://exploringjs.com/impatient-js/ch_arrays.html#filling-arrays).

- `.filter(callback: (value: T, index: number, array: Array) => any, thisArg?: any): T[]` [R, ES5]

  Returns an Array with only those elements for which `callback` returns a truthy value.

  ```js
  > [1, -2, 3].filter(x => x > 0)
  [ 1, 3 ]
  ```

- `.find(predicate: (value: T, index: number, obj: T[]) => boolean, thisArg?: any): T | undefined` [R, ES6]

  The result is the first element for which `predicate` returns a truthy value. If there is no such element, the result is `undefined`.

  ```js
  > [1, -2, 3].find(x => x < 0)
  -2
  > [1, 2, 3].find(x => x < 0)
  undefined
  ```

- `.findIndex(predicate: (value: T, index: number, obj: T[]) => boolean, thisArg?: any): number` [R, ES6]

  The result is the index of the first element for which `predicate` returns a truthy value. If there is no such element, the result is `-1`.

  ```js
  > [1, -2, 3].findIndex(x => x < 0)
  1
  > [1, 2, 3].findIndex(x => x < 0)
  -1
  ```

- `.flat(depth = 1): any[]` [R, ES2019]

  “Flattens” an Array: It descends into the Arrays that are nested inside the input Array and creates a copy where all values it finds at level `depth` or lower are moved to the top level.

  ```js
  > [ 1,2, [3,4], [[5,6]] ].flat(0) // no change
  [ 1, 2, [3,4], [[5,6]] ]
  
  > [ 1,2, [3,4], [[5,6]] ].flat(1)
  [1, 2, 3, 4, [5,6]]
  
  > [ 1,2, [3,4], [[5,6]] ].flat(2)
  [1, 2, 3, 4, 5, 6]
  ```

- `.flatMap(callback: (value: T, index: number, array: T[]) => U|Array, thisValue?: any): U[]` [R, ES2019]

  The result is produced by invoking `callback()` for each element of the original Array and concatenating the Arrays it returns.

  ```js
  > ['a', 'b', 'c'].flatMap(x => [x,x])
  [ 'a', 'a', 'b', 'b', 'c', 'c' ]
  > ['a', 'b', 'c'].flatMap(x => [x])
  [ 'a', 'b', 'c' ]
  > ['a', 'b', 'c'].flatMap(x => [])
  []
  ```

- `.forEach(callback: (value: T, index: number, array: Array) => void, thisArg?: any): void` [R, ES5]

  Calls `callback` for each element.

  ```js
  ['a', 'b'].forEach((x, i) => console.log(x, i))
  
  // Output:
  // 'a', 0
  // 'b', 1
  ```

  A `for-of` loop is usually a better choice: it’s faster, supports `break` and can iterate over arbitrary iterables.

- `.includes(searchElement: T, fromIndex=0): boolean` [R, ES2016]

  Returns `true` if the receiver has an element whose value is `searchElement` and `false`, otherwise. Searching starts at index `fromIndex`.

  ```js
  > [0, 1, 2].includes(1)
  true
  > [0, 1, 2].includes(5)
  false
  ```

- `.indexOf(searchElement: T, fromIndex=0): number` [R, ES5]

  Returns the index of the first element that is strictly equal to `searchElement`. Returns `-1` if there is no such element. Starts searching at index `fromIndex`, visiting higher indices next.

  ```js
  > ['a', 'b', 'a'].indexOf('a')
  0
  > ['a', 'b', 'a'].indexOf('a', 1)
  2
  > ['a', 'b', 'a'].indexOf('c')
  -1
  ```

- `.join(separator = ','): string` [R, ES1]

  Creates a string by concatenating string representations of all elements, separating them with `separator`.

  ```js
  > ['a', 'b', 'c'].join('##')
  'a##b##c'
  > ['a', 'b', 'c'].join()
  'a,b,c'
  ```

- `.keys(): Iterable` [R, ES6]

  Returns an iterable over the keys of the receiver.

  ```js
  > [...['a', 'b'].keys()]
  [ 0, 1 ]
  ```

- `.lastIndexOf(searchElement: T, fromIndex=this.length-1): number` [R, ES5]

  Returns the index of the last element that is strictly equal to `searchElement`. Returns `-1` if there is no such element. Starts searching at index `fromIndex`, visiting lower indices next.

  ```js
  > ['a', 'b', 'a'].lastIndexOf('a')
  2
  > ['a', 'b', 'a'].lastIndexOf('a', 1)
  0
  > ['a', 'b', 'a'].lastIndexOf('c')
  -1
  ```

- `.map(mapFunc: (value: T, index: number, array: Array) => U, thisArg?: any): U[]` [R, ES5]

  Returns a new Array, in which every element is the result of `mapFunc` being applied to the corresponding element of the receiver.

  ```js
  > [1, 2, 3].map(x => x * 2)
  [ 2, 4, 6 ]
  > ['a', 'b', 'c'].map((x, i) => i)
  [ 0, 1, 2 ]
  ```

- `.pop(): T | undefined` [W, ES3]

  Removes and returns the last element of the receiver. That is, it treats the end of the receiver as a stack. The opposite of `.push()`.

  ```js
  > const arr = ['a', 'b', 'c'];
  > arr.pop()
  'c'
  > arr
  [ 'a', 'b' ]
  ```

- `.push(...items: T[]): number` [W, ES3]

  Adds zero or more `items` to the end of the receiver. That is, it treats the end of the receiver as a stack. The return value is the length of the receiver after the change. The opposite of `.pop()`.

  ```js
  > const arr = ['a', 'b'];
  > arr.push('c', 'd')
  4
  > arr
  [ 'a', 'b', 'c', 'd' ]
  ```

- `.reduce(callback: (accumulator: U, element: T, index: number, array: T[]) => U, init?: U): U` [R, ES5]

  This method produces a summary of the receiver: it feeds all Array elements to `callback`, which combines a current summary (in parameter `accumulator`) with the current Array element and returns the next `accumulator`:

  ```js
  const accumulator_0 = callback(init, arr[0]);
  const accumulator_1 = callback(accumulator_0, arr[1]);
  const accumulator_2 = callback(accumulator_1, arr[2]);
  // Etc.
  ```

  The result of `.reduce()` is the last result of `callback` after it has visited all Array elements.

  ```js
  > [1, 2, 3].reduce((accu, x) => accu + x, 0)
  6
  > [1, 2, 3].reduce((accu, x) => accu + String(x), '')
  '123'
  ```

  If no `init` is provided, the Array element at index 0 is used and the element at index 1 is visited first. Therefore, the Array must have at least length 1.

- `.reduceRight(callback: (accumulator: U, element: T, index: number, array: T[]) => U, init?: U): U` [R, ES5]

  Works like `.reduce()`, but visits the Array elements backward, starting with the last element.

  ```js
  > [1, 2, 3].reduceRight((accu, x) => accu + String(x), '')
  '321'
  ```

- `.reverse(): this` [W, ES1]

  Rearranges the elements of the receiver so that they are in reverse order and then returns the receiver.

  ```js
  > const arr = ['a', 'b', 'c'];
  > arr.reverse()
  [ 'c', 'b', 'a' ]
  > arr
  [ 'c', 'b', 'a' ]
  ```

- `.shift(): T | undefined` [W, ES3]

  Removes and returns the first element of the receiver. The opposite of `.unshift()`.

  ```js
  > const arr = ['a', 'b', 'c'];
  > arr.shift()
  'a'
  > arr
  [ 'b', 'c' ]
  ```

- `.slice(start=0, end=this.length): T[]` [R, ES3]

  Returns a new Array containing the elements of the receiver whose indices are between (including) `start` and (excluding) `end`.

  ```js
  > ['a', 'b', 'c', 'd'].slice(1, 3)
  [ 'b', 'c' ]
  > ['a', 'b'].slice() // shallow copy
  [ 'a', 'b' ]
  ```

  Negative indices are allowed and added to `.length`:

  ```js
  > ['a', 'b', 'c'].slice(-2)
  [ 'b', 'c' ]
  ```

- `.some(callback: (value: T, index: number, array: Array) => boolean, thisArg?: any): boolean` [R, ES5]

  Returns `true` if `callback` returns a truthy value for at least one element. Otherwise, it returns `false`. It stops as soon as it receives a truthy value. This method corresponds to existential quantification (“exists”, `∃`) in mathematics.

  ```js
  > [1, 2, 3].some(x => x < 0)
  false
  > [1, -2, 3].some(x => x < 0)
  true
  ```

  Related method: `.every()` (“for all”).

- `.sort(compareFunc?: (a: T, b: T) => number): this` [W, ES1]

  Sorts the receiver and returns it. By default, it sorts string representations of the elements. It does so lexicographically and according to the code unit values (char codes) of the characters:

  ```js
  > ['pie', 'cookie', 'éclair', 'Pie', 'Cookie', 'Éclair'].sort()
  [ 'Cookie', 'Pie', 'cookie', 'pie', 'Éclair', 'éclair' ]
  > [200, 3, 10].sort()
  [ 10, 200, 3 ]
  ```

  You can customize the sort order via `compareFunc`, which returns a number that is:

  - negative if `a < b`
  - zero if `a === b`
  - positive if `a > b`

  Trick for sorting numbers (with a risk of numeric overflow or underflow):

  ```js
  > [200, 3, 10].sort((a, b) => a - b)
  [ 3, 10, 200 ]
  ```

  **`.sort()` is stable**

  Since ECMAScript 2019, sorting is guaranteed to be stable: if elements are considered equal by sorting, then sorting does not change the order of those elements (relative to each other).

- `.splice(start: number, deleteCount=this.length-start, ...items: T[]): T[]` [W, ES3]

  At index `start`, it removes `deleteCount` elements and inserts the `items`. It returns the deleted elements.

  ```js
  > const arr = ['a', 'b', 'c', 'd'];
  > arr.splice(1, 2, 'x', 'y')
  [ 'b', 'c' ]
  > arr
  [ 'a', 'x', 'y', 'd' ]
  ```

  `start` can be negative and is added to `.length` if it is:

  ```js
  > ['a', 'b', 'c'].splice(-2, 2)
  [ 'b', 'c' ]
  ```

- `.toString(): string` [R, ES1]

  Converts all elements to strings via `String()`, concatenates them while separating them with commas, and returns the result.

  ```js
  > [1, 2, 3].toString()
  '1,2,3'
  > ['1', '2', '3'].toString()
  '1,2,3'
  > [].toString()
  ''
  ```

- `.unshift(...items: T[]): number` [W, ES3]

  Inserts the `items` at the beginning of the receiver and returns its length after this modification.

  ```js
  > const arr = ['c', 'd'];
  > arr.unshift('e', 'f')
  4
  > arr
  [ 'e', 'f', 'c', 'd' ]
  ```

- `.values(): Iterable` [R, ES6]

  Returns an iterable over the values of the receiver.

  ```js
  > [...['a', 'b'].values()]
  [ 'a', 'b' ]
  ```