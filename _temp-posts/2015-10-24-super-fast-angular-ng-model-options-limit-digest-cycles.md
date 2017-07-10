---
layout: post
permalink: /super-fast-angular-ng-model-options-limit-digest-cycles/
title: Superfast Angular&#58; use ngModelOptions to limit &dollar;digest cycles
path: 2015-10-24-super-fast-angular-ng-model-options-limit-digest-cycles.md
tags:
- Performance
- ngModelOptions
- Digest
- Angular
---

The `$digest` cycle is the critical entity for keeping our Angular applications fast: the faster the cycle, the faster the two-way data binding. JavaScript has a single thread of execution, which means if our `$digest` cycle is packed full of data to be dirty-checked, the user is going to see lag in the UI whilst (for instance) typing inside an `<input>`.

`$digest` cycles run from internal Angular events (yes and `$scope.$apply()`) built into their Directives, such as `ng-click`, `ng-change` and so on. When something triggers these internal events, such as a `keypress` that triggers an `<input>` with `ng-model` bound to it, Angular will run the `$digest` loop to see if anything has changed. If something has changed, Angular will update the bound JavaScript Model. If something in the Model changes, Angular will run the `$digest` again to update the View. Basics of "dirty-checking" - simple.

### Problem

The problem with dirty-checking is that larger `$digest` loops will take longer to complete, which means the user could see some lag whilst using the application. Understanding the performance impacts upfront can help you build better applications using the right APIs.

For this article, I've written a tiny Directive that logs our `$digest` cycle counts, so we can actually see the impact that simple UI interactions may have on our applications.

Let's take a simple `<input>` for this example with `ng-model` bound to it:

{% highlight html %}
<input 
  type="text" 
  ng-model="test">
{% endhighlight %}

The live output:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/5wmv98sb/embedded/result" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Type away, you'll see the `$digest` count rockets into double and triple figures. Now, with data heavy applications this is going to cause some severe lag for the end user.

### Solution

Using `ngModelOptions`, a Directive introduced in Angular 1.3, we can add a debounce to `<input>` triggers so we have complete control over how and when `$digest` cycles occur.

Let's add the `ng-model-options` attribute and tell Angular to update our Model on the `blur` event only using the `updateOn` property:

{% highlight html %}
<input 
  type="text" 
  ng-model="test"
  ng-model-options="{
    'updateOn': 'blur'
  }">
{% endhighlight %}

Type a bunch of characters in, and then click/tab out the `<input>` to see the `$digest` count increase.

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/qmat5o8s/embedded/result" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

This is a massive performance improvement as we're not continuously running `$digest`. We're not limited to a single event, so let's add `'default'` as an option:

{% highlight html %}
<input 
  type="text" 
  ng-model="test"
  ng-model-options="{
    'updateOn': 'default blur'
  }">
{% endhighlight %}

With the `default` value also added, we're back to square one where `$digest` will run each time the user types. Time to get smart! Let's tell Angular to periodically update the Model, so that other bindings that may rely on the Model being persisted will be updated too. We can introduce the `debounce` property inside the `ng-model-options` Object to tell Angular exactly when to update the Model after specific events occur.

{% highlight html %}
<input 
  type="text" 
  ng-model="test"
  ng-model-options="{
    'updateOn': 'default blur',
    'debounce': {
      'default': 250,
      'blur': 0
    }
  }">
{% endhighlight %}

The above illustrates that `default` will be updated `250ms` after the event stops, and `blur` will update immediately as the user leaves the input (if this is the desired behaviour we want).

Start typing again, then stop and note the `$digest` count is severely lower than the initial demonstration. You can then click/tab out the `<input>` to call another `$digest` immediately.

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/ee85yhem/embedded/result" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### $digest tracking Directive

If anyone is interested in using the code from the Directive for testing purposes feel free:

{% highlight javascript %}
/**
 * trackDigest.js
 * Simple counter for counting when $digests occur
 * @author Todd Motto
 */
function trackDigests($rootScope) {
  function link($scope, $element, $attrs) {
    var count = 0;
    function countDigests() {
      count++;
      $element[0].innerHTML = '$digests: ' + count;
    }
    $rootScope.$watch(countDigests);
  }
  return {
    restrict: 'EA',
    link: link
  };
}

angular
  .module('app')
  .directive('trackDigests', trackDigests);
{% endhighlight %}

Doesn't really need annotating, the key ingredient here is `$rootScope.$watch` with just a function inside, which will run every `$digest`. Inside the callback I'm simply setting the `innerHTML` of the `$element[0]` reference to the updated count after incrementing with `count++`.
