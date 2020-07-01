# 								                                                  ActiveMQ

## 一、 ActiveMQ基础编码

### 1. ActiveMQ_QUEUE生产者编码

------

```java
public class JmsProduce {

    /**
     * mq链接的主机
     */
    private static final String ACTIVEMQ_URL = "tcp://47.105.63.60:61616";
    /**
     * 队列名称
     */
    private static final String QUEUE_NAME = "QUEUE_01";

    public static void main(String[] args) throws JMSException {

        // 1.创建链接工厂
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);

        // 2.获取connection链接,并启动访问
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();

        // 3.创建session会话
        // 两个参数 事务  签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

        // 4.创建目的地(具体是Queue 还是 topic)
        Queue queue = session.createQueue(QUEUE_NAME);
        
        // 5.创建消息的生产者
        MessageProducer messageProducer = session.createProducer(queue);
        for (int i = 0; i < 3; i++) {
            // 6.创建消息
            TextMessage textMessage = session.createTextMessage("队列" + QUEUE_NAME + "第" + i + "条消息");
            // 7.发送消息到MQ
            messageProducer.send(textMessage);
        }
        // 8.关闭资源
        messageProducer.close();
        session.close();
        connection.close();
        System.out.println("消息发布完成");
    }
}
```



### 2. ActiveMQ_QUEUE消费者编码

------

```java
public class JmsConsumer {

    /**
     * mq链接的主机
     */
    private static final String ACTIVEMQ_URL = "tcp://47.105.63.60:61616";
    /**
     * 队列名称
     */
    private static final String QUEUE_NAME = "QUEUE_01";

    public static void main(String[] args) throws JMSException, IOException {
        System.out.println("我是1号消费者");
        // 1.创建工厂
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);

        // 2.创建连接
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();

        // 3.创建session
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

        // 4.创建目的地
        Queue queue = session.createQueue(QUEUE_NAME);

        // 5.创建消费者
        MessageConsumer messageConsumer = session.createConsumer(queue);

        // 6.通过监听的方式来消费消息
        messageConsumer.setMessageListener(new MessageListener() {
            public void onMessage(Message message) {
                if (null != message && message instanceof TextMessage) {
                    TextMessage textMessage = (TextMessage) message;
                    try {
                        System.out.println("消费者接受到消息" + textMessage.getText());
                    } catch (JMSException e) {
                        e.printStackTrace();
                    }
                }
            }
        });
        // 7.关闭资源
        System.in.read();
        messageConsumer.close();
        session.close();
        connection.close();
    }
```



### 3. ActiveMQ_TOPIC生产者编码

------

```java
public class JmsTopicProduce {

    /**
     * mq链接的主机
     */
    private static final String ACTIVEMQ_URL = "tcp://192.168.138.132:61616";
    /**
     * 队列名称
     */
    private static final String TOPIC_NAME = "TOPIC_01";

    public static void main(String[] args) throws JMSException {

        // 1.创建链接工厂
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);

        // 2.获取connection链接,并启动访问
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();

        // 3.创建session会话
        // 两个参数 事务  签收
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

        // 4.创建目的地(具体是Queue 还是 topic)
        Topic topic = session.createTopic(TOPIC_NAME);
        // 5.创建消息的生产者
        MessageProducer messageProducer = session.createProducer(topic);
        for (int i = 0; i < 6; i++) {
            // 6.创建消息
            TextMessage textMessage = session.createTextMessage("队列" + TOPIC_NAME + "第" + i + "条消息");
            // 7.发送消息到MQ
            messageProducer.send(textMessage);
        }

        // 8.关闭资源
        messageProducer.close();
        session.close();
        connection.close();
        System.out.println("消息发布完成");
    }
```



### 4. ActiveMQ_TOPIC消费者编码

------

```java
public class JmsTopicConsumer {

    /**
     * mq链接的主机
     */
    private static final String ACTIVEMQ_URL = "tcp://192.168.138.132:61616";
    /**
     * 队列名称
     */
    private static final String TOPIC_NAME = "TOPIC_01";

    public static void main(String[] args) throws JMSException, IOException {
        System.out.println("我是1号消费者");
        // 1.创建工厂
        ActiveMQConnectionFactory activeMQConnectionFactory = new ActiveMQConnectionFactory(ACTIVEMQ_URL);

        // 2.创建连接
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();

        // 3.创建session
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

        // 4.创建目的地
        Topic topic = session.createTopic(TOPIC_NAME);

        // 5.创建消费者
        MessageConsumer messageConsumer = session.createConsumer(topic);

        // 6.接受消息
        while (true) {
            TextMessage textMessage = (TextMessage) messageConsumer.receive(4000L);
            if (textMessage != null){
                System.out.println("消费者接受到消息"+textMessage.getText());
            }else{
                break;
            }
        }
        
        // 7.关闭资源
        messageConsumer.close();
        session.close();
        connection.close();
    }
}
```



## 二、JMS

### 1.什么是JMS

------

JMS是JavaEE体系中13中技术栈中的一种技术栈, 全称为 Java Message Service  **Java消息服务** 

### 2.  JMS消息的可靠性

------

#### **2.1 PERSISTENCE 持久性参数设置:**

- 队列

  ```java
  /**
   * 非持久化:服务器宕机,消息丢失不存在
   */
  messageProducer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
  /**
   * 持久化:服务器宕机,消息依然存在
   */
  messageProducer.setDeliveryMode(DeliveryMode.PERSISTENT);
  ```

- 主题

  <!--1.一定要先运行一次消费者,相当于向ActiveMQ注册,类似于,我订阅了这个主题,
      2.然后在运行生产者发送消息,此时,无论消费者是否在线,都会接受到,如果不在线,下次连接的时候,会把没有收到的消息都收下来.-->

  **持久化消费者**

  ```java
  TopicSubscriber subscriber = session.createDurableSubscriber(topic, "remark");
  connection.start();
  ```

  **持久化生产者**

  ```JAVA
  MessageProducer messageProducer = session.createProducer(topic);
  messageProducer.setDeliveryMode(DeliveryMode.PERSISTENT);
  ```
  
  

#### 2.2 消息生产者的事物(事物偏向于生产者)

```java
Session session = connection.createSession(参数1, Session.AUTO_ACKNOWLEDGE);
```

ActiveMQ获取的Connection链接创建session会话时,**参数1**表示是否以事物的方式提交 **false** 不以事物的方式提交,**true** 以事物的方式提交,若不以事物的方式提交,默认自动提交,若以事物的方式提交,则发送消息后,必须在**session关闭前**手动提交到队列中.(代码如下)

```java
        /**
         * 基本提交关闭操作
         */
     	session.commit();
    	session.close();  
       /**
         * 事物在消息推送出错时会回滚,解决出现消息丢失或程序错误的问题
         */
        try {
            session.commit();
        }catch (Exception e){
            session.rollback();
        }finally {
            if (null != session){
                session.close();
            }
        }
```



#### 2.3 消息消费者的事物

```java
Session session = connection.createSession(参数1, Session.AUTO_ACKNOWLEDGE);
```

<u>ActiveMQ获取的Connection链接创建session会话时,**参数1**表示是否以事物的方式提交 **false** 不以事物的方式提交,**true** 以事物的方式提交,若不以事物的</u>

<u>方式提交,则消费消息后,ActiveMQ**未消费消息**的数据不会变化,会出现**同一条消息重复消费**的问题.(代码同上相同)</u>



#### 2.4 消息非事务模式下的消费者签收

```java
/**
 * 自动签收 AUTO_ACKNOWLEDGE
 */
Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
/**
 * 手动签收 CLIENT_ACKNOWLEDGE
 */
Session session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);
TextMessage textMessage = (TextMessage) messageConsumer.receive(1000L);
textMessage.acknowledge();
```

| 事务  |         签收模式         |          调用方法          |                             结果                             |
| :---: | :----------------------: | :------------------------: | :----------------------------------------------------------: |
| flase |  AUTO_ACKNOWLEDGE 自动   |                            |             消费者接受到消息后,自动进行签收操作              |
| flase | CLIENT_ACKNOWLEDGE  手动 | textMessage.acknowledge(); | 消费者接受到消息后,得到消息对象,调用acknowledge()方法手动进行签收 |

#### 2.5 消息事务模式下的消费者签收

事务模式下,签收无所谓,不影响消费,只要有**commit**操作,不会出现重复消费的情况,具体来说,签收只对非事务模式下的消费者消费消息时有影响



## 三、 Broker

### 1. 什么是Broker

**一个ActiveMQ服务器实例**

Broker就是实现了用代码形式启动ActiveMQ将MQ**嵌入到Java代码**中,以便随时调用启动,再用的时候再去启动,这样节省资源,也保证了可靠性

### 2. 根据不同的conf配置文件启动不同的ActiveMQ实例

启动ActiveMQ实际的配置文件时是 ActiveMQ安装路径下的**conf文件夹下的activemq.xml文件**,根据不同的实例我们可以有多个xml配置文件.如何以不同的xml文件作为启动配置启动MQ服务呢

在bin目录下 执行命令 

**linux环境:**

```
./activemq start xbean:file:/xml位置文件路径
```

### 3. 嵌入式的Broker

**在java中用代码启动嵌入式的Broker**

```java
	public static void main(String[] args) throws Exception {
        //ActiveMQ也支持vm中通信基于嵌入式的Broker
        BrokerService brokerService = new BrokerService();
        brokerService.setUseJmx(true);
        brokerService.addConnector("tcp://localhost:61616");
        brokerService.start();
    }
```

