---
title: CobaltStrike的使用
---

# CobaltStrike的使用

## Cobaltstrike简介

Cobalt Strike是一款美国Red Team开发的渗透测试神器，常被业界人称为CS，其拥有多种协议主机上线方式，集成了提权，凭据导出，端口转发，socket代理，office攻击，文件捆绑，钓鱼等功能。分为服务端（teamserver）和客户端，服务端是一个，客户端可以有多个，团队可进行分布式协团操作。客户端可在windows上运行，服务端必须在Linux上，且两者都需Java环境。

## 环境搭建

[CobaltStrike4.0](https://zhiji.lanzoui.com/iYOvjp5hqra) 密码：cs66

客户端：win10（自己的主机）

服务端： ubuntu 20.04（国外vps服务器）

首先购买一台国外的vps服务器作为服务端，当然你也可以用虚拟机，但记住一定要是Linux。

### 安装java环境

客户端windows的java环境配置可以看我前面的文章，这里就不再赘述，下面介绍服务端的配置

```shell
#进入根目录
cd /
#查看java是否安装
java -version
#安装java
apt install openjdk-14-jre-headless
```

检查是否安装成功,返回面的内容，说明已经成功了

```shell
root@vultr:/# java -version
openjdk version "14.0.2" 2020-07-14
OpenJDK Runtime Environment (build 14.0.2+12-Ubuntu-120.04)
OpenJDK 64-Bit Server VM (build 14.0.2+12-Ubuntu-120.04, mixed mode, sharing)
```

环境变量

```shell
#查看java的安装路径，这里提示我们在链接组java（提供/usr/bin/java）中只有一种选择，所以无需配置环境变量
root@vultr:/# update-alternatives --config java
There is only one alternative in link group java (providing /usr/bin/java): /usr/lib/jvm/java-14-openjdk-amd64/bin/java
Nothing to configure.
```

### 上传服务端到vps

将CobaltStrike4.0上传到vps服务端（方法很多，这里我介绍一种方式）

Transfer.sh：Linux VPS使用命令行快速的分享文件（Transfer.sh是一个可以让我们将`Windows`/`Linux`系统里的文件快速分享出去的平台，特别是Linux VPS，只需要一条命令就可以将文件快速分享出去，而且上传文件最大可达`10GB`，有效期`14`天。）

官网：[https://transfer.sh/](https://transfer.sh/)

windows直接进入官网，点击`click to browse`就可以上传文件，之后会给我们一个文件分享链接。复制然后到vps，用wegt下载即可，比如：wget https://xx.com/xxxxx

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/cs1.png)

上传后解压即可

### 启动服务端

```shell
#给权限
chmod 777 ./teamserver
#启动服务端
./teamserver vps的ip 连接密码
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/cs2.png)

### 客户端连接

这里只需要将host修改为服务器端的ip地址，端口50050固定不变（端口确保开启），用户名可以随便起，密码必须写服务器端的密码。
![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/cs3.png)

如果出现连接被拒绝情况，请开启端口`ufw allow 50050`
成功连接时出现一串数字和服务器端给的一样，说明用秘钥匹配，使客户端和服务器端之间进行连接，不经过第三者,直接点是就好
![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/cs4.png)

连接成功，环境搭建完毕。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/cs5.png)

成功上线后,我们可以使用 /names 查看当前所有用户,或者 /msg 用户名 对指定用户发送消息
另外在实战中为了避免溯源，还需要做一些处理来隐藏服务端vps真实ip,下面就不再赘述，可以自行百度。下篇文章将具体讲解木马上线方式以及msf联动。
