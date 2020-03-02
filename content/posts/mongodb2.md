---
title: MongoDB 知识点梳理(二)
date: 2018-10-13T21:53:54+08:00
categories: ["Database"]
tags : ["Database", "MongoDB"]
toc: true
featured_image : ""
keywords: ["MongoDB", "Database"]
description : "MongoDB 知识点梳理（二）"
---

### MongoDB通过配置项启动数据库
#### 启动服务器
```bash
mongod --config mongo.conf
```
#### 启动客户端
```bash
mongo --port 50000
```

|参数	|含义|
| :------:|:------:|
| --dbpath	|指定数据库文件的目录|
| --port	|端口 默认是27017 28017|
| --fork	|以后台守护的方式进行启动|
| --logpath	|指定日志文件输出路径|
| --config	|指定一个配置文件|
| --auth	|以安全方式启动数据库，默认不验证|

#### mongo.conf
```bash
dbpath=/Users/demon/coding/mongo/data
logpath=/Users/demon/coding/mongo/log
port=50000
```

### 导入导出数据
这命令是保存成了文件格式

- mongoimport 导出数据
- mongoexport 导入数据

|参数	|含义|
|:---: | :---: | :---: |
|-h |[ --host ]	连接的数据库|
|--port	|端口号|
|-u	|用户名|
|-p	|密码|
|-d|	导出的数据库|
|-d|	导出的数据库|
|-c|	指定导出的集合|
|-o	|导出的文件存储路径|
|-q|	进行过滤|

**准备数据**
```js
use school;
var students = [];
for(var i=1;i<=10;i++){
    students.push({name:'luke'+i,age:i});
}
db.students.insert(students);
db.students.find();
```
**备份记录**
```js
mongoexport -h 127.0.0.1 --port 50000  -d school -c students -o stu.json
```
**删除记录**
```bash
> db.students.remove({});
WriteResult({ "nRemoved" : 10 })
```
**导入记录**
```js
mongoimport --port 50000 --db school --collection students --file
stu.json
```

### 备份与恢复
#### mongodump
在Mongodb中我们使用mongodump命令来备份MongoDB数据。该命令可以导出所有数据到指定目录中。
```js
mongodump -h dbhost -d dbname -o dbdirectory
```
- -h MongDB所在服务器地址，例如：127.0.0.1，当然也可以指定端口号：127.0.0.1:27017
- -d 需要备份的数据库实例，例如：test
- -o 备份的数据存放位置

```js
mongodump -h 127.0.0.1 --port 50000 -d school -o data.dmp
```
#### mongorestore
**mongodb**使用 **mongorestore** 命令来恢复备份的数据。

- --host MongoDB所在服务器地址
- --db -d 需要恢复的数据库实例
- 最后的一个参数，设置备份数据所在位置


```js
mongorestore -h 127.0.0.1 --port 50000 data.dmp
mongorestore -h 127.0.0.1 --port 50000 -d school data.bmp/school
```
>Mongodump可以backup整个数据库，而mongoexport要对每个collection进行操作，最主要的区别也是选择的标准是mongoexport输出的JSON比Mongodump的BSON可读性更高，进而可以直接对JSON文件进行操作然后还原数据（BSON转换JSON存在潜在兼容问题）。

### 锁定和解锁数据库
为了数据的完整性和一致性，导出前要先锁定写入，导出后再解锁。
```bash
> use admin;
switched to db admin
> db.runCommand({fsync:1,lock:1});
{
        "info" : "now locked against writes, use db.fsyncUnlock() to unlock",
        "seeAlso" : "http://dochub.mongodb.org/core/fsynccommand",
        "ok" : 1
}
> db.fsyncUnlock();
{ "ok" : 1, "info" : "unlock completed" }
```

### 用户管理
#### 查看角色

```js
show roles;
```

#### 内置角色

- 数据库用户角色：read、readWrite；
- 数据库管理角色：dbAdmin、dbOwner、userAdmin;
- 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManage；
- 备份恢复角色：backup、restore；
- 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
- 超级用户角色：root
- 内部角色：__system

#### 创建用户的方法
```bash
> db.createUser({user:'luke2',pwd:'123456',roles:[{role:'read',db:'school'}]});
Successfully added user: {
        "user" : "luke2",
        "roles" : [
                {
                        "role" : "read",
                        "db" : "school"
                }
        ]
}
```
#### 查看用户的权限
```js
> db.runCommand({usersInfo:'luke2',showPrivileges:true});
{
        "users" : [
                {
                        "_id" : "admin.luke2",
                        "user" : "luke2",
                        "db" : "admin",
                        "roles" : [
                                {
                                        "role" : "read",
                                        "db" : "school"
                                }
                ]
}
```

### 数据库高级命令
#### 准备数据
```js
var students = [
        {name:'luke1',home:'北京',age:1},
        {name:'luke2',home:'北京',age:2},
        {name:'luke3',home:'北京',age:3},
        {name:'luke4',home:'广东',age:1},
        {name:'luke5',home:'广东',age:2},
        {name:'luke6',home:'广东',age:3}
]
db.students.insert(students);
```
#### count
查看记录数
```js
db.students.find().count();
```
#### 查找不重复的值 distinct
```js
db.runCommand({distinct:'students',key:'home'}).values;

//[ "北京", "广东" ]
```
#### group 分组
```js
db.runCommand({
        group:{
                ns:集合名称，
                key:分组的键,
                initial:初始值,
                $reduce:分解器
                condition:条件,
                finalize:完成时的处理器
        }
});
```
#### 按城市分组，求每个城市里符合条件的人的年龄总和
```js
db.runCommand({
        group:{
                ns:'students',
                key:{home:true},
                initial:{total:0},
                $reduce:function(doc,result){
                      result.total += doc.age;
                },
                condition:{age:{$gt:1}},
                finalize:function(result){
                    result.desc = '本城市的总年龄为'+result.total;
                }
        }
});
```
#### 删除集合
```js
db.runCommand({drop:'students'});
```

### 固定集合
MongoDB 固定集合（Capped Collections）是性能出色且有着固定大小的集合，对于大小固定，我们可以想象其就像一个环形队列，当集合空间用完后，再插入的元素就会覆盖最初始的头部的元素！ firstinfirstout

#### 特性
- 没有索引
- 插入和查询速度速度非常快 不需要重新分配空间
- 特别适合存储日志
#### 创建固定集合
- 我们通过createCollection来创建一个固定集合，且capped选项设置为true：
- 还可以指定文档个数,加上max:1000属性：
- 判断集合是否为固定集合: db.logs.isCapped()
- size 是整个集合空间大小，单位为【KB】
- max 是集合文档个数上线，单位是【个】
- 如果空间大小到达上限，则插入下一个文档时，会覆盖第一个文档；如果文档个数到达上限，同样插入下一个文档时，会覆盖第一个文档。两个参数上限判断取的是【与】的逻辑。
- capped 封顶的
```js
 db.createCollection('logs',{size:50,max:5,capped:true});
 ```
#### 非固定集合转为固定集合
```js
db.runCommand({convertToCapped:"logs",size:5});
```

### 索引
- 索引通常能够极大的提高查询的效率，如果没有索引，MongoDB在读取数据时必须扫描集合中的每个文件并选取那些符合查询条件的记录。
- 这种扫描全集合的查询效率是非常低的，特别在处理大量的数据时，查询可以要花费几十秒甚至几分钟，这对网站的性能是非常致命的。
- 索引是特殊的数据结构，索引存储在一个易于遍历读取的数据集合中，索引是对数据库表中一列或多列的值进行排序的一种结构
- 使用索引，方便范围查询和匹配查询。

#### createIndex() 方法
MongoDB使用 createIndex() 方法来创建索引。

>注意在 3.0.0 版本前创建索引方法为 db.collection.ensureIndex()，之后的版本使用了 db.collection.createIndex() 方法，ensureIndex() 还能用，但只是 createIndex() 的别名。

**语法**
createIndex()方法基本语法格式如下所示：
```js
db.collection.createIndex(keys, options)
```
语法中 Key 值为你要创建的索引字段，1 为指定按升序创建索引，如果你想按降序来创建索引指定为 -1 即可。

**实例**
```js
db.col.createIndex({"title":1})
```
createIndex() 方法中你也可以设置使用多个字段创建索引（关系型数据库中称作复合索引）。

```js
db.col.createIndex({"title":1,"description":-1})
```

#### 指定使用的索引
```js
db.students.find({name:'zfpx299999',age:299999}).hint({name:1}).explain(true);
```
#### 创建唯一索引并删除重复记录
```js
db.person.ensureIndex({ "name" : -1 },{ "name" : "indexname", "unique" : true,dropDups:true })
```
#### 删除索引
```js
db.students.dropIndex('namedIndex');//删除指定的索引
db.students.dropIndex('*');
db.runCommand({dropIndexes:"students",index:"namedIndex"});//删除所有的索引
```
#### 在后台创建索引
```js
db.students.ensureIndex({name:1},{name:'nameIndex',unique:true,background:true});
```
#### 过期索引
在一定的时间后会过期，过期后相应数据数据被删除,比如 session、日志、缓存和临时文件
```js
db.stus.insert({time:new Date()});
db.stus.ensureIndex({time:1},{expireAfterSeconds:10});
db.stus.find();
```
1. 索引字段的值必须Date对象，不能是其它类型比如时间戳
2. 删除时间不精确，每60秒跑一次。删除也要时间，所以有误差。
#### 全文索引
大篇幅的文章中搜索关键词,MongoDB为我们提供了全文索引
```js
db.article.insert({content:'I am a gir'});
db.article.insert({content:'I am a boy'});
$text:表示要在全文索引中查东西
$search:后边跟查找的内容, 默认全部匹配
db.article.find({$text:{$search:'boy'}});
db.article.find({$text:{$search:'girl'}});
db.article.find({$text:{$search:'boy girl'}});//多次查找，多个关键字为或的关系
db.article.find({$text:{$search:"a b"}});
db.article.find({$text:{$search:"boy -girl"}}); // -表示取消
db.article.find({$text:{$search:"a \"coco cola\" b "}}); //支持转义符的,用\斜杠来转义
```

#### 索引使用的注意事项
- 1为正序 -1为倒序
- 索引虽然可以提升查询性能，但会降低插件性能，对于插入多查询少不要创索引
- 数据量不大时不需要使用索引。性能的提升并不明显，反而大大增加了内存和硬盘的消耗。
- 查询数据超过表数据量30%时，不要使用索引字段查询
- 排序工作的时候可以建立索引以提高排序速度
- 数字索引，要比字符串索引快的多

### MongoDB 复制（副本集）
**MongoDB**复制是将数据同步在多个服务器的过程。
复制提供了数据的冗余备份，并在多个服务器上存储数据副本，提高了数据的可用性， 并可以保证数据的安全性。
复制还允许您从硬件故障和服务中断中恢复数据。
#### MongoDB复制原理
mongodb的复制至少需要两个节点。其中一个是主节点，负责处理客户端请求，其余的都是从节点，负责复制主节点上的数据。
mongodb各个节点常见的搭配方式为：一主一从、一主多从。
主节点记录在其上的所有操作oplog，从节点定期轮询主节点获取这些操作，然后对自己的数据副本执行这些操作，从而保证从节点的数据与主节点一致。

#### 流程
1. 一台活跃服务器和二个备份服务器
2. 当活跃服务器出现故障，这时集群根据权重算法推选出出活跃服务器
3. 当原来的主服务器恢复后又会变成从服务器

#### 配置副本集
A服务器
```js
dbpath=E:\repl\repl1
port=2001
replSet=group
```
B服务器
```js
dbpath=E:\repl\repl2
port=2002
replSet=group
```
C服务器
```js
dbpath=E:\repl\repl3
port=2003
replSet=group
```
#### 初始化副本集

rs.initiate() 启动一个新的副本集
rs.conf() 查看副本集的配置
rs.status() 命令
```js
use admin;
var conf=
{
    "_id" : "group",
    "members" : [
        { "_id" : 0,  "host" : "127.0.0.1:2001"  },
        { "_id" : 1,  "host" : "127.0.0.1:2002"  },
        { "_id" : 2,  "host" : "127.0.0.1:2003"  }
    ]
}
rs.initiate(conf);
rs.status();
```
#### 高级参数
- standard 常规节点 参与投票有可能成为活跃节点
- passive 副本节点 参与投票，但不能成为活跃节点
- arbiter 仲裁节点 只参与投票，不复制节点，也不能成为活跃节点
- priority 0到1000之间，0代表是副本节点，1到1000是常规节点
- arbiterOnly:true 仲裁节点
#### 读写分离操作
一般情况下作为副本节点是不能进行数据库操作的，但是在读取密集的系统中读写分离是必要的
```js
 rs.slaveOk();
```
#### Oplog
它被存储在本地数据库local中，会记录每一个操作。 如果希望在故障恢复的时候尽可能更多，可以把这个size设置的大一点
```js
--oplogSize 1024
use local;
 db.oplog.rs.find().limit(2);
```
 MongoDB的副本集与我们常见的主从有所不同，主从在主机宕机后所有服务将停止，而副本集在主机宕机后，副本会接管主节点成为主节点，不会出现宕机的情况。

### MongoDB 分片
#### 分片
在Mongodb里面存在另一种集群，就是分片技术,可以满足MongoDB数据量大量增长的需求。

当MongoDB存储海量的数据时，一台机器可能不足以存储数据，也可能不足以提供可接受的读写吞吐量。这时，我们就可以通过在多台机器上分割数据，使得数据库系统能存储和处理更多的数据。

#### 为什么使用分片
- 复制所有的写入操作到主节点
- 延迟的敏感数据会在主节点查询
- 单个副本集限制在12个节点
- 当请求量巨大时会出现内存不足。
- 本地磁盘不足
- 垂直扩展价格昂贵

MongoDB分片
下图展示了在MongoDB中使用分片集群结构分布：

![](https://img-1256541035.cos.ap-shanghai.myqcloud.com/imgs/mongodb-sharding.png)


上图中主要有如下所述三个主要组件：

- **Shard**:
用于存储实际的数据块，实际生产环境中一个shard server角色可由几台机器组个一个replica set承担，防止主机单点故障

- **Config Server**:
mongod实例，存储了整个 ClusterMetadata，其中包括 chunk信息。

- **Query Routers**:
前端路由，客户端由此接入，且让整个集群看上去像单一数据库，前端应用可以透明使用。

#### 分片实例
分片结构端口分布如下：
```js
Shard Server 1：27020
Shard Server 2：27021
Shard Server 3：27022
Shard Server 4：27023
Config Server ：27100
Route Process：40000
```
步骤一：启动Shard Server
```js
[root@100 /]# mkdir -p /www/mongoDB/shard/s0
[root@100 /]# mkdir -p /www/mongoDB/shard/s1
[root@100 /]# mkdir -p /www/mongoDB/shard/s2
[root@100 /]# mkdir -p /www/mongoDB/shard/s3
[root@100 /]# mkdir -p /www/mongoDB/shard/log
[root@100 /]# /usr/local/mongoDB/bin/mongod --port 27020 --dbpath=/www/mongoDB/shard/s0 --logpath=/www/mongoDB/shard/log/s0.log --logappend --fork
....
[root@100 /]# /usr/local/mongoDB/bin/mongod --port 27023 --dbpath=/www/mongoDB/shard/s3 --logpath=/www/mongoDB/shard/log/s3.log --logappend --fork
```
步骤二： 启动Config Server
```js
[root@100 /]# mkdir -p /www/mongoDB/shard/config
[root@100 /]# /usr/local/mongoDB/bin/mongod --port 27100 --dbpath=/www/mongoDB/shard/config --logpath=/www/mongoDB/shard/log/config.log --logappend --fork
```
注意：这里我们完全可以像启动普通mongodb服务一样启动，不需要添加—shardsvr和configsvr参数。因为这两个参数的作用就是改变启动端口的，所以我们自行指定了端口就可以。

步骤三： 启动Route Process
```js
/usr/local/mongoDB/bin/mongos --port 40000 --configdb localhost:27100 --fork --logpath=/www/mongoDB/shard/log/route.log --chunkSize 500
```
mongos启动参数中，chunkSize这一项是用来指定chunk的大小的，单位是MB，默认大小为200MB.

步骤四： 配置Sharding

接下来，我们使用MongoDB Shell登录到mongos，添加Shard节点
```js
[root@100 shard]# /usr/local/mongoDB/bin/mongo admin --port 40000
MongoDB shell version: 2.0.7
connecting to: 127.0.0.1:40000/admin
mongos> db.runCommand({ addshard:"localhost:27020" })
{ "shardAdded" : "shard0000", "ok" : 1 }
......
mongos> db.runCommand({ addshard:"localhost:27029" })
{ "shardAdded" : "shard0009", "ok" : 1 }
mongos> db.runCommand({ enablesharding:"test" }) #设置分片存储的数据库
{ "ok" : 1 }
mongos> db.runCommand({ shardcollection: "test.log", key: { id:1,time:1}})
{ "collectionsharded" : "test.log", "ok" : 1 }
```
步骤五： 程序代码内无需太大更改，直接按照连接普通的mongo数据库那样，将数据库连接接入接口40000