---
layout: post
permalink: /echo-js-simple-javascript-image-lazy-loading/
title: Echo.js, simple JavaScript image lazy loading
path: 2013-08-11-echo-js-simple-javascript-image-lazy-loading.md
tags:
- Lazy-loading
- Images
- Library
- JavaScript
---

I'm currently working on a project for Intel's [HTML5 Hub](//html5hub.com) in which I require some image lazy-loading for an HTML5 showcase piece that's high in image content. After a quick Google search for an existing lazy-load solution there was yet another mass of outdated scripts or jQuery plugins that were too time consuming to search through or modify for the project - so I ended up writing my own.

Echo.js is probably as simple as image lazy loading gets, it's less than 1KB minified and is library agnostic (no jQuery/Zepto/other).

<div class="download-box">
  <a href="//toddmotto.com/labs/echo" onclick="_gaq.push(['_trackEvent', 'Click', 'Demo echo', 'echo Demo']);">Demo</a>
  <a href="//github.com/toddmotto/echo/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download echo', 'Download echo']);">Download</a>
  <a href="//github.com/toddmotto/echo" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork echo', 'echo Fork']);">Fork</a>
</div>

Lazy-loading works by only loading the assets needed when the elements 'would' be in view, which it'll get from the server for you upon request, which is automated by simply changing the image _src_ attribute. This is also an asynchronous process which also benefits us.

### Using Echo.js
Using Echo is really easy, just include an original image to be used as a placeholder, for the demo I am using a simple AJAX _.gif_ spinner as a background image with a transparent .gif placeholder so the user will always see something is happening, but you can use whatever you like.

Here's the markup to specify the image source, which is literal so you'll be able to specify the full file path (even the full _http://_ if you like) which makes it easier when working with directories.

{% highlight html %}
<img src="img/blank.gif" alt="" data-echo="img/album-1.jpg">
{% endhighlight %}

Just drop the script into your page before the closing _&lt;/body&gt;_ tag and let it do its thing. For modern browsers I've used the _DOMContentLoaded_ event incase you _really_ need it in the _&lt;head&gt;_, which is a native 'DOM Ready', and a fallback to onload for IE7/8 if you need to go that far so all works nicely.

### JavaScript
As always, I'll talk through the script for those interested in the behind the scenes working. Here's the full script:

{% highlight javascript %}
window.echo = (function (window, document) {

  'use strict';

  /*
   * Constructor function
   */
  var Echo = function (elem) {
    this.elem = elem;
    this.render();
    this.listen();
  };

  /*
   * Images for echoing
   */
  var echoStore = [];
  
  /*
   * Element in viewport logic
   */
  var scrolledIntoView = function (element) {
    var coords = element.getBoundingClientRect();
    return ((coords.top >= 0 && coords.left >= 0 && coords.top) <= (window.innerHeight || document.documentElement.clientHeight));
  };

  /*
   * Changing src attr logic
   */
  var echoSrc = function (img, callback) {
    img.src = img.getAttribute('data-echo');
    if (callback) {
      callback();
    }
  };

  /*
   * Remove loaded item from array
   */
  var removeEcho = function (element, index) {
    if (echoStore.indexOf(element) !== -1) {
      echoStore.splice(index, 1);
    }
  };

  /*
   * Echo the images and callbacks
   */
  var echoImages = function () {
    for (var i = 0; i < echoStore.length; i++) {
      var self = echoStore[i];
      if (scrolledIntoView(self)) {
        echoSrc(self, removeEcho(self, i));
      }
    }
  };

  /*
   * Prototypal setup
   */
  Echo.prototype = {
    init : function () {
      echoStore.push(this.elem);
    },
    render : function () {
      if (document.addEventListener) {
        document.addEventListener('DOMContentLoaded', echoImages, false);
      } else {
        window.onload = echoImages;
      }
    },
    listen : function () {
      window.onscroll = echoImages;
    }
  };

  /*
   * Initiate the plugin
   */
  var lazyImgs = document.querySelectorAll('img[data-echo]');
  for (var i = 0; i < lazyImgs.length; i++) {
    new Echo(lazyImgs[i]).init();
  }

})(window, document);
{% endhighlight %}

The script takes an Object-Orientated approach, instantiating the _Echo_ object (which is our Function constructor) on each element instance of NodeList inside our _for_ loop. You can see this instantiation at the end of the script, using the _new_ operator.

The first main chunk of code we see if an _anonymous function expression_ which acts as our _Constructor_. Following convention, Constructor function names should have the first letter capitalised:

{% highlight javascript %}
var Echo = function (elem) {
  this.elem = elem;
  this.render();
  this.listen();
};
{% endhighlight %}

I pass in the _elem_ argument, which will be the current element inside the _for_ loop in which the plugin is called upon, and call _render.();_ and _listen();_ internally, this will run the prototype functions that the Object inherits.

Next is an empty array:

{% highlight javascript %}
var echoStore = [];
{% endhighlight %}

This empty array will act as our data store for pushing our images that need lazy-loading into. It's a good practice to use arrays for this type of thing so we can remove images that are already loaded from the same array, this will prevent our loops iterating over the same array, it may as well perform faster and loop over fewer items.

Next, here's a neat little function to detect whether the element is in view:

{% highlight javascript %}
var scrolledIntoView = function (element) {
  var coords = element.getBoundingClientRect();
  return ((coords.top >= 0 && coords.left >= 0 && coords.top) <= (window.innerHeight || document.documentElement.clientHeight));
};
{% endhighlight %}

This uses a great addition to JavaScript, the _.getBoundingClientRect()_ method which returns a text rectangle object which encloses a group of text rectangles, which are the _border-boxes_ associated with that element, i.e. CSS box. The returned data describes the top, right, bottom and left in pixels. We can then make a smart comparison against the _window.innerHeight_ or the _document.documentElement.clientHeight_, which gives you the visible area inside your browser on a cross-browser basis.

Next up is a very simple function that switches the current image's _src_ attribute to the associated _data-echo_ attribute once it's needed:

{% highlight javascript %}
var echoSrc = function (img, callback) {
  img.src = img.getAttribute('data-echo');
  if (callback) {
    callback();
  }
};
{% endhighlight %}

If a callback is present, it will run (I do pass in a callback here, but to prevent errors it's good to simply _if_ statement this stuff).

The next function I've setup to check if the current element exists in the array, and if it does, it removes it using the _.splice()_ method on the current index to remove 'itself':

{% highlight javascript %}
var removeEcho = function (element, index) {
  if (echoStore.indexOf(element) !== -1) {
    echoStore.splice(index, 1);
  }
};
{% endhighlight %}

The fundamental tie in for the script is listening for constant updates in the view based on our data store array. This function loops through our data store, and checks if the current element in the array is in view after initiating the _scrolledIntoView_ function. If that proves to be true, then we call the _echoSrc_ function, pass in the current element and also the current element's _index_ value, being _i_. This index value gets passed into the _removeEcho_ function which in turn removes a copy of itself from the array. This means our array has become shorter and our JavaScript doesn't have to work as hard or as long when looping through our leftover elements.

{% highlight javascript %}
var echoImages = function () {
  for (var i = 0; i < echoStore.length; i++) {
    var self = echoStore[i];
    if (scrolledIntoView(self)) {
      echoSrc(self, removeEcho(self, i));
    }
  }
};
{% endhighlight %}

The OO piece of the script looks inside the _prototype_ extension, which has a few functions inside. The first is the _init()_ function, that simply pushes the current element into our data store array. The _render()_ function checks to see if an _addEventListener_ event exists, which will then invoke the _echoImages_ function once the _DOMContentLoaded_ event is fired. If it doesn't exist, likely inside IE7/8, it'll just run _onload_. The _listen()_ function will just run the function again each time the window is scrolled, to poll and see if any elements come into view to work its magic some more.

{% highlight javascript %}
Echo.prototype = {
  init : function () {
    echoStore.push(this.elem);
  },
  render : function () {
    if (document.addEventListener) {
      document.addEventListener('DOMContentLoaded', echoImages, false);
    } else {
      window.onload = echoImages;
    }
  },
  listen : function () {
    window.onscroll = echoImages;
  }
};
{% endhighlight %}

The final piece of the script is the beautiful API where you invoke a new Object on each item in a NodeList:

{% highlight javascript %}
var lazyImgs = document.querySelectorAll('img[data-echo]');
for (var i = 0; i < lazyImgs.length; i++) {
  new Echo(lazyImgs[i]).init();
}
{% endhighlight %}

I chose to run a regular _for_ loop on this, but if you're routing for more modern JavaScript APIs you can of course do this which is much cleaner but unsupported in older IE (yes I can polyfill but the script is too small to warrant it):

{% highlight javascript %}
[].forEach.call(document.querySelectorAll('img[data-echo]'), function (img) {
  new Echo(img).init();
}
{% endhighlight %}

<div class="download-box">
  <a href="//toddmotto.com/labs/echo" onclick="_gaq.push(['_trackEvent', 'Click', 'Demo echo', 'echo Demo']);">Demo</a>
  <a href="//github.com/toddmotto/echo/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download echo', 'Download echo']);">Download</a>
  <a href="//github.com/toddmotto/echo" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork echo', 'echo Fork']);">Fork</a>
</div>
