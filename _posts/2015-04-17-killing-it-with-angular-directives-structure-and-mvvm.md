---
layout: post
permalink: /killing-it-with-angular-directives-structure-and-mvvm/
title: Killing it with Angular Directives&#59; Structure and MVVM
path: 2015-04-17-killing-it-with-angular-directives-structure-and-mvvm.md
tags:
- MVVM
- Clean code
- Directives
- Components
- Angular
---

In this post I'm going to outline my approach on writing Directives for Angular 1.x releases. There's a lot of confusion around how and why and where to do things with Directives, but they are actually very simple once you grasp the concepts and separation ideas. This post isn't going to cover nesting Directives/data flow into them from parent scopes etc, but will cover my ideal way of creating/structuring and separating all concerns in the Directive, and how to use `controller` and `link` properties correctly.

We're going to cover a basic Directive, the structure, and how things should be structured to make the best use of Angular. Let's create a pseudo "file upload" Directive to demonstrate how I'd approach.

_Note, this code will not actually work, nor do I intend it to as this is purely for demonstration purposes on structuring Directives properly._

### Structure
Following [my AngularJS style-guide](https://github.com/toddmotto/angularjs-styleguide), naturally, we'll create a basic Directive definition and pass it into Angular's `.directive()` method:

{% highlight javascript %}
function fileUpload () {
  
  return {};

}
angular
  .module('app')
  .directive('fileUpload', fileUpload);
{% endhighlight %}

Now the definition is in place, let's add some basic properties to the file upload component:

{% highlight javascript %}
function fileUpload () {
  
  return {
    restrict: 'E',
    scope: {},
    template: [
      '<div>',
      '</div>'
    ].join(''),
    controllerAs: 'vm',
    controller: function () {},
    link: function () {}
  };

}
angular
  .module('app')
  .directive('fileUpload', fileUpload);
{% endhighlight %}

This essentially completes my "Directive boilerplate", the basics I need to get something up and running.

### Controller (presentational layer)
Instead of binding the Controller (I do this with `link` as well) to the Object (`controller: fn`), I move the `controller` property's binding "out" into the main `fileUpload` function definition and assign it back to the Object. This allows me to have a bit more space, as writing functions directly to Objects I find a little less pleasing, and I can comment the functions better. It just looks like normal JavaScript rather than sticking to some "strict looking API".

Let's have a look at adding some comments and moving the functions up top.

{% highlight javascript %}
/**
 * @name fileUpload
 * @desc <file-upload> Directive
 */
function fileUpload () {

  /**
   * @name fileUploadCtrl
   * @desc File Upload Controller
   * @type {Function}
   */
  function fileUploadCtrl() {

  }

  /**
   * @name link
   * @desc File Upload Link
   * @type {Function}
   */
  function link() {

  }

  return {
    restrict: 'E',
    scope: {},
    template: [
      '<div>',
      '</div>'
    ].join(''),
    controllerAs: 'vm',
    controller: fileUploadCtrl
    link: link
  };

}
angular
  .module('app')
  .directive('fileUpload', fileUpload);
{% endhighlight %}

Sexy? Of course it is. By now you should've spotted `controllerAs: 'vm'` which I alias the Controller under `vm` (standing for ViewModel). This treats a Controller as a ViewModel for the "Presentation Model" design pattern. Read up on my [ControllerAs](http://toddmotto.com/digging-into-angulars-controller-as-syntax) article if you're not familiar. Essentially instead of injecting `$scope`, the Controller binds itself to the `$scope` under our `vm` alias, essentially creating `$scope.vm`. I treat this as a Controller "Class" and we can dive into using the `this` keyword instead of `$scope`.

{% highlight javascript %}
/**
 * @name fileUploadCtrl
 * @desc File Upload Controller
 * @type {Function}
 */
function fileUploadCtrl() {
  this.files = [];
  this.uploadFiles = function () {

  };
}
{% endhighlight %}

Using `this` is great and far better than `$scope` in my eyes, we only use `$scope` for things like `$on` events or `$watch`, treating it a little differently than our Controller Class - the "ViewModel".

We've got our function setup now, however I prefer using an "exports" style and bind all my functions and variables that way, adding any appropriate comments. Let's see what that looks like altogether:

{% highlight javascript %}
/**
 * @name fileUpload
 * @desc <file-upload> Directive
 */
function fileUpload () {

  /**
   * @name fileUploadCtrl
   * @desc File Upload Controller
   * @type {Function}
   */
  function fileUploadCtrl() {

    /**
     * @name files
     * @desc Contains all files passed in by the user
     * @type {Array}
     */
    var files = [];

    /**
     * @name uploadFiles
     * @desc Uploads our files
     * @type {Array}
     */
    function uploadFiles() {

    }

    // exports
    this.files = files;
    this.uploadFiles = uploadFiles;

  }

  /**
   * @name link
   * @desc File Upload Link
   * @type {Function}
   */
  function link() {

  }

  return {
    restrict: 'E',
    scope: {},
    template: [
      '<div>',
      '</div>'
    ].join(''),
    controllerAs: 'vm',
    controller: fileUploadCtrl
    link: link
  };

}
angular
  .module('app')
  .directive('fileUpload', fileUpload);
{% endhighlight %}

### Template integration
The next phase would be setting up some kind of file upload element with an `<input type=file>` with a model bound to it. Let's add that to our Directive with `ng-model` attributes and values, and also an "upload" button with `ng-change` (yes we could make this a form with `ng-submit` but let's keep it simple).

{% highlight javascript %}
// ...
template: [
  '<div>',
    '<input type="file" ng-model="vm.files">',
    '<button type="button" ng-change="vm.uploadFiles(vm.files);">',
  '</div>'
].join('')
// ...
{% endhighlight %}

Let's add some comments to our pseudo Controller to see how we'd handle the upload. Note I've added `UploadService` (great name) to the `fileUploadCtrl` arguments to be dependency injected.

{% highlight javascript %}
/**
 * @name fileUploadCtrl
 * @desc File Upload Controller
 * @type {Function}
 */
function fileUploadCtrl(UploadService) {

  /**
   * @name files
   * @desc Contains all files passed in by the user
   * @type {Array}
   */
  var files = [];

  /**
   * @name uploadFiles
   * @desc Uploads our files
   * @type {Array}
   */
  function uploadFiles(files) {
    // hand off our files to a Service
    UploadService
    .uploadFiles(files)
    .then(function (response) {
      // success, we could get our file Object back
      // and render it in the View for the user
      // maybe some ng-repeat with a list of files inside
    }, function (reason) {
      // error stuff if not handled globally
    })
  }

  // exports
  this.files = files;
  this.uploadFiles = uploadFiles;

}
{% endhighlight %}

Wait, not a lot has changed, why not? Let's look at why.

### Services (business logic layer)
Anything that communicates with an API, such as posting files off to a backend should never be done in a Controller, I repeat _NEVER_! Why? Separation of concerns. Of course we could do it, but that makes our lives harder as Controllers are to be treated as ViewModels, not ViewModelBusinessLogicThings.

I'm not going to write some pseudo code for our Service, but understanding why it's passed into our Controller to abstract business logic is highly important to getting your Directive structures and dependencies manageable and scalable from the beginning.

All a Service should do is provide us with necessary Model data for our Controller (ViewModel) to make a copy of, and present to the user how we see fit.

### Link function (DOM layer)
Directives are fantastic as they offer us a door into our Directives that shouldn't be handled in the presentational logic layer (Controller) or a business logic layer (Service) - that thing we call the DOM (Document Object Model). We need the DOM sometimes, and Angular gives it to us on a plate.

Our file upload Directive wouldn't be complete without some drag and drop funk, so we'll use the DOM events provided to us, `dragover, drop` etc. First let's add a `<div class="drop-zone">` to our Directive, which will serve as our "drag and drop" area.

{% highlight javascript %}
// ...
template: [
  '<div>',
    '<input type="file" ng-model="vm.files">',
    '<button type="button" ng-change="vm.uploadFiles(vm.files);">',
    '<div class="drop-zone">Drop your files here!</div>',
  '</div>'
].join('')
// ...
{% endhighlight %}

So, now we need to tie into our Directive. Our `link` function comes in handy here, I've also injected `$scope, $element, $attrs` (I like dollar-prefixing, sorry, `iAttrs` just makes me cry).

We'll need to bind our special event listeners to our `.drop-zone` element. Remember, we want to keep our `link` functions as light as possible, I rarely augment `$scope` in them and neither should you.

Adding in some event listeners on our element:

{% highlight javascript %}
/**
 * @name link
 * @desc File Upload Link
 * @type {Function}
 */
function link($scope, $element, $attrs) {
  var drop = $element.find('.drop-zone')[0];
  drop.addEventListener('dragenter', function(e) {
    // do something on "dragenter"
  }, false);
  drop.addEventListener("dragleave", function(e) {
    // do something on "dragleave"
  }, false);
  drop.addEventListener("dragover", function(e) {
    // do something on "dragover"
  }, false);
  drop.addEventListener('drop', function(e) {
    // do something on "drop"
  }, false);
}
{% endhighlight %}

Again, I'd comment and abstract these into a better, cleaner view, I don't need to provide `dragenter`, `dragleave` or `dragover` support for this demo, so we'll drop those:

{% highlight javascript %}
/**
 * @name link
 * @desc File Upload Link
 * @type {Function}
 */
function link($scope, $element, $attrs) {

  /**
   * @name drop
   * @desc Drop zone element
   * @type {Element}
   */
  var drop = $element.find('.drop-zone')[0];

  /**
   * @name onDrop
   * @desc Callback on "drop" event
   * @type {Function}
   * @param {Event} e Event passed in to grab files from
   */
  function onDrop(e) {
    
  }
  
  // events
  drop.addEventListener('drop', onDrop, false);

}
{% endhighlight %}

So what now, we've got our event listener setup, we can grab our files from `e.dataTransfer.files` and pass them off to our upload API, but we want to use the same function as in our Controller, the `uploadFiles` method.

We can pass our Controller _into_ the Directive itself, I use a `$ctrl` alias just to keep it short and sweet, but this gives us access to our functions.

{% highlight javascript %}
/**
 * @name link
 * @desc File Upload Link
 * @type {Function}
 */
function link($scope, $element, $attrs, $ctrl) {

  /**
   * @name drop
   * @desc Drop zone element
   * @type {Element}
   */
  var drop = $element.find('.drop-zone')[0];

  /**
   * @name onDrop
   * @desc Callback on "drop" event
   * @type {Function}
   * @param {Event} e Event passed in to grab files from
   */
  function onDrop(e) {
    if (e.dataTransfer && e.dataTransfer.files) {
      $ctrl.uploadFiles(e.dataTransfer.files);
    }
  }
  
  // events
  drop.addEventListener('drop', onDrop, false);

}
{% endhighlight %}

Smashing! Code reuse, using our Controller's `uploadFiles` method to pass it back into our API, this will then make any presentational logic changes alongside it, as previously mentioned we might show the uploaded file(s) to the user, so this would all be reused and handled in the Controller.

But wait, it won't work just yet... We forgot the magic line `$scope.$apply()`:

{% highlight javascript %}
/**
 * @name onDrop
 * @desc Callback on "drop" event
 * @type {Function}
 * @param {Event} e Event passed in to grab files from
 */
function onDrop(e) {
  if (e.dataTransfer && e.dataTransfer.files) {
    $ctrl.uploadFiles(e.dataTransfer.files);
    // force a $digest cycle
    $scope.$apply();
  }
}
{% endhighlight %}

The `$digest` cycle will then be kicked off so Angular can run our code and update our application with any data that's changed. We need to do this as the `drop` event listener is outside of Angular's ecosystem and it never knew the event took place, so we have to tell it something's happened.

All together now:

{% highlight javascript %}
/**
 * @name fileUpload
 * @desc <file-upload> Directive
 */
function fileUpload () {

  /**
   * @name fileUploadCtrl
   * @desc File Upload Controller
   * @type {Function}
   */
  function fileUploadCtrl(UploadService) {

    /**
     * @name files
     * @desc Contains all files passed in by the user
     * @type {Array}
     */
    var files = [];

    /**
     * @name uploadFiles
     * @desc Uploads our files
     * @type {Array}
     */
    function uploadFiles(files) {
      // hand off our files to a Service
      UploadService
      .uploadFiles(files)
      .then(function (response) {
        // success, we could get our file Object back
        // and render it in the View for the user
        // maybe some ng-repeat with a list of files inside
      }, function (reason) {
        // error stuff if not handled globally
      })
    }

    // exports
    this.files = files;
    this.uploadFiles = uploadFiles;

  }

  /**
   * @name link
   * @desc File Upload Link
   * @type {Function}
   */
  function link($scope, $element, $attrs, $ctrl) {

    /**
     * @name drop
     * @desc Drop zone element
     * @type {Element}
     */
    var drop = $element.find('.drop-zone')[0];

    /**
     * @name onDrop
     * @desc Callback on "drop" event
     * @type {Function}
     * @param {Event} e Event passed in to grab files from
     */
    function onDrop(e) {
      if (e.dataTransfer && e.dataTransfer.files) {
        $ctrl.uploadFiles(e.dataTransfer.files);
        // force a $digest cycle
        $scope.$apply();
      }
    }
    
    // events
    drop.addEventListener('drop', onDrop, false);

  }

  return {
    restrict: 'E',
    scope: {},
    template: [
      '<div>',
        '<input type="file" ng-model="vm.files">',
        '<button type="button" ng-change="vm.uploadFiles(vm.files);">',
        '<div class="drop-zone">Drop your files here!</div>',
      '</div>'
    ].join(''),
    controllerAs: 'vm',
    controller: fileUploadCtrl
    link: link
  };

}
angular
  .module('app')
  .directive('fileUpload', fileUpload);
{% endhighlight %}

### Recapping and MVVM (Model-View-ViewModel)
This approach allows the Controller to be used as a ViewModel, and to use the `link` function to properly deal with DOM manipulation whilst making easy work of communicating back to our Controller. The approach also promotes better separation of concerns, as well as separation of the code itself, such as breaking the functions out and assigning them, rather than nesting under another layer of code (such as inside an Object).

Any suggestions/improvements feel free to make changes/create issues [on GitHub](https://github.com/toddmotto/toddmotto.github.io). Enjoy!
