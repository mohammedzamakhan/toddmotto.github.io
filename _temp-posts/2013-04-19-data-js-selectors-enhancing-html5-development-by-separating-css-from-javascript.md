---
layout: post
permalink: /data-js-selectors-enhancing-html5-development-by-separating-css-from-javascript/
title: data-js selectors, enhancing HTML5 development by separating CSS from JavaScript
path: 2013-04-19-data-js-selectors-enhancing-html5-development-by-separating-css-from-javascript.md
tags:
- HTML5
- CSS3
- JavaScript
---

I've started introducing a new concept into my JavaScript workflow and I really think it's worth sharing:

_Change your HTML/CSS all you like without changing a single line of JavaScript._

It's based around how CSS documents came into play, the concept behind CSS, and how we separate style from HTML elements. CSS documents were created to give reference to HTML, to target them and apply styles to them. This is generally done with classes and/or IDs, which meant you could change your entire stylesheet without changing any HTML, giving the site an entirely new look at the switch of a CSS document.

Let's rewind for a second, before CSS documents we were seeing this (inline styles that were bound to the element):

{% highlight html %}
<table style="border-top:1px solid #000;">
  <tr>
    <td>Oh hey.</td>
  </tr>
</table>
{% endhighlight %}

Which then became this (a class defining and attributing a specific style):

{% highlight html %}
<div class="table-border">
  <p>Oh hey.</p>
</div>
<style>
.table-border {
  border-top:1px solid #000;
}
</style>
{% endhighlight %}

But I've started thinking about a cross-over from the above concept, and how scalable our websites _really_ are when it comes to JavaScript selectors.

### The problem now
When you really think about it, JavaScript is for DOM interactions, we grab an element and do something with it. But we haven't got an official method of actually doing this - at least not one that's been thought about properly.

My first question is, why are we getting elements by ID? Why are we using CSS selectors in our JavaScript. And this comes back to the CSS concept, why aren't we separating CSS from JavaScript? For purposes of this article I'm going to be using jQuery to explain.

For example, you're used to seeing this no doubt:

{% highlight javascript %}
$('.contact-button').click(function(){
  alert($(this).text());
});
{% endhighlight %}

This would alert the 'text' of the contact button. It's safe to assume the HTML would look like this:

{% highlight html %}
<a href="contact.php" class="contact-button">Contact us</a>
{% endhighlight %}

But this is where the problem is (IMO), we're targeting classes with JavaScript. Here's the W3C definition of an HTML class:

_The class attribute has several roles in HTML: 1) As a style sheet selector (when an author wishes to assign style information to a set of elements). 2) For general purpose processing by user agents._

The same is pretty similar for ID, but includes anchor targeting for hypertext links and some other stuff. Neither say 'for hooking into elements for JavaScript manipulation'.

This is where [data-js] selectors come into play.

### [data-js] selectors
I'm defining a new style/method of JavaScript development, and after coming up with a suitable name for it with [Pedro Duarte](http://twitter.com/peduarte), I'm ready to share the idea.

The main idea is; when writing JavaScript, imagine classes and IDs don't exist. HTML5 allows the use of custom data-* attributes, which allow you to store data or objects to then do something with in JavaScript. For instance, you could do this:

{% highlight html %}
<a href="#" class="user" data-user="34321" data-name="David">Welcome, David.</a>
{% endhighlight %}

...and literally create some attributes on the fly. There is an argument that you shouldn't abuse HTML5 data-* attributes, but I don't think that's what we're doing with [data-js].

So why do I call it [data-js]? That's how we target the new development concept, here's the new HTML for that contact button earlier:

{% highlight html %}
<a href="contact.php" class="contact-button" data-js="click-contact">Contact us</a>
{% endhighlight %}

I've defined a new attribute, 'data-js' which will be a consistent naming convention throughout the project, which allows me to specify JavaScript selector names. Let's run a function again on it:

{% highlight javascript %}
$('[data-js=click-contact]').click(function(){
  alert($(this).text());
});
{% endhighlight %}

Now we're using the _data-js_ attribute as a JavaScript selector. You might be thinking, what really is the point in that?...

### Why use [data-js] attributes?
You're probably ready to type a bizarre comment asking me if I've gone crazy, but there are so many valid uses for this - and yet the concept is such an obvious one.

When creating websites, we code HTML and CSS, we create objects and components that need to do something. An interesting method of creating reusable objects is Object-Orientated CSS (OOCSS), which is several components coming together to form an entity. For instance:

{% highlight html %}
<a href="#" class="btn btn-red btn-medium">Send</a>
{% endhighlight %}

This is a nice and clean OOCSS object now. How would you go about targeting that with JavaScript? Surely creating reusable components across the DOM will send multiple click functions running at the same time, but ah-ha - you might think we could do this:

{% highlight html %}
<a href="#" class="send-email btn btn-red btn-medium">Send</a>
{% endhighlight %}

I've added a _send-email_ class to the button now, I could target that class with JavaScript and run a function. But isn't that redundant? We've already established classes are for styling, and now we're adding 'blank' classes that don't do anything. This is where [data-js] comes in!

Let's revisit that:

{% highlight html %}
<a href="#" class="btn btn-red btn-medium" data-js="send-email">Send</a>
{% endhighlight %}

The JavaScript:
{% highlight javascript %}
$('[data-js=send-email]').click(function(){
  $.ajax({
    // Do some email sending
  });
});
{% endhighlight %}

Okay, so what have we achieved by doing this? We've separated CSS from JavaScript. I think that's pretty well done if you ask me. But why is this such a benefit?

### CSS isn't JavaScript
...and JavaScript isn't CSS. The two are not the same, we shouldn't mix them. Separating the JavaScript selectors from CSS helps us in many ways, here's some killer examples:

- Change your CSS without worrying. Restructuring and recoding HTML/CSS is part of a developer's life. Now you can change your HTML/CSS all you like without changing a single line of JavaScript.

- Code readability. We don't want to _add_ more classes to our HTML for the sake of 'doing JavaScript'. This adds to clutter and will be hard to maintain. Think twice before you add that pointless 'active' class. Set data and check data instead, then it will never have collision with your DOM events.

- "What does that class do again?" - how many classes have you removed before and it's broken some JavaScript. Removing classes because they might not be in use can be part of refactoring old code to create new. Do you really want to search through all your _.js_ files to check if the selector exists?

- Using [data-js] attributes tells you a JavaScript event exists, you can't seriously remember what each class does with your current naming conventions. This increases development speed, you know the [data-js] selectors, you can locate them very easily in _.js_ files.

- You're extending HTML. Just like we write CSS/JS patterns, we are writing a new HTML pattern, adding a dedicated JavaScript hook.

### Nested selectors
A common part of JavaScript is targeting nested elements, for instance an _&lt;a&gt;_ tag inside an _&lt;li&gt;_. Let's take the following HTML as a use case using [data-js]:

{% highlight html %}
<li class="list-item" data-js="click-list">
  <a href="#">Account</a>
</li>
{% endhighlight %}

{% highlight javascript %}
$('[data-js=click-list] a').click(function(){
  // Fire event for <a> tag
});
{% endhighlight %}

### Potential pitfalls
With every new solution, not everything is perfect. Admittedly using [data-js] as a selector is slower than using classes or IDs - but can you really see 15,000+ processes a second to notice a difference?

### Conclusion
It's not a perfect solution; there isn't one. We either use classes and IDs for targeting, which by now you hopefully think is a bit weird - or use the flexibility of HTML5 to create our own event hooks on elements.

We've separated CSS from JavaScript, which gives us a lot more flexibility and ease of development. I'd love to hear thoughts on naming conventions and patterns for attribute content. I've dabbled with using an event-indicator (kind of telling me what the [data-js] is doing), for example:

{% highlight html %}
<li class="list-item" data-js="click-list">
  <a href="#">Account</a>
</li>
{% endhighlight %}

This says there is JavaScript hooking into the data-*, it's a click event on the list element. So there is some correlation between it all, or some method in the madness.

### Demo
For those of you who like a nice little demo, [here](http://jsfiddle.net/toddmotto/bK6ur) it is. Change the styles all you want, swap classes, ID's and completely transform the button - but of course using [data-js] it will still work flawlessly. My blog is now updated to use the [data-js] attribute methods for creating the responsive menu, check it out.

### Shortcut usage
A great comment below from [Kasper Mikiewicz](http://twitter.com/Idered) on how to extend jQuery a little to create a nice [data-js] wrapper function!

{% highlight javascript %}
$.js = function(el){
  return $('[data-js=' + el + ']')
};
{% endhighlight %}

Simply include that in your scripts file, and use like this:

{% highlight javascript %}
$.js('click-list').on('click', function() {
  // Take it away!
});
{% endhighlight %}

### Getting [data-js] to work in XHTML documents
Thanks to [Gabriele Romanatio](http://twitter.com/gabromanato) for looking into a [solution for XHTML documents](http://gabrieleromanato.name/jquery-making-html5-custom-data-attributes-work-in-XML) (XML usage in HTML), as data-* attributes are obviously only valid in HTML5 documents! Eat your heart out XML.
