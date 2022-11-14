### 9、[内网](https://so.csdn.net/so/search?q=%E5%86%85%E7%BD%91&spm=1001.2101.3001.7020)渗透之信息收集

#### Windows（工作者和域）

**检查当前shell权限**

    whoami /user & whoami /priv
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917143444171.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
**查看系统信息**

    systeminfo
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917143704193.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
收集信息主机名->扮演角色

**Tcp/udp 网络连接状态信息**

    netstat -ano
    
        

可以获取内网IP分布状态-服务（redis)-
![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917143933693.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
**查看机器名**

    hostname
    
        

**查看当前操作系统**

    wmic OS get Caption,CSDVersion,OSArchitecture,Version 
    ver
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917144314500.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
**查杀软**

    WMIC	/Node:localhost /Namespace:\\root\SecurityCenter2 Path AntiVirusProduct Get displayName /Format:List
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917145142800.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
**查看当前安装的程序**

    wmic product get name,version
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2020091714553252.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
**查看在线用户**

    quser   							windwos7命令
    net config workstation				windwos10命令/查看当前域
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917150205981.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917150456897.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)

**查看网络配置**

    ipconfig /all
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917150955679.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
有 Primary Dns Suff就说明是域内空的则当前机器应该在工作组

**查看进程**

    tasklist /v
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917151315189.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
有些进程可能是域用户启的->通过管理员权限凭证窃取->窃取域用户的凭证

**查看当前登陆域**

    net config workstation
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917151834347.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
**远程桌面链接历史记录**

    cmdkey /l
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917152114892.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
可以把凭证取下来->本地密码

**查看本机上的用户账户列表**

    net user 
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917152301581.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
**查看本机用户xxx的信息**

    net user xxx
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917152451189.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
**查看本机用户xxx的信息**

    net user /domain 				显示所在域的用户名单
    net user 域用户 /domain			获取某个域用户的详细信息 
    net user /domain xxx 12345678 	修改域用户密码，需要域管理员权限
    
        

* * *

#### Windows（域）

    nltest /domain_trusts /all_trusts /v /server: 192.168.xx.xx    返回所有信任域列表
    nltest /dsgetdc:hack /server:192.168.xx.xx    				   返回域控和其相应的IP地
    net user /do 												   获取域用户列表
    net group "domain admins" /domain 							   获取域管理员列表
    net group "domain controllers" /domain						   查看域控制器(如果有多台)
    net group "domain computers" /domain						   查看域机器
    net group /domain 											   查询域里面的工作组
    net localgroup administrators								   本机管理员[通常含有域用户]
    net localgroup administrators /domain 						   登录本机的域管理员 
    net localgroup administrators workgroup\user001 /add 		   域用户添加到本机
    
        

     
    
    net view 				查看同一域内机器列表 
    net view \\ip			查看某IP共享
    net view \\GHQ		    查看GHQ计算机的共享资源列表 
    net view /domain		查看内网存在多少个域 
    net view /domain:XYZ	查看XYZ域中的机器列表 
    net accounts /domain	查询域用户密码过期等信息
    
        

#### Linux

**查看当前权限**

    whoami
    
        

**查看网卡配置**

    ifconfig
    
        

**查看端口状态（开启了哪些服务，内网IP连接等**

    netstat -anpt
    
        

**查看进程状态（开启了哪些服务等）**

    ps -ef
    
        

**查看管理员的历史输入命令（获取密码，网站目录，内网资产等信息）**

    cat /root/.bash_history
    
        

**查找某个文件（寻找配置文件等）**

    find / -name *.cfg
    
        

* * *