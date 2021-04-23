

# Shard keys

## Shard Key规范
```
sh.shardCollection(<namespace>, <key>) // Optional parameters omitted
```
key值如果是１，代表根据范围分片，如果是"hashed",则用hash函数分片。

## 重定义Shard Key
4.4以后支持重定义，比如原来orders文档是用{customer_id:1} 来分片，现在可以改为使用{customer_id:1, order_id:1}来分片。
单个Shard key的值如果是热点数据，就会导致巨型客机（jumbo chuncks）问题。

## Shard Key 索引
所有文档都必须有一个索引来支持shard key，索引可以是和shard key一样，或是一个组合索引中前缀部分包含shard key。
如果没有索引，在分片时候会自动创建该索引。
如果文档都没有，那么你必须在分片前，先创建索引。

## 唯一索引
Mongodb支持在一个范围分片索引上加上唯一性限制。具体细节参考原文档。

## 选择一个Shard Key

三要素
- cardinality //　集合数量
- frequency
- mononicity

### Shard Key Cardinality
如果low cardinality，会使得水平扩展受限，因为一个唯一的shard key的值不能存在于两个chunk中，如果shard key只有两个值，那么数据最多能被分配到三个分片中。
此时可以采用组合索引增加cardinanlity的数量。

### Shard Key Frequency
如果某些Key的数据特别频繁，也会导致数据分片不均匀。
此时可以采用组合索引，加上一个唯一值或低频率的值。

### Shard Key Monotonicity
如果选择的key有单调递增或递减的特性，比如把时间作为key。
新的数据要么都会加到最小值的那个分片，或者最大值的那个分片。
此时可以采用hash分片的方法。

## Missing Shard key
如果文档属性缺少shard key的部分，统一当做null值来处理。

# Hashed Sharding
要么使用单个属性hash索引或一个组合属性hash索引

