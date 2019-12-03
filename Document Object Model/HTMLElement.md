# HTMLElement

The **`HTMLElement`** interface represents any [HTML](https://developer.mozilla.org/en-US/docs/Web/HTML) element. Some elements directly implement this interface, while others implement it via an interface that inherits it.

```js
class HTMLElement extends Element {}
```



## Properties

*Inherits properties from its parent, [`Element`](https://developer.mozilla.org/en-US/docs/Web/API/Element), and implements those from [`GlobalEventHandlers`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers) and [`TouchEventHandlers`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEventHandlers).*

- [`HTMLElement.accessKey`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/accessKey)

  Is a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) representing the access key assigned to the element.

- [`HTMLElement.accessKeyLabel`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/accessKeyLabel) Read only

  Returns a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) containing the element's assigned access key.

- [`HTMLElement.contentEditable`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/contentEditable)

  Is a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString), where a value of `"true"` means the element is editable and a value of `"false"` means it isn't.

- [`HTMLElement.isContentEditable`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/isContentEditable) Read only

  Returns a [`Boolean`](https://developer.mozilla.org/en-US/docs/Web/API/Boolean) that indicates whether or not the content of the element can be edited.

- `HTMLElement.contextMenu` 

  不推荐

- [`HTMLElement.dataset`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset) Read only

  Returns a [`DOMStringMap`](https://developer.mozilla.org/en-US/docs/Web/API/DOMStringMap) with which script can read and write the element's [custom data attributes](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Using_data_attributes) (`data-*`) .

- [`HTMLElement.dir`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dir)

  Is a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString), reflecting the `dir` global attribute, representing the directionality of the element. Possible values are `"ltr"`, `"rtl"`, and `"auto"`.

- [`HTMLElement.draggable`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/draggable)

  Is a [`Boolean`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean) indicating if the element can be dragged.

- [`HTMLElement.dropzone`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dropzone) Read only

  Returns a [`DOMSettableTokenList`](https://developer.mozilla.org/en-US/docs/Web/API/DOMSettableTokenList) reflecting the `dropzone` global attribute and describing the behavior of the element regarding a drop operation.

- [`HTMLElement.hidden`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/hidden)

  Is a [`Boolean`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean) indicating if the element is hidden or not.

- [`HTMLElement.inert`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/inert)

  Is a [`Boolean`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean) indicating whether the user agent must act as though the given node is absent for the purposes of user interaction events, in-page text searches ("find in page"), and text selection.

- [`HTMLElement.innerText`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/innerText)

  Represents the "rendered" text content of a node and its descendants. As a getter, it approximates the text the user would get if they highlighted the contents of the element with the cursor and then copied it to the clipboard.

- [`HTMLElement.itemScope`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/itemScope) 

  试验

- [`HTMLElement.itemType`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/itemType) Read only

  试验

- [`HTMLElement.itemId`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/itemId) 

  试验

- [`HTMLElement.itemRef`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/itemRef) Read only

  试验

- [`HTMLElement.itemProp`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/itemProp) Read only

  试验

- [`HTMLElement.itemValue`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/itemValue) 

  试验

- [`HTMLElement.lang`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/lang)

  Is a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) representing the language of an element's attributes, text, and element contents.

- [`HTMLElement.noModule`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/noModule)

  Is a [`Boolean`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean) indicating whether an import script can be executed in user agents that support module scripts.

- [`HTMLElement.nonce`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/nonce)

  Returns the cryptographic number used once that is used by Content Security Policy to determine whether a given fetch will be allowed to proceed.

- [`HTMLElement.offsetHeight`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/offsetHeight) Read only

  试验

- [`HTMLElement.offsetLeft`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/offsetLeft) Read only

  试验

- [`HTMLElement.offsetParent`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/offsetParent) Read only

  试验

- [`HTMLElement.offsetTop`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/offsetTop) Read only

  试验

- [`HTMLElement.offsetWidth`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/offsetWidth) Read only

  试验

- [`HTMLElement.properties`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/properties) Read only

  试验

- [`HTMLElement.spellcheck`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/spellcheck)

  Is a [`Boolean`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean) that controls [spell-checking](https://developer.mozilla.org/en-US/docs/HTML/Controlling_spell_checking_in_HTML_forms). It is present on all HTML elements, though it doesn't have an effect on all of them.

- [`HTMLElement.style`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/style)

  Is a [`CSSStyleDeclaration`](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration), an object representing the declarations of an element's style attributes.

- [`HTMLElement.tabIndex`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/tabIndex)

  Is a `long` representing the position of the element in the tabbing order.

- [`HTMLElement.title`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/title)

  Is a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) containing the text that appears in a popup box when mouse is over the element.

- [`HTMLElement.translate`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/translate) 

  试验

### Event handlers



Most event handler properties, of the form `onXYZ`, are defined on the [`GlobalEventHandlers`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers) or [`TouchEventHandlers`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEventHandlers) interfaces and implemented by `HTMLElement`. In addition, the following handlers are specific to `HTMLElement`.

- [`HTMLElement.oncopy`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/oncopy) 

  非标

- [`HTMLElement.oncut`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/oncut) 

  非标

- [`HTMLElement.onpaste`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/onpaste) 

  非标

- [`TouchEventHandlers.ontouchstart`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEventHandlers/ontouchstart) 

  非标

- [`TouchEventHandlers.ontouchend`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEventHandlers/ontouchend) 

  非标

- [`TouchEventHandlers.ontouchmove`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEventHandlers/ontouchmove) 

  非标

- [`TouchEventHandlers.ontouchenter`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEventHandlers/ontouchenter) 

  非标

- [`TouchEventHandlers.ontouchleave`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEventHandlers/ontouchleave) 

  非标

- [`TouchEventHandlers.ontouchcancel`](https://developer.mozilla.org/en-US/docs/Web/API/TouchEventHandlers/ontouchcancel) 

  非标

## Methods

*Inherits methods from its parent, [`Element`](https://developer.mozilla.org/en-US/docs/Web/API/Element).*

- [`HTMLElement.attachInternals()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/attachInternals) 

  试验

- [`HTMLElement.blur()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/blur)

  Removes keyboard focus from the currently focused element.

- [`HTMLElement.click()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/click)

  Sends a mouse click event to the element.

- [`HTMLElement.focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus)

  Makes the element the current keyboard focus.

- [`HTMLElement.forceSpellCheck()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/forceSpellCheck) 

  试验

## Events

Listen to these events using `addEventListener()` or by assigning an event listener to the `on*eventname*` property of this interface.

- `invalid`

  Fired when an element does not satisfy its constraints during constraint validation. Also available via the `oninvalid` property.

### Animation events



- `animationcancel`

  Fired when an animation unexpectedly aborts. Also available via the `onanimationcancel` property.

- `animationend`

  Fired when an animation has completed normally. Also available via the `onanimationend` property.

- `animationiteration`

  Fired when an animation iteration has completed. Also available via the `onanimationiteration` property.

- `animationstart`

  Fired when an animation starts. Also available via the `onanimationstart` property.

### Input events



- `beforeinput`

  Fired when the value of an [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input), [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select), or [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea) element is about to be modified.

- `input`

  Fired when the `value` of an [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input), [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/select), or [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/textarea) element has been changed. Also available via the `oninput` property.

### Pointer events



- `gotpointercapture`

  Fired when an element captures a pointer using `setPointerCapture()`. Also available via the `ongotpointercapture` property.

- `lostpointercapture`

  Fired when a [captured pointer](https://developer.mozilla.org/en-US/docs/Web/API/Pointer_events#Pointer_capture) is released. Also available via the `onlostpointercapture` property.

- `pointercancel`

  Fired when a pointer event is canceled. Also available via the `onpointercancel` property.

- `pointerdown`

  Fired when a pointer becomes active. Also available via the `onpointerdown` property.

- `pointerenter`

  Fired when a pointer is moved into the hit test boundaries of an element or one of its descendants. Also available via the `onpointerenter` property.

- `pointerleave`

  Fired when a pointer is moved out of the hit test boundaries of an element. Also available via the `onpointerleave` property.

- `pointermove`

  Fired when a pointer changes coordinates. Also available via the `onpointermove` property.

- `pointerout`

  Fired when a pointer is moved out of the *hit test* boundaries of an element (among other reasons). Also available via the `onpointerout` property.

- `pointerover`

  Fired when a pointer is moved into an element's hit test boundaries. Also available via the `onpointerover` property.

- `pointerup`

  Fired when a pointer is no longer active. Also available via the `onpointerup` property.

### Transition events



- `transitioncancel`

  Fired when a [CSS transition](https://developer.mozilla.org/en-US/docs/CSS/Using_CSS_transitions) is canceled. Also available via the `ontransitioncancel` property.

- `transitionend`

  Fired when a [CSS transition](https://developer.mozilla.org/en-US/docs/CSS/Using_CSS_transitions) has completed. Also available via the `ontransitionend` property.

- `transitionrun`

  Fired when a [CSS transition](https://developer.mozilla.org/en-US/docs/CSS/Using_CSS_transitions) is first created. Also available via the `ontransitionrun` property.

- `transitionstart`

  Fired when a [CSS transition](https://developer.mozilla.org/en-US/docs/CSS/Using_CSS_transitions) has actually started. Also available via the `ontransitionstart` property.