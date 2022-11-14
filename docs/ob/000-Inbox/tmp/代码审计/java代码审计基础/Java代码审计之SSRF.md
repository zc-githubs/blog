# 漏洞介绍

SSRF漏洞（服务器端请求伪造）：是一种由攻击者构造形成由服务端发起请求的一个安全漏洞。一般情况下，SSRF攻击的目标是从外网无法访问的内部系统。（正是因为它是由服务端发起的，所以它能够请求到与它相连而与外网隔离的内部系统）。

# SSRF漏洞原理

SSRF形成的原因大都是由于服务端提供了从其他服务器应用获取数据的功能且没有对目标地址做过滤与限制。比如从指定URL地址获取网页文本内容，加载指定地址的图片，下载等等。利用的是服务端的请求伪造。SSRF是利用存在缺陷的web应用作为代理攻击远程和本地的服务器。  
Java网络请求中常见的几种协议：

```
file、ftp、http、https、jar、mailto、netdoc
```

# urlConnection/vul

使用file：///协议读取本地文件(或其他协议）  
[http://localhost:8080/ssrf/urlConnection/vuln?url=file:///c:/windows/win.ini](http://localhost:8080/ssrf/urlConnection/vuln?url=file:///c:/windows/win.ini)  
这里调用的是HttpUtils.URLConnection(url)

```
public String URLConnectionVuln(String url) {
	return HttpUtils.URLConnection(url);
}
```

而URLConnection里又调用了URL.openConnection()来发起请求,这个请求可以直接执行url协议（伪协议）

```
public static String URLConnection(String url) {
	try {
		URL u = new URL(url);
		URLConnection urlConnection = u.openConnection();
		BufferedReader in = new BufferedReader(new InputStreamReader(urlConnection.getInputStream())); //send request
		// BufferedReader in = new BufferedReader(new InputStreamReader(u.openConnection().getInputStream()));
		String inputLine;
		StringBuilder html = new StringBuilder();

		while ((inputLine = in.readLine()) != null) {
			html.append(inputLine);
		}
		in.close();
		return html.toString();
	} catch (Exception e) {
		logger.error(e.getMessage());
		return e.getMessage();
	}
}
```

代码跟踪  
发现传入的url被带入_URLConnection方法中_  
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220706%2F1657094479_62c5414f8c695a9a23e07.png%21small)_进入URLConnection方法，发现url对象u直接调用_openConnection方法并简单判断了是否为空，无过滤  
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220706%2F1657094491_62c5415b7ebe84b15c501.png%21small)回显出系统内部文件内容  
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220706%2F1657094506_62c5416a794541445800b.png%21small)

# urlConnection/sec(修复方法)

访问url为[http://localhost:8080/ssrf/urlConnection/sec?url=http://baidu.com](http://localhost:8080/ssrf/urlConnection/sec?url=http://baidu.com)  
这里先是对url调用了SecurityUtil.isHttp()来进行检查，随后又

```
@GetMapping("/urlConnection/sec")
public String URLConnectionSec(String url) {

	// Decline not http/https protocol
	if (!SecurityUtil.isHttp(url)) {
		return "[-] SSRF check failed";
	}

	try {
		SecurityUtil.startSSRFHook();
		return HttpUtils.URLConnection(url);
	} catch (SSRFException | IOException e) {
		return e.getMessage();
	} finally {
		SecurityUtil.stopSSRFHook();
	}

}
```

SecurityUtil.isHttp()比较简单，就是判断url是否是以http://或https://开头

```
public static boolean isHttp(String url) {
	return url.startsWith("http://") ||url.startsWith("https://");
}
```

单纯的ban掉其他协议显然是不够的，还不能够防止对内网进行探测，于是在获取url内容之前，开启了一个hook来对用户行为进行监听，SecurityUtil.startSSRFHook()，就有效防止了ssrf攻击  
代码跟踪  
url在进入中有个_isHttp方法_  
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220706%2F1657094524_62c5417ccc5a9066bcf28.png%21small)进入_isHttp方法，发现事先判断了是否是http或https_  
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220706%2F1657094535_62c541872624eeb1117ce.png%21small)我们传入的是ftp，所以判断为false，页面输出[-] SSRF check failed"  
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220706%2F1657094545_62c54191ee9d99e4b5e68.png%21small)

# openStream

openStream（）方法的实现也是调用了 openConnection生成一个 URLConnection 对象，然后再通过这个对象调用的getInputStream（）方法的  
访问url为  
[http://localhost:8080/ssrf/openStream?url=file:///c:/windows/win.ini](http://localhost:8080/ssrf/openStream?url=file:///c:/windows/win.ini)  
通过WebUtils.getNameWithoutExtension(url) + "." +WebUtils.getFileExtension(url)来获取下载文件名  
然后执行如下代码：

```
URL u = new URL(url);
inputStream = u.openStream()
```

来看一下openStream()，也是调用了openConnection()，也会根据传入的协议的不同来进行处理

```
public final InputStream openStream() throws java.io.IOException {
	return openConnection().getInputStream();
}
```

由此可以得知，同样也可以进行ssrf来探测内网以及文件下载。  
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220706%2F1657094561_62c541a1b1346ddcd0a04.png%21small)修复方案同上

# 关键词

```
URLConnection    openConnection    openStream
```