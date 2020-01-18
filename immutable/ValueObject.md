# ValueObject

```
type ValueObject
```

## Members

### [equals()](https://immutable-js.github.io/immutable-js/docs/#/ValueObject/equals)

True if this and the other Collection have value equality, as defined by `Immutable.is()`.

```
equals(other: any): boolean 
```

#### DISCUSSION

Note: This is equivalent to `Immutable.is(this, other)`, but provided to allow for chained expressions.

### [hashCode()](https://immutable-js.github.io/immutable-js/docs/#/ValueObject/hashCode)

Computes and returns the hashed identity for this Collection.

```
hashCode(): number 
```

#### DISCUSSION

The `hashCode` of a Collection is used to determine potential equality, and is used when adding this to a `Set` or as a key in a `Map`, enabling lookup via a different instance.

```js
const { List, Set } = require('immutable');
const a = List([ 1, 2, 3 ]);
const b = List([ 1, 2, 3 ]);
assert.notStrictEqual(a, b); // different instances
const set = Set([ a ]);
assert.equal(set.has(b), true);
```

Note: `hashCode()` MUST return a `Uint32` number. The easiest way to guarantee this is to return `myHash | 0` from a custom implementation.

If two values have the same `hashCode`, they are [not guaranteed to be equal](http://en.wikipedia.org/wiki/Collision_(computer_science)). If two values have different `hashCode`s, they must not be equal.

Note: `hashCode()` is not guaranteed to always be called before `equals()`. Most but not all Immutable.js collections use hash codes to organize their internal data structures, while all Immutable.js collections use equality during lookups.