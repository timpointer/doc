# 数据建模

## 文档结构
在MongoDB应用设计数据模型时，关键决定是解决文档结构和数据之间的关系。MongoDB允许嵌入式文档。

### 被嵌方式
嵌入式文档通过将相关数据存入一个文档中来捕捉数据关系。
何时应该使用：
- 实体之间有包含关系。一对对关系
- 实体之间有一对多关系。这种关系中，子文档通常和父文档视为一个整体。
通常嵌入方式，提供了更好的读性能。只需要一次查询操作。单文档下原子操作也成为了可能。

### 关联方式
关联方式通过链接或引用的方式表达关系。
何时该使用：
- 当嵌入方式导致数据重复，并不能提供高效的读性能优势。
- 表达复杂的多对多关系
- 对大型树结构建模



不同的关联方式展现了不同的依赖关系，谁拥有谁的信息，谁就依赖了谁。

## 写原子操作
MongoDB针对一个文档提供了原子操作。

4.0版本以后提供多文档事务支持。

### 操作因子与数据模型
#### 大数量的集合
```
{ log: "dev", ts: ..., info: ... }
{ log: "debug", ts: ..., info: ...}
```

可以聚合在一个文档，也可以每个日志类型放一个文档。

####　集合包含大量的小文档
每个文档可能只有一两个属性，但数量巨大。这时你可以通过某种逻辑把文档聚合起来。称为"rolling-up"。
聚合以后意味着查询时是拿到一组文档和更少的查询次数。对索引也有好处。

然后如果你每次只是拿一组数据里面其中一两条，这么做就划算了。另外如果小文档就是数据的天然模型的话，也不建议聚合它。

#### 为小文档优化存储
每个文档都有一些额外信息。但如果文档本身很小，这种额外信息就会变成负担。
有两个建议：
- 使用自定义ID,默认ID比较长，但要确保ID唯一
- 用更短的字段名，但这样会降低可读性，不推荐这么做。

# 数据模型示例和模式
## 一对一关系
### 嵌入文档模式
```
// patron document
{
   _id: "joe",
   name: "Joe Bookreader"
}

// address document
{
   patron_id: "joe", // reference to patron document
   street: "123 Fake Street",
   city: "Faketon",
   state: "MA",
   zip: "12345"
}
```
合并为
```
{
   _id: "joe",
   name: "Joe Bookreader",
   address: {
              street: "123 Fake Street",
              city: "Faketon",
              state: "MA",
              zip: "12345"
            }
}
```
### 子集合模式
如果一个文档很大，可以把一个文档拆分为摘要和详情两个文档。
```
{
  "_id": 1,
  "title": "The Arrival of a Train",
  "year": 1896,
  "runtime": 1,
  "released": ISODate("01-25-1896"),
  "poster": "http://ia.media-imdb.com/images/M/MV5BMjEyNDk5MDYzOV5BMl5BanBnXkFtZTgwNjIxMTEwMzE@._V1_SX300.jpg",
  "plot": "A group of people are standing in a straight line along the platform of a railway station, waiting for a train, which is seen coming at some distance. When the train stops at the platform, ...",
  "fullplot": "A group of people are standing in a straight line along the platform of a railway station, waiting for a train, which is seen coming at some distance. When the train stops at the platform, the line dissolves. The doors of the railway-cars open, and people on the platform help passengers to get off.",
  "lastupdated": ISODate("2015-08-15T10:06:53"),
  "type": "movie",
  "directors": [ "Auguste Lumière", "Louis Lumière" ],
  "imdb": {
    "rating": 7.3,
    "votes": 5043,
    "id": 12
  },
  "countries": [ "France" ],
  "genres": [ "Documentary", "Short" ],
  "tomatoes": {
    "viewer": {
      "rating": 3.7,
      "numReviews": 59
    },
    "lastUpdated": ISODate("2020-01-09T00:02:53")
  }
}
```
拆分为movies和movie_details
```
// movie collection

{
  "_id": 1,
  "title": "The Arrival of a Train",
  "year": 1896,
  "runtime": 1,
  "released": ISODate("1896-01-25"),
  "type": "movie",
  "directors": [ "Auguste Lumière", "Louis Lumière" ],
  "countries": [ "France" ],
  "genres": [ "Documentary", "Short" ],
}
// movie_details collection

{
  "_id": 156,
  "movie_id": 1, // reference to the movie collection
  "poster": "http://ia.media-imdb.com/images/M/MV5BMjEyNDk5MDYzOV5BMl5BanBnXkFtZTgwNjIxMTEwMzE@._V1_SX300.jpg",
  "plot": "A group of people are standing in a straight line along the platform of a railway station, waiting for a train, which is seen coming at some distance. When the train stops at the platform, ...",
  "fullplot": "A group of people are standing in a straight line along the platform of a railway station, waiting for a train, which is seen coming at some distance. When the train stops at the platform, the line dissolves. The doors of the railway-cars open, and people on the platform help passengers to get off.",
  "lastupdated": ISODate("2015-08-15T10:06:53"),
  "imdb": {
    "rating": 7.3,
    "votes": 5043,
    "id": 12
  },
  "tomatoes": {
    "viewer": {
      "rating": 3.7,
      "numReviews": 59
    },
    "lastUpdated": ISODate("2020-01-29T00:02:53")
  }
}
```
#### 子集合模式的权衡
使用小文档来包含更频繁的数据请求可以减少整体热数据集。这些小文档查询更快也占用更少内存。
然而，重点是要明白你应用的数据加载方式。如果你错误的拆分了文档，你的应用可能需要多次请求并合并数据来完成数据的查询。
另外，拆分小文档也会增加数据库的维护成本，因为更难追踪数据在哪个文档里面了。

## 一对多
### 嵌入式文档模式
```
// patron document
{
   _id: "joe",
   name: "Joe Bookreader"
}

// address documents
{
   patron_id: "joe", // reference to patron document
   street: "123 Fake Street",
   city: "Faketon",
   state: "MA",
   zip: "12345"
}

{
   patron_id: "joe",
   street: "1 Some Other Street",
   city: "Boston",
   state: "MA",
   zip: "12345"
}
```
合并为
```
{
   "_id": "joe",
   "name": "Joe Bookreader",
   "addresses": [
                {
                  "street": "123 Fake Street",
                  "city": "Faketon",
                  "state": "MA",
                  "zip": "12345"
                },
                {
                  "street": "1 Some Other Street",
                  "city": "Boston",
                  "state": "MA",
                  "zip": "12345"
                }
              ]
 }
```
### 子集模式
举例一个电商网站，一个产品文档可以包含所有评论
```
{
  "_id": 1,
  "name": "Super Widget",
  "description": "This is the most useful item in your toolbox.",
  "price": { "value": NumberDecimal("119.99"), "currency": "USD" },
  "reviews": [
    {
      "review_id": 786,
      "review_author": "Kristina",
      "review_text": "This is indeed an amazing widget.",
      "published_date": ISODate("2019-02-18")
    },
    {
      "review_id": 785,
      "review_author": "Trina",
      "review_text": "Nice product. Slow shipping.",
      "published_date": ISODate("2019-02-17")
    },
    ...
    {
      "review_id": 1,
      "review_author": "Hans",
      "review_text": "Meh, it's okay.",
      "published_date": ISODate("2017-12-06")
    }
  ]
}
```
拆分成product文档包含10条最新评论和review文档包含所有评论
```
{
  "review_id": 786,
  "product_id": 1,
  "review_author": "Kristina",
  "review_text": "This is indeed an amazing widget.",
  "published_date": ISODate("2019-02-18")
}
{
  "review_id": 785,
  "product_id": 1,
  "review_author": "Trina",
  "review_text": "Nice product. Slow shipping.",
  "published_date": ISODate("2019-02-17")
}
...
{
  "review_id": 1,
  "product_id": 1,
  "review_author": "Hans",
  "review_text": "Meh, it's okay.",
  "published_date": ISODate("2017-12-06")
}
```
可以把10条最新的评论看做是一种缓存，要结合应用实际的查询数据场景来拆分。

#### 子集模式权衡
参照1对1子集模式

## 一对多通过关联建模
嵌入文档模式会导致数据重复
```
{
   title: "MongoDB: The Definitive Guide",
   author: [ "Kristina Chodorow", "Mike Dirolf" ],
   published_date: ISODate("2010-09-24"),
   pages: 216,
   language: "English",
   publisher: {
              name: "O'Reilly Media",
              founded: 1980,
              location: "CA"
            }
}

{
   title: "50 Tips and Tricks for MongoDB Developer",
   author: "Kristina Chodorow",
   published_date: ISODate("2011-05-06"),
   pages: 68,
   language: "English",
   publisher: {
              name: "O'Reilly Media",
              founded: 1980,
              location: "CA"
            }
}
```
为了避免publisher数据重复，采用引用数组。前提是引用的对象数量集很小，而且是有限的。
```
{
   name: "O'Reilly Media",
   founded: 1980,
   location: "CA",
   books: [123456789, 234567890, ...]
}

{
    _id: 123456789,
    title: "MongoDB: The Definitive Guide",
    author: [ "Kristina Chodorow", "Mike Dirolf" ],
    published_date: ISODate("2010-09-24"),
    pages: 216,
    language: "English"
}

{
   _id: 234567890,
   title: "50 Tips and Tricks for MongoDB Developer",
   author: "Kristina Chodorow",
   published_date: ISODate("2011-05-06"),
   pages: 68,
   language: "English"
}
```
为了避免变动增长的引用集合，采用反向关联的方式
```
{
   _id: "oreilly",
   name: "O'Reilly Media",
   founded: 1980,
   location: "CA"
}

{
   _id: 123456789,
   title: "MongoDB: The Definitive Guide",
   author: [ "Kristina Chodorow", "Mike Dirolf" ],
   published_date: ISODate("2010-09-24"),
   pages: 216,
   language: "English",
   publisher_id: "oreilly"
}

{
   _id: 234567890,
   title: "50 Tips and Tricks for MongoDB Developer",
   author: "Kristina Chodorow",
   published_date: ISODate("2011-05-06"),
   pages: 68,
   language: "English",
   publisher_id: "oreilly"
}
```

# 树结构建模
## 用父关联对树结构建模
考虑层级目录的场景
Books
|-Programming
  |-Languages
  |-Databases
    |-MongoDB
    |-dbm

```
db.categories.insertMany( [
   { _id: "MongoDB", parent: "Databases" },
   { _id: "dbm", parent: "Databases" },
   { _id: "Databases", parent: "Programming" },
   { _id: "Languages", parent: "Programming" },
   { _id: "Programming", parent: "Books" },
   { _id: "Books", parent: null }
] )
```
- 查询父节点
```
db.categories.findOne( { _id: "MongoDB" } ).parent
```
- 为父节点建索引
```
db.categories.createIndex( { parent: 1 } )
```
- 直接查询某个父节点的所有子节点
```
db.categories.find( { parent: "Databases" } )
```
### 用子节点关联方式对树建模
```
db.categories.insertMany( [
   { _id: "MongoDB", children: [] },
   { _id: "dbm", children: [] },
   { _id: "Databases", children: [ "MongoDB", "dbm" ] },
   { _id: "Languages", children: [] },
   { _id: "Programming", children: [ "Databases", "Languages" ] },
   { _id: "Books", children: [ "Programming" ] }
] )
```
- 查询所有子节点
```
db.categories.findOne( { _id: "Databases" } ).children
```
- 为子节点建模
```
db.categories.createIndex( { children: 1 } )
```
- 查询一个子节点
```
db.categories.find( { children: "MongoDB" } )
```
子关联模式提供了一种合适的树存储方式，当没有子树上操作的必要时。当一个节点有多个父节点时，这种模式也提供了一种合适的方案来存储图。

### 采用祖先集合对树建模
```
db.categories.insertMany( [
  { _id: "MongoDB", ancestors: [ "Books", "Programming", "Databases" ], parent: "Databases" },
  { _id: "dbm", ancestors: [ "Books", "Programming", "Databases" ], parent: "Databases" },
  { _id: "Databases", ancestors: [ "Books", "Programming" ], parent: "Programming" },
  { _id: "Languages", ancestors: [ "Books", "Programming" ], parent: "Programming" },
  { _id: "Programming", ancestors: [ "Books" ], parent: "Books" },
  { _id: "Books", ancestors: [ ], parent: null }
] )
```

- 快速查询到所有祖先
```
db.categories.findOne( { _id: "MongoDB" } ).ancestors
```
- 为祖先创建索引
```
db.categories.createIndex( { ancestors: 1 } )
```
- 通过祖先查到所有子节点
```
db.categories.find( { ancestors: "Programming" } )
```
祖先数组模式提供了一个有效查询祖先和后代的解决方案。对于子集树这是一个很好的解决方案。
祖先数组模式比实例化路劲模式慢一点，但更加直观。

### 实例化路径模式
```
db.categories.insertMany( [
   { _id: "Books", path: null },
   { _id: "Programming", path: ",Books," },
   { _id: "Databases", path: ",Books,Programming," },
   { _id: "Languages", path: ",Books,Programming," },
   { _id: "MongoDB", path: ",Books,Programming,Databases," },
   { _id: "dbm", path: ",Books,Programming,Databases," }
] )
```

```
\\查找整个树
db.categories.find().sort( { path: 1 } )

\\用正则表达式查找后代
db.categories.find( { path: /,Programming,/ } )

\\查找以Books为起点的后代
db.categories.find( { path: /^,Books,/ } )

\\创建索引
db.categories.createIndex( { path: 1 } )
```
Path查询也准守最左原则

### NestedSet对树建模
模拟内存中的树结构
```
db.categories.insertMany( [
   { _id: "Books", parent: 0, left: 1, right: 12 },
   { _id: "Programming", parent: "Books", left: 2, right: 11 },
   { _id: "Languages", parent: "Programming", left: 3, right: 4 },
   { _id: "Databases", parent: "Programming", left: 5, right: 10 },
   { _id: "MongoDB", parent: "Databases", left: 6, right: 7 },
   { _id: "dbm", parent: "Databases", left: 8, right: 9 }
] )
```
 
```
//查询后代
var databaseCategory = db.categories.findOne( { _id: "Databases" } );
db.categories.find( { left: { $gt: databaseCategory.left }, right: { $lt: databaseCategory.right } } );
```

# 特殊场景建模

## 为原子操作建模
借书记录和库存需要原子操作
```
{
    _id: 123456789,
    title: "MongoDB: The Definitive Guide",
    author: [ "Kristina Chodorow", "Mike Dirolf" ],
    published_date: ISODate("2010-09-24"),
    pages: 216,
    language: "English",
    publisher_id: "oreilly",
    available: 3,
    checkout: [ { by: "joe", date: ISODate("2012-10-15") } ]
}
```
```
db.books.updateOne (
   { _id: 123456789, available: { $gt: 0 } },
   {
     $inc: { available: -1 },
     $push: { checkout: { by: "abc", date: new Date() } }
   }
)
```
返回结果
```
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
```

## 支持关键字查询建模
添加一个string数组，数据库为数组里面的每个key创建一个索引，如果key很多的话，会对写入有性能影响。

```
{ title : "Moby-Dick" ,
  author : "Herman Melville" ,
  published : 1851 ,
  ISBN : 0451526996 ,
  topics : [ "whaling" , "allegory" , "revenge" , "American" ,
    "novel" , "nautical" , "voyage" , "Cape Cod" ]
}
```
创建索引
```
db.volumes.createIndex( { topics: 1 } )
```
根据keyword查询
```
db.volumes.findOne( { topics : "voyage" }, { title: 1 } )
```

## 为Schema版本建模
随着应用演化，模型也会变更，通过增加Schema版本字段，来更好的应用变化。
应用中可以通过判断schema_version值，以不同的方式处理模型。
```
// users collection

{
    "_id": "<ObjectId>",
    "galactic_id": 123,
    "name": "Anakin Skywalker",
    "phone": "503-555-0000",
}
```
```
// users collection

{
    "_id": "<ObjectId>",
    "galactic_id": 123,
    "name": "Darth Vader",
    "contact_method": {
        "work": "503-555-0210",
        "home": "503-555-0220",
        "twitter": "@realdarthvader",
        "skype": "AlwaysWithYou"
    },
    "schema_version": "2"
}
```

## 对时间数据建模
一些时序性数据可以通过聚合的方式来建模
```
// temperatures collection

{
  "_id": 1,
  "sensor_id": 12345,
  "timestamp": ISODate("2019-01-31T10:00:00.000Z"),
  "temperature": 40
}
{
  "_id": 2,
  "sensor_id": 12345,
  "timestamp": ISODate("2019-01-31T10:01:00.000Z"),
  "temperature": 40
}
{
  "_id": 3,
  "sensor_id": 12345,
  "timestamp": ISODate("2019-01-31T10:02:00.000Z"),
  "temperature": 41
}
...
```

```
{
  "_id": 1,
  "sensor_id": 12345,
  "start_date": ISODate("2019-01-31T10:00:00.000Z"),
  "end_date": ISODate("2019-01-31T10:59:59.000Z"),
  "measurements": [
    {
      "timestamp": ISODate("2019-01-31T10:00:00.000Z"),
      "temperature": 40
    },
    {
      "timestamp": ISODate("2019-01-31T10:01:00.000Z"),
      "temperature": 40
    },
    ...
    {
      "timestamp": ISODate("2019-01-31T10:42:00.000Z"),
      "temperature": 42
    }
  ],
  "transaction_count": 42,
  "sum_temperature": 1783
}
```
这样减少索引的数据量，还有一些提前计算的数据被缓存了。

## 建模计算后的数据
如果有些查询需要计算大量数据，我们可以把结果持久化下来，通过触发式或轮训的方式来更新这个视图数据。
```
// screenings collection

{
    "theater": "Alger Cinema",
    "location": "Lakeview, OR",
    "movie_title": "Reservoir Dogs",
    "num_viewers": 344,
    "revenue": 3440
}
{
    "theater": "City Cinema",
    "location": "New York, NY",
    "movie_title": "Reservoir Dogs",
    "num_viewers": 1496,
    "revenue": 22440
}
{
    "theater": "Overland Park Cinema",
    "location": "Boise, ID",
    "movie_title": "Reservoir Dogs",
    "num_viewers": 760,
    "revenue": 7600
}
```

```
// movies collection

{
    "title": "Reservoir Dogs",
    "total_viewers": 2600,
    "total_revenue": 33480,
    ...
}
```

## 对货币建模
// TODO
