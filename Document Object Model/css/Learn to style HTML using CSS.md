# Learn to style HTML using CSS



Cascading Stylesheets — or [CSS](https://developer.mozilla.org/en-US/docs/Glossary/CSS) — is the first technology you should start learning after [HTML](https://developer.mozilla.org/en-US/docs/Glossary/HTML). While HTML is used to define the structure and semantics of your content, CSS is used to style it and lay it out. For example, you can use CSS to alter the font, color, size, and spacing of your content, split it into multiple columns, or add animations and other decorative features.

## Learning pathway

You should learn the basics of HTML before attempting any CSS. We recommend that you work through our [Introduction to HTML](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML) module first. In that module, you will learn about:

- CSS, starting with the [Introduction to CSS](https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS) module
- More advanced [HTML modules](https://developer.mozilla.org/en-US/docs/Learn/HTML#Modules)
- [JavaScript](https://developer.mozilla.org/en-US/docs/Learn/JavaScript), and how to use it to add dynamic functionality to web pages

Once you understand the fundamentals of HTML, we recommend that you learn HTML and CSS at the same time, moving back and forth between the two topics. This is because HTML is far more interesting and much more fun to learn when you apply CSS, and you can't really learn CSS without knowing HTML.

Before starting this topic, you should also be familiar with using computers and using the web passively (i.e., just looking at it, consuming the content). You should have a basic work environment set up as detailed in [Installing basic software](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/Installing_basic_software) and understand how to create and manage files, as detailed in [Dealing with files](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/Dealing_with_files) — both of which are parts of our [Getting started with the web](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web) complete beginner's module.

It is recommended that you work through [Getting started with the web](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web) before proceeding with this topic. However, doing so isn't absolutely necessary as much of what is covered in the CSS basics article is also covered in our [Introduction to CSS](https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS) module, albeit in a lot more detail.

## Modules

This topic contains the following modules, in a suggested order for working through them. You should definitely start with the first one.

- [CSS first steps](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps)

  CSS (Cascading Style Sheets) is used to style and lay out web pages — for example, to alter the font, color, size, and spacing of your content, split it into multiple columns, or add animations and other decorative features. This module provides a gentle beginning to your path towards CSS mastery with the basics of how it works, what the syntax looks like, and how you can start using it to add styling to HTML.

- [CSS building blocks](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks)

  This module carries on where [CSS first steps](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps) left off — now you've gained familiarity with the language and its syntax, and got some basic experience with using it, its time to dive a bit deeper. This module looks at the cascade and inheritance, all the selector types we have available, units, sizing, styling backgrounds and borders, debugging, and lots more. The aim here is to provide you with a toolkit for writing competent CSS and help you understand all the essential theory, before moving on to more specific disciplines like [text styling](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text) and [CSS layout](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout).

- [Styling text](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text)

  With the basics of the CSS language covered, the next CSS topic for you to concentrate on is styling text — one of the most common things you'll do with CSS. Here we look at text styling fundamentals, including setting font, boldness, italics, line and letter spacing, drop shadows and other text features. We round off the module by looking at applying custom fonts to your page, and styling lists and links.

- [CSS layout](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout)

  At this point we've already looked at CSS fundamentals, how to style text, and how to style and manipulate the boxes that your content sits inside. Now it's time to look at how to place your boxes in the right place in relation to the viewport, and one another. We have covered the necessary prerequisites so we can now dive deep into CSS layout, looking at different display settings, modern layout tools like flexbox, CSS grid, and positioning, and some of the legacy techniques you might still want to know about.

## Solving common CSS problems

[Use CSS to solve common problems](https://developer.mozilla.org/en-US/docs/Learn/CSS/Howto) provides links to sections of content explaining how to use CSS to solve very common problems when creating a web page.

From the beginning, you'll primarily apply colors to HTML elements and their backgrounds; change the size, shape, and position of elements; and add and define borders on elements. But there's not much you can't do once you have a solid understanding of even the basics of CSS. One of the best things about learning CSS is that once you know the fundamentals, usually you have a pretty good feel for what can and can't be done, even if you don't actually know how to do it yet!

## See also

- [CSS on MDN](https://developer.mozilla.org/en-US/docs/Web/CSS)

  The main entry point for CSS documentation on MDN, where you'll find detailed reference documentation for all features of the CSS language. Want to know all the values a property can take? This is a good place to go.

# CSS: Cascading Style Sheets

**Cascading Style Sheets** (**CSS**) is a [stylesheet](https://developer.mozilla.org/en-US/docs/DOM/stylesheet) language used to describe the presentation of a document written in [HTML](https://developer.mozilla.org/en-US/docs/Web/HTML) or [XML](https://developer.mozilla.org/en-US/docs/XML_introduction) (including XML dialects such as [SVG](https://developer.mozilla.org/en-US/docs/Web/SVG), [MathML](https://developer.mozilla.org/en-US/docs/Web/MathML) or [XHTML](https://developer.mozilla.org/en-US/docs/Glossary/XHTML)). CSS describes how elements should be rendered on screen, on paper, in speech, or on other media.

CSS is one of the core languages of the **open Web** and is standardized across Web browsers according to the [W3C specification](http://w3.org/Style/CSS/#specs). Developed in levels, CSS1 is now obsolete, CSS2.1 is a recommendation, and [CSS3](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS3), now split into smaller modules, is progressing on the standardization track.

# CSS basics

CSS (Cascading Style Sheets) is the code you use to style your webpage. *CSS basics* takes you through what you need to get started. We'll answer questions like: How do I make my text black or red? How do I make my content show up in such-and-such a place on the screen? How do I decorate my webpage with background images and colors?

## So what is CSS, really?

Like HTML, CSS is not really a programming language. It is not a *markup language* either — it is a *style sheet language*. This means that it lets you apply styles selectively to elements in HTML documents. For example, to select **all** the paragraph elements on an HTML page and turn the text within them red, you'd write this CSS:

```css
p {
  color: red;
}
```

Let's try it out: paste those three lines of CSS into a new file in your text editor, and then save the file as `style.css` in your `styles` directory.

But we still need to apply the CSS to your HTML document. Otherwise, the CSS styling won't affect how your browser displays the HTML document. (If you haven't been following on with our project, read [Dealing with files](https://developer.mozilla.org/en-US/Learn/Getting_started_with_the_web/Dealing_with_files) and [HTML basics](https://developer.mozilla.org/en-US/Learn/Getting_started_with_the_web/HTML_basics) to find out what you need to do first.)

1. Open your `index.html`  file and paste the following line somewhere in the head (that is, between the  `<head>` and `</head>` tags):

   ```html
   <link href="styles/style.css" rel="stylesheet">
   ```

2. Save `index.html` and load it in your browser. You should see something like this:

If your paragraph text is now red, congratulations! You've just written your first successful CSS.

### Anatomy of a CSS ruleset



Let's look at the above CSS in a bit more detail:

The whole structure is called a **rule set** (but often "rule" for short). Note also the names of the individual parts:

- Selector

  The HTML element name at the start of the rule set. It selects the element(s) to be styled (in this case, `<p>` elements). To style a different element, just change the selector.

- Declaration

  A single rule like `color: red;` specifying which of the element's **properties** you want to style.

- Properties

  Ways in which you can style a given HTML element. (In this case, `color` is a property of the `<p>` elements.) In CSS, you choose which properties you want to affect in your rule.

- Property value

  To the right of the property after the colon, we have the **property value**, which chooses one out of many possible appearances for a given property (there are many `color` values besides `red`).

Note the other important parts of the syntax:

- Each rule set (apart from the selector) must be wrapped in curly braces (`{}`).
- Within each declaration, you must use a colon (`:`) to separate the property from its values.
- Within each rule set, you must use a semicolon (`;`) to separate each declaration from the next one.

```js
rule set:{
    selector,
        declarations:{
            property,value
        }
}
```



So to modify multiple property values at once, you just need to write them separated by semicolons, like this:

```css
p {
  color: red;
  width: 500px;
  border: 1px solid black;
}
```

### Selecting multiple elements



You can also select multiple types of elements and apply a single rule set to all of them. Include multiple selectors separated by commas. For example:

```css
p, li, h1 {
  color: red;
}
```

### Different types of selectors



There are many different types of selectors. Above, we only looked at **element selectors**, which select all elements of a given type in the given HTML documents. But we can make more specific selections than that. Here are some of the more common types of selectors:

Element selector (sometimes called a tag or type selector)
All HTML element(s) of the specified type.
`p` Selects `<p>`

ID selector
The element on the page with the specified ID. On a given HTML page, it's a best practice to use one element per ID (and of course one ID per element) even though you are allowed to use same ID for multiple elements.
`#my-id` Selects `  <p id="my-id">  ` and `  <a id="my-id">  `

Class selector
The element(s) on the page with the specified class (multiple class instances can appear on a page).
`.my-class` Selects `  <p class="my-class">  ` and `  <a class="my-class">  `

Attribute selector
The element(s) on the page with the specified attribute.
`img[src]` Selects `  <img src="myimage.png">  ` but not `  <img>  `

Pseudo-class selector
The specified element(s), but only when in the specified state (e.g. being hovered over).
`a:hover` Selects `  <a>  `, but only when the mouse pointer is hovering over the link.



There are many more selectors to explore, and you can find a more detailed list in our [Selectors guide](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Getting_started/Selectors).

## Fonts and text

Now that we've explored some CSS basics, let's start adding some more rules and information to our `style.css` file to make our example look nice. Let's start by getting our fonts and text to look a little better.

1. First of all, go back and find the output from Google Fonts that you stored somewhere safe. Add the `<>` element somewhere inside your `index.html`'s head (again, anywhere between the `<head>` and `</head>` tags). It'll look something like this:

   ```html
   <link href="https://fonts.googleapis.com/css?family=Open+Sans" rel="stylesheet">
   ```

   This code links your page to a stylesheet that downloads the Open Sans font family along with your web page and enables you to set it on your HTML elements using your own style sheet.

2. Next, delete the existing rule you have in your `style.css` file. It was a good test, but red text doesn't actually look very good.

3. Add the following lines in its place, replacing the placeholder line with the actual `font-family` line you got from Google Fonts. (`font-family` just means the font(s) you want to use for your text.) This rule first sets a global base font and font size for the whole page (since `<html>` is the parent element of the whole page, and all elements inside it inherit the same `font-size` and `font-family`):

   ```css
   html {
     font-size: 10px; /* px means "pixels": the base font size is now 10 pixels high  */
     font-family: "Open Sans", sans-serif; /* this should be the rest of the output you got from Google fonts */
   }
   ```

   **Note**: Anything in a CSS document between `/*` and `*/` is a **CSS comment**, which the browser ignores when it renders the code. This is a place for you to write helpful notes on what you are doing.

4. Now we'll set font sizes for text-containing elements inside the HTML body (`<h1>`,`<p>`, and `<li>`). We'll also center the text of our heading and set some line height and letter spacing on the body content to make it a bit more readable:

   ```css
h1 {
     font-size: 60px;
    text-align: center;
   }

   p, li {
    font-size: 16px;
     line-height: 2;
    letter-spacing: 1px;
   }
   ```

You can adjust these `px` values to whatever you like to get your design looking how you want, but in general your design should look like this:

## Boxes, boxes, it's all about boxes

One thing you'll notice about writing CSS is that a lot of it is about boxes — setting their size, color, position, etc. Most of the HTML elements on your page can be thought of as boxes sitting on top of each other.

![a big stack of boxes or crates sat on top of one another](https://mdn.mozillademos.org/files/9441/boxes.jpg)

Not surprisingly, CSS layout is based principally on the *box model.* Each of the blocks taking up space on your page has properties like this:

- `padding`, the space just around the content (e.g., around paragraph text).
- `border`, the solid line that sits just outside the padding.
- `margin`, the space around the outside of the element.

![three boxes sat inside one another. From outside to in they are labelled margin, border and padding](https://mdn.mozillademos.org/files/9443/box-model.png)

In this section we also use:

- `width` (of an element).
- `background-color`, the color behind an element's content and padding.
- `color`, the color of an element's content (usually text).
- `text-shadow`: sets a drop shadow on the text inside an element.
- `display`: sets the display mode of an element (don't worry about this yet).

So, let's get started and add some more CSS to our page! Keep adding these new rules to the bottom of the page, and don't be afraid to experiment with changing values to see how it turns out.

### Changing the page color



```css
html {
  background-color: #00539F;
}
```

This rule sets a background color for the whole page. Change the color code above to whatever color [you chose when planning your site](https://developer.mozilla.org/en-US/Learn/Getting_started_with_the_web/What_should_your_web_site_be_like#Theme_color).

### Sorting the body out



```css
body {
  width: 600px;
  margin: 0 auto;
  background-color: #FF9500;
  padding: 0 20px 20px 20px;
  border: 5px solid black;
}
```

Now for the `<body>` element. There are quite a few declarations here, so let's go through them all one by one:

- `width: 600px;` — this forces the body to always be 600 pixels wide.
- `margin: 0 auto;` — When you set two values on a property like `margin` or `padding`, the first value affects the element's top **and** bottom side (make it `0` in this case), and the second value the left **and** right side (here, `auto` is a special value that divides the available horizontal space evenly between left and right). You can also use one, three, or four values, as documented [here](https://developer.mozilla.org/en-US/docs/Web/CSS/margin#Syntax).
- `background-color: #FF9500;` — as before, this sets the element's background color. We've used a sort of reddish orange for the body as opposed to dark blue for the `<html>` element, but feel free to go ahead and experiment.
- `padding: 0 20px 20px 20px;` — we have four values set on the padding, to make a bit of space around our content. This time we are setting no padding on the top of the body, and 20 pixels on the left, bottom and right. The values set top, right, bottom, left, in that order. As with `margin`, you can also use one, two or three values, as documented on [padding syntax](https://developer.mozilla.org/en-US/docs/Web/CSS/padding#Syntax).
- `border: 5px solid black;` — this simply sets a 5-pixel–wide, solid black border on all sides of the body.

### Positioning and styling our main page title



```css
h1 {
  margin: 0;
  padding: 20px 0;    
  color: #00539F;
  text-shadow: 3px 3px 1px black;
}
```

You may have noticed there's a horrible gap at the top of the body. That happens because browsers apply some **default styling** to the `<h1>` element (among others), even when you haven't applied any CSS at all! That might sound like a bad idea, but we want even an unstyled webpage to have basic readability. To get rid of the gap we overrode the default styling by setting `margin: 0;`.

Next up, we've set the heading's top and bottom padding to 20 pixels, and made the heading text the same color as the HTML background color.

One rather interesting property we've used here is `text-shadow`, which applies a text shadow to the text content of the element. Its four values are as follows:

- The first pixel value sets the **horizontal offset** of the shadow from the text — how far it moves across: a negative value should move it to the left.
- The second pixel value sets the **vertical offset** of the shadow from the text — how far it moves down, in this example; a negative value should move it up.
- The third pixel value sets the **blur radius** of the shadow — a bigger value will mean a more blurry shadow.
- The fourth value sets the base color of the shadow.

Again, try experimenting with different values to see what you can come up with!

### Centering the image



```css
img {
  display: block;
  margin: 0 auto;
}
```

Finally, we'll center the image to make it look better. We could use the `margin: 0 auto` trick again as we did earlier for the body, but we also need to do something else. The `<body>` element is **block level**, meaning it takes up space on the page and can have margin and other spacing values applied to it. Images, on the other hand, are **inline** elements, meaning they can't. So to apply margins to the image, we have to give the image block-level behavior using `display: block;`.

**Note**: The instructions above assume that you're using an image smaller than the width set on the body (600 pixels). If your image is larger, then it will overflow the body and spill out to the rest of the page. To rectify this, you can either 1) reduce the image's width using a [graphics editor](https://en.wikipedia.org/wiki/Raster_graphics_editor), or 2) size the image using CSS by setting the `width` property on the `<img>` element with a smaller value (e.g., `400 px;`).

**Note**: Don't worry if you don't yet understand `display: block;` and the block-level/inline distinction. You will as you study CSS in more depth. You can find out more about the different available display values at our [display reference page](https://developer.mozilla.org/en-US/docs/Web/CSS/display).

## Conclusion

If you have followed all the instructions in this article, you should end up with a page that looks something like this (you can also [view our version here](http://mdn.github.io/beginner-html-site-styled/)):

![a mozilla logo, centered, and a header and paragraphs. It now looks nicely styled, with a blue background for the whole page and orange background for the centered main content strip.](https://mdn.mozillademos.org/files/9455/website-screenshot-final.png)

If you get stuck, you can always compare your work with our [finished example code on GitHub](https://github.com/mdn/beginner-html-site-styled/blob/gh-pages/styles/style.css).

Here, we have only really scratched the surface of CSS. To find out more, go to our [CSS Learning topic](https://developer.mozilla.org/en-US/Learn/CSS).

# CSS first steps

CSS (Cascading Style Sheets) is used to style and lay out web pages — for example, to alter the font, color, size, and spacing of your content, split it into multiple columns, or add animations and other decorative features. This module provides a gentle beginning to your path towards CSS mastery with the basics of how it works, what the syntax looks like, and how you can start using it to add styling to HTML.

## Prerequisites

Before starting this module, you should have:

1. Basic familiarity with using computers, and using the Web passively (i.e. looking at it, consuming the content.)
2. A basic work environment set up as detailed in [Installing basic software](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/Installing_basic_software), and an understanding of how to create and manage files, as detailed in [Dealing with files](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/Dealing_with_files).
3. Basic familiarity with HTML, as discussed in the [Introduction to HTML](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML) module.

**Note**: If you are working on a computer/tablet/other device where you don't have the ability to create your own files, you could try out (most of) the code examples in an online coding program such as [JSBin](http://jsbin.com/) or [Thimble](https://thimble.mozilla.org/).

## Guides

This module contains the following articles, which will take you through all the basic theory of CSS, and provide opportunities for you to test out some skills.

- [What is CSS?](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps/What_is_CSS)

  **[CSS](https://developer.mozilla.org/en-US/docs/Glossary/CSS)** (Cascading Style Sheets) allows you to create great-looking web pages, but how does it work under the hood? This article explains what CSS is, with a simple syntax example, and also covers some key terms about the language.

- [Getting started with CSS](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps/Getting_started)

  In this article we will take a simple HTML document and apply CSS to it, learning some practical things about the language along the way.

- [How CSS is structured](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps/How_CSS_is_structured)

  Now that you have an idea about what CSS is and the basics of using it, it is time to look a little deeper into the structure of the language itself. We have already met many of the concepts discussed here; you can return to this one to recap if you find any later concepts confusing.

- [How CSS works](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps/How_CSS_works)

  We have learned the basics of CSS, what it is for and how to write simple stylesheets. In this lesson we will take a look at how a browser takes CSS and HTML and turns that into a webpage.

- [Using your new knowledge](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps/Using_your_new_knowledge)

  With the things you have learned in the last few lessons you should find that you can format simple text documents using CSS, to add your own style to them. This article gives you a chance to do that.

## See also

- [Intermediate Web Literacy 1: Intro to CSS](https://teach.mozilla.org/activities/intermediate-web-lit/)

  An excellent Mozilla foundation course that explores and tests a lot of the skills talked about in the *Introduction to CSS* module. Learn about styling HTML elements on a webpage, CSS selectors, attributes, and values.

# What is CSS?

**[CSS](https://developer.mozilla.org/en-US/docs/Glossary/CSS)** (Cascading Style Sheets) allows you to create great-looking web pages, but how does it work under the hood? This article explains what CSS is, with a simple syntax example, and also covers some key terms about the language. 

In the [Introduction to HTML](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML) module we covered what HTML is, and how it is used to mark up documents. These documents will be readable in a web browser. Headings will look larger than regular text, paragraphs break onto a new line and have space between them. Links are colored and underlined to distinguish them from the rest of the text. What you are seeing is the browser's default styles — very basic styles that the browser applies to HTML to make sure it will be basically readable even if no explicit styling is specified by the author of the page. 

However, the web would be a boring place if all websites looked like that. Using CSS you can control exactly how HTML elements look in the browser, presenting your markup using whatever design you like. 

## What is CSS for?

As we have mentioned before, CSS is a language for specifying how documents are presented to users — how they are styled, laid out, etc.

A **document** is usually a text file structured using a markup language — [HTML](https://developer.mozilla.org/en-US/docs/Glossary/HTML) is the most common markup language, but you may also come across other markup languages such as [SVG](https://developer.mozilla.org/en-US/docs/Glossary/SVG) or [XML](https://developer.mozilla.org/en-US/docs/Glossary/XML).

**Presenting** a document to a user means converting it into a form usable by your audience. [Browsers](https://developer.mozilla.org/en-US/docs/Glossary/browser), like [Firefox](https://developer.mozilla.org/en-US/docs/Glossary/Mozilla_Firefox), [Chrome](https://developer.mozilla.org/en-US/docs/Glossary/Google_Chrome), or [Edge](https://developer.mozilla.org/en-US/docs/Glossary/Microsoft_Edge) , are designed to present documents visually, for example, on a computer screen, projector or printer.

**Note**: A browser is sometimes called a [user agent](https://developer.mozilla.org/en-US/docs/Glossary/User_agent), which basically means a computer program that represents a person inside a computer system. Browsers are the main type of user agent we think of when talking about CSS, however, it is not the only one. There are other user agents available — such as those which convert HTML and CSS documents into PDFs to be printed.

CSS can be used for very basic document text styling — for example changing the [color](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value) and [size](https://developer.mozilla.org/en-US/docs/Web/CSS/font-size) of headings and links. It can be used to create layout — for example [turning a single column of text into a layout](https://developer.mozilla.org/en-US/docs/Web/CSS/Layout_cookbook/Column_layouts) with a main content area and a sidebar for related information. It can even be used for effects such as [animation](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations). Have a look at the links in this paragraph for specific examples.

## CSS syntax

CSS is a rule-based language — you define rules specifying groups of styles that should be applied to particular elements or groups of elements on your web page. For example "I want the main heading on my page to be shown as large red text."

The following code shows a very simple CSS rule that would achieve the styling described above:

```css
h1 {
    color: red;
    font-size: 5em;
}
```

The rule opens with a [selector](https://developer.mozilla.org/en-US/docs/Glossary/CSS_Selector) . This *selects* the HTML element that we are going to style. In this case we are styling level one headings (`<h1>`).

We then have a set of curly braces `{ }`. Inside those will be one or more **declarations**, which take the form of **property** and **value** pairs. Each pair specifies a property of the element(s) we are selecting, then a value that we'd like to give the property.

Before the colon, we have the property, and after the colon, the value. CSS [properties](https://developer.mozilla.org/en-US/docs/Glossary/property/CSS) have different allowable values, depending on which property is being specified. In our example, we have the `color` property, which can take various [color values](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units#Color). We also have the `font-size` property. This property can take various [size units](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units#Numbers_lengths_and_percentages) as a value.

A CSS stylesheet will contain many such rules, written one after the other.

```css
h1 {
    color: red;
    font-size: 5em;
}

p {
    color: black;
}
```

You will find that you quickly learn some values, whereas others you will need to look up. The individual property pages on MDN give you a quick way to look up properties and their values when you forget, or want to know what else you can use as a value.

**Note**: You can find links to all the CSS property pages (along with other CSS features) listed on the MDN [CSS reference](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference). Alternatively, you should get used to searching for "mdn *css-feature-name*" in your favourite search engine whenever you need to find out more information about a CSS feature. For example, try searching for "mdn color" and "mdn font-size"!

## CSS Modules

As there are so many things that you could style using CSS, the language is broken down into *modules*. You'll see reference to these modules as you explore MDN and many of the documentation pages are organized around a particular module. For example, you could take a look at the MDN reference to the [Backgrounds and Borders](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Backgrounds_and_Borders) module to find out what its purpose is, and what different properties and other features it contains. You will also find links to the *CSS Specification* that defines the technology (see below).

At this stage you don't need to worry too much about how CSS is structured, however it can make it easier to find information if, for example, you are aware that a certain property is likely to be found among other similar things and are therefore probably in the same specification. 

For a specific example, let's go back to the Backgrounds and Borders module — you might think that it makes logical sense for the `background-color` and `border-color` properties to be defined in this module. And you'd be right.

### CSS Specifications



All web standards technologies (HTML, CSS, JavaScript, etc.) are defined in giant documents called specifications (or simply "specs"), which are published by standards organizations (such as the [W3C](https://developer.mozilla.org/en-US/docs/Glossary/W3C), [WHATWG](https://developer.mozilla.org/en-US/docs/Glossary/WHATWG), [ECMA](https://developer.mozilla.org/en-US/docs/Glossary/ECMA), or [Khronos](https://developer.mozilla.org/en-US/docs/Glossary/Khronos)) and define precisely how those technologies are supposed to behave.

CSS is no different — it is developed by a group within the W3C called the [CSS Working Group](https://www.w3.org/Style/CSS/). This group is made of representatives of browser vendors and other companies who have an interest in CSS. There are also other people, known as *invited experts*, who act as independent voices; they are not linked to a member organization.

New CSS features are developed, or specified, by the CSS Working Group. Sometimes because a particular browser is interested in having some capability, other times because web designers and developers are asking for a feature, and sometimes because the Working Group itself has identified a requirement. CSS is constantly developing, with new features coming available. However, a key thing about CSS is that everyone works very hard to never change things in a way that would break old websites. A website built in 2000, using the limited CSS available then, should still be usable in a browser today!

As a newcomer to CSS, it is likely that you will find the CSS specs overwhelming — they are intended for engineers to use to implement support for the features in user agents, not for web developers to read to understand CSS. Many experienced developers would much rather refer to MDN documentation or other tutorials. It is however worth knowing that they exist, understanding the relationship between the CSS you are using, browser support (see below), and the specs.

## What's next

Now that you have some understanding of what CSS is, let's move on to [Getting started with CSS](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps/Getting_started), where you can start to write some CSS yourself.