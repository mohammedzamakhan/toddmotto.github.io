---
layout: post
permalink: /on-init-require-object-syntax-angular-component/
title: $onInit and new "require" Object syntax in Angular components
path: 2016-02-11-on-init-require-object-syntax-angular-component.md
---

The `component()` helper method shipped with so many great features to take us even closer towards Angular (v2+) syntax and integration. Let's explore the `$onInit` method and the new `require` property's syntax that makes the `component()` method much more powerful. If you've not checked out the `component()` method just yet, check [my write-up on it here](/exploring-the-angular-1-5-component-method).

### $onInit

Finally, the much needed callback when a component is mounted and ready - `$onInit`! It's easy to use and is part of the component's `controller`. See this example for usage:

{% highlight javascript %}
angular
  .module('app', [])
  .component('parentComponent', {
    transclude: true,
    template: `
      <div ng-transclude></div>
    `,
  })
  .component('childComponent', {
    bindings: {
      count: '='
    },
    controller: function () {
      this.state = 'Not loaded';
      this.$onInit = function() {
        this.state = 'Loaded!';
      };
    },
    template: `
      <div>
        Component: {% raw %}{{ $ctrl.state }}{% endraw %}
      </div>
    `
  });
{% endhighlight %}

And a live example:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/f1y8u4yj/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Angular has the `ngOnInit` method, a great lifecycle callback that will help us transition from AngularJS 1.x. `ngOnInit` is called right after the directive's data-bound properties have been checked for the first time, and before any of its children have been checked. It is invoked only once when the directive is instantiated - much like in this AngularJS 1.x implementation.

### Using "require" as an Object

Previously with Directives [we used "require"](/directive-to-directive-communication-with-require) to inherit methods from another Directive. This syntax was a simple `String` or `Array`, for example:

{% highlight javascript %}
angular
  .module('app', [])
  .directive('parentComponent', function () {
    scope: {},
    require: ['^parentDirective', 'ngModel'],
    controller: function () {
      // controller logic
    }
    link: function ($scope, $element, $attrs, $ctrl) {
      // $ctrl[0] === ^parentDirective
      // $ctrl[1] === ^ngModel
    },
    template: `
      <div>
        Component: {% raw %}{{ $ctrl.state }}{% endraw %}
      </div>
    `
  });
{% endhighlight %}

Note how in the above example the `require` property is an `Array`, which is horribly passed to the `link` function as `$ctrl`, where we can access the `Array` index such as `$ctrl[0]` to get specific Controllers that we need.

Thankfully, this is much nicer inside the `component()` method.

First, let's add a method to the `parentComponent` Controller and a simple `ng-transclude` to pass a child Component into:

{% highlight javascript %}
angular
  .module('app', [])
  .component('parentComponent', {
    transclude: true,
    template: `
      <div ng-transclude></div>
    `,
    controller: function () {
      this.foo = function () {
        return 'Foo from parent!';
      };
    }
  });
{% endhighlight %}

Then, we add the `childComponent` and declare `require` as an empty `Object`, empty Controller and placeholder `{% raw %}{{ $ctrl.state }}{% endraw %}` inside the template:

{% highlight javascript %}
angular
  .module('app', [])
  .component('parentComponent', {
    transclude: true,
    template: `
      <div ng-transclude></div>
    `,
    controller: function () {
      this.foo = function () {
        return 'Foo from parent!';
      };
    }
  })
  .component('childComponent', {
    require: {},
    controller: function () {

    },
    template: `
      <div>
        Component! {% raw %}{{ $ctrl.state }}{% endraw %}
      </div>
    `
  });
{% endhighlight %}

Next up, a nice syntax change, the `require` property is an `Object`, not a `String|Array` as we saw in Directives!

{% highlight javascript %}
angular
  .module('app', [])
  ...
  .component('childComponent', {
    require: {
      parent: '^parentComponent'
    },
    ...
  });
{% endhighlight %}

This allows us to now use `this.parent` as an inherited reference inside the `childComponent`'s Controller:

{% highlight javascript %}
angular
  .module('app', [])
  ...
  .component('childComponent', {
    require: {
      parent: '^parentComponent'
    },
    controller: function () {
      this.parent.foo();
    },
    ...
  });
{% endhighlight %}

But wait, this will throw an Error - it's trying to call `this.parent.foo();` before the Component is even ready. So let's use the `$onInit` method that we just learned to be able to call it when it's mounted:

{% highlight javascript %}
angular
  .module('app', [])
  ...
  .component('childComponent', {
    require: {
      parent: '^parentComponent'
    },
    controller: function () {
      this.$onInit = function () {
        this.parent.foo(); // 'Foo from parent!'
      };
    },
    ...
  });
{% endhighlight %}

Let's now bind `this.state` inside `childComponent`, and assign the result of the inherited `this.parent.foo();` call:

{% highlight javascript %}
angular
  .module('app', [])
  ...
  .component('childComponent', {
    require: {
      parent: '^parentComponent'
    },
    controller: function () {
      this.$onInit = function () {
        this.state = this.parent.foo();
      };
    },
    template: `
      <div>
        Component! {% raw %}{{ $ctrl.state }}{% endraw %}
      </div>
    `
  });
{% endhighlight %}

Altogether now:

{% highlight javascript %}
angular
  .module('app', [])
  .component('parentComponent', {
    transclude: true,
    template: `
      <div ng-transclude></div>
    `,
    controller: function () {
      this.foo = function () {
        return 'Foo from parent!';
      };
    }
  })
  .component('childComponent', {
    require: {
      parent: '^parentComponent'
    },
    controller: function () {
      this.$onInit = function () {
        this.state = this.parent.foo();
      };
    },
    template: `
      <div>
        Component! {% raw %}{{ $ctrl.state }}{% endraw %}
      </div>
    `
  });

document.addEventListener('DOMContentLoaded', function () {
  angular.bootstrap(document, ['app']);
});
{% endhighlight %}

Live output:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/zz21eh4k/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
