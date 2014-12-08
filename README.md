Overview
========

Documentation on receiving and sending live events. A fully operational Intellifi setup generates a lot of informaton. It's important to us to inform you on the events that take place and how you can access them.

Spot protocol
=============

Events that happen at spot level are transferred to the server through HTTP. The spot protocol is used to encode the information. The server will be the Intellifi Brain in most cases. In most cases you don't need the know the technical details of this protocol. You can directly access all information at brain level. It's way easier and our advanced services take advantage of receiving events from multiple spots (i.e. the localisation service).

Message bus
===========

A message bus is used to propgate events through the Intellifi brain. All the services that we offer use the message bus to accquire their information. The output of the services is also placed on the message bus.

We have choosen MQTT as our protocol for accessing the message bus. This protocol is widly used and also accesible from smaller platforms. A lot of librarys and programs are avaialble. At this moment we use [RabbitMQ](http://www.rabbitmq.com/) as our broker.

Websocket
=========

We strongly believe in easy accessible events on all levels. That's why we build an websocket server that directly plugs in to the message bus. All messages can be accessed from within a browser. This makes it very easy to create interactive websites that support pushing.
