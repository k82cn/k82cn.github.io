---
layout: post
title: Build k8s dev VM environment
categories: tech
---

## Creat second disk for k8s

In `Vagrantfile`, add the following customized command to create disk for k8s source code.
The vagrant/virtual box will create a disk with only 10Gi by default, which is not enough for k8s's build & test.

```
  config.vm.provider "virtualbox" do |v|
    unless File.exist?(disk)
      v.customize ['createhd', '--filename', disk, '--variant', 'Fixed', '--size', 40 * 1024]
    end
	v.customize ['storageattach', :id,
		'--storagectl', 'SCSI',
		'--port', 2, '--device', 0,
		'--type', 'hdd', '--medium', disk]

    v.memory = 1024 * 12
    v.cpus = 8
  end

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/xenial64"
```

In `storageattach` command, it's better to set `port` to 2; and mount the disk back. Using `fdisk` to creat partition.

```
ubuntu@ubuntu-xenial:~$ sudo fdisk /dev/sdc

Welcome to fdisk (util-linux 2.27.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xba0f14ad.

Command (m for help): n
Partition type


   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (1-4, default 1):
First sector (2048-83886079, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-83886079, default 83886079):

Created a new partition 1 of type 'Linux' and of size 40 GiB.

Command (m for help):


Command (m for help): w

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

```

And then using `mkfs.ext4` to create fs.

```
ubuntu@ubuntu-xenial:~$ sudo mkfs.ext4 /dev/sdc1
mke2fs 1.42.13 (17-May-2015)
Creating filesystem with 10485504 4k blocks and 2621440 inodes
Filesystem UUID: 6ac968f6-0dde-4702-bf31-cc0a5f4fb770
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000, 7962624

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
```

Mount to `/gopath`, edited `/etc/fstab` to add following line. It's better not to
mount to `/opt` which is used for virtual box addon iso.

```
/dev/sdc1	/gopath	 ext4	defaults	0 0
```

## Add docker as one of provision

```
  config.vm.provision "docker"
```

I'd suggest to use this in Vagrantfile; otherwise, you have to config docker group and so on.
