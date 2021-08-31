# Thinking in Ramda: Lenses



This post is Part 8 of a series about functional programming called [Thinking in Ramda](https://randycoulman.com/blog/categories/thinking-in-ramda/).

In [Part 6](https://randycoulman.com/blog/2016/06/28/thinking-in-ramda-immutability-and-objects/) and [Part 7](https://randycoulman.com/blog/2016/07/05/thinking-in-ramda-immutability-and-arrays/), we learned how to read, update, and transform object properties and array elements in a declarative, immutable way.

Ramda provides a more general tool for performing these operations, the lens.

## What is a Lens?

A lens combines a “getter” function and a “setter” function into a single unit. Ramda provides a set of functions for working with lenses.

We can think of a lens as something that focuses on a specific part of a larger data structure.

## How Do I Create a Lens?

The most generic way to create a lens in Ramda is with the `lens` function. `lens` takes a getter function and a setter function and returns the new lens.

Creating a Lens

```js
const person = {
  name: 'Randy',
  socialMedia: {
    github: 'randycoulman',
    twitter: '@randycoulman'
  }
}
const nameLens = lens(prop('name'), assoc('name'))
const twitterLens = lens(
  path(['socialMedia', 'twitter']),
  assocPath(['socialMedia', 'twitter'])
)
```

Here we’re using `prop` and `path` as our getter functions and `assoc` and `assocPath` as our setter functions.

Note that we had to duplicate the property and path arguments to these functions. Fortunately, Ramda provides nice shortcuts for the most common uses of lenses: `lensProp`, `lensPath`, and `lensIndex`.

- `lensProp` creates a lens that focuses on a property of an object.
- `lensPath` creates a lens that focuses on a nested property of an object.
- `lensIndex` creates a lens that focuses on an element of an array.

We could rewrite our lenses above with `lensProp` and `lensPath`:

Using lensProp and lensPath

```js
const nameLens = lensProp('name')
const twitterLens = lensPath(['socialMedia', 'twitter'])
```

That’s a lot simpler and gets rid of the duplication. In practice, I find that I almost never need to use the generic `lens` function.

## What Can I Do With It?

OK, great, we’ve created some lenses. What can we do with them?

Ramda provides three functions for working with lenses.

- `view` reads the value of the lens.
- `set` updates the value of the lens.
- `over` applies a transformation function to the lens.

Using lenses

```js
view(nameLens, person) // => 'Randy'
set(twitterLens, '@randy', person)
// => {
//   name: 'Randy',
//   socialMedia: {
//     github: 'randycoulman',
//     twitter: '@randy'
//   }
// }
over(nameLens, toUpper, person)
// => {
//   name: 'RANDY',
//   socialMedia: {
//     github: 'randycoulman',
//     twitter: '@randycoulman'
//   }
// }
```

Notice that `set` and `over` return the entire object with the lens’ focused property modified as specified.

## Conclusion

Lenses can be handy if we have a somewhat complex data structure that we want to abstract away from calling code. Rather than exposing the structure or providing a getter, setter, and transformer for every accessible property, we can instead expose lenses.

Client code can then work with our data structure using `view`, `set`, and `over` without being coupled to the exact shape of the structure.

## Next

We’ve now learned about a lot of what Ramda provides; certainly enough to do most of what we need to do in our programs. [Wrap-up](https://randycoulman.com/blog/2016/07/19/thinking-in-ramda-wrap-up/) reviews the series and mentions some other topics we might want to explore on our own.