### 5、Github Hacking

您可以在所有公共GitHub存储库中搜索以下类型的信息，以及您有权访问的所有私有Github存储库

    Repositories 
    Topics
    Issues and pull requests 
    Code 
    Commits 
    Users 
    Wikis 
    
        

参考 :

    Searching for repositories 
    Searching topics
    Searching code 
    Searching commits 
    Searching issues and pull requests 
    Searching users 
    Searching wikis 
    Searching in forks 
    
        

可以使用以上方式搜索页面或高级搜索页面搜索Github-
您可以使用>，>=，<，和<搜索是大于，大于或等于，小于和小于或等于另一个值的值-
下面会介绍如何搜索

**搜索仓库**

    
    >_n    		cats stars:>1000匹配关键字"cats"且star大于1000的仓库 
    
    >=_n_       cats topIcs:>=5匹配关键字"cats"且标签数量大于等于5的仓库 
    
    <_n_     	cats size:<10000匹配关键字"cats"且文件小于10KB的仓库 
    
    <=_n_    	cats stars:<=50匹配关键字"cats"且star小于等于50的仓库 
    
    _n_..*	 	cats stars:10..*匹配关键字"cats"且star大于等于10的仓库 
    
    *.._n_		cats stars:*..10匹配关键字"cats"且star小于等于10的仓库 
    
    n..n 		cats stars:10..50匹配关键字"cats"且star大于10且小于50的仓库
    
        

#### 搜索代码

**注意事项**

    只能搜索小于384KB的文件 
    只能搜索少于500,000个文件的存储库，登录的用户可以搜索所有公共存储库
    除filename搜索外，搜索源代码时必须至少包含一个搜索词。例如，搜索language: Javascript无效,而是这样: amazing language:Javascript 
    搜索结果最多可以显示来自同一文件的两个片段，但文件中可能会有更多结果。您不能将以下通配符用作搜索查询的一部分“.、! " = * ! ? # $ & + ^ | ~ <  > ( ) { } [ ] 搜索将忽略这些符号
    
        

**日期条件**

    cats pushed:<2012-07-05 搜索在2012年07月05日前push代码，且cats作为关键字
    cats pushed:2016-04-30..2016-07-04  日期区间
    cats created:>=2017-04-01  创建时间
    
        

**逻辑运算**

    AND、OR、NOT
    
        

**排除运算**

    cats pushed:<2012-07-05 language:java  搜索在2012年07月05日前push代码，且cats作为关键字，排除java语言仓库
    
        

**包含搜索**

    cats in:file			搜索文件中包含cats的代码
    cats in:path			搜索路径中包含cats的代码 
    cats in:path,file		搜索路径、文件中包含cats的代码
    console path:app/public language:javascript  搜索关键字 console，且语言为javascript，在app/public下的代码
    
        

**主体搜索**

    user: USERNAME								用户名搜索
    org: ''ORGNAME									组织搜索 
    repo: USERNAME/REPOSITORY		指定仓库搜索 
    
        

**文件大小**

    size:>1000		搜索大小大于1KB的文件 
    
        

#### 搜索案例

filename:config.php language:php 搜索文件名为 config.php，且语言为php的代码-
![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917112817153.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)

搜索Java项目配置文件: mail filename:.properties-
![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917112655292.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
搜索extension:yaml mongolab.com 中存在的代码信息等-
![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917112903829.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)

#### 自动化工具

    https://github.com/unkl4b/gitmIner
    
        

![在这里插入图片描述](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20200917113037733.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODAxNzQ1%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
example使用即可，非常好用

    https://github.com/techgaun/github-dorks  详细介绍github hacking 搜索利用代码以及方法！！
    
        

* * *