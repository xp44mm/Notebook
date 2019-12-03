# DocumentFragment

 The **`DocumentFragment`** interface represents a minimal document object that has no parent. It is used as a lightweight version of `Document` that stores a segment of a document structure comprised of nodes just like a standard document. The key difference is that because the document fragment isn't part of the active document tree structure, changes made to the fragment don't affect the document, cause reflow, or incur any performance impact that can occur when changes are made. 

```js
class DocumentFragment extends Node {}
```

## Constructor

- [`DocumentFragment()`](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment/DocumentFragment)

  Creates and returns a new `DocumentFragment` object.

## Properties

This interface has no specific properties, but inherits those of its parent, `Node`, and implements those of the `ParentNode` interface.

- [`ParentNode.children`](https://developer.mozilla.org/en-US/docs/Web/API/ParentNode/children) Read only

  Returns a live `HTMLCollection` containing all objects of type `Element` that are children of the `DocumentFragment` object.

- [`ParentNode.firstElementChild`](https://developer.mozilla.org/en-US/docs/Web/API/ParentNode/firstElementChild) Read only

  Returns the `Element` that is the first child of the `DocumentFragment` object, or `null` if there is none.

- [`ParentNode.lastElementChild`](https://developer.mozilla.org/en-US/docs/Web/API/ParentNode/lastElementChild) Read only

  Returns the `Element` that is the last child of the `DocumentFragment` object, or `null` if there is none.

- [`ParentNode.childElementCount`](https://developer.mozilla.org/en-US/docs/Web/API/ParentNode/childElementCount) Read only

  Returns an `unsigned long` giving the amount of children that the `DocumentFragment` has.

## Methods

This interface inherits the methods of its parent, `Node`, and implements those of the `ParentNode` interface.

- [`DocumentFragment.querySelector()`](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment/querySelector)

  Returns the first `Element` node within the `DocumentFragment`, in document order, that matches the specified selectors.

- [`DocumentFragment.querySelectorAll()`](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment/querySelectorAll)

  Returns a `NodeList` of all the `Element` nodes within the `DocumentFragment` that match the specified selectors.

- [`DocumentFragment.getElementById()`](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment/getElementById)

  Returns the first `Element` node within the `DocumentFragment`, in document order, that matches the specified ID. Functionally equivalent to `Document.getElementById()`.

## Usage notes

A common use for `DocumentFragment` is to create one, assemble a DOM subtree within it, then append or insert the fragment into the DOM using `Node` interface methods such as `appendChild()` or `insertBefore()`. Doing this moves the fragment's child nodes into the DOM, leaving behind an empty `DocumentFragment`. Because all of the nodes are inserted into the document at once, only one reflow and render is triggered instead of potentially one for each node inserted if they were inserted separately.

This interface is also of great use with Web components: `<template>` elements contain a `DocumentFragment` in their `HTMLTemplateElement.content` property.

An empty `DocumentFragment` can be created using the `document.createDocumentFragment()` method or the constructor.

## Example

### HTML



```html
<ul id="list"></ul>
```

### JavaScript



```js
let list = document.querySelector('#list')
let fruits = ['Apple', 'Orange', 'Banana', 'Melon']

let fragment = new DocumentFragment()

fruits.forEach(function (fruit) {
  let li = document.createElement('li')
  li.innerHTML = fruit
  fragment.appendChild(li)
})

list.appendChild(fragment)
```

###   