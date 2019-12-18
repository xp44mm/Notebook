# CSS layout

At this point we've already looked at CSS fundamentals, how to style text, and how to style and manipulate the boxes that your content sits inside. Now it's time to look at how to place your boxes in the right place in relation to the viewport, and one another. We have covered the necessary prerequisites so we can now dive deep into CSS layout, looking at different display settings, modern layout tools like flexbox, CSS grid, and positioning, and some of the legacy techniques you might still want to know about.

## Guides

These articles will provide instruction on the fundamental layout tools and techniques available in CSS. At the end of the lessons is an assessment to help you check your understanding of layout methods, by laying out a webpage.

- [Introduction to CSS layout](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Introduction)

  This article will recap some of the CSS layout features we've already touched upon in previous modules — such as different [`display`](https://developer.mozilla.org/en-US/docs/Web/CSS/display) values — and introduce some of the concepts we'll be covering throughout this module.

- [Normal flow](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Normal_Flow)

  Elements on webpages lay themselves out according to *normal flow* - until we do something to change that. This article explains the basics of normal flow as a grounding for learning how to change it.

- [Flexbox](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox)

  Flexbox is a one-dimensional layout method for laying out items in rows or columns. Items flex to fill additional space and shrink to fit into smaller spaces. This article explains all the fundamentals.

- [Grids](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Grids)

  CSS Grid Layout is a two-dimensional layout system for the web. It lets you lay content out in rows and columns, and has many feature that make building complex layouts straightforward. This article will give you all you need to know to get started with page layout.

- [Floats](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Floats)

  Originally for floating images inside blocks of text, the [`float`](https://developer.mozilla.org/en-US/docs/Web/CSS/float) property became one of the most commonly used tools for creating multiple column layouts on webpages. With the advent of Flexbox and Grid it has now returned to its original purpose, as this article explains.

- [Positioning](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Positioning)

  Positioning allows you to take elements out of the normal document layout flow, and make them behave differently, for example sitting on top of one another, or always remaining in the same place inside the browser viewport. This article explains the different [`position`](https://developer.mozilla.org/en-US/docs/Web/CSS/position) values, and how to use them.

- [Multiple-column layout](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Multiple-column_Layout)

  The multiple-column layout specification gives you a method of laying content out in columns, as you might see in a newspaper. This article explains how to use this feature.

- [Responsive design](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Responsive_Design)

  As more diverse screen sizes have appeared on web-enabled devices, the concept of responsive web design (RWD) has appeared: a set of practices that allows web pages to alter their layout and appearance to suit different screen widths, resolutions, etc. It is an idea that changed the way we design for a multi-device web, and in this article we'll help you understand the main techniques you need to know to master it.

- [Beginner's guide to media queries](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Media_queries)

  The **CSS Media Query** gives you a way to apply CSS only when the browser and device environment matches a rule that you specify, for example "viewport is wider than 480 pixels". Media queries are a key part of responsive web design, as they allow you to create different layouts depending on the size of the viewport, but they can also be used to detect other things about the environment your site is running on, for example whether the user is using a touchscreen rather than a mouse. In this lesson you will first learn about the syntax used in media queries, and then move on to use them in a worked example showing how a simple design might be made responsive.

- [Legacy layout methods](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Legacy_Layout_Methods)

  Grid systems are a very common feature used in CSS layouts, and before CSS Grid Layout they tended to be implemented using floats or other layout features. You imagine your layout as a set number of columns (e.g. 4, 6, or 12), and then fit your content columns inside these imaginary columns. In this article we'll explore how these older methods work, in order that you understand how they were used if you work on an older project.

- [Supporting older browsers](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Supporting_Older_Browsers)

  In this module we recommend using Flexbox and Grid as the main layout methods for your designs. However there will be visitors to your site who use older browsers, or browsers which do not support the methods you have used. This will always be the case on the web — as new features are developed, different browsers will prioritise different things. This article explains how to use modern web techniques without locking out users of older technology.

- [Assessment: Fundamental layout comprehension](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Fundamental_Layout_Comprehension)

  An assessment to test your knowledge of different layout methods by laying out a webpage.

## See also

- [Practical positioning examples](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Practical_positioning_examples)

  This article shows how to build some real world examples to illustrate what kinds of things you can do with positioning.

# Introduction to CSS layout

This article will recap some of the CSS layout features we've already touched upon in previous modules — such as different [`display`](https://developer.mozilla.org/en-US/docs/Web/CSS/display) values — and introduce some of the concepts we'll be covering throughout this module.

CSS page layout techniques allow us to take elements contained in a web page and control where they are positioned relative to their default position in normal layout flow, the other elements around them, their parent container, or the main viewport/window.  The page layout techniques we'll be covering in more detail in this module are

- Normal flow
- The [`display`](https://developer.mozilla.org/en-US/docs/Web/CSS/display) property
- Flexbox
- Grid
- Floats
- Positioning
- Table layout
- Multiple-column layout

Each technique has its uses, advantages, and disadvantages, and no technique is designed to be used in isolation. By understanding what each method is designed for you will be in a good place to understand which is the best layout tool for each task.

## Normal flow

Normal flow is how the browser lays out HTML pages by default when you do nothing to control page layout. Let's look at a quick HTML example:

```html
<p>I love my cat.</p>
    
<ul>
  <li>Buy cat food</li>
  <li>Exercise</li>
  <li>Cheer up friend</li>
</ul>
    
<p>The end!</p>
```

By default, the browser will display this code as follows:

Note here how the HTML is displayed in the exact order in which it appears in the source code, with elements stacked up on top of one another — the first paragraph, followed by the unordered list, followed by the second paragraph.

The elements that appear one below the other are described as *block* elements, in contrast to *inline* elements, which appear one beside the other, like the individual words in a paragraph.

**Note**: The direction in which block element contents are laid out is described as the Block Direction. The Block Direction runs vertically in a language such as English, which has a horizontal writing mode. It would run horizontally in any language with a Vertical Writing Mode, such as Japanese. The corresponding Inline Direction is the direction in which inline contents (such as a sentence) would run.

When you use CSS to create a layout, you are moving the elements away from the normal flow, but for many of the elements on your page the normal flow will create exactly the layout you need. This is why starting with a well-structured HTML document is so important, as you can then work with the way things are laid out by default rather than fighting against it.

The methods that can change how elements are laid out in CSS are as follows:

- **The [`display`](https://developer.mozilla.org/en-US/docs/Web/CSS/display) property** — Standard values such as `block`, `inline` or `inline-block` can change how elements behave in normal flow (see [Types of CSS boxes](https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS/Box_model#Types_of_CSS_boxes) for more information). We then have entire layout methods that are switched on via a value of `display`, for example [CSS Grid](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Grids) and [Flexbox](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox).
- **Floats** — Applying a [`float`](https://developer.mozilla.org/en-US/docs/Web/CSS/float) value such as `left` can cause block level elements to wrap alongside one side of an element, like the way images sometimes have text floating around them in magazine layouts.
- **The [`position`](https://developer.mozilla.org/en-US/docs/Web/CSS/position) property** — Allows you to precisely control the placement of boxes inside other boxes. `static` positioning is the default in normal flow, but you can cause elements to be laid out differently using other values, for example always fixed to the top left of the browser viewport.
- **Table layout** — features designed for styling the parts of an HTML table can be used on non-table elements using `display: table` and associated properties..
- **Multi-column layout** — The [Multi-column layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Columns) properties can cause the content of a block to layout in columns, as you might see in a newspaper.

## The display property

The main methods of achieving page layout in CSS are all values of the `display` property. This property allows us to change the default way something displays. Everything in normal flow has a value of `display`, used as the default way that elements they are set on behave. For example, the fact that paragraphs in English display one below the other is due to the fact that they are styled with `display: block`. If you create a link around some text inside a paragraph, that link remains inline with the rest of the text, and doesn’t break onto a new line. This is because the `<a>` element is `display: inline` by default.

You can change this default display behavior. For example, the `<li>` element is `display: block` by default, meaning that list items display one below the other in our English document. If we change the display value to `inline` they now display next to each other, as words would do in a sentence. The fact that you can change the value of `display` for any element means that you can pick HTML elements for their semantic meaning, without being concerned about how they will look. The way they look is something that you can change.

In addition to being able to change the default presentation by turning an item from `block` to `inline` and vice versa, there are some bigger layout methods that start out as a value of `display`. However, when using these, you will generally need to invoke additional properties. The two values most important for our purposes when discussing layout are `display: flex` and `display: grid`.

## Flexbox

Flexbox is the short name for the [Flexible Box Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout) Module, designed to make it easy for us to lay things out in one dimension — either as a row or as a column. To use flexbox, you apply `display: flex` to the parent element of the elements you want to lay out; all its direct children then become flex items. We can see this in a simple example.

The HTML markup below gives us a containing element, with a class of `wrapper`, inside which are three `<div>` elements. By default these would display as block elements, below one another, in our English language document.

However, if we add `display: flex` to the parent, the three items now arrange themselves into columns. This is due to them becoming *flex items* and using some initial values that flexbox gives them. They are displayed as a row, because the initial value of [`flex-direction`](https://developer.mozilla.org/en-US/docs/Web/CSS/flex-direction) is `row`. They all appear to stretch to the height of the tallest item, because the initial value of the [`align-items`](https://developer.mozilla.org/en-US/docs/Web/CSS/align-items) property is `stretch`. This means that the items stretch to the height of the flex container, which in this case is defined by the tallest item. The items all line up at the start of the container, leaving any extra space at the end of the row.

```css
.wrapper {
  display: flex;
}
```

```html
<div class="wrapper">
  <div class="box1">One</div>
  <div class="box2">Two</div>
  <div class="box3">Three</div>
</div>
```

In addition to the above properties that can be applied to the flex container, there are properties that can be applied to the flex items. These properties, among other things, can change the way that the items flex, enabling them to expand and contract to fit into the available space.

As a simple example of this, we can add the [`flex`](https://developer.mozilla.org/en-US/docs/Web/CSS/flex) property to all of our child items, with a value of `1`. This will cause all of the items to grow and fill the container, rather than leaving space at the end. If there is more space then the items will become wider; if there is less space they will become narrower. In addition, if you add another element to the markup the items will all become smaller to make space for it — they will adjust size to take up the same amount of space, whatever that is.

```css
.wrapper {
    display: flex;
}

.wrapper > div {
    flex: 1;
}
```

```html
<div class="wrapper">
    <div class="box1">One</div>
    <div class="box2">Two</div>
    <div class="box3">Three</div>
</div>
```

**Note**: This has been a very short introduction to what is possible in Flexbox, to find out more, see our [Flexbox](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox) article.

## Grid Layout

While flexbox is designed for one-dimensional layout, Grid Layout is designed for two dimensions — lining things up in rows and columns.

Once again, you can switch on Grid Layout with a specific value of display — `display: grid`. The below example uses similar markup to the flex example, with a container and some child elements. In addition to using `display: grid`, we are also defining some row and column tracks on the parent using the [`grid-template-rows`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template-rows) and [`grid-template-columns`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template-columns) properties respectively. We've defined three columns each of `1fr` and two rows of `100px`. I don’t need to put any rules on the child elements; they are automatically placed into the cells our grid has created.

```css
.wrapper {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    grid-template-rows: 100px 100px;
    grid-gap: 10px;
}
<div class="wrapper">
    <div class="box1">One</div>
    <div class="box2">Two</div>
    <div class="box3">Three</div>
    <div class="box4">Four</div>
    <div class="box5">Five</div>
    <div class="box6">Six</div>
</div>
```

Once you have a grid, you can explicitly place your items on it, rather than relying on the auto-placement behavior seen above. In the second example below we have defined the same grid, but this time with three child items. We've set the start and end line of each item using the [`grid-column`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-column) and [`grid-row`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-row) properties. This causes the items to span multiple tracks.

```css
.wrapper {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    grid-template-rows: 100px 100px;
    grid-gap: 10px;
}

.box1 {
    grid-column: 2 / 4;
    grid-row: 1;
}

.box2 {
    grid-column: 1;
    grid-row: 1 / 3;
}

.box3 {
    grid-row: 2;
    grid-column: 3;
}
```

```html
<div class="wrapper">
    <div class="box1">One</div>
    <div class="box2">Two</div>
    <div class="box3">Three</div>
</div>
```

**Note**: These two examples are just a small part of the power of Grid layout; to find out more see our [Grid Layout](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Grids) article.

The rest of this guide covers other layout methods, which are less important for the main layout structures of your page but can still help you achieve specific tasks. By understanding the nature of each layout task, you will soon find that when you look at a particular component of your design the type of layout best suited to it will often be clear.

## Floats

Floating an element changes the behavior of that element and the block level elements that follow it in normal flow. The element is moved to the left or right and removed from normal flow, and the surrounding content floats around the floated item.

The [`float`](https://developer.mozilla.org/en-US/docs/Web/CSS/float) property has four possible values:

- `left` — Floats the element to the left.
- `right` — Floats the element to the right.
- `none` — Specifies no floating at all. This is the default value.
- `inherit` — Specifies that the value of the `float` property should be inherited from the element's parent element.

In the example below we float a `<div>` left, and give it a `margin` on the right to push the text away from the element. This gives us the effect of text wrapped around that box, and is most of what you need to know about floats as used in modern web design.

```html
<h1>Simple float example</h1>
    
<div class="box">Float</div>
    
<p> Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla luctus aliquam dolor, eu lacinia lorem placerat vulputate. Duis felis orci, pulvinar id metus ut, rutrum luctus orci. Cras porttitor imperdiet nunc, at ultricies tellus laoreet sit amet. Sed auctor cursus massa at porta. Integer ligula ipsum, tristique sit amet orci vel, viverra egestas ligula. Curabitur vehicula tellus neque, ac ornare ex malesuada et. In vitae convallis lacus. Aliquam erat volutpat. Suspendisse ac imperdiet turpis. Aenean finibus sollicitudin eros pharetra congue. Duis ornare egestas augue ut luctus. Proin blandit quam nec lacus varius commodo et a urna. Ut id ornare felis, eget fermentum sapien.</p>

```

```css
.box {
    float: left;
    width: 150px;
    height: 150px;
    margin-right: 30px;
}
```

**Note**: Floats are fully explained in our lesson on the [float and clear](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Floats) properties. Prior to techniques such as Flexbox and Grid Layout floats were used as a method of creating column layouts. You may still come across these methods on the web; we will cover these in the lesson on [legacy layout methods](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Legacy_Layout_Methods).

## Positioning techniques

Positioning allows you to move an element from where it would be placed when in normal flow to another location. Positioning isn’t a method for creating your main page layouts, it is more about managing and fine-tuning the position of specific items on the page.

There are however useful techniques for certain layout patterns that rely on the [`position`](https://developer.mozilla.org/en-US/docs/Web/CSS/position) property. Understanding positioning also helps in understanding normal flow, and what it is to move an item out of normal flow.

There are five types of positioning you should know about:

- **Static positioning** is the default that every element gets — it just means "put the element into its normal position in the document layout flow — nothing special to see here".
- **Relative positioning** allows you to modify an element's position on the page, moving it relative to its position in normal flow — including making it overlap other elements on the page.
- **Absolute positioning** moves an element completely out of the page's normal layout flow, like it is sitting on its own separate layer. From there, you can fix it in a position relative to the edges of the page's `` element (or its nearest positioned ancestor element). This is useful for creating complex layout effects such as tabbed boxes where different content panels sit on top of one another and are shown and hidden as desired, or information panels that sit off screen by default, but can be made to slide on screen using a control button.
- **Fixed positioning** is very similar to absolute positioning, except that it fixes an element relative to the browser viewport, not another element. This is useful for creating effects such as a persistent navigation menu that always stays in the same place on the screen as the rest of the content scrolls.
- **Sticky positioning** is a newer positioning method which makes an element act like `position: static` until it hits a defined offset from the viewport, at which point it acts like `position: fixed`.

### Simple positioning example



To provide familiarity with these page layout techniques, we'll show you a couple of quick examples. Our examples will all feature the same HTML, which is as follows:

```html
<h1>Positioning</h1>

<p>I am a basic block level element.</p>
<p class="positioned">I am a basic block level element.</p>
<p>I am a basic block level element.</p>
```

This HTML will be styled by default using the following CSS:

```css
body {
  width: 500px;
  margin: 0 auto;
}

p {
    background-color: rgb(207,232,220);
    border: 2px solid rgb(79,185,227);
    padding: 10px;
    margin: 10px;
    border-radius: 5px;
}
```

The rendered output is as follows:

### Relative positioning



Relative positioning allows you to offset an item from the position in normal flow it would have by default. This means you could achieve a task such as moving an icon down a bit so it lines up with a text label. To do this, we could add the following rule to add relative positioning:

```css
.positioned {
  position: relative;
  top: 30px;
  left: 30px;
}
```

Here we give our middle paragraph a [`position`](https://developer.mozilla.org/en-US/docs/Web/CSS/position) value of `relative` — this doesn't do anything on its own, so we also add [`top`](https://developer.mozilla.org/en-US/docs/Web/CSS/top) and [`left`](https://developer.mozilla.org/en-US/docs/Web/CSS/left) properties. These serve to move the affected element down and to the right — this might seem like the opposite of what you were expecting, but you need to think of it as the element being pushed on its left and top sides, which result in it moving right and down.

Adding this code will give the following result:

```css
.positioned {
  position: relative;
  background: rgba(255,84,104,.3);
  border: 2px solid rgb(255,84,104);
  top: 30px;
  left: 30px;
}
```

### Absolute positioning



Absolute positioning is used to completely remove an element from normal flow, and place it using offsets from the edges of a containing block.

Going back to our original non-positioned example, we could add the following CSS rule to implement absolute positioning:

```css
.positioned {
  position: absolute;
  top: 30px;
  left: 30px;
}
```

Here we give our middle paragraph a [`position`](https://developer.mozilla.org/en-US/docs/Web/CSS/position) value of `absolute`, and the same [`top`](https://developer.mozilla.org/en-US/docs/Web/CSS/top) and [`left`](https://developer.mozilla.org/en-US/docs/Web/CSS/left) properties as before. Adding this code, however, will give the following result:

```css
.positioned {
    position: absolute;
    background: rgba(255,84,104,.3);
    border: 2px solid rgb(255,84,104);
    top: 30px;
    left: 30px;
}
```

This is very different! The positioned element has now been completely separated from the rest of the page layout and sits over the top of it. The other two paragraphs now sit together as if their positioned sibling doesn't exist. The [`top`](https://developer.mozilla.org/en-US/docs/Web/CSS/top) and [`left`](https://developer.mozilla.org/en-US/docs/Web/CSS/left) properties have a different effect on absolutely positioned elements than they do on relatively positioned elements. In this case the offsets have been calculated from the top and left of the page. It is possible to change the parent element that becomes this container and we will take a look at that in the lesson on [positioning](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Positioning).

### Fixed positioning



Fixed positioning removes our element from document flow in the same way as absolute positioning. However, instead of the offsets being applied from the container, they are applied from the viewport. As the item remains fixed in relation to the viewport we can create effects such as a menu which remains fixed as the page scrolls beneath it.

For this example our HTML is three paragraphs of text, in order that we can cause the page to scroll, and a box to which we will give `position: fixed`.

```html
<h1>Fixed positioning</h1>
<div class="positioned">Fixed</div>
<p>Paragraph 1.</p>
<p>Paragraph 2.</p>
<p>Paragraph 3.</p>

```

```css
.positioned {
    position: fixed;
    top: 30px;
    left: 30px;
}
```

### Sticky positioning



Sticky positioning is the final positioning method that we have at our disposal. It mixes the default static positioning with fixed positioning. When an item has `position: sticky` it will scroll in normal flow until it hits offsets from the viewport that we have defined. At that point it becomes "stuck" as if it had `position: fixed` applied.

```css
.positioned {
  position: sticky;
  top: 30px;
  left: 30px;
}
```

**Note**: to find more out about positioning, see our [Positioning](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Positioning) article.

## Table layout

HTML tables are fine for displaying tabular data, but many years ago — before even basic CSS was supported reliably across browsers — web developers used to also use tables for entire web page layouts — putting their headers, footers, different columns, etc. in various table rows and columns. This worked at the time, but it has many problems — table layouts are inflexible, very heavy on markup, difficult to debug, and semantically wrong (e.g., screen reader users have problems navigating table layouts).

The way that a table looks on a webpage when you use table markup is due to a set of CSS properties that define table layout. These properties can be used to lay out elements that are not tables, a use which is sometimes described as "using CSS tables".

The example below shows one such use; using CSS tables for layout should be considered a legacy method at this point, for those situations where you have very old browsers without support for Flexbox or Grid.

Let's look at an example. First, some simple markup that creates an HTML form. Each input element has a label, and we've also included a caption inside a paragraph. Each label/input pair is wrapped in a `<div>`, for layout purposes.

```html
<form>
  <p>First of all, tell us your name and age.</p>
  <div>
    <label for="fname">First name:</label>
    <input type="text" id="fname">
  </div>
  <div>
    <label for="lname">Last name:</label>
    <input type="text" id="lname">
  </div>
  <div>
    <label for="age">Age:</label>
    <input type="text" id="age">
  </div>
</form>
```

Now, the CSS for our example. Most of the CSS is fairly ordinary, except for the uses of the [`display`](https://developer.mozilla.org/en-US/docs/Web/CSS/display) property. The `<form>`, `<div>`s, and `<lable>`s and `<input>`s have been told to display like a table, table rows, and table cells respectively — basically, they'll act like HTML table markup, causing the labels and inputs to line up nicely by default. All we then have to do is add a bit of sizing, margin, etc. to make everything look a bit nicer and we're done.

You'll notice that the caption paragraph has been given `display: table-caption;` — which makes it act like a table `<caption>` — and `caption-side: bottom;` to tell the caption to sit on the bottom of the table for styling purposes, even though the markup is before the `<div>` elements in the source. This allows for a nice bit of flexibility.

```css
html {
  font-family: sans-serif;
}

form {
  display: table;
  margin: 0 auto;
}

form div {
  display: table-row;
}

form label, form input {
  display: table-cell;
  margin-bottom: 10px;
}

form label {
  width: 200px;
  padding-right: 5%;
  text-align: right;
}

form input {
  width: 300px;
}

form p {
  display: table-caption;
  caption-side: bottom;
  width: 300px;
  color: #999;
  font-style: italic;
}
```

This gives us the following result:

You can also see this example live at [css-tables-example.html](https://mdn.github.io/learning-area/css/styling-boxes/box-model-recap/css-tables-example.html) (see the [source code](https://github.com/mdn/learning-area/blob/master/css/styling-boxes/box-model-recap/css-tables-example.html) too.)

## Multi-column layout

The multi-column layout module gives us a way to lay out content in columns, similar to how text flows in a newspaper. While reading up and down columns is less useful in a web context as you don’t want to force users to scroll up and down, arranging content into columns can be a useful technique.

To turn a block into a multicol container we use either the [`column-count`](https://developer.mozilla.org/en-US/docs/Web/CSS/column-count) property, which tells the browser how many columns we would like to have, or the [`column-width`](https://developer.mozilla.org/en-US/docs/Web/CSS/column-width) property, which tells the browser to fill the container with as many columns of at least that width.

In the below example we start with a block of HTML inside a containing `<div>` element with a class of `container`.

```html
<div class="container">
    <h1>Multi-column layout</h1>
    <p>Paragraph 1.</p>
    <p>Paragraph 2.</p>
</div>
```

We are using a `column-width` of 200 pixels on that container, causing the browser to create as many 200-pixel columns as will fit in the container and then share the remaining space between the created columns.

```css
    .container {
        column-width: 200px;
    }
```

## Summary

This article has provided a brief summary of all the layout technologies you should know about. Read on for more information on each individual technology!

# Normal Flow

This article explains normal flow, or the way that webpage elements lay themselves out if you have not changed their layout.

As detailed in the last lesson introducing layout, elements on a webpage lay out in the normal flow, if you have not applied any CSS to change the way they behave. And, as we began to discover, you can change how elements behave either by adjusting their position in that normal flow, or removing them from it altogether. Starting with a solid, well-structured document that is readable in normal flow is the best way to begin any webpage. It ensures that your content is readable, even if the user is using a very limited browser or a device such as a screen reader that reads out the content of the page. In addition, as normal flow is designed to make a readable document, by starting in this way you are working with the document rather than fighting against it as you make changes to the layout.

Before digging deeper into different layout methods, it is worth revisiting some of the things you will have studied in previous modules with regard to normal document flow.

## How are elements laid out by default?

First of all, individual element boxes are laid out by taking the elements' content, then adding any padding, border and margin around them — it's that box model thing again, which we've looked at earlier.

By default, a block level element's content is 100% of the width of its parent element, and as tall as its content. Inline elements are as tall as their content, and as wide as their content. You can't set width or height on inline elements — they just sit inside the content of block level elements. If you want to control the size of an inline element in this manner, you need to set it to behave like a block level element with `display: block;` (or even,`display: inline-block;` which mixes characteristics from both.)

块元素像热烟充满帽子。热烟是内容，帽子是父元素。行内元素高度为行高，长度为内容长度。

That explains individual elements, but what about how elements interact with one another? The normal layout flow (mentioned in the layout introduction article) is the system by which elements are placed inside the browser's viewport. By default, block-level elements are laid out in the *block flow direction*, based on the parent's [writing mode](https://developer.mozilla.org/en-US/docs/Web/CSS/writing-mode) (*initial*: `horizontal-tb`) — each one will appear on a new line below the last one, and they will be separated by any margin that is set on them. In English therefore, or any other horizontal, top to bottom writing mode, block-level elements are laid out vertically.

Inline elements behave differently — they don't appear on new lines; instead, they sit on the same line as one another and any adjacent (or wrapped) text content, as long as there is space for them to do so inside the width of the parent block level element. If there isn't space, then the overflowing text or elements will move down to a new line.

If two adjacent elements both have the margin set on them and the two margins touch, the larger of the two remains, and the smaller one disappears — this is called margin collapsing, and we have met this before too.

margin is max between you and me.

Let's look at a simple example that explains all of this:

```html
<h1>Basic document flow</h1>

<p>I am a basic block level element. My adjacent block level elements sit on new lines below me.</p>

<p>By default we span 100% of the width of our parent element, and we are as tall as our child content. Our total width and height is our content + padding + border width/height.</p>

<p>We are separated by our margins. Because of margin collapsing, we are separated by the width of one of our margins, not both.</p>

<p>inline elements <span>like this one</span> and <span>this one</span> sit on the same line as one another, and adjacent text nodes, if there is space on the same line. Overflowing inline elements will <span>wrap onto a new line if possible (like this one containing text)</span>, or just go on to a new line if not, much like this image will do: <img src="https://mdn.mozillademos.org/files/13360/long.jpg"></p>
```

```css
body {
  width: 500px;
  margin: 0 auto;
}

p {
  background: rgba(255,84,104,0.3); 
  border: 2px solid rgb(255,84,104);
  padding: 10px;
  margin: 10px;
}

span {
  background: white;
  border: 1px solid black;
}
```

## Summary

Now that you understand normal flow, and how the browser lays things out by default, move on to understand how to change this default display to create the layout needed by your design.

# Flexbox

[Flexbox](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout) is a one-dimensional layout method for laying out items in rows or columns. Items flex to fill additional space and shrink to fit into smaller spaces. This article explains all the fundamentals.

## Why Flexbox?

For a long time, the only reliable cross browser-compatible tools available for creating CSS layouts were things like [floats](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Floats) and [positioning](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Positioning). These are fine and they work, but in some ways they are also rather limiting and frustrating.

The following simple layout requirements are either difficult or impossible to achieve with such tools, in any kind of convenient, flexible way:

- Vertically centering a block of content inside its parent.
- Making all the children of a container take up an equal amount of the available width/height, regardless of how much width/height is available.
- Making all columns in a multiple column layout adopt the same height even if they contain a different amount of content.

As you'll see in subsequent sections, flexbox makes a lot of layout tasks much easier. Let's dig in!

## Introducing a simple example

In this article we are going to get you to work through a series of exercises to help you understand how flexbox works. To get started, you should make a local copy of the first starter file — [flexbox0.html](https://github.com/mdn/learning-area/blob/master/css/css-layout/flexbox/flexbox0.html) from our github repo — load it in a modern browser (like Firefox or Chrome), and have a look at the code in your code editor. You can [see it live here](http://mdn.github.io/learning-area/css/css-layout/flexbox/flexbox0.html) also.

You'll see that we have a `<header>` element with a top level heading inside it, and a `<section>` element containing three `<article>`s. We are going to use these to create a fairly standard three column layout.

## Specifying what elements to lay out as flexible boxes

To start with, we need to select which elements are to be laid out as flexible boxes. To do this, we set a special value of [`display`](https://developer.mozilla.org/en-US/docs/Web/CSS/display) on the parent element of the elements you want to affect. In this case we want to lay out the `<article>` elements, so we set this on the `<section>` (which becomes a flex container):

```css
section {
  display: flex;
}
```

The result of this should be something like so:

So, this single declaration gives us everything we need — incredible, right? We have our multiple column layout with equal sized columns, and the columns are all the same height. This is because the default values given to flex items (the children of the flex container) are set up to solve common problems such as this. More on those later.

**Note**: You can also set a [`display`](https://developer.mozilla.org/en-US/docs/Web/CSS/display) value of `inline-flex` if you wish to lay out inline items as flexible boxes.

## An aside on the flex model

When elements are laid out as flexible boxes, they are laid out along two axes:

![flex_terms.png](https://developer.mozilla.org/files/3739/flex_terms.png)

- The **main axis** is the axis running in the direction the flex items are being laid out in (e.g. as rows across the page, or columns down the page.) The start and end of this axis are called the **main start** and **main end**.
- The **cross axis** is the axis running perpendicular to the direction the flex items are being laid out in. The start and end of this axis are called the **cross start** and **cross end**.
- The parent element that has `display: flex` set on it (the `<section>` in our example) is called the **flex container**.
- The items being laid out as flexible boxes inside the flex container are called **flex items** (the `<article>` elements in our example).

Bear this terminology in mind as you go through subsequent sections. You can always refer back to it if you get confused about any of the terms being used.

## Columns or rows?

Flexbox provides a property called [`flex-direction`](https://developer.mozilla.org/en-US/docs/Web/CSS/flex-direction) that specifies what direction the main axis runs in (what direction the flexbox children are laid out in) — by default this is set to `row`, which causes them to be laid out in a row in the direction your browser's default language works in (left to right, in the case of an English browser).

Try adding the following declaration to your `<section>` rule:

```css
flex-direction: column;
```

You'll see that this puts the items back in a column layout, much like they were before we added any CSS. Before you move on, delete this declaration from your example.

**Note**: You can also lay out flex items in a reverse direction using the `row-reverse` and `column-reverse` values. Experiment with these values too!

## Wrapping

One issue that arises when you have a fixed amount of width or height in your layout is that eventually your flexbox children will overflow their container, breaking the layout. Have a look at our [flexbox-wrap0.html](https://github.com/mdn/learning-area/blob/master/css/css-layout/flexbox/flexbox-wrap0.html) example, and try [viewing it live](http://mdn.github.io/learning-area/css/css-layout/flexbox/flexbox-wrap0.html) (take a local copy of this file now if you want to follow along with this example):

![img](https://mdn.mozillademos.org/files/13410/flexbox-example3.png)

Here we see that the children are indeed breaking out of their container. One way in which you can fix this is to add the following declaration to your `<section>` rule:

```css
flex-wrap: wrap;
```

Also, add the following declaration to your `<article>` rule:

```css
flex: 200px;
```

Try this now; you'll see that the layout looks much better with this included:

![img](https://mdn.mozillademos.org/files/13412/flexbox-example4.png)We now have multiple rows — as many flexbox children are fitted onto each row as makes sense, and any overflow is moved down to the next line. The `flex: 200px` declaration set on the articles means that each will be at least 200px wide; we'll discuss this property in more detail later on. You might also notice that the last few children on the last row are each made wider so that the entire row is still filled.

But there's more we can do here. First of all, try changing your [`flex-direction`](https://developer.mozilla.org/en-US/docs/Web/CSS/flex-direction) property value to `row-reverse` — now you'll see that you still have your multiple row layout, but it starts from the opposite corner of the browser window and flows in reverse.

## flex-flow shorthand

At this point it is worth noting that a shorthand exists for [`flex-direction`](https://developer.mozilla.org/en-US/docs/Web/CSS/flex-direction) and [`flex-wrap`](https://developer.mozilla.org/en-US/docs/Web/CSS/flex-wrap) — [`flex-flow`](https://developer.mozilla.org/en-US/docs/Web/CSS/flex-flow). So for example, you can replace

```css
flex-direction: row;
flex-wrap: wrap;
```

with

```css
flex-flow: row wrap;
```

## Flexible sizing of flex items

Let's now return to our first example, and look at how we can control what proportion of space flex items take up. Fire up your local copy of [flexbox0.html](https://github.com/mdn/learning-area/blob/master/css/css-layout/flexbox/flexbox0.html), or take a copy of [flexbox1.html](https://github.com/mdn/learning-area/blob/master/css/css-layout/flexbox/flexbox1.html) as a new starting point ([see it live](http://mdn.github.io/learning-area/css/css-layout/flexbox/flexbox1.html)).

First, add the following rule to the bottom of your CSS:

```css
article {
  flex: 1;
}
```

This is a unitless proportion value that dictates how much of the available space along the main axis each flex item will take up. In this case, we are giving each `<article>` element a value of 1, which means they will all take up an equal amount of the spare space left after things like padding and margin have been set. It is a proportion, meaning that giving each flex item a value of 400000 would have exactly the same effect.

Now add the following rule below the previous one:

```css
article:nth-of-type(3) {
  flex: 2;
}
```

Now when you refresh, you'll see that the third [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/article) takes up twice as much of the available width as the other two — there are now four proportion units available in total. The first two flex items have one each so they take 1/4 of the available space each. The third one has two units, so it takes up 2/4 of the available space (or 1/2).

You can also specify a minimum size value inside the flex value. Try updating your existing article rules like so:

```css
article {
  flex: 1 200px;
}

article:nth-of-type(3) {
  flex: 2 200px;
}
```

This basically states "Each flex item will first be given 200px of the available space. After that, the rest of the available space will be shared out according to the proportion units." Try refreshing and you'll see a difference in how the space is shared out.

![img](https://mdn.mozillademos.org/files/13406/flexbox-example1.png)

The real value of flexbox can be seen in its flexibility/responsiveness — if you resize the browser window, or add another `<article>` element, the layout continues to work just fine.

## flex: shorthand versus longhand

[`flex`](https://developer.mozilla.org/en-US/docs/Web/CSS/flex) is a shorthand property that can specify up to three different values:

- The unitless proportion value we discussed above. This can be specified individually using the [`flex-grow`](https://developer.mozilla.org/en-US/docs/Web/CSS/flex-grow) longhand property.
- A second unitless proportion value — [`flex-shrink`](https://developer.mozilla.org/en-US/docs/Web/CSS/flex-shrink) — that comes into play when the flex items are overflowing their container. This specifies how much of the overflowing amount is taken away from each flex item's size, to stop them overflowing their container. This is quite an advanced flexbox feature, and we won't be covering it any further in this article.
- The minimum size value we discussed above. This can be specified individually using the [`flex-basis`](https://developer.mozilla.org/en-US/docs/Web/CSS/flex-basis) longhand value.

We'd advise against using the longhand flex properties unless you really have to (for example, to override something previously set). They lead to a lot of extra code being written, and they can be somewhat confusing.

## Horizontal and vertical alignment

You can also use flexbox features to align flex items along the main or cross axis. Let's explore this by looking at a new example — [flex-align0.html](https://github.com/mdn/learning-area/blob/master/css/css-layout/flexbox/flex-align0.html) ([see it live also](http://mdn.github.io/learning-area/css/css-layout/flexbox/flex-align0.html)) — which we are going to turn into a neat, flexible button/toolbar. At the moment you'll see a horizontal menu bar, with some buttons jammed into the top left hand corner.

![img](https://mdn.mozillademos.org/files/13414/flexbox-example5.png)

First, take a local copy of this example.

Now, add the following to the bottom of the example's CSS:

```css
div {
  display: flex;
  align-items: center;
  justify-content: space-around;
}
```

Refresh the page and you'll see that the buttons are now nicely centered, horizontally and vertically. We've done this via two new properties.

[`align-items`](https://developer.mozilla.org/en-US/docs/Web/CSS/align-items) controls where the flex items sit on the cross axis.

- By default, the value is `stretch`, which stretches all flex items to fill the parent in the direction of the cross axis. If the parent hasn't got a fixed width in the cross axis direction, then all flex items will become as long as the longest flex items. This is how our first example got equal height columns by default.
- The `center` value that we used in our above code causes the items to maintain their intrinsic dimensions, but be centered along the cross axis. This is why our current example's buttons are centered vertically.
- You can also have values like `flex-start` and `flex-end`, which will align all items at the start and end of the cross axis respectively. See [`align-items`](https://developer.mozilla.org/en-US/docs/Web/CSS/align-items) for the full details.

You can override the [`align-items`](https://developer.mozilla.org/en-US/docs/Web/CSS/align-items) behavior for individual flex items by applying the [`align-self`](https://developer.mozilla.org/en-US/docs/Web/CSS/align-self) property to them. For example, try adding the following to your CSS:

```css
button:first-child {
  align-self: flex-end;
}
```

Have a look at what effect this has, and remove it again when you've finished.

[`justify-content`](https://developer.mozilla.org/en-US/docs/Web/CSS/justify-content) controls where the flex items sit on the main axis.

- The default value is `flex-start`, which makes all the items sit at the start of the main axis.
- You can use `flex-end` to make them sit at the end.
- `center` is also a value for `justify-content`, and will make the flex items sit in the center of the main axis.
- The value we've used above, `space-around`, is useful — it distributes all the items evenly along the main axis, with a bit of space left at either end.
- There is another value, `space-between`, which is very similar to `space-around` except that it doesn't leave any space at either end.

We'd like to encourage you to play with these values to see how they work before you continue.

## Ordering flex items

Flexbox also has a feature for changing the layout order of flex items, without affecting the source order. This is another thing that is impossible to do with traditional layout methods.

The code for this is simple: try adding the following CSS to your button bar example code:

```css
button:first-child {
  order: 1;
}
```

Refresh, and you'll now see that the "Smile" button has moved to the end of the main axis. Let's talk about how this works in a bit more detail:

- By default, all flex items have an [`order`](https://developer.mozilla.org/en-US/docs/Web/CSS/order) value of 0.
- Flex items with higher order values set on them will appear later in the display order than items with lower order values.
- Flex items with the same order value will appear in their source order. So if you have four items with order values of 2, 1, 1, and 0 set on them respectively, their display order would be 4th, 2nd, 3rd, then 1st.
- The 3rd item appears after the 2nd because it has the same order value and is after it in the source order.

You can set negative order values to make items appear earlier than items with 0 set. For example, you could make the "Blush" button appear at the start of the main axis using the following rule:

```css
button:last-child {
  order: -1;
}
```

## Nested flex boxes

It is possible to create some pretty complex layouts with flexbox. It is perfectly ok to set a flex item to also be a flex container, so that its children are also laid out like flexible boxes. Have a look at [complex-flexbox.html](https://github.com/mdn/learning-area/blob/master/css/css-layout/flexbox/complex-flexbox.html) ([see it live also](http://mdn.github.io/learning-area/css/css-layout/flexbox/complex-flexbox.html)).

![img](https://mdn.mozillademos.org/files/13418/flexbox-example7.png)

The HTML for this is fairly simple. We've got a [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/section) element containing three `<article>`s. The third `<article>` contains three `<div>`s. :

```html
section - article
          article
          article - div - button   
                    div   button
                    div   button
                          button
                          button
```

Let's look at the code we've used for the layout.

First of all, we set the children of the `<section>` to be laid out as flexible boxes.

```css
section {
  display: flex;
}
```

Next, we set some flex values on the `<article>`s themselves. Take special note of the 2nd rule here — we are setting the third `<article>` to have its children laid out like flex items too, but this time we are laying them out like a column.

```css
article {
  flex: 1 200px;
}

article:nth-of-type(3) {
  flex: 3 200px;
  display: flex;
  flex-flow: column;
}
```

Next, we select the first `<div>`. We first use `flex:1 100px;` to effectively give it a minimum height of 100px, then we set its children (the `<button>` elements) to also be laid out like flex items. Here we lay them out in a wrapping row, and align them in the center of the available space like we did in the individual button example we saw earlier.

```css
article:nth-of-type(3) div:first-child {
  flex:1 100px;
  display: flex;
  flex-flow: row wrap;
  align-items: center;
  justify-content: space-around;
}
```

Finally, we set some sizing on the button, but more interestingly we give it a flex value of 1 auto. This has a very interesting effect, which you'll see if you try resizing your browser window width. The buttons will take up as much space as they can and sit as many on the same line as they can, but when they can no longer fit comfortably on the same line, they'll drop down to create new lines.

```css
button {
  flex: 1 auto;
  margin: 5px;
  font-size: 18px;
  line-height: 1.5;
}
```

## Cross browser compatibility

Flexbox support is available in most new browsers — Firefox, Chrome, Opera, Microsoft Edge and IE 11, newer versions of Android/iOS, etc. However you should be aware that there are still older browsers in use that don't support Flexbox (or do, but support a really old, out-of-date version of it.)

While you are just learning and experimenting, this doesn't matter too much; however if you are considering using flexbox in a real website you need to do testing and make sure that your user experience is still acceptable in as many browsers as possible.

Flexbox is a bit trickier than some CSS features. For example, if a browser is missing a CSS drop shadow, then the site will likely still be usable. Not supporting flexbox features however will probably break a layout completely, making it unusable.

We discuss strategies for overcoming cross browser support issues in our [Cross browser testing](https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Cross_browser_testing) module.

## Summary

That concludes our tour of the basics of flexbox. We hope you had fun, and will have a good play around with it as you travel forward with your learning. Next we'll have a look at another important aspect of CSS layouts — CSS Grids.

# Grids

CSS Grid Layout is a two-dimensional layout system for the web. It lets you lay content out in rows and columns, and has many features that make building complex layouts straightforward. This article will give you all you need to know to get started with page layout.

## What is grid layout?

A grid is a collection of horizontal and vertical lines creating a pattern against which we can line up our design elements. They help us to create designs where elements don’t jump around or change width as we move from page to page, providing greater consistency on our websites.

A grid will typically have **columns**, **rows**, and then gaps between each row and column — commonly referred to as **gutters**.

![img](https://mdn.mozillademos.org/files/13899/grid.png)

## Creating your grid in CSS

Having decided on the grid that your design needs, you can use CSS Grid Layout to create that grid in CSS and place items onto it. We will look at the basic features of Grid Layout first and then explore how to create a simple grid system for your project.

### Defining a grid



As a starting point, download and open [the starting point file](https://github.com/mdn/learning-area/blob/master/css/css-layout/grids/0-starting-point.html) in your text editor and browser (you can also[ see it live here](https://mdn.github.io/learning-area/css/css-layout/grids/0-starting-point.html)). You will see an example with a container, which has some child items. By default these display in normal flow so the boxes display one below the other. We will be working with this file for the first part of this lesson, making changes to see how grid behaves.

To define a grid we use the `grid` value of the [`display`](https://developer.mozilla.org/en-US/docs/Web/CSS/display) property. As with Flexbox, this switches on Grid Layout, and all of the direct children of the container become grid items. Add this to the CSS inside your file:

```css
.container {
    display: grid;
}
```

Unlike flexbox, the items will not immediately look any different. Declaring `display: grid` gives you a one column grid, so your items will continue to display one below the other as they do in normal flow.

To see something that looks more grid-like, we will need to add some columns to the grid. Let's add three 200-pixel columns here. You can use any length unit, or percentages to create these column tracks.

```css
.container {
    display: grid;
    grid-template-columns: 200px 200px 200px;
}
```

Add the 2nd declaration to your CSS rule, then reload the page, and you should see that the items have rearranged themselves one into each cell of the created grid.

### Flexible grids with the fr unit



In addition to creating grids using lengths and percentages, we can use the `fr` unit to flexibly size grid rows and columns. This unit represents one fraction of the available space in the grid container.

Change your track listing to the following definition, creating three `1fr` tracks.

```css
.container {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
}
```

You should now see that you have flexible tracks. The `fr` unit distributes space in proportion, therefore you can give different positive values to your tracks, for example if you change the definition like so:

```css
.container {
    display: grid;
    grid-template-columns: 2fr 1fr 1fr;
}
```

The first track now gets `2fr` of the available space and the other two tracks get `1fr`, making the first track larger. You can mix `fr` units and fixed length tracks — in such a case the space needed for the fixed tracks is taken away before the space is distributed to the other tracks.

**Note**: The `fr` unit distributes *available* space, not *all* space. Therefore if one of your tracks has something large inside it there will be less free space to share out.

### Gaps between tracks



To create gaps between tracks we use the properties [`grid-column-gap`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-column-gap) for gaps between columns, [`grid-row-gap`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-row-gap) for gaps between rows, and [`grid-gap`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-gap) to set both at once.

```css
.container {
    display: grid;
    grid-template-columns: 2fr 1fr 1fr;
    grid-gap: 20px;
}
```

These gaps can be any length unit or a percentage, but not an `fr` unit.

**Note**: The `*gap` properties used to be prefixed by `grid-`, but this has been changed in the spec, as the intention is to make them usable in multiple layout methods. At the moment, Edge and Firefox support the non-prefixed versions, and the prefixed versions will be maintained as an alias so will be safe to use for some time. To be on the safe side, you could double up and add both properties to make your code more bulletproof.

```css
.container {
  display: grid;
  grid-template-columns: 2fr 1fr 1fr;
  grid-gap: 20px;
  gap: 20px;
}
```

### Repeating track listings



You can repeat all, or a section of, your track listing using repeat notation. Change your track listing to the following:

```css
.container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-gap: 20px;
}
```

You will now get 3 `1fr` tracks just as before. The first value passed to the repeat function is how many times you want the listing to repeat, while the second value is a track listing, which may be one or more tracks that you want to repeat.

### The implicit and explicit grid



We have only specified column tracks so far, and yet rows are being created to hold our content. This is an example of the explicit versus the implicit grid. The explicit grid is the one that you create using `grid-template-columns` or `grid-template-rows`. The implicit grid is created when content is placed outside of that grid — such as into our rows. The explicit and implicit grids are analogous to the main and cross flexbox axes.

By default, tracks created in the implicit grid are `auto` sized, which in general means that they are large enough to fit their content. If you wish to give implicit grid tracks a size you can use the [`grid-auto-rows`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-auto-rows) and [`grid-auto-columns`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-auto-columns) properties. If you add `grid-auto-rows` with a value of `100px` to your CSS, you will see that those created rows are now 100 pixels tall.

```css
.container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-auto-rows: 100px;
  grid-gap: 20px;
}
```

### The minmax() function



Our 100-pixel tall tracks won’t be very useful if we add content into those tracks that is taller than 100 pixels, in which case it would cause an overflow. It might be better to have tracks that are *at least* 100 pixels tall and can still expand if more content gets into them. A fairly basic fact about the web is that you never really know how tall something is going to be; additional content or larger font sizes can cause problems with designs that attempt to be pixel perfect in every dimension.

The `minmax` function lets us set a minimum and maximum size for a track, for example `minmax(100px, auto)`. The minimum size is 100 pixels, but the maximum is `auto`, which will expand to fit the content. Try changing `grid-auto-rows` to use a minmax value:

```css
.container {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-auto-rows: minmax(100px, auto);
    grid-gap: 20px;
}
```

If you add extra content you will see that the track expands to allow it to fit. Note that the expansion happens right along the row.

### As many columns as will fit



We can combine some of the things we have learned about track listing, repeat notation and `minmax()` to create a useful pattern. Sometimes it is helpful to be able to ask grid to create as many columns as will fit into the container. We do this by setting the value of `grid-template-columns` using `repeat()` notation, but instead of passing in a number, pass in the keyword `auto-fill`. For the second parameter of the function we use `minmax()`, with a minimum value equal to the minimum track size that we would like to have, and a maximum of `1fr`.

Try this in your file now, using the below CSS:

```css
.container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  grid-auto-rows: minmax(100px, auto);
  grid-gap: 20px;
}
```

This works because grid is creating as many 200 pixel columns as will fit into the container, then sharing whatever space is leftover between all of the columns — the maximum is 1fr which, as we already know, distributes space evenly between tracks.

## Line-based placement

We now move on from creating a grid, to placing things on the grid. Our grid always has lines, these lines start at 1 and relate to the Writing Mode of the document. Therefore in English, column line 1 is on the left hand side of the grid and row line 1 at the top. In Arabic column line 1 would be on the right hand side, as Arabic is written right to left.

We can place things according to these lines by specifying the start and end line. We do this using the following properties:

- [`grid-column-start`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-column-start)
- [`grid-column-end`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-column-end)
- [`grid-row-start`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-row-start)
- [`grid-row-end`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-row-end)

These properties can all have a line number as the value. You can also use the shorthand properties:

- [`grid-column`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-column)
- [`grid-row`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-row)

These let you specify the start and end lines at once, separated by a `/` — a forward slash character.

[Download this file as a starting point](https://github.com/mdn/learning-area/blob/master/css/css-layout/grids/8-placement-starting-point.html) or [see it live here](https://mdn.github.io/learning-area/css/css-layout/grids/8-placement-starting-point.html). It has a grid defined already, and a simple article outlined. You can see that auto-placement is placing items one into each cell of the grid that we have created.

We will instead place all of the elements for our site on the grid, using the grid lines. Add the following rules to the bottom of your CSS:

```css
header {
  grid-column: 1 / 3;
  grid-row: 1;
}

article {
  grid-column: 2;
  grid-row: 2;
}

aside {
  grid-column: 1;
  grid-row: 2;
}

footer {
  grid-column: 1 / 3;
  grid-row: 3;
}
```

**Note**: you can also use the value `-1` to target the end column or row line, and count inwards from the end using negative values. However this only works for the Explicit Grid. The value `-1` will not target the end line of the implicit grid.

## Positioning with grid-template-areas

An alternative way to place items on your grid is to use the [`grid-template-areas`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template-areas) property and giving the various elements of your design a name.

Remove the line-based positioning from the last example (or re-download the file to have a fresh starting point), and add the following CSS.

```css
.container {
  display: grid;
  grid-template-areas: 
      "header header"
      "sidebar content"
      "footer footer";
  grid-template-columns: 1fr 3fr;
  grid-gap: 20px;
}

header {
  grid-area: header;
}

article {
  grid-area: content;
}

aside {
  grid-area: sidebar;
}

footer {
  grid-area: footer;
}
```

Reload the page and you will see that your items have been placed just as before without us needing to use any line numbers!

The rules for `grid-template-areas` are as follows:

- You need to have every cell of the grid filled.
- To span across two cells, repeat the name.
- To leave a cell empty, use a `.` (period).
- Areas must be rectangular — you can’t have an L-shaped area for example.
- Areas can't be repeated in different locations.

You can play around with our layout, changing the footer to only sit underneath the content and the sidebar to span all the way down for example. This is a very nice way to describe a layout as it is obvious from the CSS exactly what is happening.

## A CSS Grid, grid framework

Grid "frameworks" tend to be based around 12 or 16 column grids and with CSS Grid, you don’t need any third party tool to give you such a framework — it's already there in the spec.

[Download the starting point file](https://github.com/mdn/learning-area/blob/master/css/css-layout/grids/11-grid-system-starting-point.html). This contains a container with a 12 column grid defined, and the same markup as we used in the previous two examples. We can now use line-based placement to place our content on the 12 column grid.

```css
header {
  grid-column: 1 / 13;
  grid-row: 1;
}

article {
  grid-column: 4 / 13;
  grid-row: 2;
}

aside {
  grid-column: 1 / 4;
  grid-row: 2;
}

footer {
  grid-column: 1 / 13;
  grid-row: 3;
}
```

If you use the [Firefox Grid Inspector](https://developer.mozilla.org/en-US/docs/Tools/Page_Inspector/How_to/Examine_grid_layouts) to overlay the grid lines on your design, you can see how our 12 column grid works.

![A 12 column grid overlaid on our design.](https://mdn.mozillademos.org/files/16045/learn-grids-inspector.png)

## Summary

In this overview we have toured the main features of CSS Grid Layout. You should be able to start using it in your designs. To dig further into the specification, read our guides to Grid Layout, which can be found below.

# Floats

Originally for floating images inside blocks of text, the [`float`](https://developer.mozilla.org/en-US/docs/Web/CSS/float) property became one of the most commonly used tools for creating multiple column layouts on webpages. With the advent of flexbox and grid it has now returned to its original purpose, as this article explains.

## The background of floats

The [`float`](https://developer.mozilla.org/en-US/docs/Web/CSS/float) property was introduced to allow web developers to implement simple layouts involving an image floating inside a column of text, with the text wrapping around the left or right of it. The kind of thing you might get in a newspaper layout.

But web developers quickly realized that you can float anything, not just images, so the use of float broadened. Our [fancy paragraph example](https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS/Pseudo-classes_and_pseudo-elements#Active_learning_A_fancy_paragraph) from earlier in the course shows how you can use float in the creation of a fun drop-cap effect.

Floats have commonly been used to create entire web site layouts featuring multiple columns of information floated so they sit alongside one another (the default behavior would be for the columns to sit below one another, in the same order as they appear in the source). There are newer, better layout techniques available and so use of floats in this way should be regarded as a [legacy technique](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Legacy_Layout_Methods).

In this article we'll just concentrate on the proper uses of floats.

## A simple float example

Let's explore how to use floats. We'll start with a really simple example involving floating a block of text around an element. You can follow along by creating a new `index.html` file on your computer, filling it with a [simple HTML template](https://github.com/mdn/learning-area/blob/master/html/introduction-to-html/getting-started/index.html), and inserting the below code into it at the appropriate places. At the bottom of the section, you can see a live example of what the final code should look like.

First, we'll start off with some simple HTML — add the following to your HTML body, removing anything that was inside there before:

```html
<h1>Simple float example</h1>

<div class="box">Float</div>

<p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla luctus aliquam dolor, eu lacinia lorem placerat vulputate. Duis felis orci, pulvinar id metus ut, rutrum luctus orci. Cras porttitor imperdiet nunc, at ultricies tellus laoreet sit amet. </p>
    
<p>Sed auctor cursus massa at porta. Integer ligula ipsum, tristique sit amet orci vel, viverra egestas ligula. Curabitur vehicula tellus neque, ac ornare ex malesuada et. In vitae convallis lacus. Aliquam erat volutpat. Suspendisse ac imperdiet turpis. Aenean finibus sollicitudin eros pharetra congue. Duis ornare egestas augue ut luctus. Proin blandit quam nec lacus varius commodo et a urna. Ut id ornare felis, eget fermentum sapien.</p>

<p>Nam vulputate diam nec tempor bibendum. Donec luctus augue eget malesuada ultrices. Phasellus turpis est, posuere sit amet dapibus ut, facilisis sed est. Nam id risus quis ante semper consectetur eget aliquam lorem. Vivamus tristique elit dolor, sed pretium metus suscipit vel. Mauris ultricies lectus sed lobortis finibus. Vivamus eu urna eget velit cursus viverra quis vestibulum sem. Aliquam tincidunt eget purus in interdum. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.</p>
```

Now apply the following CSS to your HTML (using a `<style>` element or a `<link>` to a separate `.css` file — your choice):

```css
body {
  width: 90%;
  max-width: 900px;
  margin: 0 auto;
  font: .9em/1.2 Arial, Helvetica, sans-serif
}

.box {
  width: 150px;
  height: 100px;
  border-radius: 5px;
  background-color: rgb(207,232,220);
  padding: 1em;
}
```

If you save and refresh now, you'll see something much like what you'd expect — the box is sitting above the text, in normal flow. To float the text around it add two properties to the `.box` rule:

```css
.box {
  float: left;
  margin-right: 15px;
  width: 150px;
  height: 100px;
  border-radius: 5px;
  background-color: rgb(207,232,220);
  padding: 1em;
}
```

Now if you save and refresh you'll see something like the following:

So let's think about how the float works — the element with the float set on it (the `<div>` element in this case) is taken out of the normal layout flow of the document and stuck to the left-hand side of its parent container (`<body>`, in this case). Any content that comes below the floated element in the normal layout flow will now wrap around it, filling up the space to the right-hand side of it as far up as the top of the floated element. There, it will stop.

Floating the content to the right has exactly the same effect, but in reverse — the floated element will stick to the right, and the content will wrap around it to the left. Try changing the float value to `right` and replace [`margin-right`](https://developer.mozilla.org/en-US/docs/Web/CSS/margin-right) with [`margin-left`](https://developer.mozilla.org/en-US/docs/Web/CSS/margin-left) in the last ruleset to see what the result is.

While we can add a margin to the float to push the text away, we can't add a margin to the text to move it away from the float. This is because a floated element is taken out of normal flow, and the boxes of the following items actually run behind the float. You can demonstrate this by making some changes to your example.

Add a class of `special` to the first paragraph of text, the one immediately following the floated box, then in your CSS add the following rules. These will give our following paragraph a background color.

```css
.special {
  background-color: rgb(79,185,227);
  padding: 10px;
  color: #fff;
}
```

To make the effect easier to see, change the `margin-right` on your float to `margin`, so you get space all around the float. You will be able to see the background on the paragraph running right underneath the floated box, as in the example below.

The [line boxes](https://developer.mozilla.org/en-US/docs/Web/CSS/Visual_formatting_model#Line_boxes) of our following element have been shortened so the text runs around the float, but due to the float being removed from normal flow the box around the paragraph still remains full width.

## Clearing floats

We have seen that the float is removed from normal flow and that other elements will display beside it, therefore if we want to stop the following element from moving up we need to clear it; this is achieved with the [`clear`](https://developer.mozilla.org/en-US/docs/Web/CSS/clear) property.

In your HTML from the previous example, add a class of `cleared` to the second paragraph below the floated item. Then add the following to your CSS:

```css
.cleared {
  clear: left;
}
```

You should see that the following paragraph clears the floated element and no longer comes up alongside it. The `clear` property accepts the following values:

- `left`: Clear items floated to the left.
- `right`: Clear items floated to the right.
- `both`: Clear any floated items, left or right.

## Clearing boxes wrapped around a float

You now know how to clear something following a floated element, but let's see what happens if you have a tall float and a short paragraph, with a box wrapped around both elements. Change your document so that the first paragraph and our floated box are wrapped with a `<div>` with a class of `wrapper`.

```html
<div class="wrapper">
  <div class="box">Float</div>

  <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla luctus aliquam dolor, eu lacinia lorem placerat vulputate.</p>
</div>
```

In your CSS, add the following rule for the `.wrapper` class and then reload the page:

```css
.wrapper {
  background-color: rgb(79,185,227);
  padding: 10px;
  color: #fff; 
}
```

In addition, remove the original `.cleared` class:

```css
.cleared {
    clear: left;
}
```

You will see that, just like in the example where we put a background color on the paragraph, the background color runs behind the float.

Once again, this is because the float has been taken out of normal flow. Clearing the following element doesn't help with this box clearing problem, where you want the bottom of the box to wrap the floated item and wrapping content even if the content is shorter. There are three potential ways to deal with this, two of which work in all browsers — yet are slightly hacky — and a third new way that deals with this situation properly.

### The clearfix hack



The way that this situation has traditionally been dealt with is to use something known as a "clearfix hack". This involves inserting some generated content after the box which contains the float and wrapping content, and setting that to clear both.

Add the following CSS to our example:

```css
.wrapper::after {
  content: "";
  clear: both;
  display: block;
}
```

Now reload the page and the box should clear. This is essentially the same as if you had added an HTML element such as a `<div>` below the items and set it to `clear: both`.

### Using overflow



An alternative method is to set the [`overflow`](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow) property of the wrapper to a value other than `visible`.

Remove the clearfix CSS you added in the last section, and instead add `overflow: auto` to the rules for wrapper. Once again, the box should clear.

```css
.wrapper {
  background-color: rgb(79,185,227);
  padding: 10px;
  color: #fff;
  overflow: auto; 
}
```

This example works by creating what is known as a **block formatting context** (BFC). This is like a mini layout inside your page, inside which everything is contained, therefore our floated element is contained inside the BFC and the background runs behind both items. This will usually work, however, in certain cases you might find unwanted scrollbars or clipped shadows due to unintended consequences of using overflow.

### display: flow-root



The modern way of solving this problem is to use the value `flow-root` of the `display` property. This exists only to create a BFC without using hacks — there will be no unintended consequences when you use it. Remove `overflow: auto` from your `.wrapper` rule and add `display: flow-root`. Assuming you have a [supporting browser](https://developer.mozilla.org/en-US/docs/Web/CSS/display#Browser_compatibility), the box will clear.

```css
.wrapper {
  background-color: rgb(79,185,227);
  padding: 10px;
  color: #fff;
  display: flow-root; 
}
```

## Summary

You now know all there is to know about floats in modern web development. See the article on [legacy layout methods](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Legacy_Layout_Methods) for information on how they used to be used, which may be useful if you find yourself working on older projects.

# Positioning

Positioning allows you to take elements out of the normal document layout flow, and make them behave differently; for example sitting on top of one another, or always remaining in the same place inside the browser viewport. This article explains the different [`position`](https://developer.mozilla.org/en-US/docs/Web/CSS/position) values, and how to use them.

We'd like you to follow along with the exercises on your local computer, if possible — grab a copy of `0_basic-flow.html` from our GitHub repo ([source code here](https://github.com/mdn/learning-area/blob/master/css/css-layout/positioning/0_basic-flow.html)) and use that as a starting point.

## Introducing positioning

The whole idea of positioning is to allow us to override the basic document flow behavior described above, to produce interesting effects. What if you want to slightly alter the position of some boxes inside a layout from their default layout flow position, to give a slightly quirky, distressed feel? Positioning is your tool. Or if you want to create a UI element that floats over the top of other parts of the page, and/or always sits in the same place inside the browser window no matter how much the page is scrolled? Positioning makes such layout work possible.

There are a number of different types of positioning that you can put into effect on HTML elements. To make a specific type of positioning active on an element, we use the [`position`](https://developer.mozilla.org/en-US/docs/Web/CSS/position) property.

### Static positioning



Static positioning is the default that every element gets — it just means "put the element into its normal position in the document layout flow — nothing special to see here."

To demonstrate this, and get your example set up for future sections, first add a `class` of `positioned` to the second `<p>` in the HTML:

```html
<p class="positioned"> ... </p>
```

Now add the following rule to the bottom of your CSS:

```css
.positioned {
  position: static;
  background: yellow;
}
```

If you now save and refresh, you'll see no difference at all, except for the updated background color of the 2nd paragraph. This is fine — as we said before, static positioning is the default behavior!

**Note**: You can see the example at this point live at `1_static-positioning.html` ([see source code](https://github.com/mdn/learning-area/blob/master/css/css-layout/positioning/1_static-positioning.html)).

### Relative positioning



Relative positioning is the first position type we'll take a look at. This is very similar to static positioning, except that once the positioned element has taken its place in the normal layout flow, you can then modify its final position, including making it overlap other elements on the page. Go ahead and update the `position` declaration in your code:

```css
position: relative;
```

If you save and refresh at this stage, you won't see a change in the result at all. So how do you modify the element's position? You need to use the [`top`](https://developer.mozilla.org/en-US/docs/Web/CSS/top), [`bottom`](https://developer.mozilla.org/en-US/docs/Web/CSS/bottom), [`left`](https://developer.mozilla.org/en-US/docs/Web/CSS/left), and [`right`](https://developer.mozilla.org/en-US/docs/Web/CSS/right) properties, which we'll explain in the next section.

### Introducing top, bottom, left, and right



[`top`](https://developer.mozilla.org/en-US/docs/Web/CSS/top), [`bottom`](https://developer.mozilla.org/en-US/docs/Web/CSS/bottom), [`left`](https://developer.mozilla.org/en-US/docs/Web/CSS/left), and [`right`](https://developer.mozilla.org/en-US/docs/Web/CSS/right) are used alongside [`position`](https://developer.mozilla.org/en-US/docs/Web/CSS/position) to specify exactly where to move the positioned element to. To try this out, add the following declarations to the `.positioned` rule in your CSS:

```css
top: 30px;
left: 30px;
```

**Note**: The values of these properties can take any [units](https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS/Values_and_units) you'd logically expect — pixels, mm, rems, %, etc.

If you now save and refresh, you'll get a result something like this:

Cool, huh? Ok, so this probably wasn't what you were expecting — why has it moved to the bottom and right if we specified top and left? Illogical as it may initially sound, this is just the way that relative positioning works — you need to think of an invisible force that pushes the specified side of the positioned box, moving it in the opposite direction. So for example, if you specify `top: 30px;`, a force pushes the top of the box, causing it to move downwards by 30px.

**Note**: You can see the example at this point live at `2_relative-positioning.html` ([see source code](https://github.com/mdn/learning-area/blob/master/css/css-layout/positioning/2_relative-positioning.html)).

### Absolute positioning



Absolute positioning brings very different results. Let's try changing the position declaration in your code as follows:

```css
position: absolute;
```

If you now save and refresh, you should see something like so:

First of all, note that the gap where the positioned element should be in the document flow is no longer there — the first and third elements have closed together like it no longer exists! Well, in a way, this is true. An absolutely positioned element no longer exists in the normal document layout flow. Instead, it sits on its own layer separate from everything else. This is very useful: it means that we can create isolated UI features that don't interfere with the position of other elements on the page. For example, popup information boxes and control menus; rollover panels; UI features that can be dragged and dropped anywhere on the page; and so on...

Second, notice that the position of the element has changed — this is because [`top`](https://developer.mozilla.org/en-US/docs/Web/CSS/top), [`bottom`](https://developer.mozilla.org/en-US/docs/Web/CSS/bottom), [`left`](https://developer.mozilla.org/en-US/docs/Web/CSS/left), and [`right`](https://developer.mozilla.org/en-US/docs/Web/CSS/right) behave in a different way with absolute positioning. Instead of specifying the direction the element should move in, they specify the distance the element should be from each containing element's sides. So in this case, we are saying that the absolutely positioned element should sit 30px from the top of the "containing element", and 30px from the left.

**Note**: You can use [`top`](https://developer.mozilla.org/en-US/docs/Web/CSS/top), [`bottom`](https://developer.mozilla.org/en-US/docs/Web/CSS/bottom), [`left`](https://developer.mozilla.org/en-US/docs/Web/CSS/left), and [`right`](https://developer.mozilla.org/en-US/docs/Web/CSS/right) to resize elements if you need to. Try setting `top: 0; bottom: 0; left: 0; right: 0;` and `margin: 0;` on your positioned elements and see what happens! Put it back again afterwards...

**Note**: Yes, margins still affect positioned elements. Margin collapsing doesn't, however.

**Note**: You can see the example at this point live at `3_absolute-positioning.html` ([see source code](https://github.com/mdn/learning-area/blob/master/css/css-layout/positioning/3_absolute-positioning.html)).

### Positioning contexts



Which element is the "containing element" of an absolutely positioned element? This is very much dependent on the position property of the ancestors of the positioned element (See [Identifying the containing block](https://developer.mozilla.org/en-US/docs/Web/CSS/Containing_block#Identifying_the_containing_block)). 

If no ancestor elements have their position property explicitly defined, then by default all ancestor elements will have a static position. The result of this is, the absolutely positioned element will be contained in the **initial containing block**. The initial containing block has the dimensions of the viewport, and is also the block that contains the `<html>` element. Simply put, the absolutely positioned element will be contained outside of the `<html>` element, and be positioned relative to the initial viewport.

The positioned element is nested inside the `<body>` in the HTML source, but in the final layout, it is 30px away from the top and left of the edge of the page. We can change the **positioning context** — which element the absolutely positioned element is positioned relative to. This is done by setting positioning on one of the element's ancestors — to one of the elements it is nested inside (you can't position it relative to an element it is not nested inside). To demonstrate this, add the following declaration to your `body` rule:

```css
position: relative;
```

This should give the following result:

The positioned element now sits relative to the `<body>` element.

**Note**: You can see the example at this point live at `4_positioning-context.html` ([see source code](https://github.com/mdn/learning-area/blob/master/css/css-layout/positioning/4_positioning-context.html)).

### Introducing z-index



All this absolute positioning is good fun, but there is another thing we haven't considered yet — when elements start to overlap, what determines which elements appear on top of which other elements? In the example we've seen so far, we only have one positioned element in the positioning context, and it appears on the top, since positioned elements win over non-positioned elements. What about when we have more than one?

Try adding the following to your CSS, to make the first paragraph absolutely positioned too:

```css
p:nth-of-type(1) {
  position: absolute;
  background: green;
  top: 10px;
  right: 30px;
}
```

At this point you'll see the first paragraph colored green, moved out of the document flow, and positioned a bit above from where it originally was. It is also stacked below the original `.positioned` paragraph, where the two overlap. This is because the `.positioned` paragraph is the second paragraph in the source order, and positioned elements later in the source order win over positioned elements earlier in the source order.

Can you change the stacking order? Yes, you can, by using the [`z-index`](https://developer.mozilla.org/en-US/docs/Web/CSS/z-index) property. "z-index" is a reference to the z-axis. You may recall from previous points in the course where we discussed web pages using horizontal (x-axis) and vertical (y-axis) coordinates to work out positioning for things like background images and drop shadow offsets. (0,0) is at the top left of the page (or element), and the x- and y-axes run across to the right and down the page (for left to right languages, anyway.)

Web pages also have a z-axis: an imaginary line that runs from the surface of your screen, towards your face (or whatever else you like to have in front of the screen). [`z-index`](https://developer.mozilla.org/en-US/docs/Web/CSS/z-index) values affect where positioned elements sit on that axis; positive values move them higher up the stack, and negative values move them lower down the stack. By default, positioned elements all have a `z-index` of `auto`, which is effectively 0.

To change the stacking order, try adding the following declaration to your `p:nth-of-type(1)` rule:

```css
z-index: 1;
```

You should now see the finished example, with the green paragraph on top:

Note that `z-index` only accepts unitless index values; you can't specify that you want one element to be 23 pixels up the Z-axis — it doesn't work like that. Higher values will go above lower values, and it is up to you what values you use. Using 2 and 3 would give the same effect as 300 and 40000.

**Note**: You can see the example at this point live at `5_z-index.html` ([see source code](https://github.com/mdn/learning-area/blob/master/css/css-layout/positioning/5_z-index.html)).

### Fixed positioning



Let's now look at fixed positioning. This works in exactly the same way as absolute positioning, with one key difference: whereas absolute positioning fixes an element in place relative to the `<html>` element or its nearest positioned ancestor, fixed positioning fixes an element in place relative to the browser viewport itself. This means that you can create useful UI items that are fixed in place, like persisting navigation menus.

Let's put together a simple example to show what we mean. First of all, delete the existing `p:nth-of-type(1)` and `.positioned` rules from your CSS.

Now, update the `body` rule to remove the `position: relative;` declaration and add a fixed height, like so:

```css
body {
  width: 500px;
  height: 1400px;
  margin: 0 auto;
}
```

Now we're going to give the `<h1>` element `position: fixed;`, and get it to sit at the top center of the viewport. Add the following rule to your CSS:

```css
h1 {
  position: fixed;
  top: 0;
  width: 500px;
  margin: 0 auto;
  background: white;
  padding: 10px;
}
```

The `top: 0;` is required to make it stick to the top of the screen; we then give the heading the same width as the content column and use the faithful old `margin: 0 auto;` trick to center it. We then give it a white background and some padding, so the content won't be visible underneath it.

If you save and refresh now, you'll see a fun little effect whereby the heading stays fixed, and the content appears to scroll up and disappear underneath it. But we could improve this more — at the moment some of the content starts off underneath the heading. This is because the positioned heading no longer appears in the document flow, so the rest of the content moves up to the top. We need to move it all down a bit; we can do this by setting some top margin on the first paragraph. Add this now:

```css
p:nth-of-type(1) {
  margin-top: 60px;
}
```

You should now see the finished example:

### position: sticky



There is another position value available called `position: sticky`, which is somewhat newer than the others. This is basically a hybrid between relative and fixed position, which allows a positioned element to act like it is relatively positioned until it is scrolled to a certain threshold point (e.g. 10px from the top of the viewport), after which it becomes fixed. This can be used to for example cause a navigation bar to scroll with the page until a certain point, and then stick to the top of the page. 

```css
.positioned {
  position: sticky;
  top: 30px;
  left: 30px;
}
```

An interesting and common use of `position: sticky` is to create a scrolling index page where different headings stick to the top of the page as they reach it. The markup for such an example might look like so:

```html
<h1>Sticky positioning</h1>

<dl>
    <dt>A</dt>
    <dd>Apple</dd>
    <dd>Ant</dd>
    <dd>Altimeter</dd>
    <dd>Airplane</dd>
    <dt>B</dt>
    <dd>Bird</dd>
    <dd>Buzzard</dd>
    <dd>Bee</dd>
    <dd>Banana</dd>
    <dd>Beanstalk</dd>
    <dt>C</dt>
    <dd>Calculator</dd>
    <dd>Cane</dd>
    <dd>Camera</dd>
    <dd>Camel</dd>
    <dt>D</dt>
    <dd>Duck</dd>
    <dd>Dime</dd>
    <dd>Dipstick</dd>
    <dd>Drone</dd>
    <dt>E</dt>
    <dd>Egg</dd>
    <dd>Elephant</dd>
    <dd>Egret</dd>
</dl>
```

The CSS might look as follows. In normal flow the `<dt>` elements will scroll with the content. When we add `position: sticky` to the `<dt>` element, along with a [`top`](https://developer.mozilla.org/en-US/docs/Web/CSS/top) value of 0, supporting browsers will stick the headings to the top of the viewport as they reach that position. Each subsequent header will then replace the previous one as it scrolls up to that position.

```css
dt {
  background-color: black;
  color: white;
  padding: 10px;
  position: sticky;
  top: 0;
  left: 0;
  margin: 1em 0;
}
```

## Summary

I'm sure you had fun playing with basic positioning; while it is not a method you would use for entire layouts, as you can see there are many tasks it is suited for.

# Multiple-column layout

The multiple-column layout specification gives you a method of laying content out in columns, as you might see in a newspaper. This article explains how to use this feature.

## A basic example

We will now explore how to use multiple-column layout, often referred to as *multicol*. You can follow along by [downloading the multicol starting point file](https://github.com/mdn/learning-area/blob/master/css/css-layout/multicol/0-starting-point.html) and adding the CSS into the appropriate places. At the bottom of the section you can see a live example of what the final code should look like.

Our starting point contains some very simple HTML; a wrapper with a class of `container` inside which is a heading and some paragraphs.

The `<div>` with a class of container will become our multicol container. We switch on multicol by using one of two properties [`column-count`](https://developer.mozilla.org/en-US/docs/Web/CSS/column-count) or [`column-width`](https://developer.mozilla.org/en-US/docs/Web/CSS/column-width). The `column-count` property will create as many columns as the value you give it, so if you add the following CSS to your stylesheet and reload the page, you will get three columns:

```css
.container {
  column-count: 3;
}
```

The columns that you create have flexible widths — the browser works out how much space to assign each column.

```html
<div class="container">
  <h1>Simple multicol example</h1>
    
  <p> Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla luctus aliquam dolor, eu lacinia lorem placerat vulputate.
  Duis felis orci, pulvinar id metus ut, rutrum luctus orci. Cras porttitor imperdiet nunc, at ultricies tellus laoreet sit amet. Sed auctor cursus massa at porta. Integer ligula ipsum, tristique sit amet orci vel, viverra egestas ligula.
  Curabitur vehicula tellus neque, ac ornare ex malesuada et. In vitae convallis lacus. Aliquam erat volutpat. Suspendisse
  ac imperdiet turpis. Aenean finibus sollicitudin eros pharetra congue. Duis ornare egestas augue ut luctus. Proin blandit
  quam nec lacus varius commodo et a urna. Ut id ornare felis, eget fermentum sapien.</p>
    
  <p>Nam vulputate diam nec tempor bibendum. Donec luctus augue eget malesuada ultrices. Phasellus turpis est, posuere sit amet dapibus ut, facilisis sed est. Nam id risus quis ante semper consectetur eget aliquam lorem. Vivamus tristique
  elit dolor, sed pretium metus suscipit vel. Mauris ultricies lectus sed lobortis finibus. Vivamus eu urna eget velit
  cursus viverra quis vestibulum sem. Aliquam tincidunt eget purus in interdum. Cum sociis natoque penatibus et magnis
  dis parturient montes, nascetur ridiculus mus.</p>
</div>
.container {
  column-count: 3;
}
```

Change your CSS to use `column-width` as follows:

```css
.container {
  column-width: 200px;
}
```

The browser will now give you as many columns as it can of the size that you specify; any remaining space is then shared between the existing columns. This means that you will not get exactly the width that you specify, unless your container is exactly divisible by that width.

```css
.container {
  column-width: 200px;
}  
```

## Styling the columns

The columns created by multicol cannot be styled individually. There is no way to make one column bigger than other columns, or to change the background or text color of a single column. You have two opportunities to change the way that columns display:

- Changing the size of the gap between columns using the [`column-gap`](https://developer.mozilla.org/en-US/docs/Web/CSS/column-gap).
- Adding a rule between columns with [`column-rule`](https://developer.mozilla.org/en-US/docs/Web/CSS/column-rule).

Using your example above, change the size of the gap by adding a `column-gap` property:

```css
.container {
  column-width: 200px;
  column-gap: 20px;
}
```

You can play around with different values — the property accepts any length unit. Now add a rule between the columns, with `column-rule`. In a similar way to the [`border`](https://developer.mozilla.org/en-US/docs/Web/CSS/border) property that you encountered in previous lessons, `column-rule` is a shorthand for [`column-rule-color`](https://developer.mozilla.org/en-US/docs/Web/CSS/column-rule-color), [`column-rule-style`](https://developer.mozilla.org/en-US/docs/Web/CSS/column-rule-style), and [`column-rule-width`](https://developer.mozilla.org/en-US/docs/Web/CSS/column-rule-width), and accepts the same values as `border`.

```css
.container {
  column-count: 3;
  column-gap: 20px;
  column-rule: 4px dotted rgb(79, 185, 227);
}
```

Try adding rules of different styles and colors.

Something to take note of is that the rule does not take up any width of its own. It lies across the gap you created with `column-gap`. To make more space either side of the rule you will need to increase the `column-gap` size.

## Columns and fragmentation

The content of a multi-column layout is fragmented. It essentially behaves the same way as content behaves in paged media — such as when you print a webpage. When you turn your content into a multicol container it is fragmented into columns, and the content breaks to allow this to happen.

Sometimes, this breaking will happen in places that lead to a poor reading experience. In the live example below, I have used multicol to lay out a series of boxes, each of which have a heading and some text inside. The heading becomes separated from the text if the columns fragment between the two.

```html
<div class="container">
    <div class="card">
      <h2>I am the heading</h2>
      <p> Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla luctus aliquam dolor, eu lacinia lorem placerat
                vulputate. Duis felis orci, pulvinar id metus ut, rutrum luctus orci. Cras porttitor imperdiet nunc, at ultricies
                tellus laoreet sit amet. Sed auctor cursus massa at porta. Integer ligula ipsum, tristique sit amet orci
                vel, viverra egestas ligula.</p>
    </div>

    <div class="card">
      <h2>I am the heading</h2>
      <p> Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla luctus aliquam dolor, eu lacinia lorem placerat
                vulputate. Duis felis orci, pulvinar id metus ut, rutrum luctus orci. Cras porttitor imperdiet nunc, at ultricies
                tellus laoreet sit amet. Sed auctor cursus massa at porta. Integer ligula ipsum, tristique sit amet orci
                vel, viverra egestas ligula.</p>
    </div>

    <div class="card">
      <h2>I am the heading</h2>
      <p> Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla luctus aliquam dolor, eu lacinia lorem placerat
                vulputate. Duis felis orci, pulvinar id metus ut, rutrum luctus orci. Cras porttitor imperdiet nunc, at ultricies
                tellus laoreet sit amet. Sed auctor cursus massa at porta. Integer ligula ipsum, tristique sit amet orci
                vel, viverra egestas ligula.</p>
    </div>
    <div class="card">
      <h2>I am the heading</h2>
      <p> Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla luctus aliquam dolor, eu lacinia lorem placerat
                vulputate. Duis felis orci, pulvinar id metus ut, rutrum luctus orci. Cras porttitor imperdiet nunc, at ultricies
                tellus laoreet sit amet. Sed auctor cursus massa at porta. Integer ligula ipsum, tristique sit amet orci
                vel, viverra egestas ligula.</p>
    </div>

    <div class="card">
      <h2>I am the heading</h2>
      <p> Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla luctus aliquam dolor, eu lacinia lorem placerat
                vulputate. Duis felis orci, pulvinar id metus ut, rutrum luctus orci. Cras porttitor imperdiet nunc, at ultricies
                tellus laoreet sit amet. Sed auctor cursus massa at porta. Integer ligula ipsum, tristique sit amet orci
                vel, viverra egestas ligula.</p>
    </div>

    <div class="card">
      <h2>I am the heading</h2>
      <p> Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla luctus aliquam dolor, eu lacinia lorem placerat
                vulputate. Duis felis orci, pulvinar id metus ut, rutrum luctus orci. Cras porttitor imperdiet nunc, at ultricies
                tellus laoreet sit amet. Sed auctor cursus massa at porta. Integer ligula ipsum, tristique sit amet orci
                vel, viverra egestas ligula.</p>
    </div>

    <div class="card">
      <h2>I am the heading</h2>
      <p> Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla luctus aliquam dolor, eu lacinia lorem placerat
                vulputate. Duis felis orci, pulvinar id metus ut, rutrum luctus orci. Cras porttitor imperdiet nunc, at ultricies
                tellus laoreet sit amet. Sed auctor cursus massa at porta. Integer ligula ipsum, tristique sit amet orci
                vel, viverra egestas ligula.</p>
    </div>

</div>

```

```css
.container {
  column-width: 250px;
  column-gap: 20px;
}

.card {
  background-color: rgb(207, 232, 220);
  border: 2px solid rgb(79, 185, 227);
  padding: 10px;
  margin: 0 0 1em 0;
}
```

To control this behavior we can use properties from the [CSS Fragmentation](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Fragmentation) specification. This specification gives us properties to control breaking of content in multicol and in paged media. For example, add the property [`break-inside`](https://developer.mozilla.org/en-US/docs/Web/CSS/break-inside) with a value of `avoid` to the rules for `.card`. This is the container of the heading and text, and therefore we do not want to fragment this box.

At the present time it is also worth adding the older property `page-break-inside: avoid` for best browser support.

```css
.card {
  break-inside: avoid;
  page-break-inside: avoid;
  background-color: rgb(207,232,220);
  border: 2px solid rgb(79,185,227);
  padding: 10px;
  margin: 0 0 1em 0;
}
```

Reload the page and your boxes should stay in one piece.

```css
.container {
  column-width: 250px;
  column-gap: 20px;
}

.card {
  break-inside: avoid;
  page-break-inside: avoid;
  background-color: rgb(207, 232, 220);
  border: 2px solid rgb(79, 185, 227);
  padding: 10px;
  margin: 0 0 1em 0;
}
```

## Summary

You now know how to use the basic features of multiple-column layout, another tool at your disposal when choosing a layout method for the designs you are building.

# Responsive design

In the early days of web design, pages were built to target a particular screen size. If the user had a larger or smaller screen than the designer expected results ranged from unwanted scrollbars, to overly long line lengths, and poor use of space. As more diverse screen sizes became available, the concept of *responsive web design* (RWD) appeared, a set of practices that allows web pages to alter their layout and appearance to suit different screen widths, resolutions, etc. It is an idea that changed the way we design for a multi-device web, and in this article we'll help you understand the main techniques you need to know to master it.

## Historic website layouts

At one point in history you had two options when designing a website:

- You could create a *liquid* site, which would stretch to fill the browser window
- or a *fixed width* site, which would be a fixed size in pixels.

These two approaches tended to result in a website that looked its best on the screen of the person designing the site! The liquid site resulted in a squashed design on smaller screens (as seen below) and unreadably long line lengths on larger ones.

![A layout with two columns squashed into a mobile size viewport.](https://mdn.mozillademos.org/files/16834/mdn-rwd-liquid.png)

**Note**: See this simple liquid layout: [example](https://mdn.github.io/css-examples/learn/rwd/liquid-width.html), [source code](https://github.com/mdn/css-examples/blob/master/learn/rwd/liquid-width.html). When viewing the example, drag your browser window in and out to see how this looks at different sizes.

The fixed width site risked a horizontal scrollbar on screens smaller than the site width (as seen below), and lots of white space at the edges of the design on larger screens.

![A layout with a horizontal scrollbar in a mobile viewport.](https://mdn.mozillademos.org/files/16835/mdn-rwd-fixed.png)

**Note**: See this simple fixed width layout: [example](https://mdn.github.io/css-examples/learn/rwd/fixed-width.html), [source code](https://github.com/mdn/css-examples/blob/master/learn/rwd/fixed-width.html). Again, observe the result as you change the browser window size.

**Note**: The screenshots above are taken using the [Responsive Design Mode](https://developer.mozilla.org/en-US/docs/Tools/Responsive_Design_Mode) in Firefox DevTools.

As the mobile web started to become a reality with the first feature phones, companies who wished to embrace mobile would generally create a special mobile version of their site, with a different URL (often something like *m.example.com*, or *example.mobi*). This meant that two separate versions of the site had to be developed and kept up-to-date.

In addition, these mobile sites often offered a very cut down experience. As mobile devices became more powerful and able to display full websites, this was frustrating to mobile users who found themselves trapped in the site's mobile version and unable to access information they knew was on the full-featured desktop version of the site.

## Flexible layout before responsive design

A number of approaches were developed to try to solve the downsides of the liquid or fixed width methods of building websites. In 2004 Cameron Adams wrote a post entitled [Resolution dependent layout](http://www.themaninblue.com/writing/perspective/2004/09/21/), describing a method of creating a design that could adapt to different screen resolutions. This approach required JavaScript to detect the screen resolution and load the correct CSS.

Zoe Mickley Gillenwater was instrumental in [her work](http://zomigi.com/blog/voices-that-matter-slides-available/) to describe and formalize the different ways in which flexible sites could be created, attempting to find a happy medium between filling the screen or being completely fixed in size.

## Responsive design

The term responsive design was [coined by Ethan Marcotte in 2010](https://alistapart.com/article/responsive-web-design/), and described the use of three techniques in combination.

1. The first was the idea of fluid grids, something which was already being explored by Gillenwater, and can be read up on in Marcotte's article, [Fluid Grids](https://alistapart.com/article/fluidgrids/) (published in 2009 on A List Apart).
2. The second technique was the idea of [fluid images](http://unstoppablerobotninja.com/entry/fluid-images). Using a very simple technique of setting the `max-width` property to `100%`, images would scale down smaller if their containing column became narrower than the image's intrinsic size, but never grow larger. This enables an image to scale down to fit in a flexibly-sized column, rather than overflow it, but not grow larger and become pixellated if the column becomes wider than the image.
3. The third key component was the [media query](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries). Media Queries enable the type of layout switch that Cameron Adams had previously explored using JavaScript, using only CSS. Rather than having one layout for all screen sizes, the layout could be changed. Sidebars could be repositioned for the smaller screen, or alternate navigation could be displayed.

It is important to understand that **responsive web design isn't a separate technology** — it is a term used to describe an approach to web design, or a set of best practices, used to create a layout that can *respond* to the device being used to view the content. In Marcotte's original exploration this meant flexible grids (using floats) and media queries, however in the almost 10 years since that article was written, working responsively has become the default. Modern CSS layout methods are inherently responsive, and we have new things built into the web platform to make designing responsive sites easier.

The rest of this article will point you to the various web platform features you might want to use when creating a responsive site.

## Media Queries

Responsive design was only able to emerge due to the media query. The Media Queries Level 3 specification became a Candidate Recommendation in 2009, meaning that it was deemed ready for implementation in browsers. Media Queries allow us to run a series of tests (e.g. whether the user's screen is greater than a certain width, or a certain resolution) and apply CSS selectively to style the page appropriately for the user's needs.

For example, the following media query tests to see if the current web page is being displayed as screen media (therefore not a printed document) and the viewport is at least 800 pixels wide. The CSS for the `.container` selector will only be applied if these two things are true.

```css
@media screen and (min-width: 800px) { 
  .container { 
    margin: 1em 2em; 
  } 
} 
```

You can add multiple media queries within a stylesheet, tweaking your whole layout or parts of it to best suit the various screen sizes. The points at which a media query is introduced, and the layout changed, are known as *breakpoints*.

A common approach when using Media Queries is to create a simple single-column layout for narrow-screen devices (e.g mobile phones), then check for larger screens and implement a multiple column layout when you know that you have enough screen width to handle it. This is often described as **mobile first** design.

Find out more in the MDN documentation for [Media Queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries).

## Flexible grids

Responsive sites don't just change their layout between breakpoints, they are built on flexible grids. A flexible grid means that you don't need to target every possible device size that there is, and build a pixel perfect layout for it. That approach would be impossible given the vast number of differently-sized devices that exist, and the fact that on desktop at least, people do not always have their browser window maximized.

By using a flexible grid, you only need to add in a breakpoint and change the design at the point where the content starts to look bad. For example if the line lengths become unreadably long as the screen size increases, or a box becomes squashed with two words on each line as it narrows.

In the early days of responsive design our only option for performing layout was to use [floats](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Floats). Flexible floated layouts were achieved by giving each element a percentage width, and ensuring that across the layout the totals were not more than 100%. In his original piece on fluid grids, Marcotte detailed a formula for taking a layout designed using pixels and converting it into percentages.

```
target / context = result 
```

For example if our target column size is 60 pixels, and the context (or container) it is in is 960 pixels, we divide 60 by 960 to get a value we can use in our CSS, after moving the decimal point two places to the right.

```css
.col { 
  width: 6.25%; /* 60 / 960 = 0.0625 */ 
} 
```

This approach will be found in many places across the web today, and it is documented here in the layout section of our [Legacy layout methods](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Legacy_Layout_Methods) article. It is likely that you will come across websites using this approach in your work, so it is worth understanding it, even though you would not build a modern site using a float-based flexible grid.

The following example demonstrates a simple responsive design using Media Queries and a flexible grid. On narrow screens the layout displays the boxes stacked on top of one another:

![A mobile view of the layout with boxes stacked on top of each other vertically.](https://mdn.mozillademos.org/files/16836/mdn-rwd-mobile.png)

On wider screens they move to two columns:

![A desktop view of a layout with two columns.](https://mdn.mozillademos.org/files/16837/mdn-rwd-desktop.png)

**Note**: You can find the [live example](https://mdn.github.io/css-examples/learn/rwd/float-based-rwd.html) and [source code](https://github.com/mdn/css-examples/blob/master/learn/rwd/float-based-rwd.html) for this example on GitHub.

## Modern layout technologies

Modern layout methods such as [Multiple-column layout](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Multiple-column_Layout), [Flexbox](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox), and [Grid](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Grids) are responsive by default. They all assume that you are trying to create a flexible grid and give you easier ways to do so.

### Multicol



The oldest of these layout methods is multicol — when you specify a `column-count`, this indicates how many columns you want your content to be split into. The browser then works out the size of these, a size that will change according to the screen size.

```css
.container { 
  column-count: 3; 
} 
```

If you instead specify a `column-width`, you are specifying a *minimum* width. The browser will create as many columns of that width as will comfortably fit into the container, then share out the remaining space between all the columns. Therefore the number of columns will change according to how much space there is.

```css
.container { 
  column-width: 10em; 
} 
```

### Flexbox



In Flexbox, flex items will shrink and distribute space between the items according to the space in their container, as their initial behavior. By changing the values for `flex-grow` and `flex-shrink` you can indicate how you want the items to behave when they encounter more or less space around them.

In the example below the flex items will each take an equal amount of space in the flex container, using the shorthand of `flex: 1` as described in the layout topic [Flexbox: Flexible sizing of flex items](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Flexbox#Flexible_sizing_of_flex_items).

```css
.container { 
  display: flex; 
} 

.item { 
  flex: 1; 
} 
```

**Note**: As an example we have rebuilt the simple responsive layout above, this time using flexbox. You can see how we no longer need to use strange percentage values to calculate the size of the columns: [example](https://mdn.github.io/css-examples/learn/rwd/flex-based-rwd.html), [source code](https://github.com/mdn/css-examples/blob/master/learn/rwd/flex-based-rwd.html).

### CSS grid



In CSS Grid Layout the `fr` unit allows the distribution of available space across grid tracks. The next example creates a grid container with three tracks sized at `1fr`. This will create three column tracks, each taking one part of the available space in the container. You can find out more about this approach to create a grid in the Learn Layout Grids topic, under [Flexible grids with the fr unit](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout/Grids#Flexible_grids_with_the_fr_unit).

```css
.container { 
  display: grid; 
  grid-template-columns: 1fr 1fr 1fr; 
} 
```

**Note**: The grid layout version is even simpler as we can define the columns on the .wrapper: [example](https://mdn.github.io/css-examples/learn/rwd/grid-based-rwd.html), [source code](https://github.com/mdn/css-examples/blob/master/learn/rwd/grid-based-rwd.html).

## Responsive images

The simplest approach to responsive images was as described in Marcotte's early articles on responsive design. Basically, you would take an image that was at the largest size that might be needed, and scale it down. This is still an approach used today, and in most stylesheets you will find the following CSS somewhere:

```css
img {
  max-width: 100%:
} 
```

There are obvious downsides to this approach. The image might be displayed a lot smaller than its intrinsic size, which is a waste of bandwidth — a mobile user may be downloading an image several times the size of what they actually see in the browser window. In addition, you may not want the same image aspect ratio on mobile as on desktop. For example, it might be nice to have a square image for mobile, but show the same scene as a landscape image on desktop. Or, acknowledging the smaller size of an image on mobile you might want to show a different image altogether, one which is more easily understood at a small screen size. These things can't be achieved by simply scaling down an image.

Responsive Images, using the `<picture>` element and the `<img>` `srcset` and `sizes` attributes solve both of these problems. You can provide multiple sizes along with "hints" (meta data that describes the screen size and resolution the image is best suited for), and the browser will choose the most appropriate image for each device, ensuring that a user will download an image size appropriate for the device they are using.

You can also *art direct* images used at different sizes, thus providing a different crop or completely different image to different screen sizes.

You can find a detailed [guide to Responsive Images in the Learn HTML section](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images) here on MDN.

## Responsive typography

An element of responsive design not covered in earlier work was the idea of responsive typography. Essentially, this describes changing font sizes within media queries to reflect lesser or greater amounts of screen real estate.

In this example, we want to set our level 1 heading to be `4rem`, meaning it will be four times our base font size. That's a really large heading! We only want this jumbo heading on larger screen sizes, therefore we first create a smaller heading then use media queries to overwrite it with the larger size if we know that the user has a screen size of at least `1200px`.

```css
html { 
  font-size: 1em; 
} 

h1 { 
  font-size: 2rem; 
} 

@media (min-width: 1200px) { 
  h1 {
    font-size: 4rem; 
  }
} 
```

We have edited our responsive grid example above to also include responsive type using the method outlined. You can see how the heading switches sizes as the layout goes to the two column version.

On mobile the heading is smaller:

![A stacked layout with a small heading size.](https://mdn.mozillademos.org/files/16838/mdn-rwd-font-mobile.png)

On desktop however we see the larger heading size:

![A two column layout with a large heading.](https://mdn.mozillademos.org/files/16839/mdn-rwd-font-desktop.png)

**Note**: See this example in action: [example](https://mdn.github.io/css-examples/learn/rwd/type-rwd.html), [source code](https://github.com/mdn/css-examples/blob/master/learn/rwd/type-rwd.html).

As this approach to typography shows, you do not need to restrict media queries to only changing the layout of the page. They can be used to tweak any element to make it more usable or attractive at alternate screen sizes.

### Using viewport units for responsive typography



An interesting approach is to use the viewport unit `vw` to enable responsive typography. `1vw` is equal to one percent of the viewport width, meaning that if you set your font size using `vw`, it will always relate to the size of the viewport.

```css
h1 {
  font-size: 6vw;
}
```

The problem with doing the above is that the user loses the ability to zoom any text set using the `vw` unit, as that text is always related to the size of the viewport. **Therefore you should never set text using viewport units alone**.

There is a solution, and it involves using `calc()`. If you add the `vw` unit to a value set using a fixed size such as `em`s or `rem`s then the text will still be zoomable. Essentially, the `vw` unit adds on top of that zoomed value:

```css
h1 {
  font-size: calc(1.5rem + 3vw);
}
```

This means that we only need to specify the font size for the heading once, rather than set it up for mobile and redefine it in the media queries. The font then gradually increases as you increase the size of the viewport.

See an example of this in action: [example](https://mdn.github.io/css-examples/learn/rwd/type-vw.html), [source code](https://github.com/mdn/css-examples/blob/master/learn/rwd/type-vw.html).

## The viewport meta tag

If you look at the HTML source of a responsive page, you will usually see the following `<meta>` tag in the `<head>` of the document.

```html
<meta name="viewport" content="width=device-width,initial-scale=1">
```

This meta tag tells mobile browsers that they should set the width of the viewport to the device width, and scale the document to 100% of its intended size, which shows the document at the mobile-optimized size that you intended.

Why is this needed? Because mobile browsers tend to lie about their viewport width.

This meta tag exists because when the original iPhone launched and people started to view websites on a small phone screen, most sites were not mobile optimized. The mobile browser would therefore set the viewport width to 960 pixels, render the page at that width, and show the result as a zoomed-out version of the desktop layout. Other mobile browsers (e.g. on Google Android) did the same thing. Users could zoom in and pan around the website to view the bits they were interested in, but it looked bad. You will still see this today if you have the misfortune to come across a site that does not have a responsive design.

The trouble is that your responsive design with breakpoints and media queries won't work as intended on mobile browsers. If you've got a narrow screen layout that kicks in at 480px viewport width or less, and the viewport is set at 960px, you'll never see your narrow screen layout on mobile. By setting `width=device-width` you are overriding Apple's default `width=960px` with the actual width of the device, so your media queries will work as intended.

So you should *always* include the above line of HTML in the head of your documents.

There are other settings you can use with the viewport meta tag, however in general the above line is what you will want to use.

- `initial-scale`: Sets the initial zoom of the page, which we set to 1.
- `height`: Sets a specific height for the viewport.
- `minimum-scale`: Sets the minimum zoom level.
- `maximum-scale`: Sets the maximum zoom level.
- `user-scalable`: Prevents zooming if set to `no`.

You should avoid using `minimum-scale`, `maximum-scale`, and in particular setting `user-scalable` to `no`. Users should be allowed to zoom as much or as little as they need to; preventing this causes accessibility problems.

**Note**: There is a CSS @ rule designed to replace the viewport meta tag — [@viewport](https://developer.mozilla.org/en-US/docs/Web/CSS/@viewport) — however it has poor browser support. It was implemented in Internet Explorer and Edge, however once the Chromium-based Edge ships it will no longer be part of the Edge browser.

## Summary

Responsive design refers to a site or application design that responds to the environment in which it is viewed. It encompasses a number of CSS and HTML features and techniques, and is now essentially just how we build websites by default. Consider the sites that you visit on your phone — it is probably fairly unusual to come across a site that is the desktop version scaled down, or where you need to scroll sideways to find things. This is because the web has moved to this approach of designing responsively.

It has also become much easier to achieve responsive designs with the help of the layout methods you have learned in these lessons. If you are new to web development today you have many more tools at your disposal than in the early days of responsive design. It is therefore worth checking the age of any materials you are referencing. While the historical articles are still useful, modern use of CSS and HTML makes it far easier to create elegant and useful designs, no matter what device your visitor views the site with.