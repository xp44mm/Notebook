# Seq

`Seq` describes a lazy operation, allowing them to efficiently chain use of all the higher-order collection methods (such as `map` and `filter`) by not creating intermediate collections.

```ts
type Seq extends Collection
```

#### DISCUSSION

**Seq is immutable** — Once a Seq is created, it cannot be changed, appended to, rearranged or otherwise modified. Instead, any mutative method called on a `Seq` will return a new `Seq`.

**Seq is lazy** — `Seq` does as little work as necessary to respond to any method call. Values are often created during iteration, including implicit iteration when reducing or converting to a concrete data structure such as a `List` or JavaScript `Array`.

For example, the following performs no work, because the resulting `Seq`'s values are never iterated:

```ts
const { Seq } = require('immutable')
const oddSquares = Seq([ 1, 2, 3, 4, 5, 6, 7, 8 ])
  .filter(x => x % 2 !== 0)
  .map(x => x * x)
```

Once the `Seq` is used, it performs only the work necessary. In this example, no intermediate arrays are ever created, filter is called three times, and map is only called once:

```ts
oddSquares.get(1); // 9
```

Any collection can be converted to a lazy Seq with `Seq()`.

```ts
const { Map } = require('immutable')
const map = Map({ a: 1, b: 2, c: 3 }
const lazySeq = Seq(map)
```

`Seq` allows for the efficient chaining of operations, allowing for the expression of logic that can otherwise be very tedious:

```js
lazySeq
  .flip()
  .map(key => key.toUpperCase())
  .flip()
// Seq { A: 1, B: 1, C: 1 }
```

As well as expressing logic that would otherwise seem memory or time limited, for example `Range` is a special kind of Lazy sequence.

```js
const { Range } = require('immutable')
Range(1, Infinity)
  .skip(1000)
  .map(n => -n)
  .filter(n => n % 2 === 0)
  .take(2)
  .reduce((r, n) => r * n, 1)
// 1006008
```

Seq is often used to provide a rich collection API to JavaScript Object.

```js
Seq({ x: 0, y: 1, z: 2 }).map(v => v * 2).toObject();
// { x: 0, y: 2, z: 4 }
```

#### Sub-types

[Seq.Keyed](https://immutable-js.github.io/immutable-js/docs/#/Seq.Keyed)

[Seq.Indexed](https://immutable-js.github.io/immutable-js/docs/#/Seq.Indexed)

[Seq.Set](https://immutable-js.github.io/immutable-js/docs/#/Seq.Set)

#### Construction

### [Seq()](https://immutable-js.github.io/immutable-js/docs/#/Seq/Seq)

Creates a Seq.

```ts
Seq<S>(seq: S): S
Seq<K, V>(collection: Collection.Keyed<K, V>): Seq.Keyed<K, V>
Seq<T>(collection: Collection.Indexed<T>): Seq.Indexed<T>
Seq<T>(collection: Collection.Set<T>): Seq.Set<T>
Seq<T>(collection: Collection<T>): Seq.Indexed<T>
Seq<V>(obj: {[key: string]: V}): Seq.Keyed<string, V>
Seq(): Seq<any, any>
```

#### DISCUSSION

Returns a particular kind of `Seq` based on the input.

- If a `Seq`, that same `Seq`.
- If an `Collection`, a `Seq` of the same kind (Keyed, Indexed, or Set).
- If an Array-like, an `Seq.Indexed`.
- If an Iterable Object, an `Seq.Indexed`.
- If an Object, a `Seq.Keyed`.

Note: An Iterator itself will be treated as an object, becoming a `Seq.Keyed`, which is usually not what you want. You should turn your Iterator Object into an iterable object by defining a Symbol.iterator (or @@iterator) method which returns `this`.

Note: `Seq` is a conversion function and not a class, and does not use the `new` keyword during construction.