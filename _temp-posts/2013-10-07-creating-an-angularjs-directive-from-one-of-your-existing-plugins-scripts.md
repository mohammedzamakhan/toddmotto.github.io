---
layout: post
permalink: /creating-an-angularjs-directive-from-one-of-your-existing-plugins-scripts/
title: Creating an AngularJS Directive from one of your existing plugins/scripts
path: 2013-10-07-creating-an-angularjs-directive-from-one-of-your-existing-plugins-scripts.md
tags:
- Plugins
- Directives
- Angular
- JavaScript
---

Writing scripts for your websites or web apps is often a simple process, you write your script, concatenate the file into your main scripts file and it gets pushed into the DOM. Not much to it, but when it comes to AngularJS, they believe in something slightly different...

No _DOM manipulation_ should be carried out inside a Controller, the Controller is where most of your magic happens, a communications channel between your Model data and the browser. It can be tempting to simply whack in an existing script in there (as it'll work just fine), but this goes against Angular's principles.

So here's how to migrate one of your existing scripts or plugins across into a tightly coded AngularJS directive, this also makes code readability and reuse ultra-efficient, as Directives take the strain of repetitive code out the window.

Directives are Angular's answer to Web Components 'Shadow DOM' but are compatible in all browsers (not just cutting edge HTML5 supporting ones) - bringing you the power of the future technology, today. Shadow DOM injects new content based on the element, has its own CSS and JavaScript scope and introduces some incredible behaviour mechanisms, and this is what Directives mimic to bring you this technology today.

<div class="download-box">
  <a href="//toddmotto.com/labs/fluidvids-angular">Demo</a>
  <a href="http://jsfiddle.net/toddmotto/MvGyc">jsFiddle</a>
</div>

Defining a Directive:

Directives are really easy to use once you've set them up. For purposes of this demonstration I'm going to migrate [FluidVids](http://toddmotto.com/fluid-and-responsive-youtube-and-vimeo-videos-with-fluidvids-js) across into an AngularJS Directive.

### Existing code
Here's the existing code for the script, annotated below to show what each part does.

{% highlight javascript %}
window.fluidvids = (function (window, document, undefined) {

  'use strict';

  /*
   * Constructor function
   */
  var Fluidvids = function (elem) {
    this.elem = elem;
  };

  /*
   * Prototypal setup
   */
  Fluidvids.prototype = {

    init : function () {

      var videoRatio = (this.elem.height / this.elem.width) * 100;
      this.elem.style.position = 'absolute';
      this.elem.style.top = '0';
      this.elem.style.left = '0';
      this.elem.width = '100%';
      this.elem.height = '100%';

      var wrap = document.createElement('div');
      wrap.className = 'fluidvids';
      wrap.style.width = '100%';
      wrap.style.position = 'relative';
      wrap.style.paddingTop = videoRatio + '%';
      
      var thisParent = this.elem.parentNode;
      thisParent.insertBefore(wrap, this.elem);
      wrap.appendChild(this.elem);

    }

  };

  /*
   * Initiate the plugin
   */
  var iframes = document.getElementsByTagName( 'iframe' );

  for (var i = 0; i < iframes.length; i++) {
    var players = /www.youtube.com|player.vimeo.com/;
    if (iframes[i].src.search(players) > 0) {
      new Fluidvids(iframes[i]).init();
    }
  }

})(window, document);
{% endhighlight %}

### Directive code
First you need to know how to restructure your code, instead of using a small API to apply your script to each of the elements in your selector/plugin. Remember to apply your changes to the single scoped element only, as Directives are for reused components and therefore can be repeated a lot, so they refer to themselves instead of as a NodeList of elements to loop through.

{% highlight javascript %}
// Module
var myApp = angular.module('myApp', []);

// FluidVids Directive
myApp.directive('fluidvids', function () {

  return {
    restrict: 'EA',
    replace: true,
    scope: {
      video: '@'
    },
    template: '<div class="fluidvids">' +
                '<iframe ng-src="{% raw %}{{ video }}{% endraw %}"></iframe>' +
              '</div>',
    link: function (scope, element, attrs) {
      var ratio = (attrs.height / attrs.width) * 100;
      element[0].style.paddingTop = ratio + '%';
    }
  };

});
{% endhighlight %}

I'll talk through the above for those interested in what the workings are. A Directive returns an Object, which all of the configuration sit inside for that specific Directive. I've used _restrict_ with the value of 'EA', this means either an Element or Attribute. I then use replace to replace the markup in the DOM so that it renders nice and valid. I'm using a scope here as the third property which you can grab as your element name. Using '@' means I'm just using this as a string, which you'll see I've used inside my small _template_ which gets injected in the DOM with the custom video _src_. I'm also using _ng-src_ here which Angular recommend for better browser consistency when dynamically creating _src_ attributes (mainly for legacy browsers, ofc). I then create a small _link_ function which defines any DOM manipulation past the template that needs doing. You can also bind click events any anything else here too.

### Moving JavaScript styles to CSS
Using an Angular Directive allowed me to use less individual DOM manipulation as I split out the JavaScript style objects into using CSS instead:

{% highlight css %}
.fluidvids {
    width: 100%;
    position: relative;
}
.fluidvids iframe {
    border: 0;
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
}
{% endhighlight %}

The video ratio isn't here as this is calculated by the JavaScript and appended to each individual element, that ensures each video gets a custom ratio as intended by any initial iframe embeds.

### Release Candidate errors with cross-domain media
Whilst porting FluidVids over to AngularJS, it was really easy testing until I hit the Release Candidate (version 1.2.0-rc.2). They've actually done this to help you out, but you actually need to whitelist external domains that you'll be retrieving media from. I got this error whilst changing from version 1.0.8 to 1.2.0-rc.2:

{% highlight sh %}
Error: [$interpolate:interr] http://errors.angularjs.org/undefined/$interpolate/interr?p0=%7B%7B%20src%2…%24sce%2Finsecurl%3Fp0%3D%252F%252Fplayer.vimeo.com%252Fvideo%252F23919731
{% endhighlight %}

Turns out, after a quick Google search I needed to whitelist the domains I use inside AngularJS, a somewhat protectively smart move (even though slightly irritating error) by the team. After some searching I found somebody had commented the following which whitelists all domains for quick and ease use:

{% highlight javascript %}
myApp.config(function ($sceDelegateProvider) {
  $sceDelegateProvider.resourceUrlWhitelist(['.*']);
});
{% endhighlight %}

### Custom Elements or Attributes (E or A, or both!)
If you're advocating HTML5 Web Components and developing for HTML5 browsers only, then you might as well start creating your own elements inline with the HTML5 [Web Components spec](http://www.w3.org/TR/2013/WD-components-intro-20130606).

Custom element:
{% highlight html %}
<fluidvids video="//player.vimeo.com/video/23919731" height="281" width="500"></fluidvids>
{% endhighlight %}

As an attribute:
{% highlight html %}
<div fluidvids video="//player.vimeo.com/video/23919731" height="281" width="500"></div>
{% endhighlight %}

Depending on your team and setups, it might be easier using one method or the other. They're not too indifferent, but I feel Web Components offer oddly better semantics - despite their loose markup that let the developer decide.

Web Components introduces custom attributes too, use - but _don't_ abuse :)

<iframe width="100%" height="300" src="http://jsfiddle.net/toddmotto/MvGyc/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
