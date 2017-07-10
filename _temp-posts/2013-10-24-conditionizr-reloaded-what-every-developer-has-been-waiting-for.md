---
layout: post
permalink: /conditionizr-reloaded-what-every-developer-has-been-waiting-for/
title: Conditionizr reloaded, what every developer has been waiting for
path: 2013-10-24-conditionizr-reloaded-what-every-developer-has-been-waiting-for.md
tags:
- Conditionizr
- Library
- JavaScript
---

Wouldn't it be ideal to target any browser, mobile browser, mobile device, touch device, screen resolution, operating system and conditionally load assets or tweak your code based on any environments? In a perfect world, Conditionizr wouldn't be needed, but it's here to save your ass...

You can also write logic for different environments using callbacks and simple expressions, or even dynamically load polyfills to rid messy conditional statements. You can also add your own tests to target any other environments/devices you're working on. Keep reading, and open your eyes to Conditionizr v4. It includes a full API rewrite, public API modules and is just 1kb in size (66% reduction). It ships with 5 incredibly fast APIs that make development life seamless.

_Yes, in a perfect world feature detection, blah blah blah, wake up man..._

## Core and APIs

The Conditionizr core is made up of several public APIs.

### .config()
The config API allows you to easily configure your conditional environments, once tests are added. You have a choice of loading conditional scripts, styles and class names per config test, as well as specifying an asset path to where your files are.

{% highlight javascript %}
conditionizr.config({
  assets: '/path/to/my/assets/',
  tests: {
    'safari': ['script', 'style', 'class']
  }
});
{% endhighlight %}

This would then load browser specific tweaks, or you could use the global class override:

{% highlight html %}
<html class="safari">
  <head>
    <script src="path/to/my/assets/js/safari.js"></script>
    <link href="path/to/my/assets/css/safari.css" rel="stylesheet">
  </head>
</html>
{% endhighlight %}

### .add()
Custom tests can be bolted into the Conditionizr core and used with all the APIs, making your conditional coding seamless. Conditionizr will handle all the hard work for you, you just need to provide it a test that returns a boolean, true/false.

{% highlight javascript %}
conditionizr.add('safari', [], function () {
  return /constructor/i.test(window.HTMLElement);
});
{% endhighlight %}

### .on()
Using .on() you can create custom callbacks for when conditional tests return true which are your best bet if you can avoid loading an external script and style, for instance if I’ve added a test for Safari, when a user is running Safari, your callback will run. This is preferred as it saves an HTTP request and improves performance. 

{% highlight javascript %}
conditionizr.on('safari', function () {
  // safari
});
{% endhighlight %}

Conditionizr returns an object for you to also test environment states inside expressions.

{% highlight javascript %}
if (conditionizr.safari) {
  // safari
}
{% endhighlight %}

### .polyfill() and .load()
Polyfill and load each allow you to inject custom assets based on a conditional test. All you need is the external URI, and your predefined conditional tests to declare.

{% highlight javascript %}
conditionizr.polyfill('//html5shiv.googlecode.com/svn/trunk/html5.js', ['ie6', 'ie7', 'ie8']);
{% endhighlight %}

Using the .load() API instead of .polyfill() is purely for naming conventions to differentiate between polyfills and generic assets.

{% highlight javascript %}
conditionizr.load('//cdnjs.cloudflare.com/ajax/libs/hammer.js/1.0.5/hammer.min.js', ['ios']);
{% endhighlight %}
