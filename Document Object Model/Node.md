# Node

**`Node`** is an interface from which various types of DOM API objects inherit. This allows these types to be treated similarly; for example, inheriting the same set of methods or being tested in the same way.

All of the following interfaces inherit the **`Node`** interface's methods and properties: `Document`, `Element`, `Attr`, `CharacterData` (which `Text`, `Comment`, and `CDATASection` inherit), `ProcessingInstruction`, `DocumentFragment`, `DocumentType`, `Notation`, `Entity`, `EntityReference`

These interfaces may return `null` in certain cases where the methods and properties are not relevant. They may throw an exception — for example when adding children to a node type for which no children can exist.

```js
class Node extends EventTarget {}
```

## Properties

- `Node.childNodes` Read only

  Returns a live `NodeList` containing all the children of this node. `NodeList` being live means that if the children of the `Node` change, the `NodeList` object is automatically updated.

- `Node.firstChild` Read only

  Returns a `Node` representing the first direct child node of the node, or `null` if the node has no child.


- `Node.lastChild` Read only

  Returns a `Node` representing the last direct child node of the node, or `null` if the node has no child.

- `Node.nextSibling` Read only

  Returns a `Node` representing the next node in the tree, or `null` if there isn't such node.

- `Node.nodeName` Read only

  Returns a `DOMString` containing the name of the `Node`. The structure of the name will differ with the node type. E.g.. An `HTMLElement` will contain the name of the corresponding tag, like `'audio'` for an `HTMLAudioElement`, a `Text` node will have the `'#text'` string, or a `Document` node will have the `'#document'` string.

- `Node.nodeType` Read only

  Returns an `unsigned short` representing the type of the node. Possible values are:

```js
ELEMENT_NODE                1
ATTRIBUTE_NODE              2  // 不推荐
TEXT_NODE                   3
CDATA_SECTION_NODE          4
ENTITY_REFERENCE_NODE       5  // 不推荐
ENTITY_NODE                 6  // 不推荐
PROCESSING_INSTRUCTION_NODE 7
COMMENT_NODE                8
DOCUMENT_NODE               9
DOCUMENT_TYPE_NODE          10
DOCUMENT_FRAGMENT_NODE      11
NOTATION_NODE               12 // 不推荐
```

- `Node.nodeValue`

  Returns / Sets the value of the current node

- `Node.ownerDocument` Read only

  Returns the `Document` that this node belongs to. If the node is itself a document, returns `null`.

- `Node.parentNode` Read only

  Returns a `Node` that is the parent of this node. If there is no such node, like if this node is the top of the tree or if doesn't participate in a tree, this property returns `null`.

- `Node.previousSibling` Read only

  Returns a `Node` representing the previous node in the tree, or `null` if there isn't such node.



## Methods

- `Node.appendChild()`

  Adds the specified childNode argument as the last child to the current node. If the argument referenced an existing node on the DOM tree, the node will be detached from its current position and attached at the new position.

- `Node.cloneNode()`

  Clone a `Node`, and optionally, all of its contents. By default, it clones the content of the node.


- `Node.contains()`

  Returns a `Boolean` value indicating whether a node is a descendant of a given node or not.

- `Node.getRootNode()`

  Returns the context object's root which optionally includes the shadow root if it is available.

- `Node.hasChildNodes()`

  Returns a `Boolean` indicating if the element has any child nodes, or not.

- `Node.insertBefore()`

  Inserts a `Node` before the reference node as a child of a specified parent node.

- `Node.isEqualNode()`

  Returns a `Boolean` which indicates whether or not two nodes are of the same type and all their defining data points match.

- `Node.isSameNode()`

  Returns a `Boolean` value indicating whether or not the two nodes are the same (that is, they reference the same object).


- `Node.removeChild()`

  Removes a child node from the current element, which must be a child of the current node.

- `Node.replaceChild()`

  Replaces one child `Node` of the current one with the second one given in parameter.



## Examples

### Remove all children nested within a node

```js
function removeAllChildren(element){
  while(element.firstChild){
    element.removeChild(element.firstChild);
  }
}
```

#### Sample usage

```js
/* ... an alternative to document.body.innerHTML = "" ... */
removeAllChildren(document.body);
```

### Recurse through child nodes

The following function calls a function recursively for each node contained by a root node (including the root itself):

```js
function eachNode(rootNode, callback){
  if(!callback){
    const nodes = [];
    eachNode(rootNode, function(node){
      nodes.push(node);
    });
    return nodes;
  }

  if(false === callback(rootNode))
    return false;

  if(rootNode.hasChildNodes()){
    const nodes = rootNode.childNodes;
    for(let i = 0, l = nodes.length; i < l; ++i)
      if(false === eachNode(nodes[i], callback))
        return;
  }
}
```

#### Syntax

```
eachNode(rootNode, callback);
```

#### Description

Recursively calls a function for each descendant node of `rootNode` (including the root itself).

If `callback` is omitted, the function returns an `Array` instead, which contains `rootNode` and all nodes contained therein.

If `callback` is provided, and it returns `Boolean` `false` when called, the current recursion level is aborted, and the function resumes execution at the last parent's level. This can be used to abort loops once a node has been found (such as searching for a text node that contains a certain string).

#### Parameters

- `rootNode`

  The `Node` object whose descendants will be recursed through.

- `callback`

  An optional callback function that receives a `Node` as its only argument. If omitted, `eachNode` returns an `Array` of every node contained within `rootNode` (including the root itself).

#### Sample usage

The following example prints the `textContent` properties of each `` tag in a `` element named `"box"`:

```html
<div id="box">
  <span>Foo</span>
  <span>Bar</span>
  <span>Baz</span>
</div>
```

```js
const box = document.getElementById("box");
eachNode(box, function(node){
  if(null != node.textContent){
    console.log(node.textContent);
  }
});
```

The following strings will be displayed in the user's console:

```js
"\n\t", "Foo", "\n\t", "Bar", "\n\t", "Baz"
```

**Note:** Whitespace forms part of a `Text` node, meaning indentation and newlines form separate `Text` between the `Element` nodes.

#### Realistic usage

The following demonstrates a real-world use of the `eachNode` function: searching for text on a web-page. We use a wrapper function named `grep` to do the searching:

```js
function grep(parentNode, pattern){
  const matches = [];
  let endScan = false;

  eachNode(parentNode, function(node){
    if(endScan)
      return false;

    // Ignore anything which isn't a text node
    if(node.nodeType !== Node.TEXT_NODE)
      return;

    if("string" === typeof pattern){
      if(-1 !== node.textContent.indexOf(pattern))
        matches.push(node);
    }
    else if(pattern.test(node.textContent)){
      if(!pattern.global){
        endScan = true;
        matches = node;
      }
      else matches.push(node);
    }
  });

  return matches;
}
```

For example, to find `Text` nodes that contain typos:

```js
const typos = ["teh", "adn", "btu", "adress", "youre", "msitakes"];
const pattern = new RegExp("\\b(" + typos.join("|") + ")\\b", "gi");
const mistakes = grep(document.body, pattern);
console.log(mistakes);
```



# NodeList

**`NodeList`** objects are collections of [nodes](https://developer.mozilla.org/en-US/docs/Glossary/Node/DOM), usually returned by properties such as `Node.childNodes` and methods such as `document.querySelectorAll()`.

Although `NodeList` is not an `Array`, it is possible to iterate over it with `forEach()`. It can also be converted to a real `Array` using `Array.from()`.

However, some older browsers have not implemented `NodeList.forEach()` nor `Array.from()`. This can be circumvented by using `Array.prototype.forEach()` — see this document's Example.

In some cases, the `NodeList` is *live*, which means that changes in the DOM automatically update the collection. For example, `Node.childNodes` is live:

```js
var parent = document.getElementById('parent');
var child_nodes = parent.childNodes;
console.log(child_nodes.length); // let's assume "2"
parent.appendChild(document.createElement('div'));
console.log(child_nodes.length); // outputs "3"
```

In other cases, the `NodeList` is *static,* where any changes in the DOM does not affect the content of the collection. `document.querySelectorAll()` returns a static `NodeList`.

It's good to keep this distinction in mind when you choose how to iterate over the items in the `NodeList`, and if you should cache the list's length.

## Properties

- [`NodeList.length`](https://developer.mozilla.org/en-US/docs/Web/API/NodeList/length)

  The number of nodes in the `NodeList`.

## Methods

- [`NodeList.item()`](https://developer.mozilla.org/en-US/docs/Web/API/NodeList/item)

  Returns an item in the list by its index, or `null` if the index is out-of-bounds.

  An alternative to accessing `nodeList[i]` (which instead returns `undefined` when `i` is out-of-bounds). This is mostly useful for non-JavaScript languages DOM implementations.

- [`NodeList.entries()`](https://developer.mozilla.org/en-US/docs/Web/API/NodeList/entries)

  Returns an `iterator`, allowing code to go through all key/value pairs contained in the collection. (In this case, the keys are numbers starting from 0 and the values are nodes.)

- [`NodeList.forEach()`](https://developer.mozilla.org/en-US/docs/Web/API/NodeList/forEach)

  Executes a provided function once per `NodeList` element, passing the element as an argument to the function.

- [`NodeList.keys()`](https://developer.mozilla.org/en-US/docs/Web/API/NodeList/keys)

  Returns an `iterator`, allowing code to go through all the keys of the key/value pairs contained in the collection. (In this case, the keys are numbers starting from 0.)

- [`NodeList.values()`](https://developer.mozilla.org/en-US/docs/Web/API/NodeList/values)

  Returns an `iterator` allowing code to go through all values (nodes) of the key/value pairs contained in the collection.

## Example

It's possible to loop over the items in a `NodeList` using a for loop:

```js
for (var i = 0; i < myNodeList.length; i++) {
  var item = myNodeList[i];
}
```

**Don't use `for...in` or `for each...in` to enumerate the items in `NodeList`s**, since they will *also* enumerate its `length` and `item` properties and cause errors if your script assumes it only has to deal with `element` objects. Also, `for..in` is not guaranteed to visit the properties in any particular order.

`for...of` loops **will** loop over `NodeList` objects correctly:

```js
var list = document.querySelectorAll('input[type=checkbox]');
for (var checkbox of list) {
  checkbox.checked = true;
}
```

Recent browsers also support iterator methods (`forEach()`) as well as `entries()`, `values()`, and `keys()`.

There is also an Internet Explorer-compatible way to use `Array.prototype.forEach` for iteration:

```js
var list = document.querySelectorAll('input[type=checkbox]');
Array.prototype.forEach.call(list, function (checkbox) {
  checkbox.checked = true;
});
```