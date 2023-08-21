---
title: Linux
---

# Linux相关概念

EPEL (**Extra Packages for Enterprise Linux**) is a third-party repository of packages for the Fedora-based and CentOS/RHEL-based Linux distributions. It provides additional software packages that are not included in the default package repositories of these distributions.The EPEL repository includes a wide range of software.





# Linux命令

## 快捷键

日常操作
Ctrl +r 查找历史命令
Ctrl + a \ Ctrl + e:移动光标到命令行首\行尾

Ctrl + w \Ctrl + k:删除光标之前\之后的内容



VIM操作
ZZ or :wq 文件保存并退出 

/ 查找



进程操作
Ctrl +c 强制终止程序的执行

Ctrl+z 挂起一个进程
Ctrl+d 终端中输入exit后回车



top命令中的快捷键
Shift +p:根据CPU使用率排序

Shift +m:根据内存占用排序



ctrl + c 强制停止

ctrl + d 退出或登出

ctrl + l clear 清空终端内容

history

!命令前缀，自动执行上一次匹配前缀的命令

光标移动快捷键

```
ctrl + a，跳到命令开头
ctrl + e，跳到命令结尾
ctrl + 键盘左键，向左跳一个单词
ctrl + 键盘右键，向右跳一个单词
```



## Linux的目录结构

![1684938168463](Linux%E8%AF%BE%E7%A8%8B%E7%AC%94%E8%AE%B0/1684938168463.png)

- / 根目录
- /home/test/a.txt 表示根目录下的home文件夹内有test文件夹内有a.txt

  

## 查看命令的帮助 ls --help

`命令 --help`查看命令的帮助手册



## 查看命令的详细手册 man ls

`man 命令`查看某命令的详细手册



## info ls



## ssh免密登录

```sh
#本机生成公钥私钥 在/root/.ssh下
ssh-keygen -t rsa
#copy id_ras.pub to 192.168.101.253./root/.ssh/authorized_keys
ssh-copy-id root@192.168.101.253
```

![image-20230727121922086](Linux课程笔记/image-20230727121922086.png)

## 软件安装命令

```sh
#CentOS系统使用
yum install/remove/search -y software
-y 自动确认
yum install mlocate -y

#Ubuntu系统使用
apt [install remove search] [-y] 软件名称
```

yum 和 apt 均需 root权限



docker指定版本安装

https://docs.docker.com/engine/install/centos/#install-using-the-repository

```sh
sudo yum install docker-ce-3:19.03.15-3.el7 docker-ce-cli-3:19.03.15-3.el7 containerd.io docker-buildx-plugin docker-compose-plugin
```



## ls命令

ls list 列出文件夹信息

`ls [-l -h -a] [参数]`

```sh
ls -l  ll 以列表形式查看
ll -h 显示kb
ll -a 显示隐藏文件(.开头的文件/文件夹)
ll -t 时间
ll -r 逆序
ls -F 分类显示
```



## pwd

```sh
#print work directory
pwd 
```



## cd命令

cd change directory

cd  进入当前用户目录

cd ./ 当前 	cd ..上一级



## HOME目录

每一个用户在Linux系统中都有自己的专属工作目录，称之为HOME目录。

- 普通用户的HOME目录，默认在：`/home/用户名`

- root用户的HOME目录，在：`/root`



SSH登陆终端后，默认是 用户的HOME目录



## 相对路径、绝对路径

- 相对路径

  以**当前目录**作为起点，去描述路径

  如 cd test/a.txt，表示进入当前目录内的test文件夹下的a.txt文件

- 绝对路径

  从`根`开始描述路径
  
  cd /etc/sysconfig/network-scripts/



## 特殊路径符

- `.`，  当前目录 	./a.txt 当前文件夹内的`a.txt`文件

- `..`，上级目录      ../../  上级的上级目录

- `~`，  用户的HOME目录 

  

## mkdir命令

功能：创建文件夹

语法：`mkdir [-p] 参数`

- 参数：被创建文件夹的路径
- 选项：-p，可选，表示**创建前置路径**   **mkdir /a/b/c -p**



## touch命令

创建文件

touch 被创建的文件路径

```sh
touch /etc/docker/daemon.json
vim /etc/docker/daemon.json
```

编辑新增的内容

{
  "registry-mirrors": ["https://0s0nyqqk.mirror.aliyuncs.com"]
}



## cat命令

查看文件内容

cat  被查看的文件路径

```sh
cat /etc/docker/daemon.json
```



## more命令

查看文件，可以支持翻页查看

more 被查看的文件路径

- 在查看过程中：
  - **`空格`键翻页**
  - **`q`退出查看**



## cp命令

复制文件、文件夹

`cp [-r] source destination`

- 选项：-r 复制文件夹使用

示例：

- cp a.txt b.txt	复制当前目录下的 a.txt为b.txt
- cp a.txt test/    复制当前目录a.txt到test文件夹内
- cp -r test test2 复制文件夹test到当前文件夹内为test2内



## mv命令

移动文件、文件夹

`mv source destination`

- 参数1：被移动的
- 参数2：要移动去的地方，参数2如果不存在，则会进行改名

```sh
cd /etc/yum.repos.d
mv CentOS-Base.repo CentOS-Base.repo.bak
#阿里源文件下载
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 
```



## rm

功能：删除文件、文件夹

语法：`rm [-r -f] 参数 参数`

- 参数：支持多个，每一个表示被删除的，空格进行分隔
- 选项：-r，删除文件夹使用
- 选项：-f，强制删除，不会给出确认提示，一般root用户会用到

> rm命令很危险，一定要注意，特别是切换到root用户的时候。



## which

查看命令的程序本体**文件路径**

which 命令 

```sh
[root@localhost yum.repos.d]# which cd
/usr/bin/cd
```



## find

find 起始路径 -name 搜索关键字

关键字支持通配符， 比如：*test表示搜索任意以test结尾的文件

```sh
find / -name docker
find /usr/local -name  *bin*
```



## locate



**CentOS7默认没有安装该命令** 

```sh
yum install mlocate -y
#初次使用必须先创建数据库
updatedb

locate -ie abc.txt 
#如果 abc.txt已经删除了，使用-e会检查文件是否真实存在，而不必updatedb；
#-i，忽略大小写
#在/etc中查找类似profile的文件
locate /etc/*profile 
```

locate通过数据库（/var/lib/mlocate/mlocate.db文件）来查找文件（当天新增的文件查询不到 使用sudo updatedb 可以立即更新数据库）



## grep命令

功能：过滤关键字

语法：`grep [-n] 关键字 文件路径`

- 选项-n，可选，表示在结果中显示匹配的行的行号。
- 参数，关键字，必填，表示过滤的关键字，带有空格或其它特殊符号，建议使用””将关键字包围起来
- 参数，文件路径，必填，表示要过滤内容的文件路径，可作为内容输入端口



> 参数文件路径，可以作为管道符的输入



## wc命令

统计 word count

`wc [-c -m -l -w] 文件路径`

- 选项，-c，统计bytes数量
- 选项，-m，统计字符数量
- 选项，-l，统计行数
- 选项，-w，统计单词数量
- 参数，文件路径，被统计的文件，可作为内容输入端口



> 参数文件路径，可作为管道符的输入



## 管道符|

`|`

功能：将符号左边的结果，作为符号右边的输入

示例：

`cat a.txt | grep itheima`，将cat a.txt的结果，作为grep命令的输入，用来过滤`itheima`关键字



支持嵌套：

`cat a.txt | grep itheima | grep itcast`



## echo命令

输出内容

语法：`echo 参数`

- 参数：被输出的内容

```sh
echo haha 	
echo $var
echo haha > a # 覆
echo haha >> a # 追加
```



## 反引号	

被两个反引号包围的内容，会作为命令执行

示例：

- echo \`pwd\` 相当于输入pwd



## tail命令

功能：查看文件尾部内容

语法：`tail [-f] 参数`

- 参数：被查看的文件
- 选项：-f，持续跟踪文件修改

tail -f docker.log



## head命令

功能：查看文件头部内容

语法：`head [-n] 参数`

- 参数：被查看的文件
- 选项：-n，查看的行数

head -3 docker.log



## 重定向符

功能：将符号左边的结果，输出到右边指定的文件中去

- `>` 覆盖输出
- `>>` 追加到下一行输出



## vi编辑器

### 命令模式快捷键

![image-20221027215841573](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027215841.png)

![image-20221027215846581](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027215846.png)

![image-20221027215849668](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027215849.png)

### 底线命令快捷键

![image-20221027215858967](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027215858.png)



## systemctl

控制 系统服务的 启停等

语法：`systemctl start | stop | restart | disable | enable | status 服务名`

- start
- stop
- status
- disable 取消开机自启
- enable 开机自启
- restart

系统内置的服务比较多，比如：

```
NetworkManager 主网络服务
network 	   副网络服务
firewalld      防火墙服务
sshd		   ssh服务 secure shell
```



## 软链接 ln -s 

创建文件、文件夹的软链接（快捷方式）

`ln -s 参数1 参数2`

- 参数1：被链接的
- 参数2：要链接去的地方（快捷方式的名称和存放位置）

```
ln -s /etc/yum.conf ~/yum.conf
```



## 日期

语法：`date [-d] [+格式化字符串]`

- -d 按照给定的字符串显示日期，一般用于日期计算

- 格式化字符串：通过特定的字符串标记，来控制显示的日期格式
  - %Y   年%y   年份后两位数字 (00..99)
  - %m   月份 (01..12)
  - %d   日 (01..31)
  - %H   小时 (00..23)
  - %M   分钟 (00..59)
  - %S   秒 (00..60)
  - %s   自 1970-01-01 00:00:00 UTC 到现在的秒数



示例：

- 按照2022-01-01的格式显示日期

  ![image-20221027220514640](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027220514.png)

- 按照2022-01-01 10:00:00的格式显示日期

  ![image-20221027220525625](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027220525.png)

- -d选项日期计算

  ![image-20221027220429831](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027220429.png)

  - 支持的时间标记为：

    ![image-20221027220449312](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027220449.png)





## 时区

修改时区为中国时区

```sh
rm -f /etc/localtime
sudo ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

```



## 网络时间ntp

Network Time Protocol

```sh
#查看设置系统时区
rm -rf /etc/localtime
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

#安装ntp并同步时间
yum install -y ntp
ntpdate -u ntp.aliyun.com

sudo systemctl enable ntpd
sudo systemctl start ntpd

#查看系统硬件时间
hwclock --show
#从当前系统时间设置硬件时间
hwclock -w
```





## ip地址 

ifconfig

格式：a.b.c.d

- abcd为0~255的数字

特殊IP：

- 127.0.0.1 本机
- 0.0.0.0
  - 可以表示本机
  - 也可以表示任意IP（看使用场景）





## 主机名 hostname

查看当前主机名

```
hostname
```



设置主机名

```sh
hostnamectl set-hostname newHostName
cat /etc/hosts
```



## 配置VMware固定IP

```sh
vim /etc/sysconfig/network-scripts/ifcfg-ens33
```

```shell
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"			# 改为static，固定IP
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="1b0011cb-0d2e-4eaa-8a11-af7d50ebc876"
DEVICE="ens33"
ONBOOT="yes"
IPADDR="192.168.101.111"		# IP地址，自己设置，要匹配网络范围
NETMASK="255.255.255.0"		# 子网掩码，固定写法255.255.255.0
GATEWAY="192.168.101.2"		# 网关，要和VMware中配置的一致
DNS1="192.168.101.2"			# DNS1服务器，和网关一致即可
```

## 防火墙

硬件防火墙

软件防火墙

​	iptables

​	firewalld

![image-20230731095016316](Linux课程笔记/image-20230731095016316.png)



显示当前活动的防火墙配置：

```sh
sudo firewall-cmd --list-all
```



获取特定区域（zone）的配置：

```sh
sudo firewall-cmd --zone=<zone> --list-all
```

将 `<zone>` 替换为您要获取配置的区域名称，例如 `public`、`internal` 等。



临时添加防火墙规则（仅在运行时生效，重启后失效）：

```sh
sudo firewall-cmd --zone=public --add-port=80/tcp
sudo firewall-cmd --zone=public --add-service=http
```



永久添加防火墙规则（持久生效）：

```sh
sudo firewall-cmd --zone=<zone> --add-<service/port> --permanent
```



移除防火墙规则：

```sh
sudo firewall-cmd --zone=<zone> --remove-<service/port>
```

将 `<zone>` 替换为适用的区域名称，`<service/port>` 替换为要移除的服务或端口。



## 网络相关命令

![image-20230725103457086](Linux课程笔记/image-20230725103457086.png)

### 网卡配置

```sh
#网卡命名规则受biosdevname和net.ifnames两个参数影响
#编辑/etc/default/grub文件，增加 biosdevname=0 net.ifnames=O
#更新grub
grub2-mkconfig -o / boot/grub2/grub.cfg

#重启生效
reboot
```

<img src="Linux%E8%AF%BE%E7%A8%8B%E7%AC%94%E8%AE%B0/1686106222262.png" alt="1686106222262" style="zoom:80%;" />

```sh
#查看网卡物理连接情况
[root@localhost grub2]# mii-tool ens33
ens33: negotiated 1000baseT-FD flow-control, link ok

#查看网关
route -n
```

![image-20230615142135785](Linux%E8%AF%BE%E7%A8%8B%E7%AC%94%E8%AE%B0/image-20230615142135785.png)

参数解析:
Destination表示网络号
Gateway 网关地址，网络是通过该IP出口，如果显示0.0.0.0，表示该路由信息，是从本机转发出去的

Genmask 子网掩码地址，IP地址配合子网掩码，才是一个完整的网络信息
Flags:路由标记，标记当前的网络状态
U Up运行的状态
G 表示是一个网关路由器

H 表示这个网关是一个主机
！表示当前这个路由已禁止

```
ip addr ls
ifconfig
ip addr add 10.0.0.1/24 dev eth1
ifconfig eth1 10.0.0.1 netmask 255.255.255.0. ip route add 10.0.0/24 via 192.168.0.1
route add -net 10.0.0.0 netmask 255.255.255.0 gw 192.168.0.1
```



#### 端口

1. 端口
   计算机和外部交互的出入口，可以分为 物理端口 和 虚拟端口
   物理端口：USB、HDMI、DP、VGA、RJ45等
   虚拟端口：操作系统 和 外部 交互的出入口
   IP+port 锁定 要交互的程序
2. 端口的划分
   公认端口：1~1023，用于系统内置或常用知名软件绑定使用
   注册端口：1024~49151，用于松散绑定使用（用户自定义）
   动态端口：49152~65535，用于临时使用（多用于出口）
3. 查看端口占用

```sh
#查看指定IP的对外暴露端口
nmap IP地址


netstat -anp | grep 3306
#端口号，查看本机指定端口号的占用情况
```

### 网络故障排除

#### ping

测试网络是否联通

语法：`ping [-c num] url`

 ping -c 3 www.baidu.com   ping3次



#### netstat

查看**网络状态和统计信息** ，显示当前TCP/IP连接、路由表、网络接口信息等。常用于故障排除，诊断网络问题以及监视网络活动。

常见的netstat选项：

- -a：显示所有的网络连接（包括监听和非监听状态）。
- -t：只显示TCP协议的连接。
- -u：只显示UDP协议的连接。
- -n：使用数字形式显示网络地址和端口号，不要解析成名称。
- -p：显示与连接相关的程序名和进程ID。
- -r：显示路由表信息。
- -i：显示网络接口状态。

例如，使用netstat -tunap命令可列出所有正在进行的TCP和UDP连接。该命令列出每个连接的本地和远程IP地址、端口号、协议和当前状态，以及与每个连接相关联的程序名称和进程ID。

netstat是一个强大的网络工具，能够提供有关网络连接和流量的详细信息，使管理员和用户能够定位和解决各种网络问题。





## ps命令

查看进程信息

语法：`ps -ef`，查看全部进程信息，可以搭配grep做过滤：`ps -ef | grep xxx`

```
UID：进程所属的用户ID
PID：进程的进程号ID
PPID：进程的父ID（启动此进程的其它进程）
C：此进程的CPU占用率（百分比）
STIME：进程的启动时间
TTY：启动此进程的终端序号，如显示?，表示非终端启动
TIME：进程占用CPU的时间
CMD：进程对应的名称或启动路径或启动命令
```

## 杀死进程 kill -9 pid

![image-20221027221303037](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027221303.png)







## wget命令

wget是非交互式的文件下载器，下载网络文件
语法：wget -b url
-b 可选，后台下载，会将日志写入到当前工作目录的wget-log文件
url 下载链接
示例：
下载apache-hadoop 3.3.0版本：wget http://archive.apache.org/dist/hadoop/common/hadoop-3.3.0/hadoop-3.3.0.tar.gz

在后台下载：wget -b http://archive.apache.org/dist/hadoop/common/hadoop-3.3.0/hadoop-3.3.0.tar.gz
通过tail命令可以监控后台下载进度：tail -f wget-log



## curl

发送http网络请求,用于下载文件、获取信息等

curl [-o] url 
-O 用于下载文件，当url是下载链接时，可以使用此选项保存文件



![image-20230615165149036](Linux%E8%AF%BE%E7%A8%8B%E7%AC%94%E8%AE%B0/image-20230615165149036.png)

```sh
#获取主机的公网ip
curl cip.cc
 
#先切换到对应的目录再下载文件
curl -O https://mirrors.tuna.tsinghua.edu.cn/virtualbox/LATEST.TXT
```



## top命令

查看主机运行状态

语法：`top`

shift + p

shift + c

可用选项：

![image-20221027221340729](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027221340.png)



交互式模式中，可用快捷键：

![image-20221027221354137](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027221354.png)



## df命令

查看磁盘占用

![image-20221027221413787](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027221413.png)



## iostat命令

查看CPU、磁盘的相关信息

![image-20221027221439990](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027221440.png)

![image-20221027221514237](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027221514.png)



## sar命令 查看网络统计

![image-20221027221545822](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027221545.png)



## 环境变量 export 变量名=变量值

- 临时设置：export 变量名=变量值
- 永久设置：
  - 针对用户，设置用户HOME目录内：`.bashrc`文件
  - 针对全局配置，vim /etc/profile
  - 配置完成后，source /etc/profile 立刻生效



### PATH变量

记录了 执行程序的搜索路径

可以将自定义路径加入PATH内，实现自定义命令在任意地方均可执行的效果



## $符号 取出变量的值

`$变量名`

示例：

`echo $PATH`，输出PATH环境变量的值

`echo ${PATH}ABC`，输出PATH环境变量的值以及ABC

如果变量名和其它内容混淆在一起，可以使用${}



## 压缩解压

### 压缩

`tar -zcvf 压缩包 被压缩1...被压缩2...被压缩N`

-z表示使用gzip，可以不写



`zip [-r] 参数1 参数2 参数N`

```shell
zip test.zip a.txt b.txt c.txt 	#将3个txt压缩到test.zip

zip -r test.zip test itheima a.txt #将test、itheima两个文件夹和a.txt文件，压缩到test.zip文件内


```

![image-20221027221906247](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027221906.png)



### 解压

`tar -zxvf 被解压的文件 -C 要解压去的地方`

- -z表示使用gzip，可以省略
- -C，可以省略，指定要解压去的地方，不写解压到当前目录







`unzip [-d] 参数`

![image-20221027221939899](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027221939.png)





## su命令 switch user

切换用户

语法：`su [-] [用户]`

![image-20221027222021619](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027222021.png)



## sudo命令

![image-20221027222035337](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027222035.png)



比如：

```shell
itheima ALL=(ALL)       NOPASSWD: ALL
```

visudo命令

配置如上内容，可以让itheima用户，无需密码直接使用`sudo`



## genent命令

`getent` 的全称是 "get entries from administrative database"，从系统的各种数据库（如 passwd、group 和 hosts）中获取条目的命令。使用 `getent` 命令可以在 Linux 系统上检索与用户、组和网络信息相关的条目。

```sh
#查看系统全部的用户组
getent group
```

![image-20221027222446514](https://image-set.oss-cn-zhangjiakou.aliyuncs.com/img-out/2022/10/27/20221027222446.png)

组名称:组认证(显示为x):组ID

```sh
#查看系统全部的用户
getent passwd
```

<img src="Linux%E8%AF%BE%E7%A8%8B%E7%AC%94%E8%AE%B0/1684938626013.png" alt="1684938626013" style="zoom:50%;" />

用户名:密码(x):用户ID:组ID:描述信息(无用):HOME目录:执行终端(默认bash)



![image-20230521204016977](Linux课程笔记/image-20230521204016977.png)

文件、文件夹的

权限控制信息
所属用户
所属用户组

![image-20230615113352091](Linux%E8%AF%BE%E7%A8%8B%E7%AC%94%E8%AE%B0/image-20230615113352091.png)





## 用户组管理

```sh
groupadd 用户组名
groupdel 用户组名	
```



## 用户管理

```sh
以下命令需root用户执行

创建用户
useradd [-g -d] 用户名
-g指定 用户的组，不指定-g，会创建同名组并自动加入，指定-g需要组已经存在，如已存在同名组，必须使用-g
-d指定 用户HOME路径，不指定，HOME目录默认在：/home/用户名

useradd -g root vijay

删除用户
userdel [-r] 用户名
-r 删除用户的HOME目录，
不使用-r，删除用户时，HOME目录保留

查看用户所属组
id [用户名]
参数：用户名，被查看的用户，如果不提供则查看自身

修改用户所属组
usermod -aG 用户组 用户名，将指定用户加入指定用户组
modify

```

## chmod

change mode 

用于修改文件、文件夹的访问权限

```
chmod 777 filename
chmod -R 777 文件夹及内部所有
```

`chmod [-R] 权限 参数`

权限，要设置的权限，比如755，表示：`rwx r-x r-x`

参数，被修改的文件、文件夹

-R 设置文件夹和其内部全部内容一样生效

x 1

w 2

r 4

![1684938749252](Linux%E8%AF%BE%E7%A8%8B%E7%AC%94%E8%AE%B0/1684938749252.png)





## chown

change owner

修改文件、文件夹**所属用户、组**

语法：`chown [-R] [用户][:][用户组] 文件或文件夹`

```sh
#此命令只适用于root用户执行
chown root hello.txt，将hello.txt所属用户修改为root
chown :root hello.txt，将hello.txt所属用户组修改为root
chown root:itheima hello.txt，将hello.txt所属用户修改为root，用户组修改为itheima
chown -R root test，将文件夹test的 所属用户 修改为 root并对文件夹内全部内容应用同样规则
```





## env命令

查看系统全部的环境变量

env

```
查看环境变量：
echo $VARIABLE_NAME：显示指定环境变量的值。
例如，echo $PATH 显示 PATH 环境变量的值。

printenv：列出所有环境变量及其值。

设置临时环境变量：
VARIABLE_NAME=value：设置临时环境变量的值。例如，LANG=en_US.UTF-8 设置 LANG 环境变量为 en_US.UTF-8。

设置永久环境变量（对当前用户）：
编辑 ~/.bashrc 或 ~/.bash_profile 文件，在文件末尾添加 export VARIABLE_NAME=value，然后保存并退出。例如，export JAVA_HOME=/usr/lib/jvm/java-11 设置 JAVA_HOME 环境变量为 /usr/lib/jvm/java-11。
运行 source ~/.bashrc 或 source ~/.bash_profile 使修改生效。

设置永久环境变量（对所有用户）：
编辑 /etc/environment 文件，在文件中添加 VARIABLE_NAME=value 行。例如，JAVA_HOME="/usr/lib/jvm/java-11" 设置 JAVA_HOME 环境变量为 /usr/lib/jvm/java-11。
删除环境变量：

unset VARIABLE_NAME：删除临时环境变量或当前用户的永久环境变量。
编辑相应的配置文件，将环境变量的定义删除。
```

