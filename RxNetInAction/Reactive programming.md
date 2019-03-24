# 1. Reactive programming

The reactive programming paradigm has gained increasing popularity in recent years as a model that aims to simplify the implementation of event-driven applications and the execution of asynchronous code. Reactive programming concentrates on the propagation of changes and their effects—simply put, how to react to changes and create data flows that depend on them. 

With the rise of applications such as Facebook and Twitter, every change happening on one side of the ocean (for example, a status update) is immediately observed on the other side, and a chain of reactions occurs instantly inside the application. It shouldn't come as a surprise that a simplified model to express this reaction chain is needed. Today, modern applications are highly driven by changes happening in the outside environment (such as in GPS location, battery and power management, and social networking messages) as well as by changes inside the application (such as web call responses, file reading and writing, and timers). To all of those events, the applications are reacting accordingly—for instance, by changing the displayed view or modifying stored data. 

We see the necessity for a simplified model for reacting to events in many types of applications: robotics, mobile apps, health care, and more. Reacting to events in a classic imperative way leads to cumbersome, hard-to-understand, and error-prone code, because the poor programmer who is responsible for coordinating events and data changes has to work manually with isolated islands of code that can change that same data. These changes might happen in an unpredictable order or even at the same time. *Reactive programming* provides abstractions to events and to states that change over time so that we can free ourselves from managing the dependencies between those values when we create the chains of execution that run when those events occur. 

*Reactive Extensions* (Rx) is a library that provides the reactive programming model for .NET applications. Rx makes event-handling code simpler and more expressive by using declarative operations (in LINQ style) to create queries over a single sequence of events. Rx also provides methods called combinators (combining operations) that enable you to join sequences of events in order to handle patterns of event occurrences or the correlations between them. At the time of this writing, more than 600 operations (with overloads) are in the Rx library. Each one encapsulates recurring event-processing code that otherwise you'd have to write yourself. 

This book's purpose is to teach you why you should embrace the reactive programming way of thinking and how to use Rx to build event-driven applications with ease and, most important, fun. The book will teach you step by step about the various layers that Rx is built upon, from the building blocks that allow you to create reactive data and event streams, through the rich query capabilities that Rx provides, and the Rx concurrency model that allows you to control the asynchrony of your code and the processing of your reactive handlers. But first you need to understand what being reactive means, and the difference between traditional imperative programming and the reactive way of working with events.

## 1.1 Being reactive

### 1.1.1 Reactiveness in your application

## 1.2 Introducing Reactive Extensions

Now that we've covered reactive programming, it's time to get to know our star: Reactive Extensions, which is often shortened to Rx. Microsoft developed the Reactive Extensions library to make it easy to work with streams of events and data. In a way, a time-variant value is by itself a stream of events; each value change is a type of event that you subscribe to and that updates the values that depend on it.

Rx facilitates working with streams of events by abstracting them as observable sequences, which is also the way Rx represents time-variant values. *Observable* means that you as a user can observe the values that the sequence carries, and *sequence* means an order exists to what's carried. Rx was architected by Erik Meijer and Brian Beckman and drew its inspiration from the functional programming style. In Rx, a stream is represented by *observables* that you can create from .NET events, tasks, or collections, or can create by yourself from another source. Using Rx, you can query the observables with LINQ operators and control the concurrency with *schedulers*; that's why Rx is often defined in the Rx.NET sources as $Rx = Observables + LINQ + Schedulers$. The layers of Rx.NET are shown in figure 1.4.

* LINQ operators for events
* Event streams
* Schedulers

Figure 1.4 The Rx layers. In the middle are the key interfaces that represent event streams and on the bottom are the schedulers that control the concurrency of the stream processing. Above all is the powerful operators library that enables you to create an event-processing pipeline in LINQ style.



You'll explore each component of the Rx layers as well as their interactions throughout this book, but first let's look at a short history of Rx origins.

### 1.2.1 Rx history

### 1.2.2 Rx on the client and server

### 1.2.3 Observables

*Observables* are used to implement time-variant values (which we defined as observable sequences) in Rx. They represent the *push model*, in which new data is pushed to (or notifies) the observers.

Observables are defined as the source of the events (or notifications) or, if you prefer, the publishers of a stream of data. And the push model means that instead of having the observers fetch data from the source and always checking whether there's new data that wasn't already taken (the *pull model*), the data is delivered to the observers when it's available.

Observables implement the `IObservable<T>` interface that has resided in the `System` namespace since version 4.0 of the .NET Framework. 

```F#
type IObservable<'T> =
	abstract Subscribe:IObserver<'T> -> IDisposable
```

The `IObservable<T>` interface has only one method, `Subscribe`, that allows observers to be subscribed for notifications. The `Subscribe` method returns an `IDisposable` object that represents the subscription and allows the observer to unsubscribe at any time by calling the `Dispose` method. Observables hold the collection of subscribed observers and notify them when there's something worth notifying. This is done using the `IObserver<T>` interface, which also has resided in the `System` namespace since version 4.0 of the .NET Framework, as shown here. 

```F#
type IObserver<'T> =
    abstract OnNext     :T -> unit       
    abstract OnError    :Exception->unit
    abstract OnCompleted:unit->unit
```

The basic flow of using `IObservable` and `IObserver` is shown in figure 1.6. Observables don't always complete; they can be providers of a potentially unbounded number of sequenced elements (such as an infinite collection). An observable also can be "quiet," meaning it never pushed any element and never will. Observables can also fail; the failure can occur after the observable has already pushed elements or it can happen without any element ever being pushed. 

This observable algebra is formalized in the following expression (where $*$ indicates zero or more times, $?$ indicates zero or one time, and $|$ is an OR operator):
$$
OnNext(t)* (OnCompleted() | OnError(e))? 
$$
Figure 1.6 A sequence diagram of the happy path of the observable and observer flow of interaction. In this scenario, an observer is subscribed to the observable by the application; the observable "pushes" three messages to the observers (only one in this case), and then notifies the observers that it has completed.

When failing, the observers will be notified using the `OnError` method, and the exception object will be delivered to the observers to be inspected and handled (see figure 1.7). After an error (as well as after completion), no more messages will be pushed to the observers. The default strategy Rx uses when the observer doesn't provide an error handler is to escalate the exception and cause a crash. You'll learn about the ways to handle errors gracefully in chapter 10.

Figure 1.7 In the case of an error in the observable, the observers will be notified through the `OnError` method with the exception object of the failure.

---

#### The Observer design pattern

In certain programming languages, events are sometimes offered as first-class citizens, meaning that you can define and register events with the language-provided keywords and types and even pass events as parameters to functions.

For languages that don't support events as first-class citizens, the Observer pattern is a useful design pattern that allows you to add event-like support to your application. Furthermore, the .NET implementation of events is based on this pattern.

The Observer pattern was introduced by the Gang of Four (GoF) in Design Patterns: Elements of Reusable Object-Oriented Software (Addison-Wesley Professional, 1994). The pattern defines two components: subject and observer (not to be confused with `IObserver` of Rx). The observer is the participant that's interested in an event and subscribes itself to the subject that raises the events. This is how it looks in a Unified Modeling Language (UML) class diagram:

The Observer design pattern class diagram

The observer pattern is useful but has several problems. The observer has only one method to accept the event. If you want to attach to more than one subject or more than one event, you need to implement more update methods. Another problem is that the pattern doesn't specify the best way to handle errors, and it's up to the developer to find a way to notify of errors, if at all. Last but not least is the problem of how to know when the subject is done, meaning that there will be no more notifications, which might be crucial for correct resource management. The Rx `IObservable` and `IObserver` are based on the Observer design pattern but extend it to solve these shortcomings. 

---

### 1.2.4 Operators

Reactive Extensions also brings a rich set of operators. In Rx, an *operator* is a nice way to say operation, but with the addition that it's also part of a domain-specific language (DSL) that describes event processing in a declarative way. The Rx operators allow you to take the observables and observers and create pipelines of querying, transformation, projections, and other event processors you may know from LINQ. The Rx library also includes time-based operations and Rx-specific operations for queries, synchronization, error handling, and so on.

For example, this is how you subscribe to an observable sequence of strings that will show only strings that begin with A and will transform them to uppercase:

```F#
let strings = observe {yield ""}
let subscription =
    strings
        .Where(fun str -> str.StartsWith("A"))
        .Select(fun str -> str.ToUpper())
        .Subscribe(fun _ -> ())

//Rest of the code
subscription.Dispose()
```

In this simple example, you can see the declarative style of the Rx operators—say what you want and not how you want it—and so the code reads like a story. Because I want to focus on the querying operators in this example, I don't show how the observable is created. You can create observables in many ways: from events, enumerables, asynchronous types, and more. Those are discussed in chapters 4 and 5. For now, you can assume that the observables were created for you behind the scenes. 

The operators and combinators (operators that combine multiple observables) can help you create even more complex scenarios that involve multiple observables. To achieve the resizable icon for the shops in the Shoppy example, you can write the following Rx expressions:

```F#
let stores = observe {yield ""}
let myLocation = observe {yield 0}

let iconSize = 
    rxquery {
        for store in stores do
        for currentLocation in myLocation do
        let distance = store.Length + currentLocation
        let size = distance * 2
        select (store, size)      
    }

iconSize.Subscribe(fun sz -> printf "%A" sz)
|> ignore
```

代码中的计算表达式是瞎编的，这不重要。

Even without knowing all the fine details of Reactive Extensions, you can see that the amount of code needed to implement this feature in the Shoppy application is small, and it's easy to read. All the boilerplate of combining the various streams of data was done by Rx and saved you the burden of writing the isolated code fragments required to handle the events of data change. 

### 1.2.5 The composable nature of Rx operators

Most Rx operators have the following format:

```F#
OperatorName:'arguments->IObservable<'T> 
```

Note that the return type is an observable. This allows the composable nature of Rx operators; you can add operators to the observable pipeline, and each one produces an observable that encapsulates the behavior that's been applied to the notification from the moment it was emitted from the original source. 

Another important takeaway is that from the observer point of view, an observable with or without operators that are added to it is still an observable, as shown in figure 1.8.

Figure 1.8 The composable nature of Rx operators allows you to encapsulate what happens to the notification since it was emitted from the original source.

Because you can add operators to the pipeline not only when the observable is created, but also when the observer is subscribed, it gives you the power to control the observable even if you don't have access to the code that created it. 

### 1.2.6 Marble diagrams

### 1.2.7 Pull model vs. push model

Nonobservable sequences are what we normally call *enumerables* (or collections), which implement the `IEnumerable` interface and return an iterator that implements the `IEnumerator` interface. When using enumerables, you pull values out of the collection, usually with a loop. Rx observables behave differently: instead of pulling, the values are pushed to the observer. Tables 1.1 and 1.2 show how the pull and push models correspond to each other. This relationship between the two is called the *duality principle*.

Table 1.1 How `IEnumerator` and `IObserver` correspond to each other

| `IEnumerator`             | `IObserver`               |
| ------------------------- | ------------------------- |
| `MoveNext`—when `false`   | `OnCompleted:unit->unit`  |
| `MoveNext`—when exception | `OnError:Exception->unit` |
| `Current`                 | `OnNext:'T->unit`         |


Table 1.2 How `IEnumerable` and `IObservable` correspond to each other

| `IEnumerable`                     | `IObservable`                      |
| --------------------------------- | ---------------------------------- |
| `GetEnumerator:unit->IEnumerator` | `Subscribe:IObserver->IDisposable` |

There's one exception to the duality here, because the twin of the `GetEnumerator` parameter (which is `void`) should have been transformed to the `Subscribe` method return type (and stay `void`), but instead `IDisposable` was used.

Because a reverse correspondence exists between observables and enumerables (the duality), you can move from one representation of a sequence of values to the other. A fixed collection, such as `List<T>`, can be transformed to an observable that emits all its values by pushing them to the observers. The more surprising fact is that observables can be transformed to pull-based collections. You'll dive into the details of how and when to make those transformations in later chapters. For now, the important thing to understand is that because you can transform one model into the other, everything you can do with a pull-based model can also be done with the push-based model. So when you face a problem, you can solve it in the easiest model and then transform the result if needed. 

The last point I'll make here is that because you can look at a single value as if it were a collection of one item, you can by the same logic take the asynchronous single item—the `Task<T>`—and look at it as an observable of one item, and vice versa. Keep that in mind, because it's an important point in understanding that "everything is an observable."

## 1.3 Working with reactive systems and the Reactive Manifesto

So far, we've discussed how Rx adds reactiveness to an application. Many applications aren't standalone, but rather part of a whole system that's composed of more applications (desktop, mobile, web), servers, databases, queues, service buses, and other components that you need to connect in order to create a working organism. The reactive programming model (and Rx as an implementation of that model) simplifies the way an application handles the propagation of changes and the consumption of events, thus making the application reactive. But how can you make a whole system reactive?

As a system, reactiveness is defined by being responsive, resilient, elastic, and message-driven. These four traits of reactive systems are defined in the *Reactive Manifesto* (www.reactivemanifesto.org), a collaborative effort of the software community to define the best architectural style for building a reactive system. You can join the effort of raising awareness about reactive systems by signing the manifesto and spreading the word.

It's important to understand that the Reactive Manifesto didn't invent anything new; reactive applications existed long before it was published. An example is the telephone system that has existed for decades. This type of distributed system needs to react to a dynamic amount of load (the calls), recover from failures, and stay available and responsive to the caller and the callee 24/7, and all this by passing signals (messages) from one operator to the other.

The manifesto is here to put the reactive systems term on the map and to collect the best practices of creating such a system. Let's drill into those concepts.

### 1.3.1 Responsiveness

When you go to your favorite browser and enter a URL, you expect that the page you were browsing to will load in a short time. When the loading time is longer than a few milliseconds, you get a bad feeling (and may even get angry). You might decide to leave that site and browse to another. If you're the website owner, you've lost a customer because your website wasn't responsive.

`Responsiveness` of a system is determined by the time it takes for the system to respond to the request it received. Obviously, a shorter time to respond means that the system is more responsive. A response from a system can be a positive result, such as the page you tried to load or the data you tried to get from a web service or the chart you wanted to see in the financial client application. A response can also be negative, such as an error message specifying that one of the values you gave as input was invalid.

In either case, if the time that it takes the system to respond is reasonable, you can say that the application is responsive. But a reasonable time is a problematic thing to define, because it depends on the context and on the system you're measuring. For a client application that has a button, it's assumed that the time it takes the application to respond to the button click will be a few milliseconds. For a web service that needs to make a heavy calculation, one or two seconds might also be reasonable. When you're designing your application, you need to analyze the operations you have and define the bounds of the time it should take for an operation to complete and respond. Being responsive is a goal that reactive systems are trying to achieve.

### 1.3.2 Resiliency

### 1.3.3 Elasticity

### 1.3.4 Message driven

### 1.3.5 Where is Rx?

## 1.4 Understanding asynchrony

Asynchronous message passing is a key trait of a reactive system. But what exactly is asynchrony, and why is it so important to a reactive application? Our lives are made up of many asynchronous tasks. You may not be aware of it, but your everyday activities would be annoying if they weren't asynchronous by nature. To understand what asynchrony is, you first need to understand non-asynchronous execution, or synchronous execution. 

**DEFINITION** *Synchronous*: Happening, existing, or arising at precisely the same time

*Synchronous* execution means that you have to wait for a task to complete before you can continue to the next task. A real-life example of synchronous execution takes place at a fast-food restaurant: you approach the staff at the counter, decide what to order while the clerk waits, order your food, and wait until the meal is ready. The clerk waits until you hand over the payment and then gives you the food. Only then you can continue the next task of going to your table to eat. This sequence is shown in figure 1.14.

This type of sequence feels like a waste of time (or, better said, a waste of resources), so imagine how your applications feel when you do the same for them. The next section demonstrates this. 

### 1.4.1 It’s all about resource use

Imagine what your life would be like if you had to wait for every single operation to complete before you could do something else. Think of the resources that would be waiting and used at that time. The same issues are also relevant in computer science: 

