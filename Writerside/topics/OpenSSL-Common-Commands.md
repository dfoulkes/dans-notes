# OpenSSL Common Commands

To get the SSL certificate from a website along with the chain, use the following command:

<warning>
Do not include the protocol in the domain name.
</warning>
```bash
openssl s_client -showcerts -connect $DOMAIN:$PORT 
```


Extract the SSL certificate from the chain, use the following command:

```bash
openssl s_client -connect $DOMAIN:$PORT < /dev/null 2>/dev/null | openssl x509 -outform PEM >/etc/ssl/mycert.pem
```
