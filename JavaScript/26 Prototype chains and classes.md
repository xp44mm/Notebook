## 26 Prototype chains and classes


In this book, JavaScript’s style of object-oriented programming (OOP) is introduced in four steps. This chapter covers steps 2–4, [the previous chapter](https://exploringjs.com/impatient-js/ch_single-objects.html) covers step 1. The steps are (fig. [9](https://exploringjs.com/impatient-js/ch_proto-chains-classes.html#fig:oop_steps2)):

1. **Single objects (previous chapter):** How do *objects*, JavaScript’s basic OOP building blocks, work in isolation?
2. **Prototype chains (this chapter):** Each object has a chain of zero or more *prototype objects*. Prototypes are JavaScript’s core inheritance mechanism.
3. **Classes (this chapter):** JavaScript’s *classes* are factories for objects. The relationship between a class and its instances is based on prototypal inheritance.
4. **Subclassing (this chapter):** The relationship between a *subclass* and its *superclass* is also based on prototypal inheritance.

![img](https://exploringjs.com/impatient-js/img-book/61838ddd8244a11d32162fc6ac2d6045aa51b0e0.svg)

Figure 9: This book introduces object-oriented programming in JavaScript in four steps.

### 26.1 Prototype chains



Prototypes are JavaScript’s only inheritance mechanism: each object has a prototype that is either `null` or an object. In the latter case, the object inherits all of the prototype’s properties.

In an object literal, you can set the prototype via the special property `__proto__`:

```js
const proto = {
  protoProp: 'a',
};
const obj = {
  __proto__: proto,
  objProp: 'b',
};

// obj inherits .protoProp:
assert.equal(obj.protoProp, 'a');
assert.equal('protoProp' in obj, true);
```

Given that a prototype object can have a prototype itself, we get a chain of objects – the so-called *prototype chain*. That means that inheritance gives us the impression that we are dealing with single objects, but we are actually dealing with chains of objects.

Fig. 10 shows what the prototype chain of `obj` looks like.

![img](https://exploringjs.com/impatient-js/img-book/801d9c61c17c4bfc489ce98e15dbb7495fb8479f.svg)

Figure 10: `obj` starts a chain of objects that continues with `proto` and other objects.

Non-inherited properties are called *own properties*. `obj` has one own property, `.objProp`.

#### 26.1.1 JavaScript’s operations: all properties vs. own properties

Some operations consider all properties (own and inherited) – for example, getting properties:

```js
> const obj = { foo: 1 };
> typeof obj.foo // own
'number'
> typeof obj.toString // inherited
'function'
```

Other operations only consider own properties – for example, `Object.entries()`, `Object.values()`, `Object.keys()`:

```js
> Object.keys(obj)
[ 'foo' ]
```

Read on for another operation that also only considers own properties: setting properties.

#### 26.1.2 Pitfall: only the first member of a prototype chain is mutated

One aspect of prototype chains that may be counter-intuitive is that setting *any* property via an object – even an inherited one – only changes that very object – never one of the prototypes.

Consider the following object `obj`:

```js
const proto = {
  protoProp: 'a',
};
const obj = {
  __proto__: proto,
  objProp: 'b',
};
```

In the next code snippet, we set the inherited property `obj.protoProp` (line A). That “changes” it by creating an own property: When reading `obj.protoProp`, the own property is found first and its value *overrides* the value of the inherited property.

```js
// In the beginning, obj has one own property
assert.deepEqual(Object.keys(obj), ['objProp']);

obj.protoProp = 'x'; // (A)

// We created a new own property:
assert.deepEqual(Object.keys(obj), ['objProp', 'protoProp']);

// The inherited property itself is unchanged:
assert.equal(proto.protoProp, 'a');

// The own property overrides the inherited property:
assert.equal(obj.protoProp, 'x');
```

The prototype chain of `obj` is depicted in fig. 11.

![img](https://exploringjs.com/impatient-js/img-book/3e658ec0b4c85ee734eb8fce32c26fe19b234cf7.svg)

Figure 11: The own property `.protoProp` of `obj` overrides the property inherited from `proto`.

#### 26.1.3 Tips for working with prototypes (advanced)

##### 26.1.3.1 Best practice: avoid `__proto__`, except in object literals

I recommend to avoid the pseudo-property `__proto__`: As we will see later, not all objects have it.

However, `__proto__` in object literals is different. There, it is a built-in feature and always available.

The recommended ways of getting and setting prototypes are:

- The best way to get a prototype is via the following method:

  ```js
  Object.getPrototypeOf(obj: Object) : Object
  ```

- The best way to set a prototype is when creating an object – via `__proto__` in an object literal or via:

  ```js
  Object.create(proto: Object) : Object
  ```

  If you have to, you can use `Object.setPrototypeOf()` to change the prototype of an existing object. But that may affect performance negatively.

This is how these features are used:

```js
const proto1 = {};
const proto2 = {};

const obj = Object.create(proto1);
assert.equal(Object.getPrototypeOf(obj), proto1);

Object.setPrototypeOf(obj, proto2);
assert.equal(Object.getPrototypeOf(obj), proto2);
```

##### 26.1.3.2 Check: is an object a prototype of another one?

So far, “`p` is a prototype of `o`” always meant “`p` is a *direct* prototype of `o`”. But it can also be used more loosely and mean that `p` is in the prototype chain of `o`. That looser relationship can be checked via:

```js
p.isPrototypeOf(o)
```

For example:

```js
const a = {};
const b = {__proto__: a};
const c = {__proto__: b};

assert.equal(a.isPrototypeOf(b), true);
assert.equal(a.isPrototypeOf(c), true);

assert.equal(a.isPrototypeOf(a), false);
assert.equal(c.isPrototypeOf(a), false);
```

#### 26.1.4 Sharing data via prototypes

Consider the following code:

```js
const jane = {
  name: 'Jane',
  describe() {
    return 'Person named '+this.name;
  },
};
const tarzan = {
  name: 'Tarzan',
  describe() {
    return 'Person named '+this.name;
  },
};

assert.equal(jane.describe(), 'Person named Jane');
assert.equal(tarzan.describe(), 'Person named Tarzan');
```

We have two objects that are very similar. Both have two properties whose names are `.name` and `.describe`. Additionally, method `.describe()` is the same. How can we avoid duplicating that method?

We can move it to an object `PersonProto` and make that object a prototype of both `jane` and `tarzan`:

```js
const PersonProto = {
  describe() {
    return 'Person named ' + this.name;
  },
};
const jane = {
  __proto__: PersonProto,
  name: 'Jane',
};
const tarzan = {
  __proto__: PersonProto,
  name: 'Tarzan',
};
```

The name of the prototype reflects that both `jane` and `tarzan` are persons.

![img](https://exploringjs.com/impatient-js/img-book/c58c226961223c1004c8bacd1764c0b5990b33d4.svg)

Figure 12: Objects `jane` and `tarzan` share method `.describe()`, via their common prototype `PersonProto`.

Fig. 12 illustrates how the three objects are connected: The objects at the bottom now contain the properties that are specific to `jane` and `tarzan`. The object at the top contains the properties that are shared between them.

When you make the method call `jane.describe()`, `this` points to the receiver of that method call, `jane` (in the bottom-left corner of the diagram). That’s why the method still works. `tarzan.describe()` works similarly.

```js
assert.equal(jane.describe(), 'Person named Jane');
assert.equal(tarzan.describe(), 'Person named Tarzan');
```

### 26.2 Classes

We are now ready to take on classes, which are basically a compact syntax for setting up prototype chains. Under the hood, JavaScript’s classes are unconventional. But that is something you rarely see when working with them. They should normally feel familiar to people who have used other object-oriented programming languages.

#### 26.2.1 A class for persons

We have previously worked with `jane` and `tarzan`, single objects representing persons. Let’s use a *class declaration* to implement a factory for person objects:

```js
class Person {
  constructor(name) {
    this.name = name;
  }
  describe() {
    return 'Person named '+this.name;
  }
}
```

`jane` and `tarzan` can now be created via `new Person()`:

```js
const jane = new Person('Jane');
assert.equal(jane.name, 'Jane');
assert.equal(jane.describe(), 'Person named Jane');

const tarzan = new Person('Tarzan');
assert.equal(tarzan.name, 'Tarzan');
assert.equal(tarzan.describe(), 'Person named Tarzan');
```

Class `Person` has two methods:

- The normal method `.describe()`
- The special method `.constructor()` which is called directly after a new instance has been created and initializes that instance. It receives the arguments that are passed to the `new` operator (after the class name). If you don’t need any arguments to set up a new instance, you can omit the constructor.

##### 26.2.1.1 Class expressions

There are two kinds of *class definitions* (ways of defining classes):

- *Class declarations*, which we have seen in the previous section.
- *Class expressions*, which we’ll see next.

Class expressions can be anonymous and named:

```js
// Anonymous class expression
const Person = class { ··· };

// Named class expression
const Person = class MyClass { ··· };
```

The name of a named class expression works similarly to the name of a named function expression.

This was a first look at classes. We’ll explore more features soon, but first we need to learn the internals of classes.

#### 26.2.2 Classes under the hood

There is a lot going on under the hood of classes. Let’s look at the diagram for `jane` (fig. 13).

![img](https://exploringjs.com/impatient-js/img-book/9e58a0eff1bb377a79c6c288675ac37ece33af23.svg)

Figure 13: The class `Person` has the property `.prototype` that points to an object that is the prototype of all instances of `Person`. `jane` is one such instance.

The main purpose of class `Person` is to set up the prototype chain on the right (`jane`, followed by `Person.prototype`). It is interesting to note that both constructs inside class `Person` (`.constructor` and `.describe()`) created properties for `Person.prototype`, not for `Person`.

The reason for this slightly odd approach is backward compatibility: prior to classes, *constructor functions* (ordinary functions, invoked via the `new` operator) were often used as factories for objects. Classes are mostly better syntax for constructor functions and therefore remain compatible with old code. That explains why classes are functions:

```js
> typeof Person
'function'
```

In this book, I use the terms *constructor (function)* and *class* interchangeably.

It is easy to confuse `.__proto__` and `.prototype`. Hopefully, fig. 13 makes it clear how they differ:

- `.__proto__` is a pseudo-property for accessing the prototype of an object.
- `.prototype` is a normal property that is only special due to how the `new` operator uses it. The name is not ideal: `Person.prototype` does not point to the prototype of `Person`, it points to the prototype of all instances of `Person`.

##### 26.2.2.1 `Person.prototype.constructor` (advanced)

There is one detail in fig. 13 that we haven’t looked at, yet: `Person.prototype.constructor` points back to `Person`:

```js
> Person.prototype.constructor === Person
true
```

This setup also exists due to backward compatibility. But it has two additional benefits.

First, each instance of a class inherits property `.constructor`. Therefore, given an instance, you can make “similar” objects using it:

```diff
const jane = new Person('Jane');

+ const cheeta = new jane.constructor('Cheeta');
// cheeta is also an instance of Person
// (the instanceof operator is explained later)
assert.equal(cheeta instanceof Person, true);
```

Second, you can get the name of the class that created a given instance:

```js
const tarzan = new Person('Tarzan');

assert.equal(tarzan.constructor.name, 'Person');
```

#### 26.2.3 Class definitions: prototype properties

All constructs in the body of the following class declaration create properties of `Foo.prototype`.

```js
class Foo {
  constructor(prop) {
    this.prop = prop;
  }
  protoMethod() {
    return 'protoMethod';
  }
  get protoGetter() {
    return 'protoGetter';
  }
}
```

Let’s examine them in order:

- `.constructor()` is called after creating a new instance of `Foo` to set up that instance.
- `.protoMethod()` is a normal method. It is stored in `Foo.prototype`.
- `.protoGetter` is a getter that is stored in `Foo.prototype`.

The following interaction uses class `Foo`:

```js
> const foo = new Foo(123);
> foo.prop
123
> foo.protoMethod()
'protoMethod'
> foo.protoGetter
'protoGetter'
```

#### 26.2.4 Class definitions: static properties

All constructs in the body of the following class declaration create so-called *static* properties – properties of `Bar` itself.

```js
class Bar {
  static staticMethod() {
    return 'staticMethod';
  }
  static get staticGetter() {
    return 'staticGetter';
  }
}
```

The static method and the static getter are used as follows:

```js
> Bar.staticMethod()
'staticMethod'
> Bar.staticGetter
'staticGetter'
```

#### 26.2.5 The `instanceof` operator

The `instanceof` operator tells you if a value is an instance of a given class:

```js
> new Person('Jane') instanceof Person
true
> ({}) instanceof Person
false
> ({}) instanceof Object
true
> [] instanceof Array
true
```

We’ll explore the `instanceof` operator in more detail later, after we have looked at subclassing.

#### 26.2.6 Why I recommend classes

I recommend using classes for the following reasons:

- Classes are a common standard for object creation and inheritance that is now widely supported across frameworks (React, Angular, Ember, etc.). This is an improvement to how things were before, when almost every framework had its own inheritance library.
- They help tools such as IDEs and type checkers with their work and enable new features there.
- If you come from another language to JavaScript and are used to classes, then you can get started more quickly.
- JavaScript engines optimize them. That is, code that uses classes is almost always faster than code that uses a custom inheritance library.
- You can subclass built-in constructor functions such as `Error`.

That doesn’t mean that classes are perfect:

- There is a risk of overdoing inheritance.

- There is a risk of putting too much functionality in classes (when some of it is often better put in functions).

- How they work superficially and under the hood is quite different. In other words, there is a disconnect between syntax and semantics. Two examples are:

  - A method definition inside a class `C` creates a method in the object `C.prototype`.
  - Classes are functions.

  The motivation for the disconnect is backward compatibility. Thankfully, the disconnect causes few problems in practice; you are usually OK if you go along with what classes pretend to be.


### 26.3 Private data for classes

This section describes techniques for hiding some of the data of an object from the outside. We discuss them in the context of classes, but they also work for objects created directly, e.g., via object literals.

#### 26.3.1 Private data: naming convention

The first technique makes a property private by prefixing its name with an underscore. This doesn’t protect the property in any way; it merely signals to the outside: “You don’t need to know about this property.”

In the following code, the properties `._counter` and `._action` are private.

```js
class Countdown {
  constructor(counter, action) {
    this._counter = counter;
    this._action = action;
  }
  dec() {
    this._counter--;
    if (this._counter === 0) {
      this._action();
    }
  }
}

// The two properties aren’t really private:
assert.deepEqual(
  Object.keys(new Countdown()),
  ['_counter', '_action']);
```

With this technique, you don’t get any protection and private names can clash. On the plus side, it is easy to use.

#### 26.3.2 Private data: WeakMaps

Another technique is to use WeakMaps. How exactly that works is explained in the chapter on WeakMaps. This is a preview:

```js
const _counter = new WeakMap();
const _action = new WeakMap();

class Countdown {
  constructor(counter, action) {
    _counter.set(this, counter);
    _action.set(this, action);
  }
  dec() {
    let counter = _counter.get(this);
    counter--;
    _counter.set(this, counter);
    if (counter === 0) {
      _action.get(this)();
    }
  }
}

// The two pseudo-properties are truly private:
assert.deepEqual(
  Object.keys(new Countdown()),
  []);
```

This technique offers you considerable protection from outside access and there can’t be any name clashes. But it is also more complicated to use.

#### 26.3.3 More techniques for private data

This book explains the most important techniques for private data in classes. There will also probably soon be built-in support for it. Consult the ECMAScript proposal “Class Public Instance Fields & Private Instance Fields” for details.

A few additional techniques are explained in [*Exploring ES6*](https://exploringjs.com/es6/ch_classes.html#sec_private-data-for-classes).

### 26.4 Subclassing

Classes can also subclass (“extend”) existing classes. As an example, the following class `Employee` subclasses `Person`:

```js
class Person {
  constructor(name) {
    this.name = name;
  }
  describe() {
    return `Person named ${this.name}`;
  }
  static logNames(persons) {
    for (const person of persons) {
      console.log(person.name);
    }
  }
}

class Employee extends Person {
  constructor(name, title) {
    super(name);
    this.title = title;
  }
  describe() {
    return super.describe() +
      ` (${this.title})`;
  }
}

const jane = new Employee('Jane', 'CTO');
assert.equal(
  jane.describe(),
  'Person named Jane (CTO)');
```

Two comments:

- Inside a `.constructor()` method, you must call the super-constructor via `super()` before you can access `this`. That’s because `this` doesn’t exist before the super-constructor is called (this phenomenon is specific to classes).

- Static methods are also inherited. For example, `Employee` inherits the static method `.logNames()`:

  ```js
  > 'logNames' in Employee
  true
  ```


#### 26.4.1 Subclasses under the hood (advanced)

![img](https://exploringjs.com/impatient-js/img-book/8b048ae6d82e1a091f97d0b99007994316fae790.svg)

Figure 14: These are the objects that make up class `Person` and its subclass, `Employee`. The left column is about classes. The right column is about the `Employee` instance `jane` and its prototype chain.

The classes `Person` and `Employee` from the previous section are made up of several objects (fig. 14). One key insight for understanding how these objects are related is that there are two prototype chains:

- The instance prototype chain, on the right.
- The class prototype chain, on the left.

##### 26.4.1.1 The instance prototype chain (right column)

The instance prototype chain starts with `jane` and continues with `Employee.prototype` and `Person.prototype`. In principle, the prototype chain ends at this point, but we get one more object: `Object.prototype`. This prototype provides services to virtually all objects, which is why it is included here, too:

```js
> Object.getPrototypeOf(Person.prototype) === Object.prototype
true
```

##### 26.4.1.2 The class prototype chain (left column)

In the class prototype chain, `Employee` comes first, `Person` next. Afterward, the chain continues with `Function.prototype`, which is only there because `Person` is a function and functions need the services of `Function.prototype`.

```js
> Object.getPrototypeOf(Person) === Function.prototype
true
```

#### 26.4.2 `instanceof` in more detail (advanced)

We have not yet seen how `instanceof` really works. Given the expression:

```js
x instanceof C
```

How does `instanceof` determine if `x` is an instance of `C` (or a subclass of `C`)? It does so by checking if `C.prototype` is in the prototype chain of `x`. That is, the following expression is equivalent:

```js
C.prototype.isPrototypeOf(x)
```

If we go back to fig. 14, we can confirm that the prototype chain does lead us to the following correct answers:

```js
> jane instanceof Employee
true
> jane instanceof Person
true
> jane instanceof Object
true
```

#### 26.4.3 Prototype chains of built-in objects (advanced)

Next, we’ll use our knowledge of subclassing to understand the prototype chains of a few built-in objects. The following tool function `p()` helps us with our explorations.

```js
const p = Object.getPrototypeOf.bind(Object);
```

We extracted method `.getPrototypeOf()` of `Object` and assigned it to `p`.

##### 26.4.3.1 The prototype chain of `{}`

Let’s start by examining plain objects:

```js
> p({}) === Object.prototype
true
> p(p({})) === null
true
```

![img](https://exploringjs.com/impatient-js/img-book/c65e3ba5d1d2b585da95f3ea9ae745634c29ead2.svg)

Figure 15: The prototype chain of an object created via an object literal starts with that object, continues with `Object.prototype`, and ends with `null`.

Fig. 15 shows a diagram for this prototype chain. We can see that `{}` really is an instance of `Object` – `Object.prototype` is in its prototype chain.

##### 26.4.3.2 The prototype chain of `[]`

What does the prototype chain of an Array look like?

```js
> p([]) === Array.prototype
true
> p(p([])) === Object.prototype
true
> p(p(p([]))) === null
true
```

![img](https://exploringjs.com/impatient-js/img-book/1f863fe570fc887e785e0349b5bc42de4ef17b4d.svg)

Figure 16: The prototype chain of an Array has these members: the Array instance, `Array.prototype`, `Object.prototype`, `null`.

This prototype chain (visualized in fig. 16) tells us that an Array object is an instance of `Array`, which is a subclass of `Object`.

##### 26.4.3.3 The prototype chain of `function () {}`

Lastly, the prototype chain of an ordinary function tells us that all functions are objects:

```js
> p(function () {}) === Function.prototype
true
> p(p(function () {})) === Object.prototype
true
```

##### 26.4.3.4 Objects that aren’t instances of `Object`

An object is only an instance of `Object` if `Object.prototype` is in its prototype chain. Most objects created via various literals are instances of `Object`:

```js
> ({}) instanceof Object
true
> (() => {}) instanceof Object
true
> /abc/ug instanceof Object
true
```

Objects that don’t have prototypes are not instances of `Object`:

```js
> ({ __proto__: null }) instanceof Object
false
```

`Object.prototype` ends most prototype chains. Its prototype is `null`, which means it isn’t an instance of `Object` either:

```js
> Object.prototype instanceof Object
false
```

##### 26.4.3.5 How exactly does the pseudo-property `.__proto__` work?

The pseudo-property `.__proto__` is implemented by class `Object` via a getter and a setter. It could be implemented like this:

```js
class Object {
  get __proto__() {
    return Object.getPrototypeOf(this);
  }
  set __proto__(other) {
    Object.setPrototypeOf(this, other);
  }
  // ···
}
```

That means that you can switch `.__proto__` off by creating an object that doesn’t have `Object.prototype` in its prototype chain (see the previous section):

```js
> '__proto__' in {}
true
> '__proto__' in { __proto__: null }
false
```

#### 26.4.4 Dispatched vs. direct method calls (advanced)

Let’s examine how method calls work with classes. We are revisiting `jane` from earlier:

```js
class Person {
  constructor(name) {
    this.name = name;
  }
  describe() {
    return 'Person named '+this.name;
  }
}
const jane = new Person('Jane');
```

Fig. [17](https://exploringjs.com/impatient-js/ch_proto-chains-classes.html#fig:jane_proto_chain) has a diagram with `jane`’s prototype chain.

![img](https://exploringjs.com/impatient-js/img-book/012ec88adc43f494e1b313cdb75f875435fc9cf8.svg)

Figure 17: The prototype chain of `jane` starts with `jane` and continues with `Person.prototype`.

Normal method calls are *dispatched* – the method call `jane.describe()` happens in two steps:

- Dispatch: In the prototype chain of `jane`, find the first property whose key is `'describe'` and retrieve its value.

  ```js
  const func = jane.describe;
  ```

- Call: Call the value, while setting `this` to `jane`.

  ```
  func.call(jane);
  ```

This way of dynamically looking for a method and invoking it is called *dynamic dispatch*.

You can make the same method call *directly*, without dispatching:

```js
Person.prototype.describe.call(jane)
```

This time, we directly point to the method via `Person.prototype.describe` and don’t search for it in the prototype chain. We also specify `this` differently via `.call()`.

Note that `this` always points to the beginning of a prototype chain. That enables `.describe()` to access `.name`.

##### 26.4.4.1 Borrowing methods

Direct method calls become useful when you are working with methods of `Object.prototype`. For example, `Object.prototype.hasOwnProperty(k)` checks if `this` has a non-inherited property whose key is `k`:

```js
> const obj = { foo: 123 };
> obj.hasOwnProperty('foo')
true
> obj.hasOwnProperty('bar')
false
```

However, in the prototype chain of an object, there may be another property with the key `'hasOwnProperty'` that overrides the method in `Object.prototype`. Then a dispatched method call doesn’t work:

```js
> const obj = { hasOwnProperty: true };
> obj.hasOwnProperty('bar')
TypeError: obj.hasOwnProperty is not a function
```

The workaround is to use a direct method call:

```js
> Object.prototype.hasOwnProperty.call(obj, 'bar')
false
> Object.prototype.hasOwnProperty.call(obj, 'hasOwnProperty')
true
```

This kind of direct method call is often abbreviated as follows:

```js
> ({}).hasOwnProperty.call(obj, 'bar')
false
> ({}).hasOwnProperty.call(obj, 'hasOwnProperty')
true
```

This pattern may seem inefficient, but most engines optimize this pattern, so performance should not be an issue.

#### 26.4.5 Mixin classes (advanced)

JavaScript’s class system only supports *single inheritance*. That is, each class can have at most one superclass. One way around this limitation is via a technique called *mixin classes* (short: *mixins*).

The idea is as follows: Let’s say we want a class `C` to inherit from two super classes `S1` and `S2`. That would be *multiple inheritance*, which JavaScript doesn’t support.

Our workaround is to turn `S1` and `S2` into *mixins*, factories for subclasses:

```js
const S1 = (Sup) => class extends Sup { /*···*/ };
const S2 = (Sup) => class extends Sup { /*···*/ };
```

Each of these two functions returns a class that extends a given superclass `Sup`. We create class `C` as follows:

```js
class C extends S2(S1(Object)) {
  /*···*/
}
```

We now have a class `C` that extends a class `S2` that extends a class `S1` that extends `Object` (which most classes do implicitly).

##### 26.4.5.1 Example: a mixin for brand management

We implement a mixin `Branded` that has helper methods for setting and getting the brand of an object:

```js
const Branded = (Sup) => class extends Sup {
  setBrand(brand) {
    this._brand = brand;
    return this;
  }
  getBrand() {
    return this._brand;
  }
};
```

We use this mixin to implement brand management for a class `Car`:

```js
class Car extends Branded(Object) {
  constructor(model) {
    super();
    this._model = model;
  }
  toString() {
    return `${this.getBrand()} ${this._model}`;
  }
}
```

The following code confirms that the mixin worked: `Car` has method `.setBrand()` of `Branded`.

```js
const modelT = new Car('Model T').setBrand('Ford');
assert.equal(modelT.toString(), 'Ford Model T');
```

##### 26.4.5.2 The benefits of mixins

Mixins free us from the constraints of single inheritance:

- The same class can extend a single superclass and zero or more mixins.
- The same mixin can be used by multiple classes.

### 26.5 FAQ: objects

#### 26.5.1 Why do objects preserve the insertion order of properties?

In principle, objects are unordered. The main reason for ordering properties is so that operations that list entries, keys, or values are deterministic. That helps, e.g., with testing.
