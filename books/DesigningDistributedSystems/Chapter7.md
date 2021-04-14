# 事务的概念

## ACID的含义
A 原子性，几个操作就想一个操作一样，要么一起成功要么一起失败。
C 一致性，确保数据的一致性。
I　隔离性，并发的事务相互之间不会影响，比如不会看到其他事务执行了一半的数据。　
D　持久性，事务结束后，所有结果都被持久化下来。

## 单个对象和多对象操作

有些数据库提供操作单个对象的原子性操作，比如read-modify-write或comapre-and-set。
这种单个对象的只能算作“轻量级事务”。一个事务通常被认为是将操作多个对象的多个操作打包会一个整体来执行。

### 针对多对象事务的需求
许多数据库因为夸分片的事务实现起来太难，已经摒弃事务了。然而并没有什么本质问题阻碍在一个分布式数据库中实现事务。
但我们真的需要多对象事务吗？可以应用都只都用单对象操作吗？
有许多场景需要多对象之间协调：
- 在关系型数据库中，一条记录总是外键关联到另一张表的记录。事务能确保正确更新多张表的记录。
- 在文档性数据库中，需要属性被放到了同一个文档中，许多操作是但对象操作。但由于缺乏join功能，数据被denormailzed，更新数据时，其他视图数据也一起被同步更新，这时就要用到多对象事务。
- 在数据库中，二级索引，也被看做是第二个文档。插入数据时，要同步更新二级索引。

许多应用可以不借助事务来实现。然而没有事务导致的并发问题使得错误处理机制变的异常复杂。

## 处理错误和中断
事务的一个关键特性就是可以安全重试。虽然重试一个中断的事务是一种简单有效的错误处理机制，但它不是完美的。
- 如果事务成功，回传Ack时，网络错误导致丢失，会导致客户端错误重试。
- 如果由于负载过高导致的失败，重试只会加重服务器负担。
- 重试只对暂时性错误有效，永久性错误重试于事无补。
- 如果事务之外还有其他边际效应，比如发送email。需要其他机制来确保一致性。
- 如果重试时客户端宕机，请求数据将丢失。

# 弱隔离等级
如果两个事务操作不同的数据，他们就可以安全的并行，因为他们互相不依赖，并发问题只有发生在多个事务操作相同的数据时。
并发问题很难测试和避免，所以数据库尝试提供事务隔离。理论上隔离让你的生活更简单。但serializable级别的隔离有性能损耗，许多数据库不愿意支付价格。
因此采用更弱的隔离等级。使用弱隔离等级，并不能避免所有问题，所以我们要理解这些并发问题，知道不同隔离等级分别解决了什么问题。那样才能构建出可靠正确的应用。


## Read Committed
两个保证
1. 当读数据时，只能读到已经被提交的数据
2. 当写数据时，只能覆写已经被提交的数据

### No dirty reads
避免的问题
- 如果一个事务修改几个对象，脏读会读到数据的中间状态。
- 如果事务被中断，脏读会读到没有提交成功的数据。

### No dirty writes
- 脏写，会覆盖其他事务的中间状态。

有一种情况Read committed避免不了，就是多线程更新counter。及时避免脏读也会因为另一个原因“Preventing Lost Updates”失败。

### 实现Read Committed
通常用row级别锁来避免脏写。
要如何避免脏读，一个选项是用同一把锁。但这样会有性能问题。因为一个长时间的写事务会堵塞其他读事务。
大多数数据库采用的是，维护多个值，其他事务拿到的旧值，新的值被写事务更新，但直到最后提交，新的值才能被看到。

## Snapshot Isolation and Repeatable Read
当事务开启时，数据快照下来，事务内的查询只能看到快照中的数据。
以下场景需要这种隔离性
- 数据库备份，需要需要备份数据库当前状态的快照。
- 数据分析和完整性检查。

### 实现快照隔离
像read committed isolation一样，快照隔离也医用锁来避免脏读。然而读的时候不需要任何锁。从性能的角度看，读不堵塞任何写，写不堵塞任何读。这让数据库能在不用锁的情况下支持长读操作。
为了实现快照隔离，数据库采用一种技术称之为多版本并发控制（MVCC）。
如果数据库仅仅需要支持read commited隔离，那在事务期间只要保存对象的两个版本，一个旧版本，一个新版本的数据。但MVCC需要保持多个版本。
每个事务开启后，会产生一个唯一自增的事务ID(txid)。当一个事务往数据里面写时，被写的数据都会被打上一个（createBy=txid)的标签,被删除时打上一个（deleteBy=txid)的标签。事务期间只会读到快照时的数据。

### 可见性规则
1. 在事务开始时，数据库拿到此时正在执行的事务列表。任何由这些事务写入的数据都被忽略。
2. 任何被中断的事务被忽略
3. 任何在事务ID以后的数据被忽略
4. 其他写入对查询可见

### Indexes and snaphost isolation
1. 修改已有数据方案，索引指定到多个版本数据，当GC把旧版本被删除后，也一并删除索引中的引用。
2. 不修改已有数据方案，采用append-only B-trees,每个新版本数据都创建一个新的root节点。需要异步压缩和GC。

### Repeatable read and naming confusion
命名混淆，有些数据库中把快照隔离成为可重复读，行业并没有统一标准，所以没有知道重复读到底意味着什么。

## Preventing Lost Updates

