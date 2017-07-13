---
layout: post
permalink: /angular-dependency-injection
title: "Mastering Angular dependency injection with @Inject, @Injectable, tokens and providers"
path: 2017-01-24-angular-dependency-injection.md
tag: angular
---

Providers in Angular are key to how we develop our applications, and injecting dependencies can be done in various ways. In this post, we're going to debunk some terminology behind the `@Inject()` and `@Injectable()` decorators and explore the use cases for them. Then, we'll dive into understanding tokens, providers and take a look behind the scenes at how Angular actually fetches and creates our dependencies, as well as some Ahead-of-Time source code explanations.

<div class="toc" markdown="1">
<span class="gamma">Table of contents</span>
{:.no_toc}
* TOC
{:toc}
</div>

### Injecting providers

With most things Angular, there is a lot of magic happening when it comes to dependency injection (DI). With Angular 1.x we had a pretty simple approach using string tokens to fetch particular dependencies - I'm sure you know this:

```js
function SomeController($scope) {
  // use $scope
}
SomeController.$inject = ['$scope'];
```

You can check out my [old post](https://toddmotto.com/angular-js-dependency-injection-annotation-process/) on the DI annotation process for more on that if you like.

This was a great approach - but it came with some limitations. Typically we'd create various modules when building our applications, as well as importing external modules, such as feature modules or libraries (such as `ui-router`). Different modules couldn't have controllers/services/etc with the same names, which would then cause conflicts during the compile phase (as dependencies with the same names would clash, thus overriding each other).

Fortunately for us, Angular's new dependency injection has been completely remastered and rewritten, and it comes with much more power and flexibility.

#### A new dependency injection system

When injecting a service (a provider) into your components/services, we specify what provider we need via a _type definition_ in the constructor. For example:

```js
import { Component } from '@angular/core';
import { Http } from '@angular/http';

@Component({
  selector: 'example-component',
  template: '<div>I am a component</div>'
})
class ExampleComponent {
  constructor(private http: Http) {
    // use `this.http` which is the Http provider
  }
}
```

The type definition here is `Http` (note the capital H), and Angular automagically assigns this to `http`.

At this point, it's pretty magical how it works. Type definitions are specific to TypeScript, so our compiled JavaScript code should theoretically not know anything about what our `http` parameter is when it comes to running it in the browser.

Inside our `tsconfig.json` files we'll likely have `emitDecoratorMetadata` set to `true`. This emits metadata about the type of the parameter into a decorator in our compiled JavaScript output.

Let's take a look at what our code actually gets compiled into (I've kept the ES6 imports in for clarity):

```js
import { Component } from '@angular/core';
import { Http } from '@angular/http';

var ExampleComponent = (function () {
  function ExampleComponent(http) {
    this.http = http;
  }
  return ExampleComponent;
}());
ExampleComponent = __decorate([
  Component({
    selector: 'example-component',
    template: '<div>I am a component</div>'
  }),
  __metadata('design:paramtypes', [Http])
], ExampleComponent);
```

From here, we can see the compiled code knows about `http` being equal to the `Http` service provided by `@angular/http` - it's added as a decorator for our class here:

```js
__metadata('design:paramtypes', [Http])
```

So essentially, the `@Component` decorator is transformed into plain ES5, and some additional `metadata` is supplied through the `__decorate` assignment. Which in turn tells Angular to lookup the `Http` token and supply it as a first parameter to the Component's `constructor` - assigning it to `this.http`:

```js
function ExampleComponent(http) {
  this.http = http;
}
```

This looks a little familiar to our old from `$inject`, however the _class_ is being used as a token instead of a _string_. Power, and no naming conflicts.

> You might have heard of the concept of a "token" (or even `OpaqueToken`). This is how Angular stores and retrieves our providers. A token is a key that is used to reference a provider (our `Http` import is a provider). Unlike conventional keys however, these keys can be anything - such as objects, classes, strings, etc.

#### @Inject()

So where does `@Inject` come into play? We could alternatively write our component like this:

```js
import { Component, Inject } from '@angular/core';
import { Http } from '@angular/http';

@Component({
  selector: 'example-component',
  template: '<div>I am a component</div>'
})
class ExampleComponent {
  constructor(@Inject(Http) private http) {
    // use `this.http` which is the Http provider
  }
}
```

At this point, `@Inject` is a manual way of specifying this lookup token, followed by the lowercase `http` argument to tell Angular what to assign it against.

This could (and will) get very messy when a component or service requires a lot of dependencies. As Angular supports resolving dependencies from the emitted metadata, there's no need to use `@Inject` most of the time.

The only time we'd need to use `@Inject` is alongside something like an [OpaqueToken](https://angular.io/docs/ts/latest/api/core/index/OpaqueToken-class.html) - which creates a unique blank token to be used as a dependency injection provider.

The reason we use `@Inject` is because we cannot use an `OpaqueToken` as the _type_ of a parameter, for instance this will not work:

```js
const myToken = new OpaqueToken('myValue');

@Component(...)
class ExampleComponent {
  constructor(private token: myToken) {}
}
```

Here, `myToken` is not a Type, it's a value - which means TypeScript cannot compile it. However, when we introduce `@Inject` alongside an `OpaqueToken`, things will work out nicely:

```js
const myToken = new OpaqueToken('myValue');

@Component(...)
class ExampleComponent {
  constructor(@Inject(myToken) private token) {
    // use the provider for `token`
  }
}
```

We won't dive into `OpaqueToken` any further here, but this gives you an example of using `@Inject` for manually specifying tokens to be injected, as well as showing that the token can be anything. This means, we're not restricted to what TypeScript classifies as a "type".

#### @Injectable()

It's a common misbelief that this is a required decorator on any class that we plan on injecting into a component/service in our apps. This _may_ change however, as there is a [current issue](https://github.com/angular/angular/issues/13820) to make `@Injectable()` mandatory (however this is pretty fresh and may not land for a while, or ever).

When using Angular decorators, the decorated class stores metadata about itself in a format that Angular can read - this includes the metadata about what dependencies it needs to fetch and inject.

If no Angular decorator has been used on a class there is no way for Angular to read what dependencies it requires. This is why we need to use `@Injectable()`. 

If our service injects providers we must add `@Injectable()`, which providers no extra functionality, to tell Angular to store that metadata it needs.

Therefore, if our service looks like this:

```js
export class UserService {
  isAuthenticated(): boolean {
    return true;
  }
}
```

We don't need to decorate it to be able to inject it into a component for example, because it doesn't inject any providers itself.
 
However, if our service looks like this and contains a dependency (Http):

```js
import { Http } from '@angular/http';

export class UserService {
  constructor(private http: Http) {}
  isAuthenticated(): Observable<boolean> {
    return this.http.get('/api/user').map((res) => res.json());
  }
}
```

This would break as the `Http` provider metadata would not be stored for Angular to compose it correctly. 

We can simply add `@Injectable()` to solve this:

```js
import { Injectable } from '@angular/core';
import { Http } from '@angular/http';

@Injectable()
export class UserService {
  constructor(private http: Http) {}
  isAuthenticated(): Observable<boolean> {
    return this.http.get('/api/user').map((res) => res.json());
  }
}
```

At this point, Angular is aware of the `Http` token and can supply it to `http`.

### Tokens and Dependency Injection

Now that we know *how* Angular knows what to inject, we can learn how it resolves our dependencies and instantiates them.

#### Registering a provider

Let's take a look at how we'd register a typical service inside an `NgModule`.

```js
import { NgModule } from '@angular/core';

import { AuthService } from './auth.service';

@NgModule({
  providers: [
    AuthService
  ]
})
class ExampleModule {}
```

The above is shorthand for this:

```js
import { NgModule } from '@angular/core';

import { AuthService } from './auth.service';

@NgModule({
  providers: [
    {
      provide: AuthService,
      useClass: AuthService
    }
  ]
})
class ExampleModule {}
```

The `provide` property in the object is the token for the provider that we're registering. This means that Angular can look up what is stored under the token for `AuthService` using the `useClass` value.

This provides many benefits. The first, we can now have two providers with the exact same `class` name and Angular will not have any issues in resolving the correct service. Secondly, we can also override an existing provider with a different provider whilst keeping the _token_ the same.

#### Overriding providers

Here's what our `AuthService` might look like:

```js
import { Injectable } from '@angular/core';
import { Http } from '@angular/http';

@Injectable()
export class AuthService {

  constructor(private http: Http) {}
  
  authenticateUser(username: string, password: string): Observable<boolean> {
    // returns true or false
    return this.http.post('/api/auth', { username, password });
  }
  
  getUsername(): Observable<string> {
    return this.http.post('/api/user');
  }

}
```

Imagine we use this service heavily throughout our application. For instance, our (streamlined) login form uses it to log the user in:

```js
import { Component } from '@angular/core';
import { AuthService } from './auth.service';

@Component({
  selector: 'auth-login',
  template: `
    <button (click)="login()">
      Login
    </button>
  `
})
export class LoginComponent {

  constructor(private authService: AuthService) {}
  
  login() {
    this.authService
      .authenticateUser('toddmotto', 'straightouttacompton')
      .subscribe((status: boolean) => {
        // do something if the user has logged in
      });
  }

}
```

Then we can bind our user information using the service to display the username:

```js
@Component({
  selector: 'user-info',
  template: `
    <div>
      You are {% raw %}{{ username }}{% endraw %}!
    </div>
  `
})
class UserInfoComponent implements OnInit {

  username: string;
  
  constructor(private authService: AuthService) {}
  
  ngOnInit() {
    this.authService
      .getUsername()
      .subscribe((username: string) => this.username = username);
  }

}
```

We then hook this all up into a module, such as `AuthModule`:

```js
import { NgModule } from '@angular/core';

import { AuthService } from './auth.service';

import { LoginComponent } from './login.component';
import { UserInfoComponent } from './user-info.component';

@NgModule({
  declarations: [
    LoginComponent,
    UserInfoComponent
  ],
  providers: [
    AuthService
  ]
})
export class AuthModule {}
```

There could also be various components that use the same `AuthService`. But let's assume we now have a new requirement, and need to change our authentication method over to a library that lets us use Facebook to log users in.

We could go through every single component and change all the imports to point to this new provider, however we can instead utilise the power of tokens and override our `AuthService` to use the `FacebookAuthService`:

```js
import { NgModule } from '@angular/core';

// totally made up
import { FacebookAuthService } from '@facebook/angular';

import { AuthService } from './auth.service';

import { LoginComponent } from './login.component';
import { UserInfoComponent } from './user-info.component';

@NgModule({
  declarations: [
    LoginComponent,
    UserInfoComponent
  ],
  providers: [
    {
      provide: AuthService,
      useClass: FacebookAuthService
    }
  ]
})
export class AuthModule {}
```

So you can see here we're using the long-hand form of registering the provider, and essentially swapping out the `useClass` property with a different value. This way, we can use `AuthService` everywhere in our application - without making further changes.

This is because Angular uses `AuthService` as the token to search for our provider. As we've replaced it with a new class `FacebookAuthService`, all of our components will use that instead.

### Understanding Injectors

If you've made it this far, you should hopefully have an understanding of tokens and the dependency injection system of Angular, however in this next chapter - we're actually going to break down the compiled [AoT](https://angular.io/docs/ts/latest/cookbook/aot-compiler.html) code from Angular to walk through it a little further.

#### Pre-compiled code

Before we dive into the compiled code, let's look at the pre-compiled version of the code. Precompiled? That's the code you and I write before Ahead-of-Time compilation, so essentially everything you write is pre-compiled and Angular can either compile it in the browser for you via [JiT](https://angular.io/docs/ts/latest/cookbook/aot-compiler.html#!#aot-jit) or for a more performant approach we can offline compile (AoT).

So, let's assume you've built out your application - but we're just going to walk through a single piece of `NgModule` code:

```js
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { Routes, RouterModule } from '@angular/router';
import { HttpModule } from '@angular/http';

import { AppComponent } from './app.component';

export const ROUTER_CONFIG: Routes = [
  { path: '', loadChildren: './home/home.module#HomeModule' },
  { path: 'about', loadChildren: './about/about.module#AboutModule' },
  { path: 'contact', loadChildren: './contact/contact.module#ContactModule' }
];

@NgModule({
  imports: [
    BrowserModule,
    HttpModule,
    RouterModule.forRoot(ROUTER_CONFIG),
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

This should look pretty familiar - we have a root component and some routes that we're routing to different modules with. So what's the _real_ code look like, as we know Angular is _compiled_?

Angular will produce VM (virtual machine) friendly code, to make it as performant as possible, which is fantastic. What we'll do is dive into that compiled code and explain it a little further.

#### AppModuleInjector

Angular will generate an injector for each of our modules, so in our case it will take `AppModule` (our decorated class) and create an injector named `AppModuleInjector`.

Let's look at the generated code for our `AppModuleInjector` and break it down:

```js
import { NgModuleInjector } from '@angular/core/src/linker/ng_module_factory';
import { CommonModule } from '@angular/common/src/common_module';
import { ApplicationModule, _localeFactory } from '@angular/core/src/application_module';
import { BrowserModule, errorHandler } from '@angular/platform-browser/src/browser';
import { RouterModule, ROUTER_FORROOT_GUARD } from '@angular/router/src/router_module';
import { NgLocaleLocalization, NgLocalization } from '@angular/common/src/localization';
import { ApplicationInitStatus, APP_INITIALIZER } from '@angular/core/src/application_init';
import { Testability, TestabilityRegistry } from '@angular/core/src/testability/testability';
import { HttpModule } from '@angular/http/src/http_module';
import { ApplicationRef, ApplicationRef_ } from '@angular/core/src/application_ref';
import { BrowserModule } from '@angular/platform-browser/src/browser';
import { Injector } from '@angular/core/src/di/injector';
import { LOCALE_ID } from '@angular/core/src/i18n/tokens';
import { RouterModule, provideForRootGuard } from '@angular/router/src/router_module';
import { Router } from '@angular/router/src/router';
import { NgZone } from '@angular/core/src/zone/ng_zone';
import { Console } from '@angular/core/src/console';
import { ROUTES } from '@angular/router/src/router_config_loader';
import { ErrorHandler } from '@angular/core/src/error_handler';

import { AppModule } from './app.module';
import { AppComponentNgFactory } from './app.component.ngfactory';

class AppModuleInjector extends NgModuleInjector<AppModule> {
  _CommonModule_0: CommonModule;
  _ApplicationModule_1: ApplicationModule;
  _BrowserModule_2: BrowserModule;
  _ROUTER_FORROOT_GUARD_3: any;
  _RouterModule_4: RouterModule;
  _HttpModule_5: HttpModule;
  _AppModule_6: AppModule;
  _ErrorHandler_7: any;
  _ApplicationInitStatus_8: ApplicationInitStatus;
  _Testability_9: Testability;
  _ApplicationRef__10: ApplicationRef_;
  __ApplicationRef_11: any;
  __ROUTES_12: any[];
  
  constructor(parent: Injector) {
    super(parent, [AppComponentNgFactory], [AppComponentNgFactory]);  
  }

  get _ApplicationRef_11(): any {
    if (this.__ApplicationRef_11 == null) { 
      this.__ApplicationRef_11 = this._ApplicationRef__10; 
    }
    return this.__ApplicationRef_11;
  }

  get _ROUTES_12(): any[] {
    if (this.__ROUTES_12 == null) { 
      this.__ROUTES_12 = [[
        {
          path: '', loadChildren: './home/home.module#HomeModule'
        },
        {
          path: 'about', loadChildren: './about/about.module#AboutModule'
        },
        {
          path: 'contact', loadChildren: './contact/contact.module#ContactModule'
        }
      ]]; 
    }
    return this.__ROUTES_12;
  }
  
  createInternal(): AppModule {
    this._CommonModule_0 = new CommonModule();
    this._ApplicationModule_1 = new ApplicationModule();
    this._BrowserModule_2 = new BrowserModule(this.parent.get(BrowserModule, (null as any)));
    this._ROUTER_FORROOT_GUARD_3 = provideForRootGuard(this.parent.get(Router, (null as any)));
    this._RouterModule_4 = new RouterModule(this._ROUTER_FORROOT_GUARD_3);
    this._HttpModule_5 = new HttpModule();
    this._AppModule_6 = new AppModule();
    this._ErrorHandler_7 = errorHandler();
    this._ApplicationInitStatus_8 = new ApplicationInitStatus(this.parent.get(APP_INITIALIZER, (null as any)));
    this._Testability_9 = new Testability(this.parent.get(NgZone));

    this._ApplicationRef__10 = new ApplicationRef_(
      this.parent.get(NgZone), 
      this.parent.get(Console), 
      this, 
      this._ErrorHandler_7, 
      this,
      this._ApplicationInitStatus_8,
      this.parent.get(TestabilityRegistry, (null as any)),
      this._Testability_9
    );
    return this._AppModule_6;
  }
  
  getInternal(token: any, notFoundResult: any): any {
    if (token === CommonModule) { return this._CommonModule_0; }
    if (token === ApplicationModule) { return this._ApplicationModule_1; }
    if (token === BrowserModule) { return this._BrowserModule_2; }
    if (token === ROUTER_FORROOT_GUARD) { return this._ROUTER_FORROOT_GUARD_3; }
    if (token === RouterModule) { return this._RouterModule_4; }
    if (token === HttpModule) { return this._HttpModule_5; }
    if (token === AppModule) { return this._AppModule_6; }
    if (token === ErrorHandler) { return this._ErrorHandler_7; }
    if (token === ApplicationInitStatus) { return this._ApplicationInitStatus_8; }
    if (token === Testability) { return this._Testability_9; }
    if (token === ApplicationRef_) { return this._ApplicationRef__10; }
    if (token === ApplicationRef) { return this._ApplicationRef_11; }
    if (token === ROUTES) { return this._ROUTES_12; }
    
    return notFoundResult;
  }

  destroyInternal(): void {
    this._ApplicationRef__10.ngOnDestroy();
  }
}
```

> This might look a bit insane (and the actual generated code is a lot more insane) but let's look at what is actually happening here.

I've changed all the imports to be named imports for readability as in the _actual_ generated code, each module is imported using a wildcard to avoid naming conflicts.

For instance, `HttpModule` would be imported as something like this:

```js
import * as import6 from '@angular/http/src/http_module';
```

Which then is referred to using `import6.HttpModule` instead of `HttpModule`.

There are three things that we need to take in from this generated code. The properties on the class, the module imports and how the dependency injection mechanism works.

#### AppModuleInjector properties

Properties are created on the `AppModuleInjector` for each provider/dependency:

```js
// ...
class AppModuleInjector extends NgModuleInjector<AppModule> {
  _CommonModule_0: CommonModule;
  _ApplicationModule_1: ApplicationModule;
  _BrowserModule_2: BrowserModule;
  // ...
}
```

This is a snippet from the above compiled output - so we're going to focus on the three properties defined on the class:

* CommonModule
* ApplicationModule
* BrowserModule

Our module only declares the `BrowserModule`, so where have the `CommonModule` and `ApplicationModule` come from? These are actually exported _by_ the `BrowserModule` for us, so we don't need to import them ourselves.

There is also a number appended on the end of every property in the module. Much like using wildcard imports, this is to avoid potential naming conflicts between providers.

We could import two modules that use a service with a shared name and without the incrementing numbers, they'd both be assigned to the same property, potentially causing errors further down the line.

#### Module imports

When compiled, Angular uses the direct path for each provider that it imports, so for instance when we write this code:

```js
import { CommonModule } from '@angular/common';
```

The AoT'd version will look a little like this:

```js
import * as import5 from '@angular/common/src/common_module';
```

This is so when the code is then compiled and bundled together, we can take advantage of [tree-shaking](https://angular.io/docs/ts/latest/cookbook/aot-compiler.html#!#tree-shaking) and only include the parts of each module we actually use. 

#### Dependency Injection

Each module deals with its own dependency injection, going to the parent module if it doesn't have a dependency until it is either found or not found (then we get an error).

It's important to note that all dependencies use a token to uniquely identify them, both when they're registered and when they are looked up.

There's two different ways that our dependencies are initiated, either in `createInternal` or as a getter on a property.

> For all of our imported modules and their exported modules, they are created within `createInternal`. This is invoked as soon as the module is instantiated.

For example, we are using `BrowserModule` and `HttpModule`, and they are created here:

```js
class AppModuleInjector extends NgModuleInjector<AppModule> {
  _CommonModule_0: CommonModule;
  _ApplicationModule_1: ApplicationModule;
  _BrowserModule_2: BrowserModule;
  _HttpModule_5: HttpModule;
  _AppModule_6: AppModule;
  
  createInternal(): AppModule {
    this._CommonModule_0 = new CommonModule();
    this._ApplicationModule_1 = new ApplicationModule();
    this._BrowserModule_2 = new BrowserModule(this.parent.get(BrowserModule, (null as any)));
    this._HttpModule_5 = new HttpModule();
    this._AppModule_6 = new AppModule();
    // ...
    return this._AppModule_6;
  }
}
```

You can see that `BrowserModule`'s two exports - `CommonModule` and `ApplicationModule` are created, as well our other imported modules. Our actual module is created too (`AppModule`) so it can be consumed by other modules.
 
For all the other providers, they are created as and when they're needed via a getter within the class. This is to avoid creating instances of providers when they aren't needed, also increasing the initial rendering performance.

> Whenever you hear of an injector in Angular, it's referring to the generated (compiled) code from our modules.

When Angular looks up a dependency (such as ones we inject via a `constructor`), it looks in the module injector and traverses up the parent modules if it fails to find it. Should it not exist, you'll be thrown an error.

When we use a type definition in our `constructor`, Angular uses those types (which are classes) as the token for finding our dependencies. That token is then passed into `getInternal` and the instance of the dependency is returned if it exists, source code extract again:

```js
class AppModuleInjector extends NgModuleInjector<AppModule> {

  // new BrowserModule(this.parent.get(BrowserModule, (null as any)));
  _BrowserModule_2: BrowserModule;

  // new HttpModule()
  _HttpModule_5: HttpModule;

  // new AppModule()
  _AppModule_6: AppModule;
  
  getInternal(token: any, notFoundResult: any): any {
    if (token === BrowserModule) { return this._BrowserModule_2; }
    if (token === HttpModule) { return this._HttpModule_5; }
    if (token === AppModule) { return this._AppModule_6; }
    
    return notFoundResult;
  }
}
```

So inside the `getInternal` method, you can see Angular is checking for our tokens using simple `if` statements, and will return the relevant property for the provider - if found.

Otherwise, we'll bail out the `getInternal` method with a returned `notFoundResult`. Whilst Angular is traversing up our modules to find the desired dependency, this `notFoundResult` will be `null` - until it either finds the dependency, or reaches the root module and still cannot find it, you'll be thrown an error.

### Closing thoughts

Hopefully this post has given you some in-depth insights into `@Inject`, `@Injectable`, tokens and providers and how Angular generates VM friendly code when AoT compiling.
