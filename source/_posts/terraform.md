---
title: terraform
date: 2025-03-21 11:34:02
---

Terraform

- Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently
-  Enables application software best practices to infrastructure
-  Compatible with many clouds and services


Terraform Architecture
=======
192.168.101.11

# Terraform是什么

基础设施即代码（Infrastructure as Code, IaC） 工具。

- 通过代码（如 .tf 文件）定义云资源，替代手动配置。
- 声明式语法：描述期望的最终状态，而非操作步骤（如“创建一个 EC2 实例”而非“运行命令启动实例”）。

核心组件

- Providers：连接云服务商的插件（如 AWS、Azure、Kubernetes）。  
- Resources：定义具体的云资源（如虚拟机、存储桶、网络）  
- State 文件：记录 Terraform 管理的资源实际状态（terraform.tfstate）。  
- init 下载并配置 Provider，初始化工作目录
- Plan 生成执行计划，显示资源创建/修改的详细步骤。
- Apply 根据计划创建/更新资源。
- destroy 删除所有由 Terraform 管理的资源。
- Data Source 查询现有资源信息（如获取已有的 VPC ID）



```yml
mkdir learn-terraform-aws-instance
vim main.tf
provider "aws" {
  region = "us-east-1"
    # AWS 访问密钥
    # AWS 秘钥
}

resource "aws_vpc" "web-vpc" {
  cidr_block = "10.0.0.0/16"  # VPC 的 CIDR 块
  tags = {
    Name = "web-vpc"  # VPC 的名称标签
  }
}

resource "aws_subnet" "web-subnet-1" {
  vpc_id = "${aws_vpc.web-vpc.id}"  # 关联的 VPC ID
  cidr_block = "10.0.0.0/24"  # 子网的 CIDR 块
  availability_zone = "us-east-1a"  # 可用区
  map_public_ip_on_launch = true # 自动分配公共IP
  tags = {
    Name = "web-subnet-1"  # 子网的名称标签
  }
}
resource "aws_internet_gateway" "web-igw" {
  vpc_id = aws_vpc.web-vpc.id
  tags = {
    Name = "web-igw"
  }
}
resource "aws_route_table" "web-rt" {
  vpc_id = aws_vpc.web-vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.web-igw.id
  }

  tags = {
    Name = "t-web-rt"
  }
}

resource "aws_route_table_association" "web-rt-assoc" {
  subnet_id      = aws_subnet.web-subnet-1.id
  route_table_id = aws_route_table.web-rt.id
}


resource "aws_instance" "free_tier_ec2_04271142" {
  ami           = "ami-04b4f1a9cf54c11d0" # Amazon Machine Image (AMI)
  instance_type = "t2.micro"  # 实例类型
  key_name      = "kptest"  # 替换为你的密钥对名称
  
  subnet_id     = aws_subnet.web-subnet-1.id   # 关联子网

  vpc_security_group_ids = [aws_security_group.allow_ssh.id]   # 关联安全组

  root_block_device {
    volume_size = 8 // 卷大小为8GB
    volume_type = "gp3"
  }

  tags = {
    Name = "FreeTierEC_202504271142"  # 实例的名称标签
  }
}

resource "aws_security_group" "allow_ssh" {
  name        = "allow_ssh_http"  # 安全组名称
  description = "Allow SSH HTTP inbound traffic"  # 安全组描述
  vpc_id = "${aws_vpc.web-vpc.id}"
  ingress {
    from_port   = 22  # 允许的起始端口
    to_port     = 22  # 允许的结束端口
    protocol    = "tcp"  # 协议
    cidr_blocks = ["0.0.0.0/0"]  # 允许从任何 IP 地址访问
  }
  # 新增HTTP规则
  ingress {
    from_port   = 80  # HTTP默认端口
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0  # 允许的起始端口
    to_port     = 0  # 允许的结束端口
    protocol    = "-1"  # 所有协议
    cidr_blocks = ["0.0.0.0/0"]  # 允许到任何 IP 地址
  }
}
```





