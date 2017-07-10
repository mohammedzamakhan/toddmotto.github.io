---
layout: post
permalink: /is-it-time-to-drop-jquery-essentials-to-learning-javascript-from-a-jquery-background/
title: Is it time to drop jQuery&#63; Essentials to learning JavaScript from a jQuery background
path: 2013-06-25-is-it-time-to-drop-jquery-essentials-to-learning-javascript-from-a-jquery-background.md
tags:
- jQuery
- Opinion
- JavaScript
---

jQuery has been a godsend to pretty much all of us front-end developers since its release, its intuitive methods, easy functions make light work of JavaScript's loosely typed language. JavaScript is hard, it's hard to get into, it's much harder than jQuery. But the time is nearly here, going native is going to be the future of front-end - HTML5.

HTML5 doesn't just mean a few extra HTML elements, if you're putting down on your CV/Resume that you know HTML5 because you've used the new elements, then think again! HTML5 covers such a mass of technology, and also alongside it comes ECMAScript 5, the future of JavaScript. Combining HTML5 APIs, of which most require JavaScript, we need to adopt a more native structure of working as each day jQuery becomes less important, and here's why.

This article takes a jQuery lover through some of the harder, more misunderstood, JavaScript methods, functions and more to show how native technology has caught up, how it's not as difficult as it seems and that native JavaScript is probably going to hit you like a brick in the face fairly soon - if it hasn't already. As a front-end developer I am pretty passionate about knowing my tech, and admittedly I began with jQuery and moved to learning JavaScript, I know many others have too. This article is here to talk anyone looking to dive into native JavaScript development over jQuery, and should hopefully open up some doors into the future of your coding.

### Selectors
jQuery selectors are the big seller, we don't even have to think about it, selecting our elements is a no brainer, it's super simple. jQuery uses Sizzle, an engine also created by the jQuery Foundation (but available as a standalone) to use as its selector engine. The mighty code behind Sizzle will make you think twice before overcomplicating your selectors, and the raw JavaScript alternative will make you think twice about jQuery altogether!

#### Class selectors
JavaScript had no native _className_ method for grabbing elements with classes until fairly recent, which I feel has hindered its popularity from the start. Classes are the best for our HTML/CSS development, but weren't well supported with native JavaScript - makes sense not to want to 'learn JavaScript' and go with jQuery. Until now.

Let's look at the options:
{% highlight javascript %}
// jQuery
$('.myClass');

// JavaScript
document.getElementsByClassName('myClass');
{% endhighlight %}

This returns a NodeList. A Node is a JavaScript term for element Object, and a NodeList is an ordered list of Nodes.

_Pro tip:_ the difference between jQuery and native JavaScript when using selectors like these, is that they return a NodeList that you then have to deal with. jQuery takes care of all this for you, covering up what's really happening - but it's really important to know what's happening.

#### ID selectors
The easiest of the pack:
{% highlight javascript %}
// jQuery
$('#myID');

// JavaScript
document.getElementById('myID');
{% endhighlight %}

Returns a single Node.

#### Tags
As easy as the ID selector, the tag name selector returns a NodeList too:

{% highlight javascript %}
// jQuery
$('div');

// JavaScript
document.getElementsByTagName('div');
{% endhighlight %}

#### querySelector/querySelectorAll
This is where things heat up - enter querySelector. Had it not been for jQuery, querySelector may not have made its way into the JavaScript language as quickly or as efficiently as it has - so we have jQuery to thank for this.

The magic behind querySelector is astounding, it's a multi-purpose native tool that you can use in various instances (this is raw JavaScript). There are two types of querySelector, the first which is plain old _document.querySelector('')_ returns the first Node in the NodeList, regardless of how many Node Objects it might find. The second, ultimately the best and most powerful is _document.querySelectorAll('')_ which returns a NodeList every time. I've been using _document.querySelectorAll('')_ as standard as it's easier to grab the first item in the returned NodeList than it is to reverse engineer _document.querySelector('')_.

Let's look at some examples, read the comments for better clarification:

{% highlight javascript %}
/*
 * Classes
 */
// Grab the first .myClass class name
document.querySelector('.myClass');

// Return a NodeList of all instances of .myClass
document.querySelectorAll('.myClass');

/*
 * ID
 */
// Grab the myID id
document.querySelector('#myID');

/*
 * Tags
 */
// Return a NodeList of all 'div' instances
document.querySelectorAll('div');
{% endhighlight %}

querySelectorAll is powerful, and definitely the future. It also supports more complicated selectors like so:

{% highlight javascript %}
// Grab the last list Node of .someList unordered list
document.querySelector('ul.someList li:last-child');

// Grab some data-* attribute
document.querySelectorAll('[data-toggle]');
{% endhighlight %}

You can also create a smart wrapper function for this, to save typing out _document.querySelectorAll('')_ each time:

{% highlight javascript %}
var _ = function ( elem ) {
  return document.querySelectorAll( elem );
}
// Usage
var myClass = _('.myClass');
{% endhighlight %}

You could use a _$_ symbol instead of an underscore, totes up to you. It's not ideal to begin a function expression with an underscore, but for demonstration purposes I have.

IE8 supports querySelector CSS2 selectors, I'm not sure why you'd want to perform DOM operations with CSS3 selectors entirely as CSS3 is used for progressive enhancement, whereas functionality can be broken whereas styling isn't nearly as important. If you're doing it right, you're using efficient class names and minimal selectors.

### Class manipulation
You can extend JavaScript using a prototypal inheritance method, which is what jQuery is doing behind the scenes. HTML5 however is the future, it's growing and legacy browsers are quickly diminishing. It's time to begin using native JavaScript class methods, which again a new feature in HTML5 is the classList - let's do some jQuery comparisons:

#### Add class
Adding a class is easy in jQuery, it does it all for you, taking care of the NodeList array too, we'll come onto this soon.
{% highlight javascript %}
// jQuery
$('div').addClass('myClass');

// JavaScript
var div = document.querySelector('div');
div.classList.add('myClass');
{% endhighlight %}

#### Remove class
Same as the above, super simple:
{% highlight javascript %}
// jQuery
$('div').removeClass('myClass');

// JavaScript
var div = document.querySelector('div');
div.classList.remove('myClass');
{% endhighlight %}

#### Toggle class
Toggle was a really important to the language, often tricky to replicate via _prototype_ methods. Thankfully it's here:

{% highlight javascript %}
// jQuery
$('div').toggleClass('myClass');

// JavaScript
var div = document.querySelector('div');
div.classList.toggle('myClass');
{% endhighlight %}

### Arrays
Now we'll push into the more advanced aspects of the JavaScript language, _Arrays_. Arrays are used to hold values inside one variable, which looks like so:
{% highlight javascript %}
var myArray = ['one', 'two', 'three', 'four']
{% endhighlight %}

jQuery makes this super easy with the _$.each();_ method, which again hides some of the dirty work and makes things easy. JavaScript began with no 'built-in' functionality for iterating over arrays, so we are used to manually working out the items in the array using the _length_ property and iterating over each item incrementally inside a _for_ loop:

{% highlight javascript %}
var myArray = ['one', 'two', 'three', 'four']
for (var i = 0; i < myArray.length; i++) {
  // ...
}
{% endhighlight %}

Recently, we received an upgrade from this rather manual method to the dedicated _forEach_ method, which is however slower than the above, but does provide callback functionality similar to jQuery's _$.each();_:

{% highlight javascript %}
// Bolt the array at the beginning, I like this
['one', 'two', 'three', 'four'].forEach(function(){
  // ...
});

// Or go oldschool with a variable declaration
var myArray = ['one', 'two', 'three', 'four'];
myArray.forEach(function(){
  // ...
});
{% endhighlight %}

Looking at the jQuery side of things, here's a quick comparison of the two:

{% highlight javascript %}
// jQuery
var myArray = ['one', 'two', 'three', 'four']
$.each( myArray, function ( index, value ) {
    console.log(value);
});

// JavaScript
var myArray = ['one', 'two', 'three', 'four']
for ( var i = 0; i < myArray.length; i++ ) {
    var value = myArray[i];
    console.log( value );
}
{% endhighlight %}

### NodeList looping
A large difference between jQuery is the fact we need to generate a loop using _getElementsByClassName_ or _querySelectorAll_. For instance, in jQuery, whether one class, or a _NodeList_ exists, the code is identical! This isn't the same with native JavaScript. For instance to add a class in both (notice the difference in the last two native JavaScript methods):

{% highlight javascript %}
// jQuery
var someElem = $('.someElem');
someElem.addClass('myClass');

// JavaScript - this adds the class to the first Node only!
var someElem = document.querySelector('.someElem');
someElem.classList.add('myClass');

// JavaScript - this adds the class to every Node in the NodeList
var someElem = document.querySelectorAll('.someElem');
for (var i = 0; i < someElem.length; i++) {
  someElem[i].classList.add('myClass');
}
{% endhighlight %}

So what's the difference here? We get a NodeList returned and therefore need to iterate over the NodeList and apply a new class to each. Pretty simple and makes sense. This is the kind of advanced things jQuery takes care of for us. The thing with JavaScript is that it is pretty scary to get started on, but once you're started it's addictive and it's imperative to know what's going on under the hood, as the saying goes.

### Attributes, setting, getting and removing
JavaScript offers better descriptive, if a little lengthier in character count, methods to dealing with attributes, let's look at the differences.

#### Set attributes
In jQuery, the naming convention isn't as good as native, as the _attr();_ can callback the value as well as set the value, in a way it's clever, but to those learning it could cause confusion. Let's look at how we can set attributes in both:
{% highlight javascript %}
// jQuery
$('.myClass').attr('disabled', true);

// JavaScript
document.querySelector('.myClass').setAttribute('disabled', true);
{% endhighlight %}

#### Remove attributes
Removing attributes is just as easy:
{% highlight javascript %}
// jQuery
$('.myClass').removeAttr('disabled');

// JavaScript
document.querySelector('.myClass').removeAttribute('disabled');
{% endhighlight %}

#### Get attributes
This is how we would log the attribute's vale in the Console:
{% highlight javascript %}
// jQuery
console.log($('.myClass').attr('title'));

// JavaScript
console.log(document.querySelector('.myClass').getAttribute('title'));
{% endhighlight %}

#### Data-* attributes
HTML5 data-* attributes are probably one of the best additions to the HTML specification ever, IMO of course. I use the jQuery _.data();_ API all the time, and also the native JavaScript if it's required:

{% highlight html %}
<div class="myElem" data-username="Todd"></div>

<script>
// jQuery
console.log($('.myElem').data('username')); // Logs 'Todd'

// JavaScript - use the getAttribute method, fairly static
console.log(document.querySelector('.myElem').getAttribute('data-username'));
</script>
{% endhighlight %}

HTML5 introduces the _dataset_ API, which browser support isn't bad, I don't think IE9/10 even support it. For heavy _.data();_ usage, I recommend jQuery as it works in all browsers - even legacy.

### Parsing JSON
There are neat tricks we can do to parse JSON and create objects too even in plain ol' JavaScript. It's pretty much the same! Let's take an HTML5 custom data-* attribute for a JSON example, grab the attribute, parse the JSON into an Object and then hook into that object:

{% highlight html %}
<div class="myElem" data-user='{ "name" : "Todd", "id" : "01282183" }'></div>

<script>
// jQuery
var myElem = $('.myElem').data('user'); // Gets the JSON
var myJSON = $.parseJSON(myElem); // Parses string into JSON Object
console.log(myJSON.name); // JSON Object, logs 'Todd'
console.log(myJSON.id); // JSON Object, logs '01282183'

// JavaScript
var myElem = document.querySelector('.myElem').getAttribute('data-user');
var myJSON = JSON.parse(myElem);
console.log(myJSON.name); // JSON Object, logs 'Todd'
console.log(myJSON.id); // JSON Object, logs '01282183'
</script>
{% endhighlight %}

### Events
Events play a massive part in JavaScript, and has had a bad reputation in the past with cross-browser issues. A simple click event in jQuery:

{% highlight javascript %}
$(elem).click(function () {
  // ...
});
{% endhighlight %}

I actually recommend going with jQuery's _.on();_ method should you want to use the click handler:

{% highlight javascript %}
$(elem).on('click', function () {
  // ...
});
{% endhighlight %}

Two reasons, you can chain the 'on' part like so:

{% highlight javascript %}
$(elem).on('click focus keyup', function () {
  // ...
});
{% endhighlight %}

This chains (well, binds) a couple of event handlers to register your function with. Any of them will run it. Not to mention you can easily swap them in and out.

Secondly, event delegation with dynamically created JavaScript elements:

{% highlight javascript %}
$(parent).on('click', elem, function () {
  // ...
});
{% endhighlight %}

This _captures_ the DOM event via a parent event listener. Look up event _bubbling_ and _capturing_ for homework if you're unsure of the difference.

Back to jQuery versus JavaScript now anyway, here's some event handlers:

{% highlight javascript %}
/*
 * Click
 */
// jQuery
$(elem).on('click', function () {...});

// JavaScript
document.querySelector(elem).onclick = function () {...}

/*
 * Submit
 */
// jQuery
$(elem).on('submit', function () {...});

// JavaScript
document.querySelector(elem).onsubmit = function () {...}

/*
 * Change
 */
// jQuery
$(elem).on('change', function () {...});

// JavaScript
document.querySelector(elem).onchange = function () {...}
{% endhighlight %}

You see my point...

There is one issue with JavaScript event handlers however, and you can blame Microsoft for this one (again), with their _attachEvent_ handler. Little did they decide to go down their own non-standard route and integrate _attachEvent_ when every other browser was using _addEventListener_. Still, there is a nice workaround script, provided by John Resig himself which solves this problem for us. addEventListener is very similar to jQuery's chaining of event handler methods, you can attach more than a single handler for each event - it also assists in event bubbling/catching.

{% highlight javascript %}
document.addEventListener('click', function() {
    // ...
}, false);
{% endhighlight %}

### CSS manipulation
CSS is admittedly nicer in the jQuery object methods, but check out native JavaScript's implementation of this, it's very similar and worth knowing:

{% highlight javascript %}
// jQuery
$(elem).css({
  "background" : "#F60",
  "color" : "#FFF"
});

// JavaScript
var elem = document.querySelector(elem);
elem.style.background = '#F60';
elem.style.color = '#FFF';
{% endhighlight %}

The above hooks into JavaScript's _style_ object and allows you to set lots of styles with ease.

### Document Ready Function
jQuery comes built-in with a DOM ready function handler, in which we can safely execute all of our functions knowing the DOM tree is fully populated and any manipulation we do will work and not return _undefined_ (undefined usually means it doesn't exist, or in this case it would).

As we progress towards a future of amazing technology, browsers now fire their own DOM ready function handler, in modern browsers this is called the _DOMContentLoaded_ event and can be fired like so:

{% highlight javascript %}
document.addEventListener('DOMContentLoaded', function() {
    // DOM ready, run it!
}, false);
{% endhighlight %}

jQuery has had a tendency to be called _the_ solution and there's no other alternative ever, ever ever. It's bad for upcoming developers to rely on it and it's imperative to learn, or at least have some understanding of, native JavaScript. The more powerful HTML5 becomes, the more we can utilise these rapid HTML5 native capabilities. And the more powerful the features become, the less we need jQuery, the more useless it becomes!

Embrace new technologies now, I'm not suggesting throw away your jQuery workflow and start going native instantly, but a native future is coming - are you ready!
