MQTT Topics
===========

The whole Intellifi infrastructure is event driven in nature. All events are transfered by the message bus.

MQTT requires that a topic is added to each send message. This topic can be used to request the kind of messages that you would like to subscribe to. This is really needed when your setup grows. Thousands of events per second could flow through the message bus. I can't make up a reason to request them all in a single application.

In the rest of this document knowledge of MQTT is assumed.

Format
======

The format for an Intellifi MQTT topic string is 

* `resource-type`/`id`/`action`/`optional arguments`

The `resource-type` is the plural form of a noun, this is done so that we have the same naming scheme as in the [web API resources](https://github.com/intellifi-nl/doc-webapi#resources).

The `id` is the same id as in the resource that you could request with the web API.

The `action` is a verb that represents the event that took place. i.e. `create`, or `request-complete`. A complete list of possible actions is given at resource level. 

The `optional arguments` may be added and can contain multiple levels. All these levels can be used to subscribe. It depends on the given action which levels are avaialble. In events that come from the spot directly we always add the spot serial number i.e.

In most messages the payload of MQTT messages is an UTF-8 encoded JSON message. Every action is specified and will mention a possible different payload if it's different.

TODO: Can we always include a timestamp in the payload? In exactly the same way? Would be very handy for saving events into the database. How should we do this with pb events? You could also save the receive time into the database.

Resources
=========

Items
-----

Events are send when is changed inside the items resource.

* create: New item created. Only when the item has never been seen before.
* update: An update has been done on the item. This is only send if the label or image where changed?
* appear: An item is detected by our systems. It's now present.
* disappear: An item has not been detected by our systems, we really don't know where it currently is. It's not present anymore.
* location-update: `location_now` and/or `location_last` fields have been updated. The old values are also included in the change message.
* hit/`spot-id`/`antenna-number`: Hit that was received on this item, spot antenna combination. Allows you to subscribe very specifically!
  * Example: items/48787f90s/hit/203/3

Spots
-----

Events are send when something changes in the spot resources, and when events are received on the spot.

* request-complete/`spot-serial-number`: Full HTTP post request has been 
* connect/`spot-serial-number`: Spot just came online and connected to the brain.
* disconnect/`spot-serial-number`: Spot went offline and does not have an active connection with the brain anymore.

The spot protocol events are directly transmitted on the message bus. 
* json/`spot-serial-number`/`spot-resource`/`spot-action`
* json-to/`spot-serial-number`/`spot-resource`/`spot-action`
* pb/`spot-serial-number`: Direct binary messages as send in the new format. No JSON! The length specifier is not included. 
* pb-to/`spot-serial-number`: You also need to supply a valid encoded pb message.

#### Embedded events
Low level spot event are directly send over the message bus. Both the low level JSON and PB protocol have their place in this definition. This will allow us to have a transistion period between the protocols.

Two actions on the spots resource have been defined for these events: `from` is used for messages that flow from the embedded client to the server. `to` is defined for messages that are send to the client by the server (also called commands). More abstract we call this the `direction`.

The topic definition is: `direction`/`spot-serialnumber`/`encoding`(/`spot-json-resource`/`spot-json-action`)

The `spot-serialnumber` is always added as the first argument after the `direction`, we use it because the database id is not avaialble when a spot connects for the very first time. The `encoding` can either be `json` or `pb`. The `spot-json-resource` and `spot-json-action` are only added if `json` encoding is used.

Examples:
* JSON message received from spot 234: from/234/json/presence/create
* JSON message send to spot 234: to/234/json/config/update
* PB message received from spot 310: from/310/pb
* PB message send to spot 310: to/310/pb

**Important note!** Please try to avoid using these low level messages for integrations witht the brain, they may change without further notice. All important events are avaialble on one of the resources.

Presences
---------

* create: new presence was created.
* proximity-change: proximity was changed, should also include previous value in the payload so that we also can react on that.
* delete: presence with given id is now deleted.

Todo
----

* Add more examples and elaborate a payload contents.
