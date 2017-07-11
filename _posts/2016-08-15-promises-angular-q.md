---
layout: post
permalink: /promises-angular-q
title: All about $q and Promises in Angular
path: 2016-08-15-promises-angular-q.md
---

You've seen `$q`, maybe used it, but haven't uncovered some of the awesome features `$q` provides, such as `$q.all()` and `$q.race()`. This article dives into ES2015 Promise API and how it maps across to `$q` for AngularJS. This post is all about `$q`... enjoy!

### Table of contents

- <a href="#what-is-a-promise">What is a Promise?</a>
- <a href="#es2015-promises">ES2015 Promises</a>
- <a href="#q-constructor">$q constructor</a>
- <a href="#when-to-use-q">When to use $q</a>
- <a href="#qdefer">$q.defer()</a>
- <a href="#qwhen--qresolve">$q.when()/$q.resolve()</a>
- <a href="#qreject">$q.reject()</a>
- <a href="#qall">$q.all()</a>
- <a href="#qrace">$q.race()</a>

### What is a Promise?

A promise is a special type of Object that we can either use, or construct ourselves to handle asynchronous tasks. We deem them promises because we are "promised" a result at a future point in time. For example an HTTP call could complete in `200ms` or `400ms`, a promise will execute when resolved.

A promise has three states, pending, resolved or rejected. Using `$q` in Angular, we can construct our own promises, however let's look at the ES2015 `Promise` Object first to get familiar on how to create one.

### ES2015 Promises

The main things here are `Promise` and the `resolve` and `reject` arguments:

{% highlight javascript %}
let promise = new Promise((resolve, reject) => {
  if (/* some async task is all good */) {
    resolve('Success!');
  } else {
    reject('Oops... something went wrong');
  }
});

promise.then(data => {
  console.log(data);
});
{% endhighlight %}

We simply call `new Promise()` and inside can perform an asynchronous task, which may be for wrapping particular DOM events, or even wrapping third-party libraries that are not promise Objects.

For example, wrapping a pseudo third-party library called `myCallbackLib()` which gives us a success and error callback, we can construct a Promise around this to `resolve` and `reject` where appropriate:

{% highlight javascript %}
const handleThirdPartyCallback = someArgument => {
  let promise = new Promise((resolve, reject) => {
    // assuming some third-party API, that is *not* a Promise Object
    // but fires a callback once finished
    myCallbackLib(someArgument, response => {
      // we can resolve it with the response
      resolve(response);
    }, reason => {
      // we can reject it with the reason
      reject(reason);
    });
  });
  return promise;
};

handleThirdPartyCallback({ user: 101 }).then(data => {
  console.log(data);
});
{% endhighlight %}

### $q constructor

The `$q` implementation in AngularJS is now available in the same aspect as the native ES2015 `Promise` Object, therefore we can do this:

{% highlight javascript %}
let promise = $q((resolve, reject) => {
  if (/* some async task is all good */) {
    resolve('Success!');
  } else {
    reject('Oops... something went wrong');
  }
});

promise.then(data => {
  console.log(data);
});
{% endhighlight %}

The only difference here from above is changing `new Promise()` to `$q()`, simple enough.

Ideally you'd implement this inside a service:

{% highlight javascript %}
function MyService($q) {
  return {
    getSomething() {
      return $q((resolve, reject) => {
        if (/* some async task is all good */) {
          resolve('Success!');
        } else {
          reject('Oops... something went wrong');
        }
      });
    }
  };
}

angular
  .module('app')
  .service('MyService', MyService);
{% endhighlight %}

Which would then be injected into a [component controller](/exploring-the-angular-1-5-component-method/):

{% highlight javascript %}
const stuffComponent = {
  template: `
    <div>
      {% raw %}{{ $ctrl.stuff }}{% endraw %}
    </div>
  `,
  controller(MyService) {
    this.stuff = [];
    MyService.getSomething()
      .then(data => this.stuff.unshift(data));
  }
};

angular
  .module('app')
  .component('stuffComponent', stuffComponent);
{% endhighlight %}

Or used as a `bindings` property on a [routed component](https://github.com/toddmotto/angular-styleguide#routed-components) and mapped with a [routing resolve](/resolve-promises-in-angular-routes/):

{% highlight javascript %}
const stuffComponent = {
  bindings: {
    stuff: '<'
  },
  template: `
    <div>
      {% raw %}{{ $ctrl.stuff }}{% endraw %}
    </div>
  `,
  controller(MyService) {
    // your stuff already available
    console.log(this.stuff);
  }
};

const config = $stateProvider => {
  $stateProvider
    .state('stuff', {
      url: '/stuff',
      component: 'stuffComponent',
      resolve: {
        // resolve maps the `MyService` promise response
        // Object across to `stuff` property, making it
        // available as a binding inside the .component()
        stuff: MyService => MyService.getSomething()
      }
    });
};

angular
  .module('app')
  .config(config)
  .component('stuffComponent', stuffComponent);
{% endhighlight %}

### When to use $q

At the moment we've only looked at some pseudo-examples, here's my implementation on wrapping the `XMLHttpRequest` Object to become a promise-based solution, this kind of thing should be the only real reason(s) you create your own `$q` promises:

{% highlight javascript %}
let getStuff = $q((resolve, reject) => {
  var xhr = new XMLHttpRequest();
  xhr.onreadystatechange = () => {
    if (xhr.readyState === 4) {
      if (xhr.status === 200) {
        resolve(JSON.parse(xhr.responseText));
      } else {
        reject(JSON.parse(xhr.responseText));
      }
    }
  };
  xhr.open('GET', '/api/stuff', true);
  xhr.send();
});

getStuff.then(data => {
  console.log('Boom!', data);
});
{% endhighlight %}

Please note, this is not advocating the use and creation of the `XMLHttpRequest`, use `$http` inside Angular, which creates and returns a promise Object for you:

{% highlight javascript %}
function getStuff() {
  return $http
    .get('/api/stuff');
    .then(data => {
      console.log('Boom!', data);
    });
}

getStuff().then(data => {
  console.log('Boom!', data);
});
{% endhighlight %}

Which means, you do not, and shouldn't ever do this as it's just creating a new promise Object from an existing promise Object:

{% highlight javascript %}
function getStuff() {
  // don't do this!
  let defer = $q.defer();
  $http
    .get('/api/stuff');
    .then(response => {
      // don't do this!
      $q.resolve(response);
    }, reason => {
      // don't do this!
      $q.reject(reason);
    });
  return defer.promise;
}

getStuff().then(data => {
  console.log('Boom!', data);
});
{% endhighlight %}

Golden rule: use `$q` for non-promise stuff, that's it! Well, only _create_ your Promises in this case, however you can use other methods such as `$q.all()` and `$q.race()` alongside _other_ promises.

### $q.defer()

Using `$q.defer()` is just another flavour, and the original implementation, of `$q()` as a Promise constructor. Let's assume the following code, adapted from the before example using a service:

{% highlight javascript %}
function MyService($q) {
  return {
    getSomething() {
      let defer = $q.defer();
      if (/* some async task is all good */) {
        defer.resolve('Success!');
      } else {
        defer.reject('Oops... something went wrong');
      }
      return defer.promise;
    }
  };
}
{% endhighlight %}

### $q.when() / $q.resolve()

Use `$q.when()` or `$q.resolve()` (they are identical, `$q.resolve()` is an alias for `$q.when()` to align with ES2015 Promise naming conventions) when you want to immediately resolve a promise from a non-promise Object, for example:

{% highlight javascript %}
$q.when(123).then(res => {
  // 123
  console.log(res);
});

$q.resolve(123).then(res => {
  // 123
  console.log(res);
});
{% endhighlight %}

Note: `$q.when()` is also the same as `$q.resolve()`.

### $q.reject()

Using `$q.reject()` will immediately reject a promise, this comes in handy for things such as HTTP Interceptors at the point of no return to return, so we can just return a rejected promise Object:

{% highlight javascript %}
$httpProvider.interceptors.push($q => ({
  request(config) {...},
  requestError(config) {
    return $q.reject(config);
  },
  response(response) {...},
  responseError(response) {
    return $q.reject(response);
  }
}));
{% endhighlight %}

### $q.all()

The time will likely arise where you need to resolve multiple promises at once, this is easily achieved through `$q.all()` by passing in either an Array or Object of promises which will call `.then()` once both are resolved:

{% highlight javascript %}
let promiseOne = $http.get('/api/todos');
let promiseTwo = $http.get('/api/comments');

// Array of Promises
$q.all([promiseOne, promiseTwo]).then(data => {
  console.log('Both promises have resolved', data);
});

// Object hash of Promises
// this is ES2015 shorthand for { promiseOne: promiseOne, promiseTwo: promiseTwo }
$q.all({
    promiseOne,
    promiseTwo
  }).then(data => {
  console.log('Both promises have resolved', data);
});
{% endhighlight %}

### $q.race()

The `$q.race` method is one of the newer arrivals to Angular, and is similar to `$q.all()`, however whichever promise resolves first is the only response Object passed back to you. Assume API call 1 and API call 2 are both executed simultaneously, and API call 2 resolves before API 1, you'll _only_ get API call 2 as the response Object. In other words the fastest promise wins and is returned on its own:

{% highlight javascript %}
let promiseOne = $http.get('/api/todos');
let promiseTwo = $http.get('/api/comments');

// Array of Promises
$q.race([promiseOne, promiseTwo]).then(data => {
  console.log('Fastest wins, who will it be?...', data);
});

// Object hash of Promises
// this is ES2015 shorthand for { promiseOne: promiseOne, promiseTwo: promiseTwo }
$q.race({
    promiseOne,
    promiseTwo
  }).then(data => {
  console.log('Fastest wins, who will it be?...', data);
});
{% endhighlight %}

### Conclusion

Use `$q` for constructing promises from non-promise Objects/callbacks, and utilise `$q.all()` and `$q.race()` to work with existing promises.

If you like this article, check out my advanced [Angular 1.5 master course](https://courses.toddmotto.com/products/ultimate-angularjs-master) which covers all the `$q`, `$httpProvider.interceptors`, `ui-router`, component architecture and the new `.component()` API.

For anything else, [the $q documentation](https://docs.angularjs.org/api/ng/service/$q).
