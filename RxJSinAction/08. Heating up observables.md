# Chapter 8. Heating up observables

This chapter covers

* The difference between hot and cold observables
* Working with WebSockets and event emitters via RxJS
* Sharing streams with multiple subscribers
* Understanding how hot and cold pertains to how the producer is created
* Sending events as unicast or multicast to one or multiple subscribers

As you know, an observable function is a lazy computation, which means the entire sequence of operators that encompass the observable declaration won't begin executing until an observer subscribes to it. But have you ever thought about what happens to all the events that occur before subscription happens? For instance, what does a mouse move observable do with all the mouse events or a WebSocket that has received a set of messages? Do these potentially very important occurrences get lost in the ether? The reality is that these active data sources won't wait for subscribers to listen to begin emitting events, and it's vital for you to know how to deal with this situation. Earlier, we briefly called out this idea that observables come in two different flavors: hot and cold. This isn't a simple topic to grasp; it's probably one of the most complex in the RxJS world, which is why we dedicate an entire chapter to it.

In this chapter, we'll take a closer look at hot and cold observables, how they differ, the benefits of each, and how you can take advantage of this in your own code. Up until now, we've had only a single subscriber to a stream, and we even looked at combining multiple streams funneled through one observer in chapter 6. Now, we take the opposite approach as we look into sharing a single observable sequence with multiple observers. For instance, we can take a simple numerical stream and share its values with multiple subscribers or take a single WebSocket message and broadcast it to multiple subscribers. This is very different from the single-stream subscriber cases we've dealt with. We'll begin by demonstrating the differences between hot and cold observables. 

## 8.1 Introducing hot and cold observables

Think about when you turn on your TV set and switch to the channel of your favorite show. If you catch the show 10 minutes after it began, will it start from the beginning? Restarting the show would mean that cable companies broadcast independent streams to every subscriber—which would be great. Instead, the same content is broadcast to all subscribers at a set time. So unless you have a recording device, which you can associate with a buffer operator, you can't replay content that aired in the past.

A TV's live stream is equivalent to a hot observable. RxJS divides observable sources into either of two categories: hot or cold. These categories determine the behavioral characteristics, not just of subscription semantics, but also of the entire lifetime of the stream. An observable's temperature also affects whether the stream and the producer are managed together or separately, which can greatly affect resource utilization (we'll get to this shortly). We classify an observable as either hot or cold based on the nature of the data source that it's listening to. Let's begin with cold observables.

### 8.1.1 Cold observables

In simple terms, a *cold observable* is one that doesn't begin emitting all of its values until an observer subscribes to it. Cold observables are typically used to wrap bounded data types such as numbers, ranges of numbers, strings, arrays, and HTTP requests, as well as unbounded types like generator functions. These resources are known as *passive* in the sense that their declaration is independent of their execution. This also means that these observables are truly lazy in their creation and execution.

Now, this isn't news to you, because that's what we've defined an observable to be all along, so what's the catch? Being cold means that each new subscription is creating a new independent stream with a new starting point for that stream. This means that subscribers will independently receive the exact same set of events always, from the beginning. Here's another way to conceptualize it: when creating a cold observable, you're actually creating a plan or recipe to be executed later—repeatedly, top to bottom. The recipe itself is just a set of instructions (operators) that tell the JavaScript runtime engine how to combine and cook the ingredients (data); cold observables begin emitting events only when you choose to start cooking.

> **PURE OBSERVABLES** Observables are pure when they abide by the FP principles of a pure function, which is immutable, side effect-free, and repeatable. We've talked about the first two principles at length in this book, and now we tag on this third quality. In order to support the desirable functional property of referential transparency, functions must be repeatable and predictable, which means that invoking a function with the same arguments always yields the same result. The same holds for cold observables when viewed simply as functions that produce (return) a set of values.

From a pure FP perspective, you can think of cold observables as behaving very much like functions. A function can be thought of as a lazy or to-be-computed value that's returned when you invoke it, only when needed (languages with lazy evaluation, like Haskell, work this way). Similarly, observable objects won't run until subscribed to, and you can use the provided observers to process their return values. You can visualize this resemblance in figure 8.1.

Figure 8.1 A cold observable can be thought of as a function that takes input—data that is to be processed—and, based on this, returns an output to the caller.

Furthermore, the declaration of a cold observable frequently begins with static operators such as `of()` or `from()`, and timing observables `interval()` and `timer()` also behave coldly. Here's a quick example: 

```js
const arr$ = Rx.Observable.from([1,2,3,4,5,6,7,8,9]);
const sub1 = arr$.subscribe(console.log);//*1
// ... moments later ... //
const sub2 = arr$.subscribe(console.log);//*2
```

1. Every subscriber gets their own independent copy of the same data no matter when the subscription happens.
2. `sub2` could have subscribed moments later, yet it still receives the entire array of elements.

With cold observables, all subscribers, no matter at what point the subscription occurred, will observe the same events. Another example is the `interval()` operator. Each time a new subscription occurs, a brand-new interval instance is created, like in figure 8.2. 

Figure 8.2 A cold observable is like an object factory, which can be used to create a family of subscriptions that will receive their own independent copy of all the events pushed through them.

`interval` acts like a factory for timers, where each timer operates on its own schedule and each subscription can, therefore, be independently created and cancelled as needed. Cold observables can be likened to a factory function that produces stream instances according to a template (its pipeline) for each subscriber to consume fully. The following listing demonstrates that you can use the same observable with two subscribers that listen for even and odd numbers only, respectively.

Listing 8.1 Interval factory 

```js
const interval$ = Rx.Observable.interval(500);
const isEven = x => x % 2 === 0;
interval$
    .filter(isEven)
    .take(5)
    .subscribe(x => { //*1
        console.log(`Even number found: ${x}`);
    });
interval$
    .filter(R.compose(R.not, isEven))
    .take(5)
    .subscribe(x => { //*1
        console.log(`Odd number found: ${x}`);
    });
```

1. Two subscriptions for the same observable, interval$

In the example in listing 8.1, the streams are independently created from one another. Each time a new subscriber subscribes, the logic in the pipeline is reexecuted from scratch. So these streams all receive the same numbers and don't affect each other's progress—they're isolated, as shown in figure 8.3.

Figure 8.3 Resubscribing to an interval observable yields two independent sequences, with the same events happening with the same frequency; hence, this is a cold observable.

> **DEFINITION** A cold observable is one that, when subscribed to, emits the entire sequence of events to any active subscribers.

Now, we're ready to heat things up and look closely at the other side of the coin: hot observables.

### 8.1.2 Hot observables

Streams don't always start when you want them to, nor can you reasonably expect that you'd always want every event from every observable. It may be the case that by delaying subscription, you deliberately avoid certain events like an implicit version of `skip()`. 

Hot observables are those that produce events regardless of the presence of subscribers—they are *active*. In the real world, hot observables are used to model events like clicks, mouse movement, touch, or any other events exposed via event emitters. This means that, unlike the cold counterpart where each subscription triggers a new stream, subscribers to hot observables tend to receive only the events that are emitted after the subscription is created, as shown in figure 8.4.

Figure 8.4 A mouse move handler generates unbounded events that can be captured as soon as the HTML document loads, but these events will be ignored until the stream has been created and the observer subscribed to it.

A hot observable continues to remain lazy in the sense that without a subscriber, the events are simply emitted and ignored. Only when an observer subscribes to the stream does the pipeline of operators begin to do its job and the data flow downstream.

This type of stream is often more intuitive to many developers because it closely mirrors behaviors they're already familiar with in `Promise`s and event emitters. 

> **PROMISES AND HTTP CALLS** Although a conventional HTTP request is cold, it isn't when a `Promise` is used to resolve it. As you'll learn in a bit, a `Promise` of any type represents a hot observable because it's not reexecuted after it's been fulfilled.

Because of the unpredictable and unrepeatable nature of the data that hot observables emit, you can reason that they're not completely pure, from a theoretical perspective. After all, reacting to an external stimulus, like a button click, can be considered a form of side effect dependent on the behavior of some other resource, like the DOM, or simply time. Nevertheless, from the point of view of the application and your code, all observables can be considered pure.

Unlike cold observables that create independent copies of the data source to emit to every subscriber, a hot observable shares the same subscription to all observers that listen to it, as shown in figure 8.5. Therefore, you can conclude that a hot observable is one that, when subscribed to, emits the ongoing sequence of events from the point of subscription and not from the beginning. 

Figure 8.5 Hot observables share the same stream of events to all subscribers. Each subscriber will start receiving events currently flowing through the stream after subscription.

Whether an observable is hot or cold is partly related to the type of source that it's wrapping. For instance, in the case of any mouse event handler, short of creating some new mechanism for handling mouse events, an observable is merely abstracting the existing `addEventListener()` call for a given emitter. As a result, the behavior of mouse event observables is contingent on the behavior of the system's handling of mouse events. You can further categorize this source as *natively hot*, because the source is determining the behavior. You can also make sources hot, programmatically, using operators as well, and we'll discuss this further in the sections to come. 

On the other hand, observables that either wrap a static data source (array) or use generated data (via a generator function) are typically cold, which means they don't begin producing values without a subscriber listening to them. This is intuitive because, like iteration, stepping through a data source requires a consumer or a client.

A key selling point for using RxJS is that it allows you to build logic independently of the type of data source you need to interact with—we called this a *unifying computing model*. A source can emit zero to thousands of events unpredictably fired at different times. Nevertheless, the abstraction provided by the observable type means that you don't have to worry about these peculiarities when building the logic inside the stream or within the context of the observable. This interface abstracts the underlying implementation out of sight and out of mind—for the most part.

> **DEFINITION** Hot observables are those that produce events regardless of the presence of subscribers.

In general, it's better to use cold observables wherever possible because they're inherently stateless. This means that each subscription is independent of every other subscription, so there's less shared state to worry about, from an internal RxJS perspective, because you know a new stream is starting on every subscription.

## 8.2 A new type of data source: WebSockets

In chapter 4, we mentioned that time ubiquitously exists in observables—in hot observables, to be exact. This disparity between when a data source begins emitting events and when a subscriber starts listening can lead to issues. Think about that TV show that you switched to midprogram. In such contexts, unless you've watched the show before tuning in, you'll miss some context or plot that was presented at the beginning. In the same vein, you can imagine a simple messaging system using a protocol like WebSockets—or any other event emitter, for that matter. In these cases, missing any messages can be critical to the proper functioning of your application. So if a subscription to a hot observable occurs after a critical message packet arrives, then those instructions might be lost. We haven't talked about using WebSockets with RxJS, so let's begin by briefly examining what they are and how RxJS can help you handle these asynchronous message flows. 

### 8.2.1 A brief look at WebSocket

Aside from binding to DOM events and AJAX calls, RxJS can just as easily bind to WebSockets. WebSocket (WS) is an asynchronous communication protocol that provides faster and more efficient lines of communication from client to server than traditional HTTP. This is useful for highly interactive applications like live chats, streaming services, or games. Like HTTP, WS runs on top of a TCP connection, but the advantage is that information can be passed back and forth while keeping the connection open (taking advantage of the browser's multiplexing capabilities and the keep-alive feature). The other benefit is that servers can send content to the browser without it explicitly requesting it. 

Figure 8.6 shows a simplified view of WS communication. It begins with a handshake, which bridges the world of HTTP to WS. At this time, details about the connection and security are discussed to pave the way for secure, efficient communication between the parties involved.

The steps taken between the client and the server, illustrated in figure 8.6, are these:

1. Establish a socket connection between parties for the initial handshake.
2. Switch or upgrade the communication protocol from regular HTTP to a socket-based protocol.
3. Send messages in both directions (known as *full duplex*).
4. Issue a disconnect, via either the server or the client. 

Figure 8.6 WebSocket communication diagram showing communication that begins with a handshake where client and server negotiate the terms of the connection. Afterward, communication flows freely until one of the parties closes.

The crucial point of this process is the initial handshake that negotiates the upgrade process. The upgrade needs to happen because WebSocket uses the same ports 80 and 443 for HTTP and HTTPS, respectively (443 is used by the WebSockets Secure protocol). Routing requests through the same ports is advantageous because firewalls are typically configured to allow this information to flow freely. At a very high level, this process happens with an initial secure request by the client and a proper response by the WebSocket-supporting server, shown in figure 8.7.

Figure 8.7 The handshake negotiation process begins with the client's GET request containing a secure key and instructions for the server to attempt to upgrade to a message-based WebSocket connection. If the server understands WebSocket, it will respond with a unique hash confirming the protocol upgrade.

From the point of view of asynchronous event messaging, you can think of WebSocket as an event emitter for client-server communication. We haven't really talked about server-side RxJS, but little changes there. You can easily use RxJS to support both sides of the coin, starting with the server.

### 8.2.2 A simple WebSocket server in Node.js

In the spirit of a JavaScript book, we'll write our server using Node.js. But you can use any other platform of your liking such as Python, PHP, Java, or any other platform with a socket API. Our server side will be a simple TCP application listening on port 1337 (chosen arbitrarily) using the Node.js WebSocket API. WebSocket negotiates with the HTTP server to be the vehicle to send and receive messages. Once the server receives a request, it will respond with the message "Hello Socket," as follows.

Listing 8.2 Simple RxJS Node.js server

```js
const Rx = require('rxjs/Rx');                      //*1
const WebSocketServer = require('websocket').server;//*2
const http = require('http');                       //*3

// ws port
const server = http.createServer();//*4
server.listen(1337);

// create the server
wsServer = new WebSocketServer({
    httpServer: server
});

Rx.Observable.fromEvent(wsServer, 'request')//*5
    .map(request => request.accept(null, request.origin))
    .subscribe(connection => {
        connection.sendUTF(JSON.stringify({ msg: 'Hello Socket' }));//*6
    });
```

1. Imports the RxJS core APIs
2. Imports the WebSocket server library
3. Imports the HTTP library
4. Instantiates an HTTP server to begin listening on port 1337
5. Reacts to the request received event
6. Sends a JSON object packet once a connection is established

You can run the server with Node.js using the CLI (for details about setting up RxJS on the server, please visit appendix A):

```
> node server.js
```

Now that your server is up and listening, you can build the client again using RxJS—one library to rule them all!

### 8.2.3 WebSocket client

Modern browsers come equipped with the WebSocket APIs, which allow for an interactive communication with a server. Using these APIs, you can send messages to a server and receive event-driven responses (server push) without you having to explicitly poll for data, which is what you would do with regular HTTP requests. The next listing shows a simple WebSocket connection using RxJS. 

Listing 8.3 WebSocket client with RxJS

```js
const websocket = new WebSocket('ws://localhost:1337');//*1

Rx.Observable.fromEvent(websocket, 'message')//*2
  .map(msg => JSON.parse(msg.data))//*3
  .pluck('msg')//*4
  .subscribe(console.log); 
```

1. Connects to port 1337 using the WebSocket (ws) protocol
2. Listens for the 'message' events used to transmit messages between client and server
3. Parses the serialized string into a JSON object
4. Reads the message

WebSockets are a form of loosely decoupled communication between two entities, in this case a browser and a server, that act as event emitters. This goes back to the idea of using a familiar computing model for everything. In essence, with RxJS, the details of setting up event listeners for WebSocket communication are completely abstracted and removed from your code. 

> **REACTIVE MANIFESTO** A good design principle of reactive systems is that they communicate using asynchronous message passing, in order to establish a boundary between loosely coupled components. This not only allows components to evolve independently but also delegates failures as messages. This means non-local components can react to errors appropriately.

Just like any event emitter, using RxJS with WebSocket creates a hot observable, which means it won't reenact all the messages emitted upon subscription but merely begin pushing any that occur thereafter. This can be tricky because subscribing to a hot observable a little too late can result in a loss of data. Consider this slight variation of listing 8.3:

```js
Rx.Observable.timer(3000) //*1
  .mergeMap(() => Rx.Observable.fromEvent(websocket, 'message')) 
  .map(msg => JSON.parse(msg.data))
  .pluck('msg')  
  .subscribe(console.log);  
```

1. Wraps the socket object after a three-second delay

In this stream, you tie WebSocket subscription to a wait time (3 seconds). You arbitrarily set this wait time in order to simulate a delay such as one caused by page initialization—essentially, you create a time dependency. Recall the WebSocket handshake diagram; as soon as the socket fires the open event, this hot observable can begin emitting events. Thus, if the socket opening occurs before the page has completed its initialization step, the observable will potentially miss any events emitted in the intervening period.

As an added complication, time dependencies aren't necessarily deterministic. In this scenario, you added a simple three-second timeout, but your page load or initialization logic could be much more complex. It could be affected by any number of variables, such as whether the application was loaded from a cache, how many resources the user's system has available for processing requests, how much network latency is present, or even how much animation is on the page, because they all can change when the page initialization occurs or when the WebSocket connects and begins sending events.

Certainly a dependency on time can be significant in an observable's behavior, to the point of breaking the nice functional quality that cold observables possess. Ideally, every subscriber to a cold observable should see the same sequence of events replayed, but this isn't always the case, because there's a big difference between resubscribing and replaying when side effects are at play.

## 8.3 The impact of side effects on a resubscribe or a replay

As you saw from the previous use case, hot observables, for the most part, follow a strict you-snooze-you-lose policy, which means you can't replay the contents of a hot observable by resubscribing to it, as you can with cold observables (there are ways of doing it, but this isn't the default behavior). Now, this doesn't mean that all cold observables behave this way, especially when you introduce side effects into your code. Before we discuss this further, you need to understand the difference between resubscribing and replaying in RxJS:

* A *replay* is about reemitting the same sequence of events to each subscriber—in effect, replaying the entire sequence. You must use caution when attempting to replay sequences because they potentially require using lots of memory (often with unbounded buffers) to store the contents of a stream that is to be reemitted at a later time. For obvious reasons, we recommend against doing this with streams like mouse clicks or any other infinite event emitter. 
* A good example of replay semantics is a `Promise`. Replaying the observable created from a `Promise` by means of a retry or simply attaching new subscribers doesn't cause the fulfilled `Promise` to invoke again but simply to return the value or the error, as the case may be. 
* A *resubscribe* re-creates the same pipeline and reexecutes the code that produces events. Although the results emitted by the producer will be implementation dependent, if your observable pipeline remains pure, then you can guarantee that all subscribers will receive the same events for the same input produced. 

### 8.3.1 Replay vs. resubscribe

The difference is subtle but important. In essence, it's about whether the pipeline (your business logic) gets reexecuted or not when another subscriber starts listening. Most of the canned observable factory methods—`create()`, `interval()`, `range()`, `from(Array|scalar|generator*)`, and others—are cold by default. The diagram in figure 8.8 illustrates the differences between these mechanisms. A replay emits the same output to all subscribers without invoking the operator sequence.

Figure 8.8 When replaying, the output emitted by an observable sequence is shared or broadcast to all subscribers.

In contrast, a cold resubscribe (figure 8.9) invokes the sequence of operators that lead to the result for every subscriber.

Figure 8.9 A resubscribe causes the producer and the observable sequence to execute. If the operator sequence has side effects, then new subscribers could see different results.

We'll show a simple example showcasing both scenarios and the impact a side effect can have. For this, you'll build custom observables whose behavior depends on the time of day (a side effect). In this case, you'll emit events until the time reaches 10:00 p.m., at which point they'll fire the complete signal. 

### 8.3.2 Replaying the logic of a stream

To showcase how a replay works, consider a body of time-sensitive code wrapped in a `Promise` that will emit a value of "Success!" before 10:00 p.m. or throw an exception if executed after. The first observer that subscribes before 10:00 p.m. will cause the `Promise` to execute and resolve. Any observers that subscribe later will receive the same value without invoking the body of the `Promise`. The business logic is ignored: 

```js
const p = new Promise((resolve, reject) => {
    setTimeout(() => {
        let isAtAfter10pm = moment().hour() >= 20;//*1
        if (isAtAfter10pm) {
            reject(new Error('Too late!'));
        }
        else {
            resolve('Success!');
        }
    }, 5000);
});
const promise$ = Rx.Observable.fromPromise(p);
promise$.subscribe(val => console.log(`Sub1 ${val}`));//*2
// ... after 10 pm ...//
promise$.subscribe(val => console.log(`Sub2 ${val}`));//*3
```

1. Uses moment.js to check if the current time is 10:00 p.m.
2. First subscriber executes the Promise
3. Second subscriber will emit the same value, regardless of the time it subscribed, because it won't run the code within the body of the `Promise`

Regardless of the time of the subscription, any observers that subscribe to this stream receive the same value, whether it's a success or failure. This cold observable behaved predictably, but this is only because of the way `Promise`s work. Querying the result of a fulfilled `Promise` always outputs the same value. So the body of the `Promise`, in this case, doesn't actually run when the second subscribe occurs—this is a replay, and the `fromPromise()` static operator is hot. Now let's look at the case of a resubscribe.

### 8.3.3 Resubscribing to a stream

Consider this custom observable that emits numbers every second with, again, two observers subscribed to it at different times. The first subscription, `Sub1`, happens before 10:00 p.m. and immediately begins receiving events, whereas the second, `Sub2`, happens after and is terminated immediately:

```js
"Sub1 Starting interval..."//*1
"Sub1 Next 0"
"Sub1 Next 1"
"Sub1 Next 2"

"Sub2 Starting interval..."//*2
```

1. Subscription occurs before 10 p.m. and begins to receive values.
2. Subscription occurs after, so observer never sees the numbers emitted.

Let's examine this code. The reason this behaves differently compared to the code in the previous section is that the `create()` factory operator is cold by default:

```js
const interval$ = Rx.Observable.create(observer => {
    let i = 0;
    observer.next('Starting interval...');
    let intervalId = setInterval(() => {
        let isAtAfter10pm = moment().hour() >= 20;//*1
        if (isAtAfter10pm) {
            clearInterval(intervalId);//*2
            observer.complete();
        }
        observer.next(`Next ${i++}`);
    }, 1000);
});
// ... before 10 pm ... //
const sub1 = interval$.subscribe(val => console.log(`Sub1 ${val}`));//*3
// ... after 10 pm ... //
const sub2 = interval$.subscribe(val => console.log(`Sub2 ${val}`));//*4
```

1. Uses moment.js to check if the current time is 10:00 p.m.
2. Stops emitting events
3. Subscriber `sub1` begins listening.
4. After 10 p.m., `sub2` subscribes but ends immediately.

Resubscribing to this stream (with `sub2`) would create a new, independent stream, but it won't just blindly propagate the same values again. Instead it re-invokes the logic so that different subscribers would receive (or not) events based on when they subscribed. If the time reaches 10:00 p.m. before subscriber `sub2` has a chance to listen, it won't receive any events at all. So the fact that the observable begins emitting events when subscribed to indicates that it's a cold observable. But because you have a side effect in your code, preventing it from replaying the sequence, the results that subscribers see might be significantly different.

> **SIDE EFFECT ALERT** As we mentioned, the direct use of time in your code is clearly a sign of a side effect because, intuitively, it's global to your application and ever changing. This is why hot observables are the less-pure form when compared to cold observables, which should reemit (or replay) all the items to any subscribers. 

In the same vein, consider an operator you saw in the last chapter, `retry()`, which performs a resubscribe on an observable when an error occurs. Let's revisit using it as part of your stock ticker stream. Using `retry()` directly on the `Promise` seemed like a good idea: 

```js
const requestQuote$ = symbol =>
    Rx.Observable.fromPromise(ajax(webservice.replace(/\$symbol/, symbol)))
        .retry(3)
        .map(response => response.replace(/"/g, ''))
        .map(csv);
```

You might expect to retry a web request if it fails, resulting in up to four requests sent to the server before an error is finally served. But we mentioned a small caveat in that you may have then been surprised to see that the network debugger showed only a single request being executed. Why? Let's look at this in the context of hot and cold observables.

This goes back to our examination of how sources affect the behavior of observables. A `Promise` is an eager data type (read *hot observable*), which means that upon creation it can only ever resolve or reject and will do so even without listeners. `Promise`s are not retriable. So once it's in one of those two states, it stays there, and every new handler will receive either the resolved value or the error that caused the rejection. Because `Promise`s don't have retry, any attempt to retry through `fromPromise()` is futile because it just replays whatever the final state of the `Promise` is, by design. Recall that to get around this limitation, you wrapped the creation of the `Promise` in another observable, which is retriable, so that the operation contained inside it (the `Promise`) could be retried. That's why you moved it to its outer observable:

```js
const fetchDataInterval$ = symbol => twoSecond$
    .mergeMap(() => requestQuote$(symbol))//*1
    .retry(3)//*2
    .catch(err => Rx.Observable.throw(
        new Error('Stock data not available. Try again later!')));
```

1. The inner observable that's projected onto the two-second stream
2. This is the observable that retries the failed stream three more times; if not successful, it catches and propagates the error downstream.

Remember that this worked because you actually created a new `Promise` within the `mergeMap()` operator and the retry is resubscribing to the projected observable and not to the one created directly in `fromPromise()`. The resubscription rebuilds the pipeline and reexecutes it for the single value that you passed into it. So now you have a complete and deep understanding of why this technique works. You changed the temperature of this observable, so that it essentially behaves cold. 

To summarize, resubscribing to a cold observable creates independent event channels for each subscriber, which means each observer creates its own copy of the producer. In most situations, this is desirable. For instance, if you use observables to process a set of objects originating from a generator function, you'll definitely want to work on copies of this producer (created via the cold `from()` static operator) instead of sharing it. On the other hand, replaying can be effective and save you precious computing cycles when your intent is to broadcast or share the output of an observable sequence to multiple observers. 

In practice, when the producer resource is expensive to create, such as a remote HTTP call or a WebSocket, then sharing it is a smart thing to do. Let's examine how to heat up observables to accomplish this.

## 8.4 Changing the temperature of an observable

The resubscription mechanism of cold observables is easy to reason about because each stream carries its own copy of the producer, spawning a new pipeline back up to the source. This happens in RxJS by default when wrapping synchronous data sources like scalars and arrays but also through custom observables containing asynchronous data sources such as remote event emitters, AJAX calls, or WebSockets that are created within the observable context. In all these cases, you deal with cold observables. From a functional point of view, this is the purest solution because no data is being shared and the observable acts like a template (or a recipe, as we mentioned previously) for creating data, as shown in figure 8.10.

Figure 8.10 New subscribers to cold observables fork the event sequence and obtain their own copy.

But when resources are scarce, doing this can pose significant problems. In practice, it's beneficial to spawn one HTTP request, have a single event emitter instance, or create one WebSocket connection that many observers can share, instead of one for each. In section 8.3, we showed that wrapping an external socket (created outside the observable context) represents a hot observable. The WebSocket object in this case is the producer of data, and it's important to understand that the scope in which it's created is the ultimate thermometer, so to speak, to measure whether an observable is considered hot or cold. 

### 8.4.1 Producers as thermometers

Ben Lesh, who is the project lead for RxJS 5, wrote an interesting piece on Medium.com that explains hot and cold observables from the perspective of the producer (that is, WebSockets, eager HTTP requests via `Promise`s, and event emitters). He articulates it eloquently as follows:

> **COLD** is when your observable creates the producer.
> **HOT** is when your observable closes over the producer.

In this article, he treats producers as a generic object of type `Producer`, which represents any object capable of emitting data asynchronously—without necessarily being iterated over. Let's examine the terminology he uses. A cold observable "creates the producer," meaning that it's created within the scope of the observable context, for instance: 

```js
const cold$ = new Rx.Observable(observer => {
   const producer = new Producer();//*1
   // ...Observer listens to producer, 
   //    producer pushes events to the observer... 
   producer.addEventListener('some-event', e => observer.next(e));   
   return () => producer.dispose(); 
});
```

1. The lifecycle of the producer entity (a generic object) is bound to that of the observable's 

When this stream object is garbage collected, the underlying producer object gets collected with it. Likewise, when the stream is disposed of, it will invoke the mechanism to discard the producer as well. The other implication is that anything that subscribes to `cold$` will obtain its own copy of the producer object, as we've mentioned before. This one-to-one communication between a producer and a consumer (observer) is referred to as *unicast*.

> **UNICAST** In the world of computer networking, a unicast transmission involves the sending of messages to a single network destination identified by a unique source address.

On the other hand, hot observables "close over the producer object" that's created or activated outside the observable context. In this case, the lifecycle of the event emitter source is independent of that of the observable. The term *closes over* derives from the idea that the producer object is accessible through the closure formed around the observable declaration.

```js
const producer = new Producer();//*1
const hot$ = new Rx.Observable(observer => {
    // ...Observer listens to producer,
    //    and pushes events onto it...
    producer.addEventListener('some-event', e => observer.next(e));//*1
    // producer gets disposed of outside of Observable context
});
```

1. Producer object is in scope through the closure formed around the observable declaration

From our FP discussion, you can see that this is not pure because the observable object (or function) is accessing external data directly, which is a side effect. In practice, though, the benefit of doing this is that the producer is now shared by all subscribers to `hot$` and emits data to all of them—a model known as *multicast*.

> **MULTICAST** In computer networking, *multicast* refers to a one-to-many form of communication where information is addressed to multiple destinations from a single source.

Figure 8.11 explains the difference between the two modes of message passing.

Figure 8.11 Multicast transmits messages from one source to many destinations; unicast sends dedicated messages to one destination address.

To sum up, the multicast mode is the process behind hot observables, whereas unicast messaging occurs with cold observables. Depending on your needs, you can use RxJS to make hot observables cold for isolated, dedicated connections or vice versa for shared access to resources. Let's look at examples of each scenario.

### 8.4.2 Making a hot observable cold

So far, our de facto mechanism for fetching remote data has always involved using `Promise`s. Without you realizing it, you've already had to convert observable types; you did this in chapter 5 so that you could use `Promise`s to fetch fresh stock data. Now we can discuss that technique more in depth and frame the problem this way: if `Promise`s are hot because the value they emit is shared among all subscribers and are not repeatable or retriable, how can you use them to fetch new stock quotes? Wouldn't the stock price be the same all the time? Again, this has everything to do with where the observable was instantiated. If you execute the `Promise` request globally (eagerly), as in this simple code, the same value (or error) is essentially broadcast to all subscribers:

```js
const futureVal = new Promise((resolve, reject) => {
   const value = computeValue();
   resolve(value);
});
const promise$ = Rx.Observable.fromPromise(futureVal);
promise$.subscribe(console.log);//*1
promise$.subscribe(console.log);//*2
```

1. Begins invoking the `Promise`
2. After the first invocation of the `Promise` resolves, all subsequent subscriptions will resolve to the same value.

To make this observable cold, you move the instantiation of the `Promise` within the observable context through `ajax()`. Here's a snippet of that code once more: 

```js
const requestQuote$ = symbol => 
    Rx.Observable.fromPromise(ajax('...'))//*1

const fetchDataInterval$ = symbol => twoSecond$
    .mergeMap(() => requestQuote$(symbol)
```

1. `ajax()` instantiates a new `Promise` within the observable context.

In essence, this is analogous to what you just learned, which is to move the source or the producer of events into the observable context: 

```js
const coldPromise$ = new Rx.Observable(observer => {
    const futureVal = new Promise((resolve, reject) => {//*1
        const value = computeValue();
        resolve(value);
    });
    futureVal.then(result => {
        observer.next(result);//*2
        observer.complete();  //*3
    });
});
coldPromise$.subscribe(console.log);//*4
coldPromise$.subscribe(console.log);
```

1. Shoves the instantiation of the `Promise` into the observable
2. Emits the value from the `Promise`
3. Because you're expecting only a single value, completes the stream
4. Both subscribers will invoke the internal `Promise` object.

You can apply this same principle to WebSockets. In section 8.3.3, you used an observable to wrap a global, shared WebSocket object. Here it is once more: 

```js
const websocket = new WebSocket('ws://localhost:1337');//*1
const sub = Rx.Observable.fromEvent(websocket, 'message')
  .map(msg => JSON.parse(msg.data))
  .pluck('msg') 
  .subscribe(console.log);
websocket.onclose = () => sub.unsubscribe();//*2
```

1. Globally shared object
2. Socket's lifecycle is managed outside the observable sequence

If you instead wanted to have dedicated connections to each subscriber, you could enclose the activation of the producer within the observable context. This way, every subscriber of the stream would create its own socket connection:

```js
const ws$ = new Rx.Observable(observer => {
  const socket = new WebSocket('ws://localhost:1337');//*1
  socket.addEventListener('message', e => observer.next(e));
  return () => socket.close();
});
const sub1 = ws$.map(msg => JSON.parse(msg.data))
   .subscribe(msg => console.log(`Sub1 ${msg}`));
const sub2 = ws$.map(msg => JSON.parse(msg.data))
   .subscribe(msg => console.log(`Sub2 ${msg}`));//*2
```

1. Not global but instantiated with every observer
2. Gets its own dedicated socket connection

This also applies to the RxJS static factory operators such as `create()`, `from()`, `interval()`, `range()`, and others. As mentioned previously, because they're cold by default, you may want to make them hot with the intention of sharing their content and avoiding not only duplicating your efforts but also duplicating resources. Let's look at doing that next.

### 8.4.3 Making a cold observable hot

In this section, you'll apply the reverse logic. To make a cold observables hot, you need to focus on how they emit data and how subscribers access this data. Circling back to your stock ticker widget, this means that you must move the source of events (stock ticks) away from the observable pipeline. You would want to do something like this if you had stock data being pushed into different parts of the application and you wanted them all in sync with the same event timer. In this case, simply subscribing with multiple observers won't do the trick.

```js
const sub1 = tick$.subscribe(//*1
  quoteDetails => updatePanel1(quoteDetails.symbol, quoteDetails.price)  
);
const sub2 = tick$.subscribe(//*1
  quoteDetails => updatePanel2(quoteDetails.symbol, quoteDetails.price)  
);
```

1. `sub1` and `sub2` will use their own two-second intervals, which could get out of sync, and also fetch their own copies of stock data. You could optimize this so that the HTTP response can be parsed once and consumed by multiple observers.

The other problem with this approach is resource usage. With two subscribers, if you were to open the developer console, you'd notice that for each refresh action (2 seconds apart) there would be not one but two separate requests being sent to the server with each observer. This means that every subscriber would have its own independent interval stream, which would incur the expense of establishing the same remote connection multiple times to fetch data against the Yahoo API, as shown in figure 8.12.

Figure 8.12 With two cold subscriptions, each one is responsible for initiating and allocating resources to make AJAX calls against the same service.

Why create new HTTP connections when you could just share this data downstream to multiple subscribers? Again, the answer lies in how the source of the stream is activated—as a globally accessible resource or within the observable context—we can't stress that enough. You can expose your service that fetches stock data as a resource that multiple subscribers can observe. But how can you make AJAX calls hot? There are many ways of doing this, but one idea involves using event emitters with an internal polling mechanism, so that, in conjunction with the hot `Rx.Observable.fromEvent()` observable, you can pump stock data to all subscribers. This works just like a WebSocket, broadcasting stock ticks independently of when a subscriber exists. Take a look at an example of this in the next listing.

Listing 8.4 Stock ticker as event emitter  

```js
class StockTicker extends EventEmitter {//*1
    constructor(symbol) {
        super();
        this.symbol = symbol;
        this.intId = 0;
    }
    tick(symbol, price) {//*2
        this.emit('tick', symbol, price);
    }
    start() {
        this.intId = setInterval(() => {//*3
            const webservice =
                `http://finance.yahoo.com/d/quotes.csv?s=${this.symbol}&f=sa&e=.csv`;
            ajax(webservice).then(csv).then(
                ([symbol, price]) => {
                    this.tick(symbol.replace(/\"/g, ''), price);
                });
        }, 2000);
    }
    stop() {
        clearInterval(this.intId);
    }
}
const ticker = new StockTicker('FB');
ticker.start();//*4
const tick$ = Rx.Observable.fromEvent(ticker, 'tick',
    (symbol, price) => ({ 'symbol': symbol, 'price': price }))
    .catch(()=>Rx.Observable.throw(new Error('Stock ticker exception')));
const sub1 = tick$.subscribe( //#E *5
    quoteDetails => updatePanel1(quoteDetails.symbol, quoteDetails.price)
);
const sub2 = tick$.subscribe( //#E *5
    quoteDetails => updatePanel2(quoteDetails.symbol, quoteDetails.price)
);
```

1. Event emitter to encapsulate a `StockTicker`
2. This represents the 'tick' event.
3. Every two seconds, polls the stock service for new price data
4. Starts the event emitter
5. `sub1` and `sub2` will share the data from the same source of events.

As you can see from this code, whether you subscribe to the source or not, it will continue to fetch and send out price ticks. The big difference here is that you've decoupled the subscription from the activation of the event source, essentially now a hot observable. Because of this decoupling, the observable is also removed from the lifecycle of the event source (that is, `start()` and `stop()`), just like with WebSockets earlier. Thus, when all subscribers unsubscribe, the event emitter will continue to send data—it's just that no one is there to listen. This is expected, and most of the hot observables you'll find in the wild (DOM elements, a database, or the filesystem) won't emit a complete signal, because their lifecycle extends that of the observable. 

You shouldn't expect that to make cold observables hot you need to have them depend on global producers all the time. You want the best of both worlds and certainly the nice functional qualities of cold observables. There are many benefits to encapsulating the event source and having it managed through the observable's lifecycle (this also ensures fewer possibilities of memory leaks because all resources are collected and disposed of when they're completely unsubscribed from). On the other hand, you also don't want to duplicate your efforts and instead share the events from a single source to multiple subscribers. 

### 8.4.4 Creating hot-by-operator streams

So far, you've seen that the process of converting a cold stream to hot is to place the activation of the producer resource within the context of an observable. But this isn't the only way. Fortunately, RxJS provides a convenient operator called `share()` that does just that. It's so named because it shares a single subscription to a stream among multiple subscribers (kind of like the old days of DIRECTV, where a single satellite feed could operate multiple TVs in the same house). This means that you can place this operator right after a set of operations whose results should be common, and the subscribers to each of them will all get the same stream instance (without replaying the pipeline). Just as important, this operator takes care of the management of the underlying stream's state such that upon the first subscriber subscribing, the underlying stream is also subscribed to, and when all the subscribers stop listening (either through error or cancellation), the underlying subscription is disposed of as well. Brilliant! When a new subscriber comes in, the source is reconnected and the process is restarted. Here's a quick example that shows the same result shared without replaying the entire sequence: 

```js
const source$ = Rx.Observable.interval(1000)
    .take(10)
    .do(num => {
        console.log(`Running some code with ${num}`);
    });
const shared$ = source$.share();//*1
shared$.subscribe(createObserver('SourceA'));//*2
shared$.subscribe(createObserver('SourceB'));//*3

function createObserver(tag) {//*4
    return {
        next: x => {
            console.log(`Next: ${tag} ${x}`);
        },
        error: err => {
            console.log(`Error: ${err}`);
        },
        complete: () => {
            console.log('Completed');
        }
    };
}
```

1. Converts the cold observable to hot
2. When the number of observers subscribed to a published observable goes from 0 to 1, you connect to the underlying observable sequence.
3. When the second subscriber is added, no additional subscriptions are added to the underlying observable sequence. As a result, the operations that result in side effects are not repeated per subscriber.
4. Helper method to create a simple observer for standard out

Once the observable in front of `share()` is subscribed to, it's for all intents and purposes hot; this is known as *hot-by-operator*, and it's the best way to heat up a cold observable. Running this code illustrates that a single subscription is shared to the underlying sequence:

```js
"Running some code with 0"
"Next: SourceA 0"
"Next: SourceB 0"
"Running some code with 1"
"Next: SourceA 1"
"Next: SourceB 1"
... and so on...
"Completed"
"Completed"
```

---

##### Pitfall: sharing with a synchronous event source

The `share()` operator is useful in many cases where subscribers subscribe at different times but are somewhat tolerant of data loss. Because it can be used following any observable, it's sometimes confusing to newcomers who might be tempted to do the following:

```js
const source$ = Rx.Observable.from([1,2,3,4])
  .filter(isEven)
  .map(x => x * x)
  .share();
source$.subscribe(x => console.log(`Stream 1 ${x}`));
source$.subscribe(x => console.log(`Stream 2 ${x}`));
```

This code is often seen as an easy, efficient win for those new to reactive programming. If the pipeline executes for each subscription, then it makes sense that by adding the share operator you can force it to execute only once, and both observers can use the results. As the console will tell you, however, this does not appear to occur. Instead, only Stream 1 seems to get executed. The reason for this is twofold. The first is scheduling, which we'll gloss over for now because it's covered in a later chapter. In basic terms, subscribing to a synchronous source like an array will execute and complete before the second subscribe statement is even reached. The second reason is that `share()` has introduced state into your application. With it, the first subscription always results in the observable beginning to emit, and so long as at least one subscriber continues to listen, it will continue to emit until the source completes. If you're not careful, this kind of behavior can become a subtle bug. 

When dealing with observables that run immediately, like those in the example, this can result in only a single subscriber receiving the events.

---

Let's apply this to your original stock ticker stream so that you can avoid making unnecessary HTTP calls with each subscriber that subscribes to the stream; instead, you make only one call that broadcasts to any subscribers. Making your stock ticker stream hot is as simple as adding `share()` at the end of it:

```js
const tick$ = symbol$.mergeMap(fetchDataInterval$).share();
const sub1 = tick$.subscribe( //*1
   quoteDetails => updatePanel1(quoteDetails.symbol, quoteDetails.price)  
);
const sub2 = tick$.subscribe( //*1
   quoteDetails => updatePanel2(quoteDetails.symbol, quoteDetails.price)  
);
```

1. Just like with the global event emitter example, all subscribers will receive the same tick data. We'll show a more complete version of this code in a bit.

And now you've officially completed the stock ticker code. Here's the full rendition with all the parts added from the operators you learned about in chapters 5 through 8. Let's recap:

* *Chapter 5*—You learned how to use higher-order observables and flattening operators such as `mergeMap()`.
* *Chapter 6*—You learned to coordinate multiple observables with `combineLatest()`.
* *Chapter 7*—You added fault tolerance to your streams with retry and error handling. 
* *Chapter 8*—You learned how to convert a cold observable into a hot observable that shares its event data with many subscribers. To demonstrate this, you'll have two subscribers updating different parts of the site.

You'll put all of this together in listing 8.5 and enhance your stock widget with the ability to track the price and day's change for all stocks (we omit the CSS and HTML code, which you can find in the GitHub repository). With these changes, your UI is updated to look like figure 8.13.

Figure 8.13 Screenshot of the live HTML stock ticker component as it ticks every 2 seconds and includes additional logic to compute the stock's price change from the opening price (prices are subject to market conditions)

You'll make several changes to the code you started with in listing 5.6, because you'll combine the stock ticker stream with another stream against the same service to read the stock's opening price. Subtracting the current price from the previous price gives you the next change. These will be two independent subscriptions parting from the commonly shared `tick$` stream, which is now hot. The following listing shows the complete code and uses many of the techniques you've learned about so far. 

Listing 8.5 Complete stock ticker widget with change tracking

```js
//*1
const csv = str => str.split(/,\s*/);
const cleanStr = str => str.replace(/\"|\s*/g, '');

const webservice = //*2
    'http://download.finance.yahoo.com/d/quotes.csv?s=$symbol&f=$options&e=.csv';

const requestQuote$ = (symbol, opts = 'sa') => //*3
    Rx.Observable.fromPromise(
        ajax(webservice.replace(/\$symbol/, symbol)
            .replace(/\$options/, opts))) //*4
        .retry(3)
        .catch(err => Rx.Observable.throw(
            new Error('Stock data not available. Try again later!')))
        .map(cleanStr) //*5
        .map(data => data.indexOf(',') > 0 ? csv(data) : data);//*6

const twoSecond$ = Rx.Observable.interval(2000);//*7

const fetchDataInterval$ = symbol => twoSecond$
    .mergeMap(() => requestQuote$(symbol)
        .distinctUntilChanged((previous, next) => { //*8
            let prevPrice = parseFloat(previous[1]).toFixed(2);
            let nextPrice = parseFloat(next[1]).toFixed(2);
            return prevPrice === nextPrice;
        }));

const symbol$ = Rx.Observable.of('FB', 'CTXS', 'AAPL'); //*9
const tick$ = symbol$.mergeMap(fetchDataInterval$).share(); //*10
tick$.subscribe( //*11
    ([symbol, price]) => {
        let id = 'row-' + symbol.toLowerCase();
        let row = document.querySelector(`#${id}`);
        if (!row) {
            addRow(id, symbol, price);
        }
        else {
            updateRow(row, symbol, price);
        }
    },
    error => console.log(error.message));

tick$ //*12
    .mergeMap(([symbol, price]) =>
        Rx.Observable.of([symbol, price])
            .combineLatest(requestQuote$(symbol, 'o')))
    .map(R.flatten) //*13
    .map(([symbol, current, open]) => [symbol, (current - open).toFixed(2)])
    .do(console.log)
    .subscribe(
        ([symbol, change]) => {
            let id = 'row-' + symbol.toLowerCase();
            let row = document.querySelector(`#${id}`);
            if (row) {
                updatePriceChange(row, change);
            }
        },
        error => console.log(`Fetch error occurred: ${error}`)
    );

const updatePriceChange = (rowElem, change) => {
    let [, , changeElem] = rowElem.childNodes;
    let priceClass = "green-text", priceIcon = "up-green";
    if (parseFloat(change) < 0) {
        priceClass = "red-text"; priceIcon = "down-red";
    }
    changeElem.innerHTML =
        `<span class="${priceClass}"><span class="${priceIcon}">
    (${Math.abs(parseFloat(change)).toFixed(2)})
</span></span>`;
};
```

1. Helper function to split a string into a comma-separated set of values (CSV)
2. The Yahoo Finance web service to use with additional options used to query for the pertinent data
3. Function used to create a stream that invokes a Promise that fetches data from the web service and parses the result into CSV
4. Applies options to the request URI. s = symbol; a = asking price; o = open.
5. Converts output to CSV
6. Cleans string of any white spaces and unnecessary characters
7. The two-second observable used to drive the execution of the realtime poll
8. Propagates stock price values only when they've changed. This avoids unnecessary code from executing every 2 seconds when price values remain the same.
9. Stock symbols to render data for. These can be any symbols; keep in mind that to see live changes, you must run the program during market hours.
10. Shares the stock data with all subscribers
11. First subscription. Creates all necessary rows and updates the price amount in USD.
12. Second subscription. A conformant stream that combines the price of the stock at the open, so that it can compute the change amount for the day. It appends the next change price in the stock.
13. Uses Ramda to flatten the internal array of data passing through, making it easier to parse, for example, [[symbol, price], open] -> [symbol, price, open]

The `share()` method is a useful and powerful shortcut to implement these complex examples with relatively few lines of code. But to understand what's really happening under the hood, you need to dive a little deeper and explore the domain of an observable variety called `ConnectableObservable`.

## 8.5 Connecting one observable to many observers

We mentioned previously that in RxJS and the networking world, a single point-to-point transmission is known as unicast, and the one-to-many transmission is known as multicast. As you saw from the previous example, `share()` is a multicast operator, but there are other flavors or specializations of it, all derived from a single generic function known as `multicast()`. In practice, typically you'll never actually use `multicast()` directly, but rather one of its specializations.

> **RXJS 5 IMPROVEMENT** It's important to recall that the RxJS 5 team has done a great job at cutting the API surface pertaining to the set of operators used to share and publish values by about 75%.

A thorough explanation of all the specializations for multicasting (also known as the *multicasting operators* in RxJS parlance) can require a whole book of its own. We won't be using them much in this book, and you won't need them in your initial exploration of RxJS. When the time arises, `share()` is all you need (no pun intended). Nevertheless, it's important to be aware that RxJS gives you a lot more control over the amount and types of data to emit when you need it. So we'll spend some time talking about the most common ones:

* Publish
* Publish with replay
* Publish last

### 8.5.1 Publish

The first specialization is the operator `publish()`. This is the vanilla multicast specialization. The idea is to create an observable (like `share()`) that allows a single subscription to be distributed to several subscribers. The difference between these operators is one of simplicity versus control. Whereas `share()` automatically managed the subscription and unsubscription of the source stream based solely on the number of subscribers, `publish()` is slightly more low-level. Here it is, using the same `source$` stream as before:

```js
const source$ = Rx.Observable.interval(1000)
    .take(10) //*1
    .do(num => {
        console.log(`Running some code with ${num}`);
    });
const published$ = source$.publish();
published$.subscribe(createObserver('SubA'));
published$.subscribe(createObserver('SubB'));
```

1. Makes your stream finite

Now, when you run this code, there's no output, as if the stream is sitting idle. The issue you encounter is that `publish()` returns a derivation of observables known as a `ConnectableObservable`. This new type requires a more explicit initiation step than `share()`. Whereas the latter connects on the first subscription, the former requires another call to build the underlying subscription. You can start the source observable by calling `connect()` on the resulting `ConnectableObservable`:

```js
published$.connect(); //*1
```

1. Can be invoked at any point after the call to `publish()`

> **CAUTION** We made our stream finite in this code sample for an important reason. `connect()` is a low-level operator, which means you're bypassing all of the nice subscription management logic available in the core RxJS creational operators. In other words, it's not a managed subscription. `connect()` can be a powerful tool, but it's up to you to ensure that the stream is unsubscribed from at some point; otherwise, you'll cause memory leaks.

As soon as you call the `connect()` method, your observable will act just like the one in the previous example. You can visualize this process with figure 8.14.

Figure 8.14 Publish creates a hot observable whereby all subscribers begin receiving the events as soon as the call to connect is made. Otherwise, the source stream behaves like a cold observable, sitting idle until `connect()` is called. The `share()` operator would yield the same results except that the call to `connect()` would be done internally by the library.

If you were to examine `ConnectableObservable`, you'd see an interface roughly shaped like this:

```typescript
interface ConnectableObservable<T> extends Observable<T> {
  connect() : Subscription
  refCount(): Observable<T>  
}
```

> **NOTE** Observables that share subscriptions are generally called hot, whereas those that don't are called cold. But there are also observables that start emitting events only after the first subscription (a quality seen only in cold observables) and thereafter share their data to all subscribers (a hot quality). We sometimes refer to these observables as *warm*.

These are two important concepts to understand: 

* The `connect()` method returns a `Subscription` instance that represents the shared underlying subscription. Unsubscribing from it will result in both subscribers no longer receiving any events. You saw how to use `connect` in the previous example.
* The `refCount()` method is named after the garbage collection concept known as *ref counting*, or *reference counting*. The point is that it returns an observable sequence that stays connected to the source as long as there's a least one active subscription. Does this sound familiar? It should because the `share()` operator we discussed a moment ago is little more than an alias for `publish().refCount()`. 

Now you understand why `share()` is just a shortcut for what's happening under the hood. RxJS creates warm observables that multicast their values to all connected subscribers managed through the connectable observable, which keeps a count of all active subscribers. When all subscribers have unsubscribed, it will unsubscribe from the source observable. 

The use of internal ref counting is crucial to the efficiency of RxJS. As we mentioned earlier, an important difference between hot and cold observables is when their lives start and when they can be considered to have ended. Hot observables can produce events in the absence of observers, whereas cold observables don't become active until they have a subscription. A result of this is that hot streams will often have much longer lifespans than their cold counterparts. Whereas a cold stream will shut down when the subscriber shuts down, a hot one can continue running after the end of a subscription. This can have important implications, depending on the source. If a hot observable is unmanaged, that is, its state is not being maintained by `refCount()`, then its state (and the associated resources) can easily be forgotten about and cause a memory leak—which is what happens when a connectable observable emits infinite events. A majority of the included operators, such as `Rx.Observable.fromEvent()` which intrinsically wrap hot sources, take care to manage their own disposed state. If you ever find yourself creating a hot observable explicitly with one of the `multicast()` family of operators, it's worth asking yourself when those streams will be destroyed and how many will be created. It may also be helpful to identify where and when a stream would be disposed of and to explicitly unsubscribe from it to avoid taking up unnecessary resources. This is especially important in single-page applications with multiple views, where it's easy to create streams within a view without properly disposing of them.

Publish is just one flavor of hot observable; several others can be useful, depending on the desired behavior on subsequent subscription. Suppose you wanted to have a moving window of past values to be emitted to all observables. Earlier, we talked about the differences between replaying the results of a sequence and resubscribing to execute the entire sequence again. You can mix that concept with publish. 

### 8.5.2 Publish with replay

You could use another specialization of multicast called `publishReplay()` to emit the last 1, 10, 100, or all of the most recent values to all subscribers (obviously, this is another case of a warm observable). This operator uses several parameters to determine the characteristics of a buffer to maintain. And as with any of the buffering operators you learned about in chapter 4, we caution you again that the use of buffers can be dangerous when replaying entire sequences and the buffer grows indefinitely. You can see this clearly if you inspect the signature of this operator: 

```js
publishReplay(bufferSize = Number.POSITIVE_INFINITY, 
              windowTime = Number.POSITIVE_INFINITY)
```

This operator is analogous to the RxJS 4 `shareReplay()` operator, which had the same issue. So using `publishReplay()` with empty arguments can be dangerous. Here's an example of this operator. Unlike the publish example, to showcase the use of this operator, you have to simulate a subscriber coming at a later time: 

```js
const source$ = Rx.Observable.interval(1000) //*1
    .take(10)
    .do(num => { //*2
        console.log(`Running some code with ${num}`);
    });
const published$ = source$.publishReplay(2); //*3

published$.subscribe(createObserver('SubA'));//*4
setTimeout(() => {
    published$.subscribe(createObserver('SubB')); //*5
}, 5000)
published$.connect();
```

1. Begins a counter that pushes integers every second, starting at zero
2. Creates a side effect to show that it's running
3. Creates an observable that can store two past events and reemit them to any new subscribers
4. Subscriber A connects subscribers immediately, and begins receiving events from count 0.
5. Subscribing 5 seconds later, subscriber B should begin receiving events starting with the number 4, but because of the replay it will first receive 2 and 3.

Running this code would print the following output. What you'll notice here is that as soon as the second subscriber comes, it will first make sure to emit the last events in the stream (current and previous); afterward, both streams will replay the same events:

```js
"Running some code with 0"
"Next: SubA 0"
"Running some code with 1"
"Next: SubA 1"
"Running some code with 2"
"Next: SubA 2"
"Running some code with 3"
"Next: SubA 3"
"Next: SubB 2"  
"Next: SubB 3" //*1
"Running some code with 4"
"Next: SubA 4" //*2
"Next: SubB 4"
"Running some code with 5"
"Next: SubA 5"
"Next: SubB 5"
...
"Next: SubA 9"
"Next: SubB 9"
"Completed"
"Completed"
```

1. SubB will begin receiving the last two events (previous and current).
2. Both subscribers receive the same data.

Figure 8.15 is a diagram of what's happening in the code sample.

Figure 8.15 `publishReplay` with a buffer count of 2 will emit the last two elements in the buffer (current and previous) at the moment the subscriber subscribes to the stream. Afterward, both streams receive the latest value emitted.

Alternatively, when you want only the last value to be emitted, `publishLast()` will do the trick.

### 8.5.3 Publish last

`publishLast()` is simple to understand. It returns a connectable observable sequence that shares a single subscription containing only the last notification. This operator is analogous to `last()` (except non-blocking) in that it multicasts the last observable value from a sequence to all subscribers. Follow our simple example once more: 

```js
const published$ = source$.publishLast();
published$.subscribe(createObserver('SubA'));
published$.subscribe(createObserver('SubB'));  
published$.connect();
```

Running this code prints out the following: 

```js
"Running some code with 0"
"Running some code with 1"
...
"Running some code with 9"
"Next: SubA 9"
"Next: SubB 9"
"Completed"
"Completed"
```

There are many overloaded specializations of multicast() in RxJS. You can find them all here: https://github.com/ReactiveX/rxjs/blob/master/src/operator/publish.ts. If you inspect these operators, you'll find that they delegate most of their work to entities called *subjects*.

---

##### Further reading

Theoretically, *subjects* can be used to supplant virtually all of your observable needs, and the possibilities are endless. Because this is an advanced topic, in this book we prefer sticking to the managed observables created via RxJS factory methods. But if you would like to read on further, we recommend you begin with the RxJS manual: http://reactivex.io/rxjs/manual/overview.html#subject.

---

RxJS provides hot and cold observables because the designers recognized that different types of sources must often be addressed, with different needs. There is no one-size-fits-all solution to every problem, but `share()` will get you a long way ahead. But to that end, there's also a steeper learning curve to mount in order to fully understand the library, especially the topic of multicasting operators. We hope this chapter has demystified some of the strangeness around these different observable types. But when all else fails, here's a good analogy to keep in mind (taken from the Reactive Extensions GitHub project):

* A cold observable is like watching a movie.
* A hot observable is like watching live theater.
* A hot observable replayed is like watching a play on video.

These lessons will be useful when we move into the next chapter and discuss how to test your Rx pipelines.

## 8.6 Summary

* A cold observable is passive in that it waits until a subscriber is listening to execute an individual pipeline for each subscriber. Cold observables manage the lifecycle of the event producer.
* Hot observables are active and can begin emitting events regardless of whether subscribers are listening. Hot observables close over the producer of events, so their lifecycles are independent of the source.
* Event emitters such as WebSockets and DOM elements are examples of hot observables.
* Events from hot observables will be lost if no one is listening, whereas cold observables will always rebuild their pipeline upon every subscription.
* `share()` makes observers use the same underlying source stream and disconnects when all the subscribers stop listening. This operator can be used to make a cold observable hot—or at least warm.
* Using operators such as `publish()`, `publishReplay()`, and `publishLast()` creates multicast observables. 