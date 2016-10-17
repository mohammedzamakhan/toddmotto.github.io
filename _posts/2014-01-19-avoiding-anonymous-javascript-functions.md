---
layout: post
permalink: /avoiding-anonymous-javascript-functions/
title: Avoiding anonymous JavaScript functions
path: 2014-01-19-avoiding-anonymous-javascript-functions.md
tags:
- Scope
- Functions
- JavaScript
---

Anonymous functions, the art of the callback. I'm going to propose that you never write a callback again using an anonymous function, and I'll sell you the idea now. Firstly, what is an anonymous function? Something like this:

{% highlight javascript %}
document.querySelector('.menu').addEventListener('click', function (event) {
  // we're inside the anon callback, btw...
  if (!this.classList.contains('active')) {
    this.classList.add('active');
  }
  event.preventDefault();
}, false);
{% endhighlight %}

Here's a few reasons why you should stop doing this... anonymous functions:

* Are more difficult to debug
* Cannot be reused
* Cannot be tested easily
* Do not describe the role of the function
* Make code lack structure
* Create messier/unclear code
* Documentation will suffer (things like jsDoc)

Let's investigate. Based on our above code example, I can see a `click` event was bound and it executes a function which adds a `class`. But what for? So far (apart from an educated guess), I can only assume that it toggles a tab or a menu. So why are we so reliant on using anonymous functions instead of helping ourselves write better code?

_"But what does this code do?"_. At this point, you remove your headphones, peer over to your colleague who wrote the code and ask him what the hell it adds a class to. He then gets agitated because you've stopped his code flow and he's paused his Beyonce remix only to tell you the answer. This could have all been avoided if he'd written some more classy code:

{% highlight javascript %}
function toggleMenu (event) {
  if (!this.classList.contains('active')) {
    this.classList.add('active');
  }
  event.preventDefault();
}
document.querySelector('.menu').addEventListener('click', toggleMenu, false);
{% endhighlight %}

Now, doesn't that look better? And hey, if we introduce another element, we can bind the same function again without causing grief:

{% highlight javascript %}
document.querySelector('.menu').addEventListener('click', toggleMenu, false);
document.querySelector('.myclass2').addEventListener('click', toggleMenu, false);
{% endhighlight %}

This also prevents those lazier developers to copy the entire contents of the anonymous functions and pasting it again, only to avoid moving it into a function and refactoring it for reuse.

_Abstraction._

A beautiful word. Let's use it more and abstract our code into more reusable components and parts, to make our lives much easier. How about at this stage we also abstract our selector?

{% highlight javascript %}
var menu = document.querySelector('.menu');
function toggleMenu (event) {
  if (!this.classList.contains('active')) {
    this.classList.add('active');
  }
  event.preventDefault();
}
menu.addEventListener('click', toggleMenu, false); // oozing with awesomeness
{% endhighlight %}

I really encourage this setup, because we're abstracting into _three_ different sections, the `selector`, `event` and `method`. I say death to the one-liner of jQuery-chaining rubbish that's littering the web - just because you can doesn't mean you should. Chaining methods creates more complex and often lesser quality code. Chaining sidesteps a problem of abstracting your methods into reusable parts and littering functions with them.

So let's revisit our above code, and highlight the `selector`, `event` and `method`:

{% highlight javascript %}
// selector
var menu = document.querySelector('.menu');

// method
function toggleMenu (event) {
  if (!this.classList.contains('active')) {
    this.classList.add('active');
  }
  event.preventDefault();
}

// event
menu.addEventListener('click', toggleMenu, false);
{% endhighlight %}

This opens up many benefits. Let's say that `menu` also took an `onchange` event, we could simply extend what we've written _so_ easily:

{% highlight javascript %}
// selector
var menu = document.querySelector('.menu');

// method
function toggleMenu (event) {
  if (!this.classList.contains('active')) {
    this.classList.add('active');
  }
  event.preventDefault();
}

// events
menu.addEventListener('click', toggleMenu, false);
menu.addEventListener('onchange', toggleMenu, false);
{% endhighlight %}

Based on this setup, you've probably guessed how I (on a very basic level) structure my JavaScript files that manipulate the DOM. Here's what a typical file might look like (with production ready in mind):

{% highlight javascript %}
// keep things outside the global scope plz
(function (window, document, undefined) {

  'use strict';

  /**
   * Selectors
   */
  var menu = document.querySelector('.menu');
  var users = document.querySelectorAll('.user');
  var signout = document.querySelector('.signout');

  /**
   * Methods
   */
  function toggleMenu (event) {
    if (!this.classList.contains('active')) {
      this.classList.add('active');
    }
    event.preventDefault();
  }
  function showUsers (users) {
    for (var i = 0; i < users.length; i++) {
      var self = users[i];
      self.classList.add('visible');
    }
  }
  function signout (users) {
    var xhr = new XMLHttpRequest();
    // TODO: finish signout
  }

  /**
   * Events/APIs/init
   */
  menu.addEventListener('click', toggleMenu, false);
  signout.addEventListener('click', signout, false);
  showUsers(users);


})(window, document);
{% endhighlight %}

This also has many other benefits, including caching your selectors, your team knowing the exact format in which you're writing your code, and not littering the file with random scripts here, there, everywhere, and making future changes incredibly easy.

You'll also notice I wrap all my code inside an IIFE, `(function () {...})();`, this keeps all your code outside of the global scope and helps reduce [more headaches](/everything-you-wanted-to-know-about-javascript-scope).

### Passing parameters

You may have noticed that I haven't passed in any parameters to any of the above code examples, this is because the way `addEventListener` was added to JavaScript was _nearly_ done well, but missed a vital piece of functionality, so we need to look closer and understand what's happening. You might think you can do this:

{% highlight javascript %}
element.addEventListener('click', toggleMenu(param1, param2), false);
{% endhighlight %}

...But this will invoke the function as soon as the JavaScript engine hits the function, which is bad news. So what we can do is use the `ECMAScript 5` addition `Function.prototype.bind` (modern browsers only) which sets up the values without invoking the function. This is similar to `.call()` and `.apply()` but doesn't invoke the function:

{% highlight javascript %}
element.addEventListener('click', toggleMenu.bind(null, param1, param2), false);
{% endhighlight %}

You can read more about `.bind()` [here](/everything-you-wanted-to-know-about-javascript-scope) and [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind). You can grab the `.bind()` [polyfill here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) so that all browsers can use `.bind()` (as it's current IE9+ and all modern browsers).

If you don't want to polyfill and go "oldschool" then you'll need to wrap it inside a function:

{% highlight javascript %}
element.addEventListener('click', function () {
  toggleMenu(param1, param2);
}, false);
{% endhighlight %}

_Doesn't this go against the article?_ No. This is a workaround for passing arguments into functions and has nothing to do with the benefits listed in the intro paragraph. You could even add your `event.preventDefault()` logic inside the wrapper callback depending on what the function inside did to ensure your function doesn't `preventDefault()` when you don't need it to.
