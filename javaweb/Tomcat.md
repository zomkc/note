#	Tomcat

Tomcat:web服务器软件

###### 1.下载:官网

###### 2.安装:解压即可

###### 3.卸载:删除目录

###### 4.启动:bin/startup.bat,双击

* 访问:浏览器输入:http://localhost:8080

  ​								http://ip地址:8080		

###### 可能遇到的问题:

1.窗口一闪而过:

原因:没有正确配置JAVA_HOME环境变量,或直接在文件内添加jdk路径

win10:setclasspath.bat文件中添加set "JAVA_HOME=C:\app\Java\jdk-9.0.4"

在linux下Tomcat配置连接jdk路径catalina.sh或setenv.sh文件：

export JAVA_HOME=/usr/java/jdk

2.启动报错:

* 暴力:找到占用的端口号,并且找到对应的进程,杀死该进程

  cmd命令:	netstat -ano

* 温柔:修改自身端口号

  conf/server.xml

一般会将tomcat的默认端口号修改为80.	80端口号是http协议的默认端口号

* 好处:在访问是,就不用输入端口号

###### 5.关闭

1.正常关闭:

* bin/shutdown.bat
* 在命令窗口内:ctrl+c

2.强制关闭

* 点击启动窗口的X

###### 6.配置:

* 部署项目的方式:

  1.直接将项目放到webapps目录下即可

  ​	/xxx:	项目的访问路径-->虚拟目录

  ​	简化部署:将项目打成一个war包,再将war包放置到webapps目录下

  * war包会自动解压缩

  2.卑职conf/server.xml文件

  在<Host>标签中配置

  <Contet docBase="D:\hello" path="/hehe" />

  docBase:项目存放的路径

  path:虚拟目录

  3.在conf\Catalina\loccalhost创建任意名称的xml

  <Contet docBase="D:\hello" />

  虚拟目录:xml文件的名称



* 静态项目和动态项目:

* 目录结构

  * java动态项目的目录结构

    --项目的根目录

    ​	--ERB-INF目录

    ​		--web.xml:web项目的核心配置文件

    ​		--classes目录:放置字节码文件的目录

    ​		--lib目录:放置依赖的jar包
  
* 将Tomcat集成到IDEA中,并且创建javaEE的项目,部署项目



# Servlet

server	applet

###### 概念:运行在服务器端的小程序

Servlet就是一个接口,定义了java类被浏览器访问到(tomcat识别)的规则

将来我们定义一个类,实现servlet接口,复写方法



快速入门

1.创建javaEE项目

2.定义一个类,实现Servlet接口:`implements Servlet`

3.实现接口的抽象方法

4.配置Servlet

在web.xml中配置:

```xml
<servlet>
    <servlet-name>demo1</servlet-name>
    <servlet-class>cn.youyou.web.servlet.ServletDemo1</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>demo1</servlet-name>
    <url-pattern>/demo1</url-pattern>
</servlet-mapping>
```

###### 执行原理:

1.当服务器接受到客户端的请求后,会解析请求的url路径,获取访问的Servlet的资源路径

2.查找web.xml文件,是否有对应的<url-pattern>标签体内容

3.如果有,则在找到对应的<servlet-class>全类名

4.tomcat会将字节码文件加载进内存,并且创建其对象

5.调用其方法



###### Servlet中的生命周期方法

1.被创建:	执行init方法,只执行一次

* Servlet什么时候被创建

  默认情况下,第一次被访问时,Servlet被创建

  可以配置执行Servlet的创建时机

  * 在<servlet>标签下配置:

    1.第一次被访问时创建:<load-on-startup>的值为负数

    2.在服务器启动时创建:<load-on-startup>的值为0或正整数

* Servlet的init方法,只执行一次,说明一个Servlet在内存中只存在一个对象,Servlet是单例的

  多个用户同时访问时,可能存在线程安全问题

  解决:尽量不要再servlet中定义成员变量,即使定义了成员变量,也不要对其修改值

2.提供服务:	执行servlet方法,执行多次

* 每次访问Servlet时,Servlet方法都会被调用一次

3.被销毁:	执行destroy方法,只执行一次

* Servlet被销毁时执行,服务器关闭时,Servlet被销毁
* 只有服务器正常关闭时,才会执行destroy方法
* 在Servlet被销毁之前执行,一般用于释放资源



## servlet3.0:

好处:支持注解配置,可以不需要web.xml

步骤:

1.创建javaEE项目,选择Srevlet的版本3.0以上,可以不创建web.xml

2.定义一个类,实现Servlet接口

3.复写方法

3.在类上使用@WebServlet("资源路径")





# Tomcat&Request

内容:

1.Servlet

2.HTTP协议

3.Request



##### Servlet的体系结构

Servlet	--接口

​		|

GenericServlet	--抽象类

​		|

HttpServlet	--抽象类

* GenericServlet:	将Servlet接口中其他的方法做了默认空实现,只将servlet()方法作为抽象

  将来定义Servlet类时,可以继承GenericServlet,实现servlet()方法即可

* HttpServlet:对http协议的一种封装,简化操作

  1.定义类继承HttpServlet

  2.复写doGet/doPost方法

##### Servlet相关配置

1.urlpartten:Servlet访问路径

* 一个Servlet可以定义多个访问路径:@WebServlet({"d1","dd1","ddd1"})

* 路径定义规则:

  1./xxx:路径匹配

  2./xxx/xxx:多层路径,目录结构

  3.*.do:扩展名匹配



## HTTP:

概念:Hyper Text Transfer Protocol	超文本传输协议

特点:

1.基于TCP/IP的高级协议

2.默认端口号:80

3.基于请求/响应模型的:一次请求对应一次响应

4.无状态的:每次请求之间互相独立,不能交互数据

历史版本:

1.0:每一次请求响应都会建立新的连接

1.1:复用连接



# Request

##### request对象和response对象的原理

1.request和response对象是由服务器创建的.我们来使用它们	

2.request对象是来获取请求消息,response对象是来设置响应消息



##### request对象继承体系结构

ServletRequest		--接口

​		|	继承

HttpServletRequest		--接口

​		|	实现

org.apache.catalina.connector.RequestFacade	类(tomcat)



#### request功能:

#### 1.获取请求消息数据

* ##### 获取请求行数据:

  ##### 1.获取请求方式

  `String	getMethod()`

  ##### 2.获取虚拟目录				**(重点)**

  `String	getContextPath()`

  ##### 3.获取Servlet路径

  `String	getServletPath()`

  ##### 4.获取GET方式请求参数

  `String	getQueryString()`

  ##### 5.获取请求URI				**(重点)**

  `String	getRequestURI()`

  `StringBuffer	getRequestURL()`

  ​					URl:统一资源定位符

  ​					URI:统一资源标识符

  ##### 6.获取协议及版本

  `String	getProtoco()`

  ##### 7.获取客户机的IP地址

  `Strin	getRemoteAddr()`



* ##### 获取请求头数据

  方法:

  `String getHeader(String name)`:通过请求头的名称获取请求头的值

  

  `Enumeration<String> getHetHeaderNames()`:获取所有的请求头名称

  ```java
  //1.获取所有请求头名称
  Enumeration<String> haeaderNames = request.getHeaderNames();
  //2.遍历
  while(headerNames.hasMoreElements()){
  String name = headerNames.nextElement();
      //根据名称获取请求头的值
      String value = request.getHeader(name);
      System.out.println(name+","+value)
  }
  
  ```

  



* ##### 获取请求体数据

  请求体:只有POST请求方式,才有请求体,在请求体中封装POST请求的请求参数

  步骤:

  1.获取流对象

  `BufferedReader getReader()`:获取字符输入流,只能操作字符数据

  `SrevletInputStream getInputStream()`:获取字节输入流,可以操作所有类型数据

  

  2.再从流对象中拿数据

  ```java
  //1.获取字符输入流
  BufferedReader br = req.getReader();
  //2.读取数据
  String line = br.readLine();
  ```



#### 2.其他功能

##### 1.Request通用方式获取请求参数

`String  getParameter(String name)`:根据参数名称获取参数值

`String[]  getParameterValues(String name)`:根据参数名称获取参数值的数组

`Enumeration<String>  getParameterName()`:获取所有请求的参数名称

`Map<String,String[]>  getParameterMap()`:获取所有参数的map集合





中文乱码问题:

get方式:tomcat 8 已经将get方式乱码问题解决了

post方式:会乱码

* 解决:在获取参数前,这只reques的编码

  ```java
  request.setCharacterEncoding("utf-8");
  ```

2.请求转发:一种在服务器内部的资源跳转方式

* 步骤:

  1.通过request对象获取请求转发器对象:

  ```java
  RequestDispatcher request.getRequestDispatcher()
  ```

  2.通过RequestDispatcher对象进行转发:

  `forward(ServletRequest request,ServletResponse response)`

* 特点:

  1.浏览器地址栏路径不发生变化

  2.只能转发到当前服务器内部资源中

  3.转发是一次请求

```java
request.getRequestDispatcher("/a").forward(request,response)
```



3.请求转发资源间共享数据: 使用request对象

* 域对象:一个有作用范围的对象,可以在范围内共享数据

* request域:代表一次请求的范围,一般用于请求转发的多个资源中共享数据

* 方法:

  1.`void setAttribute(String name,Object obj)`:存储数据到request域中

  2.`Object getAttitude(String name)`:通过key,获取值

  3.`void removeAttribute(String name)`:通过key,移除键值对

4.获取ServletContext

`ServletContext getServletContext()`





##### BeanUtils工具类,简化数据封装

* 用于封装javaBean的

  javaBean:标准的java类

  1.类必须被public修饰

  2.必须提供空参的构造器

  3.成员变量必须使用private修饰

  4.提供公共setter和getter方法

* 功能:封装数据

```java
//获取所有请求参数
Map<String, String[]> map = request.getParameterMap();
//创建User对象
User loginuser = new User();
//使用BeanUtils封装
try {
    BeanUtils.populate(loginuser,map);
} catch (IllegalAccessException e) {
    e.printStackTrace();
} catch (InvocationTargetException e) {
    e.printStackTrace();
}
```

###### 概念:

成员变量:

属性:setter和getter方法截取后的产物



###### 方法:

1.`setProper()`

2.`getProper()`

3.`populate(Object obj,Map map)`:将map集合的键值对信息,封装到对应的javaBean对象中





# Response

内容:

1.HTTP协议:响应消息

2.Response对象

3.ServletContext对象



### HTTP协议:

1. 请求消息:客户端发送诶服务器端的数据

   数据格式

   1.请求体

   2.请求头

   3.请求空行

   4.请求体

2. 响应消息:服务器端发送给客户端的数据

   数据格式:

   1.响应行

   组成: 协议/版本	响应状态码	状态码描述

   2.响应头

   格式:	头名称 : 值

   3.响应空行

   4.响应体



###  Response对象

功能:设置响应消息

1. 设置响应行

   ​	1.格式:HTTP/1.1 200 ok

   ​	2.设置状态码:`setStatus(int sc)`

2. 设置响应体:`setHeader(String name,String value)`

3. 设置响应体:

   使用步骤

   ​	1.获取输出流

   * 字符输出流:`PrintWriter getWriter()`
   * 字节输出流:`ServletOutputStream getOutputStream()`

   ​    2.使用输出流,将数据输出到客户端浏览器

##### 完成重定向

```java
//1.设置状态码为302
response.setStatus(302);
//2.设置响应头location
response.setHeader("location","/day14/responseDemo2");
```

```java
//简单的重定向方法
response.sendRedirect("/day14/responseDemo2")
```

##### 重定向的特点:redirect

1.地址栏发生变化

2.重定向可以访问其他站点(服务器)的资源

3.重定向是两次请求,不能使人request对象来共享数据

##### 转发的特点:forward

1.转发地址栏路径不变

2.转发只能访问当前服务器下的资源

3.转发是一次请求,可以使用request对象来共享数据

* forward 和 redirect 区别

##### 路径写法

1. 路径分类:

   ​	1.相对路径:通过相对路径不可以确定唯一资源

   规则:找到当前资源和目录资源之间的相对位置关系

   * ./    :	当前目录
   * ../    :    后退一级目录

   ​	2.绝对路径:通过绝对路径可以确定唯一资源

   * 以/开头的路径

   * 规则:判断定义的路径是给谁用的

     给客户端浏览器使用:需要加虚拟目录(项目的访问路径)

     * 建议虚拟目录动态获取:request.getContextPath()

     给服务器使用:不需要加虚拟目录

     * 转发路径

2. 服务器输出字符数据到浏览器:

   * 步骤:

     1.获取字符输出流

     2.输出数据

* 注意:乱码问题

  1.`PrintWriter pw = response.getWriter()`:获取的流默认编码为iso-8895-1

  2.设置该流的默认编码

  3.告诉浏览器响应体使用的编码

  ```java
  //简单的形式,设置编码,是在获取流之前设置
  response.setContentType("text/html;charset=UTF-8");
  ```

  `pw.write("<h1>hello</h1>");`

3. 服务器输出字节数据到浏览器:

* 步骤:

  ​	1.获取字节输出流

  ​	2.输出数据

```java
response.setContentType("text/html;charset=UTF-8");
//1.获取字节输出流
ServletOutputStream sos = response.getOutputStream()
//2.输出数据
sos.write("你好".getBytes("UTF-8"));    
```



```java
//1.获取文件
FileInutStream fis = new FileInutStream("D://a.jpg")
//2.获取response字节输出流对象
ServletOutputStream sos = response.getOutputStream()
//3.输出图片
byte[] buff = new byte[];
int len = 0;
while((len = fis.read(buff))!=-1){
    sos.while(buff,0,len);
}
fis.close();
```



# ServletContext对象

1. 概念:代表整个web应用,可以和程序的容器(服务器)来通信

2. 获取:

   1.通过request对象获取

   ​		`request.getServletContext();`

   2.通过HttpServlet获取

   ​		`this.getServletContext();`

3. 功能:

   1.获取MIME类型:

   * MIME类型:在互联网通信过程中定义的一种文件数据类型

     格式:大类型/小类型

   * 获取:`String getMimeType(String file)`

   2.域对象:共享数据:

   ​	1.`setAttribute(String name,Object value)`

   ​	2.`getAttribute(String name)`

   ​	3.`removeAttribute(String name)`

   

   ​	ServletContext对象范围:所有用户请求的数据

   3.获取文件的真实(服务器)路径:

   ​	1.方法:`String getRealPath(String path)`

   * `String b = context.getRealPath("/b.txt")`//web目录下资源访问
   * `String c = context.getRealPath("/WEB-INF/c.txt")`//WEB-INF目录下的资源访问
   * `String a = context.getRealPath("/WEB-INF/classes/a.txt")`//src目录下的资源访问

