---
title: MongoDB 知识点梳理（一）
date: 2018-10-12T21:53:54+08:00
categories: ["Database"]
tags : ["Database", "MongoDB"]
toc: true
# featured_image : ""
keywords: ["MongoDB", "Database"]
description : "MongoDB知识点梳理(一)"
summary: "MongoDB是一个基于分布式文件存储的开源数据库系统。在高负载的情况下，添加更多的节点，可以保证服务器性能。
MongoDB 旨在为WEB应用提供可扩展的高性能数据存储解决方案。
MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。"
---

### MongoDB介绍
MongoDB是一个基于分布式文件存储的开源数据库系统。在高负载的情况下，添加更多的节点，可以保证服务器性能。
MongoDB 旨在为WEB应用提供可扩展的高性能数据存储解决方案。
MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。


### MongoDB 基本概念
#### 数据库
MongoDB的单个实例可以容纳多个独立的数据库，每一个都有自己的集合和权限，不同的数据库也放置在不同的文件中。

"show dbs" 命令可以显示所有数据的列表。

```bash
$ ./mongo
MongoDB shell version: 3.0.6
connecting to: test
> show dbs
admin  0.000GB
local  0.078GB
test   0.078GB
>
```

执行 "db" 命令可以显示当前数据库对象或集合。

```bash
$ ./mongo
MongoDB shell version: 3.0.6
connecting to: test
> db
test
>
```
运行"use"命令，可以连接到一个指定的数据库。

```bash
> use local
switched to db local
> db
local
>
```
数据库名可以是满足以下条件的任意UTF-8字符串。

- 不能是空字符串（"")。
- 不得含有' '（空格)、.、$、/、\和\0 (空字符)。
- 应全部小写。
- 最多64字节。
- 有一些数据库名是保留的，可以直接访问这些有特殊作用的数据库。

**admin**： 从权限的角度来看，这是"root"数据库。要是将一个用户添加到这个数据库，这个用户自动继承所有数据库的权限。一些特定的服务器端命令也只能从这个数据库运行，比如列出所有的数据库或者关闭服务器。
**local**: 这个数据永远不会被复制，可以用来存储限于本地单台服务器的任意集合
config: 当Mongo用于分片设置时，config数据库在内部使用，用于保存分片的相关信息。

#### 文档
文档是一组键值(key-value)对(即BSON)。MongoDB 的文档不需要设置相同的字段，并且相同的字段不需要相同的数据类型，这与关系型数据库有很大的区别，也是 MongoDB 非常突出的特点。

#### RDBMS（关系型数据库）与MongoDB对应术语
| RDBMS | MongoDB |
| :------:|:------:|
|数据库	| 数据库|
|表格	|集合|
|行	|文档|
|列	|字段|
|表联合	|嵌入文档|
|主键	|主键 (MongoDB 提供了 key 为 _id )|

需要注意的是：

- 文档中的键/值对是有序的。
- 文档中的值不仅可以是在双引号里面的字符串，还可以是其他几种数据类型（甚- 至可以是整个嵌入的文档)。
- MongoDB区分类型和大小写。
- MongoDB的文档不能有重复的键。
- 文档的键是字符串。除了少数例外情况，键可以使用任意UTF-8字符。

#### 文档键命名规范：

- 键不能含有\0 (空字符)。这个字符用来表示键的结尾。
- .和$有特别的意义，只有在特定环境下才能使用。
- 以下划线"_"开头的键是保留的(不是严格要求的)。

#### 集合
集合就是 MongoDB 文档组，类似于 RDBMS （关系数据库管理系统：Relational Database Management System)中的表格。
集合存在于数据库中，集合没有固定的结构，这意味着你在对集合可以插入不同格式和类型的数据，但通常情况下我们插入集合的数据都会有一定的关联性。

#### MongoDB 数据类型

| 数据类型	| 描述|
| :------:|:------:|
| String	| 字符串。存储数据常用的数据类型。在 MongoDB 中，UTF-8 编码的字符串才是合法的。|
| Integer	| 整型数值。用于存储数值。根据你所采用的服务器，可分为 32 位或 64 位。|
| Boolean	| 布尔值。用于存储布尔值（真/假）。|
| Double	| 双精度浮点值。用于存储浮点值。|
| Min/Max keys	| 将一个值与 BSON（二进制的 JSON）元素的最低值和最高值相对比。|
| Array	| 用于将数组或列表或多个值存储为一个键。|
| Timestamp	| 时间戳。记录文档修改或添加的具体时间。|
| Object	| 用于内嵌文档。|
| Null	| 用于创建空值。|
| Symbol	| 符号。该数据类型基本上等同于字符串类型，但不同的是，它一般用于采用特殊符号类型的语言。|
| Date	| 日期时间。用 UNIX 时间格式来存储当前日期或时间。你可以指定自己的日期时间：创建 Date 对象，传入年月日信息。|
| Object ID	| 对象 ID。用于创建文档的 ID。|
| Binary Data	| 二进制数据。用于存储二进制数据。|
| Code | 代码类型。用于在文档中存储 JavaScript 代码。|
| Regular expression	| 正则表达式类型。用于存储正则表达式。|

#### ObjectId
ObjectId 类似唯一主键，可以很快的去生成和排序，包含 12 bytes，含义是：

前 4 个字节表示创建 unix 时间戳,格林尼治时间 UTC 时间，比北京时间晚了 8 个小时
接下来的 3 个字节是机器标识码
紧接的两个字节由进程 id 组成 PID
最后三个字节是随机数

MongoDB 中存储的文档必须有一个 _id 键。这个键的值可以是任何类型的，默认是个 ObjectId 对象

由于 ObjectId 中保存了创建的时间戳，所以你不需要为你的文档保存时间戳字段，你可以通过 getTimestamp 函数来获取文档的创建时间:

```bash
> ObjectId().getTimestamp()
ISODate("2018-10-12T14:07:33Z")
>
```

### 数据库操作

#### 创建数据库

```
use database_name     // database_name代表数据库的名字
```

如果数据库不存在，则创建数据库，否则切换到指定数据库。


#### 查看数据库

##### 查看所有数据库
```js
show dbs
```

#### 查看当前数据库
```js
db
// 或者
db.getName()
```

#### 删除数据库
```js
db.dropDatabase()
```

#### 创建集合
##### 创建一个空集合
```js
db.createCollection(collection_Name)
//collection_Name集合的名称
```
##### 创建集合并插入一个文档
```js
//collection_Name集合的名称
//document要插入的文档
db.collection_Name.insert(document)
```

#### 插入文档
##### insert

``` js
db.collection_name.insert(document)
```
**参数**
- collection_name 集合的名字
- document 插入的文档

>每当插入一条新文档的时候mongodb会自动为此文档生成一个_id属性,_id一定是唯一的，用来唯一标识一个文档 _id也可以直接指定，但如果数据库中此集合下已经有此_id的话插入会失败
```js
> db.students.insert({_id:1,name:'luke',age:1});
WriteResult({ "nInserted" : 1 })
> db.students.insert({_id:1,name:'luke',age:1});
```
#### save

```js
db.collection_name.save(document)
```
**参数**

- collection_name 集合的名字
- document 插入的文档
>注：如果不指定 _id 字段 save() 方法类似于 insert() 方法。如果指定 _id 字段，则会更新该 _id 的数据。

实例
```js
> db.students.save({_id:1,name:'luke',age:1});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 0 })
> db.students.save({_id:1,name:'luke',age:100});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```

#### 更新文档
```
db.collection.update(
   <query>,
   <updateObj>,
   {
     upsert: <boolean>,
     multi: <boolean>
   }
)
```
**参数说明**

- query 查询条件,指定要更新符合哪些条件的文档
- updateObj 更新后的对象或指定一些更新的操作符
   - $set 直接指定更新后的值
   - $inc 在原基础上累加
   - $unset 删除指定的键
   - $push 向数组中添加元素
- upsert 可选，这个参数的意思是，如果不存在符合条件的记录时是否插入updateObj. 默认是false,不插入。
- multi 可选，mongodb 默认只更新找到的第一条记录，如果这个参数为true,就更新所有符合条件的记录。
实例 将students集合中数据中name是luke2的值修改为luke22
```js
> db.students.update({name:'luke2'},{age:300});
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
```
注：如果有多条name是luke2的数据只更新一条,如果想全部更新需要指定{multi:true}的参数
```js
db.students.update({name:'luke2'},{$set:{age:300}},{multi:true});
WriteResult({ "nMatched" : 2, "nUpserted" : 0, "nModified" : 1 })
```

### runCommand
```js
 var modify = {
    findAndModify:'student',
    query:{name:'张三'},
    update:{$set:{age:100}},
    fields:{name:'1'},
    sort:true,
    new:true //返回修改后的值
}

var result = db.runCommand(modify);
printjson(result);
```

#### 文档的删除
**remove**方法是用来移除集合中的数据
```js
db.collection.remove(
   <query>,
   {
     justOne: <boolean>
   }
)
```
参数说明

- query :（可选）删除的文档的条件。
- justOne : （可选）如果设为 true 或 1，则只删除匹配到的多个文档中的第一个

实例 删除worker集合里name是luke2的所有文档数据
```js
> db.students.remove({name:'luke2'});
WriteResult({ "nRemoved" : 2 })
```
即使匹配多条也只删除一条
```js
> db.students.remove({name:"luke2"},{justOne:true})
WriteResult({ "nRemoved" : 1 })
```

#### 查询文档
#### find
```js
db.collection_name.find()
```
**参数**
- collection_name 集合的名字

查询students下所有的文档
```js
db.students.find()
```

#### 查询指定列

```js
db.collection_name.find({queryWhere},{key:1,key:1})
```
**参数**
- collection_name 集合的名字
- queryWhere 参阅查询条件操作符
- key 指定要返回的列
- 1 表示要显示

只返回显示age列
```js
> db.students.find({},{age:1});
```

#### findOne
查询匹配结果的第一条数据 语法
```js
db.collection_name.findOne()
```
实例
```js
db.students.findOne()
```
#### $in
```js
db.student.find({age:{$in:[30,100]}},{name:1,age:1});
```
#### $nin
```js
db.student.find({age:{$nin:[30,100]}},{name:1,age:1});
```
#### $nin
```js
db.student.find({age:{$in:[30,100]}},{name:1,age:1});
```
#### $not
```js
db.student.find({age:{$not:{$gte:20,$lte:30}}});
```
#### array
```js
//按所有元素匹配
let result = db.student.find({friends:[ "Lufy", "Nami2", "Wusopu", "A" ]});
//匹配一条
let result = db.student.find({friends:"A"});
//$all
let result = db.student.find({friends:{$all:['A',"Lufy"]}});
//$in
let result = db.student.find({friends:{$in:['A',"Lufy"]}});
//$size
let result = db.student.find({friends:{$size:4}});
//$slice
let result = db.student.find({friends:{$size:5}},{name:1,friends:{$slice:1}});
let result = db.student.find({friends:{$size:5}},{name:1,friends:{$slice:-1}});
```
#### where
```js
db.student.find({$where:"this.age>30"},{name:1,age:1});
```
#### cursor
```js
var result = db.student.find();

//while(result.hasNext()){
//    printjson(result.next());
//}

result.forEach(element=>printjson(element));
```

### 条件操作符
条件操作符用于比较两个表达式并从mongoDB集合中获取数据

#### 大于操作符

查询age 大于 30的数据
```js
db.students.find({age:{$gt:30}})
```
#### 大于等于操作符

查询age 3大于等于30 的数据
```js
db.students.find({age: {$gte: 30}})
```
#### 小于操作符
查询age 小于30的数据
```js
db.students.find({age: {$lt: 30}})
```

#### 小于等于操作符

查询age 小于等于30的数据
```js
db.students.find({age: {$lte: 30}})
```
#### 同时使用 `$gte`和`$lte`

实例 查询age 大于等于 30 并且 age 小于等于 50 的数据
```js
db.students.find({age: {$gte: 30, $lte: 50}})
```
#### 等于

查询age = 30的数据
```js
db.students.find({"age": 30})
```
#### 使用 _id进行查询

实例 查询_id是 562af23062d5a57609133974 数据
```js
> db.students.find({_id:ObjectId("5adb666ecd738e9771638985")});
{ "_id" : ObjectId("5adb666ecd738e9771638985"), "name" : "zzzz" }
```
#### 查询结果集的条数
```js
db.students.find().count()
```
#### 正则匹配

实例 查询name里包含luke的数据
```js
db.students.find({name:/luke/})
```
查询某个字段的值当中是否以另一个值开头
```js
db.students.find({name:/^luke/})
```

###与和或
#### and
find方法可以传入多个键(key)，每个键(key)以逗号隔开
```js
db.collection_name.find({key1:value1, key2:value2})
```
实例 查询name是luke并且age是1的数据
```js
db.students.find({name:'luke',age:1})
```
#### or
语法
```js
db.collection_name.find(
   {
      $or: [
   {key1: value1}, {key2:value2}
      ]
   }
)
```
实例 查询age = 30 或者 age = 50 的数据
```js
db.students.find({$or:[{age:30},{age:50}]})
```
#### and和or联用
```js
db.collection_name.find(
   {
     key1:value1,
     key2:value2,
     $or: [
   {key1: value1},
   {key2:value2}
     ]
   }
)
```
实例 查询 name是luke 并且 age是30 或者 age是 50 的数据
```js
db.students.find({name:'luke',$or:[{age:30},{age:50}]})
```

### 分页查询
#### limit
读取指定数量的数据记录
```js
db.collectoin_name.find().limit(number)
```
参数

collectoin_name集合
number读取的条数
实例 查询前3条数据
```js
db.students.find().limit(3)
```
#### skip
跳过指定数量的数据，skip方法同样接受一个数字参数作为跳过的记录条数 语法
```js
db.collectoin_name.find().skip(number)
```
参数

collectoin_name集合
number跳过的条数
实例 查询3条以后的数据
```js
db.students.find().skip(3)
```
#### skip+limit
通常用这种方式来实现分页功能 语法
```js
db.collectoin_name.find().skip(skipNum).limit(limitNum)
```
参数

collectoin_name 集合名称
skipNum 跳过的条数
limitNum 限制返回的条数
实例 查询在4-6之间的数据
```js
db.students.find().skip(3).limit(3);
```
#### sort排序
sort()方法可以通过参数指定排序的字段，并使用 1 和 -1 来指定排序的方式，其中 1 为升序排列，而-1是用于降序排列。 语法
```js
db.collectoin_name.find().sort({key:1})
db.collectoin_name.find().sort({key:-1})
```
参数

collectoin_name集合
key表示字段
实例 查询出并升序排序 {age:1} age表示按那个字段排序 1表示升序
```js
db.students.find().sort({age:1})
```
