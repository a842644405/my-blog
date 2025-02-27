---
title: ansible
date: 2023-06-01 15:50:36
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

#禁用每次执行ansbile命令检查ssh key host （默认值不用修改）
host_key_checking = False
#记录日志（默认值不用修改）
log_path = /var/log/ansible.log

#inventory目录 配置被管理的主机
vim /etc/ansible/hosts
## [webservers]
## alpha.example.org
## beta.example.org
## 192.168.1.100
## 192.168.1.110
[test] #test group
192.168.101.11
192.168.101.12
```



### ssh免密登录（推荐）

```sh
#ansible master生成密钥对 
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

#### **command/shell** module

```sh
ansible test -m command -a 'free -mh'
ansible test -m command -a 'ntpdate -u ntp.aliyun.com'
ansible test -m command -a 'date'
ansible test -m shell   -a 'df -h'
#执行远程主机的sh
ansible test -m shell -a 'sh /root/a.sh'
```

```
- name: Run a shell command with redirection
  shell: echo "Hello, World!" > /tmp/hello.txt
```



#### **copy** module 批量发送文件

```sh
#使用copy module 远程主机需依赖libselinux-python
ansible test -m shell   -a 'yum install libselinux-python -y'
ansible test -m copy    -a 'src=/etc/rancher/k3s/registries.yaml dest=/etc/rancher/k3s/registries.yaml'
```

```
- name: Run a shell command with redirection
  shell: echo "Hello, World!" > /tmp/hello.txt
```



#### shell yum 

#### file

#### 包管理

package

- **用途**：通用包管理器，根据系统自动选择合适的包管理工具

```yml
- name: Install a package using the default package manager
  package:
    name: git
    state: present
```

#### 服务管理

##### service 

用途：管理服务（启动、停止、重启等）。

```yml
- name: Start and enable the Apache service
  service:
    name: httpd
    state: started
    enabled: yes
```

##### systemd

用途：管理系统服务（使用 systemd 管理的服务）。

```yml
- name: Start and enable MySQL service
  systemd:
    name: mysql
    state: started
    enabled: yes
```

#### **用户和组管理模块**

##### user

##### group

#### **数据库管理模块**

##### mysql_db

```yml
- name: Create a MySQL database
  mysql_db:
    name: myappdb
    state: present
```

##### mysql_user



#### 容器管理模块

##### docker_container

##### docker_image

#### **其他常用模块**

#### `get_url`

```yml
- name: Download a file from the internet
  get_url:
    url: https://example.com/file.tar.gz
    dest: /tmp/file.tar.gz
```



#### `ping`



### playbook模式（）


playbook是一个yml文件，由play和task两部分组成。

play:定义要操作的 主机 或者 主机组
task:定义 对 主机或主机组 执行的任务，可以是一个任务，也可以是多个任务（模块)

playbook由一个或多个模块组成的，使用多个不同的模块，共同完成一件事情。

以下是一个简单的 Playbook 示例，展示了如何在一组主机上安装并启动 Nginx 服务：

nginx-install.yaml

```
---
- name: Install and start Nginx
  hosts: webservers
  become: yes
  tasks:
    - name: Ensure Nginx is installed
      apt:
        name: nginx
        state: present

    - name: Start Nginx service
      systemd:
        name: nginx
        state: started
        enabled: yes

```



```
#查看主机是否安装httpd
rpm -qa |grep nginx

#检查语法是否有误
ansible-playbook --syntax-check nginx-install.yaml

```







---

环境准备
服务器列表：
前端服务器：web1.example.com, web2.example.com
后端服务器：api1.example.com, api2.example.com
数据库服务器：db1.example.com
负载均衡器：lb1.example.com

inventory.yml

```
---
all:
  children:
    webservers:
      hosts:
        web1.example.com:
        web2.example.com:
    apiservers:
      hosts:
        api1.example.com:
        api2.example.com:
    databases:
      hosts:
        db1.example.com:
    loadbalancers:
      hosts:
        lb1.example.com:
```

创建一个名为 playbook.yml 的主 Playbook 文件

playbook.yml

```
---
- name: Deploy and configure microservices application
  hosts: all
  become: yes
  tasks:
    - name: Update package cache and install common packages
      apt:
        update_cache: yes
        pkg:
          - curl
          - git
          - python3-pip
          - docker.io
        state: present
       
```

创建一个名为 `deploy_web.yml` 的 Playbook 文件

```

```

deploy_api.yml

```

```

setup_db.yml

```

```

setup_lb.yml

```

```

setup_monitoring.yml 

```

```



主 Playbook 结合所有任务
最后，在主playbook.yml 中添加以下内容，引用所有子 Playbook：

```
---
- import_playbook: deploy_web.yml
- import_playbook: deploy_api.yml
- import_playbook: setup_db.yml
- import_playbook: setup_lb.yml
- import_playbook: setup_monitoring.yml 
```



### 运行 Playbook

在终端中运行以下命令来执行整个 Playbook：

```
ansible-playbook -i inventory.yml playbook.yml
```

















