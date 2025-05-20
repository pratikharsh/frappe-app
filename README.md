# ERPNext Hosting Setup

This document outlines the steps to set up and host ERPNext using the provided commands.

## Prerequisites

Ensure you have a fresh Ubuntu server instance.

## Installation Steps

1.  *Update and Upgrade System Packages:*
    bash
    sudo apt-get update -y
    sudo apt-get upgrade -y
    

2.  *Add a Dedicated User for Frappe:*
    bash
    sudo adduser frappe
    
    (Follow the prompts to set a password and user information.)

3.  *Add Frappe User to the sudo Group:*
    bash
    sudo usermod -aG sudo frappe
    

4.  *Switch to the Frappe User:*
    bash
    su frappe
    

5.  *Install Python Development Tools and Pip:*
    bash
    sudo apt-get install python3-dev python3.10-dev python3-setuptools python3-pip python3-distutils
    

6.  *Install Python Virtual Environment:*
    bash
    sudo apt-get install python3.10-venv
    

7.  *Install software-properties-common:*
    bash
    sudo apt-get install software-properties-common
    

8.  *Install MariaDB Server and Client:*
    bash
    sudo apt install mariadb-server mariadb-client
    

9.  *Install Redis Server:*
    bash
    sudo apt-get install redis-server
    

10. *Install Dependencies for PDF Generation:*
    bash
    sudo apt-get install xvfb libfontconfig wkhtmltopdf
    

11. *Install MySQL Client Development Libraries:*
    bash
    sudo apt-get install libmysqlclient-dev
    

12. *Install curl:*
    bash
    sudo apt install curl
    

13. *Install Node Version Manager (NVM):*
    bash
    curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
    

14. *Source Your Profile to Load NVM:*
    bash
    source ~/.profile
    

15. *Install Node.js Version 18:*
    bash
    nvm install 18
    

16. *Install npm (Node Package Manager):*
    bash
    sudo apt-get install npm
    

17. *Install yarn Package Manager Globally:*
    bash
    sudo npm install -g yarn
    

18. *Install Frappe Bench:*
    bash
    sudo pip3 install frappe-bench
    

19. *Generate SSH Key Pair:*
    bash
    ssh-keygen
    
    (Press Enter for default file and no passphrase unless you have specific requirements.)

20. *Retrieve Public SSH Key:*
    bash
    cat /home/frappe/.ssh/id_rsa.pub
    
    (You might need this key to add to your Git repository settings for private repositories.)

21. *Initialize Frappe Bench with Your ERPNext App Repository:*
    bash
    bench init frappe-bench --frappe-path git@gitlab.com:kri.chandni007/frappe-app.git --frappe-branch version-15
    

22. *Navigate to the Bench Directory:*
    bash
    cd frappe-bench
    

23. *Create a New ERPNext Site:*
    bash
    bench new-site erp.insightse.com --db-host 142.93.211.172 --db-name erpnext-insightse-prod --db-root-username root --db-password 'root-password' --no-mariadb-socket --admin-password admin
    
    *Important:* Ensure the database host, name, username, and password are correct for your MariaDB setup.

24. *Use the Newly Created Site:*
    bash
    bench use erp.insightse.com
    

25. *Get the ERPNext Application:*
    bash
    bench get-app --branch version-15 erpnext
    

26. *Install the ERPNext Application on Your Site:*
    bash
    bench --site erp.insightse.com install-app erpnext
    

27. *Start the Frappe Bench Services:*
    bash
    bench start
    
## ERPNext Production Setup 

bash
# 1. Enable background jobs
bench --site erp.insightse.com enable-scheduler

# 2. Disable maintenance mode to allow access
bench --site erp.insightse.com set-maintenance-mode off

# 3. Set up production (Supervisor, Nginx config, etc.)
sudo bench setup production frappe

# 4. Generate and apply Nginx configuration
bench setup nginx

# 5. Restart all services via Supervisor
sudo supervisorctl restart all

# 6. (Optional) Re-run production setup if needed
sudo bench setup production frappe

## Setup custom domain and set the ssl

You have three files in your SSL directory:

* 9e98f83b222f061f.crt - This appears to be your domain certificate
* gd_bundle-g2-g1.crt - This is a certificate bundle containing three certificates (root and intermediate certificates)
* insightse.key - This is your private key file

For GoDaddy SSL certificates, you need to create a full certificate chain by combining your domain certificate with the certificate bundle.

## Step-by-Step Configuration

1.  *Create a combined certificate file:*
    bash
    cd ~/ssl
    cat 9e98f83b222f061f.crt gd_bundle-g2-g1.crt > erp.insightse.com.cert
    

2.  *Verify you have the correct files:*
    bash
    ls -la
    
    You should now see the new combined certificate file erp.insightse.com.cert

3.  *Add your domain to your ERPNext site with SSL certificates:*
    bash
    cd ~/frappe-bench
    bench setup add-domain erp.insightse.com \
      --ssl-certificate /home/frappe/ssl/erp.insightse.com.cert \
      --ssl-certificate-key /home/frappe/ssl/insightse.key
    
    When prompted, select your site (erp.insightse.com).

4.  *Ensure proper permissions for SSL certificates:*
    bash
    sudo chmod 644 ~/ssl/erp.insightse.com.cert
    sudo chmod 600 ~/ssl/insightse.key
    

5.  *Generate the Nginx configuration:*
    bash
    bench setup nginx
    

6.  *Test your Nginx configuration:*
    bash
    sudo nginx -t
    
    This will check for any syntax errors in your configuration.

7.  *Apply the changes by reloading Nginx:*
    bash
    sudo service nginx reload
    

8.  *Verify your site configuration:*
    bash
    cat sites/[erp.insightse.com/site_config.json](https://erp.insightse.com/site_config.json)
    
    Check that the domain and SSL certificate paths are correctly listed.

## Verification and Troubleshooting

*Test your domain with SSL:*
Open a browser and navigate to https://erp.insightse.com
Ensure the connection is secure (padlock icon in browser address bar).

*If you encounter any issues:*

* *Check Nginx error logs:*
    bash
    sudo tail -f /var/log/nginx/error.log
    

* *If SSL is not working correctly:*
    You might need to restart all services:
    bash
    sudo supervisorctl restart all
    sudo service nginx restart
    

## Certificate Renewal (Important)

Since you're using GoDaddy certificates, note that these typically expire after 1-2 years. When it's time to renew:

1.  Get the new certificate files from GoDaddy.
2.  Replace the existing files in your SSL directory.
3.  Run:
    bash
    cat new-certificate.crt gd_bundle-g2-g1.crt > erp.insightse.com.cert
    bench setup nginx
    sudo service nginx reload
    

## Accessing Your ERPNext Instance

Once the services are started, you should be able to access your ERPNext instance by navigating to http://erp.insightse.com in your web browser. Use the administrator password you set during the site creation (admin in this case).

## Important Considerations

* *Database Credentials:* Double-check the database host, name, username, and password used during the bench new-site command.
* *Firewall:* Ensure that your server's firewall allows traffic on the necessary ports (typically 80 for HTTP and 443 for HTTPS if you configure SSL).
* *Production Setup:* This setup uses the development server. For a production environment, you would typically configure a reverse proxy like Nginx and secure your site with HTTPS.
* *Custom App:* You have included a custom Frappe app (git@gitlab.com:kri.chandni007/frappe-app.git). Ensure this repository is accessible and contains a valid Frappe app.
