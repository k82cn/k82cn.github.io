---
layout: post
title: Hyper&#58; Security in Kubernetes
categories: tech
---

## Goal of K8S Security

1. Ensure a clear isolation between the container and the underlying host it runs on
2. Limit the ability of the container to negatively impact the infrastructure or other containers
3. [Principle of Least Privilege](http://en.wikipedia.org/wiki/Principle_of_least_privilege) - ensure components are only authorized to perform the actions they need, and limit the scope of a compromise by limiting the capabilities of individual components
4. Reduce the number of systems that have to be hardened and secured by defining clear boundaries between components
5. Allow users of the system to be cleanly separated from administrators
6. Allow administrative functions to be delegated to users where necessary
7. Allow applications to be run on the cluster that have "secret" data (keys, certs, passwords) which is properly abstracted from "public" data.


## Roles

1. k8s admin - administers a Kubernetes cluster and has access to the underlying components of the system
2. k8s project administrator - administrates the security of a small subset of the cluster
3. k8s developer - launches pods on a Kubernetes cluster and consumes cluster resources

## Service Account

A service account binds together several things:

1. a name, understood by users, and perhaps by peripheral systems, for an identity
2. a principal that can be authenticated and authorized
3. a security context, which defines the Linux Capabilities, User IDs, Groups IDs, and other capabilities and controls on interaction with the file system and OS.
4. a set of secrets, which a container may use to access various networked resources.

## Service Context

A security context is a set of constraints that are applied to a container in order to achieve the following goals (from security design):

1. Ensure a clear isolation between container and the underlying host it runs on
2. Limit the ability of the container to negatively impact the infrastructure or other containers

## Namespace

A single cluster should be able to satisfy the needs of multiple users or groups of users (henceforth a 'user community').

Each user community wants to be able to work in isolation from other communities. Each user community has its own:

1. resources (pods, services, replication controllers, etc.)
2. policies (who can or cannot perform actions in their community)
3. constraints (this community is allowed this much quota, etc.)

A cluster operator may create a Namespace for each unique user community. The Namespace provides a unique scope for:

1. named resources (to avoid basic naming collisions)
2. delegated management authority to trusted users
3. ability to limit community resource consumption


