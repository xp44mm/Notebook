# CSS values and units

Every property used in CSS has a value or set of values that are allowed for that property, and taking a look at any property page on MDN will help you understand the values that are valid for any particular property. In this lesson we will take a look at some of the most common values and units in use. 

## What is a CSS value?

In CSS specifications and on the property pages here on MDN you will be able to spot values as they will be surrounded by angle brackets, such as `<color>` or `<length>`. When you see the value `<color>` as valid for a particular property, that means you can use any valid color as a value for that property, as listed on the `<color>` reference page.

**Note**: You'll also see CSS values referred to as *data types*. The terms are basically interchangeable — when you see something in CSS referred to as a data type, it is really just a fancy way of saying value.

**Note**: Yes, CSS values tend to be denoted using angle brackets, to differentiate them from CSS properties (e.g. the [`color`](https://developer.mozilla.org/en-US/docs/Web/CSS/color) property, versus the `<color>` data type). You might get confused between CSS data types and HTML elements too, as they both use angle brackets, but this is unlikely — they are used in very different contexts.

In the following example we have set the color of our heading using a keyword, and the background using the `rgb()` function:

```css
h1 { 
  color: black; 
  background-color: rgb(197,93,161); 
} 
```

A value in CSS is a way to define a collection of allowable sub-values. This means that if you see `<color>` as valid you don't need to wonder which of the different types of color value can be used — keywords, hex values, `rgb()` functions, etc. You can use *any* available `<color>` values assuming they are supported by your browser. The page on MDN for each value will give you information about browser support. For example, if you look at the page for `<color>` you will see that the browser compatibility section lists different types of color value and support for them.

Let's have a look at some of the types of value and unit you may frequently encounter, with examples so that you can try out different possible values.

## Numbers, lengths, and percentages

There are various numeric data types that you might find yourself using in CSS. The following are all classed as numeric:

| Data type      | Description                                                  |
| :------------- | :----------------------------------------------------------- |
| `<integer>`    | is a whole number such as `1024` or `-55`.                   |
| `<number>`     | represents a decimal number — it may or may not have a decimal point with a fractional component, for example `0.255`, `128`, or `-1.2`. |
| `<dimension>`  | is a `<number>` with a unit attached to it, for example `45deg`, `5s`, or `10px`. `<dimension>` is an umbrella category that includes the `<length>`, `<angle>`, `<time>`, and `<resolution>` types[.](https://developer.mozilla.org/en-US/docs/Web/CSS/resolution) |
| `<percentage>` | represents a fraction of some other value, for example `50%`. Percentage values are always relative to another quantity, for example an element's length is relative to its parent element's length. |

### Lengths



The numeric type you will come across most frequently is `<length>`, for example `10px` (pixels) or `30em`. There are two types of lengths used in CSS — relative and absolute. It's important to know the difference in order to understand how big things will become.

#### Absolute length units

The following are all **absolute** length units — they are not relative to anything else and are generally considered to always be the same size.

| Unit | Name                | Equivalent to       |
| :--- | :------------------ | :------------------ |
| `cm` | Centimeters         | 1cm = 96px/2.54     |
| `mm` | Millimeters         | 1mm = 1/10th of 1cm |
| `Q`  | Quarter-millimeters | 1Q = 1/40th of 1cm  |
| `in` | Inches              | 1in = 2.54cm = 96px |
| `pc` | Picas               | 1pc = 1/16th of 1in |
| `pt` | Points              | 1pt = 1/72th of 1in |
| `px` | Pixels              | 1px = 1/96th of 1in |

Most of these values are more useful when used for print, rather than screen output. For example we don't typically use `cm` (centimeters) on screen. The only value that you will commonly use is `px` (pixels).

#### Relative length units

Relative length units are relative to something else, perhaps the size of the parent element's font, or the size of the viewport. The benefit of using relative units is that with some careful planning you can make it so the size of text or other elements scale relative to everything else on the page. Some of the most useful units for web development are listed in the table below.

| Unit   | Relative to                                                  |
| :----- | :----------------------------------------------------------- |
| `em`   | Font size of the parent element.                             |
| `ex`   | x-height of the element's font.                              |
| `ch`   | The advance measure (width) of the glyph "0" of the element's font. |
| `rem`  | Font size of the root element.                               |
| `lh`   | Line height of the element.                                  |
| `vw`   | 1% of the viewport's width.                                  |
| `vh`   | 1% of the viewport's height.                                 |
| `vmin` | 1% of the viewport's smaller dimension.                      |
| `vmax` | 1% of the viewport's larger dimension.                       |

#### Exploring an example

In the example below you can see how some relative and absolute length units behave. The first box has a [`width`](https://developer.mozilla.org/en-US/docs/Web/CSS/width) set in pixels. As an absolute unit this width will remain the same no matter what else changes.

The second box has a width set in `vw` (viewport width) units. This value is relative to the viewport width, and so 10vw is 10 percent of the width of the viewport. If you change the width of your browser window, the size of the box should change.

The third box uses `em` units. These are relative to the font size. I've set a font size of `1em` on the containing `<div>`, which has a class of `.wrapper`. Change this value to `1.5em` and you will see that the font size of all the elements increases, but only the last item will get wider, as the width is relative to that font size.

After following the instructions above, try playing with the values in other ways, to see what you get.

#### ems and rems

`em` and `rem` are the two relative lengths you are likely to encounter most frequently when sizing anything from boxes to text. It's worth understanding how these work, and the differences between them, especially when you start getting on to more complex subjects like [styling text](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text) or [CSS layout](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout). The below example provides a demonstration.

The HTML is a set of nested lists — we have three lists in total and both examples have the same HTML. The only difference is that the first has a class of *ems* and the second a class of *rems*.

To start with, we set 16px as the font size on the `<html>` element.

**To recap, the em unit means "my parent element's font-size"**. The `<li>` elements inside the `<ul>` with a `class` of `ems` take their sizing from their parent. So each successive level of nesting gets progressively larger, as each has its font size set to `1.3em` — 1.3 times its parent's font size.

**To recap, the rem unit means "The root element's font-size"**. (rem standards for "root em".) The `<li>` elements inside the `<ul>` with a `class` of `rems` take their sizing from the root element (`<html>`). This means that each successive level of nesting does not keep getting larger.

However, if you change the `<html>` `font-size` in the CSS you will see that everything else changes relative to it — both `rem`- and `em`-sized text.

### Percentages



In a lot of cases a percentage is treated in the same way as a length. The thing with percentages is that they are always set relative to some other value. For example, if you set an element's `font-size` as a percentage it will be a percentage of the `font-size` of the element's parent. If you use a percentage for a `width` value, it will be a percentage of the `width` of the parent.

In the below example the two percentage-sized boxes and the two pixel-sized boxes have the same class names. Both sets are 200px and 40% wide respectively.

The difference is that the second set of two boxes is inside a wrapper that is 400 pixels wide. The second 200px wide box is the same width as the first one, but the second 40% box is now 40% of 400px — a lot narrower than the first one!

**Try changing the width of the wrapper or the percentage value to see how this works.**

 The next example has font sizes set in percentages. Each `<li>` has a `font-size` of 80%, therefore the nested list items become progressively smaller as they inherit their sizing from their parent. 

Note that, while many values accept a length or a percentage, there are some that only accept length. You can see which values are accepted on the MDN property reference pages. If the allowed value includes `<length-percentage>` then you can use a length or a percentage. If the allowed value only includes `<length>`, it is not possible to use a percentage.

### Numbers



Some values accept numbers, without any unit added to them. An example of a property which accepts a unitless number is the `opacity` property, which controls the opacity of an element (how transparent it is). This property accepts a number between `0` (fully transparent) and `1` (fully opaque).

In the below example, try changing the value of `opacity` to various decimal values between `0` and `1` and see how the box and its contents become more or less opaque.

**Note**: When you use a number in CSS as a value it should not be surrounded in quotes.

## Color

There are many ways to specify color in CSS, some of which are more recently implemented than others. The same color values can be used everywhere in CSS, whether you are specifying text color, background color, or whatever else.

The standard color system available in modern computers is 24 bit, which allows the display of about 16.7 million distinct colors via a combination of different red, green and blue channels with 256 different values per channel (256 x 256 x 256 = 16,777,216.) Let's have a look at some of the ways in which we can specify colors in CSS.

**Note**: In this tutorial we will look at the common methods of specifying color that have good browser support; there are others but they don't have as good support and are less common.

### Color keywords



Quite often in examples here in the learn section or elsewhere on MDN you will see the color keywords used, as they are a simple and understandable way of specifying color. There are a number of these keywords, some of which have fairly entertaining names! You can see a full list on the page for the `<color>` value.

Try playing with different color values in the live examples below, to get more of an idea how they work.

### Hexadecimal RGB values



The next type of color value you are likely to encounter is hexadecimal codes. Each hex value consists of a hash/pound symbol (#) followed by six hexadecimal numbers, each of which can take one of 16 values between 0 and f (which represents 15) — so `0123456789abcdef`. Each pair of values represents one of the channels — red, green and blue — and allows us to specify any of the 256 available values for each (16 x 16 = 256.)

These values are a bit more complex and less easy to understand, but they are a lot more versatile than keywords — you can use hex values to represent any color you want to use in your color scheme.

**Again, try changing the values to see how the colors vary.**

### RGB and RGBA values



The third scheme we'll talk about here is RGB. An RGB value is a function — `rgb()` — which is given three parameters that represent the red, green, and blue channel values of the colors, in much the same way as hex values. The difference with RGB is that each channel is represented not by two hex digits, but by a decimal number between 0 and 255 — somewhat easier to understand.

Let's rewrite our last example to use RGB colors:

You can also use RGBA colors — these work in exactly the same way as RGB colors, and so you can use any RGB values, however there is a fourth value that represents the alpha channel of the color, which controls opacity. If you set this value to `0` it will make the color fully transparent, whereas `1` will make it fully opaque. Values in between give you different levels of transparency.

**Note**: Setting an alpha channel on a color has one key difference to using the [`opacity`](https://developer.mozilla.org/en-US/docs/Web/CSS/opacity) property we looked at earlier. When you use opacity you make the element and everything inside it opaque, whereas using RGBA colors only makes the color you are specifying opaque.

In the example below I have added a background image to the containing block of our colored boxes. I have then set the boxes to have different opacity values — notice how the background shows through more when the alpha channel value is smaller.

In this example, try changing the alpha channel values to see how it affects the color output.

**Note**: At some point modern browsers were updated so that `rgba()` and `rgb()`, and `hsl()` and `hsla()` (see below), became pure aliases of each other and started to behave exactly the same. So for example both `rgba()` and `rgb()` accept colors with and without alpha channel values. Try changing the above example's `rgba()` functions to `rgb()` and see if the colors still work! Which style you use is up to you, but separating out non-transparent and transparent color definitions to use the different functions gives (very) slightly better browser support and can act as a visual indicator of where transparent colors are being defined in your code.

### HSL and HSLA values



Slightly less well-supported than RGB is the HSL color model (not supported on old versions of IE), which was implemented after much interest from designers. Instead of red, green, and blue values, the `hsl()` function accepts hue, saturation, and lightness values, which are used to distinguish between the 16.7 million colors, but in a different way:

- **Hue**: The base shade of the color. This takes a value between 0 and 360, representing the angles round a color wheel.
- **Saturation**: How saturated is the color? This takes a value from 0–100%, where 0 is no color (it will appear as a shade of grey), and 100% is full color saturation
- **Lightness**: How light or bright is the color? This takes a value from 0–100%, where 0 is no light (it will appear completely black) and 100% is full light (it will appear completely white)

We can update the RGB example to use HSL colors like this:

Just as RGB has RGBA, HSL has an HSLA equivalent, which gives you the same ability to specify the alpha channel. I've demonstrated this below by changing my RGBA example to use HSLA colors. 

You can use any of these color values in your projects. It is likely that for most projects you will decide on a color palette and then use those colors — and your chosen method of specifying color — throughout the whole project. You can mix and match color models, however for consistency it is usually best if your entire project uses the same one!

## Images

The `<image>` data type is used wherever an image is a valid value. This can be an actual image file pointed to via a `url()` function, or a gradient.

In the example below we have demonstrated an image and a gradient in use as a value for the CSS `background-image` property.

**Note**: there are some other possible values for `<image>`, however these are newer and currently have poor browser support. Check out the page on MDN for the `<image>` data type if you want to read about them. 

## Position

The `<position>` data type represents a set of 2D coordinates, used to position an item such as a background image (via `background-position`). It can take keywords such as `top`, `left`, `bottom`, `right`, and `center` to align items with specific bounds of a 2D box, along with lengths, which represent offsets from the top and left-hand edges of the box.

A typical position value consists of two values — the first sets the position horizontally, the second vertically. If you only specify values for one axis the other will default to `center`.

In the following example we have positioned a background image 40px from the top and to the right of the container using a keyword.

Play around with these values to see how you can push the image around.

## Strings and identifiers

Throughout the examples above, we've seen places where keywords are used as a value (for example `<color>` keywords like `red`, `black`, `rebeccapurple`, and `goldenrod`). These keywords are more accurately described as *identifiers*, a special value that CSS understands. As such they are not quoted — they are not treated as strings.

There are places where you use strings in CSS, for example [when specifying generated content](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors/Pseudo-classes_and_pseudo-elements#Generating_content_with_before_and_after). In this case the value is quoted to demonstrate that it is a string. In the below example we use unquoted color keywords along with a quoted generated content string.

## Functions

The final type of value we will take a look at is the group of values known as functions. In programming, a function is a reusable section of code that can be run multiple times to complete a repetitive task with minimum effort on the part of both the developer and the computer. Functions are usually associated with languages like JavaScript, Python, or C++, but they do exist in CSS too, as property values. We've already seen functions in action in the Colors section — `rgb()`, `hsl()`, etc. The value used to return an image from a file — `url()` — is also a function.

A value that behaves more like something you might find in a traditional programming language is the `calc()` CSS function. This function gives you the ability to do simple calculations inside your CSS. It's particularly useful if you want to work out values that you can't define when writing the CSS for your project, and need the browser to work out for you at runtime.

For example, below we are using `calc()` to make the box `20% + 100px` wide. The 20% is calculated from the width of the parent container `.wrapper` and so will change if that width changes. We can't do this calculation beforehand because we don't know what 20% of the parent will be, so we use `calc()` to tell the browser to do it for us.

On this Page[What is a CSS value?](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units#What_is_a_CSS_value)[Numbers, lengths, and percentages](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units#Numbers_lengths_and_percentages)[Color](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units#Color)[Images](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units#Images)[Position](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units#Position)[Strings and identifiers](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units#Strings_and_identifiers)[Functions](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units#Functions)[Summary](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units#Summary)[In this module](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units#In_this_module)

[ Previous](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content)[ Overview: Building blocks](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks)[Next ](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Sizing_items_in_CSS)

 

Every property used in CSS has a value or set of values that are allowed for that property, and taking a look at any property page on MDN will help you understand the values that are valid for any particular property. In this lesson we will take a look at some of the most common values and units in use.

| Prerequisites: | Basic computer literacy, [basic software installed](https://developer.mozilla.org/en-US/Learn/Getting_started_with_the_web/Installing_basic_software), basic knowledge of [working with files](https://developer.mozilla.org/en-US/Learn/Getting_started_with_the_web/Dealing_with_files), HTML basics (study [Introduction to HTML](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML)), and an idea of how CSS works (study [CSS first steps](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps).) |
| :------------- | ------------------------------------------------------------ |
| Objective:     | To learn about the different types of values and units used in CSS properties. |

## What is a CSS value?

In CSS specifications and on the property pages here on MDN you will be able to spot values as they will be surrounded by angle brackets, such as `` or ``. When you see the value `` as valid for a particular property, that means you can use any valid color as a value for that property, as listed on the `` reference page.

**Note**: You'll also see CSS values referred to as *data types*. The terms are basically interchangeable — when you see something in CSS referred to as a data type, it is really just a fancy way of saying value.

**Note**: Yes, CSS values tend to be denoted using angle brackets, to differentiate them from CSS properties (e.g. the [`color`](https://developer.mozilla.org/en-US/docs/Web/CSS/color) property, versus the [](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value) data type). You might get confused between CSS data types and HTML elements too, as they both use angle brackets, but this is unlikely — they are used in very different contexts.

In the following example we have set the color of our heading using a keyword, and the background using the `rgb()` function:

```
h1 { 
  color: black; 
  background-color: rgb(197,93,161); 
} 
```

A value in CSS is a way to define a collection of allowable sub-values. This means that if you see `` as valid you don't need to wonder which of the different types of color value can be used — keywords, hex values, `rgb()` functions, etc. You can use *any* available `` values assuming they are supported by your browser. The page on MDN for each value will give you information about browser support. For example, if you look at the page for `` you will see that the browser compatibility section lists different types of color value and support for them.

Let's have a look at some of the types of value and unit you may frequently encounter, with examples so that you can try out different possible values.

## Numbers, lengths, and percentages

There are various numeric data types that you might find yourself using in CSS. The following are all classed as numeric:

| Data type | Description                                                  |
| :-------- | :----------------------------------------------------------- |
| ``        | An `` is a whole number such as `1024` or `-55`.             |
| ``        | A `` represents a decimal number — it may or may not have a decimal point with a fractional component, for example `0.255`, `128`, or `-1.2`. |
| ``        | A `` is a `` with a unit attached to it, for example `45deg`, `5s`, or `10px`. `` is an umbrella category that includes the ``, ``, ``, and `` types[.](https://developer.mozilla.org/en-US/docs/Web/CSS/resolution) |
| ``        | A `` represents a fraction of some other value, for example `50%`. Percentage values are always relative to another quantity, for example an element's length is relative to its parent element's length. |

### Lengths

The numeric type you will come across most frequently is ``, for example `10px` (pixels) or `30em`. There are two types of lengths used in CSS — relative and absolute. It's important to know the difference in order to understand how big things will become.

#### Absolute length units

The following are all **absolute** length units — they are not relative to anything else and are generally considered to always be the same size.

| Unit | Name                | Equivalent to       |
| :--- | :------------------ | :------------------ |
| `cm` | Centimeters         | 1cm = 96px/2.54     |
| `mm` | Millimeters         | 1mm = 1/10th of 1cm |
| `Q`  | Quarter-millimeters | 1Q = 1/40th of 1cm  |
| `in` | Inches              | 1in = 2.54cm = 96px |
| `pc` | Picas               | 1pc = 1/16th of 1in |
| `pt` | Points              | 1pt = 1/72th of 1in |
| `px` | Pixels              | 1px = 1/96th of 1in |

Most of these values are more useful when used for print, rather than screen output. For example we don't typically use `cm` (centimeters) on screen. The only value that you will commonly use is `px` (pixels).

#### Relative length units

Relative length units are relative to something else, perhaps the size of the parent element's font, or the size of the viewport. The benefit of using relative units is that with some careful planning you can make it so the size of text or other elements scale relative to everything else on the page. Some of the most useful units for web development are listed in the table below.

| Unit   | Relative to                                                  |
| :----- | :----------------------------------------------------------- |
| `em`   | Font size of the parent element.                             |
| `ex`   | x-height of the element's font.                              |
| `ch`   | The advance measure (width) of the glyph "0" of the element's font. |
| `rem`  | Font size of the root element.                               |
| `lh`   | Line height of the element.                                  |
| `vw`   | 1% of the viewport's width.                                  |
| `vh`   | 1% of the viewport's height.                                 |
| `vmin` | 1% of the viewport's smaller dimension.                      |
| `vmax` | 1% of the viewport's larger dimension.                       |

#### Exploring an example

In the example below you can see how some relative and absolute length units behave. The first box has a [`width`](https://developer.mozilla.org/en-US/docs/Web/CSS/width) set in pixels. As an absolute unit this width will remain the same no matter what else changes.

The second box has a width set in `vw` (viewport width) units. This value is relative to the viewport width, and so 10vw is 10 percent of the width of the viewport. If you change the width of your browser window, the size of the box should change, however this example is embedded into the page using an ``, so this won't work. To see this in action you'll have to [try the example after opening it in its own browser tab](https://mdn.github.io/css-examples/learn/values-units/length.html).

The third box uses `em` units. These are relative to the font size. I've set a font size of `1em` on the containing [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/div), which has a class of `.wrapper`. Change this value to `1.5em` and you will see that the font size of all the elements increases, but only the last item will get wider, as the width is relative to that font size.

After following the instructions above, try playing with the values in other ways, to see what you get.



#### ems and rems

`em` and `rem` are the two relative lengths you are likely to encounter most frequently when sizing anything from boxes to text. It's worth understanding how these work, and the differences between them, especially when you start getting on to more complex subjects like [styling text](https://developer.mozilla.org/en-US/docs/Learn/CSS/Styling_text) or [CSS layout](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout). The below example provides a demonstration.

The HTML is a set of nested lists — we have three lists in total and both examples have the same HTML. The only difference is that the first has a class of *ems* and the second a class of *rems*.

To start with, we set 16px as the font size on the `` element.

**To recap, the em unit means "my parent element's font-size"**. The [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/li) elements inside the [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ul) with a `class` of `ems` take their sizing from their parent. So each successive level of nesting gets progressively larger, as each has its font size set to `1.3em` — 1.3 times its parent's font size.

**To recap, the rem unit means "The root element's font-size"**. (rem standards for "root em".) The [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/li) elements inside the [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ul) with a `class` of `rems` take their sizing from the root element (``). This means that each successive level of nesting does not keep getting larger.

However, if you change the `` `font-size` in the CSS you will see that everything else changes relative to it — both `rem`- and `em`-sized text.

 

### Percentages

In a lot of cases a percentage is treated in the same way as a length. The thing with percentages is that they are always set relative to some other value. For example, if you set an element's `font-size` as a percentage it will be a percentage of the `font-size` of the element's parent. If you use a percentage for a `width` value, it will be a percentage of the `width` of the parent.

In the below example the two percentage-sized boxes and the two pixel-sized boxes have the same class names. Both sets are 200px and 40% wide respectively.

The difference is that the second set of two boxes is inside a wrapper that is 400 pixels wide. The second 200px wide box is the same width as the first one, but the second 40% box is now 40% of 400px — a lot narrower than the first one!

**Try changing the width of the wrapper or the percentage value to see how this works.**

 

The next example has font sizes set in percentages. Each `` has a `font-size` of 80%, therefore the nested list items become progressively smaller as they inherit their sizing from their parent.

 

Note that, while many values accept a length or a percentage, there are some that only accept length. You can see which values are accepted on the MDN property reference pages. If the allowed value includes `` then you can use a length or a percentage. If the allowed value only includes ``, it is not possible to use a percentage.

### Numbers

Some values accept numbers, without any unit added to them. An example of a property which accepts a unitless number is the `opacity` property, which controls the opacity of an element (how transparent it is). This property accepts a number between `0` (fully transparent) and `1` (fully opaque).

**In the below example, try changing the value of `opacity` to various decimal values between `0` and `1` and see how the box and its contents become more or less opaque.**

 

**Note**: When you use a number in CSS as a value it should not be surrounded in quotes.

## Color

There are many ways to specify color in CSS, some of which are more recently implemented than others. The same color values can be used everywhere in CSS, whether you are specifying text color, background color, or whatever else.

The standard color system available in modern computers is 24 bit, which allows the display of about 16.7 million distinct colors via a combination of different red, green and blue channels with 256 different values per channel (256 x 256 x 256 = 16,777,216.) Let's have a look at some of the ways in which we can specify colors in CSS.

**Note**: In this tutorial we will look at the common methods of specifying color that have good browser support; there are others but they don't have as good support and are less common.

### Color keywords

Quite often in examples here in the learn section or elsewhere on MDN you will see the color keywords used, as they are a simple and understandable way of specifying color. There are a number of these keywords, some of which have fairly entertaining names! You can see a full list on the page for the `` value.

**Try playing with different color values in the live examples below, to get more of an idea how they work.**

### Hexadecimal RGB values

The next type of color value you are likely to encounter is hexadecimal codes. Each hex value consists of a hash/pound symbol (#) followed by six hexadecimal numbers, each of which can take one of 16 values between 0 and f (which represents 15) — so `0123456789abcdef`. Each pair of values represents one of the channels — red, green and blue — and allows us to specify any of the 256 available values for each (16 x 16 = 256.)

These values are a bit more complex and less easy to understand, but they are a lot more versatile than keywords — you can use hex values to represent any color you want to use in your color scheme.

 

**Again, try changing the values to see how the colors vary.**

### RGB and RGBA values

The third scheme we'll talk about here is RGB. An RGB value is a function — `rgb()` — which is given three parameters that represent the red, green, and blue channel values of the colors, in much the same way as hex values. The difference with RGB is that each channel is represented not by two hex digits, but by a decimal number between 0 and 255 — somewhat easier to understand.

Let's rewrite our last example to use RGB colors:

 

You can also use RGBA colors — these work in exactly the same way as RGB colors, and so you can use any RGB values, however there is a fourth value that represents the alpha channel of the color, which controls opacity. If you set this value to `0` it will make the color fully transparent, whereas `1` will make it fully opaque. Values in between give you different levels of transparency.

**Note**: Setting an alpha channel on a color has one key difference to using the [`opacity`](https://developer.mozilla.org/en-US/docs/Web/CSS/opacity) property we looked at earlier. When you use opacity you make the element and everything inside it opaque, whereas using RGBA colors only makes the color you are specifying opaque.

In the example below I have added a background image to the containing block of our colored boxes. I have then set the boxes to have different opacity values — notice how the background shows through more when the alpha channel value is smaller.



**In this example, try changing the alpha channel values to see how it affects the color output.**

**Note**: At some point modern browsers were updated so that `rgba()` and `rgb()`, and `hsl()` and `hsla()` (see below), became pure aliases of each other and started to behave exactly the same. So for example both `rgba()` and `rgb()` accept colors with and without alpha channel values. Try changing the above example's `rgba()` functions to `rgb()` and see if the colors still work! Which style you use is up to you, but separating out non-transparent and transparent color definitions to use the different functions gives (very) slightly better browser support and can act as a visual indicator of where transparent colors are being defined in your code.

### HSL and HSLA values

Slightly less well-supported than RGB is the HSL color model (not supported on old versions of IE), which was implemented after much interest from designers. Instead of red, green, and blue values, the `hsl()` function accepts hue, saturation, and lightness values, which are used to distinguish between the 16.7 million colors, but in a different way:

- **Hue**: The base shade of the color. This takes a value between 0 and 360, representing the angles round a color wheel.
- **Saturation**: How saturated is the color? This takes a value from 0–100%, where 0 is no color (it will appear as a shade of grey), and 100% is full color saturation
- **Lightness**: How light or bright is the color? This takes a value from 0–100%, where 0 is no light (it will appear completely black) and 100% is full light (it will appear completely white)

We can update the RGB example to use HSL colors like this:

 

Just as RGB has RGBA, HSL has an HSLA equivalent, which gives you the same ability to specify the alpha channel. I've demonstrated this below by changing my RGBA example to use HSLA colors.

 

You can use any of these color values in your projects. It is likely that for most projects you will decide on a color palette and then use those colors — and your chosen method of specifying color — throughout the whole project. You can mix and match color models, however for consistency it is usually best if your entire project uses the same one!

## Images

The `` data type is used wherever an image is a valid value. This can be an actual image file pointed to via a `url()` function, or a gradient.

In the example below we have demonstrated an image and a gradient in use as a value for the CSS `background-image` property.

 

**Note**: there are some other possible values for ``, however these are newer and currently have poor browser support. Check out the page on MDN for the `` data type if you want to read about them.

## Position

The `` data type represents a set of 2D coordinates, used to position an item such as a background image (via `background-position`). It can take keywords such as `top`, `left`, `bottom`, `right`, and `center` to align items with specific bounds of a 2D box, along with lengths, which represent offsets from the top and left-hand edges of the box.

A typical position value consists of two values — the first sets the position horizontally, the second vertically. If you only specify values for one axis the other will default to `center`.

In the following example we have positioned a background image 40px from the top and to the right of the container using a keyword.

 

**Play around with these values to see how you can push the image around.**

## Strings and identifiers

Throughout the examples above, we've seen places where keywords are used as a value (for example `` keywords like `red`, `black`, `rebeccapurple`, and `goldenrod`). These keywords are more accurately described as *identifiers*, a special value that CSS understands. As such they are not quoted — they are not treated as strings.

There are places where you use strings in CSS, for example [when specifying generated content](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Selectors/Pseudo-classes_and_pseudo-elements#Generating_content_with_before_and_after). In this case the value is quoted to demonstrate that it is a string. In the below example we use unquoted color keywords along with a quoted generated content string.

 

## Functions

The final type of value we will take a look at is the group of values known as functions. In programming, a function is a reusable section of code that can be run multiple times to complete a repetitive task with minimum effort on the part of both the developer and the computer. Functions are usually associated with languages like JavaScript, Python, or C++, but they do exist in CSS too, as property values. We've already seen functions in action in the Colors section — `rgb()`, `hsl()`, etc. The value used to return an image from a file — `url()` — is also a function.

A value that behaves more like something you might find in a traditional programming language is the `calc()` CSS function. This function gives you the ability to do simple calculations inside your CSS. It's particularly useful if you want to work out values that you can't define when writing the CSS for your project, and need the browser to work out for you at runtime.

For example, below we are using `calc()` to make the box `20% + 100px` wide. The 20% is calculated from the width of the parent container `.wrapper` and so will change if that width changes. We can't do this calculation beforehand because we don't know what 20% of the parent will be, so we use `calc()` to tell the browser to do it for us.



## Summary

This has been a quick run through of the most common types of values and units you might encounter. You can have a look at all of the different types on the [CSS Values and units](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Values_and_Units) reference page; you will encounter many of these in use as you work through these lessons.

The key thing to remember is that each property has a defined list of allowed values, and each value has a definition explaining what the sub-values are. You can then look up the specifics here on MDN.

For example, understanding that `` also allows you to create a color gradient is useful but perhaps non-obvious knowledge to have!

# Sizing items in CSS

In the various lessons so far you have come across a number of ways to size items on a web page using CSS. Understanding how big the different features in your design will be is important, and in this lesson we will summarize the various ways elements get a size via CSS and define a few terms around sizing that will help you in the future. 

## The natural or intrinsic size of things

HTML Elements have a natural size, set before they are affected by any CSS. A straightforward example is an image. An image has a width and a height defined in the image file it is embedding into the page. This size is described as the **intrinsic size** — which comes from the image itself.

If you place an image on a page and do not change its height and width, either using attributes on the `` tag or CSS, it will be displayed using that intrinsic size. We have given the image in the below example a border so that you can see the extent of the file.

 An empty `<div>` however, has no size of its own. If you add a `<div>` to your HTML with no content, then give it a border as we did with the image, you will see a line on the page. This is the collapsed border on the element — there is no content to hold it open. In our example below, that border stretches to the width of the container, because it is a block level element, a behavior that should be starting to become familiar to you. It has no height (or size in the block dimension) because there is no content. 

In the example above, try adding some text inside the empty element. The border now contains that text because the height of the element is defined by the content. Therefore the size of this `<div>` in the block dimension comes from the size of the content. Again, this is the intrinsic size of the element — its size is defined by its content.

## Setting a specific size

We can of course give elements in our design a specific size. When a size is given to an element (and the content of which then needs to fit into that size) we refer to it as an **extrinsic size**. Take our `<div>` from the example above — we can give it specific [`width`](https://developer.mozilla.org/en-US/docs/Web/CSS/width) and [`height`](https://developer.mozilla.org/en-US/docs/Web/CSS/height) values, and it will now have that size no matter what content is placed into it. As we discovered in [our previous lesson on overflow](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content), a set height can cause content to overflow if there is more content than the element has space to fit inside it.

Due to this problem of overflow, fixing the height of elements with lengths or percentages is something we need to do very carefully on the web.

### Using percentages



In many ways percentages act like length units, and as we [discussed in the lesson on values and units](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units#Percentages), they can often be used interchangeably with lengths. When using a percentage you need to be aware what it is a percentage *of*. In the case of a box inside another container, if you give the child box a percentage width it will be a percentage of the width of the parent container.

This is because percentages resolve against the size of the containing block. With no percentage applied our `<div>` would take up 100% of the available space, as it is a block level element. If we give it a percentage width, this becomes a percentage of the space it would normally fill.

### Percentage margins and padding



If you set `margins` and `padding` as a percentage you may notice some strange behavior. In the below example we have a box. We have given the inner box a [`margin`](https://developer.mozilla.org/en-US/docs/Web/CSS/margin) of 10% and a [`padding`](https://developer.mozilla.org/en-US/docs/Web/CSS/padding) of 10%. The padding and margin on the top and bottom of the box are the same size as the margin on the left and right.

On this Page[The natural or intrinsic size of things](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Sizing_items_in_CSS#The_natural_or_intrinsic_size_of_things)[Setting a specific size](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Sizing_items_in_CSS#Setting_a_specific_size)[min- and max- sizes](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Sizing_items_in_CSS#min-_and_max-_sizes)[Viewport units](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Sizing_items_in_CSS#Viewport_units)[Summary](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Sizing_items_in_CSS#Summary)[In this module](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Sizing_items_in_CSS#In_this_module)

[ Previous](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units)[ Overview: Building blocks](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks)[Next ](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Images_media_form_elements)

 

In the various lessons so far you have come across a number of ways to size items on a web page using CSS. Understanding how big the different features in your design will be is important, and in this lesson we will summarize the various ways elements get a size via CSS and define a few terms around sizing that will help you in the future.

| Prerequisites: | Basic computer literacy, [basic software installed](https://developer.mozilla.org/en-US/Learn/Getting_started_with_the_web/Installing_basic_software), basic knowledge of [working with files](https://developer.mozilla.org/en-US/Learn/Getting_started_with_the_web/Dealing_with_files), HTML basics (study [Introduction to HTML](https://developer.mozilla.org/en-US/docs/Learn/HTML/Introduction_to_HTML)), and an idea of how CSS works (study [CSS first steps](https://developer.mozilla.org/en-US/docs/Learn/CSS/First_steps).) |
| :------------- | ------------------------------------------------------------ |
| Objective:     | To understand the different ways we can size things in CSS.  |

## The natural or intrinsic size of things

HTML Elements have a natural size, set before they are affected by any CSS. A straightforward example is an image. An image has a width and a height defined in the image file it is embedding into the page. This size is described as the **intrinsic size** — which comes from the image itself.

If you place an image on a page and do not change its height and width, either using attributes on the `` tag or CSS, it will be displayed using that intrinsic size. We have given the image in the below example a border so that you can see the extent of the file.



An empty [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/div) however, has no size of its own. If you add a [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/div) to your HTML with no content, then give it a border as we did with the image, you will see a line on the page. This is the collapsed border on the element — there is no content to hold it open. In our example below, that border stretches to the width of the container, because it is a block level element, a behavior that should be starting to become familiar to you. It has no height (or size in the block dimension) because there is no content.



In the example above, try adding some text inside the empty element. The border now contains that text because the height of the element is defined by the content. Therefore the size of this `` in the block dimension comes from the size of the content. Again, this is the intrinsic size of the element — its size is defined by its content.

## Setting a specific size

We can of course give elements in our design a specific size. When a size is given to an element (and the content of which then needs to fit into that size) we refer to it as an **extrinsic size**. Take our `` from the example above — we can give it specific [`width`](https://developer.mozilla.org/en-US/docs/Web/CSS/width) and [`height`](https://developer.mozilla.org/en-US/docs/Web/CSS/height) values, and it will now have that size no matter what content is placed into it. As we discovered in [our previous lesson on overflow](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Overflowing_content), a set height can cause content to overflow if there is more content than the element has space to fit inside it.



Due to this problem of overflow, fixing the height of elements with lengths or percentages is something we need to do very carefully on the web.

### Using percentages

In many ways percentages act like length units, and as we [discussed in the lesson on values and units](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Values_and_units#Percentages), they can often be used interchangeably with lengths. When using a percentage you need to be aware what it is a percentage *of*. In the case of a box inside another container, if you give the child box a percentage width it will be a percentage of the width of the parent container.



This is because percentages resolve against the size of the containing block. With no percentage applied our `` would take up 100% of the available space, as it is a block level element. If we give it a percentage width, this becomes a percentage of the space it would normally fill.

### Percentage margins and padding

If you set `margins` and `padding` as a percentage you may notice some strange behavior. In the below example we have a box. We have given the inner box a [`margin`](https://developer.mozilla.org/en-US/docs/Web/CSS/margin) of 10% and a [`padding`](https://developer.mozilla.org/en-US/docs/Web/CSS/padding) of 10%. The padding and margin on the top and bottom of the box are the same size as the margin on the left and right.



You might expect for example the percentage top and bottom margins to be a percentage of the element's height, and the percentage left and right margins to be a percentage of the element's width. However, this is not the case!

When you use margin and padding set in percentages, the value is calculated from the **inline size** — therefore the width when working in a horizontal language. In our example, all of the margins and padding are 10% of the width. This means you can have equal sized margins and padding all round the box. This is a fact worth remembering if you do use percentages in this way.

## min- and max- sizes

In addition to giving things a fixed size, we can ask CSS to give an element a minimum or a maximum size. If you have a box that might contain a variable amount of content, and you always want it to be *at least* a certain height, you could set the [`min-height`](https://developer.mozilla.org/en-US/docs/Web/CSS/min-height) property on it. The box will always be at least this height, but will then grow taller if there is more content than the box has space for at its minimum height.

In the example below you can see two boxes, both with a defined height of 150 pixels. The box on the left is 150 pixels tall; the box on the right has content that needs more room, and so it has grown taller than 150 pixels.

This is very useful for dealing with variable amounts of content while avoiding overflow.

A common use of [`max-width`](https://developer.mozilla.org/en-US/docs/Web/CSS/max-width) is to cause images to scale down if there is not enough space to display them at their intrinsic width, while making sure they don't become larger than that width.

As an example, if you were to set `width: 100%` on an image, and its intrinsic width was smaller than its container, the image would be forced to stretch and become larger, causing it to look pixellated. If its intrinsic width were larger than its container, it would overflow it. Neither case is likely to be what you want to happen.

If you instead use `max-width: 100%`, the image is able to become smaller than its intrinsic size, but will stop at 100% of its size.

In the example below we have used the same image twice. The first image has been given `width: 100%` and is in a container which is larger than it, therefore it stretches to the container width. The second image has `max-width: 100%` set on it and therefore does not stretch to fill the container. The third box contains the same image again, also with `max-width: 100%` set; in this case you can see how it has scaled down to fit into the box.

This technique is used to make images *responsive*, so that when viewed on a smaller device they scale down appropriately. You should however not use this technique to load in really large images and then scale them down in the browser. Images should be appropriately sized to be no larger than they need to be for the largest size they are displayed in the design. Downloading overly large images will cause your site to become slow, and it can cost users more money if they are on a metered connection.

**Note**: Find out more about [responsive image techniques](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images).

## Viewport units

The viewport — which is the visible area of your page in the browser you are using to view a site — also has a size. In CSS we have units which relate to the size of the viewport — the `vw` unit for viewport width, and `vh` for viewport height. Using these units you can size something relative to the viewport of the user.

`1vh` is equal to 1% of the viewport height, and `1vw` is equal to 1% of the viewport width. You can use these units to size boxes, but also text. In the example below we have a box which is sized as 20vh and 20vw. The box contains a letter `A`, which has been given a [`font-size`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-size) of 10vh.

If you change the `vh` and `vw` values this will change the size of the box or font; changing the viewport size will also change their sizes because they are sized relative to the viewport. To see the example change when you change the viewport size you will need to load the example in a new browser window that you can resize (as the embedded `<iframe>` that contains the example shown above is its viewport). [Open the example](https://mdn.github.io/css-examples/learn/sizing/vw-vh.html), resize the browser window, and observe what happens to the size of the box and text.

Sizing things according to the viewport can be useful in your designs. For example, if you want a full page hero section to show before the rest of your content, making that part of your page 100vh high will push the rest of the content below the viewport, meaning that it will only appear once the document is scrolled.

## Summary

This lesson has given you a rundown of some key issues that you might run into when sizing things on the web. When you move onto [CSS Layout](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout), sizing will become very important in mastering the different layout methods, so it is worth understanding the concepts here before moving on.

