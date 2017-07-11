---
layout: post
permalink: /apollo-js-standalone-class-manipulation-api-for-html5-and-legacy-dom/
title: Apollo.js, standalone class manipulation API for HTML5 and legacy DOM
path: 2013-11-09-apollo-js-standalone-class-manipulation-api-for-html5-and-legacy-dom.md
tags:
- Classnames
- Library
- JavaScript
---

Apollo is a &lt;1KB standalone DOM class manipulation API for adding, removing, toggling and testing the existence of classes on an element. Apollo is the successor to an [original post](/creating-jquery-style-functions-in-javascript-hasclass-addclass-removeclass-toggleclass) I published on raw JavaScript class functions earlier this year, but is completely rewritten and enhanced for the next level, whilst integrating HTML5.

Under the hood, Apollo uses the HTML5 `classList` API (jQuery isn't even using this yet!) when available and fallbacks to manual class manipulation for legacy support, making it the most powerful class manipulation API on the web. HTML5 `classList` performance far outweighs the legacy method.

Support? IE6+ for legacy support and internal feature detection to switch to HTML5 when available. Cross-browser compatible.

<div class="download-box">
  <a href="//github.com/toddmotto/apollo/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download apollo', 'Download apollo']);">Download</a>
  <a href="//github.com/toddmotto/apollo" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork apollo', 'apollo Fork']);">Fork</a>
</div>

I'll talk you through the APIs for Apollo.

### addClass API
To add a class using Apollo, use the `addClass` API, which takes an element and a single class name.

{% highlight javascript %}
Apollo.addClass(element, className);
{% endhighlight %}

### removeClass API
To remove a class using Apollo, use the `removeClass` API, which takes an element and a single class name.

{% highlight javascript %}
Apollo.removeClass(element, className);
{% endhighlight %}

### toggleClass API
To toggle a class using Apollo, use the `toggleClass` API, which takes an element and a single class name.

{% highlight javascript %}
Apollo.toggleClass(element, className);
{% endhighlight %}

### hasClass API
To test the existence of a class using Apollo, use the `hasClass` API, which takes an element and a single class name. The `hasClass` API returns a boolean (true/false) with the result.

{% highlight javascript %}
Apollo.hasClass(element, className);
{% endhighlight %}

### Improvements from inception
When I first wrote the APIs to allow you to create your own class manipulation functions, I used some _while_ loops, and the implementation was good, not great. I'm going to look at the _removeClass_ function now, and show you the difference in the new API.

#### Old API:
The old API was complex, but worked fantastically. It's important to note than when using a library that handles classes, that it actually removes _all_ instances and doesn't assume the class exists just once.

{% highlight javascript %}
function hasClass(elem, className) {
  return new RegExp(' ' + className + ' ').test(' ' + elem.className + ' ');
}
function addClass(elem, className) {
    if (!hasClass(elem, className)) {
      elem.className += ' ' + className;
    }
}
function removeClass (elem, className) 
  var newClass = ' ' + elem.className.replace( /[\t\r\n]/g, ' ') + ' ';
  if (hasClass(elem, className)) {
    while (newClass.indexOf(' ' + className + ' ') >= 0 ) {
      newClass = newClass.replace(' ' + className + ' ', ' ');
    }
    elem.className = newClass.replace(/^\s+|\s+$/g, '');
  }
}
function toggleClass(elem, className) {
  var newClass = ' ' + elem.className.replace( /[\t\r\n]/g, " " ) + ' ';
    if (hasClass(elem, className)) {
        while (newClass.indexOf(" " + className + " ") >= 0 ) {
            newClass = newClass.replace( " " + className + " " , " " );
        }
        elem.className = newClass.replace(/^\s+|\s+$/g, '');
    } else {
        elem.className += ' ' + className;
    }
}
{% endhighlight %}

#### New API
The removeClass new API is part of an Object, so it's not declared as a function like the above. As you can see, this is much cleaner, and uses one line for each removal technique too. It detects whether _classList_ is available and rolls with that if so, or falls back to a RegExp replace on the string. The RegExp uses a _'g'_ declaration in the RegExp constructor, meaning global - and will do a global replace on the classname, removing it each time it's present. I don't know about you, but that's a big improvement over file size and performance than the previous _while_ looping.

{% highlight javascript %}
hasClass: function (elem, className) {
  if (classList) {
    return elem.classList.contains(className);
  } else {
    return new RegExp('(^|\\s)' + className + '(\\s|$)').test(elem.className);
  }
},
addClass: function (elem, className) {
  if (!this.hasClass(elem, className)) {
    if (classList) {
      elem.classList.add(className);
    } else {
      elem.className += (elem.className ? ' ' : '') + className;
    }
  }
},
removeClass: function (elem, className) {
  if (this.hasClass(elem, className)) {
    if (classList) {
      elem.classList.remove(className);
    } else {
      elem.className = elem.className.replace(new RegExp('(^|\\s)*' + className + '(\\s|$)*', 'g'), '');
    }
  }
},
toggleClass: function (elem, className) {
  if (classList) {
    elem.classList.toggle(className);
  } else {
    if (this.hasClass(elem, className)) {
      elem.removeClass(className);
    } else {
      elem.addClass(className);
    }
  }
}
{% endhighlight %}

It's also good to note that I've also _added_ the entire _classList_ Object and native manipulation checks, and it's still smaller than the original :)

### Why not Prototype?
I originally rewrote the API to fall into a Prototype pattern, which looked like this (and you can use instead if you _really_ want):

{% highlight javascript %}
Element.prototype.hasClass = function (className) {
    if (document.documentElement.classList) {
        return this.classList.contains(className);
    } else {
        return new RegExp('(^|\\s)' + className + '(\\s|$)').test(this.className);
    }
};
Element.prototype.addClass = function (className) {
    if (!this.hasClass(className)) {
        if (document.documentElement.classList) {
            this.classList.add(className);
        } else {
           this.className += (this.className ? ' ' : '') + className;
        }
    }
};
Element.prototype.removeClass = function (className) {
    if (this.hasClass(className)) {
        if (document.documentElement.classList) {
            this.classList.remove(className);
        } else {
            this.className = this.className.replace(new RegExp('(^|\\s)*' + className + '(\\s|$)*', 'g'), '');
        }
    }
};
Element.prototype.toggleClass = function (className) {
    if (document.documentElement.classList) {
        this.classList.toggle(className);
    } else {
        if (this.hasClass(className)) {
            this.removeClass(className);
        } else {
            this.addClass(className);
        }
    }
};
{% endhighlight %}

I'd advise against doing this though. If you're including other libraries you might run into a lot of conflict when extending native DOM methods. It's also considered bad practices by some to extend existing DOM by prototyping, which is exactly why I created the Apollo API.

The Apollo API is also part of a JavaScript Module and returned as an Object with several APIs. It gives you the benefit of proper abstraction, testing and speed - throwing a bunch of Prototype extensions into the DOM doesn't.

<div class="download-box">
  <a href="//github.com/toddmotto/apollo/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download apollo', 'Download apollo']);">Download</a>
  <a href="//github.com/toddmotto/apollo" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork apollo', 'apollo Fork']);">Fork</a>
</div>

