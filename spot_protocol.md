Spot protocol
=============

Technical description of communication that takes place between an Intellifi Spot and a server. Good knowledge of TCP/IP and [HTTP](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) is assumed in this document.

You don't need to read this page if you are using the Intellifi Brain to acquire your events.

Outbound strategy
-----------------

The Intellifi spots create and maintain their own TCP/IP connection to the configured server. This is an important advantage in networks that are connected to the internet. Most of these networks use a router to access the internet (more info: [NAT](http://en.wikipedia.org/wiki/Network_address_translation)). By default a router will not allow any inbound traffic to your network. Outbound traffic however is always allowed. The Intellifi Spots use an outbound socket connection to get through the router to the configured server. The fact that we use HTTP to upload the events also helps, it makes it even more unlikely that the traffic is blocked. It's exacltly the same mechamism as browsing the internet. As a result of these design choices wokring with Intellifi Spots is just a matter of plug and play.

It's obviously not required to connect to the server through the internet, you can also connect to a computer inside your local network. Our 'outband' strategy is also a big advantage on internal networks. You won't have to administer the ip addresses of your spots, they all will automatically connect to your server. 

Spot protocols
--------------

At this moment all spots use the [JSON spot protocol](spot_protocol_json.md), in the coming months we will switch to the way more effecient [PB spot protocol](spot_protocol_pb.md).

In the rest of the document we will refer to these protocols as the `spot protocol`. Both HTTP and Telnet are just a way to 'transfer' the protocol, so it doesn't matter if we are talking about JSON or PB. The examples will show the JSON protocol because it's human readable and because it's the only protocol that we have implemented at this very moment.

HTTP
----

HTTP comes with a lot of overhead if you would do a single post for every spot event (a spot can easily generate more than 100 events per second). The required headers for every post are responsible for this. That's why we encode the events with the spot protocol. It allows us to send multiple events in a single post action. We are basically wrapping the described spot protocol into HTTP.

We are using an extension of HTTP 1.1 to push events to the server as fast as possible. This is called [chunked encoding](http://en.wikipedia.org/wiki/Chunked_transfer_encoding). An HTTP post is started and kept 'open' for 1 second. Every event that is generated is directly written to the socket as a seperate chunk. The HTTP endpoint at the server is served by a [Node.js](http://nodejs.org/) application. Node.js has an event based architecture and directly provides us with events as chunks come in. As soon as the HTTP post is 'finished' by the spot the server can write the downstream events (or commands) to the spot (if any avaialble). This approach gives serious presendence to upstream events, for now this is a good thing. Almost all events are upstream. Pleae take a look at the last paragraph for planned improvements on this approach.

A big advantage of HTTP is that additional information can be encoded in the headers. TODO: Elaborate on protocl headers.

Example:
```HTTP
TODO
```

Telnet (debugging)
------------------

You may enable a telnet server on the spot for a direct connection on TCP port 23. This will allow you to receive the spot protocol on your terminal or in custom application. We do not advice you to use this option for production environments. It's not encrypted nor it's protected by a password. It also requires you to adminster all the ip addresses of your Spots. It's turned off by default.

Implementing SSH would solve some of the mentioned problems. At this moment we don't plan to do this. (As a consequence of our strong believe in the earlier mentioned 'outbound' strategy.)

Security
--------

HTTP is currently the default way of connecting to a server. It's a small step to run this over an SSL connection. This would encrypt all data between spot and server. We are planning to support SSL on spot level in the coming year. Experiments already have shown that this is a feasible step. For this moment we advice to run Intellifi Spots only inside a 'protected' network use.

Authenticaton is also beeing worked on, we are planning to solve this with known stratgies on the HTTP level. This would also fit in the planned upgrade to websockets (see improvements paragraph).

Possible improvements
---------------------

We considered using MQTT directly on the Intellifi Spot. MQTT is a lightweight protocol and could be implemented on an embedded level. It's also a client server model that would allow the spots to initiate the TCP/IP connection to the outside world. It's not running on the default HTTP port however. In bigger administrated networks the MQTT port could be closed for outband traffic. This is an important reason for not using it. Another reason is that we don't need Spots to access everything. It's enough to have a 'single' channel to a service on the Brain. This service can then collect the required information.

We still would like to offer you faster command execution however. In the current chunked encoding approach we can only download new commands every second. It we would have a bidirectional protocol then we could respons immediatly. Websockets form a very intresting apporach to this problem. It's basically an upgrade of the HTTP protocol that allows bidirectional communcation over HTTP. We are planning to implement this in the coming year.

Feedback
--------

We are always open to new approaches to improve our solution. Please find some additional consideratons in the last paragrpah. Please let us know if you feel that anything is missing (simon.koudijs@intellifi.nl).
