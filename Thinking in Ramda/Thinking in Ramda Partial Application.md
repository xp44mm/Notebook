# Thinking in Ramda: Partial Application



This post is Part 3 of a series about functional programming called [Thinking in Ramda](https://randycoulman.com/blog/categories/thinking-in-ramda/).

In [Part 2](https://randycoulman.com/blog/2016/05/31/thinking-in-ramda-combining-functions/), we talked about combining functions in various ways, ending up with the `compose` and `pipe` functions that allow us to apply a series of functions in a pipeline fashion.

In that post, we looked at simple pipelines of functions that only take one argument. But what if we want to use functions that take more than one argument?

For example, let’s say we have a collection of book objects and we want to find the titles of all of the books published in a given year. Let’s write that using only Ramda’s collection iteration functions:

First Attempt

```js
const publishedInYear = (book, year) => book.year === year
const titlesForYear = (books, year) => {
    const selected = filter(book => publishedInYear(book, year), books)
    return map(book => book.title, selected)
}
```

It would be nice to combine the `filter` and `map` into a pipeline, but we don’t know how to do that yet because `filter` and `map` both take two arguments.

It would also be nice if we didn’t need to use an arrow function in `filter`. Let’s tackle that problem first because it will teach us some things we can use to make the pipeline.

## Higher-Order Functions

In [Part 1 of this series](https://randycoulman.com/blog/2016/05/24/thinking-in-ramda-getting-started/), we talked about functions as first-class constructs. First-class functions can be passed as parameters to other functions and returned as results from other functions. We’ve been doing a lot of the former, but haven’t seen the latter yet.

Functions that take or return other functions are known as “higher-order functions”.

In the example above, we pass an arrow function to `filter`: `book => publishedInYear(book, year)`, and we’d like to try to get rid of the arrow. In order to do that, we need a function that takes a book and returns true if the book was published in a given year. But we also need to pass along the year to make this flexible.

The way we can solve this is to change `publishedInYear` into a function that returns another function. I’ll write it with full function syntax so you can see what’s going on, but then show you the shorter version using the arrow syntax:

Higher-Order Function

```js
// Full function version:
function publishedInYear(year) {
  return function(book) {
    return book.year === year
  }
}
// Arrow function version:
const publishedInYear = year => book => book.year === year
```

With this new version of `publishedInYear`, we can rewrite our `filter` call, eliminating the arrow function:

With Higher-Order Function

```js
const publishedInYear = year => book => book.year === year
const titlesForYear = (books, year) => {
   const selected = filter(publishedInYear(year), books)
   return map(book => book.title, selected)
}
```

Now, when we call `filter`, `publishedInYear(year)` is evaluated immediately, returning a function that takes a `book`, which is exactly what `filter` needs.

## Partially-Applying Functions

We could rewrite any multi-argument function in this way if we wanted to, but we don’t own all of the functions we might want to use. Also, we might want to use some multi-argument functions in the usual way.

For example, if we had some other code that just wanted to check if a book was published in a given year, we’d like to say `publishedInYear(book, 2012)`, but we can’t do that any more. Instead, we have to say `publishedInYear(2012)(book)`. That’s less readable and more annoying.

Fortunately, Ramda provides two functions to help us out: `partial`, and `partialRight`.

These two functions let us call any function with fewer arguments than it needs. They both return a new function that takes the missing arguments and then calls the original function once all of the arguments have been supplied.

The difference between `partial` and `partialRight` is whether the arguments we supply are the left-most or right-most arguments needed by the original function.

Let’s go back to our original example and use one of these functions instead of re-writing `publishedInYear`. Since we want to supply only the year, and that is the right-most argument, we need to use `partialRight`.

Using partialRight

```js
const publishedInYear = (book, year) => book.year === year
const titlesForYear = (books, year) => {
   const selected = filter(partialRight(publishedInYear, [year]), books)
   return map(book => book.title, selected)
}
```

If we had originally written `publishedInYear` to take `(year, book)` instead of `(book, year)`, we would have used `partial` instead of `partialRight`.

Note that the arguments we supply to `partial` and `partialRight` must always be in an array, even if there’s only one of them. I can’t tell you how many times I’ve forgotten that and ended up with a confusing error message:

Confusing Error Message

```
First argument to _arity must be a non-negative integer no greater than ten
```

## Curry

Having to use `partial` and `partialRight` everywhere gets verbose and tedious. But having to call multiple-argument functions as a series of single-argument functions is equally bad.

Fortunately, Ramda provides us with a solution: `curry`.

Currying is another core concept in functional programming. Technically, a curried function is always a series of single-argument functions, which is what I was just complaining about. In pure functional languages, the syntax generally makes that look no different than calling a function with multiple arguments.

But because Ramda is a JavaScript library, and JavaScript doesn’t have nice syntax for calling a series of single-argument functions, the authors have relaxed the traditional definition of currying a bit.

In Ramda, a curried function can be called with only a subset of its arguments, and it will return a new function that accepts the remaining arguments. If you call a curried function with all of its arguments, it will call just call the function.

You can think of a curried function as the best of both worlds: you can call it normally with all of its arguments and it will just work. Or you can call it with a subset of its arguments, and it will act as if you’d used `partial`.

Note that this flexibility introduces a small performance hit, because `curry` needs to figure out how the function was called and then determine what to do. In general, I only curry functions when I find I need to use `partial` in more than one place.

Let’s take advantage of `curry` with our `publishedInYear` function. Note that `curry` always works as if you had used `partial`; there isn’t a `partialRight` version. We’ll talk about that more below, but for now, we’re going to reverse the arguments to `publishedInYear` so that the year comes first.

Using curry

```js
const publishedInYear = curry((year, book) => book.year === year)
const titlesForYear = (books, year) => {
   const selected = filter(publishedInYear(year), books)
   return map(book => book.title, selected)
}
```

We can once again call `publishedInYear` with just the year and get back a function that then takes a book and executes our original function. However, we can still call it normally as `publishedInYear(2012, book)` without the annoying `)(` syntax. The best of both worlds!

## Argument Order

Notice that to make `curry` work for us, we had to reverse the argument order. This is extremely common with functional programming, so almost every Ramda function is written so that the data to be operated on comes last.

You can think of the earlier parameters as configuration for the operation. So, for `publishedInYear`, the `year` parameter is the configuration (what year are we looking for?) and the `book` parameter is the data (where are we looking for it?).

We’ve already seen examples of this with the collection iteration functions. They all take the collection as the last argument because it makes this style of programming easier.

## Arguments in the Wrong Order

What if we had left the argument order of `publishedInYear` alone? How could we still take advantage of its curried nature?

Ramda provides a couple of options.

### Flip

The first option is `flip`. `flip` takes a function of 2 or more arguments and returns a new function that takes the same arguments, but switches the order of the first two arguments. It is mostly used with two argument functions, but is more general than that.

Using `flip`, we can go back to the original argument order for `publishedInYear`:

Using flip

```js
const publishedInYear = curry((book, year) => book.year === year)
const titlesForYear = (books, year) => {
   const selected = filter(flip(publishedInYear)(year), books)
   return map(book => book.title, selected)
}
```

In most cases, I’d prefer to use the more convenient argument order, but if you need to use a function you don’t control, `flip` is a helpful option.

### Placeholder

The more general option is the “placeholder” argument (`__`).

What if we have a curried function of three arguments, and we want to supply the first and last arguments, leaving the middle one for later? We can use the placeholder for the middle argument:

Middle Argument Later

```js
const threeArgs = curry((a, b, c) => { /* ... */ })
const middleArgumentLater = threeArgs('value for a', __, 'value for c')
```

You can also use the placeholder more than once in a call. For example, what if wanted to supply only the middle argument?

Only the Middle Argument

```js
const threeArgs = curry((a, b, c) => { /* ... */ })
const middleArgumentOnly = threeArgs(__, 'value for b', __)
```

We can use the placeholder style instead of `flip` if we like:

Using the Placeholder

```js
const publishedInYear = curry((book, year) => book.year === year)
const titlesForYear = (books, year) => {
   const selected = filter(publishedInYear(__, year), books)
   return map(book => book.title, selected)
}
```

I find this version more readable, but if I needed to use the “flipped” version of `publishedInYear` a lot, I might define a helper function using `flip`, and then use the helper function everywhere. We’ll see some examples of this in future posts.

Note that `__` only works for curried functions, while `partial`, `partialRight`, and `flip` all work on any function. If you need to use `__` with a normal function, you can always wrap it with a call to `curry` first.

## Let’s Make a Pipeline

Let’s see if we can now move our `filter` and `map` calls into a pipeline. Here’s the current state of the code, with the convenient argument order for `publishedInYear`:

Where We're At

```js
const publishedInYear = curry((year, book) => book.year === year)
const titlesForYear = (books, year) => {
   const selected = filter(publishedInYear(year), books)
   return map(book => book.title, selected)
}
```

We learned about `pipe` and `compose` in the last post, but we need one more piece of information so that we can take advantage of that learning.

The missing piece of information is this: almost every Ramda function is curried by default. This includes `filter` and `map`. So `filter(publishedInYear(year))` is perfectly legal and returns a new function that’s just waiting for us to pass the `books` along later, as is `map(book => book.title)`.

And now we can write the pipeline:

The Pipeline

```js
const publishedInYear = curry((year, book) => book.year === year)
const titlesForYear = (books, year) =>
  pipe( //*
    filter(publishedInYear(year)),
    map(book => book.title)
  )(books)
```

Let’s take this one step further and reverse the arguments to `titlesForYear` to match Ramda’s conventions of data-last. We can also curry the function to allow its use in later pipelines.

Ramda-fying

```js
const publishedInYear = curry((year, book) => book.year === year)
const titlesForYear = curry((year, books) => // swaped
  pipe(
    filter(publishedInYear(year)),
    map(book => book.title)
  )(books)
)
```

## Conclusion

This post is probably the deepest one of this series. Partial application and currying can take some time and effort to wrap your head around. But once you “get” them, they introduce you to a very powerful way of transforming your data in a functional way.

They lead you to start building transformations by creating pipelines of small, simple building blocks.

## Next

In order to write code in a functional style, we need to start thinking “declaratively” instead of “imperatively”. To do that, we’re going to need to find ways of expressing the imperative constructs we’re used to in a functional way. [Declarative Programming](https://randycoulman.com/blog/2016/06/14/thinking-in-ramda-declarative-programming/) discusses these ideas.