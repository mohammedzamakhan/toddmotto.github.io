---
layout: post
permalink: /what-function-window-document-undefined-iife-really-means/
title: What (function (window, document, undefined) {})(window, document); really means
path: 2014-06-03-what-function-window-document-undefined-iife-really-means.md
tags:
- Scope
- this
- Performance
- JavaScript
---

Interestingly enough I get asked about the IIFE (immediately-invoked function expression) a lot, which takes the following setup:

{% highlight javascript %}
(function (window, document, undefined) {
  // 
})(window, document);
{% endhighlight %}

So why not write a post about it? ;-)

First, this does a series of different things. From the top:

### Scope

JavaScript has `function` scope, so first this creates some much needed "private scope". For example:

{% highlight javascript %}
(function (window, document, undefined) {
  var name = 'Todd';
})(window, document);

console.log(name); // name is not defined, it's in a different scope
{% endhighlight %}

Simple.

### How it works

A normal function looks like this:

{% highlight javascript %}
var logMyName = function (name) {
  console.log(name);
};

logMyName('Todd');
{% endhighlight %}

We get to _invoke_ it by choice, and wherever we want/can scope providing.

The reason "IIFE" was coined was because they're immediately-invoked function expressions. Which means they're immediately called at runtime - also we can't call them again they run just once like this:

{% highlight javascript %}
var logMyName = (function (name) {
  console.log(name); // Todd
})('Todd');
{% endhighlight %}

The secret sauce here is this, (which I've assigned to a variable in the previous example):

{% highlight javascript %}
(function () {
  
})();
{% endhighlight %}

The extra pair of parentheses _is_ necessary as this doesn't work:

{% highlight javascript %}
function () {
  
}();
{% endhighlight %}

Though several tricks can be done to trick JavaScript into "making it work". These force the JavaScript parser to treat the code following the `!` character as an expression:

{% highlight javascript %}
!function () {
  
}();
{% endhighlight %}

There are also other variants:

{% highlight javascript %}
+function () {
  
}();
-function () {
  
}();
~function () {
  
}();
{% endhighlight %}

But I wouldn't use them.

Check out [Disassembling JavaScript's IIFE Syntax](https://blog.mariusschulz.com/2016/01/13/disassembling-javascripts-iife-syntax) by [@mariusschulz](https://twitter.com/mariusschulz) for a detailed explanation of the IIFE syntax and its variants.

### Arguments

Now we know how it works, we can pass in arguments to our IIFE:

{% highlight javascript %}
(function (window) {
  
})(window);
{% endhighlight %}

How does this work? Remember, the closing `(window);` is where the function is invoked, and we're passing in the `window` Object. This then gets passed into the function, which I've named `window` also. You could argue this is pointless as we should name it something different - but for now we'll use `window` as well.

So what else can we do? Pass in all the things! Let's pass in the `document` Object:

{% highlight javascript %}
(function (window, document) {
  // we refer to window and document normally
})(window, document);
{% endhighlight %}

Local variables are faster to resolve than the global variables, but this is on a huge scale and you'll never notice the speed increase - but also worth considering if we're referencing our globals a lot!

### What about `undefined`?

In ECMAScript 3, `undefined` is mutable. Which means its value could be reassigned, something like `undefined = true;` for instance, oh my! Thankfully in ECMAScript 5 strict mode (`'use strict';`) the parser will throw an error telling you you're an idiot. Before this, we started protecting our IIFE's by doing this:

{% highlight javascript %}
(function (window, document, undefined) {

})(window, document);
{% endhighlight %}

Which means if someone came along and did this, we'd be okay:

{% highlight javascript %}
undefined = true;
(function (window, document, undefined) {
  // undefined is a local undefined variable
})(window, document);
{% endhighlight %}

### Minifying

Minifying your local variables is where the IIFE pattern's awesomeness really kicks in. Local variable names aren't really needed if they're passed in, so we can call them what we like.

Changing this:

{% highlight javascript %}
(function (window, document, undefined) {
  console.log(window); // Object window
})(window, document);
{% endhighlight %}

To this:

{% highlight javascript %}
(function (a, b, c) {
  console.log(a); // Object window
})(window, document);
{% endhighlight %}

Imagine it, all your references to libraries and `window` and `document` nicely minified. Of course you don't need to stop there, we can pass in jQuery too or whatever is available in the lexical scope:

{% highlight javascript %}
(function ($, window, document, undefined) {
  // use $ to refer to jQuery
  // $(document).addClass('test');
})(jQuery, window, document);

(function (a, b, c, d) {
  // becomes
  // a(c).addClass('test');
})(jQuery, window, document);
{% endhighlight %}

This also means you don't need to call `jQuery.noConflict();` or anything as `$` is assigned locally to the module. Learning how scopes and global/local variables work will help you even further.

A good minifier will make sure to rename `undefined` to `c` (for example, and only if used) throughout your script too. Important to note, _the name undefined is irrelevant_. We just need to know that the referencing Object is undefined, as `undefined` has no special meaning - `undefined` is the value javascript gives to things that are declared but have no value.

### Non-browser global environments

Due to things such as Node.js, the browser isn't always the global Object which can be a pain if you're trying to create IIFE's that work across multiple environments. For this reason, I tend to stick with this as a base:

{% highlight javascript %}
(function (root) {

})(this);
{% endhighlight %}

In a browser, the global environment `this` refers to the `window` Object, so we don't need to pass in `window` at all, we could always shorten it to `this`.

I prefer the name `root` as it can refer to non-browser environments as well as the root of the browser.

If you're interested in a universal solution (which I use all the time nowadays when creating open source project modules) is the UMD wrapper:

{% highlight javascript %}
(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    define(factory);
  } else if (typeof exports === 'object') {
    module.exports = factory;
  } else {
    root.MYMODULE = factory();
  }
})(this, function () {
  // 
});
{% endhighlight %}

This is some sexy stuff. The function is being invoked with another function passed into it. We can then assign it to the relevant environment inside. In the browser, `root.MYMODULE = factory();` is our IIFE module, elsewhere (such as Node.js) it'll use `module.exports` or requireJS if `typeof define === 'function' && define.amd` resolves true.

But this stuff is another story, but I insist you check out the [UMD repo](https://github.com/umdjs/umd).
