# Brainforce Assessment
Automated deployment process for a WordPress website using Nginx as the web server, LEMP (Linux, Nginx, MySQL, PHP) stack, and GitHub Actions as the CI/CD automation tool.




```markdown
```

This guide outlines the steps to install a secured WordPress website on Ubuntu 22.04. This README consists of two major sections:

1. Setting up and configuring a secure Ubuntu 22.04 instance.
2. Setting up LEMP (Linux, Nginx, MySQL, PHP) stack for WordPress.

## Part 1: Setting up and configuring a secure Ubuntu 22.04 instance



### i. Login to the virtual private server

Use SSH to connect to your server:

```shell
ssh myuser@ipaddress
```
### ii. Keep the system up to date.

An extremely crucial part of hardening any system is to ensure that it is always kept up to date. Doing this will keep any known bugs or vulnerabilities patched.

```shell
apt-get update && apt-get upgrade
```

### iii. Harden SSH

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

Change the default SSH port (optional):

```shell
Port 49160
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

### iv. Install Artillery Honeypot for server monitoring and hardening

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
```
```

This Markdown format places the entire content within a single code block for easy readability.
