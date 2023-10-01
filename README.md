# Brainstorm force Assessment
Automated deployment process for a WordPress website using Nginx as the web server, LEMP (Linux, Nginx, MySQL, PHP) stack, and GitHub Actions as the CI/CD automation tool..



This guide outlines the steps to install a secure WordPress website on Ubuntu 22.04. 
This README consists of two major sections:

1. Setting up and configuring a secure Ubuntu 22.04 instance.
2. Setting up LEMP (Linux, Nginx, MySQL, PHP) stack for WordPress.
3. CI/CD Configuration with Github Actions

## Part 1
Setting up and configuring a secure Ubuntu 22.04 instance



### Login to the virtual private server

Use SSH to connect to your server:

```shell
ssh myuser@ipaddress
```
### Keep the system up to date.

An extremely crucial part of hardening any system is to ensure that it is always kept up to date. Doing this will keep any known bugs or vulnerabilities patched.

```shell
apt-get update && apt-get upgrade
```


### Ensure Only Root Has UID of 0

Accounts that have a UID set to 0 have the highest access to a system. In most cases, this should only be the root account. Use the below command to list all accounts with a UID of 0:

```shell
awk -F: '($3=="0"){print}' /etc/passwd
```

### Adding New User Accounts

It’s best practice to keep the use of the root account to a minimum. To do this, add a new account that will be primarily used with the command below:

```shell
adduser wpadmin
```
Add wpadmin to sudo group

```shell
usermod -aG sudo wpadmin
```

### Sudo Configuration

The sudo package allows a regular user to run commands in an elevated context. This means a regular user can run commands normally restricted to the root account. Often, this is the ideal way of making system configurations or running elevated commands – not by using the root account. The configuration file for sudo is in /etc/sudoers. The file can  be edited by using the “visudo” command. Add the following configuration below:

```shell
wpadmin ALL=(ALL) NOPASSWD:ALL
```
### Harden SSH

Linux servers are often administered remotely using SSH, making securing the OpenSSH server crucial. These initial hardening configurations enhance server security.

Create a backup of the default SSH config file:

```shell
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

Open the SSH configuration file:

```shell
sudo nano /etc/ssh/sshd_config
```

Disable SSH login for the root user:

```shell
PermitRootLogin no
```

Change the default SSH port:

```shell
Port 3355
```

Limit the maximum number of authentication attempts:

```shell
MaxAuthTries 3
```

Set a reduced login grace period:

```shell
LoginGraceTime 20
```

Disable SSH password authentication:

```shell
PasswordAuthentication no
```

Allow Specific Users

This line will allow you to specify which users can log into the SSH service:

```shell
AllowUsers wpadmin
```

Disable authentication with empty passwords:

```shell
PermitEmptyPasswords no
```

Save and exit the file.

Validate the syntax of the new configuration:

```shell
sudo sshd -t
```

Reload SSH to apply the new settings:

```shell
sudo systemctl reload sshd.service
```

### Replace password authentication with SSH keys.

Given enough time, any password can be cracked with a brute force attack. To prevent this,  switch to using SSH keys instead.

First, create a new SSH key pair for wpadmin.

```shell
sudo su - wpadmin
ssh-keygen
```

## Enable the Firewall

This is used to protect our server’s unused ports with a firewall solution such as Uncomplicated Firewall (UFW). This firewall comes preinstalled on Ubuntu, but it’s disabled by default.

```shell
sudo ufw enable 
```
Configure the firewall to allow HTTP, HTTPS and SSH connections.

```shell
sudo ufw allow http
sudo ufw allow https
sudo ufw allow 3322/tcp
```

### Install Artillery Honeypot for server monitoring and hardening

Artillery is a multi-purpose defense tool for Linux-based systems with honeypot capabilities, OS hardening, file system monitoring, and real-time threat analysis.

#### Features:

- Sets up multiple common ports that are attacked.
- Monitors specified folders for modifications (Linux only).
- Monitors SSH logs and looks for brute force attempts (Linux only).
- Sends email notifications when attacks occur.

#### Installation steps

a. Clone Artillery Honeypot GitHub repository:

```shell
git clone https://github.com/BinaryDefense/artillery.git
```

b. Change directory to Artillery and launch the installer:

```shell
cd artillery
python3 setup.py
```

Answer 'yes' to prompts during installation.

c. Configuring Artillery:

Edit the config file with nano:

```shell
nano /var/artillery/config
```

Change the following line to enable file system monitoring for custom directories:

```shell
MONITOR_FOLDERS="/var/www","/etc","/root"
```

Enable automatic updates:

```shell
AUTO_UPDATE=ON
```

Configure DoS protection for specific ports:

```shell
ANTI_DOS_PORTS=80,443,8080,8180,10000
```

d. Maintaining Artillery:

Artillery runs as a service. It starts automatically on server boot and can be managed with systemctl:

```shell
sudo systemctl start artillery  # Starts the service.
sudo systemctl restart artillery  # Restarts the service.
```

## Conclusion

This section reviews the OpenSSH server configuration and implements various hardening measures to secure the server.


## Part 2

 Setting up LEMP (Linux, Nginx, MySQL, PHP) stack for WordPress.

### Update Packages
```shell
sudo apt update && sudo apt upgrade -y
```

### Install WordPress dependencies: Nginx & Certbot:
```shell
sudo apt install nginx certbot python3-certbot-nginx -y 
```
### Nginx Configuration
Create nginx configuration file and paste content below:

sudo nano /etc/nginx/sites-available/wordpress

```shell

server {
    listen 80;
    server_name websiteurl.com www.websiteurl.com;

    root /var/www/html/wordpress;
    index index.php;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name; 
    }

    location ~ /\.ht {
        deny all;
    }

    location = /favicon.ico {
        log_not_found off;
        access_log off;
    }

    location = /robots.txt {
        allow all;
        log_not_found off;
        access_log off;
    }

    location ~* \.(css|gif|ico|jpeg|jpg|js|png)$ {
        expires max;
        log_not_found off;
    }

    # Additional security headers
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Xss-Protection "1; mode=block" always;

    # Enable Gzip compression
    gzip on;
    gzip_disable "msie6";
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    # Cache control settings (adjust cache durations as needed)
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 1y;
    }

    # Cache static assets aggressively
    location ~* \.(pdf|html|swf)$ {
        expires 7d;
    }

    # Cache dynamic content (adjust cache durations as needed)
    location ~* \.(xml|json|rss|atom|gz|zip)$ {
        expires 1h;
        add_header Cache-Control "public, max-age=3600";
    }

    # Cache static content with query strings (adjust cache durations as needed)
    location ~* \.(css|js|png|gif|jpg|jpeg|swf|xml|json|txt|ttf|woff|woff2|svg|eot)$ {
        expires 30d;
        add_header Cache-Control "public, max-age=2592000";
    }
}

```

save and exit file editor. <br>
Run the following commands to link the configuration file in the sites-enabled directory, test configuration and reload nginx.

```shell
sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled

sudo nginx -t

sudo systemctl reload nginx
```
Enable SSL/TLS with Lets Encypt:

```shell
sudo certbot --nginx -d websiteurl.com www.websiteurl.com
```

### Install PHP and optional dependencies:
```shell
sudo apt install php8.1-fpm php-mysql
sudo apt install php-curl php-gd php-intl php-mbstring php-soap php-xml php-xmlrpc php-zip
```
### Install WordPress dependencies: MYSQL:
```shell
sudo apt install mysql-server -y
```
The installation process might ask you to acknowledge some configuration steps for the MySQL server.

You will be asked to choose the default authentication plugin you are comfortable using.

### Database Configuration:
```shell
mysql_secure_installation
```
The above command  helps create the root user password for  MySQL database alongside other important database configuration settings.

Login to MySQL shell using the created root user credentials:
```shell
sudo mysql -u root -p
```
Create a WordPress database and user.
```shell
mysql> CREATE DATABASE wordpress;
mysql> CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'pa55w0rd';
mysql> GRANT ALL ON wordpress.* TO 'wpuser'@'localhost';
mysql> FLUSH PRIVILEGES;
mysql> exit;
```
### Download WordPress
```shell
cd /var/www/html
sudo wget http://wordpress.org/latest.tar.gz
sudo tar -xzvf latest.tar.gz
```
### Configure website directory permissions
```shell
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo chmod -R 775 /var/www/html/wordpress 
```
### Setup the WordPress Configuration file
Edit secret keys to enhance the security of our website. WordPress provides a secure generator for these values, which can be gotten using the following commands:
```shell
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```
Next, Open the WordPress configuration file:
```shell
sudo mv wp-config-sample.php wp-config.php
sudo nano wp-config.php
```
Update the section that contains the dummy values for those settings
```shell
define('AUTH_KEY',         'VALUES COPIED FROM THE COMMAND LINE');
define('SECURE_AUTH_KEY',  'VALUES COPIED FROM THE COMMAND LINE');
define('LOGGED_IN_KEY',    'VALUES COPIED FROM THE COMMAND LINE');
define('NONCE_KEY',        'VALUES COPIED FROM THE COMMAND LINE');
define('AUTH_SALT',        'VALUES COPIED FROM THE COMMAND LINE');
define('SECURE_AUTH_SALT', 'VALUES COPIED FROM THE COMMAND LINE');
define('LOGGED_IN_SALT',   'VALUES COPIED FROM THE COMMAND LINE');
define('NONCE_SALT',       'VALUES COPIED FROM THE COMMAND LINE');
```
Next, update database connection parameters:
```shell
define( 'DB_NAME', 'wordpress' );

/** MySQL database username */
define( 'DB_USER', 'wpuser' );

/** MySQL database password */
define( 'DB_PASSWORD', 'pa55w0rd' );
```
### Complete installation via web interface
In your web browser, navigate to your server’s domain name or public IP address: <br>
Select the language: <br>
<br>
<p align="center">
  <img src="/screenshots/language.png">
</p>
Next, on the main setup page, enter the WordPress site name  and choose a secure username and password.

Enter an email address and select whether you want to discourage search engines from indexing your site. <br>
<br>
<p align="center">
  <img src="/screenshots/config.png">
</p>

Once you log in, you will be taken to the WordPress administration dashboard:
<p align="center">
  <img src="/screenshots/dashboard.png">
</p>


## Part 3
CI/CD Configuration with GitHub Actions

This section outlines how to set up automated deployment using GitHub Actions. <br>
In other to achieve this, we'll be using Ansible to automate the setup of WordPress dependencies (PHP, Nginx & MYSQL).

### Ansible Directory Structure

```shell
ansible/
├── install-lemp.yml
├── inventory
├── README.md
├── roles
│   ├── mysql
│   │   ├── defaults
│   │   │   └── main.yml
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   │       └── my.cnf.j2
│   ├── nginx
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── tasks
│   │   │   └── main.yml
│   │   └── templates
│   │       └── default.conf
│   ├── php
│   │   ├── handlers
│   │   │   └── main.yml
│   │   └── tasks
│   │       └── main.yml
│   └── wordpress
│       └── tasks
│           └── main.yml


```
The above contains ansible scripts that ensure a LEMP stack is installed and configured to deploy our WordPress website.

### GitHub Actions Setup

In a local terminal navigate to the project directory to create a config file:

```shell
nano .github/workflows/deploy.yml
```
Paste in the content below:

```shell
name: Deploy WordPress Website

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up SSH key
        uses: webfactory/ssh-agent@v0.5.3
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      - name: Install Ansible
        run: |
          sudo apt-get update
          sudo apt-get install -y ansible
        become: true

      - name: Deploy and Configure LEMP Stack
        run: |
          ansible-playbook -i inventory deploy.yml
        become: true
        env:
          ANSIBLE_HOST_KEY_CHECKING: False
          
      - name: Update WordPress Files
        run: |
          ssh -i ${{ secrets.SSH_PRIVATE_KEY }} ${{ secrets.SSH_USER }}@${{ secrets.IP_ADDRESS }} "cd /var/html/wordpress && git pull"

```
### Conclusion

When Github action is triggered by a push or commit to the main branch it runs an ansible playbook (deploy.yml), this playbook checks if the LEMP stack (Nginx, MySQL, PHP) is already installed. <br>

This playbook sets a variable (lemp_installed) to true if the stack is installed and then the GitHub action only updates the content of the WordPress site. However, if the variable (lemp_installed) returns false, GitHub actions run a supplementary Ansible playbook (install-lemp.yml) that installs and configures LEMP stack on the server.
