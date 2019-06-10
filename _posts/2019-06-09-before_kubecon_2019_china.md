---
layout: post
categories: tech
title: 写在KubeCon之前
---

本想在欧洲的KubeCon之前总结一下近期的工作，一直拖到现在；还好中国KubeCon六月底又要开始了，不算太晚。算算这是从2016年开始，参加的第6个KubeCon了；收获与成长都很多，也认识很多朋友。文笔较差，所以就用流水账的形式记录一下；感想建议什么的，后面单独找时间介绍吧。

## 2016-新起点

2016年的时候IBM Spectrum (前[Platform Computing](https://baike.baidu.com/item/platform/1486257))仍然在重点投入Mesos，并以Kubernetes on Mesos为主体推一款PaaS平台 (今天ICP的前身)。社区方面，开始接手Kubernetes on Mesos项目；该项目最初由Mesosphere在Kubernetes主社区维护，后来由于不再投入，转由IBM进行维护。IBM Spectrum一直比较推崇两级调度，主要是由于内部有一个相似的系统 (EGO)。EGO大概在2005年GA，提供了大量的资源管理及调度功能；但是作为一款商用软件，一直没能开源出来。关于两级调度的优劣及对Mesos社区的感想，后面再单独找时间详细介绍；一些早期的想法可以参考之前的[讨论](https://www.kubernetes.org.cn/601.html)。

接手Kubernetes on Mesos后便开始了基本的推广和维护工作，也由此开始了Kubernetes社区的工作。随着在Kubernetes工作的深入，逐渐将所有精力投入到kubernetes中；在Kubernetes on Mesos中仅剩一些宣传的工作；随后产品也转为以Kubernetes为基础的PaaS产品。Kubernetes on Mesos之后便不再参与这种跨多个社区的项目，主要是由于每个社区都有自己的目标、规范以及解决问题的习惯；重叠和互补部分很难达成一致。

在16年年底的时候中了第一个KubeCon的topic，是关于[Kuberentes on EGO](https://cnkc16.sched.com/event/8K3n/kubernetes-on-ego-bringing-enterprise-resource-management-and-scheduling-to-kubernetes-da-ma-ibm)的 (与 Kubernetes on Mesos类似)，介绍了一些EGO的基本功能。那届KubeCon在北美，只有大概 2000 人左右的样子；github的头像也是那时候CoreOS的活动画的。除了第一次与社区的各个贡献者面对面交流，更重要的一点是开始了Kubernetes中Batch作业的相关工作；在做了简单的线下交流后，在主社区open了 [Manage multiple applications in Kubernetes](https://github.com/kubernetes/kubernetes/issues/36716) ，并开始了差不多半年的讨论。

## 2017-在路上

在Kubernetes上游社区open了  [Manage multiple applications in Kubernetes](https://github.com/kubernetes/kubernetes/issues/36716) 之后，开始只是一些简单的讨论，后来由 David 起草了一个设计文档： [Resource sharing architecture for batch and serving workloads in Kubernetes](https://docs.google.com/document/d/1-H2hnZap7gQivcSU-9j4ZrJ8wE_WwcfOkTeAGjzUyLA/edit#heading=h.a1k69dgabg0w) 。该设计文档主要包含了两个部分：一部分是最近GA的priority/preemption，另一部分则是如何支持batch workload。Priority/Preemption相对来说比较直接，总体设计很快便达成一致，并开始了相应的开发工作；但 batch workload部分却有很多细节的讨论。在讨论的过程中，发现自己对k8s本身的设计及社区的运营方式有很大的欠缺；因此加大了在kubernetes上游社区贡献，并在同年8月份成功申请Maintainer。

但是关于Batch workload的讨论一直都没有停止，主要是由于Kubernetes在最开始的设计上面以Service workload为主，基本上没有关于Batch workoad的概念及功能。在大概半年之后，大家还是决定创建一个孵化项目 ([kube-arbitrator](https://groups.google.com/forum/#!search/kube-arbitrator/kubernetes-dev/K3Jn3SjBPSc/dgUvmcavBAAJ))来继续相应的工作。kube-arbitrator最开始由几个部分组成：1.) quotad：对Quota相关的加强，包括动态分配，资源租借及回收等；2.) batchd：对scheduler的加强，相比于default scheduler，主要支持针对作业级别的调度，包括作业之间的资源共享等；3.) QueueJob：另一部分就是对现在有Job的加强，可以启动一个完成的 AI 作业。由于人力的限制，一直花费了较多的精力在batchd上；quotad一直处于停滞的状态；而QueueJob一直有[IBM FfDL](https://github.com/IBM/FfDL)维护。

初期的batchd仅包含一层接口，包括资源的分配和回收等；在维护的时候发现很多功能会杂糅在一起，导致新功能很难加入，已有功能很难维护等。在年底时便将整个batchd代码重写，分为action (allocate, preempt等)和plugin (gang, fair-share等)两层，方便新功能的加入和现有功能的维护。

## 2018-变化中

2018年还在继续着kube-arbitrator的工作；并基于当时的scope，逐渐希望把它做成一个Kubernetes的Batch System。相比于现在的 batch scheduler，batch system涵盖包含scheduler在内的其它部分，例如，作业管理，命令行，异构硬件，数据对接等；即 [kube-batch的架构图](https://github.com/kubernetes-sigs/kube-batch/blob/master/doc/images/kube-batch.png)中排除的部分。

虽然对Batch workload的需求越来越多，但是kube-arbitrator最初只有两个人维护，到18年初则变成我一个人在维护。 在年初的时候，上游社区开放了新的流程，允许各个SIG创建子项目 ([kubernetes-sigs](https://github.com/kubernetes-sigs))。随着kubernetes-sigs的升温，[kubernetes-incubator](https://github.com/kubernetes-incubator) 逐渐进入废弃的状态；因此申请了将 kube-arbitrator [由 kubernetes-incubator 转入 kubernetes-sigs](https://groups.google.com/forum/?utm_medium=email&utm_source=footer#!msg/kubernetes-sig-scheduling/ki3SOUGvAo4/0jZLuWGTBQAJ)，并修改名字为 kube-batch。由于 kubernetes-sigs 下的项目仅能属于一个sig，因此在改名的同时，也将 kube-arbitrator/kube-batch的scope 限定在 batch scheduler部分；而将之前关于quota, queuejob 以及 其它与 batch system 相关的功能都排除在 kube-batch 之外。由于scope的变化，将已经有代码迁移到一个个人仓库 (vulcan)，算是[Volcano](http://github.com/volcano-sh/volcano)的前身。

2018对kube-batch来说是很重要的一年。随着机器学习逐渐被大家关注并用于生产，以及kubeflow的流行，机器学习框架对kubernetes的batch能力的需求也逐渐强烈起来。其中一个最基本的功能就是 gang-scheduling (all-or-nothing)。由于default scheduler还不具备作业级的调度，因此gang-scheduling最终也实现在kube-batch里；并与kubeflow/tf-operator等各个上层社区完成了集成。

2018另外一个比较大的变化是自己最终在11月份入职了华为；有很多因素，感谢在这个过程中的同学们。

## 2019-在华为

加入华为差不多半年左右的时间，工作紧张有序，没有外界传言的那么夸张；节奏上跟之前我在[Platform Computing](https://baike.baidu.com/item/platform/1486257)的时候差不多。在这半年的时间里，最主要的工作就是把 [Volcano](http://github.com/volcano-sh/volcano) 作为 Kubernetes Native Batch System开源出来；在参与到Kubernetes社区之后，一直希望可以加强Kubernetes对batch workload的支持，从而使Kubernetes成为一个统一的基础平台；这样方便后续的工作，例如统一的资源调度及资源管理。

[Volcano](http://github.com/volcano-sh/volcano) 包含了 Batch System 需要的主要功能，例如作业管理，资源调度，资源管理，和对异构硬件的支持。相比传统的的Batch System，Volcano 还提供不同的使用方式：作业接口 或是 调度器 接口：对于现有的operator，可以直接使用调度器的接口，而对于新的batch workload则可以尝试作业接口来简化开发和维护成本。

2019也已经过去了一半，希望 Volcano 下半年有更多的使用者。
