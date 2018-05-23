---
layout: post
title: Hyper&#58; Swarm API with Mesos
categories: tech
---

![Swarm on Mesos](/images/SwarmOnMesos.png)

## Solution & Estimations:
Current solution is to 1.) let Swarm launch tasks by Mesos 2.) for the other API, let Swarm send request to docker engine directly (red arrow)

### Tasks of the solution (Mesos):

####POST /commit (~3 man month)
  The container ID was changed after commit; Mesos did not know the new container ID. One option is to trace docker container in Mesos by container’s name (UUID).

####POST /containers/copy (Effort: ??)
  Not sure how to copy between remote container.

####POST /containers/kill & stop (Effort: n/a)
  If send this to Docker Engine directly, Mesos will mark it as TASK_FAILED instead of TASK_KILLED. Only support kill, the container will be stopped after killed.

####POST /containers/start & create (Effort: n/a)
  Currently, Mesos launch tasks by “docker run” command (sub-process); so can not support those two API separately.

####POST /containers/exec & start (~1.5 man month)
  Currently, Mesos did not support launch tasks by “docker exec”. Mesos need to support “docker exec”. But still can not support those two APIs separately.

## Mesos:

###CLI parameters: 

- Docker images
- network type (host, bridge)
- port mapping 
- privileged
- parameters (including —label=xxx=yyy)
- force_pull_image

###Docker CLIs used by Mesos:

- docker run (launch task)
- docker version (version verify in Mesos)
- docker stop (kill/stop container)
- docker inspect (update status after the docker started)
- docker pull (get docker image)
- docker ps (reconcile docker status when slave recover)
- docker rm (remove docker images when executor exit)
