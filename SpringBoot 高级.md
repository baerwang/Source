# SpringBoot 高级

## JSR 107

Java Caching定义了五个核心接口，分别是CachingProvider,CacheManager,Cache,Entry和Expir。

- CachingProvider 定义了创建，配置，获取，管理和控制多个CacheManager。一个应用可以在运行期访问多个CachingProvider。
- CacheManager定义了创建，配置，获取，管理和控制多个唯一命名的Cache，这些Cache存在于CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。
- Cache是一个类似Map的数据结构临时存储以key为索引的值。一个Cache仅被一个CacheManager所拥有的。
- Entry是一个存储在Cache中的key-value对。
- Expiry每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态。一旦过期，条目将不可访问，更新和删除。缓存有效期可以通过ExpiryPolicy设置。

## 缓存抽象

Spring从3.1开始定义了org.springframework.cache.Cache和org.springframework.cache.CacheManager接口来统一不同的缓存技术。

- 并支持使用JCache(JSR-107)注解简化我们的开发。

- Cache接口为缓存的组件规范定义，包含缓存的各种操作集合。

- Cache接口下Spring提供了各种xxCache的实现。如RedisCache，EhCacheCache，ConsurrentMapCache等。

- 每次调用需要缓存功能的方法时，Spring会检查指定参数的指定的目标方法是否已经被调用偶。如果有就直接从缓存中获取方法调用后的结果，如果没有就调用方法并缓存结果后返回给用户。下次调用直接从缓存中获取。

- 使用Spring缓存抽象时我们需要关注以下两点：

  ​		确定方法需要被缓存以及他们的缓存策略。

  ​		从缓存中读取之前缓存存储的数据。

## 重要的概念和缓存注解

| Cache          | 缓存接口，定义缓存操作。实现有：RedisCache，EhCacheCache，ConcurrentMapCache等 |
| -------------- | ------------------------------------------------------------ |
| CacheManager   | 缓存管理器，管理各种缓存(Cache)组件                          |
| @Cacheable     | 主要针对方法的配置，能够根据方法的请求参数对其结果进行缓存   |
| @CacheEvict    | 清空缓存                                                     |
| @CachePut      | 保证方法被调用，又喜欢结果被缓存                             |
| @EnableCaching | 开启基于注解的缓存                                           |
| keyGenerator   | 缓存数据时key生成策略                                        |
| serialize      | 缓存数据时value序列化策略                                    |

![](D:\images\2020-2-1\20200201162453.PNG)

![20200201162522](D:\images\2020-2-1\20200201162522.PNG)

## 体检SpringBoot缓存

1. 开启基于注解的缓存	@EnableCaching

   ```java
   @SpringBootApplication
   @EnableCaching
   @MapperScan("com.baerwang.cache.mapper")
   public class Boot01CacheApplication {
   
       public static void main(String[] args) {
           SpringApplication.run(Boot01CacheApplication.class, args);
       }
   }
   ```
   

   
2. 标注缓存注解即可

```java
    /**
     * 将方法的运行结果进行缓存：以后再要相同的数据，从缓存中获取，不用调用方法
     *  CacheManager管理多个Cache组件，对缓存的真正CRUD操作在Cache组件中，每一个缓存组件有自己的唯一一个名字
     *  属性：
     *      cacheNames/value ：指定缓存组件的名字.将方法的返回结果放在哪个缓存中，是数组的方式，可以指定多个缓存。
     *      key ：缓存数据使用的key,可以用它来指定。默认是方法参数的值 1 方法的返回值
     *      编写SpEL : #id；参数id的值,#root.methodName,#a0代表参数-1
     *      keyGenerator : key的生成器，可以自己指定key的生成器的组件id
     *          key/keyGenerator : 二选一使用
     *      cacheManager： 指定缓存管理器；或者cacheResolver指定获取解析器
     *      condition： 指定符合条件的情况下才缓存；
     *      unless： 否定缓存；当unless指定的条件为true，返回值为空不缓存，不为空设置缓存(unless = "#result == null")
     *              unless = "#a0==1" ,#a0为参数1，true为不缓存，false为缓存
     *      sync ： 是否使用异步模式
     * @param id
     * @return
     */
    @Override
    @Cacheable(cacheNames = {"emp"}, unless = "#result == null")
    public Employee getById(Integer id) {
        System.out.println("查询员工：" + id + "号");
        return employeeMapper.getById(id);
    }
```

@Cacheable使用keyGenerator

```java
    @Override
    @Cacheable(cacheNames = {"emp"}, keyGenerator = "myKeyGenerator")
    public Employee getById(Integer id) {
        System.out.println("查询员工：" + id + "号");
        return employeeMapper.getById(id);
    }
```

创建配置类

```java
@Configuration
public class MyConfiguration {

    @Bean("myKeyGenerator")
    public KeyGenerator keyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                System.out.println("被注册");
                return method.getName() + "-" + Arrays.asList(params);
            }
        };
    }
}
```

```properties
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/spring_cache?useUnicode=true&characterEncoding=utf8&useSSL=true&serverTimezone=UTC
    username: root
    password: root
  profiles:
    active: dev
mybatis:
  configuration:
    map-underscore-to-camel-case: true
logging:
  level:
    com.baerwang.cache.mapper.EmployeeMapper: debug
```

### @Cacheable 运行流程

1. 方法运行之前，先去查询Cache(缓存组件)，按照cacheNames指定的名字获取。
   	（CacheManager先获取相应的缓存)，第一次获取缓存，如果没有Cache组件会自动创建 。
2. 去Cache中寻找缓存的内容，使用一个key，默认的就是方法的参数。
   		key是按照某种策略生成的。默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key。
      	SimpleKeyGenerator生成key的默认策略：
      		如果没有参数，key=new SimpleKey();
      		如果有一个参数，key=参数的值
      		如果有多个参数，key=new SimpleKey(params);
3. 没有查到缓存就调用目标方法。
4. 将目标返回的结果，放进缓存中。
5. @Cacheable标注的方法执行之前先来检查缓存中有没有整个数据，默认按照参数的值作为key去查询缓存，如果没有就运行方法并将结果放入缓存中。以后再来调用就可以直接使用缓存中的数据。

核心：
	使用CacheManager[ConcurrentMapCacheManager]按照名字得到Cache[ConcurrentMapCache]组件
	key使用keyGenerator生成的，默认是SimpleKeyGenerator

### @CachePut 

```java
    /**
     * @CachePut： 即调用方法，又更新缓存数据
     * 修改了数据库的某个数据，同时更新缓存
     *  运行时机 ：
     *      先调用目标方法
     *      将目标方法的结果缓存起来
     *  key = "#employee.id" : 使用传入的参数的员工id
     *  key = "#result.id" : 使用返回后的id
     *       @Cacheable 的key不能使用#result
     * @param employee
     * @return
     */
    @Override
    @CachePut(value = "emp", key = "#result.id")
    public Employee updEmp(Employee employee) {
        System.out.println("updEmp：" + employee);
        employeeMapper.updEmp(employee);
        return employee;
    }
```

### @CacheEvict

```java
    /**
     * @CacheEvict 缓存数据
     *      key : 指定要清除的数据
     *      allEntries : true 清空所有缓存数据，false 清空指定的缓存数据
     *      beforeInvocation = false : 缓存的清除是否再方法之前执行
     *          默认代表缓存数据清理操作是在方法执行之后执行的，如果出现异常缓存是不会清理的
     *      beforeInvocation = true : 清除缓存操作是在方法运行之前执行，无论方法是否出现异常，缓存都会清除
     * @param id
     */
    @Override
    @CacheEvict(value = "emp", key = "#id", allEntries = true)
    public void delEmp(Integer id) {
        System.out.println("delEmp：" + id);
        employeeMapper.delEmp(id);
    }
```

### @Caching

```java
    /**
     *  @Cacheable：
     *      比如查询模糊查询，查找数据，并指定key+id，这样下次用id查询就直接从缓存里获取
     *  @CachePut   
     *      查询数据并修改缓存
     *
     * @param lastName
     * @return
     */
    @Override
    @Caching(
            cacheable = {
                    @Cacheable(value = "emp", key = "#lastName")
            }, put = {
            @CachePut(value = "emp", key = "#result.id")
    }
    )
    public Employee getEmpByLastName(String lastName) {
        return employeeMapper.getEmpByLastName(lastName);
    }
```

### @CacheConfig

```java
/**
* 抽取缓存的公共配置
*/
@CacheConfig(cacheNames = "emp")
public class xxxServiceImpl implements xxxService{}
```

## Spring-cache与redis整合

这里我是用 docker 下载的redis

```dockerfile
#下载
docker pull redis
#运行
docker run -p 6379:6379 -d redis:latest myredis
#下次启动运行
docker start myredis
#进入redis客户端
docker exec -it redis redis.cli
#查看是否连接上，出现 "PONG" 就表示运行成功
ping
```

​	导入maven

```xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

```yml
spring:
  redis:
    host: 192.168.x.x # ip地址
    port: 6379		 # redis 端口
    # password:
    database: 1 # Redis 数据库号，默认为 0 。
    timeout: 0 # Redis 连接超时时间，单位：毫秒。
```

测试：查询员工的数据添加redis中

```java
    @Autowired
    private RedisTemplate redisTemplate;
	
	 @Test
    public void test() {
        Employee byId = employeeMapper.getById(1);
		/*添加*/
        redisTemplate.opsForValue().set("emp",byId);
        /*查询*/
        Object emp = redisTemplate.opsForValue().get("emp");
        System.out.println(emp);
    }
```

格式化Json，添加配置类

```java
@Configuration
@EnableCaching  //开启基于注解的缓存
public class MyRedisConfiguration extends CachingConfigurerSupport {

    @Resource
    private LettuceConnectionFactory lettuceConnectionFactory;

    /**
     * 配置CacheManager
     *
     * @return
     */
    @Bean
    public CacheManager cacheManager() {
        /*设置redis序列化器*/
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);

        /*解决查询缓存转换异常的问题*/
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        /* 此项必须配置，否则会报java.lang.ClassCastException: java.util.LinkedHashMap cannot be cast to XXX*/
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

         /*配置序列化（解决乱码的问题）*/
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                /*设置缓存失效时间*/
                .entryTtl(Duration.ofSeconds(0))
                /*设置key*/
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
                /*设置value*/
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
                /*如果是空值不缓存*/
                .disableCachingNullValues();
        RedisCacheManager cacheManager = RedisCacheManager.builder(lettuceConnectionFactory)
                .cacheDefaults(config)
                .build();
        return cacheManager;
    }


    /**
     * RedisTemplate配置，默认使用JDK的序列化机制，存储二进制字节码，自定义序列化
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        /*使用Jackson2JsonRedisSerialize 替换默认序列化*/
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(
                Object.class);

        /*解决查询缓存转换异常的问题*/
        ObjectMapper objectMapper = new ObjectMapper();

        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        /* 此项必须配置，否则会报java.lang.ClassCastException: java.util.LinkedHashMap cannot be cast to XXX*/
        objectMapper.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);

        /*配置redisTemplate*/
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<String, Object>();
        redisTemplate.setConnectionFactory(lettuceConnectionFactory);
        RedisSerializer<?> stringSerializer = new StringRedisSerializer();

        /*key采用 String 的序列化方式*/
        redisTemplate.setKeySerializer(stringSerializer);
        /*value 采用 jackson 的序列化方式*/
        redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
        /*hash 的 key 采用String的序列化方式*/
        redisTemplate.setHashKeySerializer(stringSerializer);
        /*hash 的 value 采用jackson序列化方式*/
        redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }
}
```

配上上面写的配置类就可以只用Cache的注解了，它把里面的数据缓存到redis里面了

- 原理：CacheManager==Cache 缓存组件来实际给缓存中存取数据
- 引入redis的starter，容器中保存的是RedisCacheManager。
- RedisCacheManager帮我们创建RedisCache来作为缓存组件。
- RedisCache通过操作redis缓存数据
- 默认保存数据 key-value 都是Object。利用序列化来保存
  		引入了redis的starter，cacheManager变为RedisCacheManager。
    		默认创建的 RedisCacheManager 操作redis的时候使用的是 RedisTemplate<Object,Object>
    		RedisTemplate<Object,Object> 是默认使用 jdk的序列化机制(JdkSerializationRedisSerializer)。

## SpringBoot 与 消息

### 概述

1. 大多应用中，可通过消息服务中间件来提升系统异步通信，扩展解耦能力
2. 消息服务中两个重要概念：
   		消息代理(message broker) 和 目的地(destination)
   当消息发送者发送消息以后，将由消息代理接管，消息代理保证消息传递到指定目的地。
3. 消息队列主要有两种形式的目的地
   		队列(queue) ： 点对点消息通信(point-to-point)
      		主题(topic)： 发布(publish) / 订阅 (subscribe) 消息通信

### 异步处理

![](D:\images\2020-2-1\20200202181848.PNG)

### 应用解耦

![](D:\images\2020-2-1\20200202181922.PNG)

### 流量削峰

![](D:\images\2020-2-1\20200202181930.PNG)

### 点对点式
​		消息发送者发送消息，消息代理将其放入一个队列中，消息接受者从队列中获取消息内容，消息读取后被移出队列。
​		消息只有唯一的发送者和接受者，但并不是说只有一个接受者。

### 发布订阅式

​		发送者(发布者)发送消息到主题，多个接受者(订阅者)监听(订阅)这个主题，那么就会在消息到达时同时收到消息。

### JMS(Java Message Service) Java消息服务

​		基于JVM的消息代理的规范。ActiveMQ，HornetMQ是JMS实现的。

### AMQP(Advanced Message Queuing Protocol)

​		高级消息队列协议，也是一个消息代理的规范，兼容JMS。
​		RabbitMQ是AMQP的实现。

![](D:\images\2020-2-1\20200202184047.PNG)

### Spring支持

- Spring-jms提供了对JMS的支持
- Spring-rabbit提供了对AMQP的支持
- 需要ConnectionFactory的实现来连接消息代理
- 提供JmsTemplate，RabbitTemplate来发送消息
- @JmsListener(JMS)，@RabbitListener(AMQP)注解在方法上监听消息代理发布的消息
- @EnableJms，@EnableRabbit开启支持

### SpringBoot 自动配置

- JmsAutoConfiguration
- RabbitAutoConfiguration

## RabbitMQ

### 简介

RabbitMQ是一个由erlang开发的AMQP(Advanved Message Queue Protocol)的开源实现。

### 核心概念

消息，消息是不具名，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key(路由键)，priority(相对于其他消息的优先权)，delivery-mode(指出该消息可能需要持久性存储)等。

#### Publisher

消息的生产者，也是一个向交换器发布消息的客户端应用程序。

#### Exchange

交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。
Exchange有4种类型：direct(默认)，fanout，topic，和headers，不同类型的Exchange转发消息的策略有所区别。

#### Queue

消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。

#### Binding

绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。

#### Connection

网络连接，比如一个TCP连接。

#### Channel

信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP链接内的虚拟连接，AMQP命令都是通过信道发出去的，不管是发布消息，订阅队列还是接收消息，这些动作都是通过信道完成的。因为对于操作系统来说建立和销毁TCP都是非常昂贵的开销，所以引入了信道的概念，以复用一条TCP连接。

#### Consumer

消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。

#### Virtual Host

虚拟主机，表示一批交换器，消息队列和相关对象。虚拟主机是共享相同的身份认证加密环境的独立服务器域。每个vhost本质就是一个mini版的RabbitMQ服务器，拥有自己的队列，交换器，绑定和权限的机制。vhost是AMQP概念的基础，必须在连接时指定，RabbitMQ默认的vhost是  /  。

#### Broker

表示消息队列服务器实体

![](D:\images\2020-2-1\20200202202435.PNG)

## RabbitMQ运行机制

### AMQP中的消息路由

​		AMQP中消息的路由过程和Java开发者熟悉的JMS存在一些差别，AMQP中增加了Exchange和Binding的角色。生产者把消息发布到Exchange上，消息最终到达队列并被消费者接收，而Binding决定交换器的消息应该发送到那个队列上。

![](D:\images\2020-2-1\202022202816.png)

------

### Exchange类型

​		Exchange分发消息时根据类型不同的分发策略有区别，目前共有四种类型：direct，fanout，topic，headers。headers匹配AMQP消息的header而不是路由键，headers交换器和direct交换器完全一致，但性能查很多，目前几乎用不到了，所以直接看另外三种类型

------

### 路由键方式转发消息

![](D:\images\2020-2-1\202022203151.png)

​		消息中的路由键(routing key)如果和Binding中的binding key一致，交换器就将消息发到对应的队列中。路由键与队列完全匹配，如果一个队列绑定到交换机要求路由键为"dog"，则只转发routing key 标志为 "dog"的消息，它不会转发"dog.puppy"，也不会转发"dog.guard"等等。它是完全匹配，单播的模式。

------

### 广播方式转发消息

![](D:\images\2020-2-1\202022203557.png)

​		每个发到fanout类型交换器的消息都会分到所有绑定的队列上去。fanout交换器不处理路由键，只是简单的队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout类型转发的消息是最快的。

---

### 主题匹配方式转发消息

![](D:\images\2020-2-1\202022203946.png)

​		topic交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。它将路由键和绑定键的字符串切分成单词，这些单词之间用点隔开。它同样也会识别两个通配符：符号 " # " 和符号 " * " 。" # "匹配0个或多个单词，" * "匹配一个单词。

---

## RabbitMQ整合

这里我是用 docker 下载的RabbitMQ

```dockerfile
#下载
docker pull rabbitmq:3-management
#运行并设置 image ID，是下载容器的ID
docker run -d -p 5672:5672 -p 15672:15672 --name myrabbitmq image ID
#下次启动
docker start myrabbitmq
#访问rabbitmq主页，你的虚拟机ip地址加上rabbitmq端口号
192.168.x.x:15672
#用户名和密码
guest
```

创建SpringBoot-RabbitMQ
		pom.xml文件

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

```properties
spring:
  rabbitmq:
    host: 192.168.x.x	#ip地址
    username: guest		#用户名
    password: guest		#密码
```

### 自动配置

- RabbitAutoConfiguration
- 有自动配置链接工厂 ：ConnectionFactory
- RabbitProperties ：封装了 RabbitMQ的配置
- RabbitTemplate ：给RabbitMQ发送和接收消息
- AmqpAdmin ：RabbitMQ系统管理功能组件
- AmqpAdmin ：创建和删除Queue，Exchange，Binding
- @EnableRabbit + @RabbitListener ： 监听消息队列的内容

### 发送消息(单播，点对点)

```java
    @Autowired
    private RabbitTemplate rabbitTemplate;

	@Test
    public void send() {
        /*Message需要自己构造一个，定义消息体内容和消息头
        rabbitTemplate.send(exchange,routeKey,message);*/

        /*Object默认当成消息体，只需要传入要发送的对象，自动序列化发送给rabbitmq
        rabbitTemplate.convertAndSend(exchange,routeKey,object);*/

        Map<String, Object> map = new HashMap<>();
        map.put("msg", "first message");
        map.put("data", Arrays.asList("hello", 123, true));
        /*对象被默认序列化发送出去*/
        rabbitTemplate.convertAndSend("exchange.direct", "baerwang.news", map);
    }
```

### 接收消息

```java
    @Test
    public void receiveMsg() {
        Object o = rabbitTemplate.receiveAndConvert("baerwang.news");
        System.out.println(o.getClass());
        System.out.println(o);
    }
```

### 将 JSON 格式发送消息

#### 添加配置类

```java
@Configuration
public class MyAMQPConfig {

    /**
     * 将json格式发送出去
     * @return
     */
    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

#### JSON格式发送

```java
	@Test
    public void send() {
        /*对象被默认序列化发送出去*/
        rabbitTemplate.convertAndSend("exchange.direct", "baerwang.news", new Book("张三IT故事","张三"));
    }
```

#### 接收消息

```java
    @Test
    public void receiveMsg() {
        Book o = (Book) rabbitTemplate.receiveAndConvert("baerwang.news");
        System.out.println(o.getClass());
        System.out.println(o);
    }
```

## 广播发送消息

```java
    @Test
    public void fanOutSendMsg() {
        rabbitTemplate.convertAndSend("exchange.fanout", "", new Book("xx", "bb"));
    }
```

## 监听消息

```java
@Component
public class BookComponent {
    @RabbitListener(queues = "baerwang.news")
    public void receive(Book book) {
        System.out.println("收到消息 : " + book);
    }
}
```

```java
@EnableRabbit //开启基于注解的RabbitMQ模式
@SpringBootApplication
public class Boot02RabbitmqApplication {
    public static void main(String[] args) {
        SpringApplication.run(Boot02RabbitmqApplication.class, args);
    }
}
```

## 使用AMQP

### 创建Exchange

```java
    @Autowired
    private AmqpAdmin amqpAdmin;

	@Test
    public void createExchange() {
        amqpAdmin.declareExchange(new DirectExchange("amqpadmin.exchange"));
    }
```

### 创建Queue

```java
    @Test
    public void createQueue() {
        amqpAdmin.declareQueue(new Queue("amqpadmin.queue",true));
    }
```

### 创建绑定规则，消息和交换器之间的关联

```java
    @Test
    public void createBinding() {
        amqpAdmin.declareBinding(new Binding("amqpadmin.queue", Binding.DestinationType.QUEUE, "amqpadmin.exchange", "amqp.haha", null));
    }
```

## Spring Boot 与 任务

### 异步任务

在Java应用中，绝大多数情况下都是通过同步的方式来实现交互处理的。但是在处理与第三方系统交互的时候，容易造成响应迟缓的情况，之间大部分都是使用多线程来完成此类任务，其实在Spring 3.x之后，就一句内置了@Async来完美解决这个问题。

两个注解：
		**@EnableAysnc**，**@Aysnc**

```java
@Service
public class AsyncService {
    /* @Async 告诉 Spring 这是一个异步的方法。
    没有加async注解，会等待，返回数据，加了这个注解会直接返回数据，后台继续等待*/
    @Async
    public void hello() throws InterruptedException {
        Thread.sleep(3000);
        System.out.println("数据正在处理中");
    }
}
```

```java
@EnableAsync    //开启异步注解功能
@SpringBootApplication
public class Boot04TaskApplication {

    public static void main(String[] args) {
        SpringApplication.run(Boot04TaskApplication.class, args);
    }

}
```

### 定时任务

项目开发中经常需要执行一些定时任务，比如需要在每天凌晨时候，分析一次前一天的日志信息。Spring为我们提供了异步任务调度的方式，提供**TaskExecutor**，**TaskScheduler**接口。

两个注解：
		**@EnableScheduling**，**@Scheduled**
cron表达式：

| 字段 | 允许值                | 允许的特殊字符   |
| ---- | --------------------- | ---------------- |
| 秒   | 0-59                  | , - * /          |
| 分   | 0-59                  | , - * /          |
| 小时 | 0-23                  | , - * /          |
| 日期 | 1-31                  | , - *  ? / L W C |
| 月份 | 1-12                  | , - * /          |
| 星期 | 0-7或SUN-SAT 0,7是SUN | , - * / L C #    |

| 特殊 | 代表含义                   |
| ---- | -------------------------- |
| ,    | 枚举                       |
| -    | 区间                       |
| *    | 任意                       |
| /    | 步长                       |
| ？   | 日 / 星期冲突匹配          |
| L    | 最后                       |
| W    | 工作日                     |
| C    | 和calendar联系后计算过得值 |
| #    | 星期，4#2，第2个星期四     |

```tex
"0 0 10,14,16 * * ?" 每天上午10点，下午2点，4点
"0 0/30 9-17 * * ?" 朝九晚五工作时间内每半小时
"0 0 12 ? * WED" 表示每个星期三中午12点
"0 0 12 * * ?" 每天中午12点触发
"0 15 10 ? * *" 每天上午10:15触发
"0 15 10 * * ?" 每天上午10:15触发
"0 15 10 * * ? *" 每天上午10:15触发
"0 15 10 * * ? 2005" 2005年的每天上午10:15触发
"0 * 14 * * ?" 在每天下午2点到下午2:59期间的每1分钟触发
"0 0/5 14 * * ?" 在每天下午2点到下午2:55期间的每5分钟触发
"0 0/5 14,18 * * ?" 在每天下午2点到2:55期间和下午6点到6:55期间的每5分钟触发
"0 0-5 14 * * ?" 在每天下午2点到下午2:05期间的每1分钟触发
"0 10,44 14 ? 3 WED" 每年三月的星期三的下午2:10和2:44触发
"0 15 10 ? * MON-FRI" 周一至周五的上午10:15触发
"0 15 10 15 * ?" 每月15日上午10:15触发
"0 15 10 L * ?" 每月最后一日的上午10:15触发
"0 15 10 ? * 6L" 每月的最后一个星期五上午10:15触发
"0 15 10 ? * 6L 2002-2005" 2002年至2005年的每月的最后一个星期五上午10:15触发
"0 15 10 ? * 6#3" 每月的第三个星期五上午10:15触发
```


```java
@Component
public class ScheduledService {
    @Scheduled(cron = "0 * * * * MON-SAT")
    public void hello() {
        System.out.println("定时任务 ...");
    }
}
```

```java
@EnableScheduling   //开启基于注解定时任务功能
@SpringBootApplication
public class Boot04TaskApplication {
    public static void main(String[] args) {
        SpringApplication.run(Boot04TaskApplication.class, args);
    }

}
```

### 邮件任务

#### maven

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-mail</artifactId>
        </dependency>
```

#### application.yml


```properties
spring:
  mail:
    username: baerwang@126.com
    password: xxx
    host: smtp.126.com
```

#### test，发送mail

```java
	@Autowired
    private JavaMailSender mailSender;    

	@Test
    public void sendMail(){
        SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
        simpleMailMessage.setSubject("晚上不加班");
        simpleMailMessage.setText("明天放假一天");
        simpleMailMessage.setTo("xxx@162.com");		//接收者
        simpleMailMessage.setFrom("baerwang@126.com");	//发送者
        mailSender.send(simpleMailMessage);
    }
```

#### 创建一个复杂的消息邮件

```java
    @Test
    public void test() throws MessagingException {
        /*创建一个复杂的消息邮件*/
        MimeMessage mimeMessage = mailSender.createMimeMessage();
        MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage, true);

        /*邮件设置*/
        mimeMessageHelper.setSubject("晚上不加班");
        mimeMessageHelper.setText("<h1>明天放假一天</h1>",true);//true设置为样式显示，默认为false
        mimeMessageHelper.setTo("1780285721@qq.com");               //接收者
        mimeMessageHelper.setFrom("baerwang@126.com");              //发送者

        /*设置上传文件*/
        mimeMessageHelper.addAttachment("1.jpg",new File("D:\\upload\\2019-12-15\\0D3DF037D74640B8A4C440EFDC64CBA0.jpg"));
        mimeMessageHelper.addAttachment("2.jpg",new File("D:\\upload\\2019-12-15\\0D3DF037D74640B8A4C440EFDC64CBA0.jpg"));
        mailSender.send(mimeMessage);
    }
```

## Spring Boot 与 分布式

分布式，Dubbo / Zookeeper，Spring boot / Cloud

### 分布式应用

在分布式系统中，国内常用的zookeeper + dubbo 组合，而Spring Boot 推荐使用全栈的Spring，Spring Boot + Spring Cloud。

![](D:\images\2020-2-1\202023211029.png)

- 单一应用架构
  	当网站流量很小时，只需要一个应用，将所有功能都部署在一起，以减少部署节点和成本。此时，用于简化增删改查工作的数据访问框架(ORM)是关键。
- 垂直应用架构
  	当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的及格应用，以提升效率。此时，用于加速前端页面开发的Web框架(MVC)是关键的。
- 分布式服务架构
  	当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。此时，用于提高业务复用及整合的分布式服务框架(RPC)是关键。
- 流动计算架构
  	当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率。此时，用于提高机器利用率的资源调度和治理中心(SOA)是关键的。

### Zookeeper

Zookeeper是一个分布式，开放源码的分布式应用程序协调服务，也是一个分布式的应用程序一致性的服务软件。它的功能主要有分布式同步，配置维护，组服务，域名服务等。

### Dubbo

Dubbo是alibaba开源的分布式服务框架，它最大的特点是按照分层的方式来架构，使用这种方式可以使各个层之间的解耦合(或者最大限度地松耦合)。从服务模型的角度来看，Dubbo采用的是一种非常简单的模型，要么是提供方提供服务，要么是消费方消费服务，所以基于这一点可以抽象出服务提供方(Provider)和消费方(Consumer)两个角色。

![](D:\images\2020-2-1\202023213744.png)

### Dubbo+ZK部署环境

- 这里我是用 docker 下载的Zookeeper

  ```dockerfile
    #下载
    docker pull zookeeper
    #运行并设置 image ID，是下载容器的ID
    docker run --name zk01 -p 2181:2181 -d image Id
    #下次启动
    docker start zk01
    #启动zkCli.sh
    docker exec -it zk01 bash
    cd bid
    zkCli.sh
  ```

- 创建SpringBoot 命名为dubbo-api

  ```java
  package com.baerwang.api;
  
  public interface DemoService {
      String sayHello(String name);
  }
  
  ```

- 创建SpringBoot 命名为Provider

  ```java
  package com.baerwang.provider.service.impl;
  
  import com.alibaba.dubbo.config.annotation.Service;
  import com.baerwang.api.DemoService;
  
  @Service	//alibaba
  public class ProviderServiceImpl implements DemoService {
      @Override
      public String sayHello(String name) {
          System.out.println("Hello " + name);
          return "Hello " + name;
      }
  }
  
  ```

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
      <parent>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-parent</artifactId>
          <version>2.2.4.RELEASE</version>
          <relativePath/> <!-- lookup parent from repository -->
      </parent>
      <groupId>com.baerwang.provider</groupId>
      <artifactId>dubbo-provider</artifactId>
      <version>0.0.1-SNAPSHOT</version>
      <name>dubbo-provider</name>
      <description>Demo project for Spring Boot</description>
  
      <properties>
          <java.version>1.8</java.version>
      </properties>
      
      <dependencies>
          <dependency>
              <groupId>com.baerwang.api</groupId>
              <artifactId>dubbo-api</artifactId>
              <version>1.0-SNAPSHOT</version>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
          <dependency>
              <groupId>com.alibaba.boot</groupId>
              <artifactId>dubbo-spring-boot-starter</artifactId>
              <version>0.2.0</version>
          </dependency>
          <dependency>
              <groupId>com.github.sgroschupf</groupId>
              <artifactId>zkclient</artifactId>
              <version>0.1</version>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-test</artifactId>
              <scope>test</scope>
              <exclusions>
                  <exclusion>
                      <groupId>org.junit.vintage</groupId>
                      <artifactId>junit-vintage-engine</artifactId>
                  </exclusion>
              </exclusions>
          </dependency>
      </dependencies>
      <build>
          <plugins>
              <plugin>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-maven-plugin</artifactId>
              </plugin>
          </plugins>
      </build>
  </project>
  
  ```

  ```properties
  dubbo:
    application:
      name: dubbo-provider
    registry:
      address: 192.168.x.x:2181	# zookeeper ip地址 + 端口
      protocol: zookeeper
  
  server:
    port: 8081
  ```

  ```properties
  ###set log levels###
  log4j.rootLogger=info, stdout
  ###output to the console###
  log4j.appender.stdout=org.apache.log4j.ConsoleAppender
  log4j.appender.stdout.Target=System.out
  log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
  log4j.appender.stdout.layout.ConversionPattern=[%d{dd/MM/yy hh:mm:ss:sss z}] %t %5p %c{2}: %m%n
  ```
  
  ```java
  package com.baerwang.provider;
  
  import com.alibaba.dubbo.config.spring.context.annotation.EnableDubbo;
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  
  @EnableDubbo
  @SpringBootApplication
  public class DubboProviderApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(DubboProviderApplication.class, args);
      }
  }
  
  ```
  
- 创建SpringBoot 命名为Consumer

  ```java
  package com.baerwang.consumer.service;
  
  public interface ConsumerService {
      String nameConsumer(String name);
  }
  ```

  ```java
  package com.baerwang.consumer.service.impl;
  
  import com.alibaba.dubbo.config.annotation.Reference;
  import com.baerwang.api.DemoService;
  import com.baerwang.consumer.service.ConsumerService;
  import org.springframework.stereotype.Service;
  
  @Service		//spring
  public class ConsumerServiceImpl implements ConsumerService {
  
      @Reference	//alibaab
      private DemoService demoService;
  
      @Override
      public String nameConsumer(String name) {
          return demoService.sayHello(name);
      }
  }
  
  ```

  ```java
  package com.baerwang.consumer.controller;
  
  import com.baerwang.consumer.service.ConsumerService;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RestController;
  
  @RestController
  public class ConsumerController {
  
      @Autowired
      private ConsumerService consumerService;
  
      @RequestMapping("/consumer")
      public String consumerHello() {
          return consumerService.nameConsumer("SUCCESS");
      }
  }
  
  ```

  ```java
  package com.baerwang.consumer;
  
  import com.alibaba.dubbo.config.spring.context.annotation.EnableDubbo;
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  
  @EnableDubbo
  @SpringBootApplication
  public class DubboConsumerApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(DubboConsumerApplication.class, args);
      }
  }
  ```

  ```xml
  pom.xml,log.properties 和 provider一样
  ```

  ```properties
  dubbo:
    application:
      name: dubbo-consumer
    registry:
      address: 192.168.1.107:2181
      protocol: zookeeper
      check: false
  
  server:
    port: 8082
  ```

### Spring Cloud

Spring Cloud是一个分布式的整体解决方案。Spring Cloud 为开发者提供了在分布式系统(配置管理，服务发现，熔断，路由，微代理，控制总线，一次性token，全局锁，leader选举，分布式session，集群状态)中快速构建的工具，使用Spring Cloud的开发者可以快速的启动服务或构建应用，同时能够快速和云平台资源进行对接。

#### Spring Cloud分布式开发五大常用组件

- 服务发现——Netfilx Eureka
- 客户端负载均衡——Netflix—— Ribbon
- 断路器——Netflix Hystrix
- 服务网关——Netfilx Zuul
- 分布式配置——Spring Cloud Config

#### 创建三个项目

- cloud-eureka-server

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  	<modelVersion>4.0.0</modelVersion>
  	<parent>
  		<groupId>org.springframework.boot</groupId>
  		<artifactId>spring-boot-starter-parent</artifactId>
  		<version>2.2.4.RELEASE</version>
  		<relativePath/> <!-- lookup parent from repository -->
  	</parent>
  	<groupId>com.baerwang</groupId>
  	<artifactId>cloud-eureka-server</artifactId>
  	<version>0.0.1-SNAPSHOT</version>
  	<name>cloud-eureka-server</name>
  	<description>Demo project for Spring Boot</description>
  
  	<properties>
  		<java.version>1.8</java.version>
  		<spring-cloud.version>Hoxton.SR1</spring-cloud.version>
  	</properties>
  
  	<dependencies>
  		<dependency>
  			<groupId>org.springframework.cloud</groupId>
  			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
  		</dependency>
  
  		<dependency>
  			<groupId>org.springframework.boot</groupId>
  			<artifactId>spring-boot-starter-test</artifactId>
  			<scope>test</scope>
  			<exclusions>
  				<exclusion>
  					<groupId>org.junit.vintage</groupId>
  					<artifactId>junit-vintage-engine</artifactId>
  				</exclusion>
  			</exclusions>
  		</dependency>
  	</dependencies>
  
  	<dependencyManagement>
  		<dependencies>
  			<dependency>
  				<groupId>org.springframework.cloud</groupId>
  				<artifactId>spring-cloud-dependencies</artifactId>
  				<version>${spring-cloud.version}</version>
  				<type>pom</type>
  				<scope>import</scope>
  			</dependency>
  		</dependencies>
  	</dependencyManagement>
  
  	<build>
  		<plugins>
  			<plugin>
  				<groupId>org.springframework.boot</groupId>
  				<artifactId>spring-boot-maven-plugin</artifactId>
  			</plugin>
  		</plugins>
  	</build>
  
  </project>
  
  ```

  ```properties
  server:
    port: 8081
  eureka:
    instance:
      hostname: cloud-eureka-server # eureka实例的主机名
    client:
      register-with-eureka: false   # 不把自己注册到eureka
      fetch-registry: false         # 不从eureka上来获取服务的注册信息
      service-url:
        defaultZone: http://localhost:8081/eureka/
  ```

  ```java
  package com.baerwang;
  
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
  
  /**
   * 注册中心
   *     1. 配置Eureka信息
   *     2. @EnableEurekaServer
   */
  @EnableEurekaServer
  @SpringBootApplication
  public class CloudEurekaServerApplication {
      public static void main(String[] args) {
          SpringApplication.run(CloudEurekaServerApplication.class, args);
      }
  }
  
  ```

- cloud-provider

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
      <modelVersion>4.0.0</modelVersion>
      <parent>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-parent</artifactId>
          <version>2.2.4.RELEASE</version>
          <relativePath/> <!-- lookup parent from repository -->
      </parent>
      <groupId>com.baerwang</groupId>
      <artifactId>cloud-provider</artifactId>
      <version>0.0.1-SNAPSHOT</version>
      <name>cloud-provider</name>
      <description>Demo project for Spring Boot</description>
  
      <properties>
          <java.version>1.8</java.version>
          <spring-cloud.version>Hoxton.SR1</spring-cloud.version>
      </properties>
  
      <dependencies>
          <dependency>
              <groupId>org.springframework.cloud</groupId>
              <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
          </dependency>
  
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-web</artifactId>
          </dependency>
  
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-test</artifactId>
              <scope>test</scope>
              <exclusions>
                  <exclusion>
                      <groupId>org.junit.vintage</groupId>
                      <artifactId>junit-vintage-engine</artifactId>
                  </exclusion>
              </exclusions>
          </dependency>
      </dependencies>
  
      <dependencyManagement>
          <dependencies>
              <dependency>
                  <groupId>org.springframework.cloud</groupId>
                  <artifactId>spring-cloud-dependencies</artifactId>
                  <version>${spring-cloud.version}</version>
                  <type>pom</type>
                  <scope>import</scope>
              </dependency>
          </dependencies>
      </dependencyManagement>
  
      <build>
          <plugins>
              <plugin>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-maven-plugin</artifactId>
              </plugin>
          </plugins>
      </build>
  
  </project>
  
  ```

  ```properties
  server:
    port: 8083
  spring:
    application:
      name: cloud-provider
  eureka:
    instance:
      prefer-ip-address: true   # 注册服务的时候使用服务的ip地址
    client:
      service-url:
        defaultZone: http://localhost:8081/eureka/
  ```

  ```java
  package com.baerwang.service.impl;
  
  import com.baerwang.service.ProviderService;
  import org.springframework.stereotype.Service;
  
  @Service
  public class ProviderServiceImpl implements ProviderService {
      @Override
      public String getTicket() {
          return "china hello !";
      }
  }
  ```

  ```java
  package com.baerwang.controller;
  
  import com.baerwang.service.ProviderService;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RestController;
  
  @RestController
  public class ProviderController {
  
      @Autowired
      private ProviderService providerService;
  
      @RequestMapping("/ticket")
      public String getTicket() {
          System.out.println("Two server");
          return providerService.getTicket();
      }
  }
  ```

- cloud-consumer

  ```xml
  和 provider一样
  ```

  ```properties
  server:
    port: 9081
  spring:
    application:
      name: cloud-consumer
  eureka:
    instance:
      prefer-ip-address: true   # 注册服务的时候使用服务的ip地址
    client:
      service-url:
        defaultZone: http://localhost:8081/eureka/
  ```

  ```java
  package com.baerwang.controller;
  
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RestController;
  import org.springframework.web.client.RestTemplate;
  
  @RestController
  public class ConsumerController {
  
      @Autowired
      private RestTemplate restTemplate;
  
      @RequestMapping("/buy")
      public String buyTicket() {
          String forObject = restTemplate.getForObject("http://CLOUD-PROVIDER/ticket", String.class);
          return String.format("buy %S", forObject);
      }
  }
  
  ```

  ```java
  package com.baerwang;
  
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  import org.springframework.cloud.client.loadbalancer.LoadBalanced;
  import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
  import org.springframework.context.annotation.Bean;
  import org.springframework.web.client.RestTemplate;
  
  @EnableEurekaClient     /*开启发现服务功能*/
  @SpringBootApplication
  public class CloudConsumerApplication {
  
      public static void main(String[] args) {
          SpringApplication.run(CloudConsumerApplication.class, args);
      }
  
      @Bean
      @LoadBalanced   /*使用负载均衡*/
      public RestTemplate restTemplate() {
          return new RestTemplate();
      }
  }
  ```
  
- 先启动eureka，localhost:8081
- 后启动provider，localhost:8082,8083 ,这两个端口是一个项目，记得运行时候把端口都设置不一样，否则端口冲突报错误。还有别忘了把provider的controller里面购买票的输出的名称，防止不知道进行的是哪个端口，写这个是来区分注册服务的时候是那个端口。如何一个项目多运行呢？下面给你介绍
    ![](D:\images\2020-2-1\20200205132135.jpg)

- 在启动consumer,localhost:9081/buy，购买到票了不知道是谁的服务器，看后台就知道注册了那个服务

## Spring Boot 与监控管理

### 控制管理

通过引入spring-boot-starter-actuator，可以使用SpringBoot为我们提供的准备生产环境下的应用监控和管理功能。我们可以通过HTTP，JMX，SSH协议来进行操作，自动得到审计，健康及指标信息等。

- 步骤
  - 引入spring-boot-starter-actuator。
  - 通过http方式访问监控断点。
  - 可进行shutdown(POST 提交，此端点默认关闭)。

### 监控和管理端点

| 端点名      | 描述                        |
| ----------- | --------------------------- |
| autoconfig  | 所有自动配置信息            |
| auditevent  | 审计事件                    |
| beans       | 所有Bean的信息              |
| configprops | 所有配置属性                |
| dump        | 线程状态属性                |
| env         | 当前环境信息                |
| health      | 应用环境状况                |
| info        | 当前应用信息                |
| metrics     | 应用的各项指标              |
| mappings    | 应用@RequestMapping映射路径 |
| shutdown    | 关闭当前应用(默认关闭)      |
| trace       | 追踪信息(最新的http请求)    |

### 定制端点信息

- 定制端点一般通过endpoints+端点名+属性名来设置。
- 修改断端点id (endpoints.benas.id=mybeans)
- 开启远程应用关闭功能 (endpoints.shutdown.enabled=true)
- 关闭端点 (endpoints.beans.enabled=false)
- 开启所需端点
  - endpoints.enable=false
  - endpoints.benas.enabled=true
- 定制端点访问根路径
  - management.context-path=manage
- 关闭http端点
  - management.port=-1