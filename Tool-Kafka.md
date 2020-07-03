是一个**分布式**的基于发布订阅的消息队列，主要应用于大数据实时处理领域。



消息队列的两种模式：

点对点：RabbitMQ支持

发布订阅：

- 消费者主动拉取：长轮询
- 服务者主动推送



**Kafka是基于发布订阅**的RabbitMQ是都支持的



架构：

生产者    ======>   kafka集群  ======>   消费者消费消息



作为分布式集群，为了保证高可用，数据会有主从，并存放在不同的机器上，比如说有一个主题，有若干个分区(分区也是在不同机器上的)，那么每个分区都有从节点作为备份，存放在别的服务器上。



副本数不能大于可用服务器数目



 ![img](https://images2018.cnblogs.com/blog/1228818/201805/1228818-20180507192145249-1414897650.png) 



### 工作流程：

1. 开启多个broker，即服务器
2. 生产者创建主题，可以指定分区数目以及每个分区的副本数（主加副）。
3. 生产者向指定主题发送消息。然后这些消息会被**分散地分配**同一个主题但是不同的分区中，在某一个分区中，你可以看到消息的id是从0到n，但是他们实际上并不是连续的。
   1. 副本主动请求主数据同步，将leader的消息拷贝到自己这边。
   2. 生产者发送过来的消息，会被存储到broker的磁盘中，追加在.log文件中
4. 消费者通过主题向leader取出消息。
   1. 每次从分区中取出消息，会记录当前读取位置，以便下次访问。
   2. 对kafka来说他不能保证全局有序，只能保证某个分区是有序的。也就是说每个分区维护一个消息队列生产者随机地向不同的队列发送消息。



#### 如何确定分区partition

1. 可指定分区号
2. 使用key的hash取余得到分区数 
3. 随机===》轮询

#### 如何保证可靠性

确保有副本节点同步完成后再返回确认信息

- 半数以上
- 全部副本都同步完成（kafka方案）如果在同步过程中有一个副本挂了那么同步就不能完成，也不会向生产者回复确认信息。



 kafka在接收到生产者发送的消息之后，会根据均衡策略将消息存储到不同的分区中。因为每条消息都被append到该Partition中，属于顺序写磁盘，因此效率非常高（经验证，顺序写磁盘效率比随机写内存还要高，这是Kafka高吞吐率的一个很重要的保证） 



### ACK机制（生产者和服务器之间）

首先这个ACK机制是保证数据被完整接收的一种策略

- 0 不等待同步消息：生产者发消息给broker，不等待broker返回消息就可以发送下一条
- 1 要leader收到消息并确认后才可以发下一条数据，此时如果leader收到消息但备份到follower失败的话，后面leader失效的话数据就会丢失
- -1 leader收到消息，follower备份成功后返回ACK消息。



### 高可用

#### Replication(主从复制)



### API

1. 导入kafka依赖

2. Producer
   1. 创建Properties对象
      1. 指定集群
      2. 指定ACK应答级别
      3. 重试次数
      4. 批次大小
      5. 等待时间
      6. RecordAccumulate缓冲区大小（发送者先把消息放到这个缓冲区中）
      7. 序列化工具
   2. 使用Properties创建KafkaPrducer对象
   3. KafkaPrducer对象通过ProducerRecord对象（指定topic,partition）发送数据，重载的send方法可以传入callback对象，在发出数据之后执行回调函数。
   
3. Consumer    不是线程安全的

   首先回到一个概念，就是消费者组，一个组内有多个消费者，不存在**同组内多个**消费者消费一个分区的情况，只会有一个组内一个消费者占用一个或者多个分区的情况。在这个规则下会保持动态平衡：

   > 消费者组的成员是动态维护的：如果一个消费者故障。分配给它的分区将重新分配给同一个分组中其他的消费者。同样的，如果一个新的消费者加入到分组，将从现有消费者中移一个给它。这被称为`重新平衡分组`，并在下面更详细地讨论。当新分区添加到订阅的topic时，或者当创建与订阅的正则表达式匹配的新topic时，也将重新平衡。将通过定时刷新自动发现新的分区，并将其分配给分组的成员。

   还可以使用手动分配策略，指定某个消费者所对应的分区。

   1. 创建Properties对象

      ```java
      Properties props = new Properties();
           props.put("bootstrap.servers", "localhost:9092");
      	 //消费者组
           props.put("group.id", "test");
           props.put("enable.auto.commit", "true");
           props.put("auto.commit.interval.ms", "1000");
           props.put("key.deserializer","org.apache.kafka.common.serialization.StringDes"); 	   props.put("value.deserializer","org.apache.kafka.common.serialization.Strinr");
          
      ```

   2. 创建消费者对象 `new KafkaConsumer<>(props);`

   3. 订阅主题，使用List  `consumer.subscribe(Arrays.asList("foo", "bar"));`

   4. 使用poll持续取出信息，返回结果封装到ConsumerRecords对象中, 使用了Map<java.lang.Integer,java.util.List<ConsumerRecord>>保存获得的消息。

   5. ConsumerRecords<String, String> records = consumer.poll(100);

      ```java
   //poll的参数是时间（ms）,表示在这个时间内拉取消息，返回一个集合
      ConsumerRecords<String, String> records = consumer.poll(100);
      ```
   
      订阅一组topic后，当调用poll(long）时，消费者将自动加入到组中。只要持续的调用poll，消费者将一直保持可用，并继续从分配的分区中接收消息。此外，消费者向服务器定时发送心跳。 如果消费者崩溃或无法在session.timeout.ms配置的时间内发送心跳，则消费者将被视为死亡，并且其分区将被重新分配。
   
   6. 如果希望订阅某主题的某个分区，这样的话就不能用subscribe了

      ```java
         String topic = "foo";
         TopicPartition partition0 = new TopicPartition(topic, 0);
         TopicPartition partition1 = new TopicPartition(topic, 1);
         consumer.assign(Arrays.asList(partition0, partition1));
      ```
   
    7. 此外，还可以直接指定访问位置、完成流量控制（即多个分区消费哪个空闲一点的）
   
       > https://www.orchome.com/451



### 关于offset

​	kafka为分区中的每条消息保存一个`偏移量（offset）`，这个`偏移量`是该分区中一条消息的唯一标示符。也表示消费者在分区的位置。例如，一个位置是5的消费者(说明已经消费了0到4的消息)，下一个接收消息的偏移量为5的消息。实际上有两个与消费者相关的“位置”概念：

- 上一次消费的下一位
- 安全保存的最后偏移量

- 一般来说，消费者第一次消费一条消息，都会从当前分区的偏移值开始消费，消费一条消息后偏移值都会自动加一并保存起来，下一次如果有同组的消费者来到这个分区消费的话会从偏移值处开始消费，这样也很合理。但是在一些场景中，我们并不希望每次读取都重新提交保存一个偏移值，因为可能一批的消息才构成一次完整的处理，那我们就可以先关闭自动提交，改为手动提交

```java
 //设置手动提交消息偏移
properties.put("enable.auto.commit","false");

//一次拉取的最大消息条数
properties.put("max.poll.records",10);
//消费50条消息后提交一次，下一个消费者如果从这个分区消费消息会从50开始
while (true){
        ConsumerRecords<String,String> records = consumer.poll(10);
        for(ConsumerRecord<String ,String> record : records){
              count ++;
              if(count == 50)
                  consumer.commitSync();
              System.out.println(record.topic() + "," + record.partition() + "," + 										 record.offset() + "," + record.key() + "," + 											 record.value());
         }
         System.out.println(count);
}
```

- Zookeeper保存offset





#### 什么是Kafka

是一种基于发布订阅模式的消息队列，是一个分布式，重复的日志服务



#### 有什么优点

- 吞吐量大
- 方便进行伸缩
- 持久的：消费完并不会消失，存在硬盘中，集群之间还会进行复制



#### 能不能没有Zookeeper

不能，必须通过Zookeeper，Kafka的broker和Consumer都是连接Zookeeper的，而Producer是直接连接Broker的。

作用：

- 注册broker，使得各个单机组合成一个集群，每一个Broker都要到Zookeeper下创建自己的节点。并记录好IP地址和端口信息。

- 注册Topic：每创建一个Topic都会创建一个Topic节点，像这样   /brokers/topics ,然后如果有Broker加入并且说我要保存这个Topic的消息就会创建一个这样的节点 /brokers/topics/login/3->2   表示Broker ID为3要存储这个Topic的消息，并且有两个分区

- 生产者负载均衡：使生产者消息均匀发到不同Broker

- 消费者负载均衡

- 记录offset

- 消费者注册：会创建如下节点 /consumers/[group_id]/ids/[consumer_id] 

   ![img](https://upload-images.jianshu.io/upload_images/3149801-0d2ed2bd8b7bec25.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp) 



#### Partition和Broker ,leader,follower的关系

![img](https://upload-images.jianshu.io/upload_images/3149801-dae3a4836701dc12.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp) 

leader是针对于一个partition来说的，如上图，每一个分区都有一个leader但是有两个leader,理论上上面用三台服务器就可以做到，也就是主+从<=服务器总数

leaderController是一台控制leader选举的服务器，一般指定为第一个注册到ZK的broker，如果某个partition的leader宕机了，这个leaderController就会去宕机的leader的follower节点选择为下一个leader节点。



#### Producer是如何生产出一条消息的

把消息准备好，带上Topic和partition发送到缓冲区，然后缓冲区就会发给Broker



#### 使用场景

- 日志收集
- 消息系统：生产者和消费者解耦
- 监控用户行为：通过不同行为有对应的Topic可以结合AOP
- 流式处理，流量削峰



