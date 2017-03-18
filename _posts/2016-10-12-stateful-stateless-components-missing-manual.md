---
layout: post
permalink: /stateful-stateless-components
title: Stateful and stateless components, the missing manual
path: 2016-10-12-stateful-stateless-components-missing-manual.md
---

The goals of this article are to define what stateful and stateless components are, otherwise known as smart and dumb - or container and presentational components. For the purposes of the article, we'll be using Angular 2 Components to explain the stateful/stateless concepts. Bear in mind these concepts are not at all limited to Angular, and live in other libs/frameworks such as React.

### Table of contents

- <a href="#terminology">Terminology</a>
  - <a href="#stateful">Stateful</a>
  - <a href="#stateless">Stateless</a>
  - <a href="#components">Components</a>
- <a href="#impure-versus-pure-functions">Impure versus Pure functions</a>
  - <a href="#impure-functions-stateful">Impure functions (stateful)</a>
  - <a href="#pure-functions-stateless">Pure functions (stateless)</a>
- <a href="#stateful-components">Stateful components</a>
  - <a href="#stateful-todo-component">Stateful Todo component</a>
- <a href="#stateless-components">Stateless components</a>
  - <a href="#stateless-todoform-component">Stateless TodoForm component</a>
  - <a href="#stateless-todolist-component">Stateless TodoList component</a>
  - <a href="#final-code">Final code</a>
- <a href="#angular-1x-version">AngularJS 1.x version?</a>
  - <a href="#final-code">Final code</a>
- <a href="#further-reading">Further reading</a>

### Terminology

Before we begin, let's clarify what "stateful" and "stateless" really mean in programming terms.

#### Stateful

When something is "stateful", it is a central point that stores information in memory about the app/component's state. It also has the ability to change it. It is essentially a "living" thing that has knowledge of past, current and potential future state changes.

#### Stateless

When something is "stateless", it calculates its internal state but it never directly mutates it. This allows for complete referential transparency meaning that given the same inputs, it will _always_ produce the same output. These are not essentially "living" as they are merely passed information. This means it has no knowledge of the past, current or future state changes.

#### Components

When we talk about stateful and stateless in web application development, we can begin to apply these concepts to component paradigms. So what is a component? A component is an isolated piece of behaviour or functionality that allows us to divide behaviour into roles, much like we would with JavaScript functions.

### Impure versus Pure functions

When we think about stateful and stateless components, we can ignore any framework entirely until it comes to implementation and think about JavaScript functions. Let's first consider pure versus impure functions, and somewhat compare them to stateful and stateless. I really like comparing component types to functions to better understand UI composition.

In my eyes, after reading this you'll match up:

* Impure Function = Stateful Component
* Pure Function = Stateless Component

Check out my [Pure versus Impure functions](/pure-versus-impure-functions) article for more depth, but we'll cover the basics here.

#### Impure functions (stateful)

Let's consider the following code from the above article, which parses the user's `weight` and `height` values into Number types, then calculates the `bmi` (Body Mass Index) based on the formula.

{% highlight javascript %}
const weight = parseInt(form.querySelector('input[name=weight]').value, 10);
const height = parseInt(form.querySelector('input[name=height]').value, 10);

const bmi = (weight / (height /100 * height / 100)).toFixed(1);
{% endhighlight %}

This is great in terms of the fact that it works, however this doesn't create a reusable function that allows us to calculate the BMI elsewhere, test the formula easily, and relies on very procedural code. Let's look at how we can make it "pure". The important piece here is that this chunk of code is the driver of changes, and we can rely on pure functions to essentially create small isolated pieces of logic that accept data and return new data without relying on external variables.

#### Pure functions (stateless)

When we think about pure functions, we would be able to expect the same result every time, without the knowledge of lexical constant values such as `weight` or `height`. Let's refactor the formula to be a pure function:

{% highlight javascript %}
const weight = form.querySelector('input[name=weight]').value;
const height = form.querySelector('input[name=height]').value;

const getBMI = (weight, height) => {
  let newWeight = parseInt(weight, 10);
  let newHeight = parseInt(height, 10);
  return (newWeight / (newHeight /100 * newHeight / 100)).toFixed(1);
};

const bmi = getBMI(weight, height);
{% endhighlight %}

The `getBMI` function can easily live elsewhere, not necessarily in the same function (as this example is inside the `onSubmit`) function if you check the other article. Now the function is pure, it can be defined better in terms of "why". The function has several attributes:

* It can be easily tested with mocked data
* It can be re-used multiple times to perform the role it has been given
* It has a defined input (function arguments)
* It has a defined output (`return` statement with new data)

Here's the thing: all four of the above concepts directly map across to thinking about stateless components.

Let's now take the "impure" functions and look at the stateful component equivalent, followed by the "pure" functions that can be mapped across to stateless components.

### Stateful components

Much like an impure JavaScript function, a stateful component is the driver of what happens, and it therefore utilises any stateless components at our disposal.

Here are some attributes that a stateful component has:

* Drives state changes through functions
* Provides data (i.e. from http layers)
* May receive initial data via [route resolves](/resolve-promises-in-angular-routes/) instead of service layer calls
* Has living knowledge of the current state
* Is informed by stateless components when something needs to change
* Can communicate with external dependencies (such as an http layer)
* Renders stateless (or even stateful) child components, perhaps within a single `<div>` wrapper for layout containment
* Contain Redux actions (ngrx/store or ng2redux for example)

This list, and the one further in the article, was inspired by Dan Abramov's [Presentational and Container components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0) article.

#### Stateful Todo component

In this article, we're going to build a small todo application demonstrating these concepts, followed by their stateless counterparts.

First, let's start off by rendering our base component, the `<app>` to kick things off:

{% highlight javascript %}
import { Component } from '@angular/core';

@Component({
  selector: 'my-app',
  template: `
    <todos></todos>
  `
})
export class AppComponent { }
{% endhighlight %}

Inside here, we're rendering a `<todos>` component. This component will be stateful, let us continue! We're not going to teach how to build a todo app, we all know this, so we're going to look at how we can apply stateful and stateless paradigms to Angular 2 components and observe the ideas.

Let's look at the component composition through ASCII art as we progress, so far we have an `<app>` component:

{% highlight html %}

          ┌─────────────────┐          
          │      <app>      │          
          └─────────────────┘          
{% endhighlight %}

Now onto the `<todos>` component:

{% highlight javascript %}
import { Component, OnInit } from '@angular/core';
import { TodoService } from './todo.service';

@Component({
  selector: 'todos',
  template: `
  <div>
    <todo-form
      (onAdd)="addTodo($event)">
    </todo-form>
    <todo-list
      [todos]="todos"
      (onComplete)="completeTodo($event)"
      (onDelete)="removeTodo($event)">
    </todo-list>
  </div>  
  `
})
export class TodosComponent implements OnInit {
  todos: any[];
  constructor(private todoService: TodoService) {}
  ngOnInit() {
    this.todos = this.todoService.getTodos();
  }
  addTodo({label}) {
    this.todos = [{label, id: this.todos.length + 1}, ...this.todos];
  }
  completeTodo({todo}) {
    this.todos = this.todos.map(
      item => item.id === todo.id ? Object.assign({}, item, {complete: true}) : item
    );
  }
  removeTodo({todo}) {
    this.todos = this.todos.filter(({id}) => id !== todo.id);
  }
}
{% endhighlight %}

You can see from the above that all we have is a container `<div>` wrapping two further child (stateless) components. There is no other logic in the template other than that. The `<todo-form>` component receives no input, but expects to bind an output called `onAdd`. Next up, the `<todo-list>` component receives the todos data from the `[todos]` input binding, and two outputs `(onComplete)` and `(onDelete)`, delegating the respective functions to the stateless counterparts.

The rest of the component class are the methods that make up the functionality of the todo component. Immutable operations are taking place inside each callback, and each callback is exposed to a stateless component so that it can run with it. All these functions are expecting is a notification that something has changed, for instance "Oh hey! Here's a new todo label, go ahead and do your thing with it oh-mighty stateful component". Note how the functions are only called from the child, stateless, level.

And that's literally it on stateful. We cover some of the potential concepts that stateful components may contain. Let's move onto the more frequently used stateless components.

ASCII (the `TodoService` represents the injected Service):

{% highlight html %}

          ┌─────────────────┐          
          │      <app>      │          
          └────────┬────────┘          
                   ▼                   
     ┌─────────────────────────────┐    
     │           <todos>           │    
     │     ┌─────────────────┐     │    
     │     │   TodoService   │     │   
     └─────┴─────────────────┴─────┘   

{% endhighlight %}

### Stateless components

Much like a pure JavaScript function, a stateless component isn't aware of "lexical" variables, in the fact that it receives data via property binding (equivalent to function arguments) and emits changes via an event (equivalent to a `return` block).

So what does this mean? Well, based on how function scope chains work, this means that stateless components have no knowledge of any part of the application they're apart of. Which means they can be reused, easily tested and moved around very easily.

Here are some attributes that a stateless component has:

* Do not request/fetch data
* Are _passed_ data via property binding
* Emit data via event callbacks
* Renders further stateless (or even stateful) components
* Can contain local UI state
* Are a small piece of a bigger picture

#### Stateless TodoForm component

Before we begin with this component, we need to understand that it's a special kind of stateless component in the fact that it retrieves user input, and therefore contains UI state only:

{% highlight javascript %}
import { Component, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'todo-form',
  template: `
  <form (ngSubmit)="submit()">
    <input name="label" [(ngModel)]="label">
    <button type="submit">Add todo</button>
  </form>
  `
})
export class TodoFormComponent {
  label: string;
  @Output() onAdd = new EventEmitter();
  submit() {
    if (!this.label) return;
    this.onAdd.emit({label: this.label});
    this.label = '';
  };
}
{% endhighlight %}

This component also doesn't receive any data via property binding, and that's perfectly acceptable. The role this component plays is to capture the label of a new todo item, which is illustrated inside the `submit` function. This is a special use case for stateless components that have a function inside to capture UI state and do something with it.

#### Stateless TodoList component

Let's look at the second stateless component we have, a direct child of `<todos>`:

{% highlight javascript %}
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'todo-list',
  template: `
  <ul>
    <li *ngFor="let todo of todos">
      <todo
        [item]="todo"
        (onChange)="onComplete.emit($event)"
        (onRemove)="onDelete.emit($event)">
      </todo>
    </li>
  </ul>
  `
})
export class TodoListComponent {
  @Input() todos;
  @Output() onComplete = new EventEmitter();
  @Output() onDelete = new EventEmitter();
}
{% endhighlight %}

Our `@Input` and `@Output` is well defined here, and as you can see, nothing else exists on this component class. We're actually creating an `EventEmitter` instance for each output, and also delegating this down into further stateless components, in this case the single `<todo>` component, which will render each todo in our collection. We also delegate the `onComplete` and `onDelete` methods here, which are also bound to the parent, creating a basic chain. Let's look inside `<todo>` and we're done:

{% highlight javascript %}
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'todo',
  styles: [`
    .complete { text-decoration: line-through; }
  `],
  template: `
  <div>
    <span [ngClass]="{ complete: item.complete }">{% raw %}{{ item.label }}{% endraw %}</span>
    <button
      type="button"
      (click)="onChange.emit({ todo: item });">Done</button>
    <button
      type="button"
      (click)="onRemove.emit({ todo: item });">Delete</button>
  </div>
  `
})
export class TodoComponent {
  @Input() item;
  @Output() onChange = new EventEmitter();
  @Output() onRemove = new EventEmitter();
}
{% endhighlight %}

Hopefully you can see a pattern emerging here! Again, we have some inputs and outputs that can send event information up to the parent, then up again (if needed). All of the above Angular 2 components are stateless. They have no knowledge of their surroundings, but are passed data via property bindings and emit changes via event callbacks.

Here's the final ASCII render of the component tree that we've talked through:

{% highlight html %}

          ┌─────────────────┐          
          │      <app>      │          
          └────────┬────────┘          
                   ▼                   
     ┌─────────────────────────────┐    
     │           <todos>           │    
     │     ┌─────────────────┐     │    
    ┌┤     │   TodoService   │     ├┐   
    │└─────┴─────────────────┴─────┘│   
┌──▼──────────────┐ ┌──────────────▼──┐
│   <todo-form>    │ │   <todo-list>   │
└──────────────────┘ └──────────────┬──┘
                     ┌──────────────▼──┐
                     │     <todo>      │
                     └─────────────────┘
{% endhighlight %}

#### Final code

Altogether now:

<iframe src="//embed.plnkr.co/ygSstbXkJf5vnGz6Fzdu" frameborder="0" border="0" cellspacing="0" cellpadding="0" width="100%" height="400"></iframe>

### AngularJS 1.x version?

Oh why not...

#### Full 1.x implementation

Here's the [full source code](https://jsfiddle.net/toddmotto/88g1gef4/) for the AngularJS 1.x version (obviously in a real app we'd use ES6 `import` and `export` statements etc):

{% highlight javascript %}
const todos = {
  template: `
    <div>
      <todo-form
        new-todo="$ctrl.newTodo"
        on-add="$ctrl.addTodo($event);">
      </todo-form>
      <todo-list
        todos="$ctrl.todos"
        on-complete="$ctrl.completeTodo($event);"
        on-delete="$ctrl.removeTodo($event);">
      </todo-list>
    </div>
  `,
  controller: class TodoController {
    constructor(TodoService) {
      this.todoService = TodoService;
    }
    $onInit() {
      this.todos = this.todoService.getTodos();
    }
    addTodo({ label }) {
      this.todos = [{ label, id: this.todos.length + 1 }, ...this.todos];
    }
    completeTodo({ todo }) {
      this.todos = this.todos.map(
        item => item.id === todo.id ? Object.assign({}, item, { complete: true }) : item
      );
    }
    removeTodo({ todo }) {
      this.todos = this.todos.filter(({ id }) => id !== todo.id);
    }
  }
};

const todoForm = {
  bindings: {
    onAdd: '&'
  },
  template: `
    <form ng-submit="$ctrl.submit();">
      <input ng-model="$ctrl.label">
      <button type="submit">Add todo</button>
    </form>
  `,
  controller: class TodoFormController {
    constructor() {}
    submit() {
      if (!this.label) return;
    	this.onAdd({
        $event: { label: this.label }
      });
      this.label = '';
    };
  }
};

const todoList = {
  bindings: {
    todos: '<',
    onComplete: '&',
    onDelete: '&'
  },
  template: `
    <ul>
      <li ng-repeat="todo in $ctrl.todos">
        <todo
          item="todo"
          on-change="$ctrl.onComplete($locals);"
          on-remove="$ctrl.onDelete($locals);">
        </todo>
      </li>
    </ul>
  `
};

const todo = {
  bindings: {
    item: '<',
    onChange: '&',
    onRemove: '&'
  },
  template: `
    <div>
      <span ng-class="{ complete: $ctrl.item.complete }">{% raw %}{{ $ctrl.item.label }}{% endraw %}</span>
      <button
        type="button"
        ng-click="$ctrl.onChange({ $event: { todo: $ctrl.item } });">Done</button>
      <button
        type="button"
        ng-click="$ctrl.onRemove({ $event: { todo: $ctrl.item } });">Delete</button>
    </div>
  `
};

class TodoService {
  constructor() {}
  getTodos() {
    return [{
      label: 'Eat pizza',
      id: 0,
      complete: true
    },{
      label: 'Do some coding',
      id: 1,
      complete: true
    },{
      label: 'Sleep',
      id: 2,
      complete: false
    },{
      label: 'Print tickets',
      id: 3,
      complete: true
    }];
  }
}

angular
  .module('app', [])
  .component('todos', todos)
  .component('todo', todo)
  .component('todoForm', todoForm)
  .component('todoList', todoList)
  .service('TodoService', TodoService);
{% endhighlight %}



### Further reading

If you need some further heads up on `@Input`, `@Output` and `EventEmitter` work, check my [@Input](/passing-data-angular-2-components-input) and [@Output and EventEmitter](/component-events-event-emitter-output-angular-2) articles.
