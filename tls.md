# TLS in Mainflux

## Key and Certiuficate Generation
Following the instructions [here](https://help.github.com/enterprise/11.10.340/admin/articles/using-self-signed-ssl-certificates/), [here](http://uwsgi-docs.readthedocs.io/en/latest/HTTPS.html) and especially [here](http://www.shellhacks.com/en/HowTo-Create-CSR-using-OpenSSL-Without-Prompt-Non-Interactive)

```bash
openssl req -nodes -newkey rsa:2048 -keyout mainflux.key \
  -out mainflux.csr -subj "/C=FR/ST=IDF/L=Paris/O=Mainflux/OU=IoT/CN=localhost"
openssl x509 -req -days 365 -in mainflux.csr -signkey weioSSL.key -out mainflux.crt
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
