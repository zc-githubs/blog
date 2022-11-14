### 6、google hacking

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917114133402.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
**用法**

    Intitle 			包含标题 
    Intext  			包含内容 
    filetype 			文件类型 
    Info 				基本信息 
    site 				指定网站 
    inurl 				包含某个url
    link 				包含指定链接的网页 
    cache 				显示页面的缓存版本
    numberange			搜索一个数字
    
        

**示例**-
搜索目标包含后台的页面-
![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917114436211.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
命令：`inurl:/admin intext: 后台管理系统`

    site:"some-keywords.com"intitle: login intext: intext: 管理|后台|登陆|用户名|密码|验证码|系统|帐号| manage|admin|login|system
    
        

搜索目标是否有目录列表-
![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917115125325.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917115229301.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
可看到存在目录列表很多url-
命令：`intext: index of / | ../ | Parent Directory`

    site:"some-keywords.com" intext: index of / | ../ | Parent Directory
    
        

* * *