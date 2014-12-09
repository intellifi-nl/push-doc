Spot protocol
=============

Technical description of communication that takes place between an Intellifi spot and a server.

Outbound strategy
-----------------

The Intellifi spots create and maintain their own TCP/IP connection to the configured server. This is an important advantage in networks that are connected to the internet. Most of these networks use a router to access the internet (more info: [NAT](http://en.wikipedia.org/wiki/Network_address_translation)). By default a router will not allow any inbound traffic to your network. Outbound traffic however is always allowed. The Intellifi Spots use an outbound socket connection to get through the router to the configured server. The fact that we use HTTP to upload the events also helps, it makes it even more unlikely that the traffic is blocked. It's exacltly the same mechamism as browsing the internet. As a result of these design choices configuring an Intellifi Spot is just a matter of plug and play.

It's obviously not required to connect to the server through the internet, you can also connect to a computer inside your local network. Our 'outband' strategy is also a big advantage on internal networks. You won't have to administer the ip addresses of your spots, they all will automatically connect to your server. 

Encoding
--------

HTTP comes with a lot of overhead if you do a single post for every event that takes place. A spot can easily generate more than 200 events per second. Sending all the HTTP headers every time would make the protocol very ineffecient. That is why we are using an encoding that allows for multiple events in a single HTTP body.

Another advantage is that we can use this encoding on a plain tcp/ip socket as well, without any overhead. It's still possible to decode all the individual messages. 

At this moment all spots use [json encoding](spot_protocol_json.md), in the coming months we will switch to the way more effecient [protocol buffers encoding](spot_protocol_pb.md).

It will be especially usefull in low bandwidth network. It will also include the often asked event 'buffering': All events will be buffered in the event of a network loss, and retransmitted when the server can be reached again.

JSON is a very accessible encoding but it's also way more verbose than a binary protocol. In the spot protocol layer we will replace the JSON encoding with [protocol buffers](https://developers.google.com/protocol-buffers/) in the future. On the higher layers JSON will keep it's important role.

Telnet
------

You may enable telnet service on the spot for a direct connection on port 22. This will allow you to receive the spot protocol on your terminal. We do not advice you to use this option for production environments. It's not encrypted nor it's protected by a password. It's turned off by default. At this moment we don't plan to support SSH (as a consequence of our strong believe in the 'outbound' strategy).

Future
------


