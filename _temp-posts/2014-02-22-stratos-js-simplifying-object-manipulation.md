---
layout: post
permalink: /stratos-js-simplifying-object-manipulation/
title: Stratos.js simplifying Object manipulation
path: 2014-02-22-stratos-js-simplifying-object-manipulation.md
tags:
- Library
- Objects
- Loops
- Performance
- JavaScript
---

JavaScript Objects are usually the driving force behind applications I develop, specifically JSON which gets sent back and forth from the server as acts as the main method of comms.

To save time rewriting the same (or similar) logic over and over again when dealing with our data (typically as part of a Model/View) - wouldn't it be great to use _one_ module to encapsulate the trickier object manipulation stuff and make developing the core of the application easier? It would also be great to bulletproof the object manipulation process, reducing object tasks, limit debugging, promote code reuse and even save a tonne of KB! Yes. So I built Stratos.js, a standalone 1KB module! It also comes fully equipped with unit tests for each method.

Stratos acts as a factory and supports: AMD (require.js), browser globals and `module.exports` to run on Node/Browserify/CommonJS, so it can be used server-side too.

<div class="download-box">
  <a href="//github.com/toddmotto/stratos/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download stratos', 'Download stratos']);">Download</a>
  <a href="//github.com/toddmotto/stratos" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork stratos', 'stratos Fork']);">Fork on GitHub</a>
</div>

Stratos has a few helper utilities, as well as powerful and time/byte saving methods. The methods that Stratos currently ships with are:

* has()
* type()
* add()
* remove()
* extend()
* destroy()
* keys()
* vals()
* toJSON()
* fromJSON()

These methods take care of the heavy lifting that comes with Object manipulation, for instance to extend an Object, Stratos has a method that wraps it all up for you:

{% highlight javascript %}
// "exports" is merely the inner module namespace
// you will call Stratos.extend(); in this example
exports.extend = function (parent, child) {
  for (var key in child) {
    if (exports.has(child, key)) {
      parent[key] = child[key];
    }
  }
};
{% endhighlight %}

Another example of useful encapsulation; to delete object properties, Stratos makes the necessary safety checks as well:

{% highlight javascript %}
exports.remove = function (object, key) {
  if (exports.has(object, key)) {
    delete object[key];
  }
};
{% endhighlight %}

Stratos also has JSON support for stringifying and parsing Objects. Check out the rest of the methods and feel free to contribute.

Stratos runs in ECMAScript 5's `strict mode`, which I was [interested to find out](https://twitter.com/toddmotto/status/436967309773901824) that you can't delete Objects as a whole, so `Stratos.destroy(object)` prevents `Uncaught Errors` by emptying Objects rather than attempting to delete them entirely.

To ensure Stratos methods are called with the correct context, and do not conflict with other libraries or tools that override the `hasOwnProperty()` method (which isn't protected in JavaScript), Stratos uses `Object.prototype.hasOwnProperty.call(object, key)` to ensure the correct context and method reliability.

Read on for a more in depth look into Stratos.

### Methods/definitions

#### has()
Returns a boolean on whether an Object property exists.

{% highlight javascript %}
var obj = { name: 'Todd' };
Stratos.has(obj, 'name'); // true
{% endhighlight %}

#### type()
Returns the raw type of the Object, for example `[object Object]`.

{% highlight javascript %}
var obj = {};
var arr = [];
Stratos.type(obj); // [object Object]
Stratos.type(arr); // [object Array]
{% endhighlight %}

#### add()
Adds an Object property with corresponding value. Value can be any Object type (array/number/object).

{% highlight javascript %}
var obj = {};
Stratos.add(obj, 'name', 'Todd'); // { name: 'Todd' }
Stratos.add(obj, 'likes', ['Ellie Goulding', 'The Killers']); // { name: 'Todd', likes: ['Ellie Goulding', 'The Killers'] }
{% endhighlight %}

#### remove()
Removes an Object property.

{% highlight javascript %}
var obj = { name: 'Todd', location: 'UK' };
Stratos.remove(obj, 'name'); // { location: 'UK' }
{% endhighlight %}

#### extend()
Merges two objects for top level keys. Stratos doesn't offer a deep merge of Objects on a recursive basis.

{% highlight javascript %}
var parent = { prop1: 'hello', prop2: 'yes', prop3: 'sing' };
var child = { prop1: 'goodbye', prop2: 'no', prop4: 'song' };

// { prop1: 'goodbye', prop2: 'no', prop3: 'sing', prop4: 'song' }
Stratos.extend(parent, child);
{% endhighlight %}

#### destroy()
Destroys an Object by removing all properties inside it, leaving an empty Object. ECMAScript 5 `strict mode` doesn't allow for top level Object deletion, so we will just erase the contents.

{% highlight javascript %}
var obj = { name: 'Todd', location: 'UK' };
Stratos.destroy(obj); // {}
{% endhighlight %}

#### keys()
Traverses the Object and returns an array of the Object's own enumerable properties, in the same order as that provided by a `for in` loop.

{% highlight javascript %}
var obj = { name: 'Todd', location: 'UK' };
Stratos.keys(obj); // ['name', 'location']
{% endhighlight %}

#### vals()
Traverses the Object and returns an array of the Object's own enumerable properties, in the same order as that provided by a `for in` loop.

{% highlight javascript %}
var obj = { name: 'Todd', location: 'UK' };
Stratos.vals(obj); // ['Todd', 'UK']
{% endhighlight %}

#### toJSON()
Converts an Object to JSON.

{% highlight javascript %}
var obj = { name: 'Todd', location: 'UK' };
Stratos.toJSON(obj); // {"name":"Todd","location":"UK"}
{% endhighlight %}

#### fromJSON()
Parses JSON back to an Object.

{% highlight javascript %}
var obj = { name: 'Todd', location: 'UK' };
var json = Stratos.toJSON(obj); // {"name":"Todd","location":"UK"}
Stratos.fromJSON(json); // { name: 'Todd', location: 'UK' };
{% endhighlight %}

<div class="download-box">
  <a href="//github.com/toddmotto/stratos/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download stratos', 'Download stratos']);">Download</a>
  <a href="//github.com/toddmotto/stratos" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork stratos', 'stratos Fork']);">Fork on GitHub</a>
</div>
