# Styling text

With the basics of the CSS language covered, the next CSS topic for you to concentrate on is styling text — one of the most common things you'll do with CSS. Here we look at text styling fundamentals including setting font, boldness, italics, line and letter spacing, drop shadows, and other text features. We round off the module by looking at applying custom fonts to your page, and styling lists and links.

## Prerequisites

Before starting this module, you should already have basic familiarity with HTML, as discussed in the [Introduction to HTML](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML) module, and be comfortable with CSS fundamentals, as discussed in [Introduction to CSS](https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS).

## Guides

This module contains the following articles, which will teach you all of the essentials behind styling HTML text content.

- [Fundamental text and font styling](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text/Fundamentals)

  In this article we go through all the basics of text/font styling in detail, including setting font weight, family and style, font shorthand, text alignment and other effects, and line and letter spacing.

- [Styling lists](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text/Styling_lists)

  Lists behave like any other text for the most part, but there are some CSS properties specific to lists that you need to know about, and some best practices to consider. This article explains all.

- [Styling links](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text/Styling_links)

  When styling links, it is important to understand how to make use of pseudo-classes to style link states effectively, and how to style links for use in common varied interface features such as navigation menus and tabs. We'll look at all these topics in this article.

- [Web fonts](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text/Web_fonts)

  Here we will explore web fonts in detail — these allow you to download custom fonts along with your web page, to allow for more varied, custom text styling.

## Assessments

The following assessments will test your understanding of the text styling techniques covered in the guides above.

- [Typesetting a community school homepage](https://developer.mozilla.org/en-US/Learn/CSS/Styling_text/Typesetting_a_homepage)

  In this assessment we'll test your understanding of styling text by getting you to style the text for a community school's homepage.

# Fundamental text and font styling

In this article we'll start you on your journey towards mastering text styling with [CSS](https://developer.mozilla.org/en-US/docs/Glossary/CSS). Here we'll go through all the basic fundamentals of text/font styling in detail, including setting font weight, family and style, font shorthand, text alignment and other effects, and line and letter spacing.

## What is involved in styling text in CSS?

As you'll have already experienced in your work with HTML and CSS, text inside an element is laid out inside the element's content box. It starts at the top left of the content area (or the top right, in the case of RTL language content), and flows towards the end of the line. Once it reaches the end, it goes down to the next line and continues, then the next line, until all the content has been placed in the box. Text content effectively behaves like a series of inline elements, being laid out on lines adjacent to one another, and not creating line breaks until the end of the line is reached, or unless you force a line break manually using the [` `](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/br) element.

**Note**: If the above paragraph leaves you feeling confused, then no matter — go back and review our [Box model](https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS/Box_model) article, to brush up on the box model theory, before carrying on.

The CSS properties used to style text generally fall into two categories, which we'll look at separately in this article:

- **Font styles**: Properties that affect the font that is applied to the text, affecting what font is applied, how big it is, whether it is bold, italic, etc.
- **Text layout styles**: Properties that affect the spacing and other layout features of the text, allowing manipulation of, for example, the space between lines and letters, and how the text is aligned within the content box.

**Note**: Bear in mind that the text inside an element is all affected as one single entity. You can't select and style subsections of text unless you wrap them in an appropriate element (such as a `<span>` or `<strong>`), or use a text-specific pseudo-element like `::first-letter` (selects the first letter of an element's text), `::first-line` (selects the first line of an element's text), or `::selection` (selects the text currently highlighted by the cursor.)

## Fonts

Let's move straight on to look at properties for styling fonts. In this example we'll apply some different CSS properties to the same HTML sample, which looks like this:

```html
<h1>Tommy the cat</h1>
<p>I remember as if it were a meal ago...</p>
<p>Said Tommy the Cat as he reeled back to clear whatever foreign matter may have nestled its way into his mighty throat. Many a fat alley rat had met its demise while staring point blank down the cavernous barrel of this awesome prowling machine. Truly a wonder of nature this urban predator — Tommy the cat had many a story to tell. But it was a rare occasion such as this that he did.</p>
```

You can find the [finished example on Github](http://mdn.github.io/learning-area/css/styling-text/fundamentals/) (see also [the source code](https://github.com/mdn/learning-area/blob/master/css/styling-text/fundamentals/index.html).)

### Color



The [`color`](https://developer.mozilla.org/en-US/docs/Web/CSS/color) property sets the color of the foreground content of the selected elements (which is usually the text, but can also include a couple of other things, such as an underline or overline placed on text using the [`text-decoration`](https://developer.mozilla.org/en-US/docs/Web/CSS/text-decoration) property).

`color` can accept any [CSS color unit](https://developer.mozilla.org/en-US/Learn/CSS/Introduction_to_CSS/Values_and_units#Colors), for example:

```css
p {
  color: red;
}
```

This will cause the paragraphs to become red, rather than the standard browser default black, like so:

### Font families



To set a different font on your text, you use the [`font-family`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-family) property — this allows you to specify a font (or list of fonts) for the browser to apply to the selected elements. The browser will only apply a font if it is available on the machine the website is being accessed on; if not, it will just use a browser [default font](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text/Fundamentals#Default_fonts). A simple example looks like so:

```css
p {
  font-family: arial;
}
```

This would make all paragraphs on a page adopt the arial font, which is found on any computer.

#### Web safe fonts

Speaking of font availability, there are only a certain number of fonts that are generally available across all systems and can therefore be used without much worry. These are the so-called **web safe fonts**.

Most of the time, as web developers we want to have more specific control over the fonts used to display our text content. The problem is to find a way to know which font is available on the computer used to see our web pages. There is no way to know this in every case, but the web safe fonts are known to be available on nearly all instances of the most used operating systems (Windows, Mac, the most common Linux distributions, Android, and iOS.)

The list of actual web safe fonts will change as operating systems evolve, but it's okay to consider the following fonts web safe, at least for now (many of them have been popularized thanks to the Microsoft *[Core fonts for the Web](https://en.wikipedia.org/wiki/Core_fonts_for_the_Web)* initiative in the late 90s and early 2000s):

| Name            | Generic type | Notes                                                        |
| :-------------- | :----------- | :----------------------------------------------------------- |
| Arial           | sans-serif   | It's often considered best practice to also add *Helvetica* as a preferred alternative to *Arial* as, although their font faces are almost identical, *Helvetica* is considered to have a nicer shape, even if *Arial* is more broadly available. |
| Courier New     | monospace    | Some OSes have an alternative (possibly older) version of the *Courier New* font called *Courier*. It's considered best practice to use both with *Courier New* as the preferred alternative. |
| Georgia         | serif        |                                                              |
| Times New Roman | serif        | Some OSes have an alternative (possibly older) version of the *Times New Roman* font called *Times*. It's considered best practice to use both with *Times New Roman* as the preferred alternative. |
| Trebuchet MS    | sans-serif   | You should be careful with using this font — it isn't widely available on mobile OSes. |
| Verdana         | sans-serif   |                                                              |

**Note**: Among various resources, the [cssfontstack.com](http://www.cssfontstack.com/) website maintains a list of web safe fonts available on Windows and macOS operating systems, which can help you make your decision about what you consider safe for your usage.

**Note**: There is a way to download a custom font along with a webpage, to allow you to customize your font usage in any way you want: **web fonts**. This is a little bit more complex, and we will be discussing this in a separate article later on in the module.

#### Default fonts

CSS defines five generic names for fonts: `serif`, `sans-serif`, `monospace`, `cursive` and `fantasy`. Those are very generic and the exact font face used when using those generic names is up to each browser and can vary for each operating system they are running on. It represents a *worst case scenario* where the browser will try to do its best to provide at least a font that looks appropriate. `serif`, `sans-serif` and `monospace` are quite predictable and should provide something reasonable. On the other hand, `cursive` and `fantasy` are less predictable and we recommend using them very carefully, testing as you go.

The five names are defined as follows:

| Term         | Definition                                                   |
| :----------- | :----------------------------------------------------------- |
| `serif`      | Fonts that have serifs (the flourishes and other small details you see at the ends of the strokes in some typefaces) |
| `sans-serif` | Fonts that don't have serifs.                                |
| `monospace`  | Fonts where every character has the same width, typically used in code listings. |
| `cursive`    | Fonts that are intended to emulate handwriting, with flowing, connected strokes. |
| `fantasy`    | Fonts that are intended to be decorative.                    |

#### Font stacks

Since you can't guarantee the availability of the fonts you want to use on your webpages (even a web font *could* fail for some reason), you can supply a **font stack** so that the browser has multiple fonts it can choose from. This simply involves a `font-family` value consisting of multiple font names separated by commas, e.g.

```css
p {
  font-family: "Trebuchet MS", Verdana, sans-serif;
}
```

In such a case, the browser starts at the beginning of the list and looks to see if that font is available on the machine. If it is, it applies that font to the selected elements. If not, it moves on to the next font, and so on.

It is a good idea to provide a suitable generic font name at the end of the stack so that if none of the listed fonts are available, the browser can at least provide something approximately suitable. To emphasis this point, paragraphs are given the browser's default serif font if no other option is available — which is usually Times New Roman — this is no good for a sans-serif font!

**Note**: Fonts names that have more than one word — like `Trebuchet MS` — need to be surrounded by quotes, for example `"Trebuchet MS"`.

#### A font-family example

Let's add to our previous example, giving the paragraphs a sans-serif font:

```css
p {
  color: red;
  font-family: Helvetica, Arial, sans-serif;
}
```

This gives us the following result:

### Font size



In our previous module's [CSS values and units](https://developer.mozilla.org/en-US/docs/Learn/CSS/Introduction_to_CSS/Values_and_units) article, we reviewed [length and size units](https://developer.mozilla.org/en-US/Learn/CSS/Introduction_to_CSS/Values_and_units#Length_and_size). Font size (set with the [`font-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-size) property) can take values measured in most of these units (and others, such as [percentages](https://developer.mozilla.org/en-US/Learn/CSS/Introduction_to_CSS/Values_and_units#Percentages)), however the most common units you'll use to size text are:

- `px` (pixels): The number of pixels high you want the text to be. This is an absolute unit — it results in the same final computed value for the font on the page in pretty much any situation.
- `em`s: 1em is equal to the font size set on the parent element of the current element we are styling (more specifically, the width of a capital letter M contained inside the parent element.) This can become tricky to work out if you have a lot of nested elements with different font sizes set, but it is doable, as you'll see below. Why bother? It is quite natural once you get used to it, and you can use `em`s to size everything, not just text. You can have an entire website sized using ems, which makes maintenance easy.
- `rem`s: These work just like `em`s, except that 1`rem` is equal to the font size set on the root element of the document (i.e. `<html>`), not the parent element. This makes doing the maths to work out your font sizes much easier, but unfortunately `rem`s are not supported in Internet Explorer 8 and below. If you need to support older browsers with your project, you can either stick to using `em`s or `px`, or use a [polyfill](https://developer.mozilla.org/en-US/docs/Glossary/polyfill) such as [REM-unit-polyfill](https://github.com/chuckcarpenter/REM-unit-polyfill). 

The `font-size` of an element is inherited from that element's parent element. This all starts with the root element of the entire document — `<html>` — the `font-size` of which is set to 16px as standard across browsers. Any paragraph (or other element that doesn't have a different size set by the browser) inside the root element will have a final size of 16px. Other elements may have different default sizes, for example an `<h1>` element has a size of 2ems set by default, so will have a final size of 32px.

Things become more tricky when you start altering the font size of nested elements. For example, if you had an `<article>` element in your page, and set its font-size to `1.5em`s (which would compute to 24px final size), and then wanted the paragraphs inside the `<article>` elements to have a computed font size of 20px, what em value would you use?

```html
<!-- document base font-size is 16px -->
<article> <!-- If my font-size is 1.5em -->
  <p>My paragraph</p> <!-- How do I compute to 20px font-size? -->
</article>
```

You would need to set its em value to 20/24, or `0.83333333em`s. The maths can be complicated, so you need to be careful about how you style things. It is best to use rems where you can, to keep things simple, and avoid setting the font-size of container elements where possible.

#### A simple sizing example

When sizing your text, it is usually a good idea to set the base `font-size` of the document to 10px, so that then the maths is a lot easier to work out — required (r)em values are then the pixel font size divided by 10, not 16. After doing that, you can easily size the different types of text in your document to what you want. It is a good idea to list all your `font-size` rulesets in a designated area in your stylesheet, so they are easy to find.

Our new result is like so:

```css
html {
  font-size: 10px;
}

h1 {
  font-size: 2.6rem;
}

p {
  font-size: 1.4rem;
  color: red;
  font-family: Helvetica, Arial, sans-serif;
}
```

### Font style, font weight, text transform, and text decoration



CSS provides four common properties to alter the visual weight/emphasis of text:

- `font-style`

  : Used to turn italic text on and off. Possible values are as follows (you'll rarely use this, unless you want to turn some italic styling off for some reason):

  - `normal`: Sets the text to the normal font (turns existing italics off.)
  - `italic`: Sets the text to use the *italic version of the font* if available; if not available, it will simulate italics with oblique instead.
  - `oblique`: Sets the text to use a simulated version of an italic font, created by slanting the normal version.

- `font-weight`: Sets how bold the text is. This has many values available in case you have many font variants available (such as -light, -normal,-bold,-extrabold,-black, etc.), but realistically you'll rarely use any of them except for normal and bold:

  - `normal`, `bold`: Normal and **bold** font weight
  - `lighter`, `bolder`: Sets the current element's boldness to be one step lighter or heavier than its parent element's boldness.
  - `100`–`900`: Numeric boldness values that provide finer grained control than the above keywords, if needed. 

- `text-transform`: Allows you to set your font to be transformed. Values include:

  - `none`: Prevents any transformation.
  - `uppercase`: Transforms ALL TEXT TO CAPITALS.
  - `lowercase`: Transforms all text to lower case.
  - `capitalize`: Transforms all words to Have The First Letter Capitalized.
  - `full-width`: Transforms all glyphs to be written inside a fixed-width square, similar to a monospace font, allowing aligning of e.g. latin characters along with asian language glyphs (like Chinese, Japanese, Korean.)

- `text-decoration`: Sets/unsets text decorations on fonts (you'll mainly use this to unset the default underline on links when styling them.) Available values are:

  - `none`: Unsets any text decorations already present.
  - `underline`: Underlines the text.
  - `overline`: Gives the text an overline.
  - `line-through`: Puts a strikethrough over the text.

  You should note that `text-decoration` can accept multiple values at once, if you want to add multiple decorations simultaneously, for example `text-decoration: underline overline`. Also note that `text-decoration` is a shorthand property for `text-decoration-line`, `text-decoration-style`, and `text-decoration-color`. You can use combinations of these property values to create interesting effects, for example `text-decoration: line-through red wavy`.

Let's look at adding a couple of these properties to our example:

Our new result is like so:

```css
html {
  font-size: 10px;
}

h1 {
  font-size: 2.6rem;
  text-transform: capitalize;
}

h1 + p {
  font-weight: bold;
}

p {
  font-size: 1.4rem;
  color: red;
  font-family: Helvetica, Arial, sans-serif;
}
```

### Text drop shadows



You can apply drop shadows to your text using the [`text-shadow`](https://developer.mozilla.org/en-US/docs/Web/CSS/text-shadow) property. This takes up to four values, as shown in the example below:

```css
text-shadow: 4px 4px 5px red;
```

The four properties are as follows:

1. The horizontal offset of the shadow from the original text — this can take most available CSS [length and size units](https://developer.mozilla.org/en-US/Learn/CSS/Introduction_to_CSS/Values_and_units#Length_and_size), but you'll most commonly use px. This value has to be included.
2. The vertical offset of the shadow from the original text; behaves basically just like the horizontal offset, except that it moves the shadow up/down, not left/right. This value has to be included.
3. The blur radius — a higher value means the shadow is dispersed more widely. If this value is not included, it defaults to 0, which means no blur. This can take most available CSS [length and size units](https://developer.mozilla.org/en-US/Learn/CSS/Introduction_to_CSS/Values_and_units#Length_and_size).
4. The base color of the shadow, which can take any [CSS color unit](https://developer.mozilla.org/en-US/Learn/CSS/Introduction_to_CSS/Values_and_units#Colors). If not included, it defaults to `black`.

**Note**: Positive offset values move the shadow right and down, but you can also use negative offset values to move the shadow left and up, for example `-1px -1px`.

#### Multiple shadows

You can apply multiple shadows to the same text by including multiple shadow values separated by commas, for example:

```css
text-shadow: -1px -1px 1px #aaa,
             0px 4px 1px rgba(0,0,0,0.5),
             4px 4px 5px rgba(0,0,0,0.7),
             0px 0px 7px rgba(0,0,0,0.4);
```

If we applied this to the `<h1>` element in our Tommy the cat example, we'd end up with this:

**Note**: You can see more interesting examples of `text-shadow` usage in the Sitepoint article [Moonlighting with CSS text-shadow](http://www.sitepoint.com/moonlighting-css-text-shadow/).

## Text layout

With basic font properties out the way, let's now have a look at properties we can use to affect text layout.

### Text alignment



The [`text-align`](https://developer.mozilla.org/en-US/docs/Web/CSS/text-align) property is used to control how text is aligned within its containing content box. The available values are as follows, and work in pretty much the same way as they do in a regular word processor application:

- `left`: Left justifies the text.
- `right`: Right justifies the text.
- `center`: Centers the text.
- `justify`: Makes the text spread out, varying the gaps in between the words so that all lines of text are the same width. You need to use this carefully — it can look terrible, especially when applied to a paragraph with lots of long words in it. If you are going to use this, you should also think about using something else along with it, such as `hyphens`, to break some of the longer words across lines.

If we applied `text-align: center;` to the `<h1>` in our example, we'd end up with this:

### Line height



The [`line-height`](https://developer.mozilla.org/en-US/docs/Web/CSS/line-height) property sets the height of each line of text — this can take most [length and size units](https://developer.mozilla.org/en-US/Learn/CSS/Introduction_to_CSS/Values_and_units#Length_and_size), but can also take a unitless value, which acts as a multiplier and is generally considered the best option — the [`font-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-size) is multiplied to get the `line-height`. Body text generally looks nicer and is easier to read when the lines are spaced apart; the recommended line height is around 1.5–2 (double spaced.) So to set our lines of text to 1.5 times the height of the font, you'd use this:

```css
line-height: 1.5;
```

Applying this to the `<p>` elements in our example would give us this result:

### Letter and word spacing



The [`letter-spacing`](https://developer.mozilla.org/en-US/docs/Web/CSS/letter-spacing) and [`word-spacing`](https://developer.mozilla.org/en-US/docs/Web/CSS/word-spacing) properties allow you to set the spacing between letters and words in your text. You won't use these very often, but might find a use for them to get a certain look, or to improve the legibility of a particularly dense font. They can take most [length and size units](https://developer.mozilla.org/en-US/Learn/CSS/Introduction_to_CSS/Values_and_units#Length_and_size).

So as an example, if we applied the following to the first line of the `<p>` elements in our example:

```css
p::first-line {
  letter-spacing: 2px;
  word-spacing: 4px;
}
```

We'd get the following:

### Other properties worth looking at



The above properties give you an idea of how to start styling text on a webpage, but there are many more properties you could use. We just wanted to cover the most important ones here. Once you've become used to using the above, you should also explore the following:

Font styles:

- [`font-variant`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant): Switch between small caps and normal font alternatives.
- [`font-kerning`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-kerning): Switch font kerning options on and off.
- [`font-feature-settings`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-feature-settings): Switch various [OpenType](https://en.wikipedia.org/wiki/OpenType) font features on and off.
- [`font-variant-alternates`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant-alternates): Control the use of alternate glyphs for a given font-face.
- [`font-variant-caps`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant-caps): Control the use of alternate capital glyphs.
- [`font-variant-east-asian`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant-east-asian): Control the usage of alternate glyphs for East Asian scripts, like Japanese and Chinese.
- [`font-variant-ligatures`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant-ligatures): Control which ligatures and contextual forms are used in text.
- [`font-variant-numeric`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant-numeric): Control the usage of alternate glyphs for numbers, fractions, and ordinal markers.
- [`font-variant-position`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant-position): Control the usage of alternate glyphs of smaller sizes positioned as superscript or subscript.
- [`font-size-adjust`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-size-adjust): Adjust the visual size of the font independently of its actual font size.
- [`font-stretch`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-stretch): Switch between possible alternative stretched versions of a given font.
- [`text-underline-position`](https://developer.mozilla.org/en-US/docs/Web/CSS/text-underline-position): Specify the position of underlines set using the `text-decoration-line` property `underline` value.
- [`text-rendering`](https://developer.mozilla.org/en-US/docs/Web/CSS/text-rendering): Try to perform some text rendering optimization.

Text layout styles

- [`text-indent`](https://developer.mozilla.org/en-US/docs/Web/CSS/text-indent): Specify how much horizontal space should be left before the beginning of the first line of the text content.
- [`text-overflow`](https://developer.mozilla.org/en-US/docs/Web/CSS/text-overflow): Define how overflowed content that is not displayed is signaled to users.
- [`white-space`](https://developer.mozilla.org/en-US/docs/Web/CSS/white-space): Define how whitespace and associated line breaks inside the element are handled.
- [`word-break`](https://developer.mozilla.org/en-US/docs/Web/CSS/word-break): Specify whether to break lines within words.
- [`direction`](https://developer.mozilla.org/en-US/docs/Web/CSS/direction): Define the text direction (This depends on the language and usually it's better to let HTML handle that part as it is tied to the text content.)
- [`hyphens`](https://developer.mozilla.org/en-US/docs/Web/CSS/hyphens): Switch on and off hyphenation for supported languages.
- [`line-break`](https://developer.mozilla.org/en-US/docs/Web/CSS/line-break): Relax or strengthen line breaking for Asian languages.
- [`text-align-last`](https://developer.mozilla.org/en-US/docs/Web/CSS/text-align-last): Define how the last line of a block or a line, right before a forced line break, is aligned.
- [`text-orientation`](https://developer.mozilla.org/en-US/docs/Web/CSS/text-orientation): Define the orientation of the text in a line.
- [`overflow-wrap`](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-wrap): Specify whether or not the browser may break lines within words in order to prevent overflow.
- [`writing-mode`](https://developer.mozilla.org/en-US/docs/Web/CSS/writing-mode): Define whether lines of text are laid out horizontally or vertically and the direction in which subsequent lines flow.

## Font shorthand

Many font properties can also be set through the shorthand property [`font`](https://developer.mozilla.org/en-US/docs/Web/CSS/font). These are written in the following order: [`font-style`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-style), [`font-variant`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant), [`font-weight`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-weight), [`font-stretch`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-stretch), [`font-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-size), [`line-height`](https://developer.mozilla.org/en-US/docs/Web/CSS/line-height), and [`font-family`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-family).

Among all those properties, only `font-size` and `font-family` are required when using the `font` shorthand property.

A forward slash has to be put in between the [`font-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-size) and [`line-height`](https://developer.mozilla.org/en-US/docs/Web/CSS/line-height) properties.

A full example would look like this:

```css
font: italic normal bold normal 3em/1.5 Helvetica, Arial, sans-serif;
```

## Active learning: Playing with styling text

In this active learning session, we don't have any specific exercises for you to do: we'd just like you to have a good play with some font/text layout properties, and see what you can produce! You can either do this using offline HTML/CSS files, or enter your code into the live editable example below.

If you make a mistake, you can always reset it using the *Reset* button.

## Summary

We hoped you enjoyed playing with text in this article! The next article will give you all you need to know about styling HTML lists.

# Styling lists

[Lists](https://developer.mozilla.org/en-US/Learn/HTML/Introduction_to_HTML/HTML_text_fundamentals#Lists) behave like any other text for the most part, but there are some CSS properties specific to lists that you need to know about, and some best practices to consider. This article explains all.

## A simple list example

To begin with, let's look at a simple list example. Throughout this article, we'll look at unordered, ordered, and description lists — all have styling features that are similar, and some that are particular to their type of list. The unstyled example is [available on Github](http://mdn.github.io/learning-area/css/styling-text/styling-lists/unstyled-list.html) (check out the [source code](https://github.com/mdn/learning-area/blob/master/css/styling-text/styling-lists/unstyled-list.html) too.)

The HTML for our list example looks like so:

```html
<h2>Shopping (unordered) list</h2>
<p>Paragraph for reference, paragraph for reference, paragraph for reference, paragraph for reference, paragraph for reference, paragraph for reference.</p>
<ul>
  <li>Hummus</li>
  <li>Pita</li>
  <li>Green salad</li>
  <li>Halloumi</li>
</ul>
<h2>Recipe (ordered) list</h2>
<p>Paragraph for reference, paragraph for reference, paragraph for reference, paragraph for reference, paragraph for reference, paragraph for reference.</p>
<ol>
  <li>Toast pita, leave to cool, then slice down the edge.</li>
  <li>Fry the halloumi in a shallow, non-stick pan, until browned on both sides.</li>
  <li>Wash and chop the salad.</li>
  <li>Fill pita with salad, hummus, and fried halloumi.</li>
</ol>
<h2>Ingredient description list</h2>
<p>Paragraph for reference, paragraph for reference, paragraph for reference, paragraph for reference, paragraph for reference, paragraph for reference.</p>
<dl>
  <dt>Hummus</dt>
  <dd>A thick dip/sauce generally made from chick peas blended with tahini, lemon juice, salt, garlic, and other ingredients.</dd>
  <dt>Pita</dt>
  <dd>A soft, slightly leavened flatbread.</dd>
  <dt>Halloumi</dt>
  <dd>A semi-hard, unripened, brined cheese with a higher-than-usual melting point, usually made from goat/sheep milk.</dd>
  <dt>Green salad</dt>
  <dd>That green healthy stuff that many of us just use to garnish kebabs.</dd>
</dl>
```

If you go to the live example now and investigate the list elements using [browser developer tools](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/What_are_browser_developer_tools), you'll notice a couple of styling defaults:

- The `<ul>` and `<ol>` elements have a top and bottom margin of 16px (1em)  and a padding-left of 40px (2.5em.)

- The list items (`<li>` elements) have no set defaults for spacing.

- The `<dl>` element has a top and bottom margin of 16px (1em), but no padding set.

- The `<dd>` elements have margin-left of 40px (2.5em.)

- The `<p>` elements we've included for reference have a top and bottom margin of 16px (1em), the same as the different list types.


## Handling list spacing

When styling lists, you need to adjust their styles so they keep the same vertical spacing as their surrounding elements (such as paragraphs and images; sometimes called vertical rhythm), and the same horizontal spacing as each other (you can see the [finished styled example](http://mdn.github.io/learning-area/css/styling-text/styling-lists/) on Github, and [find the source code](https://github.com/mdn/learning-area/blob/master/css/styling-text/styling-lists/index.html) too.)

The CSS used for the text styling and spacing is as follows:

```css
/* General styles */

html {
  font-family: Helvetica, Arial, sans-serif;
  font-size: 10px;
}

h2 {
  font-size: 2rem;
}

ul,ol,dl,p {
  font-size: 1.5rem;
}

li, p {
  line-height: 1.5;
}

/* Description list styles */
dd, dt {
  line-height: 1.5;
}

dt {
  font-weight: bold;
}

dd {
  margin-bottom: 1.5rem;
}
```

- The first rule sets a sitewide font and a baseline font size of 10px. These are inherited by everything on the page.
- Rules 2 and 3 set relative font sizes for the headings, different list types (the children of the list elements inherit these), and paragraphs. This means that each paragraph and list will have the same font size and top and bottom spacing, helping to keep the vertical rhythm consistent.
- Rule 4 sets the same [`line-height`](https://developer.mozilla.org/en-US/docs/Web/CSS/line-height) on the paragraphs and list items — so the paragraphs and each individual list item will have the same spacing between lines. This will also help to keep the vertical rhythm consistent.
- Rules 5 and 6 apply to the description list — we set the same `line-height` on the description list terms and descriptions as we did with the paragraphs and list items. Again, consistency is good! We also make the description terms have bold font, so they visually stand out easier.

## List-specific styles

Now we've looked at general spacing techniques for lists, let's explore some list-specific properties. There are three properties you should know about to start with, which can be set on `<ul>` or `<ol>` elements:

- [`list-style-type`](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style-type): Sets the type of bullets to use for the list, for example, square or circle bullets for an unordered list, or numbers, letters or roman numerals for an ordered list.
- [`list-style-position`](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style-position): Sets whether the bullets appear inside the list items, or outside them before the start of each item.
- [`list-style-image`](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style-image): Allows you to use a custom image for the bullet, rather than a simple square or circle.

### Bullet styles



As mentioned above, the [`list-style-type`](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style-type) property allows you to set what type of bullet to use for the bullet points. In our example, we've set the ordered list to use uppercase roman numerals, with:

```css
ol {
  list-style-type: upper-roman;
}
```

This gives us the following look:

![an ordered list with the bullet points set to appear outside the list item text.](https://mdn.mozillademos.org/files/12962/outer-bullets.png)

You can find a lot more options by checking out the [`list-style-type`](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style-type) reference page.

### Bullet position



The [`list-style-position`](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style-position) property sets whether the bullets appear inside the list items, or outside them before the start of each item. The default value is `outside`, which causes the bullets to sit outside the list items, as seen above.

If you set the value to `inside`, the bullets will sit inside the lines:

```css
ol {
  list-style-type: upper-roman;
  list-style-position: inside;
}
```

![an ordered list with the bullet points set to appear inside the list item text.](https://mdn.mozillademos.org/files/12958/inner-bullets.png)

### Using a custom bullet image



The [`list-style-image`](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style-image) property allows you to use a custom image for your bullet. The syntax is pretty simple:

```css
ul {
  list-style-image: url(star.svg);
}
```

However, this property is a bit limited in terms of controlling the position, size, etc. of the bullets. You are better off using the [`background`](https://developer.mozilla.org/en-US/docs/Web/CSS/background) family of properties, which you'll learn a lot more about in the [Styling boxes](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_boxes) module. For now, here's a taster!

In our finished example, we have styled the unordered list like so (on top of what you've already seen above):

```css
ul {
  padding-left: 2rem;
  list-style-type: none;
}

ul li {
  padding-left: 2rem;
  background-image: url(star.svg);
  background-position: 0 0;
  background-size: 1.6rem 1.6rem;
  background-repeat: no-repeat;
}
```

Here we've done the following:

- Set the [`padding-left`](https://developer.mozilla.org/en-US/docs/Web/CSS/padding-left) of the `<ul>` down from the default `40px` to `20px`, then set the same amount on the list items. This is so that overall the list items are still lined up with the order list items and the description list descriptions, but the list items have some padding for the background images to sit inside. If we didn't do this, the background images would overlap with the list item text, which would look messy.
- Set the [`list-style-type`](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style-type) to `none`, so that no bullet appears by default. We're going to use [`background`](https://developer.mozilla.org/en-US/docs/Web/CSS/background) properties to handle the bullets instead.
- Inserted a bullet onto each unordered list item. The relevant properties are as follows:
  - [`background-image`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-image): This references the path to the image file you want to use as the bullet.
  - [`background-position`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-position): This defines where in the background of the selected element the image will appear — in this case we are saying `0 0`, which means the bullet will appear in the very top left of each list item.
  - [`background-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-size): This sets the size of the background image. We ideally want the bullets to be the same size as the list items (or very slightly smaller or larger). We are using a size of `1.6rem` (`16px`), which fits very nicely with the `20px` padding we've allowed for the bullet to sit inside — 16px plus 4px of space between the bullet and the list item text works well.
  - [`background-repeat`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-repeat): By default, background images repeat until they fill up the available background space. We only want one copy of the image inserted in each case, so we set this to a value of `no-repeat`.

This gives us the following result:

![an unordered list with the bullet points set as little star images](https://mdn.mozillademos.org/files/16226/list_formatting.png)

### list-style shorthand



The three properties mentioned above can all be set using a single shorthand property, [`list-style`](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style). For example, the following CSS:

```css
ul {
  list-style-type: square;
  list-style-image: url(example.png);
  list-style-position: inside;
}
```

Could be replaced by this:

```css
ul {
  list-style: square url(example.png) inside;
}
```

The values can be listed in any order, and you can use one, two or all three (the default values used for the properties that are not included are `disc`, `none`, and `outside`). If both a `type` and an `image` are specified, the type is used as a fallback if the image can't be loaded for some reason.

## Controlling list counting

Sometimes you might want to count differently on an ordered list — e.g. starting from a number other than 1, or counting backwards, or counting in steps of more than 1. HTML and CSS have some tools to help you here.

### start



The `start` attribute allows you to start the list counting from a number other than 1. The following example:

```html
<ol start="4">
  <li>Toast pita, leave to cool, then slice down the edge.</li>
  <li>Fry the halloumi in a shallow, non-stick pan, until browned on both sides.</li>
  <li>Wash and chop the salad.</li>
  <li>Fill pita with salad, hummus, and fried halloumi.</li>
</ol>
```

Gives you this output:

### reversed



The `reversed` attribute will start the list counting down instead of up. The following example:

```html
<ol start="4" reversed>
  <li>Toast pita, leave to cool, then slice down the edge.</li>
  <li>Fry the halloumi in a shallow, non-stick pan, until browned on both sides.</li>
  <li>Wash and chop the salad.</li>
  <li>Fill pita with salad, hummus, and fried halloumi.</li>
</ol>
```

Gives you this output:

**Note**: If there are more list items in a reversed list than the value of the `start` attribute, the count will continue to zero and then into negative values. 

### value



The `value` attribute allows you to set your list items to specific numerical values. The following example:

```html
<ol>
  <li value="2">Toast pita, leave to cool, then slice down the edge.</li>
  <li value="4">Fry the halloumi in a shallow, non-stick pan, until browned on both sides.</li>
  <li value="6">Wash and chop the salad.</li>
  <li value="8">Fill pita with salad, hummus, and fried halloumi.</li>
</ol>
```

Gives you this output:

**Note**: Even if you are using a non-number [`list-style-type`](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style-type), you still need to use the equivalent numerical values in the `value` attribute.

## Active learning: Styling a nested list

In this active learning session, we want you to take what you've learned above and have a go at styling a nested list. We've provided you with the HTML, and we want you to:

1. Give the unordered list square bullets.
2. Give the unordered list items and the ordered list items a line height of 1.5 of their font-size.
3. Give the ordered list lower alphabetical bullets.
4. Feel free to play with the list example as much as you like, experimenting with bullet types, spacing, or whatever else you can find.

If you make a mistake, you can always reset it using the *Reset* button. If you get really stuck, press the *Show solution* button to see a potential answer.

## See also

CSS counters provide advanced tools for customizing list counting and styling, but they are quite complex. We recommend looking into these if you want to stretch yourself. See:

- [`@counter-style`](https://developer.mozilla.org/en-US/docs/Web/CSS/@counter-style)
- [`counter-increment`](https://developer.mozilla.org/en-US/docs/Web/CSS/counter-increment)
- [`counter-reset`](https://developer.mozilla.org/en-US/docs/Web/CSS/counter-reset)

## Summary

Lists are relatively easy to get the hang of styling once you know a few associated basic principles and specific properties. In the next article we'll get on to link styling techniques.

# Styling links

When styling [links](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/Creating_hyperlinks), it is important to understand how to make use of pseudo-classes to style link states effectively, and how to style links for use in common varied interface features such as navigation menus and tabs. We'll look at all these topics in this article.

## Let's look at some links

We looked at how links are implemented in your HTML according to best practices in [Creating hyperlinks](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/Creating_hyperlinks). In this article we'll build on this knowledge, showing you best practices for styling links.

### Link states



The first thing to understand is the concept of link states — different states that links can exist in, which can be styled using different [pseudo-classes](https://developer.mozilla.org/en-US/Learn/CSS/Introduction_to_CSS/Selectors#Pseudo-classes):

- **Link (unvisited)**: The default state that a link resides in, when it isn't in any other state. This can be specifically styled using the [`:link`](https://developer.mozilla.org/en-US/docs/Web/CSS/:link) pseudo class.
- **Visited**: A link when it has already been visited (exists in the browser's history), styled using the [`:visited`](https://developer.mozilla.org/en-US/docs/Web/CSS/:visited) pseudo class.
- **Hover**: A link when it is being hovered over by a user's mouse pointer, styled using the [`:hover`](https://developer.mozilla.org/en-US/docs/Web/CSS/:hover) pseudo class.
- **Focus**: A link when it has been focused (for example moved to by a keyboard user using the Tab key or similar, or programmatically focused using [`HTMLElement.focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus)) — this is styled using the [`:focus`](https://developer.mozilla.org/en-US/docs/Web/CSS/:focus) pseudo class.
- **Active**: A link when it is being activated (e.g. clicked on), styled using the [`:active`](https://developer.mozilla.org/en-US/docs/Web/CSS/:active) pseudo class.

### Default styles



The following example illustrates what a link will behave like by default (the CSS is simply enlarging and centering the text to make it stand out more.)

```html
<p><a href="#">A simple link</a></p>

```

```css
p {
  font-size: 2rem;
  text-align: center;
}
```

**Note**: All the links in the examples in this page are fake links — a `#` (hash, or pound sign) is put in place of the real URL. This is because if the real links were included, clicking on them would break the examples (you'd end up with an error, or a page loaded in the embedded example that you couldn't get back from.) `#` just links to the current page.

You'll notice a few things as you explore the default styles:

- Links are underlined.
- Unvisited links are blue.
- Visited links are purple.
- Hovering a link makes the mouse pointer change to a little hand icon.
- Focused links have an outline around them — you should be able to focus on the links on this page with the keyboard by pressing the tab key (On Mac, you may need enable the *Full Keyboard Access: All controls* option by pressing Ctrl + F7 before this will work.)
- Active links are red (Try holding down the mouse button on the link as you click it.)

Interestingly enough, these default styles are nearly the same as they were back in the early days of browsers in the mid-1990s. This is because users know and have come to expect this behaviour — if links were styled differently, it would confuse a lot of people. This doesn't mean that you shouldn't style links at all, just that you should not stray too far from the expected behaviour. You should at least:

- Use underlining for links, but not for other things. If you don't want to underline links, at least highlight them in some other way.
- Make them react in some way when hovered/focused, and in a slightly different way when activated.

The default styles can be turned off/changed using the following CSS properties:

- [`color`](https://developer.mozilla.org/en-US/docs/Web/CSS/color) for the text color.
- [`cursor`](https://developer.mozilla.org/en-US/docs/Web/CSS/cursor) for the mouse pointer style — you shouldn't turn this off unless you've got a very good reason.
- [`outline`](https://developer.mozilla.org/en-US/docs/Web/CSS/outline) for the text outline (an outline is similar to a border, the only difference being that border takes up space in the box and an outline doesn't; it just sits over the top of the background). The outline is a useful accessibility aid, so think carefully before turning it off; you should at least double up the styles given to the link hover state on the focus state too.

**Note**: You are not just limited to the above properties to style your links — you are free to use any properties you like. Just try not to go too crazy!

### Styling some links



Now we've looked at the default states in some detail, let's look at a typical set of link styles.

To start off with, we'll write out our empty rulesets:

```css
a {

}


a:link {

}

a:visited {

}

a:focus {

}

a:hover {

}

a:active {

}
```

This order is important because the link styles build on one another, for example the styles in the first rule will apply to all the subsequent ones, and when a link is being activated, it is also being hovered over. If you put these in the wrong order, things won't work properly. To remember the order, you could try using a mnemonic like **L**o**V**e **F**ears **HA**te.

Now let's add some more information to get this styled properly:

```css
body {
  width: 300px;
  margin: 0 auto;
  font-size: 1.2rem;
  font-family: sans-serif;
}

p {
  line-height: 1.4;
}

a {
  outline: none;
  text-decoration: none;
  padding: 2px 1px 0;
}

a:link {
  color: #265301;
}

a:visited {
  color: #437A16;
}

a:focus {
  border-bottom: 1px solid;
  background: #BAE498;
}

a:hover {
  border-bottom: 1px solid;     
  background: #CDFEAA;
}

a:active {
  background: #265301;
  color: #CDFEAA;
}
```

We'll also provide some sample HTML to apply the CSS to:

```html
<p>There are several browsers available, such as <a href="#">Mozilla
Firefox</a>, <a href="#">Google Chrome</a>, and
<a href="#">Microsoft Edge</a>.</p>
```

Putting the two together gives us this result:

So what did we do here? This certainly looks different to the default styling, but it still provides a familiar enough experience for users to know what's going on:

- The first two rules are not that interesting to this discussion.

- The third rule uses the `a` selector to get rid of the default text underline and focus outline (which varies across browsers anyway), and adds a tiny amount of padding to each link — all of this will become clear later on.

- Next, we use the `a:link` and `a:visited` selectors to set a couple of color variations on unvisited and visited links, so they are distinct.

- The next two rules use `a:focus`  and `a:hover`  to set focused and hovered links to have different background colors, plus an underline to make the link stand out even more. Two points to note here are:

  - The underline has been created using [`border-bottom`](https://developer.mozilla.org/en-US/docs/Web/CSS/border-bottom), not [`text-decoration`](https://developer.mozilla.org/en-US/docs/Web/CSS/text-decoration) — some people prefer this because the former has better styling options than the latter, and is drawn a bit lower, so doesn't cut across the descenders of the word being underlined (e.g. the tails on g and y).
  - The [`border-bottom`](https://developer.mozilla.org/en-US/docs/Web/CSS/border-bottom) value has been set as `1px solid`, with no color specified. Doing this makes the border adopt the same color as the element's text, which is useful in cases like this where the text is a different color in each case.

- Finally, `a:active` is used to give the links an inverted color scheme while they are being activated, to make it clear something important is happening!

### Active learning: Style your own links



In this active learning session, we'd like you to take our empty set of rules and add your own declarations to make the links look really cool. Use your imagination, go wild. We are sure you can come up with something cooler and just as functional as our example above.

If you make a mistake, you can always reset it using the *Reset* button. If you get really stuck, press the *Show solution* button to insert the example we showed above.

## Including icons on links

A common practice is to include icons on links to provide more of an indicator as to what kind of content the link points to. Let's look at a really simple example that adds an icon to external links (links that lead to other sites.) Such an icon usually looks like a little arrow pointing out of a box — for this example, we'll use [this great example from icons8.com](https://icons8.com/web-app/741/external-link).

Let's look at some HTML and CSS that will give us the effect we want. First, some simple HTML to style:

```html
<p>For more information on the weather, visit our <a href="http://#">weather page</a>,
look at <a href="http://#">weather on Wikipedia</a>, or check
out <a href="http://#">weather on Extreme Science</a>.</p>
```

Next, the CSS:

```css
body {
  width: 300px;
  margin: 0 auto;
  font-family: sans-serif;
}

p {
  line-height: 1.4;
}

a {
  outline: none;
  text-decoration: none;
  padding: 2px 1px 0;
}

a:link {
  color: blue;
}

a:visited {
  color: purple;
}

a:focus, a:hover {
  border-bottom: 1px solid;
}

a:active {
  color: red;
}

a[href*="http"] {
  background: url('https://mdn.mozillademos.org/files/12982/external-link-52.png') no-repeat 100% 0;
  background-size: 16px 16px;
  padding-right: 19px;
}
```

So what's going on here? We'll skip over most of the CSS, as it's just the same information you've looked at before. The last rule however is interesting — here we are inserting a custom background image on external links in a similar manner to how we handled [custom bullets on list items](https://developer.mozilla.org/en-US/Learn/CSS/Styling_text/Styling_lists#Using_a_custom_bullet_image) in the last article — this time however we are using [`background`](https://developer.mozilla.org/en-US/docs/Web/CSS/background) shorthand instead of the individual properties. We set the path to the image we want to insert, specify `no-repeat` so we only get one copy inserted, and then specify the position as 100% of the way over to the right of the text content, and 0 pixels from the top.

We also use [`background-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-size) to specify the size we want the background image to be shown at — it is useful to have a larger icon and then resize it like this as needed for responsive web design purposes. This does however only work with IE 9 and later, so if you need to support those older browsers, you'll just have to resize the image and insert it as is.

Finally, we set some [`padding-right`](https://developer.mozilla.org/en-US/docs/Web/CSS/padding-right) on the links to make space for the background image to appear in, so we aren't overlapping it with the text.

A final word — how did we select just external links? Well, if you are writing your [HTML links](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML/Creating_hyperlinks) properly, you should only be using absolute URLs for external links — it is more efficient to use relative links to link to other parts of your own site. The text "http" should therefore only appear in external links, and we can select this with an [attribute selector](https://developer.mozilla.org/en-US/Learn/CSS/Introduction_to_CSS/Selectors#Attribute_selectors): `a[href*="http"]` selects `<a>` elements, but only if they have an `href` attribute with a value that contains "http" somewhere inside it.

So that's it — try revisiting the active learning section above and trying this new technique out!

**Note**: Don't worry if you are not familiar with [backgrounds](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_boxes) and [responsive web design](https://developer.mozilla.org/en-US/docs/Web/Apps/Progressive/Responsive/responsive_design_building_blocks) yet; these are explained in other places.

## Styling links as buttons

The tools you've explored so far in this article can also be used in other ways. For example, states like hover can be used to style many different elements, not just links — you might want to style the hover state of paragraphs, list items, or other things.

In addition, links are quite commonly styled to look and behave like buttons in certain circumstances — a website navigation menu is usually marked up as a list containing links, and this can be easily styled to look like a set of control buttons or tabs that provide the user with access to other parts of the site. Let's explore how.

First, some HTML:

```html
<ul>
  <li><a href="#">Home</a></li><li><a href="#">Pizza</a></li><li><a href="#">Music</a></li><li><a href="#">Wombats</a></li><li><a href="#">Finland</a></li>
</ul>
```

And now our CSS:

```css
body,html {
  margin: 0;
  font-family: sans-serif;
}

ul {
  padding: 0;
  width: 100%;
}

li {
  display: inline;
}

a {
  outline: none;
  text-decoration: none;
  display: inline-block;
  width: 19.5%;
  margin-right: 0.625%;
  text-align: center;
  line-height: 3;
  color: black;
}

li:last-child a {
  margin-right: 0;
}

a:link, a:visited, a:focus {
  background: yellow;
}

a:hover {     
  background: orange;
}

a:active {
  background: red;
  color: white;
}
```

This gives us the following result:

Let's explain what's going on here, focusing on the most interesting parts:

- Our second rule removes the default [`padding`](https://developer.mozilla.org/en-US/docs/Web/CSS/padding) from the `<ul>` element, and sets its width to span 100% of the outer container (the `<body>`, in this case).

- [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/li) elements are normally block by default (see [types of CSS boxes](https://developer.mozilla.org/en-US/Learn/CSS/Introduction_to_CSS/Box_model#Types_of_CSS_boxes) for a refresher), meaning that they will sit on their own lines. In this case, we are creating a horizontal list of links, so in the third rule we set the [`display`](https://developer.mozilla.org/en-US/docs/Web/CSS/display) property to inline, which causes the list items to sit on the same line as one another — they now behave like inline elements.

- The fourth rule — which styles the

   

   

  element — is the most complicated here; let's go through it step by step:

  - As in previous examples, we start by turning off the default [`text-decoration`](https://developer.mozilla.org/en-US/docs/Web/CSS/text-decoration) and [`outline`](https://developer.mozilla.org/en-US/docs/Web/CSS/outline) — we don't want those spoiling our look.
  - Next, we set the [`display`](https://developer.mozilla.org/en-US/docs/Web/CSS/display) to `inline-block` — [`<a>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a) elements are inline by default and, while we don't want them to spill onto their own lines like a value of `block` would achieve, we do want to be able to size them. `inline-block` allows us to do this.
  - Now onto the sizing! We want to fill up the whole width of the [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ul), leave a little margin between each button (but not a gap at the right hand edge), and we have 5 buttons to accommodate that should all be the same size. To do this, we set the [`width`](https://developer.mozilla.org/en-US/docs/Web/CSS/width) to 19.5%, and the [`margin-right`](https://developer.mozilla.org/en-US/docs/Web/CSS/margin-right) to 0.625%. You'll notice that all this width adds up to 100.625%, which would make the last button overflow the `<ul>` and fall down to the next line. However, we take it back down to 100% using the next rule, which selects only the last `<a>` in the list, and removes the margin from it. Done!
  - The last three declarations are pretty simple and are mainly just for cosmetic purposes. We center the text inside each link, set the [`line-height`](https://developer.mozilla.org/en-US/docs/Web/CSS/line-height) to 3 to give the buttons some height (which also has the advantage of centering the text vertically), and set the text color to black.

**Note**: You may have noticed that the list items in the HTML are all put on the same line as each other — this is done because spaces/line breaks in between inline block elements create spaces on the page, just like the spaces in between words, and such spaces would break our horizontal navigation menu layout. So we've removed the spaces. You can find more information about this problem (and solutions) at [Fighting the space between inline block elements](https://css-tricks.com/fighting-the-space-between-inline-block-elements/).

## Summary

We hope this article has provided you with all you'll need to know about links — for now! The final article in our Styling text module details how to use custom fonts on your websites, or web fonts as they are better known.

# Web fonts

In the first article of the module, we explored the basic CSS features available for styling fonts and text. In this article we will go further, exploring web fonts in detail — these allow you to download custom fonts along with your web page, to allow for more varied, custom text styling.

## Font families recap

As we looked at in [Fundamental text and font styling](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text/Fundamentals), the fonts applied to your HTML can be controlled using the [`font-family`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-family) property. This takes one or more font family names, and the browser travels down the list until it finds a font it has available on the system it is running on:

```css
p {
  font-family: Helvetica, "Trebuchet MS", Verdana, sans-serif;
}
```

This system works well, but traditionally web developers' font choices were limited. There are only a handful of fonts that you can guarantee to be available across all common systems — the so-called [Web-safe fonts](https://developer.mozilla.org/en-US/Learn/CSS/Styling_text/Fundamentals#Web_safe_fonts). You can use the font stack to specify preferable fonts, followed by web-safe alternatives, followed by the default system font, but this adds overhead in terms of testing to make sure that your designs look OK with each font, etc.

## Web fonts

But there is an alternative, which works very well, right back to IE version 6. Web fonts are a CSS feature that allows you to specify font files to be downloaded along with your website as it is accessed, meaning that any browser that supports web fonts can have exactly the fonts you specify available to it. Amazing! The syntax required looks something like this:

First of all, you have a [`@font-face`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face) block at the start of the CSS, which specifies the font file(s) to download:

```css
@font-face {
  font-family: "myFont";
  src: url("myFont.woff");
}
```

Below this you can then use the font family name specified inside @font-face to apply your custom font to anything you like, as normal:

```css
html {
  font-family: "myFont", "Bitstream Vera Serif", serif;
}
```

The syntax does get a bit more complex than this; we'll go into more detail below.

There are two important things to bear in mind about web fonts:

1. Browsers support different font formats, so you'll need multiple font formats for decent cross-browser support. For example, most modern browsers support WOFF/WOFF2 (Web Open Font Format versions 1 and 2), the most efficient format around, but older versions of IE only support EOT (Embedded Open Type) fonts, and you might need to include an SVG version of the font to support older versions of iPhone and Android browsers. We'll show you below how to generate the required code.
2. Fonts generally aren't free to use. You have to pay for them, and/or follow other license conditions such as crediting the font creator in the code (or on your site). You shouldn't steal fonts and use them without giving proper credit.

**Note**: Web fonts as a technology have been supported in Internet Explorer since version 4!

## Active learning: A web font example

With this in mind, let's build up a basic web font example from first principles. It is difficult to demonstrate this using an embedded live example, so instead, we would like you to follow the steps detailed in the below sections, to get an idea of the process.

You should use the [web-font-start.html](https://github.com/mdn/learning-area/blob/master/css/styling-text/web-fonts/web-font-start.html) and [web-font-start.css](https://github.com/mdn/learning-area/blob/master/css/styling-text/web-fonts/web-font-start.css) files as a starting point to add your code to (see the [live example](http://mdn.github.io/learning-area/css/styling-text/web-fonts/web-font-start.html)). Make a copy of these files in a new directory on your computer now. In the `web-font-start.css` file, you'll find some minimal CSS to deal with the basic layout and typesetting of the example.

### Finding fonts



For this example, we'll use two web fonts, one for the headings, and one for the body text. To start with, we need to find the font files that contain the fonts. Fonts are created by font foundries and are stored in different file formats. There are generally three types of sites where you can obtain fonts:

- A free font distributor: This is a site that makes free fonts available for download (there may still be some license conditions, such as crediting the font creator). Examples include [Font Squirrel](https://www.fontsquirrel.com/), [dafont](http://www.dafont.com/), and [Everything Fonts](https://everythingfonts.com/).
- A paid font distributor: This is a site that makes fonts available for a charge, such as [fonts.com](http://www.fonts.com/) or [myfonts.com](http://www.myfonts.com/). You can also buy fonts directly from font foundries, for example [Linotype](https://www.linotype.com/), [Monotype](http://www.monotype.com/), or [Exljbris](http://www.exljbris.com/).
- An online font service: This is a site that stores and serves the fonts for you, making the whole process easier. See the [Using an online font service](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text/Web_fonts#Using_an_online_font_service) section for more details.

Let's find some fonts! Go to [Font Squirrel](https://www.fontsquirrel.com/) and choose two fonts — a nice interesting font for the headings (maybe a nice display or slab serif font), and slightly less flashy and more readable font for the paragraphs. When you find each font, press on the download button, and save the file inside the same directory as the HTML and CSS files you saved earlier. It doesn't matter whether they are TTF (True Type Fonts) or OTF (Open Type Fonts).

In each case, unzip the font package (Web fonts are usually distributed in ZIP files containing the font file(s) and licensing information). You may find multiple font files in the package — some fonts are distributed as a family with different variants available, for example thin, medium, bold, italic, thin italic, etc. For this example, we just want you to concern yourself with a single font file for each choice.

**Note**: Under the "Find fonts" section in the right-hand column, you can click on the different tags and classifications to filter the displayed choices.

### Generating the required code



Now you'll need to generate the required code (and font formats). For each font, follow these steps:

1. Make sure you have satisfied any licensing requirement, if you are going to use this in a commercial and/or Web project.
2. Go to the Fontsquirrel [Webfont Generator](https://www.fontsquirrel.com/tools/webfont-generator).
3. Upload your two font files using the *Upload Fonts* button.
4. Check the checkbox labeled "Yes, the fonts I'm uploading are legally eligible for web embedding."
5. Click *Download your kit*.

After the generator has finished processing, you should get a ZIP file to download — save it in the same directory as your HTML and CSS.

### Implementing the code in your demo



At this point, unzip the webfont kit you just generated. Inside the unzipped directory you'll see three useful items:

- Multiple versions of each font: (for example `.ttf`, `.woff`, `.woff2`, etc.; the exact fonts provided will be updated over time as browser support requirements change). As mentioned above, multiple fonts are needed for cross browser support — this is Fontsquirrel's way of making sure you've got everything you need.
- A demo HTML file for each font — load these in your browser to see what the font will look like in different usage contexts.
- A `stylesheet.css` file, which contains the generated @font-face code you'll need.

To implement these fonts in your demo, follow these steps:

1. Rename the unzipped directory to something easy and simple, like `fonts`.

2. Open up the `stylesheet.css` file and copy both the `@font-face` blocks contained inside into your `web-font-start.css` file — you need to put them at the very top, before any of your CSS, as the fonts need to be imported before you can use them on your site.

3. Each of the `url()` functions points to a font file that we want to import into our CSS — we need to make sure the paths to the files are correct, so add `fonts/` to the start of each path (adjust as necessary).

4. Now you can use these fonts in your font stacks, just like any web safe or default system font. For example:

   ```css
   font-family: 'zantrokeregular', serif;
   ```

You should end up with a demo page with some nice fonts implemented on them. Because different fonts are created at different sizes, you may have to adjust the size, spacing, etc., to sort out the look and feel.

**Note**: If you have any problems getting this to work, feel free to compare your version to our finished files — see [web-font-finished.html](https://github.com/mdn/learning-area/blob/master/css/styling-text/web-fonts/web-font-finished.html) and [web-font-finished.css](https://github.com/mdn/learning-area/blob/master/css/styling-text/web-fonts/web-font-finished.css) ([run the finished example live](http://mdn.github.io/learning-area/css/styling-text/web-fonts/web-font-finished.html)).

## Using an online font service

Online font services generally store and serve fonts for you, so you don't have to worry about writing the `@font-face` code, and generally just need to insert a simple line or two of code into your site to make everything work. Examples include [Adobe Fonts](https://fonts.adobe.com/) and [Cloud.typography](http://www.typography.com/cloud/welcome/). Most of these services are subscription-based, with the notable exception of [Google Fonts](https://www.google.com/fonts), a useful free service, especially for rapid testing work and writing demos.

Most of these services are easy to use, so we won't cover them in great detail. Let's have a quick look at Google fonts, so you can get the idea. Again, use copies of `web-font-start.html` and `web-font-start.css` as your starting point.

1. Go to [Google Fonts](https://www.google.com/fonts).
2. Use the filters on the left-hand side to display the kinds of fonts you want to choose and choose a couple of fonts you like.
3. To select a font family, press the ⊕ button alongside it.
4. When you've chosen the font families, press the *[Number] Families Selected* bar at the bottom of the page.
5. In the resulting screen, you first need to copy the line of HTML code shown and paste it into the head of your HTML file. Put it above the existing [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link) element, so that the font is imported before you try to use it in your CSS.
6. You then need to copy the CSS declarations listed into your CSS as appropriate, to apply the custom fonts to your HTML.

**Note**: You can find a completed version at [google-font.html](https://github.com/mdn/learning-area/blob/master/css/styling-text/web-fonts/google-font.html) and [google-font.css](https://github.com/mdn/learning-area/blob/master/css/styling-text/web-fonts/google-font.css), if you need to check your work against ours ([see it live](http://mdn.github.io/learning-area/css/styling-text/web-fonts/google-font.html)).

## @font-face in more detail

Let's explore that `@font-face` syntax generated for you by fontsquirrel. This is what one of the blocks looks like:

```css
@font-face {
  font-family: 'ciclefina';
  src: url('fonts/cicle_fina-webfont.eot');
  src: url('fonts/cicle_fina-webfont.eot?#iefix') format('embedded-opentype'),
         url('fonts/cicle_fina-webfont.woff2') format('woff2'),
         url('fonts/cicle_fina-webfont.woff') format('woff'),
         url('fonts/cicle_fina-webfont.ttf') format('truetype'),
         url('fonts/cicle_fina-webfont.svg#ciclefina') format('svg');
  font-weight: normal;
  font-style: normal;
}
```

This is referred to as "bulletproof @font-face syntax", after a post by Paul Irish from early on when `@font-face` started to get popular ([Bulletproof @font-face Syntax](http://www.paulirish.com/2009/bulletproof-font-face-implementation-syntax/)). Let's go through it to see what it does:

- `font-family`: This line specifies the name you want to refer to the font as. You can put this as anything you like, as long as you use it consistently throughout your CSS.
- `src`: These lines specify the paths to the font files to be imported into your CSS (the `url` part), and the format of each font file (the `format` part). The latter part in each case is optional, but it is useful to declare it because it allows browsers to find a font they can use faster. Multiple declarations can be listed, separated by commas — the browser will search through them and use the first one it can find that it understands — it is therefore best to put newer, better formats like WOFF2 earlier on, and older, not so good formats like TTF later on. The one exception to this is the EOT fonts — they are placed first to fix a couple of bugs in older versions of IE whereby it will try to use the first thing it finds, even if it can't actually use the font.
- [`font-weight`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-weight)/[`font-style`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-style): These lines specify what weight the font has, and whether it is italic or not. If you are importing multiple weights of the same font, you can specify what their weight/style is and then use different values of [`font-weight`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-weight)/[`font-style`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-style) to choose between them, rather than having to call all the different members of the font family different names. [@font-face tip: define font-weight and font-style to keep your CSS simple](http://www.456bereastreet.com/archive/201012/font-face_tip_define_font-weight_and_font-style_to_keep_your_css_simple/) by Roger Johansson shows what to do in more detail.

**Note**: You can also specify particular [`font-variant`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant) and [`font-stretch`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-stretch) values for your web fonts. In newer browsers, you can also specify a [`unicode-range`](https://developer.mozilla.org/en-US/docs/Web/CSS/unicode-range) value, which is a specific range of characters you want to use out of the web font — in supporting browsers, only the specified characters will be downloaded, saving unnecessary downloading. [Creating Custom Font Stacks with Unicode-Range](https://24ways.org/2011/creating-custom-font-stacks-with-unicode-range/) by Drew McLellan provides some useful ideas on how to make use of this.

## Summary

Now that you have worked through our articles on text styling fundamentals, it is time to test your comprehension with our assessment for the module, Typesetting a community school homepage.

# Typesetting a community school homepage

In this assessment we'll test your understanding of all the text styling techniques we've covered throughout this module by getting you to style the text for a community school's homepage. You might just have some fun along the way.

## Starting point

To get this assessment started, you should:

- Go and grab the [HTML](https://github.com/mdn/learning-area/blob/master/css/styling-text/typesetting-a-homepage-start/index.html) and [CSS](https://github.com/mdn/learning-area/blob/master/css/styling-text/typesetting-a-homepage-start/style.css) files for the exercise, and the provided [external link icon](https://github.com/mdn/learning-area/blob/master/css/styling-text/typesetting-a-homepage-start/external-link-52.png).
- Make a copy of them on your local computer.

**Note**: Alternatively, you could use a site like [JSBin](http://jsbin.com/) or [Thimble](https://thimble.mozilla.org/) to do your assessment. You could paste the HTML and fill in the CSS into one of these online editors, and use [this URL](http://mdn.github.io/learning-area/css/styling-text/typesetting-a-homepage-start/external-link-52.png) to point the background image. If the online editor you are using doesn't have a separate CSS panel, feel free to put it in a `` element in the head of the document.

## Project brief

You have been provided with some raw HTML for the homepage of an imaginary community college, plus some CSS that styles the page into a three column layout and provides some other rudimentary styling. You are to write your CSS additions below the comment at the bottom of the CSS file, to make sure it is easy to mark the bits you have done. Don't worry if some of the selectors are repetitious; we'll let you off in this instance.

Fonts:

- First of all, download a couple of free-to-use fonts. Because this is a college, the fonts should be chosen to give the page a fairly serious, formal, trustworthy feel — a serif site-wide font for the general text body, coupled with sans-serif or slab serif for the headings might be nice.
- Use a suitable service to generate bulletproof `@font-face` code for these two fonts.
- Apply your body font to the whole page, and your heading font to your headings.

General text styling:

- Give the page a site-wide `font-size` of `10px`.
- Give your headings and other element types appropriate font-sizes defined using a suitable relative unit.
- Give your body text a suitable `line-height`.
- Center your top level heading on the page.
- Give your headings a little bit of `letter-spacing` to make them not too too squashed, and allow the letters to breathe a bit.
- Give your body text some `letter-spacing` and `word-spacing`, as appropriate.
- Give the first paragraph after each heading in the `` a little bit of text-indentation, say 20px.

Links:

- Give the link, visited, focus, and hover states some colors that go with the color of the horizontal bars at the top and bottom of the page.
- Make it so that links are underlined by default, but when you hover or focus them, the underline disappears.
- Remove the default focus outline from ALL the links on the page.
- Give the active state a noticeably different styling so it stands out nicely, but make it still fit in with the overall page design.
- Make it so that external links have the external link icon inserted next to them.

Lists:

- Make sure the spacing of your lists and list items works well with the styling of the overall page. Each list item should have the same `line-height` as a paragraph line, and each list should have the same spacing at its top and bottom as you have between paragraphs.
- Give your list items a nice bullet, appropriate for the design of the page. It is up to you whether you choose a custom bullet image or something else.

Navigation menu:

- Style your navigation menu so that it has an appropriate look for the look and feel for the page.

## Hints and tips

- You don't need to edit the HTML in any way for this exercise.
- You don't necessarily have to make the nav menu look like buttons, but it needs to be a bit taller so that it doesn't look silly on the side of the page; also remember that you need to make this one a vertical nav menu.

## Example

The following screenshot shows an example of what the finished design could look like:

## Assessment

If you are following this assessment as part of an organized course, you should be able to give your work to your teacher/mentor for marking. If you are self-learning, then you can get the marking guide fairly easily by asking on the [discussion thread for this exercise](https://discourse.mozilla.org/t/typesetting-a-community-school-home-page-assessment/24683), or in the [#mdn](irc://irc.mozilla.org/mdn) IRC channel on [Mozilla IRC](https://wiki.mozilla.org/IRC). Try the exercise first — there is nothing to be gained by cheating!