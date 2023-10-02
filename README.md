# Self-Hosted Docker Services
This Includes General Configuration Steps and Includes Docker-Compose Files, All of This Was Configured on a Raspberry Pi 4.

---

#### List of Services (Links to Their Respective Github Repositories)
* [Portainer](https://github.com/Portainer/Portainer)
* [Vaultwarden](https://github.com/Dani-Garcia/Vaultwarden)
* [Pihole](https://github.com/Pi-Hole/Pi-Hole)
  * [Cloudflared](https://github.com/Cloudflare/Cloudflared)
* [Nginx Proxy Manager](https://github.com/Nginxproxymanager/Nginx-Proxy-Manager)
* [Watchtower](https://github.com/Containrrr/Watchtower)

---

## Prerequisites
* Enough Storage for Your Particular Needs, I Used a 128GB Micro Sd-Card and for All the Services Above I Have Only Claimed 8GB of Space So Far.
* An Accessible Rasperry Pi With an Installed and Updated OS. I Had [Raspberry Pi OS](https://www.raspberrypi.com/Software/Operating-Systems/) (64-Bit) Installed Which Is Based on Debian Linux. Although What You Choose Is up to You, Just Make Sure You Know the Commands for That Particular Distro or OS.
* Static IP on the Raspberry Pi Is Highly Recommended.

## Installing Docker
To Get Started We Need to Install ```docker``` & ```docker-compose```.

Note: Installation Varies Depending on OS or Distro, Read and Follow the Instructions on How to Do It on Your OS/Distro, [Dockers Official Site](https://docs.docker.com/Desktop/Install/Debian/)
#### Docker on Debian 
``` Bash
curl -sSl https://get.docker.com | sh
```
If You’re Smarter Than Me, Avoid Running Linux as Admin/Root. Add Your User to the Docker Group to Skip Typing Sudo for Docker. You Might Have to Log Out and Log Back in for This to Work.
``` Bash
sudo usermod -aG docker ${user}
```
Verify That Docker Is Installed by Running Your Very First Container.
``` Bash
sudo docker run hello-world
```
Install Docker Compose 
``` Bash
sudo apt-get install docker-compose-plugin
```
Verify That It Worked
``` Bash
docker compose version
```

## Portainer
I Like to Keep Directories of Each Service That I Am Using and Putting the Correspondent docker-compose.yml File in Those Directories. 
##### Portainer Compose File
``` Bash
version: “3”
services:
  portainer:
    image: portainer/portainer-ce:latest
    ports:
      - 9443:9443
      volumes:
        - data:/data
        - /var/run/docker.sock:/var/run/docker.sock
    restart: unless-stopped
volumes:
  data:
```
Run Portainer Using Compose File
``` Bash
docker compose up -d
```
It's That Simple, You Should Know be able to head to https://RaspberryPiIP:9443 in your browser and reach portainer. 

## PiHole + Cloudflared
``` Bash
version: "3.4"

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8061:80/tcp"
      - "4443:443/tcp"
    environment:
      TZ: 'Europe/Stockholm'
      WEBPASSWORD: 'changeme'
    volumes:
       - './data/etc:/etc/pihole/'
       - './data/dnsmasq.d/:/etc/dnsmasq.d/'
    restart: unless-stopped
    networks:
      pihole:
        ipv4_address: 172.20.0.1

  cloudflared-cf:
    container_name: cloudflared-cf
    image: cloudflare/cloudflared:latest
    command: proxy-dns --address 0.0.0.0 --port 5353 --upstream https://1.1.1.1/dns-query --upstream https://1.0.0.1/dns-query
    restart: unless-stopped
    networks:
      pihole:
        ipv4_address: 172.20.1.1

  cloudflared-goog:
    container_name: cloudflared-goog
    image: cloudflare/cloudflared:latest
    command: proxy-dns --address 0.0.0.0 --port 5353 --upstream https://8.8.8.8/dns-query --upstream https://8.8.4.4/dns-query
    restart: unless-stopped
    networks:
      pihole:
        ipv4_address: 172.20.8.8

networks:
  pihole:
    ipam:
      config:
        - subnet: 172.20.0.0/16
```
## NGINX Proxy Manager
``` Bash
version: '3'
services:
  nginx_proxy_manager:
    image: 'jc21/nginx-proxy-manager:latest'
    container_name: nginx_proxy_manager
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./config.json:/app/config/production.json
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    restart: always
    networks:
      proxy:
        ipv4_address: 172.19.0.10

networks:
  proxy:
    ipam:
      config:
        - subnet: 172.19.0.0/16
```
## Bitwarden/Vaultwarden
``` Bash
version: '3'

services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      - WEBSOCKET_ENABLED=true               
    volumes:
      - ./data:/data
    ports:
      - 8080:80                                 
```
## Wireguard VPN
``` Bash
version: "2.1"
services:
  wireguard:
    image: linuxserver/wireguard
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Chicago                                     # Change this
      - SERVERURL=auto #optional                               # Set to automatically server's external IP   
      - SERVERPORT=51820 #optional
      - PEERS=1 #optional                                      # Change this to the number of clients needed
      - PEERDNS=auto #optional
      - INTERNAL_SUBNET=10.13.13.0 #optional
      - ALLOWEDIPS=0.0.0.0/0 #optional
    volumes:
      - /home/pi/wireguard/config:/config                      # Change this
      - /lib/modules:/lib/modules
    ports:
      - 51820:51820/udp                                        # Forward port 51820/udp on your router to the server IP
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv4.ip_forward=1
    restart: unless-stopped
```
## Watchtower
``` Bash
version: '3.8'
services:
  watchtower:
    image: containrrr/watchtower:latest
    container_name: watchtower
    environment:
      TZ: Europe/Stockholm                                                                               
      WATCHTOWER_ROLLING_RESTART: 'true'
      #WATCHTOWER_MONITOR_ONLY: 'true'
      WATCHTOWER_SCHEDULE: '0 0 0 * * 0' #Runs Once A Week (Cron Epression)
      WATCHTOWER_CLEANUP: 'true'
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    restart: unless-stopped
```
