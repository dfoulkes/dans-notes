# OpenSSL Commands


To get the SSL certificate from a website along with the chain, use the following command:
<code-block lang="bash">
openssl s_client -showcerts -connect $DOMAIN:$PORT 
</code-block>

Extract the root certificate from the chain:

<code-block lang="bash">
openssl s_client -connect $DOMAIN:$PORT < /dev/null 2>/dev/null | openssl x509 -outform PEM >root.pem
</code-block>