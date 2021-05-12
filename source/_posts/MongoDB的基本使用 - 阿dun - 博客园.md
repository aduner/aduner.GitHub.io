---
title: MongoDB的基本使用
date: 2020-08-16 21:17
categories: ['MongoDB']
tags: ['数据库', 'mongodb']
---
#  MongoDB的基本使用

##  连接实例

通过 mongo 命令来连接 MongoDB 实例：

    
    
    mongo [options] [db address] [file names]
    

默认端口号为 27017，安全模式未被开启，所以不需要输入用户名和密码即可直接连接：

    
    
    mongo 127.0.0.1:12345
    

或者通过 ` --host ` 和 ` --port ` 选项指定主机和端口。一切顺利的话，就进入了 MongoDB shell，shell
会报出一连串权限警告，不过不用担心，这并不会影响之后的操作。在添加授权用户和开启认证后，这些警告会自动消失。

##  增删改查

在进行增查改删操作之前，先了解下常用的 shell 操作：

  * ` db ` 显示当前所在数据库，默认为 test 
  * ` show dbs ` 列出可用数据库 
  * ` show tables ` ` show collections ` 列出数据库中可用集合 
  * ` use <database> ` 用于切换数据库 

MongoDB 预设有两个数据库，admin 和 local，admin 用来存放系统数据，local 用来存放该实例数据，在副本集中，一个实例的
local 数据库对于其它实例是不可见的。使用 use 命令切换数据库：

    
    
    > use admin
    > use local
    > use newDatabase
    

可以 use 一个不存在的数据库，当你存入新数据时，MongoDB 会创建这个数据库：

    
    
    > use newDatabase
    > db.newCollection.insert({x:1})
    WriteResult({ "nInserted" : 1 })
    

以上命令向数据库中插入一个文档，返回 1 表示插入成功，MongoDB 自动创建 newCollection 集合和数据库
newDatabase。下面将创建一个 drivers 集合，进行增查改删操作。

###  创建（Create）

MongoDB 提供 **insert** 方法创建新文档：

  * ` db.collection.inserOne() ` 插入单个文档 

> WriteResult({ "nInserted" : 1 })

  * ` db.collection.inserMany() ` 插入多个文档 

  * ` db.collection.insert() ` 插入单条或多条文档 

这里以 insert 方法为例：

    
    
    > db.drivers.insert({name:"Chen1fa",age:18})
    > db.drivers.insert({name:"Xiaose",age:35})
    > db.drivers.insert({_id:91,name:"Sun1feng",age:34})
    

要注意， ` age:18 ` 和 ` age:"18" ` 是不一样的，前者插入的是数值，后者插入的是字符串。插入新文档如果未指定 _id，MongoDB
会自动为插入的文档添加 _id 字段。使用 ` db.dirvers.find() ` 命令即可看到刚刚插入的文档：

    
    
    > db.dirvers.find()
    { "_id" : ObjectId("598964bd56b8c69ae1e5f36a"), "name" : "Chen1fa", "age" : 18 }
    { "_id" : ObjectId("598964d456b8c69ae1e5f36b"), "name" : "Xiaose", "age" : 35 }
    { "_id" : 91, "name" : "Sun1feng", "age" : 34 }
    

###  查找（Read）

MongoDB 提供 **find** 方法查找文档，第一个参数为查询条件：

    
    
    > db.drivers.find() #查找所有文档
    { "_id" : ObjectId("598964bd56b8c69ae1e5f36a"), "name" : "Chen1fa", "age" : 18 }
    { "_id" : ObjectId("598964d456b8c69ae1e5f36b"), "name" : "Xiaose", "age" : 35 }
    { "_id" : 91, "name" : "Sun1feng", "age" : 34 }
    
    > db.drivers.find({name: "Xiaose"}) #查找 name 为 Xiaose 的文档
    { "_id" : ObjectId("598964d456b8c69ae1e5f36b"), "name" : "Xiaose", "age" : 35 }
    
    > db.drivers.find({age:{$gt:20}}) #查找 age 大于 20 的文档
    { "_id" : ObjectId("598964d456b8c69ae1e5f36b"), "name" : "Xiaose", "age" : 35 }
    { "_id" : 91, "name" : "Sun1feng", "age" : 34 }
    

上述代码中的 ` $gt ` 对应于大于号 ` > `
的转义。第二个参数可以传入投影（projection，投影仪中，白色光源通过分光镜、液晶板、投影镜头的光学变换后，投射到反射面上，显示出了彩色图像）文档映射数据：

    
    
    > db.drivers.find({age:{$gt:20}},{name:1})
    { "_id" : ObjectId("598964d456b8c69ae1e5f36b"), "name" : "Xiaose" }
    { "_id" : 91, "name" : "Sun1feng" }
    

上述命令将查找 age 大于 20 的文档，返回 name 字段，排除其它字段。投影文档中字段为 1 或真值表示包含，0 或假值表示排除，可以设置多个字段为
1 或 0，但不能混合使用。

除此之外，还可以通过 count、skip、limit 等指针（Cursor）方法，改变文档查询的执行方式：

    
    
    > db.drivers.find().count() #统计查询文档数目
    3
    > db.drivers.find().skip(1).limit(10).sort({age:1})
    { "_id" : 91, "name" : "Sun1feng", "age" : 34 }
    { "_id" : ObjectId("598964d456b8c69ae1e5f36b"), "name" : "Xiaose", "age" : 35 }
    

上述查找命令跳过 1 个文档，限制输出 10 个，以 name 子段正序排序（大于 0 为正序，小于 0 位反序）输出结果。最后，可以使用 Cursor
方法中的 pretty 方法，提升查询文档的易读性，特别是在查看嵌套的文档和配置文件的时候：

    
    
    > db.drivers.find().pretty()
    

###  更新（Update)

MongoDB 提供 **updata** 方法更新文档：

  * ` db.collection.updateOne() ` 更新最多一个符合条件的文档 
  * ` db.collection.updateMany() ` 更新所有符合条件的文档 
  * ` db.collection.replaceOne() ` 替代最多一个符合条件的文档 
  * ` db.collection.update() ` 默认更新一个文档，可配置 multi 参数，跟新多个文档 

以 update() 方法为例。其格式：

    
    
    > db.collection.update(
        <query>,
        <update>,
        {
            upsert: <boolean>,
            multi: <boolean>
        }
    )
    

各参数意义：

  * _query_ 为查询条件 
  * _update_ 为修改的文档 
  * _upsert_ 为真，查询为空时插入文档 
  * _multi_ 为真，更新所有符合条件的文档 

下面的命令将 name 字段为 Chen1fa 的文档，更新 age 字段为 30：

    
    
    > db.drivers.update({name:"Chen1fa"},{name:"Chen1fa", age:30})
    

要注意的是，如果更新文档只传入 age 字段，那么文档会被更新为 ` {age: 30} ` ，而不是 ` {name:"Chen1fa", age:30}
` 。要避免文档被覆盖，需要用到 $set 指令，$set 仅替换或添加指定字段：

    
    
    > db.drivers.update({name:"Chen1fa"},{$set:{age:30}})
    

如果要在查询的文档不存在的时候插入文档，要把 upsert 参数设置真值：

    
    
    > db.drivers.update({name:"Alen"},{age:24},{upsert: true})
    

update 方法默认情况只更新一个文档，如果要更新符合条件的所有文档，要把 multi 设为真值，并使用 $set 指令：

    
    
    > db.drivers.update({age:{$gt:25}},{$set:{license:'A'}},{multi: true})
    > db.drivers.update({age:{$lt:25}},{$set:{license:'C'}},{multi: true})
    

最终结果：

    
    
    > db.dirvers.find()
    { "_id" : ObjectId("598964bd56b8c69ae1e5f36a"), "name" : "Chen1fa", "age" : 30, "license" : "A" }
    { "_id" : ObjectId("598964d456b8c69ae1e5f36b"), "name" : "Xiaose", "age" : 35, "license" : "A" }
    { "_id" : 91, "name" : "Sun1feng", "age" : 34, "license" : "A" }
    { "_id" : ObjectId("598968b3ed1eccef17e79abe"), "age" : 24, "license" : "C" }
    

###  删除（Delete）

MongoDB 提供了 **delete** 方法删除文档：

  * ` db.collection.deleteOne() ` 删除最多一个符合条件的文档 
  * ` db.collection.deleteMany() ` 删除所有符合条件的文档 
  * ` db.collection.remove() ` 删除一个或多个文档 

以 remove 方法为例：

    
    
    > db.drivers.remove({name:"Xiaose"}) #删除所有 name 为 Xiaose 的文档
    > db.drivers.remove({age:{$gt:30}},{justOne:true}) #删除所有 age 大于 30 的文档
    > db.drivers.find()
    { "_id" : ObjectId("598964bd56b8c69ae1e5f36a"), "name" : "Chen1fa", "age" : 30, "license" : "A" }
    { "_id" : ObjectId("598968b3ed1eccef17e79abe"), "age" : 24, "license" : "C" }
    

MongoDB 提供了 drop 方法删除集合，返回 true 表面删除集合成功：

    
    
    > db.drivers.drop()
    

