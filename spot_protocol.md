Spot protocol
=============

Technical description of communication that takes place between an Intellifi Spot and a server. Knowledge of TCP/IP and [HTTP](http://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol) is assumed in this document.

You don't need to read this page if you are using the Intellifi Brain to acquire your events.

Outbound strategy
-----------------

The Intellifi spots create and maintain their own TCP/IP connection to the configured server. This is an important advantage in networks that are connected to the internet. Most of these networks use a router to access the internet (more info: [NAT](http://en.wikipedia.org/wiki/Network_address_translation)). By default a router will not allow any inbound traffic to your network. Outbound traffic however is always allowed. The Intellifi Spots use an outbound socket connection to get through the router to the configured server. The fact that we use HTTP to upload the events also helps, it makes it even more unlikely that the traffic is blocked. It's exacltly the same mechamism as browsing the internet. As a result of these design choices working with Intellifi Spots is just a matter of plug and play.

It's obviously not required to connect to the server through the internet, you can also connect to a computer inside your local network. Our 'outband' strategy is also a big advantage on internal networks. You won't have to administer the ip addresses of your spots, they all will automatically connect to your server whenever they can.

Spot protocols
--------------

At this moment all spots use the [JSON spot protocol](spot_protocol_json.md), in the coming months we will switch to the way more effecient [PB spot protocol](spot_protocol_pb.md).

In the rest of the document we will refer to these protocols as the `spot protocol`. Both HTTP and Telnet are just a way to 'transfer' the used `spot protocol`, so it doesn't matter if we are talking about JSON or PB. The examples will show the JSON protocol because it's human readable.

HTTP
----

HTTP comes with a lot of overhead if you would do a single post for every event (a spot can easily generate more than 100 events per second). The required headers for every post are responsible for this. That's why we encode the events with the earlier mentioned `spot protocol`. It allows us to send multiple events in a single HTTP post. We are basically wrapping the `spot protocol` into HTTP.

We are using an extension of HTTP 1.1 to push events to the server as fast as possible. This extension is called [chunked encoding](http://en.wikipedia.org/wiki/Chunked_transfer_encoding). An HTTP post is started and kept 'open' for 1 second. Every event that is generated is directly written to the socket as one or more chunks (depending on size). The HTTP endpoint at the server is served by a [Node.js](http://nodejs.org/) application. Node.js has an event based architecture and directly provides us with events as chunks come in. As soon as the running HTTP post is 'finished' by the spot the server can write the downstream events (or commands) to the spot (if any avaialble). The spot immeditaly starts a new 'streaming' HTTP post when the post is finished. This approach gives serious presendence to upstream events, for now this is a good thing. Almost all events are upstream. Pleae take a look at the last paragraph for planned improvements on this approach.

A big advantage of HTTP is that additional information can be encoded in the headers. We do add the software version of the client (in the `User-Agent` header) and we add the used `spot protocol` (in the `Accept` and `Content-type` headers):

* JSON spot protocol: `text/x-intellifi-events`
* PB spot protocol: `application/x-intellifi-events-pb`

Here you can see an example with spot 203, only the event uploading is shown. The server answer is not shown:

```HTTP
POST /api/spots/203/events/ HTTP/1.1
Host: 192.168.0.10
User-Agent: spot/35-42-g0388fd9-dirty
Transfer-Encoding: chunked
Accept: text/x-intellifi-events
Content-Type: text/x-intellifi-events
Cache-Control: no-cache

132
["spot","create",{
"spot_id":203,
"version":"35-42-g0388fd9-dirty",
"pcb_version":3,
"online":true,
"dip_mask":6,
"rfid_swr":[6911,7935,839,601,5631,5631,544],
"boot_count":9347,
"watchdog_count":834,
"presence_create_count":256716,
"time_started":"2014-12-09T12:11:00Z",
"time":"2014-12-09T12:12:53Z"
}]


1f4
["config","create",{
	"rfid_power_dbm":	30,
	"rfid_continues":	true,
	"rfid_channel":	1,
	"rfid_region":	1,
	"rfid_period_ms":	250,
	"max_antenna_swr":	3500,
	"antenna_disable_mask":	64,
	"scramble_item_codes":	false,
	"rfid_immediate":	-550,
	"rfid_near":	-650,
	"rfid_hysteresis":	25,
	"rfid_max_window_size":	1,
	"rfid_target":	0,
	"rfid_action":	0,
	"rfid_dr":	1,
	"rfid_m":	2,
	"rfid_sel":	1,
	"rfid_session":	1,
	"rfid_min_q":	0,
	"rfid_max_q":	15,
	"rfid_blf":	3,
	"bt_immediate":	-550,
	"bt_n
1f4
ear":	-650,
	"bt_hysteresis":	30,
	"bt_max_window_size":	10,
	"auto_hold_delay":	true,
	"fixed_hold_delay_s":	10,
	"control_schema":	{
		"links":	{
			"snd2":	"boot",
			"ledg":	"tick",
			"ledr":	"hit",
			"out1":	"hit",
			"out2":	"hit",
			"ledfg":	"in1",
			"ledfr":	"in2"
		}
	},
	"brain_address":	"192.168.0.10",
	"dashboard_address":	"dashboard.intellifi.nl",
	"dashboard_reconnect":	true,
	"dashboard_period_s":	900,
	"alive_ds":	10,
	"use_dhcp":	true,
	"ip":	"192.168.0.118",
	"subnet":	"255
a0
.255.255.0",
	"gateway":	"192.168.0.1",
	"dns_primary":	"192.168.0.1",
	"dns_secondary":	null,
	"telnet":	true,
	"send_hits":	false,
	"hostname":	"spot203"
}]


d6
["presence","create",{
"item_code":"000000000000000000004409",
"item_codetype":"EPC Gen2",
"time_started":"2014-12-09T12:12:49.713Z",
"time_last":"2014-12-09T12:12:49.713Z",
"proximity":"far",
"hold_delay_s":4
}]


0


```

Please do note that bigger messages are send in multiple chunks. We **don't** guarantee that one event is always placed in one chunk. We do plan to offer a Node.js library that offers you an event stream.

Telnet (debugging)
------------------

You may enable a telnet server on the spot for a direct connection on TCP port 23. This will allow you to receive the `spot protocol` on your terminal or in your own application. We do not advice you to use this option for production environments. It's not encrypted nor it's protected by a password. It also requires you to administer all the ip addresses of your Spots. It's turned off by default.

Implementing SSH would solve some of the mentioned problems. At this moment we don't plan to do this. (As a consequence of our strong believe in the earlier mentioned 'outbound' strategy.)

Security
--------

HTTP is currently the default way of connecting to a server. It's a small step to run this over an SSL connection. This would encrypt all data between spot and server. We are planning to support SSL in the embedded client in the coming year. Experiments already have shown that this is a feasible step. For this moment we advice to run Intellifi Spots only inside a 'protected' network. This could be your office network or a VPN based network.

Authenticaton is also beeing worked on, we are planning to solve this with known strategies on the HTTP level. This would also fit in the planned upgrade to websockets.

Improvements
------------

We considered using MQTT directly on the Intellifi Spot. MQTT is a lightweight protocol and could be implemented on an embedded level. It's also a client server model that would allow the spots to initiate the TCP/IP connection to the outside world. It's not running on the default HTTP port however. In bigger administrated networks the MQTT port could be closed for outband traffic. This is the most important reason for not using it at spot level.

We still would like to offer you faster command execution though. In the current chunked encoding approach we can download new client commands only once per second. It we would have a bidirectional protocol then we could respond immediatly. Websockets form a very intresting approach to this problem. It's basically an upgrade of the HTTP protocol that allows bidirectional communication through the default HTTP ports. This will also help in bypassing the buffering that we noticed in reverse proxies (NGIX does this for example). We are planning to implement this in 2015.

Feedback
--------

We are always open to new approaches to improve our solution. Please let us know!

Simon Koudijs (simon.koudijs@intellifi.nl)
