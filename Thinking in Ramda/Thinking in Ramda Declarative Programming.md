# Thinking in Ramda: Declarative Programming

This post is Part 4 of a series about functional programming called [Thinking in Ramda](https://randycoulman.com/blog/categories/thinking-in-ramda/).

In [Part 3](https://randycoulman.com/blog/2016/06/07/thinking-in-ramda-partial-application/), we talked about combining functions that take more than one argument by using the techniques of partial application and currying.

As we start writing small functional building blocks and combining them, we find that we have to write a lot of functions that wrap JavaScript’s operators such as arithmetic, comparison, logic, and control flow. This can feel tedious, but Ramda has our back.

But first, some background.

## Imperative vs Declarative

There are many different ways to divide up the programming language/style landscape. There’s static typing vs dynamic typing, interpreted languages vs compiled languages, low-level vs high-level, etc.

Another such division is imperative programming vs declarative programming.

Without going too deep into this, imperative programming is a style of programming where the programmers tell the computer what to do by telling it how to do it. Imperative programming gives rise to a lot of the constructs we use every day: control flow (`if`-`then`-`else` statements and loops), arithmetic operators (`+`, `-`, `*`, `/`), comparison operators (`===`, `>`, `<`, etc.), and logical operators (`&&`, `||`, `!`).

Declarative programming is a style of programming where the programmers tell the computer what to do by telling it what they want. The computer then has to figure out how to produce the result.

One of the classic declarative languages is Prolog. In Prolog, a program consists of a set of facts and a set of inference rules. You kick off the program by asking a question, and Prolog’s inference engine uses the facts and rules to answer your question.

Functional programming is considered a subset of declarative programming. In a functional program, we define functions and then tell the computer what to do by combining these functions.

Even in declarative programs, it is necessary to do similar tasks to those we do in imperative programs. Control flow, arithmetic, comparison, and logic are still the basic building blocks we have to work with. But we need to find a way to express these constructs in a declarative way.

## Declarative Replacements

Since we’re programming in JavaScript, an imperative language, it’s fine to use the standard imperative constructs when writing “normal” JavaScript code.

But when we’re writing functional transformations using pipelines and similar constructs, the imperative constructs don’t play well.

Let’s look at some of these basic building blocks that Ramda provides to help us out of this jam.

## Arithmetic

Back in [Part 2](https://randycoulman.com/blog/2016/05/31/thinking-in-ramda-combining-functions/), we implemented a series of arithmetic transformations to demonstrate a pipeline:

Arithmetic Pipeline

```js
const multiply = (a, b) => a * b
const addOne = x => x + 1
const square = x => x * x
const operate = pipe(
  multiply,
  addOne,
  square
)
operate(3, 4) // => ((3 * 4) + 1)**2 => (12 + 1)**2 => 13**2 => 169
```

Notice how we had to write functions for all of these basic building blocks that we wanted to use.

Ramda provides `add`, `subtract`, `multiply`, and `divide` functions to use in place of the standard arithmetic operators. So we can use Ramda’s `multiply` in place of the one we wrote ourselves, we can take advantage of Ramda’s curried `add` function to replace our `addOne`, and we can write `square` in terms of `multiply` as well:

Using Ramda's Arithmetic Functions

```js
const square = x => multiply(x, x)
const operate = pipe(
  multiply,
  add(1),
  square
)
```

`add(1)` is very similar to the increment operator (`++`), but the increment operator modifies the variable being incremented, so it is a mutation. As we learned in [Part 1](https://randycoulman.com/blog/2016/05/24/thinking-in-ramda-getting-started/), immutability is a core tenet of functional programming so we don’t want to be using `++` or its cousin `--`.

We can use `add(1)` and `subtract(1)` for incrementing and decrementing, but because those two operations are so common, Ramda provides `inc` and `dec` instead.

So we can simplify our pipeline a bit more:

Using inc

```js
const square = x => multiply(x, x)
const operate = pipe(
  multiply,
  inc,
  square
)
```

`subtract` is the replacement for the binary `-` operator, but there’s also the unary `-` operator for negating a value. We could use `multiply(-1)`, but Ramda provides the `negate` function to do this job.

## Comparison

Also in [Part 2](https://randycoulman.com/blog/2016/05/31/thinking-in-ramda-combining-functions/), we wrote some functions for determining if a person was eligible to vote. The final version of that code looked like this:

Eligible Voters

```js
const wasBornInCountry = person => person.birthCountry === OUR_COUNTRY
const wasNaturalized = person => Boolean(person.naturalizationDate)
const isOver18 = person => person.age >= 18
const isCitizen = either(wasBornInCountry, wasNaturalized)
const isEligibleToVote = both(isOver18, isCitizen)
```

Notice that some of our functions are using standard comparison operators (`===` and `>=` in this case). As you might suspect by now, Ramda also provides replacements for these.

Let’s modify our code to use `equals` in place of `===` and `gte` in place of `>=`.

Using Ramda Comparison functions

```js
const wasBornInCountry = person => equals(person.birthCountry, OUR_COUNTRY)
const wasNaturalized = person => Boolean(person.naturalizationDate)
const isOver18 = person => gte(person.age, 18)
const isCitizen = either(wasBornInCountry, wasNaturalized)
const isEligibleToVote = both(isOver18, isCitizen)
```

Ramda also provides `gt` for `>`, `lt` for `<`, and `lte` for `<=`.

Note that these functions take their arguments in what seems like normal order (is the first argument greater than the second?) That makes sense when used in isolation, but can be confusing when combining functions. These functions seem to violate Ramda’s “data-last” principle, so we’ll have to be careful when we use them in pipelines and similar situations. That’s when `flip` and the placeholder (`__`) will come in handy.

In addition to `equals`, there is `identical` for determining if two values reference the same memory.

There are a couple of common uses of `===`: checking if a string or array is empty (`str === ''` or `arr.length === 0`), and checking if a variable is `null` or `undefined`. Ramda provides handy functions for both cases: `isEmpty` and `isNil`.

## Logic

In [Part 2](https://randycoulman.com/blog/2016/05/31/thinking-in-ramda-combining-functions/) (and just above), we used the `both` and `either` functions in place of `&&` and `||` operations. We also talked about `complement` in place of `!`.

These combined functions work great when the functions we are combining both operate on the same value. Above, `wasBornInCountry`, `wasNaturalized`, and `isOver18` all apply to a person.

But sometimes we need to apply `&&`, `||`, and `!` to disparate values. For those cases, Ramda gives us `and`, `or`, and `not` functions. I think of it this way: `and`, `or`, and `not` work with values, while `both`, `either`, and `complement` work with functions.

A common use of `||` is for providing default values. For example, we might write something like this:

Defaulting

```js
const lineWidth = settings.lineWidth || 80
```

This is a common idiom, and mostly works, but relies on JavaScript’s definition of “falsy”. What if `0` is a valid setting? Since `0` is falsy, we’ll end up with a line width of 80.

We could use the `isNil` function we just learned about above, but again Ramda has a nicer option for us: `defaultTo`.

Using defaultTo

```
const lineWidth = defaultTo(80, settings.lineWidth)
```

`defaultTo` checks if the second argument `isNil`. If it isn’t, it returns that as the value, otherwise it returns the first value.

## Conditionals

Control flow is less necessary in functional programs, but still occasionally useful. The collection iteration functions we talked about in [Part 1](https://randycoulman.com/blog/2016/05/24/thinking-in-ramda-getting-started/) take care of most looping situations, but conditionals are still quite important.

### ifElse

Let’s write a function, `forever21`, that takes an age and returns the next age. But, as the name implies, once the age is 21, it stays that way.

Forever 21

```js
const forever21 = age => age >= 21 ? 21 : age + 1
```

Notice that our conditional (`age >= 21`) and the second branch (`age + 1`) can both be written as functions of `age`. We could rewrite the first branch (`21`) as a constant function (`() => 21`) instead. Now we have three functions that take (or ignore) `age`.

We’re now in a position where we can use Ramda’s `ifElse` function, which is the function equivalent of the `if...then...else` structure or it’s shorter cousin, the ternary operator (`?:`).

Using ifElse

```js
const forever21 = age => ifElse(gte(__, 21), () => 21, inc)(age)
```

As mentioned above, the comparison functions don’t work as we might like when combining functions, so we were forced to introduce the placeholder here (`__`). We could also switch over to `lte` instead:

Using lt instead of gte

```js
const forever21 = age => ifElse(lte(21), () => 21, inc)(age)
```

In this case, we have to read this as “21 is less than or equal to age.” I’m going to stick with the placeholder version for the rest of this post because I find it more readable and less confusing.

### Constants

Constant functions are quite useful in situations like this. As you might imagine, Ramda provides us a shortcut. In this case, the shortcut is named `always`.

Using always

```js
const forever21 = age => ifElse(gte(__, 21), always(21), inc)(age)
```

Ramda also provides `T` and `F` as further shortcuts for `always(true)` and `always(false)`.

### Identity

Let’s try another function, `alwaysDrivingAge`. This function takes an age and returns it if it’s `gte` 16. But if it is less than 16, it returns 16. This allows anyone to pretend that they’re driving age, even if they’re not.

Driving Age

```js
const alwaysDrivingAge = age => ifElse(lt(__, 16), always(16), a => a)(age)
```

The second branch of the conditional (`a => a`) is another common pattern in functional programming. It is known as the identity function. That is, a function that returns whatever argument it is given.

As you might suspect, Ramda provides an `identity` function for us.

Using identity

```js
const alwaysDrivingAge = age => ifElse(lt(__, 16), always(16), identity)(age)
```

`identity` can take more than one argument, but it always returns its first argument. If we want to return something other than the first argument, there’s the more general `nthArg` function. It’s much less common than `identity`.

### when and unless

Having an `ifElse` statement where one of the conditional branches is `identity` is also quite common, so Ramda provides more shortcuts for us.

If, as in our case, the second branch is `identity`, we can use `when` instead of `ifElse`:

Using when

```js
const alwaysDrivingAge = age => when(lt(__, 16), always(16))(age)
```

If the first branch of the conditional is `identity`, we can use `unless`. If we reversed our condition to use `gte(__, 16)` instead, we could use `unless`.

Using unless

```js
const alwaysDrivingAge = age => unless(gte(__, 16), always(16))(age)
```

### cond

Ramda also provides the `cond` function which can replace a `switch` statement or a chain of `if...then...else` statements.

I’ll repeat the example from the Ramda documentation to show how it’s used:

Using cond

```js
const water = temperature => cond([
  [equals(0),   always('water freezes at 0°C')],
  [equals(100), always('water boils at 100°C')],
  [T,           temp => `nothing special happens at ${temp}°C`]
])(temperature)
```

I haven’t needed to use `cond` yet in my Ramda code, but I used to write Common Lisp code many years ago, so `cond` feels like an old friend.

## Conclusion

We’ve now looked at a number of functions that Ramda gives us for turning our imperative code into declarative functional code.

## Next

You may have noticed that the last few functions we wrote (`forever21`, `drivingAge`, and `water`) all take a parameter, build up a new function, and then apply that function to the parameter.

This is a common pattern, and once again Ramda provides the tools to clean this up. The next post, [Pointfree Style](https://randycoulman.com/blog/2016/06/21/thinking-in-ramda-pointfree-style/) looks at how to simplify functions that follow this pattern.