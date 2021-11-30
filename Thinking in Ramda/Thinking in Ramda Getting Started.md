# Thinking in Ramda: Getting Started

This post is the beginning of a new series about functional programming called [Thinking in Ramda](https://randycoulman.com/blog/categories/thinking-in-ramda/).

I’ll be using the [Ramda](http://ramdajs.com/) JavaScript library for this series, though many of the ideas apply to other JavaScript libraries such as [Underscore](http://underscorejs.org/) and [Lodash](https://lodash.com/) as well as to other languages.

I’m going to stick to the lighter, less-academic end of functional programming. This is mostly because I want to keep the series accessible for more people, but also partly because I’m not very far down the functional road myself.

## Ramda

I’ve talked about the [Ramda](http://ramdajs.com/) library for JavaScript on this blog a couple of times:

- In [Using Ramda With Redux](https://randycoulman.com/blog/2016/02/16/using-ramda-with-redux/), I showed some examples of how Ramda can be used in various contexts when writing [Redux](http://redux.js.org/) applications.
- In [Using Redux-api-middleware With Rails](https://randycoulman.com/blog/2016/04/19/using-redux-api-middleware-with-rails/), I used Ramda to help transform request and response payloads.

I find Ramda to be a nicely designed library that provides a lot of tools for doing functional programming in JavaScript in a clean, elegant way.

If you’d like to experiment with Ramda while reading through this series, there’s a handy [in-browser sandbox on the Ramda site](http://ramdajs.com/repl/).

## Functions

As the name might suggest, functional programming has a lot to do with functions. For our purpose, we will define a function as a reusable piece of code that is called with zero or more arguments and returns a result.

Here’s a simple function in JavaScript:

Simple Function

```js
function double(x) {
  return x * 2
}
```

With ES6 arrow functions, you can write the same function much more tersely. I mention this now, because we’ll be using a lot of arrow functions as we go along.

Simple ES6 Arrow Function

```js
const double = x => x * 2
```

Almost every language has some kind of support for functions.

Some languages go further and provide support for functions as first-class constructs. By “first-class”, I mean that functions can be used in the same way as other kinds of values. You can:

- refer to them from constants and variables
- pass them as parameters to other functions
- return them as results from other functions

JavaScript is one such language, and we’ll be taking advantage of that.

## Pure Functions

When writing functional programs, it eventually becomes important to work mostly with so-called “pure” functions.

Pure functions are functions that have no side-effects. They don’t assign to any outside variables, they don’t consume input, they don’t produce output, they don’t read from or write to a database, they don’t modify the parameters they’re passed, etc.

The basic idea is that, if you call a function with the same inputs over and over again, you always get the same result.

You can certainly do things with impure functions (and you must, if your program is going to do anything interesting), but for the most part you’ll want to keep most of your functions pure.

## Immutability

Another important concept in functional programming is that of “immutability”. What does that mean? “Immutable” means “unchangeable”.

When I’m working in an immutable fashion, once I initialize a value or an object I never change it again. That means no changing elements of an array or properties of an object.

If I need to change something in an array or object, I instead return a new copy of it with the changed value. Later posts will talk about this in great detail.

Immutability goes hand-in-hand with pure functions. Since pure functions aren’t allowed to have side-effects, they aren’t allowed to change outside data structures. They are forced to work with data in an immutable way.

## Where to Start?

The easiest way to start thinking functionally is to start replacing loops with collection-iteration functions.

If you come from another language that has these functions (Ruby and Smalltalk are but two examples), you might already be familiar with these.

Martin Fowler has a couple of great articles about “Collection Pipelines” that show [how to use these functions](http://martinfowler.com/articles/collection-pipeline/) and how to [refactor existing code into collection pipelines](http://martinfowler.com/articles/refactoring-pipelines.html).

Note that these functions are all (with the exception of `reject`) available on `Array.prototype`, so you don’t need Ramda to start using them. However, I’ll use the Ramda versions for consistency with the rest of the series.

### forEach

Rather than writing an explicit loop, try using the `forEach` function instead. That is:

forEach

```js
// Replace this:
for (const value of myArray) {
  console.log(value)
}
 // with:
forEach(value => console.log(value), myArray)
```

`forEach` takes a function and an array, and calls the function on each element of the array.

While `forEach` is the most approachable of these functions, it is used the least of any of them when doing functional programming. It doesn’t return a value, so is really only used for calling functions that have side-effects.

### map

The next most important function to learn is `map`. Like `forEach`, `map` applies a function to each element of an array. However, unlike `forEach`, map collects the results of applying the function into a new array and returns it.

Here’s an example:

map

```js
map(x => x * 2, [1, 2, 3])  // --> [2, 4, 6]
```

This is using an anonymous function, but we can just as easily use a named function here:

map with named function

```js
const double = x => x * 2
map(double, [1, 2, 3])
```

### filter/reject

Next up, let’s look at `filter` and `reject`. As its name might suggest, `filter` selects elements from an array based on some function. For example:

filter

```js
const isEven = x => x % 2 === 0
filter(isEven, [1, 2, 3, 4])  // --> [2, 4]
```

`filter` applies its function (`isEven` in this case) to each element of the array. Whenever the function returns a “truthy” value, the corresponding element is included in the result. Whenever the function returns a “falsy” value, the corresponding element is excluded (filtered out) from the array.

`reject` does exactly the same thing, but in reverse. It keeps the elements for which the function returns a falsy value and excludes the values for which it returns a truthy value.

reject

```js
reject(isEven, [1, 2, 3, 4]) // --> [1, 3]
```

### find

`find` applies a function to each element of an array and returns the first element for which the function returns a truthy value.

find

```js
find(isEven, [1, 2, 3, 4]) // --> 2
```

### reduce

`reduce` is a little bit more complicated than the other functions we’ve seen so far. It is worth knowing, but if you have trouble understanding it at first, don’t let that stop you. You can get a long way without understanding it.

`reduce` takes a two-argument function, and initial value, and the array to operate on.

The first argument to the function we pass in is called the “accumulator” and the second argument is the value from the array. The function needs to return a new accumulator value.

Let’s look at an example and then walk through what’s going on.

reduce

```js
const add = (accum, value) => accum + value
reduce(add, 5, [1, 2, 3, 4]) // --> 15
```

1. `reduce` first calls the function (`add`) with the initial value (`5`) and the first element of the array (`1`). `add` returns a new accumulator value (`5 + 1 = 6`).
2. `reduce` calls `add` again, this time with the new accumulator value (`6`) and the next value from the array (`2`). `add` returns `8`.
3. `reduce` calls `add` again with `8` and the next value (`3`), resulting in `11`.
4. `reduce` calls `add` one last time with `11` and the last value of the array (`4`), resulting in `15`.
5. `reduce` returns the final accumulated value as its result (`15`).

## Conclusion

By starting with these collection-iteration functions, you can get used to the idea of passing functions to other functions. You might have used these in other languages without realizing you were doing some functional programming.

## Next

The next post in this series, [Combining Functions](https://randycoulman.com/blog/2016/05/31/thinking-in-ramda-combining-functions/), shows how we can take the next step and start combining functions in new and interesting ways.