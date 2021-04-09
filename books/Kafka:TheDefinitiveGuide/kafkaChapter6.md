# 可靠数据传输　Reliable Data Delivery　
可靠数据传输是系统的属性之一。就像性能一样，在设计之初就必须考虑进来。不可能在系统完成后添加进来。可靠性是一个系统的资产，当我们谈到kafka保证可靠性，你必须思考整个系统和实际场景。整合kafka的系统和kafka自身一样重要。因为可靠性不仅仅是一个人的职责。每个人包括kafka管理员，linux管理员，网络和存储管理员和应用开发者必须同心协力一起构建一个可靠系统。
Kafka对于可靠数据传输是非常灵活的。Kafka有很多使用场景，从跟踪网站点击到信用卡支付。一些场景需要极致的可靠同时另一些是速度和简单性大于可靠性。Kafka提供了足够的配置项和灵活的客户端(client API)足以允许在任何场景中权衡。
因为它的灵活，也容易弄伤自己，当你认为系统需要可靠而实际并不需要时。我们先讨论不同的可靠性和他们在kafka中意味着什么。然后介绍副本机制和对可靠性有什么帮助。再讨论客户端，生产端，消费端在不同场景中如何使用。最后我们需要验证系统的可靠性而不仅仅相信它是可靠的。

## 可靠性保证

* Kafka提供单个分片上的消息有序。
* 生产者有多个等级的被提交"committed"。写到所有副本，写到leader，发送到网络。
* 一旦被提交，只要有一个副本活着，消息就不会丢失。
* 消费者只能读到被提交的消息。

这些基本保证是用来构建可靠系统的，但仅凭这些还不能保证系统完全可靠。在构建可靠系统时需要做很多权衡，kafka提供了很多配置让开发和管理员能够做这些权衡。权衡通常涉及到可用性，高吞吐，低延迟以及硬件成本。我们先来介绍机制，再看配置参数。

## Replication

Kafka的副本机制是可靠性的核心保证。一个消息被写到多个副本来确保宕机时的消息持久性。
每个Topic被分成多个分片，它是基本的数据块。一个分片被存到一个磁盘上。Kafka保证单个分片的消息是有序的，分片要么在线（available)或离线（unavailable)。每个分片有多个副本，其中一个是leader。所有消息都有leader来负责消息的接受和发送。其他副本只保持和leader的数据同步。如果leader下线了，其中一个同步的副本会变成新的leader。

一个副本被认为是同步有几种情况，要么是分片的leader
如果是follower
* 6秒一次的心跳上报
* 至少10秒从leader那拉去一次数据
* follower的数据和leader数据延迟差小于10秒

以上任何一种情况失败，就被认定为下线。一个下线的副本可以重新连上，恢复会同步状态。如果网络抖动则持续很短就恢复，如果broker宕机则会下线更久。

一个有延迟的同步副本会拖慢消费者的节奏，应该消费者需要等到消息被提交后（committed）才能收到。但一旦副本被认定为下线，就不会对消费者有影响，所以配置更少的同步副本，能让消费更快，但丢数据的可能性更大。

## Broker Configuration

在broker上有三个参数，可以是broker级别或topic级别。比如一个银行项目，管理员把整个集群设置为最高可靠性，但有一个用户吐槽topic设为低级别。

### Replication Factor
```
replication.factor
```
默认副本因子为3。意味着分片被同步到三个不同的broker上。用户可以修改这个选项。
N个副本意味着你能丢失N-1个broker时仍可读写数据。所以更高的副本因子有更高的可用性，可靠性，更少的灾难。另一面N个副本，会有N个备份数据，N倍磁盘空间。
一个topic多少副本合适？答案是一个topic有多重要和你愿意为高可用付出多少成本。
如果一个topic不可用一段时间是可以接受的，那只需要一个副本。一般需要高可用的场景都是3个，极少情况会不够，比如银行里有见到过5副本的配置。
副本被安放在哪里很重要，默认副本分片会分配到不同的broker上，但这不能避免机架整体故障。新版本能感知机架位置，使分片分布在不同的机架上。

### Unclean Leader Election
这个选项只有broker级别。
```
unclean.leader.election.enable
```
有同步副本存在的时候，这时leader选取是干净的"clean"，没有被提交的数据丢失，因为被提交的数据已经同步到所有同步副本了。
但当没有同步副本的时候，leader宕机会怎样？
这种场景有两种case会发生。1.有两个宕机的follower。2.有两个因为网络延迟导致不同步的follower。

此时要做困难的决定 
* 如果我们不同意不同步的副本成为新leader，分片会一致下线直到原leader重新上线，这可能要花几小时。
* 当我们允许不同的副本成为leader，我们将丢失没有被同步到副本的数据，同时会导致消费者数据不一致。为什么？想象一下，leader数据写到100-200，但其他副本只同步到0-100。此时leader宕机，其他副本变为leader。会有新的100-200的消息写到新leader。首先一些消费会读到老100-200的消息，有些则是新的，一些则是混合新旧的。当原leader从新上线，同步新leader的数据时。这时，它先删除哪些于新数据冲突的数据，此后这些数据就不会被其他消费者消费了。

总结一下，如果我们允许脏选举，会冒丢数据和一致性的风险，如果不允许，将面临低可用性，需要等原leader回来。
通常在一些关键的场景比如银行时，被设置为不允许。是服务几分钟或几小时不可用严重还是支付数据异常来的严重？

### Minimun In-Sync Replicas
```
min.insync.replicas
```
假设我们有3个副本。最小同步数配置为2.当有三个副本在线时，正常工作，有两个也一样。但当两个副本宕机时，生产者发送消息会收到NotEnoughReplicaseException。消费者能读要现存的数据。这种情况一下，一个同步副本变成了只读。这避免了当藏读发生时，数据丢失的情况。为了从只读模式中恢复，我们必须救回一台下线的副本。

## Using Producers in a Relicable System

### Send Acknowledgments

### Configuring Producer Retries

### Additional Error Handling

## Using Consumers in a Reliable System

### Important Consumer Configuration Properties for Reliable Processing

### Explicitly Committing Offsets in Consumers

## Validating System Reliability

### Validating Configuration

### Validating Applications

### Monitoring Reliability in Production