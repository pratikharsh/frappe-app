# ERPNext v15 Multi-Tenant Deployment on Ubuntu 22.04 📋

A comprehensive, step-by-step guide to install, configure, and run two ERPNext v15 sites (`erp1.insightse.com` & `erp2.insightse.com`) with SSL on a DigitalOcean Ubuntu 22.04 droplet.

---

## Prerequisites ✔️

• Ubuntu 22.04 LTS droplet (DigitalOcean)
• Minimum 4 GB RAM, 40 GB disk
• Domains pointed: `erp1.insightse.com`, `erp2.insightse.com` → 64.227.181.172
• User: `ubuntu` with sudo access
• SSL certificates (Let’s Encrypt or custom) for both domains

---

## 1. Initial Server Setup 🔧

1. Update & reboot:
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo reboot
   ```
2. Create `ubuntu` user & grant sudo:
   ```bash
   sudo adduser ubuntu
   sudo usermod -aG sudo ubuntu
   ```
3. (Optional) Set timezone:
   ```bash
   sudo timedatectl set-timezone Asia/Kolkata
   ```

---

## 2. Install Dependencies 📦

```bash
sudo apt install -y git python3-dev python3.10-dev python3-pip python3-distutils python3-venv software-properties-common
sudo apt install -y mariadb-server redis-server xvfb libfontconfig wkhtmltopdf libmysqlclient-dev
```

---

## 3. Secure & Configure MariaDB 🔒

```bash
sudo mysql_secure_installation
```
Answer prompts:
- Switch to unix_socket authentication: **Y**
- Change root password: **Y** (set strong password)
- Remove anonymous: **Y**
- Disallow root remote: **N**
- Remove test DB: **Y**
- Reload privileges: **Y**

Edit `/etc/mysql/my.cnf`:
```ini
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4
```
```bash
sudo service mysql restart
```

---

## 4. Install Node.js & Yarn ⚙️

```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g npm yarn
```

---

## 5. Install Frappe Bench & Init 🔨

```bash
sudo pip3 install frappe-bench
bench init frappe-bench --frappe-branch version-15
cd frappe-bench
``` 

---

## 6. Create ERPNext Sites 🏗️

### Site 1: erp1.insightse.com
```bash
bench new-site erp1.insightse.com
```
Enter DB root password and set Administrator password.

### Site 2: erp2.insightse.com
```bash
bench new-site erp2.insightse.com
```

Install ERPNext & HRMS:
```bash
bench get-app --branch version-15 erpnext
bench get-app --branch version-16 hrms
bench --site erp1.insightse.com install-app erpnext
bench --site erp1.insightse.com install-app hrms
bench --site erp2.insightse.com install-app erpnext
bench --site erp2.insightse.com install-app hrms
```

---

## 7. Production Setup 🏭

```bash
bench setup production ubuntu
``` 
This sets up Supervisor, systemd, and Nginx config templates.

Make sure Supervisor processes show **RUNNING**:
```bash
sudo supervisorctl status
```

---

## 8. Nginx & SSL Configuration 🔐

Place all site configs in a single `/etc/nginx/conf.d/frappe-bench.conf`:
```bash
sudo nano /etc/nginx/conf.d/frappe-bench.conf
```
Paste the unified config (includes upstreams, HTTP→HTTPS, SSL vhosts for both sites).

Verify cert/key match:
```bash
sudo openssl x509 -noout -modulus -in /etc/nginx/conf.d/ssl/erp1.insightse.com.crt | openssl md5
sudo openssl rsa   -noout -modulus -in /etc/nginx/conf.d/ssl/erp1.insightse.com.key | openssl md5
``` 
(both hashes should match; repeat for `erp2`).

```bash
sudo nginx -t
sudo systemctl reload nginx
``` 

Confirm listening:
```bash
sudo ss -tulnp | grep nginx
``` 
Should list ports **80** & **443**.

---

## 9. Final Verification ✅

- **Internal**:
  ```bash
  curl -I http://127.0.0.1 --resolve erp1.insightse.com:80:127.0.0.1
  curl -I https://127.0.0.1 --resolve erp2.insightse.com:443:127.0.0.1
  ```
- **External**: Navigate to
  - https://erp1.insightse.com
  - https://erp2.insightse.com

Complete the setup wizard on each site.

---

## 10. Maintenance & Backup 🔄

- Schedule daily backups:
  ```bash
  bench --site erp1.insightse.com backup
  bench --site erp2.insightse.com backup
  ```
- Monitor logs:
  - Frappe: `~/frappe-bench/logs/`
  - Nginx: `/var/log/nginx/`
- Renew Let's Encrypt SSL automatically:
  ```bash
  sudo certbot renew --dry-run
  ```

---

🎉 **Your multi-tenant ERPNext v15 + HRMS setup on Ubuntu 22.04 is ready!** 🎉
