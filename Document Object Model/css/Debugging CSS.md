# Debugging CSS

Sometimes when writing CSS you will encounter an issue where your CSS doesn't seem to be doing what you expect. Perhaps you believe that a certain selector should match an element, but nothing happens, or a box is a different size than you expected. This article will give you guidance on how to go about debugging a CSS problem, and show you how the DevTools included in all modern browsers can help you to find out what is going on.

## How to access browser DevTools

The article [What are browser developer tools](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/What_are_browser_developer_tools) is an up-to-date guide explaining how to access the tools in various browsers and platforms. While you may choose to mostly develop in a particular browser, and therefore will become most familiar with the tools included in that browser, it is worth knowing how to access them in other browsers. This will help if you are seeing different rendering between multiple browsers.

You will also find that browsers have chosen to focus on different areas when creating their DevTools. For example in Firefox there are some excellent tools for working visually with CSS Layout, allowing you to inspect and edit [Grid Layouts](https://developer.mozilla.org/en-US/docs/Tools/Page_Inspector/How_to/Examine_grid_layouts), [Flexbox](https://developer.mozilla.org/en-US/docs/Tools/Page_Inspector/How_to/Examine_Flexbox_layouts), and [Shapes](https://developer.mozilla.org/en-US/docs/Tools/Page_Inspector/How_to/Edit_CSS_shapes). However, all of the different browsers have similar fundamental tools, e.g. for inspecting the properties and values applied to elements on your page, and making changes to them from the editor.

In this lesson we will look at some useful features of the Firefox DevTools for working with CSS. In order to do so I'll be using [an example file](https://mdn.github.io/css-examples/learn/inspecting/inspecting.html). Load this up in a new tab if you want to follow along, and open up your DevTools as described in the article linked above.

## The DOM versus view source

Something that can trip up newcomers to DevTools is the difference between what you see when you [view the source](https://developer.mozilla.org/en-US/docs/Tools/View_source) of a webpage, or look at the HTML file you put on the server, and what you can see in the [HTML Pane](https://developer.mozilla.org/en-US/docs/Tools/Page_Inspector/UI_Tour#HTML_pane) of the DevTools. While it looks roughly similar to what you can see via View Source there are some differences.

In the rendered DOM the browser may have corrected some badly-written HTML for you. If you incorrectly closed an element, for instance opening an `<h2>` but closing with an `<h3>`, the browser will figure out what you were meaning to do and the HTML in the DOM will correctly close the open `<h2>` with an `</h2>`. The browser will also normalize all of the HTML, and the DOM will also show any changes made by JavaScript.

View Source in comparison, is simply the HTML source code as stored on the server. The [HTML tree](https://developer.mozilla.org/en-US/docs/Tools/Page_Inspector/How_to/Examine_and_edit_HTML#HTML_tree) in your DevTools shows exactly what the browser is rendering at any given time, so it gives you an insight into what is really going on.

## Inspecting the applied CSS

Select an element on your page, either by right/ctrl-clicking on it and selecting *Inspect*, or selecting it from the HTML tree on the left of the DevTools display. Try selecting the element with the class of `box1`; this is the first element on the page with a bordered box drawn around it.

![The example page for this tutorial with DevTools open.](https://mdn.mozillademos.org/files/16606/inspecting1.png)

If you look at the [Rules view](https://developer.mozilla.org/en-US/docs/Tools/Page_Inspector/UI_Tour#Rules_view) to the right of your HTML, you should be able to see the CSS properties and values applied to that element. You will see the rules directly applied to class `box1` and also the CSS that is being inherited by the box from its ancestors, in this case to `<body>`. This is useful if you are seeing some CSS being applied that you didn't expect. Perhaps it is being inherited from a parent element and you need to add a rule to overwrite it in the context of this element.

Also useful is the ability to expand out shorthand properties. In our example the `margin` shorthand is used.

Click on the little arrow to expand the view, showing the different longhand properties and their values.

You can toggle values in the Rules view on and off, when that panel is active — if you hold your mouse over it checkboxes will appear. Uncheck a rule's checkbox, for example `border-radius`, and the CSS will stop applying.

You can use this to do an A/B comparison, deciding if something looks better with a rule applied or not, and also to help debug it — for example if a layout is going wrong and you are trying to work out which property is causing the problem.

## Editing values

In addition to turning properties on and off, you can edit their values. Perhaps you want to see if another color looks better, or wish to tweak the size of something? DevTools can save you a lot of time editing a stylesheet and reloading the page.

With `box1` selected, click on the swatch (the small colored circle) that shows the color applied to the border. A color picker will open up and you can try out some different colors; these will update in real time on the page. In a similar fashion, you could change the width or style of the border.

![DevTools Styles Panel with a color picker open.](https://mdn.mozillademos.org/files/16607/inspecting2-color-picker.png)

## Adding a new property

You can add properties using the DevTools. Perhaps you have realised that you don't want your box to inherit the `` element's font size, and want to set its own specific size? You can try this out in DevTools before adding it to your CSS file.

You can click the closing curly brace in the rule to start entering a new declaration into it, at which point you can start typing the new property and DevTools will show you an autocomplete list of matching properties. After selecting `font-size`, enter the value you want to try. You can also click the + button to add an additional rule with the same selector, and add your new rules there.

![The DevTools Panel, adding a new property to the rules, with the autocomplete for font- open](https://mdn.mozillademos.org/files/16608/inspecting3-font-size.png)

**Note**: There are other useful features in the Rules view too, for example declarations with invalid values are crossed out. You can find out more at [Examine and edit CSS](https://developer.mozilla.org/en-US/docs/Tools/Page_Inspector/How_to/Examine_and_edit_CSS).

## Understanding the box model

In previous lessons we have discussed [the Box Model](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/The_box_model), and the fact that we have an alternate box model that changes how the size of elements are calculated based on the size you give them, plus the padding and borders. DevTools can really help you to understand how the size of an element is being calculated.

The [Layout view](https://developer.mozilla.org/en-US/docs/Tools/Page_Inspector/UI_Tour#Layout_view) shows you a diagram of the box model on the selected element, along with a description of the properties and values that change how the element is laid out. This includes a description of properties that you may not have explicitly used on the element, but which do have initial values set.

In this panel, one of the detailed properties is the `box-sizing` property, which controls what box model the element uses.

Compare the two boxes with classes `box1` and `box2`. They both have the same width applied (400px), however `box1` is visually wider. You can see in the layout panel that it is using `content-box`. This is the value that takes the size you give the element and then adds on the padding and border width.

The element with a class of `box2` is using `border-box`, so here the padding and border is subtracted from the size that you have given the element. This means that the space taken up on the page by the box is the exact size that you specified — in our case `width: 400px`.

![The Layout section of the DevTools](https://mdn.mozillademos.org/files/16609/inspecting4-box-model.png)

**Note**: Find out more in [Examining and Inspecting the Box Model](https://developer.mozilla.org/en-US/docs/Tools/Page_Inspector/How_to/Examine_and_edit_the_box_model).

## Solving specificity issues

Sometimes during development, but in particular when you need to edit the CSS on an existing site, you will find yourself having a hard time getting some CSS to apply. No matter what you do, the element just doesn't seem to take the CSS. What is generally happening here is that a more specific selector is overriding your changes, and here DevTools will really help you out.

In our example file there are two words that have been wrapped in an `` element. One is displaying as orange and the other hotpink. In the CSS we have applied:

```css
em {
  color: hotpink;
  font-weight: bold;
}
```

Above that in the stylesheet however is a rule with a `.special` selector:

```css
.special {
  color: orange;
}
```

As you will recall from the lesson on [cascade and inheritance](https://developer.mozilla.org/en-US/docs/Learn/CSS/Building_blocks/Cascade_and_inheritance) where we discussed specificity, class selectors are more specific than element selectors, and so this is the value that applies. DevTools can help you find such issues, especially if the information is buried somewhere in a huge stylesheet.

Inspect the `<em>` with the class of `.special` and DevTools will show you that orange is the color that applies, and also shows you the `color` property applied to the em crossed out. You can now see that the class is overriding the element selector.

![Selecting an em and looking at DevTools to see what is over-riding the color.](https://mdn.mozillademos.org/files/16610/inspecting5-specificity.png)

## Find out more about the Firefox DevTools

There is a lot of information about the Firefox DevTools here on MDN. Take a look at the main [DevTools section](https://developer.mozilla.org/en-US/docs/Tools), and for more detail on the things we have briefly covered in this lesson see [The How To Guides](https://developer.mozilla.org/en-US/docs/Tools/Page_Inspector#How_to).

## Debugging problems in CSS

DevTools can be a great help when solving CSS problems, so when you find yourself in a situation where CSS isn't behaving as you expect, how should you go about solving it? The following steps should help.

### Take a step back from the problem



Any coding problem can be frustrating, especially CSS problems because you often don't get an error message to search for online to help with finding a solution. If you are becoming frustrated, take a step away from the issue for a while — go for a walk, grab a drink, chat to a co-worker, or work on some other thing for a while. Sometimes the solution magically appears when you stop thinking about the problem, and even if not, working on it when feeling refreshed will be much easier.

### Do you have valid HTML and CSS?



Browsers expect your CSS and HTML to be correctly written, however browsers are also very forgiving and will try their best to display your webpages even if you have errors in the markup or stylesheet. If you have mistakes in your code the browser needs to make a guess at what you meant, and it might make a different decision to what you had in mind. In addition, two different browsers might cope with the problem in two different ways. A good first step therefore is to run your HTML and CSS through a validator, to pick up and fix any errors.

- [CSS Validator](https://jigsaw.w3.org/css-validator/)
- [HTML validator](https://validator.w3.org/)

### Is the property and value supported by the browser you are testing in?



Browsers simply ignore CSS they don't understand. If the property or value you are using is not supported by the browser you are testing in then nothing will break, but that CSS won't be applied. DevTools will generally highlight unsupported properties and values in some way. In the screenshot below the browser does not support the subgrid value of [`grid-template-columns`](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template-columns).

![Image of browser DevTools with the grid-template-columns: subgrid crossed out as the subgrid value is not supported.](https://mdn.mozillademos.org/files/16641/no-support.png)

You can also take a look at the Browser compatibility tables at the bottom of each property page on MDN. These show you browser support for that property, often broken down if there is support for some usage of the property and not others. The below table shows the compat data for the [`shape-outside`](https://developer.mozilla.org/en-US/docs/Web/CSS/shape-outside) property.

No compatibility data found. Please contribute data for "css.shape-outside" (depth: 1) to the [MDN compatibility data repository](https://github.com/mdn/browser-compat-data).

### Is something else overriding your CSS?



This is where the information you have learned about specificity will come in very useful. If you have something more specific overriding what you are trying to do, you can enter into a very frustrating game of trying to work out what. However, as described above, DevTools will show you what CSS is applying and you can work out how to make the new selector specific enough to override it.

### Make a reduced test case of the problem



If the issue isn't solved by the steps above, then you will need to do some more investigating. The best thing to do at this point is to create something known as a reduced test case. Being able to "reduce an issue" is a really useful skill. It will help you find problems in your own code and that of your colleagues, and will also enable you to report bugs and ask for help more effectively.

A reduced test case is a code example that demonstrates the problem in the simplest possible way, with unrelated surrounding content and styling removed. This will often mean taking the problematic code out of your layout to make a small example which only shows that code or feature.

To create a reduced test case:

1. If your markup is dynamically generated — for example via a CMS — make a static version of the output that shows the problem. A code sharing site like [CodePen](https://codepen.io/) is useful for hosting reduced test cases, as then they are accessible online and you can easily share them with colleagues. You could start by doing View Source on the page and copying the HTML into CodePen, then grab any relevant CSS and JavaScript and include it too. After that, you can check whether the issue is still evident.
2. If removing the JavaScript does not make the issue go away, don't include the JavaScript. If removing the JavaScript *does* make the issue go away, then remove as much JavaScript as you can, leaving in whatever causes the issue.
3. Remove any HTML that does not contribute to the issue. Remove components or even main elements of the layout. Again, try to get down to the smallest amount of code that still shows the issue.
4. Remove any CSS that doesn't impact the issue.

In the process of doing this, you may discover what is causing the problem, or at least be able to turn it on and off by removing something specific. It is worth adding some comments to your code as you discover things. If you need to ask for help, they will show the person helping you what you have already tried. This may well give you enough information to be able to search for likely sounding problems and workarounds.

If you are still struggling to fix the problem then having a reduced test case gives you something to ask for help with, by posting to a forum, or showing to a co-worker. You are much more likely to get help if you can show that you have done the work of reducing the problem and identifying exactly where it happens, before asking for help. A more experienced developer might be able to quickly spot the problem and point you in the right direction, and even if not, your reduced test case will enable them to have a quick look and hopefully be able to offer at least some help.

In the instance that your problem is actually a bug in a browser, then a reduced test case can also be used to file a bug report with the relevant browser vendor (e.g. on Mozilla's [bugzilla site](https://bugzilla.mozilla.org/)).

As you become more experienced with CSS, you will find that you get faster at figuring out issues. However even the most experienced of us sometimes find ourselves wondering what on earth is going on. Taking a methodical approach, making a reduced test case, and explaining the issue to someone else will usually result in a fix being found.

# Organizing your CSS

As you start to work on larger stylesheets and big projects you will discover that maintaining a huge CSS file can be challenging. In this article we will take a brief look at some best practices for writing your CSS to make it easily maintainable, and some of the solutions you will find in use by others to help improve maintainability.

## Tips to keep your CSS tidy

Here are some general suggestions for ways to keep your stylesheets organised and tidy.

### Does your project have a coding style guide?



If you are working with a team on an existing project, the first thing to check is whether the project has an existing style guide for CSS. The team style guide should always win over your own personal preferences. There often isn't a right or wrong way to do things, but consistency is important.

For example, have a look at the [CSS guidelines for MDN code examples](https://developer.mozilla.org/en-US/docs/MDN/Contribute/Guidelines/Code_guidelines/CSS).

### Keep it consistent



If you get to set the rules for the project or are working alone, then the most important thing to do is to keep things consistent. Consistency can be applied in all sorts of ways, such as using the same naming conventions for classes, choosing one method of describing color, or maintaining consistent formatting (for example will you use tabs or spaces to indent your code? If spaces, how many spaces?)

Having a set of rules you always follow reduces the amount of mental overhead needed when writing CSS, as some of the decisions are already made.

### Formatting readable CSS



There are a couple of ways you will see CSS formatted. Some developers put all of the rules onto a single line, like so:

```css
.box { background-color: #567895; }
h2 { background-color: black; color: white; }
```

Other developers prefer to break everything onto a new line:

```css
.box {
  background-color: #567895;
}

h2 {
  background-color: black;
  color: white;
}
```

CSS doesn't mind which one you use. We personally find it is more readable to have each property and value pair on a new line.

### Comment your CSS



Adding comments to your CSS will help any future developer work with your CSS file, but will also help you when you come back to the project after a break.

```css
/* This is a CSS comment
It can be broken onto multiple lines. */
```

A good tip is to add a block of comments between logical sections in your stylesheet too, to help locate different sections quickly when scanning through, or even give you something to search for to jump right into that part of the CSS. If you use a string which won't appear in the code you can jump from section to section by searching for it — below we have used `||`.

```css
/* || General styles */

...

/* || Typography */

...

/* || Header and Main Navigtion */

...
```

You don't need to comment every single thing in your CSS, as much of it will be self-explanatory. What you should comment are the things where you made a particular decision for a reason.

You may have used a CSS property in a specific way to get around older browser incompatibilities, for example:

```css
.box {
  background-color: red; /* fallback for older browsers that don't support gradients */
  background-image: linear-gradient(to right, #ff0000, #aa0000);
}
```

Perhaps you followed a tutorial to achieve something, and the CSS is a little non-obvious. In that case you could add the URL of the tutorial to the comments. You will thank yourself when you come back to this project in a year or so, and can vaguely remember there was a great tutorial about that thing, but where is it?

### Create logical sections in your stylesheet



It is a good idea to have all of the common styling first in the stylesheet. This means all of the styles which will generally apply unless you do something special with that element. You will typically have rules set up for:

- `body`
- `p`
- `h1`, `h2`, `h3`, `h4`, `h5`
- `ul` and `ol`
- The `table` properties
- Links

In this section of the stylesheet we are providing default styling for the type on the site, setting up a default style for data tables and lists and so on.

```css
/* || GENERAL STYLES */

body { ... }

h1, h2, h3, h4 { ... }

ul { ... }

blockquote { ... }
```

After this section we could define a few utility classes, for example a class that removes the default list style for lists we're going to display as flex items or in some other way. If you have a few things you know you will want to apply to lots of different elements, they can come in this section.

```css
/* || UTILITIES */

.nobullets {
  list-style: none;
  margin: 0;
  padding: 0;
}

...
```

Then we can add everything that is used sitewide. That might be things like the basic page layout, the header, navigation styling, and so on.

```css
/* || SITEWIDE */

.main-nav { ... }

.logo { ... }
```

Finally we will include CSS for specific things, broken down by the context, page or even component in which they are used.

```css
/* || STORE PAGES */

.product-listing { ... }

.product-box { ... }
```

By ordering things in this way, we at least have an idea in which part of the stylesheet we will be looking for something that we want to change.

### Avoid overly-specific selectors



If you create very specific selectors you will often find that you need to duplicate chunks of your CSS to apply the same rules to another element. For example, you might have something like the below selector, which applies the rule to a `<p>` with a class of `box` inside an `<article>` with a class of `main`.

```css
article.main p.box {
  border: 1px solid #ccc;
}
```

If you then wanted to apply the same rules to something outside of `main`, or to something other than a `<p>`, you would have to add another selector to these rules or create a whole new ruleset. Instead, you could create a class called `box` and apply that anywhere.

```css
.box {
  border: 1px solid #ccc;
}
```

There will be times when making something more specific makes sense, however this will generally be an exception rather than usual practice.

### Break large stylesheets into multiple smaller ones



In particular in cases where you have very different styles for distinct parts of the site, you might want to have a stylesheet that includes all the global rules and then smaller ones that include the specific rules needed for those sections. You can link to multiple stylesheets from one page, and the normal rules of the cascade apply, with rules in stylesheets linked later coming after rules in stylesheets linked earlier.

For example, we might have an online store as part of the site, with a lot of CSS used only for styling the product listings and forms needed for the store. It would make sense to have those things in a different stylesheet, only linked to on store pages.

This can make it easier to keep your CSS organised, and also means that if multiple people are working on the CSS you will have fewer situations where two people need to work on the same stylesheet at once, leading to conflicts in source control.

## Other tools that can help

CSS itself doesn't have much in the way of in-built organisation, therefore you need to do the work to create consistency and rules around how you write CSS. The web community has also developed various tools and approaches that can help you to manage larger CSS projects. As they may be helpful for you to investigate, and you are likely to come across these things when working with other people, we've included a short guide to some of these.

### CSS methodologies



Instead of needing to come up with your own rules for writing CSS, you may benefit from adopting one of the approaches already designed by the community and tested across many projects. These methodologies are essentially CSS coding guides that take a very structured approach to writing and organising CSS. Typically they tend to result in more verbose use of CSS than you might have if you wrote and optimised every selector to a custom set of rules for that project.

However, you do gain a lot of structure by adopting one and, as many of these systems are very widely used, other developers are more likely to understand the approach you are using and be able to write their CSS in the same way, rather than having to work out your own personal methodology from scratch.

#### OOCSS

Most of the approaches that you will encounter owe something to the concept of Object Oriented CSS (OOCSS), an approach made popular by [the work of Nicole Sullivan](https://github.com/stubbornella/oocss/wiki). The basic idea of OOCSS is to separate your CSS into reusable objects, which can be used anywhere you need on your site. The standard example of OOCSS is the pattern described as [The Media Object](https://developer.mozilla.org/en-US/docs/Web/CSS/Layout_cookbook/Media_objects). This is a pattern with a fixed size image, video or other element on one side, and flexible content on the other. It's a pattern we see all over websites for comments, listings, and so on.

If you are not taking an OOCSS approach you might create custom CSS for the different places this pattern is used, for example creating a class called `comment` with a bunch of rules for the component parts, then a class called `list-item` with almost the same rules as the `comment` class except for some tiny differences. The differences between these two components is that the list-item has a bottom border, and images in comments have a border whereas list-item images do not.

```css
.comment {
  display: grid;
  grid-template-columns: 1fr 3fr;
}

.comment img {
  border: 1px solid grey;
}

.comment .content {
  font-size: .8rem;
}

.list-item {
  display: grid;
  grid-template-columns: 1fr 3fr;
  border-bottom: 1px solid grey;
}

.list-item .content {
  font-size: .8rem;
}
```

In OOCSS, you would create one pattern called `media` that would have all of the common CSS for both patterns — a base class for things that are generally the shape of the media object. Then we'd add an additional class to deal with those tiny differences, thus extending that styling in specific ways.

```css
.media {
  display: grid;
  grid-template-columns: 1fr 3fr;
}

.media .content {
  font-size: .8rem;
}

.comment img {
  border: 1px solid grey;
}

 .list-item {
  border-bottom: 1px solid grey;
} 
```

In your HTML the comment would need both the `media` and `comment` classes applied:

```html
<div class="media comment">
  <img />
  <div class="content"></div>
</div>
```

The list-item would have `media` and `list-item` applied:

```html
<ul>
  <li class="media list-item">
    <img />
   <div class="content"></div>
  </li>
</ul>
```

The work that Nicole Sullivan did in describing this approach and promoting it means that even people who are not strictly following an OOCSS approach today will generally be reusing CSS in this way — it has entered our understanding as a good way to approach things in general.

#### BEM

BEM stands for Block Element Modifier. In BEM a block is a standalone entity such as a button, menu, or logo. An element is something like a list item or a title that is tied to the block it is in. A modifier is a flag on a block or element that changes the styling or behavior. You will be able to recognise code that uses BEM due to the extensive use of dashes and underscores in the CSS classes. For example, look at the classes applied to this HTML from the page about [BEM Naming conventions](http://getbem.com/naming/):

```html
<form class="form form--theme-xmas form--simple">
  <input class="form__input" type="text" />
  <input
    class="form__submit form__submit--disabled"
    type="submit" />
</form>
```

The additional classes are similar to those used in the OOCSS example, however they use the strict naming conventions of BEM.

BEM is widely used in larger web projects and many people write their CSS in this way. It is likely that you will come across examples, even in tutorials, that use BEM syntax, without mentioning why the CSS is structured in such a way.

To read more about the system read [BEM 101](https://css-tricks.com/bem-101/) on CSS Tricks.

#### Other common systems

There are a large number of these systems in use. Other popular approaches include [Scalable and Modular Architecture for CSS (SMACSS)](http://smacss.com/), created by Jonathan Snook, [ITCSS](https://itcss.io/) from Harry Roberts, and [Atomic CSS (ACSS)](https://acss.io/), originally created by Yahoo!. If you come across a project that uses one of these approaches then the advantage is that you will be able to search and find many articles and guides to help you understand how to code in the same style.

The disadvantage of using such a system is that they can seem overly complex, especially for smaller projects.

### Build systems for CSS



Another way to organise CSS is to take advantage of some of the tooling that is available for front-end developers, which allows you to take a slightly more programmatic approach to writing CSS. There are a number of tools which we refer to as *pre-processors* and *post-processors*. A pre-processor runs over your raw files and turns them into a stylesheet, whereas a post-processor takes your finished stylesheet and does something to it — perhaps to optimize it in order that it will load faster.

Using any of these tools will require that your development environment can run the scripts that do the pre and post-processing. Many code editors can do this for you, or you can install command line tools to help.

The most popular pre-processor is [Sass](https://sass-lang.com/). This is not a Sass tutorial, so I will briefly explain a couple of the things that Sass can do, which are really helpful in terms of organisation, even if you don't use any of the other Sass features.

#### Defining variables

CSS now has native [custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties), making this feature increasingly less important, however one of the reasons you might use Sass is to be able to define all of the colors and fonts used in a project as settings, then use that variable around the project. This means that if you realise you have used the wrong shade of blue, you only need change it in one place.

If we created a variable called `$base-color` as in the first line below, we could then use it through the stylesheet anywhere that required that color.

```
$base-color: #c6538c;

.alert {
  border: 1px solid $base-color;
}
```

Once compiled to CSS, you would end up with the following CSS in the final stylesheet.

```
.alert { 
  border: 1px solid #c6538c; 
}
```

#### Compiling component stylesheets

I mentioned above that one way to organise CSS is to break down stylesheets into smaller stylesheets. When using Sass you can take this to another level and have lots of very small stylesheets — even going as far as having a separate stylesheet for each component. By using the include functionality in Sass these can then all be compiled together into one, or a small number of stylesheets to actually link into your website.

You can see how one developer approaches the problem in [this blog post](https://www.lauraleeflores.com/blog/how-to-organize-your-css-files).

**Note**: A simple way to try out Sass is to use [CodePen](https://codepen.io/) — you can enable Sass for your CSS in the Settings for a Pen, and CodePen will then run the Sass parser for you, in order that you can see the resulting webpage with regular CSS applied. Sometimes you will find that CSS tutorials have used Sass rather than plain CSS in their CodePen demos, so it is handy to know a little bit about it.

#### Post-processing for optimization

If you are concerned about adding size to your stylesheets by adding a lot of additional comments and whitespace for example, then a post-processing step could be to optimize the CSS by stripping out anything unnecessary in the production version. An example of a post-processor solution for doing this would be [cssnano](https://cssnano.co/).

## Wrapping up

This is the final part of our Learning CSS Guide, and as you can see there are many ways in which your exploration of CSS can continue from this point.

To learn more about layout in CSS, see the [Learn CSS Layout](https://developer.mozilla.org/en-US/docs/Learn/CSS/CSS_layout) section.

You should also now have the skills to explore the rest of the [MDN CSS](https://developer.mozilla.org/en-US/docs/Web/CSS) material. You can look up properties and values, explore our [CSS Cookbook](https://developer.mozilla.org/en-US/docs/Web/CSS/Layout_cookbook) for patterns to use, and read more in some of the specific guides such as our [Guide to CSS Grid Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout).