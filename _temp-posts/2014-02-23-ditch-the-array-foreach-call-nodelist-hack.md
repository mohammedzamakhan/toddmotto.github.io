---
layout: post
permalink: /ditch-the-array-foreach-call-nodelist-hack/
title: Ditch the [].forEach.call(NodeList) hack
path: 2014-02-23-ditch-the-array-foreach-call-nodelist-hack.md
tags:
- NodeList
- Opinion
- JavaScript
---

I've got to admit before we go any further, I used to use this technique. It looked edgy and cool and I was doing ECMA5 hacks, but after a while and writing better JavaScript, it turned out this technique causes nothing but headaches and needless hipster-like code, of which I'm going to pour my thoughts out on why I now hate seeing this technique (hack) in people's code.

### Understanding [].forEach.call(NodeList)

Let's establish what the hack actually does before we can look into why I think it's a really bad technique. Let's take a normal Array, and loop through it using ECMA5's `.forEach` magical method:

{% highlight javascript %}
var myArray = [1,2,3,4];
myArray.forEach(function (item) {
  console.log(item); // prints each number
});
{% endhighlight %}

So this looks nice, but where does the "hack" fit into this? Enter the `NodeList`:

{% highlight javascript %}
var myNodeList = document.querySelectorAll('li'); // grabs some <li>

// Uncaught TypeError: Object #<NodeList> has no method 'forEach' 
myNodeList.forEach(function (item) {
  // :(
});
{% endhighlight %}

So, we've reached an error, because NodeLists don't share the Array's prototype, of which contains the `forEach` method. There are some, erm, "solutions" to this:

{% highlight javascript %}
NodeList.prototype.forEach = Array.prototype.forEach;
{% endhighlight %}

If you've ever done this, then it probably wasn't a good idea (and please do not use it). Extending existing DOM functionality through prototypes is often considered bad practice as this can lead to masses of issues.

The way to get around this is by doing the following:

{% highlight javascript %}
var myNodeList = document.querySelectorAll('li'); // grabs some <li>
[].forEach.call(myNodeList, function (item) {
  // :) hooray `item` can be used here
});
{% endhighlight %}

And everything works. This accesses the created (empty) array's prototype method and using call allows the NodeList to take advantage.

Now let's look at some of the issues surrounding this technique.

### Problems

#### Problem #1: No Array methods
This is a big one. NodeLists have a length property, but what if you want to add a new element or remove one from that list? You are not keeping _any_ state by using the forEach hack, and have no access to the list itself, which means it's a one way street, you can manipulate once, but only to static elements, you can't go back and add/remove other elements.

Using methods such as `.splice()` will result in an error - as NodeLists do not contain this method in their Prototype. NodeLists cannot be changed too, which is often very impractical. This also means you can't do anything exciting with your NodeList, apart from maybe bind an event handler or call a method.

#### Problem #2: Limits reuse
We are caching the selector, but we're not caching the array or even what the loop is doing, which means we can't reuse the method as it's frequently seen. I see this as a huge issue for scalability as well as reusability. What if I want to call the method again? I'll have to write the same non-descriptive code out twice.

{% highlight javascript %}
// cached, we can access this again
var myNodeList = document.querySelectorAll('li');

// this will only get called once
// and cannot be called again
[].forEach.call(myNodeList, function (item) {
  // :(
});
{% endhighlight %}

#### Problem #3: Separation of concerns
NodeLists and Arrays are two different beasts, so why are we writing code where the overlap doesn't provide us any benefit? If you need an array _from_ a NodeList, then do exactly that. There are a few options for this, the non-cross-browser version:

{% highlight javascript %}
var myArrayFromNodeList = [].slice.call(document.querySelectorAll('li'));
{% endhighlight %}

But that's yet another `Array.prototype` hack, and I don't encourage it either. It isn't cross browser as IE will not allow NodeLists to form the `host object` of an `Array.prototype.slice` call.Instead, use a method to push all Nodes into a new array:

{% highlight javascript %}
var myNodeList = document.querySelectorAll('li');
var myArrayFromNodeList = []; // empty at first
for (var i = 0; i < myNodeList.length; i++) {
  myArrayFromNodeList.push(myNodeList[i]); // ahhh, push it
}
{% endhighlight %}

We'll then have a populated Array with our Nodes! B-e-a-uuutiful. What other benefit does this give us? We have complete separation of both object types and can reference each when we need:

{% highlight javascript %}
console.log(myNodeList); // NodeList
console.log(myArrayFromNodeList); // Array of Nodes
{% endhighlight %}

From here, we can then loop through our Array and make `splice` and `push` calls to actually do something valuable.

#### Problem #4: Creates a needless Array
Using `[].forEach.call` actually _creates_ a new Array, and it then dithers in memory, why would you even want to do that? There is a workaround for this however, using `Array.prototype.forEach.call`, which is in fact faster and more reliable (some libraries will conflict using `[]` syntax) and also simply accesses the `forEach` method, rather than creating a new array and then accessing it.

#### Problem #5: It's slower and works harder
I'm not going to get into a mass debate on shaving `0.00012230ms` from the method, but `[].forEach.call` is _very_ slow, especially as it's usually instantiating new objects against elements (or something like that). First, `[]` instantiates a new Array, and then for forEach method is then chained against `.call()` which then changes the execution context for each part of the loop. I don't know about you, but that's a lot of work for such a mild task.

#### Problem #6: Stupidity vulnerabilities
Based on the current examples we've seen, did you know that this example will still work:

{% highlight javascript %}
var myNodeList = document.querySelectorAll('li');
[1,2,3,4,5].forEach.call(myNodeList, function (item) {
  // Wah?...
  // Are we looping over the NodeList or Array?!
});
{% endhighlight %}

I don't want my code to be susceptible to things like that, it could happen and probably will/has.

#### Problem #7: Scalability
If I wanted to take the NodeList and ship it into another method, I'd have to completely rewrite the `forEach` hack and then ship it into a method, which then means more testing and opening up to more bugs. Write code properly the first time and you'll be able to extend your code excellently.

#### Problem #8: Readability
A random `forEach` (usually seen at the end of a script) is completely meaningless, what does it do? Loops are usually based around manipulating objects/elements of some kind, so wrapping it inside a method of your own would likely be better.

#### Problem #9: Confusing syntax
Are you manipulating a NodeList or an Array? Why make others work out what you're doing when you can easily write a method to take care of these things for you.

#### Problem #10: Not cross-browser
I don't usually use the ECMAScript 5 `forEach` method, usually a straightforward `for` loop is _way more_ than enough:

{% highlight javascript %}
var myNodeList = document.querySelectorAll('li');
for (var i = 0; i < myNodeList.length; i++) {
  // do something with myNodeList[i]
}
{% endhighlight %}

Not to mention much faster. I also have more control over my array elements, for example if I wanted to loop in reverse (generally is faster than forwards!):

{% highlight javascript %}
var myNodeList = document.querySelectorAll('li');
for (var i = myNodeList.length; i--;) { // reverse
  // do something with myNodeList[i]
}
{% endhighlight %}

You could even create your own wrapper `forEach` method, which will work in every browser and save you some typing.

#### Problem #11: Developer misunderstanding
I've seen developers use this method to loop over arrays, which as we've established would be rather silly because the hack is meant for NodeLists, not arrays.

There are likely more problems using the hack, but for now these will shed some light on main issue areas.

### Recommendations

Based on the above, I steer clear of it, it just makes for better code and to me is more sensible. It's easy to write your own `forEach` method, which keeps away from the need for the hacky methods as it'll take a NodeList _or_ Array:

{% highlight javascript %}
// forEach method, could be shipped as part of an Object Literal/Module
var forEach = function (array, callback, scope) {
  for (var i = 0; i < array.length; i++) {
    callback.call(scope, i, array[i]); // passes back stuff we need
  }
};

// Usage:
// optionally change the scope as final parameter too, like ECMA5
var myNodeList = document.querySelectorAll('li');
forEach(myNodeList, function (index, value) {
    console.log(index, value); // passes index + value back!
});
{% endhighlight %}

Keep track of your array and nodelist states via caching through variables. It doens't hurt to write a few extra lines to improve your code tenfold, especially when we gain so much more and futureproof our code.

Any thoughts appreciated! Happy coding!
