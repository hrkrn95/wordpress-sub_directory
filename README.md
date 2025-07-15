# Zero to Hero: WordPress in a Subdirectory with Docker & Nginx Proxy Manager

A definitive guide to installing a WordPress website on a clean Ubuntu VPS, accessible in a subdirectory (`/blogs/`). This setup uses Apache as the backend web server, Docker to run Nginx Proxy Manager, and Nginx Proxy Manager as a secure reverse proxy to handle public traffic and SSL certificates.

---

## **Table of Contents**

1.  [Initial Server Hardening](#step-1-initial-server-hardening)
2.  [Install Apache & Database](#step-2-install-the-content-web-stack)
3.  [Create WordPress Database](#step-3-create-the-wordpress-database)
4.  [Install Nginx Proxy Manager](#step-4-install-the-proxy-server)
5.  [Place WordPress Files](#step-5-place-the-wordpress-files)
6.  [Configure Apache](#step-6-configure-apache)
7.  [Configure Nginx Proxy Manager](#step-7-configure-nginx-proxy-manager)
8.  [Final WordPress Configuration](#step-8-final-wordpress-configuration)
9.  [Verification & Troubleshooting](#step-9-verification--troubleshooting)

---

## **Step 1: Initial Server Hardening**

First, secure the server with a new user and a basic firewall.

### **How to connect & set up**

Connect to the server as **root** for the initial setup.

```bash
ssh root@YOUR_SERVER_IP
Use code with caution.
Markdown
Enter your root password when prompted.
Create a new user with sudo (administrative) privileges.
Generated bash
# Create the new user
adduser your_new_user

# Add the new user to the sudo group
usermod -aG sudo your_new_user
Use code with caution.
Bash
Configure the firewall (UFW) to allow essential services.
Generated bash
# Allow SSH connections
sudo ufw allow OpenSSH

# Allow the Nginx Proxy Manager admin port
sudo ufw allow 81/tcp

# Allow the port for our backend Apache server
sudo ufw allow 4042/tcp

# Enable the firewall
sudo ufw enable
Use code with caution.
Bash
Log out and reconnect as your new, non-root user. ssh your_new_user@YOUR_SERVER_IP
Step 2: Install the Content Web Stack
This is the software (Apache, MariaDB, PHP) that will run WordPress behind the proxy.
Install packages
Generated bash
# Refresh package lists and install all software
sudo apt update
sudo apt install apache2 mariadb-server php libapache2-mod-php php-mysql -y```

### **Secure the database**

Run the interactive security script and follow the prompts. It is **highly recommended** to set a strong root password.

```bash
sudo mysql_secure_installation
Use code with caution.
Bash
Step 3: Create the WordPress Database
WordPress needs its own dedicated database and user for security.
Log in to MariaDB
Generated bash
sudo mysql -u root -p
Use code with caution.
Bash
Create the database and user
Important: Replace 'your_strong_password' with a secure password you generate.
Generated sql
CREATE DATABASE wordpress_db;
CREATE USER 'wordpress_user'@'localhost' IDENTIFIED BY 'your_strong_password';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wordpress_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
Use code with caution.
SQL
Step 4: Install the Proxy Server
This will be our public-facing front door (Nginx Proxy Manager), handling all incoming traffic.
Install Docker
Generated bash
sudo apt install docker.io docker-compose -y
Use code with caution.
Bash
Configure and run NPM
Create a directory and a docker-compose.yml file to define the NPM service.
Generated bash
# Create a directory for NPM and navigate into it
mkdir ~/npm
cd ~/npm

# Create the compose file for editing
nano docker-compose.yml
Use code with caution.
Bash
Paste the following configuration into the docker-compose.yml file.
Generated yaml
version: '3'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '443:443'
      - '81:81'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
Use code with caution.
Yaml
Start the Nginx Proxy Manager container in the background.
Generated bash
sudo docker-compose up -d
Use code with caution.
Bash
Step 5: Place the WordPress Files
Download the latest version of WordPress and move it to the correct web directory.
Generated bash
# Create the final destination directory
sudo mkdir -p /var/www/html/blogs

# Navigate to a temporary directory for the download
cd /tmp

# Download and extract WordPress
wget https://wordpress.org/latest.tar.gz
tar -xzvf latest.tar.gz

# Move the extracted files (not the folder) to the destination
sudo mv /tmp/wordpress/* /var/www/html/blogs/

# Set correct ownership for the web server. This is a CRITICAL step.
sudo chown -R www-data:www-data /var/www/html/blogs
Use code with caution.
Bash
Step 6: Configure Apache
Tell Apache to listen on our custom port (4042) and how to find the WordPress files.
Update Apache ports
Open the ports configuration file for editing.
Generated bash
sudo nano /etc/apache2/ports.conf
Use code with caution.
Bash
Add the line Listen 4042 to the file and save.
Create a new site configuration
Create a new configuration file named after your domain.
Generated bash
sudo nano /etc/apache2/sites-available/your_domain.com.conf
Use code with caution.
Bash
Paste the following Virtual Host configuration. This uses an Alias to correctly map the URL path to the filesystem path.
Generated apache
<VirtualHost *:4042>
    ServerName your_domain.com
    ServerAlias www.your_domain.com

    DocumentRoot /var/www/html

    # This line explicitly maps the /blogs/ URL path to the WordPress directory
    Alias /blogs/ /var/www/html/blogs/

    # This <Directory> block now applies directly to the aliased path
    <Directory /var/www/html/blogs>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>```

### **Enable the new configuration**

```bash
# Enable the URL rewriting module, required by WordPress
sudo a2enmod rewrite

# Enable your new site configuration
sudo a2ensite your_domain.com.conf

# Disable the default welcome page to avoid conflicts
sudo a2dissite 000-default.conf
Use code with caution.
Apache
Test and restart Apache
Generated bash
# Check for syntax errors
sudo apache2ctl configtest

# Apply all changes by restarting the service
sudo systemctl restart apache2
Use code with caution.
Bash
Step 7: Configure Nginx Proxy Manager
Connect the outside world to your internal Apache server.
Log in to the Admin Panel by navigating to http://your_server_ip:81.
Default Login: admin@example.com / Password: changeme. You will be forced to change these credentials.
Add a Proxy Host: Go to Hosts -> Proxy Hosts and click Add Proxy Host.
In the Details Tab:
Domain Names: your_domain.com (and www.your_domain.com on a new line).
Scheme: http
Forward Hostname / IP: your_server_ip
Forward Port: 4042
Enable Block Common Exploits.
In the SSL Tab:
SSL Certificate: Select "Request a new SSL Certificate".
Enable Force SSL and HTTP/2 Support.
Agree to the Let's Encrypt Terms of Service.
Save the new host.
Add the Custom Location:
Edit the proxy host you just created.
Go to the Custom locations tab and click Add location.
Define location: /blogs/
Scheme: http
Forward Hostname / IP: your_server_ip
Forward Port: 4042
Save the location, then Save the host again.
Step 8: Final WordPress Configuration
Create the wp-config.php file and tell WordPress it's operating behind a reverse proxy.
Create the config file
Generated bash
sudo cp /var/www/html/blogs/wp-config-sample.php /var/www/html/blogs/wp-config.php
Use code with caution.
Bash
Edit the file
Generated bash
sudo nano /var/www/html/blogs/wp-config.php
Use code with caution.
Bash
First, fill in your database details (DB_NAME, DB_USER, DB_PASSWORD) from Step 3. Then, scroll to the bottom and paste the following code block ABOVE the line that says /* That's all, stop editing! */.
Generated php
/**
 * Fix for WordPress behind a reverse proxy
 */
if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
    $_SERVER['HTTPS'] = 'on';
}

if (isset($_SERVER['HTTP_X_FORWARDED_HOST'])) {
    $_SERVER['HTTP_HOST'] = $_SERVER['HTTP_X_FORWARDED_HOST'];
}

define('WP_HOME', 'https://your_domain.com/blogs');
define('WP_SITEURL', 'https://your_domain.com/blogs');
Use code with caution.
PHP
Step 9: Verification & Troubleshooting
Success: Navigate to https://your_domain.com/blogs/. You should see the WordPress installation wizard or login page, fully styled and secure.
Cloudflare SSL Error: If you see ERR_SSL_UNRECOGNIZED_NAME_ALERT, this is the most common issue when using Cloudflare.
Fix: Log in to your Cloudflare account, go to SSL/TLS -> Overview, and set the encryption mode to Full (Strict).
Broken Styles/CSS (Unstyled Page): This means WordPress doesn't know its correct HTTPS address.
Fix: Double-check that the reverse proxy code block in wp-config.php is present, correct, and in the right place (Step 8).
404 Not Found: This indicates a path mismatch.
Fix: Ensure the path /blogs/ is used consistently in three places: the Apache Alias directive, the NPM Custom location, and the WordPress WP_HOME/WP_SITEURL definitions.
