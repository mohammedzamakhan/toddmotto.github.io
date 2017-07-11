---
layout: post
permalink: /the-data-js-api-for-behavioural-binding-stop-using-selectors-in-your-javascript/
title: The [data-js] API for behavioural-binding; stop using selectors in your JavaScript
path: 2013-09-21-the-data-js-api-for-behavioural-binding-stop-using-selectors-in-your-javascript.md
tags:
- Opinion
- JavaScript
---

Selectors in JavaScript are bad. The bridge between the DOM communicating with JavaScript is a tough one to cross and in no way modular, but there are ways around this. Today I want to introduce something I can only call _behavioural-binding_. JavaScript is a beautiful language, it's very dynamic, and that's exactly what selectors _aren't_. Selectors are a sin, and here's why behavioural-binding is key.

A while ago [I wrote about `data-js` selectors](//toddmotto.com/data-js-selectors-enhancing-html5-development-by-separating-css-from-javascript), a fleshed out idea that was very raw and an attempt to create JavaScript logic beyond DOM selectors by binding logic to a `data-*` selector and not targeting an element. I'd experimented with it a few times and it often got a bit sticky when it came to modularity - I was repeating a lot of code and the implementation was 90% there, until now. I've finally began to see the clearer picture on the behavioural-binding and feel like it's worth sharing. 

I'm going to show you how to:

1. Drop selectors for behavioural-binding in the DOM
2. Drop selector-reliant querying in JavaScript for modular logic
3. Use the JavaScript Module pattern to return public APIs
4. Rant a little about jQuery
5. Go away feeling modular

### Behavioural-binding
The behavioural-binding concept is binding repeating behaviours to DOM elements, instead of selecting elements and manipulating them. In a sense, it's what you're not used to doing. Probably:

_Previously_; you targeted an element, wrapped it in a function and did some DOM wizardry.
_Now_; you write your JavaScript logic independent of elements, and bind the behaviours with `data-*` attributes. The implementation is quite similar, but the thinking behind it is the separation key and how you'll need to think ahead for all future elements and not tie your JS so closely to your HTML. Behavioural-binding doesn't care what element it is, it'll just do its thing (if it's a valid method).

### Reuse and the problem scenario
The initial problem with DOM logic and JavaScript binding is simple, take three inputs for example with different classes:

{% highlight html %}
<input value="Select my contents" class="header-input">
<input value="Select my contents" class="footer-input">
<input value="Select my contents" class="sidebar-input">
{% endhighlight %}

I want to autoselect the text inside the input as soon as my cursor is focussed on the input, very simple.

But uh oh, I've got three different classes, _s**t_.

Now any (hypothetical) code is ruined because it only took into account `.header-input` - I need to account for the other two. So let's take a step back for a minute, thinking JavaScript first is often a really sexy way to code, let's think functionality. I bet you're starting to think 'Hmm, but why not just add an `autoselect` class to each of them?'. No. Just no. Classes are for styling, we've established this previously - I want to bind behaviour, not classes. So...

### Behavioural [data-js] binding
So how do we tackle the problem with applying the same logic to multiple elements without modifying our scripts everytime we extend them? I just want to write logic once and let it do the same work regardless of the element's identifier.

That's where `data-js` behavioural-binding comes in, seamless JavaScript logic. Wouldn't it be nice to do this:

{% highlight html %}
<input value="Select my contents" class="header-input" data-js="select">
<input value="Select my contents" class="footer-input" data-js="select">
<input value="Select my contents" class="sidebar-input" data-js="select">
{% endhighlight %}

I've binded my JavaScript logic independently, no conflicts with my class names or even inside the class attribute. It also means that when it comes to my next project, I can just lift the JavaScript logic out for reuse and not have to fuss about changing class names and refactoring things.

You can then use a selector like so to target these inputs:

{% highlight javascript %}
var selectInputs = document.querySelectorAll('[data-js=select]');
{% endhighlight %}

This returns a NodeList of the exact inputs I need. Now I can do the following to bind a click event:

{% highlight javascript %}
var selectInputs = document.querySelectorAll('[data-js=select]');
for (var i = 0; i < selectInputs.length; i++) {
  var self = selectInputs[i];
  self.onclick = function () {
    this.select();
  };
}
{% endhighlight %}

Perfect! And we're all done.

Or are we? Here's my next problem, well - maybe not a problem, I just like refining things and am a little bit OCD. My next problem is selector names _inside_ JavaScript - I think that sucks too!

So here's the next idea; include NO selectors in your JavaScript logic and expose a public API for you to pass selectors into, sounds good?

### Selector-less JavaScript functions
JavaScript functions are sweet, you can pass arguments into them, which means we can add some dynamic functionality. Instead of declaring this:

{% highlight javascript %}
var selectInputs = document.querySelectorAll('[data-js=select]');
{% endhighlight %}

Wouldn't it be better to make the `data-js` value dynamic for ultimate reuse? Yes! Here goes:

{% highlight javascript %}
// create a datajs selector wrapper
var datajs = function (selector) {
    return document.querySelectorAll('[data-js=' + selector + ']');
};
// get the returned nodelist
var selectInputs = datajs('select');
{% endhighlight %}

Now we are thinking dynamically, that's the first step. This means we can pass in more arguments to get other `data-js` attributes, for example:

{% highlight javascript %}
// create a datajs selector wrapper
var datajs = function (selector) {
    return document.querySelectorAll('[data-js=' + selector + ']');
};
// get the returned nodelists for 'select' and 'search'
var selectInputs = datajs('select');
var searchButtons = datajs('search');
{% endhighlight %}

You'll get a fresh NodeList returned each time with a dynamic parameter, nice. Now we're seeing the power of JavaScript start to come forward. But there's still more room for improvement in my eyes.

### Creating a Module and public API
Now is the time to create an API that's totally separate from any JavaScript we write! Ever create modules for your applications/websites? Organising your code takes a little longer and takes some discipline to stick at, but here's how we can take the `data-js` modularity even further.

I'd like to be able to write a bunch of code that is fully dynamic, that doesn't rely on selectors inside it, but gets the data from outside the scope. In true module fashion, we could do this:

{% highlight javascript %}
Module.dataSelect('select');
{% endhighlight %}

And that's it. This is the type of stuff that would be called on DOM Ready inside a pair of `<script>` tags or whatever your setup allows. I've created a Module that has a 'select' method, in which I pass in the 'select' attribute (ignoring the `data-js` part) of the selector as this is already setup.

Here's the example module setup I've created (notice there is nothing DOM related in here, awesomely agnostic!):

{% highlight javascript %}
var Module = (function () {

    var datajs = function (selector) {
        return document.querySelectorAll('[data-js=' + selector + ']');
    };

    var dataSelect = function (attr) {
        var elem = datajs(attr);
        var select = function () {
            this.select();
        };
        for (var i = 0; i < elem.length; i++) {
            var self = elem[i];
            self.onclick = select;
        }
    };

    return {
        dataSelect: dataSelect
    };

})();
{% endhighlight %}

You'll see I've got my `datajs` selector wrapper, a function called select (`var select`) which handles the NodeList looping and event binding. This is a Module Pattern with a revealed API, often named the Revealing Module Pattern. In short:

{% highlight javascript %}
// module declaration
var Module = (function () {

    var _privateFunction = function () {
      // private logic
    };

    var publicFunction = function () {
      // public logic
    };

    // publicly exposed API (revealed)
    return {
        public: publicFunction
    };

})();

// usage
Module.public();
{% endhighlight %}

Why do I like this? For starters it creates a consistent namespace for all `data-js` modules, of course I can rename my `Module` to whatever I like though. It also promotes exposing only functions you need which keep private functions safe, but most importantly, you don't have a single selector in the script at all - just the `data-js` prefix that we've established is well needed.

### jQuery and selector vomit
I'll call it selector vomit, because I've seen this so many times:

{% highlight javascript %}
$('div.className > ul:first-child li.className').on('click', function () {
  // WTF...
});
{% endhighlight %}

It happens all the time, and jQuery's amazing selector engine Sizzle promotes its power, which is vastly abused by so many developers. Of course when you're learning, you don't know any different. I mean, when I faced challenges in DOM selectors and JavaScript logic in early days I'd duplicate a script and just change a selector to get it to work twice - crazy looking back at it.

If you're writing JavaScript with selector vomit you probably shouldn't be writing it at all. JavaScript provides functionality, it shouldn't be dependent on a Node somewhere in the DOM tree.

Let's aim for a cleaner future.

### Data attributes and performance
'But getting an element by ID is faster'. Getting by ID is old and not modular. Some 1970s cars are faster than today's ones but I know which one I'd rather drive - faster isn't better. Data attributes were added to the HTML5 specification for a reason and they should be used powerfully - a.k.a by writing as less code as possible to do some awesome stuff.

Using `data-*` attributes for performance selectors are actually surprisingly fast, check out some neat work conducted from [Robert Bennet](//madebyhoundstooth.com/blog/javascript-selector-performance) from my [first article](//toddmotto.com/data-js-selectors-enhancing-html5-development-by-separating-css-from-javascript).

### Demo

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/LCAjY/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### [data-js] JSON/Obj literals
How about passing in object data into our module? Here's how we could fully extend the DOM with no selectors inside our core script:

{% highlight javascript %}
Module.myPlugin({
  search: {
    selector: 'search',
    target: 'select'
  }
});
{% endhighlight %}

### Data-binding JSON (highly experimental, for reading only!)
I also have experimented with thrashing a few ideas out with JSON inside `data-*` attributes to fully configure the DOM and do crazy stuff with, it can be classed as a little close for comfort with regards to the separation of concern - but I think it's got some possible use cases and potential grounding for the future and dynamically creating elements and setups, here's an example:

{% highlight javascript %}
<div class="myPlugin" data-js='{
  "someName": {
    "option": "value",
    "option": "value",
    "options": ["value", "value", "value"]
  }
}'></div>
{% endhighlight %}

You can then use JavaScript to read the properties whilst looping through the elements to dynamically generate a unique setup for each Node, I've seen this idea once or twice also on the web, it's obviously not too crazy. For the future, I'd definitely like to experiment more.

### Food for thought
I hope you've been a little intrigued at least from this article and what it presents, if so, here's some things to remember for future coding to aim for:

1. Use `data-js` attributes and appropriate values for DOM selectors
2. For repetitive JS, create a small Module and expose an API, pass in your selectors and keep your JavaScript free of the DOM
3. Start to structure functions a little better using the (Revealing) Module Pattern where necessary
4. Separate behaviour from style
