---
layout: post
title: Mesos&#58; Oversubscription for Reservation
categories: tech
---

## Allocation

In “Oversubscription for Reservation”, reserved but not allocated resources are offered as revocable  resources. The following options focus on non-throttle revocable resources. The throttleable revocable resources are managed by ResourceEstimator/QoSController. (Add more description here on throttleable)

```
message Resource {
  ...
    message RevocableInfo {
      message ThrottleInfo {}

      // If set, indicates that the resources may be throttled at
      // any time. Throttle-able resources can be used for tasks
      // that do not have strict performance requirements and are
      // capable of handling being throttled.
      optional ThrottleInfo throttle_info = 1;
    }

  // If this is set, the resources are revocable, i.e., any tasks or
  // executors launched using these resources could be terminated at
  // any time. This could be used by frameworks to run
  // best effort tasks that do not need strict uptime
  optional RevocableInfo revocable = 9;
}
```

Option 1: Manage revocable resources in a separate sorter (the value of revocable resources is different with regular resources)
In this option, the value of revocable is considered to be different with regular resources. A new resources pool (revocableRoleSorter) is introduced to manage/allocate revocable resources. The revocable resources pool will come after DRF as stage 3 in allocator; it uses the same logic with DRF to handle revocable resources. Here’s the logic

* Stage 2 (DRF), only allocate non-revocable resources
* The revocableRoleSorter.total include both usage_slack and allocation_slack
* The total revocable resources is calculated in ::allocate(); because the idle reserved resources maybe changed after stage 2 (wDRF)
* Add a new counter for revocable resources; it need to trace allocation in stage 3 (totalRevocable and allocatedRevocable)

In “updateSlave()”, it need to update total revocable resources for oversubscription.

… add a counter for oversubscription or still use slave.total ????

In “allocate()”, add a new stage for revocable resources as follow:

```
// Calculate the total revocable resources in allocator. The idle reserved resources maybe 
// changed in stage 2 (wDRF).
foreach slaves {
  revocableRoleSorter->update(slaveId, idle reserved + totalOversubscription);
  slave[slaveId].totalRevocable = idle reserved + totalOversubscription;
}

// Allocate revocable resources based on sorters.
foreach slaves {
  foreach revocableRoleSorter->sort() {
    foreach revocableFrameworksSorter[role]->sort() {
      available = totalRevocable - allocatedRevocable;

      if (isFilter() || !allocatable(available))
        continue;

      slave[slaveId].allocatedRevocable += available;
      offerable[slaveId][frameworkId] += available;

      revocableRoleSorter->allocated(...);

      revocableFrameworkSorter->add(...);
      revocableFrameworkSorter->allocated(...);
    }
  }
}
```

In “revocableResources”, the allocation of “revocableRoleSorter” is also updated to release resources.

```
revocableFrameworkSorters[role]->unallocated(...);
revocableFrameworkSorters[role]->remove(...);
revocableRoleSorter->unallocated(...);
```

Option 2: Manage revocable resources together with regular resources in role/framework sorter (the revocable resources means the same value as regular resources)
Refer to the allocator part  above.

Prefer to #1; allocator handles revocable resources separately, “Revocable by default” JIRA will only update the logic/code on revocable resources stage. But if introduced stage 3 for revocable resources, the performance of allocator may be impacted. It need a benchmark for this option.

## Rescind Offer

After offering idle reserved resources as revocable resources to the framework, those idle reserved resources may be allocated to its owner; there’s two options for us to handle this case:
Option 1: Rescind revocable offers in allocator
For throttleable revocable resources, the offers are rescinded in “Master::updateSlave()”; and the total throttleable revocable resources are also updated in allocator.
If the total non-throttleable revocable resources is less than allocated non-throttleable revocable resources in allocator, the allocator calls “rescindOfferCallback” to ask master to rescind offers; and the allocator reduce the allocated non-throttleable revocable resource to total non-throttleable revocable resources. That’ll introduce different view on resources for master and allocator (refer to “Master Endpoint Implementation Challenges” [1] for more detail), it will be corrected when launching next tasks; refer to the following section on eviction in agent.

To enable rescind offer in allocator, introduce a new callback for allocator interfaces as follow:

```
class Allocator {
  ...
    void initialize(
        const Duration& allocationInterval,
        const lambda::function<
        void(const FrameworkID&,
          const hashmap<SlaveID, Resources>&)>& offerCallback,
        const lambda::function<
        void(const FrameworkID&,
          const hashmap<SlaveID, UnavailableResources>&)>&
        inverseOfferCallback,

        // The callback for allocator to rescind resources.
        const lambda::function<
        void(const FrameworkID&,
          const hashmap<SlaveID, Resources>&)>& rescindOfferCallback,

        const hashmap<std::string, double>& weights);
  ...
};
```

After got “rescindOfferCallback” in master, there's some race condition between launching task & rescinding offer:

Case 1: MasterEventQueue: rescindOfferCallBack , launchTask

In “launchTask”, if the offer is invalid because of “rescindOfferCallback”, return “TASK_LOST” to framework.

Case 2: MasterEventQueue: launchTask, rescindOfferCallBack

Option 1: in launchTask(), let master wait for allocator’s Future to check available revocable resource; only the task asking for revocable is necessary, because the revocable resources may be invalid when launching tasks.

Option 2: in launchTask(), keep current behaviour: launch task if offer is valid. In “rescindOfferCallback”, try to rescind offer as much as possible. There’s the case that the revocable resources maybe overcommitted, it need agent to evict revocable resources accordingly (refer to the following section for the detail).

Prefer to #2 in this JIRA; #1 may introduce performance issues when launching tasks, it’s better to handle it in MESOS-4553 (Manage Offer in Allocator). The #2 take similar behaviour with oversubscription: rescind offer when “updateSlave()”,  evict executors by controller in agent if not enough resources. So we still need the agent to evict resources accordingly. The detail of eviction in agent will describe in coming sections. 

Option 2: Evict executors until launching tasks in agent

In this option, the allocator did not rescind offer when allocated resources and total resources mismatch (total non-throttleable revocable resources < allocated non-throttleable revocable resources). It account master/agent to evict executor when launching tasks. Refer to the “Eviction” section for the detail process.
Offers

To avoid over resind offers in “Rescind Offer” section, master will separate resources into three type offers: regular offer (unreserved & reserved resources), throttleable revocable offer, non-throttleable revocable offer.  (When rescind regular offers, do we rescind revocable resources firstly? Any case?)

## Eviction

To resolve the conflict, a new field “Revocation” is introduced in “RunTaskMessage” on the resources that should be evicted in framework/role. 

```
message RunTaskMessage {
  ...

    // Evict Resources to launch tasks.
    message Revocation {
      // Framework from which the revocable throttleable
      // resources will be revoked.
      optional FrameworkID framework_id = 1;

      required string role = 2;

      // Revocable throttleable resources that needs to be
      // revoked from agent.
      repeated Resource revocable_resources = 3;
    }

  repeated Revocation revocations = 5;

  ...
}  
```

Terminology for revocation:

* Total: total regular resources (unreserved and reserved resources)
* Occupied: the resources that occupied by normal tasks
* OccupiedRev: the resources that occupied by not-throttleable revocable tasks
* Evicting: the revocable resources that are evicted/evicting, but not return to master/allocator

The theoretical basis for options to do eviction is the following formula for non-throttleable revocable resources:

```
total >= occupied + occupiedRev + evicting
```

Option 1: Agent calculates how many resources to evict, chooses which executor to evict and then does eviction

Option 2: Master calculates how many resources to evict; Agent chooses which executor to evict and does eviction

In master, the revocation information will be got by following formula in one agent: 

```
revocation = total - occupied - occupiedRev + evicting
```

There’s several cases the may trigger different behaviour:

Case 1: Over evicting (1)

```
T1                                                                                       T1 + delta
---------------------------------------------------------- launchTask (normal) -------------------
 \
      ---------- terminating (revocable) -----------------------------------------------------
```

In master, 

Case 2: Over evicting (2)

```
T1                                                                                       T1 + delta
---- launchTask (normal) ------------------------ launchTask (normal) ----------------------------
 \
  ----------------------------------- evicting -----------------------------------------------
```

In master, 

```
revocation = total resources - running regular resources - running non-throttleable resources
```



Case 3: Unnecessary reject

```
T1                                                                                       T1 + delta
---- launchTask (normal) ------------------------ launchTask (revocable) -------------------------
 \
  ----------------------------------- evicting -----------------------------------------------
```


There’re several condition we need to consider:

* pendingTasks in Master::framework
* runningTasks in agent (evicting)
* launchingTasks in Master::launchTasks

There’s some target of this options:

* Avoid over evicting
* Avoid unnecessary reject to revocable tasks (`TASK_LOST`); race condition with evicting resources.

Race conditions:

* Launching revocable tasks with evicting executors.
* Launching revocable tasks with alloator’s rescindCallback; refer to the above cases
* Launching revocable tasks with dynamic reservation

Prefer to #1; because of race condition, the master can not give accurate revocation information to slave; the slave has to recalculate how many resources need to evict based on resources usage and task info.

