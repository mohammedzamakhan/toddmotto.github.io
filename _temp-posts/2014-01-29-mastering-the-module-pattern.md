---
layout: post
permalink: /mastering-the-module-pattern/
title: Mastering the Module Pattern
path: 2014-01-29-mastering-the-module-pattern.md
tags:
- Modules
- MVVM
- JavaScript
---

I'm a massive fan of JavaScript's Module Pattern and I'd like to share some use cases and differences in the pattern, and why they're important. The Module Pattern is what we'd call a "[design pattern](http://code.tutsplus.com/tutorials/understanding-design-patterns-in-javascript--net-25930),"and it's extremely useful for a vast amount of reasons. My main attraction to the Module Pattern (and its variant, the Revealing Module Pattern) is it makes scoping a breeze and doesn't overcomplicate program design. 

It also keeps things very simple and easy to read and use, uses Objects in a very nice way, and doesn't bloat your code with repetitive `this` and `prototype` declarations. I thought I'd share some insight as to the awesome parts of the Module, and how you can master it, its variants and features.

### Creating a Module
To understand what a Module can give you, you'll need to understand what the following `function` concept does:

{% highlight javascript %}
(function () {
  // code
})();
{% endhighlight %}

It declares a function, which then calls itself immediately. These are also known as [Immediately-Invoked-Function-Expressions](http://benalman.com/news/2010/11/immediately-invoked-function-expression), in which the `function` creates new scope and creates "privacy". JavaScript doesn't have privacy, but creating new scope emulates this when we wrap all our function logic inside them. The idea then is to return only the parts we need, leaving the other code out of the `global` scope.

After creating new `scope`, we need to namespace our code so that we can access any methods we return. Let's create a namespace for our anonymous Module.

{% highlight javascript %}
var Module = (function () {
  // code
})();
{% endhighlight %}

We then have `Module` declared in the global scope, which means we can call it wherever we like, and even pass it into another Module.

### Private methods
You'll see and hear a lot about `private` methods in JavaScript. But Javascript doesn't _strictly_ have `private` methods, but we _can_ create a working equivalent.

What _are_ private methods you might be asking? Private methods are anything you don't want users/devs/hackers to be able to see/call outside the scope they're in. We might be making server calls and posting sensitive data, we _don't_ want to expose those functions publicly, they could post anything back then and take advantage of our code. So we can create closure and be more sensible (as best as we can with JavaScript) at protecting our code. It's not _all_ about protection however, there are also naming conflicts. I bet when you first started out writing jQuery/JavaScript, that you dumped all your code in one file and it was just `function, function, function`. Little did you know these were all global, and you probably suffered the consequence at some point. If so, you'll learn why, and what to do to change it.

So let's use our newly created `Module` scope to make our methods inaccessible outside of that scope. For beginners to the Module Pattern, this example will help understand how a private method would be defined:

{% highlight javascript %}
var Module = (function () {
  
  var privateMethod = function () {
    // do something
  };

})();
{% endhighlight %}

The above example declares our function `privateMethod`, which is locally declared inside the new scope. If we were to attempt calling it anywhere outside of our module, we'll get an error thrown and our JavaScript program will break! We don't want anyone to be able to call our methods, especially ones that might manipulate data and go back and forth to a server.

### Understanding "return"
Typical Modules will use `return` and return an `Object` to the Module, to which the methods bound to the `Object` will be accessible from the Module's namespace.

A real light example of returning an `Object` with a `function` as a property:

{% highlight javascript %}
var Module = (function () {
  
  return {
    publicMethod: function () {
      // code
    }
  };

})();
{% endhighlight %}

As we're returning an `Object Literal`, we can call them exactly like Object Literals:

{% highlight javascript %}
Module.publicMethod();
{% endhighlight %}

For those who haven't used the Object Literal syntax before, a standard Object Literal could look something like this:

{% highlight javascript %}
var myObjLiteral = {
  defaults: { name: 'Todd' },
  someMethod: function () {
    console.log(this.defaults);
  }
};

// console.log: Object { name: 'Todd' }
myObjLiteral.someMethod();
{% endhighlight %}

But the issue with Object Literals is the pattern can be abused. Methods _intended_ to be "private" will be accessible by users because they are part of the Object. This is where the Module comes in to save us, by allowing us to define all our "private" stuff locally and only return "the good parts".

Let's look at a more Object Literal syntax, and a perfectly good Module Pattern and the `return` keyword's role. Usually a Module will return an Object, but how that Object is defined and constructed is totally up to you. Depending on the project and the role/setup of the code, I may use one of a few syntaxes.

#### Anonymous Object Literal return
One of the easiest patterns is the same as we've declared above, the Object has no name declared locally, we just return an Object and that's it:

{% highlight javascript %}
var Module = (function () {

  var privateMethod = function () {};
  
  return {
    publicMethodOne: function () {
      // I can call `privateMethod()` you know...
    },
    publicMethodTwo: function () {

    },
    publicMethodThree: function () {

    }
  };

})();
{% endhighlight %}

### Locally scoped Object Literal
Local scope means a variable/function declared inside a scope. On the [Conditionizr](//conditionizr.com) project, we use a locally scoped namespace as the file is over 100 lines, so it's good to be able to see what are the public and private methods without checking the `return` statement. In this sense, it's _much_ easier to see what _is_ public, because they'll have a locally scoped namespace attached:

{% highlight javascript %}
var Module = (function () {

  // locally scoped Object
  var myObject = {};

  // declared with `var`, must be "private"
  var privateMethod = function () {};

  myObject.someMethod = function () {
    // take it away Mr. Public Method
  };
  
  return myObject;

})();
{% endhighlight %}

You'll then see on the last line inside the Module that `myObject` is returned. Our global `Module` doesn't care that the locally scoped `Object` has a name, we'll only get the actual Object sent back, not the name. It offers for better code management.

### Stacked locally scoped Object Literal

This is pretty much identical as the previous example, but uses the "traditional" single Object Literal notation:

{% highlight javascript %}
var Module = (function () {

  var privateMethod = function () {};

  var myObject = {
    someMethod:  function () {

    },
    anotherMethod:  function () {
      
    }
  };
  
  return myObject;

})();
{% endhighlight %}

I prefer the second approach we looked at, the _Locally scoped Object Literal_. Because here, we have to declare _other_ functions before we use them (you should do this, using `function myFunction () {}` hoists your functions and can cause issues when used incorrectly). Using `var myFunction = function () {};` syntax lets us not worry about this, as we'll declare them all before we use them, this also makes debugging easier as the JavaScript interpreter will render our code in the order we declare, rather than hoisting `function` declarations. I also don't like this approach so much, because the "stacking" method can often get verbose looking, and there is no obvious locally scoped `Object namespace` for me to bolt public methods onto.

### Revealing Module Pattern
We've looked at the Module, and there's a really neat variant which is deemed the "revealing" pattern, in which we reveal public pointers to methods inside the Module's scope. This again, can create a really nice code management system in which you can clearly see and define which methods are shipped _back_ to the Module:

{% highlight javascript %}
var Module = (function () {

  var privateMethod = function () {
    // private
  };

  var someMethod = function () {
    // public
  };

  var anotherMethod = function () {
    // public
  };
  
  return {
    someMethod: someMethod,
    anotherMethod: anotherMethod
  };

})();
{% endhighlight %}

I really like the above syntax, as it's very declarative. For bigger JavaScript Modules this pattern helps out a lot more, using a standard "Module Pattern" can get out of control depending on the syntax you go for and how you structure your code.

### Accessing "Private" Methods
You might be thinking at some stage during this article, _"So if I make some methods private, how can I call them?"_. This is where JavaScript becomes even more awesome, and allows us to actually _invoke_ private functions via our public methods. Observe:

{% highlight javascript %}
var Module = (function () {

  var privateMethod = function (message) {
    console.log(message);
  };

  var publicMethod = function (text) {
    privateMethod(text);
  };
  
  return {
    publicMethod: publicMethod
  };

})();

// Example of passing data into a private method
// the private method will then `console.log()` 'Hello!'
Module.publicMethod('Hello!');
{% endhighlight %}

You're not just limited to methods, though. You've access to Objects, Arrays, anything:

{% highlight javascript %}
var Module = (function () {

  var privateArray = [];

  var publicMethod = function (somethingOfInterest) {
    privateArray.push(somethingOfInterest);
  };
  
  return {
    publicMethod: publicMethod
  };

})();
{% endhighlight %}

### Augmenting Modules
So far we've created a nice Module, and returned an Object. But what if we wanted to extend our Module, and include another smaller Module, which extends our original Module?

Let's assume the following code:

{% highlight javascript %}
var Module = (function () {

  var privateMethod = function () {
    // private
  };

  var someMethod = function () {
    // public
  };

  var anotherMethod = function () {
    // public
  };
  
  return {
    someMethod: someMethod,
    anotherMethod: anotherMethod
  };

})();
{% endhighlight %}

Let's imagine it's part of our application, but by design we've decided to not include something into the core of our application, so we could include it as a standalone Module, creating an extension.

So far our Object for `Module` would look like:

{% highlight javascript %}
Object {someMethod: function, anotherMethod: function}
{% endhighlight %}

But what if I want to add our Module extension, so it ends up with _another_ public method, maybe like this:

{% highlight javascript %}
Object {someMethod: function, anotherMethod: function, extension: function}
{% endhighlight %}

A third method is now available, but how do we manage it? Let's create an aptly named `ModuleTwo`, and pass in our `Module` namespace, which gives us access to our Object to extend:

{% highlight javascript %}
var ModuleTwo = (function (Module) {
    
    // access to `Module`
    
})(Module);
{% endhighlight %}

We could then create _another_ method inside this module, have all the benefits of private scoping/functionality and then return our extension method. My pseudo code could look like this:

{% highlight javascript %}
var ModuleTwo = (function (Module) {
    
    Module.extension = function () {
        // another method!
    };
    
    return Module;
    
})(Module || {});
{% endhighlight %}

`Module` gets passed into `ModuleTwo`, an extension method is added and then returned _again_. Our Object is getting thrown about, but that's the flexibility of JavaScript :D

I can then see (through something like Chrome's Dev Tools) that my initial Module now has a third property:

{% highlight javascript %}
// Object {someMethod: function, anotherMethod: function, extension: function}
console.log(Module);
{% endhighlight %}

Another hint here, you'll notice I've passed in `Module || {}` into my second `ModuleTwo`, this is incase `Module` is `undefined` - we don't want to cause errors now do we ;). What this does is instantiate a _new_ Object, and bind our `extension` method to it, and return it.

### Private Naming Conventions
I personally love the Revealing Module Pattern, and as such, I have many functions dotting around my code that visually are all declared the same, and look the same when I'm scanning around. I sometimes create a locally scoped Object, but sometimes don't. When I don't, how can I distinguish between private variables/methods? The `_` character! You've probably seen this dotted around the web, and now you know why we do it:

{% highlight javascript %}
var Module = (function () {

  var _privateMethod = function () {
    // private stuff
  };

  var publicMethod = function () {
    _privateMethod();
  };
  
  return {
    publicMethod: publicMethod
  };

})();
{% endhighlight %}
