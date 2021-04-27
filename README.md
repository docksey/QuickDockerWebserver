# QuickDockerWebserver

// https://github.com/nginx-proxy/nginx-proxy

To stand up the webserver from scratch:

## The Needful
```
sudo apt update
sudo apt upgrade
```
## Install Firewall and Configure
```
sudo apt install ufw
sudo ufw allow openssh
sudo ufw enable
sudo ufw default deny incoming
```
## Install Docker
Remove existing Docker shizz (as needed)
```
sudo apt remove docker docker-engine docker.io containerd runc
sudo apt update
```
Update the apt package index and install packages to allow apt to use a repository over HTTPS:
```
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-releasey
```
Add Docker’s official GPG key:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
 | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
Set up the docker repository
```
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
 | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Install Docker
```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
```
## Install NGinx Proxy
*with support for let's encrypt companion*
*(https://cloud.google.com/community/tutorials/nginx-reverse-proxy-docker)*
```
sudo docker network create nginx-proxy
sudo docker run -d -p 80:80 -p 443:443 \
    --name nginx-proxy \
    --net nginx-proxy \
    -v /srv/TLS:/etc/nginx/certs:ro \
    -v /etc/nginx/vhost.d \
    -v /usr/share/nginx/html \
    -v /var/run/docker.sock:/tmp/docker.sock:ro \
    --label com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy=true \
    jwilder/nginx-proxy
```
## Install Let's Encrypt Companion
```
sudo docker run -d \
    --name nginx-letsencrypt \
    --volumes-from nginx-proxy \
    -v /srv/TLS:/etc/nginx/certs:rw \
    -v /var/run/docker.sock:/var/run/docker.sock:ro \
    -e NGINX_PROXY_CONTAINER=nginx-proxy \
    jrcs/letsencrypt-nginx-proxy-companion
```
## Install Portainer
Create a docker volume for portainer data
```
sudo docker volume create portainer_data
```
Create a password file *(plain text, we'll delete this after, so store the password)*
```
echo 'someadminpassword' > /tmp/poop.txt
```
Install Portainer
```
sudo docker run -d \
    -p 8000:8000 \
    --name portainer \
    --restart=always \
    --net nginx-proxy \
    --expose 9000 \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v portainer_data:/data \
    -v /tmp:/tmp \
    -e LETSENCRYPT_EMAIL=your.techteam@mailserver.com \
    -e LETSENCRYPT_HOST=portainer.yourdomain.blah \
    -e VIRTUAL_HOST=portainer.yourdomain.blah \
    -e VIRTUAL_PORT=9000 \
    portainer/portainer-ce --admin-password-file='/tmp/poop.txt'
```
Deploy websites
Create a directory on the host machine /websites.
Under this directory, you'll keep the local git repos and a single dockerfile.
```
FROM ubuntu
RUN apt-get update
RUN apt-get install nginx -y
EXPOSE 80
EXPOSE 443
CMD ["nginx","-g","daemon off;"]
```
The repos each have a www folder which is mapped to /var/www/html in their own docker container
For each website you want to puslish, run this command from the /websites directory (wherever the dockerfile lives)
```
docker run -d \
    --name yourwebsite \
    --restart=always \
    --expose 80 \
    --expose 443 \
    --net nginx-proxy \
    -v /websites/yourwebsite/www:/var/www/html \
    -e VIRTUAL_HOST=yourwebsite.meh \
    -e LETSENCRYPT_EMAIL=you.techteam@mailserver.com \
    -e LETSENCRYPT_HOST=yourwebsite.meh \
    local-webserver
```

