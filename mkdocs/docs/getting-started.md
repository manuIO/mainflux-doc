# Getting Started
Mainflux is a composition of microservices.

Docker containers are provided for each [Mainflux component](http://mainflux.io/#architecture).

## Prerequisites

1. [Docker](https://www.docker.com/community-edition). If you are not familiar with Docker, take a moment to install Docker and work through the Quickstart guide. We'll still be here.
2. [jq](https://stedolan.github.io/jq/download/) (optional). Pretty printing json in your terminal.
3. [Mosquitto](https://mosquitto.org/download/) (optional). Mainflux MQTT require the mosquitto cli broker.
4. [Go](https://golang.org/dl/) (optional). Minimum version required is 1.7.

## Installation
To install and deploy the Mainflux system, clone the repo:
```bash
git clone https://github.com/Mainflux/mainflux.git && cd mainflux
```

Then start the Docker composition:
```bash
docker-compose up
```

## Base Concepts - Devices and Channels
The Mainflux system is simple - it has only two main concepts: Devices and Channels.

1. `Device` is used to represent any device that connects to Mainflux. It is a generic model
that describes any client device of the system via simple structure like [this](https://github.com/mainflux/mainflux-manager/blob/master/models/device.go).

2. `Channel` is used to model a communication channel. It is a generic bidirectional message stream representation.
Channels are like MQTT topics - several devices or applications can subscribe or publish on a channel.
All the values that flow through channels are persisted in the database.

Using these two simple representations (Devices and Channels), it is possible to model complex IoT systems.
`Device` structures will be used for device management while `Channel` structures will be used for IoT messaging.

## SenML
Mainflux uses an [IETF standardized message format called SenML](https://tools.ietf.org/html/draft-ietf-core-senml-08) to exchange IoT messages in the system.

A Mainflux message is a JSON array of SenML hashes. This message is a temperature reading in Celsius:

```
[
     {"n":"urn:dev:ow:10e2073a01080063","u":"Cel","v":23.1}
]
```

SenML permits multiple measurements to be sent in one message. This message contains voltage and current readings:

```
[
     {"bn":"urn:dev:ow:10e2073a0108006;", "bt":1.276020076001e+09, "bu":"A", "bver":5, "n":"voltage","u":"V","v":120.1},
     {"n":"current","t":-5,"v":1.2},
     {"n":"current","t":-4,"v":1.3},
     {"n":"current","t":-3,"v":1.4},
     {"n":"current","t":-2,"v":1.5},
     {"n":"current","t":-1,"v":1.6},
     {"n":"current","v":1.7}
]
```

## Provisioning
Configuring your Mainflux system begins with a provisioning phase. Use the [mainflux-manager](https://github.com/mainflux/mainflux-manager) HTTP/REST API to provision Devices and Channels.

### Provisioning Devices
Devices are provisioned by executing a HTTP request `POST /devices`:
```bash
curl -s -S -i -X POST -H "Content-Type: application/senml+json" http://localhost:9090/devices
```

Notice the UUID of the device in the Location header of the HTTP response:
```
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Location: /devices/2670ae6d-0cdb-43b9-8cf4-b7f31a1d0a43
Date: Fri, 26 May 2017 04:12:53 GMT
Content-Length: 0
```

It is important to note that the Mainflux system generates a UUID for each Device or Channel it creates.
This ID is later used to authenticate each Device on the system.

### Provisioning Channels
Channels are provisioned by executing a HTTP request `POST /channels`:
```bash
curl -s -S -X POST -H "Accept: application/json" -H "Content-Type: application/json" http://localhost:9090/channels
```

List your Channels:
```bash
curl http://localhost:9090/channels | jq
```

## Publishing on a Channel
Once a Channel is provisioned it is possible to start publishing measurements on the channel.

This can be done via several protocols:

- HTTP
- MQTT
- WebSockets (MQTT over WebSockets)

### HTTP

Publishing and retrieving messages (values) of one particular channels is done via `POST` or `GET` on an API endpoint `/channels/:channel_id/messages` of [mainflux-http-sender](https://github.com/mainflux/mainflux-http-sender) service:

- Send message on the channel: `POST /channels/:channel_id/messages <JSON_SenML_string>`
- Get messages from the channel: `GET /channels/:channel_id/messages`

> **N.B.** publisher ID is passed to HTTP server via `Client-ID` header. In normal use-case this publisher ID is calculated automatically via `mainflux-auth` service and injected into the HTTP call via NGINX proxying, but for testing it can be added explicitly in the call.

```
curl -s -S -i -X POST -H "Client-ID: 472dceec-9bc2-4cd4-9f16-bf3b8d1d3c52" -H "Content-Type: application/senml+json" http://localhost:7070/channels/78c95058-7ef3-454f-9f60-82569ddec4e2/messages -d '[{"bn":"some-base-name:","bt":1.276020076001e+09, "bu":"A","bver":5, "n":"voltage","u":"V","v":120.1}, {"n":"current","t":-5,"v":1.2}, {"n":"current","t":-4,"v":1.3}]'

HTTP/1.1 202 Accepted
Content-Type: application/senml+json; charset=utf-8
Date: Sun, 18 Dec 2016 18:25:36 GMT
Content-Length: 28

{
  "response": "message sent"
}
```

Check if the messages have been written on the channel by reading from [mainflux-influxdb-reader](https://github.com/mainflux/mainflux-influxdb-reader) service:

```
curl -s -S -i -X GET -H "Content-Type: application/senml+json" 'http://localhost:7080/channels/78c95058-7ef3-454f-9f60-82569ddec4e2/messages' | jq

HTTP/1.1 200 OK
Content-Type: application/senml+json; charset=utf-8
Date: Sun, 18 Dec 2016 18:26:27 GMT
Content-Length: 627

[
  {
    "bver": 5,
    "n": "some-base-name:voltage",
    "u": "V",
    "t": 1276020076.001,
    "v": 120.1,
    "publisher": "472dceec-9bc2-4cd4-9f16-bf3b8d1d3c52",
    "timestamp": "2016-12-18T18:25:36Z",
    "channel": "78c95058-7ef3-454f-9f60-82569ddec4e2"
  },
  {
    "bver": 5,
    "n": "some-base-name:current",
    "u": "A",
    "t": 1276020071.001,
    "v": 1.2,
    "publisher": "472dceec-9bc2-4cd4-9f16-bf3b8d1d3c52",
    "timestamp": "2016-12-18T18:25:36Z",
    "channel": "78c95058-7ef3-454f-9f60-82569ddec4e2"
  },
  {
    "bver": 5,
    "n": "some-base-name:current",
    "u": "A",
    "t": 1276020072.001,
    "v": 1.3,
    "publisher": "472dceec-9bc2-4cd4-9f16-bf3b8d1d3c52",
    "timestamp": "2016-12-18T18:25:36Z",
    "channel": "78c95058-7ef3-454f-9f60-82569ddec4e2"
  }
]
```

`GET /channels/:channel_id/messages` supports also query parameters, so you can filter your search by time interval like this:
```
curl -s -S -i -X GET -H "Content-Type: application/senml+json" 'http://localhost:9090/channels/78c95058-7ef3-454f-9f60-82569ddec4e2/messages?start_time=1276020071.9&end_time=1276020075.999' | json | pygmentize -l json

HTTP/1.1 200 OK
Content-Type: application/senml+json; charset=utf-8
Date: Sun, 18 Dec 2016 18:30:15 GMT
Content-Length: 209

[
  {
    "bver": 5,
    "n": "some-base-name:current",
    "u": "A",
    "t": 1276020072.001,
    "v": 1.3,
    "publisher": "472dceec-9bc2-4cd4-9f16-bf3b8d1d3c52",
    "timestamp": "2016-12-18T18:25:36Z",
    "channel": "78c95058-7ef3-454f-9f60-82569ddec4e2"
  }
]
```

> Note that `start_time` and `end_time` should be in UNIX time format - i.e. float64 number.
>
> Note also that `publisher` is automatically derived from MQTT client ID and set by the Mainflux system
> - there is no action needed from user.

### MQTT
Mainflux is acting as a seamless multi-protocol bridge. If you were subscribed to an MQTT topic `mainflux/channels/:channel_id/messages/<content_type>` you would get the message published via HTTP POST on `mainflux/channels/:channel_id/messages` with `Content-Type: application/senml+json` header.

Here it is important to note that we could not pass `Content-Type` in some kind of a header, as in the case of HTTP, so we use topic itself to denote the type of content that is sent over the channel.

Mainflux currently supports `application/senml+json` and `application/octet-stream` (for sending custom binary blobs), but in the future a support for `application/senml+cbor` will be added.

These topics are denoted as following:

- `mainflux/channels/5c912c4e-e37b-4ba6-8f4b-373c7ecfeaa9/messages/senml-json`
- `mainflux/channels/5c912c4e-e37b-4ba6-8f4b-373c7ecfeaa9/messages/senml-cbor`
- `mainflux/channels/5c912c4e-e37b-4ba6-8f4b-373c7ecfeaa9/messages/octet-stream`

> Note that `+` is a wildcard character for MQTT topics, so we use `senml-json` to denote `application/senml+json` content type.

Example:

```
mosquitto_sub -t mainflux/channels/5c912c4e-e37b-4ba6-8f4b-373c7ecfeaa9/messages/senml-json

[{"bn":"e35b157f-21b8-4adb-ab59-9df21461c815","bt":1.276020076001e+09, "bu":"A","bver":5, "n":"voltage","u":"V","v":120.1}, {"n":"current","t":-5,"v":1.2}, {"n":"current","t":-4,"v":1.3}]
```

Publishing via MQTT is done in the similar way:
```
mosquitto_pub -i 472dceec-9bc2-4cd4-9f16-bf3b8d1d3c52 -t mainflux/channels/5c912c4e-e37b-4ba6-8f4b-373c7ecfeaa9/messages/seml-json -m '[{"bn":"e35b157f-21b8-4adb-ab59-9df21461c815","bt":1.276020076001e+09, "bu":"A","bver":5, "n":"voltage","u":"V","v":120.1}, {"n":"current","t":-5,"v":1.2}, {"n":"current","t":-4,"v":1.3}]'
```

> Note the `-i` option to `mosquitto_pub`: it tells to MQTT broker the client ID of the publisher by providing it `deviceID` of the device which sends the message

### Websockets
Mainflux also supports Websockets with MQTT over WS.
Every modern browser or any device is now a potential full-fledged MQTT client.
With publish/subscribe, quality of service and retain messages, clients like web apps can take full advantage
of highly scalable messaging with a very low bandwidth footprint.

Similar to MQTT, the WebSockets API supports publish and subscribe to any channel/topic on same end point as MQTT  `mainflux/channels/:channel_id/messages/senml-json`.

[Here](https://github.com/mainflux/mainflux-mqtt/blob/master/examples/paho-js-client/index.html) you will
find an example of web client implementation using the Eclipse [Paho javascript library](https://eclipse.org/paho/clients/js/)  
