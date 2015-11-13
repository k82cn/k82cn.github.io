---
layout: post
title: Mesos Tutorial (1) -- Dynamic Reservation
---

## Dynamic Reservation description

This feature is to provide better support for running stateful services on Mesos such as HDFS (Distributed Filesystem), Cassandra (Distributed Database), or MySQL (Local Database).

Current resource reservations (henceforth called "static" reservations) are statically determined by the slave operator at slave start time, and individual frameworks have no authority to reserve resources themselves.  Dynamic reservations allow a framework to dynamically reserve offered resources, such that those resources will only be re-offered to the same framework (or other frameworks with the same role). This is especially useful if the framework's task stored some state on the slave, and needs a guaranteed set of resources reserved so that it can re-launch a task on the same slave to recover that state.

## Architecture Overview



## API



## Example of Dynamic Reservation

Refer to [dynamic_reservation_framework.cpp](https://github.com/klaus1982/mesos-tutorial/blob/master/src/dynamic_reservation/dynamic_reservation_framework.cpp) for the source code of example


## Troubleshooting

1. Using '\*' as Role for dynamic reservation
1. Failed to unreserve resources when framework exit
1. RESERVE/UNRESERVE logs in master

