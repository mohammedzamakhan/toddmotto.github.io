---
layout: post
permalink: /progressively-enhancing-html5-forms-creating-a-required-attribute-fallback-with-jquery/
title: Progressively enhancing HTML5 forms, creating a required attribute fallback with jQuery
external: http://tech.pro/tutorial/1318/progressively-enhancing-html5-forms-creating-a--required--attribute-fallback-with-jquery
path: 2013-05-20-progressively-enhancing-html5-forms-creating-a-required-attribute-fallback-with-jquery.md
tags:
- jQuery
- Forms
- HTML5
- JavaScript
---

Note: this article first featured on [Tech.Pro](//tech.pro/tutorial/1318/progressively-enhancing-html5-forms-creating-a--required--attribute-fallback-with-jquery), please check out the full article there!

HTML5 required attributes are a fantastic addition to the HTML5 spec, they save a lot of time when working on client-side form validation and enhance the user's experience. The HTML5 required attribute provides us with full client-side 'required' checking and validation, which is easy to add to several types of form elements.

<div class="download-box">
  <a href="//toddmotto.com/labs/required-fallback" onclick="_gaq.push(['_trackEvent', 'Click', 'Demo Required Fallback, 'Required Fallback Demo']);">Demo</a>
  <a href="//toddmotto.com/labs/required-fallback/required-fallback.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download Required Fallback, 'Required Fallback Download']);">Download</a>
  <a href="//github.com/toddmotto/required-fallback" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork Required Fallback', 'Required Fallback Fork']);">Fork</a>
</div>

The required attribute is developer friendly. It's a boolean attribute too which means we don't need to supply it any data, it's either present or not. We can add it simply to form elements like so:

{% highlight html %}
<input type="text" required>
{% endhighlight %}

In browsers that support the attribute, you don't need to do anything when it comes to alerting users they haven't filled/checked in an input field (or textarea, radio buttons and so on...)

If someone were to not fill in/check a required input, the form would not submit the information and the user would be notified by a small popup above the relevant field that they needed to fill it in.

But what about legacy browsers?

I'm a big fan of writing as much code as I can, obviously when a project's turnaround time is limited you'll use any plugins/scripts at hand, but to really understand any polyfills/scripts - it's great to read through the code and see what they're actually doing. This will help you to understand the process, their capabilities and limitations.

In this tutorial, we are going to create a small feature detection for the HTML5 required attribute, and write a handy, markup agnostic (i.e. not limited to knowing your markup [classes and IDs etc.]) so you can take it away, build from or integrate as is. This will be a robust required attribute fallback that will allow us to do what the HTML5 required attribute does:

- Stop the form submitting if fields are left empty
- Add a custom notification message on any empty/unchecked fields

### HTML5 feature detecting
When working with HTML5, it's a good idea to feature detect on the client-side. A lot of people like using Modernizr (a feature detect library). I find it a little bloated for my needs as a quick Google search can find you any feature detect you need should you need to. I've been adding detects to a small library of my own which I keep, these are all handy one-liners that I reuse when needed. Let's get going with the feature detect!

We'll be using the JavaScript _in_ method, which will test the outcome inside an input element and run it through an _if_ statement:

{% highlight javascript %}
var supportsRequired = 'required' in document.createElement('input')
if (supportsRequired) {
  // support
} else {
  // no support
}
{% endhighlight %}

We'll want to run any fallbacks when it isn't supported, the _else_ part of the script. We can however invert the _if_ statement with a JavaScript bang:

{% highlight javascript %}
var supportsRequired = 'required' in document.createElement('input')
if (!supportsRequired) {
  // no support
}
{% endhighlight %}

### Adding the required attributes
This is easily done as you can see from the introduction, adding a required attribute to our markup is really simple (here I've created a quick form with some different input fields). The below fields include various input types: _text_, _email_, _tel_, _url_, _radio_, _checkbox_ as well as the _textarea_ element. Let's add the required attributes where appropriate (we only need to add the required attribute once to radio and checkboxes with the same _name_ attribute):

{% highlight html %}
<form class="form" action="/" method="post">
  <div class="form-row">
    <label for="name" class="form-label">Name *</label>
    <div class="form-field">
      <input id="name" name="name" placeholder="Please enter your name" type="text" required>
    </div>
  </div>
  <div class="form-row">
    <label for="email" class="form-label">Email *</label>
    <div class="form-field">
      <input id="email" name="email" placeholder="Please enter your email address" type="email" required>
    </div>
  </div>
  <div class="form-row">
    <label for="radio" class="form-label">Radio Buttons *</label>
    <div class="form-field">
      <span class="form-radios">Select 1: </span>
      <input id="radio" name="radiobutton" value="selection-one" type="radio" required>
      <span class="form-radios">Select 2: </span>
      <input name="radiobutton" value="selection-two" type="radio">
    </div>
  </div>
  <div class="form-row">
    <label for="checkbox" class="form-label">Checkboxes *</label>
    <div class="form-field">
      <span class="form-radios">Select 1: </span>
      <input id="checkbox" name="checkbox" type="checkbox" required>
      <span class="form-radios">Select 2: </span>
      <input name="checkbox" type="checkbox">
    </div>
  </div>
  <div class="form-row">
    <label for="tel" class="form-label">Telephone *</label>
    <div class="form-field">
      <input id="tel" name="telephone" placeholder="Please enter your number" type="tel" required>
    </div>
  </div>
  <div class="form-row">
    <label for="website" class="form-label">Website *</label>
    <div class="form-field">
      <input id="website" name="website" placeholder="Begin with http://" type="url" required>
    </div>
  </div>
  <div class="form-row">
    <label for="message" class="form-label">Message *</label>
    <div class="form-field">
      <textarea id="message" name="message" placeholder="Include all the details you can" required></textarea>
    </div>
  </div>
  <div class="form-row">
    <button name="submit" type="submit" class="form-submit">Send Email</button>
  </div>
</form>
{% endhighlight %}

### Required attribute loop
Now we've got a feature detect and a form full of required elements, we need to get working on the script. First of all, I'm going to loop through the elements with the required attributes, and run a feature detect inside it. This will let us extend the script at a later date in the future if we want to do anything else:

{% highlight javascript %}
$('[required]').each(function () {
  if (!supportsRequired) {
    // No support 
  }
})
{% endhighlight %}

The next step is going to be swapping the required attributes (which will be unsupported by any browser reading the script) to swap them for classes. This will help when dealing with styling for older browsers as well as continuation of selectors throughout the script.

Let's remove the attribute using jQuery's _removeAttr_ method, and add a class called 'required' - a nice straight swap. I've found it really manageable to append any required messaging initially, and simply setting them to _display:none;_ form the get-go. This is good for a few reasons; there should be a lot less (if any) flickering if any required attributes need showing, and they aren't appended when they're needed - they're already there on demand. It will help make our script a lot less bloated and easier to read later on too.

{% highlight javascript %}
$('[required]').each(function () {
  if (!supportsRequired) {
    var self = $(this)
    self.removeAttr('required').addClass('required')
    self.parent().append('<span class="form-error">Required</span>')
  }
})
{% endhighlight %}

### Form submission
Attributes are all setup now for form submission, which of course will only fire if a _required_ class exists, meaning we don't need to do another feature check and can simply include a _$('.required')_ selector inside the form handler. Let's look at how we can set that up. Our form has a class of 'form' for simplicity and is the only markup-reliant selector our script will need, the rest will automatically do its magic.

{% highlight javascript %}
$('.form').on('submit', function () {
  // on submit
})
{% endhighlight %}

I've found using jQuery's _.on_ methods are much more flexible, you can easily include event delegation as well as chaining event listeners, which we'll come onto later. Let's progress with the next loop. As it stands, all of our required attributes are now classes, allowing us to target them on submit:

{% highlight javascript %}
$('.form').on('submit', function () {
  $('.required').each(function(){
    // loop through required classes
  })
})
{% endhighlight %}

### Checking empty values
Inside the loop, we need to think of the next step; checking empty values. Let's start with the easiest - empty input fields. This can easily be done like so:

{% highlight javascript %}
if ($(element).val() === '') {
  // empty
} else {
  // not empty
}
{% endhighlight %}

It'd be good to setup the loop now to incorporate this:

{% highlight javascript %}
$('.form').on('submit', function () {
  $('.required').each(function(){
    var self = $(this)
    if (self.val() === '') {
      // empty
    } else {
      // not empty
    }
  })
})
{% endhighlight %}

Nice and easy. So what do we need to do when the field is empty? Well, two things; first we need to stop the form submitting. Stopping the form submitting is fairly easy, we capture the _submit_ event and prevent default, like so (see we're passing _e_ through the function and calling the preventDefault method on it):

{% highlight javascript %}
$('.form').on('submit', function (e) {
  $('.required').each(function(){
    var self = $(this)
    if (self.val() === '') {
      e.preventDefault()
    } else {
      // submit otherwise
    }
  })
})
{% endhighlight %}

Next we need to show the appended message:

{% highlight javascript %}
$('.form').on('submit', function (e) {
  $('.required').each(function(){
    var self = $(this)
    if (self.val() === '') {
      self.siblings('.form-error').show() // show error
      e.preventDefault() // stop submission
    } else {
      // submit otherwise
    }
  })
})
{% endhighlight %}

The events will both fire together, doing exactly what we'd like it to. Next I'm going to fill in the _else_ part to hide any errors:

{% highlight javascript %}
$('.form').on('submit', function (e) {
  $('.required').each(function(){
    var self = $(this)
    if (self.val() === '') {
      self.siblings('.form-error').show() // show error
      e.preventDefault() // stop submission
    } else {
      self.siblings('.form-error').hide() // hide errors
    }
  })
})
{% endhighlight %}

### Detecting if radio/checkboxes aren't checked
Now we've checked if the values are empty (thankfully this covers input types text, email, tel, url and textareas), which makes the process fairly seamless. The next bit is a little trickier, radio and checkboxes. First we need to check _self_ to see if it's an input type radio or checkbox (self is a variable name I've created to use instead of _$(this)_):

{% highlight javascript %}
if (self.is(':checkbox') || self.is(':radio')) {
  // is a checkbox or radio
}
{% endhighlight %}

Next we'll be using the jQuery _:not_ pseudo, which allows us to detect whether something is 'not' something else. Here's how we'll use it:

{% highlight javascript %}
self.is(':not(:checked)')
{% endhighlight %}

This will check if the element(s) are not checked for us. If you remember from earlier on, I mentioned that you need to apply the required attribute once to radios and checkboxes with the same name. This is because we can run a condition to say that at least one input with the same name has to be checked, otherwise the users selection will only be limited to the checkbox with the required attribute (we basically assign the required attribute once to a group, and any input in that group requires a selection) - or it won't let them through. Here's how we add that:

{% highlight javascript %}
self.is(':not(:checked)') && $('input[name=' + self.attr('name') + ']:checked').length === 0
{% endhighlight %}

Translating to English, this says, if this isn't checked, and (&&) the input with the same name hasn't been checked, we can do something. Converting this to a shorthand _if_ statement is best as we can add this easily to our empty input field check too. I'll create a variable called _checked_ and test against it:

{% highlight javascript %}
var checked = (self.is(':checkbox') || self.is(':radio')) 
? self.is(':not(:checked)') && $('input[name=' + self.attr('name') + ']:checked').length === 0 
: false

if (checked) {
  // stop submit, show error
}
{% endhighlight %}

The above will fire if no input type radio or checkbox have been selected with the same name. We can save some code and test against empty input and radios/checkboxes at the same time using the _or_ operator (||):

{% highlight javascript %}
if (self.val() === '' || checked) {
  // if empty value, or isn't checked
}
{% endhighlight %}

While it's looping our code will make the necessary condition checks.

### Joining components
Putting the full loop and submit handler together, we can paint a picture of how our script is looking, with comments to recap:

{% highlight javascript %}
// submit the form
$('.form').on('submit', function (e) {

  // loop through class name required
  $('.required').each(function () {
  
    // this
    var self = $(this)
    
    // check shorthand if statement for input[type] detection
    var checked = (self.is(':checkbox') || self.is(':radio')) 
    ? self.is(':not(:checked)') && $('input[name=' + self.attr('name') + ']:checked').length === 0 
    : false
    
    // run the empty/not:checked test
    if (self.val() === '' || checked) {
        
      // show error if the values are empty still (or re-emptied)
      // this will fire after it's already been checked once
      self.siblings('.form-error').show()
      
      // stop form submitting
      e.preventDefault()
    
    // if it's passed the check
    } else {
    
      // hide the error
      self.siblings('.form-error').hide()
      
    }
    
  })
  
  // all other form submit handlers here

})
{% endhighlight %}

You'll notice the penultimate line states 'all other form submit handlers here' - this is where you can carry on processing anything you need to _post_ your data to the server, could be AJAX or other script validators you may need.

### Enhancing behaviour
By default, the HTML5 required validator will only fire on form submission, I'm going to include an optional script that will notify the user that the field is required should they decide to leave the input field without filling anything in. This isn't typical behaviour of the required attribute, but I think it really benefits front-end validation as it tells the user straight away they need to fill it in.

Coming back to what I mentioned earlier about chaining event listeners, we want to listen for a few different events now on this separate part of the script:

{% highlight javascript %}
$(element).on('blur change', function () {
  // listening for blur and change events 
})
{% endhighlight %}

The _blur_ event is fired when a user leaves an input field, so it may be worth informing them the field is required at this stage and no later. Also, radio inputs and checkbox inputs will fire a _change_ event, so this script will intelligently cater for both. We'll be reusing a few components from earlier to achieve this, so much will be familiar.

First, I'm going to listen on inputs and textareas, the only two elements we're using above (it seems redundant to listen for individual input types and make our selectors complicated):

{% highlight javascript %}
$('input, textarea')
{% endhighlight %}

We want to tell jQuery that these inputs and textareas are the parent of our form, which is done by using a comma to separate the selectors:

{% highlight javascript %}
$('input, textarea', '.form')
{% endhighlight %}

And then run the event listening function:

{% highlight javascript %}
$('input, textarea', '.form').on('blur change', function () {
  // listening for input and textarea blur/changes inside .form
})
{% endhighlight %}

We don't need to create a new loop, as the jQuery library will power this automatically for us, so we can call upon the _$(this)_ method again, with all the reused components:

{% highlight javascript %}
// key change on all form inputs
$('input, textarea', '.form').on('blur change', function () {

  // this
  var self = $(this)
    
  // check shorthand if statement for input[type] detection
  var checked = (self.is(':checkbox') || self.is(':radio')) 
  ? self.is(':not(:checked)') && $('input[name=' + self.attr('name') + ']:checked').length === 0 
  : false
  
  // if empty on change, i.e. if data is removed
  if (self.val() === '' || checked) {
  
    // show/keep the error in view
    self.siblings('.form-error').show()
  
  // if there's a value or checked
  } else {
  
    // hide the error
    self.siblings('.form-error').hide()
    
  }
  
})
{% endhighlight %}

The nice part about listening for blur/change events is that the check will fire, fire and fire. Which means if the user enters data, and then removes it - the script will know and show the relevant message. This has no interaction however with the form's submission, it's purely an additional add-on for validation on-the-fly before submission.

### Full scripts
Putting everthing together, it looks as follows:

{% highlight html %}
<script src="js/jquery.min.js"></script>
<script>
$(function () {
  
  // feature detect
  var supportsRequired = 'required' in document.createElement('input')
  
  // loop through required attributes
  $('[required]').each(function () {
  
    // if 'required' isn't supported
    if (!supportsRequired) {
    
      // this
      var self = $(this)
    
      // swap attribute for class
      self.removeAttr('required').addClass('required')
      
      // append an error message
      self.parent().append('<span class="form-error">Required</span>')
      
    }
    
  })
  
  // submit the form
  $('.form').on('submit', function (e) {
  
    // loop through class name required
    $('.required').each(function () {
    
      // this
      var self = $(this)
      
      // check shorthand if statement for input[type] detection
      var checked = (self.is(':checkbox') || self.is(':radio')) 
      ? self.is(':not(:checked)') && $('input[name=' + self.attr('name') + ']:checked').length === 0 
      : false
      
      // run the empty/not:checked test
      if (self.val() === '' || checked) {
          
        // show error if the values are empty still (or re-emptied)
        // this will fire after it's already been checked once
        self.siblings('.form-error').show()
        
        // stop form submitting
        e.preventDefault()
      
      // if it's passed the check
      } else {
      
        // hide the error
        self.siblings('.form-error').hide()
        
      }
      
    })
    
    // all other form submit handlers here
  
  })
  
  // key change on all form inputs
  $('input, textarea', '.form').on('blur change', function () {
  
    // this
    var self = $(this)
      
    // check shorthand if statement for input[type] detection
    var checked = (self.is(':checkbox') || self.is(':radio')) 
    ? self.is(':not(:checked)') && $('input[name=' + self.attr('name') + ']:checked').length === 0 
    : false
    
    // if empty on change, i.e. if data is removed
    if (self.val() === '' || checked) {
    
      // show/keep the error in view
      self.siblings('.form-error').show()
    
    // if there's a value or checked
    } else {
    
      // hide the error
      self.siblings('.form-error').hide()
      
    }
    
  })

})
</script>
{% endhighlight %}

### Script testing
As the script runs according to the result of an initial feature detect, we can simply make the script run when the required attribute _is_ supported, which it no doubt is if you're reading this. Simply amend this line during the development stage of your project to test the fallback features:

{% highlight javascript %}
// no support
if (!supportsRequired) {...}
{% endhighlight %}

Change to:

{% highlight javascript %}
// supports now
if (supportsRequired) {...}
{% endhighlight %}

Which drops the bang (!). This now says 'if it supports the required attribute' - do something. And there you have it.

### Conclusion
HTML5 required attributes are just one piece of the front-end validation puzzle, but you can see their instant power and methods to improving user interaction. Just remember, not all users enable JavaScript, which means by turning it off they can bypass your system. It's best practice to include a server-side validator alongside your client-side validator, for fallback methods as well as a second reassurance. You can also sanitize data server-side and make sure no malicious code gets through.

<div class="download-box">
  <a href="//toddmotto.com/labs/required-fallback" onclick="_gaq.push(['_trackEvent', 'Click', 'Demo Required Fallback, 'Required Fallback Demo']);">Demo</a>
  <a href="//toddmotto.com/labs/required-fallback/required-fallback.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download Required Fallback, 'Required Fallback Download']);">Download</a>
  <a href="//github.com/toddmotto/required-fallback" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork Required Fallback', 'Required Fallback Fork']);">Fork</a>
</div>

### Extra: Customising HTML5 popups
We won't be needing to do this for our fallback, but essentially we just need to add a line of JavaScript to the targeted element. This uses the full HTML5 validation whilst customising it to your advantage. To customise the text, we can set a custom message like so:

{% highlight html %}
<input class="test" type="text" required>
{% endhighlight %}

And the JavaScript:

{% highlight javascript %}
document.querySelector('.test').setCustomValidity('Custom alert message, please fill this field in.')
{% endhighlight %}
