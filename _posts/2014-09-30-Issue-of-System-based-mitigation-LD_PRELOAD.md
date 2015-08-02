---
layout: post
title: Issue of “System-based mitigation - LD_PRELOAD”
---

In [Mitigating the shellshock vulnerability (CVE-2014-6271 and CVE-2014-7169)](https://access.redhat.com/articles/1212303), RedHat provide a System-based mitigation by LD_PRELOAD; but there is an issue that it generates core dump when running /lib64/libc.so.6.

## Steps:

### Compile the patch from bash_ld_preload.c:

    $ gcc bash_ld_preload.c -fPIC -shared -Wl,-soname,bash_ld_preload.so.1 -o bash_ld_preload.so

### Apply the patch for current terminal:

    LD_PRELOAD=/home/dma/bash_ld_preload.so
    export LD_PRELOAD

### Segmentation fault when running /lib64/libc.so.6:

    [dma@bjr610-04 ~]$ setenv LD_PRELOAD /home/dma/bash_ld_preload.so
    [dma@bjr610-04 ~]$ /lib64/libc.so.6
    Segmentation fault
    [dma@bjr610-04 ~]$ ls
    bash_ld_preload.c  bash_ld_preload.so  bigdata  build_cscope_db.sh  core.11613  flexlm  include  jazz  lib  pcc  software  vem_ext  workspace
    [dma@bjr610-04 ~]$ file core.11613
    core.11613: ELF 64-bit LSB core file AMD x86-64, version 1 (SYSV), SVR4-style, from 'libc.so.6'
    [dma@bjr610-04 ~]$

## Options:

* Option 1: Update bash to the latest version; RH has published patch at [https://access.redhat.com/security/cve/CVE-2014-7169](https://access.redhat.com/security/cve/CVE-2014-7169)
* Option 2: Update the patch to check whether environ is null before using it
