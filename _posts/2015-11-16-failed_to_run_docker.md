---
layout: post
categories: tech
title: Docker&#58; failed to add interface veth0f084cf to sandbox
---

## Error Message:

    Error response from daemon: Cannot start container 848ac591eec5ae7a0ada1a84
    5fb588615e00331a99fa9d72631ce755d0e01158: [8] System error: failed to add 
    interface veth0f084cf to sandbox: failed in prefunc: failed to set namespace on
    link "veth0f084cf": invalid argument

## Command:

    sudo update-rc.d autoprotect disable

