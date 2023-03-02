# Cookie&Session&JSP笔记

内容:

1.会话技术

1. Cookie
2. Session

2.JSP



## 会话技术

1.会话:一次会话中包括多次请求和响应

* 一次会话:浏览器第一次给服务器资源发生请求,会话建立,直到有一方断开为止

2.功能:在一次会话的范围内的多次请求间,共享数据

3.方式

1. 客户端会话技术:Cookie
2. 服务器端会话技术:Session





# Cookie

客户端会话技术,将数据保存到客户端

使用步骤:

1. 创建Cookie对象,绑定数据

   `new Cookie(String name , String value);`

2. 发送Cookie对象

   `response.addCookie(Cookie cookie)`

3. 获取Cookie,拿到数据

   `Cookie[] request.getCookies()`

   ```java
   //获取cookie
   Cookie[] cs = request.getCookies();
   //遍历
   if(cs != null){
   for(Cookie c : cs){
   	String name = c.getName();
   	String value = c.getValue();
   		}
   	}
   
   ```

   

实现原理:

* 基于响应头set-cookie和请求头cookie实现



cookie的细节

1.一次可不可以发送多个cookie

* 可以创建多个Cookie对象,使用response使用多次addCookie方法发送Cookie即可

2.cookie在浏览器中保存多少时间

1. 默认情况下,当浏览器关闭后,Cookie数据被销毁

2. 持久化储存:

   `setMaxAge(int seconds)`

   1.正数:将Cookie数据写到硬盘的文件中,持10久化储存,Cookie存活时间

   2.负数:默认值,cookie在当前浏览器内存中,浏览器关闭则销毁

   3.零:删除cookie信息

3.cookie能不能存中文

1. 在tomcat 8之前cookie中不能直接存储中文数据
   * 需要将中文数据转码	--一般采用URL编码
2. 在tomcat 8之后,cookie支持中文数据
   * 特殊字符还是不支持,建议使用URL编码储存,URL解码解析

4.cookie共享问题

1. 假设在一个tomcat服务器中,部署多个web项目,那么在这些web项目中

   默认情况下cookie不能共享

   `setPath(String path)`:设置cookie的获取范围,默认情况下,设置当前的虚拟目录

   如果要共享,则可以将path设置为"/"

2. 不同的tomcat服务器间cookie共享问题

   `setDomain(String path)`:如果设置一级域名相同,那么多个服务器之间cookie可以共享

   * `setDomain(".baidu.com")`,那么tieba.baidu.com和news.baidu.com中的cookie可以共享

5.Cookie的特点和作用

1. cookie存储数据在客户端浏览器
2. 浏览器对于单个cookie的大小有限制(4kb) 以及对同一个域名下的总cookie数量也有限制(20个)

* 作用:

  1.cookie一般用于存储少量的不太敏感的数据

  2.在不登录的情况下,完成服务器对客户端的身份识别



# JSP

##### 概念:

* Java Server Pages : java服务器端页面
* 一个特殊的页面,其中既可以指定定义html标签,又可以定义java代码
* 用于简化书写!!!

##### 原理:

* JSP本质上就是一个Servlet

##### JSP的脚本 : JSP定义java代码的方式

1. <%	代码	%>:定义的Java代码,在servlet方法中.servlet方法中可以定义什么,该脚本就可以定义什么
2. <%!   代码    %>:定义的java代码,在jsp转换后的java类的成员位置
3. <%=   代码    %>:定义的java代码,会输出到页面上.输出语句中可以定义什么,该脚本中就可以定义什么

##### JSp的内置对象

在jsp页面中不需要获取和创建,可以直接使用的对象

jsp一共有9个内置对象

其中3个:

`request`

`response`

`out`:字符输出流对象,可以将数据输出到页面上,和response.getWriter()类似

response.getWriter()和out.write()的区别

* 在tomcat服务器真正给客户端做出响应之前,会先找response缓冲区数据,再找out缓冲区数据
* response.getWriter()数据输出永远在out.write()之前





## 内容:

1.JSP:

* 指令
* 注释
* 内置对象

2.MVC开发模式

3.EL表达式

4.JSTL标签

5.三层架构



#### JSP

## 1.指令

##### 作用:用于配置JSP页面,导入资源文件

##### 格式:

​			<%@ 指令名称 属性名1=属性值1 属性名2=属性值2 ... %>

##### 分类:

1.page		  :配置JSP页面的

* contentType : 等同于response.setContentType()

1. 设置响应体的mime类型以及字符集
2. 设置当前jsp页面的编码(只能是高级的IDE才能生效,如果使用低级工具,则需要设置pageEncoding属性设置当前页面的字符集)

* import : 导包
* errorPage : 当前页面发生异常后,会自动跳转到指定的错误页面
* isErrorPage : 标识当前页面是否是错误页面
  * true : 是,可以使用内置对象exception
  * false : 否,默认值,不可以使用内置对象exception



2.include	 :页面包含的,导入页面的资源文件

​				<%@include file="top.jsp"%>

3.taglib		:导入资源

<%@ taglib prefix="c" url="jar包" %>

* prefix:前缀,自定义的

## 2.注释

1.html注释:

<!-- -->:只能注释html代码片段

2.jsp注释:推荐使用

<%-- --%>:可以注释所有

## 3.内置对象

在jsp页面不需要创建,直接使用的对象

一共有9个:

变量名					真实类型									作用

`pageContext`		PageContext					当前页面共享数据,还可以获取其他八个内置对象

`request`				HttpServletRequest		一次请求访问多个资源(转发)

`session`				HttpSession					一次会话的多个请求间

`application`		ServletContext				所有用户间共享数据

`response`			HttpServletResponse		响应对象

`page`					Object								当前页面(Servlet)的对象	this

`out`						JspWriter							输出对象,数据输出到页面上

`config`					ServletConfig					Servlet的配置对象

`exception`			Throwable							异常对象





# Session

##### 概念:

服务器端会话技术,在一次会话的多次请求间共享数据,将数据保存在服务器端的对象中.	HttpSession



1.获取HttpSession对象:

`HttpSession session = request.getSession();`

2.使用HttpSession对象:

`Object getAttribute(String name)`:根据key,获取值

`void setAttribute(String name,Object value)`:存储数据到session域中

`void removeAttribute(String name)`:根据key,删除该键值对



##### 原理:

Session的实现是依赖于Cookie的



##### 细节:

1.当客户端关闭后,服务器不关闭,两次获取Session是否为同一个

* 默认情况下:不是
* 如果需要相同,则可以创建Cookie,键为JSESSIONID,设置最大存活时间,让cookie持久化储存

```java
Cookie c = new Cookie("JSESSIONID",session.getId() );
c.setMaxAge(60*60);
response.addCookie(c);
```

2.客户端不关闭,服务器关闭后,两次获取的session是否是同一个

* 不是同一个,但是要确保数据不丢失

  session的钝化:

  * 在服务器正常关闭之前,将session对象序列化到硬盘上

  session的活化

  * 在服务器启动后,将session文件转化为内存中的session对象即可

3.session什么时候被销毁

1. 服务器关闭

2. session对象调用invalidate()

3. session默认失效时间   30分钟

   选择性配置修改

   <session-config>

   ​				<session-timeout>30</session-timeout>

   </session-config>



##### session的特点

1.session用于储存一次会话的多次请求的数据,存在服务器端

2.session可以储存任意类型,任意大小的数据

###### session与Cookie的区别

1.session储存数据在服务器端,Cookie在客户端

2.session没有数据大小限制,Cookie有

3.session数据安全,Cookie相对于不安全



# EL&JSTL

### MVC开发模式

1.M: Model	模型	JavaBean

* 完成具体的业务操作

2.V: View	视图	JSP

* 展示数据

3.C: Controller	控制器	Servle

* 获取用户的输入
* 调用模型
* 将数据交给视图进行展示

优缺点:

1.优点:

1. 耦合性低,方便维护,可以利于分工协作
2. 重用性高

2.缺点:

1. 使得项目结构变得复杂,对开发人员要求高



##### EL表达式

1.概念:Expression Language	表达式

2.作用:替换和简化jsp页面中java代码的编写

3语法:	${表达式}

4.注意:

* jsp默认支持el表达式的,如果要忽略el表达式

  1.设置jsp中page指令中: isELIgnored="true" 忽略当前页jsp页面中所有的el表达式

  2.\${表达式}:忽略当前这个el表达式

5.使用:

###### 1.运算

运算符:

1.算数运算符:	+	-	*	/(div)	%(mod)

2.比较运算符:	>	<	>=	<=	==	!=

3.逻辑运算符:	&&(and)	||(or)	!(not)

4.空运算符:empty

* 功能:用于判断字符串.集合.数组对象是否为null并且长度是否为0
* `${empty list}`
* `${not empty str}`:表示判断字符串.集合.数组对象是否不为null并且长度是否>0

###### 2.获取值

1. el表达式只能从域对象中获取值

2. 语法:

   1.${域名称.键名}: 从指定域中获取指定键的值

   * 域名称:

     1.`pageScope`				-->pageContext

     2.`requestScope`			-->request

     3.`sessionScope`			-->session

     4.`applicationScope`	-->application(ServletContext)

   

3. ${键名}: 表示依次从最小的域中查找是否有该键对应的值,直到找到为止



3. 获取对象.List集合.Map集合的值

   1.对象:`${域名称.键名.属性名}`

   * 本质上获取调用对象的getter方法

   2.List集合: `${域名称.键名[索引]}`

   3.Map集合:

   * `${域名称.键名.key名称}`
   * `${域名称.键名["key名称"]}`

##### 3.隐式对象:

el表达式中有11个隐式对象

pageContext:

* 获取jsp其他八个内置对象
* `${pageContext.request.contextPath}`:动态获取寻目录



### JSTL

概念:	JavaServer Pages Tag Library			JSP标准标签库



作用:用于简化和替换jsp页面上的java代码



使用步骤:

1. 导入jstl相关jar包
2. 引入标签库: taglib指令:    <%@ taglib %>
3. 使用标签



常用的JSTL标签

1.if	:	相对于java代码的if语句

1. 属性:

   test 必须属性,接收boolean表达式

   如果表达式为true,则宣誓if标签体内容,如果为false,则不显示标签体内容

   一般情况下,test属性值会结合el表达式一起使用



2.choose	:	相对于java代码的switch语句

1. 使用choose标签声明    相对于switch声明
2. 使用when标签做判断    相对于case
3. 使用otherwise标签做其他情况声明    相对于default



3.forEach	:	相对于java代码的for语句

###### 完成重复操作

属性:

begin:开始值

end:结束值

var:临时变量

step:步长

varStatus:循环状态对象

​				index:容器中的元素的索引,从0开始

​				count:循环次数,从1开始

###### 遍历容器

实现:

items:容器对象

var:容器中元素的临时变量

varStatus:循环状态对象

​				index:容器中的元素的索引,从0开始

​				count:循环次数,从1开始





