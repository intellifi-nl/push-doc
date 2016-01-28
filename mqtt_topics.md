Event types
===========

The whole Intellifi infrastructure is event driven in nature. On this page you can see which events we support for this moment. We also explain how you can create your own MQTT topic filters (as used in the subscriptions resource and the websockets implementation).

MQTT
====

The Brain server is build with a microservice architecture in mind. Services communicate with each other over a internal message bus. MQTT is used as the transport protocol. Important to take away is that every event contains a topic string and payload. We allow you to access all the events in a number of ways. In bigger setups a lot of events are fired every second. It's best that you limit yourself to the events that you really need. All events can be filtered by their topic string. We allow you to use so called topic filters to express which events you want.

You don't need to know everything about MQTT, but it's important you understand how MQTT filters work. [This page](http://www.hivemq.com/blog/mqtt-essentials-part-5-mqtt-topics-best-practices) contains a good description on using topic filters.

Intellifi topic format
----------------------

All events inside the Intellifi eco systems use a default format for their topics.

* `resource-type`/`id`/`action`/`arguments`

The `resource-type` is the plural form of a noun, this is done so that we have the same naming scheme as in the [web API resources](https://github.com/intellifi-nl/doc-webapi#resources).

The `id` is the same id as in the resource that you could request with the web API.

The `action` is a verb that represents the event that took place. i.e. `created`, `moved-to`, or `completed-request`. Please note that this verb is always placed in simple past, an activitity in the past that is completed but still has influence on the current state. This also allows us to create real english sentences from an event. Something like: `Item324` `moved-to` `Room 3` at 12:01.

The `arguments` may be added and can contain multiple levels. All these levels can be used to subscribe. It depends on the given action which levels are avaialble. New arguments might be appended later on, please try to design your topic filters so that this won't be a problem. In most cases you can just append '/#' to your topic string. So if you wanted to subscribe to item created then you should: use `items/+/created/#' instead of 'items/+/created'.

In most messages the payload of MQTT messages is an UTF-8 encoded JSON object. New fields always can be added in the object, your application should be flexible about that.

Avaialble events
================

We have a very simple web page that you can use to discover events that flow through the sytem: https://brain.intellifi.nl/ws.html, in the background it's using the websocket pusher. Just describe yourself to '#' to see all events.

For changes to all resources we guarantee that you receive the following events, these events are send immediatly when you make changes on a resource with `POST`, `PUT` or `DELETE`:
* `resource-type`/`id`/created/`used-http-method`
* `resource-type`/`id`/updated/`used-http-method`
* `resource-type`/`id`/deleted/`used-http-method`

the `used-http-method` is optional, it's only send when something was changed using HTTP. Internally created, changed or deleted resources might add other arguments.

Items
-----

* items/`id`/created
  * Send when a new item is created, an item is only created when the brain has never seen it before.
  * `id` contains the id value of the newly created item. You can use this to lookup more information in the web API.
* items/`id`/appeared
  * The `is_present` field is set to true, this is done when an item has been detected by one of the SmartSpots.
* items/`id`/disappeared
  * The `is_present` field is set to false, this is done when the item has not been detected by any of the connected SmartSpots.
* items/`id`/moved-to/`location-id`
  * The `location_url` field of the item was changed to another value because the item moved to another location. 
  * The new location is included as the first argument `location-id`.

Spots
-----

Events are send when something changes in the spot resources, and when events are received on the spot.

* spots/`id`/connected/`spot-serial-number`
  * Spot just came online and connected to the brain.
* spots/`id`/disconnected/`spot-serial-number`
  * Spot went offline and does not have an active connection with the brain anymore.

Please note that we also have some low level messages avaialble. We do not advise you to use these because the internal spot protocol may change.

Presences
---------

* presences/`id`/created/`item-id`/`location-id`/`proximity`
  * New presence was created with given `id`.
  * `item-id` and `locaton-id` are added so that you now which item was detected at which location.
* presences/`id`/changed-proximity/`item-id`/`location-id`/`new-proximity`/`previous-proximity`/`direction`
  * The proximity of a presence changed.
  * The `new-proximity` and `previous-proximity` fields return a string with the proximity: `immediate`, `near` or `far`.
  * The `direction` gives a string that indicates if the signal got stonger or weaker: `moved-closer`, `moved-away`.
* presences/`id`/deleted/`item-id`/`location-id`/`proximity`
  *  The item is not detected anymore at the given location id.
  * `item-id` and `locaton-id` are added so that you now which item was detected at which location.
