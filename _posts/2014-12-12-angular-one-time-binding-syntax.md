---
layout: post
permalink: /angular-one-time-binding-syntax/
title: AngularJS one-time binding syntax
path: 2014-12-12-angular-one-time-binding-syntax.md
tags:
- Performance
- Angular
---

Angular 1.3 shipped with an awesome new performance enhancing feature - one-time binding. What does this mean for us Angular developers and the performance of our apps? A lot! For us developers, adoption is really simple, and the performance gains for our apps are huge.

Let's look at the problem really quick, starting with the `$digest` cycle. The `$digest` cycle is essentially a loop through all our bindings which checks for changes in our data and re-renders any value changes. As our apps scale, binding counts increase and our `$digest` loop's size increases. This hurts our performance when we have a large volume of bindings per application view. So what can we do to optimise this?

Thankfully, Angular 1.3 has put a lot of effort into performance, so we can utilise their new syntaxes and APIs to build faster apps.

### What does it mean?
One-time binding is very simple, from the docs: _One-time expressions will stop recalculating once they are stable, which happens after the first digest if the expression result is a non-undefined value._

In a nut shell, when we declare a value such as `{% raw %}{{ foo }}{% endraw %}` inside the DOM, once this value becomes defined, Angular will render it, unbind it from the watchers and thus reduce the volume of bindings inside the `$digest` loop. Simple!

### The Syntax
The syntax is actually very simple, typically we're used to seeing something like this:

{% highlight html %}
<p>
  {% raw %}{{ vm.user }}{% endraw %}
</p>
{% endhighlight %}

The new syntax adds `::` in front of any values, which declares we want one time binding:

{% highlight html %}
<p>
  {% raw %}{{ ::vm.user }}{% endraw %}
</p>
{% endhighlight %}

Once `vm.user` becomes defined and contains a value, Angular will unbind it and any Model updates won't affect the View. A simple example is if we did this:

{% highlight html %}
<input type="text" ng-model="vm.user">
<p>
  {% raw %}{{ ::vm.user }}{% endraw %}
</p>
{% endhighlight %}

Anything typed into the input wouldn't render the Model value out in the View, consider it a "render-once" type method rather than bind once.

### Code examples

We already covered a simple example above using curly handlebars, we can also use it for things such as `ng-if`:

{% highlight html %}
<div ng-if="::vm.user.loggedIn"></div>
{% endhighlight %}

This helps us serve and maintain specific pieces of functionality on the front-end that we might want to simply destroy permanently based on a user's role or state (for example).

Let's see `ng-class`:

{% highlight html %}
<div ng-class="::{ loggedIn: vm.user.loggedIn }"></div>
{% endhighlight %}

Runs once, again great for initial states.

My favourite is still on the ng-repeat, instead of constructing our own navigation (for example) or using an `ng-repeat` _without_ one-time binding, we're going to be doing extra work or have our binding count higher when we really don't need it to be.

The syntax is pretty simple inside an `ng-repeat`:

{% highlight html %}
<ul>
  <li ng-repeat="user in ::vm.users"></li>
</ul>
{% endhighlight %}

Once the `ng-repeat` is populated, we're all set and everything is unbinded.

Syntax is pretty simple, albeit a little "weird", but after using it in production it sure helps other developers to know what's binding once.
