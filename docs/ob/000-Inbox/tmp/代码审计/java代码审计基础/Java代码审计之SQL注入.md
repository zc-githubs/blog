


## 漏洞原理：

web漏洞中，SQL注入漏洞较为常见，危害大。攻击者通过系统中存在的SQL注入漏洞来发起攻击，在条件允许的情况下，不仅可以获取整站数据，还可通过进一步的渗透来获取服务器权限，从而进入内网。-
注入攻击的本质，是用户输入的数据被当做代码执行。有两个关键条件，第一个是用户能够控制输入；第二个是原本程序要执行的代码，在不过滤的情况下拼接了用户输入的数据。

举例

    当用户发送GET请求：
    http://www.xxx.com/news.jsp?id=1
    

这是个新闻详情页面，会显示出新闻的title和content，程序内部会接收这个id参数传递给SQL语句，SQL如下：

    select title,content from news where id = 1
    

这是SQL的原义,也是程序员想要的结果,但是如果用户改变了id 的内容，修改成如下：

    http://www.xxx.com/news.jsp?id=1 and 1=2 UNION SELECT username, password FROM admin
    

此时内部程序执行的SQL语句为：

    SELECT title,content FROM news WHERE id = 1 and 1=2
    UNION SELECT username, password FROM admin
    

这条SQL的原义就会被改变，导致将管理员数据表中的用户名显示在页面title位置，密码显示在页面content位置，攻击成功。

## JDBC

在上古时期，人们往往这么从数据库获取数据。

    public User getUserById(String id) throws SQLException {
    	Connection connection =JDBCTOOLS.getConnection();
    	String sql = "select id,username from user where id=" + id;
    	Statement statement =connection.createStatement();
    	ResultSet resultSet =statement.executeQuery(sql);
    	resultSet.next();
    	int userId = resultSet.getInt(1);
    	String username = resultSet.getString(2);
    	User user = new User(userId, username);
    	return user;
    }
    

通过拼接字符串来构建sql语句，其中又有用户可控的部分，很容易出现问题。

### 直接拼接实例代码

    http://localhost:8080/sqli/jdbc/vul?username=admin
    

![图片.png](https://image.cubox.pro/article/2022102615442925667/37498.jpg)-
我们在执行sql语句通过断点调试，发现sql语句已经变成；

    select * from users where username = 'admin' or '1'='1'
    

![图片.png](https://image.cubox.pro/article/2022102508154516048/78134.jpg)-
执行结果：把数据库的全给查询出来-
![图片.png](https://image.cubox.pro/article/2022102508154554293/77918.jpg)

### 预编译处理后的代码（修复后代码）

    http://localhost:8080/sqli/jdbc/sec?username=admin
    

我们采用?号预先占位，再次进行调试，输入的内容已被替换(转义)成字符串-
![图片.png](https://image.cubox.pro/article/2022102508154546046/76822.jpg)-
![图片.png](https://image.cubox.pro/article/2022102508154542572/51548.jpg)-
执行结果：空-
![图片.png](https://image.cubox.pro/article/2022102508154544446/84516.jpg)-
但是预编译只能处理查询参数，很多场景下仅仅使用预编译是不够的。

### like情况

like拼接实现代码：

    http://localhost:8080/sqli/jdbc/like?username=adm
    

在模糊查询的场景中，以下写法是无法进行预编译的，程序会报错。

    String sql = "select * from user where username like '%?%'";
    

![图片.png](https://image.cubox.pro/article/2022102508154528906/34433.jpg)-
应先用？号占位，再用%test%,才可修复-
![图片.png](https://image.cubox.pro/article/2022102508154573266/39824.jpg)

### order by情况

order by与like一样，要分两步走才可避免情况-
需要按照时间、id等信息进行排序的时候，也是无法使用预编译的， sort=123

    String sort = req.getParameter("sort");
    String sql = "select * from user order by ?";
    PreparedStatement preparedStatement =connection.prepareStatement(sql); //预编译
    preparedStatement.setString(1, sort); //绑定参数
    ResultSet resultSet =preparedStatement.executeQuery();
    

如果像上面这样强行使用预编译，数据库会将字段名解析为字符串，即实际执行的sql为

    select * from user order by 'sort';
    

无法达到实际需求

### JDBC注入总结

jdbc方式进行拼接的，可以直接使用预处理来规避sql注入，但是如果有like、order by 进行参数拼接不能直接使用预处理来解决，必须在set处把%拼接上。

## Mybatis

### 前置知识

#### 识别mybatis

通过pom.xml可发现数据库采用mybatis-
![图片.png](https://image.cubox.pro/article/2022102508154527196/21042.jpg)

#### 了解Mybatis框架映射关系

1、mapper定位java代码-
parm定位方法

    List<User> findByUserNameVuln02(String username);
    定位到UserMapper.xml中的id
    <select id="findByUserNameVuln02" parameterType="String" resultMap="User">
    	select * from users where username like '%${_parameter}%'
    </select>
    _parameter是Mybatis的内置参数，代表整个参数
    

![图片.png](https://image.cubox.pro/article/2022102508154587397/85854.jpg)-
2、直接用@Select("sql语句")

    @Mapper
    public interface CategoryMapper {
    @Select("select * from category_ where name= '${name}' ")
    public CategoryM getByName(@Param("name") String name);
    }
    

![图片.png](https://image.cubox.pro/article/2022102508154543453/47397.jpg)

### Mybatis中两种数据库拼接方法

${xx}是直接拼接-
#{xxx}预处理后拼接 //string int float

#### 1、${}直接拼接

类似jdbc中的直接拼接审计方法：

    / * http://localhost:8080/sqli/mybatis/vuln01?username=joychou' or '1'='1
    */
    @GetMapping("/mybatis/vuln01")
    public List<User> mybatisVuln01(@RequestParam("username") String username) {
    return userMapper.findByUserNameVuln01(username);
    }
    这里使用的是mybatis来进行SQL查询，获取参数username后使用
    userMapper.findByUserNameVuln01(username)来进行查询。
    
    来看一下findByUserNameVuln01。
    MyBatis支持两种参数符号，一种是#，另一种是$。
    这里的参数获取使用的是${username}，而不是#{username}，
    而${username}是直接将参数拼接到了SQL查询语句中，就会造成SQL注入。
    
    @Select("select * from users where username = '${username}'")
    List<User> findByUserNameVuln01(@Param("username") String username);
    

#### 注入语句

    http://localhost:8080/sqli/mybatis/vuln01?username=admin' or '1'='1
    

![图片.png](https://image.cubox.pro/article/2022102508154555697/46879.jpg)

#### 修复方法

采用#{}，也就是jdbc中的预处理的形式

    @Select("select * from users where username = #{username}")
    User findByUserName(@Param("username") String username);
    

![图片.png](https://image.cubox.pro/article/2022102508154592506/84962.jpg)

### 2、Mybatis中like审计方法

    http://localhost:8080/sqli/mybatis/vuln02?username=admin
    

这里使用的是findByUserNameVuln02(String username);来进行查询，来看一下UserMapper.xml，也就是他的XML映射文件。

    List<User> findByUserNameVuln02(String username);
    
    UserMapper.xml:
    <select id="findByUserNameVuln02" parameterType="String" resultMap="User">
    	select * from users where username like '%${_parameter}%'
    </select>
    

这里传入未过滤的username后，插入到SQL语句中，就成了：-
select \* from users where username like '%username%'，

#### 注入语句

    http://localhost:8080/sqli/mybatis/vuln02?username=admin' or '1'='1' %23
    url编码，%23 #
    
    <select id="findByUserNameVuln02" parameterType="String" resultMap="User">
    	select * from users where username like '%admin' or '1'='1'
    </select>
    

![图片.png](https://image.cubox.pro/article/2022102508154525562/47683.jpg)

#### 修复建议

${}直接拼接字符串，而且这里在like的后面不能使用#{}预编译，不然就会产生报错。-
可以使用like concat('%',#{username}, '%')就可以避免注入了。

    http://localhost:8080/sqli/mybatis/vsec02?username=admin' or '1'='1' %23
    
    java代码：
    List<User> findByUserNameVsec02(String username);
    
    Mybatis配置：
    <select id="findByUserNameVsec02" parameterType="String" resultMap="User">
    	select * from users where username like concat('%',#{_parameter}, '%')
    </select>
    

![图片.png](https://image.cubox.pro/article/2022102508154533647/80333.jpg)-
同理不同数据库的修复代码快：-
Mysql数据库：

    SELECT * FROM user WHERE name like CONCAT('%',#{name},'%')
    

Oracle数据库：

    SELECT * FROM user WHERE name like CONCAT('%',#{name},'%')
    或
    SELECT * FROM user WHERE name like '%'||#{name}||'%'
    

Sqlserver数据库：

    SELECT * FROM user WHERE name like '%'+#{name}+'%'
    

DB2数据库：

    SELECT * FROM user WHERE name like '%'+#{name}+'%'
    或
    SELECT * FROM user WHERE name like '%'||#{name}||'%'
    

### 3、Mybatis中order by审计方法

这里使用的是findByUserNameVuln03(@Param("order") String order)来进行查询，同样也是去看UserMapper.xml。

    <select id="findByUserNameVuln03" parameterType="String" resultMap="User">
    	select * from users
    	<if test="order != null">
    		order by ${order} asc
    	</if>
    </select>
    

可以看到我们输入的参数在order by之后，也是${order}直接拼接起来了，与like相同，在这也是无法使用预编译，只能使用${}。由于没有过滤就直接拼接，很显然存在注入。

#### 注入语句

    http://localhost:8080/sqli/mybatis/orderby/vuln03?
    sort=1 and (updatexml(1,concat(0x7e,(select version()),0x7e),1))
    

![图片.png](https://image.cubox.pro/article/2022102508154593038/20183.jpg)

#### 修复代码

##### 1、后台直接写死

不传递参数到后端：

    http://localhost:8080/sqli/mybatis/sec03
    
    Java代码块：
    	User OrderByUsername();
    
    Mysql配置：
    	<select id="OrderByUsername" resultMap="User">
    		select * from users order by id asc limit 1
    	</select>
    

![图片.png](https://image.cubox.pro/article/2022102508154540721/79398.jpg)

##### 2、先过滤后使用

    http://localhost:8080/sqli/mybatis/orderby/sec04?sort=1
    
    Java代码块：
    @GetMapping("/mybatis/orderby/sec04")
    public List<User>
    mybatisOrderBySec04(@RequestParam("sort") String sort) {
    String filter_order =SecurityUtil.sqlFilter(sort);
    return userMapper.findByUserNameVuln03(filter_order);
    }
    //sqlFilter
    private static final Pattern FILTER_PATTERN =Pattern.compile("^[a-zA-Z0-9_/\\.-]+$");
    	public static String sqlFilter(String sql) {
    		if (!FILTER_PATTERN.matcher(sql).matches()) {
    		//严格限制用户输入只能包含a-zA-Z0-9_-.
    			return null;
    		}
    		return sql;
    }
    
    Mysql配置：
    <select id="findByUserNameVuln03" parameterType="String" resultMap="User">
    	select * from users
    	<if test="order != null">
    	order by ${order} asc
    	</if>
    </select>
    

\_发现经过sqlFilter后，_filter\_order变为null_-
![图片.png](https://image.cubox.pro/article/2022102508154567227/65129.jpg)-
跟下去发现使用正则处理，只要发现a-zA-Z0-9/\\.-，就把参数设置为空-
![图片.png](https://image.cubox.pro/article/2022102508154599064/99519.jpg)-
null参数跑到xml后不进行处理，也就预防了sql注入-
![图片.png](https://image.cubox.pro/article/2022102508154535346/33995.jpg)

## Hibernate

    @Autowired CategoryDAO categoryDAO; //依赖注入
    @RequestMapping("/hibernate")
    public String hibernate(@RequestParam(name = "id") int id) {
    	Category category = categoryDAO.getOne(id);
    	return category.getName();
    }
    

hibernate即我们经常使用的orm的一种实现，如果使用已封装好的方法，那么默认是使用预编译的。需要注意的有这么几种情况：

1.  对于一些复杂的sql语句，需要开发手写sql，此时要严格过滤用户输入。
    
2.  上面提到的预编译不生效的几种场景。