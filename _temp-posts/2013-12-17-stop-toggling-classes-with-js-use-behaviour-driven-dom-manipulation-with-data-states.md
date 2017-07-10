---
layout: post
permalink: /stop-toggling-classes-with-js-use-behaviour-driven-dom-manipulation-with-data-states/
title: Stop toggling classes with JS, use behaviour driven DOM manipulation with data-states
path: 2013-12-17-stop-toggling-classes-with-js-use-behaviour-driven-dom-manipulation-with-data-states.md
tags:
- Opinions
- Classnames
- JavaScript
---

Using a class to manipulate the DOM? What about this idea. Using classes have many issues, the main one for me is that adding classes to elements to change their state crosses the behaviour and styling paradigm. Behaviour is separate to style and as our web becomes richer with functionality, the line between styling and state is a challenge, and also at times messy.

Anything beyond using `:hover` pseudo to style your website components introduces JavaScript, you might add an _open_ class for your menu, a _toggled_ class for your tabs, and so on. This is a nice semantic name for our hover event, but the two don't really meet at a real solution for managing our code's behaviour.

You're probably doing this:

{% highlight javascript %}
elem1.onclick = toggleClass(elem2, 'open');
{% endhighlight %}

This is a simple example of what we regularly do to achieve DOM state differences. This _sucks_!

It's messy and hard to maintain, we have to keep writing scripts for each component and might end up repeating ourselves a lot. It also introduces styling issues if you're adding 'active' class as it might conflict with another element elsewhere. It also doesn't tell me anything about what that element's _behaviour_ is from looking at it in the stylesheet.

### Thinking in states

When I build web applications/sites, I think about the element states. It might be _open_, _visible_, _toggled_ or maybe _selected_ - it all depends on what your components are doing. There are many class naming conventions that represent state that people have tried to implement, for example:

{% highlight css %}
.myClass {}
.myClass.isSelected {
  // do something
}
{% endhighlight %}

I think this is better than using a random 'selected' class, it's tied in closer to the element.

### Introduce the boolean state

Boolean states in your development I highly recommend, true or false, on or off, or with our latest thinking, _open_ or _closed_.

Let's look at some selectors that I wish we could integrate and have control of...

### Pseudo events

Wouldn't it be nice to have things like this?

{% highlight css%}
elem:closed {
  /* some styles */
}
elem:visible {
  /* some styles */
}
elem:open {
  /* some styles */
}
elem:toggled {
  /* some styles */
}
elem:selected {
  /* some styles */
}
{% endhighlight %}

Descriptive, behaviour driven, semantical CSS?...

For our menu, wouldn't it be awesome to do this:

{% highlight css%}
.menu {
  /* generic styles */
}
.menu:closed {
  display: none;
  background: blue;
}
.menu:open {
  display: inherit;
  background: red;
}
{% endhighlight %}

This keeps so many semantic values, as well as being _so_ easy to read and maintain. There are many awesome pseudo events that we could semantically introduce to our code that would keep things maintainable and semantic.

Unfortunately _this isn't going to work_, as this CSS is invalid...

So here's my idea, _data-state_ attributes for managing this problem.

### data-state attributes

Using data-* attributes for managing behaviour is a really neat way of abstracting the interactive layer of our code. Reading the data-* value is supported in all browsers (IE7), but targeting HTML using attribute selectors is supported in IE8+, so bye bye IE7 on this one (it's dead anyway). Let's get clever!

If I told you I could replicate the above, now, wouldn't that be sweet? Well, I can:

{% highlight css %}
.menu {
  /* generic styles */
}
.menu[data-state=closed] {
  display: none;
  background: blue;
}
.menu[data-state=open] {
  display: inherit;
  background: red;
}
{% endhighlight %}

At first you might be thinking _"what on earth..."_

But, I would say that is pretty clean, and helps us a lot with our coding. I can easily tell what the code is doing, and there are no adding or removing of classes happening here. I am merely going to be toggling the value of the data-state attribute, and the CSS will do its work.

### Toggling the data-state

This is the easy part, and requires only a few lines of code to actually do it. As because we are using a _data-state_ namespace, I can create a reusable function, pass it some arguments and bind it to events:

{% highlight javascript %}
elem.setAttribute('data-state', elem.getAttribute('data-state') === A ? B : A);
{% endhighlight %}

This line of code sets a data-state attribute, checks the current value and then uses the alternate value - the world's most simplest toggle! _A_ and _B_ here are of course our two values (states) that we want to toggle, which might look like this:

{% highlight javascript %}
elem.setAttribute('data-state', elem.getAttribute('data-state') === 'open' ? 'closed' : 'open');
{% endhighlight %}

This method uses the _ternary_ operator, a shorthand _if_ statement.

Putting it altogether, we might do the following and create a function tied in with our menu:

{% highlight javascript %}
var nav = document.querySelector('.nav__toggle');
var toggleState = function (elem, one, two) {
  var elem = document.querySelector(elem);
  elem.setAttribute('data-state', elem.getAttribute('data-state') === one ? two : one);
};

nav.onclick = function (e) {
  toggleState('.nav ul', 'closed', 'open');
  e.preventDefault();
};

// ES5 using .bind() #ftw
// nav.addEventListener('click', toggleState.bind(null, '.nav ul', 'closed', 'open'), false);
{% endhighlight %}

I've created a real quick _toggleState_ function which passes in a selector, and the two values to toggle, you'll then need to declare the markup:

{% highlight html %}
<nav class="nav">
    <a href="#" class="nav__toggle">Menu</a>
    <ul data-state="closed">
        <li>Item 1</li>
        <li>Item 2</li>
        <li>Item 3</li>
        <li>Item 4</li>
    </ul>
</nav>
{% endhighlight %}

I've declared that the navigation shall be closed, which indicates I'll have an event that then opens it.

Some CSS to see how it integrates:

{% highlight css %}
.nav {
    background: #2284B5;
    color: #fff;
    border-radius: 3px;
}
.nav a {
    padding: 5px 10px;
    display: block;
    color: #fff;
    text-decoration: none;
}
.nav ul {
    list-style: none;
    margin: 0;
    padding: 0;
}
.nav ul li {
    padding: 5px 10px;
}
/* semantic data states! */
.nav ul[data-state=closed] {
    display: none;
}
.nav ul[data-state=open] {
    display: inherit;
}
{% endhighlight %}

Output below:

<iframe width="100%" height="300" src="http://jsfiddle.net/toddmotto/9v2zx/embedded/result,js,html,css" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

If you inspect element and then check the value of the data-state attribute being toggled, you'll see the simplicity of the boolean state.

Of course this is looking at the future of how we can structure our website and webapp components, but I've been using it for a long time and am really happy with how seamless it fits into a workflow - and how much code and time I save.

:)
