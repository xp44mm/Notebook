# Element

**`Element`** is the most general base class from which all element objects (i.e. objects that represent elements) in a [`Document`](https://developer.mozilla.org/en-US/docs/Web/API/Document) inherit. It only has methods and properties common to all kinds of elements. More specific classes inherit from `Element`. For example, the [`HTMLElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement) interface is the base interface for HTML elements, while the [`SVGElement`](https://developer.mozilla.org/en-US/docs/Web/API/SVGElement) interface is the basis for all SVG elements. Most functionality is specified further down the class hierarchy.

Languages outside the realm of the Web platform, like XUL through the `XULElement` interface, also implement `Element`.

```js
class Element extends Node {}
```



## Properties

Inherits properties from its parent interface, `Node`, and by extension that interface's parent, `EventTarget`. It implements the properties of `ParentNode`, `ChildNode`, `NonDocumentTypeChildNode`, and `Animatable`.

- [`Element.attributes`](https://developer.mozilla.org/en-US/docs/Web/API/Element/attributes) Read only

  Returns a `NamedNodeMap` object containing the assigned attributes of the corresponding HTML element.

- [`Element.classList`](https://developer.mozilla.org/en-US/docs/Web/API/Element/classList) Read only

  Returns a [`DOMTokenList`](https://developer.mozilla.org/en-US/docs/Web/API/DOMTokenList) containing the list of class attributes.

- [`Element.className`](https://developer.mozilla.org/en-US/docs/Web/API/Element/className)

  Is a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) representing the class of the element.

- [`Element.clientHeight`](https://developer.mozilla.org/en-US/docs/Web/API/Element/clientHeight) Read only

  Returns a [`Number`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) representing the inner height of the element.

- [`Element.clientLeft`](https://developer.mozilla.org/en-US/docs/Web/API/Element/clientLeft) Read only

  Returns a [`Number`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) representing the width of the left border of the element.

- [`Element.clientTop`](https://developer.mozilla.org/en-US/docs/Web/API/Element/clientTop) Read only

  Returns a [`Number`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) representing the width of the top border of the element.

- [`Element.clientWidth`](https://developer.mozilla.org/en-US/docs/Web/API/Element/clientWidth) Read only

  Returns a [`Number`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) representing the inner width of the element.

- [`Element.computedName`](https://developer.mozilla.org/en-US/docs/Web/API/Element/computedName) Read only

  Returns a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) containing the label exposed to accessibility.

- [`Element.computedRole`](https://developer.mozilla.org/en-US/docs/Web/API/Element/computedRole) Read only

  Returns a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) containing the ARIA role that has been applied to a particular element.

- [`Element.id`](https://developer.mozilla.org/en-US/docs/Web/API/Element/id)

  Is a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) representing the id of the element.

- [`Element.innerHTML`](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML)

  Is a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) representing the markup of the element's content.

- [`Element.localName`](https://developer.mozilla.org/en-US/docs/Web/API/Element/localName) Read only

  A [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) representing the local part of the qualified name of the element.

- [`Element.namespaceURI`](https://developer.mozilla.org/en-US/docs/Web/API/Element/namespaceURI) Read only

  The namespace URI of the element, or `null` if it is no namespace.

  **Note:** In Firefox 3.5 and earlier, HTML elements are in no namespace. In later versions, HTML elements are in the `http://www.w3.org/1999/xhtml` namespace in both HTML and XML trees.

- [`NonDocumentTypeChildNode.nextElementSibling`](https://developer.mozilla.org/en-US/docs/Web/API/NonDocumentTypeChildNode/nextElementSibling) Read only

  Is an [`Element`](https://developer.mozilla.org/en-US/docs/Web/API/Element), the element immediately following the given one in the tree, or `null` if there's no sibling node.

- [`Element.outerHTML`](https://developer.mozilla.org/en-US/docs/Web/API/Element/outerHTML)

  Is a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) representing the markup of the element including its content. When used as a setter, replaces the element with nodes parsed from the given string.

- [`Element.part`](https://developer.mozilla.org/en-US/docs/Web/API/Element/part)

  Represents the part identifier(s) of the element (i.e. set using the `part` attribute), returned as a [`DOMTokenList`](https://developer.mozilla.org/en-US/docs/Web/API/DOMTokenList).

- [`Element.prefix`](https://developer.mozilla.org/en-US/docs/Web/API/Element/prefix) Read only

  A [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) representing the namespace prefix of the element, or `null` if no prefix is specified.

- [`NonDocumentTypeChildNode.previousElementSibling`](https://developer.mozilla.org/en-US/docs/Web/API/NonDocumentTypeChildNode/previousElementSibling) Read only

  Is a [`Element`](https://developer.mozilla.org/en-US/docs/Web/API/Element), the element immediately preceding the given one in the tree, or `null` if there is no sibling element.

- [`Element.scrollHeight`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollHeight) Read only

  Returns a [`Number`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) representing the scroll view height of an element.

- [`Element.scrollLeft`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollLeft)

  Is a [`Number`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) representing the left scroll offset of the element.

- `Element.scrollLeftMax` Read only

  非标

- [`Element.scrollTop`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollTop)

  A [`Number`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) representing number of pixels the top of the document is scrolled vertically.

- `Element.scrollTopMax` Read only

  非标

- [`Element.scrollWidth`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollWidth) Read only

  Returns a [`Number`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) representing the scroll view width of the element.

- [`Element.shadowRoot`](https://developer.mozilla.org/en-US/docs/Web/API/Element/shadowRoot) Read only

  Returns the open shadow root that is hosted by the element, or null if no open shadow root is present.

- `Element.openOrClosedShadowRoot` Read only

  非标

- [`Element.slot`](https://developer.mozilla.org/en-US/docs/Web/API/Element/slot)

  试验

- [`Element.tabStop`](https://developer.mozilla.org/en-US/docs/Web/API/Element/tabStop)

  非标

- [`Element.tagName`](https://developer.mozilla.org/en-US/docs/Web/API/Element/tagName) Read only

  Returns a [`String`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String) with the name of the tag for the given element.

- [`Element.undoManager`](https://developer.mozilla.org/en-US/docs/Web/API/Element/undoManager) Read only

  试验

- [`Element.undoScope`](https://developer.mozilla.org/en-US/docs/Web/API/Element/undoScope) 

  试验

**Note:** DOM Level 3 defined `namespaceURI`, `localName` and `prefix` on the `Node` interface. In DOM4 they were moved to `Element`.

This change is implemented in Chrome since version 46.0 and Firefox since version 48.0.

### Properties included from Slotable



The `Element` interface includes the following property, defined on the [`Slotable`](https://developer.mozilla.org/en-US/docs/Web/API/Slotable) mixin.

- [`Slotable.assignedSlot`](https://developer.mozilla.org/en-US/docs/Web/API/Slotable/assignedSlot) Read only

  Returns a [`HTMLSlotElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLSlotElement) representing the `<slot>` the node is inserted in.

### Event handlers



- [`Element.onfullscreenchange`](https://developer.mozilla.org/en-US/docs/Web/API/Element/onfullscreenchange)

  An event handler for the `fullscreenchange` event, which is sent when the element enters or exits full-screen mode. This can be used to watch both for successful expected transitions, but also to watch for unexpected changes, such as when your app is running in the background.

- [`Element.onfullscreenerror`](https://developer.mozilla.org/en-US/docs/Web/API/Element/onfullscreenerror)

  An event handler for the `fullscreenerror` event, which is sent when an error occurs while attempting to change into full-screen mode.

## Methods

Inherits methods from its parents [`Node`](https://developer.mozilla.org/en-US/docs/Web/API/Node), and its own parent, [`EventTarget`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget), and implements those of [`ParentNode`](https://developer.mozilla.org/en-US/docs/Web/API/ParentNode), [`ChildNode`](https://developer.mozilla.org/en-US/docs/Web/API/ChildNode), [`NonDocumentTypeChildNode`](https://developer.mozilla.org/en-US/docs/Web/API/NonDocumentTypeChildNode), and [`Animatable`](https://developer.mozilla.org/en-US/docs/Web/API/Animatable).

- [`EventTarget.addEventListener()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener)

  Registers an event handler to a specific event type on the element.

- [`Element.attachShadow()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/attachShadow)

  Attaches a shadow DOM tree to the specified element and returns a reference to its [`ShadowRoot`](https://developer.mozilla.org/en-US/docs/Web/API/ShadowRoot).

- [`Element.animate()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/animate) 

  试验

- [`Element.closest()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/closest) 

  试验

- [`Element.createShadowRoot()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/createShadowRoot) 

  非标，不推荐

- [`Element.computedStyleMap()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/computedStyleMap) 

  试验

- [`EventTarget.dispatchEvent()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/dispatchEvent)

  Dispatches an event to this node in the DOM and returns a [`Boolean`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean) that indicates whether no handler canceled the event.

- [`Element.getAnimations()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/getAnimations) 

  试验

- [`Element.getAttribute()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/getAttribute)

  Retrieves the value of the named attribute from the current node and returns it as an [`Object`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object).

- [`Element.getAttributeNames()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/getAttributeNames)

  Returns an array of attribute names from the current element.

- [`Element.getAttributeNS()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/getAttributeNS)

  Retrieves the value of the attribute with the specified name and namespace, from the current node and returns it as an [`Object`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object).

- [`Element.getBoundingClientRect()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect)

  Returns the size of an element and its position relative to the viewport.

- [`Element.getClientRects()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/getClientRects)

  Returns a collection of rectangles that indicate the bounding rectangles for each line of text in a client.

- [`Element.getElementsByClassName()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/getElementsByClassName)

  Returns a live [`HTMLCollection`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCollection) that contains all descendants of the current element that possess the list of classes given in the parameter.

- [`Element.getElementsByTagName()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/getElementsByTagName)

  Returns a live [`HTMLCollection`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCollection) containing all descendant elements, of a particular tag name, from the current element.

- [`Element.getElementsByTagNameNS()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/getElementsByTagNameNS)

  Returns a live [`HTMLCollection`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCollection) containing all descendant elements, of a particular tag name and namespace, from the current element.

- [`Element.hasAttribute()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/hasAttribute)

  Returns a [`Boolean`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean) indicating if the element has the specified attribute or not.

- [`Element.hasAttributeNS()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/hasAttributeNS)

  Returns a [`Boolean`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean) indicating if the element has the specified attribute, in the specified namespace, or not.

- [`Element.hasAttributes()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/hasAttributes)

  Returns a [`Boolean`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean) indicating if the element has one or more HTML attributes present.

- [`Element.hasPointerCapture()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/hasPointerCapture)

  Indicates whether the element on which it is invoked has pointer capture for the pointer identified by the given pointer ID.

- [`Element.insertAdjacentElement()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentElement)

  Inserts a given element node at a given position relative to the element it is invoked upon.

- [`Element.insertAdjacentHTML()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentHTML)

  Parses the text as HTML or XML and inserts the resulting nodes into the tree in the position given.

- [`Element.insertAdjacentText()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/insertAdjacentText)

  Inserts a given text node at a given position relative to the element it is invoked upon.

- [`Element.matches()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/matches) 

  试验

- [`Element.pseudo()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/pseudo) 

  试验

- [`Element.querySelector()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/querySelector)

  Returns the first [`Node`](https://developer.mozilla.org/en-US/docs/Web/API/Node) which matches the specified selector string relative to the element.

- [`Element.querySelectorAll()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/querySelectorAll)

  Returns a [`NodeList`](https://developer.mozilla.org/en-US/docs/Web/API/NodeList) of nodes which match the specified selector string relative to the element.

- [`Element.releasePointerCapture()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/releasePointerCapture)

  Releases (stops) pointer capture that was previously set for a specific [`pointer event`](https://developer.mozilla.org/en-US/docs/Web/API/PointerEvent).

- [`ChildNode.remove()`](https://developer.mozilla.org/en-US/docs/Web/API/ChildNode/remove) 

  试验

- [`Element.removeAttribute()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/removeAttribute)

  Removes the named attribute from the current node.

- [`Element.removeAttributeNS()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/removeAttributeNS)

  Removes the attribute with the specified name and namespace, from the current node.

- [`EventTarget.removeEventListener()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/removeEventListener)

  Removes an event listener from the element.

- [`Element.requestFullscreen()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/requestFullscreen) 

  试验

- [`Element.requestPointerLock()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/requestPointerLock) 

  试验

- [`Element.scroll()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scroll)

  Scrolls to a particular set of coordinates inside a given element.

- [`Element.scrollBy()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollBy)

  Scrolls an element by the given amount.

- [`Element.scrollIntoView()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollIntoView) 

  试验

- [`Element.scrollTo()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scrollTo)

  Scrolls to a particular set of coordinates inside a given element.

- [`Element.setAttribute()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/setAttribute)

  Sets the value of a named attribute of the current node.

- [`Element.setAttributeNS()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/setAttributeNS)

  Sets the value of the attribute with the specified name and namespace, from the current node.

- [`Element.setCapture()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/setCapture) 

  非标

- [`Element.setPointerCapture()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/setPointerCapture)

  Designates a specific element as the capture target of future [pointer events](https://developer.mozilla.org/en-US/docs/Web/API/Pointer_events).

- [`Element.toggleAttribute()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/toggleAttribute)

  Toggles a boolean attribute, removing it if it is present and adding it if it is not present, on the specified element.

## Events

Listen to these events using `addEventListener()` or by assigning an event listener to the `on*eventname*` property of this interface.

- [`cancel`](https://developer.mozilla.org/en-US/docs/Web/API/Element/cancel_event)

  Fires on a `<dialog>` when the user instructs the browser that they wish to dismiss the current open dialog. For example, the browser might fire this event when the user presses the Esc key or clicks a "Close dialog" button which is part of the browser's UI. Also available via the [`oncancel`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/oncancel) property.

- `error`

  Fired when when a resource failed to load, or can't be used. For example, if a script has an execution error or an image can't be found or is invalid. Also available via the [`onerror`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onerror) property.

- [`scroll`](https://developer.mozilla.org/en-US/docs/Web/API/Element/scroll_event)

  Fired when the document view or an element has been scrolled. Also available via the [`onscroll`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onscroll) property.

- [`select`](https://developer.mozilla.org/en-US/docs/Web/API/Element/select_event)

  Fired when some text has been selected. Also available via the [`onselect`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onselect) property.

- [`show`](https://developer.mozilla.org/en-US/docs/Web/API/Element/show_event)

  不推荐

- [`wheel`](https://developer.mozilla.org/en-US/docs/Web/API/Element/wheel_event)

  Fired when the user rotates a wheel button on a pointing device (typically a mouse). Also available via the [`onwheel`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onwheel) property.

### Clipboard events



- [`copy`](https://developer.mozilla.org/en-US/docs/Web/API/Element/copy_event)

  Fired when the user initiates a copy action through the browser's user interface. Also available via the [`oncopy`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/oncopy) property.

- [`cut`](https://developer.mozilla.org/en-US/docs/Web/API/Element/cut_event)

  Fired when the user initiates a cut action through the browser's user interface. Also available via the [`oncut`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/oncut) property.

- [`paste`](https://developer.mozilla.org/en-US/docs/Web/API/Element/paste_event)

  Fired when the user initiates a paste action through the browser's user interface. Also available via the [`onpaste`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/onpaste) property.

### Composition events



- [`compositionend`](https://developer.mozilla.org/en-US/docs/Web/API/Element/compositionend_event)

  Fired when a text composition system such as an [input method editor](https://developer.mozilla.org/en-US/docs/Glossary/input_method_editor) completes or cancels the current composition session.

- [`compositionstart`](https://developer.mozilla.org/en-US/docs/Web/API/Element/compositionstart_event)

  Fired when a text composition system such as an [input method editor](https://developer.mozilla.org/en-US/docs/Glossary/input_method_editor) starts a new composition session.

- [`compositionupdate`](https://developer.mozilla.org/en-US/docs/Web/API/Element/compositionupdate_event)

  Fired when a new character is received in the context of a text composition session controlled by a text composition system such as an [input method editor](https://developer.mozilla.org/en-US/docs/Glossary/input_method_editor).

### Focus events



- [`blur`](https://developer.mozilla.org/en-US/docs/Web/API/Element/blur_event)

  Fired when an element has lost focus. Also available via the [`onblur`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onblur) property.

- [`focus`](https://developer.mozilla.org/en-US/docs/Web/API/Element/focus_event)

  Fired when an element has gained focus. Also available via the [`onfocus`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onfocus) property

- [`focusin`](https://developer.mozilla.org/en-US/docs/Web/API/Element/focusin_event)

  Fired when an element is about to gain focus.

- [`focusout`](https://developer.mozilla.org/en-US/docs/Web/API/Element/focusout_event)

  Fired when an element is about to lose focus.

### Fullscreen events



- `fullscreenchange`

  Sent to an [`Element`](https://developer.mozilla.org/en-US/docs/Web/API/Element) when it transitions into or out of [full-screen](https://developer.mozilla.org/en-US/docs/Web/API/Fullscreen_API/Guide) mode. Also available via the [`onfullscreenchange`](https://developer.mozilla.org/en-US/docs/Web/API/Element/onfullscreenchange) property.

- `fullscreenerror`

  Sent to an `Element` if an error occurs while attempting to switch it into or out of [full-screen](https://developer.mozilla.org/en-US/docs/Web/API/Fullscreen_API/Guide) mode. Also available via the [`onfullscreenerror`](https://developer.mozilla.org/en-US/docs/Web/API/Element/onfullscreenerror) property.

### Keyboard events



- `keydown`

  Fired when a key is pressed. Also available via the [`onkeydown`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onkeydown) property.

- `keypress`

  不推荐

- `keyup`

  Fired when a key is released. Also available via the [`onkeyup`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onkeyup) property.

### Mouse events



- [`auxclick`](https://developer.mozilla.org/en-US/docs/Web/API/Element/auxclick_event)

  Fired when a non-primary pointing device button (e.g., any mouse button other than the left button) has been pressed and released on an element. Also available via the [`onauxclick`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onauxclick) property.

- [`click`](https://developer.mozilla.org/en-US/docs/Web/API/Element/click_event)

  Fired when a pointing device button (e.g., a mouse's primary button) is pressed and released on a single element. Also available via the [`onclick`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onclick) property.

- [`contextmenu`](https://developer.mozilla.org/en-US/docs/Web/API/Element/contextmenu_event)

  Fired when the user attempts to open a context menu. Also available via the [`oncontextmenu`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/oncontextmenu) property.

- [`dblclick`](https://developer.mozilla.org/en-US/docs/Web/API/Element/dblclick_event)

  Fired when a pointing device button (e.g., a mouse's primary button) is clicked twice on a single element. Also available via the [`ondblclick`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/ondblclick) property.

- [`DOMActivate`](https://developer.mozilla.org/en-US/docs/Web/API/Element/DOMActivate_event) 

  不推荐

- [`mousedown`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mousedown_event)

  Fired when a pointing device button is pressed on an element. Also available via the [`onmousedown`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onmousedown) property.

- [`mouseenter`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mouseenter_event)

  Fired when a pointing device (usually a mouse) is moved over the element that has the listener attached. Also available via the [`onmouseenter`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onmouseenter) property.

- [`mouseleave`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mouseleave_event)

  Fired when the pointer of a pointing device (usually a mouse) is moved out of an element that has the listener attached to it. Also available via the [`onmouseleave`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onmouseleave) property.

- [`mousemove`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mousemove_event)

  Fired when a pointing device (usually a mouse) is moved while over an element. Also available via the [`onmousemove`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onmousemove) property.

- [`mouseout`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mouseout_event)

  Fired when a pointing device (usually a mouse) is moved off the element to which the listener is attached or off one of its children. Also available via the [`onmouseout`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onmouseout) property.

- [`mouseover`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mouseover_event)

  Fired when a pointing device is moved onto the element to which the listener is attached or onto one of its children. Also available via the [`onmouseover`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onmouseover) property.

- [`mouseup`](https://developer.mozilla.org/en-US/docs/Web/API/Element/mouseup_event)

  Fired when a pointing device button is released on an element. Also available via the [`onmouseup`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/onmouseup) property.

- [`webkitmouseforcechanged`](https://developer.mozilla.org/en-US/docs/Web/API/Element/webkitmouseforcechanged_event)

  非标

- [`webkitmouseforcedown`](https://developer.mozilla.org/en-US/docs/Web/API/Element/webkitmouseforcedown_event)

  非标

- [`webkitmouseforcewillbegin`](https://developer.mozilla.org/en-US/docs/Web/API/Element/webkitmouseforcewillbegin_event)

  非标

- [`webkitmouseforceup`](https://developer.mozilla.org/en-US/docs/Web/API/Element/webkitmouseforceup_event)

  非标

### Touch events



- [`touchcancel`](https://developer.mozilla.org/en-US/docs/Web/API/Element/touchcancel_event)

  Fired when one or more touch points have been disrupted in an implementation-specific manner (for example, too many touch points are created). Also available via the [`ontouchcancel`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/ontouchcancel) property.

- [`touchend`](https://developer.mozilla.org/en-US/docs/Web/API/Element/touchend_event)

  Fired when one or more touch points are removed from the touch surface. Also available via the [`ontouchend`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/ontouchend) property

- [`touchmove`](https://developer.mozilla.org/en-US/docs/Web/API/Element/touchmove_event)

  Fired when one or more touch points are moved along the touch surface. Also available via the [`ontouchmove`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/ontouchmove) property

- [`touchstart`](https://developer.mozilla.org/en-US/docs/Web/API/Element/touchstart_event)

  Fired when one or more touch points are placed on the touch surface. Also available via the [`ontouchstart`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers/ontouchstart) property