---
layout: post
permalink: /component-events-event-emitter-output-angular-2
title: "Component events with EventEmitter and @Output in Angular 2+"
path: 2016-03-19-component-events-event-emitter-output-angular-2.md
tags:
- Angular 2
---

Angular components have a far better way of notifying parent components that something has changed, via events. There's no longer two-way data binding in Angular in the same way we knew it in AngularJS, it's designed around a uni-directional data flow system that adopts a much more reasonable approach to application development.

Let's finalise the basics of parent-child and child-parent communication by introducing `EventEmitter` and `@Output`.

### Series

1. [Bootstrapping your first Angular app](/bootstrap-angular-2-hello-world)
2. [Creating your first Angular component](/creating-your-first-angular-2-component)
3. [Passing data into Angular components with @Input](/passing-data-angular-2-components-input)
4. Component events with EventEmitter and @Output in Angular

### Introduction

This tutorial will cover stateless component events using the `EventEmitter` API and `@Output` decorator. These hand in hand allow us to emit change or any custom event names from a custom component in Angular.

This post follows from the previous article on [passing data in Angular components with @Input](/passing-data-angular-2-components-input).

### AngularJS

For those coming from an AngularJS background, this concept could look a little like this with the `.component()` API and callback binding using `'&'`:

```js
const counter = {
  bindings: {
    count: '<',
    onChange: '&'
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
      this.onChange(this.value);
    };
    this.decrement = () => {
      this.count--;
      this.onChange(this.value);
    };
  }
};

angular
  .module('app')
  .component('counter', counter);
```

The key ingredient here is using the callback `onChange: '&'` syntax. This means we're expecting a function to be passed down from the parent component, and we can call it when our count changes (via `this.onChange()`) - which essentially is callback registered in the parent that we call in the child and pass new data into, such as this:

```html
<div class="parent">
  <counter
    value="$ctrl.someValue"
    on-change="$ctrl.valueChanged($event)">
  </counter>
</div>
```

Once the child component calls `$ctrl.valueChanged`, the data is passed into it and we are able to access that new piece of data through the `$event` object.

### Stateful (parent) component binding

Much like in the previous tutorial where we setup an `@Input` decorator to accept an input binding, we can do the same and listen in the parent for when a value changes inside our child component.

To do this, we'll head back to our parent component that's rendering out our `<counter>`:

```js
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div class="app">
      Parent: {% raw %}{{ myCount }}{% endraw %}
      <counter
        [count]="myCount"
        (change)="countChange($event)">
      </counter>
    </div>
  `
})
export class AppComponent {
  myCount: number = 10;
  countChange(event) {

  }
}
```

I've made a few additions here:

* Changed `initialCount` to `myCount`, we are no longer setting an "initialCount", therefore the count state will be managed in the parent once the child component makes a change to it
* Created a custom `change` property to the `<counter>` template, using `()` event binding syntax, like we learned when we created our first component this signifies some kind of event (such as a `click` when used on a native element Node).
* Logged the `myCount` property in the parent
* Added a `countChange() {}` method to the class, and passed it into the `(change)` event listener

This sets up our finalised uni-directional dataflow. The data flows down from the `AppComponent` class, into the `<counter>`, the counter can then change the values - and once the value has changed we expect `countChange()` to be called. We now need to wire this up.

### @Output decorator

Much like using `Input`, we can import `Output` and decorate a new `change` property inside our `CounterComponent`:

```js
import { Component, Input, Output } from '@angular/core';

@Component({...})
export class CounterComponent {

  @Input()
  count: number = 0;

  @Output()
  change;

  // ...

}
```

This will configure the metadata necessary to tell Angular this property is to be treated as an output binding. However, it needs to sit alongside something called the `EventEmitter`.

### EventEmitter

This is the interesting part. The be able to use our `Output`, we need to import and bind a new instance of the `EventEmitter` to it:

```js
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({...})
export class CounterComponent {

  // ...

  @Output()
  change = new EventEmitter();

  // ...

}
```

Using TypeScript to the fullest we'd do something like this to signify the _type_ of event value we are emitting, and our `change` output is of type `EventEmitter`. In our case we are emitting a `number` type:

```js
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({...})
export class CounterComponent {

  // ...

  @Output()
  change: EventEmitter<number> = new EventEmitter<number>();

  // ...

}
```

### Invoking the EventEmitter instance

So what's happening here? We've created a `change` property, and bound a new instance of `EventEmitter` to it - what next?

Much like the AngularJS example I showed at the beginning, we can simply call our `this.change` method - however because it references an instance of `EventEmitter`, we have to call `.emit()` to emit an event to the parent:

```js
@Component({...})
export class CounterComponent {

  @Input()
  count: number = 0;

  @Output()
  change: EventEmitter<number> = new EventEmitter<number>();

  increment() {
    this.count++;
    this.change.emit(this.count);
  }

  decrement() {
    this.count--;
    this.change.emit(this.count);
  }

}
```

This will then emit a change to our `(change)` listener we setup in the parent, to which our `countChange($event)` callback will be invoked, and the data associated with the event will be given to us via the `$event` property.

### Stateful callback assignment

Here's what we'll need to do, re-assign `this.myCount` with the `event` that's passed back. I'll explain why below:

```js
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div class="app">
      Parent: {% raw %}{{ myCount }}{% endraw %}
      <counter
        [count]="myCount"
        (change)="countChange($event)">
      </counter>
    </div>
  `
})
export class AppComponent {
  myCount: number = 10;
  countChange(event) {
    this.myCount = event;
  }
}
```

Why do we do this? This creates a pure uni-directional dataflow. The data comes from `AppComponent`, flows into `<counter>`, the counter makes a change, and emits that change back to the parent on our command - via the `EventEmitter` we setup. Once we've got that data back up, we merge those changes back into our parent (stateful) component.

The reason we're doing this is to demonstrate that `Parent: {% raw %}{{ myCount }}{% endraw %}` updates at the same time our `Output` informs the parent.

### Bonus: custom property names

Much [like we learned](/passing-data-angular-2-components-input#bonus-custom-property-names) with `@Input()` and creating custom property names, we can also do the same with `@Output()`.

Let's assume that we change the `(change)` binding to `(update)`:

```js
@Component({
  selector: 'app-root',
  template: `
    <div class="app">
      Parent: {% raw %}{{ myCount }}{% endraw %}
      <counter
        [count]="myCount"
        (update)="countChange($event)">
      </counter>
    </div>
  `
})
export class AppComponent {
  myCount: number = 10;
  countChange(event) {
    this.myCount = event;
  }
}
```

We can hook up our custom property name, whilst preserving the internal `@Output` property name:

```js
@Component({...})
export class CounterComponent {

  // ...

  @Output('update')
  change: EventEmitter<number> = new EventEmitter<number>();

  increment() {
    this.count++;
    this.change.emit(this.count);
  }

  decrement() {
    this.count--;
    this.change.emit(this.count);
  }

}
```

Essentially, we're just telling Angular here to lookup `update` as the property to be bound to, and we can continue using `this.change` internally.

### Plunker

Everything we've done here is readily available in a Plunker for you to have a look through.

You can try the version _without_ the `this.myCount` callback assignment here to see how the local state change doesn't update the parent:

<iframe src="//embed.plnkr.co/ZO9AbAE2780IURxwicN5?deferRun" frameborder="0" border="0" cellspacing="0" cellpadding="0" width="100%" height="250"></iframe>

...And try the version that _does_ update the parent here:

<iframe src="//embed.plnkr.co/i7A9Igg8NkcnDJf2bohx?deferRun" frameborder="0" border="0" cellspacing="0" cellpadding="0" width="100%" height="250"></iframe>

Happy coding!
