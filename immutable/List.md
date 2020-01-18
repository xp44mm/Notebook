# List

Lists are ordered indexed dense collections, much like a JavaScript Array.

```
type List extends Collection.Indexed
```

#### DISCUSSION

Lists are immutable and fully persistent with O(log32 N) gets and sets, and O(1) push and pop.

Lists implement Deque, with efficient addition and removal from both the end (`push`, `pop`) and beginning (`unshift`, `shift`).

Unlike a JavaScript Array, there is no distinction between an "unset" index and an index set to `undefined`. `List#forEach` visits all indices from 0 to size, regardless of whether they were explicitly defined.

## Construction


### [List()](https://immutable-js.github.io/immutable-js/docs/#/List/List)



## Static methods


### [List.isList()](https://immutable-js.github.io/immutable-js/docs/#/List/isList)



### [List.of()](https://immutable-js.github.io/immutable-js/docs/#/List/of)



## Members


### [size](https://immutable-js.github.io/immutable-js/docs/#/List/size)

```
size
```

## Persistent changes


### [set()](https://immutable-js.github.io/immutable-js/docs/#/List/set)



### [delete()](https://immutable-js.github.io/immutable-js/docs/#/List/delete)



### [insert()](https://immutable-js.github.io/immutable-js/docs/#/List/insert)



### [clear()](https://immutable-js.github.io/immutable-js/docs/#/List/clear)



### [push()](https://immutable-js.github.io/immutable-js/docs/#/List/push)



### [pop()](https://immutable-js.github.io/immutable-js/docs/#/List/pop)



### [unshift()](https://immutable-js.github.io/immutable-js/docs/#/List/unshift)



### [shift()](https://immutable-js.github.io/immutable-js/docs/#/List/shift)



### [update()](https://immutable-js.github.io/immutable-js/docs/#/List/update)



### [setSize()](https://immutable-js.github.io/immutable-js/docs/#/List/setSize)



## Deep persistent changes


### [setIn()](https://immutable-js.github.io/immutable-js/docs/#/List/setIn)



### [deleteIn()](https://immutable-js.github.io/immutable-js/docs/#/List/deleteIn)



### [updateIn()](https://immutable-js.github.io/immutable-js/docs/#/List/updateIn)



### [mergeIn()](https://immutable-js.github.io/immutable-js/docs/#/List/mergeIn)



### [mergeDeepIn()](https://immutable-js.github.io/immutable-js/docs/#/List/mergeDeepIn)



## Transient changes


### [withMutations()](https://immutable-js.github.io/immutable-js/docs/#/List/withMutations)



### [asMutable()](https://immutable-js.github.io/immutable-js/docs/#/List/asMutable)



### [wasAltered()](https://immutable-js.github.io/immutable-js/docs/#/List/wasAltered)

```
wasAltered(): boolean `SEE`Map#wasAltered
```

### [asImmutable()](https://immutable-js.github.io/immutable-js/docs/#/List/asImmutable)

```
asImmutable(): this `SEE`Map#asImmutable
```

## Sequence algorithms


### [concat()](https://immutable-js.github.io/immutable-js/docs/#/List/concat)



### [map()](https://immutable-js.github.io/immutable-js/docs/#/List/map)



### [flatMap()](https://immutable-js.github.io/immutable-js/docs/#/List/flatMap)



### [filter()](https://immutable-js.github.io/immutable-js/docs/#/List/filter)



### [zip()](https://immutable-js.github.io/immutable-js/docs/#/List/zip)



### [zipAll()](https://immutable-js.github.io/immutable-js/docs/#/List/zipAll)



### [zipWith()](https://immutable-js.github.io/immutable-js/docs/#/List/zipWith)



### [[Symbol.iterator]()](https://immutable-js.github.io/immutable-js/docs/#/List/[Symbol.iterator])

```
[Symbol.iterator](): IterableIterator 
```

#### INHERITED FROM

```
Collection.Indexed#[Symbol.iterator]
```