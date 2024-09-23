### Setting up Pi-hole with renewable certificates using Cloudflare - 2024 Update

This solution is adapted from a few older configurations. This works with a Debian 12 LXC within Proxmox. References will be included in the end.
This is not a fully detailed document as this is meant to be short. I can write a full document in another location by request.

1. Configure a new LXC in Proxmox, there's plenty of instructions available. In a Proxmox LXC container, you're logged in as root by default.
You can add an account, but you need to install sudo. The commands are listed assuming you're using the root account.

2. Update the container

```
   # apt update
   # apt upgrade
```

3. (Optional) Reconfigure the timezone - the default is UTC

```
   # dpkkg-reconfigure tzdata
```

4. Install curl

```
   # apt-get install curl
```

5. Install and configure Pi-hole and verify the web interface is working properly. Note, you need to set a static IP address.  

```
   # curl -sSL https://install.pi-hole.net | bash
```

6. Install the following packages:
   - lighttpd-mod-openssl (pi-hole doesn't install mod-openssl by default)
   - certbot
   - python3-certbot-dns-cloudflare

```
   # apt-get install certbot lighttpd-mod-openssl python3-certbot-dns-cloudflare
```

7. Get your API key from Cloudflare if you haven't done it already.

8. Pick a location to store your ini file. I suggest creating a .directory so it's not easily visible. The parent directory names doesn't matter. I like keep config files in /etc.

```
   # mkdir -p /etc/custom-config/.secrets
```

9. Create and edit the ini file for your Cloudflare information. I use vi, you can use any editor you're comfortable with.

```
   # vi /etc/custom-config/.secrets/cloudflare.ini 
```

10. Add this single line into cloudflare.ini

```
   dns_cloudflare_api_token = (Your API token from Cloudflare)
```

12. Save the file and change the permission of cloudflare.ini file to 600

```
  # chmod 600 /etc/custom-config/.secrets/cloudflare.ini
```

11. Transfer the script to your local machine. It will create your initial certificates, set up automatic certificate renewal and output the file external.conf that won't be overwritten whenever Pi-hole is updated.

```
   # wget https://github.com/maalth/Pihole-config-https/raw/main/Pi-Hole_Cloudflare_TLS.sh

   # chmod +x Pi-Hole_Cloudflare_TLS.sh

   # ./Pi-Hole_Cloudflare_TLS.sh <email address> <domain name> <path to Cloudflare auth file>
```

Once the script is complete, you have a couple of more things to do.

12. Edit /etc/hosts to add your static IP address with the hostname

``` 
   123.123.123.123 pi-hole.example.com pi-hole
```

13. Create a symbolic link in /etc/conf-enabled pointing to the external.conf  

```
   # ln -sf /etc/lighttpd/conf-enabled/external.conf /etc/lighttpd/external.conf
```

14. Edit /etc/lighttpd/lighttpd.conf then add "mod_openssl", to the server.modules section.

```
   server.modules = (
        "mod_indexfile"
   server.modules = (
        "mod_indexfile",
        "mod_access",
        "mod_alias",
        "mod_redirect",
        "mod_openssl", <- ADD THIS LINE
  )
```      

Restart lighttpd and connect to Pi-hole via http. The server will redirect to the secure version of the site if everything is done correctly.

```
   # systemctl restart lighttpd
```


References  
https://github.com/Gestas/Pi-Hole-TLS-with-Cloudflare   
https://tech.borpin.co.uk/2019/03/22/letsencrypt-ssl-certificates-by-dns-challenge-with-lighttpd/  
https://redmine.lighttpd.net/projects/lighttpd/wiki/HowToSimpleSSL
