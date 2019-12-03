# The HTML DOM API

 https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API 

The **HTML DOM API** is made up of the interfaces that define the functionality of each of the elements in HTML, as well as any supporting types and interfaces they rely upon.

The functional areas included in the HTML DOM API include:

- Access to and control of HTML elements via the DOM.
- Access to and manipulation of form data.
- Interacting with the contents of 2D images and the context of an HTML `<canvas>`, for example to draw on top of them.
- Management of media connected to the HTML media elements (`<audio>` and `<video>`).
- Dragging and dropping of content on webpages.
- Access to the browser navigation history
- Supporting and connective interfaces for other APIs such as Web Components, Web Storage, Web Workers, WebSocket, and Server-sent events.

## HTML DOM concepts and usage

In this article, we'll focus on the parts of the HTML DOM that involve engaging with HTML elements. Discussion of other areas, such as Drag and Drop, WebSockets, Web Storage, etc. can be found in the documentation for those APIs.

### Structure of an HTML document



The Document Object Model ([DOM](https://developer.mozilla.org/en-US/docs/Glossary/DOM)) is an architecture that describes the structure of a `document`; each document is represented by an instance of the interface `Document`. A document, in turn, consists of a hierarchical tree of **nodes**, in which a node is a fundamental record representing a single object within the document (such as an element or text node).

Nodes may be strictly organizational, providing a means for grouping other nodes together or for providing a point at which a hierarchy can be constructed; other nodes may represent visible components of a document. Each node is based on the `Node` interface, which provides properties for getting information about the node as well as methods for creating, deleting, and organizing nodes within the DOM.

Nodes don't have any concept of including the content that is actually displayed in the document. They're empty vessels. The fundamental notion of a node that can represent visual content is introduced by the `Element` interface. An `Element` object instance represents a single element in a document created using either HTML or an XML vocabulary such as SVG.

For example, consider a document with two elements, one of which has two more elements nested inside it:

```js
const Window = {
    Document: {
        Element,
        Element: {
            Element,
            Element,
        }
    }
}
```

While the `Document` interface is defined as part of the DOM specification, the HTML specification significantly enhances it to add information specific to using the DOM in the context of a web browser, as well as to using it to represent HTML documents specifically.

Among the things added to `Document` by the HTML standard are:

- Support for accessing various information provided by the HTTP headers when loading the page, such as the location from which the document was loaded, cookies, modification date, referring site, and so forth.
- Access to lists of elements in the document's `<head>` block and body, as well as lists of the images, links, scripts, etc. contained in the document.
- Support for interacting with the user by examining focus and by executing commands on editable content.
- Event handlers for document events defined by the HTML standard to allow access to mouse and keyboard events, drag and drop, media control, and more.
- Event handlers for events that can be delivered to both elements and documents; these presently include only copy, cut, and paste actions.

### HTML element interfaces



The `Element` interface has been further adapted to represent HTML elements specifically by introducing the `HTMLElement` interface, which all more specific HTML element classes inherit from. This expands the `Element` class to add HTML-specific general features to the element nodes. Properties added by `HTMLElement` include for example `hidden` and `innerText`. `HTMLElement` also adds all the [global event handlers](https://developer.mozilla.org/en-US/docs/Web/API/GlobalEventHandlers).

An HTML document is a DOM tree in which each of the nodes is an HTML element, represented by the `HTMLElement` interface. The `HTMLElement` class, in turn, implements `Node`, so every element is also a node (but not the other way around). This way, the structural features implemented by the `Node` interface are also available to HTML elements, allowing them to be nested within each other, created and deleted, moved around, and so forth.

The `HTMLElement` interface is generic, however, providing only the functionality common to all HTML elements such as the element's ID, its coordinates, the HTML making up the element, information about scroll position, and so forth.

In order to expand upon the functionality of the core `HTMLElement` interface to provide the features needed by a specific element, the `HTMLElement` class is subclassed to add the needed properties and methods. For example, the `<canvas>` element is represented by an object of type `HTMLCanvasElement`. `HTMLCanvasElement` augments the `HTMLElement` type by adding properties such as `height` and methods like `getContext()` to provide canvas-specific features.

The overall inheritance for HTML element classes looks like this pseudo code:

```js
const EventTarget = {
    Node: {
        Element: {
            HTMLElement: {
                HTMLAnchorElement,
                HTMLBRElement,
                ...
                HTMLUListElement,
            }
        }
    }
}
```

As such, an element inherits the properties and methods of all of its ancestors. For example, consider a `<a>` element, which is represented in the DOM by an object of type `HTMLAnchorElement`. The element, then, includes the anchor-specific properties and methods described in that class's documentation, but also those defined by `HTMLElement` and `Element`, as well as from `Node` and, finally, `EventTarget`.

Each level defines a key aspect of the utility of the element. From `Node`, the element inherits concepts surrounding the ability for the element to be contained by another element, and to contain other elements itself. Of special importance is what is gained by inheriting from `EventTarget`: the ability to receive and handle events such as mouse clicks, play and pause events, and so forth.

There are elements that share commonalities and thus have an additional intermediary type. For example, the `<audio>` and `<video>` elements both present audiovisual media. The corresponding types, `HTMLAudioElement` and `HTMLVideoElement`, are both based upon the common type `HTMLMediaElement`, which in turn is based upon `HTMLElement` and so forth. `HTMLMediaElement` defines the methods and properties held in common between audio and video elements.

These element-specific interfaces make up the majority of the HTML DOM API, and are the focus of this article. To learn more about the actual structure of the DOM, see Introduction to the DOM.

## HTML DOM target audience

The features exposed by the HTML DOM are among the most commonly-used APIs in the web developer's arsenal. All but the most simple web applications will use some features of the HTML DOM.

## HTML DOM API interfaces

The majority of the interfaces that comprise the HTML DOM API map almost one-to-one to individual HTML elements, or to a small group of elements with similar functionality. In addition, the HTML DOM API includes a few interfaces and types to support the HTML element interfaces.

### HTML element interfaces



These interfaces represent specific HTML elements (or sets of related elements which have the same properties and methods associated with them).

- [`HTMLAnchorElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLAnchorElement)
- [`HTMLAreaElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLAreaElement)
- [`HTMLAudioElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLAudioElement)
- [`HTMLBRElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLBRElement)
- [`HTMLBaseElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLBaseElement)
- [`HTMLBaseFontElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLBaseFontElement) 
- [`HTMLBodyElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLBodyElement)
- [`HTMLButtonElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement)
- [`HTMLCanvasElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement)
- [`HTMLDListElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDListElement)
- [`HTMLDataElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDataElement)
- [`HTMLDataListElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDataListElement)
- [`HTMLDetailsElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDetailsElement)
- [`HTMLDialogElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDialogElement)
- [`HTMLDirectoryElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDirectoryElement)
- [`HTMLDivElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLDivElement)
- [`HTMLElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement)
- [`HTMLEmbedElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLEmbedElement)
- [`HTMLFieldSetElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFieldSetElement)
- [`HTMLFontElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFontElement) 
- [`HTMLFormElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement)
- [`HTMLFrameElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFrameElement) 
- [`HTMLFrameSetElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFrameSetElement) 
- [`HTMLHRElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHRElement)
- [`HTMLHeadElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHeadElement)
- [`HTMLHeadingElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHeadingElement)
- [`HTMLHtmlElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHtmlElement)
- [`HTMLIFrameElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLIFrameElement)
- [`HTMLImageElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement)
- [`HTMLInputElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement)
- [`HTMLIsIndexElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLIsIndexElement) 
- [`HTMLLIElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLLIElement)
- [`HTMLLabelElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLLabelElement)
- [`HTMLLegendElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLLegendElement)
- [`HTMLLinkElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLLinkElement)
- [`HTMLMapElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMapElement)
- [`HTMLMarqueeElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMarqueeElement) 
- [`HTMLMediaElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement)
- [`HTMLMenuElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMenuElement)
- [`HTMLMenuItemElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMenuItemElement) 
- [`HTMLMetaElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMetaElement)
- [`HTMLMeterElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMeterElement)
- [`HTMLModElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLModElement)
- [`HTMLOListElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLOListElement)
- [`HTMLObjectElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLObjectElement)
- [`HTMLOptGroupElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLOptGroupElement)
- [`HTMLOptionElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLOptionElement)
- [`HTMLOutputElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLOutputElement)
- [`HTMLParagraphElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLParagraphElement)
- [`HTMLParamElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLParamElement)
- [`HTMLPictureElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLPictureElement)
- [`HTMLPreElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLPreElement)
- [`HTMLProgressElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLProgressElement)
- [`HTMLQuoteElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLQuoteElement)
- [`HTMLScriptElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLScriptElement)
- [`HTMLSelectElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLSelectElement)
- [`HTMLSlotElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLSlotElement)
- [`HTMLSourceElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLSourceElement)
- [`HTMLSpanElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLSpanElement)
- [`HTMLStyleElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLStyleElement)
- [`HTMLTableCaptionElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLTableCaptionElement)
- [`HTMLTableCellElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLTableCellElement)
- [`HTMLTableColElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLTableColElement)
- [`HTMLTableElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLTableElement)
- [`HTMLTableRowElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLTableRowElement)
- [`HTMLTableSectionElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLTableSectionElement)
- [`HTMLTemplateElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLTemplateElement)
- [`HTMLTextAreaElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLTextAreaElement)
- [`HTMLTimeElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLTimeElement)
- [`HTMLTitleElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLTitleElement)
- [`HTMLTrackElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLTrackElement)
- [`HTMLUListElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLUListElement)
- [`HTMLUnknownElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLUnknownElement)
- [`HTMLVideoElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLVideoElement)

### Web app and browser integration interfaces



These interfaces offer access to the browser window and document that contain the HTML, as well as to the browser's state, available plugins (if any), and various configuration options.

- [`ApplicationCache`](https://developer.mozilla.org/en-US/docs/Web/API/ApplicationCache) 
- [`BarProp`](https://developer.mozilla.org/en-US/docs/Web/API/BarProp)
- [`External`](https://developer.mozilla.org/en-US/docs/Web/API/External) 
- [`Navigator`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator)
- [`Plugin`](https://developer.mozilla.org/en-US/docs/Web/API/Plugin) 
- [`PluginArray`](https://developer.mozilla.org/en-US/docs/Web/API/PluginArray) 
- `Window`

### Form support interfaces



These interfaces provide structure and functionality required by the elements used to create and manage forms, including the `<form>` and `<input>` elements.

- [`FormDataEvent`](https://developer.mozilla.org/en-US/docs/Web/API/FormDataEvent)
- [`HTMLFormControlsCollection`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormControlsCollection)
- [`HTMLOptionsCollection`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLOptionsCollection)
- [`RadioNodeList`](https://developer.mozilla.org/en-US/docs/Web/API/RadioNodeList)
- [`ValidityState`](https://developer.mozilla.org/en-US/docs/Web/API/ValidityState)

### Canvas and image interfaces



These interfaces represent objects used by the Canvas API as well as the `<img>` element and `<picture>` elements.

- [`CanvasGradient`](https://developer.mozilla.org/en-US/docs/Web/API/CanvasGradient)
- [`CanvasPattern`](https://developer.mozilla.org/en-US/docs/Web/API/CanvasPattern)
- [`CanvasRenderingContext2D`](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D)
- [`ImageBitmap`](https://developer.mozilla.org/en-US/docs/Web/API/ImageBitmap)
- [`ImageBitmapRenderingContext`](https://developer.mozilla.org/en-US/docs/Web/API/ImageBitmapRenderingContext)
- [`ImageData`](https://developer.mozilla.org/en-US/docs/Web/API/ImageData)
- [`OffscreenCanvas`](https://developer.mozilla.org/en-US/docs/Web/API/OffscreenCanvas)
- [`OffscreenCanvasRenderingContext2D`](https://developer.mozilla.org/en-US/docs/Web/API/OffscreenCanvasRenderingContext2D)
- [`Path2D`](https://developer.mozilla.org/en-US/docs/Web/API/Path2D)
- [`TextMetrics`](https://developer.mozilla.org/en-US/docs/Web/API/TextMetrics)

### Media interfaces



The media interfaces provide HTML access to the contents of the media elements: `<audio>` and `<video>`.

- [`AudioTrack`](https://developer.mozilla.org/en-US/docs/Web/API/AudioTrack)
- [`AudioTrackList`](https://developer.mozilla.org/en-US/docs/Web/API/AudioTrackList)
- [`MediaError`](https://developer.mozilla.org/en-US/docs/Web/API/MediaError)
- [`TextTrack`](https://developer.mozilla.org/en-US/docs/Web/API/TextTrack)
- [`TextTrackCue`](https://developer.mozilla.org/en-US/docs/Web/API/TextTrackCue)
- [`TextTrackCueList`](https://developer.mozilla.org/en-US/docs/Web/API/TextTrackCueList)
- [`TextTrackList`](https://developer.mozilla.org/en-US/docs/Web/API/TextTrackList)
- [`TimeRanges`](https://developer.mozilla.org/en-US/docs/Web/API/TimeRanges)
- [`TrackEvent`](https://developer.mozilla.org/en-US/docs/Web/API/TrackEvent)
- [`VideoTrack`](https://developer.mozilla.org/en-US/docs/Web/API/VideoTrack)
- [`VideoTrackList`](https://developer.mozilla.org/en-US/docs/Web/API/VideoTrackList)

### Drag and drop interfaces



These interfaces are used by the HTML Drag and Drop API to represent individual draggable (or dragged) items, groups of dragged or draggable items, and to handle the drag and drop process.

- [`DataTransfer`](https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer)
- [`DataTransferItem`](https://developer.mozilla.org/en-US/docs/Web/API/DataTransferItem)
- [`DataTransferItemList`](https://developer.mozilla.org/en-US/docs/Web/API/DataTransferItemList)
- [`DragEvent`](https://developer.mozilla.org/en-US/docs/Web/API/DragEvent)

### Page history interfaces



The History API interfaces let you access information about the browser's history, as well as to shift the browser's current tab forward and backward through that history.

- [`BeforeUnloadEvent`](https://developer.mozilla.org/en-US/docs/Web/API/BeforeUnloadEvent)
- [`HashChangeEvent`](https://developer.mozilla.org/en-US/docs/Web/API/HashChangeEvent)
- [`History`](https://developer.mozilla.org/en-US/docs/Web/API/History)
- [`Location`](https://developer.mozilla.org/en-US/docs/Web/API/Location)
- [`PageTransitionEvent`](https://developer.mozilla.org/en-US/docs/Web/API/PageTransitionEvent)
- [`PopStateEvent`](https://developer.mozilla.org/en-US/docs/Web/API/PopStateEvent)

### Web Components interfaces



These interfaces are used by the Web Components API to create and manage the available custom elements.

- [`CustomElementRegistry`](https://developer.mozilla.org/en-US/docs/Web/API/CustomElementRegistry)

### Miscellaneous and supporting interfaces



These supporting object types are used in a variety of ways in the HTML DOM API; in addition, `PromiseRejectionEvent` represents the event delivered when a JavaScript `Promise` is rejected.

- [`DOMStringList`](https://developer.mozilla.org/en-US/docs/Web/API/DOMStringList)
- [`DOMStringMap`](https://developer.mozilla.org/en-US/docs/Web/API/DOMStringMap)
- [`ErrorEvent`](https://developer.mozilla.org/en-US/docs/Web/API/ErrorEvent)
- [`HTMLAllCollection`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLAllCollection)
- [`MimeType`](https://developer.mozilla.org/en-US/docs/Web/API/MimeType)
- [`MimeTypeArray`](https://developer.mozilla.org/en-US/docs/Web/API/MimeTypeArray)
- [`PromiseRejectionEvent`](https://developer.mozilla.org/en-US/docs/Web/API/PromiseRejectionEvent)

### Interfaces belonging to other APIs



There are several interfaces which are technically defined in the HTML specification while actually being part of other APIs.

#### Web storage interfaces

The Web Storage API provides the ability for web sites to store data either temporarily or permanently on the user's device for later re-use.

- [`Storage`](https://developer.mozilla.org/en-US/docs/Web/API/Storage)
- [`StorageEvent`](https://developer.mozilla.org/en-US/docs/Web/API/StorageEvent)

#### Web Workers interfaces

These interfaces are used by the Web Workers API both to establish the ability for workers to interact with an app and its content, but also to support messaging between windows or apps.

- [`BroadcastChannel`](https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel)
- [`DedicatedWorkerGlobalScope`](https://developer.mozilla.org/en-US/docs/Web/API/DedicatedWorkerGlobalScope)
- [`MessageChannel`](https://developer.mozilla.org/en-US/docs/Web/API/MessageChannel)
- [`MessageEvent`](https://developer.mozilla.org/en-US/docs/Web/API/MessageEvent)
- [`MessagePort`](https://developer.mozilla.org/en-US/docs/Web/API/MessagePort)
- [`SharedWorker`](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker)
- [`SharedWorkerGlobalScope`](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorkerGlobalScope)
- [`Worker`](https://developer.mozilla.org/en-US/docs/Web/API/Worker)
- [`WorkerGlobalScope`](https://developer.mozilla.org/en-US/docs/Web/API/WorkerGlobalScope)
- [`WorkerLocation`](https://developer.mozilla.org/en-US/docs/Web/API/WorkerLocation)
- [`WorkerNavigator`](https://developer.mozilla.org/en-US/docs/Web/API/WorkerNavigator)

#### WebSocket interfaces

These interfaces, defined by the HTML specification, are used by the WebSocket API.

- [`CloseEvent`](https://developer.mozilla.org/en-US/docs/Web/API/CloseEvent)
- [`WebSocket`](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket)

#### Server-sent events interfaces

The `EventSource` interface represents the source which sent or is sending server-sent events.

- [`EventSource`](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)

## Examples

In this example, an `<input>` element's `input` event is monitored in order to update the state of a form's "submit" button based on whether or not a given field currently has a value.

#### JavaScript

```js
const nameField = document.getElementById("userName");
const sendButton = document.getElementById("sendButton")

sendButton.disabled = true;
nameField.focus();

nameField.addEventListener("input", event => {
  const elem = event.target;
  const valid = elem.value.length != 0;

  if (valid && sendButton.disabled) {
    sendButton.disabled = false;
  } else if (!valid && !sendButton.disabled) {
    sendButton.disabled = true;
  }
});
```

This code uses the `Document` interface's `getElementById()` method to get the DOM object representing the `<input>` elements whose IDs are `userName` and `sendButton`. With these, we can access the properties and methods that provide information about and grant control over these elements.

The `HTMLInputElement` object for the "Send" button's `disabled` property is set to `true`, which disables the "Send" button so it can't be clicked. In addition, the user name input field is made the active focus by calling the `focus()` method it inherits from `HTMLElement`.

Then `addEventListener()` is called to add a handler for the `input` event to the user name input. This code looks at the length of the current value of the input; if it's zero, then the "Send" button is disabled if it's not already disabled. Otherwise, the code ensures that the button is enabled.

With this in place, the "Send" button is always enabled whenever the user name input field has a value, and disabled when it's empty.

#### HTML

The HTML for the form looks like this:

```html
<p>Please provide the information below. Items marked with "*" are required.</p>
<form action="" method="get">
  <p>
    <label for="userName" required>Your name:</label>
    <input type="text" id="userName"> (*)
  </p>
  <p>
    <label for="email">Email:</label>
    <input type="email" id="userEmail">
  </p>
  <input type="submit" value="Send" id="sendButton">
</form>
```

