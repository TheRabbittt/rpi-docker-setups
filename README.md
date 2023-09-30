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
* Static Ip on the Raspberry Pi Is Highly Recommended.

## Installing Docker
To Get Started We Need to Install ```Docker``` & ```Docker-Compose```.

Note: Installation Varies Depending on OS or Distro, Read and Follow the Instructions on How to Do It on Your OS/Distro, [Dockers Official Site](https://docs.docker.com/Desktop/Install/Debian/)
#### Docker on Debian 
``` Bash
Curl -Ssl https://get.docker.com | SH
```
If You’re Smarter Than Me, Avoid Running Linux as Admin/Root. Add Your User to the Docker Group to Skip Typing Sudo for Docker. You Might Have to Log Out and Log Back in for This to Work.
``` Bash
Sudo Usermod -Ag Docker ${User}
```
Verify That Docker Is Installed by Running Your Very First Container.
``` Bash
Docker Run Hello-World
```
Install Docker Compose 
``` Bash
Sudo Apt-Get Install Docker-Compose-Plugin
```
Verify That It Worked
``` Bash
Docker-Compose Version
```

## Installing Portainer
I Like to Keep Directories of Each Service That I Am Using and Putting the Correspondent docker-compose.yml File in Those Directories. 
##### Portainer Compose File
``` Bash
Version: “3”
Services:
  Portainer:
    Image: Portainer/Portainer-Ce:Latest
    Ports:
      - 9443:9443
      Volumes:
        - Data:/Data
        - /Var/Run/docker.sock:/Var/Run/docker.sock
    Restart: Unless-Stopped
Volumes:
  Data:
```
Run Portainer Using Compose File
``` Bash
Docker Compose up -D
```
