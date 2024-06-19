---
title: linux&windows应急响应
---

## 应急响应流程

收到客户的主机中毒事件的问题通报，然后向客户获取中毒主机的ssh远程连接权限，或者到客户现场进行现场排查。

## 注意事项

```
1. 了解安全事件情况
• 现在遇到了什么问题？
• 什么时间发现的？如何发现的？谁发现的？
• 服务器是否有异常？具体特征是什么？安全设备是否有告警？
• 什么系统？用了什么中间件？什么开发语言？业务架构是什么？网络拓扑大概是什么样子？设备有没有重启或者关机过？是否有日志服务器？
• 需要我解决什么？
2. 事件初步判定
• 根据客户所描述的信息，对事件形成初步判断，建立排查方向与应急思路
• 判断自己能否处理，不能处理是否有其他同事可以处理
3. 不要执行更改文件属性命令和操作；
• 特定环境下文件的属性是可以改变的，甚至攻击者可以不改变文件属性前
提修改文件内容
• 保持样本最原始的属性和权限以及用户，方便溯源
• 利用cp之后很可能在恢复阶段时候业务会宕掉(注意权限问题)
4. linux不要执行rm -rf 的命令；
• 可以加-f，但是-rf 组合一定慎用
• 最好还是mv，移除样本
5. 取样本的时候建议使用mv命令移到文件夹；
• mkdir $HOME/sample建一个sample文件夹
• mv 样本 $HOME/sample下
6. linux的bash默认只记录1000条，超过就会被覆盖；
• 建议先备份$HOME/.bash_history 
• 或者对history进行加固
7. linux终端交互式shell不只有bash一种； 
• zsh
• csh
```

## 排查思路，以及常用的命令

### Linux

**「查看进程」**

```
ps aux
ps -ef
pstree -aup   
top
1 每个逻辑CPU状态
b 高亮
x 高亮排序列
ps aux | grep pid
```

```
lsof -i:port
-i 筛选关键字，比如进程名、PID、USER、IPv4/IPv6、TCP/UDP

pstree -aphn
-a 显示该进程命令行参数
-p 显示pid
-h 高亮当前进程以及父进程
-n 按照pid排序
```

**「网络连接」**

```bash
netstat -antlp
-a 显示所有连接和端口
-n 以数字形式显示ip和port
-t 显示tcp
-l 显示监听的服务
-p 显示建立连接的进程名以及pid
ESTABLISHED 建立连接、LISTENING 侦听状态、TIME_WAIT 连接超时

ss –antpr
-a 显示所有连接和端口
-n 不解析服务的名称
-t 显示tcp sockets
-l 显示监听的端口
-p 显示监听端口的进程
-r 把ip解析为域名

tcpdump –i eht0 -w eee.pcap
-i 要抓取的网卡接口
-w 要保存的文件名
-C 100m 大于100M分割为另一个包
-c 10000 超过1万个停止抓包
src host xxx 仅仅捕获由该主机发起的流量
drc host xxx 仅仅捕获发往该ip地址的流量
src port xxx 仅仅捕获由该端口发起的流量

#查看 PID 所对应的进程文件路径
file /proc/$PID/exe
```

**「异常文件」**

```
ls -alth
-a 显示所有文件
-l 显示文件权限，属组属主，大小，日期
- t 按照时间顺序排序
-h 高亮

strace -f -e trace=file 接可执行文件
-e 跟踪子进程
-f 跟踪类型，file、process、trace, abbrev, verbose, raw, signal, read, write, fault,inject, status, kvm

find 
find path –name xxx search_filename/directory
find path -name "*.php" -exec tar -rvf sss.tar.gz {} \; • find / \( -path /etc -o -path /usr \) -prune -o -name '*.conf'
-mtime 修改时间
-ctime 属性权限更改
-atime 访问时间

grep file_content search_directory
-R 搜索字符串
-P 使用正则匹配
-n 包含字符串文件的行号
grep -RPn "(xx.xx.xx) *\(" /var/log
grep -RPn 
“assthru|shell_exec|system|phpinfo|base64_decode|chmod|mkdir|fopen|fclose|readfile|php_uname|eval|tcpflood|udpflood|edoced_46esab) 
*\(" /var/www

strings 
strings /usr/bin/.sshd | grep '[1-9]{1,3}.[1-9]{1,3}.' //分析sshd文件，是否包括IP信息

file

lsattr/chattr 
-a 只能增加数据，但不能删除
-i 不能删除，改名，设定链接，不能写入或新增
lsattr malaware_file
chattr –i malaware_file
chattr +i malaware_file

hexdump
-b 以8进制显示输出结果
-c 以ASCII码显示输出结果
-C 以〸进制+ASCII码显示输出结果
```

**「后门账户」**

```bash
#主要查看uid为0的账户
cat /etc/passwd

#查询特权用户特权用户(uid 为0)
awk -F: '$3==0{print $1}' /etc/passwd

#查询可以远程登录的帐号信息
awk '/\$1|\$6/{print $1}' /etc/shadow

#除root帐号外，其他帐号是否存在sudo权限。如非管理需要，普通帐号应删除sudo权限
more /etc/sudoers | grep -v "^#\|^$" | grep "ALL=(ALL)"

#禁用帐号，帐号无法登录，/etc/shadow第二栏为!开头
usermod -L user    
#将删除user用户，并且将/home目录下的user目录一并删除
userdel -r user
```

**「ssh公钥」**

```
/root/.ssh/authorized_keys
```

**「登录日志」**

```
 last lastb     登陆失败 
 lastlog        最后一次登陆
```

**「查看启动项」**

```
more /etc/rc.local
```

**「查看服务」**

```
#查看服务自启动状态
chkconfig  --list
```

**「任务计划」**

```
crontab -l  查看计划任务
crontab -e  编辑计划任务
crontab -r  删除计划任务
crontab -u  查看某用户计划任务

ls -al /var/spool/cron/* 
cat /etc/crontab
/etc/cron.d/*
/etc/cron.daily/* 
/etc/cron.hourly/* 
/etc/cron.monthly/*
/etc/cron.weekly/
/etc/anacrontab 
/var/spool/cron 
/var/spool/anacron

#查看目录下所有文件
more /etc/cron.d/*

/etc/anacrontab
/var/spool/anacron/*
```

**「系统信息排查」**

```bash
#du
-s 显示总计
-h 以合适单位输出统计结果
df
-u 显示当前文件系统空间使用情况

#lspci
-v 以初阶显示设备详细信息
-vv 以进阶显示设备详细信息
-s xx:xx –vv 仅显示总线和
插槽的设备信息，xx:xx数值可以通过lspci获得

#lsusb
-v 显示USB的详细信息
-s xx:xx –vv 仅显示总线和设备号的设备信息，xx:xx数 值可以通过lspci获得
-d xx:xx仅显示指定厂商和产品编号的设备

who 显示当前所有登陆的信息
whereis 查找二进制文件路径
uname -a 打印系统信息

#last
description: 列出目前与过去登陆系统的信息
log_path: /var/log/wtmp
会读取wtmp的文件，并把该给文件的内容记录的登入系统的用户名单全部显示出来。

#lastb
description: 列出登陆失败系统的信息
log_path: /var/log/btmp
它会读取位于/var/log目录下，名称为btmp的文件，并把该文件内容记录的登入失败的用户名单，全部显示出来
```

```bash
#systemctl 系统服务管理指令

systemctl status sshd 查看ssh服务状态
systemctl start sshd 启动ssh服务
systemctl stop sshd  关闭ssh服务
systemctl enable sshd 设置ssh开机自启动
systemctl disbale sshd 关闭ssh开机自启动
service 控制系统服务指令
command_usage: service 'service_name' status/start/stop/restart 
```

```bash
#rsyslog日志配置
configuration_file: /etc/rsyslog.conf
configuration_directory: /etc/rsyslog.d/

#lastlog
description: 显示所有用户最后一次登陆信息
log_path: /var/log/lastlog
它根据UID排序显示登录名、端口号（tty）和上次登录时间。
如果一个用户从未登录过，lastlog显示**Never logged**
```

**「一些防火墙基础」**

```bash
#iptables
-t 接表名(raw'网址过滤',nat'地址转换',filter'包过滤',mangle'数据包修改')
- L 列出表里的规则
-n 显示IP
- v 显示进阶的信息展示
-A 将规则添加到最后，只有放到最后才能执行
-i 指定网卡接口
-j 执行操作(包括有且不限于ACCEPT、DROP、RRDIRECT)

systemctl start/stop/restart/status iptables 防火墙开启、关闭、重启、状态
iptables –L INPUT/OUTPUT -n 查看入站/出站规则
iptables -D INPUT 接规则号
iptables –A INPUT –s xx.xx.xx.xx/24 –p tcp –dport 22 –j ACCEPT 只允许某个网段连接22端口
iptables –A INPUT –s xx.xx.xx.xx/24 –p tcp –dport 22 –j ACCEPT ban掉某个网段对连接22端口
```

```bash
#firewalld
--state 防火墙状态
--reload 不中断服务重启加载
--compete-reload 中断所有连接重新加载
--runtime-to-premanent 永久保存添加的规则
--check-config 检查配置
--get-log-denied 查看拒绝日志
--set-loag-denied 接参数 设置拒绝日志监测等级(all/unicats/broardcast/multicast/off)

systemctl start/stop/restart/status firewalld 防火墙开启、关闭、重启、状态
firewalld-cmd –-state 查看防火墙状态
firewalld-cmd –-list-services 查看开放的服务
firewalld-cmdd –-add-ports=22/tcp 开启22端口
firewalld-cmdd –-remove-ports=22/tcp 关闭22端口
firewalld-cmd –-reload 更新防火墙规则
```

**「关键系统命令是否被替换」**

```
这个随便输几个试一试
```

**「几个关键的日志文件」**

```
SSH                        /var/log/secure 
记录系统                    /var/log/message 
计划任务日志                 /var/log/cron 
记录所有用户的登录            /var/log/wtmp
```

**「树形系统结构与应急」**

```
/bin
binary的简写
存放着linux基础的可执行文件，例如ls、cd .......
该目录区别于/usr/bin，它可在没有安装其他文件系统单独运行比如(单用户)

/usr/bin
本地登陆用户可访问应用程序/可执行二进制文件
一般恶意软件更容易感染该目录下文件，例如ssh、top、crontab.......

/sbin
binary的简写
存放着linux基础的可执行文件，例如ls、cd .......
该目录区别于/usr/bin，它可在没有安装其他文件系统单独运行比如(单用户)

/usr/sbin
在系统启动期间本地登陆的管理用户有关系统管理的可执行二进制文件
该目录一般不容易出问题，除非是git.kernel.org被搞了，里边命令例如
fastboot、fdisk、grub.......

/usr
unix software resource的简写
登陆用户对linux操作的实际目录，包含了linux的应用程序/可执行程序、64/32 位的依赖库、登陆用户的配置文件、登陆用户的底层管理程序、头文件

/var
可以理解为linux的缓存目录，大多数情况会以只读挂载到/usr下用于系统的缓
存数据和应用程序的缓存，例如应用程序缓存、标准库、某个程序运行导致的
文件设备的锁定(apt、yum)、日志文件、pid信息

/lib、/lib32、/lib64、libex32、/libexec
/lib主要包含内核模块、引导系统以及linux运行所需要的基本共享库
/lib<qual>在不同(32/64)环境下会调用不同的基本共享库
/libx32 是面向x32 ABI(x32 Application Binary Interface)的目标文件和库
/libexec不由用户和shell执行的二进制文件，一些lfs用于产品会有libexec作为部 署程序
一些二进制可执行程序劫持，会将so文件释放到/lib下

/dev
device的简写
存放着一些特殊的文件和设备文件位置

/dev/null 是类Unix系统中的一个特殊文件设备，他的作用是接受一切输入它的
数据并丢弃这些数据。

/mnt
mount的简写，挂接光驱、USB设备的目录
mount /dev/nvme0n1pxx /mnt/xxxx // 挂载
umount /mnt/xxxx //卸载

/proc
linux伪文件系统，存储了系统正在运行的所有信息，例如高级电源管理、引导参数、
所有设备使情况......
通常我们会用ls –alth /proc/*/exe，查看某恶意进程对应的二进制文件或查看文件已
经被删除，但是进程还在跑的情况

/sys
linux伪文件系统，为linux提供内核、驱动、设备信息，包括但不限于各种总线，有点
类似于window硬件管理

/root、/home
root用户目录、普通用户目录
都存放着用户shell相关的配置文件，例如.bashrc、.bash_history、.ssh、.config、

profile
可能会存在恶意alias(bashrc)、攻击留下来的历史命令信息(.bash_history)、用来维
持权限的免密登陆(.ssh/authorized_keys)、一些软件的配置信息(.config)
/boot
主要存放linux内核文件以及引导加载程序，出现问题的比较少，利用难度比较高

/run
该目录是临时文件系统， 存储是启动之后的所有信息

/srv
区别于/var目录，主要存储用户主动产生的数据和对外的服务

/tmp
临时文件目录，用户存放用户产生的临时文件，可以随时销毁，或者系统可以自动销 毁，权限比较低
由于权限比较低，所以恶意软件或者攻击行为会在此路径下生成或落地木马以及工具。
```

**「Rootkit查杀」**

专用查杀工具 `chkrootkit` `rkhunter`

**「linux常用工具」**

nethogs 是一个小型的“网络顶部”工具。它不像大多数工具那样按协议或每个子网分解流量，而是按进程对带宽进行分组。

iftop iftop用于查看网络上的流量情况，包括实时速率、总流量、平均流量等，是一款实时流量监控工具。

### Windows

**「进程」**

```
#查看进程
tasklist
tasklist –m |findstr "string"
-m 显示所有exe/dll的服务，如果没有指定参数将显示所有加载模块
-svc 显示每个进程服务
-v 显示所有进程详细服务信息

#强制停止某进程
taskkill /T /F /PID

#获取进程的全路径
wmic process | finderstr “xxxx”
```

**「异常网络流量」**

```
#可用fistr过滤，类似Linux的grep命令
netstat -antop tcp
-a 显示所有连接和端口
-n 以数字形式显示ip和port
-t 显示连接状态
-o 显示连接关联进程ID
-p TCP/UDP
> eee.txt 如果连接很多可以接>，输出到文件, cat eee.txt |awk '{print $2}'

ESTABLISHED 建立连接 LISTENING 侦听状态 TIME_WAIT 连接超时 SYN_SENT 请求连接 SYN_RECVD 收到请求连接

#打印路由表
route print

#查看网络代理配置情况
REG QUERY "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings"
```

**「敏感文件路径」**

```
%WINDIR% 
%WINDIR%/System32
%TEMP%
%APPDATA%
%LOCALAPPDATA%
%LOCALAPPDATA%\Low
%ProgramData%\Temp
%ProgramData%\Temp\Low
%UserProfile%\Recent
```

**「账户」**

查询登陆系统会话

```
query user
```

把账户踢出会话

```
logoff ID
```

- 打开`lusrmgr.msc`，查看是否有新增/可疑的账号
- 用D盾 -> 查看服务器是否存在隐藏账号、克隆账号

**「查看启动项」**

```
运行->msconfig  引导、服务、工具

#查看系统开机时间
net statistics workstation

#查看系统计划任务
schtasks /query /fo LIST /v

#查看程序启动信息
wmic startup get command,caption

#查看主机服务信息
wmic service list brief
```

**「计划任务」**

一些恶意软件通常会写入计划任 务达到维持权限，横向的目的

命令类的: 运行->tasksch.msc

工具类的: Autoruns、这台计算机->(右键)计算机管理->计划任务程序

**「系统日志」**

```
win +r eventvwr.msc
```

或者计算机管理——事件查看器事件日志分析

**「对于Windows事件日志分析，不同的EVENT ID代表了不同的意义」**

| 事件ID | 说明                             |
| :----- | :------------------------------- |
| 4624   | 登录成功                         |
| 4625   | 登录失败                         |
| 4634   | 注销成功                         |
| 4647   | 用户启动的注销                   |
| 4672   | 使用超级用户（如管理员）进行登录 |
| 4720   | 创建用户                         |

**「每个成功登录的事件都会标记一个登录类型，不同登录类型代表不同的方式」**

| 登录类型 | 描述                           | 说明                                             |
| :------- | :----------------------------- | :----------------------------------------------- |
| 2        | c（Interactive）               | 用户在本地进行登录。                             |
| 3        | 网络（Network）                | 最常见的情况就是连接到共享文件夹或共享打印机时。 |
| 4        | 批处理（Batch）                | 通常表明某计划任务启动。                         |
| 5        | 服务（Service）                | 每种服务都被配置在某个特定的用户账号下运行。     |
| 7        | 解锁（Unlock）                 | 屏保解锁。                                       |
| 8        | 网络明文（NetworkCleartext）   | 登录的密码在网络上是通过明文传输的，如FTP。      |
| 9        | 新凭证（NewCredentials）       | 使用带/Netonly参数的RUNAS命令运行一个程序。      |
| 10       | 远程交互，(RemoteInteractive） | 通过终端服务、远程桌面或远程协助访问计算机。     |
| 11       | 缓存交互（CachedInteractive）  | 以一个域用户登录而又没有域控制器可               |

**「系统补丁是否打全」**

```bash
#cmd
systeminfo

#工具和在线地址
https://i.hacking8.com/tiquan
https://github.com/bitsadmin/wesng
```

**「其他排查项」**

杀毒软件功能是否异常？

是否有异常驱动文件？C:\Windows\System32\drivers

都记录了那些日志，Web日志

**「映像劫持」**

参考学习文档https://ssooking.github.io/2019/12/windows%E5%90%8E%E9%97%A8-%E6%98%A0%E5%83%8F%E5%8A%AB%E6%8C%81/

**「工具篇」**

**微软官方的几款工具** 

具体介绍百度，或者参考以前的安服工程师修炼手册

autoruns  https://docs.microsoft.com/en-us/sysinternals/downloads/autoruns

tcpview   https://docs.microsoft.com/en-us/sysinternals/downloads/tcpview

procexp   https://docs.microsoft.com/en-us/sysinternals/downloads/process-explorer

Network Monitor [https://www.microsoft.com/en-us/download/confirmation.aspx?id=4865](https://www.microsoft.com/en-us/download/confirmation.aspx?id=4865)

**火绒剑** 

**卡巴斯基** dr.web 无需安装、不与其他杀毒软件冲突、不会被病毒限制启动

**wirshark** 流量抓包神器 可以抓五层流量，应、表、会、传、网、数

```
ip.addr==  匹配地址(源/目的) 数据包
tcp.port== 匹配TCP端口
http.request.uri=="xxx" 匹配指定URL
http.host == "domain"   匹配域名的数据包
```

具体教程：https://gitlab.com/wireshark/wireshark/-/wikis/CaptureFilters

**Browser_Network_Analysis**

F12检查功能，用途网页篡改，可以用Burpsuite或者Wireshark替代，根据网页文件、js、css加载顺序，时间先后顺序判断网页跳转所在。

### webshell排查

> ❝
>
> 常见的检测方法有基于主机的流量-文件-日志检测、关键字(危险函数)匹配、语义分析等
>
> ❞

web日志审计`access.log`（`/var/log/nginx`）

使用工具查杀Web目录

Windows：D盾 - http://www.d99net.net/down/WebShellKill_V2.0.9.zip

Linux：河马 - https://www.shellpub.com/

但工具查杀不靠谱，还是要手动查看Web目录下的可解析执行文件；

通过Web访问日志分析可快速定位到webshell位置。

**「网站被植入WebShell的应急响应流程」**

主要关注Web日志，看有哪些异常的HTTP访问，如果有源码，可以用文件对比的方法快速定位Webshell。（一定要是客户备份的源码，千万不要直接到主源码上操作。）

- 定位时间和范围：扫描WebShell位置；定位文件创建的时间；检查Web根目录.htaccess文件
- Web日志审计：例如查看access.log（/var/log/nginx），下载到本地审计
- 漏洞分析：分析可能存在漏洞的地方，复现漏洞GetShell。
- 漏洞修复：清除WebShell并修复漏洞对系统和Web应用进行安全加固