---
layout: post
permalink: /moving-from-ng-model-parsers-to-ng-model-validates-ng-messages/
title: Moving from ngModel.$parsers /ng-if to ngModel.$validators /ngMessages
path: 2015-10-18-moving-from-ng-model-parsers-to-ng-model-validates-ng-messages.md
tags:
- Validation
- Forms
- Parsers
- Formatters
- Angular
---

Implementing custom Model validation is typically done by extending the built-in `$error` Object bound to AngularJS form models, such as a simple `<input>`.

Prior to Angular 1.3, custom validation was done by injecting a function into the `ngModel.$parsers` Array pipeline and manually setting validation states using `$setValidity` to tell Angular whether your custom validation logic passed. For example you'd set `$setValidity('visa', true);` if the Model value matched a Visa credit card expression format.

In Angular 1.3+, we have the `$validators` pipeline, which requires no manual setting of validation states, and is also just an Object that we can add properties to, which is much nicer than pushing a function into an Array.

Let's take a look at the old school way of doing things and how we can shift into using `ngModel.$validators`.

### Old school $parsers

Let's take some basic form markup, binding `name="myForm"` to the `<form>` element so Angular takes control of our form and validation states. Next we'll add an `<input>` with the name `creditCard`, which builds up the Model Object internally so we can access `myForm.creditCard` and handle our validation. I've added a `validate-visa` attribute, which will serve as the Directive bound to the input, so we can capture the Model and validate it.

{% highlight html %}
<form name="myForm">
  <h3>Visa validation ($parsers)</h3>
  <input type="text" name="creditCard" ng-model="creditCardModel" validate-visa>
  {% raw %}{{ myForm.creditCard.$error | json }}{% endraw %}
</form>
{% endhighlight %}

Adding the `{% raw %}{{ myForm.creditCard | json }}{% endraw %}` expression under the input allows us to get sight of the available validation properties, such as `required` and anything else we tell Angular to validate against. The validation Object for this particular field (note `$error` property) would look something like this:

{% highlight javascript %}
{
  "$validators": {},
  "$asyncValidators": {},
  "$parsers": [],
  "$formatters": [
    null
  ],
  "$viewChangeListeners": [],
  "$untouched": true,
  "$touched": false,
  "$pristine": true,
  "$dirty": false,
  "$valid": false,
  "$invalid": true,
  "$error": {},
  "$name": "creditCard",
  "$options": null
}
{% endhighlight %}

This generated `myForm.creditCard.$error` Object is where we need to hook into, so end up with `myForm.creditCard.$error.visa`. At this point we want to conditionally swap out DOM based on the Boolean value of this property. If `myForm.creditCard.$error.visa` is `true`, create the DOM, otherwise `false` Angular will remove it entirely.

{% highlight html %}
<form name="myForm">
  <h3>Visa validation ($parsers)</h3>
  <input type="text" name="creditCard" ng-model="creditCardModel" validate-visa>
  <p ng-if="myForm.creditCard.$error.visa" class="invalid">
    Not a valid Visa format
  </p>
</form>
{% endhighlight %}

On the JavaScript side, we need to actually write the logic behind `validate-visa` to tie into the Model and set the states. This is done using `ngModel.$parsers`, which looks like so:

{% highlight javascript %}
function validateVisa() {

  function link($scope, $element, $attrs, $ctrl) {
    var VISA_REGEXP = /^4[0-9]{12}(?:[0-9]{3})?$/;
    function visaParser(viewValue) {
      var isValid = VISA_REGEXP.test(viewValue);
      $ctrl.$setValidity('visa', isValid);
      return isValid ? viewValue : undefined; 
    }
    $ctrl.$parsers.unshift(visaParser);
  }

  return {
    require: 'ngModel',
    link: link
  };
  
}

angular
  .module('app')
  .directive('validateVisa', validateVisa);
{% endhighlight %}

The annotated version:

{% highlight javascript %}
// create a validateVisa function
function validateVisa() {

  // link function
  function link($scope, $element, $attrs, $ctrl) {
    // Some basic Visa Regular Expression
    var VISA_REGEXP = /^4[0-9]{12}(?:[0-9]{3})?$/;
    // visaParser function, passing in the current viewValue
    function visaParser(viewValue) {
      // a Boolean variable evaluated by RegExp.test(String)
      var isValid = VISA_REGEXP.test(viewValue);
      // Manually set the validity of the "visa" property on 
      // the "$error" Object bound to the Model.
      // Note: $ctrl is the fourth argument in the "link" function
      // as we're requiring "ngModel" (see below in the return {} statement)
      $ctrl.$setValidity('visa', isValid);
      // return the "viewValue" if it's valid or undefined 
      // so Angular doesn't set the value
      return isValid ? viewValue : undefined; 
    }
    // unshift the "visaParser" function into the "$parsers" Array
    $ctrl.$parsers.unshift(visaParser);
  }

  // export the Directive Object
  // which requires the "ngModel" Controller and
  // binds the above "link" function
  return {
    require: 'ngModel',
    link: link
  };
  
}

angular
  .module('app')
  .directive('validateVisa', validateVisa);
{% endhighlight %}

Observations from this are that the syntax of unshifting a function into an Array isn't very slick, and also we're manually setting the validation state, passing in a String and Boolean, which seems a very procedural way to do things.

At this point, our `$error` Object bound to this input would look like this:

{% highlight javascript %}
{
  ...
  "$error": {
    "visa": true
  },
  ...
}
{% endhighlight %}

When `visa` is `true`, the field is invalid, `false` for valid.

### New school $validators

In Angular 1.3, we've a much better way of doing things! Just like before, we `require: 'ngModel'` into our Directive and hook into `$ctrl`, but instead of using `$parsers` we can bind a function straight to the `$validators` Object:

{% highlight javascript %}
function validateVisa() {

  function link($scope, $element, $attrs, $ctrl) {
    var VISA_REGEXP = /^4[0-9]{12}(?:[0-9]{3})?$/;
    $ctrl.$validators.visa = function visaParser(modelValue, viewValue) {
      var value = modelValue || viewValue;
      return (VISA_REGEXP.test(value));
    };
  }

  return {
    require: 'ngModel',
    link: link
  };
  
}

angular
  .module('app')
  .directive('validateVisa', validateVisa);
{% endhighlight %}

The above doesn't even need annotating, any `$validator` property we add becomes the property name bound to `$error`, and we just need to return a Boolean. Super simple and much clearer to read. Usage as the Directive from an HTML perspective is identical, it's just the difference of how we implement the validation that changes.

The next stage in our mission to improving our Angular validation is by moving to `ngMessages`, let's explore the old and new ways we're doing things.

### Old school ng-if

Using `ng-if` is super simple, our initial example code above uses it. We'll tell Angular to conditionally swap DOM based on a property state bound to the `$error` Object. Here's an example extending our earlier HTML:

{% highlight html %}
<form name="myForm">
  <h3>Visa validation (ngIf)</h3>
  <input 
    type="text" 
    name="creditCard" 
    ng-model="creditCardModel" 
    required=""
    ng-minlength="13"
    ng-maxlength="16"
    validate-visa>
  <p ng-if="myForm.creditCard.$error.required" class="invalid">
    This field is required
  </p>
  <p ng-if="myForm.creditCard.$error.visa" class="invalid">
    Not a valid Visa format
  </p>
  <p ng-if="myForm.creditCard.$error.minlength" class="invalid">
    Minimum of 13 characters
  </p>
  <p ng-if="myForm.creditCard.$error.maxlength" class="invalid">
    Maximum of 16 characters
  </p>
</form>
{% endhighlight %}

This is a very manual process of dealing with each `$error` property which gets (and looks) repetitive. Let's look at the `ngMessages` and how it addresses a more encapsulated way of dealing with the validation message side of things.

### New school ngMessages and ngMessage

`ngMessages` is essentially a conditional DOM switch case that you pass a single `$error` Object into and it'll render out the necessary validation messages. Unlike the above `ng-if` approach, we're passing `myForm.creditCard.$error` only _once_ into `ngMessages`. The `ngMessage` Directive will take a property of that `$error` Object and if it evaluates to `true`, the corresponding error message will be rendered.

{% highlight html %}
<form name="myForm">
  <h3>Visa validation (ngMessages)</h3>
  <input 
    type="text" 
    name="creditCard" 
    ng-model="creditCardModel" 
    required=""
    ng-minlength="13"
    ng-maxlength="16"
    validate-visa>
  <div ng-messages="myForm.creditCard.$error">
    <p ng-message="required" class="invalid">
      This field is required
    </p>
    <p ng-message="visa" class="invalid">
      Not a valid Visa format
    </p>
    <p ng-message="minlength" class="invalid">
      Minimum of 13 characters
    </p>
    <p ng-message="maxlength" class="invalid">
      Maximum of 16 characters
    </p>
  </div>
</form>
{% endhighlight %}

For reusable/generic validation states we should use `ngMessageInclude` for things such as validating an email address, or letting the user know it's a required field. We can define and pull in an existing HTML template to contain all generic messages, allowing some decent encapsulation and management, a quick example:

{% highlight html %}
<script type="text/ng-template" id="generic-messages">
  <div ng-message="required">This field is required</div>
  <div ng-message="minlength">This field is too short</div>
</script>
{% endhighlight %}

And to pull it inside an existing `ngMessages` setup:

{% highlight html %}
<div ng-messages="myForm.creditCard.$error">
  <div ng-messages-include="generic-messages"></div>
  <p ng-message="visa" class="invalid">
    Not a valid Visa format
  </p>
  <p ng-message="minlength" class="invalid">
    Minimum of 13 characters
  </p>
  <p ng-message="maxlength" class="invalid">
    Maximum of 16 characters
  </p>
</div>
{% endhighlight %}

There are some other powerful features well worth checking out inside `ngMessages`, see [the documentation](https://docs.angularjs.org/api/ngMessages) for more.
