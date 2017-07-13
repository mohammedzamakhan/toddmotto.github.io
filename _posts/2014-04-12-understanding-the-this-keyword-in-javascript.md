---
layout: post
permalink: /understanding-the-this-keyword-in-javascript/
title: Understanding the “this” keyword in JavaScript
path: 2014-04-12-understanding-the-this-keyword-in-javascript.md
tag: js
---

It's probably safe to say that the `this` keyword is one of the most misunderstood parts of JavaScript. Admittedly, I used to throw the `this` keyword around until my script worked and it confused the hell out of me (and still confuses many other JS developers). It wasn't until I learned about lexical scope, how functions are invoked, scope context, and a few context changing methods that I really understood it.

<div class="toc" markdown="1">
<span class="gamma">Table of contents</span>
{:.no_toc}
* TOC
{:toc}
</div>

Before you dive into this article, here's a few very important points to takeaway and remember about the `this` keyword:

- The `this` keyword's value has nothing to do with the function itself, how the function is called determines the `this` value
- It can be dynamic, based on how the function is called
- You can change the `this` context through `.call()`, `.apply()` and `.bind()`

### Default `this` context

There are a few different ways the `this` value changes, and as we know it's usually the call-site that creates the context.

#### Window Object, global scope

Let's take a quick example at how simply calling regular functions binds the `this` value differently:

{% highlight javascript %}
// define a function
var myFunction = function () {
  console.log(this);
};

// call it
myFunction();
{% endhighlight %}

What can we expect the `this` value to be? By default, this should always be the `window` Object, which refers to the root - the global scope. So when we `console.log(this);` from our function, as it's invoked by the window (simply just called), we should expect the `this` value to be our `window` Object:

{% highlight javascript %}
// define a function
var myFunction = function () {
  console.log(this); // [object Window]
};

// call it
myFunction();
{% endhighlight %}

#### Object literals

Inside Object literals, the `this` value will always refer to its own Object. Nice and simple to remember. That is good news when invoking our functions, and one of the reasons I adopt patterns such as the module pattern for organising my objects.

Here's how that might look:

{% highlight javascript %}
// create an object
var myObject = {};

// create a method on our object
myObject.someMethod = function () {
  console.log(this);
};

// call our method
myObject.someMethod();
{% endhighlight %}

Here, our `window` Object didn't invoke the function - our Object did, so `this` will refer to the Object that called it:

{% highlight javascript %}
// create an object
var myObject = {};

// create a method on our object
myObject.someMethod = function () {
  console.log(this); // myObject
};

// call our method
myObject.someMethod();
{% endhighlight %}

#### Prototypes and Constructors

The same applies with Constructors:

{% highlight javascript %}
var myConstructor = function () {
    this.someMethod = function () {
        console.log(this);
    };
};

var a = new myConstructor();
a.someMethod();
{% endhighlight %}

And we can add a Prototype Object as well:

{% highlight javascript %}
var myConstructor = function () {
    this.someMethod = function () {
        console.log(this);
    };
};

myConstructor.prototype = {
    somePrototypeMethod: function () {
        console.log(this);
    }
};

var a = new myConstructor();
a.someMethod();
a.somePrototypeMethod();
{% endhighlight %}

Interestingly, in both cases the `this` value will refer to the Constructor object, which will be `myConstructor`.

#### Events

When we bind events, the same rule applies, the `this` value points to the owner. The owner in the following example would be the element.

{% highlight javascript %}
// let's assume .elem is <div class="elem"></div>
var element = document.querySelector('.elem');
var someMethod = function () {
  console.log(this);
};
element.addEventListener('click', someMethod, false);
{% endhighlight %}

Here, `this` would refer to `<div class="elem"></div>`.

### Dynamic `this`

The second point I made in the intro paragraph was that `this` is dynamic, which means the value could change. Here's a real simple example to show that:

{% highlight javascript %}
// let's assume .elem is <div class="elem"></div>
var element = document.querySelector('.elem');

// our function
var someMethod = function () {
  console.log(this);
};

// when clicked, `this` will become the element
element.addEventListener('click', someMethod, false); // <div>

// if we just invoke the function, `this` becomes the window object
someMethod(); // [object Window]
{% endhighlight %}

### Changing `this` context

There are often many reasons why we need to change the context of a function, and thankfully we have a few methods at our disposal, these being `.call()`, `.apply()` and `.bind()`.

Using any of the above will allow you to change the context of a function, which in effect will change the `this` value. You'll use this when you want `this` to refer to something different than the scope it's in.

#### Using `.call()`, `.apply()` and `.bind()`

You will often hear that "Functions are first class Objects," which means that they can also have their own methods!

The `.call()` method allows you to change the scope with a specific syntax [ref](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call):

{% highlight javascript %}
.call(thisArg[, arg1[, arg2[, ...]]]);
{% endhighlight %}

Usage would look something like this:

{% highlight javascript %}
someMethod.call(anotherScope, arg1, arg1);
{% endhighlight %}

You'll notice further arguments are all comma separated - this is the only difference between `.call()` and `.apply()`:

{% highlight javascript %}
someMethod.call(anotherScope, arg1, arg1); // commas
someMethod.apply(anotherScope, [arg1, arg1]); // array
{% endhighlight %}

With any of the above, they immediately invoke the function. Here's an example:

{% highlight javascript %}
var myFunction = function () {
  console.log(this);
};
myFunction.call();
{% endhighlight %}

Without any arguments, the function is just invoked and `this` will remain as the `window` Object.

Here's a more practical usage, this script will always refer to the `window` Object:

{% highlight javascript %}
var numbers = [{
  name: 'Mark'
},{
  name: 'Tom'
},{
  name: 'Travis'
}];
for (var i = 0; i < numbers.length; i++) {
  console.log(this); // window
}
{% endhighlight %}

The `forEach` method also has the same effect, it's a function so it creates new scope:

{% highlight javascript %}
var numbers = [{
  name: 'Mark'
},{
  name: 'Tom'
},{
  name: 'Travis'
}];
numbers.forEach(function () {
  console.log(this); // window
});
{% endhighlight %}

We could change each iteration's scope to the current element's value inside a regular `for` loop as well, and use `this` to access object properties:

{% highlight javascript %}
var numbers = [{
  name: 'Mark'
},{
  name: 'Tom'
},{
  name: 'Travis'
}];
for (var i = 0; i < numbers.length; i++) {
  (function () {
    console.log(this.name); // Mark, Tom, Travis
  }).call(numbers[i]);
}
{% endhighlight %}

This is especially extensible when passing around other Objects that you might want to run through the exact same functions.

##### forEach scoping

Not many developers using `forEach` know that you can change the initial scope context via the second argument:

{% highlight javascript %}
numbers.forEach(function () {
  console.log(this); // this = Array [{ name: 'Mark' },{ name: 'Tom' },{ name: 'Travis' }]
}, numbers); // BOOM, scope change!
{% endhighlight %}

Of course the above example doesn't change the scope to how we want it, as it changes the functions scope for every iteration, not each individual one - though it has use cases for sure!

To get the _ideal_ setup, we need:

{% highlight javascript %}
var numbers = [{
  name: 'Mark'
},{
  name: 'Tom'
},{
  name: 'Travis'
}];
numbers.forEach(function (item) {
  (function () {
    console.log(this.name); // Mark, Tom, Travis
  }).call(item);
});
{% endhighlight %}

#### `.bind()`

Using `.bind()` is an ECMAScript 5 addition to JavaScript, which means it's not supported in all browsers (but can be polyfilled so you're all good if you need it). Bind has the same effect as `.call()`, but instead binds the function's context _prior_ to being invoked, this is essential to understand the difference. Using `.bind()` _will not_ invoke the function, it just "sets it up".

Here's a really quick example of how you'd setup the context for a function, I've used `.bind()` to change the context of the function, which by default the `this` value would be the window Object.

{% highlight javascript %}
var obj = {};
var someMethod = function () {
  console.log(this); // this = obj
}.bind(obj);
someMethod();
{% endhighlight %}

This is a really simple use case, they can also be used in event handlers as well to pass in some extra information without a needless anonymous function:

{% highlight javascript %}
var obj = {};
var element = document.querySelector('.elem');
var someMethod = function () {
  console.log(this);
};
element.addEventListener('click', someMethod.bind(obj), false); // bind
{% endhighlight %}

### "Jumping scope"

I call this jumping scope, but essentially it's just some slang for accessing a lexical scope reference (also a bit easier to remember).

There are many times when we need to access lexical scope. Lexical scope is where variables and functions are still accessible to us in parent scopes.

{% highlight javascript %}
var obj = {};

obj.myMethod = function () {
  console.log(this); // this = `obj`
};

obj.myMethod();
{% endhighlight %}

In the above scenario, `this` binds perfectly, but what happens when we introduce another function. How many times have you encountered a scope challenge when using a function such as `setTimeout` inside another function? It totally screws up any `this` reference:

{% highlight javascript %}
var obj = {};
obj.myMethod = function () {
  console.log(this); // this = obj
    setTimeout(function () {
        console.log(this); // window object :O!!!
    }, 100);
};
obj.myMethod();
{% endhighlight %}

So what happened there? As we know, [functions create scope](/everything-you-wanted-to-know-about-javascript-scope), and `setTimeout` will be invoked by itself, defaulting to the `window` Object, and thus making the `this` value a bit strange inside that function.

##### Important note: `this` and the `arguments` Object are the only objects that _don't_ follow the rules of lexical scope.

How can we fix it? There are a few options! If we're using `.bind()`, it's an easy fix, note the usage on the end of the function:

{% highlight javascript %}
var obj = {};
obj.myMethod = function () {
  console.log(this); // this = obj
    setTimeout(function () {
        console.log(this); // this = obj
    }.bind(this), 100); // .bind() #ftw
};
obj.myMethod();
{% endhighlight %}

We can also use the jumping scope trick, `var that = this;`:

{% highlight javascript %}
var obj = {};
obj.myMethod = function () {
  var that = this;
  console.log(this); // this = obj
    setTimeout(function () {
        console.log(that); // that (this) = obj
    }, 100);
};
obj.myMethod();
{% endhighlight %}

We've cut the `this` short and just simply pushed a reference of the scope into the new scope. It's kind of cheating, but works wonders for "jumping scope". With newcomers such as `.bind()`, this technique is sometimes frowned upon if used and abused.

One thing I dislike about `.bind()` is that you could end up with something like this:

{% highlight javascript %}
var obj = {};
obj.myMethod = function () {
  console.log(this);
    setTimeout(function () {
        console.log(this);
        setTimeout(function () {
            console.log(this);
            setTimeout(function () {
                console.log(this);
                setTimeout(function () {
                    console.log(this);
                }.bind(this), 100); // bind
            }.bind(this), 100); // bind
        }.bind(this), 100); // bind
    }.bind(this), 100); // bind
};
obj.myMethod();
{% endhighlight %}

A tonne of `.bind()` calls, which look totally stupid. Of course this is an exaggerated issue, but it can happen very easily when switching scopes. In my opinion this would be easier - it will also be tonnes quicker as we're saving lots of function calls:

{% highlight javascript %}
var obj = {};
obj.myMethod = function () {
  var that = this; // one declaration of that = this, no fn calls
  console.log(this);
    setTimeout(function () {
        console.log(that);
        setTimeout(function () {
            console.log(that);
            setTimeout(function () {
                console.log(that);
                setTimeout(function () {
                    console.log(that);
                }, 100);
            }, 100);
        }, 100);
    }, 100);
};
obj.myMethod();
{% endhighlight %}

Do what makes sense!

#### jQuery $(this)

Yes, the same applies, don't use `$(this)` unless you actually know what it's doing. What it _is_ doing is passing the normal `this` value into a new jQuery Object, which will then inherit all of jQuery's prototypal methods (such as addClass), so you can instantly do this:

{% highlight javascript %}
$('.elem').on('click', function () {
  $(this).addClass('active');
});
{% endhighlight %}

Happy scoping ;)
