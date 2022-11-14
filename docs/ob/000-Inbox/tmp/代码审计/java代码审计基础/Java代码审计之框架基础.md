

# Java三层架构及MVC模式

java 三层架构：数据访问层、业务逻辑层、表现层-
**业务层**\-----一般不变的，主要是一些算法逻辑，用了策略模式，用了 反射技术使得它的变化相对稳定。（规则制定）~业务 （Business）或叫商务-
**持久层**\-----存储数据的，存储数据可能会由xml配置文件更改为数据库.-
**视图层**\------显示界面的，显示界面可能有c/s 更改为 b/s.

一、 **持久层（Data Access Layer DAL 数据访问层）采用DAO模式**。 建立实体类和数据库表映射（ORM映射）。也就是哪个类对应 哪个表，哪个属性对应哪个列。持久层的目的就是，完成对象数据 和关系数据的转换。-
二、**业务层（Business Logic Layer BLL 逻辑层、service层）采 用事务脚本模式**。 将一个业务中所有的操作封装成一个方法，同时保证方法中所 有的数据库更新操作，即保证同时成功或同时失败。避免部分成功 部分失败引起的数据混乱操作。-
三、**表现层（UI层、视图层、界面层）采用MVC（Model-ViewControler）模式,采用JSP/Servlet 技术进行页面效果显示。**

![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220916%2F1663308157_6324117d0f10829cf19af.png%21small)

SSH框架-
业务层——Spring-
表现层——Struts-
持久层——Hibernate

SSM框架-
业务层——Spring-
表现层——SpringMVC-
持久层——MyBatis

**MVC模式**（Model-View-Controler）

经常会有人把三层架构和MVC模式搞混，这里说明一下，**两者没有实质上得关系，可以共存**，两者是通过不同维度来说明 Javaweb得结构，三层架构是一种分层思想，利用这个思想可以将 web开发人员分为前端开发，后端开发、DBA这三种职位，而MVC 模式主要是根据数据流来讲解得web结构，我们后续学习中也主要 是根据MVC模式来进行学习。

M称为模型，也就是实体类。用于数据的封装和数据的传输。-
V为视图，也就是GUI组件，用于数据的展示。-
C为控制，也就是事件，用于流程的控制。-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220916%2F1663308186_6324119a1f2e20e342ddc.png%21small)首先，我们打开浏览器，输入网址，就是到服务器中请求页面 (JSP也可能是别的)，然后显示到浏览器上，然后通过点击JSP页面上 的内容，提交请求，到服务器中，也就到了Control(Servlet)这一 块，Servlet通过分析请求，知道用户需要什么，需要数据，那么就 通过Model，从数据库拿到数据，在将数据显示在JSP中，在将JSP 发送回浏览器，显示在用户看，所以我们经常说，JSP就是View层， 给用户看的，Servlet作为控制流程，而编写操作数据库代码，业务 逻辑代码就属于Model。这就是MVC的应用-
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220916%2F1663308210_632411b2778fbcb69b289.png%21small)

# Maven详解

Maven 是一个项目管理工具，它包含了一个项目对象模型 （Project Object Model），反映在配置中，就是一个 pom.xml 文 件。是一组标准集合，一个项目的生命周期、一个依赖管理系统， 另外还包括定义在项目生命周期阶段的插件(plugin)以及目标 (goal)。-
当我们使用 Maven 的使用，通过一个自定义的项目对象模型， pom.xml 来详细描述我们自己的项目。-
简单来说，我们开发一个JavaWeb项目是需要加载很多依赖的，使 用Maven可以便于管理这些依赖。

## pom.xml

POM是项目对象模型(Project Object Model)的简称,它是Maven项 目中的文件，使用XML表示，名称叫做pom.xml。该文件用于管 理：源代码、配置文件、开发者的信息和角色、问题追踪系统、组 织信息、项目授权、项目的url、项目的依赖关系等等。Maven项目 中必须包含pom.xml文件。-
以下是 helloMaven 项目的坐标定义。

    <project>    
    <groupId>net.biancheng.www</groupId>    
    <artifactId>helloMaven</artifactId>    
    <packaging>jar</packaging>    
    <version>1.0-SNAPSHOT</version>
    </project>
    

## Maven 坐标

主要由以下元素组成：-
groupId： 项目组 ID，定义当前 Maven 项目隶属的组织或公 司，通常是唯一的。它的取值一般是项目所属公司或组织的网址 或 URL 的反写，例如 net.biancheng.www-
artifactId： 项目 ID，通常是项目的名称。-
version：版本-
packaging：项目的打包方式，默认值为 jar

## 仓库分类

Maven 仓库可以分为 2 个大类： 本地仓库 远程仓库-
当通过 Maven 构建项目时，Maven 按照如下顺序查找依赖的构件。

1.  从本地仓库查找构件，如果没有找到，跳到第 2 步，否则继续执行其他处理。
    
2.  从中央仓库查找构件，如果没有找到，并且已经设置其他远程仓库，然后移动到第 4 步；如果找到，那么将构件下载到本地仓库中使用。
    
3.  如果没有设置其他远程仓库，Maven 则会停止处理并抛出错误。
    
4.  在远程仓库查找构件，如果找到，则会下载到本地仓库并使用，否则 Maven 停止处理并抛出错误。
    

# 耦合度介绍

一、耦合-
耦合是两个或多个模块之间的相互关联。在软件工程中，两个模块之间的耦合度越高，维护成本越高。因此，在系统架构的设计过程 中，应减少各个模块之间的耦合度，以提高应用的可维护性。-
二、紧耦合-
紧耦合架构本质是Client/Server的模型-
优点是：架构简单、设计简单、开发周期短、能够快速的开发、投 入、部署、应用。-
但随着集群规模的扩大，系统的稳定性逐渐变差，主要原因如下：-
1、同步操作导致对网络资源消耗大。同步操作在数据发送和数据返 回之间，有很大一段是空闲的，这种空闲占用是对网络资源的极大 浪费。-
2、安全控制力度差，因为服务器直接暴露给客户机，容易引发网络 攻击行为。-
3、程序代码之间关联度过高，不利于模块化处理。-
三、松耦合-
松耦合架构本质上是在client/server模型之间加入一个代理，把CS 模型变成CAS模型。-
在新的架构下，客户机的角色不变，代理服务器承担起与客户机的 通信，和对客户机的识别判断工作，服务器位于代理服务器后面， 对客户机来说不可见，它只负责数据处理工作，另外我们也把CS模 型的同步操作改为CAS的代理处理。-
优点如下：-
1、多任务并行处理能力获得极大提升。-
2、实现负载自适应机制（根据当时运行环境，松耦合架构分配并行 工作任务，避免超载现象）。-
3、基本杜绝了对Server服务端的网络攻击行为，由于代理服务器的 隔绝和筛查作用， 同时结合其它安全管理 手段，外部攻击在代理服 务器处就被识别和过滤掉了，这样就保护了后面的服务器不受影响-
4、异步操作减少了网络资源消耗和操作关联。-
5、提高了系统的可维护性。

# Spring讲解

参考：S[pringIoC（控制反转）(biancheng.net)](http://c.biancheng.net/spring/inversion-control.html)

## Spring是什么

Spring 是 Java EE 编程领域的一款轻量级的开源框架

### 广义的 Spring

Spring 技术栈-
广义上的 Spring泛指以 SpringFramework为核心的 Spring技术栈。

经过十多年的发展，Spring 已经不再是一个单纯的应用框架，而是逐渐发展成为一个由多个不同子项目（模块）组成的成熟技术，例如 SpringFramework、SpringMVC、SpringBoot、Spring-
Cloud、Spring Data、Spring Security 等，其中 SpringFramework 是其他子项目的基础。

**项目名称**

**描述**

Spring Data

Spring提供的数据访问模块，对 JDBC和 ORM提供了很好的支持。通过它，开发人员可以使用一种相对统一的方式，来访问位于不同类型数据库中的数据。

Spring Batch

一款专门针对企业级系统中的日常批处理任务的轻量级框架，能够帮助开发人员方便的开发出健壮、高效的批处理应用程序。

Spring Security

前身为 Acegi，是 Spring 中较成熟的子模块之一。它是一款可以定制化的身份验证和访问控制框架。

Spring Mobile

是对 Spring MVC 的扩展，用来简化移动端 Web 应用的开发。

Spring Boot

是 Spring 团队提供的全新框架，它为 Spring 以及第三方库一些开箱即用的配置，可以简化 Spring应用的搭建及开发过程。

|

Spring Cloud | 一款基于 Spring Boot 实现的微服务框架。它并不是某一门技术，而是一系列微服务解决方案或框架的有序集合。它将市面上成熟的、经过验证的微服务框架整合起来，并通过 Spring Boot 的思想进行再封装， 屏蔽调其中复杂的配置和实现原理，最终为开发人员提供了一套简单易懂、易部署和易维护的分布式系统开发工具包。 |

### 狭义的 Spring

Spring Framework-
Spring 框架是一个分层的、面向切面的 Java 应用程序的一站式轻量级解决方案，它是 Spring技术栈的核心和基础，是为了解决企业级应用开发的复杂性而创建的。-
Spring 有两个核心部分： IoC 和 AOP。

**核心**

**描述**

IOC

InverseofControl的简写，译为“控制反转”，指把创建对象过程交给 Spring进行管理。

|

AOP | Aspect Oriented Programming 的简写，译为“面向切面编程”。 AOP 用来封装多个类的公共行为，将那些与业务无关，却为业务模块所共同调用的逻辑封装起来，减少系统的重复代码，降低模块间的耦合度。另外，AOP 还解决一些系统层面上的问题，比如日志、事务、权限等。 |

在实际开发中，服务器端应用程序通常采用三层体系架构，分别为表现层（web）、业务逻辑层（service）、持久层（dao）

## Spring体系

Spring 致力于 Java EE 应用各层的解决方案，对每一层都提供了技术支持。-
在表现层提供了对 Spring MVC、Struts2 等框架的整合；-
在业务逻辑层提供了管理事务和记录日志的功能；-
在持久层还可以整合 MyBatis、Hibernate 和 JdbcTemplate 等技术，对数据库进行访问。-
Spring 框架基本涵盖了企业级应用开发的各个方面，它包含了 20多个不同的模块。

    spring-aop     spring-context-indexer springinstrument spring-orm   spring-web
    spring-aspects spring-context-support spring-jcl  
          spring-oxm   spring-webflux
    spring-beans   spring-core             spring-jdbc
          spring-r2dbc spring-webmvc
    spring-context spring-expression       spring-jms  
          spring-test   spring-websocket
    spring-messaging   spring-tx  
    

# SpringMVC讲解

Spring MVC 是 Spring 提供的一个基于 MVC 设计模式的轻量级Web 开发框架，本质上相当于 Servlet。-
SpringMVC是结构最清晰的 Servlet+JSP+JavaBean的实现，是一个典型的教科书式的 MVC构架，不像 Struts等其它框架都是变种或者不是完全基于 MVC系统的框架。

## Spring MVC配置

SpringMVC是基于 Servlet的，DispatcherServlet是整个 Spring MVC 框架的核心，主要负责截获请求并将其分派给相应的处理器处理。-
所以配置 Spring MVC，首先要定义 DispatcherServlet。跟所有 Servlet 一样，用户必须在 **web.xml 中**进行配置。

### 1.定义DispatcherServlet

在开发 Spring MVC 应用时需要在 web.xml 中部署-
DispatcherServlet，代码如下：

    <?xml version="1.0" encoding="UTF-8"?>
    <web-app
    xmlns:xsi="http://www.w3.org/2001/XMLSchemainstance"
        xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:web="http://java.sun.com/xml/ns/javaee/webapp_2_5.xsd"
      
    xsi:schemaLocation="http://java.sun.com/xml/ns/java
    ee http://java.sun.com/xml/ns/javaee/webapp_3_0.xsd"
        version="3.0">
        <display-name>springMVC</display-name>
        <!-- 部署 DispatcherServlet -->
        <servlet>
            <servlet-name>springmvc</servlet-name>
            <servletclass>org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
            <!-- 表示容器再启动时立即加载servlet -->
            <load-on-startup>1</load-on-startup>
        </servlet>
        <servlet-mapping>
            <servlet-name>springmvc</servlet-name>
            <!-- 处理所有URL -->
            <url-pattern>/</url-pattern>
        </servlet-mapping>
    </web-app>
    

Spring MVC 初始化时将在应用程序的 WEB-INF 目录下查找配置文件，该配置文件的命名规则是“servletName-servlet.xml”，例如springmvc-servlet.xml。-
也可以将 Spring MVC 的配置文件存放在应用程序目录中的任何地方，但需要使用 servlet 的 init-param元素加载配置文件，通过contextConfigLocation 参数来指定 Spring MVC 配置文件的位置， 示例代码如下

    <!-- 部署 DispatcherServlet -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servletclass>org.springframework.web.servlet.DispatcherServ
    let </servlet-class>
        <init-param>
            <param-name>contextConfigLocation</paramname>
            <param-value>classpath:springmvcservlet.xml</param-value>
    
        </init-param>
        <!-- 表示容器再启动时立即加载servlet -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
    

此处使用 Spring 资源路径的方式进行指定，即 classpath:springmvc-servlet.xml。-
上述代码配置了一个名为“springmvc”的 Servlet。该 Servlet 是 DispatcherServlet 类型，它就是 Spring MVC 的入口，并通过 1配置标记容器在启动时 就加载此 DispatcherServlet，即自动启动。然后通过 servletmapping 映射到“/”，即 DispatcherServlet 需要截获并处理该项目 的所有 URL 请求。

### 2.创建Spring MVC配置文件

在 WEB-INF 目录下创建 springmvc-servlet.xml 文件，如下所示。

    <?xml version="1.0" encoding="UTF-8"?>
    <beans
    xmlns="http://www.springframework.org/schema/beans"
           xmlns:xsi="http://www.w3.org/2001/XMLSchemainstance"
           xsi:schemaLocation="
            http://www.springframework.org/schema/beans
          
    http://www.springframework.org/schema/beans/springbeans.xsd">
        <!-- LoginController控制器类，映射到"/login" -->
        <bean name="/login"
            
    class="net.biancheng.controller.LoginController"/>
        <!-- LoginController控制器类，映射到"/register" -->
        <bean name="/register"
            class="net.biancheng.controller.RegisterController"/>
    </beans>
    

### 3\. 创建Controller

在 src 目录下创建 net.biancheng.controller 包，并在该包中创建 RegisterController 和 LoginController 两个传统风格的控制器类 （实现 Controller 接口），分别处理首页中“注册”和“登录”超链接的 请求。

> Controller 是控制器接口，接口中只有一个方法-
> handleRequest，用于处理请求和返回 ModelAndView

RegisterController 的具体代码如下

    package controller;
    import javax.servlet.http.HttpServletRequest;
    import javax.servlet.http.HttpServletResponse;
    import org.springframework.web.servlet.ModelAndView;
    import
    org.springframework.web.servlet.mvc.Controller;
    public class RegisterController implements
    Controller {
        public ModelAndView handleRequest(HttpServletRequest arg0,HttpServletResponsearg1) throws Exception {
            return new ModelAndView("/WEBINF/jsp/login.jsp");
       }
    }
    

### 4\. 创建View index.jsp

代码如下

    <%@ page language="java" contentType="text/html;charset=UTF-8" pageEncoding="UTF-8"%>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
    "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html;
    charset=UTF-8">
    <title>Insert title here</title>
    </head>
    <body>
       未注册的用户，请
        <a href="${pageContext.request.contextPath}/register"> 注册</a>！
        
     已注册的用户，去
        <a href="${pageContext.request.contextPath}/login"> 登录</a>！
    </body>
    </html>
    

在 WEB-INF 下创建 jsp 文件夹，将 login.jsp 和 register.jsp 放到 jsp 文件夹下。-
login.jsp 代码如下

    <%@ page language="java" contentType="text/html;charset=UTF-8"pageEncoding="UTF-8"%>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
    "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
    <meta http-equiv="Content-Type" content="text/html;
    charset=UTF-8">
    <title>Insert title here</title>
    </head>
    <body>
       登录页面！
    </body>
    </html>
    

register.jsp 代码如下

    <%@ page language="java" contentType="text/html; charset=UTF-8"pageEncoding="UTF-8" %>
    <!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
    <html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <title>Insert title here</title>
    <body>
       注册页面！
    </body>
    </html>
    </head>
    

将 springmvcDemo 项目部署到 Tomcat 服务器，首先访问 index.jsp 页面，如下图所示

# Controller及RequestMapping注解

## Controller注解

@Controller 注解用于声明某类的实例是一个控制器。例如，在 net.biancheng.controller 包中创建控制器类 IndexController，示例代码如下

    package net.biancheng.controller;
    import org.springframework.stereotype.Controller;
    @Controller
    public class IndexController {
        // 处理请求的方法
    }
    

SpringMVC使用扫描机制找到应用中所有基于注解的控制器类，所以，为了让控制器类被 SpringMVC框架扫描到，需要在配置文件中声明 spring-context，并使用 context:component-scan/元素指定控制器类的基本包（请确保所有控制器类都在基本包及其子包下）。-
例如，在 springmvcDemo 应用的配置文件 springmvc- servlet.xml 中添加以下代码：

    <!-- 使用扫描机制扫描控制器类，控制器类都在
    net.biancheng.controller包及其子包下 -->
    <context:component-scan basepackage="net.biancheng.controller" />
    

## RequestMapping注解

一个控制器内有多个处理请求的方法，如 UserController 里通常有增加用户、修改用户信息、删除指定用户、根据条件获取用户列表等。每个方法负责不同的请求操作，而 @RequestMapping 就负责将请求映射到对应的控制器方法上。-
在基于注解的控制器类中可以为每个请求编写对应的处理方法。使用 @RequestMapping注解将请求与处理方法一 一对应即可。-
@RequestMapping注解可用于类或方法上。用于类上，表示类中所有响应请求方法都以该地址作为父路径-
@RequestMapping 注解常用属性如下

### 1\. value 属性

value属性是 @RequestMapping注解的默认属性，因此如果只有value属性时，可以省略该属性名，如果有其它属性，则必须写上value属性名称。如下

    @RequestMapping(value="toUser")
    @RequestMapping("toUser")
    

value 属性支持通配符匹配，如 @RequestMapping(value="toUser/\*") 表示 http://localhost:8080/toUser/1 或 http://localhost:8080/toUser/hahaha 都能够正常 访问

### 2\. path属性

path 属性和 value 属性都用来作为映射使用。即 @RequestMapping(value="toUser") 和 @RequestMapping(path="toUser") 都能访问 toUser() 方法。-
path 属性支持通配符匹配，如 @RequestMapping(path="toUser/\*")-
表示 http://localhost:808 0/toUser/1 或 http://localhost:8080/toUser/hahaha 都能够正常 访问

### 3\. name属性

name属性相当于方法的注释，使方法更易理解。-
如 @RequestMapping(value = "toUser",name = "获取用户信息")

### 4\. method属性

method 属性用于表示该方法支持哪些 HTTP 请求。如果省略 method 属性，则说明该方法支持全部的 HTTP 请求。-
@RequestMapping(value = "toUser",method = RequestMethod.GET) 表示该方法只支持 GET 请求。也可指定多 个 HTTP 请求，如 @RequestMapping(value = "toUser",method = {RequestMethod.GET,RequestMethod.POST})，说明该方法同时 支持 GET 和 POST 请求。

### 5\. params属性

params 属性用于指定请求中规定的参数，代码如下

    @RequestMapping(value = "toUser",params ="type")
    public String toUser() {        
        return "showUser";
    }
    

以上代码表示请求中必须包含 type 参数时才能执行该请求。-
即 htt p://localhost:8080/toUser?type=xxx 能够正常访问 toUser() 方法，-
而 http://localhost:8080/toUser 则不能正常访问 toUser() 方法。

    @RequestMapping(value = "toUser",params ="type=1")
    public String toUser() {     
        return "showUser";
    }
    

以上代码表示请求中必须包含 type 参数，且 type 参数为 1 时才能 够执行该请求。-
即 http://localhost:8080/toUser?type=1 能够正常 访问 toUser() 方法，-
而 http://localhost:8080/toUser?type=2 则 不能正常访问 toUser() 方法。

### 6\. header属性

header 属性表示请求中必须包含某些指定的 header 值。-
@RequestMapping(value = "toUser",headers = "Referer=http:// www.xxx.com") 表示请求的 header 中必须包含了指定的“Referer” 请求头，以及值为“http://www.xxx.com”时，才能执行该请求

### 7\. consumers属性

consumers 属性用于指定处理请求的提交内容类型（ContentType），-
例如：application/json、text/html。如 @RequestMapping(value = "toUser",consumes = "application/json")。

### 8\. produces属性

produces 属性用于指定返回的内容类型，返回的内容类型必须是 request 请求头（Accept）中所包含的类型。如 @RequestMapping(value = "toUser",produces = "application/json")。-
除此之外，produces 属性还可以指定返回值的编码。-
如 @RequestMapping(value = "toUser",produces = "application/json,charset=utf-8")，表示返回 utf-8 编码。-
使用 @RequestMapping 来完成映射，具体包括 4 个方面的信息 项：请求 URL、请求参数、请求方法和请求头。

## 重定向和转发

Spring MVC 请求方式分为转发、重定向 2 种，分别使用 forward 和 redirect 关键字在 controller 层进行处理。-
在 Spring MVC 框架中，控制器类中处理方法的 return 语句默认就 是转发实现，只不过实现的是转发到视图。示例代码如下：

    @RequestMapping("/register")
    public String register() {
        return "register";  //转发到register.jsp
    }
    

在 Spring MVC 框架中，重定向与转发的示例代码如下：

    package net.biancheng.controller;
    import org.springframework.stereotype.Controller;
    import
    org.springframework.web.bind.annotation.RequestMappi
    ng;
    @Controller
    @RequestMapping("/index")
    public class IndexController {
        @RequestMapping("/login")
        public String login() {
            //转发到一个请求方法（同一个控制器类可以省略/index/）
            return "forward:/index/isLogin";
       }
        @RequestMapping("/isLogin")
        public String isLogin() {
            //重定向到一个请求方法
            return "redirect:/index/isRegister";
       }
        @RequestMapping("/isRegister")
        public String isRegister() {
            //转发到一个视图
            return "register";
       }
    }
    

## 拦截器

在 Spring MVC 框架中定义一个拦截器需要对拦截器进行定义和配 置，主要有以下 2 种方式。

1.  通过实现 HandlerInterceptor 接口或继承 HandlerInterceptor 接口的实现类（例如 HandlerInterceptorAdapter）来定义；
    
2.  通过实现 WebRequestInterceptor 接口或继承 WebRequestInterceptor 接口的实现类来定义。-
    \*\* 配置: \*\*
    

    <!-- 配置拦截器 -->
    <mvc:interceptors>
        <!-- 配置一个全局拦截器，拦截所有请求 -->
        <bean
    class="net.biancheng.interceptor.TestInterceptor" />
        <mvc:interceptor>
            <!-- 配置拦截器作用的路径 -->
            <mvc:mapping path="/**" />
            <!-- 配置不需要拦截作用的路径 -->
            <mvc:exclude-mapping path="" />
            <!-- 定义<mvc:interceptor>元素中，表示匹配指定路径的请求才进行拦截 -->
            <bean
    class="net.biancheng.interceptor.Interceptor1" />
        </mvc:interceptor>
        <mvc:interceptor>
            <!-- 配置拦截器作用的路径 -->
            <mvc:mapping path="/gotoTest" />
            <!-- 定义在<mvc:interceptor>元素中，表示匹配指定路径的请求才进行拦截 -->
            <bean
    class="net.biancheng.interceptor.Interceptor2" />
        </mvc:interceptor>
    </mvc:interceptors>
    

在上述示例代码中，元素说明如下。-
mvc:interceptors：该元素用于配置一组拦截器。-
该元素是 mvc:interceptors 的子元素，用于定义全局拦截 器，即拦截所有的请求。-
mvc:interceptor：该元素用于定义指定路径的拦截器。-
mvc:mapping：该元素是 mvc:interceptor 的子元素，用于配 置拦截器作用的路径，该路径在其属性 path 中定义。path 的属 性值为/\*\*时，表示拦截所有路径，值为/gotoTest时，表示拦 截所有以/gotoTest结尾的路径。如果在请求路径中包含不需 要拦截的内容，可以通过 mvc:exclude-mapping 子元素进行配 置。

## 文件上传

Spring MVC 框架的文件上传基于 commons-fileupload 组件，并 在该组件上做了进一步的封装，简化了文件上传的代码实现，取消 了不同上传组件上的编程差异-
在 Spring MVC 中实现文件上传十分容易，它为文件上传提供了直 接支持，即 MultpartiResolver 接口。MultipartResolver 用于处理 上传请求，将上传请求包装成可以直接获取文件的数据，从而方便操作。

MultpartiResolver 接口有以下两个实现类：-
StandardServletMultipartResolver：使用了 Servlet 3.0 标准的 上传方式。-
CommonsMultipartResolver：使用了 Apache 的 commonsfileupload 来完成具体的上传操作

Maven 项目在 pom.xml 文件中添加以下依赖。

    纯文本复制
    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.4</version>
    </dependency>
    <dependency>
        <groupId>commons-fileupload</groupId>
        <artifactId>commons-fileupload</artifactId>
        <version>1.2.2</version>
    </dependency>
    

## 文件下载

Spring MVC 文件下载的实现方法和实现过程；文件下载有以下两种 实现方法：-
通过超链接实现下载：实现简单，但暴露了下载文件的真实位 置，并且只能下载 Web 应用程序所在目录下的文件，WEB-INF 目录除外。-
利用程序编码实现下载：增强安全访问控制，可以下载除 Web 应用程序所在目录以外的文件，也可以将文件保存到数据库中。-
利用程序编码实现下载需要设置以下两个报头：

1.  Web 服务器需要告诉浏览器其所输出内容的类型不是普通文本 文件或 HTML 文件，而是一个要保存到本地的下载文件，这需 要设置 Content-Type 的值为 application/x-msdownload。
    
2.  Web 服务器希望浏览器不直接处理相应的实体内容，而是由用 户选择将相应的实体内容保存到一个文件中，这需要设置 Content-Disposition 报头
    

该报头指定了接收程序处理数据内容的方式，在 HTTP 应用中只有 attachment 是标准方式，attachment 表示要求用户干预。在 attachment 后面还可以指定 filename 参数，该参数是服务器建议 浏览器将实体内容保存到文件中的文件名称。

设置报头的示例如下：

    response.setHeader("Content-Type", "application/xmsdownload");
     response.setHeader("Content-Disposition",
    "attachment;filename="+filename);
    

    /**
         * 执行下载
         */
        @RequestMapping("down")
        public String down(@RequestParam String
    filename,
                HttpServletRequest request,
    HttpServletResponse response) {
            String aFilePath = null; // 要下载的文件路径
            FileInputStream in = null; // 输入流
            ServletOutputStream out = null; // 输出流
            try {
                // 从
    workspace\.metadata\.plugins\org.eclipse.wst.server.
    core\
                // tmp0\wtpwebapps下载
                aFilePath =request.getServletContext().getRealPath("uploadfiles");
                // 设置下载文件使用的报头
                response.setHeader("Content-Type","application/x-msdownload");
                response.setHeader("ContentDisposition", "attachment; filename="
                        + toUTF8String(filename));
                // 读入文件
                in = new FileInputStream(aFilePath +"\\" + filename);
                // 得到响应对象的输出流，用于向客户端输出二进制数据
                out = response.getOutputStream();
                out.flush();
                int aRead = 0;
                byte b[] = new byte[1024];
                while ((aRead = in.read(b)) != -1 & in!= null) {
                    out.write(b, 0, aRead);
               }
                out.flush();
                in.close();
                out.close();
           } catch (Throwable e) {
                e.printStackTrace();
           }
            logger.info("下载成功");
            return null;
       }