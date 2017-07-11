---
layout: post
permalink: /deprecating-the-switch-statement-for-object-literals/
title: Replacing switch statements with Object literals
path: 2014-07-17-deprecating-the-switch-statement-for-object-literals.md
tags:
- Opinions
- Style
- JavaScript
---

In many programming languages, the `switch` statement exists - but should it any longer? If you're a JavaScript programmer, you're often jumping in and out of Objects, creating, instantiating and manipulating them. Objects are really flexible, they're at the heart of pretty much everything in JavaScript, and using them instead of the `switch` statement has been something I've been doing lately.

### What is the switch statement?
If you've not used `switch` before or are a little unsure what it does, let's walk through it. What `switch` does is take input and provide an output, such as code being run.

Let's look at a usual `switch` statement:

{% highlight javascript %}
var type = 'coke';
var drink;
switch(type) {
case 'coke':
  drink = 'Coke';
  break;
case 'pepsi':
  drink = 'Pepsi';
  break;
default:
  drink = 'Unknown drink!';
}
console.log(drink); // 'Coke'
{% endhighlight %}

It's similar to `if` and `else` statements, but it should evaluate a single value - inside the `switch` we use a `case` to evaluate against each value.

When you start seeing lots of `else if` statements, something is likely wrong and generally you shoud use something like `switch` as it's more suited for the purpose and intention. Here's some `else if` abuse:

{% highlight javascript %}
function getDrink (type) {
  if (type === 'coke') {
    type = 'Coke';
  } else if (type === 'pepsi') {
    type = 'Pepsi';
  } else if (type === 'mountain dew') {
    type = 'Mountain Dew';
  } else if (type === 'lemonade') {
    type = 'Lemonade';
  } else if (type === 'fanta') {
    type = 'Fanta';
  } else {
    // acts as our "default"
    type = 'Unknown drink!';
  }
  return 'You\'ve picked a ' + type;
}
{% endhighlight %}

This implementation is too loose, there is room for error, plus it's a very verbose syntax to keep repeating yourself. There's also the room for hacks as you can evaluate multiple expressions inside each `else if`, such as `else if (type === 'coke' && somethingElse !== 'apples')`. The `switch` was the best tool for the job, albeit you need to keep adding `break;` statements to prevent cases falling through, one of its many issues.

### Problems with switch

There are multiple issues with `switch`, from its procedural control flow to its non-standard-looking way it handles code blocks, the rest of JavaScript uses curly braces yet switch does not. Syntactically, it's not one of JavaScript's best, nor is its design. We're forced to manually add `break;` statements within each `case`, which can lead to difficult debugging and nested errors further down the case should we forget! Douglas Crockford has written and spoken about it numerous times, his recommendations are to treat it with caution.

We often use Object lookups for things in JavaScript, often for things we would never contemplate using `switch` for - so why not use an Object literal to replace `switch`? Objects are much more flexible, have better readability and maintainability and we don't need to manually `break;` each "case". They're a lot friendlier on new JavaScript developers as well, as they're standard Objects.

As the number of "cases" increases, the performance of the object (hash table) gets better than the average cost of the switch (the order of the cases matter). The object approach is a hash table lookup, and the switch has to evaluate each case until it hits a match and a break.

### Object Literal lookups

We use Objects all the time, either as constructors or literals. Often, we use them for Object lookup purposes, to get values from Object properties.

Let's set up a simple Object literal that returns a `String` value only.

{% highlight javascript %}
function getDrink (type) {
  var drinks = {
    'coke': 'Coke',
    'pepsi': 'Pepsi',
    'lemonade': 'Lemonade',
    'default': 'Default item'
  };
  return 'The drink I chose was ' + (drinks[type] || drinks['default']);
}

var drink = getDrink('coke');
// The drink I chose was Coke
console.log(drink);
{% endhighlight %}

We've saved a few lines of code from the switch, and to me the data is a lot cleaner in presentation. We can even simplify it further, without a default case:

{% highlight javascript %}
function getDrink (type) {
  return 'The drink I chose was ' + {
    'coke': 'Coke',
    'pepsi': 'Pepsi',
    'lemonade': 'Lemonade'
  }[type];
}
{% endhighlight %}

We might, however, need more complex code than a `String`, which could hang inside a function. For sake of brevity and easy to understand examples, I'll just return the above strings from the newly created function:

{% highlight javascript %}
var type = 'coke';

var drinks = {
  'coke': function () {
    return 'Coke';
  },
  'pepsi': function () {
    return 'Pepsi';
  },
  'lemonade': function () {
    return 'Lemonade';
  }
};
{% endhighlight %}

The difference is we need to call the Object literal's function:

{% highlight javascript %}
drinks[type]();
{% endhighlight %}

More maintainable and readable. We also don't have to worry about `break;` statements and cases falling through - it's just a plain Object.

Usually, we would put a `switch` inside a function and get a `return` value, so let's do the same here and turn an Object literal lookup into a usable function:

{% highlight javascript %}
function getDrink (type) {
  var drinks = {
    'coke': function () {
      return 'Coke';
    },
    'pepsi': function () {
      return 'Pepsi';
    },
    'lemonade': function () {
      return 'Lemonade';
    }
  };
  return drinks[type]();
}

// let's call it
var drink = getDrink('coke');
console.log(drink); // 'Coke'
{% endhighlight %}

Nice and easy, but this doesn't cater for a "default" `case`, so we can create that easily:

{% highlight javascript %}
function getDrink (type) {
  var fn;
  var drinks = {
    'coke': function () {
      return 'Coke';
    },
    'pepsi': function () {
      return 'Pepsi';
    },
    'lemonade': function () {
      return 'Lemonade';
    },
    'default': function () {
      return 'Default item';
    }
  };
  // if the drinks Object contains the type
  // passed in, let's use it
  if (drinks[type]) {
    fn = drinks[type];
  } else {
    // otherwise we'll assign the default
    // also the same as drinks.default
    // it's just a little more consistent using square
    // bracket notation everywhere
    fn = drinks['default'];
  }
  return fn();
}

// called with "dr pepper"
var drink = getDrink('dr pepper');
console.log(drink); // 'Default item'
{% endhighlight %}

We could simplify the above `if` and `else` using the _or_ `||` operator inside an expression:

{% highlight javascript %}
function getDrink (type) {
  var drinks = {
    'coke': function () {
      return 'Coke';
    },
    'pepsi': function () {
      return 'Pepsi';
    },
    'lemonade': function () {
      return 'Lemonade';
    },
    'default': function () {
      return 'Default item';
    }
  };
  return (drinks[type] || drinks['default'])();
}
{% endhighlight %}

This wraps the two Object lookups inside parenthesis `( )`, treating them as an expression. The result of the expression is then invoked. If `drinks[type]` isn't found in the lookup, it'll default to `drinks['default']`, simple!

We don't _have_ to always `return` inside the function either, we can change references to any variable then return it:

{% highlight javascript %}
function getDrink (type) {
  var drink;
  var drinks = {
    'coke': function () {
      drink = 'Coke';
    },
    'pepsi': function () {
      drink = 'Pepsi';
    },
    'lemonade': function () {
      drink = 'Lemonade';
    },
    'default': function () {
      drink = 'Default item';
    }
  };
    
  // invoke it
  (drinks[type] || drinks['default'])();
    
  // return a String with chosen drink
  return 'The drink I chose was ' + drink;
}

var drink = getDrink('coke');
// The drink I chose was Coke
console.log(drink);
{% endhighlight %}

These are very basic solutions, and the Object literals hold a `function` that returns a `String`, in the case you only need a `String`, you _could_ use a `String` as the key's value - some of the time the functions will contain logic, which will get returned from the function. If you're mixing functions with strings, it might be easier to use a function at all times to save looking up the `type` and invoking if it's a function - we don't want to attempt invoking a `String`.

### Object Literal "fall through"

With `switch` cases, we can let them fall through (which means more than one case can apply to a specific piece of code):

{% highlight javascript %}
var type = 'coke';
var snack;
switch(type) {
case 'coke':
case 'pepsi':
  snack = 'Drink';
  break;
case 'cookies':
case 'crisps':
  snack = 'Food';
  break;
default:
  drink = 'Unknown type!';
}
console.log(snack); // 'Drink'
{% endhighlight %}

We let `coke` and `pepsi` "fall through" by not adding a `break` statement. Doing this for Object Literals is simple and more declarative - as well as being less prone to error. Our code suddenly becomes much more structured, readable and reusable:

{% highlight javascript %}
function getSnack (type) {
  var snack;
  function isDrink () {
    return snack = 'Drink';
  }
  function isFood () {
    return snack = 'Food';
  }
  var snacks = {
    'coke': isDrink,
    'pepsi': isDrink,
    'cookies': isFood,
    'crisps': isFood,
  };
  return snacks[type]();
}

var snack = getSnack('coke');
console.log(snack); // 'Drink'
{% endhighlight %}

### Summing up

Object literals are a more natural control of flow in JavaScript, `switch` is a bit old and clunky and prone to difficult debugging errors. Objects are more extensible, maintainable, and we can test them a lot better. They're also part of a design pattern and very commonly used day to day in other programming tasks. Object literals can contain functions as well as any other [Object type](//toddmotto.com/understanding-javascript-types-and-reliable-type-checking), which makes them really flexible! Each function in the literal has function scope too, so we can return the closure from the parent function we invoke (in this case `getDrink` returns the closure).

Some more interesting comments and feedback on [Reddit](http://www.reddit.com/r/javascript/comments/2b4s6r/deprecating_the_switch_statement_for_object).
