---
layout: project
title: Daggerstack
thumbnail: /images/projects/daggerstack/thumbnail-daggerstack.jpg
heroImage: /images/projects/daggerstack/hero-daggerstack.jpg
order: 17
lightNav: true
synopsis: A digital character sheet SPA for the Daggerheart system.
---
{:.externals}
 - [Website](http://daggerstack.com/){:.button}
 - [Github](https://github.com/sstadt/daggerstack){:.button}

{:.intro}
Daggerstack is a digital character sheet built for the Daggerheart TTRPG published by Darrington Press. It is built on Vue3 and Pinia, using the Nuxt3 framework with a Supabase back end. The website is super fast and performant, taking advantage of out-of-box SSR support in Nuxt3 to deliver flat HTML files that are hydrated after first page load. It also leveraged TailwindCSS to make styling quick and consistent across the app.

{:.tech-stack}
 - TailwindCSS
 - Vue3
 - Pinia
 - Supabase

Creating gaming tools and aids has always been a passion of mine. So when the folks at Critical Role released their own TTRPG alternative to the venerable Dungeons and Dragons, Daggerheart, I knew there was an opportunity to fill a brand new niche in the market. They had their own official digital character sheet on Demiplane already, but it was slow and cumbersome to use. And what's more, their cookie-cutter approach to character sheets didn't always honor the spirit or cadence it seemed had very deliberately been crafted for the system.

So I set to work building Daggerstack, a fast loading SPA that was aimed improving on what was currently available for Daggerheart digital character sheets. Supabase was chosen as a back end in an attempt to avoid previous headaches working with Firebase, and that choice was not in vain! Supabase made it super easy to store the JSON-style character sheets I had constructed during initial development and proved to be a perfect pairing for this JAMstack project.
