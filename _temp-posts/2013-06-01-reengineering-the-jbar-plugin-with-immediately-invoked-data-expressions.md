---
layout: post
permalink: /reengineering-the-jbar-plugin-with-immediately-invoked-data-expressions/
title: Re-engineering the jBar plugin with Immediately-Invoked-Data-Expressions
path: 2013-06-01-reengineering-the-jbar-plugin-with-immediately-invoked-data-expressions.md
tags:
- Library
- JavaScript
---

Earlier this week [I wrote about IIDE](//toddmotto.com/iide-immediate-invoked-data-expressions-data-init-and-using-html5-to-call-your-javascript/jquery/), Immediately-Invoked-Data-Expressions. Not an official term as such but an insight into the explanation of what our code is actually doing when utilising HTML5 advancements in plugin logic. IIDE was about getting smarter, using JSON for data-binding and bringing JavaScript and HTML5 closer to work harder for an enhanced 'view' approach. I've re-engineered my most popular plugin, the jBar into IIDE format and I'm going to share with you the process.

<div class="download-box">
  <a href="//toddmotto.com/labs/jbar" onclick="_gaq.push(['_trackEvent', 'Click', 'Demo jBar', 'jBar Demo']);">Demo</a>
  <a href="//github.com/toddmotto/jbar/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download jBar', 'Download jBar']);">Download</a>
  <a href="//github.com/toddmotto/jbar" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork jBar', 'jBar Fork']);">Fork</a>
</div>

Before we can see what's new, we need to do a comparison in the plugins. I won't go script specific, but configuration specific. Generally, when you download/use a plugin, there are three steps:

1. Adding the JavaScript
2. Adding the HTML
3. Adding a random script tag/file which calls all plugins and perhaps some custom code

And now, using the IIDE methodology we can cut out this unnecessary third step:

1. Add the JavaScript
2. Add and configure the HTML

Done.

### Old plugin development
Here's the old plugin configuration for the jBar version 1.0.0:

{% highlight html %}
<script src="jquery.js"></script>
<script src="jbar.min.js"></script>
<script>
  $(function() {
      $.jBar({
          type            : 'fixed',
          delay           : '1000',
          backgroundColor : '#DB5903',
          borderColor     : '#FFF',
          buttonTextColor : '#FFF',
          buttonColor     : '#333',
          buttonColorHover: '#222',
          calltoAction    : 'jBar Plugin! A simple and lightweight notification banner.',
          buttonText      : 'Download it!',
          buttonLink      : 'http://www.toddmotto.com'
      });
  });
</script>
{% endhighlight %}

Sure it looks okay, but it's not needed at all. We pass the plugin a bunch of _options_ which are then parsed through the plugin logic and it outputs your code entirely.

### No more default options
One thing you're probably used to seeing is jQuery plugin _defaults_ that are actually inside the plugin script itself which you essentially override with your own _options_ when you call the script. Using the IIDE methodology, I decided this wasn't at all necessary and completely ignored the need for it. Why? Because it's not really needed either - the defaults are in the markup for one, and secondly, we can check to see if the user has added the JSON data to the data-&#42; attribute and run a function if necessary - light work.

### New plugin development + config
I'm an advocate for ideas and changing things, not without good reason though, but generally because things make sense and following the direction of the industry. I really see this methodology pushing the way for future plugin development, and web development in general. I use IIDE for a lot of my application development work, with data-&#42; attributes the possibilities are endless.

Here's the new configuration, the HTML with a data-&#42; attribute named _data-init="jbar"_. In my previous article on IIDE, I mention using data-init to boot plugins is a fantastic way to get them initialised. Alongside the _data-init_, I have _data-jbar_ with a JSON config of my plugins options. This makes the process of integrating the plugin much easier for website authors (who generally find it difficult to integrate options and such into their page). It also makes things a lot easier with avoiding JavaScript errors. The plugin is created in the script file, all the intelligent work is done, we don't have to call it, add a script configuration to a particular element, we also don't have to worry about the global _.jbar_ class conflicting with other classnames that are potentially appended to it as all JavaScript logic is run through the data-&#42; attribute - completely separate. It would also be quite difficult to get this to error, whereas when dealing with live JavaScript, sometimes the positioning of your script can have an erroring effect.

{% highlight html %}
<div class="jbar" data-init="jbar" data-jbar='{
  "message" : "jBar, re-imagined. Get version 2.0.0 now!",
  "button"  : "Download",
  "url"     : "http://toddmotto.com/labs/jbar/jbar_v2.zip",
  "state"   : "closed"
}'></div>
{% endhighlight %}

So what does the above do? It's essentially some really basic JSON that we feed off the parameters to add data to our script, which in turn outputs the plugin that you need. These are the only four options that come with the jBar version 2.0.0. I know some of you requested to have the jBar at the bottom as well, but in terms of usability it's not great. However, all the styling inside the jBar is controlled via a _CSS_ document now, not injected into _style_ tags - so it's achievable with some CSS changes.

What do the options mean?
1. message: the jBar call-to-action message
2. button: the button message
3. url : where you'd like the button link to fire
4. state: choose the initial state of the jBar, open or closed are the two parameters

Now you know the ins and outs of the new plugin configuration, you can easily add it to your page like so:

{% highlight html %}
<script src="//ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
<script>window.jQuery || document.write('<script src="js/vendor/jquery-1.9.1.min.js"><\/script>')</script>
<script src="js/jbar.js"></script>

<!-- jbar -->
<div class="jbar" data-init="jbar" data-jbar='{
  "message" : "jBar, re-imagined. Get version 2.0.0 now!",
  "button"  : "Download",
  "url"     : "http://toddmotto.com/labs/jbar/jbar_v2.zip",
  "state"   : "closed"
}'></div>
<!-- /jbar -->
{% endhighlight %}

Included in the download is a local fallback to jQuery should the Google APIs CDN version not fire or be available, this is good practice for all external JavaScript that is hosted on a CDN elsewhere.

### Redeveloped + Data API
Now we'll take a look at the script:

{% highlight javascript %}
!function(window, $, undefined){

  'use strict'
  
  // jBar
  var jBar = function (elem) {
    this.elem = elem
    this.$elem = $(elem)
    this.jsonConfig = this.$elem.data('jbar')
  }
  
  // prototype
  jBar.prototype = {
    init: function () {
      this.config = $.extend({}, this.defaults, this.jsonConfig)
      this.construct().printMessage().createButton().removeData().togglejBar()
    },
    construct: function () {
      this.$elem.before(
        $('<div class="jbar-push"></div>' +
          '<a href="#" class="jbar-down-toggle">' +
          '<img src="data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7" class="jbar-down-arrow" alt=""></a>')
      )
      this.$elem.append(
        $(
          '<div class="jbar-wrap"></div><a href="#" class="jbar-up-toggle">' +
          '<img src="data:image/gif;base64,R0lGODlhAQABAIAAAAAAAP///yH5BAEAAAAALAAAAAABAAEAAAIBRAA7" class="jbar-up-arrow" alt=""></a>'
        )
      )
      return this
    },
    printMessage: function () {
      if (this.jsonConfig.message) {
        this.$elem.children('.jbar-wrap').append(
          $('<p>' + this.jsonConfig.message + '</p>')
        )
      }
      return this
    },
    createButton: function () {
      if (this.jsonConfig.button && this.jsonConfig.url) {
        this.$elem.children('.jbar-wrap').append(
          $('<a href="' + this.jsonConfig.url + '" class="jbar-button">' + this.jsonConfig.button + '</p>')
        )
      }
      return this
    },
    removeData: function () {
      if (this.jsonConfig) {
        this.$elem.removeAttr('data-jbar')
      }
      return this
    },
    togglejBar: function () {

      // toggle variables
      var $this   = this.$elem
      var $push   = $('.jbar-push')
      var $toggle = $('.jbar-down-toggle')
      var $toggles = $('.jbar-down-toggle, .jbar-up-toggle')
      var clicks;
      
      // json open and closed states
      if (this.jsonConfig.state === 'closed') {
        $this.add($push).css({
          'marginTop' : - ($this.outerHeight())
        })
        $push.css({
          'height' : ($this.outerHeight())
        })
        $toggle.css({
          'visibility' : 'visible'
        })
        setTimeout(function () {
          $this.add($push).css({
            'display' : 'block'
          })
        }, 500)
      } else if (this.jsonConfig.state === 'open') {
        $toggle.css({
          'marginTop' : - ($toggle.outerHeight() + 5)
        })
        $this.add($push).css({
          'display' : 'block'
        })
        $push.css({
          'height' : $this.outerHeight(),
        })
        setTimeout(function () {
          $toggle.css({
            'display' : 'block',
            'visibility' : 'visible'
          })
        }, 500)
        $this.data('clicks', !clicks)
      }
      
      // toggle click handlers
      $toggles.on('click', function (e) {
        
        // global scope for JSON states
        clicks = $this.data('clicks')
        
        // data clicks
        if (!clicks) {
          $this.add($push).css({
            'marginTop' : '0'
          })
          $toggle.css({
            'marginTop' : - ($this.outerHeight() + 5)
          })
        } else {
          $this.add($push).css({
            'marginTop' : - ($this.outerHeight())
          })
          $toggle.css({
            'marginTop' : '0'
          })
        }
        
        // set data
        $this.data('clicks', !clicks)
        
        // stop anchor click
        e.preventDefault()
        
      })
    }
  }
  
  // merge defaults
  jBar.defaults = jBar.prototype.defaults
  
  // jBar plugin logic
  $.fn.jBar = function () {
  
    return this.each(function () {
      new jBar(this).init()
    })
  
  }
  
  // global
  window.jBar = jBar
  
  // IIDE immediate-invoked-data-expression
  $(function () {
  
    // if the validator is set to initialise
    if($('[data-init]').data('init') === 'jbar') {
    
      // run jBar based on JSON data
      $('[data-jbar]').jBar()
    
    }
    
  })

}(window, jQuery);
{% endhighlight %}

The plugin doesn't use a single line of code from version 1.0.0, it's rewritten from the ground up. I've used the JavaScript _prototype_ method for extending and creating my own functions, which prove to work really well with jQuery and the _$.extend()_ functionality.

The plugin is wrapped in a protective wrapper, passing _jQuery_ into it, but what I want to talk about specifically is the bond between jQuery, the plugin and our JSON.

First, I've setup a small handler to grab the element and pass the data back to he plugin:

{% highlight javascript %}
var jBar = function (elem) {
  this.elem = elem
  this.$elem = $(elem)
  this.jsonConfig = this.$elem.data('jbar')
}
{% endhighlight %}

You'll notice at the end we have _this.jsonConfig_ which uses the _.data()_ API from jQuery (conveniently parsing our JSON for us - jQuery will recognise JSON inside data-&#42; attributes and return it).

To get the message that you'd specify in the HTML, we'd now simply do this:

{% highlight javascript %}
console.log(this.jsonConfig.message)
{% endhighlight %}

Which would log the output in the JavaScript console for development testing. Here's a quick example of how I've used this inside the plugin:

{% highlight javascript %}
printMessage: function () {
  if (this.jsonConfig.message) {
    this.$elem.children('.jbar-wrap').append(
      $('<p>' + this.jsonConfig.message + '</p>')
    )
  }
  return this
}
{% endhighlight %}

A little bonus, you can see at the end I've added _return this_. This is very similar to how jQuery works when chaining functions and methods. So I can do this:

{% highlight javascript %}
this.construct().printMessage().createButton().removeData().togglejBar()
{% endhighlight %}

The Data API included in the plugin is where I think things will/need to progress in plugin and script development, I'm not running the jBar plugin script on _document ready_ (DOM ready), but inside the plugin itself, I am initialising it with a DOM ready handler, which runs a check to see if our _data-init="jbar"_ attribute exists, and if so, I grab it and run the jBar on it. It's as simple as that, self-initiation.

{% highlight javascript %}
$(function () {
  if($('[data-init]').data('init') === 'jbar') {
    $('[data-jbar]').jBar()
  }
})
{% endhighlight %}

### New technology advances, transitions and data URIs
The old jBar used JavaScript for animating the bar and toggle up and down, but this is looking like a dying art now. CSS3 transitions are a much better way of handling animation events. Even Google are beginning to phase out JavaScript animation, the new Google+ UI uses CSS3 properties for transitions, including _linear_ and _cubic-bezier_ easing. This stuff is all baked into the browser and is by far the better way to develop. Yes, it does mean IE7 and IE8 won't have a smooth animation and the jBar wil literally just move instantly - but that's what progressive enhancement is all about - they've got the functionality and that's all that matters.

The up and down jBar arrow images are also no longer images, they're data URIs, which means IE7 will not be able to render them. According to browser stats IE7 is pretty much non-existant. This also means that you don't need to upload any images to your directories to get it working, just the CSS which you're free to toy and destroy.

### Valid plugin configuration
As above, the JSON configuration is very simple, but the laws of JSON are a lot more strict. In HTML and JavaScript development, we can use single or double quotes, JSON required double quotes at all times, which is why the attribute _data-jbar_ has single, so they don't conflict and divide up the element in strange ways.

<div class="download-box">
  <a href="//toddmotto.com/labs/jbar" onclick="_gaq.push(['_trackEvent', 'Click', 'Demo jBar', 'jBar Demo']);">Demo</a>
  <a href="//github.com/toddmotto/jbar/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download jBar', 'Download jBar']);">Download</a>
  <a href="//github.com/toddmotto/jbar" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork jBar', 'jBar Fork']);">Fork</a>
</div>

### But I want to use Version 1.0.0
Suit yourself, grab it [here](http://toddmotto.com/labs/jbar/jbar_v1.zip).
