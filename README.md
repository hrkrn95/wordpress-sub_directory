# Zero to Hero: WordPress in a Subdirectory with Docker & Nginx Proxy Manager

A definitive guide to installing a WordPress website on a clean Ubuntu VPS, accessible in a subdirectory (`/blogs/`).

This setup uses Apache as the backend web server, Docker to run Nginx Proxy Manager, and Nginx Proxy Manager as a secure reverse proxy to handle public traffic and SSL certificates.

---

## Table of Contents

1.  [Initial Server Hardening](#step-1-initial-server-hardening)
2.  [Install Apache & Database](#step-2-install-the-content-web-stack-apache-mariadb-php)
3.  [Create WordPress Database](#step-3-create-the-wordpress-database)
4.  [Install Nginx Proxy Manager](#step-4-install-the-proxy-server-docker--nginx-proxy-manager)
5.  [Place WordPress Files](#step-5-place-the-wordpress-files)
6.  [Configure Apache](#step-6-configure-apache-for-port-4042-and-the-blogs-path)
7.  [Configure Nginx Proxy Manager](#step-7-configure-nginx-proxy-manager)
8.  [Final WordPress Configuration](#step-8-final-wordpress-configuration)
9.  [Verification & Troubleshooting](#verification--troubleshooting)

---

## Step 1: Initial Server Hardening

First, secure the server with a new user and a basic firewall.

### How to connect & set up

Connect to the server as `root` for the initial setup.

```bash
ssh root@YOUR_SERVER_IP```

> Enter your root password when prompted.

Create a new user with `sudo` (administrative) privileges.

```bash
# Create the new user
adduser your_new_user

# Add the new user to the sudo group
usermod -aG sudo your_new_user

Configure the firewall (UFW) to allow essential services.
# Allow SSH connections
sudo ufw allow OpenSSH

# Allow the Nginx Proxy Manager admin port
sudo ufw allow 81/tcp

# Allow the port for our backend Apache server
sudo ufw allow 4042/tcp

# Enable the firewall
sudo ufw enable```

> Log out and reconnect as your new, non-root user.
> `ssh your_new_user@YOUR_SERVER_IP`

---

## Step 2: Install the "Content" Web Stack (Apache, MariaDB, PHP)

This is the software that will run WordPress behind the proxy.

### Install packages

```bash
# Refresh package lists and install software
sudo apt update
sudo apt install apache2 mariadb-server php libapache2-mod-php php-mysql -y
