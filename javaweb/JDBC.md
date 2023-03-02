### JDBC

概念:Java DataBase Connectivity,   java数据库连接,java语言操作数据库

JDBC本质:  其实是官方(sun公司)定义的一套操作所有关系型数据库的规则,即接口.各个数据库厂商去实现这套接口,提供数据库驱动的jar包,我们可以使用这套接口(JDBC)编程,真正执行的代码是驱动jar包中的实现类.



#### 步骤

一.导入驱动jar包:	1.复制jar包到项目的libs目录下.	2.右键-->Add As Library	(添加为库)

二.注册驱动:	`Class.forName("com.mysql.jdbc.Driver")`

​		5.x版本的驱动文件jar包对应的是:	`Calss.forName("com.mysql.jdbc.Driver")`

​		8.X版本的驱动文件jar包对应的是:	`Calss.forName("com.mysql.cj.jdbc.Driver")`

三.获取数据库连接对象:	`Connection conn = `

```java
DriverManager.getConnection("jdbc:mysql://localhost:3306/作业", "root", "123456");
```

四.定义sql语句:	`String sql = "update emp set name  = '林青霞' where id = 1;`

五.获取执行sql的对象:	`Statement stmt = conn.createStatement();`

六.执行sql:	`int count = stmt.executeUpdate(sql);`

七.处理结果

八.释放资源:	`conn / stmt`	.close();





#### DriverManager:驱动管理对象

功能:		

1.注册驱动:告诉程序该使用哪一个数据驱动jar

`static void registerDriver(Driver driver):注册与给定的驱动程序	DriverManager`

写代码使用:	`Class.forName("com.mysql.cj.jdbc.Driver");`

* 通过查看源码发现:	在com.mysql.cj.jdbc.Driver类中存在静态代码块

* 注意:	mysql之后的驱动jar包可以省略注册驱动的步骤.	建议写上

2.获取数据库连接

方法:	

```java
static Connection getConnection(String url,String user,	String password);
```

参数:

url:指定连接的路径

* 语法:`jdbc:mysql://ip地址(域名):端口号/数据库名称`

user:用户名

password:密码

* 如果是连接本机mysql服务器,并且mysql服务默认端口是3306,则url的ip地址和端口号可以省略

#### Connection:数据库连接对象

功能:

1.获取执行sql的对象

* Statement	createStatement()
* preparedStatement	prepareStatement(String	sql)

2.管理事务

* 开启事务:`setAutoCommit(boolean	autoCommit)`:调用该方法设置参数为false,即开启事务
* 提交事务:commit()
* 回滚事务:rollback()

#### Statement:执行sql的对象

执行sql

1.`boolean	execute(String	sql)`:可以执行任意的sql

2.`int	executeUpdate(String	sql):执行DML(insert,update,delete)语句,	DDL(create,alter,drop)语句`

* 返回值:影响的行数,可以通过这个影响的行数判断DML语句是否执行成功,返回值>0则执行成功

3.`ResultSet	executeQuery(String	sql):执行DQL(select)`语句

代码:

```java
Connection conn = null;
Statement temt = null;
try {
    //注册驱动
    Class.forName("com.mysql.cj.jdbc.Driver");
    //获取数据库连接
    conn = DriverManager.getConnection("jdbc:mysql:///作业", "root", "123456");
    //定义sql
    String sql = "delete from emp where id = 4";
    //获取1执行sql的对象
    temt = conn.createStatement();
    //执行sql
    int i = temt.executeUpdate(sql);
    if (i>0){
        System.out.println("删除成功");
    }else {
        System.out.println("删除失败");
    }

} catch (ClassNotFoundException | SQLException e) {
    e.printStackTrace();
}finally {
    if (temt!=null){
        try {
            temt.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }
    if (conn!=null){
        try {
            conn.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }
}
```

#### ResultSet:结果集的对象,封装查询结果

`boolean next()`:	游标向下移动一行,判断当前是否是最后一行末尾(是否有数据),如果是最后一行,则返回`false`,如果不是最后一行,则返回`true`

`getXxx(参数)`:	获取数据

* Xxx:	代表数据类型		如:int getInt()	,	String getString

* 参数:
* int:代表列的编号,从1开始	如:`getString(1)`
* String:代表列名称     如:`getDouble("age")`

注意:

* 使用步骤

  1.游标向下移动一行

  2.判断是否有数据

  3.获取数据

```java
//判断结果集是否是最后一行末尾
while(rs.next()) {
    //获取数据
    int id = rs.getInt(1);
    String name = rs.getString("name");
    int age = rs.getInt("age");
    System.out.println(id + "," + name + "," + age);
                }
```

#### PreparedStatement:执行sql的对象

1.SQL注入问题:在拼接sql时,有一些sql的特殊关键字参与字符串的拼接,会造成安全性问题

* 1.输入用户随便输入密码:a' or  'a'  = 'a

  2.sql:select * from user where username = '用户名' and password = 'a' or  'a'  = 'a'

2.解决sql注入问题:使用PreparedStatement对象类解决

3.预编译的SQL:参数使用?作为占位符

4.步骤:

* 1.导入驱动jar包

  2.注册驱动

  3.获取数据库连接对象	Connection

  4.定义sql语句	sql的参数使用?作为占位符.

  ​	如:`select * from user where username = ? and password = ?;`

  5.获取执行sql的对象

  `PreparedStatement Connection.prepareStatement(String sql)`

  6.gei?赋值:

  * 方法:	setXxx(参数1,	参数2)

    参数1:?的位置编号,从1开始

    参数2:?的值

  6.执行sql,接受返回结果,不需要传递sql语句

  7.处理结果

  8.释放资源

5.注意:	后期会使用PreparedStatement来完成增删改查的所有操作

* 可以防止SQL注入
* 效率更高

```java
登录案例:
public boolean login2(String username,String password){
    if (username == null || password == null){
        return false;
    }
    Connection conn = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;
    //连接数据库判断是否登录成功
    try {
        //注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
        //1.获取连接
        conn = DriverManager.getConnection("jdbc:mysql:///作业", "root","123456");
        //2.定义sql
        String sql = "select * from user where username = ? and password = ?";
        //3.获取执行sql对象
        pstmt = conn.prepareStatement(sql);
        //4.给?赋值
        pstmt.setString(1,username);
        pstmt.setString(2,password);
        //5.执行查询,不需要传参
        rs = pstmt.executeQuery();
        //6.判断
        return rs.next();//如果有下一行则返回true
    } catch (SQLException | ClassNotFoundException throwables) {
        throwables.printStackTrace();
    }finally{
        JDBCUtils.close(rs,pstmt,conn);
    }

    return false;
}
```

抽取JDBC:	JDBCUtils工具类

```java
public class JDBCUtils {
    private static String url;
    private static String user;
    private static String password;
    private static String driver;
    static{
        try {
            //读取配置文件
            //创建Properties集合类
            Properties pro = new Properties();
            //获取src路径下的文件的方式-->ClassLoader类加载器
     		ClassLoader classLoader = JDBCUtils.class.getClassLoader();
            URL res = classLoader.getResource("jdbc.properties");
            String path = res.getPath();
            //加载文件
            pro.load(new FileReader(path));
            //获取数据,赋值
            url = pro.getProperty("url");
            user = pro.getProperty("name");
            password = pro.getProperty("password");
            driver = pro.getProperty("driver");
            //注册驱动
            Class.forName(driver);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
    //获取连接
    public static Connection getConnection() throws SQLException {
           return DriverManager.getConnection(url,user,password);
    }
    //释放资源
    public static void close(Statement stmt, Connection conn){
        if (stmt!= null){
            try {
                stmt.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
        if (conn!= null){
            try {
                conn.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
    }
    public static void close(ResultSet rs,Statement stmt,Connection conn){
        if (rs!=null){
            try {
                rs.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
        if (stmt!= null){
            try {
                stmt.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
        if (conn!= null){
            try {
                conn.close();
            } catch (SQLException throwables) {
                throwables.printStackTrace();
            }
        }
    }
}
```



#### JDBC控制事务:

1.事务:一个包含多个步骤的业务操作.如果这个业务操作被事务管理,则这多个步骤要么同时成功,要么同时失败.

2.操作:

* 开启事务
* 提交事务
* 回滚事务

3.使用Connection对象来管理事务

* 开启事务:`setAutoCommit(boolean	autoCommit)`:调用该方法设置参数为false即开启事务

  ​		在执行sql之前开启事务

* 提交事务:`commit()`

  ​		当所有sql都执行完提交事务

* 回滚事务:`rollback()`

  ​		在catch中回滚事务





## Sharding-JDBC		主从复制,读写分离

````xml
        <dependency>
            <groupId>org.apache.shardingsphere</groupId>
            <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
            <version>4.0.0-RC1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.shardingsphere</groupId>
            <artifactId>sharding-jdbc-spring-namespace</artifactId>
            <version>4.0.0-RC1</version>
        </dependency>
````

````yaml
spring:
  shardingsphere:
    datasource:
      names: master,slave
      master:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://192.168.200.130:3306/takeout?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2B8
        username: root
        password: root
      slave:
        type: com.zaxxer.hikari.HikariDataSource
        driver-class-name: com.mysql.cj.jdbc.Driver
        jdbc-url: jdbc:mysql://192.168.200.131:3306/takeout?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2B8
        username: root
        password: root
    masterslave:
      #读写分离配置
      load-balance-algorithm-type: round_robin  #轮询
      name: dataSource
      master-data-source-name: master
      slave-data-source-names: slave
    props:
      sql:
        show: true #开启SQL显示,默认false
  main:
    allow-bean-definition-overriding: true #允许定义覆盖
````

