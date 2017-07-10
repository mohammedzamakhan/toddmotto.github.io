---
layout: post
permalink: /understanding-javascript-types-and-reliable-type-checking/
title: Understanding JavaScript types and reliable type checking
path: 2014-06-02-understanding-javascript-types-and-reliable-type-checking.md
tags:
- Scope
- JavaScript
- this
---

Type checking in JavaScript can often be a pain, especially for new JS developers. I want to show you how to reliably check types in JS and understand them a little more. This post digs through Objects, Primitives, shadow objects/coercion, the `typeof` operator and how we can reliably get a "real" JavaScript type.

### Objects versus Primitives

"Everything in JavaScript is an Object". Remember it, then forget it. It's not true. JavaScript makes the subject very difficult to understand though - it presents everything as some form of "object" if we dive into their Prototypes (later). For now, let's look at types.

To understand JavaScript types, we need a top level view of them:

* Number
* String
* Boolean
* Object
* Null
* Undefined

We have `Number`, `String`, `Boolean` - these are Primitives (not Objects!). This means their values are unable to be changed because they are merely _values_, they have no properties. The Primitive types are wrapped by their Object counterparts when called, JavaScript will dive between the Number/String/Boolean to an Object when needed (coercion). Underneath, it will infact construct an Object, use it, then return the result (all the instance will be shipped out for garbage collection).

For example using `'someString'.trim();` will spin up an Object underneath and call the `.trim()` method on it.

`Null` and `undefined` are weird (both Primitives too), and distinguish between _no_ value or an _unknown_ value (`null` is unknown value, `undefined` is _totally_ not known or even declared). There is also an [Error object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error).

Object's however are a different story. You'll notice I've not mentioned `Array` or `RegExp`, these are _types_ of Object, let's investigate. Under the `Object` tree we have:

* Object
  * Function
  * Array
  * Date
  * RegExp

Having broken it down, things seem a little simpler, we have Objects versus Primitives. That's it, right? No, JavaScript decided it wanted to complicate _everything_ you'd assume logical from above.

### Typeof operator

From [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof): _"The `typeof` operator returns a string indicating the type of the unevaluated operand"_.

Based on our newly acquired knowledge from the above, you wouldn't expect this to happen:

{% highlight javascript %}
typeof []; // object
typeof {}; // object
typeof ''; // string
typeof new Date() // object
typeof 1; // number
typeof function () {}; // function
typeof /test/i; // object
typeof true; // boolean
typeof null; // object
typeof undefined; // undefined
{% endhighlight %}

Whyyyyyy?! `Function` is an Object, but tells us it's a `function`, `Array` is an Object and says it is. `null` is an Object, and so is our `RegExp`. What happened?

The `typeof` operator is a bit strange. Unless you know how to _really_ use it, simply avoid it to avoid headaches. We wouldn't want something like this to happen:

{% highlight javascript %}
// EXPECTATION
var person = {
  getName: function () {
    return 'Todd';
  };
};
if (typeof person === 'object') {
  person.getName();
}

// THIS GETS LET THROUGH...
// because I stupidly refactored some code changing the names
// but the `if` still lets through `person`
var person = [];
var myPerson = {
  getName: function () {
    return 'Todd';
  }
};
if (typeof person === 'object') {
  person.getName(); // Uncaught TypeError: undefined is not a function 
}
{% endhighlight %}

`typeof` let us down here, what we really wanted to know was that `person` was a _plain_ Object.

### True Object types

There's a really simple way, though to look at it looks like a hack:

{% highlight javascript %}
Object.prototype.toString.call();
{% endhighlight %}

The `.toString()` method is accessed using `Object.prototype` because every object descending from `Object` prototypically inherits it. By default, we get `[object Object]` when calling `{}.toString()` (an `Object`).

We can use `.call()` to change the `this` context (as it converts its argument to a value of type) and, for example, if we use `.call(/test/i)` (a Regular Expression) then `[object Object]` becomes `[object RegExp]`.

Which means if we run our test again using all JS types:

{% highlight javascript %}
Object.prototype.toString.call([]); // [object Array]
Object.prototype.toString.call({}); // [object Object]
Object.prototype.toString.call(''); // [object String]
Object.prototype.toString.call(new Date()); // [object Date]
Object.prototype.toString.call(1); // [object Number]
Object.prototype.toString.call(function () {}); // [object Function]
Object.prototype.toString.call(/test/i); // [object RegExp]
Object.prototype.toString.call(true); // [object Boolean]
Object.prototype.toString.call(null); // [object Null]
Object.prototype.toString.call(); // [object Undefined]
{% endhighlight %}

We can then push this into a function and more reliably validate our previous function:

{% highlight javascript %}
var getType = function (elem) {
  return Object.prototype.toString.call(elem);
};
if (getType(person) === '[object Object]') {
  person.getName();
}
{% endhighlight %}

To keep things DRY and save writing `=== '[object Object]'` or whatever out each time, we can create methods to simply reference. I've used `.slice(8, -1);` inside the `getType` function to remove the unnecessary `[object ` and `]` parts of the String:

{% highlight javascript %}
var getType = function (elem) {
  return Object.prototype.toString.call(elem).slice(8, -1);
};
var isObject = function (elem) {
  return getType(elem) === 'Object';
};
if (isObject(person)) {
  person.getName();
}
{% endhighlight %}

Snazzy.

I put together all the above methods into a micro-library called [Axis.js](//github.com/toddmotto/axis) which you can use:

{% highlight javascript %}
axis.isArray([]); // true
axis.isObject({}); // true
axis.isString(''); // true
axis.isDate(new Date()); // true
axis.isRegExp(/test/i); // true
axis.isFunction(function () {}); // true
axis.isBoolean(true); // true
axis.isNumber(1); // true
axis.isNull(null); // true
axis.isUndefined(); // true
{% endhighlight %}

The code powering that does some cool stuff for those interested:

{% highlight javascript %}
/*! axis v1.1.0 | (c) 2014 @toddmotto | github.com/toddmotto/axis */
(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    define(factory);
  } else if (typeof exports === 'object') {
    module.exports = factory;
  } else {
    root.axis = factory();
  }
})(this, function () {

  'use strict';

  var exports = {};

  var types = 'Array Object String Date RegExp Function Boolean Number Null Undefined'.split(' ');

  var type = function () {
    return Object.prototype.toString.call(this).slice(8, -1);
  };

  for (var i = types.length; i--;) {
    exports['is' + types[i]] = (function (self) {
      return function (elem) {
        return type.call(elem) === self;
      };
    })(types[i]);
  }

  return exports;

});
{% endhighlight %}

<div class="download-box">
  <a href="//github.com/toddmotto/axis/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download axis', 'Download axis']);">Download</a>
  <a href="//github.com/toddmotto/axis" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork axis', 'axis Fork']);">Fork on GitHub</a>
</div>
