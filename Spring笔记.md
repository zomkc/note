# Spring笔记

xml:	spring依赖maven坐标

```xml
            <dependency>
                <groupId>org.springframework</groupId>
                <artifactId>spring-context</artifactId>
                <version>5.2.14.RELEASE</version>
            </dependency>
```



spring核心技术:	ioc	,	aop	能够实现模块之间,类之间的解耦合

## spring的第一个核心功能: ioc

Ioc(Inversion of Control):控制反转,是一个理论,概念,思想

描述的:	把对象的创建,赋值,管理工作都交给代码之外的容器实现,也就是对象的创建是由其他外部资源实现的

控制:创建对象,对象的属性赋值,对象之间的关系管理

反转:把原来的开发人员管理,创建对象的权限转移给代码之外的容器实现,由容器代替开发人员管理对象,创建对象,给属性赋值

正转:由开发人员在代码中,使用new 构造方法创建对象,开发人员主动管理对象

容器:是一个服务器软件,一个框架(spring)

为什么使用ioc: 目的是减少对代码的改动,也能实现不同的功能,实现解耦合

## Ioc的技术实现:

DI是ioc的技术实现

DI(Dependency Injection):依赖注入,只需要在程序中提供要使用程序的对象名称就行,至于对象如何在容器中创建,赋值,查找都由容器内部实现

spring是使用di实现ioc功能,spring底层创建对象,使用的是反射机制

spring是一个容器,管理对象,给属性赋值,底层是反射创建对象



### spring的配置文件:

1.beans:是跟标签,spring把java对象称为bean

​		声明bean,就是告诉spring要创建某个类的对象

bean属性:

​	id:	对象的自定义名称,唯一值,spring通过这个名称找到对象

​	name: 别名,可以有多个,用 `,`或`;`或`空格`分隔

​	class:	类的全限定名称(不是接口,因为spring是反射机制创建对象必须使用类)

​	scope: 定义bean的作用范围,`singleton`:单例(默认),`prototype`:非单例

​			适合交给容器进行管理的bean

* 表现层对象
* 业务层对象
* 数据层对象
* 工具对象

​			不适合交给容器进行管理的bean

* 封装实体的域对象

2.spring-beans.sd	是约束文件,跟mybatis中的dtd是一样的

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    
    <bean id="someService" class="org.example.cervice.impl.SomeServiceImpl"/>
</beans>
```



```java
//ApplicationContext就是表示spring容器,通过容器获取对象
//ClassPathXmlApplicationContext表示从类路径中加载spring的配置文件
//1.获取Ioc容器
ApplicationContext app = new ClassPathXmlApplicationContext("beans.xml");
//2.获取Bean
SomeService some = (SomeService) app.getBean("someService");
//调用bean对象的方法
some.doSome();
```

spring默认创建对象的时间:	在创建spring容器时,会创建配置文件中的所有的对象

spring创建对象:	默认调用的是无参数构造方法

容器中对象的数量:` int getBeanDefinitionCount()`

容器中每个定义的对象的名称:`String[] getBeanDefinitionNames()`

实例化bean的三种方式

* 构造方法(常用)
* 静态工厂(了解)
* 实例工厂(了解)
  * FactoryBean(实用)



### Di:依赖注入,表示创建对象,给属性赋值

di的实现有两种:

1.在spring的配置文件中

2.使用spring中的注解,完成属性赋值,叫做基于注解的di实现



#### di语法分类:

##### 1.`setter注入`(设置注入):

spring调用类的`set`方法,在set方法可以实现属性的赋值

简单类型的set注入

```xml
<bean id="xx" class="xxx">

<property name="name" value="李四"></property>
<property name="age" value="20"></property>

</bean>
```



引用类型的set注入:	spring调用类的set方法

```xml
<bean id="xx" class="xxx">

<property name="属性名称xx" ref="bean的id(对象的名称xx)"></property>

</bean>
```



##### 2.`构造注入`

spring调用类的有参数构造方法,创建对象,在构造方法中完成赋值

构造注入使用`<constructor-arg>`标签

`<constructor-arg>`标签属性:

name:表示构造方法的形参名

index:表示构造方法的参数的位置,参数从左往右位置是0,1,2的顺序

value:构造方法的形参类型是简单类型的,使用value

ref:构造方法的形参类型是引用类型,使用ref

```java
public class School {
    private String school;
    private String dz;
	//构造方法注入
    public School(String school, String dz) {
        this.school = school;
        this.dz = dz;
    }
}
```

```xml
    <bean id="school" class="org.example.ba01.School">
       <constructor-arg name="school" value="王五"/>
       <constructor-arg name="dz" value="成都"/>
    </bean>
```



##### 3.引用类型的自动注入:

spring框架根据某些规则可以给引用类型赋值,不用你给引用类型赋值了

1.`byName`(按名称注入):	不常用

java类中引用类型的属性名和spring容器中(配置文件)<bean>的id名称一样,且属性类型是一致的,这样的容器中的bean,spring能够赋值给引用类型

语法:

```xml
    <bean name="studentService" class="org.example.cervice.impl.StudentServiceImpl"/>

    <bean id="someService" class="org.example.cervice.impl.SomeServiceImpl" autowire="byName"/>
```

2.`byType`(按类型注入):

java类中引用类型的数据类型和spring容器中(配置文件)<bean>的class属性是同源关系的,这样的bean能赋值给引用类型

```java
public class SomeServiceImpl implements SomeService {

    private StudentServiceImpl studentService;
    @Override
    public void doSome() {
        studentService.save();
        System.out.println("Some Service");
    }

    public void setStudentService(StudentServiceImpl studentService) {
        this.studentService = studentService;
    }
}
```

```xml
    <bean class="org.example.cervice.impl.StudentServiceImpl"/>

    <bean id="someService" class="org.example.cervice.impl.SomeServiceImpl" autowire="byType"/>
```

注意:在byType中,在xml配置文件中声明bean只能有一个符合条件的,多于一个是错误的



##### 集合注入

```xml
    <bean id="arrayDemo" class="org.example.ArrayDemo">
        <property name="arrays">
            <array>
                <value>100</value>
                <value>200</value>
                <value>300</value>
            </array>
        </property>
    </bean>

```

数组,List,Set,Map,properties





包含关系的配置文件:

主配置文件,包含其他的配置文件,主配置文件一般不定义对象的

语法:`<import reource="其他配置文件的路径">`

关键字:"classpath:"表示类路径(class文件所在的目录)

在spring的配置文件中要指定其他文件的位置,需要classpath,告诉spring要去哪去加载读取文件

在包含关系的配置文件中,可以通配符( * : 表示任意字符)

注意:主的配置文件名称不能包含在通配符的范围内





## spring注解:

#### 使用@Component定义Bean

属性:	value就是对象的名称,也就是bean的id值

value值是唯一的,创建的对象在整个spring容器中就一个

位置:	在类的上面

#### 指定注解在你的项目中的包名

声明组件扫描器(component-scan),组件就是java对象

component-scan工作方式:	spring会扫描遍历base-package指定的包,

把包中和子包中的所有类,找到类中的注解,按照注解的功能创建对象,给属性赋值

`<context:component-scan	base-package="包路径"/>`

使用配置类代替xml配置文件

```java
@Configuration//设置当前类为配置;类
@ComponentScan("com.cn")//用于设定扫描路径,次注解只能添加一次,多个数据用数组格式
//@ComponentScan({"com.cn.service","com.cn.dao"})
public class SpringConfig{}
```

配置类的方式加载容器

```java
//加载配置类初始化容器
ApplicationContext ctx = new AnnotationConfigApplicationContext(SpringConfig.class);
```



@Import: 在主配置文件导入其他(jdbc)配置文件

#### spring中和@component功能一致的,创建对象的注解还有:

1.`@Repository`(用在持久层类上面):	放在dao的实现类上,表示创建dao对象,dao对象是能访问数据库的

2.`@Service`(用在业务层类的上面):	放在service的实现类上面,创建Service对象,service对象是做业务处理,可以有事务等功能

3.`@Controller`(用在控制器的上面):	放在控制器(处理器)类的上面,创建控制器对象的,控制器对象,能够接受用户提交的参数,显示请求的处理结果

以上三个注解的使用语法和@Component一样的,都能创建对象,但是这三个注解都有额外的功能

@Repository,@Service,@Controller是给项目的对象分层的



Bean的作用范围

```java
@Repository
//使用@Scope定义Bean作用范围
@Scope("singleton")
public class BookDaoImpl{}
```

Bean生命周期	使用@PostConstruct,@PreDestroy定义生命周期

```java
@Repository
//使用@Scope定义Bean作用范围
@Scope("singleton")
public class BookDaoImpl{
    @PostConstruct
    public void init(){}
    @PreDestroy
    public void destroy(){}
}
```





#### 自动装配

##### 使用@PropertySource注解加载properties文件

```java
@Configuration
@ComponentScan("com.cn")
@PropertySource("classpath:jdbc.properties")//classpath:可以省略
public class SpringConfig{}
```

使用@Value("@{变量值}")赋值配置文件属性



##### @Value:	简单类型的属性赋值

属性:	value 是String类型的,	表示简单类型的属性值

位置:

* 1.在属性定义的上面,无需set方法,推荐使用
* 2.在set方法上面



##### 引用类型	@Autowired

spring框架提供的注解,实现引用类型的赋值

spring中通过注解给引用类型赋值,使用的是自动注入的原理,支持byName,byType

@Autowired:	默认使用byType自动注入

属性:	required,是一个Boolean类型的,默认true

required=true:	表示引用类型赋值失败,程序报错,并终止运行

required=true:	表示引用类型赋值失败,程序正常运行,引用类型是null



如果要使用byName方式,在属性上面加入`@Qualifier(value="bean的id")`:表示使用指定的名称的bean完成赋值



##### 引用类型	@Resource

来自jdk中的注解,spring框架对这个注解的功能支持,可以使用它给引用类型赋值

使用的也是自动注入的原理,支持bName,byType,默认是byName

默认是byName,先使用byName自动注入,如果byName赋值失败,再使用byType



##### xml对比注解配置

![QQ截图20221229174904](Spring%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20221229174904-16723073616142.png)



## AOP

![](Spring%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20220421095458.png)



动态代理:可以在程序的执行过程中,创建代理对象

通过代理对象执行方法,给目标类的方法增加额外的功能(功能增强)

1.动态代理实现方式:

jdk动态代理,使用jdk中的Proxy,Method,IovocaitonHanderl创建代理对象

cglib动态代理:	第三方的工具库,创建代理对象,原理是继承,通过继承目标类,创建子类,子类就是代理对象,要求目标类不能是final的,方法也不能是final的

2.动态代理的作用:

​	1.在目标类源代码不改变的情况下,增加功能

​	2.减少代码的重复

​	3.专注业务逻辑代码

​	4.解耦合,让你的业务功能和日志,事务非业务功能分离



3.Aop:面向切面编程,基于动态代理的,可以使用jdk,cglib两种代理方式

Aop就是动态代理的规范化,把动态代理的实现步骤,方式都定义好了,让开发人员用一种统一的方式,使用动态代理



4.Aop(Aspect Orient Programming):面向切面编程

![](Spring%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20220420084813.png)



##### 5.Aop的实现

AOP是一个规范,是动态的一个规范化,一个标准

aop是技术实现框架

1.spring:	spring在内部实现了aop规范,能做aop的工作

​	spring主要在事务处理时使用aop

​	项目开发中很少使用spring的aop实现,因为很笨重

2.aspectJ:	一个开源的专门做aop的框架,spring框架中集成了aspectJ框架,通过spring就能使用aspectJ的功能

* 1.使用xml的配置文件:配置全局事务
* 2.使用注解,我们在项目中要做aop功能,一般使用注解,aspectJ有5个注解







## aspectJ框架的使用

```xml
      <!--spring依赖-->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>5.2.5.RELEASE</version>
    </dependency>
<!--  aspectJ依赖  -->
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aspects</artifactId>
      <version>5.2.5.RELEASE</version>
    </dependency>
```

```xml
<!--声明自动代理生成器
aspectj-autoproxy:会把spring容器中的所有的目标对象,一次性生成代理对象
-->
	<aop:aspectj-autoproxy/>

<!--如果期望目标类有接口,使用cglib代理
    proxy-target-class="true":  告诉框架,要使用cglib动态代理
-->
    <aop:aspectj-autoproxy proxy-target-class="true"/>
```



1.切面的执行时间,这个执行时间的规范中叫做Advice(通知,增强)

在aspectJ框架中使用注解表示的,也可以使用xml配置文件中的标签

* 1.@Before :前置通知

  ```java
  /*
  @Aspect:    是aspectJ框架中的注解
      作用:表示当前类是切面类
      切面类:是用来给业务方法增加功能的类,在这个类中由切面的功能代码
      位置:在类的定义上面
   */
  @Aspect
  public class MyAspect {
      /*
      定义方法,方法是实现切面功能的
      方法的定义要求
      1.公共方法public
      2.方法没有返回值
      3.方法名称自定义
      4.方法可以有参数,也可以没有参数
          如果有参数,参数不是自定义的,有几个参数类型可以使用
       */
  
      /**
       * @Before: 前置通知注解
       * 属性:value, 是切入点表达式,表示切面的功能执行的位置
       * 位置:在方法的上面
       * 特点:
       * 1.在目标方法之前1先执行的
       * 2.不会改变目标方法的执行结果
       * 3.不会影响目标方法的执行
       */
  //    @Before(value = "execution(public void org.ba01.SomeServiceImpl.doSome(String,Integer))")
      @Before("execution(* *..SomeServiceImpl.doSome(..))")
      public void myBefore(JoinPoint jp){
          //获取方法的完整定义
          System.out.println("方法的签名(定义)"+jp.getSignature());
          System.out.println("方法的名称"+jp.getSignature());
          //获取方法的实参
          Object[] args = jp.getArgs();
          for (Object arg:  args) {
              System.out.println("参数:"+arg);
          }
          //切面要执行的功能代码
          System.out.println("前置通知,切面功能:在目标方法之前输出时间"+new Date());
      }
  ```

  

* 2.@AfterReturning : 后置通知

  ```java
  /**
   *@AfterReturning:  后置通知
   *      属性:1.value  切入点表达式
   *      2.returning 自定义的变量,表示目标方法的返回值
   *      自定义变量名必须和通知方法的形参名一样
   *      位置:在方法定义的上面
   *  特点:
   *      1.在目标方法之后执行的
   *      2.能够获取到目标方法的返回值,可以根据这个返回值做不同的处理功能
   *      3.可以修改这个返回值
   */
  @AfterReturning(value = "execution(* *..SomeServiceImpl.doOther(..))",returning = "res")
  public void myafterReturing(Object res){
  //Object res:是目标方法执行后的返回值,根据返回值做切面的功能处理
      System.out.println("后置通知,返回值是:"+res);
  }
  ```

  

* 3.@Around: 环绕通知

  ```java
  @Aspect
  public class MyAspect {
  
  /**
   * 环绕通知方法的定义格式
   * 1.public
   * 2.必须有一个返回值,推荐使用Object
   * 3.方法名称自定义
   * 4.方法有参数,固定的参数    ProceedingJoinPoint
   */
  
      /**
       * @Around: 环绕通知
       * 属性:  value 切入点表达式
       * 位置:  在方法的定义上面
       * 特点:
       * 1.他是功能最强的通知
       * 2.在目标方法的前和后都能增强功能
       * 3.控制目标方法是否被调用执行
       * 4.修改用来的目标方法的执行结果,影响最后的调用结果
          环绕通知,等同于jdk动态代理,IovocationHandler接口
  
          参数: ProceedingJoinPoint 等同于 Method
  
          返回值:就是目标方法的执行结果,可以被修改
  
          环绕通知:   经常做事务,在目标方法之前开启事务,执行目标方法,在目标方法之后提交事务
       */
      @Around(value = "execution(* *..SomeServiceImpl.doFirst(..))")
      public Object  myAround(ProceedingJoinPoint pjp) throws Throwable {
          String name ="";
          //获取第一个参数值
          Object[] args = pjp.getArgs();
          if (args!=null && args.length>1){
              Object arg = args[0];
              name = (String) arg;
          }
          System.out.println(name);
  
          //实现环绕通知
          Object result = null;
          System.out.println("环绕通知:在目标方法之前,输出:"+new Date());
      //目标方法的调用
          if ("王五".equals(name)) {
              //名字为王五,符合条件值目标方法
              result = pjp.proceed();
          }
          System.out.println("==环绕通知.在目标方法之后,提交事务==");
  
          if (result!=null){
              result = "Hello AspectJ Aop";
          }
  
          return result;
      }
  }
  ```

* 4,AfterThrowing:   异常通知

* 5.@After:   最终通知

* 6.@Pointcut:    定义切入点





##### 切入点表达式:

AspectJ定义了专门的表达式用于切入点,表达式的原型是

```java
execution(modifiers-pattern?	ret-type-pattern)
    declaring-type-pattern?name-pattern(param-pattern)
    throws-pattern?)
```

解释:

`modifiers-pattern`:	访问权限类型

`ret-type-pattern`:	返回值类型

`declaring-type-pattern`:	包名类名

`name-pattern(param-patern)`:	方法名(参数类型和参数个数)

`throws-pattern:`	抛出异常类型

?表示可选部分



以上表达式4个部分

`execution(访问权限	方法返回值	方法声明(参数)	异常类型)`



切入点表达式要匹配的对象就是目标方法的方法名,所以,execution表达式中明显就是方法的签名

其中可以使用以下符号

`*`:	0值多个任意字符

`..`:	用在方法参数中,表示任意多个参数用在包名后,表示包及其子包路径

`+`:	用在类名后,表示当前类及其子类,用在接口后,表示当前接口及其实现类



举例:

`execution(public * *{..})`

指定切入点为:	任意公共方法

`execution(* set*(..))`

指定切入点为:	任何一个以"set"开始的方法

`execution(* com.xyz.service.*.*(..))`

指定切入点为:	定义在service包里任意类的任意方法

`execution(* com.xyz.service..*.*(..))`

指定切入点为:	定义在service包或子包里的任意类的任意方法,".."出现在类名中时,后面必须跟" * ", 表示包,子包下的所有类

`execution(* *..service.*.*(..))`

指定所有包下的service子包下所有类(接口)中所有方法为切入点



##### 指定通知方法中的参数:	JoinPoint

`JoinPoint`:	业务方法,要加入切片功能的业务方法

​	作用是:可以在通知方法中,获取方法执行时的信息,例如方法名称,方法参数

​	如果你的切片功能中需要用到方法的信息,就加入JoinPoint

​	这个Joinpoint参数的值是由框架赋予的,必须是第一个参数

```java
@Before("execution(* *..SomeServiceImpl.doSome(..))")
public void myBefore(JoinPoint jp){
    //获取方法的完整定义
    System.out.println("方法的签名(定义)"+jp.getSignature());
    System.out.println("方法的名称"+jp.getSignature());
    //获取方法的实参
    Object[] args = jp.getArgs();
    for (Object arg:  args) {
        System.out.println("参数:"+arg);
    }
    //切面要执行的功能代码
    System.out.println("前置通知,切面功能:在目标方法之前输出时间"+new Date());
}
```



##### cglib代理

```xml
<!--如果期望目标类有接口,使用cglib代理
    proxy-target-class="true":  告诉框架,要使用cglib动态代理
-->
    <aop:aspectj-autoproxy proxy-target-class="true"/>
```



Spring开启事务

1.在业务层接口上添加Spring事务管理

```java
public interface AccountService {
    @Transactionl
    public void transfer(String out,String in,Double money)
}
```

2.设置事务管理器

```java
@Bean
public PlatformTransactionManager transactionManager(Datasource datasource){
    DataSourceTransactionManager ptm = new DataSourceTransactionManager();
    ptm.setDataSource(datasource);
    return ptm;
}
```

3.开启注解式事务驱动

​		启动类上加入:	@EnableTransactionManagement 注解表明开启事务



事务相关配置	@Transactionl属性

![QQ截图20221230122528](Spring%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20221230122528.png)

事务传播行为

![QQ截图20221230123218](Spring%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20221230123218.png)







# SpringMVC笔记



```xml
<--servlet依赖-->
<dependency>
<groupId>javax.servlet</groupId>
<artifactId>javax.servlet-api</artifactId>
<version>3.1.0</version>
<scope>provided</scope>
</dependency>
    
<--springmvc依赖-->    
<dependency>
<groupId>org.springframework</groupId>
<artifactId>spring-webmvc</artifactId>
<version>5.2.5.RELEASE</version>
</dependency>
```

使用配置类代替SpringMVC.xml配置文件

```java
@Configuration
@ComponentScan("com.controller")
public class SpringMvcConfig {
}
```

tomcat会自动扫描继承该类的配置类,相当于web.xml

```java
public class ServletInitConfig extends AbstractDispatcherServletInitializer {
    //加载SpringMVC容器配置
    @Override
    protected WebApplicationContext createServletApplicationContext() {
        AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(SpringMvcConfig.class);
        return ctx;
    }
    //设置哪些请求归属MVC处理
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
    //加载Spring容器配置
    @Override
    protected WebApplicationContext createRootApplicationContext() {
               AnnotationConfigWebApplicationContext ctx = new AnnotationConfigWebApplicationContext();
        ctx.register(SpringConfig.class);
        return ctx;
    }
}
```

简化:

```java
public class ServletInitConfig extends AbstractAnnotationConfigDispatcherServletInitializer {
    //加载Spring容器配置
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }
    //加载SpringMVC容器配置
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{SpringMvcConfig.class};
    }
    //设置哪些请求归属MVC处理
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
    
    //设置过滤器,处理乱码
    @Override
    protected Filter[] getServletFilters() {
        CharacterEncodingFilter filter = new CharacterEncodingFilter();
        //只能解决post
        filter.setEncoding("UTF-8");
        return new Filter[]{filter};
    }
}
```



## 请求路径映射

@RequestMapping

类型:	方法注解,类注解

位置:	在SpringMVC控制器方法定义上方,或控制器类上方

属性: 	value(默认):	访问请求路径或访问路径前缀

```java
@Controller
@RequestMapping("/user")
public class MyController {
    @RequestMapping("/save")
    @ResponseBody//指定返回值作为视图
    public String save(){
        System.out.println("save...");
        return "{'name','zs'}";
    }
}
 /*  返回值: ModelAndView 表示本次请求的处理结果
 *      Model: 数据,请求处理完成后,要显示给用户的数据
 *      View: 视图,比如jsp等等
 */
    @RequestMapping(value = "/some.do",method = RequestMethod.GET)
    public ModelAndView doSome(){
        ModelAndView mv = new ModelAndView();

        mv.addObject("msg","欢迎使用springmvc做web开发");
        mv.addObject("fun","执行的是doSome方法");

        mv.setViewName("WEB-INF/view/show.jsp");

//        mv.setViewName("show");

        return mv;
    }
```

## 请求参数

@RequestParam

类型:	形参注解

位置:	springmvc控制器方法形参定义前面

作用:	绑定请求参数与处理器方法形参间的关系

```java
    @RequestMapping("/name")
    @ResponseBody
    public String name(@RequestParam("name") String name){
        System.out.println("======="+name);
        return name+"欢迎~";
    }
```

​	参数:

* required: 是否为必传参数
* defaultValue: 参数默认值



POJO(实体类)参数:	请求参数名与形参对象属性名相同,定义实体类形参即可接收参数

嵌套POJO参数:	请求参数名与形参对象属性名相同,按照对象层次结构关系即可(user.name=张三)

数组参数:	请求参数名与形参对象属性名相同且请求参数为多个,定义数组类型形参即可接收参数

集合保存普通参数:	请求参数名与形参集合对象名相同且请求参数为多个,使用@RequestParam绑定参数关系



##### 传递JSON数据

jackson坐标

```xml
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.9.0</version>
    </dependency>
```

@EnableWebMvc

类型:	配置类注解

位置:	springmvc配置类上

作用:	开启SpringMVC多项辅助功能

```java
@Configuration
@ComponentScan("com.controller")
@EnableWebMvc	//开启自动转换json数据的支持
public class SpringMvcConfig {
}
```

@RequestBody

类型:	形参注解

位置:	SpringMVC控制器方法形参定义前面

作用:	将请求中请求体所包含的数据传递给请求参数,此注解一个处理器方法只能使用一次

```java
    @RequestMapping("/json")
    @ResponseBody
    public String json(@RequestBody User user){
        System.out.println("======="+user);
        return user.getName()+"欢迎~";
    }
```

##### 日期类型参数传递

日期类型框架可以自动转换,前提是开启了mvc辅助功能

@DateTimeFormat

类型:	形参注解

位置:	SpringMVC控制器方法形参前面

作用:	设定日期时间型数据格式

```java
    @RequestMapping("/date")
    @ResponseBody
    public Date date(@DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") Date date){
        System.out.println("======="+date);
        return date;
    }
```

​	属性:

* pattern: 日期时间格式字符串



## 响应

@ResponseBody

类型:	方法注解

位置:	mvc控制器方法定义上方

作用:	设置当前控制器返回值作为响应体



##### 设置静态资源映射

```xml
<!--声明 springmvc框架中的  视图解析器   ,帮助开发人员设置视图文件的路径-->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
<!--前缀:视图文件的路径-->
        <property name="prefix" value="/WEB-INF/view/"/>
<!--后缀:视图文件的扩展名-->
        <property name="suffix" value=".jsp"/>
    </bean>
```

配置类的方式:

```java
@Configuration
public class WebMvcConfig extends WebMvcConfigurationSupport {
    //设置静态资源映射
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
       registry.addResourceHandler("/backend/**").addResourceLocations("classpath:/backend/");
    }
}
```





## REST风格

风格简介

按照REST风格访问资源时使用行为动作,区分对资源进行了何种操作

`http://localhost/users`			查询全部用户信息 GET (查询)

`http://localhost/users/1`		查询指定用户信息 GET (查询)

`http://localhost/users`			添加用户信息 POST (新增/保存)

 `http://localhost/users`			修改用户信息 PUT (修改/更新)

`http://localhost/users/1`		删除用户信息 DELETE (删除)

根据REST风格对资源进行访问称为RESTful



##### 注解:

​	`@RestController`: 复合注解,是@Controller和@ResponseBody组合

​	在类的上面使用RestController , 表示当前类所有方法都加入@ResponseBody



​	`@GetMappring `: 支持get请求方式,等同于

​								@RequestMapping(method=RequestMethod.GET)

​	`@PostMapping`: 支持post请求方式,等同于

​								@RequestMapping(method=RequestMethod.GET)

​	`@PutMapping`: 支持Put请求方式,等同于

​								@RequestMapping(method=RequestMethod.PUT)

​	`@DeleteMapping`: 支持Put请求方式,等同于

​								@RequestMapping(method=RequestMethod.DELETE)



​	`@PathVariable`: 从url中获取数据



```java
/*@PathVariable: (路径变量) 获取url中的数据
            属性: value : 路径变量名
            位置: 放在控制器方法的形参前面
      {stuid}: 自定义名称
      当路径变量名和形参名一样,value可以省略  
      */
@GetMapping("/student/{stuid}")
public String queryStudent(@PathVariable("stuid") Integer stuid){
    return "查询学生:"+stuid;
}
```





## SSM整合

* Spring:

  springConfig

* MyBatis

  MybatisConfig

  JdbcConfig

  jdbc.properties

* SpringMVC

  ServletConfig

  SpringMvcConfig

```xml
<!--servlet依赖-->
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>3.1.0</version>
      <scope>provided</scope>
    </dependency>
    [<!-- jsp依赖 -->]
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>jsp-api</artifactId>
      <version>2.2.1-b03</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
<!--spring和mvc依赖-->
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>5.2.5.RELEASE</version>
    </dependency>
	[<!--spring声明式事务-->]
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>5.2.5.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.2.5.RELEASE</version>
    </dependency>
	<!--json-->
    <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.9.0</version>
    </dependency>
<!--mybatis依赖-->
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.1</version>
    </dependency>
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.1</version>
    </dependency>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.28</version>
    </dependency>
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.12</version>
    </dependency>
```



##### 异常处理

异常处理器:	集中的,统一的处理项目中出现的异常

@RestControllerAdvice

名称:	类注解

位置:	Rest风格开发的控制器增强类定义上方

作用:	为Rest风格开发的控制器类做增强

说明:	此注解自带@ResponseBody注解与@Component注解,具备对应的功能

```java
@RestControllerAdvice
public class ProjectExceptionAdvice {
    @ExceptionHandler(Exception.class)
    public void doException(Exception ex){
        System.out.println(ex);
    }
}
```

@ExceptionHandler

类型:	方法注解

位置:	专用于异常处理的控制器方法上方

作用:	设置指定异常的处理方案,功能等同于控制器方法,出现异常后终止原始控制器执行,并转入当前方法执行



#### 拦截器

拦截器与过滤器的区别

归属不同:	Filter属于Servlet技术,Interceptor属于SpringMVC技术

拦截内容不同:	Filter对所有访问进行增强,Interceptor仅针对SpringMVC的访问进行增强

##### 1.声明拦截器的bean,并实现HandlerInterceptor接口

```java
//拦截器类:拦截用户的请求
@Component
public class MyInterceptor implements HandlerInterceptor {
    long btime;
    /*
    preHandle叫做预处理方法
    参数:
        Object handler: 被拦截的控制器对象
        返回值boolean
            true:表示通过了拦截器的验证,可以执行处理器方法
            false:请求没有通过拦截器的验证,请求到达拦截器就截止了,请求没有被处理

         特点:
            1方法在控制器方法之前先执行的
                用户的请求首先到达此方法

            2.在这个方法中可以获取请求的信息,验证请求是否符合要求
                可以验证用户是否登录,验证用户是否有权限访问某个连接地址(url)
                如果验证失败,可以截断请求,请求不能被处理
                如果验证成功,可以放行请求,此时控制器方法才能执行
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle执行了");

        btime = System.currentTimeMillis();
        //返回false,给浏览器一个返回结果
//        request.getRequestDispatcher("/tips.jsp").forward(request,response);
        return true;
    }

    /*
        postHandle:后处理方法
        参数:
        Object handler: 被拦截的控制器对象
        ModelAndView modelAndView:处理器方法的返回值

        特点:
        1.在处理器方法之后执行
        2.能够获取到处理器方法的返回值ModelAndView,可以修改ModelAndView中的
        数据和视图,可以影响到最后的执行结果
        3.主要是对原来的执行结果做二次修正
     */
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("拦截器拦获取到的数据"+modelAndView);
        //对原来的some执行结果调整
        if (modelAndView != null){
            //修改数据
            modelAndView.addObject("mytade",new Date());
            //修改视图
            modelAndView.setViewName("other");
        }
    }
    /*
        afterCompletion:最后执行的方法
        Object handler: 被拦截的控制器对象
        Exception ex: 程序中发生的异常
      特点:
        1.在请求处理完成后执行的,框架中规定是当你的视图处理完成后,对视图执行forward,
        就认为请求处理完成
        2.一般做资源回收工作的,程序请求过程中创建一些对象,可以在这里删除,把占用的内存回收
     */
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    long etime = System.currentTimeMillis();
        System.out.println("从preHandle到请求处理完成的时间:  "+(etime - btime));
    }
}
```

前置处理:

```java
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("preHandle执行了");
        return true;
    }
```

参数:

* request:	请求对象
* response:  响应对象
* handler:    被调用的处理器对象,本质上是一个方法对象,对反射技术中的Method对象进行了再包装

返回值:

* 返回false,备拦截的处理器将不执行

后置处理:

```java
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        System.out.println("拦截器拦获取到的数据"+modelAndView);
    }
```

* modelAndView:	如果处理器执行完成具有返回结果,可以读取对应数据与页面信息,并进行调整

完成后处理:

```java
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    long etime = System.currentTimeMillis();
        System.out.println("从preHandle到请求处理完成的时间:  "+(etime - btime));
    }
}
```

* ex:	如果处理器执行过程中出现异常对象,可以针对异常情况进行单独处理

##### 2.注册拦截器:

```xml
<!--声明拦截器:拦截器可以有0或者多个-->
<mvc:interceptors>
<!-- 声明第一个拦截器
    在框架中保存多个拦截器是ArrList
    按照声明的先后顺序放入到ArrList
-->
   <mvc:interceptor>
<!--指定拦截器的请求URI地址
    path:就是rui地址,可以使用通配符 **
    ** : 表示任意的字符,文件或者多级目录和目录中的文件   -->
   <mvc:mapping path="/user/**"/>
<!--声明拦截器对象-->
   <bean class="com.cn.handler.MyInterceptor"/>
   </mvc:interceptor>
</mvc:interceptors>
```

配置类的方式:			方法在MVC配置类也有

```java
@Configuration
public class WebMvcConfig extends WebMvcConfigurationSupport {
    @Autowired
    private MyInterceptor myInterceptor;

    //设置静态资源映射
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/backend/**").addResourceLocations("classpath:/backend/");
    }
    //注册拦截器
    @Override
    protected void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(myInterceptor).addPathPatterns("/books/*");
    }
}
```

拦截器链的运行顺序参照拦截器添加顺序为准
