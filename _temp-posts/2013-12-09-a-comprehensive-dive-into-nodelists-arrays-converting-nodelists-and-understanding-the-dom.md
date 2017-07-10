---
layout: post
permalink: /a-comprehensive-dive-into-nodelists-arrays-converting-nodelists-and-understanding-the-dom/
title: A comprehensive dive into NodeLists, Arrays, converting NodeLists and understanding the DOM
path: 2013-12-09-a-comprehensive-dive-into-nodelists-arrays-converting-nodelists-and-understanding-the-dom.md
tags:
- NodeLists
- Arrays
- Objects
- JavaScript
---

Manipulating the DOM is JavaScript's role when developing websites and applications, and we do this by grabbing collections of elements called NodeLists. NodeLists are captured using a selector of some kind (jQuery or native JS), but do you really understand NodeLists and their differences from an Array collection of DOM nodes? This post is here to clear a few things up and hopefully answer some questions.

If you're a jQuery user, you're probably used to doing this:

{% highlight javascript %}
var divs = $('div');
{% endhighlight %}

This then introduces the [black box](http://en.wikipedia.org/wiki/Black_box) scenario for many new JavaScript developers, jQuery "just works". If you're one of those people, you're probably going to tread on a nail one day and realise you'd wished you learned how the DOM really works with JavaScript, so here's a quick lesson for you if you're in that boat.

For JavaScript developers (yay), there are a few ways to do the above as we dig a little deeper into the DOM's core:

{% highlight javascript %}
var divs = document.getElementsByTagName('div');
{% endhighlight %}

or...

{% highlight javascript %}
var divs = document.querySelectorAll('div');
{% endhighlight %}

All of these (apart from jQuery) return a _NodeList_. Any JavaScript/jQuery developer will have played around with the old _document.getElementsByTagName()_ method, but do they know it returns a _NodeList_ rather than an _Array_? And what difference/importance does this really play?

A lot. If you've never heard of NodeLists, or haven't learned about them but are using jQuery on a daily basis, then you need to learn what you're really dealing with underneath for many reasons.

Understanding the DOM and JavaScript will help you write much better JavaScript.

### What is a NodeList?
NodeLists are _very_ similar to Array collections of elements, often referred to as "array-like", but with a subtle difference - you're missing out on a lot of JavaScript functionality by keeping your collection as a NodeList, such as true Array iteration and Prototypal methods.

#### Array iteration
What's iteration? This means looping over your collection of elements, which you can then do something with each individual element's value or index. Looping over a NodeList is the exact same as an Array when using a regular _for_ loop:

{% highlight javascript %}
var divs = document.querySelectorAll('div');
for (var i = 0; i < divs.length; i++) {
  // access to individual element:
  var elem = divs[i];
}
{% endhighlight %}

But when we introduce the modern JavaScript _forEach()_ method, problems arise with the native API itself, the _forEach()_ method is to be used when iterating over Arrays (BTW, you can use _forEach()_ for Arrays in older browsers with a Polyfill, see end of article):

{% highlight javascript %}
var myArray = [1,2,3,4,5];
myArray.forEach(function (item) {
  // access to individual element
  var elem = item;
});
{% endhighlight %}

So that should work great when it comes to a NodeList, they're pretty similar. Take the following example:

{% highlight javascript %}
// NodeList collection
var divs = document.querySelectorAll('div');

// let's casually loop over the NodeList
divs.forEach(function () {
  
});
{% endhighlight %}

BAM!

{% highlight javascript %}
Uncaught TypeError: Object #<NodeList> has no method 'forEach'
{% endhighlight %}

"_What happened? Why is my code broken? Waaaahhh?_" the recent jQuery convert says.

You can't manipulate NodeLists the same way you can an Array.

#### Prototypal methods
Arrays come with a bunch of awesomely inherited prototypal methods, things like _splice()_, _push()_, _join()_, _indexOf()_ and many more. When our collections are NodeLists, we miss out on all this goodness. Check out [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array) for a comprehensive list of methods.

Which means we cannot remove an item from a NodeList like you'd simply expect:

{% highlight javascript %}
var divs = document.querySelectorAll('div');
for (var i = 0; i < divs.length; i++) {
    var self = divs[i];
    divs.splice(i, 1); // Remove this element from the NodeList
}
{% endhighlight %}

Uh oh...

{% highlight javascript %}
Uncaught TypeError: Object #<NodeList> has no method 'splice'
{% endhighlight %}

### What _isn't_ a NodeList?
A NodeList isn't an Array (applause).

NodeLists are actually really interesting collections of Nodes, and are separate to their close cousin Arrays for a few good reasons, they can contain what we call _live_ Nodes.

If I had the following HTML (3 divs):

{% highlight html %}
<div></div>
<div></div>
<div></div>
{% endhighlight %}

And ran a _document.getElementsByTagName()_ method, this will return a live collection:

{% highlight javascript %}
var nodes = document.getElementsByTagName('div');

// outputs 3
console.log(nodes);
{% endhighlight %}

If I were to do the following and insert a new _div_ element into the page:

{% highlight javascript %}
var nodes = document.getElementsByTagName('div');

// outputs 3
console.log(nodes);

// create a new element
var newDiv = document.createElement('div');
document.body.appendChild(newDiv);

// outputs 4
console.log(nodes);
{% endhighlight %}

As if by magic, our _nodes_ collection has automatically updated. I'm sure you can see the use of that, so you might not always want to convert a NodeList to Array.

### Converting NodeLists to Arrays
The plan of attack here really does vary depending entirely on your browser support and use case for that particular NodeList/Array.

#### Browser Support
If you need IE8 and below support, the easiest way of converting a NodeList to an Array is pushing each element from a NodeList into a new Array:

{% highlight javascript %}
var myNodeList = document.querySelectorAll('div');
var myArray = []; // empty Array
for (var i = 0; i < myNodeList.length; i++) {
    var self = myNodeList[i];
    myArray.push(self);
}
{% endhighlight %}

And you're all done. It's a nice and simple process. I absolutely love this method, as it still keeps your original NodeList reference if you need it, for instance keeping a tab on your live NodeList collection. Please note though, using _document.querySelectorAll()_ returns a _static_ NodeList, not _live_, therefore it won't automatically update. However, _document.getElementsByTagName()_ will keep a live record, but getting elements by their tag name is slowly dying. I personally would've liked to have seen live Nodes in _querySelectorAll_.

Moving swiftly forward, you'd be interested (maybe) to know that some performance/speed tests were done and the quickest method (apparently) of converting a NodeList to Array is:

{% highlight javascript %}
var arr = [];
var divs = document.querySelectorAll('div');
for(var i = divs.length; i--; arr.unshift(divs[i]));
{% endhighlight %}

Check out some of the other [NodeList to Array perf tests](http://jsperf.com/nodelist-to-array/27).

If you're fortunate enough to not care about IE8 and below, then you can use a neat trick to convert your NodeList instantly using _Array.prototype.slice.call()_:

{% highlight javascript %}
// 'divs' is now an Array
var divs = Array.prototype.slice.call(document.querySelectorAll('div'));
{% endhighlight %}

Accessing the Prototype Object here, we grab the _slice()_ method, and pass our NodeList into it. This API then internally converts it to an Array using the _slice()_ method (which returns a new Array). It cleverly pushes each Node into a new Array, yay!

Now we can access all the Array methods and use the _forEach()_ method as intended:

{% highlight javascript %}
var divs = Array.prototype.slice.call(document.querySelectorAll('div'));
divs.forEach(function () {
  //...
});
{% endhighlight %}

And no more TypeErrors, everything is good.

We can _shorten_ this entire declaration however using an empty Array, which has access to the Prototype methods:

{% highlight javascript %}
var divs = [].slice.call(document.querySelectorAll('div'));
{% endhighlight %}

... But I wouldn't advise it, this can cause issues with other libraries, even though it's sexier and shorter, use the long version and you'll be writing more bulletproof code.

#### ECMAScript 6 Array.from()
The new ECMAScript 6 Harmony standard introduces the `Array.from` method which makes Array-like Objects (such as the NodeList) and other iterable Objects (such as an `Object` or `String`) to Array conversion a breeze.

{% highlight javascript %}
var divs = document.querySelectorAll('div');
var arr = Array.from(divs); // Array of <div>s
{% endhighlight %}

More on the [Array.from](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/from) method.

#### Looping through NodeLists on-the-fly

For some time I thought it was pretty cool to do this, which takes the Prototypal methods one step further:

{% highlight javascript %}
var divs = document.querySelectorAll('div');
Array.prototype.forEach.call(divs, function (item) {
  // Individual access to element:
  var elem = item;
});
{% endhighlight %}

Using the _forEach()_ method and using call, again, this iterates over the NodeList is an Array fashion, almost converting it on the fly but never changing the original reference.

As above, we can use the shorthand empty array reference like so, but we've established that's not a good idea:

{% highlight javascript %}
var divs = document.querySelectorAll('div');
[].forEach.call(divs, function (item) {
  // Individual access to element:
  var elem = item;
});
{% endhighlight %}

### Polyfill(s)

As promised, polyfill(s) for you to drop in:

#### _array.forEach()_, [reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)

{% highlight javascript %}
if (!Array.prototype.forEach) {
  Array.prototype.forEach = function (fn, scope) {
    var i, len;
    for (i = 0, len = this.length; i < len; ++i) {
      if (i in this) {
        fn.call(scope, this[i], i, this);
      }
    }
  };
}
{% endhighlight %}

Dropping in the above will run a quick feature detect on the _forEach_ method and patch the browser functionality for you, which means you can do this and it'll work in every browser:

{% highlight javascript %}
var myArray = [1,2,3,4,5];
myArray.forEach(function () {
  //...
});
{% endhighlight %}

Hooray for ECMAScript 5!

### Summing up

I particularly don't like iterating over the NodeList on the fly, my advice would be to always convert your NodeLists and then you'll never have any issues at a later date or with other parts of your scripts. Again, the method that you choose to manipulate over your NodeLists is project and script dependent, so learn what each method does and make your decision wisely :)
