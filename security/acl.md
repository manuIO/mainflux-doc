# Access Control

Authorization (AuthZ) in Mainflux is obtained via Policy based Access Control, as implemented in [Ladon](https://github.com/ory-am/ladon). On it's turn, Ladon was inspired by [AWS IAM Policies](http://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies.html).

## Entities
There are 4 types of entities in Mainflux:
- User
- Device
- Channel
- Application

Each entity can get a token that is used for AuthX and AuthZ.

## Tokens
Tokens in Mainflux have 2 roles:
- Authentication, or AuthX - they uniquily identify the token bearer
- Authorization, or AuthZ - they hold the Access Control Policy identifier

Example of token structure:
```json
token: {
  "mainflux-id": "<device_id>"
  "type": "device"
  "policy-id": "<policy_id>"
}
```

Out of this JWT is created and handed to the entity to be used as a bearer token

## Access Control
This is the process of AC:
- Entity (for example device) provides bearer token
- Token is decrypted and verified
- Token body is retrieved and "id" field is read
- At this point AuthX part is finished - we know the identity of the entity that sent the request
- We now read "policy-id" and use this as a "subject" in Ladon's Warden
- Example of policy to send to Warden when`GET /channels/<channel_id>` was executed:
```json
{
  "subject": "<policy-id>",
  "action" : "read",
  "resource": "channels/<channel_id>",
}
```
In this JSON `<policy-id>` is read from bearer token, and it is a subject used to create all policies belonging to this token (tokens can be observed as API key), and `resource` field is read from request URI.




























