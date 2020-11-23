## What is MQ? ##


All the explanations so far are accurate and to the point - but might be missing something: one of the main benefits of message queueing: resilience.

Imagine this: you need to communicate with two or three other systems. A common approach these days will be web services which is fine if you need an answers right away.

However: web services can be down and not available - what do you do then? Putting your message into a message queue (which has a component on your machine/server, too) typically will work in this scenario - your message just doesn't get delivered and thus processed right now - but it will later on, when the other side of the service comes back online.

So in many cases, using message queues to connect disparate systems is a more reliable, more robust way of sending messages back and forth. It doesn't work well for everything (if you want to know the current stock price for MSFT, putting that request into a queue might not be the best of ideas) - but in lots of cases, like putting an order into your supplier's message queue, it works really well and can help ease some of the reliability issues with other technologies.

## Why use MQ?

解耦、异步、削峰

传统模式的缺点：

系统间耦合性太强，如上图所示，系统A在代码中直接调用系统B和系统C的代码，如果将来D系统接入，系统A还需要修改代码，过于麻烦！
一些非必要的业务逻辑以同步的方式运行，太耗费时间。
并发量大的时候，所有的请求直接怼到数据库，造成数据库连接异常

中间件模式的的优点：

将消息写入消息队列，需要消息的系统自己从消息队列中订阅，从而系统A不需要做任何修改。
将消息写入消息队列，非必要的业务逻辑以异步的方式运行，加快响应速度
系统A慢慢的按照数据库能处理的并发量，从消息队列中慢慢拉取消息。在生产中，这个短暂的高峰期积压是允许的。

MQ的问题：
可用性降低
复杂度/不稳定性增加


## 如何实现高可用性？
集群化
Producer 与 NameServer集群中的其中一个节点（随机选择）建立长连接，定期从 NameServer 获取 Topic 路由信息，并向提供 Topic 服务的 Broker Master 建立长连接，且定时向 Broker 发送心跳。Producer 只能将消息发送到 Broker master，但是 Consumer 则不一样，它同时和提供 Topic 服务的 Master 和 Slave建立长连接，既可以从 Broker Master 订阅消息，也可以从 Broker Slave 订阅消息。

类似于kafka
一个典型的Kafka集群中包含若干Producer（可以是web前端产生的Page View，或者是服务器日志，系统CPU、Memory等），若干broker（Kafka支持水平扩展，一般broker数量越多，集群吞吐率越高），若干Consumer Group，以及一个Zookeeper集群。Kafka通过Zookeeper管理集群配置，选举leader，以及在Consumer Group发生变化时进行rebalance。Producer使用push模式将消息发布到broker，Consumer使用pull模式从broker订阅并消费消息。

