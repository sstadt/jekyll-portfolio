---
layout: article
heroImage: /posts/svg-store/header-SVGStoreVue.jpg
thumbnail: /posts/svg-store/share-SVGStore.jpg
title: "Creating an SVG Icon Store with Vue.js"
date: 2017-05-01 10:00:00 -0500
---
{:.intro}
Creating an SVG icon store can help to give your site a custom feel, and eliminate extra http requests for potentially bulky icon library files. Paired with a component-based front end framework like Vue.js, you can exercise an amazing amount of control over your icon libraries while keeping your pages lean and fast.

SVG icon stores can be a streamlined alternative to the standard font library options. They can keep page weight down by eliminating an extra font library request, and retain infinite scalability by virtue of being a vector graphic. For this tutorial, we're going to be building an SVG icon library using Vue.js.

### Project Setup

First thing's first. Lets get a project set up. The fastest way to do this is to use [vue-loader](https://github.com/vuejs/vue-loader) to bootstrap a project. You'll need at least version 6+ of Node installed to use the webpack-simple template.

{% highlight bash %}
npm install -g vue-cli
vue init webpack-simple svg-store-demo
cd svg-store-demo
npm install
npm run dev
{% endhighlight %}

Now that we have a project set up, let's get rid of that `App.vue` file and then clean up `src/main.js`. Make sure you have executed the `npm run dev` command at the end; This will compile our assets, keep them up to date throughout the tutorial, and make the page available at [localhost:8080](http://localhost:8080).

{% highlight javascript %}
// src/main.js
import Vue from 'vue'

new Vue({
  el: '#app'
})
{% endhighlight %}

### Building the Icon Store

Looking good, but before we create an icon component we'll need to build out the SVG store. Start off with adding the svg wrapper to `./index.html` right at the top of the body tag.

{% highlight html %}
<!-- index.html -->
<body>
  <svg xmlns="http://www.w3.org/2000/svg" style="display: none;">
    <!-- We'll place our icons here -->
  </svg>

  <!-- ... -->
</body>
{% endhighlight %}

Now we need some icons. Head over to [Simple Icons](https://simpleicons.org/) and download a few icons. I'll grab the icons for Facebook, Twitter, and Youtube. You can also use your own SVG icons from Sketch or Illustrator.

Getting these icons into the store is really simple. All we have to do is open the SVG in a text editor and copy the shapes for the icon into our SVG store in `index.html`. We'll have to massage a few things into place, but the process is pretty simple.

Here is the SVG code for the Facebook icon:

{% highlight xml %}
<!-- Facebook.svg -->
<svg viewBox="0 0 16 16" xmlns="http://www.w3.org/2000/svg" fill-rule="evenodd" clip-rule="evenodd" stroke-linejoin="round" stroke-miterlimit="1.414">
  <path d="M15.117 0H.883C.395 0 0 .395 0 .883v14.234c0 .488.395.883.883.883h7.663V9.804H6.46V7.39h2.086V5.607c0-2.066 1.262-3.19 3.106-3.19.883 0 1.642.064 1.863.094v2.16h-1.28c-1 0-1.195.48-1.195 1.18v1.54h2.39l-.31 2.42h-2.08V16h4.077c.488 0 .883-.395.883-.883V.883C16 .395 15.605 0 15.117 0" fill-rule="nonzero"/>
</svg>
{% endhighlight %}

The first thing we need to do is change the SVG tag to a `symbol` tag. We will also strip out all of the attributes on the existing SVG with the exception of the viewbox attribute. You may need to leave the `fill-rule` property for some of the icons. You could also process the icons through Sketch or Illustrator to simplify the structure, but that is beyond the scope of this tutorial.

We'll also want to add an id attribute. I'll give it a value of `icon-facebook`. This will come into play when we build out the icon component. Once that's done we can copy the entire svg into our svg store in `./index.html`.

{% highlight html %}
<!-- index.html -->
<body>
  <svg xmlns="http://www.w3.org/2000/svg" style="display: none;">
    <symbol viewBox="0 0 16 16" id="icon-facebook">
      <path d="M15.117 0H.883C.395 0 0 .395 0 .883v14.234c0 .488.395.883.883.883h7.663V9.804H6.46V7.39h2.086V5.607c0-2.066 1.262-3.19 3.106-3.19.883 0 1.642.064 1.863.094v2.16h-1.28c-1 0-1.195.48-1.195 1.18v1.54h2.39l-.31 2.42h-2.08V16h4.077c.488 0 .883-.395.883-.883V.883C16 .395 15.605 0 15.117 0" fill-rule="nonzero"/>
    </symbol>
  </svg>

  <!-- ... -->
</body>
{% endhighlight %}

Now we have a good start on our icon store. Repeat this process for however many icons you would like to include in your store. Here is my completed icon store given the 3 SVG files I downloaded:

{% highlight html %}
<!-- index.html -->
<svg xmlns="http://www.w3.org/2000/svg" style="display: none;">
  <!-- Facebook -->
  <symbol viewBox="0 0 16 16" id="icon-facebook">
    <path d="M15.117 0H.883C.395 0 0 .395 0 .883v14.234c0 .488.395.883.883.883h7.663V9.804H6.46V7.39h2.086V5.607c0-2.066 1.262-3.19 3.106-3.19.883 0 1.642.064 1.863.094v2.16h-1.28c-1 0-1.195.48-1.195 1.18v1.54h2.39l-.31 2.42h-2.08V16h4.077c.488 0 .883-.395.883-.883V.883C16 .395 15.605 0 15.117 0" fill-rule="nonzero"/>
  </symbol>
  <!-- Twitter -->
  <symbol viewBox="0 0 16 16" id="icon-twitter">
    <path d="M16 3.038c-.59.26-1.22.437-1.885.517.677-.407 1.198-1.05 1.443-1.816-.634.37-1.337.64-2.085.79-.598-.64-1.45-1.04-2.396-1.04-1.812 0-3.282 1.47-3.282 3.28 0 .26.03.51.085.75-2.728-.13-5.147-1.44-6.766-3.42C.83 2.58.67 3.14.67 3.75c0 1.14.58 2.143 1.46 2.732-.538-.017-1.045-.165-1.487-.41v.04c0 1.59 1.13 2.918 2.633 3.22-.276.074-.566.114-.865.114-.21 0-.41-.02-.61-.058.42 1.304 1.63 2.253 3.07 2.28-1.12.88-2.54 1.404-4.07 1.404-.26 0-.52-.015-.78-.045 1.46.93 3.18 1.474 5.04 1.474 6.04 0 9.34-5 9.34-9.33 0-.14 0-.28-.01-.42.64-.46 1.2-1.04 1.64-1.7z" fill-rule="nonzero"/>
  </symbol>
  <!-- YouTube -->
  <symbol viewBox="0 0 16 16" id="icon-youtube">
    <path d="M0 7.345c0-1.294.16-2.59.16-2.59s.156-1.1.636-1.587c.608-.637 1.408-.617 1.764-.684C3.84 2.36 8 2.324 8 2.324s3.362.004 5.6.166c.314.038.996.04 1.604.678.48.486.636 1.588.636 1.588S16 6.05 16 7.346v1.258c0 1.296-.16 2.59-.16 2.59s-.156 1.102-.636 1.588c-.608.638-1.29.64-1.604.678-2.238.162-5.6.166-5.6.166s-4.16-.037-5.44-.16c-.356-.067-1.156-.047-1.764-.684-.48-.487-.636-1.587-.636-1.587S0 9.9 0 8.605v-1.26zm6.348 2.73V5.58l4.323 2.255-4.32 2.24z"/>
  </symbol>
</svg>
{% endhighlight %}

Awesome. Now we have 3 icons in our store that can be used anywhere on the page - all without loading an external font library. You can use these icons on the page right now by adding an SVG tag to the page with a reference to the ID of the symbol in your SVG store. To show the facebook icon, for example, you would drop the following onto your page:

{% highlight html %}
<!-- index.html -->
<svg>
  <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#icon-facebook"></use>
</svg>
{% endhighlight %}

### Creating the Icon Component

This is looking good, but that's a lot of markup to drop all over your app, and it's not very DRY. Well, it's better than pasting the full SVG path all over, but we can still do better. Let's build out our icon component.

Start by adding a new file: `src/icon.vue`

{% highlight html %}
<!-- src/icon.vue -->
<template></template>

<script>
  export default {}
</script>

<style lang="scss"></style>
{% endhighlight %}

The icon itself is actually pretty simple. We want to be able to provide it with a name and have that name link up to a symbol in our icon store. So we'll need a property for our icon name, and a computed value that will include the `icon-` prefix. Setting up our component this way allows us to namespace our icons to prevent collisions with other SVG images we may want to add to the store. Using a computed value will allow us to specify an icon without having to prefix our icon name every time we use an icon.

{% highlight html %}
<!-- src/icon.vue -->
<script>
  export default {
    props: ['name'],
    computed: {
      iconId() {
        return `#icon-${this.name}`;
      }
    }
  }
</script>
{% endhighlight %}

We'll also need to add our SVG code to the component template, using our computed `iconId` for the id attribute of the SVG element.

{% highlight html %}
<!-- src/icon.vue -->
<template>
  <svg>
    <use xmlns:xlink="http://www.w3.org/1999/xlink" :xlink:href="iconId"></use>
  </svg>
</template>
{% endhighlight %}

Our icon component should be functional now, so let's register it to Vue in `src/main.js`.

{% highlight javascript %}
// src/main.js
import Vue from 'vue'
import iconComponent from './icon.vue'

Vue.component('icon', iconComponent);

new Vue({
  el: '#app'
})
{% endhighlight %}

Now that Vue is aware of our component, we can add an instance of it to our markup. Since our Vue app is being initialized on the `#app` element, we can add our component anywhere inside that element and our icon should render.

{% highlight html %}
<!-- index.html -->
<div id="app">
  <icon name="facebook"></icon>
</div>
{% endhighlight %}

### Finishing Touches

We're almost done now, but there are a couple things we can add to give this just a little more polish. first off, this is a rather large icon, so let's beef up our properties with some validation and an extra size attribute.

{% highlight html %}
<!-- src/icon.vue -->
<template>
  <svg :height="size" :width="size">
    <use xmlns:xlink="http://www.w3.org/1999/xlink" :xlink:href="iconId"></use>
  </svg>
</template>

<script>
  export default {
    props: {
      name: {
        type: String,
        required: true
      },
      size: {
        type: Number,
        default: 20
      }
    },
    computed: {
      iconId() {
        return `#icon-${this.name}`;
      }
    }
  }
</script>
{% endhighlight %}

Perfect. Now we have a reasonably sized icon component linked up to an icon store, and we can grow or shrink our icons on an individual basis. An alternative to controlling the icon size through props, you could add a class to the SVG in the template and then control the size through component styling. And adding icons to the store is as easy as adding another symbol!

Like this post? Have a suggestion or improvement to the technique? Let me know in the comments below!

View the code on [Github](https://github.com/sstadt/svg-store-demo).
