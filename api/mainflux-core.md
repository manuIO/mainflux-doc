Mainflux Core API
=================
Mainflux IoT platform core service API.

**Version:** 0.0.1

### /status
---
##### ***GET***
**Summary:** Service health check

**Description:** The endpoint returns information about the service status, i.e. whether
or not it is capable to respond to any incoming request.


**Responses**

| Code | Description |
| ---- | ----------- |
| 200 | Service is alive and able to respond to new requests. |
| 500 | Service cannot fulfill any further requests. |

### /devices
---
##### ***POST***
**Summary:** Provision new device

**Description:** New devices are registered. Once the device is created,
it is provided with an UUID (used for suthenticating device onto the system).


**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| name | body | Device name. | No |  |

**Responses**

| Code | Description |
| ---- | ----------- |
| 201 | Device is created. |
| 400 | Missing or invalid username and/or password. |
| 409 | Provided username is not unique. |
| 500 | Server-side error occurred. |

##### ***GET***
**Summary:** Get all devices

**Description:** Get all devices provisioned in the system.


**Responses**

| Code | Description |
| ---- | ----------- |
| 201 | Devices retrieved. |
| 405 | Device not found. |
| 500 | Server-side error occurred. |

### /devices/{device_id}
---
##### ***PUT***
**Summary:** Update device

**Description:** Update device by device ID. 


**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| device_id | path | Device Id. | Yes | integer |
| device | body | Account credentials. | No |  |

**Responses**

| Code | Description |
| ---- | ----------- |
| 201 | Device is created. |
| 500 | Server-side error occurred. |

##### ***GET***
**Summary:** Get device

**Description:** Get device by device ID.


**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| device_id | path | Device Id. | Yes | integer |

**Responses**

| Code | Description |
| ---- | ----------- |
| 201 | Device retrieved. |
| 500 | Server-side error occurred. |

### /channels
---
##### ***POST***
**Summary:** Provision new channel

**Description:** New channel is registered. Once the channel is created,
it is provided with an UUID (used for identifying this channel in the system).


**Responses**

| Code | Description |
| ---- | ----------- |
| 201 | Channel is created. |
| 400 | Invalid channel specification submitted. |
| 500 | Server-side error occurred. |

##### ***GET***
**Summary:** Get all channels

**Description:** Get all provisioned channels.


**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| climit | query | Channel Limit - limits the number of retrieved channels. | No | integer |
| elimit | query | Entry Limit - limits the nb of retrieved channel entries. | No | integer |

**Responses**

| Code | Description |
| ---- | ----------- |
| 201 | Channels retrieved. |
| 500 | Server-side error occurred. |

### /channels/{channel_id}
---
##### ***PUT***
**Summary:** Update channel

**Description:** Update channel by channel ID.


**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| channel_id | path | Channel Id. | Yes | integer |
| entry | body | SenML. | Yes |  |

**Responses**

| Code | Description |
| ---- | ----------- |
| 201 | Channel updated. |
| 500 | Server-side error occurred. |

##### ***GET***
**Summary:** Get the channel

**Description:** Get the channel by channel ID.


**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| channel_id | path | Channel Id. | Yes | integer |
| elimit | query | Entry Limit - limits the nb of retrieved channel entries. | Yes | integer |

**Responses**

| Code | Description |
| ---- | ----------- |
| 201 | Device retrieved. |
| 500 | Server-side error occurred. |
