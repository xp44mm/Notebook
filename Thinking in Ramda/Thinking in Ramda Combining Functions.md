# Thinking in Ramda: Combining Functions

This post is Part 2 of a series about functional programming called [Thinking in Ramda](https://randycoulman.com/blog/categories/thinking-in-ramda/).

In [Part 1](https://randycoulman.com/blog/2016/05/24/thinking-in-ramda-getting-started/), I introduced Ramda and some of the basic ideas about functional programming, such as functions, pure functions, and immutability. I then suggested that a good place to start is with the collection-iteration functions such as `forEach`, `map`, `filter`, `reduce` and friends.

## Simple Combinations

Once you’ve gotten used to the idea of passing functions to other functions, you might start to find situations where you want to combine several functions together.

Ramda provides several functions for doing simple combinations. Let’s look at a few.

### Complement

In the last post, we used `find` to find the first even number in a list:

find

```js
const isEven = x => x % 2 === 0
find(isEven, [1, 2, 3, 4]) // --> 2
```

What if we wanted to find the first odd number instead. We could always write an `isOdd` function and use it, but we know that any number that isn’t even is odd. Let’s reuse our `isEven` function.

Ramda provides a higher-order function, `complement`, that takes another function and returns a new function that returns `true` when the original function returns a falsy value, and `false` when the original function returns a truthy value.

find with complement

```js
const isEven = x => x % 2 === 0
find(complement(isEven), [1, 2, 3, 4]) // --> 1
```

Even better is to give the `complement`ed function its own name so it can be reused:

isOdd with complement

```js
const isEven = x => x % 2 === 0
const isOdd = complement(isEven)
find(isOdd, [1, 2, 3, 4]) // --> 1
```

Note that `complement` implements the same idea for functions as the `!` (not) operator does for values.

### Both/Either

Let’s say we’re working on a voting system. Given a person, we’d like to be able to determine if that person is eligible to vote. Based on our current knowledge, a person must be at least 18 years old and be a citizen in order to be able to vote. Someone is a citizen if they were born in the country or if they later became a citizen through naturalization.

Eligible Voters

```js
const wasBornInCountry = person => person.birthCountry === OUR_COUNTRY
const wasNaturalized = person => Boolean(person.naturalizationDate)
const isOver18 = person => person.age >= 18
const isCitizen = person => wasBornInCountry(person) || wasNaturalized(person)
const isEligibleToVote = person => isOver18(person) && isCitizen(person)
```

What we’ve written above works, but Ramda provides a couple of handy functions to help us clean it up a bit.

`both` takes two other functions and returns a new function that returns `true` if both functions return a truthy value when applied to the arguments and `false` otherwise.

`either` takes two other functions and returns a new function that returns `true` if either function returns a truthy value when applied to the arguments and `false` otherwise.

Using these two functions, we can simplify `isCitizen` and `isEligibleToVote`:

Using both and either

```js
const isCitizen = either(wasBornInCountry, wasNaturalized)
const isEligibleToVote = both(isOver18, isCitizen)
```

Note that `both` implements the same idea for functions as the `&&` (and) operator does for values, and `either` implements that same idea for functions as the `||` (or) operator does for values.

Ramda also provides `allPass` and `anyPass` that take an array of any number of functions. As their names suggest, `allPass` works like `both`, and `anyPass` works like `either`.

## Pipelines

Sometimes we want to apply several functions to some data in a pipeline fashion. For example, we might want to take two numbers, multiply them together, add one, and square the result. We could write it like this:

Pipeline the hard way

```js
const multiply = (a, b) => a * b
const addOne = x => x + 1
const square = x => x * x
const operate = (x, y) => {
  const product = multiply(x, y)
  const incremented = addOne(product)
  const squared = square(incremented)
  return squared
}
operate(3, 4) // => ((3 * 4) + 1)^2 => (12 + 1)^2 => 13^2 => 169
```

Notice how each operation is applied to the result of the previous one.

### pipe

Ramda provides the `pipe` function, which takes a list of one or more functions and returns a new function.

The new function takes the same number of arguments as the first function it is given. It then “pipes” those arguments through each function in the list. It applies the first function to the arguments, passes its result to the second function and so on. The result of the last function is the result of the `pipe` call.

Note that all of the functions after the first must only take a single argument.

Knowing this, we can use `pipe` to simplify our `operate` function:

Using pipe

```
const operate = pipe(
  multiply,
  addOne,
  square
)
```

When we call `operate(3, 4)`, `pipe` passes the `3` and `4` to the `multiply` function, resulting in `12`. It passes that `12` to `addOne`, which returns `13`. It then passes that `13` to `square`, which returns `169`, and that becomes the final result of `operate`.

### compose

Another way we could have written our original `operate` function is to inline all of the temporary variables:

Inlined Pipeline

```js
const operate = (x, y) => square(addOne(multiply(x, y)))
```

That’s much more compact, but somewhat harder to read. In that form, however, it lends itself to be rewritten using Ramda’s `compose` function.

`compose` works exactly the same way as `pipe`, except that it applies the functions in right-to-left order instead of left-to-right. Let’s write `operate` with `compose`:

Using compose

```js
const operate = compose(
  square,
  addOne,
  multiply
)
```

This is exactly the same as `pipe` above, but with the functions in the opposite order. In fact, Ramda’s `compose` function is written in terms of `pipe`.

I always think of `compose` this way: `compose(f, g)(value)` is equivalent to `f(g(value))`.

As with `pipe`, note that all of the functions except the last must only take a single argument.

### compose or pipe?

I think that `pipe` is probably the easiest to understand when coming from a more imperative background since you read the functions left-to-right. But `compose` is a bit easier to translate to nested-function form as I showed above.

I haven’t yet developed a good rule for when I prefer `compose` and when I prefer `pipe`. Since they are essentially equivalent in Ramda, it probably doesn’t matter which one you choose. Just go with whichever one reads the best in your situation.

## Conclusion

By combining several functions in specific ways, we can start to write more powerful functions.

## Next

You may have noticed that we mostly ignored the function arguments when we were combining functions. We only supply the arguments when we finally call the combined function.

This is common in functional programming, and we talk about that a lot more in the next post in this series, [Partial Application](https://randycoulman.com/blog/2016/06/07/thinking-in-ramda-partial-application/). We also talk about how to combine functions that take more than one argument.