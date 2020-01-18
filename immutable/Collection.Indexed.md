# Collection.Indexed

Indexed Collections have incrementing numeric keys. They exhibit slightly different behavior than `Collection.Keyed` for some methods in order to better mirror the behavior of JavaScript's `Array`, and add methods which do not make sense on non-indexed Collections such as `indexOf`.

```ts
type Collection.Indexed extends Collection
```

#### DISCUSSION

Unlike JavaScript arrays, `Collection.Indexed`s are always dense. "Unset" indices and `undefined` indices are indistinguishable, and all indices from 0 to `size` are visited when iterated.

All Collection.Indexed methods return re-indexed Collections. In other words, indices always start at 0 and increment until size. If you wish to preserve indices, using them as keys, convert to a Collection.Keyed by calling `toKeyedSeq`.

## Construction


### [Collection.Indexed()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/Collection.Indexed)

Creates a new Collection.Indexed.

```ts
Collection.Indexed<T>(collection: Collection<T>): Collection.Indexed<T>
```

#### DISCUSSION

Note: `Collection.Indexed` is a conversion function and not a class, and does not use the `new` keyword during construction.

## Conversion to JavaScript types


### [toJS()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/toJS)



### [toJSON()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/toJSON)



### [toArray()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/toArray)



## Reading values


### [get()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/get)



## Conversion to Seq


### [toSeq()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/toSeq)



### [fromEntrySeq()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/fromEntrySeq)



## Combination


### [interpose()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/interpose)



### [interleave()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/interleave)



### [splice()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/splice)



### [zip()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/zip)



### [zipAll()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/zipAll)



### [zipWith()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/zipWith)



## Search for value


### [indexOf()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/indexOf)



### [lastIndexOf()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/lastIndexOf)



### [findIndex()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/findIndex)



### [findLastIndex()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/findLastIndex)



## Sequence algorithms


### [concat()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/concat)



### [map()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/map)



### [flatMap()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/flatMap)



### [filter()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/filter)



### [[Symbol.iterator]()](https://immutable-js.github.io/immutable-js/docs/#/Collection.Indexed/[Symbol.iterator])

```
[Symbol.iterator](): IterableIterator
```