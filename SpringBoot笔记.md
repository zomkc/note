# 第一章

##### 1.1 配置类

JavaConfig: 使用java类作为xml配置文件的代替,是配置spring容器的纯java的方式

在这个java类可以创建java对象,把对象放入到spring容器中(注入到容器中)

使用两个注解

1)@Configuration: 放在一个类的上面,表示这个类是作为配置文件使用的

2)@Bean: 声明对象,把对象注入到容器中

```java
/*
   Configuration:表示当前类是作为配置文件使用的,就是用来配置文件
                位置:在类的上面
 */
@Configuration
public class SpringConfig {
    /*
    创建方法,方法的返回值是对象,在方法的上面加入@Bean
    方法的返回值对象就注入到容器中

        @Bean: 把对象注入到Spring容器中,作用相当于<Bean>
            位置: 方法的上面
            说明: @Bean,不指定对象的名称,默认的方法名是 id
     */
    @Bean
    public Student createStudent(){
        Student s = new Student();
        s.setName("张三");
        s.setAge(20);
        s.setSex("男");
        return s;
    }

    /*
        指定对象在容器中的名称(指定<bean>的id属性)
        @Bean的name属性,指定对象的名称(id)
     */
    @Bean(name = "lisiStudent")
    public Student makeStudent(){
        Student s = new Student();
        s.setName("张三");
        s.setAge(20);
        s.setSex("男");
        return s;
    }
}
```

##### 1.2@ImportResource 

@ImportResource作用导入其他的xml配置文件,等于 在xml

@Import	加载配置类

```xml
<import resources="其他配置文件"/>
```

列如:

```java
@Configuration
@ImportResource(value = {"classpath:applicationContext.xml","classpath:beans.xml"})
public class SpringConfig {}
```

##### 1.3@PropertyResource

@PropertyResource: 读取properties属性配置文件,使用属性配置文件可以实现外部化配置, 在程序代码之外提供数据

步骤:

1. 在resources目录下,创建properties文件,使用k=y的格式提供数据
2. 在PropertyResources指定properties文件的位置
3. 使用@Value(value="${key}")

```java
@Configuration
@ImportResource(value = {"classpath:applicationContext.xml","classpath:beans.xml"})
@PropertySource(value = "classpath:config.properties")
@ComponentScan(basePackages = "com.cn.vo")
public class SpringConfig { 		}
```



# 第二章 Spring Boot

## 介绍

SpringBoot是Spring中的一个成员， 可以简化Spring，SpringMVC的使用。 他的核心还是IOC容器。

特点：

- Create stand-alone Spring applications

  创建spring应用

  

- Embed Tomcat, Jetty or Undertow directly (no need to deploy WAR files)

  内嵌的tomcat， jetty ， Undertow 

  

- Provide opinionated 'starter' dependencies to simplify your build configuration

  提供了starter起步依赖，简化应用的配置。   

  

- Automatically configure Spring and 3rd party libraries whenever possible

  尽可能去配置spring和第三方库。叫做自动配置（就是把spring中的，第三方库中的对象都创建好，放到容器中， 开发人员可以直接使用）

  

- Provide production-ready features such as metrics, health checks, and externalized configuration

  提供了健康检查， 统计，外部化配置

  

- Absolutely no code generation and no requirement for XML configuration

  不用生成代码， 不用使用xml，做配置



##### 创建Spring Booot项目

第一种方式， 使用Spring提供的初始化器， 就是向导创建SpringBoot应用

使用的地址： https://start.spring.io

使用国内的地址:	https://start.springboot.io

阿里地址:	http://start.aliyun.com

启动类注解注解的使用

```java
@SpringBootApplication
复合注解：有
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan
```

1.@SpringBootConfiguration

说明：使用了@SpringBootConfiguration注解标注的类，可以作为配置文件使用的，
可以使用Bean声明对象，注入到容器

2.@EnableAutoConfiguration

启用自动配置， 把java对象配置好，注入到spring容器中。例如可以把mybatis的对象创建好，放入到容器中

3.@ComponentScan

@ComponentScan 扫描器，找到注解，根据注解的功能创建对象，给属性赋值等等。
默认扫描的包： @ComponentScan所在的类所在的包和子包。



##### 起步依赖

继承此依赖,由boot做依赖版本管理

```xml
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.7.7</version>
	</parent>
```

SpringBoot	web开发起步依赖

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
```

使用Maven依赖管理变更起步依赖

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
			<exclusions>
				<!--排除tomcat依赖-->
				<exclusion>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter-tomcat</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<!--使用Jetty-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jetty</artifactId>
		</dependency>
```

Jetty比tomcat更轻量级,可扩展性更强(相较于tomcat)



##### SpringBoot的配置文件

配置文件名称 : application

扩展名有 : properties(k=v) ; yml(  k:v)

使用application.properties , application.yml

列1: application.properties

```properties
#设置端口号
server.port=80
#设置访问应用上下文路径, contextpath
server.servlet.context-path=/myboot
```

列2 : application.yml/yaml

```yaml
server:
  port: 8083
  servlet:
    context-path: /mybatis2
```



多环境配置:

有开发环境, 测试环境, 上线环境

每个环境有不同的配置信息,列如端口, 上下文件, 数据库url, 用户名, 密码等

使用多环境配置文件, 可以方便的切换不同的配置

第一种方式: 

​	创建多个配置文件, 名称规则: application-环境名称.properties

​	创建开发环境的配置文件: application-dev.properties

​	创建测试者使用的配置: application-test.properties

```yaml
#激活使用dev的环境
spring:
  profiles:
    active: dev
    include:	#同时对多个环境生效,用逗号[,]分隔
```

第二种方式:	使用`---`分隔不同环境

```yaml
#设置启用的环境
spring:
  profiles:
    active: dev
---
#开发
spring:
  config:
    activate:
      on-profile: dev
server:
  port: 80
---
#测试
spring:
  config:
    activate:
      on-profile: pro
server:
  port: 81
---
#生产
spring:
  config:
    activate:
      on-profile: test
server:
  port: 82
```

多环境配置命令格式:

​		带参数启动SpringBoot

```shell
java -jar boot.jar --spring.profiles.active=test --server.port=88
```



yml数据读取:

1.@Value读取单个数据,属性名引用方式:	${一级属性名.二级属性名}

2.封装全部数据到Environment对象

```yaml
enterprise:
  name: 张三
  age: 18
  subject:
    - Java
    - 大数据
```

```java
    @Autowired
    private Environment environment;

    @GetMapping
    public String test(){
        String property = environment.getProperty("enterprise.name");
        System.out.println(property);
        return "hello,Boot";
    }
```

3.使用自定义对象封装指定数据,@ConfigurationProperties

```java
@Component
@ConfigurationProperties(prefix = "enterprise")
public class Enterprise {
    private String name;
    private String age;
    private String[] subject;
}
```

自定义对象封装数据警告解决方案

```xml
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-configuration-processor</artifactId>
		<optional>true</optional>
	</parent
```

`@ConfigurationProperties`为第三方bean绑定属性

```java
@Bean
@ConfigurationProperties(prefix = "datasource")
public DruidDataSource dataSource(){
    DruidDataSource ds = new DruidDataSource();
    return ds;
}
```

```yaml
datasource:
  driverClassName: com.mysql.cj.jdbc.Driver
```

@EnableConfigurationProperties注解可以将使用@ConfigurationProperties注解的类加入Spring容器



##### 日志

1.添加日志记录操作

```java
private static final Logger log = LoggerFactory.getLogger(BookController.class)
    @GetMapper
    public String getById(){
    log.debug("debug...");
    log.info("info...");
    log.warn("warn...");
    log.error("error...");
    return "running..."
}
/**
日志级别
	TRACE:运行堆栈信息,使用率低
	DEBUG:调试代码使用
	INFO:记录运维过程中数据
	WARN:记录运维过程警报数据
	ERROR:记录错误堆栈信息
	FATAL:灾难信息,合并计入ERROR
**/
```

2.设置日志输出级别

```yaml
#开启debug模式,输出调试信息,常用于检查系统运行状况
debug: true

#设置日志级别,root表示根节点,即整体应用日志级别
logging:
  level:
    root: info
    #设置某个包的日志级别
    com.cn.controller: debug
```

使用lombok提供的注解`@Slf4j`简化开发,减少日志对象的声明操作



日志输出格式控制:

![QQ截图20230112134446](assets/QQ%E6%88%AA%E5%9B%BE20230112134446.png)

```yaml
#设置日志输出格式
logging:
  pattern:
    console: "%d - %m %n" #%d:日期 %m:消息 %n:换行
```

设置日志文件:

```yaml
logging:
  file:
    name: server.log
    #详细配置
    logback:
      rollingpolicy:
        max-file-size: 3kb
        file-name-pattern: server.%d(yyyy-MM-dd).%i.log
```





##### SpringBoot整合Junit

```java
@SpringBootTest
class SpringBootTest {
    @Autowired
    private BookService bookService;
    @Test
    public void test(){
        bookService.save();
    }
}
```

@SpringBootTest

类型:	测试类注解

位置:	测试类定义上方

作用:	设置Junit加载的SpringBoot启动类

```java
@SpringBootTest(classes = SpringBootApplication.class)
class SpringBootTest {}
```

相关属性:

* classes:  设置SpringBoot启动类
* properties: 为当前测试用例添加临时的属性配置
* args: 为当前测试用例添加临时的命令行参数
* webEnvironment: 模拟端口

@AutoConfigureMockMvc:  web环境虚拟测试

![QQ截图20230113191628](assets/QQ%E6%88%AA%E5%9B%BE20230113191628.png)



![QQ截图20230113203004](assets/QQ%E6%88%AA%E5%9B%BE20230113203004.png)



##### 使用jsp

SpringBoot不推荐使用jsp, 而是使用模板技术代替jsp

使用jsp需要配置:

1)加入一个处理jsp的依赖, 负责编译jsp文件

```xml
<dependency>
      <groupId>org.apache.tomcat.embed</groupId>
      <artifactId>tomcat-embed-jasper</artifactId>
</dependency>
```

2)如果需要使用servlet , jsp , jstl的功能

```xml
<!--如果要使用 servlet 必须添加该以下两个依赖-->
<!-- servlet 依赖的 jar 包-->//
<dependency>
 	<groupId>javax.servlet</groupId>
 	<artifactId>javax.servlet-api</artifactId>
</dependency>
<!-- jsp 依赖 jar 包-->
<dependency>
 	<groupId>javax.servlet.jsp</groupId>
 	<artifactId>javax.servlet.jsp-api</artifactId>
 	<version>2.3.1</version>
</dependency>
<!--如果使用 JSTL 必须添加该依赖-->
<!--jstl 标签依赖的 jar 包 start-->
<dependency>
 	<groupId>javax.servlet</groupId>
 	<artifactId>jstl</artifactId>
</dependency>
```

3)创建一个存放jsp的目录, 一般叫做webapp

​	index.jsp

4)需要在pom.xml指定jsp文件编译后的存放目录

​	META-INF/resources

5)创建Controller , 访问jsp

6)在application.propertis文件中配置视图解析器



##### 使用容器

你想通过代码，从容器中获取对象。

通过SpringApplication.run(Application.class, args); 返回值获取容器。

```java

public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
        return run(new Class[]{primarySource}, args);
}

ConfigurableApplicationContext : 接口，是ApplicationContext的子接口
public interface ConfigurableApplicationContext extends ApplicationContext
```



ComnandLineRunner 接口 ，  ApplcationRunner接口

这两个接口都 有一个run方法。 执行时间在容器对象创建好后， 自动执行run（）方法。

可以完成自定义的在容器对象创建好的一些操作。

```java
@FunctionalInterface
public interface CommandLineRunner {
    void run(String... args) throws Exception;
}

@FunctionalInterface
public interface ApplicationRunner {
    void run(ApplicationArguments args) throws Exception;
}
```





# 第三章 Web组件

三个内容: 拦截器 , Servlet , Filter

##### 拦截器

拦截器是springMVC中的一种对象, 能拦截对Controller的请求

在框架中有系统默认的拦截器,还可以自定义拦截器, 实现对请求的预先处理



实现自定义拦截器

1. 创建类, 实现SpringMVC框架的HandlerInterceptor接口

```java
    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return true;
    }

    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable ModelAndView modelAndView) throws Exception {
    }

    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, @Nullable Exception ex) throws Exception {
    }
```

2. 需在SpringMVC的配置文件中, 声明拦截器

```xml
<mvc:interceptors>
	<mvc:Interceptor>
    	<mvc:path="url"/>
        <bean class="拦截器的全限定名称"/>
    </mvc:Interceptor>
</mvc:interceptors>
```



##### SpringBoot中注册拦截器:

```java
@Configuration
public class MyAppConfig implements WebMvcConfigurer {
    //添加拦截器对象,注入到容器中
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
       //创建拦截器对象
        HandlerInterceptor interceptor = new LogInterceptor();

       //指定拦截的请求url地址
       String[] path = {"/user/**"};
       //指定不拦截的地址
       String excludePath [] = {"/user/login"};
       registry.addInterceptor(interceptor)
               .addPathPatterns(path[0])
               .excludePathPatterns(excludePath);
    }
}
```



##### Servlet

在SpringBoot框架中使用Servlet对象

使用步骤:

1. 创建Servlet类, 创建类继承HttpServlet
2. 注册Servlet, 让框架能找到Servlet

1.创建自定义的Servlet

```java
public class MyServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=utf-8");
        PrintWriter out = resp.getWriter();
        out.println("执行的是Servlet");
        out.flush();
        out.close();
    }
```

2.注册Servlet

```java
@Configuration
public class WebApplictionConfig {
    //定义方法注册Servlet对象
    @Bean
    public ServletRegistrationBean servletRegistrationBean(){
        //第一个参数是 Servlet对象, 第二个是url地址
        //ServletRegistrationBean bean = new
                //ServletRegistrationBean(new MyServlet(),"/myservlet");

        ServletRegistrationBean bean = new ServletRegistrationBean();
        bean.setServlet(new MyServlet());
        bean.addUrlMappings("/login","/test");

        return bean;
    }
}
```



##### 过滤器Filter

filter是Servlet规范中的过滤器, 可以处理请求, 对请求的参数,属性进行调整. 尝尝在过滤器中处理字符编码

1. 创建自定义过滤器类
2. 注册Filter过滤器对象



```java
public class  MyFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("执行了MyFilter, doFilter");
        filterChain.doFilter(servletRequest,servletResponse);
    }
}
```

注册Filter

```java
@Configuration
public class WebApplicationConfig {
    @Bean
    public FilterRegistrationBean filterRegistrationBean(){
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new MyFilter());
        bean.addUrlPatterns("/user/*");
        return bean;
    }
}
```



##### 字符集过滤器

CharacterEncoding: 解决post请求中乱码的问题

在SpringMVC框架, 在web.xml注册过滤器, 配置其他的属性



第一种方式:

使用步骤:

1.配置字符集过滤器

```java
@Configuration
public class WebApplicationConfig {
    //注册Filter
    @Bean
    public FilterRegistrationBean filterRegistrationBean(){
        FilterRegistrationBean reg = new FilterRegistrationBean();
        //使用框架中的过滤器类
        CharacterEncodingFilter filter = new CharacterEncodingFilter();
        //指定使用的编码方式
        filter.setEncoding("utf-8");
        //指定request , response都使用encoding的值
        filter.setForceEncoding(true);

        reg.setFilter(filter);

        //指定 过滤的url地址
        reg.addUrlPatterns("/*");

        return reg;
    }
}
```

2.修改application.properties文件, 让自定义的过滤器起作用

```properties
#SpringBoot中默认已经配置了CharacterEncodingFilter, 编码默认ISO-8859-1
#设置enabled-false, 作用是关闭系统中配置好的过滤器, 使用自定义的CharacterEncodingFilter
server.servlet.encoding.enabled=false
```



第二种方式

```properties
#让系统的CharacterEncodingFilter生效
server.servlet.encoding.enabled=true
#指定使用的编码方式
server.servlet.encoding.charset=utf-8
#强制request , response都使用charset属性的值
server.servlet.encoding.force=true
```



# 第四章 整合Mybatis

使用MyBatis框架操作数据, 在springboot框架集成MyBatis

##### 1.起步依赖:

```xml
       SpringBoot整合mybatis
		<dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.3.0</version>
        </dependency>
		mysql驱动
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>
		[druid连接池]
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.2.3</version>
        </dependency>
		或
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.2.6</version>
        </dependency>
```

##### 2.设置数据源参数

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost/db1?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2B8
    username: root
    password: 123456
    #连接池配置
    type: com.alibaba.druid.pool.DruidDataSource
```

SpringBoot提供三种内置数据源对象供开发者选择

* HikariCP: 默认内置数据源对象
* Tomcat提供DataSource: HikariCP不可用的情况下,且在web环境中使用
* Commons DBCP: Hikari不可用,tomcat数据源不可用,将使用dbcp数据源

```yaml
spring:
  datasource:
    driver-class-name:...
    #hikari连接池配置
    hikari:
      maximum-pool-size: 50
```





##### 3.定义mapper

第一种方式 : @Mapper

@Mapper: 放在dao接口的上面,每个接口都需要使用这个注解

```java
/*
    @Mapper: 告诉Mybatis这是dao接口, 创建此接口的代理对象
        位置: 在类的上面
 */
@Mapper
public interface StudentDao {
    Student selectByID(@Param("stuId") Integer id);
    
    @Select("select * from student where id=#{id}")
    Student getById(Integer id);
}
```

第二种方式 : @MapperScan

```java
/*
    @MapperScan: 找到Dap接口和Mapper文件
        basePackages: Dao接口所在的包名
 */
@SpringBootApplication
@MapperScan(basePackages = "com.cn.dao")
public class rApplication {
```

第三种方式 : Mapper文件和Dao接口分开管理

现在把Mapper文件放到resources目录下

1)在resources目录创建子目录(自定义的),例如mapper

2)把mapper文件放到mapper目录中

3)在application.properties文件中,指定mapper文件的目录

```properties
#指定mapper文件的位置
mybatis.mapper-locations=classpath:mapper/*.xml
#指定mybatis的日志
mybatis.configuration.log-impl=org.apache.ibatis.logging.stdout.StdOutImpl
```



##### 事务

Spring框架中的事务

1)管理事务的对象 : 事务管理器(接口, 接口有很多的实现类)

​	例如: 使用dbc或mysql访问数据库, 使用的事务管理器: DataSourceTransactionManager

2)声明式事务: 在xml配置文件或者使用注解说明事务控制的内容

​	控制事务: 隔离级别 , 传播行为 , 超时时间

3)事务处理方式:

1. Spring框架中的@Transactional
2. aspectj框架可以在xml配置文件中 , 声明事务控制的内容



SpringBoot中使用事务 : 上面的两种方式都可以

1)在业务方法的上面加入@Transactional,加入注解后,方法就有事务功能了

2)明确的在主启动类的上面,加入@EnableTransactionManager

 

列子:

```java
    @Resource
    private StudentMapper studentMapper;

    /*
        @Transactional:表示方法有事务支持
            默认: 使用数据库的隔离级别 , REQUIRED 传播新闻: 超时时间 -1
            抛出运行时异常,回滚事务
     */
    @Transactional
    @Override
    public int addStudent(Student student) {
        System.out.println("业务方法addStudent");
        int rows  = studentMapper.insert(student);
        System.out.println("执行sql语句");
        //抛出一个运行时异常,目的是回滚事务
//        int m = 10 / 0;
        return rows;
    }
```



# 第五章 接口架构风格 --RESTful

接口 : API(Application Programming Interface , 应用程序接口) 是一些预先定义的接口(如函数, HTTP接口), 或指软件系统不同组成部分衔接的约定,用来提供应用程序与开发人员基于某软件或硬件得以访问的一组例程,而又无需访问源码,或理解内部工作机制的细节

接口(API) : 可以指访问servlet , controller的url , 调用其他程序的 函数

架构风格: api组织方式(样子)

​	一个传统的 : [localhost:9001/mytrans/addStudent?name=林青霞&age=23](http://localhost:9001/mytrans/addStudent?name=林青霞&age=23)

​				在地址上提供了访问资源名称addStudent,在其后使用get方式传递参数

##### 5.1 REST

RESTful架构风格

1 )REST : (英文 : 英文：Representational State Transfer , 中文 :表现层状态转移)

​	REST : 是一种接口的架构风格和设计的理念 , 不是标准

​	优点 :更简洁 , 更有层次感

2 )REST中的要素:

用REST表示资源和对资源的操作, 在互联网中,表示一个资源或者一个操作

资源使用url表示的,在互联网,使用的图片,视频,文本,网页等等都是资源

资源是用名词表示

对资源:

​	查询资源 : 通过rul找到资源

​	创建资源 : 添加资源

​	更新资源 : 更新资源,编辑

​	删除资源 : 去除

资源使用rul表示,通过名词表示资源

​	在url中,使用名词表示资源,以及访问资源的信息,在url中,使用"  \ "分隔对资源的信息

使用http中的动作(请求方式) , 表示对资源的操作(CURD)

GET : 查询资源 --sql select

​		处理单个资源 : 用它的单数方式

​		http://localhost:8080/myboot/studet/1001

​		处理多个资源 : 使用复数形式

​		http://localhost:8080/myboot/studet/1001/1004

POST : 创建资源 --sql insert

​		http://localhost:8080/myboot/studet		在post请求中传递参数

```xml
<form action="http://localhost:8080/myboot/studet" method="post">
	姓名:<input type="text" name="name"/>
    年龄:<input type="text" name="age"/>
</form>
```

PUT : 更新资源 --sql update

```xml
<form action="http://localhost:8080/myboot/studet/1" method="post">
	姓名:<input type="text" name="name"/>
    年龄:<input type="text" name="age"/>
    <input type="hidden" name="_method" value="PUT"/>
</form>
```

DELETE : 删除资源 --sql delete

```xml
<a href="http://localgost:8080/myboot/student/1">删除</a>
```

3 )使用url表示资源,使用http动作操作资源就是REST

##### 4 )注解

​	@PathVariable : 从url中获取数据

​	@GetMappring : 支持get请求方式,等同于

​								@RequestMapping(method=RequestMethod.GET)

​	@PostMapping : 支持post请求方式,等同于

​								@RequestMapping(method=RequestMethod.GET)

​	@PutMapping : 支持Put请求方式,等同于

​								@RequestMapping(method=RequestMethod.PUT)

​	@RestController : 复合注解,是@Controller和@ResponseBody组合

​	在类的上面使用RestController , 表示当前类所有方法都加入@ResponseBody

```java
/*@PathVariable: (路径变量) 获取url中的数据
            属性: value : 路径变量名
            位置: 放在控制器方法的形参前面
      {stuid}: 自定义名称
      当路径变量名和形参名一样 ,@PathVariable的value可以省略  */
@GetMapping("/student/{stuid}")
public String queryStudent(@PathVariable("stuid") Integer stuid){
    return "查询学生:"+stuid;
}
```



##### 5.2 在页面中或者ajax中, 支持put , delete请求

在SpringMVC中有一个过滤器 , 支持post请求转为put , delete

过滤器: org.springframework.web.filter.HttpPutFormContentFilter

作用: 把请求中的post请求转为put , delete

实现步骤:

​	1.application.properties(yml) : 开启使用HttpPutFormContentFilter 过滤器

```properties
#启用支持put delete
spring.mvc.formcontent.filter.enabled=true
```

​	2.在请求页面中, 包含 _method参数 , 它的值是put , delete , 发起这个请求使用post方式

```html
<form action="student/test" method="post">
        <input type="hidden" name="_method" value="put">
        <input type="submit" value="put提交测试请求">
    </form>
```



# 第六章 NoSQL

##### Redis

一个NoSQL数据库 , 常用作缓存使用(cache)

Redis的数据类型 : String , hash , set , zset , list



Redis是一个中间件 : 是一个独立的服务器

java中注明的客户端 : jedis , lettuce , Redisson

lettuce与jedis的区别

* jedis连接Redis服务器是直连模式,当多线程模式下使用jedis会存在线程安全问题,解决方案可以通过配置连接池是每个连接专用,这样整体性能会大受影响
* lettuce基于Netty框架进行与Redis服务器连接,底层设计中采用StatefulRedisConnection,是线程安全的,可以保障并发访问安全问题,所以一个连接可以被多线程复用,当然lettuce也支持多连接实例一起工作



Spring , SpringBoot中有一个RedisTemplate(StringRedisTemplate) , 处理和redis交互



RedisTemplate 使用的是 lettuce 客户端库

```xml
<!--redis起步依赖: 直接在项目中使用RedisTemplate(StringRedisTemplate)-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

data-redis使用的 lettuce客户端库

在程序中使用RedisTemplate类的方法 操作redis数据,实际上就是调用lettuce 客户端
```

对比 StringRedisTemplate 和 RedisTemplate

StringRedisTemplate :把k , v都是作为String处理 , 使用的是String的序列化 , 可读性好

RedisTemplate : 把k , v 经过序列化存到redis, k, v是序列化的内容 , 不能直接识别

​								默认使用jdk序列化 , 可以修改为其他的序列化



序列化: 把对象转为可传输的字节序列过程为序列化

反序列化: 把字节序列还原为对象的过程称为反序列化



为什么需要序列化

序列化最终的目的是为了对象可以跨平台存储，和进行网络传输。而我们进行跨平台存储和网络传输的方式就是IO，而我们的IO支持的数据格式就是字节数组。我们必须在把对象转成字节数组的时候就制定一种规则（序列化），那么我们从IO流里面读出数据的时候再以这种规则把对象还原回来（反序列化）。



什么情况下需要序列化

通过上面我想你已经知道了凡是需要进行“跨平台存储”和”网络传输”的数据，都需要进行序列化。

本质上存储和网络传输 都需要经过 把一个对象状态保存成一种跨平台识别的字节格式，然后其他的平台才可以通过字节信息解析还原对象信息。



序列化的方式

序列化只是一种拆装组装对象的规则，那么这种规则肯定也可能有多种多样，比如现在常见的序列化方式有：

JDK（不支持跨语言）、JSON、XML、Hessian、Kryo（不支持跨语言）、Thrift、Protofbuff、



Student( name=zs, age=20)   ----  { "name":"zs", "age":20 }



java的序列化： 把java对象转为byte[], 二进制数据

json序列化：json序列化功能将对象转换为 JSON 格式或从 JSON 格式转换对象。例如把一个Student对象转换为JSON字符串{"name":"李四", "age":29} )，反序列化(将JSON字符串 {"name":"李四", "age":29} 转换为Student对象)



设置key或者value的序列化方式

```java
// 使用RedisTemplate ，在存取值之前，设置序列化
// 设置 key 使用String的序列化
redisTemplate.setKeySerializer( new StringRedisSerializer());

// 设置 value 的序列化
redisTemplate.setValueSerializer( new StringRedisSerializer());

redisTemplate.opsForValue().set(k,v);
```



##### MongoDB

MongoDB是一个开源,高性能,无模式的文档型数据库,NoSQL数据库产品中的一种,是最像关系型数据库的非关系型数据库

起步依赖:

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-mongodb</artifactId>
        </dependency>
```

配置:

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://localhost/it
```

使用MongoTemplate的api操作数据库

```java
@Autowired
private MongoTemplate mongoTemplate;

```



##### SpringBoot内置的缓存技术

1.导入坐标

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-cache</artifactId>
        </dependency>
```

2.启用缓存

* @EnableCaching

3.设置当前操作结果数据进入缓存

```java
@Cacheable(value="cacheSpace",key="#id")
public Book get(Integer id){
    return bookDao.selectById(id);
}
```

SpringBoot提供的缓存技术除了默认的缓存方案,还可以对其他缓存技术进行整合,统一接口,方便缓存技术的开发与管理





# 第七章 SpringBoot集成Dubbo

 SpringBoot集成Dubbo的文档

 https://github.com/apache/dubbo-spring-boot-project/blob/master/README_CN.md



##### 公共项目

独立的maven项目： 定义了接口和数据类

```java
public class Student implements Serializable {
    private static final long serialVersionUID = 1901229007746699151L;

    private Integer id;
    private String name;
    private Integer age;
}

public interface StudentService {

    Student queryStudent(Integer id);
}

```



##### 提供者

创建SpringBoot项目

1） pom.xml

```xml
<dependencies>

   <!--加入公共项目的gav-->
   <dependency>
      <groupId>com.bjpowernode</groupId>
      <artifactId>022-interface-api</artifactId>
      <version>1.0.0</version>
   </dependency>

   <!--dubbo依赖-->
   <dependency>
      <groupId>org.apache.dubbo</groupId>
      <artifactId>dubbo-spring-boot-starter</artifactId>
      <version>2.7.8</version>
   </dependency>


   <!--zookeeper依赖-->
   <dependency>
      <groupId>org.apache.dubbo</groupId>
      <artifactId>dubbo-dependencies-zookeeper</artifactId>
      <version>2.7.8</version>
      <type>pom</type>
      <exclusions>
         <!-- 排除log4j依赖 -->
         <exclusion>
            <artifactId>slf4j-log4j12</artifactId>
            <groupId>org.slf4j</groupId>
         </exclusion>
      </exclusions>
   </dependency>
</dependencies>
```



2）实现接口

```java
/**
 * 使用dubbo中的注解暴露服务
 * @Component 可以不用加
 */
@DubboService(interfaceClass = StudentService.class,version = "1.0",timeout = 5000)
public class StudentServiceImpl implements StudentService {
    @Override
    public Student queryStudent(Integer id) {
        Student student  = new Student();
        if( 1001 == id){
            student.setId(1001);
            student.setName("------1001-张三");
            student.setAge(20);
        } else if(1002  == id){
            student.setId(1002);
            student.setName("#######1002-李四");
            student.setAge(22);
        }

        return student;
    }
}
```



3）application.properties

```properties
#配置服务名称 dubbo:application name="名称"
spring.application.name=studentservice-provider

#配置扫描的包， 扫描的@DubboService
dubbo.scan.base-packages=com.bjpowernode.service

#配置dubbo协议
#dubbo.protocol.name=dubbo
#dubbo.protocol.port=20881

#注册中心
dubbo.registry.address=zookeeper://localhost:2181
```



4)在启动类的上面

```java
@SpringBootApplication
@EnableDubbo
public class ProviderApplication {

   public static void main(String[] args) {
      SpringApplication.run(ProviderApplication.class, args);
   }
}
```



##### 消费者

创建SpringBoot项目

1） pom.xml

```xml
<dependencies>

   <!--加入公共项目的gav-->
   <dependency>
      <groupId>com.bjpowernode</groupId>
      <artifactId>022-interface-api</artifactId>
      <version>1.0.0</version>
   </dependency>

   <!--dubbo依赖-->
   <dependency>
      <groupId>org.apache.dubbo</groupId>
      <artifactId>dubbo-spring-boot-starter</artifactId>
      <version>2.7.8</version>
   </dependency>


   <!--zookeeper依赖-->
   <dependency>
      <groupId>org.apache.dubbo</groupId>
      <artifactId>dubbo-dependencies-zookeeper</artifactId>
      <version>2.7.8</version>
      <type>pom</type>
      <exclusions>
         <!-- 排除log4j依赖 -->
         <exclusion>
            <artifactId>slf4j-log4j12</artifactId>
            <groupId>org.slf4j</groupId>
         </exclusion>
      </exclusions>
   </dependency>
</dependencies>
```

2)  创建了Controller 或者 Service都可以

```java
@RestController
public class DubboController {

    /**
     * 引用远程服务， 把创建好的代理对象，注入给studentService
     */
    //@DubboReference(interfaceClass = StudentService.class,version = "1.0")

    /**
     * 没有使用interfaceClass，默认的就是 引用类型的 数据类型
      */
    @DubboReference(version = "1.0")
    private StudentService studentService;

    @GetMapping("/query")
    public String queryStudent(Integer id){
        Student student   = studentService.queryStudent(id);
        return "调用远程接口，获取对象："+student;
    }
}
```



3）application.properties

```properties
#指定服务名称
spring.application.name=consumer-application
#指定注册中心
dubbo.registry.address=zookeeper://localhost:2181
```





# 第八章  打包

##### 打包war

1.创建了一个jsp应用

2.修改pom.xml

 1)指定打包后的文件名称

```xml
<build>
   <!--打包后的文件名称-->
   <finalName>myboot</finalName>
</build>
```



2)指定jsp编译目录

```xml
<!--resources插件， 把jsp编译到指定的目录-->
<resources>
   <resource>
      <directory>src/main/webapp</directory>
      <targetPath>META-INF/resources</targetPath>
      <includes>
         <include>**/*.*</include>
      </includes>
   </resource>

   <!--使用了mybatis ，而且mapper文件放在src/main/java目录-->
   <resource>
      <directory>src/main/java</directory>
      <includes>
         <include>**/*.xml</include>
      </includes>
   </resource>

   <!--把src/main/resources下面的所有文件，都包含到classes目录-->
   <resource>
      <directory>src/main/resources</directory>
      <includes>
         <include>**/*.*</include>
      </includes>
   </resource>
</resources>
```



3）执行打包是war

```xml
<!--打包类型-->
<packaging>war</packaging>
```



4）主启动类继承SpringBootServletInitializer

```java
/**
 * SpringBootServletInitializer: 继承这个类， 才能使用独立tomcat服务器
 */
@SpringBootApplication
public class JspApplication  extends SpringBootServletInitializer  {

   public static void main(String[] args) {
      SpringApplication.run(JspApplication.class, args);
   }

   @Override
   protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
      return builder.sources(JspApplication.class);
   }
}
```



5）部署war

 把war放到tomcat等服务器的发布目录中。  tomcat为例， myboot.war放到tomcat/webapps目录。





##### 打包为jar

1.创建了一个包含了jsp的项目

2.修改pom.xml

​     1) 指定打包后的文件名称

```xml
<build>
   <!--打包后的文件名称-->
   <finalName>myboot</finalName>
</build>
```



    2) 指定springboot-maven-plugin版本

```xml
<plugins>
   <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
      <!--打包jar， 有jsp文件时，必须指定maven-plugin插件的版本是 1.4.2.RELEASE-->
      <version>1.4.2.RELEASE</version>
   </plugin>
</plugins>
```



3）最后执行 maven clean package

​       在target目录中，生成jar 文件， 例子是myboot.jar



​       执行独立的springboot项目  在cmd中 java  -jar  myboot.jar







# 第九章 Thymeleaf

Thymeleag : 是使用java开发的模板技术, 在服务器端运行, 把处理后的数据发送给浏览器

​	模板是做视图层工作的, 显示数据的, Thymeleaf是基于Html语言, Thymeleaf语法是应用在html标签中.	SpringBoot框架集成了Thymeleaf, 使用Thymeleaf代替jsp



Thymeleaf 的官方网站：http://www.thymeleaf.org

Thymeleaf 官方手册：https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html

```properties
#在开发阶段,关闭模板缓存,让修改立即生效
spring.thymeleaf.cache=false

#默认的
#编码格式
spring.thymeleaf.encoding=UTF-8

#模板的类型(默认HTML , 模板是html文件)
spring.thymeleaf.mode=HTML

#模板的前缀: 类路径的 classpath:/templates
spring.thymeleaf.prefix=classpath:/templates/

#后缀,默认.html
spring.thymeleaf.suffix=.html
```





##### 表达式

1.标准变量表达式

​	语法:	${key}

​	作用:	获取key对应的文本数据,key是request作用域中的key,使用

request.setAttribute , model.addAttribute

​	在页面中的html标签中, 使用`th:text="${key}"`

```html
    <h3>标准变量表达式</h3>
    <p th:text="${data}">被替换的内容</p>

    <p th:text="${student.id}"></p>
    <p th:text="${student.name}"></p>
    <p th:text="${student.age}"></p>
    <p th:text="${student.getName()}"></p>
```





2.选择变量表达式(*表达式)

​	语法:	*{key}

​	作用:	获取这个key对应的数据,	*{key}需要和`th:object`这个属性一起使用

​	目的是简单获取对象的属性值

```html
    <h3>选择变量表达式</h3>
    <div th:object="${student}">
        <p th:text="*{getId()}"></p>
        <p th:text="*{name}"></p>
        <p th:text="*{age}"></p>
    </div>
    <p th:text="*{student.id}"></p>
```



3.连接表达式

​	语法:	@{url}

​	作用:	表示连接

```html
<script src="..." , <link href="..." , <from action="..."
<img src="@{/地址(id=11 , name="张三" , age=${age})}"/>                                             
```



##### Thymeleaf属性

属性是放在html元素中的,就是html元素的属性,加入了th前缀,属性的作用不变,

加入th, 属性的值由模板引擎处理了,在属性中可以使用变量表达式

例如:

```html
<form action="/aaa" method="post"></form>
<form th:action="/aaa" th:method="${me}"></form>
```



##### each

each循环,可以循环List , Array , Map

语法:

在一个html标签中,使用th:each

```html
<div th:each="集合循环成员,循环的姿态变量:${key}">
    <p th:text="${集合循环成员}"></p>
</div>
集合循环成员,循环的姿态变量:两个名称都是自定义的
"循环的姿态变量"这个名称可以不定义,默认是"集合循环成员Stat"
```



each循环Map

在一个html标签中,使用th:each

```html
<div th:each="集合循环成员,循环的姿态变量:${key}">
    <p th:text="${集合循环成员.key}"></p>
     <p th:text="${集合循环成员.value}"></p>
</div>
集合循环成员,循环的姿态变量:两个名称都是自定义的
"循环的姿态变量"这个名称可以不定义,默认是"集合循环成员Stat"

key: map集合中的key
value: map集合key对应的value值
```



##### th:if

"th:if"  : 判断语句， 当条件为true， 显示html标签体内， 反之不显示 没有else语句

```xml
语法：
<div th:if=" 10 > 0 "> 显示文本内容 </div>

```



还有一个 th:unless  和 th:if相反的行为

```xml
语法：
<div th:unless=" 10 < 0 "> 当条件为false显示标签体内容 </div>
```



例子：if

```xml
<div style="margin-left: 400px">
        <h3> if 使用</h3>
        <p th:if="${sex=='m'}">性别是男</p>
        <p th:if="${isLogin}">已经登录系统</p>
        <p th:if="${age > 20}">年龄大于20</p>
        <!--""空字符是true-->
        <p th:if="${name}">name是“”</p>
        <!--null是false-->
        <p th:if="${isOld}"> isOld是null</p>
 </div>

```



##### th:switch

th:switch和java中的switch一样的

```html
语法：
<div th:switch="要比对的值">
    <p th:case="值1">
        结果1
    </p>
    <p th:case="值2">
        结果2
    </p>
    <p th:case="*">
        默认结果
    </p>
    以上的case只有一个语句执行
    
</div>
```



##### th:inline

1. 内联text：  在html标签外，获取表达式的值

   语法： 

   ```xml
   <p>显示姓名是：[[${key}]]</p>
   
    <div style="margin-left: 400px">
           <h3>内联 text, 使用内联表达式显示变量的值</h3>
           <div th:inline="text">
               <p>我是[[${name}]]，年龄是[[${age}]]</p>
               我是<span th:text="${name}"></span>,年龄是<span th:text="${age}"></span>
           </div>
   
           <div>
               <p>使用内联text</p>
               <p>我是[[${name}]],性别是[[${sex}]]</p>
           </div>
   </div>
   ```

   2.内联javascript

   ```html
   例子：
    <script type="text/javascript" th:inline="javascript">
            var myname = [[${name}]];
            var myage = [[${age}]];
   
            //alert("获取的模板中数据 "+ myname + ","+myage)
   
           function fun(){
               alert("单击事件，获取数据 "+ myname + ","+ [[${sex}]])
           }
       </script>
   ```










# 第十章 总结

## 10.1 注解

Spring + SpringMVC + SpringBoot 

```java
创建对象的：
@Controller: 放在类的上面，创建控制器对象，注入到容器中
@RestController: 放在类的上面，创建控制器对象，注入到容器中。
             作用：复合注解是@Controller , @ResponseBody, 使用这个注解类的，里面的控制器方法的返回值                   都是数据

@Service ： 放在业务层的实现类上面，创建service对象，注入到容器
@Repository : 放在dao层的实现类上面，创建dao对象，放入到容器。 没有使用这个注解，是因为现在使用MyBatis框               架，  dao对象是MyBatis通过代理生成的。 不需要使用@Repository、 所以没有使用。
@Component:  放在类的上面，创建此类的对象，放入到容器中。 

赋值的：
@Value ： 简单类型的赋值， 例如 在属性的上面使用@Value("李四") private String name
          还可以使用@Value,获取配置文件者的数据（properties或yml）。 
          @Value("${server.port}") private Integer port

@Autowired: 引用类型赋值自动注入的，支持byName, byType. 默认是byType 。 放在属性的上面，也可以放在构造             方法的上面。 推荐是放在构造方法的上面
@Qualifer:  给引用类型赋值，使用byName方式。   
            @Autowird, @Qualifer都是Spring框架提供的。

@Resource ： 来自jdk中的定义， javax.annotation。 实现引用类型的自动注入， 支持byName, byType.
             默认是byName, 如果byName失败， 再使用byType注入。 在属性上面使用


其他：
@Configuration ： 放在类的上面，表示这是个配置类，相当于xml配置文件

@Bean：放在方法的上面， 把方法的返回值对象，注入到spring容器中。

@ImportResource ： 加载其他的xml配置文件， 把文件中的对象注入到spring容器中

@PropertySource ： 读取其他的properties属性配置文件

@ComponentScan： 扫描器 ，指定包名，扫描注解的

@ResponseBody: 放在方法的上面，表示方法的返回值是数据， 不是视图
@RequestBody : 把请求体中的数据，读取出来， 转为java对象使用。

@ControllerAdvice:  控制器增强， 放在类的上面， 表示此类提供了方法，可以对controller增强功能。

@ExceptionHandler : 处理异常的，放在方法的上面

@Transcational :  处理事务的， 放在service实现类的public方法上面， 表示此方法有事务


SpringBoot中使用的注解
    
@SpringBootApplication ： 放在启动类上面， 包含了@SpringBootConfiguration
                          @EnableAutoConfiguration， @ComponentScan


    
MyBatis相关的注解

@Mapper ： 放在类的上面 ， 让MyBatis找到接口， 创建他的代理对象    
@MapperScan :放在主类的上面 ， 指定扫描的包， 把这个包中的所有接口都创建代理对象。 对象注入到容器中
@Param ： 放在dao接口的方法的形参前面， 作为命名参数使用的。
    
Dubbo注解
@DubboService: 在提供者端使用的，暴露服务的， 放在接口的实现类上面
@DubboReference:  在消费者端使用的， 引用远程服务， 放在属性上面使用。
@EnableDubbo : 放在主类上面， 表示当前引用启用Dubbo功能。
```



##### SpringBoot定时任务

1.开启定时任务功能

​	`@EnableScheduling`

2.设置定时执行的任务,并设定执行周期

```java
@Component
public class a{
    @Scheduled(cron = "0/5 * * * * ?")
    public void printLog(){
        System.out.println("run");
    }
}
```

##### SpringBoot整合JavaMail 邮件

SMTP: 简单邮件传输协议,用于发送电子邮件的传输协议

POP3: 用于接收电子邮件的标准协议

IMAP: 互联网消息协议,是POP3的代替协议

发送简单消息:

1.起步依赖:	导入SpringBoot整合javaMail的坐标

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
```

2.配置javaMail

​		到邮件供应商开启SMTP服务

```yaml
spring:
  mail:
    host: smtp.qq.com
    username: ***@qq.com
    password: ***
```

3.发送邮件

```java
@Service
public class SendMailServiceImpl implements SendMailService {
    @Autowired
    private JavaMailSender javaMailSender;
    //发件人
    private String from = "3531014073@qq.com";
    //接收人
    private String to = "cxzbnh@163.com";
    //标题
    private String subject = "测试邮件";
    //正文
    private String context = "测试邮件正文内容";
    @Override
    public void senMail() {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setFrom(from+"(小甜甜)");//括号内容替代发送人名字
        message.setTo(to);
        message.setSubject(subject);
        message.setText(context);
        javaMailSender.send(message);
    }
}
```

发送复杂消息:

```java
        //发送复杂邮件
            MimeMessage message = javaMailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(message,true);//true表示允许发送附件
            helper.setFrom(from+"(小甜甜)");//括号内容替代发送人名字
            helper.setTo(to);
            helper.setSubject(subject);
            helper.setText(context,true);//允许解析html标签
            //发送附件
            File f1 = new File("C:\\Users\\Administrator.LAPTOP-4JK1EL3F\\Documents\\9708.jpeg");
            helper.addAttachment("1.jpeg",f1);
            javaMailSender.send(message);
```



##### 消息

企业级应用中广泛使用的三种异步消息传递技术

* JMS: 一个规范,等同于jdbc规范,提供了与消息服务相关的API接口
* AMQP: 一种协议(高级消息队列,也就是消息代理规范),规范了网络交换的数据格式,兼容JMS
* MQTT: 消息队列遥测传输,专为小设备涉及,是物联网(IOT)生态系统中主要成分之一



SpringBoot整合ActiveMQ

1.导入坐标

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-activemq</artifactId>
        </dependency>
```

2.配置ActiveMQ

```yaml
spring:
  activemq:
    broker-url: tcp://localhost:61616
  jms:
    pub-sub-domain: true #消息发布订阅模式
```

3.生产与消费消息

```java
@Autowired
private JmsMessagingTemplate jmsMessagingTemplate;
//发送消息
jmsMessagingTemplate.convertAndSend("order.id",1);
//接收消息
jmsMessagingTemplate.receiveAndConvert("order.id",Integer.class);
```

使用消息监听器对消息队列监听

```java
@Component
public class MessageListener {
    @JmsListener(destination = "order.id")
    //流程性业务消息消费完转入下一个消息队列
    @SendTo("order.other.id")
    public String receive(String id){
        System.out.println("收到消息,id="+id)
    }
}
```



##### 可视化监控平台	SpringBoot Admin

Admin服务端

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-admin-starter-server</artifactId>
            <version>2.5.4</version>
        </dependency>
```

服务端加入web依赖,并设置启用Spring-Admin

```yaml
server:
  port: 8080
```

`@EnableAdminServer`



Admin客户端

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-admin-starter-client</artifactId>
            <version>2.5.4</version>
        </dependency>
```

```yaml
spring:
  boot: 
    admin:
      client:
        url: http://localhost:8080
management:
  endpoint:
    health:
      show-details: always
  endpoints:
    web:
      exposure:
        include: "*"
```



##### 统一异常处理	ControllerAdvice

```java
@Slf4j
@RestControllerAdvice(basePackages = "com.zomkc.product.controller")
public class ExceptionController {

    @ExceptionHandler(value = Exception.class)
    public R handleAdviceException(Exception e){
        log.error("异常类型:"+e);
        return R.error();
    }
}
```
