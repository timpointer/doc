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
