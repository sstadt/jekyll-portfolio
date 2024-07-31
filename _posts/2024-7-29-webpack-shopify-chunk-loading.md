---
layout: article
heroImage: /images/posts/webpack-shopify-chunking/header-webpack-shopify-chunking.jpg
thumbnail: /images/posts/webpack-shopify-chunking/share-webpack-shopify-chunking.jpg
title: "Chunk Webpack modules for Shopify"
lightNav: true
date: 2024-07-29 10:00:00 -0500
---
{:.intro}
Webpack is a one of the current defacto build tools, and it's code splitting can do wonders for optimizing performance. Unfortunately Shopify doesn't play nice with this feature of Webpack out of the box. Read on to learn how to fix this...

## The Problem

While you can configure code splitting or asynchronous module loading in a Webpack build and then use those assets in a Shopify theme, Shopify caches these files on their end - and without teaching Shopify how to talk to Webpack, any future updates to a chunk will go un-busted when Shopify renders the page.

If you have recently tried to setup chunking with Shopify and can't figure out why your changes are not reflecting in production, maybe this article can help!

## The Standard Solve (And Pitfall)

Typically the workaround for this might involve allowing filenames to change, and rendering the appropriate name for the new file dynamically via the build tool modifying liquid files.

While this works, it tends to generate a lot of junk in the assets folder. You end up not only with `main.js`, but `main.123456.js`, `main.123457.js`, `main.123458.js`, etc. This can be overwhelming for new developers... and for admins that are dangerous enough to snoop through site files for a quick production fix.

![Sister of the seven on a shame walk]({{ site.assets_url }}/images/posts/webpack-shopify-chunking/shame.gif)

This bloat has no tangible value as these deprecated files would probably break the site if they were ever used, and it makes audits a nightmare since the folder is clogged with useless files.

## A Better Solution

There is a better way to bust the cache, one of the tried and true techniques for which is respected by even Shopify: arbitrary query parameters.

### Webpack Setup

First thing's first. We need to set up Webpack to chunk our files. This is simple enough:

{% highlight javascript %}
const path = require('path');

module.exports = {
  output: {
    path: path.resolve(__dirname, 'assets'),
    filename: '[name].js',
    chunkFilename: '[name].js?[chunkhash:5]',
  },
  optimization: {
    chunkIds: 'named',
    runtimeChunk: 'single',
    splitChunks: {
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendor',
          enforce: true,
          chunks: 'all',
        }
      }
    }
  },
  // ... additional webpack config
};
{% endhighlight %}

This sets up the Webpack build to properly chunk out files, separate vendors assets (for more performant caching), and sets our output to the `assets/` directory.

The previously mentioned standard solve for setting up filenames is to use something like this: `[name].[chunkhash:5].js`. This is unfortunately what probably brought you to this article in the first place! This will generate a new file for each compilation and create a ton of clutter in the assets folder.

We don't want that.

So make sure your chunkhash is set up to look like a query string in the code sample above (with chunkhash AFTER `.js?`). That way, when Webpack queries for a chunk on the front end, it will include the freshly generated chunkhash from the build.

Files are still generated as usual, and then cached by Shopify. But since each new Webpack build generates a new set of chunkhash values for each file, the request to Shopify will include those new values. Since the file in question will not have been requested before (at least not with the new cache string), Shopify will have no cache for the file and serves it up fresh.

### Problem Solved!

Well... no, not quite.

While this addresses the Shopify cache problem, if you deploy this code to a theme your storefront JavaScript will almost certainly break. This is because Webpack doesn't know how to look for the chunks, so when it needs to load one it attempts to load the file from `assets/` (where the entry point file lives).

Shopify doesn't actually serve up any assets from the `assets/` folder on your storefront, it creates a cache folder within it's own file structure and serve up all your assets from there. So we need to tell Webpack about this folder and point it at the right place.

### Teaching Webpack and Shopify to Communicate

To do this, we need to grab the actual path to the assets folder and tell Webpack where to look. This can be done by rendering any asset in a `capture` block and extracting the path from that URL. The path can then be sent over to Webpack during runtime by setting the `__webpack_public_path__` window variable.

{% highlight html %}
{% raw %}
{%- capture entryFile %}{{ "main.js" | asset_url }}{% endcapture -%}
{%- assign assetsPath = entryFile | split: "main.js" | first -%}

<script>
  window.__webpack_public_path__ = "{{ assetsPath }}";
</script>
{% endraw %}
{% endhighlight %}

I like to do this on a snippet and then include it on `theme.liquid` near the opening `body` tag using a `render` tag.

Now that Webpack knows where to look for asset files things should all be working. Webpack can now properly chunk files (without needing a new filename), and Shopify will only serve up a cached version if the Webpack build has not run since the last time the page was loaded.

Now go forth and properly chunk out your webpack builds for Shopify!
