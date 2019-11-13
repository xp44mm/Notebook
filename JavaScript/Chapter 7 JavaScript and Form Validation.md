# Chapter 7 JavaScript and Form Validation

It is inevitable that, when encountering a form, one considers the fate of the data for that form. One of the first practical applications of JavaScript was providing a way to validate data on the client side, instead of having to endure a round trip to the server. Form validation was a bit ad hoc at the time, with no practical API and no real integration with the browser. Instead, programmers bound together events and basic text manipulation to provide a handy user interface enhancement.

Fast-forward to the present day, and form validation is in much better shape. With modern browsers, we have an integrated validation API, which works with both HTML and CSS to provide an extensive set of form validation features. We also have regular expressions, which—for all their complications—are much better for data validation than, say, iterating over a string character by character.

Our concern in this chapter is JavaScript and forms. While we will focus on form validation, we will also look at general improvements in the ways JavaScript interacts with forms, as well as some newly available form-related APIs.

## HTML and CSS Form Validation

As mentioned, form validation has come a long way since the early days of JavaScript. To really dive into the state of form validation, we will need to look at not only JavaScript, but also HTML5 and CSS. Let's start with the HTML side of things. In the last few years, HTML has evolved and added many new features, thanks to the hard work of the Web Hypertext Application Technology Working Group (WHATWG). This organization has pushed for the evolution and updating of HTML into what has come to be known as HTML5. Although the scope of the HTML5 specification means that we can't discuss the details here, you can find more information in HTML5 Programmer's Reference by Jonathan Reid (Apress, 2015).

Of particular note in HTML5 are the advances to the set of form controls. These changes have broadly fallen into two categories: addition of new controls or styles of controls (URL fields, date pickers, and the like), and form validation. Initially, our focus will be on the latter. Simple form validation has moved into HTML, without the need of any JavaScript whatsoever. This validation is available through the addition of certain attributes to form controls. A simple example is the required attribute, which pairs with input elements and forces the field to have value before the form can be submitted. Listing 7-1 is a basic example.

Listing 7-1. A Simple Form

```html
<!DOCTYPE html>
<html>
<head>
  <title>A basic form</title>
</head>
<body>
<h2>A basic form</h2>
<p>Please provide your first and last names.</p>
<form>
  <fieldset>
    <label for="firstName">First Name:</label><br/>
    <input id="firstName" name="firstName" type="text" required/><br/>
    <label for="lastName">Last Name:</label><br/>
    <input type="text" name="lastName" id="lastName"/><br/>
  </fieldset>
  <input type="submit" value="Submit the form"/> <input type="reset" value="Reset the
form"/>
</form>
</body>
</html>
```

Notice that in the form, we have an input field with an ID of firstName, which has added the aforementioned required attribute. Were we to attempt to submit the form without filling out this field, we would see a result something like Figure 7-1.

The display looks roughly the same on Chrome and IE 11 (Chrome does not surround the field with a red border, but IE has a blockier, more assertive red border). If you were to make both the firstName and lastName fields required, the border would appear on each of the fields, but the popup tooltip would only be associated with the first field that had a problem. What about customizing the popup? We will deal with that soon, but it will require JavaScript.

There are several other types of validation that can be activated via HTML attributes. They are

• pattern: This attribute takes a regular expression as an argument. The regular expression does not need to be surrounded by slashes. The regular expression language engine is the same as JavaScript's (with the same issues as well). This is attached to the input element. Note that input types of email and url imply pattern values appropriate to valid email addresses and URLs, respectively. Pattern validation does not work with Safari 8, iOS Safari 8.1, or Opera Mini.

• step: Forces the value to match a multiple of the specified step value. Constrained to input types of number, range, or one of the date-times. Step validation works in Chrome 6.0, Firefox 16.0, IE 10, Opera 10.62, and Safari 5.0.

• min / max: Minimum or maximum values, appropriate to not only numbers but also date-times. This method works in Chrome 41, Opera 27, and Chrome for Android 41.

• maxlength: Maximum length, in characters, of the data in the field. Only valid for input types of text, email, search, password, tel, or url. This method usually does not validate so much as prevent a user from entering too much data in the field it is attached to. It works on all modern browsers.

At the form level, you can turn off validation as a whole one of two ways. Either you can add the formnovalidate attribute to the Submit button for the form, or you can add the novalidate attribute to the form element itself.

### CSS

Not content to have HTML5 do all the work, the CSS specification has been updated to address form validation as well. Form elements that are in an invalid state can be accessed via the :invalid pseudoclass. Unfortunately, the implementation of this pseudoclass leaves a bit to be desired. First, form elements are checked for their validity at page load. Thus, if you had styling like the following:

```css
:invalid { background-color: yellow }
```

when the page loaded, many of your fields would have a yellow background. Second, Chrome and IE apply :invalid only to form elements. Firefox will apply it to the entire form, if any element in the form is invalid. Consider Listing 7-2.

Listing 7-2. Using the :invalid Pseudoclass

```html
<!DOCTYPE html>
<html>
<head>
  <title>A basic form</title>
  <style>
    :invalid {
      background-color: yellow
    }
  </style>
</head>
<body>
<h2>A basic form</h2>

<p>Please provide your first and last names.</p>
<form>
  <fieldset>
  <label for="firstName">First Name:</label><br/>
  <input id="firstName" name="firstName" type="text" required/><br/>
  <label for="lastName">Last Name:</label><br/>
  <input type="text" name="lastName" id="lastName"/><br/>
  </fieldset>
  <input type="submit" value="Submit the form"/> <input type="reset" value="Reset the
form"/>
</form>
</body>
</html>
```

In this listing, Firefox displays the entire form with a yellow background, since one element of the form is in an invalid state. Fix this by changing the styling for :invalid to input:invalid, which gives you consistent behavior across browsers.

CSS also provides a few other pseudoclasses:

• :valid covers elements that are in a valid state.

• :required gets elements that have their required attribute set to true.

• :optional gets elements that do not have the required attribute set.

• :in-range is used for elements that are within their min/max boundaries; it is not supported by IE.

• :out-of-range is used for those that are outside those bounds; it is not supported by IE.

Finally, let's talk about the red glow and the popup message. There is a red glow effect around an invalid element after submission in Firefox. (In Internet Explorer, it's a straightforward red border, without a glow effect.) Firefox exposes the effect as the :-moz-ui-invalid pseudoclass. You can override it as follows:

```css
:-moz-ui-invalid { box-shadow: none }
```

Internet Explorer, alas, does not expose its effect as a pseudoclass. This means we have reached the limit of what we can do with HTML and CSS alone. There are features we would like to change and features we would like to implement. This is where JavaScript comes back into play.

## JavaScript Form Validation

Thanks in large part to the HTML5 living standard, JavaScript now has a coherent API for form validation.

This rests on a relatively simple lifecycle for validation checking: Does this form element have a validation routine? If so, does it pass? If it fails, why did it fail? Interweaved with this process are logical access points for JavaScript, either through method calls or capturing events. It's a good system, although that's not to say it could not bear a bit of improvement. But let's not get ahead of ourselves.

The simplest way to check a form element's validity is to call checkValidity on it. The JavaScript object backing every form element now has this checkValidity method available. This method accesses the validation constraint set in the HTML for the element. Each constraint is tested against the element's current value. If any of the constraints fail, checkValidity returns false. If all pass, checkValidity returns true. Calls to checkValidity are not limited to individual elements. They can also be made against a form tag. If this is the case, the checkValidity call will be delegated to each of the form elements within the form. If all of the subcalls come back true (that is, all of the form elements are valid), then the form is valid as a whole. Conversely, if any of the subcalls come back false, then the form is invalid.

In addition to getting a simple Boolean answer on the validity of an element, we can find out why the element failed validity. Any element's validity property is an object containing all the possible reasons it could fail validation, known as a ValidityState object. We can iterate over its properties and, if one is true, we know that this is one of the reasons that the element failed its validity check. The properties are shown in Table 7-1.

Table 7-1. Validity State Properties

```js
valid //Whether or not the element's value is valid. Start with this property first.
valueMissing //A required element without a value.
patternMismatch //Failed a check against a regexp specified by pattern.
rangeUnderflow //Value is lower than min value.
rangeOverflow //Value is higher than max value.
stepMismatch //Value is not a valid step value.
tooLong //Value is greater (in characters) than maxlength allows.
typeMismatch //Value fails a check for the email or url input types.
customError //True if a custom error has been thrown.
badInput //A sort of catch-all for when the browser thinks the value is invalid but not for one of the reasons already listed; not implemented in Internet Explorer.
```

The act of checking an element's validity property runs the validity checks. It is not necessary to invoke element.checkValidity first.

Let's take a look at validity checks in action. First, Listing 7-3 shows the relevant section of our HTML.

Listing 7-3. Our HTML Form

```html
<body>
<h2>A basic form</h2>
<p>Please fill in the requested information.</p>
<form id="nameForm">
  <div id="fields">
    <label for="firstName">First Name:</label><br/>
    <input id="firstName" name="firstName" type="text" class="foo" required/><br/>
    <label for="lastName">Last Name:</label><br/>
    <input type="text" name="lastName" id="lastName" required/><br/>
    <label for="phone">Phone</label><br/>
    <input type="tel" id="phone"/><br/>
    <label for="age">Age (must be over 13):</label><br/>
    <input type="number" name="age" id="age" step="2" min="14" max="100"/><br/>
    <label for="email">Email</label><br/>
    <input type="email" id="email"/><br/>
    <label for="url">Website</label><br/>
    <input type="url" id="url"/><br/>
  </div>
  <div id="buttons">
    <input id="overallBtn" value="Check overall validity" type="button"/>
    <input id="validBtn" type="button" value="Display validity"/>
    <input id="submitBtn" type="submit" value="Submit the form"/>
    <input type="reset" id="resetBtn" value="Reset the form"/>
  </div>
</form>

<div>
  <h2>Validation results</h2>
  <div id="vResults"></div>
  <div id="vDetails"></div>
</div>

</body>
```

Note that the submit, reset, and validity checking buttons are in their own div. This makes it easier to use document.querySelectorAll to retrieve only the relevant form fields, which are in a separate div. Now, on to our JavaScript (Listing 7-4).

Listing 7-4. Form Validation and Validity

```js
window.addEventListener( 'DOMContentLoaded', function () {
  var validBtn = document.getElementById( 'validBtn' );
  var overAllBtn = document.getElementById( 'overallBtn' );
  var form = document.getElementById( 'nameForm' ); // Or document.forms[0]
  var vDetails = document.getElementById( 'vDetails' );
  var vResults = document.getElementById( 'vResults' );

  overallBtn.addEventListener( 'click', function () {
    var formValid = form.checkValidity();
    vResults.innerHTML = 'Is the form valid? ' + formValid;
  } );

  validBtn.addEventListener( 'click', function () {
    var output = '';

    var inputs = form.querySelectorAll( '#fields > input' );

    for ( var x = 0; x < inputs.length; x++ ) {
      var el = inputs[x];
      output += el.id + ' : ' + el.validity.valid;
      if (! el.validity.valid) {
        output += ' [';
        for (var reason in el.validity) {
          if (el.validity[reason]) {
            output += reason
          }
        }
        output += ']';
      }
      output += '<br/>'
    }

    vDetails.innerHTML = output;
  } );
} );
```

The entire code block is an event tied to when the DOM has loaded. Recall that we don't want to try to add event handlers to elements that may not have been created yet. First, we will retrieve relevant elements within the page: the two validity checking buttons, the output divs, and the form. Next, we will set up event handling for the overall validity check. Note that in this case, we are checking the validity of the entire form for simplicity's sake. We display the results of this check in the vResults div.

The second event handler covers checking the individual validity state of each of the form elements. We capture the appropriate elements by using querySelectorAll to grab all the input fields under the div with ID fields. (This winds up being simpler than writing an extended CSS selector to find input types that do not include submits, resets, and buttons.) After obtaining the elements we want, it's simple enough to iterate over the elements and check the valid subproperty of their validity property. If they are invalid (valid is false), then we print out the reason the field is invalid. We encourage you to try this out with a variety of different input values.

This demonstration reveals some interesting things. First, if you load up the page and click the "Display validity" button, the firstName and lastName fields are invalid (as you would expect, since they are empty), but the phone, age, email, and url fields (also empty) are valid! If the field is not required, an empty value is valid. Also, note that the email field has two validations, the implied validation of email, as well as a pattern requirement. Try entering an email that does not contain something like “@foo.com” and you will see it is possible to fail multiple validations at once. Firefox will also tell you that the value fails for typeMismatchbadInput if you enter an incomplete email address (say, just a username). You might be inclined to rely on only the valid property, but knowing which reason(s) the field failed validation is important information to convey to the user, who will not, after all, be able to submit the form successfully without passing the various validation constraints.

### Validation and Users

So far, we have spent most of our time on the technical aspects of form validation. We should also discuss when form validation should be performed. We have a number of options. Simply using the form validation API means that we have automatic form validation at submission time. And we have the capability to invoke validation on any given element when we want to, thanks to the checkValidity method. As a best practice, we should perform form validation as early as possible. What this means in practice depends on the field being validated. Start with the idea that we should validate on a change in the form field. Attach a change event handler to your form control and have it call checkValidity on that control. Working within the form validation API, this is a fairly straightforward answer to the question of when to validate.

But what if we aren't working within the form validation API? One of the more significant limitations of working within the form validation API is that it has no facility for custom validations. You cannot add a chunk of custom code, bound in a function, say, to run as a validation routine. But you will doubtless want to do that at some point. When you find yourself in this case, it still makes general and practical sense to tie the validation to the change event handler. There are possible exceptions. Consider a field that requires an Ajax call to validate its value, perhaps based on the first few characters entered into the field. In this case, you would tie validation to a keypress event, possibly integrating autosuggest functionality as well. In the next chapter, covering Ajax, we will look at an example of this.

At whatever stage you choose to validate, keep your users in mind. It is very frustrating to fill out a form and then find out that much of the data is invalid for whatever reasons. Users tend to be more accepting of in-line fixes, rather than being presented with a list of errors at submission time.

### Validation Events

Another addition to the form validation API is the fact that invalid form elements now throw an invalid event. This event is only thrown in response to a call to checkValidity. The checkValidity call can be made on either the element itself or the form that contains the element. The invalid event does not bubble. Forms do not have an invalid event themselves, despite the fact that forms can be invalid.

You can capture the event with the usual call to addEventListener on the emitting control. Once inside the event handler, the event object itself does not provide any validation-related information. You will have to retrieve the element via the event.target property, and then query its validity property to find out exactly why the element is invalid. But you can do something quite interesting with the preventDefault method of the event. When you invoke preventDefault, the browser's styling behavior for invalid elements will not be applied. Keep in mind that styling changes are only consistently applied when the form is submitted. (Firefox will apply styling changes if you change the value of the form control and blur away from it.) This means different things for different browsers:

•Chrome, which does not style invalid elements but does give them a popup message, will suppress the popup for that element.

•Firefox, which has both popup and styling, will suppress the popup but will not suppress or prevent the red glow effect around the element.

•Internet Explorer, which has both popup and a red border around the element, will suppress both the popup and the border around the element.

Let's look at an example that shows this behavior in action. Start with a relatively familiar HTML form in Listing 7-5.

Listing 7-5. Validity Events Form

```html
<!DOCTYPE html>
<html>
<head>
  <title>A basic form</title>
  <style>
    input:invalid {
      background-color: yellow
    }
  </style>
</head>
<body>
<h2>A basic form</h2>

<p>Please provide your first and last names.</p>

<form id="nameForm">
  <fieldset>
    <label for="firstName">First Name:</label><br/>
    <input id="firstName" name="firstName" type="text" required/><br/>
    <label for="lastName">Last Name:</label><br/>
    <input type="text" name="lastName" id="lastName" required/><br/>
  </fieldset>
  <div>
    <input type="submit" value="Submit the form"/> <input type="reset" value="Reset the
form"/>

  </div>
  <div>
    <input id="firstNameBtn" type="button" value="Check first name validity."/>
    <input id="formBtn" type="button" value="Check form validity"/>
    <input id="preventBtn" type="button" value="Prevent default behavior"/>
    <input id="restoreBtn" type="button" value="Restore default behavior"/>
  </div>
</form>
<div id="vResults"></div>
<script src="listing_7_5.js"></script>
</body>
</html>
```

Note that we have added styling for invalid input elements. This styling is not associated with the default behavior for an invalid event. Let's look at the backing code (Listing 7-6).

Listing 7-6. Validity Events in JavaScript

```js
window.addEventListener( 'DOMContentLoaded', function () {
  var outputDiv = document.getElementById( 'vResults' );
  var firstName = document.getElementById( 'firstName' );
   firstName.addEventListener("focus", function(){
    outputDiv.innerHTML = '';
   });
  function preventDefaultHandler( evt ) {
    evt.preventDefault();
  }
  firstName.addEventListener( 'invalid', function ( event ) {
    outputDiv.innerHTML = 'firstName is invalid';
  } );
  document.getElementById( 'firstNameBtn' ).addEventListener( 'click', function () {
    firstName.checkValidity();
  } );
  document.getElementById( 'formBtn' ).addEventListener( 'click', function () {
    document.getElementById( 'nameForm' ).checkValidity();
  } );
  document.getElementById( 'preventBtn' ).addEventListener( 'click', function () {
    firstName.addEventListener( 'invalid', preventDefaultHandler );
  } );
  document.getElementById( 'restoreBtn' ).addEventListener( 'click', function () {
    firstName.removeEventListener( 'invalid', preventDefaultHandler );
  } );
} );
```

As usual, all of our code is activated after the DOMContentLoaded event has fired. We have a basic event handler for invalid events on the firstName field, which outputs to the vResults div. Then we add event handlers for the specialized buttons. First we create two convenience buttons: one for checking the validity of the firstName field, the other for checking the validity of the entire form. Then we add behavior for overriding or restoring the default behavior associated with invalid events. Try it out!

## Customizing Validation

We now have almost all the tools available to us to control form validation comprehensively. We can choose which validations to activate. We can control when validation is performed. And we can capture invalid events and prevent the default behavior (particularly as regards styling) from firing. As discussed before, we cannot customize the actual validation routines (alas). So what is left for us to work with? We might like to control the message in the validation popup users see on submitting a form. (The look and feel of the popup is also not customizable. Remember that we mentioned there were some shortcomings with the API and its implementation?)

To change the validation message that appears when a form field is invalid, use the setCustomValidity function associated with the form control. Pass setCustomValidity a string as an argument, and that string will appear as the text of the popup. This does have some other side effects, though. In Firefox, the field will show, at page load time, as invalid, with the red glow effect. Using setCustomValidity with either Internet Explorer or Chrome has no effect at page load time. The Firefox styling can be turned off, as mentioned earlier, by overriding the :-moz-ui-invalid pseudoclass. But more problematic is that when setCustomValidity is used, the customError property of the validityState of the form control in question is set to true. This means that the valid property of the validityState is false, which means it reads as invalid. All for simply changing the message associated with the validity check! This is unfortunate and renders setCustomValidity nearly useless.

An alternative would be to use a polyfill. There is a long list of polyfills, not only for form validation but also for other HTML5 items that may not have support on every browser you need to work on. You can find them here:

https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-Browser-Polyfills

### Preventing Form Validation

There is one aspect of form validation we have not explored: turning it off. The majority of this chapter has focused on using the new form validation API, and exploring its limits. But what if there were a deal-breaker with the API, some feature (or bug, or inconsistency) that prevented our wholesale use of the API? Were this the case, we would want to do two things: discontinue the automatic form validation, and substitute our own. It is the former which interests us, as the latter is simply the case of reimplementing an API according to our whims. To turn off form validation behavior, add the novalidate attribute to the form element. You can prevent behavior per submit-button click by adding the formnovalidate attribute to a submit button, but this does not turn off form validation for the form as a whole. Since we might want to substitute our own customized API for form validation, we want to use the novalidate attribute to deactivate form validation (for the parent element) entirely.

## Summary

In this chapter we spent the bulk of our time looking at the new JavaScript form validation API. Within its constraints, it is a powerful tool for automatically validating user data. Validation happens automatically on form submission, and we can validate at any time (or times) of our own choosing as well. Validation can be turned off if needed. We can customize the appearance of valid and invalid elements, and can even customize the message displayed when an element is invalid.

The API is not without problems. it lacks some critical customization capabilities, like allowing styling for error messages, or custom validation routines. There are small implementation differences between the major browsers that can be a bit of a pain to clean up after. Some official parts of the API (think the willValidate property) are not currently implemented and other parts (setCustomValidity) have crippling problems.

Overall the API is a big step forward for JavaScript, HTML, CSS, and the browser. We look forward to seeing how it will be refined in the future.
