---
layout: article
heroImage: /posts/sails-pipe/hero-SailsPipe.jpg?v=1
thumbnail: /posts/sails-pipe/share-SailsPipe.jpg
title: "Compile Your SVG Sprites With Gulp"
lightNav: true
date: 2017-12-10 10:00:00 -0500
---
{:.intro}
No matter what kind of app you're building, chances are you're going to need icons at some point. As of this writing, the best, most accessible method of working with icon sprites is to use SVG graphics. This can lead to a lot of tedious copy and paste of SVG code into a symbol library for optimization. Or... we could have gulp take care of that for us!

As you may remember from my previous post about using an [SVG icon store with Vue.js](/2017/05/01/svg-icon-store-with-vuejs.html), symbol libraries are convenient to use, and eliminate the need to download an extra sprite sheet [old school style](https://www.w3schools.com/css/css_image_sprites.asp). When writing the article for the Vue.js icon store, there ended up being a fair amount of copy/paste work and some custom editing of SVG files in a text editor. That's all well and good, but we can do better.

## Project Setup

Let's make a new project named **gulp-sprites** with the following structure:

{% highlight bash %}
gulp-sprites-demo
├── index.html
├── gulpfile.js
└── sprites
{% endhighlight %}

Nothing special or crazy, just a gulpfile, index.html for setting up our example, and a sprites folder to dump our sprites in. Speaking of sprites, head on over to [SimpleIcons.org](https://simpleicons.org/) and download a few icons to the sprites folder. Based on what I downloaded, my project now looks like this:

{% highlight bash %}
gulp-sprites-demo
├── index.html
├── gulpfile.js
└─┬ sprites
  ├── facebook.svg
  ├── instagram.svg
  ├── pinterest.svg
  └── twitter.svg
{% endhighlight %}

We'll also need to install some dependencies and then set up our **index.html** file with a basic boilerplate.

{% highlight bash %}
cd /path/to/project/root
npm init
npm install --save gulp gulp-svgstore gulp-inject
{% endhighlight %}

{% highlight html %}
<!-- index.html -->
<!DOCTYPE html>
<html>
  <head>
    <title>Compile Your SVG Sprites With Gulp</title>
  </head>
  <body>
    <!-- inject:svg --><!-- endinject -->
  </body>
</html>
{% endhighlight %}

## Compiling the Store

Now it's time to crack open that gulp file and compile those SVGs.

{% highlight javascript %}
var gulp = require('gulp');
var svgstore = require('gulp-svgstore');
var inject = require('gulp-inject');

gulp.task('default', function () {
  var svgs = gulp.src('sprites/*.svg').pipe(svgstore({ inlineSvg: true }));

  function fileToString(filePath, file) {
    return file.contents.toString();
  }

  return gulp.src('./index.html')
    .pipe(inject(svgs, { transform: fileToString }))
    .pipe(gulp.dest('./'));
});
{% endhighlight %}

So in our default task we are using **gulp-svgstore** to grab all of our icons from the **sprites** directory and parse them as inline SVGs. We can then pipe those inline SVGs into our inject code in **index.html**. Pretty sweet, right?

Now that we've got all of our icons compiling into our markup, they'll be available anywhere on the page through our auto-magic symbol library.

{% highlight html %}
<!-- index.html -->
<body>
  <!-- ... -->
  <svg>
    <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#facebook"></use>
  </svg>
  <svg>
    <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#instagram"></use>
  </svg>
  <svg>
    <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#pinterest"></use>
  </svg>
  <svg>
    <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#twitter"></use>
  </svg>
</body>
{% endhighlight %}

The biggest caveat here has to do with naming. Be default, **gulp-svgstore** will use the name of the SVG file as the ID of each individual sprite, which is why we've used the **#facebook** ID to reference the graphic in the **facebook.svg** file. For the full list of features, and many more configuration examples, you can check out the [gulp-svgstore Github page](https://github.com/w0rm/gulp-svgstore).

Want to DRY up your code a little more? Check out my article on [converting an SVG store to a Vue.js component](/2017/05/01/svg-icon-store-with-vuejs.html).

View the code on [Github](https://github.com/sstadt/gulp-sprites-demo).
