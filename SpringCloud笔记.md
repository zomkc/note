# 实用部分

## Eureka注册中心

在Eureka框架中,微服务角色有两类

##### EurekaServer: 服务端,注册中心

* 记录服务信息
* 心跳监控

##### EurekaClient: 客户端

* Provider: 服务提供者
  * 注册自己的信息到EurekaServer
  * 每隔30秒向EurekaServer发送心跳

* consumer: 服务消费者
  * 根据服务名称从EurekaServer拉去服务列表
  * 基于服务列表做负载均衡,选中一个微服务后发起远程调用

#### 搭建EurekaServer服务

1.创建项目,引入spring-cloud-starter-netflix-eureka-server的依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

2.编写启动类,添加@EnableEurekaServer注解

3.添加application.yml文件,编写下面的配置

```yaml
server:
  port: 10086 #服务端口
spring:
  application:
    name: eurekaserver # eureka的服务名称
eureka:
  client:
    service-url:  #eureka的地址信息,=
      defaultZone: http://127.0.0.1:10086/eureka
```



##### 服务注册

1.在项目引入spring-cloud-starter-netflix-eureka-client依赖

```xml
<!--eureka客户端依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

2.在application.yml文件,编辑下面的配置

```yaml
spring:
  application:
    name: orderserver # eureka的服务名称
eureka:
  client:
    service-url:  #eureka的地址信息,=
      defaultZone: http://127.0.0.1:10086/eureka 
```



##### 服务发现

1.修改访问的url路径,用服务名代替ip,端口:

```java
String url = "http://userserver/user/" + order.getUserId();
```

2.在项目的启动类中的RestTemplate添加负载均衡注解

```java
//http请求
@Bean
@LoadBalanced
public RestTemplate restTemplate(){
    return new RestTemplate();
}
```



## Ribbon负载均衡

##### 负载均衡策略

![QQ截图20221024185225](SpringCloud%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20221024185225.png)

通过定义IRule实现可以修改负载均衡规则

1.代码方式: 在配置类中定义一个新的IRule(全局,调用此服务都会遵循)

```java
//修改负载均衡策略为随机
@Bean
public IRule randomRule(){
    return new RandomRule();
}
```

2.配置文件方式: 在yml文件中.添加新的配置也可以修改规则(仅在当前服务生效)

```yaml
userservice: #服务名称
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule # 负载均衡规则
```

##### 饥饿加载

Ribbon默认是采用懒加载,即第一次访问时才会去创建LoadBalanceClient,请求时间会长一些,

而饥饿加载则会在项目启动时创建,降低第一次访问的耗时,通过下面的配置开启饥饿模式

```yaml
ribbon:
  eager-load:
    enabled: true #开启饥饿加载
    clients:
      -userservice #指定对userservice这个服务饥饿加载
```



## Nacos注册中心

Nacos服务搭建

下载安装包

解压

在bin目录下运行指令: `startup.cmd -m standalone`

##### 服务注册到Nacos

1.在cloud父工程中添加spring-cloud-alibaba-dependencies的管理依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-dependencies</artifactId>
    <version>2.2.5.RELEASE</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

2.注释掉原有的eureka依赖

3.添加nacos的客户端依赖

````xml
<!-- nacos客户端依赖包 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
````

6.启动类加上`@EnableDiscoveryClient`开启nacos服务注册

5.注册到nacos

```yaml
spring:
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #老版本cloud写法
      config:
        server-addr: localhost:8848
  application:
    name: zmarket-member
```



##### Nacos服务分级储存模型

1.一级是服务,列如userservice

2.二级是集群,列如杭州上海

3.三级是实例,列如杭州机房的某台部署了userservice的服务器

##### 服务器集群属性

```yaml
spring:
  cloud:
    nacos:
      server-addr: localhost:8848 #nacos服务地址
      discovery:
        cluster-name: SH #配置集群名称,也就是机房位置
```

配置优先访问本地集群

```yaml
userservice:
  ribbon:
    NFLoadBalancerRuleClassName: com.alibaba.cloud.nacos.ribbon.NacosRule # 负载均衡规则
```



##### 实例的权重控制

1.Nacos控制台可以设置实例的权重值,0~1之间

2.同集群内的多个实例,权重越高被访问的频率越高

3.权重设置为0则不会被访问



##### Nacos环境隔离

1.namespace用来做环境隔离

```
cloud:
  nacos:
    server-addr: localhost:8848 #nacos服务地址
    discovery:
      cluster-name: SH #配置集群名称,也就是机房位置
      namespace: 185edccb-e42b-4721-b3b6-4a064ce795c4 #指定命名空间,命名空间id  nacos控制台创建命名空间后复制
```

2.每个namespace都有唯一的id

3.不同namespace下的服务不可见



##### 临时实例和非临时实例

服务注册到Nacos时,可以选择注册为临时或非临时实例,通过下面的配置来设置

```yaml
cloud:
  nacos:
    discovery:
      ephemeral: false #是否是临时实例
```

临时实例宕机时,会从nacos的服务列表中剔除,而非临时实例则不会



## Nacos配置管理

##### 统一配置管理

1.在Nacos中添加配置文件

2.引入Nacos的配置客户端依赖

```xml
<!--nacos配置管理依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

3.在微服务的resource目录添加一个bootstrap.yml文件,这个文件时引导文件,优先级高于application.yml

```yaml
spring:
  application:
    name: userservice #服务名称
  profiles:
    active: dev #开发环境配置
  cloud:
    nacos:
      server-addr: localhost:8848 #Nacos地址
      config:
        file-extension: yaml #文件后缀名
        namespace: 548ed73c-f3f4-4227-b799-56e4006e8ab9 #指定命名空间
        group: DEFAULT_GROUP #指定
```

4.在nacos配置列表添加userservice.dev.yaml配置文件

```yaml
zomkc:
  name: 张三
```

​	读取数据:

```java
@Value("${zomkc.name}")
private String name;
```



##### 配置自动刷新

Nacos中的配置文件变更后,微服务无需重启就可以感知,不过需要通过下面两种配置实现

方式1: 在@Value注入的变量所在的类上添加注解@RefreshScope

```java
@RestController
@RequestMapping("/user")
@RefreshScope
public class UserController {

    //注入nacos中的配置属性
    @Value("${pattern.dateformat}")
    private String dateformat;
```

方式2:使用@ConfigurationProperties注解

```java
@Component
@Data
@ConfigurationProperties(prefix = "pattern")
public class PatternProperties {
    private String dateformat;
}
```

多种配置的优先级

服务名-profile.yaml	>	服务名称.yaml	>	本地配置



##### nacos基本概念

1.**命名空间**:	配置隔离

​	用于进行租户粒度的配置隔离,不同的命名空间下,可以存在相同的Group或Data ID的配置,Namespace的常用场景之一是不同环境的配置的分区隔离,例如开发测试环境和生产环境的资源 (如配置,服务) 隔离等

```properties
spring.cloud.nacos.config.namespace=空间id #指定命名空间
```

2.**配置集**:

3.**配置集ID**:

4.**配置分组**:

```properties
spring.cloud.nacos.config.group=dev #指定分组
```



## http客户端Feign

##### 定义和使用Feign客户端

1.引入依赖

```xml
<!--Feign客户端-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

2.在启动类添加注解开启Feign的功能		@EnableFeignClients

```java
@MapperScan("cn.itcast.order.mapper")
@SpringBootApplication
@EnableFeignClients
public class OrderApplication {
```

3.编写Feign客户端

```java
@FeignClient("userservice")
public interface UserClient {   
    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);   
}
```

​	调用:

```java
    @Autowired
    private interface interface;

    @Test
    public void test(){
        System.out.println(interface.findById());
    }
```

主要是基于SpringMVC的注解来声明远程调用的信息,比如:

服务名称:	userservice

请求方式:	GET

请求路径:	/user/{id}

请求参数:	Long id

返回值类型:	User



##### 自定义Feign的配置

Feign运行自定义配置来覆盖默认配置,可以修改的配置如下

`feign.Logger.Level`		修改日志级别		包含四种不同的级别: NONE,BASIC,HEADERS,FULL

`feign.codec.Decoder`		响应结果的解析器		http远程调用的结果做解析,列如解析json字符串为java对象

`feign.code.Encoder`		请求参数编码		将请求参数编码,便于通过http请求发送

`feign.Contract`		支持的注解格式		默认是SpringMVC的注解

`feign.Retryer`		失败重试机制		请求失败的重试机制,默认是没有的,不过会使用Rebbon的重试

一般我们需要配置的就是日志级别



##### 配置Feign日志的两种方式

方式1:	配置文件方式

​	1.全局生效

```yaml
feign:
  client:
    config:
      default:  #这里的default就是全局配置,如果是写服务名称,则针对某个微服务的配置
        loggerLevel: FULL #日志级别
```

​	2.局部生效

```yaml
feign:
  client:
    config:
      userservice:
        loggerLevel: FULL #日志级别
```

方式2:	java代码方式,首先需要声明一个Bean:

```java
public class FeignClientConfiguration {
    @Bean
    public Logger.Level feignLogLevel(){
        return Logger.Level.BASIC;
    }
}
```

​	1.如果是全局配置,则把它放到@EnableFeignClients这个注解中

```java
@EnableFeignClients(defaultConfiguration = FeignClientConfiguration.class)
```

​	2.如果是局部配置,则把它放到@FeignClient这个注解中

```java
@FeignClient(value = "userservice",configuration = FeignClientConfiguration.class)
```

##### Feign的性能优化

Feign底层的客户端实现

URLConnection:	默认实现,不支持连接池

Apache	HttpClient:	支持连接池

OKHttp:	支持连接池



因此优化Feign的性能主要包括:

1.使用连接池代替默认的URLConnectionn

2.日志级别,最好使用basic或none



Feign添加HttpClient的支持

```xml
<!--httpclient依赖-->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```

配置连接池

```yaml
feign:
  httpclient:
    enabled: true #开启
    max-connections: 200 #最大的连接数
    max-connections-per-route: 50 #每个路径的最大连接数
```

##### Feign的最佳实践

1.让controller和FeignClient继承同一接口

2.将FeignClient,实体类,配置类的默认配置都定义到一个项目中,供所有消费者使用



当定义的FeignClient不再SpringBootApplication的扫描包范围时,这些FeignClient无法使用,有两种方法解决

方式1:	指定FeignClient所在包

```java
@EnableFeignClients(basePackages = "cn.itcast.feign.client")
```

方式2:	指定FeignClient字节码

```java
@EnableFeignClients(clients = {UserClient.class})
```





## 统一网关Gateway

##### 网关功能:

* 对用户的请求做身份认证,权限校验
* 将用户的请求路由到微服务,并实现负载均衡
* 对用户请求做限流

在springcloud中网关的实现包括两种

* gateway
* zuul

Zull是基于Servlet的实现,属于阻塞式编程,而SpringCloud则是基于spring5中提供的WebFlux,属于响应式编程的实现,具备更好的性能



##### 搭建网关服务

1.创建新的模块,引入PringCloudGateway的依赖和nacos的服务发现依赖

```xml
        <!-- nacos客户端依赖包 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--网关gateway依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
```

在启动类加上 `@EnableDiscoveryClient`

```java
@EnableDiscoveryClient //开启服务的注册发现
```

2.编写路由配置及nacos地址	手打,带空格编译不过

```yaml
server:
  port: 10010 #网关端口
spring:
  application:
    name: gateway #服务名称
  cloud:
    nacos:
      server-addr: localhost:8848 #nacos地址
    gateway:
      routes: #网关路由配置
        - id: user-service #路由id,自定义,只要唯一即可
#          uri: http://127.0.0.1:8081 #路由的目标地址,http就是固定地址
          uri: lb://userservice #路由的目标地址, lb就是负载均衡,后面跟服务名称
          predicates: #路由断言,也就是判断请求是否符合路由规则的条件
            - Path=/user/** #这个是按照路径匹配,只要以/user/开头就符合要求,跳转去路由的目标地址
        - id: order-service
          uri: lb://orderserver
          predicates:
            - Path=/order/**
            - Query=url,baidu #请求参数:url=baidu,就去路由的目标地址
```

##### 网关路由可以配置的内容包括

路由id:	路由唯一标识

uri:	路由目的地,支持lb和http两种

predicates:	路由断言,判断请求是否符合要求,符合则	转发到路由目的地

filters: 路由过滤器,处理请求或响应



##### 路由断言工厂 Route Predicate Factory

Predicate Factory的作用: 读取用户定义的断言条件,对请求做出判断

spring提供了11种基本的Predicate工厂:

![QQ截图20221026103555](SpringCloud%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20221026103555.png)



##### 路由过滤器 GatewayFilter

GatewayFilter是网关中提供的一种过滤器,可以对进入网关的请求和微服务返回的响应做处理

过滤器工厂	GatewayFilterFactory

spring提供了31种不同的路由过滤器工厂,列如:

![QQ截图20221026104708](SpringCloud%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20221026104708.png)

详细看spring cloud官方文档

案例:

```yaml
spring:
  application:
    name: gateway #服务名称
  cloud:
    nacos:
      server-addr: localhost:8848 #nacos地址
    gateway:
      routes: #网关路由配置
        - id: user-service #路由id,自定义,只要唯一即可
#          uri: http://127.0.0.1:8081 #路由的目标地址,http就是固定地址
          uri: lb://userservice #路由的目标地址, lb就是负载均衡,后面跟服务名称
          predicates: #路由断言,也就是判断请求是否符合路由规则的条件
            - Path=/user/** #这个是按照路径匹配,只要以/user/开头就符合要求
        - id: order-service
          uri: lb://orderserver
          predicates:
            - Path=/order/**
      default-filters: #默认过滤器,会对所有的路由请求都生效
        - AddrequestHeader=Truth, I am request header!  #给所有请求添加请求头
```



##### 全局过滤器 GlobalFilter

全局过滤器的作用:	对所有路由都生效,并且可以自定义处理逻辑

实现全局过滤器:

1.实现GlobalFilter接口

```java
@Order(-1)
@Component
public class AuthorizeFilter implements GlobalFilter {
    /*
        处理当前请求,有必要的话通过{@like GatewayFilterChain}将请求交给下一个过滤器处理
        exchange: 请求上下文,里面可以获取Request,Response等信息
        chain: 用来把请求委托给下一个过滤器
        {@code Mono<Void>} 返回标示当前过滤器业务结束
     */
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        //1.获取请求参数
        ServerHttpRequest request = exchange.getRequest();
        MultiValueMap<String, String> queryParams = request.getQueryParams();
        //2.获取参数中的 authorization 参数
        String auth = queryParams.getFirst("authorization");
        //3.判断参数值是否等于 admin
        if ("admin".equals(auth)){
            //4.是,放行
            return chain.filter(exchange);
        }
        //5.否,拦截
        //设置状态码
        exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
        //拦截请求
        return exchange.getResponse().setComplete();
    }
}
```

2.添加@Order注解或者实现Ordered接口

3.编写处理逻辑



##### 过滤器执行顺序

每一个过滤器都必须指定一个int类型的order值,order值越小,优先级越高,执行顺序越靠前

GlobalFilter通过实现Ordered接口,或者添加@Order注解来指定order值,由我们自己指定

路由过滤器和defaultFilter的order有spring指定,默认是按声明顺序从1递增

当过滤器的order值一样时,会按照 defaultFilter > 路由过滤器 > GlobalFilter的顺序执行

![QQ截图20221026120231](SpringCloud%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20221026120231.png)



##### 网关的cors跨域配置

```yaml
spring:
  application:
    name: gateway #服务名称
  cloud:
    nacos:
      server-addr: localhost:8848 #nacos地址
    gateway:
      routes:
        - id: renren-fast
          uri: lb://renren-fast
          predicates:
            - Path=/api/**
          filters:
            - RewritePath=/api/(?<segment>.*),/renren-fast/$\{segment} #/api/ 重写 /renren-fast/

      globalcors: # 全局的跨域处理
        add-to-simple-url-handler-mapping: true # 解决options请求被拦截问题
        corsConfigurations:
          '[/**]':
            allowedOrigins: # 允许哪些网站的跨域请求
              - "http://localhost:8090"
            allowedMethods: # 允许的跨域ajax的请求方式
              - "GET"
              - "POST"
              - "DELETE"
              - "PUT"
              - "OPTIONS"
            allowedHeaders: "*" # 允许在请求中携带的头信息
            allowCredentials: true # 是否允许携带cookie
            maxAge: 360000 # 这次跨域检测的有效期
```

​	bean方式

```java
    @Bean
    public CorsWebFilter corsWebFilter() {
        UrlBasedCorsConfigurationSource source =
                new UrlBasedCorsConfigurationSource();
        CorsConfiguration configuration = new CorsConfiguration();
        //配置跨域 Access-Control-Allow-Origin
        configuration.addAllowedHeader("*");
        //允许哪些请求来源跨域
        configuration.addAllowedOrigin("http://localhost:8001");
        //允许哪些请求跨域
        configuration.addAllowedMethod("*");
        //是否允许携带cookie跨域
        configuration.setAllowCredentials(true);
        //任意路径都进行跨域配置,跨域配置
        source.registerCorsConfiguration("/**", configuration);
        return new CorsWebFilter(source);
    }
```





## MQ - 异步通信

![QQ截图20221101083207](SpringCloud%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20221101083207.png)



##### RabbitMQ

RabbitMQ中的几个概念:

channel:	操作mq的工具

exchange:	路由消息到队列中

queue:	缓存消息

virtual host:	虚拟主机,是对queue,exchange等资源的逻辑分组



#### SpringAMQP

```xml
<!--AMQP依赖，包含RabbitMQ-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

在application.yml,添加mq连接信息

```yaml
spring:
  rabbitmq:
    host: 192.168.200.130 #rabbitMQ的ip地址
    port: 5672 #端口
    username: itcast
    password: 123321
    virtual-host: / #虚拟主机
```

测试:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringAmqpTest {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Test
    public void test1(){
        String queueName = "simple.queue";
        String message = "hello, spring amqp";
        rabbitTemplate.convertAndSend(queueName,message);
    }
```

接收消息:

```java
@Component
public class SpringRabbitListener {

    @RabbitListener(queues = "simple.queue")
    public void list(String msg){
        System.out.println("获取到消息:"+msg);
    }

}
```



消费预取限制:

设置preFetch这个值,可以控制预取消息的上限

```yaml
spring:
  rabbitmq:
    host: 192.168.200.130 #rabbitMQ的ip地址
    port: 5672 #端口
    username: itcast
    password: 123321
    virtual-host: /
    listener:
      simple:
        prefetch: 1 #每次只能获取一条消息,处理完成才能获取下一个消息
```



##### 交换机

交换机的作用:

​	接收publisher发送的消息

​	将消息按照规则路由到与之绑定的队列

​	不能缓存消息,路由失败,消息丢失

​	FanoutExchange会将消息路由到每个绑定的队列





发布订阅 -FanoutExchange:

FanoutExchange会将接收到的消息路由到每一个跟其绑定的queue

```java
@Configuration
public class FanoutConfig {
    //声明FanoutExchange交换机
    @Bean
    public FanoutExchange fanoutExchange(){
        return new FanoutExchange("itcast.fanout");
    }

    //声明第一个队列
    @Bean
    public Queue queue1(){
        return new Queue("fanout.queue1");
    }

    //绑定队列1到交换机
    @Bean
    public Binding bindingQueue1(Queue queue1,FanoutExchange fanoutExchange){
        return BindingBuilder.bind(queue1).to(fanoutExchange).with("direct");
    }
    }
```

```java
@Test
public void testFanoutExchange(){
    //队列名称
    String exchangeName = "itcast.fanout";
    //消息
    String message = "hello, everyone";
    //发送消息,参数分别是: 交换机名称,RoutingKey,消息
    rabbitTemplate.convertAndSend(exchangeName,"direct",message);
}
```



发布订阅 -DirectExchange

DirectExchange会将接收到的消息根据规则路由到指定的Queue,因此称为路由模式(routes)

```java
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(name = "direct.queue1"),
        exchange = @Exchange(name = "zomkc.direct"),
        key = {"red","blue"}
))
public void DirectQueue1(String msg){
    System.out.println("消费者1接收到Direct消息:"+msg);
}

@RabbitListener(bindings = @QueueBinding(
        value = @Queue(name = "direct.queue2"),
        exchange = @Exchange(name = "zomkc.direct",type = ExchangeTypes.DIRECT),
        key = {"red","yellow"}
))
public void DirectQueue2(String msg){
    System.out.println("消费者2接收到Direct消息:"+msg);
}
```

```java
@Test
public void testDirectExchange(){
    //队列名称
    String exchangeName = "zomkc.direct";
    //消息
    String message = "hello, blue";
    //发送消息,参数分别是: 交换机名称,RoutingKey,消息
    rabbitTemplate.convertAndSend(exchangeName,"blue",message);
}
```



发布订阅 -TopicExchange

ToppicExchange与DirectExchange类似,区别在于routingKey必须是多个单词组成,并且以`.`分割

Queue与Exchange指定BindingKey时,可以使用通配符

`#`: 代表0或者多个单词

`*`: 代表一个单词

```java
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(name = "topic.queue1"),
        exchange = @Exchange(name = "zomkc.topic",type = ExchangeTypes.TOPIC),
        key = {"china.#"}
))
public void TopicQueue1(String msg){
    System.out.println("消费者1接收到Direct消息:"+msg);
}

@RabbitListener(bindings = @QueueBinding(
        value = @Queue(name = "topic.queue2"),
        exchange = @Exchange(name = "zomkc.topic",type = ExchangeTypes.TOPIC),
        key = {"#.news"}
))
public void TopicQueue2(String msg){
    System.out.println("消费者2接收到Direct消息:"+msg);
}
```

```java
@Test
public void testTopicExchange(){
    //队列名称
    String exchangeName = "zomkc.topic";
    //消息
    String message = "hello, topic";
    //发送消息,参数分别是: 交换机名称,RoutingKey,消息
    rabbitTemplate.convertAndSend(exchangeName,"china.news",message);
}
```



##### 消息转换器

spring的消息对象的处理是由org.springframework.amqp.support.converter.MessageConverter来处理的,而默认实现是SimpleMessageConverter,基于JDK的ObjectOutputStream完成系列化

如果需要修改只需要定义一个MessageConverter类型的Bean即可

使用JSON方式序列化

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

```java
@Bean
public MessageConverter messageConverter(){
    return new Jackson2JsonMessageConverter();
}
```





## Elasticsearch -es搜索引擎

#### 索引库操作

##### mapping属性

mapping是对索引库中文档的约束,常见的mapping属性包括:

* type: 字段数据类型,常见的简单类型有:
  * 字符串: text(可分词文本),keyword(精确值)
  * 数值: long,integer,short,byte,double,float
  * 布尔: boolean
  * 日期: date
  * 对象: object
* index: 是否创建索引,默认为true
* analyzer: 使用哪种分词器
* properties: 该字段的子字段



##### 创建索引库

```http
#创建索引库
PUT /zomkc
{
  "mappings": {
    "properties": {
      "info": {
        "type": "text",
        "analyzer": "ik_smart"
      },
      "email": {
        "type": "keyword",
        "index": false
      },
      "name": {
        "type": "object",
        "properties": {
          "firstName": {
            "type": "keyword"
          },
          "lastName": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

##### 查看索引库

GET	/索引库名

##### 删除索引库

DELETE	/索引库名

##### 修改索引库

索引库和mapping一旦创建无法修改,但是可以添加新字段

```http
PUT	/索引库名/_mapping
{
	"properties": {
			"新字段名": {
				"type": "integer"	
				}
		}
}
```



#### 文档操作

新增文档的DSL语法:

```http
POST	/索引库名/_doc/文档id
{
	"字段1": "值1",
	"字段2": "值2",
	"字段3": {
		"子字段1": "值3",
		"子字段2": "值4"
	}
}
```

查询文档语法:

```http
GET /索引库名/_doc/文档id
```

删除文档语法:

```http
DELETE	/索引库名/_doc/文档id
```

修改文档语法:

方式1:	全量修改,会删除旧文档,添加新文档

```http
PUT	/索引库名/_doc/文档id
{
	"字段1": "值1",
	"字段2": "值2",
	...
}
```

方式2:	增量修改,修改指定字段值

```http
POST	/索引库名/_update/文档id
{
	"doc": {
		"字段名": "新的值",
	}
}
```



### RestClient操作索引库

ES官方提供了各种不同语言的客户端,用来操作ES

1.引入es的RestHighLevelClient依赖

```xml
<!--elasticsearch-->
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
</dependency>
```

2.因为springboot默认的es版本是7.6.2,所以需要覆盖该版本

```xml
<properties>
    <java.version>1.8</java.version>
    <elasticsearch.version>7.12.1</elasticsearch.version>
</properties>
```

3.初始化RestHighLevelClient

```java
client = new RestHighLevelClient(RestClient.builder(
        HttpHost.create("http://192.168.200.130:9200")
));

//结束请求
client.close();
```



```java
private RestHighLevelClient client;

//新增索引库
@Test
void createHotelIndex() throws IOException {
    //1.创建Request对象
    CreateIndexRequest request = new CreateIndexRequest("hotel");
    //2.准备请求的参数: DSL语句
    request.source(MAPPING_TEMPLATE, XContentType.JSON);
    //3.发起请求
    client.indices().create(request, RequestOptions.DEFAULT);
}

//判断索引库是否存在
@Test
void ExistsHotelIndex() throws IOException {
    GetIndexRequest request = new GetIndexRequest("hotel");
    boolean exists = client.indices().exists(request, RequestOptions.DEFAULT);
    System.out.println(exists);
}
//删除索引库
@Test
void DeleteHotelIndex() throws IOException {
    DeleteIndexRequest request = new DeleteIndexRequest("hotel");
    client.indices().delete(request,RequestOptions.DEFAULT);
}
```



文档操作:

```java
//新增文档
@Test
void testInit() throws IOException {
    //根据id查询酒店数据
    Hotel byId = hotelService.getById(45870L);
    //酒店数据转换为文档类型
    HotelDoc doc = new HotelDoc(byId);

    //1.准备request对象
    IndexRequest request = new IndexRequest("hotel").id(doc.getId().toString());
    //2.准备Json文档
    request.source(JSON.toJSONString(doc),XContentType.JSON);
    //3.发送请求
    client.index(request,RequestOptions.DEFAULT);

}

//查询文档
 @Test
void GetDocumentByid() throws IOException {
        //1.创建request对象
        GetRequest request = new GetRequest("hotel","45870");
        //2.发送请求得到结果
        GetResponse response = client.get(request, RequestOptions.DEFAULT);
        //3.解析结果
        String json = response.getSourceAsString();

        HotelDoc jsonObject = JSON.parseObject(json,HotelDoc.class);
        System.out.println(jsonObject);
    }
```

修改文档数据的两种方式:

方式1: 全量更新,再次写入id一样的文档,会删除旧文档,添加新文档

方式2: 局部更新,只更新部分字段

```java
//修改文档
@Test
void testUpdateDocumentById() throws IOException {
    //1.创建request对象
    UpdateRequest request = new UpdateRequest("hotel","45870");
    //2.准备参数,没2个参数为队
    request.doc(
            "price",728,
            "starName","五星级"
    );
    //3.更新文档
    client.update(request,RequestOptions.DEFAULT);
}
```

```java
//删除文档
@Test
void DeleteDocumentById() throws IOException {
    //1.创建request对象
    DeleteRequest request = new DeleteRequest("hotel","45870");
    //2.删除文档
    client.delete(request,RequestOptions.DEFAULT);
}
```

```java
//批量添加文档
@Test
void BulkRequest() throws IOException {
    List<Hotel> list = hotelService.list();
    //1.创建request
    BulkRequest request = new BulkRequest();
    //2.准备参数,添加多个Request
    for (Hotel hotel : list) {
        HotelDoc hotelDoc = new HotelDoc(hotel);
        request.add(new IndexRequest("hotel")
                .id(hotelDoc.getId().toString())
                .source(JSON.toJSONString(hotelDoc),XContentType.JSON));
    }
    //3.发起请求
    client.bulk(request,RequestOptions.DEFAULT);
}
```



#### DSL查询语法

DSL	Query的分类

查询所有:	查询出所有数据,一般测试用,列如: math_all

全文检索(full text)查询:	利用分词器对用户输入内容分词,然后去倒排索引库中匹配,例如:

* math_query
* multi_math_query

精确查询:	根据精确词条值查找数据,一般是查找keyword,数值,日期,boolean等类型字段,列如:

* ids
* range
* term

地理(geo)查询:	根据经纬度查询,列如:

* geo_distance
* geo_bounding_box

复合(compound)查询:	复合查询可以将上述各种查询条件组合起来,合并查询条件,列如:

* bool
* function_score



查询的基本语法:

```http
GET	/indexName/_search
{
	"query": {
		"查询类型":{
			"查询条件": "条件值"
		}
	}
}
```



##### 全文检索查询

match查询:	全文检索查询的一种,会对用户输入内容分词,然后去倒排索引库检索

```http
GET	/indexName/_search
{
	"query": {
		"match": {
			"FIELD": "TEXT"
		}
	}
}
```

multi_match:	与match查询类似,只不过允许同时查询多个字段

```http
GET	/indexName/_search
{
	"query": {
		"multi_match": {
			"query": "TEXT",
			"fields": ["FIELD_1","字段_2"]
		}
	}
}
```



##### 精确查询

* term: 根据词条精确查询
* range: 根据值的范围查询

term查询:

```http
GET /indexName/_search
{
  "query": {
    "term": {
      "FIELD": {
        "value": "VALUE"
      }
    }
  }
}
```

range查询:

```http
GET /indexName/_search
{
  "query": {
    "range": {
      "FIELD": {
        "gte": 100,
        "lte": 300
      }
    }
  }
}
```



##### 地理查询

​	geo_bounding_box:	查询geo_point值落在某个矩形范围内的所有文档

```http
GET /indexName/_search
{
	"query": {
		"geo_bounding_box": {
			"FIELD": {
				"top_left": {
					"lat": 31.1,
					"lon": 121.5
				},
				"bottom_right": {
					"lat": 30.9,
					"lon": 121.7
				}
			}
		}
	}
}
```

​	geo_distance: 查询到指定中心点小于某个值的所有文档

```http
GET /indexName/_search
{
  "query": {
    "geo_distance": {
      "distance":"15km",
      "FIELD": "31.21, 121.5"
    }
  }
}
```



##### 复合查询

复合查询可以将其他简单查询组合起来,实现更复杂的搜索逻辑

`function_score`

![QQ截图20221103105354](SpringCloud%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20221103105354.png)

```http
GET /hotel/_search
{
  "query": {
    "function_score": {
      "query": {"match_all": {}},
      "functions": [
        {
          "filter": {
            "match": {
              "brand": "如家"
            }
          },
          "weight": 10
        }
      ],
      "boost_mode": "sum"
    }
  }
}
```



`Boolean`

布尔查询是一个或多个子句的组合,子查询的组合方式有:

* must: 必须匹配每个子查询,类似"与"
* should: 选择性匹配子查询,类似"或"
* must_not: 必须不匹配.不参与算分,类似"非"
* filter: 必须匹配,不参与算分

```http
GET /hotel/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "name": "如家"
          }
        }
      ],
      "must_not": [
        {
          "range": {
            "price": {
              "gt": 400
            }
          }
        }
      ],
      "filter": [
        {
          "geo_distance": {
            "distance": "10km",
            "location": {
              "lat": 31.21,
              "lon": 121.5
            }
          }
        }
      ]
    }
  }
}
```



##### 排序

对搜索结果排序,默认是根据相关度算分( _sort)排序,,可以排序的字段有:

keyword类型,数值类型,地理坐标类型,日期类型等

```http
#sort排序
GET /hotel/_search
{
  "query": {
    "match_all": {}
  },
    "sort": [
      {
        "score": "desc",		#评分降序
        "price": "asc"			#价格升序
      }
    ]
}

GET /hotel/_search
{
  "query": {
    "match_all": {}
  },
    "sort": [
      {
        "_geo_distance": {
          "location": {
            "lat": 31,
            "lon": 121
          },
          "order": "asc",
          "unit": "km"
        }
      }
    ]
}
```



##### 分页

默认只返回top10的数据,如果要查询更多的数据,就需要修改分页参数了

通过修改from,size参数来控制要返回的分页结果

```http
#分页
GET /hotel/_search
{
  "query": {"match_all": {}},
  "from": 0,		#分页开始的位置,默认为0
  "size": 10		#期望获取的文档总数
}
```

针对深度分页,es提供了两种解决方案

search	after: 分页时需要排序,原理是从上一次的排序开始,查询下一页数据

scroll: 原理是排序数据形成快照,保存在内存,官方已经不推荐使用了



##### 高亮

高亮: 就是在搜索结果中把搜索关键字突出显示

原理是这样的:

* 将搜索结果中的关键字用标签记起来
* 在页面中给标签添加css样式

语法:

```http
默认情况下,es搜索字段必须与高亮字段一致
GET /hotel/_search
{
  "query": {
    "match": {
      "字段名": "TEXT"
    }
  },
  "highlight": {
    "fields": {		#指定要高亮的字段
      "字段名": {
        "pre_tags": "<em>",		#添加的前置标签,默认em
        "post_tags": "</em>"	#添加的后置标签
       "require_field_match": "false" #是否需要字段匹配,默认需要
      }
    }
  }
}
```



### RestClient查询文档

RestAPI中其中构建DSL是通过HighLevelRestClient中的resource()来实现,其中包含了查询,排序,分页,高亮等

RestAPI中其中构造查询条件的核心部分是由一个名为QueryBuilders的工具类提供



查询的基本步骤:

1.创建SearchRequest对象

2.准备request.source(),也就是DSL

1. QueryBuilders	来构建查询条件
2. 传入request.source()的query()方法

```java
@Test
void MatchTest() throws IOException {
    //1.准备request
    SearchRequest request = new SearchRequest("hotel");
    //2.组织DSL参数
    request.source()
            .query(QueryBuilders.matchAllQuery());
    //3.发送请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    //4.解析结果
    System.out.println("共查询到"+response.getHits().getTotalHits().value+"条信息");
    SearchHits hits = response.getHits();
    SearchHit[] searchHits = hits.getHits();
    for (SearchHit searchHit : searchHits) {
        //获取文档source
        String string = searchHit.getSourceAsString();
        //反序列化
        HotelDoc hotelDoc = JSON.parseObject(string, HotelDoc.class);
        System.out.println(hotelDoc);
    }
}
```

全文检索查询:

​		match和multi查询的api

```java
//单字段查询
QueryBuilders.matchQuery("all","如家")
//多字段查询
QueryBuilders.multiMatchQuery("如家","name","business")  
```

精确查询:

​		term查询和range查询

```java
//词条查询
QueryBuilders.termQuery("city","杭州")
//范围查询
QueryBuilders.rangeQuery("price").gte(100).lte(150)    
```

复合查询:

```java
//创建布尔查询
BoolQueryBuilder boolQuery =  QueryBuilders.boolQuery();
//must条件
boolQuery.must(QueryBuilders.termQuery("name","北京"));
//添加filter条件
boolQuery.filter(QueryBuilders.rangeQuery("price").lte(250));
//添加布尔查询条件到request
request.source().query(boolQuery);
```

排序和分页:

```java
//页码
int from = 2,size = 5;
request.source().
    query(boolQuery)	//查询条件
    .from((from-1) * size).size(size)	//分页
    .sort("price", SortOrder.ASC);	//价格排序
```

高亮:

```java
@Test
void highlightTest() throws IOException {
    //1.准备request
    SearchRequest request = new SearchRequest("hotel");
    //2.组织DSL参数
    request.source().query(QueryBuilders.termQuery("city","上海"));
    //高亮
    request.source().highlighter(new HighlightBuilder()
            .field("name")
            .requireFieldMatch(false)); //是否需要字段匹配
    //3.发送请求
    SearchResponse response = client.search(request, RequestOptions.DEFAULT);
    //4.解析结果
    SearchHits hits = response.getHits();
    SearchHit[] searchHits = hits.getHits();
    for (SearchHit searchHit : searchHits) {
        //获取文档source
        String string = searchHit.getSourceAsString();
        //反序列化
        HotelDoc hotelDoc = JSON.parseObject(string, HotelDoc.class);
        //获取高亮结果
        Map<String, HighlightField> highlightFields = searchHit.getHighlightFields();
        if (!CollectionUtils.isEmpty(highlightFields)){ //判断map不为空才执行覆盖操作
                //根据字段名获取高亮结果
                HighlightField highlightField = highlightFields.get("name");
                if (highlightField != null){
                    //获取高亮值
                    String name = highlightField.getFragments()[0].string();
                    //将高亮结果赋值给源数据
                    hotelDoc.setName(name);
                }
        }
        System.out.println(hotelDoc);
    }
}
```

距离排序:

```java
request.source().sort(SortBuilders
        .geoDistanceSort("location",new GeoPoint("31,121"))
        .order(SortOrder.ASC)
        .unit(DistanceUnit.KILOMETERS)
  );
```

复合查询:

![QQ截图20221107102357](SpringCloud%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20221107102357.png)







## 聚合

##### aggs代表聚合,与query同级

此时query的作用是:

* 限定搜索结果

集合必须的三要素

* 聚合名称
* 集合类型
* 聚合字段

集合可配置的属性:

* size: 指定聚合结果数量
* order: 指定聚合结果排序方式
* field: 指定聚合字段

```http
GET /hotel/_search
{
  "size": 0,	//设置size为0,结果中不包含文档,只包含聚合结果
  "aggs": {		//定义聚合
    "brandAgg": {	//给聚合取名字
      "terms": {	//聚合的类型,搜索类型
        "field": "brand.keyword",	//参与聚合的字段,可指定类型
        "size": 10,			//希望获取的聚合结果
        "order": {
          "_count": "asc"	//排序规则
        }
      }
    }
  }
}
```

##### Metrics聚合:

```http
GET /hotel/_search
{
  "size": 0,
  "aggs": {
    "brandAgg": {
      "terms": {
        "field": "brand.keyword",
        "size": 10,
        "order": {
          "scoreAggs.avg": "desc" //排序条件
        }
      },
      "aggs": {		//brands聚合的子聚合,也就是分组后对每组分别计算
        "scoreAggs": {	//聚合名称
          "stats": {	//聚合类型,这里stats可以计算min,max,avg
            "field": "score" //聚合字段
          }
        }
      }
    }
  }
}
```

### RestAPI实现聚合

```java
//1.准备request
SearchRequest request = new SearchRequest("hotel");
//2.组织DSL参数
request.source().size(0);
request.source().aggregation(AggregationBuilders
        .terms("brand_aggs")
        .field("brand.keyword")
        .size(20)
);
//3.发送请求
SearchResponse response = client.search(request, RequestOptions.DEFAULT);
//4.解析结果
//获取聚合结果
Aggregations aggregations = response.getAggregations();
//根据名称获取聚合结果
Terms brandThread = aggregations.get("brand_aggs");
//获取桶
List<? extends Terms.Bucket> buckets = brandThread.getBuckets();
  for (Terms.Bucket bucket : buckets) {
     System.out.println(bucket.getKeyAsString());
     }
```



## 自动补全





## 数据同步

方式1:	同步调用

* 优点:实现简单,粗暴
* 缺点:业务耦合度高

方式2:	异步通知

* 优点:低耦合,实现难度一般
* 缺点:依赖mq的可靠性

方式3:	监听binlog

* 优点:完全解除服务间耦合
* 缺点:开启binlog增加数据库负担,实现复杂度高



## es集群





# 高级部分



## 微服务保护	-Sentinel

解决雪崩问题常见方式:

* 超时处理: 设定超时时间,请求超过一定时间没有响应就返回错误信息,不会无休止等待
* 舱壁模式: 限定每个业务能够使用的线程数,避免耗尽整个tomcat资源,因此也叫线程隔离
* 熔断降级: 由断路器统计业务执行的异常比例,如果超出阈值则会熔断该业务,拦截访问该业务的一切请求
* 流量处理: 限制业务访问的QPS,避免服务因流量的突增而故障



##### 安装Sentinel控制台

sentinel官方提供了ui控制台

启动jar包,访问localhost:8080

配置项:

server.port	配置服务端口

sentinel.dashboard.auth.username	默认用户名

sentinel.dashboard.auth.password	默认密码



##### 微服务整合Sentinel

1,引入sentinel依赖:

```xml
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
```

2.配置控制台地址

```yaml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8090
```

3.访问微服务任意端点,触发sentinel监控



蔟点链路:

​	就是项目内的调用链路,链路中被监控的每一个接口就是一个资源,默认情况下sentinel会监控SpringMVC的每一个端点(Endpoint),因此SpringMVC的每一个端点就是调用链路中的一个资源.

​	流控,熔断等都是针对蔟点链路中的资源来设置,因此我们可以点击对应资源后面的按钮来设置规则



点击资源后面的流控按钮,就可以弹出表单,表单中可以添加流控规则

![QQ截图20230207143335](SpringCloud%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20230207143335.png)

其含义是限制 /order/{orderId} 这个资源的单机QPS为1,即每秒只允许一次访问,超出的请求会被拦截并报错



##### 流控模式

在添加限流规则是,点击高级选项,可以选择三种流控模式:

* 直接: 统计当前资源的请求,触发阈值时对当前资源直接限流,也是默认模式

* 关联: 统计与当前资源相关的另一个资源,另一个资源触发阈值时,对当前资源限流
* 链路: 统计从指定链路访问到本资源的请求,触发阈值时,对指定链路限流

**流控模式-关联:**	比如用户支付时需要修改订单状态,同时用户要查询订单.查询和修改操作要争抢数据库锁,产生竞争.业务需求是优先支付和更新订单的业务,因此当修改订单业务触发阈值时,需要对查询订单业务进行限流

满足下面条件可以使用关联模式:

* 两个有竞争关系的资源
* 一个优先级较高,一个优先级较低

**流控模式-链路**:	

Sentinel默认只标记Controller中的方法作为资源,如果要标记其他方法,需要使用`@SentinelResource`

```java
    @SentinelResource("goods")
    public void queryGoods(){
        System.out.println("一键三连...");
    }
```

Sentinel默认会将Controller方法作context整合,导致链路模式的流控失效,需要修改配置文件

```yaml
spring:
  cloud:
    sentinel:
      web-context-unify: false #关闭context整合
```



##### 流控效果

流控效果是指请求达到流控阈值时应该采取的措施,包括三种:

* **快速失败**:  达到阈值后,新的请求会会被立即拒绝并抛出FlowException异常,是默认的处理方式
* **warm up**: 预热模式,对超出阈值的请求同样是拒绝并抛出异常,但这种模式阈值会动态变化,从一个较小值增加到最大阈值
* **排队等待**: 让所有请求按照先后次序排队执行,两个请求的间隔不能大于指定时长

**流控效果-warm up:**

​	warm up也叫预热模式,是应对服务冷启动的一种方案. 请求阈值初始值是 threshold / coldFactor,持续指定时长后,逐渐提高到threshold值. 而coldFactor的默认值是3

​	列如, 设置QPS的threshold为10,预热时间为5秒,那么初始阈值是10/3,也就是3,然后在5秒后逐渐增长到10

**流控效果-排队等待:**

​	当请求超过QPS阈值时,快速失败和warm up会拒绝新的请求并抛出异常.而排队等待是让所有的请求进入一个队列中,然后按照阈值允许的时间间隔依次执行.后来的请求必须等待前面的执行完成,如果请求预期的等待时间超过最大时长,则会被拒绝

​	例如: QPS=5,意味着每200ms处理一个队列中的请求,timeout=2000,意味着预期等待时间超过2000ms会被拒绝并抛出异常



##### 热点参数限流

前面的限流是统计访问某个资源的所有请求,判断是否超过QPS阈值.而热点限流是分别统计参数值相同的请求,判断是否超过QPS阈值![QQ截图20230208091701](SpringCloud%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20230208091701.png)

​	在热点参数限流的高级选项中,可以对部分参数设置例外配置:

![QQ截图20230208091939](SpringCloud%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20230208091939.png)

**注意:**	热点参数限流对默认的SpringMVC资源无效,需要使用@SentinelResource指定



### 隔离和降级

​	虽然限流可以尽量避免高并发而引起的服务故障,但服务还会因为其他原因故障,而要将这些故障控制在一定范围,避免雪崩,就要靠线程隔离(舱壁模式)和熔断降级手段

​	不管是线程隔离还是熔断降级,都是对**客户端(调用方)**的保护



##### Feign整合Sentinel

SpringCloud中,微服务调用都是通过Feign来实现的,因此做客户端保护必须整合Feign和Sentinel

1.修改OrderService的yml文件,开启Feign的Sentinel功能

```yaml
feign:
  sentinel:
    enabled: true #开启Feign的Sentinel功能
```

2.给FeignClient编写失败后的降级逻辑

* 方式一: FallbackClass,无法对远程调用的异常做处理
* 方式二: FallbackFactory,可以对远程调用的异常做处理,我们选择这种



**步骤一:** 实现FallbackFactory

```java
@Slf4j
public class UserClientFallbackFactory implements FallbackFactory<UserClient> {
    @Override
    public UserClient create(Throwable throwable) {
        //创建UserClient接口实现类,实现其中的方法,编写失败降级的处理逻辑
        return new UserClient() {
            @Override
            public User findById(Long id) {
                //记录异常信息
                log.error("查询用户失败",throwable);
                return new User();
            }
        };
    }
}
```

**步骤二:** 在Config类中将UserClientFallbackFactory注册为一个Bean

```java
    @Bean
    public UserClientFallbackFactory userClientFallbackFactory(){
        return new UserClientFallbackFactory();
    }
```

**步骤三:** 在UserClient接口中使用UserClientFallbackFactory

```java
@FeignClient(value = "userservice",fallbackFactory = UserClientFallbackFactory.class)
public interface UserClient {

    @GetMapping("/user/{id}")
    User findById(@PathVariable("id") Long id);

}
```



##### 线程隔离

* 线程池隔离

  优点: 支持主动超时 支持异步调用

  缺点: 线程的额外开销比较大

  场景: 低扇出

* 信号量隔离(Sentinel默认采用)

  优点: 轻量级,无额外开销

  缺点: 不支持主动超时 不支持异步调用

  场景: 高频调用 高扇出

在添加限流规则时,可以选择两种阈值类型:

* QPS: 就是每秒的请求数
* 线程数: 是该资源能使用的tomcat线程的最大值,也就是通过限制线程数,实现**舱壁模式**

##### 熔断降级

​	熔断降级是解决雪崩问题的重要手段.其思路是由**断路器**统计服务调用的异常比例、慢请求比例,如果超出阈值则会**熔断**该服务。即拦截该服务的一切请求;而当服务恢复时,断路器会放行访问该服务的请求

![QQ截图20230208145037](SpringCloud%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20230208145037.png)

断路器熔断策略有三种: 慢调用、异常比例、异常数

**熔断策略-慢调用:**

​	慢调用: 业务的响应时长(RT)大于指定时长的请求认定为慢调用请求。如果请求数量超过设定的最小数量,慢调用比例大于设定的阈值,则触发熔断

![QQ截图20230208150404](SpringCloud%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20230208150404.png)

​	RT超过500ms的调用是慢调用,统计最近10000ms内的请求,如果请求量超过10次,并且慢调用比例不低于0.5,则触发熔断,熔断时长为5秒,然后进入half-open状态,放行一次请求做测试

**熔断策略-异常比例、异常数**

​	异常比例或者异常数: 统计指定时间内的调用,如果调用次数超过指定请求数,并且出现异常的比例超过设定的比例阈值(或超过指定异常数),则触发熔断

![QQ截图20230208152553](SpringCloud%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20230208152553.png)

​	统计最近1000ms内的请求,如果请求超过10次,并且异常比例不低于0.4,则触发熔断,熔断时间为5秒,然后进入half-open状态,放行一次请求做测试。



### 授权规则

授权规则可以对调用方的来源做控制,有白名单和黑名单两种方式

* 白名单: 来源(origin) 在白名单内的调用者允许访问
* 黑名单: 来源在黑名单的调用者不允许访问

![QQ截图20230208160330](SpringCloud%E7%AC%94%E8%AE%B0.assets/QQ%E6%88%AA%E5%9B%BE20230208160330.png)

例如,我们限定只允许从网关来的请求访问order-server,那么流控应用中就填写网关的名称

Sentinel是通过RequestOriginParser这个接口的parseOrigin来获取请求来源

```java
public interface RequestOriginParser {
    //从请求request对象获取origin,获取需要方式自定义
    String parseOrigin(HttpServletRequest request);
}
```

例如,我们尝试从request中获取一个名为origin的请求头,作为origin的值

```java
@Component
public class HeaderOriginParser implements RequestOriginParser {
    @Override
    public String parseOrigin(HttpServletRequest request){
        String origin = request.getHeader("origin");
        if(StringUtils.isEmpty(origin)){
            //请求头为空返回blank
            return "blank";
        }
        return origin;
    }
}
```

​	在网关服务中,利用过滤器添加名为gateway的origin头

```yaml
spring:
  cloud:
    gateway:
      default-filters:
        - AddrequestHeader=origin, gateway  #给所有请求添加请求头
```



##### 自定义异常结果

默认情况下,发生限流、降级、授权拦截时,都会抛出异常到调用方。如果要自定义异常时的返回结果,需要实现BlockExceptionHandler接口

```java
public interface BlockExceptionHandler {
    //处理请求被限流,降级,授权拦截时抛出的异常: BlockException
    viod handle(HttpServletRequest request,
                HttpServletResponse response,
                BlockException e)throws Exception;
}
```

BlockException包含很多子类,分别应对不同的场景

* FlowException	限流异常
* ParamFlowException   热点参数限流的异常
* DegradeException   降级异常
* AuthorityException   授权规则异常
* SystemBlockException   系统规则异常

```java
@Component
public class SentinelExceptionHandler implements BlockExceptionHandler {
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response, BlockException e) throws Exception {
        String msg = "未知异常";
        int status = 429;

        if (e instanceof FlowException) {
            msg = "请求被限流了";
        } else if (e instanceof ParamFlowException) {
            msg = "请求被热点参数限流";
        } else if (e instanceof DegradeException) {
            msg = "请求被降级了";
        } else if (e instanceof AuthorityException) {
            msg = "没有权限访问";
            status = 401;
        }

        response.setContentType("application/json;charset=utf-8");
        response.setStatus(status);
        response.getWriter().println("{\"msg\": " + msg + ", \"status\": " + status + "}");
    }
}
```



##### 规则管理模式

Sentinel的控制台规则管理有三种模式

* 原始模式: Sentinel的默认模式,将规则保存在内存中,重启服务会丢失
* pull模式: 保存在本地文件或数据库,定时去读取
* push模式: 保存在nacos,监听变更实时更新

**规则管理模式-pull模式:**

​	pull模式: 控制台将配置的规则推送到Sentinel客户端,而客户端会将配置规则保存在本地文件或数据库中,以后会定时去本地文件或数据库中查询,更新本地规则

**规则管理模式-push模式:**

​	push模式: 控制台将配置规则推送到远程配置中心,例如nacos。Sentinel客户端监听nacos,获取配置变更的推送消息,完成本地配置更新



### Sentinel 规则持久化

#### 一、修改order-service服务

修改OrderService，让其监听Nacos中的sentinel规则配置。

具体步骤如下：

##### 1.引入依赖

在order-service中引入sentinel监听nacos的依赖：

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```



##### 2.配置nacos地址

在order-service中的application.yml文件配置nacos地址及监听的配置信息：

```yaml
spring:
  cloud:
    sentinel:
      datasource:
        flow:
          nacos:
            server-addr: localhost:8848 # nacos地址
            dataId: orderservice-flow-rules
            groupId: SENTINEL_GROUP
            rule-type: flow # 还可以是：degrade、authority、param-flow
```





#### 二、修改sentinel-dashboard源码

SentinelDashboard默认不支持nacos的持久化，需要修改源码。

##### 1. 解压

解压课前资料中的sentinel源码包：

![image-20210618201340086](SpringCloud%E7%AC%94%E8%AE%B0.assets/image-20210618201340086.png)

然后并用IDEA打开这个项目，结构如下：

![image-20210618201412878](SpringCloud%E7%AC%94%E8%AE%B0.assets/image-20210618201412878.png)

##### 2. 修改nacos依赖

在sentinel-dashboard源码的pom文件中，nacos的依赖默认的scope是test，只能在测试时使用，这里要去除：

![image-20210618201607831](SpringCloud%E7%AC%94%E8%AE%B0.assets/image-20210618201607831.png)

将sentinel-datasource-nacos依赖的scope去掉：

```xml
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
```



##### 3. 添加nacos支持

在sentinel-dashboard的test包下，已经编写了对nacos的支持，我们需要将其拷贝到main下。

![image-20210618201726280](SpringCloud%E7%AC%94%E8%AE%B0.assets/image-20210618201726280.png)



##### 4. 修改nacos地址

然后，还需要修改测试代码中的NacosConfig类：

![image-20210618201912078](SpringCloud%E7%AC%94%E8%AE%B0.assets/image-20210618201912078.png)

修改其中的nacos地址，让其读取application.properties中的配置：

![image-20210618202047575](SpringCloud%E7%AC%94%E8%AE%B0.assets/image-20210618202047575.png)

在sentinel-dashboard的application.properties中添加nacos地址配置：

```properties
nacos.addr=localhost:8848
```



##### 5. 配置nacos数据源

另外，还需要修改com.alibaba.csp.sentinel.dashboard.controller.v2包下的FlowControllerV2类：

![image-20210618202322301](SpringCloud%E7%AC%94%E8%AE%B0.assets/image-20210618202322301.png)

让我们添加的Nacos数据源生效：

![image-20210618202334536](SpringCloud%E7%AC%94%E8%AE%B0.assets/image-20210618202334536.png)



##### 6. 修改前端页面

接下来，还要修改前端页面，添加一个支持nacos的菜单。

修改src/main/webapp/resources/app/scripts/directives/sidebar/目录下的sidebar.html文件：

![image-20210618202433356](SpringCloud%E7%AC%94%E8%AE%B0.assets/image-20210618202433356.png)



将其中的这部分注释打开：

![image-20210618202449881](SpringCloud%E7%AC%94%E8%AE%B0.assets/image-20210618202449881.png)



修改其中的文本：

![image-20210618202501928](SpringCloud%E7%AC%94%E8%AE%B0.assets/image-20210618202501928.png)



##### 7. 重新编译、打包项目

运行IDEA中的maven插件，编译和打包修改好的Sentinel-Dashboard：

![image-20210618202701492](SpringCloud%E7%AC%94%E8%AE%B0.assets/image-20210618202701492.png)



##### 8.启动

启动方式跟官方一样：

```sh
java -jar sentinel-dashboard.jar
```

如果要修改nacos地址，需要添加参数：

```sh
java -jar -Dnacos.addr=localhost:8848 sentinel-dashboard.jar
```





## 分布式事务	-seata

分布式服务的事务问题:

​	在分布式系统下,一个业务跨越多个服务或数据源,每个服务都是一个分支事务,要保证所有分支事务最终状态一致,这样的事务就是**分布式事务**

**CAP定理:**	1998年,加州大学的计算机科学家 Eric Brewer 提出,分布式系统有三个指标

* Consistency (一致性): 用户访问分布式系统中的任意节点,得到的数据必须一致
* Availability (可用性): 用户访问集群中的任意健康节点,必须得到响应,而不是超时拒绝
* Partition tolerance (分区容错性): 因为网络故障或其他原因导致分布式系统中的部分节点与其他节点失去连接,形成独立分区。在集群出现分区时,整个系统也要持续对外提供服务

Eric Brewer说:分布式系统无法同时满足这三个指标,这个结论就叫CAP定理



**BASE理论:**	BASE理论是对CAP的一种解决思路,包含三个思想

* Basically Available (基本可用): 分布式系统在出现故障时,允许损失部分可用性,即保证核心可用
* Soft State (软状态): 在一定时间内,允许出现中间状态,比如临时的不一致状态
* Eventually Consistent (最终一致性): 虽然无法保证强一致性,但是在软状态结束后,最终达到数据一致

​	解决分布式事务,各个子系统之间必须能感知到彼此的事务状态,才能保持状态一致,因此需要一个事务协调者来协调每一个事务的参与者(子系统事务)

这里的子系统事务,称为分支事务,有关联的各个分支事务在一起称为全局事务



#### 微服务集成seata

1.引入依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
    <exclusions>
        <!--版本较低，1.3.0，因此排除-->
        <exclusion>
            <artifactId>seata-spring-boot-starter</artifactId>
            <groupId>io.seata</groupId>
        </exclusion>
    </exclusions>
</dependency>
<!--seata starter 采用1.4.2版本-->
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-spring-boot-starter</artifactId>
    <version>${seata.version}</version>
</dependency>
```

2.修改配置文件

```yaml
seata:
  registry: # TC服务注册中心的配置，微服务根据这些信息去注册中心获取tc服务地址
    # 参考tc服务自己的registry.conf中的配置
    type: nacos
    nacos: # tc
      server-addr: 127.0.0.1:8848
      namespace: ""
      group: DEFAULT_GROUP
      application: seata-tc-server # tc服务在nacos中的服务名称
      cluster: SH
  tx-service-group: seata-demo # 事务组，根据这个获取tc服务的cluster名称
  service:
    vgroup-mapping: # 事务组与TC服务cluster的映射关系
      seata-demo: SH
```



## 

## alicloud OSS	阿里对象存储

1.引入依赖

```xml
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alicloud-oss</artifactId>
        </dependency>
```

2.配置

```yaml
spring:
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    alicloud:
      access-key: LTAI5tPUgq9Zw4DecUSK6r7y
      secret-key: ObYRrzCgykyQGhjiPvIMdaXQ4LflAw
      oss:
        endpoint: oss-cn-chengdu.aliyuncs.com
```

3.使用OSSClient	的API进行相关操作

