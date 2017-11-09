---
layout: article
heroImage: /posts/sails-pipe/hero-SailsPipe.jpg?v=1
thumbnail: /posts/sails-pipe/share-SailsPipe.jpg
title: "Centralize Your Sails.js Sockets with a Pipe"
lightNav: true
date: 2017-06-12 10:00:00 -0500
---
{:.intro}
When building a realtime application, [Sails.js integration with Socket.io](https://sailsjs.com/documentation/reference/web-sockets/socket-client) can make your like a lot easier. Sails does a lot to help set up web socket connections and streamline model update notifications to connected users, but being a Back End framework, it doesn't offer a whole lot in the way of organizing those updates on the front end. But fear not! You can easily take control of overzealous and scattered socket handlers with a simple pipe!

A couple years ago while watching the Overwatch developer panel at [Blizzcon](https://blizzcon.com/) one of the developers talked about organizing client updates through a game "pipe". The pipe was always there, streaming game data from the servers, and if a part of the interface needed some part of the data stream they would put in a "tap" and get notifications for that type of data.

At the time I had been struggling with organization of socket calls in my [game table application](/portfolio/game-table.html), and desperately wanted a way to clean up some very unreadable code...

{% highlight javascript %}
// listen for game updates
io.socket.on('game', function (message) {
  if (socketHandler.isValidMessage(message, self.game.id)) {
    if (socketHandler.player.hasOwnProperty(message.data.type)) {
      socketHandler.player[message.data.type](self.game, message.data.data, self.user);
    } else if (socketHandler.game.hasOwnProperty(message.data.type)) {
      socketHandler.game[message.data.type](self.game, message.data.data);
    } else if (socketHandler.gameLog.hasOwnProperty(message.data.type)) {
      socketHandler.gameLog[message.data.type](self.gameLog, message.data.data);

      // if this is a chat log message, adjust scrolling appropriately
      if (message.data.type === 'newLogMessage' && self.isScrolledToBottom) {
        Vue.nextTick(self.scrollChatToBottom);
      }
    }
  }
});
{% endhighlight %}

It worked, but it was terrible. Just a series of if/else statements designed as a catch all for the disparate types of data I might bring back from the back end. What made things even worse was that I had no good place to stick the socketHandler code, so it ended up living in it's own file outside of the Vue.js component I was using for the application logic, making context a tedious thing to keep track of and complicating testing.

The next best solution, however, was to start spreading socket handlers throughout my scripts and handle them all individually. That might be more readable, but it's not very DRY. So I decided to give the overwatch pipe a try. After some tears and refactoring my application logic was much better organized and far more readable.

{% highlight javascript %}
// Set up the GamePipe
var GamePipe = new Pipe('game');

GamePipe.on('playerRequestedJoin', this.playerRequestedJoin);
GamePipe.on('playerJoinApproved', this.playerJoinApproved);
GamePipe.on('playerJoinDeclined', this.playerJoinDeclined);
GamePipe.on('mapAdded', this.mapAdded);
GamePipe.on('mapRemoved', this.mapRemoved);
GamePipe.on('mapUpdated', this.mapUpdated);
// etc ...
{% endhighlight %}

Much better!

Context is easier to keep track of. Responses from the back end are passed to the pipe handler directly allowing for individualized reactions to response data without the nightmarish set of if/else statements. And now game logic can logically live within the game component instead of a separate file, so testing becomes much simpler.

## Project Setup

To show how a pipe can be built, let's build a simple todo list app. In doing so, we'll combine a couple different types of socket updates together to help show the usefulness of the pipe architecture. If you'd like to skip ahead and just grab the code, [click here](#pipe_class)!

To start, we need a new sails app. While we're at it let's go ahead and add a collection.

{% highlight bash %}
sails new todo-pipe
cd todo-pipe
sails generate api todo
{% endhighlight %}

Just a couple more housekeeping items to take care of, open up `/config/models.js` and uncomment line 30, `migrate: 'alter'`. Otherwise, sails will badger you about that setting every time you lift the application. You can connect to a database if you like, but that is beyond the scope of this tutorial. This tutorial will be using the default file storage for our collections.

One last thing before we get going is to overwrite `/views/homepage.ejs` with the following markup:

{% highlight html %}
<style>
  .content { max-width: 1280px; margin: 30px auto 0; padding: 0 20px; }
  .menu a.checked { text-decoration: line-through; }
</style>

<div class="content">
  <h1>TODO</h1>

  <form class="js-add-item">
    <label>New Item
      <input type="text" id="item_name">
    </label>
  </form>

  <ul class="vertical menu" class="js-item-list"></ul>
</div>

<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/foundation/6.4.3/css/foundation.min.css" />
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.2.1/jquery.min.js"></script>
{% endhighlight %}

This sets us up with [Foundation](https://foundation.zurb.com/sites/docs/) for some quick CSS, some markup to key off of, and [jQuery](http://api.jquery.com/) to build some interactions. Typically you'll want to use a pipe with a front end framework like Vue.js or React to better organize your application, but for this example we'll be using jQuery to get things up and running with a little less boilerplate.

At this point we should be able to lift the application by running `sails lift` in the root of the project. You'll then be able to see our budding application at [http://localhost:1337/](http://localhost:1337/).

{:.note}
Whenever you make changes to any of the API files, you'll need to restart the app by hitting `ctrl+c` in the terminal and then running `sails lift` again.

## Basic Functionality

To get a good look at the before and after, let's build out the todo app with some simple jQuery callbacks.

Create a new file `/assets/js/app.js` and let's set up our basic todo application:

{% highlight javascript %}
(function ($, io) {
  var $itemName = $('#item_name');
  var $itemList = $('.js-item-list');
  var $addItemForm = $('.js-add-item');

  function addItemToDOM(item) {
    var itemClass = item.checked ? 'class="checked"' : '';
    $itemList.append(`<li><a href="#" ${itemClass} data-id="${item.id}">${item.name}</a></li>`);
  }

  // fetch existing items
  io.socket.get('/todo', function (response) {
    response.forEach((item) => addItemToDOM(item));
  });

  // add item
  $addItemForm.submit(function (event) {
    event.preventDefault();

    var newItem = $itemName.val();

    io.socket.post('/todo/create', { name: newItem, checked: false }, function (response) {
      addItemToDOM(response);
      $itemName.val('');
    });
  });

  // update item
  $itemList.on('click', 'a', function (event) {
    event.preventDefault();

    var $btn = $(event.currentTarget);
    var id = $btn.data('id');
    var checked = $btn.hasClass('checked');

    if (checked) {
      io.socket.post(`/todo/destroy`, { id }, function () {
        $itemList.find(`[data-id="${id}"]`).closest('li').remove();
      });
    } else {
      io.socket.post(`/todo/${id}`, { checked: true }, function (response) {
        var $item = $itemList.find(`[data-id="${id}"]`);
        if (response.checked === true) $item.addClass('checked');
      });
    }
  });
})(window.jQuery, window.io);
{% endhighlight %}

That's a good amount of code so let's go through what's happening so far.

First, we cache some jQuery objects so we can reference them easily. The `addItemToDOM` function will be a shortcut for adding new items to our todo list once they've been received from sails. Then we have handlers for submitting a new item, and updating or deleting an item. Pretty simplistic, but this will give us a couple different types of messages to differentiate between with our pipe later on.

So this application works well enough, but it would currently only work for a single user because we're handling all of our updates through post responses from the back end. In a realtime app this would only be half of the update lifecycle within the application. To make this a realtime app usable by more than one person, we need to move our update handlers out to `io.socket.on` callback functions.

Before we can do that, though, we'll need to set up custom controller actions that will dispatch socket messages to listeners, and subscribe the user to the model. Open up `/api/controllers/TodoController.js` and add the following actions to the controller:

{% highlight javascript %}
module.exports = {

  index(req, res) {
    Todo.find(function (err, items) {
      if (err) {
        res.error(err);
      } else {
        Todo.subscribe(req.socket, items);
        res.json(items);
      }
    });
  },

  create(req, res) {
    var name = req.param('name');

    Todo.create({ name: name }, function (err, item) {
      if (err) {
        res.error(err);
      } else {
        Todo.subscribe(req.socket, item);
        res.json(item);
      }
    });
  },

  update(req, res) {
    var id = req.param('id');
    var checked = req.param('checked');

    Todo.update(id, { checked: checked }, function (err, item) {
      if (err) {
        res.error(err);
      } else {
        res.json(item[0]);
      }
    });
  },

  destroy(req, res) {
    var id = req.param(id);

    Todo.destroy(id, function (err) {
      if (err) {
        res.error(err);
      } else {
        res.send(200);
      }
    });
  }

};
{% endhighlight %}

Now the app should work exactly the same as before, but we've set up subscriptions in the get and create methods so that all connected users will be able to listen for updates on the model. While Sails sets up blueprint routes for these operations for us, having these custom controller actions will allow us to dispatch nitifications in the next step.

{:.warning}
To set up watchers for adding items to the list, we would need to have a model to subscribe to that can push notifications to the user. For the sake of brevity, we'll be restricting this app to update and destroy notifications.

## Setting up Notifications

Now that we have a place on the backend actions to inject code, let's add some notifications. Open up `/api/controllers/TodoController.js` and update the `update` and `destroy` actions to look like this:

{% highlight javascript %}
module.exports = {
  // ...

  update(req, res) {
    var id = req.param('id');
    var checked = req.param('checked');

    Todo.update(id, { checked: checked }, function (err, item) {
      if (err) {
        res.error(err);
      } else {
        Todo.message(id, {
          type: 'itemUpdated',
          data: { item: item[0] }
        });
        res.json(item[0]);
      }
    });
  },

  destroy(req, res) {
    var id = req.param(id);

    Todo.destroy(id, function (err) {
      if (err) {
        res.error(err);
      } else {
        Todo.message(id, {
          type: 'itemDestroyed',
          data: { itemId: id }
        });
        res.send(200);
      }
    });
  }
};
{% endhighlight %}

We've added some notifications using the socket.io integration Sails ships with. The `Model.message()` method will dispatch an update to all connected sockets that have subscribed to the instance of the Todo model with an ID that matched the first argument passed to the `message()` function. The second argument passed to `message` is the data object that the socket client will receive on the front end. Let break down the object we're sending a little bit.

{% highlight javascript %}
{
  type: 'itemUpdated',
  data: { item: item }
}
{% endhighlight %}

Typically with Sails.js you might use some of the more specialized socket methods like `Model.update()` or `Model.destroy()`. however, these come with some additional baggage that we don't necessarily need, so we'll be using the more generic `Model.message()` to allow for a little more flexibility.

The object we're dispatching has a couple attributes, `type` and `data`.

You might think we only need the data, but in order to streamline all updates into a single pipe we need to be more consistent with the notifications we're sending. Since we'll also need some level of granularity to account for different types of responses I've packaged response specific data into the data property of the notification object. The type will come into play later on when we implement the Pipe class to wrangle our disparate notifications.

To make sure our updates will be received by users other than ourselves, we'll need to pull our callback handlers out of the `io.socket.post` callbacks and build out some `io.socket.on` handlers to update the page. This will allow application updates to come through our notifications instead of immediately after making a post request.

Switch over to `/assets/js/app.js` and let's post callbacks out into a socket update handler:

{% highlight javascript %}
// ...

// update item
$itemList.on('click', 'a', function (event) {
  event.preventDefault();

  var $btn = $(event.currentTarget);
  var id = $btn.data('id');
  var checked = $btn.hasClass('checked');

  if (checked) {
    io.socket.post(`/todo/destroy`, { id });
  } else {
    io.socket.post(`/todo/update`, { id: id, checked: true });
  }
});

function itemUpdated(item) {
  var $item = $itemList.find(`[data-id="${item.id}"]`);
  if (item.checked === true) $item.addClass('checked');
}

function itemDestroyed(id) {
  $itemList.find(`[data-id="${id}"]`).closest('li').remove();
}

// notifications
io.socket.on('todo', function (message) {
  if (message.data.type === 'itemUpdated') {
    itemUpdated(message.data.data.item);
  } else if (message.data.type === 'itemDestroyed') {
    itemDestroyed(message.data.data.itemId);
  }
});
{% endhighlight %}

So this is a step in the right direction. We've pulled the handlers out of the post callbacks and dropped then into a socket update handler. This means that if another user were to update any of these items, we would get a push notification from sails and be able to update our own DOM to reflect the updated todo data.

But you can already see how different types of requests can start contributing to difficult to read code. Imagine adding another half dozen handlers on the same model and the socket handler can quickly become bloated and difficult to maintain.

It is time at long last to clean this up with a pipe!

## new Pipe('model')

Let's add a new JavaScript file, `/assets/js/_Pipe.js`. I've prepended the underscore to make sure this file gets added to the DOM *before* `app.js`, but if you prefer, you can configure the asset ordering in `/tasks/pipeline.js`. Drop the following code into `_Pipe.js` and then restart your Sails app.

{% highlight javascript %}
class Pipe {
  constructor(model) {
    this.handlers = {};

    io.socket.on(model, (message) => {
      if (!message.data.type) {
        console.error('Socket message does not have a type');
        return;
      }

      if (this.handlers[message.data.type]) {
        this.handlers[message.data.type](message.data.data);
      }
    });
  }

  on(type, callback) {
    this.handlers[type] = callback;
  }
}
{% endhighlight %}

Let's break this down real quick...

The constructor takes a single string argument `model`. This is passed to the `io.socket.on` method much like we've been doing manually so far. The constructor will register a listener for the model that's passed in. Any socket messages received by the handler will be checked for type - which we added to all our `message()` calls on the back end. If a handler has been registered to the Pipe, it will pass the data received from the socket message to the handler's callback function.

To register a callback to the Pipe, we can call the `on()` method, which takes a `type` and `callback` method which are a string and function respectively.

Now that we have a pipe in place we can update our code to use the new class like so:

{% highlight javascript %}
function itemUpdated(data) {
  var $item = $itemList.find(`[data-id="${data.item.id}"]`);
  if (data.item.checked === true) $item.addClass('checked');
}

function itemDestroyed(data) {
  $itemList.find(`[data-id="${data.itemId}"]`).closest('li').remove();
}

// notifications
var todoPipe = new Pipe('todo');
todoPipe.on('itemUpdated', itemUpdated);
todoPipe.on('itemDestroyed', itemDestroyed);
{% endhighlight %}

So we've completely replaced the notification handling with calls to `Pipe.on`, making our handlers more consistent, and more readable. We've also updated the handler functions to assume we're receiving the raw response data for better consistency.

While this example app is not necessarily large enough to see some real optimization using this technique, any sails app that relies heavily on sockets could see a huge amount of optimization and code consolidation using this, or a similar technique. Not only can this help to clean up any number of socket handlers, but the Pipe class can now be used as a single point of configuration for any middleware you might want to add to your front end code like CSRF or JWT token management, or any number of third party authentication or API calls.

Like this solution or have a better one? Let me know in the comments below!
