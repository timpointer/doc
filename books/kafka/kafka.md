

## Kafka Producers
不同的用例暗示了各种各样的需求。
我们开发产生需求前需要创建ProducerRecord
发送消息，分为几个步骤，首先创建ProducerRecord类，然后经过序列化节点将数据序列化二进制格式，在经过分片节点选择对应的分片，最后添加到对应的待发送队列中，由另一个线程去发送消息到kafka的broker。
发送接口很简单，大多数的控制行为在配置阶段就已经设定好。
主要的发送方式有三种，对应不同的错误报告机制。
Fire-and-forget，Synchronous send，Asynchronous send
你可以通过增加线程来提升，在提升写入端性能。