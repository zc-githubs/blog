---
date: 2022-06-30-星期四 09:31:45
update: 
tags: [time/year2022,time/month06]
id![[未命名 2022-06-30 09.31.39.excalidraw]] 20220630093145
---

# Hydra密码爆破工具命令大全

Hydra是一款非常专业的密码爆破工具，它一般工作在Linux环境下面，不过在Windows版本早就已经有了，这款密码爆破工具的功能可谓是非常强大，支持的协议非常之多，比如常见的[rdp](https://so.csdn.net/so/search?q=rdp&spm=1001.2101.3001.7020)、ssh、FTP、smb等，甚至连普通web表单也是可以直接拿来爆破的。这款工具的安装非常简单就不多说了，下面就介绍一下Hydra的一些常用密码爆破命令。

* * *


命令语法
\# hydra \[\[\[-l LOGIN|-L FILE\] \[-p PASS|-P FILE\]\] | \[-C FILE\]\] \[-e ns\]-
\[-o FILE\] \[-t TASKS\] \[-M FILE \[-T TASKS\]\] \[-w TIME\] \[-f\] \[-s PORT\] \[-S\] \[-vV\] server service \[OPT\]-
参数介绍:-
\-R 继续从上一次进度接着破解-
\-S 大写，采用SSL链接-
\-s<PORT> 小写，可通过这个参数指定非默认端口-
\-l<LOGIN> 指定破解的用户，对特定用户破解-
\-L<FILE> 指定用户名字典-
\-p<PASS> 小写，指定密码破解，少用，一般是采用密码字典-
\-P<FILE> 大写，指定密码字典-
\-e<ns> 可选选项，n：空密码试探，s：使用指定用户和密码试探-
\-C<FILE> 使用冒号分割格式，例如“登录名:密码”来代替-L/-P参数-
\-M<FILE> 指定目标列表文件一行一条-
\-o<FILE> 指定结果输出文件-
\-f 在使用-M参数以后，找到第一对登录名或者密码的时候中止破解-
\-t<TASKS> 同时运行的线程数，默认为16-
\-w<TIME> 设置最大超时的时间，单位秒，默认是30s-
\-v /-V 显示详细过程




# 各协议爆破命令实例
## 1、爆破SSH
hydra -l 用户名 -p 密码字典 -t 线程 -vV -e ns ip ssh-
hydra -l 用户名 -p 密码字典 -t 线程 -o save.log -vV ip ssh
## 2、爆破FTP
hydra ip ftp -l 用户名 -P 密码字典 -t 线程(默认16) -vV-
hydra ip ftp -l 用户名 -P 密码字典 -e ns -vV
## 3、爆破SMB-
hydra -l administrator -P pass.txt 192.168.0.2 smb


## 4、爆破rdp
hydra ip rdp -l administrator -P pass.txt -V


## 5、爆破POP3-
hydra -l muts -P pass.txt my.pop3.mail pop3


## 6、爆破http-proxy-
hydra -l admin -P pass.txt http-proxy://192.168.0.2


## 7、爆破web表单
hydra -l 用户名 -P 密码字典 -s 80 域名 http-post-form"/admin/login.php:username=^USER^&password=^PASS^:<密码错误的时候的报错>"

-
参数解释：username是form表单的账号name;password是密码字段的name；USER和PASS是固定的，代表账号和密码。冒号后面紧跟着登录失败的时的提示信息，用来判别爆破是否成功；http-post-form代表是以post方式提交数据，如果是GET方式直接将post修改为get即可；/admin/login代表表单提交的地址使用的是绝对路径。


结束语-
密码爆破的关键点其实还在于强大的字典，只要字典够强大，爆破成功其实就是时间+运气了。


# Quote
