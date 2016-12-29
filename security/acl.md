# Access Control

Authorization (AuthZ) in Mainflux is obtained via Policy based Access Control, as implemented in [Ladon](https://github.com/ory-am/ladon). On it's turn, Ladon was inspired by [AWS IAM Policies](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html).

## Entities
There are 3 types of entities in Mainflux that can send/recieve requests:
- User
- Device
- Application

Each entity can get a token that is used for AuthX and AuthZ.

## Tokens
Tokens in Mainflux have 2 roles:
- Authentication, or AuthX - they uniquily identify the token bearer
- Authorization, or AuthZ - they hold the Access Control Policy identifier

Example of token structure:
```json
{
  "mainflux-id": "<device_id>",
  "type": "device",
  "api-key": "<api-key-id>"
}
```

Out of this JWT is created and handed to the entity to be used as a bearer token

## Access Control
This is the process of AC:
- Entity (for example device) provides bearer token
- Token is decrypted and verified
- Token body is retrieved and `mainflux-id` field is read
- At this point AuthX part is finished - we know the identity of the entity that sent the request
- We now read `api-key` and use this as a `context` condition in Ladon's Warden
- Example of Warden check when`GET /channels/<channel_id>` was executed:
```go
var err = warden.IsAllowed(&ladon.Request{
    // ...
    Subject: "<mainflux-id>",
    Action: "read"
    Resource: "channels/<channel_id>",
    Context: &ladon.Context{
         "api-key": "<api-key>",
    },
}
```
In this JSON `<policy-id>` is read from bearer token, and it is a subject used to create all policies belonging to this token (tokens can be observed as API key), and `resource` field is read from request URI.

## Policies and API Key
An entity in Mainflux is uniquely identified by it's UUID. We can create different Ladon policies for one entity, but if we use the same Entity ID in a `subject` body and only `EqualsSubjectCondition` then all the policies will be applied to our entity. 

What we want is to create a different access scenarios (cases) - i.e. different API keys.

One API key can be regarded as a group of policies, all applied one after another to create a unique access scenario. We group these policies in one access scenario by adding `StringMatchCondition` - all the policies from this group must match to the field `"api-key": "<some-unique-identifier>"`, and we produce this `api-key` identifier prior to making policies and provide it during the process of policy making.

This way, although it is written into JWT and potentially burned in the falsh of some device, we can still change what `api-key` really means, i.e. what access scenario it really represents.

### Creating Policies
#### Master Key (MK)
- User API key called Master Key (MK) is created when user is created.
- This key can do anything on both `/devices/*` and `/channels/*` resources

#### Device Master Key (DMK)
- DMK is created when a device is created
- In the beginning it can only do all operations on `/devices/<device_id>`
- As we connect it (plug) to the channels policies are added with the same `api-key` ID that it can do some (all) operations on `channels/<channel_id>`

#### Application Master Key (AMK)
- Same like DMK

#### Secondary Keys
- These keys are alway derived from equivalent Master Keys
- They can be only **subset** in scope - never superset
- First MK (or DMK or AMK) key is analyzed by retrieving all policies, overlapping them and thus generating a list of what is allowed (maximum set created by union of policies)
- A subset of policies is choosen by the creator of new (secondary) API key
- Out of this subset new policies are created with `"api-key": "<new_api_key>"` tag

































