# V-Server Setup Documentation

## Creating the SSH Key

First, I generated an SSH key on my computer:

ssh-keygen -t ed25519

Next, I copied my public key to the server with:

ssh-copy-id username@server-ip

Disabling Password Login

First, I made sure SSH key login works. Then, I edited /etc/ssh/sshd_config and changed:

PasswordAuthentication no

After that, I restarted SSH:
sudo systemctl restart

I installed NGINX:
sudo apt update
sudo apt install nginx -y

To customize the site, I created a new index file:
sudo touch /var/www/alternatives/alternate-index.html

sudo nano /var/www/alternatives/alternate-index.html

Then, I edited the NGINX site configuration:

sudo nano /etc/nginx/sites-enabled/alternatives

After making the changes, I restarted NGINX:
sudo service nginx restart

I added a custom alias to my shell configuration:

alias dal_connect="ssh -o StrictHostKeyChecking=False -i /Users/.ssh/id_ed25519 user@ip

With this alias, I could connect directly to the server by running:
dal_connect
