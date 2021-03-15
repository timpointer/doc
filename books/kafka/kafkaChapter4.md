## Chapter4 Kafka Consumers

### Consumers and Consumer Groups

Kafka consumers are typically part of a consumer group. When multiple consumers are subscribed to a topic and belong to the same consumer grorup, each consumer in the group will receive messages from a different subset of the parititions in the topic.

### Consumer Groups and Partition Rebalance

## Commits and Offset
消费者提交每个分片的offset到kafka的__consumer_offset这个topic上。
当有消费者宕机或者有新消费者加入时，会出发再平衡，每个消费者会收到新的分片的，然后从新分片的offset处开始更新数据。

### 管理Offset对客户端应用至关重要
自动提交Offset，默认客户端会每隔5秒自动提交上一次poll的offset。
手动同步提交，会重试，如果失败会上抛异常。
手动异步提交，不会重试，可以指定回调函数，用来记录日志。
如果想异步重试，注意需要控制提交顺序。

组合同步和异步提交
正常情况下采用异步，当要退出或再平衡时，使用同步。参考书上代码。

分片提交
当一次poll的数量太大时，可以采用分片提交，自己现实计数，并中途提交，参考书上。

### 监听再平衡事件
消费者系统在丢失分片所有权之前，进行一些自定义操作，比如关闭资源，清空积压任务，提交offset等等，参考书上代码。

### 从指定的offset消费消息
如果offset记录在kafka，则消费消息和记录offset不能在一个事务中。但如果将消息和offset都写进消费者自己的数据库则可以。
下次启动时，只需要从数据中取出对应分片的offset，并调用seek方法初始化消费者，就可以继续消费后续消息，达到只有一次的一致性，参考书上代码。

### 如何正确退出
唯一的方法就是在另一个线程里面调用consumer.wakeup方法。消费者会上抛一个WakeUpException，记住在退出是调用consumer.close方法。客户端会自动通知协调者自己已退出消费组，系统会自动再平衡，而不需要等待session超时。

## 单独的消费者，为什么只使用消费者而不同消费组
有些情况下，就只有一个消费者，这时就不需要引入复杂的消费组，也没有再平衡。
只需要在创建消费者时指定topic或是具体的分片，但不同同时使用。
如果是具体分片，那么有新分片加入topic时，是不会被通知到的，需要自己去定期轮询分片信息。