## 27 Synchronous iteration

### 27.1 What is synchronous iteration about?

Synchronous iteration is a *protocol* (interfaces plus rules for using them) that connects two groups of entities in JavaScript:

- **Data sources:** On one hand, data comes in all shapes and sizes. In JavaScript’s standard library, you have the linear data structure `Array`, the ordered collection `Set` (elements are ordered by time of addition), the ordered dictionary `Map` (entries are ordered by time of addition), and more. In libraries, you may find tree-shaped data structures and more.
- **Data consumers:** On the other hand, you have a whole class of constructs and algorithms that only need to access their input *sequentially*: one value at a time, until all values were visited. Examples include the `for-of` loop and spreading into function calls (via `...`).

The iteration protocol connects these two groups via the interface `Iterable`: data sources deliver their contents sequentially “through it”; data consumers get their input via it.

```
Data Consumers   Interface  Data sources
for-of loop                 Arrays
                 Iterable   Maps
spreading                   Strings
```

Figure 18: Data consumers such as the `for-of` loop use the interface `Iterable`. Data sources such as `Arrays` implement that interface.

Fig.18 illustrates how iteration works: data consumers use the interface `Iterable`; data sources implement it.

> **The JavaScript way of implementing interfaces**
>
> In JavaScript, an object *implements* an interface if it has all the methods that it describes. The interfaces mentioned in this chapter only exist in the ECMAScript specification.

Both sources and consumers of data profit from this arrangement:

- If you develop a new data structure, you only need to implement `Iterable` and a raft of tools can immediately be applied to it.
- If you write code that uses iteration, it automatically works with many sources of data.

### 27.2 Core iteration constructs: iterables and iterators

Two roles (described by interfaces) form the core of iteration (fig.19):

- An *iterable* is an object whose contents can be traversed sequentially.
- An *iterator* is the pointer used for the traversal.

```diff
-Iterable: 
traversable data structure
{
+[Symbol.iterator]()         -----+       
}                                 |
                                  |returns
-Iterator:                        |
pointer for traversing iterable   |
+next()         <-----------------+
```

Figure 19: Iteration has two main interfaces: `Iterable` and `Iterator`. The former has a method that returns the latter.

These are type definitions (in TypeScript’s notation) for the interfaces of the iteration protocol:

```ts
interface Iterable<T> {
  [Symbol.iterator]() : Iterator<T>;
}

interface Iterator<T> {
  next() : IteratorResult<T>;
}

interface IteratorResult<T> {
  value: T;
  done: boolean;
}
```

The interfaces are used as follows:

- You ask an `Iterable` for an iterator via the method whose key is `Symbol.iterator`.
- The `Iterator` returns the iterated values via its method `.next()`.
- The values are not returned directly, but wrapped in objects with two properties:
  - `.value` is the iterated value.
  - `.done` indicates if the end of the iteration has been reached yet. It is `true` after the last iterated value and `false` beforehand.

### 27.3 Iterating manually

This is an example of using the iteration protocol:

```js
const iterable = ['a', 'b'];

// The iterable is a factory for iterators:
const iterator = iterable[Symbol.iterator]();

// Call .next() until .done is true:
expect(iterator.next()).toEqual({ value: 'a', done: false })
expect(iterator.next()).toEqual({ value: 'b', done: false })
expect(iterator.next()).toEqual({ value: undefined, done: true })
```

#### 27.3.1 Iterating over an iterable via `while`

The following code demonstrates how to use a `while` loop to iterate over an iterable:

```js
function logAll(iterable) {
  const iterator = iterable[Symbol.iterator]();
  while (true) {
    const {value, done} = iterator.next();
    if (done) break;
    console.log(value);
  }
}

logAll(['a', 'b']);
// Output:
// 'a'
// 'b'
```


### 27.4 Iteration in practice

We have seen how to use the iteration protocol manually, and it is relatively cumbersome. But the protocol is not meant to be used directly — it is meant to be used via higher-level language constructs built on top of it. This section shows what that looks like.

#### 27.4.1 Iterating over Arrays

JavaScript’s Arrays are iterable. That enables us to use the `for-of` loop:

```js
const myArray = ['a', 'b', 'c'];

for (const x of myArray) {
  console.log(x);
}
// Output:
// 'a'
// 'b'
// 'c'
```

Destructuring via Array patterns (explained later) also uses iteration under the hood:

```js
const [first, second] = myArray;
assert.equal(first, 'a');
assert.equal(second, 'b');
```

#### 27.4.2 Iterating over Sets

JavaScript’s Set data structure is iterable. That means `for-of` works:

```js
const mySet = new Set().add('a').add('b').add('c');

for (const x of mySet) {
  console.log(x);
}
// Output:
// 'a'
// 'b'
// 'c'
```

As does Array-destructuring:

```js
const [first, second] = mySet;
assert.equal(first, 'a');
assert.equal(second, 'b');
```

### 27.5 Quick reference: synchronous iteration

#### 27.5.1 Iterable data sources

The following built-in data sources are iterable:

- Arrays
- Strings
- Maps
- Sets
- (Browsers: DOM data structures)

To iterate over the properties of objects, you need helpers such as `Object.keys()` and `Object.entries()`. That is necessary because properties exist at a different level that is independent of the level of data structures.

#### 27.5.2 Iterating constructs

The following constructs are based on iteration:

- Destructuring via an Array pattern:

  ```js
  const [x,y] = iterable;
  ```

- The `for-of` loop:

  ```js
  for (const x of iterable) { /*···*/ }
  ```

- `Array.from()`:

  ```js
  const arr = Array.from(iterable);
  ```

- Spreading (via `...`) into function calls and Array literals:

  ```js
  func(...iterable);
  const arr = [...iterable];
  ```

- `new Map()` and `new Set()`:

  ```js
  const m = new Map(iterableOverKeyValuePairs);
  const s = new Set(iterableOverElements);
  ```

- `Promise.all()` and `Promise.race()`:

  ```js
  const promise1 = Promise.all(iterableOverPromises);
  const promise2 = Promise.race(iterableOverPromises);
  ```

- `yield*`:

  ```js
  function* generatorFunction() {
    yield* iterable;
  }
  ```