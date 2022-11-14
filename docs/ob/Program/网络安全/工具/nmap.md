---
date: 2022-07-04-星期一 08:17:49
update: 
tags: [time/year2022,time/month07]
id: 20220704081749
---

# nmap
## Nmap可以完成以下任务：
*   主机探测：探测网络上的主机，如列出响应TCP和ICMP请求、ICMP请求、开放特别端口的主机
*   端口扫描：探测目标主机所开放的端口
*   版本检测：检测目标主机的网络服务，判断其服务名称及版本号
*   系统检测：探测目标主机的操作系统及网络设备的硬件特征
*   支持探测脚本的编写：使用Nmap的脚本引擎（NSE）和Lua 的编程语言

##   Nmap在实际中应用场合如下：
-   通过对设备或者防火墙的探测来审计它的安全性
-   探测目标主机所开放的端口
-   通过识别新的服务器审计网络的安全性
-   探测网络上的主机
常用命令
系统漏洞检查 nmap 目标 --script=auth,vuln
扫描出可爆破的端口 nmap 目标 --script=ftp-brute,imap-brute,smtp-brute,pop3-brute,mongodb-brute,redis-brute,ms-sql-brute,rlogin-brute,rsync-brute,mysql-brute,pgsql-brute,oracle-sid-brute,oracle-brute,rtsp-url-brute,snmp-brute,svn-brute,telnet-brute,vnc-brute,xmpp-brute
收集应用服务 nmap -sC 









#### 4\. nmap命令

1----可以通过命令行查看参数列表-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719100533867.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lkX18zOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
2----nmap命令形式-
nmap \[Scan Type(s)\] \[Options\] {target specification}

3----设置扫描目标时用到的相关参数-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719102833615.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lkX18zOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
4----常见的端口扫描方法相关参数-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719103107480.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lkX18zOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719103126705.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lkX18zOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
5----跟端口参数与扫描顺序的设置相关的参数-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719103240713.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lkX18zOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
6----nmap常用命令

 命令手册：https://nmap.org/book/man-briefoptions.html

（1）扫描单个目标地址

    命令：nmap ip 或 ip地址的名称（xxxservice.texxx.com）
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719102523400.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719102613286.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lkX18zOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)（2）扫描多个目标地址，不在同一个网段或者并不多

    nmap ip1 ip2 ip3 ...
    
        

首先需要另个Ip,我们打开kali-linux虚拟机，查看自己的ip-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719104303137.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lkX18zOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
在kali中开启一个Apache服务（kali中自带Apache服务、mysql服务）-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719104459642.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lkX18zOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
开启后浏览器中可以看到-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719105347645.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lkX18zOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719105228479.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lkX18zOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
用sudo比较麻烦，顺便设置一个root账号与密码

设置root账号的密码-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719105743680.png)-
这样下次使用就可以用root账号登录或切换到root账号-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719105931285.png)-
再开启一个ssh服务，肯目前kali linux下的服务-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719110242405.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719111619115.png)-
（3）使用ip地址的最后一个字节扫描

    nmap 192.168.1.12,13,14
    
        

（4）扫描一个ip地址的范围，通过- 连接

    nmap 192.168.0.109-115
    
        

（5）从文件中扫描对应ip

    nmap -iL 文件所在路径
    
        

如：新建一个记事本，写入几个ip-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719142056837.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lkX18zOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
（6）扫描整个子网

    nmap 192.168.1.*
    
        

（7）排除一些远程主机后再扫描

    使用-exclude
    nmap 192.168.1.114-118 -exclude 192.168.1.115
    
        

（8）扫描排除某文件中的ip

    使用 -excludefile 文件路径
    
        

（9）扫描操作系统信息和路由信息

    使用 -A
    nmap -A ip
    
        

（10）使用nmap进行系统探测

    -o
    nmap -O ip
    
        

（11）主机检测是否有防火墙保护

    参数：-PN
    nmap -PN ip
    
        

（12）快速扫描-
扫描的端口少于默认的端口，只扫描nmap-service文件中的端口，避免其他端口

    nmap -F ip
    
        

（13）扫描指定的端口号，避免扫描不需要的端口

    参数 -p
    nmap ip -p 端口号1,端口号2,端口号... 
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719152255614.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lkX18zOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
（14）扫描指定的连续的端口号

    nmap ip -p port1-port2
    
        

    目标规格：
      可以传递主机名，IP地址，网络等。
      例如：scanme.nmap.org，microsoft.com / 24、192.168.0.1；10.0.0-255.1-254
      -iL <输入文件名>：来自主机/网络列表的输入
      -iR <num主机>：选择随机目标
      --exclude <host1 [，host2] [，host3]，...>：排除主机/网络
      --excludefile <exclude_file>：从文件中排除列表
    
    主机发现：
      -sL：列出扫描-仅列出要扫描的目标
      -sn：Ping扫描-禁用端口扫描
      -Pn：将所有主机视为联机-跳过主机发现
      -PS / PA / PU / PY [端口列表]：对给定端口的TCP SYN / ACK，UDP或SCTP发现
      -PE / PP / PM：ICMP回显，时间戳和网络掩码请求发现探针
      -PO [协议列表]：IP协议Ping
      -n / -R：从不进行DNS解析/始终解析[默认值：有时]
      --dns-servers <serv1 [，serv2]，...>：指定自定义DNS服务器
      --system-dns：使用操作系统的DNS解析器
      --traceroute：跟踪到每个主机的跃点路径
    
    扫描技术：
      -sS / sT / sA / sW / sM：TCP SYN / Connect（）/ ACK / Window / Maimon扫描
      -sU：UDP扫描
      -sN / sF / sX：TCP空，FIN和Xmas扫描
      --scanflags <标志>：自定义TCP扫描标志
      -sI <僵尸主机[：probeport]>：空闲扫描
      -sY / sZ：SCTP INIT / COOKIE-ECHO扫描
      -sO：IP协议扫描
      -b <FTP中继主机>：FTP退回扫描
    
    端口规格和扫描顺序：
      -p <端口范围>：仅扫描指定的端口
        例如：-p22; -p1-65535; -p U：53,111,137，T：21-25,80,139,8080，S：9
      --exclude-ports <端口范围>：从扫描中排除指定端口
      -F：快速模式-扫描的端口少于默认扫描
      -r：连续扫描端口-不要随机化
      --top-ports <编号>：扫描<编号>最常见的端口
      --port-ratio <比率>：扫描端口比<比率>更常见
    
    服务/版本检测：
      -sV：探测打开的端口以确定服务/版本信息
      --version-intensity <级别>：设置为0（浅）至9（尝试所有探针）
      --version-light：限制为最可能的探针（强度2）
      --version-all：尝试每个探针（强度9）
      --version-trace：显示详细的版本扫描活动（用于调试）
    
    脚本扫描：
      -sC：相当于--script = default
      --script = <Lua脚本>：<Lua脚本>是逗号分隔的列表
               目录，脚本文件或脚本类别
      --script-args = <n1 = v1，[n2 = v2，...]>：提供脚本参数
      --script-args-file =文件名：在文件中提供NSE脚本args
      --script-trace：显示所有发送和接收的数据
      --script-updatedb：更新脚本数据库。
      --script-help = <Lua脚本>：显示有关脚本的帮助。
               <Lua脚本>是逗号分隔的脚本文件列表，或者
               脚本类别。
    
    操作系统检测：
      -O：启用操作系统检测
      --osscan-limit：将操作系统检测限制为有希望的目标
      --osscan-guess：更积极地猜测操作系统
    时序和性能：
      花费<time>的选项以秒为单位，或附加“ ms”（毫秒），
      “ s”（秒），“ m”（分钟）或“ h”（小时）到值（例如30m）。
      -T <0-5>：设置时序模板（越高越快）
      --min-hostgroup / max-hostgroup <大小>：并行主机扫描组大小
      --min-parallelism / max-parallelism <numprobes>：探针并行化
      --min-rtt-timeout / max-rtt-timeout / initial-rtt-timeout <时间>：指定
          探测往返时间。
      --max-retries <tries>：限制端口扫描探针重传的次数。
      --host-timeout <时间>：长时间后放弃目标
      --scan-delay /-max-scan-delay <时间>：调整探头之间的延迟
      --min-rate <number>：每秒发送数据包的速度不低于<number>
      --max-rate <number>：每秒发送数据包的速度不超过<number>
    
    防火墙/标识闪避和疏散：
      -F; --mtu <val>：分段数据包（可选，带有给定的MTU）
      -D <decoy1，decoy2 [，ME]，...>：用诱饵掩盖扫描
      -S <IP地址>：欺骗源地址
      -e <iface>：使用指定的接口
      -g /-source-port <端口号>：使用给定的端口号
      --proxies <url1，[url2]，...>：通过HTTP / SOCKS4代理中继连接
      --data <十六进制字符串>：将自定义有效负载附加到发送的数据包
      --data-string <字符串>：将自定义ASCII字符串附加到发送的数据包
      --data-length <num>：将随机数据附加到发送的数据包
      --ip-options <选项>：发送具有指定ip选项的数据包
      --ttl <val>：设置IP生存时间字段
      --spoof-mac <mac地址/前缀/供应商名称>：欺骗您的MAC地址
      --badsum：发送带有虚假TCP / UDP / SCTP校验和的数据包
    输出：
      -oN / -oX / -oS / -oG <文件>：以普通，XML，s | <rIpt kIddi3，
         和Grepable格式分别更改为给定的文件名。
      -oA <basename>：一次以三种主要格式输出
      -v：提高详细程度（使用-vv或更高的效果更好）
      -d：提高调试级别（使用-dd或更多以获得更大的效果）
      --reason：显示端口处于特定状态的原因
      --open：仅显示打开（或可能打开）的端口
      --packet-trace：显示所有发送和接收的数据包
      --iflist：打印主机接口和路由（用于调试）
      --append-output：追加而不是破坏指定的输出文件
      --resume <文件名>：恢复中止的扫描
      --stylesheet <路径/ URL>：XSL样式表，可将XML输出转换为HTML
      --webxml：Nmap.Org的参考样式表，用于更便携式的XML
      --no-stylesheet：防止关联带有XML输出的XSL样式表
    MISC：
      -6：启用IPv6扫描
      -A：启用操作系统检测，版本检测，脚本扫描和跟踪路由
      --datadir <目录名>：指定自定义Nmap数据文件位置
      --send-eth /-send-ip：使用原始以太网帧或IP数据包发送
      --privileged：假设用户具有完全特权
      --unprivileged：假设用户缺乏原始套接字特权
      -V：打印版本号
      -h：打印此帮助摘要页面。
    
        

附带一个链接，写的hen详细：https://blog.csdn.net/yalecaltech/article/details/70943898

#### 5.nmap 进阶

**NSE脚本引擎**-
Nmap Script Engine，内置很多用来扫描针对特定任务的脚本-
Windows的：nmap安装目录下的scripts-
kali linux中：/usr/share/nmap/scripts 文件夹中-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719162250155.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lkX18zOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
每个文件的命名开头的含义：-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719162423327.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719162445943.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lkX18zOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719162505905.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lkX18zOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719162523863.png)-
NSE使用：-
（1）使用nmap探测web服务的title信息

    nmap --script 脚本名称 目标（可以是ip、www.xxx.）
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719163125661.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2lkX18zOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
（2） 探测头部信息-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719165459685.png)-
（3）使用 漏洞分类脚本vuln

    nmap  -sV  --script  vuln  目标
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200719165824349.png)-
（4）使用 --script=“version,discovery” 进行信息分类

    nmap  -sV -F   --script="version,discovery"  192.168.0.109
    
        

[查看原网页: blog.csdn.net](https://blog.csdn.net/id__39/article/details/107413307)



# Quote
[Nmap中文手册 - Nmap中文网](http://www.nmap.com.cn/doc/manual.shtm#13)
