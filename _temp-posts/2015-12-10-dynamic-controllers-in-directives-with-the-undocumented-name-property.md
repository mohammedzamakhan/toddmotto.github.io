---
layout: post
permalink: /dynamic-controllers-in-directives-with-the-undocumented-name-property/
title: Dynamic Controllers in Directives with the undocumented "name" property
path: 2015-12-10-dynamic-controllers-in-directives-with-the-undocumented-name-property.md
tags:
- Dynamic Controllers
- Controllers
- Performance
- Angular
---

Assigning Controllers to Angular Directives is the norm when building out components. But what if you wanted to dynamically assign a Controller to the same Directive and template? There are reasons for using this technique (though uncommon), and the undocumented `name` property bound to each Directive's definition Object can allow us to do exactly that.

### Hard-coded Controller lookups

Let's look at a "hard-coded" Directive Controller which we use everyday:

{% highlight javascript %}
function FirstCtrl() {
  this.name = 'First Controller';
}
function fooDirective() {
  return {
    scope: {},
    controller: 'FirstCtrl',
    controllerAs: 'foo',
    template: '<div>{{ foo.name }}</div>',
    link: function ($scope, $element, $attrs, $ctrl) {
    
    }
  };
}

angular
  .module('app', [])
  .directive('fooDirective', fooDirective)
  .controller('FirstCtrl', FirstCtrl);
{% endhighlight %}

We have `FirstCtrl` which is passed into the Angular module, and then we reference it inside our Directive using `controller: 'FirstCtrl'`. This is great, however we can make this Controller lookup completely dynamic very easily!

Let's add another Controller and call it `SecondCtrl`:

{% highlight javascript %}
function SecondCtrl() {
  this.name = 'Second Controller';
}
{% endhighlight %}

### Dynamic Controller lookups

The next step is changing the Directive to allow the Controller lookup value to be dynamic. This is done using the undocumented `name` property:

{% highlight javascript %}
function fooDirective() {
  return {
    ...
    name: 'ctrl',
    ...
  };
}
{% endhighlight %}

What does `name: [value]` do? It allows us to pass value through an attribute into the Directive which we can use as a dynamic String for looking up Controllers in the module. The value of `name` becomes the attribute we bind to the Directive. For instance our `fooDirective` element will have an attribute `ctrl`.

If assigning a Controller, we would simply declare it as an attribute:

{% highlight html %}
<foo-directive ctrl="FirstCtrl"></foo-directive>
{% endhighlight %}

Pretty simple, right?

It won't work just yet, as our Directive `controller: 'FirstCtrl'` is still referencing wrong. We can actually change the `'FirstCtrl'` value to `'@'` and it'll work perfectly.

{% highlight javascript %}
function fooDirective() {
  return {
    ...
    name: 'ctrl',
    controller: '@',
    ...
  };
}
{% endhighlight %}

All together now:

{% highlight javascript %}
function FirstCtrl() {
  this.name = 'First Controller';
}

function SecondCtrl() {
  this.name = 'Second Controller';
}

function fooDirective() {
  return {
    scope: {},
    name: 'ctrl',
    controller: '@',
    controllerAs: 'foo',
    template: '<div>{{ foo.name }}</div>',
    link: function ($scope, $element, $attrs, $ctrl) {
    
    }
  };
}
  
angular
  .module('app', [])
  .directive('fooDirective', fooDirective)
  .controller('FirstCtrl', FirstCtrl)
  .controller('SecondCtrl', SecondCtrl);
{% endhighlight %}

Then we can add the same Directive again, passing in another Controller name as a String:

{% highlight html %}
<foo-directive ctrl="FirstCtrl"></foo-directive>
<foo-directive ctrl="SecondCtrl"></foo-directive>
{% endhighlight %}

This also plays extremely nicely with the `controllerAs` syntax inside our Directive's template, we don't need to change anything!

Live output:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/2k6xyscc/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### Link function

Inside the `link` function we also get the fourth argument (which I alias as `$ctrl`) given to us, with the correct instance of each dynamically assigned Controller:


{% highlight javascript %}
function fooDirective() {
  return {
    scope: {},
    name: 'ctrl',
    controller: '@',
    controllerAs: 'foo',
    template: '<div>{{ foo.name }}</div>',
    link: function ($scope, $element, $attrs, $ctrl) {
      console.log($ctrl.name);
    }
  };
}
{% endhighlight %}
