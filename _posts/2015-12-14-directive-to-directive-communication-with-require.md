---
layout: post
permalink: /directive-to-directive-communication-with-require/
title: Directive to Directive communication with "require"
path: 2015-12-14-directive-to-directive-communication-with-require.md
tags:
- Communication
- Directives
- Controllers
- Angular
---

Communication between Directives can be done in various ways. When dealing with Directives that have a hierarchical relationship we can use Directive Controllers to talk between them.

In this article we'll build a `tabs` Directive that uses another Directive to add the tabs, using the `require` property of a Directive's definition Object.

First let's define the HTML:

{% highlight html %}
<tabs>
  <tab label="Tab 1">
    Tab 1 contents!
   </tab>
   <tab label="Tab 2">
    Tab 2 contents!
   </tab>
   <tab label="Tab 3">
    Tab 3 contents!
   </tab>
</tabs>
{% endhighlight %}

We can assume at this point we need two Directives, `tabs` and `tab`. Let's setup the base for `tabs`:

{% highlight javascript %}
function tabs() {
  return {
    restrict: 'E',
    scope: {},
    transclude: true,
    controller: function () {
      this.tabs = [];
    },
    controllerAs: 'tabs',
    template: `
      <div class="tabs">
        <ul class="tabs__list"></ul>
        <div class="tabs__content" ng-transclude></div>
      </div>
    `
  };
}

angular
  .module('app', [])
  .directive('tabs', tabs);
{% endhighlight %}

For this `tabs` Directive, we're going to use transclusion to pass each `tab` through and manage each `tab` individually.

Inside the `tabs` Controller, we'll need a function to add a new tab, so that our tabs are dynamically added to the parent/host Directive:

{% highlight javascript %}
function tabs() {
  return {
    ...
    controller: function () {
      this.tabs = [];
      this.addTab = function addTab(tab) {
        this.tabs.push(tab);
      };
    },
    ...
  };
}

angular
  .module('app', [])
  .directive('tabs', tabs);
{% endhighlight %}

The Controller now has an `addTab` method bound. But how do we add a tab? We need to get started with adding the child `tab` Directive, and require its Controller functionality:

{% highlight javascript %}
function tab() {
  return {
    restrict: 'E',
    scope: {
      label: '@'
    },
    require: '^tabs',
    transclude: true,
    template: `
      <div class="tabs__content" ng-if="tab.selected">
        <div ng-transclude></div>
      </div>
    `,
    link: function ($scope, $element, $attrs) {
      
    }
  };
}

angular
  .module('app', [])
  .directive('tab', tab)
  .directive('tabs', tabs);
{% endhighlight %}

We've successfully used `require: '^tabs'` to include the parent `tabs` Directive's Controller, so we now have access to its functionality through the `link` function. Now we need to inject a fourth argument, `$ctrl`, to get our Controller reference:

{% highlight javascript %}
function tab() {
  return {
    ...
    link: function ($scope, $element, $attrs, $ctrl) {
      
    }
  };
}
{% endhighlight %}

If we were to `console.log($ctrl);` we would see an Object similar to:

{% highlight javascript %}
{
  tabs: Array,
  addTab: function addTab(tab)
}
{% endhighlight %}

Let's utilise the `addTab` function and pass the newly created tab's information back up the Directive into the parent Directive's Controller:

{% highlight javascript %}
function tab() {
  return {
    ...
    link: function ($scope, $element, $attrs, $ctrl) {
      $scope.tab = {
        label: $scope.label,
        selected: false
      };
      $ctrl.addTab($scope.tab);
    }
  };
}
{% endhighlight %}

Now, each time a new `tab` Directive is used, this `$ctrl.addTab` function is called and pushed into the `this.tabs` Array inside the `tabs` Controller.

With three tabs our `$ctrl.addTab` function will be called three times, and the Array will contain three items. We can then use the Array to iterate over the tab titles and add clickable behaviour to select the correct tab when each title is clicked:

{% highlight javascript %}
function tabs() {
  return {
    restrict: 'E',
    scope: {},
    transclude: true,
    controller: function () {
      this.tabs = [];
      this.addTab = function addTab(tab) {
        this.tabs.push(tab);
      };
      this.selectTab = function selectTab(index) {
        for (var i = 0; i < this.tabs.length; i++) {
          this.tabs[i].selected = false;
        }
        this.tabs[index].selected = true;
      };
    },
    controllerAs: 'tabs',
    template: `
      <div class="tabs">
        <ul class="tabs__list">
          <li ng-repeat="tab in tabs.tabs">
            <a href="" ng-bind="tab.label" ng-click="tabs.selectTab($index);"></a>
          </li>
        </ul>
        <div class="tabs__content" ng-transclude></div>
      </div>
    `
  };
}
{% endhighlight %}

You'll now notice `selectTab` has been added to the `tabs` Controller. This allows us specify a tab's index, which will then reveal that tab's content. For instance `this.selectTab(0);` will set the first tab's content to be displayed, as we're using Array indexes to manage our logic.

Due to the Angular's compiling procedure, the `controller` is instantiated first, then the `link` function once the Directive has been compiled and linked, which means to set an initial tab's visibility we need to inject our Directive's Controller using `$ctrl` and call the method there:

{% highlight javascript %}
function tabs() {
  return {
    ...
    link: function ($scope, $element, $attrs, $ctrl) {
      // set the first tab to show first
      $ctrl.selectTab(0);
    },
  };
}
{% endhighlight %}

However, we might want to be more clever and allow an attribute to set the initial tab, giving the developer more flexibility:

{% highlight html %}
<tabs active="2">
  <tab>...</tab>
  <tab>...</tab>
  <tab>...</tab>
</tabs>
{% endhighlight %}

This would set the tab at Array index `2` (so the third element in the Array). We can utilise `$attrs` in the `link` function to obtain the attribute's presence, and set the index that way, or if the `$attrs.active` doesn't exist or is falsy (`0` evaluates to `false` so it'll fall back to `0` anyway), it'll fall back to setting the initial tab's index.

{% highlight javascript %}
function tabs() {
  return {
    ...
    link: function ($scope, $element, $attrs, $ctrl) {
      // set the active tab, or the first tab
      $ctrl.selectTab($attrs.active || 0);
    },
  };
}
{% endhighlight %}

And of course, the live demonstration of using `require` to pass new `tab` information back up into the parent Directive:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/4comjcdm/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
