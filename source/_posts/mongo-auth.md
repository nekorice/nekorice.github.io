title: mongo权限认证
date: 2017-06-29 23:16:05
tags:
- mongo
- auth
---

记录一下mongo开启权限认证，作为备忘

<!-- more -->

### 1）开启权限之前需要先新建一个admin用户

进入mongo shell

```
db.auth("myUserAdmin", "abc123" )

use admin
db.createUser(
  {
    user: "myUserAdmin",
    pwd: "abc123",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)
```


### 2）修改配置文件的配置段开启mongo权限认证

先关闭mongo

```
security
authoration： enabled

```

如果没有使用配置文件启动也可以用以下指令启动mongod

```
mongod --auth --port 27017 --dbpath /data/db1
```



### 3）设置细分权限

roles权限是一个数组，可以在createUser里面的roles字段里面设置，也可以使用下面两个函数来去除和增加用户权限。

db.revokeRolesFromUser()
db.grantRolesToUser(）

每一个权限的role则对应mongo里预定义权限。主要以下这些。

* root  root用户，超级管理员
* read  只读用户。除了system collection其他都可以读取。
* readWriter 读写用户。不可读写system collection。
* useradmin  可以管理用户权限，不能读写
* dbadmin 可以维护db的配置，不能读写
* dbOwner  等于dbadmin，useradmin，readWriter权限相加。

除了root，上述role都可以加上后缀AnyDatabase，来表示对所有db（mongo3.x以上除去local和config）赋予对应权限。

另外还可以针对集群设置权限。

除了以上的角色，如果觉得mongodb的权限还不够细致，mongodb还支持自定义角色。

具体请参考文档：
https://docs.mongodb.com/manual/core/security-user-defined-roles/

### 4）使用pymongo连接mongodb 

**pymongo 2.x**

```python
>>> import urllib
>>> password = urllib.quote_plus('pass/word')
>>> password
'pass%2Fword'
>>> MongoClient('mongodb://user:' + password + '@127.0.0.1')
MongoClient('127.0.0.1', 27017)
```

**pymongo 3.x**

```python
>>> from pymongo import MongoClient
>>> client = MongoClient('example.com')
>>> client.the_database.authenticate('user', 'password', mechanism='SCRAM-SHA-1')
```

参考文档：

https://docs.mongodb.com/manual/tutorial/enable-authentication/
https://docs.mongodb.com/manual/reference/configuration-options/#file-format
https://docs.mongodb.com/manual/core/security-built-in-roles/
https://docs.mongodb.com/manual/reference/method/db.createRole/#db.createRole
https://docs.mongodb.com/manual/reference/method/#role-management-methods
http://api.mongodb.com/python/current/examples/authentication.html

