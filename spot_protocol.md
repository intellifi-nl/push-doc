Spot protocol
=============

Technical description of communication that takes place between an Intellifi spot and a server.

Outbound strategy
-----------------

The Intellifi spots create and maintain their own TCP/IP connection to the configured server. This is an important advantage in networks that are connected to the internet. Most of these networks use a router to access the internet (more info: [NAT](http://en.wikipedia.org/wiki/Network_address_translation)). By default a router will not allow any inbound traffic to your network. Outbound traffic however is always allowed. The Intellifi Spots use an outbound socket connection to get through the router to the configured server. The fact that we use HTTP to upload the events also helps, it makes it even more unlikely that the traffic is blocked. It's exacltly the same mechamism as browsing the internet. As a result of these design choices configuring an Intellifi Spot is just a matter of plug and play.

It's obviously not required to connect to the server through the internet, you can also connect to a computer inside your local network. Our 'outband' strategy is also a big advantage on internal networks. You won't have to administer the ip addresses of your spots, they all will automatically connect to your server. 

Events
------

On the wire [JSON](http://www.json.org/) is used to represent the spot events. In this document basic knowledge of JSON is assumed. An event shall always take this form:

```JSON
["resource_type","action",{}]\n\n
```

TODO: Create a table for this
spot,create (information about the spot)
config,create (configuration of the reader, a collection of key/value pairs that may grow in the future)
presence,create (whever a new item has been seen, always includes the first value)
presence,update (whenever the proximity changes)
presence,delete (when you didn't receive a hit for x seconds. Where x is the hold_time_s)
hit,create (single tag read on a certain antenna, avaialble for debugging purpose)

Example
-------

```JSON
["spot","create",{
    "spot_id":"201",
    "version":"beta13",
    "online":true,
    "output_1":"false",
    "output_2":"false",
    "input_1":"false",
    "input_2":"false",
    "time":"1970-01-01T00:00:00.0Z"
}]
 
["config", "create", {
    "rfid_power_dbm":30,
    "max_antenna_swr":3500,
    "antenna_disable_mask":64,
    "auto_hold_delay":true,
    "fixed_hold_delay_s":10,
    "event_beep":true,
    "presence_beep":false,
    "hit_beep":false,
    "brain_address":"192.168.250.250",
    "update_address":"dashboard.intellifi.nl",
    "use_dhcp":true,
    "ip":null,
    "subnet":null,
    "gateway":null,
    "dns_address1":null,
    "dns_address2":null,
    "hostname":"spot201"
}]
 
["presence","create",{
    "item_code":"404000001111000066667777",
    "item_codetype":"EPC Gen2",
    "time_started":"2014-02-24T07:16:56.508Z",
    "time_last":"2014-02-24T08:31:52.156Z",
    "proximity":"far",
    "hold_delay_s":4
}]
 
["presence","create",{
    "item_code":"404040401111111166667778",
    "item_codetype":"EPC Gen2",
    "time_started":"2014-02-24T08:31:46.29Z",
    "time_last":"2014-02-24T08:31:49.141Z",
    "proximity":"far",
    "hold_delay_s":4
}]
 
["presence","update",{
    "item_code":"404040401111111166667777",
    "item_codetype":"EPC Gen2",
    "time_started":"2014-02-24T08:31:46.29Z",
    "time_last":"2014-02-24T08:31:49.141Z",
    "proximity":"near",
    "hold_delay_s":4
}]

["spot","update",{"online":true}]
 
["spot","update",{"online":true}]
 
["spot","update",{"online":true}]
 
["presence","delete",{
    "item_code":"404040401111111166667777",
    "item_codetype":"EPC Gen2",
    "time_started":"2014-02-24T08:31:46.29Z",
    "time_last":"2014-02-24T08:31:49.141Z",
    "proximity":"far",
    "hold_delay_s":4
}]
 
["spot","update",{"online":true}]
 
["spot","update",{"online":true}]

["hit","create",{
    "time":"2014-05-06T09:38:37.151Z",
    "item_code":"6d1680a0aee93234a58a5102",
    "item_codetype":"EPC Gen2",
    "antenna":3,
    "signal_strength":-428
}]
```

Telnet
------

You may enable telnet service on the spot for a direct connection on port 22. This will allow you to receive the spot protocol on your terminal. We do not advice you to use this option for production environments. It's not encrypted nor it's protected by a password. It's turned off by default. At this moment we don't plan to support SSH (as a consequence of our strong believe in the 'outbound' strategy).

Future
------

JSON is a very accessible encoding but it's also way more verbose than a binary protocol. In the spot protocol layer we will replace the JSON encoding with [protocol buffers](https://developers.google.com/protocol-buffers/) in the future. On the higher layers JSON will keep it's important role.
