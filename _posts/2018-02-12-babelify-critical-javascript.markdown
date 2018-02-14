---
layout: article
heroImage: /posts/babelify-critical-css/header-KeepCalmAndBabel.jpg
thumbnail: /posts/babelify-critical-css/share-KeepCalmAndBabel.jpg
title: "Babelify Critical JavaScript with Gulp"
lightNav: true
date: 2018-02-12 10:00:00 -0500
---
{:.intro}
Inlining critical CSS and JavaScript can seriously improve time to first render for above the fold content, but current tooling for JavaScript has an achilles heel: ES2015+ syntax. Don't let that stop you from optimizing your page load __and__ using the new features during development. We can build it... we have the technology!

If you're concerned at all about page load speed you've probably heard of the term [Critical Rendering Path](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/). In a nutshell, the aim is to speed up initial page rendering by inlining certain assets instead of making external requests for them. There is typically a larger focus on CSS for inlining critical path assets, but for smaller sites you may be thinking of inlining your JavaScript as well. One of the best tools for doing this with Gulp is [gulp-inline-source](https://www.npmjs.com/package/gulp-inline-source).

As of the time of this writing, it's achilles heel is ES2015+ syntax. Let's set up a sample project and take a look:

{% highlight bash %}
gulp-inline-babelify-demo
├── index.html
├── gulpfile.js
└── script.js
{% endhighlight %}

These should be all the files we need for the demo, so let's initialize the project and install some modules...

{% highlight bash %}
cd /path/to/gulp-inline-babelify-demo
npm init -y
npm install --save gulp gulp-serve gulp-inline-source
{% endhighlight %}

Now that all the initial packages are set up, it's time to flesh out the basic app files. For the purpose of this demo, I'll build out a simple VanillaJS clock. This will give us enough JavaScript that we can optimize a bit with some of ES2015's syntactic sugar. To start with, we'll need some simple markup to attach to in **index.html**:

{% highlight html %}
<!-- index.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>Babelify Your Critical JavaScript with Gulp</title>
  </head>
  <body>
    <p>The current time is: <span class="now"></span></p>
    <script type="text/javascript" src="script.js" inline></script>
  </body>
</html>
{% endhighlight %}

This markup includes a `span.now` element that we can populate with the current time, and a reference to the script file. That file will look like this:

{% highlight javascript %}
// script.js
window.addEventListener('DOMContentLoaded', function () {
  var $el = document.querySelector('.now');

  setInterval(function () {
    var now = new Date();
    var time = now.getHours() + ':' + now.getMinutes() + ':' + now.getSeconds();

    $el.innerHTML = time;
  }, 1000);
});
{% endhighlight %}

Again, nothing too complicated here. When the DOM loads we grab a reference to `span.now` so we can populate it with the time. Then we set an interval and populate the current time on each cycle.

Since this is just a little bit of Javascript it's probably not work keeping it as a separate asset. We could just dump it into **index.html**, but that's no good for development. So let's set up gulp to run that task for us so we can have the best of both worlds.

{% highlight javascript %}
var gulp = require('gulp');
var serve = require('gulp-serve');
var inline = require('gulp-inline-source');

gulp.task('inline', function () {
  gulp.src('./index.html')
    .pipe(inline())
    .pipe(gulp.dest('./dist'));
});

gulp.task('serve', serve('dist'));

gulp.task('default', ['inline', 'serve']);
{% endhighlight %}

Now if you run gulp you should be able to view the clock in a browser at **localhost:3000**.

![The current time is: 20:49:11]({{ site.s3_url }}/posts/babelify-critical-css/interstitial-WorkingClock.jpg?v=1)

We can also see the result of the **inline** task by checking out the freshly compiled **dist/index.html**:

{% highlight html %}
<!-- dist/index.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>Babelify Your Inline JavaScript with Gulp</title>
  </head>
  <body>
    <p>The current time is: <span class="now"></span></p>
    <script type="text/javascript">window.addEventListener("DOMContentLoaded",function(){var e=document.querySelector(".now");setInterval(function(){var n=new Date,t=n.getHours()+":"+n.getMinutes()+":"+n.getSeconds();e.innerHTML=t},1e3)});</script>
  </body>
</html>
{% endhighlight %}

This is awesome. We've completely eliminated the asset request for the external JavaScript file without being forced to do JavaScript development in an HTML file.

But what if we want to use ES2015 syntax in our javascript file? For this file in particular I might swatch out some `var` keywords for `const`, use a [template literal](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) to generate the time, and replace the setInterval callback with an [arrow function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions). With those changes made, our **script.js** file now looks like this:

{% highlight javascript %}
window.addEventListener('DOMContentLoaded', function () {
  var $el = document.querySelector('.now');

  setInterval(() => {
    const now = new Date();
    const time = `${now.getHours()}:${now.getMinutes()}:${now.getSeconds()}`;

    $el.innerHTML = time;
  }, 1000);
});
{% endhighlight %}

This gives us all the sugary goodness of ES2015, but if we run `gulp` again we get an error:

{% highlight html %}
SyntaxError: Unexpected token: punc ())
{% endhighlight %}

The explanation for this is that `gulp-inline-source` is a thin wrapper for `inline-source`, and the `inline-source` package does not yet support ES2015+ syntax. Eventually these packages will be updated as new syntax becomes more widely adopted, but until then we need to transpile the code ourselves in a custom handler for `inline-source`. As it turns out, this is actually pretty straight forward.

Let's start by installing some babel packages for transpiling the JavaScript code.

{% highlight bash %}
npm install --save babel-core babel-preset-es2015 babel-preset-stage-0
{% endhighlight %}

The next step is adding our custom handler and passing that as an option to `gulp-inline-source`. The basic idea is that we want to intercept the content, transpile it to ES5, and then pass it along for the plugin to finish inlining the script. Here is the updated gulpfile:

{% highlight javascript %}
var gulp = require('gulp');
var serve = require('gulp-serve');
var inline = require('gulp-inline-source');
var babelCore = require('babel-core');

function transpileString(source, context, next) {
  if (source.fileContent && !source.content && (source.type == 'js')) {
    source.content = babelCore.transform(source.fileContent, {
      minified: true,
      comments: false,
      presets: ['es2015', 'stage-0']
    }).code;
    next();
  } else {
    next();
  }
}

gulp.task('inline', function () {
  gulp.src('./index.html')
    .pipe(inline({ handlers: [ transpileString ] }))
    .pipe(gulp.dest('./dist'));
});

gulp.task('serve', serve('dist'));

gulp.task('default', ['inline', 'serve']);
{% endhighlight %}

The `gulp-inline-source` plugin can take an array of handler functions to do a custom transform on items before they're injected into the markup. Taking advantage of this feature, you can pass a function that will check `source.type` to target a file type, and then use `babel-core` to so a custom transform on `source.content` before passing the results along for inlining in an HTML file.

Now that we have our custom handler, running `gulp` once again should successfully compile the script into our page. Easy, right?

View the code on [Github](https://github.com/sstadt/gulp-inline-babelify-demo).
