# Zookeeper+Replicated-LevelDB-Store主从集群搭建

## 集群搭建

1. 创建文件夹(myZK)(三台机器，就是三个虚拟机)

   ​		mkdir myZK

2.  [下载ZK](http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz)

   ​	进入目录 cd myZK

   ​		把下载的zk压缩包解压到本目录

   ​				tar -zxvf zookeeper-3.4.14.tar.gz

   ​		解压好的文件进入conf目录 cd zookeeper-3.4.14/conf

   ​		目录下有个zoo_sample.cfg，

   ​		修改配置文件 名字叫 zoo.cfg 

   ​				cp zoo_sample.cfg zoo.cfg
   ​		修改zoo.cfg的文件

   ​				vim zoo.cfg

   ​		设置一下自定义的数据文件夹(注意要手动创建这个目录,后面会用到),,注意最后一定要有“/”结尾,否则会报错
   ​				dataDir=/myZK/zookeeper-3.4.14/zkData/

   ​		在zoo.cfg件最后面追加集群服务器
   ​				server.1=192.168.x.x:2888:3888
   ​				server.2=192.168.x.x:2888:3888
   ​				server.3=192.168.x.x:2888:3888

   ​	server.1=leantaot-zk-01:2888:3888
   ​	1是一个数字，标识这个是第几号服务器
   ​	leantaot-zk-01是这个服务器的IP地址（或者是与IP地址做了映射的主机名）
   ​	2888第一个端口用来集群成员的信息交换，标识这个服务器与集群中的leader服务器交换信息的端口
   ​	3888是在leader挂掉时专门用来进行选举leader所用的端口
   ​			其他集群也要做以上的步骤

3. 创建myid文件， id 与 zoo.cfg 中的序号对应

   ​	其中1就是序号

   ```liunx
   echo 1 > /opt/module/zookeeper-3.4.14/data/zookeeper_server.pid /data/myid
   echo 1 > /opt/module/zookeeper-3.4.14/data/myid
   ```

   

4. 在myZK的bin目录下测试运行zk

   ​	运行

   ​		./zkServer.sh start

   ​	输出(显示运行成功)

   ​		ZooKeeper JMX enabled by default
   ​		Using config: /opt/module/zookeeper-3.4.14/bin/../conf/zoo.cfg
   ​		Starting zookeeper ... STARTED

   ​	关闭 

   ​		./zkServer.sh stop

   ​		输出(显示关闭成功)

   ​		ZooKeeper JMX enabled by default
   ​		Using config: /opt/module/zookeeper-3.4.14/bin/../conf/zoo.cfg
   ​		Stopping zookeeper ... STOPPED

5. 创建文件夹(zkMQ)(一个机器，就是一个虚拟机)

   ​	mkdir zkMQ

6. [下载ActiveMQ](http://archive.apache.org/dist/activemq/5.15.9/apache-activemq-5.15.9-bin.tar.gz)

   ​	进入目录 cd zkMQ

   ​		把下载的zk压缩包解压到本目录

   ​				tar -zxvf zookeeper-3.4.14.tar.gz

    	然后把压缩的文件 复制三份

   ​			cp -r apache-activemq-5.15.11 mq_node01

   ​	![](D:\images\2020-1-27\捕获.PNG)

7. 修改管理控制台端口(ActiveMQ后台管理访问的端口)

   ​	mq_node01 不修改(默认端口为8161)

   ​	mq_node-02 (端口改成8162)

   ​			![](D:\images\2020-1-27\捕获3.PNG)

   ​			cd mq_node-02 

   ​			cd conf

   ​			vim jetty.xml

   ​			![](D:\images\2020-1-27\捕获2.PNG)

   ​	mq_node-03 (端口改成8163)

   ​			步骤mq02一样

8. 修改ActiveMQ的机器的hostname

   ​		vim /etc/hosts

   ​			192.168.x.x mq-server

   ​		ip地址要对应

   ​		如果不知道ip是什么，可以输入ipconfig 查看en33 ip地址

9. ActiveMQ集群配置

   ​	三个节点的BrokerName要求全部一致

   ​	必须配持久化配置

   ​		cd mq_node01
   ​		vim activemq.xml

   ​	要修改的配置(brokerName就是刚才修改的机器hostname 的名字映射)

   ![](D:\images\2020-1-27\捕获4.PNG)

   ​	添加持久化配置

   ```java
   <persistenceAdapter>
      <replicatedLevelDB
           directory="${activemq.data}/leveldb"
           replicas="3"
           bind="tcp://0.0.0.0:62621"
           zkAddress="192.168.x.x:2181,192.168.x.x:2181,192.168.x.x:2181"
           hostname="mq-server"
           zkPath="/activemq/leveldb-stores" />
   </persistenceAdapter>
   ```

   

   ​	图中有sync，不添加就行了，不用管它

   

   ![](D:\images\2020-1-27\20200126140221.png)

   ​		修改各个节点的端口

   ​	![](D:\images\2020-1-27\20200126141713.jpg)

10. 在mq_cluster各个activemq路的bin目录下运行

    ​	(运行)./activemq start

    ![](D:\images\2020-1-27\捕获5.PNG)

    ​	(关闭)./activemq stop

11. 先启动zk后启动mq

    ​	查看zk集群状态，任意一台zk机器连接客户端验证mq是否注册上了zk

    ​		使用zkCli.sh连接一台Zookeeper

    ​			./zkCli.sh -server 192.168.x.x:2181(进入zk的客户端)

    ​		![](D:\images\2020-1-27\20200126170614.jpg)

    ​		如果显示了这些，说明mq已经注册了zk

12. 查看master

    ​	第一个elected不为空说明这个节点为master,为空就是slave![](D:\images\2020-1-27\20200126171446.jpg)

13. 集群可用性测试

    ![](D:\images\2020-1-27\fff.png)

    ​			ActiveMQ的客户端只能访问Master的Broker，其他除Slave的Broker不能访问，所以客户端连接的Broker应该用failover协议(失败转移)。

    ​			当一个ActiveMQ节点挂掉或者一个ZK节点挂点，ActiveMQ服务依然正常运转，如果仅剩一个ActiveMQ节点，由于不能选举Master,所以ActiveMQ不能正常运行。

    ​			同样如果Zookeeper仅剩一个活动节点，不管ActiveMQ各节点存货，ActiveMQ也不能正常提供服务。

    ​			(ActiveMQ集群的高可用依赖于ZK集群的高可用)。

14. 写一个Produce和ConSumer类

    ​	生产者

    ```java
    public class Produce {
    
        private static final String ACTIVEMQ_URL = "failover:(tcp://192.168.x.x:61616,tcp://192.168.x.x:61617,tcp://192.168.x.x:61618)?randomize=false";
        private static final String CLUSTER_NAME = "queue-cluster";
    
        public static void main(String[] args) throws JMSException {
            /*1.创建连接工厂,按照给定的URL,采用默认的用户名和密码*/
            ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
            /*2.通过连接工厂，获得connection并启动访问*/
            Connection connection = activeMQConnectionFactory.createConnection();
            connection.start();
            /*3.创建会话session*/
            /*两个参数transacted=事务，acknowledgeMode=确认模式(签收)*/
            Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
            /*3.创建目的地(具体是队列queue还是主题topic)*/
            Queue queue = session.createQueue(CLUSTER_NAME);
            /*5.创建消息的生产者*/
            MessageProducer messageProducer = session.createProducer(queue);
            /*持久化，设置为NON_PERSISTENCE保存到内存,设置为PERSISTENT保存到broker相应的文件或者数据库*/
            messageProducer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
            /*6.通过使用消息生产者，生成三条消息，发送到MQ的队列里面*/
            for (int i = 0; i <= 3; i++) {
                /*7.创建消息*/
                /*理解为字符串*/
                TextMessage textMessage = session.createTextMessage("cluster:" + i);
    
                /*8.通过messageProduct发送给MQ队列*/
                messageProducer.send(textMessage);
            }
            /*9.关闭资源*/
            messageProducer.close();
            session.close();
            connection.close();
            System.out.println("消息发布到MQ队列完成");
        }
    }
    ```

    ​	消费者

    ```java
    public class Consumer {
        private static final String ACTIVEMQ_URL = "failover:(tcp://192.168.x.x:61616,tcp://192.168.x.x:61617,tcp://192.168.x.x:61618)?randomize=false";
        private static final String CLUSTER_NAME = "queue-cluster";
    
        public static void main(String[] args) throws JMSException, IOException {
            /*1.创建连接工厂，按照给定的URL,采用默认的用户名和密码*/
            ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
            /*2.通过连接工厂，获得connection并启动方位*/
            Connection connection = activeMQConnectionFactory.createConnection();
            connection.start();
            /*3.创建会话session*/
            /*两个参数transacted事务,acknowledgeMode=确认模式(签收)*/
            Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
            /*4.创建目的地(具体是队列queue还是主题topic)*/
            Queue queue = session.createQueue(CLUSTER_NAME);
            /*5.创建消息的消费者，指定消费哪一个队列里面的消息*/
            MessageConsumer messageConsumer = session.createConsumer(queue);
            /*同步阻塞方式(receive())
              订阅者或接受者调用MessageConsumer的receive()方法来接收消息，
              receive方法能够接收到消息之间(或超之间)将一直阻塞。
            */
            //循环获取
            while (true) {
                //*6.通过消费者调用方法获取队列里面的消息(发送的消息是什么类型，接收的时候就强转成什么类型*//*
                TextMessage textMessage = (TextMessage) messageConsumer.receive(4000L);
                if (textMessage != null) {
                    System.out.println("消费者接收到的消息：" + textMessage.getText());
                } else {
                    break;
                }
            }
            /*7关闭资源*/
            messageConsumer.close();
            session.close();
            connection.close();
        }
    }
    ```

    

    ​	生产者发送消息

    ​		![](D:\images\2020-1-27\20200126174610.jpg)

    ​		![](D:\images\2020-1-27\202001261746378.jpg)

    ​	消费者消费消息

    ​		![](D:\images\2020-1-27\20200126174731.jpg)

    ​		![](D:\images\2020-1-27\20200126174811.jpg)

15. 测试三台机器中的ActiveMQ只会有一个MQ可以被客户端连接使用，在测试的时候可以把Master关掉，然后在重试客户端消息发送和消费可以正常使用，集群搭建正常。

    ​		现在是Master/8161，可以正常访问，说明这台是Master

    ​		lsof -i:8161 如果显示了就是Master不显示就不是Master

    ​		lsof -i:8162 

    ​		lsof -i 8163

    ​		现在down Master 来测试一下

    ​				kill -9 pid (pid就是刚才lsof -i:端口显示的pid) 这个命令目的是给进程杀掉

    ​		lsof -i:8161 不显示了 Master down掉了

    ​		lsof -i:8162 显示了，代替down掉的Master

    ​		运行Produce

    ​		![](D:\images\2020-1-27\20200126192019.jpg)

    ​		运行Consumer

    ​		![](D:\images\2020-1-27\20200126192119.jpg)

    ​		发现了Master挂掉后，slave来代替他

## 异步投递Async Sends

​	异步投递：对于一个Slow Consumer，使用同步发送消息可能出现Producer堵塞的情况，慢消费者合适使用异步发送

​	异步投递是什么：

​		ActiveMQ支持同步，异步两种发送的模式将消息发送到broker，模式的选择对发送延时有巨大的邮箱。Producer能达到怎么样的产出率(产出率=发送数据总量/时间)主要受发送延时的邮箱，试音异步发送可以显著提高发送的性能。

​		ActiveMQ默认使用异步发送的模式：除非明确指定使用同步发送的方式或者在未使用事务的提前下发送持久化的消息，这两种情况都是同步发送的。

​		如果你没有使用事务且发送的持久化的消息，每一次发送都是同步发送的且会阻塞Producer知道broker返回一个确认，表示消息以及被安全的持久化到磁盘重。确认机制提供了消息安全的保障，但同时会阻塞客户端带来了很大的延时。

​		很多高性能的应用，允许在失败的情况下有少量的数据丢失。如果你的应用满足这个特点，你可以使用异步发送来提高生产率，即使发送的持久化的消息。

​	异步发送

​		它可以最大化的Producer端的发送效率。我们通常在发送消息量比较密集的情况下使用一步发送，他可以很大的提高Producer性能，不过这也带来了额外的问题，就是需要消耗更多的Client端内存同时也会导致broker端性能消耗增加。此外它不能有效地确保消息的发送成功。在userAsyncSend=true的情况下客户端需要容忍消息丢失的可能。

​	异步消息如何启动发送成功？

​		异步发送丢失消息的场景是：生产者设置UseAsyncSend=true，使producer.send(msg)持续发送消息。由于消息补阻塞，生产者会认为书有send的消息均被成功发送至MQ。如果MQ突然down机，生产者会认为所有有send的消息均被成功发送MQ。

​		如果MQ突然down机，此时生产者端中尚未被发送至MQ的消息都会丢失。

​		所以，正确的异步发送的方法是需要接受回调的。

​		同步发送和异步发送的区别。同步发送等send不阻塞了就表示一定发送成功了，异步发送需要接受回执并有客户端在判断一次是否发送成功。

```java
public class Produce_AsyncSend {

    private static final String ACTIVEMQ_URL = "tcp://192.168.x.x:61616";
    private static final String queue02 = "queue02";

    public static void main(String[] args) throws JMSException {
        /*1.创建连接工厂,按照给定的URL,采用默认的用户名和密码*/
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        /*开启异步投递*/
        activeMQConnectionFactory.setUseAsyncSend(true);
        /*2.通过连接工厂，获得connection并启动访问*/
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        /*3.创建会话session*/
        /*两个参数transacted=事务，acknowledgeMode=确认模式(签收)*/
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        /*3.创建目的地(具体是队列queue还是主题topic)*/
        Queue queue = session.createQueue(queue02);
        ActiveMQMessageProducer activeMQMessageProducer = (ActiveMQMessageProducer) session.createProducer(queue);
        for (int i = 0; i < 3; i++) {
            TextMessage textMessage = session.createTextMessage("message - " + i);
            textMessage.setJMSMessageID(UUID.randomUUID().toString());
            String jmsMessageID = textMessage.getJMSMessageID();
            activeMQMessageProducer.send(textMessage, new AsyncCallback() {
                @Override
                public void onSuccess() {
                    System.out.println(jmsMessageID + "发送成功");
                }

                @Override
                public void onException(JMSException exception) {
                    System.out.println(jmsMessageID + "发送失败");
                }
            });
        }
        /*9.关闭资源*/
        activeMQMessageProducer.close();
        session.close();
        connection.close();
    }
}
```



![](D:\images\2020-1-27\20200126233105.jpg)

## 延时投递和定时投递

四大属性：

![](D:\images\2020-1-27\delay.bmp)

​		在activemq.xml中配置schedulerSupport属性为true

​		Java代码里面封装的辅助消息类型：ScheduledMessage

​	生产者

```java
public class Produce_DelayAndSchedule {

    private static final String ACTIVEMQ_URL = "tcp://192.168.1.112:61616";
    private static final String ACTIVEMQ_QUEUE_NAME = "queue-delay";

    public static void main(String[] args) throws JMSException {
        /*1.创建连接工厂,按照给定的URL,采用默认的用户名和密码*/
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory();
        activeMQConnectionFactory.setBrokerURL(ACTIVEMQ_URL);
        /*2.通过连接工厂，获得connection并启动访问*/
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        /*3.创建会话session*/
        /*两个参数transacted=事务，acknowledgeMode=确认模式(签收)*/
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        /*3.创建目的地(具体是队列queue还是主题topic)*/
        Queue queue = session.createQueue(ACTIVEMQ_QUEUE_NAME);
        /*向上转型到ActiveMQMessageProducer*/
        MessageProducer messageProducer = session.createProducer(queue);
        /*延迟投递的时间*/
        long delay = 3 * 1000;
        /*每次投递的事件间隔*/
        long period = 4 * 1000;
        /*投递的次数*/
        int repeat = 5;
        for (int i = 0; i < 3; i++) {
            TextMessage textMessage = session.createTextMessage("message - " + i);
             System.out.println(textMessage.getText());
            /*给消息设置属性以便MQ服务器读取到这些信息，好做对应的处理*/
            textMessage.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_DELAY, delay);
            textMessage.setLongProperty(ScheduledMessage.AMQ_SCHEDULED_PERIOD, period);
            textMessage.setIntProperty(ScheduledMessage.AMQ_SCHEDULED_REPEAT,repeat);
            messageProducer.send(textMessage);
        }
        /*9.关闭资源*/
        messageProducer.close();
        session.close();
        connection.close();
    }
}
```

​	消费者

```java
public class Consumer_DelayAndSchedule {

    private static final String ACTIVEMQ_URL = "tcp://192.168.1.112:61616";
    private static final String ACTIVEMQ_QUEUE_NAME = "queue-delay";

    public static void main(String[] args) throws JMSException, IOException {
        /*1.创建连接工厂，按照给定的URL,采用默认的用户名和密码*/
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        /*2.通过连接工厂，获得connection并启动方位*/
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        /*3.创建会话session*/
        /*两个参数transacted事务,acknowledgeMode=确认模式(签收)*/
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        /*4.创建目的地(具体是队列queue还是主题topic)*/
        Queue queue = session.createQueue(ACTIVEMQ_QUEUE_NAME);
        /*5.创建消息的消费者，指定消费哪一个队列里面的消息*/
        MessageConsumer messageConsumer = session.createConsumer(queue);
        /*同步阻塞方式(receive())
          订阅者或接受者调用MessageConsumer的receive()方法来接收消息，
          receive方法能够接收到消息之间(或超之间)将一直阻塞。
        */
        //循环获取
        while (true) {
            //*6.通过消费者调用方法获取队列里面的消息(发送的消息是什么类型，接收的时候就强转成什么类型*//*
            TextMessage textMessage = (TextMessage) messageConsumer.receive(4000L);
            if (textMessage != null) {
                System.out.println("消费者接收到的消息：" + textMessage.getText());
            } else {
                break;
            }
        }
        /*7关闭资源*/
        messageConsumer.close();
        session.close();
        connection.close();
    }
}
```

## 消息重试机制

面试题：

​		具体哪些情况回引发消息重发

1. Client用了transactions且再session重调用了rollback

2. Client用了transactions且再调用commit之前关闭或者没有commit

3. Client再CLIENT_ACKNOWLEDGE的传递模式下，session重调用了recover()

​		消息重发时间间隔和重发次数

1.    间隔：1，次数：6，每秒发：6次


​		有毒消息Posion ACK

1.  一个消息被redelivedred超过默认的最大重发次数(默认6次)时，消费的回个MQ发一个“poison ack"表示这个消息有毒，告诉broker不要再发了，这时候broker回把这个消息放到DLQ(私信队列)。
2. 缺省的私信队列是ACTIVEMQ.DLQ，没有特别指定，私信队徽被发送到这个队列。
3. 缺省持久消息过期，会被送到DLQ，非持久消息不会送到DLQ。
4. 可以通过配置文件(activemq.xml)来吊证死信发送策略。


RedeliveryPolicy属性：

![](D:\images\2020-1-27\property.bmp)

Produce

```java
public class Produce {
    private static final String ACTIVEMQ_URL = "tcp://192.168.x.x:61616";
    private static final String ACTIVEMQ_QUEUE_NAME = "queue";

    public static void main(String[] args) throws JMSException {
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
        Queue queue = session.createQueue(ACTIVEMQ_QUEUE_NAME);
        MessageProducer messageProducer = session.createProducer(queue);
        for (int i = 0; i < 3; i++) {
            TextMessage textMessage = session.createTextMessage("msg : " + i);
            messageProducer.send(textMessage);
        }
        messageProducer.close();
        session.close();
        connection.close();
        System.out.println("发送MQ完成");
    }
}
```

Consumer

```java
public class Consumer {
    private static final String ACTIVEMQ_URL = "tcp://192.168.x.x:61616";
    private static final String ACTIVEMQ_QUEUE_NAME = "queue";

    public static void main(String[] args) throws JMSException {
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);

        RedeliveryPolicy redeliveryPolicy = new RedeliveryPolicy();
        /*是否在每次尝试中心发送失败后，增长这个等待时间*/
        redeliveryPolicy.setUseExponentialBackOff(true);
        /*重发次数，默认6次*/
        redeliveryPolicy.setMaximumRedeliveries(3);
        /*重发时间间隔，默认1秒*/
        redeliveryPolicy.setInitialRedeliveryDelay(1000);
        /*第一次失败后重新发送之前等待500毫秒，第二次在等待500*2毫秒 2指的是value*/
        redeliveryPolicy.setBackOffMultiplier(2);
        /*最大传送延迟
         *假设首次重连间隔为10ms倍数为2，那么二次重连时间间隔为20ms,三次重连时间间为40ms
         * 当重连时间间隔大的最大重连时间间隔，以后每次重连时间间隔都为最大重连时间间隔
         * */
        redeliveryPolicy.setMaximumRedeliveryDelay(1000);
        activeMQConnectionFactory.setRedeliveryPolicy(redeliveryPolicy);

        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        Session session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);
        Queue queue = session.createQueue(ACTIVEMQ_QUEUE_NAME);
        MessageConsumer messageConsumer = session.createConsumer(queue);
        TextMessage textMessage = null;
        while (true) {
            textMessage = (TextMessage) messageConsumer.receive(1000L);
            if (null != textMessage) {
                System.out.println(textMessage.getText());
            } else {
                break;
            }
        }
        //session.commit();
        session.close();
        connection.close();
    }
```



![](D:\images\2020-1-27\2020127191120.bmp)

## 死信队列

ActiveMQ中引入了"死信队列"(Dead Letter Queue)的概念。即一条消息再被重发了多次后(默认为重发6此redeliveryCounter==6)，将会被ActiveMQ引入了"死神队列"。开发人员可以这个个Queue中查看处理错误的消息，进行人工干预。
![](D:\images\2020-1-27\20200127192114.jpg)

死信队里的使用：处理失败的消息

1. 一般生产环境中使用MQ的世界设计两个队列：一个是核心业务队列，一个是死神队列。

2. 核心业务队列，就是专门用来让订单系统发送订单消息的，然后另外一个死神队里就是用来处理异常的情况。
3. 加入第三方物流系统故障此时无法请求，那么仓储系统每次消费到一条订单消息，尝试通知发货和配送都会遇到对方的接口报错。此时仓储系统就可以把这条消息拒绝访问或者标志位处理失败。一旦标志这条消息处理失败之后，MQ就会把这条消息转入提前设置好的一个死信队列中。然后会看到在第三方物流系统故障期间，所有订单消息全部处理失败，全部会转入死信队列。然后你的仓储系统得专门有一个后台线程，监控第三方物流系统是否正常，能否请求的，不停的监视。一旦发现对方恢复正常，这个后台线程就从死信队列消费出来处理失败的订单，重新执行发货和配送的通知逻辑。

死信队列的配置介绍：

​	SharedDeadLetterStrategy

​	将所有的DeadLetter保存在一个共享的队列中，这是ActiveMQ broker端默认的策略。

​	共享队列默认为"ActiveMQ.DLQ"，可以通过"deadLetterQueue"属性来设定。

```xml
<deadLetterStrategy>
    <sharedDeadLetterStrategy deadLetterQueue="DLQ-QUEUE"/>
</deadLetterStrategy>
```

​	IndividualDeadLetterStrategy

​	把DeadLetter放入各自的死信通道中：

1. 对于Queue而言，死信通道的前缀默认为"ActiveMQ.DLQ.Queue"。
2. 对于Topic而言，死信通道的前缀默认为"ActiveMQ.DLQ.Topic"。

 比如队列Order，那么它对应的死信通道为"ActiveMQ.DLQ.Queue.Order"。

我们使用"queuePrefix"-"topicPrefix"来指定上述前缀。

默认情况下，无论是Topic还是Queue，broker将使用Queue来保存DeadLeader，即死信通道通畅为Queue；不过开发者也可以指定为Topic。

```xml
<policyEntry queue="order">
    <deadLetterStrategy>
    	<individualDeadLetterStrategy queuePrefix="DLQ" useQueueForQueueMessages="false" />
    </deadLetterStrategy>
</policyEntry>
```



将队列Order出现的DeadLetter保存在DLQ.Order中，不过此时DLQ.Order为Topic。

配置案例：

​	自动删除过期消息。

1. 有事需要直接删除过期的消息而不需要发送到死信队列中，"processExpired"表示是否将过期消息放入死信队列，默认为true。

```xml
<policyEntry queue=">">
    <deadLetterStrategy>
    	<sharedDeadLetterStrategy processExpired="false"/>
    </deadLetterStrategy>
</policyEntry>
```



​	存放非持久消息到死信队列中。

1. ​	默认情况下，ActiveMQ不会把非持久的死消息发送到死信队列中。
2. ​	"processNonPersistent"表示是否将"非持久化"消息放入死信队列，默认为false。
3. ​	非持久性如果想把非持久的消息发送到死信队列中，需要设置属性processNonPersistent="true"

```xml
<policyEntry queue=">">
    <deadLetterStrategy>
    	<sharedDeadLetterStrategy processExpired="true"/>
    </deadLetterStrategy>
</policyEntry>
```



activemq.xml

```xml
<policyEntry queue=">">
    <deadLetterStrategy>
    	<individualDeadLetterStrategy queuePrefix="DLQ" useQueueForQueueMessages="true" />
    </deadLetterStrategy>
</policyEntry>
```

如何保证消息不被重复消费

1. 网络延迟传输中，会造成进行MQ重试中，在重试过程中，会造成重复消费。
2. 如果消息是做数据库插入操作，给这个消息做一个唯一主键，那么就算出现重复的情况下，就会导致主键冲突，避免数据库出现脏数据。
3. 上面情况还不行，准备第三方服务来做消费记录。以Redis为例，给消息分配一个全局Id，只要消费过该消息，将<id，message>以形式键入Redis，那么消费者开始消费前，先去Redis中查询有没有消费记录即可。