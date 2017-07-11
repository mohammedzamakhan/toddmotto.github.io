---
layout: post
permalink: /web-components-concepts-shadow-dom-imports-templates-custom-elements/
title: Web Components and concepts&#44; ShadowDOM&#44; imports&#44; templates&#44; custom elements
path: 2014-07-02-web-components-concepts-shadow-dom-imports-templates-custom-elements.md
tags:
- Web Components
- HTML5
- JavaScript
---

Web Components, the future of the web, inspired from attending [Google I/O](https://twitter.com/toddmotto/status/482624009553465344) I decided to pick up Web Components and actually build something. Since learning the basics around a year ago, it's changed and advanced a lot! Thought I'd write a post on it and share [my first web component](//github.com/toddmotto/fluidvids-polymer) yesterday (built with Polymer).

Before I get into Polymer, we'll look at Web Components in this post, what it means for the web and how it completely changes things and our outlook on building for the web platform from today.

Gone are the days of actually creating HTML structures and "pages" (what're those?). The web is becoming "all about components", and those components are completely up to us thanks to Web Components.

We aren't really at a stage where we can use Web Components to its fullest, browser support is still ongoing implementations and IE are [in consideration](http://status.modern.ie) of the entire spec (blows a single fanfare). But it's coming together, give it a few years and we'll get there. Or do we have to wait that long?...

Google are innovating in this area like no tomorrow with [Polymer.js](http://www.polymer-project.org), a polyfill and platform (that provides additional features such as data-binding, event callbacks and much more) for those missing pieces in modern browsers that don't fully support Web Components.

### Building blocks of Web Components

Before we get over excited about this stuff though, let's actually understand what the [Web Components spec](http://www.w3.org/TR/components-intro) really means. First thing's first, Web Components are a collection of building blocks, not a single thing. Let's look at each block to see what's up.

This will be a very high level view, otherwise this post could end up being three days worth of reading!

#### Templates
Templates are where we define reusable code, we even get an element for it with `<template>`. The first time you use it, don't panic - it's invisible in the visible interface output, until you view source you won't know anything is even there. It's merely a declarative element to create a new template for... anything you like.

An example of a `<template>` to populate a profile section for a user:

{% highlight html %}
<template id="profileTemplate">
  <div class="profile">
    <img src="" class="profile__img">
    <div class="profile__name"></div>
    <div class="profile__social"></div>
  </div>
</template>
{% endhighlight %}

Sprinkle some JavaScript to populate it, and append it to the `<body>`:

{% highlight javascript %}
var template = document.querySelector('#profileTemplate');
template.querySelector('.profile__img').src = 'toddmotto.jpg';
template.querySelector('.profile__name').textContent = 'Todd Motto';
template.querySelector('.profile__social').textContent = 'Follow me on Twitter';
document.body.appendChild(template);
{% endhighlight %}

You'll notice that this is just JavaScript, no new APIs or anything confusing. Nice! For me, a `<template>` is useless without its good buddy _Custom Elements_. We need this to do something useful with the tech, things are all global and disgusting as of now.

#### Custom Elements
Custom Elements allow us to define (you guessed it), our own element. This can be anything, but before you go crazy, your elements must have a dash, presumably to avoid any potential naming clashes with future HTML implementations - I think that's a good idea as well.

So, with our custom element, how do we do it? Simple really, we get the `<element>` element, so meta. Well, we _had_ the `<element>` element. Read on, as `<element>` was recently deprecated and thus needs a JavaScript implementation, but this is the older way:

{% highlight html %}
<element>
  <template id="profileTemplate">
    <div class="profile">
      <img src="" class="profile__img">
      <div class="profile__name"></div>
      <div class="profile__social"></div>
    </div>
  </template>
</element>
{% endhighlight %}

This example is still deprecated but worth showing. We would've given `<element>` a `name=""` attribute to define the custom element:

{% highlight html %}
<element name="user-profile">
  <template id="profileTemplate">
    <div class="profile">
      <img src="" class="profile__img">
      <div class="profile__name"></div>
      <div class="profile__social"></div>
    </div>
  </template>
</element>

// usage
<user-profile></user-profile>
{% endhighlight %}

##### So what's replacing `<element>`?

Use of `<element>` was [deprecated](http://lists.w3.org/Archives/Public/public-webapps/2013JulSep/0287.html) towards the end of 2013, which means we simply use the JavaScript API instead, which I think offers more flexibility and less bloat on the markup:

{% highlight html %}
<template id="profileTemplate">
  <div class="profile">
    <img src="" class="profile__img">
    <div class="profile__name"></div>
    <div class="profile__social"></div>
  </div>
</template>
<script>
var MyElementProto = Object.create(HTMLElement.prototype);
window.MyElement = document.registerElement('user-profile', {
  prototype: MyElementProto
  // other props
});
</script>
{% endhighlight %}

New elements must inherit from the `HTMLElement.prototype`. More on the above setup and callbacks etc [here](https://github.com/webcomponents/hello-world-element/blob/master/src/hello-world.html), cheers [Zeno](//twitter.com/zenorocha).

##### Extending and inheriting
What if we wanted to extend an existing element, such as an `<h1>` tag? There will be many cases of this, such as riding off an existing element and creating a "special" version of it, rather than a totally new element. We introduce the `{ extends: '' }` property to declare where what element we're extending. Using an extended element is simple, drop the `is=""` attribute on an existing element and it'll inherit its new extension. Pretty simple, I guess.

{% highlight html %}
<template>
  // include random, funky things
</template>
<script>
var MyElementProto = Object.create(HTMLElement.prototype);
window.MyElement = document.registerElement('funky-heading', {
  prototype: MyElementProto,
  extends: 'h1' // extends declared here
});
</script>

<h1 is="funky-heading">
  Page title
</h1>
{% endhighlight %}

Using `extends=""` as an attribute on `<element>` was the way to do it before it was deprecated.

So what next? Enter the shadows...

#### ShadowDOM
ShadowDOM _is_ as cool as it sounds, and provides a DOM encapsulation within DOM. Whaaat? Essentially, nested document fragments, that are shadow-y... In ShadowDOM, we're observing nested DOM trees/hierarchies. Typically in web documents, there is one DOM. Think about DOM hosting DOM, which hosts more DOM. You'll see something like this in Chrome inspector (note `#shadow-root`, which is completely encapsulated DOM):

{% highlight html %}
▾<user-profile>
  ▾#shadow-root (user-agent)
  <div class="profile">
    <img src="" class="profile__img">
    <div class="profile__name"></div>
    <div class="profile__social"></div>
  </div>
 </user-profile>
{% endhighlight %}

There are a few different concepts with Shadow DOM, for me, it's that there is no "global" Object, no `window`, I can create a new document root. The "host" of my this new document root is either referred to as the root or host. We can create new ShadowDOM by invoking `.createShadowRoot();` on an element.

ShadowDOM already exists in the wild today though, as soon as you use `<input type=range>` in the browser, we get a nice input with a slider, guess what - that's ShadowDOM! It's a nested structure that's hidden inside our DOM tree. Now we can create it ourselves, this opens up an entire plethora of opportunities.

##### Why is this _really_ cool?

ShadowDOM gives us _true_ encapsulation, with scoped components. CSS is _scoped_ (wow, although we tried this with `<style scoped>` but Blink have since removed it from the core to make way for Web Components). This means any CSS we write inside ShadowDOM only affects the DOM of that particular ShadowDOM!

{% highlight html %}
<template>
  <style>
  :host {
    border: 1px solid red;
  }
  </style>
  // stuff
</template>
<script>
var MyElementProto = Object.create(HTMLElement.prototype);
window.MyElement = document.registerElement('funky-heading', {
  prototype: MyElementProto,
  extends: 'h1'
});
</script>
{% endhighlight %}

This also means each document can also have a unique `id`, and we can avoid crazy naming conventions for scaling our apps/websites (a minor bonus).

We can also put scripts in there too and talk to the current element:

{% highlight html %}
<template>
  <style>
  :host {
    border: 1px solid red;
  }
  </style>
  // stuff
</template>
<script>
(function () {
  // stuff with JS...
})();

var MyElementProto = Object.create(HTMLElement.prototype);
window.MyElement = document.registerElement('funky-heading', {
  prototype: MyElementProto,
  extends: 'h1'
});
</script>
{% endhighlight %}

JavaScript events that are fired, also are encapsulated to the ShadowDOM tree.

##### How can I see this ShadowDOM?

In true shadow style, you need to enable it via the `Show user agent ShadowDOM` checkbox inside Chrome Dev Tools. Upon inspecting element, you can see the nested DOM trees. Chrome also allows you to edit the CSS, which is even more awesome.

#### HTML Imports
Importing dependencies into our language of choice comes in many shapes and sizes. For CSS, we have `@import`, for JavaScript in ES6 modules we have `import {Module} from './somewhere';`, and _finally_, HTML. We can import HTML components at the top of our document to define which ones we need to use in our app:

{% highlight html %}
<link rel="import" href="user-profile.html">

<!-- 
  <user-profile> now available, ooo yeah!
-->
{% endhighlight %}

This is massive! Encapsulated components all in one file. Out of the box and working. Let's take Google Maps API for example, we need to include the Maps API v3, import the 'Hello world' code and then style a basic map. Wouldn't it be great to just do this:

{% highlight html %}
<link rel="import" href="google-map.html">

<!-- boom! -->
<google-map></google-map>
{% endhighlight %}

All encapsulated, tested, I could just pass in values via attributes and job done:

{% highlight html %}
<google-map coords="37.2350, 115.8111"></google-map>
{% endhighlight %}

#### Decorators

Decorators are part of Web Components, but actually have _no spec_ (according to the [spec](http://www.w3.org/TR/components-intro/#decorator-section)). Apparently they might look something like this, with their intention to enhance or override the presentation of an existing element. So ignore them for now, I guess _(see Addy's comment on Decorators, they might even disappear from Web Components entirely)_.

{% highlight html %}
<decorator id="details-open">
  <template>
    <a id="summary">
      &blacktriangledown;
      <content select="summary"></content>
    </a>
    <content></content>
  </template>
</decorator>
{% endhighlight %}

### Can I get started now? Enter Polymer.js

Yes. Web Components are going to be a little while before fully landing and being the next generation of the web, but they're certainly making fast traction. We can get to grips with the technology and concepts now and start building using a framework such as Polymer - which polyfills things for modern browsers to let us use Web Components now.

An example of using Polymer to define an element. Here, we simply swap out (_was_) `<element>` for `<polymer-elememt>` and that's it.
{% highlight html %}
<polymer-element name="my-element">
  <template>
    // take it away!
  </template>
  <script>
    Polymer('my-element', {});
  </script>
</polymer-element>

<my-element></my-element>
{% endhighlight %}

Polymer has some really sweet features, such as data-binding (the Angular dev inside me loves this) and a tonne of simple events built in, from new instances of the element, to creation and injection callbacks that make it really simple to creating new elements.

### Takeaways
This post isn't meant to be a full tutorial - these components are vast and best explored individually, but I wanted to provide an eye opener on the rapidly approaching technology that is Web Components.

For me, one of the biggest selling points of Web Components is to prevent the inclusion of a huge JavaScript file, a huge CSS file and a tonne of HTML to make our website or app. In such cases, we no doubt come back to it a few months later and have forgotten what each thing does and it's painful to get back up to speed again. We don't forget what the `<google-map>` element does though, or the `<fluid-vids>` element, they're declarative and self-explanatory, we know exactly where their logic is, and where the styles are.

The biggest win? Logic is _contained_. We've all struggled managing logic, markup and styles, and now the web has listened. Encapsulated behaviour and scoping, but a very powerful engine for componentising the web, anything from a navigation to google maps to an image slider.

Benefits of Web Components are very clear, and I'm interested to see where it takes us in the next few years. This post is by far from exhaustive, but I feel we should all take a dive into what the future of the web will bring us, we'll be there sooner than you think!


Links to definitely keep an eye on (any others feel free to share below):

- [WebComponents.org](http://webcomponents.org)
- [Polymer](http://www.polymer-project.org)
- [customelements.io](http://customelements.io)
- [HTML5 Rocks](http://html5rocks.com)
- Eric Bidelman, Google I/O 2013 [Tectonic shift for the web](https://www.youtube.com/watch?v=fqULJBBEVQE)
- Eric Bidelman, Google I/O 2014 [Polymer and Web Components](https://www.youtube.com/watch?v=8OJ7ih8EE7s)
