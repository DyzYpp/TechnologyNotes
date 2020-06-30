# 								ActiveMQ

## 一、JMS

#### 1.什么是JMS

JMS是JavaEE体系中13中技术栈中的一种技术栈, 全称为 Java Message Service  **Java消息服务** 

#### 2.  JMS消息的可靠性

1.  PERSISTENCE **持久性参数设置**:

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

   2. **消息生产者的事物(事物偏向于生产者)**

      ```java
      Session session = connection.createSession(参数1, Session.AUTO_ACKNOWLEDGE);
      ```

      <u>ActiveMQ获取的Connection链接创建session会话时,**参数1**表示是否以事物的方式提交 **false** 不以事物的方式提交,**true** 以事物的方式提交,若不以事物的方式提交,默认自动提交,若以事物的方式提交,则发送消息后,必须在**session关闭前**手动提交到队列中.(代码如下)</u>

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

   3. 消息消费者的事物

      ```java
      Session session = connection.createSession(参数1, Session.AUTO_ACKNOWLEDGE);
      ```

      <u>ActiveMQ获取的Connection链接创建session会话时,**参数1**表示是否以事物的方式提交 **false** 不以事物的方式提交,**true** 以事物的方式提交,若不以事物的</u>

      <u>方式提交,则消费消息后,ActiveMQ**未消费消息**的数据不会变化,会出现**同一条消息重复消费**的问题.(代码同上相同)</u>

​						

#### 2. ActiveMQ生产者编码

```java
 	/**
     * mq链接的主机
     */
    private static final String ACTIVEMQ_URL = "mq服务器ip:61616";
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
        for (int i = 0; i < 5; i++) {
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
```



#### 3. ActiveMQ消费者编码

```java
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
                if (null != message && message instanceof TextMessage){
                   TextMessage textMessage =  (TextMessage)message;
                    try {
                        System.out.println("消费者接受到消息"+textMessage.getText());
                    } catch (JMSException e) {
                        e.printStackTrace();
                    }
                }
                if (null != message && message instanceof MapMessage){
                    MapMessage mapMessage =  (MapMessage)message;
                    try {
                        System.out.println("消费者接受到消息"+mapMessage.getString("k1"));
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



