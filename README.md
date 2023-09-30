# Self-Hosted Docker Services Setup
This includes general configuration steps and includes docker-compose files, all of this was configured on a Raspberry Pi 4.

---

#### List of Services (Links To Their Respective GitHub Repositories)
* [Portainer](https://github.com/portainer/portainer)
* [Vaultwarden](https://github.com/dani-garcia/vaultwarden)
* [PiHole](https://github.com/pi-hole/pi-hole)
  * [Cloudflared](https://github.com/cloudflare/cloudflared)
* [Nginx_Proxy_Manager](https://github.com/NginxProxyManager/nginx-proxy-manager)
* [Watchtower](https://github.com/containrrr/watchtower)

---

## Prerequisites
* Enough storage for your particular needs, I used a 128GB micro SD-card and for all the services above I have only claimed 8GB of space so far.
* An accessible Rasperry Pi with an installed and updated OS. I had [Raspberry Pi OS](https://www.raspberrypi.com/software/operating-systems/) (64-bit) installed which is based on Debian Linux. Although what you choose is up to you, just make sure you know the commands for that particular distro or OS.

## Installing Docker
To get started we need to install ```Docker``` & ```Docker-Compose```.

NOTE: Installation varies depending on OS or Distro, read and follow the instructions on how to do it on your OS/Distro, [Dockers official site](https://docs.docker.com/desktop/install/debian/)
### Docker on Debian 
``` bash
curl -sSL https://get.docker.com | sh
```
If you're smarter than me, avoid running Linux as admin/root. Add your user to the docker group to skip typing sudo for docker.
``` bash
sudo usermod -aG docker ${USER}
```
You might have to log out and log back in for this to work.

To test if docker is working you can run your very first container by typing in "docker run hello-world", it should pull the image ""hello-world" and run it. 

To install docker compose you type in: 
sudo apt-get install docker-compose-plugin
type in docker-compose version to verify that it is installed










