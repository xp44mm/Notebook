# chapter 5. Applied reactive streams

This chapter covers

* Handling multiple observable sequences with one subscription
* Learning to make observable streams conformant
* Flattening nested observables structures
* Merging a collection of observables into a single output
* Preserving sequence order with concatenation
* Implementing real-world problems: search box, live stock ticker, and drag and drop 

In the previous chapter, we firmly rooted the notion that an observable is a sequence of events over time. You can think about it as the orchestrator or channel through which events are pushed and transformed. So far, we've discussed how to process observable sequences in isolation for the most part, and you learned how you could apply familiar operators over all the elements within an observable in the same way that you could with an array, irrespective of when those elements were emitted. Toward the end of the chapter, we briefly gave you some exposure to an RxJS operator called `combineLatest()`, which is used to combine streams. The reason for these discussions is that other than trivial examples, most of your programming tasks will involve the use of these operators so that the events from one stream propagate and cause a reaction somewhere else. This is where RxJS and the entire reactive paradigm begin to shine and set themselves superior to other conventional asynchronous libraries. 

Now you've entered into new, more sophisticated territory. This chapter is our point of inflection where we'll look at combinatorial operators similar to `combineLatest()` like `merge()`, `switch()`, `concat()`, and others to solve real-world, complex problems. Although there's still a lot to gain from using observables to handle a single event source, like a single button click, in all but the most trivial of applications, you'll quickly find that your logic will require more than a single stream to get the job done. You had a taste of this toward the end of chapter 4 when you implemented a slightly more complicated problem, which required you to combine and buffer the output of multiple streams.

Depending on the complexity of your application, your logic will likely involve the interplay between many observables, potentially containing different types of data. For instance, a user typing keys on the search bar kicks off an entirely different type of stream for fetching data from a server—to put it in Newton's terms, user actions may create reactions in different parts of the application. But combining different observables presents a challenge because you have to learn how to fuse them together. What if their interfaces are different? This happens quite often, especially when modeling complex state machines such as UIs or mashing up data from remote locations. Thus, in this chapter, we'll focus on an important principle in functional programming called *flattening a data type*, which stems from the need to project other observables carrying needed data into a single source. After learning this technique, you'll be prepared to tackle all sorts of complicated flows. Let's begin with the idea that all data sources, whether static or dynamic, can be treated in the exact same manner under the observable programming model, because everything is a stream.

## 5.1 One for all, and all for one!

The addition of time to the observable introduced a new dimension that let you delay, filter, and manipulate elements based on when they were dispatched. By playing with various types of operators and their corresponding selector functions, you were able to determine if, when, and how events made it downstream, so this gave your business logic full control of how data is transformed within the stream. 

Up until now, we've been focusing on single streams. In this chapter, we expand our use of operators and introduce more-advanced ones that allow you to create complex flows and combine them into a single flow. This is essential for any non-trivial state machines where input taken from the user creates a ripple effect on other parts of the system in real time.

Multi-stream scenarios are common in the real world. Generally, you'll find that the more complex your system becomes, the more entangled your streams will be. This linear relation shouldn't surprise you because complexity grows as you inject more business logic into your code. But this is far more maintainable than the exponential complexity growth of a system that deals with asynchronous flows relying only on nested callback functions or even solely on promises.

The combination operators you'll learn in this chapter are indispensable because if isolated streams were unable to interact with each other, it would be up to you to create the necessary boilerplate for those streams to work together. Ugh! You'd likely be no better off than before you started using RxJS!

> **REACTIVE MANIFESTO** According to the Reactive Manifesto, one of the central principles of reactive architectures is elasticity. An elastic system is one that stays responsive under a varying workload. RxJS nicely achieves this because multiple sources of data with varying input rates can be combined in different ways without you having to rewrite or refactor how your code works. 

As an example of when multiple streams come into play, imagine that in addition to supporting mouse handling, you wanted to support touch interfaces. JavaScript already provides built-in support for touch events in most browsers, but adding touch support to an application means introducing a second set of events and logic. Without the right architecture, you'll most likely have to create a whole new set of event handlers for those as well. With your newly developed reactive mindset, you realize that those are all just different streams passing through the same channel. Whether you're using mouse or touch, most of the time those streams kick-start events that need to combine key presses with other HTTP calls, timed intervals, animations, and much more—complex UIs work this way.

Let's look at a quick example. Just as mouse events have `mousedown`, `mouseup`, and `mousemove`, touch events have `touchstart`, `touchend`, and `touchmove`, respectively. This means that, at a minimum, you'll need to create three new streams in order to emulate the same behaviors from mouse events as with touch events and have your code work on mobile browsers. Consider the scenario of two independent streams. The `touchend` event is probably equivalent to `mouseup`, which means that in most cases users will use it in the same way:

```js
const mouseUp$ = Rx.Observable.fromEvent(document, 'mouseup');  
const touchEnd$ = Rx.Observable.fromEvent(document, 'touchend');
```

Each of the streams represents the same action of completing some interaction with your application, whether it's through a mouse or the screen. You know from the previous chapters that you can subscribe to each of the handlers individually as such:

```js
mouseUp$.subscribe(/* Handle mouse click */);  
touchEnd$.subscribe(/* Handle touch click* /); 
```

But this setup is less than ideal. For one, you now have two subscription blocks for what will most likely be identical code. Any code that must be shared between the two will now require a shared external state. In addition, now two subscriptions must be tracked, which introduces one more area of potential memory leaks. It would be in many ways preferable if you could manage both subscriptions with a single block of code without having to worry about the synchronization of both streams, as shown in figure 5.1, especially in this case, because both observables will emit very similar events.

Figure 5.1 A simplified diagram of the desired behavior of ingesting two streams, `mouseUp$` and `touchEnd$`, through a combination operator, and creating a single output block for consumption. In this chapter, you'll learn about the most important combination operators in RxJS.

There are many different ways to join multiple streams into one and take advantage of using a single observer to handle them all. In this section, we'll look at the following strategies:

* *Interleave events by merging streams*—This strategy is useful for forwarding events from multiple streams and is ideal for handling different types of user interaction events like mouse or touch.
* *Preserve order of events by concatenating streams*—This one is used when the order of the events emitted by multiple streams needs to be preserved.
* *Switch to the latest stream data*—This is used when one type of event kicks off another, such as a button click initiating a remote HTTP call or beginning a timer. 

### 5.1.1 Interleave events by merging streams

The simplest operator that combines multiple streams together is `merge()`. It's easy to understand; its purpose is to forward the events from several streams in order of arrival into one observable, like a funnel. Logically, you can think of `merge` as performing an OR of the two events, as shown in figure 5.2. 

Figure 5.2 The `merge()` operator has no logic of its own, other than to combine events from multiple streams in the order in which they're emitted.

We can illustrate this with a simple example: 

```js
const source1$ = Rx.Observable.interval(1000)
   .map(x => `Source 1 ${x}`)
   .take(3);

const source2$ = Rx.Observable.interval(1000)
   .map(y => `Source 2 ${y}`)
   .take(3);

Rx.Observable.merge(source1$, source2$)
   .subscribe(console.log);
```

The resulting stream created from `merge` will emit values every second, alternating between sources 1 and 2. `merge()` has no logic of its own other than placing the events onto a single stream in the order in which they arrive.

In the example of mouse and touch streams, because the two types are virtually interchangeable, you can funnel both through this operator so that you can react to either one in the exact same way or with the same observer logic. Just like most of the operators you'll learn about soon, `merge()` can be found in the static method form:

```js
Rx.Observable.merge(source1$, source2$, ...)
```

Or, it can be found in instance form:

```js
source1$.merge(source$2).merge(...)
```

In the static form, it's a creational operator, as we showed in chapter 4. In other words, you're creating a new stream from the combination of two or more observables. Using it in instance form, we say that an observable is *projected* or *mapped* to the source observable. Both yield the same results, and because operators are pure, both create new observables.

`merge()` can accept either an array or a variable number of streams that are to have their outputs merged into one. Thus, for the previous example, you can create a merged stream easily by passing both sources into the `merge()` operator, as shown in figure 5.3, for both static and instance forms.

Figure 5.3 Static and instance versions of the merge operator between two streams. The results are the same. `merge()` is the simplest operator because it only forwards events from either stream as they come.

Here's the code that combines both streams, in creational form:

```js
Rx.Observable.merge(mouseUp$, touchEnd$)
    .subscribe(/* single observer to consume either event */);
```

Again, you can also write this code where a source observable merges onto another in midstream in instance form:

```js
mouseUp$.merge(touchEnd$) //*1
    .subscribe(/*single observer to consume either event */);
```

1. In this case, both `mouseUp$` and `touchEnd$` are still referred to as the source observables.

The outcome of merging two observables is that they now appear as a single one from the point of view of any instance methods used, all the way down to the observer, so all their outputs are piped to a single output block. This is true for all of the combinatorial operators you'll learn about in this chapter. You now need only worry about a single subscription for the two streams. 

Now, here's something to think about when merging streams is order. The `merge` operator will interleave events from each stream in the order in which the events are received; internally, RxJS does a good job of timestamping each event that gets pushed through the observable.

Each stream, though independent, contributes to the overall output of the sequence. ~~In more statically typed languages, the compiler will also often constrain the types that can go into a merge. This forces the input streams to have a uniform and predicable type for the output.~~ In JavaScript, because these constraints don't exist, it's much easier to merge types that may not even be compatible. But this flexibility can result in some unexpected errors downstream when you're looking to handle one type of event differently than the other. For this, one thing you could do is check your use cases to determine the type, as in the following listing. Again, going back to the touch and click streams, assume you need to print out the coordinates of both touch and click events.

Listing 5.1 Case matching event data resulting from merging mouse and touch streams

```js
Rx.Observable.merge(mouseUp$, touchEnd$) //*1
  .do(event => console.log(event.type))
  .map(event => {
    switch(event.type) { //*2
        case 'touchend':
            return {left: event.changedTouches[0].clientX,
                    top: event.changedTouches[0].clientY};
        case 'mouseup':
            return {left: event.clientX,
                    top:  event.clientY};
    }
}).subscribe(obj => 
             console.log(`Left: ${obj.left}, Top: ${obj.top}`)); 
```

1. Merges the outputs of the two streams
2. Detects the type of the event, builds a compatible type, and constructs the object accordingly

But this sort of behavior should raise some flags as a potential code smell. One of the main reasons to combine observables is that they possess some similarities that you'd like to leverage into simpler code. Introducing more boilerplate code into the mix, only to then switch on type, doesn't serve this purpose; it simply moves the complexity to a different location. 

On another note, keep in mind that in FP you try to avoid imperative control statements like `if/else` whenever possible. Instead, each event should be at least contract compatible with any other, which means that the data emitted from all of them should at least follow the same protocols or structure in order to be consumed by the same observer code. Now, why you should avoid imperative control structures as much as possible has more to do with the notion of using RxJS in a functional style as well as to continue the fluent declaration of an observable chain.

---

**FUNCTIONAL VS. IMPERATIVE** Generally speaking, most of the cases using structures like `for/of` or `if/else` can be satisfied by using RxJS operators like `filter()`, `map()`, `reduce()`, `take()`, `first()`, `last()`, and others. In other words, most of the complex tasks for which you need to implement loops and branches are just instances of a certain combination of these operators. A familiar example of this is an array. Most uses of `for/of` and `if/else` statements can be supplanted by a combination of `map()` and `filter()`. 

---

In cases where you absolutely need to apply specific logic, you can insert operators upstream of where the merge occurs in order to create compatible types ahead of time. Instead of forcing observers to use conditional logic to discern between different types of events (figure 5.4), you should make the stream data conformant before the merge (figure 5.5) to make your subscribers happier by avoiding any checks. 

Figure 5.4 The logic that decides how to handle events from the merged streams is pushed down to the observer. The observer code at this point could use `if/else` or `switch` blocks based on type.

Figure 5.5 Applying an upstream transformation to both streams to normalize the data and facilitate the observer code

Listing 5.2 shows how you can do this with our mouse clicks and touch events example. As you can imagine, the touch interface doesn't exactly correspond to that of the mouse interface, so to make them work together, you need to normalize the events so that you can reuse the same functions to handle the merged stream. You do this in the next listing by creating conformant streams or streams that emit data with similar structure.

Listing 5.2 Normalizing upstream event data merges the streams

```js
const conformantMouseUp$ = mouseUp$.map(event => ({ //*1
  left: event.clientX, 
  top: event.clientY
}));

const conformantTouchEnd$ = touchEnd$.map(event => ({ //*1
  left: event.changedTouches[0].clientX, 
  top: event.changedTouches[0].clientY, 
}));  

Rx.Observable.merge(conformantMouseUp$, conformantTouchEnd$) //*2
   .subscribe(obj => 
              console.log(`Left: ${obj.left}, Top: ${obj.top}`));
```

1. Converts each type upstream before it's merged into the final stream
2. Merges the converted streams; the observer logic stays the same

It may seem more verbose to create separate streams, but as the complexity of the application grows, updating and managing a switch statement will become a tedious process. By requiring that the source observables be conformant with the expected output interface, you can more easily expand this and continue adding more logic as needed, in case you want to include streams generated from, say, the `pointerup` event of a pointer device, like a pen (https://w3c.github.io/pointerevents). The pointer event commands may have a completely different interface from either the mouse click or the touch events, but you can still use them to control your application by forcing the incoming stream to conform to the interface expected by the subscribe block. 

One important point about `merge` that could be confusing to an RxJS beginner is that it will emit all the data that's immediately present in memory from any merged observables. The interleaving happens when events arrive asynchronously, like with `interval()` and mouse movements, but when the data is synchronously loaded, it will emit one entire stream before emitting the next. Consider this code: 

```js
const source1$ = Rx.Observable.of(1, 2, 3);
const source2$ = Rx.Observable.of('a', 'b', 'c');
Rx.Observable.merge(source1$, source2$).subscribe(console.log);
```

From what you just learned, you might think `merge()` alternates between numbers and letters, but it iterates through all numbers first and then all letters. This is because the data is synchronously available to emit. The same would happen whether you were passing scalar values one at a time, whole arrays, or generators. 

Whether data is synchronous or asynchronous, if your goal is to preserve the order of events in the combined streams, then you need to use concatenation.

### 5.1.2 Preserve order of events by concatenating streams

The RxJS `merge()` operator uses a naïve strategy of outputting all the observable data in the order in which events are received from the source streams. But in other scenarios, you might be more interested in preserving the order of the entire observable sequences when you join them instead of interleaving them. That is, given two observables, you might want to receive all the events from `source1$` and then all the events from `source2$`. This is useful in cases when you'd like to give priority to one type of event versus another. We refer to this type of operation as a *concatenation* of the two streams.

Just as you can concatenate two strings or two arrays, you can also concatenate two streams that will generate a brand-new observable made from the events of both constituent observables—similar to a set union operation. The signature is almost identical to that of the `merge` operator:

```js
const source$ = Rx.Observable.concat(...streams)
```

The behavior of this new observable operator is shown in figure 5.6.

Figure 5.6 A marble diagram of the `concat()` operator, which waits for the first observable to complete before subscribing to the next one. In concatenation, the data from both observables is appended rather than interleaved.

It's important to note that a merge differs from a concatenation on one key behavior: whereas the `merge()` operator will allow you to immediately subscribe to all of the source observables, `concat()` will subscribe to only one observable at a time. Although it continues to manage the subscriptions to each of the underlying streams, it will hold only a single subscription at a time and process that before the next one in order. You can see this behavior clearly with this simple example:

```js
const source1$ = Rx.Observable.range(1, 3).delay(3000); //*1
const source2$ = Rx.Observable.of('a', 'b', 'c');       //*2
const result = Rx.Observable.concat(source1$, source2$); 
result.subscribe(console.log);
```

1. First stream is delayed by 3 seconds, which means it will start emitting after 3 seconds
2. Second stream is set to three letters immediately

Intuitively, because the first stream is delayed by 3 seconds, you'd expect to see the letters a, b, and c emitted first. But because `concat()` keeps the order, it will complete the first stream before appending the next stream. So you'd see the numbers 1, 2, and 3 before the letters, as shown in figure 5.7.

```javascript
scheduler.run(({ cold, hot, expectObservable, expectSubscriptions, flush }) => {
    const s1 = '-1-2-3-|';
    const s2 = '-a-b-c-|';

    const ex = '3000ms -1-2-3--a-b-c-|'
    
    const source1$ = delay(3000)(cold(s1))
    const source2$ = cold(s2)
    const result = concat(source1$, source2$);
    
    expectObservable(result).toBe(ex);
});
```

Figure 5.7 Marble diagram showing the interaction of sources with delay. Regardless of the delay offsetting the entire observable sequence, the order of the events in a stream is preserved using `concat`.

Consequently, if you were to take the example of merging the touch events and the mouse events, for instance, changing from `merge()` to `concat()` would result in very different (possibly undesired) behavior, as figure 5.8 shows.

Figure 5.8 The concatenation of an infinite stream with any other stream will only ever emit an event from the first stream.

Keep in mind that `concat()` begins emitting data from a second observable once the first one completes (fires the `complete()` observer method). But this actually never occurs, because all DOM events (including touch) are essentially infinite streams. In other words, you'll only process observables from `mouseUp$` under normal circumstances. In order to see later concatenated observables, you'd need to terminate or cancel `mouseUp$` gracefully at some point in the pipeline. For instance, you could use `take()`, which you learned about in chapter 3, to have `mouseUp$` complete (made finite) after a set number of events were emitted. Say you take only the first 50 events from the mouse interface:

```js
Rx.Observable.concat(mouseUp$.take(50), touchEnd$)
    .subscribe(event => console.log(event.type)); 
```

Voila! Now you'll see touch events after you've received the first 50 mouse events. In the case of the mouse and touch streams, the concatenation operation works a little bit differently than the merge mechanism. This has to do with a basic distinction between two types of observables known as *hot* and *cold*, which we'll discuss in chapter 8. In this case, when you do begin receiving touch events, they're only for events that occurred after the end of the mouse events and not from the beginning offset to the end. Essentially, this use of `concat()` is equivalent to a nested subscription of the following form:

```js
mouseUp$
    .take(50)
    .subscribe(
        function next(event) {
            console.log(event.type);//*1
        },
        function error(e) {
            console.log(e);
        },
        function complete() {
            touchEnd$.subscribe(  //*2
                event => console.log(event.type)
            );
        }
    ); 
```

1. Handles the first 50 mouse events
2. Starts the touch stream when mouse events finish. Any touch events that occurred prior to the first 50 mouse events are missed.

> **BEST PRACTICE** We show this sample code merely to illustrate the behavior of the `concat()` operator in this example. We're not promoting that you nest subscriptions this way, which is tempting for new RxJS users. Nesting subscriptions breaks the downstream flow of data from one observable to the next, which is an antipattern in the RxJS model. Also, not only would you be duplicating your efforts by having to write the same observer code in two difference places, but you'd hamper RxJS's optimizations in the pipeline chain by impeding its ability to reuse internal data structures and lazy evaluation.

A diagram of this would look like figure 5.9.

Figure 5.9 Because we've made the `mouseUp$` stream finite, subscribers will see data emitted from `touchEnd$` after the first 50 `mouseUp$` events.

As you can see, the type of strategy employed makes a huge difference in how a downstream observable will behave. It's therefore important to understand that when you combine streams, there's a difference between when a stream is created and when a subscriber subscribes to it. You can use `merge()` when you want to receive the latest event from any observable as it's emitted, whereas `concat()` is useful when you wish to preserve the absolute ordering between observables. 

As we mentioned briefly, there will be cases when you'll need to cancel one of the observables and receive data from only another one. Let's look at another combinator, `switch()`.

### 5.1.3 Switch to the latest observable data

Both `merge` and `concat` propagate the input stream data forward into the pipeline. But suppose you wanted a different behavior, such as cancelling the first sequence when a new one begins emitting. Let's study this simple example:

```js
Rx.Observable.fromEvent(document, 'click') //*1
   .map(click => Rx.Observable.range(1, 3))//*2
   .switch()                               //*3
   .subscribe(console.log);
```

1. Listens for any clicks on the page
2. Maps another observable to the source observable
3. Uses `switch` to begin emitting data from the projected observable

Running this code logs the numbers 1, 2, 3 to the console after the mouse is clicked. In essence, when the click event occurs, this event is cancelled and replaced by another observable with numbers 1 through 3. In other words, subscribers never see the click event come in—a switch happened.

`switch()` occurs only as an instance operator, and it's one of the hardest ones to understand because it carries a bit of logic of its own. As you can see from the previous code, `switch()` takes another observable that has been mapped to the source observable and fuses it into the source observable. The caveat is that each time the source observable emits, `switch()` immediately unsubscribes from it and begins emitting events from the latest observable that was mapped. To showcase the difference, consider what the same code would look like using `merge` instead:

```js
Rx.Observable.fromEvent(document, 'click') 
   .merge(Rx.Observable.range(1, 3)) 
   .subscribe(console.log);
```

In this case, because the source observable (click events) is not cancelled, observers will receive click events mixed with numbers from 1 through 3. Hence, this sort of behavior can be useful when one stream (a button click, for instance) is used to initiate another stream. At this point, there's no interest in listening for the original stream's data (the button clicks). So `switch()` is also a suitable operator for our suggested search stream:

```js
const search$ = Rx.Observable.fromEvent(inputText, 'keyup')
    .pluck('target', 'value')
    .debounceTime(1000)
    .do(query => console.log(`Querying for ${query}...`))
    .map(query =>
        sendRequest(testData, query))
    .switch()    //*1
    .subscribe();
```

1. Switch operator causes the observable stream to switch to the projected observable

This is because as soon as the `keyup` event fires, there's no need to push that event onto the observers; instead, it's best to switch to the search results streams and push that data downstream to any subscribers, which is what gets displayed as search results. As you can see in the code, before calling `switch()`, you *mapped* (or *projected*) an observable to the source. This terminology is important, and you'll see us repeat it quite a bit in this chapter. At this point, `switch()` will cancel the previous subscription when the new one comes in. In this way, the downstream observable is always guaranteed to have the latest data by removing operations that have become stale or are no longer of interest. You can visualize this interaction in figure 5.10.

Figure 5.10 A marble diagram of the `switch()` operator, which allows only the most recently received observable to run

In using the latest value, you guarantee that no cycles will be wasted on data that will be overridden immediately when new data arrives, and you don't have to guard against that same data coming back out of order. Also, the observers don't need to worry about handling events from key presses because those are not of interest and have been suppressed by `switch()`. Because network requests can be processed and returned out of order, in certain scenarios you could see earlier requests arrive after later ones. If those requests were not excluded from the final stream, then you could see old data overwriting newer data, which is obviously not the desired behavior.

> **DISCLAIMER** If `switch()` is somewhat confusing at this point, there's a reason for this. Internally, this operator is addressing a fundamental need in functional programs, which is to flatten nested context types into a single one. In this case, the observable that's mapped to the source internally creates a nested observable object. You'll learn about this in the next section in the form of a single `switchMap()`, and then everything will be clear. 

These three operators, `merge()`, `concat()`, and `switch()`, grouped observable data in a flat manner, each with a different strategy. In other words, you're receiving events from one source or another and processing them accordingly. All these operators have higher-order observable equivalents, which handle nested observables. 

> **DEFINITION** The term *higher-order observable* originates from the notion of a higher-order function, which can receive other functions as parameters. In this case, we mean observables that receive other observables as arguments.

At this point in the book, you'll make the leap from a novice RxJS developer to a practitioner, and the techniques you're about to learn will drastically shape the way in which you approach reactive and functional programs. We began with the idea that all types of data are treated equally when wrapped with an observable; this includes the observable itself.

## 5.2 Unwinding nested observables: the case of mergeMap

The previous section detailed how streams can have their outputs combined simultaneously. In each case, the input consisted of several streams funneled into a single output stream. Depending on the strategy of combination, you can extract different behaviors from observables, as you saw in the previous section. But there are also cases that occur frequently where an observable emits other observables of its own, a situation we call *nesting the observable*, as shown in figure 5.11.

Figure 5.11 A nested observable structure that occurs when subsequent observables are created within the one observable's pipeline.

Nesting observables is useful when certain actions result in or initiate subsequent asynchronous operations whose results need to be brought back into the source observable. Recall that when you map a function to an observable, the result of this function (internally passed through to `subscriber.next()`) gets wrapped into another observable and propagated through. But what happens when the function you're trying to map also returns an observable? In other words, instead of mapping a function that returns some scalar value as you've been doing all along, the mapped function returns another observable—known as *projecting an observable onto another*, or an *observable of observables*. This is by no means a flaw in the design of RxJS; it's an expected and desired behavior. 

This situation arises frequently in the world of FP because the protocol of `map()` is that it's a structure-preserving function (for example, `Array.map()` also returns a new array). This is the contractual obligation of the implementing data type, in this case the observable, so that it retains its functor-like behavior. ~~Again, we don't cover functors in this book because they're a more advanced functional programming topic. Suffice it to say that RxJS has already implemented this for you because the `Rx.Observable` type, as you might have guessed, behaves very much like a functor.~~

Let's see how this can manifest itself in a real-world scenario. In chapter 4, we implemented a simple search box that streamed data from a small, static dataset. In that version, we used `map()` to transform a stream carrying a keyword entered by the user into an array of search results matching the term. Here are the relevant parts of it:

```js
const search$ = Rx.Observable.fromEvent(inputText, 'keyup')
  ...
  .map(query => sendRequest(testData, query)))
  .subscribe()
```

But consider what would have happened if `sendRequest()` had returned an observable object, as it would have if it was actually invoking an AJAX call. As you know, all operators create new observable instances; as a result, mapping this function will produce values of this type:

```typescript
Observable<Observable<Array>> 
```

Now subscribers need to deal with `Observable<Array>` directly instead of the data that's contained within it. This is not the desired behavior. 

The `search$` stream is now essentially a nested observable, as shown in figure 5.12. But observers shouldn't be reacting to a layer of wrapped observable values; this is unnecessary exposure or lack of proper encapsulation. They should always receive the underlying data that resulted from applying all of your business rules. So you need to somehow flatten or unwind these layers into a single one.

Figure 5.12 Mapping an observable-returning function to a source observable yields a nested observable type. `switch` can be used to flatten this structure back to a single observable.

You can do this in two ways: either you can perform a nested subscribe, wherein you subscribe to the inner observable in the parent's subscription block (bad idea for the same reasons discussed earlier), or you can merge the streams such that `search$` actually appears as a simple stream of search results to any subscriber (better idea). To get there, you need to learn and master the `mergeMap()` operator ~~(previously known as `flatMap()` in RxJS 4)~~.

Unlike `merge()` and `concat()`, `mergeMap()` and others you'll learn about shortly have additional logic under the hood to compress the inner observable back into a single observable structure (you'll learn what this means in a bit):

```js
const search$ = Rx.Observable.fromEvent(inputText, 'keyup')
  ...
  .mergeMap(query => sendRequest(testData, query)))        //*1
  .subscribe(...)
search$; //-> Observable<Array>                            //*2
```

1. Merges the outputs and switches to the observable values emitted from the query results
2. Subscribers of this stream will deal only with values of type T.

The same would hold if you wrapped a function that returns a non-observable value directly as part of the mapping function, like so: 

```js
const search$ = Rx.Observable.fromEvent(inputText, 'keyup')
  ...
  .mergeMap(query => Rx.Observable.from(queryResults(query))); 
```

Both of these cases occur often. This makes this stream much more useful in that you can now reason about the keystrokes as though they directly mapped to your search results, without worrying about the dependency between the asynchronous calls. Now that you understand `mergeMap()`, let's collect all the pieces and come up with the full solution to the search program that queries real data from Wikipedia. We'll also introduce a few other helpful operators along the way. Because it involves making an AJAX call, we'll use RxJS's version of `ajax()`, as well as `appendResults()`, `clearResults()`, and `notEmpty()` from the initial search code from chapter 4 (listing 4.12). The next listing combines most of the concepts you've learned up to now, including debouncing, into a single program.

Listing 5.3 Reactive search solution

```js
const searchBox = document.querySelector('#search'); //-> <input>
const results = document.querySelector('#results');  //-> <ul>
const count = document.querySelector('#count');      //-> <label>

const URL = 'https://en.wikipedia.org/w/api.php?action=query&format=json&list=search&utf8=1&srsearch='; //*1

const search$ = Rx.Observable.fromEvent(searchBox, 'keyup')
    .pluck('target', 'value')
    .debounceTime(500)
    .filter(notEmpty)
    .do(term => console.log(`Searching with term ${term}`))
    .map(query => URL + query)
    .mergeMap(query =>
        Rx.Observable.ajax(query)                 //*2
            .pluck('response', 'query', 'search')
            .defaultIfEmpty([]))                  //*3
    .map(R.map(R.prop('title')))             //*4
    .do(arr => count.innerHTML = `${arr.length} results`)
    .subscribe(arr => {
        clearResults(results);
        appendResults(arr, results);
    });
```

1. Wikipedia's API URL
2. Mapping an observable-returning function and flattening it (or merging it) into the source observable
3. If the result of the AJAX call happens to be an empty object, converts it to an empty array by default
4. Extracts all title properties of the resulting response array

---

##### ~~Warning: Working with CORS~~

~~Throughout the book, we'll make use of several external APIs; if you plan to copy and paste these samples directly into your browser, make sure that you have CORS protection disabled or a plugin for your browser active. If you don't want to worry about these issues, please use the code in the sample repository (https://github.com/RxJSInAction/ rxjs-in-action), which uses a simple proxy method to bypass CORS issues.~~ 

---

Figure 5.13 shows how running this program looks with a little CSS.

Figure 5.13 Reactive search program querying for "reactive programming"

You just implemented your first sample program, and despite the added complexity, you still hold true to all the principles of immutability and side effect-free coding. You'll continue to tackle problems of different types as you get comfortable with the RxJS operators. This will prepare you for chapter 10, which will show you a complete end-to-end application that integrates RxJS into React and Redux, to create a fully reactive solution.

---

##### Where does the notion of flattening data come from?

The notion of flattening data comes from the world of arrays when you need to reduce the levels of a multidimensional array. Here's an example: 

```js
[[0, 1], [2, 3], [4, 5]].flatten(); //-> [0, 1, 2, 3, 4, 5]
```

JavaScript doesn't actually have an array-flatten method, but you can easily use reduce with subsequent array concatenation to achieve the same effect: 

```js
[[0, 1], [2, 3], [4, 5]].reduce(function(a, b) {
  return a.concat(b);
}, []); //-> [0, 1, 2, 3, 4, 5]
```

As you can see, flattening is achieved by repeated concatenation at each nested level of the multi-dimensional array. For this reason, `flatten()` also goes by the name `concatAll()`. 

Instead of rolling your own, use a functional library like Ramda.js to achieve this: 

```js
R.flatten([[0, 1], [2, 3], [4, 5]]); //-> [0, 1, 2, 3, 4, 5]
```

---

If you come from an OO background, you might not be accustomed to flattening data structures as a core operation. This is because observables manage and control the data that flows through them—this idea is known as *containerizing data*. There are various benefits to doing this, some of which you've already been exposed to, such as enforcing immutability and side effect-free code, data-handling abstractions to support many event types seamlessly, and using higher-order functions to declaratively chain operators fluently. You want to have all these benefits regardless of how complex or deep your observable graph is. 

Working with nested observables is analogous to inserting an array into another array and expecting `map()` and `filter()` to work with just the data inside them. Now, instead of you having to deal with the nested structure explicitly, you allow operators to take care of this for you so that you can reason about your code better. Graphically, the notion of flattening observables means extracting the value from a nested observable and consolidating the nested structure so that the user sees only one level. This process is shown in figure 5.14.

Figure 5.14 The process of mapping a function onto an observable, resulting in a nested observable, and then flattening to extract its value. You do this so that subscribers deal with only the processed data. Here, `flatten` can be any one of the RxJS operators, such as `switch()` or `merge()`.

We'll show an example that uses simple arrays so that you can see more clearly the difference between flattening an array and leaving it as is. Consider some simple code that fills in numbers in an array:

```js
const source = [1, 2, 3];
source.map(value => 
           Array(value).fill(value));//*1
//-> [[1],[2,2],[3,3,3]]

source.map(value => Array(value).fill(value)).concatAll();//*2
//-> [1,2,2,3,3,3]
```

1. Mapping a function that returns an array onto an array results in a multi-dimensional array.
2. After a call to `concatAll` (or `flatten`), the resulting array is single dimensional and much easier to work with.

Wrapping your head around the idea of observables propagating other observables may take some time, but we guarantee it will come to you naturally by the end of the book. This software pattern is at the root of the FP paradigm and implemented data types known as monads. In simple terms, a monad exposes an interface with three simple requirements that observables satisfy: a unit function to lift values into the monadic context (`of()`, `from()`, and the like), a mapping function (`map()`), and a map-with-flatten function (`flatMap()` or `mergeMap()`).

---

##### ~~More information about monads~~

~~We don't cover monads in this book, but we encourage you to learn more about them because they're a central pillar of FP. If you're interested in learning more, please read chapter 5 of Functional Programming in JavaScript (Manning, 2016) by Luis Atencio.~~

---

In the real world, nested streams occur so frequently that for most of the combinatorial operators there exists a set of joint operators so that you don't have to use two every time you need to use switching, merging, and concatenating. We introduced `mergeMap()` in this section, and now we're going to use it to tackle more-complex problems. 

## 5.3 Mastering asynchronous streams

Recall that for our initial implementation of our stock ticker widget in chapter 4 we used a simple random-number generator to model the variability of our hypothetical company ABC and its stock price in the market. In this chapter, we'll integrate with a real stock service, such as Yahoo Finance, to fetch data for a given symbol. The task at hand is to use the Yahoo Finance API to query for a company's real stock price in real time so that the user sees updates pushed to the application reflecting changes in the stock's value. This simple-to-use API responds with CSV. To start off, let's use one symbol, Facebook (FB). This time, instead of using RxJS's `ajax()`, we'll show you how to plug in promise-based AJAX calls. The process is roughly outlined in figure 5.15.

Figure 5.15 Using a promise call to the Yahoo web service for Facebook's stock data

You'll approach this stream by tackling its individual components. The first one you'll need is a stream that uses a promise to query stock data via AJAX, as shown in the following listing.

Listing 5.4 The request quote stream

```js
const csv = str => str.split(/,\s*/); //*1
const webservice = 'http://download.finance.yahoo.com/d/quotes.csv?s=$symbol&f=sa&e=.csv'; //*2
const requestQuote$ = symbol =>
    Rx.Observable.fromPromise(
        ajax(webservice.replace(/\$symbol/, symbol)))//*3
        .map(response => response.replace(/"/g, ''))
        .map(csv);//*4
```

1. Helper function that creates an array from a CSV string
2. Yahoo Finance REST API link and requesting output format to be CSV
3. Uses the promise based `ajax()` to query the service
4. Cleans up and parses the CSV output

The CSV response string emitted by the Yahoo API needs to be cleaned up and parsed into a string. Here, you create a function that returns an observable capable of fetching data from a remote service and publishing the result. You'll combine this with a two-second interval to poll and provide the real-time feed:

```js
const twoSecond$ = Rx.Observable.interval(2000);
```

Now, you have two isolated streams that need to be combined, or mapped one to the other. You can use `mergeMap()` to accomplish this. You can create a function that takes any stock symbol and creates a stream that requests stock quote data every 2 seconds. You'll call this resulting observable function `fetchDataInterval$`, as shown here.

Listing 5.5 Mapping one stream into another

```js
const fetchDataInterval$ = symbol => twoSecond$
.mergeMap(() => requestQuote$(symbol)); 
```

All that’s left to do now is call this function with any stock symbol, in this case FB:

```js
fetchDataInterval$('FB')
  .subscribe(([symbol, price])=> 
             console.log(`${symbol}, ${price}`)
            ); 
```

Notice how declarative, succinct, and easy to read this code is. At a glance, it describes the process of taking a two-second poll and mapping a function to fetch stock data— as simple as that. This works well for a single item, but when scaling out to multiple items, it's common to lift the collection of stock symbols to search for into an observable context and then map other operations to it. The next listing shows how to use `mergeMap()` again to fetch quotes for companies: Facebook (FB), Citrix Systems (CTXS), and Apple Inc. (AAPL).

Listing 5.6 Updating multiple stock symbols

```js
const symbols$ = Rx.Observable.of('FB', 'CTXS', 'AAPL');
const ticks$ = symbols$.mergeMap(fetchDataInterval$);
ticks$.subscribe(
    ([symbol, price]) => {
        let id = 'row-' + symbol.toLowerCase();
        let row = document.querySelector(`#${id}`);
        if (!row) {
            addRow(id, symbol, price);//*1
        }
        else {
            updateRow(row, symbol, price);//*1
        }
    },
    error => console.log(error.message));
```

1. For brevity, we won't show the body of these functions that simply manipulate HTML to add or update rows. ~~You can visit the code repository to get all the details.~~

Essentially, the semantic meaning of `mergeMap()` is to transform the nature of the stream (map) by merging a projected observable. This operator is incredibly useful because now you can use it to orchestrate a complex business process containing multiple observable levels. The program flow looks like figure 5.16. 

Figure 5.16 Steps to implement the stock ticker widget with multiple symbols and a refresh interval of 2 seconds. Every step involves the use of nested observables that are merged or concatenated accordingly as the information gets transformed into different types of streams. At the end, subscribers will see only the scalar values representing the stock symbols and their respective prices.

 This program is simple and high level, but it accomplishes a lot:

1. You start by lifting the stock symbols involved in your component into a stream so that you can begin to fetch their quote data. This technique of lifting or wrapping a scalar value into an observable is beneficial because you can initiate asynchronous processes involving these values. Also, you unlock the power of RxJS when involving blocks of code related to these values. 
2. You map an interval (every 2 seconds) to this observable, so that you cycle through the stock symbols every minute. This gives it the appearance of real time.
3. At each interval, you execute AJAX calls for each stock symbol against the Yahoo web service.
4. Finally, you map functions that strip out unnecessary company data and leave just the stock symbol and price, before emitting it to subscribers.

The result is that the subscribers see the data pairs shown in figure 5.17 emitted every 2 seconds.

Figure 5.17 A Stocks table that updates with real stock symbols every 2 seconds

There's room for improvement here, because in cases when there's not a whole lot of fluctuation in a company's stock price, you don't want to bother updating the DOM unnecessarily. One optimization you can do is to allow the stream to flow down to the observer only when a change is detected, by means of a filtering operator called `distinctUntilChanged()`. First, we'll show you how this operator works, and then we'll include it in our code. Here's an example of using it with a simple input: 

```js
Rx.Observable.of('a', 'b', 'c', 'a', 'a', 'b', 'b')
  .distinctUntilChanged()
  .subscribe(console.log);
```

`distinctUntilChanged()` belongs in the category of filtering operators and emits a sequence of distinct, contiguous elements. So the fifth element, `'a'`, is skipped because it's the same as the previous one, and the same for the last, `'b'`. You can see this mechanism in action in figure 5.18.

Figure 5.18 `distinctUntilChanged()` returns an observable that emits all items emitted by the source observable that are distinct when compared to the previous item.

This is perfect for the task at hand. Adding this feature to your stream involves using it with a key selector function, a function that instructs the operator what to use as the property to compare: 

```js
const fetchDataInterval$ = symbol => twoSecond$
.mergeMap(() => requestQuote$(symbol))
.distinctUntilChanged(([symbol, price]) => price); //*1
```

1. Performs a distinct comparison based on price, so that the DOM will get updated only when the price changes

Now, you'll update the DOM strictly on a price change, which is much more optimal than doing it naïvely everything 2 seconds.

You were able to combine observable flows and solve this problem without creating a single external variable, conditional, or for loop. Because this task involves the combination of multiple streams—iterating through the symbols, a timed interval, and remote HTTP requests with promises—you had to unwind nested streams using a combination of a couple of nested `mergeMap()` operators to convert the intricate business logic into a more flattened and linear sequence of data that observers subscribe to and easily consume. As we mentioned previously, it's much more efficient and fluent to do it this way, rather than to subscribe to each nested stream at each step. Using RxJS operators to handle this, as well as all the business logic, is always preferred over sending raw nested observables to any downstream observers.

As you can see from this example, RxJS's combinatorial operators like `mergeMap()` are about more than simply reducing the number of subscribe blocks you have to write and the number of subscriptions to keep. They also serve as an important way to craft more-intricate flows to support complex business logic. By leveraging nested streams, you can think of each block within a nest as a mini application (that can be partitioned out into its own stream, as you learned previously). This is a clear sign of modularity in your code and something that functional programs exhibit extremely well. By breaking down problems into individual, independent blocks, you can create code that's more modular and composable. Now that we've covered merging complex observable flows, let's look at a more complex example.

Aside from `mergeMap()`, other operators work in a similar manner but with a slightly different flavor driven by the behavior of the composed function, whether it be `switch()`, `merge()`, or `concat()`, depending on what you're trying to accomplish. Table 5.1 describes the three joint operators we'll use in this book.

##### Table 5.1 A breakdown of the three most used RxJS joint operators

| Split operator      | Joint operator | Description                                                  |
| ------------------- | -------------- | ------------------------------------------------------------ |
| map()...merge()     | mergeMap()     | Projects an observable-returning function to each item value in the source observable and flattens the output observable. ~~You might know this operator by `flatMap()`, as used in previous versions of RxJS.~~ |
| map()...concatAll() | concatMap()    | Similar to `mergeMap()` with the merging happening serially; in other words, each observable waits for the previous one to complete. Why not `map()...concat()`? We'll explain this disparity shortly. |
| map()...switch()    | switchMap()    | Similar to `mergeMap()` as well but emitting only values from the recently projected observable. In other words, it switches to the projected observable emitting its most recent values, cancelling any previous inner observables. ~~You might know this operator by `flatMapLatest()`, as used in previous versions of RxJS.~~ We'll come back to this operator in chapter 6. |

Now that you've mastered handling nested observables, let's move on to other types of higher-order observable combinations, continuing with `concatMap()`.

## 5.4 Drag and drop with concatMap

We've looked at two ways of composing streams such that they produce a single output stream. In each case, the resulting stream appears identical to an observer. Both 

```js
Rx.Observable.merge(source1$, source2$) //*1
```

1. Observers see data emitted from either observable.

and

```js
source1$.mergeMap(() => source2$) //*1
```

1. Observers see only data from `source2$`.

result in observables that return a type compatible with either source. As we mentioned before, this is entirely up to you because JavaScript won't enforce that observables wrap values of the same type, as statically typed languages will. 

But the behavior of these two approaches is slightly different. In the former case, you create a set of simultaneous streams; this means that both streams are subscribed to at the same time and the resulting observable can output from either observable— both are active simultaneously.

Sequential streams, on the other hand, are streams in which the output of one stream generates a new one. In the second case, the observer won't receive any events from the first stream; an observer will only see the results of the observable projected by `mergeMap()`. By combining sequential and simultaneous streams, you can make fairly complex logic relatively easily.

One canonical example of a sequential stream is drag and drop, present in most modern web content management systems and customizable dashboards. Implementing this behavior in vanilla JavaScript is fairly difficult to get right because it involves keeping track of fast-changing states with multiple targets and interaction rules. Using streams, you can implement this in a somewhat streamlined manner. 

To implement drag and drop, you need to first identify the three types of mouse events that are necessary for basic drag and drop. First, you need to know when the mouse button is clicked, because this indicates that the drag has started, and then the mouse-button-up event to determine when it has stopped, because this indicates a drop. In order to track the drag, you need to capture the mouse-move event as well. You'll need only those three events (`mouseup`, `mousemove`, and `mousedown`); each, of course, modeled as a stream. 

A drag starts when the user clicks and stops only when the mouse button is released or until the `mouseup` event fires. As the mouse is moved with the button down, the object is being dragged; it also moves in a fluent manner, so you'll anticipate performing a side effect by manipulating the DOM element. When the button is released, you drop that object into the coordinates of the location of the mouse on the screen—thus terminating the `mousemove` event. As before, the gist is to combine these streams, representing the three mouse events emitting data. 

First, you'll create streams from the different types of events you're interested in, as shown in the following listing.

Listing 5.7 Streams needed to implement drag and drop with a mouse

```js
const panel = document.querySelector('#dragTarget');               //*1
const mouseDown$ = Rx.Observable.fromEvent(panel, 'mousedown');    //*2
const mouseUp$ = Rx.Observable.fromEvent(document, 'mouseup');     //*3
const mouseMove$ = Rx.Observable.fromEvent(document, 'mousemove'); //*4
```

1. A reference to the panel or target you want to drag (My Stocks widget)
2. Observable for mousedown events on the target
3. Observable for mouseup events over the entire page
4. Observable for mousemove events over the entire page

After you've established the streams that will be used to control the drag, you can build logic around it. In the next step, you need to implement the logic, first to handle detecting a click on a drag target that will initiate the drag event. Then, you need to have the element follow the mouse around the screen until it's released and dropped somewhere. You've learned in this chapter how you can use the RxJS joint operators to combine and flatten multiple nested streams so that subscribers see a simple representation of the data flowing through the stream. So you need an order-preserving operator (the mouse is pressed, then dragged, and finally released), just like `concat()`, but you also need to be able to flatten the observable that's pushed through it. Care to take a guess? Yes, this is the job of `concatMap()`. This operator works just like `mergeMap()` but performs the additional logic of retaining the order of the observable sequences, instead of interleaving the events.

The logic for handling the drag is made up of a sequence of streams that emit mouse events together, as shown in the next listing.

Listing 5.8 Drag-and-drop stream logic

```js
const drag$ = mouseDown$.concatMap(() => mouseMove$.takeUntil(mouseUp$));
drag$.subscribe(event => {
  panel.style.left = event.clientX + 'px';
  panel.style.top = event.clientY + 'px'; 
});
```

Sorry, were you expecting more? This is all that's required to drag the stock widget around the page. If you think about it—likely, you've implemented drag and drop before—you're probably aware that using plain JavaScript would take much more code, using many more variables to store some transient state. This code is not only shorter, but it also has a higher level of abstraction, because all side effects were pushed elegantly onto the observer. 

Here, we've introduced another variation of the `take()` operator called `takeUntil()`. The name is straightforward; this operator also belongs to the filtering category and allows the source observable to emit values until the provided notifier observable emits a value. The notion of *notifier observables* occurs frequently in RxJS, to be used as signals to either start or stop some kind of interaction; in this case, to take any `mousemove` concatenated with the `mousedown` events until a `mouseup` event is fired. This is the gist behind dragging.

Here's a simple use of `takeUntil()` so that you can fully appreciate how it works. This example starts a simple one-second interval, which will print values to the console until the user clicks a button. This could be useful to implement a site inactivity feature, for instance:

```js
const interval$ = Rx.Observable.interval(1000);
const clicks$ = Rx.Observable.fromEvent(document, 'click');
interval$.takeUntil(clicks$) //*1
    .subscribe(
        x => console.log(x),
        err => console.log(`Error: ${err}`),
        () => console.log('OK, user is back!'));
```

1. As soon as a click event is emitted, the interval stream is cancelled.

The other benefit of RxJS's unified computing model is that if you were implementing drag through a touch interface, it would just be a matter of changing the event names in the stream declaration to `touchmove`, `touchstart`, and `touchend`, respectively. The business logic stays the same, and the code would work exactly the same!

There's a small caveat here. From our earlier discussions, you might be led to believe that calling `map()...concat()` would work in a fashion similar to the split operator `concatMap()`. You might intuitively think that this code would work exactly the same way as listing 5.7:

```js
const drag$ = mouseDown$.map(() =>   
    mouseMove$.takeUntil(mouseUp$))
    .concat(); //*1
```

1. Using the instance operator `concat()` in place of `concatMap()`

Unfortunately, it doesn't, because there's no mechanism here to flatten the observable type running through the stream. Recall that the `concat()` instance method takes a number of observables and concatenates them in order. It's not designed to work with an observable of observable type—a higher-order observable. For this, you need to use a variation of `concat()` that works with nested observables and also flattens them, called `concatAll()` (just as you implemented with arrays before). So the RxJS nomenclature here is a little inconsistent, because `concatMap()` is really `map()...concatAll()`.

Now this code works just like listing 5.7:

```js
const drag$ = mouseDown$
   .map(() => mouseMove$.takeUntil(mouseUp$))
   .concatAll();
```

Figure 5.19 is a visual of how `concatAll()` works. 

Figure 5.19 Workings of `concatAll()` combining three mouse events. This operator not only preserves order but can also flatten a sequence of nested observables. Also, the use of `takeUntil()` causes `mouseMove$` to cancel as soon as `mouseUp$` emits a value.

In the code repository, you'll find a more complete version of this example, which we simplified for the sake of highlighting the important elements. In reality, you can add any number of bells and whistles to this functionality, and for our sample application, we added some helper code for dealing with CSS. For instance, if you wanted to prevent users from accidentally dragging a widget by allowing them to confirm the drag, you could just filter the `mouseup` stream to include this confirmation: 

```js
const drag$ = mouseDown$.
  concatMap(() =>
    mouseMove$.takeUntil(
      mouseUp$.filter(() => confirm('Drop widget here?'))));
```

It's as simple as that. The examples that we've discussed in this chapter are only a small portion of the total number of use cases we could support. As we mentioned at the outset, almost all non-trivial applications will use flattened streams, so understanding them is a huge step toward understanding RxJS.

In this chapter, we entered more complex territory by combining the outputs of multiple streams. This was key for us to begin implementing more real-world tasks instead of just showcasing the different operators. Initially, we looked at static streams that merely had their outputs piped together to appear as a single stream. But as a more complex case, we examined how to nest observables within each other and then flatten the result to deliver the appropriate data to observers. We discussed how making more-complex applications, especially those that deal with UIs and other state machines, will almost always necessitate the use of nested or merged streams. Finally, we explored a couple of practical examples of flattening and merging that helped you understand the design principle behind projecting observables. 

In the next chapter, we'll continue expanding on this topic by examining how you can further coordinate observables and have them work together. We'll also explore other interesting ways that observables combine, using an operator called `combineLatest()`.

## 5.5 Summary

* You can merge the outputs of several observables into a single stream to simplify subscription logic.
* You can use different merge strategies that contain different behavior for combining streams, depending on your needs. 
* You can interleave streams with `merge()`, cancel and switch to a new projected observable with `switch()`, or preserve entire observable sequences in order by using `concat()`.
* You can use split operators to combine and flatten a series of nested observable streams.
* You can combine and project observables into a source observable using the higher-order operators such as `mergeMap()` and `concatMap()`.
* You implemented an auto-suggest search box.
* You implemented a live stock ticker with deeply nested streams.
* You implemented drag and drop using stream concatenation. 



