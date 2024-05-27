# Springboot + redis

###### 0526 项目名：connectRedis

1. 新建springboot项目，包含依赖：web，session，data-redis

2. application.properties：

   注意这个spring.data.和spring.的区别
   
   （好像是版本问题，*<u>port加.data会报错</u>*，好神奇

```
server.port=8080

spring.data.redis.host=localhost
spring.redis.port=6379
```

3. 编写控制类

```
package com.myredis.mycontrol;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import javax.servlet.http.HttpSession;

@RestController
@RequestMapping("/session")
public class SessionController {

    @GetMapping("/set")
    public String setSessionAttribute(HttpSession session) {
        session.setAttribute("username", "JohnDoe");
        return "Session attribute set successfully.";
    }

    @GetMapping("/get")
    public String getSessionAttribute(HttpSession session) {
        String username = (String) session.getAttribute("username");
        return "Session attribute username: " + (username != null ? username : "not set");
    }
}

```

4. 运行springboot之前，先打开redis服务器

在redis的文件夹**D:\software\Redis**目录下打开cmd，输入

```
redis-server.exe redis.windows.conf
```

参考链接：[Window下Redis的安装和部署详细图文教程（Redis的安装和可视化工具的使用）_redis安装-CSDN博客](https://blog.csdn.net/weixin_44893902/article/details/123087435)



5. 运行springboot项目，打开postman测试

1）发送get请求到http://localhost:8080/session/set

返回：Session attribute set successfully.

说明运行正确

2）发送get请求到http://localhost:8080/session/get

返回：Session attribute username: JohnDoe

说明运行正确