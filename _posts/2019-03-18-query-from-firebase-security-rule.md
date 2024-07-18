---
layout: article
heroImage: /images/posts/firebase-security-query/header-FirebaseSecurityQuery.jpg
thumbnail: /images/posts/firebase-security-query/share-FirebaseSecurityQuery.jpg
title: "Query Firestore to Validate Security Rules"
lightNav: true
date: 2019-03-18 10:00:00 -0500
---
{:.intro}
Tools like Firebase help make the headless future a possibility for everyone, but exposing enough data to users to make a connection puts the onus of security on your Firestore database rules. If you're structuring your collections to be shallow like Firebase recommends, this can be a challenge. By using queries in your security rules, this problem is easily solved.

### Structuring the Data

Let's say we're building a Firestore app to store some game data for a roleplaying game. In this game our players can pick up quests, and we'll need to store all this data in Firestore. Actually building an app like this is beyond the scope of this article, but such an app might have the following collections in the database:

{% highlight bash %}
games
├── id
├── gameMaster
└── players

quests
├── id
├── gameId
├── title
└── description
{% endhighlight %}

We want to keep our collections shallow to improve query performance, so splitting quests out will suit our needs. The next step is to set up some basic security rules for these collection.

### Setting up Some Basic Security Rules

Open up the firebase console. Navigate to Database from the sidebar, and open the Rules tab. The basic idea will go something like this:

1. anyone can create a game
2. the players can view the game
3. the game master can edit and delete the game
4. the players can view quests associated with the game
5. the game master can add, edit, or delete quests associated with the game

Our first 3 objectives can be achieved easily enough with basic security rules.

{% highlight bash %}
service cloud.firestore {
  match /databases/{database}/documents {
    match /games/{gameId} {
      allow read: if request.auth.uid == resource.data.gameMaster || request.auth.uid in resource.data.players;
      allow update, delete: if request.auth.uid == resource.data.gameMaster;
      allow create;
    }
  }
}
{% endhighlight %}

### Adding Queries to Security Rules

For the last step we have a bit of a snag. Because our data has been kept shallow, we don't have any user IDs on the quest object. It would be easy enough to add the game master at the time of creation, but that doesn't present a solution for the game's players. We _could_ pass along the players as well, but this would create a data redundancy that would have to be maintained to avoid `quest.players` and `game.players` from falling out of sync.

That might be ok for your use case, but let's keep things simple and assume anyone who can view the game can view the quest. Let's start by adding a route for quests to the rules:

{% highlight bash %}
service cloud.firestore {
  match /databases/{database}/documents {
    match /games/{gameId} {
      allow read: if request.auth.uid == resource.data.gameMaster || request.auth.uid in resource.data.players;
      allow update, delete: if request.auth.uid == resource.data.gameMaster;
      allow create;
    }
    match /quests/{questId} {

    }
  }
}
{% endhighlight %}

In our rules for the `games` collection we check the current `uid` against values on the resource, but for quests we need to check values on a document in a different collection. While NoSQL databases don't have true relationships, for this app we're using `quest.gameId` as a bit of connective tissue between the two documents.

To do this we can make a query for a game with the matching ID from within the security rule, and then check the current user against the values stored in `game.gameMaster` and `game.players`. To keep the example as short as possible let's take a look at the create, update, and delete rule for game masters.

{% highlight javascript %}
allow create, update, delete: if get(/databases/$(database)/documents/games/$(resource.data.gameId)).data.gameMaster == request.auth.uid;
{% endhighlight %}

Firestore provides a handy `get()` function allowing you to retrieve another record from the database. In our query, we're using `resource.data.gameId` to query for the appropriate game document, and then comparing the `gameMaster` property against the current user just as we did with the game security rules. Don't worry too much about query performance; the responses are cached so even on compound queries with multiple reads on the same record you'll only be dinged for the first request.

Extrapolating on this, we can flesh out the rest of the security rules for quests in a similar fashion, checking again against the `gameMaster` and `players` values to determine permissions.

Our final set of rules look like this:

{% highlight bash %}
service cloud.firestore {
  match /databases/{database}/documents {
    match /games/{gameId} {
      allow read: if request.auth.uid == resource.data.gameMaster || request.auth.uid in resource.data.players;
      allow update, delete: if request.auth.uid == resource.data.gameMaster;
      allow create;
    }
    match /quests/{questId} {
      match /journalEntries/{entryId} {
      	allow read: if get(/databases/$(database)/documents/games/$(resource.data.gameId)).data.gameMaster == request.auth.uid || request.auth.uid in get(/databases/$(database)/documents/games/$(resource.data.gameId)).data.players;
				allow create, update, delete: if get(/databases/$(database)/documents/games/$(resource.data.gameId)).data.gameMaster == request.auth.uid;
      }
    }
  }
}
{% endhighlight %}

Happy app building!

Did I get something wrong? Are these instructions not working for you? Is there a way I can improve my security rules? I'd love to hear from you in the comments!
