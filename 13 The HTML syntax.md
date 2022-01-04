# https://html.spec.whatwg.org

## 13 The HTML syntax


This section only describes the rules for resources labeled with an HTML MIME type. Rules for XML resources are discussed in the section below entitled "The XML syntax".

### 13.1 Writing HTML documents

*This section only applies to documents, authoring tools, and markup generators. In particular, it does not apply to conformance checkers; conformance checkers must use the requirements given in the next section ("parsing HTML documents").*

Documents must consist of the following parts, in the given order:

1. Optionally, a single U+FEFF BYTE ORDER MARK (BOM) character.
2. Any number of [comments](#syntax-comments) and `/\s/`.
3. A [DOCTYPE](#syntax-doctype).
4. Any number of [comments](#syntax-comments) and `/\s/`.
5. The [document element](https://dom.spec.whatwg.org/#document-element), in the form of an `html` [element](#syntax-elements).
6. Any number of [comments](#syntax-comments) and `/\s/`.

The various types of content mentioned above are described in the next few sections.

In addition, there are some restrictions on how [character encoding declarations](https://html.spec.whatwg.org/multipage/semantics.html#character-encoding-declaration) are to be serialized, as discussed in the section on that topic.

`/\s/` before the `html` element, at the start of the `html` element and before the `head` element, will be dropped when the document is parsed; `/\s/` *after* the `html` element will be parsed as if it were at the end of the `body` element. Thus, `/\s/` around the [document element](https://dom.spec.whatwg.org/#document-element) does not round-trip.

It is suggested that newlines be inserted after the DOCTYPE, after any comments that are before the document element, after the `html` element's start tag (if it is not [omitted](#syntax-tag-omission)), and after any comments that are inside the `html` element but before the `head` element.

Many strings in the HTML syntax (e.g. the names of elements and their attributes) are case-insensitive, but only for upper alphas and lower alphas. For convenience, in this section this is just referred to as "case-insensitive".

#### 13.1.1 The DOCTYPE

A DOCTYPE is a required preamble.

DOCTYPEs are required for legacy reasons. When omitted, browsers tend to use a different rendering mode that is incompatible with some specifications. Including the DOCTYPE in a document ensures that the browser makes a best-effort attempt at following the relevant specifications.

A DOCTYPE must consist of the following components, in this order:

1. A string that is an case-insensitive match for the string "`<!DOCTYPE`".
2. One or more `/\s/`.
3. A string that is an case-insensitive match for the string "`html`".
4. Optionally, a [DOCTYPE legacy string](#doctype-legacy-string).
5. Zero or more `/\s/`.
6. A U+003E GREATER-THAN SIGN character (>).

In other words, `<!DOCTYPE html>`, case-insensitively.

------

For the purposes of HTML generators that cannot output HTML markup with the short DOCTYPE "`<!DOCTYPE html>`", a DOCTYPE legacy string may be inserted into the DOCTYPE (in the position defined above). This string must consist of:

1. One or more `/\s/`.
2. A string that is an case-insensitive match for the string "`SYSTEM`".
3. One or more `/\s/`.
4. A U+0022 QUOTATION MARK or U+0027 APOSTROPHE character (the *quote mark*).
5. The literal string "`about:legacy-compat`".
6. A matching U+0022 QUOTATION MARK or U+0027 APOSTROPHE character (i.e. the same character as in the earlier step labeled *quote mark*).

In other words, `<!DOCTYPE html SYSTEM "about:legacy-compat">` or `<!DOCTYPE html SYSTEM 'about:legacy-compat'>`, case-insensitively except for the part in single or double quotes.

The [DOCTYPE legacy string](#doctype-legacy-string) should not be used unless the document is generated from a system that cannot output the shorter string.

#### 13.1.2 Elements

There are six different kinds of elements: [void elements](#void-elements), [the `template` element](#the-template-element-2), [raw text elements](#raw-text-elements), [escapable raw text elements](#escapable-raw-text-elements), [foreign elements](#foreign-elements), and [normal elements](#normal-elements).

- Void elements

  `area`, `base`, `br`, `col`, `embed`, `hr`, `img`, `input`, `link`, `meta`, `param`, `source`, `track`, `wbr`

- The `template` elements

  `template`

- Raw text elements

  `script`, `style`

- Escapable raw text elements

  `textarea`, `title`

- Foreign elements

  Elements from the [MathML namespace](https://infra.spec.whatwg.org/#mathml-namespace) and the [SVG namespace](https://infra.spec.whatwg.org/#svg-namespace).

- Normal elements

  All other allowed [HTML elements](https://html.spec.whatwg.org/multipage/infrastructure.html#html-elements) are normal elements.

Tags are used to delimit the start and end of elements in the markup. [Raw text](#raw-text-elements), [escapable raw text](#escapable-raw-text-elements), and [normal](#normal-elements) elements have a [start tag](#syntax-start-tag) to indicate where they begin, and an [end tag](#syntax-end-tag) to indicate where they end. The start and end tags of certain [normal elements](#normal-elements) can be [omitted](#syntax-tag-omission), as described below in the section on [optional tags](#syntax-tag-omission). Those that cannot be omitted must not be omitted. [Void elements](#void-elements) only have a start tag; end tags must not be specified for [void elements](#void-elements). [Foreign elements](#foreign-elements) must either have a start tag and an end tag, or a start tag that is marked as self-closing, in which case they must not have an end tag.

The [contents](https://html.spec.whatwg.org/multipage/dom.html#concept-html-contents) of the element must be placed between just after the start tag (which [might be implied, in certain cases](#syntax-tag-omission)) and just before the end tag (which again, [might be implied in certain cases](#syntax-tag-omission)). The exact allowed contents of each individual element depend on the [content model](https://html.spec.whatwg.org/multipage/dom.html#content-models) of that element, as described earlier in this specification. Elements must not contain content that their content model disallows. In addition to the restrictions placed on the contents by those content models, however, the five types of elements have additional *syntactic* requirements.

[Void elements](#void-elements) can't have any contents (since there's no end tag, no content can be put between the start tag and the end tag).

[The `template` element](#the-template-element-2) can have [template contents](https://html.spec.whatwg.org/multipage/scripting.html#template-contents), but such [template contents](https://html.spec.whatwg.org/multipage/scripting.html#template-contents) are not children of the `template` element itself. Instead, they are stored in a `DocumentFragment` associated with a different `Document` — without a [browsing context](https://html.spec.whatwg.org/multipage/browsers.html#browsing-context) — so as to avoid the `template` contents interfering with the main `Document`. The markup for the [template contents](https://html.spec.whatwg.org/multipage/scripting.html#template-contents) of a `template` element is placed just after the `template` element's start tag and just before `template` element's end tag (as with other elements), and may consist of any [text](#syntax-text), [character references](#syntax-charref), [elements](#syntax-elements), and [comments](#syntax-comments), but the text must not contain the character U+003C LESS-THAN SIGN (<) or an [ambiguous ampersand](#syntax-ambiguous-ampersand).

[Raw text elements](#raw-text-elements) can have [text](#syntax-text), though it has [restrictions](#cdata-rcdata-restrictions) described below.

[Escapable raw text elements](#escapable-raw-text-elements) can have [text](#syntax-text) and [character references](#syntax-charref), but the text must not contain an [ambiguous ampersand](#syntax-ambiguous-ampersand). There are also [further restrictions](#cdata-rcdata-restrictions) described below.

[Foreign elements](#foreign-elements) whose start tag is marked as self-closing can't have any contents (since, again, as there's no end tag, no content can be put between the start tag and the end tag). [Foreign elements](#foreign-elements) whose start tag is *not* marked as self-closing can have [text](#syntax-text), [character references](#syntax-charref), [CDATA sections](#syntax-cdata), other [elements](#syntax-elements), and [comments](#syntax-comments), but the text must not contain the character U+003C LESS-THAN SIGN (<) or an [ambiguous ampersand](#syntax-ambiguous-ampersand).

The HTML syntax does not support namespace declarations, even in [foreign elements](#foreign-elements).

For instance, consider the following HTML fragment:

```html
<p>
 <svg>
  <metadata>
   <!-- this is invalid -->
   <cdr:license xmlns:cdr="https://www.example.com/cdr/metadata" name="MIT"/>
  </metadata>
 </svg>
</p>
```

The innermost element, `cdr:license`, is actually in the SVG namespace, as the "`xmlns:cdr`" attribute has no effect (unlike in XML). In fact, as the comment in the fragment above says, the fragment is actually non-conforming. This is because SVG 2 does not define any elements called "`cdr:license`" in the SVG namespace.

[Normal elements](#normal-elements) can have [text](#syntax-text), [character references](#syntax-charref), other [elements](#syntax-elements), and [comments](#syntax-comments), but the text must not contain the character U+003C LESS-THAN SIGN (<) or an [ambiguous ampersand](#syntax-ambiguous-ampersand). Some [normal elements](#normal-elements) also have [yet more restrictions](#element-restrictions) on what content they are allowed to hold, beyond the restrictions imposed by the content model and those described in this paragraph. Those restrictions are described below.

Tags contain a tag name, giving the element's name. HTML elements all have names that only use [ASCII alphanumerics](https://infra.spec.whatwg.org/#ascii-alphanumeric). In the HTML syntax, tag names, even those for [foreign elements](#foreign-elements), may be written with any mix of lower- and uppercase letters that, when converted to all-lowercase, matches the element's tag name; tag names are case-insensitive.

##### 13.1.2.1 Start tags

Start tags must have the following format:

1. The first character of a start tag must be a U+003C LESS-THAN SIGN character (<).
2. The next few characters of a start tag must be the element's [tag name](#syntax-tag-name).
3. If there are to be any attributes in the next step, there must first be one or more `/\s/`.
4. Then, the start tag may have a number of attributes, the [syntax for which](#syntax-attributes) is described below. Attributes must be separated from each other by one or more `/\s/`.
5. After the attributes, or after the [tag name](#syntax-tag-name) if there are no attributes, there may be one or more `/\s/`. (Some attributes are required to be followed by a space. See the [attributes section](#syntax-attributes) below.)
6. Then, if the element is one of the [void elements](#void-elements), or if the element is a [foreign element](#foreign-elements), then there may be a single U+002F SOLIDUS character (/). This character has no effect on [void elements](#void-elements), but on [foreign elements](#foreign-elements) it marks the start tag as self-closing.
7. Finally, start tags must be closed by a U+003E GREATER-THAN SIGN character (>).

##### 13.1.2.2 End tags

End tags must have the following format:

1. The first character of an end tag must be a U+003C LESS-THAN SIGN character (<).
2. The second character of an end tag must be a U+002F SOLIDUS character (/).
3. The next few characters of an end tag must be the element's [tag name](#syntax-tag-name).
4. After the tag name, there may be one or more `/\s/`.
5. Finally, end tags must be closed by a U+003E GREATER-THAN SIGN character (>).

##### 13.1.2.3 Attributes

Attributes for an element are expressed inside the element's start tag.

Attributes have a name and a value. Attribute names must consist of one or more characters other than [controls](https://infra.spec.whatwg.org/#control), U+0020 SPACE, U+0022 ("), U+0027 ('), U+003E (>), U+002F (/), U+003D (=), and [noncharacters](https://infra.spec.whatwg.org/#noncharacter). In the HTML syntax, attribute names, even those for [foreign elements](#foreign-elements), may be written with any mix of [ASCII lower](https://infra.spec.whatwg.org/#ascii-lower-alpha) and upper alphas.

Attribute values are a mixture of [text](#syntax-text) and [character references](#syntax-charref), except with the additional restriction that the text cannot contain an [ambiguous ampersand](#syntax-ambiguous-ampersand).

Attributes can be specified in four different ways:

- Empty attribute syntax

  Just the [attribute name](#syntax-attribute-name). The value is implicitly the empty string.In the following example, the `disabled` attribute is given with the empty attribute syntax:`<input *disabled*>`If an attribute using the empty attribute syntax is to be followed by another attribute, then there must be `/\s/` separating the two.

- Unquoted attribute value syntax

  The [attribute name](#syntax-attribute-name), followed by zero or more `/\s/`, followed by a single U+003D EQUALS SIGN character, followed by zero or more `/\s/`, followed by the [attribute value](#syntax-attribute-value), which, in addition to the requirements given above for attribute values, must not contain any literal `/\s/`, any U+0022 QUOTATION MARK characters ("), U+0027 APOSTROPHE characters ('), U+003D EQUALS SIGN characters (=), U+003C LESS-THAN SIGN characters (<), U+003E GREATER-THAN SIGN characters (>), or U+0060 GRAVE ACCENT characters (\`), and must not be the empty string. In the following example, the `value` attribute is given with the unquoted attribute value syntax:`<input *value=yes*>`If an attribute using the unquoted attribute syntax is to be followed by another attribute or by the optional U+002F SOLIDUS character (/) allowed in step 6 of the [start tag](#syntax-start-tag) syntax above, then there must be `/\s/` separating the two.

- Single-quoted attribute value syntax

  The [attribute name](#syntax-attribute-name), followed by zero or more `/\s/`, followed by a single U+003D EQUALS SIGN character, followed by zero or more `/\s/`, followed by a single U+0027 APOSTROPHE character ('), followed by the [attribute value](#syntax-attribute-value), which, in addition to the requirements given above for attribute values, must not contain any literal U+0027 APOSTROPHE characters ('), and finally followed by a second single U+0027 APOSTROPHE character (').In the following example, the `type` attribute is given with the single-quoted attribute value syntax:`<input *type='checkbox'*>`If an attribute using the single-quoted attribute syntax is to be followed by another attribute, then there must be `/\s/` separating the two.

- Double-quoted attribute value syntax

  The [attribute name](#syntax-attribute-name), followed by zero or more `/\s/`, followed by a single U+003D EQUALS SIGN character, followed by zero or more `/\s/`, followed by a single U+0022 QUOTATION MARK character ("), followed by the [attribute value](#syntax-attribute-value), which, in addition to the requirements given above for attribute values, must not contain any literal U+0022 QUOTATION MARK characters ("), and finally followed by a second single U+0022 QUOTATION MARK character (").In the following example, the `name` attribute is given with the double-quoted attribute value syntax:`<input *name="be evil"*>`If an attribute using the double-quoted attribute syntax is to be followed by another attribute, then there must be `/\s/` separating the two.

There must never be two or more attributes on the same start tag whose names are an case-insensitive match for each other.

------

When a [foreign element](#foreign-elements) has one of the namespaced attributes given by the local name and namespace of the first and second cells of a row from the following table, it must be written using the name given by the third cell from the same row.

| Local name | Namespace                                                    | Attribute name  |
| ---------- | ------------------------------------------------------------ | --------------- |
| `actuate`  | [XLink namespace](https://infra.spec.whatwg.org/#xlink-namespace) | `xlink:actuate` |
| `arcrole`  | [XLink namespace](https://infra.spec.whatwg.org/#xlink-namespace) | `xlink:arcrole` |
| `href`     | [XLink namespace](https://infra.spec.whatwg.org/#xlink-namespace) | `xlink:href`    |
| `role`     | [XLink namespace](https://infra.spec.whatwg.org/#xlink-namespace) | `xlink:role`    |
| `show`     | [XLink namespace](https://infra.spec.whatwg.org/#xlink-namespace) | `xlink:show`    |
| `title`    | [XLink namespace](https://infra.spec.whatwg.org/#xlink-namespace) | `xlink:title`   |
| `type`     | [XLink namespace](https://infra.spec.whatwg.org/#xlink-namespace) | `xlink:type`    |
| `lang`     | [XML namespace](https://infra.spec.whatwg.org/#xml-namespace) | `xml:lang`      |
| `space`    | [XML namespace](https://infra.spec.whatwg.org/#xml-namespace) | `xml:space`     |
| `xmlns`    | [XMLNS namespace](https://infra.spec.whatwg.org/#xmlns-namespace) | `xmlns`         |
| `xlink`    | [XMLNS namespace](https://infra.spec.whatwg.org/#xmlns-namespace) | `xmlns:xlink`   |

No other namespaced attribute can be expressed in [the HTML syntax](#syntax).

Whether the attributes in the table above are conforming or not is defined by other specifications (e.g. SVG 2 and MathML); this section only describes the syntax rules if the attributes are serialized using the HTML syntax.

##### 13.1.2.4 Optional tags

Certain tags can be omitted.

Omitting an element's [start tag](#syntax-start-tag) in the situations described below does not mean the element is not present; it is implied, but it is still there. For example, an HTML document always has a root `html` element, even if the string `<html>` doesn't appear anywhere in the markup.

An `html` element's [start tag](#syntax-start-tag) may be omitted if the first thing inside the `html` element is not a [comment](#syntax-comments).

For example, in the following case it's ok to remove the "`<html>`" tag:

```html
<!DOCTYPE HTML>
<html>
  <head>
    <title>Hello</title>
  </head>
  <body>
    <p>Welcome to this example.</p>
  </body>
</html>
```

Doing so would make the document look like this:

```html
<!DOCTYPE HTML>

  <head>
    <title>Hello</title>
  </head>
  <body>
    <p>Welcome to this example.</p>
  </body>
</html>
```

This has the exact same DOM. In particular, note that whitespace around the [document element](https://dom.spec.whatwg.org/#document-element) is ignored by the parser. The following example would also have the exact same DOM:

```html
<!DOCTYPE HTML><head>
    <title>Hello</title>
  </head>
  <body>
    <p>Welcome to this example.</p>
  </body>
</html>
```

However, in the following example, removing the start tag moves the comment to before the `html` element:

```html
<!DOCTYPE HTML>
<html>
  <!-- where is this comment in the DOM? -->
  <head>
    <title>Hello</title>
  </head>
  <body>
    <p>Welcome to this example.</p>
  </body>
</html>
```

With the tag removed, the document actually turns into the same as this:

```html
<!DOCTYPE HTML>
<!-- where is this comment in the DOM? -->
<html>
  <head>
    <title>Hello</title>
  </head>
  <body>
    <p>Welcome to this example.</p>
  </body>
</html>
```

This is why the tag can only be removed if it is not followed by a comment: removing the tag when there is a comment there changes the document's resulting parse tree. Of course, if the position of the comment does not matter, then the tag can be omitted, as if the comment had been moved to before the start tag in the first place.

An `html` element's [end tag](#syntax-end-tag) may be omitted if the `html` element is not immediately followed by a [comment](#syntax-comments).

A `head` element's [start tag](#syntax-start-tag) may be omitted if the element is empty, or if the first thing inside the `head` element is an element.

A `head` element's [end tag](#syntax-end-tag) may be omitted if the `head` element is not immediately followed by `/\s/` or a [comment](#syntax-comments).

A `body` element's [start tag](#syntax-start-tag) may be omitted if the element is empty, or if the first thing inside the `body` element is not `/\s/` or a [comment](#syntax-comments), except if the first thing inside the `body` element is a `meta`, `link`, `script`, `style`, or `template` element.

A `body` element's [end tag](#syntax-end-tag) may be omitted if the `body` element is not immediately followed by a [comment](#syntax-comments).

Note that in the example above, the `head` element start and end tags, and the `body` element start tag, can't be omitted, because they are surrounded by whitespace:

```html
<!DOCTYPE HTML>
<html>
  <head>
    <title>Hello</title>
  </head>
  <body>
    <p>Welcome to this example.</p>
  </body>
</html>
```

(The `body` and `html` element end tags could be omitted without trouble; any spaces after those get parsed into the `body` element anyway.)

Usually, however, whitespace isn't an issue. If we first remove the whitespace we don't care about:

```html
<!DOCTYPE HTML><html><head><title>Hello</title></head><body><p>Welcome to this example.</p></body></html>
```

Then we can omit a number of tags without affecting the DOM:

```html
<!DOCTYPE HTML><title>Hello</title><p>Welcome to this example.</p>
```

At that point, we can also add some whitespace back:

```html
<!DOCTYPE HTML>
<title>Hello</title>
<p>Welcome to this example.</p>
```

This would be equivalent to this document, with the omitted tags shown in their parser-implied positions; the only whitespace text node that results from this is the newline at the end of the `head` element:

```html
<!DOCTYPE HTML>
<html><head><title>Hello</title>
</head><body><p>Welcome to this example.</p></body></html>
```

An `li` element's [end tag](#syntax-end-tag) may be omitted if the `li` element is immediately followed by another `li` element or if there is no more content in the parent element.

A `dt` element's [end tag](#syntax-end-tag) may be omitted if the `dt` element is immediately followed by another `dt` element or a `dd` element.

A `dd` element's [end tag](#syntax-end-tag) may be omitted if the `dd` element is immediately followed by another `dd` element or a `dt` element, or if there is no more content in the parent element.

A `p` element's [end tag](#syntax-end-tag) may be omitted if the `p` element is immediately followed by an `address`, `article`, `aside`, `blockquote`, `details`, `div`, `dl`, `fieldset`, `figcaption`, `figure`, `footer`, `form`, `h1`, `h2`, `h3`, `h4`, `h5`, `h6`, `header`, `hgroup`, `hr`, `main`, `menu`, `nav`, `ol`, `p`, `pre`, `section`, `table`, or `ul` element, or if there is no more content in the parent element and the parent element is an [HTML element](https://html.spec.whatwg.org/multipage/infrastructure.html#html-elements) that is not an `a`, `audio`, `del`, `ins`, `map`, `noscript`, or `video` element, or an [autonomous custom element](https://html.spec.whatwg.org/multipage/custom-elements.html#autonomous-custom-element).

We can thus simplify the earlier example further:

```html
<!DOCTYPE HTML><title>Hello</title><p>Welcome to this example.
```

An `rt` element's [end tag](#syntax-end-tag) may be omitted if the `rt` element is immediately followed by an `rt` or `rp` element, or if there is no more content in the parent element.

An `rp` element's [end tag](#syntax-end-tag) may be omitted if the `rp` element is immediately followed by an `rt` or `rp` element, or if there is no more content in the parent element.

An `optgroup` element's [end tag](#syntax-end-tag) may be omitted if the `optgroup` element is immediately followed by another `optgroup` element, or if there is no more content in the parent element.

An `option` element's [end tag](#syntax-end-tag) may be omitted if the `option` element is immediately followed by another `option` element, or if it is immediately followed by an `optgroup` element, or if there is no more content in the parent element.

A `colgroup` element's [start tag](#syntax-start-tag) may be omitted if the first thing inside the `colgroup` element is a `col` element, and if the element is not immediately preceded by another `colgroup` element whose [end tag](#syntax-end-tag) has been omitted. (It can't be omitted if the element is empty.)

A `colgroup` element's [end tag](#syntax-end-tag) may be omitted if the `colgroup` element is not immediately followed by `/\s/` or a [comment](#syntax-comments).

A `caption` element's [end tag](#syntax-end-tag) may be omitted if the `caption` element is not immediately followed by `/\s/` or a [comment](#syntax-comments).

A `thead` element's [end tag](#syntax-end-tag) may be omitted if the `thead` element is immediately followed by a `tbody` or `tfoot` element.

A `tbody` element's [start tag](#syntax-start-tag) may be omitted if the first thing inside the `tbody` element is a `tr` element, and if the element is not immediately preceded by a `tbody`, `thead`, or `tfoot` element whose [end tag](#syntax-end-tag) has been omitted. (It can't be omitted if the element is empty.)

A `tbody` element's [end tag](#syntax-end-tag) may be omitted if the `tbody` element is immediately followed by a `tbody` or `tfoot` element, or if there is no more content in the parent element.

A `tfoot` element's [end tag](#syntax-end-tag) may be omitted if there is no more content in the parent element.

A `tr` element's [end tag](#syntax-end-tag) may be omitted if the `tr` element is immediately followed by another `tr` element, or if there is no more content in the parent element.

A `td` element's [end tag](#syntax-end-tag) may be omitted if the `td` element is immediately followed by a `td` or `th` element, or if there is no more content in the parent element.

A `th` element's [end tag](#syntax-end-tag) may be omitted if the `th` element is immediately followed by a `td` or `th` element, or if there is no more content in the parent element.

The ability to omit all these table-related tags makes table markup much terser.

Take this example:

```html
<table>
 <caption>37547 TEE Electric Powered Rail Car Train Functions (Abbreviated)</caption>
 <colgroup><col><col><col></colgroup>
 <thead>
  <tr>
   <th>Function</th>
   <th>Control Unit</th>
   <th>Central Station</th>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td>Headlights</td>
   <td>✔</td>
   <td>✔</td>
  </tr>
  <tr>
   <td>Interior Lights</td>
   <td>✔</td>
   <td>✔</td>
  </tr>
  <tr>
   <td>Electric locomotive operating sounds</td>
   <td>✔</td>
   <td>✔</td>
  </tr>
  <tr>
   <td>Engineer's cab lighting</td>
   <td></td>
   <td>✔</td>
  </tr>
  <tr>
   <td>Station Announcements - Swiss</td>
   <td></td>
   <td>✔</td>
  </tr>
 </tbody>
</table>
```

The exact same table, modulo some whitespace differences, could be marked up as follows:

```html
<table>
 <caption>37547 TEE Electric Powered Rail Car Train Functions (Abbreviated)
 <colgroup><col><col><col>
 <thead>
  <tr>
   <th>Function
   <th>Control Unit
   <th>Central Station
 <tbody>
  <tr>
   <td>Headlights
   <td>✔
   <td>✔
  <tr>
   <td>Interior Lights
   <td>✔
   <td>✔
  <tr>
   <td>Electric locomotive operating sounds
   <td>✔
   <td>✔
  <tr>
   <td>Engineer's cab lighting
   <td>
   <td>✔
  <tr>
   <td>Station Announcements - Swiss
   <td>
   <td>✔
</table>
```

Since the cells take up much less room this way, this can be made even terser by having each row on one line:

```html
<table>
 <caption>37547 TEE Electric Powered Rail Car Train Functions (Abbreviated)
 <colgroup><col><col><col>
 <thead>
  <tr> <th>Function                              <th>Control Unit     <th>Central Station
 <tbody>
  <tr> <td>Headlights                            <td>✔                <td>✔
  <tr> <td>Interior Lights                       <td>✔                <td>✔
  <tr> <td>Electric locomotive operating sounds  <td>✔                <td>✔
  <tr> <td>Engineer's cab lighting               <td>                 <td>✔
  <tr> <td>Station Announcements - Swiss         <td>                 <td>✔
</table>
```

The only differences between these tables, at the DOM level, is with the precise position of the (in any case semantically-neutral) whitespace.

**However**, a [start tag](#syntax-start-tag) must never be omitted if it has any attributes.

Returning to the earlier example with all the whitespace removed and then all the optional tags removed:

```html
<!DOCTYPE HTML><title>Hello</title><p>Welcome to this example.
```

If the `body` element in this example had to have a `class` attribute and the `html` element had to have a `lang` attribute, the markup would have to become:

```html
<!DOCTYPE HTML><html lang="en"><title>Hello</title><body class="demo"><p>Welcome to this example.
```

This section assumes that the document is conforming, in particular, that there are no [content model](https://html.spec.whatwg.org/multipage/dom.html#content-models) violations. Omitting tags in the fashion described in this section in a document that does not conform to the [content models](https://html.spec.whatwg.org/multipage/dom.html#content-models) described in this specification is likely to result in unexpected DOM differences (this is, in part, what the content models are designed to avoid).

##### 13.1.2.5 Restrictions on content models

For historical reasons, certain elements have extra restrictions beyond even the restrictions given by their content model.

A `table` element must not contain `tr` elements, even though these elements are technically allowed inside `table` elements according to the content models described in this specification. (If a `tr` element is put inside a `table` in the markup, it will in fact imply a `tbody` start tag before it.)

A single [newline](#syntax-newlines) may be placed immediately after the [start tag](#syntax-start-tag) of `pre` and `textarea` elements. This does not affect the processing of the element. The otherwise optional [newline](#syntax-newlines) *must* be included if the element's contents themselves start with a [newline](#syntax-newlines) (because otherwise the leading newline in the contents would be treated like the optional newline, and ignored).

The following two `pre` blocks are equivalent:

```html
<pre>Hello</pre>
<pre>
Hello</pre>
```

##### 13.1.2.6 Restrictions on the contents of raw text and escapable raw text elements

The text in [raw text](#raw-text-elements) and [escapable raw text elements](#escapable-raw-text-elements) must not contain any occurrences of the string "`</`" (U+003C LESS-THAN SIGN, U+002F SOLIDUS) followed by characters that case-insensitively match the tag name of the element followed by one of U+0009 CHARACTER TABULATION (tab), U+000A LINE FEED (LF), U+000C FORM FEED (FF), U+000D CARRIAGE RETURN (CR), U+0020 SPACE, U+003E GREATER-THAN SIGN (>), or U+002F SOLIDUS (/).

#### 13.1.3 Text

Text is allowed inside elements, attribute values, and comments. Extra constraints are placed on what is and what is not allowed in text based on where the text is to be put, as described in the other sections.

##### 13.1.3.1 Newlines

Newlines in HTML may be represented either as U+000D CARRIAGE RETURN (CR) characters, U+000A LINE FEED (LF) characters, or pairs of U+000D CARRIAGE RETURN (CR), U+000A LINE FEED (LF) characters in that order.

Where [character references](#syntax-charref) are allowed, a character reference of a U+000A LINE FEED (LF) character (but not a U+000D CARRIAGE RETURN (CR) character) also represents a [newline](#syntax-newlines).

#### 13.1.4 Character references

In certain cases described in other sections, [text](#syntax-text) may be mixed with character references. These can be used to escape characters that couldn't otherwise legally be included in [text](#syntax-text).

Character references must start with a U+0026 AMPERSAND character (&). Following this, there are three possible kinds of character references:

- Named character references

  The ampersand must be followed by one of the names given in the [named character references](https://html.spec.whatwg.org/multipage/named-characters.html#named-character-references) section, using the same case. The name must be one that is terminated by a U+003B SEMICOLON character (;).

- Decimal numeric character reference

  The ampersand must be followed by a U+0023 NUMBER SIGN character (#), followed by one or more `/\d/`, representing a base-ten integer that corresponds to a code point that is allowed according to the definition below. The digits must then be followed by a U+003B SEMICOLON character (;).

- Hexadecimal numeric character reference

  The ampersand must be followed by a U+0023 NUMBER SIGN character (#), which must be followed by either a U+0078 LATIN SMALL LETTER X character (x) or a U+0058 LATIN CAPITAL LETTER X character (X), which must then be followed by one or more `/[\dA-F]/i`, representing a hexadecimal integer that corresponds to a code point that is allowed according to the definition below. The digits must then be followed by a U+003B SEMICOLON character (;).

The numeric character reference forms described above are allowed to reference any code point excluding U+000D CR, [noncharacters](https://infra.spec.whatwg.org/#noncharacter), and [controls](https://infra.spec.whatwg.org/#control) other than `/\s/`.

An ambiguous ampersand is a U+0026 AMPERSAND character (&) that is followed by one or more [ASCII alphanumerics](https://infra.spec.whatwg.org/#ascii-alphanumeric), followed by a U+003B SEMICOLON character (;), where these characters do not match any of the names given in the [named character references](https://html.spec.whatwg.org/multipage/named-characters.html#named-character-references) section.

#### 13.1.5 CDATA sections

CDATA sections must consist of the following components, in this order:

1. The string "`<![CDATA[`".
2. Optionally, [text](#syntax-text), with the additional restriction that the text must not contain the string "`]]>`".
3. The string "`]]>`".

CDATA sections can only be used in foreign content (MathML or SVG). In this example, a CDATA section is used to escape the contents of a [MathML `ms`](https://www.w3.org/Math/draft-spec/chapter3.html#presm.ms) element:

```html
<p>You can add a string to a number, but this stringifies the number:</p>
<math>
 <ms><![CDATA[x<y]]></ms>
 <mo>+</mo>
 <mn>3</mn>
 <mo>=</mo>
 <ms><![CDATA[x<y3]]></ms>
</math>
```

#### 13.1.6 Comments

Comments must have the following format:

1. The string "`<!--`".
2. Optionally, [text](#syntax-text), with the additional restriction that the text must not start with the string "`>`", nor start with the string "`->`", nor contain the strings "`<!--`", "`-->`", or "`--!>`", nor end with the string "`<!-`".
3. The string "`-->`".

The [text](#syntax-text) is allowed to end with the string "`<!`", as in `<!--My favorite operators are > and <!-->`.
