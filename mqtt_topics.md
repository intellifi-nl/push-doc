MQTT Topics
===========

The whole Intellifi infrastructure is event driven in nature. All events are transfered by the message bus.

MQTT requires that a topic is added to each send message. This topic can be used to request the kind of messages that you would like to subscribe to. This is really needed when your setup grows. Thousands of events per second could flow through the message bus. I can't make up a reason to request them all in a single application.

In the rest of this document knowledge of MQTT is assumed.

Format
======

The format for an Intellifi MQTT topic string is 

* `resource-type`/`id`/`action`/`arguments`

The `resource-type` is the plural form of a noun, this is done so that we have the same naming scheme as in the [web API resources](https://github.com/intellifi-nl/doc-webapi#resources).

The `id` is the same id as in the resource that you could request with the web API.

The `action` is a verb that represents the event that took place. i.e. `created`, `moved`, or `completed-request`. Please note that this verb is always placed in simple past, an activitity in the past that is completed but still has influence on the current state. This also allows us to create real english sentences from an event. Something like: `Item324` `moved` to `Room 3` at 12:01.

The `arguments` may be added and can contain multiple levels. All these levels can be used to subscribe. It depends on the given action which levels are avaialble. In events that come from the spot directly we always add the spot serial number as the first element.

In most messages the payload of MQTT messages is an UTF-8 encoded JSON message. Every action is specified and will mention a possible different payload if it's different.

TODO: Can we always include a timestamp in the payload? In exactly the same way? Would be very handy for saving events into the database. How should we do this with pb events? You could also save the receive time into the database.

Resources
=========

Items
-----

Events are send when is changed inside the items resource.

* created: New item created. Only when the item has never been seen before.
* updated: An update has been done on the item. This is only send if the label or image where changed?
* appeared: An item is detected by our systems. It's now present.
* disappeared: An item has not been detected by our systems, we really don't know where it currently is. It's not present anymore.
* moved or updated-location: `location_now` and/or `location_last` fields have been updated. The old values are also included in the change message.
* hit/`spot-id`/`antenna-number`: Hit that was received on this item, spot antenna combination. Allows you to subscribe very specifically!
  * Example: items/48787f90s/hit/203/3

Spots
-----

Events are send when something changes in the spot resources, and when events are received on the spot.

* connected/`spot-serial-number`: Spot just came online and connected to the brain.
* disconnected/`spot-serial-number`: Spot went offline and does not have an active connection with the brain anymore.

#### Embedded events
Low level spot event are directly send over the message bus. Both the low level JSON and PB protocol have their place in this definition. This will allow us to have a transistion period between the protocols.

Two actions on the spots resource have been defined for direct messages: `from` is used for messages that flow from the embedded client to the server. `to` is defined for messages that are send to the client by the server (also called commands). More abstract we call this the `direction`.

The topic definition is: `direction`/`spot-serialnumber`/`encoding`(/`spot-json-resource`/`spot-json-action`)

The `spot-serialnumber` is always added as the first argument after the `direction`, we use it because the database id is not avaialble when a spot connects for the very first time. The `encoding` can either be `json` or `pb`. The `spot-json-resource` and `spot-json-action` are only added if `json` encoding is used.

Examples:
* JSON message received from spot 234: from/234/json/presence/create
* JSON message send to spot 234: to/234/json/config/update
* PB message received from spot 310: from/310/pb
* PB message send to spot 310: to/310/pb

PB messages do no include a length prefix, you can use the length indication that is included in the MQTT protocol. One MQTT message always contains one PB message.

**Important note!** Please try to avoid using these low level messages for integrations witht the brain, they may change without further notice. All important events are avaialble on one of the resources.

### Alive messages
We also send out a message when a full HTTP post request has been received from a spot (action is called `completed-request`). This can be used to check if the spot is still alive. Please note that this is also internal information, you can use the earlier mentioned `connected` and `disconnected` to watch for changes in the spot status.

Example:
* HTTP post has been received from spot 312: completed-request/312

Presences
---------

* created: new presence was created.
* changed-proximity: proximity was changed, should also include previous value in the payload so that we also can react on that.
* deleted: presence with given id is now deleted.

Todo
----

* Add more examples and elaborate a payload contents.
