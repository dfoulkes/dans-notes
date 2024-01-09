# Java Import TLS Certificate


<procedure title="Install a root certificate on Java">
<step>
<p>Extract root certificate from the website</p>

```bash
openssl s_client -connect $DOMAIN:$PORT < /dev/null 2>/dev/null | openssl x509 -outform PEM >/etc/ssl/mycert.pem
```
</step>
<step>
<p>Import the root certificate into the keystore</p>

```bash
sudo keytool -import -alias mycert -keystore $JAVA_HOME/jre/lib/security/cacerts -file /etc/ssl/mycert.pem
```
</step>
<step>
<p>Enter the password for the keystore</p>

```Bash
changeit
```
</step>
<step>
<p>Verify the certificate is in the keystore</p>

```bash
sudo keytool -list -keystore $JAVA_HOME/jre/lib/security/cacerts
```
</step>
</procedure>