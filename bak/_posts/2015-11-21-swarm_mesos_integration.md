---
layout: post
title: Hyper&#58; Swarm on Mesos
categories: tech
---

## Swarm on Mesos Overview

![Swarm+Mesos](/images/SwarmMesosIntegration.png)

1. Swarm API gets REST API request, and then create a task and put it into message queue. 
2. Swarm Cluster (registered as Mesos framework) gets offer from Mesos
3. Swarm Cluster picks up a task from message queue 
4. Swarm Cluster calls Scheduler’s routine to gets target host 
5. Swarm Cluster calls Mesos API to create Docker container in target host.
6. Mesos master creates Docker container in Mesos cluster
7. Mesos sends the container notification back to Swarm Cluster (planned after Mesos 0.23+, currently still replies on Swarm Cluster calling Docker API via Engine)
8. Swarm Cluster update container into to task channel
9. Swarm API gets result of request and then return as REST API result

## Docker

![Swarm on Mesos](/images/SwarmOnMesos.png)

Both Swarm manager & Mesos Slave will communicate with docker daemon: Swarm access Docker daemon through REST API, such as `docker ps; Mesos run docker image by docker command through unix domain socket. So the docker daemon in each node much accept tcp & unix domain socket as follow:

    docker daemon -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock


## Mesos 

Each node in your swarm must run a Mesos slave. The slave must be capable of starting tasks in a Docker Container using the `--containerizers=docker` option. For example:

    mesos-slave --containerizers=docker,mesos --master=zk://zknode:2181/mesos --isolation=cgroups


## Swarm Manager

Start Swam cluster by following command:

    swarm --debug manage -c mesos-experimental \
        --cluster-opt mesos.address=0.0.0.0 \
        --cluster-opt mesos.tasktimeout=10m \
        --cluster-opt mesos.user=root \
        --cluster-opt mesos.offertimeout=1m \
        --cluster-opt mesos.port=3375 --host 0.0.0.0:4375 zk://zhhost1:2181/mesos

## Reference

* [github: swarm/cluster/mesos](https://github.com/docker/swarm/tree/master/cluster/mesos)
* [Mesos Agent Configuration](http://mesos.apache.org/documentation/latest/configuration/)
* [Swarm API on Mesos](/tech/2015/11/20/swarm_on_mesos/)

