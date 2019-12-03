# HTMLAnchorElement

The **`HTMLAnchorElement`** interface represents hyperlink elements and provides special properties and methods (beyond those of the regular `HTMLElement` object interface that they inherit from) for manipulating the layout and presentation of such elements. This interface corresponds to `<a>` element; not to be confused with `<link>`, which is represented by `HTMLLinkElement`)

提示：`<link>`元素链接层叠样式表文件。

```js
class HTMLAnchorElement  extends HTMLElement {}
```



## Properties

Inherits properties from its parent, `HTMLElement`, and implements those from `HTMLHyperlinkElementUtils`.

- [`Element.accessKey`](https://developer.mozilla.org/en-US/docs/Web/API/Element/accessKey)

  Is a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) representing a single character that switches input focus to the hyperlink.

- [`HTMLAnchorElement.download`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLAnchorElement/download) 

  试验

- [`HTMLHyperlinkElementUtils.hash`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHyperlinkElementUtils/hash)

  Is a [`USVString`](https://developer.mozilla.org/en-US/docs/Web/API/USVString) representing the fragment identifier, including the leading hash mark ('`#`'), if any, in the referenced URL.

- [`HTMLHyperlinkElementUtils.host`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHyperlinkElementUtils/host)

  Is a [`USVString`](https://developer.mozilla.org/en-US/docs/Web/API/USVString) representing the hostname and port (if it's not the default port) in the referenced URL.

- [`HTMLHyperlinkElementUtils.hostname`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHyperlinkElementUtils/hostname)

  Is a [`USVString`](https://developer.mozilla.org/en-US/docs/Web/API/USVString) representing the hostname in the referenced URL.

- [`HTMLHyperlinkElementUtils.href`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHyperlinkElementUtils/href)

  Is a [`USVString`](https://developer.mozilla.org/en-US/docs/Web/API/USVString) that reflects the `href` HTML attribute, containing a valid URL of a linked resource.

- [`HTMLAnchorElement.hreflang`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLAnchorElement/hreflang)

  Is a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) that reflects the `hreflang` HTML attribute, indicating the language of the linked resource.

- [`HTMLAnchorElement.media`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLAnchorElement/media)

  Is a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) that reflects the `media` HTML attribute, indicating the intended media for the linked resource.

- [`HTMLHyperlinkElementUtils.password`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHyperlinkElementUtils/password)

  Is a [`USVString`](https://developer.mozilla.org/en-US/docs/Web/API/USVString) containing the password specified before the domain name.

- [`HTMLHyperlinkElementUtils.origin`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHyperlinkElementUtils/origin) Read only

  Returns a [`USVString`](https://developer.mozilla.org/en-US/docs/Web/API/USVString) containing the origin of the URL, that is its scheme, its domain and its port.

- [`HTMLHyperlinkElementUtils.pathname`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHyperlinkElementUtils/pathname)

  Is a [`USVString`](https://developer.mozilla.org/en-US/docs/Web/API/USVString) representing the path name component, if any, of the referenced URL.

- [`HTMLHyperlinkElementUtils.port`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHyperlinkElementUtils/port)

  Is a [`USVString`](https://developer.mozilla.org/en-US/docs/Web/API/USVString) representing the port component, if any, of the referenced URL.

- [`HTMLHyperlinkElementUtils.protocol`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHyperlinkElementUtils/protocol)

  Is a [`USVString`](https://developer.mozilla.org/en-US/docs/Web/API/USVString) representing the protocol component, including trailing colon ('`:`'), of the referenced URL.

- [`HTMLAnchorElement.referrerPolicy`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLAnchorElement/referrerPolicy) 

  试验

- [`HTMLAnchorElement.rel`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLAnchorElement/rel)

  Is a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) that reflects the `rel` HTML attribute, specifying the relationship of the target object to the linked object.

- [`HTMLAnchorElement.relList`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLAnchorElement/relList) Read only

  Returns a [`DOMTokenList`](https://developer.mozilla.org/en-US/docs/Web/API/DOMTokenList) that reflects the `rel` HTML attribute, as a list of tokens.

- [`HTMLHyperlinkElementUtils.search`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHyperlinkElementUtils/search)

  Is a [`USVString`](https://developer.mozilla.org/en-US/docs/Web/API/USVString) representing the search element, including leading question mark ('`?`'), if any, of the referenced URL.

- [`HTMLElement.tabindex`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/tabindex)

  Is a `long` containing the position of the element in the tabbing navigation order for the current document.

- [`HTMLAnchorElement.target`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLAnchorElement/target)

  Is a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) that reflects the `target` HTML attribute, indicating where to display the linked resource.

- [`HTMLAnchorElement.text`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLAnchorElement/text)

  Is a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) being a synonym for the [`Node.textContent`](https://developer.mozilla.org/en-US/docs/Web/API/Node/textContent) property.

- [`HTMLAnchorElement.type`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLAnchorElement/type)

  Is a [`DOMString`](https://developer.mozilla.org/en-US/docs/Web/API/DOMString) that reflects the `type` HTML attribute, indicating the MIME type of the linked resource.

- [`HTMLHyperlinkElementUtils.username`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHyperlinkElementUtils/username)

  Is a [`USVString`](https://developer.mozilla.org/en-US/docs/Web/API/USVString) containing the username specified before the domain name.

### Obsolete properties



- `HTMLAnchorElement.charset` 
- `HTMLAnchorElement.coords` 
- `HTMLAnchorElement.name` 
- `HTMLAnchorElement.rev` 
- `HTMLAnchorElement.shape` 

## Methods

Inherits methods from its parent, `HTMLElement`, and implements those from `HTMLHyperlinkElementUtils`.

- [`HTMLElement.blur()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/blur)

  Removes the keyboard focus from the current element.

- [`HTMLElement.focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus)

  Gives the keyboard focus to the current element.

- [`HTMLHyperlinkElementUtils.toString()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHyperlinkElementUtils/toString)

  Returns a [`USVString`](https://developer.mozilla.org/en-US/docs/Web/API/USVString) containing the whole URL. It is a synonym for [`HTMLHyperlinkElementUtils.href`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLHyperlinkElementUtils/href), though it can't be used to modify the value.

The `blur()` and `focus()` methods are inherited from `HTMLElement` from HTML5 on, but were defined on `HTMLAnchorElement` in DOM Level 2 HTML and earlier specifications.