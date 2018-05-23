---
layout: post
title: Docker&#58; Install docker-machine and docker-compose
categories: 
---

## docker-machine

```
DOCKER_MACHINE=https://github.com/docker/machine/releases/download/v0.5.0/docker-machine_linux-amd64.zip
curl -L $DOCKER_MACHINE > machine.zip && \
  unzip machine.zip && \
  rm machine.zip && \
  mv docker-machine* /usr/local/bin
```

## docker-compose

```
DOCKER_COMPOSE=https://github.com/docker/compose/releases/download/1.5.1/docker-compose-`uname -s`-`uname -m`
curl -L ${DOCKER_COMPOSE} > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

