# Map

Immutable Map is an unordered Collection.Keyed of (key, value) pairs with `O(log32 N)` gets and `O(log32 N)` persistent sets.

```
type Map extends Collection.Keyed
```

#### DISCUSSION

Iteration order of a Map is undefined, however is stable. Multiple iterations of the same Map will iterate in the same order.

Map's keys can be of any type, and use `Immutable.is` to determine key equality. This allows the use of any value (including NaN) as a key.

Because `Immutable.is` returns equality based on value semantics, and Immutable collections are treated as values, any Immutable collection may be used as a key.

```js
const { Map, List } = require('immutable');
Map().set(List([ 1 ]), 'listofone').get(List([ 1 ]));
// 'listofone'
```

Any JavaScript object may be used as a key, however strict identity is used to evaluate key equality. Two similar looking objects will represent two different keys.

Implemented by a hash-array mapped trie.

## Construction


### [Map()](https://immutable-js.github.io/immutable-js/docs/#/Map/Map)

Creates a new Immutable Map.

```ts
Map<K, V>(entries: Collection<[K, V]>): Map<K, V>
Map<V>(obj: {[key: string]: V}): Map<string, V>
Map<K, V>(): Map<K, V>
Map(): Map<any, any>
```

#### DISCUSSION

Created with the same key value pairs as the provided `Collection.Keyed` or JavaScript Object or expects a Collection of [K, V] tuple entries.

Note: `Map` is a factory function and not a class, and does not use the `new` keyword during construction.

```ts
const { Map } = require('immutable')
Map({ key: "value" })
Map([ [ "key", "value" ] ])
```

Keep in mind, when using JS objects to construct Immutable Maps, that JavaScript Object properties are always strings, even if written in a quote-less shorthand, while Immutable Maps accept keys of any type.

```ts
let obj = { 1: "one" }
Object.keys(obj) // [ "1" ]
assert.equal(obj["1"], obj[1]) // "one" === "one"

let map = Map(obj)
assert.notEqual(map.get("1"), map.get(1)) // "one" !== undefined
```

Property access for JavaScript Objects first converts the key to a string, but since Immutable Map keys can be of any type the argument to `get()` is not altered.

## Static methods


### [Map.isMap()](https://immutable-js.github.io/immutable-js/docs/#/Map/isMap)

True if the provided value is a Map

```ts
Map.isMap(maybeMap: any): boolean
```

#### DISCUSSION

```ts
const { Map } = require('immutable')
Map.isMap({}) // false
Map.isMap(Map()) // true
```

## Members


### [size](https://immutable-js.github.io/immutable-js/docs/#/Map/size)

```
size
```

## Persistent changes


### [set()](https://immutable-js.github.io/immutable-js/docs/#/Map/set)

Returns a new Map also containing the new key, value pair. If an equivalent key already exists in this Map, it will be replaced.

```ts
set(key: K, value: V): this
```

#### DISCUSSION

```ts
const { Map } = require('immutable')
const originalMap = Map()
const newerMap = originalMap.set('key', 'value')
const newestMap = newerMap.set('key', 'newer value')

originalMap
// Map {}
newerMap
// Map { "key": "value" }
newestMap
// Map { "key": "newer value" }
```

Note: `set` can be used in `withMutations`.

### [delete()](https://immutable-js.github.io/immutable-js/docs/#/Map/delete)

Returns a new Map which excludes this

```ts
delete(key: K): this
```

#### ALIAS

```
remove()
```

#### DISCUSSION

Note: `delete` cannot be safely used in IE8, but is provided to mirror the ES6 collection API.

```ts
const { Map } = require('immutable')
const originalMap = Map({
  key: 'value',
  otherKey: 'other value'
})
// Map { "key": "value", "otherKey": "other value" }
originalMap.delete('otherKey')
// Map { "key": "value" }
```

Note: `delete` can be used in `withMutations`.

### [deleteAll()](https://immutable-js.github.io/immutable-js/docs/#/Map/deleteAll)

Returns a new Map which excludes the provided `keys`.

```ts
deleteAll(keys: Collection<K>): this
```

#### ALIAS

```
removeAll()
```

#### DISCUSSION

```ts
const { Map } = require('immutable')
const names = Map({ a: "Aaron", b: "Barry", c: "Connor" })
names.deleteAll([ 'a', 'c' ])
// Map { "b": "Barry" }
```

Note: `deleteAll` can be used in `withMutations`.

### [clear()](https://immutable-js.github.io/immutable-js/docs/#/Map/clear)

Returns a new Map containing no keys or values.

```ts
clear(): this
```

#### DISCUSSION

```ts
const { Map } = require('immutable')
Map({ key: 'value' }).clear()
// Map {}
```

Note: `clear` can be used in `withMutations`.

### [update()](https://immutable-js.github.io/immutable-js/docs/#/Map/update)

Returns a new Map having updated the value at this `key` with the return value of calling `updater` with the existing value.

```diff
-update(key: K, notSetValue: V, updater: (value: V) => V): this
+update(key: K, updater: (value: V) => V): this
-update<R>(updater: (map: this) => R): R
```

#### OVERRIDES

```
Collection#update
```

#### DISCUSSION

Similar to: `map.set(key, updater(map.get(key)))`.

```ts
const { Map } = require('immutable')
const aMap = Map({ key: 'value' })
const newMap = aMap.update('key', value => value + value)
// Map { "key": "valuevalue" }
```

This is most commonly used to call methods on collections within a structure of data. For example, in order to `.push()` onto a nested `List`, `update` and `push` can be used together:

```ts
const aMap = Map({ nestedList: List([ 1, 2, 3 ]) })
const newMap = aMap.update('nestedList', list => list.push(4))
// Map { "nestedList": List [ 1, 2, 3, 4 ] }
```

When a `notSetValue` is provided, it is provided to the `updater` function when the value at the key does not exist in the Map.

```ts
const aMap = Map({ key: 'value' })
const newMap = aMap.update('noKey', 'no value', value => value + value)
// Map { "key": "value", "noKey": "no valueno value" }
```

However, if the `updater` function returns the same value it was called with, then no change will occur. This is still true if `notSetValue` is provided.

```ts
const aMap = Map({ apples: 10 })
const newMap = aMap.update('oranges', 0, val => val)
// Map { "apples": 10 }
assert.strictEqual(newMap, map);
```

For code using ES2015 or later, using `notSetValue` is discourged in favor of function parameter default values. This helps to avoid any potential confusion with identify functions as described above.

The previous example behaves differently when written with default values:

```ts
const aMap = Map({ apples: 10 })
const newMap = aMap.update('oranges', (val = 0) => val)
// Map { "apples": 10, "oranges": 0 }
```

If no key is provided, then the `updater` function return value is returned as well.

```ts
const aMap = Map({ key: 'value' })
const result = aMap.update(aMap => aMap.get('key'))
// "value"
```

This can be very useful as a way to "chain" a normal function into a sequence of methods. RxJS calls this "let" and lodash calls it "thru".

For example, to sum the values in a Map

```ts
function sum(collection) {
  return collection.reduce((sum, x) => sum + x, 0)
}

Map({ x: 1, y: 2, z: 3 })
  .map(x => x + 1)
  .filter(x => x % 2 === 0)
  .update(sum)
// 6
```

Note: `update(key)` can be used in `withMutations`.

### [merge()](https://immutable-js.github.io/immutable-js/docs/#/Map/merge)

Returns a new Map resulting from merging the provided Collections (or JS objects) into this Map. In other words, this takes each entry of each collection and sets it on this Map.

```ts
merge<KC, VC>(...collections: Array<Collection<[KC, VC]>>): Map<K | KC, V | VC>
merge<C>(...collections: Array<{[key: string]: C}>): Map<K | string, V | C>
```

#### ALIAS

```
concat()
```

#### DISCUSSION

Note: Values provided to `merge` are shallowly converted before being merged. No nested values are altered.

```ts
const { Map } = require('immutable')
const one = Map({ a: 10, b: 20, c: 30 })
const two = Map({ b: 40, a: 50, d: 60 })
one.merge(two) // Map { "a": 50, "b": 40, "c": 30, "d": 60 }
two.merge(one) // Map { "a": 10, "b": 20, "c": 30, "d": 60 }
```

Note: `merge` can be used in `withMutations`.

### [mergeWith()](https://immutable-js.github.io/immutable-js/docs/#/Map/mergeWith)

Like `merge()`, `mergeWith()` returns a new Map resulting from merging the provided Collections (or JS objects) into this Map, but uses the `merger` function for dealing with conflicts.

```ts
mergeWith(
    merger: (oldVal: V, newVal: V, key: K) => V,
    ...collections: Array<Collection<[K, V]> | {[key: string]: V}>
): this
```

#### DISCUSSION

```ts
const { Map } = require('immutable')
const one = Map({ a: 10, b: 20, c: 30 })
const two = Map({ b: 40, a: 50, d: 60 })
one.mergeWith((oldVal, newVal) => oldVal / newVal, two)
// { "a": 0.2, "b": 0.5, "c": 30, "d": 60 }
two.mergeWith((oldVal, newVal) => oldVal / newVal, one)
// { "b": 2, "a": 5, "d": 60, "c": 30 }
```

Note: `mergeWith` can be used in `withMutations`.

### [mergeDeep()](https://immutable-js.github.io/immutable-js/docs/#/Map/mergeDeep)

Like `merge()`, but when two Collections conflict, it merges them as well, recursing deeply through the nested data.

```ts
mergeDeep(...collections: Array<Collection<[K, V]> | {[key: string]: V}>): this
```

#### DISCUSSION

Note: Values provided to `merge` are shallowly converted before being merged. No nested values are altered unless they will also be merged at a deeper level.

```ts
const { Map } = require('immutable')
const one = Map({ a: Map({ x: 10, y: 10 }), b: Map({ x: 20, y: 50 }) })
const two = Map({ a: Map({ x: 2 }), b: Map({ y: 5 }), c: Map({ z: 3 }) })
one.mergeDeep(two)
// Map {
//   "a": Map { "x": 2, "y": 10 },
//   "b": Map { "x": 20, "y": 5 },
//   "c": Map { "z": 3 }
// }
```

Note: `mergeDeep` can be used in `withMutations`.

### [mergeDeepWith()](https://immutable-js.github.io/immutable-js/docs/#/Map/mergeDeepWith)

Like `mergeDeep()`, but when two non-Collections conflict, it uses the `merger` function to determine the resulting value.

```ts
mergeDeepWith(
    merger: (oldVal: any, newVal: any, key: any) => any,
    ...collections: Array<Collection<[K, V]> | {[key: string]: V}>
): this
```

#### DISCUSSION

```ts
const { Map } = require('immutable')
const one = Map({ a: Map({ x: 10, y: 10 }), b: Map({ x: 20, y: 50 }) })
const two = Map({ a: Map({ x: 2 }), b: Map({ y: 5 }), c: Map({ z: 3 }) })
one.mergeDeepWith((oldVal, newVal) => oldVal / newVal, two)
// Map {
//   "a": Map { "x": 5, "y": 10 },
//   "b": Map { "x": 20, "y": 10 },
//   "c": Map { "z": 3 }
// }
```

Note: `mergeDeepWith` can be used in `withMutations`.

## Deep persistent changes


### [setIn()](https://immutable-js.github.io/immutable-js/docs/#/Map/setIn)

Returns a new Map having set `value` at this `keyPath`. If any keys in `keyPath` do not exist, a new immutable Map will be created at that key.

```ts
setIn(keyPath: Collection<any>, value: any): this
```

#### DISCUSSION

```ts
const { Map } = require('immutable')
const originalMap = Map({
  subObject: Map({
    subKey: 'subvalue',
    subSubObject: Map({
      subSubKey: 'subSubValue'
    })
  })
})

const newMap = originalMap.setIn(['subObject', 'subKey'], 'ha ha!')
// Map {
//   "subObject": Map {
//     "subKey": "ha ha!",
//     "subSubObject": Map { "subSubKey": "subSubValue" }
//   }
// }

const newerMap = originalMap.setIn(
  ['subObject', 'subSubObject', 'subSubKey'],
  'ha ha ha!'
)
// Map {
//   "subObject": Map {
//     "subKey": "subvalue",
//     "subSubObject": Map { "subSubKey": "ha ha ha!" }
//   }
// }
```

Plain JavaScript Object or Arrays may be nested within an Immutable.js Collection, and setIn() can update those values as well, treating them immutably by creating new copies of those values with the changes applied.

```ts
const { Map } = require('immutable')
const originalMap = Map({
  subObject: {
    subKey: 'subvalue',
    subSubObject: {
      subSubKey: 'subSubValue'
    }
  }
})

originalMap.setIn(['subObject', 'subKey'], 'ha ha!')
// Map {
//   "subObject": {
//     subKey: "ha ha!",
//     subSubObject: { subSubKey: "subSubValue" }
//   }
// }
```

If any key in the path exists but cannot be updated (such as a primitive like number or a custom Object like Date), an error will be thrown.

Note: `setIn` can be used in `withMutations`.

### [deleteIn()](https://immutable-js.github.io/immutable-js/docs/#/Map/deleteIn)

Returns a new Map having removed the value at this `keyPath`. If any keys in `keyPath` do not exist, no change will occur.

```ts
deleteIn(keyPath: Collection<any>): this
```

#### ALIAS

```
removeIn()
```

#### DISCUSSION

Note: `deleteIn` can be used in `withMutations`.

### [updateIn()](https://immutable-js.github.io/immutable-js/docs/#/Map/updateIn)

Returns a new Map having applied the `updater` to the entry found at the keyPath.

```ts
updateIn(
    keyPath: Collection<unknown>, 
    notSetValue: unknown, 
    updater: (value: unknown) => unknown): this
updateIn(
    keyPath: Collection<unknown>, 
    updater: (value: unknown) => unknown): this
```

#### DISCUSSION

This is most commonly used to call methods on collections nested within a structure of data. For example, in order to `.push()` onto a nested `List`, `updateIn` and `push` can be used together:

```js
const { Map, List } = require('immutable')
const map = Map({ inMap: Map({ inList: List([ 1, 2, 3 ]) }) })
const newMap = map.updateIn(['inMap', 'inList'], list => list.push(4))
// Map { "inMap": Map { "inList": List [ 1, 2, 3, 4 ] } }
```

If any keys in `keyPath` do not exist, new Immutable `Map`s will be created at those keys. If the `keyPath` does not already contain a value, the `updater` function will be called with `notSetValue`, if provided, otherwise `undefined`.

```js
const map = Map({ a: Map({ b: Map({ c: 10 }) }) })
const newMap = map.updateIn(['a', 'b', 'c'], val => val * 2)
// Map { "a": Map { "b": Map { "c": 20 } } }
```

If the `updater` function returns the same value it was called with, then no change will occur. This is still true if `notSetValue` is provided.

```js
const map = Map({ a: Map({ b: Map({ c: 10 }) }) })
const newMap = map.updateIn(['a', 'b', 'x'], 100, val => val)
// Map { "a": Map { "b": Map { "c": 10 } } }
assert.strictEqual(newMap, aMap)
```

For code using ES2015 or later, using `notSetValue` is discourged in favor of function parameter default values. This helps to avoid any potential confusion with identify functions as described above. The previous example behaves differently when written with default values:

```js
const map = Map({ a: Map({ b: Map({ c: 10 }) }) })
const newMap = map.updateIn(['a', 'b', 'x'], (val = 100) => val)
// Map { "a": Map { "b": Map { "c": 10, "x": 100 } } }
```

Plain JavaScript Object or Arrays may be nested within an Immutable.js Collection, and `updateIn()` can update those values as well, treating them immutably by creating new copies of those values with the changes applied.

```js
const map = Map({ a: { b: { c: 10 } } })
const newMap = map.updateIn(['a', 'b', 'c'], val => val * 2)
// Map { "a": { b: { c: 20 } } }
```

If any key in the path exists but cannot be updated (such as a primitive like number or a custom Object like `Date`), an error will be thrown. 

Note: `updateIn` can be used in `withMutations`.

### [mergeIn()](https://immutable-js.github.io/immutable-js/docs/#/Map/mergeIn)

A combination of `updateIn` and `merge`, returning a new Map, but performing the merge at a point arrived at by following the `keyPath`. In other words, these two lines are equivalent:

```ts
mergeIn(keyPath: Collection<unknown>, ...collections: Array<unknown>): this
```

#### EXAMPLE


```js
map.updateIn(['a', 'b', 'c'], abc => abc.merge(y))
map.mergeIn(['a', 'b', 'c'], y)
```

Note: `mergeIn` can be used in `withMutations`.

### [mergeDeepIn()](https://immutable-js.github.io/immutable-js/docs/#/Map/mergeDeepIn)

A combination of `updateIn` and `mergeDeep`, returning a new Map, but performing the deep merge at a point arrived at by following the `keyPath`. In other words, these two lines are equivalent:

```ts
mergeDeepIn(
    keyPath: Collection<any>, 
    ...collections: Array<any>): this
```

#### EXAMPLE

```js
map.updateIn(['a', 'b', 'c'], abc => abc.mergeDeep(y))
map.mergeDeepIn(['a', 'b', 'c'], y)
```

Note: `mergeDeepIn` can be used in `withMutations`.

## Transient changes


### [withMutations()](https://immutable-js.github.io/immutable-js/docs/#/Map/withMutations)

Every time you call one of the above functions, a new immutable Map is created. If a pure function calls a number of these to produce a final return value, then a penalty on performance and memory has been paid by creating all of the intermediate immutable Maps.

```ts
withMutations(mutator: (mutable: this) => unknown): this
```

#### DISCUSSION

If you need to apply a series of mutations to produce a new immutable Map, `withMutations()` creates a temporary mutable copy of the Map which can apply mutations in a highly performant manner. In fact, this is exactly how complex mutations like `merge` are done.

As an example, this results in the creation of 2, not 4, new Maps:

```js
const { Map } = require('immutable')
const map1 = Map()
const map2 = map1.withMutations(map => {
  map.set('a', 1).set('b', 2).set('c', 3)
})
assert.equal(map1.size, 0)
assert.equal(map2.size, 3)
```

Note: Not all methods can be used on a mutable collection or within `withMutations`! Read the documentation for each method to see if it is safe to use in `withMutations`.

### [asMutable()](https://immutable-js.github.io/immutable-js/docs/#/Map/asMutable)

Another way to avoid creation of intermediate Immutable maps is to create a mutable copy of this collection. Mutable copies *always* return `this`, and thus shouldn't be used for equality. Your function should never return a mutable copy of a collection, only use it internally to create a new collection.

```ts
asMutable(): this
```

#### SEE

```
Map#asImmutable
```

#### DISCUSSION

If possible, use `withMutations` to work with temporary mutable copies as it provides an easier to use API and considers many common optimizations.

Note: if the collection is already mutable, `asMutable` returns itself.

Note: Not all methods can be used on a mutable collection or within `withMutations`! Read the documentation for each method to see if it is safe to use in `withMutations`.

### [wasAltered()](https://immutable-js.github.io/immutable-js/docs/#/Map/wasAltered)

Returns true if this is a mutable copy (see `asMutable()`) and mutative alterations have been applied.

```ts
wasAltered(): boolean 
```

#### SEE

```
Map#asMutable
```

### [asImmutable()](https://immutable-js.github.io/immutable-js/docs/#/Map/asImmutable)

The yin to `asMutable`'s yang. Because it applies to mutable collections, this operation is *mutable* and may return itself (though may not return itself, i.e. if the result is an empty collection). Once performed, the original mutable copy must no longer be mutated since it may be the immutable result.

```
asImmutable(): this 
```

#### SEE

```
Map#asMutable
```

#### DISCUSSION

If possible, use `withMutations` to work with temporary mutable copies as it provides an easier to use API and considers many common optimizations.

## Sequence algorithms


### [map()](https://immutable-js.github.io/immutable-js/docs/#/Map/map)

Returns a new Map with values passed through a `mapper` function.

```ts
    map<M>(
        mapper: (value: V, key: K, iter: this) => M,
        context?: unknown
    ): Map<K, M>
```

#### OVERRIDES

```
Collection#map
```

#### EXAMPLE

```js
Map({ a: 1, b: 2 }).map(x => 10 * x) 
// Map { a: 10, b: 20 }
```

### [mapKeys()](https://immutable-js.github.io/immutable-js/docs/#/Map/mapKeys)

```ts
    mapKeys<M>(
        mapper: (key: K, value: V, iter: this) => M,
        context?: unknown
    ): Map<M, V>
```

#### OVERRIDES

```
Collection.Keyed#mapKeys
```

#### SEE

Collection.Keyed.mapKeys

### [mapEntries()](https://immutable-js.github.io/immutable-js/docs/#/Map/mapEntries)

```
mapEntries(mapper: (entry: [K, V], index: number, iter: this) => [KM, VM],context?: any): Map 
```

#### OVERRIDES

```
Collection.Keyed#mapEntries
```

#### SEE

Collection.Keyed.mapEntries

### [flatMap()](https://immutable-js.github.io/immutable-js/docs/#/Map/flatMap)

Flat-maps the Map, returning a new Map.

```
    flatMap<KM, VM>(
        mapper: (value: V, key: K, iter: this) => Collection<[KM, VM]>,
        context?: unknown
    ): Map<KM, VM>
```

#### OVERRIDES

```
Collection#flatMap
```

#### DISCUSSION

Similar to `data.map(...).flatten(true)`.

### [filter()](https://immutable-js.github.io/immutable-js/docs/#/Map/filter)

Returns a new Map with only the entries for which the `predicate` function returns true.

```ts
    filter<F extends V>(
        predicate: (value: V, key: K, iter: this) => value is F,
        context?: unknown
    ): Map<K, F>
    filter(
        predicate: (value: V, key: K, iter: this) => unknown,
        context?: unknown
    ): this
```

#### OVERRIDES

```
Collection#filter
```

#### DISCUSSION

Note: `filter()` always returns a new instance, even if it results in not filtering out any values.

### [flip()](https://immutable-js.github.io/immutable-js/docs/#/Map/flip)

```
flip(): Map 
```

#### OVERRIDES

```
Collection.Keyed#flip
```

#### SEE

Collection.Keyed.flip
