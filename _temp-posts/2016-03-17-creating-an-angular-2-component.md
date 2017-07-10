---
layout: post
permalink: /creating-your-first-angular-2-component
title: Creating your first Angular 2+ component
path: 2016-03-17-creating-an-angular-2-component.md
tags:
- Angular 2
---

This is a beginner level tutorial to ease you into Angular (v2+), although there are many resources online to creating components, these articles exist as part of a series. This article will guide you through creating your first Angular component.

### Series

1. [Bootstrapping your first Angular app](/bootstrap-angular-2-hello-world)
2. Creating your first Angular component
3. [Passing data into Angular components with @Input](/passing-data-angular-2-components-input)
4. [Component events with EventEmitter and @Output in Angular](/component-events-event-emitter-output-angular-2)

### Introduction

For the purposes of this tutorial, we'll be creating a "counter" component that allows for incrementing and decrementing of values via buttons, which then change the value of an `<input>`.

Before creating your first component, you'll need to learn how to [bootstrap an Angular app](/bootstrap-angular-2-hello-world) before continuing.

### Creating an ES6/TypeScript class

All components in Angular are classes, and to tell Angular they're a component we use a special decorator which we'll move onto in the next section, however for now, let's create a class:

```js
class AppComponent {

}
```

Inside this class, we can add properties, such as a message:

```js
class AppComponent {
  message: string = 'Hello world!';
}
```

If you're new to TypeScript, you may be more familiar with this approach:

```js
class AppComponent {
  constructor() {
    this.message = 'Hello world!';
  }
}
```

These are essentially the same thing, but using TypeScript we can declare the types of properties we're using, for instance I'm saying `message: string`, denoting that it will be of type "string". I've also given it a default value of "Hello world!" as well, which may be done dynamically inside a real world application.

From here, we need to somehow render this message into the component, which is where we need to create a template to enable us to bind the message to the component.

### Using the @Component decorator

To tell Angular that our class is a component, we need to import the component decorator and use it on our class.

> Decorators are just functions, you can read my in-depth guide to [Angular decorators](/angular-decorators) once you're more familiar with using them.

To import the component decorator, we simply grab it from the Angular `core` module:

```js
// app.component.ts
import { Component } from '@angular/core';

export class AppComponent {
  message: string = 'Hello world!';
}
```

Now `Component` is imported, we simply add it above our class (which is called decorating a class):

```js
// app.component.ts
import { Component } from '@angular/core';

@Component()
export class AppComponent {
  message: string = 'Hello world!';
}
```

> There's an official [TC39 proposal](https://github.com/tc39/proposal-decorators) for decorators, currently at Stage-2, so expect decorators to become a core language feature soon in JavaScript as well.

The next two things we need are configuration properties `selector` and `template`:

```js
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div class="app">
      {% raw %}{{ message }}{% endraw %}
    </div>
  `
})
export class AppComponent {
  message: string = 'Hello world!';
}
```

You can see we're using `{% raw %}{{ message }}{% endraw %}` to interpolate the values of the class in which they correspond with the template, this will then render "Hello world!" dynamically out for us in the browser.

It's fairly obvious what `template` does here, but what does `selector` mean? The `selector` property creates a component under the name of the string you just passed in, to use it like so:

```html
<app-root>
  Loading...
</app-root>
```

We've simply put some _"Loading..."_ text inside of here, which you can customise if you'd like, to display as the client-side application is loading.

### Creating a counter component

So let's move onto a more complex example and create a `CounterComponent` we mentioned in the introduction.

#### Component definition

By now we should understand how to do this based off the above explanation:

```js
// counter.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'counter',
  template: `
    <div class="counter">
      {% raw %}{{ count }}{% endraw %}
    </div>
  `
})
export class CounterComponent {
  count: number = 0;
}
```

#### Property binding to an &lt;input&gt;

To bind our `count` to an `<input>`, we need to use something called property binding, whereby we bind to a specific property on a DOM element (we can also bind to components, which we'll learn in the next tutorial).

So, which property do we need to bind to? The `value` property! You've likely done something like this before:

```html
<input type="text" value="Hello">
```

This creates an `<input>` with a default value of `Hello`. To bind a value from our component class to the template, we need to do this:

```js
@Component({
  selector: 'counter',
  template: `
    <div class="counter">
      <input type="text" [value]="count">
    </div>
  `
})
export class CounterComponent {
  count: number = 0;
}
```

The `[]` square bracket notation here signifies a property binding, which as you build out Angular apps will become more clear and visually help you with what types of bindings you are making.

To think about property binding in a simpler way, take `element.value` for example. The `value` is a property on the DOM Node object, and we can do this as well to look it up:

```js
element['value'] = 'Hello';
```

It's an easier trick to remember that you are essentially asking for a built-in JavaScript property when using Angular syntax. It will _set_ your property with the value supplied, in our case we are supplying a dynamic `count` value, which is subject to change.

#### Component methods

So to increment and decrement our count, we can create two methods on our class:

```js
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

These methods directly manipulate the `count` property inside our component, which will automatically be reflected in our template due to the property binding we have made. We need to add buttons for the user to increment and decrement the value:

```js
@Component({
  selector: 'counter',
  template: `
    <div class="counter">
      <button>
        Decrement
      </button>
      <input type="text" [value]="count">
      <button>
        Increment
      </button>
    </div>
  `
})
export class CounterComponent {
  // ...
}
```

Now that we have buttons for the user to click, we need to bind a `click` event to each button. This is done through event binding, which uses rounded brackets `()` instead of square brackets `[]`. Inside the rounded brackets, we need to specify the name of the event that we want to listen for:

```js
@Component({
  selector: 'counter',
  template: `
    <div class="counter">
      <button (click)="decrement()">
        Decrement
      </button>
      <input type="text" [value]="count">
      <button (click)="increment()">
        Increment
      </button>
    </div>
  `
})
export class CounterComponent {
  // ...
}
```

We pass the callback method as the value of the added attribute. You can think of it like we are calling `addEventListener()` on an `element` Node:

```js
element.addEventListener('click', increment);
```

#### Styling the component

We'll introduce one more concept, which is styling. To do this we can add a `styles` property to our `@Component` decorator and pass an array of strings:

```js
@Component({
  selector: 'counter',
  styles: [`
    .counter {
      position: relative;
    }
    input {
      border: 0;
      border-radius: 3px;
      height: 30px;
      max-width: 100px;
      text-align: center;
    }
    button {
      outline: 0;
      cursor: pointer;
      height: 30px;
      border: 0;
      border-radius: 3px;
      background: #0088cc;
      color: #fff;
    }
  `],
  template: `
    <div class="counter">
      <button (click)="decrement()">
        Decrement
      </button>
      <input type="text" [value]="count">
      <button (click)="increment()">
        Increment
      </button>
    </div>
  `
})
export class CounterComponent {
  // ...
}
```

Angular supports multiple style declarations per component but most of the time we'll only need to pass one in. This is useful if you have shared styles between components, you can create a file that both components use that contain their styles. An alternative is using `styleUrls` instead, which allows us to use external styles and have them compiled through a preprocessor such as Sass or Less:

```js
@Component({
  selector: 'counter',
  styleUrls: ['counter.component.scss'],
  template: `
    <div class="counter">
      <button (click)="decrement()">
        Decrement
      </button>
      <input type="text" [value]="count">
      <button (click)="increment()">
        Increment
      </button>
    </div>
  `
})
export class CounterComponent {
  // ...
}
```

> Angular also supports an external template for a component should you wish to separate them out into individual files. You can specify this via `templateUrl` and point to the file.

### @NgModule registration

Now we've created our new Angular component, we must add it to our `@NgModule` definition:

```js
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

// import our new component!
import { CounterComponent } from './counter.component.ts';

import { AppComponent } from './app.component.ts';

@NgModule({
  declarations: [
    AppComponent,
    // register it inside the declarations array
    CounterComponent
  ],
  imports: [
    BrowserModule
  ],
  bootstrap: [
    AppComponent
  ]
})
export class AppModule {}
```

This important step allows us to use the `CounterComponent` in our application, such as `<counter>`.

### Plunker

Everything we've done here is readily available in a Plunker for you to have a look through:

<iframe src="//embed.plnkr.co/Zev2kRraWB1WnEbFBoo0?deferRun" frameborder="0" border="0" cellspacing="0" cellpadding="0" width="100%" height="250"></iframe>

### Next steps

Now we've learned how to do the basics, let's move on and learn how to [pass data into Angular Components with @Input](/passing-data-angular-2-components-input).
