---
title: AWVS的安装破解
---

# AWVS的安装破解

Acunetix Web Vulnerability Scanner（AWVS）经典商业漏扫工具

## 下载地址

本文破解工具均来自于Pwn3rzs大神的频道，致敬前辈的无私共享精神，后续工具更新可直接参考频道更新即可

频道地址：https://t.me/Pwn3rzs

Windows: 

https://ponies.cloud/scanner_web/acunetix/Acunetix-v24.4.240514098-Windows-Pwn3rzs-CyberArsenal.rar

Linux: 

https://ponies.cloud/scanner_web/acunetix/Acunetix-v24.4.240514098-Linux-Pwn3rzs-CyberArsenal.7z

Password: Pwn3rzs

## 安装方法

### Windows

下一步下一步安装即可。

破解过程：

1、添加以下内容到`C:\Windows\System32\drivers\etc\hosts`文件

```
127.0.0.1  erp.acunetix.com
127.0.0.1  erp.acunetix.com.
::1  erp.acunetix.com
::1  erp.acunetix.com.

192.178.49.174  telemetry.invicti.com
192.178.49.174  telemetry.invicti.com.
2607:f8b0:402a:80a::200e  telemetry.invicti.com
2607:f8b0:402a:80a::200e  telemetry.invicti.com.
```

3、将文件夹内的`wvsc.exe` 覆盖 `C:\Program Files (x86)\Acunetix\24.4.240514098\` 目录下同名文件

4、删除 `C:\ProgramData\Acunetix\shared\license\`文件夹内的所有文件

5、将文件夹内`license_info.json`和`wa_data.dat` 移动到`C:\ProgramData\Acunetix\shared\license` 目录下同名文件，选中两个文件，并设置权限为“只读”

重启电脑 再启动AWVS即可完成破解

### Linux

1、向 `/etc/hosts`添加以下内容

```bash
127.0.0.1 erp.acunetix.com
127.0.0.1 erp.acunetix.com.
::1 erp.acunetix.com
::1 erp.acunetix.com.

192.178.49.174 telemetry.invicti.com
192.178.49.174 telemetry.invicti.com.
2607:f8b0:402a:80a::200e telemetry.invicti.com
2607:f8b0:402a:80a::200e telemetry.invicti.com.
```

2、运行sh脚本安装

```bash
sudo bash acunetix_xxxxx.sh
```

3、安装后，使用以下命令停止其服务：

```bash
sudo systemctl stop acunetix
```

4、替换 wvsc 文件

```bash
sudo cp wvsc /home/acunetix/.acunetix/v_240514098/scanner/wvsc

sudo chown acunetix:acunetix /home/acunetix/.acunetix/v_240514098/scanner/wvsc

sudo chmod +x /home/acunetix/.acunetix/v_240514098/scanner/wvsc
```

5、添加许可证的时间：

注意复制和粘贴正确，这可能会损坏你的整个操作系统！！！

```bash
sudo cp license_info.json /home/acunetix/.acunetix/data/license/
sudo cp wa_data.dat /home/acunetix/.acunetix/data/license/
sudo chown acunetix:acunetix /home/acunetix/.acunetix/data/license/license_info.json
sudo chown acunetix:acunetix /home/acunetix/.acunetix/data/license/wa_data.dat
sudo chmod 444 /home/acunetix/.acunetix/data/license/license_info.json
sudo chmod 444 /home/acunetix/.acunetix/data/license/wa_data.dat
sudo chattr +i /home/acunetix/.acunetix/data/license/license_info.json
sudo chattr +i /home/acunetix/.acunetix/data/license/wa_data.dat
```

6、重新启动 acunetix

```bash
sudo systemctl start acunetix
```

7、现在重新登录应用程序，您应该可以使用它了

### MacOS

linux系统 mac系统 都推荐`docker`安装

x86的镜像也能在m芯片上运行

拉取镜像

```bash
docker pull quay.io/hiepnv/acunetix
```

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/image-20240620000341793.png)

设置&破解

```lua
docker volume create acunetix_data
```

```bash
docker run -d -p 3443:3443 --restart=unless-stopped --name=acunetix_web quay.io/hiepnv/acunetix
```

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/image-20240620000426927.png)

## 登录信息

访问 

https://127.0.0.1:3443/

用户名：`admin@acu.com`

密码：`Passw0rd!`

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/image-20240620002507533.png)

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/image-20240620003725576.png)
