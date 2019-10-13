# Chapter 3 Core operators

This chapter covers

* Introducing disposal of streams
* Exploring common RxJS operators
* Building fluent method chains with `map`, `reduce`, and `filter`
* Additional aggregate operators

In the first two chapters, you learned that RxJS draws inspiration from functional programming and reactive programming. Both paradigms are oriented around data flows and the propagation of change through a chain of functions known as operators. Operators are pure functions that create a new observable based on the current one—the original is unchanged. In this chapter, you'll learn about some of the most widely used RxJS observable operators that you can use to create a pipeline that transforms a sequence of events into the output you desire. 

A common theme in this chapter is creating observables using a declarative style of coding, which originates from FP. You can lift sequences of data of any size into an observable context, as well as data generated or emitted over time, with the goal of creating a unified programming model for any type of data source, static or dynamic. Before we dive into the operators used to apply transformations onto the data that flows through an observable sequence, it's important to understand that, unlike many AJAX libraries, observables can be cancelled. 

## 3.1 Evaluating and cancelling streams

Imagine someone making a long-running AJAX call requesting considerable data from the server. But shortly after spawning this call, the user navigates away from the page by clicking some other button. What happens to the original AJAX request? Consider another example. You begin a client-side interval to poll for certain data to become available, but an exception occurs and the data never becomes available. Should these processes be allowed to run wild and take up system resources? We're guessing no.

A stream, as it exists in RxJS, is an object with a deterministic lifespan defined almost entirely by you, the programmer. JavaScript, unlike some other languages, has few distinct types, most of those types mirroring the simplicity of JSON. Additionally, there's little support within JavaScript for memory management because this has historically been left to browser manufacturers to worry about. Although both of these features make JavaScript a marvelously simple language to learn and use, they also somewhat obscure what's really happening under your application's plumbing.

In languages like C and C++, there exists an extremely fine-grained approach to control not only the specific data structure you use but also its exact lifetime in memory—you have complete control of allocating and deallocating objects in memory. On the other hand, in JavaScript, the lifetime of objects is controlled by the garbage collector, and rightfully so. The garbage collector is a process operated by the runtime engine that's running your application. It will periodically run and free up memory associated with any unused references. The garbage collector does so by keeping track of the references that are kept between various objects in the application—this is known as *ref counting*. When it detects that an object is no longer referenced, it becomes a candidate for disposal. A failure to find references that are no longer in use results in a memory leak. Memory leaks are generally an indicator of either sloppy design or reference tracking and can result in a runaway system footprint that results in either the user or the system killing your application, because it becomes unresponsive at that point.

> **CAUTION** This notion of automatic garbage collection gives us JavaScript developers a false impression that we need not care about memory management. This is a mistake when attempting to write our own event-handling code. RxJS frees us from this by implementing a mechanism to unsubscribe or effectively clean up attached listeners from any event emitters such as the DOM.

In older browser implementations, this used to be a big problem, particularly with Internet Explorer's event-handling system. Modern browsers are now much more efficient at this, and libraries such as RxJS are tuned to avoid many of these problems.

### 3.1.1 Downside of eager allocation

An important point to remember when dealing with RxJS is that the lifetime of a stream doesn't start with the creation of an observable. It begins when it's subscribed to. Hence, there's little overhead in creating and initializing one, because it begins in a dormant state and doesn't generate or emit events without an observer subscribed to it. It's analogous to the old adage "If an observable is created by your application, and no one subscribes to it, does it emit an event?"

In computing terms, an object that creates data only when needed is known as a *lazy data source*. This is in sharp contrast to JavaScript, which has strict eager evaluation. The terms *lazy* and *eager* refer to when an application requests memory from the system and how much it requests up front. Lazy allocation is always done when the space is actually needed (or on demand), whereas eager allocation is performed up front as soon as the object is scoped. In the eager scheme, there's an up-front cost to allocation and there exists the possibility that you'll overallocate because you don't know how much space will be used. Lazy allocation, on the other hand, waits until the space is needed and pays the penalty for allocation at runtime. This allows frameworks to be really smart and avoid overallocating space in certain situations. To illustrate the difference, we'll show you how a popular JavaScript array method, `slice()`, would work under eager and lazy evaluation. Consider a function called `range(start, end)`, which generates an array of numbers from start to end. Generating an infinite number of elements and taking the first five would look like the scheme in figure 3.1.

Figure 3.1 In an eager allocation scheme, the program halts before it can execute the `slice(5)` function because it runs out of memory.

Here it is in code:

```js
range(1, Number.POSITIVE_INFINITY).slice(0, 5); //-> Browser halts
```

With JavaScript's eager evaluation, this code will never get past the `range()` function, because it needs to generate numbers infinitely (or until you run out of memory and crash). In other words, eager evaluation means executing each portion of an expression fully before moving on to the next. On the other hand, if JavaScript functions were lazy, then this code would only need to generate the first five elements, as shown in figure 3.2.

Figure 3.2 In a lazy allocation scheme, the runtime waits until the result of this expression is needed and only then runs it through the program, allocating only the resources it needs to request.

In this case, the entire evaluation of the expression waits until the result of the expression is needed. In RxJS, the strategy is precisely this: wait until a subscriber subscribes to the observable expression, and then begin to initialize any required data structures. You'll see later on that using lazy evaluation allows RxJS to perform internal data structure optimizations and reuse.

### 3.1.2 Lazy allocation and subscribing to observables

RxJS avoids premature allocation of data in two ways. The first, as we mentioned, is the use of a lazy subscription mechanism. The second is that an observable pushes data as soon as the event is emitted instead of holding it statically in memory. In chapter 4, we'll discuss the buffering operators, which can be used to transiently store data for either a period of time or until a certain condition is met, if you wish to do so. But by default, the data is emitted downstream as soon as it's received. 

A lazy subscription means that an observable will remain in a dormant state until it's activated by an event that it finds interesting. Consider this example that again generates an infinite number of events separated by half a second:

```js
const source$ = Rx.Observable.create(observer => {
  let i = 0;
  setInterval(() => {
    observer.next(i++); //*1
  }, 500);  
});
```

1. This interval will keep on emitting events every 500 milliseconds until something stops it.

The cost of allocating an observable instance is fixed, unlike an array, which is a dynamic object with potential unbounded growth. It must be this way; otherwise, it would be impossible to store each user's clicks, key presses, or mouse moves. To activate `source$`, an observer must first subscribe to it via `subscribe()`. A call to `subscribe` will take the observable out of its dormant state and inform it that it can begin producing values—in this case, starting the allocation for events 1, 2, 3, 4, 5, and so on every half-second. Because an observable is an abstraction over many different data sources, the effect will vary from source to source. 

The second advantage to a lazy subscription is that the observable doesn't hold onto data by default. In the previous example, each event generated by the interval will be processed and then dropped. This is what we mean when we say that the observable is streaming in nature rather than pooled. This discard-by-default semantic means that you never have to worry about unbounded memory growth sneaking up on you, causing memory leaks. When writing native event-driven JavaScript code, especially in older browsers, memory leaks can occur if you neglect event management and disposal.

### 3.1.3 Disposing of subscriptions: explicit cancellation

Just as important as when memory is allocated is when it is deallocated or released back to the application. For example, rich JavaScript UIs bind event handlers to potentially thousands of elements. After the user has finished interacting with a certain part of the UI, there's no reason for those objects to exist and take up memory. As discussed earlier, the garbage collector is fairly smart about how it cleans up memory. Unfortunately, it's able to do so only if the references to those objects are found to be unused or if no reference cycles are formed, which tends to occur frequently in native event-handling code. 

It's easy to initialize objects and then forget about them without removing references to them, which prevents the application from ever recovering that memory (this might not be a concern with small scripts but can easily become an issue with modern, client-heavy applications). For example, you can see the problem better in this simple code where we listen for right-click events on a menu item, perhaps to show a custom context menu:

```js
document.addEventListener('mouseup', e => {    
  if (e.button === 2)
    showCustomContextMenu();
  e.stopPropagation();
});
```

Many developers may not even recognize the problem with this code. The problem arises from the fact that in order to unsubscribe from this event we need the reference to the function that was passed into the event handler (the inlined lambda expression). Because we're trying to use this idiom as much as possible, we end up creating a handler that we can't unsubscribe from—if we even remember to unsubscribe from it at all. To make matters worse, if we had nested event handlers or subscribed to other events from within this one, we'd created yet another level of complexity and potential for more memory to leak.

For older web applications (the Web 1.0 years), memory deallocation wasn't so much of a problem because navigation between pages forced a page reload, which cleared out JavaScript's runtime footprint. Today, as single-page applications grow in popularity (the Web 2.0+ era) and clients become more modern and richer, memory pressure becomes a real threat; objects can now conceivably exist for the duration of the entire application's lifespan loaded into the browser.

All this is not meant to sound alarmist. We expect that (just as they have since JavaScript's inception) garbage collectors will continue to improve, and many applications will run without issue. But proper memory management is still a good thing for all applications.

This is why we need sophisticated libraries like RxJS. In RxJS, the producer is the one responsible for unsubscribing. Managing a subscription is handled through an object of type `Subscription` ~~(also known as a `Disposable` in RxJS 4)~~ returned from a call to `subscribe()`, which implements the mechanism to dispose of the source stream. If we've finished with the observable and no longer wish to receive events from it, we can call `unsubscribe()` to tear it down; this is known as explicit cancellation. Here's a short example:

```js
const mouseClicks = Rx.Observable.fromEvent(document, 'mouseup'); 
const subscription = mouseClicks.subscribe(someMouseClickObserver);
... moments later
subscription.unsubscribe(); //*1
```

1. Tears down the stream and frees up any allocated objects

This tearing-down process will stop further events from going to any registered observers and will immediately release all resources allocated by the observable. The subscription instance handles the entire unsubscription process, and it's able to do this because every observable also defines how it will be disposed. Additionally, as you'll see in later chapters, this behavior acts on the *entire* observable, meaning that *all* resources allocated by a given stream can be deallocated cleanly without additional boilerplate and without the risk of orphaning objects in memory.

Recall that in chapter 2 we introduced the `Rx.Observable.create()` method, which could be used to create arbitrary observables. The final step in creating it was to indicate how `subscribe` would dispose of it, and that's your responsibility to implement. Going back to our progress indicator code, add the unsubscription mechanism at the end, like this. 

Listing 3.1 Disposing of an observable

```js
const progressBar$ = Rx.Observable.create(observer => {
    const OFFSET = 3000;
    const SPEED = 50;
    let val = 0;
    let timeoutId = 0;
    function progress() {
        if (++val <= 100) {
            observer.next(val);
            timeoutId = setTimeout(progress, SPEED);
        }
        else {
            observer.complete();
        }
    };
    timeoutId = setTimeout(progress, OFFSET);
    return () => {                             //*1
        clearTimeout(timeoutId);
    };
});
```

1. Function that executes when the unsubscribe method is called. Describes how to cancel that timeout upon disposal.

The function added at the end of the observable body becomes the body of the `unsubscribe()` method of the returned `Subscription` object. In essence, each observable provides the keys to its own destruction during its creation. Every time a subscription occurs from an observer, it passes back a way to clean itself up (analogous to the `finally` clause after a `try/catch`). Because every observable provides this self-contained, self-destruct button, you can also compose its subscriptions such that you can always tear them down correctly no matter how complex the underlying observable is. 

> **CUSTOM OBSERVABLES** If you're creating a custom observable with `create()`, and it happens to emulate an infinite interval stream, you're responsible for supplying the proper unsubscribe behavior, or it will run indefinitely and cause memory to leak.

The examples we've shown so far for the most part involve setting timed intervals to generate events, which support cancellation through `clearInterval()`. But what happens to data sources that don't support cancellation? Let's jump into that next. 

### 3.1.4 Cancellation mismatch between RxJS and other APIs

RxJS observables provide a straightforward mechanism for cancelling and disposing of event streams. But this simplicity can be deceiving when used in conjunction with other JavaScript APIs. For example, you might encounter problems when trying to cancel observables that wrap promises; look at the next listing.

Listing 3.2 Disposing of a promise

```js
const promise = new Promise((resolve, reject) => { //*1
    setTimeout(() => {
        resolve(42);
    }, 10000);
});
promise.then(val => {
    console.log(`In then(): ${val}`);              //*2
});

const subscription$ = Rx.Observable.fromPromise(promise).subscribe(val => {
    console.log(`In subscribe(): ${val}`);         //*3
});
subscription$.unsubscribe();                       //*4
```

1. Creates a promise that resolves to 42 after 10 
2. Handles the resolved promise value
3. Wraps an observable around the Promise API
4. Attempts to dispose of the observable

As you can see from listing 3.2, you dispose of the observable thinking it would also take care of the underlying promise. The observable object itself was properly disposed of; surprisingly, although you attempt to explicitly cancel the event as well, after 10 seconds this program emits the following (apparently, JavaScript promises can't be broken after all):

```js
"In then(): 42"
```

So, what happened? This process is explained in figure 3.3.

Figure 3.3 The cancellation of an observable doesn't affect the underlying promise.

What happens is that promises were not design to be cancelled. Once a `Promise` object begins executing (gets into a pending status), it tries to become fulfilled by either resolving or rejecting the underlying result, as the case may be.

RxJS makes it easy to integrate with external APIs, but you must be mindful that there's a mismatch of design philosophies between an API designed to emit a single value (promise) and one that supports infinite values (observable). This is one use case, but it could also happen if you integrate with other APIs that aren't RxJS aware. Most of the time, though, you don't have to worry about cancelling subscriptions yourself because many RxJS operators do this for you. 

Now that we've covered creating and cancelling streams, in the next section we'll begin with the more popular operators that are essential to any RxJS program.

## 3.2 Popular RxJS observable operators

Though the subscription and disposal semantics of RxJS are useful in managing resources to avoid leaky event handlers, they're only part of the story. But keep in mind what thinking reactively is all about; instead of you controlling what goes on a stream by creating a custom observable and pushing events through `observer.next()`, it's preferable to relinquish that control and react when the time comes—you want to always be reactive! This means allowing RxJS factory operators (`of()`, `from()`, and others) to wrap an event source of interest and create the observable sequence with which to apply the business logic you desire. Hence, being reactive involves defining what a program will do when a value is pushed some time in the future. 

This is where RxJS shines, and it's because of its fully loaded arsenal of out-of-the-box operators, which you can use to create expressive streams of logical data flows. You can create flows to solve virtually any problem, including creating responsive web forms, drag and drop, and even games. 

An operator is a small piece of declarative functionality that allows you to inject logic into an observable's pipeline. An operator is a pure, higher-order function as well, which means it never changes the observable object it's operating under (called the source), but rather it returns a new observable that continues the chain. FP best practices come into play at this point because the functions composing your business logic, the building blocks of your solution, should be done using pure functions as much as possible. These operators can be used to inspect, alter, create, or delay events after they leave the data source but before they reach the consumer; in other words, anything in your business logic pipeline is handled by the combination of one or more operators, which drive the execution of the pure functions of your program. And if that's not enough, RxJS operators are also lazily evaluated!

Recall that in chapter 2 (figure 2.10) we highlighted four fundamental types of computing tasks. We split them into two dimensions depending on whether they performed work synchronously or asynchronously and whether they acted on single values or collections. Manipulating a single value is a relatively trivial task (known as a *singleton stream*), given that you can inspect its properties and manipulate it directly. In most cases, though, you want streams to act across a range of values rather than just one and done. The computing model behind RxJS encourages you to work with function chains that process data, similar to a conveyor belt in an assembly line, as shown in figure 3.4.

Figure 3.4 An assembly line where operators represent individual stations and each has its own task to perform on each piece of data that passes by

Another important design principle of RxJS is to provide a computing model that's similar to what you're accustomed to. Inspired in the Array#extras APIs introduced in ES5, RxJS features its own version of core operators such as `map`, `filter`, and `reduce`. Because these are some of the more frequently used, let's start with them.

### 3.2.1 Introducing the core operators

Operators come in two varieties: as instance methods or as static methods of the observable type. Part of the RxJS 5 rework was the drastic simplification of the API surface, which consisted of a sheer reduction in the number of operators as well as a simplification of their usage. Hence, most of the operators in RxJS 5 can be invoked as static or as instance methods (when we say *instance*, we refer to invoking them using the dot (.) notation on an observable instance). 

RxJS comes with many operators built in that handle many common tasks such as working with collections, extracting elements from the stream, manipulating and transforming the data, handling errors, and others. In this section, we'll focus on the three that you'll use about 80% of the time—`map`, `filter`, and `reduce`—as well as a variation of `reduce` called `scan`.

#### MAPPING OPERATIONS ON OBSERVABLES

By far the most common operator that you'll likely come across when dealing with RxJS is `map()`. RxJS isn't the only library to implement it, and all the libraries follow the same FP principles. In FP, `map()` belongs to a category of operations called *transformational* because it changes the nature of data running through the observable by applying a function; therefore, it's a single output value or a one-to-one transformation. In symbolic notation, you write it as `map :: x -> f(x)`, where for a given value `x` you can associate an input of `x` with an output of `f(x)`. Consider a quick example that applies a given percentage value onto a set of prices:

```js
const addSixPercent = x => x + (x * .06);
Rx.Observable.of(10.0, 20.0, 30.0, 40.0)
  .map(addSixPercent) //*1
  .subscribe(console.log); //-> 10.6, 21.2, 31.8, 42.4
```

1. Applies this function onto each value of the source observable

Mapping functions is a fundamental process when transforming data from one type to another. For example, say you had a list of user IDs for which you wanted to fetch GitHub information. Mapping a function like `ajax()` over the set of IDs yields an array of JSON account objects. 

In RxJS, you want to map functions across all the elements emitted from an observable. To help you better visualize operators, we'll use the marble diagrams. Recall that arrows and symbolic characters represent the various operations that convert the input stream into the output stream, as shown in figure 3.5.

Figure 3.5 The `map` operator will produce a one-to-one transformation that will convert an input value into an output value by a given process. In this case, `map` takes a URL string and converts it into an array of users by means of the mapping function. In the diagram, operators are encoded inside a box that illustrates the function that's passed in.

We'll use these vertical transformations in figure 3.5 to depict operations that take one form of data and convert it to another. By design, this function in RxJS has the exact same signature as that of array:

```js
Array.prototype.map         :: a => b; for all a in Array<a>
Rx.Observable.prototype.map :: a => b; for all a in Observable<a>
```

Like with arrays, observable's `map()` is immutable, which means it won't change the original but instead will transform the value passed through it. Also, as you can see in figure 3.5, the output will always be the same size as the input because mapping is a one-to-one relationship that preserves structure. It's left up to you to decide exactly what the transform function is, depending on your business logic; `map()` simply guarantees that it will be called on every value passing through the stream as it's propagated downstream to the next operator in the chain. 

We mentioned briefly before that all RxJS operators are pure. To show you what this means, we'll demonstrate the case of `map()`. Transforming a `String` into an ID looks like figure 3.6. 

Figure 3.6 Mapping a function from `String` to `Array` to a source Observable creates a new `Observable` with the result of the function.

Now, let's look at the code for this. Suppose you need to convert a collection of strings into a corresponding comma-separated value (CSV) array. Here's a simple stream that will accomplish this.

Listing 3.3 Mapping functions over streams

```js
Rx.Observable.from([
   'The quick brown fox',
   'jumps over the lazy dog'
   ])
 .map(str => str.split(' '))          //*1
 .do(arr => console.log(arr.length))  //*2
 .subscribe(console.log);
```

1. Maps a set of functions to extract the value from 
2. RxJS `.do()` is a utility operator that's useful for effective actions such as logging to the screen. This can be handy for debugging or tracing the values flowing through a stream.

For those familiar with design patterns, mapping functions is analogous to the adapter pattern (as shown in the famous "Gang of Four" book titled *Design Patterns: Elements of Reusable Object-Oriented Software*). In the adapter pattern, an object interacts with two otherwise incompatible interfaces and allows information to flow between them, adapting them to each other. In a similar fashion, you use `map()` to create a type compatibility between the producer and the consumer of data. The purpose of using it is to convert the raw input data into something the consumers can understand. In this way, the adaptation is done from the producer to the consumer.

But sometimes there can be too much data to process, and you may not be interested in all of it. For this, there's an operator to discard unwanted events.

#### FILTERING OUT UNWANTED EVENTS

Filtering is the process of removing unwanted items from a stream. The criteria to remove these elements is passed in as a selector function, also called the `predicate`. Here's a simple example of how this operator works. Say you need you need to place restrictions on input boxes for numerical quantities. It's probably a good idea to place a business rule over text boxes rejecting any non-numerical input. Whenever you're thinking about rejecting, removing, narrowing, or selecting data, you can do that easily using a filtering operator called `filter()` and inspecting the keyCode property of the keystroke, as shown in the following listing.

Listing 3.4 Filtering events from a stream

```js
const isNumericalKeyCode = code => code >= 48 && code <= 57;
const input = document.querySelector('#input');
Rx.Observable.fromEvent(input, 'keyup')
  .pluck('keyCode')            //*1
  .filter(isNumericalKeyCode)  //*2
  .subscribe(code => console.log(`User typed: ${String.fromCharCode(code)}`));
```

1. Extracts this property from the object passing through the observable
2. Accepts only keys in the numerical range

Also, you can use it to ignore unwanted mouse clicks, touch events, and others. It could be that you're interested only in data that meets certain criteria, or you need only a certain subset of the data. In some cases, allowing too much data through can have an adverse effect on the performance of your application. Think about building an API for users to access their account history for the month; if on every request you simply dump their entire account history, you'd quickly find both your API and your clients overwhelmed. To make matters worse, your application won't scale to the size of data being processed. Filtering could be used to generate different views if the user wanted only debits, credits, or transactions after a certain month.

An easy way to think about filtering is to consider the job interview process (every developer's favorite activity). When recruiting people for a specific job, one of the first things to look for is the candidates' previous experience in order to determine if they have the right skill-set for the position. If the job requires programming, then you'd expect that the candidates should have some sort of programming background listed on their resume. If not, you could exclude them from your final interview list.

Suppose you modeled your applicant-screening process as an array; then, you could write your filtering operation using the array's `filter()` method. Because observables implement the same filtering semantics, you're already familiar with using a predicate function (also called a *discriminant*) in `filter()` that returns true for candidates who will be selected to move on to the next round. Here's the dataset you'll use:

```js
let candidates = [               
    {name: 'Brendan Eich', experience : 'JavaScript Inventor'},
    {name: 'Emmet Brown', experience: 'Historian'},
    {name: 'George Lucas', experience: 'Sci-fi writer'},
    {name: 'Alberto Perez', experience: 'Zumba Instructor'},
    {name: 'Bjarne Stroustrup', experience: 'C++ Developer'}
];
```

Whether this data arrives because of an AJAX call or a DOM event, the observable treats it all the same way. So for now, you'll stick with a simple array. In this case, you can wrap the data with an observable and keep only the candidates who will be considered for this JavaScript job:

```js
const hasJsExperience = bg => bg.toLowerCase().includes('javascript');
const candidates$ = Rx.Observable.from(candidates);
candidates$
   .filter(candidate => hasJsExperience(candidate.experience))
   .subscribe(console.log); //-> prints "Brendan Eich" 
```

Figure 3.7 shows what's happening behind the scenes. Like `map()`, `filter()` works vertically removing values from the resulting stream.

Figure 3.7 The `filter` operator is used to discard candidates who don't have any JavaScript experience.

Functions `map()` and `filter()` are similar in that they take a single function as their parameter. But whereas the function passed to `map` converted the input value into an output value, the `filter()` function is used merely as a criterion that decides whether to keep the event in the stream or not. As you know, JavaScript being loosely typed will accept any "truthy" value as a pass, while any "falsy" values will cause it to reject the event. 

---

##### Truthy vs. falsy

In JavaScript, *truthy* is any value that can be coerced to a `true` Boolean value. This includes objects, arrays, non-zero numbers, non-empty strings, and of course the `true` Boolean value. Meanwhile, *falsy* would be represented by `0`, `''`, `null`, `undefined`, or `false`. In practice, although JavaScript will accept all these types without question, it's often best for clarity's sake to return a Boolean value.

---

`map` and `filter` work well together in scenarios where you don't want to apply a mapping function to each element but apply it to only the subset you care about. But `filter` isn't the only function married to `map`; let's not forget about the powerful `map/reduce` combinations. 

#### AGGREGATING RESULTS WITH REDUCE

Sometimes you aren't interested in acting on each item in a collection in isolation; sometimes you want to look at the collection in aggregate rather than piecemeal. For instance, suppose you want to take the average value of a collection of numbers or you want to turn a sequence into a mathematical series. This type of operation is called a *reduction* or an *aggregation*, with the result as a single value output instead of another collection. Once again, arrays come with a built-in `reduce` operator for this purpose, and observables follow suit. `reduce` is a bit more involved than the other two; here's the function signature: 

```js
Rx.Observable.reduce(accumulatorFunction, [initialValue]);
```

The `accumulator` function is called on every element, and it's given the current running total and the new value as parameters. The initial value (optional) is used to begin the accumulation process; we're using 0 to begin the addition. Here's a simple example to illustrate how `reduce()` works. Suppose you want to compute the user's spending for the month by totaling all their transactions. For this example, these transaction objects have a property called `amount`.

Listing 3.5 Using `reduce()` to compute spending 

```js
const add = (x, y) => x + y;

Rx.Observable.from([
    { date: '2016-07-01', amount: -320.00, },
    { date: '2016-07-13', amount: 1000.00, },
    { date: '2016-07-22', amount: 45.0, },
])
    .pluck('amount') //*1
    .reduce(add, 0)  //*2
    .subscribe(console.log);
```

1. Extracts the amount property
2. Reduces the set of amount values with an add function

It's important to notice that `reduce()` with observables works a bit differently than `map()` and `filter()`. With arrays, `reduce()` doesn't return another array; instead, it produces a single raw value, which is the result of the reduction. The observable's `reduce()`, on the other hand, continues the previous pattern of returning a new singleton observable. This distinction will become important in section 3.3 when we talk more about operator chaining. Figure 3.8 is a visual representation of the previous code. Reduction is an operation that moves horizontally through the stream. 

Figure 3.8 The `reduce` operator moving horizontally, accumulating every value through the stream using the `add` function

Suppose you needed to traverse through the candidate stream and group all the candidates with a technical background (that is, with knowledge of C++ or JavaScript):

```js
Rx.Observable.from(candidates)
    .filter(candidate => {   //*1
        const bg = candidate.experience.toLowerCase();
        return bg.includes('javascript') || bg.includes('c++');
    })
    .reduce((acc, obj) => {
        acc.push(obj.name);  //*2
        return acc;
    }, [])                   //*3
    .subscribe(console.log); //-> ["Brendan Eich", "Bjarne Stroustrup"]
```

1. Filters all candidates who have no knowledge of a programming language
2. Adds a candidate name to the array
3. Begins with an empty array (called the seed)

As you can see, `reduce` applies an accumulator function over the observable sequence initialized with the first seed value, which will be used to begin the aggregation process. Because `reduce()` returns a single value, there's a need for partial accumulation as well. We'll look at a variation of `reduce()` called `scan()`.

#### SCANNING AGGREGATE DATA

RxJS uses `scan()` to apply an accumulator function over an observable sequence (just like `reduce()`) but returns each intermediate result as the accumulation process is happening and not all at once. This is useful to obtain progress information about how data is being aggregated with each event.

Changing the previous code to use `scan()` as a direct swap-in replacement of `reduce()` reveals the intermediate steps of the accumulation:

```js
Rx.Observable.from(candidates)
    .filter(candidate => {
        const bg = candidate.experience.toLowerCase();
        return bg.includes('javascript') || bg.includes('c++');
    })
    .scan((acc, obj) => {     //*1
        acc.push(obj.name);
        return acc;
    }, [])
    .subscribe(console.log);
//-> ["Brendan Eich"]         //*2
//-> ["Brendan Eich", "Bjarne Stroustrup"]
```

1. Scan can be used as a direct replacement of reduce. ~~In RxJS 4, you would have had to change the seed parameter to be the first one. This was fixed in RxJS 5,~~ with scan now having the same signature as reduce.
2. As soon as it finds the first event, it emits it and accumulates it. A second emission happens when the second event is found, returning the current state of the accumulation.

Aside from `scan()`, the symmetries between arrays and observables are no coincidence. This signature was chosen specifically because it's so simple and because it's one that many JavaScript developers are already familiar with. But here the similarities end. Remember that these methods by themselves don't cause any work to run on the stream (only a subscriber can); instead, when an operator is called on an observable, it's configuring the observable for future values. Recall our definition of a stream as a specification of a dynamic value. This is a key distinction between the operators that you'll see with arrays and those with observables, and you'll learn in later chapters that arrays represent work happening now, whereas observables represent work in the future.

## 3.3 Sequencing operator pipelines with aggregates

One principle of FP is the ability to construct lazy function chains. In this section, we'll show you how to mix and match the main observable operators you just learned about together with a few other functions known as *aggregates*. Aggregate functions let you do useful things like keeping track of a running total, taking only a subset of the total set of data, returning default values, and others. Some functional libraries you might have heard of or used before, such as `Lodash.js` and `Underscore.js`, have ample support for this. First, it's important to understand that observable sequences must be self-contained. 

### 3.3.1 Self-contained pipelines and referential transparency

Function chains utilize JavaScript's power of higher-order functions to act as the single providers of the business logic. You saw examples of this before such as the `filter` function taking a predicate parameter. Also, observable pipelines should be self-contained, which essentially means they're side effect-free (keep in mind that if your business logic functions are pure, your entire program is pure and stable as well). A pure pipeline doesn't allow any references to leak out of the observable's context. Once an event is lifted into the context, it's contained and transformed through a sequence of operators. Earlier we showed that it's possible to group operations together to create more-expressive logic. In RxJS, we call this process *operator chaining* or *fluent programming*. The analogy of a self-contained pipeline works great as a visualization aid, as shown in figure 3.9.

Consider this example:

```js
let sinceLast = new Date();
Rx.Observable.fromEvent(document, 'mouseup')
    .filter(e => {
        let timeElapsed = new Date() - sinceLast; //*1
        sinceLast = new Date();
        return timeElapsed < 200;
    }).subscribe(() => console.log('double clicked'));
```

1. Careless side effects of reading and writing to an external variable

Figure 3.9 A self-contained pipeline is one where all of its operations are side effect-free and work 
strictly on the data coming from previous operators. Operators might be any of `map`, `filter`, 
`reduce`, and others you'll learn about in this book.

This code is an example of poorly designed scope management in which the state variable `sinceLast` is allowed to live outside the observable's context. The result is that the observable is no longer stateless, and the lifecycles of the state and the observable are now dependent on each other.

It's important to understand that when you create an observable, you're creating an ecosystem or a bounded context. That ecosystem is a closed loop that begins with a subscription and ends with a disposal. If you were to look at the observable through an FP lens, you'd see that the internals of that observable remain completely stateless and walled off somewhat from the rest of the application. The scope of the callbacks that are passed into the operators should remain small and local. Mixing code that has external side effects not only introduces difficult-to-track complexity but also removes one of the key advantages to using observables, which is their well-defined lifespan—creation and disposal should leave the system in the same state they found it in.

---

##### What is a bounded context?

A bounded context is a design principle originating from domain-driven design, which states that entities pertaining to a single domain model should be highly cohesive and expose only the necessary interface to interact with other contexts. You can extend this definition to the Observable type as a form of context that hides the nature of the data that's pushed through it, allowing you to transform it by a ubiquitous language made up from the limited set of operators being exposed and independently of what happens in the outside world.

---

At a glance, a single subscription to this observable will function correctly, assuming that no other code manipulates `sinceLast`. But if this observable is subscribed to a second time, the result is no longer the same. An observable must always produce the same results given the same events passing through it (that is, pressing the same key combination should always yield the same data to the observers), a quality known in FP as *referential transparency*.

Each invocation of `subscribe()` does more than start an event emitter. It spins off a brand-new pipeline that will be independent of any other pipelines that were created by subsequent calls to `subscribe()`. This behavior is intentional in order to minimize side effects and be referentially transparent; similarly, the result of an observable should be the result of the data passed through it, not the number of parallel observables that are also active. You'll see in the next chapter on dealing with time in RxJS that the sort of operation used in the previous code sample is unnecessary.

As mentioned earlier, the operator chain is core to the design of an RxJS operator: every operator must perform some work on the data passing through it and then wrap it into another observable instance that gets returned. In this manner, the subscription gets internally passed around from one context to the next. To show how this works, you'll add your own operator using prototype extension (using ES6, you could also do it by extending from the `Observable` class); this operator is the logical inverse of `filter()`, called `exclude()`, and is shown in the next listing.

Listing 3.6 Custom exclude operator

```js
function exclude(predicate) {
    return Rx.Observable.create(subscriber => { //*1
        let source = this;                      //*2
        return source.subscribe(value => {
            try {                               //*3
                if (!predicate(value)) {
                    subscriber.next(value);     //*4
                }
            }
            catch (err) {
                subscriber.error(err);
            }
        },
            err => subscriber.error(err),        //*5
            () => subscriber.complete());
    });
}

Rx.Observable.prototype.exclude = exclude;       //*6
```

1. Creates a new observable context to return with the new result
2. Because you're in a lambda function, "this" points to the outer scope.
3. Catches errors from user-provided callbacks
4. Passes the next value to the new operator in the chain
5. Be sure to handle errors appropriately and pass them along.
6. Adds the operator by extending the Observable prototype

As you can see from this snippet, every operator creates a brand-new observable, transforming the data in its own way and delegating it to the next subscriber in the chain. You can use it to exclude all even numbers as such:

```js
Rx.Observable.from([1, 2, 3, 4, 5])
  .exclude(x => x % 2 === 0)
  .subscribe(console.log);
```

Furthermore, operation chaining in combination with an observable's lazy evaluation gives RxJS an important performance advantage over arrays, which we'll discuss next.

### 3.3.2 Performance advantages of sequencing with RxJS

Aside from the declarative style of development that encourages you to write side effect-free code, the primary advantage of using observable operators is that there is little or no performance penalty for chaining two methods like `map` and `filter`. Behind the scenes, RxJS produces little overhead because observables themselves are lightweight and inexpensive to create. On the other hand, operator calls on arrays create new instances along the way, which naturally incurs more memory allocations when the collection being processed is large. You can see this with a simple example that uses the full set of parameters for `map()` and `filter()` array functions: 

```js
const original = [1, 2, 3];
const result = original
    .filter((x, idx, arr) => {                          //*1
        console.log(`filtering ${x}, same as original?${original === arr}`); //*2
        return x % 2 !== 0;
    })
    .map((x, idx, arr) => {                             //*1
        console.log(`mapping, same as original?  ${original === arr}`);      //*2
        return x * x;
    }); 
result; //-> [1, 9]
```

1. `map` and `filter` expose extra parameters such as the current index and the source array. Typical implementations of these methods don't use these parameters, but it's good to know they're there.
2. Logging to the console within the pipeline is considered a side effect. We're bending the rule here a bit to illustrate this concept.

Running this code logs the following messages:

```js
"filtering, same as original? true"
"filtering, same as original? true"
"filtering, same as original? true"
"mapping, same as original? false"
"mapping, same as original? false"
```

You can visualize the difference between both approaches in figure 3.10. 

Figure 3.10 The array's `filter` and `map` operators generate intermediate, wasteful data structures. RxJS observables are optimized and process events entirely through all functions at once, avoiding intermediate storage altogether.

RxJS, by contrast, doesn't create intermediate data structures. As you can see in the previous example, `filter()` works on the same data structure as the original because it's first on the chain. This operation returns a brand-new array instance that becomes the new owning object on which you call `map`. This can be inefficient on very large collections because new data structures are created and used only once before being garbage collected. In RxJS, the underlying data structure is optimized to process each item through the pipeline from the producer to the consumer at once, avoiding the creation of extra data structures along the way. Let's convert the same code to use observables:

```js
Rx.Observable.from(original)
    .filter(x => {
        console.log(`filtering ${x}`);
        return x % 2 !== 0;
    })
    .map(x => {
        console.log(`mapping ${x}`);
        return x * x;
    })
    .subscribe();
```

Running this code shows you that each element (or mouse click, key press, asynchronous data, and others) passes through the pipeline by itself without creating intermediary storage. The first value, 1, passes through filtering and then through mapping before 2 and 3 are looked at:

```js
"filtering 1"
"mapping 1"
"filtering 2"
"filtering 3"
"mapping 3"
```

Now this is optimal. This fluent chaining pattern hinges on the return type of all observable methods to always return observables. As you know, in arrays, the `reduce` operator breaks the chain of commands because it doesn't return an array, so further chaining becomes impossible. In RxJS, every operator will return an observable instance so that it can support further chaining. This property means that a virtually unlimited variety of combinations can be assembled. Whereas observables are abstractions over various data sources, their operators are just abstractions of those abstractions. That is, just like the adapter methods used to create observables from other library types, an operator is simply an adapter to convert an existing observable into a new one with more-specific functionality.

Before we continue having fun building more chains, we'll introduce another set of aggregate methods that will become handy for building nice expressive business logic. Table 3.1 briefly explains each of these aggregate functions.

Table 3.1 More aggregate operators

| Name        | Description                              |
| ----------- | ---------------------------------------- |
| take(count) | Filtering operator. Returns a specified amount (count) of contiguous elements from an observable sequence. Later, you'll see this is useful to extract a finite set of events from an otherwise infinite stream. |
| first, last | A refinement on the `take` function. Returns the first element in the observable stream or the last, respectively. |
| min, max    | Filtering operators. Work on observables that emit numbers returning the minimum or maximum value of a finite stream, respectively. |
| do          | Utility operator. Invokes an action for each element in the observable sequence to perform some type of side effect. This operator is for debugging and tracing purposes and can be plugged into any step in the pipeline. |

Now, let's have fun with some examples that put some of these to work.

Listing 3.7 Using aggregate operators

```js
Rx.Observable.from(candidates)
    .pluck('experience')
    .take(2)                                   //*1
    .do(val => console.log(`Visiting ${val}`)) //*2
    .subscribe();

// prints "Visiting JavaScript Guru"
//        "Visiting Historian"
```

1. Takes only the first two elements (another filtering operator)
2. Performs the logging routine and passes along the observable sequence

---

##### Effectful computations

The `do` operator is known as an *effectful computation*, which means it will typically cause an effect such as I/O, a database insert, append to the DOM, or write to a file—all of these side effects, of course. The reason why `do()` still preserves the chain is rooted in an FP artifact called the *K combinator*. In simple terms, this is a function that executes any effect but ignores its outcome, just passing the value along in the stream to the next operator. In a way, it's a bridge that intercepts the stream that allows you to invoke any function. It's known in other libraries as the `tap()` operator.

---

Being able to use this repertoire of operators is certainly beneficial because it frees you from having to write them yourself, reducing the probably for bugs to occur (you can find a complete list of all the operators used in this book in appendix B—you're free to use it as a guide). Nevertheless, the functions passed into these operators are solely your responsibility, so please test them thoroughly. We'll revisit testing further in chapter 9.

In this chapter, we talked at length about several of the core operators that come bundled with RxJS. We purposely avoided specifically enumerating all the operators that are available for mapping, filtering, and other tasks. That job is better left to the reference material on GitHub or on the internet. Instead, we wanted to demonstrate how operators are used in conjunction with observables to build chains of logic that let you write streams declaratively, so that they're both easy to understand and easy to extend. We chose what we think of as the set of core operators. We explored how you can build complex logic intuitively using fluent operators. These are operators that act primarily on a single observable and don't introduce any time-based operations. In the next chapter, we'll explore the time aspect of observables, which allows you to handle future data.

## 3.4 Summary

* Streams provide their own mechanisms for cancellation and disposal, which is an improvement over JavaScript's native event system.
* The `Observable` data type enables fluent function chaining that allows the sequential application of operators, using a model similar to that of arrays.
* Unlike JavaScript's native promises, observables have built-in capabilities for disposal and cancellation.
* Functions injected into the operators of an observable sequence contain the business logic of your application and should be side effect-free.
* Observables are self-contained with indefinitely chainable operators.
* Operators act independent of each other and work only on the output of the operator that preceded them.
* The order and type of operators used determine the behavior and the performance characteristics of an observable.























