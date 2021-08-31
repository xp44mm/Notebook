# Thinking in Ramda: Immutability and Objects



This post is Part 6 of a series about functional programming called [Thinking in Ramda](https://randycoulman.com/blog/categories/thinking-in-ramda/).

In [Part 5](https://randycoulman.com/blog/2016/06/21/thinking-in-ramda-pointfree-style/), we talked about writing our functions in “pointfree” or “tacit” style where the main data argument to our function is not explicitly shown.

At that time, we were unable to convert all of our functions to pointfree style because we were missing some tools. It’s now time to learn those tools.

## Reading Object Properties

Let’s look back at the eligible voters example that we revisited in [Part 5](https://randycoulman.com/blog/2016/06/21/thinking-in-ramda-pointfree-style/):

Eligible Voters

```js
const wasBornInCountry = person => person.birthCountry === OUR_COUNTRY
const wasNaturalized = person => Boolean(person.naturalizationDate)
const isOver18 = person => person.age >= 18
const isCitizen = either(wasBornInCountry, wasNaturalized)
const isEligibleToVote = both(isOver18, isCitizen)
```

As you can see, we’ve made `isCitizen` and `isEligibleToVote` pointfree, but we couldn’t do that with the first three functions.

As we learned in [Part 4](https://randycoulman.com/blog/2016/06/14/thinking-in-ramda-declarative-programming/), we can make the functions more declarative using `equals` and `gte`. Let’s start there.

Using `equals` and `gte`

```js
const wasBornInCountry = person => equals(person.birthCountry, OUR_COUNTRY)
const wasNaturalized = person => Boolean(person.naturalizationDate)
const isOver18 = person => gte(person.age, 18)
```

In order to make these functions pointfree, we need a way to build up a function that we can then apply to the `person` at the end. The problem is that we need to access properties on the `person` and the only way we know how to do that is imperatively.

### prop

Fortunately, Ramda can help us out. It provides the `prop` function for accessing properties of an object.

Using `prop`, we can turn `person.birthCountry` into `prop('birthCountry', person)`. Let’s start with that.

Using prop

```js
const wasBornInCountry = person => equals(prop('birthCountry', person), OUR_COUNTRY)
const wasNaturalized = person => Boolean(prop('naturalizationDate', person))
const isOver18 = person => gte(prop('age', person), 18)
```

Wow, that looks a lot worse now. But let’s keep going with the refactoring. First, let’s swap the order of the arguments we’re passing to `equals` so that `prop` comes last. `equals` works the same with either order, so this is safe.

Swap equals argument order

```js
const wasBornInCountry = person => equals(OUR_COUNTRY, prop('birthCountry', person))
const wasNaturalized = person => Boolean(prop('naturalizationDate', person))
const isOver18 = person => gte(prop('age', person), 18)
```

Next, let’s use the curried nature of `equals` and `gte` to make new functions that we apply to the result of the `prop` calls.

Apply Currying

```js
const wasBornInCountry = person => equals(OUR_COUNTRY)(prop('birthCountry', person))
const wasNaturalized = person => Boolean(prop('naturalizationDate', person))
const isOver18 = person => gte(__, 18)(prop('age', person))
```

That’s still a bit worse, but let’s keep going anyway. Let’s take advantage of currying again with all of the `prop` calls.

More Currying

```js
const wasBornInCountry = person => equals(OUR_COUNTRY)(prop('birthCountry')(person))
const wasNaturalized = person => Boolean(prop('naturalizationDate')(person))
const isOver18 = person => gte(__, 18)(prop('age')(person))
```

Worse again. But now we see a familiar pattern. All three of our functions have the same shape as `f(g(person))`, and we know from [Part 2](https://randycoulman.com/blog/2016/05/31/thinking-in-ramda-combining-functions/) that this is equivalent to `compose(f, g)(person)`.

Let’s take advantage of that.

Using compose

```js
const wasBornInCountry = person => compose(equals(OUR_COUNTRY), prop('birthCountry'))(person)
const wasNaturalized = person => compose(Boolean, prop('naturalizationDate'))(person)
const isOver18 = person => compose(gte(__, 18), prop('age'))(person)
```

Now we’re getting somewhere. Now all three functions look like `person => f(person)`. We know from [Part 5](https://randycoulman.com/blog/2016/06/21/thinking-in-ramda-pointfree-style/) that we can make these functions pointfree.

Finally Pointfree

```js
const wasBornInCountry = compose(equals(OUR_COUNTRY), prop('birthCountry'))
const wasNaturalized = compose(Boolean, prop('naturalizationDate'))
const isOver18 = compose(gte(__, 18), prop('age'))
```

It wasn’t obvious when we started that our methods were doing two different things. They were both accessing a property of an object and performing some operation on the value of that property. This refactoring to pointfree style has made that very explicit.

Let’s look at some more tools that Ramda provides for working with objects.

### pick

Where `prop` reads a single property from an object and returns the value, `pick` reads multiple properties from an object and returns a new object with just those properties.

For example, if wanted just the names and ages of a person, we could use

```js
pick(['name', 'age'], person)
```



### has

If we just want to know if an object has a property without reading the value, we can use `has` for checking own properties, and `hasIn` for checking up the prototype chain: 

```js
has('name', person)
```



### path

Where `prop` reads a property from an object, `path` dives into nested objects. For example, we could access the zip code from a deeper structure as

```js
path(['address', 'zipCode'], person)
```

Note that `path` is more forgiving than `prop`. `path` will return `undefined` if anything along the path (including the original argument) is `null` or `undefined` whereas `prop` will raise an error.

### propOr / pathOr

`propOr` and `pathOr` are similar to `prop` and `path` combined with `defaultTo`. They let you provide a default value to use if the property or path cannot be found in the target object.

For example, we can provide a placeholder when we don’t know a person’s name:

```js
propOr('<Unnamed>', 'name', person)
```

Note that unlike `prop`, `propOr` will not raise an error if `person` is `null` or `undefined`; it will instead return the default value.

### keys / values

`keys` returns an array containing the names of all of the own properties in an object. `values` returns the values of those properties. These functions can be useful when combined with the collection iteration functions we learned about in [Part 1](https://randycoulman.com/blog/2016/05/24/thinking-in-ramda-getting-started/).

## Adding, Updating, and Removing Properties

We now have lots of tools for reading from objects declaratively, but what about when we want to make changes?

Since immutability is important, we don’t want to change objects directly. Instead, we want to return new objects that have been changed in the way we want.

Once again, Ramda provides a lot of help for us.

### assoc / assocPath

When programming imperatively, we could set or change the name of a person with the assignment operator: 

```js
person.name = 'New name'
```

In our functional, immutable world we use `assoc` instead: 

```js
const updatedPerson = assoc('name', 'New name', person)
```

`assoc` returns a new object with the added or updated property value, leaving the original object unchanged.

There is also `assocPath` for updating a nested property: 

```js
const updatedPerson = assocPath(['address', 'zipcode'], '97504', person)
```



### dissoc / dissocPath / omit

What about deleting properties? Imperatively, we might want to say

```js
delete person.age
```

In Ramda, we’d use `dissoc`: 

```js
const updatedPerson = dissoc('age', person)
```

`dissocPath` is similar, but works deeper into the object structure:

```js
dissocPath(['address', 'zipCode'], person)
```

There is also `omit`, which can remove several properties at once. 

```js
const updatedPerson = omit(['age', 'birthCountry'], person)
```

Note that `pick` and `omit` are quite similar and complement each other nicely. They’re very handy for white-listing (keep only this set of properties using `pick`) or black-listing (get rid of this set of properties using `omit`).

## Transforming Properties

We now know enough to work with objects in a declarative and immutable fashion. Let’s write a function, `celebrateBirthday`, that updates a person’s age on their birthday.

celebrateBirthday

```js
const nextAge = compose(inc, prop('age'))
const celebrateBirthday = person => assoc('age', nextAge(person), person)
```

This is a pretty common pattern. Rather than updating a property to a known new value, we really want to transform the value by applying a function to the old value as we’ve done here.

I don’t know of a good way to write this with less duplication and in pointfree style given the tools we know about.

Ramda to the rescue once more with its `evolve` function. I introduced `evolve` in [using-ramda-with-redux](https://randycoulman.com/blog/2016/02/16/using-ramda-with-redux/).

`evolve` takes an object that specifies a transformation function for each property to be transformed. Let’s refactor `celebrateBirthday` to use `evolve`:

Using evolve

```js
const celebrateBirthday = evolve({ age: inc })
```

This code is saying to evolve the target object (not shown here because of pointfree style) by making a new object with the same properties and values, but whose `age` is obtained by applying `inc` to the original value of `age`.

`evolve` can transform multiple properties at once and at multiple levels of nesting. The transformation object can have the same shape as the object being evolved and `evolve` will recursively traverse both structures, applying transformation functions as it goes.

Note that `evolve` will not add new properties; if you specify a transformation for a property that doesn’t appear in the target object, `evolve` will just ignore it.

I’ve found that `evolve` has quickly become a workhorse in my applications.

## Merging Objects

Sometimes, you’ll want to merge two objects together. A common case is when you have a function that takes named options and you want to combine those options with a set of default options. Ramda provides `merge` for this purpose.

Function with Options

```js
function f(a, b, options = {}) {
  const defaultOptions = { value: 42, local: true }
  const finalOptions = merge(defaultOptions, options)
}
```

`merge` returns a new object containing all of the properties and values from both objects. If both objects have the same property, the value from the second argument is used.

Having the second argument win makes sense when using `merge` by itself, but less so in a pipeline situation. In that case, you’re often performing a series of transformations to an object, and one of those transformations is to merge in some new property values. In this case, you want the first argument to win.

Simply using `merge(newValues)` in the pipeline will not do what you expect.

For this situation, I typically define a utility function called `reverseMerge`. It can be written as

```js
const reverseMerge = flip(merge)
```

Recall that `flip` reverses the first two arguments of the function it is applied to.

`merge` performs a shallow merge. If the objects being merged both have a property whose value is a sub-object, those sub-objects will not be merged. Ramda does not currently have a “deep merge” capability, where sub-objects are merged recursively.

> webpack-merge provides “deep merge”. 见另一随笔[对此函数用法测试](https://www.cnblogs.com/cuishengli/p/13948947.html)

Note that `merge` only takes two arguments. If you want to merge multiple objects into one, there is `mergeAll` that takes an array of the objects to be merged.

## Conclusion

This has given us a nice set of tools for working with objects in a declarative and immutable way. We can now read, add, update, delete, and transform properties in objects without changing the original objects. And we can do these things in a way that works when combining functions.

## Next

Now that we can work with objects in an immutable way, what about arrays? [Immutability and Arrays](https://randycoulman.com/blog/2016/07/05/thinking-in-ramda-immutability-and-arrays/) shows us how.