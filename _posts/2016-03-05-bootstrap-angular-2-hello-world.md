---
layout: post
permalink: /bootstrap-angular-2-hello-world
title: Bootstrapping your first Angular 2+ app
path: 2016-03-05-bootstrap-angular-2-hello-world.md
tags:
- Angular 2
---

In this series of four Angular (v2+) posts, we'll explore how to bootstrap an Angular app, create a component, pass data into a component and pass new data out of a component using events.

### Series

1. Bootstrapping your first Angular app
2. [Creating your first Angular component](/creating-your-first-angular-2-component)
3. [Passing data into Angular components with @Input](/passing-data-angular-2-components-input)
4. [Component events with EventEmitter and @Output in Angular](/component-events-event-emitter-output-angular-2)

### Introduction

[Angular quickstart](https://angular.io/docs/ts/latest/quickstart.html) guide is a great place to get started with the next version of Angular, however there are some crucial aspects of the tutorial that can be elaborated on those new to Angular.

Let's walk through the bare essentials in a sensible order to get started and actually teach you what's happening with all the boilerplate setup we get, as well as how to create your first Angular component and bootstrap your app.

### Tooling options

A quick brief on the tooling options available when setting up your first Angular project.

#### Webpack

Webpack has become the defacto standard when building Angular applications, you can check an introduction to Webpack post [here](https://angular.io/docs/ts/latest/guide/webpack.html) on the Angular documentation.

#### System.js

System was heavily used in the beginning whilst Angular was being built and during the release candidate stages before official release, however the team have switched to advocating Webpack instead due to it being much more powerful.

#### Angular CLI

The Angular CLI was built to help scaffold new projects, create components and help with deployment. Underneath it utilises Webpack for you, it's a great starting place for beginners to alleviate new tooling associated around Angular.

### Root Component

To bootstrap an Angular app, we need a root component. We'll be covering how to create our own component in the next guide. Let's take a typical root component that you'll have likely seen when bootstrapping Angular. Angular needs this root component to bootstrap the application from:

```js
// app.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div class="app">
      Hello world!
    </div>
  `
})
export class AppComponent {}
```

### Root module

Each root component lives within a module, and these are defined using `@NgModule`. Here's the typical `@NgModule` for an Angular application, which will need to import our root component:

```js
// app.module.ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  bootstrap: [
    AppComponent
  ],
  imports: [
    BrowserModule
  ]
})
export class AppModule {}
```

You can read more on [NgModule here](https://angular.io/docs/ts/latest/guide/ngmodule.html). Here's a quick explanation on the three `@NgModule` properties used above:

* `declarations`: Registers particular components within this module
* `bootstrap`: Tells the module which component to bootstrap
* `imports`: Imports other modules into this module

### Bootstrapping

Now we've got a root component and root module, we need to learn how to bootstrap that particular module.

#### Module export/import

You may have noticed above that when using `@NgModule` we use the `export` keyword on the module `AppModule`. This allows us to import it elsewhere and tell Angular to bootstrap that particular module.

Typically you'd bootstrap an Angular app by importing that module into a new file:

```js
// main.ts
import { AppModule } from './app.module';
```

#### Browser bootstrapping

As Angular can be bootstrapped in multiple environments, such as [server-side](https://universal.angular.io), we need to import a module specific to the environment we're bootstrapping in. For browser bootstrapping, we need to import something called the `platform-browser-dynamic` module:

```js
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { AppModule } from './app.module';
```

You can also see we've got `platformBrowserDynamic` as the sole import from this module, this is actually a bootstrapping function that allows us to instantiate our app.

#### Bootstrapping the module

The next step is telling the above imported method what module you'd like to bootstrap. In our case, we've created `AppModule`, which registers `AppComponent`. We also have `AppModule` imported in the file above, which we can simply pass into a `bootstrapModule` method:

```js
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';
import { AppModule } from './app.module';

platformBrowserDynamic().bootstrapModule(AppModule);
```

And that's it! These are the key concepts of bootstrapping an Angular application.

### Plunker

Everything we've done here is readily available in a Plunker for you to have a look through, when using something like Plunker, you'll have to use System.js as we cannot use Webpack (as it's an offline development tool):

<iframe src="https://embed.plnkr.co/KQF2M8mA0L0trMrWgeLT?deferRun" frameborder="0" border="0" cellspacing="0" cellpadding="0" width="100%" height="250"></iframe>

### Next steps

Now we've learned how the basic bootstrapping process works, let's move on and learn how to [create an Angular component](/creating-your-first-angular-2-component).
