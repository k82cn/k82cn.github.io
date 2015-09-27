---
layout: default
title: About
---

## Da (Klaus) Ma
Detail-oriented software professional with 6+ years of experience on BigData & Distributed System, e.g. Hadoop, Spark, Mesos. Skilled at complex project design and escalation handling. Good written and oral communication skills; capable of leading team for projects.

## Skills

Distributed System, BigData, PMP (#1684623), C/C++, Java

## Experience

* __Advisory Software Engineer__: Platform Computing, an IBM company (2014/07 ~ Present) at Beijing
* __Team Lead of CE__: Platform Computing, an IBM company (2011/11 ~ 2014/06) at Beijing
* __Software Engineer__: Platform Computing (2010/09 ~ 2011/11) at Beijing
* __Software Engineer__: Baidu (2008/06 ~ 2010/08) at Beijing

## Contacts
__E-mail__: [klaus@cguru.net](mailto:klaus@cguru.net); __Skype__: klausma1982; __Github__: [@klaus1982](http://www.github.com/klaus1982)

## Education

* Jilin University Master, Computer Science and technology 2005 – 2008
* JiLin University BS, Computer Science and technology 2001 – 2005

## Projects

* __Multiple Tasks in One Service Instance (MTS)__

  In IBM Platform Symphony, it's used to be one process for each running task in compute host; it's unconvinced for user to share data & computing result. With MTS feature, there is only one service instance/ process for all running tasks from the same job or application. As the owner of the project, worked on drafting FS, coding and driving the release. 

  __Tools__: C/C++, GDB, gtest, ACE

* __Service Affinity Group (SAG)__

  The target of this project is to avoid the overhead when slots transfer between burst jobs, by allowing different jobs in the same SAG will run on the same service instance where the data has previously been loaded. As the owner of the project, worked on drafting FS, coding and driving the release. 
  
  __Tools__: C/C++, Java, GDB, gtest, ACE

* __Recursive Workload__

  Customer's workload pattern is recursive, where a task (parent task) will create another job (child job) and generate more tasks (child tasks) that are submitted to the child job. After post-submission processing, parent task will wait for its child tasks to complete. While it waits, the compute slot is wasted. To increase resource usage, this feature allows a running task to yield the slot(s), which can be used to run the tasks of its direct descendants. As local peer, worked with Toronto team on coding, bug fix and driving release.
  
  __Tools__: C/C++, Java, GDB, gtest, ACE

* __R/Python integration__

  R & Python are two major integration projects. IBM Platform symphony provides lots of API for user to submit/create task to the cluster system. The purpose of integration project is to provide a native API in those programming language. As a project member, worked on researching, coding & bug fix.
  
  __Tools__: C/C++, R/ Python, GDB

* __20K cores per application (Multiple-SSM)__

  The multi-ssm feature provides better scalability by employing multiple physical applications to do the real work. Jobs created in logical application are logical jobs, which internally be mapped to a physical job of one physical application, or physical jobs of multiple physical applications. Tasks of a logical job will be redirected to the related physical job(s). Take part in the project as a freshman, worked on coding and unit testing; and help QA on system test.
  
  __Tools__: C/C++, R/Python, GDB

* __Workflow form engine__

  In workflow, forms engine is another key module to submit, process and display data. The purpose of this project is to provide a WYSIWYG system for administrator to manage workflow forms, a data processing system to handle user's input and a display system to generate form according to workflow's status and data. As the owner of project, worked on researching, design and coding.
  
  __Tools__: JEE, MySQL, Shark (workflow), JavaScript/Html/Css

## Publication

* __Customized Plug-in Modules in Metascheduler CSF4 for Life Sciences Applications__:

  New Generation Computing March 9, 2008

  Authors: Da Ma, Zhaohui Ding, Xiaohui Wei, Yuan Luo, Peter Arzberger, Wilfred Li

  As more and more life science researchers start to take advantages of grid technologies in their work, the demand increases for a robust yet easy to use metascheduler or resource broker. In this paper, we have extended the metascheduler CSF4 by providing a Virtual Job Model (VJM) to synchronize the resource coallocation for cross-domain parallel jobs. The VJM eliminates dead-locks and improves resource usage for multi-cluster parallel applications compiled with MPICH-G2. Taking advantage of the extensible scheduler plug-in model of CSF4, one may develop customized metascheduling policies for life sciences applications.  As an example, an array-job scheduler plug-in is developed for pleasantly parallel applications such as AutoDock and Blast. The performance of the VJM is evaluated through experiments with mpiBLAST-g2 using a Gfarm data grid testbed. Furthermore, a CSF4 portlet has been released to provide a graphical user interface for transparent grid access, with the use of Gfarm for data staging and simplified data management.  The platform is open source at sourceforge.net/projects/gcsf/ and has been deployed in life science gateways by projects such as My WorkSphere, and PRAGMA Biosciences Portal. The VJM enables the development of support for more sophisticated workflows and metascheduling policies in the near future.

* __A Virtual Job Model to Support Cross-Domain Synchronized Resource Allocation__:

  Journal of Software

  Authors: Da Ma, Zhaohui Ding, Xiaohui Wei

  Although more and more scientists start to take advantages of grid technologies to facilitate their researches, running parallel jobs crossing domains in a grid environment is still a challenge. Even MPICH-G2 is able to run MPI applications on across domain resources, however, the resource allocations are not synchronized which will cause dead lock and other serious problems. In this paper, we introduced a virtual job model (VJM) which achieves synchronized cross-domain resource allocation for parallel grid applications. VJM is able to prevent the resource allocation deadlock caused by multiple parallel jobs competing resource, and alleviate the resource waste by backfilling small jobs. VJM can work with almost all kinds of local schedulers via standard Grid Resource Allocation and Management (GRAM) protocol as it does not depend on resource reservation. We have implemented VJM in meta-scheduler CSF4 and validate the rationality of VJM by mpiBLAST-g2, a parallel bioinformatics application. 


