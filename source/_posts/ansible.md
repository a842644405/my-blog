---

title: ansible
date: 2023-06-01 15:50:36
tags:
---

### ansible install

```sh
 #安装依赖和ansible
 yum install -y epel-release
 yum install ansible -y
 
 #检查安装是否成功
 ansible --version
 
 #test
 ansible localhost -m ping 
```

### ansible configuration file

```sh
#修改主配置文件
vim /etc/ansible/ansible.cfg
#禁用每次执行ansbile命令检查ssh key host 
host_key_checking = False
#记录日志
log_path = /var/log/ansible.log

#inventory目录 配置被管理的主机
vim /etc/ansible/hosts

```





### ssh免密登录

```sh
#ansible主机生成密钥对 
ssh-keygen -t rsa
ls /root/.ssh
id_rsa(privite key)  id_rsa.pub

#发送公钥到远程主机
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.101.11

ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.101.12
```

### SSH密码登录

```sh
将密码直接写入ansible配置文件
格式例子如:
[root@keeper-01 ~]# cat /etc/ansible/hosts 
#ssh默认22端口时候
[maya]
keeper-01 ansible_ssh_host="192.168.14.128"ansible_ssh_user="root" ansible_ssh_pass="root"
maya-001-129 ansible_ssh_host="192.168.14.129" ansible_ssh_user="root" ansible_ssh_pass="root"
[mem]
mem1 ansible_ssh_host="192.168.14.130"ansible_ssh_user="root" ansible_ssh_pass="root"
mem2 ansible_ssh_host="192.168.14.131" ansible_ssh_user="root" ansible_ssh_pass="root"

#远端机器ssh的端口为var时，需多加一条配置
ansible_ssh_port=var

```



### ad-hoc模式（命令行模式）

command/shell module

```sh
ansible test -m command -a 'free -mh'
ansible test -m shell   -a 'df -h'
#执行远程主机的sh
ansible test -m shell -a 'sh /root/a.sh'
```

copy module 批量发送文件

```sh
#使用copy module 远程主机需依赖libselinux-python
ansible test -m shell   -a 'yum install libselinux-python -y'
ansible test -m copy    -a 'source  des'
```



### playbook模式（）


playbook是一个yml文件，由play和task两部分组成。

play:主要定义要操作主机或者主机组
task:主要定义对主机或主机组具体执行的任务，可以是一个任务，也可以是多个任务（模块)

playbook由一个或多个模块组成的，使用多个不同的模块，共同完成一件事情。



```
httpd-install.yml
#this is a playbook of ansible
#name:描述信息,tasks里有3个同级别的列表步骤
#yum:远端安装服务,yum模块安装服务(installed)
#copy:远端拷贝文件,copy模块传送文件到远端
#service:远端启动服务(started)
#remote_user: root是指定远程主机上使用的用户
#gather_facts: no是默认执行playbook时候,默认会收集目标主机的信息,禁用掉能提高效率
#

- hosts: test
  remote_user: root
  gather_facts: no
  tasks:
    - name: install httpd service
      yum: name=httpd,httpd-tools state=installed
    - name: configure httpd service
      copy: src=/root/httpd.conf dest=/etc/httpd.conf
    - name: service httpd start
      service: name=httpd state=started enabled=yes


```



```
#查看远程主机是否安装httpd
rpm -qa |grep httpd

#检查语法是否有误
ansible-playbook --syntax-check httpd-install.yaml

```























