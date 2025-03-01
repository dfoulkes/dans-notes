# Cluster Setup

<warning>
<title>Warning</title>
Read below when a fresh install of Raspbian is done.

</warning>

## Pre-requisites

### Configuring cgroups
```bash
sudo nano /boot/firmware/cmdline.txt
```
append the following to the end of the file:
`cgroup_memory=1 cgroup_enable=memory`


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

<warning>
<title>Important</title>
for new nodes, ensure all the longhorn prerequisites are met.
<a href="https://longhorn.io/docs/1.8.0/deploy/install/">Installation Requirements</a>
</warning>

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

<tip>
if running on a Pi, run the follow after installing Raspberian

raspi-config --expand-rootfs

</tip>

## Other Settings

### Increase the number of open files
add the following to `/etc/security/limits.conf`
```bash
*               soft    nofile          100000
*               hard    nofile          100000
```
edit the following file: `/etc/pam.d/common-session` adding the following before end of the file.

```bash
session required        pam_unix.so
```

edit `/etc/pam.d/common-session-noninteractive` and add the following line to the end of the file:
```bash
session required        pam_unix.so
```
<note>
on my last Rasbian install, this was already in place.
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