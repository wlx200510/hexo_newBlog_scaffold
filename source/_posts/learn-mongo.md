---
title: mongoDB数据库学习总结
date: 2018-06-22 09:56:50
categories:
- 技术
tags:
- 学习
- 积累
---
MongoDB数据库是Node做后台时最常用的数据库形式之一，这是完整学习Node的关键一步，学习下这个数据库的底层命令，为`mongoose`学习打下基础吧。
<!-- more -->
<!-- excerpt -->
### 基本概念与安装

mongo > 非关系性数据库 > 集合 > 文件

- Windows 用户向导：https://docs.mongodb.com/manual/tutorial/install-mongodb-on-windows/
- Linux 用户向导：https://docs.mongodb.com/manual/administration/install-on-linux/
- Mac 用户向导：https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/

可视化的`mongo`数据库工具推荐：[MongoChef](https://studio3t.com/) 是另一款强大的`MongoDB`可视化管理工具，支持`Windows`、`Linux`和`Mac`。

![操作界面](learn-mongo/example.png)

### 基本的`shell`操作

```mongo
show dbs  // 链接`mongo`数据库后显示所有非空的库
use admin // 进入某一个库，如果不存在则新建
show collections // 显示所在库的所有集合
db // 显示当前所在的数据库
exit // 退出至上一层，直至断开数据库链接

db.集合名.insert({数据})         // 在库下对新建集合或已有集合插入数据(不存在则新建)
db.集合名.update({查询}, {修改})  // 在库中的此集合下更新某一条数据
db.集合名.remove({数据})         // 在库中的此集合下删除某一条数据
db.集合名.find()                // 查找在库下此集合中的所有数据
db.集合名.findOne()             // 查询库下此集合中第一个文件数据
db.集合名.drop()                // 删除库下此集合
```

### 使用`js`脚本来编写`mongoShell`命令

本质就是`mongo`支持直接通过脚本文件来执行一系列的命令，并且在这个脚本中默认注入了一些方法，如`connect`等

```javascript
var insertName = "karl"
var timeStamp = Date.parse(new Date())
var jsonDatabase = {"loginName": insertName, "timeStamp": timeStamp}
var db = connect('test') // 这个含义是在mongo中`use test`
db.name.insert(jsonDatabase)

print('login success')
```

把以上代码保存为`jsTask.js`, 在命令行中使用`mongo jsTask.js`即可 或者`load('./jsTask.js')`

如果`db.name.insert()`的参数是一个元素为json对象的数组，则是批量插入

### `update`修改器用法

> 在实际应用中，由于修改操作与关系性数据库不同，容易覆盖之前的所有数据，因此引入了修改器关键字，在此重点介绍

`update()`这个方法可以传入第三个参数，第三个参数可以理解为`options`的对象

```js
{   // 这个是workmate的储存数据结构, 更新相关操作都是在这个基础上生成的
    name:'MinJie',
    age:20,
    sex:0,
    job:'UI设计',
    skill:{
        skillOne:'PhotoShop',
        skillTwo:'UI',
        skillThree:'Word+Excel+PPT'
    },
    interest: ['看电影', '读书'],
    regeditTime:new Date()
}
```

- 单文件修改

`$set`修改器，用于修改指定的键值：
    `db.workmate.update({"name":"MinJie"},{$set:{"sex":2,"age":21}})`
    上面这条命令就可以把`MinJie`这条数据对应的`sex`和`age`的键值给更新了，其他内容保持不变。
    `db.workmate.update({"name":"MinJie"}, {$set:{skill.skillThree:"PPT"}})`
    这条命令说明`$set`支持通过`.`符号来修改更新内嵌的对象内容

`$unset`修改器，用于删除指定的键：
    `db.workmate.update({"name":"MinJie"}, {$unset:{"age":''}})`

`$inc`数字修改器，只针对数字项的计算修改，如在原来数字基础上减去5：
    `db.workmate.update({"name":"MinJie"}, {$inc:{"age":-5}})`

`options`中的两个参数`multi`和`upsert`：
    `db.workmate.update({name:'xiaoWang'},{$set:{"age":20}},{upsert:true})`
    用于更新集合中的键值对时，如果不存在此键，则执行插入操作，默认是不插入
    `db.workmate.update({}, {$set:{"interest":[]}}}, {multi: true})`
    用于在向表中更新新的键名时，使之应用于全部的文件，使每条数据都发生更改
    注意简写：update的默认第三个参数表征`upsert`, 第四个参数表征`multi`

- 数组批量修改

`$push`修改器，用于追加数组/内嵌文档值：
    为数组类型的键增加值 `db.workmate.update({name:'xiaoWang'}, {$push:{"interest":'draw'}})`
    为内嵌文档内容增加值 `db.workmate.update({name:'xiaoWang'}, {$push:{"skill.skillFour":'draw'}})`

`$addToSet`修改器，当数组中不存在要更新的值时再触发追加操作：
    用法示例(已经有`draw`则不触发) `db.workmate.update({name:'xiaoWang'}, {$addToSet:{"interest":'draw'}})`
    使用`$ne`的等效语句 `db.workmate.update({name:'xiaoWang',"interest":{$ne:'draw'}},{$push:{"interest":'draw'}})`

`$each`数组修改器，用于一次性插入多个数据到数组中：
    现在要给xiaoWang,一次加入三个爱好，唱歌（Sing），跳舞（Dance），编码（Code）：
```javascript
    var newInterset=["Sing","Dance","Code"]
    db.workmate.update({name: 'xiaoWang'}, {$addToSet:{"interest":{$each:newInterset}}})
```

`$pop`数组删除修改器，通过传参数来决定从数组首位(-1)还是末尾(1)删除：
    删除最后一个兴趣 `db.workmate.update({name:'xiaoWang'},{$pop:{"interest":1}})`

数组定位修改可以用类似内嵌文档的语法：
    把第二个兴趣值改成读书 `db.workmate.update({name: 'xiaoWang'}, {$set:{"interest.1":'read'}})`

### 应答式操作的修改方式

> 应答式写入就会给我们直接返回结果（报表），结果里边的包含项会很多，从而可以灵活地根据上一步的结果来决定下一步的结果

- 之前的所有内容都是非应答式操作，也就是操作完之后没有反馈，并不知道是否成功
- 需要引入`db.runCommand()`这个执行器，用于包裹执行命令，然后可以根据返回内容判断是否操作成功。
- `db.listCommonds()`用于看执行器可以执行的命令，如`db.runCommand({ping:1})`看数据库链接是否成功
- `findAndModify`也是执行器的命令之一，可以用于查找并触发删除或者修改等动作并通过执行器反馈，示例如下

```javascript
var myModify={
    findAndModify:"workmate", // 填入要操作的集合名
    query:{name:'xiaoWang'},
    update:{$set:{age:18}}, // 表示修改该条查询到的数据
    new:true    //更新完成，需要查看结果，如果为false不进行查看结果
}
var ResultMessage=db.runCommand(myModify);
 
printjson(ResultMessage) //打印JSON数据
```

`findAndModify`可用的属性值：

1. query：需要查询的条件/文档
2. sort： 进行排序
3. remove：[boolean]是否删除查找到的文档，值填写`true`，可以删除。(与`update`不可共存)
4. new:[boolean]返回更新前的文档还是更新后的文档。
5. fields：需要返回的字段
6. upsert：没有这个值是否增加。

接下来的课程的所有涉及删除和更新的操作都会用到`findAndModify`这个功能，从而保持数据操作的严谨性。

### `find`命令来查找数据

> `find`操作是数据库最常用的查询命令，其语法与关系性数据库差异是最大的，用例子来讲方便入手

`db.workmate.find({"skill.skillOne":"HTML+CSS"}, {name: 1, _id: 0, job: 1})`
`find`的第一个参数是要查询的`query`，后面的对象参数是指定返回来的字段，默认是全返回，`_id`是这条数据的默认`key`键，默认返回，如果不需要返回需指定此项是`0`或`false`，其他项需返回，则要指定为`1`或`true`。

`db.workmate.find({age:{$lte:30,$gte:25}},{name:true,age:true,"skill.skillOne":true,_id:false})`
上面这个命令用于查找出年龄大于25小于30的同事，这里用到了不等修饰符

- 小于($lt):英文全称`less-than`
- 小于等于($lte)：英文全称`less-than-equal`
- 大于($gt):英文全称`greater-than`
- 大于等于($gte):英文全称`greater-than-equal`
- 不等于($ne):英文全称`not-equal`

```javascript
var startDate= new Date('01/01/2018');
db.workmate.find(
    {regeditTime:{$gt:startDate}},
    {name:true,age:true,"skill.skillOne":true,_id:false}
)
```
上面的命令用于找出注册时间大于1月1日的数据，可以看出用时间来比较很方便。

> `find`同样支持多条件查询，其对应的关键字为：`$and、$or、$not`这三个

```javascript
db.workmate.find({$and:[ // 同时满足两个条件的数据
    {age:{$gte:30}},
    {"skill.skillThree":'PHP'}
]}, {name:1,"skill.skillThree":1,age:1,_id:0})

db.workmate.find({$or:[ // 只要满足一个条件即可
    {age:{$gte:30}},
    {"skill.skillThree":'PHP'}
]}, {name:1,"skill.skillThree":1,age:1,_id:0})

db.workmate.find({
    age:{ // 筛选不满足条件的数据
        $not:{ $lte:30, $gte:20 }
    }
}, {name:1,"skill.skillOne":1,age:1,_id:0})
```

> `find`的数组查询，即当储存的某一个键的值是数组时，有额外的查询方法和技巧

现在我把之前的数据结构加以改变，增加一个`interests`数组类的储存

基本查询(很少用，得知概念即可)：

不带中括号的查询：为只要这一项是待匹配数据的数组中有的，就会找出来，常用的是`tag`标示。
`db.workmate.find({interest:'看电影'}, {name:1,interest:1,age:1,_id:0})`

带中括号的查询：必须是数组中的元素完全匹配才能找出来，也就是必须完全一致
`db.workmate.find({interest:['画画','聚会','看电影']}, {name:1,interest:1,age:1,_id:0})`

- `$all`-数组多项查询，用于同时查询数组中多个元素，类似非数组中的`$and`，但用法不同
- `$in`-数组或查询，用于用多个元素来筛选数据，类似非数组中的`$or`，但用法不同
- `$size`-数组数量查询，用于指定查出数组的长度为指定值的数据，数组查询特有
- `$slice`-非查询指令，用于指定数组项的返回个数，用在第二个参数中，如显示前两项等要求

```javascript
db.workmate.find({age:{$in:[25,33]}}, //传递一个数组，值在其中即为满足 类似$or 还有$nin是相反的作用
    {name:1,"skill.skillOne":1,age:1,_id:0}
)
db.workmate.find( // 查询出喜欢看电影并且喜欢看书的同事
    {interest:{$all:["看电影","看书"]}},
    {name:1,interest:1,age:1,_id:0} 
)
db.workmate.find( // 查询出兴趣数量为5个的同事
    {interest:{$size:5}},
    {name:1,interest:1,age:1,_id:0} 
)
db.workmate.find({}, // 每个人的兴趣信息只显示前两项
    {name:1,interest:{$slice:2},age:1,_id:0} 
)
```

> 学习使用`find`命令进行分页查询之前，首先看一下如何编写`js`脚本来进行数据查询，本质就是赋值后循环打印

使用`forEach`方法：

```javascript
var db = connect("company")  //进行链接对应的集合collections
var result = db.workmate.find() //声明变量result，并把查询结果赋值给result
result.forEach(function(result){
    printjson(result)  //用json格式打印结果
})
```

使用`hasNext+while`的方法：

```javascript
var db = connect("company")  //进行链接对应的集合collections
var result = db.workmate.find() //声明变量result，并把查询结果赋值给result
//利用游标的hasNext()进行循环输出结果。
while(result.hasNext()){
    printjson(result.next())
}
```

接下来就可以尝试通过写脚本来完成查询数据库的操作啦

> 分页查询指令的学习，可以使用类似函数式的链式调用，可读性很强

`find`参数：
- `query`：这个就是查询条件，MongoDB默认的第一个参数。
- `fields`：（返回内容）查询出来后显示的结果样式，可以用true和false控制是否显示。
- `limit`：返回的数量，后边跟数字，控制每次查询返回的结果数量。
- `skip`:跳过多少个显示，和limit结合可以实现分页。
- `sort`：排序方式，从小到大排序使用1，从大到小排序使用-1。

使用传参数操作的脚本:

```javascript
var db = connect("company")
var findData = {
    query: {interest: {$size:3}}, // 兴趣数量3个
    fields: {_id: 0, name: 1, sex: 1, interest: {$slice: 4}},
    limit: 3, // 每页三个数据
    skip: 1, // 跳过第一个显示，如果取limit的值 就是下一页
    sort: {age: 1} //根据年龄从小到大排序
}
var result = db.workmate.find(findData.query, findData.fields, findData.limit, findData.skip, findData.sort)
result.forEach(function(json){
    printjson(json)
})
```

链式操作脚本:

```javascript
var db = connect("company")
var query = {interest: '看电影'}, // 兴趣包括看电影
    fields = {_id: 0, name: 1, sex: 1, interest: {$slice: 3}}
var result = db.workmate.find(findData.query, findData.fields).limit(4).skip(4).sort({age: -1})
result.forEach(function(json){
    printjson(json)
})
```

> `$where`操作符，可以通过`js`语法来查询，方便，但要慎用，安全性和性能较低

```javascript
db.workmate.find({ $where: "this.interest.length < 4" })

db.workmate.find( // 查找年龄小于30的，this指向所检索数据对象
    {$where:"this.age>30"},
    {name:true,age:true,_id:false}
)
```

### mongoDB数据库权限管理

> 对于数据库，我们有必要根据不同的用户赋予不同的权限，还有设置用户名和密码，以及鉴权等操作，看一下基本的命令

首先进入`admin`库中，进入方法是直接使用`use admin` 就可以。
进入后可以使用`show collections`来查看数据库中的集合。默认是只有一个集合的`system.version`。

新建用户的脚本:

```javascript
db.createUser({
    user:"karl",
    pwd:"123456",
    customData:{
        name:'geekarl',
        email:'web@126.com',
        age:28,
    },
    roles:[ // 数据库用户角色只有两个选项 read 和 readWrite
        {
            role:"readWrite",
            db:"company" // 针对特定数据库
        },
        'read' // 针对其他数据库
    ]
})
```

内置角色(了解即可)：

- 数据库用户角色：read、readWrite；
- 数据库管理角色：dbAdmin、dbOwner、userAdmin;
- 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManage；
- 备份恢复角色：backup、restore；
- 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、dbAdminAnyDatabase
- 超级用户角色：root
- 内部角色：__system

查找和删除用户(user加s)：

`db.system.users.find()`
`db.system.users.remove({user: 'karl'})`

`mongoDB`的鉴权操作：`db.auth("karl", "123456")` 成功返回1，失败返回0

关闭重启`mongo`，以鉴权方式启动: `mongod --auth` 此后使用用户名和密码登陆后才能对`mongo`进行操作
`mongo -u karl -p 123456 127.0.0.1:27017/admin`然后就可以正常操作了

以上是对`mongo`的操作进行权限保护，在工作中用后端应用连接数据库操作是一个很频繁且需要慎重的过程，因此这一块需要明确的认知

### `mongo`数据库的备份和恢复

> 这个工作大部分是`DBA`或者运维同学来操作，但如果做一个完整的项目，必须要有容错的手段和容灾备份。

`mongodump`为数据备份的操作命令，其`shell`中基本格式如下：

```shell
mongodump
    --host 127.0.0.1
    --port 27017
    --out D:/databack/backup
    --collection myCollections
    --db test
    --username username
    --password password
```

把当前`Mongo`的库备份到D盘下的`dataBack`文件夹(登陆者需要有相应的权限)：
`mongodump --host 127.0.0.1 --port 27017 --out D:/databack`

备份好之后就可以用于数据库的还原了，还原数据库的命令是`mongorestore`，在数据库出现意外时还原

```shell
mongorestore
    --host 127.0.0.1
    --port 27017
    --username username
    --password password
    <path to the backup>
```

使用命令进行恢复：`mongorestore --host 127.0.0.1 --port 27017 D:/databack/`

备份和恢复都可以做成脚本任务，备份可能需要运维定时间运行。脚本设置可以参考[这里](https://blog.csdn.net/huwei2003/article/details/44059797)，需要有`shell`基础。

### 数据库的索引

> 用于数据检索时，通过指定字段建立对应索引值，从而在检索`find`的命令执行时，相应字段的检索效率会成倍提升。

显示索引: `db.randomInfo.getIndexes()` 会发现`_id`是唯一的索引

```json
[
    {
        "v" : 2,
        "key" : {
            "_id" : 1
        },
        "name" : "_id_",
        "ns" : "company.randomInfo"
    }
]
```

为集合添加索引: `db.randomInfo.ensureIndex({username:1})` 把用户名建立索引，通过用户名来检索的速度提升
删除索引(): `db.randomInfo.dropIndex('randNum0_1');` 
这里需要注意的是删除时填写的值，并不是我们的字段名称(`key`)，而是我们索引查询表中的`name`值。
如果复合查询的两个字段都建立了索引，可以通过`hit`方法来指定以何种字段优先检索:
`var  rs= db.randomInfo.find({username:'7xwb8y3',randNum0:565509}).hint({randNum0:1});`

整个简易教程到此结束，希望对入门`mongoDB`的人有一定帮助和借鉴~