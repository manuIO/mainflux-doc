# Mainflux

Mainflux is IoT cloud platform. It allows device, user and application connections over various network protocols,
like HTTP, MQTT, WebSocket and CoAP, making a seamless bridge between them.
It is used as the IoT middleware for building complex IoT solutions.

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
