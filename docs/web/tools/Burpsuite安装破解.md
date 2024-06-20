---
title: Burpsuite安装破解
---

# burpsuite 2020.12.1专业版安装破解

## 前言

离写这篇文章已经过去了三年多了 相关破解资源已经出现了较多bug 所以这里更新下资源链接 最新的破解教程已经导出为pdf放到了里面啦 下载解压就好了

下载链接
https://www.123pan.com/s/LPX9-ktD5v.html
提取码:osoX

Burp Suite 是一款集成化的渗透测试工具，包含了很多功能，Burp Suite是由java语言编写，所以他的安装以及使用需要Java环境，下面我就来简单的介绍一下它的安装破解过程。

## 破解过程

1、首先解压我上面给的资源，先编写一个cmd程序命名为Burp Suite.cmd

```bash
java.exe -noverify -javaagent:BurpSuiteLoader.jar -jar burpsuite_pro_v2020.12.1.jar
```

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/20210204155029160.png)

2、双击上面编写的程序Burp Suite.cmd运行，打开后提示需要输入key

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/20210204155140956.png)

2、这个时候，我们回到目录双击运行BurpSuite KeyGen By Uncia.jar，将lisense里的码复制（这里不能右键复制，用ctrl+c/v)，然后粘贴到burpsuite的key里，然后点Next

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/2021020415541217.png)

3、然后点击Manual activation

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/2021020415553136.png)

4、点击copy request复制密钥。

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/20210204155734746.png)

5、然后粘贴到burpsuite keygen里的第二个窗口。它会自动在第三个窗口生成密钥

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/20210204155707219.png)

6、然后再将第三个窗口生成的密钥复制到burp suite里的第三个窗口里，然后点击Next下一步

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/20210204155937281.png)

7、最后单击Finish，结束破解过程。

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/20210204160144773.png)

8、但是每次运行时都有命令窗口出现，不好看，所以我们写一个程序做个后期处理

```bash
set shell = wscript.createobject("wscript.shell")
a = shell.run("""burp suite.cmd""",0)
```

![](https://img-blog.csdnimg.cn/20210204160508490.png)

9、以后直接双击运行这个程序就好了，也可以直接在桌面生成一个快捷方式再换个图标啥的就美观便捷啦
