# CSS building blocks

This module carries on where [CSS first steps](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps) left off — now you've gained familiarity with the language and its syntax, and got some basic experience with using it, its time to dive a bit deeper. This module looks at the cascade and inheritance, all the selector types we have available, units, sizing, styling backgrounds and borders, debugging, and lots more.

The aim here is to provide you with a toolkit for writing competent CSS and help you understand all the essential theory, before moving on to more specific disciplines like [text styling](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text) and [CSS layout](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout).

## Prerequisites

Before starting this module, you should have:

1. Basic familiarity with using computers, and using the Web passively (i.e. just looking at it, consuming the content.)
2. A basic work environment set up as detailed in [Installing basic software](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/Installing_basic_software), and an understanding of how to create and manage files, as detailed in [Dealing with files](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/Dealing_with_files).
3. Basic familiarity with HTML, as discussed in the [Introduction to HTML](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML) module.
4. An understanding of the basics of CSS, as discussed in the [CSS first steps](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps) module.

**Note**: If you are working on a computer/tablet/other device where you don't have the ability to create your own files, you could try out (most of) the code examples in an online coding program such as [JSBin](http://jsbin.com/) or [Thimble](https://thimble.mozilla.org/).

## Guides

This module contains the following articles, which cover the most essential parts of the CSS language. Along the way you'll come across plenty of exercises to allow you to test your understanding.

- [Cascade and inheritance](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Cascade_and_inheritance)

  The aim of this lesson is to develop your understanding of some of the most fundamental concepts of CSS — the cascade, specificity, and inheritance — which control how CSS is applied to HTML and how conflicts are resolved.

- [CSS selectors](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors)

  There are a wide variety of CSS selectors available, allowing for fine-grained precision when selecting elements to style. In this article and its sub-articles we'll run through the different types in great detail, seeing how they work. The sub-articles are as follows:

  [Type, class, and ID selectors](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors/Type_Class_and_ID_Selectors)

  [Attribute selectors](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors/Attribute_selectors)

  [Pseudo-classes and pseudo-elements](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors/Pseudo-classes_and_pseudo-elements)

  [Combinators](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors/Combinators)

- [The box model](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/The_box_model)

  Everything in CSS has a box around it, and understanding these boxes is key to being able to create layouts with CSS, or to align items with other items. In this lesson we will take a proper look at the CSS *Box Model*, in order that you can move onto more complex layout tasks with an understanding of how it works and the terminology that relates to it.

- [Backgrounds and borders](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Backgrounds_and_borders)

  In this lesson we will take a look at some of the creative things you can do with CSS backgrounds and borders. From adding gradients, background images, and rounded corners, backgrounds and borders are the answer to a lot of styling questions in CSS.

- [Handling different text directions](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Handling_different_text_directions)

  In recent years, CSS has evolved in order to better support different directionality of content, including right-to-left but also top-to-bottom content (such as Japanese) — these different directionalities are called **writing modes**. As you progress in your study and begin to work with layout, an understanding of writing modes will be very helpful to you, therefore we will introduce them in this article.

- [Overflowing content](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content)

  In this lesson we will look at another important concept in CSS — **overflow**. Overflow is what happens when there is too much content to be contained comfortably inside a box. In this guide you will learn what it is and how to manage it.

- [CSS values and units](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units)

  Every property used in CSS has a value or set of values that are allowed for that property. In this lesson we will take a look at some of the most common values and units in use.

- [Sizing items in CSS](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Sizing_items_in_CSS)

  In the various lessons so far you have come across a number of ways to size items on a web page using CSS. Understanding how big the different features in your design will be is important, and in this lesson we will summarize the various ways elements get a size via CSS and define a few terms around sizing that will help you in the future.

- [Images, media, and form elements](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Images_media_form_elements)

  In this lesson we will take a look at how certain special elements are treated in CSS. Images, other media, and form elements behave a little differently in terms of your ability to style them with CSS than regular boxes. Understanding what is and isn't possible can save some frustration, and this lesson will highlight some of the main things that you need to know.

- [Styling tables](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Styling_tables)

  Styling an HTML table isn't the most glamorous job in the world, but sometimes we all have to do it. This article provides a guide to making HTML tables look good, with some specific table styling techniques highlighted.

- [Debugging CSS](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Debugging_CSS)

  Sometimes when writing CSS you will encounter an issue where your CSS doesn't seem to be doing what you expect. This article will give you guidance on how to go about debugging a CSS problem, and show you how the DevTools included in all modern browsers can help you find out what is going on.

- [Organizing your CSS](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Organizing)

  As you start to work on larger stylesheets and big projects you will discover that maintaining a huge CSS file can be challenging. In this article we will take a brief look at some best practices for writing your CSS to make it easily maintainable, and some of the solutions you will find in use by others to help improve maintainability.

## Assessments

Want to test your CSS skills? The following assessments will test your understanding of the CSS covered in the guides above.

- [Fundamental CSS comprehension](https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS/Fundamental_CSS_comprehension)

  This assessment tests your understanding of basic syntax, selectors, specificity, box model, and more.

- [Creating fancy letterheaded paper](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_boxes/Creating_fancy_letterheaded_paper)

  If you want to make the right impression, writing a letter on nice letterheaded paper can be a really good start. In this assessment, we'll challenge you to create an online template to achieve such a look.

- [A cool looking box](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_boxes/A_cool_looking_box)

  Here you'll get some practice in using background and border styling to create an eye-catching box.

## See also

- [Advanced styling effects](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Advanced_styling_effects)

  This article acts as a box of tricks, providing an introduction to some interesting advanced styling features such as box shadows, blend modes, and filters.

# Cascade and inheritance

The aim of this lesson is to develop your understanding of some of the most fundamental concepts of CSS — the cascade, specificity, and inheritance — which control how CSS is applied to HTML and how conflicts are resolved.

While working through this lesson may seem less immediately relevant and a little more academic than some other parts of the course, an understanding of these things will save you much pain later on! We encourage you to work through this section carefully, and check that you understand the concepts before moving on.

## Conflicting rules

CSS stands for Cascading Style Sheets, and that first word *cascading* is incredibly important to understand — the way that the cascade behaves is key to understanding CSS.

At some point, you will be working on a project and you will find that the CSS you thought should be applied to an element is not working. Usually the problem is that you have created two rules which could potentially apply to the same element. The **cascade**, and the closely-related concept of **specificity**, are mechanisms that control which rule applies when there is such a conflict. Which rule is styling your element may not be the one you expect, so you need to understand how these mechanisms work.

Also significant here is the concept of **inheritance**, which means that some CSS properties by default inherit values set on the current element's parent element, and some don't. This can also cause some behavior that you might not expect.

Let's start by taking a quick look at the key things we are dealing with, then we'll look at each in turn and see how they interact with each other and your CSS. This can seem like a set of tricky concepts to understand, however, as you practice writing CSS the way that it works will become more obvious to you.

### The cascade



Stylesheets **cascade** — at a very simple level this means that the order of CSS rules matter; when two rules apply that have equal specificity the one that comes last in the CSS is the one that will be used.

In the below example, we have two rules that could apply to the `h1`. The `h1` ends up being colored blue — these rules have an identical selector and therefore carry the same specificity, so the last one in the source order wins.



###  Specificity



Specificity is how the browser decides which rule applies if multiple rules have different selectors, but could still apply to the same element. It is basically a measure of how specific a selector's selection will be:

- An element selector is less specific — it will select all elements of that type that appear on a page — so will get a lower score.
- A class selector is more specific — it will select only the elements on a page that have a specific `class` attribute value — so will get a higher score.

Example time! Below we again have two rules that could apply to the `h1`. The below `h1` ends up being colored red — the class selector gives its rule a higher specificity, and so it will be applied even though the rule with the element selector appears further down in the source order.

 

We'll explain specificity scoring and other such things later on.

### Inheritance



Inheritance also needs to be understood in this context — some CSS property values set on parent elements are inherited by their child elements, and some aren't.

For example, if you set a `color` and `font-family` on an element, every element inside it will also be styled with that color and font, unless you've applied different color and font values directly to them.

 

Some properties do not inherit — for example if you set a `width` of 50% on an element, all of its descendants do not get a width of 50% of their parent's width. If this was the case, CSS would be very frustrating to use!

**Note**: On MDN CSS property reference pages you can find a technical information box, usually at the bottom of the specifications section, which lists a number of data points about that property, including whether it is inherited or not. See the [color property Specifications section](https://developer.mozilla.org/en-US/docs/Web/CSS/color#Specifications), for example.

## Understanding how the concepts work together

These three concepts together control which CSS applies to what element; in the below sections we'll see how they work together. It can sometimes seem a little bit complicated, but you will start to remember them as you get more experienced with CSS, and you can always look up the details if you forget! Even experienced developers don't remember all the details.

## Understanding inheritance

We'll start with inheritance. In the example below we have a `ul`, with two levels of unordered lists nested inside it. We have given the outer `ul` a border, padding, and a font color.

The color has applied to the direct children, but also the indirect children — the immediate child `li`s, and those inside the first nested list. We have then added a class of `special` to the second nested list and applied a different color to it. This then inherits down through its children.

Things like widths (as mentioned above), margins, padding, and borders do not inherit. If a border were to be inherited by the children of our list, every single list and list item would gain a border — probably not an effect we would ever want!

Which properties are inherited by default and which aren't is largely down to common sense.

### Controlling inheritance



CSS provides four special universal property values for controlling inheritance. Every CSS property accepts these values.

- [`inherit`](https://developer.mozilla.org/en-US/docs/Web/CSS/inherit)

  Sets the property value applied to a selected element to be the same as that of its parent element. Effectively, this "turns on inheritance".

- [`initial`](https://developer.mozilla.org/en-US/docs/Web/CSS/initial)

  Sets the property value applied to a selected element to be the same as the value set for that element in the browser's default style sheet. If no value is set by the browser's default style sheet and the property is naturally inherited, then the property value is set to `inherit` instead.

- [`unset`](https://developer.mozilla.org/en-US/docs/Web/CSS/unset)

  Resets the property to its natural value, which means that if the property is naturally inherited it acts like `inherit`, otherwise it acts like `initial`.

**Note**: There is also a newer value, [`revert`](https://developer.mozilla.org/en-US/docs/Web/CSS/revert), which has limited browser support.

**Note**: See [Origin of CSS declarations](https://developer.mozilla.org/en-US/docs/Web/CSS/Cascade#Origin_of_CSS_declarations) in [Introducing the CSS Cascade](https://developer.mozilla.org/en-US/docs/Web/CSS/Cascade) for more information on each of these and how they work.

We can look at a list of links and explore how the universal values work. The live example below allows you to play with the CSS and see what happens when you make changes. Playing with code really is the best way to get to grips with HTML and CSS.

For example:

1. The second list item has the class `my-class-1` applied. This sets the color of the a element nested inside to inherit. If you remove the rule how does it change the color of the link?
2. Do you understand why the third and fourth links are the color that they are? Check the description of the values above if not.
3. Which of the links will change color if you define a new color for the `a` element — for example `a { color: red; }`?

### Resetting all property values



The CSS shorthand property `all` can be used to apply one of these inheritance values to (almost) all properties at once. Its value can be any one of the inheritance values (`inherit`, `initial`, `unset`, or `revert`). It's a convenient way to undo changes made to styles so that you can get back to a known starting point before beginning new changes.

In the below example we have two blockquotes. The first has styling applied to the blockquote element itself, the second has a class applied to the blockquote which sets the value of `all` to `unset`.

 

Try setting the value of `all` to some of the other available values and observe what the difference is.

## Understanding the cascade

We now understand why a paragraph nested deep in the structure of your HTML is the same color as the CSS applied to the body, and from the introductory lessons we have an understanding of how to change the CSS applied to something at any point in the document — whether by assigning CSS to an element or creating a class. We will now take a proper look at how the cascade defines which CSS rules apply when more than one thing could style an element.

There are three factors to consider, listed here in decreasing order of importance. Earlier ones overrule later ones:

1. Importance
2. Specificity
3. Source order

We will look at these from the bottom up, to see how browsers figure out exactly what CSS should be applied.

### Source order



We have already seen how source order matters to the cascade. If you have more than one rule, which has exactly the same weight, then the one that comes last in the CSS will win. You can think of this as rules which are nearer the element itself overwriting early ones until the last one wins and gets to style the element.

### Specificity



Once you understand the fact that source order matters, at some point you will run into a situation where you know that a rule comes later in the stylesheet, but an earlier, conflicting, rule is applied. This is because that earlier rule has a **higher specificity** — it is more specific, and therefore is being chosen by the browser as the one that should style the element.

As we saw earlier in this lesson, a class selector has more weight than an element selector, so the properties defined on the class will override those applied directly to the element.

Something to note here is that although we are thinking about selectors, and the rules that are applied to the thing they select, it isn't the entire rule which is overwritten, only the properties which are the same.

This behavior helps avoid repetition in your CSS. A common practice is to define generic styles for the basic elements, and then create classes for those which are different. For example, in the stylesheet below we have defined generic styles for level 2 headings, and then created some classes which change only some of the properties and values. The values defined initially are applied to all headings, then the more specific values are applied to the headings with the classes.

Let's now have a look at how the browser will calculate specificity. We already know that an element selector has low specificity and can be overwritten by a class. Essentially a value in points is awarded to different types of selectors, and adding these up gives you the weight of that particular selector, which can then be assessed against other potential matches.

The amount of specificity a selector has is measured using four different values (or components), which can be thought of as thousands, hundreds, tens and ones — four single digits in four columns:

1. **Thousands**: Score one in this column if the declaration is inside a `style` attribute, aka inline styles. Such declarations don't have selectors, so their specificity is always simply 1000.
2. **Hundreds**: Score one in this column for each ID selector contained inside the overall selector.
3. **Tens**: Score one in this column for each class selector, attribute selector, or pseudo-class contained inside the overall selector.
4. **Ones**: Score one in this column for each element selector or pseudo-element contained inside the overall selector.

**Note**: The universal selector (`*`), combinators (`+`, `>`, `~`, ' '), and negation pseudo-class (`:not`) have no effect on specificity.

The following table shows a few isolated examples to get you in the mood. Try going through these, and making sure you understand why they have the specificity that we have given them. We've not covered selectors in detail yet, but you can find details of each selector on the MDN [selectors reference](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors).

| Selector                                                     | k    | h    | d    | p    | Total specificity |
| :----------------------------------------------------------- | :--- | :--- | :--- | :--- | :---------------- |
| `h1`                                                         | 0    | 0    | 0    | 1    | 0001              |
| `h1 + p::first-letter`                                       | 0    | 0    | 0    | 3    | 0003              |
| `li > a[href*="en-US"] > .inline-warning`                    | 0    | 0    | 2    | 2    | 0022              |
| `#identifier`                                                | 0    | 1    | 0    | 0    | 0100              |
| No selector, with a rule inside an element's `style` attribute | 1    | 0    | 0    | 0    | 1000              |

 Before we move on, let's look at an example in action. 

```css
a {
    display: inline-block;
    line-height: 40px;
    font-size: 20px;
    text-decoration: none;
    text-align: center;
    width: 200px;
    margin-bottom: 10px;
}

ul {
    padding: 0;
}

li {
    list-style-type: none;
}      
```



So what's going on here? First of all, we are only interested in the first seven rules of this example, and as you'll notice, we have included their specificity values in a comment before each one.

- The first two selectors are competing over the styling of the link's background color — the second one wins and makes the background color blue because it has an extra ID selector in the chain: its specificity is 201 vs. 101.

```css

/* specificity: 0101 */
#outer a {
    background-color: red;
}
        
/* specificity: 0201 */
#outer #inner a {
    background-color: blue;
}
```



- The third and fourth selectors are competing over the styling of the link's text color — the second one wins and makes the text white because although it has one less element selector, the missing selector is swapped out for a class selector, which is worth ten rather than one. So the winning specificity is 113 vs. 104.

```css
/* specificity: 0104 */
#outer div ul li a {
    color: yellow;
}

/* specificity: 0113 */
#outer div ul .nav a {
    color: white;
}
```



- Selectors 5–7 are competing over the styling of the link's border when hovered. Selector six clearly loses to five with a specificity of 23 vs. 24 — it has one fewer element selectors in the chain. Selector seven, however, beats both five and six — it has the same number of sub-selectors in the chain as five, but an element has been swapped out for a class selector. So the winning specificity is 33 vs. 23 and 24.

```css
/* specificity: 0024 */
div div li:nth-child(2) a:hover {
    border: 10px solid black;
}

/* specificity: 0023 */
div li:nth-child(2) a:hover {
    border: 10px dashed black;
}

/* specificity: 0033 */
div div .nav:nth-child(2) a:hover {
    border: 10px double black;
}
```



**Note**: This has only been an approximate example for ease of understanding. In actuality, each selector type has its own level of specificity that cannot be overwritten by selectors with a lower specificity level. For example, a *million* **class** selectors combined would not be able to overwrite the rules of *one* **id** selector.

A more accurate way to evaluate specificity would be to score the specificity levels individually starting from highest and moving on to lowest when necessary. Only when there is a tie between selector scores within a specificity level do you need to evaluate the next level down; otherwise, you can disregard the lower specificity level selectors since they can never overwrite the higher specificity levels.

### !important



There is a special piece of CSS that you can use to overrule all of the above calculations, however you should be very careful with using it — `!important`. This is used to make a particular property and value the most specific thing, thus overriding the normal rules of the cascade.

Take a look at this example where we have two paragraphs, one of which has an ID.

```css
#winning {
    background-color: red;
    border: 1px solid black;
}
    
.better {
    background-color: gray;
    border: none !important;
}
    
p {
    background-color: blue;
    color: white;
    padding: 5px;
}
```

Let's walk through this to see what's happening — try removing some of the properties to see what happens if you are finding it hard to understand:

1. You'll see that the third rule's `color` and `padding` values have been applied, but the `background-color` hasn't. Why? Really all three should surely apply, because rules later in the source order generally override earlier rules.
2. However, The rules above it win, because class selectors have higher specificity than element selectors.
3. Both elements have a `class` of `better`, but the 2nd one has an `id` of `winning` too. Since IDs have an *even higher* specificity than classes (you can only have one element with each unique ID on a page, but many elements with the same class — ID selectors are *very specific* in what they target), the red background color and the 1 pixel black border should both be applied to the 2nd element, with the first element getting the gray background color, and no border, as specified by the class.
4. The 2nd element *does* get the red background color, but no border. Why? Because of the `!important` declaration in the second rule — including this after `border: none` means that this declaration will win over the border value in the previous rule, even though the ID has higher specificity.

**Note**: The only way to override this `!important` declaration would be to include another `!important` declaration on a declaration with the *same specificity* later in the source order, or one with a higher specificity.

It is useful to know that `!important` exists so that you know what it is when you come across it in other people's code. **However, we strongly recommend that you never use it unless you absolutely have to.** `!important` changes the way the cascade normally works, so it can make debugging CSS problems really hard to work out, especially in a large stylesheet.

One situation in which you may have to use it is when you are working on a CMS where you can't edit the core CSS modules, and you really want to override a style that can't be overridden in any other way. But really, don't use it if you can avoid it.

## The effect of CSS location

Finally, it is also useful to note that the importance of a CSS declaration depends on what stylesheet it is specified in — it is possible for users to set custom stylesheets to override the developer's styles, for example the user might be visually impaired, and want to set the font size on all web pages they visit to be double the normal size to allow for easier reading.

## To summarize

Conflicting declarations will be applied in the following order, with later ones overriding earlier ones:

1. Declarations in user agent style sheets (e.g. the browser's default styles, used when no other styling is set).
2. Normal declarations in user style sheets (custom styles set by a user).
3. Normal declarations in author style sheets (these are the styles set by us, the web developers).
4. Important declarations in author style sheets
5. Important declarations in user style sheets

It makes sense for web developers' stylesheets to override user stylesheets, so the design can be kept as intended, but sometimes users have good reasons to override web developer styles, as mentioned above — this can be achieved by using `!important` in their rules.

## Active learning: playing with the cascade

In this active learning, we'd like you to experiment with writing a single new rule that will override the color and background color that we've applied to the links by default. Can you use one of the special values we looked at in the [Controlling inheritance](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Cascade_and_inheritance#Controlling_inheritance) section to write a declaration in a new rule that will reset the background color back to white, without using an actual color value?

If you make a mistake, you can always reset it using the *Reset* button. If you get really stuck, [take a look at the solution here](https://github.com/mdn/css-examples/blob/master/learn/solutions.md#the-cascade).

## What's next

If you understood most of this article, then well done — you've started getting familiar with the fundamental mechanics of CSS. Next up, we'll look at selectors in detail.

If you didn't fully understand the cascade, specificity, and inheritance, then don't worry! This is definitely the most complicated thing we've covered so far in the course, and is something that even professional web developers sometimes find tricky. We'd advise that you return to this article a few times as you continue through the course, and keep thinking about it.

Refer back here if you start to come across strange issues with styles not applying as expected. It could be a specificity issue.

# CSS selectors

In [CSS](https://developer.mozilla.org/en-US/docs/Glossary/CSS), selectors are used to target the [HTML](https://developer.mozilla.org/en-US/docs/Glossary/HTML) elements on our web pages that we want to style. There are a wide variety of CSS selectors available, allowing for fine-grained precision when selecting elements to style. In this article and its sub-articles we'll run through the different types in great detail, seeing how they work. 

## What is a selector?

You have met selectors already. A CSS selector is the first part of a CSS Rule. It is a pattern of elements and other terms that tell the browser which HTML elements should be selected to have the CSS property values inside the rule applied to them. The element or elements which are selected by the selector are referred to as the *subject of the selector*.

![Some code with the h1 highlighted.](https://mdn.mozillademos.org/files/16550/selector.png)

In earlier articles you met some different selectors, and learned that there are selectors that target the document in different ways — for example by selecting an element such as `h1`, or a class such as `.special`.

In CSS, selectors are defined in the CSS Selectors specification; like any other part of CSS they need to have support in browsers for them to work. The majority of selectors that you will come across are defined in the [Level 3 Selectors specification](https://www.w3.org/TR/selectors-3/), which is a mature specification, therefore you will find excellent browser support for these selectors.

## Selector lists

If you have more than one thing which uses the same CSS then the individual selectors can be combined into a *selector list* so that the rule is applied to all of the individual selectors. For example, if I have the same CSS for an `h1` and also a class of `.special`, I could write this as two separate rules.

```css
h1 { 
  color: blue; 
} 

.special { 
  color: blue; 
} 
```

I could also combine these into a selector list, by adding a comma between them.

```css
h1, .special { 
  color: blue; 
} 
```

White space is valid before or after the comma. You may also find the selectors more readable if each is on a new line.

```css
h1, 
.special { 
  color: blue; 
} 
```

In the live example below try combining the two selectors which have identical declarations. The visual display should be the same after combining them.

When you group selectors in this way, if any selector is invalid the whole rule will be ignored.

In the following example, the invalid class selector rule will be ignored, whereas the `h1` would still be styled.

```css
h1 { 
  color: blue; 
} 

..special { 
  color: blue; 
} 
```

When combined however, neither the `h1` nor the class will be styled as the entire rule is deemed invalid.

```css
h1, ..special { 
  color: blue; 
} 
```

## Types of selectors

There are a few different groupings of selectors, and knowing which type of selector you might need will help you to find the right tool for the job. In this article's subarticles we will look at the different groups of selectors in more detail.

### Type, class, and ID selectors



This group includes selectors that target an HTML element such as an `h1`.

```css
h1 { }
```

It also includes selectors which target a class:

```css
.box { }
```

or, an ID:

```css
#unique { }
```

### Attribute selectors



This group of selectors gives you different ways to select elements based on the presence of a certain attribute on an element:

```css
a[title] { }
```

Or even make a selection based on the presence of an attribute with a particular value:

```css
a[href="https://example.com"] { }
```

### Pseudo-classes and pseudo-elements



This group of selectors includes pseudo-classes, which style certain states of an element. The `:hover` pseudo-class for example selects an element only when it is being hovered over by the mouse pointer:

```css
a:hover { }
```

It also includes pseudo-elements, which select a certain part of an element rather than the element itself. For example, `::first-line` always selects the first line of text inside an element (a `p` in the below case), acting as if a `span` was wrapped around the first formatted line and then selected.

```css
p::first-line { }
```

### Combinators



The final group of selectors combine other selectors in order to target elements within our documents. The following for example selects paragraphs that are direct children of `article` elements using the child combinator (`>`):

```css
article > p { }
```

## Next steps

You can take a look at the reference table of selectors below for direct links to the various types of selectors in this Learn section or on MDN in general, or continue on to start your journey by finding out about [type, class, and ID selectors](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors/Type_Class_and_ID_Selectors).

## Reference table of selectors

The below table gives you an overview of the selectors you have available to use, along with links to the pages in this guide which will show you how to use each type of selector. I have also included a link to the MDN page for each selector where you can check browser support information. You can use this as a reference to come back to when you need to look up selectors later in the material, or as you experiment with CSS generally.

| Selector                                                     | Example             | Learn CSS tutorial                                           |
| :----------------------------------------------------------- | :------------------ | :----------------------------------------------------------- |
| [Type selector](https://developer.mozilla.org/en-US/docs/Web/CSS/Type_selectors) | `h1 { }`            | [Type selectors](https://developer.mozilla.org/en-US/docs/user:chrisdavidmills/CSS_Learn/CSS_Selectors/Type_Class_and_ID_Selectors#Type_selectors) |
| [Universal selector](https://developer.mozilla.org/en-US/docs/Web/CSS/Universal_selectors) | `* { }`             | [The universal selector](https://developer.mozilla.org/en-US/docs/user:chrisdavidmills/CSS_Learn/CSS_Selectors/Type_Class_and_ID_Selectors#The_universal_selector) |
| [Class selector](https://developer.mozilla.org/en-US/docs/Web/CSS/Class_selectors) | `.box { }`          | [Class selectors](https://developer.mozilla.org/en-US/docs/user:chrisdavidmills/CSS_Learn/CSS_Selectors/Type_Class_and_ID_Selectors#Class_selectors) |
| [id selector](https://developer.mozilla.org/en-US/docs/Web/CSS/ID_selectors) | `#unique { }`       | [ID selectors](https://developer.mozilla.org/en-US/docs/user:chrisdavidmills/CSS_Learn/CSS_Selectors/Type_Class_and_ID_Selectors#ID_Selectors) |
| [Attribute selector](https://developer.mozilla.org/en-US/docs/Web/CSS/Attribute_selectors) | `a[title] { }`      | [Attribute selectors](https://developer.mozilla.org/en-US/docs/User:chrisdavidmills/CSS_Learn/CSS_Selectors/Attribute_selectors) |
| [Pseudo-class selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-classes) | `p:first-child { }` | [Pseudo-classes](https://developer.mozilla.org/en-US/docs/User:chrisdavidmills/CSS_Learn/CSS_Selectors/Pseuso-classes_and_Pseudo-elements#What_is_a_pseudo-class) |
| [Pseudo-element selectors](https://developer.mozilla.org/en-US/docs/Web/CSS/Pseudo-elements) | `p::first-line { }` | [Pseudo-elements](https://developer.mozilla.org/en-US/docs/User:chrisdavidmills/CSS_Learn/CSS_Selectors/Pseuso-classes_and_Pseudo-elements#What_is_a_pseudo-element) |
| [Descendant combinator](https://developer.mozilla.org/en-US/docs/Web/CSS/Descendant_combinator) | `article p`         | [Descendant combinator](https://developer.mozilla.org/en-US/docs/User:chrisdavidmills/CSS_Learn/CSS_Selectors/Combinators#Descendant_Selector) |
| [Child combinator](https://developer.mozilla.org/en-US/docs/Web/CSS/Child_combinator) | `article > p`       | [Child combinator](https://developer.mozilla.org/en-US/docs/User:chrisdavidmills/CSS_Learn/CSS_Selectors/Combinators#Child_combinator) |
| [Adjacent sibling combinator](https://developer.mozilla.org/en-US/docs/Web/CSS/Adjacent_sibling_combinator) | `h1 + p`            | [Adjacent sibling](https://developer.mozilla.org/en-US/docs/User:chrisdavidmills/CSS_Learn/CSS_Selectors/Combinators#Adjacent_sibling) |
| [General sibling combinator](https://developer.mozilla.org/en-US/docs/Web/CSS/General_sibling_combinator) | `h1 ~ p`            | [General sibling](https://developer.mozilla.org/en-US/docs/User:chrisdavidmills/CSS_Learn/CSS_Selectors/Combinators#General_sibling) |

# Type, class, and ID selectors

In this lesson we will take a look at the simplest selectors that are available, which you will probably use the most in your work. 

## Type selectors

A **type selector** is sometimes referred to as a *tag name selector* or *element selector*, because it selects an HTML tag/element in your document. In the below example we have used span, em and strong selectors. All instances of `span`, `em` and `strong` elements are therefore styled.

**Try adding a CSS rule to select the `h1` element and change its color to blue.**

## The universal selector

The universal selector is indicated by an asterisk (`*`) and selects everything in the document (or inside the parent element if it is being chained together with another element and a descendant combinator, for example). In the following example we have used the universal selector to remove the margins on all elements. This means that instead of the default styling added by the browser, which spaces out headings and paragraphs with margins, everything is close together and we can't see the different paragraphs easily.

This kind of behavior can sometimes be seen in "reset stylesheets", which strip out all of the browser styling. These were very popular at one point, however stripping out all styling usually meant that you then had to do the job of putting it all back! We tend to use the universal selector carefully therefore, to deal with very specific situations such as the one outlined below.

### Using the universal selector to make your selectors easier to read



One use of the universal selector is to make selectors easier to read and more obvious in terms of what they are doing. For example, if I wanted to select the first child of any `article` element, no matter what element it was, and make it bold, I could use the `:first-child` selector, which we will learn more about in the lesson on [pseudo-classes and pseudo-elements](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors/Pseudo-classes_and_pseudo-elements), as a descendant selector along with the `article` element selector: 

```css
article :first-child {

}
```

This could be confused however with `article:first-child`, which will select any `article` element that is the first child of another element.

To avoid this confusion we can add the universal selector to the `:first-child` selector, so it is obvious what the selector is doing. It is selecting *any* element which is the first-child of an `article` element:

```css
article *:first-child { 

} 
```

## Class selectors

The class selector starts with a full stop (`.`) character and will select everything in the document with that class applied to it. In the live example below we have created a class called `.highlight`, and have applied it to several places in my document. All of the elements that have the class applied are highlighted.

### Targeting classes on particular elements



You can create a selector that will target specific elements with the class applied. In this next example we will highlight a `span` with a class of `highlight` differently to an `h1` heading with a class of `highlight`. We do this by using the type selector for the element I want to target, with the class appended, with no white space in between.

```css
span.highlight {
    background-color: yellow;
}

h1.highlight {
    background-color: pink;
}
```



This approach does make the CSS less reusable as the class will now only apply to that particular element, and you would need to add another selector if you decided that the rules should apply to other elements too.

### Target an element if it has more than one class applied



You can apply multiple classes to an element and target them individually, or only select the element when all of the classes in the selector are present. This can be helpful when building up components that can be combined in different ways on your site.

In the example below we have a `div` that contains a note. The grey border is applied when the box has a class of `notebox`. If it also has a class of `warning` or `danger`, we change the [`border-color`](https://developer.mozilla.org/en-US/docs/Web/CSS/border-color).

We can tell the browser that we only want to match the element if it has all of these classes by chaining them together with no white space between them.

```css
.notebox {
    border: 4px solid #666;
    padding: .5em;
}

.notebox.warning {
    border: 4px solid orange;
    font-weight: bold;
}

.notebox.danger {
    border: 4px solid red;
    font-weight: bold;
}
```

## ID Selectors

An ID selector begins with a `#` rather than a full stop character, but is basically used in the same way as a class selector. An ID however can be used only once per document. It can select an element that has the `id` set on it, and you can precede the ID with a type selector to only target the element if both the element and ID match. You can see both of these uses in the following example:

```css
#one {
    background-color: yellow;
}

h1#heading {
    color: rebeccapurple;
}
```



**Note**: As we learned in the lesson on specificity, an ID has high specificity and will overrule most other selectors. This can make them difficult to deal with. In most cases it is preferable to add a class to the element rather than use an ID, however if using the ID is the only way to target the element — perhaps because you do not have access to the markup and so cannot edit it — this will work. 

## In the next article

We'll continue exploring selectors by looking at [attribute selectors](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors/Attribute_selectors).

# Attribute selectors

As you know from your study of HTML, elements can have attributes that give further detail about the element being marked up. In CSS you can use attribute selectors to target elements with certain attributes. This lesson will show you how to use these very useful selectors.

## Presence and value selectors

These selectors enable the selection of an element based on the presence of an attribute alone (for example `href`), or on various different matches against the value of the attribute.

| Selector            | Example                         | Description                                                  |
| :------------------ | :------------------------------ | :----------------------------------------------------------- |
| `[*attr*]`          | `a[title]`                      | Matches elements with an attribute name of *attr* — the value in square brackets. |
| `[*attr*=*value*]`  | `a[href="https://example.com"]` | Matches elements with an attribute name of *attr* whose value is exactly *value* — the string inside the quotes. |
| `[*attr*~=*value*]` | `p[class~="special"]`           | Matches elements with an attribute name of *attr* whose value is exactly *value*, or elements with a `class` attribute containing one or more class names, at least one of which matches *value*. Note that in a list of multiple class names the separate classes are whitespace-separated. |
| `[*attr*|=*value*]` | `div[lang|="zh"]`               | Matches elements with an attribute name of *attr* whose value can be exactly *value* or can begin with *value* immediately followed by a hyphen. |

In the example below you can see these selectors being used.

- By using `li[class]` we can match any selector with a class attribute. This matches all but the first list item.
- `li[class="a"]` matches a selector with a class of `a`, but not a selector with a class of `a` with another space-separated class as part of the value. It selects the second list item.
- `li[class~="a"]` will match a class of `a` but also a value that contains the class of `a` as part of a whitespace-separated list. It selects the second and third list items.

```css
li[class] {
    font-size: 200%;
}

li[class="a"] {
    background-color: yellow;
}

li[class~="a"] {
    color: red;
}
```

## Substring matching selectors

These selectors allow for more advanced matching of substrings inside the value of your attribute. For example, if you had classes of `box-warning` and `box-error` and wanted to match everything that started with the string "box-", you could use `[class^="box-"]` to select them both.

| Selector            | Example             | Description                                                  |
| :------------------ | :------------------ | :----------------------------------------------------------- |
| `[*attr*^=*value*]` | `li[class^="box-"]` | Matches elements with an attribute name of *attr* whose value has the substring *value* at the start of it. |
| `[*attr*$=*value*]` | `li[class$="-box"]` | Matches elements with an attribute name of *attr* whose value has the substring *value* at the end of it. |
| `[*attr**= ]`       | `li[class*="box"]`  | Matches elements with an attribute name of *attr* whose value contains at least one occurrence of the substring *value* anywhere within the string. |

The next example shows usage of these selectors:

- `li[class^="a"]` matches any attribute value which starts with `a`, so matches the first two list items.
- `li[class$="a"]` matches any attribute value that ends with `a`, so matches the first and third list item.
- `li[class*="a"]` matches any attribute value where `a` appears anywhere in the string, so it matches all of our list items.

```css
li[class^="a"] {
    font-size: 200%;
}

li[class$="a"] {
    background-color: yellow;
}

li[class*="a"] {
    color: red;
}
```

## Case-sensitivity

If you want to match attribute values case-insensitively you can use the value `i` before the closing bracket. This flag tells the browser to match ASCII characters case-insensitively. Without the flag the values will be matched according to the case-sensitivity of the document language — in HTML's case it will be case sensitive.

In the example below, the first selector will match a value that begins with `a` — it only matches the first list item because the other two list items start with an uppercase A. The second selector uses the case-insensitive flag and so matches all of the list items.

```css
li[class^="a"] {
    background-color: yellow;
}

li[class^="a" i] {
    color: red;
}
```

**Note**: There is also a newer value `s`, which will force case-sensitive matching in contexts where matching is normally case-insensitive, however this is less well supported in browsers and isn't very useful in an HTML context.

## Try it out

In the live example below, add CSS using attribute selectors to do the following:

- Target the `a` element with a `title` attribute and make the border pink (`border-color: pink`).
- Target the `a` element with an `href` attribute that contains the word `contact` somewhere in its value and make the border orange (`border-color: orange`).
- Target the `a` element with an `href` value starting with `https` and give it a green border (`border-color: green`).

## Next steps

Now we are done with attribute selectors, you can continue on to the next article and read about [pseudo-class and pseudo-element selectors](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors/Pseudo-classes_and_pseudo-elements).

# Pseudo-classes and pseudo-elements

The next set of selectors we will look at are referred to as **pseudo-classes** and **pseudo-elements**. There are a large number of these, and they often serve quite specific purposes. Once you know how to use them, you can look at the list to see if there is something which works for the task you are trying to achieve. Once again the relevant MDN page for each selector is helpful in explaining browser support. 

## What is a pseudo-class?

A pseudo-class is a selector that selects elements that are in a specific state, e.g. they are the first element of their type, or they are being hovered over by the mouse pointer. They tend to act as if you had applied a class to some part of your document, often helping you cut down on excess classes in your markup, and giving you more flexible, maintainable code.

Pseudo-classes are keywords that start with a colon:

```
:pseudo-class-name
```

### Simple pseudo-class example



Let's look at a simple example. If we wanted to make the first paragraph in an article larger and bold, we could add a class to that paragraph and then add CSS to that class, as shown in the first example below:

```css
.first {
    font-size: 120%;
    font-weight: bold;
}
```

However, this could be annoying to maintain — what if a new paragraph got added to the top of the document? We'd need to move the class over to the new paragraph. Instead of adding the class, we could use the [`:first-child`](https://developer.mozilla.org/en-US/docs/Web/CSS/:first-child) pseudo-class selector — this will *always* target the first child element in the article, and we will no longer need to edit the HTML (this may not always be possible anyway, maybe due to it being generated by a CMS.) 

```css
article p:first-child {
    font-size: 120%;
    font-weight: bold;
} 
```

All pseudo-classes behave in this same kind of way. They target some bit of your document that is in a certain state, behaving as if you had added a class into your HTML. Take a look at some other examples on MDN:

- `:last-child`
- `:only-child`
- `:invalid`

### User-action pseudo classes



Some pseudo-classes only apply when the user interacts with the document in some way. These **user-action** pseudo-classes, sometimes referred to as **dynamic pseudo-classes**, act as if a class had been added to the element when the user interacts with it. Examples include:

- `:hover` — mentioned above; this only applies if the user moves their pointer over an element, typically a link.
- `:focus` — only applies if the user focuses the element using keyboard controls.

```css
a:link,
a:visited {
    color: rebeccapurple;
    font-weight: bold;
}

a:hover {
    color:hotpink;
}  
```

## What is a pseudo-element?

Pseudo-elements behave in a similar way, however they act as if you had added a whole new HTML element into the markup, rather than applying a class to existing elements. Pseudo-elements start with a double colon `::`.

```
::pseudo-element-name
```

**Note**: Some early pseudo-elements used the single colon syntax, so you may sometimes see this in code or examples. Modern browsers support the early pseudo-elements with single- or double-colon syntax for backwards compatibility.

For example, if you wanted to select the first line of a paragraph you could wrap it in a `span` element and use an element selector; however, that would fail if the number of words you had wrapped were longer or shorter than the parent element's width. As we tend not to know how many words will fit on a line — as that will change if the screen width or font-size changes — it is impossible to robustly do this by adding HTML.

The `::first-line` pseudo-element selector will do this for you reliably — if the number of words increases and decreases it will still only select the first line.

```css
article p::first-line {
    font-size: 120%;
    font-weight: bold;
}
```

It acts as if a `span` was magically wrapped around that first formatted line, and updated each time the line length changed.

You can see that this selects the first line of both paragraphs.

## Combining pseudo-classes and pseudo-elements

If you wanted to make the first line of the first paragraph bold you could chain the `:first-child` and `::first-line` selectors together. Try editing the previous live example so it uses the following CSS. We are saying that we want to select the first line, of the first `p` element, which is inside an `article` element.

```css
article p:first-child::first-line { 
  font-size: 120%; 
  font-weight: bold; 
}
```

## Generating content with ::before and ::after

There are a couple of special pseudo-elements, which are used along with the `content` property to insert content into your document using CSS.

You could use these to insert a string of text, such as in the live example below. Try changing the text value of the `content` property and see it change in the output. You could also change the `::before` pseudo-element to `::after` and see the text inserted at the end of the element instead of the beginning.

```css
.box::before {
    content: "This should show before the other content."
}
```

Inserting strings of text from CSS isn't really something we do very often on the web. however, as that text is inaccessible to some screen readers and might be hard for someone to find and edit in the future.

A more valid use of these pseudo-elements is to insert an icon, for example the little arrow added in the example below, which is a visual indicator that we wouldn't want read out by a screen reader:

```css
.box::after {
    content: " ➥"
}
```

These pseudo-elements are also frequently used to insert an empty string, which can then be styled just like any element on the page.

In this next example, we have added two empty strings using the `::before` and `::after` pseudo-elements. We have set these to `display: block` in order that we can style them with a width and height. We then use CSS to style them just like any element. You can play around with the CSS and change how these look and behave.

```css
.box::before {
    content: "";
    display: block;
    width: 100px;
    height: 100px;
    background-color: rebeccapurple;
    border: 1px solid black;
}
```

The use of the `::before` and `::after` pseudo-elements along with the `content` property is referred to as "Generated Content" in CSS, and you will often see this technique being used for various tasks in CSS. A great example is the site [CSS Arrow Please](http://www.cssarrowplease.com/), which helps you to generate an arrow with CSS. Look at the CSS as you create your arrow and you will see the `::before` and `::after` pseudo-elements in use. Whenever you see these selectors, look at the `content` property to see what is being added to the document.

## Reference section

There are a large number of pseudo-classes and pseudo-elements, and it is useful to have a list to refer to. Below are tables listing them, with links to their reference pages on MDN. Use this as a reference to see the kind of things that are available for you to target.

# Combinators

The final selectors we will look at are called combinators, because they combine other selectors in a way that gives them a useful relationship to each other and the location of content in the document. 

## Descendant selector

You have already come across descendant selectors in previous lessons — selectors with spaces in between them:

```html
body article p
```

These selectors select elements that are descendants of others selector. They do not need to be direct children to match.

In the example below we are matching only the `p` element which is inside an element with a class of `.box`.

```css
.box p {
    color: red;
}
```

## Child combinator

The child combinator is a greater-than symbol (`>`), which matches only when the selectors select elements that are direct children. Descendants further down the hierarchy don't match. For example, to select only `p` elements that are direct children of `article` elements:

```html
article > p
```

In this next example we have an ordered list, nested inside of which is another unordered list. I am using the child combinator to select only the `li` elements which are a direct child of a `ul`, and have given them a top border.

If you remove the `>` that designates this as a child combinator, you end up with a descendant selector and all `li` elements will get a red border.

```css
ul > li {
    border-top: 5px solid red;
}
```

## Adjacent sibling

The adjacent sibling selector (`+`) is used to select something if it is right next to another element at the same level of the hierarchy. For example, to select all `img` elements that come right after `p` elements:

```css
p + img
```

A common use case is to do something with a paragraph that follows a heading, as in my example below. Here we are looking for a paragraph which is directly adjacent to an `h1`, and styling it.

If you insert some other element such as a `h2` in between the `h1` and the `p`, you will find that the paragraph is no longer matched by the selector and so does not get the background and foreground color applied when the element is adjacent.

```css
h1 + p {
    font-weight: bold;
    background-color: #333;
    color: #fff;
    padding: .5em;
}
```

## General sibling

If you want to select siblings of an element even if they are not directly adjacent, then you can use the general sibling combinator (`~`). To select all `img` elements that come *anywhere* after `p` elements, we'd do this:

```html
p ~ img
```

In the example below we are selecting all `p` elements that come after the `h1`, and even though there is a `div` in the document as well, the `p` that comes after it is selected.

```css
h1 ~ p {
    font-weight: bold;
    background-color: #333;
    color: #fff;
    padding: .5em;
}
```

## Using combinators

You can combine any of the selectors that we discovered in previous lessons with combinators in order to pick out part of your document. For example if we want to select list items with a class of "a", which are direct children of a `ul`, I could use the following.

```css
ul > li[class="a"]  {  }
```

Take care however when creating big lists of selectors that select very specific parts of your document. It will be hard to reuse the CSS rules as you have made the selector very specific to the location of that element in the markup.

It is often better to create a simple class and apply that to the element in question. That said, your knowledge of combinators will come in very useful if you need to get to something in your document and are unable to access the HTML, perhaps due to it being generated by a CMS.

## Moving on

This is the last section in our lessons on selectors. Next we will move on to another important part of CSS — the [CSS Box Model](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/The_box_model).