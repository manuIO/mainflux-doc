# TLS in Mainflux

## Key and Certiuficate Generation
Following the instructions [here](https://help.github.com/enterprise/11.10.340/admin/articles/using-self-signed-ssl-certificates/):

```bash
openssl genrsa -out mainflux.key 2048
openssl req -x509 -new -nodes -key mainflux.key -days 365 -out mainflux.crt
```

## Iris and TLS
Example here: https://github.com/iris-contrib/examples/tree/master/tls

## SSL Debugging
Following the instructions [here](https://www.kamailio.org/wiki/tutorials/tls/testing-and-debugging)
```bash
openssl s_client -connect localhost:443 -no_ssl2 -bugs
```

```bash
openssl s_client -showcerts -debug -connect localhost:443 -no_ssl2 -bugs
```

### Curl Testing
```bash
curl --cacert tls/mainflux.crt https://localhost:7070/devices
```
