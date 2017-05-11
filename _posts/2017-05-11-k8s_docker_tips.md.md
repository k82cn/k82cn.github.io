---
layout: post
title: k8s and docker tips
categories: tech
---

## Change Docker Imange location

Edit `/etc/docker/daemon.json` in Ubuntu:


    {
        "live-restore": true,
        "graph": "/data/docker"
    }


## Kuberentes Erros

In following erro may because of changing docker image location, no solution for now:

	3m	3m	1	kubelet, 9.111.143.205		Warning	FailedSync	Error syncing pod, skipping: failed to "StartContainer" for "POD" with RunContainerError: "addNDotsOption: ResolvConfPath \"/data/docker/containers/cdc30b87403ef938964754e34ad3ce5b4c1ceb950a0af72ff1e531c235eaad21/resolv.conf\" does not exist"
