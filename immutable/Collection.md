# Collection

The `Collection` is a set of (key, value) entries which can be iterated, and is the base class for all collections in `immutable`, allowing them to make use of all the Collection methods (such as `map` and `filter`).

```js
type Collection extends ValueObject
```

#### DISCUSSION

Note: A collection is always iterated in the same order, however that order may not always be well defined, as is the case for the `Map` and `Set`.

Collection is the abstract base class for concrete data structures. It cannot be constructed directly.

Implementations should extend one of the subclasses, `Collection.Keyed`, `Collection.Indexed`, or `Collection.Set`.

## Sub-types

[Collection.Keyed](https://immutable-js.github.io/immutable-js/docs/#/Collection.Keyed)

[Collection.Indexed](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed)

[Collection.Set](https://immutable-js.github.io/immutable-js/docs/#/Collection.Set)

## Construction

### [Collection()](https://immutable-js.github.io/immutable-js/docs/#/Collection/Collection)

Creates a Collection.

```ts
Collection<I>(collection: I): I
Collection<T>(collection: Collection<T>): Collection.Indexed<T>
Collection<V>(obj: {[key: string]: V}): tsCollection.Keyed<string, V>
```

#### DISCUSSION

The type of Collection created is based on the input.

- If an `Collection`, that same `Collection`.
- If an Array-like, an `Collection.Indexed`.
- If an Object with an Iterator defined, an `Collection.Indexed`.
- If an Object, an `Collection.Keyed`.

This methods forces the conversion of Objects and Strings to Collections. If you want to ensure that a Collection of one item is returned, use `Seq.of`.

Note: An Iterator itself will be treated as an object, becoming a `Seq.Keyed`, which is usually not what you want. You should turn your Iterator Object into an iterable object by defining a `Symbol.iterator` (or @@iterator) method which returns `this`.

Note: `Collection` is a conversion function and not a class, and does not use the `new` keyword during construction.

## Value equality

### [equals()](https://immutable-js.github.io/immutable-js/docs/#/Collection/equals)

#### OVERRIDES

```
ValueObject#equals
```


### [hashCode()](https://immutable-js.github.io/immutable-js/docs/#/Collection/hashCode)


#### OVERRIDES

```
ValueObject#hashCode
```

## Reading values

### [get()](https://immutable-js.github.io/immutable-js/docs/#/Collection/get)

Returns the value associated with the provided key, or notSetValue if the Collection does not contain this key.

```ts
get<NSV>(key: K, notSetValue: NSV): V | NSV
get(key: K): V | undefined
```

#### DISCUSSION

Note: it is possible a key may be associated with an `undefined` value, so if `notSetValue` is not provided and this method returns `undefined`, that does not guarantee the key was not found.

### [has()](https://immutable-js.github.io/immutable-js/docs/#/Collection/has)

True if a key exists within this `Collection`, using `Immutable.is` to determine equality

```ts
has(key: K): boolean
```

### [includes()](https://immutable-js.github.io/immutable-js/docs/#/Collection/includes)

True if a value exists within this `Collection`, using `Immutable.is` to determine equality

```ts
includes(value: V): boolean 
```

#### ALIAS

```ts
contains()
```

### [first()](https://immutable-js.github.io/immutable-js/docs/#/Collection/first)

In case the `Collection` is not empty returns the first element of the `Collection`. In case the `Collection` is empty returns the optional default value if provided, if no default value is provided returns undefined.

```ts
first<NSV>(notSetValue?: NSV): V | NSV
```

### [last()](https://immutable-js.github.io/immutable-js/docs/#/Collection/last)

In case the `Collection` is not empty returns the last element of the `Collection`. In case the `Collection` is empty returns the optional default value if provided, if no default value is provided returns undefined.

```ts
last<NSV>(notSetValue?: NSV): V | NSV
```

## Reading deep values

### [getIn()](https://immutable-js.github.io/immutable-js/docs/#/Collection/getIn)

Returns the value found by following a path of keys or indices through nested Collections.

```ts
getIn(searchKeyPath: Collection<any>, notSetValue?: any): any
```

#### DISCUSSION

```ts
const { Map, List } = require('immutable')
const deepData = Map({ x: List([ Map({ y: 123 }) ]) });
deepData.getIn(['x', 0, 'y']) // 123
```

Plain JavaScript Object or Arrays may be nested within an Immutable.js Collection, and `getIn()` can access those values as well:

```ts
const { Map, List } = require('immutable')
const deepData = Map({ x: [ { y: 123 } ] });
deepData.getIn(['x', 0, 'y']) // 123
```

### [hasIn()](https://immutable-js.github.io/immutable-js/docs/#/Collection/hasIn)

True if the result of following a path of keys or indices through nested Collections results in a set value.

```
hasIn(searchKeyPath: Collection<any>): boolean
```

## Persistent changes

### [update()](https://immutable-js.github.io/immutable-js/docs/#/Collection/update)

This can be very useful as a way to "chain" a normal function into a sequence of methods. RxJS calls this "let" and lodash calls it "thru".

```
update<R>(updater: (collection: this) => R): R
```

#### DISCUSSION

For example, to sum a Seq after mapping and filtering:

```ts
const { Seq } = require('immutable')

function sum(collection) {
  return collection.reduce((sum, x) => sum + x, 0)
}

Seq([ 1, 2, 3 ])
  .map(x => x + 1)
  .filter(x => x % 2 === 0)
  .update(sum)
// 6
```

## Conversion to JavaScript types

### [toJS()](https://immutable-js.github.io/immutable-js/docs/#/Collection/toJS)

Deeply converts this Collection to equivalent native JavaScript Array or Object.

```ts
toJS(): Array | {[key: string]: any} 
```

#### DISCUSSION

`Collection.Indexed`, and `Collection.Set` become `Array`, while `Collection.Keyed` become `Object`, converting keys to Strings.

### [toJSON()](https://immutable-js.github.io/immutable-js/docs/#/Collection/toJSON)

Shallowly converts this Collection to equivalent native JavaScript Array or Object.

```
toJSON(): Array | {[key: string]: V} 
```

#### DISCUSSION

`Collection.Indexed`, and `Collection.Set` become `Array`, while `Collection.Keyed` become `Object`, converting keys to Strings.

### [toArray()](https://immutable-js.github.io/immutable-js/docs/#/Collection/toArray)

Shallowly converts this collection to an Array.

```
toArray(): Array | Array<[K, V]> 
```

#### DISCUSSION

`Collection.Indexed`, and `Collection.Set` produce an Array of values. `Collection.Keyed` produce an Array of [key, value] tuples.

### [toObject()](https://immutable-js.github.io/immutable-js/docs/#/Collection/toObject)

Shallowly converts this Collection to an Object.

```
toObject(): {[key: string]: V} 
```

#### DISCUSSION

Converts keys to Strings.

## Conversion to Collections

### [toMap()](https://immutable-js.github.io/immutable-js/docs/#/Collection/toMap)

Converts this Collection to a Map, Throws if keys are not hashable.

```
toMap(): Map 
```

#### DISCUSSION

Note: This is equivalent to `Map(this.toKeyedSeq())`, but provided for convenience and to allow for chained expressions.

### [toOrderedMap()](https://immutable-js.github.io/immutable-js/docs/#/Collection/toOrderedMap)

Converts this Collection to a Map, maintaining the order of iteration.

```
toOrderedMap(): OrderedMap 
```

#### DISCUSSION

Note: This is equivalent to `OrderedMap(this.toKeyedSeq())`, but provided for convenience and to allow for chained expressions.

### [toSet()](https://immutable-js.github.io/immutable-js/docs/#/Collection/toSet)

Converts this Collection to a Set, discarding keys. Throws if values are not hashable.

```
toSet(): Set 
```

#### DISCUSSION

Note: This is equivalent to `Set(this)`, but provided to allow for chained expressions.

### [toOrderedSet()](https://immutable-js.github.io/immutable-js/docs/#/Collection/toOrderedSet)

Converts this Collection to a Set, maintaining the order of iteration and discarding keys.

```
toOrderedSet(): OrderedSet 
```

#### DISCUSSION

Note: This is equivalent to `OrderedSet(this.valueSeq())`, but provided for convenience and to allow for chained expressions.

### [toList()](https://immutable-js.github.io/immutable-js/docs/#/Collection/toList)

Converts this Collection to a List, discarding keys.

```
toList(): List 
```

#### DISCUSSION

This is similar to `List(collection)`, but provided to allow for chained expressions. However, when called on `Map` or other keyed collections, `collection.toList()` discards the keys and creates a list of only the values, whereas `List(collection)` creates a list of entry tuples.

```js
const { Map, List } = require('immutable')
var myMap = Map({ a: 'Apple', b: 'Banana' })
List(myMap) // List [ [ "a", "Apple" ], [ "b", "Banana" ] ]
myMap.toList() // List [ "Apple", "Banana" ]
```

### [toStack()](https://immutable-js.github.io/immutable-js/docs/#/Collection/toStack)

Converts this Collection to a Stack, discarding keys. Throws if values are not hashable.

```
toStack(): Stack 
```

#### DISCUSSION

Note: This is equivalent to `Stack(this)`, but provided to allow for chained expressions.

## Conversion to Seq

### [toSeq()](https://immutable-js.github.io/immutable-js/docs/#/Collection/toSeq)

Converts this Collection to a Seq of the same kind (indexed, keyed, or set).

```
toSeq(): Seq
```

### [toKeyedSeq()](https://immutable-js.github.io/immutable-js/docs/#/Collection/toKeyedSeq)

Returns a Seq.Keyed from this Collection where indices are treated as keys.

```
toKeyedSeq(): Seq.Keyed 
```

#### DISCUSSION

This is useful if you want to operate on an Collection.Indexed and preserve the [index, value] pairs.

The returned Seq will have identical iteration order as this Collection.

```js
const { Seq } = require('immutable')
const indexedSeq = Seq([ 'A', 'B', 'C' ])
// Seq [ "A", "B", "C" ]
indexedSeq.filter(v => v === 'B')
// Seq [ "B" ]
const keyedSeq = indexedSeq.toKeyedSeq()
// Seq { 0: "A", 1: "B", 2: "C" }
keyedSeq.filter(v => v === 'B')
// Seq { 1: "B" }
```

### [toIndexedSeq()](https://immutable-js.github.io/immutable-js/docs/#/Collection/toIndexedSeq)

Returns an Seq.Indexed of the values of this Collection, discarding keys.

```js
toIndexedSeq(): Seq.Indexed
```

### [toSetSeq()](https://immutable-js.github.io/immutable-js/docs/#/Collection/toSetSeq)

Returns a Seq.Set of the values of this Collection, discarding keys.

```js
toSetSeq(): Seq.Set
```

## Iterators

### [keys()](https://immutable-js.github.io/immutable-js/docs/#/Collection/keys)

An iterator of this `Collection`'s keys.

```
keys(): IterableIterator 
```

#### DISCUSSION

Note: this will return an ES6 iterator which does not support Immutable.js sequence algorithms. Use `keySeq` instead, if this is what you want.

### [values()](https://immutable-js.github.io/immutable-js/docs/#/Collection/values)

An iterator of this `Collection`'s values.

```
values(): IterableIterator 
```

#### DISCUSSION

Note: this will return an ES6 iterator which does not support Immutable.js sequence algorithms. Use `valueSeq` instead, if this is what you want.

### [entries()](https://immutable-js.github.io/immutable-js/docs/#/Collection/entries)

An iterator of this `Collection`'s entries as `[ key, value ]` tuples.

```
entries(): IterableIterator<[K, V]> 
```

#### DISCUSSION

Note: this will return an ES6 iterator which does not support Immutable.js sequence algorithms. Use `entrySeq` instead, if this is what you want.

## Collections (Seq)

### [keySeq()](https://immutable-js.github.io/immutable-js/docs/#/Collection/keySeq)

Returns a new Seq.Indexed of the keys of this Collection, discarding values.

```
keySeq(): Seq.Indexed
```

### [valueSeq()](https://immutable-js.github.io/immutable-js/docs/#/Collection/valueSeq)

Returns an Seq.Indexed of the values of this Collection, discarding keys.

```
valueSeq(): Seq.Indexed
```

### [entrySeq()](https://immutable-js.github.io/immutable-js/docs/#/Collection/entrySeq)

Returns a new Seq.Indexed of [key, value] tuples.

```
entrySeq(): Seq.Indexed<[K, V]>
```

## Sequence algorithms

### [map()](https://immutable-js.github.io/immutable-js/docs/#/Collection/map)

Returns a new Collection of the same type with values passed through a `mapper` function.

```
map<M>(
  mapper: (value: V, key: K, iter: this) => M,
  context?: any
): Collection<K, M>
```

#### DISCUSSION

```js
const { Collection } = require('immutable')
Collection({ a: 1, b: 2 }).map(x => 10 * x)
// Seq { "a": 10, "b": 20 }
```

Note: `map()` always returns a new instance, even if it produced the same value at every step.

### [filter()](https://immutable-js.github.io/immutable-js/docs/#/Collection/filter)

Returns a new Collection of the same type with only the entries for which the `predicate` function returns true.

```js
filter(
  predicate: (value: V, key: K, iter: this) => any, 
  context?: any
): this
```

#### DISCUSSION

```js
const { Map } = require('immutable')
Map({ a: 1, b: 2, c: 3, d: 4}).filter(x => x % 2 === 0)
// Map { "b": 2, "d": 4 }
```

Note: `filter()` always returns a new instance, even if it results in not filtering out any values.

### [filterNot()](https://immutable-js.github.io/immutable-js/docs/#/Collection/filterNot)

Returns a new Collection of the same type with only the entries for which the `predicate` function returns false.

```js
filterNot(
  predicate: (value: V, key: K, iter: this) => boolean,
  context?: any
): this
```

#### DISCUSSION

```js
const { Map } = require('immutable')
Map({ a: 1, b: 2, c: 3, d: 4}).filterNot(x => x % 2 === 0)
// Map { "a": 1, "c": 3 }
```

Note: `filterNot()` always returns a new instance, even if it results in not filtering out any values.

### [reverse()](https://immutable-js.github.io/immutable-js/docs/#/Collection/reverse)

Returns a new Collection of the same type in reverse order.

```js
reverse(): this
```

### [sort()](https://immutable-js.github.io/immutable-js/docs/#/Collection/sort)

Returns a new Collection of the same type which includes the same entries, stably sorted by using a `comparator`.

```
sort(comparator?: (valueA: V, valueB: V) => number): this 
```

#### DISCUSSION

If a `comparator` is not provided, a default comparator uses `<` and `>`.

`comparator(valueA, valueB)`:

- Returns `0` if the elements should not be swapped.
- Returns `-1` (or any negative number) if `valueA` comes before `valueB`
- Returns `1` (or any positive number) if `valueA` comes after `valueB`
- Is pure, i.e. it must always return the same value for the same pair of values.

When sorting collections which have no defined order, their ordered equivalents will be returned. e.g. `map.sort()` returns OrderedMap.

```js
const { Map } = require('immutable')
Map({ "c": 3, "a": 1, "b": 2 }).sort((a, b) => {
  if (a < b) { return -1; }
  if (a > b) { return 1; }
  if (a === b) { return 0; }
});
// OrderedMap { "a": 1, "b": 2, "c": 3 }
```

Note: `sort()` Always returns a new instance, even if the original was already sorted.

Note: This is always an eager operation.

### [sortBy()](https://immutable-js.github.io/immutable-js/docs/#/Collection/sortBy)

Like `sort`, but also accepts a `comparatorValueMapper` which allows for sorting by more sophisticated means:

```js
sortBy<C>(
  comparatorValueMapper: (value: V, key: K, iter: this) => C,
  comparator?: (valueA: C, valueB: C) => number
): this
```

第二個參數給定返回值的比較器

#### EXAMPLE

```js
hitters.sortBy(hitter => hitter.avgHits)
```

Note: `sortBy()` Always returns a new instance, even if the original was already sorted.

Note: This is always an eager operation.

### [groupBy()](https://immutable-js.github.io/immutable-js/docs/#/Collection/groupBy)

Returns a `Collection.Keyed` of `Collection.Keyeds`, grouped by the return value of the `grouper` function.

```ts
groupBy<G>(
  grouper: (value: V, key: K, iter: this) => G,
  context?: any
): Seq.Keyed<G, Collection<K, V>>
```

#### DISCUSSION

Note: This is always an eager operation.

```ts
const { List, Map } = require('immutable')
const listOfMaps = List([
  Map({ v: 0 }),
  Map({ v: 1 }),
  Map({ v: 1 }),
  Map({ v: 0 }),
  Map({ v: 2 })
])
const groupsOfMaps = listOfMaps.groupBy(x => x.get('v'))
// Map {
//   0: List [ Map{ "v": 0 }, Map { "v": 0 } ],
//   1: List [ Map{ "v": 1 }, Map { "v": 1 } ],
//   2: List [ Map{ "v": 2 } ],
// }
```

## Side effects

### [forEach()](https://immutable-js.github.io/immutable-js/docs/#/Collection/forEach)

The `sideEffect` is executed for every entry in the Collection.

```ts
forEach(
  sideEffect: (value: V, key: K, iter: this) => any,
  context?: any
): number
```

#### DISCUSSION

Unlike `Array#forEach`, if any call of `sideEffect` returns `false`, the iteration will stop. Returns the number of entries iterated (including the last iteration which returned false).

## Creating subsets

### [slice()](https://immutable-js.github.io/immutable-js/docs/#/Collection/slice)

Returns a new Collection of the same type representing a portion of this Collection from start up to but not including end.

```js
slice(begin?: number, end?: number): this 
```

#### DISCUSSION

If begin is negative, it is offset from the end of the Collection. e.g. `slice(-2)` returns a Collection of the last two entries. If it is not provided the new Collection will begin at the beginning of this Collection.

If end is negative, it is offset from the end of the Collection. e.g. `slice(0, -1)` returns a Collection of everything but the last entry. If it is not provided, the new Collection will continue through the end of this Collection.

If the requested slice is equivalent to the current Collection, then it will return itself.

### [rest()](https://immutable-js.github.io/immutable-js/docs/#/Collection/rest)

Returns a new Collection of the same type containing all entries except the first.

```js
rest(): this
```

### [butLast()](https://immutable-js.github.io/immutable-js/docs/#/Collection/butLast)

Returns a new Collection of the same type containing all entries except the last.

```js
butLast(): this
```

### [skip()](https://immutable-js.github.io/immutable-js/docs/#/Collection/skip)

Returns a new Collection of the same type which excludes the first `amount` entries from this Collection.

```
skip(amount: number): this
```

### [skipLast()](https://immutable-js.github.io/immutable-js/docs/#/Collection/skipLast)

Returns a new Collection of the same type which excludes the last `amount` entries from this Collection.

```
skipLast(amount: number): this
```

### [skipWhile()](https://immutable-js.github.io/immutable-js/docs/#/Collection/skipWhile)

Returns a new Collection of the same type which includes entries starting from when `predicate` first returns false.

```
skipWhile(
predicate: (value: V, key: K, iter: this) => boolean,
context?: any
): this
```

#### DISCUSSION

```js
const { List } = require('immutable')
List([ 'dog', 'frog', 'cat', 'hat', 'god' ])
  .skipWhile(x => x.match(/g/))
// List [ "cat", "hat", "god"" ]
```

### [skipUntil()](https://immutable-js.github.io/immutable-js/docs/#/Collection/skipUntil)

Returns a new Collection of the same type which includes entries starting from when `predicate` first returns true.

```
skipUntil(
predicate: (value: V, key: K, iter: this) => boolean,
context?: any
): this
```

#### DISCUSSION

```
const { List } = require('immutable')
List([ 'dog', 'frog', 'cat', 'hat', 'god' ])
  .skipUntil(x => x.match(/hat/))
// List [ "hat", "god"" ]
```

### [take()](https://immutable-js.github.io/immutable-js/docs/#/Collection/take)

Returns a new Collection of the same type which includes the first `amount` entries from this Collection.

```js
take(amount: number): this
```

### [takeLast()](https://immutable-js.github.io/immutable-js/docs/#/Collection/takeLast)

Returns a new Collection of the same type which includes the last `amount` entries from this Collection.

```js
takeLast(amount: number): this
```

### [takeWhile()](https://immutable-js.github.io/immutable-js/docs/#/Collection/takeWhile)

Returns a new Collection of the same type which includes entries from this Collection as long as the `predicate` returns true.

```
takeWhile(
  predicate: (value: V, key: K, iter: this) => boolean,
  context?: any
): this
```

#### DISCUSSION

```js
const { List } = require('immutable')
List([ 'dog', 'frog', 'cat', 'hat', 'god' ])
  .takeWhile(x => x.match(/o/))
// List [ "dog", "frog" ]
```

### [takeUntil()](https://immutable-js.github.io/immutable-js/docs/#/Collection/takeUntil)

Returns a new Collection of the same type which includes entries from this Collection as long as the `predicate` returns false.

```
takeUntil(
  predicate: (value: V, key: K, iter: this) => boolean,
  context?: any
): this
```

#### DISCUSSION

```js
const { List } = require('immutable')
List([ 'dog', 'frog', 'cat', 'hat', 'god' ])
  .takeUntil(x => x.match(/at/))
// List [ "dog", "frog" ]
```

## Combination

### [concat()](https://immutable-js.github.io/immutable-js/docs/#/Collection/concat)

Returns a new Collection of the same type with other values and collection-like concatenated to this one.

```
concat(...valuesOrCollections: Array): Collection 
```

#### DISCUSSION

For Seqs, all entries will be present in the resulting Seq, even if they have the same key.

### [flatten()](https://immutable-js.github.io/immutable-js/docs/#/Collection/flatten)

Flattens nested Collections.

```
flatten(depth?: number): Collection flatten(shallow?: boolean): Collection 
```

#### DISCUSSION

Will deeply flatten the Collection by default, returning a Collection of the same type, but a `depth` can be provided in the form of a number or boolean (where true means to shallowly flatten one level). A depth of 0 (or shallow: false) will deeply flatten.

Flattens only others Collection, not Arrays or Objects.

Note: `flatten(true)` operates on `Collection<any, Collection<K, V>>` and returns `Collection<K, V>`

### [flatMap()](https://immutable-js.github.io/immutable-js/docs/#/Collection/flatMap)

Flat-maps the Collection, returning a Collection of the same type.

```js
flatMap<M>(
  mapper: (value: V, key: K, iter: this) => Collection<M>,
  context?: any
): Collection<K, M>

flatMap<KM, VM>(
  mapper: (value: V, key: K, iter: this) => Collection<[KM, VM]>,
  context?: any
): Collection<KM, VM>
```

#### DISCUSSION

Similar to `collection.map(...).flatten(true)`. Used for Dictionaries only.

## Reducing a value

### [reduce()](https://immutable-js.github.io/immutable-js/docs/#/Collection/reduce)

Reduces the Collection to a value by calling the `reducer` for every entry in the Collection and passing along the reduced value.

```
reduce<R>(
  reducer: (reduction: R, value: V, key: K, iter: this) => R,
  initialReduction: R,
  context?: any
): R
reduce<R>(
  reducer: (reduction: V | R, value: V, key: K, iter: this) => R
): R
```

#### SEE

`Array#reduce`.

#### DISCUSSION

If `initialReduction` is not provided, the first item in the Collection will be used.

### [reduceRight()](https://immutable-js.github.io/immutable-js/docs/#/Collection/reduceRight)

Reduces the Collection in reverse (from the right side).

```
reduceRight<R>(
reducer: (reduction: R, value: V, key: K, iter: this) => R,
initialReduction: R,
context?: any
): R
reduceRight<R>(
reducer: (reduction: V | R, value: V, key: K, iter: this) => R
): R
```

#### DISCUSSION

Note: Similar to `this.reverse().reduce()`, and provided for parity with `Array#reduceRight`.

### [every()](https://immutable-js.github.io/immutable-js/docs/#/Collection/every)

True if `predicate` returns true for all entries in the Collection.

```js
every(
  predicate: (value: V, key: K, iter: this) => boolean,
  context?: any
): boolean
```

### [some()](https://immutable-js.github.io/immutable-js/docs/#/Collection/some)

True if `predicate` returns true for any entry in the Collection.

```js
some(
  predicate: (value: V, key: K, iter: this) => boolean,
  context?: any
): boolean
```

### [join()](https://immutable-js.github.io/immutable-js/docs/#/Collection/join)

Joins values together as a string, inserting a separator between each. The default separator is `","`.

```js
join(separator?: string): string
```

### [isEmpty()](https://immutable-js.github.io/immutable-js/docs/#/Collection/isEmpty)

Returns true if this Collection includes no values.

```
isEmpty(): boolean 
```

#### DISCUSSION

For some lazy `Seq`, `isEmpty` might need to iterate to determine emptiness. At most one iteration will occur.

### [count()](https://immutable-js.github.io/immutable-js/docs/#/Collection/count)

Returns the size of this Collection.

```
count(): number
count(
  predicate: (value: V, key: K, iter: this) => boolean,
  context?: any
): number
```

#### DISCUSSION

Regardless of if this Collection can describe its size lazily (some Seqs cannot), this method will always return the correct size. E.g. it evaluates a lazy `Seq` if necessary.

If `predicate` is provided, then this returns the count of entries in the Collection for which the `predicate` returns true.

### [countBy()](https://immutable-js.github.io/immutable-js/docs/#/Collection/countBy)

Returns a `Seq.Keyed` of counts, grouped by the return value of the `grouper` function.

```
countBy(grouper: (value: V, key: K, iter: this) => G,context?: any): Map 
```

#### DISCUSSION

Note: This is not a lazy operation.

## Search for value

### [find()](https://immutable-js.github.io/immutable-js/docs/#/Collection/find)

Returns the first value for which the `predicate` returns true.

```
find(
predicate: (value: V, key: K, iter: this) => boolean,
context?: any,
notSetValue?: V
): V | undefined
```

### [findLast()](https://immutable-js.github.io/immutable-js/docs/#/Collection/findLast)

Returns the last value for which the `predicate` returns true.

```
findLast(
predicate: (value: V, key: K, iter: this) => boolean,
context?: any,
notSetValue?: V
): V | undefined
```

#### DISCUSSION

Note: `predicate` will be called for each entry in reverse.

### [findEntry()](https://immutable-js.github.io/immutable-js/docs/#/Collection/findEntry)

Returns the first [key, value] entry for which the `predicate` returns true.

```
findEntry(predicate: (value: V, key: K, iter: this) => boolean,context?: any,notSetValue?: V): [K, V] | undefined
```

### [findLastEntry()](https://immutable-js.github.io/immutable-js/docs/#/Collection/findLastEntry)

Returns the last [key, value] entry for which the `predicate` returns true.

```
findLastEntry(predicate: (value: V, key: K, iter: this) => boolean,context?: any,notSetValue?: V): [K, V] | undefined 
```

#### DISCUSSION

Note: `predicate` will be called for each entry in reverse.

### [findKey()](https://immutable-js.github.io/immutable-js/docs/#/Collection/findKey)

Returns the key for which the `predicate` returns true.

```
findKey(predicate: (value: V, key: K, iter: this) => boolean,context?: any): K | undefined
```

### [findLastKey()](https://immutable-js.github.io/immutable-js/docs/#/Collection/findLastKey)

Returns the last key for which the `predicate` returns true.

```
findLastKey(predicate: (value: V, key: K, iter: this) => boolean,context?: any): K | undefined 
```

#### DISCUSSION

Note: `predicate` will be called for each entry in reverse.

### [keyOf()](https://immutable-js.github.io/immutable-js/docs/#/Collection/keyOf)

Returns the key associated with the search value, or undefined.

```
keyOf(searchValue: V): K | undefined
```

### [lastKeyOf()](https://immutable-js.github.io/immutable-js/docs/#/Collection/lastKeyOf)

Returns the last key associated with the search value, or undefined.

```
lastKeyOf(searchValue: V): K | undefined
```

### [max()](https://immutable-js.github.io/immutable-js/docs/#/Collection/max)

Returns the maximum value in this collection. If any values are comparatively equivalent, the first one found will be returned.

```
max(comparator?: (valueA: V, valueB: V) => number): V | undefined 
```

#### DISCUSSION

The `comparator` is used in the same way as `Collection#sort`. If it is not provided, the default comparator is `>`.

When two values are considered equivalent, the first encountered will be returned. Otherwise, `max` will operate independent of the order of input as long as the comparator is commutative. The default comparator `>` is commutative *only* when types do not differ.

If `comparator` returns 0 and either value is NaN, undefined, or null, that value will be returned.

### [maxBy()](https://immutable-js.github.io/immutable-js/docs/#/Collection/maxBy)

Like `max`, but also accepts a `comparatorValueMapper` which allows for comparing by more sophisticated means:

```
maxBy(comparatorValueMapper: (value: V, key: K, iter: this) => C,comparator?: (valueA: C, valueB: C) => number): V | undefined 
```

#### EXAMPLE

```
hitters.maxBy(hitter => hitter.avgHits);
```

### [min()](https://immutable-js.github.io/immutable-js/docs/#/Collection/min)

Returns the minimum value in this collection. If any values are comparatively equivalent, the first one found will be returned.

```
min(comparator?: (valueA: V, valueB: V) => number): V | undefined 
```

#### DISCUSSION

The `comparator` is used in the same way as `Collection#sort`. If it is not provided, the default comparator is `<`.

When two values are considered equivalent, the first encountered will be returned. Otherwise, `min` will operate independent of the order of input as long as the comparator is commutative. The default comparator `<` is commutative *only* when types do not differ.

If `comparator` returns 0 and either value is NaN, undefined, or null, that value will be returned.

### [minBy()](https://immutable-js.github.io/immutable-js/docs/#/Collection/minBy)

Like `min`, but also accepts a `comparatorValueMapper` which allows for comparing by more sophisticated means:

```
minBy(comparatorValueMapper: (value: V, key: K, iter: this) => C,comparator?: (valueA: C, valueB: C) => number): V | undefined 
```

#### EXAMPLE

```
hitters.minBy(hitter => hitter.avgHits);
```

## Comparison

### [isSubset()](https://immutable-js.github.io/immutable-js/docs/#/Collection/isSubset)

True if `iter` includes every value in this Collection.

```
isSubset(iter: Collection): boolean
```

### [isSuperset()](https://immutable-js.github.io/immutable-js/docs/#/Collection/isSuperset)

True if this Collection includes every value in `iter`.

```
isSuperset(iter: Collection): boolean
```