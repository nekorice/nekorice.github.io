title: celery任务状态监控
date: 2017-11-19 18:18:32
tags:
- celery
- django
- 异步任务
---

celery是python里常用的一个异步任务队列，使用celery可以大大简化搭建任务队列的工作。实际使用中可能会需要监控一些任务或者定时任务的运行状态。

这里就讲一下celery的任务状态监控相关的方法。

<!-- more -->

> 单独使用celery命令格式为 celery -A [proj] [cmd]
   在django下使用时，用manage.py启动时则不需要-A参数，命令格式为
   python manage.py celery [cmd]

#### 1. 通用命令

1）  查看celery worker的状态

```
python manage.py status

celery@DEV_192_168_X_XX: OK
celery@DEV_192_168_X_XX: OK
celery@DEV_192_168_X_XX: OK
celery@DEV_192_168_X_XX: OK
celery@DEV_192_168_X_XX: OK

1 node online.
```

返回内容表示目前有5个worker运行中，运行正常，
@前面是worker的name，后面是worker的hostname，在启动worker时用-n workername@%h参数传入，如果不传递，默认为celery@hostname。
会报一个warning，建议给每个worker定义一个唯一的名字。

第二部分返回值是worker的计数，只是简单的统计了不同名称的个数.比如上面没有指定队列名称，就会统计只有一个活动的node.

2） 查看 celery task和queue详细信息

**查看当前正在运行的task**

```
python manage.py inspect active
-> celery@localhost: OK
    - empty -
```
**查看当前的队列的具体信息**

```
python manage.py inspect active-queues   

-> celery@localhost: OK
    * {u'exclusive': False, u'name': u'celery', u'exchange': {u'name': u'celery'
, u'durable': True, u'delivery_mode': 2, u'passive': False, u'arguments': None,
u'type': u'direct', u'auto_delete': False}, u'durable': True, u'routing_key': u'
celery', u'no_ack': False, u'alias': None, u'queue_arguments': None, u'binding_a
rguments': None, u'bindings': [], u'auto_delete': False}
```

输出 worker的内存占用(需要安装psutil库)
```
python manage.py celery inspect memdump  
-> celery@localhost: OK
    - rss (end): 37.352MB.
```
输出worker的统计信息
```
python manage.py celery inspect stats  

-> celery@localhost: OK
    {
        "broker": {
            "alternates": [],
            "connect_timeout": 4,
            "failover_strategy": "round-robin",
            "heartbeat": 0,
            "hostname": "192.168.99.100",
            "insist": false,
            "login_method": null,
            "port": 6379,
            "ssl": false,
            "transport": "redis",
            "transport_options": {},
            "uri_prefix": null,
            "userid": null,
            "virtual_host": "1"
        },
        "clock": "7624",
        "pid": 8308,
        "pool": {
            "max-concurrency": 4,
            "max-tasks-per-child": "N/A",
            "processes": [
                8584,
                9180,
                9792,
                5600
            ],
            "put-guarded-by-semaphore": true,
            "timeouts": [
                0,
                0
            ],
            "writes": "N/A"
        },
        "prefetch_count": 16,
        "rusage": "N/A",
        "total": {
            "tasks.jobs.test": 1562
        }
    }
```

broker段记录了连接的broker信息，pool段有当前子进程/子线程号，total则是当前task运行到现在记录的所有数据总和。这些信息在worker退出后会被重置。
如果worker进程已经异常退出，则无法用inspect获取信息统计。

字段更多详细内容可查阅以下文档
http://docs.celeryproject.org/en/latest/userguide/workers.html#worker-statistics

Inspect还有很多其他参数，可以输入 celery inspect --help 来查看。
主要常用的就是以上几条命令.

3）  清空队列

当消息堆积，或者worker挂掉，但是队列已经塞入了大量无用信息时。如果不关注堆积消息的处理（比如一些定时重复计算的任务）。可以直接清空队列。
防止启动work后，需要处理冗长的堆积消息而导致后续消息处理延迟。

```
python manage.py celery purge -Q [queue_name]
```
建议带上-Q参数，清空指定队列。
旧版本的celery不支持-Q参数，可以使用对应的broke指令，清理一下对应的队列。可以参考下文中redis清除队列，以及rabbitmq purge队列的指令

4） 使用flower监控task运行结果和状态。

直接安装flower
```
pip install flower
```

启动flower，开启持久化，如果不带persisitent参数。重启flower就会导致数据丢失
```
python manage.py flower --port=5555  --persistent=True --db=./flower  
```

Flower的除了界面样式优点陈旧之外其他都很好用，另外可以使用参数开启权限认证。不过界面上有停止worker的操作和调整work的参数，可能会导致一些安全性的问题。内网监控可以使用。

#### 2. 查看消息队列内部内容

1） 使用redis作为Broker

>需要先安装redis环境，使用redis-cli查看。
基本就是使用redis指令对存储在redis里的list进行各种操作。

**查看当前所有队列**

即查看当前存储的所有的key

redis-cli -h HOST -p PORT -n DB_NUMBER(默认为0) keys \* 
```
#  redis-cli -n 1 keys \* 
1) "_kombu.binding.celery.pidbox"
2) "_kombu.binding.celeryev"
3) "_kombu.binding.celery"
```
_kombu.binding.celery是当前的工作队列， pidbox和celeryev是celery内部用来发送event和管理其他队列的队列。

**清空队列**

```
# 清空队列全部内容
redis-cli -h HOST -p PORT -n DB_NUMBER ltrim key 0 0
 
# 清除前100条消息
redis-cli -h HOST -p PORT -n DB_NUMBER ltrim key 100 -1

# 保留前100条消息
redis-cli -h HOST -p PORT -n DB_NUMBER ltrim key 0 100
```

**查看消息长度**

redis-cli -h HOST -p PORT -n DB_NUMBER llen [上面列出的key的最后一段]
```
#  redis-cli -n 1 llen celery
(integer) 9
```

**查看消息队列具体内容**

redis-cli -h HOST -p PORT -n DB_NUMBER LRANGE key 0 -1

```
#  redis-cli -n 1 LRANGE celery 0 -1
17) "{\"body\": \"eyJleHBpcmVzIjogbnVsbCwgInV0YyI6IHRydWUsICJhcmdzIjogW10sICJjaG
9yZCI6IG51bGwsICJjYWxsYmFja3MiOiBudWxsLCAiZXJyYmFja3MiOiBudWxsLCAidGFza3NldCI6IG
51bGwsICJpZCI6ICI2OGQyZGJlYS1jNjdiLTRmMmYtOGQ1My02NDQ5M2JkMzlmNmQiLCAicmV0cmllcy
I6IDAsICJ0YXNrIjogInRhc2tzLmpvYnMudGVzdCIsICJ0aW1lbGltaXQiOiBbbnVsbCwgbnVsbF0sIC
JldGEiOiBudWxsLCAia3dhcmdzIjoge319\", \"headers\": {}, \"content-type\": \"appli
cation/json\", \"properties\": {\"body_encoding\": \"base64\", \"correlation_id\
": \"68d2dbea-c67b-4f2f-8d53-64493bd39f6d\", \"reply_to\": \"42f2e00e-cd9a-3259-
a201-d5e94c869b90\", \"delivery_info\": {\"priority\": 0, \"routing_key\": \"cel
ery\", \"exchange\": \"celery\"}, \"delivery_mode\": 2, \"delivery_tag\": \"d93d
25ee-dc33-486a-88ac-af61b123b615\"}, \"content-encoding\": \"utf-8\"}"
18) "{\"body\": \"eyJleHBpcmVzIjogbnVsbCwgInV0YyI6IHRydWUsICJhcmdzIjogW10sICJjaG
9yZCI6IG51bGwsICJjYWxsYmFja3MiOiBudWxsLCAiZXJyYmFja3MiOiBudWxsLCAidGFza3NldCI6IG
51bGwsICJpZCI6ICI4MTk5Y2RiMy0wNjNmLTQ1ZTYtYTIwOS1kNWE5NjNmN2ZiYzIiLCAicmV0cmllcy
I6IDAsICJ0YXNrIjogInRhc2tzLmpvYnMudGVzdCIsICJ0aW1lbGltaXQiOiBbbnVsbCwgbnVsbF0sIC
JldGEiOiBudWxsLCAia3dhcmdzIjoge319\", \"headers\": {}, \"content-type\": \"appli
cation/json\", \"properties\": {\"body_encoding\": \"base64\", \"correlation_id\
": \"8199cdb3-063f-45e6-a209-d5a963f7fbc2\", \"reply_to\": \"42f2e00e-cd9a-3259-
a201-d5e94c869b90\", \"delivery_info\": {\"priority\": 0, \"routing_key\": \"cel
ery\", \"exchange\": \"celery\"}, \"delivery_mode\": 2, \"delivery_tag\": \"2246
c451-3221-4579-b43b-ed5fa8085661\"}, \"content-encoding\": \"utf-8\"}"
```
大致可以看出消息被编码成了base64的格式，还有各个信息的routing_key和exchange，关于消息的编码，各位有兴趣可以继续深究。


2 ）使用Rabbit-mq作为Broker

>需要rabbitmq环境，因为rmq本身就是消息队列，所以每个队列对应于rmq里的一个queue，具体操作和管理rabbitmq相同。

**队列状态**

```
sudo rabbitmqctl list_queues 
sudo rabbitmqctl list_queues name messages messages_ready messages_unacknowledged 
sudo rabbitmqctl list_queues name consumers
sudo rabbitmqctl list_queues name memory
```

**队列消息查看**

需要使用python的工具来辅助管理，参考文档
http://www.rabbitmq.com/management-cli.html

```
sudo python rabbitmqadmin get queue=queuename requeue=true count=10
```

#### 3. 监控celery的事件

> 注意：如果work里面设置event=disable，就无法监听。

直接参考以下代码进行监控
```
from celery import Celery
def my_monitor(app):
    state = app.events.State()

    def announce_failed_tasks(event):
        state.event(event)
        # task name is sent only with -received event, and state
        # will keep track of this for us.
        task = state.tasks.get(event['uuid'])

        print('TASK FAILED: %s[%s] %s' % (
            task.name, task.uuid, task.info(),))

    def announce_receive_tasks(event):
        state.event(event)
        # task name is sent only with -received event, and state
        # will keep track of this for us.
        task = state.tasks.get(event['uuid'])

        print('TASK RECEIVED: %s[%s] %s' % (
            task.name, task.uuid, task.info(),))
            
    with app.connection() as connection:
        recv = app.events.Receiver(connection, handlers={
                'task-failed': announce_failed_tasks,
                'task-received': announce_receive_tasks,
                '*': state.event,
        })
        recv.capture(limit=None, timeout=None, wakeup=True)

if __name__ == '__main__':
    print 'monitor start'
    app = Celery(broker='redis://127.0.0.1:6379/1')
    my_monitor(app)
```

可以监听到的所有事件在这里
http://docs.celeryproject.org/en/latest/userguide/monitoring.html#event-reference

#### 4. 其他补充

Celery本身运行的worker进程直接监控运行中的进程状态即可。
比如使用supervisor管理进程。这里不再赘述







