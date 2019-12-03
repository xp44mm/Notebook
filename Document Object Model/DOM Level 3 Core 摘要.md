# DOM Level 3 Core 摘要

### 1.1.1 The DOM Structure Model

The DOM presents documents as a hierarchy of `Node` objects that also implement other, more specialized interfaces. Some types of nodes may have child nodes of various types, and others are leaf nodes that cannot have anything below them in the document structure. For XML and HTML, the node types, and which node types they may have as children, are as follows:

```js
let children = {
    Document             : [
    Element/*maximum of one*/, 
    ProcessingInstruction, 
    Comment, 
    DocumentType/*maximum of one*/],
    DocumentFragment     : [
    Element, 
    ProcessingInstruction, 
    Comment, Text, 
    CDATASection, EntityReference],
    DocumentType         : [],
    EntityReference      : [Element, ProcessingInstruction, Comment, Text, CDATASection, EntityReference],
    Element              : [Element, Text, Comment, ProcessingInstruction, CDATASection, EntityReference],
    Attr                 : [Text, EntityReference],
    ProcessingInstruction: [],
    Comment              : [],
    Text                 : [],
    CDATASection         : [],
    Entity               : [Element, ProcessingInstruction, Comment, Text, CDATASection, EntityReference],
    Notation             : [],
}
```

空数组表示：no child，有许多节点已经废弃，或者不推荐。

The DOM also specifies a `NodeList` interface to handle ordered lists of `Node`s, such as the children of a `Node`, or the elements returned by the `Element.getElementsByTagNameNS(namespaceURI, localName)` method, and also a `NamedNodeMap` interface to handle unordered sets of nodes referenced by their name attribute, such as the attributes of an `Element`. `NodeList` and `NamedNodeMap` objects in the DOM are *live*; that is, changes to the underlying document structure are reflected in all relevant `NodeList` and `NamedNodeMap` objects. For example, if a DOM user gets a `NodeList` object containing the children of an `Element`, then subsequently adds more children to that element(or removes children, or modifies them), those changes are automatically reflected in the `NodeList`, without further action on the user's part. Likewise, changes to a `Node` in the tree are reflected in all references to that `Node` in `NodeList` and `NamedNodeMap` objects.

Finally, the interfaces `Text`, `Comment`, and `CDATASection` all inherit from the `CharacterData` interface.

### 1.1.3 Naming Conventions

While it would be nice to have attribute and method names that are short, informative, internally consistent, and familiar to users of similar APIs, the names also should not clash with the names in legacy APIs supported by DOM implementations. Furthermore, both OMG IDL and ECMAScript have significant limitations in their ability to disambiguate names from different namespaces that make it difficult to avoid naming conflicts with short, familiar names. So, DOM names tend to be long and descriptive in order to be unique across all environments.

The Working Group has also attempted to be internally consistent in its use of various terms, even though these may not be common distinctions in other APIs. For example, the DOM API uses the method name `"remove"` when the method changes the structural model, and the method name `"delete"` when the method gets rid of something inside the structure model. The thing that is deleted is not returned. The thing that is removed may be returned, when it makes sense to return it.

### 1.1.4 Inheritance vs. Flattened Views of the API

The DOM Core APIs present two somewhat different sets of interfaces to an XML/HTML document: one presenting an "object oriented" approach with a hierarchy of inheritance, and a "simplified" view that allows all manipulation to be done via the `Node` interface without requiring casts (in Java and other C-like languages) or query interface calls in COM environments. These operations are fairly expensive in Java and COM, and the DOM may be used in performance-critical environments, so we allow significant functionality using just the `Node` interface. Because many other users will find the inheritance hierarchy easier to understand than the "everything is a `Node`" approach to the DOM, we also support the full higher-level interfaces for those who prefer a more object-oriented API.

In practice, this means that there is a certain amount of redundancy in the API. The Working Group considers the "inheritance" approach the primary view of the API, and the full set of functionality on `Node` to be "extra" functionality that users may employ, but that does not eliminate the need for methods on other interfaces that an object-oriented analysis would dictate. (Of course, when the O-O analysis yields an attribute or method that is identical to one on the `Node` interface, we don't specify a completely redundant one.) Thus, even though there is a generic `Node.nodeName` attribute on the `Node` interface, there is still a `Element.tagName` attribute on the `Element` interface; these two attributes must contain the same value, but the it is worthwhile to support both, given the different constituencies the DOM API must satisfy.

## 1.2 Basic Types

To ensure interoperability, this specification specifies the following basic types used in various DOM modules. Even though the DOM uses the basic types in the interfaces, bindings may use different types and normative bindings are only given for Java and ECMAScript in this specification.

### 1.2.1 The `DOMString` Type

The `DOMString` type is used to store Unicode characters as a sequence of 16-bit units using UTF-16 as defined in Unicode and Amendment 1 of ISO/IEC 10646.

For ECMAScript, `DOMString` is bound to the `String` type because both languages also use UTF-16 as their encoding.

### 1.2.2 The `DOMTimeStamp` Type

The `DOMTimeStamp` type is used to store an absolute or relative time.

For ECMAScript, `DOMTimeStamp` is bound to the `Date` type because the range of the `integer` type is too small.

### 1.2.3 The `DOMUserData` Type

The `DOMUserData` type is used to store application data.

For ECMAScript, `DOMUserData` is bound to `any type`.

### 1.2.4 The `DOMObject` Type

The `DOMObject` type is used to represent an object.

For ECMAScript, `DOMObject` is bound to the `Object` type.


`XxxList`可以理解为泛型`Array<Xxx>`类型。因为JavaScript没有泛型。





