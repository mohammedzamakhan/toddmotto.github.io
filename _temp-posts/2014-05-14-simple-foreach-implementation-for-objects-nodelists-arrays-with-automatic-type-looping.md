---
layout: post
permalink: /simple-foreach-implementation-for-objects-nodelists-arrays-with-automatic-type-looping/
title: Simple forEach implementation for Objects/NodeLists/Arrays
path: 2014-05-14-simple-foreach-implementation-for-objects-nodelists-arrays-with-automatic-type-looping.md
tags:
- Loops
- NodeList
- Objects
- Arrays
- JavaScript
---

Looping Objects is easy. Looping Arrays is also easy. Looping NodeLists is easy. They can be a little repetitive though and often take time to construct each loop and pass in the index, property, element or whatever...

There is no "standard" way of iterating over everything. We can use `Array.prototype.forEach` to loop over Arrays (or the regular `for` loop), a `for in` loop for Objects, and a regular `for` loop again for NodeLists or HTML collections. No, you're not going to use that `forEach.call(NodeList)` [hack](//toddmotto.com/ditch-the-array-foreach-call-nodelist-hack).

Wouldn't it be nice to just forget about what type of collection we're looping, forget about browser support and write a nice little function that handles everything for us. Yes.

So I did...

<div class="download-box">
  <a href="//github.com/toddmotto/foreach/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download foreach', 'Download foreach']);">Download</a>
  <a href="//github.com/toddmotto/foreach" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork foreach', 'foreach Fork']);">Fork on GitHub</a>
</div>

### forEach.js

`forEach.js` is a simple script, it's not part of a library or even a module, it's just a function, here's its syntax and a quick example using an `Array`:

{% highlight javascript %}
// syntax
forEach(collection[, callback[, context]]);

// example
var myArray = ['A', 'B', 'C', 'D'];
forEach(myArray, function (value, index) {
	// `this` will reference myArray: []
}, myArray); // note third param changing execution context
{% endhighlight %}

#### forEach() for Arrays/NodeLists
You can loop over an Array or NodeList using a standard `for` loop, however, NodeLists cannot be used in conjunction with the newer ECMAScript 5 `Array.prototype.forEach`. This script takes care of that in the same way it loops over an `Array`, you'll get the same stuff passed back:

{% highlight javascript %}
// Array:
forEach(['A', 'B', 'C', 'D'], function (value, index) {
	console.log(index); // 0, 1, 2, 3
	console.log(value); // A, B, C, D
});
// NodeList:
forEach(document.querySelectorAll('div'), function (value, index) {
	console.log(index); // 0, 1, 2, 3
	console.log(value); // <div>, <div>, <div>...
});
{% endhighlight %}

#### forEach() for Objects
Object iteration is usually done via a `for in` loop, we can wrap this up by passing back values which makes our loops cleaner and easier to manage:

{% highlight javascript %}
// Object:
forEach({ name: 'Todd', location: 'UK' }, function (value, prop, obj) {
	console.log(value); // Todd, UK
	console.log(prop); // name, location
  console.log(obj); // { name: 'Todd', location: 'UK' }, { name: 'Todd', location: 'UK' }
});
{% endhighlight %}

##### collection
Type: `Array|Object|NodeList`

Collection of items to iterate, could be an `Array`, `Object` or `NodeList`.

##### callback
Type: `Function`

Callback function for each iteration.

##### context
Type: `Array|Object|NodeList` Default: `null`

Object/NodeList/Array that `forEach` is iterating over, to use as the `this` value when executing callback.

### Code
For those interested, check out the code below, the latest version is available [on GitHub](//github.com/toddmotto/foreach).


{% highlight javascript %}
var forEach = function (collection, callback, scope) {
  if (Object.prototype.toString.call(collection) === '[object Object]') {
    for (var prop in collection) {
      if (Object.prototype.hasOwnProperty.call(collection, prop)) {
        callback.call(scope, collection[prop], prop, collection);
      }
    }
  } else {
    for (var i = 0; i < collection.length; i++) {
      callback.call(scope, collection[i], i, collection);
    }
  }
};
{% endhighlight %}

<div class="download-box">
  <a href="//github.com/toddmotto/foreach/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download foreach', 'Download foreach']);">Download</a>
  <a href="//github.com/toddmotto/foreach" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork foreach', 'foreach Fork']);">Fork on GitHub</a>
</div>
