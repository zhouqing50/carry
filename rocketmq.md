##### 使用场景


1. 系统解耦
2. 异步
3. 并行

##### 基本概念

----------

- Producer，Producer Group
消息生产者，负责产生消息，一般由业务系统负责产生消息。
通常具有同样属性（处理的消息种类-topic、以及消息处理逻辑流程—分布式多个客户端）的一些producer可以归为同一个group。在事务消息机制中，如果某条发送某条消息的producer-A宕机，使得事务消息一直处于PREPARED状态并超时，则broker会回查 同一个group的其他producer，确认这条消息应该commit还是rollback。

- Consumer，Consumer Group
消息消费者，负责消费消息，一般是后台系统负责异步消费。
具有同样逻辑消费同样消息的consumer，可以归并为一个group。同一个group内的消费者，可以共同消费对应topic的消息，达到分布式并行处理的功能。
-  -  广播消费
一条消息被多个 Consumer 消费，即使返些 Consumer 属亍同一个 Consumer Group，消息也会被 ConsumerGroup 中的每个 Consumer 都消费一次，广播消费中的 Consumer Group 概念可以讣为在消息划分方面无意丿。
- -  集群消费（默认）
一个 Consumer Group 中的 Consumer 实例平均分摊消费消息。例如某个 Topic 有 9 条消息，其中一个Consumer Group 有 3 个实例（可能是 3 个迕程，或者 3 台机器），那每个实例只消费其中的 3 条消息。
- -  Push Consumer
Consumer 的一种，应用通常吐 Consumer 对象注册一个 Listener 接口，一旦收到消息，Consumer 对象立刻回调 Listener 接口方法。
- -  Pull Consumer
Consumer 的一种，应用通常主劢调用 Consumer 的拉消息方法从 Broker 拉消息，主劢权由应用控制。

- Broker
消息中转角色，负责存储消息，转収消息，一般也称为 Server。在 JMS 规范中称为 Provider。

- Topoic
消息的逻辑管理单位。消息一级标签，类似书的一级目录

- Tag
消息二级标签，类似书的二级目录，
broket端的consumequeue会保存tag的hashcode，方便过滤消息

- Queue
消息的物理管理单位。一个Topic下可以有多个Queue，Queue的引入使得消息存储可以分布式集群化，具有了水平扩展的能力。


##### 基本原理

----------



- RocketMQ以Topic来管理不同应用的消息。对于生产者而言，发送消息是，需要指定消息的Topic，对于消费者而言，在启动后，需要订阅相应的Topic，然后可以消费相应的消息。Topic是逻辑上的概念，在物理实现上，一个Topic由多个Queue组成，采用多个Queue的好处是可以将Broker存储分布式化，提高系统性能。


- producer将消息发送给Broker时，需要制定发送到哪一个队列中，默认情况下，producer会轮询的将消息发送到每个队列中（所有broker下的Queue合并成一个List去轮询）。
