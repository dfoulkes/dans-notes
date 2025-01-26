# Setup Cert Manager With K3S

 Firsts install the cert-manager helm chart. (see [Helmfile](Helmfile.md) for more information)

Once installed, we now will want to create a ClusterIssuer. This is a resource that tells cert-manager how to issue certificates. 
Let's first test using the staging issuer from Let's Encrypt.

Before we create the ClusterIssuer, we need to create a secret that will hold the email address that will be used to register with Let's Encrypt.

```bash
kubectl create secret generic letsencrypt-staging --from-literal=email=<your-email>
```

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
    acme:
        server: https://acme-staging-v02.api.letsencrypt.org/directory
        email: <your-email>
        privateKeySecretRef:
            name: letsencrypt-staging
        solvers:
        - http01:
            ingress:
                class: traefik
```