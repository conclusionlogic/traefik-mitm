Provides a simple sandbox for testing TLS/mTLS connections with.

Can be used to validate the client app if in doubt how it will behave in MITM scenarios, or just for fun.

## Network diagram

![](diagram.svg)

## Server certificate?

Generate a self-signed certificate for the domain the client app connects to

```
$ openssl genrsa 4096 > .certs/server/private.key
$ openssl req -new -x509 -nodes -sha256 -days 365 -key .certs/server/private.key -outform PEM -out .certs/server/certificate.pem
```

## Client certificate?

Obtain them from the server that enforces client certificate validation by generating the client certificates with the CA.

## Testing

Update `conf.yml` according to the hostname/certifiate that needs testing and then run:

```
docker compose up
```

Test scenario 1: (One-way TLS) The client app should not work with self-signed certificates.
Connections from the app or website should fail when connecting to the server (via the MITM proxy) that presents a self-signed certificate or a certificate issued to a diffent hostname. Normally, this is handled correctly by the network stack and such connections are dropped by the client device unless the self-signed server certificate is explicitly trusted on the client platform.
One mitigation option is to configure the client to accept only the real, known server certificate or its CA.


Test scenario 2: (Mutual TLS) Traefik should not be able to proxy client connections to the target (real) servers without supplying client certificates for the server to validate.


## Conclusion
If both scenarios fail, the result should be a full end-to-end MITM scenario where client sends requests through Traefik which proxies them to the backend, terminating and rebuilding TLS in the middle.

