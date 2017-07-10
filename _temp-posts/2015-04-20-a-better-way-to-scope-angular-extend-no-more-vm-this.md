---
layout: post
permalink: /a-better-way-to-scope-angular-extend-no-more-vm-this/
title: A better way to $scope&#44; angular.extend&#44; no more &ldquo;vm &#61; this&rdquo;
path: 2015-04-20-a-better-way-to-scope-angular-extend-no-more-vm-this.md
tags:
- Scope
- this
- JavaScript
- Angular
---

The evolution of Angular Controllers has changed over the last year. As of now, most of us are working with the most recent addition to "Controller" syntax with the `controllerAs` style (doing away with binding directly to `$scope`).

There have been many style opinions around this, of which I've adopted myself, primarily the `var vm = this;` declaration at the top of Controllers. Of recent months, I've been doing away with using `vm` inside the actual JavaScript Controller, and steering towards plain JavaScript variables and functions, and binding those I need out as some kind of "exports".

To get a feel of what I was previously working from, let's start from the `var vm = this;` days.

### var vm = this;
This has been a really popular way of binding our variables to the Controller (which gets bound to `$scope`). Taking a simple example (note the `// exports` comment where I "bind" to the `vm` variable:

{% highlight javascript %}
function MainCtrl () {

  var vm = this;

  function doSomething() {

  }

  // exports
  vm.doSomething = doSomething;

}

angular
  .module('app')
  .controller('MainCtrl', MainCtrl);
{% endhighlight %}

This pattern is great (and has other variations such as binding `vm.doSomething = function () {}` directly without declaring the function above and assigning), and has been really helpful in working with Angular. The reason `vm` was created was to help with referencing the correct context inside other functions, as `this` doesn't follow the lexical scoping rules like other variables do, so we assign `this` to `vm` to make a "reference".

When you start having to bind a lot of things, we repeat `vm` so many times and end up with `vm.*` references all over our code. We don't really need to bind directly to the `this` value, JavaScript can work on its own (such as updating `var foo = {};` locally in a callback rather than `vm.foo`) for instance. An example with lots of `vm.*` bindings:

{% highlight javascript %}
function MainCtrl () {

  var vm = this;

  function doSomething1() {}
  function doSomething2() {}
  function doSomething3() {}
  function doSomething4() {}
  function doSomething5() {}
  function doSomething6() {}

  // exports
  vm.doSomething1 = doSomething1;
  vm.doSomething2 = doSomething2;
  vm.doSomething3 = doSomething3;
  vm.doSomething4 = doSomething4;
  vm.doSomething5 = doSomething5;
  vm.doSomething6 = doSomething6;
}

angular
  .module('app')
  .controller('MainCtrl', MainCtrl);
{% endhighlight %}

### Using angular.extend
This isn't a new idea, as we all know `angular.extend`, but I came across [this article](http://moduscreate.com/angularjs-tricks-with-angular-extend) from Modus Create which gave me the idea to completely drop all `vm` references given my current Angular Controller strategy/pattern. Their examples actually use `angular.extend($scope, {...});`, whereas my examples all adopt the `controllerAs` syntax.

Here's a quick example dropping `vm` references and just binding to `this`:
{% highlight javascript %}
function MainCtrl () {
  this.someVar = {
    name: 'Todd'
  };
  this.anotherVar = [];
  this.doSomething = function doSomething() {

  };
}

angular
  .module('app')
  .controller('MainCtrl', MainCtrl);
{% endhighlight %}

Using `angular.extend`, we keep things cleaner and more Object-driven, and we can pass out a simple exports Object instead of a list of items:

{% highlight javascript %}
function MainCtrl () {
  angular.extend(this, {
    someVar: {
      name: 'Todd'
    },
    anotherVar: [],
    doSomething: function doSomething() {

    }
  });
}

angular
  .module('app')
  .controller('MainCtrl', MainCtrl);
{% endhighlight %}

This way, we're not repeating the `this` keyword (or `$scope` if you're still on that).

It also makes it a little easier to use "private" methods, and see clearly between them:

{% highlight javascript %}
function MainCtrl () {
  
  // private
  function someMethod() {

  }

  // public
  var someVar = { name: 'Todd' };
  var anotherVar = [];
  function doSomething() {
    someMethod();
  }
  
  // exports
  angular.extend(this, {
    someVar: someVar,
    anotherVar: anotherVar,
    doSomething: doSomething
  });
}

angular
  .module('app')
  .controller('MainCtrl', MainCtrl);
{% endhighlight %}

Curious to hear thoughts and/or other practices.
