# QuickDockerWebserver

// https://github.com/nginx-proxy/nginx-proxy

Create a directory on the host machine /websites.
Under this directory, you'll keep the local git repos and a single dockerfile.

```
FROM ubuntu
RUN apt-get update
RUN apt-get install nginx -y
EXPOSE 80
CMD ["nginx","-g","daemon off;"]
```
The repos each have a www folder which is mapped to /var/www/html in their own docker container
To stand up the webserver from scratch:

```
docker network create nginx-proxy

docker run -d -p 80:80 --name nginx-proxyc --restart=always --net nginx-proxy -v /var/run/docker.sock:/tmp/docker.sock jwilder/nginx-proxy
```
For each website you want to puslish, run this command from the /websites directory (wherever the dockerfile lives)
```
docker run -d --name yourwebsite --restart=always --expose 80 -v /websites/yourwebsite/www:/var/www/html --net nginx-proxy -e VIRTUAL_HOST=yourwebsite.meh local-webserver
```

