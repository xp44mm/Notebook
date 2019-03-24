# Part 2. Get functional

# Chapter 3. Few data structures, many operations

## 3.1. Understanding your application’s control flow

The path a program takes to arrive at a solution is known as its *control flow*.

An imperative program made up of a series of operations (or statements) controlled by branches and loops.

On the other hand, declarative programs, specifically functional ones, raise the level of abstraction by using a minimally structured flow made up of independent black-box operations that connect in a simple topology. These connected operations are nothing more than higher-order functions that move state from one operation to the next. Working functionally with data structures such as arrays lends itself to this style of development and treats data and control flow as simple connections between high-level components.

## 3.2. Method chaining

*Method chaining* is an OOP pattern that allows multiple methods to be called in a single statement. When these methods all belong to the same object, method chaining is referred to as *method cascading*.

## 3.3. Function chaining

Lodash.js 函数链：

```js
_([1,2,3]).reverse().map(
    p => 2*p
).value()
```

## 3.4. Reasoning about your code

### 3.4.1. Declarative and lazy function chains

functional programming focuses on operations more than on the structure of the data.

*point-free*: 不使用所要处理的值，只合成运算过程。

### 3.4.2. SQL-like data: functions as data

## 3.5. Learning to think recursively

### 3.5.1. What is recursion?

Recursion is a technique designed to solve problems by decomposing them into smaller, self-similar problems that, when combined, arrive at the original solution. A recursive function has two main parts:

- Base cases (also known as the terminating condition)
- Recursive cases

### 3.5.2. Learning to think recursively

*lateral thinking*

*tail-call optimization*

*tail position*

### 3.5.3. Recursively defined data structures

A *node* is an object that contains a value, a reference to its parent, and an array of children. If a node has no parent, it’s considered the root. Here’s the definition of the `Node` type.

```typescript
class Knob {
    _children: Knob[] = []
    _parent: Knob | null = null

    constructor(public value: any) {
    }

    isRoot() {
        return isValid(this._parent);
    }

    get children() {
        return this._children;
    }

    hasChildren() {
        return this._children.length > 0;
    }

    append(child) {
        child._parent = this;
        this._children.push(child);
        return this;
    }

    toString() {
        return `Node (val: ${this.value}, children: ${this._children.length})`;
    }
}
```

Trees are recursively defined data structures that contain a root node:

A preorder traversal of this tree has the following steps, starting with root:


1. Display the data part of the root element

2. Traverse the left subtree by recursively calling the preorder function

3. Traverse the right subtree the same way

# Chapter 4. Toward modular, reusable code

## 4.1. Method chains vs. function pipelines

Viewing functions as mappings of types.

Functional programming removes the limitations present in *method chaining* and provides the flexibility to combine any set of functions, no matter where they come from. A *pipeline* is a directional sequence of functions loosely arranged so that the output of one is input into the next.

方法链是同一对象的紧连接，函数管道是不同类型的松耦合。

Formally speaking, two functions `f` and `g` are *type-compatible* if the output of `f` has a type equivalent to the set of inputs of `g`.

*Arity* can be defined as the number of arguments a function accepts; it’s also referred to as the function’s *length*.

函数参数的长度反映了函数的复杂性。越长的元数代表更高的复杂性。

*Partial application* is an operation that initializes a subset of a nonvariadic function’s parameters to fixed values, creating a function of smaller arity.

separating a function’s description from its evaluation.

programming to interfaces



### 4.5.4. Coping with pure and impure code

副作用是指：

- 依赖了外部可变化的数据。
- 修改了外部的数据。

Using `compose` (or `pipe`) means never having to declare arguments (known as the *points* of a function), making your code declarative and more succinct or *point-free*.

Combinators are higher-order functions that can combine primitive artifacts like other functions (or other combinators) and behave as control logic. Combinators typically don’t declare any variables of their own or contain any business logic; they’re meant to orchestrate the flow of a functional program.

相关、独立的操作序列

### 5.2.1. Wrapping unsafe values

访问一个被包裹的值只能靠映射一个操作到它的容器

映射是一个函数。引用透明性：给定同样的输入参数，必须始终映射到同样的结果。映射是一个控制门，插lambda表达式，特定行为，转换封装值。

#### Listing 5.1 Functional data type to wrap values

```javascript
export class Wrapper {
    constructor(value) {
        this._value = value;
    }

    // map :: (A -> B) -> A -> B
    map(f) {
        return f(this._value);
    }

    // fmap :: (A -> B) -> Wrapper[A] -> Wrapper[B]
    fmap(f) {
        return new Wrapper(f(this._value));
    }

    toString() {
        return 'Wrapper (' + this._value + ')';
    }
};

// wrap :: A -> Wrapper(A)
export const wrap = (val) => new Wrapper(val);
```

`fmap` knows how to apply functions to values wrapped in a context. It first opens the container, then applies the given function to its value, and finally closes the value back into a new container of the same type. This type of function is known as a *functor*.

数组是函子

`compose`是函子

函子重要属性：

- They must be side effect-free
- They must be composable

Functors map functions of one type to another

Monads are the containers that functors “reach into.”

### 5.3.1. Monads: from control flow to data flow

But a better strategy is to make this function more honest about how it handles each case and state that it returns a valid number when given the correct input value, or ignores it otherwise.



Listing 5.3. Wrapper monad

```typescript
export class Wrapper {
    _value: any;
	constructor(value) {
		this._value = value;
    }

    static of(a) {
        return new Wrapper(a)
    }
    
	// map :: (A -> B) -> Wrapper(A) -> Wrapper(B)
	map (f) {
		return Wrapper.of(f(this._value));
	}

    join() {
        if (!(this._value instanceof Wrapper)) {
            return this
        } else {
            return this._value.join()
        }
    }

	toString() {
		return 'Wrapper (' + this._value + ')';
	}
}
```

hello monad:

```typescript
let hello = () => {
    let r =
        Wrapper.of('Hello Monads!')
            .map(R.toUpper)
            .map(R.identity); //-> Wrapper('HELLO MONADS!')
    console.log(r)
}
```

Listing 5.4. Flattening a monadic structure

```typescript
let flat = () => {
    let find = (db, id) => (<Array<any>>db).find(e => e.id === id)
    let DB_student = ['a', 'b', 'c'].map((address, id) => ({ address, id }))

    // findObject :: DB -> String -> Wrapper
    const findObject = R.curry(function (db, id) {
        return Wrapper.of(find(db, id));
    });

    // getAddress :: Student -> Wrapper
    const getAddress = function (student) {
        return Wrapper.of(student.map(R.prop('address')));
    }

    const studentAddress = R.compose(getAddress, findObject(DB_student));

    let res = studentAddress(1).join()//.get(); // Address
    console.log(res)
}
```

Listing 5.5. Maybe monad with subclasses Just and Nothing

