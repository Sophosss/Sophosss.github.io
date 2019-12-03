---
title: HttpSession(1)
author: Sophosss
layout: post
---
WebSocket 虽然能实时接收到消息，但每次刷新浏览器，之前发送过的消息都会丢失，所以一种简单的做法就是利用HttpSession 实现了消息储存功能。

利用HttpSession，将每次会话，用户推送的消息都储存到 HttpSession 中。前端利用 Vue 的 `created()` 钩子函数，每次刷新页面时都先请求获取 HTTPSession 中已储存的消息列表。

- 源码分析

获取 HttpSession 是一个很有必要讨论的问题，因为 Java 后台需要知道当前是哪个用户，用以处理该用户的业务逻辑，或者是对该用户进行授权之类的，但是由于 WebSocket 的协议与 Http 协议是不同的，所以造成了无法直接拿到 Session 。但是问题总是要解决的，不然这个 WebSocket 协议所用的场景也就没了。

下面是 @ServerEndpoint 注解的源码

```java
package javax.websocket.server;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import javax.websocket.Decoder;
import javax.websocket.Encoder;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface ServerEndpoint {

    /**
     * URI or URI-template that the annotated class should be mapped to.
     * @return The URI or URI-template that the annotated class should be mapped
     *         to.
     */
    String value();

    String[] subprotocols() default {};

    Class<? extends Decoder>[] decoders() default {};

    Class<? extends Encoder>[] encoders() default {};

    public Class<? extends ServerEndpointConfig.Configurator> configurator()
            default ServerEndpointConfig.Configurator.class;
}
```

最后一个方法是关键，要求返回一个 ServerEndpointConfig.Configurator 的子类，我们要写一个类去继承它。

```Java
public class HttpSessionConfig extends Configurator {

    @Override
    public void modifyHandshake(ServerEndpointConfig sec, HandshakeRequest request, HandshakeResponse response) {
        
        ...
    }
}
```

这个配置类要怎么写呢？我们先点开 HandshakeRequest 的源码进去看看。

```java
package javax.websocket.server;

import java.net.URI;
import java.security.Principal;
import java.util.List;
import java.util.Map;

/**
 * Represents the HTTP request that asked to be upgraded to WebSocket.
 */
public interface HandshakeRequest {

    static final String SEC_WEBSOCKET_KEY = "Sec-WebSocket-Key";
    static final String SEC_WEBSOCKET_PROTOCOL = "Sec-WebSocket-Protocol";
    static final String SEC_WEBSOCKET_VERSION = "Sec-WebSocket-Version";
    static final String SEC_WEBSOCKET_EXTENSIONS= "Sec-WebSocket-Extensions";

    Map<String,List<String>> getHeaders();

    Principal getUserPrincipal();

    URI getRequestURI();

    boolean isUserInRole(String role);

    /**
     * Get the HTTP Session object associated with this request. Object is used
     * to avoid a direct dependency on the Servlet API.
     * @return The javax.servlet.http.HttpSession object associated with this
     *         request, if any.
     */
    Object getHttpSession();

    Map<String, List<String>> getParameterMap();

    String getQueryString();
}
```

那么就很清晰了，HandshakeRequest 是一个接口，接口中规范了`Object getHttpSession();`可以从 Servlet API中获取到相应的HttpSession。具体的代码如下。

```Java
public class HttpSessionConfig extends Configurator {

    @Override
    public void modifyHandshake(ServerEndpointConfig sec, HandshakeRequest request, HandshakeResponse response) {
        HttpSession session = (HttpSession) request.getHttpSession();
        sec.getUserProperties().put(HttpSession.class.getName(), session);
    }
}
```

再多说几句。

```java
 sec.getUserProperties().put(HttpSession.class.getName(), httpSession);
```

这行代码又是什么意思呢？

我们看一下ServerEnpointConfig的声明。

```
public interface ServerEndpointConfig extends EndpointConfig
```

我们发现这个接口继承了EndpointConfig的接口，我们看一下 EndpointConfig 的源码：

```java
package javax.websocket;

import java.util.List;
import java.util.Map;

public interface EndpointConfig {

    List<Class<? extends Encoder>> getEncoders();

    List<Class<? extends Decoder>> getDecoders();

    Map<String,Object> getUserProperties();
}
```

我们发现了这样的一个方法定义

```java
Map<String,Object> getUserProperties();
```

可以看到，它是一个map，从方法名也可以理解到，它是用户的一些属性的存储，那既然它提供了get方法，那么也就意味着我们可以拿到这个map，并且对这里面的值进行操作。