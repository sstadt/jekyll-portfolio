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
├── gameMaster
└── players

quests
├── id
├── ...
└── gameId
{% endhighlight %}

implement basic rules for collections

talk about strategies (adding players through cloud function, advantages/disadvantages)

implement query based rule for quests

{% highlight javascript %}
service cloud.firestore {
  function ownerOrPlayer() {
    return request.auth.uid == resource.data.gameMaster || request.auth.uid in resource.data.players;
  }
  match /databases/{database}/documents {
    match /games/{gameId} {
      allow read: if ownerOrPlayer();
      allow update, delete: if request.auth.uid == resource.data.gameMaster;
      allow create;
    }
    match /quests/{questId} {
      match /journalEntries/{entryId} {
      	allow read: if get(/databases/$(database)/documents/games/$(gameId)).data.gameMaster == request.auth.uid || request.auth.uid in get(/databases/$(database)/documents/games/$(gameId)).data.players;
				allow create, update, delete: if get(/databases/$(database)/documents/games/$(gameId)).data.gameMaster == request.auth.uid;
      }
    }
  }
}
{% endhighlight %}


View the code on [Github](https://github.com/sstadt/svg-store-demo).
