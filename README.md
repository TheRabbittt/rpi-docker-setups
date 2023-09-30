# Self-Hosted Docker Services Setup
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
docker run hello-world
```
Install Docker Compose 
``` Bash
sudo apt-get install docker-compose-plugin
```
Verify That It Worked
``` Bash
docker-compose version
```

## Installing Portainer
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

## Installing PiHole + Cloudflared
Same Idea as before, create a directory and copy paste this into a docker-compose.yml file.
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

