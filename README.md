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
  - [VM hard drive space expanding](#vm-hard-drive-space-expanding)
  - [Configure GPU passthrough](#configure-gpu-passthrough)
    - [Step 1: Configuring the Grub](#step-1-configuring-the-grub)
    - [Step 2: VFIO Modules](#step-2-vfio-modules)
    - [Step 3: IOMMU interrupt remapping](#step-3-iommu-interrupt-remapping)
    - [Step 4: Blacklisting Drivers](#step-4-blacklisting-drivers)
    - [Step 5: Adding GPU to VFIO](#step-5-adding-gpu-to-vfio)
  - [VM hard drive space expanding - alternative](#vm-hard-drive-space-expanding---alternative)
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
- [Matrix synapse Docker deployment \[TEST environment\]](#matrix-synapse-docker-deployment-test-environment)
  - [Generate a config file](#generate-a-config-file)
  - [Running synapse container](#running-synapse-container)
  - [Generating an (admin) user](#generating-an-admin-user)
  - [Get yourself a matrix client](#get-yourself-a-matrix-client)
  - [Setting up Nginx Proxy manager (federation)](#setting-up-nginx-proxy-manager-federation)
- [Matrix synapse Docker deployment \[PostgreSQL\]](#matrix-synapse-docker-deployment-postgresql)
  - [Create docker-compose file](#create-docker-compose-file)
  - [Generate a config file](#generate-a-config-file-1)
  - [Generating an (admin) user](#generating-an-admin-user-1)
  - [Get yourself a matrix client](#get-yourself-a-matrix-client-1)
  - [PostgreSQL basic usage](#postgresql-basic-usage)
- [Matrix dendrite Docker deployment \[PostgreSQL\]](#matrix-dendrite-docker-deployment-postgresql)
- [TrueNAS Core VM](#truenas-core-vm)
  - [Adding HDD's to VM](#adding-hdds-to-vm)
- [Jellyfin media server deployment (docker)](#jellyfin-media-server-deployment-docker)

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
- Media Server

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
- [https://support.zadarastorage.com/hc/en-us/articles/213024986-How-to-Mount-a-SMB-Share-in-Ubuntu](https://support.zadarastorage.com/hc/en-us/articles/213024986-How-to-Mount-a-SMB-Share-in-Ubuntu)
- [I found an Excellent Raspberry Pi Replacement for Home Assistant / IOTstack (incl. Proxmox)](https://www.youtube.com/watch?v=rXc_zGRYhLo)
  
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

## VM hard drive space expanding

To expand your VM hard drive space, first log into proxmox web UI, then resize the VM disk.

Then log into your VM and run few commands. Run `df -h` command to list filesystem:

```
df -h
```

```
sus@sussy:~$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              198M  8.6M  190M   5% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   28G  7.8G   19G  30% /
tmpfs                              989M     0  989M   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          1.7G  261M  1.4G  17% /boot
tmpfs                              198M  4.0K  198M   1% /run/user/1000
```

Note the name of your `mapper`: `/dev/mapper/ubuntu--vg-ubuntu--lv`

Run as root vgdisplay

```
sudo vgdisplay
```

```
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <30.25 GiB
  PE Size               4.00 MiB
  Total PE              7743
  Alloc PE / Size       7231 / <28.25 GiB
  Free  PE / Size       512 / 2.00 GiB
  VG UUID               wyALx3-gIEp-80O8-EFvS-Fzd0-brTQ-UzrTR4
```

Note `Free  PE / Size` and the avaliable size. In my case i already have expanded my volume, yours may be different.

Now expand the volume (as root)

```
lvextend -L +2G /dev/mapper/ubuntu--vg-ubuntu--lv
```


Now we resize/update filesystem:
```
resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```
Now youre good to go.

## Configure GPU passthrough

As per guide [https://www.reddit.com/r/homelab/comments/b5xpua/the_ultimate_beginners_guide_to_gpu_passthrough/](https://www.reddit.com/r/homelab/comments/b5xpua/the_ultimate_beginners_guide_to_gpu_passthrough/)
### Step 1: Configuring the Grub

Assuming you are using an Intel CPU, either SSH directly into your Proxmox server, or utilizing the noVNC Shell terminal under "Node", open up the /etc/default/grub file. I prefer to use nano, but you can use whatever text editor you prefer.
```
nano /etc/default/grub
```
Look for this line:

`GRUB_CMDLINE_LINUX_DEFAULT="quiet"`

Then change it to look like this:

For Intel CPUs:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
```
For AMD CPUs:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on"
```
IMPORTANT ADDITIONAL COMMANDS

You might need to add additional commands to this line, if the passthrough ends up failing. For example, if you're using a similar CPU as I am (Xeon E3-12xx series), which has horrible IOMMU grouping capabilities, and/or you are trying to passthrough a single GPU.

These additional commands essentially tell Proxmox not to utilize the GPUs present for itself, as well as helping to split each PCI device into its own IOMMU group. This is important because, if you try to use a GPU in say, IOMMU group 1, and group 1 also has your CPU grouped together for example, then your GPU passthrough will fail.

Here are my grub command line settings:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt pcie_acs_override=downstream,multifunction nofb nomodeset video=vesafb:off,efifb:off"
```
For more information on what these commands do and how they help:

A. Disabling the Framebuffer: video=vesafb:off,efifb:off

B. ACS Override for IOMMU groups: pcie_acs_override=downstream,multifunction

When you finished editing /etc/default/grub run this command:
```
update-grub
```
### Step 2: VFIO Modules

You'll need to add a few VFIO modules to your Proxmox system. Again, using nano (or whatever), edit the file /etc/modules
```
nano /etc/modules
```
Add the following (copy/paste) to the /etc/modules file:
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```
Then save and exit.

### Step 3: IOMMU interrupt remapping

I'm not going to get too much into this; all you really need to do is run the following commands in your Shell:
```
echo "options vfio_iommu_type1 allow_unsafe_interrupts=1" > /etc/modprobe.d/iommu_unsafe_interrupts.conf
```
```
echo "options kvm ignore_msrs=1" > /etc/modprobe.d/kvm.conf
```
### Step 4: Blacklisting Drivers

We don't want the Proxmox host system utilizing our GPU(s), so we need to blacklist the drivers. Run these commands in your Shell:
```
echo "blacklist radeon" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist.conf
echo "blacklist nvidia" >> /etc/modprobe.d/blacklist.conf
```
### Step 5: Adding GPU to VFIO

Run this command:
```
lspci -v
```
Your shell window should output a bunch of stuff. Look for the line(s) that show your video card. It'll look something like this:

01:00.0 VGA compatible controller: NVIDIA Corporation GP104 [GeForce GTX 1070] (rev a1) (prog-if 00 [VGA controller])

01:00.1 Audio device: NVIDIA Corporation GP104 High Definition Audio Controller (rev a1)

Make note of the first set of numbers (e.g. 01:00.0 and 01:00.1). We'll need them for the next step.

Run the command below. Replace 01:00 with whatever number was next to your GPU when you ran the previous command:
```
lspci -n -s 01:00
```
Doing this should output your GPU card's Vendor IDs, usually one ID for the GPU and one ID for the Audio bus. It'll look a little something like this:

01:00.0 0000: 10de:1b81 (rev a1)

01:00.1 0000: 10de:10f0 (rev a1)

What we want to keep, are these vendor id codes: 10de:1b81 and 10de:10f0.

Now we add the GPU's vendor id's to the VFIO ``(remember to replace the id's with your own!)``:
```
echo "options vfio-pci ids=10de:1b81,10de:10f0 disable_vga=1"> /etc/modprobe.d/vfio.conf
```
Finally, we run this command:
```
update-initramfs -u
```
And restart:
```
reset
```
Now your Proxmox host should be ready to passthrough GPUs!

## VM hard drive space expanding - alternative

Firstly shutdown your VM, then to expand your VM hard drive space, first log into proxmox web UI, then resize the VM disk.

Check your partitions and get the name of your hard drive partition:
```
sudo lsblk
```

```bash
sus@sussy:~$ sudo lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0     4K  1 loop /snap/bare/5
loop1    7:1    0    62M  1 loop /snap/core20/1611
loop2    7:2    0  63.2M  1 loop /snap/core20/1623
loop3    7:3    0 163.3M  1 loop /snap/firefox/1670
loop4    7:4    0 400.8M  1 loop /snap/gnome-3-38-2004/112
loop5    7:5    0  67.8M  1 loop /snap/lxd/22753
loop6    7:6    0    47M  1 loop /snap/snapd/16292
loop7    7:7    0   284K  1 loop /snap/snapd-desktop-integration/14
loop8    7:8    0  91.7M  1 loop /snap/gtk-common-themes/1535
loop9    7:9    0  67.2M  1 loop /snap/lxd/21835
loop10   7:10   0 346.3M  1 loop /snap/gnome-3-38-2004/115
loop11   7:11   0 176.9M  1 loop /snap/firefox/1810
loop12   7:12   0    48M  1 loop /snap/snapd/16778
sda      8:0    0   104G  0 disk
├─sda1   8:1    0     1M  0 part
└─sda2   8:2    0    64G  0 part /
sr0     11:0    1   1.2G  0 rom  /media/gamemaster/Ubuntu-Server 20.04.4 LTS amd641
```

In my case its sda2.

Now grow that partition with:

```
sudo growpart /dev/sda 2
```

```
sus@sussy:~$ sudo growpart /dev/sda 2
CHANGED: partition=2 start=4096 old: size=134211584 end=134215680 new: size=218099679 end=218103775

sus@sussy:~$ sudo lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0     4K  1 loop /snap/bare/5
loop1    7:1    0    62M  1 loop /snap/core20/1611
loop2    7:2    0  63.2M  1 loop /snap/core20/1623
loop3    7:3    0 163.3M  1 loop /snap/firefox/1670
loop4    7:4    0 400.8M  1 loop /snap/gnome-3-38-2004/112
loop5    7:5    0  67.8M  1 loop /snap/lxd/22753
loop6    7:6    0    47M  1 loop /snap/snapd/16292
loop7    7:7    0   284K  1 loop /snap/snapd-desktop-integration/14
loop8    7:8    0  91.7M  1 loop /snap/gtk-common-themes/1535
loop9    7:9    0  67.2M  1 loop /snap/lxd/21835
loop10   7:10   0 346.3M  1 loop /snap/gnome-3-38-2004/115
loop11   7:11   0 176.9M  1 loop /snap/firefox/1810
loop12   7:12   0    48M  1 loop /snap/snapd/16778
sda      8:0    0   104G  0 disk
├─sda1   8:1    0     1M  0 part
└─sda2   8:2    0   104G  0 part /
sr0     11:0    1   1.2G  0 rom  /media/gamemaster/Ubuntu-Server 20.04.4 LTS amd641

```

Now to resize the partition for real:

```
sudo resize2fs /dev/sda2
```

```
sus@sussy:~$ sudo resize2fs /dev/sda2
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/sda2 is mounted on /; on-line resizing required
old_desc_blocks = 8, new_desc_blocks = 13
The filesystem on /dev/sda2 is now 27262459 (4k) blocks long.
```

You should be good now.

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

Make sure to prune docker images once a month or more often.

```
docker image prune -a
```

This will save up on your storage.

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

## Create docker-compose file

Now create `docker-compose.yml` file and add the contents (be sure to edit your domain names and passwords):

```
nano docker-compose.yml
```

And add this to it:

```yml
version: "3.3"

services:
    synapse:
        image: "matrixdotorg/synapse:latest"
        ports:
          - "8008:8008"
        restart: always
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
    pgbackups:
        container_name: "Backup"
        image: prodrigestivill/postgres-backup-local
        restart: always
        volumes:
          - "./backup:/backups"
        links:
          - postgresql:postgresql
        depends_on:
          - postgresql
        environment:
          - POSTGRES_HOST="sub.domain.com"
          - POSTGRES_DB="synapse"
          - POSTGRES_USER="synapse"
          - POSTGRES_PASSWORD="somepassword"
          - POSTGRES_EXTRA_OPTS="-Z9 --schema=public --blobs"
          - SCHEDULE="@every 0h30m0s"
          - BACKUP_KEEP_DAYS="7"
          - BACKUP_KEEP_WEEKS="4"
          - BACKUP_KEEP_MONTHS="6"
          - HEALTHCHECK_PORT="81"
networks:
    matrix-server:
```
As you can se we have created backup contianer (`pgbackups`) to make backups of the postgres DB. You can read more about it on [https://github.com/prodrigestivill/docker-postgres-backup-local](https://github.com/prodrigestivill/docker-postgres-backup-local)

Be sure to edit `VIRTUAL_HOST`, `SYNAPSE_SERVER_NAME`, `POSTGRES_PASSWORD` with your use case values. Save the file edits. To generate random password use this:
```
openssl rand -base64 32
```

## Generate a config file
Now lets generate new synapse config file in `/synapse/data/` directory. Run command (be sure to change the `SYNAPSE_SERVER_NAME` to your domain):
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

## Generating an (admin) user

Now before you get your panties wet, you must register yourself as an admin account. Use this (edit before use):
```docker
docker exec -it synapse register_new_matrix_user http://local-ip-of-your-docker-machine:8008 -c /data/homeserver.yaml -u Your-username -p Your-password -a
```

## Get yourself a matrix client

Now Get yourself a client, read up [here](#get-yourself-a-matrix-client). And try to join your server, and create an account.

## PostgreSQL basic usage

If you wan to edit users in your PostgreSQL DB then you will have to enter the the bash of the container.

```
docker exec -it postgres bash
```

Then you log into the postgres:
```
PGPASSWORD=super-secret-password psql -h localhost -p 5432 -U synapse -d synapse
```
Then you can check if there is data in your db:
```
\dt
```

Use `SELECT` * `FROM` `rooms`; to querry the db table.

# Matrix dendrite Docker deployment [PostgreSQL]

Dendrite is a second-generation Matrix homeserver written in Go. It intends to provide an efficient, reliable and scalable alternative to Synapse. To install we will follow the official guide.
Check it out here: [https://github.com/matrix-org/dendrite](https://github.com/matrix-org/dendrite). We will use docker but we will need to clone this repo.

```
git clone https://github.com/matrix-org/dendrite ./dendrite
```
Not sure if this is needed, but it worked for me.

```
cd dendrite
```
Make sure go is installed, if not then:

```
sudo apt install golang-go
```

Then build it:

```
./build.sh
```

Now create another folder outside `./dendrite` and cal it `./dendrite-compose`. This is where out dendrite will live.

```
mkdir dendrite-compose
```

Now get the example docker-compose.yml file from this repo: [https://github.com/matrix-org/dendrite/blob/main/build/docker/docker-compose.monolith.yml](https://github.com/matrix-org/dendrite/blob/main/build/docker/docker-compose.monolith.yml). And yes we using monolith because we are running it on one machine. And save it to your `./dendrite-compose/docker-compose.yml`.

Or:

```
cp ./dendrite/build/docker/docker-compose.monolith.yml ./dendrite-compose/docker-compose.yml 
```
Now lets edit this file:

```
sudo nano ./dendrite-compose/docker-compose.yml
```

The most important parts of compose file to edit are:

```yaml
version: "3.4"
services:
  postgres:
    environment:
      POSTGRES_PASSWORD: itsasecret

  monolith:
    restart: always
```

Set the password to the and monolith ro restart always.
Next create folder` ./dendrite-compose/config` and copy file from `./dendrite/dendrite-sample.monolith.yaml` to `./dendrite-compose/config/dendrite.yaml`.

```
mkdir ./dendrite-compose/config
```

```
cp ./dendrite/dendrite-sample.monolith.yaml ./dendrite-compose/config/dendrite.yaml
```

Now edit most important parts of this config file:

```
sudo nano ./dendrite-compose/config/dendrite.yaml
```

```yaml
global:
  # The domain name of this homeserver.
  server_name: your.domain.com

  # Global database connection pool, for PostgreSQL monolith deployments only. If
  # this section is populated then you can omit the "database" blocks in all other
  # sections. For polylith deployments, or monolith deployments using SQLite databases,
  # you must configure the "database" block for each component instead.
  database:
    connection_string: postgresql://dendrite:itsasecret@postgres/dendrite?sslmode=disable

# Configuration for the Client API.
client_api:

  # If set, allows registration by anyone who knows the shared secret, regardless
  # of whether registration is otherwise disabled.
  registration_shared_secret: "someothersecret"

# Configuration for enabling experimental MSCs on this homeserver.
mscs:
  mscs:
   - msc2836  # (Threading, see https://github.com/matrix-org/matrix-doc/pull/2836)
   - msc2946  # (Spaces Summary, see https://github.com/matrix-org/matrix-doc/pull/2946)

```

Copy this yaml file back to ./dendrite folder.

```
cp ./dendrite-compose/config/dendrite.yaml ./dendrite/dendrite.yaml 
```

Now go back to your ./dendrite folder and run this command to generate tls certs:

```
go run github.com/matrix-org/dendrite/cmd/generate-keys \
  --private-key=matrix_key.pem \
  --tls-cert=server.crt \
  --tls-key=server.key
```

Then copy over generated files: matrix_key.pem, server.crt, server.key from `./dendrite` to `./dendrite-compose/config` folder.

```
cp ./dendrite/matrix_key.pem ./dendrite-compose/config/matrix_key.pem
```
```
cp ./dendrite/server.crt ./dendrite-compose/config/server.crt
```
```
cp ./dendrite/server.key ./dendrite-compose/config/server.key
```

Now go to ./dendrite-compose and run:

```
docker compose up -d
```

Once it is running we will create admin account:

```
docker exec -it CONTAINER_ID /usr/bin/create-account -config /etc/dendrite/dendrite.yaml -username YOUR_USERNAME -admin -url http://localhost:8008
```

Bob's your uncle.


# TrueNAS Core VM

Lets create a VM with minimal settings - 16GB storage for the OS and 8GB RAM, 2 CPU cores. Dont start the VM just yet.

## Adding HDD's to VM

Now lets add our existing HDD that are connected to the motherboard to this VM.

Log into your proxmox node as root and run a command:

```
lsblk -o +model,serial
```

Now locate your HDD's that you want to add to the VM and note the serial numbers.

```
root@sussy:~# lsblk -o +model,serial
NAME         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT MODEL                          SERIAL
loop0          7:0    0     4G  0 loop                                           
loop1          7:1    0    28G  0 loop                                           
sda            8:0    0 465.8G  0 disk            ST9500423AS                    5WR0QLDQ
sdb            8:16   0 465.8G  0 disk            ST9500423AS                    5WR0PT3Y
nvme0n1      259:0    0 465.8G  0 disk            Samsung SSD 970 EVO Plus 500GB S4EVNX0T406477V
├─nvme0n1p1  259:1    0  1007K  0 part                                           
├─nvme0n1p2  259:2    0   512M  0 part /boot/efi                                 
└─nvme0n1p3  259:3    0 465.3G  0 part                                           
  ├─pve-swap 253:0    0     8G  0 lvm  [SWAP]                                    
  └─pve-root 253:1    0 457.3G  0 lvm  /  
```

Next lets get the device name from /dev/ folder


```
ls /dev/disk/by-id
```

You should see something like this:

```
root@apefront:~# ls /dev/disk/by-id
ata-ST9500423AS_5WR0PT3Y                                                      nvme-eui.0025385421b05990-part2
ata-ST9500423AS_5WR0QLDQ                                                      nvme-eui.0025385421b05990-part3
dm-name-pve-root                                                              nvme-Samsung_SSD_970_EVO_Plus_500GB_S4EVNX0T406477V
dm-name-pve-swap                                                              nvme-Samsung_SSD_970_EVO_Plus_500GB_S4EVNX0T406477V-part1
dm-uuid-LVM-b2KUYxTy8RVM6K9LvWRqef2qAucbKnCiHpbnjGLfpLak1YOmoOoZ1lubbU4ZqT8G  nvme-Samsung_SSD_970_EVO_Plus_500GB_S4EVNX0T406477V-part2
dm-uuid-LVM-b2KUYxTy8RVM6K9LvWRqef2qAucbKnCiZiWFVg15QYk9uoMCqYkqOXUZFgCACdDI  nvme-Samsung_SSD_970_EVO_Plus_500GB_S4EVNX0T406477V-part3
lvm-pv-uuid-qOuhBY-CW3X-eXzl-yjIj-jzdA-zyKw-bJahq3                            wwn-0x5000c5003d6e0c92
nvme-eui.0025385421b05990                                                     wwn-0x5000c5003d6f0f02
nvme-eui.0025385421b05990-part1
```

Now lets add them to our VM. Note your VM id from the proxmox node list. And note your VM scsi drive count. Bydefault you should only have one drive added tou your VM - scsi0

```
qm set ID-OF-VM -scsi1 /dev/disk/by-id/ata-ST9500423AS_5WR0PT3Y
```

Rinse and repeat for all the drives and just update the -scsiX count.

Now you can start the VM and install the TrueNAS. To log into web UI use root and the password you set during installation.

# Jellyfin media server deployment (docker)

Here we will deploy jellyfin using docker compose. Before that if we have a media that is shared on network such as NAS. Easiest way is to make ir SMB share and then you will need to mount the share on your OS where jellyfin is going to live.

Firstly lets create few folders

```
mkdir jellyfin
```

```
cd jellyfin
```

```
mkdir config tvseries movies share
```

Lets mount the networkshare you have going. You will need to find out the ip/media folder of your network share. Also username and password to access the folder. Lets install the things we need to mount the SMB share.

```
sudo apt-get install cifs-utils
```

Now lets mount our share folder.

```
sudo mount -t cifs -o user=<user on VPSA> //<vpsa_ip_address>/<export_share> /mnt/<local_share>
```

Example:
```
sudo mount -t cifs -o user=sussy //192.168.12.22/media ./share
```

In case you reboot just add to fstab file to mount this on boot:

Mounting at boot

add to /etc/fstab file:
```
//<vpsa_ip_address>/<export_share>  /mnt/<local_share> cifs user=<user on VPSA>,pass=<passwd on VPSA> 0 0
```
Now lets find out what user is your current user, minde the group id and user id. We will need these values later.

```
id YOUR-USER-NAME
```

OUTPUT:
```bash
uid=1000(YOUR-USER-NAME) gid=1000(YOUR-USER-NAME) groups=1000(YOUR-USER-NAME),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd)
```

Now lets create our docker-compose.yaml file:

```
sudo nano docker-compose.yaml
```

```yaml
version: "2.1"
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/Helsinki
      #- JELLYFIN_PublishedServerUrl=192.168.0.5 #optional
    volumes:
      - ./config:/config
      - ./tvseries:/data/tvshows
      - ./movies:/data/movies
      - ./truenas_share:/data/share
    ports:
      - 8096:8096
      #- 8920:8920 #optional
      #- 7359:7359/udp #optional
      #- 1900:1900/udp #optional
    restart: always
```

Finally lets launch it:

```
sudo docker compose up -d
```

Now you just have to go to your http://ip:8096 and create account.
