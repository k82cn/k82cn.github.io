---
layout: post
title: Docker&#58; Install docker-machine and docker-compose
categories: 
---

##docker-machine

    curl -L https://github.com/docker/machine/releases/download/v0.5.0/docker-machine_linux-amd64.zip >machine.zip && \
         unzip machine.zip && \
         rm machine.zip && \
         mv docker-machine* /usr/local/bin

##docker-compose

    curl -L https://github.com/docker/compose/releases/download/1.5.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose
    docker-compose --version

