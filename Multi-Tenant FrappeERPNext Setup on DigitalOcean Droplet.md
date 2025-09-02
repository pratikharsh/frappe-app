# Complete Guide: Multi-Tenant Frappe/ERPNext Setup on DigitalOcean Droplet

This comprehensive guide covers setting up multiple ERPNext sites (multi-tenant) on a single DigitalOcean droplet with SSL, custom domains, and proper configuration.

## Prerequisites

- DigitalOcean Droplet (Ubuntu 20.04/22.04)
- Existing Frappe/ERPNext installation
- Domain with wildcard SSL certificate
- Root/sudo access to the server

## Overview

We'll configure multiple ERPNext sites on subdomains:
- `erp.insightse.com` (main site)
- `tvis-erp.insightse.com`
- `deltastudy-erp.insightse.com`
- `stpeters-erp.insightse.com`

All sites will share the same SSL certificate (`*.insightse.com`) and run on the same server.

## Step 1: Verify Current Setup

First, check your existing site and bench configuration:

```bash
cd ~/frappe-bench
bench --version
frappe --version
ls sites/
```

Check current Nginx configuration:
```bash
sudo cat /etc/nginx/conf.d/frappe-bench.conf
```

## Step 2: Create New Sites

### 2.1 Create Site Directories

Create directories for each new site:

```bash
cd ~/frappe-bench
mkdir -p sites/tvis-erp.insightse.com
mkdir -p sites/deltastudy-erp.insightse.com
mkdir -p sites/stpeters-erp.insightse.com
```

### 2.2 Copy Site Structure from Existing Site

Copy necessary files from your existing site (`erp.insightse.com`) to new sites:

```bash
# Copy site structure (excluding database-specific files)
cp sites/erp.insightse.com/apps.txt sites/tvis-erp.insightse.com/
cp sites/erp.insightse.com/apps.txt sites/deltastudy-erp.insightse.com/
cp sites/erp.insightse.com/apps.txt sites/stpeters-erp.insightse.com/

# Create public directories
mkdir -p sites/tvis-erp.insightse.com/public
mkdir -p sites/deltastudy-erp.insightse.com/public
mkdir -p sites/stpeters-erp.insightse.com/public

# Copy public assets if needed
cp -r sites/erp.insightse.com/public/* sites/tvis-erp.insightse.com/public/
cp -r sites/erp.insightse.com/public/* sites/deltastudy-erp.insightse.com/public/
cp -r sites/erp.insightse.com/public/* sites/stpeters-erp.insightse.com/public/
```

## Step 3: Create Site Configuration Files

### 3.1 Create site_config.json for tvis-erp.insightse.com

```bash
cat > sites/tvis-erp.insightse.com/site_config.json << 'EOF'
{
  "db_host": "142.93.211.172",
  "db_name": "tvis-erp-insightse-prod",
  "db_password": "5hTT9ZUvOUD6Gq1l",
  "db_type": "mariadb",
  "host_name": "tvis-erp.insightse.com",
  "domains": [
    {
      "domain": "tvis-erp.insightse.com",
      "ssl_certificate": "/home/frappe/ssl/erp.insightse.com.cert",
      "ssl_certificate_key": "/home/frappe/ssl/insightse.key"
    }
  ]
}
EOF
```

### 3.2 Create site_config.json for deltastudy-erp.insightse.com

```bash
cat > sites/deltastudy-erp.insightse.com/site_config.json << 'EOF'
{
  "db_host": "142.93.211.172",
  "db_name": "deltastudy-erp-insightse-prod",
  "db_password": "lwNlxKaEyI0CBxkS",
  "db_type": "mariadb",
  "host_name": "deltastudy-erp.insightse.com",
  "domains": [
    {
      "domain": "deltastudy-erp.insightse.com",
      "ssl_certificate": "/home/frappe/ssl/erp.insightse.com.cert",
      "ssl_certificate_key": "/home/frappe/ssl/insightse.key"
    }
  ]
}
EOF
```

### 3.3 Create site_config.json for stpeters-erp.insightse.com

```bash
cat > sites/stpeters-erp.insightse.com/site_config.json << 'EOF'
{
  "db_host": "142.93.211.172",
  "db_name": "stpeters-erp-insightse-prod",
  "db_password": "9JEb5vSE443yUSZa",
  "db_type": "mariadb",
  "host_name": "stpeters-erp.insightse.com",
  "domains": [
    {
      "domain": "stpeters-erp.insightse.com",
      "ssl_certificate": "/home/frappe/ssl/erp.insightse.com.cert",
      "ssl_certificate_key": "/home/frappe/ssl/insightse.key"
    }
  ]
}
EOF
```

**Note:** Replace the database credentials with your actual database information.

## Step 4: Update Server Hosts File

Add domain mappings to the server's hosts file:

```bash
sudo nano /etc/hosts
```

Add these lines:
```
142.93.211.172   erp.insightse.com
142.93.211.172   tvis-erp.insightse.com
142.93.211.172   deltastudy-erp.insightse.com
142.93.211.172   stpeters-erp.insightse.com
```

**Note:** Replace `142.93.211.172` with your actual server IP address.

## Step 5: Configure Nginx

### 5.1 Complete Nginx Configuration

Create or update `/etc/nginx/conf.d/frappe-bench.conf`:

```bash
sudo nano /etc/nginx/conf.d/frappe-bench.conf
```

Add this complete configuration:

```nginx
upstream frappe-bench-frappe {
    server 127.0.0.1:8000 fail_timeout=0;
}
upstream frappe-bench-socketio-server {
    server 127.0.0.1:9000 fail_timeout=0;
}

map $host $site_name_vmdswwn {
    erp.insightse.com               erp.insightse.com;
    tvis-erp.insightse.com         tvis-erp.insightse.com;
    deltastudy-erp.insightse.com   deltastudy-erp.insightse.com;
    stpeters-erp.insightse.com     stpeters-erp.insightse.com;
    default                         $host;
}

# Redirect all HTTP to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name
        erp.insightse.com
        tvis-erp.insightse.com
        deltastudy-erp.insightse.com
        stpeters-erp.insightse.com;
    return 301 https://$host$request_uri;
}

# HTTPS server block for all domains
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name
        erp.insightse.com
        tvis-erp.insightse.com
        deltastudy-erp.insightse.com
        stpeters-erp.insightse.com;

    root /home/frappe/frappe-bench/sites;

    ssl_certificate      /home/frappe/ssl/erp.insightse.com.cert;
    ssl_certificate_key  /home/frappe/ssl/insightse.key;

    ssl_session_timeout  5m;
    ssl_session_cache    shared:SSL:10m;
    ssl_session_tickets  off;

    ssl_stapling         on;
    ssl_stapling_verify  on;

    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;

    ssl_protocols        TLSv1.2 TLSv1.3;
    ssl_ciphers          EECDH+AESGCM:EDH+AESGCM;
    ssl_ecdh_curve       secp384r1;
    ssl_prefer_server_ciphers on;

    proxy_buffer_size 128k;
    proxy_buffers 4 256k;
    proxy_busy_buffers_size 256k;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header Referrer-Policy "same-origin, strict-origin-when-cross-origin";

    location /assets {
        try_files $uri =404;
        add_header Cache-Control "max-age=31536000";
    }

    location ~ ^/protected/(.*) {
        internal;
        try_files /$site_name_vmdswwn/$1 =404;
    }

    location /socket.io {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header X-Frappe-Site-Name $site_name_vmdswwn;
        proxy_set_header Origin $scheme://$http_host;
        proxy_set_header Host $host;
        proxy_pass http://frappe-bench-socketio-server;
    }

    location / {
        rewrite ^(.+)/$ $1 permanent;
        rewrite ^(.+)/index\.html$ $1 permanent;
        rewrite ^(.+)\.html$ $1 permanent;

        location ~* ^/files/.*\.(htm|html|svg|xml) {
            add_header Content-disposition "attachment";
            try_files /$site_name_vmdswwn/public/$uri @webserver;
        }
        try_files /$site_name_vmdswwn/public/$uri @webserver;
    }

    location @webserver {
        proxy_http_version 1.1;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Frappe-Site-Name $site_name_vmdswwn;
        proxy_set_header Host $host;
        proxy_set_header X-Use-X-Accel-Redirect True;
        proxy_read_timeout 120;
        proxy_redirect off;
        proxy_pass http://frappe-bench-frappe;
    }

    error_page 502 /502.html;
    location /502.html {
        root /usr/local/lib/python3.10/dist-packages/bench/config/templates;
        internal;
    }

    access_log  /var/log/nginx/access.log main;
    error_log   /var/log/nginx/error.log;

    sendfile on;
    keepalive_timeout 15;
    client_max_body_size 50m;
    client_body_buffer_size 16K;
    client_header_buffer_size 1k;

    gzip on;
    gzip_http_version 1.1;
    gzip_comp_level 5;
    gzip_min_length 256;
    gzip_proxied any;
    gzip_vary on;
    gzip_types
        application/atom+xml
        application/javascript
        application/json
        application/rss+xml
        application/vnd.ms-fontobject
        application/x-font-ttf
        application/font-woff
        application/x-web-app-manifest+json
        application/xhtml+xml
        application/xml
        font/opentype
        image/svg+xml
        image/x-icon
        text/css
        text/plain
        text/x-component;
}
```

### 5.2 Test and Reload Nginx

```bash
sudo nginx -t
sudo systemctl reload nginx
```

## Step 6: Update Firewall Settings

Ensure ports 80 and 443 are open:

```bash
sudo ufw allow 80
sudo ufw allow 443
sudo ufw reload
```

## Step 7: Clear Frappe Cache and Restart Services

```bash
cd ~/frappe-bench
bench clear-cache
bench clear-website-cache
sudo supervisorctl restart all
```

## Step 8: Disable Maintenance Mode

Turn off maintenance mode for all new sites:

```bash
bench --site tvis-erp.insightse.com set-maintenance-mode off
bench --site deltastudy-erp.insightse.com set-maintenance-mode off
bench --site stpeters-erp.insightse.com set-maintenance-mode off
```

## Step 9: Set Administrator Passwords

Set up login credentials for each site:

```bash
bench --site tvis-erp.insightse.com set-admin-password
bench --site deltastudy-erp.insightse.com set-admin-password
bench --site stpeters-erp.insightse.com set-admin-password
```

Each command will prompt you to enter a new password for the Administrator user.

## Step 10: DNS Configuration

### 10.1 For Testing (Local Hosts File)

Add these lines to your local machine's hosts file:
- **Linux/macOS:** `/etc/hosts`
- **Windows:** `C:\Windows\System32\drivers\etc\hosts`

```
142.93.211.172 erp.insightse.com
142.93.211.172 tvis-erp.insightse.com
142.93.211.172 deltastudy-erp.insightse.com
142.93.211.172 stpeters-erp.insightse.com
```

### 10.2 For Production (DNS Provider)

In your DNS provider's control panel, create A records:

| Hostname | Type | Value |
|----------|------|--------|
| erp.insightse.com | A | 142.93.211.172 |
| tvis-erp.insightse.com | A | 142.93.211.172 |
| deltastudy-erp.insightse.com | A | 142.93.211.172 |
| stpeters-erp.insightse.com | A | 142.93.211.172 |

## Step 11: Verification

### 11.1 Test Site Status

```bash
bench --site tvis-erp.insightse.com doctor
bench --site deltastudy-erp.insightse.com doctor
bench --site stpeters-erp.insightse.com doctor
```

### 11.2 Test HTTPS Access

From your local machine (after DNS/hosts configuration):

```bash
curl -I https://tvis-erp.insightse.com
curl -I https://deltastudy-erp.insightse.com
curl -I https://stpeters-erp.insightse.com
```

You should receive `HTTP/2 200 OK` responses.

### 11.3 Browser Testing

Open each URL in your browser:
- https://tvis-erp.insightse.com
- https://deltastudy-erp.insightse.com
- https://stpeters-erp.insightse.com

You should see the ERPNext login page for each site.

## Login Credentials

For all sites:
- **Username:** Administrator
- **Password:** (the password you set in Step 9)

## Troubleshooting

### Common Issues and Solutions

1. **503 "Updating" Error**
   ```bash
   bench --site <site-name> set-maintenance-mode off
   sudo supervisorctl restart all
   ```

2. **Connection Refused Error**
   - Check if DNS/hosts file is configured correctly
   - Verify Nginx is running: `sudo systemctl status nginx`
   - Check firewall: `sudo ufw status`

3. **SSL Certificate Issues**
   ```bash
   sudo openssl x509 -in /home/frappe/ssl/erp.insightse.com.cert -text -noout | grep DNS:
   ```

4. **Site Configuration Validation**
   ```bash
   python3 -m json.tool sites/<site-name>/site_config.json
   ```

5. **Nginx Configuration Test**
   ```bash
   sudo nginx -t
   ```

### Log Files

Monitor these log files for troubleshooting:
- Nginx access: `/var/log/nginx/access.log`
- Nginx error: `/var/log/nginx/error.log`
- Frappe logs: `~/frappe-bench/logs/`

## Security Considerations

1. **Database Security**
   - Use strong, unique passwords for each database
   - Regularly update database credentials
   - Limit database access to localhost only

2. **SSL Configuration**
   - Keep SSL certificates updated
   - Use strong SSL ciphers (already configured)
   - Enable HSTS headers (already configured)

3. **Firewall**
   - Only open necessary ports (80, 443, 22)
   - Consider using fail2ban for SSH protection
   - Regularly update the system

4. **Backup Strategy**
   - Regular database backups
   - Site files backup
   - SSL certificate backup

## Maintenance

### Regular Tasks

1. **Update Frappe/ERPNext**
   ```bash
   bench update
   ```

2. **Database Backup**
   ```bash
   bench --site <site-name> backup
   ```

3. **Monitor Logs**
   ```bash
   tail -f /var/log/nginx/error.log
   tail -f ~/frappe-bench/logs/web.error.log
   ```

4. **System Updates**
   ```bash
   sudo apt update && sudo apt upgrade
   ```

## Conclusion

This guide provides a complete setup for multi-tenant ERPNext hosting on a single DigitalOcean droplet. Each site operates independently with its own database and configuration while sharing server resources and SSL certificates.

The setup is production-ready with proper SSL configuration, security headers, and performance optimizations. Regular maintenance and monitoring will ensure optimal performance and security.

For additional customization or scaling requirements, consider:
- Load balancing for high traffic
- Database clustering for better performance
- CDN integration for static assets
- Automated backup solutions
- Monitoring and alerting systems
