---
layout: post
permalink: /hacking-the-html5-video-element-with-suave-js/
title: Hacking the HTML5 &lt;video&gt; element with Suave.js
path: 2013-07-28-hacking-the-html5-video-element-with-suave-js.md
tags:
- Library
- HTML5
- JavaScript
---

Suave, for elegant HTML5 videos (how they should have been). Suave was built to re-engineer the unstructured mess the HTML5 &lt;video&gt; tag presents. Suave cleverly takes all the strain of nested &lt;source&gt; tags and file types away from you allowing you to code an HTML5 video in just one line (it's also <1KB).

HTML5 video is awesome, what's not awesome is the markup. The semantics and unmodular approach with HTML5 video upset me and I was inspired to fix it. So here's my idea, which is very modular and works in all browsers supporting HTML5 video.

<div class="download-box">
  <a href="//toddmotto.com/labs/suave" onclick="_gaq.push(['_trackEvent', 'Click', 'Demo suave', 'suave Demo']);">Demo</a>
  <a href="//github.com/toddmotto/suave/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download suave', 'Download suave']);">Download</a>
  <a href="//github.com/toddmotto/suave" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork suave', 'suave Fork']);">Fork</a>
</div>

### The problem
HTML5 video is a brilliant invention, cleverly injecting useful pieces of Shadow DOM in for us so we no longer need to code in video controls and other funky buttons. It seems someone had a little too much coffee when thinking about a solution for the markup (don't get me started on the responsive images &lt;picture&gt; element). Why must we configure (manually) a video, what if there are a vast amount of videos on the page?

For those wondering what I'm really digging into, let's take a cross-browser &lt;video&gt; tag with some &lt;source&gt; tags inside:

{% highlight html %}
<video>
  <source src="video/trailer.mp4" type="video/mp4">
  <source src="video/trailer.ogv" type="video/ogv">
  <source src="video/trailer.webm" type="video/webm">
</video>
{% endhighlight %}

I love the naming conventions here, source 'src'. Source source (they definitely had too much coffee by this point). But seriously, what on earth happened here? HTML5 is meant to be intelligent and in my eyes this is a little dumb. What happens if I suddenly change the file name and/or directories, I've then got to change it multiple times... Crazy.

### My solution
So here's where Suave comes in. Thanks to my little script, you no longer have to worry about the above catastrophe and can code an HTML5 video with just one line of code (this is proper valid HTML5, too!):

{% highlight html %}
<video data-src="video/mymovie.{mp4, ogv, webm}"></video>
{% endhighlight %}

All you need to do is feed it the file extensions you require for each video inside a _data-*_ attribute, easy. Suave is fully modular as well, call it as many times on the page and it'll just keep doing its thing. What I also like about this solution is that I'm enhancing HTML5, _with_ HTML5. Of course some people will disagree and say I'm missing a few codecs, lost my mind and am hashing out strange ideas, but my project would be finished ontime and save countless future hours.

I've been using Grunt.js a lot recently and I love how you can simply include some curly braces to say 'or this too', so that's where the idea came from to simplify an overcomplicated system. This is fully semantic stuff too, if anything this improves the semantics of the &lt;video&gt; tag! With the current HTML5 spec, if you only have one HTML format, you can do this:

{% highlight html %}
<video src="video/mymovie.mp4"></video>
{% endhighlight %}

And that's where the simplicity of my idea came from. Sure it isn't how the HTML5 spec intended it, but remember this stuff is still new to everyone and remember this is still a huge work in progress.

### JavaScript
For those interested in how Suave works, here's a break down of the script:

{% highlight javascript %}
window.suave = ( function ( window, document, undefined ) {

  'use strict';

  /*
   * Constructor function
   */
  var Suave = function ( elem ) {
    this.elem = elem;
  };

  /*
   * Prototypal setup
   */
  Suave.prototype = {

    init : function () {

      var dataAttr = this.elem.getAttribute('data-src');
      var videoSource = dataAttr.match(/^([^]+){/)[1];
      var fileExts = dataAttr.match(/{([^]+)}$/)[1].toString().replace(/\s/g, '').split(',');
      
      for (var i = 0; i < fileExts.length; i++) {
        var extension = fileExts[i];
        var source = document.createElement('source');
        source.src = videoSource + extension;
        source.type = 'video/' + extension;
        this.elem.appendChild(source);
      }

    }

  };

  /*
   * Initiate the plugin
   */
  [].forEach.call(document.querySelectorAll('video[data-src]'), function (suave) {
    new Suave(suave).init();
  });

})( window, document );
{% endhighlight %}

From the top, I create the constructor function, which I pass the current element into (passed in at the bottom loop). This then has some internal Prototype workings that grab the _data-src_ attribute (we're looking at the _init_ function here).

First I grab the _videoSource_ from the attribute, which uses a RegExp to capture the file path and file name, but not the extension.

Next I grab the file extensions (fileExts) which captures everything inside the curlies {}. From here, I use the _.toString()_ method, which converts the array sent back from _.match()_ to a string (you guessed it), from here, I _.replace()_ any whitespace to get a clean array for adding the file extensions, and then use the _.split(',')_ method to split the string by commas which then returns a new array. I then loop through that array of file extensions and create the right amount of &lt;source&gt; tags, populating them with the necessary _src_ and _type_ attributes.

At the bottom, I then hook into the Array.prototype (but use an empty array shorthand to access this) and loop through all _video[data-src]_ tags, which will hold our Suave.js videos! Inside this loop, I pass in the current element and create a new _Suave_ instance to the current item.

<div class="download-box">
  <a href="//toddmotto.com/labs/suave" onclick="_gaq.push(['_trackEvent', 'Click', 'Demo suave', 'suave Demo']);">Demo</a>
  <a href="//github.com/toddmotto/suave/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download suave', 'Download suave']);">Download</a>
  <a href="//github.com/toddmotto/suave" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork suave', 'suave Fork']);">Fork</a>
</div>

Feedback welcome :)
