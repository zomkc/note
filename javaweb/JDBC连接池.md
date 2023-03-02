## JDBC连接池

# 内容

1.数据库连接池

2.Spring	JDBC	:	JDBC	Template



# 数据库连接池

###### 1.概念:其实就是一个容器(集合),存放数据库连接的容器.

当系统初始化好后,容器被创建,容器中会申请一些连接对象,当用户来访问数据库时,从容器中获取连接对象,用户访问完之后,会将连接对象归还给容器

###### 2.好处:

1.节约资源

2.用户访问高效

###### 3.实现:

1.标准接口:DataSource	java.sql包下的

* 方法:

  获取连接:`getConnection()`

  归还连接:如果连接对象Connection是从连接池中获取的,那么调用`Connection.cloce()`方法,则不会再关闭连接了,而是归还连接

2.一般我们不去实现他,由数据库厂商来实现

* 1.C3P0:数据库连接池技术
* 2.Druid:数据库连接池实现技术,由阿里巴巴提供的



###### 4.C3P0:数据库连接池技术

步骤:

1.导入jar包(两个)

2.定义配置文件

3.创建核心对象,数据库连接池对象	ComboPooledDateSource

3.获取连接:getConnection

###### 5.Druid:数据库连接实现技术,由阿里巴巴提供的

步骤:

1.导入jar包	druid-1.0.9.jar

2.定义配置文件

* 是properties形式的
* 可以叫任意名称,可以放在任意目录下

3.加载配置文件:`Properties`

4.获取数据库连接池对象:通过工厂类来获取,	`DruidDataSourceFactory`

5.获取连接:`getConnection`



##### 定义工具类

1.定义一个类	JDBCUtils

2.提供静态代码块加载配置文件,初始化连接池对象

3.提供方法

* 获取连接方法:通过数据库连接池获取连接
* 释放资源
* 获取连接池的对象

```java
package cn.it.web.enroll;

import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;
import java.util.Properties;

public class JDBCUtil {
    //定义成员变量 DataSource
    private static DataSource ds;

    static{
        try {
            //1.加载配置文件
            Properties pro = new Properties();
            pro.load(JDBCutil.class.getClassLoader().getResourceAsStream("druid.properties"));
            //2.获取DataSource
            ds = DruidDataSourceFactory.createDataSource(pro);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    //获取连接
    public static Connection getConnection() throws Exception {
        return ds.getConnection();
    }
    //释放资源
    public static void close(Statement stmt, Connection conn){
        close(null,stmt,conn);
    }

    public static void close(ResultSet rs, Statement stmt, Connection conn){
        if (rs!=null){
            try {
                rs.close();
            } catch (Exception throwables) {
                throwables.printStackTrace();
            }
        }
        if (stmt!=null) {
            try {
                stmt.close();
            } catch (Exception throwables) {
                throwables.printStackTrace();
            }
        }
        if (conn!=null) {
            try {
                conn.close();//归还连接
            } catch (Exception throwables) {
                throwables.printStackTrace();
            }
        }
    }
    //获取连接池方法
    public static DataSource getDataSource(){
        return ds;
    }
}
```

```java
public static void main(String[] args) {
    Connection conn = null;
    PreparedStatement pstmt =null;
    try {
        //获取连接
        conn = JDBCUtils.getConnection();
        //定义sql
        String sql = "insert into emp values(?,?,?)";
        //获取pstmt对象
        pstmt = conn.prepareStatement(sql);
        //给?赋值
        pstmt.setInt(1,4);
        pstmt.setString(2,"王五");
        pstmt.setInt(3,34);
        //执行sql
        int count = pstmt.executeUpdate();
        System.out.println(count);
    } catch (Exception throwables) {
        throwables.printStackTrace();
    }finally {
        JDBCUtils.close(pstmt,conn);
    }
}
```

# Spring JDBCT

Spring框架对JDBC的简单封装,提供了一个JDBCTemplate对象简化JDBC开发

步骤:

1.导入jar包

maven坐标

```xml
		<dependency>
            <groupId>org.springframework.s.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
```



2.创建jdbcTemplate对象,依赖于数据源DataSource

* `jdbcTemplate	template	=	new	jdbcTemplate(ds);`

3.调用jdbcTemplate的方法来完成CRUD的操作

* ###### `update()`:执行DML语句.增,删,改语句

* ###### `queryForMap()`:查询结果将结果集封装为map集合,将列名作为key,将值作为value,将这条记录封装为map集合

  `注意:	这个方法查询的结果集长度只能是1`

* ###### `queryForList()`:查询结果将结果集封装为list集合

  `注意:	将每一条记录封装为一个Map集合,再将Map集合装载到List集合中`

* ###### `query()`:查询结果,将结果封装为javaBean对象

  query的参数:RowMapper

  * 一般我们使用`BeanPropertyRowMapper`实现类,可以完成数据到javaBean的自动封装

    ```java
    new BeanPropertyRowMapper<类型>(类型.class)
    ```

* ###### `queryForObject`:查询结果,将结果封装为对象

  `queryForObject( sql, new BeanPropertyRowMapper<类型>(类型.class),给?赋值,...)`
  
  一般用于聚合函数的查询

```java
public static void main(String[] args) {
    //1.导入jar包
    //2.创建JDBCTemplate
    JdbcTemplate template = new 				         JdbcTemplate(JDBCUtils.getDataSource());
    //3.定义sql
    String sql = "update emp set age = 35 where id = ?";
    //4.调用方法
    template.update(sql,3);
}
```



