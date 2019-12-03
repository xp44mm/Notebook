# Block-level elements

HTML (**Hypertext Markup Language**) elements historically were categorized as either "block-level" elements or "inline" elements. By default, a block-level element occupies the entire space of its parent element (container), thereby creating a "block." This article helps to explain what this means.

Browsers typically display the block-level element with a newline both before and after the element. You can visualize them as a stack of boxes.

A block-level element always starts on a new line and takes up the full width available (stretches out to the left and right as far as it can).

The following example demonstrates the block-level element's influence:

## Block-level elements

### HTML



```html
<p>This paragraph is a block-level element; its background has been colored to display the paragraph's parent element.</p>
```

### CSS



```css
p { background-color: #8ABB55; }
```

## Usage

- Block-level elements may appear only within a `<body>` element.

## Block-level vs. inline

There are a couple of key differences between block-level elements and inline elements:

- Content model

  Generally, block-level elements may contain inline elements and (sometimes) other block-level elements. Inherent in this structural distinction is the idea that block elements create "larger" structures than inline elements.

- Default formatting

  By default, block-level elements begin on new lines, but inline elements can start anywhere in a line.

The distinction of block-level vs. inline elements was used in HTML specifications up to 4.01. In HTML5, this binary distinction is replaced with a more complex set of [content categories](https://developer.mozilla.org/en-US/docs/HTML/Content_categories). While the "inline" category roughly corresponds to the category of [phrasing content](https://developer.mozilla.org/en-US/docs/HTML/Content_categories#Phrasing_content), the "block-level" category doesn't directly correspond to any HTML5 content category, but *"block-level" and "inline" elements combined together* correspond to the [flow content](https://developer.mozilla.org/en-US/docs/HTML/Content_categories#Flow_content) in HTML5. There are also additional categories, e.g. [interactive content](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Content_categories#Interactive_content).

## Elements

The following is a complete list of all HTML "block-level" elements (although "block-level" is not technically defined for elements that are new in HTML5).

```html
<address>
<article>
<aside>
<blockquote>
<dd>
<details>
<dialog>
<div>
<dl>
<dt>
<fieldset>
<figcaption>
<figure>
<footer>
<form>
<h1>
<h2>
<h3>
<h4>
<h5>
<h6>
<header>
<hgroup>
<hr>
<li>
<main>
<nav>
<ol>
<p>
<pre>
<section>
<table>
<ul>
```

# Inline elements

HTML (**Hypertext Markup Language**) elements historically were categorized as either "block-level" elements or "inline" elements. Inline elements are those which only occupy the space bounded by the tags defining the element, instead of breaking the flow of the content. In this article, we'll examine HTML inline elements and how they differ from block-level elements.

An inline element does not start on a new line and only takes up as much width as necessary.

## Inline vs. block-level elements: a demonstration

This is most easily demonstrated with a simple example. First, some simple CSS that we'll be using:

```css
.highlight {
  background-color:#ee3;
}
```

### Inline



Let's look at the following example which demonstrates an inline element:

```html
<div>The following span is an <span class="highlight">inline element</span>;
its background has been colored to display both the beginning and end of
the inline element's influence.</div>
```

In this example, the `<div>` block-level element contains some text. Within that text is a `<span>` element, which is an inline element. Because the `<span>` element is inline, the paragraph correctly renders as a single, unbroken text flow, like this:

### Block-level



Now let's change that `<span>` into a block-level element, such as `<p>`:

```html
<div>The following paragraph is a <p class="highlight">block-level element;</p>
its background has been colored to display both the beginning and end of
the block-level element's influence.</div>
```

Rendered using the same CSS as before, we get:

See the difference? The `<p>` element totally changes the layout of the text, splitting it into three segments: the text before the `<p>`, then the `<p>`'s text, and finally the text following the `<p>`.

### Changing element levels



You can change the *visual presentation* of an element using the CSS [`display`](https://developer.mozilla.org/en-US/docs/Web/CSS/display) property. For example, by changing the value of `display` from `"inline"` to `"block"`, you can tell the browser to render the inline element in a block box rather than an inline box, and vice versa. However, doing this will not change the *category* and the *content model* of the element. For example, even if the `display` of the `span` element is changed to `"block"`, it still would not allow to nest a `div` element inside it.

## Conceptual differences

In brief, here are the basic conceptual differences between inline and block-level elements:

- Content model

  Generally, inline elements may contain only data and other inline elements. You can't put block elements inside inline elements.

- Formatting

  By default, inline elements do not force a new line to begin in the document flow. Block elements, on the other hand, typically cause a line break to occur (although, as usual, this can be changed using CSS).

## List of "inline" elements

The following elements are inline by default (although block and inline elements are no longer defined in HTML 5, use [content categories](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Content_categories) instead):

```html
<a>
<abbr>
<acronym>
<audio>
<b>
<bdi>
<bdo>
<big>
<br>
<button>
<canvas>
<cite>
<code>
<data>
<datalist>
<del>
<dfn>
<em>
<embed>
<i>
<iframe>
<img>
<input>
<ins>
<kbd>
<label>
<map>
<mark>
<meter>
<noscript>
<object>
<output>
<picture>
<progress>
<q>
<ruby>
<s>
<samp>
<script>
<select>
<slot>
<small>
<span>
<strong>
<sub>
<sup>
<svg>
<template>
<textarea>
<time>
<tt>
<u>
<var>
<video>
<wbr>
```

