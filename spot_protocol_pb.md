PB Spot protocol
================

JSON is a very accessible encoding but it's also way more verbose than a binary protocol. In the [spot server communication](spot_protocol.pb) we will replace the JSON with [protocol buffers](https://developers.google.com/protocol-buffers/). On the higher layers JSON will keep it's important role offcourse.

Why?
----

* In the first place it's way faster to encode or decode a binary protocol in embedded software. You don't have to parse numbers, you can just copy it to your memory. 
* It's more robust because of the definition files (`.proto` files). All fields in the messages are defined in these seperate file. These files will be made avaialble inside this repository once ready. It's always possible to add new fields and every encoded field includes it's type. So it's not possible to misread them. Even for future versions.
* It's way more compact, especially in low bandwidth situations it's important to be carefull with the number of bytes that you transmit.

Optimizations
-------------

* We often send items, in presences and especially in hits (when enabled). We should replace an item that is already known with a short identifying code.
* Almost all events include a timestamp. This is unix time in milliseconds (the amount of milliseconds since 1970-1-1). Events are always send in order, so we could replace succeeding timestamps with the offset since the previous event. We will always reserve field number 9 for timestamps, so that this trick can be applied automatically in the send layer.

We will write a Node.js stream library that 'removes' these tricks directly on receiving the information.

Buffering
---------

We are often asked to keep an history of events in the event of a network loss. The server can then receive them when it's reachable again. We already prepared a lot to make this possible. It will be introduced when it's ready.
