---
layout: post
permalink: /angular-constructor-ngoninit-lifecycle-hook
title: "Angular constructor versus ngOnInit"
path: 2017-03-06-constructor-ngoninit.md
tag: angular
---


Angular has [many lifecycle hooks](https://angular.io/docs/ts/latest/guide/lifecycle-hooks.html), as well as a `constructor`. In this post, we'll quickly cover the differences between the `ngOnInit` lifecycle hook which has been the source of some confusion for those getting started with Angular.

So, why do we need the `ngOnInit` lifecycle hook when we've got the `constructor` available?

<div class="toc" markdown="1">
<span class="gamma">Table of contents</span>
{:.no_toc}
* TOC
{:toc}
</div>

### What's the difference?

The `constructor` method on an ES6 class (or TypeScript in this case) is a feature of a class itself, rather than an Angular feature. It's out of Angular's control when the `constructor` is invoked, which means that it's not a suitable hook to let you know when Angular has finished initialising the component.

Demonstrating the `constructor` alone:

```js
import { Component } from '@angular/core';

@Component({})
class ExampleComponent {
  // this is called by the JavaScript engine
  // rather than Angular
  constructor() {
    console.log('Constructor initialised');
  }
}

// internally calls the constructor
new ExampleComponent();
```

Note that the main piece here is that the JavaScript engine calls the `constructor`, not Angular directly. Which is why the `ngOnInit` (and `$onInit` in AngularJS) lifecycle hook was created.

By adding this lifecycle hook, Angular can fire a method once it has finished setting the component up, and as the naming suggests, the hook is part of the component _lifecycle_:

```js
import { Component, OnInit } from '@angular/core';

@Component({})
class ExampleComponent implements OnInit {
  constructor() {}
  
  // called on demand by Angular
  ngOnInit() {
    console.log('ngOnInit fired');
  }
}

const instance = new ExampleComponent();

// Angular calls this when necessary
instance.ngOnInit();
```

### Constructor usage

Bearing this in mind, there is a suitable scenario for using the `constructor`. This is when we want to utilise dependency injection - essentially for "wiring up" dependencies into the component.

As the `constructor` is initialised by the JavaScript engine, and TypeScript allows us to tell Angular what dependencies we require to be mapped against a specific property:

```js
import { Component, ElementRef } from '@angular/core';
import { Router } from '@angular/router';

@Component({})
class ExampleComponent {
  constructor(
    private router: Router,
    private el: ElementRef
  ) {}
}
```

> You can read more on Angular's [dependency injection here](/angular-dependency-injection).

This essentially will bind `Router` to `this.router`, making it accessible as part of the component class.

### ngOnInit

`ngOnInit` is purely there to give us a signal that Angular has finished initialising the component.

This phase includes the first pass at Change Detection against the properties that we may bind to the component itself - such as using an `@Input()` [decorator](/angular-decorators).
 
Due to this, the `@Input()` properties are available inside `ngOnInit`, however are `undefined` inside the `constructor`, by design.

```js
import { Component, ElementRef, Input, OnInit } from '@angular/core';
import { Router } from '@angular/router';

@Component({})
class ExampleComponent implements OnInit {
  @Input()
  person: Person;

  constructor(
    private router: Router,
    private el: ElementRef
  ) {
    // undefined
    console.log(this.person);
  }
  
  ngOnInit() {
    this.el.nativeElement.style.display = 'none';
    // { name: 'Todd Motto', location: 'England, UK' }
    console.log(this.person);
  }
}
```

The `ngOnInit` lifecycle hook is a _guarantee_ that your bindings are readily available.