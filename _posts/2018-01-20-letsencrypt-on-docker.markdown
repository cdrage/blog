---
layout: post
category: linux
title: "Using Let's Encrypt with Docker containers"
date: 2018-01-20 11:01
---

This has to do with [docker](https://docker.com), if you don't know what it is, I *highly* suggest you check it out. It's awesome.


Configuring TLS (https certificates) is a pain in the ass. Whether it's figuring out what the correct way to configure NGINX, or throwing certificates in-front of a proxy. There's *always* something that comes up or goes wrong.


[Let's Encrypt](https://letsencrypt.org/) launched in 2016 and effectively made it easy and FREE to create certificates. Even better, they're also introducing [Wildcard support](https://letsencrypt.org/2017/07/06/wildcard-certificates-coming-jan-2018.html).


Anyways. Getting to the point.


Day to day I work with [containers](https://github.com/cdrage/dockerfiles) and one of the biggest annoyances is HTTPS support. While working on my [Seafile](https://www.seafile.com/en/home/) server container (an open source Dropbox-alternative), I was required to configure https in order to get file retrieval working on Android or iOS. Painlessly I tried configuring the container to use TLS certificates and then I thought, why am I doing this? I could just throw a reverse proxy infront of it.


After some research, I found [docker-letsencrypt-nginx-proxy-companion](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion).


Essentially, you throw nginx-proxy and the above container on your Docker host and add a simple environment variable to your container (`-e "VIRTUAL_HOST=foo.bar.com"`). Then BAM, you've got a brand-spanking-new TLS certificate proxying https to your container without ever needing to touch any certificates or NGINX configs.


Here's how you do it.

__Prereqs:__

Make sure you already have an `A Record` of the subdomain pointing towards your host! In this example, we're going to use `example.domain.com`.

__First off, we're going to launch nginx-proxy:__

This automatically forwards traffic to Docker containers when they come up. The brain and braun of this operation.


The only volume we're interested about is `/var/nginx/certs` which will be hosted on your machine.

```sh
docker run -d -p 80:80 -p 443:443 \
    --name nginx-proxy \
    -v /var/nginx/certs:/etc/nginx/certs:ro \
    -v /etc/nginx/vhost.d \
    -v /usr/share/nginx/html \
    -v /var/run/docker.sock:/tmp/docker.sock:ro \
    --label com.github.jrcs.letsencrypt_nginx_proxy_companion.nginx_proxy \
    jwilder/nginx-proxy
```

__Now, we're going to use the Let's Encrypt companion:__

This part is awesome. When a container is launched not only will nginx-proxy quickly restart, but this companion container will automatically generate, retrieve and configure the TLS certificate. IT'S MAGIC.

```sh
docker run -d \
	--name lets-encrypt \
	-v /var/nginx/certs:/etc/nginx/certs:rw \
	-v /var/run/docker.sock:/var/run/docker.sock:ro \
	--volumes-from nginx-proxy \
	jrcs/letsencrypt-nginx-proxy-companion
```

__Let's launch an example container (or a few!):__

Now I've got to prove to you this whole operation works. 


Let's launch an example container!


One prereq: If the container isn't already a port, pass in `--expose PORT`. We'll need this to actually access the container from nginx-proxy.

```sh
docker run -d \
	--expose 80 \
	--name example-app \
	-e "VIRTUAL_HOST=example.domain.com" \
	-e "LETSENCRYPT_HOST=example.domain.com" \
	-e "LETSENCRYPT_EMAIL=youremail@domain.com" \
	tutum/apache-php
```

Alternatively, if your container happens to be on a different port, simply pass in `-e "VIRTUAL_PORT=8080"` to the command line. It doesn't matter if the container is running on port 80!


__Finally, access your service:__

From the public IP address of your host. Simply go to `example.domain.com` to access your service!


You can do this with ANY container on any port (if you pass in VIRTUAL_PORT). It makes like *way* easier and accessible.


If you enjoyed this blog post, feel free to check our my [Dockerfiles](https://github.com/cdrage/dockerfiles). Specifically, if you wish to host something on your home internet with DynamicDNS (not having to use a VPS), check our my [digialocean-dns](https://github.com/cdrage/dockerfiles/tree/master/digitalocean-dns) container.
