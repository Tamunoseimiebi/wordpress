# Brainstorm force Assessment
Automated deployment process for a WordPress website using Nginx as the web server, LEMP (Linux, Nginx, MySQL, PHP) stack, and GitHub Actions as the CI/CD automation tool.



This guide outlines the steps to install a secure WordPress website on Ubuntu 22.04. 
This README consists of two major sections:

1. Setting up and configuring a secure Ubuntu 22.04 instance.
2. Setting up LEMP (Linux, Nginx, MySQL, PHP) stack for WordPress.

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

### Install WordPress dependencies: Nginx:
```shell
sudo apt install nginx -y
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
mysql> GRANT ALL PRIVILEGES ON wordpress;
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
![Alt text](/screenshots/language.jpg?raw=true "select language")

Next, on the main setup page, enter the WordPress site name  and choose a secure username and password.

Enter an email address and select whether you want to discourage search engines from indexing your site. <br>
![Alt text](/screenshots/language.jpg?raw=true "select language")

Once you log in, you will be taken to the WordPress administration dashboard:
