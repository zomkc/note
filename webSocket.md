#### Sring支持原生webSocket

##### 起步依赖

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
```

##### 配置类

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig {

    @Bean
    public ServerEndpointExporter serverEndpoint() {
        return new ServerEndpointExporter();
    }
}
```

##### 使用原生websocket

```java
@ServerEndpoint("/myWs")
@Component
public class WsServerEndpoint {
    /**
     * 连接成功
     *
     * @param session
     */
    @OnOpen
    public void onOpen(Session session) {
        System.out.println("连接成功");
    }

    /**
     * 连接关闭
     *
     * @param session
     */
    @OnClose
    public void onClose(Session session) {
        System.out.println("连接关闭");
    }

    /**
     * 接收到消息
     *
     * @param text
     */
    @OnMessage
    public String onMsg(String text) throws IOException {
        return "servet 发送：" + text;
    }
}
```

##### 说明:

他们的包都在 **javax.websocket **下。并不是 spring 提供的，而 jdk 自带的，下面是他们的具体作用。

``@ServerEndpoint``

​		通过这个 spring boot 就可以知道你暴露出去的 ws 应用的路径，有点类似我们经常用的@RequestMapping。比如你的启动端口是 8080，而这个注解的值是 ws，那我们就可以通过 ws://127.0.0.1:8080/ws 来连接你的应用

``@OnOpen``

​		当 websocket 建立连接成功后会触发这个注解修饰的方法，注意它有一个  Session 参数

``@OnClose``

​		当 websocket 建立的连接断开后会触发这个注解修饰的方法，注意它有一个  Session 参数

``@OnMessage``

​		当客户端发送消息到服务端时，会触发这个注解修改的方法，它有一个 String 入参表明客户端传入的值

``@OnError``

​		当 websocket 建立连接时出现异常会触发这个注解修饰的方法，注意它有一个  Session 参数





