---
layout: post
permalink: /angular-js-dependency-injection-annotation-process/
title: Angular’s dependency injection annotation process
path: 2014-07-11-angular-js-dependency-injection-annotation-process.md
tags:
- Style
- Performance
- Angular
---

For those familiar with Angular's dependency injection (DI), this post will dive into some of the "under the hood" methods that power the DI process. For those who aren't familiar with the concept, DI is a way of asking for necessary dependencies instead of passing them. We merely tell a method, function, object what dependencies we want - and tucked away we get our dependencies given to us.

So, how does this work in AngularJS?

Until now, I've kind of let it be a black box - it just works and I use it. But after a stint of looking through the source code, I think I've cracked how it all works underneath - and the implementation is _so_ clever and simple - it's definitely worth looking into. I've also learned a few things along the way!

### Dependency injection in Angular

There are three ways we can pass dependencies into Angular functions, the first cannot be minified without breaking the app, the second annotations aid minification aliasing - let's look at them.

#### 1. Function arguments
Function arguments work perfectly until we minify our app. Minification is important as we want to compress our JavaScript as much as possible, for obfuscation but mainly for performance purposes.

Let's see how that works:

{% highlight javascript %}
function SomeCtrl ($scope) {
  // do something with $scope
}

angular
  .module('app', [])
  .controller('SomeCtrl', SomeCtrl);
{% endhighlight %}

Easy, we just pass in `$scope` as an argument and Angular passes it in for us.

#### 2: Array arguments

Possibly the most common way to do it, we pass in an Array to an Angular module method, we'll use a Controller in all examples to keep things easy and consistent.

{% highlight javascript %}
function SomeCtrl ($scope) {
  // do something with $scope
}

angular
  .module('app', [])
  .controller('SomeCtrl', ['$scope', SomeCtrl]);
{% endhighlight %}

This is also commonly seen as this, however, with functions inside the `.controller()` method:

{% highlight javascript %}
angular
  .module('app', [])
  .controller('SomeCtrl', ['$scope', function SomeCtrl ($scope) {
    // do something with $scope
  }]);
{% endhighlight %}

[John Papa](//twitter.com/John_Papa) and I have been discussing the first pattern in depth recently and advocate its use over the latter, that's another story though.

#### 3. $inject

Using `$inject` makes things simple, and does the same as the above by annotating the dependencies:

{% highlight javascript %}
function SomeCtrl ($scope) {
  // do something with $scope
}

SomeCtrl.$inject = ['$scope'];

angular
  .module('app', [])
  .controller('SomeCtrl', SomeCtrl);
{% endhighlight %}

##### What does this mean though?

It means when our application is minified, dependency references remain intact due to Angular's internal annotations, for example `function ($scope) {}` becoming `function (a) {}` . Visual example:

{% highlight javascript %}
function SomeCtrl (a) {
  // a = $scope
}

SomeCtrl.$inject = ['$scope'];

angular
  .module('app', [])
  .controller('SomeCtrl', SomeCtrl);
{% endhighlight %}

Angular magically maps this `['$scope']` dependency onto the local `a` argument variable - how does it know?!...

### Annotation process

This is where things get seriously awesome, we're out of API land now and deep within Angular's source code. How does Angular map our dependencies then onto minified variables? Let's find out.

First, there's a bunch of `RegExp` variables in the source, these play a part in it ;)

{% highlight javascript %}
var FN_ARGS = /^function\s*[^\(]*\(\s*([^\)]*)\)/m;
var FN_ARG_SPLIT = /,/;
var FN_ARG = /^\s*(_?)(\S+?)\1\s*$/;
var STRIP_COMMENTS = /((\/\/.*$)|(\/\*[\s\S]*?\*\/))/mg;
{% endhighlight %}

There's a bunch of steps that each of these RegExps correspond with, let's follow them:

Angular turns a function it needs to annotate into a `String`, then reads what you've passed in and changes that into an Array of arguments and returns it. First, it calls the `toString` method on the function - I'm going to recreate the bare basics of this - it won't be a direct copy of the source code.

{% highlight javascript %}
// declare our function
function ExampleMethod ($scope, /* $rootScope needed! */$rootScope, SomeService) {
  
}

// call toString on it
var fnString = ExampleMethod.toString();

// "function ExampleMethod($scope, /* $rootScope needed! */$rootScope, SomeService) {}"
console.log(fnString);
{% endhighlight %}

What now? Someone's put a comment in the arguments (totally cool) but Angular needs to do some more work on this. Enter, the `STRIP_COMMENTS` constant:

{% highlight javascript %}
// "function ExampleMethod($scope, $rootScope, SomeService) {}"
fnString.replace(STRIP_COMMENTS, '');
{% endhighlight %}

With the comments gone, Angular moves onto the next phase and calls `String.prototype.match`:

{% highlight javascript %}
/*
  [
    "function ExampleMethod($scope, $rootScope, SomeService) {}",
    "$scope, $rootScope, SomeService"
  ]
*/
fnString.match(FN_ARGS);
{% endhighlight %}

We then get the above `Array` as the output. We don't care about the first part, we just want the comma separated second item in the `Array`, which means we would do this instead to get just the `String` value of the second item:

{% highlight javascript %}
/*
  "$scope, $rootScope, SomeService"
*/
fnString.match(FN_ARGS)[1];
{% endhighlight %}

Next, Angular calls `fnString.split(FN_ARG_SPLIT);`, which returns an `Array` of our `arguments`:

{% highlight javascript %}
// ["$scope", " $rootScope", " SomeService"]
fnString.split(FN_ARG_SPLIT);
{% endhighlight %}

You'll notice that some of the `Array` items have a single space before the name, such as `" $rootScope"`, this is because the function arguments were separated with a comma then space. Angular then loops over these final `Array` items, replaces function arg whitespace and pushes them into a _new_ `Array` called `$inject`:

{% highlight javascript %}
var $inject = [];
var args = fnString.split(FN_ARG_SPLIT);
args.forEach(function (arg) {
  arg.replace(FN_ARG, function (all, underscore, name) {
    $inject.push(name);
  });
});
return $inject;
{% endhighlight %}

The function returns `$inject`. And we're done. Putting the above together, I can replicate what Angular does (in a very simple form):

{% highlight javascript %}
var FN_ARGS = /^function\s*[^\(]*\(\s*([^\)]*)\)/m;
var FN_ARG_SPLIT = /,/;
var FN_ARG = /^\s*(_?)(\S+?)\1\s*$/;
var STRIP_COMMENTS = /((\/\/.*$)|(\/\*[\s\S]*?\*\/))/mg;

function annotate (fn) {
  var $inject = [];
  fn = fn.toString();
  var first = fn.replace(STRIP_COMMENTS, '');
  var second = first.match(FN_ARGS)[1];
  var third = second.split(FN_ARG_SPLIT);
  third.forEach(function (arg) {
    arg.replace(FN_ARG, function (all, underscore, name) {
      $inject.push(name);
    });
  });
  return $inject;
}
{% endhighlight %}

Nice! Here's how it works:

{% highlight javascript %}
function ExampleMethod ($scope, /* $rootScope needed! */$rootScope, SomeService) {
  
}
var annotated = annotate(ExampleMethod);

// ["$scope", "$rootScope", "SomeService"]
console.log(annotated);
{% endhighlight %}

This only takes into account arguments being named correctly, such as `$scope` instead of minified `a`. This means that the above method is only for annotating functions that haven't been minified or have no Array dependencies, in other words, this code isn't used in production if you're minifying your code.

#### Annotating the Array syntax

This is actually really simple! Take our `Array` setup, I can pass it into my `annotate` function:

{% highlight javascript %}
function ExampleMethod ($scope, /* $rootScope needed! */$rootScope, SomeService) {
  
}
var arraySyntax = ['$scope', '$rootScope', 'SomeService', ExampleMethod];
var annotated = annotate(arraySyntax);

// ["$scope", "$rootScope", "SomeService"]
console.log(annotated);
{% endhighlight %}

If we're already this example, then Angular doesn't need to grab the names as above, it assumes you've already supplied them correctly and uses your `Array` order as the new arguments order, using this line in the source:

{% highlight javascript %}
...
else if (isArray(fn)) {
  last = fn.length - 1;
  assertArgFn(fn[last], 'fn');
  $inject = fn.slice(0, last);
}
...
{% endhighlight %}

We're only interested in this part:

{% highlight javascript %}
...
else if (isArray(fn)) {
  last = fn.length - 1;
  $inject = fn.slice(0, last);
}
...
{% endhighlight %}

What does it do? The `last` variable holds the `Array.length - 1`, which (very cleverly) the last item in our `Array` is the function to be invoked (the `arraySyntax` variable I just created above). We could write a _tiny_ function to do exactly that:

{% highlight javascript %}
function annotate (fn) {
  var $inject = [];
  $inject = fn.slice(0, fn.length - 1);
  return $inject;
}
{% endhighlight %}

Or even:

{% highlight javascript %}
function annotate (fn) {
  return fn.slice(0, fn.length - 1);
}
{% endhighlight %}

Both would output what we need, as `Array.prototype.slice` returns a portion of an `Array`, which Angular says to be the length of the `Array`, minus `1` - omitting our `function`:

{% highlight javascript %}
function annotate (fn) {
  return fn.slice(0, fn.length - 1);
}

// returns ["$scope", "$rootScope", "SomeService"]
var annotated = annotate([
  '$scope',
  '$rootScope',
  'SomeService',
  function ExampleMethod ($scope, $rootScope, SomeService) {}
]);

{% endhighlight %}

#### Annotating $inject

Inside Angular's annotate function, it also checks to see if `fn.$inject` exists, if it does, it can just return those, as we'll have provided a setup like this:

{% highlight javascript %}
function ExampleMethod ($scope, $rootScope, SomeService) {
  
}
ExampleMethod.$inject = ['$scope', '$rootScope', 'SomeService'];
{% endhighlight %}

`ExampleMethod.$inject` is a straight up `Array`, and Angular will run with that.

### Mapping function arguments

This took some further digging, and the way Angular maps minified function arguments to real names is _so_ unbelievably simple! The steps are:

* Get the dependency list in Array form (as we have above)
* Apply the dependencies list over the top of the minified variables (pass them in as arguments when invoking the function)

We've already completed the first step, let's assume we have `['$scope', '$rootScope', 'SomeService']` on standby from an `$inject` property on the function, and the said function with minified arguments:

{% highlight javascript %}
function ExampleMethod (a, b, c) {
  
}
ExampleMethod.$inject = ['$scope', '$rootScope', 'SomeService'];
{% endhighlight %}

How does `$scope` match up with `a`? Easy:

{% highlight javascript %}
function ExampleMethod (a, b, c) {
  console.log(a, b, c);
}
ExampleMethod.$inject = ['$scope', '$rootScope', 'SomeService'];
ExampleMethod.apply(null, ExampleMethod.$inject);
{% endhighlight %}

Three lines to completely remap the underlying argument names. As they're just placeholder arguments anyway (which Angular has read and passed in what we actually want, there isn't really "remapping" going on, but it feels like it). This is just a simple case of invoking the function based off a predefined arguments list.

Angular doesn't do much with the `$inject` property, it just gets returned from the function (after a few function checks, no manipulation), so I've left it out of my remade `annotate` function as it'll just get returned as is.

### Altogether now

Let's slap it all together. 

{% highlight javascript %}
/**
 * annotate remake
 */
var FN_ARGS = /^function\s*[^\(]*\(\s*([^\)]*)\)/m;
var FN_ARG_SPLIT = /,/;
var FN_ARG = /^\s*(_?)(\S+?)\1\s*$/;
var STRIP_COMMENTS = /((\/\/.*$)|(\/\*[\s\S]*?\*\/))/mg;

function annotate (fn) {
    var $inject;
    if (typeof fn === 'function') {
        $inject = [];
        fn = fn.toString();
        var first = fn.replace(STRIP_COMMENTS, '');
        var second = first.match(FN_ARGS)[1];
        var third = second.split(FN_ARG_SPLIT);
        third.forEach(function (arg) {
            arg.replace(FN_ARG, function(all, underscore, name){
                $inject.push(name);
              });
        });
    } else {
        $inject = fn.slice(0, fn.length - 1);
    }
    return $inject;
}

/**
 * functions
 */
function withoutDependencies ($scope, $rootScope, SomeFactory) {
  // $scope, $rootScope, SomeFactory
  console.log($scope, $rootScope, SomeFactory);
}
function withDependencies (a, b, c) {
  // BOOM! We did it!
  // $scope, $rootScope, SomeFactory
  console.log(a, b, c);
}

var annotatedWithout = annotate(withoutDependencies);
withoutDependencies.apply(null, annotatedWithout);

var annotatedWith = annotate(['$scope', '$rootScope', 'SomeFactory', withDependencies]);
withDependencies.apply(null, annotatedWith);
{% endhighlight %}

The above `annotate` function invokes the `withoutDependencies` function without any dependencies, so `annotate` loops through the arguments and invokes it with each argument name directly. The `withDependencies` function is looking for `a, b, c` arguments, but what are they? We grab `$scope, $rootScope, SomeFactory` from the `$inject` property and use `Function.prototype.apply` to invoke the function with our `$inject` Array of arguments.

That's it! This doesn't cover how Angular physically "injects" the actual dependencies, but how it resolves the dependency names internally, I thought it was really clever, and learned a thing or two from the source. Here's the end result in a [jsFiddle](http://jsfiddle.net/toddmotto/fgFWL), inspect the `console` to see it working.

### Best route for performance?

Based on the following conditions in the [Angular source](https://code.angularjs.org/1.3.0-beta.7/angular.js), using `$inject` would be best for performance! I've annotated the good parts of the function:

{% highlight javascript %}
function annotate(fn, strictDi, name) {
  var $inject,
      fnText,
      argDecl,
      last;
 
  // if the function passed in is a function, let's roll with it...
  // ...there are two things that can happen inside here, either we're using $inject or
  // we aren't using $inject, which means we'll manually need to get the dependencies
  // from the arguments
  if (typeof fn === 'function') {

    // !($inject = fn.$inject) checks for the presence
    // of fn.$inject and assigns it to the $inject variable, if this is true
    // there is no further action needed, and $inject is returned
    // which is a very fast operation
    if (!($inject = fn.$inject)) {

      // if fn.$inject doesn't exist, it's bad news and we're going to need to
      // do some manual work reading the dependencies from the arguments, so they
      // need to be spelled correctly against the proper names or they'll be an
      // unknown provider
      $inject = [];
      if (fn.length) {
        if (strictDi) {
          if (!isString(name) || !name) {
            name = fn.name || anonFn(fn);
          }
          throw $injectorMinErr('strictdi',
            '{0} is not using explicit annotation and cannot be invoked in strict mode', name);
        }
        fnText = fn.toString().replace(STRIP_COMMENTS, '');
        argDecl = fnText.match(FN_ARGS);
        forEach(argDecl[1].split(FN_ARG_SPLIT), function(arg){
          arg.replace(FN_ARG, function(all, underscore, name){
            $inject.push(name);
          });
        });
      }
      // here after we're done, it completes what we wanted before this 
      // operation, the fn.$inject, so it just assigns $inject to the function
      // and $inject gets returned below
      fn.$inject = $inject;
    }
  } else if (isArray(fn)) {
    // if the function isn't a function, but is an Array containing a function
    // we need to remove it from the Array and send the leftover portion of the
    // Array back as $inject

    // calculate the Array length
    last = fn.length - 1;
    assertArgFn(fn[last], 'fn');

    // use slice which returns the portion of the Array minus the final item
    $inject = fn.slice(0, last);
  } else {
    assertArgFn(fn, 'fn', true);
  }
  // returns $inject from one of the methods above!
  return $inject;
}
{% endhighlight %}

Want to dig in more? Search for this `function annotate` to find this implementation, and `function invoke` to see where the `.apply()` call takes place and how the two hang together.
