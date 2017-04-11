---
layout: post
permalink: /passing-data-angular-2-components-input
title: "Passing data into Angular 2+ components with @Input"
path: 2016-03-18-passing-data-angular-2-components-input.md
tags:
- Angular 2
---

In a component-driven application architecture we typically use [stateful and stateless](/stateful-stateless-components) components. A key concept is having some form of "stateful" component that delegates down into a "stateless" child, or children, component(s).

We do this through something called property binding in Angular, which we learned about in the previous article when we bound to an `<input>` element to display a count. To bind data to Angular components, we need to create a custom property bind, which is done via "input" binding to pass data from one component to another (typically parent to child).

This custom input binding is created via the `@Input()` decorator! Let's explore.

### Series

1. [Bootstrapping your first Angular app](/bootstrap-angular-2-hello-world)
2. [Creating your first Angular component](/creating-your-first-angular-2-component)
3. Passing data into Angular components with @Input
4. [Component events with EventEmitter and @Output in Angular](/component-events-event-emitter-output-angular-2)

### Introduction

This tutorial will cover passing data into a component, and we'll be using the Counter component we built in the previous article. If you've not dived in and learned how to create a component in Angular, [check that out here](/creating-your-first-angular-2-component), as we'll be using the same source code to continue building.

### AngularJS

For those coming from an AngularJS background, this concept looks a little like this with the [.component method](/exploring-the-angular-1-5-component-method/):

```js
const counter = {
  bindings: {
    count: '<'
  },
  template: `
    <div class="counter">
      <button ng-click="$ctrl.decrement()">
        Decrement
      </button>
      <input type="text" ng-model="$ctrl.count">
      <button ng-click="$ctrl.increment()">
        Increment
      </button>
    </div>
  `,
  controller() {
    this.$onInit = () => {
      this.count = this.count || 0;
    };
    this.increment = () => {
      this.count++;
    };
    this.decrement = () => {
      this.count--;
    };
  }
};

angular
  .module('app')
  .component('counter', counter);
```

The key ingredient here is using the `bindings` object to declare `count` as an input [one-way binding](/one-way-data-binding-in-angular-1-5/). The old school way was using `scope` and perhaps `bindToController` with the `.directive()` method.

### Stateful (parent) component binding

Using the same concept above with `bindings`, which relies on parent data, we need to tell Angular what is coming into our component. We'll create a stateful parent component, where we can set some initial data to be delegated down into our `CounterComponent`.

In the [previous article](/creating-your-first-angular-2-component), we registered our `CounterComponent` in our `@NgModule` which allows us to use it inside our module's registered components.

Jumping to our `AppComponent`, this means we can declare it as a custom element _inside_ the `template`:

```js
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div class="app">
      <counter></counter>
    </div>
  `
})
export class AppComponent {
  initialCount: number = 10;
}
```

So what about `initialCount`? We need to bind it to our component!

We learned about property binding in the previous article, and the same applies with our own custom components. The difference lies in we have to _tell_ Angular the name of the property binding. This will make more sense momentarily, but let's create a binding called `count` on our component and pass _through_ our `initialCount` value:

```js
@Component({
  selector: 'app-root',
  template: `
    <div class="app">
      <counter [count]="initialCount"></counter>
    </div>
  `
})
export class AppComponent {
  initialCount: number = 10;
}
```

To recap quickly, we're creating a custom property called `count`, and supplying the value of `initialCount`, which can be any number.


### @Input decorator, stateless component

Now we're creating a stateless, or "dumb" component, to pass our data _into_, which we can mutate locally and get data back _out_. We'll be getting new data back out of the component in the next article.

Let's jump into our `CounterComponent` (some `@Component` metadata has been removed for brevity):

```js
import { Component } from '@angular/core';

@Component({...})
export class CounterComponent {

  count: number = 0;

  increment() {
    this.count++;
  }

  decrement() {
    this.count--;
  }

}
```

There is one key thing we need to do here. At the moment we have a fully isolated component in terms of data, but we need to be able to pass data into this component.

To do this, we can import the `Input` decorator from the Angular core, and simply decorate the `count` property:

```js
import { Component, Input } from '@angular/core';

@Component({...})
export class CounterComponent {

  @Input()
  count: number = 0;

  increment() {
    this.count++;
  }

  decrement() {
    this.count--;
  }

}
```

This decorator tells Angular to treat `count` as an input binding, much like the AngularJS 1.x `'<'` syntax if you're coming from an AngularJS 1.x background. This syntax is easier and shorter, as we only manage our bindings in a single place, rather than a `bindings` object like we saw at the beginning of this tutorial.

> Tip: you can see `number = 0`, we're keeping this as an optional default value, so if no input binding is supplied, the component will be initialised with a count of `0`.

And that's all you need to do!

### Bonus: custom property names

It may be that you'd want your "public" property names to differ from the internal input names. Here's what we might want to do:

```js
@Component({
  selector: 'app-root',
  template: `
    <div class="app">
      <counter [init]="initialCount"></counter>
    </div>
  `
})
export class AppComponent {
  initialCount: number = 10;
}
```

You can see I've changed `[count]` to `[init]`, so how does this now affect our internal input binding inside the `CounterComponent`? Currently, this will break:

```js
@Component({...})
export class CounterComponent {

  @Input()
  count: number = 0;

  // ...

}
```

Why? Because `count` is no longer being bound to, we're trying to bind to an `init` property instead. To keep the internal property name(s) different to the public names, we can do this:

```js
@Component({...})
export class CounterComponent {

  @Input('init')
  count: number = 0;

  // ...

}
```

We simply pass a string into the `@Input()` decorator with the name of the property we want to bind to. That's it, and we can use `this.count` as usual inside `CounterComponent`.

### Plunker

Everything we've done here is readily available in a Plunker for you to have a look through:

<iframe src="//embed.plnkr.co/hsx0gnjiaNLeWGNu0Tzb?deferRun" frameborder="0" border="0" cellspacing="0" cellpadding="0" width="100%" height="250"></iframe>

### Next steps

Wouldn't it be great to be notified of changes when the internal `counterValue` (inside `CounterComponent`) has changed? Well, instead of `@Input`, we can use `@Output` and `EventEmitter` - [let's explore in the next tutorial](/component-events-event-emitter-output-angular-2).
