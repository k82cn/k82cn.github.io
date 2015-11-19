---
layout: post
title: Hyper&#58; Kubernetes/Docker Network
categories: 
---


## Native Docker Networking Status
Docker now supports both single-host and multi-host networking this means that both containers on the same Docker host and containers on different Docker hosts can connect with each other with native docker network support. 
For multi-host networking support, docker is using overlay network to support this. An overlay network requires a key-value store. The key-value stores information about the network state which includes discovery, networks, endpoints, ip-addresses, and more. Docker supports Consul, Etcd, and ZooKeeper (Distributed store) key-value stores. 


## Weave

![Weave Overview](/images/Weave.png)

Source Code: [https://github.com/weaveworks/weave](https://github.com/weaveworks/weave)

Weave create a single virtual network to connect all containers. The way is to create a weave router on each host and let the router to forward packet among containers. The weave router encapsulate the incoming packets and route the packet to peer routers.

How does it works:

1. It runs on each Docker host (as a container)
1. It manages the subnet information and ensures the info is synchronised with other Weave routers
1. It routes container packets from the local Docker Bridge to Weave routers on other Docker hosts.


## Flannel

![Flannel Overview](/images/Flannel.png)

Flannel creates an overlay network to encapsulate IP packets. A distributed system etcd is used to do subnet allocation. Each hosts has a host internal private network (same as original docker0 network). Flannel encapsulate the packet with IP address of the host. It is similar to GRE tunnel.

Flannel allows you to create an overlay network for a Kubernetes cluster.

Kubernetes has very specific network requirements. It requires that each Docker host has its own subnet (e.g., host1: 10.10.10.0/24, host2: 10.10.11.0/24, etc..) , and requires creating an overlay on top to join the individual subnets (e.g., 10.10.10.0/16).

Flannel manages the subnet assignments for each Docker host (e.g., host1 is assigned 10.10.10.0/24 subnet, etc...)

How does flannel works:

1. It runs on each Docker host (as a container)
1. It manages the subnet assignments for each Docker host and ensures the info is synchronized with other Flannel daemons using etcd.
1. It routes container packets from the local Docker bridge to Flannel daemons on other Docker hosts, using Universal TAP/TUN devices to encapsulate packets before they're transferred to other Docker hosts


## Kubernetes Network with Tunnel

The tunnel type could be GRE or VxLAN. VxLAN is preferable when large scale isolation needs to be performed within the network.

The docker bridge is replaced with a brctl generated linux bridge (kbr0) with a 256 address space subnet. Basically, a node gets 10.244.x.0/24 subnet and docker is configured to use that bridge instead of the default docker0 bridge.

Also, an OVS bridge is created (obr0) and added as a port to the kbr0 bridge. All OVS bridges across all nodes are linked with GRE tunnels. So, each node has an outgoing GRE tunnel to all other nodes. It does not need to be a complete mesh really, just meshier the better. STP (spanning tree) mode is enabled in the bridges to prevent loops.

Routing rules enable any 10.244.0.0/16 target to become reachable via the OVS bridge connected with the tunnels.

