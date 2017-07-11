---
layout: post
permalink: /polyfills-suck-use-a-featurefill-instead/
title: Polyfills suck&#44; use a featurefill instead
path: 2014-12-1-polyfills-suck-use-a-featurefill-instead.md
tags:
- Polyfills
- JavaScript
---

I'm going to dub this a featurefill as the post title suggests, but it's more a feature-detect-closure-binding-smart-polyfill-api-checker-reusable-function-awesomeness.

So, what's the deal?... I'm a huge fan of polyfilling behaviour for older browsers that don't support specific APIs, such as `Function.prototype.bind` or `Array.prototype.forEach`. Typically, we'd drop these polyfills into our apps like so:

{% highlight javascript %}
// https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    var aArgs   = Array.prototype.slice.call(arguments, 1),
        fToBind = this,
        fNOP    = function() {},
        fBound  = function() {
          return fToBind.apply(this instanceof fNOP && oThis
                 ? this
                 : oThis,
                 aArgs.concat(Array.prototype.slice.call(arguments)));
        };

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();

    return fBound;
  };
}
{% endhighlight %}

Then we can get on with our development and start using `Function.prototype.bind` until our heart's content.

The thing I'm questioning though, is this a good way to do things? Our `if` statement checks for the existence of a method on the `prototype` chain, and if it's not there it patches it.

I'm thinking there may be better ways to do this. Instead of checking for something that doesn't exist, and hope there are no other polyfills that create strange behaviour from modifying our Objects, we could wrap the contents of our polyfill inside a clever closure, and return if the API _does_ exist, rather than if it doesn't.

This would require us to construct our own methods, however, but it tightly packs the functionality for solid code reuse in later projects.

A quick starting function to demonstrate the concept, we'll create an `isArray` method, instead of polyfilling the ES5 method:

{% highlight javascript %}
function isArray (collection) {
  
}
{% endhighlight %}

Let's check out the polyfill for the `isArray` method:

{% highlight javascript %}
if (!Array.isArray) {
  Array.isArray = function(arg) {
    return Object.prototype.toString.call(arg) === '[object Array]';
  };
}
{% endhighlight %}

Again, it detects for the presence of the method and creates it for us if it doesn't exist. Let's detect if the method _does_ exist, and get smarter with our polyfilling by using the native methods when available first.

{% highlight javascript %}
function isArray (collection) {
  if (Array.isArray) {
    return Array.isArray(collection);
  }
}
{% endhighlight %}

We don't need an `else` statement because we'll have already returned if the method exists, so we'll drop a modified version of the above `!Array.isArray` polyfill in:

{% highlight javascript %}
function isArray (collection) {
  if (Array.isArray) {
    return Array.isArray(collection);
  }
  return Object.prototype.toString.call(collection) === '[object Array]';
}
{% endhighlight %}

Done. Simple! This uses the native method when available, and gracefully falls back to a polyfill if it's not.

There is one slight problem with this approach, however, in that the `if` statement is checked every time the function is called. We'll use a closure to return only the things we need at runtime, to increase the performance of multiple checks.

First, we'll switch the `isArray` function for a variable:

{% highlight javascript %}
var isArray;
{% endhighlight %}

Then assign an IIFE:

{% highlight javascript %}
var isArray = (function () {
  
})();
{% endhighlight %}

This function executes immediately, so we can return a value to it so it's bound for the lifetime of the program. Should `Array.isArray` be natively available, let's give that back to the developer:

{% highlight javascript %}
var isArray = (function () {
  if (Array.isArray) {
    return Array.isArray;
  }
})();
{% endhighlight %}

Notice we don't now have to call `Array.isArray(collection);`, we just return the native API. The polyfill requires returning a function closure with our `collection` argument, which then returns what we need for the polyfill:

{% highlight javascript %}
var isArray = (function () {
  if (Array.isArray) {
    return Array.isArray;
  }
  return function (collection) {
    return Object.prototype.toString.call(collection) === '[object Array]';
  };
})();
{% endhighlight %}

If our browser supports the `Array.isArray` method then we'll actually get this (if we logged it out in the console):

{% highlight javascript %}
function isArray() { [native code] } 
{% endhighlight %}

If our browser doesn't support it, we get the patch:

{% highlight javascript %}
function (collection) {
  return Object.prototype.toString.call(collection) === '[object Array]';
} 
{% endhighlight %}

This is great, as it means that we're actually getting a single return value bound to the variable, so there's no `if` checking for API presence and we should get some performance gains from this.

If you're running test suites on your functions, baking your own library, then using this method I'd highly recommend over dropping in random polyfills. It provides expected behaviour that doesn't modify the existence of APIs that might or might not be there.
