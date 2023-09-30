# Brainforce Assessment
Automated deployment process for a WordPress website using Nginx as the web server, LEMP (Linux, Nginx, MySQL, PHP) stack, and GitHub Actions as the CI/CD automation tool.



This guide outlines the steps to install a secured WordPress website on Ubuntu 22.04. 
This README consists of two major sections:

1. Setting up and configuring a secure Ubuntu 22.04 instance.
2. Setting up LEMP (Linux, Nginx, MySQL, PHP) stack for WordPress.

## Part 1: Setting up and configuring a secure Ubuntu 22.04 instance



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



