---
layout: post
permalink: /should-you-learn-angular-1-or-angular-2
title: "Should you learn Angular 1.x or 2?"
path: 2017-01-20-should-you-learn-angular-1-or-angular-2.md
---

A question that I'm frequently asked, "should I learn Angular 1 or 2?". I hope this post offers some insight, help and guidance into answering that question for you. No one can answer it directly, because it's specific to yourself, so here are some thoughts.

### Typical questions

Here are a few questions that I get asked a lot:

- _"I'm new to Angular, version 1 or 2?"_
- _"Should I learn Angular 2 yet?"_
- _"Should I learn Angular 1 and 2?"_

First thing to remember, there is no "official" answer. However, there are large factors that come into play when you are considering which framework to learn next or get better at. This next section is my aim to offer some advice on how to answer that question for yourself - as nobody else can for you.

### Your codebase and team

Regardless of whether you are working solo, or in a team, if you're currently in a job, here's some thoughts on answering the question.

Firstly, Angular 1.x is the dominant predator of the framework world, it's likely the biggest and most used framework currently out there being used in the wild. If you're in a job using 1.x, that is not an issue. If your company is building a single product/application for a customer, then you'll likely want to heads-down focus on your project deliverables and keep iterating your project.

Secondly, why would you want to rewrite those potentially years of hard work, just to upgrade your codebase? There are several factors that may come into play when answering this question. Angular 1.x has been around since 2009, and despite being an extremely productive and fast framework, there are several limitations that can hinder it's potential lifespan in your project. But this isn't something to hit the panic button about and move directly to Angular 2.

Regardless of how you decide, remember - you are essentially switching frameworks. It's not an "upgrade" or "migration" - you are moving to an entirely different codebase (either big bang style rewrite or incremental). A CEO/CTO is not going to simply want a rewrite without solid clarification as to why, and the business impact costs of doing so. For them, delivering for their customers are the key focus, not your version number.

#### Scenario: Single Product company

So what am I getting at? Let's take GitHub for example, and assume they are using Angular 1.x. The codebase may be several years old, but now they want to share their codebase to deploy to mobile, or distribute on desktop clients as well. They are stuck with a few options - such as writing standalone mobile applications (likely Android/iOS), and also building a native desktop client. These are skillset differences that require more monetary investment.

A business objective that, in my opinion, would encourage my employer to move to Angular 2, is to allow for all of the above to be done in a single framework. With Angular 2, you can compile to native mobile code with NativeScript, deploy to mobile with Ionic and use Electron for desktop. Single codebase.

But let's step back - what about your main web application? A single page app (SPA) can be fast if you're chunking your code, minifying and following great performance tactics and shipping content to users as fast as possible, however - it can be faster. Angular 2 can be server-side rendered (SSR) via Angular Universal, where Angular 1.x cannot. This is another selling point on whether you should learn Angular 2 or not.

#### Scenario: Multi-Product and Green Field

I've worked on both single product applications in Angular 1.x, as well as building several projects in 1.x for companies during the last few years - so I've had a taste of both sides of the coin. Let's assume that you have 10 great clients and 10 Angular 1.x apps, and client number 11 comes along offering an opportunity for a green field, fresh start product. What do you do?

For future-proofing, a great consideration at this point is Angular 2, because of some of the items listed above. Angular 2 was also rewritten from scratch and focuses on best practices that have been established through one-way dataflow and component architecture. You can do these things in Angular 1.x, however they have only (really) recently landed. Which means more refactoring of your existing codebase(s) to upgrade to using 1.6+ and the [.component](/exploring-the-angular-1-5-component-method/) API.

There is no doubt that Angular 1.x will be "discontinued" in the coming years, but then again - won't all of our apps and versions? You don't have to use the latest and greatest, but after 3 hard years Angular 2 finally shipped and it's incredible power is certainly worth investigating.

If client number 11 wants a new application from you, then go for it. But first, you need to understand things like [lifecycle hooks](/angular-1-5-lifecycle-hooks), [stateful and stateless](/stateful-stateless-components) components as well as [one-way dataflow and events](https://github.com/toddmotto/angular-styleguide#one-way-dataflow-and-events).

### You, personally

There are a few scenarios again here, and they all differ from person to person based on what _you_ are doing. There is no silver bullet answer.

#### Scenario: Employed using 1.x

If you're using Angular 1.x at work, it's likely you've already looked at Angular 2 and poked around the documentation and realised it is a different beast to Angular 1.x and the MVC patterns you were once used to. At this point - it is completely up to you. If you want to dive deeper into Angular 2 - go for it! If you don't, that's absolutely fine too. There is no point asking someone's advice on whether you should or shouldn't, unless you have a specific reason for the question (i.e. can I render on the server, or do XYZ?). It's like asking "Should I buy a Porsche or a Ferrari?" - the answer is in your head only.

This doesn't mean that outside of work - you cannot learn Angular 2 and bring a pitch to your boss if you wanted to move to Angular 2. Learn it in your own time and see if you like it - it's that simple.

#### Scenario: New to Angular

If you're completely new to Angular, this is a tricky question. Based on the above monolithic marketshare that Angular 1.x has, and company adoption rates of Angular 2 - you may have to learn both. If you learn Angular 1.x with the `.component()` API, and at least understand how to "go back to the MVC way", then you can walk into a job tomorrow with a company using Angular 1.x.

That being said - if you are looking for an "Angular 2 only" job, those are quite hard to come by right now, as per all the reasons listed above. If you are new to Angular, you may need to learn both, but again you'll have to decide that for yourself based on your own goals in life and your career.

Angular 2 is rapidly growing, and the growth is _phenomenal_, however this doesn't mean you can ping your new CV round a bunch of companies with just "Angular 2+" listed as a job spec. It's likely many companies still, and for years to come, will be using Angular 1.x. In this case, you need to decide what job you want, what skills you want, and where you want to be hired. I am generalising here a lot and assuming everyone reading this is "employed" - but there are many self-employed folks, like me, that also have to deal with this question.

If you're a self-employed engineer, you obviously need to fender for yourself and "put food on the table". Let's assume you have 50 requests for 1.x apps, and 1 request for Angular 2 apps - which are you going to focus more time on? Angular 1.x.

On the flip side, you may be in a position where you're getting 50/50 requests for new work, consultancy or whatever it is you're doing - and therefore you'll have to learn both. Most Angular 2 developers I personally know all know Angular 1.x as well, as we've been using it for years.

#### Scenario: Employed using something else

Perhaps you're using something like React, Ember, Backbone, Knockout for example, and you're considering Angular 2. Firstly, you need to investigate what Angular 2 will _give you_. Things like [Ahead-of-Time](https://angular.io/docs/ts/latest/cookbook/aot-compiler.html) compilation to drastically decrease the size of your codebase before you even ship to the browser and deploy your app are key considerations to look for with Angular 2.

### Closing thoughts

In a quick summary - it's all down to you and your job. Angular 2 jobs will be on the rise, and Angular 1.x jobs will still be around and rising no doubt, in fact the need for 1.x support in the enterprise (unless they decide to migrate) will be even higher.

The tl:dr; answer to this is if you're using Angular 1.x, consider your future goals for the project(s) and company. If you are new to Angular and want a job, you need to investigate the market and companies you're approaching and what tech stack they'll likely be using.

There is no right answer regardless, but hopefully this post has given you some decent insight as to what you need to consider. All the best with it!