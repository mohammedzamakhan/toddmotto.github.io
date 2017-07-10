---
layout: post
permalink: /es6-arrow-functions-syntaxes-and-lexical-scoping/
title: ES6 arrow functions, syntax and lexical scoping
path: 2015-04-14-es6-arrow-functions-syntaxes-and-lexical-scoping.md
tags:
- Arrow functions
- ES6
- JavaScript
---

ES2015 (ES6) introduces a really nice feature that punches above its weight in terms of simplicity to integrate versus time saving and feature output. This feature is the arrow function.

Before we dive into the features of the arrow function and what it actually does for us, let's understand what an arrow function is _not_. It's not a replacement for the `function` keyword, at all. This means you can't do a find and replace on every single `function` keyword and everything works perfectly, because it likely won't.

If you're competent with the way [JavaScript scope](//toddmotto.com/everything-you-wanted-to-know-about-javascript-scope) works, and have a great understanding of lexical scope, the `this` keyword and Prototype methods such as `.call()`, `.apply()` and `.bind()`, then you're in good hands to continue reading.

### Syntax
Let's look at what the arrow function's construct is from [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions):

{% highlight javascript %}
// example 1
([param] [, param]) => {
  statements
}

// example 2
param => expression
{% endhighlight %}

The "normal JavaScript" (ES5) equivalents to help transition:

{% highlight javascript %}
// example 1
function ([param] [, param]) {
  statements
}

// example 2
function (param) {
  return expression
}
{% endhighlight %}

The ES6 and ES5 differences in `example 1` are that the `function` keyword is omitted, and `=>` now exists _after_ the arguments. In `example 2`, our function has been reduced to one line, this is great for single line function expressions that get `return`'d.

#### Hint: arrows are anonymous
Arrow functions are always anonymous, which means we can't do this with ES6:

{% highlight javascript %}
// ES5
function doSomething() {
  //...
}
{% endhighlight %}

Instead of this, we could assign our anonymous arrow function it to a variable (using `var` here instead of `let` as ES6 block scoping is another topic):

{% highlight javascript %}
// ES6
var doSomething = () => {
  //...
}
{% endhighlight %}

Let's look at the syntaxes a little further and then the functionality differences when using arrow functions.

### Syntax: single line expressions
We touched briefly above on single line expressions, let's look at a great use case for them.

Let's take some junky ES5 example that iterates over an Array using `Array.prototype.map`:

{% highlight javascript %}
var numbers = [1,2,3,4,5];
var timesTwo = numbers.map(function (number) {
  return number * 2;
});
console.log(timesTwo); // [2, 4, 6, 8, 10]
{% endhighlight %}

We can reduce this down to a single line with an arrow function, which saves us a lot of typing and can actually enhance readability in my opinion as this piece of code has one clear role:

{% highlight javascript %}
var numbers = [1,2,3,4,5];
var timesTwo = numbers.map((number) => number * 2);
console.log(timesTwo); // [2, 4, 6, 8, 10]
{% endhighlight %}

### Syntax: single argument functions
Arrow functions also give us a small "sugar" syntax that allows us to remove parenthesis when only using a single argument in a function.

Taking the last piece of code for example we had this:

{% highlight javascript %}
numbers.map((number) => number * 2);
{% endhighlight %}

When we could remove the parens from `(number)` to leave us with this:

{% highlight javascript %}
numbers.map(number => number * 2);
{% endhighlight %}

This is great and a little clearer initially, but as we all know applications grow and code scales, and to save us headaches (be it forgetting syntaxes or lesser experienced developers "not knowing" to add parens back with more than one argument), I'd recommend always using the parens out of habit, even for single args:

{% highlight javascript %}
// we still rock with ES6
numbers.map((number) => number * 2);
{% endhighlight %}

### Functionality: lexical scoping "this"
Now we're past the sugar syntax excitement, we can dig into the benefits of the arrow function and its implications on execution context.

Typically if we're writing ES5, we'll use something like `Function.prototype.bind` to grab the `this` value from another scope to change a function's execution context. This will mainly be used in callbacks inside a different scope.

In Angular, I adopt the `controllerAs` syntax which allows me to use `this` inside the Controller to refer to itself (so here's an example). Inside a function the `this` value may change, so I could have a few options, use `that = this` or `.bind`:

{% highlight javascript %}
function FooCtrl (FooService) {
  this.foo = 'Hello';
  FooService
  .doSomething(function (response) {
    this.foo = response;
  });
}
{% endhighlight %}

The `this.foo = response;` won't work correctly as it's been executed in a different context. To change this we could use `.bind(this)` to give our desired effect:

{% highlight javascript %}
function FooCtrl (FooService) {
  this.foo = 'Hello';
  FooService
  .doSomething(function (response) {
    this.foo = response;
  }.bind(this));
}
{% endhighlight %}

Or you may be used to keeping a top level `this` reference, which can make more sense when dealing with many nested contexts, we don't want a gross tree of `.bind(this), .bind(this), .bind(this)` and a tonne of wasted time binding those new functions (`.bind` is very slow). So we could look at `that = this` to save the day:

{% highlight javascript %}
function FooCtrl (FooService) {
  var that = this;
  that.foo = 'Hello';
  FooService
  .doSomething(function (response) {
    that.foo = response;
  });
}
{% endhighlight %}

With arrow functions, we have a better option, which allows us to "inherit" the scope we're in if needed. Which means if we changed our initial example to the following, the `this` value would be bound correctly:

{% highlight javascript %}
function FooCtrl (FooService) {
  this.foo = 'Hello';
  FooService
  .doSomething((response) => { // woo, pretty
    this.foo = response;
  });
}
{% endhighlight %}

We could then refactor some more into a nice single line expression, push to git and head home for the day:

{% highlight javascript %}
function FooCtrl (FooService) {
  this.foo = 'Hello';
  FooService
  .doSomething((response) => this.foo = response);
}
{% endhighlight %}

The interesting thing to note is that the `this` value (internally) is not _actually_ bound to the arrow function. Normal functions in JavaScript bind their own `this` value, however the `this` value used in arrow functions is actually fetched lexically from the scope it sits inside. It has no `this`, so when you use `this` you're talking to the outer scope.
