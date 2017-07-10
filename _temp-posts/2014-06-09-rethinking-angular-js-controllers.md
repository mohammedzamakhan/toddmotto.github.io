---
layout: post
permalink: /rethinking-angular-js-controllers/
title: Rethinking AngularJS Controllers
path: 2014-06-09-rethinking-angular-js-controllers.md
tags:
- Controllers
- MVVM
- Angular
- JavaScript
---

*Note: please don't use this approach anymore, use [Components and one-way dataflow](/exploring-the-angular-1-5-component-method/)*

Last month [Jonathan Creamer](https://twitter.com/jcreamer898) wrote an awesome article on Angular and MVC, _[The state of AngularJS Controllers](http://jonathancreamer.com/the-state-of-angularjs-controllers/)_. The article touches on misconceptions of client-side MVC and true Model and Controller separation, as well as things we often do wrong by incorrectly using a Controller as a Model.

This article is my rethinking of Angular Controllers, and best practices when using them in your team or for yourself. Getting and setting data outside of the Controller, resolving data is key to using a Model correctly.

### Controller as Model

What we're used to seeing in Controllers is most likely this (Angular docs even show us everything using this approach):

{% highlight javascript %}
app.controller('InboxCtrl',
  function InboxCtrl ($scope, $location, NotificationFactory) {

  $scope.messages = [];

  $scope.openMessage = function (message) {
    // myapp.com/message?id=19286396
    $location.search('id', message.id).path('/message');
  };

  $scope.deleteMessage = function (message) {
    $http.post('/message/delete', message)
    .success(function () {
      $scope.messages.splice(index, 1);
      NotificationFactory.showSuccess();
    });
  };

  $http.get('/messages')
  .success(function (data) {
    $scope.messages = data;
    NotificationFactory.showSuccess();
  })
  .error(function (data) {
    NotificationFactory.showError();
  });

});
{% endhighlight %}

This example totally abuses the Controller, almost treating it as a logic provider as well as the Model itself. We're using it to hold, create and maintain the Model state as well as littering our Factories and other Services. This has massive impact across _many_ things:

* Makes code reuse very difficult
* Tightly couples our code to a single Controller
* Makes testing very difficult
* Isn't separating the Model and Controller
* Inconsistent implementations across Controllers

This results in the Controller becoming a bloated playground which can quickly spiral out of control into an intangible mess.

### Abstracting the Model

Jonathan's post made it strictly clear that absolutely _no_ Model data should be created or persisted in the Controller. I totally second that. When I first started writing Angular applications, I began thinking "What is the best way to create reusable functions?". I'm pretty settled on the following...

Reworking our Controller, we could aim to completely kill all Model creations in the Controller:

{% highlight javascript %}
app.controller('InboxCtrl',
  function InboxCtrl ($scope, InboxFactory) {

  $scope.messages = InboxFactory.messages;

  $scope.openMessage = function (message) {
    InboxFactory.openMessage(message);
  };

  $scope.deleteMessage = function (message) {
    InboxFactory.deleteMessage(message);
  };

  InboxFactory
    .getMessages()
    .then(function () {
      $scope.messages = InboxFactory.messages;
    });

});
{% endhighlight %}

+1 for readability, +1 for simplicity and +1 for being so easy to reuse. I've included an example of using a promise to resolve asynchronously fetched data as well.

We of course need the Factory to go with it:

{% highlight javascript %}
app.factory('InboxFactory',
  function InboxFactory ($location, NotificationFactory) {

  InboxFactory.messages = [];

  InboxFactory.openMessage = function (message) {
    $location.search('id', message.id).path('/message');
  };

  InboxFactory.deleteMessage = function (message) {
    $http.post('/message/delete', message)
    .success(function (data) {
      InboxFactory.messages.splice(index, 1);
      NotificationFactory.showSuccess();
    })
    .error(function () {
      NotificationFactory.showError();
    });
  };

  InboxFactory.getMessages = function () {
    return $http.get('/messages')
    .success(function (data) {
      // magic line, we resolve the data IN the factory!
      InboxFactory.messages = data;
    })
    .error(function () {
      NotificationFactory.showError();
    });
  };

});
{% endhighlight %}

Binding into our Controllers just got a lot simpler, the Model is abstracted, as is the Controller.

### Resolving Model data, no callback arg binding in Controllers

You may have noticed from the above that I did this:

{% highlight javascript %}
InboxFactory
  .getMessages()
  .then(function () {
    $scope.messages = InboxFactory.messages;
  });
{% endhighlight %}

We might be used to something like this:

{% highlight javascript %}
InboxFactory
  .getMessages()
  .then(function (data) {
    $scope.messages = data;
  });
{% endhighlight %}

Which at first looks great, but breaks the golden rule of maintaining state in the Controller (as I've referenced `data` from the promise argument). The trick to this is resolving the data _inside_ the Factory itself, which allows it to manage itself and the Controller simply be told it's updated:

{% highlight javascript %}
InboxFactory.getMessages = function () {
  return $http.get('/messages')
  .success(function (data) {
    // magic line, we resolve the data IN the factory!
    InboxFactory.messages = data;
  })
  .error(function () {
    NotificationFactory.showError();
  });
{% endhighlight %}

At this point, the `$http.get()` is returned, and inside our Controller we can call `.then()` again and know our data has been resolved and bound to the Factory. We only reference the updated value in the Controller in the callback.

### "Controller as" syntax

I recently wrote about the ["Controller as"](http://toddmotto.com/digging-into-angulars-controller-as-syntax) syntax in Angular, and combined with the new MVC approach things just hit the next level of awesome inside Controllers:

{% highlight javascript %}
// <div ng-controller="InboxCtrl as inbox"></div>
app.controller('InboxCtrl',
  function InboxCtrl (InboxFactory) {

  this.messages = InboxFactory.messages;

  this.openMessage = function (message) {
    InboxFactory.openMessage(message);
  };

  this.deleteMessage = function (message) {
    InboxFactory.deleteMessage(message);
  };

  InboxFactory
    .getMessages()
    .then(function () {
    this.messages = InboxFactory.messages;
  }.bind(this)); // use angular.bind for < IE9

});
{% endhighlight %}

[John Papa](https://twitter.com/John_Papa) left an interesting comment on another article of mine, where he uses `var vm;` as a descriptor inside the Controller, meaning "View Model". His suggestion was to use the following, which also avoids needing to use `.bind()`:

{% highlight javascript %}
app.controller('InboxCtrl',
  function InboxCtrl (InboxFactory) {

  var vm = this;

  vm.messages = InboxFactory.messages;

  vm.openMessage = function (message) {
    InboxFactory.openMessage(message);
  };

  vm.deleteMessage = function (message) {
    InboxFactory.deleteMessage(message);
  };

  InboxFactory
    .getMessages()
    .then(function () {
    vm.messages = InboxFactory.messages;
  });

});
{% endhighlight %}

### What is the Model?

I've been chatting with [Jonathan](https://twitter.com/jcreamer898) recently on what _"is"_ the Model in client-side MVC. Is it the data? Is it the way to get the data? Is it both? Is it a way to persist the data?

We kind of agreed on a mix of both and that there are many options and opinions. We also agreed that using `Controller as` mixed with the above approach is superb architecture for Angular. The word "Model" might not be completely correct on what I'm describing above, business logic might be another word, but to keep things easy I've stuck with "Model".

Jasmine/Karma unit testing is made much easier as well, we can test the Factory to ensure it's hitting all its endpoints, fetching and updating the local data - and when testing the Controller we can go in knowing our Factory is bulletproof which will help us track down errors faster, and make our Controller tests even slimmer.

### Using $scope

If you need to use `$scope`, you'll likely need to listen to an event or emit one, or `$watch` for Model changes. Based on the above, this isn't a good idea to tie Model behaviour into the Controller again. Using `Controller as` allows us to inject the $scope, but think about how you can abstract the same behaviour into a Factory.

The `$scope` Object can be dependency injected into the Controller and used if needed.

### Rules for the Controller going forward:

* Controllers should hold zero logic
* Controllers should bind references to Models only (and call methods returned from promises)
* Controllers only bring logic together
* Controller drives Model _changes_, and View _changes_. Keyword; drives, not creates/persists, it triggers them!
* Delegate updating of logic inside Factories, don't resolve data inside a Controller, only update the Controller's value with updated Factory logic, this avoids repeated code across Controllers as well as Factory tests made easier
* Use "Controller as" ([read the article](http://toddmotto.com/digging-into-angulars-controller-as-syntax/))
* Keep things simple, I prefer XXXXCtrl and XXXXFactory, I know exactly what the two do, we don't need fancy names for things
* Keep method/prop names consistent across shared methods, such as `this.something = MyFactory.something;` otherwise it becomes confusing
* Factories hold the Model, change, get, update, and persist the Model changes
* Think about the Factory as an Object that you need to persist, rather than persisting inside a Controller
* Talk to other Factories inside your Factory, keep them out the Controller (things like success/error handling)
* Try to avoid injecting `$scope` into Controllers, generally there are better ways to do what you need, such as avoiding `$scope.$watch()`
