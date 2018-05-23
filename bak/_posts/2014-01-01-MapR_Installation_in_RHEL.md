---
layout: post
categories: tech
title: MapR Installation in RHEL
---

## Environment

    LSB Version:    :base-4.0-amd64:base-4.0-noarch:core-4.0-amd64:core-4.0-noarch: graphics-4.0-amd64:graphics-4.0-noarch:printing-4.0-amd64:printing-4.0-noarch
    Distributor ID: RedHatEnterpriseServer
    Description:    Red Hat Enterprise Linux Server release 6.4 (Santiago)
    Release:        6.4
    Codename:       Santiago

## SSH Configuration

    Install root, it is SSH keyless

## Disk Configuration

On each node (the real environment needs raw disks, but there are no such diskw now):

    mkdir /opt/mapr-disks
    dd if=/dev/zero of=/opt/mapr-disks/loop0 bs=1G count=8
    /sbin/losetup -f
    /sbin/losetup /dev/loop0 /opt/mapr-disks/loop0
    echo /dev/loop0 > /tmp/disks.txt

or using the file directly:

    mkdir /opt/mapr-disks
    dd if=/dev/zero of=/opt/mapr-disks/loop0 bs=1G count=8
    echo /opt/mapr-disks/loop0 > /tmp/disks.txt

## Ensure no NFS server running no output for
    "rpcinfo -p | grep nfs"

## Download Configuration

create /etc/yum.repos.d/maprtech.repo on each node

    [maprtech]
    name=MapR Technologies
    baseurl=http://package.mapr.com/releases/v3.0.2/redhat/
    enabled=1
    gpgcheck=0
    protect=1

    [maprecosystem]
    name=MapR Technologies
    baseurl=http://package.mapr.com/releases/ecosystem/redhat
    enabled=1
    gpgcheck=0
    protect=1

## Install the packages

Install management node:

    yum install mapr-cldb mapr-fileserver mapr-jobtracker mapr-nfs mapr-tasktracker mapr-webserver mapr-zookeeper

Install compute node run:

    yum install mapr-fileserver mapr-tasktracker mapr-zookeeper

Note: Add /etc/yum.conf with proxy=http://IP:port if necessary.

## Mount disk to MapR-FS

    /opt/mapr/server/configure.sh -C <ManageNodeIP> -Z <ManageNodeIP>,<ComputeNodeIP> -u root
    /opt/mapr/server/disksetup -F /tmp/disks.txt

## Start Zookeeper:

    export JAVA_HOME=/pcc/app/Linux_jdk1.6.0_13_x86_64
    /etc/init.d/mapr-zookeeper start

## Management node:

    /etc/init.d/mapr-warden start
    /opt/mapr/bin/maprcli acl edit -type cluster -user root:fc

Register user: http://mapr.com/register/login.html

Register cluster: http://www.mapr.com/index.php?option=com_comprofiler&task=myclusters&Itemid=0

Get cluster name: /opt/mapr/conf/mapr-clusters.conf

Get id: maprcli license showid

Enable license: // save license to /opt/mrlicense.txt maprcli license add -cluster my.cluster.com -license /opt/mrlicense.txt -is_file true

Start NFS: maprcli node services -nodes ManageNodeIP -nfs start

Start jobtracker: maprcli node services -nodes nodename -jobtracker start

## Compute node:

    /etc/init.d/mapr-warden start
    maprcli node services -nodes nodename -tasktracker start

## Run Sample in MapR-FS

Pi:

    hadoop jar /opt/mapr/hadoop/hadoop-0.20.2/hadoop-0.20.2-dev-examples.jar pi 2 50

Word Count:

    hadoop fs -mkdir input  hadoop fs -put input/* input
    hadoop jar /opt/mapr/hadoop/hadoop-0.20.2/hadoop-0.20.2-dev-examples.jar wordcount /user/root/input /user/root/output
