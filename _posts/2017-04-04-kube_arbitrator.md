---
layout: post
tags: k8s
title: Kuberentes中基于策略的资源分配
categories: tech
---
**前言**：清明放假，抽空把在社区提的Proposal翻译一下。另外，我们产品的新版本(1.1) 节前也 release 了，有兴趣可以下载试用一下；有问题欢迎随时讨论 ([我是传送门](http://ibm.biz/ConductorForContainers)，戳我，戳我) 。

===================

原文 [传送门](https://github.com/kubernetes/community/pull/328/)，译文如下：


**用户场景和要求**：

作为集群管理员，希望建立一个环境来运行不同的工作负载，例如 “长服务”，大数据。 由于这些应用程序由不同的部门管理，我必须为每个应用程序提供资源保证，如下所示：

长服务（app area）和 大数据（bigdata area）可以共享资源：
* 定义每个区域的资源使用情况，例如 40％的资源到应用程序区域，60％到大数据区域。
* 借款/贷款协议：如果资源在一个区域空闲，可以借出并被抢占

在大数据区域运行多个群集：
* 定义bigdata区域内每个群集的资源使用情况，例如 Spark，Hadoop
* 在这些大数据集群之间共享资源，例如 借贷方式

以大数据为例，下面是一些要求细节：
* 运行一组应用程序
* 提供每个应用程序保证访问一定数量的资源
* 提供了所有应用程序尽力访问所有未使用的资源，根据某些目标权重（每个应用程序分配一个权重，即如果所有应用程序都想使用所有可用资源，则允许以相对比例的方式执行此操作）
* 如果一些应用程序A的使用少于其保证，然后如果它决定使用其保证，并且没有足够的可用资源来执行此操作，则应该能够从某些其他应用程序或应用程序（即/使用超过他们的保证）以获得其保证
此外，将“大数据”应用和“服务”应用分成两个桶，提供每个桶（总计）保证访问集群的一小部分，并尽可能地访问整个集群，同时了解上述保证的使用可以随时撤销。
根据这个 [mesos-style.md](https://github.com/kubernetes/kubernetes/blob/master/docs/devel/mesos-style.md) 文档，我们可以建立一个定制的组件来做到这一点；但由于这是资源管理和资源共享等一般性功能要求，最好由Kubernetes提供资源规划。

**术语**:

* arbitrator：根据策略分配资源（resource allocation）的新组件；kube-scheduler仍通过现有的策略(亲和等)为Pods指定使用的资源（resource assignment）
* deserved：arbitrator分配给namespace的资源总数
* overused：如果使用的资源超过deserved的资源，命名空间将被认为是overused的

**背景**:
随着Kubernetes的成长，目前可以通过以下几个功能实现资源的共享。

_Admission 和 ResourceQuota_：

在kubernetes中，namespace-level quota定义了命名空间可以使用的最大资源。通过ResourQuotaAdmission 实现，如果超过配额，则拒绝Pod创建。如果总配额大于集群的能力，则不会定义如何在namespace之间共享资源：如果一个namespace使用的资源少于其配额而但其它namespacey的资源请求超出了配额，则不能将这些资源借给其他资源（并在必要时抢占回来）。
Admission Controller是一个一般概念：Kubernetes可以根据任意策略拒绝Pod创建。例如，即使应用程序具有足够的配额来运行pod，如果应用程序在群集中使用的资源超过了比例，它也可以拒绝Pod创建。之前有过关于 “Job”（per-controller）进行Admission Control的讨论；这样就可以像批量任务系统那样进行处理。作为一般概念，可以建立定制的Admission Controller以拒绝Pod创建;但为了达到以下要求，引入了arbitrator：
* 需要根据namespace的请求（例如挂起的pod）计算每个namespace的deserved资源 （e.g. pending Pods）
* 如果namespace使用了超过deserved的资源 (overused)，需要在指定时间内将部分资源回收以供其它namespace使用
* 允许超出deserved/quota来创建“高优先级”pod，这样高优先级的pod可以抢占资源（“高优先级”可能是workload customized）

_抢占和重新安排_：

一个Pod可以被杀死然后重启，因为一些其他Pod需要它使用的资源（抢占）。有一个基于优先级的抢占方案（见Borge）的讨论（每个pod都有一个优先级，而具有更高和可能相同优先级的pod可以抢占它；但谁做出决定哪个pod要抢占还不确定：可以是默认的调度程序，rescheduler 和/或 具有调度功能的 customized controller）。抢占总是 graceful 的终止。优先权方案通常意味着一个优先级别基础上的配额分配，以便应用程序可以在最高优先级级别给予一定数量的配额，并且可以给予较低的优先级更大量的配额（甚至是无限的，即集群的整体容量）。并且通过resetuler驱逐pod来重新优化集群级别的策略（目前有一个执行策略的原始调度器: “关键的pod，如Heapster，DNS等不会由于集群中的空闲资源不足也无法运行；但有其他很多政策可以添加并执行）。它通过驱逐一个或多个pod来允许一些待处理的pod（s）进行调度。

抢占需要在namespace之间调度资源；arbitrator 可是定义“优先”的决策者之一。没有满足deserved的namespace的优先级高于overused的namespace。arbitrator将利用eviction功能进行抢占。rescheduler 将会继续为关键Pod抢占资源，也会重新调度Pods来达到更高的资源使用效率。有了arbitrator，kube-system具有无限的deserved资源：deserved的资源总是等于它的请求，其他namespace则共享其余的资源。

_Customized Controller和ThirdPartyResource_:

ThirdPartyResource是使用新的API对象类型来扩展Kubernetes API的一种方法。新的API类型将被赋予一个API endpoint并支持相应的增、删、改、查操作。您可以使用此API endpoint 创建自定义对象。通过 [mesos-style.md](https://github.com/kubernetes/kubernetes/blob/master/docs/devel/mesos-style.md) 和ThirdPartyResource，开发人员可以使用自定义对象构建workload customized controller。[k82cn/kube-arbitrator](https://github.com/k82cn/kube-arbitrator) 有一个例子，它通过ThirdPartyResource功能提供资源共享和抢占功能。

_水平/垂直缩放和节点级QoS_:
节点级资源使用率的改进，对集群级资源共享并无贡献。

PS: 最近有一个关于"垂直缩放"的讨论/提案，个人感觉可以通过 vpa/hpa 与 arbitrator/scheduler 联动来管理集群资源，从而动态调整集群的资源分配情况来提高资源的使用效率。

**解决方案/提案**：

_概述_:

为了达到上述要求，引入了一个新的组件（k8s-arbitrator）和两个 ThirdPartyResource（Consumer / Allocation）。

以下yaml文件演示了Consumer的定义：

```yaml
apiVersion: kuabe-arbitrator.incubator.k8s.io/v1
kind: Consumer
metadata:
  name: defaults
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
  reserved:
    limits.memory: 1Gi
```

对于每个Consumer对象，它有两个字段: `hard` 和 `reserved`：

* `reserved`：定义了为namespace保留的资源。它使用与"Compute Resource Quota"和"Storage Resource Quota"相同的资源类型。`reserved`不能超过`hard` 定义的资源。
如果启用了ResourceQuota Admission，它还将检查“预留”的总数是否超过了群集中的资源。
* `hard`：定义了namespace可以使用的最大资源；它不能超过namespace的`Quota.hard`。

`Consumer`是由仲裁员为每个命名空间创建的，并且必要时由集群管理员进行更新；仲裁员创建具有无限`hard`和零`reserved`的保留的`Consumer`，因此namespace默认共享集群资源。

arbitrator将创建或更新Allocation中的额外字段: `deserved`

* `deserved`：类似于Quota中的“Used”，它没有在yaml文件中定义，而是由arbitrator更新。它定义了arbitrator分配给命名空间的总资源。它不会走过`Quota.hard`，也可能因namespace的资源请求而改变。
* `hard`/`deserved`：从`Consumer`复制；如果`Consumer`被更新，它也将在下一个调度周期被更新

```yaml
apiVersion: kuabe-arbitrator.incubator.k8s.io/v1
kind: Allocation
metadata:
  name: defaults
spec:
  deserved:
    cpu: "1.5"
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
  reserved:
    limits.memory: 1Gi
```

下图显示了Consumer/Allocation中的`hard`，`reserved`和`deserved`的关系。

注意：只有"Compute Resource Quota"和"Storage Resource Quota"可用于`reserved`和`deserved`。

```text
    -------------  <-- hard
    |           |
    |           |
    - - - - - - -  <-- deserved
    |           |
    |           |
    |           |
    -------------  <-- reserved
    |           |
    -------------
```

k8s-arbitrator 是一个新的组件，它会创建/更新 Consumer 和 Allocation:

* 基于arbitrator的策略计算 deserved 资源（Allocation.deserved）；例如：DRF和namespace的请求（PoC中使用了pending pod）
* 如果namespace 使用了过多的资源 (used > deserved)，arbitrator 通知相应的controller，并在指定时间后有选择的终止Pod

同时，k8s默认调度器仍然根据其策略来分派任务到主机，例如 PodAffinity：k8s-arbitrator 负责 resource allocation, k8s-scheduler 负责 resource assignment

Arbitrator 将DRF作为默认策略。它将从 k8s-apiserver 中取得pod/node，并根据DRF算法计算每个namespace的deserved资源；然后更新相应的配置。默认调度间隔为1s，可配置。arbitrator不会将主机名分配给deserved资源中；它依赖默认的调度器(kube-scheduler)在适当的主机上分派任务。

仲裁员还符合以下要求：

* namespace的总体`deserved`资源不能超过群集中的资源
* `deserved`资源不能超过消费者的`hard`资源
* 如果集群中有足够的资源，`deserved`资源不能少于`reserved`资源

**抢占**：

当资源请求/配额发生变化时，每个命名空间的deserved资源也可能会发生变化。 “较高”的优先级Pods将可能触发eviction，下图显示了由于`deserved`资源变化而引发eviction的情况。

```text
T1:                T2:                              T3:
 --------------    -------------- --------------    -------------- --------------
 | Consumer-1 |    | Consumer-1 | | Consumer-2 |    | Consumer-1 | | Consumer-2 |
 |   cpu:2    | => |   cpu:1    | |   cpu:0    | => |   cpu:1    | |   cpu:1    |
 |   mem:2    |    |   mem:1    | |   mem:0    |    |   mem:1    | |   mem:1    |
 --------------    -------------- --------------    -------------- --------------
```

* T1：集群中只有一个namespace：Consumer-1; 所有资源（cpu：2，mem：2）都被分配给它
* T2：创建一个新的namespace: Consuemr-2; arbitrator 重新计算每个namespace的资源分配，缩小overused的namespace
* T3：管理overused的namespace的controller必须选择一个Pod来杀死，否则arbitrator将会随机抽取需要杀死的Pods。Evict后，资源会被分配给underused的namespace

Arbitrator使用pod的 “/evict” REST API 来回收资源。但是当arbitrator选择需要被杀死的Pods时，至少有两个要求：
  * Evict后，pods不能少于PodDisruptionBudget
  * Evict后，namespace的资源不能少于`reserved`

namespace在驱逐后可能会变成`underused`; arbitrator将尝试从最overused的namespace尝试杀死Pods。 对于资源碎片的问题，暂时不在本文的讨论范围内; 将在抢占实施文档中讨论设计细节。

**功能交互**：

_Workload customized controller_:

Aribtrator也对workload customized controller 起作用，它会杀死overused的namespace中的Pods。Customized controller不能比Allocation.deserved使用更多的资源，如果Allocation.deserved变小，customized controller 需要选择Pods杀死；否则arbitrator将在grace period后杀死 Pods（例如FCFS）。

_Multi-scheduler_:

如果使用multi-scheduler，只能启动一个arbitrator以避免竞争条件

_Admission Controller_:

AribtratorAdmission检查Consumer相对于Quota中"Compute Resource"和"Storage Resource"的定义：`Consumer.hard`不能超过`Quota.hard`。 ResourceQuotaAdmission的其他指标将遵循当前行为。对其他Admission插件没有影响。

_节点级QoS_

在原型中，仅考虑`request`；`limit`会在后面的版本考虑

_ReplicaController/ReplicaSet_

由于Consumer/Allocation是namespace级别而不是Job/RC级别，k8s-controller-manager不能根据RC的比例创建pod；它需要最终用户在RC之间平衡Pods或在Consumer中请求预留资源。

_kubelet_:

现在还不涉及kubelet，虽然命名空间的Allocation.deserved也是kubelet中evict的一个因素，例如：

* 拒绝overused的namespace的请求
* 如果节点资源耗尽，则选择最overused的namespace里的Pods

**目前进展**:

* 基于DRF的arbitrator策略（完成）
* 由k8s-arbitrator自动创建Consumer/Allocation（持续）
* 面向 Consumer.reserved 的策略（持续）
* 其它，例如doc，测试（持续）

**路线图和未来**:

* 高级的策略，例如不打破PodDisruptionBudget
* 使用 “resourceRequest” 来减少 pending pod的使用
* 加强关于存储策略，例如“PV / PVC”
* k8s-arbitrator的HA
* 层级Consumer
* 无限Allocation用于kube-system
* 面向 limit 的策略（节点/资源QoS）
* Consumer / Allocation 客户端库，用于开发新的workload customized controller

**引用**:

* [Kubernetes] ResourceQuota: [http://kubernetes.io/docs/admin/resourcequota/](http://kubernetes.io/docs/admin/resourcequota/)
* [Kubernetes] Admission Control: [http://kubernetes.io/docs/admin/admission-controllers/](http://kubernetes.io/docs/admin/admission-controllers/)
* [Kubernetes] Preemption and Re-scheduler: [https://github.com/kubernetes/kubernetes/blob/master/docs/proposals/rescheduling.md](https://github.com/kubernetes/kubernetes/blob/master/docs/proposals/rescheduling.md)
* [Kubernetes] Node-level QoS: [http://kubernetes.io/docs/user-guide/compute-resources/](http://kubernetes.io/docs/user-guide/compute-resources/) 
* Kubernetes on EGO: [http://sched.co/8K3n](http://sched.co/8K3n) 
* Kubernetes on Mesos: [https://github.com/kubernetes-incubator/kube-mesos-framework/](https://github.com/kubernetes-incubator/kube-mesos-framework/) 
* IBM Spectrum Conductor for Container: [http://ibm.biz/ConductorForContainers](http://ibm.biz/ConductorForContainers) 
* Support Spark natively in Kubernetes: [https://github.com/kubernetes/kubernetes/issues/34377](https://github.com/kubernetes/kubernetes/issues/34377) 
* Multi-Scheduler in Kubernetes: [https://github.com/kubernetes/kubernetes/blob/master/docs/proposals/multiple-schedulers.md](https://github.com/kubernetes/kubernetes/blob/master/docs/proposals/multiple-schedulers.md) 
