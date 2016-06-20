Push documentation
==================
Events are pushed as soon as they are available. In this repository we describe the different events. We assume that you already have knowledge about our [Web API](https://github.com/intellifi-nl/doc-webapi). You still will need the web API to request the initial state. Following the events will allow you to stay in sync.

Internally we are using a message bus to distribute events, we have create a standard [list with event types](mqtt_topics.md) that you should become familiar with.

Two types of push services are available: Webhooks and Websocket.

Webhooks
--------

We support [webhooks](http://en.wikipedia.org/wiki/Webhook) in our product. You can add your own end-point to our server and then we perform an HTTP request as soon as a certain event takes place. This would make it very easy to integrate services that don't need to much information.

The [subscriptions resource](https://github.com/intellifi-nl/webapi-doc/blob/master/resources.md#subscriptions) can be used to create/change and delete subscriptions for events you want to receive. You just need to enter your end-point and the topic filter that you want to use. Please note that this subscriptions resource also manages how long events are saved into the database. We always queue events into the internal database so that we can resend events in case your server is not reachable.

Websocket
---------

We strongly believe in easy accessible events on all levels. That's why we build an websocket server that directly plugs in to our server message bus. All messages can be accessed from within a browser. This makes it very easy to create interactive websites that support pushing.

We are using [socket.io](http://socket.io/) to accomplish this (a Node.js library).

You can send a `subscribe` command with a JSON object `{"topic_filter":"spots/#"}` to subscribe yourself to a specific topic on the message bus. The given example would show you all events that are directly transmitted by the SmartSpots. This filter string is formatted as a MQTT subscribe string (`/` for levels, `+` for level wildcard and `#` as 'all that follows' wildcard).

The events are send to you by a `event` command.

Example code
```javascript
var io = require('socket.io')(http);

// Subscribe for all spot events
io.emit('subscribe', `{"topic_filter":"spots/#"}`);

// Process event data
io.on('connection', function(socket){
  socket.on('event', function(data){
    console.log('event: ' + data);
  });
});
```
