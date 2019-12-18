DOM笔记

节点类型被过滤掉以后，实际使用有两种[`Node.nodeType`](https://developer.mozilla.org/en-US/docs/Web/API/Node/nodeType)节点：

```js
document.ELEMENT_NODE
document.TEXT_NODE
```

节点关系，每个节点有导航属性

```js
parentNode
childNodes
previousSibling
nextSibling
firstChild
lastChild
```

[`Document.createDocumentFragment()`](https://developer.mozilla.org/en-US/docs/Web/API/Document/createDocumentFragment)

[`Document.createElement()`](https://developer.mozilla.org/en-US/docs/Web/API/Document/createElement)

[`Document.createTextNode()`](https://developer.mozilla.org/en-US/docs/Web/API/Document/createTextNode)

[`Document.getElementsByClassName()`](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByClassName)

[`Document.getElementsByTagName()`](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByTagName)

[`document.getElementById(String id)`](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementById)

Returns an object reference to the identified element.

[`Document.querySelector()`](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector)

Returns the first Element node within the document, in document order, that matches the specified selectors.

[`Document.querySelectorAll()`](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelectorAll)

Returns a list of all the Element nodes within the document that match the specified selectors.

[`Document.getElementsByName()`](https://developer.mozilla.org/en-US/docs/Web/API/Document/getElementsByName)

Returns a list of elements with the given name.



```js
document.addEventListener('DOMContentLoaded' functionHandler);
```