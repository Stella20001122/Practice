# Springboot+websocket

###### --0514--SXJ练习

###### 项目：testwebsocket

1. 新建springboot项目，加入websocket依赖
2. 新建配置类：

```
package com.websocket.myconfig;

import org.springframework.context.annotation.Configuration;
import org.springframework.messaging.simp.config.MessageBrokerRegistry;
import org.springframework.web.socket.config.annotation.*;

@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        // 注册一个STOMP协议的endpoint，并指定使用的Socket
        registry.addEndpoint("/websocket").withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        // 配置消息代理
        registry.enableSimpleBroker("/topic"); // 设置一个前缀为"/topic"的消息代理
        registry.setApplicationDestinationPrefixes("/app"); // 设置客户端发送消息的前缀为"/app"
    }
}
```

3. 新建控制类

```
package com.websocket.mycontrol;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.messaging.handler.annotation.MessageMapping;
import org.springframework.messaging.handler.annotation.SendTo;
import org.springframework.stereotype.Controller;

@Controller
public class WebSocketController {

    @MessageMapping("/hello") // 客户端发送消息到此地址时，会调用下面的方法
    @SendTo("/topic/greetings") // 将方法的返回值发送到"/topic/greetings"地址
    public String greeting(String message) throws Exception {
        // 这里可以添加业务逻辑，比如记录日志、验证消息等
        return "Hello, " + message + "!";
    }
}
```

4. 如何测试？

**如何模拟客户端？**

