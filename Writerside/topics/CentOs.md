# CentOs Import TLS Certificate

<procedure title="Install a root certificate on CentOS">

<step>

<p>Extract root certificate from the website</p>

```bash
sudo openssl s_client -connect $DOMAIN:$PORT < /dev/null 2>/dev/null | openssl x509 -outform PEM >/etc/ssl/mycert.pem
```

</step>

<step>

<p>install ca-certificates and update the ca-trust</p>

```bash
sudo yum install -y ca-certificates
sudo update-ca-trust force-enable
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