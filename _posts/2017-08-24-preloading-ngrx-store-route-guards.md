---
layout: post
permalink: /preloading-ngrx-store-route-guards
title: "Preloading ngrx/store with Route Guards"
path: 2017-08-24-preloading-ngrx-store-route-guards.md
tag: angular
---

Using ngrx/store (and some love for ngrx/effects) is definitely a great step for my productivity when managing state in Angular, and I want to share a small setup that'll allow you to preload data as normal with a Route Guard. The interesting thing is that the route guard will also make an API request if we currently have no data in the Store - and populate the store with the data before we finish transitioning to the route.

This is great, because I can hit a particular route, it loads the data and populates my store, at which point I can navigate away from the route and come back. When I come back - as long as I haven't refreshed the app the data will still be in the Store, and skip the API request. This is neat!

<div class="toc" markdown="1">
<span class="gamma">Table of contents</span>
{:.no_toc}
* TOC
{:toc}
</div>

### Routes setup

Let's take a snippet from the code I'm currently working on, where I have a higher level `CourseComponent` which embeds a `<router-outlet>` for child routes.

On the route, I've got a `canActivate` property which passes in my `CoursesGuard`. The role of this guard is to actually check our Store for data, and dispatch an action to make an API request if there's no data.

```js
export const ROUTES: Routes = [
  {
    path: ':id',
    canActivate: [CoursesGuard],
    component: CourseComponent,
    children: [
      { path: '', redirectTo: 'metadata', pathMatch: 'full' },
      { path: 'metadata', component: CourseMetadataComponent },
      { path: 'curriculum', component: CourseCurriculumComponent },
      { path: 'prices', component: CoursePricesComponent },
      { path: 'coupons', component: CourseCouponsComponent },
    ],
  },
];
```

### Route Guard

First, let's assume the basics of a typical Route Guard:

```js
import { Injectable } from '@angular/core';
import { CanActivate } from '@angular/router';

import { Observable } from 'rxjs/Observable';
import { of } from 'rxjs/observable/of';

@Injectable()
export class CoursesGuard implements CanActivate {
  canActivate(): Observable<boolean> {
    // return of(true | false)
  }
}
```

Now let's add some further code and explain:

```js
// ...imports etc.

@Injectable()
export class CoursesGuard implements CanActivate {
  constructor(private store: Store<CoursesState>) {}

  // wrapping the logic so we can .switchMap() it
  getFromStoreOrAPI(): Observable<any> {

    // return an Observable stream from the store
    return this.store
      // selecting the courses state using a feature selector
      .select(getCoursesState)
      // the .do() operator allows for a side effect, at this
      // point, I'm checking if the courses property exists on my 
      // Store slice of state
      .do((data: any) => {
        // if there are no courses, dispatch an action to hit the backend
        if (!data.courses.length) {
          this.store.dispatch(new Courses.CoursesGet());
        }
      })
      // filter out data.courses, no length === empty!
      .filter((data: any) => data.courses.length)
      // which if empty, we will never .take()
      // this is the same as .first() which will only
      // take 1 value from the Observable then complete
      // which does our unsubscribing, technically.
      .take(1);
  }

  // our guard that gets called each time we 
  // navigate to a new route
  canActivate(): Observable<boolean> {
    // return our Observable stream from above
    return this.getFromStoreOrAPI()
      // if it was successful, we can return Observable.of(true)
      .switchMap(() => of(true))
      // otherwise, something went wrong
      .catch(() => of(false));
  }
}
```

This technique is quite nice, simple and readible. Here's the full non-annotated source code:

```js
import { Injectable } from '@angular/core';
import { CanActivate } from '@angular/router';

import { Store } from '@ngrx/store';

import { Observable } from 'rxjs/Observable';
import 'rxjs/add/operator/take';
import 'rxjs/add/operator/switchMap';
import 'rxjs/add/operator/do';
import 'rxjs/add/operator/catch';
import { of } from 'rxjs/observable/of';

import { CoursesState, getCoursesState } from '../store/reducers/';
import * as Courses from '../store/actions/courses.actions';

@Injectable()
export class CoursesGuard implements CanActivate {
  constructor(private store: Store<CoursesState>) {}

  getFromStoreOrAPI(): Observable<any> {
    return this.store
      .select(getCoursesState)
      .do((data: any) => {
        if (!data.courses.length) {
          this.store.dispatch(new Courses.CoursesGet());
        }
      })
      .filter((data: any) => data.courses.length)
      .take(1);
  }

  canActivate(): Observable<boolean> {
    return this.getFromStoreOrAPI()
      .switchMap(() => of(true))
      .catch(() => of(false));
  }
}
```

### Actions

I'm using a few actions to manage the `GET` request to the courses API, here's a glimpse of the actions which you can see `CoursesGet()` being used in the above guard:

```js
export const COURSES_GET = '[Courses] Courses Get';
export const COURSES_GET_SUCCESS = '[Courses] Courses Get Success';
export const COURSES_GET_FAILURE = '[Courses] Courses Get Failure';

export class CoursesGet implements Action {
  readonly type = COURSES_GET;
}

export class CoursesGetSuccess implements Action {
  readonly type = COURSES_GET_SUCCESS;
  constructor(public payload: { courses: Course[] }) {}
}

export class CoursesGetFailure implements Action {
  readonly type = COURSES_GET_FAILURE;
  constructor(public payload: any) {}
}
```

In typical redux, you'd do something like:

```js
store.dispatch({
  type: 'COURSES_GET_SUCCESS',
  payload: {...}
});
```

This class based approach allows us for better type checking and `payload` configuration - as well as the logic being contained in a single place which is simply imported and a new instance of an action is created.

Let's finish this off by taking a look at the `ngrx/effects` side of this logic.

### Effects

The effect for this simply listens for the `COURSES_GET` action, and once invoked will hit the `.getCourses()` method on my `CoursesService`

```js
@Injectable()
export class CoursesEffects {

  constructor(
    private actions$: Actions,
    private coursesService: CoursesService
  ) {}

  // ...

  @Effect()
  getCourses$ = this.actions$
    .ofType(Courses.COURSES_GET)
    .exhaustMap(() =>
      this.coursesService
        .getCourses()
        .map(courses => new Courses.CoursesGetSuccess({ courses }))
        .catch(error => of(new Courses.CoursesGetFailure(error)))
    );

  // ...
}
```

Once that's been 200 ok'd from the backend, the `.map()` then invokes a new action, passing in the payload - which then merges the new state in my reducer.