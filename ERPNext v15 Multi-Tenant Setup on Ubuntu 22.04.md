# ERPNext v15 Multi-Tenant Setup on Ubuntu 22.04 🛠️

> Comprehensive guide to install and configure ERPNext v15 with multi-tenancy, SSL, Nginx, and Supervisor on Ubuntu 22.04.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Initial Server Setup](#initial-server-setup)
3. [Install Dependencies](#install-dependencies)
4. [Configure MariaDB](#configure-mariadb)
5. [Install Node.js & Yarn](#install-nodejs--yarn)
6. [Setup Frappe Bench](#setup-frappe-bench)
7. [Create ERPNext Sites](#create-erpnext-sites)
8. [Configure Supervisor](#configure-supervisor)
9. [Configure Nginx & SSL](#configure-nginx--ssl)
10. [Verification & Troubleshooting](#verification--troubleshooting)
11. [Backup Strategy](#backup-strategy)

---

## Prerequisites 🎯

- Ubuntu 22.04 LTS server (DigitalOcean droplet)
- 4GB+ RAM recommended
- Public IP (e.g., `64.227.181.172`)
- DNS A records for `erp1.insightse.com` and `erp2.insightse.com` pointing to your server IP
- User with sudo privileges (e.g., `ubuntu`)
- SSL certificates (will use Let's Encrypt in this guide)

---

## Initial Server Setup 🛠️

