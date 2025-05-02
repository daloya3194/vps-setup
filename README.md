# VPS Server Setup Guide with Ubuntu 22, NGINX, and SSL Configuration

This guide walks through the process of setting up a VPS server with Ubuntu 22, configuring NGINX with SSL, and installing necessary development tools.

## Table of Contents
- [User Management](#user-management)
- [SSH Key Generation](#ssh-key-generation)
- [Securing Your Server](#securing-your-server)
- [Installing PHP 8.4](#installing-php-84)
- [Installing Composer](#installing-composer)
- [Installing Node.js 22](#installing-nodejs-22)
- [Installing Docker](#installing-docker)
- [Installing NGINX](#installing-nginx)
- [Configuring SSL with Certbot](#configuring-ssl-with-certbot)
- [DNS Configuration](#dns-configuration)

## User Management

To create a new user with sudo privileges in Ubuntu:

```bash
# Create a new user
sudo adduser username

# Add the user to the sudo group
sudo usermod -aG sudo username
```

Replace "username" with your desired username.
The `adduser` command will prompt you to:
- Set a password for the new user
- Enter optional user information (full name, room number, etc.) - you can press Enter to skip these

To verify the user has sudo privileges:

```bash
# Check if the user is in the sudo group
groups username

# Or see all groups the user belongs to
id username
```

## SSH Key Generation

To generate an SSH key in Ubuntu:

```bash
# Generate an Ed25519 SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# To view your public SSH key
cat ~/.ssh/id_ed25519.pub
```

## Securing Your Server

### Update the System

Always start by updating your system:

```bash
sudo apt update
sudo apt upgrade -y
```

### Configure SSH

For better security, modify the SSH configuration:

1. Edit the SSH config file:
   ```bash
   sudo nano /etc/ssh/sshd_config
   ```

2. Make the following changes:
    - Disable root login: `PermitRootLogin no`
    - Change the default SSH port (optional): `Port 2222` (choose any port between 1024-65535)
    - Allow only specific users: `AllowUsers username`
    - Disable password authentication (if using SSH keys): `PasswordAuthentication no`

3. Restart the SSH service:
   ```bash
   sudo systemctl restart sshd
   ```

### Setup Firewall

Configure UFW (Uncomplicated Firewall):

```bash
# Install UFW if not already installed
sudo apt install ufw

# Allow SSH (replace 22 with your custom port if changed)
sudo ufw allow 22/tcp

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable the firewall
sudo ufw enable
```

## Installing PHP 8.4

```bash
# Add the Ondřej Surý PPA repository
sudo apt update
sudo apt install software-properties-common -y
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update

# Install PHP 8.4 and common extensions
sudo apt install php8.4 php8.4-cli php8.4-common php8.4-fpm -y

# Install additional commonly used PHP extensions
sudo apt install php8.4-mysql php8.4-xml php8.4-curl php8.4-mbstring php8.4-zip php8.4-gd php8.4-intl -y
sudo apt install php8.4-fileinfo php8.4-pdo php8.4-pgsql php8.4-sqlite3 -y

# Verify the installation
php -v

# Check PHP-FPM status
sudo systemctl status php8.4-fpm

# Enable PHP-FPM to start at boot
sudo systemctl enable php8.4-fpm
```

## Installing Composer

Install the latest version of Composer with PHP 8.4:

```bash
# 1. Update packages (just in case)
sudo apt update

# 2. Ensure you have required dependencies
sudo apt install -y curl unzip php8.4-cli php8.4-mbstring

# 3. Download the Composer installer script
curl -sS https://getcomposer.org/installer -o composer-setup.php

# 4. Verify integrity (optional but recommended)
HASH=$(curl -sS https://composer.github.io/installer.sig)
php8.4 -r "if (hash_file('sha384', 'composer-setup.php') === '$HASH') { echo '✔️ Verification successful' . PHP_EOL; } else { echo '❌ Verification failed' . PHP_EOL; unlink('composer-setup.php'); }"

# 5. Install Composer globally
sudo php8.4 composer-setup.php --install-dir=/usr/local/bin --filename=composer

# 6. Remove the installation file
rm composer-setup.php

# 7. Verify that Composer works with PHP 8.4
composer --version
```

> 💡 **Tip**: If you want Composer to always use PHP 8.4 even if it's not the default in `php -v`, you can create an alias in your `.bashrc` or `.zshrc`:
> ```bash
> alias composer='php8.4 /usr/local/bin/composer'
> ```

## Installing Node.js 22

```bash
sudo apt-get install -y curl
curl -fsSL https://deb.nodesource.com/setup_22.x -o nodesource_setup.sh
sudo -E bash nodesource_setup.sh
sudo apt-get install -y nodejs
node -v
```

## Installing Docker

```bash
# Update package lists
sudo apt update

# Install prerequisites
sudo apt install -y ca-certificates curl gnupg

# Create directory for keyrings if it doesn't exist
sudo mkdir -p /etc/apt/keyrings

# Download Docker's official GPG key and add to the keyring
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# Set proper permissions
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the Docker repository
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Update package lists again with the new repository
sudo apt update

# Install Docker
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add your user to the docker group to run Docker without sudo
sudo usermod -aG docker $USER

# Verify installation
sudo docker --version
 
# Start a new shell session with the docker group
newgrp docker

# Restart the Docker service
sudo systemctl restart docker

# Make sure your user was properly added to the docker group
sudo usermod -aG docker $USER

# Check if your user is in the docker group
groups
```

## Installing NGINX

```bash
# Update your package lists
sudo apt update

# Install NGINX
sudo apt install nginx -y

# Start NGINX service
sudo systemctl start nginx

# Enable NGINX to start at boot
sudo systemctl enable nginx

# Check the status to make sure it's running
sudo systemctl status nginx
```

You can also check if the firewall is active and allow NGINX through it:

```bash
# Check UFW status
sudo ufw status

# If UFW is active, allow NGINX
sudo ufw allow 'Nginx Full'
```

The 'Nginx Full' profile allows both HTTP (port 80) and HTTPS (port 443) traffic.

**Important NGINX configuration files:**
- Main config: `/etc/nginx/nginx.conf`
- Site configs: `/etc/nginx/sites-available/` and `/etc/nginx/sites-enabled/`
- Web root directory: `/var/www/html/`

## Configuring SSL with Certbot

### 1. Install Certbot and the NGINX plugin

```bash
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

### 2. Create a folder and an index.html file for your domain

```bash
# Create the directory structure
sudo mkdir -p /var/www/example.com/html

# Create the index.html file
sudo nano /var/www/example.com/html/index.html
```

Add the following content to your index.html (replace "example.com" with your domain):

```html
<!DOCTYPE html>
<html>
<head>
    <title>Welcome to example.com!</title>
</head>
<body>
    <h1>Success! Your NGINX server is running!</h1>
    <p>This is the default page for example.com</p>
</body>
</html>
```

### 3. Configure NGINX

Create a configuration file for your domain:

```bash
sudo nano /etc/nginx/sites-available/example.com
```

Add the following configuration (replace "example.com" with your domain):

```nginx
server {
    server_name example.com www.example.com;
    root /var/www/example.com/html;
    index index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Disable the default NGINX configuration:

```bash
sudo unlink /etc/nginx/sites-enabled/default
```

Enable your domain's configuration:

```bash
sudo ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### 4. Obtain an SSL certificate with Certbot

```bash
sudo certbot --nginx -d example.com -d www.example.com
```

### 5. Test automatic renewal

```bash
sudo certbot renew --dry-run
sudo systemctl reload nginx
```

### 6. Enable NGINX to start at boot

```bash
sudo systemctl enable nginx
```

### 7. Update your NGINX configuration after SSL validation

After the certificate is validated, update your configuration with proper redirects:

```bash
sudo nano /etc/nginx/sites-available/example.com
```

Replace the content with:

```nginx
# HTTP to HTTPS + www redirect
server {
    listen 80;
    server_name example.com www.example.com;

    # Redirect everything to https://www.example.com
    return 301 https://www.example.com$request_uri;
}

# HTTPS non-www to https://www.example.com redirect
server {
    listen 443 ssl;
    server_name example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    # Redirect to www
    return 301 https://www.example.com$request_uri;
}

# Main server: https://www.example.com
server {
    listen 443 ssl;
    server_name www.example.com;

    root /var/www/example.com/html;
    index index.html index.htm;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Then reload NGINX:

```bash
sudo systemctl reload nginx
```

## DNS Configuration

### Expected DNS Records

| Type | Name | Value |
|------|------|-------|
| A | @ | your.server.ip.address |
| A | www | your.server.ip.address |
| AAAA | @ | your:ipv6:address |
| AAAA | www | your:ipv6:address |

### Records to Keep

| Type | Keep? | Notes |
|------|-------|-------|
| A / AAAA | ✅ | Point to your server |
| NS / SOA | ✅ | Required, managed by your DNS provider |
| MX | ❓ | Keep if you use email with your domain |
| TXT | ❓ | Keep for email verification or Google verification |
| SRV | ❓ | Keep if you have an email service |
| CNAME | ❓ | Keep for email autoconfiguration |
| Others | ❌ | Can be removed if unused |

#### NS (Name Servers)
Keep these records (e.g., ns1.first-ns.de, etc.) if your provider manages your DNS.
Do not modify these entries unless you change registrars or delegate to another DNS provider.

#### SOA (Start of Authority)
Automatically managed by your DNS provider. Do not modify.

#### MX (Mail Exchange)
Keep if you want to receive emails with your domain (@example.com).
You can remove them if you don't intend to use email addresses with your domain.

#### TXT (SPF, DKIM, etc.)
These are verification records.
Keep the v=spf1 record if you want to send emails from this domain.
It's harmless to leave it even if you don't send emails.

#### SRV Records
Related to email server configuration (IMAP, SMTP, etc.).
Keep if you use an email provider.
Can be removed if you don't use email services.

#### CNAME Record
The autoconfig.example.com → mail.your-server.de record is used for email autoconfiguration.
Keep if you want to easily configure email clients.
Can be removed if not needed.
