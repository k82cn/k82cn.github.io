---
layout: post
categories: tech
title: CSF重构
---

## 主要想法：

将作业与插件分离，通过定义新的作业融合各个插件。

## 实现细节：

### 客户端：

1.提供服务配置查询命令。
2.由客户端指定作业类型。 例如 csf-job-run -t workflow myjobs.xml

提交者能过查询命令查看CSF配置情况，结合自己作业类型决定提交类型及是否提交。
与现有CSF相比，增加了作业类型，比较容易的融合各插件；由于提交者更了解自己的作业类型，方便作业的分类。
服务端根据作业类型创建作业（为JobResource的子类）

### 服务端：

增加配置文件语义。如下：

    <queues>
        <queue name='normal' ... />
            <plugin name='wfplugin' ... />
        </queue>
    </queues>

    <jobs>
        <job name='workflow' class='com.platform.metascheduler.WFJob'/>
    </jobs>

    <plugins>
        <plugin name='wfplugin' job='workflow' class='com.platform.metascheduler.WorkFlowPlugin'/>
    </plugins>

开发者可以仅通过开发新的作业类型使用多个插件；而作业和插件通过配置文件结合在一起。普通作业也应作与其它作业同等对待，作为默认作业类型。

### 融合多个插件如下：

    <queues>
        <queue name='workflow_parallel' ... />
            <plugin name='wp_wf_plugin' ... />
            <plugin name='wp_pa_plugin' ... />
        </queue>
    </queues>

    <jobs>
        <job name='wf_pa' class='com.platform.metascheduler.WFPAJob' ... />
    </jobs>

    <plugins>
        <plugin name='wp_wf_plugin' job='wf_pa' class='com.platform.metascheduler.WorkFlowPlugin' ... />
        <plugin name='wp_pa_plugin' job='wf_pa' class='com.platform.metascheduler.ParallelPlugin' ... />
    </plugins>

插件开发者应提供其插件应用的接口，各插件就面向接口编程。应用多个插件作业实现各个接口。如下：

    public class WFPAClass implements WorkFlowJob, ParallelJob{
        ......
    }

在实现类中使作业描述匹配各插件。

### 关于消息驱动机制：

消息驱动机制与轮询机制都做为调度模块存在于csf中。不同队列注册到不同调度模块上。如下：

    <queue type='notify' ... />
    <queue type='normal' ... />

当增加其它机制时不需要对以有的队列做太多改动。

## 优缺点：

* 优点：应用设计模式中面向接口编程，提高了灵活性；将插件、作业与调度框架中分离，提高调度框架的纯度。
* 缺点：对CSf需作较多改动，测试工作量比较大。建意从配置文件改起，先定义配置文件语义。
