---
title: Vulstudy靶场
---

# Vulstudy靶场

Vulstudy是专门收集当下流行的漏洞学习平台，并将其制作成docker镜像，方便大家快速搭建环境，节省搭建时间，专注于漏洞学习上。包含以下漏洞平台

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bac-1.png)

下面就来详细介绍下环境搭建过程。

## 1、下载docker

```zsh
#安装docker
apt-get install docker.io
#安装pip3
apt-get install python3-pip
#安装docker-compose
pip3 install docker-compose
#下载vulstudy项目，相当于git仓库克隆
git clone https://github.com/c0ny1/vulstudy.git
```

## 2、配置docker

因为docker是国外的，所以在拉取镜像的时候特别慢，故需要配置下国内的镜像加速。这里直接访问[阿里云的容器镜像服务]( https://cr.console.aliyun.com/ )根据自己系统的配置文档配置即可

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bac-2.png)

配置完成后，就可实现国内镜像拉取加速。

## 3、使用

### 3.1 单个使用

```zsh
#进入你想运行的靶场目录
cd vulstudy/漏洞目录
#启动容器
docker-compose up -d
```

打开kali自带浏览器访问127.0.0.1，或者主机浏览器访问kali的ip地址即可访问你刚刚运行的靶场
![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bac-3.png)

如果你不想玩了，就可以关闭容器

```zsh
#停止容器 
docker-compose stop 
```

### 3.2 全部使用
（全部开启的话，可能有端口冲突，可进入每个靶场的配置文件里改下对应的端口即可）

```shell
cd vulstudy
#启动容器
docker-compose up -d 
#停止容器
docker-compose stop
```

ok!下面就可以愉快的在十几个靶场间来回玩耍了。

## 4、报错解决方案（持续更新）

在第一次启动bWAPP时会有报错

```
Connection failed: Unknown database ‘bWAPP’
```

可以访问127.0.0.1/install.php,或ip/install.php安装数据库![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bac-4.png)
安装完后再次访问127.0.0.1或者kali的ip即可，用户bee,密码bug。
