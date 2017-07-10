---
layout: post
permalink: /digging-into-angulars-controller-as-syntax/
title: Digging into Angular’s “Controller as” syntax
path: 2014-05-26-digging-into-angulars-controller-as-syntax.md
tags:
- ControllerAs
- Controllers
- Angular
- JavaScript
---

AngularJS Controllers have recently gone under some changes (version 1.2 to be precise). What this means for scopes, Controllers and Angular development is some very subtle but powerful changes. One of those changes I believe is improved architecture, clearer scoping and smarter Controllers.

Controllers as we know them are class-like Objects that drive Model and View changes, but they all seem to revolve around this mystical `$scope` Object. Angular Controllers have been pushed to change the way `$scope` is declared, with many developers suggesting using the `this` keyword instead of `$scope`.

Pre v1.2.0 Controllers looked similar to this:

{% highlight javascript %}
// <div ng-controller="MainCtrl"></div>
app.controller('MainCtrl', function ($scope) {
  $scope.title = 'Some title';
});
{% endhighlight %}

Here, the concept of the Controller is separate from the $scope itself, as we have to dependency inject it. Some argued this would've been better:

{% highlight javascript %}
app.controller('MainCtrl', function () {
  this.title = 'Some title';
});
{% endhighlight %}

We didn't _quite_ get there, but we got something pretty awesome in return.

### Controllers as Classes

If you instantiate a "class" in JavaScript, you might do this:

{% highlight javascript %}
var myClass = function () {
  this.title = 'Class title';
}
var myInstance = new myClass();
{% endhighlight %}

We can then use the `myInstance` instance to access `myClass` methods and properties. In Angular, we get the feel of proper instantiation with the new `Controller as` syntax. Here's a quick look at declaring and binding:

{% highlight javascript %}
// we declare as usual, just using the `this` Object instead of `$scope`
app.controller('MainCtrl', function () {
  this.title = 'Some title';
});
{% endhighlight %}

This is more of a class based setup, and when instantiating a Controller in the DOM we get to instantiate against a variable:

{% highlight html %}
<div ng-controller="MainCtrl as main">
  // MainCtrl doesn't exist, we get the `main` instance only
</div>
{% endhighlight %}

To reflect `this.title` in the DOM, we need to ride off our instance:

{% highlight html %}
<div ng-controller="MainCtrl as main">
   {% raw %}{{ main.title }}{% endraw %}
</div>
{% endhighlight %}

Namespacing the scopes is a great move I think, it cleans up Angular _massively_. I've always disliked the "floating variables" such as `{% raw %}{{ title }}{% endraw %}`, I much prefer hitting the instance with `{% raw %}{{ main.title }}{% endraw %}`.

### Nested scopes

Nested scopes is where we see great return from the `Controller as` syntax, often we've had to use the current scope's `$parent` property to scale back _up_ scopes to get where we need.

Take this for example:

{% highlight html %}
<div ng-controller="MainCtrl">
  {% raw %}{{ title }}{% endraw %}
  <div ng-controller="AnotherCtrl">
    {% raw %}{{ title }}{% endraw %}
    <div ng-controller="YetAnotherCtrl">
      {% raw %}{{ title }}{% endraw %}
    </div>
  </div>
</div>
{% endhighlight %}

Firstly, we're going to get interpolation issues as `{% raw %}{{ title }}{% endraw %}` will be very confusing to use and most likely one scope will take precedence over another. We also don't know which one that might be. Whereas if we did this things are far clearer and variables can be accessed properly across scopes:

{% highlight html %}
<div ng-controller="MainCtrl as main">
  {% raw %}{{ main.title }}{% endraw %}
  <div ng-controller="AnotherCtrl as another">
    {% raw %}{{ another.title }}{% endraw %}
    <div ng-controller="YetAnotherCtrl as yet">
      {% raw %}{{ yet.title }}{% endraw %}
    </div>
  </div>
</div>
{% endhighlight %}

I can also access parent scopes without doing this:

{% highlight html %}
<div ng-controller="MainCtrl">
  {% raw %}{{ title }}{% endraw %}
  <div ng-controller="AnotherCtrl">
    Scope title: {% raw %}{{ title }}{% endraw %}
    Parent title: {% raw %}{{ $parent.title }}{% endraw %}
    <div ng-controller="YetAnotherCtrl">
      {% raw %}{{ title }}{% endraw %}
      Parent title: {% raw %}{{ $parent.title }}{% endraw %}
      Parent parent title: {% raw %}{{ $parent.$parent.title }}{% endraw %}
    </div>
  </div>
</div>
{% endhighlight %}

And make things more logical:

{% highlight html %}
<div ng-controller="MainCtrl as main">
  {% raw %}{{ main.title }}{% endraw %}
  <div ng-controller="AnotherCtrl as another">
    Scope title: {% raw %}{{ another.title }}{% endraw %}
    Parent title: {% raw %}{{ main.title }}{% endraw %}
    <div ng-controller="YetAnotherCtrl as yet">
      Scope title: {% raw %}{{ yet.title }}{% endraw %}
      Parent title: {% raw %}{{ another.title }}{% endraw %}
      Parent parent title: {% raw %}{{ main.title }}{% endraw %}
    </div>
  </div>
</div>
{% endhighlight %}

No hacky `$parent` calls. If a Controller's position in the DOM/stack were to change, the position in sequential `$parent.$parent.$parent.$parent` may change! Accessing the scope lexically makes perfect sense.

### $watchers/$scope methods
The first time I used the `Controller as` syntax I was like "yeah, awesome!", but then to use scope watchers or methods (such as `$watch`, `$broadcast`, `$on` etc.) we need to dependency inject `$scope`. Gargh, this is what we tried so hard to get away from. But then I realised this was awesome.

The way the `Controller as` syntax works, is by _binding_ the Controller to the current `$scope` rather than it being all one `$scope`-like class-like Object. For me, the key is the separation between the class and special Angular features.

This means I can have my pretty class-like Controller:

{% highlight javascript %}
app.controller('MainCtrl', function () {
  this.title = 'Some title';
});
{% endhighlight %}

When I need something above and beyond generic bindings, I introduce the magnificent `$scope` dependency to do something _special_, rather than ordinary.

Those special things include all the `$scope` methods, let's look at an example:

{% highlight javascript %}
app.controller('MainCtrl', function ($scope) {
  this.title = 'Some title';
  $scope.$on('someEventFiredFromElsewhere', function (event, data) {
    // do something!
  });
});
{% endhighlight %}

#### Ironing a quirk
Interestingly enough, whilst writing this I wanted to provide a `$scope.$watch()` example. Doing this usually is very simple, but using the `Controller as` syntax doesn't work quite as expected:

{% highlight javascript %}
app.controller('MainCtrl', function ($scope) {
  this.title = 'Some title';
  // doesn't work!
  $scope.$watch('title', function (newVal, oldVal) {});
  // doesn't work!
  $scope.$watch('this.title', function (newVal, oldVal) {});
});
{% endhighlight %}

Uh oh! So what do we do? Interestingly enough I was reading the other day, and you can actually pass in a function as the first argument of a `$watch()`:

{% highlight javascript %}
app.controller('MainCtrl', function ($scope) {
  this.title = 'Some title';
  // hmmm, a function
  $scope.$watch(function () {}, function (newVal, oldVal) {});
});
{% endhighlight %}

Which means we can return our `this.title` reference:

{% highlight javascript %}
app.controller('MainCtrl', function ($scope) {
  this.title = 'Some title';
  // nearly there...
  $scope.$watch(function () {
    return this.title; // `this` isn't the `this` above!!
  }, function (newVal, oldVal) {});
});
{% endhighlight %}

Let's change some execution context using `angular.bind()`:

{% highlight javascript %}
app.controller('MainCtrl', function ($scope) {
  this.title = 'Some title';
  // boom
  $scope.$watch(angular.bind(this, function () {
    return this.title; // `this` IS the `this` above!!
  }), function (newVal, oldVal) {
    // now we will pickup changes to newVal and oldVal
  });
});
{% endhighlight %}

## Declaring in `$routeProvider`/Directives/elsewhere
Controllers can by dynamically assigned, we don't need to always bind them via attributes. Inside Directives, we get a `controllerAs:` property, this is easily assigned:

{% highlight javascript %}
app.directive('myDirective', function () {
  return {
    restrict: 'EA',
    replace: true,
    scope: true,
    template: [].join(''),
    controllerAs: '', // woohoo, nice and easy!
    controller: function () {}, // we'll instantiate this controller "as" the above name
    link: function () {}
  };
});
{% endhighlight %}

The same inside `$routeProvider`:

{% highlight javascript %}
app.config(function ($routeProvider) {
  $routeProvider
  .when('/', {
    templateUrl: 'views/main.html',
    controllerAs: '',
    controller: ''
  })
  .otherwise({
    redirectTo: '/'
  });
});
{% endhighlight %}

### Testing controllerAs syntax

There's a subtle difference when testing `controllerAs`, and thankfully we no longer need to dependency inject `$scope`. This means we also don't need to have a reference property when testing the Controller (such as `vm.prop`), we can simply use the variable name we assign `$controller` to.

{% highlight javascript %}
// controller
angular
  .module('myModule')
  .controller('MainCtrl', MainCtrl);

function MainCtrl() {
  this.title = 'Some title';
};

// tests
describe('MainCtrl', function() {
  var MainController;

  beforeEach(function(){
    module('myModule');

    inject(function($controller) {
      MainController = $controller('MainCtrl');
    });
  });

  it('should expose title', function() {
    expect(MainController.title).equal('Some title');
  });
});
{% endhighlight %}

You can alternatively use the `controllerAs` syntax in the `$controller` instantiation but you will need to inject a `$scope` instance into the Object that is passed into `$controller`. The alias (for instance `scope.main`) for the Controller will be added to this `$scope` (like it is in our actual Angular apps), however this is a less elegant solution.

{% highlight javascript %}
// Same test becomes
describe('MainCtrl', function() {
  var scope;

  beforeEach(function(){
    module('myModule');

    inject(function($controller, $rootScope) {
      scope = $rootScope.$new();
      var localInjections = {
        $scope: scope,
      };
      $controller('MainCtrl as main', localInjections);
    });
  });

  it('should expose title', function() {
    expect(scope.main.title).equal('Some title');
  });
});
{% endhighlight %}
