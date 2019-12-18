# Altering the DOM with JavaScript

26th apr 2017

If you’re learning JavaScript, the first thing you should learn (after understanding the basics like variables, functions, etc.) is to alter the DOM. This is one of the things you do daily as a frontend developer.

Changing the DOM used to be difficult. We needed jQuery to make things easier. Luckily, there’s no need for jQuery anymore.

In this article, I’ll show you the things you need to be familiar with as a frontend developer.

## What do you do with the DOM?

When you work with the DOM, you’ll find yourself needing to do one or more of the following things:

1. Selecting HTML elements
2. Adding or removing event listeners
3. Adding or removing classes
4. Adding, changing or removing attributes
5. Adding or removing HTML elements

I’ll explain what each of these things are, why you use them and how to do them in the following sections. Let’s jump into the very first one — selecting HTML elements.

## Selecting HTML elements

Knowing how to select HTML elements is the first step before you do anything else with the DOM. Once you’ve selected an element, you’ll be able to add event listeners, change classes, and do other fancy things.

You only need to know two methods to select anything you want — `querySelector` and `querySelectorAll`.

### querySelector

`querySelector` helps you **select one HTML element**. If multiple HTML elements are found with your selection, `querySelector` always returns the first element. It looks like this:

```js
document.querySelector(selector)
```

You can select an element by its id, class or even tag with `querySelector`.

Let’s walk through a quick example. Say you have the following HTML:

```html
<div id="the-one">ID</div>
<div class="an-awesome-class">Class</div>
<p>A tag</p>
```

- To **select the element with an id**, you prepend the id with a `#`
- To **select the element with a class**, you prepend the class with a `.`
- To **select the element with a tag**, you simply write the tag as your selector.

```js
document.querySelector('#the-one')
// => <div id="id">ID</div>

document.querySelector('.an-awesome-class')
// => <div class="an-awesome-class">Class</div>

document.querySelector('p')
// => <p>A tag</p>
```

### Complicated selections

`querySelector` is incredibly powerful. You can perform complicated selections by chaining ids, classes and tags together, like this:

```js
document.querySelector('div#the-one')
```

Although it’s possible to chain selectors, I recommend you don’t do this because it’s unnecessary most of the time.

### Selecting elements within elements

Here’s one thing great about `querySelector`. You can instruct it to look for an element within another element, which reduces the time needed to lookup a deep selector.

To do so, you add a space between your classes, ids or tags. Here’s an example:

```html
<div class="container">
  <div class="inner-item">Inner item!</div>
</div>
let innerItem = document.querySelector('.container .inner-item')
// => <div class="inner-item">Inner item!</div>
```

Alternatively, if you’ve already selected an element with `querySelector` you can also use that element to perform another `querySelector` call:

```js
let container = document.querySelector('.container')
let innerItem = container.querySelector('.inner-item')
// => innerItem is <div class="inner-item">Inner item!</div>
```

That’s all you need to know about `querySelector`.

Now, what if you needed to select more than one element? This is where `querySelectorAll` comes in.

### querySelectorAll

`querySelectorAll` is a method that helps you select multiple elements.

```js
let allELements = document.querySelectorAll(selectors);
```

`selectors`, in this case, has the same syntax as `querySelector`. The only exception is that you can perform multiple selections by separating selections with a comma (`,`).

```html
<div class="thing">A thing</div>
<div class="thing">A thing</div>
<div class="another-thing">Another thing</div>
let allThings = document.querySelectorAll('.thing, .another-thing')
// => [
//   <div class="thing">A thing</div>,
//   <div class="thing">A thing</div>,
//   <div class="another-thing">Another thing</div>
// ]
```

Here’s the important part.

`querySelectorAll` returns a [NodeList](https://developer.mozilla.org/en-US/docs/Web/API/NodeList) (even though it looks like an array).

If you’re only working with modern browsers, you can get individual elements within the Nodelist with a `NodeList.forEach` call.

If you’re working with older browsers, you need to convert the NodeList into an Array before looping through it with a forEach call. The easiest way to do so is to use `Array.from()`.

```js
// Modern browsers
let allThings = document.querySelectorAll('.thing, .another-thing')
allThings.forEach(el => {/* do something with element */})

// Older browsers
let allThings = document.querySelectorAll('.thing, .another-thing')
// You might need a polyfill for Array.from.
// Alternatively, use Array.prototype.slice.call(allThings);
let allThingsArray = Array.from(allThings)
allThingsArray.forEach(el => {/* do something with element */})
```

Next, let’s move on to adding and removing event listeners.

## Adding and removing event listeners

Event listeners allow your JavaScript to perform an action whenever an event is triggered. This is how you know when a user has interacted with the DOM. One example is when they clicked a button:

See the Pen [Altering DOM with JS demo](https://codepen.io/zellwk/pen/eWBLdZ/) by Zell Liew ([@zellwk](https://codepen.io/zellwk)) on [CodePen](https://codepen.io/).

Here, you only need to know two methods — `addEventListener` and `removeEventListener`.

### Adding event listeners

To add your event listener, you first select your HTML element, then call the `addEventListener` method. It accepts two parameters, like this:

```js
let thing = document.querySelector('.thing')
thing.addEventListener(event, callback)
```

`event` is the name of the event you want to listen to. These events are already predetermined in the spec. Here’s a [handy list of common event types](https://developer.mozilla.org/en-US/docs/Web/Events) you’ll want.

`callback` is a function that does what you want whenever the event is triggered. It contains one parameter — the event object. Here’s what a typical callback looks like:

```js
thing.addEventListener('click', callback)

function callback (e) {
  e.preventDefault() // Prevents default behavior. Only use this when necessary
  console.log('thing is clicked!')
}
```

We can do [a lot of stuff](https://developer.mozilla.org/en/docs/Web/API/Event) with the event object (`e`), but that’s a topic for another day. Let’s move on to removing event listeners.

### Removing Event Listeners

Removing an event listener is similar to adding an event listener. Here, you call the `removeEventListener` method, then pass in two parameters — the `event` type and your callback.

```js
thing.removeEventListener('click', callback)
```

Typically, you will only remove an eventListener after a task is completed. So, it’s common to find `removeEventListener` within an `addEventListener` call.

```js
thing.addEventListener('click', callback)

function callback () {
  console.log('thing is clicked!')
  // removes event listener
  thing.removeEventListener('click', callback)
}
```

When you remove an event listener, you no longer listen to the event. So, in the code above, the callback only triggers when `.thing` gets clicked for the first time. Further clicks on `.thing` will not trigger the callback.

Note: you should remove an event listener when you have no more need for it. By doing so, you free up resources for other tasks.

Let’s move on.

## Adding and removing classes

Remember button demo above?

See the Pen [Altering DOM with JS demo](https://codepen.io/zellwk/pen/eWBLdZ/) by Zell Liew ([@zellwk](https://codepen.io/zellwk)) on [CodePen](https://codepen.io/).

Here’s what I did to make this demo work:

1. Add `.is-open` to `` when a user clicks on the button
2. Remove `.is-open` from `` if `` is already open when the user clicks on the button.
3. Transitioning the `` is done with CSS.

This demo shows you the power of CSS when combined with JavaScript. You can create all sorts of interactions just by adding (or removing) a class.

Here’s how you can add a class, remove a class or check if a class exists:

1. To **add a class**, use `element.classList.add('classname')`
2. To **remove a class**, use `element.classList.remove('classname')`
3. To **check if a class exists**, use `element.classList.contains('classname')`

Here’s the code to add or remove `.is-open` from `` when you click on the button.

```js
let button = document.querySelector('button')
let nav = document.querySelector('nav')

button.addEventListener('click', toggleNav)

function toggleNav() {
  // Checks if nav has is-open class
  if (nav.classList.contains('is-open')) {
    // removes is-open class
    nav.classList.remove('is-open')
  } else {
    // adds is-open class
    nav.classList.add('is-open')
  }
}
```

Let’s move on to adding, changing and removing attributes

## Adding, changing and removing attributes

Attributes are an important part of HTML elements. Sometimes, you need to extract information from these attributes to give context to your JavaScript. Other times, you can use these attributes to help write accessible interfaces.

Here’s a demo of the above nav, written in an accessible way:

See the Pen [Altering DOM with JS demo (Accessible way)](https://codepen.io/zellwk/pen/aWBaME/) by Zell Liew ([@zellwk](https://codepen.io/zellwk)) on [CodePen](https://codepen.io/).

In this demo, two things changed:

1. I added `aria-expanded` to `button` to tell screen readers if the menu is expanded.
2. I added `aria-hidden` to `nav` to prevent screen readers from reading the menu when it’s hidden.

Here’s how you can extract information from and attribute, set an attribute and remove an attribute:

1. To **get an attribute**, use `getAttribute('attribute-name')`
2. To **change/set an attribute**, use `setAttribute('attribute-name', 'attribute-value')`
3. To **remove an attribute**, use `removeAttribute('attribute-name')`

```js
// Get attribute
button.getAttribute('aria-expanded')

// Set attribute
button.setAttribute('aria-expanded', true)

// Remove attribute
button.removeAttribute('aria-expanded')
```

Finally, let’s move on to adding or removing elements.

## Adding or removing elements

Let’s start this section with a demo:

See the Pen [Altering DOM with JS demo (Adding and removing elements)](https://codepen.io/zellwk/pen/EmNdWp/) by Zell Liew ([@zellwk](https://codepen.io/zellwk)) on [CodePen](https://codepen.io/).

If you clicked on the prepend or append button above, you’d see I’ve added the text `Hello again, world!` into the DOM as another list item.

### Adding elements to the DOM

There are three steps to adding this text into the DOM. They are:

1. Create an HTML element with `document.createElement`
2. Add content to the HTML element by setting the `innerHTML`.
3. Add it to the DOM with `parentNode.prepend` or `parentNode.append`.

```js
let ul = document.querySelector('ul')

// Creating a <li> element
let li = document.createElement('li')

// Adding content to the <li> element
li.innerHTML = 'Hello again, world!'

// Adding it to the DOM
ul.append(li)
```

### Removing elements from the DOM

To remove an element from the DOM, you need to call `parentNode.removeChild`. This method takes in a parameter — the element to remove.

```js
ul.removeChild(li)
```

We can’t simply say remove `` and expect the JavaScript to know which list item to remove. We need to tell our JavaScript which one to remove explicitly.

If you can use `querySelector` to choose with element to remove, that’s going to be the easiest method:

```js
let parent = document.querySelector('.parent')
let elToRemove = document.querySelector('.element-to-remove')
parent.removeChild(elToRemove)

// Of if you don't want to write a separate querySelector
elToRemove.parentNode.removeChild(elToRemove)
```

In the demo above, we can’t do this because there’s no way to tell which is the first or last list item with classes or ids.

Instead, we can use `parentNode.children` to get a NodeList of elements within `ul`, then, use Array methods to get the specific element to remove.

Here’s the code to remove the first child element:

```js
let list = document.querySelector('ul')
removeFirst.addEventListener('click', e => {
  if (list.children.length) {
    let firstNode = list.children[0]
    list.removeChild(firstNode)
  }
})
```

## Wrapping up

Altering the DOM is one of the most important things you need to know as a frontend developer. You’ll be able to do all sorts of fancy stuff the moment you learn to work with the DOM.

In this article, I’ve showed you five common ways you need to alter the DOM, plus the relevant code you need to know. Now, go and play with the DOM and create some magic 😎.