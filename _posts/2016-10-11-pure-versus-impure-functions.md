---
layout: post
permalink: /pure-versus-impure-functions
title: Pure versus impure functions
path: 2016-10-11-pure-versus-impure-functions.md
tag: js
---


Understanding pure and impure functions is a simple transition into cleaner, more role-based and testable code. In this post we'll explore pure and impure functions by looking at a simple Body Mass Index (BMI) calculator that estimates your "healthy weight" by some simple input factors of height and weight. BMI isn't considered the most reliable tool for checking your weight, but that's not the point of this article ;)

<div class="toc" markdown="1">
<span class="gamma">Table of contents</span>
{:.no_toc}
* TOC
{:toc}
</div>

> Note: if you're not familiar with `kg` and `cm` units, use something like `70` for `kg` and `180` for `cm` to try it out.

### Terminology

Before we begin, let's clarify what "impure" and "pure" functions really mean in programming terms.

#### Impure functions

An impure function is a function that mutates variables/state/data outside of it's lexical scope, thus deeming it "impure" for this reason. There are many ways to write JavaScript, and thinking in terms of impure/pure functions we can write code that is much easier to reason with.

#### Pure functions

A pure function is much easier to comprehend, especially as our codebase may scale, as well as role-based functions that do one job and do it well. Pure functions don't modify external variables/state/data outside of the scope, and returns the same output given the same input. Therefore it is deemed "pure".

Let's refactor our BMI calculator that I've created in a fully impure fashion, into multiple functions that make use of pure functions.

### HTML and submit event

Here's the markup I've created to use for capturing the user's input data:

{% highlight html %}
<form name="bmi">
  <h1>BMI Calculator</h1>
  <label>
    <input type="text" name="weight" placeholder="Weight (kg)">
  </label>
  <label>
    <input type="text" name="height" placeholder="Height (cm)">
  </label>
  <button type="submit">
    Calculate BMI
  </button>
  <div class="calculation">
    <div>
      BMI calculation: <span class="result"></span>
    </div>
    <div>
      This means you are: <span class="health"></span>
    </div>
  </div>
</form>
{% endhighlight %}

And as a base, we'll attach an event listener as a base and `preventDefault` on the `submit` event:

{% highlight javascript %}
(() => {

  const form = document.querySelector('form[name=bmi]');

  const onSubmit = event => {

    event.preventDefault();

  };

  form.addEventListener('submit', onSubmit, false);

})();
{% endhighlight %}

The live output (which doesn't work yet) here:

<iframe width="100%" height="500" src="//jsfiddle.net/toddmotto/g5qthfb8/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### Impure implementation

We'll cut out the IIFE and event handler fluff for now and focus on the `onSubmit` function:

{% highlight javascript %}
const onSubmit = event => {

  event.preventDefault();

  let healthMessage;

  const result = form.querySelector('.result');
  const health = form.querySelector('.health');

  const weight = parseInt(form.querySelector('input[name=weight]').value, 10);
  const height = parseInt(form.querySelector('input[name=height]').value, 10);

  const bmi = (weight / (height /100 * height / 100)).toFixed(1);

  if (bmi < 18.5) {
    healthMessage = 'considered underweight';
  } else if (unit >= 18.5 && unit <= 25) {
    healthMessage = 'a healthy weight';
  } else if (bmi > 25) {
    healthMessage = 'considered overweight';
  }

  result.innerHTML = bmi;
  health.innerHTML = healthMessage;

};
{% endhighlight %}

That's all our function contains, and once you enter your height/weight it'll update the DOM with those results. Now, this is what I would personally consider a bit of a mess, and extremely difficult to debug and understand the role of the function. Let's explain what's happening here with some code comments:

{% highlight javascript %}
const onSubmit = event => {

  // prevent the form actually submitting
  event.preventDefault();

  // create a variable to wait and hold for our "health message"
  // which will be mutated and bound a new String with the correct message later
  let healthMessage;

  // grabbing both the result and health <span> tags to inject the results into
  const result = form.querySelector('.result');
  const health = form.querySelector('.health');

  // parsing to Integers with base 10, based on the weight and height <input> values
  const weight = parseInt(form.querySelector('input[name=weight]').value, 10);
  const height = parseInt(form.querySelector('input[name=height]').value, 10);

  // run the formula to obtain the BMI result
  // finally, use toFixed(1) for 1 decimal place
  const bmi = (weight / (height /100 * height / 100)).toFixed(1);

  // run the logic to see "how healthy" the person's weight is considered
  // this overrides the "healthMessage" variable based on the expression that passes
  if (bmi < 18.5) {
    healthMessage = 'considered underweight';
  } else if (unit >= 18.5 && unit <= 25) {
    healthMessage = 'a healthy weight';
  } else if (bmi > 25) {
    healthMessage = 'considered overweight';
  }

  // bind results to DOM
  result.innerHTML = bmi;
  health.innerHTML = healthMessage;

};
{% endhighlight %}

Upon first look, this is absolutely fine in terms of the fact that "it works". However if we began to scale this, we would end up with a monstrosity codebase with a bible of procedural code that is very easily broken.

We can do better, but here's the live demo for this implementation:

<iframe width="100%" height="500" src="//jsfiddle.net/toddmotto/3nnLwjn6/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

### Pure implementation

Before we can start using pure functions, we need to decide what functions will be pure. In the above and 100% impure implementation we did _way too many_ things in a single function:

* Read values from the DOM
* Parsed values to numbers
* Calculated the BMI from the parsed values
* Conditionally checked the BMI result and assigned the correct message to an undefined variable `healthMessage`
* Wrote values to the DOM

To "go pure", we'll implement functions that handle these actions:

* Parse values to numbers and calculate the BMI
* Return us the correct message for binding to the DOM

#### Going pure

Let's start with the input value parsing and calculating the BMI, specifically addressing this section of code:

{% highlight javascript %}
const weight = parseInt(form.querySelector('input[name=weight]').value, 10);
const height = parseInt(form.querySelector('input[name=height]').value, 10);

const bmi = (weight / (height /100 * height / 100)).toFixed(1);
{% endhighlight %}

This deals with `parseInt()` and the formula to calculate the BMI. This is not very flexible and likely very error prone when at some point in an application we'd come to refactoring or adding more features.

To refactor, we're only going obtain each input's value property alone, and delegate those into a `getBMI` function:

{% highlight javascript %}
const weight = form.querySelector('input[name=weight]').value;
const height = form.querySelector('input[name=height]').value;

const bmi = getBMI(weight, height);
{% endhighlight %}

This `getBMI` function would be 100% pure in the fact that it accepts arguments and returns a new piece of data based on those arguments. Given the same input, you'll get the same output.

Here's how I'd implement the `getBMI` function:

{% highlight javascript %}
const getBMI = (weight, height) => {
  let newWeight = parseInt(weight, 10);
  let newHeight = parseInt(height, 10);
  return (newWeight / (newHeight /100 * newHeight / 100)).toFixed(1);
};
{% endhighlight %}

This function takes the `weight` and `height` as arguments, converts them to Numbers through `parseInt` and then performs the calculation for the BMI. Whether we pass a String or Number as each argument, we can safety check and `parseInt` regardless here.

Onto the next function. Instead of `if` and `else if` logic to assign the `healthMessage`, we'll create the expected result to look like this:

{% highlight javascript %}
health.innerHTML = getHealthMessage(bmi);
{% endhighlight %}

Again, this is much easier to reason with. The implementation of `getHealthMessage` would look like this:

{% highlight javascript %}
const getHealthMessage = unit => {
  let healthMessage;
  if (unit < 18.5) {
    healthMessage = 'considered underweight';
  } else if (unit >= 18.5 && unit <= 25) {
    healthMessage = 'a healthy weight';
  } else if (unit > 25) {
    healthMessage = 'considered overweight';
  }
  return healthMessage;
};
{% endhighlight %}

Putting everything together, we have this:

{% highlight javascript %}
(() => {

  const form = document.querySelector('form[name=bmi]');

  const getHealthMessage = unit => {
    let healthMessage;
    if (unit < 18.5) {
      healthMessage = 'considered underweight';
    } else if (unit >= 18.5 && unit <= 25) {
      healthMessage = 'a healthy weight';
    } else if (unit > 25) {
      healthMessage = 'considered overweight';
    }
    return healthMessage;
  };

  const getBMI = (weight, height) => {
    let newWeight = parseInt(weight, 10);
    let newHeight = parseInt(height, 10);
    return (newWeight / (newHeight /100 * newHeight / 100)).toFixed(1);
  };

  const onSubmit = event => {

    event.preventDefault();

    const result = form.querySelector('.result');
    const health = form.querySelector('.health');

    const weight = form.querySelector('input[name=weight]').value;
    const height = form.querySelector('input[name=height]').value;

    const bmi = getBMI(weight, height);

    result.innerHTML = bmi;
    health.innerHTML = getHealthMessage(bmi);

  };

  form.addEventListener('submit', onSubmit, false);

})();
{% endhighlight %}

You can see how much clearer this becomes. It also means we can test the `getBMI` and `getHealthMessage` functions on their own, without any external variables being needed. This means our "impure" `onSubmit` function becomes much clearer and easier to extend, refactor without breaking any isolated pieces of logic that may have before relied on variables in the lexical scope(s).

### Final solution

The final output with a mix of impure and pure functions:

<iframe width="100%" height="500" src="//jsfiddle.net/toddmotto/gfru1y9b/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>
