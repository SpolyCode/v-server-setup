# V-Server Setup

## Table of Contents

* [Prerequisites](#prerequisites)
* [Quickstart](#quickstart)
* [Usage](#usage)
* [Best Practices](#best-practices)
* [Troubleshooting](#troubleshooting)
* [Additional Informations](#additional-informations)

---

## Prerequisites

* SSH access to the target server
* A user account with **sudo privileges**
* Basic familiarity with the Linux terminal
* Installed packages:

  * `openssh-client`
  * `nginx`

> 💡 **Note:** Make sure your server’s firewall allows SSH (port 22) and HTTP (port 80).

---

## Quickstart

### 1. Generate and Install SSH Key

```bash
# Generate SSH key (ed25519 recommended)
ssh-keygen -t ed25519
```

```bash
# Copy public key to the server
ssh-copy-id username@server-ip
```

---

### 2. Disable Password Authentication (Optional, Recommended)

```bash
sudo nano /etc/ssh/sshd_config
```

Change or uncomment this line:

```
PasswordAuthentication no
```

Then restart the SSH service:

```bash
sudo systemctl restart ssh
```

> ⚠️ **Important:** Verify that SSH key login works **before** disabling password authentication. Otherwise, you may lose access.

---

### 3. Install and Configure NGINX

```bash
sudo apt update
sudo apt install nginx -y
```

Create a custom HTML file:

```bash
sudo touch /var/www/alternatives/alternate-index.html
sudo nano /var/www/alternatives/alternate-index.html
```

Edit the NGINX site configuration:

```bash
sudo nano /etc/nginx/sites-enabled/alternatives
```

Restart NGINX to apply changes:

```bash
sudo service nginx restart
```

> 💡 **Tip:** Test configuration before restarting:
>
> ```bash
> sudo nginx -t
> ```

---

### 4. Create a Custom SSH Alias

```bash
alias dal_connect="ssh -o StrictHostKeyChecking=False -i /Users/.ssh/id_ed25519 user@ip"
```

Now you can connect quickly using:

```bash
dal_connect
```

---

## Usage

* Connect to the server: `dal_connect`
* Customize your NGINX page:
  Edit `/var/www/alternatives/alternate-index.html`
* Check NGINX status:

  ```bash
  systemctl status nginx
  ```
* Restart or reload NGINX safely:

  ```bash
  sudo nginx -s reload
  ```

---

## Best Practices

* ✅ Keep your SSH private keys **secure** and never share them.
* ✅ Use `ufw` or another firewall tool to restrict access.
* ✅ Automate deployments using Git or Ansible when possible.
* ✅ Regularly update your server:

  ```bash
  sudo apt update && sudo apt upgrade -y
  ```

---

## Troubleshooting

| Issue                   | Possible Fix                        |
| ----------------------- | ----------------------------------- |
| Can't connect via SSH   | Check firewall and `sshd_config`    |
| NGINX won’t start       | Run `sudo nginx -t` to check syntax |
| Alias not found         | Add it to `~/.bashrc` or `~/.zshrc` |
| Custom page not showing | Verify site path and NGINX config   |

---

## Additional Informations

* Official NGINX docs: [https://nginx.org/en/docs/](https://nginx.org/en/docs/)
* Test web configuration using:

---
