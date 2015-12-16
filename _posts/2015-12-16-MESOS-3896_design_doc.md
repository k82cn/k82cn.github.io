---
layout: post
title: MESOS-3896&#58; Add accounting for reservation slack in the allocator
categories: tech
---

## Description: 

The target of this JIRA is to offer the allocation slack resources to the framework.

## JIRAs of Design

### MESOS-XXX: Optimsistic accounter

    class HierarchicalAllocatorProcess 
    {
      struct Slave
      {
        ...
        struct Optimistic 
        {
          Resources total; // The total allocation slack resources
          Resources allocated; // The allocated allocation slack resources
        };
    
        Optimistic optimistic;
      };
    }

### MESOS-4146: flatten & allocationSlack for Optimistic Offer

    class Resources
    {
        // Returns a Resources object with the same amount of each resource
        // type as these Resources, but with all Resource objects marked as
        // the specified `RevocableInfo::Type`; the other attribute is not
        // affected.
        Resources flatten(Resource::RevocableInfo::Type type);

        // Return a Resources object that:
        //   - if role is given, the resources did not include role's reserved
        //     resources.
        //   - the resources's revocable type is `ALLOCATION_SLACK`
        //   - the role of resources is set to "*"
        Resources allocationSlack(Option<string> role = None());
    }

### MESOS-XXX: Allocate the allocation_slack resources to framework

    void HierarchicalAllocatorProcess::allocate(
        const hashset<SlaveID>& slaveIds_)
    {
      foreach slave; foreach role; foreach framework
      {
        Resource optimistic;

        if (framework.revocable) {
          Resources total = slaves[slaveId].optimistic.total.allocationSlack(role);
          optimistic = total - slaves[slaveId].optimistic.allocated;
        }

        ...
        offerable[frameworkId][slaveId] += resources + optimistic;

        ...
        slaves[slaveId].optimistic.allocated += optimistic;
      }
    }

  
Here's some consideration about `ALLOCATION_SLACK`:

1. 'Old' resources (available/total) did not include ALLOCATION_SLACK
2. After `Quota`, `remainingClusterResources.contains` should not check ALLOCATION_SLACK; if there no enough resources,  master can still offer ALLOCATION_SALCK resources.
3. In sorter, it'll not include ALLOCATION_SLACK; as those resources are borrowed from other role/framework
4. If either normal resources or ALLOCATION_SLACK resources are allocable/!filtered, it can be offered to framework
5. Currently, allocator will assign all ALLOCATION_SALCK resources in slave to one framework

### MESOS-XXX: Update ALLOCATION_SLACK for dynamic reservation (updateAllocation)

    void HierarchicalAllocatorProcess::updateAllocation(
        const FrameworkID& frameworkId,
        const SlaveID& slaveId,
        const vector<Offer::Operation>& operations)
    {
        ...
        Try<Resources> updatedOptimistic =
            slaves[slaveId].optimistic.total.apply(operations);
        CHECK_SOME(updatedTotal);

        slaves[slaveId].optimistic.total =
            updatedOptimistic.get().stateless().reserved().flatten(ALLOCATION_SLACK);
        ...
    }
    
### MESOS-XXX: Add ALLOCATION_SLACK when slaver register/re-register (addSlave)

    void HierarchicalAllocatorProcess::addSlave(
        const SlaveID& slaveId,
        const SlaveInfo& slaveInfo,
        const Option<Unavailability>& unavailability,
        const Resources& total,
        const hashmap<FrameworkID, Resources>& used)
    {
      ...
      slaves[slaveId].optimistic.total =
          total.stateless().reserved().flatten(ALLOCATION_SLACK);
      ...
    }
  
No need to handle `removeSlave`, it'll all related info from `slaves` including `optimistic`.

### MESOS-XXX: return resources to allocator (recoverResources)

    void HierarchicalAllocatorProcess::recoverResources(
        const FrameworkID& frameworkId,
        const SlaveID& slaveId,
        const Resources& resources,
        const Option<Filters>& filters)
    {
      if (slaves.contains(slaveId))
      {
        ...
        slaves[slaveId].optimistic.allocated -= resources.allocationSlack();
        ...
      }
    }

