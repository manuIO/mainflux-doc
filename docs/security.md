# Security
Professional deployments of Mainflux include [Authentication and Authorization
Service][auth-git], which adds fine-grained security layer to the platform.

## Introduction
The entrypoint to the platform is user account registration. Once an account is
registered, it is granted with **master key**. This key should be used to identify
resource owners, i.e. it will have "superuser" permissions on resources owned by
that user. Note that the resource owners can customize access to their resources
by creating additional API keys with different, in most cases lowered, permissions.

Internally, the service maintains a list of keys allowed to perform the requested
action, per each of the available resources. Recognized actions are **Create**,
**Retrieve**, **Update** and **Delete**, each of them corresponding to HTTP methods
**POST**, **GET**, **PUT** and **DELETE**, respectively.

The clients are obligated to provide the **Authorization** header, with the following
content: **Bearer KEY**, KEY being the API key they're assigned with. Once the service
intercept a request, it will try to find the provided key in the appropriate list. If
the key is found, the request is allowed and vice versa.

[auth-git]: https://github.com/mainflux/mainflux-auth
[crud-wiki]: https://en.wikipedia.org/wiki/Create,_read,_update_and_delete  

## Managing Users
### Creating user
Users are created by issuing `POST /users` and providing JSON with `username` and `password` fields:
```
curl -s -S -X POST -H "Accept: application/json" -H "Content-Type: application/json" http://localhost:8180/users -d '{"username": "john", "password": "bigSecret"}' | json | pygmentize -l json

{
  "id": "109609c7-e44e-4289-a772-9fc2e56da02b",
  "key": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjE0ODE0ODgwNTEsImlzcyI6Im1haW5mbHV4Iiwic3ViIjoiMTA5NjA5YzctZTQ0ZS00Mjg5LWE3NzItOWZjMmU1NmRhMDJiIn0.EoD2tRQYe_Nu8ANgAC9Uz52HyiL2ENXYY9e5je9U82c"
}
```

### Logging-in User (Starting the Session)
To retrieve their master keys, users are required to provide their username and password:
```
curl -s -S -X POST -H "Accept: application/json" -H "Content-Type: application/json" http://localhost:8180/sessions -d '{"username": "john", "password": "bigSecret"}' | json | pygmentize -l json


{
  "id": "109609c7-e44e-4289-a772-9fc2e56da02b",
  "key": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjE0ODE0ODgwNTEsImlzcyI6Im1haW5mbHV4Iiwic3ViIjoiMTA5NjA5YzctZTQ0ZS00Mjg5LWE3NzItOWZjMmU1NmRhMDJiIn0.EoD2tRQYe_Nu8ANgAC9Uz52HyiL2ENXYY9e5je9U82c"
}

```

## API Keys
### Obtaining API key
Mainflux Auth Server uses different API keys to define level of access. API keys are based on powrful concept of JWT,
which provide a mechanism to scope the access to the resources.

There are **Master API Key** and 3 additional types of API keys:

- User API Key
- Device API Key
- Channel API Key

Master API Key opens all the locks - everything that belongs to a particilar user can be opened with Master API Key.
Then additional API keys can be issued with various access scopes and expiration dates to fine-grain security scheme:

```
curl -s -S -X POST -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjE0ODE0ODgwNTEsImlzcyI6Im1haW5mbHV4Iiwic3ViIjoiMTA5NjA5YzctZTQ0ZS00Mjg5LWE3NzItOWZjMmU1NmRhMDJiIn0.EoD2tRQYe_Nu8ANgAC9Uz52HyiL2ENXYY9e5je9U82c" -H "Content-Type: application/json" http://localhost:8180/api-keys -d '{"owner": "john", "scopes":[{"actions":"CR","type":"devices","id":"e35b157f-21b8-4adb-ab59-9df21461c815"}]}' | json | pygmentize -l json

{
  "key": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjE0ODE0OTM2ODMsImlzcyI6Im1haW5mbHV4Iiwic3ViIjoiam9obiJ9.7ROoxijqYBzdKBl33lguC9WyoKpYksHKqspvB3r9ib8"
}
```

This generated API key allows user `john` to access the device `e35b157f-21b8-4adb-ab59-9df21461c815` for Create and Retrieve scope.
However:

```
curl -s -S -X POST -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjE0ODE0ODgwNTEsImlzcyI6Im1haW5mbHV4Iiwic3ViIjoiMTA5NjA5YzctZTQ0ZS00Mjg5LWE3NzItOWZjMmU1NmRhMDJiIn0.EoD2tRQYe_Nu8ANgAC9Uz52HyiL2ENXYY9e5je9U82c" -H "Content-Type: application/json" http://localhost:8180/api-keys -d '{"owner": "97bd76d3-8f8f-4e1f-8c20-4a6c84d3575f", "scopes":[{"actions":"UR","type":"devices","id":"e35b157f-21b8-4adb-ab59-9df21461c815"}]}' | json | pygmentize -l json

{
  "key": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjE0ODE0OTU0NDksImlzcyI6Im1haW5mbHV4Iiwic3ViIjoiOTdiZDc2ZDMtOGY4Zi00ZTFmLThjMjAtNGE2Yzg0ZDM1NzVmIn0.0JPM28zO1ZGjcc-5hU_s2m-61fPA6O_e5zjdXD5M9yU"
}
```

generates an API key that allows device	`97bd76d3-8f8f-4e1f-8c20-4a6c84d3575f` to access to device `e35b157f-21b8-4adb-ab59-9df21461c815` and change it's resources.

Note the field `owner` that we had to supply as an impout paramter - it is marking who are we foging this key for
(i.e. who will use this key, who will be the client, to which device we will burn it or to which app we will give it).
This is important for identifying the client, and is explained in more details in following chapter. 

### Retrieving and Updating the API Key
User "john" can easily retrieve or update his keys:

```
curl -s -S -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjE0ODE0ODgwNTEsImlzcyI6Im1haW5mbHV4Iiwic3ViIjoiMTA5NjA5YzctZTQ0ZS00Mjg5LWE3NzItOWZjMmU1NmRhMDJiIn0.EoD2tRQYe_Nu8ANgAC9Uz52HyiL2ENXYY9e5je9U82c" -X GET http://localhost:8180/api-keys/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjE0ODE0OTU0NDksImlzcyI6Im1haW5mbHV4Iiwic3ViIjoiOTdiZDc2ZDMtOGY4Zi00ZTFmLThjMjAtNGE2Yzg0ZDM1NzVmIn0.0JPM28zO1ZGjcc-5hU_s2m-61fPA6O_e5zjdXD5M9yU | json | pygmentize -l json

{
  "owner": "97bd76d3-8f8f-4e1f-8c20-4a6c84d3575f",
  "scopes": [
    {
      "actions": "CR",
      "type": "devices",
      "id": "e35b157f-21b8-4adb-ab59-9df21461c815"
    }
  ]
}
```

```
curl -s -S -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjE0ODE0ODgwNTEsImlzcyI6Im1haW5mbHV4Iiwic3ViIjoiMTA5NjA5YzctZTQ0ZS00Mjg5LWE3NzItOWZjMmU1NmRhMDJiIn0.EoD2tRQYe_Nu8ANgAC9Uz52HyiL2ENXYY9e5je9U82c" -X PUT http://localhost:8180/api-keys/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpYXQiOjE0ODE0OTU0NDksImlzcyI6Im1haW5mbHV4Iiwic3ViIjoiOTdiZDc2ZDMtOGY4Zi00ZTFmLThjMjAtNGE2Yzg0ZDM1NzVmIn0.0JPM28zO1ZGjcc-5hU_s2m-61fPA6O_e5zjdXD5M9yU -d '{"owner": "97bd76d3-8f8f-4e1f-8c20-4a6c84d3575f", "scopes":[{"actions":"R","type":"devices","id":"e35b157f-21b8-4adb-ab59-9df21461c815"}]}' | json | pygmentize -l json
```

## AuthX and AuthZ

Why is `owner` field important when creating API key? Because this API key can be given **only** to the appropriate client (user or device or app), and it identifies a client that gives the token.
I.e. when client passes the above token to the Mainflux Auth serverm then the server will know that the client's ID is `97bd76d3-8f8f-4e1f-8c20-4a6c84d3575f`.

This is a process of Authentication, otr AuthX - through "owner" field we know the identity of the client that sends the request.

In the process of Authorization, or AuthZ, this request is further analyzed and endpoint of the resource is matched to scope in JWT.
If this request tries to access device `e35b157f-21b8-4adb-ab59-9df21461c815`, i.e. do `GET /devices/e35b157f-21b8-4adb-ab59-9df21461c815` then this request is authorized,
because it has an action `R` (retrieve) in the `scopes` field. However, if this request tries to do `DELETE /devices/e35b157f-21b8-4adb-ab59-9df21461c815`, then it is refused,
because `D` (delete) action is not present in the `scopes` field of JWT.







