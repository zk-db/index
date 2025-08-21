---
title: Kubernetes基础概念
date: 2022-06-02 10:51:41
tags:
  - k8s
---

# 什么是Kubernetes

开源的容器管理平台。通过配置即可自动调度管理容器。拥有以下功能：

- **服务发现与负载均衡** k8s可以通过DNS或者IP暴露容器服务并自动发现，如果对于容器服务请求过高，k8s可以进行负载均衡流量，使服务平稳运行
- **存储编排** 可以自由挂在存储资源，例如本地存储，云存储等
- **自动状态更新** 通过配置新的容器状态，随后k8s将创建新的容器并进行资源转移
- **机器资源配置** 可以为每个容器配置需要的CPU以及内存资源
- **自愈** 发现服务异常自动重启
- **敏感信息配置** 对于密码，token，keys等敏感信息由k8s私密管理

<!--more-->

# Kubernates集群组件

k8s集群包换了一些`worker`机器，被称为`nodes`。worker上会运行容器程序，每个集群至少有一个`worker`。`control plan`管理这些worker nodes 以及 Pods，通常在生产环境会运行于多个节点上。提供high availability服务。

组件：

![components-of-kubernetes](components-of-kubernetes.svg)

## Control Plane Components

Control Plane Components 主要用来做一些决策，例如启动新的副本如果副本数不足。一些请求事件响应，如创建了dployment就同步创建启动pod。通常情况下部署在相同节点下，并且不在该节点下运行用户容器。参考：[Kubbernetes Components](https://kubernetes.io/docs/concepts/overview/components/)



- kube-apiserver

  用于暴露k8s api

- etcd

  高可用键值数据库

- kube-scheduler

  根据resource定义分配满足执行条件节点并运行

- kube-controller-manager

  运行控制器进程的控制组件。从逻辑上讲，每个控制器都是一个单独的进程，但为了降低复杂性，它们都编译成单个二进制文件并在单个进程中运行。

  这些控制器的某些类型是：

  - 节点控制器：负责节点发生故障时的注意和响应。

  - 作业控制器：监视表示一次性任务的作业对象，然后创建Pod来运行这些任务完成。

  - 端点控制器：填充端点对象（即加入服务和Pod）。

  - 服务帐户和令牌控制器：为新命名空间创建默认帐户和API访问令牌。

- cloud-controller-manager

  可以将当前集群加入云集群，具体参考[Kubbernetes Components](https://kubernetes.io/docs/concepts/overview/components/)

## Node Components

- kubelet
- kube-proxy
- Container runtime



## Addons

- DNS
- Web UI（Dashboard）
- Container Resource Monitoring
- Cluster-level Logging

# Kubernetes常用对象

Kubernetes对象是Kubernetes系统中的持久实体。Kubernetes使用这些实体来表示集群的状态。具体来说，他们可以描述：

- 正在运行哪些容器化应用程序（以及哪些节点）

- 这些应用程序可用的资源

- 围绕这些应用程序行为的策略，例如重新启动策略、升级和容错性

Kubernetes对象是“意图记录”，一旦您创建对象，Kubernetes系统将不断工作以确保该对象的存在。通过创建对象，您可以有效地告诉Kubernetes系统您希望集群的工作负载是什么样子；这是集群所需的状态。

要使用Kubernetes对象，无论是创建、修改还是删除它们——您需要使用Kubernetes API。例如，当您使用kubectl命令行界面时，CLI会为您进行必要的Kubernetes API调用。您还可以使用客户端库之一直接在自己的程序中使用Kubernetes API。

## Object Spec and Status

对象规格和状态

几乎每个Kubernetes对象都包含两个控制对象配置的嵌套对象字段：对象规范和对象状态。对于具有规范的对象，您必须在创建对象时设置此设置，并描述您希望资源具有的特征：其所需的状态。

该状态描述了对象的当前状态，由Kubernetes系统及其组件提供和更新。Kubernetes控制平面持续并主动地管理每个对象的实际状态，以匹配您提供的期望状态。

例如：在Kubernetes中，部署是一个可以表示在集群上运行的应用程序的对象。当您创建部署时，您可以将部署规范设置为指定您希望运行应用程序的三个副本。Kubernetes系统读取部署规范，并启动所需应用程序的三个实例——更新状态以匹配您的规范。如果其中任何实例出现故障（状态更改），Kubernetes系统会通过更正来响应规范和状态之间的差异——在这种情况下，启动替换实例。

有关对象规范、状态和元数据的更多信息，请参阅[Kubernetes API Conventions](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md).

## Describing a Kubernetes object

当您在Kubernetes中创建对象时，您必须提供描述其所需状态的对象规范，以及有关对象的一些基本信息（例如名称）。当您使用Kubernetes API创建对象（直接或通过kubectl）时，该API请求必须在请求主体中包含该信息作为JSON。通常，您在.yaml文件中向kubectl提供信息。kubectl在提出API请求时将信息转换为JSON。

这里有一个.yaml文件示例，显示Kubernetes部署的必填字段和对象规范：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

可以使用`kubectl apply`命令创建Deployment

```bash
kubectl apply -f https://k8s.io/examples/application/deployment.yaml
# output:
# deployment.apps/nginx-deployment created
```

## 必须字段

在您要创建的Kubernetes对象的.yaml文件中，您需要为以下字段设置值：

- apiVersion - 您正在使用哪个版本的Kubernetes API来创建此对象
- kind - 您想创建哪种对象

- metadata - 有助于唯一识别对象的数据，包括名称字符串、UID和可选命名空间

- spec - 您希望对象处于什么状态

- 对象规范的精确格式对每个Kubernetes对象都不同，并包含特定于该对象的嵌套字段。Kubernetes API参考可以帮助您找到可以使用Kubernetes创建的所有对象的规范格式。

例如，请参阅Pod API参考的规范字段。对于每个Pod，.spec字段指定pod及其所需状态（例如该pod中每个容器的容器映像名称）。对象规范的另一个例子是StatefulSet API的规范字段。对于StatefulSet，.spec字段指定StatefulSet及其所需状态。在StatefulSet的.spec中是Pod对象的模板。该模板描述了StatefulSet控制器为满足StatefulSet规范而创建的Pod。不同类型的对象也可以具有不同的.status；同样，API参考页面详细说明了该.status字段的结构，以及每种不同类型对象的内容。

# 其他

- namespace
- node
- pod
- deployment
- service
- volumn

# 待补充。。。。。。

