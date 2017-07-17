---
layout: post
permalink: /lazy-loading-angular-code-splitting-webpack
title: "Lazy loading: code splitting NgModules with Webpack"
path: 2017-07-17-lazy-loading-angular-code-splitting-webpack.md
tag: angular
---

Let's talk about code splitting in Angular, lazy-loading and a sprinkle of Webpack. Code splitting allows us to essentially break our codebase down into smaller chunks and serve those chunks on demand, which we call "lazy loading". So, let's learn how to do it and some of the concepts/terminology behind it.

<div class="toc" markdown="1">
<span class="gamma">Table of contents</span>
{:.no_toc}
* TOC
{:toc}
</div>

> Want the code? Go [straight to GitHub](https://github.com/toddmotto/angular-lazy-load-demo) or view the [live demo](https://toddmotto.com/angular-lazy-load-demo/)

<img src="/img/posts/lazy.gif">

The above `.gif` demonstrates lazy loading, you can see `0-chunk.js` and `1-chunk.js` are both fetched over the network when navigating to these routes. The above recording is also AoT compiled.

### Terminology

For some further clarity let's cover some of the terminology.

#### Code splitting

Code splitting is the process of, putting it very obviously, _splitting_ our code. But what, how and where do we split? We'll figure this piece out as we progress through the article, but code splitting allows us to essentially take our full application bundle, and chop it up into different pieces. This is all code splitting is, and Webpack allows us to do it super easily with a loader for Angular. In a nut shell, your application becomes lots of small applications, which we typically call "chunks". These chunks can be loaded on demand.

#### Lazy loading

This is where "on demand" comes into play. Lazy loading is the _process_ in taking already "code split" chunks of our application, and simply loading them on demand. With Angular, the router is what allows us to lazy load. We call it "lazy" because it's not "eagerly" loading - which would mean loading assets upfront. Lazy loading helps _boost performance_ - as we're only downloading a fraction of our app's bundle instead of the _entire_ bundle. Instead, we can code split per `@NgModule` with Angular, and we can serve them lazily via the router. Only when a specific route is matched, Angular's router will load the code split module.

### Webpack setup

Setting up the Webpack side of things is fairly trivial, you can check the [full config](https://github.com/toddmotto/angular-lazy-load-demo/blob/master/webpack.config.js) to see how everything hangs together, but essentially we need just a few key pieces.

#### Choosing a router loader

You may wish to use the [angular-router-loader](https://www.npmjs.com/package/angular-router-loader) or [ng-router-loader](https://www.npmjs.com/package/ng-router-loader) to accomplish your lazy loading mission - I'm going to roll with the former, `angular-router-loader` as it's pretty simple to get working and both cover the base set of features we'd need for lazy loading.

Here's how I've added it to my Webpack config:

```js
{
  test: /\.ts$/,
  loaders: [
    'awesome-typescript-loader',
    'angular-router-loader',
    'angular2-template-loader'
  ]
}
```

Here I'm including the `angular-router-loader` in the loaders array for TypeScript files, this will kick things off and let us use the awesome loader to lazy load! The next step is the `output` property on our Webpack config:

```js
output: {
  filename: '[name].js',
  chunkFilename: '[name]-chunk.js',
  publicPath: '/build/',
  path: path.resolve(__dirname, 'build')
}
```

This is where we can specify our "chunk" names, which are drive dynamically and typically end up looking like:

```bash
0-chunk.js
1-chunk.js
2-chunk.js
3-chunk.js
```

> Check the [full config](https://github.com/toddmotto/angular-lazy-load-demo/blob/master/webpack.config.js) again if necessary for tying it together in perhaps your own Webpack configuration.

### Lazy @NgModules

To illustrate the setup as shown in the live demo and gif, we have three feature modules that are identical, apart from renaming of the module and components to suit.

#### Feature modules

Feature modules, aka child modules, are the modules that we can lazy load using the router. Here are the three _child_ module names:

```bash
DashboardModule
SettingsModule
ReportsModule
```

And the parent, app module:

```bash
AppModule
```

The `AppModule` has the responsibility at this point to somehow "import" those other modules. There are a few ways we can do this, asynchronously and synchronously. 


#### Async module lazy loading

We look to the router to power our lazy loading, and all we need for it is the magical `loadChildren` property on our routing definitions.

Here's the `ReportsModule`:

```js
// reports.module.ts
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

// containers
import { ReportsComponent } from './reports.component';

// routes
export const ROUTES: Routes = [
  { path: '', component: ReportsComponent }
];

@NgModule({
  imports: [
    RouterModule.forChild(ROUTES)
  ],
  declarations: [
    ReportsComponent
  ]
})
export class ReportsModule {}
```

Note how we're using an empty `path`:

```js
// reports.module.ts
export const ROUTES: Routes = [
  { path: '', component: ReportsComponent }
];
```

This module can then be used together with `loadChildren` and `path` in a _parent_ module, letting `AppModule` dictate the URL. This creates a flexible module structure where your feature modules are "unaware" of their absolute path, they become relative paths based on the `AppModule` paths.

This means that inside `app.module`, we can do this:

```js
// app.module.ts
export const ROUTES: Routes = [
  { path: 'reports', loadChildren: '../reports/reports.module#ReportsModule' }
];
```

This says to Angular "when we hit `/reports`, please load this module". Note how the routing definition inside the `ReportsModule` is an empty path, this is how it's achievable. Similarly, our other routing definitions are also empty:

```js
// reports.module.ts
export const ROUTES: Routes = [
  { path: '', component: ReportsComponent }
];

// settings.module.ts
export const ROUTES: Routes = [
  { path: '', component: SettingsComponent }
];

// dashboard.module.ts
export const ROUTES: Routes = [
  { path: '', component: DashboardComponent }
];
```

The full picture of the `AppModule` routing definitions:

```js
export const ROUTES: Routes = [
  { path: '', pathMatch: 'full', redirectTo: 'dashboard' },
  { path: 'dashboard', loadChildren: '../dashboard/dashboard.module#DashboardModule' },
  { path: 'settings', loadChildren: '../settings/settings.module#SettingsModule' },
  { path: 'reports', loadChildren: '../reports/reports.module#ReportsModule' }
];
```

This means at any moment in time, we can "move" an entire module under a new route path and everything will work as intended to, which is great!

> Notice in the recording below how `*-chunk.js` files are being loaded in as we navigate to these particular routes

<img src="/img/posts/lazy.gif">

We call this "lazy loading" when we make the call to a chunk _asynchronously_. When using `loadChildren` and the string value to point to a module, these will typically load async, unless using the loader you [specify sync loading](https://github.com/brandonroberts/angular-router-loader#synchronous-loading).

#### Sync module loading

If, like in my application, your base path redirects to another route - like this:

```js
{ path: '', pathMatch: 'full', redirectTo: 'dashboard' },
```

You have a potential area to specify one module to be loaded _synchronously_. This means it'll be bundled into your `app.js` (in my case, this may change depending on the depth in feature modules you are lazy loading). As I'm redirecting straight away to `DashboardModule`, is there any benefit to me chunking it? Yes and no.

*Yes:* if the user goes to `/settings` first (page refresh), we don't want to load even more code, so there's again an initial payload savings here.

*No:* this module may be used most frequently, so it's probably best to load it upfront eagerly.

> Both yes/no depend on your scenario, however.

Here's how we can sync load our `DashboardModule` using an `import` and arrow function:

```js
import { DashboardModule } from '../dashboard/dashboard.module';

export const ROUTES: Routes = [
  { path: '', pathMatch: 'full', redirectTo: 'dashboard' },
  { path: 'dashboard', loadChildren: () => DashboardModule },
  { path: 'settings', loadChildren: '../settings/settings.module#SettingsModule' },
  { path: 'reports', loadChildren: '../reports/reports.module#ReportsModule' }
];
```

I prefer this way as it's more implicit for the intentions. At this point, `DashboardModule` would be bundled in with `AppModule` and served up in `app.js`. You can [try it yourself](https://github.com/toddmotto/angular-lazy-load-demo) by running the project locally and changing things.

The `angular-router-loader` project has a nice feature also which is worth mentioning for a custom syntax which dictates which modules are loaded sync by appending `?sync=true` to our string:

```js
loadChildren: '../dashboard/dashboard.module#DashboardModule?sync=true'
```

This has the same effects as using the arrow function approach.

### Performance

With a simple application demo like mine, you're not really going to notice a performance increase, however with a bigger application with a nice sized codebase, you'll benefit greatly from code splitting and lazy loading!

#### Lazy loading modules

Let's imagine we have the following:

```bash
vendor.js [200kb] // angular, rxjs, etc.
app.js [400kb] // our main app bundle
```

Now let's assume we code split:

```bash
vendor.js [200kb] // angular, rxjs, etc.
app.js [250kb] // our main app bundle
0-chunk.js [50kb]
1-chunk.js [50kb]
2-chunk.js [50kb]
```

Again, on a much bigger scale, the performance savings would be huge for things such as PWAs (Progressive Web Apps), initial network requests and severely decrease initial payloads.

#### Preloading lazy modules

There's another option we have, the [PreloadAllModules](https://angular.io/api/router/PreloadAllModules) feature that allows Angular, once bootstrapped, to go and fetch all the remaining module chunks from your server. This could again be part of your performance story and you choose to eagerly download your chunked modules. This would lead to faster navigation between different modules, and they download asynchronously once you add it to your root module's routing. An example of doing this:

```js
import { RouterModule, Routes, PreloadAllModules } from @angular/router;

export const ROUTES: Routes = [
  { path: '', pathMatch: 'full', redirectTo: 'dashboard' },
  { path: 'dashboard', loadChildren: '../dashboard/dashboard.module#DashboardModule' },
  { path: 'settings', loadChildren: '../settings/settings.module#SettingsModule' },
  { path: 'reports', loadChildren: '../reports/reports.module#ReportsModule' }
];

@NgModule({
  // ...
  imports: [
    RouteModule.forRoot(ROUTES, { preloadingStrategy: PreloadAllModules })
  ],
  // ...
})
export class AppModule {}
```

In my application demo, Angular would bootstrap then go ahead and load the rest of the chunks by using this approach.

> View the full source code [on GitHub](https://github.com/toddmotto/angular-lazy-load-demo) or check out the [live demo](https://toddmotto.com/angular-lazy-load-demo/)!

I highly recommend trying these out and seeing the different scenarios available to you so you can paint your own performance picture.