---
layout: post
permalink: /resolve-promises-in-angular-routes/
title: Resolve promises in Angular routes
path: 2015-12-14-resolve-promises-in-angular-routes.md
tags:
- Routing
- Promises
- Angular
---

Phase 1 of our Angular careers involved `$http` inside a Controller. Phase 2 involved abstracting into a Service and calling the Service inside a Controller. Phase 3 is the `resolve` property.

Let's take a look what `resolve` gives us.

### Phase 1: $http inside Controllers

Please don't do this, this is bad design and business logic is riddled in the Controller. This is merely to demonstrate the first way we probably used `$http`:

{% highlight javascript %}
function InboxCtrl($http) {
  this.messages = [];
  $http.get('/messages').then(function (response) {
    this.messages = response.data;
  }.bind(this));
}

angular
  .module('app', [])
  .controller('InboxCtrl', InboxCtrl);
{% endhighlight %}

We'll then probably have an `ng-repeat` that populates the DOM from `this.messages` once it has updated and the `$digest` has looped.

### Phase 2: Service abstraction

After realising that using `$http` in Controllers was a bad idea, we then kept things DRY, testable/reusable and abstracted out into the correct design pattern.

{% highlight javascript %}
// create an awesome InboxService to fetch our messages
function InboxService($http) {
  function getMessages() {
    return $http.get('/messages').then(function (response) {
      return response.data;
    });
  }
  return {
    getMessages: getMessages
  };
}

// inject InboxService and bind the 
// response to `this.messages`
function InboxCtrl(InboxService) {
  this.messages = [];
  InboxService.getMessages().then(function (response) {
    this.messages = response;
  }.bind(this));
}

angular
  .module('app', [])
  .controller('InboxCtrl', InboxCtrl)
  .factory('InboxService', InboxService);
{% endhighlight %}

This is pretty good, but it can be done a lot better. We're still instantiating our Controllers, _then_ making the call to `InboxService.getMessages()`. Would it not be better to first get the data, _then_ instantiate the Controller? Kerching, hello `resolve`.

### Phase 3: Resolve property on routes

For these examples I'm going to be using `ui-router`, so we'll be using `$stateProvider` instead of `$routeProvider`, but the syntax is exactly the same on both so you're golden to use it wherever.

We add a `resolve` property to each route we want to resolve data before Controller instantiation:

{% highlight javascript %}
$stateProvider
  .state('inbox', {
    url: '/inbox',
    templateUrl: 'partials/inbox.html',
    controller: 'InboxCtrl as vm',
    resolve: {
      messages: function (InboxService) {
        return InboxService.getMessages();
      }
    }
  });
{% endhighlight %}

Let's take a closer inspection and annotate the areas of interest that we need to learn:

{% highlight javascript %}
$stateProvider
  .state('inbox', {
    ...
    // Use an Object as the value of `resolve`
    resolve: {
      // create an Object property called "messages"
      // which will later be used for Dependency Injection
      // inside our Controller. Inject any Services we need as usual.
      messages: function (InboxService) {
        // Return our Service call, that returns a Promise
        return InboxService.getMessages();
      }
    }
  });
{% endhighlight %}

If the fact we called our Object property `messages` for dependency injection purposes hasn't clicked yet, it will now. We take this `messages` property name, and inject it into our Controller. The result of the Service call will be bound to the value (in this case the list of messages), which means our Controller is extremely slim, and is only instantiated once that data is there. This makes binding to our Controller extremely lightweight and simple.

{% highlight javascript %}
function InboxCtrl(messages) {
  this.messages = messages;
}
{% endhighlight %}

You can also have multiple Object properties on a resolve:

{% highlight javascript %}
$stateProvider
  .state('inbox', {
    ...
    resolve: {
      user: function (UserService) {
        return UserService.getUser();
      },
      messages: function (InboxService) {
        return InboxService.getMessages();
      }
    }
  });
{% endhighlight %}

And inject `user` and `messages` where necessary.

### Coupling routing logic with Controllers

In my [Angular styleguide](https://github.com/toddmotto/angularjs-styleguide#routing-resolves), I recommend using a `.resolve` property on Controllers, to contain the necessary `resolve` logic, and use hoisting to achieve the desired effects.

For example, I have this Object bound to the route:

{% highlight javascript %}
// router.js
resolve: {
  messages: function (InboxService) {
    return InboxService.getMessages();
  }
}
{% endhighlight %}

And this code inside the Controller.

{% highlight javascript %}
// InboxCtrl.js
function InboxCtrl(messages) {
  this.messages = messages;
}
{% endhighlight %}

It's easy to lose sight on what `messages` means whilst inspecting this file (a made-up `InboxCtrl.js`), as well as the overhead of having to go back to the pseudo `router.js` file, and scrolling through all our routes and resolve logic that continues to grow to make any changes.

The solution? Add a `.resolve` property to each Controller, full file example:

{% highlight javascript %}
// InboxCtrl.js
function InboxCtrl(messages) {
  this.messages = messages;
}

InboxCtrl.resolve = {
  messages: function (InboxService) {
    return InboxService.getMessages();
  }
}

angular
  .module('app')
  .controller('InboxCtrl', InboxCtrl);
{% endhighlight %}

And the `router.js`:

{% highlight javascript %}
$stateProvider
  .state('inbox', {
    url: '/inbox',
    templateUrl: 'partials/inbox.html',
    controller: 'InboxCtrl as vm',
    resolve: InboxCtrl.resolve
  });
{% endhighlight %}

### Hoisting reliance

This technique relies on hoisting. This means do not wrap all your files inside IIFE's, function scopes, closures or whatever you're calling it - the `InboxCtrl` needs to be available in the same scope so `InboxCtrl.resolve` can be accessed on the router.

"I wrap all my files inside an IIFE". Don't, it's a bit of a pointless and very manual venture. Wrap each module you create inside an IIFE or the entire application. You shouldn't have naming collisions so one IIFE should be golden to protect your app from the "dreaded" global state.

### DI resolve naming conventions

I'm not happy with simply injecting `messages` in this case, as route resolving is a special use case when injecting into Controllers, so from a developer perspective I'd like to know what dependencies are bound to the router's resolve Object without even looking at the `resolve = {}` Object.

Saying that, I'm also not settled on a specific convention, however I do consider prefixing dependencies a potential idea. Something like:

{% highlight javascript %}
// InboxCtrl.js
function InboxCtrl(_messages) {
  this.messages = _messages;
}

InboxCtrl.resolve = {
  _messages: function (InboxService) {
    return InboxService.getMessages();
  }
}

angular
  .module('app')
  .controller('InboxCtrl', InboxCtrl);
{% endhighlight %}

Nothing huge from the naming convention side, however could be an important decision when deciding for your application or team.

### Minification

To play nicely with minification you'll need to annotate in the following ways. If you're using ng-annotate then `@ngInject` is your friend here.

{% highlight javascript %}
// InboxCtrl.js
function InboxCtrl(_messages) {
  this.messages = _messages;
}

InboxCtrl.$inject = ['_messages'];

InboxCtrl.resolve = {
  _messages: ['InboxService', function (InboxService) {
    return InboxService.getMessages();
  }]
}

angular
  .module('app')
  .controller('InboxCtrl', InboxCtrl);
{% endhighlight %}
