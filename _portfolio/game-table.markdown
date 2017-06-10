---
layout: default
title: Game Table
thumbnail: /projects/game-table/share-GameTable.jpg
heroImage: /projects/game-table/hero-GameTable.jpg
headerImage: /projects/game-table/header-GameTable.png
lightNav: true
links:
  - name: Portfolio
    route: /portfolio
---
{:.externals}
 - [Demo Site](https://sw.scottstadt.com){:.button}
 - [Github](https://github.com/sstadt/sw-game-table){:.button}

{:.intro}
The Game Table application is a pet project of mine, but to support the FFG Star Wars RPG system. It's meant to be both functional and a playground to test out application architecture. I've incorporated a lot of what I've learned over the years into this application.

{:.tech-stack}
 - Grunt
 - Sails.js
 - Vue.js
 - VueMaterial
 - Karma/Jasmine
 - Socket.io

The system this game table is built to support is the Fantasy Flight Games Star Wars RPG system. This gives a rather narrow focus, but online alternatives for this system are few and far between. Those that do exist are buggy, hack on experiences, or have just terrible UX.

![Overwatch](//placehold.it/400x230){:.float-left}
One of my favorite parts about this project was inspired by the Overwatch developer panel at Blizzcon 2016. They spoke of a pipe of game data constantly streaming updates to the client, that you would tap to get a more specific set of application data to display or manipulate for the user.

This turned out the be analagous to the way sockets work in Sails.js. Centralizing the game data updates also lent itself well to the top-down data flow model used by front end frameworks these days. It also cleaned up my socket handler callbacks, which were getting a little ugly to be honest...
