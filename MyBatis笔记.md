# MyBatis

##### 三层架构对应的处理框架

界面层---servlet---springMvc框架

业务逻辑层---service类---spring框架

数据访问层---dao类---mybatis框架



mybatis是一个sql映射框架,提供的数据库的操作能力,增强的JDBC

使用mybatis让开发人员集中精神写sql,不必关心Connection,Statement,ResultSet

的创建,销毁,sql的执行

##### pom.xml

```xml
     <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.2</version>
      </dependency>

    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>8.0.28</version>
    </dependency>
```



##### mybatis配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://www.mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
<!--settings : 控制mybatis全局行为-->
    <settings>
<!-- 设置mybatis输出日志       -->
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>
    
<!--环境配置，数据库的连接信息-->
    <environments default="development">
        <environment id="development">
    <!--指定事务管理的类型，这里简单使用Java的JDBC的提交和回滚设置-->
            <transactionManager type="JDBC"></transactionManager>
    <!--dataSource 指连接源配置，POOLED是JDBC连接对象的数据源连接池的实现-->
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"></property>
                <property name="url" value="jdbc:mysql://localhost:3306/db1"></property>
                <property name="username" value="root"></property>
                <property name="password" value="123456"></property>
            </dataSource>
        </environment>
    </environments>

    <mappers>        <!--这是告诉Mybatis区哪找持久化类的映射文件，对于在src下的文件直接写文件名，            如果在某包下，则要写明路径,如：com/mybatistest/config/User.xml-->
        <mapper resource="org/com/dao/StudentDao.xml"></mapper>
    </mappers>
</configuration>
```



##### StudentDao.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://www.mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.com.dao.StudentDao">
<!--
    select: 表示查询操作
        id: 你要执行的sql语句的唯一标识, mybatis会使用这个id的值来找到要执行的sql语句
            可以自定义,但是要求你使用接口中的方法名称

        resultType: 表示结果类型的,是sql语句执行后得到Resultset,
                    遍历这个ResultSet得到的java对象的类型.
                    值写的是类型的全限定名称
-->
    <select id="selectStudents" resultType="org.com.domain.Student">
        select id,name,email,age from student order by id
    </select>

    <!--插入操作-->
    <insert id="insertStudent">
        insert into student value(#{id},#{name},#{email},#{age})
    </insert>
</mapper>
<!--
    sql映射文件:    写sql语句的,mybatis会执行这些sql
    1.指定的约束文件
        <!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

        mybatis-3-mapper.dtd是约束文件的名称,扩展名是dtd的
    2.约束文件的作用:  限制,检查当前文件中出现的标签,属性必须符合mybatis的要求

    3.mapper 是当前文件的跟标签,必须的
        namespace: 叫做命名空间,唯一值的,可以是自定义的字符串
        要求你使用dao接口的全限定名称
    4.当前文件中,可以使用特定的标签,表示数据库的特定操作
        <select>: 表示执行查询,select语句
        <update>: 表示更新数据库的操作,就是在<update>标签中,写的是update sql语句
        <insert>: 表示插入,放的是insert语句
        <delete>: 表示删除,执行的delete语句
-->
```





```java


import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.com.domain.Student;
import org.junit.Test;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

public class TestMybatis {

    @Test
    public void testInsert() throws IOException {
        //访问mybatis读取student数据
        //1.定义mybatis主配置文件的名称,从类路径的根开始(target/clasess)
        String config="mybatis.xml";
        //2.读取这个config表示的文件
        InputStream in = Resources.getResourceAsStream(config);
        //3.创建了SqlSessionFactoryBuilder对象
        SqlSessionFactoryBuilder buider = new SqlSessionFactoryBuilder();
        //4.创建SqlSessionFactory对象
        SqlSessionFactory factory = buider.build(in);
        //5.[重要]获取SqlSession对象,从SqlSessionFactory中获取SqlSession
        SqlSession sqlSession = factory.openSession();
        //6.[重要]指定要执行的sql语句的标识,sql映射文件中的namespace+"."+标签的id值
        String sqlId = "org.com.dao.StudentDao.insertStudent";
        //7.执行sql语句.通过sqlID找打语句
       // List<Student> o = sqlSession.selectList(sqlId);
        Student student = new Student();
        student.setId(1004);
        student.setName("张飞2");
        student.setEmail("zf2@163.com");
        student.setAge(20);
        int o = sqlSession.insert(sqlId,student);

        //mybatis默认不是自动提交事务的,所以在insert,update,delete后要手工提交事务
        sqlSession.commit();

        //8.输出结果
        //o.forEach(stu -> System.out.println(stu));
        System.out.println(o);

        //9.关闭SqlSession对象
        sqlSession.close();
    }
}
```

### 主要类的介绍

##### 1.Resources:	mybatis中的一个类, 负责读取主配置文件

`InputStream in = Resources.getResourceAsStream("mybatis.xml")`

##### 2.SqlSessionFactoryBuilder:	创建SqlSessionFactory对象

`SqlSessionFactoryBuilder buider = new SqlSessionFactoryBuilder();`

//创建SqlSessionFactory对象

`SqlSessionFactory factory = buider.build(in);`

##### 3.SqlSessionFactory:	重量级对象, 程序创建一个对象耗时比较长,使用资源比较多,在整个项目中,有一个就够用了

SqlSessionFactory:接口	,	接口实现类:	DefauitSqlSessionFactory

SqlSessionFactory作用:	获取SqlSession对象

`SqlSession sqlSession = factory.openSession();`



openSession()方法说明:

1.openSession():	无参数的,获取是非自动提交事务的SqlSession对象

2.openSession(boolean):	openSession(true):	获取自动提交事务的SqlSession

​												openSession(false):	非自动提交事务的SqlSession对象



##### 4.SqlSession

​	SqlSession接口:	定义了操作数据库的方法,例如: selectOne(), selectList(), insert(), update(), delete(), commit(), rollback()

​	SqlSession接口的实现类DefaultSqlSession

使用要求:	SqlSession对象不是线程安全的,需要在方法内部使用,在执行sql语句之前,使用openSession()获取SqlSession对象

​						在执行完sql语句后,需要关闭它,执行SqlSession.close();



##### MybatisUtils工具类

```java
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;


public class MybatisUtils {

   private static SqlSessionFactory factory = null;
   static {
      try {
         InputStream in = Resources.getResourceAsStream("mybatis.xml");
         //创建SqlSessionFactory对象,使用SqlSessionFactoryBuild
         factory = new SqlSessionFactoryBuilder().build(in);
      } catch (IOException e) {
         e.printStackTrace();
      }
   }

   //获取SqlSession的方法
   public static SqlSession getSqlSession(){
      SqlSession sqlSession = null;
      if (factory != null){
         sqlSession = factory.openSession();//非自动提交事务
      }
      return sqlSession;
   }
}
```

```java
SqlSession sqlSession = MybatisUtils.getSqlSession();
//[重要]指定要执行的sql语句的标识,sql映射文件中的namespace+"."+标签的id值
String sqlId = "org.com.dao.StudentDao"+"."+"selectStudents";
//执行sql语句.通过sqlID找到语句
List<Student> o = sqlSession.selectList(sqlId);
//输出结果
o.forEach(stu -> System.out.println(stu));
//关闭SqlSession对象
sqlSession.close();
```



## Mybatis动态代理实现

##### StudentDao接口

```java
import com.org.domian.Student;

import java.util.List;

public interface StudentDao {

    List<Student> selectStudents();

    int insertStudent(Student student);
}
```

```java
import com.org.dao.StudentDao;
import com.org.domian.Student;
import com.org.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

public class TestMybatis {
    @Test
    public void testSelectStudent(){
        /**
         * 使用mybatis的动态代理机制,使用	SqlSession.getMapper(dao接口)
            getMapper能获取dao接口对应的实现类对象   */
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        //jdk的动态代理
        StudentDao dao = sqlSession.getMapper(StudentDao.class);
        //调用dao的方法,执行数据库的操作
        List<Student> students = dao.selectStudents();
         for (Student stu : students){
             System.out.println("学生="+stu);
         }
    }

    @Test
    public void testTnsertStudent(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        StudentDao dao = sqlSession.getMapper(StudentDao.class);
        Student student = new Student();
        student.setId(1006);
        student.setName("王富贵");
        student.setEmail("wfg@163.com");
        student.setAge(28);
        int num = dao.insertStudent(student);
        sqlSession.commit();
        System.out.println("添加对象的数量"+num);
    }
}
```



##### 1.动态代理:

使用SqlSession.getMapper(dao接口.class) 获取这个dao接口的文件

##### 2.传入参数: 从java代码中把数据传入到mapper文件的sql语句中

1. parameterType:	卸载mapper文件中的一个属性,表示dao接口中的方法的参数的数据类型

2. 一个简单类型的参数

   简单类型:mybatis把Java的基本数据类型和String都叫简单类型

   在mapper文件获取简单类型的一个参数的值,使用#{任意字符}

   接口: `pubilc Student selectStudentById(Integer id)`

   mapper:	`select id,name,email,age from student where id=#{studentId}`

3. 多个参数使用@Param命名参数

   接口:`List<Student> selectMultiParam(@Param("myname") String name,@Param("myage")`

   使用 @Param("参数名") String name

   mapper:

   ```xml
   <select id="selectMultiParam" resultType="com.org.domian.Student">
       select id,name,email,age from student where name=#{myname} or age=#{myage}
   </select>
   ```

4. 多个参数,使用java对象

   语法: #{属性名}

   

5. 多个参数-简单类型,按位置传值

   mybatis 3.4之前,使用#{0},#{1}

   mybatis 3.4之后,使用#{arr0},#{arr1}

   

   还可以使用map存放多个值,不建议使用



##### #: 占位符:	告诉mybatis使用实际的参数值代替,并使用PrepareStatement对象执行sql语句,#{...}代替sql语句的   ?   这样做更安全,更迅速

##### $ : 字符串替换



##### #	和	$	的区别:

1.#使用 ? 在sql语句中做站位的,使用PrepareStatement执行sql,效率高

2.#可以避免sql注入,更安全

3.$不使用占位符,是字符串连接方式,使用statement对象执行sql,效率低

4.$有sql注入的风险,缺乏安全性

5.$可以替换表名或者列名



##### mybatis的输出结果

mybatis执行了sql语句,得到java对象

1)resultType结果类型,指sql语句执行完毕后,数据转为的java对象,java类型是任意的

resultType结果类型它的值:

* 1.类型的全限定名称
* 2.类型的别名,      列如java.lang.Integer别名是int

处理方式

​		1.mybatis执行sql语句,然后mybatis调用类的无参构造方法,创建对象

​		2.mybatis把ResultSet指定列值付给同名的属性

2)定义自定义类型的别名

​		1)在mybatis主配置文件中定义,是<typeAlias>定义别名

​		2)可以在resultType中使用自定义别名

3)resultMap:结果映射,指定列名和java对象的属性对应关系

​		1)自定义列值赋值给哪个属性

​		2)当你的列名和属性名不一样是,一定使人resultMap



## 动态sql

动态sql:sql的内容是变化的,可以根据条件获取到不同的sql语句

​				主要是where部分发生变化

动态sql的实现,使用的是mybatis提供的标签,<if>,<where>,<foreach>

1 )<if>是判断条件的

​			语法:<if test="判断java对象的属性值">

​				部分sql语句

​			</if>

2 )<where>用来包含<if>的,当多个if有一个成立的,<where>会自动增加一个where关键字,并去掉if中多余的 and , or 等

3 )<foreach>循环java中的数组,list集合的,主要用在sql的in语句中

`<foreach collection="" item="" open="" close="" separator=""> `

#{xxx}

`<foreach>`

collection:表示接口中的方法参数的类型,入股是数组使用array,如果是list集合使用list

inem:自定义的,表示数组和集合成员的变量

open:循环开始时的字符

close:循环结束时的字符

separator:集合成员之间的分隔符

4 )sql代码片段,就是复用一些语句

步骤:

​		1.先定义<sql id="自定义名称唯一"> sql语句,表名,字段等</sql>

​		2.再使用 <include refid="id的值" />





##### 1.数据库的属性配置文件: 把数据库连接信息放到一个单独的文件中,和mybatis主配置文件分开,目的是便于修改,保存,处理多个数据库的信息

1. 在resources目录中定义一个顺序配置文件,   xxx.properties

   在属性配置文件中,定义数据,格式是 key=value

   key一般使用.做多级目录

2. 在mybatis的主配置文件,使用<properties>指定文件的位置,在需要使用值的地方,   ${key}

##### 2.mapper文件,使用package指定路径

​		<mappers>

​			使用包名

​			name:   xml文件(mapper文件)所在的包名,这个包中所有的xml文件一次都能			加载给mybatis

​			使用package的要求:

1. mapper文件名称需要和接口名称一样,区分大小写的

2. mapper文件和dao接口需要在同一目录

   <package name=""/>

​		</mappers>







![mybatis](MyBatis%E7%AC%94%E8%AE%B0.assets/mybatis.jpg)
