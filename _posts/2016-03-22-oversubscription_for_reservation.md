---
layout: post
title: Mesos&#58; Oversubscription for Reservation
categories: tech
---

## Background

Resources can be reserved by frameworks in a variety of ways, including:

* Static reservations
* Dynamic reservations via Offer::Operations
* Dynamic reservations via HTTP endpoints
* Quotas (None-Goal)

Reserved resources allow frameworks and cluster operators to ensure sufficient resources are available when needed.  Reservations are usually made to guarantee there are enough resources under peak loads. Often times, reserved resources are not actually allocated; in other words, the frameworks do not use those resources and they sit reserved, but idle.

This underutilization is either an opportunity cost or a direct cost, particularly to the cluster operator.  Reserved but unallocated resources held by a Lender Framework could be optimistically offered to other frameworks, which we refer to as Tenant Frameworks.  When the resources are requested back by the Lender Framework, some of the Tenant Framework-s tasks are evicted and the original resource offer guarantee is preserved.

The first step is to identify when resources are reserved, but not allocated.  We then offer these reserved resources to other frameworks, but mark these offered resources as revocable resources.  This allows Tenant Frameworks to use these resources temporarily in a 'best-effort' fashion, knowing that they could be revoked or reclaimed at any time.

## Allocation

### Option 1: Manage revocable resources in a separate sorter (the value of revocable resources is different with regular resources)

In this option, the value of revocable is considered to be different with regular resources. A new resources pool (revocableRoleSorter) is introduced to manage/allocate revocable resources. The revocable resources pool will come after DRF as stage 3 in allocator; it uses the same logic with DRF to handle revocable resources. Here's the logic 

1. Stage 2 (DRF), only allocate non-revocable resources
1. The revocableRoleSorter.total include both usage_slack and allocation_slack
1. The total revocable resources is calculated in `::allocate()`; because the idle reserved resources maybe changed after stage 2 (wDRF).
1. Add counter for revocable resources; it need to trace allocation in stage 3


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
      offerable[frameworkId] += available;

      revocableRoleSorter->allocated(...);

      revocableFrameworkSorter->add(...);
      revocableFrameworkSorter->allocated(...);
    }
  }
}
```

In `revocableResources`, the allocation of revocableRoleSorter is also updated to release resources.


```
revocableFrameworkSorters[role]->unallocated(...);
revocableFrameworkSorters[role]->remove(...);
revocableRoleSorter->unallocated(...);
```

### Option 2: Manage revocable resources together with regular resources in role/framework sorter (the revocable resources means the same value as regular resources)

Refer to the allocator part in [Optimistic Offer Phase 1](https://docs.google.com/document/d/1RGrkDNnfyjpOQVxk_kUFJCalNMqnFlzaMRww7j7HSKU/edit)

Prefer to #1; allocator handles revocable resources separately, "Revocable by default" JIRA will only update the logic/code on revocable resources stage. But if introduced stage 3 for revocable resources, the performance of allocator maybe impacted. It need a benchmark for this option.


## Rescind Offer

After offering idle reserved resources as revocable resources to the framework, those idle reserved resources maybe allocated to its owner; there's two options for us to handle this case:

### Option 1: Rescind revocable offers in allocator

If the total revocable resources is less than allocated revocable resources in allocator, the allocator calls __rescindOfferCallback__ to ask master to rescind offers. A new counter is introduced in allocator to trace rescinding resources, it is upated when master recovery resources.

In master, there's some race condition between launching task & rescinding offer:

Case 1: 
    --------- rescindOfferCallBack --------------- launchTask ---------------


Case 2:
    --------- launchTask --------------- rescindOfferCallBack ---------------

Option 1: 


- how about the used resources?

1. identify the resources that should be rescinded
2. send resources to master; when?
3. in maser, try to rescind offers for resources; but how to handle race condition?
    1. call resolveConflict before launching tasks??
    1. ignore race condition; let agent correct it??



Update allocator interfaces for rescind offer as follow:

```
class Allocator {
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
};
```



### Option 2: Evict executors until launching tasks in agent



## Eviction

To resolve the conflict , there are several options to us:

### Option 1: Agent calculates how many resources to evict, chooses which executor to evict and then does eviction


### Option 2: Master calculates how many resources to evict; Agent chooses which executor to evict and does eviction


### Option 3: Master calculates how many resources to evict and chooses which executor to evict, Agent does eviction



## Feature Interaction

### Dynamic Reservation

### Maintainenance

### Quota
