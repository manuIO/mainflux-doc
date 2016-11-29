# Mainflux

## About
Mainflux is modern, highly-secured open source and patent-free IoT cloud platform written in Go and Erlang, based on a set of microservices.

It allows device, user and application connections over various network protocols, like HTTP, MQTT, WebSocket and CoAP, making a seamless bridge between them. It is used as the IoT middleware for building complex IoT solutions.

![gophersBanner](https://raw.githubusercontent.com/mainflux/mainflux-doc/master/img/gopherBanner.jpg)

More precisely, Mainflux is multi-protocol device-agnostic message relay with distributed time-series
data storage and multi-user, multi-tenant device and application management middleware.

Even more precisely Mainflux is:

- Device manager - accepts device connections on southbound interface
- Application manager - accepts application connections on northbound interface
- Messaging bridge - relays messages between devices and applications
- Time-series storage engine - stores and queries measurements datapoints in the time-series format

We say that Mainflux is "multi-protocol" because it supports several networking protocols that proliferate in IoT world:

- HTTP
- WebSocket
- MQTT
- LwM2M (CoAP)

We say that Mainflux is "device-agnostic" because type of device that connects to it is completely transparent to
Mainflux due it's generic internal device model (representation).
That means that you can connect absolutely every device to Mainflux

Mainflux is built with <3 by [Mainflux Company](http://mainflux.com) and community contributors.

## Architecture
Mainflux IoT cloud is composed of several components, i.e. microservices:

| Link                                                       | Description           |
|:-----------------------------------------------------------|:----------------------|
| [mainflux-core](https://github.com/mainflux/mainflux-core) | Mainflux core service |
| [mainflux-auth](https://github.com/mainflux/mainflux-auth) | Authentication and authorization server |
| [mainflux-mqtt](https://github.com/mainflux/mainflux-mqtt) | MQTT PUB/SUB broker (with WebSocket support) |
| [mainflux-ui](https://github.com/mainflux/mainflux-ui)     | System dashboard in Angular 2 Material |
| [mainflux-cli](https://github.com/mainflux/mainflux-cli)   | Interactive command-line interface |
| [MongoDB](https://github.com/mongodb/mongo)                | Devices/Measurements NoSQL database |
| [Redis](https://github.com/antirez/redis)                  | System async events and caching |
| [NATS](https://github.com/nats-io/gnatsd)                  | System enet bus |

Docker composition that constitues Mainflux IoT infrastructure is defined in the [`docker-compose.yml`](https://github.com/Mainflux/mainflux/blob/master/docker-compose.yml).

## Security
For professional deployments Mainflux is usually combined with [Mainflux Authentication and Authorization Server](https://github.com/mainflux/mainflux-auth-server) which adds fine-grained security based on customizable API keys.

Mainflux Auth Server also provides user accounts and device and application access control with simple customizable scheme based on scoped JWTs.

## Features
An extensive (and incomplete) list of features includes:

- Responsive and scalable architecture based on a set of [microservices](https://en.wikipedia.org/wiki/Microservices)
- Set of clean APIs: HTTP RESTful, MQTT, WebSocket and CoAP
- SDK - set of client libraries for many HW platforms in several programming languages: C/C++, JavaScript, Go and Python
- Device management and provisioning and OTA FW updates
- Highly secured connections via TLS and DTLS
- Enhanced and fine-grained security via deployment-ready [Mainflux Authentication and Authorization Server](https://github.com/mainflux/mainflux-auth-server) with Access Control scheme based on customizable API keys and scoped JWT
- Easy deployment and high system scalability via [Docker](https://www.docker.com/) images
- Clear project roadmap, extensive development ecosystem and highly skilled developer community
- And many more

## Community
### Mailing list
[mainflux](https://groups.google.com/forum/#!forum/mainflux) Google group

For quick questions and suggestions you can also use [GitHub Issues](https://github.com/Mainflux/mainflux/issues).

### IRC
[Mainflux Gitter](https://gitter.im/Mainflux/mainflux?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

### Twitter
[@mainflux](https://twitter.com/mainflux)

## Authors
Main architect and BDFL of Mainflux project is [@drasko](https://github.com/drasko). Additionaly, initial version of Mainflux was architectured and crafted by [@janko-isidorovic](https://github.com/janko-isidorovic), [@nmarcetic](https://github.com/nmarcetic) and [@mijicd](https://github.com/mijicd).

