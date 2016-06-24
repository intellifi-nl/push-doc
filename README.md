Push documentation
==================
Events are pushed as soon as they are available. In this repository we describe the different events. We assume that you already have knowledge about our [Web API](https://github.com/intellifi-nl/doc-webapi). You still will need the web API to request the initial state. Following the events will allow you to stay in sync.

Internally we are using a message bus to distribute events, we have create a standard [list with event types](mqtt_topics.md) that you should become familiar with.

Two types of push services are available: Webhooks and Websockets.

Webhooks
--------

[Webhooks](http://en.wikipedia.org/wiki/Webhook) are used to push event data from the Brain to third-party servers. The Brain will push the event data as a HTTP request as soon as the data is available.

The [subscriptions] (https://github.com/intellifi-nl/webapi-doc/blob/master/resources.md#subscriptions) resource can be used to create, update and delete subscriptions for events you want to receive. You just need to enter your server end-point in target_url and the topic filter that you want to use. Please note that this subscriptions resource also manages how long events are saved into our database. We always queue events into the internal database so that we can resend events in case your server is not reachable.

Websockets
---------

[Websockets](https://en.wikipedia.org/wiki/WebSocket) are commonly used to communicate between web server and web applications. The Brain offers the possibility to push events directly to web applications. 

There are several libraries available that can be used to implement Websockets in your web application, a few examples are listed below:

Javascript:
- [socket.io](http://socket.io/)

AngularJS:
- [btford.socket-io](https://github.com/btford/angular-socket-io) + socket.io ([example](http://briantford.com/blog/angular-socket-io))
- ngSocket

To subscribe to a specific event topic, emit a `subscribe` command with a JSON object `{"topic_filter":"items/#"}`. The given example would give you all item events. This filter string is formatted as a MQTT subscribe string (`/` for levels, `+` for level wildcard and `#` as 'all that follows' wildcard).

The events are emitted to you by the `event` command.

Example code:
```javascript
// javascript + socket.io

var socket = io('https://some.brain.com?key=<ApiKey>');

// Subscribe for all item events
socket.emit('subscribe', `{"topic_filter":"items/#"}`);

// Process event data
socket.on('event', function(data){
  console.log('event: ' + data);
});
```
