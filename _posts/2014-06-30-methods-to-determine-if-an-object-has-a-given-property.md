---
layout: post
permalink: /methods-to-determine-if-an-object-has-a-given-property/
title: Methods to determine if an Object has a given property
path: 2014-06-30-methods-to-determine-if-an-object-has-a-given-property.md
tags:
- Performance
- JavaScript
---

There are multiple ways to detect whether an Object has a property. You'd think it'd be as easy as `myObject.hasOwnProperty('prop');` - but no, there are a few different ways with their own problems and gotchas. Let's look at the few ways to check property existence, concepts that confuse JavaScript developers, prototype chain lookups and problems JavaScript might provide us.

### Double bang !! property lookup
We've all seen it, probably in something such as Modernizr for simple feature detection, the infamous `!!` amongst our JS. Important note before we begin this one, it doesn't actually check if an Object has a property "as such", it checks the _value_ of the Object property. Which means if the property value is false, or the object property doesn't even exist, they give the same `falsy` result - which can be really bad if you use it without knowing what it does and its limitations.

#### What does it mean?

The double-bang is a simple way to typecast something to `Boolean`. The `Boolean` will cast `true` for _truthy_ values. Even things such as `undefined` and `null` (both falsy values, `!!null` is `false` when cast to `Boolean`). The _absolute key_ here is that it casts _values_. I'll say it again, _values_! This is irrelevant to the shape and size of your Object. We convert truthy and falsy values to Boolean.

#### Examples

An empty `Array` is an example of a _truthy_ value:
{% highlight javascript %}
var a = []; // []
{% endhighlight %}

What if we want to convert it to a `Boolean` though? It's truthy, so we should expect `true`:
{% highlight javascript %}
var a = !![]; // true
{% endhighlight %}

`null` is an example of a _falsy_ value:
{% highlight javascript %}
var a = null; // null
{% endhighlight %}

And the expected output of `false`:
{% highlight javascript %}
var a = !!null; // false
{% endhighlight %}

This means that we can use it when looking up our Objects!

{% highlight javascript %}
var toddObject = {
  name: 'Todd',
  cool: false
};
!!toddObject.name // true (correct result as it's a truthy value)
{% endhighlight %}

This method also looks up the Object's `prototype` chain to see if the property exists, which can cause unintended side effects if naming of properties is the same as a prototypes.

{% highlight javascript %}
// Object.prototype.toString
!!toddObject.toString // true

// !!Array.prototype.forEach
!![]['forEach'] // true
{% endhighlight %}

#### Gotchas

Beware of using it for detecting your own Objects. We often create Objects and defaults such as this:

{% highlight javascript %}
var toddObject = {
  name: 'Todd',
  favouriteDrink: null
};
{% endhighlight %}

If we're using the double-bang to check if an Object property exists using this method, then it's definitely a silly idea:

{% highlight javascript %}
var toddObject = {
  name: 'Todd',
  favouriteDrink: null
};
if (!!toddObject.favouriteDrink) { // false
  // do something if it exists, I think...
}
{% endhighlight %}

That would be naive! The above code (to the new developer or non-double-banger) might say _"If toddObject.favouriteDrink exists, do something"_. But no, because (I'll say it again...) this casts _values_, the value is `null` and falsy - even though the property exists. It's generally not a good idea in this case to use it for checking if a property exists incase it has a falsy value to begin with.

### hasOwnProperty

We went as far as getting a native method for this, but it's not 100% reliable for a few reasons. Let's examine it first.

#### What does it mean?
Using `myObject.hasOwnProperty('prop')` is a great way of accessing the Object's keys directly, which _does not_ look into the Object's `prototype` - hooray, this is great for specific use cases. `hasOwnProperty` returns a Boolean for us on whether a property exists.

#### Examples

{% highlight javascript %}
var toddObject = {
  name: 'Todd',
  favouriteDrink: null
};
if (toddObject.hasOwnProperty('favouriteDrink')) { // true
  // do something if it exists, fo sho
}
{% endhighlight %}

But don't be sold on this exact implementation... read below for best practice.

#### Gotchas

IE messes up the `hasOwnProperty` method completely as it's painful with `host` Objects (host objects don't have the hasOwnProperty method).

JavaScript also decided not to protect the method's name, so we can infact do this:

{% highlight javascript %}
var toddObject = {
  hasOwnProperty: 'hello...'
};
{% endhighlight %}

This makes it hard to fully trust it. What we can do however is access the `Object.prototype` directly to guarantee any `hasOwnProperty` calls haven't been tampered with or overridden. 

Let's bulletproof the process:

{% highlight javascript %}
var toddObject = {
  name: 'Todd',
  favouriteDrink: null
};
if (Object.prototype.hasOwnProperty.call(toddObject, 'favouriteDrink')) { // true
  // do something if it exists, fo sho sho!
}
{% endhighlight %}

The secret sauce here is `.call()` to change the context of `hasOwnProperty` (take that, IE) and ensure we've the exact `hasOwnProperty` we want from the `Object.prototype`.

Obviously you'd want to wrap it inside a helper function or something to save writing out that `prototype` each time:

{% highlight javascript %}
function hasProp (obj, prop) {
  return Object.prototype.hasOwnProperty.call(obj, prop);
}
if (hasProp(toddObject, 'favouriteDrink')) {}
{% endhighlight %}

### 'prop' in myObject

The `in` operator isn't so widely used as the former methods, but is probably worth using after reading this. It also returns a `Boolean` much like `!!myObject`, but _does not_ evaluate the _value_, it evaluates the _existence_ of the property!. This means if a property has a value of `false`, we get a correct reading that the property does infact exist.

{% highlight javascript %}
var toddObject = {
  name: 'Todd',
  favouriteDrink: null,
  cool: false
};
'cool' in toddObject; // true
{% endhighlight %}

The `in` operator is probably your best friend for checking the existence of a property, it's also pretty concise.

#### Gotchas

The `in` operator also looks up the `prototype`, which _may_ cause unintended side effects:

{% highlight javascript %}
// inherits Object.prototype.toString
'toString' in toddObject; // true
{% endhighlight %}

But we should know these property names and not create conflicts, right ;)

### typeof

We can use `typeof` as well.

#### What does it mean?

The standard `typeof` operator returns a `String` ([not a very reliable one](http://toddmotto.com/understanding-javascript-types-and-reliable-type-checking)), and we can evaluate it against something, such as `!== 'undefined'` - which indicates it exists.

{% highlight javascript %}
if (typeof toddObject.name !== 'undefined') {
  // do something
}
{% endhighlight %}

It looks a little ugly, as well as being quite long to write out if we were to make multiple checks using this method. Also, `null` would fall under this check unless using `!= 'undefined'` (single `=`) as `null == undefined` anyway.

#### Gotchas

Only use it [if you know what you're doing](http://toddmotto.com/understanding-javascript-types-and-reliable-type-checking) as it's very unreliable for standard type checking.

### Feature detection

I can't recall exactly what was said, but someone (I think) once told me that some  vendor once implemented a feature with a falsy value if it didn't exist (though I'm not even certain that's true, worth a mention though)... and as such the `in` operator is best for these such cases:

{% highlight javascript %}
// just an example, not the one somebody mentioned...
if ('draggable' in document.createElement('div')) {
  // do something if prop exists
}
{% endhighlight %}
