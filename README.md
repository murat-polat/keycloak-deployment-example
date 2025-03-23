# keycloak-deployment-example
Keycloack production deployment with secure SSL/HTTPS, and Caddy server



### This  is a simple example  for deploying a Keycloak Docker application, to production environment.Application will be secure SSL/HTTPS via Caddy server.




### What we need ?

- Linux machine/VM 
- Domainname
- Docker and Docker-compose
- Keycloak Dockerfile or Docker-compose file
- Caddy server and reverse proxy configuration for HTTPS/SSL

### Linux VM :
You can order a Linux VM from the Digital Ocean, Contabo, Vultr, Linode, Mvps etc. For this tutorial, we choose an Ubuntu24.04  instance on [MVPS](https://www.mvps.net/).
After  first login as a root user, run:

`sudo apt update`

`sudo apt upgrade -y`


Optional : If you want to change your root password:

`passwd`  

than change your root password

We will run Keycloak inside a Docker container. To run Docker as a root user is not good choice.
Therefore we need to create a new user, and give it admin privileges.

`sudo adduser <newuser>`

`sudo usermod -aG  sudo <newuser>`

Change root user to new user

`su <newuser>`

## Install Docker and Docker-compose:


### Add Docker's official GPG key:
```
sudo apt-get update

sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
### Add the repository to Apt sources:
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
### To install the latest version, run:

`sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`

### Give to the current user to run Docker and Docker-compose without "sudo" permissions

`sudo usermod -aG docker $USER `

Than reboot your server 

`sudo reboot`

Check if Docker runs properely without "permissions denied" etc.

`docker version`

```
mpo@vps:~$ docker version
Client: Docker Engine - Community
 Version:           28.0.2
 API version:       1.48
 Go version:        go1.23.7
 Git commit:        0442a73
 Built:             Wed Mar 19 14:36:49 2025
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          28.0.2
  API version:      1.48 (minimum version 1.24)
  Go version:       go1.23.7
  Git commit:       bea4de2
  Built:            Wed Mar 19 14:36:49 2025
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.7.25
  GitCommit:        bcc810d6b9066471b0b6fa75f557a15a1cbf31bb
 runc:
  Version:          1.2.4
  GitCommit:        v1.2.4-0-g6c52b3f
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

```


### Install Docker Compose plugin
```
sudo apt-get update
sudo apt-get install docker-compose-plugin
```

Check if Docker compose runs:

```
mpo@vps:~$ docker compose version
Docker Compose version v2.34.0

```




### Domain configuration

We need a domain name for publishing our application in World Wide Web. We choose a domainname from the https://www.namecheap.com. And we must tell the domain provider which server/IP provider will be used for publishing. Our Keycloak application will be served on MVPS. From  domain list => management => Nameservers => Custom DNS add three nameservers (ns1.mvps-hosted.com, ns1.mvps-hosted.com, ns1.mvps-hosted.com) and save.

![](/src/doaminNamecheap.png)

From MVPS "DNS Zone Management", add your domainname to "Domain", and  "Target IP address" select your VM instance/IP from dropdown menu => then click "create zone".

![](/src/DNS_register_server_side.png)





Now everything is working properly with HTTP on port 8000. But we need SSL/HTTPS,and configure Caddy server for reverse proxy.



### Caddy Server configuration
#### Installation:

https://caddyserver.com/docs/install#debian-ubuntu-raspbian 

```
sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https


curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo gpg --dearmor -o /usr/share/keyrings/caddy-stable-archive-keyring.gpg

curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list

sudo apt update

sudo apt install caddy


```

`sudo systemctl start caddy `

`sudo systemctl enable caddy`

`sudo systemctl status caddy`


```
mpo@vps:~$ sudo systemctl status caddy
● caddy.service - Caddy
     Loaded: loaded (/usr/lib/systemd/system/caddy.service; enabled; preset: enabled)
     Active: active (running) since Sun 2025-03-23 18:28:58 UTC; 3min 35s ago
       Docs: https://caddyserver.com/docs/
   Main PID: 3071 (caddy)
      Tasks: 7 (limit: 4610)
     Memory: 10.0M (peak: 10.5M)
        CPU: 126ms
     CGroup: /system.slice/caddy.service
             └─3071 /usr/bin/caddy run --environ --config /etc/caddy/Caddyfile

Mar 23 18:28:58 vps caddy[3071]: {"level":"warn","ts":1742754538.7090259,"logger":"http.auto_https","msg":"server is listening only on the>
Mar 23 18:28:58 vps caddy[3071]: {"level":"info","ts":1742754538.7093666,"logger":"tls.cache.maintenance","msg":"started background certif>
(END)

```


### Keycloak Docker compose configuration


#### Coming soon ................






#### Configuration reverse proxy with Caddyfile

`sudo nano /etc/caddy/Caddyfile`

```
yourdomain.com {

        reverse_proxy  localhost:8080

}

```

`sudo systemctl reload caddy`






