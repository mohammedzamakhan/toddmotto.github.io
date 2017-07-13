---
layout: post
permalink: /component-architecture-reactive-forms-angular
title: "Component architecture recipes for Angular&rsquo;s reactive forms"
path: 2017-05-08-component-architecture-reactive-forms-angular.md
tag: angular
tags:
  - rxjs
  - observables
---

Component architecture is the fundamental building block of applications, and isn't just limited to Angular. Building with components allows for things such as encapsulation, isolation and reusability for a single or set of components. When dealing with forms, using components has typically been a challenge - and I'd like to share some patterns when creating reactive forms with Angular's component based system that make dealing with forms a bit of a breeze.

<div class="toc" markdown="1">
<span class="gamma">Table of contents</span>
{:.no_toc}
* TOC
{:toc}
</div>

For the purposes of this article, we'll be focusing on some of Angular's reactive form APIs and how we can apply them to [container and presentational](/stateful-stateless-components) components to build out this small pizza application that demonstrates some things I've been thinking about lately.

<img src="/img/posts/pizza.gif" alt="Angular Reactive Pizza">

> You can find the full [source code here](https://github.com/toddmotto/reactive-pizza) for this app.

### Component architecture

Before we get started, let's look at how we can break down this form into some component building blocks.

We could further break this down into a few more components, but again they'd just be smaller presentational components. What we're focusing on is how to structure a componentised form and how we can tie those into container components that control the data for each presentational component.

<img src="/img/posts/pizza-breakdown.jpg" alt="Angular Reactive Pizza">

Moving forward from the above diagram, the *pink* box is a _container_ component, and the *red* boxes are presentational components.

### Container / presentational splits

Let's define how they'll look from an HTML perspective (this is _not_ template code, just conceptual):

```html
<pizza-creator>

  <pizza-form>
    <toppings-selector></toppings-selector>
    <pizza-name></pizza-name>
    <pizza-selected></pizza-selected>
    <pizza-button></pizza-button>
  </pizza-form>

  <pizza-list></pizza-list>

</pizza-creator>
```

So, now we've got some structure - let's continue with the first component, the container that holds everything else.

### The Container

Our container component will be the `<pizza-creator>` - let's take a look at the code:

```js
import { Component } from '@angular/core';

import { PizzaService } from '../../pizza.service';

import { Pizza, Topping } from '../../pizza.interface';

@Component({
  selector: 'pizza-creator',
  styleUrls: ['pizza-creator.component.scss'],
  template: `
    <div class="pizza-creator">
      <div class="pizza-creator__title">
        <h1>
          <img src="assets/logo.svg">
          Pizza Creator
        </h1>
      </div>
      <div class="pizza-creator__panes">
        <pizza-form
          [toppings]="toppings$ | async"
          (add)="addPizza($event)">
        </pizza-form>
        <pizza-list
          [pizzas]="pizzas$ | async">
        </pizza-list>
      </div>
    </div>
  `
})
export class PizzaCreatorComponent {

  pizzas$ = this.pizzaService.select<Pizza[]>('pizzas');
  toppings$ = this.pizzaService.select<Topping[]>('toppings');

  constructor(
    private pizzaService: PizzaService
  ) {}

  addPizza(event: any) {
    this.pizzaService.addPizza(event);
  }

}
```

First, let's dissect the component class. All we've got here is two Observable streams that map directly across to two child presentational components. The form is a presentational component and fully controlled via it's `@Input` and `@Output` bindings. Before we go further, let's take a look at the `PizzaService` to show the Observable layer.

The idea here is to pass the Observables into the template and subscribe directly to them, meaning `<pizza-form>` and `<pizza-list>` receive purely Objects or Arrays rather than an Observable. This means we can utilise things such as `OnPush` change detection strategies along with immutable `@Input` bindings to keep our components fast.

### Service and store

The code for the store uses a `BehaviorSubject` to notify subscribers of the store's state that it's time to update - updates are driven from the `addPizza` method on this class, which calls `.next()` on the subject to pass the next value. 

Our state for this service is driven from the `state` constant, which holds the initial state at runtime (populating the form with the toppings available for the store, and any existing pizzas in the inventory). This state initialises the `BehaviorSubject`.

You'll also notice the `pluck` operator to fetch properties from our state and return them as an Observable stream - we have a stream of `pizzas` as well as a stream of `toppings`.

```js
import { Injectable } from '@angular/core';

import { Pizza, Topping } from './pizza.interface';

import { BehaviorSubject } from 'rxjs/BehaviorSubject';
import { Observable } from 'rxjs/Observable';
import 'rxjs/add/operator/pluck';
import 'rxjs/add/operator/distinctUntilChanged';

export interface State {
  pizzas: Pizza[],
  toppings: Topping[]
}

const state: State = {
  pizzas: [
    { name: 'New Yorker', toppings: ['Bacon', 'Pepperoni', 'Ham', 'Mushrooms'] },
    { name: 'Hot & Spicy', toppings: ['Jalapenos', 'Herbs', 'Pepperoni', 'Chicken'] },
    { name: 'Hawaiian', toppings: ['Ham', 'Pineapple', 'Sweetcorn'] }
  ],
  toppings: [
    'Bacon', 'Pepperoni', 'Mushrooms', 'Herbs',
    'Chicken', 'Pineapple', 'Ham', 'Jalapenos'
  ]
};

@Injectable()
export class PizzaService {

  private subject = new BehaviorSubject<State>(state);
  store = this.subject.asObservable().distinctUntilChanged();

  select<T>(name: string): Observable<T> {
    return this.store.pluck(name);
  }

  addPizza(pizza: Pizza) {
    const value = this.subject.value;
    this.subject.next({ ...value, pizzas: [...value.pizzas, pizza] });
  }

}
```

Note the above `select<T>() {}` method, inspired by `ngrx/store` implementation - we can add similar behaviour in a few lines to fetch properties on our store. The only way to update our store is calling `addPizza`, which wraps the `.next()` method to the `BehaviorSubject`.

### Presentational components

Next, we'll take a look at some of the attributes and characteristics of our presentational components to see how they're composed and flow in our component-based system. Everything from here accepts data from the above store, which is injected and data is passed down through our component layers.

#### The Presentational Form

Forms can either be containers or presentational elements, or a mixture of both depending on data structure and the goal for the form. I've been trying to make the forms as "presentational" (or "dumb") as possible.

This keeps the internal workings of the form powered, and it simply just takes data and performs it's intended role.

Let's first take a look at the full code, then discuss:

```js
import { Component, Input, Output, EventEmitter, ChangeDetectionStrategy } from '@angular/core';
import { FormBuilder, FormControl, FormGroup, FormArray, Validators } from '@angular/forms';

import { Pizza, Topping } from '../../pizza.interface';
import { ToppingsValidator } from '../../toppings.validator';

@Component({
  selector: 'pizza-form',
  changeDetection: ChangeDetectionStrategy.OnPush,
  styleUrls: ['pizza-form.component.scss'],
  template: `
    <form [formGroup]="form">

      <toppings-selector
        [parent]="form"
        [toppings]="toppings"
        [selected]="control.value"
        (select)="selectTopping($event)">
      </toppings-selector>

      <pizza-name 
        [parent]="form">
      </pizza-name>

      <pizza-selected
        [parent]="form"
        [selected]="control.value"
        (remove)="removeTopping($event)">
      </pizza-selected>

      <pizza-button
        [parent]="form"
        (add)="onSubmit()">
        Add pizza
      </pizza-button>

    </form>
  `
})
export class PizzaFormComponent {

  @Input()
  toppings: Topping[];

  @Output()
  add = new EventEmitter<FormGroup>();

  form = this.fb.group({
    name: ['', Validators.required],
    toppings: this.fb.array([])
  }, {
    validator: ToppingsValidator
  });

  constructor(
    private fb: FormBuilder
  ) {}

  get control() {
    return this.form.get('toppings') as FormArray;
  }

  addTopping(topping: Topping) {
    this.control.push(new FormControl(topping));
  }

  removeTopping(index: number) {
    this.control.removeAt(index);
  }

  selectTopping(topping: Topping) {
    const index = this.control.value.indexOf(topping);
    if (!!~index) {
      this.removeTopping(index);
    } else {
      this.addTopping(topping);
    }
  }

  onSubmit() {
    if (this.form.invalid) {
      return;
    }
    this.add.emit(this.form.value);
  }

}
````

There is a lot happening here! First thing to note is that we have a `<form>` tag with 4 components inside which render further templates and respective data.

Everything is pretty standard, however we have a custom validator to add:

```js
import { FormArray, AbstractControl } from '@angular/forms';

export const ToppingsValidator = (control: AbstractControl): {[key: string]: boolean} => {
  const toppings = (control.get('toppings') as FormArray).value;
  return toppings.length ? null : { noToppings: true };
};
```

This gets used further down in the form to render an error message - but that's pretty much the full setup picture.

#### Uni-directional Form logic

One thing I've been looking at is "how" reactive forms control themselves. You'll notice we have 4 child components, yet we have `FormArray` methods being used in this form such as `.push()` or `.removeAt()` inside callbacks - why?

I think this approach is the most sensible for composing reactive forms and their local state. We simply just care about the outputted value when the form is submitted, not the way it's composed. It allows us to control the flow of the form from a single point.

This plays into the same concept as how uni-directional dataflow would also be achieved through `@Output()` callbacks to then merge that new event data into a particular function or variable - updating the data in the "container". With a form like this, we're saying that this form acts as a "container" - but for the form APIs - such as `FormControl`, `FormArray` and friends.

> If you want a new item in your `FormArray` - it's the form container that adds it upon request of a child component.

The alternative way of doing this would be communicating with the form instance that's delegated into the child components and calling the methods there (which I've found to be more troublesome).

#### Topping selector component

The toppings selector component is deliberately named something a little different to the other form components as it's not "technically" part of the form. It simply allows you to display whatever the data gives you, for which you'll then select a value from that component to add a topping to your new pizza.

```js
import { Component, Input, Output, EventEmitter, ChangeDetectionStrategy } from '@angular/core';
import { FormGroup } from '@angular/forms';

import { Topping } from '../../pizza.interface';

@Component({
  selector: 'toppings-selector',
  changeDetection: ChangeDetectionStrategy.OnPush,
  styleUrls: ['toppings-selector.component.scss'],
  template: `
    <div class="toppings-selector" [formGroup]="parent">
      <div
        class="toppings-selector__item"
        *ngFor="let topping of toppings"
        [class.active]="isActive(topping)"
        (click)="onSelect(topping)">
        {% raw %}{{ topping }}{% endraw %}
      </div>
      <div 
        class="error"
        *ngIf="invalid">
        Select at least 1 topping
      </div>
    </div>
  `
})
export class ToppingsSelectorComponent {

  touched = false;

  @Input()
  parent: FormGroup;

  @Input()
  selected: Topping[];

  @Input()
  toppings: Topping[];

  @Output()
  select = new EventEmitter<Topping>();

  get invalid() {
    return (
      this.parent.hasError('noToppings') &&
      this.touched
    );
  }

  exists(topping: Topping) {
    return !!~this.selected.indexOf(topping);
  }

  isActive(topping: Topping) {
    return this.exists(topping);
  }

  onSelect(topping: Topping) {
    this.touched = true;
    this.select.emit(topping);
  }

}
```

This component takes two collections of `Topping[]` - the list of toppings you _can_ select, followed by the `selected` toppings - so we can update some UI state for selected any toppings.

#### Pizza name component

Our first true "form" component. It allows the user to give their pizza a custom name before submitting the form:

```js
import { Component, Input, ChangeDetectionStrategy } from '@angular/core';
import { FormGroup } from '@angular/forms';

@Component({
  selector: 'pizza-name',
  changeDetection: ChangeDetectionStrategy.OnPush,
  styleUrls: ['pizza-name.component.scss'],
  template: `
    <div class="pizza-name" [formGroup]="parent">
      <input 
        type="text" 
        placeholder="Pizza name, e.g. Blazin' hot" 
        formControlName="name">
      <div 
        class="error"
        *ngIf="invalid">
        Pizza name is required
      </div>
    </div>
  `
})
export class PizzaNameComponent {

  @Input()
  parent: FormGroup;

  get invalid() {
    return (
      this.parent.get('name').hasError('required') &&
      this.parent.get('name').touched
    );
  }

}
```

The key here is passing down the `FormGroup` as a binding - to which I've called `parent` to give me some clear indication. To get our form components communicating with the parent form we must do this and re-bind `[formGroup]="parent"` to the containing element.

This then allows us to use our `formControlName="name"` without a hitch. You'll also notice the property getter `get invalid() {}` where I return the state of the form's `'name'` control.

Doing this is a crucial step to fully componentising forms, as we need to nest components to create complex, yet easy to maintain forms and components. The above example demonstrates `formControlName` bindings and registering within the parent form.

#### Validation notes

We'll step aside real quick to discuss how validation is being integrated here. Because of the way `parent` is passed down - I think in this case it's fine to leave the validation logic local to the component. It will just respond to Angular's validation states and look after itself. More complex validation states that rely on multiple controls could be passed down as bindings - or left internal as above.

#### Pizza selected component

Here, we're showing some of the concepts we've described before with uni-directional form logic:

```js
import { Component, Input, Output, EventEmitter, ChangeDetectionStrategy } from '@angular/core';
import { FormGroup } from '@angular/forms';

import { Topping } from '../../pizza.interface';

@Component({
  selector: 'pizza-selected',
  changeDetection: ChangeDetectionStrategy.OnPush,
  styleUrls: ['pizza-selected.component.scss'],
  template: `
    <div class="pizza-selected" [formGroup]="parent">
      <div class="pizza-selected__empty" *ngIf="!selected.length">
        Select toppings to create pizza
      </div>
      <div
        class="pizza-selected__list" 
        *ngIf="selected.length"
        formArrayName="toppings">
        <div 
          class="pizza-selected__item" 
          *ngFor="let topping of selected; index as i;">
          <div [formGroupName]="i">
            <img src="assets/check.svg">
            {% raw %}{{ topping }}{% endraw %}
            <button 
              type="button"
              (click)="onRemove(i)">
              <img src="assets/cross.svg">
            </button>
          </div>
        </div>
      </div>
    </div>
  `
})
export class PizzaSelectedComponent {

  @Input()
  parent: FormGroup;

  @Input()
  selected: Topping[];

  @Output()
  remove = new EventEmitter<number>();

  onRemove(index: number) {
    this.remove.emit(index);
  }

}
```

Again, we're passing the `parent` form down, binding and then utilising the `formArrayName` directive. The interesting piece is when I click the remove topping button - it _doesn't_ do something like this:

```js
onRemove(index: number) {
  (this.parent.get('toppings') as FormArray).removeAt(index);
}
```

It in fact emits an event:

```js
onRemove(index: number) {
  this.remove.emit(index);
}
```

To which the parent is in control:

```js
// template code
`
  <pizza-selected
    [parent]="form"
    [selected]="control.value"
    (remove)="removeTopping($event)">
  </pizza-selected>
`

// component class
get control() {
  return this.form.get('toppings') as FormArray;
}

removeTopping(index: number) {
  this.control.removeAt(index);
}
```

This, as previously mentioned, helps us control the form state from a predictive single point. Call it a "presentational form" which acts like a container _but_ for form APIs only - not data. All data is still received as an `@Input()`.

#### Pizza button component

This component isn't too exciting, but if you've got multiple buttons per form that trigger different actions - then you may wish to take a similar approach to this:

```js
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { FormGroup } from '@angular/forms';

@Component({
  selector: 'pizza-button',
  styleUrls: ['pizza-button.component.scss'],
  template: `
    <div class="pizza-button" [formGroup]="parent">
      <button 
        type="button"
        (click)="onClick()"
        [disabled]="parent.invalid">
        <img src="assets/add.svg">
        <ng-content></ng-content>
      </button>
    </div>
  `
})
export class PizzaButtonComponent {

  @Input()
  parent: FormGroup;

  @Output()
  add = new EventEmitter<any>();

  onClick() {
    this.add.emit();
  }

}
```

Binds `parent`, emits action, triggers parent callback. The reason we're binding the `[formGroup]` here is to `disable` the button if the form currently has an active `invalid` state.

#### Pizza list component

The `<pizza-list>` component is the second sibling component inside our `<pizza-creator>` container, it's job is to take the unwrapped Observable `@Input()` and render the full list of pizzas:

```js
import { Component, Input, ChangeDetectionStrategy } from '@angular/core';

import { Pizza } from '../../pizza.interface';

@Component({
  selector: 'pizza-list',
  changeDetection: ChangeDetectionStrategy.OnPush,
  styleUrls: ['pizza-list.component.scss'],
  template: `
    <div class="pizza-list">
      <h2>Store inventory</h2>
      <div *ngFor="let pizza of pizzas">
        <p>{% raw %}{{ pizza.name }}{% endraw %}</p>
        <span>{% raw %}{{ pizza.toppings | join }}{% endraw %}</span>
      </div>
    </div>
  `
})
export class PizzaListComponent {

  @Input()
  pizzas: Pizza[];

}
```

And that's about it!

### Conclusion

Thinking of uni-directional form communication is a great strategy for componentising forms, and dynamic aspects of forms, as well as helping tell the component architecture story. It makes actions clean, controlled in a single place - and the form manages itself.

Once the form is happy, we fire the `@Output()` event with the form data and our container takes care of the rest of the data layer, delegating off to our service.

You can checkout the [full source code here](https://github.com/toddmotto/reactive-pizza).