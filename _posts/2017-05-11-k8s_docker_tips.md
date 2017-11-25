---
layout: post
title: FAQ of k8s, docker and glusterfs
categories: tech
---

## Change Docker Imange location

Edit `/etc/docker/daemon.json` in Ubuntu:

```
{
    "live-restore": true,
    "graph": "/data/docker"
}
```

## Kuberentes Errors

In following erro may because of changing docker image location, no solution for now:

```
3m	3m	1	kubelet, 9.111.143.205		Warning	FailedSync	Error syncing pod, skipping: failed to "StartContainer" for "POD" with RunContainerError: "addNDotsOption: ResolvConfPath \"/data/docker/containers/cdc30b87403ef938964754e34ad3ce5b4c1ceb950a0af72ff1e531c235eaad21/resolv.conf\" does not exist"
```

## Glusterfs Errors

```
Redirecting to /bin/systemctl status  glusterd.service
glusterd.service - GlusterFS, a clustered file-system server
   Loaded: loaded (/usr/lib/systemd/system/glusterd.service; enabled)
   Active: failed (Result: exit-code) since Fri 2014-09-26 01:47:54 EDT; 5min ago
  Process: 25116 ExecStart=/usr/sbin/glusterd -p /run/glusterd.pid (code=exited, status=1/FAILURE)

Sep 26 01:47:54 datasvr1 systemd[1]: glusterd.service: control process exited, code=exited status=1
Sep 26 01:47:54 datasvr1 systemd[1]: Failed to start GlusterFS, a clustered file-system server.
Sep 26 01:47:54 datasvr1 systemd[1]: Unit glusterd.service entered failed state.
```

The root cause is glusterfs can not use non-empty working directory; the solution is to clear
 `working-directory` in `/etc/glusterfs/glusterd.vol`.


