---
layout: post
permalink: /use-controller-filters-to-prevent-digest-performance-issues/
title: Using Controller $filters to prevent $digest performance issues
path: 2015-12-15-use-controller-filters-to-prevent-digest-performance-issues.md
tags:
- Filters
- Performance
- Controllers
---

Filters in Angular massively contribute to slow performance, so let's adopt a sensible way of doing things, which may take you an additional ten minute to code, but will dramatically enhance your application's performance.

Let's look at how removing filters in the DOM actually impacts our `$digest` cycles.

### What's a DOM filter?

A DOM filter is where we use a `|` pipe inside an expression to filter data:

{% highlight html %}
<p>{% raw %}{{ foo | uppercase }}{% endraw %}</p>
{% endhighlight %}

This might output a String `'todd'` as `'TODD'` as we've used the `| uppercase` filter. If you're not well versed with filters, check out [my article on custom Filters](http://toddmotto.com/everything-about-custom-filters-in-angular-js).

### Why are DOM filters bad?

They're easy to use, which means they probably have some internal performance overhead. And that's true, filters in the DOM are slower than running filters in JavaScript, however the main concern here is how often DOM filters get run.

Take this example, it uses a DOM filter on an `ng-repeat`, type some letters to see how it filters:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/sszcvb5L/1/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

This looks great, and has no immediate performance concerns, no page lag or input delay. However let's dig a little deeper.

### Filter $digest evaluation

For us Angular developers, DOM filters are branded as the "awesomeness" that ships with the core, and by all means, this is tremendous, but there's actually very little about the performance impacts of filters.

Are you ready to realise how your `$$watchers` are being choked by using DOM filters? Okay here goes.

Let's setup a basic test filter:

{% highlight javascript %}
function testFilter() {
  var filterCount = 0;
  return function (values) {
    return values.map(function (value) {
      filterCount++;
      // don't do this, this is just a hack to inject
      // the filter count into the DOM
      // without forcing another $digest
      document.querySelector('.filterCount').innerHTML = (
        'Filter count: ' + filterCount
      );
      return value;
    });
  };
}

angular
  .module('app', [])
  .filter('testFilter', testFilter);
{% endhighlight %}

This majestic `testFilter` will be bound to an `ng-repeat`, and each time the filter is called, it'll increment its internal counter and log it out in the console for us.

We can add it to the `ng-repeat` like so:

{% highlight html %}
<ul>
  <li ng-repeat="user in user.filteredUsers | testFilter">
    {% raw %}{{ user.name }}{% endraw %}
  </li>
</ul>
{% endhighlight %}

Let's also add some data to a Controller and force a `$digest` every `~1000ms` to see how quickly these filters start stacking up.

{% highlight javascript %}
function UserCtrl($interval) {

  var users = [{
    name: 'Todd Motto'
  },{
    name: 'Ryan Clark'
  },{
    name: 'Niels den Dekker'
  },{
    name: 'Jurgen Van de Moere'
  },{
    name: 'Jilles Soeters'
  }];
  
  // force a $digest every ~1000ms
  $interval(function () {}, 1000);
  
  this.filteredUsers = users;

}
{% endhighlight %}

And the output (note the "Filter count: X" text in the code embed). Please note, the `<input ng-model="">` functionality no longer exists however has an `ng-model` attribute bound, this is merely just to show filter evaluations inside the `$digest` loop.

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/jdmqmtmp/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Boom boom boom. Five filter calls every `$digest` loop. This can't be very efficient right? Especially with an `ng-repeat` with `1000` items inside. Think about it.

What's more, typing something inside the `<input ng-model="">` runs a `$digest` and numbers soar into double and triple figures. By the time you've read down to this stage the figure is probably into the hundreds, if not thousands.

Why are our filters running when we're not even filtering the `ng-repeat` with the `ng-model` anymore?

The filters here are run every single `$digest`.

### $filter in Controller

We almost certainly don't want the desired behaviour above, which if you've got large collections inside an `ng-repeat` and DOM filters, this is going to grind your application's performance to a halt very quickly.

Let's look at implementing a filter in the Controller using `$filter`.

{% highlight javascript %}
function UserCtrl($filter, $interval) {

  var users = [...];
  
  // force a $digest every ~1000ms
  $interval(function () {}, 1000);

  // updateUsers will call `testFilter` ourselves
  this.updateUsers = function (username) {
    this.filteredUsers = $filter('testFilter')(users);
  };
  
  this.filteredUsers = users;

}
{% endhighlight %}

And bind the `this.updateUsers` method to the `<input ng-model="">` as an `ng-change` event:

{% highlight html %}
<input 
  type="text" 
  placeholder="Type to filter" 
  ng-model="username" 
  ng-change="user.updateUsers(username);">
{% endhighlight %}

Yes, we're still forcing a `$digest` every `~1000ms` in this example. Let's observe the output and see when our `ng-repeat` filter gets called...

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/79rmgu99/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Answer: no filters are run until you type, rather than every `$digest`. We're making serious progress. Our filter is only called when we activate it through the `this.updateUsers` function, it's not cycled through each `$digest`.

Okay that's great, but how can I run my filter so it'll work?

Currently the `this.updateUsers` method takes an argument `(username)`, we can pass this into Angular's generic (and so wonderfully named) `filter` filter:

{% highlight javascript %}
this.updateUsers = function (username) {
  this.filteredUsers = $filter('filter')(users, username);
};
{% endhighlight %}

And that's all we need. Now we have a `$filter` that is actually only called when we need to filter, rather than having the filter function run every single `$digest` loop. This will significantly reduce the `$digest` overhead when not filtering, and run filters when actually necessary.

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/r7dm6dLa/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### "Chaining filters"

Yes, chaining filters is great, but their order of declaration affects the output of the filtered collection, and two, you're going to make the above `$digest` filter issues even worse.

How can I chain filters inside a Controller?

Easy, it's not essentially chaining, filters are just function calls. Filter your first collection, and pass the filtered collection off to another filter, then assign that finalised collection to the public property:

{% highlight javascript %}
this.updateUsers = function (username) {
  var filtered = $filter('filter')(users, username);
  filtered = $filter('orderBy')(filtered, 'name');
  this.filteredUsers = filtered;
};
{% endhighlight %}

With this example, I can use `ng-init="user.updateUsers(username);"` to instantly show you how this works:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/zLttm8bn/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

The filter runs `$filter('filter')(users, username)` followed by `$filter('orderBy')(filtered, 'name');`. Which passes in `var filtered...` as a variable. Essentially it's doing this inside itself, passing a filter into a filter:

{% highlight javascript %}
this.updateUsers = function (username) {
  this.filteredUsers = $filter('orderBy')($filter('filter')(users, username), 'name');
};
{% endhighlight %}

Filters in Controllers: do it.
