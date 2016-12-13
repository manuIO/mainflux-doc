Mainflux Auth API
=================
Mainflux IoT platform authentication and authorization service API.

**Version:** 1.0.0

### /status
---
##### ***GET***
**Summary:** Service health check.

**Description:** The endpoint returns information about the service status, i.e. whether
or not it is capable to respond to any incoming request.


**Responses**

| Code | Description |
| ---- | ----------- |
| 200 | Service is alive and able to respond to new requests. |
| 500 | Service cannot fulfill any further requests. |

### /users
---
##### ***POST***
**Summary:** Register new user account.

**Description:** New accounts are registered given their username and password. Provided
username must be platform-wide unique. Once the account is created, its
unique identifier is generated together with new master key.


**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| credentials | body | Account credentials. | No |  |

**Responses**

| Code | Description |
| ---- | ----------- |
| 201 | Account is created. |
| 400 | Missing or invalid username and/or password. |
| 409 | Provided username is not unique. |
| 500 | Service is unable to fulfill the request. |

### /sessions
---
##### ***POST***
**Summary:** Retrieves account's master key.

**Description:** To retrieve their master keys, users are required to provide their
username and password.


**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| credentials | body | Account username and password. | No |  |

**Responses**

| Code | Description |
| ---- | ----------- |
| 201 | Valid username and password submitted. The client has been provided with master key and an account ID.  |
| 400 | Invalid request submitted. |
| 403 | Missing or invalid username and/or password. |
| 500 | Service is unable to fulfill the request. |

### /api-keys
---
##### ***GET***
**Summary:** Retrieves all keys created by the client.

**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| Authorization | header | Client's master key. | Yes | string |

**Responses**

| Code | Description |
| ---- | ----------- |
| 200 | Retrieved a list of created keys. |
| 403 | Missing or invalid master key. |
| 500 | Service is unable to fulfill the request. |

##### ***POST***
**Summary:** Creates new API key.

**Description:** An API key can be given to the user, device or channel. The consequence
of this fact is that the owner's ID must be explicitly provided during key
creation. It is allowed, but not mandatory, to limit the key scope at this
point.


**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| Authorization | header | Client's master key. | Yes | string |
| spec | body | Key specification containing its owner and scope(s). | Yes |  |

**Responses**

| Code | Description |
| ---- | ----------- |
| 201 | API key created. |
| 403 | Missing or invalid master key. |
| 500 | Service is unable to fulfill the request. |

### /api-keys/{key}
---
##### ***GET***
**Summary:** Retrieves key info.

**Description:** Retrieved data provides a key owner (user, device or channel ID), together
with all actions key owner can perform.


**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| Authorization | header | Client's master key. | Yes | string |
| key | path | The key. | Yes | string |

**Responses**

| Code | Description |
| ---- | ----------- |
| 200 | Retrieved key info. |
| 403 | Missing or invalid master key. |
| 404 | Non-existent key requested. |
| 500 | Service is unable to fulfill the request. |

##### ***PUT***
**Summary:** Updates key scopes.

**Description:** Updates the key scope by completely replacing the current scope with the
provided one.


**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| Authorization | header | Client's master key. | Yes | string |
| key | path | The key. | Yes | string |
| scopes | body | Key's scope specification. | Yes |  |

**Responses**

| Code | Description |
| ---- | ----------- |
| 200 | The key has been updated. |
| 400 | Invalid key specification provided. |
| 403 | Missing or invalid master key. |
| 404 | Cannot update a non-existent key. |
| 500 | Service is unable to fulfill the request. |

##### ***DELETE***
**Summary:** Revokes the key.

**Description:** Completely removes the key.

**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| Authorization | header | Client's master key. | Yes | string |
| key | path | The key to be revoked. | Yes | string |

**Responses**

| Code | Description |
| ---- | ----------- |
| 204 | The key has been revoked. |
| 403 | Missing or invalid master key. |
| 404 | Cannot revoke a non-existent key. |
| 500 | Service is unable to fulfill the request. |

### /access-checks
---
##### ***POST***
**Summary:** Request an access to the platform resource.

**Description:** This endpoint is used as an entrypoint to the protected system resources.
The clients are required to provide a full access specification, regardless
of their origin (e.g. HTTP, MQTT).


**Parameters**

| Name | Located in | Description | Required | Type |
| ---- | ---------- | ----------- | -------- | ---- |
| Authorization | header | Client's API key. | Yes | string |
| X-Resource | header | Resource URI (e.g. /channels/123). | Yes | string |
| X-Action | header | Proxy-forwarded HTTP request method. | Yes | string |

**Responses**

| Code | Description |
| ---- | ----------- |
| 200 | Granted resource access. |
| 400 | Missing or invalid access specification. |
| 403 | Resource access is not allowed. |
| 500 | Service is unable to fulfill the request. |
