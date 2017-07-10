---
layout: post
permalink: /creating-a-tabs-component-with-react/
title: Creating a tabs component with React
path: 2015-12-28-creating-a-tabs-component-with-react.md
tags:
- Components
- Classes
---

I have to say, this is my first proper component built in React. I tried it out last year and absolutely loved it. Feel free to rip this component apart, suggest best practices and improvements!

### Component design

First we'll want to "design" the markup. Obviously for this I'll be using the wonderful JSX syntax, so let's look at what we want to create (this would be used inside the `render` function so I've omitted the rest):

{% highlight html %}
<Tabs>
  <Pane label="Tab 1">
    <div>This is my tab 1 contents!</div>
  </Pane>
  <Pane label="Tab 2">
    <div>This is my tab 2 contents!</div>
  </Pane>
  <Pane label="Tab 3">
    <div>This is my tab 3 contents!</div>
  </Pane>
</Tabs>
{% endhighlight %}

This means we need a `Tabs` component and `Pane` child component.

### Tab Component

This component will do most of the leg work, so let's start by defining the Class:

{% highlight javascript %}
const Tabs = React.createClass({
  displayName: 'Tabs',
  render() {
    return (
      <div className="tabs"></div>
    );
  }
});
{% endhighlight %}

I've added the `displayName: 'Tabs'` to help with JSX's debugging (JSX will set this automatically but I've added it for clarity for the Component's name).

Next up I've added the `render` function that returns the chunk of HTML I need.

Now it's time to show the tab's contents passed through. I'll create a "private" method on the Class, it won't actually be private but its naming convention with the underscore prefix will let me know it is.

{% highlight javascript %}
const Tabs = React.createClass({
  displayName: 'Tabs',
  _renderContent() {
    return (
      <div className="tabs__content">
        {this.props.children}
      </div>
    );
  },
  render() {
    return (
      <div className="tabs">
        {this._renderContent()}
      </div>
    );
  }
});
{% endhighlight %}

I've then added the `{this._renderContent()}` call inside the `render` function to return my JSX.

At this point, all the tab contents gets pushed into the tab, so it's not actually working as we'd like it to. Next up is setting up the `_renderContent` method to take a dynamic child state using an Array index lookup using `[this.state.selected]`.

{% highlight javascript %}
const Tabs = React.createClass({
  displayName: 'Tabs',
  _renderContent() {
    return (
      <div className="tabs__content">
        {this.props.children[this.state.selected]}
      </div>
    );
  },
  render() {
    return (
      <div className="tabs">
        {this._renderContent()}
      </div>
    );
  }
});
{% endhighlight %}

Currently `this.state.selected` doesn't exist, so we need to add some default props and states:

{% highlight javascript %}
const Tabs = React.createClass({
  displayName: 'Tabs',
  getDefaultProps() {
    return {
      selected: 0
    };
  },
  getInitialState() {
    return {
      selected: this.props.selected
    };
  },
  _renderContent() {
    return (
      <div className="tabs__content">
        {this.props.children[this.state.selected]}
      </div>
    );
  },
  render() {
    return (
      <div className="tabs">
        {this._renderContent()}
      </div>
    );
  }
});
{% endhighlight %}

I've told `getDefaultProps` to give me the component defaults, and then I'm passing those defaults (or overwritten user options) to bind to the `getInitialState` returned Object. Using `state` allows me to mutate the local properties, as `props` are immutable.

One thing we want users to do is be able to pass in a default `selected` tab, this would be passed through an attribute as a Number.

Now the tab content is setup, we need to actually create the clickable tab links and bind the corresponding click events. Let's add another pseudo "private" method to the component called `_renderTitles`:

{% highlight javascript %}
const Tabs = React.createClass({
  ...
  _renderTitles() {
    function labels(child, index) {
      return (
        <li key={index}>
          <a href="#">
            {child.props.label}
          </a>
        </li>
      );
    }
    return (
      <ul className="tabs__labels">
        {this.props.children.map(labels.bind(this))}
      </ul>
    );
  },
  ...
  render() {
    return (
      <div className="tabs">
        {this._renderTitles()}
        {this._renderContent()}
      </div>
    );
  }
});
{% endhighlight %}

This one's a little more complex, it maps over the `this.props.children` Nodes and returns the relevant JSX representation of each clickable tab item.

So far each tab item is an `<a>` element, however no click events are bound. Let's bind them by adding a `handleClick` method, which uses `preventDefault()` to stop the `#` bouncing when clicked. Then I can update the selected item using `this.setState()` by assigning the clicked `index`.

{% highlight javascript %}
const Tabs = React.createClass({
  ...
  handleClick(index, event) {
    event.preventDefault();
    this.setState({
      selected: index
    });
  },
  ...
});
{% endhighlight %}

We can then bind this event listener in the JSX using `onClick={this.handleClick.bind(this, index, child)}`:

{% highlight javascript %}
const Tabs = React.createClass({
  ...
  _renderTitles() {
    function labels(child, index) {
      return (
        <li key={index}>
          <a href="#" 
            onClick={this.handleClick.bind(this, index)}>
            {child.props.label}
          </a>
        </li>
      );
    }
    return (
      <ul className="tabs__labels">
        {this.props.children.map(labels.bind(this))}
      </ul>
    );
  },
  ...
});
{% endhighlight %}

Using `this.handleClick.bind()` allows me to set the context of the `handleClick` function and pass in the `index` of the current mapped element.

This now works nicely, but I want to allow the `selected` tab to be highlighted using an `active` className:

{% highlight javascript %}
const Tabs = React.createClass({
  ...
  _renderTitles() {
    function labels(child, index) {
      let activeClass = (this.state.selected === index ? 'active' : '');
      return (
        <li key={index}>
          <a href="#" 
            className={activeClass}
            onClick={this.handleClick.bind(this, index)}>
            {child.props.label}
          </a>
        </li>
      );
    }
    return (
      <ul className="tabs__labels">
        {this.props.children.map(labels.bind(this))}
      </ul>
    );
  },
  ...
});
{% endhighlight %}

This ternary operator allows me to conditionally assign the `'active'` String as the className when the `this.state.selected` value is equal to the index of the currently clicked element. React takes care of the adding/removing classes for all Nodes for me which is fantastic.

Put together we have our completed `Tab` component:

{% highlight javascript %}
const Tabs = React.createClass({
  displayName: 'Tabs',
  getDefaultProps() {
    return {
      selected: 0
    };
  },
  getInitialState() {
    return {
      selected: this.props.selected
    };
  },
  handleClick(index, event) {
    event.preventDefault();
    this.setState({
      selected: index
    });
  },
  _renderTitles() {
    function labels(child, index) {
      let activeClass = (this.state.selected === index ? 'active' : '');
      return (
        <li key={index}>
          <a href="#" 
            className={activeClass}
            onClick={this.handleClick.bind(this, index)}>
            {child.props.label}
          </a>
        </li>
      );
    }
    return (
      <ul className="tabs__labels">
        {this.props.children.map(labels.bind(this))}
      </ul>
    );
  },
  _renderContent() {
    return (
      <div className="tabs__content">
        {this.props.children[this.state.selected]}
      </div>
    );
  },
  render() {
    return (
      <div className="tabs">
        {this._renderTitles()}
        {this._renderContent()}
      </div>
    );
  }
});
{% endhighlight %}

### Pane Component

The `Pane` component is much more simple, and simply passes the contents of the component into itself:

{% highlight javascript %}
const Pane = React.createClass({
  displayName: 'Pane',
  render() {
    return (
      <div>
        {this.props.children}
      </div>
    );
  }
});
{% endhighlight %}

### propTypes validation

React is absolutely fantastic with its debugging error messages, and we can improve that inline by using `propTypes` and the relevant validation of the type. Let's start with the tab component:

{% highlight javascript %}
const Tabs = React.createClass({
  ...
  propTypes: {
    selected: React.PropTypes.number,
    children: React.PropTypes.oneOfType([
      React.PropTypes.array,
      React.PropTypes.element
    ]).isRequired
  },
  ...
});
{% endhighlight %}

I've told React to throw an error if `selected` is not of type "Number", and if the Child nodes are not of type "Array" or "Element".

This means that if somebody passes a property in that gets bound to `this.props.selected` that isn't a Number, it'll throw an error. This allows us to use propery JavaScript Objects in attributes, hooray for that.

{% highlight html %}
// Errors
<Tabs selected="0">
  <Pane label="Tab 1">
    <div>This is my tab 1 contents!</div>
  </Pane>
  <Pane label="Tab 2">
    <div>This is my tab 2 contents!</div>
  </Pane>
  <Pane label="Tab 3">
    <div>This is my tab 3 contents!</div>
  </Pane>
</Tabs>

// Works
<Tabs selected={0}>
  <Pane label="Tab 1">
    <div>This is my tab 1 contents!</div>
  </Pane>
  <Pane label="Tab 2">
    <div>This is my tab 2 contents!</div>
  </Pane>
  <Pane label="Tab 3">
    <div>This is my tab 3 contents!</div>
  </Pane>
</Tabs>
{% endhighlight %}

I'm using JSX's `{}` syntax to ensure that plain JavaScript runs in between the braces.

Let's also add some validation to the `Pane` component:

{% highlight javascript %}
const Pane = React.createClass({
  ...
  propTypes: {
    label: React.PropTypes.string.isRequired,
    children: React.PropTypes.element.isRequired
  },
  ...
});
{% endhighlight %}

I'm telling React here that `label` is absolutely required and is a String, and that `children` should be an element and is also required.

### Render

Now for the cherry on top, let's render it to the DOM:

{% highlight javascript %}
const Tabs = React.createClass({
  displayName: 'Tabs',
  propTypes: {
    selected: React.PropTypes.number,
    children: React.PropTypes.oneOfType([
      React.PropTypes.array,
      React.PropTypes.element
    ]).isRequired
  },
  getDefaultProps() {
    return {
      selected: 0
    };
  },
  getInitialState() {
    return {
      selected: this.props.selected
    };
  },
  handleClick(index, event) {
    event.preventDefault();
    this.setState({
      selected: index
    });
  },
  _renderTitles() {
    function labels(child, index) {
      let activeClass = (this.state.selected === index ? 'active' : '');
      return (
        <li key={index}>
          <a href="#" 
            className={activeClass}
            onClick={this.handleClick.bind(this, index)}>
            {child.props.label}
          </a>
        </li>
      );
    }
    return (
      <ul className="tabs__labels">
        {this.props.children.map(labels.bind(this))}
      </ul>
    );
  },
  _renderContent() {
    return (
      <div className="tabs__content">
        {this.props.children[this.state.selected]}
      </div>
    );
  },
  render() {
    return (
      <div className="tabs">
        {this._renderTitles()}
        {this._renderContent()}
      </div>
    );
  }
});

const Pane = React.createClass({
  displayName: 'Pane',
  propTypes: {
    label: React.PropTypes.string.isRequired,
    children: React.PropTypes.element.isRequired
  },
  render() {
    return (
      <div>
        {this.props.children}
      </div>
    );
  }
});

const App = React.createClass({
  render() {
    return (
      <div>
        <Tabs selected={0}>
          <Pane label="Tab 1">
            <div>This is my tab 1 contents!</div>
          </Pane>
          <Pane label="Tab 2">
            <div>This is my tab 2 contents!</div>
          </Pane>
          <Pane label="Tab 3">
            <div>This is my tab 3 contents!</div>
          </Pane>
        </Tabs>
      </div>
    );
  }
});
 
ReactDOM.render(<App />, document.querySelector('.container'));
{% endhighlight %}

And of course the live demo:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/rzv6Lrjh/embedded/result,js/dark/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

By all means this is not a complete solution for someone to use in production, but could be adapted to suit for sure. Please feel free to fork/improve/share :)

P.S big thanks to [Ken Wheeler](//twitter.com/ken_wheeler) for letting me pester him with syntax and obscure questions.
