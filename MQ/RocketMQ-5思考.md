**幂等，幂等，幂等！**

 应用程序在使用RocketMQ进行消息消费时必须支持幂等消费，即同一个消息被消费多次和消费一次的结果一样。这一点在使用RoketMQ或者分析RocketMQ源代码之前再怎么强调也不为过。

  “至少一次送达”的消息交付策略，和消息重复消费是一对共生的因果关系。要做到不丢消息就无法避免消息重复消费。原因很简单，试想一下这样的场景：客户端接收到消息并完成了消费，在消费确认过程中发生了通讯错误。从Broker的角度是无法得知客户端是在接收消息过程中出错还是在消费确认过程中出错。为了确保不丢消息，重发消息是唯一的选择。

 有了消息幂等消费约定的基础，RocketMQ就能够有针对性地采取一些性能优化措施，例如：并行消费、消费进度同步机制等，这也是RocketMQ性能优异的原因之一。





**（会导致重复的地方）Duplication**

Many circumstances could cause duplication, such as:

- Producer resend messages(i.e, in case of FLUSH_SLAVE_TIMEOUT)
- Consumer shutdown with some offsets not updated to the Broker in time.

So you may need to do some external work to handle this if your application cannot tolerate duplication. For example, you may check the primary key of your DB.

 http://rocketmq.apache.org/docs/best-practice-consumer/ 