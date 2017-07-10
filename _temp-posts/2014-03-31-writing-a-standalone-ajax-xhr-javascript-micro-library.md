---
layout: post
permalink: /writing-a-standalone-ajax-xhr-javascript-micro-library/
title: Writing a standalone Ajax/XHR JavaScript micro-library
path: 2014-03-31-writing-a-standalone-ajax-xhr-javascript-micro-library.md
tags:
- XHR
- Library
- JavaScript
---

Whether you're working with websites or applications, you're bound to have faced a task dealing with Ajax requests, whether it be getting a new document's content or fetching updated JSON data. You're bound to have also used some form of library to gloss over the mess of an implementation XHR is.

There are a tonne of libraries, and a few decent XHR modules that allow you to make simple XHRs. Working with AngularJS daily I love the syntax for working with XHR, over their `$http` method:

{% highlight javascript %}
$http.get('/endpoint')
.success(function (data) {
  
})
.error(function (data) {
  
});
{% endhighlight %}

Angular makes it really easy, and nice and readable, note the shorthand `.get()` method. It also comes with other methods such as `.post()`, `.put()` and `.delete()` to do the majority of things you need.

I wanted to take this sugar syntax approach and write the simplest cross-browser XHR module I could, so I'll take you through Atomic.js, which is the result of that.

<div class="download-box">
  <a href="//github.com/toddmotto/atomic/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download atomic', 'Download atomic']);">Download</a>
  <a href="//github.com/toddmotto/atomic" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork atomic', 'atomic Fork']);">Fork on GitHub</a>
</div>

### XMLHttpRequest and ActiveXObject
It all started (interestingly enough) with Microsoft, when they first came up with Ajax technologies, implemented through ActiveXObject. There was then a standardised approach via XMLHttpRequest (XHR) which formed years later, and is now the way we communicate with the servers using Ajax techniques today.

Around the web, you can find scripts like this, for "cross-browser" Ajax ([source](http://www.webmasterworld.com/javascript/4027629.htm)):

{% highlight javascript %}
function getXHR() { 
  if (window.XMLHttpRequest) {
    // Chrome, Firefox, IE7+, Opera, Safari
    return new XMLHttpRequest(); 
  } 
  // IE6
  try { 
    // The latest stable version. It has the best security, performance, 
    // reliability, and W3C conformance. Ships with Vista, and available 
    // with other OS's via downloads and updates. 
    return new ActiveXObject('MSXML2.XMLHTTP.6.0');
  } catch (e) { 
    try { 
      // The fallback.
      return new ActiveXObject('MSXML2.XMLHTTP.3.0');
    } catch (e) { 
      alert('This browser is not AJAX enabled.'); 
      return null;
    } 
  } 
}
{% endhighlight %}

At first, if you've never seen what I'd call "raw Ajax", underneath all the libraries and wrappers, you're probably wondering what the hell happened. It's an ugly sight indeed.

So, amongst browsing for simpler solutions, I stumbled upon a [GitHub Gist](https://gist.github.com/jed/993585#file-annotated-js) from [Jed Schmidt](https://github.com/jed), where an amazing conversation slowly refactored Jed's initial stab at a very concise supported XHR instance.

What started as this (annotated version):

{% highlight javascript %}
function(
  a // cursor placeholder
){
  for(                    // for all a
    a = 3;                // from 3
    a--;                  // to 0,
  ) try {                 // try
    return new(           // returning a new
      this.XMLHttpRequest // XMLHttpRequest (w3c)
      ||                  // or
      ActiveXObject       // ActiveXObject (MS)
    )([                   // reflecting
      "Msxml2",           // the
      "Msxml3",           // various
      "Microsoft"][a] +   // MS flavors
      ".XMLHTTP"          // and appropriate suffix,
    )
  }
  
  catch(e){}              // and ignore when it fails.
}
{% endhighlight %}

Ended up with a very [minimal version](https://gist.github.com/jed/993585#comment-40084):

{% highlight javascript %}
(function () {
  try {
    return new(this.XMLHttpRequest || ActiveXObject)('MSXML2.XMLHTTP.3.0');
  } catch (e) {}
})();
{% endhighlight %}

I don't know about you, but that is magic - I love it. It makes a conditional call inside the Constructor based on what Object is available in your browser. Apparently you don't need the loop Jed implemented above, and you can get away with the above parameter only, which works in IE5.5+. Fantastic.

So I thought I'd start with this great implementation as the base of atomic.js.

All the above does is provide me with a supported instance however, and no actual communication with the server, there's a little bit more to XHR than that. Here's how the next steps will pan out, using a `GET` method and simplified for this example:

{% highlight javascript %}
// get XHR
var xhr = new XMLHttpRequest(); 

// function to fire each time `onreadystatechange` fires
xhr.onreadystatechange = function () {

};

// open the connection and make the `GET`
xhr.open('GET', '/endpoint', true);

// send it!
xhr.send();
{% endhighlight %}

Inside `onreadystatechange`, we need to then look out for the `readyState` we need. Here's a list of readyStates and their meanings:

0: Request not initialized 
1: Server connection established
2: Request received 
3: Processing request 
4: Request finished and response is ready

So, we need to check that it's all okay, we look for `4`:

{% highlight javascript %}
xhr.onreadystatechange = function () {
  if (xhr.readyState === 4) {
    // request finished and response is ready
  }
};
{% endhighlight %}

We'll hopefully get a 200 status code next, which means all is okay. Anything else and something's probably wrong, or missing from the server, or not authenticated. But for simplicity, we're all good:

{% highlight javascript %}
xhr.onreadystatechange = function () {
  if (xhr.readyState === 4) {
    // request finished and response is ready
    if (xhr.status === 200) {
      // all is well!
    }
  }
};
{% endhighlight %}

So what about when it fails? We can just put an `else` function:

{% highlight javascript %}
xhr.onreadystatechange = function () {
  if (xhr.readyState === 4) {
    // request finished and response is ready
    if (xhr.status === 200) {
      // all is well!
    } else {
      // things are not well :(
    }
  }
};
{% endhighlight %}

All together now:

{% highlight javascript %}
// get XHR
var xhr = new XMLHttpRequest(); 

xhr.onreadystatechange = function () {
  if (xhr.readyState === 4) {
    // request finished and response is ready
    if (xhr.status === 200) {
      // all is well!
    } else {
      // things are not well :(
    }
  }
};

// open the connection and make the `GET`
xhr.open('GET', '/endpoint', true);

// send it!
xhr.send();
{% endhighlight %}

And that's pretty much the basics of using XHR!

It's pretty complex to keep rewriting though, which is why I wanted to wrap it up in a module for simple reuse. Loving the Angular syntax, I thought about doing something like this:

{% highlight javascript %}
atomic.get('/endpoint')
.success(function (data) {
  
})
.error(function (data) {
  
});
{% endhighlight %}

Look familiar? ;)

So working from what we've already got above, I created some chained methods for the module, adding in some automatic JSON parsing when available, and ended up with the following (which is atomic.js v1.0.0):

{% highlight javascript %}
/*! atomic v1.0.0 | (c) 2015 @toddmotto | github.com/toddmotto/atomic */
(function (root, factory) {
  if (typeof define === 'function' && define.amd) {
    define(factory);
  } else if (typeof exports === 'object') {
    module.exports = factory;
  } else {
    root.atomic = factory(root);
  }
})(this, function (root) {

  'use strict';

  var exports = {};

  var config = {
    contentType: 'application/x-www-form-urlencoded'
  };

  var parse = function (req) {
    var result;
    try {
      result = JSON.parse(req.responseText);
    } catch (e) {
      result = req.responseText;
    }
    return [result, req];
  };

  var xhr = function (type, url, data) {
    var methods = {
      success: function () {},
      error: function () {},
      always: function () {}
    };
    var XHR = root.XMLHttpRequest || ActiveXObject;
    var request = new XHR('MSXML2.XMLHTTP.3.0');

    request.open(type, url, true);
    request.setRequestHeader('Content-type', config.contentType);
    request.onreadystatechange = function () {
      var req;
      if (request.readyState === 4) {
        req = parse(request);
        if (request.status >= 200 && request.status < 300) {
          methods.success.apply(methods, req);
        } else {
          methods.error.apply(methods, req);
        }
        methods.always.apply(methods, req);
      }
    };
    request.send(data);

    var atomXHR = {
      success: function (callback) {
        methods.success = callback;
        return atomXHR;
      },
      error: function (callback) {
        methods.error = callback;
        return atomXHR;
      },
      always: function (callback) {
        methods.always = callback;
        return atomXHR;
      }
    };

    return atomXHR;
  };

  exports.get = function (src) {
    return xhr('GET', src);
  };

  exports.put = function (url, data) {
    return xhr('PUT', url, data);
  };

  exports.post= function (url, data) {
    return xhr('POST', url, data);
  };

  exports.delete = function (url) {
    return xhr('DELETE', url);
  };

  exports.setContentType = function(value) {
    config.contentType = value;
  };

  return exports;

});
{% endhighlight %}

Using atomic.js is just as easy as any other library, except it's got a very readable syntax, it's less than 1KB, and punches well above its weight in functionality.

I've got some ideas planned for the future development of atomic, and of course feel free to help out!

<div class="download-box">
  <a href="//github.com/toddmotto/atomic/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download atomic', 'Download atomic']);">Download</a>
  <a href="//github.com/toddmotto/atomic" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork atomic', 'atomic Fork']);">Fork on GitHub</a>
</div>
