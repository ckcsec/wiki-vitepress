---
title: Vulnhub—LazySysAdmin1
---

# Vulnhub—LazySysAdmin1

## 靶机信息

### 下载链接

https://download.vulnhub.com/lazysysadmin/Lazysysadmin.zip

### 靶机说明

The story of a lonely and lazy sysadmin who cries himself to sleep

 一个孤独而懒惰的系统管理员哭着睡觉的故事

### 目标

获得root权限和flag。

### 运行环境

靶机：NAT连接，靶机自动获取IP。

攻击机：Widows11，kali linux2021.1

### 通关提示

- Enumeration is key
- Try Harder
- Look in front of you
- Tweet @togiemcdogie if you need more hints

## 信息收集

### 目标发现以及端口服务识别

```shell
#确定目标机IP
arp-scan -l
#确定目标机开放端口以及服务版本
nmap -p1-65535 -sV -A -oN lazy1.txt 192.168.160.191
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Lazysysadmin%20(2).png)

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Lazysysadmin%20(1).png)

根据扫描结果可知目标机开放了22、80、139、445、3306、6667端口，以及ip为192.168.160.201

访问网络web服务页面，静态页面，没有任何可利用的信息

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Lazysysadmin%20(3).png)

使用dirb来爆破目标存在的目录

```shell
dirb http://192.168.100.200
```

发现目标文章用的是wordpress，且还有phpmyadmin，而且还有个phpinfo

访问`http://192.168.100.200/wordpress` 发现用户名提示togie

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Lazysysadmin%20(4).png)

/phpmyadmin

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Lazysysadmin%20(5).png)

/phpinfo

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Lazysysadmin%20(6).png)

## 漏洞挖掘

wpscan扫描、用户名枚举爆破并没有什么有用的信息

获取目标站点信息

```shell
enum4linux 192.168.160.201
```

到这里就非常无奈了，好像没有漏洞，只能尝试暴力破解了，但我又想到了该目标开启了samba服务，尝试匿名访问，嘿嘿，成功了

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Lazysysadmin%20(7).png)

查看文件夹

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Lazysysadmin%20(8).png)

直接连接，查看各个文件，或者拖出来代码审计

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Lazysysadmin%20(9).png)

发现账户密码，猜测是上面在wordpress界面发现的用户密码togie:12345

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Lazysysadmin%20(10).png)

尝试ssh连接，成功了！好耶！

尝试sudo -i提权

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Lazysysadmin%20(11).png)

成功

查看flag

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Lazysysadmin%20(12).png)

就这？？？

至此靶机渗透结束