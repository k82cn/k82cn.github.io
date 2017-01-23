---
layout: page
title: About
permalink: /about/
---

IBM Software Architect, Mesos/Kubernetes Contributor. Jilin University master's degree, majoring in grid computing and distributed system. After graduation, he joined Baidu, then IBM; focus on resource management, resource scheduling in distributed system.

## Experience

### IBM, Beijing, China (2015 ~ now)

__Open Source Developer of Spectrum Conductor for Container__

Kubernetes contributor: 45 patches in 2 Months; one major proposal:
  * Run multiple application in k8s [#36716] (proposing to be incubator, prototype at k82cn/kube-arbitrator)

Kube-mesos maintainer: several proposals:
  * Integrate Namespace/Quota with CloudProviders [#31069]
  * Integrate kube-DNS with external DNS [#28453]
  * Make k8sm-scheduler using reserved resource firstly [#31068]
  * Kubernetes use revocable resources from Mesos [#19529]
  * Refactor k8sm to avoid re-ship binaries [k8sm#20] (on-going)

Mesos contributor: 27 patches; several major proposals:
  * Reusable/Cacheable Offer [MESOS-4811]
  * Oversubscription for reservation [MESOS-4967] (with prototype)
  * Manage offers in allocator [MESOS-4553]
  * Enable `requestResource` for Marathon/Kubernetes

### IBM, Beijing, China (2014 ~ 2015)

__Team Lead of Spectrum Symphony L3__

Lead ~5 size of team on critical customer issues handling, by working with global Product manager and Support team, to meet business priorities in time

### IBM, Beijing, China (2010 ~ 2014)

__Team Lead of Spectrum Symphony CE__

Lead ~10 size of team on agile development for new requirements, by working with global Product manager, to meet business priorities in time; the major requirements includes:
  * SSM migration
  * DAG workload for Symphony (research)
  * Single SIM for Symphony (investigation)
  * Multiple Tasks in One Service Instance (MTS)
  * Service Affinity Group (SAG)
  * Recursive Workload
  * 20K cores per application (Multiple-SSM)

### Baidu, Beijing, China (2008 ~ 2010)

R&D of PS department; focus on coverage of spider/crawler.

R&D of IT department; focus on “workload form engine”, “user management system” and so on.


## Contacts
__E-mail__: [klaus1982.cn@gmail.com](mailto:klaus1982.cn@gmail.com); __Github__: [@k82cn](http://www.github.com/k82cn); __Linkedin__: [k82cn](http://cn.linkedin.com/in/k82cn)

## Education

* Jilin University Master, Computer Science and technology 2005 – 2008

Computer System Architecture Lib, major in Grid Computing (Globus, MPICH-G2); focus on “Virtual Job Model”, 3 papers published, 1 conference

* JiLin University BS, Computer Science and technology 2001 – 2005

Major in Computer Science and Technology; Scholarship every year

## Presentation

  * KubeCon 2016: [Kubernetes on EGO -- Bringing Enterprise Resource Management and Scheduling to Kubernetes](http://sched.co/8K3n)
  * MesosCon 2016 Asia: [Kubernetes on Mesos: Not Just Another Mesos Framework](http://sched.co/8QFB)
  * GOPS 2016 Beijing: [Resource Management and Scheduler of PaaS](http://gops2016-beijing.eventdove.com/)
  * COSC 2016: [Kubernetes on Mesos: The Present and the Future](http://www.huodongxing.com/go/coscon2016)
  * CSDN 2016: [Round table on micro service in enterprise](http://cctc.csdn.net/m/zone/cctc2016/schedule)

## Publication

### [__Customized Plug-in Modules in Metascheduler CSF4 for Life Sciences Applications__](http://link.springer.com/article/10.1007/s00354-007-0024-6?no-access=true)

New Generation Computing March 9, 2008

__Authors__: Zhaohui Ding, Xiaohui Wei, Yuan Luo, Da Ma, Peter Arzberger, Wilfred Li

As more and more life science researchers start to take advantages of grid technologies in their work, the demand increases for a robust yet easy to use metascheduler or resource broker. In this paper, we have extended the metascheduler CSF4 by providing a Virtual Job Model (VJM) to synchronize the resource coallocation for cross-domain parallel jobs. The VJM eliminates dead-locks and improves resource usage for multi-cluster parallel applications compiled with MPICH-G2. Taking advantage of the extensible scheduler plug-in model of CSF4, one may develop customized metascheduling policies for life sciences applications.  As an example, an array-job scheduler plug-in is developed for pleasantly parallel applications such as AutoDock and Blast. The performance of the VJM is evaluated through experiments with mpiBLAST-g2 using a Gfarm data grid testbed. Furthermore, a CSF4 portlet has been released to provide a graphical user interface for transparent grid access, with the use of Gfarm for data staging and simplified data management.  The platform is open source at sourceforge.net/projects/gcsf/ and has been deployed in life science gateways by projects such as My WorkSphere, and PRAGMA Biosciences Portal. The VJM enables the development of support for more sophisticated workflows and metascheduling policies in the near future.

### [__A Virtual Job Model to Support Cross-Domain Synchronized Resource Allocation__](http://www.cs.indiana.edu/~yuanluo/publications/VJM.pdf)

Journal of Software

__Authors__: Zhaohui Ding, Xiaohui Wei, Da Ma

Although more and more scientists start to take advantages of grid technologies to facilitate their researches, running parallel jobs crossing domains in a grid environment is still a challenge. Even MPICH-G2 is able to run MPI applications on across domain resources, however, the resource allocations are not synchronized which will cause dead lock and other serious problems. In this paper, we introduced a virtual job model (VJM) which achieves synchronized cross-domain resource allocation for parallel grid applications. VJM is able to prevent the resource allocation deadlock caused by multiple parallel jobs competing resource, and alleviate the resource waste by backfilling small jobs. VJM can work with almost all kinds of local schedulers via standard Grid Resource Allocation and Management (GRAM) protocol as it does not depend on resource reservation. We have implemented VJM in meta-scheduler CSF4 and validate the rationality of VJM by mpiBLAST-g2, a parallel bioinformatics application.

## Skills

Kubernetes, Mesos, HPC, PMP (#1684623), C/C++, Java, Golang

## Title

* __Software Architect__: Platform Computing, an IBM company (2016/07 ~ Present) at Beijing
* __Advisory Software Engineer__: Platform Computing, an IBM company (2014/07 ~ 2016/07) at Beijing
* __Team Lead of L3__: Platform Computing, an IBM company (2014/07 ~ 2015/07) at Beijing
* __Team Lead of CE__: Platform Computing, an IBM company (2011/11 ~ 2014/06) at Beijing
* __Software Engineer__: Platform Computing (2010/09 ~ 2011/11) at Beijing
* __Software Engineer__: Baidu (2008/06 ~ 2010/08) at Beijing

## Others

![Euler](https://projecteuler.net/profile/k82cn.png)

