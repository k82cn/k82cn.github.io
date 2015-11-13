---
layout: post
categories: tech
title: Storage in Docker & Kubernetes
---


## Docker Volume Plugin

![Docker Volume Plugin](/images/DockerVolumePlugin.png)

The above picture show the overall architecture of Docker volume plugin. The docker daemon communicate with VolumePlugin by volume_driver.sock (Unix Domain Socket); the volume plugin take responsibility to mount the distributed FS into local FS and return the mount point to docker daemon. The docker daemon handle it as normal volume.

**Notes**:

* In docker volume plugin, it trigger docker daemon to call volume plugin if host path did not start with '/' and --volume-dirver is not empty
* Mesos 0.26.0 does not support volume plugin; mesos consider it's related path (against mesos-slave sandbox) if host path did not start with '/'
 * The volume plugin is not necessary to be a separated daemon, but it's recommended
 * Refer to [here](http://docs.docker.com/engine/extend/plugins_volume/) for the detail of Docker Volume Plugin Protocol

## Kubernetes Volume

![Kubernetes Volume](/images/KubernetesVolume.png)

In Kubernetes, kubelet takes the responsibility to mount distributed FS into local FS; it also maintenance the mapping and sent the local mout point when create/start container by Docker API.

## Kubernetes Persistent Volumes and Claims

Ongoing
