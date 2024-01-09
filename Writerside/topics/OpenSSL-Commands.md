# OpenSSL Commands


To get the SSL certificate from a website along with the chain, use the following command:
<code-block lang="bash">
openssl s_client -showcerts -connect $DOMAIN:$PORT 
</code-block>

## Extract the root certificate from the chain

Extract the SSL certificate from the chain, use the following command:

<code-block lang="bash">
openssl s_client -connect $DOMAIN:$PORT < /dev/null 2>/dev/null | openssl x509 -outform PEM >/etc/ssl/mycert.pem
</code-block>

---

<procedure title="Install a root certificate on CentOS">
<step>
<p>Extract root certificate from the website</p>
<code-block lang="bash">
openssl s_client -connect $DOMAIN:$PORT < /dev/null 2>/dev/null | openssl x509 -outform PEM >/etc/ssl/mycert.pem
</code-block>
</step>
<step>
<p>install ca-certificates and update the ca-trust</p>
<code-block lang="bash">
yum install -y ca-certificates
update-ca-trust force-enable
</code-block>
</step>
<step>
<p>Create a synlink to the root certificate</p>
<code-block lang="bash">
sudo ln -s /etc/ssl/mycert.pem /etc/pki/ca-trust/source/anchors/mycert.pem
</code-block>
</step>
<step>
<p>Update the ca-trust store</p>
<code-block lang="bash">
sudo update-ca-trust
</code-block>
</step> 
</procedure>




