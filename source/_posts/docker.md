---
title: Docker
---

# Docker

# 1.初识Docker

### 1.1.1.应用部署的环境问题

大型项目 组件多，运行环境复杂，部署碰到问题：

- 依赖关系 复杂，兼容性问题

- 开发、测试、生产环境 有差异



<img src="docker/image-20210731141907366.png" alt="image-20210731141907366" style="zoom:80%;" />



一个项目中，部署需要依赖node.js、Redis、RabbitMQ、MySQL等，这些服务部署所需的函数库、依赖项各不相同，甚至会有冲突，给部署带来困难。



### 1.1.2.Docker解决依赖兼容问题

Docker为解决依赖的兼容问题的，采用 两个手段：

- 将 应用的Libs（函数库）、Deps（依赖）、配置 与 应用 一起打包

- 将每个应用放到一个隔离**容器**去运行，避免互相干扰

![image-20210731142219735](docker/image-20210731142219735.png)



这样打包好的应用包中，包含应用本身极其所需要Libs、Deps，无需再在操作系统上安装，就不存在不同应用之间的兼容问题了。

虽然解决了兼容问题，但是开发、测试等环境依然有差异

### 1.1.3.Docker解决操作系统环境差异

要解决不同**操作系统环境差异问题**，必须先了解操作系统结构。以一个Ubuntu操作系统为例，结构如下：

![image-20210731143401460](docker/image-20210731143401460.png)



结构包括：

- 计算机硬件：例如CPU、内存、IO、硬盘、网卡等
- 系统内核：所有Linux发行版的内核都是Linux，例如CentOS、Ubuntu、Fedora等。内核与计算机硬件交互，通过**内核指令**操作计算机硬件。
- 系统应用：操作系统自带的应用、函数库。这些**函数库是对内核指令的封装**。centos和ubuntu对内核指令的封装不同



应用与计算机交互的流程：

1）应用 调用 操作系统应用（函数库），实现各种功能

2）系统函数库是对内核指令集的封装，会调用内核指令

3）内核指令 操作 计算机硬件



Ubuntu和CentOS都基于Linux内核，无非是 **系统应用**、**函数库**有差异：

![image-20210731144304990](docker/image-20210731144304990.png)



此时，如果将一个Ubuntu版本的MySQL应用安装到CentOS系统，MySQL在调用Ubuntu函数库时，会发现找不到或者不匹配，就会报错了：

![image-20210731144458680](docker/image-20210731144458680.png)



Docker如何解决？

- Docker将 应用 与所需 调用的系统(比如Ubuntu)函数库一起打包
- Docker运行到不同操作系统时，直接基于打包的函数库，借助于操作系统的Linux内核来运行

如图：

![image-20210731144820638](docker/image-20210731144820638.png)



### 1.1.4.小结

- Docker允许将 应用、依赖、函数库、配置一起**打包**，形成一个可移植的镜像
- Docker应用运行在容器中，使用沙箱机制，相互**隔离**

- Docker镜像中包含完整运行环境，包括系统函数库，**仅依赖系统的Linux内核**，因此可以在任意Linux操作系统上运行



## 1.2.Docker和虚拟机的区别

**虚拟机**（virtual machine）是在操作系统中**模拟**硬件设备，然后运行另一个操作系统

比如在 Windows 系统里面运行 Ubuntu 系统，这样就可以运行任意的Ubuntu应用了。

**Docker**是封装函数库，并没有模拟完整的操作系统，如图：

![image-20210731145914960](docker/image-20210731145914960.png)

对比来看：

<img src="docker/image-20210731152243765.png" alt="image-20210731152243765" style="zoom: 33%;" />



小结：

Docker和虚拟机的差异：

- **docker是一个系统进程**；虚拟机是在操作系统中的操作系统

- docker体积小、启动速度快、性能好；虚拟机体积大、启动速度慢、性能一般



## 1.3.Docker架构

### 1.3.1.镜像和容器

**镜像（Image）**：应用程序及其所需的依赖、函数库、环境、配置等文件的打包

**容器（Container）**：镜像运行后形成的进程，Docker会给容器进程 做隔离，对外不可见。



一切应用最终都是代码组成，都是硬盘中的一个个的字节形成的文件。只有运行时，才会加载到内存，形成进程。



而**镜像**，就是把一个应用在硬盘上的文件、及其运行环境、部分系统函数库文件一起打包 形成的 文件包。是只读的。

**容器**，就是将这些文件中 编写的程序、函数加载到内存中，形成进程，只不过要隔离起来。因此一个镜像可以启动多次，形成多个容器进程。



![image-20210731153059464](docker/image-20210731153059464.png)



### 1.3.2.DockerHub

开源应用程序非常多，打包这些应用是重复的劳动。为避免这些重复劳动，人们将打包的应用镜像，例如Redis、MySQL镜像上传到网络上，共享使用，就像GitHub的代码共享一样。

- DockerHub：DockerHub是一个官方的Docker镜像的托管平台。这样的平台称为Docker Registry。

- 国内也有类似于DockerHub 的公开服务，比如 [网易云镜像服务](https://c.163yun.com/hub)、[阿里云镜像库](https://cr.console.aliyun.com/)等。

![image-20210731153743354](docker/image-20210731153743354.png)



### 1.3.3.Docker架构

CS架构：

- 服务端(server)：Docker守护进程，负责处理Docker指令，管理镜像、容器等

- 客户端(client)：通过命令或RestAPI向Docker服务端发送指令。可以在本地或远程向服务端发送指令。

如图：

![image-20210731154257653](docker/image-20210731154257653.png)



### 1.3.4.小结

镜像：

​	将应用程序及其依赖、环境、配置打包在一起的 文件

容器：

​	镜像运行后的 进程，一个镜像可以运行多个容器

Docker架构：

​	服务端：接收命令或远程请求，操作镜像或容器

​	客户端：发送命令或者请求到Docker服务端

镜像仓库：

​	公有：dockerhub、阿里云、华为云镜像服务

　私有：harbor



## 1.4.安装Docker

官方安装参考链接

https://docs.docker.com/engine/install/centos/



CentOS下安装Docker

1.脚本快速安装

```sh
#Docker Engine安装
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

#Docker Compose安装
curl -SL https://github.com/docker/compose/releases/download/v2.26.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```



2.仓库安装

```sh
#Set up the repository for download docker
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#To install a specific version, start by listing the available versions in the repository
#yum list docker-ce --showduplicates | sort -r

#<tag>替换为对应版本
#sudo yum install docker-ce-<tag> docker-ce-cli-<tag> containerd.io docker-buildx-plugin docker-compose-plugin

sudo yum install -y docker-ce-24.0.0  docker-ce-cli-24.0.0  containerd.io docker-buildx-plugin docker-compose-plugin
sudo yum remove -y docker-ce-20.10.9  docker-ce-cli-20.10.9  containerd.io docker-buildx-plugin docker-compose-plugin

#给docker配置阿里云镜像源
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://dockerproxy.com",
    "https://hub-mirror.c.163.com",
    "https://mirror.baidubce.com",
    "https://ccr.ccs.tencentyun.com"
  ]
}
EOF

sudo systemctl start docker
```

# 2.Docker操作

## 2.1.镜像操作

### 2.1.1.镜像名称

repository:tag

redis:7.2.0

- 在没有指定tag时，默认是latest

docker images

![1682673267683](docker/1682673267683.png)



### 2.1.2.镜像命令

![image-20210731155649535](docker/image-20210731155649535.png)



### 2.1.3.拉取、查看镜像

从DockerHub中拉取nginx镜像

1）DockerHub 搜索 nginx 镜像

![image-20210731155844368](docker/image-20210731155844368.png)

2）docker pull nginx

![image-20210731155856199](docker/image-20210731155856199.png)

3）docker images 

![image-20210731155903037](docker/image-20210731155903037.png)



### 2.1.4.保存、导入镜像

```shell
docker save -o nginx.tar nginx:latest
docker load -i nginx.tar
```

查看save用法

```sh
docker save --help
```

![image-20210731161104732](docker/image-20210731161104732.png)



```shell
docker save -o [保存的目标文件名称] [镜像名称]
```

```sh
docker save -o /root/nginx.tar nginx:latest
```

![image-20210731161354344](docker/image-20210731161354344.png)



使用docker load加载镜像

先删除本地的nginx镜像：

```sh
docker rmi nginx:latest
```

再加载本地文件：

```sh
docker load -i nginx.tar
```

![image-20210731161746245](docker/image-20210731161746245.png)



需求：DockerHub搜索并拉取一个Redis镜像

1）去DockerHub搜索Redis镜像

2）查看Redis镜像的名称和版本

3）利用docker pull命令拉取镜像

docker pull redis:6.0

4）利用docker save命令将 redis:6.0 打包为一个 redis6.0.tar 包

docker save -o redis6.0.tar  redis:6.0

5）利用docker rmi 删除本地的redis:6.0

docker rmi redis:6.0

6）利用docker load 重新加载 redis6.0.tar文件

docker load  -i redis6.0.tar



## 2.2.容器操作

### 2.2.1.容器相关命令

![image-20210731161950495](docker/image-20210731161950495.png)

容器的三状态

- 运行（running）：进程正常运行
- 暂停（suspend）：进程暂停，CPU不再运行，并不释放内存
- 停止(halt)：进程终止，回收进程占用的内存、CPU等资源



- docker run：创建并运行一个容器，处于运行状态
- docker pause：让一个运行的容器暂停
- docker unpause：让一个容器从暂停状态恢复运行
- docker stop：停止一个运行的容器
- docker start：让一个停止的容器再次运行

- docker rm：删除一个容器



### 2.2.2.创建并运行一个容器

通过镜像运行容器

```sh
docker run -it busybox sh
docker run --name nginx -p 80:80 -d nginx:1.21
```

run  创建并运行一个容器

--name   给容器命名

-p  宿主机端口 ：容器端口

-d  后台运行容器

nginx:1.21 镜像名称



默认情况下，容器是隔离环境，直接访问宿主机的80端口，肯定访问不到容器中的nginx 80端口。

只有将 容器的80 与 宿主机的80 做映射，访问宿主机的80 就能映射到 容器的80：

![image-20210731163255863](docker/image-20210731163255863.png)



### 2.2.3.进入容器，修改文件

**需求**：进入Nginx容器，修改HTML文件内容，添加“传智教育欢迎您”

1）进入Nginx容器

```sh
docker exec -it nginx bash
```

- exec ：进入容器内部

- -it : 给当前进入的容器创建一个标准输入、输出终端，允许我们与容器交互

- nginx：要进入的容器的名称

- bash：进入容器后执行的命令 

2）进入nginx的HTML所在目录 /usr/share/nginx/html

容器内部会模拟一个独立的Linux文件系统

![image-20210731164159811](docker/image-20210731164159811.png)

nginx的环境、配置、运行文件  都在这个文件系统中，包括我们要修改的html文件。

查看DockerHub网站中的nginx页面，可以知道nginx的html目录位置在`/usr/share/nginx/html`

```sh
cd /usr/share/nginx/html
```

 查看目录下文件：

![image-20210731164455818](docker/image-20210731164455818.png)



3）修改index.html的内容

容器内没有vi命令，用下面的命令来修改：

```sh
sed -i -e 's#Welcome to nginx#传智教育欢迎您#g' -e 's#<head>#<head><meta charset="utf-8">#g' index.html
```



在浏览器访问**自己的宿主机地址**，即可看到结果：

![image-20210731164717604](docker/image-20210731164717604.png)



### 2.2.4.小结

docker run的常见参数

--name 指定容器名称

-p 		 指定端口映射

-d  	    让容器后台运行

查看容器日志的命令：

docker logs nginx

docker logs -f nginx

-f  持续查看日志



查看容器状态：

docker ps

docker ps -a 查看所有容器，包括已经停止的

运行后退出删除容器

```
docker run -it --rm busybox sh
```



### 2.2.5 安装pg

```text
docker run --name postgres -e POSTGRES_PASSWORD=root -p 5432:5432 -v /root/db/postgres/data:/var/lib/postgresql/data -d postgres
```

  

## 2.3.数据卷（容器数据管理）

在之前的nginx案例中，修改nginx的html页面时，需要进入nginx内部。并且因为没有编辑器，修改文件也很麻烦。

这就是因为 容器与数据（容器内文件）耦合带来的后果。

![image-20210731172440275](docker/image-20210731172440275.png)

要解决这个问题，必须将数据与容器解耦，这就要用到**数据卷**了。



### 2.3.1.什么是数据卷

**数据卷（volume）**是一个虚拟目录，指向宿主机文件系统中的某个目录。

![image-20210731173541846](docker/image-20210731173541846.png)

一旦完成数据卷挂载，对容器的一切操作都会作用在数据卷对应的宿主机目录了。

这样，我们操作宿主机的/var/lib/docker/volumes/html目录，就等于操作容器内的/usr/share/nginx/html目录了





### 2.3.2.数据卷操作命令

基本语法如下：

```sh
docker volume [COMMAND]
```

docker volume命令是数据卷操作，根据命令后跟随的command来确定下一步的操作：

- create 创建一个volume
- inspect 显示一个或多个volume的信息
- ls 列出所有的volume
- prune 删除未使用的volume
- rm 删除一个或多个指定的volume



### 2.3.3.创建和查看数据卷

**需求**：创建一个数据卷，并查看数据卷在宿主机的目录位置

创建 数据卷

```sh
docker volume create html
```

查看 所有数据

```sh
docker volume ls
```

结果：

![image-20210731173746910](docker/image-20210731173746910.png)



查看 数据卷 详细信息

```sh
docker volume inspect html
```

结果：

![image-20210731173809877](docker/image-20210731173809877.png)

可以看到，我们创建的html这个数据卷关联的宿主机目录为`/var/lib/docker/volumes/html/_data`目录。







**小结**：

数据卷的作用

将 容器 与 数据 分离，解耦，方便操作容器内数据，保证数据安全



数据卷操作

```sh
docker volume create #创建数据卷
docker volume ls 	 #查看所有数据卷
docker volume inspect#查看数据卷详细信息，包括关联的宿主机目录位置
docker volume rm 	 #删除指定数据卷
docker volume prune  #删除所有未使用的数据卷
```





### 2.3.4.挂载数据卷

我们在创建容器时，可以通过 -v 参数来挂载一个数据卷到某个容器内目录，命令格式如下：

```sh
docker run \
  --name mn \
  -v html:/root/html \
  -p 8080:80
  nginx \
```

-v 挂载数据卷

- `-v html:/root/htm`  把 本地html数据卷 挂载到 容器内的/root/html 



### 2.3.5.案例-给nginx挂载数据卷

**需求**：创建一个nginx容器，修改容器内的html目录内的index.html内容



**分析**：上个案例中，我们进入nginx容器内部，已经知道nginx的html目录所在位置/usr/share/nginx/html ，我们需要把这个目录挂载到html这个数据卷上，方便操作其中的内容。

**提示**：运行容器时使用 -v 参数挂载数据卷

步骤：

① 创建容器并挂载数据卷到容器内的HTML目录

```sh
docker run --name mn -v html:/usr/share/nginx/html -p 80:80 -d nginx
```



② 进入html数据卷所在位置，并修改HTML内容

```sh
# 查看html数据卷的位置
docker volume inspect html
# 进入该目录
cd /var/lib/docker/volumes/html/_data
# 修改文件
vi index.html
```



### 2.3.6.案例-给MySQL挂载本地目录

容器不仅仅可以挂载数据卷，也可以直接挂载到宿主机目录上。关联关系如下：

- 带数据卷模式：宿主机目录 --> 数据卷 ---> 容器内目录
- 直接挂载模式：宿主机目录 ---> 容器内目录

如图：

![image-20210731175155453](docker/image-20210731175155453.png)

**语法**：

目录挂载与数据卷挂载的语法是类似的：

- -v [宿主机目录]:[容器内目录]
- -v [宿主机文件]:[容器内文件]



**需求**：创建并运行一个MySQL容器，将宿主机目录直接挂载到容器

```
version: '3'
services:
  mysql:
    image: mysql:8.0.33
    container_name: mysql-8.0.33
    restart: always
    ports:
      - 3306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - TZ=Asia/Shanghai
    volumes:
      - /home/dockerdata/mysql/data:/var/lib/mysql 
      - /home/dockerdata/mysql/conf.d:/etc/mysql/conf.d
      - /home/dockerdata/mysql/log:/var/log/mysql
      - /etc/localtime:/etc/localtime:ro
```



### 2.3.7.小结

docker run的命令中通过 -v 参数挂载文件或目录到容器中：

- -v volume名称:容器内目录
- -v 宿主机文件:容器内文件
- -v 宿主机目录:容器内目录

数据卷挂载与目录直接挂载的

- 数据卷挂载耦合度低，由docker来管理目录，但是目录较深，不好找
- 目录挂载耦合度高，需要我们自己管理目录，不过目录容易寻找查看

# 3 Dockerfile自定义镜像

常见的镜像在DockerHub就能找到，但 自己写的项目  就必须自己 构建镜像。

而要 自定义镜像，就必须先了解 镜像的结构 才行。

## 3.1.镜像结构

镜像是将 **应用程序及其需要的系统函数库、环境、配置、依赖** 打包而成。

以MySQL为例，来看看镜像的组成结构：

![image-20210731175806273](docker/image-20210731175806273.png)



简单来说，镜像 就是在  **系统函数库、运行环境基础上，添加应用程序文件、配置文件、依赖文件等组合**，然后编写好 启动脚本 打包在一起形成的文件。

我们要 构建镜像，就是实现上述 打包的过程。



## 3.2.Dockerfile语法

构建自定义的镜像时，并不需要一个个文件去拷贝，打包。

只需要告诉Docker，镜像的组成，需要哪些BaseImage、需要拷贝什么文件、需要安装什么依赖、启动脚本是什么，将来Docker会帮助我们构建镜像。

而描述上述信息的文件 就是 Dockerfile文件。



**Dockerfile** 是一个文本文件，包含 一个个的**指令(Instruction)**，用指令来说明要执行什么操作来构建镜像。每一个指令都会形成一层Layer。

![image-20210731180321133](docker/image-20210731180321133.png)

更新详细语法说明，请参考官网文档： https://docs.docker.com/engine/reference/builder



## 3.3.构建Java项目

### 3.3.1.基于Ubuntu构建Java项目

需求：基于Ubuntu镜像构建一个新镜像，运行一个java项目

- 步骤1 新建一个空文件夹docker-demo

  ![image-20210801101207444](docker/image-20210801101207444.png)

- 步骤2 拷贝课前资料中的docker-demo.jar文件到docker-demo这个目录

  ![image-20210801101314816](docker/image-20210801101314816.png)

- 步骤3：拷贝课前资料中的 jdk8.tar.gz文件 到 docker-demo这个目录

  ![image-20210801101410200](docker/image-20210801101410200.png)

- 步骤4：拷贝课前资料提供的Dockerfile到docker-demo这个目录

  ![image-20210801101455590](docker/image-20210801101455590.png)

  其中的内容如下：

  ```dockerfile
  # 指定基础镜像
  FROM ubuntu:16.04
  
  # 配置环境变量，JDK的安装目录
  ENV JAVA_DIR=/usr/local
  
  # 拷贝jdk和java项目的包
  COPY ./jdk8.tar.gz $JAVA_DIR/
  COPY ./docker-demo.jar /tmp/app.jar
  
  # 安装JDK
  RUN cd $JAVA_DIR \
   && tar -xf ./jdk8.tar.gz \
   && mv ./jdk1.8.0_144 ./java8
  
  # 配置环境变量
  ENV JAVA_HOME=$JAVA_DIR/java8
  ENV PATH=$PATH:$JAVA_HOME/bin
  
  # 暴露端口
  EXPOSE 8090
  
  # 入口，java项目的启动命令
  ENTRYPOINT java -jar /tmp/app.jar
  ```

  

- 步骤5：进入docker-demo

  将准备好的docker-demo上传到虚拟机任意目录，然后进入docker-demo目录下

  

- 步骤6：运行命令：

  ```sh
  docker build -t javaweb:1.0 .
  ```

  

最后访问 http://192.168.150.101:8090/hello/count，其中的ip改成你的虚拟机ip



### 3.3.2.基于java8构建Java项目

虽然我们可以基于Ubuntu基础镜像，添加任意自己需要的安装包，构建镜像，但是却比较麻烦。所以大多数情况下，我们都可以在一些安装了部分软件的基础镜像上做改造。

例如，构建java项目的镜像，可以在已经准备了JDK的基础镜像基础上构建。



需求：基于java:8-alpine镜像，将一个Java项目构建为镜像

实现思路如下：

- ① 新建一个空的目录，然后在目录中新建一个文件，命名为Dockerfile

- ② 拷贝课前资料提供的docker-demo.jar到这个目录中

- ③ 编写Dockerfile文件：

  - a ）基于java:8-alpine作为基础镜像

  - b ）将app.jar拷贝到镜像中

  - c ）暴露端口

  - d ）编写入口ENTRYPOINT

    内容如下：

    ```dockerfile
    FROM java:8-alpine
    COPY ./app.jar /tmp/app.jar
    EXPOSE 8090
    ENTRYPOINT java -jar /tmp/app.jar
    ```

    

- ④ 使用docker build命令构建镜像

- ⑤ 使用docker run创建容器并运行



## 3.4.小结

小结：

1. Dockerfile的本质是一个文件，通过指令描述镜像的构建过程

2. Dockerfile的第一行必须是FROM，从一个基础镜像来构建

3. 基础镜像可以是基本操作系统，如Ubuntu。也可以是其他人制作好的镜像，例如：java:8-alpine



# 4.Docker-Compose

Docker Compose基于 Compose文件 快速部署分布式应用，而无需手动一个个创建和运行容器！

![image-20210731180921742](docker/image-20210731180921742.png)

## 4.1.初识DockerCompose

Compose文件是一个文本文件,定义集群中的每个容器如何运行。格式如下：

```json
version: "3.8"
 services:
  mysql:
    image: mysql:5.7.25
    environment:
     MYSQL_ROOT_PASSWORD: 123 
    volumes:
     - "/tmp/mysql/data:/var/lib/mysql"
     - "/tmp/mysql/conf/hmy.cnf:/etc/mysql/conf.d/hmy.cnf"
  web:
    build: .
    ports:
     - "8090:8090"

```

上面的Compose文件就描述一个项目，其中包含两个容器：

- mysql：一个基于`mysql:5.7.25`镜像构建的容器，并且挂载了两个目录
- web：一个基于`docker build`临时构建的镜像容器，映射端口时8090



DockerCompose的详细语法参考官网：https://docs.docker.com/compose/compose-file/

DockerCompose文件可以看做是将 多个docker run命令 写到一个文件。



## 4.2.安装DockerCompose

```
curl -SL https://github.com/docker/compose/releases/download/v2.17.2/docker-compose-linux-x86_64 -o /usr/local/bin
# 所有用户 添加 x权限
chmod +x /usr/local/bin/docker-compose

docker-compose version
```



## 4.3.部署微服务集群

**需求**：将之前学习的cloud-demo微服务集群利用DockerCompose部署



**实现思路**：

① 查看课前资料提供的cloud-demo文件夹，里面已经编写好了docker-compose文件

② 修改自己的cloud-demo项目，将数据库、nacos地址都命名为docker-compose中的服务名

③ 使用maven打包工具，将项目中的每个微服务都打包为app.jar

④ 将打包好的app.jar拷贝到cloud-demo中的每一个对应的子目录中

⑤ 将cloud-demo上传至虚拟机，利用 docker-compose up -d 来部署



### 4.3.1.compose文件

查看课前资料提供的cloud-demo文件夹，里面已经编写好了docker-compose文件，而且每个微服务都准备了一个独立的目录：

![image-20210731181341330](docker/image-20210731181341330.png)

内容如下：

```yaml
version: "3.2"

services:
  nacos:
    image: nacos/nacos-server
    environment:
      MODE: standalone
    ports:
      - "8848:8848"
  mysql:
    image: mysql:5.7.25
    environment:
      MYSQL_ROOT_PASSWORD: 123
    volumes:
      - "$PWD/mysql/data:/var/lib/mysql"
      - "$PWD/mysql/conf:/etc/mysql/conf.d/"
  userservice:
    build: ./user-service
  orderservice:
    build: ./order-service
  gateway:
    build: ./gateway
    ports:
      - "10010:10010"
```

可以看到，其中包含5个service服务：

- `nacos`：作为注册中心和配置中心
  - `image: nacos/nacos-server`： 基于nacos/nacos-server镜像构建
  - `environment`：环境变量
    - `MODE: standalone`：单点模式启动
  - `ports`：端口映射，这里暴露了8848端口
- `mysql`：数据库
  - `image: mysql:5.7.25`：镜像版本是mysql:5.7.25
  - `environment`：环境变量
    - `MYSQL_ROOT_PASSWORD: 123`：设置数据库root账户的密码为123
  - `volumes`：数据卷挂载，这里挂载了mysql的data、conf目录，其中有我提前准备好的数据
- `userservice`、`orderservice`、`gateway`：都是基于Dockerfile临时构建的



查看mysql目录，可以看到其中已经准备好了cloud_order、cloud_user表：

![image-20210801095205034](docker/image-20210801095205034.png)

查看微服务目录，可以看到都包含Dockerfile文件：

![image-20210801095320586](docker/image-20210801095320586.png)

内容如下：

```dockerfile
FROM java:8-alpine
COPY ./app.jar /tmp/app.jar
ENTRYPOINT java -jar /tmp/app.jar
```





### 4.3.2.修改微服务配置

因为微服务将来要部署为docker容器，而容器之间互联不是通过IP地址，而是通过容器名。这里我们将order-service、user-service、gateway服务的mysql、nacos地址都修改为基于容器名的访问。

如下所示：

```yaml
spring:
  datasource:
    url: jdbc:mysql://mysql:3306/cloud_order?useSSL=false
    username: root
    password: 123
    driver-class-name: com.mysql.jdbc.Driver
  application:
    name: orderservice
  cloud:
    nacos:
      server-addr: nacos:8848 # nacos服务地址
```



### 4.3.3.打包

接下来需要将我们的每个微服务都打包。因为之前查看到Dockerfile中的jar包名称都是app.jar，因此我们的每个微服务都需要用这个名称。

可以通过修改pom.xml中的打包名称来实现，每个微服务都需要修改：

```xml
<build>
  <!-- 服务打包的最终名称 -->
  <finalName>app</finalName>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```

打包后：

![image-20210801095951030](docker/image-20210801095951030.png)

### 4.3.4.拷贝jar包到部署目录

编译打包好的app.jar文件，需要放到Dockerfile的同级目录中。注意：每个微服务的app.jar放到与服务名称对应的目录，别搞错了。

user-service：

![image-20210801100201253](docker/image-20210801100201253.png)

order-service：

![image-20210801100231495](docker/image-20210801100231495.png)

gateway：

![image-20210801100308102](docker/image-20210801100308102.png)

### 4.3.5.部署

最后，我们需要将文件整个cloud-demo文件夹上传到虚拟机中，理由DockerCompose部署。

上传到任意目录：

![image-20210801100955653](docker/image-20210801100955653.png)

部署：

进入cloud-demo目录，然后运行下面的命令：

```sh
docker-compose up -d
```

# 5.Docker镜像仓库 



## 5.1.搭建私有镜像仓库

参考课前资料《CentOS7安装Docker.md》



## 5.2.推送、拉取镜像

推送镜像到私有镜像服务必须先tag，步骤如下：

① 重新tag本地镜像，名称前缀为私有仓库的地址：192.168.150.101:8080/

 ```sh
docker tag nginx:latest 192.168.150.101:8080/nginx:1.0 
 ```



② 推送镜像

```sh
docker push 192.168.150.101:8080/nginx:1.0 
```



③ 拉取镜像

```sh
docker pull 192.168.150.101:8080/nginx:1.0 
```





