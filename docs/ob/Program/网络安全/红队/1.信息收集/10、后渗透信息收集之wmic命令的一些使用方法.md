### 10、后渗透信息收集之wmic命令的一些使用方法

**前言**

wmic和cmd一样在所有的windows版本中都存在，同时wmic有很多cmd下不方便使用的部分，今天给大家介绍一些在后渗透过程中非常适用的使用wmic进行信息收集的命令

**关于wmic**

WMI命令行（WMIC）实用程序为WMI提供了命令行界面。WMIC与现有的Shell和实用程序命令兼容。在WMIC出现之前，如果要管理WMI系统，必须使用一些专门的WMI应用，例如SMS，或者使用WMI的脚本编程API，或者使用象CIM Studio之类的工具。如果不熟悉C++之类的编程语言或VBScript之类的脚本语言，或者不掌握WMI名称空间的基本知识，要用WMI管理系统是很困难的，WMIC改变了这种情况

#### wmic的简单使用

首先在cmd命令行输入wmic进入交互式页面，这里说一下在powershell也可以和cmd命令行一样的操作-
![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917162035110.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
进入wmic和powershell模式下-
![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917162550726.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)

    /?				查看WMIC命令的全局选项以及命令属性等
    process /?		进程管理的帮助
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917163130617.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)

    wmic process get /?   属性获取操作帮助
    
        

根据实际的需要去对相关的信息进行读取

* * *

#### 以进行为例展现wmic的使用

这里的靶机是win7 x86的虚拟机，这里以查看进程为例：

    wmic process get caption,executablepath,processid
    
        

获取系统当前正在运行的进程等信息-
![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2020091716373167.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)

    wmic service where (state="running") get name ,processid ,pathname ,startmode ,caption
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917164158711.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
查看服务进程详细信息

    wmic /namespace:\\root\securitycenter2 path antivirusproduct GET displayName,productState, pathToSignedProductExe
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917164630317.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
查看安装的杀软进程运行情况

    wmic onboarddevice get Description, DeviceType, Enabled, Status /format:list
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2020091716513311.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
查看存在状态

    wmic product get name
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917165405423.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)

系统安装软件情况

    wmic environment get Description, VariableValue
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917165447294.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
系统环境变量

    wmic computersystem get Name, Domain, Manufacturer, Model, Username, Roles/format:list
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917165708165.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)

    wmic sysdriver get Caption, Name, PathName, ServiceType, State, Status /format:list
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917165910670.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)

关于更多的信息可以通过官方的说明文档

    https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/wmic
    
        

* * *

#### 关于powershell的Get-Wmi对象

Get-Wmi是获取Windows Management Instrumentation（WMI）类的实例或有关可用类的信息。我们需要首先知道自己的 windows计算机支持那些可用的WMI类

    Get-Wmiobject -list 	自己的windows计算机支持那些可用的WMI类
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917195023569.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)

    get-wmiobject
    get-wmiobject -class win32_process
    
        

在本地计算机上获取进程

具体的参数以及命令在官方文档中进行查询：

    https://docs.microsoft.com/zh-cn/powershell/module/Microsoft.PowerShell.Management/Get-WmiObject?view=powershell-5.l#parameters
    
        

很棒的powershell官方命令