---
layout: project
title: D100 Guru
thumbnail: /images/projects/d100-guru/thumbnail-d100Guru.jpg
heroImage: /images/projects/d100-guru/hero-d100Guru.jpg
order: 13
lightNav: true
synopsis: A TTRPG tool aid for Game Masters, built on Nuxt and Firebase
---
{:.externals}
 - [Website](https://d100-guru.web.app/){:.button}
 - [Github](https://github.com/sstadt/d100-guru){:.button}

{:.intro}
D100 Guru was a short and sweet SPA built on Nuxt, aimed at centralizing random lists of things. What kinds of things? All kinds. You never know when you'll need a random name of an NPC, a rumor at a local Tavern, or an epitaph written on a tombstone while running an RPG. Sample lists can be found on the d100 subreddit, but those lists can be tedious to comb through. This app aimed to give Game Masters one click access to all kinds of random results through a single app.

{:.tech-stack}
 - Nuxt
 - Vue.js
 - Vuex
 - SCSS
 - Firebase

This SPA was built on Vue 2.x using the Nuxt framework, and Vuex to manage the data store. Though firebase can eb a challenge to learn, I was luckily able to take advantage of the native Nuxt-Firebase module to streamline development and maintain focus on building out the front end as much as possible. The project also served as an excellent opportunity to unify my styling conventions into a boilerplate that would serve me well on Shopify projects for years to come.

One of the biggest takeaways from this project was managing publicly shared items vs user owned items. Firebase has amazing tools for managing permissions, but pairing those with realtime watchers on data can be a pain. In the end when the implementation finally came together, it because an incredibly intuitive tool to use for these situations.
