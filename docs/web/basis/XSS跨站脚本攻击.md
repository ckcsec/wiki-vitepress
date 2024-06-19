---
title: XSS跨站脚本攻击
---

# XSS跨站脚本攻击

## 碎碎念

今天行程码终于没星星啦，是个值得庆祝的日子，水一篇文章助助兴（手动狗头）

## XSS跨站原理

> 指攻击者利用网站程序对用户输入过滤不足，输入可以显示在页面上对其他用户造成影响的HTML代码，从而盗取用户资料、利用用户身份进行某种动作或者对访问者进行病毒侵害的一种攻击方式。通过在用户端注入恶意的可执行脚本，若服务器对用户的输入不进行处理或处理不严，则浏览器就会直接执行用户注入的脚本。

## 主要存在的点

**数据交互的地方**

- get、post、headers
- 反馈与浏览
- 富文本编辑器
- 各类标签插入和自定义

**数据输出的地方**

- 用户资料
- 关键词、标签、说明
- 文件上传

## XSS跨站分类

### XSS-Reflected反射型（非持久型）

最常见的是Payload是构造在网址的某个**GET参数的值**里。 比如这样的：

```
http://www.xx.com/company/search.html?key_pro="><script>confirm(ckcsec)</script>
```

与存储型相反，反射型XSS的是通过提交内容，然后不经过数据库，直接反射回显在页面上，比如说以下代码就存在反射型的XSS，通过参数get的值提交Payload：

```
echo $_GET\['get'\];
```

攻击流程图如下

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/tyxxss-2.png)

#### 案例

发货100 cms

```
<script>alert(1)</script>
```

http://123.57.94.143/

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/%E5%9B%BE%E7%89%871.png)

### XSS-Stored存储型（持久型）

存储型XSS也叫持久型XSS，存储的意思就是Payload是有经过存储的，当一个页面存在存储型XSS的时候，XSS注入成功后，那么每次访问该页面都将触发XSS，典型的例子是：

如留言板

1. 插入留言=>内容存储到数据库
2. 查看留言=>内容从数据库提取出来
3. 内容在页面显示

如果这里存在XSS，Payload可以通过**留言内容**提交，然后显示在页面的时候可以生效，那么就是典型的存储型XSS。

攻击流程图如下

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/tyxxss-3.png)

#### 案例

DVWA靶场  XSS（Reflected） 

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/%E5%9B%BE%E7%89%872.png)

### XSS-DOM型

> DOM全称Document Object Model,使用DOM可以使程序和脚本能够动态访问和更新文档的内容、结构及样式。DOM型XSS其实是一种特珠类型的反射型XSS.，它是基于DOM文档对象模型的一种漏洞。基于DOM型的XSS漏洞不需要与服务器交互，他在发生在客户端处理数据阶段。

其实DOM型也属于反射型的一种，不过比较特殊，所以一般也当做一种单独类型。payload一般如下

```
http://xxx.com/xx/xxx.html#<img src=0 onerror='alert(0)'>
```

而其背后代码生效大概是这样的：

```
<script>
var name = location.hash;
document.write(name);
</script>
```

通过页面加入img标签的的方式发起了一个GET请求，应该是一个访问来源记录的东西，而对于Referer的值没有做处理，于是就存在DOM型XSS。

#### 案例

jQuery-with-XSS

https://vulnerabledoma.in/jquery_htmlPrefilter_xss.html

主要的类型就是上面三种，还有些其他类型这里也做一个简单的介绍

### UXSS（Universal Cross-Site Scripting）

UXSS是利用浏览器或者浏览器扩展漏洞来制造产生XSS并执行代码的一种攻击类型。

**一些实际案例可以参考理解**

MICROSOFT EDGE uXSS   CVE-2021-34506

Edge浏览器翻译功能导致JS语句被调用执行

[https://www.bilibili.com/video/BV1fX4y1c7rX](https://www.bilibili.com/video/BV1fX4y1c7rX)

### Flashxss-swf引用js的xss

Flash 产生的XSS主要来源于：

- getURL/navigateToURL  访问跳转
- ExternalInterface.call 调用js函数

### PDFXSS-上传后直链触发

- 创建PDF，加入动作JS
- 通过文件上传获取直链
- 直链地址访问后被触发

### UTF-7 XSS

在以下两种场景可以在低版本IE浏览器触发UTF-7 XSS：

- meta未指定编码，特定版本IE发现内容存在UTF-7编码内容，则自动以UTF-7解码处理
- 指定编码为UTF-7

UTF-7 XSS与普通XSS的区别就在于构造的Payload是UTF-7编码的，而基于上面两个场景的特性，低版本IE浏览器会自动解码，于是就可以产生XSS。

### MHTML XSS

> MHTML XSS 同样只存在于低版本的IE中，MHTML是MIME HTML (Multipurpose Internet Mail Extension HTML，聚合超文本标记语言)的缩写，把一个多附件（如图片，flash动画等）的网页内容都保存到单一档案的标准，是类似HTTP的协议，在IE中，当嵌入资源的URL的协议为MHTML 时，IE将调用MHTML Protocol Handler，把该资源当作MHTML格式文件解析处理。

x.html 内容：

```
Content-Type:multipart/related;boundary="x"
--x
Content-Location:xss
Content-Transfer-Encoding:base64
PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg==
--x--
```

其中

```
PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg== BASE64解码:<script>alert(1)</script>
```

通过特定的访问方式：

```
mhtml:www.x.com/a.html!xss
```

就可以触发XSS

### CSS XSS

```
<style>
    body {width:expression(alert(1));: red;}
</style>
```

CSS XSS 是缘于IE8 Beta2以前版本支持使用expression在CSS中定义表达式(公式)来达到建立元素间属性之间的联系等作用，于是就可以通过以上代码的方式触发XSS。

### VBScript XSS

VBScript XSS 同上面几种XSS一样，也是微软的产物，也可以触发XSS。

```
<input type ="button" onClick="VBScript:Document.Write 'ckcsec'

MsgBox 'xss'">
```

### mXSS(突变型XSS)

参考：[https://www.fooying.com/the-art-of-xss-1-introduction/#mxss](https://www.fooying.com/the-art-of-xss-1-introduction/#mxss)

## 常用payload

[https://github.com/payloadbox/xss-payload-list](https://github.com/payloadbox/xss-payload-list)

## 测试神器

* [XSStrike](https://github.com/UltimateHackers/XSStrike)

* [BruteXSS Terminal](https://github.com/shawarkhanethicalhacker/BruteXSS)

* [BruteXSS GUI](https://github.com/rajeshmajumdar/BruteXSS)

* [XSS Scanner Online](http://xss-scanner.com/)

* [XSSer](https://tools.kali.org/web-applications/xsser)

* [xsscrapy](https://github.com/DanMcInerney/xsscrapy)

## 总结

漏洞原理：接受输入数据，输出显示数据后解析执行

基础类型：反射(非持续)，存储(持续)，DOM-BASE

常用标签：https://www.freebuf.com/articles/web/340080.html

攻击利用：盲打，COOKIE盗取，凭据窃取，页面劫持，网络钓鱼，权限维持等

挖掘方法：代码审计、XRAY扫描、XSCAN、手工探测、burp_fuzz

安全修复：字符过滤，实例化编码，http_only，CSP防护，WAF拦截等

测试流程：看输出想输入在哪里，更改输入代码看执行（标签，过滤决定）

> 漏洞挖掘非一朝一夕  贵在坚持！

## 经典XSS_POC整理（持续更新）

**jQuery-with-XSS**

https://github.com/mahp/jQuery-with-XSS

**CVE-2021-41349**

```
POST /autodiscover/autodiscover.json

%3Cscript%3Ealert%28document.domain%29%3B+a=%22%3C%2Fscript%3E&x=1
```
