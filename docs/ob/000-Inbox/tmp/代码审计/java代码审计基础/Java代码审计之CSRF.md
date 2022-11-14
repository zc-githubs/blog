# 漏洞原理

CSRF跨站请求伪造（Cross-site request forgery），当某个接口没有设置CSRF验证，点击了别人恶意的链接，可能会造成对这个接口发送相应的数据，造成某个数据被更改。-
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220901%2F1661995069_6310083d31a7f6df0d09e.png%21small)

# GET型

利用十分简单，构造一个IMG标签，加载的时候即可发送一个恶意get请求（可以和xss联合使用，也可以是有钓鱼，诱骗的方式让其点击get请求链接）

    <img src=https://xxx.cn/csrf?xx=11 />
    

# POST型

controller/CSRF.java

    @GetMapping("/")
    public String index() {
    	return "form";
    }
    @PostMapping("/post")
    @ResponseBody
    public String post() {
    	return "CSRF passed.";
    }
    

前端提交数据页面

    <div>
    	<!--Spring 3.2+和 Thymeleaf 2.1+可以通过这条语句自动生成一个csrf_token来验证-->
    	<form name="f" action="/csrf/post"method="post">
    		<input type="text" name="input" />
    		<input type="submit" value="Submit" />
    		<input type="hidden"th:name="${_csrf.parameterName}"th:value="${_csrf.token}" /><!--使用spring中内置token-->
    	</form>
    </div>
    

不提交token验证上面防护-
这个字段就是用来表单提交，POST请求验证的csrf\_token，后端生成的，提交后会和后端校验。如果我们直接通过POSTMAN或者其他post请求，缺少了csrf的token是无法完成的。如图-
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220901%2F1661995385_6310097909ca81f6fdbfc.png%21small)-
加了token可以提交数据-
![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220901%2F1661995401_631009894dabcca3fb39e.png%21small)-
审计前端html、jsp等前端页面，在提交表单时是否有token（隐藏属性）