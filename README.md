# Home Lab project

Project goal is to deploy a home lab server. Server is multi purpose and will contain multiple applications and services.

# List of services/applications to host

List bellow will contain services/application that i want to deploy on this home lab server.

- Proxmox
- Proxmox backup server
- Steam OS Virtual machine
- Windows 10 Pro Virtual machine
- Docker (in LXC)
  - Portainer
    - Nginx proxy manager
    - Nginx (web server - blog or something)
    - Nextcloud
    - Grafana
    - Prometheus
- Pi-hole (in LXC)
- EmulationStation (with wii-u and ps3) Virtual machine
- Pterodactyl (game server management)
  - Valheim New World
  - Valheim Old World
  - Mordhau (optional)
  - Xonotic
  - Quake LIVE

# Disclaimer

I am not a professional techno jesus. Instructions bellow is for my personal use case. For all the n00bs out there use it at your own risk.

# Resources

I have condensed all the tutorial into this instruction set. Bellow you will find links to resources that i used. (Mostly videos).

- [Virtual Machines Pt. 2 (Proxmox install w/ Kali Linux)](https://www.youtube.com/watch?v=_u8qTN3cCnQ)
- [Docker Containers On Proxmox! (Two Different Ways - No VMs!)](https://www.youtube.com/watch?v=hDR_1opHGNQ)
- [Portainer Install Ubuntu tutorial - manage your docker containers](https://www.youtube.com/watch?v=ljDI5jykjE8)
- [Nginx Proxy Manager - How-To Installation and Configuration](https://www.youtube.com/watch?v=P3imFC7GSr0&t=0s)
- [You're running Pi-Hole wrong! Setting up your own Recursive DNS Server!](https://www.youtube.com/watch?v=FnFtWsZ8IP0)
## TODO

List of resources to go through and make proper instruction set for deployment.

- [My new Proxmox Monitoring Tools: InfluxDB2 + Grafana](https://www.youtube.com/watch?v=f2eyVfCTLi0)
- [https://github.com/theVakhovskeIsTaken/holoiso](https://github.com/theVakhovskeIsTaken/holoiso)
- [https://www.youtube.com/watch?v=fzBCR_C26QE](https://www.youtube.com/watch?v=fzBCR_C26QE)
  
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

## Laptop settings

If you are running this on laptop, then its a good idea to make sure your laptop wont shut down once the lid is closed. In order to do that navigate to proxmox node and open up >_ Shell

Change lid properties:

```
nano /etc/systemd/logind.conf
```

Unncomment and edit these lines:

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

It means after 60s the screen will turn off.

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

Set storage CPU and Memory to your needs. For my test rig i set up 1 CPU core and 2G memory.

Set Network parameters to your needs. You can always leave it on DHCP to get IP from your router. If you are hosting web services then its better to set up a static IP.

Create and start your container and make sure its running.

## Contianer console is blank?

Sometimes console on your proxmox container goes blank and does not show you login information. Its a bug (still present in 7.2). 
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

Docker compose: [Template](Docker-compose-templates/nginxproxymanager.yml)
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

Now navigate to your http://proxmox-container-ip:81/ and login as usr: admin@example.com pw: changeme

Create new account and set up a password.

Then add new proxy host (you have to get FQDN or in other words domain name for your host. Make sure it is an "A" record not an "CNAME" otherwise you won't be able to get Let's encrypt certificates) 

Set the domain name, scheme for portainer is http. Set portainer IP/or use hostname and port 9000.

Set up SSL for your proxy host. Turn on Force SSL, HTTP/2 Support, HSTS Enabled. Get Let's encrypt certificates.

And in the advanced tab add these:

```
# This is very important if you are using proxmox behind a proxy, otherwise you wont be able to access consoles and you will get noVNC errors and wont be able to see VM.

location / {
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade"; 
        proxy_pass https://proxmox:8006;
        proxy_buffering off;
        client_max_body_size 0;
        proxy_connect_timeout  3600s;
        proxy_read_timeout  3600s;
        proxy_send_timeout  3600s;
        send_timeout  3600s;
    }

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

# This is optional if you have pi-hole set up.
	location /pi-hole/ {
		proxy_set_header Upgrade $http_upgrade;
		proxy_pass http://pi-hole/admin/;
		proxy_http_version 1.1;
		proxy_set_header Connection "";
	}
```

## Final touches

We need to add portainer to nginxproxymanager docker network.

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

# Server monitoring

I have chosen to monitor my server and all the resources that i have using influxdb2, grafana. I will deploy them using another LXC container with portainer stack and hide the services behind a proxy.

Create new file:

```
nano /etc/prometheus/prometheus.yml
```

Use [prometheus.yml](Configs/prometheus.yml) as a template.

Creating docker compose: [Template](Configs/monitoring.yml)
```
---
version: '3'

volumes:
  grafana-data:
    driver: local
  prometheus-data:
    driver: local

services:
  grafana:
    image: grafana/grafana-oss:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    restart: unless-stopped
  environment:
      GF_PATHS_CONFIG: /etc/grafana/grafana.ini
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - /etc/prometheus:/etc/prometheus
      - prometheus-data:/prometheus
    restart: unless-stopped
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"

  node_exporter:
    image: quay.io/prometheus/node-exporter:latest
    container_name: node_exporter
    command:
      - '--path.rootfs=/host'
    pid: host
    restart: unless-stopped
    volumes:
      - '/:/host:ro,rslave'

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
    - 8080:8080
    volumes:
    - /:/rootfs:ro
    - /var/run:/var/run:rw
    - /sys:/sys:ro
    - /var/lib/docker/:/var/lib/docker:ro
 #cadvisor:
 #  image: gcr.io/cadvisor/cadvisor
 #  container_name: cadvisor
 #  ports:
 #    - "8084:8080"
 #  volumes:
 #    - /:/rootfs:ro
 #    - /var/run:/var/run:ro
 #    - /sys:/sys:ro
 #    - /var/lib/docker/:/var/lib/docker:ro
 #    - /dev/disk/:/dev/disk:ro
 #  devices:
 #    - /dev/kmsg:/dev/kmsg
 #  restart: unless-stopped
    
  valheim-old-server-status:
    container_name: valheim-old-world-status
    image: aldjinn/valheim-server-status:latest
    ports:
      - "13090:13090"
    environment:
      - VALHEIM_HOST=xx.xx.xx.xx
      - VALHEIM_PORT=3456
      - VALHEIM_QUERY_CRON=*/5 * * * *
      - TELEGRAM_CHAT_ID=-123456789
      - TELEGRAM_BOT=bot123456789:nuG0iuy7ae9eVah5eef8tahXee6eij8nieD
      - TELEGRAM_ENABLED=false
      - TELEGRAM_STARTUP_MESSAGE=false
      - METRICS_ENABLED=true
      - WEBHOOK_ENABLED=false
      - CORS_ENABLED=true
      - CORS_ALLOW_ORIGIN=*
      
  valheim-new-server-status:
    container_name: valheim-new-world-status
    image: aldjinn/valheim-server-status:latest
    ports:
      - "13080:13080"
    environment:
      - VALHEIM_HOST=xx.xx.xx.xx
      - VALHEIM_PORT=2456
      - VALHEIM_QUERY_CRON=*/5 * * * *
      - TELEGRAM_CHAT_ID=-123456789
      - TELEGRAM_BOT=bot123456789:nuG0iuy7ae9eVafd59f8tahXee6eij8nieD
      - TELEGRAM_ENABLED=false
      - TELEGRAM_STARTUP_MESSAGE=false
      - METRICS_ENABLED=true
      - WEBHOOK_ENABLED=false
      - CORS_ENABLED=true
      - CORS_ALLOW_ORIGIN=*
```

# Pi-hole (in LXC)

I tried few CT templates (debian 11, ubuntu 22.04) and failed, some packages could not be found so i used ubuntu 21.10.

Setting up CT make sure you give static ipv4 address and also add dns and name servers outside your network.

I used [One-Step Automated Install](https://github.com/pi-hole/pi-hole/#one-step-automated-install)

Firstly install curl:
```
apt update && apt upgrade && apt install curl -y
```

Then use the automated script:
```
curl -sSL https://install.pi-hole.net | bash
```

Use either google or open DNS at first, later we will chnage to recursive local DNS.

After install is complete make sure to note the password that shows up on screen.

Log into admin panel using http://static-ipv4-address/admin

In case you forgot the password, you can always change it with this command:
```
pihole -a -p
```

Next we setup unbound using [this documentation](https://docs.pi-hole.net/guides/dns/unbound/#setting-up-pi-hole-as-a-recursive-dns-server-solution).

```
apt install unbound -y
```

Create configuration file:
```
nano /etc/unbound/unbound.conf.d/pi-hole.conf
```

```
server:
    # If no logfile is specified, syslog is used
    # logfile: "/var/log/unbound/unbound.log"
    verbosity: 0

    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes

    # May be set to yes if you have IPv6 connectivity
    do-ip6: no

    # You want to leave this to no unless you have *native* IPv6. With 6to4 and
    # Terredo tunnels your web browser should favor IPv4 for the same reasons
    prefer-ip6: no

    # Use this only when you downloaded the list of primary root servers!
    # If you use the default dns-root-data package, unbound will find it automatically
    #root-hints: "/var/lib/unbound/root.hints"

    # Trust glue only if it is within the server's authority
    harden-glue: yes

    # Require DNSSEC data for trust-anchored zones, if such data is absent, the zone becomes BOGUS
    harden-dnssec-stripped: yes

    # Don't use Capitalization randomization as it known to cause DNSSEC issues sometimes
    # see https://discourse.pi-hole.net/t/unbound-stubby-or-dnscrypt-proxy/9378 for further details
    use-caps-for-id: no

    # Reduce EDNS reassembly buffer size.
    # IP fragmentation is unreliable on the Internet today, and can cause
    # transmission failures when large DNS messages are sent via UDP. Even
    # when fragmentation does work, it may not be secure; it is theoretically
    # possible to spoof parts of a fragmented DNS message, without easy
    # detection at the receiving end. Recently, there was an excellent study
    # >>> Defragmenting DNS - Determining the optimal maximum UDP response size for DNS <<<
    # by Axel Koolhaas, and Tjeerd Slokker (https://indico.dns-oarc.net/event/36/contributions/776/)
    # in collaboration with NLnet Labs explored DNS using real world data from the
    # the RIPE Atlas probes and the researchers suggested different values for
    # IPv4 and IPv6 and in different scenarios. They advise that servers should
    # be configured to limit DNS messages sent over UDP to a size that will not
    # trigger fragmentation on typical network links. DNS servers can switch
    # from UDP to TCP when a DNS response is too big to fit in this limited
    # buffer size. This value has also been suggested in DNS Flag Day 2020.
    edns-buffer-size: 1232

    # Perform prefetching of close to expired message cache entries
    # This only applies to domains that have been frequently queried
    prefetch: yes

    # One thread should be sufficient, can be increased on beefy machines. In reality for most users running on small networks or on a single machine, it should be unnecessary to seek performance enhancement by increasing num-threads above 1.
    num-threads: 1

    # Ensure kernel buffer is large enough to not lose messages in traffic spikes
    so-rcvbuf: 1m

    # Ensure privacy of local IP ranges
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: fd00::/8
    private-address: fe80::/10

```

Start your local recursive server and test that it's operational:

```
service unbound restart
```
```
dig pi-hole.net @127.0.0.1 -p 5335
```

Now in the pi-hole admin panel we need to change upstream DNS servers. Settings > DNS > Uncheck ipv4 google or openDNS from the upstream servers, add Custom 1 ipv4 DNS server: 127.0.0.1#5335 > Save

And now the last thing is to set up your router to point to pi-hole as a DNS sever. Look up your router configuration manual and see how to change DNS server. I usually leave one DNS ip that my ISP provided in case my pi-hole goes down, so that i dont lose connection to the interwebs.

It is also a good practice to reboot your VM's and other services to get the DNS from DHCP of your router.

# Pterodactyl installation

The easiest way is to follow [the official documentation](https://pterodactyl.io/). Just create a dedicated VM with a static IP. I used ubuntu 20.04.4-live-server, gave it 8GiB RAM and 64GiB Hdd.

RAM usage based on game server so far (for one server instance):
- 2GiB
  - Valheim
  - Quake Live
- 2.5GiB
  - Xonotic

Setting up web server i used Nginx without SSL because im using Nginx reverse proxy manager to handle SSL certs. But note that you wont be able to access game server consoles from outside networks, because by default pterodactyl needs SSL certs to manage some kind of services.

Also make sure to forward any ports that your game server needs.