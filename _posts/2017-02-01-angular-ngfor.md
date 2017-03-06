---
layout: post
permalink: /angular-ngfor-template-element
title: "Angular ngFor, &lt;template&gt; and the compiler"
path: 2017-02-01-angular-ngfor.md
---

Angular `ngFor` is a built-in Directive that allows us to iterate over a collection. This collection is typically an array, however can be "array-like". To demonstrate this, we'll be using `Rx.Observable.of()` to initialise our collection with an Observable instead of a static array.

We'll also be exploring some other under-the-hood properties of `ngFor`, as well as looking at how Angular expands our `ngFor` to a `<template>` element and composes our view.

### Table of contents

- <a href="#using-the-ngfor-directive">Using the ngFor directive</a>
  - <a href="#ngmodule-import">NgModule import</a>
  - <a href="#iterating-collections">Iterating collections</a>
  - <a href="#using-ngfor">Using ngFor</a>
  - <a href="#using-trackby-for-keys">Using trackBy for keys</a>
  - <a href="#capturing-index-and-count">Capturing “index” and “count”</a>
  - <a href="#accessing-first-last-odd-even">Accessing first, last, odd, even</a>
- <a href="#lttemplategt-element">&lt;template&gt; element</a>
  - <a href="#lttemplategt-and-web-components">&lt;template&gt; and Web Components</a>
  - <a href="#ngfor-and-lttemplategt">ngFor and &lt;template&gt;</a>
  - <a href="#ngfor-and-embedded-view-templates">ngFor and embedded view templates</a>
  - <a href="#context-and-passing-variables">Context and passing variables</a>

### Using the ngFor directive

In this section, we'll be taking a look at the featureset `ngFor` provides, as well as some use case examples.

#### NgModule import

First of all, to use `ngFor`, you need to import the `CommonModule`. However, if you're using the `BrowserModule` in the root module, you won't need to import it, as the `BrowserModule` exposes the `CommonModule` for us - so you only need to import the `CommonModule` when creating further modules with `@NgModule`.

Our `@NgModule` will look like this:

```js
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

For this article, we'll be including a further `ContactCardComponent` component in our `@NgModule`:

```js
// ...
import { ContactCardComponent } from './contact-card.component';

@NgModule({
  declarations: [
    AppComponent,
    ContactCardComponent
  ],
  // ...
})
export class AppModule {}
```

Our `ContactCardComponent` simply looks like this, taking a single `@Input` of `contact`:

```js
import { Component, Input } from '@angular/core';

import { Contact } from './models/contact.interface';

@Component({
  selector: 'contact-card',
  template: `
    <div class="contact-card">
      <p>{% raw %}{{ contact.name }}{% endraw %} ( {% raw %}{{ contact.age }}{% endraw %} )</p>
      <p>{% raw %}{{ contact.email }}</{% endraw %}p>
    </div>
  `
})
export class ContactCardComponent {
  
  @Input()
  contact: Contact;
  
}
```

So now we're all setup, what's next?

#### Iterating collections

Now that our `ContactCardComponent` is included in our module, we can setup our `AppComponent` to use this dataset:

```js
@Component({...})
export class AppComponent implements OnInit {
  contacts: Observable<Contact[]>;
  ngOnInit() {
    this.contacts = Observable.of([
      {
        "id": 1,
        "name": "Laura",
        "email": "lbutler0@latimes.com",
        "age": 47
      },
      {
        "id": 2,
        "name": "Walter",
        "email": "wkelley1@goodreads.com",
        "age": 37
      },
      {
        "id": 3,
        "name": "Walter",
        "email": "wgutierrez2@smugmug.com",
        "age": 49
      },
      {
        "id": 4,
        "name": "Jesse",
        "email": "jarnold3@com.com",
        "age": 47
      },
      {
        "id": 5,
        "name": "Irene",
        "email": "iduncan4@oakley.com",
        "age": 33
      }
    ]);
  }
}
```

As mentioned in the introduction, I'm using `Observable.of` here from RxJS to give me an Observable stream from the results, this is a nice way to mimic an Observable response, such as when using the Angular `Http` module to return data from your API.

#### Using ngFor

Now we're setup, we can look into our `AppComponent` template:

```js
@Component({
  selector: 'app-root',
  template: `
    <div class="app">
      <ul>
        <li>
          <contact-card></contact-card>
        </li>
      </ul>
    </div>
  `
})
```

You can see I'm declaring `<contact-card>` inside of here, as we want to iterate our dataset and populate each contact via the `@Input` setup inside our `ContactCardComponent`.

One way we could do this is using `ngFor` on the component itself, however for simplicity we'll use the unordered list. Let's add `ngFor`:

```html
<ul>
  <li *ngFor="let contact of contacts">
    <contact-card></contact-card>
  </li>
</ul>
```

There are a few things happening here, the first you'll notice a `*` character at the beginning of the `ngFor`, we'll come onto what this means in the next section when we look at the `<template>` element. Secondly, we're creating a context called `contact`, using a "for of" loop - just [like in ES6](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Statements/for...of).

The `ngFor` Directive will clone the `<li>` and the _child nodes_. In this case, the `<contact-card>` is a child node, and a card will be "stamped out" in the DOM for each particular item inside our `contacts` collection.

So, now we have `contact` available as an individual Object, we can pass the individual `contact` into the `<contact-card>`:

```html
<ul>
  <li *ngFor="let contact of contacts">
    <contact-card [contact]="contact"></contact-card>
  </li>
</ul>
```

If you're using a static array, or binding the _result_ of an Observable to the template, you can leave the template as it currently is. However, we can optionally bind the Observable directly to the template, which means we'll need the `async` pipe here to finish things off:

```html
<ul>
  <li *ngFor="let contact of contacts | async">
    <contact-card [contact]="contact"></contact-card>
  </li>
</ul>
```

You can check a live output of what we've covered so far here:

<iframe src="//embed.plnkr.co/SzY3lZ2oLhFNAQYDG1uT?deferRun" frameborder="0" border="0" cellspacing="0" cellpadding="0" width="100%" height="400"></iframe>

#### Using trackBy for keys

If you're coming from an Angular 1.x background, you'll have likely seen "track by" when using an `ng-repeat`, and similarly in React land, using `key` on a collection item.

So what do these do? They associate the objects, or keys, with the particular DOM nodes, so should anything change or need to be re-rendered, the framework can do this much more efficiently. Angular's `ngFor` defaults to using _object identity_ checking for you, which is fast, but can be _faster_!

This is where `trackBy` comes into play, let's add some more code then explain:

```html
<ul>
  <li *ngFor="let contact of contacts | async; trackBy: trackById;">
    <contact-card [contact]="contact"></contact-card>
  </li>
</ul>
```

Here we've added `trackBy`, then given it a value of `trackById`. This is a _function_ that we'll add in the component class:

```js
trackById(index, contact) {
  return contact.id;
}
```

All this function does is use a custom tracking solution for our collection. Instead of using object identity, we're telling Angular here to use the unique `id` property that each `contact` object contains. Optionally, we can use the `index` (which is the index in the collection of each item, i.e. 0, 1, 2, 3, 4).

If your API returns unique data, then using that would be a preferable solution over `index` - as the `index` may be subject to change if you reorder your collection. Using a unique identifier allows Angular to locate that DOM node associated with the object much faster, and it will reuse the component in the DOM should it need to be updated - instead of destroying it and rebuilding it.

You can check a live output of what we've covered here:

<iframe src="//embed.plnkr.co/OAgfU6vKxd1bevusnxDQ?deferRun" frameborder="0" border="0" cellspacing="0" cellpadding="0" width="100%" height="400"></iframe>

#### Capturing "index" and "count"

The `ngFor` directive doesn't just stop at iteration, it also provides us a few other niceties. Let's explore `index` and `count`, two public properties exposed to us on each `ngFor` iteration.

Let's create another variable called `i`, which we'll assign the value of `index` to. Angular exposes these values under-the-hood for us, and when we look at the next section with the `<template>` element, we can see how they are composed.

To log out the index, we can simply interpolate `i`:

```html
<ul>
  <li *ngFor="let contact of contacts | async; let i = index;">
    Index: {% raw %}{{ i }}{% endraw %}
    <contact-card [contact]="contact"></contact-card>
  </li>
</ul>
```

This will give us every index, starting from `0`, for each item in our collection. Let's also expose `count`:

```html
<ul>
  <li *ngFor="let contact of contacts | async; let i = index; let c = count;">
    Index: {% raw %}{{ i }}{% endraw %}
    Count: {% raw %}{{ c }}{% endraw %}
    <contact-card [contact]="contact"></contact-card>
  </li>
</ul>
```

The `count` will return a live collection length, equivalent of `contacts.length`. These can optionally be bound and passed into each `<contact-card>`, for example you may wish to log out the total length of your collection somewhere, and also pass the `index` of the particular contact into a function `@Output`:

```html
<ul>
  <li *ngFor="let contact of contacts | async; let i = index; let c = count;">
    <contact-card 
      [contact]="contact"
      [collectionLength]="c"
      (update)="onUpdate($event, i)">
    </contact-card>
  </li>
</ul>
```

You can check a live output of what we've covered here:

<iframe src="//embed.plnkr.co/ZZqNBKnsAgWjrpux5eLn?deferRun" frameborder="0" border="0" cellspacing="0" cellpadding="0" width="100%" height="400"></iframe>

#### Accessing first, last, odd, even

Four more properties exposed by `ngFor` (well, actually underneath it uses `NgForRow`, a class which generates each `ngFor` item internally). Let's quickly look at the [source code](https://github.com/angular/angular/blob/8578682dcfccc2ccccb68df37c3ebc738f98bf0e/modules/%40angular/common/src/directives/ng_for.ts) for this:

```js
export class NgForRow {
  constructor(public $implicit: any, public index: number, public count: number) {}
  get first(): boolean { return this.index === 0; }
  get last(): boolean { return this.index === this.count - 1; }
  get even(): boolean { return this.index % 2 === 0; }
  get odd(): boolean { return !this.even; }
}
```

As I mentioned above, the `NgForRow` is what constructs our `ngFor` items, and you can see in the `constructor` we've already taken a look at `index` and `count`! The last things we need to look at are the getters, which we can explain from the source code above:

* _first_: returns `true` for the first item in the collection, matches the index with zero

* _last_: returns `true` for the last item in the collection, matches the index with the total count, minus one to shift the "count" down one to cater for zero-based indexes

* _even_: returns `true` for even items (e.g. 2, 4) in the collection, uses `%` modulus operator to calculate based off index

* _odd_: returns `true` for odd items (e.g. 1, 3), simply inverts `this.even` result

Using this, we can add conditionally apply things such as styling, or hook into the `last` property to know when the collection has finished rendering.

For this quick demo, we'll use `ngClass` to add some styles to each `<li>` (note how we create more variables, just like `index`):

```html
<ul>
  <li
    *ngFor="let contact of contacts | async; let o = odd; let e = even;"
    [ngClass]="{
      'odd-active': o,
      'even-active': e
    }">
    <contact-card 
      [contact]="contact"
      (update)="onUpdate($event, index)">
    </contact-card>
  </li>
</ul>
```

And some styles:

```js
@Component({
  selector: 'app-root',
  styles: [`
    .odd-active { background: purple; color: #fff; }
    .even-active { background: red; color: #fff; }
  `],
  template: `
    <div class="app">
      <ul>
        <li
          *ngFor="let contact of contacts | async; let o = odd; let e = even;"
          [ngClass]="{
            'odd-active': o,
            'even-active': e
          }">
          <contact-card 
            [contact]="contact"
            (update)="onUpdate($event, index)">
          </contact-card>
        </li>
      </ul>
    </div>
  `
})
```

We won't demonstrate `first` and `last`, as it's fairly obvious from the above how we can hook those up!

You can check a live output of what we've covered here:

<iframe src="//embed.plnkr.co/5YvhRZYT8W5mLDY898mV?deferRun" frameborder="0" border="0" cellspacing="0" cellpadding="0" width="100%" height="400"></iframe>

### &lt;template&gt; element

We mentioned earlier in this article that we'd look at understanding what the `*` meant in our templates. This also shares the same syntax as `*ngIf`, which you've likely also seen before.

So in this next section, we'll take a deeper dive on `ngFor`, `*` and the `<template>` element to explain in more detail what's really happening here.

When using an asterisk (`*`) in our templates, we are informing Angular we're using a structural directive, which is also sugar syntax (a nice short hand) for using the `<template>` element.

#### &lt;template&gt; and Web Components

So, what is the `<template>` element? First, let's take a step back. We'll roll back to showing some Angular 1.x code here, perhaps you've done this before or done something similar in another framework/library:

```html
<script id="myTemplate" type="text/ng-template">
  <div>
    My awesome template!
  </div>
</script>
```

This overrides the `type` on the `<script>` tag, which prevents the JavaScript engine from parsing the contents of the `<script>` tag. This allows us, or a framework such as Angular 1.x, to fetch the contents of the script tag and use it as some form of HTML template.

Web Components introduced something similar to this idea, called the `<template>`:

```html
<template id="myTemplate">
  <div>
    My awesome template!
  </div>
</template>
```

To grab our above template and instantiate it, we'd do this in plain JavaScript:

```js
<div id="host"></div>
<script>
  let template = document.querySelector('#myTemplate');
  let clone = document.importNode(template.content, true);
  let host = document.querySelector('#host');
  host.appendChild(clone);
</script>
```

Note how we have `id=host`, which is our "host" Node for the template to be injected into.

You may have seen this term floating around Angular in a few ways, such as `_nghost` prefixes on Nodes (ng-host) or the `host` property in directives.

#### ngFor and &lt;template&gt;

So how does the above `<template>` explanation tell us more about `ngFor` and the `*`? The asterisk is shorthand syntax for using the `<template>` element.

Let's start from the basic `ngFor` example:

```html
<ul>
  <li *ngFor="let contact of contacts | async">
    <contact-card [contact]="contact"></contact-card>
  </li>
</ul>
```

And demonstrate the `<template>` equivalent:

```html
<ul>
  <template ngFor let-contact [ngForOf]="contacts | async">
    <li>
      <contact-card [contact]="contact"></contact-card>
    </li>
  </template>
</ul>
```

That's a lot different! What's happening here?

When we use `*ngFor`, we're telling Angular to essentially treat the element the `*` is bound to as a template.

> Note: Angular's `<template>` element is not a true Web Component, it merely mirrors the concepts behind it to allow you to use `<template>` as it's intended in the spec. When we Ahead-of-Time compile, we will see no `<template>` elements. However, this doesn't mean we can't use things like Shadow DOM, as they are still [completely possible](/emulated-native-shadow-dom-angular-2-view-encapsulation).

Let's continue, and understand what `ngFor`, `let-contact` and `ngForOf` are doing above.

#### ngFor and embedded view templates

First thing's first, `ngFor` is a directive! Let's check some of the source code:

```js
@Directive({selector: '[ngFor][ngForOf]'})
export class NgFor implements DoCheck, OnChanges {...}
```

Here, Angular is using attribute selectors as the value of `selector` to tell the `@Directive` decorator what attributes to look for (don't confuse this with property binding such as `<input [value]="foo">`).

The directive uses `[ngFor][ngForOf]`, which implies there are two attributes as a chained selector. So, how does `ngFor` work if we're not using `ngForOf`?

Angular's compiler transforms any `<template>` elements and directives used with a asterisk (`*`) into views that are separate from the root component view. This is so each view can be created multiple times.

> During the compile phase, it will take `let contact of contacts` and capitalise the `of`, and create a custom key to create `ngForOf`.

In our case, Angular will construct a view that creates everything from the `<li>` tag inwards:

```html
<!-- view -->
<li>
  <contact-card [contact]="contact"></contact-card>
</li>
<!-- /view -->
```

It also creates an invisible view container to contain all instances of the template, acting as a placeholder for content. The view container Angular has created essentially wraps the "views", in our case this is just inside the `<ul>` tags. This houses all the templates that are created by `ngFor` (one for each row).

A pseudo-output might look like this:

```html
<ul>
<!-- view container -->
  <!-- view -->
  <li>
    <contact-card [contact]="contact"></contact-card>
  </li>
  <!-- /view -->
  <!-- view -->
  <li>
    <contact-card [contact]="contact"></contact-card>
  </li>
  <!-- /view -->
  <!-- view -->
  <li>
    <contact-card [contact]="contact"></contact-card>
  </li>
  <!-- /view -->
<!-- /view container -->
</ul>
```

`ngFor` creates an "embedded view" for each row, passing through the view it has created and the context of the row (the index and the row data). This embedded view is then inserted into the view container. When the data changes, it tracks the items to see if they have moved. If they've moved, instead of recreating the embedded views, it moves them around to be in the correct position, or destroys them if they no longer exist.

#### Context and passing variables

The next step is understanding how Angular passes the context to each `<template>`:

```html
<template ngFor let-contact [ngForOf]="contacts | async">
  <li>
    <contact-card [contact]="contact"></contact-card>
  </li>
</template>
```

So now we've understood `ngFor` and `ngForOf`, how does Angular associate `let-contact` with the individual `contact` that we then property bind to?

Because `let-contact` has no value, it's merely an attribute, this is where Angular provides an "implied" value, or `$implicit` as it's called under-the-hood.

Whilst Angular is creating each `ngFor` item, it uses an `NgForRow` class alongside an `EmbeddedViewRef`, and passes these properties in dynamically. Here's a small snippet from the source code:

```js
changes.forEachIdentityChange((record: any) => {
  const viewRef = <EmbeddedViewRef<NgForRow>>this._viewContainer.get(record.currentIndex);
  viewRef.context.$implicit = record.item;
});
```

Alongside this section of code, we can also see how our aforementioned `index` and `count` properties are kept updated:

```js
for (let i = 0, ilen = this._viewContainer.length; i < ilen; i++) {
  const viewRef = <EmbeddedViewRef<NgForRow>>this._viewContainer.get(i);
  viewRef.context.index = i;
  viewRef.context.count = ilen;
}
```

> You can dig through the directive source code in more detail [here](https://github.com/angular/angular/blob/d4d3782d455a484e8aa26ec9a57ee2b4727bc1da/modules/%40angular/common/src/directives/ng_for.ts).

This is how we can then access the `index` and `context` like this:

```html
<ul>
  <template ngFor let-i="index" let-c="count" let-contact [ngForOf]="contacts | async">
    <li>
      <contact-card [contact]="contact"></contact-card>
    </li>
  </template>
</ul>
```

Note how we're supplying `let-i` and `let-c` _values_ which are exposed from the `NgForRow` instance, unlike `let-contact`.

You can check out the `<template>` version of the things we've covered here:

<iframe src="//embed.plnkr.co/7FqWhp9u2zzX6qT7uF1H?deferRun" frameborder="0" border="0" cellspacing="0" cellpadding="0" width="100%" height="400"></iframe>
