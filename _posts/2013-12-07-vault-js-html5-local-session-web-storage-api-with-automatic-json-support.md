---
layout: post
permalink: /vault-js-html5-local-session-web-storage-api-with-automatic-json-support/
title: Vault.js, HTML5 (local/session) Web Storage API with automatic JSON support
path: 2013-12-07-vault-js-html5-local-session-web-storage-api-with-automatic-json-support.md
tags:
- Library
- JavaScript
---

Vault is a 0.6KB standalone HTML5 (local/session) Web Storage API with automatic JSON support. Web Storage only accepts a String as value of an Object property, Vault makes it possible to store entire JavaScript Objects using JSON when setting/getting. It also abstracts the storage APIs for both localStorage and sessionStorage, making it easy to work with both simultaneously.

IE8 supports Web Storage, therefore Vault can be used with IE8+, but note browser storage limitations in IE8 compared to modern browsers.

Using native HTML5 APIs can be a huge pain, that's why we should aim to create _abstractions_. Let's build a script/module that bakes in the required functionality so that it makes all future development seamless.

Vault.js (name inspired by some kind of storage vault), is exactly that - an abstraction. JavaScript is a low level language, which means the code is very raw, and provides little or no abstraction from itself. jQuery, for example is a DOM abstraction library, lots of things tucked away to make things easier for developers.

So what is Vault? Vault is for localStorage and sessionStorage. These two HTML5 APIs allow for either storage to be kept persistently (localStorage) or just for a session (sessionStorage), a session is until a browser tab is closed killing the process/web document.

<div class="download-box">
  <a href="//github.com/toddmotto/vault/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download vault', 'Download vault']);">Download</a>
  <a href="//github.com/toddmotto/vault" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork vault', 'vault Fork']);">Fork on GitHub</a>
</div>

### Looking at native methods

HTML5 localStorage will be the main lead for these short examples as the syntax is identical to sessionStorage, just the word is different for each type. Here's how to quickly _set_ and _get_ a value:

{% highlight javascript %}
// setting a value
localStorage.setItem(key, value);

// getting a value
localStorage.getItem(key);
{% endhighlight %}

HTML5 localStorage accepts only key + value pairs, which means you cannot store Objects - which would be the only really useful way of using localStorage. Or can we?...

Using JSON we can convert an Object to a String. If I then wanted to store an Object in localStorage, I can simply push the String. To do this, you might do the following using the native _JSON.stringify()_ method inside the _setItem_ call:

{% highlight javascript %}
localStorage.setItem(key, JSON.stringify(value));
{% endhighlight %}

...But now our Object has been stored as a String and will still remain a String. We need to use JSON again to parse it back into an Object:

{% highlight javascript %}
var value = localStorage.getItem(key);
JSON.parse(value); // parses String back into an Object
{% endhighlight %}

What happens now when we want to use sessionStorage? We'll need to write all the same code and keep it repetitive - which I don't like as it keeps code messy and not that manageable. These are fairly nice APIs compared to some JavaScript, but we could definitely make a higher level API.

This whole going back and forth between JSON parsing/stringify-ing and writing the same code is what inspired Vault, it makes things easy, especially if you're interchanging between localStorage and sessionStorage in the same application - you've got the whole power under the hood.

I'll talk you through the APIs for Vault and you can get using it. Vault builds in these above methods, automatically parses and stringifies your code for you, letting you develop with minimal code. It also allows you to switch easily between localStorage and sessionStorage.

### set API
To set data into web storage, you must use the `set()` API. With this API, there are three ordered arguments, `type`, which denotes the type of Web Storage, `key` for the Object's key, and `value` for the key value:

{% highlight javascript %}
Vault.set(type, key, value);
{% endhighlight %}

Example:

{% highlight javascript %}
// localStorage, object key = name, value = 'Tom Delonge'
Vault.set('local', 'name', 'Tom Delonge');

// sessionStorage, object key = name, value = 'Mark Hoppus'
Vault.set('session', 'name', 'Mark Hoppus');
{% endhighlight %}

### get API
Obtaining set data is easy with the `get()` API, simply reference a previously set key with `type` and the `key`:

{% highlight javascript %}
Vault.get(type, key);
{% endhighlight %}

Example:

{% highlight javascript %}
// getting 'name' from localStorage
// returns 'Tom Delonge'
Vault.get('local', 'name');
{% endhighlight %}

### remove API
Removing set data is easy with the `remove()` API, again reference a previously set key with `type` and the `key`:

{% highlight javascript %}
Vault.remove(type, key);
{% endhighlight %}

Example:

{% highlight javascript %}
// removes 'name' from localStorage
Vault.remove('local', 'name');
{% endhighlight %}

### empty API
It's a good idea to empty the user's Web Storage when possible to avoid overloading it, there are limits which differ per browser. Specifically modern browsers allow around `5MB` but IE versions are limited. IE8 also supports Web Storage and Vault.

{% highlight javascript %}
Vault.empty(type);
{% endhighlight %}

Example to empty `localStorage`:

{% highlight javascript %}
Vault.empty('localStorage');
{% endhighlight %}

### Vault.js
Here's a glimpse at the full script, which you can see adds some additional checks in there to quicken up the APIs and return if no results are found, which will help against errors in your JavaScript applications.

{% highlight javascript %}
window.Vault = (function (window, document, undefined) {

  'use strict';

  var getStore = function (type) {
    return type === 'local' ? localStorage : sessionStorage;
  };

  return {
    set: function (type, key, value) {
      if (!key || !value) return;
      getStore(type).setItem(key, JSON.stringify(value));
    },
    get: function (type, key) {
      var value = getStore(type).getItem(key);
      if (!value) return;
      return JSON.parse(value);
    },
    remove: function (type, key) {
      if (!getStore(type).getItem(key)) return;
      getStore(type).removeItem(key);
    },
    empty: function (type) {
      getStore(type).clear();
    }
  };

})(window, document);
{% endhighlight %}

As always, grab the code on [GitHub](//github.com/toddmotto/vault) and help contribute to open source!

<div class="download-box">
  <a href="//github.com/toddmotto/vault/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download vault', 'Download vault']);">Download</a>
  <a href="//github.com/toddmotto/vault" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork vault', 'vault Fork']);">Fork on GitHub</a>
</div>
