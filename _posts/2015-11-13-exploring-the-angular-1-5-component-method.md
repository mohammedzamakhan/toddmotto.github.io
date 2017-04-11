---
layout: post
permalink: /exploring-the-angular-1-5-component-method/
title: Exploring the Angular 1.5 .component() method
path: 2015-11-13-exploring-the-angular-1-5-component-method.md
tags:
- New features
- Performance
- Components
- Directives
- Angular
---

AngularJS 1.5 introduced the `.component()` helper method, which is much simpler than the `.directive()` definition and advocates best practices and common default behaviours. Using `.component()` will allow developers to write in an Angular (v2+) style as well, which will in turn make upgrading to Angular an easier feat.

Let's compare the differences in syntax and the super neat abstraction that `.component()` gives us over using the `.directive()` method.

> Build an AngularJS 1.5 component architecture app, end-to-end with Firebase. Check out my [Angular 1.x Pro](https://ultimateangular.com/courses/#angular-1) course.

### Update: use component() now in AngularJS 1.3+

I've back-ported the AngularJS 1.5 `.component()` functionality to AngularJS 1.3 and above! [Read the article](/angular-component-method-back-ported-to-1.3) and grab the _latest_ [source code on GitHub](https://github.com/toddmotto/angular-component).

### .directive() to .component()

The syntax change is very simple:

{% highlight javascript %}
// before
module.directive(name, fn);

// after
module.component(name, options);
{% endhighlight %}

The `name` argument is what we want to define our Component as, the `options` argument is a definition Object passed into the component, rather than a function that we know so well in versions 1.4 and below.

I've prebuilt a simple `counter` component for the purposes of this exercise in AngularJS `1.4.x` which we'll refactor into a version `v1.5.0` build to use `.component()`.

{% highlight javascript %}
.directive('counter', function counter() {
  return {
    scope: {},
    bindToController: {
      count: '='
    },
    controller: function () {
      function increment() {
        this.count++;
      }
      function decrement() {
        this.count--;
      }
      this.increment = increment;
      this.decrement = decrement;
    },
    controllerAs: 'counter',
    template: `
      <div class="todo">
        <input type="text" ng-model="$ctrl.count">
        <button type="button" ng-click="$ctrl.decrement();">-</button>
        <button type="button" ng-click="$ctrl.increment();">+</button>
      </div>
    `
  };
});
{% endhighlight %}

A live embed of the `1.4.x` Directive:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/avdezer7/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

We'll continue building this alongside how we'd build the AngularJS 1.4 version to compare differences.

### Function to Object, method name change

Let's start from the top and refactor the `function` argument to become an `Object`, and change the name from `.directive()` to `.component()`:

{% highlight javascript %}
// before
.directive('counter', function counter() {
  return {

  };
});

// after
.component('counter', {

});
{% endhighlight %}

Nice and simple. Essentially the `return {};` statement inside the `.directive()` becomes the Object definition inside `.component()` - easy!

### "scope" and "bindToController", to "bindings"

In a `.directive()`, the `scope` property allows us to define whether we want to isolate the `$scope` or inherit it, this has now become a sensible default to (usually) always make our Directives have isolate scope. So repeating ourselves each time just creates excess boilerplate. With the introduction of `bindToController`, we can explicitly define which properties we want to pass into our isolate scope and bind directly to the Controller.

With the `bindings` property on `.component()` we can remove this boilerplate and simply define what we want to pass down to the component, under the assumption that the component will have isolate scope.

{% highlight javascript %}
// before
.directive('counter', function counter() {
  return {
    scope: {},
    bindToController: {
      count: '='
    }
  };
});

// after
.component('counter', {
  bindings: {
    count: '='
  }
});
{% endhighlight %}

### Controller and controllerAs changes

Nothing has changed in the way we declare `controller`, however it's now a little smarter and has a default `controllerAs` value of `$ctrl`.

If we're using a controller local to the component, we'll do this:

{% highlight javascript %}
// 1.4
{
  ...
  controller: function () {}
  ...
}
{% endhighlight %}

If we're using another Controller defined elsewhere, we'll do this:

{% highlight javascript %}
// 1.4
{
  ...
  controller: 'SomeCtrl'
  ...
}
{% endhighlight %}

If we want to define `controllerAs` at this stage (which will over-ride the default `$ctrl` value), we'll need to create a new property and define the instance alias:

{% highlight javascript %}
// 1.4
{
  ...
  controller: 'SomeCtrl',
  controllerAs: 'something'
  ...
}
{% endhighlight %}

This then allows us to use `something.prop` inside our `template` to talk to the instance of the Controller.

Now, there are some changes in `.component()` that make sensible assumptions and automatically create a `controllerAs` property under the hood for us, and automatically assign a name based on three possibilities:

{% highlight javascript %}
// inside angular.js
controllerAs: identifierForController(options.controller) || options.controllerAs || '$ctrl',
{% endhighlight %}

Possibility one uses this aptly named `identifierForController` function that looks like so:

{% highlight javascript %}
// inside angular.js
var CNTRL_REG = /^(\S+)(\s+as\s+(\w+))?$/;
function identifierForController(controller, ident) {
  if (ident && isString(ident)) return ident;
  if (isString(controller)) {
    var match = CNTRL_REG.exec(controller);
    if (match) return match[3];
  }
}
{% endhighlight %}

This allows us to do the following inside `.component()`:

{% highlight javascript %}
// 1.5
{
  ...
  controller: 'SomeCtrl as something'
  ...
}
{% endhighlight %}

This saves adding the `controllerAs` property... _however_...

We can add the `controllerAs` property to maintain backwards compatibility or keep it if that's within your style for writing Directives/Components.

The third option, and better yet, completely removes all need to think about `controllerAs`, and Angular automatically uses the name `$ctrl`. For instance:

{% highlight javascript %}
.component('test', {
  controller: function () {
    this.testing = 123;
  }
});
{% endhighlight %}

The would-be `controllerAs` definition automatically defaults to `$ctrl`, so we can use `$ctrl.testing` in our `template` which would give us the value of `123`.

Based on this information, we add our `controller`, and refactor our Directive into a Component by dropping the `controllerAs` property:

{% highlight javascript %}
// before
.directive('counter', function counter() {
  return {
    scope: {},
    bindToController: {
      count: '='
    },
    controller: function () {
      function increment() {
        this.count++;
      }
      function decrement() {
        this.count--;
      }
      this.increment = increment;
      this.decrement = decrement;
    },
    controllerAs: 'counter'
  };
});

// after
.component('counter', {
  bindings: {
    count: '='
  },
  controller: function () {
    function increment() {
      this.count++;
    }
    function decrement() {
      this.count--;
    }
    this.increment = increment;
    this.decrement = decrement;
  }
});
{% endhighlight %}

Things are becoming much simpler to use and define with this change.

### Template

There's a subtle difference in the `template` property worth noting. Let's add the `template` property to finish off our rework and then take a look.

{% highlight javascript %}
.component('counter', {
  bindings: {
    count: '='
  },
  controller: function () {
    function increment() {
      this.count++;
    }
    function decrement() {
      this.count--;
    }
    this.increment = increment;
    this.decrement = decrement;
  },
  template: `
    <div class="todo">
      <input type="text" ng-model="$ctrl.count">
      <button type="button" ng-click="$ctrl.decrement();">-</button>
      <button type="button" ng-click="$ctrl.increment();">+</button>
    </div>
  `
});
{% endhighlight %}

The `template` property can be defined as a function that is now injected with `$element` and `$attrs` locals. If the `template` property _is_ a function then it needs to return a String representing the HTML to compile:

{% highlight javascript %}
{
  ...
  template: function ($element, $attrs) {
    // access to $element and $attrs
    return `
      <div class="todo">
        <input type="text" ng-model="$ctrl.count">
        <button type="button" ng-click="$ctrl.decrement();">-</button>
        <button type="button" ng-click="$ctrl.increment();">+</button>
      </div>
    `
  }
  ...
}
{% endhighlight %}

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/xqauz9aa/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

That's it for our Directive to Component refactor; however, there are a few other changes worth exploring before we finish.

### Inheriting behaviour using "require"

If you're not familiar with "require", check [my article on using require](/directive-to-directive-communication-with-require).

{% highlight javascript %}
{
  ...
  require: {
    parent: '^^parentComponent'
  },
  controller: function () {
    // use this.parent to access required Objects
    this.parent.foo();
  }
  ...
}
{% endhighlight %}

Inherited Directive or Component methods will be bound to the `this.parent` property in the Controller.

### One-way bindings

A new syntax expression for isolate scope values, for example:

{% highlight javascript %}
{
  ...
  bindings: {
    oneWay: '<',
    twoWay: '='
  },
  ...
}
{% endhighlight %}

Read my full write-up about [one-way bindings](/one-way-data-binding-in-angular-1-5).

### Lifecycle hooks

Each component has a well-defined set of lifecycle hooks, read the [full article here](https://toddmotto.com/angular-1-5-lifecycle-hooks).

### Disabling isolate scope

Components are always created with isolate scope. Here's the relevant part from the source code:

{% highlight javascript %}
{
  ...
  scope: {},
  ...
}
{% endhighlight %}

### Stateless components

There's now the ability to create "stateless" components, read my in-depth article on [stateless components](/stateless-angular-components) in the `.component()` method.

Essentially we can just use a `template` and `bindings`:

{% highlight javascript %}
var NameComponent = {
  bindings: {
    name: '<',
    age: '<'
  },
  template: `
    <div>
      <p>Name: {% raw %}{{ $ctrl.name }}{% endraw %}</p>
      <p>Age: {% raw %}{{ $ctrl.age }}{% endraw %}</p>
    </div>
  `
};

angular
  .module('app', [])
  .component('nameComponent', NameComponent);
{% endhighlight %}

### Sourcecode for comparison

Throughout the article I've referred to some AngularJS source code snippets to cross reference against. Here's the source code below:

{% highlight javascript %}
this.component = function registerComponent(name, options) {
  var controller = options.controller || function() {};

  function factory($injector) {
    function makeInjectable(fn) {
      if (isFunction(fn) || isArray(fn)) {
        return function(tElement, tAttrs) {
          return $injector.invoke(fn, this, {$element: tElement, $attrs: tAttrs});
        };
      } else {
        return fn;
      }
    }

    var template = (!options.template && !options.templateUrl ? '' : options.template);
    var ddo = {
      controller: controller,
      controllerAs: identifierForController(options.controller) || options.controllerAs || '$ctrl',
      template: makeInjectable(template),
      templateUrl: makeInjectable(options.templateUrl),
      transclude: options.transclude,
      scope: {},
      bindToController: options.bindings || {},
      restrict: 'E',
      require: options.require
    };

    // Copy annotations (starting with $) over to the DDO
    forEach(options, function(val, key) {
      if (key.charAt(0) === '$') ddo[key] = val;
    });

    return ddo;
  }

  // TODO(pete) remove the following `forEach` before we release 1.6.0
  // The component-router@0.2.0 looks for the annotations on the controller constructor
  // Nothing in Angular looks for annotations on the factory function but we can't remove
  // it from 1.5.x yet.

  // Copy any annotation properties (starting with $) over to the factory and controller constructor functions
  // These could be used by libraries such as the new component router
  forEach(options, function(val, key) {
    if (key.charAt(0) === '$') {
      factory[key] = val;
      // Don't try to copy over annotations to named controller
      if (isFunction(controller)) controller[key] = val;
    }
  });

  factory.$inject = ['$injector'];

  return this.directive(name, factory);
};
{% endhighlight %}

Again, please note that AngularJS 1.5 isn't released just yet, so this article uses an API that _may_ be subject to slight change.

### Upgrading to Angular (v2+)

Writing components in this style will allow you to upgrade your Components using `.component()` into Angular very easily, it'd look something like this in ECMAScript 5 and new template syntax:

{% highlight javascript %}
import {Component} from '@angular/core';

@Component({
  selector: 'counter',
  template: `
    <div class="todo">
      <input type="text" [(ngModel)]="count">
      <button type="button" (click)="decrement();">-</button>
      <button type="button" (click)="increment();">+</button>
    </div>
  `
})
export default class CounterComponent {
  constructor() {

  }
  increment() {
    this.count++;
  }
  decrement() {
    this.count--;
  }
}
{% endhighlight %}
