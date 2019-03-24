# Chapter 10. RxJS in the wild

This chapter covers

* Integrating RxJS with other popular JavaScript libraries
* Introducing React and Redux
* Compartmentalizing UI components using React
* Feed-forward state propagation using Redux
* Rolling your own functional, asynchronous middleware using RxJS subjects
* Building a banking application using only reactive frameworks

The moment is finally here to answer a question you may have asked yourself a few times along this journey: "I can use RxJS to solve all of my asynchronous programming needs, but how can I use it in the context of an entire application?" This is a valid question and one that you'll answer by getting your hands dirty and seeing how RxJS plays out in the "Wild Wild Web."

This chapter is structured slightly differently from the previous ones, because we expect you to know most of the techniques from the earlier lessons by now. Given that you have a good understanding of Rx, we take the opportunity to introduce you to other technologies under the reactive umbrella called React and Redux. You can use these frameworks in conjunction with RxJS, and we think they're well worth your time learning about, especially if you're looking to scale the reactive paradigm to large JavaScript applications.

Arguably, you could use RxJS in conjunction with any competent web framework of your choosing, such as Backbone, Angular, or Ember, to name a few. But these frameworks promote paradigms (mostly object-oriented) that are very different from the functional and reactive paradigms you've learned about in this book. Although it's true that both can coexist, your aim is to create fully FRP applications, so you should prefer using frameworks that share similar principles.

Fortunately, the most difficult hurdle is the mental leap of changing paradigms, and you've successfully done that because you're here now. At a high level, the goal of this chapter is to teach you the components of an RxJS + React + Redux (what we call the *3R*) architecture. Understanding how 3R works can be overwhelming at first, and a basic understanding of React and Redux would help you going through this chapter, but it's certainly not necessary. To make all this content easier to digest, we've laid out the following roadmap: 

1. You'll explore the basic parts of React and Redux with simple examples (if you have previous experience with these technologies, feel free to skip these beginning sections and jump directly into section 10.4).
2. After rendering the UI using plain React components, you'll create a state management controller that allows you to communicate or pass state among these components. For this, you'll use Redux to dispatch actions that carry information as to how your state and corresponding UI changes.
3. You can implement simple synchronous state changes purely with Redux and simple functions, but asynchronous changes are much more complex (the story of our lives). Finally, you'll build a simple Redux middleware layer from scratch based entirely on RxJS. This middleware will allow you to dispatch asynchronous actions that can help you cope with asynchronous APIs like PouchDB.

We'll go through all these steps as you put together a simple banking site used to simulate the action of withdrawing money from a user's account and creating corresponding transactions. 

---

##### Where can I find this code?

In some places in this chapter, we'll provide roadmap cues to guide you along with this code. It's all available on GitHub for you to play with and manipulate at your leisure (https://github.com/RxJSInAction/banking-in-action). Instructions on how to set up and run it are available in the README and appendix A.

---

Let's begin by briefly talking about the application in question and why we chose these technologies.

## 10.1 Building a basic banking application

Before we begin, we should first set the scene. Our goal for the chapter is to build a basic banking application that can handle various aspects of a user's finances. In this case, we'll focus on the actions that occur when a user attempts to withdraw money from their account and all the actions that happen behind the scenes, like making sure every transaction is captured and stored.

Informally, this application has a few functional requirements that serve as the basis for our implementation decisions:

* It must track a user's current balance and update it based on transactions, which can either add (credit) or subtract (debit) from the balance.
* It must allow the user to change the specifics of a transaction, such as which account to access and how much money to add or subtract.
* It must not allow transactions that would create a negative balance for the user.

As with all applications, it's important to have an idea of the overall structure before diving into the specifics of it. In figure 10.1, we present the UI of the component you're about to implement.

Figure 10.1 Screenshot of the component that handles user transactions related to withdrawal and deposit

The architecture we're about to present fulfills four key requirements:

* Unidirectional state flow to go along with RxJS
* Immutable and side effect-free, or functional
* Unopinionated and lightweight
* Decoupled state and UI effects so that the business logic is agnostic to the view technology

The first two constraints (unidirectional state flow and immutable and side effect-free) should be obvious in light of the fact that this book is about functional, reactive programming using RxJS. The third requirement, to implement a lightweight architecture, is more about hitting as large a cross section of today's web developers as possible. Certainly, we could target older libraries or build from the ground up in vanilla JavaScript, but the reality is that, by and large, the JavaScript community has embraced various frameworks. Thus, in addition to showing how RxJS can stand alone, as we have for most of the book, we want to show off its use when other libraries are at play and how you can use it to extend your reactive architecture. The final point, about decoupling state changes and UI effects from the main logic, is in line with RxJS's theme of decoupling the production of the event from the logic that consumes it; now, we take this principle to heart and apply it globally to the entire application. 

For those reasons, we decided to present to you a lightweight RxJS architecture based on React and Redux. At a high level, these frameworks will layer in the manner illustrated in figure 10.2. With this simple picture in mind, our first stop in this journey is to introduce React and Redux.

Figure 10.2 Layered diagram of the 3R architecture that shows the hierarchy of the different layers of the system and the purpose each serves

## 10.2 Introduction to React and Redux

In order to understand why we chose React and Redux, you must first understand a little about them. First, what are they not? Neither React nor Redux is a complete framework compared to, say, AngularJS. We like this quality because we wanted to implement a lightweight architecture that includes RxJS. Unlike Angular, which is highly opinionated in its application design, React and Redux target single areas of concern and attempt to address them as well as they can. 

> **TERMINOLOGY** An opinionated framework is one that makes assumptions as to how you should organize and implement the components of your application. Redux has a minimal footprint and relies largely on using simple functions, thereby qualifying as an unopinionated framework.

Let's begin by learning more deeply about React.

### 10.2.1 Rendering UI components with React

React focuses on rendering the visible components that users interact with. React is all about controlling how (and if) components render on the screen. What sets it aside from other rendering technologies is how efficiently it decides to make updates and how decoupled it is from the logic leading up to the UI updates. 

#### MEET REACT

React UIs are built compositionally, which means that a single HTML element can result from the composition of multiple (smaller) React components. Updating React state triggers the component to rerender itself recursively, like a nested DOM, checking at each step if a change is needed to a given section.

Other contenders in the reactive web space are Cycle.js and Yolk. One reason for choosing React over these alternatives is that we wanted to give you a chance to see how RxJS works with a framework that doesn't directly use RxJS from the ground up. Although React and Redux certainly have properties that make using them with RxJS relatively straightforward, they were not designed explicitly to use it, thus making it a more interesting integration problem when it comes to understanding how to design around interfaces that don't quite fit—this is what RxJS in the wild is all about! 

#### REACT COMPONENTS

React components are the main unit of modularity and are remarkably simple. Most React components need only a `render()` method, which tells the React subsystem what to do when a component must be rerendered or drawn on the page. You'll need to know about only a couple of functions of the top-level API in this chapter. The first one is called `React.createElement()`; this is the method that's called when you want React to instantiate a particular element, like a `<span>` or a `<div>`. Here's what the top-level API looks like for this method: 

```js
React.createElement(
  type,
  [props],
  [...children]
) 
```

It effectively creates a virtual version of a DOM (also known as a shadow DOM) element, which is used by React to render your application on the screen. Now, this call on its own doesn't draw anything onscreen; that's the job of the particular renderer that you're using, like ReactDOM, for example. Aside from `render()`, ReactDOM offers other APIs for removing or unmounting elements and finding DOM nodes in the HTML tree. These aren't used that often. Here's a simple `"Hello RxJS!"` application with empty properties:

```js
const element = ReactDOM.render(
  React.createElement('div', {}, 'Hello RxJS!'), //*1
  document.getElementById('root')
);
```

1. Passes an empty props object (this component is essentially stateless)

This type of static rendering isn't particularly interesting, however; after all, you could have done this entirely with HTML in far fewer lines:

```js
const element = '<div>Hello RxJS!</div>';
```

The next step is to build components that can actually do something. For instance, what if you want to change the language of your greeting? To do that you'll need a new approach. In React, you can think of all components as functions; they receive an input and output a visual. The important argument to understand about these functions is the `props` object, which is the mechanism by which you can transfer state into the React component and its inner children—the input argument. Thus, when defining components, it makes sense that the simplest type of component would be a function. You can define a localized Hello RxJS component by creating a function like so:

```js
const HelloWorld = props =>
  React.createElement('div', {}, `${props.greeting} RxJS!`);
```

* By convention, it's typical to initial cap the function name to denote this is a React component function.

React components are analogous to treating UI updates like functions as well—they take an input and emit some output. In the previous example, you take the input properties in an object called `props` and use them to render a single element—a `component function`. Now, you can create it whenever you want with different parameters:

```js
ReactDOM.render(
  React.createElement(HelloWorld, {greeting: 'Hola'}),
  document.getElementById('root')
);
```

It's as simple as that. It's important to mention the close connection between React and FP in that all React components must act like pure functions with respect to their `props`. This means that a React component renders the exact same HTML output for the same `props`. This functional quality is known as *referential transparency* and it's the secret to why these components are so easy to compose and reuse.

Let's move on to a more interesting case. Imagine that instead of static data, you now want your data to change and update. A simple scenario for your banking application is an account balance component, which shows the current balance in a user's account. Remember the RxJS `interval()` factory operator? You can easily apply that here as well, given a simple AccountBalance component definition:

```js
const AccountBalance = props =>
    React.createElement('div', {},
        `Checking: ${props.checking} USD Savings: ${props.savings}`);
```

Next, you could invoke the render method each second to update the UI, mixing RxJS with some React to create a simple widget that updates account balances every second:

```js
Rx.Observable.interval(1000)
    .map(value => ({ checking: value, savings: value }))
    .subscribe(props =>
        ReactDOM.render(
            React.createElement(AccountBalance, props),
            document.getElementById('root')
        )
    );
```

As we mentioned earlier, React components are just plain functions that get invoked with parameters and return React (virtual HTML) elements, making them pure functions. Sometimes, however, this route is insufficient because you may want to also add event handlers, listen for when a component becomes mounted or attached to the DOM, or apply customizations to the state that's being passed in; then, you'd end up with very complex function bodies, which defeats the purpose. It's sometimes necessary for the developer to have more fine-grained control over a component's lifecycle as well as how it manages its state. React also exposes a mechanism for creating new components that offers more capabilities than a trivial function provides.

#### REACT CLASSES

The `React.createClass()` method is used for building more-advanced components where a function isn't enough. This static function takes an object bag as its argument, which contains definitions of the methods used during the lifetime of a component. The most important among these is, again, the `render()` method, which must be defined in order for the component to be used.

Try rewriting the account balances component to make use of this new approach:

```js
const AccountBalance = React.createClass({
    render() {
        let { checking, savings } = this.props
        return React.createElement('div', {},
            `Checking: ${checking} USD Savings: ${savings} USD`)
    }
});
```

Because this component is syntactically closer to a JavaScript class, notice that we're now reading `props` from the this object. The this keyword refers to the current context of a component, so it gives you access only to the component's state.

> **THE USE OF "THIS"** The use of this isn't common in functional programs because it implies you're accessing a scope outside the function or method, as the case may be. Nevertheless, considering that React is doing a good job of confining data access and mutations to the scope of the component itself (called the component's context) while you render data on the screen, it's a good trade-off to make. For the sake of making it easy to use, React relaxes the scope of data access from strictly local function scope in pure functional programs to the entire scope of a component.

So far, the previous component still isn't showing any new behavior, so let's add the next twist. The object bag passed to `React.createClass()` takes another method called `componentDidMount()`. It's called once by the React framework internally when a component instance has been initialized and has rendered on the page. So it's a good place to set up any initial logic and add some state into your UI. A React class lets you access this state through a property called `this.state` and allows you to update it through a method called `this.setState()`.

---

##### props vs. state

Earlier, we mentioned that `props` carries a React component's input. So what exactly is the difference between the props and state attributes? They're similar in concept but play different roles. First, both props and state make up the totality of a component's state. The former is used to configure the component—its options. It's received from its parent or the root, and it's immutable. Just like a pure function's input, `props` are not meant to change. On the other hand, state is meant to store data that will suffer mutations in time throughout the lifetime of a component.

---

But doesn't the thought of components exposing a window for other code to make changes to them violate the core functional principle of immutable objects? If we've done our job right, the very notion of sharing and changing state should be sending shivers down your spine. After all, state mutation is the root of all evil and what you're trying so hard to avoid! 

Although all those reasons for disliking state mutation are valid (you'll be thankful for Redux later on), React minimizes its effect in a couple of ways and protects you from the normal cesspool of state management:

* *All mutation is done through the `setState()` method.* This means that the state variable isn't directly accessed and changed; there's some intelligence behind it. In fact, as you'll see, the local context of state is always safe when calling `setState()`. 
* *React protects the individual state of components and contains it locally.* All state propagation is done through the `props` object that is then passed on to child states, meaning the parent component is responsible for initializing the `props` of a child component, which the child should never change thereafter. Remember, all components must act like pure functions with respect to their `props`, and this applies to all levels of the React element (DOM) tree.

With that said, let's move the interval logic into the `componentDidMount()` of `AccountBalances` so that it initiates with the component and localizes the effects of mutation to the component's context. You'll notice as well that instead of `this.props`, you're now effectively using `this.state` and `this.setState()` to read and write the current state of the component, respectively; the following listing shows these changes.

Listing 10.1 A React account balances that updates every second

```js
const AccountBalances = React.createClass({
    getInitialState() {
        return { checking: 0, savings: 0 };//*1
    },
    componentDidMount() {
        Rx.Observable.interval(1000)
            .map(value => ({ checking: value, savings: value }))
            .subscribe(state => this.setState(state))//*2
    },
    render() {
        return React.createElement('div', {},
            `Checking: ${this.state.checking} USD Savings: ${this.state.savings} USD`
        );
    }
});
```

1. Sets the initial state for the component
2. Because you're using an arrow function, the keyword "this" in this case refers to the component instance rather than the observable.

> **NOTE** It's imperative that you understand to never mutate `this.state` directly; it should be treated as immutable, and you should allow only React to manage these mutations. `setState()` doesn't directly or immediately cause the mutation to occur; instead, it creates a pending transition state that also gives you hooks to control whether the state should occur and which portions of it are allowed to change.

You can plug this back into the original render component, and voila! Self-rendering components that change every second!

```js
ReactDOM.render(
  React.createElement(AccountBalances, {}),
  document.getElementById('root')
);
```

This is the extent to which we'll cover core React in this book. The reason for this is that, for simplicity, we won't go into is JSX notation. A JSX preprocessor allows you to elegantly embed HTML directly into React components. We suggest you seek out materials devoted specifically to the topic in case you'd like to continue exploring it. And we honestly recommend you do. You can start with Manning's React in Action (www.manning.com/books/react-in-action) by Mark Thomas.

#### MANAGING THE STATE OF A REACT COMPONENT

Now that you can render React components, how can they communicate? A core philosophy of React is that every component feeds its state forward, just as you've become accustomed to with streams. In this case, it's propagated down to its children and not back up to its parent or out to any other component. This is very important to understand, because it will become the main reason why you'll use Redux later to dispatch state changes from one component to another. 

For instance, in your banking app, suppose you want to show the account balances of checking and savings accounts independently. For this, you'd create two balance components to handle each and possibly wrap over them a parent component that would receive the data and send it down to each child. Listing 10.2 shows a simple AccountBalance function that you call to render for checking and savings and then a wrapping Balances React class that sends data down to each. Another concept shown here is injecting an observable sequence as `props` of Balances, which will produce the required state dynamically to simulate a steady stream of cash flow (if it were only that easy).

Listing 10.2 Communicating to child components using a single parent

```js
//*1
const AccountBalance = props =>
    React.createElement('div', {}, `${props.name}: ${props.value} USD`);
//*2
const checking$ = Rx.Observable.timer(0, 1000);
//*3
const savings$ = Rx.Observable.timer(0, 1000 * 5);
//*4
const balance$ = Rx.Observable.combineLatest(checking$, savings$);
//*5
const Balances = React.createClass({
    getInitialState() {
        return { checking: 0, savings: 0 }; //*6
    },
    componentDidMount() {
        this.props.balance$
            .subscribe(([checking, savings]) => //*7
                this.setState({ checking, savings })
            );
    },
    render() {
        const { checking, savings } = this.state;
        return ( //*8
            React.createElement('div', {},
                React.createElement(AccountBalance,
                    { name: 'Checking', value: checking }),
                React.createElement(AccountBalance,
                    { name: 'Savings', value: savings }))
        );
    }
});
ReactDOM.render(
    React.createElement(Balances, { balance$ }), //*9
    document.getElementById('root')
);
```

1. Creates a simple subcomponent that formats its input
2. Updates checking every second
3. Updates savings every 5 seconds
4. Combines the two inputs into a single stream
5. Creates a composite class for both accounts
6. Initializes the balances to zero
7. Subscribes to updates on the balances
8. Renders the component as the composition of two subcomponents
9. Renders the balances component to the DOM and passes the balance$ stream as props to populate the UI with the illusion of constant cash flow into both accounts

Obviously, there are a number of ways that you could implement a seemingly simple text field. So, for those of you who are new to React, this may seem a bit of a roundabout way to render such a simple system. But what you've seen is the basis of a powerful concept. By combining RxJS streams with React components, you can create components that not only update in real time but also largely separate the concerns of the application. You achieve this separation by isolating UI changes and letting React efficiently manage the process, so that none of the components need to have an awareness of any others. This separation of the concerns and immutability of the data structures also prevents new features from interfering with existing ones. 

So far, you've seen how to render DOM components and how to change them. But most of the dynamic interaction with rich state UIs originates from handling user input. User interfaces that use React that need to both handle user input and then render some output on another component based on those interactions present a slight challenge to RxJS's unidirectional flow (producers to consumers). The challenge is that these observables are then inextricably linked to the state of a single component but need to make changes somewhere else. 

Take the standard search engine example you implemented in chapter 5. You learned that standard inputs can be used as event sources for data that comes from, say, keyup events. In the DOM, you could use a query to get the search bar element, attach the corresponding event handlers, and begin listening for events. After the stream is set up, you could begin subscribing to it from other parts of the application. If you were to implement the input box and the output list as React components, then the search results list, located somewhere else on the page, could subscribe to those events and update as new results arrive. But this becomes an issue in a feed-forward (unidirectional) model like React. Why? Remember that state flows smoothest when it's travelling to child components and not so well when it's going back upstream and into another isolated component, as shown in figure 10.3.

Figure 10.3 React components for search. Two different React components have no direct way of communicating because each component's state is completely encapsulated within the component. So you must resort to using external, shared variables or patterns such as an event bus.

When two isolated components need to communicate, this inevitably leads to sharing state variables, and you end up losing all the benefits of encapsulating state changes into a single downstream pipeline. This is important to understand. As you can see from figure 10.3, React components are good about keeping state to themselves, and it's meant to flow only downward. Therefore, if you modeled the search bar and the search results widgets as React components, there wouldn't be a direct way of sharing the data that originates in one with the other. This is a philosophical decision of React that states that components should never know (or care) whether another component is stateless or stateful. 

In React, because components are composed hierarchically, it's difficult for one state to update another if those components don't have a direct parent/child relationship (as we did with the AccountBalance earlier), as shown in figure 10.4.

Figure 10.4 React components are walled off from each other, so that state changes can be propagated only from the parent component down to its children but never in isolation.

In this figure, there isn't a clear way for these components to communicate. Of course, you could think of wrapping the entire search page into an overarching React component. But then you'd run the risk of the entire page rerendering instead of just the bits and pieces that change. Another common solution might be to use two-way data binding of models to views, like in Angular. But this is known to become challenging as the size of the application grows. In the same way as decoupled architectures, you could use a variation of the observer pattern called an *event bus*. Using an event bus, several isolated components could subscribe to a single source of information and receive messages pertaining to the topics they're listening to. Although this would arguably work, the downside is that event buses are multicast and omnidirectional because events can flow in any direction; this can lead to problems that are hard to troubleshoot once many components loaded onto the page subscribe to receive messages, as shown in figure 10.5.

Figure 10.5 A central event bus sends information from one component to another. These components can be React components or any other UI-rendering technology.

A downside to an event bus is that it's hard to picture what information flows through an event bus at any given time, and it clearly breaks the desired pattern of having information flow in a single direction. Also, it places the responsibility of writing the business logic of handling events of interest on the component itself. 

What can you do to fix this issue, and how can you transform this unwieldy web of events into a single-direction flow of events? Again, it's the same mindset at play. And here we reveal the key to reactive applications. As it turns out, while you're tempted to look at unidirectional flow in the context of a line, *you could also visualize it as a circle* (a line that ties back to itself). Say what? In its path, this circle has multiple components that have different responsibilities. Consider the diagram in figure 10.6.

Figure 10.6 The flow of states among different React components needs to be managed using actions and a controller that can dispatch such state changes and propagate them down to all components.

This figure shows that the final step is actually the first step and vice versa. So in the context of your banking application, a React component that needs to communicate with another may dispatch some sort of action (click the withdraw button). This action kicks off the necessary transformation in the state of the system (compute the final balance amount), and then this new data is passed on to the balances component (display the final balance). You can accomplish this by plugging in some type of storage controller layer that can help you manage your state, one than can act on a certain action to perform and have the results propagated downward into all React components that are listening for such an event. As you might have guessed, this state controller layer in your circle is called Redux!

### 10.2.2 State management with Redux

You've rendered simple components on the page, so now you'll move to your second stop on the roadmap, which is to use Redux to model your state management layer, the part of the system in charge of sharing data among your components. This section is a brief introduction to the pieces of Redux you'll need to know. 

Redux is a state container for JavaScript, and it takes care of how information flows through an application. Remember that state in RxJS is always transient, so for the purpose of temporarily retaining it, instead of creating mutable, global variables, Redux provides a read-only storage component.

In addition, Redux follows several principles of FP that you explored in earlier chapters. Primary among these features is the use of single-directional flow to eliminate the side effects of sharing global data between components on the page. In a typical React/Redux application, data flows to a React component whenever it's changed within the Redux store, and, likewise, actions triggered in React (a button click) cause state to update in the Redux store. Redux stores and then completes the circuit, so to speak, by using a simple subscription mechanism to notify React of state updates. Those updates can then be picked up by calling `getState()` on the Redux store (we'll circle back to the Redux subscribe system and how you can build on it later in the chapter). Another functional dogma, as mentioned earlier, is Redux's immutable store, where changes can be made only through pure functions that create new copies of this state and preserve the original. These functions are called *reducer*s. 

Redux implements a singleton store container—a *single source of truth* in Redux jargon. This makes tracking changes predictable, especially when used with React, because it makes updating several components easy to reason about, unlike an event bus or Angular's tight data binding of views and models. The Redux philosophy is to centralize all of an application's data into a single object in memory rather than having it spread out into multiple objects (the net memory footprint is much the same). This simple concept has far-reaching benefits, especially for debugging, the most important being that through browser tools you can inspect your entire application's state as a single object tree.

Visually, with Redux you can turn a complex update graph of connected components, like in figure 10.7, into a simple graph where arrows move from a centralized object out to each component as needed. Then, every React component decides whether to accept the change and how it should be made in the most efficient fashion (figure 10.8).

Figure 10.7 State changes should be done directly, because they create a dependency web that's too difficult to reason about.

Figure 10.8 The Redux `Store` object becomes the storage controller that, upon receiving an event (action), can dispatch and cause state to propagate down to all subscribed React components.

Understanding Redux requires a few simple principles that, given the exposure you just had to Rx, shouldn't be too complicated to grasp. Let's discuss how React components can communicate in an FRP manner using Redux. After we cover this integration, we'll work on integrating RxJS into the Redux middleware. 

## 10.3 Redux-ing application state

Now let's dive deeply into the components that make up a Redux implementation, starting with actions and reducers.

### 10.3.1 Actions and reducers

A *reducer* is a pure function that takes state and action objects and returns a brand-new state. An *action* is a simple object signaling the reducer function to invoke. You've seen instances of these sorts of functions when we were talking about the `reduce()` and `scan()` RxJS instance operators earlier in the book. Recall that you pass two arguments to `reduce`: a reducer function and an initial state. After that, each new event that arrives is subjected to this function and a new state is returned. The same concept happens in Redux. Reducers are similar to the simple React components that you saw earlier in that you can describe them as pure functions. Here's a simple reducer that can perform addition or subtraction, depending on the type of action that's being invoked:

```js
const mathReducer = (state = {
    result: 0
}, action) => {
    switch (action.type) {//*1
        case 'ADD':
            return {//*2
                ...state, result: state.result + action.value
            };
        case 'SUBTRACT':
            return {//*2
                ...state, result: state.result - action.value
            };
        default:
            return state;
    }
};
```

1. Reducers always key off of a type of operation to perform, and using string constants is the best approach and best practice in Redux.
2. The ES6 spread operator is a common idiom in Redux, because it helps you copy all the properties of the state object. The only thing that's left to do is to update only the attributes that change. Because the Redux single state is read-only, fresh copies of it are always passed back and forth.

The reducer itself is remarkably simple; it takes in a previous state, as well as an action, and it emits a new state object with the new result. It's important to note that a reducer should *never mutate the state object directly* under any circumstances, which is why the ES6 spread operator is a common Redux idiom to clone and return the state object in a single call. This is necessary to avoid polluting the state of the system. 

> **NOTE** Always return fresh objects from a reducer when any type of action takes place.

The next step is to apply that reducer to a piece of state and allow that state to change over time. This is critical if you expect to have the application respond to changes during the lifetime of a session. 

### 10.3.2 Redux store

To begin, you use `createStore()`, which creates a Redux store that holds the complete state tree of your application. `createStore()` takes a given reducer function and matches it with a persistent state. It can optionally take in an initial state so that it has somewhere to start.

```js
const store = createStore(mathReducer, 0);
```

* Stores points to your centralized Redux store object

You can see how this works with an example. In the example of bank accounts, you had a simple object with two balances, one for each of your accounts:

```js
const accounts = {
  checking: 100,
  savings: 100
}; 
```

You could build a reducer analogous to the `mathReducer` that updates the account balances like so:

```js
const updateAccounts = (state = {//*1
    checking: 0,
    savings: 0
}, action) => {
    switch (action.type) {
        case 'WITHDRAW':
            return { //#B
                ...state,
                [action.account]: state[action.account] -
                    parseFloat(action.amount)
            };
        case 'DEPOSIT':
            return { //*2
                ...state,
                [action.account]: state[action.account] +
                    parseFloat(action.amount)
            };
        default:
            return state;
    }
}
```

1. Initial (default) state
2. Clones the shape of the state object, and overrides only the checking and savings amounts with the new values in the action's payload

This code creates a reducer that returns an updated balance for the accounts after a withdrawal or a deposit has occurred. Thus, you can use it like so:

```js
//*1
const withdraw = {
    type: 'WITHDRAW',
    amount: 50,
    account: 'checking'
};

//*2
const newAccounts = updateAccounts(accounts, withdraw);
```

1. Actions have a type and a data (payload) property. The type of the action is what invokes the appropriate reducer.
2. Invokes the reducer with the current state and the action

The value returned from `updateAccounts()` would be the new balance state for the accounts. Plug that reducer into your app to get started:

```js
const store = createStore(updateAccounts, accounts);
```

This store comes with the additional logic to break down actions and create a new state based on those actions. This means that instead of manually calling `updateAccounts()`, you now update accounts through the store using `store.dispatch (action)`. Redux will manage invoking the proper reducer depending on the action. This goes back to our previous discussion about not making changes directly but using the frameworks to dispatch and manage such changes. Figure 10.9 shows this interaction.

Figure 10.9 The circular state diagram implemented using actions and reducers with Redux

A simple interaction between React and Redux, shown in figure 10.9, works in the following manner. The event originates from the React component and fires its event handlers, for instance, a button click or text box change. Then, the handler instantiates the corresponding action to take and uses Redux to dispatch the action, which in turn modifies the state in the centralized store. Lastly, this store gets propagated down to all React components, which then decide how best to react to this change.

Let's move into implementing the pieces of this interaction and see how you can begin to include RxJS in the mix. The store executes all the reducers, and the type of the action represents how parts of the state need to be updated. Instead of just instantiating action objects, the best practice is to use *action factories* (functions that return action objects). Then, you can create actions at will and avoid having to type them in all the time. You can see this in our next snippet:

```js
function withdraw(payload) { //*
    return { type: 'WITHDRAW', ...payload };
}
const action = withdraw({ amount: 50, account: 'checking' });
store.dispatch(action);
store.getState(); //-> {checking: 50, savings: 100}
```

* Action factory that creates an action object

As you can see, actions are easy to create. The only required element in Redux is the `type` property. Any other logic that you want to perform in the action body is up to you. We say Redux is an unopinionated framework because it applies few assumptions about the way you write your logic. Now, if you were to access the state of the store by calling `store.getState()`, you'd see that it had been updated by the `withdraw()` action. Now let's tie this into React. The `dispatch()` method of the store can then be used in your React components to send out events without worrying about their consumers or tightly coupling them to each other. For instance, to begin implementing your withdraw functionality in your simple banking form, you can use `dispatch()` in place of event handlers, as shown in the following listing.

Listing 10.3 Simple banking form with checking text field and withdraw button

```js
function handleClick(amount) { //*1
    const { checking, savings } = store.getState();
    if (checking > amount) {
        store.dispatch(withdraw({ amount, account: 'checking' })); //*2
    }
    else {
        throw 'Overdraft error!'; //*3
    }
}

React.DOM.button(
    {
        id: 'withdraw',
        onClick: () =>
            handleClick(document.getElementById('amount').value)
    },
    'Withdraw'); //*4
```

1. Computes the result of a withdraw synchronously and emits an action to update the balances in the store
2. Updates the accounts if the transaction is allowed to occur
3. Otherwise, fires an overdraft error
4. Reads the value from the text box and attempts a withdraw

This code allows you to send events to the store, but how would you then access those events? Remember, your application needs to be reactive, which means that ideally you should be able to listen for changes in the application state about the store you created.

As it turns out, there is a mechanism for this behavior. The store is already an observable or is "observable-like" with a much narrower observable specification and doesn't have all of the fancy bells and whistles that you've come to expect from your RxJS streams. That being said, it does feature a `subscribe()` method (it's a "subscribable" object), so you can convert it into an RxJS stream with little difficulty. Because Redux deviates from what you'd consider your normal observable pattern, you'll need to do a bit of adaptation to make it fit. Here's where RxJS fits into the mix.

## 10.4 Building a hot RxJS and Redux store adapter

Integrating RxJS into your React/Redux architecture is easy. It's important to mention that a lot of the code you'll see in the next sections used to integrate RxJS and Redux has already been implemented in a third-party, open source library called **redux-observable** (https://github.com/redux-observable/redux-observable), which makes this integration seamless using a higher level of abstraction. But because this book is devoted to RxJS, we didn't want to just glance over this feature and thought it would be nice to implement this integration ourselves using pure RxJS constructs. It would also give us the chance to introduce another cool feature called `Subject`, which you're bound to come across as you explore more RxJS.

Although RxJS and Redux are designed to solve different problems, you have amazing leverage in the fact that Redux stores are observable-like and so you can cross over framework lines with a simple adaption. 

The first thing you always should do is run down the list of normal factory methods that you could use for this purpose. The best option is to use the `from()` operator. This operator is special and very intelligent because it takes any observable-like objects and converts them into real observables. These include arrays and generators, as you've seen all along, but also objects that conform to the ES7 Observable specification, which Redux stores do to enough extent. Let's see how this works:

```js
function createStreamFromStore(store) {
    return Rx.Observable.from(store)
        .map(() => store.getState())       //*1
        .publishBehavior(store.getState()) //*2
        .refCount();                       //*3
}
```

1. `store.getState()` is called twice, so that subscribers always receive the latest state changes.
2. `publishBehavior()` is a flavor of a multicast (hot) operator that emits the latest value to all subscribers.
3. Makes this stream go live as soon as the first observer subscribes

As you can see, Redux's observable-like behavior is somewhat primitive compared to what you've dealt with in this book. Internally, Redux has been passed a `next` callback, which it invokes on each state change. But each time the `next` callback is invoked, it's only notifying observers of available data, not emitting it. Observers are required to explicitly call `store.getState()` in order to see the current state of the world. To amend this, you can use the map operator to call the getState method on every update and forward that state downstream. 

It's important to note that in `createStreamFromStore`, the first call to `store.getState()` is equivalent to the initial state. If there are no changes between the creation of the stream and its first subscribers' `subscribe()`, then they'll all receive that state. If a change occurs before a subscriber subscribes, then that change is now stored in `publishBehavior()`, and all new subscribers will receive the new state instead. This function is trivial but very powerful when used to build the bridge connecting Redux with Rx. Here, you've simply lifted the store into an RxJS observable and then mapped each next value onto the current state of the store's state. This simple mapping makes the Redux store seamlessly work like a normal RxJS observable. Internally, RxJS is subscribing to the store. The last part of this function converts the stream into a hot one. This is to optimize the stream and serve many subscribers at once. The `publishBehavior()` operator is a special flavor of the `publish()` operators from chapter 8, which store and emit the most recent value shared with all subscribers. This is useful if components will be hooking up at different times, which may or may not be before the observable emits. This way, you make sure that subscribers always get the latest state when they subscribe. 

Earlier, we showed how you can create dynamic behavior using RxJS to update React components. Now, you'll use `createStreamFromStore()` to add asynchronous data flows, so that you can use async APIs like PouchDB, for example, to persist your withdrawal transactions. 

## 10.5 Asynchronous middleware with RxJS Subject

Finally, you've arrived at the last stop of our roadmap before we show you all the pieces of your 3R architecture: building your Rx-based asynchronous middleware. What's the benefit of integrating RxJS into Redux? The issue is that, by design, everything in the eyes of Redux happens synchronously: dispatch an action, execute the reducers, and modify the state; all steps occur one after the other. Suppose you needed to perform some asynchronous action. How can you introduce wait times, or the latency of making an AJAX call to fetch data, or perhaps invoke a PouchDB call to persist a record in local storage? In this case, you want to be able to track every transaction (withdrawal or deposit) in the local store. You can examine the interaction diagram in figure 10.10 in order to understand where the challenge is.

Figure 10.10 Async actions introduce the complicated state management needed to keep track of the progress of the action from the moment the action starts processing to when it eventually returns. In this case, an async flow is used to call the PouchDB APIs to retrieve initialization data.

Notice how the linear flow is broken in step 2 in an effort to accommodate asynchronous logic. The issue is that, to account for latency, your middleware actions need to invoke multiple subactions or subflows to signal when the long-running operation has completed (the normal Redux idiom is to dispatch actions with status flags like DONE). One of the canons of reactive architectures is that the application must always be responsive. Hence, the first action that needs to get dispatched is the one signaling that the asynchronous call began (DONE, false). You place that flag into the store, so that your application knows not to fire another action simultaneously. 

> **TIP** For those with experience in concurrent processing, this is similar to using a semaphore.

Once the call completes, a second subflow is dispatched, signaling the completion of the event (DONE, true). Finally, the processing status is reset in the store, and the application is free to spawn subsequent actions.

For brevity, we won't show you the full code, but you can imagine your withdraw action being conditioned like so: 

```js
function shouldWithdraw(payload) {
    return (dispatch) => { //*
        if (payload.done) {
            return dispatch(withdraw(payload));
        }
    }
}

store.dispatch(shouldWithdraw({ amount: 50, account: 'checking' }));
```

* This form of action definition is also supported by Redux.

Checking for additional flags in your actions clutters up all of your code. As you know by now very well, RxJS is the perfect tool to model asynchronous logic as a single unidirectional line. In other words, you can build an observable pipeline through which actions that can execute all sorts of asynchronous behavior can flow serially. Thus, you can refactor step 2 to spawn asynchronous requests linearly as RxJS operators so that you can regain that logical unidirectional data flow and keep everything serial, as if it all happened synchronously. Remember the main goal of RxJS that you began learning about in chapter 1: Treat asynchronous code as though it was synchronous.

For simplicity, we'll zoom in on just that part of the interaction in step 2, as shown in figure 10.11.

Figure 10.11 Observables can string together actions and model them as a single downstream flow. The business logic encoded in the observable pipeline might produce an exception, which the withdraw action transforms into an error action sent back to the store. This logic will live in a function called an epic.

Using RxJS in the middleware means that every action that passes through gets wrapped within an observable, processed through the pipeline, and emitted as an event. That means that you can eliminate the use of callbacks from your action logic. Also, you unlock all the power of RxJS to, say, throttle the withdraw action as the user repeatedly clicks the withdraw button. This will be implemented in a component called an epic, which we'll come back to after you build your asynchronous middleware. 

To build your asynchronous middleware, you need to learn about one last core RxJS feature. Let's take a brief pause from Redux to discuss another cool feature of RxJS called a Subject. We'll look at a couple of simple examples using subjects, and then we'll circle back (no pun intended) to wrap up our discussion of reactive architecture.

### 10.5.1 RxJS subjects

If observables emit and observers receive, wouldn't the ultimate monster mash-up be an object that can do both? Rest assured; RxJS has you covered. A Subject is a two-headed beast that implements both the Observable and the Observer interfaces, so it can both produce and consume events. If you were curious, you'd find that subjects have this rough interface: 

```typescript
interface Subject extends Observable implements Subscription {}
```

This ability suggests that they're the brains behind creating hot observables. Subjects allow you to do things that you might not otherwise be able to do with regular observable factory operators. For instance, they allow you to multicast a single source into multiple outputs, which is why they're so attractive when mixed with Redux to propagate changes to multiple React components. Here's a simple example that uses subjects.

Listing 10.4 Multiple subscriptions with subjects

```js
const subject = new Rx.Subject();                      //*1
subject.subscribe(x => console.log(`Source 1: ${x}`)); //*2
subject.subscribe(x => console.log(`Source 2: ${x}`)); //*3
subject.next(0);                                       //*4
Rx.Observable.from([1, 2, 3, 4, 5])                    //*5
    .map(x => x * x)
    .subscribe(subject);
```

1. Explicitly creates a new `Subject`
2. First subscription to `subject`
3. Second subscription to `subject`
4. Explicitly passes a value to the `subject`
5. Uses another observable to pass values to the `Subject`

Running this code emits the values 0, 1, 4, 9, 16, 25 to all subscribers, just like `publish()`. In fact, `publish()` is just a facade that uses `Rx.Subject` to carry out its work. The publish family of operators that you learned about in chapter 8 leverages this ability to allow multiple subscribers to listen to the same source. This is possible because the subject won't notify the upstream when it gets new subscribers; thus, the only time that the source is subscribed to is when the subject initially subscribes. In general terms, this operation is described by an even more generic operator, aptly named `multicast()`, of which the set of overloaded `publish*()` operators is just calls to `multicast()` with different types of subjects. We show this in figure 10.12.

Figure 10.12 Multicast overloaded operators

`multicast()` accepts a subject (or subject variant) as its first argument and returns a `ConnectableObservable`. Downstream subscribers will always be subscribing to the subject, but the subject itself won't subscribe until `connect()` is called. Embedding subjects in a controlled and encapsulated manner can be extremely powerful; you'll use subjects to receive, process, and further propagate events to implement your business logic into the middleware of your banking application (more on this in the next section).

> **A WORD OF CAUTION** For all their power, subjects can also be dangerous. For many newcomers to Rx, they're a panacea of possibility because they allow developers to use observables without all the baggage of FP. Unfortunately, this kind of usage often leads to overexposure of state, so we recommend them only when you really need this feature.

The case for subjects comes down to a limited set of behaviors that need to be tightly constrained. For instance, you could emulate standard promise functionality natively in Rx using the `Subject` derivation called `AsyncSubject`.

The async subject behaves just like the vanilla subject when it comes to accepting values and then reemitting them. But it has an additional constraint; it holds onto only a single value and emits only that one value to all current and future subscribers once the subject has been completed. So it shouldn't surprise you that the `publishLast()` specialization is really just an async subject behind the scenes. In the following listing, we show a simple promise implementation using an async subject internally.

Listing 10.5 Build a promise-like operator with subjects

```js
Rx.Observable.promiseLike = function (fn) {
    let subject = new Rx.AsyncSubject();//*1
    let resolve = x => {                //*2
        subject.next(x);
        subject.complete();
    };
    let reject = e => {                 //*3
        subject.error(e);
    };
    fn(resolve, reject);                //*4
    return subject.asObservable();      //*5
};
```

1. Creates a new `Subject`
2. Simulates a promise's resolve method by delegating to the subject's next method to emit a value and then immediately completing the stream (one value only)
3. Simulates a promise's reject method by delegating to the async subject's error observer method
4. Invokes the function with the callbacks
5. Returns the `Subject` disguised as a regular Observable

This code creates a naïve promise-like interface that returns an observable instead. Using a random number function, we'll prove to you that this observable behaves like a promise:

```js
const randomInt = (min, max) => Math.floor(Math.random() * (max - min)) + min;
const random$ = Rx.Observable.promiseLike((resolve, reject) => {
    resolve(randomInt(0, 1000));
});
random$.subscribe(console.log);  //744
random$.subscribe(console.log);  //744
random$.subscribe(console.log);  //744
random$.subscribe(console.log);  //744
random$.subscribe(console.log);  //744
```

* All subscribers will receive the same value, in this case 744 (because it's a random function, results will vary).

In this code, you can see that we've addressed two of the issues that we highlighted earlier. First, we constrained the subject to a highly restricted scope and we do not expose references to the subject in the return value. And second, we have a defined lifetime for the subject. These two solutions tend to be good rules of thumb when explicitly including a subject in your code. Having learned subjects, you can build your middleware layer.

### 10.5.2 Building epic, reactive middleware

Generally speaking, middleware is a component or set of components, usually based on some plugin architecture, that can be injected between certain chunks of logic without impacting other parts of the system. Middleware allows for clean, noninvasive code that doesn't clutter your actions, reducers, or the UI. Because Redux is agnostic to whether state changes need to happen synchronously or not, you need some way of injecting this additional waiting logic where actions are generated and before they're consumed by a reducer. Let's see how you can accomplish this.

Redux is small but supports injecting asynchronous plugins like redux-thunk (https://github.com/gaearon/redux-thunk), redux-promise-middleware (https:// github.com/pburtchaell/redux-promise-middleware), and others. Figure 10.13 shows the flow of actions through the system when we plug in an RxJS-based middleware component that has actions flowing through observable streams that trigger a corresponding reducer on the Redux store.

Figure 10.13 A flow diagram of how actions and state move in the application. Actions flow from React/UI events to the Redux/React component after being processed by middleware and are then sent back to the UI as state updates.

This figure shows that any actions dispatched by the UI components can flow through your middleware component (implemented using RxJS), which carries all of your business logic and is ideal to handle your asynchronous needs while keeping the feed-forward flow intact. This is where you could write to your PouchDB database or invoke an AJAX call to fetch search results. When the data is available, it flows through a set of reducers before being propagated out (multicast) to all React components through `setState()`. RxJS brings you this advantage by encoding your asynchronous business logic into a set of operators—hence, reactive middleware.

The objective of using RxJS in the middleware is to create a layer that can consume an observable action sequence and then asynchronously emit potentially different actions that will be consumed by the store that resulted from the logic flowing through the observable. That's the purpose of the middleware: to receive actions, do some processing, and return similar or different actions. The actions to process are contained in a set of epics (we borrow the term from redux-observable). 

Epics are nothing more than functions that have access to the state as well as incoming actions that are dispatched from your UI components. You can think of them as functions that take a stream of actions and return a stream of actions—actions in, actions out. Say, for instance, that you wanted to create an epic to handle persisting every transaction to local storage as withdrawal (or deposit) actions are dispatched. For this, you can create a `transactionLogEpic()` function, as shown in the next listing.

Listing 10.6 Plugging into the middleware

```js
const txDb = new PouchDB('transactions');
class Transaction {//*1
    constructor(account, amount, balance, timestamp) {
        this.account = account;
        this.amount = amount;
        this.balance = balance;
        this.timestamp = timestamp;
    }
}
function transactionLogEpic(action$, store) {//*2
    return action$.ofType('WITHDRAW', 'DEPOSIT')//*3
        .timestamp()
        .map(obj => ({ ...obj.value, timestamp: obj.timestamp }))
        .map(action => ({
            ...action,
            balance: store.getState()[action.account] - action.amount//*4
        }))
        .map(datedAction => (
            new Transaction(
                datedAction.account,
                datedAction.amount,
                datedAction.balance,
                datedAction.timestamp
            )
        ))
        .mergeMap(datedTx =>
            Rx.Observable.fromPromise(txDb.post(datedTx))
                .mapTo({ payload: { ...datedTx }, type: 'LOG' })
                .catch(() =>//*5
                    Rx.Observable.of({ type: 'LOG', payload: 'TX WRITE FAILURE!' })
                )
        );
}
```

1. Slightly simplified version of the Transaction class from chapter 6
2. Epic middleware function
3. Convenient new operator implemented to filter incoming actions based on type (analogous to how reducers work on `action.type`). We'll show it in the next listing.
4. Gets a snapshot of the current state of the store and updates the targeted account
5. Dispatches instead an error action to the store to signal to the user that an error has occurred

---

##### ES 7 Object.assign() 

For most of this book, we've been strictly sticking to ES 6 (ECMAScript 6) syntax. For this chapter, for the purposes of both brevity and popularity, we'll be using ES 7 Object.assign syntax. This syntax appears as `{...oldState, prop: 'VALUE'}`, where this block would normally be written `Object.assign({}, oldState, {prop: 'VALUE'})`. As of this writing, most browsers do not yet support this syntax, so if you wish to use it in your project, you'll need to use a transpiler like Babel (https://babeljs.io/).

---

With this change, your middleware will intercept any action of type WITHDRAW or DEPOSIT and create the proper transaction to store it. Functions like the one in listing 10.6 are the foundation of your observable-based middleware component. In order to make these functions come together, you need one more piece that you'll build yourself. You need to merge the resulting streams so that you can feed all the results into the Redux store. Consider the wiring where you define several such functions and add them to an array. Because these are functions that create new observables, we've taken to naming them "factories." You'll add your only epic function now; at the end of this chapter, we'll show you how to inject additional functionality by adding another epic to this array:

```js
const epics = [
  transactionLogEpic,
  /* Add more epics for more functionality */
];
```

Just as you're used to switching on action types in reducers, you'll see that it's typical of the middleware to selectively process certain actions. For this, it's useful to extend the Observable type with an additional operator, `ofType()`. As you implement more epic functions, bringing this concept along as a first-class citizen will make your code much more succinct. The next listing shows how you can easily augment the `Observable` prototype with this new operator.

Listing 10.7 Implementing custom ofType operator

```js
Rx.Observable.prototype.ofType = function (...types) {
    return this.filter(({ type }) => {
        const len = types.length;
        switch (len) {
            case 0:
                throw new Error('Must specify at least one type! ');
            case 1:
                return type === types[0];
            default:
                return types.indexOf(type) > -1;
        }
    });
}
```

Now that you understand what epics are, you need to connect them to Redux so that the processing of dispatching an action begins to flow through the observable sequence. This connective tissue is implemented entirely with RxJS using `Subject`s.

A `Subject` is a key ingredient for building your middleware layer and dispatching actions that flow through Redux from React components. As far as your banking application is concerned, it's recommended to reduce subjects to the smallest possible scope to avoid abuse. In this case, you'll use them to dispatch and proxy changes along the middleware layer with code like this:

```js
const action$ = new Rx.Subject();
const dispatch = action => action$.next(action);
```

This creates a `Subject` and wraps the `next` function in a lambda, which is what you'll expose to the rest of the application instead of the reference to the Subject itself. The logic is not that simple; you still need to do a bit of work. To make this all portable, you'll wrap the proxy mechanism into a function called `createMiddleware()` that knows how to mimic Redux's interface and glue observables into your middleware layer, as shown in this listing.

Listing 10.8 Building your middleware

```js
function createMiddleware(store, epics) {//*1
    const input$ = new Rx.Subject();//*2
    const actions =
        epics.map(epic =>//*3
            epic(input$, store));//*4
    const combinedActions$ = Rx.Observable
        .merge(...actions)//*5
        .publish();//*6
    combinedActions$.subscribe(input$);//*7
    combinedActions$.subscribe(action => store.dispatch(action));//*8
    const sub = combinedActions$.connect();//*9
    return {
        dispatch: (action) => input$.next(action),//*10
        unsubscribe: () => sub.unsubscribe()//*11
    };
}
```

1. At the top level, the middleware accepts a `store` and a set of `epics`
2. Creates a new private `Subject` instance used to emit actions to both the store and the epics
3. Invokes all the factories and stores their outputs as your middleware streams
4. Each factory takes the actions (`input$`) and state (`store`) to create a new stream.
5. Merges all the resulting streams into a single output
6. Converts that stream into a hot observable, so it's shared
7. Feeds the output of the epic functions (action streams) so that they can get handled by subsequent middleware in the chain
6. Simultaneously sends all events to the store as well, in case it can handle them
7. Connects the stream (makes it hot); this prevents the stream from emitting before both subscribers are subscribed
10. Returns a proxied version of dispatch that invokes `next` on the subject (thus sending actions to the middleware)
11. Puts the user in control of disposing the observable middleware

Listing 10.8 is probably a bit of a mind bender. So here's a quick, isolated example that shows the chain of commands as actions flowing into the middleware and out to the reducers:

```js
function simpleReducer(state, action) {
    switch (action.type) {
        case 'LOG':
            return { ...state, messages: [...action.payload, 'in Redux!'] };
        default:
            return state;
    }
}
const store = createStreamFromStore(
    createStore(simpleReducer, { messages: [] }));
const observableStore = createStreamFromStore(store);
observableStore.subscribe(({ messages }) => console.log(messages.join('=>')));

function simpleEpic(action$, store) {
    return action$.ofType('LOG')
        .delay(1000)
        .map(action => ({ ...action, payload: [...action.payload, 'in Rx!'] }));
}
const disposableDispatcher = createMiddleware(store, [simpleEpic]);
disposableDispatcher.dispatch({
    type: 'LOG',
    payload: ['Hello']
});
```

This code snippet shows a simple epic and a simple reducer. It's meant to illustrate the chain of commands. When the action is dispatched, it first reaches the middleware, and then the reducer. The middleware epic pipes the action through the observable and defers propagating it by a second. Afterward, you process the action and append "in Rx!" to the payload. The action finally makes it to the reducers, where it's modified once more to append "in Redux!" Hence, the output would look like the following:

```js
(...after 1 second) 
Hello=>in Rx!=>in Redux!
```

Figure 10.14 illustrates the sequence of actions in this data flow roller coaster. 

Figure 10.14 This diagram shows the flow of actions within `createMiddleware()`. The `input$` subject is in charge of injecting the world of Redux into RxJS. After the action is wrapped in a `Subject` observable, it flows through each of the epics in the array in order. Finally, the actions resulting from executing this series of epics are dispatched further into the Redux reducers.

The loopback logic is only one part of the code, though; in the second half of listing 10.8, you're also subscribing to the combined output of actions and dispatching all output actions to the store. In effect, this means that an action is evaluated at two points: first, by the epics, to determine if it can be further processed; and second, by the reducer itself, to see if the store must be updated and a new state emitted. To optimize this process, you utilized the publish/connect operators, which were discussed in chapter 8. Recall that by publishing the stream, you share it between all subscribers so that new pipelines aren't created for each subscriber. Finally, you `connect()` the parent stream so that the two subscribers (the `Subject` and the `store`) can begin receiving events when they're emitted. You allow the user to shut down the system by exposing the unsubscribe logic to the user through a method. This allows the user of your library to instigate a controlled shutdown of the application and switch off the streams when they're finished with them.

The result is an interface that more or less resembles that of the original store but that transparently handles asynchronous actions without changes to the Redux store. This is important, as you'll see in the next section, because of how the middleware will fit into the overall system. Now that you finally have all of your components and have seen how you might use them, let's hook them up so that your banking application can handle user transactions. You can do this in any order; remember, your components are all independent of each other.

## 10.6 Bringing it all home

So far, you've learned how to implement all the pieces. Now, you're ready to put everything together to complete your simple banking app that's ready to receive withdrawal or deposit actions. You start with a default value of $100 in each account. You add the transaction epic (to insert all transactions into the database), which then gets added to the middleware via `createMiddleware()`. Your store component knows only about a single reducer at the moment, called `updateAccounts()`. Finally, you create the parent React component that renders the UI into the document body. You make it capable of dispatching actions through the middleware through its `props` object. The next listing shows all of this wired up.

Listing 10.9 Building the application

```js
const Balances = React.createClass({/**/ });//*1
const accounts = {//*2
    checking: 100, savings: 100
};
const epics = [//*3
    transactionLogEpic
    /*  add more epics here */
];
const store = createStore(updateAccounts, accounts);//*4
const state$ = createStreamFromStore(store);//*5
const middleware = createMiddleware(store, epics);//*6

ReactDOM.render(//*7
    React.createElement(Balances,
        { appState$: state$, dispatch: middleware.dispatch }),
    document.getElementById('root')
);
```

1. Declares the specification for the `Balances` component
2. Creates an initial state for the application to start in
3. Declares the middleware components that will be invoked
4. Creates an instance of a Redux store to house state changes
5. Uses the method defined earlier to convert the store into an Rx Stream (you can do this because the store is a subscribable object)
6. Combines the store with the middleware to build the application
7. Creates an instance of the `Balances` component powered by the application

There are several major advantages to organizing your code as you have. For one, you've virtually removed coupling between components. You have streams, reducers, and React components, all tied together loosely by the types of events they create and consume. This means that you can add more components to your application unobtrusively. Note as well that this is superior to a standard event bus, typical of this kind of architecture, because the single direction that events follow means that you avoid race conditions. Further, the compartmentalization of the handlers and reducers means that you can test each of them in isolation. Finally, by using immutable state throughout the application, you prevent new features from unexpectedly creating changes to other components in the system and polluting your system-wide state, which is always problematic for JavaScript developers.

Lastly, to demonstrate how easy the task of adding new features is, you can add another quick feature. For instance, consider a task to calculate interest payments periodically. These payments would be some fraction of the overall balance that's applied over a fixed period of time. How would you add such a feature to your system?

The first question to ask is, where does this logic actually reside? In this case, you could think of automated transactions just like any user transaction that you've seen so far. But with interest, there's no user interaction. Remember as well that your transaction management is done without knowing who initiated a transaction. This means that you'll likely need to create a new handler:

```js
const computeInterest = p => 0.1 / 365 * p;
function interestEpic(action$, store) {
    return Rx.Observable.interval(15000)
        .map(() => store.getState())
        .map(({ savings }) => ({
            type: 'DEPOSIT',
            account: 'savings', amount: computeInterest(savings)
        }))
}
```

Then, you need to add that handler to your initial stream list:

```js
const epics = [
  transactionLogEpic,
  interestEpic
];
```

Voila! You now have new functionality enabling interest payments. We should point out as well that at this point, you're dealing almost exclusively with streams, which is exactly what you want. What you've seen here is merely a small sample of what you could do with this pattern. We invite you to also check out and run the sample application that was built using this architecture (https://github.com/RxJSInAction/rxjs-inaction). We've included several more examples with varying degrees of complexity. 

## 10.7 Parting words

This journey through RxJS has taken you to some new places: from the fundamentals of reactive programming, all the way to a full-fledged web application. Along the way, you've explored how observables are created and destroyed. You looked at how you can merge and split observables, injecting or extracting the information you need. You experimented with notions of time, which surround and interweave with observables, and you used that understanding to build purer, more testable functions. As we said at the outset, this book wasn't meant as a definitive guide to all things RxJS; instead, by focusing on a variety of topics and how they pertain to your real applications, we've hopefully given you both a taste and a hunger for more, because the lessons here are merely the beginning of what you can do with RxJS. Also, remember that the principles we covered in the previous chapters are not confined to JavaScript. You can utilize them across the stack in a variety of languages. Finally, we hope that this book has encouraged you not just to use RxJS in your projects (and we really do hope you do!) but also to examine your code with a more critical eye toward many of the concepts covered in the book, namely, purity, immutability, composability, testability, and laziness. These are valuable ideas that can be applied even without fancy frameworks (though they do tend to make it easier). 

## 10.8 Summary

* Understanding how data is transformed and moved will inform decisions on how to include RxJS in your project.
* Keep events moving in a single direction by looping streams in order to create complex UI interactions that are easy to reason about.
* Manage state immutably and keep all components separate. This will ensure a clear separation of concerns that will allow you to scale your architecture to support new features without linearly increasing the complexity of your application.
* You can use Subjects to implement advanced middleware or stream-proxying solutions. While powerful, Subjects can be hard to troubleshoot given that they can act as both observables and observers. We recommend you keep Subjects to a minimum and well encapsulated.
* You can use RxJS to create middleware that handles asynchronous data flows so that actions dispatched from the UI can flow through an observable pipeline to be translated into a separate action that flows out of an epic.
* RxJS in an intricate part of the Redux/React architecture, which we call 3R. 

