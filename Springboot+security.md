# Springboot+security

##### [springboot整合springsecurity最完整，只看这一篇就够了 - QianTM - 博客园 (cnblogs.com)](https://www.cnblogs.com/qiantao/p/14605154.html)

###### --0514 --SXJ练习

###### 项目：testsecurity

1. 新建springboot项目，加入web和spring security依赖
2. 编写control类

```
package com.security.mycontroller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class TestController {
    @GetMapping("/hello")
    public String request() {
        return "hello";
    }
}
```

运行项目后，给出了一组password，说明被security保护起来了

![](D:\godblessing\forwork\操作记录\figure\springboot+security-1.png)

3. 浏览器访问http://localhost:8080/hello，会自动重定向到登录界面（security自带的默认界面）

![](D:\godblessing\forwork\操作记录\figure\springboot+security-2.png)

输入：user 4028f297-8e51-49f3-8f43-9254ff2b2e87

![](D:\godblessing\forwork\操作记录\figure\springboot+security-3.png)

<u>*从 Spring Security 5.7 开始，WebSecurityConfigurerAdapter类已被弃用，Spring 团队鼓励用户转向基于组件的安全配置。*</u>

因此xmind中的代码无法成功完成，如何替代呢...

