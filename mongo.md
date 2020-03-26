# mongo

## 简介

MongoDB 是一个跨平台的，面向文档的数据库，是当前 NoSQL 数据库产品中最热门的一种。它介于关系数据库和非关系数据库之间，是非关系数据库当中功能最丰富，最像关系数据库的产品。它支持的数据结构非常松散，是类似 JSON 的 BSON 格式，因此可以 存储比较复杂的数据类型。 MongoDB 的官方网站地址是：http://www.mongodb.org/

## 特点

MongoDB 最大的特点是他支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。它是一个面向集合的,模式自由的文档型数据库。具体特点总结如下： 

- 面向集合存储，易于存储对象类型的数据 
- 模式自由 
- 支持动态查询 
- 支持完全索引，包含内部对象 
- 支持复制和故障恢复 
- 使用高效的二进制数据存储，包括大型对象（如视频等） 
- 自动处理碎片，以支持云计算层次的扩展性 
- 支持 Python，PHP，Ruby，Java，C，C#，Javascript，Perl 及 C++语言的驱动程序，社区中也提供了对 Erlang 及.NET 等平台的驱动程序 
- 文件存储格式为 BSON（一种 JSON 的扩展） 

## 体系结构

MongoDB 的逻辑结构是一种层次结构。主要由： 
文档(document)、集合(collection)、数据库(database)这三部分组成的。逻辑结构是面向用户的，用户使用 MongoDB 开发应用程序使用的就是逻辑结构。 

- MongoDB 的文档（document），相当于关系数据库中的一行记录。 

- 多个文档组成一个集合（collection），相当于关系数据库的表。 

- 多个集合（collection），逻辑上组织在一起，就是数据库（database）。 

- 一个 MongoDB 实例支持多个数据库（database）。 

文档(document)、集合(collection)、数据库(database)的层次结构如下图：

![](D:\images\2020-2-4\20200218173531.jpg)

下表是MongoDB与MySQL数据库逻辑结构概念的对比

| 文档型数据库MongoDB | 关系型数据库MySQL |
| ------------------- | ----------------- |
| 数据库(databases)   | 数据库(databases) |
| 集合(collections)   | 表(table)         |
| 文档(document)      | 行(row)           |

## 数据类型

基本数据类型 

- null：用于表示空值或者不存在的字段，{“x”:null} 
- 布尔型：布尔类型有两个值true和false，{“x”:true} 
- 数值：shell默认使用64为浮点型数值。{“x”：3.14}或{“x”：3}。对于整型值，可以使用 
- NumberInt（4字节符号整数）或NumberLong（8字节符号整数）， 
- {“x”:NumberInt(“3”)}{“x”:NumberLong(“3”)} 
- 字符串：UTF-8字符串都可以表示为字符串类型的数据，{“x”：“呵呵”} 
- 日期：日期被存储为自新纪元依赖经过的毫秒数，不存储时区，{“x”:new Date()}
- 正则表达式：查询时，使用正则表达式作为限定条件，语法与JavaScript的正则表达式相 
- 同，{“x”:/[abc]/} 
- 数组：数据列表或数据集可以表示为数组，{“x”： [“a“，“b”,”c”]} 
- 内嵌文档：文档可以嵌套其他文档，被嵌套的文档作为值来处理，{“x”:{“y”:3 }} 
- 对象Id：对象id是一个12字节的字符串，是文档的唯一标识，{“x”: objectId() } 
- 二进制数据：二进制数据是一个任意字节的字符串。它不能直接在shell中使用。如果要 
- 将非utf-字符保存到数据库中，二进制数据是唯一的方式。 
- 代码：查询和文档中可以包括任何JavaScript代码，{“x”:function(){/*…*/}} 

## 安装

- 下载Mongo镜像

  ```dockerfile
  docker pull mongo
  ```

- 运行安装命令

  ```dockerfile
  docker run --name mongodb -p 27017:27017 -d mongo --auth
  ```

- 为MongoDB添加管理员用户

  ```dockerfile
  docker exec -it mongo容器ID mongo admin
  ```

- 创建一个 admin 管理员账号

  ```dockerfile
  db.createUser({ user: 'root', pwd: 'root', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });
  ```

- 退出

  ```dockerfile
  exit
  ```

- 操作记录

  ```dockerfile
  [root@localhost local]# docker exec -it mongo容器ID mongo admin
  MongoDB shell version v4.2.3
  connecting to: mongodb://127.0.0.1:27017/admin?compressors=disabled&gssapiServiceName=mongodb
  Implicit session: session { "id" : UUID("16471332-6e33-4001-9850-d3ea7ecb303f") }
  MongoDB server version: 4.2.3
  Welcome to the MongoDB shell.
  For interactive help, type "help".
  For more comprehensive documentation, see
          http://docs.mongodb.org/
  Questions? Try the support group
          http://groups.google.com/group/mongodb-user
  > db.createUser({ user: 'root', pwd: 'root', roles: [ { role: "userAdminAnyDatabase", db: "admin" } ] });
  Successfully added user: {
          "user" : "root",
          "roles" : [
                  {
                          "role" : "userAdminAnyDatabase",
                          "db" : "admin"
                  }
          ]
  }
  > exit
  bye
  
  ```

  - 以 admin 用户身份进入mongo

    ```cmd
    docker exec -it mongo容器ID mongo admin
    ```

  - 对 admin 进行身份认证：
    PS:就是上一步创建的管理员账号密码
    
    ```cmd
    db.auth("root","root");
    ```

  - 退出

    ```cmd
    exit
    ```

  - 操作记录

    ```cmd
    [root@localhost local]# docker exec -it 1f8e329fefff  mongo admin
    MongoDB shell version v4.2.3
    connecting to: mongodb://127.0.0.1:27017/admin?compressors=disabled&gssapiServiceName=mongodb
    Implicit session: session { "id" : UUID("8ddb2d4e-5343-4039-b616-55bc1c7e15d9") }
    MongoDB server version: 4.2.3
    > db.auth("root","root");
    1
    > exit
    bye
    ```

## 选择和创建数据库

- 选择和创建数据库的语法格式

  如果数据库不存在则自动创建 

```cmd
use spitdb
```

## 插入与查询文档

- 插入文档的语法格式

```cmd
db.spit.insert({id:1,age:NumberInt(20),name:"baerwang"})
```

- 查询集合的语法格式

```cmd
db.spit.find()
```

这里你会发现每条文档会有一个叫_id的字段，这个相当于我们原来关系数据库中表的主键，当你在插入文档记录时没有指定该字段，MongoDB会自动创建，其类型是ObjectID类型。如果我们在插入文档记录时指定该字段也可以，其类型可以是ObjectID类型，也 可以是MongoDB支持的任意类型。 

```cmd
db.spit.insert({_id:"1",content:"hello world"})
```

如果我想按一定条件来查询，比如我想查询userid为1013的记录，怎么办？很简单！只要在find()中添加参数即可，参数也是json格式

```cmd
db.spit.find({_id:'1'})
```

如果你只需要返回符合条件的第一条数据，我们可以使用findOne命令来实现

```cmd
db.spit.findOne({_id:'1'})
```

如果你想返回指定条数的记录，可以在find方法后调用limit来返回结果

```cmd
db.spit.find().limit(3)
```

## 修改与删除文档

修改文档的语法结构

如果我们想修改_id为1的记录，name为张三

```cmd
db.spit.update({_id:"1"},{name:"张三"})
```

执行后，我们会发现，这条文档除了name字段其它字段都不见了，为了解决这个问题， 

我们需要使用修改器$set来实现

```cmd
db.spit.update({_id:"1"},{$set:{name:"张三"}})
```

删除文档的语法结构

```cmd
db.spit.remove(条件)
```

删除全部数据

```cmd
db.spit.remove({})
```

如果删除age=18的记录，输入以下语句 

```cmd
db.spit.remove({age:18})
```

## 统计条数 

```cmd
db.spit.count()
```

如果按条件统计 ，例如：统计age为18的记录条数

```cmd
db.spit.count({age:'18'})
```

## 模糊查询 

MongoDB的模糊查询是通过正则表达式的方式实现的。格式为： 

```xxx
/模糊查询字符串/
```

查询姓名包含“张三”的所有文档

```cmd
db.spit.find({content:/张三/})
```

查询姓名中以“王”开头的，

```cmd
db.spit.find({content:/^王小虎/})
```

## 大于 小于 不等于 

```cmd
db.spit.find({ "field" : { $gt: value }}) // 大于: field > value
db.spit.find({ "field" : { $lt: value }}) // 小于: field < value
db.spit.find({ "field" : { $gte: value }}) // 大于等于: field >= value
db.spit.find({ "field" : { $lte: value }}) // 小于等于: field <= value
db.spit.find({ "field" : { $ne: value }}) // 不等于: field != value
```

##  包含与不包含 

包含使用 **$in** 操作符

```cmd
db.spit.find({id:{$in:["1","12"]}})
```

不包含使用 **$nin** 操作符

```cmd
db.spit.find({id:{$nin:["1","12"]}})
```

## 条件连接

我们如果需要查询同时满足两个以上条件，需要使用 **$and** 操作符将条件进行关联。（相 当于SQL的and）

格式为

```cmd
$and:[ { },{ },{ } ]
```

查询集合中age大于等于18并且小于20的文档 

```cmd
db.spit.find({$and:[ {age:{$gte:100018 ,{age:{$lt:20} }]})
```

如果两个以上条件之间是或者的关系，我们使用 **$or** 操作符进行关联，与前面and的使用方式相同

格式为

```cmd
$or:[ { },{ },{ } ]
```

查询集合中age为18，或者姓名等于张三的文档记录

```cmd
db.spit.find({$or:[ {age:"18"} ,{name:"张三" }]})
```

## 列值增长

如果我们想实现对某列值在原有值的基础上进行增加或减少，可以使用$inc运算符来实现 

增长

```cmd
db.spit.update({_id:"1"},{$inc:{age:NumberInt(1)}} )
```

减少

```cmd
db.spit.update({_id:"1"},{$inc:{age:-NumberInt(1)}} )
```

