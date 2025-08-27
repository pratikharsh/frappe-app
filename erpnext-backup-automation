# ERPNext v15 Backup to DigitalOcean Spaces Guide

Automate your ERPNext v15 backups to DigitalOcean Spaces (S3-compatible) with zero code changes. This guide walks you through the full process—from Spaces setup to custom scheduling and troubleshooting.

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Step 1: Create DigitalOcean Space](#step-1-create-digitalocean-space)
- [Step 2: Configure ERPNext S3 Backup Settings](#step-2-configure-erpnext-s3-backup-settings)
- [Step 3: Test Your Backup](#step-3-test-your-backup)
- [Step 4: Troubleshooting](#step-4-troubleshooting)
- [Step 5: Customize Backup Schedule](#step-5-customize-backup-schedule)
- [Step 6: Monitor and Maintain Backups](#step-6-monitor-and-maintain-backups)
- [Step 7: Cost Optimization Tips](#step-7-cost-optimization-tips)
- [Security Best Practices](#security-best-practices)
- [Support and Resources](#support-and-resources)

---

## Overview

This documentation provides a step-by-step guide to configuring ERPNext v15 backups using DigitalOcean Spaces — a cost-effective, scalable, and S3-compatible object storage solution.

---

## Prerequisites

- ERPNext v15 installed and running
- DigitalOcean account with Spaces enabled
- Admin access to your ERPNext instance

---

## Step 1: Create DigitalOcean Space

### 1.1 Create Your Space

1. Login to the [DigitalOcean Control Panel](https://cloud.digitalocean.com/)
2. Navigate to **Spaces** from the sidebar
3. Click **Create a Spaces Bucket**
4. Choose your datacenter region (e.g., **BLR1** for Bangalore recommended for India-based setups)
5. Enter a unique bucket name (3 to 63 characters, lowercase)
    ```
    Example: mycompany-erpnext-backup
    ```
6. Optionally, enable the **Content Delivery Network (CDN)**
7. Select your project and click **Create a Spaces Bucket**

### 1.2 Generate API Credentials

1. Go to **API** in the DigitalOcean Control Panel
2. Click the **Spaces Keys** tab
3. Click **Generate New Key**
4. Enter a name (e.g., `ERPNext Backup Access`)
5. Copy and safely store:
    - Access Key ID
    - Secret Access Key

---

## Step 2: Configure ERPNext S3 Backup Settings

### 2.1 Access the Settings

1. In ERPNext, press `Ctrl + G` and search for **S3 Backup Settings**
2. Select or create a new configuration

### 2.2 Fill in Required Fields

| Field                    | Value                                | Example                          |
|--------------------------|------------------------------------|---------------------------------|
| Enable Automatic Backup  | ✔ Check                            | ✔                              |
| Access Key ID            | Your DigitalOcean Spaces Access Key| `DO80IBVAVXWWU8FFQWQR`          |
| Access Key Secret        | Your DigitalOcean Secret Key       | `your-secret-key-here`           |
| Bucket Name              | Your Space bucket name             | `mycompany-erpnext-backup`       |
| Endpoint URL             | DigitalOcean Spaces endpoint       | `https://blr1.digitaloceanspaces.com` |
| Send Notifications To    | Your email                        | `admin@yourcompany.com`          |
| Backup Frequency         | Select: Daily / Weekly / Monthly   | `Daily`                         |
| Backup Files             | ✔ Check to include files           | ✔                              |

### 2.3 Region-Specific Endpoint URLs

| Region | Endpoint URL                          |
|--------|-------------------------------------|
| BLR1 (Bangalore) | `https://blr1.digitaloceanspaces.com` |
| NYC3 (New York)  | `https://nyc3.digitaloceanspaces.com`  |
| SFO3 (San Francisco) | `https://sfo3.digitaloceanspaces.com` |
| AMS3 (Amsterdam) | `https://ams3.digitaloceanspaces.com` |
| SGP1 (Singapore) | `https://sgp1.digitaloceanspaces.com` |
| FRA1 (Frankfurt) | `https://fra1.digitaloceanspaces.com` |

### 2.4 Save

Click **Save** and the configuration will be applied.

---

## Step 3: Test Your Backup

### 3.1 Manual Backup

- In S3 Backup Settings, click **Take Backup Now**
- Wait for backup completion message

### 3.2 Verify in DigitalOcean Spaces

- Navigate to your Space in DigitalOcean Console
- Check for backup files named like:
  - `database-backup-YYYY-MM-DD-HH-MM-SS.sql.gz`
  - `files-backup-YYYY-MM-DD-HH-MM-SS.tar`
  - `site_config_backup.json`

---

## Step 4: Troubleshooting

### Bucket Not Found Error

- Ensure your **bucket name** matches exactly (case-sensitive)
- Use endpoint without bucket name, e.g. `https://blr1.digitaloceanspaces.com`
- Try S3-style endpoint: `https://s3-blr1.digitaloceanspaces.com`

### No Region Selection Field

- Region is implicitly set; use `us-east-1` in internal configs
- Optionally edit `site_config.json` with S3 params (see advanced section)

---

## Step 5: Customize Backup Schedule

### Change Backup Time Without Coding

1. Search for **Scheduler Settings** in ERPNext
2. Find “Daily Backup” (or Weekly/Monthly)
3. Adjust the **Cron Expression** for desired backup time  
   Example: For 2:30 AM daily  
