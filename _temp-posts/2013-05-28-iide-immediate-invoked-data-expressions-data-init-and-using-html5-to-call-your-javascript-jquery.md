---
layout: post
permalink: /iide-immediate-invoked-data-expressions-data-init-and-using-html5-to-call-your-javascript-jquery/
title: IIDE, Immediate-Invoked-Data-Expressions, data-init and using HTML5 to call your JavaScript/jQuery
path: 2013-05-28-iide-immediate-invoked-data-expressions-data-init-and-using-html5-to-call-your-javascript-jquery.md
tags:
- Opinion
- JavaScript
---

There's something that's been at the back of my mind ever since I've started writing JavaScript and jQuery, and I think I've finally got it down on paper - so I'm going to introduce a new method of writing your JavaScript which utilises some HTML5 technology.

Firstly, I've been writing a lot about HTML5 data-&#42; attributes recently, and with good reason. They're definitely the future of HTML, and with data-binding, HTML templating and some incredible advances announced at this year's Google I/O, it's all come together for me to start a new method of writing, configuring and executing JavaScript.

I've coined this term IIDE (Immediate-Invoked-Data-Expression), derived from IIFE (Immediate-Invoked-Function-Expressions) - which means a _closed_ JavaScript function that calls itself, executing when it parses. IIDE is the same, but executes based on data-&#42; states instead. I'll talk more about IIDE in a second, but here's what drove the idea:

Let's take a made up slider jQuery plugin:
{% highlight html %}
<script src="js/vendor/jquery.min.js"></script>
<script src="js/slider.min.js"></script>
<script>
  $(function () {
    $('#slider').mySlider()
  })
</script>
{% endhighlight %}

We load the files, and call the function. This is a rather pointless exercise I feel nowadays, and has no benefit - not to mention the drastic challenge of maintaining your plugins configuration, you're never entirely sure what's going on and every plugin is different!

So I'm proposing to drop this lame practice of 'calling' our functions/plugins, and getting more intelligent. Queue data-&#42; attributes!

### Introducing data-init
When we typically write a function and call it, we're *init*ialising it! As HTML5 moves closer to JavaScript by the day, I'm talking about MVC frameworks like Knockout.js and AngularJS here - HTML templating is on the rise, it's all about components, reusable parts and shadow DOM. Static HTML is a thing of the past and we really need to take advantage of the superiority of modern browsers and even JavaScript/jQuery at a base level.

So what is data-init? It's a way of calling your function when it exists, and also being able to configure a plugin should it have configurable options. Plugins are great for reusable code, ones that actually have purpose, but a good developer will bake this in whilst building their script(s).

Instead of this:
{% highlight html %}
<script>
  $(function () {
    $('#slider').mySlider()
  })
</script>
{% endhighlight %}

We do this:
{% highlight html %}
<div class="slides" data-init="slides"></div>
{% endhighlight %}

If an element exists with a data-&#42; attribute with the value of _slides_, it runs the function. You may think this is weird, but it's really very sensible. I spend my days writing Object-Orientated front-end web software and this really makes a difference to productivity and reusable components.

This is great for a few reasons, we bake in the function calls to the scripts/plugins themselves so they only run when the elements required are there, which means the data-&#42; calls are not bound to our HTML. Before, calling your plugin/script on a particular element was bound/restricting that plugin numerous times to one use, unless you called it multiple times (unproductive). Sometimes you'll get Console errors saying things like _Cannot set property X of null_ - which means the element probably doesn't exist on the page. The beauty of this is that it will only fire when it exists (run the element check inside the plugin/script).

### JSON Configuration
You may have heard of JSON, if not it stands for JavaScript Object Notation and looks a little like this (example):

{% highlight javascript %}
{"menu": {
  "id": "file",
  "value": "File",
  "popup": {
    "menuitem": [
      {"value": "New", "onclick": "CreateNewDoc()"},
      {"value": "Open", "onclick": "OpenDoc()"},
      {"value": "Close", "onclick": "CloseDoc()"}
    ]
  }
}}
{% endhighlight %}

It's derived from JavaScript for representing data structures and arrays and objects - and this is where things get interesting. In web application development, we automate/create dynamic HTML as much as possible. If I've got an image slider (for example) you're probably used to seeing this:

{% highlight html %}
<div class="slides">
  <img src="img/slides/img-1.jpg" alt="">
  <img src="img/slides/img-2.jpg" alt="">
  <img src="img/slides/img-3.jpg" alt="">
  <img src="img/slides/img-4.jpg" alt="">
  <img src="img/slides/img-5.jpg" alt="">
</div>
{% endhighlight %}

This is fine, but what about when the website scales, or you want to change your view (HTML)? If you add or change a class name, you're going to have to add it to each, which is repetitive work and unnecessary. For a simple slider this is fine, but it's not maintainable when you're thinking big or HTML agnostic development.

Coming back to JSON now, let's use HTML5 data-&#42; attributes to define an array of images inside a JSON array. The beauty of JSON arrays/objects is that they can be manually typed (like I have below), or dynamically fed down from a server - perfect for so many use cases.

In comes JSON array of images inside an attribute (I've named this data-slides):

{% highlight html %}
<div class="" data-init="slides" data-slides='{
  "imgs" : [
    "img/slides/img-1.jpg",
    "img/slides/img-2.jpg",
    "img/slides/img-3.jpg",
    "img/slides/img-4.jpg",
    "img/slides/img-5.jpg"
  ]
}'></div>
{% endhighlight %}

This one HTML element defines the plugins role, to initiate the slides (data-init="slides") and secondly define some more data to feed off, an array of images.

I can then setup a script to create an image for each item in the data-&#42; array:

{% highlight javascript %}
(function ($) {
  var slider = $('[data-init="slides"]')
  if (slider.length) {
    var slides = slider.data('slides').imgs
    $.each(slides, function (index, value) {
      slider.prepend('<img src="' + value + '" alt="">')
    })
  }
})(jQuery)
{% endhighlight %}

Which then outputs:

{% highlight html %}
<div class="" data-init="slides" data-slides='{"imgs" : ["img/slides/img-1.jpg","img/slides/img-2.jpg","img/slides/img-3.jpg","img/slides/img-4.jpg","img/slides/img-5.jpg"]}'>
  <img src="img/slides/img-1.jpg" alt="">
  <img src="img/slides/img-2.jpg" alt="">
  <img src="img/slides/img-3.jpg" alt="">
  <img src="img/slides/img-4.jpg" alt="">
  <img src="img/slides/img-5.jpg" alt="">
</div>
{% endhighlight %}

### Extending JSON configurations
We could move an entire plugin's configuration into a JSON data structure, for example, I can create a namespace for my plugin, dynamically add a class to each slide, and again loop through my array of images:

{% highlight javascript %}
<div class="" data-init="slides" data-slides='
  {"slides" : {
    "namespace" : "my-slides",
    "class"     : "slide-item",
    "imgs" : [
      "img/slides/img-1.jpg",
      "img/slides/img-2.jpg",
      "img/slides/img-3.jpg",
      "img/slides/img-4.jpg",
      "img/slides/img-5.jpg"
    ]
  }
}'>
{% endhighlight %}

The choices are unlimited, and I don't see why you wouldn't do this. Alternatively, you could move the JSON configuration into the same file as the script itself too. But for ultimate reuse, we'll be looping through the data-&#42; attributes and running the identical function, so it's good that the model data is bound to the view.

### Real examples of _why_ data-init
I'm currently writing some new open source projects that invoke this method of development, but here's a real life use case. In the past I've used plugins that create 'responsive type' - aka shrink the browser and the text stays the width of the window - very nice. But here's a huge use case for the popular [FitText](http://fittextjs.com) plugin, to be configured in the view rather than a pointless call script!

The redundant call (taken from FitText website):
{% highlight javascript %}
$("h1").fitText(0.273);
$(".download").fitText(2);
{% endhighlight %}

I don't know about you, but calling a plugin twice, is kiiiiinda poor development.

What they really should have done:

{% highlight html %}
<html>
  <head>
  <script src="js/vendor/jquery.min.js"></script>
  <script src="js/fittext.min.js"></script>
  </head>
  <body data-init="fittext">
    <h1 data-fittext='{"font-size" : "0.273"}'>FitText</h1>
    <a href="#" data-fittext='{"max-font-size" : "2"}'>Download</a>
  </body>
</html>
{% endhighlight %}

Immediately-Invoked-Data-Expression with data-init function calling. Remember, valid JSON contains double quotes _"like" : "this"_, which means you'll want to use single quotes on your HTML.
