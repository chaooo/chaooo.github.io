---
title: MongoDB学习笔记(2)
date: 2016-07-30 8:20:16
tags: mongodb
categories: 数据库
---

## part2 CRUD操作(Creat,Read,Update,Delete)

### 一、基础：

1、document(文档)

MongoDB把所有数据存放在类似于JSON数据结构的文档内：
``` json
  { "item": "pencil", "qty": 500, "type": "no.2" }
```
<!-- more -->
2、collection(集合)

集合是一组相关的文档，MongoDB存储所有的文档在集合里,他们拥有一套共享的通用索引。
``` json
  { "item": "pencil", "qty": 500, "type": "no.1" }
  { "item": "pencil2", "qty": 550, "type": "no.2" }
  { "item": "pencil3", "qty": 800, "type": "no.3" }
```

3、database(数据库)

MongoDB的默认数据库为"db"，该数据库存储在data目录中。一个mongodb中可以建立多个数据库。

### 二、数据库操作：

连接及运行mongoDB
"`show dbs`"命令可以显示所有的数据的列表
"`db`"命令可以显示当前数据库对象或集合
"`use`"命令可以连接到一个指定的数据库
数据库也通过名字来标识。数据库名可以是满足以下条件的任意UTF-8字符串。
  1.不能是空字符串（"")。
  2.不得含有' '（空格)、.、$、/、\和\0 (空宇符)。
  3.应全部小写。
  4.最多64字节。

1、创建数据库：`use Database_Name`
``` bash
  use test  ##创建名为test的数据库
```
2、删除当前数据库：
``` bash
  db.dropDatabase()
```

### 三、文档操作（以 Collection_Name = col 为例）

#### 1、插入：
``` bash
  db.col.insert(Document)     ##插入一条或多组数据
  db.col.insertOne(Document)  ##插入一条数据
  db.col.insertMany(Document) ##插入多条数据
  ##例如：
      db.col.insertOne({ "item": "pencil", "type": "no.1" })
      db.col.insertMany([
      { "item": "dog", "type": "no.2" },
      { "item": "apple", "type": "no.3" },
      { "item": "orange", "type": "no.4" }
      ])
```

#### 2、删除：
``` bash
  db.col.remove({})                    ##删除所有数据
  db.col.remove(query <,options>)
      #  query: 查询条件(数据索引或名字)
      #  ptions:两个可选参数
      #      {justOne: <boolean>,     //默认false，删除所有匹配到的。
      #       writeConcern: <document>//抛出异常的级别。
      #      }
  db.col.deleteOne(query <,options>)   ##同上，无justOne参数，只删除第一条
  db.col.deleteMany(query <,options>)  ##同上，无justOne参数，只删除多条
```

### 3、更新：
``` bash
  db.col.update(query, update <,options>)
      #  query:  查询条件(数据索引或名字)
      #  update: 更新的内容，语法：{$set:query}
      #  options:三个可选参数
      #      {upsert: <boolean>,      //如果不存在update的记录，是否插入新数据，默认:false。
      #       multi: <boolean>,       //只更新找到的第一条记录，默认是false,如果为true,多条记录全部更新。
      #       writeConcern: <document>//#抛出异常的级别。
      #      }
  ##例如：
      db.col.update(
          {"type": "no.1"}, 
          {$set: {"item": "human"}}, 
          {upsert: true, multi: true}
          )
  db.col.updateOne()                    ##同上，无multi参数，只更新第一条
  db.col.updateMany()                   ##同上，无multi参数
  db.col.replaceOne()                   ##同updateOne
  db.col.save(document <,writeConcern>) ##通过传入的文档整个替换
```
##### insert 与 save的区别
如果插入的数据的_id相同,save将会更新该文档,而insert将会报错

##### update常用操作符
``` bash
  $set         ##当文档中包含该字段的时候,更新该字段,如果该文档中没有该字段,则为本文档添加一个字段.
  $unset       ##删除文档中的一个字段.
  $rename      ##重命名某个列
  $inc         ##增长某个列
  $setOnInsert ##当upsert为true时,并且发生了insert操作时,可以补充的字段
  $push        ##将一个数字存入一个数组,分为三种情况,如果该字段存在,则直接将数字存入数组.如果该字段不存在,创建字段并且将数字插入该数组.如果更新的字段不是数组,会报错的.
  $pushAll     ##将多个数值一次存入数组.上面的push只能一个一个的存入
  $addToSet    ##与$push功能相同将一个数字存入数组,不同的是如果数组中有这个数字,将不会插入,只会插入新的数据,同样也会有三种情况,与$push相同.
  $pop         ##删除数组最后一个元素
  $pull        ##删除数组中的指定的元素,如果删除的字段不是数组,会报错
  $pullAll     ##删除数组中的多个值,跟pushAll与push的关系类似.
```

### 4、查询
``` bash
  db.col.find({})          ##查询所有文档
  db.col.find().pretty()   ##以易读的方式来读取数据
  db.collection.find(query, projection)
      #  query：查询条件(数据索引或名字)
      #  projection：可选。指定返回的字段。
```

#### 4.1、深入查询表达式
``` bash
  db.col.find()##查询所有
  db.col.find({filed: value})                              ##等值查询
  db.col.find({filed: {$ne: value}})                       ##不等于 $ne
  db.col.find({filed: {$nin: [value1, value2, ...]}})      ##不能包含给定的值 $nin
  db.col.find({filed: {$all: [value1, value2, ...]}})      ##必须包含所有给定的值 $all
  db.col.find({filed: {$in: [value1, value2, ...]}})       ##只要包含一个或多个给定的值 $in
  db.col.find({filed: {$exists:1}})                        ##存在filed字段的
  db.col.find({filed: {$exists:0}})                        ##不存在filed字段的
  db.col.find({filed: {$mod:[3,1]}})                       ##模三余一，$mod(取模操作)
  db.col.find({$or: [{filed1: vulue1}, {filed2: vulue2}]}) ##或 $or
  db.col.find({$nor: [{filed1: vulue1}, {filed2: vulue2}]})##排除 $nor
  db.col.find({filed: {$size: 3}})                         ##返回值得数组是给定的长度(3) $size
  db.col.find({$where: function(){return ...}})            ##回调，隐式迭代，符合条件才返回
  db.col.find({$where: '...'}})                            ##同上
  db.col.find({age: {$lt: 5}}).limit(3)                    ##查询age的值小于5，限制3条
      #范围查询：
      #    $lt  （小于）
      #    $gt  （大于）
      #    $lte （小于等于）
      #    $gte （大于等于）
      #    limit（限制显示）
  db.col.find().skip(2).limit(3)                           ##跳过前两个文档查询后面三个
      #  skip(num):表示跳过前面num个文档
  db.col.find().sort({age: 1})                             ##查询后以age升序排列显示
      #  sort():排序，这里 1 代表升序, -1 代表降序.
  db.col.find({filed: /user.*/i})                          ##正则，查询filed以user开头不区分大小写（正则效率低）
  db.col.find({filed: {$type: 1}})                         ##查找filed为双精度的文档
      # 根据数据类型查询 $type
      #      |类型　　　　　　　　|编号|
      #      |双精度　　　　　　　|1 　|
      #      |字符串　　　　　　　|2 　|
      #      |对象　　　　　　　　|3   |
      #      |数组　　　　　　　　|4   |
      #      |二进制数据　　　　　|5   |
      #      |对象ID　　　　　　　|7   |
      #      |布尔值　　　　　　　|8   |
      #      |日期　　　　　　　　|9   |
      #      |空　　　　　　　　　|10  |
      #      |正则表达式　　　　　|11  |
      #      |JavaScript　　　　|13  |
      #      |符号　　　　　　　　|14  |
      #      |JavaScript(带范围)|15  |
      #      |32位整数　　　　　　|16  |
      #      |时间戳　　　　　　　|17  |
      #      |64位整数　　　　　　|18  |
      #      |最小键　　　　　　　|255 |
      #      |最大键　　　　　　　|127 |
```

#### 4.2、group分组查询
group做的聚合有些复杂。先选定分组所依据的键，此后MongoDB就会将集合依据选定键值的不同分成若干组。然后可以通过聚合每一组内的文档，产生一个结果文档。
``` bash
  group({
    key:{字段:1},
    initial:{变量:初始值},
    $reduce:function(doc,prev){函数代码}
  })
```
其中key下的字段代表,需要按哪个字段分组.
initial下的变量表示这一个分组中会使用的变量,并且给一个初始值.可以在后面的$reduce函数中使用.
$reduce的两个参数,分别代表当前的文档和上个文档执行完函数后的结果.

栗子：如下我们按年龄分组,同级不同年龄的用户的多少:
``` bash
  db.user.find()
      { "_id" : ObjectId("5198c286c686eb50e2c843b2"), "name" : "user0", "age" : 0 }
      { "_id" : ObjectId("5198c286c686eb50e2c843b3"), "name" : "user1", "age" : 1 }
      { "_id" : ObjectId("5198c286c686eb50e2c843b4"), "name" : "user2", "age" : 2 }
      { "_id" : ObjectId("5198c286c686eb50e2c843b5"), "name" : "user3", "age" : 1 }
      { "_id" : ObjectId("5198c286c686eb50e2c843b6"), "name" : "user4", "age" : 1 }
      { "_id" : ObjectId("5198c286c686eb50e2c843b7"), "name" : "user5", "age" : 2 }

  db.user.group({
      key:{age:1},
      initial:{count:0},
      $reduce:function(doc,prev){
          prev.count++
      }
  }); 
      [
          {"age": 0, "count": 1},
          {"age": 1, "count": 3},
          {"age": 2, "count": 2}
      ]

  db.user.group({
      key:{age:1},
      initial:{users:[]},
      reduce:function(doc,prev){
          prev.users.push(doc.name)
      }
  });
    [
        {"age": 0, "users": ["user0"]},
        {"age": 1, "users": ["user1", "user3", "user4"]},
        {"age": 2, "users": ["user2", "user5"]}
    ]
```

另外本函数还有两个可选参数 condition 和 finalize
condition就是分组的条件筛选类似mysql中的having
``` bash
  db.user.group({
      key:{age:1},
      initial:{users:[]},
      $reduce:function(doc,prev){
          prev.users.push(doc.name)
      },
      condition:{age:{$gt:0}}})
　##筛选出age大于0的:
  [
      {"age": 1, "users": ["user1", "user3", "user4"]},
      {"age": 2, "users": ["user2", "user5"]}
  ]
```

#### 4.3、count统计
``` bash
  db.goods.count()            ##统计该集合总数
  db.goods.count({cat_id: 3}) ##统计cat_id=3的总数
```

#### 4.4、distinct排重
``` bash
 db.user.find()
    { "_id" : ObjectId("5198c286c686eb50e2c843b2"), "name" : "user0", "age" : 0 }
    { "_id" : ObjectId("5198c286c686eb50e2c843b3"), "name" : "user1", "age" : 1 }
    { "_id" : ObjectId("5198c286c686eb50e2c843b4"), "name" : "user2", "age" : 2 }
    { "_id" : ObjectId("5198c286c686eb50e2c843b5"), "name" : "user3", "age" : 1 }
    { "_id" : ObjectId("5198c286c686eb50e2c843b6"), "name" : "user4", "age" : 1 }
    { "_id" : ObjectId("5198c286c686eb50e2c843b7"), "name" : "user5", "age" : 2 }

  db.user.distinct("age") ## 特殊,传入的参数直接是字符串,而不是对象;
      [0, 1, 2]
```
#### 4.5、子文档查询$elemMatch

elemMatch投影操作符将限制查询返回的数组字段的内容只包含匹配elemMatch条件的数组元素。
注意：
(1)数组中元素是内嵌文档。
(2)如果多个元素匹配$elemMatch条件，操作符返回数组中第一个匹配条件的元素。
假设集合school有如下数据：
``` bash
{
 _id: 1,
 zipcode: 63109,
 students: [
              { name: "john", school: 102, age: 10 },
              { name: "jess", school: 102, age: 11 },
              { name: "jeff", school: 108, age: 15 }
           ]
}
{
 _id: 2,
 zipcode: 63110,
 students: [
              { name: "ajax", school: 100, age: 7 },
              { name: "achilles", school: 100, age: 8 },
           ]
}
{
 _id: 3,
 zipcode: 63109,
 students: [
              { name: "ajax", school: 100, age: 7 },
              { name: "achilles", school: 100, age: 8 },
           ]
}
{
 _id: 4,
 zipcode: 63109,
 students: [
              { name: "barney", school: 102, age: 7 },
           ]
}
```
下面的操作将查询邮政编码键值是63109的所有文档。 $elemMatch操作符将返回 students数组中的第一个匹配条件（内嵌文档的school键且值为102）的元素。
``` bash
  db.school.find({zipcode: 63109 },{ students: { $elemMatch: { school: 102 } } } );

  {"_id": 1, "students": [{"name":"john", "school":102, "age":10}]}
  {"_id": 3}
  {"_id": 4, "students": [{"name":"barney", "school":102, "age":7}]}
```
查询结果说明：
`_id为1的文档`，students数组包含多个元素中存在school键且值为102的元素，$elemMatch只返回一个匹配条件的元素。
`_id为3的文档`，因为students数组中元素无法匹配$elemMatch条件，所以查询结果不包含"students"字段。

$elemMatch可以指定多个字段的限定条件，下面的操作将查询邮政编码键值是63109的所有文档。 $elemMatch操作符将返回 students数组中的第一个匹配条件（内嵌文档的school键且值为102且age键值大于10）的元素。
``` bash
db.school.find( { zipcode: 63109 },{ students: { $elemMatch: { school: 102, age: { $gt: 10} } } } );

  {"_id": 1, "students": [{"name":"jess", "school":102, "age":11}]}
  {"_id": 3}
  {"_id": 4}
```
