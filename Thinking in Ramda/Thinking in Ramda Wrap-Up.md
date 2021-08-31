# Thinking in Ramda: Wrap-Up



This post completes a series about functional programming called [Thinking in Ramda](https://randycoulman.com/blog/categories/thinking-in-ramda/).

Over the past eight posts, we’ve been talking about the [Ramda JavaScript library](http://ramdajs.com/) that provides functions for working with JavaScript in a functional, declarative, and immutable way.

During the series, we learned that Ramda has some underlying principles that drive its API:

- Data last: Almost all of the functions take the data parameter as the last parameter.
- Currying: Almost every function in Ramda is “curried”. That is, you can call a function with only a subset of its required arguments, and it will return a new function that takes the remaining arguments. Once all of the arguments are provided, the original function is invoked.

These two principles allow us to write very clear functional code that combines basic building blocks into more powerful operations.

## Summary

For reference, here’s a quick summary of the series.

- [Getting Started](https://randycoulman.com/blog/2016/05/24/thinking-in-ramda-getting-started/) introduces us to the idea of functions, pure functions, and immutability. It then gets us started by looking at the collection iteration functions like `map`, `filter`, and `reduce`.
- [Combining Functions](https://randycoulman.com/blog/2016/05/31/thinking-in-ramda-combining-functions/) shows us that we can combine functions in various ways using tools such as `both`, `either`, `pipe`, and `compose`.
- [Partial Application](https://randycoulman.com/blog/2016/06/07/thinking-in-ramda-partial-application/) helps us discover that it can be useful to only supply some of the arguments to a function, allowing a later function to supply the rest. We use `partial` and `curry` to help us with this and learn about `flip` and the placeholder (`__`).
- [Declarative Programming](https://randycoulman.com/blog/2016/06/14/thinking-in-ramda-declarative-programming/) teaches us about the difference between imperative and declarative programming. We learn how to use Ramda’s declarative replacements for arithmetic, comparisons, logic, and conditionals.
- [Pointfree Style](https://randycoulman.com/blog/2016/06/21/thinking-in-ramda-pointfree-style/) introduces us to the idea of pointfree style, also known as tacit programming. In pointfree style, we don’t actually see the data parameter that we’re operating on; it’s implicit. Our programs are made up of small, simple building blocks that are combined together to do what we need. Only at the end do we apply our compound functions to the actual data.
- [Immutability and Objects](https://randycoulman.com/blog/2016/06/28/thinking-in-ramda-immutability-and-objects/) returns us to the idea of working declaratively, this time giving us the tools we need to read, update, delete, and transform properties of objects.
- [Immutability and Arrays](https://randycoulman.com/blog/2016/07/05/thinking-in-ramda-immutability-and-arrays/) continues the theme and shows us how to do the same for arrays.
- [Lenses](https://randycoulman.com/blog/2016/07/12/thinking-in-ramda-lenses/) concludes by introducing the concept of a lens, a construct that allows us to focus on a small part of a larger data structure. Using the `view`, `set`, and `over` functions, we can read, update, and transform the focused value in the context of its larger data structure.

## What Next?

We didn’t cover every part of Ramda in this series. In particular, we didn’t talk about functions for working with strings, and we didn’t talk about more advanced concepts such as [transducers](http://ramdajs.com/0.21.0/docs/#transduce).

To learn more about what Ramda can do, I recommend perusing the [documentation](http://ramdajs.com/docs/). There’s a wealth of information there. All of the functions are grouped by the type of data they work with, though there is some overlap. For example, several of the array functions will also work on strings, and `map` works on both arrays and objects.

If you’re interested in more advanced functional topics, here are some places you can go:

- Transducers: There’s a [good introductory article](http://simplectic.com/blog/2015/ramda-transducers-logs/) on parsing logs with transducers.
- Algebraic Data Types: If you’ve read much about functional programming, you’ll have heard about algebraic types and terms such as “Functor”, “Applicative”, and “Monad.” If you’re interested in exploring these ideas in the context of Ramda, Specifically, we suggest these:

* Maybe: [sanctuary-js/sanctuary-maybe](https://github.com/sanctuary-js/sanctuary-maybe)
* Either: [sanctuary-js/sanctuary-either](https://github.com/sanctuary-js/sanctuary-either)
* Future: [fluture-js/Fluture](https://github.com/fluture-js/Fluture)
* State: [fantasyland/fantasy-states](https://github.com/fantasyland/fantasy-states)
* Tuple: [fantasyland/fantasy-tuples](https://github.com/fantasyland/fantasy-tuples)
* Reader: [fantasyland/fantasy-readers](https://github.com/fantasyland/fantasy-readers)
* IO: [fantasyland/fantasy-io](https://github.com/fantasyland/fantasy-io)
* Identity: [sanctuary-js/sanctuary-identity](https://github.com/sanctuary-js/sanctuary-identity)
