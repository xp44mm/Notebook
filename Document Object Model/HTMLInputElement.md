# HTMLInputElement

The **`HTMLInputElement`** interface provides special properties and methods for manipulating the options, layout, and presentation of `<input>` elements.

```js
class HTMLInputElement extends HTMLElement {}
```



## Properties

Properties related to the parent form

| `form `Read only | `HTMLFormElement` object: **Returns** a reference to the parent `<form>` element. |
| ---------------- | ------------------------------------------------------------ |
| `formAction`     | *`string`:* **Returns / Sets** the element's `formaction` attribute, containing the URI of a program that processes information submitted by the element. This overrides the `action` attribute of the parent form. |
| `formEncType`    | *`string`:* **Returns / Sets** the element's `formenctype` attribute, containing the type of content that is used to submit the form to the server. This overrides the `enctype` attribute of the parent form. |
| `formMethod`     | *`string`:* **Returns / Sets** the element's `formmethod` attribute, containing the HTTP method that the browser uses to submit the form. This overrides the `method` attribute of the parent form. |
| `formNoValidate` | *`Boolean`:* **Returns / Sets** the element's `formnovalidate` attribute, indicating that the form is not to be validated when it is submitted. This overrides the `novalidate` attribute of the parent form. |
| `formTarget`     | *`string`:* **Returns / Sets** the element's `formtarget` attribute, containing a name or keyword indicating where to display the response that is received after submitting the form. This overrides the `target` attribute of the parent form. |

Properties that apply to any type of input element that is not hidden

| `name`                        | *`string`:* **Returns / Sets** the element's `name` attribute, containing a name that identifies the element when submitting the form. |
| ----------------------------- | ------------------------------------------------------------ |
| `type`                        | `*string*`: **Returns / Sets** the element's `type` attribute, indicating the type of control to display. See `type` attribute of `<input>` for possible values. |
| `disabled`                    | *`Boolean`:* **Returns / Sets** the element's `disabled` attribute, indicating that the control is not available for interaction. The input values will not be submitted with the form. See also `readonly` |
| `autofocus`                   | *`Boolean`:* **Returns / Sets** the element's `autofocus` attribute, which specifies that a form control should have input focus when the page loads, unless the user overrides it, for example by typing in a different control. Only one form element in a document can have the `autofocus` attribute. It cannot be applied if the `type` attribute is set to `hidden` (that is, you cannot automatically set focus to a hidden control). |
| `required`                    | *`Boolean`:* **Returns / Sets** the element's `required` attribute, indicating that the user must fill in a value before submitting a form. |
| `value`                       | `string`*:* **Returns / Sets** the current value of the control. **Note:** If the user enters a value different from the value expected, this may return an empty string. |
| `validity` Read only          | `ValidityState` object: **Returns** the element's current validity state. |
| `validationMessage` Read only | `*string*`*:* **Returns** a localized message that describes the validation constraints that the control does not satisfy (if any). This is the empty string if the control is not a candidate for constraint validation (`willvalidate` is false), or it satisfies its constraints. This value can be set by the `setCustomValidity` method. |
| `willValidate `Read only      | *`Boolean`:* **Returns** whether the element is a candidate for constraint validation. It is false if any conditions bar it from constraint validation, including: its `type` is `hidden`, `reset`, or `button`; it has a datalist ancestor; its `disabled` property is `true`. |

Properties that apply only to elements of type checkbox or radio

| `checked`        | *`Boolean`:* **Returns / Sets** the current state of the element when `type` is `checkbox` or `radio`. |
| ---------------- | ------------------------------------------------------------ |
| `defaultChecked` | *`Boolean`:* **Returns / Sets** the default state of a radio button or checkbox as originally specified in HTML that created this object. |
| `indeterminate`  | *`Boolean`:* **Returns** whether the checkbox or radio button is in indeterminate state. For checkboxes, the effect is that the appearance of the checkbox is obscured/greyed in some way as to indicate its state is indeterminate (not checked but not unchecked). Does not affect the value of the `checked` attribute, and clicking the checkbox will set the value to false. |

Properties that apply only to elements of type image

| `alt`    | *`string`:* **Returns / Sets** the element's `alt` attribute, containing alternative text to use when `type` is `image.` |
| -------- | ------------------------------------------------------------ |
| `height` | *`string`:* **Returns / Sets** the element's `height` attribute, which defines the height of the image displayed for the button, if the value of `type` is `image`. |
| `src`    | `string`*:* **Returns / Sets** the element's `src` attribute, which specifies a URI for the location of an image to display on the graphical submit button, if the value of `type` is `image`; otherwise it is ignored. |
| `width`  | `string`*:* **Returns / Sets** the document's `width` attribute, which defines the width of the image displayed for the button, if the value of `type` is `image`. |

Properties that apply only to elements of type file

| `accept`                                                     | *`string`:* **Returns / Sets** the element's `accept` attribute, containing comma-separated list of file types accepted by the server when `type` is `file`. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `allowdirs`                                                  | 非标                                                         |
| `files`                                                      | **Returns/accepts** a `FileList` object, which contains a list of `File` objects representing the files selected for upload. |
| [`webkitdirectory`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/webkitdirectory) | 非标                                                         |
| [`webkitEntries`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/webkitEntries) | 非标                                                         |

Properties that apply only to text/number-containing or elements

| `autocomplete` | `string`: **Returns / Sets** the element's `autocomplete` attribute, indicating whether the value of the control can be automatically completed by the browser. Ignored if the value of the `type` attribute is `hidden`, `checkbox`, `radio`, `file`, or a button type (`button`, `submit`, `reset`, `image`). Possible values are: `on`: the browser can autocomplete the value using previously stored value `off`: the user must explicity enter a value |
| -------------- | ------------------------------------------------------------ |
| `max`          | *`string`:* **Returns / Sets** the element's `max` attribute, containing the maximum (numeric or date-time) value for this item, which must not be less than its minimum (`min` attribute) value. |
| `maxLength`    | *`long`:* **Returns / Sets** the element's `maxlength` attribute, containing the **maximum number of characters** (in Unicode code points) that the value can have. (If you set this to a negative number, an exception will be thrown.) |
| `min`          | *`string`:* **Returns / Sets** the element's `min` attribute, containing the minimum (numeric or date-time) value for this item, which must not be greater than its maximum (`max` attribute) value. |
| `minLength`    | *`long`:* **Returns / Sets** the element's `minlength` attribute, containing the **minimum number of characters** (in Unicode code points) that the value can have. (If you set this to a negative number, an exception will be thrown.) |
| `pattern`      | *`string`:* **Returns / Sets** the element's `pattern` attribute, containing a **regular expression** that the control's value is checked against. Use the `title` attribute to describe the pattern to help the user. This attribute applies when the value of the `type` attribute is `text`, `search`, `tel`, `url` or `email`; otherwise it is ignored. |
| `placeholder`  | *`string`:* **Returns / Sets** the element's `placeholder` attribute, containing a hint to the user of what can be entered in the control. The placeholder text must not contain carriage returns or line-feeds. This attribute applies when the value of the `type` attribute is `text`, `search`, `tel`, `url` or `email`; otherwise it is ignored. |
| `readOnly`     | *`boolean`:* **Returns / Sets** the element's `readonly` attribute, indicating that the user cannot modify the value of the control. [HTML5](https://developer.mozilla.org/en-US/docs/HTML/HTML5)This is ignored if the value of the `type` attribute is `hidden`, `range`, `color`, `checkbox`, `radio`, `file`, or a button type. |
| `size`         | *`unsigned long`:* **Returns / Sets** the element's `size` attribute, which contains the **visual size of the control**. This value is in pixels unless the value of `type` is `text` or `password`, in which case, it is an integer number of characters. Applies only when `type` is set to `text`, `search`, `tel`, `url`, `email`, or `password`; otherwise it is ignored. |

Properties that apply only to elements with type text/password/search/tel/url/week/month

| `selectionStart`     | *`unsigned long`:* **Returns / Sets** the beginning index of the selected text. When nothing is selected, this returns the position of the text input cursor (caret) inside of the `<input>` element. |
| -------------------- | ------------------------------------------------------------ |
| `selectionEnd`       | *`unsigned long`:* **Returns / Sets** the end index of the selected text. When there's no selection, this returns the offset of the character immediately following the current text input cursor position. |
| `selectionDirection` | *`string`:* **Returns / Sets** the direction in which selection occurred. Possible values are: `forward` if selection was performed in the start-to-end direction of the current locale `backward` for the opposite direction `none` if the direction is unknown |


Properties not yet categorized

| `defaultValue`                                               | *`string`:* **Returns / Sets** the default value as originally specified in the HTML that created this object. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `dirName`                                                    | *`string`:* **Returns / Sets** the directionality of the element. |
| `accessKey`                                                  | *`string`:* **Returns** a string containing a single character that switches input focus to the control when pressed. |
| `list` Read only                                             | *`HTMLElement`` object`:* **Returns** the element pointed by the `list` attribute. The property may be `null` if no HTML element found in the same tree. |
| `multiple`                                                   | *`Boolean`:* **Returns / Sets** the element's `multiple` attribute, indicating whether more than one value is possible (e.g., multiple files). |
| `files`                                                      | *`FileList`` array`:* **Returns** the list of selected files. |
| [`HTMLInputElement.labels`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/labels) Read only | *`NodeList`` array`:* **Returns** a list of `<label>` elements that are labels for this element. |
| `step`                                                       | `*string*`*:* **Returns / Sets** the element's `step` attribute, which works with `min` and `max` to limit the increments at which a numeric or date-time value can be set. It can be the string `any` or a positive floating point number. If this is not set to `any`, the control accepts only values at multiples of the step value greater than the minimum. |
| `valueAsDate`                                                | `Date` object: **Returns / Sets** the value of the element, interpreted as a date, or `null` if conversion is not possible. |
| `valueAsNumber`                                              | *`double`:* **Returns** the value of the element, interpreted as one of the following, in order:A time valueA number`NaN` if conversion is impossible |
| `autocapitalize`                                             | 试验                                                         |
| `inputmode`                                                  | Provides a hint to browsers as to the type of virtual keyboard configuration to use when editing this element or its contents. |

- [`HTMLImageElement.align`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/align) 

  废弃

- [`HTMLObjectElement.useMap`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLObjectElement/useMap) 

  废弃

## Methods

| [`blur()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/blur) | Removes focus from the input element; keystrokes will subsequently go nowhere. |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`click()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/click) | Simulates a click on the input element.                      |
| [`focus()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus) | Focuses on the input element; keystrokes will subsequently go to this element. |
| [`select()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/select) | Selects all the text in the input element, and focuses it so the user can subsequently replace all of its content. |
| [`setSelectionRange()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/setSelectionRange) | Selects a range of text in the input element (but does not focus it). |
| [`setRangeText()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/setRangeText) | Replaces a range of text in the input element with new text. |
| [`stepUp()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/stepUp) | Increments the `value` by (`step` * n), where n defaults to 1 if not specified. |
| [`stepDown()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/stepDown) | Decrements the `value` by (`step` * n), where n defaults to 1 if not specified. |
| [`setCustomValidity()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLObjectElement/setCustomValidity) | Sets a custom validity message for the element. If this message is not the empty string, then the element is suffering from a custom validity error, and does not validate. Inherited from [`HTMLObjectElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLObjectElement) |
| [`checkValidity()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLObjectElement/checkValidity) | Returns a [`Boolean`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean) that is `false` if the element is a candidate for constraint validation, and it does not satisfy its constraints. In this case, it also fires an `invalid` event at the element. It returns `true` if the element is not a candidate for constraint validation, or if it satisfies its constraints. Inherited from [`HTMLObjectElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLObjectElement) |
| [`reportValidity()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement/reportValidity) | Runs the `checkValidity()` method, and if it returns false (for an invalid input or no pattern attribute provided), then it reports to the user that the input is invalid in the same manner as if you submitted a form. Inherited from [`HTMLFormElement`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLFormElement) |

### Non-standard methods



- [`HTMLInputElement.mozSetFileArray()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/mozSetFileArray) 

- [`HTMLInputElement.mozGetFileNameArray()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/mozGetFileNameArray) 

- [`HTMLInputElement.mozSetFileNameArray()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement/mozSetFileNameArray) 

## Events

Listen to these events using `addEventListener()` or by assigning an event listener to the `on*eventname*` property of this interface:

- [`input`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/input_event)

  Fires when the `value` of an `<input>`, `<select>`, or `<textarea>` element has been changed. Note that this is actually fired on the `HTMLElement` interface and also applies to `contenteditable` elements, but we've listed it here because it is most commonly used with form input elements. Also available via the `oninput` event handler property.

- `invalid`

  Fired when an element does not satisfy its constraints during constraint validation. Also available via the `oninvalid` event handler property.

- `search`

  Fired when a search is initiated on an `<input>` of `type="search"`. Also available via the `onsearch` event handler property.