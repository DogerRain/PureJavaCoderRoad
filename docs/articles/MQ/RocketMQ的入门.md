[RocketMQ的介绍](https://rain.baimuxym.cn/article/49)

[RocketMQ(1)-架构原理](https://www.cnblogs.com/qdhxhz/p/11094624.html)

[rocketmq-常见问题总结(消息的顺序、重复、消费模式)](https://www.cnblogs.com/xuwc/p/9034352.html)



## 1、几种消息队列的比较

### kafka

优点：吞吐量大，在常规机器配置下，单机可以达到几十万的QPS，非常强悍。性能高，基本发消息都是毫秒级。可用性高，支持集群，部分机器可以宕机。

缺点：丢数据，kafka收到消息后会写入一个磁盘缓冲区，并没有直接落地到硬盘，所以要是机器本身故障了，可能会导致磁盘缓冲区里的数据丢失。功能比较单一。 一般适用于日志分析场景。kafkaStrem流式计算。

### RabbitMQ

优点：可以保证数据不丢失，也能保证高可用性。支持高级功能，如死信队列、消息重试等。

缺点：吞吐量比较低，一般就是每秒几万的级别，而且消息堆积量太高就会影响整体性能。erlang语言编写，很难扩展。

### RocketMQ

优点：几乎同时解决了kafka和RabbitMQ的缺陷吞吐量高，单机可以达到10WQPS，高可用，性能高，也支持延迟消息、事务消息、消息回溯、死信队列等。消息积压也可以很多。使用java开发，可以灵活扩展

缺点：官方文档比较简单，社区活跃度没有RabbitMQ高。云上版本比开源版本强，所以要用好，基
本上都需要定制。客户端只支持java





## 2、搭建环境

https://blog.csdn.net/weidong22/article/details/79246726



为什么有有序？



如果不指定MessageQueue，默认是轮流发到不同的MessageQueue上的，所以消费的时候就可能回乱序。

如果指定了，就会把这批消息放在同一个MessageQueue。