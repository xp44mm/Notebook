# Why Y? Deriving the Y Combinator in JavaScript

------

<iframe width="160" height="430" src="https://leanpub.com/javascriptallongesix/embed" frameborder="0" allowtransparency="true" style="margin: 0px 0px 0px -160px; padding: 0px; border: 0px; font: inherit; vertical-align: baseline; position: relative; float: right; left: 180px; top: 0px;"></iframe>

*…and two practical applications…*

------

The [Y Combinator](https://en.wikipedia.org/wiki/Fixed-point_combinator) is an important result in theoretical computer science.[1](https://raganwald.com/2018/09/10/why-y.html#fn:yc)

In this essay, after a brief review of the work we’ve already done on the Mockingbird, we’ll derive the Why Bird, known most famously as the [Y Combinator](https://en.wikipedia.org/wiki/Fixed-point_combinator). The why bird provides all the benefits of the mockingbird, but allows us to write more idiomatic JavaScript. We’ll see that one of the benefits of writing recursive functions in “why bird form” is that we can compose and decorate them easily.

We’ll then derive the “Decoupled Trampoline,” a/k/a “Long-Tailed Widowbird.” The decoupled trampoline builds on the why bird and Y Combinator to allow us to write tail-recursive functions that execute in constant stack space, while hewing closely to idiomatic JavaScript.

While this use case is admittedly rare in production code, it does arise from time to time and it is pleasing to contemplate a direct connection between one of programming’s most cerebrally theoretical constructs, and a tool for overcoming the limitations of today’s JavaScript implementations.

------

[![Hood Mockingbird copyright 2007](https://raganwald.com/assets/images/hood-mockingbird.jpg)](https://www.flickr.com/photos/putneymark/1225867875)

------

## Preamble: Revisiting the mockingbird

To review what we saw in [To Grok a Mockingbird](http://raganwald.com/2018/08/30/to-grok-a-mockingbird.html), a typical recursive function calls itself by name, like this:[2](https://raganwald.com/2018/09/10/why-y.html#fn:realism)

```js
function exponent (x, n) {
  if (n === 0) {
    return 1;
  } else if (n % 2 === 1) {
    return x * exponent(x * x, Math.floor(n / 2));
  } else {
    return exponent(x * x, n / 2);
  }
}

exponent(2, 7)
  //=> 128
```

Because it calls itself by name, it is tightly coupled to itself. This means that if we want to decorate it–such as by memoizing its return values, or if we want to change its implementation strategy–like employing [trampolining](https://en.wikipedia.org/wiki/Trampoline_(computing))–we have to rewrite the function.

We saw that we can decouple a recursive function from itself. Instead of calling itself by name, we arrange to pass the recursive function to itself as a parameter. We begin by rewriting our function to take itself as a parameter, and also to pass itself as a parameter.

We call that writing a recursive function in mockingbird form. It looks like this:

```
(myself, x, n) => {
  if (n === 0) {
    return 1;
  } else if (n % 2 === 1) {
    return x * myself(myself, x * x, Math.floor(n / 2));
  } else {
    return myself(myself, x * x, n / 2);
  }
};
```

Given a function written in mockingbird form, we use a JavaScript implementation of the mockingbird to turn it into a recursive function:[3](https://raganwald.com/2018/09/10/why-y.html#fn:m)

```
const mockingbird =
  fn =>
    (...args) =>
      fn(fn, ...args);

const exponent =
  mockingbird(
    (myself, x, n) => {
      if (n === 0) {
        return 1;
      } else if (n % 2 === 1) {
        return x * myself(myself, x * x, Math.floor(n / 2));
      } else {
        return myself(myself, x * x, n / 2);
      }
    }
  );

exponent(3, 3)
  //=> 27
```

Because the recursive function has been decoupled from itself, we can do things like memoize it:

```
const memoized = (fn, keymaker = JSON.stringify) => {
  const lookupTable = Object.create(null);

  return function (...args) {
    const key = keymaker.call(this, args);

    return lookupTable[key] || (lookupTable[key] = fn.apply(this, args));
  }
};

const ignoreFirst = ([_, ...values]) => JSON.stringify(values);

const exponent =
  mockingbird(
    memoized(
      (myself, x, n) => {
        if (n === 0) {
          return 1;
        } else if (n % 2 === 1) {
          return x * myself(myself, x * x, Math.floor(n / 2));
        } else {
          return myself(myself, x * x, n / 2);
        }
      },
      ignoreFirst
    )
  );
```

Memoizing our recursive function does not require any changes to its code. We can easily reuse it elsewhere if we wish.

------

[![Curlicue copyright 2017 Anja Pietsch](https://raganwald.com/assets/images/curlicue.jpg)](https://www.flickr.com/photos/a_peach/33299615155)

------

### why the mockingbird needs improvement

The mockingbird is a useful tool, but it has a drawback: In addition to rewriting our functions to take themselves as a parameter, we also have to rewrite them to pass themselves along. So in addition to this:

```
(myself, x, n) => ...
```

We must also write this:

```
myself(myself, x * x, n / 2)
```

The former is the point of decoupling. The latter is nonsense!

The idea behind the way we use the mockingbird (as opposed to a literal interpretation of the M combinator) is to write idiomatic JavaScript. But there’s nothing “idiomatic” about a function invoking itself with `myself(myself, ...)`.

What we want is a function *like* the mockingbird, but it must support functions calling themselves idiomatically, e.g. `myself(x * x, n / 2)`.

Let’s visualize exactly what we want. With the mockingbird, we write:

```
const exponent =
  mockingbird(
    (myself, x, n) => {
      if (n === 0) {
        return 1;
      } else if (n % 2 === 1) {
        return x * myself(myself, x * x, Math.floor(n / 2));
      } else {
        return myself(myself, x * x, n / 2);
      }
    }
  );
```

We want a better recursive combinator, one that lets us write:

```
const exponent =
  _____(
    (myself, x, n) => {
      if (n === 0) {
        return 1;
      } else if (n % 2 === 1) {
        return x * myself(x * x, Math.floor(n / 2));
      } else {
        return myself(x * x, n / 2);
      }
    }
  );
```

Let’s build that!

------

[![Sage Grouse Lek © 2006 BLM Wyoming](https://raganwald.com/assets/images/lek.jpg)](https://www.flickr.com/photos/134389515@N06/38964205040)

------

## Part I: Deriving the Y Combinator from the Mockingbird

Before we begin, there are some rules we have to follow if we are to take the mockingbird and derive another combinator from it. Every combinator has the following properties:

1. It is a function.
2. It can only use its parameters or previously defined combinators that have these same properties. This means it cannot refer to non-combinators, like constants, ordinary functions, or object from the global namespace.
3. It cannot be a named function declaration or named function expression.
4. It cannot create a named function expression.
5. It cannot declare any bindings other than via parameters.
6. It can invoke a function in its implementation.

In combinatory logic, all combinators take exactly one parameter, as do all of the functions that combinators create. Combinatory logic also eschews all gathering and spreading of parameters. When creating an idiomatic JavaScript combinator (like `mockingbird`), we eschew these limitations. Idiomatic JavaScript combinators can:

- Gather parameters, and;
- Spread parameters.

------

[![Sage Thrasher © 2016 Bettina Arrigoni](https://raganwald.com/assets/images/sage-thrasher.jpg)](https://www.flickr.com/photos/barrigoni/25566626012)

------

### from mockingbird to why bird in seven easy pieces

**Step Zero**: We begin with the mockingbird:

```
const mockingbird =
  fn =>
    (...args) =>
      fn(fn, ...args);
```

**Step One**, we name our combinator. Honouring Raymond Smullyan’s choice, we shall call it the why bird:

```
const why =
  fn =>
    (...args) =>
      fn(fn, ...args);
```

**Step Two**, we identify the key change we have to make:

```
const why =
  fn =>
    (...args) =>
      fn(?, ...args);
```

We’ve replaced that one `fn` with a placeholder. Why? Well, our `fn` is a function that looks like this: `(myself, arg0, arg1, ..., argn) => ...`. But whatever we pass in for `myself` will look like this: `(arg0, arg1, ..., argn) => ...`. So it can’t be `fn`.

But what will it be?

Well, the approach we are going to take is to think about the mockingbird. What does it do? It takes a function like `(myself, arg0, arg1, ..., argn) => ...`, and returns a function that looks like `(arg0, arg1, ..., argn) => ...`.

The mockingbird isn’t what we want, but let’s airily assume that there is such a function. We’ll call it `maker`, because it makes the function we want.

**Step Three**, we replace `?` with `maker(??)`. We know it will make the function we want, but we don’t yet know what we must pass to it:

```
const why =
  fn =>
    (...args) =>
      fn(maker(??), ...args);
```

This leaves us two things to figure out:

1. Where do we get `maker`, and;
2. What parameter(s) do we pass to it.

For `1`, we could define `maker` as an anonymous function expression. Another option arises. In the days before ES6, if we wanted to define variables within a scope smaller than a function, we created an immediately invoked function expression.

**Step Four**, we could, after some experimentation, consider this format that binds a function expression to `maker` with an immediately invoked function expression;

```
const why =
  fn =>
    (
      maker =>
        (...args) =>
          fn(maker(??), ...args)
    )(???);
```

This still leaves us two things to work out: `??` is what we pass to `maker`, and `???` is maker’s expression. Here’s the “Eureka!” moment:

`maker` is a function that takes one or more parameters and returns a function that looks like `(...args) => fn(maker(??), ...args)`. That’s the function we want to pass to `fn` as `myself`. The mockingbird isn’t such a function, but we can see one right in front of us:

`maker => (...args) => fn(maker(??), ...args)` is a function that takes one parameter(`maker`) and returns a function that looks like `(...args) => fn(maker(??), ...args)`!

**Step Five**, let’s fill that in for `???`:

```
const why =
  fn =>
    (
      maker =>
        (...args) =>
          fn(maker(??), ...args)
    )(
      maker =>
        (...args) =>
          fn(maker(??), ...args)
    );
```

Now what about `??`? Well, we have just decided two things:

1. `maker` takes one or more parameters and returns a function that looks like `(...args) => fn(maker(??), ...args)`, and;
2. `maker => (...args) => fn(maker(??), ...args)` is a function that takes one parameter(`maker`) and returns a function that looks like `(...args) => fn(maker(??), ...args)`.

Conclusion: `maker` takes one parameter, `maker`, and returns a function that looks like `(...args) => fn(maker(??), ...args)`. Therefore, the expression we want is `maker(maker)`, and `??` is nothing more than `maker`!

**Step Six**:

```
const why =
  fn =>
    (
      maker =>
        (...args) =>
          fn(maker(maker), ...args)
    )(
      maker =>
        (...args) =>
          fn(maker(maker), ...args)
    );
```

Let’s test it:

```
const exponent =
  why(
    (myself, x, n) => {
      if (n === 0) {
        return 1;
      } else if (n % 2 === 1) {
        return x * myself(x * x, Math.floor(n / 2));
      } else {
        return myself(x * x, n / 2);
      }
    }
  );

exponent(2, 9)
  //=> 512
```

Voila! A working why bird!!

------

[![Underwood Typewriter Keys ©2010 Steve Depolo](https://raganwald.com/assets/images/typewriter-keys.jpg)](https://www.flickr.com/photos/stevendepolo/4550901923)

------

### from why bird to y combinator

Our why bird is written in–and for–idiomatic JavaScript, especially with respect to employing functions that take more than one parameter. A direct implementation of a formal combinator only takes one parameter and only works with functions that take one parameter.

We can translate our why bird to its formal combinator, the **y combinator**. To aid us, let’s first imagine a recursive function:

```
const isEven =
  n =>
    (n === 0) || !isEven(n - 1);
```

In why bird form, it becomes:

```
const _isEven =
  (myself, n) =>
    (n === 0) || !myself(n - 1);
```

Alas, it now takes two parameters. We fix this by [currying](http://raganwald.com/2013/03/07/currying-and-partial-application.html) it:

```
const __isEven =
  myself =>
    n =>
      (n === 0) || !myself(n - 1);
```

Instead of taking two parameters (`myself` and `n`), it is now a function taking one parameter, `myself`, and returning a function that takes another parameter, `n`.

To accommodate functions in this form, we take our why bird and perform some similar modifications. We’ll start as above by renaming it:

```
const Y =
  fn =>
    (
      maker =>
        (...args) =>
          fn(maker(maker), ...args)
    )(
      maker =>
        (...args) =>
          fn(maker(maker), ...args)
    );
```

Next, we observe that `(...args) => fn(maker(maker), ...args)` is not allowed, we do not gather and spread parameters. First, we change `...args` into just `arg`, since only one parameter is allowed:

```
const Y =
  fn =>
    (
      maker =>
        arg => fn(maker(maker), arg)
    )(
      maker =>
        arg => fn(maker(maker), arg)
    );
```

`fn(maker(maker), arg)` is also not allowed, we do not pass two parameters to any function. Instead, we pass one parameter, get a function back, and pass the second parameter to that function. Like this:

```
const Y =
  fn =>
    (
      maker =>
        arg => fn(maker(maker))(arg)
    )(
      maker =>
        arg => fn(maker(maker))(arg)
    );
```

Let’s try it:

```
Y(
  myself =>
    n =>
      (n === 0) || !myself(n - 1)
)(1962)
```

It works too, and now we have derived one of the most important results in theoretical computer science. The Y Combinator matters deeply, because in the kind of formal computation models that are simple enough to prove results (like the Lambda Calculus and Combinatory Logic), we do not have any iterative constructs, and must use recursion for nearly everything non-trivial.[4](https://raganwald.com/2018/09/10/why-y.html#fn:ya)

The Y Combinator makes recursion possible without requiring variable declarations. As we showed above, we can even make an anonymous function recursive, which is necessary in systems where functions do not have names.[5](https://raganwald.com/2018/09/10/why-y.html#fn:yy)

------

![Dame Judy Dench as Lady Miles Messervy](https://raganwald.com/assets/images/m.jpg)

------

### if a forest contains a mockingbird, it also contains a why bird

Looking this expression of the Y combinator, we can see why it was named after the letter “Y,” the code literally looks like a forking branch:

```
const Y =
  fn =>
    (m => a => fn(m(m))(a))(
      m => a => fn(m(m))(a)
    );
```

Since we’re talking direct implementations of formal combinators, let’s have a look at the M combinator:

```
const M =
  fn => fn(fn);
```

We can combine `M` and `Y` to create a more compact expression of the Y combinator:

```
const Y =
  fn =>
    M(m => a => fn(m(m))(a));
```

The compact expression of the Y combinator is usually expressed with the M combinator “reduced” to `(x => x(x))`:

```
const Y =
  fn =>
    (x => x(x))(m => a => fn(m(m))(a));
```

We can use the reduced M combinator make a compact why bird, too:

```
const why =
  fn =>
    (x => x(x))(
      maker =>
        (...args) =>
          fn(maker(maker), ...args)
    );
```

And with that, we have derived compact implementations of both the Y combinator and its idiomatic JavaScript equivalent, the why bird, from our mockingbird implementation.

------

[![Spiral ©2012 Renzo Borgatti](https://raganwald.com/assets/images/spiral.jpg)](https://www.flickr.com/photos/reborg/9222072041)

------

## Part II: Two practical applications for the Y Combinator

This function for determining whether a number is even is extremely slow:

```
const _isEven =
  (myself, n) =>
    (n === 0) || !myself(n - 1);

const isEven = why(_isEven);

isEven(1000)
  //=> Go for coffee,
  //   then return to find out that the answer is `true`

isEven(1001)
  //=> Go for another coffee,
  //   then return to find out that the answer is `false`
```

As discussed in [To Grok a Mockingbird](http://raganwald.com/2018/08/30/to-grok-a-mockingbird.html), one of the things we can do to improve performance is to [memoize](https://en.wikipedia.org/wiki/Memoization) our function:

```
const memoized = (fn, keymaker = JSON.stringify) => {
  const lookupTable = Object.create(null);

  return function (...args) {
    const key = keymaker.call(this, args);

    return lookupTable[key] || (lookupTable[key] = fn.apply(this, args));
  }
};

const ignoreFirst = ([_, ...values]) => JSON.stringify(values);

const isEvenFast = why(memoized(_isEven));

isEvenFast(1000)
  //=> Go for coffee,
  //   then return to find out that the answer is `true`

isEvenFast(1001)
  //=> false, immediately
```

Because the why bird decouples our `_isEven` function from itself, we can compose it with our `memoized` decorator or not as we see fit. Nothing in `_isEven` is coupled to whether we are memoizing the function or not.

Thus, the first benefit of functions written in “why bird form” is that they can be easily composed with decorators.[6](https://raganwald.com/2018/09/10/why-y.html#fn:prior-art)

------

[![Stack ©2014 alemjusic](https://raganwald.com/assets/images/stack.jpg)](https://www.flickr.com/photos/alemjusic/14076753751)

------

### trampolining tail-recursive functions

There’s another–albeit rarer–benefit to rewriting functions in why bird form. Consider this extreme use case:

```
why(
  (myself, n) =>
    (n === 0) || !myself(n - 1)
)(1000042)
  //=> Maximum call stack exceeded
```

Our function consumes stack space equal to the magnitude of the argument `n`. Naturally, this is a contrived example, but recursive functions that consume the entire stack to occur from time to time, and it is not always appropriate to rewrite them in iterative form.

One solution to this problem is to rewrite the function in tail-recursive form. If the JavaScript engine supports tail-call optimization, the function will execute in constant stack space:

```
// Safari Browser, c. 2018

why(
  (myself, n) => {
    if (n === 0)
      return true;
    else if (n === 1)
      return false;
    else return myself(n - 2);
  }
)(1000042)
  //=> true
```

However, not all engines support tail-call optimization, despite it being part of the JavaScript specification. If we wish to execute such a function in constant stack space, one of our options is to “greenspun” tail-call optimization ourselves by implementing a [trampoline](http://raganwald.com/2013/03/28/trampolines-in-javascript.html):[7](https://raganwald.com/2018/09/10/why-y.html#fn:recursion)

> A trampoline is a loop that iteratively invokes [thunk](https://en.wikipedia.org/wiki/Thunk_(functional_programming))-returning functions ([continuation-passing style](https://en.wikipedia.org/wiki/Continuation-passing_style)). A single trampoline is sufficient to express all control transfers of a program; a program so expressed is trampolined, or in trampolined style; converting a program to trampolined style is trampolining. Trampolined functions can be used to implement [tail-recursive](https://en.wikipedia.org/wiki/Tail-recursive_function) function calls in stack-oriented programming languages.–[Wikipedia](https://en.wikipedia.org/wiki/Trampoline_(computing))

As we saw in [To Grok a Mockingbird](http://raganwald.com/2018/08/30/to-grok-a-mockingbird.html), this necessitates having our recursive function become tightly coupled to its execution strategy. In other words, above and beyond being rewritten in tail-recursive form, it must explicitly return thunks rather than call `myself`:

```
class Thunk {
  constructor (delayed) {
    this.delayed = delayed;
  }

  evaluate () {
    return this.delayed();
  }
}

const trampoline =
  fn =>
    (...initialArgs) => {
      let value = fn(...initialArgs);

      while (value instanceof Thunk) {
        value = value.evaluate();
      }

      return value;
    };

const isEven =
  trampoline(
    function myself (n, parity = 0) {
      if (n === 0) {
        return parity === 0;
      } else {
        return new Thunk(() => myself(n - 1, 1 - parity));
      }
    }
  );

isEven(1000001)
  //=> false
```

In [To Grok a Mockingbird](http://raganwald.com/2018/08/30/to-grok-a-mockingbird.html), we solved this problem for functions written “in mockingbird form” with the Jackson’s Widowbird function. We created a function with the same contract as the mockingbird, but its implementation used a trampoline to execute recursive functions in constant stack space.

Functions written “in why bird form” are more idiomatically JavaScript than functions written in mockingbird form. If we can create a similar function that has the same contract as the why bird, but uses a trampoline to evaluate the recursive function, we could execute tail-recursive functions in constant stack space.

We will call this function the “Long-tailed Widowbird.” Let’s derive it.

------

![Long-Tailed Widowbird](https://raganwald.com/assets/images/long-tailed-widowbird.jpg)

------

### deriving the long-tailed widowbird from the why bird

Our goal is to create a trampolining function. So let’s start with the basic outline of a trampoline, and call it `longtailed`:

```
class Thunk {
  constructor (delayed) {
    this.delayed = delayed;
  }

  evaluate () {
    return this.delayed();
  }
}

const longtailed =
  fn =>
    (...initialArgs) => {
      let value = fn(...initialArgs);

      while (value instanceof Thunk) {
        value = value.evaluate();
      }

      return value;
    };
```

We won’t even bother trying this, we know that `fn(...initialArgs)` is not going to work without injecting a function for `myself`. But we do know a function that we can call with `...initialArgs`:

```
class Thunk {
  constructor (delayed) {
    this.delayed = delayed;
  }

  evaluate () {
    return this.delayed();
  }
}

const longtailed =
  fn =>
    (...initialArgs) => {
      let value = why(fn)(...initialArgs);

      while (value instanceof Thunk) {
        value = value.evaluate();
      }

      return value;
    };
```

This works, but never actually creates any thunks. To do that, let’s reduce `why(fn)`:

```
class Thunk {
  constructor (delayed) {
    this.delayed = delayed;
  }

  evaluate () {
    return this.delayed();
  }
}

const longtailed =
  fn =>
    (...initialArgs) => {
      let value =
        (x => x(x))(
          maker =>
            (...args) =>
              fn(maker(maker), ...args)
        )(...initialArgs);

      while (value instanceof Thunk) {
        value = value.evaluate();
      }

      return value;
    };
```

Now we see where the value for `myself` comes from, it’s `maker(maker)`. Let’s replace that with a function that, given some arguments, returns a new thunk that—when evaluated—returns `maker(maker)` invoked with those arguments:

```
class Thunk {
  constructor (delayed) {
    this.delayed = delayed;
  }

  evaluate () {
    return this.delayed();
  }
}

const longtailed =
  fn =>
    (...initialArgs) => {
      let value =
        (x => x(x))(
          maker =>
            (...args) =>
              fn((...argsmm) => new Thunk(() => maker(maker)(...argsmm)), ...args)
        )(...initialArgs);

      while (value instanceof Thunk) {
        value = value.evaluate();
      }

      return value;
    };

longtailed(
  (myself, n) => {
    if (n === 0)
      return true;
    else if (n === 1)
      return false;
    else return myself(n - 2);
  }
)(1000042)
  //=> true
```

It works! And it executes in constant stack space, as we wanted.[8](https://raganwald.com/2018/09/10/why-y.html#fn:works)

------

[![Ink & Water ©2010 Gagneet Parmar](https://raganwald.com/assets/images/ink-and-water.jpg)](https://www.flickr.com/photos/45818813@N05/4785640636)

------

### from long-tailed widowbird to decoupled trampoline

The long-tailed widowbird works, but it is code that only its author could love.

Let’s begin our cleanup by moving `Thunk` inside our function. This has certain technical advantages if we ever create a recursive program that itself returns thunks. Since it is now a special-purpose class that only ever invokes a single function, we’ll give it a more specific implementation:

```
const longtailed =
  fn => {
    class Thunk {
      constructor (fn, ...args) {
        this.fn = fn;
        this.args = args;
      }

      evaluate () {
        return this.fn(...this.args);
      }
    }

    return (...initialArgs) => {
      let value =
        (x => x(x))(
          maker =>
            (...args) =>
              fn((...argsmm) => new Thunk(maker(maker), ...argsmm), ...args)
        )(...initialArgs);

      while (value instanceof Thunk) {
        value = value.evaluate();
      }

      return value;
    };
  };
```

Next, let’s extract the creation of a function that delays the invocation of `maker(maker)`:

```
const longtailed =
  fn => {
    class Thunk {
      constructor (fn, ...args) {
        this.fn = fn;
        this.args = args;
      }

      evaluate () {
        return this.fn(...this.args);
      }
    }

    const thunkify =
      fn =>
        (...args) =>
          new Thunk(fn, ...args);

    return (...initialArgs) => {
      let value =
        (x => x(x))(
          maker =>
            (...args) =>
              fn(thunkify(maker(maker)), ...args)
        )(...initialArgs);

      while (value instanceof Thunk) {
        value = value.evaluate();
      }

      return value;
    };
  };
```

And now we have a considerably less ugly long-tailed widowbird. Well, actually, we are ignoring “the elephant in the room,” *the name of the function*. “Long-tailed Widowbird” is a touching tribute to the genius of Raymond Smullyan, and there is an amusing correlation between its long tail and the business of optimizing tail-recursive functions.

Nevertheless, if we are to work with others, we might want to consider the possibility that they would prefer a less poetic approach:

```
const decoupledTrampoline =
  fn => {
    class Thunk {
      constructor (fn, ...args) {
        this.fn = fn;
        this.args = args;
      }

      evaluate () {
        return this.fn(...this.args);
      }
    }

    const thunkify =
      fn =>
        (...args) =>
          new Thunk(fn, ...args);

    return (...initialArgs) => {
      let value =
        (x => x(x))(
          maker =>
            (...args) =>
              fn(thunkify(maker(maker)), ...args)
        )(...initialArgs);

      while (value instanceof Thunk) {
        value = value.evaluate();
      }

      return value;
    };
  };
```

And there we have our decoupled trampoline in its final form.

------

[![Clouds at the end ©2008 Richard Hammond](https://raganwald.com/assets/images/clouds-at-the-end.jpg)](https://www.flickr.com/photos/mrcubs/8780767395)

------

### summary

The mockingbird that we explored in [To Grok a Mockingbird](http://raganwald.com/2018/08/30/to-grok-a-mockingbird.html) is easy to understand, and decouples recursive functions from themselves. This provides us with more ways to compose recursive functions with other functions like decorators. However, it requires us to pass `myself` along when making recursive calls.

This is decidedly not idiomatic, so in Part I we derived the why bird, an idiomatic JavaScript recursive combinator that enables recursive functions to call themselves without any additional parameters. We then derived a JavaScript implementation of the Y combinator from the why bird, and finished by using a reduced version of the M combinator to produce “compact” implementations of both the why bird and the Y combinator.

In Part II, we revisited the need for optimizing tail-recursive recursive functions such that they operate in constant stack space. We derived the long-tailed widowbird from the why bird, then polished its implementation, naming the finished function the Decoupled Trampoline.

To recapitulate the use case for the decoupled trampoline, in the rare but nevertheless valid case where we wish to refactor a singly recursive function into a trampolined function to ensure that it does not consume the stack, we previously had to:

1. Refactor the function into tail-recursive form;
2. Refactor the tail-recursive version to explicitly invoke a trampoline;
3. Wrap the result in a trampoline function.

With the decoupled trampoline, we can:

1. Refactor the function into tail-recursive form;
2. Refactor the function into “why bird form,” then;
3. Wrap the result in the decoupled trampoline.

Why is this superior? We’re going to refactor into tail-recursive form either way, and we’re going to wrap the function either way, however:

1. Refactoring into “why bird form” is less intrusive than rewriting the code to explicitly return thunks, and;
2. The refactored code is decoupled from trampolining, so it is easier to reverse the procedure if need be, or even just used with the why bird;

If we compare and contrast:

```
const isEven =
  trampoline(
    function myself (n) {
      if (n === 0)
        return true;
      else if (n === 1)
        return false;
      else return new Thunk(() => myself(n - 2));
    }
  );
```

With:

```
const isEven =
  decoupledTrampoline(
    (myself, n) => {
      if (n === 0)
        return true;
      else if (n === 1)
        return false;
      else return myself(n - 2);
    }
  );
```

The latter has clearer separation of concerns and is thus easier to grok at first sight. And thus, we have articulated a practical (albeit infrequently needed) use for the Y Combinator.

That’s all!

(discuss on [hacker news](https://news.ycombinator.com/item?id=17951920) or [reddit](https://www.reddit.com/r/programming/comments/9g4p5v/why_y_deriving_the_y_combinator_in_javascript/))

------

The essays in this series on recursive combinators are: [To Grok a Mockingbird](http://raganwald.com/2018/08/30/to-grok-a-mockingbird.html), and [Why Y? Deriving the Y Combinator in JavaScript](http://raganwald.com/2018/09/10/why-y.html). Enjoy them both!

------

## Notes

1. Not to mention that there’s a famous technology investment firm and startup incubator that takes its name from the Y Combinator, likely because the Y Combinator acts as a kind of “bootstrap” to allow a function to build upon itself. [↩](https://raganwald.com/2018/09/10/why-y.html#fnref:yc)

2. The paradox of instructional explorations is that if we wish to illustrate a mechanism like recursive combinators, choosing trivial functions like exponentiation makes it easier to focus on the thing we’re exploring, the combinators. The tradeoff is that with such simple functions, it will always feel over-complicated to use recursive combinators. Whereas, if we work with functions with real-world implications, the mechanism we’re exploring gets lost in the complexity of the functions it operates upon. [↩](https://raganwald.com/2018/09/10/why-y.html#fnref:realism)

3. The mockingbird is more formally known as the M Combinator. Our naming convention is that when discussing formal combinators from combinatory logic, or direct implementations in JavaScript, we will use the formal name. But when using variations designed to work more idiomatically in JavaScript–such as versions that work with functions taking more than one argument), we will use Raymond Smullyan’s ornithological nicknames.

   For a formalist, the M Combinator’s direct translation is `const M = fn => fn(fn)`. This is only useful if `fn` is implemented in “curried” form, e.g. `const isEven = myself => n => n === 0 || !myself(n - 1)`. If we wish to use a function written in idiomatic JavaScript form, such as `const isEven = (myself, n) => n === 0 || !myself(n - 1)`, we use the mockingbird, which is given later as `const mockingbird = fn => (...args) => fn(fn, ...args)`. This is far more practical for programming purposes. [↩](https://raganwald.com/2018/09/10/why-y.html#fnref:m)

4. Well, actually, what we have derived is the applicative form of the Y Combinator, often called the *Z Combinator*. The difference between this combinator and the version you will find in the Lambda Calculus is driven by the fact that JavaScript is an eagerly evaluated language, and the Lambda Calculus is lazily evaluated. [↩](https://raganwald.com/2018/09/10/why-y.html#fnref:ya)

5. As alluded to, there is an enormous significance to the Y combinator beyond writing recursive JavaScript functions that are decoupled from themselves. Deriving the Y combinator is interesting in its own right, and highlighting the relationship between the M combinator and the Y combinator is something that is rarely mentioned in casual blogs.

   If the subject piques your interest, be sure to look into point-free programming, fixed point functions, recursion theory, … and most especially, read Raymond Smullyan’s [To Mock a Mockingbird](https://en.wikipedia.org/wiki/To_Mock_a_Mockingbird). [↩](https://raganwald.com/2018/09/10/why-y.html#fnref:yy)

6. The use of the why bird or Y Combinator to implement recursive memoization has been discussed a number of times before, including [here](https://medium.com/@JosephJnk/variations-on-the-y-combinator-and-recursion-cd8d2a7f1a2c) and [here](http://blog.klipse.tech/lambda/2016/08/10/y-combinator-app.html) and [here](https://stackoverflow.com/questions/48261905/how-to-get-caching-with-the-y-combinator-for-this-function), and most especially [here](http://matt.might.net/articles/implementation-of-recursive-fixed-point-y-combinator-in-javascript-for-memoization/). [↩](https://raganwald.com/2018/09/10/why-y.html#fnref:prior-art)

7. A more complete exploration of ways to convert recursive functions to non-recursive functions can be found in [Recursion? We don’t need no stinking recursion!](https://raganwald.com/2018/05/20/we-dont-need-no-stinking-recursion.html), and its follow-up, [A Trick of the Tail](https://raganwald.com/2018/05/27/tail). [↩](https://raganwald.com/2018/09/10/why-y.html#fnref:recursion)

8. Well, *actually*: If we use the original `trampoline` implementation, we explicitly create the thunks that cause the trampolining, so if we return a thunk, we explicitly expect this call to be in tail position. With this function, it’s all done behind the scenes by a function that has nearly the same contract as the why bird.

   However, the long-tailed widowbird is not exactly the same as the why bird. The why bird works with any function (even a non-recursive function). If it happens to be tail-recursive, the why bird works just fine, although it leaves optimization up to the JavaScript engine.

   The long-tailed widowbird, on the other hand, does **not** work with any function, it works with functions that are not recursive, and functions that are tail-recursive. If we pass it a function that is recursive but not tail-recursive, it will not work. We have decoupled the function from its implementation, but not the implementation from the function.