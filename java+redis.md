# java+redis

###### 0526 项目：connectredistest

#### 练习：java项目连接redis

1. 新建普通java项目，在modulepath里添加jar包（classpath里添加好像读不到）

   ![](D:\godblessing\forwork\操作记录\figure\java+redis-1.png)

或者直接先写代码，在new Jedis时通过自带加包功能导入，注意module-info.java里要添加requires

```
module connectredistest {
	requires redis.clients.jedis;
}
```

2. 编写代码

```
package connectredistest;

import redis.clients.jedis.Jedis;

public class RedisJava {
    public static void main(String[] args) {
//        //连接本地的 Redis 服务
//        Jedis jedis = new Jedis("localhost");
//        // 如果 Redis 服务设置了密码，需要下面这行，没有就不需要
//        // jedis.auth("123456"); 
//        System.out.println("连接成功");
//        //查看服务是否运行
//        System.out.println("服务正在运行: "+jedis.ping());
    	
    	// 连接到Redis服务器，默认端口是6379
        Jedis jedis = new Jedis("redis://127.0.0.1:6379/0");
        
        try {
            // 验证Redis服务器是否运行正常
            System.out.println("Server is running: " + jedis.ping());

            // 设置缓存数据
            String key = "popular_product_info";
            String value = "{\"id\":123,\"name\":\"Popular Product\",\"price\":99.99}";
            jedis.set(key, value);
            System.out.println("Cached data: " + jedis.get(key));

            // 获取缓存数据
            String cachedData = jedis.get(key);
            if (cachedData != null) {
                System.out.println("Retrieved data from cache: " + cachedData);
            } else {
                System.out.println("No data found in cache for key: " + key);
            }

        } finally {
            // 关闭连接
            if (jedis != null) {
                jedis.close();
            }
        }
    }
}

```

注意，如果

`Jedis jedis = new Jedis("localhost");`

报错（Cannot open Redis connection due invalid URI "localhost".

那就改成

`Jedis jedis = new Jedis("redis://127.0.0.1:6379/0");`



3. 运行，返回结果

```
Server is running: PONG

Cached data: {"id":123,"name":"Popular Product","price":99.99}

Retrieved data from cache: {"id":123,"name":"Popular Product","price":99.99}
```



4. redis服务器窗口（cmd）也可以观察到一些记录

![](D:\godblessing\forwork\操作记录\figure\java+redis-2.png)

#### 练习：计数器

1. 计数器代码

```
package counter;
import redis.clients.jedis.Jedis;

public class RedisCounter {
    private Jedis jedis;

    public RedisCounter(Jedis jedis) {
        this.jedis = jedis;
    }

    /**
     * 递增计数器
     * @param key 计数器的键
     * @return 递增后的值
     */
    public long increment(String key) {
        return jedis.incr(key);
    }

    /**
     * 获取计数器的当前值
     * @param key 计数器的键
     * @return 当前值，如果不存在则返回null
     */
    public Long get(String key) {
        return jedis.get(key) != null ? Long.parseLong(jedis.get(key)) : null;
    }

    // ... 其他可能的方法，如重置计数器等
}
```

2. 测试一下，在刚才的RedisJava类中加入计数器，

```
package connectredistest;

import counter.RedisCounter;
import message.RedisPublisher;
import message.RedisSubscriber;
import redis.clients.jedis.Jedis;

public class RedisJava {
    public static void main(String[] args) {
//        //连接本地的 Redis 服务
//        Jedis jedis = new Jedis("localhost");
//        // 如果 Redis 服务设置了密码，需要下面这行，没有就不需要
//        // jedis.auth("123456"); 
//        System.out.println("连接成功");
//        //查看服务是否运行
//        System.out.println("服务正在运行: "+jedis.ping());
    	
    	// 连接到Redis服务器，默认端口是6379
//    	Jedis jedis = new Jedis("localhost");
        Jedis jedis = new Jedis("redis://127.0.0.1:6379/0");
        
        
        try {
            // 验证Redis服务器是否运行正常
            System.out.println("Server is running: " + jedis.ping());

            // 设置缓存数据
            String key = "popular_product_info";
            String value = "{\"id\":123,\"name\":\"Popular Product\",\"price\":99.99}";
            jedis.set(key, value);
            System.out.println("Cached data: " + jedis.get(key));
            
            // 设置缓存数据
            String key12 = "popular_product_info2";
            String value11 = "{\"id\":456,\"name\":\"Popular Product\",\"price\":66.66}";
            jedis.set(key12, value11);
            System.out.println("Cached data: " + jedis.get(key12));
            
            
            // 获取缓存数据
            String cachedData = jedis.get(key);
            if (cachedData != null) {
                System.out.println("Retrieved data from cache: " + cachedData);
            } else {
                System.out.println("No data found in cache for key: " + key);
            }
            
            //计数器
            RedisCounter ccounter= new RedisCounter(jedis);
            String key1="1"; //要求是数字
            ccounter.increment(key1);
            System.out.println("计数器: " + ccounter.get(key1));
            ccounter.increment(key1);
            System.out.println("计数器: " + ccounter.get(key1));
            

        } finally {
            // 关闭连接
            if (jedis != null) {
                jedis.close();
            }
        }
    }
}

```

3. 运行后，输出结果

```
计数器: 28
计数器: 29
```



#### 练习：发布和订阅消息

1. 发布

```
package message;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPubSub;

public class RedisPublisher {
	
	public void publishmessage(Jedis jedis, String channel,String message) {
		// 发布消息到 "my-channel"
        jedis.publish(channel, message);
        System.out.println("Message published to my-channel");
        
//        System.out.println("Message published to my-channel");
	}
    public static void main(String[] args) {
        Jedis jedis = new Jedis("redis://127.0.0.1:6379/0"); // 假设 Redis 服务器运行在本地
        System.out.println("Connected to Redis server!");

        // 发布消息到 "my-channel"
        jedis.publish("my-channel", "Hello, Redis Publish/Subscribe!");
        System.out.println("Message published to my-channel");

        jedis.close();
    }
}
```

2. 订阅消息

```
package message;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPubSub;

public class RedisSubscriber extends JedisPubSub {
    @Override
    public void onMessage(String channel, String message) {
        System.out.println("Received message: " + message + " on channel: " + channel);
    }
    
    public void listening(Jedis jedis,String channel) {
    	//Jedis jedis = new Jedis("redis://127.0.0.1:6379/0"); // 假设 Redis 服务器运行在本地
        System.out.println("Connected to Redis server!");

        // 订阅 "my-channel"
        new Thread(new Runnable() {
            @Override
            public void run() {
                jedis.subscribe(new RedisSubscriber(), channel);
            }
        }).start();
        
        System.out.println("listening: my-channel");
    }

    public static void main(String[] args) {
        Jedis jedis = new Jedis("redis://127.0.0.1:6379/0"); // 假设 Redis 服务器运行在本地
        System.out.println("Connected to Redis server!");

        // 订阅 "my-channel"
        new Thread(new Runnable() {
            @Override
            public void run() {
                jedis.subscribe(new RedisSubscriber(), "my-channel");
            }
        }).start();
    }
}
```

问题：这个要怎么测试呢？



#### 练习：任务队列

1. 任务生产者

```
package task;

import redis.clients.jedis.Jedis;

public class TaskProducer {
    private Jedis jedis;
    private final String queueKey = "task-queue";

    public TaskProducer(Jedis jedis) {
        this.jedis = jedis;
    }

    public void produceTask(String task) {
        jedis.lpush(queueKey, task); // 将任务推入队列左侧（头部）
        System.out.println("Produced task: " + task);
    }

    public static void main(String[] args) {
        Jedis jedis = new Jedis("redis://127.0.0.1:6379/0"); // 假设 Redis 服务器运行在本地
        TaskProducer producer = new TaskProducer(jedis);

        // 生产一些任务
        producer.produceTask("Task 1");
        producer.produceTask("Task 2");
        producer.produceTask("Task 3");
        producer.produceTask("Task 4");

        // 关闭连接（在实际应用中，你可能希望在应用程序结束时关闭连接）
        jedis.close();
    }
}
```

运行后，命令行显示

```
Produced task: Task 1
Produced task: Task 2
Produced task: Task 3
Produced task: Task 4
```

说明有四个任务放进了队列了；

同时redis服务器可以看到记录；

2. 任务消费

```
package task;

import redis.clients.jedis.Jedis;

public class TaskConsumer implements Runnable {
    private Jedis jedis;
    private final String queueKey = "task-queue";

    public TaskConsumer(Jedis jedis) {
        this.jedis = jedis;
    }

    @Override
    public void run() {
        while (true) { // 无限循环，直到程序被外部终止
            String task = jedis.brpop(0, queueKey).get(1); // 从队列右侧（尾部）取出并移除任务，阻塞直到有任务可用
            if (task != null) {
                System.out.println("Consumed task: " + task);
                // 在这里处理任务...
                // ...
            }
        }
    }

    public static void main(String[] args) {
        Jedis jedis = new Jedis("redis://127.0.0.1:6379/0"); // 假设 Redis 服务器运行在本地
        TaskConsumer consumer = new TaskConsumer(jedis);

        // 在新的线程中运行消费者，以便它可以持续地从队列中取出任务
        new Thread(consumer).start();
    }
}
```

运行后，刚才生产的任务会被消费

```
Consumed task: Task 1
Consumed task: Task 2
Consumed task: Task 3
Consumed task: Task 4
```

