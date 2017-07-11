---
layout: post
permalink: /all-about-angulars-emit-broadcast-on-publish-subscribing/
title: Understanding Angular’s $scope and $rootScope event system $emit, $broadcast and $on
path: 2014-06-11-all-about-angulars-emit-broadcast-on-publish-subscribing.md
tags:
- Events
- Angular
- JavaScript
---

Angular's `$emit`, `$broadcast` and `$on` fall under the common "publish/subscribe" design pattern, or can do, in which you'd publish an event and subscribe/unsubscribe to it somewhere else. The Angular event system is brilliant, it makes things flawless and easy to do (as you'd expect!) but the concept behind it isn't so simple to master and you can often be left wondering why things don't work as you thought they might.

For those who are new to Angular and haven't used or seen `$emit`, `$broadcast` or `$on`, let's clarify what they do before we look at `$scope` and `$rootScope` event and scope relationships and how to utilise the event system correctly - as well as understand what's really going on.

### $scope.$emit up, $scope.$broadcast down

Using `$scope.$emit` will fire an event _up_ the `$scope`. Using `$scope.$broadcast` will fire an event _down_ the `$scope`. Using `$scope.$on` is how we listen for these events. A quick example:

{% highlight javascript %}
// firing an event upwards
$scope.$emit('myCustomEvent', 'Data to send');

// firing an event downwards
$scope.$broadcast('myCustomEvent', {
  someProp: 'Sending you an Object!' // send whatever you want
});

// listen for the event in the relevant $scope
$scope.$on('myCustomEvent', function (event, data) {
  console.log(data); // 'Data to send'
});
{% endhighlight %}

### $scope.($emit/$broadcast)

The key thing to remember when using `$scope` to fire your events, is that they will communicate only with _immediate_ parent or child scopes only! Scopes aren't always child and parent. We might have sibling scopes. Using `$scope` to fire an event will miss out sibling scopes, and just carry on up! _They do not go sideways!_

The simplest way to emulate parent and child scopes are to use Controllers. Each Controller creates _new_ `$scope`, which Angular neatly outputs an `ng-scope` class on newly scoped elements for us:

{% highlight html %}
<div ng-controller="ParentCtrl as parent" class="ng-scope">
  {% raw %}{{ parent.data }}{% endraw %}
  <div ng-controller="SiblingOneCtrl as sib1" class="ng-scope">
      {% raw %}{{ sib1.data }}{% endraw %}
  </div>
</div>
{% endhighlight %}

We could fire an event down from `ParentCtrl` to `SiblingOneCtrl` using `$broadcast`:

{% highlight javascript %}
app.controller('ParentCtrl',
  function ParentCtrl ($scope) {

  $scope.$broadcast('parent', 'Some data'); // going down!

});

app.controller('SiblingOneCtrl',
  function SiblingOneCtrl ($scope) {

  $scope.$on('parent', function (event, data) {
    console.log(data); // 'Some data'
  });

});
{% endhighlight %}

If we wanted to communicate upwards, from `SiblingOneCtrl` to `ParentCtrl`, you guessed it, we can use `$emit`.

{% highlight javascript %}
app.controller('ParentCtrl',
  function ParentCtrl ($scope) {

  $scope.$on('child', function (event, data) {
    console.log(data); // 'Some data'
  });

});

app.controller('SiblingOneCtrl',
  function SiblingOneCtrl ($scope) {

  $scope.$emit('child', 'Some data'); // going up!

});
{% endhighlight %}

To demonstrate how `$scope` works when firing the events, here's a simple hierarchy:

{% highlight html %}
<div ng-controller="ParentCtrl as parent" class="ng-scope">
  <div ng-controller="SiblingOneCtrl as sib1" class="ng-scope"></div>
  <div ng-controller="SiblingTwoCtrl as sib2" class="ng-scope"></div>
</div>
{% endhighlight %}

If `SiblingTwoCtrl` fired `$scope.$broadcast`, then `SiblingOneCtrl` would _never_ know it happened. This can be an annoyance, but a (slightly hacky-feely) remedy can be done:

{% highlight javascript %}
$scope.$parent.$broadcast('myevent', 'Some data');
{% endhighlight %}

What this does is jump up to `ParentCtrl` and then fire the `$broadcast` from there. 

### $rootScope.($emit/$broadcast)

If things weren't complicated enough, let's throw in `$rootScope` as well. `$rootScope` is the parent of _all_ scopes, which makes every newly created `$scope` a descendent! I mentioned above about how `$scope` is limited to direct scopes, `$rootScope` is how we could communicate across scopes with ease. Doing this will fit certain scenarios better than others. It's not as simple as up or down the scopes though, unfortunately...

#### $rootScope.$emit versus $rootScope.$broadcast

The `$rootScope` Object has the identical `$emit`, `$broadcast`, `$on` methods, but they work slightly differently to how `$scope` implements them. As `$rootScope` has no `$parent`, using an `$emit` would be pointless, right? Nope, instead, `$rootScope.$emit` will fire an event for all `$rootScope.$on` listeners _only_. The interesting part is that `$rootScope.$broadcast` will notify all `$rootScope.$on` _as well as_ `$scope.$on` listeners, subtle but very important difference if you want to avoid issues in your application.

#### $rootScope examples

Let's take an even deeper hierarchy:

{% highlight html %}
<div ng-controller="ParentCtrl as parent" class="ng-scope">
  // ParentCtrl
  <div ng-controller="SiblingOneCtrl as sib1" class="ng-scope">
    // SiblingOneCtrl
  </div>
  <div ng-controller="SiblingTwoCtrl as sib2" class="ng-scope">
    // SiblingTwoCtrl
    <div ng-controller="ChildCtrl as child" class="ng-scope">
      // ChildCtrl
    </div>
  </div>
</div>
{% endhighlight %}

The above has _3_ [lexical scopes](http://toddmotto.com/everything-you-wanted-to-know-about-javascript-scope) (where parent scopes are accessible in the current scope, kind of hurts your brain to think about it in terms of DOM scoping, but the concepts are there) and _4_ Angular scopes, `ParentCtrl`, `SiblingOneCtrl`, `SiblingTwoCtrl` and `ChildCtrl`. Two sibling scopes.

Using `$scope.$emit` inside `ChildCtrl` would result in `SiblingTwoCtrl` and `ParentCtrl` only being notified, as the event doesn't hit sibling scopes only _direct_ ancestors (completely ignoring `SiblingOneCtrl`). If we used `$rootScope`, however, then we can target `$rootScope` listeners as well.

{% highlight javascript %}
app.controller('SiblingOneCtrl',
  function SiblingOneCtrl ($rootScope) {

  $rootScope.$on('rootScope:emit', function (event, data) {
    console.log(data); // 'Emit!'
  });
  
  $scope.$on('rootScope:broadcast', function (event, data) {
    console.log(data); // 'Broadcast!'
  });
  
  $rootScope.$on('rootScope:broadcast', function (event, data) {
    console.log(data); // 'Broadcast!'
  });

});

app.controller('ChildCtrl',
  function ChildCtrl ($rootScope) {

  $rootScope.$emit('rootScope:emit', 'Emit!'); // $rootScope.$on
  $rootScope.$broadcast('rootScope:broadcast', 'Broadcast'); // $rootScope.$on && $scope.$on

});
{% endhighlight %}

### Unsubscribing from events

As part of the event system, you can unsubscribe from events at any time with the `$on` listener. Unlike other libraries, there is no `$off` method. The Angular docs aren't particularly clear on _how_ to "unsubscribe", the docs say that `$on` _"Returns a deregistration function for this listener."_. We can assume by that they mean a `closure` which allows us to unsubscribe.

Inside the source code of `v1.3.0-beta.11`, we can locate the `$on` method and confirm suspicions of a closure:

{% highlight javascript %}
$on: function(name, listener) {
  var namedListeners = this.$$listeners[name];
  if (!namedListeners) {
    this.$$listeners[name] = namedListeners = [];
  }
  namedListeners.push(listener);

  var current = this;
  do {
    if (!current.$$listenerCount[name]) {
      current.$$listenerCount[name] = 0;
    }
    current.$$listenerCount[name]++;
  } while ((current = current.$parent));

  var self = this;
  return function() {
    namedListeners[indexOf(namedListeners, listener)] = null;
    decrementListenerCount(self, 1, name);
  };
}
{% endhighlight %}

We can subscribe and unsubscribe very easily:

{% highlight javascript %}
app.controller('ParentCtrl',
  function ParentCtrl ($scope) {

  // subscribes...
  var myListener = $scope.$on('child', function (event, data) {
    // do something
  });

  // unsubscribes...
  // this would probably sit in a callback or something
  myListener();

});
{% endhighlight %}

#### $rootScope $destroy

When using `$rootScope.$on`, we need to unbind those listeners each time the `$scope` is destroyed. `$scope.$on` listeners are automatically unbound, but we'll need to call the above closure manually on the `$destroy` event:

{% highlight javascript %}
app.controller('ParentCtrl',
  function ParentCtrl ($scope) {

  // $rootScope $on
  var myListener = $rootScope.$on('child', function (event, data) {
    //
  });

  // $scope $destroy
  $scope.$on('$destroy', myListener);

});
{% endhighlight %}

### Cancelling events

If you choose to use `$emit`, one of your other `$scope` listeners can cancel it, so prevent it bubbling further. Using `$broadcast` has the opposite effect in which it _cannot_ be cancelled!

Cancelling an event which was sent via `$emit` looks like this:

{% highlight javascript %}
$scope.$on('myCustomEvent', function (event, data) {
  event.stopPropagation();
});
{% endhighlight %}

### $rootScope.$$listeners

Every Angular Object has several properties, we can dig into them and observe what's happening "under the hood". We can take a look at `$rootScope.$$listeners` to observe the listeners lifecycle. We can also unsubscribe from events that way as well by using this (but I wouldn't encourage it):

{% highlight javascript %}
$rootScope.$$listeners.myEventName = [];
{% endhighlight %}

### Event namespacing

Generally if I'm working on a particular Factory, I'll communicate to other Directives or Controllers or even Factories using a specific namespace for cleaner pub/subs, which keeps things consistent and avoid naming conflicts.

If I were building an email application with an Inbox, we might use an `inbox` namespace for that specific section. This is easily integrated with a few simple examples:

{% highlight javascript %}
$scope.$emit('inbox:send'[, data]);
$scope.$on('inbox:send', function (event, data) {...});

$scope.$broadcast('inbox:delete'[, data]);
$scope.$on('inbox:delete', function (event, data) {...});

$scope.$emit('inbox:save'[, data]);
$scope.$on('inbox:save', function (event, data) {...});
{% endhighlight %}

#### Further reading

Dig through the [docs](https://docs.angularjs.org/api/ng/type/$rootScope.Scope) for anything further! :)
