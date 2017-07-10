---
layout: post
permalink: /using-grunt-and-chrome-workspaces-for-in-browser-development/
title: Using Grunt and Chrome Workspaces for in-browser development
path: 2013-07-27-using-grunt-and-chrome-workspaces-for-in-browser-development.md
tags:
- Grunt
- Chrome
- JavaScript
- Performance
---

After some great interest in why I [ditched Sublime Text 2](//twitter.com/toddmotto/status/360672131710844929) for the day to fully code inside Google Chrome, here's the promised screencast on how to do it yourself.

I'm using Sass/SCSS (.scss) and JavaScript, both of which compile, concat + minify and livereload the browser, ice cool. In the video I'm using Grunt and Chrome's Workspaces to manage all of this, enjoy!

<div class="download-box">
  <a href="//github.com/toddmotto/workspaces-grunt">Files on GitHub</a>
  <a href="//github.com/toddmotto/workspaces-grunt/archive/master.zip">Download files</a>
</div>

Tip: if not default, change the video quality to 720p :)

<div class="screencast">
  <iframe width="420" height="315" src="//www.youtube.com/embed/tHhwJDuW73c" frameborder="0" allowfullscreen></iframe>
</div>

### Setting up Grunt
Workspaces don't require Grunt at all, it just saves me using software and refreshing the page myself, but Grunt is very flexible and reliable and I really advocate using it. It's a fairly simple learning curve if you're used to structuring JavaScript/JSON. If not, it's a very readable format that you'll pick up quickly anyway. Workspaces is very easy to setup, but here's how to do the Grunt bit.

First you'll need to setup [Grunt](//gruntjs.com) locally, which requires [Node.js](//nodejs.org) so make sure you've got that installed first. I love using [SourceTree](//sourcetreeapp.com) by Atlassian which has a neat 'Terminal' button that auto-locates your project so you don't have to change directory yourself (that's if you're using Git, anyway, if not slap your wrists).

Now, you'll want to do as I did in the video, and locate your project by using 'cd' (change directory) on the command line. Next, we need to install all project dependencies (internet connection required), which uses Node Package Manager (npm) to fetch. Once your Terminal is pointing at your project folder with your _package.json_ and _Gruntfile.js_ in, run the following:

{% highlight sh %}
npm install
{% endhighlight %}

This will then loop through your _package.json_ and install all the necessary stuff. If your permissions are uptight, you'll need to run the following instead (which you'll need to authenticate with a password):

{% highlight sh %}
sudo npm install
{% endhighlight %}

Once that's successfully downloaded all the dependency components, just run Grunt:

{% highlight sh %}
grunt
{% endhighlight %}

You'll then hopefully see the following:

{% highlight sh %}
Running "sass:dist" (sass) task

Running "uglify:dist" (uglify) task
File "dist/js/scripts.min.js" created.

Running "connect:livereload" (connect) task
Started connect web server on localhost:9000.

Running "open:server" (open) task

Running "watch" task
Waiting...
{% endhighlight %}

That's good news, you're good to go. Happy coding.

### Sourcemapping
One thing I didn't mention inside the video was Sass/SCSS sourcemapping (though sourcemapping is standalone tech and not limited to Sass itself). It essentially allows you to Inspect Element, and instead of seeing _style.min.css_ inside the developer tools, you'll actually going to drill down into the non-compiled Sass and you'll see something like __inputs.scss_! This is coming in the latest version of Sass but is available now on prerelease:

{% highlight sh %}
gem install sass --pre
{% endhighlight %}
