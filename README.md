# Home Lab project

Project goal is to deploy a home lab server. Server is multi purpose and will contain multiple applications and services.

# Table of contents
- [Home Lab project](#home-lab-project)
- [Table of contents](#table-of-contents)
- [List of services/applications to host](#list-of-servicesapplications-to-host)
- [Disclaimer](#disclaimer)
- [Resources](#resources)
  - [TODO](#todo)
- [Proxmox installation](#proxmox-installation)
- [Proxmox first time setup](#proxmox-first-time-setup)
  - [Laptop settings](#laptop-settings)
- [Docker in proxmox LXC](#docker-in-proxmox-lxc)
  - [CT Templates](#ct-templates)
  - [Create CT](#create-ct)
  - [Contianer console is blank?](#contianer-console-is-blank)
  - [Pre Docker installation settings](#pre-docker-installation-settings)
  - [Docker installation](#docker-installation)
  - [Portainer setup](#portainer-setup)
    - [Setting up Nginx proxy manager](#setting-up-nginx-proxy-manager)
  - [Final touches](#final-touches)
- [Server monitoring](#server-monitoring)
- [Pi-hole (in LXC)](#pi-hole-in-lxc)
  - [Adblock lists](#adblock-lists)
- [Pterodactyl installation](#pterodactyl-installation)
- [Matrix synapse Docker deployment [TEST environment]](#matrix-synapse-docker-deployment-test-environment)
  - [Generate a config file](#generate-a-config-file)
  - [Running synapse container](#running-synapse-container)
  - [Generating an (admin) user](#generating-an-admin-user)
  - [Get yourself a matrix client](#get-yourself-a-matrix-client)
  - [Setting up Nginx Proxy manager (federation)](#setting-up-nginx-proxy-manager-federation)
- [Matrix synapse Docker deployment [PostgreSQL]](#matrix-synapse-docker-deployment-postgresql)


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
- [https://www.youtube.com/watch?v=Qrglquxw-6I](https://www.youtube.com/watch?v=Qrglquxw-6I)
- [https://www.youtube.com/watch?v=Qrglquxw-6I](https://www.youtube.com/watch?v=Qrglquxw-6I)
  
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
```sh
apt-get update
```

```sh
apt-get install ca-certificates curl gnupg lsb-release
```

```sh
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

```sh
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
     $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```sh
apt-get update
```

Now you can install docker containers.

## Portainer setup

Navigate to your proxmox container shell/console.

Installing portainer for the first time:
```docker
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```
Then add Nginx proxy manager container, set it up to reverse into portainer
### Setting up Nginx proxy manager

Set up a new stack and name it: nginxproxymanager

Docker compose: [Template](Docker-compose-templates/nginxproxymanager.yml)
```yaml
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

```nginx
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

```docker
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
```yaml
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
```sh
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

```conf
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

## Adblock lists

In the group settings find Adlist. From site [https://firebog.net/](https://firebog.net/) you add these links to your Adlist. After that update the gravity.

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

# Matrix synapse Docker deployment [TEST environment]

Before we start, you need to have docker engine running on your system. After that we will follow the [official instructions](https://hub.docker.com/r/matrixdotorg/synapse/).

## Generate a config file

The first step is to generate a valid config file. To do this, you can run the image with the generate command line option.

You will need to specify values for the` SYNAPSE_SERVER_NAME` and `SYNAPSE_REPORT_STATS` environment variable, and mount a docker volume to store the configuration on. For example:
```docker
docker run -it --rm \
    --mount type=volume,src=synapse-data,dst=/data \
    -e SYNAPSE_SERVER_NAME=my.matrix.host \
    -e SYNAPSE_REPORT_STATS=yes \
    matrixdotorg/synapse:latest generate
```

After that you will have to navigate to your docker volume. Most likely you will have to use `root` user to access the file.
```
sudo su root
```
then
```
nano /var/lib/docker/volumes/synapse-data/_data/homeserver.yaml
```
contents should look like this:
```yaml
# Configuration file for Synapse.
#
# This is a YAML file: see [1] for a quick introduction. Note in particular
# that *indentation is important*: all the elements of a list or dictionary
# should have the same indentation.
#
# [1] https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html
#
# For more information on how to configure Synapse, including a complete accounting of
# each option, go to docs/usage/configuration/config_documentation.md or
# https://matrix-org.github.io/synapse/latest/usage/configuration/config_documentation.html
server_name: "my.matrix.host"
pid_file: /data/homeserver.pid
listeners:
  - port: 8008
    tls: false
    type: http
    x_forwarded: true
    resources:
      - names: [client, federation]
        compress: false
database:
  name: sqlite3
  args:
    database: /data/homeserver.db
log_config: "/data/my.matrix.host.log.config"
media_store_path: /data/media_store
registration_shared_secret: "super-secret-key"
report_stats: true
macaroon_secret_key: "super-secret-key"
form_secret_key: "super-secret-key"
signing_key_path: "/data/my.matrix.host.signing.key"
trusted_key_servers:
  - server_name: "matrix.org"
use_presence: true
client_max_body_size: 50M
```

## Running synapse container

Next we start our synapse container.

```docker
docker run -d --name synapse \
    --mount type=volume,src=synapse-data,dst=/data \
    -p 8008:8008 \
    matrixdotorg/synapse:latest
```

## Generating an (admin) user

To see the optional arguments use this:

```docker
docker exec -it synapse register_new_matrix_user http://local-ip-of-your-docker-machine:8008 -c /data/homeserver.yaml --help
```

To register admin account do this:

```docker
docker exec -it synapse register_new_matrix_user http://local-ip-of-your-docker-machine:8008 -c /data/homeserver.yaml -u Your-username -p Your-password -a
```

## Get yourself a matrix client

To access your server you must get or use 3rd party clients. There are many to choose from. More infromation can be found at [https://matrix.org/clients/](https://matrix.org/clients/).

## Setting up Nginx Proxy manager (federation)

To be able to access the federated homeserver/servers (other public servers and their channels/rooms from your homeserver) you need to do this:

Open up `Nginx proxy manager` UI. Under `Proxy Hosts` add a new proxy host to your matrix server. Enter domain name, scheme: http, enter your matrix-synapse server ip and port should be 8008. Check `Cache Assets`: true, `Block Common exploits`: true, `Websockets Support`: true. Under SSL tab get a new certificate, `Force SSL`: true, `HSTS Enabled`: true. Under advanced enter this:
```
add_header Access-Control-Allow-Origin *;
listen 8448 ssl http2;

location /.well-known/matrix/server {
    return 200 '{"m.server": "my.matrix.host:443"}'; # <- Be sure to  change this to your domain
    default_type application/json;
    add_header Access-Control-Allow-Origin *;
}
```

Now you should be able to access other servers/channels.

# Matrix synapse Docker deployment [PostgreSQL]

First we will create directory called `synapse`, then add another directory inside it called `data`.
```
mkdir synapse && cd synapse && mkdir data
```
Now create `docker-compose.yml` file and add the contents:

```
nano docker-compose.yml
```

And add this to it:

```yml
version: "3.3"

services:
    synapse:
        image: "matrixdotorg/synapse:latest"
        container_name: "synapse"
        volumes:
            - "./data:/data"
        environment:
            VIRTUAL_HOST: "sub.domain.com"
            VIRTUAL_PORT: 8008
            SYNAPSE_SERVER_NAME: "sub.domain.com"
            SYNAPSE_REPORT_STATS: "yes"
        networks: ["matrix-server"]
    postgresql:
        image: postgres:latest
        restart: always
        environment:
            POSTGRES_PASSWORD: somepassword
            POSTGRES_USER: synapse
            POSTGRES_DB: synapse
            POSTGRES_INITDB_ARGS: "--encoding='UTF8' --lc-collate='C' --lc-ctype='C'"
        volumes:
            - "./postgresdata:/var/lib/postgresql/"
        networks: ["matrix-server"]

networks:
    server:
        external: true
```

Be sure to edit `VIRTUAL_HOST`, `SYNAPSE_SERVER_NAME`, `POSTGRES_PASSWORD` with your use case values. Save the file edits. To generate random password use this:
```
openssl rand -base64 32
```

Now lets generate new synapse config file in `/synapse/data/` directory. Run command:
```docker
docker run -it --rm \
    --mount type=bind,src="$(pwd)"/data,dst=/data \
    -e SYNAPSE_SERVER_NAME=my.matrix.host \
    -e SYNAPSE_REPORT_STATS=yes \
    matrixdotorg/synapse:latest generate
```
Lets edit that homeserver.yaml file:

```
sudo nano data/homeserver.yaml
```

Now you should see something like this:

```yml
# Configuration file for Synapse.
#
# This is a YAML file: see [1] for a quick introduction. Note in particular
# that *indentation is important*: all the elements of a list or dictionary
# should have the same indentation.
#
# [1] https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html
#
# For more information on how to configure Synapse, including a complete accounting of
# each option, go to docs/usage/configuration/config_documentation.md or
# https://matrix-org.github.io/synapse/latest/usage/configuration/config_documentation.html
server_name: "my.matrix.host"
pid_file: /data/homeserver.pid
listeners:
  - port: 8008
    tls: false
    type: http
    x_forwarded: true
    resources:
      - names: [client, federation]
        compress: false
database:
  name: sqlite3
  args:
    database: /data/homeserver.db
log_config: "/data/my.matrix.host.log.config"
media_store_path: /data/media_store
registration_shared_secret: "@Wi=-m16+cQ5E#WBzLv:e.,K2dUzQsrdWf8SYSCSVe+Ro.K3jw"
report_stats: true
macaroon_secret_key: "@A8d~i1Qous5w+Q4*VqCrWwrmAsb:;VGmv0Zuv87SVVM=4FaZz"
form_secret: "kidxcd-.NnXBnKor,2GvrzV8XB1emK.e6PEpF@*,ajc0AV:.Nd"
signing_key_path: "/data/my.matrix.host.signing.key"
trusted_key_servers:
  - server_name: "matrix.org"


# vim:ft=yaml
```

Now in order to use PostgreSQL with your synapse you need to delete the database part of this config:

```yml
database:
  name: sqlite3
  args:
    database: /data/homeserver.db
```

And replace it with this (making appropriate changes):

```yml
database:
    name: psycopg2
    args:
        user: synapse
        password: "super-secret-password"
        host: postgresql
        database: synapse
        cp_min: 5
        cp_max: 10
```

The `host: postgresql` is from docker-compose.yml, we will launch the synapse service along with postgresql.

Now add more settings to the `homeserver.yaml`:
```yml
trusted_key_servers:
  - server_name: "matrix.org"
enable_registration: true
enable_registration_without_verification: true
client_max_body_size: 50M
auto_join_rooms:                         # this is optional and you can comment this out.
  - "#exampleroom:sub.domain.com"        # change the exampleroom for the name of your public room you will have on the server for people to auto join.
  - "#anotherexampleroom:sub.domain.com" # you can add another room for new users to join
```

Make sure that your reverse proxy is set up to allow federation. Follow the instructions that are written [here](#setting-up-nginx-proxy-manager-federation).


Now youre ready to deploy. 

```
docker-compose up -d
```

Now Get yourself a client, read up [here](#get-yourself-a-matrix-client). And try to join your server, and create an account.
