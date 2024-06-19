---
title: Nmap的使用
---

# Nmap的使用

## 基础篇

```bash
nmap 192.168.0.100 #单个
nmap 192.168.0.100 192.168.0.105 #单点多个
nmap 192.168.0.100-110 #连续
nmap 192.168.0.100/24 #目标所在的某一个网段的所有地址
nmap 192.168.0.100/24 -exclude 192.168.0.105 #扫描在网段除目标地址之外的其他地址
nmap -iL C:\Users\zhiji\desktop\targets.txt #扫描文件中所有目标地址
nmap 192.168.0.100/24 -excludefile C:\Users\zhiji\desktop\targets.txt #扫描除某一文件中的目标地址之外的目标地址
nmap 192.168.0.100 -p 21,22,23,80 #指定端口
nmap --traceroute 192.168.0.105 #路由跟踪
nmap -sP 192.168.0.100/24 #所在C段的在线状况，目标主机存活
nmap -O 192.168.0.105 #指纹识别-判断目标操作系统版本
nmap -A 192.168.0.105 #扫描目标操作系统信息和路由跟踪
nmap -sV 192.168.0.105 #开放端口对应服务版本
nmap -sF -T4 192.168.0.105 #探测防火墙状态 open-开放；filtered-被防火墙过滤；closed-关闭，未开启；其他均为不能识别
```



## 进阶篇

1、鉴权 弱口令检测

```bash
nmap --script=auth 192.168.0.105
```

2、暴力破解

```bash
nmap --script=brute ip
```

3、漏扫

```bash
nmap --script=vuln ip
```

4、应用服务扫描（VNC、MySQL、Telnet、Rsync）

```bash
nmap --script=realvnc-auth-bypass ip #VNC服务为例
```

5、局域网服务探测

```bash
nmap -m -p 445 --script=broadcast ip
```

6、whios解析

```bash
nmap -script external baidu.com
```