## 23 Callable values


### 23.1 Kinds of functions



JavaScript has two categories of functions:

- An ordinary function can play several roles:

  - Real function
  - Method
  - Constructor function

- A specialized function can only play one of those roles – for example:

  - An *arrow function* can only be a real function.
  - A *method* can only be a method.
  - A *class* can only be a constructor function.

The next two sections explain what all of those things mean.

### 23.2 Ordinary functions



The following code shows three ways of doing (roughly) the same thing: creating an ordinary function.

```js
// Function declaration (a statement)
function ordinary1(a, b, c) {
  // ···
}

// const plus anonymous function expression
const ordinary2 = function (a, b, c) {
  // ···
};

// const plus named function expression
const ordinary3 = function myName(a, b, c) {
  // `myName` is only accessible in here
};
```

As we have seen in [§10.8 “Declarations: scope and activation”](https://exploringjs.com/impatient-js/ch_variables-assignment.html#declarations-scope-activation), function declarations are activated early, while variable declarations (e.g., via `const`) are not.

The syntax of function declarations and function expressions is very similar. The context determines which is which. For more information on this kind of syntactic ambiguity, consult [§6.5 “Ambiguous syntax”](https://exploringjs.com/impatient-js/ch_syntax.html#ambiguous-syntax).

#### 23.2.1 Parts of a function declaration

Let’s examine the parts of a function declaration via an example:

```js
function add(x, y) {
  return x + y;
}
```

- `add` is the *name* of the function declaration.
- `add(x, y)` is the *head* of the function declaration.
- `x` and `y` are the *parameters*.
- The curly braces (`{` and `}`) and everything between them are the *body* of the function declaration.
- The `return` statement explicitly returns a value from the function.

#### 23.2.2 Roles played by ordinary functions



Consider the following function declaration from the previous section:

```js
function add(x, y) {
  return x + y;
}
```

This function declaration creates an ordinary function whose name is `add`. As an ordinary function, `add()` can play three roles:

- Real function: invoked via a function call.

  ```js
  assert.equal(add(2, 1), 3);
  ```

- Method: stored in property, invoked via a method call.

  ```js
  const obj = { addAsMethod: add };
  assert.equal(obj.addAsMethod(2, 4), 6); // (A)
  ```

  In line A, `obj` is called the *receiver* of the method call. It can be accessed via `this` inside the method.

- Constructor function/class: invoked via `new`.

  ```js
  const inst = new add();
  assert.equal(inst instanceof add, true);
  ```

  (As an aside, the names of classes normally start with capital letters.)

**Ordinary function vs. real function**

In JavaScript, we distinguish:

- The entity *ordinary function*
- The role *real function*, as played by an ordinary function

In many other programming languages, the entity *function* only plays one role – *function*. Therefore, the same name *function* can be used for both.

#### 23.2.3 Names of ordinary functions

The name of a function expression is only accessible inside the function, where the function can use it to refer to itself (e.g., for self-recursion):

```js
const func = function funcExpr() { return funcExpr };
assert.equal(func(), func);

// The name `funcExpr` only exists inside the function:
assert.throws(() => funcExpr(), ReferenceError);
```

In contrast, the name of a function declaration is accessible inside the current scope:

```js
function funcDecl() { return funcDecl }

// The name `funcDecl` exists in the current scope
assert.equal(funcDecl(), funcDecl);
```

### 23.3 Specialized functions

Specialized functions are single-purpose versions of ordinary functions. Each one of them specializes in a single role:

- The purpose of an *arrow function* is to be a real function:

  ```js
  const arrow = () => { return 123 };
  assert.equal(arrow(), 123);
  ```

- The purpose of a *method* is to be a method:

  ```js
  const obj = { method() { return 'abc' } };
  assert.equal(obj.method(), 'abc');
  ```

- The purpose of a *class* is to be a constructor function:

  ```js
  class MyClass { /* ··· */ }
  const inst = new MyClass();
  ```

Apart from nicer syntax, each kind of specialized function also supports new features, making them better at their jobs than ordinary functions.

- Arrow functions are explained later in this chapter.
- Methods are explained in the chapter on single objects.
- Classes are explained in the chapter on classes.

Tbl. 15 lists the capabilities of ordinary and specialized functions.

|                   | Function call          | Method call      | Constructor call |
| :---------------- | :--------------------- | :--------------- | :--------------- |
| Ordinary function | (`this === undefined`) | `✔`              | `✔`              |
| Arrow function    | `✔`                    | (lexical `this`) | `✘`              |
| Method            | (`this === undefined`) | `✔`              | `✘`              |
| Class             | `✘`                    | `✘`              | `✔`              |

#### 23.3.1 Specialized functions are still functions

It’s important to note that arrow functions, methods, and classes are still categorized as functions:

```js
> (() => {}) instanceof Function
true
> ({ method() {} }.method) instanceof Function
true
> (class SomeClass {}) instanceof Function
true
```

#### 23.3.2 Recommendation: prefer specialized functions

Normally, you should prefer specialized functions over ordinary functions, especially classes and methods. The choice between an arrow function and an ordinary function is less clear-cut, though:

- On one hand, an ordinary function has `this` as an implicit parameter. That parameter is set to `undefined` during function calls – which is not what you want. An arrow function treats `this` like any other variable. For details, see [§25.4.6 “Avoiding the pitfalls of `this`”](https://exploringjs.com/impatient-js/ch_single-objects.html#avoiding-pitfalls-of-this).

- On the other hand, I like the syntax of a function declaration (which produces an ordinary function). If you don’t use `this` inside it, it is mostly equivalent to `const` plus arrow function:

  ```js
  function funcDecl(x, y) {
    return x * y;
  }
  const arrowFunc = (x, y) => {
    return x * y;
  };
  ```

#### 23.3.3 Arrow functions



Arrow functions were added to JavaScript for two reasons:

1. To provide a more concise way for creating functions.
2. To make working with real functions easier: You can’t refer to the `this` of the surrounding scope inside an ordinary function.

Next, we’ll first look at the syntax of arrow functions and then how they help with `this`.

##### 23.3.3.1 The syntax of arrow functions

Let’s review the syntax of an anonymous function expression:

```js
const f = function (x, y, z) { return 123 };
```

The (roughly) equivalent arrow function looks as follows. Arrow functions are expressions.

```js
const f = (x, y, z) => { return 123 };
```

Here, the body of the arrow function is a block. But it can also be an expression. The following arrow function works exactly like the previous one.

```js
const f = (x, y, z) => 123;
```

If an arrow function has only a single parameter and that parameter is an identifier (not [a destructuring pattern](https://exploringjs.com/impatient-js/ch_destructuring.html)) then you can omit the parentheses around the parameter:

```js
const id = x => x;
```

That is convenient when passing arrow functions as parameters to other functions or methods:

```js
> [1,2,3].map(x => x+1)
[ 2, 3, 4 ]
```

This previous example demonstrates one benefit of arrow functions – conciseness. If we perform the same task with a function expression, our code is more verbose:

```js
[1,2,3].map(function (x) { return x+1 });
```

##### 23.3.3.2 Arrow functions: lexical `this`

Ordinary functions can be both methods and real functions. Alas, the two roles are in conflict:

- As each ordinary function can be a method, it has its own `this`.
- The own `this` makes it impossible to access the `this` of the surrounding scope from inside an ordinary function. And that is inconvenient for real functions.

The following code demonstrates the issue:

```js
const person = {
  name: 'Jill',
  someMethod() {
    const ordinaryFunc = function () {
      assert.throws(
        () => this.name, // (A)
        /^TypeError: Cannot read property 'name' of undefined$/);
    };
    const arrowFunc = () => {
      assert.equal(this.name, 'Jill'); // (B)
    };
    
    ordinaryFunc();
    arrowFunc();
  },
}
```

In this code, we can observe two ways of handling `this`:

- Dynamic `this`: In line A, we try to access the `this` of `.someMethod()` from an ordinary function. There, it is *shadowed* by the function’s own `this`, which is `undefined` (as filled in by the function call). Given that ordinary functions receive their `this` via (dynamic) function or method calls, their `this` is called *dynamic*.
- Lexical `this`: In line B, we again try to access the `this` of `.someMethod()`. This time, we succeed because the arrow function does not have its own `this`. `this` is resolved *lexically*, just like any other variable. That’s why the `this` of arrow functions is called *lexical*.



##### 23.3.3.3 Syntax pitfall: returning an object literal from an arrow function

If you want the expression body of an arrow function to be an object literal, you must put the literal in parentheses:

```js
const func1 = () => ({a: 1});
assert.deepEqual(func1(), { a: 1 });
```

If you don’t, JavaScript thinks, the arrow function has a block body (that doesn’t return anything):

```js
const func2 = () => {a: 1};
assert.deepEqual(func2(), undefined);
```

`{a: 1}` is interpreted as a block with [the label `a:`](https://exploringjs.com/impatient-js/ch_control-flow.html#labels) and the expression statement `1`. Without an explicit `return` statement, the block body returns `undefined`.

This pitfall is caused by [syntactic ambiguity](https://exploringjs.com/impatient-js/ch_syntax.html#ambiguous-syntax): object literals and code blocks have the same syntax. We use the parentheses to tell JavaScript that the body is an expression (an object literal) and not a statement (a block).

For more information on shadowing `this`, consult §25.4.5 “`this` pitfall: accidentally shadowing `this`”.

### 23.4 More kinds of functions and methods

**This section is a summary of upcoming content**

This section mainly serves as a reference for the current and upcoming chapters. Don’t worry if you don’t understand everything.

So far, all (real) functions and methods, that we have seen, were:

- Single-result
- Synchronous

Later chapters will cover other modes of programming:

- *Iteration* treats objects as containers of data (so-called *iterables*) and provides a standardized way for retrieving what is inside them. If a function or a method returns an iterable, it returns multiple values.
- *Asynchronous programming* deals with handling a long-running computation. You are notified when the computation is finished and can do something else in between. The standard pattern for asynchronously delivering single results is called *Promise*.

These modes can be combined – for example, there are synchronous iterables and asynchronous iterables.

Several new kinds of functions and methods help with some of the mode combinations:

- *Async functions* help implement functions that return Promises. There are also *async methods*.
- *Synchronous generator functions* help implement functions that return synchronous iterables. There are also *synchronous generator methods*.
- *Asynchronous generator functions* help implement functions that return asynchronous iterables. There are also *asynchronous generator methods*.

That leaves us with 4 kinds (2 × 2) of functions and methods:

- Synchronous vs. asynchronous
- Generator vs. single-result

Tbl. [16](https://exploringjs.com/impatient-js/ch_callables.html#tbl:syntax-functions-methods) gives an overview of the syntax for creating these 4 kinds of functions and methods.

|                              |                       | **Result**     | **Values** |
| :--------------------------- | :-------------------- | :------------- | :--------- |
| **Sync function**            | **Sync method**       |                |            |
| `function f() {}`            | `{ m() {} }`          | value          | 1          |
| `f = function () {}`         |                       |                |            |
| `f = () => {}`               |                       |                |            |
| **Sync generator function**  | **Sync gen. method**  |                |            |
| `function* f() {}`           | `{ * m() {} }`        | iterable       | 0+         |
| `f = function* () {}`        |                       |                |            |
| **Async function**           | **Async method**      |                |            |
| `async function f() {}`      | `{ async m() {} }`    | Promise        | 1          |
| `f = async function () {}`   |                       |                |            |
| `f = async () => {}`         |                       |                |            |
| **Async generator function** | **Async gen. method** |                |            |
| `async function* f() {}`     | `{ async * m() {} }`  | async iterable | 0+         |
| `f = async function* () {}`  |                       |                |            |

### 23.5 Returning values from functions and methods

(Everything mentioned in this section applies to both functions and methods.)

The `return` statement explicitly returns a value from a function:

```
function func() {
  return 123;
}
assert.equal(func(), 123);
```

Another example:

```
function boolToYesNo(bool) {
  if (bool) {
    return 'Yes';
  } else {
    return 'No';
  }
}
assert.equal(boolToYesNo(true), 'Yes');
assert.equal(boolToYesNo(false), 'No');
```

If, at the end of a function, you haven’t returned anything explicitly, JavaScript returns `undefined` for you:

```
function noReturn() {
  // No explicit return
}
assert.equal(noReturn(), undefined);
```

### 23.6 Parameter handling

Once again, I am only mentioning functions in this section, but everything also applies to methods.

#### 23.6.1 Terminology: parameters vs. arguments



The term *parameter* and the term *argument* basically mean the same thing. If you want to, you can make the following distinction:

- *Parameters* are part of a function definition. They are also called *formal parameters* and *formal arguments*.
- *Arguments* are part of a function call. They are also called *actual parameters* and *actual arguments*.

#### 23.6.2 Terminology: callback



A *callback* or *callback function* is a function that is an argument of a function or method call.

The following is an example of a callback:

```js
const myArray = ['a', 'b'];
const callback = (x) => console.log(x);
myArray.forEach(callback);

// Output:
// 'a'
// 'b'
```

**JavaScript uses the term *callback* broadly**

In other programming languages, the term *callback* often has a narrower meaning: it refers to a pattern for delivering results asynchronously, via a function-valued parameter. In this meaning, the *callback* (or *continuation*) is invoked after a function has completely finished its computation.

Callbacks as an asynchronous pattern, are described [in the chapter on asynchronous programming](https://exploringjs.com/impatient-js/ch_async-js.html#callback-pattern).

#### 23.6.3 Too many or not enough arguments

JavaScript does not complain if a function call provides a different number of arguments than expected by the function definition:

- Extra arguments are ignored.
- Missing parameters are set to `undefined`.

For example:

```js
function foo(x, y) {
  return [x, y];
}

// Too many arguments:
assert.deepEqual(foo('a', 'b', 'c'), ['a', 'b']);

// The expected number of arguments:
assert.deepEqual(foo('a', 'b'), ['a', 'b']);

// Not enough arguments:
assert.deepEqual(foo('a'), ['a', undefined]);
```

#### 23.6.4 Parameter default values



Parameter default values specify the value to use if a parameter has not been provided – for example:

```js
function f(x, y=0) {
  return [x, y];
}

assert.deepEqual(f(1), [1, 0]);
assert.deepEqual(f(), [undefined, 0]);
```

`undefined` also triggers the default value:

```js
assert.deepEqual(
  f(undefined, undefined),
  [undefined, 0]);
```

#### 23.6.5 Rest parameters



A rest parameter is declared by prefixing an identifier with three dots (`...`). During a function or method call, it receives an Array with all remaining arguments. If there are no extra arguments at the end, it is an empty Array – for example:

```js
function f(x, ...y) {
  return [x, y];
}
assert.deepEqual(
  f('a', 'b', 'c'),
  ['a', ['b', 'c']]);
assert.deepEqual(
  f(),
  [undefined, []]);
```

##### 23.6.5.1 Enforcing a certain number of arguments via a rest parameter

You can use a rest parameter to enforce a certain number of arguments. Take, for example, the following function:

```js
function createPoint(x, y) {
  return {x, y};
    // same as {x: x, y: y}
}
```

This is how we force callers to always provide two arguments:

```js
function createPoint(...args) {
  if (args.length !== 2) {
    throw new Error('Please provide exactly 2 arguments!');
  }
  const [x, y] = args; // (A)
  return {x, y};
}
```

In line A, we access the elements of `args` via [*destructuring*](https://exploringjs.com/impatient-js/ch_destructuring.html).

#### 23.6.6 Named parameters



When someone calls a function, the arguments provided by the caller are assigned to the parameters received by the callee. Two common ways of performing the mapping are:

1. Positional parameters: An argument is assigned to a parameter if they have the same position. A function call with only positional arguments looks as follows.

   

   ```js
   selectEntries(3, 20, 2)
   ```

2. Named parameters: An argument is assigned to a parameter if they have the same name. JavaScript doesn’t have named parameters, but you can simulate them. For example, this is a function call with only (simulated) named arguments:

   

   ```js
   selectEntries({start: 3, end: 20, step: 2})
   ```

Named parameters have several benefits:

- They lead to more self-explanatory code because each argument has a descriptive label. Just compare the two versions of `selectEntries()`: with the second one, it is much easier to see what happens.
- The order of the arguments doesn’t matter (as long as the names are correct).
- Handling more than one optional parameter is more convenient: callers can easily provide any subset of all optional parameters and don’t have to be aware of the ones they omit (with positional parameters, you have to fill in preceding optional parameters, with `undefined`).

#### 23.6.7 Simulating named parameters

JavaScript doesn’t have real named parameters. The official way of simulating them is via object literals:

```js
function selectEntries({start=0, end=-1, step=1}) {
  return {start, end, step};
}
```

This function uses [*destructuring*](https://exploringjs.com/impatient-js/ch_destructuring.html) to access the properties of its single parameter. The pattern it uses is an abbreviation for the following pattern:

```js
{start: start=0, end: end=-1, step: step=1}
```

This destructuring pattern works for empty object literals:

```js
> selectEntries({})
{ start: 0, end: -1, step: 1 }
```

But it does not work if you call the function without any parameters:

```js
> selectEntries()
TypeError: Cannot destructure property `start` of 'undefined' or 'null'.
```

You can fix this by providing a default value for the whole pattern. This default value works the same as default values for simpler parameter definitions: if the parameter is missing, the default is used.

```js
function selectEntries({start=0, end=-1, step=1} = {}) {
  return {start, end, step};
}
assert.deepEqual(
  selectEntries(),
  { start: 0, end: -1, step: 1 });
```

#### 23.6.8 Spreading (`...`) into function calls



If you put three dots (`...`) in front of the argument of a function call, then you *spread* it. That means that the argument must be [an *iterable* object](https://exploringjs.com/impatient-js/ch_sync-iteration.html) and the iterated values all become arguments. In other words, a single argument is expanded into multiple arguments – for example:

```js
function func(x, y) {
  console.log(x);
  console.log(y);
}
const someIterable = ['a', 'b'];
func(...someIterable);
  // same as func('a', 'b')

// Output:
// 'a'
// 'b'
```

Spreading and rest parameters use the same syntax (`...`), but they serve opposite purposes:

- Rest parameters are used when defining functions or methods. They collect arguments into Arrays.
- Spread arguments are used when calling functions or methods. They turn iterable objects into arguments.

##### 23.6.8.1 Example: spreading into `Math.max()`

`Math.max()` returns the largest one of its zero or more arguments. Alas, it can’t be used for Arrays, but spreading gives us a way out:

```js
> Math.max(-1, 5, 11, 3)
11
> Math.max(...[-1, 5, 11, 3])
11
> Math.max(-1, ...[-5, 11], 3)
11
```

##### 23.6.8.2 Example: spreading into `Array.prototype.push()`

Similarly, the Array method `.push()` destructively adds its zero or more parameters to the end of its Array. JavaScript has no method for destructively appending an Array to another one. Once again, we are saved by spreading:

```js
const arr1 = ['a', 'b'];
const arr2 = ['c', 'd'];

arr1.push(...arr2);
assert.deepEqual(arr1, ['a', 'b', 'c', 'd']);
```

**Exercises: Parameter handling**

- Positional parameters: `exercises/callables/positional_parameters_test.mjs`
- Named parameters: `exercises/callables/named_parameters_test.mjs`

### 23.7 Dynamically evaluating code: `eval()`, `new Function()` (advanced)



Next, we’ll look at two ways of evaluating code dynamically: `eval()` and `new Function()`.

#### 23.7.1 `eval()`

Given a string `str` with JavaScript code, `eval(str)` evaluates that code and returns the result:

```js
> eval('2 ** 4')
16
```

There are two ways of invoking `eval()`:

- *Directly*, via a function call. Then the code in its argument is evaluated inside the current scope.
- *Indirectly*, not via a function call. Then it evaluates its code in global scope.

“Not via a function call” means “anything that looks different than `eval(···)`”:

- `eval.call(undefined, '···')`
- `(0, eval)('···')` (uses the comma operator)
- `globalThis.eval('···')`
- `const e = eval; e('···')`
- Etc.

The following code illustrates the difference:

```js
globalThis.myVariable = 'global';
function func() {
  const myVariable = 'local';
  
  // Direct eval
  assert.equal(eval('myVariable'), 'local');
  
  // Indirect eval
  assert.equal(eval.call(undefined, 'myVariable'), 'global');
}
```

Evaluating code in global context is safer because the code has access to fewer internals.

#### 23.7.2 `new Function()`

`new Function()` creates a function object and is invoked as follows:

```js
const func = new Function('«param_1»', ···, '«param_n»', '«func_body»');
```

The previous statement is equivalent to the next statement. Note that `«param_1»`, etc., are not inside string literals, anymore.

```js
const func = function («param_1», ···, «param_n») {
  «func_body»
};
```

In the next example, we create the same function twice, first via `new Function()`, then via a function expression:

```js
const times1 = new Function('a', 'b', 'return a * b');
const times2 = function (a, b) { return a * b };
```

**`new Function()` creates non-strict mode functions**

Functions created via `new Function()` are sloppy.

#### 23.7.3 Recommendations

Avoid dynamic evaluation of code as much as you can:

- It’s a security risk because it may enable an attacker to execute arbitrary code with the privileges of your code.
- It may be switched off – for example, in browsers, via [a Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP).

Very often, JavaScript is dynamic enough so that you don’t need `eval()` or similar. In the following example, what we are doing with `eval()` (line A) can be achieved just as well without it (line B).

```js
const obj = {a: 1, b: 2};
const propKey = 'b';

assert.equal(eval('obj.' + propKey), 2); // (A)
assert.equal(obj[propKey], 2); // (B)
```

If you have to dynamically evaluate code:

- Prefer `new Function()` over `eval()`: it always executes its code in global context and a function provides a clean interface to the evaluated code.
- Prefer indirect `eval` over direct `eval`: evaluating code in global context is safer.