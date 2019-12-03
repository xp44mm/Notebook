# Images, media, and form elements

In this lesson we will take a look at how certain special elements are treated in CSS. Images, other media, and form elements behave a little differently in terms of your ability to style them with CSS than regular boxes. Understanding what is and isn't possible can save some frustration, and this lesson will highlight some of the main things that you need to know. 

## Replaced elements

Images and video are described as **[replaced elements](https://developer.mozilla.org/en-US/docs/Web/CSS/Replaced_element)**. This means that CSS cannot affect the internal layout of these elements — only their position on the page amongst other elements. As we will see however, there are various things that CSS can do with an image.

Certain replaced elements, such as images and video, are also described as having an **aspect ratio**. This means that it has a size in both the horizontal (x) and vertical (y) dimensions, and will be displayed using the intrinsic dimensions of the file by default.

## Sizing images

As you already know from following these lessons, everything in CSS generates a box. If you place an image inside a box that is smaller or larger than the intrinsic dimensions of the image file in either direction, it will either appear smaller than the box, or overflow the box. You need to make a decision about what happens with the overflow.

In the example below we have two boxes, both 200 pixels in size:

- One contains an image which is smaller than 200 pixels — it is smaller than the box and doesn't stretch to fill it.
- The other is larger than 200 pixels and overflows the box.

So what can we do about the overflowing issue?

As we learned [our previous lesson](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Sizing_items_in_CSS), a common technique is to make the [`max-width`](https://developer.mozilla.org/en-US/docs/Web/CSS/max-width) of an image 100%. This will enable the image to become smaller in size than the box but not larger. This technique will work with other replaced elements such as `<video>`s, or `<iframe>`s.

Try adding `max-width: 100%` to the `<img>` element in the example above. You will see that the smaller image remains unchanged, but the larger one becomes smaller to fit into the box.

You can make other choices about images inside containers. For example, you may want to size an image so it completely covers a box.

The [`object-fit`](https://developer.mozilla.org/en-US/docs/Web/CSS/object-fit) property can help you here. When using `object-fit` the replaced element can be sized to fit a box in a variety of ways.

Below we have used the value `cover`, which sizes the image down, maintaining the aspect ratio so that it neatly fills the box. As the aspect ratio is maintained some parts of the image will be cropped by the box.

If we use `contain` as a value the image will be scaled down until it is small enough to fit inside the box. This will result in "letterboxing" if it is not the same aspect ratio as the box.

You could also try the value of `fill`, which will fill the box but not maintain the aspect ratio.

## Replaced elements in layout

When using various CSS layout techniques on replaced elements, you may well find that they behave slightly differently to other elements. For example, in a flex or grid layout elements are stretched by default to fill the entire area. Images will not stretch, and instead will be aligned to the start of the grid area or flex container.

You can see this happening in the example below where we have a two column, two row grid container, which has four items in it. All of the `<div>` elements have a background color and stretch to fill the row and column. The image, however, does not stretch.

If you are following these lessons in order then you may not have looked at layout yet. Just keep in mind that replaced elements, when they become part of a grid or flex layout, have different default behaviors, essentially to avoid them being stretched strangely by the layout.

To force the image to stretch to fill the grid cell it is in, you'd have to do something like following:

```css
img {
  width: 100%;
  height: 100%;
}
```

This would however stretch the image, so probably isn't what you'd want to do.

## Form elements

Form elements are a tricky issue when it comes to styling with CSS, and the [HTML Forms module](https://developer.mozilla.org/en-US/docs/Learn/HTML/Forms) contains detailed guides to styling form elements, which I will not fully reproduce here. There are a few key things worth highlighting in this section of the article.

Many form controls are added to your page by way of the `<input>` element — this defines simple form fields such as text inputs, through to more complex fields added in HTML5 such as color and date pickers. There are some additional elements, such as `<textarea>` for multiline text input, and also elements used to contain and label parts of forms such as `<fieldset>` and `<legend>`.

HTML5 also contains attributes that enable web developers to indicate which fields are required, and even the type of content that needs to be entered. If the user enters something unexpected, or leaves a required field blank, the browser can show an error message. Different browsers are inconsistent in how much styling and customization they allow for such items.

### Styling text input elements



Elements that allow for text input, such as `<input type="text">`, specific types such as `<input type="email">`, and the `<textarea>` element are quite easy to style and tend to behave just like other boxes on your page. The default styling of these elements will differ however based on the operating system and browser that your user visits the site with.

In the example below we have styled some text inputs using CSS — you can see that things such as borders, margins and padding all apply as you would expect. We are using attribute selectors to target the different input types. Try changing how this form looks by adjusting the borders, adding background colors to the fields, and changing fonts and padding.

```css
input[type="text"],
input[type="email"] {
  border: 2px solid #000;
  margin: 0 0 1em 0;
  padding: 10px;
  width: 100%;
}

input[type="submit"] {
  border: 3px solid #333;
  background-color: #999;
  border-radius: 5px;
  padding: 10px 2em;
  font-weight: bold;
  color: #fff;
}

input[type="submit"]:hover {
  background-color: #333;
}
```

**Important**: You should take care when changing the styling of form elements to make sure it is still obvious to the user they are form elements. You could create a form input with no borders and background that is almost indistinguishable from the content around it, but this would make it very hard to recognise and fill in.

As explained in the lessons on [form styling](https://developer.mozilla.org/en-US/docs/Learn/HTML/Forms/Styling_HTML_forms) in the HTML part of this course, many of the more complex input types are rendered by the operating system and are inaccessible to styling. You should therefore always assume that forms are going to look quite different for different visitors and test complex forms in a number of browsers.

### Inheritance and form elements



In some browsers, form elements do not inherit font styling by default. Therefore if you want to be sure that your form fields use the font defined on the body, or on a parent element, you should add this rule to your CSS.

```css
button, 
input, 
select, 
textarea { 
  font-family : inherit; 
  font-size : 100%; 
} 
```

### Form elements and box-sizing



Across browsers form elements use different box sizing rules for different widgets. You learned about the `box-sizing` property in [our box model lesson](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/The_box_model) and you can use this knowledge when styling forms to ensure a consistent experience when setting widths and heights on form elements.

For consistency it is a good idea to set margins and padding to `0` on all elements, then add these back in when styling particular controls

```css
button, 
input, 
select, 
textarea {  
  box-sizing: border-box; 
  padding: 0;
  margin: 0; 
}
```

### Other useful settings



In addition to the rules mentioned above, you should also set `overflow: auto` on `<textarea>`s to stop IE showing a scrollbar when there is no need for one:

```css
textarea {
  overflow: auto;
}
```

### Putting it all together into a "reset"



As a final step, we can wrap up the various properties discussed above into the following "form reset" to provide a consistent base to work from. This includes all the items mentioned in the last three sections:

```css
button, 
input, 
select, 
textarea { 
  font-family: inherit; 
  font-size: 100%; 
  box-sizing: border-box; 
  padding: 0; margin: 0; 
} 

textarea { 
  overflow: auto; 
} 
```

**Note**: Normalizing stylesheets are used by many developers to create a set of baseline styles to use on all projects. Typically these do similar things to those described above, making sure that anything different across browsers is set to a consistent default before you do your own work on the CSS. They are not as important as they once were, as browsers are typically more consistent than in the past. However if you want to take a look at one example, check out [Normalize.css](http://necolas.github.io/normalize.css/), which is a very popular stylesheet used as a base by many projects.

For further information on styling forms, take a look at the two articles in the HTML section of these guides.

- [Styling HTML Forms](https://developer.mozilla.org/en-US/docs/Learn/HTML/Forms/Styling_HTML_forms)
- [Advanced Styling for HTML Forms](https://developer.mozilla.org/en-US/docs/Learn/HTML/Forms/Advanced_styling_for_HTML_forms)

## Summary

This lesson has highlighted some of the differences you will encounter when working with images, media, and other unusual elements in CSS. In the next article we'll look over a few tips you'll find useful when you have to style HTML tables.