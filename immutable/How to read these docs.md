Immutable data encourages pure functions (data-in, data-out) and lends itself to much simpler application development and enabling techniques from functional programming such as lazy evaluation.

While designed to bring these powerful functional concepts to JavaScript, it presents an Object-Oriented API familiar to Javascript engineers and closely mirroring that of Array, Map, and Set. It is easy and efficient to convert to and from plain Javascript types.

## How to read these docs

In order to better explain what kinds of values the Immutable.js API expects and produces, this documentation is presented in a statically typed dialect of JavaScript (like [Flow](https://flowtype.org/) or [TypeScript](http://www.typescriptlang.org/)). You *don't need* to use these type checking tools in order to use Immutable.js, however becoming familiar with their syntax will help you get a deeper understanding of this API.

**A few examples and how to read them.**

All methods describe the kinds of data they accept and the kinds of data they return. For example a function which accepts two numbers and returns a number would look like this:

```typescript
sum(first: number, second: number): number
```

Sometimes, methods can accept different kinds of data or return different kinds of data, and this is described with a *type variable*, which is typically in all-caps. For example, a function which always returns the same kind of data it was provided would look like this:

```typescript
identity<T>(value: T): T
```

Type variables are defined with classes and referred to in methods. For example, a class that holds onto a value for you might look like this:

```ts
class Box<T> {
  constructor(value: T)
  getValue(): T
}
```

In order to manipulate Immutable data, methods that we're used to affecting a Collection instead return a new Collection of the same type. The type `this` refers to the same kind of class. For example, a List which returns new Lists when you `push` a value onto it might look like:

```ts
class List<T> {
  push(value: T): this
}
```

Many methods in Immutable.js accept values which implement the JavaScript [Collection](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Iteration_protocols) protocol, and might appear like `Collection` for something which represents sequence of strings. Typically in JavaScript we use plain Arrays (`[]`) when an Collection is expected, but also all of the Immutable.js collections are iterable themselves!

For example, to get a value deep within a structure of data, we might use `getIn` which expects an `Collection` path:

```ts
getIn(path: Collection<string | number>): any
```

To use this method, we could pass an array: `data.getIn([ "key", 2 ])`.

Note: All examples are presented in the modern [ES2015](https://developer.mozilla.org/en-US/docs/Web/JavaScript/New_in_JavaScript/ECMAScript_6_support_in_Mozilla) version of JavaScript. Use tools like Babel to support older browsers.

For example:

```ts
// ES2015
const mappedFoo = foo.map(x => x * x);
// ES5
var mappedFoo = foo.map(function (x) { return x * x; });
```

#### API

### [List](https://immutable-js.github.io/immutable-js/docs/#/List)

Lists are ordered indexed dense collections, much like a JavaScript Array.

### [Map](https://immutable-js.github.io/immutable-js/docs/#/Map)

Immutable Map is an unordered `Collection.Keyed` of (key, value) pairs with `O(log32 N)` gets and `O(log32 N)` persistent sets.

### [OrderedMap](https://immutable-js.github.io/immutable-js/docs/#/OrderedMap)

A type of Map that has the additional guarantee that the iteration order of entries will be the order in which they were set().

### [Set](https://immutable-js.github.io/immutable-js/docs/#/Set)

A Collection of unique values with `O(log32 N)` adds and has.

### [OrderedSet](https://immutable-js.github.io/immutable-js/docs/#/OrderedSet)

A type of Set that has the additional guarantee that the iteration order of values will be the order in which they were added.

### [Stack](https://immutable-js.github.io/immutable-js/docs/#/Stack)

Stacks are indexed collections which support very efficient O(1) addition and removal from the front using `unshift(v)` and `shift()`.

### [Range()](https://immutable-js.github.io/immutable-js/docs/#/Range)

Returns a Seq.Indexed of numbers from `start` (inclusive) to `end` (exclusive), by `step`, where `start` defaults to 0, `step` to 1, and `end` to infinity. When `start` is equal to `end`, returns empty range.

### [Repeat()](https://immutable-js.github.io/immutable-js/docs/#/Repeat)

Returns a Seq.Indexed of `value` repeated `times` times. When `times` is not defined, returns an infinite `Seq` of `value`.

### [Record](https://immutable-js.github.io/immutable-js/docs/#/Record)

A record is similar to a JS object, but enforces a specific set of allowed string keys, and has default values.

### [Seq](https://immutable-js.github.io/immutable-js/docs/#/Seq)

`Seq` describes a lazy operation, allowing them to efficiently chain use of all the higher-order collection methods (such as `map` and `filter`) by not creating intermediate collections.

### [Collection](https://immutable-js.github.io/immutable-js/docs/#/Collection)

The `Collection` is a set of (key, value) entries which can be iterated, and is the base class for all collections in `immutable`, allowing them to make use of all the Collection methods (such as `map` and `filter`).

### [ValueObject](https://immutable-js.github.io/immutable-js/docs/#/ValueObject)

### [fromJS()](https://immutable-js.github.io/immutable-js/docs/#/fromJS)

Deeply converts plain JS objects and arrays to Immutable Maps and Lists.

### [is()](https://immutable-js.github.io/immutable-js/docs/#/is)

Value equality check with semantics similar to `Object.is`, but treats Immutable Collections as values, equal if the second `Collection` includes equivalent values.

### [hash()](https://immutable-js.github.io/immutable-js/docs/#/hash)

The `hash()` function is an important part of how Immutable determines if two values are equivalent and is used to determine how to store those values. Provided with any value, `hash()` will return a 31-bit integer.

### [isImmutable()](https://immutable-js.github.io/immutable-js/docs/#/isImmutable)

True if `maybeImmutable` is an Immutable Collection or Record.

### [isCollection()](https://immutable-js.github.io/immutable-js/docs/#/isCollection)

True if `maybeCollection` is a Collection, or any of its subclasses.

### [isKeyed()](https://immutable-js.github.io/immutable-js/docs/#/isKeyed)

True if `maybeKeyed` is a Collection.Keyed, or any of its subclasses.

### [isIndexed()](https://immutable-js.github.io/immutable-js/docs/#/isIndexed)

True if `maybeIndexed` is a Collection.Indexed, or any of its subclasses.

### [isAssociative()](https://immutable-js.github.io/immutable-js/docs/#/isAssociative)

True if `maybeAssociative` is either a Keyed or Indexed Collection.

### [isOrdered()](https://immutable-js.github.io/immutable-js/docs/#/isOrdered)

True if `maybeOrdered` is a Collection where iteration order is well defined. True for Collection.Indexed as well as OrderedMap and OrderedSet.

### [isValueObject()](https://immutable-js.github.io/immutable-js/docs/#/isValueObject)

True if `maybeValue` is a JavaScript Object which has *both* `equals()` and `hashCode()` methods.

### [isSeq()](https://immutable-js.github.io/immutable-js/docs/#/isSeq)

True if `maybeSeq` is a Seq.

### [isList()](https://immutable-js.github.io/immutable-js/docs/#/isList)

True if `maybeList` is a List.

### [isMap()](https://immutable-js.github.io/immutable-js/docs/#/isMap)

True if `maybeMap` is a Map.

### [isOrderedMap()](https://immutable-js.github.io/immutable-js/docs/#/isOrderedMap)

True if `maybeOrderedMap` is an OrderedMap.

### [isStack()](https://immutable-js.github.io/immutable-js/docs/#/isStack)

True if `maybeStack` is a Stack.

### [isSet()](https://immutable-js.github.io/immutable-js/docs/#/isSet)

True if `maybeSet` is a Set.

### [isOrderedSet()](https://immutable-js.github.io/immutable-js/docs/#/isOrderedSet)

True if `maybeOrderedSet` is an OrderedSet.

### [isRecord()](https://immutable-js.github.io/immutable-js/docs/#/isRecord)

True if `maybeRecord` is a Record.

### [get()](https://immutable-js.github.io/immutable-js/docs/#/get)

Returns the value within the provided collection associated with the provided key, or notSetValue if the key is not defined in the collection.

### [has()](https://immutable-js.github.io/immutable-js/docs/#/has)

Returns true if the key is defined in the provided collection.

### [remove()](https://immutable-js.github.io/immutable-js/docs/#/remove)

Returns a copy of the collection with the value at key removed.

### [set()](https://immutable-js.github.io/immutable-js/docs/#/set)

Returns a copy of the collection with the value at key set to the provided value.

### [update()](https://immutable-js.github.io/immutable-js/docs/#/update)

Returns a copy of the collection with the value at key set to the result of providing the existing value to the updating function.

### [getIn()](https://immutable-js.github.io/immutable-js/docs/#/getIn)

Returns the value at the provided key path starting at the provided collection, or notSetValue if the key path is not defined.

### [hasIn()](https://immutable-js.github.io/immutable-js/docs/#/hasIn)

Returns true if the key path is defined in the provided collection.

### [removeIn()](https://immutable-js.github.io/immutable-js/docs/#/removeIn)

Returns a copy of the collection with the value at the key path removed.

### [setIn()](https://immutable-js.github.io/immutable-js/docs/#/setIn)

Returns a copy of the collection with the value at the key path set to the provided value.

### [updateIn()](https://immutable-js.github.io/immutable-js/docs/#/updateIn)

Returns a copy of the collection with the value at key path set to the result of providing the existing value to the updating function.

### [merge()](https://immutable-js.github.io/immutable-js/docs/#/merge)

Returns a copy of the collection with the remaining collections merged in.

### [mergeWith()](https://immutable-js.github.io/immutable-js/docs/#/mergeWith)

Returns a copy of the collection with the remaining collections merged in, calling the `merger` function whenever an existing value is encountered.

### [mergeDeep()](https://immutable-js.github.io/immutable-js/docs/#/mergeDeep)

Returns a copy of the collection with the remaining collections merged in deeply (recursively).

### [mergeDeepWith()](https://immutable-js.github.io/immutable-js/docs/#/mergeDeepWith)

Returns a copy of the collection with the remaining collections merged in deeply (recursively), calling the `merger` function whenever an existing value is encountered.