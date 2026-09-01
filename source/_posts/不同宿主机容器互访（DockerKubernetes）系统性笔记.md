下面这份笔记按照 **"为什么 → 是什么 → 怎么实现 → 数据包怎么走 → 常见方案 → 排障"** 的顺序组织，目标是建立一个完整的网络模型，而不是死记命令。

------

# 不同宿主机容器互访（Docker/Kubernetes）系统性笔记

## 一、先理解本质（一句话）

> **不同宿主机容器互访，本质就是：如何让不同机器上的虚拟网卡能够互相通信。**

所以问题其实变成：

```
Container A
    │
veth
    │
Docker Bridge
    │
Host A
    │
=================== 网络 ===================
    │
Host B
    │
Docker Bridge
    │
veth
    │
Container B
```

整个过程就是：

> **如何让 HostA 知道 ContainerB 的 IP 在哪里。**

------

# 二、第一层：单机容器为什么能通信？

例如：

```
Host

docker0 (172.17.0.1)
      │
 ┌────┴────┐
 │         │
C1       C2
172.17.0.2
172.17.0.3
```

Docker 创建：

- Bridge
- veth pair
- namespace

通信过程：

```
C1
↓

veth

↓

docker0

↓

veth

↓

C2
```

全部发生在 Linux Kernel。

**没有经过物理网卡。**

------

# 三、为什么跨宿主机不能直接通信？

HostA：

```
docker0

172.17.0.0/16
```

HostB：

```
docker0

172.17.0.0/16
```

问题来了：

ContainerA：

```
172.17.0.2
```

ContainerB：

```
172.17.0.2
```

IP 重复。

HostA 根本不知道：

```
172.17.0.2
```

到底是谁。

所以：

**Docker Bridge 默认只能单机。**

------

# 四、那跨宿主机需要什么？

需要两个东西：

## 第一：

全局唯一 IP

例如：

```
HostA

PodA

10.244.1.2
```

HostB

```
PodB

10.244.2.3
```

整个集群：

```
10.244.x.x
```

不会重复。

------

第二：

HostA 必须知道：

```
10.244.2.3

下一跳是谁？
```

即：

需要路由。

所以：

跨宿主机容器通信

= IP规划

- 

路由

------

# 五、数据包到底怎么走？

假设：

```
PodA

10.244.1.2
```

访问：

```
PodB

10.244.2.3
```

流程：

```
PodA

↓

veth

↓

Bridge

↓

HostA Routing Table

↓

eth0

↓

交换机

↓

HostB eth0

↓

HostB Routing

↓

Bridge

↓

veth

↓

PodB
```

整个过程：

```
容器

↓

Linux Bridge

↓

Linux Routing

↓

物理网络

↓

Linux Routing

↓

Linux Bridge

↓

容器
```

所以：

真正工作的：

不是 Docker。

而是：

Linux Networking。

------

# 六、Linux 为什么知道去哪里？

例如：

HostA：

```
Destination

10.244.2.0/24

via

192.168.1.20
```

意思：

```
去 Pod 网段

10.244.2.*

先发给：

HostB
```

所以：

HostA：

```
Pod IP

↓

HostB

↓

Pod
```

------

# 七、Overlay 网络怎么解决？

最经典：

VXLAN。

原来：

```
PodA

↓

HostA

↓

HostB

↓

PodB
```

现在：

中间变成：

```
Pod Packet

↓

VXLAN

↓

UDP

↓

IP

↓

Ethernet
```

即：

```
原始包

↓

再包一层
```

称为：

**封装（Encapsulation）**

例如：

```
Inner IP

10.244.1.2

↓

10.244.2.3
```

外面再包：

```
Outer IP

192.168.1.10

↓

192.168.1.20
```

于是：

物理网络只认识：

```
HostA

↓

HostB
```

HostB 收到后：

```
拆 VXLAN

↓

恢复原来的 Pod IP

↓

交给 Pod
```

------

# 八、为什么叫 Overlay？

因为：

它是在已有网络上：

再盖了一层网络。

```
真实网络：

192.168.x.x
```

Overlay：

```
10.244.x.x
```

物理交换机：

不知道 Pod。

它只认识：

```
Host
```

Overlay：

负责：

```
Host

↓

Pod
```

映射。

------

# 九、Kubernetes 为什么需要 CNI？

Docker：

负责：

```
Container
```

Kubernetes：

负责：

```
Pod
```

但是：

K8s 不会创建网络。

于是：

交给：

```
CNI
```

例如：

- Flannel
- Calico
- Cilium

职责：

```
创建 veth

↓

配置 IP

↓

配置路由

↓

配置 VXLAN/BGP/eBPF

↓

保证 Pod 通
```

------

# 十、几种典型实现

| 方案           | 实现方式           | 特点               |
| -------------- | ------------------ | ------------------ |
| Bridge         | Linux Bridge       | 仅单机             |
| Host Network   | 直接使用宿主机网卡 | 性能高，无隔离     |
| Overlay(VXLAN) | 封装 UDP           | 最常见             |
| BGP Routing    | 直接路由           | 性能高，无封装     |
| eBPF           | 内核转发           | 性能最好，现代方案 |

------

# 十一、三种主流 CNI 对比

## Flannel

```
Pod

↓

VXLAN

↓

Host
```

特点：

- 最简单
- Overlay
- 易部署

------

## Calico

默认：

```
BGP

HostA

↓

HostB
```

无需 VXLAN。

也支持：

```
VXLAN
```

特点：

- 网络策略强
- 企业最常见

------

## Cilium

没有：

```
iptables
```

直接：

```
eBPF

↓

Kernel
```

实现：

- Routing
- NAT
- LoadBalance
- Policy

性能最高。

------

# 十二、Service 为什么还能访问？

例如：

```
Client

↓

Service IP

↓

iptables/IPVS/eBPF

↓

Pod
```

真正通信：

还是：

```
Pod

↓

Pod
```

Service：

只是：

虚拟 IP。

------

# 十三、最终形成完整脑图

```
Container

↓

Namespace

↓

veth

↓

Linux Bridge

↓

Host Routing

↓

CNI

↓

Overlay / BGP / eBPF

↓

Physical Network

↓

另一台 Host

↓

Routing

↓

Bridge

↓

veth

↓

Container
```

------

# 十四、一张图理解全部

```text
                Kubernetes Cluster

          ┌────────────────────────────────┐

HostA                                    HostB

+-------------------+              +-------------------+
|     Pod A         |              |      Pod B        |
|   10.244.1.2      |              |    10.244.2.3     |
+---------▲---------+              +---------▲---------+
          │ veth                               │ veth
+---------┴---------+              +----------┴--------+
| Linux Bridge(CNI) |              | Linux Bridge(CNI) |
+---------▲---------+              +----------▲--------+
          │                                 │
      Linux Routing                    Linux Routing
          │                                 │
       eth0(HostA)======================eth0(HostB)
              192.168.1.x     Physical Network
```

------

# 十五、面试/实践记忆口诀

> **单机靠 Bridge，跨机靠路由；Overlay 靠封装，BGP 靠宣告，eBPF 靠内核。**

再进一步浓缩为一条主线：

> **容器通信 = Namespace 隔离 + veth 接入 + Bridge 二层转发 + Routing 三层转发 + CNI 跨主机互联（VXLAN/BGP/eBPF） + 物理网络承载。**

只要始终围绕这条主线理解，你面对 Docker、Kubernetes、Flannel、Calico、Cilium 等不同技术时，就能迅速判断它们分别是在解决哪一层的问题，而不会把各种网络概念混在一起。