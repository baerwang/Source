# RabbitMQ

## 简介

RabbitMQ是使用Erlang语言开发的开源消息队列系统，基于AMQP协议来实现。AMQP的主要特征是面向消息，队列，路由(包括点对点和发布/订阅)，可靠性，安全。AMQP协议更多用在企业系统内，对数据一致性，稳定性和可靠性要求很高的场景，对性能和吞吞吐量的要求还在其次。

## AMQP

什么是AMQP高级消息队列协议

AMQP定义：是具有现代特征的二进制协议。是一个提供统一消息服务的应用程序标准高级消息队列协议，是应用协议的一个开放标准，为面向消息的中间件设计。

## AMQP核心概念

Server：又称Broker，接收客户端的链接，实现AMQP实体服务。

Connection：连接，应用程序与Broker的网络连接。

Channel：网络通道，几乎所有的操作都在Channel中进行，Channel是进行消息读写的通道。客户端可建立在多个Channel，每个Channel代表一个会话任务。

Message：消息，服务器和应用程序之间传送的数据，由Properties和Body组成。Properties可以对消息进行修饰，比如消息的优先级，延迟等高级特性；Body则就是消息体内容。

Virtual host：虚拟地址，用于进行逻辑隔离，最上层的消息路由。一个Virtual Host里面可以有若干个Exchange和Queue，同一个Virtual Host里面不能有相同名称的Exchange或Queue。

Exchange：交换器，接收消息，根据路由键转发消息到绑定的队列。

Binding：Exchange和Queue之间的虚拟连接，binding中可以包含routing key。

Routing key：一个路由规则，虚拟机可用它来确定如何路由一个特定消息。

Queue：也称为Message Queue，消息队列，保存消息并将它们转发给消费者。

## 安装

我这里是docker安装的

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

## 用户以及vhost配置

### 添加用户

![](D:\images\2020-2-4\20200206224129.jpg)

### virtual hosts 管理

virtual hosts 相当于mysql的db

一般以 / 开头

### 对用户进行授权

![](D:\images\2020-2-4\20200206225032.jpg)

![20200206225056](D:\images\2020-2-4\20200206225056.jpg)

### 登录
![](D:\images\2020-2-4\20200206225500.jpg)

## Simple协议

![20200206231550](D:\images\2020-2-4\20200206231550.PNG)

### 创建maven项目

```xml	
   <dependencies>
        <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>4.0.2</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.30</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.5</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
        </dependency>
    </dependencies>
```

### 模型

![](D:\images\2020-2-4\20200206231550.PNG)

- P：消息的生产者
- 红色：队列
- C：消费者

三个对象 生产者，队列，消费者

### 获取 MQ 连接

```java
    public static Connection getConnection() throws IOException, TimeoutException {
        /*定义连接工厂*/
        ConnectionFactory factory = new ConnectionFactory();
        /*服务地址*/
        factory.setHost("192.168.x.x");
        /*AMPQ 端口*/
        factory.setPort(5672);
        /*vhost*/
        factory.setVirtualHost("/");
        /*用户名*/
        factory.setUsername("guest");
        /*密码*/
        factory.setPassword("guest");
        return factory.newConnection();
    }
```

### 生产者生产消息

```java
    private static final String QUEUE_NAME = "test_simpe_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        /*获取一个连接*/
        Connection connection = ConnectionUtils.getConnection();
        /*从链接中获取一个通道*/
        Channel channel = connection.createChannel();
        /*创建队列声明*/
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        String msg = "HELLO SIMPLE";
        /*发送消息*/
        channel.basicPublish("", QUEUE_NAME, null, msg.getBytes());
        System.out.println("send msg");
        /*关闭通道*/
        channel.close();
        /*关闭连接*/
        connection.close();
    }
```

![](D:\images\2020-2-4\20200206233833.PNG)

![20200206233848](D:\images\2020-2-4\20200206233848.PNG)

### 消费者接受消息

```java
    public static void main(String[] args) throws Exception {
        final String QUEUE_NAME = "test_simpe_queue";
        /*获取一个连接*/
        Connection connection = ConnectionUtils.getConnection();
        /*创建频道*/
        Channel channel = connection.createChannel();

//        /*定义队列的消费者*/
//        QueueingConsumer queueingConsumer = new QueueingConsumer(channel);
//
//        /*监听队列*/
//        channel.basicConsume(QUEUE_NAME, true, queueingConsumer);
//        while (true) {
//            QueueingConsumer.Delivery delivery = queueingConsumer.nextDelivery();
//            String msg = new String(delivery.getBody());
//            System.out.println(msg);
//        }


        /*队列声明*/
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        /*定义消费者*/
        DefaultConsumer defaultConsumer = new DefaultConsumer(channel) {
            /*获取到达消息*/
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String string = new String(body, "utf-8");
                System.out.println(string);
            }
        };

        /*监听队列*/
        channel.basicConsume(QUEUE_NAME, true, defaultConsumer);
    }
```

### 简单队列的不足

耦合性高，生产者——对应消费者(如果我想有多个消费者消费队列中消息，这时候就不行了)，队列名更变，这时候得同时变更。

## Work queues 工作队列之Round-robin轮询分发

### 模型

![](D:\images\2020-2-4\20200207122452.jpg)

为什么会出现工作队列

Simple 队列是——对应的，而且我们实际开发，生产者发送消息是毫不费力的，而消费者一般是要跟业务相结合的消费者接收到消息之后就需要处理，可能需要花费时间，这时候队列就会积压了很多消息。

### 生产者

```java
    public static void main(String[] args) throws Exception {
        final String QUEUE_NAME = "test_work_queue";
        /*获取连接*/
        Connection connection = ConnectionUtils.getConnection();
        /*获取网络通信*/
        Channel channel = connection.createChannel();
        /*声明队列*/
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        for (int i = 0; i < 50; i++) {
            String msg = "send one hello  " + i;
            System.out.println(msg);
            channel.basicPublish("", QUEUE_NAME, null, msg.getBytes());
            Thread.sleep(i * 20);
        }

        channel.close();
        connection.close();
    }
```

### 消费者 - 1

```java
    public static void main(String[] args) throws IOException, TimeoutException {
        final String QUEUE_NAME = "test_work_queue";
        /*获取连接*/
        Connection connection = ConnectionUtils.getConnection();
        /*获取网络通信*/
        Channel channel = connection.createChannel();
        /*声明队列*/
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        /*定义消费者*/
        DefaultConsumer defaultConsumer = new DefaultConsumer(channel) {
            /*消息到达触发方法*/
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "UTF-8");
                System.out.println("receive one : " + msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally{
                    System.out.println("one done");
                }
            }
        };

        channel.basicConsume(QUEUE_NAME, true, defaultConsumer);
    }
```

### 消费者 - 2

```java
    public static void main(String[] args) throws IOException, TimeoutException {
        final String QUEUE_NAME = "test_work_queue";
        /*获取连接*/
        Connection connection = ConnectionUtils.getConnection();
        /*获取网络通信*/
        Channel channel = connection.createChannel();
        /*声明队列*/
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        /*定义消费者*/
        DefaultConsumer defaultConsumer = new DefaultConsumer(channel) {
            /*消息到达触发方法*/
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "UTF-8");
                System.out.println("receive two : " + msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally{
                    System.out.println("tow done");
                }
            }
        };

        channel.basicConsume(QUEUE_NAME, true, defaultConsumer);
    }
```

### 现象

消费者1和消费者2处理的消息是一样的，消费者1是偶数，消费者2是奇输，这种方式叫做轮询分发(round-robin)结果就是不管谁忙谁闲，都不会多给一个消息，任务消息总是你一个我一个。RabbitMQ是采用轮询机制将消息队列Queue中的消息依次发给不同的消费者

## 公平分发 fair dipatch

### 生产者

```java
    public static void main(String[] args) throws Exception {
        final String QUEUE_NAME = "test_work_queue";
        /*获取连接*/
        Connection connection = ConnectionUtils.getConnection();
        /*获取网络通信*/
        Channel channel = connection.createChannel();
        /*声明队列*/
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        /*
         * 每个消费者发送确认消息之间，消息队列不发送下一个消息到消费者，一次只处理一个消息
         * 限制发送给同一个消费者不得查过一条消息
         * */
        int prefetchCount = 1;
        channel.basicQos(prefetchCount);

        for (int i = 0; i < 50; i++) {
            String msg = "send one hello  " + i;
            System.out.println(msg);
            channel.basicPublish("", QUEUE_NAME, null, msg.getBytes());
            Thread.sleep(i * 20);
        }

        channel.close();
        connection.close();
    }
```

### 消费者 - 1

```java
public static void main(String[] args) throws IOException, TimeoutException {
        final String QUEUE_NAME = "test_work_queue";

        /*获取连接*/
        Connection connection = ConnectionUtils.getConnection();
        /*获取网络通信*/
        final Channel channel = connection.createChannel();
        /*声明队列*/
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        /*保证一次只发一个*/
        channel.basicQos(1);

        /*定义消费者*/
        DefaultConsumer defaultConsumer = new DefaultConsumer(channel) {
            /*消息到达触发方法*/
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "UTF-8");
                System.out.println("receive one : " + msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("one done");
                    /*手动会执*/
                    channel.basicAck(envelope.getDeliveryTag(), false);
                }
            }
        };
        /*自动应答false*/
        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, defaultConsumer);
    }
```

### 消费者 - 2

```java
    public static void main(String[] args) throws IOException, TimeoutException {
        final String QUEUE_NAME = "test_work_queue";

        /*获取连接*/
        Connection connection = ConnectionUtils.getConnection();
        /*获取网络通信*/
        final Channel channel = connection.createChannel();
        /*声明队列*/
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        /*保证一次只发一个*/
        channel.basicQos(1);
        /*定义消费者*/
        DefaultConsumer defaultConsumer = new DefaultConsumer(channel) {
            /*消息到达触发方法*/
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "UTF-8");
                System.out.println("receive two : " + msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("tow done");
                    /*手动会执*/
                    channel.basicAck(envelope.getDeliveryTag(), false);
                }
            }
        };
        /*自动应答false*/
        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, defaultConsumer);
    }
```

## 消息应答与消息持久化

### 消息应答

 ```java
boolean autoAck = false;
channel.basicConsume(QUEUE_NAME,autoAck,consumer);
autoAck = true ; //(自动确认模式)一旦rabbitmq将消息分发给消费者，就会从内存中删除。
autoAck = false; //(手动模式)，如果有一个消费者挂掉，就会交付给其他的消费者。rabbitmq支持消息应答，消费者发送一个消息应答，高数rabbitmq这个消息我已经处理完成，你可以删除，rabbitmq就删除内存中的消息。
 ```

- 消息应答默认是打开的，false

### 消息的持久化

```java
boolean durable = false;
/*声明队列*/
channel.queueDeclare(QUEUE_NAME, durable, false, false, null);
```

我们将这个程序中的durable = false;改成true，尽管代码是正确的，他也不会运行成功。因为我们定义了一个叫test_work_queue 这个queue是末持久化，rabbitmq不准许重新定义(不同参数)一个存在的队列，不允许对一个已存在的队列重新定义参数信息。

## 订阅模式 publish / subscribe

### 模型

![](D:\images\2020-2-4\20200207155740.jpg)

1. 一个生产者，多个消费者。
2. 每一个消费者都有自己的队列。
3. 生产者没有直接把消息发送到队列，而是发到了交换机，转发器 exchange。
4. 每个队列都要绑定到交换机上。
5. 生产者发送的消息 经过交换器 到达对了 就能实现一个消息被多个消费者消费。

### 生产者

```java
    public static void main(String[] args) throws Exception {
        final String EXCHANGE_NAME = "text_exchange_fanout";
        Connection connection = ConnectionUtils.getConnection();
        Channel channel = connection.createChannel();
        /*声明交换机*/
        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");//分发
        String msg = "hello ps ";
        channel.basicPublish(EXCHANGE_NAME, "", null, msg.getBytes());

        System.out.println("snd：" + msg);

        channel.close();
        connection.close();
    }
```

![](D:\images\2020-2-4\20200207161846.png)

发送的消息不见了，丢失了，因为交换器没有存储的能力，在rabbitmq里面只有队列有存储能力，因为这时候还没有队列绑定到交换机，所以数据丢失了。

### 消费者 - 1

```java
public static void main(String[] args) throws Exception {
        final String QUEUE_NAME = "text_queue_fanout_eamil";
        final String EXCHANGE_NAME = "text_exchange_fanout";
        Connection connection = ConnectionUtils.getConnection();

        final Channel channel = connection.createChannel();
        /*队列声明*/
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        /*绑定队列到交换机转发器*/
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "");
        /*保证一次分发一次*/
        channel.basicQos(1);
        /*定义消费者*/
        DefaultConsumer defaultConsumer = new DefaultConsumer(channel) {
            /*到达消息触发方法*/
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "UTF-8");
                System.out.println("receive one " + msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("one done");
                    channel.basicAck(envelope.getDeliveryTag(), false);
                }
            }
        };
        /*自动应答*/
        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, defaultConsumer);
    }
```

### 消费者 - 2

```java
 public static void main(String[] args) throws Exception {
        final String QUEUE_NAME = "text_queue_fanout_sms";
        final String EXCHANGE_NAME = "text_exchange_fanout";
        Connection connection = ConnectionUtils.getConnection();

        final Channel channel = connection.createChannel();
        /*队列声明*/
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        /*绑定队列到交换机转发器*/
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "");
        /*保证一次分发一次*/
        channel.basicQos(1);
        /*定义消费者*/
        DefaultConsumer defaultConsumer = new DefaultConsumer(channel) {
            /*到达消息触发方法*/
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "UTF-8");
                System.out.println("receive two " + msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("two done");
                    channel.basicAck(envelope.getDeliveryTag(), false);
                }
            }
        };
        /*自动应答*/
        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, defaultConsumer);
    }
```

![](D:\images\2020-2-4\20200207172107.png)

## Exchange(交换器，转发器)

一方面是接受生产者的消息，另一方面是面向队列推送消息。

匿名转发	""

### Fanout(不处理路由键)

![](D:\images\2020-2-1\202022203557.png)

​		每个发到fanout类型交换器的消息都会分到所有绑定的队列上去。fanout交换器不处理路由键，只是简单的队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout类型转发的消息是最快的。

------



### Direct(处理路由键)

![](D:\images\2020-2-1\202022203151.png)

​		消息中的路由键(routing key)如果和Binding中的binding key一致，交换器就将消息发到对应的队列中。路由键与队列完全匹配，如果一个队列绑定到交换机要求路由键为"dog"，则只转发routing key 标志为 "dog"的消息，它不会转发"dog.puppy"，也不会转发"dog.guard"等等。它是完全匹配，单播的模式。

------

### Topic

将路由键和某模式匹配

	-  “ # ” 匹配一个或者多个
	-  “ * ” 匹配一个





![](D:\images\2020-2-1\202022203946.png)

​		topic交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。它将路由键和绑定键的字符串切分成单词，这些单词之间用点隔开。它同样也会识别两个通配符：符号 " # " 和符号 " * " 。" # "匹配0个或多个单词，" * "匹配一个单词。

------



### 路由模式

#### 模型

![](D:\images\2020-2-4\202027173138.png)

#### 生产者

```java
 public static void main(String[] args) throws IOException, TimeoutException {
        final String EXCHANGE_NAME = "text_exchange_direct";
     	/*获取连接*/
        Connection connection = ConnectionUtils.getConnection();
		/*获得一个通道*/
        Channel channel = connection.createChannel();
		/*创建队列声明*/
        channel.exchangeDeclare(EXCHANGE_NAME, "direct");
        String msg = "hello direct";
        String routingKey = "error";	//指定转发路键的key
		/*发送消息*/
        channel.basicPublish(EXCHANGE_NAME, routingKey, null, msg.getBytes());
        System.out.println("send msg " + msg);

        channel.close();
        connection.close();
    }
```

#### 消费者 - 1

```java
    public static void main(String[] args) throws IOException, TimeoutException {
        final String EXCHANGE_NAME = "text_exchange_direct";
        final String QUEUE_NAME = "text_queue_direct_one";
        /*获取连接*/
        Connection connection = ConnectionUtils.getConnection();
        /*获得一个通道*/
        final Channel channel = connection.createChannel();
        /*创建队列声明*/
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        /*保证一次只分发一个*/
        channel.basicQos(1);
        /*绑定队列到交换机转发器*/
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "error");
        /*定义消费者*/
        DefaultConsumer defaultConsumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "UTF-8");
                System.out.println("receive one " + msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("one done");
                    channel.basicAck(envelope.getDeliveryTag(), false);
                }
            }
        };

        channel.basicConsume(QUEUE_NAME, false, defaultConsumer);
    }
```

#### 消费者 - 2

```java
public static void main(String[] args) throws IOException, TimeoutException {
        final String EXCHANGE_NAME = "text_exchange_direct";
        final String QUEUE_NAME = "text_queue_direct_two";
        /*获取连接*/
        Connection connection = ConnectionUtils.getConnection();
        /*获得一个通道*/
        final Channel channel = connection.createChannel();
        /*创建队列声明*/
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        /*保证一次只分发一个*/
        channel.basicQos(1);
        /*绑定队列到交换机转发器*/
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "error");
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "info");
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "warning");
        /*定义消费者*/
        DefaultConsumer defaultConsumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "UTF-8");
                System.out.println("receive two " + msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("two done");
                    channel.basicAck(envelope.getDeliveryTag(), false);
                }
            }
        };

        channel.basicConsume(QUEUE_NAME, false, defaultConsumer);
    }
```

### Topic

#### 模型

![](D:\images\2020-2-4\202027173201.png)

商品：发布，删除，查询，修改

#### 生产者

```java
    public static void main(String[] args) throws Exception {
        final String EXCHANGE_NAME = "text_exchange_topic";
        Connection connection = ConnectionUtils.getConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare(EXCHANGE_NAME, "topic");
        String msgGood = "商品";
        channel.basicPublish(EXCHANGE_NAME, "good.del", null, msgGood.getBytes());
        System.out.println("send msg " + msgGood);
        channel.close();
        connection.close();
    }
```

#### 消费者 - 1

```java
    public static void main(String[] args) throws Exception {
        final String EXCHANGE_NAME = "text_exchange_topic";
        final String QUEUE_NAME = "text_queue_topic_one";
        /*获取连接*/
        Connection connection = ConnectionUtils.getConnection();
        /*获得一个通道*/
        final Channel channel = connection.createChannel();
        /*创建队列声明*/
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        /*保证一次只分发一个*/
        channel.basicQos(1);
        /*绑定队列到交换机转发器*/
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "good.add");
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "good.del");
        /*定义消费者*/
        DefaultConsumer defaultConsumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "UTF-8");
                System.out.println("receive one " + msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("one done");
                    channel.basicAck(envelope.getDeliveryTag(), false);
                }
            }
        };
        channel.basicConsume(QUEUE_NAME, false, defaultConsumer);
    }
```

#### 消费者 - 2

```java
public static void main(String[] args) throws IOException, TimeoutException {
        final String EXCHANGE_NAME = "text_exchange_topic";
        final String QUEUE_NAME = "text_queue_topic_two";
        /*获取连接*/
        Connection connection = ConnectionUtils.getConnection();
        /*获得一个通道*/
        final Channel channel = connection.createChannel();
        /*创建队列声明*/
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        /*保证一次只分发一个*/
        channel.basicQos(1);
        /*绑定队列到交换机转发器*/
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "good.#");
        /*定义消费者*/
        DefaultConsumer defaultConsumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "UTF-8");
                System.out.println("receive two " + msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("two done");
                    channel.basicAck(envelope.getDeliveryTag(), false);
                }
            }
        };
        channel.basicConsume(QUEUE_NAME, false, defaultConsumer);
    }
```

## RabbitMQ的消息确认机制(事务+confirm)

### 在RabbitMQ中我们可以通过持久化数据解决RabbitMQ服务器异常的数据丢失的问题

问题：生产者将消息发送出去之后，消息到底有没有到达RabbitMQ，服务器默认的情况是不知道的。

两种方式：

- AMQP实现了事务机制
- Confirm模式

### 事务机制

- txSelect txCommit txRollback
- txSelect：用户将当前channel设置transaction模式
- txCommit：用于提交事务
- txRollback：回滚事务

### 生产者

```java
    public static void main(String[] args) throws Exception {
        final String QUEUE_NAME = "test_queue_tx";

        Connection connection = ConnectionUtils.getConnection();

        Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        String msg = "hello tx";

        try {
            channel.txSelect();
            channel.basicPublish("", QUEUE_NAME, null, msg.getBytes());
            int i = 1 / 0;
            System.out.println("confirm msg");
            channel.txCommit();
        } catch (Exception e) {
            e.printStackTrace();
            channel.txRollback();
            System.out.println("confirm rollback");
        }

        channel.close();
        connection.close();
    }
```

### 消费者

```java
    public static void main(String[] args) throws Exception {
        final String QUEUE_NAME = "test_queue_tx";

        Connection connection = ConnectionUtils.getConnection();

        Channel channel = connection.createChannel();

        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        DefaultConsumer defaultConsumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "UTF-8");
                System.out.println(msg);
            }
        };

        channel.basicConsume(QUEUE_NAME, true, defaultConsumer);
    }
```

这种模式很耗时的，采用这种方式，降低了RabbitMQ的消息吞吐量。

### 理解Confirm消息确认机制

- 消息的确认，是指生产者投递消息后，如果Broker收到消息，则会给我们生产者一个应答。
- 生产者进行接收应答，用来确定这条消息是否正常的发送到broker，这种方式也是消息的可靠性投递的核心的保障。

### Confirm 模式

#### 生产者端confirm模式的实现原理

​		生产者将信道设置成confirm模式，一旦信道进入confirm模式，所有在该信道上面发布的消息都会被指派一个唯一的ID(从1开始)，一旦消息被投递到所有匹配队列之后，broker就会发送一个确认的生产者(包含消息的唯一ID)，这就使得生产者知道消息以及正确到达目的队列了，如果消息和队列是可持久化的，那么确认消息会将消息写入磁盘之后发出，broker回传给生产者的确认消息中deliver-tag域包含了确认消息的序列号，此外broker也可以设置basic.ack的multiple域，表示到这个序列号之前的所有消息都已经得到了处理。

Confirm模式最大的好处在于他是异步：
	Nack
开启confirm模式
	channel.confirmSelect()
编程模式：

	- 发一条数据，waitForConfirms()
	- 批量发数据，waitForConfirms()
	- 异步 confirm模式，提供一个回调的方法

#### Confirm单条消息，生产者

```java
    public static void main(String[] args) throws Exception {
        final String QUEUE_NAME = "test_queue_confirm";
        Connection connection = ConnectionUtils.getConnection();
        Channel channel = connection.createChannel();
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        /*生产者调用confirmSelect，将channel设置为confirm模式*/
        channel.confirmSelect();
        String msg = "hello confirm";
        channel.basicPublish("", QUEUE_NAME, null, msg.getBytes());
        if (!channel.waitForConfirms()) {
            System.out.println("send msg failed");
        }else{
            System.out.println("send msg ok");
        }
        channel.close();
        connection.close();
    }
```

#### Confirm 批量消息，生产者

```java
    public static void main(String[] args) throws Exception {
        final String QUEUE_NAME = "test_queue_confirm";
        Connection connection = ConnectionUtils.getConnection();

        Channel channel = connection.createChannel();
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        /*生产者调用confirmSelect，将channel设置为confirm模式*/
        channel.confirmSelect();

        String msg = "hello confirm batch";
        for (int i = 0; i < 10; i++) {
            channel.basicPublish("", QUEUE_NAME, null, msg.getBytes());
        }

        if (!channel.waitForConfirms()) {
            System.out.println("send msg failed");
        } else {
            System.out.println("send msg ok");
        }
        channel.close();
        connection.close();
    }
```

#### 消费者

```java
    public static void main(String[] args) throws Exception {
        final String QUEUE_NAME = "test_queue_confirm";
        Connection connection = ConnectionUtils.getConnection();

        Channel channel = connection.createChannel();
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        channel.basicConsume(QUEUE_NAME, true, new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "UTF-8");
                System.out.println(msg);
            }
        });
    }
```

#### 异步模式

Channel对象提供的 ConfirmListener()回调方法质保函deliveryTag(当前Channel发出的消息序号)，我们需要自己为每一个Channel维护一个unconfirm的消息序号集合，每publish一条数据，集合中元素加1，每回调一次handleAck方法，unconfirm集合删掉相应的一条(multiple=false)或者多条(multiple=true)记录，从程序运行效率上看，这个unconfirm集合最好采用有序集合SortedSet存储结构。

## Spring 集成 RabbitMQ-client

### pom.xml

```xml
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
</dependency>
```

### applicationContext.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/rabbit http://www.springframework.org/schema/rabbit/spring-rabbit.xsd">
    <!--1.定义RabbitMQ的连接工厂-->
    <rabbit:connection-factory id="connectionFactory"
                               host="192.168.1.106" port="5672" username="guest" password="guest"/>
    <!--定义Rabbit模板，指定连接工厂以及定义exchange(交换器)-->
    <rabbit:template id="rabbitTemplate" connection-factory="connectionFactory" exchange="fanoutExchange"/>
    <!--MQ的管理，包括队列，交换器声明-->
    <rabbit:admin connection-factory="connectionFactory"/>
    <!--定义队列，自定声明-->
    <!--durable，如果当前有这个name的名称，durable设置为false 运行修改，如果为true不允许修改，否则会报错-->
    <rabbit:queue name="queue" auto-declare="true" durable="true"/>
    <rabbit:queue name="test_queue_confirm" auto-declare="true" durable="false"/>
    <rabbit:queue name="test_queue_confirm_three" auto-declare="true" durable="false"/>
    <!--定义交换器，自动声明-->
    <rabbit:fanout-exchange name="fanoutExchange" auto-declare="true">
        <rabbit:bindings>
            <rabbit:binding queue="queue"/>
            <rabbit:binding queue="test_queue_one"/>
            <rabbit:binding queue="test_queue_two"/>
        </rabbit:bindings>
    </rabbit:fanout-exchange>
    <!--队列监听-->
    <rabbit:listener-container connection-factory="connectionFactory">
        <rabbit:listener ref="foo" method="listenerConsumer" queue-names="queue"/>
    </rabbit:listener-container>
    <!--消费者-->
    <bean id="foo" class="com.baerwang.rabbitmq.spring.MyConsumer"/>
</beans>
```

### 监听消费者

```java
public class MyConsumer {
    public void listenerConsumer(String foo) {
        System.out.println("消费者：" + foo);
    }
}
```

### 启动

```java
    public static void main(String[] args) throws Exception {
        AbstractApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
        RabbitTemplate template = ctx.getBean(RabbitTemplate.class);
        template.convertAndSend("hello,rabbitmq");
        /*容器销毁*/
        ctx.destroy();
    }
```

## 幂等性

### 幂等性是什么

- 我们可以借鉴数据库乐欢锁机制：
- 比如我们执行一条更新库存的SQL语句
- UPDATE table SET COUNT = COUNT - 1,VERSION = VERSION +1 WHERE VERSION =1
- 幂等性强调的是同一个操作多次重复执行

### 消费端-幂等性保障

在海量订单产生的业务高峰期，如何避免消息的重复消费问题？

- 消费端实现幂等性，就意味着，我们的消息永远不会消费多次，即使我们收到了多条一样的消息。

业务主流的幂等性操作：

- 唯一的ID + 指纹码机制
  - 唯一的ID + 指纹码机制，利用数据库主键去重
  - SELECT COUNT(1) FROM table WHERE ID = 唯一ID + 指纹码
  - 好处：实现简单
  - 坏处：高并发下有数据库写入的性能瓶颈
  - 解决方案：跟进ID进行分库分表进行算法路由
- 利用Redis的原子性去实现。
  - 使用Redis进行幂等，需要考虑的问题
  - 第一：我们是否要进行数据落库，如果落库的话，关键解决的问题是数据库和缓存如何做到原子性。
  - 如果不进行落库，那么都存储到缓存中，如何设置定时同步的策略。
  - 给消息分配一个全局id，只要消费过该消息，将<id,message>以形式键入Redis，消息者开始消费前，先去redis中查询有没有消费记录。

### 消息的幂等性的必要性

- 保障消息的幂等性，这也是我们在使用MQ中至关重要的环节
- 可能导致消息出现非幂等的原因：
  1. 可靠性消息投递机制
  2. MQ Broker服务与消费端传输消息的过程中网络抖动
  3. 消费端故障或异常

### 消息的幂等性的设计

![](D:\images\2020-2-4\202027213529.PNG)