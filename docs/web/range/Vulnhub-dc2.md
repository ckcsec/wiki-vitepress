---
title: Vulnhub-dc2
---

# Vulnhub-DC2

## 靶机信息

### 下载链接

[https://download.vulnhub.com/dc/DC-2.zip](https://download.vulnhub.com/dc/DC-2.zip)

### 靶机介绍

与 DC-1 非常相似，DC-2 是另一个专门建造的易受攻击的实验室，目的是在渗透测试领域获得经验。与最初的 DC-1 一样，它的设计考虑到了初学者。必须具备 Linux 技能和熟悉 Linux 命令行，以及一些基本渗透测试工具的经验。就像 DC-1 一样，有五个标志，包括最终标志。再一次，就像 DC-1 一样，标志对初学者很重要，但对有经验的人来说并不那么重要。

### 目标

Boot to root

### 运行环境

靶机：NAT模式，靶机自动获取IP

攻击机：Windows10、kali linux2021.1

## 信息收集

### 目标发现

```bash
arp-scan-l
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(11).png)

成功发现目标机IP为192.168.160.193

### 端口和服务识别

使用nmap扫描1-65535全端口，并做服务指纹识别，扫描结果保存到txt文件，命令：

```shell
nmap -p1-65535 -A 172.20.10.7 -oN dc2.txt
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(12).png)

发现目标靶机端口和服务

```shell
#端口 协议 后端服务
TCP 80 HTTP APache httpd 2.4.10(debian)
tcp 7744 SSH Openssh 6.7p1
```

### 主页信息收集

尝试访问80端口

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(7).png)

发现输入的IP地址变为域名，初步猜测为域名重定向，进入/etc/hosts，进行配置，添加如下内容

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(1).png)

再次访问，成功访问目标主页

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(77).png)

并发现有个标题为Flag，点击查看，成功发现第一个Flag，并提示我们用CeWL（CeWL是一款以爬虫模式在指定URL上收集单词的工具，可以将它收集到的单词纳入密码字典，以提高密码破解工具的成功率。）

<img src="https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(18).png" alt="" style="zoom:50%;" />

### 目录扫描

```shell
dirb http://dc-2
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(4).png)

尝试访问得到的多个目录，发现wordpress后台登录页面`http://dc-2/wp-admin`

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(3).png)

## 渗透阶段

### Cewl扫描生成字典

根据前面的提示（要使用cewl工具），这里直接开始扫描并生成字典：pass.dic

```shell
cewl http://dc-2/ -w pass.dic
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(17).png)

### wpscan枚举爆破

上面得到了密码字典，因为目标靶机使用的是wordpress框架，所以这里直接抬一手wpscan枚举用户名，来得到用户名字典，而后爆破上面得到的后台页面

```bash
wpscan --url http://dc-2/ -e u
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(5).png)

成功获取

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(14).png)

将上面枚举的3个用户名，写入user.dic,用户名字典，并用wpscan爆破

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(16).png)

```bash
wpscan --url http://dc-2/ -U user.dic -P pass.dic
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(19).png)

成功爆破，得到两个用户名和密码

```bash
jerry / adipiscing
tom / parturient
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(20).png)

使用jerry用户和tom用户登入网站后台,用户jerry登录成功，Tom不行，登录成功后发现第二个Flag

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(10).png)

这里提示我们用exp去打wordpress是不可能的，希望我们找另外一条路。

### ssh登录

尝试用之前得到的用户名和密码登录ssh（7744），最终Tom登录成功

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(8).png)

### rbash限制绕过

查看当前目录下的文件，发现第三个flag，但是发现shell命令受限，不能查看

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(2).png)

这里是rbash限制，需要绕过

> 有很多不同的限制外壳可供选择。其中一些只是普通的shell，有一些简单的常见限制，实际上并不是可配置的，例如rbash（限制Bash）rzsh和rksh（受限模式下的Korn Shell），这些都非常容易绕过。其他人有一个完整的配置集，可以重新设计以满足管理员的需求，如lshell（Limited Shell）和rssh（Restricted Secure Shell）。
> 一旦配置可以被管理员收紧，可配置的shell就更难以绕过。在这些shell上绕过技术通常依赖于管理员有点被迫为普通用户提供某些不安全命令的事实。。如果在没有适当安全配置的情况下允许，它们会为攻击者提供升级权限的工具，有时还会向root用户升级。
> 其他原因是，有时管理员只是Linux系统管理员，而不是真正的安全专业人员，因此从渗透测试人员的角度来看，他们并不真正了解部队的方式，并最终允许太多危险命令。

然后小弟就一顿谷歌，找到了一篇比较全面的绕过方法，参考链接：[https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/](https://fireshellsecurity.team/restricted-linux-shell-escaping-techniques/)

绕过：

```bash
#把/bin/bash给a变量,绕过首先的shell
BASH_CMDS[a]=/bin/sh;a
#使用并添加环境变量，将/bin 作为PATH环境变量导出
export PATH=$PATH:/bin/ 
#将/usr/bin作为PATH环境变量导出
export PATH=$PATH:/usr/bin
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(15).png)

成功绕过，并查看flag3.txt,获得第3个flag![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(6).png)

这里提示我们要su 切换用户。

切换用户到jerry，并进入到jerry用户的目录，从查看文件，发现第4个flag

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(13).png)

提示还不是最终的flag，提示git，查看sudo配置文件`sudo l`，发现git是root，不用密码可以运行，搜索git提权

## git提权

使用 sudo git -p help 且一页不能显示完，在最底下面输入 `!/bin/bash`,最后完成提权

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(22).png)

cd 到root目录下，查看文件，发现最后一个flag，并查看

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Vulnhub-dc2%20(23).png)

至此，DC-2靶机渗透完毕。

