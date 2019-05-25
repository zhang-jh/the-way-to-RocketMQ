# RocketMQ组件

### RocketMQ网络部署图
[RocketMQ网络部署图]: https://github.com/zhang-jh/the-way-to-RocketMQ/blob/master/images/reocketMq.png
![RocketMQ网络部署图]

RocketMQ消息队列集群中的几个角色：
* NameServer：在MQ集群中做的是做命名服务，更新和路由发现 broker服务；
* Broker-Master：broker 消息主机服务器；
* Broker-Slave：broker 消息从机服务器；
* Producer：消息生产者；
* Consumer：消息消费者。

RocketMQ集群的通信如下(一部分)：
* Broker启动后需要完成一次将自己注册至NameServer的操作；随后每隔30s时间定期向NameServer上报Topic路由信息；
* 消息生产者Producer作为客户端发送消息时候，需要根据Msg的Topic从本地缓存的TopicPublishInfoTable获取路由信息。如果没有则更新路由信息会从NameServer上重新拉取；
* 消息生产者Producer根据所获取的路由信息选择一个队列（MessageQueue）进行消息发送；Broker作为消息的接收者接收消息并落盘存储。
&nbsp;&nbsp;

### 重要概念
* 生产者（Producer）：消息发送方，将业务系统中产生的消息发送到brokers（brokers可以理解为消息代理，生产者和消费者之间是通过brokers进行消息的通信），rocketmq提供了以下消息发送方式：同步、异步、单向。    
* 生产者组（Producer Group）：相同角色的生产者被归为同一组，比如通常情况下一个服务会部署多个实例，这多个实例就是一个组，生产者分组的作用只体现在消息回查的时候，即如果一个生产者组中的一个生产者实例发送一个事务消息到broker后挂掉了，那么broker会回查此实例所在组的其他实例，从而进行消息的提交或回滚操作。    
* 消费者（Consumer）：消息消费方，从brokers拉取消息。站在用户的角度，有以下两种消费者。    
  * 主动消费者（PullConsumer）：从brokers拉取消息并消费。    
  * 被动消费者（PushConsumer）：内部也是通过pull方式获取消息，只是进行了扩展和封装，并给用户预留了一个回调接口去实现，当消息到底的时候会执行用户自定义的回调接口。    
* 消费者组（Consumer Group）：和生产者组类似。其作用体现在实现消费者的负载均衡和容错，有了消费者组变得异常容易。需要注意的是：同一个消费者组的每个消费者实例订阅的主题必须相同。    
* 主题（Topic）：主题就是消息传递的类型。一个生产者实例可以发送消息到多个主题，多个生产者实例也可以发送消息到同一个主题。同样的，对于消费者端来说，一个消费者组可以订阅多个主题的消息，一个主题的消息也可以被多个消费者组订阅。    
* 消息（Message）：消息就像是你传递信息的信封。每个消息必须指定一个主题，就好比每个信封上都必须写明收件人。    
* 消息队列（Message Queues）：在主题内部，逻辑划分了多个子主题，每个子主题被称为消息队列。这个概念在实现最大并发数、故障切换等功能上有巨大的作用。    
* 标签（Tag）：标签，可以被认为是子主题。通常用于区分同一个主题下的不同作用或者说不同业务的消息。同时也是避免主题定义过多引起性能问题，通常情况下一个生产者组只向一个主题发送消息，其中不同业务的消息通过标签或者说子主题来区分。    
* 消息代理（Broker）：消息代理是RockerMQ中很重要的角色。它接收生产者发送的消息，进行消息存储，为消费者拉取消息服务。它还存储消息消耗相关的元数据，包括消费群体，消费进度偏移和主题/队列信息。    
* 命名服务（Name Server）：命名服务作为路由信息提供程序。生产者/消费者进行主题查找、消息代理查找、读取/写入消息都需要通过命名服务获取路由信息。
消息顺序（Message Order）：当我们使用DefaultMQPushConsumer时，我们可以选择使用“orderly”还是“concurrently”。    
  * orderly：消费消息的有序化意味着消息被生产者按照每个消息队列发送的顺序消费。如果您正在处理全局顺序为强制的场景，请确保您使用的主题只有一个消息队列。注意：如果指定了消费顺序，则消息消费的最大并发性是消费组订阅的消息队列数。    
  * concurrently：当同时消费时，消息消费的最大并发仅限于为每个消费客户端指定的线程池。注意：此模式不再保证消息顺序。    
&nbsp;&nbsp;


