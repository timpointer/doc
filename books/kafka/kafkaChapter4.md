## Chapter4 Kafka Consumers

### Consumers and Consumer Groups

Kafka consumers are typically part of a consumer group. When multiple consumers are subscribed to a topic and belong to the same consumer grorup, each consumer in the group will receive messages from a different subset of the parititions in the topic.

### Consumer Groups and Partition Rebalance

