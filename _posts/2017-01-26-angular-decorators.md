---
layout: post
permalink: /angular-decorators
title: "A deep dive on Angular decorators"
path: 2017-01-26-angular-decorators.md
---

Decorators are a core concept when developing with Angular (versions 2 and above). There's also an official [TC39 proposal](https://github.com/tc39/proposal-decorators), currently at Stage-2, so expect decorators to become a core language feature soon in JavaScript as well.

Back to Angular, the internal codebase uses decorators extensively and in this post we’re going to look at the different types of decorators, the code they compile to and how they work.

When I was first introduced to TypeScript and decorators, I wondered why we needed them at all, but once you dig a little deeper you can understand the benefits to creating decorators (not only for use in Angular).

AngularJS didn't use decorators, opting for a different registration method - such as defining a component for example with the `.component()` method. So why has Angular chose to use them? Let's explore.

### Table of contents

- <a href="#angular-decorators">Angular Decorators</a>
  - <a href="#class-decorators">Class Decorators</a>
  - <a href="#property-decorators">Property Decorators</a>
  - <a href="#method-decorators">Method Decorators</a>
  - <a href="#parameter-decorators">Parameter Decorators</a>
- <a href="#creating-a-decorator">Creating a decorator</a>
  - <a href="#decorator-functions">Decorator Functions</a>
  - <a href="#passing-data-to-a-decorator">Passing data to a decorator</a>
- <a href="#what-angular-decorators-actually-do">What Angular decorators actually do</a>
  - <a href="#storing-metadata">Storing metadata</a>
  - <a href="#chaining-decorators">Chaining decorators</a>
- <a href="#how-decorators-are-applied">How decorators are applied</a>
- <a href="#summary">Summary</a>

### Angular Decorators

Before we look at creating a custom decorator and why/how Angular uses them, let's look at the different types of decorators that Angular offers. There are four main types:

- Class decorators, e.g. `@Component` and `@NgModule`
- Property decorators for properties inside classes, e.g. `@Input` and `@Output`
- Method decorators for methods inside classes, e.g. `@HostListener`
- Parameter decorators for parameters inside class constructors, e.g. `@Inject`

Each decorator has a unique role, let's jump to some examples to expand on the list above.

#### Class Decorators

Angular offers us a few class decorators. These are the top-level decorators that we use to express _intent_ for classes. They allow us to tell Angular that a particular class is a component, or module, for example. And the decorator allows us to define this intent without having to actually put any code inside the class.

A `@Component` and `@NgModule` decorator example with classes:

```js
import { NgModule, Component } from '@angular/core';

@Component({
  selector: 'example-component',
  template: '<div>Woo a component!</div>'
})
export class ExampleComponent {
  constructor() {
    console.log('Hey I am a component!');
  }
}

@NgModule({
  imports: [],
  declarations: []
})
export class ExampleModule {
  constructor() {
    console.log('Hey I am a module!');
  }
}
```

Notice how both classes by themselves are effectively the same. No code is needed within the class to tell Angular that it is a component or a module. All we need to do is decorate it, and Angular will do the rest.

#### Property Decorators

These are probably the second most common decorators that you'll come across. They allow us to decorate specific properties within our classes - an extremely powerful mechanism.

Let's take a look at `@Input()`. Imagine that we have a property within our class that we want to be an input binding.

Without decorators, we'd have to define this property in our class anyway for TypeScript to know about it, and then somewhere else tell Angular that we've got a property that we want to be an input.

With decorators, we can simply put the `@Input()` decorator above the property - which Angular's compiler will automatically create an input binding from the property name and link them.

<div class="language-js highlighter-rouge"><pre class="highlight"><code><span class="kr">import</span> <span class="p">{</span> <span class="nx" style="opacity: 0.3">Component</span><span class="p" style="opacity: 0.3">,</span> <span class="nx">Input</span> <span class="p">}</span> <span class="nx">from</span> <span class="s1">'@angular/core'</span><span class="p">;</span>

<span style="opacity: 0.3"><span class="err">@</span><span class="nx">Component</span><span class="p">({</span>
  <span class="na">selector</span><span class="p">:</span> <span class="s1">'example-component'</span><span class="p">,</span>
  <span class="na">template</span><span class="p">:</span> <span class="err">'</span><span class="o">&lt;</span><span class="nx">div</span><span class="o">&gt;</span><span class="nx">Woo</span> <span class="nx">a</span> <span class="nx">component</span><span class="o">!</span><span class="o">&lt;</span><span class="sr">/div</span><span class="err">&gt;</span><span class="err">'</span>
<span class="p">})</span></span>
<span class="kr">export</span> <span class="kr">class</span> <span class="nx">ExampleComponent</span> <span class="p">{</span>
  <span class="err">@</span><span class="nx">Input</span><span class="p">()</span>
  <span class="nx">exampleProperty: string</span><span class="p">;</span>
<span class="p">}</span>
</code></pre>
</div>

We'd then pass the input binding via a component property binding:

```html
<example-component
  [exampleProperty]="exampleData">
</example-component>
```

The property decorator and "magic" happens _within_ the `ExampleComponent` definition.

In AngularJS 1.x (I'm going to use TypeScript here also, just to declare a property on a class), we had a different mechanism using `scope` or `bindToController` with Directives, and `bindings` within the new [component method](/exploring-the-angular-1-5-component-method/):

```js
const exampleComponent = {
  bindings: {
    exampleProperty: '<'
  },
  template: `
    <div>Woo a component!</div>
  `,
  controller: class ExampleComponent {
    exampleProperty: string;
    $onInit() {
      // access this.exampleProperty
    }
  }
};

angular
  .module('app')
  .component('exampleComponent', exampleComponent);
```

You can see above that we have two separate properties to maintain should we expand, refactor or change our component's API - `bindings` and the property name inside the class. However, in Angular there is a single property `exampleProperty` which is decorated, which is easier to change, maintain and track as our codebase grows.

#### Method Decorators

Method decorators are very similar to property decorators but are used for methods instead. This let's us decorate specific methods within our class with functionality. A good example of this is `@HostListener`. This allows us to tell Angular that when an event on our host happens, we want the decorated method to be called with the event.

<div class="language-js highlighter-rouge"><pre class="highlight"><code><span class="kr">import</span> <span class="p">{</span> <span class="nx" style="opacity: 0.3">Component</span><span class="p" style="opacity: 0.3">,</span> <span class="nx">HostListener</span> <span class="p">}</span> <span class="nx">from</span> <span class="s1">'@angular/core'</span><span class="p">;</span>

<span style="opacity: 0.3"><span class="err">@</span><span class="nx">Component</span><span class="p">({</span>
  <span class="na">selector</span><span class="p">:</span> <span class="s1">'example-component'</span><span class="p">,</span>
  <span class="na">template</span><span class="p">:</span> <span class="s1">'&lt;div&gt;Woo a component!&lt;/div&gt;'</span>
<span class="p">})</span></span>
<span class="kr">export</span> <span class="kr">class</span> <span class="nx">ExampleComponent</span> <span class="p">{</span>
  <span class="err">@</span><span class="nx">HostListener</span><span class="p">(</span><span class="s1">'click'</span><span class="p">, [</span><span class="s1">'$event'</span><span class="p">])</span>
  <span class="nx">onHostClick</span><span class="p">(</span><span class="nx">event</span><span class="err">:</span> <span class="nx">Event</span><span class="p">)</span> <span class="p">{</span>
    <span class="c1">// clicked, `event` available</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>
</div>

#### Parameter Decorators

Parameter decorators are quite interesting. You may have come across these when injecting primitives into a constructor, where you need to manually tell Angular to inject a particular provider.

> For a deep dig into Dependency Injection (DI), tokens, `@Inject` and `@Injectable`, check out [my previous article](/angular-dependency-injection).

Parameter decorators allow us to decorate parameters in our class constructors. An example of this is `@Inject` that lets us tell Angular what we want that parameter to be initiated with:

<div class="language-js highlighter-rouge"><pre class="highlight"><code><span class="kr">import</span> <span class="p">{</span> <span class="nx" style="opacity: 0.3">Component</span><span class="p" style="opacity: 0.3">,</span> <span class="nx">Inject</span> <span class="p">}</span> <span class="nx">from</span> <span class="s1">'@angular/core'</span><span class="p">;</span>
<span class="kr">import</span> <span class="p">{</span> <span class="nx">MyService</span> <span class="p">}</span> <span class="nx">from</span> <span class="s1">'./my-service'</span><span class="p">;</span>

<span style="opacity: 0.3"><span class="err">@</span><span class="nx">Component</span><span class="p">({</span>
  <span class="na">selector</span><span class="p">:</span> <span class="s1">'example-component'</span><span class="p">,</span>
  <span class="na">template</span><span class="p">:</span> <span class="s1">'&lt;div&gt;Woo a component!&lt;/div&gt;'</span>
<span class="p">})</span></span>
<span class="kr">export</span> <span class="kr">class</span> <span class="nx">ExampleComponent</span> <span class="p">{</span>
  <span class="nx">constructor</span><span class="p">(</span><span class="err">@</span><span class="nx">Inject</span><span class="p">(</span><span class="nx">MyService</span><span class="p">)</span> <span class="nx">myService</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="nx">myService</span><span class="p">);</span> <span class="c1">// MyService</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>
</div>

Due to the metadata that TypeScript exposes for us we don't actually have to do this for our providers. We can just allow TypeScript and Angular to do the hard work for us by specifying the provider to be injected as the parameter _type_:

<div class="language-js highlighter-rouge"><pre class="highlight"><code><span style="opacity: 0.3"><span class="kr">import</span> <span class="p">{</span> <span class="nx">Component</span> <span class="p">}</span> <span class="nx">from</span> <span class="s1">'@angular/core'</span><span class="p">;</span></span>
<span class="kr">import</span> <span class="p">{</span> <span class="nx">MyService</span> <span class="p">}</span> <span class="nx">from</span> <span class="s1">'./my-service'</span><span class="p">;</span>

<span style="opacity: 0.3"><span class="err">@</span><span class="nx">Component</span><span class="p">({</span>
  <span class="na">selector</span><span class="p">:</span> <span class="s1">'example-component'</span><span class="p">,</span>
  <span class="na">template</span><span class="p">:</span> <span class="s1">'&lt;div&gt;Woo a component!&lt;/div&gt;'</span>
<span class="p">})</span></span>
<span class="kr">export</span> <span class="kr">class</span> <span class="nx">ExampleComponent</span> <span class="p">{</span>
  <span class="nx">constructor</span><span class="p">(</span><span class="nx">myService</span><span class="err">:</span> <span class="nx">MyService</span><span class="p">)</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="nx">myService</span><span class="p">);</span> <span class="c1">// MyService</span>
  <span class="p">}</span>
<span class="p">}</span>
</code></pre>
</div>

Now that we've covered the types of decorators we can use, let's dig into what they actually are doing - and why we need them.

### Creating a decorator

It makes things a lot easier if we understand what a decorator is actually doing before we look into how Angular uses them under the hood. To do this, we can create a quick example decorator.

#### Decorator functions

Decorators are actually just functions, it's as simple as that, and are called with whatever they are decorating. A method decorator will be called with the value of the method it's decorating, and a class decorator will be called with the class to be decorated.

Let's quickly make a decorator that we can use on a class to demonstrate this a little further. This decorator is just going to simply log the class to the console:

```js
function Console(target) {
  console.log('Our decorated class', target);
}
```

Here, we have created `Console` (using the uppercase naming convention Angular uses) and are specifying a single argument called `target`. The target will in fact be the class that we decorate, which means we can now decorate any class with our decorator and see it outputted in the console:

```js
@Console
class ExampleClass {
  constructor() {
    console.log('Yo!');
  }
}
```

Want to see it in action? Check out the [live demo](https://jsfiddle.net/toddmotto/kunm0xm3/).

#### Passing data to a decorator

When we use the decorators in Angular we pass in some form of configuration, specific to the decorator.

For example, when we use `@Component` we pass through an object, and with `@HostListener` we pass through a string as the first argument (the event name, such as `'click'`) and optionally an array of strings for further variables (such as `$event`) to be passed through to the decorated method.

Let's change our code above to execute the `Console` function with a value to match how we use the Angular decorators.

<div class="language-js highlighter-rouge"><pre class="highlight"><code><span class="err">@</span><span class="nx">Console</span><span class="p">(</span><span class="s1">'Hey!'</span><span class="p">)</span>
<span style="opacity: 0.3"><span class="kr">class</span> <span class="nx">ExampleClass</span> <span class="p">{</span>
  <span class="nx">constructor</span><span class="p">()</span> <span class="p">{</span>
    <span class="nx">console</span><span class="p">.</span><span class="nx">log</span><span class="p">(</span><span class="s1">'Yo!'</span><span class="p">);</span>
  <span class="p">}</span>
<span class="p">}</span></span>
</code></pre>
</div>

If we ran this code now, we'd only get `'Hey!'` outputted to the console. That's because our decorator hasn't returned a function for the class to be given to. The output of `@Console('Hey!')` is `void`.

We would need to adapt our `Console` decorator to return a function closure for the class to be given to. That way we can both receive a value from the decorator (in our case, the string `Hey!`) and also the class that it's applied to:

```js
function Console(message) {
  // access the "metadata" message
  console.log(message);
  // return a function closure, which
  // is passed the class as `target`
  return function (target) {
    console.log('Our decorated class', target);
  }
}

@Console('Hey!')
class ExampleClass {
  constructor() {
    console.log('Yo!');
  }
}

// console output: 'Hey!'
// console output: 'Our decorated class', class ExampleClass{}...
```

You can see the [changes here](https://jsfiddle.net/toddmotto/qaqeapn6/).

This is the basis for how the decorators in Angular work. They first of all take a configuration value and then receive the class/method/property to apply the decoration to. Now that we have a brief understanding of what a decorator actually does, we're going to walk through how Angular creates and uses it's own decorators.

### What Angular decorators actually do

Every type of decorator shares the same core functionality. From a purely decorative point of view, `@Component` and `@Directive` both work in the same way, as do `@Input` and `@Output`. Angular does this by using a factory for each type of decorator.

Let's look at the most common decorator in Angular, the `@Component`.

We're not going to dive into the *actual* code that Angular uses to create these decorators because we only need to understand them on a higher level.

#### Storing metadata

The whole point of a decorator is to store metadata about a class, method or property as we've already explored. When you configure a component for example, you're providing metadata for that class that tells Angular that we have a component, and that component has a specific configuration.

Each decorator has a base configuration that you can provide for it, with some defaults applied for you. When the decorator is created using the relevant factory, the default configuration is passed through. For instance, let's take a look at the possible configuration that you can use when creating a component:

```js
{
  selector: undefined,
  inputs: undefined,
  outputs: undefined,
  host: undefined,
  exportAs: undefined,
  moduleId: undefined,
  providers: undefined,
  viewProviders: undefined,
  changeDetection: ChangeDetectionStrategy.Default,
  queries: undefined,
  templateUrl: undefined,
  template: undefined,
  styleUrls: undefined,
  styles: undefined,
  animations: undefined,
  encapsulation: undefined,
  interpolation: undefined,
  entryComponents: undefined
}
```

There are a lot of different options here, and you'll notice that only one has a default value - `changeDetection`. This is specified when the decorator is created so we don't need to add it whenever we create a component. You may have applied this line of code to modify the change strategy:

```js
changeDetection: ChangeDetectionStrategy.OnPush
```

An annotation instance is created when you use a decorator. This merges the default configuration for that decorator (for instance the object you see above) with the configuration that you have specified, for example:

```js
import { NgModule, Component } from '@angular/core';

@Component({
  selector: 'example-component',
  styleUrls: ['example.component.scss'],
  template: '<div>Woo a component!</div>'
})
export class ExampleComponent {
  constructor() {
    console.log('Hey I am a component!');
  }
}
```

Would create an annotation instance with the properties of:

```js
{
  selector: 'example-component',
  inputs: undefined,
  outputs: undefined,
  host: undefined,
  exportAs: undefined,
  moduleId: undefined,
  providers: undefined,
  viewProviders: undefined,
  changeDetection: ChangeDetectionStrategy.Default,
  queries: undefined,
  templateUrl: undefined,
  template: '<div>Woo a component!</div>',
  styleUrls: ['example.component.scss'],
  styles: undefined,
  animations: undefined,
  encapsulation: undefined,
  interpolation: undefined,
  entryComponents: undefined
}
```

Once this annotation instance has been created it is then stored so Angular can access it.

#### Chaining decorators

If a decorator is used on a class for the first time, it creates a new array and pushes the annotation instance into it. If this isn't the first decorator that has been used on the class, it pushes it into the existing annotation array. This allows decorators to be chained together and all stored in one place.

For example, in Angular you could do this for a property inside a class:

```js
export class TestComponent {
  @Input()
  @HostListener('click', ['$event'])
  onClick: Function;
}
```

At the same time, Angular also uses the reflect API (commonly polyfilled using `reflect-metadata`) to store these annotations, using the class as an array. This means that it can then later on fetch all of the annotations for a specific class just by being pointed to the class.

### How decorators are applied

So we know now how and why Angular uses decorators, but how are they actually applied to a class?

As mentioned, decorators aren't native to JavaScript just yet - TypeScript currently provides the functionality for us. This means that we can check the compiled code to see what actually happens when we use a decorator.

Take a standard, ES6 class -

```js
class ExampleClass {
  constructor() {
    console.log('Yo!');
  }
}
```

TypeScript will then convert this over to a function for us:

```js
var ExampleClass = (function () {
  function ExampleClass() {
    console.log('Yo!');
  }
  return ExampleClass;
}());
```

Now, if we decorate our class, we can see where the decorators are then actually applied.

```js
@ConsoleGroup('ExampleClass')
class ExampleClass {
  constructor() {
    console.log('Yo!');
  }
}
```

TypeScript then outputs:

```js
var ExampleClass = (function () {
  function ExampleClass() {
    console.log('Yo!');
  }
  return ExampleClass;
}());
ExampleClass = __decorate([
  ConsoleGroup('ExampleClass')
], ExampleClass);
```

This gives us some actual context as to how our decorators are applied.

The `__decorate` call is a helper function that would be outputted at the top of our compiled file. All that this does is apply our decorators to our class, (calling `ConsoleGroup('ExampleClass')` with `ExampleClass` as the argument).

### Summary

Demystifying decorators is one step into understanding some more of the Angular "magic" and how Angular uses them. They give Angular the ability to store metadata for classes and streamline our workflow simultaneously.
