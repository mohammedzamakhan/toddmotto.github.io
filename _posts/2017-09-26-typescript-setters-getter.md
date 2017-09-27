---
layout: post
permalink: /typescript-setters-getter
title: "Exploring set(), get() and Object.defineProperty() in TypeScript"
path: 2017-09-26-typescript-setters-getter.md
tag: typescript
---

When working with Javascript or TypeScript, you may have seen the `set` and `get` keywords being thrown around in various blogs or codebases - and they're extremely useful for parsing or modifying data that's about to be set on a particular object.

In this post, we're going to dive through what `set` and `get` actually mean, do, and look like in both ES5 and TypeScript. Hold onto your hats.

<div class="toc" markdown="1">
<span class="gamma">Table of contents</span>
{:.no_toc}
* TOC
{:toc}
</div>

### Getting and setting properties

Even if you've never explicitly used `set` and `get`, you've definitely used them - and now let's understand them.

Here's a trivial vanilla example to demonstrate a `set` and `get` in action:

```js
const element = document.querySelector('.foo');

// set: here we write the value
element.className = 'setting-a-property';

// get: we read the value
console.log(element.className); // 'setting-a-property';
```

When we write a value, we set it. When we read a value, we get it. Get it?

With `set` and `get`, we can control what happens between the read/write operations of values.

Okay let's move on. Why don't we create our own `set` and `get` methods to demonstrate how we can use them.

### ES5 and Object.defineProperty

Taking the above example, why don't we write our own implementation?

First, let's define a quick module pattern using ES5 (wait for TypeScript later, okay) to encapsulate our small constructor example:

```js
var Element = (function() {
  function Element() {}
  return Element;
})();

// new instance of Element
const element = new Element();
```

Let's now define our `set` and `get`, using `Object.defineProperty`. To do so, we need to essentially define a property to the constructor's `prototype`:

```js
var Element = (function() {
  function Element() {}

  Object.defineProperty(Element.prototype, 'className', {
    enumerable: true,
    configurable: true
  });

  return Element;
})();
```

So at this point, you can see we've defined `'className'` in a string, to tell our program to attach a new property to the prototype called `className`. What's next? Set and get obviously.

Before we can set anything, we need to define a further, internal (private) property on the constructor. We'll use `_class` and give it a default value of `null` for ease of demonstration:

```js
var Element = (function() {
  function Element() {
    this._class = null;
  }

  // ...
})();
```

> The underscore (`_class`) prefix is a naming convention typically used to denote a private field.

Why a private property? We don't want to access this directly, however we do want to potentially mutate it's value under-the-hood. Let's finish this off to get it fully working:

```js
var Element = (function() {
  function Element() {
    this._class = null;
  }
  Object.defineProperty(Element.prototype, 'className', {
    get: function() {
      return this._class;
    },
    set: function(name) {
      this._class = name;
    },
    enumerable: true,
    configurable: true,
  });
  return Element;
})();
```

Here, we're using `get` to _return_ the value (read it). Secondly, we're using `set` to directly _write_ a value to the internal property.

> You can think of `set` and `get` as hooks between your properties when they are defined, mutated or accessed. They allow you to apply changes, mutations or parsing to data before setting it elsewhere.

Let's see it in action, first we'll create an instance, set the property and then get it:

```js
const element = new Element();

element.className = 'foo';

console.log(element.className); // 'foo'
```

We did it! Nice and easy. Let's look at a better use case that actually changes the values before we set or get them.

For this use case, let's just assume we're creating some form of library and want to add a prefix to the `className`, for example `'todd-foo'` instead of just `'foo'`:

```js
var Element = (function() {
  function Element() {
    this._class = null;
  }
  Object.defineProperty(Element.prototype, 'className', {
    get: function() {
      return this._class;
    },
    set: function(name) {
      this._class = 'todd-' + name;
    },
    enumerable: true,
    configurable: true,
  });
  return Element;
})();
```

We could add this to `get` or `set`, depending what you want to do. Above, I'm using `set` so that each time our property is accessed, we don't have to keep adding the prefix each time `get` is called (which is every time the property is accessed).

There are lots of possibilities for use cases, parsing date objects, checking lengths of arrays, if properties exist on an object you passed through, and so forth.

<iframe width="100%" height="300" src="https://jsfiddle.net/toddmotto/vw8gn76m/embedded/result,js,html,css" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### With TypeScript

Let's move away from our ES5 example and convert this over to a TypeScript `class`.

```js
export class Element {
  private _class: string = null;
  
  get className() {
    return this._class;
  }
  
  set className(name) {
    this._class = `todd-${name}`;
  }
}
```

Well, this looks a _lot_ cleaner! There are a few interesting observations here. We can set the initial `_class` property on the class, and using `set className`, we not only define the _name_ of the property we want to communicate with publicly, but we have a much cleaner syntax (in my opinion, anyway). Readability is also improved, we know `_class` is `private`, and if we try accessing it anywhere else, the program will throw an error for us before we even compile it.

#### TypeScript's ES5 output

It's well worth a look to see what code is generated from the TypeScript compiler, targetting ES5 here:

```js
var Element = (function() {
  function Element() {}
  Object.defineProperty(Element.prototype, 'className', {
    get: function() {
      return this._class;
    },
    set: function(name) {
      this._class = 'todd-' + name;
    },
    enumerable: true,
    configurable: true,
  });
  return Element;
})();
```

Hmmm, looks familiar. We've come full circle, but let's take a further look at a combination with Angular.

### Angular decorators and setters

Interestingly enough, with TypeScript and Angular's decorators we can combine these approaches. Let's assume we have a date _timestamp_ coming through an `@Input` (from a server response perhaps), and we want to parse it before it's set in a particular component:


```js
@Component({...})
export class DateComponent {
  private _dateObject: Date;

  @Input() set date(date: number) {
    // assuming `date` is something like 1506439684321
    this._dateObject = new Date(date);
  }

  get date() {
    return `The date is ${this._dateObject}`;
  }
}
```

At this point, we can rock out `{% raw %}{{ date }}{% endraw %}` in our templates. This hopefully gives you some insight as to mutating `@Input` value in Angular before they reach the property.
