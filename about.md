---
layout: post
title: Klaus Ma
permalink: /about/
---

Team leader, system architect, designer, software developer with 10+ years of experience across a variety of industries and technology bases, including cloud computing, machine learning, bigdata and financial services. 

Founding Volcano & kube-batch, Kubernetes SIG-Scheduling co-Leader, CNCF Research User Group & SIG-Runtime Tech Lead. Global Team Lead of IBM Spectrum Symphony CE & L3. Currently, Architect, R&D, Nvidia Technologies.

## Experience

### Nvidia, Beijing, China (2022 ~ now)

Engineer & Architect of DPU Cloud Management

### Huawei, Beijing, China (2018 ~ 2022)

Founding [Volcano](http://github.com/volcano-sh/volcano) & [kube-batch](https://github.com/kubernetes-sigs/kube-batch); architect of Batch Container Service of Huawei Cloud, lead ~10 size of team to build cloud service for batch workload, including AI, BigData, Gene and so on.

### IBM, Beijing, China (2015 ~ 2018)

__Open Source Developer of Spectrum Conductor__

Kubernetes:

* [Kubernetes Maintainer]( https://groups.google.com/forum/#!topic/kubernetes-membership/EkxXeeLuV_w )
* [Kubernetes SIG-Scheduling co-leader]( https://groups.google.com/forum/#!msg/kubernetes-sig-scheduling/xS4RYbtUItE/3Wn0RwnvAAAJ )
* [kube-arbitrator creator]( https://groups.google.com/forum/#!msg/kubernetes-dev/K3Jn3SjBPSc/7gZDCGluBQAJ )
* kube-mesos maintainer

Mesos Contributor, several patches and major proposals:

* Reusable/Cacheable Offer [MESOS-4811]
* Oversubscription for reservation [MESOS-4967] (with prototype)
* Manage offers in allocator [MESOS-4553]
* Enable `requestResource` for Marathon/Kubernetes

### IBM, Beijing, China (2014 ~ 2015)

__Team Lead of Spectrum Symphony L3__

Lead ~5 size of team on critical customer issues handling, by working with global Product manager and Support team, to meet business priorities in time

### IBM, Beijing, China (2010 ~ 2014)

__Team Lead of Spectrum Symphony CE__

Lead ~10 size of team on agile development for new requirements, by working with global Product manager, to meet business priorities in time.

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

* KubeCon 2019 NA: [Improving Performance of Deep Learning Workloads With Volcano](https://sched.co/UaZi)
* KubeCon 2019 NA: [Batch Capability of Kubernetes Intro](https://sched.co/Uajv)
* COSCon'19: [华为云原生之路：开源加速企业开放式创新](https://bbs.huaweicloud.com/blogs/127619)
* ArchSummit 2019: [Volcano 在 Kubernetes 中运行高性能作业实践](https://archsummit.infoq.cn/2019/shenzhen/presentation/1817)
* Huawei Connection 2019: [Volcano：基于云原生的高密计算解决方案](https://agenda.events.huawei.com/2019/cn/minisite/agenda.html#dayTab=day7&tagName={"language"%3A"Cn"}&seminarId=1743)
* KubeCon 2019 China: [Volcano: Running AI/DL workload on Kubernetes ](https://sched.co/QXj2)
* KubeCon 2019 China: [Intro + Deep Dive: SIG Scheduling](https://sched.co/O9pG)
* KubeCon 2019 EU: [Intro: Kubernetes Batch Scheduling](https://sched.co/MPi7)
* KubeCon 2018 NA: [Intro: Scheduling SIG](https://sched.co/HDr0)
* KubeCon 2018 China: [Kube-Arbitrator: A Batch System of Kubernetes](https://sched.co/FuKC)
* KubeCon 2018 China: [Intro: SIG Scheduling](https://sched.co/FuLN)
* KubeCon 2018 China: [Deep Dive: Kubernetes Policy WG ](https://sched.co/FuLb)
* KubeCon 2018 China: [Deep Dive: SIG Scheduling](https://sched.co/FuLj)
* KubeCon 2016 NA: [Kubernetes on EGO -- Bringing Enterprise Resource Management and Scheduling to Kubernetes](http://sched.co/8K3n)
* MesosCon 2016 Asia: [Kubernetes on Mesos: Not Just Another Mesos Framework](http://sched.co/8QFB)
* GOPS 2016 Beijing: [Resource Management and Scheduler of PaaS](http://gops2016-beijing.eventdove.com/)
* COSC 2016: [Kubernetes on Mesos: The Present and the Future](http://www.huodongxing.com/go/coscon2016)
* CSDN 2016: [Round table on micro service in enterprise](http://cctc.csdn.net/m/zone/cctc2016/schedule)

## Publication

### [__Customized Plug-in Modules in Metascheduler CSF4 for Life Sciences Applications__](http://link.springer.com/article/10.1007/s00354-007-0024-6?no-access=true)

New Generation Computing March 9, 2008

__Authors__: Zhaohui Ding, Xiaohui Wei, Yuan Luo, Da Ma, Peter Arzberger, Wilfred Li

As more and more life science researchers start to take advantages of grid technologies in their work, the demand increases for a robust yet easy to use metascheduler or resource broker. In this paper, we have extended the metascheduler CSF4 by providing a Virtual Job Model (VJM) to synchronize the resource coallocation for cross-domain parallel jobs. The VJM eliminates dead-locks and improves resource usage for multi-cluster parallel applications compiled with MPICH-G2. Taking advantage of the extensible scheduler plug-in model of CSF4, one may develop customized metascheduling policies for life sciences applications.  As an example, an array-job scheduler plug-in is developed for pleasantly parallel applications such as AutoDock and Blast. The performance of the VJM is evaluated through experiments with mpiBLAST-g2 using a Gfarm data grid testbed. Furthermore, a CSF4 portlet has been released to provide a graphical user interface for transparent grid access, with the use of Gfarm for data staging and simplified data management.  The platform is open source at sourceforge.net/projects/gcsf/ and has been deployed in life science gateways by projects such as My WorkSphere, and PRAGMA Biosciences Portal. The VJM enables the development of support for more sophisticated workflows and metascheduling policies in the near future.

### [__A Virtual Job Model to Support Cross-Domain Synchronized Resource Allocation__](https://pdfs.semanticscholar.org/e992/2ac732d62b3b007414a39d7070530f571cd1.pdf)

Journal of Software

__Authors__: Zhaohui Ding, Xiaohui Wei, Da Ma

Although more and more scientists start to take advantages of grid technologies to facilitate their researches, running parallel jobs crossing domains in a grid environment is still a challenge. Even MPICH-G2 is able to run MPI applications on across domain resources, however, the resource allocations are not synchronized which will cause dead lock and other serious problems. In this paper, we introduced a virtual job model (VJM) which achieves synchronized cross-domain resource allocation for parallel grid applications. VJM is able to prevent the resource allocation deadlock caused by multiple parallel jobs competing resource, and alleviate the resource waste by backfilling small jobs. VJM can work with almost all kinds of local schedulers via standard Grid Resource Allocation and Management (GRAM) protocol as it does not depend on resource reservation. We have implemented VJM in meta-scheduler CSF4 and validate the rationality of VJM by mpiBLAST-g2, a parallel bioinformatics application.

## Skills

Kubernetes, Mesos, HPC, PMP (#1684623), C/C++, Java, Golang

