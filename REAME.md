# Setting up Pi-hole with renewable certificates using Cloudflare - 2024 Update

This solution is adapated from a few configurations. This works with a Debian 12 LXC within Proxmox. References will be included.

1. Configuree a new LXC in Proxmox, there's plenty of instructions available. In a Proxmox LXC container, you're logged in as root by default.
You can add an account, but you need to install sudo. The commands are listed assuming you're using the root account.

2. Update the container

apt update
apt upgrade

3. (Optional) Recongifure the timezone - the default is UTC
dpkkg-reconfigure tzdata

5. Install curl
apt-get install curl

6. Install and configure Pi-hole and verify the web interface is working properly. Note, you need to set a static IP address.
curl -sSL https://install.pi-hole.net | bash

7. Install the following packages
   - lighttpd-mod-openssl (pihole doesn't install mod-openssl by default)
   - certbot
   - python3-certbot-dns-cloudflare




 
References
https://github.com/Gestas/Pi-Hole-TLS-with-Cloudflare
https://tech.borpin.co.uk/2019/03/22/letsencrypt-ssl-certificates-by-dns-challenge-with-lighttpd/
