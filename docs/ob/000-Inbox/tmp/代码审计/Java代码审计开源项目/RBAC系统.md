
# JavaWeb代码审计实战之某RBAC系统，非常适合小白入门练习的系统

[power7089.github.io](https://power7089.github.io/2022/08/22/JavaWeb%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1%E5%AE%9E%E6%88%98%E4%B9%8B%E6%9F%90RBAC%E7%B3%BB%E7%BB%9F%EF%BC%8C%E9%9D%9E%E5%B8%B8%E9%80%82%E5%90%88%E5%B0%8F%E7%99%BD%E5%85%A5%E9%97%A8%E7%BB%83%E4%B9%A0%E7%9A%84%E7%B3%BB%E7%BB%9F/)Power7089


## 一、前置知识

#### **A、涉及相关技术简介**

#### A1、Maven简介

Maven 是一个项目管理工具，它包含了一个项目对象模型（Project Object Model），反映在配置中，就是一个 pom.xml 文件。是一组标准集合，一个项目的生命周期、一个依赖管理系统，另外还包括定义在项目生命周期阶段的插件(plugin)以及目标(goal)。

当我们使用 Maven 的使用，通过一个自定义的项目对象模型，pom.xml 来详细描述我们自己的项目。

简单来说，我们开发一个JavaWeb项目是需要加载很多依赖的，使用Maven可以便于管理这些依赖。

*   pom.xml

POM是项目对象模型(Project Object Model)的简称,它是Maven项目中的文件，使用XML表示，名称叫做`pom.xml`。该文件用于管理：源代码、配置文件、开发者的信息和角色、问题追踪系统、组织信息、项目授权、项目的url、项目的依赖关系等等。Maven项目中必须包含`pom.xml`文件。

需要导入的依赖应该在`pom.xml`中进行配置与填写。比如导入某些依赖，如下图所示：

![maven的pom例子](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fmaven%25E7%259A%2584pom%25E4%25BE%258B%25E5%25AD%2590.jpg)

![maven依赖版本信息](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fmaven%25E4%25BE%259D%25E8%25B5%2596%25E7%2589%2588%25E6%259C%25AC%25E4%25BF%25A1%25E6%2581%25AF.jpg)

`project`\- `project` 是 pom.xml 中描述符的根。

`modelVersion` - `modelVersion` 指定 pom.xml 符合哪个版本的描述符。maven 2 和 3 只能为 4.0.0。

`parent` - maven 支持继承功能。子 POM 可以使用 `parent` 指定父 POM ，然后继承其配置。

`dependencies` - 在dependencise中进行依赖配置

`groupId` - 团体、组织的标识符。团体标识的约定是，它以创建这个项目的组织名称的逆向域名(reverse domain name)开头。一般对应着 java 的包结构。

`artifactId` - 单独项目的唯一标识符。比如我们的 tomcat、commons 等。不要在 artifactId 中包含点号(.)。

`version` - 版本信息。

*   使用IDEA创建Maven项目

1、打开IDEA，点击`Create New Porject`，选择`Maven`，如下图所示：

![maven创建](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fmaven%25E5%2588%259B%25E5%25BB%25BA.jpg)

2、默认即可（在真实需求中，可以根据自己的项目，选择不同模板），点击Next，然后点击Finish。一个最基本的`Maven项目结构`如下图所示：

![maven项目结构](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fmaven%25E9%25A1%25B9%25E7%259B%25AE%25E7%25BB%2593%25E6%259E%2584.jpg)

配置源加速，自行百度就可以了。

#### A2、SpringBoot简介

SpringBoot是一款基于JAVA的开源框架。目的是为了简化Spring应用搭建和开发流程。是目前比较流行，大中小型企业常用的框架。正因为极大简化了开发流程，才收到绝大开发人员的喜爱。SpringBoot核心原理是自动装配（自动配置），在这之前，开发一个JavaWeb，Spring等项目要进行很多配置，使用了SpringBoot就不用在过多考虑这些方面。并且在SpringBoot中还内置了Tomcat。

*   SpringBoot之HelloWorld

通过经典HelloWorld程序，来看看Springboot项目搭建多么简便。

1、打开IDEA，选择`Create New Project`，选择`Spring Initializer`，右侧勾选`Default`，如下图所示：

![SpringBootHelloworld1](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2FSpringBootHelloworld1.jpg)

2、点击Next，`Srping Initializr Project Settings`配置内容默认就好，我们不做实际项目开发，如下图所示：

![helloworld配置](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fhelloworld%25E9%2585%258D%25E7%25BD%25AE.jpg)

3、点击Next，进入依赖项选择页面，我们选择`Web -> Spring Web`这一个即可，如下图所示：

![helloworld依赖选择](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fhelloworld%25E4%25BE%259D%25E8%25B5%2596%25E9%2580%2589%25E6%258B%25A9.jpg)

4、点击Next，填写项目名称和存放地址。练习项目，默认就可以，点击`Finish`，完成创建。

5、Maven自动加载完所需依赖后，整体项目结构如下图所示：

![helloworld项目结构](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fhelloworld%25E9%25A1%25B9%25E7%259B%25AE%25E7%25BB%2593%25E6%259E%2584.jpg)

`@SpringBootApplication`注解表示这个类为SpringBoot的主配置类，SpringBoot项目应运行这个类下面的main方法来启动SpringBoot应用。

6、创建`HelloController`，创建一个`controller`包，下面创建一个HelloController，在该controller中编写代码，如下图所示：

![helloworld代码](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fhelloworld%25E4%25BB%25A3%25E7%25A0%2581.jpg)

7、点击右上方运行，打开浏览器输入`http://127.0.0.1:8080/hello`，即可看到helloworld，如下图所示：

![返回helloworld](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E8%25BF%2594%25E5%259B%259Ehelloworld.jpg)

`@Controller`注解：标注该类为controller类，可以处理http请求。@Controller一般要配合模版来使用。现在项目大多是前后端分离，后端处理请求，然后返回JSON格式数据即可，这样也就不需要模板了。

`@ResponseBody`注解：将该注解写在类的外面，表示这个类所有方法的返回的数据直接给浏览器。 `@RestController 相当于 @ResponseBody 加上 @Controller`

`@RequestMapping`注解：配置`URL映射`，可以作用于某个Controller类上，也可以作用于某Controller类下的具体方法中，说白了就是URL中请求路径会直接映射到具体方法中执行代码逻辑。

`@PathVariable`注解：接受请求URL路径中占位符的值，示例代码如下图所示：

1-
2-
3-
4-
5-
6-
7-
8-
9-

    @Controller@ResponseBody@RequestMapping("/hello")public class HelloController {    @RequestMapping("/whoami/{name}/{sex}")    public String  hello(@PathVariable("name") String name, @PathVariable("sex") String sex){        return "Hello" + name + sex;    }}

`@RequestParam`注解：将请求参数绑定到你控制器的方法参数上（是springmvc中接收普通参数的注解），常用于POST请求处理表单。

#### A3、SpringSecurity简介

Spring 是一个非常流行和成功的java应用开发框架。 Spring Security 基于Spring 框架，提供了一套web应用安全性的完整解决方案。

一般来说，Web 应用的安全性包括两部分：

　　1. 用户**认证**（Authentication）

　　用户认证指的是验证某个用户是否为系统中的合法主体，也就是说用户能否访问该系统。用户认证一般要求用户提供用户名和密码。系统通过校验用户名和密码来完成认证过程。

　　2. 用户**授权**（Authorization）

　　用户授权指的是验证某个用户是否有权限执行某个操作。在一个系统中，不同用户所具有的权限是不同的。比如对一个文件来说，有的用户只能进行读取，而有的用户可以进行修改。

一般来说，系统会为不同的用户分配不同的角色，而每个角色则对应一系列的权限。

对于上面提到的两种应用情景，Spring Security 框架都有很好的支持。

在用户**认证**方面，Spring Security 框架支持主流的认证方式，包括 HTTP 基本认证、HTTP 表单验证、HTTP 摘要认证、OpenID 和 LDAP 等。

在用户**授权**方面，Spring Security 提供了基于角色的访问控制和访问控制列表（Access Control List，ACL），可以对应用中的领域对象进行细粒度的控制。

#### A4、Mybatis简介

MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。

MyBatis可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

它内部封装了jdbc，使开发者只需要关注sql语句本身，而不需要花费精力去处理加载驱动、创建连接、创建statement等繁杂的过程。

官网：

1-

    https://mybatis.org/mybatis-3/zh/index.html

配置文件常存放在`src/main/resources/mapper`中，配置文件命名为`xxxxMapper.xml`

Mybatis拼接sql有下面两种方式：

`#{}`告诉 MyBatis 创建一个预编译语句（PreparedStatement）参数，在 JDBC 中，这样的一个参数在 SQL 中会由一个“?”来标识，并被传递到一个新的预处理语句中。

`${}` 仅仅是纯粹的 string 替换，在动态 SQL 解析阶段将会进行变量替换，类似于直接替换字符串，会导致SQL注入产生。like+#{ }

#### A5、Swagger简介

Swagger 是一款RESTful接口的文档在线自动生成加功能测试的软件。目的是为了减少与其他团队的沟通成本，因此会使用Swagger构建RESTful API文档来描述所有的接口信息。

![swagger页面](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fswagger%25E9%25A1%25B5%25E9%259D%25A2.jpg)

官方网站：

1-

    https://swagger.io/

常见Swagger敏感信息泄露的路径：

1-
2-
3-
4-
5-
6-
7-
8-
9-
10-
11-
12-
13-
14-

    /swagger//api/swagger//swagger/ui//api/swagger/ui//swagger-ui.html//api/swagger-ui.html//user/swagger-ui.html//swagger/ui//api/swagger/ui//libs/swaggerui//api/swaggerui//swagger-resources/configuration/ui//swagger-resources/configuration/security/......

Swagger组件特征固定title：`Swagger UI`

#### A6、Thymeleaf简介

官方学习文档：

1-
2-

    https://www.thymeleaf.org/https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html

Thymeleaf是一个流行的模板引擎，该模板引擎采用Java语言开发。模板引擎（这里特指用于Web开发的模板引擎）是为了使用户界面与业务数据（内容）分离而产生的，它可以生成特定格式的文档，用于网站的模板引擎就会生成一个标准的html文档。从字面上理解模板引擎，最重要的就是模板二字，这个意思就是做好一个模板后套入对应位置的数据，最终以html的格式展示出来，这就是模板引擎的作用。

例子：

1-
2-
3-
4-
5-
6-
7-
8-
9-
10-
11-
12-
13-
14-

    <table>  <thead>    <tr>      <th th:text="#{msgs.headers.name}">Name</th>      <th th:text="#{msgs.headers.price}">Price</th>    </tr>  </thead>  <tbody>    <tr th:each="prod : ${allProducts}">      <td th:text="${prod.name}">Oranges</td>      <td th:text="${#numbers.formatDecimal(prod.price,1,2)}">0.99</td>    </tr>  </tbody></table>

拓展学习，大致了解每个标签作用：

1-

    https://www.w3xue.com/exp/article/20199/54847.html

#### A7、SpringBoot Actuator简介

Actuator主要用于公开有关正在运行的应用程序的运行信息 - 运行状况，指标，信息，转储，env等等。它使用HTTP端点或JMX bean来使我们能够与它进行交互。

一些常见的执行端点：

1-
2-
3-
4-
5-
6-
7-
8-
9-
10-

    /beans：此端点返回应用程序中配置的所有bean的列表。/env：提供有关Spring Environment属性的信息。/health：显示应用程序运行状况/info：显示应用程序信息，我们可以在Spring环境属性中配置它。/mappings：显示所有 @RequestMapping 路径的列表 。/shutdown：允许我们正常关闭应用程序。/threaddump：提供应用程序的线程转储。完整可执行的端点，详见官网：https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.endpoints

#### A8、Druid简介

Druid是阿里研发的一款数据库连接池，其开发语言为java，druid集合了c3p0、dbcp、proxool等连接池的优点，还加入了日志监控、session监控等数据监控功能，使用Druid能有效的监控DB池连接和SQL的执行情况。

项目地址：

1-

    https://github.com/alibaba/druid

### **B、相关漏洞**

上述常见的组件存在已知的漏洞，具体对应如下（不完全统计）：

漏洞名称

Spring Security验证绕过漏洞

Spring Security认证绕过

Spring Boot框架SPEL表达式注入漏洞

Spring Security未经授权的访问

Spring Boot Actuator命令执行漏洞

Spring Boot Actuator hikari配置不当导致的远程命令执行漏洞

Spring Boot Actuator jolokia 配置不当导致的XXE漏洞

Spring Boot Thymeleaf 模板注入

Spring Boot Tomcat导致的JNDI注入

Spring Boot eureka xstream deserialization rce

Spring Boot h2 database query rce

Spring Boot mysql jdbc deserialization rce

Spring Boot sql

Spring Boot whitelabel error page SpEL rce

Spring Boot 修改环境属性导致的rce

Spring Boot 提取内存密码

Spring Boot 获取被星号脱敏的密码的明文 (方法一)

Spring Boot 路由地址及接口调用详情泄漏

Spring Boot 配置不当而暴露的路由

Druid未授权访问

## 二、项目安装

### **A、所需环境**

本项目安装基于**windows 10**操作系统搭建而成。

#### 1、Java环境部署

Java版本如下图所示：

![javaversion](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fjavaversion.jpg)

JDK下载链接：

1-

    https://www.oracle.com/java/technologies/downloads/#java8-windows

安装步骤操作简单，只需下一步即可，不过多赘述。

#### 2、Maven环境部署

关于Maven环境部署与安装，可参考下面的文章，安装最新版即可。

1-

    https://www.runoob.com/maven/maven-setup.html

在IDEA中内置了Maven，对于我们来说足够用了。

**Maven加速配置**

配置国内源下载一些依赖组件会非常快，但会有极个别情况，有些组件使用国内源无法下载，则需要再更改配置，大家留有印象就好。

①、访问`c:\Users\当前用户\.m2`目录，当前用户文件夹需要根据当前用户来定，如下图所示：

![maven加速](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fmaven%25E5%258A%25A0%25E9%2580%259F.jpg)

②、打开`settings.xml`文件，复制粘贴以下内容：



    <?xml version="1.0" encoding="UTF-8"?><settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"	xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">	<pluginGroups></pluginGroups>	<proxies></proxies>    <localRepository>C:\Users\当前用户\.m2\repository</localRepository>	<servers>	</servers>		<mirrors>		<mirror>			<id>aliyunmaven</id>			<mirrorOf>*</mirrorOf>			<name>阿里云公共仓库</name>		<url>https://maven.aliyun.com/repository/public</url>		</mirror> 	</mirrors>		<profiles>	</profiles>		<activeProfiles>		<activeProfile>nexus</activeProfile>	</activeProfiles></settings>

注意代码中的`当前用户`该位置路径应与你当前用户一致。

#### 3、Mysql环境部署

个人偏好于使用phpstudy，它集成了很多常用组件，如apache，mysql等。作为练习，方便至极，一键启动即可使用。

**官方下载链接：**

1-

    https://www.xp.cn/

下载完成后，双击进入软件。进入首页处，选择mysql套件，点击启动即可，如下图所示：

![mysql启动](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fmysql%25E5%2590%25AF%25E5%258A%25A8.jpg)

点击左侧`数据库`，可以对数据库进行`密码修改`操作。

#### 4、IDEA

官方下载地址，可选择使用`Ultimate`版本。

1-

    https://www.jetbrains.com/zh-cn/idea/download/#section=windows

### **B、环境搭建**

安装整体过程首先将数据导入mysql数据库中，然后将项目导入IDEA中，修改配置文件中数据库信息，点击启动即完成环境搭建。

①、在phpstudy中启动Mysql

![启动mysql](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E5%2590%25AF%25E5%258A%25A8mysql.jpg)

②、启动cmd命令行，进入mysql数据库，命令：`mysql -u root -p`，然后键入你的密码。

![进入mysql](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E8%25BF%259B%25E5%2585%25A5mysql.jpg)

③、使用命令创建数据库：`create database rbac;`。使用命令选择rbac数据库：`use rbac;`

![创建rbac数据库](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E5%2588%259B%25E5%25BB%25BA%25E4%25B8%258E%25E9%2580%2589%25E6%258B%25A9rbac%25E6%2595%25B0%25E6%258D%25AE%25E5%25BA%2593.jpg)

④、导入`rbac.sql`文件，该文件位于`RefiningStone-RBAC` 项目文件夹内。在导入时务必注意路径中的正斜杠。使用命令`source`导入数据，如下图所示：

![导入rbac](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E5%25AF%25BC%25E5%2585%25A5rbac.jpg)

最后全部为`Query OK...`，无报错，即为全部导入成功。

⑤、打开IDEA，点击`Open or import`，选择项目文件夹下的`pom.xml`文件，最后选择打开方式为`Open as project`，如下图所示：

![导入项目1](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E5%25AF%25BC%25E5%2585%25A5%25E9%25A1%25B9%25E7%259B%25AE1.jpg)

![导入项目2](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E5%25AF%25BC%25E5%2585%25A5%25E9%25A1%25B9%25E7%259B%25AE2.jpg)

也可以右键选择文件夹，点击`Open folder as Intellij IDEA project......`

第一次导入项目，Maven会自动下载所需依赖，会花费一些时间。

⑥、几个现象表明项目部署成功。`pom.xml`文件无报错，项目代码已编译为`class`，`Edit Configurations...`处显示可以运行。

![项目导入成功](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E9%25A1%25B9%25E7%259B%25AE%25E5%25AF%25BC%25E5%2585%25A5%25E6%2588%2590%25E5%258A%259F.jpg)

⑦、进入`src - main - resources - application.yml`，对配置文件进行相关修改，主要修改Mysql数据库连接账号密码，具体可以去phpstudy下数据库处查看。

![修改数据库密码](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E4%25BF%25AE%25E6%2594%25B9%25E6%2595%25B0%25E6%258D%25AE%25E5%25BA%2593%25E5%25AF%2586%25E7%25A0%2581.jpg)

⑧、万事具备，点击右上侧启动即可。下侧console控制台中信息无任何报错，即为启动成功，另外需要注意的是启动端口是多少，启动端口可以在`application.yml`文件中更改`port`的值。![点击启动](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E7%2582%25B9%25E5%2587%25BB%25E5%2590%25AF%25E5%258A%25A8.jpg)

⑨、打开浏览器，键入`http://127.0.0.1:8088/login.html`，访问项目。

![访问项目](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E8%25AE%25BF%25E9%2597%25AE%25E9%25A1%25B9%25E7%259B%25AE.jpg)

至此，环境已搭建完毕，可以开始面的练习了。

**注意：**

在搭建中难免会出现各种问题。务必先自行分析报错，尝试解决问题。

实在搞不定，可以在群里对问题进行描述后@我。

## 三、漏洞挖掘之渗透测试

### 3.1、信息收集

*   dirsearch扫目录

使用dirsearch等相关敏感目录扫描工具，对目标进行扫描，探测敏感目录。

执行命令：



    github:https://github.com/maurosoria/dirsearchpy -3 dirsearch.py -u http://127.0.0.1:8088/ -e * -x 404,500,400-e *：探测的扩展列表，比如asp,jsp,php等，*（星号）表示所有。-x：表示屏蔽的响应码，根据自己实际需求填写，使用逗号分隔。

扫描结果，如下图所示：

![dirsearch扫描结果](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fdirsearch%25E6%2589%25AB%25E6%258F%258F%25E7%25BB%2593%25E6%259E%259C.jpg)

*   异常信息泄露

很多系统没有处理好全局异常信息的话，会直接将异常报错信息返回到前端。异常报错信息中可能会携带敏感信息。

一般情况下`异常信息泄露`会配合`4XX`，`5XX`等响应码出现。

**举例说明：**

`400`响应码：因发送的请求语法错误，服务器无法正常读取。

`500`响应码：（服务器内部错误） 服务器遇到错误，无法完成请求。

在使用dirsearch探测目录时，如果没有屏蔽400，500等响应码时，可能会触发相关报错。我们可以复制路径进一步查看报错，如下图所示：

![异常信息泄露-dirsearch](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E5%25BC%2582%25E5%25B8%25B8%25E4%25BF%25A1%25E6%2581%25AF%25E6%25B3%2584%25E9%259C%25B2-dirsearch.jpg)

![异常信息泄露2](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E5%25BC%2582%25E5%25B8%25B8%25E4%25BF%25A1%25E6%2581%25AF%25E6%25B3%2584%25E9%259C%25B22.jpg)

也可以在请求报文中构造畸形报文，比如使用`@#$%!`等字符，或使用空格分隔路径，触发4XX或5XX错误，如下图所示：

![异常信息泄露-构造畸形报文](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E5%25BC%2582%25E5%25B8%25B8%25E4%25BF%25A1%25E6%2581%25AF%25E6%25B3%2584%25E9%259C%25B2-%25E6%259E%2584%25E9%2580%25A0%25E7%2595%25B8%25E5%25BD%25A2%25E6%258A%25A5%25E6%2596%2587.jpg)

*   Tomcat版本号泄露

如果没有设置统一的报错回显页面，使用服务器默认的报错回显页面，可能会有服务器版本信息。比如下图400报错页面中有Tomcat版本号信息：

![tomcat版本号泄露](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Ftomcat%25E7%2589%2588%25E6%259C%25AC%25E5%258F%25B7%25E6%25B3%2584%25E9%259C%25B2.jpg)

### 3.2、敏感目录之Druid登陆暴力破解

开发人员的粗心和糟糕的安全意识，会导致Druid直接暴露在公网上，并且可能存在未授权访问。尽量是将Druid放在内网，如必需暴露在公网应使用强口令。

Druid如果存在未授权访问或登陆可暴力破解，进入监控页面的话，可以查看很多敏感信息，比如session信息，SQL执行情况等等。

问题路径：

1-

    ip/druid/index.html

使用常见的登陆账号`admin`，对登陆页面进行暴力破解。如下图所示：

![druid登录爆破](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fdruid%25E7%2599%25BB%25E5%25BD%2595%25E7%2588%2586%25E7%25A0%25B4.jpg)

根据响应长度，结果不同得到正确的账号密码。如下图所示：

![druid爆破结果1](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fdruid%25E7%2588%2586%25E7%25A0%25B4%25E7%25BB%2593%25E6%259E%259C1.jpg)

![druid爆破结果2](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fdruid%25E7%2588%2586%25E7%25A0%25B4%25E7%25BB%2593%25E6%259E%259C2.jpg)

**爆破字典**推荐如下：


    建议自己整合一份自己的弱口令字典口令爆破字典，有键盘组合字典、拼音字典、字母与数字混合这三种类型：https://github.com/huyuanzhi2/password_brute_dictionaryWeb Pentesting Fuzz 字典,一个就够了：https://github.com/TheKingOfDuck/fuzzDicts2011-2019年Top100弱口令密码字典 Top1000密码字典 服务器SSH/VPS密码字典 后台管理密码字典 数据库密码字典 子域名字典：https://github.com/k8gege/PasswordDic基于实战沉淀下的各种弱口令字典：https://github.com/fuzz-security/SuperWordlistgithub搜索字典可获取更多。

关于Druid进一步利用，推荐阅读我之前写的一篇文章：



    记录一次Druid未授权访问的实战应用https://cloud.tencent.com/developer/article/1771986或搜索公众号 "玄魂工作室" 后，搜索 “记录一次Druid未授权访问的实战应用”，同样可以阅读该文章。

### 3.3、敏感目录之SpringBoot Actuator的未授权访问

在信息收集阶段，使用`dirsearch`扫描出来了`/actuator`目录，我们访问看看是什么内容，如下图所示：

![actuator端点](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Factuator%25E7%25AB%25AF%25E7%2582%25B9.jpg)

从上图可以看到，该路径泄露了springboot actuator的所有端点，每个端点都会泄露不同程度的信息泄露。

关于具体每个端点的作用，在`项目简介 - A7、SpringBoot Actuator简介`中有提及到。

*   获取`/actuator/env`中星号（\*）密码

访问路径`/env`或者`/actuator/env`，搜索（\*），可以看到被星号脱敏的密码，如下图所示：

![env的星号](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fenv%25E7%259A%2584%25E6%2598%259F%25E5%258F%25B7.jpg)

观察上图脱敏星号地方，被脱敏的密码应该是数据库密码，我们尝试找到密码明文。

**利用下述该方法前提条件：**

1-

    GET请求可正常访问目标 /heapdump 或 /actuator/heapdump 

`heapdump`端点：用于获取应用的 JVM Heap Dump（堆转储文件）

①、首先访问`/heapdump`或`/actuator/heapdump` 端点，下载heapdump文件，如下图所示：

![heapdump下载](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fheapdump%25E4%25B8%258B%25E8%25BD%25BD.jpg)

②、下载`Memory Analyzer`工具：

1-
2-

    该工具需要最低Java11版本https://www.eclipse.org/mat/downloads.php

③、双击打开`Memory Analyzer`工具，点击左上侧`File -> Open Heap Dump...`，选择打开下载下来的heapdump文件，如下图所示：

![打开heapdump](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E6%2589%2593%25E5%25BC%2580heapdump.jpg)

④、等待分析后，默认点击Finish即可

⑤、点击`OQL`，输入以下命令后，点击红色感叹号执行，如下图所示：

1-

    select * from java.util.Hashtable$Entry x WHERE (toString(x.key).contains("password"))

![heapdump数据库密码](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fheapdump%25E6%2595%25B0%25E6%258D%25AE%25E5%25BA%2593%25E5%25AF%2586%25E7%25A0%2581.jpg)

从上图可以看到获取到了数据库密码。

其他获取星号脱敏密码方式，推荐学习：

1-

    https://github.com/LandGrey/SpringBootVulExploit#0x03%E8%8E%B7%E5%8F%96%E8%A2%AB%E6%98%9F%E5%8F%B7%E8%84%B1%E6%95%8F%E7%9A%84%E5%AF%86%E7%A0%81%E7%9A%84%E6%98%8E%E6%96%87-%E6%96%B9%E6%B3%95%E4%B8%80

### 3.4、敏感目录之Swagger接口文档泄露

Swagger接口文档泄露，也就会导致所有的后端接口暴露出来，造成巨大的风险。毕竟每个接口都有详细说明，并且可以直接在线调试。

在dirsearch扫描敏感目录环境，发现了几个有关swagger的路径，访问结果如下图所示：

![swagger接口文档1](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fswagger%25E6%258E%25A5%25E5%258F%25A3%25E6%2596%2587%25E6%25A1%25A31.jpg)

所有的接口都暴露出来了，可以直接进行漏洞挖掘了。即使不进入后台，也是可以正常进行漏洞挖掘。但以实际情况来说，涉及的接口大多也是需要身份认证与授权。但也不要放弃，将所有接口都试试，也许会发现被忽略的地方。

*   调试接口

选择某个接口，点击`Try it out`，如下图所示：

![调试接口1](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E8%25B0%2583%25E8%25AF%2595%25E6%258E%25A5%25E5%258F%25A31.jpg)

然后填写必要参数后，点击`excute`执行，如下图所示：

![接口调试](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E6%258E%25A5%25E5%258F%25A3%25E8%25B0%2583%25E8%25AF%2595.jpg)

上面也说到了，有些接口是需要认证授权才可以进一步操作的，但并不影响我们进行测试，基于此我们可以挖掘未授权相关的漏洞。对于每一个接口都看仔细一些，是否全都需要认证授权。

漏洞挖掘方向就是常规的手法，比如：越权，未授权访问，sql注入，如果存在文件上传/下载功能，也尝试任意文件下载/上传，具体需要根据实际功能分析。

### 3.5、登录暴力破解之绕过图形验证码小技巧

进行完前置信息收集，我们该进入后台进行一番探索了。奈何需要登录，那我们也一展神通，攻破一下登录。

经过简单分析，没有登录信息泄露，比如：`密码错误`，`账号错误`。

这些关键词的提示，可以让我们准确了解到登录账号对错。但是我们测试的系统错误提示为`用户名或密码错误`。

这种情况，我们采用常用账号名与弱口令进行交叉爆破。刨除这种方式，我们还可以通过搜索引擎，通过社工来获取登录账号。

另外，我们发现登录处还有图形验证码。首先尝试进行图形验证码绕过操作，或者使用脚本识别图形验证码，进而进行登录爆破。

*   绕过图形验证码

几个小技巧


    1、重复发送数据包，观察图形验证码是否可重复利用2、如果登录报错会有弹框提示，在不关闭弹窗情况下，在burp中重复发送数据包3、删除图形验证码参数4、将图形验证码参数值置空5、使用万能验证码，比如：000000,6666666、多刷新几次验证码观察是否是重复的几个7、观察图形验证码是否返回到前端8、记住图形验证码，在前端文件中进行搜索9、前端进行的图形验证码认证，实际没向后端发送验证10、删除cookie10、等等，思路决定出路，多尝试，多观察

经过上述测试，发现`图形验证码可重复利用`，或者说图形验证码判断逻辑仅检测了图形验证码是否为空（这是结合了代码审计得出的结果，在实际中也要学会去揣测）。

随便输入几个字符，在分析时发现前端验证了图形验证码的位数，而实际发现后端并没有验证，但这并不影响我们。如下图所示：

![图形验证码绕过](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E5%259B%25BE%25E5%25BD%25A2%25E9%25AA%258C%25E8%25AF%2581%25E7%25A0%2581%25E7%25BB%2595%25E8%25BF%2587.jpg)

*   登录爆破

经过图形验证码分析，我们发现图形验证码可重复利用，或者说图形验证码参数仅判断了是否为空。不必纠结是哪种，没有图形验证码的限制，我们可以进一步进行登录爆破了。

我们已知登录账号为`admin`。

①、将登录数据包发送到`Intruder`模块进行爆破，右键选择`Send to Intruder`，或者使用快捷键`ctrl+i`。

②、在Intruder模块设置密码变量，如下图所示：

![Intruder爆破](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2FIntruder%25E7%2588%2586%25E7%25A0%25B4.jpg)

③、选择完变量后，我们将弱口令加入`Payloads`，进行爆破，如下图所示：

![intruder设置payload](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fintruder%25E8%25AE%25BE%25E7%25BD%25AEpayload.jpg)

④、点击`Start attack`，开始进行爆破，稍等一会，我们观察结果，如下图所示：

![intruder爆破结果1](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fintruder%25E7%2588%2586%25E7%25A0%25B4%25E7%25BB%2593%25E6%259E%259C1.jpg)

![intruder爆破结果2](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fintruder%25E7%2588%2586%25E7%25A0%25B4%25E7%25BB%2593%25E6%259E%259C2.jpg)

使用弱口令进行爆破，我们首先通过观察响应长度不同来初步确定密码，在通过观察响应内容，或手工尝试登录来确定账号密码。

弱口令字典碰撞实际效果不太好，需要根据目标进行定制化构造字典。并且要有意识收集常见的弱口令，或者说账号密码。

Github搜索关键字：爆破字典，弱口令，字典等等

### 3.6、XSS漏洞挖掘

*   什么是XSS漏洞

XSS是跨站脚本攻击(Cross Site Scripting)的简写，为了不与层叠样式表混淆而改写的。攻击者可以利用XSS窃取账号，网页挂马，发动拒绝服务攻击，发送垃圾邮件等等。

XSS漏洞有三种形式，`反射型XSS`，`DOM型XSS`，`存储型XSS`。

`反射型XSS`：直接将输入的攻击语句返回在前端页面，不经过数据库。

`DOM型XSS`：操作DOM节点直接将攻击的语句返回在前端页面，同样不经过数据库。

`存储型XSS`：攻击语句被存储在数据库中，每次调用该页面都会触发XSS攻击语句。

*   XSS靶场练习

推荐经典XSS靶场进行练习，掌握基础，才有突破。

1-
2-

    http://xss-quiz.int21h.jp/https://github.com/lyshark/xss-labs

*   XSS漏洞挖掘

手工挖掘，针对每个输入框，或者有输入回显的地方构造攻击语句，进行漏洞挖掘，观察回显结果。

前端或后端代码中对XSS特殊字符进行了转义/过滤。我们只能一点点通过手工构造攻击语句观察响应结果。

自动化核心大多采用模糊测试进行漏洞挖掘。比如使用工具`xray`的xss检测，`XSSer`，`XSSFORK`……

手工测试几个XSS弹框语句：



    "><script>alert(1)</script><img src=1 onerror=alert(1) />"><svg onload=alert(1)>javascript:alert(1)'-alert(1)-'<x onclick=alert(1)>......

①、登录到后台，寻找输入框进行XSS攻击，经过针对每一个输入框，每一个表单测试后发现系统内存在多处XSS漏洞。

②、举个例子，访问`系统管理 -> 用户管理`，点击新增，在角色名和角色描述中输入XSS攻击语句，如下图所示：

![角色新增XSS](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E8%25A7%2592%25E8%2589%25B2%25E6%2596%25B0%25E5%25A2%259EXSS.jpg)

③、点击提交后，观察响应，发现前端发生了弹框行为，如下图所示：

![角色新增xss弹框](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E8%25A7%2592%25E8%2589%25B2%25E6%2596%25B0%25E5%25A2%259Exss%25E5%25BC%25B9%25E6%25A1%2586.jpg)

上述功能插入的XSS攻击语句被保存在了数据库，每当你访问该页面时都会有弹框行为。

④、发现`系统监控 -> 操作日志`中也触发了XSS漏洞，也就是如果后台存在日志记录，且没对XSS攻击语句进行过滤/转义的话，也会出现XSS漏洞。

或许以后在挖掘XSS漏洞时，我们应该多注意些什么呢。

![XSS操作日志弹框](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2FXSS%25E6%2593%258D%25E4%25BD%259C%25E6%2597%25A5%25E5%25BF%2597%25E5%25BC%25B9%25E6%25A1%2586.jpg)

### 3.7、SQL注入漏洞挖掘

*   什么是SQL注入漏洞

SQL注入主要发生在与数据库交互的地方，前端将数据传入后端后进行了数据库相关操作。如果我们传入的数据没有有被过滤，拼接到SQL中，被当成SQL语句执行，那么我们可以构造攻击语句，进而可以操作数据库。

**SQL几种注入模式：**

1.  基于布尔的盲注，即可以根据返回页面判断条件真假的注入；
2.  基于报错注入，即页面会返回错误信息，或者把注入的语句的结果直接返回在页面中；
3.  基于时间的盲注，即不能根据页面返回内容判断任何信息，用条件语句查看时间延迟语句是否执行（即页面返回时间是否增加）来判断；
4.  联合查询注入，可以使用 union 的情况下的注入；
5.  堆查询注入，可以同时执行多条语句的执行时的注入

对于不同数据库，攻击语句不同。常见的数据库有`Mysql`，`Oracle`，`SQL Server`等等。

**SQL注入必练靶场：**

1-
2-

    SQLi-labs：https://github.com/Audi-1/sqli-labs

*   SQL注入漏洞挖掘

SQL注入漏洞挖掘第一步是找与数据库交互的地方，也所谓的`注入点`。

**手工挖掘，初步判断：**

针对每个与数据库交互的点，使用下述攻击POC，观察响应，进行初步判断。

1-
2-
3-
4-
5-
6-
7-
8-
9-
10-
11-
12-
13-

    ‘’ and '1'='1' or '1'='1’ and '1'='2' or '1'='2' and\u0020'1'='1' and\u0020'%'='' or\u0020'1'='1' and\u0020'1'='2' or\u0020length(user)=3 or\u0020'1'='1' or\u0020length(user)=2 or\u0020'1'='1更多探测语句，可收集相关字典。

**自动化挖掘：**

使用SQL注入神器`SQLmap`，官方地址：`https://sqlmap.org/`

SQLMap 是一个开源的渗透测试工具，可以用来进行自动化检测，利用 SQL 注入漏洞，获取数据库服务器的权限。它具有功能强大的检测引擎，针对各种不同类型数据库的渗透测试的功能选项，包括获取数据库中存储的数据，访问操作系统文件甚至可以通过外带数据连接的方式执行操作系统命令。

使用SQLmap对每个与数据交互的点都进行SQL注入探测。

经过一番探测，发现`系统管理 -> 用户管理 -> 查询`和`系统管理 -> 字典管理 -> 查询`两处存在SQL注入。以其中一个为例进行演示。

①、访问`系统管理 -> 用户管理 -> 查询`功能，输入关键字，点击查询，此时使用Burp抓取数据包，如下图所示：

![sql注入抓包1](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fsql%25E6%25B3%25A8%25E5%2585%25A5%25E6%258A%2593%25E5%258C%25851.jpg)

②、在桌面新建文本文件`r.txt`，并将抓取到的数据包复制进去，如下图所示：

![sqlmap注入rtxt](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fsqlmap%25E6%25B3%25A8%25E5%2585%25A5rtxt.jpg)

③、开启命令行，进入sqlmap文件夹，输入命令`py -3 sqlmap.py -r xxxxx`，回车进行sql注入漏洞探测，如果需要输入选项，敲击回车默认即可，如下图所示：

![sqlmap-r](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fsqlmap-r.jpg)

④、等待一段一时间，经过探测，发现`nickName`参数存在SQL注入漏洞，如下图所示：

![nicknameSQL注入](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2FnicknameSQL%25E6%25B3%25A8%25E5%2585%25A5.jpg)

⑤、使用sqlmap获取当前数据库，命令`py -3 sqlmap.py -r 绝对路径\r.txt --dbs`，如下图所示：

![sqlmap获取dbs](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fsqlmap%25E8%258E%25B7%25E5%258F%2596dbs.jpg)

⑥、sqlmap获取数据库表信息，命令`py -3 sqlmap.py -r xxx\r.txt -D information_schema --tables`，如下图所示：

![sqlmap获取表信息](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fsqlmap%25E8%258E%25B7%25E5%258F%2596%25E8%25A1%25A8%25E4%25BF%25A1%25E6%2581%25AF.jpg)

### 3.8、越权漏洞挖掘

*   什么是越权漏洞

越权，顾名思义，是超越权限或权力范围的意思。越权漏洞也就是超越该账户权限操作其他账号。比如：越权获取敏感信息、越权删除他人订单、越权添加管理账号等等。

越权漏洞主要形成原因为在`对数据进行增删改查时没有进行权限的判定`，或验证权限不充分，从而导致越权漏洞的出现。

**水平越权**：同权限下账号数据的越权读取/操作。

**垂直越权**：不同权限下账号数据的越权读取/操作。比如：普通用户越权操作/读取了管理员数据。

挖掘越权漏洞前置条件最好是准备`两个账号`。对于水平越权准备两个`同权限`账号。对于垂直越权准备两个`不同权限`账号。

除非是自己搭建环境或者是授权的渗透测试项目，否则不建议通过遍历ID方式来测试越权漏洞。

*   越权漏洞手工挖掘

手工漏洞挖掘漏洞比较耗时耗力，但优点在于可以对所有流程有更深入的了解，可以发现更深层问题。

经过深入的手工挖掘，发现系统中存在越权漏洞。详细步骤如下。

①、登录`admin`账号，访问`系统管理 -> 用户管理`，点击`新增`，添加越权测试用的账号，如下图所示：

![添加越权账号](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E6%25B7%25BB%25E5%258A%25A0%25E8%25B6%258A%25E6%259D%2583%25E8%25B4%25A6%25E5%258F%25B7.jpg)

②、点击提交后，退出admin账号，登录`yuequan`账号，初始密码为`123456`，访问`系统管理 -> 用户管理`，随便添加个账号，对比`admin`和`yuequan`两个账号数据，如下图所示：

![越权账号数据](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E8%25B6%258A%25E6%259D%2583%25E8%25B4%25A6%25E5%258F%25B7%25E6%2595%25B0%25E6%258D%25AE.jpg)

![admin账号数据](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fadmin%25E8%25B4%25A6%25E5%258F%25B7%25E6%2595%25B0%25E6%258D%25AE.jpg)

③、通过访问两个不同权限账号下的用户管理，观察到两个账号的用户数据不同。`admin`账号下的用户管理包括了所有的用户信息，`yuequan`账号下仅能看到自己新添加的`yuequan1`的账号。我们进一步验证越权漏洞。登录`admin`账号，访问用户管理功能，选择一个不存在于`yuequan`账号下的账号数据。通过上图对比，选择`test4`为例。

④、此时浏览器打开代理连接到Burp，Burp下的Proxy打开`Intercept on`，然后点击删除`test4`，此时Burp抓取到删除的数据包，记录`userId`参数，将数据包发送到`Repeater`模块，最后将数据包`Drop`掉，我们不做删除操作。如下图所示：

![记录userid](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E8%25AE%25B0%25E5%25BD%2595userid.jpg)

⑤、退出admin账号，登录yuequan账号后，访问用户管理功能，点击删除某个账号，此时同样开始Burp拦截数据包，发送到`Repeater模块`，并将`userId`替换成`6`后，点击`Send`发送数据包，提示`删除用户成功`，如下图所示：

![越权删除账号](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E8%25B6%258A%25E6%259D%2583%25E5%2588%25A0%25E9%2599%25A4%25E8%25B4%25A6%25E5%258F%25B7.jpg)

⑥、最后登录admin账号，进一步验证`test4`账号被越权删除，如下图所示：

![越权删除test4账号](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E8%25B6%258A%25E6%259D%2583%25E5%2588%25A0%25E9%2599%25A4test4%25E8%25B4%25A6%25E5%258F%25B7.jpg)

上述操作，我们可以采用两种方式来进行验证。一是通过替换cookie，二是通过修改userid，这两者的意义是一样的。

*   越权漏洞自动化挖掘

工具推荐BurpSuite插件`Autorize`。官方地址： `https://github.com/portswigger/autorize` ，可通过BurpSuite的`Extender-BApp Store`安装。

安装`Autorize`之前需要先安装`Jython`环境，下载地址：`https://www.jython.org/download.html`。安装教程可自行搜索。

①、登录`yuequan账号`，记录cookie值，并将该值导入到`Autorize`模块，规则配置如下图所示：

![atuorize编写规则](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fatuorize%25E7%25BC%2596%25E5%2586%2599%25E8%25A7%2584%25E5%2588%2599.jpg)

②、然后登录到`admin`账号，对各个功能进行点击操作，即可看到测试结果。有三个测试结果`Bypassed - 红色标记：不存在`，`Enforced - 绿色标记：存在`，`Is enforced??? - 黄色标记：不确定是否存在越权漏洞，需要手工验证`。扫描结果如下图所示：

![autorize扫描结果](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fautorize%25E6%2589%25AB%25E6%258F%258F%25E7%25BB%2593%25E6%259E%259C.jpg)

注意：上述仅为演示，过滤规则不是最优，可根据实际项目场景，进行构造。

举一反三，后台系统一些功能点还存在相关问题，比如`角色管理`，`部门管理`等等。思考一下问题会出在哪里，再配合使用自动化工具，赶紧动手试试吧。

另外，我们通过挖掘越权漏洞时发现，低权限在用户管理处添加的账号，admin账号也能看到。那么配合XSS漏洞，是不是能做些什么骚操作呢？

## 四、项目介绍

#### 4.1、Java的MVC模式

MVC 模式是一种软件框架模式，被广泛应用在 JavaEE 项目的开发中。

MVC 即模型（Model） 、视图（View）、控制器（Controller）。

*   模型（Model）

模型是用于处理数据逻辑的部分。

所谓数据逻辑，也就是数据的映射以及对数据的增删改查，Bean、DAO（data access object，数据访问对象）等都属于模型部分。

*   视图（View）

视图负责数据与其它信息的显示，也就是给用户看到的页面。

html、JSP 等页面都可以作为视图。

*   控制器（controller）

控制器是模型与视图之间的桥梁，控制着数据与用户的交互。

控制器通常负责从视图读取数据，处理用户输入，并向模型发送数据，也可以从模型中读取数据，再发送给视图，由视图显示。

**Java分层思想拓展学习：**

1-

    https://www.cnblogs.com/java-123/p/9174547.html

#### 4.2、本项目目录结构

拿到一个JavaWeb项目，首先要了解项目整体结构。大致了解作者编写逻辑，搞清请求流程。

`src/main`下面有两个目录，分别是`java`和`resources`

`java`目录中主要存放的是java代码

`resources`目录中主要存放的是资源文件，比如：html、js、css等

![目录结构1](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E7%259B%25AE%25E5%25BD%2595%25E7%25BB%2593%25E6%259E%25841.jpg)

在`java`目录下还有其他一些常见目录，具体含义整理如下：

**/java目录下：**

`annotation`：放置项目自定义注解

`controller/`: 存放控制器，接收从前端传来的参数，对访问控制进行转发、各类基本参数校验或者不复用的业务简单处理等。

`dao/`: 数据访问层，与数据库进行交互，负责数据库操作，在Mybaits框架中存放自定义的Mapper接口

`entity/`: 存放实体类

`interceptor/`: 拦截器

`service/`: 存放服务类，负责业务模块逻辑处理。`Service`层中有两种类，一是`Service`，用来声明接口；二是`ServiceImpl`，作为实现类实现接口中的方法。

`utils/`: 存放工具类

`dto/`: 存放数据传输对象（Data Transfer Object），如请求参数和返回结果

`vo/`: 视图对象（View Object）用于封装客户端请求的数据，防止部分数据泄漏，保证数据安全

`constant/`: 存放常量

`filter/`: 存放过滤器

**/resources目录下：**

`mapper/`: 存放Mybaits的mapper.xml文件

`static/`: 静态资源文件目录（Javascript、CSS、图片等），在这个目录中的所有文件可以被直接访问

`templates/`: 存放模版文件

`application.properties`或`application.yml`: Spring Boot默认配置文件

推荐阅读《阿里巴巴Java开发手册》：

1-

    https://ucc-private-download.oss-cn-beijing.aliyuncs.com/e9af0d4111234bfda5813c72a368a168.pdf?Expires=1635387273&OSSAccessKeyId=LTAIvsP3ECkg4Nm9&Signature=oPo6ow0JdAwUqPnXMsYPHZCpxUI%3D

#### 4.3、请求流程（简化）

用户请求URL发送到服务器，服务器解析请求后发送到后端代码处理请求。

在后端代码处，首先经过`Filter(过滤器)`和`Interceptor(拦截器)`，然后根据请求的URL映射到绑定的`Controller`，之后调用`Service`接口类，然后再调用`serviceImpl`接口实现类，最后调用`DAO`。

`controller`：负责简单的逻辑处理和参数校验功能，之后调用Service。

`service`：接口类，主要负责业务模块逻辑处理。

`serviceImpl`：接口实现类，实现类实现service接口中的方法。

`DAO`：如果service涉及数据库操作就会调用dao。DAO主要处理数据库操作。DAO只做中间传递角色，涉及的SQL语句都写在了配置文件Mapper.xml中。位于`src/main/resources/mapper`中。

## 五、漏洞挖掘之代码审计

### 5.1、信息收集

*   pom.xml漏洞挖掘

基本上所有项目都会根据项目所需引入第三方依赖库来使用，毕竟好用又方便。但如果引入带有漏洞的版本就会出现问题。

通过上面也了解到了，`pom.xml`统一管理第三方依赖。通过`groupId`，`artifactId` ，`version` 三个标识，我们可以了解到使用了哪个依赖库以及版本，从而可以确定是否存在漏洞。

使用`Fastjson 1.2.56`版本，该版本存在漏洞，如下图所示：

![fastjson版本漏洞](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Ffastjson%25E7%2589%2588%25E6%259C%25AC%25E6%25BC%258F%25E6%25B4%259E.jpg)

在发现了第三依赖库存在漏洞版本时，可以进一步利用。

但有时，尽管使用了带有漏洞版本的组件，但并没有使用存在漏洞的相关函数。

比如`Fastjson`反序列化漏洞，该项目实际并没有使用到`parseObject`。

**注意：**

`通过组件及版本寻找漏洞，最起码需要知道什么组件时干什么的，然后对于常见的组件看多了，也就知道什么版本有什么漏洞了，熟能生巧。`

自动化检测第三方依赖包漏洞可使用`dependency-check`。

1-
2-
3-
4-

    官方地址：https://owasp.org/www-project-dependency-check/Github地址：https://github.com/jeremylong/DependencyCheck

*   查看配置文件

配置文件都在`src/main/resources`下面，名字通常为`application.yml`或者`application.properties`。配置文件中可能会存在数据库或其他组件的连接信息。

**数据库连接信息账号密码：**

![数据库配置信息](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E6%2595%25B0%25E6%258D%25AE%25E5%25BA%2593%25E9%2585%258D%25E7%25BD%25AE%25E4%25BF%25A1%25E6%2581%25AF.jpg)

**其他配置信息：**

![druid配置信息](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fdruid%25E9%2585%258D%25E7%25BD%25AE%25E4%25BF%25A1%25E6%2581%25AF.jpg)

### 5.2、SQL注入漏洞挖掘

本项目基于Mybatis操作数据库。通过前面的学习，我们了解到Mybatis错误的配置会导致SQL注入漏洞的存在，这是我们挖掘SQL注入漏洞的入口点。

我们先来回顾下：

Mybatis拼接sql有下面两种方式：

`#{}`告诉 MyBatis 创建一个预编译语句（PreparedStatement）参数，在 JDBC 中，这样的一个参数在 SQL 中会由一个“?”来标识，并被传递到一个新的预处理语句中，

`${}` 仅仅是纯粹的 string 替换，在动态 SQL 解析阶段将会进行变量替换，直接替换字符串，会导致SQL注入产生。

一些不能使用`#{}`的场景：

1-
2-
3-
4-

    表名/字段名order by/group bylike模糊查询in

因此，我们在代码审计阶段进行SQL注入漏洞挖掘，应关注`xxxxMapper.xml`中使用`${}`拼接SQL的地方，全局搜索关键字符。Windows快捷键`CTRL+SHIFT+F`（如果快捷键冲突，需自己更改）。

调出`Find in path`，File mask选择文件类型为`*.xml`，键入关键字符`${`，如下图所示：

![sql注入关键词搜索](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fsql%25E6%25B3%25A8%25E5%2585%25A5%25E5%2585%25B3%25E9%2594%25AE%25E8%25AF%258D%25E6%2590%259C%25E7%25B4%25A2.png)

通过上述搜索，发现了SQL语句使用了`like`语句并使用`$`进行拼接参数，这种情况下无疑是存在SQL注入的。

我们逆向追踪所拼接的参数，看看是从前端哪个地方输入进来的。

首先在IDEA中安装`Free Mybatis Plugin` ，该插件方便mapper.xml与mapper接口之间跳转。需在左上角`File -> Settings -> Plugin`中安装。

**逆向追踪参数流程，举例说明**

①、通过搜索还发现`src/main/resources/mybatis-mappers/DictMapper.xml`文件，第17行，存在使用`like`语句以及`$`拼接SQL。安装了`Free Mybatis Plugin`插件后，左侧会有绿色箭头，点击即可跳转到mapper接口处，也就是DAO层文件，如下图所示：

![dictmapper第17行](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fdictmapper%25E7%25AC%25AC17%25E8%25A1%258C.jpg)

![dictmapper跳转到接口](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fdictmapper%25E8%25B7%25B3%25E8%25BD%25AC%25E5%2588%25B0%25E6%258E%25A5%25E5%258F%25A3.jpg)

②、我们继续向前找到谁调用了`DictDao.java`中的`getFuzzyDictByPage`，Windows系统按住`ctrl`键，鼠标左击`getFuzzyDictByPage`进入该方法，或者说查看谁用了这个方法。如下图所示：

![查看调用链](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E6%259F%25A5%25E7%259C%258B%25E8%25B0%2583%25E7%2594%25A8%25E9%2593%25BE.jpg)

③、点击进入`DICTServiceImpl.java`文件，发现`getDictPage`函数中使用了\`\`getFuzzyDictByPage\`，如下图所示：

![getdictpage函数](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fgetdictpage%25E5%2587%25BD%25E6%2595%25B0.jpg)

④、继续按住`ctrl`键，鼠标左击`getDictPage`方法，点击后，直接进入到了`DictController`文件中第43行，如下图所示：

![dictcontroller](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fdictcontroller.jpg)

找到Controller层后，发现并没有dictname参数，因为Controller中以实体类接收表单数据。进入`MyDict`发现该类为实体类，其中定义多个参数，如下图所示：

![MyDict实体类](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2FMyDict%25E5%25AE%259E%25E4%25BD%2593%25E7%25B1%25BB.jpg)

⑥、经过上述参数逆向追踪，确定了`dictName为字典名称`位于`查询字典功能`处，接下来我们访问该功能，配合渗透测试进一步挖掘验证SQL注入漏洞。

### 5.3、越权漏洞挖掘

在渗透测试阶段挖掘漏洞时发现了越权漏洞，我们现在从代码审计角度跟踪一下删除流程。

通过URL路径`api/user`查看相关Controller，搜索发现为`UserController`，位于`src/main/java/com/codermy/myspringsecurity/plus/admin/controller/UserControlle`。进一步查找关于删除用户的的方法，发现在该文件的第115行~123行，通过`userId`删除用户，如下图所示：

![越权controller](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E8%25B6%258A%25E6%259D%2583controller.jpg)

跟踪一下删除用户逻辑流程。

①、通过上述代码，发现`UserController`从前端接收`userId`参数，调用`userService.deleteUser(userId)`进行删除，此处无任何权限关联。

②、通过`userService`找到`userServiceImpl`实现类，位于`src/main/java/codermy/myspringsecurity/admin/service/impl/UserServiceImpl`。从中找到`deleteUser()`方法，如下图所示：

![删除用户实现类](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E5%2588%25A0%25E9%2599%25A4%25E7%2594%25A8%25E6%2588%25B7%25E5%25AE%259E%25E7%258E%25B0%25E7%25B1%25BB.jpg)

③、从中可以看到，该方法首先使用了`checkUserAllowed()`方法根据`userId`校验用户是否允许操作。然后分别调用了`roleUserDao.deleteRoleUserByUserId(userId);`，`userJobDao.deleteUserJobByUserId(userId);`，`userDao.deleteUserById(userId);`通过`userId`删除用户，并且通过用户ID删除用户和岗位关联。按住`CTRL`键左击`deleteUserxxx`调用的方法，可以跳转到DAO层文件。如下图所示：

![userdao层](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fuserdao%25E5%25B1%2582.jpg)

![userjobdao层](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fuserjobdao%25E5%25B1%2582.jpg)

![role](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Frole.jpg)

跟着删除用户整体流程逻辑走下来，发现userId是从用户侧获取的，也没有将userId与所删除的数据做关联，而且userId也不是从session中获取的，从而导致任意用户都可以调用该接口进行删除。

并且用户ID值极易进行遍历，从而进行批量任意用户删除。

### 5.4、XSS漏洞挖掘

在开发一个网站项目时，开发人员不可能将所有的参数分别做过滤或转义。最好的办法是统一写个过滤/转义XSS的Filter。

在本项目中全局搜索，没发现XSS相关过滤器（Filter）或拦截器（Interceptor）。初步判断存在XSS漏洞。

对后端简单分析后，初步判断存在XSS漏洞，然后我们对前端进行分析，是否有转义机制。

在了解了整个项目之后，发现前端使用了`Thymeleaf模板引擎`和`Layui框架`。

Thymeleaf中`th:text`标签进行渲染的时候，默认对特殊字符进行了转义，`th:utext`不会将字符转义。也就是说使用了`th:utext`标签会出现XSS漏洞。经过全局搜索，发现只有两处使用了`th:text`标签。没有地方使用`th:utext`标签。

其经过分析`src/resouces/templates/`下的html文件。发现本项目前端还使用了`Layui框架`。

全局搜索`Layui`关键字，得到layui使用版本为`2.5.6`

通过翻阅Layui的github中ISSUES发现该版本存在XSS漏洞。链接`https://github.com/sentsin/layui/issues/711`

#### **Layui.table：**

1-

    https://www.layuion.com/doc/modules/table.html#use

table 模块是我们的又一走心之作，在 layui 2.0 的版本中全新推出，是 layui 最核心的组成之一。它用于对表格进行一些列功能和动态化数据操作，涵盖了日常业务所涉及的几乎全部需求。

Layui.table有三种渲染方式，本项目中使用了了`方法渲染`，使用了`table.render`。

在渗透测试阶段挖掘到了XSS漏洞。经过从代码层分析，该漏洞的出现主要分为两点：

*   一是：后端没有对用户输入的字符进行转义或过滤。
    
*   二是：使用的前端框架存在XSS漏洞版本，也没有对返回到前端的参数进行过滤或转义。
    

### 5.5、SpringSecurity框架下未授权漏洞挖掘

本项目SprintSecurity配置代码位于`src/main/java/codermy/myspringsecurityplus/security/config/SpringSecurityConfig`。

SpringSecurity会配置一些放行的静态资源，有时错误的配置会将一些关键资源放行，造成未授权访问。这有可能是开发人员便于调试，有可能是粗心导致的。比如本项目中直接放行了`spring acatutor`监控页面，按理说这个页面需要登录才可以访问。不登录就可以未授权访问这个页面是存在很大风险的。如下图所示：

![springsecurity放行资源](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2Fspringsecurity%25E6%2594%25BE%25E8%25A1%258C%25E8%25B5%2584%25E6%25BA%2590.jpg)

SpringSecurity配置是链式结构，可以同时配置多项过滤。

### 5.6、图形验证码绕过

搜索关键字`captcha`等，找到相关代码。如下图所示：

![图形验证码搜索captcha](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E5%259B%25BE%25E5%25BD%25A2%25E9%25AA%258C%25E8%25AF%2581%25E7%25A0%2581%25E6%2590%259C%25E7%25B4%25A2captcha.jpg)

经搜索，图形验证码代码位于`src/main/java/com/codemy/mysringsecurityplus/security/filter/VerifyCoderFilter.java`

让我们分析代码逻辑。

首先，判断了请求方法和请求路径。两者都为True进入验证码逻辑部分。如下图所示：

![判断方法和路径](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E5%2588%25A4%25E6%2596%25AD%25E6%2596%25B9%25E6%25B3%2595%25E5%2592%258C%25E8%25B7%25AF%25E5%25BE%2584.jpg)

然后获取属性值。使用`request.getSession()`获取cookie的session值，使用`request.getParameter("captcha")`获取请求body中captcha参数值，使用`request.getSession().getAttribute("captcha")`获取cookie中captcha值。如下图所示：

![获取captcha值](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E8%258E%25B7%25E5%258F%2596captcha%25E5%2580%25BC.jpg)

接下来，仅判断了从请求参数中获取的`captcha`值是否为空，为空的话先删除缓存里的验证码信息，然后报错。如下图所示：

![判断captcha是否为空](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E5%2588%25A4%25E6%2596%25ADcaptcha%25E6%2598%25AF%25E5%2590%25A6%25E4%25B8%25BA%25E7%25A9%25BA.jpg)

仅判断从参数中获取的值是否为空逻辑是不严谨的。但在代码下方是有完整的验证逻辑。我们继续分析下。

判断从Cookie中获取captcha属性值是否为空。为空报错。

最后使用`StrUtil.equalsIgnoreCase()`函数忽略大小写判断，从cookie中获取的captcha值和从请求中获取的captcha值是否相同。如果不同的话就会报错。如下图所示：

![判断captcha两处值是否相同](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E5%2588%25A4%25E6%2596%25ADcaptcha%25E4%25B8%25A4%25E5%25A4%2584%25E5%2580%25BC%25E6%2598%25AF%25E5%2590%25A6%25E7%259B%25B8%25E5%2590%258C.jpg)

开发人员有时为了便于调试程序，可能会暂时关闭繁琐的验证逻辑。到了最后程序正式上线忘记了将注释删除掉，粗心的举动导致问题的发生。

当然，不同的程序会存在不同的问题。需要我们细心分析。

### 5.7、异常信息泄露

搜索关键字`e.printStackTrace()`，该函数会将异常调用信息。如下图所示：

![异常信息泄露](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fpower7089.github.io%2Fimg%2F%25E5%25BC%2582%25E5%25B8%25B8%25E4%25BF%25A1%25E6%2581%25AF%25E6%25B3%2584%25E9%259C%25B2.jpg)

### 5.8、Fortify自动化扫描

Fottify 全名叫：Fortify SCA ，是HP的产品 ，是一个静态的、白盒的软件源代码安全测试工具。它通过内置的五大主要分析引擎：数据流、语义、结构、控制流、配置流等对应用软件的源代码进行静态的分析，分析的过程中与它特有的软件安全漏洞规则集进行全面地匹配、查找，从而将源代码中存在的安全漏洞扫描出来，并给予整理报告。扫描的结果包含详细的安全漏洞信息、安全知识说明、修复意见。

1-
2-

    破解版地址：https://www.shungg.cn/301.html

## 结束语

本套练习基于真实项目进行了相关漏洞环境的改造。

1-

    https://github.com/witmy/my-springsecurity-plus

但本身原系统也存在一些安全风险问题。也就是说大家迈出了第一步，就挖到了一枚小小的0day了呢。

愿君在安全漫漫之路，能收获至尊0day。

从基础动手练习，完成最完美的蜕变。

[查看原网页: power7089.github.io](https://power7089.github.io/2022/08/22/JavaWeb%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1%E5%AE%9E%E6%88%98%E4%B9%8B%E6%9F%90RBAC%E7%B3%BB%E7%BB%9F%EF%BC%8C%E9%9D%9E%E5%B8%B8%E9%80%82%E5%90%88%E5%B0%8F%E7%99%BD%E5%85%A5%E9%97%A8%E7%BB%83%E4%B9%A0%E7%9A%84%E7%B3%BB%E7%BB%9F/)