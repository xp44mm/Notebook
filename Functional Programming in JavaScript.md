## 1.2. What is functional programming?

In simple terms, functional programming is a software development style that places a major emphasis on the use of functions. FP requires you to think a bit differently about how to approach the tasks you're facing. It's not a matter of just applying functions to come up with a result; the goal, rather, is to abstract control flows and operations on data with functions in order to avoid side effects and reduce mutation of state in your application.

Listing 1.1 captures the process of decomposing a program into smaller pieces that are more reusable, more reliable, and easier to understand, and then combining them to form an entire program that is easier to reason about as a whole. Every functional program follows this fundamental principle.

使函数的每一部分都可以被替换。

### 1.2.1. Functional programming is declarative

Declarative programming paradigms: it's a paradigm that expresses a set of operations without revealing how they're implemented or how data flows through them. 

Imperative programming treats a computer program as merely a sequence of top-to-bottom statements that changes the state of the system in order to compute a result.

Imperative programming tells the computer, in great detail, how to perform a certain task.

Declarative programming, on the other hand, separates program description from evaluation. It focuses on the use of *expressions* to describe what the logic of a program is without necessarily specifying its control flow or state changes.

Stateless code has zero chance of changing or breaking global state.

### 1.2.2. Pure functions and the problem with side effects

A pure function has the following qualities:

- It depends only on the input provided and not on any hidden or external state that may change during its evaluation or between calls.
- It doesn't inflict changes beyond their scope, such as modifying a global object or a parameter passed by reference.

Side effects can occur in many situations, including these:

- Changing a variable, property, or data structure globally.
- Changing the original value of a function's argument.
- Processing user input.
- Throwing an exception, unless it's caught within the same function.
- Printing to the screen or logging.
- Querying the HTML documents, browser cookies, or databases

### 1.2.3. Referential transparency and substitutability

*Purity* in this sense refers to the existence of a pure mapping between a function's arguments and its return value. Hence, if a function consistently yields the same result on the same input, it's said to be *referentially transparent*.

### 1.2.4. Preserving immutable data

Functional programming refers to the declarative evaluation of pure functions to create immutable programs by avoiding externally observable side effects.

## 1.3. Benefits of functional programming

### 1.3.1. Encouraging the decomposition of complex tasks

单一职责, which states that functions should have a single purpose.

函数的输入参数越少，函数越简单。

组合：

```F#
f>>g = f(g(x))
```

`f` composed of `g`

Functional composition leads to code in which the meaning of the entire expression can be understood from the meaning of its individual pieces.

Because `compose` accepts other functions as arguments, it's known as a *higher-order function*.

### 1.3.2. Processing data using fluent chains

A chain is a sequential invocation of functions that share a common object return value.

A function chain is a *lazy evaluated* program, which means it defers its execution until needed. This effectively simulates the *call-by-need* behavior built into other functional languages.

#### Listing 1.5. Programming with function chains

```js
import * as _ from 'lodash'

export function test() {
    let enrollment = [
        { enrolled: 2, grade: 100 },
        { enrolled: 2, grade: 80 },
        { enrolled: 1, grade: 89 }
    ];

    let v =
        _.chain(enrollment)
            .filter(s => s.enrolled > 1)
            .map(s => s.grade)
            .sum()
            .value()

    console.log(v)
}
```



变量、循环、分支、异常。

### 1.3.3. Reacting to the complexity of asynchronous applications

```typescript
export class SSN extends MyComponent<any, any> {
    componentDidMount() {
        const value$ = Observable.fromEvent<KeyboardEvent<any>>(this.ssn, 'keyup')
            .map(e => (<HTMLInputElement>e.target).value)
            .filter(ssn => ssn !== null && ssn.length !== 0)
            .map(ssn => ssn.replace(/^\s*|\s*$|\-/g, ''))
            .skipWhile(ssn => ssn.length !== 9)

        this.subscribe(
            value$.subscribe(
                validSsn => console.log(`Valid SSN ${validSsn}`)
            )

        )
    }

    ssn: HTMLInputElement;
    render() {
        return div(
            input({ ref: i => this.ssn = i! }),
        )
    }
}
```

# Chapter 2. Higher-order JavaScript

## 2.1. Why JavaScript?

## 2.2. Functional vs. object-oriented programming

But adding more functionality to existing objects can be tricky when it doesn't necessarily apply to all of its descendants. The reason for painting this model is that the main difference between object-oriented and functional applications is how this data (the object's properties) and behavior (functions) are organized.

The focus of OOP is to create inheritance hierarchies (such as Student from Person) with methods and data tightly bound together. Functional programming, on the other hand, favors general polymorphic functions that crosscut different data types and avoid the use of this.

FP is based on purity and referential transparency, by isolating the behavior from the state you can add more operations by defining and combining new functions that work on those types. Doing this, you end up with simple objects in charge of storing data, and versatile functions that can work on those objects as arguments, which can be composed to achieve specialized functionality.

Table 2.1. Comparing some important qualities of object-oriented and functional programming. These qualities are themes that are discussed throughout this book.

|                     | Functional                                      | Object-oriented                                 |
| ------------------- | ----------------------------------------------- | ----------------------------------------------- |
| Unit of composition | Functions                                       | Objects (classes)                               |
| Programming style   | Declarative                                     | Imperative                                      |
| Data and behavior   | Loosely coupled into pure, standalone functions | Tightly coupled in classes with methods         |
| State management    | Treats objects as immutable values              | Favors mutation of objects via instance methods |
| Control flow        | Functions and recursion                         | Loops and conditionals                          |
| Thread safety       | Enables concurrent programming                  | Difficult to achieve                            |
| Encapsulation       | Not needed because everything is immutable      | Needed to protect data integrity                |

### 2.2.1. Managing the state of JavaScript objects

The *state* of a program can be defined as a snapshot of the data stored in all of its objects at any moment in time.

An immutable object that contains immutable functionality can be considered pure.

### 2.2.2. Treating objects as values

Strings and numbers, these primitive types are inherently immutable. In functional programming, we call types that behave this way *values*.

A value object is one whose equality doesn't depend on identity or reference, just on its value; once declared, its state may not change. In addition to `number`s and `string`s, some examples of value objects are types like `tuple`, `pair`, `point`, `zipCode`, `coordinate`, `money`, `date`, and others.

In JavaScript, you can use functions and guard access to a ==ZIP code==’s internal state by returning an *object literal interface* that exposes a small set of methods to the caller and treats `_code` and `_location` as pseudo-private variables. These variables are only accessible in the object literal via *closures*.

Using methods to return new copies.

### 2.2.4. Navigating and modifying object graphs with lenses

*Lenses*, also known as *functional references*, are functional programming's solution to accessing and immutably manipulating attributes of stateful data types. Internally, lenses work similarly to a copy-on-write strategy by using an internal storage component that knows how to properly manage and copy state.

```js
let person = new Person('Alonzo', 'Church', '444-44-4444');
let lastnameLens = R.lensProp('lastname');

console.log(R.view(lastnameLens, person));

let newPerson = R.set(lastnameLens, 'Mourning', person);

console.log(newPerson.lastname); //-> 'Mourning'
console.log(person.lastname); //-> 'Church'

person.address = new Address(
    'US', 'NJ', 'Princeton', zipCode('08544', '1234'),
    'Alexander St.');

let zipPath = ['address', 'zip']
let zipLens = R.lens(R.path(zipPath), R.assocPath(zipPath))

console.log(R.view(zipLens, person).toString())

let newerPerson = R.set(zipLens, zipCode('90210', '5678'), person);

console.log(R.view(zipLens, newerPerson).toString()); //-> zipCode('90210', '5678')
console.log(R.view(zipLens, person).toString());    //-> zipCode('08544', '1234')
console.log(newerPerson !== person); //-> true
```

## 2.3. Functions

A *function* is any callable expression that can be evaluated by applying the `()` operator to it.

### 2.3.1. Functions as first-class citizens

In JavaScript, the term *first-class* comes from making functions actual objects in the language, also called first-class citizens.

### 2.3.2. Higher-order functions

Because functions behave like regular objects, you can intuitively expect that they can be passed in as function arguments and returned from other functions. These are called *higher-order functions*.

Because functions are first-class and higher-order, JavaScript functions can behave as values, which implies that a function is nothing more than a yet-to-be-executed value defined immutably based on the input provided to the function.

函数参数可以是被动名词：`multiplier`, `comparator`, and `action`.

### 2.3.3. Types of function invocation

### 2.3.4. Function methods

## 2.4. Closures and scopes

A *closure* is a data structure that binds a function to its environment at the moment it's declared. It's based on the textual location of the function declaration; therefore, a closure is also called a *static scope* or *lexical scope* surrounding the function definition.

The rules that govern the behavior of a function's closure are closely related to JavaScript's scoping rules. A scope groups a set of variable bindings and defines a section of code in which a variable is defined. In essence, a closure is a function's inheritance of scopes akin to how an object's method has access to its inherited instance variables—both have references to their parents.

嵌套函数作为词法边界。

It's important to notice that even though the variables in nested function are no longer in the active scope, they're still accessible from the returned function when invoked. Essentially, you can imagine the nested functions as functions that package not only their computation but also a snapshot of all variables surrounding them. 

A closure contains variables that appear in the outer (global) scope, the parent function's inner scope, and the parent function's parameters ~~and additional variables declared after the function declaration~~. The code defined in the function's body can access variables and objects defined in each of these scopes. All functions share the global scope.

### 2.4.1. Problems with the global scope

### 2.4.2. JavaScript's function scope

JavaScript's *name-resolution order*, which first looks into the closest scope level surrounding a variable lookup and moves outward. It first checks the function's scope (local), then moves into its parent's scope (if there is one), and finally moves into the global scope. If the variable x isn't found, the function returns undefined.

### 2.4.3. A pseudo-block scope

### 2.4.4. Practical applications of closures

Making asynchronous server-side calls

Closures can also provide a way to manage your global namespace to avoid globally shared data. Library and module authors take closures to the next level by hiding an entire module's private methods and data. This is referred to as the *Module pattern* because it uses a single *immediately invoked function expression* (IIFE) to encapsulate internal variables while allowing you to export the necessary set of functionality to the outside world and severely reduce the number of global references.

```js
let MyModule = (function MyModule(exp) {
    let x = []
    exp.me = function () {

    }
    exp.you = function () {

    }
}(MyModule || {}))
```

