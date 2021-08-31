# Thinking in Ramda: Pointfree Style



This post is Part 5 of a series about functional programming called [Thinking in Ramda](https://randycoulman.com/blog/categories/thinking-in-ramda/).

In [Part 4](https://randycoulman.com/blog/2016/06/14/thinking-in-ramda-declarative-programming/), we talked about writing code declaratively (telling the computer what to do) instead of imperatively (telling the computer how to do it).

You may have noticed that several of the functions we’ve written (`forever21`, `drivingAge`, and `water`, for example) all take a parameter, build up a new function, and then apply that function to the parameter.

This is a very common pattern in functional programming, and once again Ramda provides the tools to clean this up.

## Pointfree Style

There are two main guiding principles of Ramda that we talked about in [Part 3](https://randycoulman.com/blog/2016/06/07/thinking-in-ramda-partial-application/):

- Put the data last
- Curry all the things

These two principles lead to a style that functional programmers call “pointfree”. I like to think of pointfree code as “Data? What data? There’s no data here.”

There’s a great blog post, [Why Ramda?](http://fr.umio.us/why-ramda/), that illustrates pointfree style really well. Tellingly, it has section headings like “Where’s the Data?”, “All Right, Already! May I See Some Data?”, and “I Just Want My Data, Thanks”.

We don’t yet have the tools we need to make all of our examples completely pointfree, but we can start.

Let’s look at `forever21` again:

forever21

```js
const forever21 = age => ifElse(gte(__, 21), always(21), inc)(age)
```

Notice that `age` only appears twice: once in the argument list, and once at the very end of the function as we apply the new function returned by `ifElse` to it.

If we pay attention while working with Ramda, we’ll see this pattern a lot. It almost always means that there’s a way to convert the function to pointfree style.

Let’s see what that would look like:

Pointfree forever21

```js
const forever21 = ifElse(gte(__, 21), always(21), inc)
```

And, *poof*! We just made the `age` disappear. Pointfree style. Note that there is no behavioral difference in these two versions. We’re still returning a function that takes an age, but now we’re not explicitly specifying the age parameter.

We can do the same thing with `alwaysDrivingAge` and `water` as well.

When we left off, `alwaysDrivingAge` looked like this:

Original alwaysDrivingAge

```js
const alwaysDrivingAge = age => ifElse(lt(__, 16), always(16), identity)(age)
```

We can apply the same transformation to make it pointfree.

Pointfree alwaysDrivingAge

```js
const alwaysDrivingAge = when(lt(__, 16), always(16))
```

And here’s where we left `water`:

Using cond

```js
const water = temperature => cond([
  [equals(0),   always('water freezes at 0°C')],
  [equals(100), always('water boils at 100°C')],
  [T,           temp => `nothing special happens at ${temp}°C`]
])(temperature)
```

And here it is, pointfree style:

Using cond

```js
const water = cond([
  [equals(0),   always('water freezes at 0°C')],
  [equals(100), always('water boils at 100°C')],
  [T,           temp => `nothing special happens at ${temp}°C`]
])
```

## Multi-argument Functions

What about functions that take more than one argument? Let’s look back at the `titlesForYear` example from [Part 3](https://randycoulman.com/blog/2016/06/07/thinking-in-ramda-partial-application/).

titlesForYear

```js
const titlesForYear = curry((year, books) =>
  pipe(
    filter(publishedInYear(year)),
    map(book => book.title)
  )(books)
)
```

Notice that `books` only appears twice: once as the last parameter in the argument list (data last!), and once at the very end of the function as we apply our pipeline to it. This is similar to the pattern we saw with `age` above, so let’s apply the same transformation to it:

Pointfree titlesForYear

```js
const titlesForYear = year =>
  pipe(
    filter(publishedInYear(year)),
    map(book => book.title)
  )
```

It works! We now have a pointfree version of `titlesForYear`.

Honestly, I probably wouldn’t aim for pointfree style in this case because JavaScript doesn’t make it convenient to call a series of single-argument functions, as we discussed in earlier posts.

If we want to use `titlesForYear` in a pipeline, we’re fine. We can call `titlesForYear(2012)` very easily. But if we want to use it by itself, we have to go back to the `)(` pattern we saw in the previous post: `titlesForYear(2012)(books)`. To me, that’s not worth the tradeoff.

But any time I have a single-argument function that follows (or can be refactored to follow) the pattern above, I’ll almost always make it pointfree.

## Refactoring to Pointfree

There will be times when our functions don’t follow the pattern. We might be operating on the data multiple times in the same function.

This was the case in several of the examples in [Part 2](https://randycoulman.com/blog/2016/05/31/thinking-in-ramda-combining-functions/). In those examples, we refactored our code to combine functions using things like `both`, `either`, `pipe`, and `compose`. Once we’d done that, making our functions pointfree was a relatively easy transformation.

Let’s look back at the `isEligibleToVote` example. Here’s where we started:

Eligible Voters

```js
const wasBornInCountry = person => person.birthCountry === OUR_COUNTRY
const wasNaturalized = person => Boolean(person.naturalizationDate)
const isOver18 = person => person.age >= 18
const isCitizen = person => wasBornInCountry(person) || wasNaturalized(person)
const isEligibleToVote = person => isOver18(person) && isCitizen(person)
```

Let’s start with `isCitizen`. It takes a `person` and then applies two different functions to that person, combining the results with `||`. As we learned in [Part 2](https://randycoulman.com/blog/2016/05/31/thinking-in-ramda-combining-functions/), we can instead use `either` to combine the two functions into a new function first, and then apply the combined function to the person.

Using either

```js
const isCitizen = person => either(wasBornInCountry, wasNaturalized)(person)
```

We can do the same thing with `isEligibleToVote` using `both`:

Using both

```js
const isEligibleToVote = person => both(isOver18, isCitizen)(person)
```

Now that we’ve done these refactorings, we can see that both functions follow the pattern we talked above above: `person` is mentioned twice, once as the function argument, and once at the end as we apply our combined function to it. We can now refactor to pointfree style:

With pointfree style

```js
const isCitizen = either(wasBornInCountry, wasNaturalized)
const isEligibleToVote = both(isOver18, isCitizen)
```

## Why?

Pointfree style takes time to get used to. It can be hard to adapt to the missing data arguments everywhere. It is also important to have some familiarity with Ramda’s functions to know how many arguments they eventually need.

But once you get used to it, it becomes very powerful to have a bunch of small pointfree functions combined together in interesting ways.

What’s the advantage of pointfree style? One could argue that it’s just an academic exercise designed to win a functional programming merit badge. However, I think there are a few advantages, even in spite of the work it takes to get used to the style:

- It makes programs simpler and more concise. This isn’t always a good thing, but it can be.
- It makes algorithms clearer. By focusing only on the functions being combined, we get a better sense of what’s going on without the data arguments getting in the way.
- It forces us to think more about the transformation being done than about the data being transformed.
- It helps us think about our functions as generic building blocks that can work with different kinds of data, rather than thinking about them as operations on a particular kind of data. By giving the data a name, we’re [anchoring](https://en.wikipedia.org/wiki/Anchoring) our thoughts about where we can use our functions. By leaving the data argument out, it allows us to be more creative.

## Conclusion

Pointfree style, also known as [tacit programming](https://en.wikipedia.org/wiki/Tacit_programming), can make our code clearer and easier to reason about. By refactoring our code to combine all of our transformations into a single function, we end up with smaller building blocks that can be used in more places.

## Next

In our examples, we haven’t been able to refactor everything to pointfree style. We still have code that is written in an imperative style. Most of this code is dealing with objects and arrays.

We need to find declarative ways of interacting with objects and arrays. And what about immutability? How do we manipulate objects and arrays in an immutable way?

The next post in this series, [Immutability and Objects](https://randycoulman.com/blog/2016/06/28/thinking-in-ramda-immutability-and-objects/) discusses how to work with objects in a functional and immutable way. The post after that, [Immutability and Arrays](https://randycoulman.com/blog/2016/07/05/thinking-in-ramda-immutability-and-arrays/) does the same for arrays.

