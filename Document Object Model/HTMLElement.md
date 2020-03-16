# HTMLElement

The **`HTMLElement`** interface represents any HTML element. Some elements directly implement this interface, while others implement it via an interface that inherits it.

```js
class HTMLElement extends Element {}
```



## Properties

*Inherits properties from its parent, `Element`, and implements those from `GlobalEventHandlers` and `TouchEventHandlers`.*

- `HTMLElement.accessKey`

  Is a `DOMString` representing the access key assigned to the element.

- `HTMLElement.accessKeyLabel` Read only

  Returns a `DOMString` containing the element's assigned access key.

- `HTMLElement.contentEditable`

  Is a `DOMString`, where a value of `"true"` means the element is editable and a value of `"false"` means it isn't.

- `HTMLElement.isContentEditable` Read only

  Returns a `Boolean` that indicates whether or not the content of the element can be edited.

- `HTMLElement.dataset` Read only

  Returns a `DOMStringMap` with which script can read and write the element's custom data attributes (`data-*`) .

- `HTMLElement.dir`

  Is a `DOMString`, reflecting the `dir` global attribute, representing the directionality of the element. Possible values are `"ltr"`, `"rtl"`, and `"auto"`.

- `HTMLElement.draggable`

  Is a `Boolean` indicating if the element can be dragged.

- `HTMLElement.dropzone` Read only

  Returns a `DOMSettableTokenList` reflecting the `dropzone` global attribute and describing the behavior of the element regarding a drop operation.

- `HTMLElement.hidden`

  Is a `Boolean` indicating if the element is hidden or not.

- `HTMLElement.inert`

  Is a `Boolean` indicating whether the user agent must act as though the given node is absent for the purposes of user interaction events, in-page text searches ("find in page"), and text selection.

- `HTMLElement.innerText`

  Represents the "rendered" text content of a node and its descendants. As a getter, it approximates the text the user would get if they highlighted the contents of the element with the cursor and then copied it to the clipboard.

- `HTMLElement.lang`

  Is a `DOMString` representing the language of an element's attributes, text, and element contents.

- `HTMLElement.noModule`

  Is a `Boolean` indicating whether an import script can be executed in user agents that support module scripts.

- `HTMLElement.nonce`

  Returns the cryptographic number used once that is used by Content Security Policy to determine whether a given fetch will be allowed to proceed.

- `HTMLElement.spellcheck`

  Is a `Boolean` that controls spell-checking. It is present on all HTML elements, though it doesn't have an effect on all of them.

- `HTMLElement.style`

  Is a `CSSStyleDeclaration`, an object representing the declarations of an element's style attributes.

- `HTMLElement.tabIndex`

  Is a `long` representing the position of the element in the tabbing order.

- `HTMLElement.title`

  Is a `DOMString` containing the text that appears in a popup box when mouse is over the element.



## Methods

*Inherits methods from its parent, `Element`.*


- `HTMLElement.blur()`

  Removes keyboard focus from the currently focused element.

- `HTMLElement.click()`

  Sends a mouse click event to the element.

- `HTMLElement.focus()`

  Makes the element the current keyboard focus.



## Events

Listen to these events using `addEventListener()` or by assigning an event listener to the `oneventname` property of this interface.

- `invalid`

  Fired when an element does not satisfy its constraints during constraint validation. Also available via the `oninvalid` property.



### Input events



- `beforeinput`

  Fired when the value of an `<input>`, `<select>`, or `<textarea>` element is about to be modified.

- `input`

  Fired when the `value` of an `<input>`, `<select>`, or `<textarea>`  element has been changed. Also available via the `oninput` property.

