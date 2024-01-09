# OpenSSL Commands

To get the SSL certificate from a website along with the chain, use the following command:

```bash
openssl s_client -showcerts -connect $DOMAIN:$PORT 
```

Extract the SSL certificate from the chain, use the following command:

```bash
openssl s_client -connect $DOMAIN:$PORT < /dev/null 2>/dev/null | openssl x509 -outform PEM >/etc/ssl/mycert.pem
```

<procedure title="Install a root certificate on CentOS">

<step>

<p>Extract root certificate from the website</p>

```bash
openssl s_client -connect $DOMAIN:$PORT < /dev/null 2>/dev/null | openssl x509 -outform PEM >/etc/ssl/mycert.pem
```

</step>

<step>

<p>install ca-certificates and update the ca-trust</p>

```bash
yum install -y ca-certificates
update-ca-trust force-enable
```

</step>

<step>
<p>Create a synlink to the root certificate</p>

```bash
sudo ln -s /etc/ssl/mycert.pem /etc/pki/ca-trust/source/anchors/mycert.pem
```
</step>

<step>

<p>Update the ca-trust store</p>

```bash
sudo update-ca-trust
```

</step> 

</procedure>




