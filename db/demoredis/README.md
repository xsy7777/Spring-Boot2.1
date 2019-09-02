# 易筋SpringBoot 2.1 | 第九篇：SpringBoot使用Redis内存数据库
写作时间：2019-01-29<br>
Spring Boot: 2.1 ,JDK: 1.8, IDE: IntelliJ IDEA, MySQL 8.0.13
# Redis 介绍
Redis是目前业界使用最广泛的内存数据存储。相比memcached，Redis支持更丰富的数据结构，例如hashes, lists, sets等，同时支持数据持久化。除此之外，Redis还提供一些类数据库的特性，比如事务，HA，主从库。可以说Redis兼具了缓存系统和数据库的一些特性，因此有着丰富的应用场景。

# 安装 Redis Server环境
1. 安装Redis
```shell
brew install redis

```
2. 启动Redis服务器：
```shell
redis-server

```
启动成功日志：
```shell
20068:C 29 Jan 2019 09:35:17.742 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
20068:C 29 Jan 2019 09:35:17.743 # Redis version=5.0.3, bits=64, commit=00000000, modified=0, pid=20068, just started
20068:C 29 Jan 2019 09:35:17.743 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
20068:M 29 Jan 2019 09:35:17.744 * Increased maximum number of open files to 10032 (it was originally set to 256).
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 5.0.3 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 20068
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

20068:M 29 Jan 2019 09:35:17.747 # Server initialized
20068:M 29 Jan 2019 09:35:17.747 * DB loaded from disk: 0.000 seconds
20068:M 29 Jan 2019 09:35:17.747 * Ready to accept connections

```
3.  启动Redis客户端
```shell
redis-cli

```
> 操作命令都在客户端显示

4. 操作字符串
```shell
127.0.0.1:6379> set name zgpeace
OK
127.0.0.1:6379> get name
"zgpeace"

```
5. Hashes 哈希值
```shell
127.0.0.1:6379> hmset student username zgpeace password ThePassword age 18
OK
127.0.0.1:6379> hgetall student
1) "username"
2) "zgpeace"
3) "password"
4) "ThePassword"
5) "age"
6) "18"
```
6. Lists 列表
```shell
127.0.0.1:6379> lpush classmates JackMa
(integer) 1
127.0.0.1:6379> lpush classmates Lucy
(integer) 2
127.0.0.1:6379> lrange classmates 0 10
1) "Lucy"
2) "JackMa"

```
7. Redis有序集合
```shell
127.0.0.1:6379> zadd db 1 redis
(integer) 1
127.0.0.1:6379> zadd db 2 mongodb
(integer) 1
127.0.0.1:6379> zadd db 3 mysql
(integer) 1
127.0.0.1:6379> zadd db 4 oracle
(integer) 1
127.0.0.1:6379> zrange db 0 10 withscores
1) "redis"
2) "1"
3) "mongodb"
4) "2"
5) "mysql"
6) "3"
7) "oracle"
8) "4"

```
8. Redis发布订阅
启动客户端一```redis-cli```
监听两个channel，wechat和messages
```shell
subscribe wechat messages
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "wechat"
3) (integer) 1
1) "subscribe"
2) "messages"
3) (integer) 2

```

开启新的控制台，启动客户端二```redis-cli```， 并发布消息
```shell
127.0.0.1:6379> publish wechat "Hello"
(integer) 1

```
客户端一收到消息
```shell
1) "message"
2) "wechat"
3) "Hello"

```
9. 更多命令参考[Redis官网](https://redis.io/commands)

# 工程建立
工程建立的时候，需要勾选NOSQL的Redis：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190129101053415.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pncGVhY2U=,size_16,color_FFFFFF,t_70)
参照教程[【SpringBoot 2.1 | 第一篇：构建第一个SpringBoot工程】](https://blog.csdn.net/zgpeace/article/details/85111272)新建一个Spring Boot项目，名字叫demoredis, 在目录`src/main/java/resources` 下找到配置文件`application.properties`，重命名为`application.yml`。

# 配置文件
### 数据库连接信息配置`src/main/resources/application.yml`：
```yml
spring:
  redis:
    host: localhost #服务器地址
    port: 6379      #服务器连接端口
    database: 1     #数据库索引(默认为0)
    password:       #服务器连接密码(默认为空)
```
# 测试访问
通过编写测试用例，设置Redis的key, value, 读取验证。
修改测试类`com.zgpeace.demoredis.DemoredisApplicationTests` 
```java
package com.zgpeace.demoredis;

import org.junit.Assert;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@SpringBootTest
public class DemoredisApplicationTests {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    @Test
    public void test() throws Exception {
        String key = "yourState";
        String value = "passion";
        stringRedisTemplate.opsForValue().set(key, value);
        Assert.assertEquals("passion", stringRedisTemplate.opsForValue().get(key));
    }

    @Test
    public void contextLoads() {
    }

}
```

### 注意必须启动redis-server，否则连接失败，
Terminal启动redis-server
```shell
% redis-server
```
在方法左侧，点击单元测试按钮，验证通过。

关掉redis-server的方法，先找到正在运行的pid
```shell
$ ps aux | grep redis
MyUser  8821   0.0  0.0  2459704    596   ??  S    4:54PM   0:03.40 redis-server *:6379

```
或者用 `ps -ef|grep redis`
手动kill掉pid
```shell
$ kill -9 8821
```

# 操作对象
Redis存储String类型，也可以存储对象，使用类似RedisTemplate<String, City>来初始化并进行操作。Spring Boot并不支持直接使用，需要自己实现RedisSerializer<T>接口来对传入对象进行序列化和反序列化，下面通过一个实例来完成对象的读写操作。

## 修改pom.xml Redis的依赖
Spring Boot 2.0中spring-boot-starter-data-redis默认使用Lettuce方式替代了Jedis。使用Jedis的话先排除掉Lettuce的依赖，然后手动引入Jedis的依赖。
**pom.xml** 依赖项**spring-boot-starter-data-redis**修改
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <exclusion>
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
        
```

## 创建Bean对象
>对象需要在Redis中的语言跟Java语言之间转换，所以需要序列化
>com.zgpeace.demoredis.bean.City
```java
package com.zgpeace.demoredis.bean;

import java.io.Serializable;

public class City implements Serializable {

    private static final long serialVersionUID = -1L;

    private String name;
    private String state;
    private String country;

    public City(String name, String state, String country) {
        this.name = name;
        this.state = state;
        this.country = country;
    }

    // getter setter ..
}


```
## 通用对象序列化类
> com.zgpeace.demoredis.dao.RedisObjedctSerializer
```java
package com.zgpeace.demoredis.dao;

import org.springframework.core.convert.converter.Converter;
import org.springframework.core.serializer.support.DeserializingConverter;
import org.springframework.core.serializer.support.SerializingConverter;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.SerializationException;

public class RedisObjedctSerializer implements RedisSerializer<Object> {

    private Converter<Object, byte[]> serailizer = new SerializingConverter();
    private Converter<byte[], Object> deserializer = new DeserializingConverter();

    static final byte[] EMPTY_ARRAY = new byte[0];

    @Override
    public byte[] serialize(Object o) throws SerializationException {
        if (o == null) {
            return EMPTY_ARRAY;
        }
        try {
            return serailizer.convert(o);
        } catch (Exception ex) {
            return EMPTY_ARRAY;
        }
    }

    @Override
    public Object deserialize(byte[] bytes) throws SerializationException {
        if (isEmpty(bytes)) {
            return null;
        }

        try {
            return deserializer.convert(bytes);
        } catch (Exception ex) {
            throw new SerializationException("Connot deserialize", ex);
        }
    }

    private boolean isEmpty(byte[] data) {
        return (data == null || data.length == 0);
    }
}

```

## 配置针对City的RedisTemplate实例
>com.zgpeace.demoredis.dao.CityRedisConfig
```java
package com.zgpeace.demoredis.dao;

import com.zgpeace.demoredis.bean.City;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.StringRedisSerializer;

@Configuration
public class CityRedisConfig {

    @Bean
    RedisConnectionFactory jedisConnectionFactory() {
        return new JedisConnectionFactory();
    }

    @Bean
    public RedisTemplate<String, City> redisTemplate(RedisConnectionFactory jedisConnectionFactory) {
        RedisTemplate<String, City> template = new RedisTemplate<String, City>();
        template.setConnectionFactory(jedisConnectionFactory);
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new RedisObjedctSerializer());
        return template;
    }
}

```
## 添加新的测试用例
>com.zgpeace.demoredis.DemoredisApplicationTests
```java
@Test
    public void testRedisCity() throws Exception {
        City city = new City("San Jose", "California", "America");
        cityRedisTemplate.opsForValue().set(city.getName(), city);

        city = new City("Vancouver", "British Columbi", "Canada");
        cityRedisTemplate.opsForValue().set(city.getName(), city);

        Assert.assertEquals("California", cityRedisTemplate.opsForValue().get("San Jose").getState());
        Assert.assertEquals("Canada", cityRedisTemplate.opsForValue().get("Vancouver").getCountry());
    }

```

# 监听消息
## 接收消息对象
消息应用都有消息接收方，消息发送方。创建一个消息接收方，实现打印接收到的消息。
>com.zgpeace.demoredis.bean.Receiver
```java
package com.zgpeace.demoredis.bean;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;

import java.util.concurrent.CountDownLatch;

public class Receiver {

    private static final Logger LOGGER = LoggerFactory.getLogger(Receiver.class);

    private CountDownLatch latch;

    @Autowired
    public Receiver(CountDownLatch latch) {
        this.latch = latch;
    }

    public void receiveMessage(String message) {
        LOGGER.info("Received <" + message + ">");
        latch.countDown();
    }

}

```
>消息接收对象是个POJO，定义一个方法去接收消息。一旦注册对象为监听消息，可以取任何方法的名字。`receiveMessage`
## 注册一个监听者，并发送消息
**Spring Data Redis** 提供了所有组件在Redis之间发送和接收消息，主要配置一下信息：
1. 连接工厂connection factory
2. 消息监听容器message listener container
3. Redis template

Redis template发送消息，注册的接收消息对象会接收消息。连接工厂驱动前面两者连接到Redis server。

例子连接工厂用Spring Boot的RedisConnectionFactory，JedisConnectionFactory的一个实例。 连接工厂注入到消息监听对象和Redis template.
>com.zgpeace.demoredis.DemoredisApplication

```java
package com.zgpeace.demoredis;

import com.zgpeace.demoredis.bean.Receiver;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.listener.PatternTopic;
import org.springframework.data.redis.listener.RedisMessageListenerContainer;
import org.springframework.data.redis.listener.adapter.MessageListenerAdapter;

import java.util.concurrent.CountDownLatch;

@SpringBootApplication
public class DemoredisApplication {

    private static final Logger LOGGER= LoggerFactory.getLogger(DemoredisApplication.class);

    @Bean
    RedisMessageListenerContainer container(RedisConnectionFactory connectionFactory,
                                            MessageListenerAdapter listenerAdapter) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        container.addMessageListener(listenerAdapter, new PatternTopic("chat"));

        return container;
    }

    @Bean
    MessageListenerAdapter listenerAdapter(Receiver receiver) {
        return new MessageListenerAdapter(receiver, "receiveMessage");
    }

    @Bean
    Receiver receiver(CountDownLatch latch) {
        return new Receiver(latch);
    }

    @Bean
    CountDownLatch latch() {
        return new CountDownLatch(1);
    }

    @Bean
    StringRedisTemplate template(RedisConnectionFactory connectionFactory) {
        return new StringRedisTemplate(connectionFactory);
    }

    public static void main(String[] args) throws InterruptedException {

        ApplicationContext ctx = SpringApplication.run(DemoredisApplication.class, args);

        StringRedisTemplate template = ctx.getBean(StringRedisTemplate.class);
        CountDownLatch latch = ctx.getBean(CountDownLatch.class);

        LOGGER.info("Sending message...");
        template.convertAndSend("chat", "Hello from Redis!");

        latch.await();

        //System.exit(0);

    }

}

```
Terminal启动Redis server
```shell
% redis-server

```
运行结果
```shell
2019-01-31 09:49:34.007  INFO 50210 --- [           main] c.z.demoredis.DemoredisApplication       : Started DemoredisApplication in 2.073 seconds (JVM running for 2.625)
2019-01-31 09:49:34.008  INFO 50210 --- [           main] c.z.demoredis.DemoredisApplication       : Sending message...
2019-01-31 09:49:34.020  INFO 50210 --- [    container-2] com.zgpeace.demoredis.bean.Receiver      : Received <Hello from Redis!>

```
程序解析：
方法 `listenerAdapter` 注册为消息监听，并监听"chat" Topic. 因为接收方为POJO，所以需要 `addMessageListener` 把接收到的消息传递过去。 `listenerAdapter()` 实例配置去调用接收方方法 `receiveMessage()` 。

连接工厂和连接容器使应用可以监听消息。用Redis template去发送消息，StringRedisTemplate是实现了RedisTemplate, 针对Redis中的keys和values都是String。

`main()` 方法创建了**Spring application context**，context然后启动了监听容器。 **StringRedisTemplate** 在topic为"chat"发送消息"Hello from Redis!", 接收方打印消息。

# 分布式Session
分布式系统中，sessiong共享有很多的解决方案，其中托管到缓存中应该是最常用的方案之一，

Spring Session官方说明
Spring Session provides an API and implementations for managing a user’s session information.
## pom.xml引入依赖
```xml
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>

```
## Session配置类
>com.zgpeace.demoredis.dao.SessionConfig
```java
package com.zgpeace.demoredis.dao;

import org.springframework.context.annotation.Configuration;
import org.springframework.session.data.redis.config.annotation.web.http.EnableRedisHttpSession;

@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 86400*30)
public class SessionConfig {
}


```
maxInactiveIntervalInSeconds: 设置Session失效时间，使用Redis Session之后，原Boot的server.session.timeout属性不再生效
## 测试方法
>com.zgpeace.demoredis.web.SessionController
```java
package com.zgpeace.demoredis.web;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpSession;
import java.util.UUID;

@RestController
public class SessionController {

    @RequestMapping(value = "/uid", method = RequestMethod.GET)
    public String uid(HttpSession session){
        UUID uid = (UUID)session.getAttribute("uid");
        if (uid == null) {
            uid = UUID.randomUUID();
        }
        session.setAttribute("uid", uid);
        return session.getId();
    }
}

```
Terminal调用接口
```shell
% curl http://localhost:8080/uid
f2b87954-2d97-4029-909f-636dfe584d9a%   

```
Terminal开启Redis client，查看生产的sessionId, 和过期时间
```shell
% redis-cli
127.0.0.1:6379> keys '*sessions*'
1) "spring:session:sessions:expires:f2b87954-2d97-4029-909f-636dfe584d9a"
2) "spring:session:sessions:f2b87954-2d97-4029-909f-636dfe584d9a"

```

>如何在两台或者多台中共享session
其实就是按照上面的步骤在另一个项目中再次配置一次，
启动后自动就进行了session共享。

# 总结
恭喜你！学会了操作Redis字符串，对象，以及监听消息。
代码地址：https://github.com/zgpeace/Spring-Boot2.1/tree/master/demoredis

# 参考

https://blog.csdn.net/forezp/article/details/70991675
https://blog.csdn.net/forezp/article/details/61471712
https://spring.io/guides/gs/messaging-redis/
https://www.cnblogs.com/ityouknow/p/5748830.html
http://blog.didispace.com/springbootredis/
https://www.concretepage.com/questions/599
