---
layout: article
heroImage: /posts/svg-store/header-SVGStoreVue.jpg
thumbnail: /posts/svg-store/share-SVGStore.jpg
title: "Query Firestore to Validate Security Rules"
date: 2017-08-15 10:00:00 -0500
---
{:.intro}
Tools like Firebase help make the headless future a possibility for everyone, but exposing enough data to users to make a connection puts the onus of security on your Firestore database rules. If you're structuring your collections to be shallow like Firebase recommends, this can be a challenge. By using queries in your security rules, this problem is easily solved.

Let's say we have a Firestore with the following collections:

{% highlight bash %}
games
├── id
└── players

quests
├── id
├── ...
└── gameID
{% endhighlight %}









{% highlight javascript %}
// src/main.js
import Vue from 'vue'
import iconComponent from './icon.vue'

Vue.component('icon', iconComponent);

new Vue({
  el: '#app'
})
{% endhighlight %}


View the code on [Github](https://github.com/sstadt/svg-store-demo).
