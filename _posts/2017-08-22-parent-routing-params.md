---
layout: post
permalink: /angular-parent-routing-params
title: "Accessing parent Route params in Angular"
path: 2017-08-22-parent-routing-params.md
tag: angular
---

With the router/URL being an application's "source of truth", we need to be able to access parts of the URL for data purposes, such as grabbing a dynamic `:id` property from the URL, passing it into a service and bringing back the relevant data. Let's look at how we can use Angular's Observable-driven Router so get automatic updates when a parent route changes.

<div class="toc" markdown="1">
<span class="gamma">Table of contents</span>
{:.no_toc}
* TOC
{:toc}
</div>

### Scenario

I'm currently working on building out [Ultimate Angular's](https://ultimateangular.com) platform, and as such have these routes (in the URL, not my routing definitions) - with us assuming `1234` is our dynamic `:id`:

```
/admin/courses/1234/metadata
/admin/courses/1234/curriculum
/admin/courses/1234/prices
/admin/courses/1234/coupons
```

What this structure allows me to do is click a particular "course" from `/admin/courses`, and be navigated to the `/1234/metadata` view where I can edit the course's name, author and so forth.

### Parent Route Param subscriptions

What I needed to do is (based on the parent's param), is go off to my `Store` and fetch the appropriate course I'm viewing.

First let's walk through the routes:

```js
// routes
export const ROUTES: Routes = [
  {
    path: '',
    canActivate: [CoursesGuard],
    component: CoursesComponent,
  },
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

When my path is `''`, I use the `CoursesComponent`, and when we navigate to (for example `/courses/1234/`) - the `CourseComponent` is loaded (non-plural version). At this point, this `CourseComponent` is simply a container which renders a `<router-outlet>`. Once we hit that `:id`, we're directed to the first _child_ route, `metadata`.

Let's look what's inside the `CourseMetadataComponent`, paying attention to `ActivatedRoute` and the `course$` Observable:

```js
// ... other imports etc.
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'course-metadata',
  styleUrls: ['course-metadata.component.scss'],
  template: `
    <div class="course-metadata">
      {% raw %}{{ course$ | async | json }}{% endraw %}
    </div>
  `,
})
export class CourseMetadataComponent {
  course$ = this.router.parent.params.switchMap(params => {
    return this.store
      .select(getCoursesState)
      .map((courses: any) => courses.courses.find(course => course._id === params.id));
  });

  constructor(
    private router: ActivatedRoute,
    private store: Store<CoursesState>
  ) {}

}
```

At this point, the `switchMap` is given the `params` Object, which as you can see is driven from `this.router.parent.params` - which you may have used this before (without the `.parent`):

```js
// not the parent, it's the current param
course$ = this.router.params.switchMap(params => {
    // do something with local "params"
});
```

So what is `params` when we do this?

```js
course$ = this.router.parent.params.switchMap(params => {
  // params = { id: 1234 }
});
```

In my case, with `/admin/courses/1234/metadata`, the `params` on the parent are in fact an Object containing `{ id: 1234 }`.

This means from `*/1234/metadata`, `*/1234/curriculum`, `*/1234/prices` and `*/1234/coupons` - I have access to `1234` for passing off into `ngrx/store` to fetch the correct course. The courses are always available when hitting these routes because I'm using:

```js
{ canActivate: [CoursesGuard] }
```

This way, it'll make an API request if `ngrx/store` doesn't contain any `courses`, otherwise it'll pass the API request and head straight across to the correct course, pulling the data from the pre-populated store.