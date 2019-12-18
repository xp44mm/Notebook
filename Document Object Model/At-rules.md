# At-rules

**At-rules** are [CSS statements](https://developer.mozilla.org/en-US/docs/Web/CSS/Syntax#CSS_statements) that instructs CSS how to behave. They begin with an at sign, '`@`' (`U+0040 COMMERCIAL AT`), followed by an identifier and includes everything up to the next semicolon, '`;`' (`U+003B SEMICOLON`), or the next [CSS block](https://developer.mozilla.org/en-US/docs/Web/CSS/Syntax#CSS_declarations_blocks), whichever comes first.

```css
/* General structure */
@IDENTIFIER (RULE);

/* Example: tells browser to use UTF-8 character set */
@charset "utf-8";
```

There are several at-rules, designated by their identifiers, each with a different syntax:

- [`@charset`](https://developer.mozilla.org/en-US/docs/Web/CSS/@charset) — Defines the character set used by the style sheet.
- [`@import`](https://developer.mozilla.org/en-US/docs/Web/CSS/@import) — Tells the CSS engine to include an external style sheet.
- [`@namespace`](https://developer.mozilla.org/en-US/docs/Web/CSS/@namespace) — Tells the CSS engine that all its content must be considered prefixed with an XML namespace.
- *Nested at-rules* — A subset of nested statements, which can be used as a statement of a style sheet as well as inside of conditional group rules:
  - [`@media`](https://developer.mozilla.org/en-US/docs/Web/CSS/@media) — A conditional group rule that will apply its content if the device meets the criteria of the condition defined using a *media query*.
  - [`@supports`](https://developer.mozilla.org/en-US/docs/Web/CSS/@supports) — A conditional group rule that will apply its content if the browser meets the criteria of the given condition.
  - [`@page`](https://developer.mozilla.org/en-US/docs/Web/CSS/@page) — Describes the aspect of layout changes that will be applied when printing the document.
  - [`@font-face`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face) — Describes the aspect of an external font to be downloaded.
  - [`@keyframes`](https://developer.mozilla.org/en-US/docs/Web/CSS/@keyframes) — Describes the aspect of intermediate steps in a CSS animation sequence.
  - [`@counter-style`](https://developer.mozilla.org/en-US/docs/Web/CSS/@counter-style) — Defines specific counter styles that are not part of the predefined set of styles. *(at the Candidate Recommendation stage, but only implemented in Gecko as of writing)*
  - [`@font-feature-values`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-feature-values) (plus `@swash`, `@ornaments`, `@annotation`, `@stylistic`, `@styleset` and `@character-variant`)
    — Define common names in [`font-variant-alternates`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-variant-alternates) for feature activated differently in OpenType. *(at the Candidate Recommendation stage, but only implemented in Gecko as of writing)*

## Conditional group rules

Much like the values of properties, each at-rule has a different syntax. Nevertheless, several of them can be grouped into a special category named **conditional group rules**. These statements share a common syntax and each of them can include *nested statements*—either *rulesets* or *nested at-rules*. Furthermore, they all convey a common semantic meaning—they all link some type of condition, which at any time evaluates to either **true** or **false**. If the condition evaluates to **true**, then all of the statements within the group will be applied.

Conditional group rules are defined in [CSS Conditionals Level 3](http://dev.w3.org/csswg/css3-conditional/) and are:

- [`@media`](https://developer.mozilla.org/en-US/docs/Web/CSS/@media),
- [`@supports`](https://developer.mozilla.org/en-US/docs/Web/CSS/@supports),
- [`@document`](https://developer.mozilla.org/en-US/docs/Web/CSS/@document). *(deferred to Level 4 of CSS Spec)*

Since each conditional group may also contain nested statements, there may be an unspecified amount of nesting.

# @import

The **`@import`** [CSS](https://developer.mozilla.org/en-US/docs/Web/CSS) [at-rule](https://developer.mozilla.org/en-US/docs/Web/CSS/At-rule) is used to import style rules from other style sheets. These rules must precede all other types of rules, except [`@charset`](https://developer.mozilla.org/en-US/docs/Web/CSS/@charset) rules; as it is not a [nested statement](https://developer.mozilla.org/en-US/docs/Web/CSS/Syntax#nested_statements), `@import` cannot be used inside [conditional group at-rules](https://developer.mozilla.org/en-US/docs/Web/CSS/At-rule#Conditional_Group_Rules).

```css
@import url("fineprint.css") print;
@import url("bluish.css") speech;
@import 'custom.css';
@import url("chrome://communicator/skin/");
@import "common.css" screen;
@import url('landscape.css') screen and (orientation:landscape);
```

So that [user agent](https://developer.mozilla.org/en-US/docs/Glossary/user_agent)s can avoid retrieving resources for unsupported media types, authors may specify media-dependent `@import` rules. These conditional imports specify comma-separated [media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries) after the URL. In the absence of any media query, the import is unconditional. Specifying `all` for the medium has the same effect.

## Syntax

```css
@import url;
@import url list-of-media-queries;
```

where:

- *url*

  Is a `<string>` or a `<url>` representing the location of the resource to import. The URL may be absolute or relative. Note that the URL for a Mozilla package need not actually specify a file; it can just specify the package name and part, and the appropriate file is chosen automatically (e.g. **chrome://communicator/skin/**). [See here](https://developer.mozilla.org/en-US/docs/Mozilla/Tech/XUL/Tutorial/The_Chrome_URL) for more information.

- *list-of-media-queries*

  Is a comma-separated list of [media queries](https://developer.mozilla.org/en-US/docs/Web/CSS/Media_Queries/Using_media_queries) conditioning the application of the CSS rules defined in the linked URL. If the browser does not support any these queries, it does not load the linked resource.

# \<string\>

The **`<string>`** [CSS](https://developer.mozilla.org/en-US/docs/Web/CSS) [data type](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Types) represents a sequence of characters. Strings are used in numerous CSS properties, such as [`content`](https://developer.mozilla.org/en-US/docs/Web/CSS/content), [`font-family`](https://developer.mozilla.org/en-US/docs/Web/CSS/font-family), and [`quotes`](https://developer.mozilla.org/en-US/docs/Web/CSS/quotes).

## Syntax

The `<string>` data type is composed of any number of [Unicode](http://en.wikipedia.org/wiki/Unicode) characters surrounded by either double (`"`) or single (`'`) quotes.

Most characters can be represented literally. All characters can also be represented with their respective [Unicode code points](http://en.wikipedia.org/wiki/Unicode#Code_point_planes_and_blocks) in hexadecimal, in which case they are preceded by a backslash (`\`). For example, `\22` represents a double quote, `\27` a single quote (`'`), and `\A9` the copyright symbol (`©`).

Importantly, certain characters which would otherwise be invalid can be escaped with a backslash. These include double quotes when used inside a double-quoted string, single quotes when used inside a single-quoted string, and the backslash itself. For example, `\\` will create a single backslash.

To output new lines, you must escape them with a line feed character such as `\A` or `\00000A`. In your code, however, strings can span multiple lines, in which case each new line must be escaped with a `\` as the last character of the line.

**Note:** [HTML entities](https://developer.mozilla.org/en-US/docs/Glossary/Entity) (such as `&nbsp;` or `&#8212;`) cannot be used in a CSS `<string>`.

## Examples

Simple strings

```css
/* Simple strings */
"This string is demarcated by double quotes."
'This string is demarcated by single quotes.'
```

Character escaping

```css
/* Character escaping */
"This is a string with \" an escaped double quote."
"This string also has \22 an escaped double quote."
'This is a string with \' an escaped single quote.'
'This string also has \27 an escaped single quote.'
"This is a string with \\ an escaped backslash."
```

New line in a string

```css
/* New line in a string */
"This string has a \Aline break in it."
```

String spanning two lines of code (these two strings will have identical output)

```css
/* String spanning two lines of code (these two strings will have identical output) */
"A really long \
awesome string"
"A really long awesome string"
```

# \<url>

The **`<url>`** [CSS](https://developer.mozilla.org/en-US/docs/Web/CSS) [data type](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Types) denotes a pointer to a resource, such as an image or a font. URLs can be used in numerous CSS properties, such as [`background-image`](https://developer.mozilla.org/en-US/docs/Web/CSS/background-image), [`cursor`](https://developer.mozilla.org/en-US/docs/Web/CSS/cursor), and [`list-style`](https://developer.mozilla.org/en-US/docs/Web/CSS/list-style).

**URI or URL?** There is a difference between a [URI](https://developer.mozilla.org/en-US/docs/Glossary/URI) and a [URL](https://developer.mozilla.org/en-US/docs/Glossary/URL). A URI simply identifies a resource. A URL is a type of URI, and describes the *location* of a resource. A URI can be either a URL or a name ([URN](https://developer.mozilla.org/en-US/docs/Glossary/URN)) of a resource.

In CSS Level 1, the `url()` functional notation described only true URLs. In CSS Level 2, the definition of `url()` was extended to describe any URI, whether a URL or a URN. Confusingly, this meant that `url()` could be used to create a `<uri>` CSS data type. This change was not only awkward but, debatably, unnecessary, since URNs are almost never used in actual CSS. To alleviate the confusion, CSS Level 3 returned to the narrower, initial definition. Now, `url()` denotes only true `<url>`s.

## Syntax

The `<url>` data type is specified using the `url()` functional notation. It may be written without quotes, or surrounded by single or double quotes. Relative URLs are allowed, and are relative to the URL of the stylesheet (not to the URL of the web page).

```css
<a_css_property>: url("http://mysite.example.com/mycursor.png")
<a_css_property>: url('http://mysite.example.com/mycursor.png')
<a_css_property>: url(http://mysite.example.com/mycursor.png)
```

**Note:** Control characters above 0x7e are not allowed in unquoted URLs, starting with Firefox 15. See [bug 752230](https://bugzilla.mozilla.org/show_bug.cgi?id=752230) for more details.

## Examples

```css
.topbanner {
  background: url("topbanner.png") #00D no-repeat fixed;
}
ul {
  list-style: square url(http://www.example.com/redball.png);
}
```