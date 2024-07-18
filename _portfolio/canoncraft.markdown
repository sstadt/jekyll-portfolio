---
layout: project
title: Canon Craft
thumbnail: /images/projects/canon-craft/thumbnail-canonCraft.jpg
heroImage: /images/projects/canon-craft/hero-canonCraft.jpg
order: 12
lightNav: true
synopsis: A TTRPG tool for tracking game and session notes
---
{:.externals}
 - [Website](https://canon-craft.web.app/){:.button}
 - [Github](https://github.com/sstadt/canon-craft){:.button}

{:.intro}
Canon Craft was an attempt at building a management tool for notes and worldbuilding whilte playing any TTRPG. While most platforms of this type focused on expansive lists of game mechanics and location names, I wanted Canon Craft to be focused on providing useful reference material and session recaps for players.

{:.tech-stack}
 - Nuxt
 - Vue.js
 - Vuex
 - SCSS
 - Firebase

The app was my first attempt at using Firebase for a back end, and as such required a lot of time and effort to get the pub/sub model of Firebase working properly with a Nuxt front end. In the end the effort was worth it, as it gave me a way to structure my notes and handouts for players at my table and gave them something to look forward to when I sent our the clarion call of "updates are up in canon craft!"

The feature build I ended up being proudest of was the custom linking built into the WYSIWYG. The app is capable of tracking and displaying info for things like NPCs, organizations, towns, all the things you would normally need to track data for in an RPG. But while in any WYSIWYG I build out autofill functionality that allows you to embed a link to one of those pieces of data within the text. Then, when rendered for reading, the embed created a link that, when clicked, would open up a sidebar containing the most recently saved info for that NPC, organization... or whatever other piece of data had been linked. Players loved it because it kept relevant data out of sight, but easy to reference without losing your place in the recap.
