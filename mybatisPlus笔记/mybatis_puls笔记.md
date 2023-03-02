# MyBatis-Plus

springboot: 2.7.3

mybatis-plus: 3.5.1

##### 依赖: 

```xml
<!--mybatis-plus启动器-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.1</version>
        </dependency>
<!--lombok用于简化实体类开发-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    <!--mysql驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.29</version>
        </dependency>
```

##### 配置文件

```properties
#配置数据源
spring.datasource.type=com.zaxxer.hikari.HikariDataSource

spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis-plus?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=123456
mybatis-plus.configuration.log-impl=  #开启日志
```

##### 实体类

使用lombok

```java
/*
//lombok 编译后生成 无参构造
@NoArgsConstructor
//有参构造
@AllArgsConstructor
//get方法
@Getter
//set方法
@Setter
@EqualsAndHashCode */

//@Data相当于 无参+get+set+toString+equals+hashcode
@Data
public class User {
    private Long id;

    private String name;

    private Integer age;

    private String email;
}
```

##### mapper接口

继承BaseMapper,泛型写实体类

```java
//标识为持久层
@Repository
public interface UserMapper extends BaseMapper<User> {

}
```





## 基本功能

#### 通用mapper

```java
@SpringBootTest
public class MyBatisPlusTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testSelectList(){
        //通过条件构造器查询一个list集合,若没有条件,则可以设置null为参数
        List<User> list = userMapper.selectList(null);
        list.forEach(System.out::println);
    }

    @Test
    public void testInsert(){
        //新增方法
        // INSERT INTO user ( id, name, age, email ) VALUES ( ?, ?, ?, ? )
        User user = new User();
        user.setName("张三");
        user.setAge(22);
        user.setEmail("zstest@163.com");
        userMapper.insert(user);
        System.out.println("id="+user.getId());
    }

    @Test
    public void testDeleteByid(){
        //通过id删除    长度超过范围,加L标识为Long类型
        //DELETE FROM user WHERE id=?
        //userMapper.deleteById(1569953783822200833L);

        //根据map集合中设置的条件删除
        //DELETE FROM user WHERE name = ? AND age = ?
//        Map<String, Object> map = new HashMap<>();
//        map.put("name","张三");
//        map.put("age",22);
//        userMapper.deleteByMap(map);

        //根据多个id实现批量删除
        //DELETE FROM user WHERE id IN ( ? , ? , ? )
        List<Long> list = Arrays.asList(1L,2L,3L);
        userMapper.deleteBatchIds(list);
    }

    @Test
    public void testUpdate(){
        //根据id进行修改
        //UPDATE user SET name=?, email=? WHERE id=?
        User user = new User();
        user.setId(4L);
        user.setName("李四");
        user.setEmail("lisi@qq.com");
        userMapper.updateById(user);
    }

    @Test
    public void testSelect(){
        //SELECT id,name,age,email FROM user WHERE id=?
//        userMapper.selectById(1L);

        //根据多个id查询
        //SELECT id,name,age,email FROM user WHERE id IN ( ? , ? , ? )
//        List<Long> list = Arrays.asList(1L,2L,3L);
//        userMapper.selectBatchIds(list);

        //格努map集合中的条件查询
        //SELECT id,name,age,email FROM user WHERE name = ? AND age = ?
//        Map<String, Object> map = new HashMap<>();
//        map.put("name","张三");
//        map.put("age",43);
//        userMapper.selectByMap(map);

        //根据list查询,空就是查询所有
//       List<User> list = userMapper.selectList(null);

        //自定义方法
        userMapper.selectMapById(2L);
    }
}
```

#### 通用servive

service接口

```java
public interface UserService extends IService<User> {
}
```

service接口实现类

```java
@Service
public class UserServiceImpl 
extends ServiceImpl<UserMapper, User> implements UserService {
}
```

```java
@Autowired
private UserService userService;

@Test
public void testGetCount(){
    //查询总记录数
    //SELECT COUNT( * ) FROM user
    userService.count();
}

@Test
public void testInsertMore(){
        //批量添加
        //INSERT INTO user ( id, name, age, email ) VALUES ( ?, ?, ?, ? )
        List<User> list = new ArrayList<>();
        User user = new User();
        user.setName("yll");
        user.setAge(20);

        User user2 = new User();
        user2.setName("yll2");
        user2.setAge(21);

        list.add(user);
        list.add(user2);
        
        boolean b = userService.saveBatch(list);
        System.out.println(b);
}
```

#### 常用注解

1.`@TableName` :	设置实体类对应的表名,放在实体类上

第二种方式:设置mybatis-plus的全局配置

```properties
mybatis-plus.global-config.db-config.table-prefix: t_ :设置实体类对应的表的统一前缀
```

2.`@TableId` : 将属性对应的字段指定为主键,主键字段不是id使用

@TableId( value = "uid" ): 用于指定主键数据库的字段

@TableId( type= IdType.AUTO ): 用于设置主键生成策略,默认雪花算法

AUTO:主键自增长策略,需要保证数据库设置了id自增,否则无效

设置mybatis-plus的全局配置:	统一的主键生成策略

`mybatis-plus.global-config.db-config.id-type:auto`



普通字段与属性名不一致时:

3.`@TableField`	: 指定属性所对应的字段名

```java
指定为公共字段

@TableField(fill = FieldFill.INSERT) //插入时填充字段
private LocalDateTime createTime;


     * 默认不处理
     */
    DEFAULT,
    /**
     * 插入时填充字段
     */
    INSERT,
    /**
     * 更新时填充字段
     */
    UPDATE,
    /**
     * 插入和更新时填充字段
     */
    INSERT_UPDATE
```

```java
@Component
//自定义元数据对象处理器
public class MyMetaObjectHandler implements MetaObjectHandler {

    //插入操作自动填充
    @Override
    public void insertFill(MetaObject metaObject) {
        metaObject.setValue("createTime", LocalDateTime.now());
        metaObject.setValue("updateTime",LocalDateTime.now());
        metaObject.setValue("createUser",BaseContext.getCurrentId());
        metaObject.setValue("updateUser",BaseContext.getCurrentId());
    }

    //更新操作,自动填充
    @Override
    public void updateFill(MetaObject metaObject) {
        metaObject.setValue("updateTime",LocalDateTime.now());
        metaObject.setValue("updateUser",BaseContext.getCurrentId());
    }
}
```





###### 4.`@TableLongic`

逻辑删除

物理删除:	真实删除,将对应数据从数据库删除,之后查询不到被删除的数据

逻辑删除:	假删除,将对应数据中代表是否被删除字段的状态修改为"被删除状态",

之后在数据库中仍然可以看到此条数据

使用场景:	可以进行数据恢复



实现逻辑删除:

1.数据库中创建逻辑删除状态列,设置默认值为0

2.实体类添加逻辑删除属性

```java
@TableLogic
private Integer isDelete;
```



#### 条件构造器

```java
@Autowired
private UserMapper userMapper;

@Test
public void test01(){
    //查询用户名包含张,年龄在20到30之间,邮箱信息不为空的用户信息
    //SELECT uid AS id,name,age,email,is_delete FROM user
    //WHERE is_delete=0 AND (name LIKE ? AND age BETWEEN ? AND ? AND email IS NOT NULL)
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.like("name","张")
            .between("age",20,30)
            .isNotNull("email");
    List<User> list = userMapper.selectList(queryWrapper);
    list.forEach(System.out::println);
}

@Test
public void test02(){
    //查询用户信息,安装年龄降序,若年龄相同,按id升序
    //SELECT uid AS id,name,age,email,is_delete FROM user
    // WHERE is_delete=0 ORDER BY age DESC,uid ASC
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.orderByDesc("age")
            .orderByAsc("uid");
    userMapper.selectList(queryWrapper);
}

@Test
public void test03(){
    //删除邮箱为空的用户信息
    //UPDATE user SET is_delete=1 WHERE is_delete=0 AND (email IS NULL)
    QueryWrapper<User> q = new QueryWrapper<>();
    q.isNull("email");
    userMapper.delete(q);
}

@Test
public void test04(){
    //修改年龄大于20,名字带有张,或者邮箱为空的用户
    //UPDATE user SET name=?, email=? WHERE is_delete=0 AND (name LIKE ? AND age > ? OR email IS NULL)
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.like("name","张")
            .gt("age",20)
            .or()
            .isNull("email");
    User user = new User();
    user.setName("小明");
    user.setEmail("xm@qq.com");
    userMapper.update(user,queryWrapper);
}

@Test
public void test05(){
    //将用户名中包含有张,并且(年龄大于20或者邮箱为null)的用户信息修改
    //UPDATE user SET name=?, email=? WHERE is_delete=0 AND (name LIKE ? AND (age > ? OR email IS NULL))
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.like("name","张")
            .and(a -> a.gt("age",20).or().isNull("email"));
    User user = new User();
    user.setName("小明");
    user.setEmail("xm@qq.com");
    userMapper.update(user,queryWrapper);
}

@Test
public void test06(){
    //查询用户名,年龄  SELECT name,age FROM user WHERE is_delete=0
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.select("name","age");
    userMapper.selectMaps(queryWrapper);

}

@Test
public void test07(){
    //UpdateWrapper可以设置修改条件,和修改的字段
    //将用户名中包含有张,并且(年龄大于20或者邮箱为null)的用户信息修改,使用updateWrapper
    UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
    updateWrapper.like("name","明")
            .and(i -> i.gt("age",20).or().isNull("email"));
    updateWrapper.set("name","张三").set("email","zs@qq.com");
    userMapper.update(null,updateWrapper);
}

@Test
public void test08(){
    String name ="张";
    Integer ageBegin = null;
    Integer ageEnd = 50;
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.like(StringUtils.isNotBlank(name),"name",name)
            .ge(ageBegin != null,"age",ageBegin)
            .le(ageEnd!=null,"age",ageEnd);
    userMapper.selectList(queryWrapper);
}

@Test
public void test09(){
    //使用LambdaQueryWrapper
    String name ="张";
    Integer ageBegin = null;
    Integer ageEnd = 50;
    LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
    queryWrapper.like(StringUtils.isNotBlank(name),User::getName,name)
            .ge(ageBegin != null,User::getAge,ageBegin)
            .le(ageEnd!=null,User::getAge,ageEnd);
    userMapper.selectList(queryWrapper);
}

@Test
public void test10(){
    //使用 LambdaUpdateWrapper
    //将用户名中包含有张,并且(年龄大于20或者邮箱为null)的用户信息修改,使用updateWrapper
    LambdaUpdateWrapper<User> updateWrapper = new LambdaUpdateWrapper<>();
    updateWrapper.like(User::getName,"明")
            .and(i -> i.gt(User::getAge,20).or().isNull(User::getEmail));
    updateWrapper.set(User::getName,"张三").set(User::getEmail,"zs@qq.com");
    userMapper.update(null,updateWrapper);
}
```



#### 通用枚举

枚举类:

```java
@Getter
public enum SexEnum {
    MALE(1,"男"),
    FEMALE(2,"女")
    ;

    @EnumValue //将注解所标识的属性的值存储到数据库中
    private Integer sex;
    private String sexName;

    SexEnum(Integer sex, String sexName) {
        this.sex = sex;
        this.sexName = sexName;
    }
}
```



配置:

```properties
#扫描通用枚举的包
mybatis-plus.type-enums-package=com.cn.enums
```

## 插件

#### 1.分页插件

#### 分页查询

配置类配置分页插件

```java
@Configuration
public class MyBatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
```

使用:

```java
@Autowired
private UserMapper userMapper;

@Test
public void testPage(){
    Page<User> page = new Page<>(1,3);
    userMapper.selectPage(page,null);
    System.out.println(page.getRecords());
    System.out.println("查询出的数据由有:"+page.getSize());
    System.out.println("共"+page.getPages()+"条数据");
    System.out.println("数据库总共有"+page.getTotal()+"条数据");
    System.out.println("是否有下一页:"+page.hasNext());
    System.out.println("是否有上一页:"+page.hasPrevious());
}
```





#### 乐观锁

```java
@Data
public class Product {

    private Long id;
    private String name;
    private Integer price;
    
    @Version //标识乐观锁版本字段
    private Integer version;
}
```

```java
@Configuration
public class MyBatisPlusConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        //添加分页插件
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        //添加乐观锁插件
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return interceptor;
    }


}
```



#### 代码生成器

##### 依赖:

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.5.1</version>
</dependency>

<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.31</version>
</dependency>
```



##### 运行:

```java
public class FastAutoGeneratorTest {

    public static void main(String[] args) {
        FastAutoGenerator.create("jdbc:mysql://localhost:3306/mybatis-plus?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2B8", "root", "123456")
                .globalConfig(builder -> {
                    builder.author("baomidou") // 设置作者
                            //.enableSwagger() // 开启 swagger 模式
                            .fileOverride() // 覆盖已生成文件
                            .outputDir("D://mybatis_plus"); // 指定输出目录
                })
                .packageConfig(builder -> {
                    builder.parent("com.cn") // 设置父包名
                            .moduleName("mybatisplus") // 设置父包模块名
                            .pathInfo(Collections.singletonMap(OutputFile.mapperXml, "D://mybatis_plus")); // 设置mapperXml生成路径
                })
                .strategyConfig(builder -> {
                    builder.addInclude("user") // 设置需要生成的表名
                            .addTablePrefix("t_", "c_"); // 设置过滤表前缀
                })
                .templateEngine(new FreemarkerTemplateEngine()) // 使用Freemarker引擎模板，默认的是Velocity引擎模板
                .execute();
    }
}
```





MyBatisX插件



###### 多数据源