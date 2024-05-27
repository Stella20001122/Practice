# springboot+rocketMQ

###### 5.27 

参考链接：[RocketMQ与Springboot整合（rocketmq-spring-boot-starter）实战教程-CSDN博客](https://blog.csdn.net/qq_36737803/article/details/112261352)

1. pom中引入依赖

```
dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>2.2.0</version>
 </dependency>
```

一开始版本是5.2.0，但是好像没有（properties会报错），所以换成低一点的版本

2. properties中添加配置

```
#rocketmq.name-server=127.0.0.1:9876  # 更改为您的RocketMQ名称服务器地址
rocketmq.nameServer=127.0.0.1:9876  # 更改为您的RocketMQ名称服务器地址
rocketmq.producer.group=my-producer-group
# 消息发送超时时长，默认3s
rocketmq.producer.send-message-timeout: 3000
# 同步发送消息失败重试次数，默认2
rocketmq.producer.retry-times-when-send-failed: 3
# 异步发送消息失败重试次数，默认2
rocketmq.producer.retry-times-when-send-async-failed: 3
```

注意版本匹配问题；且不是name-server，而是nameServer

![](D:\godblessing\forwork\操作记录\figure\springboot+rocketMQ.png)

3. 代码

```
package com.rocketmq.test;

import org.apache.catalina.User;
import org.apache.rocketmq.client.producer.SendCallback;
import org.apache.rocketmq.client.producer.SendResult;
import org.apache.rocketmq.spring.core.RocketMQTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.stereotype.Component;

//import com.alibaba.fastjson.JSON;

//import jakarta.annotation.Resource;
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Component
public class MQProducerService {

	@Value("${rocketmq.producer.send-message-timeout}")
    private Integer messageTimeOut;

	// 建议正常规模项目统一用一个TOPIC
    private static final String topic = "RLT_TEST_TOPIC";
    
	// 直接注入使用，用于发送消息到broker服务器
    @Autowired
    private RocketMQTemplate rocketMQTemplate;

	/**
     * 普通发送（这里的参数对象User可以随意定义，可以发送个对象，也可以是字符串等）
     */
    public void send(User user) {
        rocketMQTemplate.convertAndSend(topic + ":tag1", user);
//        rocketMQTemplate.send(topic + ":tag1", MessageBuilder.withPayload(user).build()); // 等价于上面一行
    }

    /**
     * 发送同步消息（阻塞当前线程，等待broker响应发送结果，这样不太容易丢失消息）
     * （msgBody也可以是对象，sendResult为返回的发送结果）
     */
    public SendResult sendMsg(String msgBody) {
        SendResult sendResult = rocketMQTemplate.syncSend(topic, MessageBuilder.withPayload(msgBody).build());
        //log.info("【sendMsg】sendResult={}",JSON.toJSONString(sendResult));
        return sendResult;
    }

	/**
     * 发送异步消息（通过线程池执行发送到broker的消息任务，执行完后回调：在SendCallback中可处理相关成功失败时的逻辑）
     * （适合对响应时间敏感的业务场景）
     */
    public void sendAsyncMsg(String msgBody) {
        rocketMQTemplate.asyncSend(topic, MessageBuilder.withPayload(msgBody).build(), new SendCallback() {
            @Override
            public void onSuccess(SendResult sendResult) {
                // 处理消息发送成功逻辑
            }
            @Override
            public void onException(Throwable throwable) {
                // 处理消息发送异常逻辑
            }
        });
    }
    
	/**
     * 发送延时消息（上面的发送同步消息，delayLevel的值就为0，因为不延时）
     * 在start版本中 延时消息一共分为18个等级分别为：1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
     */
    public void sendDelayMsg(String msgBody, int delayLevel) {
        rocketMQTemplate.syncSend(topic, MessageBuilder.withPayload(msgBody).build(), messageTimeOut, delayLevel);
    }

    /**
     * 发送单向消息（只负责发送消息，不等待应答，不关心发送结果，如日志）
     */
    public void sendOneWayMsg(String msgBody) {
        rocketMQTemplate.sendOneWay(topic, MessageBuilder.withPayload(msgBody).build());
    }
    
	/**
     * 发送带tag的消息，直接在topic后面加上":tag"
     */
    public SendResult sendTagMsg(String msgBody) {
        return rocketMQTemplate.syncSend(topic + ":tag2", MessageBuilder.withPayload(msgBody).build());
    }
    
}
```

4. 运行前先打开rocketMQ服务器

参考链接：[RocketMQ的下载与安装（全网最细保姆级别教学）_rocketmq下载安装-CSDN博客](https://blog.csdn.net/weixin_50503886/article/details/129680320)

1）开启nameserver：

在bin目录（D:\software\rocketmq\rocketmq-all-5.2.0-bin-release\bin）下执行cmd命令呼出命令框，执行 start mqnamesrv.cmd

正常情况会弹出黑框，显示

```
The Name Server boot success. serializeType=JSON, address 0.0.0.0:9876
```

2）开启broker：

与上述同样的路径下呼出对话框，执行 start mqbroker.cmd -n 127.0.0.1:9876 autoCreateTopicEnable = true

正常情况会弹出黑框，最后一句显示

```
The broker[LAPTOP-2TA21OUC, 172.28.101.163:10911] boot success. serializeType=JSON and name server is 127.0.0.1:9876
```



5. 运行springboot，发现报错(未能加载bean文件)

```
The injection point has the following annotations:
- @org.springframework.beans.factory.annotation.Autowired(required=true)

Action:

Consider defining a bean of type ‘org.apache.rocketmq.spring.core.RocketMQTemplate’ in your configuration.
```

参考链接：[SpringBoot3.0整合RocketMQ时出现未能加载bean文件_consider defining a bean of type 'org.apache.rocke-CSDN博客](https://blog.csdn.net/qq_47070121/article/details/131462589)

解决方案：定义一个配置类

```
package com.rocketmq.test;

import org.apache.rocketmq.client.producer.DefaultMQProducer;
import org.apache.rocketmq.spring.core.RocketMQTemplate;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RocketMqConfig {
    @Value("${rocketmq.producer.group}")
    private String producerGroup;

    @Value("${rocketmq.name-server}")
    private String nameServer;

    /**
     * 由于使用的Spring版本是3.0.0以上，与rocketMq不是很兼容，对于rocketMqTemplate
     * 的自动注入存在差异，如果不采用这种方式注入则会报出缺少bean的信息
     */
    @Bean("RocketMqTemplate")
    public RocketMQTemplate rocketMqTemplate(){
        RocketMQTemplate rocketMqTemplate = new RocketMQTemplate();
        DefaultMQProducer defaultMqProducer = new DefaultMQProducer();
        defaultMqProducer.setProducerGroup(producerGroup);
        defaultMqProducer.setNamesrvAddr(nameServer);
        rocketMqTemplate.setProducer(defaultMqProducer);
        return rocketMqTemplate;
    }
}
```

然后可以正常运行，但是日志有个warn：

![](D:\godblessing\forwork\操作记录\figure\springboot+rocketMQ-1.png)

应该是日志没配好；参考代码里45行的log一直搞不对，所以注释掉了