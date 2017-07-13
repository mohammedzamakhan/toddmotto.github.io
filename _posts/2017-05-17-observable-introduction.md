---
layout: post
permalink: /rxjs-observables-observers-operators
title: "RxJS: Observables, observers and operators introduction"
path: 2017-05-17-rxjs-observables-observers-operators.md
tag: rxjs
---

RxJS is an incredible tool for reactive programming, and today we're going to dive a little deeper into what Observables and observers are - as well as learn how to create our own operators.

If you've used RxJS before and want to understand some of the inner workings and internals to "how" Observables work, as well as the operators, then this post is for you.

<div class="toc" markdown="1">
<span class="gamma">Table of contents</span>
{:.no_toc}
* TOC
{:toc}
</div>

### What is an Observable?

An Observable is just a function, with a few special characteristics. These are that it takes in an "observer" (an object with "next", "error" and "complete" methods on it), and returns cancellation logic. In our examples we will use a simple "unsubscribe" function to handle this further on when we write our own. In RxJS it would be a Subscription object with an `unsubscribe` method on it.

An Observable sets up the observer (we'll learn more about this) and connects it to the "thing" we want to get values from. This "thing" is called a producer, and is a source of values, perhaps from a `click` or `input` event in the DOM. It could even be something more complex such as communication over HTTP.

To better understand Observables, we're going to write our own! But first, let's take a look at an example with a subscription:

```js
const node = document.querySelector('input[type=text]');

const input$ = Rx.Observable.fromEvent(node, 'input');

input$.subscribe({
  next: (event) => console.log(`You just typed ${event.target.value}!`),
  error: (err) => console.log(`Oops... ${err}`),
  complete: () => console.log(`Complete!`)
});
```

This example takes an `<input type="text">` element and passes it into `Rx.Observable.fromEvent()`, which returns us an Observable of our input's `Event` object when the event name we specified emits (which is why we're using `${event.target.value}` in the console).

When the input's event listener fires, the Observable passes the value to the _observer_.

### What is an observer?

An observer is quite simple, in the above example the observer is the object literal we pass into our `.subscribe()` (subscribe will invoke our Observable).

> `.subscribe(next, error, complete)` is also valid syntax, but we'll be exploring the object literal form in this post

When an Observable produces values, it then informs the _observer_, calling `.next()` when a new value was successfully captured and `.error()` when an error occurs.

When we subscribe to an Observable, it will keep passing any values to an observer until one of two things happens. Either the producer says there are no more values to be sent, in which case it will call `.complete()` on our observer, or we (as the "consumers") decide we are no longer interested in the values and we unsubscribe.

When we want to compose the values returned from an Observable, before they reach our final `.subscribe()` block, the value is passed (or can be passed) through a chain of Observables, which is typically done via "operators". This chain is what we call an Observable sequence. Each operator returns a new Observable to continue our sequence - also known as a "stream".

### What is an operator?

As we've mentioned, Observables can be chained, which means we can do something like this:

```js
const input$ = Rx.Observable.fromEvent(node, 'input')
  .map(event => event.target.value)
  .filter(value => value.length >= 2)
  .subscribe(value => {
    // use the `value`
  });
```

Here are the steps of this sequence:

* Let's assume the user types "a" into our input
* The Observable then reacts to this event, passing the value to the _next_ observer
* The value "a" is passed to `.map()`, which is subscribing to our _initial_ observable
* `.map()` returns a new Observable of `event.target.value` and calls `.next()` on it's observer
* The `.next()` call will invoke `.filter()`, which is subscribing to `.map()`, with the _resulting_ value of the `.map()` call
* `.filter()` will then return another Observable with the filtered results, calling `.next()` with the value if the `.length` is 2 or above
* We get the final value through our `.subscribe()` block

Quite a lot happening, and if you're a little unsure, remember:

Each time a new Observable is returned, a new _observer_ is hooked up to the previous Observable, thus allowing us to pass values along a "stream" of observers that simply do something you've asked and call `.next()` when it's done, passing it to the next observer.

In short, an operator typically returns a new Observable each time - allowing us to continue our stream. As users we don’t need to worry about all the Observables and observers which are created and used behind scenes, we only use one per chain - our subscription.

### Building our own Observable

So, let's get started and write our own Observable implementation. It won't be as advanced as Rx's implementation, but we'll hopefully build the picture enough.

#### Observable constructor

First, we'll create an Observable constructor function that takes a `subscribe` function as its only argument. We'll store the subscribe property on the instance of Observable, so that we can call it later with an observer:

```js
function Observable(subscribe) {
  this.subscribe = subscribe;
}
```

Each `subscribe` callback that we assign to `this.subscribe` will be invoked either by us or another Observable. This will make more sense as we continue.

#### Observer example

Before we dive into our real world example, let's give a basic one.

As we've setup our Observable function, we can now invoke our observer, passing in `1` as a value and subscribe to it:

```js
const one$ = new Observable((observer) => {
  observer.next(1);
  observer.complete();
});

one$.subscribe({
  next: (value) => console.log(value) // 1
});
```

We subscribe to the Observable instance, and pass our observer (object literal) into the constructor (which is then assigned to `this.subscribe`).

#### Observable.fromEvent

That's all we actually needed to create the basis of our Observable, the next piece we need is a `static` method on the Observable:

```js
Observable.fromEvent = (element, name) => {

};
```

We're going to use our Observable just like in RxJS:

```js
const node = document.querySelector('input');

const input$ = Observable.fromEvent(node, 'input');
```

Which means we need to return a new Observable and pass a function in as the argument:

```js
Observable.fromEvent = (element, name) => {
  return new Observable((observer) => {
    
  });
};
```

This then passes our function to our `this.subscribe` in the constructor. Next up, we need to hook our event in:

```js
Observable.fromEvent = (element, name) => {
  return new Observable((observer) => {
    element.addEventListener(name, (event) => {}, false);
  });
};
```

So, what's this `observer` argument, and where does it come from?

The `observer` is actually your object literal with `next`, `error` and `complete` on.

> Here is the interesting piece. The `observer` is never passed through _until_ `.subscribe()` is invoked. This means the `addEventListener` is never "setup" by our Observable until it's subscribed to.

Once subscribe is invoked, inside the Observable's constructor the `this.subscribe` is then called, which _invokes_ the callback we passed to `new Observable(callback)` and also passes through our observer literal. This then allows the Observable to do it's thing and once it's done, it'll `.next()` on our observer with the updated value.

Okay so what now? We've got an event listener setup, but nothing is calling `.next()`, let's fix that:

```js
Observable.fromEvent = (element, name) => {
  return new Observable((observer) => {
    element.addEventListener(name, (event) => {
      observer.next(event);
    }, false);
  });
};
```

As we know, Observables need a "tear down" function which is called when the Observable is destroyed, in our case we'll remove the event:

```js
Observable.fromEvent = (element, name) => {
  return new Observable((observer) => {
    const callback = (event) => observer.next(event);
    element.addEventListener(name, callback, false);
    return () => element.removeEventListener(name, callback, false);
  });
};
```

We've not called `.complete()` because this Observable is dealing with DOM APIs and events, so technically they're infinitely available.

Let's try it out! Here's the full code of what we've done:

```js
const node = document.querySelector('input');
const p = document.querySelector('p');

function Observable(subscribe) {
  this.subscribe = subscribe;
}

Observable.fromEvent = (element, name) => {
  return new Observable((observer) => {
    const callback = (event) => observer.next(event);
    element.addEventListener(name, callback, false);
    return () => element.removeEventListener(name, callback, false);
  });
};

const input$ = Observable.fromEvent(node, 'input');

const unsubscribe = input$.subscribe({
  next: (event) => {
    p.innerHTML = event.target.value;
  }
});

// automatically unsub after 5s
setTimeout(unsubscribe, 5000);
```

Live example (type, then watch): 

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/3zgv7q2g/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### Building our own operator

Building our own operator should be a little easier now we understand the concepts behind an Observable and observer. On our `Observable` object, we'll add a new prototype method:

```js
Observable.prototype.map = function (mapFn) {

};
```

This method will be used as such, pretty much like `Array.prototype.map` in JavaScript but for any value:

```js
const input$ = Observable.fromEvent(node, 'input')
	.map(event => event.target.value);
```

So we need to take the callback function and invoke it, which in turn will return our desired data. Before we can do this, we need the latest value in the _stream_.

Here comes the clever part, we need to gain access to the instance of the Observable that invoked our `.map()` operator. Because it's on the prototype we can do exactly that:

```js
Observable.prototype.map = function (mapFn) {
  const input = this;
};
```

Ready for more funk? Now we subscribe inside a returned Observable:

```js
Observable.prototype.map = function (mapFn) {
  const input = this;
  return new Observable((observer) => {
  	return input.subscribe();
  });
};
```

> We are _returning_ the `input.subscribe()` because when we unsubscribe, the unsubscriptions (is that a word?) will flow up the chain, unsubscribing from each Observable.

This subscription will allow us to be passed the previous value from our `Observable.fromEvent`, because it returns a new Observable with a `subscribe` property in the constructor, we can simply subscribe to any updates it makes! Let's finish this off by invoking our `mapFn()` passed through map:

```js
Observable.prototype.map = function (mapFn) {
  const input = this;
  return new Observable((observer) => {
    return input.subscribe({
      next: (value) => observer.next(mapFn(value)),
      error: (err) => observer.error(err),
      complete: () => observer.complete()
    });
  });
};
```

Now we can chain it!

```js
const input$ = Observable.fromEvent(node, 'input')
  .map(event => event.target.value);

input$.subscribe({
  next: (value) => {
    p.innerHTML = value;
  }
});
```

Notice how the final `.subscribe()` block is passed only the `value` and not the `Event` object like before? You've successfully created an Observable stream.

Try it again:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/0rpkchm8/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Hopefully this post was good fun for you :)