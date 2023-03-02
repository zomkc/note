# AJAX&JSON笔记



##### AJAX:

##### 1.概念:Asynchronous JavaScript And XML	异步的JavaScript和XML

1. 异步和同步:客户端和服务器端相互通信的基础上

   异步:客户端必须等待服务器端的响应,在等待的期间客户端不能进行其他操作

   同步:客户端不需要等待服务器的响应,在服务器处理请求的过程中,客户端可以进行其他操作



​				提升用户体验

##### 2.实现方式:

1. 原生的JS实现方式



2. JQuery实现方式

   1.`$.ajax()`

   ```java
   //使用$.ajax()发送异步请求
    $.ajax({
        url:"ajaxServlet",//请求路径
        type:"POST",//请求方式
        data:{"username":"jack","age":23},
        success:function (data) {
           alert(data);
        },//响应成功后回调函数
        error:function () {
           alert("出错啦...")
        },//表示如果请求响应出现错误,会执行的回调函数
        dataType:"text"//设置接受到的响应数据的格式
    });
   ```
   
   2.`$.get()`:发送get请求
   
   语法:`$.get(url,[data],[callback],[type])`
   
   * 参数:
   
   * url:请求路径
   
   * data:请求参数
   
   * callback:回调函数
   
   * type:响应结果的类型
   
     
   
   3.`$.post()`:发送post请求



##### JSON

概念:JavaScript Object Notation	JavaScript对象表示法

json现在多用于存储和交换文本信息的语法

进行数据的传输

JSON 比 XML 更小,更快,更易解读



语法:

##### 1.基本规则

数据在名称/值对中:json数据是由键值对构成的

键用引号(单双都行)引起来,也可以不使用引号

值的取值类型:

1. 数字(整点或浮点数
2. 字符串(在双引号中)
3. 逻辑值(true或false)
4. 数组(在方括号中) {"persons": [{}, {}, {}] }
5. 对象(在花括号中) {"address": {"province":"陕西",  ... } }
6. null

数据由逗号分隔:多个键值对由逗号分隔

花括号保存对象:使用{}定义json格式

方括号保存数组: [ ]



##### 2.获取数据:

1. json对象.键名
2. json对象["键名"]
3. 数组对象[索引]

```html
<script>
let person = {"name":"张三",age:23,'gender':true}
let name = person.name;
let age = person["age"]
let ps = [
        {"name":"张三","age":12,"gender":true},
        {"name":"张四","age":13,"gender":true},
        {"name":"张五","age":14,"gender":false}
    ];
for (let key in person) {
    alert(key+":"+person[key]);
}

for (let i = 0; i < ps.length; i++) {
    let p = ps[i];
    for (let key in p) {
        alert(key+":"+p[key]);
    }
}

</script>
```



3.JSON数据和Java对象的相互转换

JSON解析器:

* 常见的解析器:Jsonlib, Gson, fastjson, jackson

1.JSON转为Java对象

1. 导入jackson相关的jar包

2. 创建jackson核心对象 ObjectMapper

3. 创建ObjecttMapper的相关方法进行转换

   `readValue(json字符串数据,Class)`

##### 3.Java对象转换JSON

使用步骤:

1. 导入jackson相关的jar包

2. 创建jackson核心对象 ObjectMapper

3. 创建ObjecttMapper的相关方法进行转换

   1.转换方法:

   `writeValue(参数1,obj)`:

   参数1:

   File:将obj对象转为JSON字符串,并保存到指定的文件中

   writer:将obj对象转换为JSON字符串,并将json数据填充到字节输出流中

   OutputStream:将obj对象转换为JSON字符串,并将json数据填充到字符输出流中

   `writeValueAsString(obj)`:将对象转为json字符串

   2.注解:

   * @JsonIgnore:排除属性
   * @JsonFormat:属性值的格式化
   
     @JsonFormat(pattern = "yyyy-MM-dd")
   
   3.复杂java对象转换
   
   * List:数组
   * Map:对象格式一致

