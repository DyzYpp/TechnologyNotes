# 第一章 Kafka概述

## 1.1 定义

Kafka是一个分布式的基于发布/订阅模式的消息队列 (Message Queue)，主要用于大数据实时处理领域

## 1.2 消息队列

#### 1.2.1 传统消息队列的应用场景

**MQ传统应用场景**

------

- **同步处理**

![](img\imgFile\MessageQueue.png)

------



- **异步处理**



![](img\imgFile\MessageQueueAsync.png)

------

**使用消息队列的好处**

1. **解耦**

   允许你独立的扩展或修改两边的处理过程，只要确保他们遵守同样的接口约束

2.  **可恢复性**

   系统的一部分组件失效时，不会影响到整个系统，消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。

3. **缓冲**

   有助于控制和优化数据流经过系统的速度，解决生产消息和处理消息速度不一致的情况。

4. **灵活性 和 峰值处理能力**

   在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见，如果为以能处理这类峰值访问为标准来投入资源随时待命无疑时巨大的浪费，使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。

5. **异步通信**

   很多时候，用户也不需要立即处理消息，消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它，想向队列中放入多少消息就放多少，在后在需要的时候再去处理它们。

   ------

#### 1.2.2  消息队列的两种模式

1. 点对点模式（一对一，消费者主动拉去数据，消息收到后消息清除）

   消息生产者生产消息发送到Queue中，然后消息消费者从Queue中取出并且消费消息，消息被消费以后，queue中不在存储，所以消息消费者不可能消费到已经被消费的消息，queue支持存在多个消费者，但对于一个消息而言，只会有一个消费者可以消费。

   

![](img\imgFile\点对点模式.png)

 2. 发布/订阅模式（一对多，消费者消费数据后不会清除消息）

    消息生产者（发布）将消息发布到 topic 中，同时有多个消费者订阅该topic，和点对点方式不同，发布到topic的消息会被所有订阅者消费。

    1. 生产者主动推送
    2. 消费者主动拉取

<img src="img\imgFile\发布订阅模式.png"  />

## 



## 1.3 Kafka基础架构



![](img\imgFile\kafka.png)

- Producer : 消息生产者，就是向 kafka broker 发消息的客户端；
- Consumer：消息消费者，就是向 kafka broker 取消息的客户端；
- Consumer group (CG)：消费者组，由多个Consumer组成，消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个组内消费者消费；消费者组之间互不影响。所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者
- Broker：一台 kafka 服务器就是一个 broker，一个集群由多个broker组成，一个broker可以容纳多个topic；
- Topic ：可以理解为一个队列，生产者和消费者面向的都是topic
- Partition：为了实现扩展性，一个非常大的topic可以分布在多个broker上，一个topic可以分为多个partition，每个partition是一个 有序的队列。
- Replica：副本，为保证集群中的某个节点发生故障时，该节点上的partition数据不丢失。且 kafka仍能够继续工作，kafka提供了副本机制，一个topic的每个分区都有若干个副本，一个leader和若干个follower
- Leader：每个分区多个副本的 “主” ，生产者发送数据的对象，以及消费者消费数据的对象都是leader;
- follower：每个分区多个副本的“从”，实时从leader中同步数据，保持和leader数据的同步，leader发生故障时，某个follower会成为新的leader。

## 



## 1.4 kafka命令行操作

**启动服务**

**bin/kafka-server-start.sh -daemon config/server.properties**   



**创建Topic**

​	**创建失败：**

![](img\imgFile\kafka创建topic报错.png)

​	出现如上报错：当前的可用kafka服务只有一个，但副本数大于可用服务数。创建topic时，副本数应小于当前可用kafka服务数。

​	原因：待研究。

​	**创建成功：**

![](img\imgFile\成功创建topic.png)

​	 --bootstrap-server localhost:9092  指定该topic创建在哪一个个kafka服务上

​	 --replication-factor 指定副本数 需小于当前可用kafka服务数

​	 --partitions  分区数

​	 --topic topic名称

**启动生产者**

![](img\imgFile\启动生产者.png)

​	 如上图所示，'my name is test'  即为发送的消息   kafka通常为集群部署，故在发送消息时，需要指定服务器。



**启动消费者**

![](D:\i-exercise\TechnologyNotes\img\imgFile\启动消费者.png)

​	 --from-beginnging  当消费者宕机重启后，会继续消费宕机期间生产者发送的消息。理论上不会有消息丢失。

​	

**删除topic**

![](img\imgFile\删除topic.png)



**查看topic列表**

![](img\imgFile\查看topic列表.png)

