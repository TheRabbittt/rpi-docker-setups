# Self-Hosted Docker Services Setup
This includes general configuration steps and includes docker-compose files, all of this was configured on a Raspberry Pi 4.

---

#### List of Services (Links To Their Respective GitHub Repositories)
* [Portainer](https://github.com/portainer/portainer)
* [Vaultwarden](https://github.com/dani-garcia/vaultwarden)
* [PiHole](https://github.com/pi-hole/pi-hole)
  * [Cloudflared](https://github.com/cloudflare/cloudflared)
* [Nginx_Proxy_Manager](https://github.com/NginxProxyManager/nginx-proxy-manager)
  * [Nginx_Proxy_DB](https://github.com/NginxProxyManager/nginx-proxy-manager)
* [Watchtower](https://github.com/containrrr/watchtower)

---

## Prerequisites
Enough storage for your particular needs, I used a 128GB micro SD-card and for all the services above I have only claimed 8GB of space so far. You need an accessible Rasperry Pi with an installed and updated OS. I had [Raspberry Pi OS](https://www.raspberrypi.com/software/operating-systems/) (64-bit) installed which is based on Debian Linux. Although what you choose is up to you, just make sure you know the commands for that particular distro or OS. 

## Getting Started
Installing ```Docker``` & ```Docker-Compose```.

NOTE: This varies depending on OS or Distro, if you are not using a debian based distro read on how to do it on your OS/Distro on [Dockers official site](https://docs.docker.com/desktop/install/debian/)








