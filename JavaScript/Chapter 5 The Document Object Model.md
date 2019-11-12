# Chapter 5 The Document Object Model

Working with the Document Object Model (the DOM) is a critical component of the professional JavaScript programmer's toolkit. A comprehensive understanding of DOM scripting yields benefits not only in the range of applications we can build, but also in the quality of those applications. Like most features of JavaScript, the DOM has a somewhat checkered history. But with modern browsers, it is easier than ever to manipulate and interact with the DOM unobtrusively. Understanding how to use this technology and how best to wield it can give you a head start toward developing your next web application. In this chapter we discuss a number of topics related to the DOM. For readers new to the DOM, we will start out with the basics and move through all the important concepts. For those of you already familiar with the DOM, we provide a number of cool techniques that we are sure you will enjoy and start using in your own web pages.

The DOM is also at a crossroads. Historically, because DOM interface updates were not in sync with browser or JavaScript updates, there was a disconnect between browsers and DOM support. This disconnect was only exacerbated by buggy implementations. Popular libraries like jQuery and Dojo have arisen to address these problems. But with modern browsers, the DOM has normalized and the interface has settled quite a bit. We will need to address the issue of whether to use libraries to help in our access of the DOM or to do everything with the standard DOM interface.

## An Introduction to the Document Object Model

Initially, the DOM was created as a way to represent parts of an HTML document within a browser. Using JavaScript, a developer could look at forms, anchors, images and other components of the page, but not necessarily the entire page. This is sometimes referred to as the “legacy DOM” or DOM Level 0. Eventually, the DOM changed into an interface, overseen by the W3C. From humble beginnings, the DOM has become the official interface not only to HTML documents but also to XML documents. It is not necessarily the fastest, lightest, or easiest-to-use interface, but it is the most ubiquitous, with an implementation existing in most programming languages (such as Java, Perl, PHP, Ruby, Python, and, of course, JavaScript). As we work with the DOM interface, remember that just about everything you learn can be applied to HTML and XML, even though most of the time we will refer exclusively to HTML.

The World Wide Web Consortium oversees the DOM specification. For a variety of historical reasons, versions of the DOM specification are identified as DOM Level n. The current specification (as of publication time) is DOM Level 4. This can sometimes be confusing, as a DOM tree itself can have levels. We will endeavor to refer to versions of the DOM specification as DOM Level (with a capital L) and then DOM tree levels with a lowercase l.

Before anything else, we should quickly discuss the structure of HTML documents. Because this is a book on JavaScript, not HTML, we will focus on the effects of an HTML document on our JavaScript. Let us set forth a few simple principles:

  1. Our HTML documents should start with an HTML 5 doctype. They are very simple: `<!DOCTYPE html>`. Including the doctype prevents browsers from falling into quirks mode, where the behavior of the browser is less consistent.

  2. We should prefer including separate JavaScript files via `<script>` tags over script blocks or in-line scripts. This makes for easier development (separating our JavaScript from our HTML) and easier management. There are rare cases where script blocks make more sense than included files. But as a general rule, prefer included files.

  3. Our script tags should appear at the bottom of the HTML document, immediately before the closing `</body>` tag.

The third item requires some explanation. By having our `<script>` includes at the bottom of the page, we gain several advantages. The majority (if not all) of our HTML should have loaded (as well as associated files: images, audio, video, CSS, and so on). Why is this important? Processing JavaScript code locks up the rendering of a page! Browsers cannot render other page elements while JavaScript code is being parsed (and sometimes when JavaScript code is running!). Therefore, we should wait until the last minute to load our JavaScript code wherever possible. Also, in mobile or slow-connection scenarios, fetching and loading JavaScript can be slower than on a desktop browser. Early loading of the rest of the page means your users are not stuck looking at a blank page with nothing but a spinner. The principle here is that the user should be able to see feedback that some of the page has loaded as soon as possible.

Right. So what does this ideal HTML page look like? Check out Listing 5-1.

Listing 5-1. A Sample HTML File

```html
<!DOCTYPE html>
<html>
<head>
  <title>Introduction to the DOM</title>
</head>
<body>
  <h1>Introduction to the DOM</h1>

  <p id="intro" class="test">
    There are a number of reasons why the DOM is awesome; here are
    some:
  </p>
  <ul id="items">
    <li id="everywhere">It can be found everywhere.</li>
    <li class="test">It's easy to use.</li>
    <li class="test">It can help you to find what you want, really quickly.</li>
  </ul>
  <script src="01-sample.js"></script>
</body>
</html>
```

Sometimes, in our examples, we will have an in-line script block. If so, it will appear within the page as appropriate to its functionality. If its location is irrelevant to its functionality, we will have the script block at the bottom of the page, much like our script includes.

### DOM Structure

The structure of an HTML document is represented in the DOM as a navigable tree. All the terminology used is akin to that of a genealogical tree (parents, children, siblings, and so on). For our purposes, the trunk of the tree is the `Document` node, also known as the document element. This element contains pointers to its children and, in turn, each child node then contains pointers back to its parent, its fellow siblings, and its children. The DOM uses particular terminology to refer to the different objects within the HTML tree. Just about everything in a DOM tree is a node: HTML elements are nodes, the text within elements is a node, comments are nodes, the DOCTYPE is a node, and even attributes are nodes! Obviously, we will need to be able to differentiate among these nodes, so each node has a node type property called, appropriately, `nodeType` (Table 5-1). We can query this property to figure out which type of node we are looking at. If you get a reference to a node, it will be an instance of a `Node` type, implementing all of the methods and having all of the properties of that type.

Table 5-1. Node Types and Their Constant Values

```
  ELEMENT_NODE                1
  TEXT_NODE                   3
  PROCESSING_INSTRUCTION_NODE 7
  COMMENT_NODE                8
  DOCUMENT_NODE               9
  DOCUMENT_TYPE_NODE          10
  DOCUMENT_FRAGMENT_NODE      11

  (deprecated)
  ATTRIBUTE_NOTE              2
  CDATA_SECTION_NODE          4
  ENTITY_REFERENCE_NODE       5
  ENTITY_NODE                 6
  NOTATION_NODE               12
```

Node types marked deprecated are superseded and may be removed, but it's very unlikely. They probably still work, as they have been in use for a few years now.

As you can see in the table, nodes have various specializations, which correspond to interfaces in the DOM specification. Of particular interest are documents, elements, attributes, and text. Each of these has its own implementing type: `Document`, `Element`, `Attr`, and `Text`, respectively.

 ■ Note attributes are a special case. under Dom levels 1, 2, and 3, the `attr` interface implemented the node interface. this is **no longer** true for Dom level 4. thankfully, this is more of a common-sense change than anything else. more details can be found in the section on attributes.

In general, the `Document` is concerned with managing the HTML document as a whole. Each tag within that document is an `Element`, which is itself specialized into specific HTML element types (for example, `HTMLLIElement`, `HTMLFormElement`). The attributes of an element are represented as instances of `Attr`. Any plain text within an `Element` is a text node, represented by the `Text` type. These subtypes are not all of the types that inherit from `Node`, but they are the ones we are most likely to interact with.

Given our listing, let's look at the structure: the entire document, from `<!DOCTYPE html>` to `</html>` is the `Document`. The doctype is itself an instance of the `Doctype` type. The `<html>...</html>` element is our first and main `Element`. It contains child Elements for the `<head>` and `<body>` tags. Diving a little more deeply, we can see that the `<p>` element within the `<body>` has two attributes, id and class. That same `<p>` element has a single `Text` node for its content. The hierarchical structure of the document is duplicated in the relationships between the instances of the various DOM types. We should look at these relationships in greater detail.

### DOM Relationships

Let's examine a very simple document fragment, to show the various relationships between nodes:

```html
<p><strong>Hello</strong> how are you doing?</p>
```

Each portion of this snippet breaks down into a DOM node with pointers from each node pointing to its direct relatives (parents, children, siblings). If you were to completely map out the relationships that exist, it would look something like Figure 5-1. Each portion of the snippet (rounded boxes represent elements, regular boxes represent text nodes) is displayed along with its available references.

```
[p]-firstChild->[strong]
[p]-lastChild->[ how are you doing?]
[strong]-firstChild lastChild->[Hello]
[strong]-nextSibling->[ how are you doing?]
[strong]-parentNode->[p]
[Hello]-parentNode[strong]
[ how are you doing?]-previousSibling->[strong]
[ how are you doing?]-parentNode->[p]
```

Figure 5-1. Relationships between nodes

Every single DOM node contains a collection of pointers that it can use to refer to its relatives. You'll be using these pointers to learn how to navigate the DOM. All the available pointers are displayed in Figure 5-2. Each of these properties, available on every DOM node, is a pointer to another Node or subclass thereof. The only exception is `childNodes` (a collection of all of the child nodes of the current node). And, of course, if one of these relationships is undefined, the value of the property will be null (think of an `<img>` tag, which will have neither `firstChild` nor `lastChild` defined).

```
DOM Node:
  parentNode
  previousSibling
  nextSibling
  firstChild
  lastChild
  childNodes
```

Figure 5-2. Navigating the DOM tree using pointers

Using nothing but the different pointers, it's possible to navigate to any element or text block on a page. Recall Listing 5-1, which showed a typical HTML page. Before, we looked at it from the perspective of JavaScript types. Now we will look at it from the perspective of the DOM.

In the example document, the document node is the `<html>` element. Accessing this element is trivial in JavaScript: `document.documentElement` refers directly to the `<html>` element. The root node has all the pointers used for navigation, just like any other DOM node. Using these pointers you have the ability to start browsing the entire document, navigating to any element that you desire. For example, to get the `<h1>` element, you could use the following:

```js
// Does not work! == null
document.documentElement.firstChild.nextSibling.firstChild
document.documentElement.firstChild // <head>
document.documentElement.firstChild.nextSibling // #text
```

But we have just hit a major snag: The DOM pointers can point to both text nodes and elements. Our JavaScript statement doesn't actually point to the `<h1>` element; it points to the `<title>` element instead. Why did this happen? It happened because of one of the stickiest and most-debated aspects of XML: white space. If you will notice in Listing 5-1, between the `<html>` and `<head>` elements there is actually an end line, which is considered white space, which means that there's actually a text node first, not the `<head>` element. We can learn four things from this:

• Writing nice, clean HTML markup can actually make things very confusing when attempting to browse the DOM using nothing but pointers.

• Using nothing but DOM pointers to navigate a document can be very verbose and impractical.

• In fact, DOM pointers are clearly quite brittle, as they tie your JavaScript logic to your HTML entirely too closely.

• Frequently, you don't need to access text nodes directly, only the elements that surround them.

This leads us to a question: Is there a better way to find elements in a document? Yes, there is! More accurately: there are! We have two major approaches available for accessing elements within the page. On the one hand, we could continue down the line of relative access, sometimes known as **DOM traversal**. For the reasons just listed, we will avoid this approach for general DOM access. We will revisit DOM traversal later on, though, when we have a better handle on accessing specific elements. Instead, we will take a second path, focusing on the various element retrieval functions provided with the modern DOM interface.

## Accessing DOM Elements

All modern DOM implementations contain several methods that make it easy to find elements within the page. Using these methods together with some custom functions can make navigating the DOM a much smoother experience. To start with, let's look at how we can access a single element:

`document.getElementById('everywhere')`: This method, which can only be run on the document object, finds all elements that have an ID equal to everywhere. This is a very powerful function and is the fastest way to access an element immediately.

The `getElementById` method returns a reference to the HTML element with the supplied ID, or null otherwise. The returned object is specifically an instance of the `Element` type. We will discuss what we can do with this `Element` soon.

 ■ Caution `getElementById` works as you would imagine with html documents: it looks through all elements and finds the one single element that has an attribute named id with the specified value. however, if you are loading in a remote Xml document and using `getElementById` (or using a Dom implementation in any language other than javaScript), it doesn't use the id attribute by default. this is by design; an Xml document must explicitly specify what the id attribute is, generally using an Xml definition or a schema.

Let's continue our tour of element-accessing functions. The next two functions provide access to collections of elements:

`getElementsByTagName('li')`: This method, which can be run on any element, finds all descendent elements that have a tag name of `li` and returns them as a live `NodeList` (which is nearly identical to an array).

`getElementsByClassName('test')`: Similar to `getElementsByTagName`, this method can be run from any instance of `Element`. It returns a live `HTMLCollection` of matching elements.

These two functions allow us to access multiple elements at once. Putting aside the difference in return type for a moment, the collection returned is live. This means that if the DOM is modified, and those modifications would be included in the collection (or would remove elements from the collection), the collection will automatically update with those changes. Very powerful!

It is odd that these two methods, similar in function, return two different types. First, let's consider the simple parts: Both types have array-like positional access. That is, for the following:

```js
const lis = document.getElementsByTagName('li');
```

you can access the second list item in the lis collection via `lis[1]`. Both collections have a `length` property, which tells you how many items are in the collection. They also have an `item` method, which takes as its argument the position to access and returns the element at that position. The `item` method is a functional way to access elements positionally. Finally, neither collection has any of the higher-order Array methods, like `push`, `pop`, `map`, or `filter`.

If you would like to use Array methods on your `HTMLCollection` or `NodeList`, you can always use them as shown in Listing 5-2.

Listing 5-2.  Array Functions on `NodeLists`/`HTMLCollections`

```js
// A simple filtering function
// An Element's nodeName property is always the name of the underlying tag.
function filterForListItems(el) {
    return el.nodeName === 'LI';
}

const testElements = document.getElementsByClassName( 'test' );
console.log( 'There are ' + testElements.length + ' elements in testElements.');

// Generating an array from the elements gathered from testElements
// based on whether they pass the filtering proccess set up by filterForListItems
const liElements = Array.prototype.filter.call(testElements, filterForListItems);
console.log( 'There are ' +  liElements.length + ' elements in liElements.');
```

The difference between methods in the return type is caused by the vagaries of DOM implementation in browsers. In the future, both should return `HTMLCollection` instances, but that future is not yet here. Because the access patterns for `NodeLists` and `HTMLCollections` are virtually identical, we do not have to concern ourselves too much with which method returns which type.

读书笔记：`getElementsByTagName`现在返回的类型已经改为`HTMLCollections`类型。

When using either `getElementsByClassName` or `getElementsByTagName`, it is worth remembering that they belong not only to `Document` instances, but also `Element` instances. When called from the document, they will conduct searches over the entire document. Consider that your `<head>` section will be searched for `<li>` tags or that you will be looking there for elements with the class `foo`. This is, as you can imagine, somewhat inefficient. Imagine that you are searching through your house for your keys. You would probably not search in the refrigerator, or in the shower, as they are not likely spots to have left your keys. So you will look in the bedroom, the living room, the entryway, and so on. Wherever possible, limit the scope of your search to the appropriate containing element. Take a look at Listing 5-3, which gets the same results as Listing 5-2 but limits its scope to a specific parent element.

Listing 5-3. Limiting Search Scope

```js
const ul = document.getElementById( 'items' );
const liElements = ul.getElementsByClassName( 'test' );
console.log( 'There are ' +  liElements.length + ' elements in liElements.');
```

 ■ Note `document.getElementById`, unlike `getElementsByClassName` or `getElementsByTagName`, is not available on instances of the element type. It is only available on the document or an instance of the `Document` type.

These three methods are available in all modern browsers and can be immensely helpful for locating specific elements. Going back to the earlier example where we tried to find the `<h1>` element, we can now do the following:

```js
document.getElementsByTagName('h1')[0];
```

This code is guaranteed to work and will always return the first `<h1>` element in the document.

读书笔记：`document.getElementsByName(name)`是书中没有提到的另一个获取元素方法。可能是新增的。用于根据属性`name`的值获取元素集合`NodeList`。

### Finding Elements by CSS Selector

As a web developer, you already know of an alternative way to select HTML elements: CSS selectors. A CSS selector is the expression used to apply CSS styles to a set of elements. With each revision of the CSS standard (1, 2, and 3, also sometimes referred to as CSS Level 1, Level 2 or Level 3, respectively) more features have been added to the selector specification, so that developers can more easily locate the exact elements they desire. Browsers were occasionally slow to provide full implementations of CSS 2 and 3 selectors, and so you may not know of some of the cool new features that they provide. This has largely been resolved in modern browsers. If you're interested in all the cool new features in CSS, we recommend exploring the W3C's pages on the subject:

• CSS 1 selectors: http://www.w3.org/TR/CSS1/#basic-concepts

• CSS 2.1 selectors: http://www.w3.org/TR/CSS21/selector.html

• CSS 3 selectors: http://www.w3.org/TR/css3-selectors/

The features that are available from each CSS selector specification are generally similar, in that each subsequent release contains all the features from the past ones, too. However, with each release a number of new features are added. As an example, CSS 2.1 contains attribute and child selectors, while CSS 3 provides additional language support, selecting by attribute type, and negation. For modern browsers, all of these are valid CSS selectors:

  - `#main div p`: This expression finds an element with an ID of `main`, all `<div>` element descendants, and then all `<p>` element descendants. All of this is a proper CSS 1 selector.

  - `div.items > p`: This expression finds all `<div>` elements that have a class of items, then locates all child `<p>` elements. This is a valid CSS 2 selector.

- `div:not(.items)`: This locates all `<div>` elements that do not have a class of items. This is a valid CSS 3 selector.

There are two methods that provide access to elements via CSS selectors: `querySelector` and `querySelectorAll`. Give `querySelector` a valid CSS selector, and it will return a reference to the first element it finds matching that selector. The only thing that changes when using `querySelectorAll` is that you get back a non-live `NodeList` of matching elements. (The list is not live, because a live list would be resource-intensive). As with `getElementsByTagName` and `getElementsByClassName`, you can call `querySelector` and `querySelectorAll` from any instance of `Element`. Where possible, prefer limiting the scope of searches this way for greater efficiency and faster returns.

We now have four ways to access elements. Which should we use? First, for single-element access, `document.getElementById` should always be the fastest. But for multiple-element access, or if the element you want doesn't have an ID, consider using `getElementsByTagName`, then `getElementsByClassName`, then `querySelectorAll`. But keep in mind that this only takes into account speed. Sometimes, ease of querying, or accuracy of matched elements, or even the need for a live collection matters more than speed. Use the method that suits your needs best.

## Waiting for the HTML DOM to Load

One of the difficulties when working with HTML DOM documents is that your JavaScript code is able to execute before the DOM is completely loaded, potentially causing a number of problems in your code. The order of operation inside a browser looks something like this:

  1. HTML is parsed.

  2. External style sheets are loaded.

  3. Scripts are executed as they are parsed in the document.

  4. HTML DOM is fully constructed.

  5. Images and external content are loaded.

  6. The page is finished loading.

Of course, all of this is largely dependent on the structure of your HTML. If you have a `<script>` tag before the `<link>` tag that loads your CSS, then the JavaScript will load before the CSS does. (By the way, do not do this. It is inefficient.) Scripts that are in the head and loaded from an external file are executed before the HTML DOM is actually constructed. As mentioned previously, this is a significant problem because all scripts executed in those two places won't have access to the DOM. That is part of why we have avoided putting our script tags in the `<head>` section. But even when we follow best practices and include our `<script>` tags just before the closing `<body>` tag, there is the possibility that the DOM is not ready for processing by our JavaScript. Thankfully, there exist a number of workarounds for this problem.

### Waiting for the Page to Load

By far the most common technique is simply waiting for the entire page to load before performing any DOM operations. This technique can be utilized by simply attaching a function, to be fired on page load, to the load event of the window object. We will discuss events in greater detail in Chapter 6. Listing 5-4 shows an example of executing DOM-related code after the page has finished loading.

Listing 5-4. The `addEventListener` Function for Attaching a Callback to the Window load Property

```js
// Wait until the page is loaded
// (Uses addEventListener, described in the next chapter)
window.addEventListener('load', function() {
    // Perform HTML DOM operations
    const theSquare = document.getElementById('square');
    theSquare.style.background = 'blue';
});
```

While this operation may be the simplest, it will always be the slowest. From the order of loading operations, you will notice that the page being loaded is the last step taken. The load event does not fire until all elements with src attributes have had their files downloaded. This means that if your page has a significant number of images, videos, and so on, your users might be waiting quite a while until the JavaScript finally executes. On the other hand, this is the most backward-compatible solution.

### Waiting for the Right Event

If you have a more modern browser, you can check for the `DOMContentLoaded` event. This event fires when the document has been completely loaded and parsed. In our list, this matches roughly to “HTML DOM is fully constructed.” But keep in mind that images, stylesheets, videos, audio, and the like may not have completely loaded by the time this event fires. If you need your code to fire after a particular image or video file has loaded, consider using the load event for that particular tag. If you need to wait until all elements with a src attribute have downloaded their files, fall back to using the window load event. Look at Listing 5-5 for details.

Listing 5-5. Using DOMContentLoaded

```js
document.addEventListener('DOMContentLoaded' functionHandler);
```

> Internet Explorer 8 does not support `DOMContentLoaded`, 此段内容已删除，详见原书。

## Getting the Contents of an Element

All DOM elements can contain one of three things: text, more elements, or a mixture of text and elements. Generally speaking, the most common situations are the first and last. In this section you're going to see the common ways available for retrieving the contents of an element.

### Getting the Text of an Element

Getting the text inside an element is probably the most confusing task for those who are new to the DOM. However, it is also a task that works in both HTML DOM documents and XML DOM documents, so knowing how to do this will serve you well. In the example DOM structure shown in Figure 5-3, there is a root `<p>` element that contains a `<strong>` element and a block of text. The `<strong>` element itself also contains a block of text.

```
[p]-->[strong]
[p]-->[ how are you doing?]
[strong]-->[Hello]
```

Figure 5-3. A sample DOM structure containing both elements and text

Let's look at how to get the text of each of these elements. The `<strong>` element is the easiest to start with, since it contains only one text node and nothing else.

> It should be noted that there exists a property called `innerText` that captures the text inside an element in all non–Mozilla-based browsers. It's incredibly handy in that respect. Unfortunately, because it doesn't work in a noticeable portion of the browser market, and it doesn't work in XML DOM documents, you still need to explore viable alternatives.

The trick with getting the text contents of an element is that you need to remember that text is not contained within the element directly; it's contained within the child text node, which may seem a bit strange. For example, Listing 5-7 shows how to extract text from inside an element using the DOM; it is assumed that the variable `strongElem` contains a reference to the `<strong>` element.

Listing 5-7. Getting the Text Contents of the `<strong>` Element

```js
// Non-Mozilla Browsers:
strongElem.innerText

// All platforms including Non-Mozilla browsers:
strongElem.firstChild.nodeValue
```

Now that you know how to get the text contents of a single element, you need to look at how to get the combined text contents of the `<p>` element. In doing so, you might as well develop a generic function to get the text contents of any element, regardless of what it actually contain, as shown in Listing 5-8. Calling `text(Element)` will return a string containing the combined text contents of the element and all child elements that it contains.

Listing 5-8. A Generic Function for Retreiving the Text Contents of an Element

```js
function text(e) {
    let t = '' ;
    // If an element was passed, get its children,
    // otherwise assume it's an array
    e = e.childNodes || e;
    // Look through all child nodes
    for (let j = 0; j < e.length; j++ ) {
        // If it's not an element, append its text value
        // Otherwise, recurse through all the element's children
        t += e[j].nodeType != 1 ? //ELEMENT_NODE = 1
            e[j].nodeValue : text(e[j].childNodes);
    }

    // Return the matched text
    return t;
}
```

With a function that can be used to get the text contents of any element, you can retrieve the text
contents of the `<p>` element, used in the previous example. The code to do so would look something like this:

```js
// Get the text contents of the <p> Element
const pElm = document.getElementsByTagName ('p');
console.log(text( pElem ));
```

The particularly nice thing about this function is that it's guaranteed to work in both HTML and XML DOM documents, which means you now have a consistent way of retrieving the text contents of any element.

### Getting the HTML of an Element

Unlike getting the text inside an element, getting an element's HTML is one of the easiest DOM tasks that can be performed. Thanks to a feature developed by the Internet Explorer team, all modern browsers now include an extra property on every HTML DOM element: `innerHTML`. With this property you can get all the HTML and text inside of an element. Additionally, using the `innerHTML` property is very fast—often much faster than doing a recursive search to find all the text contents of an element. However, it isn't all roses.

It's up to the browser to figure out how to implement the `innerHTML` property, and because there's no true standard for that, the browser can return whatever content it deems worthy. For example, here are some of the weird bugs you can look forward to when using the `innerHTML` property:

• Mozilla-based browsers don't return the `<style>` elements in an `innerHTML` statement.

• Internet Explorer 8 and lower returns its elements in all caps, which if you're looking for consistency can be frustrating.

• The `innerHTML` property is consistently available only as a property on elements of HTML DOM documents; trying to use it on XML DOM documents will result in retrieving null values.

Using the `innerHTML` property is straightforward; accessing the property gives you a string containing the HTML contents of the element. If the element doesn't contain any subelements and only text, the returned string will contain only the text. To look at how it works, we're going to examine the two elements shown in Figure 5-2:

```js
// Get the innerHTML of the <strong> element
// Should return "Hello"
strongElem.innerHTML

// Get the innerHTML of the <p> element
// Should return "<strong>Hello</strong> how are you doing?"
pElem.innerHTML
```

读书笔记：`innerHTML` property返回`string`类型。

If you're certain that your element contains nothing but text, this method could serve as a very simple replacement to the complexities of getting the element text. On the other hand, being able to retrieve the HTML contents of an element means that you can build some cool dynamic applications that take advantage of in-place editing; more on this topic can be found in Chapter 10.

## Working with Element Attributes

Next to retrieving the contents of an element, getting and setting the value of an element's attribute is one of the most frequent operations. Typically, the list of attributes that an element has is preloaded with information collected from the XML representation of the element itself and stored in an associative array for later access, as in this example of an HTML snippet inside a web page:

```html
<form name="myForm" action="/test.cgi" method="POST">
    ...
</form>
```

Once loaded into the DOM and the variable formElem, the HTML form element would have an associative array from which you could collect name/value attribute pairs. The result would look something like this:

```js
formElem.attributes = {
    name: 'myForm',
    action: '/test.cgi',
    method: 'POST'
};
```

Figuring out whether an element's attribute exists should be absolutely trivial using the attributes array, but there's one problem: for whatever reason Safari doesn't support this feature. Internet Explorer version 8 and above support it, as long as IE8 is in standards mode. So how are you supposed to find out if an attribute exists? One possible way is to use the `getAttribute` function (covered in the next section) and test to see whether the return value is null, as shown in Listing 5-9.

Listing 5-9. Determining Whether an Element Has a Certain Attribute

```js
function hasAttribute( elem, name ) {
    return elem.getAttribute(name) != null;
}
```

With this function in hand, and knowing how attributes are used, you are now ready to begin retrieving and setting attribute values.

### Getting and Setting an Attribute Value

There are two methods to retrieve attribute data from an element, depending on the type of DOM document you're using. If you want to be safe and always use generic XML DOM–compatible methods, there are `getAttribute()` and `setAttribute()`. They can be used in this manner:

```js
// Get an attribute
document.getElementById('everywhere').getAttribute('id');
// Set an attribute value
document.getElementsByTagName('input')[0].setAttribute('value', 'Your Name');
```

In addition to this standard `getAttribute`/`setAttribute` pair, HTML DOM documents have an extra set of properties that act as quick getters/setters for your attributes. These are universally available in modern DOM implementations (but only guaranteed for HTML DOM documents), so using them can give you a big advantage when writing short code. The following example shows how you can use DOM properties to both access and set DOM attributes:

```js
// Quickly get an attribute
document.getElementsByTagName('input')[0].value;

// Quickly set an attribute
document.getElementsByTagName('div')[0].id = 'main';
```

There are a couple of strange cases with attributes that you should be aware of. The one that's most frequently encountered is that of accessing the class name attribute. If you are referencing the name of a class directly, `elem.className` will let you set and get the name. However, if you're using the `getAttribute`/`setAttribute` method, you can refer to it as `getAttribute('class')`. To work with class names consistently in all browsers you must access the `className` attribute using `elem.className`, instead of using the more appropriately named `getAttribute('class')`. This problem also occurs with the for attribute, which gets renamed to `htmlFor`. Additionally, it is also the case with a couple of CSS attributes: `cssFloat` and `cssText`. This particular naming convention arose because words such as `class`, `for`, `float`, and `text` are all reserved words in JavaScript.

To work around all these strange cases and simplify the process of dealing with getting and setting the right attributes, you should use a function that will take care of all those particulars for you. Listing 5-10 shows a function for getting and setting the values of element attributes. Calling the function with two parameters, such as `attr(element, id)`, returns that value of that attribute. Calling the function with three parameters, such as `attr(element, class, test)`, will set the value of the attribute and return its new value.

Listing 5-10. Getting and Setting the Values of Element Attributes

```js
function attr(elem, name, value) {
    // Make sure that a valid attribute name was provided
    if ( !name || name.constructor != String ) return '' ;

    // Figure out if the name is one of the weird naming cases
    name = { 'for': 'htmlFor', 'className': 'class' }[name] || name;

    // If the user is setting a value, also
    if ( typeof value != 'undefined' ) {
        // Set the quick way first
        elem[name] = value;

        // If we can, use setAttribute
        if ( elem.setAttribute )
            elem.setAttribute(name,value);
    }

     // Return the value of the attribute
    return elem[name] || elem.getAttribute(name) || '';
}
```

Having a standard way to both access and change attributes, regardless of their implementation, is a powerful tool. Listing 5-11 shows some examples of how you could use the `attr` function in a number of common situations to simplify the process of dealing with attributes.

Listing 5-11. Using the `attr` Function to Set and Retrieve Attribute Values from DOM Elements

```js
// Set the class for the first <h1> Element
attr( document.getElementByTagName('h1')[0], 'class', 'header' );

const input = document.getElementByTagName('input');

// Set the value for each <input> element
for ( let i = 0; i < input.length; i++ ) {
  attr( input[i], 'value', ''  );
}

// Add a border to the <input> Element that has a name of 'invalid'
for ( let i = 0; i < input.length; i++ ) {
  if ( attr( input[i], 'name' ) == 'invalid' ) {
    input[i].style.border = '2px solid red';
  }
}
```

Up until now, we've only discussed getting/setting attributes that are commonly used in the DOM (ID, class, name, and so on). However, a very handy technique is to set and get nontraditional attributes. For example, you could add a new attribute (which can only be seen by accessing the DOM version of an element) and then retrieve it again later, all without modifying the physical properties of the document. For example, let's say that you want to have a definition list of items, and whenever a term is clicked have the definition expand. The HTML for this setup would look something like Listing 5-12.

Listing 5-12. An HTML Document with a Definition List, with the Definitions Hidden

```html
<html>
<head>
  <title>Expandable Definition List</title>
  <style>dd { display: none; }</style>
</head>
<body>
  <h1>Expandable Definition List</h1>
  <dl>
  <dt>Cats</dt>
  <dd>A furry, friendly, creature.</dd>
  <dt>Dog</dt>
  <dd>Like to play and run around.</dd>
  <dt>Mice</dt>
  <dd>Cats like to eat them.</dd>
  </dl>
</body>
</html>
```

We'll talk more about the particulars of events in Chapter 6, but for now we'll try to keep our event code simple enough. What follows is a quick script that allows you to click the terms and show (or hide) their definitions. Listing 5-13 shows the code required to build an expandable definition list.

Listing 5-13. Allowing for Dynamic Toggling to the Definitions

```js
// Wait until the DOM is Ready
document.addEventListener('DOMContentLoaded', addEventClickToTerms);

// Watch for a user click on the term
function addEventClickToTerms(){
    const dt = document.getElementsByTagName('dt');
    for ( let i = 0; i < dt.length; i++ ) {
        dt[i].addEventListener('click', checkIfOpen);
    }
}

// See if the definition is already open or not
function checkIfOpen(e){
    const dd = e.target.nextSibling.nextSibling
    if(dd.style.display == '' || dd.style.display == 'none'){
        dd.style.display = 'block';
    } else {
        dd.style.display = 'none';
    }
}
```

Binding `<dd>` need two nextSiblings because the first sibling is a text node (the words that were clicked on). If the dt's never been clicked, the style will be blank ''. or if it has been, the style will be 'none', so we check for both with an if statement.

Now that you know how to traverse the DOM and how to examine and modify attributes, you need to learn how to create new DOM elements, insert them where they are needed, and remove elements that you no longer need.

## Modifying the DOM

By knowing how to modify the DOM, you can do anything from creating custom XML documents on the fly to building dynamic forms that adapt to user input; the possibilities are nearly limitless. Modifying the DOM comes in three steps: first you need to learn how to create a new element, then you need to learn how to insert it into the DOM, then you need to learn how to remove it again.

### Creating Nodes Using the DOM

The primary method of modifying the DOM is the `createElement` function, which gives you the ability to create new elements on the fly. However, this new element is not immediately inserted into the DOM when you create it (a common point of confusion for people just starting with the DOM). First, we'll focus on creating a DOM element.

The `createElement` method takes one parameter, the tag name of the element, and returns the virtual DOM representation of that element—no attributes or styling included. If you're developing applications that use XSLT-generated XHTML pages (or if the applications are XHTML pages served with an accurate content type), you have to remember that you're actually using an XML document and that your elements need to have the correct XML namespace associated with them. To work around this seamlessly, you can have a simple function that quietly tests to see whether the HTML DOM document you're using has the ability to create new elements with a namespace (a feature of XHTML DOM documents). If this is the case, you must create a new DOM element with the correct XHTML namespace, as shown in Listing 5-14.

Listing 5-14. A Generic Function for Creating a New DOM Element

```js
function create( elem ) {
    return document.createElementNS ?
        document.createElementNS('http://www.w3.org/1999/xhtml', elem ) :
        document.createElement( elem );
}
```

For example, using the previous function you can create a simple `<div>` element and attach some additional information to it:

```js
const div = create('div');
div.className = 'items';
div.id = 'all';
```

Additionally, it should be noted that there is a DOM method for creating new text nodes, called `createTextNode`. It takes a single argument, the text that you want inside the node, and it returns the created text node.

Using the newly created DOM elements and text nodes, you can now insert them into your DOM document right where you need them.

### Inserting into the DOM

Inserting into the DOM is confusing and can feel clumsy at times, even for those experienced with the DOM. You have two functions in your arsenal that you can use to get the job done.

The first function, `insertBefore`, allows you to insert an element before another child element. When you use the function, it looks something like this:

```js
parentOfBeforeNode.insertBefore( nodeToInsert, beforeNode );
```

The mnemonic that we use to remember the order of the arguments is the phrase “You're inserting the first element, before the second.”

Now that you have a function to insert nodes (including both elements and text nodes) before other nodes, you should be asking yourself: “How do I insert a node as the last child of a parent?” There is another function you can use, called `appendChild`, that allows you to do just that. `appendChild` is called on an element, appending the specified node to the end of the list of child nodes. Using the function looks something like this:

```js
parentElem.appendChild( nodeToInsert );
```

Listing 5-15 is an example of how you can use both `insertBefore` and `appendChild` in your application.

Listing 5-15.  A Function for Inserting an Element Before Another Element

```js
document.addEventListener(DOMContentLoaded, 'addElement');

function addElement(){
  //Grab the ordered list that is in the document
  //Remember that getElementById returns an array like object
  const orderedList = document.getElementById('myList');

  //Create an <li>, add a text node then append it to <li>
  const li = document.createElement('li');
  li.appendChild(document.createTextNode('Thanks for visiting'));

  //element [0] is how we access what is inside the orderedList
  orderedList.insertBefore(li, orderedList[0]);
}
```

The instant you “insert” this information into the DOM (with either `insertBefore` or `appendChild`) it will be immediately rendered and seen by the user. Because of this, you can use it to provide instantaneous feedback. This is especially helpful in interactive applications that require user input.

Now that you've seen how to create and insert nodes using nothing but DOM-based methods, it should be especially beneficial to look at alternative methods of injecting content into the DOM.

### Injecting HTML into the DOM

A technique that is even more popular than creating normal DOM elements and inserting them into the DOM is that of injecting HTML straight into the document. The simplest method for achieving this is by using the previously discussed `innerHTML` method. Besides being a way to retrieve the HTML inside an element, it is also a way to set the HTML inside an element. As an example of its simplicity, let's assume that you have an empty `<ol>` element and you want to add some `<li>`s to it; the code to do so would look like this:

```js
// Add some LIs to an OL element
document.getElementsByTagName('ol')[0].innerHTML = "<li>Cats.</li><li>Dogs.</li><li>Mice.</li>";
```

Isn't that so much simpler than obsessively creating a number of DOM elements and their associated text nodes? You'll be happy to know that it's much faster than using the DOM methods, too. It's not all perfect, however—there are a number of tricky problems that exist with using the `innerHTML` injection method:

• As mentioned previously, the `innerHTML` method doesn't exist in XML DOM documents, meaning that you'll have to continue to use the traditional DOM creation methods.

• XHTML documents that are created using client-side XSLT don't have an `innerHTML` method, as they, too, are pure XML documents.

• `innerHTML` completely removes any nodes that already exist inside the element, meaning that there's no way to conveniently append or insert before, as we can with the pure DOM methods.

The last point is especially troublesome, as inserting before another element or appending onto the end of a child list is a particularly useful feature. Let's look at how it can be done in Listing 5-16 using the same methods we were using before.

Listing 5-16. Adding New DOM Nodes to an Existing Ordered List

```js
document.addEventListener('DOMContentLoaded', activateButtons);

function activateButtons(){
    //ad event listeners to buttons
    const appendBtn = document.querySelector('#appendButon');
    appendBtn.addEventListener('click', appendNode);

    const addBtn = document.querySelector('#addButton');
    addBtn.addEventListener('click', addNode);
}

function appendNode(e){
    //get the <li>s that exist and make a new one.
    const listItems = document.getElementsByTagName('li');
    const newListItem = document.createElement('li');
    //append a new text node
    newListItem.appendChild(document.createTextNode('Mouse trap.'));

    //append to existing list as the new 4th item
    listItems[2].appendChild(newListItem);
}

function addNode(e){
    //get the whole list
    const orderedList = document.getElementById('myList');

    //get all the <li>s
    const listItems = document.getElementsByTagName('li');
    //make a new <li> and attach text node
    const newListItem = document.createElement('li');
    newListItem.appendChild(document.createTextNode('Zebra.'));
    //add to list, pushing the 2nd one down to 3rd
    orderedList.insertBefore(newListItem, listItems[1]);
}
```

Using this example you can see that it is not very difficult to make changes to an existing document. However, what if you want to move the other way and remove nodes from the DOM? As always, there's another method to handle that, too.

### Removing Nodes from the DOM

Removing nodes from the DOM occurs nearly as frequently as creating and inserting them. When you're creating a dynamic form asking for an unlimited number of items, for example, it becomes important to allow the user to be able to remove portions of the page that they no longer wish to deal with. The ability to remove a node is encapsulated into one function: `removeChild`. It's used just like `appendChild`, but it has the opposite effect. The function in action looks something like this:

```js
NodeParent.removeChild( NodeToRemove );
```

With this in mind, you can create two separate functions to quickly remove nodes. The first removes a single node, as shown in Listing 5-17.

Listing 5-17. Function for Removing a Node from the DOM

```js
// Remove a single Node from the DOM
function remove( elem ) {
    if ( elem ) elem.parentNode.removeChild( elem );
}
```

Listing 5-18 shows a function for removing all child nodes from an element, using only a reference to the DOM element.

Listing 5-18. A Function for Removing All Child Nodes from an Element

```js
// Remove all of an Element's children from the DOM
function empty( elem ) {
    while ( elem.firstChild )
        remove( elem.firstChild );
}
```

As an example, let's say you want to remove an `<li>` that you added in a previous section, assuming that you've already given the user enough time to view the `<li>` and that it can be removed without implication. Listing 5-19 shows the JavaScript code that you can use to perform such an action, creating a desirable result.

Listing 5-19. Removing Either a Single Element or All Elements from the DOM

```js
// Remove the last <li> from an <ol>
const listItems = document.getElementsByTagName('li');
remove(listItems[2]);
```

The preceding will convert this:

```html
<ol>
    <li>Learn Javascript.</li>
    <li>???</li>
    <li>Profit!</li>
</ol>
```

Into this:

```html
<ol>
    <li>Learn Javascript.</li>
    <li>???</li>
</ol>
```

If we were to run the `empty()` function instead of `remove()`

```js
const orderedList = document.getElementById('myList');
empty(orderedList);
```

It would simply empty out our `<ol>`, leaving:

```html
<ol></ol>
```

### Handling White Space in the DOM

Let's go back to our example HTML document. Previously, you attempted to locate the single `<h1>` element and had difficulty because of the extraneous text nodes. That may be acceptable for one single element, but what if you want to find the next element after the `<h1>` element? You still hit the infamous white space bug, and you'll need to do .nextSibling.nextSibling to skip past the end lines between the `<h1>` and the `<p>` elements. All is not lost, though. There is one technique that can act as a workaround for the white-space bug, shown in Listing 5-20. This particular technique removes all white-space–only text nodes from a DOM document, making it easier to traverse. Doing this will have no noticeable effect on how your HTML renders, but it will make it easier for you to navigate by hand. It should be noted that the results of this function are not permanent and will need to be rerun every time the HTML document is loaded.

Listing 5-20. A Workaround for the White-Space Bug in XML Documents

```js
function cleanWhitespace( element ) {
    // If no element is provided, do the whole HTML document
    element = element || document;
    // Use the first child as a starting point
    let cur = element.firstChild;
    // Go until there are no more child nodes
    while ( cur != null ) {
        // If the node is a TEXT_NODE, and it contains nothing but whitespace
        if ( cur.nodeType == 3 && ! /\S/.test(cur.nodeValue) ) {
            // Remove the text node
            element.removeChild( cur );
        // Otherwise, if it's an ELEMENT_NODE
        } else if ( cur.nodeType == 1 ) {
             // Recurse down through the document
             cleanWhitespace( cur );
        }
        cur = cur.nextSibling; // Move through the child nodes
    }
}
```

Let's say that you want to use this function in your example document to find the element after the first `<h1>` element. The code to do so would look something like this:

```js
cleanWhitespace();
// Find the H1 Element
document.documentElement
    .firstChild         // Find the Head Element
    .nextSibling        // Find the <body> Element
    .firstChild         // Get the H1 Element
    .nextSibling        // Get the adjacent Paragraph
```

This technique has both advantages and disadvantages. The greatest advantage is that you get to maintain some level of sanity when trying to navigate your DOM document. However, this technique is particularly slow, considering that you have to traverse every single DOM element and text node looking for the text nodes that contain nothing but white space. If you have a document with a lot of content in it, it could significantly slow down the loading of your site. Additionally, every time you inject new HTML into your document, you'll need to rescan that portion of the DOM, making sure that no additional space-filled text nodes were added.

One important aspect of this function is the use of node types. A node's type can be determined by checking its `nodeType` property for a particular value. We have a list of types at the beginning of this chapter. So you can see are a number of possible values, but the three that you'll encounter the most are the following:

Element (`nodeType` = 1): This matches most elements in an XML file. For example, `<li>`, `<a>`, `<p>`, and `<body>` elements all have a `nodeType` of 1.

Text (`nodeType` = 3): This matches all text segments within your document. When navigating through a DOM structure using `previousSibling` and `nextSibling`, you'll frequently encounter pieces of text inside and between elements.

Document (`nodeType` = 9): This matches the root element of a document. For example, in an HTML document it's the `<html>` element.

Additionally, you can use constants to refer to the different DOM node types (in version 9 of IE and later). For example, instead of having to remember 1, 3, or 9, you could just use `document.ELEMENT_NODE`, `document.TEXT_NODE`, or `document.DOCUMENT_NODE`. Since constantly cleaning the DOM's white space has the potential to be cumbersome, you should explore other ways to navigate a DOM structure.

### Simple DOM Navigation

Using the principle of pure DOM navigation (having pointers in every navigable direction), you can develop functions that might better suit you when navigating an HTML DOM document. This particular principle reflects the fact that most web developers only need to navigate around DOM elements and very rarely navigate through sibling text nodes. To aid you, there are a number of helpful functions that can be used in place of the standard `previousSibling`, `nextSibling`, `firstChild`, `lastChild`, and `parentNode`. Listing 5-21 shows a function that returns the element previous to the current element, or null if no previous element is found, similar to the `previousSibling` element property.

Listing 5-21. A Function for Finding the Previous Sibling Element in Relation to an Element

```js
///上一个元素节点，而非文本节点
function prev( elem ) {
    do {
        elem = elem.previousSibling;
    } while ( elem && elem.nodeType != 1 ); //ELEMENT_NODE
    return elem;
}
```

Listing 5-22 shows a function that returns the element next to the current element, or null if no next element is found, similar to the `nextSibling` element property.

Listing 5-22. A Function for Finding the Next Sibling Element in Relation to an Element

```js
///下一个元素节点，而非文本节点
function next( elem ) {
    do {
        elem = elem.nextSibling;
    } while ( elem && elem.nodeType != 1 ); //ELEMENT_NODE
    return elem;
}
```

Listing 5-23 shows a function that returns the first element child of an element, similar to the `firstChild` element property.

Listing 5-23. A Function for Finding the First Child Element of an Element

```js
///首元素节点，而非文本节点
function first( elem ) {
    elem = elem.firstChild;
    return elem && elem.nodeType != 1 ?
        next ( elem ) : elem;
}
```

Listing 5-24 shows a function that returns the last element child of an element, similar to the `lastChild` element property.

Listing 5-24. A Function for Finding the Last Child Element of an Element

```js
///尾元素节点，而非文本节点
function last( elem ) {
    elem = elem.lastChild;
    return elem && elem.nodeType != 1 ?
        prev ( elem ) : elem;
}
```

Listing 5-25 shows a function that returns the parent element of an element, similar to the `parentNode` element property. You can optionally provide a number to navigate up multiple parents at a time—for example, parent(elem,2) is equivalent to parent(parent(elem)).

Listing 5-25. A Function for Finding the Parent of an Element

```js
function parent( elem, num ) {
    num = num || 1;
    for ( let i = 0; i < num; i++ )
        if ( elem != null ) elem = elem.parentNode;
    return elem;
}
```

Using these new functions you can quickly browse through a DOM document without having to worry about the text in between each element. For example, to find the element next to the `<h1>` element, as before, you can now do the following:

```js
const h1 = first( document.body )
// Find the Element next to the <h1> Element
next( h1 )
```

You should notice two things with this code. First, there is a new reference: `document.body`. All modern browsers provide a reference to the `<body>` element inside the body parameter of an HTML DOM document. You can use this to make your code shorter and more understandable. The other thing you might notice is that the way the functions are written is very counterintuitive. Normally, when you think of navigating you might say, “Start at the `<body>` element, get the first element, and then get the next element,” but with the way it's physically written, it seems backward.

## Summary

In this chapter we talked a lot about what the DOM is and how it's structured. We also covered relationships between nodes, node types and how to access elements using JavaScript. When we have access to these elements we can change the attributes of them by using `element.getAttribute()`/`element.setAttribute()`. We also discussed creating and adding new nodes to the DOM, handling white space, and navigating though the DOM. In the next chapter we will talk about events with JavaScript.
