---
layout: post
title: Mesos (2) -- User Defined Resources
---

## Motivation

Mesos supports pre-defined resource to run tasks, e.g. CPU, memory, disk and port. But in datacenter, some frameworks ask special hardwares (GPU, Xenphi) as resources. In order to support the un-defined hardwares, Mesos need to provide a module interface to the customer to report hardware information as resources.

## Goals

* Provide a module interface for user to define resources
* Provide a way for user to filter the user-defined resources for framework

## Non-Goals

* Add more resource type, e.g. map

## Solution Overview

### Resource Information Collector

In the compute host, slave collects host's information when startup and report it to the master. There're two ways for the slave to collect it:

* Assign available or virtual resources by "--resources" paramenter when start the slave
* Collect host's information by pre-defined function: only CPU, memory, disk and port are collected

To support user-defined resources, a new interface is introduced in ResourceEstimator; the new ResourceEstimator module interface is:

    class ResourceEstimator {
      public:
        virtual Future<Resources> resources() = 0;
        virtual Future<Resources> oversubscribed() = 0;
    };


This module is also configured by --resource_estimator parameter when start slave. When slave started, the "resources" function is executed to get the total resource in the compute hosts; and those resources information is reported to the master.

### Filters

In the datacenter, some frameworks only ask special hardware to run the task: it requests all offers with special hardware, e.g. GPU, and rejects all offers without that hardware. In this feature, Mesos provide a new attributes in FrameworkInfo to request the necessary resources only; if a filters is also submitted by tasks, the offers meeting both fillters (Framework & Task) will be provided. The resources filters in FrameworkInfo is not renewable after driver started and it'll not expire. Refer to the following code slices for protobuf define of Filters:

    message Filter {
      enum FilterType {
        REFUSE = 1;
        ACCEPT = 2;
      }
  
      FilterType type = 1;
  
      Resources resources = 2;
    }
  
    message Filters {
      optional double refuse_seconds = 1;
      repeated Filter filters = 2;
    }


## Example of User-Defined Resources

### Example estimator:

    class GpuResourceEstimator : public ResourceEstimator
    {
    public:
      process::Future<Resources> oversubscribed()
      {
        // Noop.
        return Resources();
      }

      process::Future<Resources> resources()
      {
        return Resources::parse("gpus:2");
      }
    };

In this estimator, it report 2 GPU to slave; so framework will launch tasks in this slave which requires GPU hardware.

### Example framework:

    class GpuScheduler : public Scheduler
    {
    public:
      virtual void resourceOffers(SchedulerDriver* driver,
                                  const vector<Offer>& offers)
      {
        foreach (const Offer& offer, offers) {
          static const Resources TASK_RESOURCES = Resources::parse("gpus:1").get();
          ... // Launch tasks.
          driver->launchTasks(offer.id(), tasks);
        }
      }
    };

    int main(int argc, char** argv)
    {
      ...
      Filters filters;
      Filter filter;
  
      // Define a filter to only accept GPU resources.
      filter.mutable_type().set_value(Filter::ACCEPT);
      filter.mutable_resources()->add_resources()->CopyFrom(Resources::parse("gpus:1"));
      filters.mutable_filters->add_filters()->CopyFrom(filter);
  
      FrameworkInfo framework;
      framework.set_user(""); // Have Mesos fill in the current user.
      framework.set_name("Test Framework (C++)");
      framework.set_role(role); 
      // Add filters to framework
      framework.mutable_filters()->CopyFrom(filters);
      ...
    }

### Expected Result:

* Two tasks run on the slave, and finish successfully
* Get 1 GPU offer in the log

## Reference

N/A
