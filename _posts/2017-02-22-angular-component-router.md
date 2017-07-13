---
layout: post
permalink: /angular-component-router
title: "Getting started with Angular&#39;s Router"
path: 2017-02-22-angular-component-router.md
tag: angular
---

Angular's router acts as the main hub of any application, it loads the relevant components relevant to the route requested, as well as dealing with fetching the relevant data for that particular route. This allows us to control different routes, data and the components that render the data (as well as many additional features).
 
So let's explore some of the main building blocks for the Angular router, following the table of contents below.

<div class="toc" markdown="1">
<span class="gamma">Table of contents</span>
{:.no_toc}
* TOC
{:toc}
</div>

### Installing the router

First thing's first, we need to install the router. You can do this by running either of the following:

```bash
yarn add @angular/router
# OR
npm i --save @angular/router
```

The latter choice may give you more ping pong / table football time whilst npm is installing.

This will download the router into your `node_modules` folder, ready for you to use and configure it.

#### Base href

The final thing we need to do before we can get going with the router is add the `base` tag to our `index.html` file.

The router needs this to define where the root of our application is. When we go to to `http://example.com/page1` for example, if we didn't define the base of our application, the router wouldn't know if our application was hosted at `http://example.com` or `http://example.com/page1`.

It's super easy to define this, open up your `index.html` and add the `base` tag into the header:

```html
<!doctype html>
<html>
  <head>
    <base href="/">
    <title>Application</title>
    <link href="css/app.css" rel="stylesheet">
  </head>
  <body>
    <app-root></app-root>
    <script src="app.js"></script>
  </body>
</html>
```

This tells Angular to use the root of the domain (`/`) as the starting point for all routes.

### Using the router

To use the router, we'll need to import the router's `NgModule` and include it in our main application module. This import is called `RouterModule`, where we can add it to our main application module like so:

```js
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { RouterModule } from '@angular/router';

import { AppComponent } from './app.component';

@NgModule({
  imports: [
    BrowserModule,
    RouterModule
  ],
  bootstrap: [
    AppComponent
  ],
  declarations: [
    AppComponent
  ]
})
export class AppModule {}
```

Our router setup isn't that useful at this point as it has no information about our components or routes. To pass a config through to the router, there are actually two static methods on `RouterModule` that we can use to finish the "setup" of the router module - then we can start building it out.

#### RouterModule.forRoot

The first of the static methods is `RouterModule.forRoot`, which we use when defining the root config for our application in our main module.

This gives our main module access to all the router directives (more on those coming up), as well as defining the main config. Let's have a look how we use the `forRoot` method to invoke the module:

```js
// ...
import { Routes, RouterModule } from '@angular/router';

export const ROUTES: Routes = [];

@NgModule({
  imports: [
    BrowserModule,
    RouterModule.forRoot(ROUTES)
  ],
  // ...
})
export class AppModule {}
```

We simply pass in an array of routes as our configuration, which will be an array of objects that describe the configuration. Typically it's a good pattern to pass a constant/variable into the `forRoot` method, rather than the array directly, to keep the module more visible and controlling the inputs to the module either above, or externally in a separate file - whereby `const ROUTES` is exported for use elsewhere.

We've also assigned the type of `Routes` exposed by the `@angular/router` package to our `ROUTES` array, allowing us to define routes whilst utilising the power of TypeScript to check our configuration as we type it to make sure we aren't doing anything that the router doesn't support. This also gives us the added bonus of intellisense and autocompletion - as well as making sure our configuration is setup correctly.
 
#### RouterModule.forChild

`RouterModule.forChild` is quite similar to the `forRoot` method, however is likely to be used with something like a feature module - rather than our main (root) module.

> Tip for remembering: "root" modules use `forRoot`, "child" modules use `forChild`

This feature is extremely powerful as we don't have to define all of our routes in one place (our main module). Instead, we can allow our modules to configure routes for themselves  - and they will be imported into our main application module where necessary.

Let's take a look at how we'd use this:

```js
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { Routes, RouterModule } from '@angular/router';

export const ROUTES: Routes = [];

@NgModule({
  imports: [
    CommonModule,
    RouterModule.forChild(ROUTES)
  ],
  // ...
})
export class ChildModule {}
```

As you can see, the setup is identical - the only difference is the method name that is being invoked. So now we've understood the higher level setup, let's dive into some route creation.

### Configuring a route

All routes that we define are objects inside our `ROUTES` array. To begin, let's define a simple route for our pseudo-homepage, pointing to the `HomeComponent`:

```js
// ...
import { Routes, RouterModule } from '@angular/router';

import { HomeComponent } from './home/home.component';

export const ROUTES: Routes = [
  { path: '', component: HomeComponent }
];

@NgModule({
  imports: [
    BrowserModule,
    RouterModule.forRoot(ROUTES)
  ],
  // ...
})
export class AppModule {}
```

What we're doing here is defining `path` (which is the path that we want to match and route to) and finally the property `component` - which as you guessed it points to the component that we want to load when the path is matched.

> We use `path: ''` to match the empty path, i.e. https://yourdomain.com

Before our component is rendered, there is one more step we need to make to reach "Hello world!" status.

### Displaying routes

After configuring our routes, the next step is to tell Angular where to load the components using a directive named `router-outlet`. When the router has matched a route and found the component(s) to load, it will dynamically create our component and inject it as a sibling alongside the `router-outlet` element.

Inside our `AppComponent` (the component we bootstrapped in our module), we can insert this directive wherever we want:

```js
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <div class="app">
      <h3>Our app</h3>
      <router-outlet></router-outlet>
    </div>
  `
})
export class AppComponent {}
```

We've now established the main building blocks for setting up the router in our application.

We've got a main route configured, as well as a place to load the route. We can now go on a little further and look at some of the other options that the router gives us.

### Further configuration

The setup we've covered so far is just the beginning - let's take a look at a few other options and features.

#### Dynamic routes

Routes wouldn't be that useful if they were always static, for instance `path: ''` is a static route which loads our `HomeComponent`. The next basic building block of the Angular router is dynamic routes - where we can associate a component against a route and pull in different data based on the params.

For example, if we wanted to have user "profile" pages with their unique "username" in the route, we could define the path like this:

```js
// ...
import { HomeComponent } from './home/home.component';
import { ProfileComponent } from './profile/profile.component';

export const ROUTES: Routes = [
  { path: '', component: HomeComponent },
  { path: '/profile/:username', component: ProfileComponent }
];
```

The key ingredient here is the colon `:` in front of the URL segment, as it tells the router to be a route parameter rather than an actual part of the URL. 

> If we didn't use the colon, it would match the path `/profile/username` literally, and would remain a static route.

So, now that we've established the use for a dynamic route, where we'll be swapping out `/:username` with a real username, such as `/toddmotto`.

At this point, we can take this dynamic piece of information from the route parameter, and inside something like `ngOnInit` we can hook in the `ProfileComponent` to run further logic using the param.

To access information about the current route, we need to import something named `ActivatedRoute` from the router. This gives our components the power to know what the current route is, as well as any extra information about them:

```js
import { Component, OnInit } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'profile-page',
  template: `
    <div class="profile">
      <h3>{% raw %}{{ username }}{% endraw %}</h3>
    </div>
  `
})
export class SettingsComponent implements OnInit {
  username: string;
  constructor(private route: ActivatedRoute) {}
  ngOnInit() {
    this.route.params.subscribe((params) => this.username = params.username);
  }
}
```

Here we're subscribing to the `params` Observable on the `ActivatedRoute`. It's important that we subscribe here, because if we decided to transition to another profile page, the profile component wouldn't actually get destroyed - which aids DOM performance by not destroying and recreating each time we access the same dynamic route. By subscribing, we will get notified when the parameters change and can update the component to match the latest parameters.

We access the dynamic `username` property inside our `subscribe` block - this is the same name as what we defined in our route path. Here would be a great place to feed the param into a service to fetch the dataset for the particular username we're querying.

Next, we'll explore how to create what we call "child routes".

#### Child routes

Every route can actually support child routes inside of it. Envision that we have a settings page at `/settings`, and a few pages *inside* the settings page such as `/settings/profile` and `/settings/password`. 

We might want our `/settings` page to have it's own component, and then have the `/settings/profile` and `/settings/password` pages be rendered *inside* the settings page component. We can do this like so:

```js
// ...
import { SettingsComponent } from './settings/settings.component';
import { ProfileSettingsComponent } from './settings/profile/profile.component';
import { PasswordSettingsComponent } from './settings/password/password.component';

export const ROUTES: Routes = [
  { 
    path: 'settings', 
    component: SettingsComponent,
    children: [
      { path: 'profile', component: ProfileSettingsComponent },
      { path: 'password', component: PasswordSettingsComponent }
    ]
  }
];

@NgModule({
  imports: [
    BrowserModule,
    RouterModule.forRoot(ROUTES)
  ],
  // ...
})
export class AppModule {}
```

Here we've defined two child routes in our settings route. These will inherit the path from above, so the `password` path will actually match `settings/password`, and so on and so forth.
 
One last thing that we need to do is put a `router-outlet` inside of our `SettingsComponent` - as we mentioned above, we want the child routes rendered in our settings page. If we didn't, only the `SettingsComponent` would render on the page regardless of the URL we went to, as the children routes will not be able to find a `router-outlet` to render them.
 
```js
import { Component } from '@angular/core';

@Component({
  selector: 'settings-page',
  template: `
    <div class="settings">
      <settings-header></settings-header>
      <settings-sidebar></settings-sidebar>
      <router-outlet></router-outlet>
    </div>
  `
})
export class SettingsComponent {}
```

#### Component-less routes

Another great routing feature is component-less routes. Using a component-less route allows us to group routes together and have them all share configuration and outlet in the DOM.

For example, we could define the settings routes without the parent `SettingsComponent`:

```js
// ...
import { ProfileSettingsComponent } from './settings/profile/profile.component';
import { PasswordSettingsComponent } from './settings/password/password.component';

export const ROUTES: Routes = [
  {
    path: 'settings',
    children: [
      { path: 'profile', component: ProfileSettingsComponent },
      { path: 'password', component: PasswordSettingsComponent }
    ]
  }
];

@NgModule({
  imports: [
    BrowserModule,
    RouterModule.forRoot(ROUTES)
  ],
  // ...
})
export class AppModule {}
```

Now, `/settings/profile` and `/settings/password` would be rendered inside our main `AppComponent` `router-outlet`, whilst they are grouped together under the `settings` route.

This means that we refactored at a later date, changing the path to `/config` instead, we only have to update the route in a single place, rather than changing `settings` to `config` for every child route in our definition objects.

#### loadChildren

We can also tell the router to fetch child routes from another module. This links the two ideas we spoke about together - we can specify a route that has `children: []` defined in another module, as well as taking advantage of the component-less routes functionality by defining all these child routes to be under a specific path.

Let's make a `SettingsModule` that will hold all of our settings routes:

```js
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { Routes, RouterModule } from '@angular/router';

export const ROUTES: Routes = [
  {
    path: '',
    component: SettingsComponent,
    children: [
      { path: 'profile', component: ProfileSettingsComponent },
      { path: 'password', component: PasswordSettingsComponent }
    ]
  }
];

@NgModule({
  imports: [
    CommonModule,
    RouterModule.forChild(ROUTES)
  ],
  // ...
})
export class SettingsModule {}
```

Notice how we're using `forChild` here as we're defining routes inside a child module rather than our main module.

One other major difference is we're defining the `path` for our main settings route as an empty path. This is because we're going to be loading these routes in as children of a route with the path `settings` already. 

If we put the route path as `/settings`, it would match `/settings/settings` which isn't our intention. By specifying an empty path, it will still match `/settings`, which is what we want.

So where does the `/settings` come from? In our `AppModule` config, we simply change the route to use a property called `loadChildren`:

```js
export const ROUTES: Routes = [
  {
    path: 'settings',
    loadChildren: './settings/settings.module#SettingsModule'
  }
];

@NgModule({
  imports: [
    BrowserModule,
    RouterModule.forRoot(ROUTES)
  ],
  // ...
})
export class AppModule {}
```

Notice how we don't import the `SettingsModule` into our `AppModule` config. This is because we're telling the router to go off and fetch the `SettingsModule` and include it in our application for us.

This is where lazy loading comes in - Angular will actually only load our children settings routes when the user attempts to go to `/settings/**`, reducing the payload that is initially served up to the user.

We're passing in a string as the value of `loadChildren`. This is the relative path to the module that we want to import (same as if we were to `import` it normally), and then a hash separator followed by the name of the exported module class.

### Router Directives

Alongside `router-outlet`, there are a few other directives that the router gives us. Let's take a look at how they piece together alongside our configuration options we've explored so far.
 
#### routerLink
 
If we were to create a standard hyperlink to `/settings/profile`, the browser doesn't actually know about the router, therefore it would treat it as you going to a normal link, and will reload the page - defeating the purpose of our single-page-app.

To allow us to link to our routes, we can use a directive called `routerLink`. This works the same as `href` in this case, which compiles out the links for us:

```html
<nav>
  <a routerLink="/">Home</a>
  <a routerLink="/settings/password">Change password</a>
  <a routerLink="/settings/profile">Profile Settings</a>
</nav>
```

When we click on any link the page won't be reloaded. Instead, our route will be reflected in the URL bar, followed by a subsequent view update to match the value of the clicked `routerLink`.

> We can also change `routerLink` to accept an array - enabling us to pass particular information into routes.

If we wanted to link to a dynamic route (such as the profile page inside perhaps an `ngFor`) and we had a `username` variable, we can wrap `routerLink` in square brackets (`[]`) and pass an array.

This array constructs segments of the URL that we want to navigate to. As we want to navigate to `/profile/:username`, we pass through `'/profile'` and then the username that we have in the variable:

```html
<a [routerLink]="['/profile', username]">
  Go to {% raw %}{{ username }}{% endraw %}'s profile.
</a>
```

#### routerLinkActive

It's useful for us to indicate to the user which route is currently active, we typically do this by adding a class to the link that's active.

To add this functionality to our `routerLink`, we can use the directive `routerLinkActive`. We just simply pass through the name of the class that we want to add when that route is active, and it'll add it for us:

```html
<nav>
  <a routerLink="/settings" routerLinkActive="active">Home</a>
  <a routerLink="/settings/password" routerLinkActive="active">Change password</a>
  <a routerLink="/settings/profile" routerLinkActive="active">Profile Settings</a>
</nav>
```

A link above will have the class `active` when the user has successfully transitioned to the relevant route.

### Router API

We can achieve the same as `routerLink` via the imperative API that the router also provides. We might want to redirect the user in our code, rather than after they click a link, which offers a little more flexibility.

To do this, we simply inject the `Router` into a component class:

```js
import { Component } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-root',
  template: `
    <div class="app">
      <h3>Our app</h3>
      <router-outlet></router-outlet>
    </div>
  `
})
export class AppComponent {
  constructor(private router: Router) {}
}
```

This then exposes the API method `navigate`. This takes the same value as when we use an array with `routerLink`, and will navigate the user to it as soon as it's called. So by learning the `routerLink` syntax, you've also learned the other when using `this.router`:

```js
import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-root',
  template: `
    <div class="app">
      <h3>Our app</h3>
      <router-outlet></router-outlet>
    </div>
  `
})
export class AppComponent implements OnInit {
  constructor(private router: Router) {}
  ngOnInit() {
    setTimeout(() => {
      this.router.navigate(['/settings']);
    }, 5000);
  }
}
```

You guessed it - this will redirect the user to the `/settings` page after 5 seconds. This is extremely useful - for example, we might want to redirect a user to the login page when they aren't logged in.

Another example demonstrating how to pass data, via a pseudo event:

```js
import { Component, OnInit } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-root',
  template: `
    <div class="app">
      <h3>Users</h3>
      <div *ngFor="let user of users">
        <user-component 
          [user]="user"
          (select)="handleSelect($event)">
        </user-component>
      </div>
      <router-outlet></router-outlet>
    </div>
  `
})
export class AppComponent implements OnInit {
  users: Username[] = [
    { name: 'toddmotto', id: 0 },
    { name: 'travisbarker', id: 1 },
    { name: 'tomdelonge', id: 2 }
  ];
  constructor(private router: Router) {}
  handleSelect(event) {
    this.router.navigate(['/profile', event.name]);
  }
}
```

The router is extremely powerful, both using directives or the imperative APIs, and hopefully this guide has given you a boost into getting started with the Angular router.

For more, check out the official [Angular docs](https://angular.io/docs/ts/latest/guide/router.html) on the router.
