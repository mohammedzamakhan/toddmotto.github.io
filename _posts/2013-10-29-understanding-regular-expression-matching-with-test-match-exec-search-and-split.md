---
layout: post
permalink: /understanding-regular-expression-matching-with-test-match-exec-search-and-split/
title: Understanding Regular Expression matching with .test(), .match(), .exec(), .search() and .split()
path: 2013-10-29-understanding-regular-expression-matching-with-test-match-exec-search-and-split.md
tags:
- RegExp
- Test
- Match
- JavaScript
---

RegularExpressions, often noted as RegEx or RegExp, are seen and used pretty much everywhere in JavaScript. I use them all the time and you've probably seen them out in the wild too. When you first stumble(d) across a RegExp, you probably thought the person had fallen asleep on their keyboard and pushed to production, they look crazy at first, but they are a must have in your developer toolkit.

Here's an example of a RegExp that's intended to blow your mind if you've never seen one, if you've seen one, you may proceed:

{% highlight javascript %}
/^[a-zA-Z0-9.!#$%&’*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$/
{% endhighlight %}

This is infact a RegExp for matching the HTML5 email format (what the `input[type=email]` looks for in its native validation). I'm going to cover the use cases for testing RegExps in a few ways, each with a specific use case. This will include the most popular JavaScript APIs; `.test()`, `.match()` and `.exec()`.

### .test()
Using `.test()` is probably my favourite method of testing RegExps, it's a beautiful API, the fastest and the simplest to use. The `.test()` API runs a search for a match between a RegExp and a String.

_Notable features/tips:_

1. The `.test()` API returns a boolean (true/false) - returns true if your test passes (if the pattern you're matching is present) and false if it doesn't
2. Using `.test()` returns _no data_, so don't expect any

{% highlight javascript %}
if (/^localhost/.test(window.location.host)) {
  // returns true if working locally
}
{% endhighlight %}

### .match()
Using `.match()` is best when you require or are expecting data back in a test result, `.match()` returns an array with the matches, or simply `null` if there are none. The matches are retrieved from a string again. With `.match()`, you won't just be testing for the _presence_ of data, you'll want to see if a data pattern exists, and return that data. An example might be matching a credit card's four-digit pattern and returning those digits.

_Notable features/tips:_

1. Returns an array of matches
2. Generally doesn't describe the _actual_ data you want, uses capture groups
3. Capture groups match your data, and return the data inside the group to you
4. Think of it as a kind of validator
5. Can look the same as `.test()` but reversed

{% highlight javascript %}
var creditCardNumber = document.querySelector('input').value;

// returns the values in the input
// inside an ARRAY *if* matched
creditCardNumber.match(/\b(4[0-9]{12}(?:[0-9]{3})?)\b/);
{% endhighlight %}

You can access items in the array instantly by specifying the index:

{% highlight javascript %}
// notice the array index [1]
creditCardNumber.match(/\b(4[0-9]{12}(?:[0-9]{3})?)\b/)[1];
{% endhighlight %}

### .exec()
Exec is similar to `.match()`, although it actually returns the part of the string you were looking to match. For instance, if I wanted to search a string for the word 'Todd', I could get it returned if it matches.

_Notable features/tips:_

1. Returns the matching pattern, almost 'removing it from a string'

{% highlight javascript %}
// returns 'Todd'
/todd/i.exec('Hello, my name is Todd Motto');
{% endhighlight %}

You'll notice I added in the `/i` at the end, this means it's not case sensitive.

### .search()
Very similar to the `.exec()` method, but using `.search()` will tell you the index value of where the match was found.

_Notable features/tips:_

1. Returns the matching pattern's index value, how far into the string it occurred

{% highlight javascript %}
// returns '18'
var str = 'Hello, my name is Todd Motto';
str.search(/todd/i);
{% endhighlight %}

### .split()
Split is absolutely perfect for neat little tricks when dealing with returned data, using `.split()` will cut your string into two (or more) pieces.

_Notable features/tips:_

1. Great for splitting chunks of data
2. Returns a new array

Here's an example of splitting a string by its RegExp equivalent of whitespace:

{% highlight javascript %}
// returns ["Hello,", "my", "name", "is", "Todd", "Motto"]
'Hello, my name is Todd Motto'.split(/\s/g);
{% endhighlight %}

### Quirks
It's also interesting to note, that doing the following still produce the same result and pass the _if_ statement check:

{% highlight javascript %}
// .test()
if (/^localhost/.test(window.location.host)) {
  // true, let's me through
}

// .match()
if (window.location.host.match(/^localhost/)) {
  // returns data (so, true in the if statement)
  // let's me through
}

// .search()
if (window.location.host.search(/^localhost/)) {
  // returns data (so, true in the if statement)
  // let's me through
}

// ... probs a few other variants too
{% endhighlight %}

You'll come across a few more tricks when using RegExps and JavaScript APIs, but the above should get you started and clarify the use cases for each of them.
