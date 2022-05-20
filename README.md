# Home Lab project

Project goals is to deploy a home lab server. Server is multi purpose and will contain multiple applications and services.

# List of services/applications to host

List bellow will contain services/application that i want to deploy on this home lab server.

- Proxmox
- Proxmox backup server
- Steam OS Virtual machine
- Docker
  - Portainer
    - Nginx proxy manager
    - Nginx (web server - blog or something)
    - Nextcloud
    - Grafana
    - Prometheus
- EmulationStation (with wii-u and ps3)
- Pterodactyl (game server management)
  - Valheim New World
  - Valheim Old World
  - Mordhau (optional)
  - Xonotic
  - Quake LIVE

# Proxmox installation

Proxmox uses virtualization so you better check bios on your machine to enable CPU virtualization.

Get [Proxmox](https://www.proxmox.com/en/downloads). Download Proxmox VE not Proxmox Backup Server.

Get [Rufus](https://rufus.ie/en/) to burn the installation ISO on to a USB flash drive. Be sure to use DD to burn Proxmox if you are burning from Windows.

Make sure your proxmox machine has ethernet cable plugged in, otherwise it won't resolve IP address.

Boot in to Proxmox installation and just press next. Default values are fine.

On final screen remember the IP address of proxmox VE.

Once your machine is booted into proxmox then you can connect to it on your local network from a web browser. Just go to https://the-ip-address-of-your-proxmox:8006.

Default username is root. Password was set during installation.

# Proxmox first time setup

Lets expand our storage.

Navigate to Datacenter > Storage 

Now remove local-lvm.

Edit local storage, add Disk image, Containers to Content.

Navigate to proxmox node and open up >_ Shell and lets enter few commands.

```
lvremove /dev/pve/data
```

```
lvresize -l +100%FREE /dev/pve/root
```

```
resize2fs /dev/mapper/pve-root
```

If you are running this on laptop, then its a good idea to make sure your laptop wont shut down once the lid is closed. In order to do that navigate to proxmox node and open up >_ Shell

Change lid properties:

```
nano /etc/systemd/logind.conf
```

Edit/uncomment these lines:

```
HandleLidSwitch=ignore 
HandleLidSwitchDocked=ignore
```
Restart service:

```
systemctl restart systemd-logind.service
```

Change screen sleep properties:

```
nano /etc/default/grub
```

Edit line:

```
GRUB_CMDLINE_LINUX="consoleblank=60"
```

It means after 60s the screen will go to sleep.

Now update grub.

```
update-grub
```

Now everything should be good.
# Docker in proxmox LXC

## CT Templates
First you will need to set up a container. In order to do that you will need to get a container template.

Navigate to storage and check CT templates. Click Templates button and get yourself a template.

For this example i went with debian 11. Everything worked fine on (20.05.2022).

When its says TASK OK then it means that everything has been downloaded and you can close the window.

In case you cant download the template, check proxmox DNS tab and make sure the DNS IPs are set to valid ones. You can always add more DNS IPs. If you need, you can add 8.8.8.8, and you will be fine.

## Create CT

Now we will create container based on our template.

Navigate to proxmox node and on top right click Create CT. Give it a hostname and set up a root password

I used Unprivilaged container your needs may be different.

Set storage CPU and Memory to your needs.

Set Network parameters to your needs. You can always leave it on DHCP to get IP from your router. If you are hosting web services then its better to set up a static IP.

Create and start your container and make sure its running.

## Contianer console is blank?

Sometimes console on your container boes blank and does not show you login information. Its a bug (still present in 7.2). 
To fix it select your container > Options > Console mode > Edit

Now instead of the default Console mode set it to shell. Reboot container.

## Pre Docker installation settings

Navigate to your container >_ Shell and edit this file:

```
nano /etc/pve/local/lxc/THE-ID-OF-YOUR-CONTAINER.conf
```

For example:

```
nano /etc/pve/local/lxc/100.conf
```

Now make sure you have a line like this:
```
features:  keyctl=1,nesting=1
```

Save changes and reboot container.

## Docker installation

Run these commands:
```
apt-get update
```

```
apt-get install ca-certificates curl gnupg lsb-release
```

```
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
     $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```
apt-get update
```

Now you can install docker containers.

## Portainer setup

Navigate to your proxmox container shell/console.

Installing portainer for the first time:
```
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```
Then add Nginx proxy manager container, set it up to reverse into portainer
### Setting up Nginx proxy manager

Set up a new stack and name it: nginxproxymanager

Docker compose:
```
---
version: '3'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    ports:
      - '80:80' # <---- make sure these ports are forwarded if youre behind a router.
      - '81:81' # <---- make sure these ports are forwarded if youre behind a router.
      - '443:443' # <---- make sure these ports are forwarded if youre behind a router.
    restart: always
    environment:
      DB_MYSQL_HOST: "db"
      DB_MYSQL_PORT: 3306
      DB_MYSQL_USER: "npm"
      DB_MYSQL_PASSWORD: "npm"
      DB_MYSQL_NAME: "npm"
    volumes:
      -  ./nginxproxy-volume/data:/data
      -  ./nginxproxy-volume/letsencrypt:/etc/letsencrypt
  db:
    image: 'jc21/mariadb-aria:latest'
    environment:
      MYSQL_ROOT_PASSWORD: 'npm'
      MYSQL_DATABASE: 'npm'
      MYSQL_USER: 'npm'
      MYSQL_PASSWORD: 'npm'
    volumes:
      -  ./nginxproxy-volume/data/mysql:/var/lib/mysql
    restart: always
```

Deploy stack.

Now navigate to your http://container-ip:81/ and login as usr: admin@example.com pw: changeme

Create new account and set up a password.

Then add new proxy host (you have to get FQDN or in other words domain name for your host. Make sure it is an "A" record not an "CNAME" because you won't be able to get Let's encrypt certificates) 

Set the domain name, scheme for portainer is http. Set portainer IP and port 9000.

Set up SSL for your proxy host. Turn on Force SSL, HTTP/2 Support, HSTS Enabled. Get Let's encrypt certificates.

And in the advanced tab add these:

```
location /portainer/ {
		rewrite ^/portainer(/.*)$ /$1 break;
		proxy_pass http://portainer:9000/;
		proxy_http_version 1.1;
		proxy_set_header Connection "";
	}

	location /portainer/api {
		proxy_set_header Upgrade $http_upgrade;
		proxy_pass http://portainer:9000/api;
		proxy_set_header Connection 'upgrade';
		proxy_http_version 1.1;
	}

# this is optional if you have pi-hole set up.
	location /pi-hole/ {
		proxy_set_header Upgrade $http_upgrade;
		proxy_pass http://pi-hole/admin/;
		proxy_http_version 1.1;
		proxy_set_header Connection "";
	}
```

## Docker final touches

Navigate to your proxmox container shell/console.

Stop and remove existing portainer and set it up once more, so that its accessible only from proxy:

```
docker stop portainer
```

```
docker rm portainer
```

Now check the nginxproxymanager network name with this command:

```
docker network ls
```

Now deploy portainer once and for all:

```
docker run -d -p 8000:8000 --network nginxproxymanager_default --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```

Now you are ready to get to work.