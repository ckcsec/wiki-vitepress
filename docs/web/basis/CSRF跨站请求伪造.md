---
title: CSRF跨站请求伪造
---

# CSRF跨站请求伪造

## 对象

web客户端

## 利用点

CSRF最关键的是利用受害者的Cookie向服务器发送伪造请求。

主要危害：

以你名义发送邮件、发消息、盗取你的账号、甚至于购买商品，虚拟货币转账......

原理：

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/image-20240619113354663.png)

## CSRF攻击流程

1. 登录受信任网站A，并在本地生成Cookie。
2. 在不登出A的情况下，访问危险网站B。

## CSRF利用方式

### GET型

仅需要一个HTTP请求。就能够构造一次简单的CSRF。

1. 链接利用(a标签)
2. iframe利用

可以设置iframe的style为display:none，以此来不显示iframe加载的内容

1. img标签利用

img标签内的内容会随着页面加载而被请求，以此src指向的位置会在页面加载过程中进行请求

1. background利用

可以利用CSS中background样式中的url来加载远程机器上的内容，从而对url中的内容发送HTTP请求

### POST型

危害没有GET型的大，利用通常使用的是一个自动提交的表单。如：

```
<form name="csrf" action="http://edu.xss.tv/payload/xss/csrf2.php" method="post">
    <input type="hidden" name="name" value="zhangsan">
    <input type="hidden" name="money" value="1000">
</form>
<script type="text/javascript">document.csrf.submit();</script>
```

访问该页面后，表单会自动提交，相当于模拟用户完成了一次POST操作。

**CSRF漏洞探测**

利用自动化探测工具CSRFTester或者burp自带CSRF POC

1.CSRFTester设置浏览器代理:127.0.0.1:8008，burp是8080

2.登录web应用程序，提交表单，在CSRF工具中修改表单内容，查看是否更改，如果更改就存在CSRF漏洞

3.生成CSRF的POC

参考：[Web安全Day3 - CSRF实战攻防](https://xz.aliyun.com/t/6128#toc-7)

**CSRF漏洞防御**

- 设置和判断cookie时采用hash值认证。
- 尽量采用post类型传参，这就减少了请求被直接伪造的可能。
- 验证HTTP Referer字段
- 在 HTTP 头中自定义属性并验证
- 在请求地址中添加token并验证
- 采用验证码判断，进行防御。
