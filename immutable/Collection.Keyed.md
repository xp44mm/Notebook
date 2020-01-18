# Collection.Keyed

Keyed Collections have discrete keys tied to each value.

```
type Collection.Keyed extends Collection
```

#### DISCUSSION

When iterating `Collection.Keyed`, each iteration will yield a `[K, V]` tuple, in other words, `Collection#entries` is the default iterator for Keyed Collections.

## Construction


### [Collection.Keyed()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Keyed/Collection.Keyed)

Creates a Collection.Keyed

```ts
Collection.Keyed<K, V>(collection: Collection<[K, V]>): Collection.Keyed<K, V>
Collection.Keyed<V>(obj: {[key: string]: V}): Collection.Keyed<string, V>
```

#### DISCUSSION

Similar to `Collection()`, however it expects collection-likes of [K, V] tuples if not constructed from a Collection.Keyed or JS Object.

Note: `Collection.Keyed` is a conversion function and not a class, and does not use the `new` keyword during construction.


## Conversion to JavaScript types


### [toJS()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Keyed/toJS)

Deeply converts this Keyed collection to equivalent native JavaScript Object.

```
toJS(): Object 
```

#### OVERRIDES

```
Collection#toJS
```

#### DISCUSSION

Converts keys to Strings.

### [toJSON()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Keyed/toJSON)

Shallowly converts this Keyed collection to equivalent native JavaScript Object.

```
toJSON(): {[key: string]: V} 
```

#### OVERRIDES

```
Collection#toJSON
```

#### DISCUSSION

Converts keys to Strings.

### [toArray()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Keyed/toArray)

Shallowly converts this collection to an Array.

```
toArray(): Array<[K, V]> 
```

#### OVERRIDES

```
Collection#toArray
```

## Conversion to Seq


### [toSeq()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Keyed/toSeq)

Returns Seq.Keyed.

```
toSeq(): Seq.Keyed 
```

#### OVERRIDES

```
Collection#toSeq
```

## Sequence functions


### [flip()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Keyed/flip)

Returns a new Collection.Keyed of the same type where the keys and values have been flipped.

```
flip(): Collection.Keyed 
```

#### DISCUSSION

```js
const { Map } = require('immutable')
Map({ a: 'z', b: 'y' }).flip()
// Map { "z": "a", "y": "b" }
```

### [concat()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Keyed/concat)

Returns a new Collection with other collections concatenated to this one.

```
concat<KC, VC>(
...collections: Array<Collection<[KC, VC]>>
): Collection.Keyed<K | KC, V | VC>
concat<C>(
...collections: Array<{[key: string]: C}>
): Collection.Keyed<K | string, V | C>
```

#### OVERRIDES

```
Collection#concat
```

### [map()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Keyed/map)

Returns a new Collection.Keyed with values passed through a `mapper` function.

```
map<M>(
mapper: (value: V, key: K, iter: this) => M,
context?: any
): Collection.Keyed<K, M>
```

#### OVERRIDES

```
Collection#map
```

#### EXAMPLE

```
const { Collection } = require('immutable')
Collection.Keyed({ a: 1, b: 2 }).map(x => 10 * x)
// Seq { "a": 10, "b": 20 }
```

Note: `map()` always returns a new instance, even if it produced the same value at every step.

### [mapKeys()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Keyed/mapKeys)

Returns a new Collection.Keyed of the same type with keys passed through a `mapper` function.

```
mapKeys<M>(
mapper: (key: K, value: V, iter: this) => M,
context?: any
): Collection.Keyed<M, V>
```

#### DISCUSSION

```js
const { Map } = require('immutable')
Map({ a: 1, b: 2 }).mapKeys(x => x.toUpperCase())
// Map { "A": 1, "B": 2 }
```

Note: `mapKeys()` always returns a new instance, even if it produced the same key at every step.

### [mapEntries()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Keyed/mapEntries)

Returns a new Collection.Keyed of the same type with entries ([key, value] tuples) passed through a `mapper` function.

```ts
mapEntries<KM, VM>(
  mapper: (entry: [K, V], index: number, iter: this) => [KM, VM],
  context?: any
): Collection.Keyed<KM, VM>
```

#### DISCUSSION

```ts
const { Map } = require('immutable')
Map({ a: 1, b: 2 })
  .mapEntries(([ k, v ]) => [ k.toUpperCase(), v * 2 ])
// Map { "A": 2, "B": 4 }
```

Note: `mapEntries()` always returns a new instance, even if it produced the same entry at every step.

### [flatMap()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Keyed/flatMap)

Flat-maps the Collection, returning a Collection of the same type.

```
flatMap<KM, VM>(
  mapper: (value: V, key: K, iter: this) => Collection<[KM, VM]>,
  context?: any
): Collection.Keyed<KM, VM>
```

#### OVERRIDES

```
Collection#flatMap
```

#### DISCUSSION

Similar to `collection.map(...).flatten(true)`.

### [filter()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Keyed/filter)

Returns a new Collection with only the values for which the `predicate` function returns true.

```js
filter<F>(
  predicate: (value: V, key: K, iter: this) => boolean,
  context?: any
): Collection.Keyed<K, F>
filter(
  predicate: (value: V, key: K, iter: this) => any, 
  context?: any
): this
```

#### OVERRIDES

```
Collection#filter
```

#### DISCUSSION

Note: `filter()` always returns a new instance, even if it results in not filtering out any values.

### [[Symbol.iterator]()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Keyed/[Symbol.iterator])

```
[Symbol.iterator](): IterableIterator<[K, V]>
```