# Cluster Setup


## Master Nodes

 - K3S Token stored in 1Password
 - MySQL password stored in 1Password

### Simple Master Node Setup
For a simple master node setup, run the following command on each master node:
```bash
curl -sfL https://get.k3s.io | sh -s - server --token="<k3s_token>" --datastore-endpoint="mysql://<mysql_usere>:<my_sql_password>@tcp(<my_sql_ip>:3306)/homelab" --tls-san=<nginx_ip>
```

### Master Node Setup for Prometheus
If you want to setup a master node with a custom label, you can use the following command:
```bash
curl -sfL https://get.k3s.io | sh -s - server --token="<k3s_token>" --datastore-endpoint="mysql://<mysql_usere>:<my_sql_password>@tcp(<my_sql_ip>:3306)/homelab" --tls-san=<nginx_ip> --node-label="prometheus=true"
```


## Agent Nodes
Setting up each agent node with:
```bash
curl -sfL https://get.k3s.io | sh -s - agent --token="<k3s_token>" --server https://<nginx_ip>:6443
```
<note type="info">
 K3S Token stored in 1Password
</note>
<note type="info">
 MySQL password stored in 1Password
</note>


## Common Commands


Copy kube config file to local machine
```bash
cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
```

Uninstall K3s
```bash
/usr/local/bin/k3s-uninstall.sh
```