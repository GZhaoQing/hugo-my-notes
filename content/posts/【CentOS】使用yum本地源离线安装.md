---
title: "使用yum本地源离线安装"
date: 2021-11-02T15:25:27+08:00
draft: true
tags: [yum本地源, rpm安装]
catagories: [CentOS]
---

[toc]

## 前言

商业场景中，有些软件应用需要部署在不连通互联网的局域网，意图用物理方式避免网络攻击和信息泄露。这时局域网的服务器上要安装软件，就要用离线方式。本文是CentOS离线环境安装软件的迷你教程，记录整个流程。

## 用到的资源

几个好用的网站：

```shell
# 查找&下载安装包
https://centos.pkgs.org/
http://rpmfind.net/
```

几个用的到的yum的命令：

```shell
yum install 安装的软件
yum update 更新软件
#通过yun源寻找软件安装包
yum whatprovides "*/软件名称"

#安装下载rpm的插件
yum install yum-plugin-downloadonly
#下载安装包（但不安装），默认路径/var/cache/yum/x86_64/7/base/packages/
yum [update|install] --downloadonly 软件名称

#yum变化时可能会有明明有rpm包，但是yum找不到的情况，这时clean all可以重新刷新yum的元数据
yum clean all && yum makecache
```



## 操作系统环境：CentOS 7.5.1804

## 操作步骤

增加和更改yum源主要对象是/etc/yum.repos.d文件夹

yum会从各.repo文件读取安装源

相应的，把.repo后缀改掉（如.repo.bk），yum就不读。凭此我们可以模拟离线环境。

以本版本的centos75为例：

```
/etc/yum.repos.d/
├── CentOS-Base.repo.bk
├── CentOS-CR.repo.bk
├── CentOS-Debuginfo.repo
├── CentOS-fasttrack.repo.bk
├── CentOS-Media.repo
├── CentOS-Sources.repo
├── CentOS-Vault.repo
└── myrepo.repo
```

这里把CentOS-Base.repo、CentOS-CR.repo、CentOS-fasttrack.repo屏蔽掉，就相当于yum断网了。

myrepo.repo是我们自己的建的本地源，192.168.64.129是本机IP地址：

```shell
[myLocalRepo]
name=myLocalRepo
baseurl=http://192.168.64.129/my-yum/
gpgcheck=0
enabled=1
#忽略签名检查
```

为了支撑myrepo.repo的运行，我们需要一些准备：

#### 开启网络服务

yum源只接受远程网络地址，所以我们需要在本地启动一个网络文件服务，httpd、nginx等皆可：

```shell
# 安装httpd
yum install httpd
# 启动httpd
systemctl start httpd.service
# 开机自启
systemctl enable httpd.service

# 建立软链接
ln -s /var/www/html/my-yum  本地源文件夹
```



#### 下载安装文件&鸡生蛋问题

如果httpd这些基础软件局域网没安装怎么办？

最好的办法是在互联网环境准备一个一样版本的虚拟机，然后在互联网环境把基础软件包和依赖下载下来，考进去安装。

操作层面基本的办法是去网上查安装包。

这里推荐一个yum工具yum-plugin-downloadonly

可以直接把安装包下到你的模拟环境，而且依赖齐全。

```shell
#安装下载rpm的插件
yum install yum-plugin-downloadonly
#下载安装包（但不安装），默认路径/var/cache/yum/x86_64/7/base/packages/
yum [update|install] --downloadonly 软件名称
```

如果幸运的找到了网上的repo，可以直接下载

```shell
yum install wget
wget -r -c -np -nd -P 文件夹 网址
```

说明：
-r 表示递归下载
-np 不下载旁站连接
-c 断点续传
-nd 递归下载时不创建一层一层的目录，把所有的文件下载到当前目录
-P 表示下载那个目录



#### 建立源数据

createrepo用于创建yum源，建立元数据

```shell
yum install createrepo

createrepo 本地源文件夹
```

TIP：删掉本地源的html文件

此时访问http://IP/my-yum，能看到展现的文件目录。



#### 完成

回到开头的myrepo.repo，这时baseurl就有效的路径了

```
[myLocalRepo]
name=myLocalRepo
baseurl=http://192.168.64.129/my-yum/
gpgcheck=0
enabled=1
```

