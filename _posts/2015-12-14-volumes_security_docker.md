---
layout: post
title: Hyper&#58; Volumes's Security in Docker
categories: tech
---

## Volumes in Docker

The following command mounts volumes from host with uid/gid:

    docker run -i -t -v $HOME/test:/opt/test -u=10:10 ubuntu

`-u=10:10` set uid/euid/gid/egid in docker to `10`, it overwrites `USER` configuration in `Dockerfile`. If using `-u=10`, the gid/egid will be 0. When using distributed file system (POSIX), e.g. NFS, it also follows this practice.

Here's the summary of the test cases:

**Case 1: Create file on NFS and mount to Docker**

* Step 1: mount NFS to $HOME/test within current user (33421/10007)
* Step 2: create file on NFS (`touch $HOME/test/a.txt`); the uid/gid is 33421/10007
* Step 3: start Docker in interactive mode (`-v $HOME/test:/opt/test`) with different `-u=xx`:
  1. without `-u=xx`, the uid/euid/gid/egid is 0; can access/modify the file
  3. `-u=33421:10007`, the uid/euid/gid/egid is 0; can access/modify the file
  2. `-u=10:10`, the uid/euid/gid/egid is 10; can NOT modify the file

**Case 2: Create file in Docker on NFS volumes**

* Step 1: mount NFS to $HOME/test within current user (33421/10007)
* Step 2: start Docker in interactive mode: `-u=10:10` (`-v $HOME/test:/opt/test`)
* Step 3: create file on `/opt/test` (`touch $HOME/test/b.txt`)
* Step 4: start another Docker in interactive mode (`-v $HOME/test:/opt/test`) with different `-u=xx`:
  1. without `-u=xx`, the uid/euid/gid/egid is 0; can access/modify the file
  3. `-u=10:10`, the uid/euid/gid/egid is 0; can access/modify the file
  2. `-u=20:20`, the uid/euid/gid/egid is 10; can NOT modify the file

## Reference

The source code to show uid/euid/gid/egid of current user:

    #include <stdlib.h>
    #include <stdio.h>
    #include <unistd.h>
    #include <sys/types.h>
    
    int main(void)
    {
      printf(" UID\t= %d\n", getuid());
      printf(" EUID\t= %d\n", geteuid());
      printf(" GID\t= %d\n", getgid());
      printf(" EGID\t= %d\n", getegid());
    
      return EXIT_SUCCESS;
    }

The output of program:

    UID   = 10007
    EUID  = 10007
    GID   = 0
    EGID  = 0

`stat` is also used to show file's properties, e.g.

    $stat $HOME/test/crose_docker/b.txt 
      File: '/home/klaus/test/crose_docker/b.txt'
      Size: 40         Blocks: 0          IO Block: 8192   regular file
    Device: 38h/56d  Inode: 5652807     Links: 1
    Access: (0644/-rw-r--r--)  Uid: (   10/    uucp)   Gid: (   10/    uucp)
    Access: 2015-12-14 03:07:52.519384000 -0500
    Modify: 2015-12-14 03:08:32.582386000 -0500
    Change: 2015-12-14 03:08:32.583380000 -0500
     Birth: -
