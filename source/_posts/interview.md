---
title: interview
date: 2026-08-17 09:48:00
---

# http和https的本质区别

**HTTP 和 HTTPS 本质区别：**

|            | HTTP               | HTTPS                   |
| ---------- | ------------------ | ----------------------- |
| 全称       | 超文本传输协议     | HTTP + TLS/SSL 安全协议 |
| 数据传输   | 明文               | 加密传输                |
| 端口       | 80                 | 443                     |
| 身份认证   | 无法确认服务器身份 | 通过证书验证服务器身份  |
| 数据完整性 | 容易被篡改         | 防止中间人篡改          |
| 性能       | 快一点             | TLS握手增加少量开销     |

**本质一句话：**

> HTTP = 浏览器和服务器直接通信（不加密）
> HTTPS = HTTP + TLS加密层（保证机密性、完整性、身份可信）

HTTPS解决三个核心问题：

1. **加密**
   - 防止别人抓包看到账号密码、Token等
2. **认证**
   - 确认访问的是正规服务器，不是假网站
3. **完整性**
   - 防止数据在传输过程中被修改

调用链理解：

```
HTTP:
浏览器
  |
  | 明文HTTP
  ↓
服务器


HTTPS:
浏览器
  |
  | TLS握手
  ↓
建立加密通道
  |
  | 加密HTTP数据
  ↓
服务器
```

在企业微服务架构中：

```
用户
 ↓ HTTPS(443)
SLB/网关
 ↓ HTTP或HTTPS
微服务
```

通常公网入口使用 **HTTPS**，内部服务之间可能使用 **HTTP**（因为在可信内网环境），也可能全链路 HTTPS（零信任架构）。



# 网络模型理解

用一次 **浏览器访问 `https://www.baidu.com`** 的完整链路举例（假设IP、端口均为虚构），把 DNS、TCP、IP、MAC 都串起来。

假设：

客户端电脑：

```
电脑IP:      192.168.1.100
网关路由器IP: 192.168.1.1
客户端临时端口: 52000
MAC地址:     AA-AA-AA-AA-AA-AA
```

百度服务器：

```
域名:
www.baidu.com

DNS解析结果:
www.baidu.com → 110.242.68.66

服务器IP:
110.242.68.66

HTTPS端口:
443
```

------

1. 浏览器输入域名

输入：

```
https://www.baidu.com
```

浏览器不知道服务器IP：

```
www.baidu.com = ?
```

需要DNS解析。

------

2. DNS解析过程

第一步：查询本地缓存

浏览器缓存：

```
www.baidu.com ?
```

没有。

操作系统缓存：

```
www.baidu.com ?
```

没有。

发送DNS请求。

------

第二步：发送DNS请求

DNS服务器假设：

```
DNS服务器:
223.5.5.5
```

数据：

```
源IP:
192.168.1.100

目标IP:
223.5.5.5

源端口:
53000

目标端口:
53

协议:
UDP
```

请求：

```
请告诉我：
www.baidu.com 的IP是多少？
```

返回：

```
www.baidu.com

↓

110.242.68.66
```

------

3. 建立HTTPS连接（TCP三次握手）

浏览器现在知道：

```
目标:
110.242.68.66:443
```

开始TCP连接。

------

第一次握手

客户端发送：

```
源IP:
192.168.1.100

源端口:
52000


目标IP:
110.242.68.66

目标端口:
443


TCP:
SYN
```

意思：

> 你好，我想连接你的443端口

------

第二次握手

服务器返回：

```
源IP:
110.242.68.66

源端口:
443


目标IP:
192.168.1.100

目标端口:
52000


TCP:
SYN + ACK
```

意思：

> 收到了，我可以连接

------

第三次握手

客户端：

```
ACK
```

TCP连接建立。

现在形成一个连接：

```
192.168.1.100:52000

        ↓

110.242.68.66:443
```

这就是一个TCP连接。

------

4. HTTPS TLS握手

TCP建立后：

浏览器：

```
我要HTTPS通信
```

发送：

```
Client Hello
```

服务器：

```
返回数字证书
baidu.com证书
```

双方协商：

```
加密算法
会话密钥
```

建立加密通道。

------

5. 发送HTTP请求

浏览器发送：

```
GET /

Host:
www.baidu.com
```

实际数据：

```
应用层:

GET / HTTP/1.1
Host: www.baidu.com
Cookie: xxx
```

------

6. 数据封装过程

应用层：

```
HTTP数据

GET /
Host:www.baidu.com
```

↓

传输层 TCP：

增加：

```
源端口:
52000

目标端口:
443
```

变成：

```
TCP Segment
192.168.1.100:52000
        |
        |
110.242.68.66:443
```

↓

网络层 IP：

增加：

```
源IP:
192.168.1.100

目标IP:
110.242.68.66
```

↓

数据链路层：

增加MAC：

因为目标服务器不在你的局域网：

所以第一跳不是百度服务器，而是你的路由器。

查询：

```
下一跳是谁？

192.168.1.1
```

ARP（Address Resolution Protocol） 地址解析协议获取：

```
192.168.1.1

↓

路由器MAC:
BB-BB-BB-BB-BB-BB
```

封装：

```
源MAC:
AA-AA-AA-AA-AA-AA


目标MAC:
BB-BB-BB-BB-BB-BB
```

------

7. 网卡发送

最终离开电脑的数据：

```
以太网帧

MAC:
AA-AA-AA-AA-AA-AA
        |
        |
BB-BB-BB-BB-BB-BB


IP:
192.168.1.100
        |
        |
110.242.68.66


TCP:
52000
        |
        |
443


HTTPS:
加密后的HTTP数据
```

------

8. 到达互联网

经过：

```
电脑
 |
交换机
 |
家庭路由器
 |
运营商路由器
 |
骨干网络
 |
百度机房路由器
 |
百度服务器
```

注意：

中间路由器只看：

```
目标IP:
110.242.68.66
```

不断转发。

MAC地址每一跳都会变化：

例如：

第一跳：

```
你的电脑MAC
 ↓
家庭路由器MAC
```

下一跳：

```
家庭路由器MAC
 ↓
运营商路由器MAC
```

但是：

IP基本保持：

```
192.168.1.100
        ↓
110.242.68.66
```

（公网出口可能经过NAT转换）

------

最终记忆版

一次HTTPS访问：

```
输入域名
    |
    ↓
DNS
www.baidu.com
    |
    ↓
110.242.68.66


TCP:
192.168.1.100:52000
        |
        |
110.242.68.66:443


IP:
找服务器在哪里


MAC:
找下一跳是谁


网卡:
变成电信号发送
```

一句话总结：

> DNS负责“问名字对应哪个IP”，TCP负责“建立可靠连接”，IP负责“找到服务器位置”，MAC负责“找到下一跳设备”，HTTPS负责“把内容安全传过去”。

放到你熟悉的 Kubernetes 场景：

```
浏览器
 ↓ HTTPS
SLB
 ↓ TCP/IP
Ingress Gateway
 ↓ HTTP
Service
 ↓ ClusterIP
iptables/IPVS
 ↓
Pod IP
 ↓
Java应用
```

本质没有变化，K8s 只是把传统网络里的“服务器”变成了“Pod”。



# 企业微服务系统访问链路(HTTP和HTTPS)

## 一、整体架构

假设：

域名：

```
www.znyth.com
```

架构：

```
                 用户浏览器
                      |
                      |
                 公司内部DNS
                      |
                      |
                 阿里云SLB
                      |
                      |
          Spring Cloud Gateway集群
                      |
                      |
              Nacos服务发现
                      |
                      |
        +-------------+-------------+
        |             |             |
   auth-service  user-service  order-service
                      |
                MySQL/Redis/MQ
```

---

## 二、HTTP访问流程

### 场景

用户访问：

```
http://www.znyth.com
```

特点：

```
协议: HTTP
端口: 80
数据: 明文传输
```

---

### 1. DNS解析

用户输入：

```
http://www.znyth.com
```

浏览器查询：

```
www.znyth.com
```

DNS返回：

```
10.10.10.100

DNS查询核心链路：
应用/浏览器缓存
        ↓
操作系统解析机制
        ↓
hosts等本地静态解析
        ↓
本地DNS / DNS Resolver
        ↓
查询缓存
        ↓
如果缓存未命中
        ↓
根DNS
        ↓
顶级域DNS（如 .com）
        ↓
权威DNS（如 znyth.com）
        ↓
返回IP
        ↓
DNS Resolver缓存结果（按TTL）
        ↓
返回客户端

① 查到即止
② 客户端通常不直接访问根/顶级/权威DNS
③ DNS逐级查询主要由Resolver完成
DNS不是“固定必须走完一条链路”，
而是“从近到远查找，哪里有可用答案就在哪里结束”。
```

| 缓存             | 在哪里            | 给谁服务        |
| ---------------- | ----------------- | --------------- |
| 浏览器DNS缓存    | 浏览器            | 当前浏览器      |
| OS DNS缓存       | 用户电脑          | 当前电脑        |
| DNS Resolver缓存 | DNS服务器 8.8.8.8 | 大量客户端      |
| 权威DNS记录      | 权威DNS           | 提供正式DNS记录 |

这个IP：

不是Gateway。

而是：

```
SLB VIP
```

此时：

```
浏览器
目标: 10.10.10.100:80
```

---

### 2. TCP连接建立

HTTP基于TCP。

三次握手：

```
SYN 是“我想连”，ACK 是“我收到了” 本质是TCP 协议里两个控制信号位
客户端
  |
  | SYN
  |
SLB

客户端
  |
  | SYN+ACK
  |
SLB

客户端
  |
  | ACK
  |
SLB
```

TCP连接建立。

---

### 3. 浏览器发送HTTP请求

例如：

```
GET / HTTP/1.1
Host: www.znyth.com
User-Agent: Chrome
Cookie: xxx
```

发送到：

```
SLB:80
```

---

### 4. SLB接收请求

SLB配置：

监听：

```
协议: HTTP
端口: 80
```

后端服务器组：

```
Gateway-1  10.10.20.11:8080
Gateway-2  10.10.20.12:8080
```

SLB-本质上是一个**动态调度器**，核心任务是在一组后端服务器之间高效、公平地分发请求。

---

### 5. SLB健康检查

SLB周期检测：

```
GET http://10.10.20.11:8080/health
```

返回：

```
200 OK
```

加入负载池。

异常：

```
timeout / 500 / 连接失败
```

摘除。

---

### 6. SLB选择Gateway

例如：

算法：

```
加权轮询
```

当前：

```
Gateway-1 权重5
Gateway-2 权重5
```

请求：

```
用户A → Gateway-1
用户B → Gateway-2
```

---

### 7. Gateway处理

请求：

```
GET /
Host: www.znyth.com
```

Gateway：

执行Filter：

```
认证
日志
限流
路由
```

---

### 8. Gateway转发微服务

例如：

访问：

```
/user/list
```

Gateway配置：

```
/user/** → user-service
```

通过：

```
Nacos
```

发现：

```
user-service
10.10.30.11:8080
10.10.30.12:8080
```

客户端负载：

```
Gateway → LoadBalancer → 选择实例
```

---

### HTTP完整链路

```
浏览器
  |
  | HTTP :80
  |
公司DNS
  |
  |
SLB VIP (10.10.10.100)
  |
  |
SLB监听80
  |
  |
Gateway
  |
  |
Nacos
  |
  |
微服务
  |
  |
数据库
```

---

### HTTP特点总结

优点：

```
简单
性能高
```

缺点：

```
明文
容易被窃听
```

例如：

请求：

```
username=admin
password=123456
```

网络中可能看到。

所以生产环境基本不用HTTP公网传输。

---

## 三、HTTPS访问流程

### 场景

访问：

```
https://www.znyth.com
```

特点：

```
协议: HTTPS
端口: 443
加密: TLS
```

---

### HTTPS比HTTP多了什么？

HTTP：

```
TCP → HTTP
```

HTTPS：

```
TCP → TLS握手 → HTTP
```

也就是：

```
HTTP + TLS = HTTPS
```

---

### 1. DNS解析

一样：

```
www.znyth.com → 10.10.10.100
```

访问：

```
SLB:443
```

---

### 2. TCP三次握手

一样：

```
客户端 → SLB 建立TCP连接
```

---

### 3. TLS握手（核心）（Transport Layer Security）

TCP建立后：

开始TLS。

---

#### 第一步：客户端Hello

浏览器发送：

```
Client Hello
包含：
  支持TLS版本
  加密算法
  随机数
```

---

#### 第二步：服务器返回证书

SLB返回：

```
Server Hello + SSL证书
```

证书：

例如：

```
www.znyth.com
有效期: 2026-2027
CA签发
```

浏览器验证：

```
域名是否匹配
证书是否可信
是否过期
```

---

#### 第三步：协商密钥

双方生成：

```
会话密钥
```

以后：

HTTP数据：

全部加密。

---

### 4. HTTPS请求

浏览器发送：

实际上：

```
GET /user/list
```

但是：

网络看到：

```
加密数据 xxxxxxxxxxxxx
```

---

### 5. SSL在哪里解密？

企业生产最常见：

#### SSL卸载模式

架构：

```
浏览器
  HTTPS
   |
   |
  SLB
   |
 TLS解密
   |
  HTTP
   |
Gateway
```

也就是：

```
HTTPS → SLB → HTTP → Gateway
```

为什么这么做？

因为：

SLB专门处理加密。

Gateway不用承担大量TLS计算。

---

### HTTPS完整链路

```
浏览器
  |
  | HTTPS:443
  |
DNS
  |
  |
SLB
  |
  | TLS证书验证
  |
SSL卸载
  |
  | HTTP
  |
Gateway
  |
  |
Nacos
  |
  |
微服务
  |
  |
数据库
```

---

## 四、HTTP和HTTPS核心区别

| 项目     | HTTP | HTTPS   |
| -------- | ---- | ------- |
| 端口     | 80   | 443     |
| 加密     | 无   | TLS     |
| 证书     | 无   | 需要    |
| 数据     | 明文 | 密文    |
| 握手     | TCP  | TCP+TLS |
| 性能     | 稍快 | 稍慢    |
| 安全     | 低   | 高      |
| 生产使用 | 少   | 主流    |

---

## 五、生产环境常见HTTPS部署方式

### 方式1：SLB SSL卸载（最常见）

```
用户
  HTTPS
   ↓
SLB (HTTPS证书)
   ↓
  HTTP
   ↓
Gateway
```

优点：

* Gateway压力小
* 证书统一管理
* 运维简单

---

### 方式2：SLB四层转发

```
用户
  HTTPS
   ↓
SLB
   ↓
Gateway (HTTPS)
```

特点：

SLB不知道HTTP内容。

SSL由Gateway处理。

---

### 方式3：全链路HTTPS

```
用户
  HTTPS
   ↓
SLB (HTTPS)
   ↓
Gateway (HTTPS)
   ↓
微服务 (HTTPS)
```

安全最高。

但是：

* 配置复杂
* 性能开销高

---

## 六、运维排障思路

### HTTP问题

访问失败：

```
http://www.znyth.com
```

检查：

```
1 DNS
2 网络连通
3 SLB监听80
4 Gateway状态
5 服务路由
```

---

### HTTPS问题

访问失败：

```
https://www.znyth.com
```

多检查：

```
1 证书是否过期
2 域名是否匹配
3 TLS版本
4 SLB证书配置
5 HTTPS监听443
```

---

## 七、面试总结版

### HTTP链路

一句话：

> 用户通过DNS解析域名获得SLB VIP，通过HTTP 80端口访问SLB，SLB根据健康检查和负载算法选择Gateway节点，Gateway根据路由规则和服务发现将请求转发到具体微服务。

链路：

```
DNS → SLB:80 → Gateway → Nacos → Micro Service
```

---

### HTTPS链路

一句话：

> 用户通过DNS访问SLB 443端口，首先完成TCP和TLS握手，SLB使用证书完成SSL卸载并解密请求，然后以HTTP方式转发给Gateway，后续流程和HTTP一致。

链路：

```
DNS → SLB:443 → TLS解密 → Gateway → Nacos → Micro Service
```

---

对于你做 **K8s + Spring Cloud + 运维平台实施运维** 的方向，这部分建议重点背熟：

```
DNS → SLB → Gateway → 认证中心 → Nacos → LoadBalancer → 微服务 → DB/Redis/MQ
```

以及：

```
HTTPS = HTTP + TLS
TLS终止位置决定整个网络模型
```

这是云上微服务架构排障和面试非常高频的一条主线。



# 系统访问不了系统性排查

ipconfig /all

- 本机IP是否正确
- 是否获取到网关地址

ping 127.0.0.1 不通则说明本机网卡驱动或TCP/IP协议栈故障

ping 网关IP

不通 → 内网故障（网线、交换机、VLAN、ACL等问题）。



nslookup 内网系统域名

- 能解析出正确的内网IP → DNS正常
- 解析失败或返回错误IP → DNS配置错误或DNS服务器故障

