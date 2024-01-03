# OpenSSL Commands


To get the SSL certificate from a website along with the chain, use the following command:
<code-block lang="bash">
openssl s_client -showcerts -connect $DOMAIN:$PORT 
</code-block>