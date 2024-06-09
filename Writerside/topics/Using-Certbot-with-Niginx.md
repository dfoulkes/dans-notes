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

```bash