---
layout: post
permalink: /building-tesla-range-calculator-angular-2-reactive-forms
title: Building Tesla&#39;s battery range calculator with Angular 2 reactive forms
path: 2016-12-13-building-tesla-range-calculator-angular-2-reactive-forms.md
---

In this epic tutorial, we're going to build some advanced Angular (v2+) components that rebuild [Tesla's battery range calculator](https://tesla.com/en_GB/models#battery-options) and then compile it to [AoT](https://angular.io/docs/ts/latest/cookbook/aot-compiler.html) and deploy on GitHub pages. We'll be using the [reactive forms](/angular-2-forms-reactive) API as well and building custom form controls and use some stateful and stateless component practices, as well as change detection strategies.

This is the final project `gif` of what we're about to build:

<img src="img/posts/tesla/final.gif" alt="" style="width: 100%; max-width: 100%;">

> Check out [the live version](https://toddmotto.com/angular-tesla-range-calculator/) before we get started

We'll be building the above app step by step, so you can follow along with the tutorial.

> Straight to the source code? [Go here](https://github.com/toddmotto/angular-tesla-range-calculator)!

### Table of contents

- <a href="#setup-and-angular-cli">Setup and Angular CLI</a>
  - <a href="#new-github-repo">New GitHub repo</a>
  - <a href="#new-cli-project">CLI installation</a>
  - <a href="#serving-the-project">Serving the project</a>
- <a href="#project-imagesassets">Project images/assets</a>
- <a href="#root-and-sub-modules">Root and sub-modules</a>
  - <a href="#root-ngmodule">Root @NgModule</a>
  - <a href="#tesla-sub-module">Tesla Sub-module</a>
- <a href="#injectable-data-service">Injectable data service</a>
- <a href="#container-and-presentational-components">Container and presentational components</a>
  - <a href="#formgroup-setup">FormGroup setup</a>
  - <a href="#wiring-into-the-app-component">Wiring into the app component</a>
- <a href="#car-component">Car component</a>
  - <a href="#rendering-the-car">Rendering the car</a>
- <a href="#stats-component">Stats component</a>
  - <a href="#stats-and-datastructure-models">Stats and datastructure models</a>
  - <a href="#private-stats-calculation">Private stats calculation</a>
- <a href="#re-usable-counter-component">Re-usable counter component</a>
  - <a href="#counter-and-controlvalueaccessor">Counter and ControlValueAccessor</a>
  - <a href="#displaying-the-counters">Displaying the counters</a>
- <a href="#aircon-and-heating-controls">Aircon and Heating controls</a>
  - <a href="#conditional-airconheating-limits">Conditional aircon/heating limits</a>
- <a href="#wheel-size-component">Wheel size component</a>
- <a href="#formgroup-valuechange-subscription">FormGroup valueChange subscription</a>
- <a href="#deploying-with-ahead-of-time-compilation">Deploying with Ahead-of-Time compilation</a>
  - <a href="#deploying-to-github-pages">Deploying to GitHub pages</a>
  - <a href="#source-code">Source code</a>

### Setup and Angular CLI

Head over to the [Angular CLI](https://cli.angular.io/) website and familiarise yourself with it. We'll be running our local server and deploying with it.

> Versions: this tutorial uses CLI version `1.0.0-beta.22-1` and Angular `2.2.3`

#### New GitHub repo

First step, you'll need a GitHub account if you actually want to deploy this to a GitHub pages instance. Go to GitHub and create your own repo called `angular-tesla-range-calculator`.

> Tip: It's a good idea to name your `repo` the same as the `cli` project you're about to create

#### CLI installation

Let's assume you've just created a repo called `angular-tesla-range-calculator` and are available to commit code to it from your machine. If you've not got the Angular CLI, you'll want to run:

{% highlight bash %}
npm install -g angular-cli
{% endhighlight %}

Then (note the same name as the repo):

{% highlight bash %}
cd <somewhere-you-want-your-project> # e.g. /Users/toddmotto/git
ng new angular-tesla-range-calculator
{% endhighlight %}

It'll take a few moments to download the required dependencies for the project. Then we can add the project to the remote:

{% highlight bash %}
cd angular-tesla-range-calculator
git remote add origin https://github.com/<YOUR_USERNAME>/angular-tesla-range-calculator.git
git push -u origin master
{% endhighlight %}

Now if you check back on GitHub, the project should be there. Voila. Now we'll get started.

#### Serving the project

Now we're ready to roll, so let's boot up our application:

{% highlight bash %}
ng serve # or npm start
{% endhighlight %}

Then you'll be able to hit `localhost:4200` and see the app running.

### Project images/assets

We'll make this easy and just drop in all our images before we really get started.

* Download `assets.zip` [here](/static/assets.zip) (all image)
* Download the `favicon.ico` [here](/static/favicon.ico) (favicon 'cos why not)

Once you're done, unzip the `assets.zip` folder and replace the downloaded favicon with the one in the project, and locate:

{% highlight bash %}
angular-tesla-range-calculator/src/assets/
{% endhighlight %}

And then just drop all the images in there (and replace the `favicon.ico` in the root).

### Root and sub-modules

First thing we'll do is create our sub-module, a feature specific module for handling our Tesla app.

> Directories: Everything we're going to do with be inside `/src/app/` so any folder references will refer to in there

#### Root @NgModule

First up, change your `app.module.ts` to this (remove comments if you like):

{% highlight javascript %}
/*
 * app.module.ts
 */
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';

// our feature module
import { TeslaBatteryModule } from './tesla-battery/tesla-battery.module';

// our app component
import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    // include our TeslaBatteryModule
    TeslaBatteryModule
  ],
  providers: [],
  // bootstrap the AppComponent
  bootstrap: [AppComponent]
})
export class AppModule {}
{% endhighlight %}

This will error if we save the file as our module doesn't exist just yet, so let's create it.

#### Tesla Sub-module

From the above code example, you can see we're importing our `tesla-battery` module, so next up we want to create a new folder:

{% highlight bash %}
**/src/app/tesla-battery/
{% endhighlight %}

Inside here, create two files:

{% highlight bash %}
tesla-battery.module.ts
tesla-battery.service.ts
{% endhighlight %}

> Any time you feel like you're missing a step or unsure if you're putting something in the right place, check the [full source code](https://github.com/toddmotto/angular-tesla-range-calculator) as a reference.

Inside your `tesla-battery.module.ts` file, paste this in:

{% highlight javascript %}
/*
 * tesla-battery.module.ts
 */
import { CommonModule } from '@angular/common';
import { NgModule } from '@angular/core';
import { ReactiveFormsModule } from '@angular/forms';

// services
import { BatteryService } from './tesla-battery.service';

@NgModule({
  declarations: [],
  imports: [
    CommonModule,
    ReactiveFormsModule
  ],
  providers: [
    // add the service to our sub-module
    BatteryService
  ],
  exports: []
})
export class TeslaBatteryModule {}
{% endhighlight %}

We'll be populating this with new components as we go.

#### Injectable data service

The data from Tesla's website is actually hard-coded `*.json` files that live on the server, I ripped them apart and created a new datastructure that made it easier to access properties once our values change.

> IMPORTANT: The data file is hard-coded, and absolutely huge - so [go here](https://github.com/toddmotto/angular-tesla-range-calculator/blob/master/src/app/tesla-battery/tesla-battery.service.ts).

Once you've copied the data from the above link, our service will look a little like this:

{% highlight javascript %}
/*
 * tesla-battery.service.ts
 */
import { Injectable } from '@angular/core';

@Injectable()
export class BatteryService {
  constructor() {}
  getModelData(): Object {
    return {...};
  }
}
{% endhighlight %}

The `Injectable` is a decorator from Angular that allows us to inject our service into component, thus inferring they're "smart" components. Once you've copy and pasted the enormous amount of data into your service, move onto the next step. We'll come back to the datastructure later.

### Container and presentational components

This is a new idea I'm currently working with in my Angular apps, separating "container" and "presentational" components, otherwise known as [stateful and stateless components](/stateful-stateless-components) which I've previously written about, I'd urge you to check that out if you're up for further reading.

The idea is that stateful components, which we'll refer to as "container" components in the rest of this tutorial, will live inside our module's `containers` directory. Any stateless components, i.e. presentational components, will just live inside `components`.

So, go ahead and create these two directories:

{% highlight bash %}
**/src/app/tesla-battery/containers
**/src/app/tesla-battery/components
{% endhighlight %}

A container component is in charge of sourcing data and delegating it down into smaller, more focused components. Let's start with our container component (we only need one in this tutorial), so go ahead and create our first component directory `tesla-battery`:

{% highlight bash %}
**/src/app/tesla-battery/containers/tesla-battery/
{% endhighlight %}

Inside `**/containers/tesla-battery/` you should create two files:

{% highlight bash %}
tesla-battery.component.ts
tesla-battery.component.scss
{% endhighlight %}

Why no `tesla-battery.component.html`? At the moment I enjoy using `template` instead of a template file, it helps reduce context switching and keeps my thinking contained. With the CLI, you're welcome to use `templateUrl` should you wish to.

Next up, add these styles to your `tesla-battery.component.scss` file:

{% highlight css %}
.tesla-battery {
  width: 1050px;
  margin: 0 auto;
  h1 {
    font-family: 'RobotoNormal';
    font-weight: 100;
    font-size: 38px;
    text-align: center;
    letter-spacing: 3px;
  }
  &__notice {
    margin: 20px 0;
    font-size: 15px;
    color: #666;
    line-height: 20px;
  }
}
.tesla-climate {
  float: left;
  width: 420px;
  padding: 0 40px;
  margin: 0 40px 0 0;
  border-left: 1px solid #ccc;
  border-right: 1px solid #ccc;
}
.tesla-controls {
  display: block;
  width: 100%;
}
{% endhighlight %}

#### FormGroup setup

We're going to be using a `FormGroup` in our component to define the data structure for the view.

> Read more [here on reactive forms](/angular-2-forms-reactive)

Inside your `tesla-battery.component.ts` file:

{% highlight javascript %}
/*
 * tesla-battery.component.ts
 */
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup } from '@angular/forms';

@Component({
  selector: 'tesla-battery',
  template: `
    <form class="tesla-battery" [formGroup]="tesla">
      <h1>{% raw %}{{ title }}{% endraw %}</h1>
      <div class="tesla-battery__notice">
        <p>
          The actual amount of range that you experience will vary based
          on your particular use conditions. See how particular use conditions
          may affect your range in our simulation model.
        </p>
        <p>
          Vehicle range may vary depending on the vehicle configuration,
          battery age and condition, driving style and operating, environmental
          and climate conditions.
        </p>
      </div>
    </form>
  `,
  styleUrls: ['./tesla-battery.component.scss']
})
export class TeslaBatteryComponent implements OnInit {

  title: string = 'Range Per Charge';
  tesla: FormGroup;

  constructor(public fb: FormBuilder) {}

  ngOnInit() {
    this.tesla = this.fb.group({
      config: this.fb.group({
        speed: 55,
        temperature: 20,
        climate: true,
        wheels: 19
      })
    });
  }

}
{% endhighlight %}

This is pretty much good for now. Go back to `tesla-battery.module.ts` and let's import the new component:

{% highlight javascript %}
// containers
import { TeslaBatteryComponent } from './containers/tesla-battery/tesla-battery.component';
{% endhighlight %}

Our `@NgModule()` needs to also look like this:

{% highlight javascript %}
@NgModule({
  declarations: [
    // registering our container component
    TeslaBatteryComponent
  ],
  imports: [
    CommonModule,
    ReactiveFormsModule
  ],
  providers: [
    // add the service to our sub-module
    BatteryService
  ],
  exports: [
    // exporting so our root module can access
    TeslaBatteryComponent
  ]
})
export class TeslaBatteryModule {}
{% endhighlight %}

We're using `exports` to export that particular component from our module, so we can use it in other modules that our `TeslaBatteryModule` is imported into.

#### Wiring into the app component

Jump across to `app.component.ts` and replace the entire file with this:

{% highlight javascript %}
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  template: `
    <header class="header">
      <img [src]="logo">
    </header>
    <div class="wrapper">
      <tesla-battery></tesla-battery>
    </div>
  `,
  styleUrls: ['./app.component.scss']
})
export class AppComponent {
  logo: string = 'assets/logo.svg';
}
{% endhighlight %}

> Cannot find module "./app.component.scss" - if you see this, rename your `app.component.css` to `app.component.scss` so we can use Sass

Now open up `app.component.scss` and add this:

{% highlight css %}
:host {
  display: block;
}
.header {
  padding: 25px 0;
  text-align: center;
  background: #222;
  img {
    width: 100px;
    height: 13px;
  }
}
.wrapper {
  margin: 100px 0 150px;
}
{% endhighlight %}

You should hopefully see some text in the app now as well as the logo header, but we need to add some more styling to our global `styles.css` file. Locate that file in the _root of your project_ and replace the contents with this:

{% highlight css %}
@font-face {
  font-family: 'RobotoNormal';
  src: url('./assets/fonts/Roboto-Regular-webfont.eot');
  src: url('./assets/fonts/Roboto-Regular-webfont.eot?#iefix') format('embedded-opentype'),
       url('./assets/fonts/Roboto-Regular-webfont.woff') format('woff'),
       url('./assets/fonts/Roboto-Regular-webfont.ttf') format('truetype'),
       url('./assets/fonts/Roboto-Regular-webfont.svg#RobotoRegular') format('svg');
  font-weight: normal;
  font-style: normal;
}

*, *:before, *:after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font: 300 14px/1.4 'Helvetica Neue', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
}

.cf:before,
.cf:after {
    content: '';
    display: table;
}
.cf:after {
    clear: both;
}
.cf {
  *zoom: 1;
}
{% endhighlight %}

Upon saving this file, things will look a lot nicer. Right - back to the components!

### Car component

Go ahead and create a `/tesla-car/` directory inside a new _`/components`_ directory (where we'll keep our "stateless" components):

{% highlight bash %}
**/src/app/tesla-battery/components/tesla-car/
{% endhighlight %}

Then inside of there, create these two components:

{% highlight bash %}
tesla-car.component.ts
tesla-car.component.scss
{% endhighlight %}

This is what will produce our car image and make the wheels spin:

{% highlight javascript %}
/*
 * tesla-car.component.ts
 */
import { Component, Input, ChangeDetectionStrategy } from '@angular/core';

@Component({
  selector: 'tesla-car',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="tesla-car">
      <div class="tesla-wheels">
        <div class="tesla-wheel tesla-wheel--front tesla-wheel--{% raw %}{{ wheelsize }}{% endraw %}"></div>
        <div class="tesla-wheel tesla-wheel--rear tesla-wheel--{% raw %}{{ wheelsize }}{% endraw %}"></div>
      </div>
    </div>
  `,
  styleUrls: ['./tesla-car.component.scss']
})
export class TeslaCarComponent {
  @Input() wheelsize: number;
  constructor() {}
}
{% endhighlight %}

> We're also telling Angular not to bother with change detection in this component by using `ChangeDetectionStrategy.OnPush`, which Angular will tell the component to treat props coming down through the `@Input()` as immutable.

Now some styles for the `tesla-car.component.scss` file:

{% highlight javascript %}
.tesla-car {
  width: 100%;
  min-height: 350px;
  background: #fff url(assets/tesla.jpg) no-repeat top center;
  background-size: contain;
}
.tesla-wheels {
  height: 247px;
  width: 555px;
  position: relative;
  margin: 0 auto;
}
.tesla-wheel {
  height: 80px;
  width: 80px;
  bottom: 0;
  position: absolute;
  background-repeat: no-repeat;
  background-position: 0 0;
  background-size: cover;
  &--front {
    left: 53px;
  }
  &--rear {
    right: 72px;
  }
  &--19 {
    background-image: url(assets/wheel-19.png);
    -webkit-animation: infinite-spinning 250ms steps(6) infinite;
    -moz-animation: infinite-spinning 250ms steps(6) infinite;
    -o-animation: infinite-spinning 250ms steps(6) infinite;
    animation: infinite-spinning 250ms steps(6) infinite;
  }
  &--21 {
    background-image: url(assets/wheel-21.png);
    -webkit-animation: infinite-spinning 480ms steps(12) infinite;
    -moz-animation: infinite-spinning 480ms steps(12) infinite;
    -o-animation: infinite-spinning 480ms steps(12) infinite;
    animation: infinite-spinning 480ms steps(12) infinite;
  }
}

@keyframes infinite-spinning {
  from {
    -webkit-transform: rotate(0deg);
    -moz-transform: rotate(0deg);
    -ms-transform: rotate(0deg);
    -o-transform: rotate(0deg);
    transform: rotate(0deg);
  }
  to {
    -webkit-transform: rotate(360deg);
    -moz-transform: rotate(360deg);
    -ms-transform: rotate(360deg);
    -o-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}

@-webkit-keyframes infinite-spinning {
  from {
    -webkit-transform: rotate(0deg);
    -moz-transform: rotate(0deg);
    -ms-transform: rotate(0deg);
    -o-transform: rotate(0deg);
    transform: rotate(0deg);
  }
  to {
    -webkit-transform: rotate(360deg);
    -moz-transform: rotate(360deg);
    -ms-transform: rotate(360deg);
    -o-transform: rotate(360deg);
    transform: rotate(360deg);
  }
}
{% endhighlight %}

This gives us our animations and the component base for the car, which is displayed as background images. The `@Input()` value will be the wheel size that we need to pass in, but first we need to add these components to our module again (back to `tesla-battery.module.ts`):

{% highlight javascript %}
...

/* put this code below the // containers piece */
// components
import { TeslaCarComponent } from './components/tesla-car/tesla-car.component';

...

@NgModule({
  declarations: [
    TeslaBatteryComponent,
    // new addition
    TeslaCarComponent
  ],
  ...
})
...
{% endhighlight %}

We don't need to `export` this component as we're only using it locally to this module.

#### Rendering the car

Jump back into `tesla-battery.component.ts` and add the component with the `[wheelsize]` binding:

{% highlight javascript %}
...
@Component({
  selector: 'tesla-battery',
  template: `
    <form class="tesla-battery" [formGroup]="tesla">
      <h1>{% raw %}{{ title }}{% endraw %}</h1>
      <tesla-car [wheelsize]="tesla.get('config.wheels').value"></tesla-car>
      ...
      ...
    </form>
  `
})
...
{% endhighlight %}

Because we're using the `FormBuilder`, we can access the `config.wheels` property (which sets the default wheelsize like Tesla's website does) through the `tesla.get()` method, which returns us the form control. So all we're doing here is accessing the `.value` property and delegating it into the `<tesla-car>` component through the `@Input()` binding we just setup.

Here's what you should be seeing:

<img src="img/posts/tesla/step-1.gif" alt="" style="width: 100%; max-width: 100%;">

At this point you _could_ go change the `wheels: 19` value in the `FormGroup` to `21` to see the wheel size change, however we'll be building that soon.

### Stats component

Now we're going to render the stats for each Tesla car model.

Go ahead and create a `/tesla-stats/` directory inside the `/components` directory just like our previous component:

{% highlight bash %}
**/src/app/tesla-battery/components/tesla-stats/
{% endhighlight %}

Then inside of there, create these two components:

{% highlight bash %}
tesla-stats.component.ts
tesla-stats.component.scss
{% endhighlight %}

Before we dive in, we need to define an `interface` for our "stats", save this as `stat.interface.ts` inside a new `/models/` directory in our `tesla-battery` root:

{% highlight javascript %}
// src/app/tesla-battery/models/stat.interface.ts
export interface Stat {
  model: string,
  miles: number
}
{% endhighlight %}

Each stat will contain the name of the Tesla car `model` as well as the `miles` associated with the model based on the specific calculations we implement (this will become apparent as we continue).

Now we'll define the stats _component_:

{% highlight javascript %}
/*
 * tesla-stats.component.ts
 */
import { Component, Input, ChangeDetectionStrategy } from '@angular/core';

import { Stat } from '../../models/stat.interface';

@Component({
  selector: 'tesla-stats',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="tesla-stats">
      <ul>
        <li *ngFor="let stat of stats">
          <div class="tesla-stats-icon tesla-stats-icon--{% raw %}{{ stat.model | lowercase }}{% endraw %}"></div>
          <p>{% raw %}{{ stat.miles }}{% endraw %}</p>
        </li>
      </ul>
    </div>
  `,
  styleUrls: ['./tesla-stats.component.scss']
})
export class TeslaStatsComponent {
  @Input() stats: Stat[];
}
{% endhighlight %}

This component is purely stateless as well, and takes a single `@Input()` of the `stats`. You can see we're expecting a `Stat[]`, which means an Array of `Stat` Objects that align with our interface.

All this component is doing is iterating with `*ngFor` over the stats that are passed in, and will then bind a specific class to the element based on the `stat.model`, which will allow us to swap out the background images to display the Tesla models.

Onto the CSS, drop this into `tesla-stats.component.scss`:

{% highlight css %}
.tesla-stats {
  margin: -70px 0 30px;
  ul {
    text-align: center;
    li {
      display: inline-block;
      width: 130px;
      position: relative;
      p {
        font-size: 40px;
        font-weight: normal;
        font-family: 'RobotoNormal';
        display: block;
        padding: 0 18px 0 0;
        position: relative;
        color: #008dff;
        text-align: right;
        &:after {
          font-size: 14px;
          font-weight: normal;
          font-family: 'RobotoNormal';
          content: 'MI';
          position: absolute;
          top: 8px;
          right: 0;
        }
      }
    }
  }
  &-icon {
    height: 20px;
    background-size: auto 13px;
    background-position: top right;
    background-repeat: no-repeat;
    &--60 {
      background-image: url(assets/models/60.svg);
    }
    &--60d {
      background-image: url(assets/models/60d.svg);
    }
    &--75 {
      background-image: url(assets/models/75.svg);
    }
    &--75d {
      background-image: url(assets/models/75d.svg);
    }
    &--90d {
      background-image: url(assets/models/90d.svg);
    }
    &--p100d {
      background-image: url(assets/models/p100d.svg);
    }
  }
}
{% endhighlight %}

You'll notice at the end that we have values such as `&--60` and `&--p100d` being extended from the `icon` class, where we appropriately swap out the SVG backgrounds. These are the car models we'll hook up and render momentarily.

Back to our `tesla-battery.module.ts`, we need to add:

{% highlight javascript %}
...
import { TeslaStatsComponent } from './components/tesla-stats/tesla-stats.component';

@NgModule({
  declarations: [
    TeslaBatteryComponent,
    TeslaCarComponent,
    // new addition
    TeslaStatsComponent
  ],
  ...
})
...
{% endhighlight %}

#### Stats and datastructure models

We've already implemented the huge amount of data for our `tesla-battery.service.ts`, which we did at the beginning of this tutorial. Now it's time to get the data and start rendering it.

Jump back into your `tesla-battery.component.ts` file and add the following imports, to grab our `Stat` interface and our `BatteryService`:

{% highlight javascript %}
import { Stat } from '../../models/stat.interface';
import { BatteryService } from '../../tesla-battery.service';
{% endhighlight %}

We've already dependency injected the `FormBuilder`, so now it's time to add our service, ensure the top of your `tesla-battery.component.ts` looks like this:

{% highlight javascript %}
// tesla-battery.component.ts
@Component({...})
export class TeslaBatteryComponent implements OnInit {

  title: string = 'Range Per Charge';
  models: any;
  stats: Stat[];
  tesla: FormGroup;

  private results: Array<String> = ['60', '60D', '75', '75D', '90D', 'P100D'];

  constructor(public fb: FormBuilder, private batteryService: BatteryService) {}
  ...
  ...
}
{% endhighlight %}

A few additions here, the `models` which I've just set to `any`, a `stats` property which will again be our array of `Stat` Objects. The `private results` is a list of the Tesla models that will then be passed down into the child component for rendering and switching out with the correct background image - but before they reach the child component they'll be processed against our data model to return the mileage estimates Tesla provide as well.

#### Private stats calculation

Drop this method inside your `tesla-battery.component.ts` file on the component class, it is our helper function to calculate the current stat that it needs to find in our monolithic Object model returned from our `BatteryService`:

{% highlight javascript %}
// tesla-battery.component.ts
private calculateStats(models, value): Stat[]  {
  return models.map(model => {
    const { speed, temperature, climate, wheels } = value;
    const miles = this.models[model][wheels][climate ? 'on' : 'off'].speed[speed][temperature];
    return {
      model,
      miles
    };
  });
}
{% endhighlight %}

Now into the `ngOnInit`, ensure yours looks like this:

{% highlight javascript %}
// tesla-battery.component.ts
ngOnInit() {

  this.models = this.batteryService.getModelData();

  this.tesla = this.fb.group({
    config: this.fb.group({
      speed: 55,
      temperature: 20,
      climate: true,
      wheels: 19
    })
  });

  this.stats = this.calculateStats(this.results, this.tesla.controls['config'].value);

}
{% endhighlight %}

You can note our `models` is now being bound to the synchronous response from our `batteryService` we injected, in a real world data-driven application your models may look different and be loaded via routing resolves or an RxJS subscription.

What we've just done is taken `private results`, and passed it into `calculateStats`, with the second argument being the default value of our `FormGroup`. This allows us to then run some calculations and render to our `stats`, fetching the correct units for each Tesla model.

This bit is complete, but just simply need to bind the `tesla-stats` component to our template now:

{% highlight javascript %}
...
@Component({
  selector: 'tesla-battery',
  template: `
    <form class="tesla-battery" [formGroup]="tesla">
      <h1>{% raw %}{{ title }}{% endraw %}</h1>
      <tesla-car [wheelsize]="tesla.get('config.wheels').value"></tesla-car>
      <tesla-stats [stats]="stats"></tesla-stats>
      ...
      ...
    </form>
  `
})
...
{% endhighlight %}

Here's what you should be seeing:

<img src="img/posts/tesla/step-2.gif" alt="" style="width: 100%; max-width: 100%;">

### Re-usable counter component

Tesla's _Speed_ and _Outside Temperature_ controls should be re-usable components, so we're going to create a generic counter component that accepts a `step`, `min` value, `max` value and some other metadata such as a `title` and `unit` (mph/degrees) to inject in.

Go ahead and create a `/tesla-counter/` directory inside the `/components` directory just like our previous component:

{% highlight bash %}
**/src/app/tesla-battery/components/tesla-counter/
{% endhighlight %}

Then inside of there, create these two components:

{% highlight bash %}
tesla-counter.component.ts
tesla-counter.component.scss
{% endhighlight %}

#### Counter and ControlValueAccessor

This bit is the complex bit, where we implement a `ControlValueAccessor` to read and write directly to a `FormControl`, which we will implement after. I've annotated this file (which you need to paste into `tesla-counter.component.ts`) so you can understand what's happening. Essentially it allows our component to directly communicate to the reactive `FormControl` we're binding to it:

{% highlight javascript %}
// importing forwardRef as an extra here
import { Component, Input, ChangeDetectionStrategy, forwardRef } from '@angular/core';
// importing necessary accessors
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

// NUMBER_CONTROL_ACCESSOR constant to allow us to use the "TeslaCounterComponent" as
// a custom provider to the component and enforce the ControlValueAccessor interface
const NUMBER_CONTROL_ACCESSOR = {
  provide: NG_VALUE_ACCESSOR,
  // forwardRef allows us to grab the TypeScript class
  // at a later (safer) point as classes aren't hoisted
  useExisting: forwardRef(() => TeslaCounterComponent),
  multi: true
};

@Component({
  selector: 'tesla-counter',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="tesla-counter">
      <p class="tesla-counter__title">{% raw %}{{ title }}{% endraw %}</p>
      <div class="tesla-counter__container cf">
        <div
          class="tesla-counter__item"
          (keydown)="onKeyUp($event)"
          (blur)="onBlur($event)"
          (focus)="onFocus($event)"
          tabindex="0">
          <p class="tesla-counter__number">
            {% raw %}{{ value }}{% endraw %}
            <span>{% raw %}{{ unit }}{% endraw %}</span>
          </p>
          <div class="tesla-counter__controls" tabindex="-1">
            <button tabindex="-1" (click)="increment()" [disabled]="value === max"></button>
            <button tabindex="-1" (click)="decrement()" [disabled]="value === min"></button>
          </div>
        </div>
      </div>
    </div>
  `,
  // set the custom accessor as a provider
  providers: [NUMBER_CONTROL_ACCESSOR],
  styleUrls: ['./tesla-counter.component.scss']
})
export class TeslaCounterComponent implements ControlValueAccessor {
  // step count, default of 1
  @Input() step: number = 1;
  // minimum number allowed before disabling buttons
  @Input() min: number;
  // maximum number allowed before disabling buttons
  @Input() max: number;

  // title to be passed to the control
  @Input() title: string = '';
  // unit to be used alongside the title (mph/degrees/anything)
  @Input() unit: string = '';

  value: number;
  focused: boolean;

  // internal functions to call when ControlValueAccessor
  // gets called
  private onTouch: Function;
  private onModelChange: Function;

  // our custom onChange method
  private onChange(value: number) {
    this.value = value;
    this.onModelChange(value);
  }

  // called by the reactive form control
  registerOnChange(fn: Function) {
    // assigns to our internal model change method
    this.onModelChange = fn;
  }

  // called by the reactive form control
  registerOnTouched(fn: Function) {
    // assigns our own "touched" method
    this.onTouch = fn;
  }

  // writes the value to the local component
  // that binds to the "value"
  writeValue(value: number) {
    this.value = value;
  }

  // increment function
  increment() {
    if (this.value < this.max) {
      this.onChange(this.value + this.step);
    }
    this.onTouch();
  }
  // decrement function
  decrement() {
    if (this.value > this.min) {
      this.onChange(this.value - this.step);
    }
    this.onTouch();
  }

  // our onBlur event, has effect on template
  private onBlur(event: FocusEvent) {
    this.focused = false;
    event.preventDefault();
    event.stopPropagation();
  }

  // our onKeyup event, will respond to user
  // ArrowDown and ArrowUp keys and call
  // the relevant functions we need
  private onKeyUp(event: KeyboardEvent) {
    let handlers = {
      ArrowDown: () => this.decrement(),
      ArrowUp: () => this.increment()
    };
    // events here stop the browser scrolling up
    // when using the keys, as well as preventing
    // event bubbling
    if (handlers[event.code]) {
      handlers[event.code]();
      event.preventDefault();
      event.stopPropagation();
    }
  }

  // when we focus on our counter control
  private onFocus(event: FocusEvent) {
    this.focused = true;
    event.preventDefault();
    event.stopPropagation();
  }

}
{% endhighlight %}

Once you're done here, time for the styles for `tesla-counter.component.scss`:

{% highlight css %}
.tesla-counter {
  float: left;
  width: 230px;
  &__title {
    letter-spacing: 2px;
    font-size: 16px;
  }
  &__container {
    margin: 10px 0 0;
    padding-right: 40px;
    input[type=number] {
      border: 0;
      clip: rect(0 0 0 0);
      height: 1px;
      margin: -1px;
      overflow: hidden;
      padding: 0;
      position: absolute;
      width: 1px;
    }
  }
  &__number {
    font-family: 'RobotoNormal';
    font-size: 25px;
    line-height: 25px;
    font-weight: 400;
    position: relative;
    span {
      position: absolute;
      top: 0;
      left: 35px;
      font-size: 15px;
      text-transform: uppercase;
    }
  }
  &__item {
    position: relative;
    width: 100%;
    height: 65px;
    border: 1px solid #ccc;
    display: inline-block;
    padding: 18px 0 0 30px;
    margin: 0 8px 0 0;
    background-color: #f7f7f7;
    background-position: 24.21053% 9px;
    background-repeat: no-repeat;
    background-size: 44px;
    &:focus {
      background-color: #f2f2f2;
      outline: none;
    }
  }
  &__controls {
    position: absolute;
    right: 10px;
    top: 7px;
    button {
      outline: 0;
      width: 30px;
      color: #008dff;
      cursor: pointer;
      display: block;
      padding: 11px 0;
      vertical-align: middle;
      border: 0;
      background-size: 60%;
      background-position: center;
      background-repeat: no-repeat;
      background-color: transparent;
      &[disabled] {
        opacity: 0.4;
        cursor: not-allowed;
      }
      &:first-child {
        border-bottom: 1px solid #fff;
        background-image: url(assets/counter/up.svg);
      }
      &:last-child {
        border-top: 1px solid #ccc;
        background-image: url(assets/counter/down.svg);
      }
    }
  }
}

{% endhighlight %}

That was a bigger more complex implementation, but once you view it in the browser you'll see the power behind it.

Back to our `tesla-battery.module.ts`, we need to add:

{% highlight javascript %}
...
import { TeslaCounterComponent } from './components/tesla-counter/tesla-counter.component';

@NgModule({
  declarations: [
    TeslaBatteryComponent,
    TeslaCarComponent,
    TeslaStatsComponent,
    // new addition
    TeslaCounterComponent
  ],
  ...
})
...
{% endhighlight %}

Now we have a generic counter component that we can pass our `FormGroup` values into.

#### Displaying the counters

Let's jump back into our `tesla-battery.component.ts` and add our custom form controls, as well as the `formGroupName`:

{% highlight javascript %}
...
@Component({
  selector: 'tesla-battery',
  template: `
    <form class="tesla-battery" [formGroup]="tesla">
      <h1>{% raw %}{{ title }}{% endraw %}</h1>
      <tesla-car [wheelsize]="tesla.get('config.wheels').value"></tesla-car>
      <tesla-stats [stats]="stats"></tesla-stats>
      <div class="tesla-controls cf" formGroupName="config">
        <tesla-counter
          [title]="'Speed'"
          [unit]="'mph'"
          [step]="5"
          [min]="45"
          [max]="70"
          formControlName="speed">
        </tesla-counter>
        <div class="tesla-climate cf">
          <tesla-counter
            [title]="'Outside Temperature'"
            [unit]="'°'"
            [step]="10"
            [min]="-10"
            [max]="40"
            formControlName="temperature">
          </tesla-counter>
        </div>
      </div>
      ...
      ...
    </form>
  `
})
...
{% endhighlight %}

Here we're using `formGroupName="config"` to target the `config` scope in our initial `FormBuilder` setup, then delegating the `speed` and `temperature` controls down to our custom `<tesla-counter>` components.

At this point, you should see this:

<img src="img/posts/tesla/step-3.gif" alt="" style="width: 100%; max-width: 100%;">

### Aircon and Heating controls

This is a fun one. We have to monitor the value of the `temperature` control, and once it hits `20` degrees, we switch "heating" to "aircon". When it's below `20` degrees we switch it back to heating. Let's do it!

Go ahead and create a `/tesla-climate/` directory inside the `/components` directory just like our previous component:

{% highlight bash %}
**/src/app/tesla-battery/components/tesla-climate/
{% endhighlight %}

Then inside of there, create these two components:

{% highlight bash %}
tesla-climate.component.ts
tesla-climate.component.scss
{% endhighlight %}

Once you're done, populate your `tesla-climate.component.ts` component with this, which should look a little familiar:

{% highlight javascript %}
import { Component, Input, ChangeDetectionStrategy, forwardRef } from '@angular/core';
import { FormControl, ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

const CHECKBOX_VALUE_ACCESSOR = {
  provide: NG_VALUE_ACCESSOR,
  useExisting: forwardRef(() => TeslaClimateComponent),
  multi: true
};

@Component({
  selector: 'tesla-climate',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="tesla-climate">
      <label
        class="tesla-climate__item"
        [class.tesla-heat]="!limit"
        [class.tesla-climate__item--active]="value"
        [class.tesla-climate__item--focused]="focused === value">
        <p>{% raw %}{{ (limit ? 'ac' : 'heat') }}{% endraw %} {% raw %}{{ value ? 'on' : 'off' }}{% endraw %}</p>
        <i class="tesla-climate__icon"></i>
      <input
        type="checkbox"
        name="climate"
        [checked]="value"
        (change)="onChange(value)"
        (blur)="onBlur($event)"
        (focus)="onFocus($event)">
    </label>
  </div>
  `,
  providers: [CHECKBOX_VALUE_ACCESSOR],
  styleUrls: ['./tesla-climate.component.scss']
})
export class TeslaClimateComponent implements ControlValueAccessor {

  @Input() limit: boolean;

  value: boolean;
  focused: boolean;

  private onTouch: Function;
  private onModelChange: Function;

  private onChange(value: boolean) {
    this.value = !value;
    this.onModelChange(this.value);
  }

  registerOnChange(fn: Function) {
    this.onModelChange = fn;
  }

  registerOnTouched(fn: Function) {
    this.onTouch = fn;
  }

  writeValue(value: boolean) {
    this.value = value;
  }

  private onBlur(value: boolean) {
    this.focused = false;
  }

  private onFocus(value: boolean) {
    this.focused = value;
    this.onTouch();
  }

}
{% endhighlight %}

We're pretty much doing the same thing as the previous component, however we're directly writing the `value` property to a `checkbox` as seen here:

{% highlight html %}
<input
  type="checkbox"
  name="climate"
  [checked]="value"
  (change)="onChange(value)"
  (blur)="onBlur($event)"
  (focus)="onFocus($event)">
{% endhighlight %}

So when `value === true`, the checkbox is ticked. Pretty simple, and we can monitor those changes with our custom form control, switch out some text and class names when the value changes.

Our `@Input() limit` is when the temperature reaches a specific limit (`20` degrees) we need to tell the component from the outside as we'll be monitoring changes, which we'll complete once we add the component to the `tesla-battery` template shortly.

Let's add some styles to `tesla-climate.component.scss`:

{% highlight css %}
.tesla-climate {
  float: left;
  &__item {
    cursor: pointer;
    display: block;
    width: 100px;
    height: 100px;
    border: 6px solid #f7f7f7;
    border-radius: 50%;
    box-shadow: 0px 1px 3px rgba(0, 0, 0, 0.3);
    color: #666;
    background: #fff;
    &--active {
      color: #fff;
      background: #33a0ff;
      background: -moz-linear-gradient(top,  #33a0ff 0%, #388bff 100%);
      background: -webkit-linear-gradient(top,  #33a0ff 0%,#388bff 100%);
      background: linear-gradient(to bottom,  #33a0ff 0%,#388bff 100%);
      &.tesla-heat {
        background: #d64800;
        background: -moz-linear-gradient(top,  #d64800 0%, #d20200 100%);
        background: -webkit-linear-gradient(top,  #d64800 0%,#d20200 100%);
        background: linear-gradient(to bottom,  #d64800 0%,#d20200 100%);
      }
    }
  }
  &__icon {
    display: block;
    width: 22px;
    height: 22px;
    margin: 8px auto 0;
    background-repeat: no-repeat;
    background-position: center;
    background-image: url(assets/climate/ac-off.svg);
    .tesla-heat & {
      background-image: url(assets/climate/heat-off.svg);
    }
    .tesla-climate__item--active & {
      background-image: url(assets/climate/ac-on.svg);
    }
    .tesla-climate__item--active.tesla-heat & {
      background-image: url(assets/climate/heat-on.svg);
    }
  }
  p {
    margin: 14px 0 0;
    text-align: center;
    font-size: 10px;
    text-transform: uppercase;
  }
  input[type=checkbox] {
    border: 0;
    clip: rect(0 0 0 0);
    height: 1px;
    margin: -1px;
    overflow: hidden;
    padding: 0;
    position: absolute;
    width: 1px;
  }
}
{% endhighlight %}

Back to our `tesla-battery.module.ts`, we need to add:

{% highlight javascript %}
...
import { TeslaClimateComponent } from './components/tesla-climate/tesla-climate.component';

@NgModule({
  declarations: [
    TeslaBatteryComponent,
    TeslaCarComponent,
    TeslaStatsComponent,
    TeslaCounterComponent,
    // new addition
    TeslaClimateComponent
  ],
  ...
})
...
{% endhighlight %}

Now for the fun part, we need to implement that `limit`!

#### Conditional aircon/heating limits

Let's jump back into our `tesla-battery.component.ts` and add our custom form `tesla-climate` control (make sure it sits exactly as shown here as the styling keeps it looking jazzy):

{% highlight javascript %}
...
@Component({
  selector: 'tesla-battery',
  template: `
    <form class="tesla-battery" [formGroup]="tesla">
      <h1>{% raw %}{{ title }}{% endraw %}</h1>
      <tesla-car [wheelsize]="tesla.get('config.wheels').value"></tesla-car>
      <tesla-stats [stats]="stats"></tesla-stats>
      <div class="tesla-controls cf" formGroupName="config">
        <tesla-counter
          [title]="'Speed'"
          [unit]="'mph'"
          [step]="5"
          [min]="45"
          [max]="70"
          formControlName="speed">
        </tesla-counter>
        <div class="tesla-climate cf">
          <tesla-counter
            [title]="'Outside Temperature'"
            [unit]="'°'"
            [step]="10"
            [min]="-10"
            [max]="40"
            formControlName="temperature">
          </tesla-counter>
          <tesla-climate
            [limit]="tesla.get('config.temperature').value > 10"
            formControlName="climate">
          </tesla-climate>
        </div>
      </div>
      ...
      ...
    </form>
  `
})
...
{% endhighlight %}

The magic piece here is simply `tesla.get('config.temperature').value > 10` and passing that expression down as a binding to `[limit]`. This will be re-evaluated when Angular runs change detection on our component, and the `boolean` result of the expression down into the component. You can check the styling to see how it works internally with particular class name swapping.

<img src="img/posts/tesla/step-4.gif" alt="" style="width: 100%; max-width: 100%;">

### Wheel size component

This one's my favourite (and the final component) just because I love the animation on the wheels.

Go ahead and create a `/tesla-wheels/` directory inside the `/components` directory just like our previous component:

{% highlight bash %}
**/src/app/tesla-battery/components/tesla-wheels/
{% endhighlight %}

Then inside of there, create these two components:

{% highlight bash %}
tesla-wheels.component.ts
tesla-wheels.component.scss
{% endhighlight %}

Once you're done, populate your `tesla-wheels.component.ts` component with this, another custom form control that accesses `radio` inputs:

{% highlight javascript %}
import { Component, Input, ChangeDetectionStrategy, forwardRef } from '@angular/core';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

const RADIO_CONTROL_ACCESSOR = {
  provide: NG_VALUE_ACCESSOR,
  useExisting: forwardRef(() => TeslaWheelsComponent),
  multi: true
};

@Component({
  selector: 'tesla-wheels',
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div class="tesla-wheels">
      <p class="tesla-wheels__title">Wheels</p>
      <div class="tesla-wheels__container cf">
        <label
          *ngFor="let size of sizes;"
          class="tesla-wheels__item tesla-wheels__item--{% raw %}{{ size }}{% endraw %}"
          [class.tesla-wheels__item--active]="value === size"
          [class.tesla-wheels__item--focused]="focused === size">
          <input
            type="radio"
            name="wheelsize"
            [attr.value]="size"
            (blur)="onBlur(size)"
            (change)="onChange(size)"
            (focus)="onFocus(size)"
            [checked]="value === size">
          <p>
            {% raw %}{{ size }}{% endraw %}"
          </p>
        </label>
      </div>
    </div>
  `,
  providers: [RADIO_CONTROL_ACCESSOR],
  styleUrls: ['./tesla-wheels.component.scss']
})
export class TeslaWheelsComponent implements ControlValueAccessor {
  constructor() {}
  private onModelChange: Function;
  private onTouch: Function;
  private value: string;
  private focused: string;
  private sizes: number[] = [19, 21];

  registerOnChange(fn: Function) {
    this.onModelChange = fn;
  }

  registerOnTouched(fn: Function) {
    this.onTouch = fn;
  }

  writeValue(value: string) {
    this.value = value;
  }

  private onChange(value: string) {
    this.value = value;
    this.onModelChange(value);
  }

  private onBlur(value: string) {
    this.focused = '';
  }

  private onFocus(value: string) {
    this.focused = value;
    this.onTouch();
  }
}
{% endhighlight %}

The only real thing to note here is that we're using `private sizes` to dynamically generate the wheel sizes and then assign the correct class names to the elements. As it's a `radio` button, only one can be selected at a time, you'll also be able to use keyboard left/right/up/down arrows to flick through the sizes once we've implemented it!

As always, the styles. Jump into `tesla-wheels.component.scss`:

{% highlight css %}
.tesla-wheels {
  float: left;
  width: 355px;
  &__title {
    letter-spacing: 2px;
    font-size: 16px;
  }
  &__container {
    margin: 10px 0 0;
  }
  &__item {
    cursor: pointer;
    width: 47%;
    height: 65px;
    border: 1px solid #ccc;
    display: inline-block;
    padding: 20px 0 0 90px;
    margin: 0 8px 0 0;
    background-color: #f7f7f7;
    background-position: 24.21053% 9px;
    background-repeat: no-repeat;
    background-size: 44px;
    &--19 {
      background-image: url(assets/wheels/19.svg);
    }
    &--21 {
      background-image: url(assets/wheels/21.svg);
    }
    &--focused {
      background-color: #f2f2f2;
    }
    &--active {
      border-color: #39f;
      box-shadow: inset 0px 0px 0px 1px #39f;
    }
    p {
      font-family: 'RobotoNormal';
      font-size: 16px;
      font-weight: 400;
      color: #333;
    }
    input[type=radio] {
      border: 0;
      clip: rect(0 0 0 0);
      height: 1px;
      margin: -1px;
      overflow: hidden;
      padding: 0;
      position: absolute;
      width: 1px;
    }
  }
}
{% endhighlight %}

Back to our `tesla-battery.module.ts`, we need to add:

{% highlight javascript %}
...
import { TeslaWheelsComponent } from './components/tesla-wheels/tesla-wheels.component';

@NgModule({
  declarations: [
    TeslaBatteryComponent,
    TeslaCarComponent,
    TeslaStatsComponent,
    TeslaCounterComponent,
    TeslaClimateComponent,
    // new addition
    TeslaWheelsComponent
  ],
  ...
})
...
{% endhighlight %}

This one's an easy addition to our `tesla-battery.component.ts` (ensure it's outside the `<div>` containing the counters for styling purposes):

{% highlight javascript %}
...
@Component({
  selector: 'tesla-battery',
  template: `
    <form class="tesla-battery" [formGroup]="tesla">
      <h1>{% raw %}{{ title }}{% endraw %}</h1>
      <tesla-car [wheelsize]="tesla.get('config.wheels').value"></tesla-car>
      <tesla-stats [stats]="stats"></tesla-stats>
      <div class="tesla-controls cf" formGroupName="config">
        <tesla-counter
          [title]="'Speed'"
          [unit]="'mph'"
          [step]="5"
          [min]="45"
          [max]="70"
          formControlName="speed">
        </tesla-counter>
        <div class="tesla-climate cf">
          <tesla-counter
            [title]="'Outside Temperature'"
            [unit]="'°'"
            [step]="10"
            [min]="-10"
            [max]="40"
            formControlName="temperature">
          </tesla-counter>
          <tesla-climate
            [limit]="tesla.get('config.temperature').value > 10"
            formControlName="climate">
          </tesla-climate>
        </div>
        <tesla-wheels formControlName="wheels"></tesla-wheels>
      </div>
      ...
      ...
    </form>
  `
})
...
{% endhighlight %}

Now we're done! Or are we? Nothing actually changes when we change our form controls.

### FormGroup valueChange subscription

Now to implement the final feature, then we'll deploy it to GitHub pages with Ahead-of-Time compilation.

Jump inside your `tesla-battery.component.ts` again, inside `ngOnInit` add this:

{% highlight javascript %}
this.tesla.controls['config'].valueChanges.subscribe(data => {
  this.stats = this.calculateStats(this.results, data);
});
{% endhighlight %}

All we're doing here is accessing the `controls.config` Object (square bracket notation as TypeScript enjoys moaning) and subscribing to value changes. Once a value is changed, we can simply run the `calculateStats` method again with our existing results that we set at runtime, as well as the new data Object being passed as the second argument instead of the initial form value. The Objects are the same as the initial form value, so we can reuse the function, they just have different values.

Your `ngOnInit` should look like this:

{% highlight javascript %}
ngOnInit() {
  this.models = this.batteryService.getModelData();
  this.tesla = this.fb.group({
    config: this.fb.group({
      speed: 55,
      temperature: 20,
      climate: true,
      wheels: 19
    })
  });
  this.stats = this.calculateStats(this.results, this.tesla.controls['config'].value);
  this.tesla.controls['config'].valueChanges.subscribe(data => {
    this.stats = this.calculateStats(this.results, data);
  });
}
{% endhighlight %}

You should have a fully working Tesla range calculator:

<img src="img/posts/tesla/final.gif" alt="" style="width: 100%; max-width: 100%;">

### Deploying with Ahead-of-Time compilation

AoT means Angular will precompile everything (including our templates) and give us the bare minimum Angular needs for our application. I'm getting around `313 KB` for this entire project, including images, fonts. `184 KB` of that is Angular code!

#### Deploying to GitHub pages

Angular CLI to the rescue. Ready to deploy what you've just built?

Make sure you've _pushed all your changes_ to `master`, then run it:

{% highlight bash %}
ng github-pages:deploy
{% endhighlight %}

It should give you something like this:

{% highlight bash %}
Child html-webpack-plugin for "index.html":
    Asset       Size          Chunks       Chunk Names
    index.html  2.75 kB       0
    chunk    {0} index.html 286 bytes [entry] [rendered]
Deployed! Visit https://toddmotto.github.io/angular-tesla-range-calculator/
Github pages might take a few minutes to show the deployed site.
{% endhighlight %}

Visit the URL the CLI gives you and enjoy.

> Check out [my live version](https://toddmotto.com/angular-tesla-range-calculator/) if you'd like instead

#### Source code

Grab it all on [GitHub](https://github.com/toddmotto/angular-tesla-range-calculator).
