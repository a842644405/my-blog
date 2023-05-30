---
title: Jenkins插件下载流程
---

# 一、`Jenkins` 插件下载流程

------

### 1、`default.json` 文件

首先，*Jenkins* 会读取 **工作目录**（*默认为/var/jenkins_home*）下的 `/updates/default.json` 文件内容，里面包含了插件**更新和下载**的地址。下面是部分配置内容：

```json
{
  "connectionCheckUrl": "http://www.google.com/",
  "core": {
    "buildDate": "Mar 15, 2022",
    "name": "core",
    "sha1": "fIBTb/g4+vyNN2yi+beijYEn6nA=",
    "sha256": "bul6WzwoaeivM4fskRAGlinL399cN545SF1w/Yv/t/0=",
    "size": 91224208,
    "url": "https://updates.jenkins.io/download/war/2.339/jenkins.war",
    "version": "2.339"
  },
  ...
}
12345678910111213
```

可以看到，插件的**连接搜索地址**为 `http://www.google.com/`，插件的**下载地址**为 `https://updates.jenkins.io/download`，显然，如果使用国内网络进行访问的话大概率会出现下载缓慢或失败的情况。

### 2、`hudson.model.UpdateCenter.xml` 文件

如果 *default.json* 文件不存在的话，就会从 `{Jenkins工作目录}/hudson.model.UpdateCenter.xml` 文件中读取插件配置文件的地址，并通过地址下载文件为 `{Jenkins工作目录}/updates/default.json`。

查看 *hudson.model.UpdateCenter.xml* 的配置内容如下：

```xml
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>https://updates.jenkins.io/update-center.json</url>
  </site>
</sites>
1234567
```

我们可以通过将 *url* 地址放到浏览器中访问查看，可以发现其内容就是默认 *default.json* 的配置信息。

而我们在 *Jenkins* 访问页面中的站点设置页面（`Dashboard -> 插件管理 -> 升级站点`）设置的升级站点地址：

![1682908711342](Jenkins%E6%8F%92%E4%BB%B6%E4%B8%8B%E8%BD%BD%E6%B5%81%E7%A8%8B/1682908711342.png)





其修改的内容就是 *hudson.model.UpdateCenter.xml* 配置文件中的 *url* 地址：

```bash
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json</url>
  </site>
</sites>
1234567
```

### 3、下载安装插件

有了 `default.json` 配置文件后，*Jenkins* 就会根据插件配置文件中更新和下载地址进行插件的下载安装。



# 二、更换 `Jenkins` 插件下载地址

------

### 1、更换插件站点地址无效问题

**很遗憾的是**，由于国内的镜像地址，比如[清华](https://so.csdn.net/so/search?q=清华&spm=1001.2101.3001.7020)的镜像配置地址 *https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json*，其配置内容使用的下载更新地址也是 *Jenkins* 官方默认下载的地址：

```json
{
	"connectionCheckUrl": "http://www.google.com/",
	"core": {
		"buildDate": "Mar 15, 2022",
		"name": "core",
		"sha1": "fIBTb/g4+vyNN2yi+beijYEn6nA=",
		"sha256": "bul6WzwoaeivM4fskRAGlinL399cN545SF1w/Yv/t/0=",
		"size": 91224208,
		"url": "https://updates.jenkins.io/download/war/2.339/jenkins.war",
		"version": "2.339"
	},
	...
}
12345678910111213
```

所以只是修改插件站点地址为国内站点地址是没用的，只能手动修改 *default.json* 的配置内容才能生效。

### 2、编辑 `default.json`

要修改的地址有两部分：

- 插件查找搜索地址（*默认为 http://www.google.com/*），我们更换成百度的地址；
- 插件下载地址（*默认为 https://updates.jenkins.io/download*），我们更换成国内镜像地址；

我们以[清华镜像](https://so.csdn.net/so/search?q=清华镜像&spm=1001.2101.3001.7020)地址为例：

```bash
sed -i 's#https://updates.jenkins.io/download#https://mirrors.tuna.tsinghua.edu.cn/jenkins#g' default.json
sed -i 's#http://www.google.com#https://www.baidu.com#g' default.json
12
```

这里，我们使用 `sed` 命令对配置文件内容进行全局替换。

下面列举了一些国内常用的镜像地址：

| 来源     | 地址                                          |
| -------- | --------------------------------------------- |
| tencent  | https://mirrors.cloud.tencent.com/jenkins/    |
| huawei   | https://mirrors.huaweicloud.com/jenkins/      |
| tsinghua | https://mirrors.tuna.tsinghua.edu.cn/jenkins/ |
| ustc     | https://mirrors.ustc.edu.cn/jenkins/          |
| bit      | http://mirror.bit.edu.cn/jenkins/             |

**修改完后，需要重启 Jenkins 来生效。**