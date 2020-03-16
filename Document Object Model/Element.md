# Element

**`Element`** is the most general base class from which all element objects (i.e. objects that represent elements) in a `Document` inherit. It only has methods and properties common to all kinds of elements. More specific classes inherit from `Element`. For example, the `HTMLElement` interface is the base interface for HTML elements, while the `SVGElement` interface is the basis for all SVG elements. Most functionality is specified further down the class hierarchy.

Languages outside the realm of the Web platform, like XUL through the `XULElement` interface, also implement `Element`.

```js
class Element extends Node {}
```



## Properties

Inherits properties from its parent interface, `Node`, and by extension that interface's parent, `EventTarget`. It implements the properties of `ParentNode`, `ChildNode`, `NonDocumentTypeChildNode`, and `Animatable`.

- `Element.attributes` Read only

  Returns a `NamedNodeMap` object containing the assigned attributes of the corresponding HTML element.

- `Element.classList` Read only

  Returns a `DOMTokenList` containing the list of class attributes.

- `Element.className`

  Is a `DOMString` representing the class of the element.

- `Element.id`

  Is a `DOMString` representing the id of the element.

- `Element.tagName` Read only

  Returns a `String` with the name of the tag for the given element.


## Methods

Inherits methods from its parents `Node`, and its own parent, `EventTarget`, and implements those of `ParentNode`, `ChildNode`, `NonDocumentTypeChildNode`, and `Animatable`.



- `Element.getAttribute()`

  Retrieves the value of the named attribute from the current node and returns it as an `Object`.

- `Element.getAttributeNames()`

  Returns an array of attribute names from the current element.

- `Element.getElementsByClassName()`

  Returns a live `HTMLCollection` that contains all descendants of the current element that possess the list of classes given in the parameter.

- `Element.getElementsByTagName()`

  Returns a live `HTMLCollection` containing all descendant elements, of a particular tag name, from the current element.

- `Element.hasAttribute()`

  Returns a `Boolean` indicating if the element has the specified attribute or not.

- `Element.hasAttributes()`

  Returns a `Boolean` indicating if the element has one or more HTML attributes present.

- `Element.insertAdjacentElement()`

  Inserts a given element node at a given position relative to the element it is invoked upon.


- `Element.querySelector()`

  Returns the first `Node` which matches the specified selector string relative to the element.

- `Element.querySelectorAll()`

  Returns a `NodeList` of nodes which match the specified selector string relative to the element.

- `Element.removeAttribute()`

  Removes the named attribute from the current node.

- `Element.setAttribute()`

  Sets the value of a named attribute of the current node.

- `Element.toggleAttribute()`

  Toggles a boolean attribute, removing it if it is present and adding it if it is not present, on the specified element.
  

## Events

Listen to these events using `addEventListener()` or by assigning an event listener to the `oneventname` property of this interface.

- `cancel`

  Fires on a `<dialog>` when the user instructs the browser that they wish to dismiss the current open dialog. For example, the browser might fire this event when the user presses the Esc key or clicks a "Close dialog" button which is part of the browser's UI. Also available via the `oncancel` property.

- `error`

  Fired when when a resource failed to load, or can't be used. For example, if a script has an execution error or an image can't be found or is invalid. Also available via the `onerror` property.

- `scroll`

  Fired when the document view or an element has been scrolled. Also available via the `onscroll` property.

- `select`

  Fired when some text has been selected. Also available via the `onselect` property.


- `wheel`

  Fired when the user rotates a wheel button on a pointing device (typically a mouse). Also available via the `onwheel` property.

### Clipboard events



- `copy`

  Fired when the user initiates a copy action through the browser's user interface. Also available via the `oncopy` property.

- `cut`

  Fired when the user initiates a cut action through the browser's user interface. Also available via the `oncut` property.

- `paste`

  Fired when the user initiates a paste action through the browser's user interface. Also available via the `onpaste` property.
  


### Focus events



- `blur`

  Fired when an element has lost focus. Also available via the `onblur` property.

- `focus`

  Fired when an element has gained focus. Also available via the `onfocus` property

- `focusin`

  Fired when an element is about to gain focus.

- `focusout`

  Fired when an element is about to lose focus.



### Keyboard events



- `keydown`

  Fired when a key is pressed. Also available via the `onkeydown` property.

- `keypress`

  不推荐

- `keyup`

  Fired when a key is released. Also available via the `onkeyup` property.
  

### Mouse events


- `auxclick`

  Fired when a non-primary pointing device button (e.g., any mouse button other than the left button) has been pressed and released on an element. Also available via the `onauxclick` property.

- `click`

  Fired when a pointing device button (e.g., a mouse's primary button) is pressed and released on a single element. Also available via the `onclick` property.

- `contextmenu`

  Fired when the user attempts to open a context menu. Also available via the `oncontextmenu` property.

- `dblclick`

  Fired when a pointing device button (e.g., a mouse's primary button) is clicked twice on a single element. Also available via the `ondblclick` property.

- `DOMActivate` 

  不推荐

- `mousedown`

  Fired when a pointing device button is pressed on an element. Also available via the `onmousedown` property.

- `mouseenter`

  Fired when a pointing device (usually a mouse) is moved over the element that has the listener attached. Also available via the `onmouseenter` property.

- `mouseleave`

  Fired when the pointer of a pointing device (usually a mouse) is moved out of an element that has the listener attached to it. Also available via the `onmouseleave` property.

- `mousemove`

  Fired when a pointing device (usually a mouse) is moved while over an element. Also available via the `onmousemove` property.

- `mouseout`

  Fired when a pointing device (usually a mouse) is moved off the element to which the listener is attached or off one of its children. Also available via the `onmouseout` property.

- `mouseover`

  Fired when a pointing device is moved onto the element to which the listener is attached or onto one of its children. Also available via the `onmouseover` property.

- `mouseup`

  Fired when a pointing device button is released on an element. Also available via the `onmouseup` property.


### Touch events



- `touchcancel`

  Fired when one or more touch points have been disrupted in an implementation-specific manner (for example, too many touch points are created). Also available via the `ontouchcancel` property.

- `touchend`

  Fired when one or more touch points are removed from the touch surface. Also available via the `ontouchend` property

- `touchmove`

  Fired when one or more touch points are moved along the touch surface. Also available via the `ontouchmove` property

- `touchstart`

  Fired when one or more touch points are placed on the touch surface. Also available via the `ontouchstart` property