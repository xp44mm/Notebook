# CSS Transitions

**CSS Transitions** is a module of CSS that lets you create gradual transitions between the values of specific CSS properties. The behavior of these transitions can be controlled by specifying their timing function, duration, and other attributes.

# Using CSS transitions

**CSS transitions** provide a way to control animation speed when changing CSS properties. Instead of having property changes take effect immediately, you can cause the changes in a property to take place over a period of time. For example, if you change the color of an element from white to black, usually the change is instantaneous. With CSS transitions enabled, changes occur at time intervals that follow an acceleration curve, all of which can be customized.

Animations that involve transitioning between two states are often called *implicit transitions* as the states in between the start and final states are implicitly defined by the browser.

![A CSS transition tells the browser to draw the intermediate states between the initial and final states, showing the user a smooth transitions.](https://developer.mozilla.org/files/4529/TransitionsPrinciple.png)

CSS transitions let you decide which properties to animate (by *listing them explicitly*), when the animation will start (by setting a *delay)*, how long the transition will last (by setting a *duration*), and how the transition will run (by defining a *timing function*, e.g. linearly or quick at the beginning, slow at the end).

## Which CSS properties can be transitioned?

The Web author can define which property has to be animated and in which way. This allows the creation of complex transitions. As it doesn't make sense to animate some properties, the [list of animatable properties](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_animated_properties) is limited to a finite set.

Note: The set of properties that can be animated is changing as the specification develops.

The `auto` value is often a very complex case. The specification recommends not animating from and to `auto`. Some user agents, like those based on Gecko, implement this requirement and others, like those based on WebKit, are less strict. Using animations with `auto` may lead to unpredictable results, depending on the browser and its version, and should be avoided.

## Defining transitions

CSS Transitions are controlled using the shorthand [`transition`](https://developer.mozilla.org/en-US/docs/Web/CSS/transition) property. This is the best way to configure transitions, as it makes it easier to avoid out of sync parameters, which can be very frustrating to have to spend lots of time debugging in CSS.

You can control the individual components of the transition with the following sub-properties:

(Note that these transitions loop infinitely only for the purpose of our examples; CSS `transition`s only visualize a property change from start to finish. If you need visualizations that loop, look into the CSS [`animation`](https://developer.mozilla.org/en-US/docs/Web/CSS/animation) property.)

- [`transition-property`](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-property)

  Specifies the name or names of the CSS properties to which transitions should be applied. Only properties listed here are animated during transitions; changes to all other properties occur instantaneously as usual.

- [`transition-duration`](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-duration)

  Specifies the duration over which transitions should occur. You can specify a single duration that applies to all properties during the transition, or multiple values to allow each property to transition over a different period of time.

The shorthand CSS syntax is written as follows:

```css
div {
    transition: <property> <duration> <timing-function> <delay>;
}
```

## Examples

### Simple example



This example performs a four-second font size transition with a two-second delay between the time the user mouses over the element and the beginning of the animation effect:

```css
#delay {
  font-size: 14px;
  transition-property: font-size;
  transition-duration: 4s;
  transition-delay: 2s;
}

#delay:hover {
  font-size: 36px;
}
```

### Multiple animated properties example



#### CSS Content

```css
.box {
    border-style: solid;
    border-width: 1px;
    display: block;
    width: 100px;
    height: 100px;
    background-color: #0000FF;
    transition: width 2s, height 2s, background-color 2s, transform 2s;
}

.box:hover {
    background-color: #FFCCCC;
    width: 200px;
    height: 200px;
    transform: rotate(180deg);
}
```

### When property value lists are of different lengths



If any property's list of values is shorter than the others, its values are repeated to make them match. For example:

```css
div {
  transition-property: opacity, left, top, height;
  transition-duration: 3s, 5s;
}
```

This is treated as if it were:

```css
div {
  transition-property: opacity, left, top, height;
  transition-duration: 3s, 5s, 3s, 5s;
}
```

Similarly, if any property's value list is longer than that for [`transition-property`](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-property), it's truncated, so if you have the following CSS:

```css
div {
  transition-property: opacity, left;
  transition-duration: 3s, 5s, 2s, 1s;
}
```

This gets interpreted as:

```css
div {
  transition-property: opacity, left;
  transition-duration: 3s, 5s;
}
```

### Using transitions when highlighting menus



A common use of CSS is to highlight items in a menu as the user hovers the mouse cursor over them. It's easy to use transitions to make the effect even more attractive.

Before we look at code snippets, you might want to take a look at the [live demo](https://codepen.io/anon/pen/WOEpva) (assuming your browser supports transitions).

First, we set up the menu using HTML:

```html
<nav>
  <a href="#">Home</a>
  <a href="#">About</a>
  <a href="#">Contact Us</a>
  <a href="#">Links</a>
</nav>
```

Then we build the CSS to implement the look and feel of our menu. The relevant portions are shown here:

```css
a {
  color: #fff;
  background-color: #333;
  transition: all 1s ease-out;
}

a:hover,
a:focus {
  color: #333;
  background-color: #fff;
}
```

This CSS establishes the look of the menu, with the background and text colors both changing when the element is in its [`:hover`](https://developer.mozilla.org/en-US/docs/Web/CSS/:hover) and [`:focus`](https://developer.mozilla.org/en-US/docs/Web/CSS/:focus) states.

## JavaScript examples

Care should be taken when using a transition immediately after:

- adding the element to the DOM using `.appendChild()`
- removing an element's `display: none;` property.

This is treated as if the initial state had never occurred and the element was always in its final state. The easy way to overcome this limitation is to apply a `window.setTimeout()` of a handful of milliseconds before changing the CSS property you intend to transition to.

### Using transitions to make JavaScript functionality smooth



Transitions are a great tool to make things look much smoother without having to do anything to your JavaScript functionality. Take the following example.

```html
<p>Click anywhere to move the ball</p>
<div id="foo"></div>
```

Using JavaScript you can make the effect of moving the ball to a certain position happen:

```js
var f = document.getElementById('foo');
document.addEventListener('click', function(ev){
    f.style.transform = 'translateY('+(ev.clientY-25)+'px)';
    f.style.transform += 'translateX('+(ev.clientX-25)+'px)';
},false);
```

With CSS you can make it smooth without any extra effort. Simply add a transition to the element and any change will happen smoothly:

```css
p {
  padding-left: 60px;
}

#foo {
  border-radius: 50px;
  width: 50px;
  height: 50px;
  background: #c00;
  position: absolute;
  top: 0;
  left: 0;
  transition: transform 1s;
}
```

You can play with this here: http://jsfiddle.net/9h261pzo/291/

### Detecting the start and completion of a transition



You can use the `transitionend` event to detect that an animation has finished running. This is a [`TransitionEvent`](https://developer.mozilla.org/en-US/docs/Web/API/TransitionEvent) object, which has two added properties beyond a typical [`Event`](https://developer.mozilla.org/en-US/docs/Web/API/Event) object:

- `propertyName`

  A string indicating the name of the CSS property whose transition completed.

- `elapsedTime`

  A float indicating the number of seconds the transition had been running at the time the event fired. This value isn't affected by the value of [`transition-delay`](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-delay).

As usual, you can use the [`addEventListener()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener) method to monitor for this event:

```js
el.addEventListener("transitionend", updateTransition, true);
```

You detect the beginning of a transition using `transitionrun` (fires before any delay) and `transitionstart` (fires after any delay), in the same kind of fashion:

```js
el.addEventListener("transitionrun", signalStart, true);
el.addEventListener("transitionstart", signalStart, true);
```

**Note**: The `transitionend` event doesn't fire if the transition is aborted before the transition is completed because either the element is made [`display`](https://developer.mozilla.org/en-US/docs/Web/CSS/display)`: none` or the animating property's value is changed.