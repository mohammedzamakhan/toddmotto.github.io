---
layout: post
permalink: /angular-component-method-back-ported-to-1.3/
title: Angular component() method back-ported to 1.3+
path: 2015-12-07-angular-component-method-back-ported-to-1.3.md
tags:
- New features
- Performance
- Components
- Directives
- Angular
---

AngularJS 1.5 is soon to release the `component()` method which [I wrote about last month](/exploring-the-angular-1-5-component-method). I decided to back-port the functionality from the new feature so anyone running AngularJS 1.3 and above can start using `component()` right now. It's 100% ported from the upcoming 1.5 release so the API is identical.

Grab the [source code here](https://github.com/toddmotto/angular-component) and use `component()` today.

Here's a live example running `component()` with AngularJS 1.3.0:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/wwzeo0sv/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### Why make this change?

The Angular component code on GitHub will work directly the same as AngularJS 1.5's implementation, so you can upgrade to using `component()` right now before a major version upgrade, such as jumping from 1.3 to 1.5.

Once you're levelled up with AngularJS 1.5, you can simply remove the `angular-component.js` script and your app will continue working exactly the same.

Angular (v2+) is here; a minor refactor to using `controllerAs`, `bindToController`, ditching `$scope` and writing in a component style will help you massively when upgrading your application to Angular.

For anyone interested in how the script works, I'll walk through the code.

### Hacking the AngularJS core

First, I had to hack into the AngularJS core:

{% highlight javascript %}
var ng = angular.module;

function module() {
  var hijacked = ng.apply(this, arguments);
  function component() {

  }
  hijacked.component = component;
  return hijacked;
}

angular.module = module;
{% endhighlight %}

These are the key players in hooking into the AngularJS core and adding another method. The annotated version:

{% highlight javascript %}
// save a reference to angular.module
var ng = angular.module;

// create a new function that will be called when .module() is instantiated
function module() {
  // hijack the existing module, apply context and arguments to mimic existing behaviour
  var hijacked = ng.apply(this, arguments);

  // create the new component function
  function component() {

  }

  // expose the component function on the hijacked module
  hijacked.component = component;

  // return the hijacked module as the new angular.module
  return hijacked;

}

// assign the new module function to angular.module
// so that each time it's called it takes the extra step above
angular.module = module;
{% endhighlight %}

### 1.5 Source-code hacking

I then took the source code from the upcoming 1.5, and threw it into the hijacked module definition:

{% highlight javascript %}
component: function(name, options) {
  function factory($injector) {
    function makeInjectable(fn) {
      if (angular.isFunction(fn)) {
        return function(tElement, tAttrs) {
          return $injector.invoke(fn, this, {$element: tElement, $attrs: tAttrs});
        };
      } else {
        return fn;
      }
    }

    var template = (!options.template && !options.templateUrl ? '' : options.template);
    return {
      controller: options.controller || function() {},
      controllerAs: identifierForController(options.controller) || options.controllerAs || name,
      template: makeInjectable(template),
      templateUrl: makeInjectable(options.templateUrl),
      transclude: options.transclude === undefined ? true : options.transclude,
      scope: options.isolate === false ? true : {},
      bindToController: options.bindings || {},
      restrict: options.restrict || 'E'
    };
  }

  if (options.$canActivate) {
    factory.$canActivate = options.$canActivate;
  }
  if (options.$routeConfig) {
    factory.$routeConfig = options.$routeConfig;
  }
  factory.$inject = ['$injector'];

  return moduleInstance.directive(name, factory);
}
{% endhighlight %}

I made a few basic changes such as moving a `template` variable inline:

{% highlight javascript %}
// before
var template = (!options.template && !options.templateUrl ? '' : options.template);
template: makeInjectable(template)

// after
template: makeInjectable(
  !options.template && !options.templateUrl ? '' : options.template
),
{% endhighlight %}

Had to fetch the `identifierForController` function from elsewhere in the source, and make a few changes to the `isString` function and do a simple `typeof X === 'string'` check (which is what the function does underneath anyway), and also moved the RegExp inline to save a variable declaration for this little script:

{% highlight javascript %}
// before
var CNTRL_REG = /^(\S+)(\s+as\s+(\w+))?$/;
function identifierForController(controller, ident) {
  if (ident && isString(ident)) return ident;
  if (isString(controller)) {
    var match = CNTRL_REG.exec(controller);
    if (match) return match[3];
  }
}

// after
function identifierForController(controller, ident) {
  if (ident && typeof ident === 'string') return ident;
  if (typeof controller === 'string') {
    var match = /^(\S+)(\s+as\s+(\w+))?$/.exec(controller);
    if (match) return match[3];
  }
}
{% endhighlight %}

### Patching "bindToController"

The `bindToController` property was introduced in AngularJS 1.3, however its value was limited to a Boolean until 1.4. If we wanted to use it, we would declare `bindToController: true` on the Directive definition Object. This means we had to use `scope: { prop: '=' }` when accessing inherited members. In AngularJS 1.4, we could use `scope: {}, bindToController: { prop: '=' }` and move our bindings to the `bindToController` property to be more explicit in saying "I want to bind these to a Controller". This is just syntax sugar, and obviously AngularJS 1.3 doesn't support an Object as the value of `bindToController`, so this needed to change.

Underneath it's identical, and it's also under a wrapper as we use `bindings` inside the `component()` method, so it doesn't really matter how we do the underlying detection and bindings.

{% highlight javascript %}
// before
scope: options.isolate === false ? true : {},
bindToController: options.bindings || {},

// after
scope: (
  options.isolate === false ?
  true :
  options.bindings
),
bindToController: !!options.bindings,
{% endhighlight %}

The first code snippet says: "If isolate scope is set to `false`, then set `scope` to `true` to inherit scope and prevent isolate scope. Otherwise set it to an empty Object to create isolate scope. Place my bindings on the `bindToController` property or use an empty Object if there aren't any".

The second (my change) code snippet says: "If isolate scope is set to `false`, then set `scope` to `true` to inherit scope and prevent isolate scope. Otherwise assign my `bindings` property value. Set `bindToController` to the result of `options.bindings` cast to Boolean, using `!!` casting".

Grab the code on [GitHub](https://github.com/toddmotto/angular-component) and start writing Angular-style components now. Enjoy!
