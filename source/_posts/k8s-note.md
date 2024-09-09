---
title: k8s-note
date: 2024-05-23 15:29:16
---

k8s

[Kubernetes](https://v1-27.docs.kubernetes.io/docs/concepts/overview/), also known as K8s, is an open-source system for **automating deployment, scaling, and management of containerized application**

It groups containers that make up an application into logical units for easy management and discovery. Kubernetes builds upon [15 years of experience of running production workloads at Google](http://queue.acm.org/detail.cfm?id=2898444), combined with best-of-breed ideas and practices from the community



## 部署方式

1. 传统的**服务进程**    手工繁琐，需使用自动化工具 ansible/saltstack/shell
2. 容器应用管理     应用功能状态最大化封装

![image-20240412193454226](k8s-note/image-20240412193454226.png?lastModify=1716449400)



​												传统二进制	  																					kubeadm



![image-20240412193615615](k8s-note/image-20240412193615615.png?lastModify=1716449400)				..

![image-20240412193834338](k8s-note/image-20240412193834338.png?lastModify=1716449400)



![image-20240412193922752](k8s-note/image-20240412193922752.png?lastModify=1716449400)

![image-20240416230722588](k8s-note/image-20240416230722588.png?lastModify=1716449400)

![image-20240416230934803](k8s-note/image-20240416230934803.png?lastModify=1716449400)

pod创建方式及流程

**命令行**      kubectl run my-pod --image=nginx --restart=Never 

**yaml**   kubectl apply -f my-pod.yaml

```yaml
    #my-pod.yaml
     apiVersion: v1
     kind: Pod
     metadata:
       name: my-pod
     spec:
       containers:
       - name: nginx-container
         image: nginx
     
```

**Kubernetes Pod创建流程图**

1. 用户提交Pod配置文件（YAML）
   - 使用`kubectl create`或`kubectl apply`命令

2. API Server接收请求
   - 验证请求并授权
   - 将Pod配置信息存储到etcd数据库

3. Scheduler调度Pod
   - 根据资源需求和集群状态选择合适的节点
   - 将Pod绑定到选定的节点

4. Kubelet执行Pod创建
   - 在目标节点上创建Pod目录
   - 拉取Pod所需的镜像
   - 启动容器并监控其状态

5. 容器运行时（如Docker）启动容器
   - 解析镜像并创建容器
   - 启动容器进程

6. Pod状态更新
   - Kubelet定期向API Server报告Pod状态
   - API Server更新etcd数据库中的Pod信息

7. 用户查看Pod状态
   - 使用`kubectl get pods`命令查看Pod列表
   - 使用`kubectl describe pod <pod_name>`查看Pod详细信息





主机名规划

| 序号 | 主机ip          | 主机名规划                           |
| ---- | --------------- | ------------------------------------ |
| 1    | 192.168.101.120 | k8s-master.sswang.com k8s-master     |
| 2    | 192.168.101.121 | k8s-node1.sswang.com k8s-node1       |
| 3    | 192.168.101.122 | k8s-node2.sswang.com k8s-node2       |
| 4    | 192.168.101.123 | k8s-node3.sswang.com k8s-node3       |
| 5    | 192.168.101.124 | k8s-register.sswang.com k8s-register |

跨主机免密码认证

```
生成秘钥对
ssh-keygen -t rsa 

跨主机免密码认证
ssh-copy-id root@192.168.101.121
ssh-copy-id root@192.168.101.122
ssh-copy-id root@192.168.101.123
ssh-copy-id root@192.168.101.124
```

master安装ansible

```shell
yum install -y epel-release ansible

vim /etc/ansible/hosts
#添加test group
[test]
192.168.101.121
192.168.101.122
192.168.101.123
192.168.101.124

echo '192.168.101.120 k8s-master.sswang.com k8s-master
192.168.101.121 k8s-node1.sswang.com k8s-node1
192.168.101.122 k8s-node2.sswang.com k8s-node2
192.168.101.123 k8s-node3.sswang.com k8s-node3
192.168.101.124 k8s-register.sswang.com k8s-register'  >> /etc/hosts

echo "192.168.101.120 k8s-master.sswang.com k8s-master
192.168.101.121 k8s-node1.sswang.com k8s-node1
192.168.101.122 k8s-node2.sswang.com k8s-node2
192.168.101.123 k8s-node3.sswang.com k8s-node3
192.168.101.124 k8s-register.sswang.com k8s-register" |  tee -a /etc/hosts # tee将字符串写入指定的文件 -a append 

ansible test -m copy   -a "src=/etc/hosts dest=/etc/hosts"
ansible test -m shell  -a "reboot"
```



```sh
#临时禁用swap
swapoff -a

#内核参数调整
cat >> /etc/sysctl.d/k8s.conf << EOF
vm.swappiness=0
EOF

#网络参数调整
cat >> /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

#配置生效
modprobe br_netfilter
modprobe overlay

sysctl -p /etc/sysctl.d/k8s.conf

#将/etc/sysctl.d/k8s.conf传输到test集群节点
ansible test -m copy -a  "src=/etc/sysctl.d/k8s.conf  dest=/etc/sysctl.d/k8s.conf"
ansible test -m shell  -a "modprobe br_netfilter"
ansible test -m shell  -a "modprobe overlay"
ansible test -m shell  -a "sysctl -p /etc/sysctl.d/k8s.conf"
```





docker安装

```sh
#设置Docker的安装环境：首先安装yum-utils工具，然后通过yum-config-manager添加Docker的官方仓库
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

#To install a specific version, start by listing the available versions in the repository
yum list docker-ce --showduplicates | sort -r


#指定版本 <tag>替换为对应版本
#sudo yum install docker-ce-<tag> docker-ce-cli-<tag> containerd.io docker-buildx-plugin docker-compose-plugin
yum install docker-ce-20.10.9 docker-ce-cli-20.10.9 containerd.io docker-buildx-plugin docker-compose-plugin -y 

#最新版
yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

#配置阿里云镜像源 除了安装harbor的机器都执行
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "http://74f21445.m.daocloud.io",
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn"
  ],
  "insecure-registries": [
    "k8s-register.sswang.com"
  ],
  "exec-opts": [
    "native.cgroupdriver=systemd"
  ]
}
EOF

systemctl start docker
systemctl enable docker
```



cri部署

```sh
#方式一 rpm 本次采用
mkdir /data/softs -p && cd /data/softs
rpm -ivh cri-dockerd-0.3.12-3.el7.x86_64.rpm

vim /usr/lib/systemd/system/cri-docker.service
#加上--pod-infra-container-image=k8s-register.sswang.com/google_containers/pause:3.9（需确保可以访问harbor）否则无法创建pod 或者加上registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.9
ExecStart=/usr/bin/cri-dockerd --pod-infra-container-image=k8s-register.sswang.com/google_containers/pause:3.9


#启动
systemctl daemon-reload && systemctl enable cri-docker.socket
systemctl start cri-docker.socket cri-docker.service


/usr/lib/systemd/system/cri-docker.socket
```



方式二

```shell
mkdir /data/softs -p && cd /data/softs
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.2/cri-dockerd-0.3.2.amd64.tgz

解压软件
tar xf cri-dockerd-0.3.2.amd64.tgz
mv cri-dockerd/cri-dockerd /usr/local/bin/

检查效果
cri-dockerd --version
```



vim /etc/systemd/system/cri-dockerd.service

```shell
[Unit]
Description=CRI Interface for Docker Application Container Engine
Documentation=https://docs.mirantis.com
After=network-online.target firewalld.service docker.service
Wants=network-online.target

[Service]
Type=notify
ExecStart=/usr/local/bin/cri-dockerd --pod-infra-container-image=k8s-register.sswang.com/google_containers/pause:3.9 --network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin --container-runtime-endpoint=unix:///var/run/cri-dockerd.sock --docker-endpoint=unix:///var/run/docker.sock --cri-dockerd-root-directory=/var/lib/docker
ExecReload=/bin/kill -s HUP
TimeoutSec=0
RestartSec=2
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
KillMode=process

[Install]
WantedBy=multi-user.target
cat > /etc/systemd/system/cri-dockerd.socket <<-EOF
[Unit]
Description=CRI Docker Socket for the API
PartOf=cri-docker.service

[Socket]
ListenStream=/var/run/cri-dockerd.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target
EOF
```





harbor构建 (仅在124上执行)

```sh
# docker compose install
curl -SL https://github.com/docker/compose/releases/download/v2.17.2/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose


mkdir /data/{softs,server} -p && cd /data/softs 
wget https://github.com/goharbor/harbor/releases/download/v2.5.0/harbor-offline-installer-v2.5.0.tgz 
tar -zxvf harbor-offline-installer-v2.5.0.tgz -C /data/server/ 
cd /data/server/harbor/ 
docker load < harbor.v2.5.0.tar.gz 
docker images 
#备份配置 
cp harbor.yml.tmpl harbor.yml
#修改配置
[root@kubernetes-register /data/server/harbor]# vim harbor.yml

# 修改主机名
hostname: k8s-register.sswang.com
http:
  port: 80
#https:  注释ssl相关的部分
  #  port: 443
  #  certificate: /your/certificate/path
  #  private_key: /your/private/key/path
# 默认harbor的登录密码
harbor_admin_password:  Harbor12345

# 设定harbor的数据存储目录
data_volume: /data/server/harbor/data

#配置harbor 
./prepare 

#启动harbor 
./install.sh 

#检查效果
docker-compose ps
```





以下harbor服务暂未使用，需要请自行使用

```sh
vim /etc/systemd/system/harbor.service
[Unit]
Description=Harbor
After=docker.service systemd-networkd.service systemd-resolved.service
Requires=docker.service
Documentation=http://github.com/vmware/harbor

[Service]
Type=simple
Restart=on-failure
RestartSec=5
#需要注意harbor的安装位置
ExecStart=/usr/local/bin/docker-compose --file /data/server/harbor/docker-compose.yml up
ExecStop=/usr/local/bin/docker-compose --file /data/server/harbor/docker-compose.yml down

[Install]
WantedBy=multi-user.target


加载服务配置文件
systemctl daemon-reload
该命令 只会 重新加载配置文件，而不会 重启或停止正在运行的服务。这意味着，如果你更改了服务的配置文件，你需要使用其他命令（如 systemctl restart<service>）来重启服务，以使新的配置生效。

启动服务
systemctl start harbor
检查状态
systemctl status harbor
设置开机自启动
systemctl enable harbor
```





harbor仓库测试

```sh
harbor 创建 用户vj 
用户vj  创建 项目k8s为公开项目

登录仓库
docker login k8s-register.sswang.com -u vj
Password:   # 输入登录密码 159357qQ！

下载镜像
docker pull busybox

定制镜像标签
docker tag busybox k8s-register.sswang.com/vj/busybox:v0.1
docker tag busybox harbor域名    			 /仓库/镜像:tag

推送镜像
docker push kubernetes-register.sswang.com/vj/busybox:v0.1

从集群中的其他节点上拉取镜像
docker pull kubernetes-register.sswang.com/vj/busybox:v0.1
```







![image-20240422155216084](k8s-note/image-20240422155216084.png?lastModify=1716449400)





Installing kubeadm

https://v1-27.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

K8s集群初始化

**软件源定制-安装软件--镜像获取--主节点初始-工作节点加入集群**

```sh
#软件源定制
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

#更新软件源
yum makecache fast
```



![image-20240419103257266](k8s-note/image-20240419103257266.png?lastModify=1716449400)

```sh
# 安装软件
#所有节点安装 kubeadm kubelet kubectl
yum install -y kubeadm kubelet kubectl
yum remove  -y kubeadm kubelet kubectl

kubeadm version
kubelet --version
kubectl version

#检查镜像文件列表 k8s运行所需镜像 
kubeadm config images list

#镜像获取 只在master执行
vim get-kubeadm-images.sh
images=$(kubeadm config images list | awk -F "/" '{print $NF}')
for i in ${images}
do
  docker pull registry.aliyuncs.com/google_containers/$i
  docker tag registry.aliyuncs.com/google_containers/$i k8s-register.sswang.com/google_containers/$i
  docker push k8s-register.sswang.com/google_containers/$i
  docker rmi registry.aliyuncs.com/google_containers/$i
done


chmod +x get-kubeadm-images.sh
./get-kubeadm-images.sh

docker images
```





master节点初始化

```
kubeadm init --apiserver-advertise-address=192.168.101.120 \
      --image-repository=k8s-register.sswang.com/google_containers \
      --service-cidr=10.96.0.0/12 \
      --pod-network-cidr=10.244.0.0/16 \
      --ignore-preflight-errors=Swap \
      --cri-socket=unix:///var/run/cri-dockerd.sock \
      --v=5
```

node节点加入集群

```sh
# Then you can join any number of worker nodes by running the following on each as root:
kubeadm join 192.168.101.120:6443 --token q59806.z4krgs4h1yp13qjr --discovery-token-ca-cert-hash sha256:b4051b7389188882a46ed8e20dff4199adee93097d2a89641fba66eb3b2bbb6c --cri-socket=unix:///var/run/cri-dockerd.sock
```



```sh
#安装失败后可能用到的命令
yum install lsof -y
sudo kill $(sudo lsof -t -i :6443) $(sudo lsof -t -i :10259) $(sudo lsof -t -i :10257) $(sudo lsof -t -i :10250)&&$(sudo lsof -t -i :2380)&&$(sudo lsof -t -i :2379)&&
sudo rm /etc/kubernetes/manifests/kube-apiserver.yaml&&
sudo rm /etc/kubernetes/manifests/kube-controller-manager.yaml&&
sudo rm /etc/kubernetes/manifests/kube-scheduler.yaml&&
sudo rm /etc/kubernetes/manifests/etcd.yaml&&
sudo rm -rf /var/lib/etcd/*

%s/192.168.101.12/192.168.101.121/g
```



自动补全功能

```sh
yum -y install bash-completion
source /usr/share/bash-completion/bash_completion
echo 'source <(kubectl completion bash)' >>  ~/.bashrc
echo "source <(kubeadm completion bash)" >> ~/.bashrc 
source ~/.bashr
```



```sh
#Flannel是CoreOS团队针对Kubernetes设计的一个网络规划服务。主要功能是让集群中的不同节点主机创建的Docker容器都具有全集群唯一的虚拟IP地址。这样，即使容器在不同的主机上运行，它们之间也可以像在同一个网络中一样进行通信。
mkdir /data/kubernetes/flannel -p
cd /data/kubernetes/flannel/
wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
cp kube-flannel.yml kube-flannel.yml-bak

docker pull flannel/flannel:v0.25.1
docker pull flannel/flannel-cni-plugin:v1.4.0-flannel1

docker tag flannel/flannel:v0.25.1  k8s-register.sswang.com/vj/flannel:v0.25.1
docker tag flannel/flannel-cni-plugin:v1.4.0-flannel1 k8s-register.sswang.com/vj/flannel-cni-plugin:v1.4.0-flannel1

docker push k8s-register.sswang.com/vj/flannel:v0.25.1
docker push k8s-register.sswang.com/flannel/flannel:v0.25.1
vim kube-flannel.yml #修改镜像地址为harbor中的镜像
grep image: kube-flannel.yml
kubectl apply -f kube-flannel.yml
#The connection to the server localhost:8080 was refused - did you specify the right host or port?
ansible test -m copy -a "src=/etc/kubernetes/admin.conf dest=/etc/kubernetes/admin.conf"
ansible test -m command -a "echo 'export KUBECONFIG=/etc/kubernetes/admin.conf' >> ~/.bash_profile"
ansible test -m shell -a "source ~/.bash_profile"
systemctl is-active docker cri-docker kubelet
systemctl enable docker cri-docker kubelet
```





![image-20240422163711486](k8s-note/image-20240422163711486.png?lastModify=1716449400)

![image-20240422165134938](k8s-note/image-20240422165134938.png?lastModify=1716449400)

```
#service
kubectl expose deployment nginx-deployment --name=nginx-service --port=8080 --target-port=80
```

Service(服务)

![image-20240520104527406](k8s-note/image-20240520104527406.png?lastModify=1716449400)

Service将运行在一组 [Pods](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 上的应用程序 公开为 网络服务 的抽象方法。

Service为一组 Pod 提供相同的 DNS 名(service name），并且在它们之间进行负载均衡。

Kubernetes 为 Pod 分配了IP 地址，但IP地址可能会发生变化。

集群内的容器可以通过 service名称 访问服务，而不需要担心Pod的IP发生变化。

Kubernetes Service 定义了这样一种抽象：

逻辑上的一组可以互相替换的 Pod，通常称为微服务。 

Service 对应的 Pod 集合通常是通过[选择算符](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/)来确定的。 

举个例子，在一个Service中运行了3个nginx的副本。这些副本是可互换的，我们不需要关心它们调用了哪个nginx，也不需要关注 Pod的运行状态，只需要调用这个服务就可以了。



创建Service对象



Service_type：

- ClusterIP：将服务公开在集群内部。kubernetes会给服务分配一个集群内部的 IP，集群内的所有主机都可以通过这个Cluster-IP访问服务。集群内部的Pod可以通过service名称访问服务。
- [NodePort](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#type-nodeport)：通过每个节点的主机IP 和静态端口（NodePort）暴露服务。 集群的外部主机可以使用节点IP和NodePort访问服务.
- [ExternalName](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#externalname)：将集群外部的网络引入集群内部。
- [LoadBalancer](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#loadbalancer)：使用云提供商的负载均衡器向外部暴露服务。 

2种方式 命令行 yml

```
# port是service访问端口,target-port是Pod端口 二者通常是一样的 默认--type=ClusterIP
kubectl expose deployment/nginx-deployment \
--name=nginx-service --type=ClusterIP --port=8080 --target-port=80

# 随机产生主机端口
kubectl expose deployment/nginx-deployment \
--name=nginx-service2 --type=NodePort --port=8081 --target-port=80
k get svc -owide ，
nginx-service2   NodePort    10.108.213.2   <none>        8081:31398/TCP   11s   app=nginx
集群的外部主机可以使用 节点IP 和 NodePort访问服务 192.168.101.120:31398
1.NodePort端口是随机的，范围为:30000-32767。
2.集群中每一个主机节点的NodePort端口都可以访问。
3.如果需要指定端口，不想随机产生，需要使用yml配置文件来声明

#集群内访问
curl 10.108.213.2:8081

#容器内访问
kubectl run nginx-test --image=nginx:1.22 -it --rm -- sh
curl nginx-service:8080
```



![image.png](https://cdn.nlark.com/yuque/0/2022/png/28915315/1664261419003-e9a5a68a-f988-48da-a51d-b8756bce8f24.png?x-oss-process=image%2Fformat%2Cwebp)



## Namespace(命名空间)

命名空间(Namespace)是一种资源隔离机制，将同一集群中的资源划分为相互隔离的组。 命名空间可以在多个用户之间划分集群资源（通过[资源配额](https://kubernetes.io/zh-cn/docs/concepts/policy/resource-quotas/)）。 例如我们可以设置开发、测试、生产等多个命名空间。 同一ns内的资源名称要唯一，但跨ns时没有这个要求。  ns作用域仅针对带有名字空间的对象，例如 Deployment、Service 等。 这种作用域对集群访问的对象不适用，例如 StorageClass、Node、PersistentVolume 等。



Kubernetes 会创建四个初始ns：

 default		    默认的命名空间，不可删除，未指定命名空间的对象都会被分配到default中。 

kube-system Kubernetes 系统对象(控制平面和Node组件)所使用的命名空间。

 kube-public   自动创建的公共命名空间，所有用户（包括未经过身份验证的用户）都可以读取它。通常我们约定，将整个集群中公用的可见和可读的资源放在这个空间中。 

 kube-node-lease  租约（Lease）对象使用的命名空间。每个节点都有一个关联的 lease 对象，lease 是一种轻量级资源。lease对象通过发送心跳，检测集群中的每个节点是否发生故障。



管理ns

```
#创建命名空间
kubectl create namespace dev
#查看命名空间
kubectl get ns

#在命名空间内运行Pod
kubectl run nginx    --image=nginx:1.21 --namespace=dev
kubectl run my-nginx --image=nginx:1.21 -n=dev

#查看命名空间内的Pod
kubectl get pods -n=dev

#查看命名空间内所有对象
kubectl get all
# 删除命名空间会删除命名空间下的所有内容
kubectl delete ns dev

#查看当前上下文
kubectl config current-context

#将dev设为当前命名空间，后续所有操作都在此命名空间下执行。
kubectl config set-context $(kubectl config current-context) --namespace=dev
```



声明式对象配置

云原生的代表技术包括： 容器 服务网格 微服务 不可变基础设施 声明式API



**管理对象**

**命令行指令**

例如，使用kubectl命令来创建和管理 Kubernetes 对象。

命令行就好比口头传达，简单、快速、高效。

但功能有限，不适合复杂场景，操作不容易追溯，多用于开发和调试。

**声明对象配置yaml**

使用yaml文件来描述 Kubernetes 对象。

声明式配置就好比申请表，学习难度大且配置麻烦。

好处是操作留痕，适合操作复杂的对象，多用于生产。

 

常用命令缩写

| 名称         | 缩写   | Kind        |
| ------------ | ------ | ----------- |
| namespaces   | ns     | Namespace   |
| nodes        | no     | Node        |
| pods         | po     | Pod         |
| services     | svc    | Service     |
| deployments  | deploy | Deployment  |
| replicasets  | rs     | ReplicaSet  |
| statefulsets | sts    | StatefulSet |



配置对象

在创建的 Kubernetes 对象所对应的 `yaml`文件中，需要配置的字段如下：

- apiVersion	Kubernetes API 的版本
- kind			   对象类别，例如`Pod`、`Deployment`、`Service`、`ReplicaSet`等
- metadata     描述对象的元数据，包括一个 name 字符串、UID 和可选的 namespace
- spec              对象的配置



## [Pod配置模版](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/#using-pods)

```
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
  app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
#创建对象
kubectl apply -f pod.yaml
#编辑对象
kubectl edit nginx
#删除对象
kubectl delete -f pod.yaml
```



**标签（Labels）** 是附加到对象（比如 Pod）上的键值对，用于补充对象的描述信息。

标签使用户能够以松散的方式管理对象映射，而无需客户端存储这些映射。

由于一个集群中可能管理成千上万个容器，我们可以使用标签高效的进行选择和操作容器集合。

## [label配置模版](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/#syntax-and-character-set)

```
apiVersion: v1
kind: Pod
metadata:
  name: label-demo
  labels: #定义Pod标签
    environment: dev
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.22
    ports:
    - containerPort: 80
kubectl get pod --show-labels
kubectl get pod -l environment=dev,app=nginx
```



 

## 选择器

标签选择器 可以识别一组对象。标签不支持唯一性。

标签选择器最常见的用法是 为Service选择一组Pod作为后端。

## [Service配置模版](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#type-nodeport)

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector: #与Pod的标签一致
    environment: test
    app: nginx
  ports:
      # 默认情况下，为了方便起见，`targetPort` 被设置为与 `port` 字段相同的值。
    - port: 80
      targetPort: 80
      # 可选字段
      # 默认情况下，为了方便起见，Kubernetes 控制平面会从某个范围内分配一个端口号（默认：30000-32767）
      nodePort: 30007
```

目前支持两种类型的选择运算：基于等值的和基于集合的。 

多个选择条件使用逗号分隔，相当于And(&&)运算。

**●**等值选择

```
selector:
  matchLabels: # component=redis && version=7.0
    component: redis
    version: 7.0
```

**●**集合选择

```
selector:
  matchExpressions: # tier in (cache, backend) && environment not in (dev, prod)
    - {key: tier, operator: In, values: [cache, backend]}
    - {key: environment, operator: NotIn, values: [dev, prod]}
```

参考资料：

https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/kubernetes-objects/

https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/object-management/

https://kubernetes.io/docs/reference/kubectl/#resource-types

https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/

https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/



mysql

```
mkdir /data/kubernetes/mysql -p
vim mysql-pod.yml

apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
spec:
  containers:
    - name: mysql
      image: mysql:5.7
      env:
        - name: MYSQL_ROOT_PASSWORD
          value: "root"
      ports:
        - containerPort: 3306
      volumeMounts:
        - mountPath: /var/lib/mysql #容器中的目录
          name: data-volume
  volumes:
    - name: data-volume
      hostPath:
        # 宿主机上目录位置
        path: /data/kubernetes/mysql/data
        type: DirectoryOrCreate
```

创建MySQL数据库

配置环境变量

使用[MySQL镜像](https://hub.docker.com/_/mysql)创建Pod，需要使用环境变量设置MySQL的初始密码。 [环境变量配置示例](https://kubernetes.io/zh-cn/docs/tasks/inject-data-application/define-environment-variable-container/#define-an-env-variable-for-a-container)

挂载卷 将数据存储在容器中，一旦容器被删除，数据也会被删除。 将数据存储到卷(Volume)中，删除容器时，卷不会被删除。

hostPath卷 hostPath 卷将主机节点上的文件或目录挂载到 Pod 中。



[hostPath配置示例](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath-configuration-example)

hostPath的type值：

| **DirectoryOrCreate** | **目录不存在则自动创建。**                                   |
| --------------------- | ------------------------------------------------------------ |
| **Directory**         | **挂载已存在目录。不存在会报错。**                           |
| **FileOrCreate**      | **文件不存在则自动创建。\****不会自动创建文件的父目录，必须确保文件路径已经存在。** |
| **File**              | **挂载已存在的文件。不存在会报错。**                         |
| **Socket**            | **挂载 UNIX 套接字。例如挂载\****/var/run/docker.sock进程**  |

**注意：hostPath** 仅用于在单节点集群上进行开发和测试，不适用于多节点集群；

例如，当Pod被重新创建时，可能会被调度到与原先不同的节点上，导致新的Pod没有数据。

在多节点集群使用本地存储，可以使用`local`卷。





## ConfigMap与Secret

Docker中，一般通过 绑定挂载 的方式将 配置文件 挂载到 容器里。 在Kubernetes集群中，容器可能被调度到任意节点，配置文件需要能在 集群中任意节点上 访问、分发和更新。

 ConfigMap 

ConfigMap 用来在 键值对数据库(etcd)中保存非加密数据。**一般用来保存配置文件**。

ConfigMap 可用作 环境变量、命令行参数或者存储卷。

ConfigMap 将 环境配置信息 与 [容器镜像](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-image) 解耦，便于配置的修改。

ConfigMap 在设计上不是用来保存大量数据的。

在 ConfigMap 中保存的数据不可超过 1 MiB。

超出此限制，需要考虑 挂载存储卷 或者 访问文件存储服务。

 ConfigMap用法 

●[ConfigMap配置示例](https://kubernetes.io/docs/concepts/configuration/configmap/#configmaps-and-pods)

●[Pod中使用ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/#using-configmaps-as-files-from-a-pod)

```
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
  labels:
    app: mysql
spec:
  containers:
    - name: mysql
      image: mysql:5.7
      env:
        - name: MYSQL_ROOT_PASSWORD
          value: "root"
      volumeMounts:
        - mountPath: /var/lib/mysql
          name: data-volume
        - mountPath: /etc/mysql/conf.d
          name: conf-volume
          readOnly: true
  volumes:
    - name: conf-volume
      configMap:
        name: mysql-config
    - name: data-volume
      hostPath:
        # directory location on host
        path: /data/kubernetes/mysql/data
        # this field is optional
        type: DirectoryOrCreate
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  mysql.cnf: |
    [mysqld]
    character-set-server=utf8mb4
    collation-server=utf8mb4_general_ci
    init-connect='SET NAMES utf8mb4'

    [client]
    default-character-set=utf8mb4

    [mysql]
    default-character-set=utf8mb4
```





## Secret

Secret 用于 保存机密数据 的对象。一般由于保存密码、令牌或密钥等。 

data字段用来存储 base64 编码数据。

stringData存储未编码的字符串。

Secret 意味着你不需要在应用程序代码中包含机密数据，减少机密数据(如密码)泄露的风险。

Secret 可以用作环境变量、命令行参数或者存储卷文件。

Secret用法

- [Secret配置示例](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/#use-case)
- [将Secret用作环境变量](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)

```
echo -n 'root' | base64
echo 'cm9vdA==' | base64 --decode
apiVersion: v1
kind: Secret
metadata:
  name: mysql-password
type: Opaque
data:
  PASSWORD: cm9vdA==
---
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
  labels:
    app: mysql
spec:
  containers:
    - name: mysql
      image: mysql:5.7
      env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-password
              key: PASSWORD
      volumeMounts:
        - mountPath: /var/lib/mysql
          name: data-volume
        - mountPath: /etc/mysql/conf.d
          name: conf-volume
          readOnly: true
  volumes:
    - name: conf-volume
      configMap:
        name: mysql-config
    - name: data-volume
      hostPath:     
        path: /data/kubernetes/mysql/data
        type: DirectoryOrCreate
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-config
data:
  mysql.cnf: |
    [mysqld]
    character-set-server=utf8mb4
    collation-server=utf8mb4_general_ci
    init-connect='SET NAMES utf8mb4'

    [client]
    default-character-set=utf8mb4

    [mysql]
    default-character-set=utf8mb4
```





## 卷(Volume)

将 数据存储 在容器中，一旦容器被删除，数据也会被删除。

卷 独立于 容器之外 的 一块存储区域，通过挂载(Mount)的方式 供Pod中的容器使用。 使用场景

- 在多个容器之间共享数据。
- 将容器数据存储在外部存储或云存储上。
- 容易备份或迁移。



#### 常见的卷类型

- **临时卷(Ephemeral Volume)：**与 Pod 一起创建和删除，生命周期与 Pod 相同
- - [emptyDir](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#emptydir)  - 作为缓存或存储日志
  - [configMap](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#configmap) 、[secret](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#secret)、 [downwardAPI](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#downwardapi) - 给Pod注入数据
- **持久卷(Persistent Volume)：**删除Pod后，持久卷不会被删除
- - 本地存储 - [hostPath](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#hostpath)、 [local](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#local)
- - 网络存储 - [NFS](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#nfs)
  - 分布式存储 - Ceph([cephfs](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#cephfs)文件存储、[rbd](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#rbd)块存储)
- **投射卷(Projected Volumes)：**[projected](https://kubernetes.io/zh-cn/docs/concepts/storage/projected-volumes/) 卷可以将多个卷映射到同一个目录上



#### 后端存储

一个集群中 可包含多种存储(如`local`、`NFS`、`Ceph`或云存储)。

每种存储都对应一个**存储类（StorageClass）** ，存储类 用来 创建和管理持久卷，是集群与存储服务之间的桥梁。

管理员创建持久卷(**PV**)时，通过设置不同的**StorageClass**来创建不同类型的持久卷。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/28915315/1666320617840-eeb42675-6e6d-4306-910a-080017b7975b.png?x-oss-process=image%2Fformat%2Cwebp%2Fresize%2Cw_1406%2Climit_0)

参考文档：

https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/

https://kubernetes.io/zh-cn/docs/concepts/storage/ephemeral-volumes/

https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-volume-storage/



```
[root@k8s-worker2 conf-volume]# ll
总用量 0
lrwxrwxrwx 1 root root 16 2月   7 19:58 mysql.cnf -> ..data/mysql.cnf
[root@k8s-worker2 conf-volume]# pwd
/var/lib/kubelet/pods/539cd2e5-e4cf-4019-b518-feed135182d1/volumes/kubernetes.io~configmap/conf-volume
删除pod时539cd2e5目录会删除
```



Ephemeral Volume

与 Pod 一起创建和删除，生命周期与 Pod 相同

[emptyDir](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#emptydir) - 初始内容为空的本地临时目录

[configMap](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#configmap) - 为Pod注入配置文件

[secret](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#secret) - 为Pod注入加密数据



## 持久卷(PV)与持久卷声明(PVC)

持久卷(Persistent Volume) 删除Pod后，卷不会被删除

本地存储

- [hostPath](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#hostpath) - 节点主机上的目录或文件

- (仅供单节点测试使用；多节点集群请用 local 卷代替)

- [local](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#local) - 节点上挂载的本地存储设备(不支持动态创建卷)

- 网络存储

- [NFS](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#nfs) - 网络文件系统 (NFS) 

- 分布式存储

- Ceph([cephfs](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#cephfs)文件存储、[rbd](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#rbd)块存储)

  

  pv pvc

  持久卷（PV） 是 集群中的一块存储。可以理解为一块虚拟硬盘。

  持久卷 可以由 管理员事先创建， 或者 使用[存储类（Storage Class）](https://kubernetes.io/zh-cn/docs/concepts/storage/storage-classes/)根据用户请求来动态创建。 

  持久卷 属于集群的公共资源，并不属于某个namespace; 

  

  持久卷声明（PVC） 表达的是 用户对存储的请求。

  PVC类似申请单，它更贴近云服务的使用场景，使用资源先申请，便于统计和计费。

  Pod 将 PVC 声明当做存储卷来使用，PVC 可以请求指定容量的存储空间和[访问模式](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/#access-modes) 。PVC对象是带有namespace的。

  

  ![image-20240513163338077](k8s-note/image-20240513163338077.png?lastModify=1716449400)

  

  创建持久卷（PV） 

  创建持久卷(PV)是服务端的行为，通常 集群管理员提前创建一些常用规格的持久卷以备使用。

  hostPath仅供单节点测试使用，当Pod被重新创建时，可能会被调度到与原先不同的节点上，导致新的Pod没有数据。

  多节点集群使用本地存储，可以使用local卷

  **创建local类型的持久卷，需要先创建存储类(StorageClass)**。

  

  ```
  # 创建本地存储类
  apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
    name: sc-mysql
  provisioner: kubernetes.io/no-provisioner
  volumeBindingMode: Immediate
  ---
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: pv-mysql
  spec:
    capacity:
      storage: 2Gi
    volumeMode: Filesystem
    accessModes:
    - ReadWriteOnce
    persistentVolumeReclaimPolicy: Delete
    storageClassName: sc-mysql #通过指定存储类来设置卷的类型
    local:
      path: /mnt/disks/ssd1    #该目录须在worker1上手动创建
    nodeAffinity:
      required:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: In
            values:
            - k8s-worker1
  ```

  

pod不直接使用pv，而是使用pvc

apiVersion: v1 kind: PersistentVolumeClaim metadata:  name: pvc-mysql spec:  storageClassName: sc-mysql # 与PV中的storageClassName一致  accessModes:

```
- ReadWriteOnce
resources:
    requests:
  storage: 2Gi
```











构建镜像

nginx tomcat举例

docker commit方式创建

```
for i in nginx-ingress-controller:v1.3.1 kube-webhook-certgen:v1.3.0
do
  docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$i
  docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$i k8s-register.sswang.com/google_containers/$i
  docker push k8s-register.sswang.com/google_containers/$i
  docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$i
done
```



# helm

## the best way to find, share, and use software built for [Kubernetes](https://kubernetes.io/).

![image-20240523162326257](k8s-note/image-20240523162326257.png)

| 概念       | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| Chart      | 一个Helm包，其中包含了运行一个应用所需要的镜像、依赖和资源定义等，还可能包含Kubernetes集群中的服务定义，类似Homebrew中的formula、APT的dpkg或者Yum的rpm文件 |
| Repository | 存储Helm Charts的地方                                        |
| Release    | Chart在k8s上运行的Chart的一个实例，例如，如果一个MySQL Chart想在服务器上运行两个数据库，可以将这个Chart安装两次，并在每次安装中生成自己的Release以及Release名称 |
| Value      | Helm Chart的参数，用于配置Kubernetes对象                     |
| Template   | 使用Go模板语言生成Kubernetes对象的定义文件                   |

![image-20240523162809528](k8s-note/image-20240523162809528.png)

```sh
#helm下载
cd /data/softs && wget https://get.helm.sh/helm-v3.13.0-linux-amd64.tar.gz

#配置环境
mkdir /data/server/helm/bin -p
tar xf helm-v3.13.0-linux-amd64.tar.gz
mv linux-amd64/helm /data/server/helm/bin/

#添加环境变量
echo 'export PATH=$PATH:/data/server/helm/bin' >> /etc/profiles
source /etc/profiles

确认效果
# helm  version
```

```sh
#仓库管理
#添加仓库 
helm repo add az-stable http://mirror.azure.cn/kubernetes/charts/
helm repo add bitnami https://charts.bitnami.com/bitnami

#查看仓库 
helm repo list
NAME            URL
az-stable       http://mirror.azure.cn/kubernetes/charts/
bitnami         https://charts.bitnami.com/bitnami

#更新仓库属性信息
helm repo update

#从自定义仓库中获取软件源信息 
helm search repo redis

#查看chart的所有信息 
helm show all bitnami/redis
```

```sh
#安装chart 
helm install mysql-helm bitnami/mysql
#删除应用 
helm uninstall my-helm 
#更新应用 
helm install my-helm bitnami/redis --set master.persistence.enabled=false --set replica.persistence.enabled=false 
#查看效果 
helm list 

kubectl get pod
```

