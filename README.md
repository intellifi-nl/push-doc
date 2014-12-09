Push documentation
==================

We offer a product that allows you to 'sense' events arround you. It is our strong believe that this should be a fast and smooth experience. If you bring an item to one of our antennas then you should immediatly see a response. We try to keep to overall latency lower than 100 milliseconds. All elements in the chain have been optimised to bring the event to you as fast as possible. We, and others, call this **push**. Events are pushed as soon as they are available. In this repository we describe all levels of events. We start with the spot protocol and end with the websocket protocol that transfers the event to your internet browser.

Spot protocol
-------------

Events that happen at spot level are transferred to the server through HTTP. The spot protocol is used to encode the information. The server will be the Intellifi Brain in most cases. 

You probably don't need the know the [technical details](spot_protocol.md) of this protocol. You can directly access all information at brain level. It's way easier and our advanced services take advantage of receiving events from multiple spots (i.e. the localisation service).

Message bus
-----------

A message bus is used to propgate events through the Intellifi brain. All the services that we offer use the message bus to accquire their information. The output of the services is also placed on the message bus.

We have choosen MQTT as our protocol for accessing the message bus. This protocol is widly used and also accesible from smaller platforms. A lot of librarys and programs are avaialble. At this moment we use [RabbitMQ](http://www.rabbitmq.com/) as our broker.

You can connect to this message bus with your own applicaton if you wish.

TODO: Elaborate on this!

Websocket
---------

We strongly believe in easy accessible events on all levels. That's why we build an websocket server that directly plugs in to the message bus. All messages can be accessed from within a browser. This makes it very easy to create interactive websites that support pushing.

TODO: Elaborate on this!
