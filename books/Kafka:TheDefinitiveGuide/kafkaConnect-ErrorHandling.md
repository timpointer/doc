# Fail fast

有时候在消费消息遇到坏的数据，如果此时配置为
```
errors.tolerance = none
```
则消费者会堵塞，然后将错误记录在日志中。

# YOLO:沉默忽略坏消息
```
errors.tolerance = all
```
我只希望继续消费，忽略所有坏消息。默认情况下不会记录消费被丢弃的事实。以为着需要通过其他途径来监控。最简单的方法是对比输入和输出的数据条数。
更靠谱的方式是用JMX metrics监控错误消息比率。
这时我们并不知道是哪条消息被丢弃了，谁关心呢！但真实场景中我们往往应该关心任何一条被丢弃的消息。这时dead letter queue就出现了。

# Route messages to a dead letter queue
Kafka connect能配置将不能被消费的消息（比如反序列化失败）加入死亡队列，Kafka中另一个Topic。有效的消息继续被消费，不会被都塞。之后根据需要对死亡队列中无效的消息进行忽略，修复或重试。

# 查看无效消息原因
## Recording the failure reason for a message: Message headers
```
errors.deadletterqueue.context.headers.enable = true
```
## Recording the failure reason for a message: Log
```
errors.log.enable = true
```
# Processing messages from a dead letter queue
死亡队列就是一个普通的kafka topic，用一般kafka客户端来消费就可以，将消息回写到原来的目的地。

# 总结
错误处理是可靠数据流处理的重要一环。依据数据如何被使用，你有两个选择。如果上游数据很重要，不希望错误扩散，就应该堵塞下游消费，默认选项就是这么做的。
另一方面，如果你只是分析数据，或是一些不太重要的处理，则尽可能保持流畅通。自定义错误处理，应该就采用死亡队列和密切留意监控指标。





