---
layout: post
permalink: /nofi-detecting-no-wifi-callbacks-for-offline-states-with-html5/
title: NoFi, detecting no WiFi, callbacks for offline states with HTML5
path: 2013-10-28-nofi-detecting-no-wifi-callbacks-for-offline-states-with-html5.md
tags:
- Offline
- Library
- JavaScript
---

An HTML5 API buried inside the `navigator` Object called `onLine` is where the inspiration for this little tool came from. NoFi detects no WiFi (yes, it also includes non-WiFi, your cables) and allows you to run custom callbacks when the offline event is fired.

The plugin's only small but may well help with your website/application development.

<div class="download-box">
  <a href="//toddmotto.com/labs/nofi" onclick="_gaq.push(['_trackEvent', 'Click', 'Demo nofi', 'nofi Demo']);">Demo</a>
  <a href="//github.com/toddmotto/nofi/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download nofi', 'Download nofi']);">Download</a>
  <a href="//github.com/toddmotto/nofi" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork nofi', 'nofi Fork']);">Fork</a>
</div>

### HTML and setup
Just include the script inside your HTML, and initialise the NoFi `.init()` module. NoFi takes just three options when you pass in the object, the custom event name, the interval you want to check for lost internet connection and if you want NoFi to exit the function if connection is lost, otherwise the event will keep firing. If you want to keep firing you could keep `exit` set to `false`.

{% highlight html %}
<script src="js/nofi.js"></script>
<script>
NoFi.init({
  eventName: 'offline',
  interval: 1000,
  exit: true
});
</script>
{% endhighlight %}

### HTML5 API
The HTML5 API is actually so simple to use standalone:

{% highlight javascript %}
if (!navigator.onLine) {
  // You're offline, I think...
}
{% endhighlight %}

There is one issue with this, older browsers will also think they're offline, so I've wrapped my script in a feature detect to test the presence of `onLine` inside the navigator Object, which means the browser supports the `onLine` API and is currently offline:

{% highlight javascript %}
if ('onLine' in navigator) {
  if (!navigator.onLine) {
    // You're REALLY offline
  }
}
{% endhighlight %}

### Recursive setTimeout()
Recursive functions are awesome, and so is a recursive `setTimeout()`. Stop using `setInterval()`, these are very bad and will keep setting an interval despite whether it's finished its operations or not. Using a recursive `setTimeout()` means all the operations inside the timeout are finished as we recursive right at the bottom, it's sexy:

{% highlight javascript %}
if ('onLine' in navigator) {
  (function checkStatus() {
    setTimeout(function () {
      if (!navigator.onLine) {
        emitEvent('offline');
        if (exit) {
          return;
        }
      }
      checkStatus(); // recurse
    }, 10000);
  })();
}
{% endhighlight %}

### Full script
Putting an IIFE wrapper around the script to top off the recursive HTML5-ness:

{% highlight javascript %}
window.NoFi = (function (window, document, undefined) {

  'use strict';

  var emitEvent = function (name) {

    var event = document.createEvent('Event');
    event.initEvent(name, true, true);
    window.dispatchEvent(event);

  };

  var init = function (obj) {

    var options = obj || {};
    var interval = options.interval || 10000;
    var eventName = options.eventName || 'offline';
    var exit = options.exit || false;

    if ('onLine' in navigator) {
      (function checkStatus() {
        setTimeout(function () {
          if (!navigator.onLine) {
            emitEvent(eventName);
            if (exit) {
              return;
            }
          }
          checkStatus();
        }, interval);
      })();
    }

  };

  return {
    init: init
  };

})(window, document);
{% endhighlight %}

<div class="download-box">
  <a href="//toddmotto.com/labs/nofi" onclick="_gaq.push(['_trackEvent', 'Click', 'Demo nofi', 'nofi Demo']);">Demo</a>
  <a href="//github.com/toddmotto/nofi/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download nofi', 'Download nofi']);">Download</a>
  <a href="//github.com/toddmotto/nofi" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork nofi', 'nofi Fork']);">Fork</a>
</div>

### Disclaimer: HTML5 implementations
The `navigator.onLine` has had some bad press as vendors can't decide on best implementation, some results are a little flakey, but this is an HTML5 API and I'm going to use it, don't hate the player and all that...
