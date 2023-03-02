# redis笔记

内容:

redis概念

命令操作,数据结构

持久化操作

使用java客户端操作redis





### Redis

##### 1.概念:redis是一款高性能的NOSQL系列的非关系型数据库

* redis.windows-service.conf : 配置文件
* redis-cli.exe : redis的客户端
* redis-server.exe : redis服务器端

##### 2.命令操作

1. redis的数据结构:

   redis存储的是: key,value格式的数据,其中key都是字符串,value有五种不同的数据结构

   value的数据结构:

   ​	1.字符串类型	string

   ​	2.哈希类型	hash : map格式

   ​	3.列表类型	list : linkedlist格式

   ​	4.集合类型	set

   ​	5.有序集合类型	sortedset

   

2. 字符串类型 string

   * 1.存储: `set key value`
   * 2.获取: `get key`
   * 3.删除: `del key`
   * 

3. 哈希类型 hash

   * 1.存储: `hset key value`

   * 2.获取:

     `hget key field`: 获取指定的field对应的值

     `hgetall key`: 获取所有的field和value

   * 3.删除: `hdel key field`

     

4. 列表类型 list： 可以添加一个元素到列表的头部（左边）或者尾部（右边）

   * 1.添加：

     `lpush key value`：将元素加入列表左边

     `rpush key value`：将元素加入列表右边

   * 2.获取：

     `lrange key start end`:范围获取

   * 3.删除:

     `lpop key`:删除列表最左边的元素,并将元素返回

     `rpop key`:删除列表最右边的元素,并将元素返回

     

   5.集合类型 set: 不允许重复元素

   * 1.储存: `sadd key value`
   * 2.获取: `smembers key` : 获取set集合中所有元素
   * 3.删除: `srem key value`: 删除set集合中的某个元素 
   * 

   6.有序集合类型 sortedset : 不允许重复元素,且元素有顺序

   * 1.存储: `zadd key score value`

   * 2.获取: `zrange key start end`

   * 3.删除: `zrem key value`

     

   7.通用命令

   * `keys *` : 查询所有的键

   * `type key` : 获取键对应的value的类型

   * `del key` : 删除指定的key   value

     

##### 3.持久化

1. redis是一个内存数据库,当redis服务器重启,获取电脑重启,数据后丢失,我们可以将redis内存中的数据持久化保存到硬盘的文件中

2. redis持久化机制:

   

   1.RDB:默认方式,不需要进行配置,默认就使用这种机制

   在一定的间隔时间中,检测key的变化情况,然后持久化数据

   

   在redis.windows.conf文件

   save 900 1

   save 300 10

   save 60 10000

   重启redis服务器,并指定配置文件

   

   2.AOF : 日志记录的方式,可以记录每一条命令的操作,可以每一次命令操作后,持久化数据

   编辑redis.windows.conf文件

   ​	appendonly no (关闭aof) --> appendonly yes (开启aof)

   

   #appendfsync always : 每一次操作都进行持久化

   appendfsync everysec : 每隔一秒进行一次持久化

   #appendfsync no : 不进行持久化



##### 4.Java客户端 Jedis

jedis:一款java操作redis数据库的工具

使用步骤:

```java
//1.获取连接
Jedis jedis = new Jedis("localhost",6379);
//2.操作
jedis.set("username","pipi");
//3.关闭连接
jedis.close();
```

可以使用setex()方法储存可以指定过期时间的key value

`jedis.setex("key",20,"value")`:将key:value键值对存入redis,并且20秒后自动删除该键值对



##### Jedis操作各种redis中的数据结构

1.字符串类型 string

set

get

2.哈希类型 hash : map格式

hset

hget

hgetAll

3.列表类型 list： 可以添加一个元素到列表的头部（左边）或者尾部（右边）

lpush	/	rpush

lpop	/	rpop

lrange	start	end	: 范围获取

4.集合类型 set: 不允许重复元素

sadd

smembers	: 获取所有元素

5.有序集合类型 sortedset : 不允许重复元素,且元素有顺序

zadd

zrange 	: 范围获取





jedis工具类

```java
package cn.Jedis;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

public class JedisPoolUtils {

    private static JedisPool jedisPool;

    static {
        //读取配置文件
        InputStream is = JedisPoolUtils.class.getClassLoader().getResourceAsStream("jedis.properties");
        //创建Properties对象
        Properties pro = new Properties();
        //关联文件
        try {
            pro.load(is);
        } catch (IOException e) {
            e.printStackTrace();
        }
        //获取数据,设置到JedisPoolConfig中
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(Integer.parseInt(pro.getProperty("maxTotal")));
        config.setMaxIdle(Integer.parseInt(pro.getProperty("maxIdle")));
        //初始化JedisPool
        jedisPool = new JedisPool(config,pro.getProperty("host"),Integer.parseInt(pro.getProperty("port")));
    }
    //获取连接方法
    public static Jedis getJedis(){
        return jedisPool.getResource();
    }
}
```

使用

```java
//通过连接池工具类获取
Jedis jedis = JedisPoolUtils.getJedis();

//使用
jedis.set("popo","heihei");

//关闭
jedis.close();
```

##### jedis的maven坐标

```xml
<dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>4.2.0</version>
        </dependency>
```





#### Springboot使用redis

##### 在项目导入redis的maven坐标

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

##### Spring Data Redis

Spring Data Redis中提供了一个高度封装的类: RedisTemplate,针对jedis客户端中大量的api进行了归类封装,将同一类型操作封装为operation接口

* ValueOperations: 简单K-V操作
* SetOperations: set类型数据操作
* ZSetOperations: zset类型数据操作
* HashOperations: 针对map类型的数据操作
* ListOperations: 针对list类型的数据操作





##### 在application.yml中加入redis相关配置

```yaml
spring:
  cache:
    redis:
      time-to-live: 18000 #设置缓存有效期
  redis:
    host: 127.0.0.1
    port: 6379
    password: root
    database: 0 #redis有16个数据库,默认0号
    jedis:
      #redis连接池配置
      pool:
        max-active: 8 #最大连接数
        max-wait: 1ms # 连接池最大阻塞等待时间
        max-idle: 4 #连接池中最大的空闲连接
        min-idle: 0 #连接池中最小的空闲连接
```





## Spring Cache

介绍:	Spring Cache是一个框架,实现了基于注解的缓存功能

Spring Cache提供了一层抽象,底层可以切换不同的Cache实现,

提供CacheManager接口来统一不同的缓存技术

CacheManager是spring提供的各种缓存技术抽象接口

* EhCacheCacheManager:	使用EhCahe作为缓存技术
* GuavaCacheManager:    使用Google的GuavaCache作为缓存技术
* RedisCacheManager:   使用Redis作为缓存技术

##### Soing Cache	常用注解

`@EnableCaching`:	开启缓存注解功能

`@Cacheable`:	在方法执行前spring先查看缓存中是否有数据,如果有数据,则直接返回缓存数据;	若没有数据,调用方法并将方法返回值放到缓存中

```java
@Cacheable(value = "userCache" , key = "#id" , condition ="#result != null")
	value: 缓存的名称,每个缓存名称下面可以有多个key
    key: 缓存的key
    condition: 条件; 满足条件才缓存数据
       unless: 满足条件不缓存数据      

@Cacheable({"category"})   // 代表当前方法的结果需要缓存，如果缓存中有，方法不用调用，如果缓存中没有，会调用方法，最后将方法的结果放入缓存   key默认自动生成，key的值为 “缓存的名称” + "：：" + “SimpleKey[]”         
```

`@CachePut`:	将方法的返回值放到缓存中,或者更新缓存

```java
@CachePUt(value = "userCache", key = "#user.id")
	value: 缓存的名称,每个缓存名称下面可以有多个key
    key: 缓存的key		(#p0.id)(#root.args[0].id)
```

`@CacheEvict`:	将一条或多条数据从缓存中删除

````java
@CacheEvict(value = "userCache", key = "#user.id")
	value: 缓存的名称,每个缓存名称下面可以有多个key
    key: 缓存的key		(#p0.id)(#root.args[0].id)
    
    allEntries = true: 删除当前属性下的所有数据    
````

`@Caching`：组合多个缓存操作

````java
该注解可以组合多个缓存操作。当我们执行更新操作的时候，需要更新多个缓存的时候，这时候就需要使用该注解
@Caching(evict = {
        @CacheEvict(value={"category"},key = "'getLevel1Categorys'"),
        @CacheEvict(value={"category"},key = "'getCatagoryJson'")
````

##### Spring Cache缓存的不足

对于读模式，sprig cache对缓存穿透、缓存击穿、缓存雪崩等场景都要考虑

1）缓存穿透：查询一个null数据，在SpringCache中，可以缓存空数据，通过配置spring.cache.redis.cache-null-values=true

2）缓存击穿：大量并发进来同时查询一个正好过期的数据。默认没有加锁，通过sync=true来解决

3）缓存雪崩：大量的key同时过期，指定过期时间spring.cache.redis.time-to-live=3600000来解决

但是对于写模式，没有过多考虑，所以，对于读多写少 即一致性要求不高的数据可以使用spring-cache，对于写比较多的场景，如果业务场景对一致性要求不高，只要缓存的数据设置了过期时间就足够了，对于那些对一致性要求比较高的数据，需要特殊设计




##### 使用Redis作为缓存技术,只需要导入Spring data Redis的maven坐标即可

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

