---
title: docker-k8s-deep-understandind
date: 2026-07-20 16:57:07
---

# 深入理解docker k8s

## 容器的本质是进程

![image-20260720165951727](docker-k8s-deep-understandind/image-20260720165951727.png)

![image-20260720170037586](docker-k8s-deep-understandind/image-20260720170037586.png)![image-20260720170129306](docker-k8s-deep-understandind/image-20260720170129306.png)

containerd-shim-runc-v2进程是 **containerd** 为管理**某个特定容器**而创建的一个“代理进程”。

它的核心职责是：**将容器进程与 containerd 守护进程解耦**，即使 containerd 自身重启或崩溃，容器本身也不会因此退出。

容器退出删除后，代理进程消失



```
-namespace moby 是 Docker 官方使用的默认命名空间，Docker 通过 containerd 管理容器时，所有容器都放在这个命名空间里。
如果你看到 k8s.io，说明是 Kubernetes 管理的容器；如果是 default，则是纯 containerd 创建的容器。

-id b65a3d534300...
这是容器的完整 ID。
对应于 docker inspect 里看到的容器 ID，通常 docker ps 只显示前 12 位短 ID。

-address /run/containerd/containerd.sock
这是 containerd 的 gRPC socket 地址。
shim 进程通过这个 socket 与 containerd 守护进程通信。
它用来上报容器状态、接收控制指令、转发日志等。
```



## NameSpace限制容器能够看到的世界

ls ps ifconfig在容器和宿主机中执行结果不一样 为什么？

![image-20260720172443777](docker-k8s-deep-understandind/image-20260720172443777.png)



![image-20260720172616913](docker-k8s-deep-understandind/image-20260720172616913.png)

![image-20260907155116425](docker-k8s-deep-understandind/image-20260907155116425.png)

从以上2张图中注意到vim进程（systemd的子进程）对应的ipc mnt net pid user uts都和宿主机的NS一样



![image-20260720173608186](docker-k8s-deep-understandind/image-20260720173608186.png)

而容器b1的ns只有user和宿主机一样，mnt、pid、net不一样，所以ls ps ifconfig结果也不一样。



![image-20260720174000062](docker-k8s-deep-understandind/image-20260720174000062.png)

![image-20260720174126612](docker-k8s-deep-understandind/image-20260720174126612.png)

## Cgroup限制容器能够使用的资源（CPU篇）



## Cgroup限制容器能够使用的资源（内存篇）



## overlay2文件系统

![image-20260720174751385](docker-k8s-deep-understandind/image-20260720174751385.png)

![image-20260720175037275](docker-k8s-deep-understandind/image-20260720175037275.png)

![image-20260720175303606](docker-k8s-deep-understandind/image-20260720175303606.png)

![image-20260720175608691](docker-k8s-deep-understandind/image-20260720175608691.png)

## 容器镜像解析

![image-20260720175703500](docker-k8s-deep-understandind/image-20260720175703500.png)

![image-20260720175915531](docker-k8s-deep-understandind/image-20260720175915531.png)

```
docker images
docker inspect b1
```

docker inspect centos:centos7.9.2009

![image-20260722153912483](docker-k8s-deep-understandind/image-20260722153912483.png)

![image-20260722154143223](docker-k8s-deep-understandind/image-20260722154143223.png)

centos是基础镜像，没有lower层

```
#Dockerfile
FROM centos:centos7.9.2009
COPY ./mem /bin/mem

#通过当前目录Dockerfile制作镜像
docker build -t mem:v1 .
```

![image-20260722154430302](docker-k8s-deep-understandind/image-20260722154430302.png)

docker inspect mem:v1

![image-20260722154610766](docker-k8s-deep-understandind/image-20260722154610766.png)

LowerDir就是原本centos镜像的upper层

UpperDir就是新增的mem文件

![image-20260722154737544](docker-k8s-deep-understandind/image-20260722154737544.png)

![image-20260722154857684](docker-k8s-deep-understandind/image-20260722154857684.png)

![image-20260722155103328](docker-k8s-deep-understandind/image-20260722155103328.png)

![image-20260722155146082](docker-k8s-deep-understandind/image-20260722155146082.png)

## 容器的文件系统

![image-20260722155418660](docker-k8s-deep-understandind/image-20260722155418660.png)

docker run -it --rm c1 centos:centos7.9.2009

docker inspect c1

![image-20260722161256645](docker-k8s-deep-understandind/image-20260722161256645.png)

![image-20260722162120822](docker-k8s-deep-understandind/image-20260722162120822.png)

![image-20260722162327393](docker-k8s-deep-understandind/image-20260722162327393.png)

![image-20260722162535614](docker-k8s-deep-understandind/image-20260722162535614.png)

## 容器卷

![image-20260722173127229](docker-k8s-deep-understandind/image-20260722173127229.png)

![image-20260722173417467](docker-k8s-deep-understandind/image-20260722173417467.png)

docker inspect volumeName

![image-20260722173841344](docker-k8s-deep-understandind/image-20260722173841344.png)

## 容器网络

![image-20260722190853353](docker-k8s-deep-understandind/image-20260722190853353.png)

```
 docker network
Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks

Run 'docker network COMMAND --help' for more information on a command.
```

核心作用

1. 实现**容器与容器**、**容器与主机**、**容器与外网**的数据通信

2. 隔离不同容器、不同服务的网络环境，避免端口、网段冲突

3. 支撑容器集群、服务编排的网络互通需求

核心本质：Docker通过**虚拟网桥、虚拟网卡、端口映射**实现容器网络虚拟化。

**5大原生网络模式（重点）**

执行 `docker network ls` 可查看所有原生网络，默认自带5种：

1. bridge 桥接网络（默认模式）

容器默认网络，Docker自动创建`docker0` 虚拟网桥(虚拟交换机)

![image-20260722193417364](docker-k8s-deep-understandind/image-20260722193417364.png)

```
brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.024292a09d1f       no              veth4b5dd97

docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        inet6 fe80::42:92ff:fea0:9d1f  prefixlen 64  scopeid 0x20<link>
        ether 02:42:92:a0:9d:1f  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 7  bytes 746 (746.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker inspect bridge
```

**通信规则**：

- 同一bridge网络内容器**可直接互通**（通过内网IP）
- 跨bridge网络容器**默认隔离不通**
- 容器可访问外网，外网无法直接访问容器（需端口映射）

**适用场景**：单机多容器、独立服务部署（最常用）

**启动命令**：默认无需指定，或 `docker run --network bridge`

2. host 主机网络

   ![image-20260722195902159](docker-k8s-deep-understandind/image-20260722195902159.png)

**核心特点**：**容器直接复用宿主机网卡、IP、端口**，无独立网络隔离

**优缺点**：

- 优点：网络性能最好，无需端口映射，外网可直接访问容器端口
- 缺点：丧失网络隔离性，容器端口与主机端口冲突风险高

**适用场景**：高性能需求（监控、日志收集）、需要占用主机端口的服务

**启动命令**：`docker run --network host`

3. none 无网络模式

   ![image-20260722195932201](docker-k8s-deep-understandind/image-20260722195932201.png)

**核心特点**：关闭容器所有网络功能，容器无网卡、无IP、无法联网

**适用场景**：纯本地运行、无需任何网络通信的隔离型容器（安全测试、离线任务）

**启动命令**：`docker run --network none`

4. container 容器互联网络

**核心特点**：新容器**复用已有容器的网络栈**，共享IP、端口、网络命名空间

**关键点**：两个容器进程相互独立，但网络完全打通，localhost可互相访问

**适用场景**：容器配套辅助服务（如日志 sidecar、监控探针）

**启动命令**：`docker run --network container:目标容器名/ID`

5. overlay 覆盖网络（集群专用）

**核心特点**：**跨主机网络**，打通不同服务器上的Docker容器，实现集群互通

**适用场景**：Docker Swarm、K8s集群、多主机容器通信

**核心优势**：无视宿主机网段，集群内容器可通过容器名直接通信

![image-20260722200117281](docker-k8s-deep-understandind/image-20260722200117281.png)

## Veth

veth pair 就是一根**虚拟交叉网线**，两端各有一个“水晶头”。你把这根线的两头分别插到两个网络命名空间的“网卡槽位”上，它们就能直接通信，不需要经过任何物理设备。

![image-20260722200701857](docker-k8s-deep-understandind/image-20260722200701857.png)

```
案例：使用veth将network==none的busybox容器，接入docker0 bridge，连通后可访问bridge上的容器和外网

创建容器b1,无网络
#docker run -it --rm --name bl --network=none busybox

将docker存放netns信息的目录软连接到宿主机的存放netns的目录，这样ip netns命令就可以获得容器netns信息
#ln -s /var/run/docker/netns /var/run/netns

创建veth0,veth1的veth设备对
#ip link add veth0 type veth peer name veth1

获得容器sandboxkey
#docker inspect b1 grep -i sandboxkey"SandboxKey": "/var/run/docker/netns/8f28329cfec5"

将veth0放入b1的net ns
#ip link set veth0 netns 8f28329cfec5

将veth0改名为eth0
#ip netns exec 8f28329cfec5 ip link set dev veth0 name eth0

设置eth0的IP地址，子网掩码
#ip netns exec 8f28329cfec5 ip addr add 172.17.0.10/24 dev eth0

设置eth0接口up
#ip netns exec 8f28329cfec5 ip link set eth0 up

设置默认路由
#ip netns exec 8f28329cfec5 route add default gw 172.17.0.1 eth0

将veth1连接到docker0 bridge
# ip link set dev vethl master docker0 

设置veth1接口up
# ip link set veth1 up
```

此时容器b1内可ping通百度和brige上的其他容器



## 容器访问外网

![image-20260727204617481](docker-k8s-deep-understandind/image-20260727204617481.png)

![image-20260729093827904](docker-k8s-deep-understandind/image-20260729093827904.png) 

![image-20260907205145558](docker-k8s-deep-understandind/image-20260907205145558.png)

![image-20260727200124360](docker-k8s-deep-understandind/image-20260727200124360.png)

```
第一条：default via 172.17.0.1 dev eth0
含义：默认路由。
作用：当要访问的目标 IP 地址不在其他更具体的路由规则中时，所有流量都通过 eth0 网络接口发送给网关 172.17.0.1。
通俗理解：相当于“凡是没指明去哪儿的网络请求，统一走这个出口”。
典型场景：172.17.0.1 通常是 Docker 宿主机的 docker0 网桥地址，这条路由让容器能访问外网（如互联网）。

第二条：172.17.0.0/16 dev eth0 scope link  src 172.17.0.2
含义：直连路由（链路本地路由）。
作用：目标地址属于 172.17.0.0/16 这个网段（即 172.17.0.0 ~ 172.17.255.255）的数据包，直接通过 eth0 接口发送，无需经过网关。
scope link 表示这条路由只在本地链路有效。
src 172.17.0.2 表示从这个接口发出的数据包，源 IP 地址默认为 172.17.0.2（即本机 IP）。
通俗理解：“同一网段的邻居可以直接打招呼，不用找网关。”
典型场景：这台机器的 IP 是 172.17.0.2，子网掩码为 255.255.0.0，所以它能直接与同网段（如 172.17.0.3、172.17.1.5 等）通信。
```



## 外网访问容器

docker run --name n1  -p 80:80 nginx 

**curl和浏览器的区别**

curl完整请求

![image-20260729095559126](docker-k8s-deep-understandind/image-20260729095559126.png)

浏览器会有缓存

![image-20260729095438944](docker-k8s-deep-understandind/image-20260729095438944.png)

状态代码 `304 Not Modified`

这是核心点，**不是错误**，而是一种优化机制：

- 浏览器第一次访问 `192.168.101.42/` 时，服务器返回 `200 OK` 并带上资源内容，同时可能附带**缓存标识**（如 `Last-Modified` 时间或 `ETag`）。
- 当浏览器**再次请求**同一资源时，会自动带上条件请求头（`If-Modified-Since` 或 `If-None-Match`），相当于问服务器：
  **“我这有缓存，你的内容和我上次拿到的一样吗？”**
- 服务器检查后，如果资源**没有变化**，就直接返回 `304 Not Modified`，**不再重复传输资源内容**。
- 浏览器收到 304，就从本地缓存中取出内容来用。

**好处**：节省带宽，加快页面加载速度。所以你看到 304，说明服务器上的内容没更新，浏览器聪明地复用了缓存。



容器缺命令，宿主机执行命令到容器中

```
[root@worker1 ~]# docker inspect n1 | grep -i sandboxkey
            "SandboxKey": "/var/run/docker/netns/0f7bbff5ab2b",
[root@worker1 ~]# ip netns exec 0f7bbff5ab2b netstat -anutpl
Cannot open network namespace "0f7bbff5ab2b": No such file or directory
[root@worker1 ~]# ln -s /var/run/docker/netns /var/run/netns
[root@worker1 ~]# ip netns exec 0f7bbff5ab2b netstat -anutpl
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      5399/nginx: master
tcp6       0      0 :::80                   :::*                    LISTEN      5399/nginx: master

ll /proc/5399//ns/net
lrwxrwxrwx 1 root root 0 7月  29 10:16 /proc/5399//ns/net -> net:[4026532546]

[root@worker1 ~]# netstat -anutpl|grep :80
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      5353/docker-proxy
tcp6       0      0 :::80                   :::*                    LISTEN      5359/docker-proxy

```

以上操作，本质上是**让 `ip netns` 工具能够“看见”Docker 创建的网络命名空间**。

------

1. 核心概念：网络命名空间

Linux 内核提供了**网络命名空间（Network Namespace）**的功能，可以让不同进程拥有完全隔离的网络栈（独立网卡、IP、路由、iptables 等）。

- **Docker** 用它来实现容器间的网络隔离，每个容器（如果你没使用 host 网络模式）都运行在一个独立的网络命名空间里。
- **`ip netns`** 是 Linux 原生的网络命名空间管理工具，可以用来查看、操作这些命名空间。

------

2. 矛盾点：存放位置不同

问题就出在 **`ip netns` 查找命名空间的默认路径** 上：

- **`ip netns`** 认为所有可由它管理的命名空间，都应该在 **`/var/run/netns/`** 目录下有一个对应的文件。这些文件通常是通过 `ip netns add` 命令创建的 bind mount 文件。
- **Docker** 创建容器时，直接将网络命名空间的文件描述符放在了它自己的目录 **`/var/run/docker/netns/`** 下。

这就导致：

- `docker inspect n1 | grep SandboxKey` 能找到 Docker 存放的命名空间文件 `/var/run/docker/netns/0f7bbff5ab2b`。
- 但 `ip netns exec 0f7bbff5ab2b` 去 `/var/run/netns/` 下找，根本找不到这个文件，所以报错：
  `Cannot open network namespace "0f7bbff5ab2b": No such file or directory`

------

3. 破局操作：符号链接

你执行的命令：

bash

```
ln -s /var/run/docker/netns /var/run/netns
```



创建了一个符号链接：**`/var/run/netns` → `/var/run/docker/netns`**

这样做之后：

- `ip netns` 再去 `/var/run/netns/` 下找文件时，实际上被重定向到了 Docker 的目录。
- 于是它成功发现了 `0f7bbff5ab2b` 这个文件，并把它当作一个合法命名空间来处理。
- 后续 `ip netns exec 0f7bbff5ab2b netstat -anutpl` 就能进入容器 `n1` 的网络空间，看到 nginx 在监听 80 端口。

------

4. 本质总结

**本质就是：**
Linux 内核中的网络命名空间只是一个抽象资源，`/var/run/netns/` 只是 `ip netns` 工具用来**索引和引用**这些命名空间的约定路径。
Docker 把它放到了自己的目录下，只要让两个路径统一（这里是让 `ip netns` 的路径指向 Docker 的路径），`ip netns` 就能识别和管理 Docker 创建的命名空间了。

更底层的实质是：那个文件（如 `0f7bbff5ab2b`）其实是一个**指向 `/proc/<pid>/ns/net` 的 bind mount**，它只是内核命名空间对象的入口。符号链接解决的纯粹是**用户空间工具的路径匹配问题**。



![image-20260729110450446](docker-k8s-deep-understandind/image-20260729110450446.png)

![image-20260729110643399](docker-k8s-deep-understandind/image-20260729110643399.png)

```
 docker inspect n1 |grep box
            "SandboxID": "0f7bbff5ab2b6a26b187c28880a415a7d378de171bd629ed7194bd74af7b9edd",
            "SandboxKey": "/var/run/docker/netns/0f7bbff5ab2b",
            
 ip netns exec 0f7bbff5ab2b ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
9: eth0@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
       
[root@worker1 ~]# ip addr|grep 10:
10: veth352b066@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default

tcpdump -i veth352b066 -w veth352b066.pcap
tcpdump -i ens33 -w ens33.pcap
然后通过curl 192.168.101.42访问nginx

通过wireshark打开pcap文件 过滤http

```

![image-20260729112009085](docker-k8s-deep-understandind/image-20260729112009085.png)

![image-20260729112032704](docker-k8s-deep-understandind/image-20260729112032704.png)  

![image-20260729111933772](docker-k8s-deep-understandind/image-20260729111933772.png)

docker stop n1后80端口不再监听 iptables的规则也被删除。



## 同一宿主机内的容器互访

![image-20260729112546930](docker-k8s-deep-understandind/image-20260729112546930.png)

容器2、3的/etc/resolve.conf和容器1（同宿主机的resolve.conf）不一样

[root@c7c8525d4c48 /]# cat /etc/resolv.conf

nameserver 127.0.0.11
options ndots:0



## 不同宿主机内的容器互访 

**本质是：如何让不同机器上的虚拟网卡能够互相通信。**

详细可参考：不同宿主机容器互访（Docker/Kubernetes）系统性笔记.md

2台机器的容器不能同时连接一个虚拟网卡，通过docker swarm部署docker集群

docker info可以查看集群信息，判断当前主机为管理节点还是工作节点

docker network ls

![image-20260730093128409](docker-k8s-deep-understandind/image-20260730093128409.png)

docker service create 会将容器默认接入ingress

--attachable 使用docker run运行的容器可通过--network加入该网络，不然只能通过docker service的方式

![image-20260729173826686](docker-k8s-deep-understandind/image-20260729173826686.png)

![image-20260729163426270](docker-k8s-deep-understandind/image-20260729163426270.png)

```
宿主机 A  (物理 IP: 192.168.1.10)                宿主机 B  (物理 IP: 192.168.1.20)
┌──────────────────────────────────┐            ┌──────────────────────────────────┐
│  Docker Swarm (Overlay 网络)      │            │  Docker Swarm (Overlay 网络)      │
│                                  │            │                                  │
│  ┌──────────────┐                │            │                ┌──────────────┐  │
│  │ 容器 A1      │  原始二层帧      │            │  原始二层帧      │ 容器 B1      │  │
│  │ IP:10.0.0.2 │  src:10.0.0.2  │            │  src:10.0.0.2  │ IP:10.0.0.3 │  │
│  │              │  dst:10.0.0.3  │            │  dst:10.0.0.3  │              │  │
│  └──────┬───────┘                │            │                └──────▲───────┘  │
│         │ 发出                    │            │                       │ 交给      │
│         ▼                        │            │                       │          │
│  ┌──────────────────────┐        │            │  ┌──────────────────────┐        │
│  │  VXLAN 封装 (VTEP)    │        │            │  │  VXLAN 解封 (VTEP)    │        │
│  │  外层 IP/UDP 头:       │        │            │  │  去掉外层头，恢复原始帧  │        │
│  │  Src: 192.168.1.10    │        │            │  │                      │        │
│  │  Dst: 192.168.1.20    │        │            │  └──────────────────────┘        │
│  │  UDP Dst Port: 4789   │        │            │                                  │
│  └──────────┬───────────┘        │            │                                  │
└─────────────┼────────────────────┘            └──────────────────────────────────┘
              │  封装后的 UDP 数据包
              │  (通过物理网络传输)
              │
   ┌──────────▼──────────────────────────────────────────┐
   │              物理网络 (Underlay)                      │
   │                                                     │
   │  UDP 包: 192.168.1.10:xxxx → 192.168.1.20:4789     │
   └─────────────────────────────────────────────────────┘
```

**图解要点：**

1. **容器** 只知道虚拟 IP (`10.0.0.x`)，完全感知不到底层物理网络。
2. **VTEP** (VXLAN Tunnel Endpoint) 负责将原始二层帧封装成 UDP 包，目标端口 **4789**。
3. 物理网络只看到宿主机 IP 之间的 UDP 通信，对 Overlay 网络内部完全透明。
4. 目标宿主机解封后，直接把原始帧交给对应容器，跨主机容器就像连在同一个交换机上。



## docker swarm service

service 是运行在docker集群模式下，使用同一个镜像创建的，一个或多个容器组成的集合。Service是对这些容器的一个抽象或封装。访问集群任何一个节点，都可以访问到这个service。
service可以一键动态的扩缩容，对一个service创建多个副本

```
docker service create --name mynginx --replicas 3 -p 80:80 nginx
```

![image-20260730102157055](docker-k8s-deep-understandind/image-20260730102157055.png)

![image-20260730105417418](docker-k8s-deep-understandind/image-20260730105417418.png)

![image-20260730105310990](docker-k8s-deep-understandind/image-20260730105310990.png)

```
nsenter --net=/var/run/docker/netns/ingress_sbox ipvsadm --list
```

![image-20260730134754244](docker-k8s-deep-understandind/image-20260730134754244.png)

![image-20260730140253870](docker-k8s-deep-understandind/image-20260730140253870.png)

![image-20260730140413172](docker-k8s-deep-understandind/image-20260730140413172.png)

docker network inspect docker_gwbridge

![image-20260730140736256](docker-k8s-deep-understandind/image-20260730140736256.png)

![image-20260730141154426](docker-k8s-deep-understandind/image-20260730141154426.png)

![image-20260730141258739](docker-k8s-deep-understandind/image-20260730141258739.png)

![image-20260730141339275](docker-k8s-deep-understandind/image-20260730141339275.png) 

![image-20260730145823242](docker-k8s-deep-understandind/image-20260730145823242.png)

![image-20260730145857831](docker-k8s-deep-understandind/image-20260730145857831.png)

源ip从192.168.158.100转换成10.0.0.2

![image-20260730150318226](docker-k8s-deep-understandind/image-20260730150318226.png)

![image-20260730150606201](docker-k8s-deep-understandind/image-20260730150606201.png)

![image-20260730150454314](docker-k8s-deep-understandind/image-20260730150454314.png)

容器内redirect将8000端口转换成80

![image-20260730150506011](docker-k8s-deep-understandind/image-20260730150506011.png)



![image-20260730151251668](docker-k8s-deep-understandind/image-20260730151251668.png)

## kubernetes集群的启动过程

![image-20260730153252959](docker-k8s-deep-understandind/image-20260730153252959.png)

**控制平面节点组件：**

- **API Server**（提供 Kubernetes API 接口，处理所有 REST 请求，作为集群的统一入口）
- **etcd**（分布式键值存储，保存集群所有资源对象的持久化数据）
- **Scheduler**（监听未调度的 Pod，根据策略将其绑定到合适的 Node）
- **Controller Manager**（运行各种控制器，如 Node Controller、Replication Controller 等，确保集群实际状态与期望状态一致）

**Node 节点（工作节点）组件：**

- **kubelet**（管理节点上 Pod 的生命周期，与 API Server 交互，确保容器按预期运行）
- **kube-proxy**（维护节点上的网络规则，实现 Service 的负载均衡与流量转发）
- **容器运行时**（如 containerd / Docker / CRI-O，负责拉取镜像、创建和运行容器）

![image-20260801143006219](docker-k8s-deep-understandind/image-20260801143006219.png)    

kubelet直接先启动核心组件



![image-20260801143301666](docker-k8s-deep-understandind/image-20260801143301666.png)

## 什么是Pod

![image-20260801143720565](docker-k8s-deep-understandind/image-20260801143720565.png)![image-20260801143739130](docker-k8s-deep-understandind/image-20260801143739130.png)

![image-20260801143821126](docker-k8s-deep-understandind/image-20260801143821126.png)

## Pause容器

![image-20260801150234710](docker-k8s-deep-understandind/image-20260801150234710.png)

 我们创建一个：

> **Nginx 主容器 + 日志 Sidecar 容器 + Init Container 初始化页面**

结构

```
Pod
│
├── initContainer
│       └── 初始化共享目录 index.html
│
├── nginx-container        (主业务容器)
│       └── 提供 HTTP 服务 :80
│
└── log-container          (sidecar)
        └── 实时读取 nginx 日志
        

共享：
├── Network Namespace
│       ├── nginx:80
│       └── sidecar
│
└── Volume
        └── /data
```

1.创建pod-demo.yaml

```
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-demo
  labels:
    app: demo
spec:
  nodeName: worker1
  # 初始化容器
  initContainers:
  - name: init-page
    image: busybox
    imagePullPolicy: IfNotPresent
    command:
    - sh
    - -c
    - |
      echo "hello from initContainer" > /data/index.html
    volumeMounts:
    - name: shared-data
      mountPath: /data

  containers:
  # 主容器
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-data
      mountPath: /usr/share/nginx/html

    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5
      periodSeconds: 5
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 500m
        memory: 256Mi

  # sidecar
  - name: log-sidecar
    image: busybox
    imagePullPolicy: IfNotPresent
    command:
    - sh
    - -c
    - |
      tail -f /var/log/nginx/access.log
    volumeMounts:
    - name: nginx-log
      mountPath: /var/log/nginx
  volumes:
  - name: shared-data
    emptyDir: {}
  - name: nginx-log
    emptyDir: {}
```

创建了一个固定运行在 worker1 节点上的多容器 Pod：先用 initContainer 初始化共享网页文件，然后由 nginx 容器提供 Web 服务，同时通过共享 Volume 让 sidecar 容器读取 nginx 日志，实现“初始化 + 主业务 + 辅助日志采集”的 Pod 协作模式



```
docker ps |grep pause
18a8073d9a4a   kubesphere/pause:3.9     "/pause"                  23 hours ago   Up 23 hours             k8s_POD_multi-container-demo_default_b9cff324-531a-4ba7-8ad6-793145c3c7be_0
k8s_<container_name>_<pod-name>_<namespace>_<pod-uid>_<restart-count> 
     POD就代表pause容器名

k get po -n default multi-container-demo -o jsonpath='{.metadata.uid}'
b9cff324-531a-4ba7-8ad6-793145c3c7be

docker ps |grep multi
CONTAINER ID   IMAGE                    COMMAND                   CREATED        STATUS        PORTS     NAMES
291ebecae25d   busybox                  "sh -c 'touch /var/l…"   23 hours ago   Up 23 hours             k8s_log-sidecar_multi-container-demo_default_b9cff324-531a-4ba7-8ad6-793145c3c7be_0
041c951795db   nginx                    "sh -c 'cat >/etc/ng…"   23 hours ago   Up 23 hours             k8s_nginx_multi-container-demo_default_b9cff324-531a-4ba7-8ad6-793145c3c7be_0
18a8073d9a4a   kubesphere/pause:3.9     "/pause"                  23 hours ago   Up 23 hours           k8s_POD_multi-container-demo_default_b9cff324-531a-4ba7-8ad6-793145c3c7be_0

```

![image-20260811151326690](docker-k8s-deep-understandind/image-20260811151326690.png)

## 创建pod

```
 命令行方式：
 kubectl run nginx --image=nginx
 kubectl run -i -t busybox --image=busybox --restart=Never --image-pull-policy=IfNotPresent
 kubectl delete pod nginx
 
 yaml文件方式（推荐）
 kubectl apply -f busybox.yaml
 
#删除pod
kubectl delete -f nginx.yaml

```

![image-20260811151958156](docker-k8s-deep-understandind/image-20260811151958156.png)

![image-20260811151922415](docker-k8s-deep-understandind/image-20260811151922415.png)

![image-20260813152017194](docker-k8s-deep-understandind/image-20260813152017194.png)

## Namespace

![image-20260813152057518](docker-k8s-deep-understandind/image-20260813152057518.png)

![image-20260813152303030](docker-k8s-deep-understandind/image-20260813152303030.png)

![image-20260813152613470](docker-k8s-deep-understandind/image-20260813152613470.png)

## emptyDir volume

![image-20260814093942255](docker-k8s-deep-understandind/image-20260814093942255.png)![image-20260814110919398](docker-k8s-deep-understandind/image-20260814110919398.png)![image-20260814111026156](docker-k8s-deep-understandind/image-20260814111026156.png)

![image-20260814121515291](docker-k8s-deep-understandind/image-20260814121515291.png)

## hostPath volume

![image-20260814121620068](docker-k8s-deep-understandind/image-20260814121620068.png)

## pv and pvc

![image-20260814121901228](docker-k8s-deep-understandind/image-20260814121901228.png)![image-20260814121957303](docker-k8s-deep-understandind/image-20260814121957303.png)
