# V-Server Setup

This documentation describes the setup of a V-Server including SSH key authentication, mandatory deactivation of password login, and a basic NGINX configuration to serve a custom HTML page.  
It serves both as a step-by-step guide and as a reference for future deployments.

---

## 📚 Table of Contents
- [Prerequisites](#prerequisites)  
- [Quickstart](#quickstart)  
  - [1. Generate and Install SSH Key](#1-generate-and-install-ssh-key)  
  - [2. Disable Password Authentication](#2-disable-password-authentication)  
  - [3. Install and Configure NGINX](#3-install-and-configure-nginx)  
  - [4. Create a Custom SSH Alias](#4-create-a-custom-ssh-alias)  
- [Usage](#usage)  
- [Best Practices](#best-practices)  
- [Troubleshooting](#troubleshooting)  
- [Additional Information](#additional-information)

---

## 🧩 Prerequisites

Before starting the setup, ensure that you have:

- SSH access to the server  
- A user account with **sudo privileges**  
- Basic Linux terminal knowledge  
- Installed packages:
  - `openssh-client`
  - `nginx`

---

# 🚀 Quickstart

---

## 1. Generate and Install SSH Key

First, generate an SSH key pair on your local machine:

```bash
ssh-keygen -t ed25519
```

This generates two files:

| File | Description |
|------|-------------|
| `id_ed25519` | Private key → **must never be shared** |
| `id_ed25519.pub` | Public key → uploaded to the server |

Upload your public key to the server:

```bash
ssh-copy-id username@server-ip
```

This enables passwordless login via SSH keys.

---

## 2. Disable Password Authentication  
> **Required in this project:** password login must be disabled.

Before disabling password login:

- Ensure that SSH key login already works.  
- Keep a second terminal session open to avoid getting locked out while you change SSH settings.

Open the SSH config:

```bash
sudo nano /etc/ssh/sshd_config
```

Set or uncomment the following line:

```
PasswordAuthentication no
```

Save and exit the editor, then restart the SSH service:

```bash
sudo systemctl restart ssh
```

After this, the server will only allow login via SSH keys.

---

## 3. Install and Configure NGINX

### Install NGINX

```bash
sudo apt update
sudo apt install nginx -y
```

Check the service status:

```bash
systemctl status nginx
```

### Create a custom HTML file

```bash
sudo mkdir -p /var/www/alternatives
sudo nano /var/www/alternatives/alternate-index.html
```

Example content:

    <h1>Hello from your NGINX server!</h1>
    <p>This content is served from /var/www/alternatives.</p>

Make sure the files are readable by the `www-data` user (typical NGINX user):

```bash
sudo chown -R www-data:www-data /var/www/alternatives
sudo chmod -R 755 /var/www/alternatives
```

### Configure the NGINX site

Open or create the configuration file:

```bash
sudo nano /etc/nginx/sites-available/alternatives
```

Example configuration (paste into `/etc/nginx/sites-available/alternatives`):

```
server {
    listen 80;
    listen [::]:80;

    root /var/www/alternatives;
    index alternate-index.html;

    server_name _;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Enable the site by creating a symlink in `sites-enabled`:

```bash
sudo ln -s /etc/nginx/sites-available/alternatives /etc/nginx/sites-enabled/alternatives
```

Before restarting NGINX, test the configuration:

```bash
sudo nginx -t
```

If the test is successful, restart or reload NGINX:

```bash
sudo systemctl reload nginx
# or
sudo service nginx restart
```

---

## 4. Create a Custom SSH Alias

A custom alias makes connecting to the server much easier:

```bash
alias dal_connect="ssh -o StrictHostKeyChecking=False -i </path/to/your/ssh-key> user@server-ip"
```

Use it like this:

```bash
dal_connect
```

To make the alias persistent across sessions, add it to your shell config:

```bash
# For bash
echo 'alias dal_connect="ssh -o StrictHostKeyChecking=False -i </path/to/your/ssh-key> user@server-ip"' >> ~/.bashrc

# For zsh
echo 'alias dal_connect="ssh -o StrictHostKeyChecking=False -i </path/to/your/ssh-key> user@server-ip"' >> ~/.zshrc

# Then reload the file, e.g.
source ~/.bashrc
```

**NOTE:** Replace `</path/to/your/ssh-key>` and `user@server-ip` with the actual path to your private key and your server's user and IP address.

---

# 🛠 Usage

### Connect to the server:
```bash
dal_connect
```

### Edit your custom HTML page:
```bash
sudo nano /var/www/alternatives/alternate-index.html
```

### Reload NGINX configuration:
```bash
sudo nginx -s reload
```

### Check NGINX status:
```bash
systemctl status nginx
```

---

# 🔧 Best Practices

* Keep your private SSH key secure — treat it like a password.  
* Disable password authentication (mandatory for this project).  
* Keep your server updated:

```bash
sudo apt update && sudo apt upgrade -y
```

**What does this update?**

- System package lists  
- Security updates  
- Bug fixes  
- Kernel updates (if applicable)  
- Updates for all installed applications  

* Always test your NGINX configuration before reloading or restarting:

```bash
sudo nginx -t
```

* Use separate users for different services and restrict `sudo` access where possible.  
* Keep a log or changelog on the server (e.g., `/root/NOTES.md`) for configuration changes, installed packages, and important commands used.  
* Backup critical configuration files before editing:

```bash
sudo cp /etc/nginx/sites-available/alternatives /etc/nginx/sites-available/alternatives.bak
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

---

# 🐛 Troubleshooting

| Issue | Cause | Solution |
|-------|-------|---------|
| ❌ Cannot connect via SSH | Public key missing, wrong permissions, or `sshd_config` misconfigured | - Ensure public key exists in `~/.ssh/authorized_keys` on the server. - Set correct permissions: `chmod 700 ~/.ssh` and `chmod 600 ~/.ssh/authorized_keys`. - Check `/etc/ssh/sshd_config` for errors and restart SSH. |
| ❌ NGINX won’t start | Syntax error or missing files in configuration | - Run `sudo nginx -t` to see the error. - Check error logs at `/var/log/nginx/error.log`. |
| ❌ Alias not found | Alias not saved in shell startup file | - Add alias to `~/.bashrc` or `~/.zshrc` and run `source` on the file. |
| ❌ Custom HTML not displayed | Wrong root path, wrong index file, or permissions issue | - Verify `root` and `index` directives in site config. - Ensure file ownership/permissions allow NGINX to read the files. |
| ❌ Getting locked out after disabling password auth | SSH key login wasn't working before disabling | - If you still have a session open, revert `sshd_config` or re-add your key. - If locked out completely, use your hosting provider's recovery console or rebuild with correct keys. |

---

# 📘 Additional Information

Official NGINX documentation:  
https://nginx.org/en/docs/

Helpful commands reference:

```bash
# Check active SSH sessions
who

# View SSH server logs (Debian/Ubuntu)
sudo journalctl -u ssh -e
# or
sudo tail -n 200 /var/log/auth.log

# View NGINX logs
sudo tail -n 200 /var/log/nginx/error.log
sudo tail -n 200 /var/log/nginx/access.log
```

