# Chapter 6 Events

~~It was the best of times; it was the worst of times. It was the age of Netscape; it was the age of Internet Explorer. The new event handlers before us made it the spring of hope. The fact that browsers implemented event handling differently left us in a winter of despair. But in recent years, the sun has shone clear and bright, and the event-handling API has standardized across browsers (most aspects of the API, anyway).~~

~~The ultimate goal of writing usable JavaScript code has always been to have a web page that will work for the users, no matter what browser they use or what platform they are on. For too long, this has meant writing event-handling code that managed two different event-handling models. But with the advent of modern browsers, we developers never have to worry about that again.~~

~~The concept of events in JavaScript has advanced through the years to the reliable, usable plateau where we now stand. Once Internet Explorer implemented the W3C model for event handling in version 8, we could stop writing libraries for managing differences between browsers and instead focus on doing interesting and amazing things with events. Eventually, this leads us toward the powerful Model-View-Controller (MVC) model for JavaScript, which we will discuss in a later chapter.~~

上面的段落是抒情文。

In this chapter we will start with an introduction to how events work in JavaScript. Following this theory with a practical application, we will look at how to bind events to elements. Then we will examine the information the event model provides and how you can best control it. Of course, we also need to cover the types of events available to us. We conclude with event delegation and a few bits of advice about events and best practices.

## Introduction to JavaScript Events

If you look at the core of any JavaScript code, you'll see that events are the glue that holds everything together. Whether using a full MVC-based single-page application or simply using JavaScript to add some functionality to a page or two, event handlers are how the user communicates with our code. Our data will be bound in JavaScript, probably as object literals. We will represent this data in the DOM, using it as our view. Events, raised from the DOM, handled by JavaScript code, capture user interactions and guide the flow of our application. The combination of using the DOM and JavaScript events is the fundamental union that makes all modern web applications possible.

### The Stack, the Queue, and the Event Loop

In many programming languages, including JavaScript, there are metaphors which describe the flow of control, elements in memory, and planning for what happens next. The code we run, whether from the global context, directly as a function, or as a function called from (or within!) another function, is known as the **stack**. If you are running a function `foo`, which calls a function `bar`, then the stack is three `frames` deep (global, foo, and then bar). What's up after this code runs? That's the province of the **queue**, which manages the next set of code to run after the current stack is resolved. Any time the stack empties, it goes to the queue and picks up a new bit of code to run. These are the elements that are critical to our understanding of events.

There is a third element, though: the **heap**. This is where variables and functions and other named objects live. When JavaScript needs to access an object, a function, or a variable, it goes to the heap to get access to the information. For us, the heap is less relevant, as it does not play as big a role in event handling as the stack and the queue do.

How do the stack and the queue factor into event handling? To answer this question, we need to introduce the **event loop**. This is a collaboration between two threads in your browser: the **event-tracking thread**, and the **JavaScript thread**.

 ■ Note remember that, except for web workers, Javascript is single-threaded.

These threads work together to capture user events and then sort them according to the events for which we have registered events handlers. This process is collectively known as the **event loop**. Each time it runs, user events are checked to see if there are event handlers registered against them. If not, then nothing happens. If there are event handlers, the loop pushes them onto the top of JavaScript's queue, so that the handler is executed at JavaScript's earliest convenience.

And there's the rub. The queue manages the notion of “earliest convenience.” Generally, this means after the current stack has been resolved. This may give event handling an asynchronous feel, particularly if the stack is many frames deep or contains long-running code. Events are allowed to jump to the head of the queue, but they may not interrupt the stack. Most of the time, the distinction is immaterial to developers, because the duration between when an event fires, a stack frame resolves, and event-handling code runs may not be perceptible in human terms. Nonetheless, it is important for us to understand that the event loop only jumps events to the front of the line; it does not push currently running code out of the way.

We now understand how the browser, the queue, and the stack work together to determine when an event handler will run. Soon, we will look at the mechanics of binding events to event handlers. But there is one architectural issue we need to cover first. Consider this: If you click a link in a list item in an unordered list in a paragraph in a div in the body of your HTML document, which of those elements handles that event? Could more than one element handle the event? If so, which element gets the event first? To answer this question, we'll need to look at event phases.

### Event Phases

JavaScript events are executed in two phases, called `capturing` and `bubbling`. What this means is that when an event is fired from an element (for example, the user clicking a link, causing the click event to fire), the elements that are allowed to handle it, and in what order, vary. You can see an example of the execution order in Figure 6-1. It shows which event handlers are fired and in what order, whenever a user clicks the first `<a>` element on the page.

```html
<body>                       ↓ capturing
  <div id="body">            ↓
    <ul class="links">       ↓
      <li>                   ↓
        <a href="/">Home</a> + bubbling
      </li>                  ↓
    </ul>                    ↓
  </div>                     ↓
</body>                      ↓
```

Figure 6-1. The two phases of event handling

Looking at this simple example of someone clicking a link, you can see the order of execution for an event. Imagine that the user clicked the `<a>` element; the click handler for the document is fired first, then the `<body>`'s handler, then the `<div>`'s handler, and so on, down to the `<a>` element, a cycle called the capturing phase. Once that finishes, it moves back up the tree again, and the `<li>`, `<ul>`, `<div>`, `<body>`, and document event handlers are all fired, in that order.

~~There are very specific historical reasons why event handling is built this way. When Netscape introduced event handling, it decided that event capturing should be used. When Internet Explorer caught up with its own version of event handling, it went with event bubbling. It was the time of the browser wars, and diametrically opposed architectural choices like this were commonplace. They hampered development of JavaScript for years, as programmers had to waste time maintaining libraries that normalized event handling (and some of the DOM, and Ajax, and a few other things!).~~

历史回顾：微软与网景的战争。

The good news is that we now live in the future. Modern browsers allow users to choose at which phase to capture events. In fact, you can assign event handlers at both phases, if you so choose. It's a brave new world. Regardless of at which phase you bind events, two things should be immediately apparent. First, we discussed the idea that you might click on an anchor tag within a list item. Shouldn't that send you off to wherever the href attribute for the link points? Maybe there is some way to override that behavior. Additionally, consider the general premise of event phases: whether capturing or bubbling, an event is communicated through the DOM hierarchy. What if we do not want that event to be communicated? Can we prevent an event from being passed up (or down) the hierarchy?

But we are getting ahead of ourselves. We have not even discussed how to bind event listeners yet! Let's take care of that right now.

## Binding Event Listeners

The best way to bind event handlers to elements has been a constantly evolving quest in JavaScript. It began with browsers forcing users to write their event-handler code inline, in the HTML document. First efforts are thought of as drafts or alpha code for a reason! It turns out that, later on, when we want to follow established best practices like separating logic from presentation, using inline event handlers is, let's say suboptimal. OK, it's severely problematic. Try to imagine managing a codebase where half of your critical paths depend on code embedded in your presentation layer. Not something that the professional JavaScript programmer wants to do! Thankfully, that technique has been obviated by evolving browser APIs, as well as evolving standards of best practices.

When Netscape and Internet Explorer were actively competing with each other, they each developed separate, but very similar, event registration models. In the end, Netscape's model was modified to become a W3C standard, and Internet Explorer's stayed the same. Until Internet Explorer 9, that is, when Microsoft finally caved and implemented what is generally called W3C event handling. In fact, it went further and deprecated its older API for event handling. This was a boon for developers, as now we no longer had to write and maintain libraries dealing with the quibbling difficulties between browsers.

Today, there are two ways of reliably registering events. The traditional method is an offshoot of the old inline way of attaching event handlers, but it's reliable and works consistently, even on older browsers. The other method is to use the W3C standard for registering events. We will, of course, look at both, as you are likely to encounter both.

### Traditional Binding

The traditional way of binding events is the simplest way of binding event handlers. This takes advantage of the fact that event handlers are properties of DOM elements. To use this method, you attach a function as a property to the DOM element that you wish to watch. Retrieve an element with `document.getElementById` (or any of the other element-retrieving functions we discussed in Chapter 5). Let's assume that you want to watch for click events. Simply assign a function to the `onclick` property of the retrieved element. Done! For the examples in this chapter, we will use a standard HTML page with many targetable elements.

The content for the page is presented in Listing 6-1.

Listing 6-1. Example HTML Code Used for Event Handling

```html
<!DOCTYPE html>
<html>
<head lang="en">
  <meta charset="UTF-8">
  <title>Event Handling</title>
  <link rel="stylesheet" href="school.css" />
</head>
<body>
  <div id="main">
    <nav id="navbar">
      <ul>
        <li>
          Students
          <ul>
            <li id="Academics">Academics</li>
            <li id="Athletics">Athletics</li>
            <li id="Extracurriculars">Extracurriculars</li>
          </ul>
        </li>
        <li>
          Faculty
          <ul>
            <li id="Frank Walsh">Frank Walsh</li>
            <li id="Diane Walsh">Diane Walsh</li>
            <li id="John Mullin">John Mullin</li>
            <li id="Lou Garaventa">Lou Garaventa</li>
            <li id="Dan Tully">Dan Tully</li>
            <li id="Emily Su">Emily Su</li>
          </ul>
        </li>
      </ul>
    </nav>
    <div id="welcome">
      <h1>Welcome to the School of JavaScript</h1>
      <h3 id="welcome-header">Click here for a welcome message!</h3>
      <p id="welcome-content">
        Welcome to the School of JavaScript. Here, you will find many
        <a href="/examples" id="examples-link">examples</a> of JavaScript,
        taught by our most esteemed <a href="/faculty">faculty</a>. <span id="disclaimer">
          Please note that these are only examples, and are not
          necessarily <a href="/production-ready">production-ready code</a>.
        </span>
      </p>
    </div>
    <hr />
    <div id="form-container">
      <h2>Contact Form</h2>
      <p>
        Thank you for your interest in the School of JavaScript. Please fill out the form
        below so we can send you even more materials!
      </p>
      <form id="main-form">
        <ul>
          <li><label for="firstName">First Name: </label><input id="firstName" type="text" /></li>
          <li><label for="lastName">Last Name: </label><input id="lastName" type="text" /></li>
          <li><label for="city">City: </label><input id="city" type="text" /></li>
          <li><label for="state">State: </label><input id="state" type="text" /></li>
          <li><label for="postCode">Postal Code: </label><input id="postCode" type="text" /></li>
          <li>
            <label for="comments">Comments: </label><br />
            <textarea name="" id="comments" cols="30" rows="10"></textarea>
          </li>
          <li><input type="submit" /> <input type="reset" /></li>
        </ul>
      </form>
    </div>
  </div>
</body>
</html>
```

As you can see, there are many elements in the front page for our imaginary School of JavaScript. The navbar will eventually have appropriate event handling to function as a menu, we will add event handling to the form for simple validation (with more complex validations coming in Chapter 8), and we also plan to have a bit of interactivity in the welcome message.

For now, let's do something simple. When we click into the `firstName` field, let's record that on the console, and then set a yellow background for our element. Obviously, we are planning to do more soon, but baby steps first! Let's bind this event with traditional event handling (Listing 6-2).

Listing 6-2.  Binding a Click Event the Traditional Way

```js
// Retrieve the firstName element
const firstName = document.getElementById('firstName');

// Attach the event handler
firstName.onclick = function() {
    console.log('You clicked in the first name field!');
    firstName.style.background = 'yellow';
};
```

Terrific! It works. But it lacks a certain something. That something is flexibility. At this rate, we would have to write a separate event-handling function for each of the form fields. Tedious! Could there be a way to get a reference to the element that fired the event?

In fact, there are two ways! The first, and most straightforward, is to provide an argument in your event-handling function, as shown in Listing 6-3. This argument is the event object, which contains information about the event that just fired. We will be looking at the event object in greater detail shortly. For now, know that the `target` property of that event object refers to the DOM element that emitted the event in the first place.

Listing 6-3. Event Binding with an Argument

```js
// Retrieve the firstName element
const firstName = document.getElementById('firstName');

// Attach the event handler
firstName.onclick = function(e) {
    console.log('You clicked in the ' + e.target.id + ' field!');
    e.target.style.background = 'yellow';
};
```

Since `e.target` points to the `firstName` field, and is, in fact, a reference to the DOM element for the firstName field, we can check its `id` property to see what field we clicked in. More importantly, we can change its `style` property as well! This means we can broaden this event handler to work on just about any of the text fields in the form.

There is an alternative to using the event object explicitly. We could also use the `this` keyword in the function, as shown in Listing 6-4. In the context of an event-handling function, `this` refers to the emitter of the event. Put another way, `event.target` and `this` are synonymous, or, at least, they point to the same thing.

Listing 6-4. Event Binding Using the this Keyword

```js
// Retrieve the firstName element
const firstName = document.getElementById('firstName');

// Attach the event handler
firstName.onclick = function() {
    console.log('You clicked in the ' + this.id + ' field!');
    this.style.background = 'yellow';
};
```

Which should you use? The event object gives you all of the information you need, while the `this` object is somewhat limited, as it only points to the DOM element that emitted the event. There is no cost in using one over the other, so, in general, we recommend preferring the **event object**, as you will always have all the details of the event immediately available. However, there are some cases where the `this` object is still useful. The target always refers to the closest element emitting the event. Look at the `<div>` with the ID welcome back in Listing 6-1. Let's say we added a mouseover event handler to change the background color when we hovered over the element, and a mouseout event handler to change the background color back when the mouse leaves the `<div>`. If you make the style change on `e.target`, the event will fire for each of the subelements (welcome-header, welcome-content, and more)! On the other hand, if you make the style change on `this`, the change is made only on the welcome `<div>`. When we discuss event delegation, we will cover this difference in greater detail.

#### Advantages of Traditional Binding

Traditional binding has the following advantages:

• The biggest advantage of using the traditional method is that it is incredibly simple and consistent, in that you're pretty much guaranteed that it will work the same no matter what browser you use it in.

• When handling an event, the `this` keyword refers to the current element, which can be useful (as demonstrated in Listing 6-4).

#### Disadvantages of Traditional Binding

It also has some disadvantages, however:

• The traditional method allows no control over event capturing or bubbling. All events bubble, and there is no possibility of changing to event capturing.

• It's only possible to bind one event handler to an element at a time. This has the potential to cause confusing results when working with the popular `window.onload` property (effectively overwriting other pieces of code that have used the same method of binding events). An example of this problem is shown in Listing 6-5, where an event handler overwrites an earlier event handler.

Listing 6-5. Event Handlers Overwriting Each Other

```js
// Bind your initial load handler
window.onload = myFirstHandler;

// somewhere, in another library that you've included,
// your first handler is overwritten
// only 'mySecondHandler' is called when the page finishes loading
window.onload = mySecondHandler;
```

• The event object argument is not available in Internet Explorer 8 and older. Instead, you would have to use `window.event`.

Knowing that it's possible to blindly override other events, you should probably opt to use the traditional means of event binding only in simple situations, where you can trust all the other code that is running alongside yours. One way to get around this troublesome mess, however, is to use the W3C event-binding method implemented by modern browsers.

### DOM Binding: W3C

The W3C's method of binding event handlers to DOM elements is the only truly standardized means of doing so. With that in mind, every modern browser supports this way of attaching events. Internet Explorer 8 and older do not, but old versions of Internet Explorer are hardly modern browsers. If you must design for those, consider using traditional binding.

The code for attaching a new handler function is simple. It exists as a function available on every DOM element. The function is named `addEventListener` and takes three parameters: the name of the event (such as click; note the lack of the prefix on), the function that will handle the event, and a Boolean flag to enable or disable event capturing. An example of `addEventListener` in use is shown in Listing 6-6.

参考：https://wiki.developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener

Listing 6-6. Sample Code That Uses the W3C Way of Binding Event Handlers

```js
// Retrieve the firstName element
const firstName = document.getElementById( 'firstName' );

// Attach the event handler
firstName.addEventListener( 'click', function ( e ) {
  console.log( 'You clicked in the ' + e.target.id + ' field!' );
  e.target.style.background = 'yellow';
} );
```

Note that in this example, we are not passing a third argument to `addEventListener`. In this case, the third argument defaults to false, meaning that event bubbling will be used. If we had wanted to use event capturing, we could have passed a true value explicitly.

#### Advantages of W3C Binding

The advantages to the W3C event-binding method are the following:

• This method supports both the capturing and bubbling phases of event handling. The event phase is toggled by setting the last parameter of `addEventListener` to false (the default, for bubbling) or true (for capturing).

• Inside the event-handler function, the `this` keyword refers to the current element, as it did in traditional event handling.

• The event object is always available in the first argument of the handling function.

• You can bind as many events to an element as you wish, with no overwriting previously bound handlers. Handlers are stacked internally by JavaScript and run in the order that they were registered.

#### Disadvantage of W3C Binding

The W3C event-binding method has only one disadvantage:

• It does not work in Internet Explorer 8 and older. IE uses `attachEvent` with a similar syntax.

### Unbinding Events

Now that we have bound events, what if we want to unbind events? Perhaps that button we tied a click event handler to is now disabled. Or we no longer need to highlight that div when hovering over it. Disconnecting an event and its handler is relatively simple.

For traditional event handling, simply assign an empty string or null to the event handler, as shown here:

```js
document.getElementById('welcome-content').onclick = null;
```

Not too difficult, right?

The situation with W3C event handling is somewhat more complex. The relevant function is `removeEventListener`. Its three arguments are the same: the type of event to remove, the associated handler, and a true/false value for capture or bubble mode. There is a catch, though. First and foremost, the function must be a reference to the same function that was assigned with `addEventListener`. Not just the same lines of code, but the same reference. So if you assigned an anonymous, in-line function with `addEventListener`, you cannot remove it.


 ■ Tip You should always use a named function for an event handler if you think you might need to remove that handler later on.

读书笔记：事件处理函数定义在顶层，有名称。不要嵌套在函数内，作为lambda表达式，防止误引用局部变量。

In a similar vein, if you set the third argument when you originally invoked `addEventListener`, you must set it again in `removeEventListener`. If you leave the argument off, or give it the wrong value, `removeEventListener` will silently fail. Listing 6-7 has an example of unbinding an event handler.

Listing 6-7. Unbinding an Event Handler

```js
// Assume we have two buttons 'foo' and 'bar'
const foo = document.getElementById( 'foo' );
const bar = document.getElementById( 'bar' );

// When we click on foo, we want to log to the console "Clicked on foo!"
function fooHandler() {
  console.log( 'Clicked on the foo button!' );
}

foo.addEventListener( 'click', fooHandler );

// When we click on bar, we want to _remove_ the event handler for foo.
function barHandler() {
  console.log( 'Removing event handler for foo....' );
  foo.removeEventListener( 'click', fooHandler );
}

bar.addEventListener( 'click', barHandler );
```


## Common Event Features

JavaScript events have a number of relatively consistent features that give you more power and control when developing. The simplest and oldest concept is that of the event object, which provides you with a set of metadata and contextual functions so you can deal with things such as mouse events and key presses. Additionally, there are functions that can be used to modify the normal capture/bubbling flow of an event. Learning these features inside and out can make your life much simpler.

### The Event Object

One standard feature of event handlers is some way to access an event object, which contains contextual information about the current event. This object serves as a very valuable resource for certain events. For example, when handling key presses you can access the `keyCode` property of the object to get the specific key that is pressed. There are some subtle differences between event objects, but we will address those later in the chapter. For now, let us address two dangling issues: event propagation and default behavior.

### Canceling Event Bubbling

You know how event capturing/bubbling works, so let's explore how you can take control of it. An important point brought up in the previous example is that if you want an event to occur only on its target and not on its parent elements, you have no way to stop it. Stopping the flow of an event bubble would cause an occurrence similar to what is shown in Figure 6-2, where the result of an event is captured by the first `<a>` element and the subsequent bubbling is canceled.

```html
<body>                       ↓ capturing
  <div id="body">            ↓
    <ul class="links">       ↓
      <li>                   ↓
        <a href="/">Home</a> + stop
      </li>
    </ul>
  </div>
</body>
```

Figure 6-2. The result of an event being captured by the first `<a>` element

Stopping the bubbling (or capturing) of an event can prove immensely useful in complex applications. And it's simple to implement. Invoke the event object's `stopPropagation` method to prevent the event from traversing further up (or down) the hierarchy. Listing 6-8 shows an example.

```js
document.getElementById( 'disclaimer' ).addEventListener( 'click', function ( e ) {

  // When clicking on the disclaimer, highlight it by making it bold
  e.target.style.fontWeight = 'bold';

  // The parent element wants to hide itself if this element is clicked on. We need to prevent that behavior
  e.stopPropagation();
} );

document.getElementById( 'welcome-content' ).addEventListener( 'click', function ( e ) {
  e.target.style.visibility = 'hidden';
} );
```

Listing 6-9 shows a brief snippet that adds a red border around the element that the user is hovering over. You do this by adding a mouseover and a mouseout event handler to every DOM element. If you don't stop the event bubbling, every time the mouse is moved over an element, the element and all of its parent elements will have the red border, which isn't what you want.

Listing 6-9. Using `stopPropagation` to Keep All the Elements from Changing Color

```js
// Event handling functions
function mouseOverHandler( e ) {
  e.target.style.border = '1px solid red';
  e.stopPropagation();
}

function mouseOutHandler( e ) {
  this.style.border = '0px';
  e.stopPropagation();
}

// Locate, and traverse, all the elements in the DOM
const all = document.getElementsByTagName( '*' );
for ( let i = 0; i < all.length; i++ ) {

  // Watch for when the user moves the mouse over the element
  // and add a red border around the element
  all[i].addEventListener( 'mouseover', mouseOverHandler );

  // Watch for when the user moves back out of the element
  // and remove the border that we added
  all[i].addEventListener( 'mouseout', mouseOutHandler );

}
```

With the ability to stop the event bubbling, you now have complete control over which elements can see and handle an event. This is a fundamental tool necessary for exploring the development of dynamic web applications. The final aspect is to cancel the default action of the browser, allowing you to completely override what the browser does and implement new functionality instead.

### Overriding the Browser's Default Action

For most events that take place, the browser has some default action that will always occur. For example, clicking an `<a>` element will take you to its associated web page; this is a default action in the browser. This action will always occur after both the capturing and the bubbling event phases, as shown in Figure 6-3, which illustrates the results of a user clicking an `<a>` element in a web page. The event begins by traveling through the DOM in both a capturing and bubbling phase (as discussed previously). However, once the event has finished traversing, the browser attempts to execute the default action for that event and element. In this case, it's visiting the / web page.

```html
<body>                       ↓ capturing
  <div id="body">            ↓
    <ul class="links">       ↓
      <li>                   ↓
        <a href="/">Home</a> + bubbling
      </li>                  ↓
    </ul>                    ↓
  </div>                     ↓
</body>                      ↓ Default (window.location '/')
```

Figure 6-3. The full life cycle of an event

Default actions can be summarized as anything the browser does that you did not explicitly tell it to do. Here's a sampling of the different types of default actions that occur, and on what events:

• Clicking an `<a>` element will redirect you to a URL provided in its `href` attribute.

• Using your keyboard and pressing Ctrl+S, the browser will attempt to save a physical representation of the site.

• Submitting an HTML `<form>` will submit the query data to the specified URL and redirect the browser to that location.

• Moving your mouse over an `<img>` with an `alt` or a `title` attribute (depending on the browser) will cause a tool tip to appear, providing the value of the attribute.

All of the previous actions are executed by the browser even if you stop the event bubbling or if you have no event handler bound at all. This can lead to significant problems in your scripts. What if you want your submitted forms to behave differently? Or what if you want `<a>` elements to behave differently than their intended purpose? Because canceling event bubbling isn't enough to prevent the default action, you need some specific code to handle that directly. The W3C event handling API provides this functionality with the `preventDefault` method of the event object (Listing 6-10). With many browsers you can choose to simply return false from your event handler as an alternative, and you may see this behavior coded in some examples and libraries. Using `preventDefault` is preferred, though, as it is self-documenting—unlike the somewhat obscure technique of occasionally returning false from an event handler.

Listing 6-10. A Generic Function for Preventing the Default Browser Action from Occurring

```js
document.getElementById('examples-link').addEventListener('click', function(e) {
  e.preventDefault();
  console.log("examples-link clicked");
});
```

Using the `preventDefault` function, you can now stop any default action presented by the browser. For example, this allows you to take advantage of mouseover events for a link, without worrying about the user accidentally clicking on the link and sending the browser elsewhere. And you can override the default behavior of showing where the link goes in the status bar. Or consider a Submit button that you want to use to kick off form validation. You can now hold off on submitting the form (the default behavior) if that validation fails.

### Event Delegation

We have almost all the tools in place to manipulate event handlers. The one lingering problem is a matter of technique. Presume that we have an unordered list with 20 items in it. We want to add an event handler for each list item. More accurately, we want to be able to handle clicks from each list item differently. We could grab all of the elements with `document.querySelectorAll`, iterate over the results, and attach individual event handlers. This is inefficient both as a process and in the browser. We are setting up 20 event handlers (even if they all point to the same handling function) when we could set up just one.

All of the list items are contained within an unordered list tag, so why not take advantage of the fact that we can capture click events at the `<ul>` level? The only thing we need is some way to differentiate between the various list items. Back in the section on traditional event binding, when we discussed the this object, we noted that this refers to the element where the event is captured, while `event.target` refers to the element that actually emits the event in question. Clearly, we could use a combination of `this` and `event.target`. But the event-handling specification provides the `event.currentTarget` property to solve this problem.

In our list item scenario, we attach a click event handler to the unordered list. In the event handler, the `<ul>` is `event.currentTarget`. Each list item will be the `event.target` property. Therefore, we can check `event.target` to see which list item was clicked and dispatch to the appropriate function. Listing 6-11 shows an example of event delegation in action.

Listing 6-11. Event Delegation

```js
function clickHandler(e) {
  console.log( 'Handled at ' + e.currentTarget.id );
  console.log( 'Emitted by ' + e.target.id );
}

const navbar = document.getElementById('navbar');
navbar.addEventListener( 'click', clickHandler );
```

The `clickHandler` function handles events at the `<nav>` level, but it receives events emitted from a variety of list items under the `<nav>` element.

读书笔记：点击事件会在被添加侦听器的当前目标的所有后代元素的每一层都触发，应该对`event.target`进行识别，过滤，不触发不该触发的事件。

## The Event Object

The event object is provided, or is available, inside every event-handler function. In general, the properties of the event object cover the details you might want to know about an event: what kind of event it was, where it originated from, what coordinates were clicked, or maybe what keys were pressed. There are some subtle differences between the ways different browsers communicate this information, though.

### General Properties

A number of properties exist on the event object for every type of event being captured. All of these event object properties relate directly to the event itself, and nothing is browser-specific. What follows is a list of all the event object properties with explanations and example code.

#### type

This property contains the name of the event currently being fired (such as `click` or `mouseover`). It can be used to provide a generic event-handler function, which then deterministically executes related code. Listing 6-12 shows an example of using this property to make a handler have different effects depending on the **event type**.

Listing 6-12. Using the `type` Property to Provide Hoverlike Functionality for an Element

```js
function mouseHandler(e){
  // Toggle the background color of the <div>, depending on the
  // type of mouse event that occurred
  this.style.background = (e.type === 'mouseover') ? '#EEE' : '#FFF';
}

// Locate the <div> that we want to hover over
var div = document.getElementById('welcome');

// Bind a single function to both the mouseover and mouseout events
div.addEventListener( 'mouseover', mouseHandler );
div.addEventListener( 'mouseout', mouseHandler );
```

#### target

This property contains a reference to the element that fired to the event. For example, binding a click handler to an `<a>` element would have a `target` property equal to the `<a>` element itself.

触发事件最内层的元素。

#### stopPropagation

The `stopPropagation` method stops the event bubbling (or capturing) process, making the current element the last one to receive the particular event.

#### preventDefault

Calling the `preventDefault` method stops the browser's default action from occurring in all modern W3C-compliant browsers.

### Mouse Properties

Mouse properties exist within the event object only when a mouse-related event is initiated (such as `click`, `mousedown`, `mouseup`, `mouseover`, `mousemove`, `mouseout`, `mouseenter`, and `mouseleave`). At any other time, you can assume that the values being returned do not exist or are not reliably present. This section lists all the properties that exist on the event object during a mouse event.

#### pageX and pageY

These properties contain the x- and y-coordinates of the mouse cursor relative to the absolute upper-left corner of the browser window. They will be the same regardless of scrolling.

#### clientX and clientY

These properties contain the x- and y-coordinates of the mouse cursor relative to the browser window. Therefore, if you have scrolled the document down (or across), the numbers are relative to the edges of the browser window. These numbers change as you scroll through your document.

#### offsetX/offsetY

These properties should contain the x- and y-coordinates of the mouse cursor relative to the event's target element. The `offsetX`/`offsetY` properties work in Chrome and IE, but not in Firefox. Firefox supports `layerX` and `layerY`, but they don't contain the same information. Instead, the `layerX`/`layerY` properties seem to be equivalent to the appropriate `pageX`/`pageY` property.

读书笔记： page是试验属性。offset是试验属性。layer是非标准属性。

#### button

This property, available only on the `click`, `mousedown`, and `mouseup` events, is a number representing the mouse button that's currently being clicked. Left clicks are 0 (zero), middle clicks are 1, right clicks are 2.

#### relatedTarget

This event property contains a reference to the element that the mouse has just left. More often than not, `relatedTarget` is used in situations where you need to use `mouseover`/`mouseout`, but you also need to know where the mouse just was, or where it is going. Listing 6-13 shows a variation on a tree menu (`<ol>` elements containing other `<ol>` elements) in which the subtrees display only the first time the user moves the mouse over the `<li>` subelement.

Listing 6-13. Using the `relatedTarget` Property to Create a Navigable Tree

```js
// When DOMContent is ready, get the references to the elements.
document.addEventListener('DOMContentLoaded', init);
function init(){
  const top = document.getElementById("top");
  const bottom = document.getElementById("bottom");
  top.addEventListener("mouseover", onMouseOver);
  top.addEventListener("mouseout", onMouseOut);
  bottom.addEventListener("mouseover", onMouseOver);
  bottom.addEventListener("mouseout", onMouseOut);
}

function onMouseOut(event) {
  console.log("exited " + event.target.id + " for " + event.relatedTarget.id);
}

function onMouseOver(event) {
  console.log("entered " + event.target.id + " from " + event.relatedTarget.id);
}
```

// Sample HTML:

```html
<style>
div > div {
  height: 128px;
  width: 128px;
}
#top    { background-color: red; }
#bottom { background-color: blue; }
</style>
<title>Untitled Document</title>
</head>
<body>
<div id="outer">
  <div id="top"></div>
  <div id="bottom"></div>
</div>
```

### Keyboard Properties

Keyboard properties generally only exist within the event object when a keyboard-related event is initiated (such as `keydown`, `keyup`, and `keypress`). The exception to this rule is for the `ctrlKey` and `shiftKey` properties, which are available during mouse events (allowing the user to Ctrl+click an element). At any other time, you can assume that the values contained within a property do not exist or are not reliably present.

#### ctrlKey

This property returns a Boolean value representing whether the keyboard Ctrl key is being held down. The `ctrlKey` property is available for both keyboard and mouse events.

#### keyCode

This property contains a number corresponding to the different keys on a keyboard. The availability of certain keys (such as PageUp and Home) can vary, but generally speaking, all other keys work reliably. Table 6-1 is a reference for all of the commonly used keyboard keys and their associated key codes.

Table 6-1. Commonly Used Key Codes

```
Key Key Code
Backspace 8
Tab 9
Enter 13
Space 32
Left arrow 37
Up arrow 38
Right arrow 39
Down arrow 40
0–9 48–57
A–Z 65–90
```

#### shiftKey

This property returns a Boolean value representing whether the keyboard Shift key is being held down. The `shiftKey` property is available for both keyboard and mouse events.

## Types of Events

Common JavaScript events can be grouped into several categories. Probably the most commonly used category is that of mouse interaction, followed closely by keyboard and form events. The following list provides a broad overview of the different classes of events that exist and can be handled in a web application.

- Loading and error events: Events of this class relate to the page itself, observing its load state. They occur when the user first loads the page (the `load` event) and when the user finally leaves the page (the `unload` and `beforeunload` events). Additionally, JavaScript errors are tracked using the `error` event, giving you the ability to handle errors individually.

- UI events: These are used to track when users are interacting with one aspect of a page over another. With these tools you can reliably know when a user has begun input into a form element, for example. The two events used to track this are `focus` and `blur` (for when an object loses focus).

- Mouse events: These fall into two categories: events that track where the mouse is currently located (`mouseover`, `mouseout`), and events that track where the mouse is clicking (`mouseup`, `mousedown`, `click`).

- Keyboard events: These are responsible for tracking when keyboard keys are pressed and within what context—for example, tracking keystrokes inside form elements as opposed to keystrokes that occur within the entire page. As with the mouse, three event types are used to track the keyboard: `keyup`, `keydown`, and `keypress`.

- Form events: These relate directly to interactions that occur only with forms and form input elements. The `submit` event is used to track when a form is submitted; the `change` event watches for user input into an element; and the `select` event fires when a `<select>` element has been updated.

事件类型可以参考：https://wiki.developer.mozilla.org/en-US/docs/Web/Events

### Page Events

All page events deal specifically with the function and status of the entire page. The majority of the event types handle the loading and unloading of a page (whenever a user visits the page and then leaves again).

#### load

The `load` event is fired once the page has completely finished loading; this event includes all images, external JavaScript files, and external CSS files. It's also available on most elements with a `src` attribute (`img`, `script`, `audio`, `video`, and so on). Load events do not bubble.

#### beforeunload

This event is something of an oddity, as it's completely nonstandard but widely supported. It behaves very similarly to the `unload` event, with an important difference. Within your event handler for the `beforeunload` event, if you return a string, that string will be shown in a confirmation message asking users if they wish to leave the current page. If they decline, they will be able to stay on the current page. Dynamic web applications, such as Gmail, use this to prevent users from potentially losing any unsaved data.

#### error

The `error` event is fired every time an error occurs within your JavaScript code. It can serve as a way to capture error messages and display or handle them gracefully. This event handler behaves differently than others, in that instead of passing in an event object, it includes a message explaining the specific error that occurred.

#### resize

Page events: The `resize` event occurs every time the user resizes the browser window. When the user adjusts the size of the browser window, the `resize` event will only fire once the resize is complete, not at every step of the way.

#### scroll

The `scroll` event occurs when the user moves the position of the document within the browser window. This can occur from keyboard presses (such as using the arrow keys, Page Up/Down, or the spacebar) or by using the scrollbar.

#### unload

This event fires whenever the user leaves the current page (for example, by clicking a link, hitting the Back button, or even closing the browser window). Preventing the default action does not work for this event (the next best thing is the `beforeunload` event).

### UI Events

User interface events deal with how the user is interacting with the browser or page elements. The UI events can help you determine what elements on the page the user is currently interacting with and provide them with more context (such as highlighting or help menus).

#### focus

The `focus` event is a way of determining where the page cursor is currently located. By default, the focus is within the entire document; however, whenever a link or a form input element is clicked or tabbed into using the keyboard, it moves to that instead. (An example of this event is shown in Listing 6-14).

#### blur

The `blur` event occurs when the user shifts focus from one element to another (within the context of links, `input` elements, or the page itself). (An example of this event is shown in Listing 6-14).

### Mouse Events

Mouse events occur when the user either moves the mouse pointer or clicks one of the mouse buttons.

#### click

A `click` event occurs when a user presses the left mouse button down on an element (see the `mousedown` event) and releases the mouse button (see the `mouseup` event) on the same element.

#### dblclick

The `dblclick` event occurs after the user has completed two click events in rapid succession. The rate of the double-click depends upon the settings of the operating system.

#### mousedown

The `mousedown` event occurs when the user presses down a mouse button. Unlike the `keydown` event, this event will only fire once while the mouse is down.

#### mouseup

The `mouseup` event occurs when the user releases the pressed mouse button. If the button is released on the same element that the button is pressed on, a `click` event also occurs.

#### mousemove

A `mousemove` event occurs whenever the user moves the mouse pointer at least one pixel on the page. The number of `mousemove` events fired (for a full movement of the mouse) depends on how fast the user is moving the mouse and how quickly the browser can keep up with the updates.

#### mouseover

The `mouseover` event occurs whenever the user moves the mouse into an element from another. To find which element the user has come from, use the `relatedTarget` property. This event is resource-intensive, as it can fire once for each pixel or subelement that it goes over. Prefer `mouseenter`, described shortly.

#### mouseout

The `mouseout` event occurs whenever the user moves the mouse outside an element. This includes moving the mouse from a parent element to a child element (which may seem unintuitive at first). To find which element the user is going to, use the `relatedTarget` property. This event is resource-intensive, as, paired with `mouseover`, it can fire many, many times. Prefer `mouseleave`, described shortly.

#### mouseenter

Functions like `mouseover`, but intelligently pays attention to where it is within an element. Will not fire again until it leaves the element's box.

#### mouseleave

Functions like `mouseout`, but intelligently pays attention to when it leaves an element. Listing 6-14 shows an example of attaching pairs of events to elements to allow for keyboard-accessible (and mouse-accessible) web page use. Whenever the user moves the mouse over a link or uses the keyboard to navigate to it, the link will receive some additional color highlighting.

Listing 6-14. Creating a Hover Effect by Using the mouseover and mouseout Events

```js
// mouseEnter handler
function mouseEnterHandler() {
  this.style.backgroundColor = 'blue';
}

// mouseLeave handler
function mouseLeaveHandler() {
  this.style.backgroundColor = 'white';
}

// Find all the <a> elements, to attach the event handlers to them
const a = document.getElementsByTagName('a');
for ( let i = 0; i < a.length; i++ ) {
  // Attach a mouseover and focus event handler to the <a> element,
  // which changes the <a>s background to blue when the user either
  // mouses over the link, or focuses on it (using the keyboard)
  a[i].addEventListener('mouseenter', mouseEnterHandler);
  a[i].addEventListener('focus', mouseEnterHandler);

  // Attach a mouseout and blur event handler to the <a> element
  // which changes the <a>s background back to its default white
  // when the user moves away from the link
  a[i].addEventListener('mouseleave', mouseLeaveHandler);
  a[i].addEventListener('blur', mouseLeaveHandler);

}
```

### Keyboard Events

Keyboard events handle all instances of a user pressing keys on the keyboard, whether inside or outside a text input area.

#### keydown/keypress

The `keydown` event is the first keyboard event to occur when a key is pressed. If the user continues to hold down the key, the `keydown` event will continue to fire. The `keypress` event is a common synonym for the `keydown` event; they behave virtually identically, with one exception: if you wish to prevent the default action of a key being pressed, you must do it on the `keypress` event.

#### keyup

The `keyup` event is the last keyboard event to occur (after the `keydown` event). Unlike the `keydown` event, this event will only fire once when released (since you can't release a key for a long period of time).

### Form Events

Form events deal primarily with `<form>`, `<input>`, `<select>`, `<button>`, and `<textarea>` elements, the staples of HTML forms.

#### select

The `select` event fires every time a user selects a different block of text within an input area, using the mouse. With this event, you can redefine the way a user interacts with a form.

#### change

The `change` event occurs when the value of an input element (this includes `<select>` and `<textarea>` elements) is modified by a user. The event fires only after the user has already left the element, letting it lose focus.

#### submit

The `submit` event occurs only in forms and only when a user clicks a Submit button (located within the form) or hits Enter/Return within one of the `input` elements. By binding a Submit handler to the form, and not a click handler to the Submit button, you'll be sure to capture all attempts to submit the form by the user.

#### reset

The `reset` event only occurs when a user clicks a Reset button inside a form (as opposed to the Submit button, which can be duplicated by hitting the Enter key).

### Event Accessibility

The final piece to take into consideration when developing a purely unobtrusive web application is to make sure that your events will work even without the use of a mouse. By doing this, you help two groups of people: those in need of accessibility assistance (vision-impaired users), and people who don't like to use a mouse. (Sit down one day, disconnect your mouse from your computer, and learn how to navigate the Web using only the keyboard. It's a real eye-opening experience).

To make your JavaScript events more accessible, any time you use the `click`, `mouseover`, and `mouseout` events, you should strongly consider providing alternative nonmouse bindings. Thankfully, there are easy ways to remedy this situation quickly:

- The `click` event: One smart move on the part of browser developers was to make the `click` event work whenever the Enter key is pressed. This completely removes the need to provide an alternative to this event. One point to note, however, is that some developers like to bind click handlers to Submit buttons in forms to watch for when a user submits a web page. Instead of using that event, the developer should bind to the `submit` event on the form object, a smart alternative that works reliably.

- The `mouseover` event: When navigating a web page using a keyboard, you're actually changing the focus to different elements. By attaching event handlers to both the `mouseover` and `focus` events, you can make sure that you'll have an equivalent solution for both keyboard and mouse users.

- The `mouseout` event: Like the `focus` event for the `mouseover` event, the `blur` event occurs whenever the user's focus moves away from an element. You can then use the `blur` event as a way to simulate the `mouseout` event with the keyboard.

In reality, adding the ability to handle keyboard events, in addition to typical mouse events, is completely trivial. If nothing else, doing so can help keyboard-dependent users better use your site, which is a huge win for everyone.

## Summary

In this chapter we started with an introduction to how events work in JavaScript and compared them to event models in other languages. Then you saw what information the event model provides and how you can best control it. We then explored binding events to DOM elements, and the different types of events that are available. We concluded with a discussion of event object properties, event types, and how to code for accessibility.
