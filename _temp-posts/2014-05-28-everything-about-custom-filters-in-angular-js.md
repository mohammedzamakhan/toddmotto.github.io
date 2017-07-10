---
layout: post
permalink: /everything-about-custom-filters-in-angular-js/
title: Everything about custom filters in AngularJS
path: 2014-05-28-everything-about-custom-filters-in-angular-js.md
tags:
- Filters
- Performance
- Angular
- JavaScript
---

Angular filters are one of the toughest concepts to work with. They're a little misunderstood and it actually hurt my brain whilst learning them. Filters are insanely great, they're very powerful for transforming our data _so_ easily into reusable and scalable little chunks.

<div class="airpairme">
 <a href="http://airpair.me/toddmotto?utm_source=expert&utm_medium=homepage&utm_content=button&utm_campaign=airpairme">
  <img src="http://www.airpair.com/images/me.png" />
 </a>
 <p>For live <a href="http://www.airpair.com/angularjs" style="color:#CE5424">AngularJS help</a> on AirPair</p>
 <style>
  .airpairme > a {
   width:100%;
   display:block;
   padding:10px 0;
   margin: 10px 0;
   background:#fff;
   text-align:center;
   border:1px solid #d3d3d3;
   border-radius:5px;
  }
  .airpairme > a:hover { background: #f9f9f9 }
  .airpairme p { font:11px arial;text-align:center;margin:0;padding:3px 0 0 6px }
 </style>
</div>

I think it's best to understand what we even want to learn. To do that, we need to understand what filters really are and how we use them. For me, there are _four_ *types* of filters. Yes, four, but there of course can be other variants. Let's rattle through them:

### Filter 1: Static (single) use filter
Filter 1 just filters a single piece of Model data (not a loop or anything fancy) and spits it out into the View for us. This could be something like a date:

{% highlight html %}
<p>{% raw %}{{ 1400956671914 | date: 'dd-MM-yyyy' }}{% endraw %}</p>
{% endhighlight %}

When rendered, the DOM would look like this:

{% highlight html %}
<p>24-05-2014</p>
{% endhighlight %}

So how do we create this, or something similar?

Let's take my full name for example, if I wanted to quickly filter it and make it uppercase, how would we do it?

Angular has a `.filter()` method for each Module, which means we can write our own custom filters. Let's look at a stripped down filter:

{% highlight javascript %}
app.filter('', function () {
  return function () {
    return;
  };
});
{% endhighlight %}

As you can see, we can name our filter and we `return` a function. What the heck do these do?

The returned function gets invoked each time Angular calls the filter, which means two-way binding for our filters. The user makes a change, the filter runs again and updates as necessary. The name of our filter is how we can reference it inside Angular bindings.

Let's fill it in with some data:

{% highlight javascript %}
app.filter('makeUppercase', function () {
  return function (item) {
    return item.toUpperCase();
  };
});
{% endhighlight %}

So what do these mean? I'll annotate:

{% highlight javascript %}
// filter method, creating `makeUppercase` a globally
// available filter in our `app` module
app.filter('makeUppercase', function () {
  // function that's invoked each time Angular runs $digest()
  // pass in `item` which is the single Object we'll manipulate
  return function (item) {
    // return the current `item`, but call `toUpperCase()` on it
    return item.toUpperCase();
  };
});
{% endhighlight %}

As an example app:

{% highlight javascript %}
var app = angular.module('app', []);

app.filter('makeUppercase', function () {
  return function (item) {
      return item.toUpperCase();
  };
});

app.controller('PersonCtrl', function () {
  this.username = 'Todd Motto';
});
{% endhighlight %}

Then we declare it in the HTML:

{% highlight html %}
<div ng-app="app">
  <div ng-controller="PersonCtrl as person">
    <p>
      {% raw %}{{ person.username | makeUppercase }}{% endraw %}
    </p>
  </div>
</div>
{% endhighlight %}

And that's it, [jsFiddle link](//jsfiddle.net/toddmotto/xz39g) and output:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/xz39g/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### Filter 2: Filters for repeats

Filters are really handy for iterating over data and without much more work, we can do exactly that.

The syntax is quite similar when filtering a repeat, let's take some example data:

{% highlight javascript %}
app.controller('PersonCtrl', function () {
  this.friends = [{
    name: 'Andrew'        
  }, {
    name: 'Will'
  }, {
    name: 'Mark'
  }, {
    name: 'Alice'
  }, {
    name: 'Todd'
  }];
});
{% endhighlight %}

We can setup a normal `ng-repeat` on it:

{% highlight html %}
<ul>
  <li ng-repeat="friend in person.friends">
    {% raw %}{{ friend }}{% endraw %}
  </li>
</ul>
{% endhighlight %}

Add a filter called `startsWithA`, where we only want to show names in the Array beginning with `A`:

{% highlight html %}
<ul>
  <li ng-repeat="friend in person.friends | startsWithA">
    {% raw %}{{ friend }}{% endraw %}
  </li>
</ul>
{% endhighlight %}

Let's create a new filter:

{% highlight javascript %}
app.filter('startsWithA', function () {
  return function (items) {
    var filtered = [];
    for (var i = 0; i < items.length; i++) {
      var item = items[i];
      if (/a/i.test(item.name.substring(0, 1))) {
        filtered.push(item);
      }
    }
    return filtered;
  };
});
{% endhighlight %}

There are _two_ different things happening here! First, `item` previously is now `items`, which is our Array passed in from the `ng-repeat`. The second thing is that we need to return a _new_ Array. Annotated:

{% highlight javascript %}
app.filter('startsWithA', function () {
  // function to invoke by Angular each time
  // Angular passes in the `items` which is our Array
  return function (items) {
    // Create a new Array
    var filtered = [];
    // loop through existing Array
    for (var i = 0; i < items.length; i++) {
      var item = items[i];
      // check if the individual Array element begins with `a` or not
      if (/a/i.test(item.name.substring(0, 1))) {
        // push it into the Array if it does!
        filtered.push(item);
      }
    }
    // boom, return the Array after iteration's complete
    return filtered;
  };
});
{% endhighlight %}

ES5 version using `Array.prototype.filter` for a super clean filter:

{% highlight javascript %}
app.filter('startsWithA', function () {
  return function (items) {
    return items.filter(function (item) {
      return /a/i.test(item.name.substring(0, 1));
    });
  };
});
{% endhighlight %}

And that's it, [jsFiddle link](//jsfiddle.net/toddmotto/GDmN7) and output:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/GDmN7/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### Filter 3: Filters for repeats _with arguments_

Pretty much the same as the above, but we can pass arguments into the functions from other Models. Let's create an example that instead of "filtering by letter A", we can let the user decide, so they can type their own example:

{% highlight html %}
<input type="text" ng-model="letter">
<ul>
  <li ng-repeat="friend in person.friends | startsWithLetter:letter">
    {% raw %}{{ friend }}{% endraw %}
  </li>
</ul>
{% endhighlight %}

Here I'm passing the filter the `letter` Model value from `ng-model="letter"`. How does that wire up inside a custom filter?

{% highlight javascript %}
app.filter('startsWithLetter', function () {
  return function (items, letter) {
    var filtered = [];
    var letterMatch = new RegExp(letter, 'i');
    for (var i = 0; i < items.length; i++) {
      var item = items[i];
      if (letterMatch.test(item.name.substring(0, 1))) {
        filtered.push(item);
      }
    }
    return filtered;
  };
});
{% endhighlight %}

The most important thing to remember here is _how_ we're passing in arguments! Notice `letter` now exists inside the `return function (items, letter) {};`? This corresponds directly to the `:letter` part. Which means we can pass in as many arguments as we need (for example):

{% highlight html %}
<input type="text" ng-model="letter">
<ul>
  <li ng-repeat="friend in person.friends | startsWithLetter:letter:number:somethingElse:anotherThing">
    {% raw %}{{ friend }}{% endraw %}
  </li>
</ul>
{% endhighlight %}

We'd then get something like this:

{% highlight javascript %}
app.filter('startsWithLetter', function () {
  return function (items, letter, number, somethingElse, anotherThing) {
    // do a crazy loop
  };
});
{% endhighlight %}

And that's it, [jsFiddle link](//jsfiddle.net/toddmotto/53Xuk) and output:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/53Xuk/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### Filter 4: Controller/$scope filter

This one's a bit of a cheat, and I would only use it if you really have to use it. We take advantage of the `:arg` syntax and pass a `$scope` function into Angular's `filter` Object!

The difference with these type of filters is that the functions declared are the ones _passed into_ the filter function, so we're technically writing a function that gets passed into our return function. We don't get Array access, just the individual element. Important to remember.

Let's create another function that filters by letter `w` instead. First let's define the function in the Controller:

{% highlight javascript %}
app.controller('PersonCtrl', function () {
  // here's our filter, just a simple function
  this.startsWithW = function (item) {
    // note, that inside a Controller, we don't return
    // a function as this acts as the returned function!
    return /w/i.test(item.name.substring(0, 1));
  };
  this.friends = [{
    name: 'Andrew'        
  }, {
    name: 'Will'
  }, {
    name: 'Mark'
  }, {
    name: 'Alice'
  }, {
    name: 'Todd'
  }];
});
{% endhighlight %}

Then the repeat:

{% highlight html %}
<div ng-controller="PersonCtrl as person">
  <ul>
    <li ng-repeat="friend in person.friends | filter:person.startsWithW">
      {% raw %}{{ friend }}{% endraw %}
    </li>
  </ul>
</div>
{% endhighlight %}

These functions are obviously scoped and not reusable elsewhere. If it makes sense then use this setup, else don't...

And that's it, [jsFiddle link](//jsfiddle.net/toddmotto/gSXa7) and output:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/gSXa7/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Happy filtering ;)
