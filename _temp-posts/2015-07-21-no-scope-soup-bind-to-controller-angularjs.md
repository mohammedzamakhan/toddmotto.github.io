---
layout: post
permalink: /no-scope-soup-bind-to-controller-angularjs/
title: No $scope soup, bindToController in AngularJS
path: 2015-07-21-no-scope-soup-bind-to-controller-angularjs.md
tags:
- Scope
- Clean code
- Controllers
- Angular
---

Namespacing, code consistency and proper design patterns _really_ matter in software engineering, and Angular addresses a lot of issues we face as front-end engineers really nicely.

I'd like to show you some techniques using the `bindToController` property on Directives that will help clean up your DOM-Controller namespacing, help keep code consistent, and help follow an even better design pattern when constructing Controller Objects and inheriting data from elsewhere.

### Prerequisites

Use `bindToController` alongside `controllerAs` syntax, which treats Controllers as Class-like Objects, instantiating them as constructors and allowing us to namespace them once instantiated, such as the following:

{% highlight html %}
<div ng-controller="MainCtrl as vm">
  {% raw %}{{ vm.name }}{% endraw %}
</div>
{% endhighlight %}

Previously, without `controllerAs` we'd have no native namespacing of a Controller, and JavaScript Object properties simply floated around the DOM making it harder to keep code consistent inside Controllers, as well as running into inheritance issues with `$parent`. That's all we'll cover on this during this article, there's a [mighty post](//toddmotto.com/digging-into-angulars-controller-as-syntax) I've already published about it.

### Problem

Issues arise when writing Controllers that use the `controllerAs` syntax, we begin writing our components using a Class-like Object, only to end up injecting `$scope` to get access to inherited data (from "isolate scope"). A simple example of what we'd start with:

{% highlight javascript %}
// controller
function FooDirCtrl() {

  this.bar = {};
  this.doSomething = function doSomething(arg) {
    this.bar.foobar = arg;
  }.bind(this);

}

// directive
function fooDirective() {
  return {
    restrict: 'E',
    scope: {},
    controller: 'FooDirCtrl',
    controllerAs: 'vm',
    template: [
        // vm.name doesn't exist just yet!
        '<div><input ng-model="vm.name"></div>'
    ].join('')
  };
}

angular
  .module('app')
  .directive('fooDirective', fooDirective)
  .controller('FooDirCtrl', FooDirCtrl);
{% endhighlight %}

Now we need to "inherit" scope, so let's create the isolation hash in `scope: {}` to reference the binding we want:

{% highlight javascript %}
function fooDirective() {
  return {
    ...
    scope: {
      name: '='
    },
    ...
  };
}
{% endhighlight %}

And stop. Now we need to inject `$scope`, my Class-like Object has been vandalised by this `$scope` Object I've tried so hard to get rid of to adopt better design principles, and now I've got to inject it.

Onwards with the mess:

{% highlight javascript %}
// controller
function FooDirCtrl($scope) {

  this.bar = {};
  this.doSomething = function doSomething(arg) {
    this.bar.foobar = arg;
    $scope.name = arg.prop; // reference the isolate property
  }.bind(this);

}
{% endhighlight %}

At this point, we've likely ruined all excitement we had about the new Directive now our Class-like Object pattern has been ruined by `$scope`.

Not only this, but our pseudo-template would be affected with an un-namespaced variable floating amidst `vm.` prefixed ones:

{% highlight html %}
<div>
  {% raw %}{{ name }}{% endraw %}
  <input type="text" ng-model="vm.username">
</div>
{% endhighlight %}

### Solution

Before we go into what we'll deem as a solution, there are a lot of negative comments about Angular's attempts to replicate Class-like Object patterns, and I'm aware of the design, but we're making the most of what we've got - nothing's perfect and likely never will be, even with the rewrite v2.0. This post covers a great solution to cleaning up Angular's bad `$scope` habits as best as we can to write "proper" JavaScript designed in a better way.

Enter the `bindToController` property. In the docs, `bindToController` suggests that setting the value to `true` enables the inherited properties to be bound to the Controller, _not_ the `$scope` Object.

{% highlight javascript %}
function fooDirective() {
  return {
    ...
    scope: {
      name: '='
    },
    bindToController: true,
    ...
  };
}
{% endhighlight %}

This means we can refactor the previous code example, removing `$scope`:

{% highlight javascript %}
// controller
function FooDirCtrl() {

  this.bar = {};
  this.doSomething = function doSomething(arg) {
    this.bar.foobar = arg;
    this.name = arg.prop; // reference the isolate property using `this`
  }.bind(this);

}
{% endhighlight %}

The Angular documentation doesn't suggest that you can use an Object instead of `bindToController: true`, but in the [Angular source code](https://code.angularjs.org/1.4.3/angular.js) this line is present:

{% highlight javascript %}
if (isObject(directive.bindToController)) {
  bindings.bindToController = parseIsolateBindings(directive.bindToController, directiveName, true);
}
{% endhighlight %}

If it's an Object, parse the isolate bindings there instead. This means we can move our `scope: { name: '=' }` example binding across to it to make it more explicit that isolate bindings are in fact inherited and bound to the controller (my preferred syntax):

{% highlight javascript %}
function fooDirective() {
  return {
    ...
    scope: {},
    bindToController: {
      name: '='
    },
    ...
  };
}
{% endhighlight %}

Now we've solved the JavaScript solution, let's look at the template change impact this has.

Previously, we might have had `name` inherited and bound to `$scope`, whereas now we can use the same namespace as our Controller - rejoice. This keeps everything very consistent and readable. Finally we can `vm.` prefix our inherited `name` property to keep things in our template consistent!

{% highlight html %}
<div>
  {% raw %}{{ vm.name }}{% endraw %}
  <input type="text" ng-model="vm.username">
</div>
{% endhighlight %}

### Live Refactor examples

I've setup a few live examples on jsFiddle to demonstrate the refactor process (this was a great change for me and my team migrating from Angular 1.2 to 1.4 recently).

_Note: Each example uses two way isolate binding from a parent Controller passed down into the Directive, type to see changes reflected back up to the parent._

First example, using `$scope` Object's passed in. Would leave templating inconsistencies and Controller logic `$scope` and `this` mashups.

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/2n5skwqj/embedded/result,js" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Second example, refactor `$scope` with `bindToController: true` Boolean value. Fixes templating namespace issues as well as keeping the Controller logic consistent under the `this` Object.

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/2n5skwqj/1/embedded/result,js" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Third example (preferred), refactor `bindToController: true` into an Object, moving `scope: {}` properties across to it for clarity. Fixes same as example two, but adds clarity for other developers working/revisiting the piece of code.

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/2n5skwqj/2/embedded/result,js" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
