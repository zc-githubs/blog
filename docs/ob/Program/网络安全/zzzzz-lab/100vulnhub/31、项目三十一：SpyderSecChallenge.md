
# mynew
图片 视频数据枚举分析
hex十六进制  =》ascia base64 
eval(function(p,a,c,k,e,d)

# 

VMware双击打开nat模式即可！

1、nmap 192.168.253.0/24 -sP
MAC Address: 00:0C:29:48:C6:6D (VMware)
Nmap scan report for 192.168.253.177


2、nmap 192.168.253.177 -p- -A
22/tcp closed ssh
80/tcp open   http    Apache httpd

3、web信息收集

http://192.168.253.177/

通过静态页面查看到图片，下载：
wget http://192.168.253.177/SpyderSecLogo200.png
wget http://192.168.253.177/Challenge.png

在Challenge.png发现数值：strings Challenge.png
iTXtComment
35:31:3a:35:33:3a:34:36:3a:35:37:3a:36:34:3a:35:38:3a:33:35:3a:37:31:3a:36:34:3a:34:35:3a:36:37:3a:36:61:3a:34:65:3a:37:61:3a:34:39:3a:33:35:3a:36:33:3a:33:30:3a:37:38:3a:34:32:3a:34:66:3a:33:32:3a:36:37:3a:33:30:3a:34:61:3a:35:31:3a:33:64:3a:33:64


https://www.rapidtables.com/convert/number/hex-to-ascii.html

51:53:46:57:64:58:35:71:64:45:67:6a:4e:7a:49:35:63:30:78:42:4f:32:67:30:4a:51:3d:3d

继续转码：
QSFWdX5qdEgjNzI5c0xBO2g0JQ==

echo 'QSFWdX5qdEgjNzI5c0xBO2g0JQ==' | base64 -d  ==
A!Vu~jtH#729sLA;h4%


还发现静态页面：
<script>
eval(function(p,a,c,k,e,d){e=function(c){return c.toString(36)};if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('7:0:1:2:8:6:3:5:4:0:a:1:2:d:c:b:f:3:9:e',16,16,'6c|65|72|27|75|6d|28|61|74|29|64|62|66|2e|3b|69'.split('|'),0,{}))
</script>

在谷歌上搜索了eval函数，发现是JavaScript它可以在其后隐藏代码，使用javascript解压缩器对其进行解压缩：
使用javascript unpacker进行转化：
http://matthewfl.com/unPacker.html
或者：https://www.strictly-software.com/unpack-javascript

61:6c:65:72:74:28:27:6d:75:6c:64:65:72:2e:66:62:69:27:29:3b
alert('mulder.fbi');   ---继续转ASICC即可


4、burpsuite分析

Cookie: URI=%2Fv%2F81JHPbvyEQ8729161jd6aKQ0N4%2F

发现了Cookie值，URL，转码：
URI=/v/81JHPbvyEQ8729161jd6aKQ0N4/

http://192.168.253.177/v/81JHPbvyEQ8729161jd6aKQ0N4/
发现无法访问！

http://192.168.253.177/v/81JHPbvyEQ8729161jd6aKQ0N4/mulder.fbi
下载分析！！！


5、视频分析

file mulder.fbi                                
mulder.fbi: ISO Media, MP4 v2 [ISO 14496-14]
发现它是一个视频文件！

参考文章：
http://oskarhane.com/hide-encrypted-files-inside-videos/


执行：python tcsteg2.py mulder.fbi
<TrueCrypt Container> is a TrueCrypt hidden volume
发现TrueCtype存在隐藏卷！

安装：TrueCrypt_7.1a.exe
安装好丢将mulder.fbi拖入TrueCrypt_7.1a！
需要输入密码：
A!Vu~jtH#729sLA;h4%


----------------
Congratulations! 

You are a winner. 

Please leave some feedback on your thoughts regarding this challenge?Was it fun? Was it hard enough or too easy? What did you like or dislike, what could be done better?

https://www.spydersec.com/feedback
---------------






















































































