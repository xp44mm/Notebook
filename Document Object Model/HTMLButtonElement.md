# HTMLButtonElement

The **`HTMLButtonElement`** interface provides properties and methods (beyond the regular `HTMLElement` interface it also has available to it by inheritance) for manipulating `<button>` elements.

```js
class HTMLButtonElement extends HTMLElement {}
```



## Properties

Inherits properties from its parent, `HTMLElement`.

- [`HTMLButtonElement.accessKey`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/accessKey)

  Is a `DOMString` indicating the single-character keyboard key to give access to the button.

- [`HTMLButtonElement.autofocus`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/autofocus)

  Is a `Boolean` indicating whether or not the control should have input focus when the page loads, unless the user overrides it, for example by typing in a different control. Only one form-associated element in a document can have this attribute specified.

- [`HTMLButtonElement.disabled`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/disabled)

  Is a `Boolean` indicating whether or not the control is disabled, meaning that it does not accept any clicks.

- [`HTMLButtonElement.form`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/form) Read only

  Is a `HTMLFormElement` reflecting the form that this button is associated with. If the button is a descendant of a form element, then this attribute is the ID of that form element. If the button is not a descendant of a form element, then the attribute can be the ID of any form element in the same document it is related to, or the `null` value if none matches.

- [`HTMLButtonElement.formAction`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/formAction)

  Is a `DOMString` reflecting the URI of a resource that processes information submitted by the button. If specified, this attribute overrides the `action` attribute of the `<form>` element that owns this element.

- [`HTMLButtonElement.formEnctype`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/formEnctype)

  Is a `DOMString` reflecting the type of content that is used to submit the form to the server. If specified, this attribute overrides the `enctype` attribute of the `<form>` element that owns this element.

- [`HTMLButtonElement.formMethod`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/formMethod)

  Is a `DOMString` reflecting the HTTP method that the browser uses to submit the form. If specified, this attribute overrides the `method` attribute of the `<form>` element that owns this element.

- [`HTMLButtonElement.formNoValidate`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/formNoValidate)

  Is a `Boolean` indicating that the form is not to be validated when it is submitted. If specified, this attribute overrides the `novalidate` attribute of the `<form>` element that owns this element.

- [`HTMLButtonElement.formTarget`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/formTarget)

  Is a `DOMString` reflecting a name or keyword indicating where to display the response that is received after submitting the form. If specified, this attribute overrides the `target` attribute of the `<form>` element that owns this element.

- [`HTMLButtonElement.labels`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/labels) Read only

  Is a `NodeList` that represents a list of `<form>` elements that are labels for this button.

- [`HTMLButtonElement.menu`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/menu) 

  试验

- [`HTMLButtonElement.name`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/name)

  Is a `DOMString` representing the name of the object when submitted with a form. If specified, it must not be the empty string.

- [`HTMLButtonElement.tabIndex`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/tabIndex)

  Is a `long` that represents this element's position in the tabbing order.

- [`HTMLButtonElement.type`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/type)

  Is a `DOMString` indicating the behavior of the button. This is an enumerated attribute with the following possible values:

  - `"submit"`: The button submits the form. This is the default value if the attribute is not specified, or if it is dynamically changed to an empty or invalid value.
  - `"reset"`: The button resets the form.
  - `"button"`: The button does nothing.
  - `"menu"`: 试验

- [`HTMLButtonElement.willValidate`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/willValidate) Read only

  Is a `Boolean` indicating whether the button is a candidate for constraint validation. It is `false` if any conditions bar it from constraint validation, including: its `type` property is `reset` or `button`; it has a [datalist](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/datalist) ancestor; or the `disabled` property is set to `true`.

- [`HTMLButtonElement.validationMessage`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/validationMessage) Read only

  Is a `DOMString` representing the localized message that describes the validation constraints that the control does not satisfy (if any). This attribute is the empty string if the control is not a candidate for constraint validation (`willValidate` is `false`), or it satisfies its constraints.

- [`HTMLButtonElement.validity`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/validity) Read only

  Is a `ValidityState` representing the validity states that this button is in.

- [`HTMLButtonElement.value`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLButtonElement/value)

  Is a `DOMString` representing the current form control value of the button.

## Methods

Inherits methods from its parent, `HTMLElement`

| Name                                    | Return Type | Description                                 |
| :-------------------------------------- | :---------- | :------------------------------------------ |
| `checkValidity()`                       | `Boolean`   | Not supported for reset or button elements. |
| `reportValidity()`                      | `Boolean`   | Not supported for reset or button elements. |
| `setCustomValidity(in DOMString error)` | `void`      | Not supported for reset or button elements. |

