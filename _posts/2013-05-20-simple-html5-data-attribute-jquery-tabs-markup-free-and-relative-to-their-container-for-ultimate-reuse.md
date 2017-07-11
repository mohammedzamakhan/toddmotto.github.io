---
layout: post
permalink: /simple-html5-data-attribute-jquery-tabs-markup-free-and-relative-to-their-container-for-ultimate-reuse/
title: Simple HTML5 data-* jQuery tabs, markup free and relative to their container for ultimate re-use
path: 2013-05-20-simple-html5-data-attribute-jquery-tabs-markup-free-and-relative-to-their-container-for-ultimate-reuse.md
tags:
- HTML5
- JavaScript
---

One of my favourite additions to the HTML5 spec is data-&#42; attributes, they're useful for such an array of things. I love integrating them into jQuery/JavaScript and seeing what difference to HTML they make.

Two things have inspired this post to create some mega simple tabbed content UI components, the first - Twitter Bootstrap. Twitter Bootstrap is used by so many people, but I really have no use for it other than pinching one of two of the jQuery plugins whilst working in development. The second reason for creating this is [AngularJS](//angularjs.org) from Google - a super intelligent web application framework that I am learning more and more of.

The Bootstrap tabs are okay, they're a little heavy for what they really achieve and the code isn't very self explanatory for developers wanting to learn from.

And back to AngularJS, I just love the way it works. It focuses on the view (being the DOM), in which you bind a model to (JavaScript). This makes the framework so reusable and flexible, and the future of web development definitely lies within where AngularJS is going. So here we are - the next idea. AngularJS makes use of their own attributes, literally extending the DOM and HTML's capabilities, a simple Angular demo:

{% highlight html %}
<div ng-app>
  <input type=text ng-model="inputted">
</div>
{% endhighlight %}

The above may not look like much, but you can see I've binded 'ng-model' onto the input element, and can essentially mirror/call the model using double curly brackets _&#123;&#123;inputted&#125;&#125;_ - which means anything I type into the input will be reflected into the DOM too. Built into AngularJS are directives that get this to work obviously, but you can see the simplicity behind it, as well as the fact it's totally reusable on as many elements throughout the DOM as you need. So let's head that way. Enough with the UI components that need actual hard coding - let's create objects that are reusable.

<div class="download-box">
  <a href="//toddmotto.com/labs/data-tabs" onclick="_gaq.push(['_trackEvent', 'Click', 'Demo Data Tabs, 'Data Tabs Demo']);">Demo</a>
  <a href="//toddmotto.com/labs/data-tabs/data-tabs.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download Data Tabs, 'Data Tabs Download']);">Download</a>
  <a href="//github.com/toddmotto/data-tabs" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork Data Tabs', Data Tabs Fork']);">Fork</a>
</div>

### HTML5 data-&#42; attributes
AngularJS doesn't just use 'ng-&#42;' prefixes to their binding, for validation purposes you can use data-ng-&#42; to stay safe - and data attributes are the way to go here too. Let's create two types of data, a tab and the content:

{% highlight html %}
<a href="#" data-tab="">Tab</a>
<div data-content="">Content</div>
{% endhighlight %}

This sets up the DOM for us to build upon. What I want to do next is essentially _match_ the tab clicked with the content box, so we need to pair the data-&#42; attributes, however the developer decides to match them with naming conventions is up to them; this script should be really flexible as long as the data-&#42; values pair:

{% highlight html %}
<a href="#" data-tab="1">Tab</a>
<div data-content="1">Content</div>
{% endhighlight %}

This now pairs them! So what next? We need to get started with the jQuery. We need to grab the _data-tab_ value once clicked, and match it against an element that contains the match pair inside _data-content_. Let's setup the click handler to target our _data-tab_ elements first:

{% highlight javascript %}
$('[data-tab]').on('click', function (e) {
  
})
{% endhighlight %}

Then log a result using jQuery's built-in _.data()_ API:

{% highlight javascript %}
$('[data-tab]').on('click', function (e) {
  console.log($(this).data('tab'))
})
{% endhighlight %}

You'll then see this logs the value inside the _data-tab_ attribute inside the console, step one is complete. Now step two, making it dynamically match by looking for the element's matching data-&#42; pair:

{% highlight javascript %}
$(this).siblings('[data-content=' + $(this).data('tab') + ']')
{% endhighlight %}

The above scans the sibling elements from the _$(this)_ element (current element clicked on) - and then scans the sibling elements to find an element that contains a _data-content_ selector with the identical data value.

We need to create some fuller markup now to get a better picture of what's happening:

{% highlight html %}
<div class="tabs">
  <a href="#" data-tab="1" class="tab active">Tab 1</a>
  <a href="#" data-tab="2" class="tab">Tab 2</a>
  <a href="#" data-tab="3" class="tab">Tab 3</a>
  
  <div data-content="1" class="content active">Tab 1 Content</div>
  <div data-content="2" class="content">Tab 2 Content</div>
  <div data-content="3" class="content">Tab 3 Content</div>
</div>
{% endhighlight %}

An _active_ classes need pushing around now it's in the markup, let's put the above together and swap out some classes:

{% highlight javascript %}
$('[data-tab]').on('click', function () {
  $(this).addClass('active').siblings('[data-tab]').removeClass('active')
  $(this).siblings('[data-content=' + $(this).data('tab') + ']').addClass('active').siblings('[data-content]').removeClass('active')
})
{% endhighlight %}

The active tab is set to _display:block;_ and all _data-content_ blocks are set to _display:none;_ which means only the content with the 'active' class will be shown. After chaining some jQuery methods, in 4 lines of code the tabs are fully functional, and completely independent of any markup, no specifying parent selectors, giving it a class or ID - it just works. It's very similar to Angular in a few ways, but obviously Angular is a massive web framework that allows for an MVC approach.

The finishing touch on the script is to prevent the &lt;a href="#"&gt; links from bouncing when you click the anchors, we simply capture the click event passed through the function, and prevent default on it:

{% highlight javascript %}
$('[data-tab]').on('click', function (e) {
  $(this).addClass('active').siblings('[data-tab]').removeClass('active')
  $(this).siblings('[data-content=' + $(this).data('tab') + ']').addClass('active').siblings('[data-content]').removeClass('active')
  e.preventDefault()
})
{% endhighlight %}

### One line of code
What's interesting about jQuery is the fact you can chain functions/methods together as it keeps returning the jQuery object after each one. I've actually split this code onto two lines (you see both calls for _$(this)_ but actually, these tabs are totally functional with chaining everything on just _one_ line of code (ignoring the click handler):

{% highlight javascript %}
$(this).addClass('active').siblings('[data-tab]').removeClass('active').siblings('[data-content=' + $(this).data('tab') + ']').addClass('active').siblings('[data-content]').removeClass('active')
{% endhighlight %}

### Ultimate re-use
Because the script is setup to search for sibling selectors, feeding from a _$(this)_ element - it means we can have multiple tabs per page with the same data-&#42; values!

### Extending the tabs
The tabs are setup to be totally markup free, and in true AngularJS fashion, you literally can just add more data-&#42; attributes and let it do it all for you:

{% highlight html %}
<div class="tabs">
  <a href="#" data-tab="1" class="tab active">Tab 1</a>
  <a href="#" data-tab="2" class="tab">Tab 2</a>
  <a href="#" data-tab="3" class="tab">Tab 3</a>
  <a href="#" data-tab="4" class="tab">Tab 4</a>
  <a href="#" data-tab="5" class="tab">Tab 5</a>
  <a href="#" data-tab="6" class="tab">Tab 6</a>
  <a href="#" data-tab="7" class="tab">Tab 7</a>
  
  <div data-content="1" class="content active">Tab 1 Content</div>
  <div data-content="2" class="content">Tab 2 Content</div>
  <div data-content="3" class="content">Tab 3 Content</div>
  <div data-content="4" class="content">Tab 4 Content</div>
  <div data-content="5" class="content">Tab 5 Content</div>
  <div data-content="6" class="content">Tab 6 Content</div>
  <div data-content="7" class="content">Tab 7 Content</div>
</div>
{% endhighlight %}

7, 8, 9, 10... and so on!

Just add more data-&#42; attributes and you're golden! :)

<div class="download-box">
  <a href="//toddmotto.com/labs/data-tabs" onclick="_gaq.push(['_trackEvent', 'Click', 'Demo Data Tabs, 'Data Tabs Demo']);">Demo</a>
  <a href="//toddmotto.com/labs/data-tabs/data-tabs.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download Data Tabs, 'Data Tabs Download']);">Download</a>
  <a href="//github.com/toddmotto/data-tabs" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork Data Tabs', Data Tabs Fork']);">Fork</a>
</div>
