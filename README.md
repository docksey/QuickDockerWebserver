# QuickDockerWebserver

// https://github.com/nginx-proxy/nginx-proxy

I have a directory on the host machine /websites.
Under this directory I keep the local git repos.
Each of these is mapped to 

```
docker network create nginx-proxy

docker run -d -p 80:80 --name nginx-proxy --net nginx-proxy -v /var/run/docker.sock:/tmp/docker.sock jwilder/nginx-proxy

docker run -d --name igotkindahigh --expose 80 -v /websites/IGotKindaHigh/www:/var/www/html --net nginx-proxy -e VIRTUAL_HOST=igotkindahigh.today local-webserver
```
