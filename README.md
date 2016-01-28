Push documentation
==================

We offer a product that allows you to 'sense' events arround you. It is our strong believe that this should be a fast and smooth experience. If you bring an item to one of our antennas then you should immediatly see a response. We try to keep to overall latency lower than 100 milliseconds. All elements in the chain have been optimised to bring the event to you as fast as possible. We, [and others](http://en.wikipedia.org/wiki/Push_technology), call this **push**. 

Events are pushed as soon as they are available. In this repository we describe the different events. We assume that you already have knowledge about our [Web API](https://github.com/intellifi-nl/doc-webapi). You still will need the API to request the initial state. Following the events will allow you to stay in sync.

Internally we are using a message bus to distribute events, we have create a standard [list with event types](mqtt_topics.md) that you should become familiar with.

Webhooks
--------

We support [webhooks](http://en.wikipedia.org/wiki/Webhook) in our product. You can add your own end-point to our server and then we perform an HTTP request as soon as a certain event takes place. This would make it very easy to integrate services that don't need to much information.

We where inspired by the [resthooks](http://resthooks.org/) guidelines. We have created an WebAPI endpoint that manages which events you want to receive. The [subscriptions resource](https://github.com/intellifi-nl/webapi-doc/blob/master/resources.md#subscriptions) can be used to create/change and delete subscriptions. You just need to enter your end-point and the topic filter that you want to be used. Automated or by hand. Please note that this subscriptions resource also manages how events are saved into the database. We always queue the events into the internal database as well so that we can resend events in case your server is not reachable.

A nice thing to notice is that it's not very difficult to couple our product to [ifft](https://ifttt.com/recipes) or [zapier](https://zapier.com/), given the 

Websocket
---------

We strongly believe in easy accessible events on all levels. That's why we build an websocket server that directly plugs in to the message bus. All messages can be accessed from within a browser. This makes it very easy to create interactive websites that support pushing.

We are using [socket.io](http://socket.io/) to accomplish this (a Node.js library).

You can send a `subscribe` command with a JSON object `{"topic_filter":"spots/#"}` to subscribe yourself to some topic on the message bus. The given example would show you all events that are directly transmitted by the spots. This filter string is formatted as a MQTT subscribe string (`/` for levels, `+` for level wildcard and `#` as 'all that follows' wildcard).

The events are send to you by a `event` command.
