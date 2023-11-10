# Setting Up K3S in High Availability


## Prerequisites Hardware

| Item                                                                             | Description |
|----------------------------------------------------------------------------------|-------------|
| [Raspberry Pi 4](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/)   | 4GB or 8GB  |
| [Proxmox](https://www.proxmox.com/en/proxmox-ve)                                 | 7.4-17      |

Between Raspberry Pi's and proxmox, create Four instances of Ubuntu 23.04.1 LTS.


<procedure title="Configure a SQL Server">
<code-block lang="bash">
sudo apt install mysql-server
sudo systemctl start mysql.service
</code-block>

Replace ```10.0.0.0``` with the home network address range.

</procedure>

<procedure title="Create a user for K3s on the SQL Database">
<code-block lang="sql">
CREATE DATABASE homelab;
CREATE USER 'k8s' IDENTIFIED BY 'password'
GRANT ALL PRIVILEGES ON *.* TO 'k8s'@'10.0.0.0/255.255.255.0';
</code-block>
</procedure>

<procedure title="Install K3S Running as Master on Two Nodes">
<step>For each master node, run the following
<code-block lang="bash">
curl -sfL https://get.k3s.io | sh -s - server --datastore-endpoint="mysql://k8s:k8s_password@tcp(SQL_DB_IP:3306)/homelab"
</code-block>
</step>

</procedure>

<procedure title="Configure Nginx to load balance between the two instances.">
<step>Install Nginx
<code-block lang="bash">
sudo apt install nginx
</code-block>
</step>

<step>
Update the file <code>/etc/nginx/nginx.conf</code> to the following
</step>
</procedure>
```bash
stream {
  server {
    listen 6443;
    proxy_pass stream_master_nodes;
  }

upstream stream_master_nodes {
server IP_of_Node_1:6443;
server IP_of_Node_2:6443;
}
}
```

<procedure title="Install K3S Running as Worker on Two Nodes">
<step>For each worker node, run the following
<code-block>
curl -sfL https://get.k3s.io | sh -s - agent --server http://IP_of_Nginx_Load_Balancer:6443
</code-block>
</step>
</procedure>

