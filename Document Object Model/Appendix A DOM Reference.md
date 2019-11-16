# Appendix A DOM Reference

This appendix serves as a reference for the functionality provided by the Document Object Model discussed in Chapter 5.

## Resources

DOM functionality has come in a variety of flavors, starting with the original prespecification DOM Level 0 on up to DOM Level 3. One of the things to understand about the DOM is that it is considered a living standard. Each level is describes the features and behaviors that are added. The DOM itself is a representation of the document with nodes and properties the can have events associated with them.

If you wanted to understand some of the details of DOM the W3C's web sites serve as an excellent reference for learning how the DOM should work as well as the Web Hypertext Application Technology Working Group (WHATWG):

• HTML DOM Level 3: http://www.w3.org/TR/DOM-Level-3-Core/

• WHATWG DOM: https://dom.spec.whatwg.org/

Additionally, there exist a number of excellent references for learning how DOM functionality works, but none is better than the resources that exist at Quirksmode.org, a site run by Peter-Paul Koch. He has comprehensively looked at every available DOM method and compared its results in all modern browsers (plus some). It's an invaluable resource for figuring out what is, or is not, possible in the browsers that you're developing for. Another source is also caniuse.com created by Alexis Devera. Here you can search for a feature you would like to use and see a compatibioity table for witch browsers support that feature.

## Terminology

In Chapter 5 on the Document Object Model and in this appendix, I use common XML and DOM terminology to describe the different aspects of a DOM representation of an XML document. The following words and phrases are terminology that relate to the Document Object Model and XML documents in general. All of the terminology examples will relate to the sample HTML document shown in Listing A-1.

Listing A-1. A Reference Point for Discussing DOM and XML Terminology

```html
<!doctype html>
<html>
<head>
    <title>Introduction to the DOM</title>
</head>
<body>
    <h1>Introduction to the DOM</h1>
    <p class="test">There are a number of reasons why the DOM is awesome,
        here are some:</p>
    <ul>
        <li id="everywhere">It can be found everywhere.</li>
        <li class="test">It's easy to use.</li>
        <li class="test">It can help you to find what you want, really quickly.</li>
    </ul>
</body>
</html>
```

### Ancestor

Very similar to the genealogical term, ancestor refers to the parent of the current element, and that parent's parent, and that parent's parent, and so on. In Listing A-1 the ancestor elements of the `<ul>` element are the `<body>` element and the `<html>` element.

### Attribute

Attributes are properties of elements that contain additional information about them. In Listing A-1 the `<p>` element has the attribute `class` that contains the value `test`.

### Child

Any element can contain any number of nodes (each of which are considered to be children of the parent element). In Listing A-1 the `<ul>` contains seven child nodes; three of the child nodes are the `<li>` elements, the other four are the endlines that exist in between each element (contained within text nodes).

### Document

An XML document consists of one element (called the root node or document element) that contains all other aspects of the document. In Listing A-1 the `<html>` is the document element containing the rest of the document.

### Descendant

An element's descendants include its child nodes, its children's children, and their children, and so on. In Listing A-1 the `<body>` element's descendants include `<h1>`, `<p>`, `<ul>`, all the `<li>` elements, and all the text nodes contained inside all of them.

### Element

An element is a container that holds attributes and other nodes. The primary, and most noticeable, component of any HTML document is its elements. In Listing A-1 there are a ton of elements; the `<html>`, `<head>`, `<title>`, `<body>`, `<h1>`, `<p>`, `<ul>`, and `<li>` tags all represent elements.

### Node

A node is the common unit within a DOM representation. Elements, attributes, comments, documents, and text nodes are all nodes and therefore have typical node properties (for example, `nodeType`, `nodeName`, and `nodeValue` exist in every node).

### Parent

Parent is the term used to refer to the element that contains the current node. All nodes have a parent, except for the root node. In Listing A-1 the parent of the `<p>` element is the `<body>` element.

### Sibling

A sibling node is a child of the same parent node. Generally this term is used in the context of `previousSibling` and `nextSibling`, two attributes found on all DOM nodes. In Listing A-1 the siblings of the `<p>` element are the `<h1>` and `<ul>` elements (along with a couple white space–filled text nodes).

### Text Node

A text node is a special node that contains only text; this includes visible text and all forms of white space. So when you're seeing text inside of an element (for example, `<b>hello world!</b>`), there is actually a separate text node inside of the `<b>` element that contains the “hello world!” text. In Listing A-1, the text “It's easy to use” inside of the second `<li>` element is contained within a text node.

## Global Variables

Global variables exist within the global scope of your code, but they exist to help you work with common DOM operations.

### document

This variable contains the active HTML DOM document, which is viewed in the browser. However, just because this variable exists and has a value, doesn't mean that its contents have been fully loaded and parsed. See Chapter 5 for more information on waiting for the DOM to load. Listing A-2 shows some examples of using the document variable that holds a representation of the HTML DOM to access document elements.

Listing A-2. Using the Document Variable to Access Document Elements

```js
// Locate the element with the ID of 'body'
document.getElementById("body")

// Locate all the elements with the tag name of <div>.
document.getElementsByTagName("div")
```

### HTMLElement

This variable is the superclass object for all HTML DOM elements. Extending the prototype of this element extends all HTML DOM elements. This superclass is available by default in Mozilla-based browsers and Opera. It's possible to add it to Internet Explorer and Safari. Listing A-3 shows an example of binding new functions to a global HTML element superclass. Attaching a `hasClass` function provides the ability to see whether an element has a specific class.

Listing A-3. Binding New Functions to a Global HTML Element SuperClass

```js
// Add a new method to all HTML DOM Elements
// that can be used to see if an Element has a specific class, or not.
HTMLElement.prototype.hasClass = function( class ) {
    return new RegExp("(^|\\s)" + class + "(\\s|$)").test( this.className );
};
```

## DOM Navigation

The following properties are a part of all DOM elements and can be used to traverse DOM documents.

### body

This property of the global HTML DOM document (the document variable) points directly to the HTML `<body>` element (of which there should only be the one). This particular property is one that has been carried over from the days of DOM 0 JavaScript. Listing A-4 shows some examples of accessing the `<body>` element from the HTML DOM document.

Listing A-4. Accessing the `<body>` Element Inside of an HTML DOM Document

```js
// Change the margins of the <body>
document.body.style.margin = "0px";

// document.body is equivalent to:
document.getElementsByTagName("body")[0]
```

### childNodes

This is a property of all DOM elements, containing an array of all child nodes (this includes elements, text nodes, comments, etc.). This property is read-only. Listing A-5 shows how you would use the `childNodes` property to add a style to all child elements of a parent element.

Listing A-5. Adding a Red Border Around Child Elements of the `<body>` Element Using the childNodes Property

```js
// Add a border to all child elements of <body>
const c = document.body.childNodes;
for ( let i = 0; i < c.length; i++ ) {
    // Make sure that the Node is an Element(ELEMENT_NODE == 1)
    if ( c[i].nodeType == 1 )
        c[i].style.border = "1px solid red";
}
```

### documentElement

This is a property of all DOM nodes acting as a reference to the root element of the document (in the case of HTML documents, this will always point to the `<html>` element). Listing A-6 shows an example of using the `documentElement` to find a DOM element.

Listing A-6. Example of Locating the Root Document Element From Any DOM Node

```js
// Find the documentElement, to find an Element by ID
someRandomNode.documentElement.getElementById("body")
```

### firstChild

This is a property of all DOM elements, pointing to the first child node of that element. If the element has no child nodes, `firstChild` will be equal to null. Listing A-7 shows an example of using the `firstChild` property to remove all child nodes from an element.

Listing A-7. Removing All Child Nodes From an Element

```js
// Remove all child nodes from an element
const e = document.getElementById("body");
while ( e.firstChild )
    e.removeChild( e.firstChild );
```

### getElementById( elemID )

This is a powerful function that locates the one element in the document that has the specified ID. The function is only available on the document element. Additionally, the function may not work as intended in non-HTML DOM documents; generally with XML DOM documents you have to explicitly specify the ID attribute in a DTD (Document Type Definition) or schema.

This function takes a single argument: the name of the ID that you're searching for, as demonstrated in Listing A-8.

Listing A-8. Two Examples of Locating HTML Elements by Their ID Attributes

```js
// Find the Element with an ID of body
document.getElementById("body")

// Hide the Element with an ID of notice
document.getElementById("notice").style.display = 'none';
```

### getElementsByTagName( tagName )

This property finds all descendant elements—beginning at the current element—that have the specified tag name. This function works identically in XML DOM and HTML DOM documents.

In all modern browsers, you can specify `*` as the tag name and find all descendant elements, which is much faster than using a pure-JavaScript recursive function.

This function takes a single argument: the tag name of the elements that you're searching for. Listing A-9 shows examples of `getElementsByTagName`. The first block adds a highlight class to all `<div>` elements in the document. The second block finds all the elements inside of the element with an ID of body, and hides any that have a class of highlight.

Listing A-9. Two Code Blocks That Demonstrate How `getElementsByTagName` Is Used

```js
// Find all <div> Elements in the current HTML document
// and set their class to 'highlight'
const d = document.getElementsByTagName("div");
for ( let i = 0; i < d.length; i++ ) {
    d[i].className = 'hilite';
}

// Go through all descendant elements of the element with
// an ID of body. Then find all elements that have one class
// equal to 'hilite'. Then hide all those elements that match.
const all = document.getElementById("body").getElementsByTagName("*");
for ( let i = 0; i < all.length; i++ ) {
    if ( all[i].className == 'hilite' )
        all[i].style.display = 'none';
}
```

### lastChild

This is a reference available on all DOM elements, pointing to the last child node of that element. If no child nodes exist, `lastChild` will be null. Listing A-10 shows an example of using the `lastChild` property to insert an element into a document.

Listing A-10. Creating a New `<div>` Element and Inserting It Before the Last Element in the `<body>`

```js
// Insert a new Element just before the last element in the <body>
const n = document.createElement("div");
n.innerHTML = "Thanks for visiting!";
document.body.insertBefore( n, document.body.lastChild );
```

### nextSibling

This is a reference available on all DOM nodes, pointing to the next sibling node. If the node is the last sibling, `nextSibling` will be null. It's important to remember that `nextSibling` may point to a DOM element, a comment, or even a text node; it does not serve as an exclusive way to navigate DOM elements. Listing A-11 is an example of using the `nextSibling` property to create an interactive definition list.

Listing A-11. Making All `<dt>` Elements Expand Their Sibling `<dd>` Elements Once Clicked

```js
// Find all <dt> (Defintion Term) elements
const dt = document.getElementsByTagName("dt");
for ( let i = 0; i < dt.length; i++ ) {
    // Watch for when the term is clicked
    dt[i].addEventListener( 'click', function( e ) {
        // Since each Term has an adjacent <dd> (Definition) element
        // We can display it when it's clicked

        // NOTE: Only works when there's no whitespace between <dd> elements
        e.target.nextSibling.style.display = 'block';
    };
}
```

### parentNode

This is a property of all DOM nodes. Every DOM node's `parentNode` points to the element that contains it, except for the document element, which points to null (since nothing contains the root element). Listing A-12 is an example of using the `parentNode` property to create a custom interaction. Clicking the Cancel button hides the parent element.

Listing A-12. Using the parentNode Property to Create a Custom Interaction

```js
// Watch for when a link is clicked (e.g. a Cancel link)
// and hide the parent element
document.getElementById("cancel").addEventListener( 'click', function( e ){
    e.target.parentNode.style.display = 'none';
};
```

### previousSibling

This is a reference available on all DOM nodes, pointing to the previous sibling node. If the node is the first sibling, the `previousSibling` will be null. It's important to remember that `previousSibling` may point to a DOM element, a comment, or even a text node; it does not serve as an exclusive way to navigate DOM elements. Listing A-13 shows an example of using the `previousSibling` property to hide elements.

Listing A-13. Hiding All Elements Before the Current Element

```js
// Find all elements before this one and hide them
let cur = document.getElementById("last");
while ( cur != null ) {
    cur.style.display = 'none';
    cur = cur.previousSibling;
}
```

## Node Information

These properties exist on most DOM elements in order to give you easy access to common element information.

### innerText

This is a property of all DOM elements. This property returns a string containing all the text inside of the current element. You can utilize a workaround (where you use a function to collect the values of descendant text nodes). Listing A-14 shows an example of using the `innerText` property and the `text()` function from Chapter 5.

读书笔记：`innerText`于 2016 年正式进入 HTML 标准。

Let's assume that we have an `<li>` element like this, stored in the variable '`li`':

```html
<li>Please visit <a href="http://mysite.com/">my web site</a>.</li>
```

Listing A-14. Using the `innerText` Property to Extract Text Information From an Element

```js
// Using the innerText property
li.innerText

// or the text() function described in Chapter 5
text( li )

// The result of either the property or the function is:
"Please visit my web site."
```

### nodeName

This is a property available on all DOM elements that contains an uppercase version of the element name. For example, if you have an `<li>` element and you access its nodeName property, it will return `LI`. Listing A-15 shows an example of using the `nodeName` property to modify the class names of parent elements.

Listing A-15. Locating All Parent `<li>` Elements and Setting Their Class to current

```js
// Find all the parents of this node, that are an <li> element
let cur = document.getElementById("target");
while ( cur != null ) {
    // Once the element is found, and the name verified, add a class
    if ( cur.nodeName === 'LI' )
        cur.className += " current";
    cur = cur.parentNode;
}
```

### nodeType

This is a common property of all DOM nodes, containing a number corresponding to the type of node that it is. The three most popular node types used in HTML documents are the following:

• Element node (a value of 1 or `document.ELEMENT_NODE`)

• Text node (a value of 3 or `document.TEXT_NODE`)

• Document node (a value of 9 or `document.DOCUMENT_NODE`)

Using the `nodeType` property is a reliable way of making sure that the node that you're trying to access has all the properties that you think it does (e.g., a `nodeName` property is only useful on a DOM element; so you could use `nodeType` to make sure that it's equal to 1 before accessing it). Listing A-16 shows an example of using the `nodeType` property to add a class to a number of elements.

Listing A-16. Locating the First Element in the HTML <body> and Applying a header Class to It

```js
// Find the first element in the <body>
let cur = document.body.firstChild;
while ( cur != null ) {
    // If an element was found, add the header class to it
    if ( cur.nodeType == 1 ) { // ELEMENT_NODE == 1
        cur.className += " header";
        break;
    // Otherwise, continue navigating through the child nodes
    } else {
        cur = cur.nextSibling;
    }
}
```

### nodeValue

This is a useful property of text nodes that can be used to access and manipulate the text that they contain. The best example of this in use is the `text` function presented in Chapter 5, which is used to retrieve all the text contents of an element. Listing A-17 shows an example of using the `nodeValue` property to build a simple text value function.

Listing A-17. A Function That Accepts an Element and Returns the Text Contents of It and All Its Descendant Elements

```js
function text(e) {
    // If an element was passed, get its children,
    // otherwise assume it's an array
    e = e.childNodes || e;

    let t = '';
    // Look through all child nodes
    for ( let j = 0; j < e.length; j++ ) {
        // If it's not an element, append its text value
        // Otherwise, recurse through all the element's children
        t += e[j].nodeType != 1 ?
            e[j].nodeValue : text(e[j].childNodes);
    }

    // Return the matched text
    return t;
}
```

读书笔记：与Listing 5-8相同。

## Attributes

Most attributes are available as properties of their containing element. For example, the attribute ID can be accessed using the simple `element.id`. This feature is residual from the DOM 0 days, but it's very likely that it's not going anywhere, due to its simplicity and popularity.

### className

This property allows you to add and remove classes from a DOM element. This property exists on all DOM elements. The reason I'm mentioning this specifically is that its name, `className`, is very different from the expected name of class. The strange naming is due to the fact that the word `class` is a reserved word in most object-oriented programming languages; so its use is avoided to limit difficulties in programming a web browser. Listing A-18 shows an example of using the `className` property to hide a number of elements.

Listing A-18. Finding All `<div>` Elements That Have a Class of special and Hiding Them

```js
// Find all the <div> elements in the document
const div = document.getElementsByTagName("div");
for ( let i = 0; i < div.length; i++ ) {
    // Find all the <div> elements that have a single class of 'special'
    if ( div[i].className == "special" ) {
        // And hide them
        div[i].style.display = 'none';
    }
}
```

### getAttribute( attrName )

This is a function that serves as the proper way of accessing an attribute value contained within a DOM element. Attributes are initialized with the values that the user has provided in the straight HTML document. The function takes a single argument: the name of the attribute that you want to retrieve. Listing A-19 shows an example of using the `getAttribute()` function to find input elements of a specific type.

Listing A-19. Finding the `<input>` Element Named text and Copying Its Value Into an Element With an ID of preview

```js
// Find all the form input elements
const input = document.getElementsByTagName("input");
for ( let i = 0; i < input.length; i++ ) {

    // Find the element that has a name of "text"
    if ( input[i].getAttribute("name") == "text" ) {

        // Copy the value into another element
        document.getElementById("preview").innerHTML =
            input[i].getAttribute("value");
    }
}
```

### removeAttribute( attrName )

This is a function that can be used to completely remove an attribute from an element. Typically, the result of using this function is comparable to doing a `setAttribute` with a valueof `""` (an empty string) or null; in practice, however, you should be sure to always clean up extra attributes in order to avoid any unexpected consequences.

This function takes a single argument: the name of the attribute that you wish to remove. Listing A-20 shows an example of unchecking some check boxes in a form.

Listing A-20. Finding All Check Boxes in a Document and Unchecking Them

```js
// Find all the form input elements
const input = document.getElementsByTagName("input");
for ( let i = 0; i < input.length; i++ ) {

    // Find all the checkboxes
    if ( input[i].getAttribute("type") == "checkbox" ) {

        // Uncheck the checkbox
        input[i].removeAttribute("checked");
    }
}
```

### setAttribute( attrName, attrValue )

This is a function that serves as a way of setting the value of an attribute contained within a DOM element. Additionally, it's possible to add in custom attributes that can be accessed again later while leaving the appearance of the DOM elements unaffected. `setAttribute` tends to behave rather strangely in Internet Explorer, keeping you from setting particular attributes (such as `class` or `maxlength`). This is explained more in Chapter 5. The function takes two arguments. The first is the name of the attribute. The second is the value to set the attribute to. Listing A-21 shows an example of setting the value of an attribute on a DOM element.

Listing A-21. Using the `setAttribute` Function to Create an `<a>` Link to Google

```js
// Create a new <a> element
var a = document.createElement("a").

// Set the URL to visit to Google's web site
a.setAttribute("href","http://google.com/");

// Add the inner text, giving the user something to click
a.appendChild( document.createTextNode( "Visit Google!" ) );

// Add the link at the end of the document
document.body.appendChild( a );
```

## DOM Modification

The following are all the properties and functions that are available to manipulate the DOM.

### appendChild( nodeToAppend )

This is a function that can be used to add a child node to a containing element. If the node that's being appended already exists in the document, it is moved from its current location and appended to the current element. The `appendChild` function must be called on the element that you wish to append into.

The function takes one argument: a reference to a DOM node (this could be one that you just created or a reference to a node that exists elsewhere in the document). Listing A-22 shows an example of creating a new `<ul>` element and moving all `<li>` elements into it from their original location in the DOM, then appending the new `<ul>` to the document body.

Listing A-22. Appending a Series of `<li>` Elements to a Single `<ul>`

```js
// Create a new <ul> element
const ul = document.createElement("ul");

// Find all the first <li> elements
const li = document.getElementsByTagName("li");
for ( let i = 0; i < li.length; i++ ) {

    // append each matched <li> into  our new <ul> element
    ul.appendChild( li[i] );
}

// Append our new <ul> element at the end of the body
document.body.appendChild( ul );
```

### cloneNode( true|false )

This function is a way for developers to simplify their code by duplicating existing nodes, which can then be inserted into the DOM. Since doing a normal `insertBefore` or `appendChild` call will physically move a DOM node in the document, the `cloneNode` function can be used to duplicate it instead.

The function takes one true or false argument. If the argument is true, the node and everything inside of it is cloned; if false, only the node itself is cloned. Listing A-23 shows an example of using this function to clone an element and append it to itself.

Listing A-23. Finding the First `<ul>` Element in a Document, Making a Complete Copy of It, and Appending It to Itself

```js
// Find the first <ul> element
const ul = document.getElementsByTagName("ul")[0];

// Clone the node and append it after the old one
ul.parentNode.appendChild( ul.cloneNode( true ) );
```

### createElement( tagName )

This is the primary function used for creating new elements within a DOM structure. The function exists as a property of the document within which you wish to create the element.

 ■ Note if you're using xHTML served with a content-type of application/xhtml+xml instead of regular HTML served with a content-type of text/html, you should use the `createElementNS` function instead of the `createElement` function.

This function takes one argument: the tag name of the element to create. Listing A-24 shows an example of using this function to create an element and wrap it around some other elements.

Listing A-24. Wrapping the Contents of a `<p>` Element in a `<strong>` Element

```js
// Create a new <strong> element
const s = document.createElement("strong");

// Find the first paragraph
const p = document.getElementsByTagName("p")[0];

// Wrap the contents of the <p> in the <strong> element
while ( p.firstChild ) {
    s.appendChild( p.firstChild );
}

// Put the <strong> element (containing the old <p> contents)
// back into the <p> element
p.appendChild( s );
```

### createElementNS( namespace, tagName )

This function is very similar to the `createElement` function, in that it creates a new element; however, it also provides the ability to specify a namespace for the element (for example, if you're adding an item to an XML or XHTML document).

This function takes two arguments: the namespace of the element that you're adding, and the tag name of the element. Listing A-25 shows an example of using this function to create a DOM element in a valid XHTML document.

Listing A-25. Creating a New XHTML `<p>` Element, Filling It With Some Text, and Appending It to the Document Body

```js
// Create a new XHTML-compliant <p>
const p = document.createElementNS("http://www.w3.org/1999/xhtml", "p");

// Add some text into the <p> element
p.appendChild( document.createTextNode( "Welcome to my site." ) );

// Add the <p> element into the document
document.body.insertBefore( p, document.body.firstChild );
```

### createTextNode( textString )

This is the proper way to create a new text string to be inserted into a DOM document. Since text nodes are just DOM-only wrappers for text, it is important to remember that they cannot be styled or appended to. The function only exists as a property of a DOM document.

The function takes one argument: the string that will become the contents of the text node. Listing A-26 shows an example of using this function to create a new text node and appending it to the body of an HTML page.

Listing A-26. Creating an `<h1>` Element and Appending a New Text Node

```js
// Create a new <h1> element
var h = document.createElement("h1");

// Create the header text and add it to the <h1> element
h.appendChild( document.createTextNode("Main Page") );

// Add the header to the start of the <body>
document.body.insertBefore( h, document.body.firstChild );
```

### innerHTML

This is an HTML DOM–specific property for accessing and manipulating a string version of the HTML contents of a DOM element. If you're only working with an HTML document (and not an XML one), this method can be incredibly useful, as the code it takes to generate a new DOM element can be cut down drastically (not to mention it is a faster alternative to traditional DOM methods). While this property is not part of any particular W3C standard, it still exists in every modern browser. Listing A-27 shows an example of using the `innerHTML` property to change the contents of an element whenever the contents of a `<textarea>` are changed.

Listing A-27. Watching a `<textarea>` for Changes and Updating a Live Preview With Its Value

```js
// Get the textarea to watch for updates
const t = document.getElementsByTagName("textarea")[0];

// Grab the current value of a <textarea> and update a live preview,
// everytime that it's changed
t.addEventListener('onkeypress', function ( e ) {
    document.getElementById("preview").innerHTML = this.value;
});
```

### insertBefore( nodeToInsert, nodeToInsertBefore )

This function is used to insert a DOM node anywhere into a document. The function must be called on the parent element of the node that you wish to insert it before. This is done so that you can specify null for `nodeToInsertBefore` and have your node inserted as the last child node.

The function takes two arguments. The first argument is the node that you wish to insert into the DOM; the second is the DOM node that you're inserting before. This should be a reference to a valid node. Listing A-28 shows an example of using this function to insert the favicon (the icon that you see next to a URL in the address bar of a browser) of a site next to a set of URLs on a page.

Listing A-28. Going Through All `<a>` Elements and Adding an Icon Consisting of the Site's Favicon

```js
// Find all the <a> links within the document
const a = document.getElementsByTagName("a");
for ( let i = 0; i < a.length; i++ ) {

    // Create an image of the linked-to site's favicon
    const img = document.createElement("img");
    img.src = a[i].href.split('/').splice(0,3).join('/') + '/favicon.ico';

    // Insert the image before the link
    a[i].parentNode.insertBefore( img, a[i] );
}
```

### removeChild( nodeToRemove )

This function is used to remove a node from a DOM document. The `removeChild` function must be called on the parent element of the node that you wish to remove.

The function takes one argument: a reference to the DOM node to remove from the document. Listing A-29 shows an example of running through all the `<div>` elements in the document, removing any that have a single class of warning.

Listing A-29. Removing All Elements That Have a Particular Class Name

```js
// Find all <div> elements
const div = document.getElementsByTagName("div");
for ( let i = 0; i < div.length; i++ ) {
    // If  the <div> has one class of 'warning'
    if ( div[i].className == "warning" ) {

        // Remove the <div> from the document
        div[i].parentNode.removeChild( div[i] );
    }
}
```

### replaceChild( nodeToInsert, nodeToReplace )

This function serves as an alternative to the process of removing a node and inserting another node in its place. This function must be called by the parent element of the node that you are replacing.

This function takes two arguments: the node that you wish to insert into the DOM, and the node that you are going to replace. Listing A-30 shows an example of replacing all `<a>` elements with a `<strong>` element containing the URL originally being linked to.

Listing A-30. Converting a Set of Links Into Plain URLs

```js
// Convert all links to visible URLs (good for printing
// Find all <a> links in the document
const a = document.getElementsByTagName("a");
while ( a.length > 0 ) {

    // Create a <strong> element
    const s = document.createElement("strong");

    // Make the contents equal to the <a> link URL
    s.appendChild( document.createTextNode( a[i].href ) );

    // Replace the original <a> with the new <strong> element
    a[i].replaceChild( s, a[i] );
}
```

