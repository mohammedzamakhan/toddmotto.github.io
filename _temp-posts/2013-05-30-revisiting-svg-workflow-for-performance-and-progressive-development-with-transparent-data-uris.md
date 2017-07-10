---
layout: post
permalink: /revisiting-svg-workflow-for-performance-and-progressive-development-with-transparent-data-uris/
title: Revisiting SVG workflow for performance and progressive development with transparent data URIs
path: 2013-05-30-revisiting-svg-workflow-for-performance-and-progressive-development-with-transparent-data-uris.md
tags:
- SVG
- Performance
- JavaScript
---

A few months ago I covered a range of [SVG techniques](//toddmotto.com/mastering-svg-use-for-a-retina-web-fallbacks-with-png-script) that proved to be a great progression for developers looking to 'get into' SVG development. For those who are new to web development or SVG in general, give it a good read and then drop back to this article to help take you to the next phase.

This article sums up my SVG and HTML5 workflow stemming from my previous article, and how it's evolved over the last few months to improve furthermore in terms of speed, reliability and new advances in techniques and thinking.

### Feature detect
First thing's first, a proper progressive enhancement technique will require some form of base code that allows for all users to view some form of feature. When dealing with SVG, we split this into two - SVG for enhanced, and PNG for the base layer.

Since the last feature detect, I've optimised the JavaScript even further. Here's where I was previously at, and lots of developers were using my feature detection script and SVG fallback, which ripped the _.svg_ file extension back to _.png_ for inline images - but I'm not sure this is the way to continue anymore for a few reasons (no, don't worry your old sites are fine). First, let's talk a look at the old script:

{% highlight javascript %}
function supportsSVG() {
    return !! document.createElementNS && !! document.createElementNS('http://www.w3.org/2000/svg','svg').createSVGRect;  
}
if (!supportsSVG()) {
    var imgs = document.getElementsByTagName('img');
    var dotSVG = /.*\.svg$/;
    for (var i = 0; i != imgs.length; ++i) {
        if(imgs[i].src.match(dotSVG)) {
            imgs[i].src = imgs[i].src.slice(0, -3) + 'png';
        }
    }
}
{% endhighlight %}

This grabbed all images on the page, tested to see if they were using an _.svg_ file extension and made the necessary adjustments should the feature detect fail. There are two reasons why my next script is better, the first, it's much smaller. Secondly, inline SVGs inside &lt;img&gt; tags prove to be slightly quirky in older rendering engines. I had a few emails from kind folk letting me know my logo looked a little squashed on their slightly older Android device - a quick screenshot confirmed it. For those who are unsure of the backgrounds of SVG, it's an XML-based file which essentially is markup and co-ordinates which combined with colours and dimensions - creating a scalable shape. You can edit the attributes and SVG elements, and I did look into a better understanding of these to overcome this quirky display issue, but to no avail. I also managed to see the same logo on an older iPhone 3GS, to which the same was happening - squashed!

Time for a new idea:

{% highlight javascript %}
!function () {
  function supportsSVG() { return !!document.createElementNS && !! document.createElementNS('http://www.w3.org/2000/svg', 'svg').createSVGRect }
    if (supportsSVG()) document.documentElement.className += ' svg'
    else document.documentElement.className += ' no-svg'
}()
{% endhighlight %}

This doesn't do any DOM manipulation, just simply adds an _svg_ class to the &lt;html&gt; element, and _no-svg_ for non-supporting.

To be fair, if we're only going to use _svg_ progressive enhancement techniques, we can ommit the _no-svg_ 'else' statement to get the detect even shorter:

{% highlight javascript %}
!function () {
  function supportsSVG() { return !!document.createElementNS && !! document.createElementNS('http://www.w3.org/2000/svg', 'svg').createSVGRect }
    if (supportsSVG()) document.documentElement.className += ' svg'
}()
{% endhighlight %}

### blank.gif - the transparent window
I'm employed to build the UI/front-end of software for Google Enterprise customers and we work closely with Google - this is great as I can keep in touch with their modern software and development techniques. At the time of seeing the aforementioned squashed logo, I'd noticed Google had been using a new form of image and icon development, adding a background image to an inline transparent image, a _blank.gif_ to be precise:

{% highlight html %}
<img src="blank.gif" class="chromium">
{% endhighlight %}

With the accompanying CSS for example:

{% highlight css %}
.chromium {
  background:url(//ssl.google.com/imagepath.png) no-repeat;
  width:250px;
  height:250px;
}
{% endhighlight %}

I really liked this as it suddenly made some sense in a crazy way. The _blank.gif_ image was 1px by 1px in dimensions, and literally stretched into the width and height it's needed to be by the CSS, not the physical HTML dimensions - really smart.

The above 'Google' technique I like to explain to others as a transparent window image with a background image, essentially you're looking through a transparent image to see a background image. This is also amazing for icons...

### Why a clear &lt;img&gt; tag, over the &lt;i&gt; element for icons?
I've stopped using &lt;i&gt; for icons, it really isn't a good element. Its semantic meaning is that the contents should be italic, yes it starts with 'i' for 'icon' so I assume this is why its popularity has risen, but its semantic use is incorrect and it should be swapped for the _blank.gif_ technique, as really - icons are images too.

### Data URIs
Instead of using a _blank.gif_ physical image, we could trump Google a little and create a transparent Data URI image out of it, and embed the image data inline. This is done to save uploading and creating a transparent image as well as to save an HTTP request:

{% highlight html %}
<img src="data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7" class="chromium">
{% endhighlight %}

It might look weird at first, but after a while you get used to it, and it starts to look pretty in a weird kind of way. If you're unsure what a Data URI is, please Google it, and then use [this tool](//websemantics.co.uk/online_tools/image_to_data_uri_convertor) for all your conversions. Data URIs are usually encoded in a base64 format, which syntax looks like so:

{% highlight html %}
data:[<mediatype>][;base64],<data>
{% endhighlight %}

### Bolting on SVG enhancement
So far, I've covered a better SVG detect, and a better way to use icons and images if you're wanting to use them as background images, so why not merge this with an SVG enhancement technique. Let's start with some HTML, and pretend that the page has rendered and the supporting browser has SVG support:

{% highlight html %}
<html class=" svg">
  <head>
    <script>
      !function () {
        function supportsSVG() { return !!document.createElementNS && !! document.createElementNS('http://www.w3.org/2000/svg', 'svg').createSVGRect }
          if (supportsSVG()) document.documentElement.className += ' svg'
      }()
    </script>
    <style>
      .chromium {
        background:url(//ssl.google.com/imagepath.png) no-repeat;
        width:250px;
        height:250px;
      }
    </style>
  </head>
  <body>
    <img src="data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7" class="chromium">
  </body>
</html>
{% endhighlight %}

### Script before Style (just this time)
Advancing from the above markup, the &lt;html&gt; tag has an _svg_ class and the feature detect comes _before_ the &lt;style&gt; tag. You've probably been slapped on the wrist before for putting scripts before styles (please never do this with _.js_ files!) - but for feature detects this is perfect. The script is not running on document ready (DOM ready) as we want to run it as fast as possible and get the feature detect result for the CSS to do its thing.

When we now add SVG, this will benefit our performance too. If SVG is supported, the SVG override in the CSS will take action _before_ the PNG fallback is loaded, meaning this saves an HTTP request and pointless image download. We don't want to load extra images and override them with prettier SVGs - just one is a perfect scenario.

### SVG transparent Data URIs
Now we add the SVG optimisation:

{% highlight css %}
.chromium {
  background:url(//ssl.google.com/imagepath.png) no-repeat;
  width:250px;
  height:250px;
}
.svg .chromium {
  background:url(//ssl.google.com/imagepath.svg) no-repeat;
}
{% endhighlight %}

Nice and easy isn't it! This way, all the optimisation is done out of sight in our CSS file, and nothing can go wrong with our HTML, and should JavaScript error or not run correctly, we'll have that progressive base layer PNG image as a worst-case scenario.

SVG optimisation doesn't stop there however...

### SVG and CSS3 Background Sizing, solving responsive design issues
One of the biggest challenges with responsive development is changing your image widths and heights and backgrounds to 1) fit the viewport of the device you're optimising for, and 2) to use the correct image enhancing technique.

With the introduction of the new Chrome Pixel, x2 optimisation isn't enough - SVG is key. To make sure our SVG responds correctly and we don't get any squashing, we let CSS3 background-size take hold:

{% highlight css %}
.chromium {
  background:url(//ssl.google.com/imagepath.png) no-repeat;
  width:250px;
  height:250px;
}
.svg .chromium {
  background:url(//ssl.google.com/imagepath.svg) no-repeat;
  background-size:250px 250px;
}
{% endhighlight %}

This technique is also good as the background-size property only applies to the SVG, so worst case if the browser supports SVG and the CSS3 property, or something went wrong, the PNG wouldn't be strangely sized.

### Live demo
To view a live demo of the above technique (with some proper images), view my [jsFiddle](//jsfiddle.net/toddmotto/67BEq) and see what you think. I encourage you to Inspect Element, and either delete the SVG background image (or untick it using dev tools), and watch the PNG fallback load in, proof that we've only loaded one image.

### Sass
I'm a recent convert to Sass, and I love it. It's really easy to optimise all your CSS inside one selector wrap by doing the following:

{% highlight css %}
.chromium {
  background:url(//toddmotto.com/img/posts/chromium.png) no-repeat;
    width:250px;
    height:250px;
  .svg & {
    background:url(//toddmotto.com/img/posts/chromium.svg) no-repeat;
    background-size:250px 250px;
  }
}
{% endhighlight %}

Using the ampersand _&_ after the _.svg_ is what gets this working, which nicely compiles to this:

{% highlight css %}
.chromium {
    background:url(//toddmotto.com/img/posts/chromium.png) no-repeat;
    width:250px;
    height:250px;
}
.svg .chromium {
    background:url(//toddmotto.com/img/posts/chromium.svg) no-repeat;
    background-size:250px 250px;
}
{% endhighlight %}

Remember, it's important to put the SVG override in your CSS _after_ the PNG declaration, this will prevent any priority order issues in browsers and ensure your SVG always comes out ontop when supported.

### Browser support
Data URIs are supported in IE8 and up, which means for IE7 inclusive development you'll want to use the Google 'blank.gif' technique instead of a Data URI! If you're IE8+, then you could go the whole hog and convert all your icons, patterns and logos to Data URIs. Though, IE8 doens't like Data URIs above 32kb, so keep it trim. I use [TinyPNG](//tinypng.org) to compress my images, it often saves between 50%-80% of image size.

### Bonus: actually embedding SVG+XML inside CSS!
As crazy as it sounds, I recently found this out; you can embed an SVG image inside CSS - essentially embedding XML markup as a background image (which is actually the stuff _inside_ the .svg file). This is really the next step in Data URIs, but just because we can, should we:

{% highlight css %}
.svg .chromium {
    background:url('data:image/svg+xml;utf8,<svg> <!-- SVG + XML HERE! --> </svg>');
}
{% endhighlight %}
