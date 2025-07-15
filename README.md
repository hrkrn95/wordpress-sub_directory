```markdown
# 🚀 **Zero to Hero: WordPress in a Subdirectory with Docker & Nginx Proxy Manager**

A definitive guide to installing a **WordPress** website on a clean **Ubuntu VPS**, accessible in a subdirectory (`/blogs/`). This setup uses **Apache** as the backend web server, **Docker** to run **Nginx Proxy Manager**, and **Nginx Proxy Manager** as a secure reverse proxy to handle public traffic and SSL certificates.

---

## 📖 **Table of Contents**

1. [Initial Server Hardening](#step-1-initial-server-hardening)  
2. [Install Apache & Database](#step-2-install-the-content-web-stack)  
3. [Create WordPress Database](#step-3-create-the-wordpress-database)  
4. [Install Nginx Proxy Manager](#step-4-install-the-proxy-server)  
5. [Place WordPress Files](#step-5-place-the-wordpress-files)  
6. [Configure Apache](#step-6-configure-apache)  
7. [Configure Nginx Proxy Manager](#step-7-configure-nginx-proxy-manager)  
8. [Final WordPress Configuration](#step-8-final-wordpress-configuration)  
9. [Verification & Troubleshooting](#step-9-verification--troubleshooting)  

---

## 🔒 **Step 1: Initial Server Hardening**

First, secure the server with a new user and a basic firewall.

### **How to connect & set up**

1. Connect to the server as **root**:

   ```bash
   ssh root@YOUR_SERVER_IP
Create a new user with sudo privileges:

bash
Copy
Edit
adduser your_new_user
usermod -aG sudo your_new_user
Configure UFW to allow essential services:

bash
Copy
Edit
sudo ufw allow OpenSSH
sudo ufw allow 81/tcp       # Nginx Proxy Manager admin
sudo ufw allow 4042/tcp     # Apache backend
sudo ufw enable
Log out and reconnect as your new user:

bash
Copy
Edit
ssh your_new_user@YOUR_SERVER_IP
