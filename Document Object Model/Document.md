# Document

The **`Document`** interface represents any web page loaded in the browser and serves as an entry point into the web page's content, which is the DOM tree. The DOM tree includes elements such as `<body>` and `<table>`, among many others. It provides functionality globally to the document, like how to obtain the page's URL and create new elements in the document.

```
[EventTarget]<|-[Node]<|-[Document]
```

The `Document` interface describes the common properties and methods for any kind of document. Depending on the document's type (e.g. HTML, XML, SVG, …), a larger API is available: HTML documents, served with the `"text/html"` content type, also implement the `HTMLDocument` interface, whereas XML and SVG documents implement the `XMLDocument` interface.

## Properties

- `Document.body`

  Returns the `<body>` or `<frameset>` node of the current document.

- `Document.documentElement`Read only

  Returns the `Element` that is a direct child of the document. For HTML documents, this is normally the `HTMLHtmlElement` object representing the document's `<html>` element.

- `Document.head`Read only

  Returns the `<head>` element of the current document.

The `Document` interface is extended with the `ParentNode` interface:

- `ParentNode.children` Read only

  Returns a live `HTMLCollection` containing all of the `Element` objects that are children of this `ParentNode`, omitting all of its non-element nodes.




### Extensions for HTMLDocument

The `Document` interface for HTML documents inherits from the `HTMLDocument` interface or, since HTML5, is extended for such documents.

- `Document.cookie`

  Returns a semicolon-separated list of the cookies for that document or sets a single cookie.

- `Document.title`

  Sets or gets the title of the current document.

- `Document.URL`Read only

  Returns the document location as a string.
  
  

## Methods

- `Document.createDocumentFragment()`

  Creates a new document fragment.

- `Document.createElement()`

  Creates a new element with the given tag name.

- `Document.createTextNode()`

  Creates a text node.

- `Document.getElementsByClassName()`

  Returns a list of elements with the given class name.

- `Document.getElementsByTagName()`

  Returns a list of elements with the given tag name.

- `document.getElementById()`

  Returns an object reference to the identified element.

- `Document.querySelector()`

  Returns the first Element node within the document, in document order, that matches the specified selectors.

- `Document.querySelectorAll()`

  Returns a list of all the Element nodes within the document that match the specified selectors.



### Extension for HTML documents

The `Document` interface for HTML documents inherit from the `HTMLDocument` interface or, since HTML5, is extended for such documents:

- `Document.getElementsByName()`

  Returns a list of elements with the given name.

- `Document.hasFocus()`

  Returns `true` if the focus is currently located anywhere inside the specified document.



