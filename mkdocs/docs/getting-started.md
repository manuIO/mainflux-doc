# Getting Started
Mainflux is a composition of microservices.

A Docker containers are provided for each microservice,
and system is run as a Docker composition.

## Installation
To install and deploy Mainflux system, clone the repo:
```bash
git clone https://github.com/Mainflux/mainflux.git && cd mainflux
```

Then start the Docker composition:
```bash
docker-compose up
```

## Base Concepts - Devices and Channels
Mainflux system is simple - it has only two main concepts: Devices and Channels.

`Device` is used to represent any device that connects to Mainflux. It is thus a generic model
that describes any client device of the system via simple structure like [this](https://github.com/mainflux/mainflux-core/blob/master/models/device.go).

`Channel` is used to model a communication channel. It is a generic bidirectional message stream representation.
Channles shold be looked at like MQTT topics - several devices or applications can subscribe or publish on a channel.
All the values that flow through channels are persisted in the database.

Using these two simple represenataions (Devices and Channels), it is possible to model complex IoT systems.
`Device` structures will be used for device managelent while `Channel` structures will be used for IoT messaging.

## SenML
Mainflux uses standardized message format called SenML to exchange IoT messages in the system.
SenML is an IETF standard, and the latest standard draft can be found [here](https://tools.ietf.org/html/draft-ietf-core-senml-04)

Mainflux messages are thus JSON representation of SenML array:

```
[
     {"n":"urn:dev:ow:10e2073a01080063","u":"Cel","v":23.1}
]
```

SenML permits multiple measurements to be sent in one message:

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
Configuring and defining Mainflux system begins by provisioning phase. It is necessary
to provision in the system Devices and Channels that will be used.

Proces of provisioning and configuration (i.e. control) of Mainflux is done by using HTTP RESTful API that
Mainflux server exposes for this purpose.

### Provisioning Devices
Devices are provisioned by executing a HTTP request `POST /devices`:
```bash
curl -s -S -i -X POST -H "Content-Type: application/senml+json" http://localhost:9090/devices | json | pygmentize -l json
```

> As a side note, `json` and `pygmentize` command line tools are used to get pretty-printing of `curl` results,
> as explained in [this article](http://benw.me/posts/colourized-pretty-printed-json-with-curl/)

Expected response:
```
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Date: Tue, 29 Nov 2016 21:49:26 GMT
Content-Length: 69

{
  "response": "created",
  "id": "e35b157f-21b8-4adb-ab59-9df21461c815"
}
```

It is important to note that Mainflux system generates a UUID (Unique Universal ID) for each Device or Channel it creates.
This ID is later used to authentificate device on the system.

### Provisioning Channels
Channels are provisioned by executing a HTTP request `POST /channels`:
```bash
curl -s -S -X POST -H "Accept: application/json" -H "Content-Type: application/json" http://localhost:9090/channels | json | pygmentize -l json
```

Expected response:
```json
{
  "response": "created",
  "id": "5c912c4e-e37b-4ba6-8f4b-373c7ecfeaa9"
}
```

## Publishing values
Once `Channel` is provisioned it is possible to start publishing values (measurements on the channel).

This can be done via several protocols:

- HTTP RESTful API
- MQTT
- WS (MQTT over WS)

### HTTP

Publishing and retrieving messages (values) of one particular channels is done via `POST` or `GET` on an API endpoint `/channels/:channel_id/messages`:

- Senf message on the channel: `POST /channels/:channel_id/messages <JSON_SenML_string>`
- Get messages from the channel: `GET /channels/:channel_id/messages`

```
curl -s -S -i -X POST -H "Content-Type: application/senml+json" http://localhost:9090/channels/78c95058-7ef3-454f-9f60-82569ddec4e2/messages -d '[{"bn":"some-base-name:","bt":1.276020076001e+09, "bu":"A","bver":5, "n":"voltage","u":"V","v":120.1}, {"n":"current","t":-5,"v":1.2}, {"n":"current","t":-4,"v":1.3}]' | json | pygmentize -l json

HTTP/1.1 202 Accepted
Content-Type: application/senml+json; charset=utf-8
Date: Sun, 18 Dec 2016 18:25:36 GMT
Content-Length: 28

{
  "response": "message sent"
}
```

Check if the messages have been written on the channel:

```
curl -s -S -i -X GET -H "Content-Type: application/senml+json" 'http://localhost:9090/channels/78c95058-7ef3-454f-9f60-82569ddec4e2/messages' | json | pygmentize -l json

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

`GET /channels/:channel_id/messages` supports also query parameters, so you can filter your serach by time interval like this:
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
Mainflux is acting as a seamless multi-protocol bridge. If you were subscribed to an MQTT topic `mainflux/channels/:channel_id` you would get the message published via HTTP:

```
mosquitto_sub -t mainflux/channels/5c912c4e-e37b-4ba6-8f4b-373c7ecfeaa9

[{"bn":"e35b157f-21b8-4adb-ab59-9df21461c815","bt":1.276020076001e+09, "bu":"A","bver":5, "n":"voltage","u":"V","v":120.1}, {"n":"current","t":-5,"v":1.2}, {"n":"current","t":-4,"v":1.3}]
```

Publishing via MQTT is done in the similar way:
```
mosquitto_pub -i 472dceec-9bc2-4cd4-9f16-bf3b8d1d3c52 -t mainflux/channels/5c912c4e-e37b-4ba6-8f4b-373c7ecfeaa9 -m '[{"bn":"e35b157f-21b8-4adb-ab59-9df21461c815","bt":1.276020076001e+09, "bu":"A","bver":5, "n":"voltage","u":"V","v":120.1}, {"n":"current","t":-5,"v":1.2}, {"n":"current","t":-4,"v":1.3}]'
```

> Note the `-i` option to `mosquitto_pub`: it tells to MQTT broker the client ID of the publisher by providing it `deviceID` of the device which sends the message

### Websockets
Mainflux also supports Websockets with MQTT over WS.
Every modern browser or any device is now a potential full-fledged MQTT client.
With publish/subscribe, quality of service and retain messages, clients like web apps can take full advantage
of highly scalable messaging with a very low bandwidth footprint.

Simiral to MQTT, Websockets API supports publish and subscribe to any channel/topic on same end point as MQTT  `mainflux/channels/:channel_id`.

[Here](https://github.com/mainflux/mainflux-mqtt/blob/master/examples/paho-js-client/index.html) you will
find an example of web client implementation using Eclipse [Paho javascript library](https://eclipse.org/paho/clients/js/)  
