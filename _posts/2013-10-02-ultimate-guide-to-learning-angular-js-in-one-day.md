---
layout: post
permalink: /ultimate-guide-to-learning-angular-js-in-one-day/
title: Ultimate guide to learning AngularJS in one day
path: 2013-10-02-ultimate-guide-to-learning-angular-js-in-one-day.md
tags:
- Tutorial
- MVVM
- Components
- Directives
- Angular
- JavaScript
---

<h5>Mastered this? Try my <a href="//toddmotto.com/opinionated-angular-js-styleguide-for-teams" style="text-decoration: underline;">Opinionated AngularJS styleguide for teams</a></h5>

### What is AngularJS?
Angular is a client-side MVC/MVVM framework built in JavaScript, essential for modern single page web applications (and even websites). This post is a full end to end crash course from my experiences, advice and best practices I've picked up from using it.

### Terminology
Angular has an initial short learning curve, you'll find it's up and down after mastering the basics. It's mainly getting to grips with the terminology and "thinking MVC". MVC stands for _Model-View-Controller_. Here are the higher level and essential APIs that Angular comes with, and some terminology.

#### MVC
You've probably heard of MVC, used in many programming languages as a means of structuring/architecting applications/software. Here's a quick breakdown of meanings:

- _Model_: the data structure behind a specific piece of the application, usually ported in JSON. Read up on JSON before getting started with Angular as it's essential for communicating with your server and view. For instance a group of _User IDs_ could have the following model:

{% highlight javascript %}
{
  "users" : [{
    "name": "Joe Bloggs",
    "id": "82047392"
  },{
    "name": "John Doe",
    "id": "65198013"
  }]
}
{% endhighlight %}

You'll then grab this information either from the server via an XHR (XMLHttp Request), in jQuery you'll know this as the _$.ajax _method, and Angular wraps this in _$http_, or it'll be written into your code whilst the page is parsing (from a datastore/database). You can then push updates to your model and send it back.

- _View_: The view is simple, it's your HTML and/or rendered output. Using an MVC framework, you'll pull down Model data which updates your View and displays the relevant data in your HTML.

- _Controller_: Do what they say on the tin, they control things. But what things? Data. Controllers are your direct access from the _server_ to the _view_, the middle man, so you can update data on the fly via comms with the server and the client.

#### Setting up an AngularJS project (bare essentials)

First, we need to actually setup the essentials to an Angular project. There are certain things to note before we begin, which generally consist of an _ng-app_ declaration to define your app, a _Controller_ to talk to your view, and some DOM binding and inclusion of Angular. Here are the bare essentials:

Some HTML with _ng-*_ declarations:
{% highlight html %}
<div ng-app="myApp">
    <div ng-controller="MainCtrl">
        <!-- controller logic -->
    </div>
</div>
{% endhighlight %}

An Angular Module and Controller:
{% highlight javascript %}
var myApp = angular.module('myApp', []);

myApp.controller('MainCtrl', ['$scope', function ($scope) {
  // Controller magic
}]);
{% endhighlight %}

Before jumping in, we need to create an _Angular module_ which all our logic will bolt onto. There are many ways to declare modules, and you can chain all of your logic like this (I don't like this method):

{% highlight javascript %}
angular.module('myApp', [])
.controller('MainCtrl', ['$scope', function ($scope) {...}])
.controller('NavCtrl', ['$scope', function ($scope) {...}])
.controller('UserCtrl', ['$scope', function ($scope) {...}]);
{% endhighlight %}

Setting up a global Module has proved to be the best for Angular projects I've worked on. The lack of semi-colons and accidental closing of the 'chain' proved counter-productive and frequented unnecessary compiling errors. Go for this:

{% highlight javascript %}
var myApp = angular.module('myApp', []);
myApp.controller('MainCtrl', ['$scope', function ($scope) {...}]);
myApp.controller('NavCtrl', ['$scope', function ($scope) {...}]);
myApp.controller('UserCtrl', ['$scope', function ($scope) {...}]);
{% endhighlight %}

Each new file I create simply grabs the _myApp_ namespace and automatically bolts itself into the application. Yes, I'm creating new files for each Controller, Directive, Factory and everything else (you'll thank me for this). Concatenate and minify them all and push the single script file into the DOM (using something like Grunt/Gulp).

#### Controllers
Now you've grasped the concept of MVC and a basic setup, let's check out Angular's implementation on how you can get going with Controllers.

Taking the example from above, we can take a baby step into pushing some data into the DOM from a controller. Angular uses a templating-style _{% raw %}{{ handlebars }}{% endraw %}_ syntax for talking to your HTML. Your HTML should (ideally) contain no physical text or hard coded values to make the most of Angular. Here's an example of pushing a simple String into the DOM:

{% highlight html %}
<div ng-app="myApp">
    <div ng-controller="MainCtrl">
         {% raw %}{{ text }}{% endraw %}
    </div>
</div>
{% endhighlight %}

{% highlight javascript %}
var myApp = angular.module('myApp', []);

myApp.controller('MainCtrl', ['$scope', function ($scope) {
    
    $scope.text = 'Hello, Angular fanatic.';
    
}]);
{% endhighlight %}

And the live output:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/mN7QB/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

The key rule concept here is the _$scope_ concept, which you'll tie to all your functions inside a specific controller. The _$scope_ refers to the current element/area in the DOM (no, not the same as _this_), and encapsulates a very clever scoping capability that keeps data and logic completely scoped inside elements. It brings JavaScript public/private scoping to the DOM, which is fantastic.

The _$scope_ concept may seem scary at first, but it's your way into the DOM from the server (and static data if you have that too)! The demo gives you a basic idea of how you can 'push' data to the DOM.

Let's look at a more representative data structure that we've hypothetically retrieved from the server to display a user's login details. For now I'll use static data; I'll show you how to fetch dynamic JSON data later.

First we'll setup the JavaScript:

{% highlight javascript %}
var myApp = angular.module('myApp', []);

myApp.controller('UserCtrl', ['$scope', function ($scope) {
    
    // Let's namespace the user details
    // Also great for DOM visual aids too
    $scope.user = {};
    $scope.user.details = {
      "username": "Todd Motto",
      "id": "89101112"
    };
    
}]);
{% endhighlight %}

Then port it over to DOM to display this data:

{% highlight html %}
<div ng-app="myApp">
    <div ng-controller="UserCtrl">
        <p class="username">Welcome, {% raw %}{{ user.details.username }}{% endraw %}</p>
        <p class="id">User ID: {% raw %}{{ user.details.id }}{% endraw %}</p>
    </div>
</div>
{% endhighlight %}

Output:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/425KU/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

It's important to remember that Controllers are for _data_ only, and creating functions (event functions too!) that talk to the server and push/pull JSON data. No DOM manipulation should be done here, so put your jQuery toolkit away. Directives are for DOM manipulation, and that's up next.

Protip: throughout the Angular documentation (at the time of writing this) their examples show this usage to create Controllers:

{% highlight javascript %}
var myApp = angular.module('myApp', []);

function MainCtrl ($scope) {
  //...
};
{% endhighlight %}

... DON'T do this. This exposes all your functions to the global scope and doesn't keep them tied in very well with your app. This also means that you can't minify your code or run tests very easily. Don't pollute the global namespace and keep your controllers _inside_ your app.

#### Directives

A directive ([checkout my post on Directives from existing scripts/plugins](//toddmotto.com/creating-an-angularjs-directive-from-one-of-your-existing-plugins-scripts)) in its simplest form is a small piece of templated HTML, preferably used multiple times throughout an application where needed. It's an easy way to inject DOM into your application with no effort at all, or perform custom DOM interactions. Directives are not simple at all; there is an incredible learning curve to fully conquering them, but this next phase will let you hit the ground running.

So what are directives useful for? A lot of things, including DOM components, for example tabs or navigation elements - really depends on what your app makes use of in the UI. Let me put it this way, if you've toyed with _ng-show_ or _ng-hide_, those are directives (though they don't inject DOM).

For this exercise, I'm going to keep it really simple and create a custom type of button (called _customButton_) that injects some markup that I hate to keep typing out. There are various ways to define Directives in the DOM, these could look like so:

{% highlight html %}
<!-- 1: as an attribute declaration -->
<a custom-button>Click me</a>

<!-- 2: as a custom element -->
<custom-button>Click me</custom-button>

<!-- 3: as a class (used for old IE compat) -->
<a class="custom-button">Click me</a>

<!-- 4: as a comment (not good for this demo, however) -->
<!-- directive: custom-button -->
{% endhighlight %}

I prefer using them as an attribute, custom elements are coming in the future of HTML5 under the Web Components, but Angular report these quite buggy in some older browsers.

Now you know how to declare where Directives are used/injected, let's create this custom button. Again, I'll hook into my global namespace _myApp_ for the application, this is a directive in its simplest form:

{% highlight javascript %}
myApp.directive('customButton', function () {
  return {
    link: function (scope, element, attrs) {
      // DOM manipulation/events here!
    }
  };
});
{% endhighlight %}

I define my directive using the _.directive()_ method, and pass in the directive's name 'customButton'. When you capitalise a letter in a Directive's name, its use case is then split via a hyphen in the DOM (as above).

A directive simply returns itself via an Object and takes a number of parameters. The most important for me to master first are, _restrict_, _replace_, _transclude_, _template_ and _templateUrl_, and of course the _link_ property. Let's add those others in:

{% highlight javascript %}
myApp.directive('customButton', function () {
  return {
    restrict: 'A',
    replace: true,
    transclude: true,
    template: '<a href="" class="myawesomebutton" ng-transclude>' +
                '<i class="icon-ok-sign"></i>' +
              '</a>',
    link: function (scope, element, attrs) {
      // DOM manipulation/events here!
    }
  };
});
{% endhighlight %}

Output:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/VC4H2/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Make sure you _Inspect Element_ to see the additional markup injected. Yes, I know, there is no icon included as I never included Font Awesome, but you see how it works. Now for the Directive properties explanations:

- _restrict_: This goes back to usage, how do we restrict the element's usage? If you're using a project that needs legacy IE support, you'll probably need attribute/class declarations. Restricting as 'A' means you restrict it as an _Attribute_. 'E' for _Element_, 'C' for _Class_ and 'M' for _Comment_. These have a default as 'EA'. Yes, you can restrict to multiple use cases.
- _replace_: This replaces the markup in the DOM that defines the directive, used in the example, you'll notice how initial DOM is replaced with the Directive's template.
- _transclude_: Put simply, using transclude allows for existing DOM content to be copied into the directive. You'll see the words 'Click me' have 'moved' into the Directive once rendered.
- _template_: A template (as above) allows you to declare markup to be injected. It's a good idea to use this for tiny pieces of HTML only. Injected templates are all compiled through Angular, which means you can declare the handlebar template tags in them too for binding. 
- _templateUrl_: Similar to a template, but kept in its own file _or_ &lt;script&gt; tag. You can do this to specify a template URL, which you'll want to use for manageable chunks of HTML that require being kept in their own file, just specify the path and filename, preferably kept inside their own _templates_ directory:

{% highlight javascript %}
myApp.directive('customButton', function () {
  return {
    templateUrl: 'templates/customButton.html'
    // directive stuff...
  };
});
{% endhighlight %}

And inside your file (filename isn't sensitive at all):

{% highlight html %}
<!-- inside customButton.html -->
<a href="" class="myawesomebutton" ng-transclude>
  <i class="icon-ok-sign"></i>
</a>
{% endhighlight %}

What's really good about doing this, is the browser will actually _cache_ the HTML file, bravo! The other alternative which isn't cached is declaring a template inside &lt;script&gt; tags:

{% highlight html %}
<script type="text/ng-template" id="customButton.html">
<a href="" class="myawesomebutton" ng-transclude>
  <i class="icon-ok-sign"></i>
</a>
</script>
{% endhighlight %}

You'll tell Angular that it's an _ng-template_ and give it the ID. Angular will look for the _ng-template_ or the _*.html_ file, so whichever you prefer using. I prefer creating _*.html_ files as they're very easy to manage, boost performance and keep the DOM very clean, you could end up with 1 or 100 directives, you want to be able to navigate through them easily.

#### Services
Services are often a confusing point. From experience and research, they're more a stylistic design pattern rather than providing _much_ functional difference. After digging into the Angular source, they look to run through the same compiler and they share a lot of functionality. From my research, you should use Services for _singletons_, and Factories for more complex functions such as Object Literals and more complicated use cases.

Here's an example Service that multiples two numbers:

{% highlight javascript %}
myApp.service('Math', function () {
  this.multiply = function (x, y) {
    return x * y;
  };
});
{% endhighlight %}

You would then use this inside a Controller like so:

{% highlight javascript %}
myApp.controller('MainCtrl', ['$scope', function ($scope) {
    var a = 12;
    var b = 24;

    // outputs 288
    var result = Math.multiply(a, b);
}]);
{% endhighlight %}

Yes, multiplication is very easy and doesn't need a Service, but you get the gist.

When you create a Service (or Factory) you'll need to use dependency injection to tell Angular it needs to grab hold of your new Service - otherwise you'll get a compile error and your Controller will break. You may have noticed the _function ($scope)_ part inside the Controller declaration by now, and this is simple dependency injection. Feed it the code! You'll also notice _['$scope']_ before the _function ($scope)_ too, I'll come onto this later. Here's how to use dependency injection to tell Angular you need your service:

{% highlight javascript %}
// Pass in Math
myApp.controller('MainCtrl', ['$scope', 'Math', function ($scope, Math) {
    var a = 12;
    var b = 24;

    // outputs 288
    var result = Math.multiply(a, b);
}]);
{% endhighlight %}

#### Factories
Coming from Services to Factories should be simple now, we could create an Object Literal inside a Factory or simply provide some more in-depth methods:

{% highlight javascript %}
myApp.factory('Server', ['$http', function ($http) {
  return {
    get: function(url) {
      return $http.get(url);
    },
    post: function(url) {
      return $http.post(url);
    },
  };
}]);
{% endhighlight %}

Here I'm creating a custom wrapper for Angular's XHR's. After dependency injection into a Controller, the usage is simple:

{% highlight javascript %}
myApp.controller('MainCtrl', ['$scope', 'Server', function ($scope, Server) {
    var jsonGet = '//myserver/getURL';
    var jsonPost = '//myserver/postURL';
    Server.get(jsonGet);
    Server.post(jsonPost);
}]);
{% endhighlight %}

If you wanted to Poll the server for changes, you could then setup a _Server.poll(jsonPoll)_, or perhaps if you're using a Socket you could setup _Server.socket(jsonSocket)_. It opens up doors to modularising code as well as creating tools for you to use and keep code inside the Controllers to a complete minimum.

#### Filters
Filters are used in conjunction with arrays of data and also outside of loops. If you're looping through data and want to filter out specific things, you're in the right place, you can also use Filters for filtering what a user types inside an &lt;input&gt; field for example. There are a few ways to use Filters, inside Controllers or as a defined method. Here's the method usage, which you can use globally:

{% highlight javascript %}
myApp.filter('reverse', function () {
    return function (input, uppercase) {
        var out = '';
        for (var i = 0; i < input.length; i++) {
            out = input.charAt(i) + out;
        }
        if (uppercase) {
            out = out.toUpperCase();
        }
        return out;
    }
});

// Controller included to supply data
myApp.controller('MainCtrl', ['$scope', function ($scope) {
    $scope.greeting = 'Todd Motto';
}]);
{% endhighlight %}

DOM usage:

{% highlight html %}
<div ng-app="myApp">
    <div ng-controller="MainCtrl">
        <p>No filter: {% raw %}{{ greeting }}{% endraw %}</p>
        <p>Reverse: {% raw %}{{ greeting | reverse }}{% endraw %}</p>
    </div>
</div>
{% endhighlight %}

Output:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/pmh4s/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

You can see that we passed the greeting data to a filter via the pipe (|) character, applying the reverse filter to the greeting data.

And its usage inside an _ng-repeat_:

{% highlight html%}
<ul>
  <li ng-repeat="number in myNumbers |filter:oddNumbers">{% raw %}{{ number }}{% endraw %}</li>
</ul>
{% endhighlight %}

And here's a real quick example of a Filter _inside_ a Controller:

{% highlight javascript %}
myApp.controller('MainCtrl', ['$scope', function ($scope) {
    
    $scope.numbers = [10, 25, 35, 45, 60, 80, 100];
    
    $scope.lowerBound = 42;
    
    // Does the Filters
    $scope.greaterThanNum = function (item) {
        return item > $scope.lowerBound;
    };
    
}]);
{% endhighlight %}

And its usage inside an _ng-repeat_:

{% highlight html%}
<li ng-repeat="number in numbers | filter:greaterThanNum">
  {% raw %}{{ number }}{% endraw %}
</li>
{% endhighlight %}

Output:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/cZbCf/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

That's the main bulk behind AngularJS and its APIs, this is only diving into the shallow end of the waters, but is more than enough to get you building your own Angular application.

### Two-way data-binding
When I first heard about two-way data-binding, I wasn't really sure what it was. Two-way data-binding is best described as a full-circle of synchronised data: update the _Model_ and it updates the _View_, update the _View_ and it updates the _Model_. This means that data is kept in sync without making any fuss. If I bind an _ng-model_ to an &lt;input&gt; and start typing, this creates (or updates an existing) model at the same time.

Here I create the &lt;input&gt; and bind a Model called 'myModel', I can then use the curly handlebars syntax to reflect this model and its updates in the view all at once:

{% highlight html %}
<div ng-app="myApp">
    <div ng-controller="MainCtrl">
        <input type="text" ng-model="myModel" placeholder="Start typing..." />
        <p>My model data: {% raw %}{{ myModel }}{% endraw %}</p>
    </div>
</div>
{% endhighlight %}

{% highlight javascript %}
myApp.controller('MainCtrl', ['$scope', function ($scope) {
    // Capture the model data
    // and/or initialise it with an existing string
    $scope.myModel = '';
}]);
{% endhighlight %}

Output:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/qrr3q/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### XHR/Ajax/$http calls and binding JSON
You've got the idea when it comes to pushing basic data to the _$scope_ now, and a rough idea of how the models and two-way data-binding works, so now it's time to emulate some real XHR calls to a server. For websites, this isn't essential unless you have specific Ajax requirements, this is mainly focused on grabbing data for a web application.

When you're developing locally, you're possibly using something like Java, ASP .NET, PHP or something else to run a local server. Whether you're contacting a local database or actually using the server as an API to communicate to another resource, this is much the same setup.

Enter 'dollar http'. Your best friend from now on. The _$http_ method is a nice Angular wrapper for accessing data from the server, and so easy to use you could do it blindfolded. Here's a simple example of a 'GET' request, which (you guessed it) gets data from the server. Its syntax is very jQuery-like so transitioning is a breeze:

{% highlight javascript %}
myApp.controller('MainCtrl', ['$scope', '$http', function ($scope, $http) {
  $http({
    method: 'GET',
    url: '//localhost:9000/someUrl'
  });
}]);
{% endhighlight %}

Angular then returns something called a _promise_, which is a much more efficient and readable way of handling callbacks. Promises are chained onto the function they're initiated from using dot notation _.myPromise()_. As expected, we've got error and success handlers:

{% highlight javascript %}
myApp.controller('MainCtrl', ['$scope', function ($scope) {
  $http({
    method: 'GET',
    url: '//localhost:9000/someUrl'
  })
  .success(function (data, status, headers, config) {
    // successful data retrieval
  })
  .error(function (data, status, headers, config) {
    // something went wrong :(
  });
}]);
{% endhighlight %}

Very nice and readable. This is where we merge the View and the server by binding a Model or updating Model data to the DOM. Let's assume a setup and push a username to the DOM, via an Ajax call.

Ideally, we should setup and design our JSON first, which will affect how we bind our data. Let's keep it simple, this is what a backend guy will setup as an API feed down to your application, you'll be expecting the following:

{% highlight javascript %}
{
  "user": {
    "name": "Todd Motto",
    "id": "80138731"
  }
}
{% endhighlight %}

This means we'll get an Object returned back from the server (under an alias we'll call 'data' [you can see _data_ is passed into our promise handlers]), and have to hook into the _data.user_ property. Inside the _data.user_ prop, we have _name_ and _id_. Accessing those is easy, we'll need to look for  _data.user.name_ which will give us 'Todd Motto'. Now let's fetch it!

The JavaScript (check inline annotations for what's going on here):
{% highlight javascript %}
myApp.controller('UserCtrl', ['$scope', '$http', function ($scope, $http) {

  // create a user Object
  $scope.user = {};

  // Initiate a model as an empty string
  $scope.user.username = '';

  // We want to make a call and get
  // the person's username
  $http({
    method: 'GET',
    url: '//localhost:9000/someUrlForGettingUsername'
  })
  .success(function (data, status, headers, config) {
    // See here, we are now assigning this username
    // to our existing model!
    $scope.user.username = data.user.name;
  })
  .error(function (data, status, headers, config) {
    // something went wrong :(
  });
}]);
{% endhighlight %}

And now in the DOM, we can just do this:

{% highlight html %}
<div ng-controller="UserCtrl">
  <p>{% raw %}{{ user.username }}{% endraw %}</p>
</div>
{% endhighlight %}

This will now print the username. Now we'll take this even further with understanding declarative data-binding which is where things get really exciting.

### Declarative data-binding
Angular's philosophy is creating dynamic HTML that's rich in functionality and does a hell of a lot of work seamlessly that you would never expect on the client-side of the web. This is exactly what they've delivered.

Let's imagine we've just made an Ajax request to get a list of emails and their Subject line, date they were sent and want to render them in the DOM. This is where jaws drop at the power of Angular. First I'll need to setup a Controller for Emails:

{% highlight javascript %}
myApp.controller('EmailsCtrl', ['$scope', function ($scope) {

  // create a emails Object
  $scope.emails = {};

  // pretend data we just got back from the server
  // this is an ARRAY of OBJECTS
  $scope.emails.messages = [{
        "from": "Steve Jobs",
        "subject": "I think I'm holding my phone wrong :/",
        "sent": "2013-10-01T08:05:59Z"
    },{
        "from": "Ellie Goulding",
        "subject": "I've got Starry Eyes, lulz",
        "sent": "2013-09-21T19:45:00Z"
    },{
        "from": "Michael Stipe",
        "subject": "Everybody hurts, sometimes.",
        "sent": "2013-09-12T11:38:30Z"
    },{
        "from": "Jeremy Clarkson",
        "subject": "Think I've found the best car... In the world",
        "sent": "2013-09-03T13:15:11Z"
    }];

}]);
{% endhighlight %}

Now we need to plug it into our HTML. This is where we'll use declarative bindings to _declare_ what the application will do - to create our first piece of dynamic HTML. We're going to be using Angular's built-in _ng-repeat_ directive, which will iterate over data and render an output with absolutely no callbacks or state changes, it's all for free:

{% highlight html %}
<ul>
  <li ng-repeat="message in emails.messages">
    <p>From: {% raw %}{{ message.from }}{% endraw %}</p>
    <p>Subject: {% raw %}{{ message.subject }}{% endraw %}</p>
    <p>{% raw %}{{ message.sent | date:'MMM d, y h:mm:ss a' }}{% endraw %}</p>
  </li>
</ul>
{% endhighlight %}

Output:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/TAVQc/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

I've also snuck in a _date filter_ in there too so you can see how to render UTC dates.

Dig into Angular's suite of _ng-*_ directives to unleash the full power of declarative bindings, this shows you how to join the dots from server to Model to View and render the data.

### Scope functions
As a continuation from declarative-binding, scope functions are the next level up in creating an application with some functionality. Here's a basic function to _delete_ one of our emails in our data:

{% highlight javascript %}
myApp.controller('MainCtrl', ['$scope', function ($scope) {

  $scope.deleteEmail = function (index) {
    $scope.emails.messages.splice(index, 1)
  };

}]);
{% endhighlight %}

Pro tip: It's important to think about deleting _data_ from the Model. You're not deleting elements or anything actual DOM related, Angular is an MVC framework and will handle all this for you with its two-way binding and callback-free world, you just need to setup your code intelligently to let it respond to your data!

Binding functions to the scope are also run through _ng-*_ Directives, this time it's an _ng-click_ Directive:

{% highlight html %}
<a ng-click="deleteEmail($index)">Delete email</a>
{% endhighlight %}

This is way different to inline click handlers, for many reasons. This will be covered soon. You'll see I'm also passing in the _$index_, Angular knows which item you're deleting (how much code and logic does that save you!).

Output (delete some emails!):

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/BgZmt/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### Declarative DOM methods
Now we'll move onto _DOM Methods_, these are also Directives and simulate functionality in the DOM which you'd normally end up writing even more script logic for. A great example of this would be a simple toggling navigation. Using _ng-show_ and a simple _ng-click_ setup, we can create a flawless toggling nav:

{% highlight html %}
<a href="" ng-click="toggle = !toggle">Toggle nav</a>
<ul ng-show="toggle">
  <li>Link 1</li>
  <li>Link 2</li>
  <li>Link 3</li>
</ul>
{% endhighlight %}

This is where we enter MVVM, you'll notice there is no Controller being introduced here, we'll come onto MVVM soon.

Output (get toggling!):

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/ZUyW5/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### Expressions
One of my favourite parts of Angular, what you'd usually use JavaScript for and write a lot of repetitive code.

Have you ever done this?

{% highlight javascript %}
elem.onclick = function (data) {
  if (data.length === 0) {
    otherElem.innerHTML = 'No data';
  } else {
    otherElem.innerHTML = 'My data';
  }
};
{% endhighlight %}

This would potentially be a callback from a _GET_ request, and you'll alter the DOM based on the data's state. Angular gives you this for free too, and you'll be able to do it inline without writing any JavaScript!

{% highlight html %}
<p>{% raw %}{{ data.length > 0 && 'My data' || 'No data' }}{% endraw %}</p>
{% endhighlight %}

This will just update itself dynamically without callbacks as your application polls/fetches data. If there's no data, it'll tell you - if there's data, it'll say. There are so many use cases for this and Angular handles it all automatically via two-way binding magic.

Output:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/XCwcr/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### Dynamic views and routing
The philosophy behind single-page web applications (and also websites!). You have a header, footer, sidebar, and the content in the middle magically injects new content based on your URL.

Angular makes this setup a breeze to configure what we'd call _dynamic views_. Dynamic views inject specific Views based on the URL, through the _$routeProvider_. A simple setup:

{% highlight javascript %}
myApp.config(['$routeProvider', function ($routeProvider) {

  /**
   * $routeProvider
   */
  $routeProvider
  .when('/', {
    templateUrl: 'views/main.html'
  })
  .otherwise({
    redirectTo: '/'
  });

}]);
{% endhighlight %}

You'll see that 'when' the URL is '/' (i.e. the root of the site), you'll want to inject _main.html_. It's a good idea to call your initial View _main.html_ and not _index.html_ as you'll already have an _index.html_ page for your single-page setup (a mouthful, yes). Adding more Views based on the URL is so simple:

{% highlight javascript %}
myApp.config(['$routeProvider', function ($routeProvider) {

  /**
   * $routeProvider
   */
  $routeProvider
  .when('/', {
    templateUrl: 'views/main.html'
  })
  .when('/emails', {
    templateUrl: 'views/emails.html'
  })
  .otherwise({
    redirectTo: '/'
  });

}]);
{% endhighlight %}

We could then have _emails.html_ simply loaded with our HTML which generates our email list. You end up creating a very sophisticated application with little effort at this point.

There is a lot more to the _$routeProvider_ service which is well worth reading up on, but this'll get the ball rolling for you. There are things such as _$http_ interceptors that'll fire events when an Ajax call is in progress, things we could show some spinners for whilst we load in fresh data.

### Global static data
Gmail handles a lot of its initial data by writing the JSON into the page (right-click View Source). If you want to instantly set data in your page, it'll quicken up rendering time and Angular will hit the ground running.

When I develop our apps, Java tags are placed in the DOM and when rendered, the data is sent down from the backend. [I've zero experience with Java so you'll get a made up declaration from me below, you could use any language though on the server.] Here's how to write JSON into your page and then pass it into a Controller for immediate binding use:

{% highlight html %}
<!-- inside index.html (bottom of page ofc) -->
<script>
window.globalData = {};
globalData.emails = <javaTagHereToGenerateMessages>;
</script>
{% endhighlight %}

My made up Java tag will then render the data whilst the page parses and Angular will render your emails instantly. Just feed it your data inside a Controller:

{% highlight javascript %}
myApp.controller('EmailsCtrl', ['$scope', function ($scope) {

    $scope.emails = {};
    
    // Assign the initial data!
    $scope.emails.messages = globalData.emails;
    
}]);
{% endhighlight %}

### Minification
I'll talk a little about minifying your Angular code. You'll probably have experimented a little at this point and perhaps ran your code through a minifier - and perhaps encountered an error!

Minifying your AngularJS code is simple, you need to specify your dependency injected content in an array before the function:

{% highlight javascript %}
myApp.controller('MainCtrl',
['$scope', 'Dependency', 'Service', 'Factory',
function ($scope, Dependency, Service, Factory) {

  // code

}]);
{% endhighlight %}

Once minified:

{% highlight javascript %}
myApp.controller('MainCtrl',
['$scope', 'Dependency', 'Service', 'Factory',
function (a,b,c,d) {

  // a = $scope
  // b = Dependency
  // c = Service
  // d = Factory

  // $scope alias usage
  a.someFunction = function () {...};

}]);
{% endhighlight %}

Just remember to keep the injectors in the order the appear, or you'll probably cause you and your team a headache.

### Scope comments
Scope comments I think are a really nice addition to my workflow, instead of declaring chunks of my HTML with comments like this:

{% highlight html %}
<!-- header -->
<header>
  Stuff.
</header>
<!-- /header -->
{% endhighlight %}

When introducing Angular, let's think about Views and Scopes, not the DOM! Scopes are in fact _scoped_, meaning unless you deliberately share the data between Controllers, your data is scoped and inaccessible elsewhere. I find laying out my scoped areas to be a real helper:

 {% highlight html %}
<!-- scope: MainCtrl -->
<div class="content" ng-controller="MainCtrl">

</div>
<!-- /scope: MainCtrl -->
{% endhighlight %}

### Debugging AngularJS
There's an awesome Chrome Extension that the guys at Google recommend for developing and debugging with Angular, it's called Batarang and you can grab it [here](https://chrome.google.com/webstore/detail/angularjs-batarang/ighdmehidhipcmcojjgiloacoafjmpfk).

Happy coding.

### Further reading

- [Structuring Angular Controllers](/rethinking-angular-js-controllers)
- [All about custom Angular filters](/everything-about-custom-filters-in-angular-js)
- [Controller as syntax](/digging-into-angulars-controller-as-syntax)
- [$scope and $rootScope event system $emit, $broadcast and $on](/all-about-angulars-emit-broadcast-on-publish-subscribing)
- [Modular IIFE modules](/minimal-angular-module-syntax-approach-using-an-iife)
- This post, translated in [French!](http://vfsvp.fr/article/apprendre-angular-en-un-jour-le-guide-ultime)
- [German translation](https://github.com/futape/angularjs-in-one-day-german/blob/master/angularjs-in-einem-tag.md) of this post
- This post, on [SpeakerDeck in slides](https://speakerdeck.com/toddmotto/angularjs-in-one-day)
- [Code your own Directive](/creating-an-angularjs-directive-from-one-of-your-existing-plugins-scripts) from a custom script or plugin
