# Security
Professional deployments of Mainflux include [Authentication and Authorization
Service][auth-git], which adds fine-grained security layer to the platform.

## How does it work?
The entrypoint to the platform is user account registration. Once an account is
registered, it is granted with **master key**. This key should be used to identify
resource owners, i.e. it will have "superuser" permissions on resources owned by
that user. Note that the resource owners can customize access to their resources
by creating additional API keys with different, in most cases lowered, permissions.

Internally, the service maintains a list of keys allowed to perform the requested
action, per each of the available resources. Recognized actions are **Create**,
**Retrieve**, **Update** and **Delete**, each of them corresponding to HTTP methods
**POST**, **GET, **PUT** and **DELETE**, respectively.

The clients are obligated to provide the **Authorization** header, with the following
content: *Bearer KEY*, KEY being the API key they're assigned with. Once the service
intercept a request, it will try to find the provided key in the appropriate list. If
the key is found, the request is allowed and vice versa.

[auth-git]: https://github.com/mainflux/mainflux-auth
[crud-wiki]: https://en.wikipedia.org/wiki/Create,_read,_update_and_delete  
