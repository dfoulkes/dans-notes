# TLS Certificate Management

To get the SSL certificate from a website along with the chain, use the following command:

```bash
openssl s_client -showcerts -connect $DOMAIN:$PORT 
```

Extract the SSL certificate from the chain, use the following command:

```bash
openssl s_client -connect $DOMAIN:$PORT < /dev/null 2>/dev/null | openssl x509 -outform PEM >/etc/ssl/mycert.pem
```

---

## Understanding the TLS Handshake

The TLS handshake is the process your browser uses to establish a secure connection to a website. 
The handshake begins when your browser connects to the server or website you want to access. 
The two parties exchange the following information:


TLS uses asymmetric cryptography to establish a secure connection. This means two different keys are used on either end of the connection.
In public key cryptography, each party has two keys: a public key and a private key. The public key is shared with the world, while the private key is kept secret.

During the exchange of the two keys, the following happens:

1. The client sends a ClientHello message to the server, along with the TLS version number, a random number, a list of supported cipher suites, and other information.
2. The server responds with a ServerHello message, along with the TLS version number, a random number, the cipher suite chosen by the server, and other information.
3. The server sends its public key to the client in the form of a certificate.
4. The client verifies the certificate. If the verification is successful, the client sends a random number encrypted with the server’s public key to the server.
5. The server decrypts the random number using its private key. This is the pre-master secret.
6. Both the client and the server generate the session keys from the random numbers they have received. The session keys are symmetric keys used to encrypt and decrypt information exchanged during the TLS session.
7. The client sends a message to the server informing it that future messages from the client will be encrypted with the session key.
8. The server sends a message to the client informing it that future messages from the server will be encrypted with the session key. The TLS handshake is now complete, and the session begins.

## How Clients verify the server's certificate

The client verifies the server’s certificate using the following steps: 
1. The client checks the certificate’s signature to ensure it is valid.
2. The client checks the certificate’s validity date to ensure it has not expired.
3. The client checks the certificate’s common name to ensure it matches the domain name of the website it is trying to access.
4. The client checks the certificate’s issuer to ensure it is issued by a trusted CA.

## How Clients Verify trust in the CA

The client checks the certificate’s issuer to ensure it is issued by a trusted CA. Different technologies have different implementations
of trust stores. For example, Java uses a keystore, while Windows uses the certificate store and Centos uses the ca-certificates package.

## Java

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


## CentOs 


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




