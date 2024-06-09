# Using Certbot with Nginx


## Overview
When using a reverse proxy as a HTTPS endpoint for services within your network, you will need to ensure that the certificates are valid. This document will walk you through how to use Certbot to generate and renew certificates for your Nginx reverse proxy.

## Prerequisites 
- Nginx installed on Ubuntu 22.04
- A domain managed by Cloudflare
- A Cloudflare account
- A Cloudflare API token


## Installation

### Install Certbot
```bash
sudo apt install certbot python3-certbot-nginx
```

### Create a Cloudflare API Token
1. Log into your Cloudflare account
2. Go to the `My Profile` section
3. Click on `API Tokens`
4. Click on `Create Token`
5. Select `Edit Zone DNS`
6. Select the domain you want to generate the certificate for
7. Click on `Continue to Summary`
8. Click on `Create Token`
9. Copy the token

### Creating a certbox configuration file
Create a file called `cloudflare.ini` with the following content:
```ini
dns_cloudflare_api_token = YOUR_API_TOKEN
``` 
Save the file under ~/.secrets/certbot/cloudflare.ini


## Common Commands

### Generate a Certificate
```bash
 certbot certonly --dns-cloudflare --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini -d example.com
```

## List Certificates
```bash
sudo certbot certificates
```

## Renew Certificates
```bash
sudo certbot renew --dns-cloudflare -v --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini
```

### Generate Certificate for new Niignx site
<warning>
Assumption is that the Nginx site is already configured under /etc/nginx/conf.d/site-name.conf
</warning>


## Order a new Certificate
```bash
sudo certbot certonly --dns-cloudflare --dns-cloudflare-credentials /root/.secrets/certbot/cloudflare.ini -d my.domain
```
```bash
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Requesting a certificate for my.domain
Waiting 10 seconds for DNS changes to propagate

Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/my.domain/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/my.domain/privkey.pem
This certificate expires on 2024-09-07.
These files will be updated when the certificate renews.
Certbot has set up a scheduled task to automatically renew this certificate in the background.
```

## Edit the Nginx Configuration

Now add the newly generated certificate to the Nginx configuration file.

under `/etc/nginx/conf.d/my.domain.conf` add the following lines:
```nginx
server {
    listen 443 ssl;
    server_name torrent.foulkes.cloud;

    ssl_certificate /etc/letsencrypt/live/my.domain/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/my.domain/privkey.pem;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```


