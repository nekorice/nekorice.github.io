title: mongo常用操作语句对照
date: 2017-05-26 13:15:08
tags:
- mongo
- sql mapping
---

虽然官方已经有一版表格，但是并不完全，而且都是shell中的指令，没有对应语言版本的操作。

这里记录一下常用的语句变换做一个补充。

语言实现版本对应为pymongo中的使用方法。

<!--more-->

官方的参考详见 [SQL to MongoDB Mapping Chart](https://docs.mongodb.com/manual/reference/sql-comparison/#examples)

<style>
  .entry-content>table td,.entry-content table th { word-break: break-all; padding: 5px;}
  .entry-content>table td { border: 1px solid #CCC; }
  .entry-content>table { border: 1px solid #CCC;  margin-bottom: 20px;}
</style>

## insert 

| sql | mongo shell |
| ------ | ------ |
|  INSERT INTO users(user_id, age) <br>VALUES ("001",45),("002", 30)  |  db.users.insert([{'user_id':'001', 'age':45}, <br>{'user_id':'002', 'age':30}])  |
|  INSERT INTO users(user_id, age)<br>VALUE ("001", 45) <br>ON DUPLICATE KEY UPDATE age = 45 |  db.users.update({user_id:'001'}, {user_id:'001', age:45}, upsert=true)  |


**pymongo**

pymongo中insert语句对应于

db.users.insert_many([dict, dict])

db.users.insert_one(dict)

insert on duplicate可以用replace one，也可以用 update_one+upsert=True

db.users.replace_one({'user_id':'001'}, {'user_id':'001', 'age':45}, upsert=True)

db.users.update_one({'user_id':'001'}, {'$set':{'age':45}}, upsert=True)

## update

| sql | mongo shell | 
| ------ | ------ | 
| UPDATE users SET status = "C" WHERE age > 25 | db.users.update({ age: { $gt: 25 } },{ $set: { status:"C" } },{ multi: true }) | 
| UPDATE users SET age = age + 3 WHERE status = "A" | db.users.update({ status: "A" },{ $inc: { age: 3 }},{ multi: true }) | 

**pymongo**

pymongo里面基本和shell一样，直接使用参数就可以，不过python里面字典的key需要是字符串，记得加上引号

db.users.update_many({ 'age': { '$gt': 25 } },{ '$set': { 'status':"C" } })

db.users.update_many({ 'status': 'A' },{ '$inc': { 'age': 3 }})


## delete&&drop

| sql | mongo shell | 
| ------ | ------ | 
| DELETE FROM users <br>WHERE status = "D" | db.users.remove( { status: "D" } ) |
| DELETE FROM users | db.users.remove() | 
| DROP TABLE users | db.users.drop() | 


**pymongo**

pymongo中全删除也要传入一个空的字典

db.user.remove({'status':'D'})

db.user.remove({})

db.user.drop()

## select (simple)

这里就列一个用来参考格式，官网select的例子很全面，[SQL to MongoDB Mapping Chart](https://docs.mongodb.com/manual/reference/sql-comparison/#examples)

| sql | mongo shell | 
| ------ | ------ | 
| SELECT name <br>FROM users <br>WHERE status = "A" <br>OR age <= 50 | db.users.find({ $or: [ { status: "A" } ,<br>{ age:{$lte: 50}}, {name:1} ] }) |

判断条件

$lt <
$gt >
$neq  !=
$contains 对应于  in（）
like  对应于正则表达式（js） /a/

WHERE 条件

$or  对应于  Where .. Or ..
$and 对应于  Where .. And ..


**pymongo**

像count，limit, sort这些方法都是用在结果集上面调用的方法。

db.users.find(dict).count(), db.users.find(dict).limit(), db.users.find(dict).sort()

查找一个元素时使用find_one直接返回一个字典，否则返回的是一个迭代器。如果需要多次使用建议强制转换成list，否则迭代一次之后，第二次将取出空值。

```python

result = db.user.find()
names = [u['name'] for u in result]    
status = [u['status'] for u in result]

#name有值
#status为空
```


## select (group by)

计算sum，avg，max，min

SQL 

```
SELECT count(1) as count, sum(score) as sumScore, 
       avg(score) as avgScore, min(score) as minScore
FROM users
WHERE team = "B"
GROUP BY status
ORDER BY sumScore desc
```

mongo

```
db.users.aggregate([
  {"$match":{"team":"B"}},
  {"$group":{"_id":'$status', "count":{"$sum":1}, 
             "sumScore":{$sum:"$score"},"avgScore":{$avg:$score}, 
             "minScore":{$min:$score}}},
  { $sort: { sumScore: -1 } }
  }])
```

aggregate 接受一个数组的参数，每个object以key区分有以下几个参数


$match对应于where条件
$group

&emsp;&emsp;_id对应于group by的key,支持组合字段。_id : null 则是对整个document进行处理。
&emsp;&emsp;其他key对应于查询出来的字段别名，
&emsp;&emsp;指代原来表格的字段，需要加$,比如其中的$score,$status
&emsp;&emsp;group字段之中还支持四则运算，比如两个字段的相乘
&emsp;&emsp;totalPrice: { $sum: { $multiply: [ "$price", "$quantity" ] } },  
&emsp;&emsp;除去sql中的基本操作，mongo还支持$push，$addToSet，把对应内容直接加到数组里。

$sort 类似sort，使用group中定义的字段名称排序
$limit 类似limit，返回的结果集个数
$skip 同skip，跳过结果集个数
$projection 修改文档的输入结构，个人感觉类似创建子查询，不知道效率如何，具体参考[官方文档](https://docs.mongodb.com/manual/reference/operator/aggregation/project/)
$unwind 当你需要统计embedded的array内容时，展开数组内容需要unwind对应的字段，比如这种情况：
[https://stackoverflow.com/questions/14570577/sum-in-nested-document-mongodb](https://stackoverflow.com/questions/14570577/sum-in-nested-document-mongodb)

pymongo 语法和mongo语法一致。另外使用aggregate查询出来的结果集和其他不一样，是一个dict。需要一些额外处理。

```
#db.users.aggregate([match, group, sort]
#aggragate return {'ok': 1 or 0, 'result': the_actual_result}
```


## select embedded

查找嵌入的数据对象，比如array， object（dict）等等

假设有以下结构的collection

```
db.inventory.find()

{ item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" },
{ item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "A" },
{ item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
{ item: "planner", qty: 75, size: { h: 22.85, w: 30, uom: "cm" }, status: "D" },
{ item: "postcard", qty: 45, size: { h: 10, w: 15.25, uom: "cm" }, status: "A" }

```

如果是要搜索完全匹配，直接使用find语法即可,如果只是包含查找条件，不是完全匹配，不会返回结果

```
db.inventory.find({ size:{ h: 14, w: 21, uom: "cm" }})
{ item: "journal", qty: 25, size: { h: 14, w: 21, uom: "cm" }, status: "A" }

db.inventory.find({ size:{ h: 14, w: 21}})
null
```

如果只是关注某个key的值，则使用这种查找，用‘.’操作符连接字段

```
db.inventory.find( { "size.uom": "in" } )
{ item: "notebook", qty: 50, size: { h: 8.5, w: 11, uom: "in" }, status: "A" },
{ item: "paper", qty: 100, size: { h: 8.5, w: 11, uom: "in" }, status: "D" },
```

假设有以下数组的document

```
{ item: "journal", qty: 25, tags: ["blank", "red"], dim_cm: [ 14, 21 ] },
{ item: "notebook", qty: 50, tags: ["red", "blank"], dim_cm: [ 14, 21 ] },
{ item: "paper", qty: 100, tags: ["red", "blank", "plain"], dim_cm: [ 14, 21 ] },
{ item: "planner", qty: 75, tags: ["blank", "red"], dim_cm: [ 22.85, 30 ] },
{ item: "postcard", qty: 45, tags: ["blue"], dim_cm: [ 10, 15.25 ] }

```

普通查找依然是完全匹配，部分匹配则使用$all 操作符

```
db.inventory.find( { tags: { $all: ["red", "blank"] } } )
{ item: "journal", qty: 25, tags: ["blank", "red"], dim_cm: [ 14, 21 ] },
{ item: "notebook", qty: 50, tags: ["red", "blank"], dim_cm: [ 14, 21 ] },
{ item: "paper", qty: 100, tags: ["red", "blank", "plain"], dim_cm: [ 14, 21 ] },
{ item: "planner", qty: 75, tags: ["blank", "red"], dim_cm: [ 22.85, 30 ] },
```

直接查询是否包含字段，不用加任何额外操作符

```
db.inventory.find( { tags: "blank" } } )
{ item: "journal", qty: 25, tags: ["blank", "red"], dim_cm: [ 14, 21 ] },
{ item: "notebook", qty: 50, tags: ["red", "blank"], dim_cm: [ 14, 21 ] },
{ item: "paper", qty: 100, tags: ["red", "blank", "plain"], dim_cm: [ 14, 21 ] },
{ item: "planner", qty: 75, tags: ["blank", "red"], dim_cm: [ 22.85, 30 ] },

```

条件查找，只要有一个元素符合要求就会返回。使用$elemMatch，则保证至少有一个元素，完全符合所有条件。

```
db.inventory.find( { dim_cm: { $gt: 15, $lt: 20 } } )
{ item: "journal", qty: 25, tags: ["blank", "red"], dim_cm: [ 14, 21 ] },
{ item: "notebook", qty: 50, tags: ["red", "blank"], dim_cm: [ 14, 21 ] },
{ item: "paper", qty: 100, tags: ["red", "blank", "plain"], dim_cm: [ 14, 21 ] },
{ item: "postcard", qty: 45, tags: ["blue"], dim_cm: [ 10, 15.25 ] }


db.inventory.find( { dim_cm: { $elemMatch: { $gt: 22, $lt: 30 } } } )
{ item: "planner", qty: 75, tags: ["blank", "red"], dim_cm: [ 22.85, 30 ] },
```

也可以使用.操作符取下标,数组下标规则是0base

```
db.inventory.find( { "dim_cm.1": { $gt: 25 } } )
{ item: "planner", qty: 75, tags: ["blank", "red"], dim_cm: [ 22.85, 30 ] },
```

另外还支持查询数组长度
```
db.inventory.find( { "tags": { $size: 3 } } )
{ item: "paper", qty: 100, tags: ["red", "blank", "plain"], dim_cm: [ 14, 21 ] },

```

官方参考：

[https://docs.mongodb.com/manual/tutorial/query-embedded-documents/](https://docs.mongodb.com/manual/tutorial/query-embedded-documents/)
[https://docs.mongodb.com/manual/tutorial/query-arrays/](https://docs.mongodb.com/manual/tutorial/query-arrays/)
[https://docs.mongodb.com/manual/tutorial/query-array-of-documents/](https://docs.mongodb.com/manual/tutorial/query-array-of-documents/)


## Map-Reduce

这是mongodb独有的操作。

主要要提供一个Map函数和一个reduce函数（js函数），生成统计结果。

执行图如下。

![mapreduce](https://docs.mongodb.com/manual/_images/map-reduce.bakedsvg.svg)

query提供输入条件，和其他查询条件语法一致。

map函数使用emit方法，emit（key，value）的键值对，相同的key的value会放入到一个数组values中。

生成的key和values作为reduce的输入参数使用，reduce把values数组处理成一个value， 返回一个value。

output设定是返回结果存储的collection名称。

map-reduce据说效率比较低（未验证），不适合做实时查询。














