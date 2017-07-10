---
layout: post
permalink: /psswrd-the-show-hide-password-javascript-plugin/
title: Psswrd, the show/hide password JavaScript plugin
path: 2013-07-20-psswrd-the-show-hide-password-javascript-plugin.md
tags:
- Library
- JavaScript
---

Show/hide toggling for password inputs. Psswrd is a neat little script I've put together to aid in better user experience when users are completing your forms or actioning things inside web apps. For instance, instead of another irritating 'confirm password' field, we provide a 'show password' for them to double-check before signing up, logging in, filling out various 'secret questions', or whatever else you can think of.

Psswrd is a small script (1KB minified) that does exactly that. It's also really easy to integrate. It might not be an everyday usage script, but it certainly has its place. At the minute, there are no simple to integrate scripts that are written in raw JavaScript, most are jQuery dependent - so I wanted to mix it up and go all out on raw JS.

<div class="download-box">
  <a href="//toddmotto.com/labs/psswrd" onclick="_gaq.push(['_trackEvent', 'Click', 'Demo Psswrd', 'Psswrd Demo']);">Demo</a>
  <a href="//github.com/toddmotto/psswrd/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download Psswrd', 'Download Psswrd']);">Download</a>
  <a href="//github.com/toddmotto/psswrd" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork Psswrd', 'Psswrd Fork']);">Fork</a>
</div>

### Configuring Psswrd
Psswrd self-initiates when you tell it to, it doesn't need calling like a regular plugin, it just watches for the _data-init-psswrd_ attribute and will fire when ready. Configuring only takes a minute and you're good to go.

You'll probably have a form on your page to post the info back to the server, so you'll need to _init_ the plugin on it (though the init is limited to any element):

{% highlight html %}
<form action="/" method="post" data-init-psswrd>

</form>
{% endhighlight %}

Then inside that, you'll want to tell it what input fields to watch:

{% highlight html %}
<input type="password" data-psswrd-toggle>
{% endhighlight %}

That's it.

The structure of your form, however, is advised to be as follows to allow for optimal styling and structure:

{% highlight html %}
<form action="/" method="post" data-init-psswrd>
    <label>
        Username:
        <input type="text">
    </label>
    <label>
        Password:
        <input type="password" data-psswrd-toggle>
    </label>
    <button type="submit">Submit</button>
</form>
{% endhighlight %}

The above uses the _&lt;label&gt;_ element to provide a container for when the checkbox input and toggle text are appended inside. Psswrd takes care of the rest of things and your form will look like this:

{% highlight html %}
<form action="/" method="post" data-init-psswrd>
    <label>
        Username:
        <input type="text">
    </label>
    <label>
        Password:
        <input type="password" data-psswrd-toggle>
        <input id="data-psswrd-id-####" class="data-psswrd-checkbox" type="checkbox" data-psswrd-checkbox>
        <label for="data-psswrd-id-####" class="data-psswrd-text" data-psswrd-text>Show password</label>
    </label>
    <button type="submit">Submit</button>
</form>
{% endhighlight %}

### JavaScript
I'll talk through the main parts of what's happening in the script, let's start at the top:

{% highlight javascript %}
var Psswrd = function ( elem ) {
  this.elem = elem;
};
{% endhighlight %}

Here I'm creating a constructor function for the plugin, in which I'll tap into the prototypal inheritance features:

{% highlight javascript %}
Psswrd.prototype = {

  init : function () {

    var docFrag = document.createDocumentFragment();
    var random = 'data-psswrd-id-' + [ Math.floor( Math.random() * 9999 ) ];

    var dataCheckbox = document.createElement( 'input' );
    dataCheckbox.id = random;
    dataCheckbox.className = 'data-psswrd-checkbox';
    dataCheckbox.type = 'checkbox';
    dataCheckbox.setAttribute( 'data-psswrd-checkbox', '' );

    var dataText = document.createElement( 'label' );
    dataText.setAttribute( 'for', random );
    dataText.className = 'data-psswrd-text';
    dataText.setAttribute( 'data-psswrd-text', '' );
    dataText.innerHTML = 'Show password';

    docFrag.appendChild( dataCheckbox );
    docFrag.appendChild( dataText );
    this.elem.parentNode.appendChild( docFrag );

  }

};
{% endhighlight %}

The above creates all the necessary elements and appends them to the _this_ object, which as we loop through any configured elements, it creates a new instance of the function on each object.

Then it'd be wise to loop through the elements needed and create the _new_ instance of the function on each element:

{% highlight javascript %}
var dataInit = document.querySelectorAll( '[data-init-psswrd]' );
var psswrdNodes = document.querySelectorAll( '[data-psswrd-toggle]' );
if ( dataInit.length > 0 ) {
  for ( var i = 0; i < psswrdNodes.length; i++ ) {
    new Psswrd( psswrdNodes[i] ).init();
  }
}
{% endhighlight %}

The _if_ statement above checks if a _NodeList_ is returned for the _'data-init-psswrd'_ selector, and if so, boots the plugin for us. The rest of this section merely loops through the _'data-psswrd-toggle'_ selector and applies our function to it, which does all the heavy lifting for us.

Next the onchange event needs to be listened to:

{% highlight javascript %}
var changeFunction = function () {

  var labelChildNodes = this.parentNode.childNodes;

  for ( var z = 0; z < labelChildNodes.length; z++ ) {
    var self = labelChildNodes[z];
    if ( ( self.nodeName.toLowerCase() === 'input' ) && ( self.hasAttribute( 'data-psswrd-toggle' ) ) ) {
      self.type = this.checked ? 'text' : 'password';
    } else if ( ( self.nodeName.toLowerCase() === 'label' ) && ( self.hasAttribute( 'data-psswrd-text' ) )) {
      self.innerHTML = this.checked ? 'Hide password' : 'Show password';
    }
  }

};

var dataCheckbox = document.querySelectorAll( '[data-psswrd-checkbox]' );
for ( var j = 0; j < dataCheckbox.length; j++ ) {
  dataCheckbox[j].onchange = changeFunction;
}
{% endhighlight %}

This onchange event does all the clever toggling using the ternary operator. After a clever selector which grabs the siblings (parentNode.childNodes) and returns them as a NodeList, I can then loop through them and track down the correct elements. Two elements in the NodeList will be the ones I want, I make this manual safety check as whitespace actually returns as a Node inside the NodeList, a potential snag if you never knew it was coming.

We make the necessary checks and then assign the onchange handler onto the checkbox.

Putting it all together in a logical order, we can then see the bigger picture:

{% highlight javascript %}
window.psswrd = ( function ( window, document, undefined ) {

  'use strict';

  /*
   * Constructor function
   */
  var Psswrd = function ( elem ) {
    this.elem = elem;
  };

  /*
   * Fetch the data-psswrd-toggle inputs
   */
  var dataInit = document.querySelectorAll( '[data-init-psswrd]' );
  var psswrdNodes = document.querySelectorAll( '[data-psswrd-toggle]' );

  /*
   * Prototypal setup
   */
  Psswrd.prototype = {

    init : function () {

      var docFrag = document.createDocumentFragment();
      var random = 'data-psswrd-id-' + [ Math.floor( Math.random() * 9999 ) ];

      var dataCheckbox = document.createElement( 'input' );
      dataCheckbox.id = random;
      dataCheckbox.className = 'data-psswrd-checkbox';
      dataCheckbox.type = 'checkbox';
      dataCheckbox.setAttribute( 'data-psswrd-checkbox', '' );

      var dataText = document.createElement( 'label' );
      dataText.setAttribute( 'for', random );
      dataText.className = 'data-psswrd-text';
      dataText.setAttribute( 'data-psswrd-text', '' );
      dataText.innerHTML = 'Show password';

      docFrag.appendChild( dataCheckbox );
      docFrag.appendChild( dataText );
      this.elem.parentNode.appendChild( docFrag );

    }

  };

  /*
   * Change event that fires
   * when an input is checked
   */
  var changeFunction = function () {

    var labelChildNodes = this.parentNode.childNodes;

    for ( var z = 0; z < labelChildNodes.length; z++ ) {
      var self = labelChildNodes[z];
      if ( ( self.nodeName.toLowerCase() === 'input' ) && ( self.hasAttribute( 'data-psswrd-toggle' ) ) ) {
        self.type = this.checked ? 'text' : 'password';
      } else if ( ( self.nodeName.toLowerCase() === 'label' ) && ( self.hasAttribute( 'data-psswrd-text' ) )) {
        self.innerHTML = this.checked ? 'Hide password' : 'Show password';
      }
    }

  };

  /*
   * Initiate the plugin
   */
  if ( dataInit.length > 0 ) {
    for ( var i = 0; i < psswrdNodes.length; i++ ) {
      new Psswrd( psswrdNodes[i] ).init();
    }
  }

  /*
   * Bind onchange events to the checkboxes
   */
  var dataCheckbox = document.querySelectorAll( '[data-psswrd-checkbox]' );
  for ( var j = 0; j < dataCheckbox.length; j++ ) {
    dataCheckbox[j].onchange = changeFunction;
  }

})( window, document );
{% endhighlight %}

I originally went for a _&lt;div&gt;_ for the text holder, but instead used the _&ltlabel&gt;_ element so I could assign a _for_ attribute and _id_ to the paired checkbox and text, so the user's experience is enhanced again as it allows them to select the text as well to toggle the field as sometimes checkboxes alone can be a challenge to click (and more time consuming).

### Multiple Psswrds
Psswrd was built so you can have as many of the fields or even forms on the page, which means the possibilities are endless for whatever you're setting out to achieve.

<div class="download-box">
  <a href="//toddmotto.com/labs/psswrd" onclick="_gaq.push(['_trackEvent', 'Click', 'Demo Psswrd', 'Psswrd Demo']);">Demo</a>
  <a href="//github.com/toddmotto/psswrd/archive/master.zip" onclick="_gaq.push(['_trackEvent', 'Click', 'Download Psswrd', 'Download Psswrd']);">Download</a>
  <a href="//github.com/toddmotto/psswrd" onclick="_gaq.push(['_trackEvent', 'Click', 'Fork Psswrd', 'Psswrd Fork']);">Fork</a>
</div>
