# Backgrounds and borders

In this lesson we will take a look at some of the creative things you can do with CSS backgrounds and borders. From adding gradients, background images, and rounded corners, backgrounds and borders are the answer to a lot of styling questions in CSS. 

## Styling backgrounds in CSS

The CSS `background` property is a shorthand for a number of background longhand properties that we will meet in this lesson. If you discover a complex background property in a stylesheet, it might seem a little hard to understand as so many values can be passed in at once.

```css
.box { 
  background: linear-gradient(105deg, rgba(255,255,255,.2) 39%, rgba(51,56,57,1) 96%) center center / 400px 200px no-repeat, 
  url(big-star.png) center no-repeat, rebeccapurple; 
} 
```

We'll return to how the shorthand works later in the tutorial, but first let's have a look at the different things you can do with backgrounds in CSS, by looking at the individual background properties.

### Background colors



The [`background-color`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-color) property defines the background color on any element in CSS. The property accepts any valid `<color>`. A `background-color` extends underneath the content and padding box of the element.

In the example below we have used various color values to add a background color to the box, a heading, and a `<span>` element.

Play around with these, using any available `<color>` value.

```css
.box {
  background-color: #567895;
}

h2 {
  background-color: black;
  color: white;
}
span {
  background-color: rgba(255,255,255,.5);
}
```

### Background images



The [`background-image`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-image) property enables the display of an image in the background of an element. In the example below we have two boxes — one has a background image which is larger than the box, the other has a small image of a star.

This example demonstrates two things about background images. By default, the large image is not scaled down to fit the box, so we only see a small corner of it, whereas the small image is tiled to fill the box. In this case the actual image is just a single star.

```css
.a {
  background-image: url(balloons.jpg);
}

.b {
  background-image: url(star.png);
}
```

If you specify a background color in addition to a background image then the image displays on top of the color. Try adding a `background-color` property to the example above to see that in action.

#### Controlling background-repeat

The [`background-repeat`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-repeat) property is used to control the tiling behavior of images. The available values are:

- `no-repeat` — stop the background from repeating altogether.
- `repeat-x` — repeat horizontally.
- `repeat-y` — repeat vertrically.
- `repeat` — the default; repeat in both directions.

Try these values out in the example below. We have set the value to `no-repeat` so you will only see one star. Try out the different values — `repeat-x` and `repeat-y` — to see what their effects are.

#### Sizing the background image

In the example above, we have a large image that has ended up being cropped as it is larger than the element it is a background of. In this case we could use the [`background-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-size) property, which can take [length](https://developer.mozilla.org/en-US/docs/Web/CSS/length) or [percentage](https://developer.mozilla.org/en-US/docs/Web/CSS/percentage) values, to size the image to fit inside the background.

You can also use keywords:

- `cover` — the browser will make the image just large enough so that it completely covers the box area while still retaining its aspect ratio. In this case some of the image is likely to end up outside the box.
- `contain` — the browser will make the image the right size to fit inside the box. In this case you may end up with gaps on either side or on the top and bottom of the image, if the aspect ratio of the image is different to that of the box.

In the example below I have used the larger image from the example above, and used length units to size it inside the box. You can see this has distorted the image.

Try the following.

- Change the length units used to modify the size of the background.
- Remove the length units and see what happens when you use `background-size: cover` or `background-size: contain`.
- If your image is smaller than the box, you can change the value of `background-repeat` to repeat the image.

#### Positioning the background image

The [`background-position`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-position) property allows you to choose the position in which the background image appears on the box it is applied to. This uses a coordinate system in which the top-left-hand corner of the box is `(0,0)`, and the box is positioned along the horizontal (`x`) and vertical (`y`) axes.

**Note**: The default `background-position` value is `(0,0)`.

The most common `background-position` values take two individual values — a horizontal value followed by a vertical value.

You can use keywords such as `top` and `right` (look up the others on the [`background-image`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-image) page):

```css
.box { 
  background-image: url(star.png); 
  background-repeat: no-repeat; 
  background-position: top center; 
} 
```

And [Lengths](https://developer.mozilla.org/en-US/docs/Web/CSS/length), and [percentages](https://developer.mozilla.org/en-US/docs/Web/CSS/percentage):

```
.box { 
  background-image: url(star.png); 
  background-repeat: no-repeat; 
  background-position: 20px 10%; 
} 
```

You can also mix keyword values with lengths or percentages, for example:

```css
.box {
  background-image: url(star.png);
  background-repeat: no-repeat;
  background-position: top 20px;
}
```

Finally, you can also use a 4-value syntax in order to indicate a distance from certain edges of the box — the length unit in this case is an offset from the value that preceeds it. So in the CSS below we are positioning the background 20px from the top and 10px from the right:

```
.box { 
  background-image: url(star.png); 
  background-repeat: no-repeat; 
  background-position: top 20px right 10px; 
} 
```

Use the example below to play around with these values and move the star around inside the box.

**Note**: `background-position` is a shorthand for [`background-position-x`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-position-x) and [`background-position-y`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-position-y), which allow you to set the different axis position values individually.

### Gradient backgrounds



A gradient — when used for a background — acts just like an image and is also set by using the [`background-image`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-image) property.

You can read more about the different types of gradients and things you can do with them on the MDN page for the `<gradient>` data type. A fun way to play with gradients is to use one of the many CSS Gradient Generators available on the web, such as [this one](https://cssgradient.io/). You can create a gradient then copy and paste out the source code that generates it.

Try some different gradients in the example below. In the two boxes respectively we have a linear gradient which is stretched over the whole box, and a radial gradient with a set size, which therefore repeats.

```css
.a {
  background-image: linear-gradient(105deg, rgba(0,249,255,1) 39%, rgba(51,56,57,1) 96%);
}

.b {
  background-image: radial-gradient(circle, rgba(0,249,255,1) 39%, rgba(51,56,57,1) 96%);
  background-size: 100px 50px;
}
```

### Multiple background images



It is also possible to have multiple background images — you specify multiple `background-image` values in a single property value, separating each one with a comma.

When you do this you may end up with background images overlapping each other. The backgrounds will layer with the last listed background image at the bottom of the stack, and each previous image stacking on top of the one that follows it in the code.

**Note**: Gradients can be happily mixed with regular background images.

The other `background-*` properties can also have values comma-separated in the same way as `background-image`:

```css
background-image: url(image1.png), url(image2.png), url(image3.png), url(image1.png);
background-repeat: no-repeat, repeat-x, repeat;
background-position: 10px 20px,  top right;
```

Each value of the different properties will match up to the values in the same position in the other properties. Above, for example, `image1`'s `background-repeat` value will be `no-repeat`. However, what happens when different properties have different numbers of values? The answer is that the smaller numbers of values will cycle — in the above example there are four background images but only two `background-position` values. The first two position values will be applied to the first two images, then they will cycle back round again — `image3` will be given the first position value, and `image4` will be given the second position value.

Let's play. In the example below I have included two images. To demonstrate the stacking order, try switching which background image comes first in the list. Or play with the other properties to change the position, size, or repeat values.

### Background attachment



Another option we have available for backgrounds is specifying how they scroll when the content scrolls. This is controlled using the [`background-attachment`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-attachment) property, which can take the following values:

- `scroll`: Causes the element's background to scroll when the page is scrolled. If the element content is scrolled, the background does not move. In effect, the background is fixed to the same position on the page, so it scrolls as the page scrolls.
- `fixed`: Causes an element's background to be fixed to the viewport, so that it doesn't scroll when the page or element content is scrolled. It will always remain in the same position on the screen.
- `local`: This value was added later on (it is only supported in Internet Explorer 9+, whereas the others are supported in IE4+) because the `scroll` value is rather confusing and doesn't really do what you want in many cases. The `local` value fixes the background to the element it is set on, so when you scroll the element, the background scrolls with it.

The [`background-attachment`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-attachment) property only has an effect when there is content to scroll, so we've made a demo to demonstrate the differences between the three values — have a look at [background-attachment.html](http://mdn.github.io/learning-area/css/styling-boxes/backgrounds/background-attachment.html) (also [see the source code](https://github.com/mdn/learning-area/tree/master/css/styling-boxes/backgrounds) here).

### Using the background shorthand property



As I mentioned at the beginning of this lesson, you will often see backgrounds specified using the [`background`](https://developer.mozilla.org/en-US/docs/Web/CSS/background) property. This shorthand lets you set all of the different properties at once.

If using multiple backgrounds, you need to specify all of the properties for the first background, then add your next background after a comma. In the example below we have a gradient with a size and position, then an image background with `no-repeat` and a position, then a color.

There are a few rules that need to be followed when writing background image shorthand values, for example:

- A `background-color` may only be specified after the final comma.
- The value for `background-size` may only be included immediately after `background-position`, separated with the '/' character, like this: `center/80%`.

Take a look at the MDN page for [`background`](https://developer.mozilla.org/en-US/docs/Web/CSS/background) to see all of the considerations.

### Accessibility considerations with backgrounds



When placing text on top of a background image or color, you should take care that you have enough contrast for the text to be legible for your visitors. If specifying an image, and text will be placed on top of that image, you should also specify a `background-color` that will allow the text to be legible if the image does not load.

Screen readers cannot parse background images, therefore they should be purely decoration; any important content should be part of the HTML page and not contained in a background.

## Borders

When learning about the Box Model, we discovered how borders affect the size of our box. In this lesson we will look at how to use borders creatively. Typically when we add borders to an element with CSS we use a shorthand property that sets the color, width, and style of the border in one line of CSS. We can set a border for all four sides of a box with [`border`](https://developer.mozilla.org/en-US/docs/Web/CSS/border) .

```css
.box { 
  border: 1px solid black; 
} 
```

Or we can target one edge of the box, for example:

```css
.box { 
  border-top: 1px solid black; 
} 
```

The individual properties for these shorthands would be:

```css
.box { 
  border-width: 1px; 
  border-style: solid; 
  border-color: black; 
} 
```

And for the longhands:

```css
.box { 
  border-top-width: 1px; 
  border-top-style: solid; 
  border-top-color: black; 
} 
```

**Note**: These top, right, bottom, and left border properties also have mapped *logical* properties which relate to the writing mode of the document (e.g. left-to-right or right-to-left text, or top-to-bottom). We'll be exploring these in the next lesson, which covers [handling different text directions](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Handling_different_text_directions).

There are a variety of styles that you can use for borders. In the example below we have used a different border style for the four sides of my box. Play with the border style, width, and color to see how borders work.

```css
.box {
  background-color: #567895;
  border: 5px solid #0b385f;
  border-bottom-style: dashed;
  color: #fff;
}

h2 {
  border-top: 2px dotted rebeccapurple;
  border-bottom: 1em double rgb(24, 163, 78);
}
```

### Rounded corners



Rounding corners on a box is achieved by using the [`border-radius`](https://developer.mozilla.org/en-US/docs/Web/CSS/border-radius) property and associated longhands which relate to each corner of the box. Two lengths or percentages may be used as a value, the first value defining the horizontal radius, and the second the vertical radius. In a lot of cases you will only pass in one value, which will be used for both.

For example, to make all four corners of a box have a 10px radius:

```css
.box { 
  border-radius: 10px; 
} 
```

Or to make the top right corner have a horizontal radius of 1em, and a vertical radius of 10%:

```css
.box { 
  border-top-right-radius: 1em 10%; 
} 
```

We have set all four corners in the example below, and then changed the values for the top right corner to make it different. You can play with the values to change the corners. Take a look at the property page for [`border-radius`](https://developer.mozilla.org/en-US/docs/Web/CSS/border-radius) to see the available syntax options.

```css
.box {
  border: 10px solid rebeccapurple;
  border-radius: 1em;
  border-top-right-radius: 10% 30%;
}
```

## Playing with backgrounds and borders

To test out your new knowledge try to create the following using backgrounds and borders, using the example below as a starting point:

1. Give the box a 5px black solid border, with rounded corners of 10px.
2. Add a background image (use the URL `balloons.jpg`) and size it so that it covers the box.
3. Give the `h2` a semi-transparent black background color, and make the text white.

## Summary

We have covered quite a lot here, and you can see that there is quite a lot to adding a background or a border to a box. Do explore the different property pages if you want to find out more about any of the features we have discussed. Each page on MDN has more examples of usage for you to play with and enhance your knowledge.

In the next lesson we will find out how the Writing Mode of your document interacts with your CSS. What happens when text does not flow from left to right?

# Handling different text directions

Many of the properties and values that we have encountered so far in our CSS learning have been tied to the physical dimensions of our screen. We create borders on the top, right, bottom, and left of a box, for example. These physical dimensions map very neatly to content that is viewed horizontally, and by default the web tends to support left-to-right languages (e.g. English or French) better than right-to-left languages (such as Arabic).

In recent years however, CSS has evolved in order to better support different directionality of content, including right-to-left but also top-to-bottom content (such as Japanese) — these different directionalities are called **writing modes**. As you progress in your study and begin to work with layout, an understanding of writing modes will be very helpful to you, therefore we will introduce them now.

## What are writing modes?

A writing mode in CSS refers to whether the text is running horizontally or vertically. The [`writing-mode`](https://developer.mozilla.org/en-US/docs/Web/CSS/writing-mode) property lets us switch from one writing mode to another. You don't need to be working in a language which uses a vertical writing mode to want to do this — you could also change the writing mode of parts of your layout for creative purposes.

In the example below we have a heading displayed using `writing-mode: vertical-rl`. The text now runs vertically. Vertical text is common in graphic design, and can be a way to add a more interesting look and feel to your web design.

The three possible values for the `writing-mode` property are:

- `horizontal-tb`: Top-to-bottom block flow direction. Sentences run horizontally.
- `vertical-rl`: Right-to-left block flow direction. Sentences run vertically.
- `vertical-lr`: Left-to-right block flow direction. Sentences run vertically.

So the `writing-mode` property is in reality setting the direction in which block-level elements are displayed on the page — either from top-to-bottom, right-to-left, or left-to-right. This then dictates the direction text flows in sentences.

## Writing modes and block and inline layout

We have already discussed [block and inline layout](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/The_box_model#Block_and_inline_boxes), and the fact that some things display as block elements and others as inline elements. As we have seen described above, block and inline is tied to the writing mode of the document, and not the physical screen. Blocks are only displayed from the top to the bottom of the page if you are using a writing mode that displays text horizontally, such as English.

If we look at an example this will become clearer. In this next example I have two boxes that contain a heading and a paragraph. The first uses `writing-mode: horizontal-tb`, a writing mode that is written horizontally and from the top of the page to the bottom. The second uses `writing-mode: vertical-rl`; this is a writing mode that is written vertically and from right to left.

On this Page[What are writing modes?](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Handling_different_text_directions#What_are_writing_modes)[Writing modes and block and inline layout](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Handling_different_text_directions#Writing_modes_and_block_and_inline_layout)[Logical properties and values](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Handling_different_text_directions#Logical_properties_and_values)[Summary](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Handling_different_text_directions#Summary)[In this module](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Handling_different_text_directions#In_this_module)

[ Previous](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Backgrounds_and_borders)[ Overview: Building blocks](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks)[Next ](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content)

 

Many of the properties and values that we have encountered so far in our CSS learning have been tied to the physical dimensions of our screen. We create borders on the top, right, bottom, and left of a box, for example. These physical dimensions map very neatly to content that is viewed horizontally, and by default the web tends to support left-to-right languages (e.g. English or French) better than right-to-left languages (such as Arabic).

In recent years however, CSS has evolved in order to better support different directionality of content, including right-to-left but also top-to-bottom content (such as Japanese) — these different directionalities are called **writing modes**. As you progress in your study and begin to work with layout, an understanding of writing modes will be very helpful to you, therefore we will introduce them now.

| Prerequisites: | Basic computer literacy, [basic software installed](https://developer.mozilla.org/en-US/Learn/Getting_started_with_the_web/Installing_basic_software), basic knowledge of [working with files](https://developer.mozilla.org/en-US/Learn/Getting_started_with_the_web/Dealing_with_files), HTML basics (study [Introduction to HTML](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML)), and an idea of how CSS works (study [CSS first steps](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps).) |
| :------------- | ------------------------------------------------------------ |
| Objective:     | To understand the importance of writing modes to modern CSS. |

## What are writing modes?

A writing mode in CSS refers to whether the text is running horizontally or vertically. The [`writing-mode`](https://developer.mozilla.org/en-US/docs/Web/CSS/writing-mode) property lets us switch from one writing mode to another. You don't need to be working in a language which uses a vertical writing mode to want to do this — you could also change the writing mode of parts of your layout for creative purposes.

In the example below we have a heading displayed using `writing-mode: vertical-rl`. The text now runs vertically. Vertical text is common in graphic design, and can be a way to add a more interesting look and feel to your web design.



The three possible values for the `writing-mode` property are:

- `horizontal-tb`: Top-to-bottom block flow direction. Sentences run horizontally.
- `vertical-rl`: Right-to-left block flow direction. Sentences run vertically.
- `vertical-lr`: Left-to-right block flow direction. Sentences run vertically.

So the `writing-mode` property is in reality setting the direction in which block-level elements are displayed on the page — either from top-to-bottom, right-to-left, or left-to-right. This then dictates the direction text flows in sentences.

## Writing modes and block and inline layout

We have already discussed [block and inline layout](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/The_box_model#Block_and_inline_boxes), and the fact that some things display as block elements and others as inline elements. As we have seen described above, block and inline is tied to the writing mode of the document, and not the physical screen. Blocks are only displayed from the top to the bottom of the page if you are using a writing mode that displays text horizontally, such as English.

If we look at an example this will become clearer. In this next example I have two boxes that contain a heading and a paragraph. The first uses `writing-mode: horizontal-tb`, a writing mode that is written horizontally and from the top of the page to the bottom. The second uses `writing-mode: vertical-rl`; this is a writing mode that is written vertically and from right to left.



When we switch the writing mode, we are changing which direction is block and which is inline. In a `horizontal-tb` writing mode the block direction runs from top to bottom; in a `vertical-rl` writing mode the block direction runs right-to-left horizontally. So the **block dimension** is always the direction blocks are displayed on the page in the writing mode in use. The **inline dimension** is always the direction a sentence flows.

This figure shows the two dimensions when in a horizontal writing mode.![Showing the block and inline axis for a horizontal writing mode.](https://mdn.mozillademos.org/files/16574/horizontal-tb.png)

This figure shows the two dimensions in a vertical writing mode.

![Showing the block and inline axis for a vertical writing mode.](https://mdn.mozillademos.org/files/16575/vertical.png)

Once you start to look at CSS layout, and in particular the newer layout methods, this idea of block and inline becomes very important. We will revisit it later on.

### Direction



In addition to writing mode we also have text direction. As mentioned above, some languages such as Arabic are written horizontally, but right-to-left. This is not something you are likely to use in a creative sense — if you simply want to line something up on the right there are other ways to do so — however it is important to understand this as part of the nature of CSS. The web is not just for languages that are displayed left-to-right!

Due to the fact that writing mode and direction of text can change, newer CSS layout methods do not refer to left and right, and top and bottom. Instead they will talk about *start* and *end* along with this idea of inline and block. Don't worry too much about that right now, but keep these ideas in mind as you start to look at layout; you will find it really helpful in your understanding of CSS.

## Logical properties and values

The reason to talk about writing modes and direction at this point in your learning however, is because of the fact we have already looked at a lot of properties which are tied to the physical dimensions of the screen, and make most sense when in a horizontal writing mode.

Let's take a look at our two boxes again — one with a `horizontal-tb` writing mode and one with `vertical-rl`. I have given both of these boxes a [`width`](https://developer.mozilla.org/en-US/docs/Web/CSS/width). You can see that when the box is in the vertical writing mode, it still has a width, and this is causing the text to overflow.

What we really want in this scenario, is to essentially swap height and width along with the writing mode. When we're in a vertical writing mode we want the box to expand in the block dimension just like it does in the horizontal mode.

To make this easier, CSS has recently developed a set of mapped properties. These essentially replace physical properties — things like `width` and `height` — with **logical**, or **flow relative** versions.

The property mapped to `width` when in a horizontal writing mode is called [`inline-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/inline-size) — it refers to the size in the inline dimension. The property for `height` is named [`block-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/block-size) and is the size in the block dimension. You can see how this works in the example below where we have replaced `width` with `inline-size`.

```css
.box {
  inline-size: 150px;
}

.horizontal {
  writing-mode: horizontal-tb;
}

.vertical {
  writing-mode: vertical-rl;
}
```

### Logical margin, border, and padding properties



In the last two lessons we have learned about the CSS box model, and CSS borders. In the margin, border, and padding properties you will find many instances of physical properties, for example [`margin-top`](https://developer.mozilla.org/en-US/docs/Web/CSS/margin-top), [`padding-left`](https://developer.mozilla.org/en-US/docs/Web/CSS/padding-left), and [`border-bottom`](https://developer.mozilla.org/en-US/docs/Web/CSS/border-bottom). In the same way that we have mappings for width and height there are mappings for these properties.

The `margin-top` property is mapped to [`margin-block-start`](https://developer.mozilla.org/en-US/docs/Web/CSS/margin-block-start) — this will always refer to the margin at the start of the block dimension.

The [`padding-left`](https://developer.mozilla.org/en-US/docs/Web/CSS/padding-left) property maps to [`padding-inline-start`](https://developer.mozilla.org/en-US/docs/Web/CSS/padding-inline-start), the padding that is applied to the start of the inline direction. This will be where sentences start in that writing mode. The [`border-bottom`](https://developer.mozilla.org/en-US/docs/Web/CSS/border-bottom) property maps to [`border-block-end`](https://developer.mozilla.org/en-US/docs/Web/CSS/border-block-end), which is the border at the end of the block dimension.

You can see a comparison between physical and logical properties below.

If you change the writing mode of the boxes by switching the `writing-mode` property on `.box` to `vertical-rl`, you will see how the physical properties stay tied to their physical direction, whereas the logical properties switch with the writing mode.

You can also see that the `<h2>` has a black `border-bottom`. Can you work out how to make that bottom border always go below the text in both writing modes?

There are a huge number of properties when you consider all of the individual border longhands, and you can see all of the mapped properties on the MDN page for [Logical Properties and Values](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Logical_Properties).

### Logical values



We have so far looked at logical property names. There are also some properties that take physical values of `top`, `right`, `bottom`, and `left`. These values also have mappings, to logical values — `block-start`, `inline-end`, `block-end`, and `inline-start`.

For example, you can float an image left to cause text to wrap round the image. You could replace `left` with `inline-start` as shown in the example below.

Change the writing mode on this example to `vertical-rl` to see what happens to the image. Change `inline-start` to `inline-end` to change the float.

Here we are also using logical margin values to ensure the margin is in the correct place no matter what the writing mode is.

### Should you use physical or logical properties?



The logical properties and values are newer than their physical equivalents, and therefore have only recently been implemented in browsers. You can check any property page on MDN to see how far back the browser support goes. If you are not using multiple writing modes then for now you might prefer to use the physical versions. However, ultimately we expect that people will transition to the logical versions for most things, as they make a lot of sense once you start also dealing with layout methods such as flexbox and grid.

## Summary

The concepts explained in this lesson are becoming increasingly important in CSS. An understanding of the block and inline direction — and how text flow changes with a change in writing mode — will be very useful going forward. It will help you in understanding CSS even if you never use a writing mode other than a horizontal one.

In the next module we will take a good look at overflow in CSS.

# Overflowing content

In this lesson we will look at another important concept in CSS — **overflow**. Overflow is what happens when there is too much content to be contained comfortably inside a box. In this guide you will learn what it is and how to manage it. 

## What is overflow?

We already know that everything in CSS is a box, and that we can constrain the size of these boxes by giving them values of [`width`](https://developer.mozilla.org/en-US/docs/Web/CSS/width) and [`height`](https://developer.mozilla.org/en-US/docs/Web/CSS/height) (or [`inline-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/inline-size) and [`block-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/block-size)). Overflow is what happens when you have too much content in a box, so it won't fit inside it comfortably. CSS gives you various tools to manage this overflow, and it is also a useful concept to understand at this early stage. You will come across overflow situations quite often when writing CSS, especially when you get deeper into CSS layout.

## CSS tries to avoid "data loss"

Let's start off by looking at two examples that demonstrate how CSS behaves by default when you have overflow.

The first is a box that has been restricted in the block dimension by giving it a `height`. We have then added more content than there is space for in this box. The content is overflowing the box and laying itself rather messily over the paragraph below the box.

On this Page[What is overflow?](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content#What_is_overflow)[CSS tries to avoid "data loss"](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content#CSS_tries_to_avoid_data_loss)[The overflow property](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content#The_overflow_property)[Overflow establishes a Block Formatting Context](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content#Overflow_establishes_a_Block_Formatting_Context)[Unwanted overflow in web design](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content#Unwanted_overflow_in_web_design)[Summary](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content#Summary)[In this module](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content#In_this_module)

[ Previous](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Handling_different_text_directions)[ Overview: Building blocks](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks)[Next ](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units)

 

In this lesson we will look at another important concept in CSS — **overflow**. Overflow is what happens when there is too much content to be contained comfortably inside a box. In this guide you will learn what it is and how to manage it.

| Prerequisites: | Basic computer literacy, [basic software installed](https://developer.mozilla.org/en-US/Learn/Getting_started_with_the_web/Installing_basic_software), basic knowledge of [working with files](https://developer.mozilla.org/en-US/Learn/Getting_started_with_the_web/Dealing_with_files), HTML basics (study [Introduction to HTML](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML)), and an idea of how CSS works (study [CSS first steps](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps).) |
| :------------- | ------------------------------------------------------------ |
| Objective:     | To understand overflow and how to manage it.                 |

## What is overflow?

We already know that everything in CSS is a box, and that we can constrain the size of these boxes by giving them values of [`width`](https://developer.mozilla.org/en-US/docs/Web/CSS/width) and [`height`](https://developer.mozilla.org/en-US/docs/Web/CSS/height) (or [`inline-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/inline-size) and [`block-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/block-size)). Overflow is what happens when you have too much content in a box, so it won't fit inside it comfortably. CSS gives you various tools to manage this overflow, and it is also a useful concept to understand at this early stage. You will come across overflow situations quite often when writing CSS, especially when you get deeper into CSS layout.

## CSS tries to avoid "data loss"

Let's start off by looking at two examples that demonstrate how CSS behaves by default when you have overflow.

The first is a box that has been restricted in the block dimension by giving it a `height`. We have then added more content than there is space for in this box. The content is overflowing the box and laying itself rather messily over the paragraph below the box.



The second is a word in a box that is restricted in the inline dimension. The box has been made too small for that word to fit and so it breaks out of the box.

You might wonder why CSS has by default taken the rather untidy approach of causing the content to overflow messily? Why not hide the additional content, or cause the box to grow?

Wherever possible CSS does not hide your content; to do so would cause data loss, which is usually a problem. In CSS terms, this means some content vanishing. The problem with content vanishing is that you might not notice it has vanished. Your visitors may not notice it has vanished. If it is the submit button on a form that disappears, and no-one can complete the form, that's a big problem! So instead, CSS tends to overflow in a visible way. It is likely you will see the mess, or at worst a visitor to your site will let you know that some content is overlapping so it needs fixing.

If you have restricted a box with a `width` or a `height`, CSS assumes you know what you are doing, and that you are managing the potential for overflow. In general, restricting the block dimension is problematic when text is going to be put in a box, as there may be more text than you expected when designing the site or the text may be bigger — for example if the user has increased their font size.

In the next couple of lessons we will look at different ways to control sizing that might be less prone to overflow. However, if you need a fixed size you can also control how the overflow behaves. Let's read on!

## The overflow property

The [`overflow`](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow) property is how you take control of an element's overflow and tell the browser how you want it to behave. The default value of overflow is `visible`, which is why by default we can see our content when it overflows.

If you want to crop the content when it overflows you can set `overflow: hidden` on your box. This will do exactly what it says — hide the overflow. This may well cause things to vanish so you should only ever do this if hiding content is not going to cause a problem.

Perhaps you would instead like to add scrollbars when content overflows? If you use `overflow: scroll` then your browser will always display scrollbars — even if there is not enough content to overflow. You may want this, as it prevents scrollbars appearing and disappearing depending on content.

If you remove some of the content from the box below, you'll see that the scrollbars still remain even with nothing to scroll (or at least just the scrollbar tracks).

 In the above example we only need to scroll on the `y` axis, however we get scrollbars in both axes. You could instead use the [`overflow-y`](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-y) property, setting `overflow-y: scroll` to only scroll on the `y` axis. 

On this Page[What is overflow?](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content#What_is_overflow)[CSS tries to avoid "data loss"](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content#CSS_tries_to_avoid_data_loss)[The overflow property](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content#The_overflow_property)[Overflow establishes a Block Formatting Context](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content#Overflow_establishes_a_Block_Formatting_Context)[Unwanted overflow in web design](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content#Unwanted_overflow_in_web_design)[Summary](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content#Summary)[In this module](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content#In_this_module)

[ Previous](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Handling_different_text_directions)[ Overview: Building blocks](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks)[Next ](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units)

 

In this lesson we will look at another important concept in CSS — **overflow**. Overflow is what happens when there is too much content to be contained comfortably inside a box. In this guide you will learn what it is and how to manage it.

| Prerequisites: | Basic computer literacy, [basic software installed](https://developer.mozilla.org/en-US/Learn/Getting_started_with_the_web/Installing_basic_software), basic knowledge of [working with files](https://developer.mozilla.org/en-US/Learn/Getting_started_with_the_web/Dealing_with_files), HTML basics (study [Introduction to HTML](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML)), and an idea of how CSS works (study [CSS first steps](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps).) |
| :------------- | ------------------------------------------------------------ |
| Objective:     | To understand overflow and how to manage it.                 |

## What is overflow?

We already know that everything in CSS is a box, and that we can constrain the size of these boxes by giving them values of [`width`](https://developer.mozilla.org/en-US/docs/Web/CSS/width) and [`height`](https://developer.mozilla.org/en-US/docs/Web/CSS/height) (or [`inline-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/inline-size) and [`block-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/block-size)). Overflow is what happens when you have too much content in a box, so it won't fit inside it comfortably. CSS gives you various tools to manage this overflow, and it is also a useful concept to understand at this early stage. You will come across overflow situations quite often when writing CSS, especially when you get deeper into CSS layout.

## CSS tries to avoid "data loss"

Let's start off by looking at two examples that demonstrate how CSS behaves by default when you have overflow.

The first is a box that has been restricted in the block dimension by giving it a `height`. We have then added more content than there is space for in this box. The content is overflowing the box and laying itself rather messily over the paragraph below the box.



The second is a word in a box that is restricted in the inline dimension. The box has been made too small for that word to fit and so it breaks out of the box.



You might wonder why CSS has by default taken the rather untidy approach of causing the content to overflow messily? Why not hide the additional content, or cause the box to grow?

Wherever possible CSS does not hide your content; to do so would cause data loss, which is usually a problem. In CSS terms, this means some content vanishing. The problem with content vanishing is that you might not notice it has vanished. Your visitors may not notice it has vanished. If it is the submit button on a form that disappears, and no-one can complete the form, that's a big problem! So instead, CSS tends to overflow in a visible way. It is likely you will see the mess, or at worst a visitor to your site will let you know that some content is overlapping so it needs fixing.

If you have restricted a box with a `width` or a `height`, CSS assumes you know what you are doing, and that you are managing the potential for overflow. In general, restricting the block dimension is problematic when text is going to be put in a box, as there may be more text than you expected when designing the site or the text may be bigger — for example if the user has increased their font size.

In the next couple of lessons we will look at different ways to control sizing that might be less prone to overflow. However, if you need a fixed size you can also control how the overflow behaves. Let's read on!

## The overflow property

The [`overflow`](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow) property is how you take control of an element's overflow and tell the browser how you want it to behave. The default value of overflow is `visible`, which is why by default we can see our content when it overflows.

If you want to crop the content when it overflows you can set `overflow: hidden` on your box. This will do exactly what it says — hide the overflow. This may well cause things to vanish so you should only ever do this if hiding content is not going to cause a problem.



Perhaps you would instead like to add scrollbars when content overflows? If you use `overflow: scroll` then your browser will always display scrollbars — even if there is not enough content to overflow. You may want this, as it prevents scrollbars appearing and disappearing depending on content.

If you remove some of the content from the box below, you'll see that the scrollbars still remain even with nothing to scroll (or at least just the scrollbar tracks).



In the above example we only need to scroll on the `y` axis, however we get scrollbars in both axes. You could instead use the [`overflow-y`](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-y) property, setting `overflow-y: scroll` to only scroll on the `y` axis.



You could also scroll on the x axis using [`overflow-x`](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-x), although this is not a recommended way to deal with long words! If you do need to deal with a long word in a small box then you could look at the [`word-break`](https://developer.mozilla.org/en-US/docs/Web/CSS/word-break) or [`overflow-wrap`](https://developer.mozilla.org/en-US/docs/Web/CSS/overflow-wrap) properties. In addition some of the methods discussed in the [Sizing items in CSS](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Sizing_items_in_CSS) lesson may help you to create boxes that cope better with varying amounts of content.

As with `scroll`, you will get a scrollbar in the scrolling dimension whether or not there is enough content to cause a scrollbar.

**Note**: that you can specify x and y scrolling using the `overflow` property and passing in two values. If two keywords are specified, the first applies to `overflow-x` and the second to `overflow-y`. Otherwise, both `overflow-x` and `overflow-y` are set to the same value. For example, `overflow: scroll hidden` would set `overflow-x` to `scroll` and `overflow-y` to `hidden`.

If you only want scrollbars to appear if there is more content than can fit in the box, then use `overflow: auto`. In this case it is left up to the browser to decide whether to display scrollbars. Desktop browsers will typically only do so once there is enough content to cause overflow.

In the below example, remove some of the content until it fits into the box and you should see the scrollbars disappear.

## Overflow establishes a Block Formatting Context

There is a concept in CSS of the **Block Formatting Context** (BFC). This isn't something you need to worry too much about right now, but it is useful to know that when you use a value of overflow such as `scroll` or `auto` you create a BFC. The result is that the content of the box you have changed the value of `overflow` for becomes a mini layout of its own. Things outside the container cannot poke into the container, and nothing can poke out of that box into the surrounding layout. This is to enable the scrolling behavior, as all content of your box will need to be contained and not overlap other items on the page, in order to create a consistent scrolling experience.

## Unwanted overflow in web design

Modern layout methods (as covered in the [CSS layout](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout) module) manage overflow very well. They have been designed to cope with the fact that we tend not to be able to predict how much content we have on the web. In the past however, developers would often use fixed heights to try to line up the bottoms of boxes that really had no relationship to each other. This was fragile, and in a legacy application you may occasionally come across a box where the content is overlaying other content on the page. If you see this you now know that what is happening is overflow; ideally you would refactor the layout to not rely on fixing the box size.

When developing a site you should always keep overflow issues in mind. You should test designs with large and small amounts of content, increase the font size of text and ensure that your CSS can cope in a robust way. Changing the value of overflow to hide content or add scrollbars is likely to be something you reserve only for a few special cases — where you really do want a scrolling box for example.

## Summary

This short lesson has introduced the concept of overflow; you now understand that CSS tries not to make overflowing content invisible as this will cause data loss. You have discovered that you can manage potential overflow, and also that you should test your work to make sure you do not accidentally cause problematic overflow.