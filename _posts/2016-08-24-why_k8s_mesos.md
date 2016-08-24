---
layout: post
title: Exploring Kubernetes as a Mesos Framework&#58; Does it make sense?
categories: tech
---

When looking at container orchestration platforms, open-source communities have produced a number of viable options including Kubernetes, Marathon-Mesos, or Docker Swarm. Kubernetes stands out as a popular choice amongst many users looking to run cloud-native on-line workloads. It has built-in support for a number of useful features, including automated deployment, load-balancing, auto-scaling, rolling upgrades, and so forth that DevOps users find essential.

At the same time, when one looks at the breadth of workloads in typical enterprise environments, there is the argument that a more generic resource manager is needed to handle common concerns across frameworks. This is where Apache Mesos comes in. It is a lightweight system – only 230K lines of code vs 1.3M+ in Kubernetes. Its focus is to provide a base resource abstraction layer that tracks the availability of resources, allocates them to frameworks, handles base task execution capabilities, and also handles admin operations such as host maintenance mode.

At IBM we are exploring how Mesos and Kubernetes can work together with Kubernetes running as a Mesos framework. While there is some overlap between Mesos and the built-in resource management capabilities available in Kubernetes, it is possible to use both tools in a synergistic manner. One of the advantages of this model is the ability to support a broader class of workloads like Big Data, Spark, and other analytics systems that can benefit from the fine-grained resource allocation and negotiation logic that Mesos supports. We think this will lead to improved resource utilization and performance when running highly dynamic workloads.

Because it is light-weight, Mesos easily scales to 10s of thousands of nodes in production environments. Kubernetes on the other hand is improving rapidly in terms of scale and performance, but is still limited to 1000 nodes. By running Kubernetes on Mesos, this opens the possibility of running multiple Kubernetes instances on a common Mesos substrate.

While Kubernetes is strictly focused on containers, Mesos is more general purpose in that it can support any OS process-level workload. This can be important to enterprises that have existing workloads that they want to bring into a common management environment. Kubernetes allows enterprises to enter the new world of cloud-native containerized workloads, while Mesos forms a bridge to existing Windows, UNIX, and Linux environments.

As part of this exploration, IBM is taking the lead in making contributions to the Kubernetes on Mesos integration project. This includes work on helping Kube-DNS work with external DNS ([#28453](https://github.com/kubernetes/kubernetes/issues/28453)), improvements to scheduling algorithms in Kubernetes ([#31068](https://github.com/kubernetes/kubernetes/issues/31068)), getting Kubernetes namespaces to work better with Mesos ([#31069](https://github.com/kubernetes/kubernetes/issues/31069)), as well as better support for running Kubernetes in heterogeneous hardware environments ([#29901](https://github.com/kubernetes/kubernetes/issues/29901)). This work is in addition to the contributions IBM is making directly to the Kubernetes and Mesos projects.

We will soon be sharing more information on how we are packaging these technologies in a consumable way to enable average enterprise customers to easily set up their own container management platform for diverse workloads. Stay tuned!
