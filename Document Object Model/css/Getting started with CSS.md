# Getting started with CSS

In this article we will take a simple HTML document and apply CSS to it, learning some practical things about the language along the way. 

## Starting with some HTML

Our starting point is an HTML document. You can copy the code from below if you want to work on your own computer. Save the code below as `index.html` in a folder on your machine.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Getting started with CSS</title>
</head>
<body>
  <h1>I am a level one heading</h1>
  <p>
    This is a paragraph of text. In the text is a <span>span element</span>
    and also a <a href="http://example.com">link</a>.
  </p>
  <p>This is the second paragraph. It contains an <em>emphasized</em> element.</p>
  <ul>
    <li>Item one</li>
    <li>Item two</li>
    <li>Item <em>three</em></li>
  </ul>
</body>
</html>
```



## Adding CSS to our document

The very first thing we need to do is to tell the HTML document that we have some CSS rules we want it to use. There are three different ways to apply CSS to an HTML document that you'll commonly come across, however, for now, we will look at the most usual and useful way of doing so — linking CSS from the head of your document.

Create a file in the same folder as your HTML document and save it as `styles.css`. The `.css` extension shows that this is a CSS file.

To link `styles.css` to `index.html` add the following line somewhere inside the `<head>` of the HTML document:

```html
<link rel="stylesheet" href="styles.css">
```

This `<lik>` element tells the browser that we have a stylesheet, using the `rel` attribute, and the location of that stylesheet as the value of the `href` attribute. You can test that the CSS works by adding a rule to `styles.css`. Using your code editor add the following to your CSS file:

```css
h1 {
  color: red;
}
```

Save your HTML and CSS files and reload the page in a web browser. The level one heading at the top of the document should now be red. If that happens, congratulations — you have successfully applied some CSS to an HTML document. If that doesn't happen, carefully check that you've typed everything correctly.

You can continue to work in `styles.css` locally, or you can use our interactive editor below to continue with this tutorial. The interactive editor acts as if the CSS in the first panel is linked to the HTML document, just as we have with our document above.

## Styling HTML elements

By making our heading red we have already demonstrated that we can target and style an HTML element. We do this by targeting an *element selector* — this is a selector that directly matches an HTML element name. To target all paragraphs in the document you would use the selector `p`. To turn all paragraphs green you would use:

```css
p {
  color: green;
}
```

You can target multiple selectors at once, by separating the selectors with a comma. If I want all paragraphs and all list items to be green my rule looks like this:

```css
p, li {
    color: green;
}
```

Try this out in the interactive editor below (edit the code boxes), or in your local CSS document.



 

## Changing the default behavior of elements

When we look at a well-marked up HTML document, even something as simple as our example, we can see how the browser is making the HTML readable by adding some default styling. Headings are large and bold and our list has bullets. This happens because browsers have internal stylesheets containing default styles, which they apply to all pages by default; without them all of the text would run together in a clump and we would have to style everything from scratch. All modern browsers display HTML content by default in pretty much the same way.

However, you will often want something other than the choice the browser has made. This can be done by simply choosing the HTML element that you want to change, and using a CSS rule to change the way it looks.  A good example is our `<ul>`, an unordered list. It has list bullets, and if I decide I don't want those bullets I can remove them like so:

```css
li {
  list-style-type: none;
}
```

Try adding this to your CSS now.

The `list-style-type` property is a good property to look at on MDN to see which values are supported. Take a look at the page for `list-style-type` and you will find an interactive example at the top of the page to try some different values in, then all allowable values are detailed further down the page.

Looking at that page you will discover that in addition to removing the list bullets you can change them — try changing them to square bullets by using a value of `square`.

## Adding a class

So far we have styled elements based on their HTML element names. This works as long as you want all of the elements of that type in your document to look the same. Most of the time that isn't the case and so you will need to find a way to select a subset of the elements without changing the others. The most common way to do this is to add a class to your HTML element and target that class.

In your HTML document, add a [class attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/class) to the second list item. Your list will now look like this:

```
<ul>
  <li>Item one</li>
  <li class="special">Item two</li>
  <li>Item <em>three</em></li>
</ul>
```

In your CSS you can target the class of `special` by creating a selector that starts with a full stop character. Add the following to your CSS file:

```css
.special {
  color: orange;
  font-weight: bold;
}
```

Save and refresh to see what the result is.

You can apply the class of `special` to any element on your page that you want to have the same look as this list item. For example, you might want the `<span>` in the paragraph to also be orange and bold. Try adding a `class` of `special` to it, then reload your page and see what happens.

Sometimes you will see rules with a selector that lists the HTML element selector along with the class:

```css
li.special {
  color: orange;
  font-weight: bold;
}
```

This syntax means "target any `li` element that has a class of special". If you were to do this then you would no longer be able to apply the class to a `<span>` or another element by simply adding the class to it; you would have to add that element to the list of selectors:

```css
li.special,
span.special {
  color: orange;
  font-weight: bold;
}
```

As you can imagine, some classes might be applied to many elements and you don't want to have to keep editing your CSS every time something new needs to take on that style. Therefore it is sometimes best to bypass the element and simply refer to the class, unless you know that you want to create some special rules for one element alone, and perhaps want to make sure they are not applied to other things.

## Styling things based on their location in a document

There are times when you will want something to look different based on where it is in the document. There are a number of selectors that can help you here, but for now we will look at just a couple. In our document are two `<em>` elements — one inside a paragraph and the other inside a list item. To select only an `<em>` that is nested inside an `<li>` element I can use a selector called the **descendant combinator**, which simply takes the form of a space between two other selectors.

Add the following rule to your stylesheet.

```css
li em {
  color: rebeccapurple;
}
```

This selector will select any `<em>` element that is inside (a descendant of) an `<li>`. So in your example document, you should find that the `<em>` in the third list item is now purple, but the one inside the paragraph is unchanged.

Something else you might like to try is styling a paragraph when it comes directly after a heading at the same hierarchy level in the HTML. To do so place a `+` (an **adjacent sibling combinator**) between the selectors.

Try adding this rule to your stylesheet as well:

```css
h1 + p {
  font-size: 200%;
}
```

The live example below includes the two rules above. Try adding a rule to make a span red, if it is inside a paragraph. You will know if you have it right as the span in the first paragraph will be red, but the one in the first list item will not change color.

**Note**: As you can see, CSS gives us several ways to target elements, and we've only scratched the surface so far! We will be taking a proper look at all of these selectors and many more in our [Selectors](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors) articles later on in the course.

## Styling things based on state

The final type of styling we shall take a look at in this tutorial is the ability to style things based on their state. A straightforward example of this is when styling links. When we style a link we need to target the `<a>` (anchor) element. This has different states depending on whether it is unvisited, visited, being hovered over, focused via the keyboard, or in the process of being clicked (activated). You can use CSS to target these different states — the CSS below styles unvisited links pink and visited links green.

```css
a:link {
  color: pink;
}

a:visited {
  color: green;
}
```

You can change the way the link looks when the user hovers over it, for example removing the underline, which is achieved by in the next rule:

```css
a:hover {
  text-decoration: none;
}
```

In the live example below, you can play with different values for the various states of a link. I have added the rules above to it, and now realise that the pink color is quite light and hard to read — why not change that to a better color? Can you make the links bold?

We have removed the underline on our link on hover. You could remove the underline from all states of a link. It is worth remembering however that in a real site, you want to ensure that visitors know that a link is a link. Leaving the underline in place, can be an important clue for people to realize that some text inside a paragraph can be clicked on — this is the behavior they are used to. As with everything in CSS, there is the potential to make the document less accessible with your changes — we will aim to highlight potential pitfalls in appropriate places.

**Note**: you will often see mention of [accessibility](https://developer.mozilla.org/en-US/docs/Learn/Accessibility) in these lessons and across MDN. When we talk about accessibility we are referring to the requirement for our webpages to be understandable and usable by everyone.

Your visitor may well be on a computer with a mouse or trackpad, or a phone with a touchscreen. Or they might be using a screenreader, which reads out the content of the document, or they may need to use much larger text, or be navigating the site using the keyboard only.

A plain HTML document is generally accessible to everyone — as you start to style that document it is important that you don't make it less accessible.

## Combining selectors and combinators

It is worth noting that you can combine multiple selectors and combinators together. For example:

```css
/* selects any <span> that is inside a <p>, which is inside an <article>  */
article p span { ... }

/* selects any <p> that comes directly after a <ul>, which comes directly after an <h1>  */
h1 + ul + p { ... }
```

You can combine multiple types together, too. Try adding the following into your code:

```css
body h1 + p .special {
  color: yellow;
  background-color: black;
  padding: 5px;
}
```

This will style any element with a class of `special`, which is inside a `<p>`, which comes just after an `<h1>`, which is inside a `<body>`. Phew!

In the original HTML we provided, the only element styled is ` <span class="special"> `.

Don't worry if this seems complicated at the moment — you'll soon start to get the hang of it as you write more CSS.

## Wrapping up

In this tutorial, we have taken a look at a number of ways in which you can style a document using CSS. We will be developing this knowledge as we move through the rest of the lessons. However you now already know enough to style text, apply CSS based on different ways of targetting elements in the document, and look up properties and values in the MDN documentation.

In the next lesson we will be taking a look at how CSS is structured.

# How CSS is structured

Now that you have an idea about what CSS is and the basics of using it, it is time to look a little deeper into the structure of the language itself. We have already met many of the concepts discussed here; you can return to this one to recap if you find any later concepts confusing. 



## Applying CSS to your HTML

The first thing we will look at are the three methods of applying CSS to a document.

### External stylesheet



In the [Getting started with CSS](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps/Getting_started) we linked an external stylesheet to our page. This is the most common and useful method of attaching CSS to a document as you can link the CSS to multiple pages, allowing you to style them all with the same stylesheet. In most cases, the different pages of a site will all look pretty much the same, therefore you can use the same set of rules for the basic look and feel.

An external stylesheet is when you have your CSS written in a separate file with a `.css` extension, and you reference it from an HTML `<html>` element:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>My CSS experiment</title>
    <link rel="stylesheet" href="styles.css">
  </head>
  <body>
    <h1>Hello World!</h1>
    <p>This is my first CSS example</p>
  </body>
</html>
```

The CSS file might look something like this:

```css
h1 {
  color: blue;
  background-color: yellow;
  border: 1px solid black;
}

p {
  color: red;
}
```

The `href` attribute of the `<link>` element needs to reference a file on your filesystem.

In the example above, the CSS file is in the same folder as the HTML document, but you could place it somewhere else and adjust the specified path to suit, for example:

```html
<!-- Inside a subdirectory called styles inside the current directory -->
<link rel="stylesheet" href="styles/style.css">

<!-- Inside a subdirectory called general, which is in a subdirectory called styles, inside the current directory -->
<link rel="stylesheet" href="styles/general/style.css">

<!-- Go up one directory level, then inside a subdirectory called styles -->
<link rel="stylesheet" href="../styles/style.css">
```

### Internal stylesheet



An internal stylesheet is where you don't have an external CSS file, but instead place your CSS inside a `<style>` element contained inside the HTML `<head>`.

So the HTML would look like this:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>My CSS experiment</title>
    <style>
      h1 {
        color: blue;
        background-color: yellow;
        border: 1px solid black;
      }

      p {
        color: red;
      }
    </style>
  </head>
  <body>
    <h1>Hello World!</h1>
    <p>This is my first CSS example</p>
  </body>
</html>
```

This can be useful in some circumstances (maybe you're working with a content management system where you can't modify the CSS files directly), but it isn't quite as efficient as external stylesheets — in a website, the CSS would need to be repeated across every page, and updated in multiple places if changes were required.

### Inline styles



Inline styles are CSS declarations that affect one element only, contained within a `style` attribute:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>My CSS experiment</title>
  </head>
  <body>
    <h1 style="color: blue;background-color: yellow;border: 1px solid black;">Hello World!</h1>
    <p style="color:red;">This is my first CSS example</p>
  </body>
</html>
```

**Please don't do this, unless you really have to!** It is really bad for maintenance (you might have to update the same information multiple times per document), and it also mixes your presentational CSS information with your HTML structural information, making the code harder to read and understand. Keeping different types of code separated makes for a much easier job for all who work on the code.

There are a few places where inline styles are more common, or even advisable. You might have to resort to using them if your working environment is really restrictive (perhaps your CMS only allows you to edit the HTML body). You will also see them used a lot in HTML email in order to get compatibility with as many email clients as possible.

## Playing with the CSS in this article

There is a lot of CSS to play with in this article. To do so, we'd recommend creating a new directory/folder on your computer, and inside it creating a copy of the following two files:

index.html:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>My CSS experiments</title>
    <link rel="stylesheet" href="styles.css">
  </head>
  <body> 

    <p>Create your test HTML here</p>

  </body>
</html>
```

styles.css:

```css
/* Create your test CSS here */

p {
  color: red;
}
```

Then, when you come across some CSS you want to experiment with, replace the HTML `<body>` contents with some HTML to style, and start adding CSS to style it inside your CSS file.

If you are not using a system where you can easily create files, you can instead use the interactive editor below to experiment.

Read on, and have fun!

## Selectors

You can't talk about CSS without meeting selectors, and we have already discovered several different types in the [Getting started with CSS](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps/Getting_started) tutorial. A selector is how we target something in our HTML document in order to apply styles to it. If your styles are not applying then it is likely that your selector does not match the thing you think it should match.

Each CSS rule starts with a selector or a list of selectors in order to tell the browser which elements or elements the rules should apply to. All of the following are examples of valid selectors, or lists of selectors.

```css
h1
a:link
.manythings
#onething
*
.box p
.box p:first-child
h1, h2, .intro
```

Try creating some CSS rules that use the above selectors, and some HTML to be styled by them. If you don't know what some of the above syntax means, try search for it on MDN!

**Note**: You will learn a lot more about selectors in our [CSS selectors](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors) tutorials, in the next module.

### Specificity



There will often be scenarios where two selectors could select the same HTML element. Consider the stylesheet below where I have a rule with a `p` selector that will set paragraphs to blue, and also a class that will set selected elements red.

```css
.special {
  color: red;
}

p {
  color: blue;
}
```

Let's say that in our HTML document we have a paragraph with a class of `special`. Both rules could apply, so which one wins? What color do you think our paragraph will become?

```html
<p class="special">What color am I?</p>
```

The CSS language has rules to control which rule will win in the event of a collision — these are called **cascade** and **specificity**. In the below code block we have defined two rules for the `p` selector, but the paragraph ends up being colored blue. This is because the declaration that sets it to blue appears later in the stylesheet, and later styles override earlier ones. This is the cascade in action.

```css
p {
  color: red;
}

p {
  color: blue;
}
```

However, in the case of our earlier block with the class selector and the element selector, the class will win, making the paragraph red — even thought it appears earlier in the stylesheet. A class is described as being more specific, or having more specificity than the element selector, so it wins.

Try the above experiment for yourself — add the HTML to your experiment, then add the two `p { ... }` rules to your stylesheet. Next, change the first `p` selector to `.special` to see how it changes the styling.

The rules of specificity and the cascade can seem a little complicated at first and are easier to understand once you have built up further CSS knowledge. In our [Cascade and inheritance](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Cascade_and_inheritance) article, which you'll get to in the next module, I'll explain this in detail, including how to calculate specificity. For now, remember that this exists, and that sometimes CSS might not apply like you expect it to because something else in your stylesheet has a higher specificity. Identifying that more than one rule could apply to an element is the first step in fixing such issues.

## Properties and values

At its most basic level, CSS consists of two building blocks:

- **Properties**: Human-readable identifiers that indicate which stylistic features (e.g. `font-size`, `width`, `background-color`) you want to change.
- **Values**: Each specified property is given a value, which indicates how you want to change those stylistic features (e.g. what you want to change the font, width or background color to.)

The below image highlights a single property and value. The property name is `color`, and the value `blue`.

![A declaration highlighted in the CSS](https://mdn.mozillademos.org/files/16498/declaration.png)

A property paired with a value is called a *CSS declaration*. CSS declarations are put within *CSS Declaration Blocks*. This next image shows our CSS with the declaration block highlighted.

![A highlighted declaration block](https://mdn.mozillademos.org/files/16499/declaration-block.png)

Finally, CSS declaration blocks are paired with *selectors* to produce *CSS Rulesets* (or *CSS Rules*). Our image contains two rules, one for the `h1` selector and one for the `p` selector. The rule for `h1` is highlighted.

![The rule for h1 highlighted](https://mdn.mozillademos.org/files/16500/rules.png)

Setting CSS properties to specific values is the core function of the CSS language. The CSS engine calculates which declarations apply to every single element of a page in order to appropriately lay it out and style it. What is important to remember is that both properties and values are case-sensitive in CSS. The property and value in each pair is separated by a colon (`:`).

**Try looking up different values of the following properties, and writing CSS rules that apply them to different HTML elements:**

- **[`font-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-size)**
- **[`width`](https://developer.mozilla.org/en-US/docs/Web/CSS/width)**
- **[`background-color`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-color)**
- **[`color`](https://developer.mozilla.org/en-US/docs/Web/CSS/color)**
- **[`border`](https://developer.mozilla.org/en-US/docs/Web/CSS/border)**

**Important**: If a property is unknown or if a value is not valid for a given property, the declaration is deemed *invalid* and is completely ignored by the browser's CSS engine.

**Important**: In CSS (and other web standards), US spelling has been agreed on as the standard to stick to where language uncertainty arises. For example, `color` should *always* be spelled `color`. `colour` won't work.

### Functions



While most values are relatively simple keywords or numeric values, there are some possible values which take the form of a function. An example would be the `calc()` function. This function allows you to do simple math from within your CSS, for example:

```html
<div class="outer"><div class="box">The inner box is 90% - 30px.</div></div>

```

css

```css
.outer {
  border: 5px solid black;
}

.box {
  padding: 10px;
  width: calc(90% - 30px);
  background-color: rebeccapurple;
  color: white;
}
```



This renders like so:

A function consists of the function name, and then some brackets into which the allowed values for that function are placed. In the case of the `calc()` example above I am asking for the width of this box to be 90% of the containing block width, minus 30 pixels. This isn't something I can calculate ahead of time and just enter the value into the CSS, as I don't know what 90% will be. As with all values, the relevant page on MDN will have usage examples so you can see how the function works.

Another example would be the various values for `tansform`, such as `rotate()`.

```html
<div class="box"></div>
```

```css
.box {
  margin: 30px;
  width: 100px;
  height: 100px;
  background-color: rebeccapurple;
  transform: rotate(0.8turn)
}
```



The output from the above code looks like this:

**Try looking up different values of the following properties, and writing CSS rules that apply them to different HTML elements:**

- **[`transform`](https://developer.mozilla.org/en-US/docs/Web/CSS/transform)**
- **[`background-image`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-image), in particular gradient values**
- **[`color`](https://developer.mozilla.org/en-US/docs/Web/CSS/color), in particular rgb/rgba/hsl/hsla values**

## @rules

As yet, we have not encountered `@rules` (pronounced "at-rules"). These are special rules giving CSS some instruction on how to behave. Some `@rules` are simple with the rule name and a value. For example, to import an additional stylesheet into your main CSS stylesheet you can use `@import`:

```css
@import 'styles2.css';
```

One of the most common `@rules` you will come across is `@media`, which allows you to use [media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries) to apply CSS only when certain conditions are true (e.g. when the screen resolution is above a certain amount, or the screen is wider than a certain width).

In the below CSS, we have a stylesheet that gives the `<body>` element to have a pink background color. However, we then use `@media` to create a section of our stylesheet that will only be applied in browsers with a viewport wider than 30em. If the browser is wider than 30em then the background color will be blue.

```css
body {
  background-color: pink;
}

@media (min-width: 30em) {
  body {
    background-color: blue;
  }
}
```

You will encounter other `@rules` throughout these tutorials.

**See if you can add a media query to your CSS that changes styles based on the viewport width. Change the width of your browser window to see the result.**

## Shorthands

Some properties like `font`, `background`, `padding`, `border`, and `margin` are called **shorthand properties** — this is because they allow you to set several property values in a single line, saving time and making your code neater in the process.

For example, this line:

```css
/* In 4-value shorthands like padding and margin, the values are applied
   in the order top, right, bottom, left (clockwise from the top). There are also other 
   shorthand types, for example 2-value shorthands, which set padding/margin
   for top/bottom, then left/right */
padding: 10px 15px 15px 5px;
```

Does the same thing as all these together:

```css
padding-top: 10px;
padding-right: 15px;
padding-bottom: 15px;
padding-left: 5px;
```

Whereas this line:

```css
background: red url(bg-graphic.png) 10px 10px repeat-x fixed;
```

Does the same thing as all these together:

```css
background-color: red;
background-image: url(bg-graphic.png);
background-position: 10px 10px;
background-repeat: repeat-x;
background-scroll: fixed;
```

We won't attempt to teach these exhaustively now — you'll come across many examples later on in the course, and you are advised to look up the shorthand property names in our [CSS reference](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference) to find out more.

**Try adding the above declarations to your CSS to see how it affects the styling of your HTML. Try experimenting with some different values.**

**Warning**: While shorthands often allow you to miss out values, they will then reset any values that you do not include to their initial values. This ensures that a sensible set of values are used. However, this might be confusing if you were expecting the shorthand to only change the values you passed in.

## Comments

As with HTML, you are encouraged to make comments in your CSS, to help you understand how your code works when coming back to it after several months, and to help others coming to the code to work on it understand it.

Comments in CSS begin with `/*` and end with `*/`. In the below code block I have used comments to mark the start of different distinct code sections. This is useful to help you navigate your codebase as it gets larger — you can search for the comments in your code editor.

```css
/* Handle basic element styling */
/* -------------------------------------------------------------------------------------------- */
body {
  font: 1em/150% Helvetica, Arial, sans-serif; 
  padding: 1em; 
  margin: 0 auto; 
  max-width: 33em;
}

@media (min-width: 70em) {
  /* Let's special case the global font size. On large screen or window,
     we increase the font size for better readability */
  body {
    font-size: 130%;
  }
}

h1 {font-size: 1.5em;}

/* Handle specific elements nested in the DOM  */
/* -------------------------------------------------------------------------------------------- */
div p, #id:first-line {
  background-color: red; 
  background-style: none
}

div p{
  margin: 0; 
  padding: 1em;
}

div p + p {
  padding-top: 0;
}
```

Comments are also useful for temporarily *commenting out* certain parts of the code for testing purposes, for example if you are trying to find which part of your code is causing an error. In the next example I have commented out the rules for the `.special` selector.

```css
/*.special { 
  color: red; 
}*/

p { 
  color: blue; 
}
```

**Add some comments to your CSS, to get used to using them.**

## Whitespace

White space means actual spaces, tabs and new lines. In the same manner as HTML, the browser tends to ignore much of the whitespace inside your CSS; a lot of the whitespace is just there to aid readability.

In our first example below we have each declaration (and rule start/end) on its own line — this is arguably a good way to write CSS, as it makes it easy to maintain and understand:

```css
body {
  font: 1em/150% Helvetica, Arial, sans-serif;
  padding: 1em;
  margin: 0 auto;
  max-width: 33em;
}

@media (min-width: 70em) {
  body {
    font-size: 130%;
  }
}

h1 {
  font-size: 1.5em;
}

div p,
#id:first-line {
  background-color: red;
  background-style: none
}

div p {
  margin: 0;
  padding: 1em;
}

div p + p {
  padding-top: 0;
}
```

You could write exactly the same CSS like so, with most of the whitespace removed — this is functionally identical to the first example, but I'm sure you'll agree that it is somewhat harder to read:

```css
body {font: 1em/150% Helvetica, Arial, sans-serif; padding: 1em; margin: 0 auto; max-width: 33em;}
@media (min-width: 70em) { body {font-size: 130%;} }

h1 {font-size: 1.5em;}

div p, #id:first-line {background-color: red; background-style: none}
div p {margin: 0; padding: 1em;}
div p + p {padding-top: 0;}
```

The code layout you choose is usually a personal preference, although when you start to work in teams, you may find that the existing team has its own styleguide that specifies an agreed convention to follow.

The whitespace you do need to be careful of in CSS is the whitespace between the properties and their values. For example, the following declarations are valid CSS:

```css
margin: 0 auto;
padding-left: 10px;
```

But the following are invalid:

```css
margin: 0auto;
padding- left: 10px;
```

`0auto` is not recognised as a valid value for the `margin` property (`0` and `auto` are two separate values,) and the browser does not recognise `padding-` as a valid property. So you should always make sure to separate distinct values from one another by at least a space, but keep property names and property values together as single unbroken strings.

**Try playing with whitespace inside your CSS, to see what breaks things and what doesn't.**

## What's next?

It's useful to understand a little about how the browser takes your HTML and CSS and turns it into a webpage, so in the next article — [How CSS works](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps/How_CSS_works) — we will take a look at that process.

# How CSS works

 We have learned the basics of CSS, what it is for and how to write simple stylesheets. In this lesson we will take a look at how a browser takes CSS and HTML and turns that into a webpage. 

## How does CSS actually work?

When a browser displays a document, it must combine the document's content with its style information. It processes the document in a number of stages, which we've listed below. Bear in mind that this is a very simplified version of what happens when a browser loads a webpage, and that different browsers will handle the process in different ways. But this is roughly what happens.

1. The browser loads the HTML (e.g. receives it from the network).
2. It converts the [HTML](https://developer.mozilla.org/en-US/docs/Glossary/HTML) into a [DOM](https://developer.mozilla.org/en-US/docs/Glossary/DOM) (*Document Object Model*). The DOM represents the document in the computer's memory. The DOM is explained in a bit more detail in the next section.
3. The browser then fetches most of the resources that are linked to by the HTML document, such as embedded images and videos ... and linked CSS! JavaScript is handled a bit later on in the process, and we won't talk about it here to keep things simpler.
4. The browser parses the fetched CSS, and sorts the different rules by their selector types into different "buckets", e.g. element, class, ID, and so on. Based on the selectors it finds, it works out which rules should be applied to which nodes in the DOM, and attaches style to them as required (this intermediate step is called a render tree).
5. The render tree is laid out in the structure it should appear in after the rules have been applied to it.
6. The visual display of the page is shown on the screen (this stage is called painting).

The following diagram also offers a simple view of the process.

![img](https://mdn.mozillademos.org/files/11781/rendering.svg)

## About the DOM

A DOM has a tree-like structure. Each element, attribute, and piece of text in the markup language becomes a [DOM node](https://developer.mozilla.org/en-US/docs/Glossary/Node/DOM) in the tree structure. The nodes are defined by their relationship to other DOM nodes. Some elements are parents of child nodes, and child nodes have siblings.

Understanding the DOM helps you design, debug and maintain your CSS because the DOM is where your CSS and the document's content meet up. When you start working with browser DevTools you will be navigating the DOM as you select items in order to see which rules apply.

## A real DOM representation

Rather than a long, boring explanation, let's look at an example to see how a real HTML snippet is converted into a DOM.

Take the following HTML code:

```html
<p>
  Let's use:
  <span>Cascading</span>
  <span>Style</span>
  <span>Sheets</span>
</p>
```

In the DOM, the node corresponding to our `p` element is a parent. Its children are a text node and the three nodes corresponding to our `span` elements. The `SPAN` nodes are also parents, with text nodes as their children:

```html
P
├─ "Let's use:"
├─ SPAN
|  └─ "Cascading"
├─ SPAN
|  └─ "Style"
└─ SPAN
   └─ "Sheets"
```

This is how a browser interprets the previous HTML snippet —it renders the above DOM tree and then outputs it in the browser like so:

## Applying CSS to the DOM

Let's say we added some CSS to our document, to style it. Again, the HTML is as follows:

```html
<p>
  Let's use:
  <span>Cascading</span>
  <span>Style</span>
  <span>Sheets</span>
</p>
```

Let's suppose we apply the following CSS to it:

```css
span {
  border: 1px solid black;
  background-color: lime;
}
```

The browser will parse the HTML and create a DOM from it, then parse the CSS. Since the only rule available in the CSS has a `span` selector, the browser will be able to sort the CSS very quickly! It will apply that rule to each one of the three `span`s, then paint the final visual representation to the screen.

The updated output is as follows:

In our [Debugging CSS](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Debugging_CSS) article in the next module we will be using browser DevTools to debug CSS problems, and will learn more about how the browser interprets CSS.

## What happens if a browser encounters CSS it doesn't understand?

[In an earlier lesson](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps/What_is_CSS#Browser_support) I mentioned that browsers do not all implement new CSS at the same time. In addition, many people are not using the latest version of a browser. Given that CSS is being developed all the time, and is therefore ahead of what browsers can recognise, you might wonder what happens if a browser encounters a CSS selector or declaration it doesn't recognise.

The answer is that it does nothing, and just moves on to the next bit of CSS!

If a browser is parsing your rules, and encounters a property or value that it doesn't understand, it ignores it and moves on to the next declaration. It will do this if you have made an error and misspelled a property or value, or if the property or value is just too new and the browser doesn't yet support it.

Similarly, if a browser encounters a selector that it doesn't understand, it will just ignore the whole rule and move on to the next one.

In the below example I have used the British English spelling for color, which makes that property incorrect. So my paragraph has not been colored blue. All of the other CSS has been applied however; only the invalid line is ignored.

```html
<p> I want this text to be large, bold and blue.</p>
```

```css
p {
  font-weight: bold;
  colour: blue; /* incorrect spelling of the color property */
  font-size: 200%;
}
```

This behavior is very useful. It means that you can use new CSS as an enhancement, knowing that no error will occur if it is not understood — the browser will either get the new feature or not. Coupled with the way that the cascade works, and the fact that browsers will use the last CSS they come across in a stylesheet when you have two rules with the same specificity you can also offer alternatives for browsers that don't support new CSS.

This works particularly well when you want to use a value that is quite new and not supported everywhere. For example, some older browsers do not support `calc()` as a value. I might give a fallback width for a box in pixels, then go on to give a width with a `calc()` value of `100% - 50px`. Old browsers will use the pixel version, ignoring the line about `calc()` as they don't understand it. New browsers will interpret the line using pixels, but then override it with the line using `calc()` as that line appears later in the cascade.

```css
.box {
  width: 500px;
  width: calc(100% - 50px);
}
```

We will look at many more ways to support varying browsers in later lessons.

## And finally

You've nearly finished this module; we only have one more thing to do. In the next article you'll [use your new knowledge](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps/Using_your_new_knowledge) to restyle an example, testing out some CSS in the process.

# Using your new knowledge

With the things you have learned in the last few lessons you should find that you can format simple text documents using CSS, to add your own style to them. This article gives you a chance to do that. 

## Let's play with some CSS

The following live example shows a biography, which has been styled using CSS. The CSS properties that I have used are as follows — each one links to its property page on MDN, which will give you more examples of its use.

- [`font-family`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-family)
- [`color`](https://developer.mozilla.org/en-US/docs/Web/CSS/color)
- [`border-bottom`](https://developer.mozilla.org/en-US/docs/Web/CSS/border-bottom)
- [`font-weight`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-weight)
- [`font-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-size)
- [`text-decoration`](https://developer.mozilla.org/en-US/docs/Web/CSS/text-decoration)

I have used a mixture of selectors, styling elements such as h1 and h2, but also creating a class for the job title and styling that.

Try using CSS to change how this biography looks by changing the values of the properties I have used, removing some rules, or adding your own. Then try looking up some properties not mentioned on this page in the [MDN CSS reference](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference) and get adventurous!

For example you could:

- Change the colors.
- Change the size of the headings.
- Style the [`ul`](https://developer.mozilla.org/en-US/docs/Web/CSS/ul) and change the way the list of contact details displays.
- Add some exciting backgrounds and borders to some of the elements.
- Animate some parts of the HTML when they are hovered over (colors and sizes are obvious candidates for animation.)
- Add gradients or shadows.

Remember that there is no wrong answer here — at this stage in your learning you can afford to have a bit of fun.

## What's next?

Congratulations on finishing this first module. You should now have a good general understanding of CSS, and be able to understand much of what is happening in a stylesheet. In the next module, [CSS building blocks](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks), we will go on to look at a number of key areas in depth.