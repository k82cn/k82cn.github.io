---
layout: post
categories: tech
title: Monitoring hanging tasks in large cluster
---

In a large cluster, some un-expected long running task hold back the job’s runtime sometimes. And those long running tasks are hard to locate in such a large cluster. Some ideas are shown here to help to locate those long running tasks. One of them will be implemented in the coming released of OGL.

## Customer complain:

* DEV cluster: Customer want to check the issue SI to debug
* Product cluster: customer complain the long running tasks
    * the job can not complete because of long running tasks
    * all other tasks are useless because some tasks (the long running tasks) are un-finished (killed)
        * when to check the long running task? If check the task when the job is almost closed, no much value because:
        * almost all tasks are done; the resource are waste
        * customer can check it by oglview

## Condition of the long running task:

* Task number
    * c1.1: one task
    * c1.2: multiple tasks
* Task’s order of the job
    * c2.1: first task of the job
    * c2.2: last task of the job
    * c2.3: in the middle of the task queue of the job
* Average of task run time
    * c3.1: Variance(runing time) is large
    * c3.2: Variance(runing time) ~ 0
* The root cause of hang
    * c4.1: environment issue
    * c4.2: logic/data issue

## Options:

* Configure the timeout of the task; if one task did not finish within the duration, restart is and send SNMP notification
* Configure the timeout of the task; if one task did not finish within the duration, only send SNMP notification; DEV will attach to the process in the compute host, oglctrl task jobId:taskId restart
* Provide a new callback “onCheckStatus”; JobRunner will ping service to get its status, because only service known the status
    * StatusContext: isTimeOut
    * But how to deal with onCheckStatus timeout, SNMP?
* Check the throughoutput of JobRunnerObject per min (or second?), report the JobRunner which has the lowest thoughoutput in a Job.
    * set the task count to zero when JobRunnerObject bind to a Job;
    * including success & failed tasks
