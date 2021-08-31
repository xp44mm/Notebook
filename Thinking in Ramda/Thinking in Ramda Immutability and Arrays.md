# Thinking in Ramda: Immutability and Arrays



This post is Part 7 of a series about functional programming called [Thinking in Ramda](https://randycoulman.com/blog/categories/thinking-in-ramda/).

In [Part 6](https://randycoulman.com/blog/2016/06/28/thinking-in-ramda-immutability-and-objects/), we talked about working with JavaScript objects in a functional and immutable way.

In this post, we’ll do the same for arrays.

## Reading Array Elements

In [Part 6](https://randycoulman.com/blog/2016/06/28/thinking-in-ramda-immutability-and-objects/), we learned about various Ramda functions for reading object properties, including `prop`, `pick`, and `has`. Ramda has even more methods for reading array elements.

The array equivalent of `prop` is `nth`; the equivalent of `pick` is `slice`, and the equivalent of `has` is `contains`. Let’s look at those.

Reading Array Elements

```js
const numbers = [10, 20, 30, 40, 50, 60]
nth(3, numbers) // => 40  (0-based indexing)
nth(-2, numbers) // => 50 (negative numbers start from the right)
slice(2, 5, numbers) // => [30, 40, 50] (see below)
contains(20, numbers) // => true
```

Slice takes two indexes and returns the sub array starting at the first index (0-based) and including all of the elements up to, but not including the second index.

Accessing the first (`nth(0)`) and last (`nth(-1)`) elements is quite common, so Ramda provides shortcuts for those special cases, `head` and `last`. It also provides functions for accessing all-but-the-first element (`tail`), all-but-the-last element (`init`), the first `N` elements (`take(N)`), and the last `N` elements (`takeLast(N)`). Let’s see these in action.

More Array Reading

```js
const numbers = [10, 20, 30, 40, 50, 60]
head(numbers) // => 10
tail(numbers) // => [20, 30, 40, 50, 60]
last(numbers) // => 60
init(numbers) // => [10, 20, 30, 40, 50]
take(3, numbers) // => [10, 20, 30]
takeLast(3, numbers) // => [40, 50, 60]
```

## Adding, Updating, and Removing Array Elements

For objects, we learned about `assoc`, `dissoc`, and `omit` for adding, updating, and removing properties.

Because arrays are an ordered data structure, there are several methods that do the same job as `assoc`. The most general are `insert` and `update`, but Ramda also provides `append` and `prepend` for the common case of adding elements at the beginning or end of the array. `insert`, `append`, and `prepend` add new elements to the array; `update` “replaces” an element with a new value.

As you might expect from a functional library, all of these functions return a new array with the desired changes; the original array is never changed.

Adding and Updating Elements

```js
const numbers = [10, 20, 30, 40, 50, 60]
insert(3, 35, numbers) // => [10, 20, 30, 35, 40, 50, 60]
append(70, numbers) // => [10, 20, 30, 40, 50, 60, 70]
prepend(0, numbers) // => [0, 10, 20, 30, 40, 50, 60]
update(1, 15, numbers) // => [10, 15, 30, 40, 50, 60]
```

For combining two objects into one, we learned about `merge`. Ramda provides `concat` for doing the same with arrays.

concat

```js
const numbers = [10, 20, 30, 40, 50, 60]
concat(numbers, [70, 80, 90]) // => [10, 20, 30, 40, 50, 60, 70, 80, 90]
```

Note that the second array is appended to the first. This looks right when used by itself, but as with `merge`, it may not do what you expect when used in a pipeline. I find it useful to define a helper function, `concatAfter`: 

```js
const concatAfter = flip(concat)
```

 for use in pipelines.

Ramda also provides several options for removing elements. `remove` removes elements by index, while `without` removes them by value. There’s also `drop` and `dropLast` for the common case of removing elements from the beginning or end of the array.

Removing Elements

```js
const numbers = [10, 20, 30, 40, 50, 60]
remove(2, 3, numbers) // => [10, 20, 60]
without([30, 40, 50], numbers) // => [10, 20, 60]
drop(3, numbers) // => [40, 50, 60]
dropLast(3, numbers) // => [10, 20, 30]
```

Note that `remove` takes an index and a count whereas `slice` takes two indexes. This inconsistency can be confusing if you’re not aware of it.

## Transforming Elements

As with objects, we may want to `update` an array element by applying a function to the original value.

Transforming an Element the Hard Way

```js
const numbers = [10, 20, 30, 40, 50, 60]
update(2, multiply(10, nth(2, numbers)), numbers) // => [10, 20, 300, 40, 50, 60]
```

To simplify this common use case, Ramda provides `adjust` that works much like `evolve` does for objects. Unlike `evolve`, `adjust` only works for a single array element.

Using adjust

```js
const numbers = [10, 20, 30, 40, 50, 60]
adjust(multiply(10), 2, numbers)
```

Note that the first two arguments to `adjust` are swapped when compared with `update`. This can be a source of confusion, but makes sense when you consider partial application. You might want to provide an adjustment function with `adjust(multiply(10))` and then later decide which index and array to apply that adjustment to.

## Conclusion

We now have tools for working with arrays and objects in a declarative and immutable way. This allows us to build programs out of small, functional building blocks, combining functions to do what we need to do, all without mutating our data structures.

## Next

We’ve learned ways of reading, updating, and transforming object properties and array elements. Ramda provides a more general tool for performing these operations, the lens. [Lenses](https://randycoulman.com/blog/2016/07/12/thinking-in-ramda-lenses/) shows us how they work.