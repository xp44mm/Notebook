## 35 Synchronous generators (advanced)


### 35.1 What are synchronous generators?



Synchronous generators are special versions of function definitions and method definitions that always return synchronous iterables:

```js
// Generator function declaration
function* genFunc1() { /*···*/ }

// Generator function expression
const genFunc2 = function* () { /*···*/ };

// Generator method definition in an object literal
const obj = {
  * generatorMethod() {
    // ···
  }
};

// Generator method definition in a class definition
// (class declaration or class expression)
class MyClass {
  * generatorMethod() {
    // ···
  }
}
```

Asterisks (`*`) mark functions and methods as generators:

- Functions: The pseudo-keyword `function*` is a combination of the keyword `function` and an asterisk.
- Methods: The `*` is a modifier (similar to `static` and `get`).

#### 35.1.1 Generator functions return iterables and fill them via `yield`



If you call a generator function, it returns an iterable (actually, an iterator that is also iterable). The generator fills that iterable via the `yield` operator:

```js
function* genFunc1() {
  yield 'a';
  yield 'b';
}

const iterable = genFunc1();
// Convert the iterable to an Array, to check what’s inside:
assert.deepEqual([...iterable], ['a', 'b']);

// You can also use a for-of loop
for (const x of genFunc1()) {
  console.log(x);
}
// Output:
// 'a'
// 'b'
```

#### 35.1.2 `yield` pauses a generator function

Using a generator function involves the following steps:

- Function-calling it returns an iterator `iter` (that is also an iterable).
- Iterating over `iter` repeatedly invokes `iter.next()`. Each time, we jump into the body of the generator function until there is a `yield` that returns a value.

Therefore, `yield` does more than just add values to iterables — it also pauses and exits the generator function:

- Like `return`, a `yield` exits the body of the function and returns a value (via `.next()`).
- Unlike `return`, if you repeat the invocation (of `.next()`), execution resumes directly after the `yield`.

Let’s examine what that means via the following generator function.

```js
let location = 0;
function* genFunc2() {
  location = 1; yield 'a';
  location = 2; yield 'b';
  location = 3;
}
```

In order to use `genFunc2()`, we must first create the iterator/iterable `iter`. `genFunc2()` is now paused “before” its body.

```js
const iter = genFunc2();
// genFunc2() is now paused “before” its body:
assert.equal(location, 0);
```

暫停位於函數躰的開括號後面。

`iter` implements the iteration protocol. Therefore, we control the execution of `genFunc2()` via `iter.next()`. Calling that method resumes the paused `genFunc2()` and executes it until there is a `yield`. Then execution pauses and `.next()` returns the operand of the `yield`:

```js
assert.deepEqual(
  iter.next(), {value: 'a', done: false});
// genFunc2() is now paused directly after the first `yield`:
assert.equal(location, 1);
```

Note that the yielded value `'a'` is wrapped in an object, which is how iterators always deliver their values.

We call `iter.next()` again and execution continues where we previously paused. Once we encounter the second `yield`, `genFunc2()` is paused and `.next()` returns the yielded value `'b'`.

```js
assert.deepEqual(
  iter.next(), {value: 'b', done: false});
// genFunc2() is now paused directly after the second `yield`:
assert.equal(location, 2);
```

We call `iter.next()` one more time and execution continues until it leaves the body of `genFunc2()`:

```js
assert.deepEqual(
  iter.next(), {value: undefined, done: true});
// We have reached the end of genFunc2():
assert.equal(location, 3);
```

This time, property `.done` of the result of `.next()` is `true`, which means that the iterator is finished.

#### 35.1.3 Why does `yield` pause execution?

What are the benefits of `yield` pausing execution? Why doesn’t it simply work like the Array method `.push()` and fill the iterable with values without pausing?

Due to pausing, generators provide many of the features of *coroutines* (think processes that are multitasked cooperatively). For example, when you ask for the next value of an iterable, that value is computed *lazily* (on demand). The following two generator functions demonstrate what that means.

```js
/**
 * Returns an iterable over lines
 */
function* genLines() {
  yield 'A line';
  yield 'Another line';
  yield 'Last line';
}

/**
 * Input: iterable over lines
 * Output: iterable over numbered lines
 */
function* numberLines(lineIterable) {
  let lineNumber = 1;
  for (const line of lineIterable) { // input
    yield lineNumber + ': ' + line; // output
    lineNumber++;
  }
}
```

Note that the `yield` in `numberLines()` appears inside a `for-of` loop. `yield` can be used inside loops, but not inside callbacks (more on that later).

Let’s combine both generators to produce the iterable `numberedLines`:

```js
const numberedLines = numberLines(genLines());
assert.deepEqual(
  numberedLines.next(), {value: '1: A line', done: false});
assert.deepEqual(
  numberedLines.next(), {value: '2: Another line', done: false});
```

The key benefit of using generators here is that everything works incrementally: via `numberedLines.next()`, we ask `numberLines()` for only a single numbered line. In turn, it asks `genLines()` for only a single unnumbered line.

This incrementalism continues to work if, for example, `genLines()` reads its lines from a large text file: If we ask `numberLines()` for a numbered line, we get one as soon as `genLines()` has read its first line from the text file.

Without generators, `genLines()` would first read all lines and return them. Then `numberLines()` would number all lines and return them. We therefore have to wait much longer until we get the first numbered line.


#### 35.1.4 Example: Mapping over iterables

The following function `mapIter()` is similar to the Array method `.map()`, but it returns an iterable, not an Array, and produces its results on demand.

```js
function* mapIter(iterable, func) {
  let index = 0;
  for (const x of iterable) {
    yield func(x, index);
    index++;
  }
}

const iterable = mapIter(['a', 'b'], x => x + x);
assert.deepEqual([...iterable], ['aa', 'bb']);
```


### 35.2 Calling generators from generators (advanced)

#### 35.2.1 Calling generators via `yield*`



`yield` only works directly inside generators — so far we haven’t seen a way of delegating yielding to another function or method.

Let’s first examine what does *not* work: in the following example, we’d like `foo()` to call `bar()`, so that the latter yields two values for the former. Alas, a naïve approach fails:

```js
function* bar() {
  yield 'a';
  yield 'b';
}
function* foo() {
  // Nothing happens if we call `bar()`:
  bar();
}
assert.deepEqual(
  [...foo()], []);
```

Why doesn’t this work? The function call `bar()` returns an iterable, which we ignore.

What we want is for `foo()` to yield everything that is yielded by `bar()`. That’s what the `yield*` operator does:

```js
function* bar() {
  yield 'a';
  yield 'b';
}
function* foo() {
  yield* bar();
}
assert.deepEqual(
  [...foo()], ['a', 'b']);
```

In other words, the previous `foo()` is roughly equivalent to:

```js
function* foo() {
  for (const x of bar()) {
    yield x;
  }
}
```

Note that `yield*` works with any iterable:

```js
function* gen() {
  yield* [1, 2];
}
assert.deepEqual(
  [...gen()], [1, 2]);
```

#### 35.2.2 Example: Iterating over a tree

`yield*` lets us make recursive calls in generators, which is useful when iterating over recursive data structures such as trees. Take, for example, the following data structure for binary trees.

```js
class BinaryTree {
  constructor(value, left=null, right=null) {
    this.value = value;
    this.left = left;
    this.right = right;
  }

  * [Symbol.iterator]() {
    yield this.value; // Prefix iteration: parent before children
    if (this.left) {
      // Same as yield* this.left[Symbol.iterator]()
      yield* this.left;
    }
    if (this.right) {
      yield* this.right;
    }
  }
}
```

Method `[Symbol.iterator]()` adds support for the iteration protocol, which means that we can use a `for-of` loop to iterate over an instance of `BinaryTree`:

```js
const tree = new BinaryTree('a',
  new BinaryTree('b',
    new BinaryTree('c'),
    new BinaryTree('d')),
  new BinaryTree('e'));

for (const x of tree) {
  console.log(x);
}
// Output:
// 'a'
// 'b'
// 'c'
// 'd'
// 'e'
```


### 35.3 Background: external iteration vs. internal iteration



In preparation for the next section, we need to learn about two different styles of iterating over the values “inside” an object:

- External iteration (pull): Your code asks the object for the values via an iteration protocol. For example, the `for-of` loop is based on JavaScript’s iteration protocol:

  ```js
  for (const x of ['a', 'b']) {
    console.log(x);
  }
  // Output:
  // 'a'
  // 'b'
  ```

- Internal iteration (push): You pass a callback function to a method of the object and the method feeds the values to the callback. For example, Arrays have the method `.forEach()`:

  ```js
  ['a', 'b'].forEach((x) => {
    console.log(x);
  });
  // Output:
  // 'a'
  // 'b'
  ```

The next section has examples for both styles of iteration.

### 35.4 Use case for generators: reusing traversals

One important use case for generators is extracting and reusing traversals.

#### 35.4.1 The traversal to reuse

As an example, consider the following function that traverses a tree of files and logs their paths (it uses [the Node.js API](https://nodejs.org/en/docs/) for doing so):

```js
function logPaths(dir) {
  for (const fileName of fs.readdirSync(dir)) {
    const filePath = path.resolve(dir, fileName);
    console.log(filePath);
    const stats = fs.statSync(filePath);
    if (stats.isDirectory()) {
      logPaths(filePath); // recursive call
    }
  }
}
```

Consider the following directory:

```
mydir/
    a.txt
    b.txt
    subdir/
        c.txt
```

Let’s log the paths inside `mydir/`:

```js
logPaths('mydir');

// Output:
// 'mydir/a.txt'
// 'mydir/b.txt'
// 'mydir/subdir'
// 'mydir/subdir/c.txt'
```

How can we reuse this traversal and do something other than logging the paths?

#### 35.4.2 Internal iteration (push)

One way of reusing traversal code is via *internal iteration*: Each traversed value is passsed to a callback (line A).

```js
function visitPaths(dir, callback) {
  for (const fileName of fs.readdirSync(dir)) {
    const filePath = path.resolve(dir, fileName);
    callback(filePath); // (A)
    const stats = fs.statSync(filePath);
    if (stats.isDirectory()) {
      visitPaths(filePath, callback);
    }
  }
}
const paths = [];
visitPaths('mydir', p => paths.push(p));
assert.deepEqual(
  paths,
  [
    'mydir/a.txt',
    'mydir/b.txt',
    'mydir/subdir',
    'mydir/subdir/c.txt',
  ]);
```

#### 35.4.3 External iteration (pull)

Another way of reusing traversal code is via *external iteration*: We can write a generator that yields all traversed values (line A).

```js
function* iterPaths(dir) {
  for (const fileName of fs.readdirSync(dir)) {
    const filePath = path.resolve(dir, fileName);
    yield filePath; // (A)
    const stats = fs.statSync(filePath);
    if (stats.isDirectory()) {
      yield* iterPaths(filePath);
    }
  }
}
const paths = [...iterPaths('mydir')];
```

### 35.5 Advanced features of generators

[The chapter on generators](https://exploringjs.com/es6/ch_generators.html) in *Exploring ES6* covers two features that are beyond the scope of this book:

- `yield` can also *receive* data, via an argument of `.next()`.
- Generators can also `return` values (not just `yield` them). Such values do not become iteration values, but can be retrieved via `yield*`.