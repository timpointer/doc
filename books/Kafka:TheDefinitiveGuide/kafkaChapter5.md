
## Ｃluster Membership
kafka使用zookeeper来维护当前broker集群的成员。每个成员都有一个ID，可以手动设置和自动分配。每当成员启动后会向zookeeper注册自己，创建一个ephemeral node（零时节点，zookeeper会管理其生命周期，如果成员失联，则自动删除）。其他kafka组件会订阅/brokers/ids下的消息。
如果你启动另一个同名的broker则会报错。
但broker关闭后，其他数据结构中关联的broker ID不会被删除，只要新的broker用的一样的ID就可以接替原来位置，继续工作。

### The Controller
kafka brokers中有一个节点担任管理者(controller)的角色，除了承担平常的职责外，还负责每个分片的leader的选择。通过zookeeper来确保每个时刻只有一个管理者。
当管理者失联后，每个节点接受到事件后，可以申请成为新的管理者。系统为管理者分片一个自增的编号，大家可以根据这个编号来忽略来自前管理者的消息。
当集群发生变化时，管理者有责任指定分片和副本配置。


