---
title: HttpSession(2)
author: Sophosss
layout: post
---
上一篇说了如何获取 HttpSession ，接下来就要配置 HttpSession 的类，需要写一个监听器。

```java
@Component
public class RequestListener implements ServletRequestListener {

    @Override
    public void requestDestroyed(ServletRequestEvent sre) {

    }

    @Override
    public void requestInitialized(ServletRequestEvent sre) {
        //将所有request请求都携带上httpSession
        ((HttpServletRequest) sre.getServletRequest()).getSession();
    }
}
```

接着把这个类注册为监听器，使用 @Bean 注解的方式。

```java
@SpringBootApplication
public class BootChatApplication {

    public static void main(String[] args) {
        SpringApplication.run(BootChatApplication.class, args);
    }

    @Autowired
    private RequestListener requestListener;

    @Bean
    public ServletListenerRegistrationBean<RequestListener> servletListenerRegistrationBean() {
        ServletListenerRegistrationBean<RequestListener> servletListenerRegistrationBean = new ServletListenerRegistrationBean<>();
        servletListenerRegistrationBean.setListener(requestListener);
        return servletListenerRegistrationBean;
    }
}
```

那么如何在 WebSocket 中获取用户的 Session 问题就解决了。

刚才通过源码分析，是知道 @ServerEndpoint 注解中是有一个参数可以配置我们刚才继承的类，那么就是

```java
@ServerEndpoint(value = "/chat/{id}", configurator = HttpSessionConfig.class)
```

下面我的这段代码，在 @OnOpen 注解所修饰的方法中，拿到 EndpointConfig 对象，并且通过这个对象，拿到之前我们设置进去的 map，我们就从 Java 的 WebScoket 中拿到了用户的 Session。

```java
@OnOpen
    public void onOpen(Session session, @PathParam("id") String id, EndpointConfig config) {
        log.info("链接成功");
        this.session = session;
        httpSession = (HttpSession) config.getUserProperties().get(HttpSession.class.getName());

        //将当前websocket对象存入到Set集合中
        websocketServerEndpoints.add(this);

        addOnlineCount();

        log.info("有新窗口开始监听：" + id + ", 当前在线人数为：" + getOnlineCount());

        this.fromId = id;
        try {
            User user = new User();
            Object attribute = httpSession.getAttribute(fromId);
            if (attribute instanceof User) user = (User) attribute;
            //群发消息
            Map<String, Object> map = new HashMap<>();
            map.put("online", getOnlineCount());
            map.put("msg", "用户 " + user.getName() + " 已上线");
            sendMore(JSONObject.toJSONString(map));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

