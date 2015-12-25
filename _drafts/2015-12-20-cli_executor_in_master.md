---
layout: post
title: Mesos&#58; Add command line executor in Master
categories: tech
---

## Description

Currently, if framework submit a task using command line executor, the master did not know the executor information which is built by Slave. The resources of executors are also overcommitted at slave.

This is to be Post-MVP of [Optimistic Offer Phase 1](https://issues.apache.org/jira/browse/MESOS-1607); because slave has already known enough information to evict executors.

## Design Detail

1. Move `Slave::getExecutorInfo()` to `Master::getExecutorInfo()`
2. Cut 0.1 CPU/32M memory from task to run cli executor
3. Slave report sandbox directory, launch directory to master

## Review Requests

[https://reviews.apache.org/r/41302/](https://reviews.apache.org/r/41302/)
[https://reviews.apache.org/r/41305/](https://reviews.apache.org/r/41305/)
[https://reviews.apache.org/r/41306/](https://reviews.apache.org/r/41306/)
[https://reviews.apache.org/r/41308/](https://reviews.apache.org/r/41308/)


