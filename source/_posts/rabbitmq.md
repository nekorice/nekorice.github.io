title: AMQP 消息协议和 rabbitmq入门
date: 2017-05-17 17:00:21
tags:
- amqp
- rabbitmq
- messagequeue
---

## 1.一些概念

AMQP 协议全称Advanced Message Queuing Protocol.是一个规定了客户端如何和消息中间件进行通讯的消息协议.

消息中间件(brokers),就是从 publishers (也就是所谓的生产者)接受消息,然后进行处理,转发给 consumers(消费者)

AMQP 是一个网络传输的协议, brokers,publishers,consumers 都是支持集群和扩展的.

Rabbitmq 就是扮演其中的 brokers. 目前支持的协议是 AMQP 0-9-1模型

<!-- more  -->

## 2.AMQP

基本的消息流程类似图中所示

![rabbitmq](https://www.rabbitmq.com/img/tutorials/intro/hello-world-example-routing.png)

Exchange 是个 AMQP 实体, 定义了消息应该怎样路由给 Queue
Queue 则是一个先进先出的队列
每个 Queue 需要声明绑定到某个 Exchange 才可以接受消息.
Publisher就是我们自己编写的消息发送方程序
Consumer 就是消息接收方

### 2.1 Exchange

Exchange 主要有以下几种


|Name Default | pre-declared names|
|-------------|-------------------|
|Direct exchange |(Empty string) and amq.direct|
|Fanout exchange | amq.fanout |
|Topic exchange | amq.topic |
|Headers exchange | amq.match (and amq.headers in RabbitMQ)|

#### 1)  Direct Exchange

每个 Queue 通过 routing_key 和 Exchange 绑定
发送的消息根据 routing key 发送到指定队列

#### 2)  Fanout Exchange

就是广播到所有队列
绑定时不用带 routing key

#### 3)  Topic Exchange

则有点像订阅模式,发布给订阅了该主题的队列
也是使用绑定时的 routing key 来处理发布订阅的逻辑.
Queue 绑定的时候可以使用通配符来定义自己需要订阅的routing key模式

#### 4)  Headers Exchange 
不是使用 route key 来进行路由,而是使用 message 的 Header 来进行路由
自定义每个消息都使用的头部,来发送消息,并进行路由.
这样可以在发送消息的时候动态设定发动到哪个队列

除此之外还有一个名字为空字符的默认 Exchange, 这个 Exchange 也是一个 direct Exchange, 每个队列创建的时候就会默认绑定到这个 Exchange 上.而 routing key, 就是每个Queue 的名字.


另外Exchange 有如下几个属性:

* Name 名称
* Durability (exchanges survive broker restart) 设置持久化有可能在消息队列发生消息堆积导致大量信息遗保存硬盘文件上.可能会导致硬盘空间被占满需要注意.
* Auto-delete (exchange is deleted when all queues have finished using it)
* Arguments (these are broker-dependent)

### 2.2 Queue

AMQP 模型下的 Queue 有这些属性

* Name
* Durable (the queue will survive a broker restart)
* Exclusive (used by only one connection and the queue will be deleted when that connection closes)
只允许一个连接
* Auto-delete (queue is deleted when last consumer unsubscribes)
* Arguments (some brokers use it to implement additional features like message TTL)

大部分参数都和 Exchange 一样.多了一个 Exclusive 的参数.主要是设定某个队列必须串行处理用的.

### 2.3 消息相关

AMQP 设定了接受消息可以使用推( push API)和拉(pull API)的模型.

关于消息的删除, AMQP 也定义了两种模式,一种是一旦发送到对应的 consumer 就自动删除消息,还有一种则需要 consumer 使用basic.ack AMQP 方法来回应这个消息, broker 才会把这个消息移除.

在需要 basic.ack 的消息模型中,如果一个 cosumer 一直没有 ack 一个消息,它就不会收到下一个消息.而且在这种情况当没有 consumer 接收消息时,消息会堆积在 borker 上,导致硬盘空间被占用.

所以如果使用 Durable 持久化队列,请务必设定Queue的名字,保证在 consumer 崩溃后可以连回原来的 Queue, 以免 publisher 一直在往一个没有 consumer 接收信息的队列里面发消息,导致硬盘空间爆炸.


## 3. Rabbitmq
### 3.1 安装和配置

安装方法参考官方文档
https://www.rabbitmq.com/download.html

rabbitmq 的配置分成在配置文件里面设定和环境变量设定两种.

配置文件 rabbitmq.conf的语法比较蛋疼,是使用Erlang 的 config 语法.感觉就像一个数组的配置.最后的.号不要漏掉了

比如远程连接的设置:

[
{rabbit, [{tcp_listeners, [{"127.0.0.1", 5672},
                           {"0.0.0.0", 10087}]}, 
          {msg_store_file_size_limit, 504800},
          {loopback_users, ["guest"]}]}
].

连接用的用户名和密码则使用 rabbitmqctl 来创建

官方提供的一个配置例子,可以作为参考
https://github.com/rabbitmq/rabbitmq-server/blob/stable/docs/rabbitmq.config.example

对于环境变量的设置项,按照文档,在 shell中设定的全局的 env 会覆盖 rabbimq-env.conf 的设置.然而这边实际使用时,在 debian下设定全局的环境变量并不生效.最后修改了在 rabbitmq 的配置目录下的 rabbitmq-env.conf才生效.

https://www.rabbitmq.com/relocate.html中列出了一些目录配置,
主要修改这些环境变量来配置 rabbitmq的目录相关的信息,比如RABBITMQ_MNESIA_DIR是默认存放队列和持久化.默认 rabbitmq 把持久化放在/ var 下面,很多情况这里并没有多少空间来存放大量的数据.


### 3.2 rabbitmqctl

这是一个用来查看rabbitmq 状态以及操作 rabbitmq的命令行工具

常用操作:

```
rabbitmqctl start 
rabbitmqctl stop stop 整个 rabbitmq 和对应的 erlang node
rabbitmqctl start_app 
rabbitmqctl stop_app  注意 stop_app 会移除所有的 virtual host 和用户的配置.
这个不会 stop erlang node, 不过会终止 rabbitmq

rabbitmqctl list_queues  列出所有队列,参数可以选择显示具体内容
rabbitmqctl list_exchanges 列出 exchang
rabbitmqctl  list_connections

rabbitmqctl add_user 增加用户
rabbitmqctl set_permissions 增加权限

purge_queue  移除队列里的所有消息
```

更多信息参考官方文档:
https://www.rabbitmq.com/man/rabbitmqctl.1.man.html


### 3.3 方法

rabbitmq 就是对 amqp 中的 broker 的方法进行了实现(其中带 xxx-ok的方法).而 client 则需要每个语言自己实现客户端的方法.

[https://www.rabbitmq.com/amqp-0-9-1-reference.html](https://www.rabbitmq.com/amqp-0-9-1-reference.html)

主要的使用方法并不难,就是

1.  declear_exchange
2.  declear_queue
3.  bind Queue to Exchange

publisher 使用 basic_publish 来发布消息
consumer 使用 basic_consumer 来接受消息,并且处理完毕之后使用 basic_ack, 发送消息消费确认.

这方面可以一边参考官方的[6个示例](https://www.rabbitmq.com/getstarted.html)来进行理解.





