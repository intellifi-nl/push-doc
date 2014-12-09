Spot protocol JSON
------------------

This document elaborates the JSON encoding that we currently use in our embedded devices. Please make sure that you are familiar with the [spot protocol basics](spot_protocol.md) before diving into these details. In this document knowledge of [JSON](http://www.json.org/) is assumed. 

This protocol is used for communicaton between the spot and the Brain server. But also for communication between our spots and the Intellifi Dashboard.

Important: The JSON protocol is not actively developed anymore. In the coming months we are planning to switch to a protocol buffers based protocol. We will elaborate this on [this page](spot_protocol_pb.md) when ready.

Framing
-------

An event shall be transmitted in this format:

```JSON
["resource_type","action",{}]\n\n
```

The combination of a `resource_type`, `action` and a valid JSON object defines an event. You may recognise the used resources from the [web API description](https://github.com/intellifi-nl/doc-webapi). This is no coincidence, we are trying to connect our abstractions where we can. In this protocol the `resource_type` is always a noun in it's singular form (other protocol use the plural). The `action` is a verb that is applied to the resource. This can be "create", "update" and "delete".

General remarks:
* The message shall always be ended by a double line feed (\n\n). This effectively makes the protocol readable by humans and makes it easy to find messages in a stream. 
* All white-space including a single line feed character (\n) may be used to increase human readability. You may not use a double line feed (\n\n) inside the message (this is the message separator!). 
* It is strongly encouraged to prevent the use of cariage return characters (\r). 
* It is highly recommended to use a decent JSON parser for interpretation, e.g. to make the interpretation robust for potential future expansions.

Events
------

Most events flow from the spot to the server, upstream. These events are avaialble:

| `resource_type` | `action` | description |
| --------------- | -------- | ----------- |
| spot            | create   | First message send when connecting, contains object with information about the spot. |
| spot            | update   | Alive signal, transmitted every second. Only in telnet mode! |
| config          | create   | Second message send when connecting, contains key value pairs for all parameters |
| config          | update   | Changes the config, can contain one or more config key value pairs. |
| hit             | create | Single item read was executed, only when `send_hits` setting is true (false by default). A lot of hit events may be created if a lot of tags are nearby. Values higher than 150 hits per second are not strange. Only use this setting in an environment with high bandwidth. |
| presence        | create | A new item has been seen. |
| presence        | update | The proximity of an item changed. |
| presence        | delete | Item has not been seen for x seconds, x can be configured by the `hold_delay_s` config setting. |

It's possible to 'command' the spot by sending events from the server to the spot. These are called commands or downstream events. At this moment we support these commands:

| `resource_type` | `action` | description |
| --------------- | -------- | ----------- |
| config          | update   | Changes the config, can contain one or more config key value pairs from the spot config. i.e. `{"brain_address":"192.168.0.120"}` |
| config          | update   | `{"reset":true}` Resets spot to factory defaults. |
| firmware        | update   | `{"url":"/firmware/beta/beta22.bin","version":"beta22"}` Instructs spot to download and install new firmware version. Url path is relative to server sending this command. Firmware updating is only allowd on dashboard connection at this moment. |
| firmware        | update   | `{"reset_counters":true}` Resets the boot counters to 0. |

The actual JSON objects send are defined by example in paragraph below. Please note that it's always possible that extra key/value pairs are added to these objects in the future.

Example
-------

This example has been acquired by setting up a telnet session to an Intellifi Spot. This can only be done when the `telnet` setting is configured to true (by defaults it's false, for security reasons).

Please note that the ```["spot","update",{"online":true}]``` are send every second in telnet mode only. You will not see these messages in HTTP mode.

Also note the the `config` object is far from complete, the latest versions of the Spot software offer more settings that can be configured. The basics stay the same however, they are just extra key / value pairs that where added to the config object.

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
