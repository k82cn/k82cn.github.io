---
layout: post
categories: tech
title: Enable NFS in Ubuntu 24.04
---

## Motivation

A quick steps to enable NFS for test environment.

## NFS Host

### Install NFS

```
$ sudo apt install nfs-kernel-server
```

### Export mount point

```
$ sudo mkdir -p /var/nfs/dpf
$ sudo vim /etc/exports
/var/nfs/dpf *(insecure,rw,sync,no_subtree_check)
```

*NOTE*:

- `*` means any client connect to this NFS server for mount point
- `insecure` is only used for test/dev environment

### Restart NFS service

```
$ sudo systemctl restart nfs-kernel-server
```

## NFS Client

### Install NFS

```
$ sudo apt install nfs-common
```

### Mount Remove Directory

```
mount 10.100.3.55:/var/nfs/dpf /opt/dpf
```

## References

* [How to Set Up an NFS Mount on Ubuntu (Step-by-Step Guide)](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-20-04)

