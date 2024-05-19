# Springboot+异常处理

###### --0514--SXJ练习

###### 项目：testexception

1. 建立springboot项目，添加web依赖
2. 建立handler包，新建控制类

```
package com.exception.handler;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MyController {

    @GetMapping("/endpoint")
    public String endPoint() throws Exception {
//    	System.out.println("成功访问");
//    	return "成功访问";
        throw new Exception("异常出错！");
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleMyException(Exception e) {
        return new ResponseEntity<>(e.getMessage(), HttpStatus.BAD_REQUEST);
    }
}
```

Properties中添加配置：（好像不加也可以？

```
spring.mvc.servlet.load-on-startup=10
```

3. 测试，postman发送get到http://localhost:8080/endpoint

![](D:\godblessing\forwork\操作记录\figure\springboot+exception-1.png)