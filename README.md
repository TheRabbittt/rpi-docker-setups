# Self-Hosted Docker Services
This includes general configuration steps and docker-compose files, all of this was configured on a Raspberry Pi 4.

---

#### List of Services (Links to Their Respective Github Repositories)
* [Portainer](https://github.com/Portainer/Portainer)
* [Vaultwarden](https://github.com/Dani-Garcia/Vaultwarden)
* [Pihole](https://github.com/Pi-Hole/Pi-Hole)
  * [Cloudflared](https://github.com/Cloudflare/Cloudflared)
* [Nginx Proxy Manager](https://github.com/Nginxproxymanager/Nginx-Proxy-Manager)
* [WireGuard](https://www.wireguard.com/)
* [Watchtower](https://github.com/Containrrr/Watchtower)

---

## Prerequisites
* Enough storage for your particular needs, I used a 128GB Micro SD-card and for all of the services above I have only claimed 8GB of space so far.
* An accessible Rasperry Pi with an installed and updated OS. I had [Raspberry Pi OS](https://www.raspberrypi.com/Software/Operating-Systems/) (64-Bit) installed which is based on debian. Although what you choose is up to you, just make sure you know the commands for that particular distro or OS.
* Static IP on the Raspberry Pi is highly recommended.

## Installing Docker
To get started we need to install ```docker``` & ```docker-compose```.

Docker is a tool used to automate deployment of applications in lightweight containers. Containers are isolated from one another and can be connected through other ways. The benfit to containers compared to VMs is that they use fewer resources because the containers share the services of a single OS.

Docker compose is a tool that makes it easier to deploy single and multiple docker containers. These is where the YAML files come in which is that we use to configure the containers. It is basically just a standard that defines how to configure and start a container.

Note: Installation varies depending on OS or distro, read and follow the instructions on how to do it on your OS/Distro, [Dockers Official Site](https://docs.docker.com/Desktop/Install/Debian/)
#### Docker on Debian 
``` Bash
curl -sSl https://get.docker.com | sh
```
If you’re smarter than me, avoid running Linux as admin/root. Add your user to the docker group to skip typing sudo for docker. You might have to log out and log back in for this to work.
``` Bash
sudo usermod -aG docker ${whoami}
```
Verify that docker is installed by running your very first container.
``` Bash
sudo docker run hello-world
```
Install docker compose 
``` Bash
sudo apt-get install docker-compose-plugin
```
Verify that it worked
``` Bash
docker compose version
```

## Portainer
Portainer is a platform that can be used to manage docker and docker containers. It gives you a pretty GUI to monitor and manage your containers.

Let's begin, I like to keep directories of each service/application that I am using and putting the correspondent docker-compose.yml file in those directories. 

##### Portainer Compose File
``` Bash
services:
  portainer:
    container_name: portainer
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
Run portainer using compose file
``` Bash
docker compose up -d
```
It's that simple, you should know be able to head to https://{raspberrypiIP}:9443 in your browser and reach portainer. 

## Pi-Hole + Cloudflared

Pi-Hole is a DNS sinkhole that is most known for blocking ads when you visit various sites. In my experience it will block a lot of telemtry requests, popups and images on on the sidebars of random sites. Although it will not block youtube ads or twitch ads, this is due to how those ads are delivered. Youtube delivers ads through the youtube domain so for Pi-Hole to block the ads it would have to block the whole youtube domain, which wouldn't be good.

Pi-Hole in combination with cloudflared allows you to enable DNS over HTTPS (DoH). This ensures that your DNS queries are encrypted, protecting them from being seen or manipulated.

``` Bash
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
        ipv4_address: 172.20.0.10 

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
After running the docker compose yml you should be able to reach pihole through http://{raspberrypi_ip}:8061/admin. Login password should be "changeme" although you should change the password which you can do by going into the docker container.

``` Bash
docker exec -it <container_id> bash
pihole -a -p <password>
```

For the finale configuration, go into settings in pihole and change the upstream DNS to the docker container IP addresses of cloudflared. Now that should be it for the raspberry pi, change your DNS server (typically your router) to point to the raspberry pi and boom.... you are done.

<img src="https://github.com/user-attachments/assets/ec1399ff-109d-4830-ad8d-698d63ad988f" width="700" />

## Bitwarden/Vaultwarden
Bitwarden is a password manager and vaultwarden is a more lightweight option that you can host yourself. This works with the bitwarden app and extension. 

``` Bash
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

Once the container is up you should be able to reach bitwarden through http://{raspberrypiIP}:8080, although you won't be able to create an account or use it just yet. Bitwarden needs to go through HTTPS otherwise errors will occur. There are multiple ways of doing this, one way is through a reverse proxy which I found to be the easiest. 

## NGINX Proxy Manager

This is a proxy manager which you use to manage your existing domains and to what application a domain should reach. With this you can have domain names for your applications, for example instead of typing in the Pi-Hole IP each time you want to reach the GUI you can create a domain such as this:pihole.website.io. Obviously this wouldn't be public and only devices that can reach the NGINX Proxy Manager will be able to take advantage of this. This is a video that shows how to do it although I used deSec which is free (https://www.youtube.com/watch?v=qlcVx-k-02E)

``` Bash
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

## Wireguard VPN
[VPN to reach local network and use pihole from outside of your home]
``` Bash
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
Watchtower kekeps containers up to date automatically, in the docker compose file you can see watchtower_schedule, this shows how often it will run. Change this to your liking. Once you run it, it's done.... if you want to configure it to ignore certain containers look up the documentation. It is pretty simple, in my case it is not needed. To check if it working you can type in
``` Bash
docker logs watchtower
```

``` Bash
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
