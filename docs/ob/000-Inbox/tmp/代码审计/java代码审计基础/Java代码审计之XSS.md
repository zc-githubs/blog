# 漏洞原理

XSS攻击通常指的是通过利用网页开发时留下的漏洞，通过巧妙的方法注入恶意指令代码到网页，使用户加载并执行攻击者恶意制造的网页程序。这些恶意网页程序通常是JavaScript，但实际上也可以包括Java、 VBScript、ActiveX、 Flash 或者甚至是普通的HTML。

# 反射型xss

为了让我们方便理解，这里没有使用其他花里胡哨的html页面

@RequestMapping("/reflect")-
@ResponseBody-
publicstaticStringreflect(Stringxss) {-
returnxss;-
}

payload

xss=<script>alert(1)</script>

return将这个变量返回到html,导致了XSS漏洞可以看到最后执行了js代码，实现了弹窗。

![1657455296_62cac2c00d4d36783a5b9.png!small](https://image.cubox.pro/article/2022102615445491366/86286.jpg)

# 存储型xss

攻击者事先将恶意代码上传或储存到漏洞服务器中，只要受害者浏览包含此恶意代码的页面就会执行恶意代码。这就意味着只要访问了这个页面的访客，都有可能会执行这段恶意脚本，因此储存型XSS的危害会更大。因为存储型XSS的代码存在于网页的代码中，可以说是永久型的。

@RequestMapping("/stored/store")-
@ResponseBody-
publicStringstore(Stringxss, HttpServletResponseresponse) {-
Cookiecookie=newCookie("xss", xss);-
response.addCookie(cookie);-
return"Set param into cookie";-
}-
@RequestMapping("/stored/show")-
@ResponseBody-
publicStringshow(@CookieValue("xss") Stringxss) {-
returnxss;-
}

store方法将输入的便xss，存放在cookie里。show方法将获得的cookie返回到页面。payload

xss=<script>alert(1)</script>

xss的利用手段主要是网络蠕虫攻击和窃取用户cookie信息。xss蠕虫通过漏洞点嵌入恶意的js代码，执行代码后，就会添加带有恶意代码的页面或DOM元素，从而进行传播。而如果盗取cookie信息，常见的就是进行跨域请求的问题。

# 漏洞修复

@RequestMapping("/safe")-
@ResponseBody-
publicstaticStringsafe(Stringxss) {-
returnencode(xss);-
}-
-
privatestaticStringencode(Stringorigin) {-
/\* origin = StringUtils.replace(origin, "&", "&amp;");-
origin = StringUtils.replace(origin, "<", "&lt;");-
origin = StringUtils.replace(origin, ">", "&gt;");-
origin = StringUtils.replace(origin, "\\"", "&quot;");-
origin = StringUtils.replace(origin, "'", "&#x27;");-
origin = StringUtils.replace(origin, "/", "&#x2F;");\*/-
origin=StringUtils.replace(origin, "&", "＆");-
origin=StringUtils.replace(origin, "<", "＜");-
origin=StringUtils.replace(origin, ">", "＞");-
origin=StringUtils.replace(origin, "\\"", "＼");-
origin=StringUtils.replace(origin, "'", "＇");-
origin=StringUtils.replace(origin, "/", "／");-
returnorigin;-
}

以上就是通过字符转义的方法进行过滤，基本上杜绝了xss漏洞。但如果少写一个，可能就会被利用绕过。