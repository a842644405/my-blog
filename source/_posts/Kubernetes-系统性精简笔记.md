---
title: Kubernetes 系统性精简笔记
date: 2026-08-01 09:32:22
---

既然你已经有 **Kubernetes 集群部署 + 微服务部署** 的经验，那么你的复习重点不应该是 **"怎么用"**，而应该是 **"为什么这样设计"** 和 **"各组件之间如何协作"**。

我建议采用**自上而下（Top-Down）+ 数据流驱动**的方式复习，而不是按官方文档一章章看。

------

# Kubernetes 系统性精简笔记（适合中高级复习）

> 一句话理解：
>
> **Kubernetes = 声明式资源管理 + 控制器模式 + 调度系统 + 容器网络 + 服务发现。**

整个 Kubernetes 可以拆成五层：

```text
应用层
    ↓
资源层
    ↓
控制层
    ↓
节点层
    ↓
基础设施层
```

------

# 第一层：应用层（我要运行什么）

用户永远只关心：

> 我要运行一个微服务。

例如：

```text
Deployment

↓

ReplicaSet

↓

Pod

↓

Container
```

所以：

```
Deployment
    ↓
管理 Pod 生命周期
```

需要牢记：

| 资源        | 作用         |
| ----------- | ------------ |
| Pod         | 最小运行单元 |
| Deployment  | 无状态管理   |
| StatefulSet | 有状态应用   |
| DaemonSet   | 每节点一个   |
| Job         | 一次性任务   |
| CronJob     | 定时任务     |

一句话：

> **工作负载（Workload）决定 Pod 如何存在。**

------

# 第二层：资源层（资源如何暴露）

Pod 生命周期很短。

所以不能直接访问 Pod。

于是：

```
Pod

↓

Service

↓

Ingress
```

记住三句话：

Pod：

> 提供能力

Service：

> 提供稳定入口

Ingress：

> 提供外部访问

------

Service：

```
ClusterIP

↓

VIP（Virtual IP）

↓

iptables/IPVS

↓

Pod
```

本质：

**负载均衡。**

## service

## 第一步：如果没有 Service

假设有两个 Pod：

```markdown
PodA
10.244.1.2

PodB
10.244.2.3
```

PodA 要访问 PodB：

```text
curl 10.244.2.3:8080
```

数据包：

```text
PodA
    │
    ▼
10.244.2.3
    │
    ▼
PodB
```

非常简单。

但是有个问题：

PodB 挂了。

新的 Pod：

```text
10.244.2.8
```

原来的：

```text
10.244.2.3
```

已经不存在了。

那么：

```text
curl 10.244.2.3
```

立刻失败。

所以：

> **Pod IP 不稳定。**

------

## 第二步：Service 解决什么问题？

于是 Kubernetes 发明了：

```text
Service
```

例如：

```yaml
Service

ClusterIP:

10.96.0.100
```

以后：

PodA：

不是访问：

```text
10.244.2.3
```

而是：

```text
10.96.0.100
```

也就是：

```text
ClusterIP
```

现在变成：

```text
PodA

↓

10.96.0.100

↓

PodB
```

你可以理解成：

> **ClusterIP 就像一个"固定电话号码"。**

Pod：

就是：

> **员工。**

员工天天换工位。

但是：

客服电话：

永远：

```text
400-xxxx
```

------

## 第三步：那 ClusterIP 到底存在吗？

这是最关键的一步。

很多人误以为：

```text
10.96.0.100
```

是一张真实网卡。

其实：

**不是。**

它只是：

> **一个虚拟 IP（VIP，Virtual IP）。**

例如：

服务器：

根本没有：

```text
eth0

10.96.0.100
```

你：

```bash
ip addr
```

也找不到：

```text
10.96.0.100
```

那么：

为什么还能访问？

因为：

**iptables（或 IPVS）拦截了数据包。**

------

## 第四步：iptables 做了什么？

假设：

PodA：

```text
curl 10.96.0.100:80
```

Linux Kernel：

发现：

目标：

```text
10.96.0.100
```

iptables：

提前配置了一条规则：

```text
如果：

目标IP == 10.96.0.100

那么：

改成：

10.244.2.3
```

于是：

原来：

```text
Destination

10.96.0.100
```

变成：

```text
Destination

10.244.2.3
```

所以：

真正发送的是：

```text
PodA

↓

10.244.2.3

↓

PodB
```

这里最重要的一句话：

> **Service 不转发数据，Service 只是一个规则；真正修改数据包的是 Linux 内核中的 iptables（或 IPVS）。**

------

## 第五步：为什么还能负载均衡？

假设：

后端有三个 Pod：

```text
Pod1

10.244.2.3

Pod2

10.244.2.4

Pod3

10.244.2.5
```

iptables：

可以这样：

第一次：

```text
10.96.0.100

↓

10.244.2.3
```

第二次：

```text
10.96.0.100

↓

10.244.2.4
```

第三次：

```text
10.96.0.100

↓

10.244.2.5
```

于是：

看起来就是：

```text
Service

↓

负载均衡

↓

多个 Pod
```

其实：

真正工作的：

还是：

```text
iptables
```

------

## 第六步：为什么又出现 IPVS？

iptables：

本质：

就是：

很多很多规则。

例如：

```text
如果：

ClusterIP=A

跳这里

如果：

ClusterIP=B

跳那里

如果：

ClusterIP=C

跳那里
```

当：

Service：

越来越多：

例如：

```text
5000 个 Service
```

iptables：

匹配：

越来越慢。

所以：

后来：

Linux：

提供：

```text
IPVS
```

IPVS：

就是：

Linux 内核里面：

专门做：

```text
Load Balance
```

的数据结构。

于是：

以前：

```text
iptables

↓

改地址
```

后来：

```text
IPVS

↓

直接查哈希表

↓

找到 Pod
```

性能：

高很多。

------

## 第七步：现在再看这条链路

你之前看到的是：

```text
ClusterIP

↓

VIP

↓

iptables/IPVS

↓

Pod
```

现在可以理解成：

```text
PodA

↓

访问固定地址

10.96.0.100

↓

这个地址其实不存在

↓

Linux Kernel发现：

这是Service

↓

iptables/IPVS

修改目标地址

↓

真正Pod

10.244.2.3

↓

PodB
```

------

## 最后，一张图彻底理解

```text
                PodA

                  │
        curl 10.96.0.100
                  │
                  ▼
         ClusterIP (VIP)
         10.96.0.100
                  │
                  │（没有真实网卡）
                  ▼
        kube-proxy 提前写好的
    iptables / IPVS 规则生效
                  │
                  ▼
     改写目标地址（DNAT）
                  │
      ┌───────────┼───────────┐
      ▼           ▼           ▼
10.244.2.3   10.244.2.4   10.244.2.5
   Pod1         Pod2         Pod3
```

------

### 你可以记住一句最重要的话（面试几乎必问）

> **ClusterIP 不是一个真实存在的 IP，而是一个虚拟 IP（VIP）。kube-proxy 会在每个 Node 上下发 iptables 或 IPVS 规则，当访问 ClusterIP 时，Linux 内核会根据这些规则将请求做 DNAT，改写为某个后端 Pod 的真实 IP，实现 Service 的负载均衡。**

如果你理解了这一点，那么 **kube-proxy、Service、EndpointSlice、Ingress** 的工作原理就能串成一条完整的链路了。



------

## Ingress：

```
HTTP

↓

Ingress Controller

↓

Service

↓

Pod
```

注意：

Ingress：

不是负载均衡器。

Ingress Controller：

才是真正工作的组件。

------

# 第三层：控制层（为什么 YAML 会变成 Pod）

这是 Kubernetes 核心。

牢记：

> Kubernetes 从来不会主动执行 YAML。

真正执行的是：

```
Controller
```

例如：

Deployment：

```
Deployment

↓

Deployment Controller

↓

ReplicaSet

↓

ReplicaSet Controller

↓

Pod
```

Controller 永远做一件事：

```
Desired

VS

Current
```

如果：

```
Desired

3 Pods
```

实际：

```
2 Pods
```

Controller：

```
创建一个
```

所以：

> Kubernetes 本质就是一个无限循环的状态修正器（Reconciliation Loop）。

------

# 第四层：调度层（Pod 为什么跑到某节点）

Pod 创建以后：

Scheduler 接管。

Scheduler：

```
Pending Pod

↓

Filter

↓

Score

↓

Bind
```

过程：

第一步：

过滤：

```
NodeSelector

Taint

Resource

Affinity
```

第二步：

评分：

例如：

```
CPU

Memory

Topology

Image
```

第三步：

绑定：

```
Pod

↓

Node
```

一句话：

> Scheduler 不创建 Pod，只决定去哪。

------

# 第五层：节点层（Node 怎么运行 Pod）

Master：

告诉 Node：

```
运行 Pod
```

Node：

```
kubelet

↓

CRI(Containerd)

↓

OCI(runc)

↓

Container
```

流程：

```
API Server: Kubernetes API的入口，负责认证、授权、准入控制以及所有资源操作的REST接口。
↓
kubelet: 节点上的核心代理，管理Pod的生命周期，确保容器按预期运行并与控制面通信。
↓
containerd: 容器运行时守护进程，负责拉取镜像、管理容器存储和执行。
↓
runc: 符合OCI标准的低层容器运行时，直接调用Linux内核特性创建和运行容器。
↓
Linux Namespace: 内核隔离机制，为容器提供独立的进程、网络、挂载等视图。
↓
Container: 由镜像创建的轻量级、可执行的软件包，包含应用及其依赖，在独立环境中运行。
```

------

# 第六层：网络（Pod 为什么互通）

牢记一句：

> Kubernetes 不负责网络。

交给：

```
CNI Container Network Interface
```

例如：

```
Calico

Flannel

Cilium
```

职责：

```
创建 veth

↓

配置 IP

↓

配置路由

↓

跨节点通信
```

Pod：

要求：

```
任意 Pod

↓

无需 NAT

↓

直接互通
```

这是 Kubernetes 网络模型。

------

# 第七层：Service 如何找到 Pod

Service：

并不知道 Pod。

真正维护的是：

```
EndpointSlice
```

流程：

```
Pod

↓

Label

↓

Service Selector

↓

EndpointSlice

↓

kube-proxy

↓

iptables/IPVS

↓

Pod
```

所以：

Service：

不是 DNS。

真正转发：

```
kube-proxy
```

------

# 第八层：DNS

为什么：

```
mysql.default.svc.cluster.local
```

能访问？

因为：

```
CoreDNS

↓

解析 Service

↓

ClusterIP

↓

iptables

↓

Pod
```

流程：

```
Application

↓

DNS

↓

ClusterIP

↓

Pod
```

------

# 第九层：配置

配置：

```
ConfigMap
```

敏感数据：

```
Secret
```

挂载方式：

```
ENV

Volume
```

更新：

```
修改 ConfigMap

↓

不会自动重启 Pod

↓

需要滚动更新
```

------

# 第十层：存储

Pod：

生命周期：

```
短
```

Volume：

生命周期：

```
长
```

流程：

```
PVC

↓

PV

↓

StorageClass

↓

CSI

↓

Storage
```

牢记：

PVC：

申请。

PV：

资源。

StorageClass：

自动创建。

------

# 第十一层：Master 核心组件

```
kubectl

↓

API Server

↓

etcd
```

所有组件：

```
Scheduler

Controller

kubelet
```

全部：

```
访问 API Server
```

API Server：

唯一入口。

------

组件职责：

| 组件               | 职责         |
| ------------------ | ------------ |
| API Server         | 统一入口     |
| etcd               | 保存状态     |
| Scheduler          | 调度         |
| Controller Manager | 修正状态     |
| CoreDNS            | DNS          |
| kube-proxy         | Service 转发 |

------

# 第十二层：一次完整的数据流（最重要）

用户：

```
kubectl apply deployment.yaml
```

发生：

```
API Server

↓

保存 etcd

↓

Deployment Controller

↓

ReplicaSet

↓

Scheduler

↓

Node

↓

kubelet

↓

containerd

↓

Pod Running
```

访问：

```
Browser

↓

Ingress

↓

Service

↓

Pod
```

Pod：

访问：

```
mysql.default

↓

CoreDNS

↓

Service

↓

iptables

↓

MySQL Pod
```

------

补充信息：

Pod 对象本质上是 **API Server 中 etcd 里的一条 JSON/YAML 格式的资源记录**，可以理解为一份描述性的“声明文件”。

具体来说，Pod 对象包含以下几类信息：

1. **元数据**：名称、命名空间、标签、UID 等唯一标识。
2. **Spec（期望状态）**：定义了该 Pod 要运行哪些容器（镜像、端口、环境变量、卷挂载等）、是否需要共享网络命名空间等。
3. **Status（实际状态）**：由 kubelet 持续更新，反映 Pod 当前所处的阶段（Pending、Running、Succeeded 等）、容器的状态、IP 地址等信息。

所以 Pod 对象本身**不是进程，也不是容器**，而是一个持久化的数据结构，用来声明“我希望一组容器以某种方式运行”。kubelet 根据这个对象去创建真实的 Linux 容器进程，并把容器的实时状态同步回该对象。

# 第十三层：升级流程

Deployment：

```
v1

↓

RollingUpdate

↓

新 Pod Ready

↓

删除旧 Pod
```

保证：

```
不停机
```

回滚：

```
kubectl rollout undo
```

------

# 第十四层：排障思路（面试高频）

Pod 起不来：

```
describe

↓

Events

↓

Logs

↓

Node

↓

Image

↓

资源
```

Service 不通：

```
Pod

↓

Label

↓

EndpointSlice

↓

Service

↓

kube-proxy
```

Ingress 不通：

```
Ingress

↓

Controller

↓

Service

↓

Pod
```

PVC Pending：

```
PVC

↓

StorageClass

↓

PV

↓

CSI
```

Node NotReady：

```
kubelet

↓

containerd

↓

网络

↓

磁盘
```

------

# 第十五层：一张脑图（建议每天看 5 分钟）

```text
                    Kubernetes

                        │
        ┌───────────────┼────────────────┐
        │               │                │
   Workload         Networking      Storage
        │               │                │
 Deployment        Service          PVC
 StatefulSet       Ingress          PV
 DaemonSet         CoreDNS          CSI
        │               │
        └───────Pod─────┘
                │
           kubelet
                │
          containerd
                │
              runc
                │
        Linux Namespace
                │
              Node
                │
 Scheduler ← API Server → Controller
                │
              etcd
```

------

# 最后一页（建议背熟）

**一句话概括 Kubernetes：**

> **用户提交期望状态（YAML）→ API Server 持久化到 etcd → Controller 持续对比期望与实际并修正状态 → Scheduler 为 Pod 选择节点 → kubelet 调用 containerd/runc 创建容器 → CNI 提供 Pod 网络 → kube-proxy/CoreDNS 提供服务发现与访问 → CSI 提供持久化存储。**

**复习建议（DevOps 面试非常有效）：**

每天按下面 8 个问题自问一遍，能回答出来就说明框架已经很扎实：

1. `kubectl apply` 后，资源经历了哪些组件？
2. Deployment 为什么能自动拉起 Pod？
3. Scheduler 是如何给 Pod 选节点的？
4. Service 为什么能访问到后端 Pod？
5. 不同节点上的 Pod 为什么可以直接通信？
6. ConfigMap 和 Secret 更新后为什么 Pod 不会自动生效？
7. PVC 是如何最终绑定到真实存储的？
8. 一个请求从浏览器进入集群，最终到达 Pod 的完整链路是什么？

如果这 8 个问题都能用自己的话完整讲清楚，大多数 Kubernetes 运维、平台工程师或 DevOps 面试中的原理类问题都会有扎实的基础。
