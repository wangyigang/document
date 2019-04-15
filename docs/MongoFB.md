

### Mongo DB

定义：一个基于分布式文件存储的开源数据库系统

MongoDB将数据存储为一个文档，数据结构由键值对(key-value)组成，MongoDB文档类似于JSon对象



存储更加灵活

不需要预先设定表结构

##### MongoDB特性

```
层级： Database-Collection-Document
灵活的类Json数据存储，每条文档的字段可以完全不同
方便的即席查询，索引和实时聚合
update()命令可以实现替换完成的文档,
```

##### MongoDB主要概念

| 关系型DB    | MongoDB     | 解释/说明              |
| ----------- | ----------- | ---------------------- |
| database    | database    | 数据库                 |
| table       | collection  | 数据库表/集合          |
| row         | document    | 数据记录行/文档        |
| column      | field       | 数据字段/域            |
| index       | index       | 索引                   |
| primary key | primary key | Mongo自动将_id设为主键 |

##### 创建MongoDB连接

```
标准URL
mongodb://username:password@host1:prot1/database[?options]
```

##### 数据库shell

```
显示当前数据库： db
查看所有数据库： show dbs
显示表： show tables
创建数据库： use XXx(数据库名)
```

##### 集合shell

```
新建集合：
db.createCollection(name,options)
ex:db.createCollection("tmpCollection")
删除集合
db.tmpCollection.drop()
```

##### 文档操作shell

```
插入文档:
document={"name"="Alice","age"=20}
db.COLLECTION.insert(document)
删除文档：
db.COLLECTION_NAME.remove(
<query>  //删除的文件条件
{
    justOne:<boolean>,  //若设为true或1，只删除一个文档
    writeConcern:<document>  //抛出异常的级别
})

db.COLLECTION_NAME.remove({})  //不给任何条件=>全部删除
```

###### 文档更新update

```
db.COLLECTION_NAME.update(
	<query>, //update的查询条件，
	<update>,{ //$set $inc
    upsert:<boolean>,
    multi:<boolean>,
    writeConcern:<document>
})
ex:db.mytable.update({"name":"IPHONE"},$set:{"color":"red"})
直接使用save()
db.collection_name.save(document) 
```

###### 文档查询

```
db.COLLECTION_NAME.find(query,projection)
-- query:可选，使用查询操作符指定查询条件
--projection: 可选，使用投影操作符指定返回的键
db.COLLECTION_NAME.find( {"name": "iPhone"}, {"name": 1, _id: 0} )
ex：db.mytable.find(age:{$gt:20})

and条件：以逗号隔开,
or条件： 使用关键字$or
排序：sort   1升序  -1:降序
索引：index createIndex()创建索引
	db.COLLECTION_NAME.createIndex(keys,options)
		options取1升序， -1位降序
```

