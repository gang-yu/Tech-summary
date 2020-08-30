---**队列本身**---

## 消息量太大，一时消费不了，怎么办？
hint：惰性队列


## 消费者处理不过来而宕机。重启之后再次因为处理不了而宕机。怎么办？
A：
通知MQx限流



---**队列综合应用**---

## 事务中包含消息发送，该如何处理好？
hint：一致性
A：
发送消息->DB:
如果消息发送成功而DB失败，会出现一致性问题。
Db->发送消息：
如果DB成功而消息发送失败，可以进行DB的补偿或者回滚，从而保证一致性。
这需要在生产者/消费者两端都增加DB表来对应待发送、已收取的消息。

* 如下是一个他人针对activemq的实现。
  https://github.com/zjpjohn/ActiveMQ
  这个repo中，分3步来解决：事务消息只存入DB，事务成功时将DB数据信息key存入本地cache，本地cache/定时任务触发DB数据发送到MQ。



## 如何保证消息不丢失？
hint：
确认机制+重试+消息持久化




## 如何处理重复消息？
hint：生产者、MQ、消费者
A：
生产者没有收到ack确认，生产者就可以重试，多次发送。需要MQ能唯一标识一条消息。
消费者没有回复ack给MQ，MQ就可以重试，对此推送。需要消费者能唯一标识一条消息，同时做到处理能力的幂等性。




## 消息的到达顺序与发送顺序不一致？顺序不是严格要求一致
比如，第三方支付的回报，第2条成功回报先到，第1条收到回报后到。
虽然这个例子与MQ无关，但是MQ也会碰上。尤其是消息分发给消费者集群中的不同service的时候。
hint：
服务要robust，比如有状态机的判断，这样一来，状态不会退化，避免了错误。




## 消息的到达顺序与发送顺序不一致？顺序要求严格一致
比如，OA审批流程的消息，前一个节点消息是后一个节点的判断依据。
hint
我想想，这个有点搞大了



## MQ如果要做HA高可用高并发，该怎么做？
Ans:
https://blog.csdn.net/shuangzh115/article/details/50989182
单mq broker，有单点，不是高可用。
master/slave 主从broker，能够实现HA, 避免单点故障。但是处理能力并没有增加。在master因为高负荷而down，那么slave切换为主时，也会很快挂掉，只是时间问题。
一个办法是activemq集群。多activemq broker同时工作，每个topic在多个boker上都有。1到2个mq实例出现问题时，消息可经过其他broker处理，整个系统依然可以健康工作，从而实现高可用。采用轮循方式给多个broker发送消息，并发能力也能提升。
如下是一个参考实现
https://gitee.com/shuangzh/toolkit/tree/master/toolkit/src/main/java/com/shuangzh/toolkit/activemq



## rocketmq怎么保证队列完全顺序消费？

Ans:

 Jaskey Lam的回答 - 知乎 https://www.zhihu.com/question/30195969/answer/142416274

从rocketmq的集群消费模式看，生产者、消费者的消息都会被broker做负载均衡。

* 在broker数量不变的情况下，单个message queue中的消息，其消费顺序能保证。或者使用sharding key，让某个需要有序的消息，全部进入到同一个队列，那么就能做到。

* 如果broker的数量有增减(宕机/重启/扩容/缩容)，某个queue的消息，在rebalance后(轮询取模的参数变了)会分发给不同的consumer，这样顺序就很难保证了。
  * 如果还要保证顺序，那么对生产者，要选择牺牲掉failover特性，等待宕机恢复或者不发送接下来的消息。
  * 如果还要保证顺序，那么对消费者，无法跳过当前消息，必须暂停整个队列的消费



#### 解耦MQ与DB事务的框架

https://github.com/zjpjohn/ActiveMQ
https://blog.csdn.net/shuangzh115/article/details/50989182
综合这两个可以看出来，可以研发一个封装mq中间件，他封装如下功能：

1. 存储消息到DB，做到消息与db的事务一致性
2. 事务完成/定时任务触发消息到本地cache
3. 本地cache触发mq broker路由、以及消息发送MQ