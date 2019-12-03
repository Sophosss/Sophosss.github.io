---
title: WebSocket的@Autowired注入失败
author: Sophosss
layout: post
---
直接上问题！

Q: Spring Boot 的 WebSocket 里面使用 @Autowired 注入 Service 或 Bean 时，报空指针异常，service 为 null（注意：并不是不能被注入）。

那么解决方式也很简单，如下代码，将要注入的 Service 改成 static，就不会为null了。

```java
@Component
@ServerEndpoint(value = "/chat/{id}", configurator = HttpSessionConfig.class)
public class WebsocketServerEndpoint {
    private HttpSession httpSession;

    private static ChatSessionService chatSessionService;

    @Autowired
    public void setChatSessionService(ChatSessionService chatSessionService) {
        WebsocketServerEndpoint.chatSessionService = chatSessionService;
    }
    
    ...
}
```

那么问题的本质是什么呢？

一句话，搞懂它！

Spring管理的都是单例（singleton），和 WebSocket （多对象）相冲突。

还没懂那就再加几句。

项目启动时要进行初始化，会初始化非用户连接的 WebSocket ，Spring同时会为其注入 Service ，该对象的 Service 不是 null，被成功注入。但是，由于 Spring 默认管理的是单例，所以只会注入一次 Service 。当新用户进入聊天时，系统又会创建一个新的 WebSocket 对象，这时矛盾出现了：Spring管理的都是单例，不会给第二个 WebSocket 对象注入 Service ，所以导致只要是用户连接创建的 WebSocket 对象，都不能再注入了。
