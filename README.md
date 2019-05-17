# nginx-certbot
Create and automatically renew website SSL certificates using the letsencrypt free certificate authority, and its client *certbot*, built on top of the nginx server.

# forked from
I forked this from: https://github.com/staticfloat/docker-nginx-certbot
I did this because I need to lock down the image as the dockerhub published version
only uses the latest tag and docker-compose doesn't appear to be able to reference a sha hash for the image.

## Modifications
### docker-compose
I've added a docker-compose for building this image and publishing it to my dockerhub.  This replaces the functionality in the original Makefile.
Use the following:
```
docker-compose build
docker-compose push
```

# More information

Find out more about letsencrypt: https://letsencrypt.org

Certbot github: https://github.com/certbot/certbot

This repository was originally forked from `@henridwyer`, many thanks to him for the good idea.  I've rewritten about 90% of this repository, so it bears almost no resemblance to the original.  This repository is _much_ more opinionated about the structure of your webservers/containers, however it is easier to use as long as all of your webservers follow that pattern.

# Usage

Create a config directory for your custom configs:
```
mkdir conf.d
```

And a `.conf` file such as in that directory:
```nginx
server {
    listen              443 ssl;
    server_name         server.company.com;
    ssl_certificate     /etc/letsencrypt/live/server.company.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/server.company.com/privkey.pem;

    location / {
        ...
    }
}
```

Wrap this all up with a `docker-compose.yml` file:
```yml
version: '3'
services:
    frontend:
        restart: unless-stopped
        build: frontend
        ports:
            - 80:80/tcp
            - 443:443/tcp
        environment:
            - CERTBOT_EMAIL=owner@company.com
        volumes:
          - ./conf.d:/etc/nginx/user.conf.d
  ...
```

# Changelog

### 0.8
- Ditch cron, it never liked me anway.  Just use `sleep` and a `while` loop instead.

### 0.7
- Complete rewrite, build this image on top of the `nginx` image, and run `cron`/`certbot` alongside `nginx` so that we can have nginx configs dynamically enabled as we get SSL certificates.

### 0.6
- Add `nginx_auto_enable.sh` script to `/etc/letsencrypt/` so that users can bring nginx up before SSL certs are actually available.

### 0.5
- Change the name to `docker-certbot-cron`, update documentation, strip out even more stuff I don't care about.

### 0.4
- Rip out a bunch of stuff because `@staticfloat` is a monster, and likes to do things his way

### 0.3
- Add support for webroot mode.
- Run certbot once with all domains.

### 0.2
- Upgraded to use certbot client
- Changed image to use alpine linux

### 0.1
- Initial release
