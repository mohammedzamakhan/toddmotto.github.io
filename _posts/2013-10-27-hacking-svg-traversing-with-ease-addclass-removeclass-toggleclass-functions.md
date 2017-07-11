---
layout: post
permalink: /hacking-svg-traversing-with-ease-addclass-removeclass-toggleclass-functions/
title: Hacking SVG, traversing with ease - addClass, removeClass, toggleClass functions
path: 2013-10-27-hacking-svg-traversing-with-ease-addclass-removeclass-toggleclass-functions.md
tags:
- SVG
- Classnames
- JavaScript
---

### Update: I've turned this into a small JavaScript module named [Lunar](https://github.com/toddmotto/lunar) and pushed to GitHub, please use that code instead as it doesn't extend native DOM APIs and also has accompanying unit tests :)

----

I encountered how painful traversing inline SVG can be when working on a recent project, simple DOM APIs such as adding, removing and toggling classes just aren't there, or supported by tools such as jQuery (yes, I even tried jQuery).

Inline SVG is SVG in the DOM, rendered from its XML. Here's a quick look at example inline SVG which would sit anywhere in the DOM:

{% highlight html %}
<svg id="svg" xmlns="http://www.w3.org/2000/svg" version="1.1" height="190">
  <circle cx="100" cy="50" r="40" fill="red" />
</svg>
{% endhighlight %}

The `svg` element acts as a wrapper to the XML inside, whilst defining a few things such as height, width, namespace and version. You'll notice I've added an `id` attribute, with the value of `svg`. Current DOM APIs make this seamlessly easy to target:

{% highlight javascript %}
// grabs <svg>
var mySVG = document.querySelector('#svg');
{% endhighlight %}

### Problem: DOM stuff

But the problem begins when trying to do the 'usual' DOM stuff, adding a class, or removing a class. You'd think it would be pretty simple, but even using jQuery's APIs don't allow it to work either, so I wrote my own and I'm pretty pleased with its compactness. The trick is to _set_ the attribute again, you can't keep adding/removing classes using the _.className_ method. The _getAttribute_ method is what I've used to grab the class attribute's value, and then the idea behind it is to grab that attribute, manipulate it and then set it back again.

Let's say I have a function, I need to add a class to an SVG circle onclick:

{% highlight javascript %}
// grabs <circle>
var mySVG = document.querySelector('#svg circle');
mySVG.setAttribute('class', 'myClass');
{% endhighlight %}

... will give us:

{% highlight html %}
<svg id="svg" xmlns="http://www.w3.org/2000/svg" version="1.1" height="190">
  <circle cx="100" cy="50" r="40" fill="red" class="myClass" />
</svg>
{% endhighlight %}

I have to think as the 'class' attribute as totally made up and that _className_ doesn't exist. And manipulating this is where the magic happens.

### hasClass API
As with all the APIs, I'm hanging these off the SVGElement's prototype constructor so all SVG nodes inherit the methods. With _hasClass_, I'm extending the native Object with a function. This method allows simple declaration of the APIs. Inside the _hasClass_ function, I'm creating a new Regular Expression, which gets dynamically created through its _className_ parameter and immediately tested against its attribute value. JavaScript's _.test()_ returns a boolean (true/false), a simple way to test a class presence.

{% highlight javascript %}
SVGElement.prototype.hasClass = function (className) {
  return new RegExp('(\\s|^)' + className + '(\\s|$)').test(this.getAttribute('class'));
};
{% endhighlight %}

### addClass API
Adding a class is simple, just set the attribute. Here I simply make a check using the _hasClass_ API, and if it doesn't exist I add it. There's no point adding it again if it does exist. If it doesn't exist, I set the attribute _class_ with the current class value, plus my new class name, super simple.

{% highlight javascript %}
SVGElement.prototype.addClass = function (className) {
  if (!this.hasClass(className)) {
    this.setAttribute('class', this.getAttribute('class') + ' ' + className);
  }
};
{% endhighlight %}

### removeClass API
Removing a class was the most fun, there's also the issue of keeping spaces intact, for instances I had to work out how to remove a class and keep the appropriate spaces around that class name. You can see I create a new class here called _removedClass_, where I get the current value, then replace the passed in _className_ using a dynamically created RegExp again. This RegExp has some coolness added to it, you'll see I declare _'g'_ at the end of the RegExp declaration, this means global, and will replace all instances of the class, for example if it was declared more than once throughout the class value. I then make a safety check to ensure the class is there, and set the attribute back on the element.

You'll also see I used a second parameter as well in the _replace_ method, which says _'$2'_. This is a nifty little trick, which refers to the capture group in the RegExp. Capture groups are denoted by circular brackets, my example at the end of the RegExp says _'(\\s|$)'_, this denotes a capture group, and then looks for whitespace after the className, OR it's at the end of the string, which is what the _$_ means. I can then remove the className and leave whatever was in the capture group, either a space or nothing, which keeps the class value tidy.

{% highlight javascript %}
SVGElement.prototype.removeClass = function (className) {
  var removedClass = this.getAttribute('class').replace(new RegExp('(\\s|^)' + className + '(\\s|$)', 'g'), '$2');
  if (this.hasClass(className)) {
    this.setAttribute('class', removedClass);
  }
};
{% endhighlight %}

### toggleClass API
Toggling from hereon is super simple, I'll check if the element has the class, and based on that I'll either add or remove the class using the above APIs.

{% highlight javascript %}
SVGElement.prototype.toggleClass = function (className) {
  if (this.hasClass(className)) {
    this.removeClass(className);
  } else {
    this.addClass(className);
  }
};
{% endhighlight %}

### Usage
Usage is simple, and simple API style:

{% highlight javascript %}
// Grab my Node
var mySVG = document.querySelector('#svg circle');

// hasClass
mySVG.hasClass('zzz');

// addClass
mySVG.addClass('zzz');

// removeClass
mySVG.removeClass('zzz');

// toggleClass
mySVG.toggleClass('zzz');
{% endhighlight %}
