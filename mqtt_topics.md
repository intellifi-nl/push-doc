MQTT Topics
===========

The whole Intellifi instracture is event driven in nature. All events are transfered by the message bus. The big advantage of this is that all messages are also avaialble to request from the message bus.

MQTT allows one the add a topic to each send message. This topic can be used to request the kind of messages that you would like to subscribe to. This is really needed when your setup grows. Thousands of events per second could flow through the message bus. I can't make up a reason to request them all in a single application.

In the rest of this document knowledge of MQTT is assumed.

Format
======

The MQTT topic is 

`resource`/`id`/`event`

The `resource` is the plural form of a noun, this is done so that we have the same naming scheme as in the [web API resources](https://github.com/intellifi-nl/doc-webapi#resources).

The `id` is the same id as in the resource that you could request with the web API. In most situations this is MongoDB id.

The `event` is a verb that represents the event that took place. i.e. `create`, or `request-complete`. A complete list of possible actions is given at resource level.

An event string may contain extra slashes. These allow you to make more specific subscriptions.

You can assume that the payload of MQTT messages is an utf8 encoded string with a JSON message. It's specified if this is different.

Resources
=========

Items
-----

Events are send when is changed inside the items resource.

* create: A new item is received in the system.
* update: An update has been done on the item. This is only send if the label or image where changed?
* location-update: location_now and/or location_last have been updated.
* hit/{spot-id}/{antenna-number (or id?!)}: Hit that was received on this item, spot antenna combination. Allows you to subscribe very specifically!
  * Example: items/48787f90s/hit/203/3

Spots
-----

Events are send when something changes in the spot resources, and when events are received on the spot.

* request-complete: Full HTTP post request has been 
* connect: Spot just came online and connected to the brain.
* disconnect: Spot went offline and does not have an active connection with the brain anymore.

The spot protocol events are directly transmitted on the message bus. Both the JSON as the PB protocol have their place in this definition. This will allow us to have a transistion period between the protocols. Please don't rely on these messages. They are retransmitted on their webAPI resourec as well. We may change these low level protocols without furher notice!
* json/`spot-resource`/`spot-action`
* json-to/`spot-resource`/`spot-action`
* pb: Direct binary messages as send in the new format. No JSON! The length specifier is not included. That already part of a MQTT message.
* pb-to: You also need to supply a valid encoded pb message.

Presences
---------

* create: new presence was created.
* proximity-change: proximity was changed
* delete: presence with given id is now deleted.

Todo
----

* Add more examples and elaborate a payload contents.
