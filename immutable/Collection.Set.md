# Collection.Set

Set Collections only represent values. They have no associated keys or indices. Duplicate values are possible in the lazy `Seq.Set`s, however the concrete `Set` Collection does not allow duplicate values.

```
type Collection.Set extends Collection
```

#### DISCUSSION

Collection methods on Collection.Set such as `map` and `forEach` will provide the value as both the first and second arguments to the provided function.

```
const { Collection } = require('immutable') const seq = Collection.Set([ 'A', 'B', 'C' ]) // Seq { "A", "B", "C" } seq.forEach((v, k) => assert.equal(v, k) )
```

## Construction


### [Collection.Set()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Set/Collection.Set)

Similar to `Collection()`, but always returns a Collection.Set.

```
Collection.Set(collection: Collection): Collection.Set 
```

#### DISCUSSION

Note: `Collection.Set` is a factory function and not a class, and does not use the `new` keyword during construction.

## Conversion to JavaScript types


### [toJS()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Set/toJS)

Deeply converts this Set collection to equivalent native JavaScript Array.

```
toJS(): Array 
```

#### OVERRIDES

```
Collection#toJS
```

### [toJSON()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Set/toJSON)

Shallowly converts this Set collection to equivalent native JavaScript Array.

```
toJSON(): Array 
```

#### OVERRIDES

```
Collection#toJSON
```

### [toArray()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Set/toArray)

Shallowly converts this collection to an Array.

```
toArray(): Array 
```

#### OVERRIDES

```
Collection#toArray
```

## Conversion to Seq


### [toSeq()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Set/toSeq)

Returns Seq.Set.

```
toSeq(): Seq.Set 
```

#### OVERRIDES

```
Collection#toSeq
```

## Sequence algorithms


### [concat()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Set/concat)

Returns a new Collection with other collections concatenated to this one.

```
concat(...collections: Array>): Collection.Set 
```

#### OVERRIDES

```
Collection#concat
```

### [map()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Set/map)

Returns a new Collection.Set with values passed through a `mapper` function.

```
map(mapper: (value: T, key: T, iter: this) => M,context?: any): Collection.Set 
```

#### OVERRIDES

```
Collection#map
```

#### EXAMPLE

```
Collection.Set([ 1, 2 ]).map(x => 10 * x) // Seq { 1, 2 }
```

Note: `map()` always returns a new instance, even if it produced the same value at every step.

### [flatMap()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Set/flatMap)

Flat-maps the Collection, returning a Collection of the same type.

```
flatMap(mapper: (value: T, key: T, iter: this) => Collection,context?: any): Collection.Set 
```

#### OVERRIDES

```
Collection#flatMap
```

#### DISCUSSION

Similar to `collection.map(...).flatten(true)`.

### [filter()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Set/filter)

Returns a new Collection with only the values for which the `predicate` function returns true.

```
filter(predicate: (value: T, key: T, iter: this) => boolean,context?: any): Collection.Set filter(predicate: (value: T, key: T, iter: this) => any, context?: any): this 
```

#### OVERRIDES

```
Collection#filter
```

#### DISCUSSION

Note: `filter()` always returns a new instance, even if it results in not filtering out any values.

### [[Symbol.iterator]()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Set/[Symbol.iterator])

```
[Symbol.iterator](): IterableIterator
```